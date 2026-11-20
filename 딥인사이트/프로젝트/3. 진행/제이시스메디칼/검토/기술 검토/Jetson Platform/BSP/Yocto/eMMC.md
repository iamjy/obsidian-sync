---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
# REPORT-001 — D315AO 1단계: 개발킷 MACHINE eMMC 부팅 결과

| 항목 | 내용 |
|------|------|
| 일자 | 2026-11-19 |
| 가이드 | [`../sdk-guide/D315AO_EMMC_BOOT_WITH_DEVKIT_MACHINE.md`](../sdk-guide/D315AO_EMMC_BOOT_WITH_DEVKIT_MACHINE.md) **8.2 통과 기준** |
| 보드 | AVerMedia D315AO + Jetson AGX Orin **32GB (P3701 SKU 0004, FAB 500, REV L.0)** |
| 빌드 | Docker 22.04 컨테이너, `MACHINE = p3737-0000-p3701-0004`, `core-image-base`, 빌드 디렉터리 `build-docker-agx-orin-emmc` |
| 이미지 | `core-image-base-p3737-0000-p3701-0004.rootfs-20261119014247.tegraflash.tar.gz` |
| 결론 | ✅ **1단계 목표 달성.** D315AO가 meta-tegra 이미지로 eMMC(`/dev/mmcblk0p1`)에서 부팅된다. ⚠ 실패 서비스 2개가 있다(4장). ⏳ 재부팅 확인은 아직 하지 않았다 |

---

## 1. 판정 요약 (가이드 8.2)

| # | 확인 항목 | 결과 | 판정 |
|---|-----------|------|------|
| 1 | `cat /etc/nv_tegra_release` | `# R36 (release), REVISION: 5.2, GCID: 46426093, BOARD: generic` | ✅ |
| 2 | `uname -a` | `5.15.199-l4t-r36.5-1050.50+ge6b364044e43 ... Tue May 26 22:57:38 UTC 2026` | ✅ (2.2 참고) |
| 3 | `findmnt /` | `/dev/mmcblk0p1 ext4 rw,relatime` | ✅ **핵심 기준 통과** |
| 4 | `lsblk` | `mmcblk0` 59.3G, 파티션 15개가 `flash.xml` 과 일치 | ✅ (2.4 참고) |
| 5 | `systemctl --failed` | `nvpmodel.service`, `systemd-networkd-wait-online.service` 실패 | ⚠ 판정 기준은 아니지만 기록함 (4장) |
| 6 | `lsmod \| grep -E 'nvgpu\|nvmap'` | `nvgpu`, `nvmap`, `host1x` 로드됨 | ✅ |
| 7 | `reboot` 후 `findmnt /` | 결과를 받지 않았다 | ⏳ 미확인 |

가이드 0.2의 통과 기준 1~4(빌드 성공, `Flashing completed`, root 로그인, `findmnt /` = `/dev/mmcblk0p1`)는 모두 충족했다.

---

## 2. 항목별 결과와 해석

### 2.1 BSP 버전

```
root@p3737-0000-p3701-0004:~# cat /etc/nv_tegra_release
# R36 (release), REVISION: 5.2, GCID: 46426093, BOARD: generic, EABI: aarch64, DATE: Thu Jul 16 18:56:22 UTC 2026
```

- meta-tegra가 쓰는 L4T **R36.5.2** 와 같다(`downloads/Jetson_Linux_R36.5.2_aarch64.tbz2`).
- 프롬프트의 호스트명 `p3737-0000-p3701-0004` 은 Yocto 기본 호스트명(`= MACHINE`)이다. AVerMedia BSP(`tegra-ubuntu`)가 아니라 이번에 빌드한 이미지로 부팅됐다는 뜻이다.

### 2.2 커널

```
Linux p3737-0000-p3701-0004 5.15.199-l4t-r36.5-1050.50+ge6b364044e43 #1 SMP PREEMPT Tue May 26 22:57:38 UTC 2026 aarch64 GNU/Linux
```

