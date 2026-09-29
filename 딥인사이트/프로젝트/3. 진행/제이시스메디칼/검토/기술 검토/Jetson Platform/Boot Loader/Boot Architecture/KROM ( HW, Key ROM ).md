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

---

**PSC-ROM**과 **KROM(Key ROM)**은 ==NVIDIA Jetson(Orin 등) 및 DRIVE 하드웨어 플랫폼의 초전기(Root of Trust, RoT) 보안 부팅을 담당하는 핵심 내부 요소==입니다. [[1](https://developer.nvidia.com/docs/drive/drive-os/archives/6.0.3/linux/sdk/oxy_ex-1/common/topics/security_concepts/SecureBoot16.html), [2](https://docs.nvidia.com/jetson/archives/r36.5/DeveloperGuide/SD/Security/FirmwareTPM.html)]

이 두 시스템은 OEM(제조사)이 NVIDIA의 서명 권한이나 암호화 정보를 함부로 덮어쓰거나 수정할 수 없도록 철저히 격리되어 작동합니다. [[1](https://developer.nvidia.com/docs/drive/drive-os/archives/6.0.3/linux/sdk/oxy_ex-1/common/topics/security_concepts/SecureBoot16.html)]

1. KROM (Key ROM)

- **역할:** NVIDIA 하드웨어 칩 내부에 내장된 **NVIDIA 전용 암호화 키 저장소(영역)**입니다.

- **보안 격리:** 외부에서 임의로 오버라이트(덮어쓰기)할 수 없도록 물리적·논리적으로 보호됩니다.

- **접근 제어:** KROM 내에 보관된 키들은 오직 **PSC-ROM에 의해서만 암호화된 형태로 읽을 수 있습니다**. [[1](https://developer.nvidia.com/docs/drive/drive-os/archives/6.0.3/linux/sdk/oxy_ex-1/common/topics/security_concepts/SecureBoot16.html)]

2. PSC-ROM (Platform Security Controller ROM)

- **역할:** 시스템 내부의 플랫폼 보안 컨트롤러(PSC)에 내장된 고정 불변의 하드웨어 ROM 코드입니다. 프로세서가 리셋(부팅)되는 즉시 구동을 시작합니다. [[1](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/AR/BootArchitecture/JetsonThorBootFlow.html)]

- **핵심 기능:** NVIDIA 및 OEM의 인증·복호화에 필요한 모든 마스터 키들을 관리하고 통제합니다. [[1](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/AR/BootArchitecture/JetsonThorBootFlow.html)]

- **보안 서비스 제공:** 1차 부팅 코드인 BootROM에 암호화·복호화 및 인증 서비스를 제공하며, 다음 부팅 단계인 **MB1**(Microbootloader 1)이나 **PSC-BL1**이 올바른 바이너리인지 무결성을 검증(Audit)합니다. [[1](https://docs.nvidia.com/jetson/archives/r39.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html)]

주요 보안 알고리즘 기본 규격

NVIDIA는 보안 가이드라인에 따라 바이너리 인증 시 혼선을 줄이기 위해 단일 표준 알고리즘을 강제합니다.

- **인증(Authentication):** **3072-bit RSA** 알고리즘 사용

- **인증된 암호화(Authenticated Encryption):** **AES-GCM (256-bit)** 알고리즘 사용

---

**추가 정보 제안**  
현재 이 기능들을 활용해 하드웨어의 **보안 부팅(Secure Boot)**을 직접 구현 중이신가요? 혹은 특정 **Jetson/DRIVE SoC 모델**(예: Orin, Thor)의 초기 부팅 플로우나 **퓨즈(Fuse) 설정**에 대해 찾고 계시는지 알려주시면 상세 정보를 더 안내해 드릴 수 있습니다. [[1](https://docs.nvidia.com/jetson/archives/r36.5.2/DeveloperGuide/AR/BootArchitecture/JetsonOrinSeriesBootFlow.html), [2](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/AR/BootArchitecture/JetsonThorBootFlow.html)]