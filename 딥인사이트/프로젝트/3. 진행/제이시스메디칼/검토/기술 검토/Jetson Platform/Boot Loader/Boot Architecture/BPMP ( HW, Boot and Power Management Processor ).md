---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
NVIDIA의 **BPMP(Boot and Power Management Processor)**는 ==테그라(Tegra) 및 오린(Orin) 같은 SoC(시스템 온 칩) 내부에 탑재된 소형 보조 프로세서(코프로세서)==입니다. [1, 2, 3]

## 주요 역할과 기능

- 초기 부팅 처리: 전원이 켜지면 메인 CPU(CCPLEX)보다 먼저 깨어나 `BootROM` 등 초기 부팅 코드를 실행합니다. [4, 5]
- 전력 및 클록 관리: 메인 CPU의 부하를 덜어주기 위해 런타임 시 전력 모니터링, 동적 전압·주파수 스케일링(DVFS), 클록 및 리셋 제어 등의 작업을 전담합니다. [1, 3, 6, 7, 8]
- 하드웨어 구성: 내부적으로 ARM Cortex-R5 코어와 캐시, 전용 메모리 및 주변 장치들로 구성되어 있습니다. [6]

만약 **특정 플랫폼(Jetson, DRIVE 등)**에서의 BPMP펌웨어 설정이나 부팅 흐름에 대해 더 자세히 알고 싶으시다면 말씀해 주세요.

  

[1] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/what-does-bpmp-stand-for/156332)

[2] [https://www.cs.wpi.edu](http://www.cs.wpi.edu/~claypool/mqp/sv/2019/nvidia-bpmp/)

[3] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/text/SD/PlatformPowerAndPerformance/JetsonOrinNxSeriesAndJetsonAgxOrinSeries.html)

[4] [https://developer.nvidia.com](https://developer.nvidia.com/docs/drive/drive-os/6.0.8/public/drive-os-linux-sdk/common/topics/bootloader_setup/UnderstandingtheBootFlowProcess1.html)

[5] [https://www.thegoodpenguin.co.uk](https://www.thegoodpenguin.co.uk/blog/diving-into-the-nvidia-jetson-nano-boot-process/)

[6] [https://developer.nvidia.com](https://developer.nvidia.com/docs/drive/drive-os/7.0.3/public/drive-os-linux-sdk/platform-customization/System_Software_Settings/ConfiguringPoweronBPMPFirmware1.html)

[7] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/l4t/Tegra%20Linux%20Driver%20Package%20Development%20Guide/power_management_jetson_xavier.html)

[8] [https://ghaf.tii.ae](https://ghaf.tii.ae/ghaf/dev/technologies/nvidia_virtualization_bpmp/)

---

결론부터 말씀드리면, ==네, 맞습니다. BR-BCT, PSCBL1, MB1, MB1-BCT와 같은 초기 부팅 바이너리들을 시스템 저장소(QSPI Flash 등)에서 읽어와 AOTZRAM에 로딩하고 실행하는 물리적인 주체는 바로 BPMP 내의 R5 코어==입니다. [1, 2]

이 과정이 하드웨어와 소프트웨어 관점에서 어떻게 일어나는지 구체적으로 풀어서 설명해 드리겠습니다.

## 1. 물리적 연산의 주체는 항상 'BPMP R5'

앞서 설명해 드린 대로 BPMP는 하드웨어 영역(IP)의 이름이고, 그 안에서 부팅 코드를 1줄씩 읽어와 연산하고 하드웨어를 제어하는 실제 CPU 코어는 ARM Cortex-R5(BPMP R5)입니다. 따라서 칩에 전원이 들어오는 순간부터 부팅 가이드 상에 적히는 모든 'BPMP의 동작'은 물리적으로 BPMP R5 코어가 연산 장치로서 수행하는 것입니다.

## 2. 하드웨어 소스 코드가 동작하는 원리 (BootROM 단계)

엔비디아 테그라(Tegra/Jetson Orin 등) SoC가 켜지면 다음과 같은 순서로 진행됩니다.

1. 하드웨어 리셋 해제: 시스템 전원이 켜지면 BPMP R5 코어가 가장 먼저 리셋 상태에서 빠져나옵니다.
2. BootROM(BR) 실행: BPMP R5 코어는 칩 내부에 하드와이어드(Hard-wired) 형태로 고정된 BootROM 영역의 초기 코드를 읽어 실행하기 시작합니다. [1]
3. 저장소 접근 및 로딩: BootROM 코드가 R5 코어 위에서 실행되면서 QSPI Flash, eMMC 같은 부팅 미디어를 초기화합니다. 그리고 그곳에 저장되어 있던 BR-BCT, PSCBL1, MB1, MB1-BCT 파일을 하나씩 읽어와 가속기 및 I/O 컨트롤러를 거쳐 AOTZRAM(Always-On Trust Zone RAM) 영역에 복사(로딩)합니다. [1]
4. 핸드오버(Handover): 로딩이 끝나면 R5 코어는 BootROM 실행을 멈추고(Halt), 방금 AOTZRAM에 올려둔 MB1(Microboot1)의 시작 주소로 점프하여 MB1을 실행하게 됩니다. [1, 2]

## 3. 보안 프로세서(PSC)와의 협업

NVIDIA Orin 및 최신 아키텍처에서는 보안성 강화를 위해 PSC(Platform Security Controller)라는 별도의 독립된 보안 가속기 프로세서가 함께 동작합니다. [3]

- BPMP R5가 저장소에서 바이너리(MB1 등)를 읽어와 AOTZRAM에 배치하는 동안, PSC는 이 바이너리들이 위변조되지 않았는지 검증(Authentication)하고 암호화를 푸는(Decryption) 보안 처리를 나란히 수행합니다. [3, 4]

## 요약

"BPMP 내부의 R5 코어가 칩 내부의 BootROM 프로그램을 탑재하여 작동함으로써, 외부 저장소에 있던 BR-BCT, PSCBL1, MB1, MB1-BCT를 칩 내부의 안전한 메모리인 AOTZRAM에 로딩하고 순차적으로 실행하게 됩니다." [1, 2]

이 초기 부팅 단계(BootROM -> MB1) 이후에 일어나는 메인 CPU(CCPLEX)로의 제어권 이관 과정이나, MB1-BCT를 통한 하드웨어 레지스터 설정에 대해 더 궁금한 점이 있으신가요?

  

[1] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/archives/r35.6.5/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html)

[2] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/archives/r34.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html)

[3] [https://developer.nvidia.com](https://developer.nvidia.com/docs/drive/drive-os/6.0.8/public/drive-os-linux-sdk/common/topics/bootloader_setup/UnderstandingtheBootFlowProcess1.html)

[4] [https://developer.nvidia.com](https://developer.nvidia.com/docs/drive/drive-os/6.0.9/public/drive-os-linux-sdk/common/topics/bootloader_setup/BootROM4.html)