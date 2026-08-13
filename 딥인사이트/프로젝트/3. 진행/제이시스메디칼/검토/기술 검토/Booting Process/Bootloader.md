---
title: Bootloader (UEFI & U-Boot)
source:
author:
published:
created: 2025-09-07
description: Jetson Orin 부팅 과정에서의 부트로더 - UEFI(EDK2)와 U-Boot 비교 정리
tags:
  - jetson
  - bootloader
  - uefi
  - u-boot
  - embedded
---

# Bootloader

임베디드 시스템(Jetson 등)에서 전원 인가 후 OS(리눅스 커널)를 메모리에 로드해 제어권을 넘겨주는 펌웨어 계층. ==Jetson Orin 세대(Jetpack 5/6)부터는 U-Boot가 폐지되고 **UEFI(EDK2)** 로 전환되었다.==

---

## 1. UEFI 란?

**UEFI(Unified Extensible Firmware Interface, 통일 확장 가능 펌웨어 인터페이스)** 는 컴퓨터가 전원을 켰을 때 운영체제(OS)가 부팅되기 전, 하드웨어를 초기화하고 OS를 메모리에 로드해 주는 최신 표준 펌웨어 인터페이스다.

과거 40년 가까이 사용되던 구형 **BIOS(Basic Input/Output System)** 의 기술적 한계를 극복하기 위해 인텔과 글로벌 IT 기업들이 만든 표준 규격이다.

### 1.1 BIOS vs UEFI 비교

| 구분 | 구형 BIOS (Legacy) | 최신 UEFI |
| --- | --- | --- |
| 디스크 방식 | MBR (최대 2TB 인식 제한) | GPT (최대 8.4ZB 인식 가능) |
| 부팅 속도 | 하드웨어 전수 점검으로 느림 | 빠른 부팅 (C 언어 기반 드라이버) |
| 보안 | 부팅 시 악성코드 검증 불가 | 보안 부팅 (Secure Boot) 지원 |
| 사용자 인터페이스 | 텍스트/키보드 기반 16비트 화면 | 그래픽 UI, 마우스/네트워크 지원 |

### 1.2 핵심 장점

- **대용량 스토리지 지원**: GPT(GUID Partition Table) 방식을 사용하여 2TB를 넘어서는 고용량 NVMe SSD나 HDD도 제약 없이 부팅 디바이스로 사용 가능.
- **보안 부팅 (Secure Boot)**: 위조되거나 악성코드가 포함된 부트로더가 실행되지 않도록 디지털 서명을 검증하는 보안 체계 제공.
- **네트워크 및 모듈성**: OS가 부팅되기 전 단계에서도 네트워크(PXE 부팅)를 통한 정비나 펌웨어 업데이트가 가능.

---

## 2. 임베디드 시스템(Jetson)에서의 UEFI

==PC 환경뿐만 아니라 NVIDIA Jetson Orin 시리즈(Jetpack 5/6) 등 최신 임베디드 리눅스 시스템에서도 UEFI가 표준 부트로더로 채택되어 사용된다.==

- **동작 방식**: 전원이 켜지면 SoC 내부의 극초기 부트로더(MB1, MB2)가 QSPI-NOR 플래시의 UEFI 펌웨어(EDK2 기반)를 실행한다.
- **역할**: UEFI가 PCIe, USB, SDIO 등의 드라이버를 로드한 뒤, NVMe SSD나 SD 카드에 있는 `extlinux.conf` 파일 및 리눅스 커널(`Image`)을 찾아 커널에 제어권을 넘겨준다.

> [!summary] 한 줄 요약
> UEFI는 "하드웨어(SoC)와 운영체제(Linux/Windows) 사이를 연결해 주는 똑똑하고 안전한 부팅 가이더"라고 이해하면 된다.

---

## 3. U-Boot는 사용하지 않는가?

==**결론: Jetson Orin 시리즈(Jetpack 5/6)부터는 U-Boot를 사용하지 않는다.**==

과거 임베디드 리눅스 개발 환경에서는 U-Boot(Universal Boot Loader)가 사실상 표준 부트로더로 사용되었고, 이전 세대 Jetson(Nano, TX2, Xavier 등 / Jetpack 4 이하)에서도 `MB2 → U-Boot → Linux Kernel` 순서로 부팅했다.

하지만 Orin 시리즈부터 NVIDIA는 U-Boot를 완전히 폐지하고 표준 **UEFI(EDK2 기반)** 로 대체했다.

### 3.1 U-Boot → UEFI 전환 이유

1. **서버/엔터프라이즈 및 엔드포인트 표준화**
   - ARM 서버 및 컴퓨팅 아키텍처가 표준 UEFI/ACPI 또는 UEFI/Device Tree 규격으로 통합되는 추세.
   - Red Hat, Ubuntu, SUSE 등 범용 리눅스 배포판의 아키텍처 표준에 맞춰 OS 이식성과 호환성 향상.

2. **NVMe / PCIe 스토리지 지원 강화**
   - U-Boot 환경에서는 복잡한 PCIe 초기화, NVMe 드라이버, USB Type-C 드라이버 등을 커스텀 구현하기가 번거로웠음.
   - UEFI(EDK2)는 풍부한 표준 드라이버 생태계를 갖추고 있어 NVMe SSD, 네트워크(PXE), USB 등 다양한 스토리지 부팅을 안정적으로 구현 가능.

3. **강력한 보안 체계 (Secure Boot)**
   - 최신 차량용/산업용 AI 장비에서 보안이 중요해짐에 따라, UEFI 표준 Secure Boot(디지털 서명 검증) 및 TPM 연동 프로세스를 U-Boot보다 훨씬 수월하게 적용 가능.

### 3.2 Orin 시리즈 전체 부팅 흐름 (Boot Flow)

U-Boot가 빠지면서 Orin의 부팅 흐름은 아래와 같이 단순화되었다.

```text
[ 전원 ON ]
    ↓
1. BootROM (SoC 내부 롬)
    ↓
2. MB1 (Microbootloader 1 - QSPI-NOR) : 하드웨어 및 DRAM 최하위 초기화
    ↓
3. MB2 / TFA (Trusted Firmware-A) : ARM 시큐어 월드(EL3) 세팅
    ↓
4. UEFI (EDK2 Bootloader) : PCIe/NVMe 초기화 및 extlinux.conf 탐색
    ↓
5. Linux Kernel (Image & DTB) 로드 및 실행
```

### 3.3 제어 방식의 변화

| 구분 | 이전 (U-Boot) | 현재 (UEFI) |
| --- | --- | --- |
| 부팅 명령 설정 | `bootcmd`, `bootargs` | `extlinux.conf` |
| 인터페이스 | U-Boot CLI (명령줄) | UEFI 메뉴 / OS 내 설정 파일 |
| 커널 파라미터 경로 | U-Boot 환경변수 | `/boot/extlinux/extlinux.conf` |

기존 U-Boot에서 다루던 `bootcmd`, `bootargs` 설정이나 U-Boot 명령줄 인터페이스(CLI) 제어 방식은 Orin NX에서 더 이상 사용되지 않으며, 대신 UEFI 메뉴나 OS 내의 `/boot/extlinux/extlinux.conf` 설정을 통해 동일한 제어를 수행한다.

---

## 관련 노트

- [[Boot & Storage 구성]]
