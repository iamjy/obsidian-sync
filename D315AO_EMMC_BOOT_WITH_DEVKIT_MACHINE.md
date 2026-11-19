# D315AO eMMC 기본 부팅 확인 가이드 — 개발킷 MACHINE, Docker 22.04 빌드 → 24.04 호스트 플래시

이 문서는 [`../sdk-analysis/D315AO_CUSTOM_LAYER_NECESSITY.md`](../sdk-analysis/D315AO_CUSTOM_LAYER_NECESSITY.md)의 **권장 순서 1단계**를 실행하기 위한 가이드다.

> 1단계: 커스텀 레이어 없이 meta-tegra 개발킷 MACHINE(`p3737-0000-p3701-0005`)으로 이미지를 빌드해 D315AO의 **eMMC 부팅**을 확인한다.

명령마다 **왜 실행하는지**를 붙였다. 프롬프트 표기는 다음과 같다.

| 표기 | 실행 위치 |
|------|-----------|
| `host$` | Ubuntu 24.04 호스트 (`louis`) |
| `ctr$` | 기존 Ubuntu 22.04 컨테이너 `prj-aicamsys-medspa-js-22.04` (사용자 `dinsight`, uid 1000) |
| `d315#` | D315AO 보드의 시리얼 콘솔 (root) |

---

## 0. 범위와 전제

### 0.1 이 단계에서 확인하는 것 / 확인하지 않는 것

| 확인한다 | 확인하지 않는다 (이후 단계) |
|----------|-----------------------------|
| Docker 22.04 컨테이너에서 meta-tegra 이미지가 끝까지 빌드되는지 | D315AO 전용 pinmux/GPIO (2·3단계, 커스텀 레이어) |
| 빌드 결과를 24.04 호스트에서 D315AO에 플래시할 수 있는지 | NVMe 부팅 (3단계 이후) |
| D315AO가 eMMC rootfs로 부팅해 시리얼 콘솔에 로그인되는지 | 캐리어 기능(USB 포트 배치, 오디오, 디스플레이, CAN 등) |

**왜 개발킷 MACHINE으로 충분한가?**
`D315AO_CUSTOM_LAYER_NECESSITY.md` 분석 결과, D315AO BSP와 개발킷 MACHINE은 부팅 체인 설정이 같다.

- 같은 항목: 메모리 BCT, PMIC, BPMP DTB, 파티션 레이아웃, ODMDATA, padvoltage
- 다른 항목: MB1 pinmux/GPIO, 커널 DTB(USB/오디오/디스플레이/UART)

eMMC와 디버그 UART는 모듈 쪽 자원이라 부팅까지는 될 가능성이 높다. 다만 **실제 보드로 확인한 적은 없으므로**, 이 문서가 그 확인 절차다.

### 0.2 통과 기준

1. `bitbake core-image-base` 가 컨테이너에서 에러 없이 끝난다.
2. 호스트에서 `doflash.sh` 가 `Flashing completed`(또는 동등한 성공 메시지)로 끝난다.
3. D315AO가 재부팅되어 시리얼 콘솔에 `root` 로 로그인된다.
4. `findmnt /` 의 SOURCE가 **`/dev/mmcblk0p1`** 이다.
5. 재부팅 후에도 3·4가 같다.

### 0.3 이 문서를 쓰면서 실제로 확인한 것 (2026-11-17)

| 항목 | 결과 |
|------|------|
| 컨테이너 | `prj-aicamsys-medspa-js-22.04` 가 실행 중이다(이미지 `dinsight/prj-aicamsys-medspa-js-22.04:0.0.0`). Ubuntu 22.04.5, Python 3.10.12, uid/gid 1000(`dinsight`) |
| 경로 매핑 | 호스트 `/home/louis/work` → 컨테이너 **`/home/1000/work`** (같은 경로가 **아니다**). 1.3 참고 |
| 로케일 | 컨테이너 기본 `LANG` 이 **비어 있다**. `en_US.utf8` 로케일은 설치돼 있다 → 2.2에서 export |
| 빌드 패키지 | chrpath, diffstat, gawk, cpio, socat, makeinfo, lz4, zstd, python3-git/jinja2/pexpect/subunit 모두 있음 |
| user namespace | 컨테이너 안 `unshare -rn true` **성공** → BitBake의 태스크 네트워크 격리가 동작한다 |
| 기존 빌드 디렉터리 | `build-docker-agx-orin-emmc` 가 이 컨테이너에서 이미 만들어져 있다. `bblayers.conf` 에 meta-tegra가 추가돼 있고, `local.conf` 는 아직 기본값(MACHINE 미설정)이다 |
| 파싱 검증 | 컨테이너에서 `MACHINE=p3737-0000-p3701-0005` 로 변수를 조회했다. `NATIVELSBSTRING=ubuntu-22.04`, `KERNEL_DEVICETREE=tegra234-p3737-0000+p3701-0005-nv.dtb`, `IMAGE_FSTYPES=tegraflash`, `TNSPEC_BOOTDEV=mmcblk0p1` |
| 레이어 커밋 | poky `scarthgap`, meta-tegra `scarthgap` `e7d217e2` (L4T R36.5.2) |
| 플래시 시 보드 확인 | meta-tegra `tegra-flash-helper.sh` 는 **모듈(CVM) EEPROM만** 읽는다(L329). 캐리어 EEPROM이 개발킷과 달라도 플래시가 막히지 않는다. 모듈 SKU는 검사한다(L383) |
| 호스트 도구 | `python3-yaml`, `dtc`, `cpp`, `sgdisk`, `udisksctl`, `minicom`, `screen`, `lsusb` 있음. `picocom` 없음 |
| ⚠️ **디스크** | `$YROOT` 가 있는 `/`(nvme0n1p3) 여유 **23 GB(98%)** — **빌드하기에 부족하다**. 1.2 참고 |

