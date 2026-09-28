---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
엔비디아(NVIDIA) 시스템 아키텍처 및 보안 부팅(Secure Boot) 프로세스에서 **PSCROM**과 **KROM**은 ==칩(SoC) 내부의 보안을 담당하는 핵심 하드웨어 컴포넌트이자 메모리 영역==입니다. 이 둘은 시스템이 켜질 때 최초로 실행되는 신뢰점(Root of Trust) 역할을 수행합니다. [[1](https://docs.nvidia.com/jetson/archives/r35.6.5/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html), [2](https://developer.nvidia.com/docs/drive/drive-os/6.0.5/public/drive-os-linux-sdk/common/topics/security_concepts/SecureBoot16.html), [3](https://docs.nvidia.com/jetson/archives/r39.2/DeveloperGuide/SD/Security/FirmwareTPM/Provisioning.html)]

두 개념의 주요 역할과 차이점은 다음과 같습니다.

1. PSCROM (Platform Security Controller ROM)

- **역할**: 프로세서가 리셋(부팅)되는 즉시 구동을 시작하는 하드웨어 컴포넌트입니다.

- **기능**: 엔비디아 및 OEM 인증·복호화에 필요한 키를 관리하며, BootROM에 인증 서비스를 제공합니다. 다음 부팅 단계인 MB1(Memory Boot 1)이나 PSC-BL1 단계가 안전한지 검증(Audit)하는 역할을 맡습니다.

- **성격**: 고정되어 수정할 수 없는 읽기 전용 메모리(ROM) 형태의 보안 제어 코드입니다. [[1](https://developer.ridgerun.com/wiki/index.php/RidgeRun_Platform_Security_Manual/Platform_Security/Secure_Boot), [2](https://docs.nvidia.com/jetson/archives/r35.6.5/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html), [3](https://docs.nvidia.com/jetson/archives/r39.2/DeveloperGuide/SD/Security/FirmwareTPM/Provisioning.html)]

2. KROM (Key ROM)

- **역할**: 칩 내부에서 **엔비디아의 암호화 키(Keys)들을 안전하게 보관**하기 위해 별도로 할당된 전용 메모리 영역입니다. [[1](https://developer.nvidia.com/docs/drive/drive-os/6.0.5/public/drive-os-linux-sdk/common/topics/security_concepts/SecureBoot16.html)]

- **기능**: KROM에 저장된 엔비디아 키들은 외부에서 직접 읽을 수 없으며, 오직 앞서 언급한 **PSCROM에 의해서만 암호화된 형태로 읽을 수 있도록** 철저히 격리되어 보호됩니다. [[1](https://developer.nvidia.com/docs/drive/drive-os/6.0.5/public/drive-os-linux-sdk/common/topics/security_concepts/SecureBoot16.html)]

- **엔비디아 보안 표준**: 이 영역과 연계되어 엔비디아의 모든 바이너리 인증에는 3072-bit RSA 알고리즘이, 암호화에는 AES-GCM(256-bit) 알고리즘이 표준으로 사용됩니다. [[1](https://developer.nvidia.com/docs/drive/drive-os/6.0.5/public/drive-os-linux-sdk/common/topics/security_concepts/SecureBoot16.html)]

---

**추가로 궁금한 점이 있으신가요?**

- 엔비디아 젯슨(Jetson)이나 드라이브(DRIVE) 플랫폼의 **전체 부팅 프로세스(Boot Flow)**가 궁금하신가요?

- **보안 부팅(Secure Boot)** 과정에서 암호화 키가 어떻게 검증되는지 알고 싶으신가요? [[1](https://docs.nvidia.com/jetson/archives/r34.1/DeveloperGuide/text/AR/BootArchitecture.html), [2](https://developer.nvidia.com/docs/drive/drive-os/6.0.5/public/drive-os-linux-sdk/common/topics/security_concepts/SecureBoot16.html)]