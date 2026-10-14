---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
# RCM Boot (Recovery Mode Boot)

NVIDIA Jetson의 **RCM boot**란 ==호스트 PC에서 USB를 통해 부팅 바이너리들을 직접 SDRAM으로 전송하여 실행하는 방식==입니다. [[1](https://forums.developer.nvidia.com/t/rcm-boot-and-normal-flash/82118), [2](https://docs.nvidia.com/jetson/archives/r35.6.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html)]

---

### 🛠 RCM Boot의 주요 특징
- **저장소 쓰기 없음 (No Storage Write)**
    - 일반적인 부팅(Cold Boot)과 달리, 바이너리를 내부 저장소(eMMC, NVMe 등)에 플래싱(기록)하지 않습니다.
    - 대신 일시적으로 메모리(**SDRAM**)에 로드하여 실행하는 방식을 취합니다. [[1](https://docs.nvidia.com/jetson/archives/r35.6.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html), [2](https://forums.developer.nvidia.com/t/rcm-boot-and-normal-flash/82118)]
- **호스트 PC 연결 기반**
    - USB 케이블을 통해 호스트 PC와 물리적으로 연결된 상태에서 동작합니다. [[1](https://forums.developer.nvidia.com/t/rcm-boot-and-normal-flash/82118), [2](https://docs.nvidia.com/jetson/archives/r35.6.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html)]
- **주요 사용 목적**
    - **초기화 및 설치**: 운영체제(Linux/BSP)를 설치하거나 **플래싱(Flashing)** 하는 과정에서 필수적으로 사용됩니다.
    - **복구 및 진단**: 시스템이 손상되어 일반적인 부팅이 불가능할 때 복구 작업을 수행합니다. [[1](https://blog.csdn.net/m0_50136807/article/details/140484479), [2](https://forums.developer.nvidia.com/t/rcm-boot-and-normal-flash/82118)]
---

### 📊 일반 부팅(Cold Boot) vs RCM Boot 비교

| 구분 | Cold Boot (일반 부팅) | RCM Boot (복구 모드 부팅) |
| :--- | :--- | :--- |
| **부팅 경로** | 내부 저장소 $\rightarrow$ RAM | 호스트 PC (USB) $\rightarrow$ RAM |
| **데이터 로드** | 내장된 부트로더와 OS를 읽어 실행 | 핵심 컴포넌트(MB1, MB2 등)를 전송받아 실행 |
| **저장소 영향** | 저장소의 데이터를 기반으로 작동 | 저장소 쓰기 없이 메모리에서 일시 실행 |

*근거: [[1](https://docs.nvidia.com/jetson/archives/r36.5.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html), [2](https://docs.nvidia.com/jetson/archives/r35.6.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html), [3](https://blog.csdn.net/m0_50136807/article/details/140484479)]*

---
> [!QUESTION] **추가 확인 사항**
> - RCM 모드에 진입하시려는 **특정 Jetson 모델명**이 있으신가요?
> - 현재 **플래싱이나 복구 작업 중** 발생한 구체적인 문제가 있으신가요?