---

## 1. 사전 확인 (호스트)

### 1.1 작업 루트

```bash
host$ cd ~/work/project/jeisys-medical/aicamsys-medspa-js/sdk/AVerMedia/Yocto/scarthgap
host$ export YROOT=$PWD
host$ ls $YROOT
# build-docker-agx-orin-emmc  meta-tegra  poky  sdk-analysis  sdk-guide ...
```

**왜?** 이후 호스트 명령에서 작업 루트를 짧게 쓰기 위해서다. 새 터미널을 열 때마다 다시 설정한다.

### 1.2 디스크 확보 — 빌드 전 필수

```bash
host$ df -h $YROOT
host$ du -xsh ~/work/* 2>/dev/null | sort -h | tail -15
```

**왜?**
- 빌드 결과(`downloads/`, `sstate-cache/`, `build-*/tmp`)는 컨테이너가 아니라 **bind mount된 호스트 디스크**(`/`)에 쌓인다.
- meta-tegra `core-image-base` 한 벌의 예상 사용량은 다음과 같다(추정치, 실측 아님).

  | 항목 | 예상 크기 |
  |------|-----------|
  | `downloads` | 약 15 GB |
  | `sstate-cache` | 약 15~20 GB |
  | `tmp` (`rm_work` 사용 시) | 약 30~40 GB |
  | 합계 | 약 60~80 GB |

- 현재 여유는 23 GB뿐이다. 이 상태로 시작하면 `BB_DISKMON_DIRS` 가 빌드를 중간에 멈추거나(`STOPTASKS`/`HALT`), 디스크가 가득 차서 호스트 전체가 불안정해진다. **여유 100 GB 이상을 확보한 뒤 진행할 것을 권장한다.**
- `du` 로 큰 디렉터리를 찾는다. 무엇을 지울지는 직접 판단해야 한다(이 문서는 아무것도 지우지 않는다).

참고:
- `docker system df` 에 표시되는 회수 가능 공간(이미지 약 237 GB, 빌드 캐시 약 17 GB)은 **다른 디스크(nvme1n1, `/var/lib/docker`)** 에 있다. 지워도 `/` 의 여유는 늘지 않는다.
- 컨테이너는 `/home/louis/work` 만 마운트하고 있다. 다른 디스크를 쓰려면 컨테이너를 다시 만들어야 하므로 이 문서 범위 밖이다.
- `DL_DIR`/`SSTATE_DIR` 은 NFS에 둘 수 있다. `TMPDIR` 은 NFS에 둘 수 없다(BitBake sanity 검사에서 거부된다).

### 1.3 경로 매핑 이해

| 호스트 | 컨테이너 |
|--------|----------|
| `/home/louis/work/project/.../Yocto/scarthgap` | `/home/1000/work/project/.../Yocto/scarthgap` |

**왜 중요한가?**
- `build-docker-agx-orin-emmc/conf/bblayers.conf` 에는 **컨테이너 경로**(`/home/1000/...`)가 적혀 있다. 그래서 이 빌드 디렉터리의 `bitbake` 는 **반드시 이 컨테이너 안에서** 실행한다. 호스트에서 실행하면 레이어 경로를 찾지 못한다.
- 플래시는 호스트에서 한다. 플래시에 쓰는 `*.tegraflash.tar.gz` 는 그 자체로 완결된 패키지라 경로 차이의 영향을 받지 않는다. 호스트에서는 `/home/louis/...` 경로로 같은 파일을 읽으면 된다.

### 1.4 모듈 SKU 확인 (MACHINE 선택)

D315AO에 지금 AVerMedia BSP(Ubuntu)가 설치돼 있다면 보드에서 확인한다.

```bash
d315$ cat /proc/device-tree/nvidia,dtsfilename; echo
d315$ free -g | head -2
```

**왜?**
- meta-tegra 플래시 도구는 모듈 EEPROM의 SKU를 MACHINE과 비교하고, 다르면 `ERR: actual board SKU ... does not match expected board SKU` 로 멈춘다. 몇 시간 걸리는 빌드를 하기 전에 맞는 MACHINE을 정해 둔다.
- 다음과 같이 고른다.

  | 확인 결과 | 모듈 | MACHINE |
  |-----------|------|---------|
  | `p3701-0005` 포함 / 메모리 약 64 GB | 64GB 모듈 | `p3737-0000-p3701-0005` (이 문서 기준) |
  | `p3701-0004` 포함 / 메모리 약 32 GB | 32GB 모듈 | `p3737-0000-p3701-0004` |

- AVerMedia BSP의 flash 이미지에는 `p3701-0005-nv-d315`, `bpmp-3701-0005` 가 들어 있어서 0005일 가능성이 높다. 그래도 보드에서 직접 확인하는 것이 확실하다.
- 보드에 접근할 수 없으면 0005로 진행한다. SKU가 틀리면 7.2 플래시 단계에서 위 에러로 멈추므로 보드에 기록되기 전에 알 수 있다.

---

## 2. 컨테이너 진입과 셸 환경

### 2.1 호스트 tmux 안에서 컨테이너 셸 열기

