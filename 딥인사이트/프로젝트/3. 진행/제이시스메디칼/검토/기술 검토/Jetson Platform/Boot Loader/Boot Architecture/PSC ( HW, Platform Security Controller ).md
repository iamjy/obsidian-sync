---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
NVIDIA Orin의 전체 부트 플로우(Boot Flow)를 짚어보셨군요. Orin SoC에서는 시스템이 거대해지고 보안(Security)과 차량·로봇용 안전성(Functional Safety) 요구가 극대화되면서, ==기존의 BPMP 외에도 PSC와 FSI라는 핵심 독립 프로세서 블록이 초기 부트 체인에 매우 깊숙이 관여하게 되었습니다==. [1, 2]

각각의 역할과 부팅 흐름에서의 위치를 쉽게 정리해 드릴게요.

---

## 1. PSC (Platform Security Controller) — "보안의 총책임자"

PSC는 시스템의 보안과 암호화 키 관리를 전담하는 격리된 하드웨어 보안 프로세서입니다. [1, 3]

- 역할: 암호화/복호화 엔진, 보안 키 저장소, 하드웨어 난수 생성기 등을 내장하고 있습니다. 부팅 단계뿐만 아니라 시스템이 켜져 있는 내내 모든 보안 인증을 관리합니다. [1, 2, 3]
- 부트 플로우에서의 개입:
    
    1. 전원이 켜지자마자 가장 먼저 PSCROM이라는 하드웨어 코드가 실행됩니다.
    2. PSC는 시스템에 저장된 암호화 키를 사용해 다음 단계로 실행될 BPMP의 BootROM과 MB1(Microboot 1) 이미지의 무결성을 검증(Authentication)하고 서명을 확인합니다.
    3. 이 검증이 끝나야 비로소 BPMP가 안전하게 부팅을 시작할 수 있습니다. [1, 3]
    

## 2. FSI (Functional Safety Island) — "기능 안전의 감시자"

FSI는 자율주행이나 산업용 로봇 등에서 시스템 오류로 인한 사고를 막기 위한 차량용 안전 표준(ISO 26262 ASIL-D)을 충족하는 하드웨어 섬(Island)입니다. [4, 5]

- 역할: 메인 CPU(CCPLEX)나 GPU가 죽거나 오작동하더라도 안전하게 시스템을 멈추거나 백업 제어를 할 수 있도록, 완전히 독립된 전원(Power Rail)과 클록을 사용합니다. 내부에 4개의 ARM Cortex-R52 코어가 2쌍씩 묶여 DCLS(Dual-Core Lock-Step) 방식으로 똑같은 연산을 동시에 비교하며 오류를 잡아냅니다. [5, 6]
- 부트 플로우에서의 개입:
    
    - 부팅 초기 단계에서 부트로더가 FSI 전용 바이너리(펌웨어)를 FSI 내장 메모리에 로드해 줍니다. FSI가 깨어나면 SoC 전반의 하드웨어 진단(In-System-Test)을 수행하고 오류 모니터링 모듈(HSM)을 활성화하여 시스템이 안전하게 운영체제(OS)로 진입할 수 있는 환경인지 감시하기 시작합니다. [2, 7, 8]
    

---

## 🔄 Orin의 최종 부트 플로우 순서 요약

전원 인가 후 메인 OS가 켜지기까지의 핵심 흐름은 아래와 같습니다. [1]

1. ⚡ 전원 인가 (Power On)
2. 🔐 PSC 단계: `PSCROM` 가동 → 시스템 키 로드 → BPMP 코드 검증
3. 🔋 BPMP 단계: `BPMP BootROM` 실행 → `MB1(Microboot 1)` 실행 → 기본적인 클록, 전원, 메모리(DDR) 초기화 → FSI 및 메인 CPU용 부트로더 준비
4. 🛡️ FSI 단계: `FSI 펌웨어` 가동 → 하드웨어 안전 진단 및 차량/로봇 관점의 세이프티 모니터링 시작
5. 💻 CCPLEX 단계: 메인 CPU 클러스터가 깨어나 `MB2` → `UEFI` 부트로더를 거쳐 최종적으로 `Linux / DriveOS` 같은 메인 OS 커널을 로드. [1, 2, 3, 7, 9, 10]

결과적으로 PSC(보안 검증) → BPMP(전원/기반 초기화) → FSI(안전 감시) → CCPLEX(메인 OS 구동)의 철저한 단계를 거쳐 안전하게 부팅이 완료됩니다. [1, 2, 10]

혹시 이 프로세서들 간의 통信 구조(예: BPMP와 CCPLEX 간의 IPC)나, Secure Boot를 위해 키를 퓨징(Fusing)하는 과정 중 어떤 단계가 더 궁금하신가요?

  

[1] [https://developer.nvidia.com](https://developer.nvidia.com/docs/drive/drive-os/6.0.8/public/drive-os-linux-sdk/common/topics/bootloader_setup/UnderstandingtheBootFlowProcess1.html)

[2] [https://developer.nvidia.com](https://developer.nvidia.com/docs/drive/drive-os/6.0.9/public/drive-os-linux-sdk/common/topics/fsi_integration/Functional_Safety_Island.html)

[3] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html)

[4] [https://www.aivon.com](https://www.aivon.com/blog/automotive-electronics/nvidia-orin-soc-analysis-for-adas/)

[5] [https://developer.nvidia.com](https://developer.nvidia.com/docs/drive/drive-os/7.2.5/public/drive-os-linux-sdk-ea/embedded-software-components/Functional_Safety_Island_FSI/Functional_Safety_Island.html)

[6] [https://developer.download.nvidia.com](https://developer.download.nvidia.com/assets/igx/robotics-product-brief-igx-thor-safety-4473375.pdf)

[7] [https://developer.nvidia.com](https://developer.nvidia.com/blog/using-the-power-of-ai-to-make-factories-safer/)

[8] [https://developer.nvidia.com](https://developer.nvidia.com/docs/drive/drive-os/7.2.5/public/drive-os-linux-sdk-ea/core-concepts/safety_framework_error_reporting.html)

[9] [https://developer.nvidia.com](https://developer.nvidia.com/docs/drive/drive-os/archives/6.0.3/linux/sdk/oxy_ex-1/common/topics/bootloader_setup/UnderstandingtheBootFlowProcess1.html)

[10] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/agx-orin-devkit-36-4-disk-encryption/361868)