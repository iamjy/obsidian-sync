---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
# Cold Boot (콜드 부트)

NVIDIA Jetson 시리즈에서 **콜드 부트(Cold Boot)**란 ==**장치의 전원이 완전히 꺼진 상태(Power-off)에서 처음으로 전원을 켜서 시스템을 기동하는 과정**==을 의미합니다. [[1](https://www.thegoodpenguin.co.uk/blog/diving-into-the-nvidia-jetson-nano-boot-process/), [2](https://www.lenovo.com/kr/ko/glossary/cold-boot/?srsltid=AU7gw4Us_PDsI-Dr5c5b-4VPR0YyHYQhyhewo4TA6m9WJNAB-NWDXUIm)]

Jetson 임베디드 보드의 부팅 프로세스는 크게 **콜드 부트**와 **웜 부트(Warm Boot)**로 구분됩니다. [[1](https://www.thegoodpenguin.co.uk/blog/diving-into-the-nvidia-jetson-nano-boot-process/)]
### 1. 콜드 부트 vs 웜 부트 비교
| 구분 | 콜드 부트 (Cold Boot) | 웜 부트 (Warm Boot / Reboot) |
| :--- | :--- | :--- |
| **시작 상태** | 전원이 완전히 차단되었거나 Reset 상태 | 전원이 공급된 상태 (서스펜드 해제, 소프트웨어 재시작) |
| **동작 특징** | 칩셋 하드웨어 및 RAM 초기화 $\rightarrow$ BootROM부터 실행 | 상당 단계의 하드웨어 초기화를 건너뛰고 OS 커널 등만 재기동 |
| **소요 시간** | 상대적으로 느림 (전 단계 부트로더 거침) | 빠름 (이미 초기화된 장치 활용) |
---

### 2. NVIDIA Jetson 콜드 부트의 기술적 시퀀스

[NVIDIA Jetson Boot Flow](https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html) 문서에 따르면, 콜드 부트 시 다음과 같은 하드웨어 시퀀스가 진행됩니다: [[1](https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html)]
- **Step 1. BootROM 및 하드웨어 초기화**
    - 전원 인가 시 내장된 BootROM이 최우선 실행됩니다.
    - 미디어 저장소, 외부 메모리 컨트롤러(EMC), CPU 등을 깨끗한 상태(**Clean State**)에서 완전히 초기화합니다. [[1](https://translate.google.com/translate?u=https://forums.developer.nvidia.com/t/recovery-mode-cold-reset-or-warm-reset/365791&hl=ko&sl=en&tl=ko&client=sge), [2](https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html)]
- **Step 2. 부트로더 레벨 초기화**
    - MB1 $\rightarrow$ MB2 $\rightarrow$ UEFI 등의 다단계 부트로더 바이너리가 순차적으로 로드됩니다.
    - 이 과정에서 보안 매개변수를 설정하고 신뢰성 체인(**Chain of Trust**)을 구축합니다. [[1](https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html)]
- **Step 3. 복구 모드(Recovery Mode) 진입 시 주의사항**
    - 새 펌웨어나 OS(JetPack) 플래싱을 위해 `FORCE_RECOVERY` 모드 진입 시, 잔류 데이터로 인한 오류를 방지하기 위해 **콜드 리셋(Cold Reset) 방식**을 강력히 권장합니다. [[1](https://forums.developer.nvidia.com/t/recovery-mode-cold-reset-or-warm-reset/365791), [2](https://translate.google.com/translate?u=https://forums.developer.nvidia.com/t/recovery-mode-cold-reset-or-warm-reset/365791&hl=ko&sl=en&tl=ko&client=sge)]
---

### 3. 개발 시 주요 고려사항 및 이슈
#### 🚀 부팅 시간 최적화
자율주행, 로봇 등 실시간성이 중요한 제품 적용 시 초기 콜드 부트 시간이 길어질 수 있습니다.
- **해결 방안**: [NVIDIA 부팅 최적화 가이드](https://docs.nvidia.com/jetson/archives/r34.1/DeveloperGuide/text/SD/Kernel/BootTimeOptimzation.html)를 참조하여 사용하지 않는 컨트롤러의 **디바이스 트리(Device Tree)** 비활성화 또는 커널 드라이버 로드 방식 변경을 통해 단축 가능합니다. [[1](https://forums.developer.nvidia.com/t/how-to-optimize-the-cold-boot-time-of-xavier-nx/374119), [2](https://docs.nvidia.com/jetson/archives/r34.1/DeveloperGuide/text/SD/Kernel/BootTimeOptimzation.html)]

#### ❄️ 저온 환경 부팅 실패
영하 20도 이하의 극한 환경에서는 하드웨어적 특성으로 인해 콜드 부트가 정상 진행되지 않는 사례가 보고됩니다.
- **원인**: 내부 온도 측정 `SOC_THERM` 센서 검증 오류 등 [[1](https://forums.developer.nvidia.com/t/jetson-agx-orin-cold-boot-issue/262601), [2](https://forums.developer.nvidia.com/t/jetson-orin-nx-cold-boot-failure-at-20-c-soc-therm-reports-invalid-temperature/356467), [3](https://forums.developer.nvidia.com/t/jetson-xavier-nx-cold-temperatures/280232), [4](https://forums.developer.nvidia.com/t/orin-nano-boot-up-in-cold-temperatures/265535)]
- **대응**: 프리히팅(Pre-heating) 스위치 회로 구성 등을 통해 해결합니다.