```bash
host$ tmux new -s yocto-d315ao
host$ docker ps --filter name=prj-aicamsys-medspa-js-22.04 --format '{{.Names}} {{.Status}}'
host$ docker start prj-aicamsys-medspa-js-22.04      # Status가 Exited일 때만
host$ docker exec -it prj-aicamsys-medspa-js-22.04 bash
```

**왜?**
- `tmux` : 빌드는 몇 시간 걸린다. `docker exec` 로 띄운 셸은 호스트 터미널이 닫히면 함께 종료되고, 그 아래의 bitbake도 같이 끊긴다. 호스트 tmux 안에서 실행하면 터미널을 닫아도 빌드가 유지된다. 다시 붙을 때는 `tmux attach -t yocto-d315ao` 를 쓴다.
- `docker ps` / `start` : 이 컨테이너는 `--rm` 없이 상주하는 개발 컨테이너다. 호스트를 재부팅했다면 멈춰 있을 수 있다.
- `docker exec` : 새 컨테이너를 만들지 않고 기존 컨테이너에 셸을 하나 더 연다. 컨테이너 기본 사용자가 `1000:1000` 이라 root가 아니다. BitBake는 root로 실행하면 거부하므로 이 상태가 맞다.

### 2.2 로케일과 작업 루트

```bash
ctr$ export LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
ctr$ cd /home/1000/work/project/jeisys-medical/aicamsys-medspa-js/sdk/AVerMedia/Yocto/scarthgap
ctr$ export YROOT=$PWD
ctr$ id -u; df -h $YROOT | tail -1
```

**왜?**
- `LANG`/`LC_ALL` : 이 컨테이너는 `LANG` 이 비어 있다(0.3). BitBake는 UTF-8 로케일이 아니면 sanity 검사에서 멈춘다(`Please use a locale setting which supports UTF-8`). **셸을 새로 열 때마다** 실행한다.
- `cd` : 컨테이너 쪽 경로(`/home/1000/...`)를 쓴다(1.3).
- `id -u` 는 `1000` 이어야 한다. 그래야 호스트에서 결과 파일의 소유자가 `louis` 가 된다.
- `df` 로 1.2에서 확보한 공간이 컨테이너 안에서도 보이는지 확인한다(같은 디스크다).

---

## 3. 빌드 설정 (컨테이너)

### 3.1 레이어 버전 기록

```bash
ctr$ git -C poky       log -1 --oneline; git -C poky       branch --show-current
ctr$ git -C meta-tegra log -1 --oneline; git -C meta-tegra branch --show-current
```

**왜?** 나중에 "어떤 BSP 버전으로 부팅을 확인했는가"를 기록하기 위해서다. 기대값: poky `scarthgap`, meta-tegra `scarthgap` `e7d217e2`.
meta-tegra 커밋에 따라 L4T 버전(현재 R36.5.2)이 달라진다.

### 3.2 빌드 환경 초기화

```bash
ctr$ source poky/oe-init-build-env build-docker-agx-orin-emmc
ctr$ bitbake-layers show-layers
```

**왜?**
- `source oe-init-build-env <dir>` : 현재 셸에 `bitbake` PATH와 `BUILDDIR` 를 설정하고 빌드 디렉터리로 `cd` 한다. 이미 있는 `conf/` 는 덮어쓰지 않는다. `source` 로 실행해야 셸에 반영된다.
- 기존 디렉터리 `build-docker-agx-orin-emmc` 를 그대로 쓴다. 이 컨테이너에서 만들었고 meta-tegra가 이미 추가돼 있기 때문이다(0.3).
- `show-layers` 에 `core`, `yocto`, `yoctobsp`, `tegra` 4개가 보여야 한다. `tegra` 가 없으면 다음을 실행한다.
  ```bash
  ctr$ bitbake-layers add-layer $YROOT/meta-tegra
  ```

### 3.3 `conf/local.conf` 설정 추가

먼저 이전에 추가한 블록이 없는지 확인한다(두 번 추가하면 뒤 값이 이긴다).

```bash
ctr$ grep -n 'D315AO step1' conf/local.conf || echo "not yet"
```

`not yet` 이 나오면 아래 블록을 추가한다.

```bash
ctr$ cat >> conf/local.conf <<'EOF'

# ===== [D315AO step1] devkit MACHINE, eMMC rootfs, Docker 22.04 build =====
MACHINE = "p3737-0000-p3701-0005"

DL_DIR     = "${TOPDIR}/../downloads"
SSTATE_DIR = "${TOPDIR}/../sstate-cache"

INIT_MANAGER = "systemd"

BB_NUMBER_THREADS = "12"
PARALLEL_MAKE     = "-j 12"

INHERIT += "rm_work"

IMAGE_INSTALL:append = " nv-tegra-release pciutils usbutils util-linux e2fsprogs gptfdisk tegra-tools i2c-tools"
EXTRA_IMAGE_FEATURES += "ssh-server-openssh"
EOF
```

**각 줄이 왜 필요한가**

