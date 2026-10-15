---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
# Warm Boot (웜 부팅)

**NVIDIA Jetson의 Warm Boot**란 ==시스템의 **전원 공급을 완전히 차단하지 않은(메인 전원이 살아있는) 상태에서 시스템을 재시작하는 과정**==을 의미합니다. [[1](https://www.thegoodpenguin.co.uk/blog/diving-into-the-nvidia-jetson-nano-boot-process/), [2](https://www.lenovo.com/us/en/glossary/cold-boot/?srsltid=AU7gw4WKQsLbAF6mLRnBW_iR_BF0O8X9UFm3oe8ywzKoJNU5bwekW5iN), [3](https://yeonsozzz.tistory.com/101), [4](https://aliencoder.tistory.com/110)]
### 📌 트리거 조건
- 하드웨어 리셋 버튼 클릭
- OS 내 `sudo reboot` 명령어 실행
- 시스템이 딥 슬립(Deep Sleep, SC7 상태)에서 깨어날 때

---

### 🛠 주요 기술적 특징

Cold Boot와 비교했을 때 Warm Boot는 다음과 같은 차별점이 있습니다. [[1](https://testbook.com/question-answer/what-is-the-fundamental-difference-between-a--68678e686f3a7bbaeab127b5), [2](https://www.lenovo.com/us/en/glossary/cold-boot/?srsltid=AU7gw4WKQsLbAF6mLRnBW_iR_BF0O8X9UFm3oe8ywzKoJNU5bwekW5iN)]
- **하드웨어 초기화 생략**
    - Cold Boot는 BootROM이 하드웨어 진단(POST), 전원 관리 칩(PMIC) 설정, RAM 메모리 컨트롤러 등을 완전히 초기화합니다.
    - 반면 Warm Boot는 이미 전원이 공급되어 안정화된 상태이므로 이 과정의 상당수를 생략합니다. [[1](https://translate.google.com/translate?u=https://docs.nvidia.com/jetson/archives/r34.1/DeveloperGuide/text/AR/BootArchitecture/JetsonAgxOrinBootFlow.html&hl=ko&sl=en&tl=ko&client=sge), [2](https://translate.google.com/translate?u=https://docs.nvidia.com/jetson/archives/r35.6.5/DeveloperGuide/AR/BootArchitecture/JetsonXavierNxAndJetsonAgxXavierBootFlow.html&hl=ko&sl=en&tl=ko&client=sge), [3](https://www.lenovo.com/us/en/glossary/cold-boot/?srsltid=AU7gw4WKQsLbAF6mLRnBW_iR_BF0O8X9UFm3oe8ywzKoJNU5bwekW5iN), [4](https://www.difference.wiki/cold-booting-vs-warm-booting/)]
- **빠른 부팅 속도**
    - 하드웨어 점검 및 초기화 단계를 건너뛰기 때문에 OS 진입까지의 시간이 훨씬 짧습니다. [[1](https://www.lenovo.com/us/en/glossary/cold-boot/?srsltid=AU7gw4WKQsLbAF6mLRnBW_iR_BF0O8X9UFm3oe8ywzKoJNU5bwekW5iN), [2](https://www.difference.wiki/cold-booting-vs-warm-booting/)]
- **전용 바이너리 사용**
    - 부팅 경로에 따라 다른 펌웨어가 작동합니다. (예: Jetson Nano의 경우 일반 부팅용 `nvtboot.bin` $\leftrightarrow$ 웜 부팅 전용 `warmboot.bin`) [[1](https://translate.google.com/translate?u=https://www.thegoodpenguin.co.uk/blog/diving-into-the-nvidia-jetson-nano-boot-process/&hl=ko&sl=en&tl=ko&client=sge), [2](https://www.thegoodpenguin.co.uk/blog/diving-into-the-nvidia-jetson-nano-boot-process/)]
### 📊 Cold Boot vs Warm Boot 비교
| 구분 | Cold Boot (콜드 부팅) | Warm Boot (웜 부팅) |
| :--- | :--- | :--- |
| **트리거 방식** | 전원 플러그 연결, 전원 버튼 켜기 | `reboot` 명령, 리셋 핀 트리거, 슬립 해제 |
| **전원 상태** | 완전히 차단 $\rightarrow$ 다시 공급됨 | 메인 전원(메모리 등)이 계속 유지됨 |
| **하드웨어 검사** | BootROM 단계부터 전체 초기화 수행 | 레지스터 및 하드웨어 초기화 상당수 생략 |
| **RAM 상태** | 데이터가 완전히 소멸 및 초기화됨 | 기존 데이터나 잔류 전하가 영향을 줄 수 있음 |

---

### ⚠️ 임베디드 개발 시 주의사항

> [!CAUTION] **Warm Boot 먹통 현상 (Residual State)**
> 커스텀 캐리어 보드 설계나 드라이버 개발 시, **Cold Boot에서는 정상 작동하나 Warm Boot(재부팅) 시에만 특정 센서, PCIe 장치, 디스플레이가 동작하지 않는 현상**이 자주 발생합니다.

- **원인**: 
    1. 시스템 전원이 유지되어 주변 장치가 이전의 에러 상태(**Residual state**)를 그대로 가지고 있는 경우.
    2. 웜 부팅 시 부트로더가 레지스터를 깨끗하게 리셋하지 못하는 하드웨어/드라이버 이슈. [[1](https://forums.developer.nvidia.com/t/kexec-no-display-out-with-warm-boot/362046), [2](https://forums.developer.nvidia.com/t/jetson-nano-power-up-issues-over-temperature/187983), [3](https://forums.developer.nvidia.com/t/recovery-mode-cold-reset-or-warm-reset/365791), [4](https://f.osdev.org/viewtopic.php?t=20851)]
- **권장 설계**: 안정성이 최우선인 산업용 장비에서는 Warm Boot 대신 전원을 완전히 차단 후 켜는 **Cold Reset(전원 사이클링) 방식**을 권장합니다. [[1](https://forums.developer.nvidia.com/t/recovery-mode-cold-reset-or-warm-reset/365791)]
---
> [!QUESTION] **도움이 필요하신가요?**
> 현재 Jetson 장비에서 **Warm Boot 관련 트러블슈팅(예: 재부팅 시 먹통 현상)**을 겪고 계시거나 **부팅 속도 최적화**를 진행 중이신가요? 구체적인 상황을 알려주시면 알맞은 해결 방법을 안내해 드리겠습니다. [[1](https://docs.nvidia.com/jetson/archives/r35.6.1/DeveloperGuide/SD/BootTimeOptimization.html), [2](https://forums.developer.nvidia.com/t/jetson-nano-power-up-issues-over-temperature/187983)]