빌드한 커널이 맞는지 확인한 근거:

| 보드 값 | 빌드 쪽 값 | 일치 |
|---------|------------|------|
| 버전 `5.15.199`, git `e6b364044e` | `$DEPLOY/Image--5.15.199+git0+e6b364044e-r0-p3737-0000-p3701-0004-20261119014247.bin` | ✅ |
| 빌드 시각 `Tue May 26 22:57:38 UTC 2026` | `tmp/work/.../linux-jammy-nvidia-tegra/5.15.199+git/source-date-epoch/__source_date_epoch.txt` = `1779836258` = 2026-05-26 22:57:38 UTC | ✅ |

**가이드 기준을 고쳤다.** 가이드 8.2는 "빌드 시각이 4.2의 빌드 시각과 맞는다"고 했지만 이 기준은 틀렸다. Yocto는 재현 가능 빌드(reproducible build)를 위해 `uname` 의 시각을 `SOURCE_DATE_EPOCH`(커널 소스의 마지막 커밋 시각)로 고정한다. 그래서 실제 빌드 날짜(2026-11-19)가 아니라 소스 커밋 날짜(2026-05-26)가 나온다. 가이드는 git 해시와 `SOURCE_DATE_EPOCH` 로 비교하도록 수정했다.

### 2.3 rootfs 위치 — 핵심 기준

```
/ /dev/mmcblk0p1 ext4 rw,relatime
```

`APP`(rootfs)이 eMMC의 첫 번째 파티션 번호로 마운트됐다(`flash.xml` 의 `APP id="1"`, 가이드 7.3.3). **1단계의 목표인 "개발킷 MACHINE으로 D315AO eMMC 부팅"이 확인됐다.**

### 2.4 파티션

| 장치 | 크기 | PARTLABEL | 가이드 7.3.3 대비 |
|------|------|-----------|-------------------|
| `mmcblk0` | **59.3G** | — | 실제 eMMC 용량. 64 GB eMMC다(가이드에서 "미확인"이던 항목) |
| `mmcblk0p1` | **57.8G** | APP (`/`) | ⚠ 가이드에는 28 GiB로 적었다. 아래 설명 참고 |
| `p2` / `p5` | 128M | A/B_kernel | ✅ |
| `p3` / `p6` | 768K | A/B_kernel-dtb | ✅ |
| `p4` / `p7` | 31.6M | A/B_reserved_on_user | ✅ |
| `p8` / `p11` | 80M | recovery / recovery_alt | ✅ |
| `p9` / `p12` | 512K | recovery-dtb / recovery-dtb_alt | ✅ |
| `p10` | 64M | esp (`/boot/efi`) | ✅ ESP가 마운트됐다 |
| `p13` | 64M | esp_alt | ✅ |
| `p14` | 400M | UDA | ✅ |
| `p15` | 479.5M | reserved | ✅ |
| `mmcblk0boot0/1` | 31.5M ×2 | — | eMMC 하드웨어 boot 영역. 이 레이아웃은 쓰지 않는다(부트로더는 QSPI에 있다) |

**`APP` 크기가 다른 이유.** `flash.xml` 의 `APP` 은 `<size> 30064771072 </size>`(28 GiB)이다. 하지만 `<allocation_attribute> 0x808 </allocation_attribute>` 의 `0x800` 비트가 "남은 공간을 모두 사용"이라는 뜻이다. 그래서 플래시 도구가 다른 파티션을 배치하고 남은 공간 전체를 `APP` 에 줬다. 59.3G − 다른 파티션 합계(약 1.5G) ≈ 57.8G로 계산이 맞는다. 가이드 7.3.3과 8.2의 "약 28 GiB"는 틀린 값이라 고쳤다.