| 설정 | 이유 |
|------|------|
| `MACHINE = "p3737-0000-p3701-0005"` | 개발킷 MACHINE. 1.4에서 32GB 모듈로 확인됐으면 `p3737-0000-p3701-0004` 로 바꾼다. `local.conf` 기본값이 `MACHINE ??= "qemux86-64"`(약한 기본값)이므로 여기서 `=` 로 지정하면 덮어쓴다. |
| `DL_DIR` / `SSTATE_DIR` | `${TOPDIR}/..` = `$YROOT`. 빌드 디렉터리를 지우거나 이후 커스텀 레이어 단계(2·3단계)용 빌드 디렉터리를 새로 만들어도 다운로드와 sstate를 재사용한다. **2단계 빌드 시간이 크게 줄어든다.** |
| `INIT_MANAGER = "systemd"` | poky 기본은 sysvinit이다. meta-tegra/L4T 서비스(nvstartup, nvpmodel 등)는 systemd 기준이다. |
| `BB_NUMBER_THREADS` / `PARALLEL_MAKE` = 12 | 이 PC는 32코어 / 31 GB RAM이지만 다른 컨테이너·서비스가 약 15 GB를 쓰고 있다. 남은 약 16 GB에서 C++ 컴파일·링크가 동시에 돌 때 OOM을 피하려고 12로 제한한다. 여유가 생기면 16으로 올려도 된다. |
| `INHERIT += "rm_work"` | 레시피 빌드가 끝나면 작업 디렉터리를 지운다. 디스크가 부족한 이 환경에서는 사실상 필수다(1.2). |
| `IMAGE_INSTALL:append` | 부팅 확인과 2단계(차이 기록)에 쓸 도구들이다. `nv-tegra-release`(`/etc/nv_tegra_release`), `lspci`, `lsusb`, `lsblk`/`findmnt`, `sgdisk`, `tegrastats`, `i2cdetect`. **앞의 공백이 반드시 필요하다**(`:append` 는 공백을 넣어 주지 않는다). |
| `ssh-server-openssh` | 이더넷이 잡히면 SSH로도 접속할 수 있게 한다. poky 기본 `debug-tweaks` 때문에 root에 암호가 없다(**개발용만**). |

### 3.4 설정 확인 (빌드 전)

```bash
ctr$ bitbake-getvar --value MACHINE
ctr$ bitbake-getvar --value NATIVELSBSTRING
ctr$ bitbake-getvar --value KERNEL_DEVICETREE
ctr$ bitbake-getvar -r core-image-base --value TNSPEC_BOOTDEV
```

**왜?** `local.conf` 오타나 중복을 빌드 전에 잡기 위해서다. 0.3에서 확인한 기대값은 다음과 같다.

| 변수 | 기대값 | 의미 |
|------|--------|------|
| `MACHINE` | `p3737-0000-p3701-0005` | 3.3이 반영됨 |
| `NATIVELSBSTRING` | `ubuntu-22.04` | 22.04 컨테이너에서 빌드 중. native sstate가 이 이름으로 저장된다 |
| `KERNEL_DEVICETREE` | `tegra234-p3737-0000+p3701-0005-nv.dtb` | **개발킷 DTB**. D315 DTB가 아니라는 점을 기억해 둔다(8.3) |
| `TNSPEC_BOOTDEV` | `mmcblk0p1` | rootfs가 **eMMC** 로 설정됨 ← 이 단계의 핵심 |

---

## 4. 다운로드 → 빌드 (컨테이너)

### 4.1 다운로드만 먼저

```bash
ctr$ time bitbake core-image-base --runall=fetch
```

**왜?**
- 컴파일 없이 소스와 바이너리만 먼저 받는다. NVIDIA 서버의 L4T 패키지, GitHub의 커널·edk2·OP-TEE 소스를 받다가 생기는 네트워크 오류를 빌드 중간이 아니라 여기서 걸러낸다.
- 끝나면 `df -h $YROOT` 로 디스크를 다시 본다. 이 시점에 `downloads/` 가 약 15 GB 늘어난다. 남은 여유가 50 GB보다 작으면 빌드를 시작하지 말고 1.2로 돌아간다.

### 4.2 이미지 빌드

```bash
ctr$ time bitbake core-image-base
```

**왜?**
- 크로스 툴체인 → 커널 + NVIDIA OOT 모듈 → UEFI(edk2) → OP-TEE → rootfs → `do_image_tegraflash`(플래시 패키지 생성) 순서로 실행된다. 첫 빌드는 1.5~3시간 걸린다(추정).
- 중간에 끊겨도 같은 명령을 다시 실행하면 이어서 진행한다(끝난 태스크는 sstate에서 가져온다).
- 실패하면 마지막에 출력된 `ERROR: <recipe> do_<task>` 의 `log.do_<task>` 경로를 열어 원인을 확인한다(9장).

### 4.3 산출물 확인

```bash
ctr$ ls -lh tmp/deploy/images/p3737-0000-p3701-0005/ | grep -E 'tegraflash|\.dtb$'
```

**왜?**
- 플래시에 쓸 파일은 `core-image-base-p3737-0000-p3701-0005.rootfs.tegraflash.tar.gz` 하나다. 이 이름은 심볼릭 링크이고, 실제 파일 이름에는 타임스탬프가 붙는다.
- `.dtb` 목록에서 `tegra234-p3737-0000+p3701-0005-nv.dtb` 를 확인한다. 이번 이미지에 들어간 커널 DTB이며, 개발킷용이다.

이후 단계는 전부 호스트에서 한다. 컨테이너 셸은 `exit` 로 닫아도 된다. 컨테이너 자체는 계속 실행 상태로 둔다.

---

## 5. 호스트 플래시 준비 (1회)

```bash
host$ sudo apt install -y picocom
host$ sudo usermod -aG plugdev,dialout $USER
host$ cat <<'EOF' | sudo tee /etc/udev/rules.d/70-jetson-recovery.rules
SUBSYSTEM=="usb", ATTRS{idVendor}=="0955", MODE="0664", GROUP="plugdev"
EOF
host$ sudo udevadm control --reload-rules && sudo udevadm trigger
host$ cpp --version | head -1
host$ python3 -c 'import yaml; print("pyyaml", yaml.__version__)'
```

