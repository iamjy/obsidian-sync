---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---

# Jetson Orin Yocto 부트 아키텍처 분석

NVIDIA Jetson AGX Orin의 경우, Yocto 빌드 환경에서도 부트로더가 **UEFI로 완전히 전환**되었습니다. [1, 2]

### 📌 주요 변경 사항
- **기존 (Nano, Xavier 계열)**: U-Boot 또는 NVIDIA 자체 부트로더인 CBoot를 결합하여 사용.
- **현재 (Orin 아키텍처 - AGX Orin, Orin NX, Orin Nano)**: 공식 BSP(L4T) 레벨에서 U-Boot를 완전히 폐지하고, **UEFI (TianoCore EDK2 기반)**를 표준 부트로더로 채택하였습니다. [1, 3, 4, 5]
- **Yocto 적용**: Jetson 전용 Yocto 레이어인 [`meta-tegra`](https://github.com/OE4T/meta-tegra) 사용 시 UEFI 기반 부트 아키텍처가 강제 적용됩니다. [5, 6]

---

## 1. Jetson Orin의 Yocto 부트 흐름 (Boot Flow)

AGX Orin의 부팅 파이프라인은 하드웨어 단계를 거쳐 곧바로 UEFI로 진입합니다. [7, 8]

**부팅 시퀀스:**
`BootROM` $\rightarrow$ `PSCROM` $\rightarrow$ `MB1` $\rightarrow$ `MB2` $\rightarrow$ `UEFI (EDK2)` $\rightarrow$ `Linux Kernel`

- **빌드 과정**: Yocto 빌드 시 `edk2-firmware-tegra` 패키지가 자동 빌드되어 UEFI 바이너리(`uefi_jetson.bin`)를 생성하며, 이는 QSPI 플래시 메모리에 기록됩니다. [7, 8, 9, 10]

## 2. OS 및 커널 로드 방식 (L4TLauncher)

Jetson Orin의 UEFI는 NVIDIA 표준 **L4TLauncher** 구성을 따릅니다. [11]

- **로드 메커니즘**: UEFI 내부의 BDS(Boot Device Selection) 단계에서 `/boot/extlinux/extlinux.conf` 파일을 읽어 커널, 디바이스 트리(DTB), 커널 아규먼트를 로드합니다.
- **특징**: 별도의 3rd party 부트로더(GRUB 등) 없이 UEFI 자체 기능만으로 Yocto 리눅스 커널 부팅이 가능합니다. [5, 11]

## 3. Secure Boot 및 다중 부팅 (GRUB / GRUB-EFI)

엔터프라이즈 레벨의 보안 요구사항이나 A/B 시스템 업데이트(듀얼 부팅) 체계가 필요한 경우, `UEFI + GRUB-EFI` 조합을 수동으로 결합할 수 있습니다. [5]

- **구현 방법**: 기본 UEFI 상단에 `grub-efi`를 페이로드로 추가하여 완전한 UEFI 규격의 보안 부팅 체인을 완성합니다. [5]

---

## 💡 요약 및 가이드

| 항목 | 내용 |
| :--- | :--- |
| **U-Boot 지원 여부** | Jetson Orin 계열에서는 Yocto 사용 시에도 **사용 불가** |
| **플래싱 유의사항** | 빌드 결과물인 `ESP(EFI System Partition)` 이미지를 생성한 후, NVIDIA 툴(`flash.sh` 또는 `initrd-flash`)을 사용하여 QSPI 영역과 메인 저장장치(NVMe 등)에 함께 기록해야 함 [1, 2, 5, 12, 13] |

> **추가 안내**: 사용 중이신 `meta-tegra` 레이어의 브랜치(mickledore, scarthgap, wrynose 등)나 Secure Boot/A/B 뱅크 이중화 업데이트 같은 특수 요구사항을 알려주시면 구체적인 `local.conf` 설정법을 안내해 드리겠습니다. [6, 14]

  
[1] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/yocto-on-jetson-orin-nano-orin-agx-u-boot-support-and-nvidia-s-shift-to-uefi/337919)
<!-- ... (이후 [2]~[14]까지의 레퍼런스 리스트는 원문과 동일하게 유지) ... -->
--- -
네, NVIDIA Jetson AGX Orin의 경우 Yocto 빌드 환경에서도 부트로더가 UEFI로 완전히 전환되었습니다. [1, 2]

기존 Jetson Nano나 Xavier 계열에서는 주로 U-Boot나 NVIDIA 자체 부트로더인 CBoot를 결합해 사용했으나, Orin 아키텍처(Jetson AGX Orin, Orin NX, Orin Nano)부터는 NVIDIA가 공식 BSP(L4T) 레벨에서 U-Boot를 완전히 폐지하고 UEFI(TianoCore EDK2 기반)를 표준 부트로더로 채택했습니다. [1, 3, 4, 5]

