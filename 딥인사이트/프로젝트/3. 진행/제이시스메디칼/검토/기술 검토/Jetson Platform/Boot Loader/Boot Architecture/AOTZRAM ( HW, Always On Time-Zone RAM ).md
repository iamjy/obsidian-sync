---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
**AOTZRAM**은 ==엔비디아(NVIDIA)의 임베디드 시스템 온 칩(SoC)인 **Jetson AGX Orin 시리즈의 부트 아키텍처**에서 사용되는 특수 보안 메모리 영역==을 뜻합니다. [[1](https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html)]

일반적으로 **Always On Time-Zone RAM**(또는 Always-On Trust Zone RAM)의 약어로 해석되며, 시스템이 최초로 부팅될 때 보안 유지를 위해 다음과 같은 흐름으로 작동합니다.

- **초기 부팅 단계:** 하드웨어 리셋 직후 BootROM(BR) 프로세스가 시작되면, 시스템은 가장 먼저 **AOTZRAM** 공간에 **Microboot1(MB1)**이라는 첫 번째 부팅 소프트웨어 컴포넌트를 로드합니다. [[1](https://docs.nvidia.com/jetson/archives/r35.6.5/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html), [2](https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html)]

- **주요 역할:** 이 영역에 로드된 MB1은 SoC의 특정 하드웨어 부품을 초기화하고, 보안 구성을 설정하며, SDRAM(메모리) 및 가상 방화벽 등을 구축하는 핵심적인 징검다리 역할을 수행합니다. [[1](https://docs.nvidia.com/jetson/archives/r35.6.5/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html)]

궁금하신 내용이 엔비디아 젯슨(Jetson) 칩셋의 하드웨어 개발이나 부트로더 커스텀과 관련된 부분인가요? 추가로 알고 싶으신 **임베디드 보안 영역**이나 **Orin 부팅 시퀀스(Boot Flow)**가 있다면 편하게 말씀해 주세요!

---

AOTZRAM은 ==엔비디아(NVIDIA)의 임베디드 시스템인 Jetson Orin 시리즈(AGX Orin, Orin NX, Orin Nano 등)의 부팅 프로세스에서 사용되는 보안 메모리 영역(Always-On TrustZone RAM)==을 의미합니다. [1, 2]

하드웨어 부팅 초기 단계에서 시스템의 안전성과 신뢰성을 확보하기 위해 사용되는 핵심 컴포넌트입니다.

## 💡 주요 역할 및 부팅 흐름

1. BootROM 단계: 칩에 전원이 켜지면 하드웨어에 내장된 고정 코드인 BootROM(BR)이 가장 먼저 실행됩니다. [1, 2]
2. MB1 로드: BootROM은 첫 번째 소프트웨어 부팅 컴포넌트인 Microboot1(MB1)을 바로 이 AOTZRAM 영역에 로드하여 실행합니다. [1, 3]
3. 초기화 및 보안 설정: AOTZRAM 내에서 실행되는 MB1은 SoC의 내부 핀 매핑(Pinmux), GPIO, 외부 메모리 컨트롤러(SDRAM) 등을 초기화하고, 시스템 전반의 보안 방화벽을 설정하는 고도로 보안된 작업을 수행합니다. [2, 4]

상기 정보는 [NVIDIA Jetson Linux Developer Guide](https://docs.nvidia.com/jetson/archives/r35.6.5/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html) 부팅 아키텍처 공식 문서에서 기술적인 세부 사항을 확인할 수 있습니다. [2]

현재 Jetson Orin 보드의 부팅 로그(Serial Log) 분석이나 임베디드 펌웨어 개발/디버깅 중에 이 용어를 접하셨나요? 관련하여 더 궁금한 부분(예: MB1, MB2 구조나 부팅 루프 트러블슈팅)이 있다면 편하게 말씀해 주세요.

  

[1] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html)

[2] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/archives/r35.6.5/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html)

[3] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/archives/r34.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html)

[4] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/archives/r35.6.5/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html)

---

AOTZRAM은 SRAM의 한 종류가 맞습니다. 정확히는 SoC(칩) 내부에 결합된 내부 SRAM(On-chip SRAM) 공간 중 일부를 보안 전용 영역으로 지정해 둔 것입니다.

두 개념의 차이와 관계를 명확하게 이해하실 수 있도록 핵심만 정리해 드립니다.

## 1. 하드웨어적 본질은 SRAM입니다

- SRAM (Static RAM): 전원이 공급되는 동안 데이터가 유지되는 매우 빠른 메모리입니다. 컴퓨터의 CPU 캐시 메모리나 임베디드 칩의 내부 메모리로 사용됩니다.
- AOTZRAM의 물리적 구조: Jetson Orin 칩 내부(On-chip)에 물리적으로 존재하는 고속 SRAM 소자입니다. 메인 메모리(DRAM)가 켜지기 전에도 기본적으로 작동합니다.

## 2. 'AOTZRAM'이라는 이름이 붙은 이유 (차이점)

일반적인 SRAM과 달리, 앞에 붙은 글자들은 이 메모리의 특수한 기능과 목적을 나타냅니다.

- AO (Always-On): 칩이 초절전 모드(Deep Sleep 등)에 들어가 다른 물리 장치들의 전원이 꺼져도, 이 영역은 항상 전원이 공급(Always-On)되어 데이터와 보안 상태를 유지합니다.
- TZ (TrustZone): Arm 아키텍처의 하드웨어 보안 기술인 TrustZone(트러스트존)에 의해 보호받는 영역입니다. 일반 애플리케이션이나 OS(리눅스 등)는 하드웨어 방화벽에 막혀 이 메모리 영역에 절대 접근할 수 없고, 오직 허가된 보안 프로세스(예: MB1 부트로더)만 접근할 수 있습니다.

## 💡 한눈에 보는 비교

|구분|일반 SRAM|AOTZRAM|
|---|---|---|
|물리적 소자|SRAM|SRAM (동일함)|
|위치|칩 내부 또는 외부|칩 내부 (On-chip)|
|전원 관리|절전 모드 시 전원 차단 가능|칩이 켜져 있는 한 항상 전원 유지 (Always-On)|
|보안 레벨|누구나 읽고 쓸 수 있음 (일반 목적)|최고 등급 보안 (TrustZone 보호)|
|주요 용도|일반 데이터 임시 저장, 캐시|부팅 초기 보안 코드(MB1) 실행, 암호화 키 처리|

요약하자면, "AOTZRAM은 칩 안에서 항상 켜져 있고 보안 처리가 된 특수 목적용 SRAM이다"라고 이해하시면 정확합니다.

현재 작업 중이신 환경에서 메모리 맵(Memory Map) 할당이나 부팅 단계의 하드웨어 초기화와 관련해 구체적으로 확인하고 싶으신 부분이 있으신가요?

필요하시다면 Jetson Orin의 전체 메모리 구조(SRAM vs DRAM)나 부팅 프로세스에서의 역할을 더 상세히 설명해 드릴 수 있습니다.