**왜?**
- 호스트는 빌드를 하지 않는다. 그래서 Yocto 빌드 패키지는 필요 없고, 플래시 패키지 안의 스크립트가 부르는 도구만 있으면 된다. `dtc`, `sgdisk`, `udisksctl`, `python3-yaml` 은 이미 있다(0.3).
- `picocom` : D315AO 디버그 UART용 시리얼 콘솔이다. 지금은 없다. 이미 설치된 `minicom` 이나 `screen` 을 써도 된다.
- `plugdev` + udev 규칙 : 리커버리 모드 Jetson(USB vendor `0955`)에 `sudo` 없이 접근하기 위해서다.
- `dialout` : `/dev/ttyUSB*` 시리얼 장치에 접근하기 위해서다.
- 그룹 변경은 **다시 로그인해야** 적용된다. 건너뛰면 플래시와 시리얼 명령에 `sudo` 를 붙인다.
- `cpp` : BCT `.dts` 전처리는 GNU cpp여야 한다. 출력이 `cpp (Ubuntu 13.x…)` 형태면 된다.
- eMMC 플래시(`doflash.sh`)는 USB 대용량 저장장치를 쓰지 않는다. 그래서 GNOME 자동마운트 해제나 `disk` 그룹은 이 단계에서는 필요 없다. NVMe 단계에서 다시 다룬다.

---

## 6. D315AO 연결

### 6.1 커넥터 (출처: `AVerMedia/D315AO/user_manual/AVerMedia_UM_D315 V1.8` 2.8 / 2.24 / 2.25 / 2.26장, 3.0장)

| 용도 | D315AO 위치 | 호스트 쪽 | 비고 |
|------|-------------|-----------|------|
| 플래시(리커버리) | **J8** USB 2.0 Micro-B ("BSP Installation as recovery mode") | 호스트 USB 포트에 **직결**(허브 금지) | 데이터 전송이 되는 고품질 케이블을 쓴다 |
| 시리얼 콘솔 | **JP8** Debug port, 1×4핀 2.54 mm: 1 = +3.3V, 2 = RX, 3 = TX, 4 = GND | **3.3V USB-TTL 변환기**(FTDI/CP2102 등) → `/dev/ttyUSB0` | 개발킷처럼 USB가 내장된 디버그 포트가 아니다 |
| 전원 버튼 | **BSW1** | — | |
| 리커버리 버튼 | **BSW3** | — | |
| 전원 모드 | **SW1-1** DIP: OFF = Auto Power, ON = Button Power | — | Auto Power면 DC를 연결하는 순간 켜진다 |

JP8 배선은 다음과 같다. **+3.3V(1번 핀)은 연결하지 않는다.**

| D315AO JP8 | USB-TTL 변환기 |
|------------|----------------|
| 2 RX | TX |
| 3 TX | RX |
| 4 GND | GND |

**왜?**
- 개발킷 MACHINE 이미지는 커널 DTB가 개발킷용이다. 그래서 D315AO의 HDMI/DP 출력이나 USB 키보드가 동작한다는 보장이 없다. 디스플레이 설정 데이터(DCB)와 USB 포트 매핑이 다르기 때문이다(`D315AO_CUSTOM_LAYER_NECESSITY.md`). **부팅 성공 여부를 확실히 볼 수 있는 곳은 시리얼 콘솔뿐이다.**
- RX/TX는 교차 연결한다. +3.3V를 연결하지 않는 이유는 변환기와 보드 전원이 서로 부딪히는 것을 피하기 위해서다.
- 부트로더(MB1/MB2/UEFI) 로그는 TCU(`ttyTCU0`, 115200 8N1)로 나온다. JP8이 이 TCU 포트일 것으로 보지만 **D315AO 회로도로 확인하지는 않았다.** 아무 출력이 없으면 9장을 참고한다.

### 6.2 첫 부팅 전 하드웨어 상태

- M.2 SSD, mPCIe/LTE, 카메라·SerDes 보드, CAN 배선, 40핀 헤더 장치는 **분리한 상태**로 진행할 것을 권장한다.
- 이더넷 케이블과 JP8 시리얼 연결만 남긴다.

**왜?** 개발킷 pinmux는 D315가 부팅 초기에 High/Low로 구동하는 GPIO 핀 10여 개(AG1, E0, E1, A6, K0, K1, X5, EE1 등)를 다르게 둔다(`D315AO_CUSTOM_LAYER_NECESSITY.md`). 이 핀들의 연결처는 회로도가 없어 모른다. 그래서 이 단계는 **모듈 자원(eMMC, UART)만으로 부팅되는지**를 보는 데 집중하고, 주변 장치는 이후 단계에서 붙인다.

### 6.3 시리얼 콘솔 열기

```bash
host$ ls /dev/ttyUSB*
host$ picocom -b 115200 /dev/ttyUSB0 --logfile $YROOT/flash/d315ao-step1-console.log
# 종료: Ctrl-A Ctrl-X
```

**왜?** 플래시 직후 첫 부팅 로그를 처음부터 받기 위해 **플래시 전에** 열어 둔다. `--logfile` 로 남긴 로그는 2단계(차이 기록)의 기준 자료가 된다.
`$YROOT/flash` 가 없으면 먼저 `mkdir -p $YROOT/flash` 를 실행한다.

### 6.4 리커버리 모드 진입