이에 따라 Jetson 전용 Yocto 오픈소스 레이어인 [meta-tegra](https://github.com/OE4T/meta-tegra)를 사용하여 최신 Yocto를 빌드할 때도 다음과 같이 UEFI 기반의 부트 아키텍처가 강제 적용됩니다. [5, 6]

## 1. Jetson Orin의 Yocto 부트 흐름 (Boot Flow)

AGX Orin의 부팅 파이프라인은 하드웨어 단계를 거쳐 곧바로 UEFI로 진입합니다. [7, 8]

- BootROM ➔ PSCROM ➔ MB1 ➔ MB2 ➔ UEFI (EDK2) ➔ Linux Kernel
- Yocto 빌드 시 `edk2-firmware-tegra` 패키지가 자동으로 함께 빌드되어 UEFI 바이너리(`uefi_jetson.bin`)를 생성하고, 이것이 QSPI 플래시 메모리에 기록됩니다. [7, 8, 9, 10]

## 2. OS 및 커널 로드 방식 (L4TLauncher)

기본적으로 Jetson Orin의 UEFI는 NVIDIA가 제공하는 표준 L4TLauncher 구성을 따릅니다. [11]

- UEFI 내부의 BDS(Boot Device Selection) 단계에서 `/boot/extlinux/extlinux.conf` 파일을 읽어 커널과 디바이스 트리(DTB), 커널 아규먼트를 로드하는 방식입니다.
- 따라서 별도의 3rd party 부트로더(GRUB 등)를 올리지 않아도 UEFI 자체 기능만으로 깔끔하게 Yocto 리눅스 커널로 부팅됩니다. [5, 11]

## 3. Yocto Secure Boot 및 다중 부팅 (GRUB / GRUB-EFI)

만약 단순 부팅을 넘어 엔터프라이즈 레벨의 UEFI Secure Boot나 완벽한 A/B 시스템 업데이트(듀얼 부팅) 체계를 굳건히 하고 싶다면, Yocto 설정을 통해 `UEFI + GRUB-EFI` 조합을 수동으로 결합하기도 합니다. [5]

- 기본 UEFI 상단에 `grub-efi`를 페이로드로 얹어서 완전한 UEFI 규격의 보안 부팅 체인을 완성할 수 있습니다. [5]

## 💡 요약 및 팁

- U-Boot 지원 여부: Jetson Orin 계열에서는 Yocto를 쓰더라도 더 이상 U-Boot를 사용할 수 없습니다.
- 플래싱 유의사항: Yocto 빌드가 완료되면 결과물로 `ESP(EFI System Partition)` 이미지가 생성됩니다. 처음 보드에 구동할 때는 NVIDIA 툴인 `flash.sh` 또는 `initrd-flash` 스크립트를 사용하여 보드의 QSPI(UEFI 펌웨어 영역)와 메인 저장장치(NVMe 등)를 함께 구워주어야 정상 작동합니다. [1, 2, 5, 12, 13]

현재 작업 중이신 **meta-tegra 레이어의 브랜치(예: mickledore, scarthgap, wrynose 등)**나, 구현하시려는 시스템에 **Secure Boot(보안 부팅)**나 A/B 뱅크 이중화 업데이트 같은 특수 요구사항이 있는지 알려주시면 추가로 구체적인 local.conf 설정법을 안내해 드리겠습니다. [6, 14]

  

[1] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/yocto-on-jetson-orin-nano-orin-agx-u-boot-support-and-nvidia-s-shift-to-uefi/337919)

[2] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/yocto-on-jetson-orin-nano-orin-agx-u-boot-support-and-nvidia-s-shift-to-uefi/337919)

[3] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/is-u-boot-available-in-orin/198783)

[4] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/what-does-the-cboot-of-jetson-agx-orin-means-how-does-uefi-replace-cboot/235661)

[5] [https://github.com](https://github.com/orgs/OE4T/discussions/2145)

[6] [https://developer.ridgerun.com](https://translate.google.com/translate?u=https://developer.ridgerun.com/wiki/index.php/Yocto_Support_for_NVIDIA_Jetson_Platforms_-_Setting_up_Yocto&hl=ko&sl=en&tl=ko&client=sge)

[7] [https://blog.bycode.kr](https://blog.bycode.kr/tech/edge-os/first-boot-yocto-on-jetson-orin/)

[8] [https://blog.bycode.kr](https://blog.bycode.kr/tech/edge-os/first-boot-yocto-on-jetson-orin/)

[9] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/flashing-uefi-uefi-jetson-bin-to-development-kit/281087)

[10] [https://github.com](https://github.com/OE4T/meta-tegra/issues/2138)

[11] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/archives/r36.5/DeveloperGuide/SD/Bootloader/UEFI.html)

[12] [https://proventusnova.com](https://proventusnova.com/blog/yocto-custom-image-flash-jetson-orin-nano-super)

[13] [https://www.jetson-ai-lab.com](https://www.jetson-ai-lab.com/tutorials/yocto-on-jetson/)

[14] [https://www.jetson-ai-lab.com](https://translate.google.com/translate?u=https://www.jetson-ai-lab.com/tutorials/yocto-on-jetson/&hl=ko&sl=en&tl=ko&client=sge)