**아직 모르는 것 — 파일시스템 크기.** 파티션은 57.8G지만, 그 안의 ext4는 빌드 때 만든 크기 그대로일 수 있다. 빌드한 ext4 이미지는 `Block count 7340032 × 4096` = **28 GiB** 이다. 이미지에 `systemd-growfs-root.service` 가 들어 있지만, 실제로 확장됐는지는 보드에서 확인해야 한다.

```bash
d315# df -h /        # Size 가 약 28G 이면 확장 안 됨, 약 57G 이면 확장됨
```

### 2.5 GPU 모듈

```
nvgpu   2809856  0
nvmap    241664  1 nvgpu
host1x   208896  7 host1x_nvhost,host1x_fence,tegra_se,nvgpu,tegra_drm,nvhost_nvdla,nvhost_pva
```

`nvgpu`, `nvmap` 이 로드됐다. `host1x` 를 쓰는 모듈로 `tegra_drm`, `nvhost_nvdla`(DLA), `nvhost_pva`(PVA), `tegra_se` 가 보인다. OOT(out-of-tree) 커널 모듈(`nvidia-kernel-oot`)이 정상적으로 설치되고 로드된 것이다.

---

## 3. 1단계 결론

| 질문 | 답 |
|------|----|
| Docker 22.04 컨테이너에서 meta-tegra 이미지가 빌드되는가 | ✅ 된다 |
| 24.04 호스트에서 D315AO에 플래시할 수 있는가 | ✅ 된다 (`Flashing completed`) |
| 커스텀 레이어 없이 D315AO가 eMMC로 부팅되는가 | ✅ 된다 (`/dev/mmcblk0p1`) |
| 모듈 SKU | **0004 (32GB, 8코어)**. 앞으로 MACHINE은 `p3737-0000-p3701-0004` 를 기준으로 한다 |

`D315AO_CUSTOM_LAYER_NECESSITY.md` 의 분석("부팅 체인은 개발킷과 같고, 다른 것은 MB1 pinmux/GPIO와 커널 DTB")과 맞는 결과다. **eMMC 부팅까지는 커스텀 레이어 없이 된다.**

---

## 4. 실패한 서비스

| 서비스 | 판단 | 조치 |
|--------|------|------|
| `nvpmodel.service` | ❌ **이미지 설정 문제(추정 원인 있음).** rootfs의 `/etc/nvpmodel.conf` 가 **64GB(12코어)용 `nvpmodel_p3701_0000.conf`** 이다. 32GB 모듈(SKU 0004)은 CPU가 8코어(cpu0~7)인데, 이 설정은 `cpu8~11` 의 sysfs를 건드린다 | [ISSUE-002](../sdk-debugging/ISSUE-002_nvpmodel_wrong_conf_sku0004.md)에 기록했다. 보드 로그 확인 대기 |
| `systemd-networkd-wait-online.service` | ⚠ **환경 요인일 가능성이 높다.** 부팅 중 네트워크 링크가 올라오지 않으면 타임아웃으로 실패한다. 이더넷 케이블 연결 여부를 받지 않았다 | 아래 명령으로 확인한다. 케이블 없이 실패한 것이면 1단계에서는 무시해도 된다 |

```bash
d315# journalctl -b -u nvpmodel --no-pager | tail -20     # ISSUE-002 원인 확정용
d315# nproc; cat /sys/devices/system/cpu/possible          # 8 / 0-7 이면 8코어 모듈
d315# ip -br link; networkctl                              # 링크 상태
d315# journalctl -b -u systemd-networkd-wait-online --no-pager | tail -5
```

---

## 5. 남은 일

| 항목 | 가이드 위치 | 상태 |
|------|-------------|------|
| `reboot` 후 다시 `findmnt /` | 8.2 마지막 줄 | ⏳ |
| `df -h /` (rootfs 확장 여부) | 이 보고서 2.4 | ⏳ |
| `nvpmodel` 로그 · `nproc` | 이 보고서 4장, ISSUE-002 | ⏳ |
| 이더넷 링크 상태 | 8.3 "모르는 것" | ⏳ |
| 2단계 입력 로그 수집(`dmesg`, `lsusb -t`, `gpio` 등) | 8.4 | ⏳ |