1. DC 전원을 분리한다.
2. **BSW3(Recovery)** 를 누른 상태로 DC 전원을 연결한다. SW1-1이 Button Power(ON)면, 전원을 연결한 뒤 BSW3를 누른 채 **BSW1(Power)** 을 누른다.
3. 2초 후 손을 뗀다.

```bash
host$ lsusb -d 0955:
# 예) Bus 001 Device 012: ID 0955:7023 NVIDIA Corp. APX
host$ sudo dmesg | tail -5      # Product: APX / Manufacturer: NVIDIA Corp
```

**왜?** 리커버리(RCM) 모드는 BootROM이 USB로 부트로더를 받으려고 기다리는 상태다. `0955:xxxx` 장치가 보여야 플래시 도구가 보드와 통신할 수 있다(사용자 매뉴얼 3.1장의 확인 방법과 같다). 이 상태에서는 시리얼 콘솔에 아무것도 나오지 않는 것이 정상이다.

---

## 7. eMMC 플래시 (호스트)

> ⚠️ 이 플래시는 D315AO의 **QSPI 부트로더와 eMMC 전체를 덮어쓴다.** 현재 설치된 AVerMedia BSP(Ubuntu)와 그 안의 데이터는 지워진다.
> 필요한 파일은 먼저 백업한다. 원래 상태로 되돌리는 방법은 10장에 있다.

### 7.1 패키지 풀기

```bash
host$ D=$YROOT/build-docker-agx-orin-emmc/tmp/deploy/images/p3737-0000-p3701-0005
host$ ls -lh $D/core-image-base-p3737-0000-p3701-0005.rootfs.tegraflash.tar.gz
host$ mkdir -p $YROOT/flash/d315ao-step1-emmc && cd $YROOT/flash/d315ao-step1-emmc
host$ ls -A
host$ tar -xf $D/core-image-base-p3737-0000-p3701-0005.rootfs.tegraflash.tar.gz
host$ ls doflash.sh tegra-flash-helper.sh tegrarcm_v2 tegraflash.py
```

**왜?**
- 호스트 경로(`/home/louis/...`)로 컨테이너가 만든 파일을 그대로 읽는다(1.3). 복사할 필요가 없다.
- `ls -A` 결과가 **비어 있어야** 한다. 이전 시도에서 생긴 `signed/` 등과 섞이면 다른 버전의 바이너리가 기록될 수 있다. 비어 있지 않으면 내용을 확인한 뒤 디렉터리를 비우거나 새 디렉터리를 쓴다.
- `tar` 는 CLI로 푼다(GUI 압축 도구는 sparse 파일을 깨뜨린다는 보고가 있다).
- 마지막 `ls` 로 플래시 도구가 패키지 안에 들어 있는지 확인한다. 호스트에 Yocto가 없어도 플래시할 수 있는 이유가 이것이다.

### 7.2 사전 점검 → 플래시

```bash
host$ ./tegrarcm_v2 --help >/dev/null 2>&1; echo "tegrarcm_v2 exit=$?"
host$ python3 tegraflash.py --help >/dev/null 2>&1; echo "tegraflash.py exit=$?"
host$ lsusb -d 0955:
host$ sudo ./doflash.sh 2>&1 | tee doflash.log
```

**왜?**
- 처음 두 줄은 빌드 환경(22.04)과 다른 호스트(24.04)에서 플래시 도구가 실행되는지 **보드를 건드리기 전에** 확인한다. 둘 다 `exit=0` 이어야 한다. 아니면 `--help` 를 직접 실행해서 오류 메시지를 확인한다.
- `lsusb` : 6.4 이후 시간이 지나 리커버리 모드가 풀리지 않았는지 다시 확인한다.
- `doflash.sh` 가 하는 일은 다음 순서다(10~20분).
  1. `tegra-flash-helper.sh` 를 호출한다.
  2. RCM으로 보드에 접속해 **모듈 EEPROM**에서 BOARDID(3701)와 SKU를 읽고 MACHINE과 비교한다.
  3. BCT와 DTB를 고르고 서명한다.
  4. **QSPI(부트로더)와 eMMC(GPT, esp, APP=rootfs …)** 에 기록하고 재부팅한다.
- 캐리어 EEPROM은 읽지 않는다(0.3). 그래서 개발킷이 아닌 D315AO 캐리어에서도 이 확인 단계는 통과해야 정상이다.
- `tee` : 실패했을 때 원인을 볼 수 있게 로그를 남긴다.
- `sudo` : 5장에서 그룹 설정을 하고 다시 로그인했다면 생략할 수 있다.

---

## 8. 부팅 검증 (D315AO 시리얼 콘솔)

### 8.1 콘솔에서 볼 것

플래시가 끝나면 보드가 자동으로 재부팅된다. 6.3에서 연 콘솔에 다음 순서로 로그가 나와야 한다.

1. MB1/MB2 로그
2. UEFI 로그(`Jetson UEFI firmware`)
3. 커널 로그
4. `p3737-0000-p3701-0005 login:`

`root` 로 로그인한다(암호 없음).

### 8.2 통과 기준 확인

