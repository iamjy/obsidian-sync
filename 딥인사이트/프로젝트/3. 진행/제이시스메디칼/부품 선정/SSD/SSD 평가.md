---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
온보드 eMMC가 없는 NVIDIA Jetson Orin NX 및 Orin Nano 모듈에서 메인 부팅 드라이브로 사용할 M.2 NVMe SSD를 선정하기 위한 기술적 평가 기준

- Orin NX/Nano는 QSPI-NOR 부트로더 이후 PCIe를 통해 NVMe에서 직접 OS를 적재하므로, UEFI/커널 호환성, 전원 관리(ASPM/APST) 안정성, 발열 및 단면 실장 기구 적합성, 전원 차단(SPO) 내성, 그리고 양산 공급망(Fixed BOM)의 5가지 핵심 기준으로 평가해야 한다.

## 부팅용 NVMe SSD 핵심 평가 기준
