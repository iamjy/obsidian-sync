---
title: Boot & Storage 구성
source: https://professional.avermedia.com/Products/embedded-systems/orin-nx
author:
published:
created: 2025-09-07
description: AVerMedia Orin NX 캐리어 보드 시리즈별 Storage / SD Card 지원 정리
tags:
  - jetson
  - orin-nx
  - avermedia
  - storage
---

## AVerMedia Orin NX 캐리어 보드 — Storage / SD Card 지원 정리

> 출처: [AVerMedia Orin NX 제품 페이지](https://professional.avermedia.com/Products/embedded-systems/orin-nx) 및 각 제품 상세 페이지 (2026-08 확인)

### 핵심 요약
- **모든 시리즈 공통**: 주 저장장치는 **M.2 Key M 2280 슬롯 (NVMe SSD) 1개**. 별도의 **M.2 Key E 2230 슬롯**은 Wi-Fi 전용(저장용 아님).
- **eMMC / SATA**: 캐리어 보드 차원에서는 미지원. (eMMC는 일부 Orin NX/Nano 모듈(SOM) 자체에 내장될 수 있으나, 이는 캐리어 보드가 아닌 모듈 속성)
- ==**microSD/SD Card**: 물리 슬롯이 있는 보드라도 **Orin NX/Orin Nano에서는 동작하지 않음** — SOM(모듈) 제약사항. Orin NX/Nano 기준 실질적으로 **SD Card 사용 불가**.==
- **NVMe 용량 제한**: 제품 페이지에 명시된 상한 없음 — 장착하는 2280 NVMe SSD 용량에 따름.

### 시리즈별 정리

| 시리즈          | 모델        | Storage 지원                                                | SD Card (microSD)                                          | 비고         |
| ------------ | --------- | --------------------------------------------------------- | ---------------------------------------------------------- | ---------- |
| **D133**     | D133      | M.2 Key M 2280 (NVMe SSD) ×1 + M.2 Key E 2230 (Wi-Fi)     | ❌ 슬롯 없음                                                    |            |
|              | D133-1    | 동일                                                        | ❌ 슬롯 없음                                                    |            |
|              | D133S     | 동일 (Super Mode)                                           | ❌ 슬롯 없음                                                    |            |
| **D115**     | D115S     | M.2 Key M 2280 (NVMe SSD) ×1 + M.2 Key E 2230 (Wi-Fi)     | ❌ 슬롯 없음                                                    | Super Mode |
|              | D115W     | 동일                                                        | ❌ 슬롯 없음                                                    |            |
|              | D115W-1   | 동일                                                        | ❌ 슬롯 없음                                                    |            |
| ==**D131**== | ==D131==  | ==M.2 Key M 2280 (NVMe SSD) ×1 + M.2 Key E 2230 (Wi-Fi)== | ==⚠️ 슬롯 존재하나 **Orin NX/Nano 미지원** (SOM 제약)==               |            |
|              | ==D131L== | ==동일==                                                    | ==❌ 슬롯 미표기==                                               |            |
|              | ==D131S== | ==동일 (Super Mode)==                                       | ==⚠️ 슬롯 존재하나 **Orin NX/Nano 미지원** (SOM 제약)==               |            |
| **NX215**    | NX215     | M.2 Key M 2280 (NVMe SSD) ×1 + M.2 Key E 2230 (Wi-Fi)     | ⚠️ 슬롯 존재하나 **Orin NX/Nano 미지원** (Xavier NX/TX2 NX/Nano 전용) | 멀티 모듈 호환   |

==**범례**: ❌ 미지원 / 슬롯 없음 · ⚠️ 물리 슬롯은 있으나 Orin NX/Nano에서 사용 불가==

### 부팅/스토리지 관점 시사점
- Orin NX/Nano 조합에서는 **NVMe SSD(M.2 2280)가 유일한 실용 저장/부팅 매체**. SD Card 부팅은 사실상 선택지에서 제외.
- eMMC 부팅이 필요하면 **eMMC 내장 SOM 모델** 선택 필요(캐리어 보드가 아닌 모듈 사양 확인).
- 어떤 시리즈를 택하든 저장 아키텍처는 동일하므로, 스토리지 기준으로는 캐리어 보드 간 차이가 없고 I/O·폼팩터·Super Mode 지원 여부로 선택하면 됨.

---

## Jetson Modules and Configurations (L4T r36.5)

> 출처: [NVIDIA Jetson Linux r36.5 Developer Guide — Quick Start](https://docs.nvidia.com/jetson/archives/r36.5/DeveloperGuide/IN/QuickStart.html)
> `flash.sh` / `l4t_initrd_flash.sh` 사용 시 지정하는 보드 설정(config) 이름과 각 모듈의 플래시 대상 저장매체 정리

### 모듈 ↔ 설정(config) 매핑

| 모듈                             | Part No.       | 타입              | 캐리어 보드                          | Configuration                                                     | 플래시 대상 저장매체                                           |
| ------------------------------ | -------------- | --------------- | ------------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------- |
| Jetson Orin NX 16GB-DRAM       | P3767-0000     | Production      | Orin Nano ref. (P3768-0000)     | `jetson-orin-nano-devkit` / `jetson-orin-nano-devkit-super` *     | QSPI-NOR + USB/NVMe (initrd flash 전용)                 |
| Jetson Orin NX 8GB-DRAM        | P3767-0001     | Production      | Orin Nano ref. (P3768-0000)     | `jetson-orin-nano-devkit` / `jetson-orin-nano-devkit-super` *     | QSPI-NOR + USB/NVMe (initrd flash 전용)                 |
| Jetson Orin Nano 8GB-DRAM      | P3767-0003     | Production      | Orin Nano ref. (P3768-0000)     | `jetson-orin-nano-devkit` / `jetson-orin-nano-devkit-super` *     | QSPI-NOR + USB/NVMe (initrd flash 전용)                 |
| Jetson Orin Nano 4GB-DRAM      | P3767-0004     | Production      | Orin Nano ref. (P3768-0000)     | `jetson-orin-nano-devkit` / `jetson-orin-nano-devkit-super` *     | QSPI-NOR + USB/NVMe (initrd flash 전용)                 |
| ==Jetson Orin Nano 8GB-DRAM==  | ==P3767-0005== | ==Development== | ==Orin Nano ref. (P3768-0000)== | ==`jetson-orin-nano-devkit` / `jetson-orin-nano-devkit-super` *== | ==QSPI-NOR + **microSD**/USB/NVMe (initrd flash 전용)== |
| Jetson AGX Orin Dev-Kit Module | P3701-0000     | Development     | AGX Orin ref. (P3737-0000)      | `jetson-agx-orin-devkit`                                          | QSPI-NOR + eMMC                                       |
| Jetson AGX Orin 32GB-DRAM      | P3701-0004     | Production      | AGX Orin ref. (P3737-0000)      | `jetson-agx-orin-devkit`                                          | QSPI-NOR + eMMC                                       |
| Jetson AGX Orin 64GB-DRAM      | P3701-0005     | Production      | AGX Orin ref. (P3737-0000)      | `jetson-agx-orin-devkit`                                          | QSPI-NOR + eMMC                                       |
| Jetson AGX Orin Industrial     | P3701-0008     | Production      | AGX Orin ref. (P3737-0000)      | `jetson-agx-orin-devkit-industrial`                               | QSPI-NOR + eMMC                                       |

`*` **Super 설정**: 표준 `jetson-orin-nano-devkit.conf` 대비 `jetson-orin-nano-devkit-super.conf`는 **더 높은 전력 예산(power budget)과 확장된 클럭 주파수 스텝**을 제공(성능 향상). 세부는 Developer Guide의 *Supported Modes and Power Efficiency* 참고.

### 시사점 (본 프로젝트 관점)
- **Orin NX 16GB/8GB(P3767-0000/0001)**: 저장 대상이 **QSPI-NOR + USB/NVMe** 뿐 → **microSD 부팅 옵션 없음**. `l4t_initrd_flash.sh`로만 NVMe 플래시 지원.
  - 이는 앞선 AVerMedia 캐리어 보드 결과(**Orin NX/Nano는 SD Card 사용 불가**)와 일치 → **NVMe SSD가 사실상 유일 부팅 매체**.
- **microSD 플래시가 표에 명시된 모듈은 Orin Nano 8GB Development(P3767-0005) 하나뿐** — 개발용 모듈이며 Production Orin NX 라인업에는 해당 없음.
- config 이름은 Orin NX/Nano 모듈 모두 `jetson-orin-nano-devkit(-super)`로 공유됨 → 플래시 스크립트 관점에서 동일 보드 설정 사용.

---

## Partition Configuration Files (L4T r36.5)

> 출처: [NVIDIA Jetson Linux r36.5 Developer Guide — Partition Configuration](https://docs.nvidia.com/jetson/archives/r36.5/DeveloperGuide/AR/BootArchitecture/PartitionConfiguration.html)
> 저장장치(내장/외장)의 파티션 레이아웃을 정의하는 XML 설정 파일

### 목적
- 저장장치(mass storage)와 그 파티션 구성을 기술 → **OS 이미지 · 부트로더 · 펌웨어 · 스플래시 화면**의 저장 위치를 정의.
- 플래시 시 `flash.sh`가 이 XML을 읽어 파티션 레이아웃을 구성.

### 플랫폼별 파티션 config 파일 매핑

| 플랫폼 / 구성                                                                                                               | Boot 파티션 디바이스 | User 파티션 디바이스              | 파일                          |
| ---------------------------------------------------------------------------------------------------------------------- | ------------- | -------------------------- | --------------------------- |
| Jetson AGX Orin Developer-Kit                                                                                          | QSPI_NOR      | sdmmc_user (32GB eMMC)     | `flash_t234_qspi_sdmmc.xml` |
| ==Jetson Orin Nano Developer-Kit==                                                                                         | ==QSPI-NOR==      | ==SD card / USB / NVMe drive== | ==`flash_t234_qspi_sd.xml`==    |
| **상용 모듈**: AGX Orin 32GB (P3701-0004), AGX Orin 64GB (P3701-0005), AGX Orin Industrial (P3701-0008)                    | QSPI_NOR      | sdmmc_user (eMMC)          | `flash_t234_qspi_sdmmc.xml` |
| **상용 모듈**: Orin NX 16GB (P3767-0000), Orin NX 8GB (P3767-0001), Orin Nano 8GB (P3767-0003), Orin Nano 4GB (P3767-0004) | QSPI_NOR      | **USB / NVMe drive**       | `flash_t234_qspi_sd.xml`    |

> ⭐ **본 프로젝트 대상(Orin NX 16GB/8GB P3767-0000/0001)**: Boot = **QSPI-NOR**, User(rootfs) = **USB/NVMe drive**, 파티션 config 파일 = **`flash_t234_qspi_sd.xml`**. (파일명은 `_sd`지만 Orin NX/Nano 상용 모듈에서는 SD Card가 아닌 USB/NVMe가 실제 User 파티션 대상)

### 파일 포맷 & 명명 규칙
- **XML 형식**. 파일명에 다음 요소 포함:
  - 프로세서 타입 (예: `t234` = Orin)
  - 저장 매체 타입: `sd`(SD Card) · `spi`(SPI/QSPI) · `emmc`(eMMC) · `sdmmc`
  - (선택) 모듈 Part No.
- 예시: `flash_t234_qspi_sdmmc.xml`, `flash_t234_qspi_sd.xml`

### 처리 흐름 (플래시 시)
1. `flash.sh`가 파티션 config XML을 읽음
2. XML 내 키워드를 `.conf` 파일 값 또는 커맨드라인 옵션 값으로 치환
3. 치환 결과를 `bootloader/flash.xml`로 저장
4. `tegraflash.py`가 `flash.xml`을 읽어 실제 디바이스에 플래시

### XML 구조
- **Root**: `<partition_layout version="01.00.0000">`
- **`<device>`**: 저장장치 정의
  - `type`: `sdmmc_boot` · `sdmmc_user` · `SPI` · `nvme`
  - `instance`: 0–3
- **`<partition>`**: 파티션 정의
  - 속성: `name`, `type`, `oem_sign`(True/False)
  - **name 최대 길이: 36자**

### `<partition>` 하위 요소

| 요소 | 값 | 용도 |
|------|-----|------|
| `<allocation_policy>` | `sequential` | 파티션 배치 순서 |
| `<filesystem_type>` | `basic` | raw 파티션 포맷 지정 |
| `<size>` | bytes (10진/16진) | 파티션 크기 |
| `<filename>` | 파일명 또는 공백 | 파티션에 기록할 데이터 파일 |
| `<allocation_attribute>` | `0x008` 또는 `0x808` | secondary GPT 직전 파티션은 반드시 `0x808` |

### 치환 키워드 (기본값 예시)
- `APPSIZE`: 30064771072 bytes (≈28GB, APP/rootfs 파티션)
- `APPFILE`: `system.img`
- `LNXSIZE`: 67108864 bytes (64MB)
- `LNXFILE`: `boot.img`
- 부트로더 파일: `MB1FILE`, `SPEFILE`, `TEGRABOOT` 등

### 외장 저장장치(NVMe/SCSI) 파티션
- 디바이스 타입 `nvme`, `instance 0` 사용.
- **최소 3개 파티션 필수**:
  - `master_boot_record` (보호용 MBR)
  - `primary_gpt`
  - `secondary_gpt`
- `num_sectors`는 디바이스 용량에 맞게 조정: **총 바이트 ÷ 512**.

### 시사점 (본 프로젝트 관점)
- Orin NX(NVMe 부팅) 기준 → **외장 NVMe용 파티션 XML(`type="nvme"`)** 을 사용, `master_boot_record`/`primary_gpt`/`secondary_gpt` 포함한 레이아웃 필요.
- 커스텀 파티션(예: 별도 데이터 영역 추가, rootfs 크기 조정)이 필요하면 이 XML의 `<size>`·`<partition>` 수정으로 대응 → `APPSIZE` 등 키워드 또는 직접 값 지정.
- QSPI-NOR(부트체인)와 NVMe(rootfs)가 분리되므로, 파티션 config도 QSPI용과 NVMe용을 함께 다뤄야 함(`flash_t234_qspi_sdmmc.xml` 계열 참고).