| 명령 | 기대 결과 / 의미 |
|------|------------------|
| `d315# cat /etc/nv_tegra_release` | `# R36 (release), REVISION: 5.2` — meta-tegra 기준 BSP가 올라갔다 |
| `d315# uname -a` | 빌드 시각이 4.2의 빌드 시각과 맞는다(다른 이미지가 올라가지 않았다) |
| `d315# findmnt /` | SOURCE **`/dev/mmcblk0p1`** ← **이 단계의 핵심** |
| `d315# lsblk -o NAME,SIZE,PARTLABEL,MOUNTPOINT` | `mmcblk0` 약 29 GB에 `APP`, `esp` 등의 파티션이 있다 |
| `d315# systemctl --failed` | 실패한 서비스 목록. 0개가 이상적이지만 8.3의 "예상 차이" 때문에 실패가 있을 수 있다. 목록을 기록해 둔다 |
| `d315# lsmod \| grep -E 'nvgpu\|nvmap'` | GPU 핵심 모듈이 로드됐다 |
| `d315# reboot` 후 다시 로그인, `findmnt /` | 재부팅해도 eMMC로 부팅된다 |

**왜 이 항목들인가?** 이 단계의 목표는 "D315AO에서 meta-tegra 이미지가 eMMC로 부팅된다"는 것 하나다. 나머지 주변장치 동작은 판정 기준이 아니다.

### 8.3 예상되는 차이 (실패가 아님)

개발킷 DTB와 개발킷 pinmux를 썼기 때문에 다음은 **정상적으로 예상되는 차이**다. 원인은 `D315AO_CUSTOM_LAYER_NECESSITY.md` 의 DTB 비교 결과에서 나온 것이다.

| 증상 | 원인 |
|------|------|
| `cat /proc/device-tree/model` 이 `NVIDIA Jetson AGX Orin Developer Kit` 로 나온다 | 개발킷 DTB를 썼다. D315 DTB의 model 문자열과 다르다 |
| Type-C 컨트롤러(`cypd4226`, i2c) probe 에러가 dmesg에 나온다 | D315에는 이 칩이 없는데 개발킷 DTB에는 있다 |
| 일부 USB 포트가 동작하지 않거나 USB3 장치가 USB2로 붙는다 | USB2/USB3 포트 짝 배정(companion)과 usb2-0 모드(otg/peripheral)가 다르다 |
| 오디오 코덱(`rt5640`) probe 에러, 사운드카드가 없다 | D315는 TLV320AIC3101 코덱을 쓴다 |
| HDMI/DP에 화면이 나오지 않는다 | 디스플레이 출력 구성 정보(DCB)가 다르다 |
| CAN, 40핀 일부 기능이 동작하지 않는다 | MB1 pinmux가 개발킷 기준이다 |

**모르는 것 — 관찰해서 기록할 것**
- 이더넷이 링크업되는지 (`ip -br link`). MGBE 노드는 두 DTB가 같지만, 실제로 어떤 포트가 연결돼 있는지는 확인하지 않았다.
- 6.2에서 말한 GPIO 핀들이 개발킷 설정에서 어떤 상태가 되는지.

### 8.4 2단계로 넘길 로그 남기기

```bash
d315# mkdir -p /root/step1 && cd /root/step1
d315# dmesg > dmesg.txt; lsusb -t > lsusb-t.txt; lspci -vv > lspci.txt 2>&1
d315# systemctl --failed > failed.txt; cat /sys/kernel/debug/gpio > gpio.txt 2>&1
d315# cat /proc/device-tree/model > model.txt; ip -br link > link.txt
```

이더넷이 연결됐다면 호스트로 가져온다. 안 되면 콘솔 로그(6.3 `--logfile`)에서 출력을 복사한다.

```bash
host$ scp -r root@<D315AO_IP>:/root/step1 $YROOT/flash/d315ao-step1-logs
```

**왜?** 권장 순서 2단계("위 차이가 실제로 어떻게 나타나는지 기록")의 입력 자료다. 특히 `gpio.txt` 는 3단계에서 D315 pinmux/GPIO를 넣은 이미지와 비교할 기준값이 된다.

---

## 9. 트러블슈팅

| 증상 | 원인 / 조치 |
|------|-------------|
| `Please use a locale setting which supports UTF-8` (컨테이너) | 2.2의 `export LANG=... LC_ALL=...` 를 빠뜨렸다. 셸마다 다시 실행한다. |
| `ERROR: ... is not a directory` / layer 경로 오류 | 호스트에서 bitbake를 실행했다. 이 빌드 디렉터리는 컨테이너 경로로 설정돼 있다(1.3). |
| `No new tasks can be executed since the disk space monitor action is "STOPTASKS"` | 디스크 부족(1.2). 공간을 확보한 뒤 같은 명령을 다시 실행하면 이어서 진행한다. |
| 컴파일 중 `Killed` / `c++: fatal error: Killed signal` | 메모리 부족(OOM). 3.3의 스레드 수를 8로 낮춘다. |
| fetch 실패(`Fetcher failure`) | 네트워크/프록시 문제. 컨테이너에서 `curl -I https://developer.nvidia.com` 로 확인한다. 회사 프록시가 필요하면 컨테이너 셸에서 `export http_proxy=... https_proxy=...` 를 설정한다. |
| `lsusb -d 0955:` 에 아무것도 없다 | J8 케이블이 충전 전용이거나, 허브를 거쳤거나, 리커버리 순서가 틀렸다(6.4). DC를 완전히 분리한 뒤 다시 시도한다. |
| `ERR: actual board SKU XXXX does not match expected board SKU 0005` | 64GB 모듈이 아니다. 1.4 표대로 MACHINE을 바꾸고 다시 빌드한다(sstate 덕분에 대부분 재사용된다). |
| `ERR: actual board ID ... does not match` | AGX Orin(3701) 모듈이 아니다. 모듈을 확인한다. |
| 플래시 중 `tegrarcm_v2` 타임아웃 / USB 오류 | 호스트 USB autosuspend 때문일 수 있다. 다른 USB 포트(가능하면 메인보드 후면)를 쓴다. `tlp` 가 있으면 제거를 검토한다. |
| 플래시는 성공했는데 시리얼 출력이 전혀 없다 | ① RX/TX 교차 연결 확인 ② 115200 8N1, 흐름제어 off ③ 변환기가 3.3V 레벨인지 확인 ④ JP8이 TCU가 아닐 수 있다 → AVerMedia BSP를 다시 올려 같은 배선으로 콘솔이 나오는지 비교한다(10장). 출력이 나오면 배선 문제가 아니라 이미지 문제다. |
| UEFI까지 나오고 커널에서 멈추거나 재부팅이 반복된다 | 로그를 보존하고 MB1/UEFI 마지막 메시지를 기록한다. 개발킷 DTB/pinmux가 D315AO와 맞지 않는 경우일 수 있다. 이 경우 1단계 결론은 "개발킷 MACHINE으로는 부팅 불가"이며, 커스텀 레이어(3단계)를 먼저 해야 한다. |
| 커널은 올라오는데 rootfs를 못 찾는다(`Waiting for root device`) | `TNSPEC_BOOTDEV`(3.4)를 확인한다. `mmcblk0p1` 이 아니면 NVMe용 설정이 섞인 것이다. |