---

## 6. 이번 결과로 바뀐 가이드 내용

| 위치 | 변경 |
|------|------|
| 7.3.3 `APP` 행과 표 아래 설명 | 28 GiB → "`<size>` 는 28 GiB지만 `0x808` 때문에 남은 공간 전체(이 보드 57.8G)". ext4는 28 GiB로 만들어진다는 점 추가 |
| 8.2 `uname -a` 행 | 빌드 시각 비교 → git 해시와 `SOURCE_DATE_EPOCH` 비교 |
| 8.2 `lsblk` 행 | `APP` 기대값을 "eMMC 남은 공간 전체"로 수정, `df -h /` 확인 추가 |
| 9장 | `nvpmodel.service` 실패 행 추가(ISSUE-002 링크) |

---

## 부록 — 원본 출력

<details>
<summary>보드 콘솔 출력 전체 (2026-11-19)</summary>

```
root@p3737-0000-p3701-0004:~# cat /etc/nv_tegra_release
# R36 (release), REVISION: 5.2, GCID: 46426093, BOARD: generic, EABI: aarch64, DATE: Thu Jul 16 18:56:22 UTC 2026
root@p3737-0000-p3701-0004:~# uname -a
Linux p3737-0000-p3701-0004 5.15.199-l4t-r36.5-1050.50+ge6b364044e43 #1 SMP PREEMPT Tue May 26 22:57:38 UTC 2026 aarch64 GNU/Linux
root@p3737-0000-p3701-0004:~# findmnt /
TARGET
  SOURCE         FSTYPE OPTIONS
/ /dev/mmcblk0p1 ext4   rw,relatime
root@p3737-0000-p3701-0004:~# lsblk -o NAME,SIZE,PARTLABEL,MOUNTPOINT
NAME           SIZE PARTLABEL          MOUNTPOIN
mmcblk0       59.3G
|-mmcblk0p1   57.8G APP                /
|-mmcblk0p2    128M A_kernel
|-mmcblk0p3    768K A_kernel-dtb
|-mmcblk0p4   31.6M A_reserved_on_user
|-mmcblk0p5    128M B_kernel
|-mmcblk0p6    768K B_kernel-dtb
|-mmcblk0p7   31.6M B_reserved_on_user
|-mmcblk0p8     80M recovery
|-mmcblk0p9    512K recovery-dtb
|-mmcblk0p10    64M esp                /boot/efi
|-mmcblk0p11    80M recovery_alt
|-mmcblk0p12   512K recovery-dtb_alt
|-mmcblk0p13    64M esp_alt
|-mmcblk0p14   400M UDA
`-mmcblk0p15 479.5M reserved
mmcblk0boot0  31.5M
mmcblk0boot1  31.5M
root@p3737-0000-p3701-0004:~# systemctl --failed
  UNIT                                 LOAD   ACTIVE SUB    DESCRIPTION
* nvpmodel.service                     loaded failed failed NVIDIA power model daemon
* systemd-networkd-wait-online.service loaded failed failed Wait for Network to be Configured

Legend: LOAD   -> Reflects whether the unit definition was properly loaded.
        ACTIVE -> The high-level unit activation state, i.e. generalization of SUB.
        SUB    -> The low-level unit activation state, values depend on unit type.

2 loaded units listed.
root@p3737-0000-p3701-0004:~# lsmod | grep -E 'nvgpu|nvmap'
nvgpu                2809856  0
nvmap                 241664  1 nvgpu
nvsciipc               24576  1 nvmap
host1x                208896  7 host1x_nvhost,host1x_fence,tegra_se,nvgpu,tegra_drm,nvhost_nvdla,nvhost_pva
mc_utils               16384  3 nvidia,nvgpu,tegra_camera_platform
```

</details>