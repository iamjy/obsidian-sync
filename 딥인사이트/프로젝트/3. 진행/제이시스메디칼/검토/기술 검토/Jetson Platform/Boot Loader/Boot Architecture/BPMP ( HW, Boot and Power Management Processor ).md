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