---

## 10. 원래 AVerMedia BSP로 되돌리기

```bash
host$ cd ~/work/project/jeisys-medical/aicamsys-medspa-js/sdk/AVerMedia/JetPack_6.2.2_Linux_JETSON_desktop_in_docker/Linux_for_Tegra
host$ ls install.sh jetson-agx-orin-d315ao.conf
```

D315AO를 리커버리 모드로 넣고(6.4), 이 디렉터리에서 기존에 쓰던 AVerMedia 절차(`install.sh`, 사용자 매뉴얼 3.1장)대로 다시 플래시한다.

**왜?**
- meta-tegra 플래시는 QSPI 부트로더까지 바꾼다. 그래서 eMMC만 복원하는 것으로는 부족하고, AVerMedia BSP 전체를 다시 플래시해야 원래 상태가 된다.
- 이 BSP 디렉터리가 로컬에 있다는 것이 1단계를 시도해도 되는 안전망이다.
- `jetson-agx-orin-d315ao.conf` 의 동작은 `../sdk-analysis/D315AO_BOARD_CONF_ANALYSIS.md` 참고.

---

## 부록 A. 명령 요약

```bash
# ---------- 호스트: 준비 ----------
host$ cd ~/work/project/jeisys-medical/aicamsys-medspa-js/sdk/AVerMedia/Yocto/scarthgap && export YROOT=$PWD
host$ df -h $YROOT                                   # 여유 100 GB 이상 확보
host$ tmux new -s yocto-d315ao
host$ docker exec -it prj-aicamsys-medspa-js-22.04 bash

# ---------- 컨테이너: 빌드 ----------
ctr$ export LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
ctr$ cd /home/1000/work/project/jeisys-medical/aicamsys-medspa-js/sdk/AVerMedia/Yocto/scarthgap && export YROOT=$PWD
ctr$ source poky/oe-init-build-env build-docker-agx-orin-emmc
ctr$ grep -n 'D315AO step1' conf/local.conf || cat >> conf/local.conf <<'EOF'   # 3.3의 블록
...
EOF
ctr$ bitbake-getvar -r core-image-base --value TNSPEC_BOOTDEV        # mmcblk0p1
ctr$ bitbake core-image-base --runall=fetch && bitbake core-image-base

# ---------- 호스트: 플래시 ----------
host$ picocom -b 115200 /dev/ttyUSB0 --logfile $YROOT/flash/d315ao-step1-console.log   # 별도 창
#   BSW3 누른 채 전원 → lsusb -d 0955:
host$ mkdir -p $YROOT/flash/d315ao-step1-emmc && cd $_
host$ tar -xf $YROOT/build-docker-agx-orin-emmc/tmp/deploy/images/p3737-0000-p3701-0005/core-image-base-p3737-0000-p3701-0005.rootfs.tegraflash.tar.gz
host$ sudo ./doflash.sh 2>&1 | tee doflash.log

# ---------- 보드: 검증 ----------
d315# findmnt /          # /dev/mmcblk0p1
d315# cat /etc/nv_tegra_release; systemctl --failed
```

## 부록 B. 관련 문서

| 문서 | 관계 |
|------|------|
| [`../sdk-analysis/D315AO_CUSTOM_LAYER_NECESSITY.md`](../sdk-analysis/D315AO_CUSTOM_LAYER_NECESSITY.md) | 이 단계의 근거, 개발킷 MACHINE과 D315AO의 차이, 2~4단계 |
| [`../sdk-analysis/D315AO_BOARD_CONF_ANALYSIS.md`](../sdk-analysis/D315AO_BOARD_CONF_ANALYSIS.md) | AVerMedia `jetson-agx-orin-d315ao.conf` 분석 |
| [`../sdk-analysis/YOCTO_DOCKER_BUILD_HOST_FLASH_GUIDE.md`](../sdk-analysis/YOCTO_DOCKER_BUILD_HOST_FLASH_GUIDE.md) | 개발킷 보드 기준 Docker 빌드 → 호스트 플래시 일반 가이드(새 컨테이너 `yocto-scarthgap:22.04`, NVMe 포함) |
