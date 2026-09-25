---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
CCPLEX(CPU Cluster Complex)는 ==NVIDIA의 테그라(Tegra) 및 오린(Orin) SoC(시스템 온 칩)에서 메인 CPU 코어들이 모여 있는 핵심 클러스터 집합==을 의미합니다.

BPMP가 부팅과 전력을 관리하는 '보조 프로세서' 역할을 수행한다면, CCPLEX는 실제 운영체제(리눅스, 안드로이드 등)와 고부하 연산 애플리케이션이 실행되는 '메인 CPU'입니다.

## 💡 핵심 특징

- **기능**: 리눅스, QNX, 안드로이드 등의 주 운영체제(OS)가 구동됩니다.
- **구성**: 플랫폼에 따라 NVIDIA 자체 개발 ARM 기반 코어(예: Carmel) 또는 ARM Cortex-A 시리즈(예: Cortex-A78AE) 코어 여러 개가 클러스터로 묶여 있습니다.
- **상호 연결**: 각 코어는 고속 내부 버스(Interconnect)와 L3 캐시를 공유하며 하나의 거대한 CPU 복합체를 형성합니다.
==NVIDIA DRIVE Orin 및 Jetson Orin== SoC의 CCPLEX는 ARM Cortex-A78AE 코어로 구성된 강력한 메인 CPU 클러스터입니다. 특히 차량용 안전 표준(ISO 26262)을 충족하는 'AE(Automotive Enhanced)' 코어를 사용하여 자율주행 및 로봇 공학의 핵심 연산을 처리합니다.

## 📊 Orin 라인업별 CPU(CCPLEX) 비교

| 제품명 | CPU 코어 수 | 아키텍처 | L3 캐시 용량 |
| :--- | :---: | :---: | :---: |
| AGX Orin (64GB) | 12코어 | ARM Cortex-A78AE | 6 MB |
| AGX Orin (32GB) | 8코어 | ARM Cortex-A78AE | 4 MB |
| Orin NX (16GB) | 8코어 | ARM Cortex-A78AE | 4 MB |
| Orin NX (8GB) | 6코어 | ARM Cortex-A78AE | 3 MB |
| Orin Nano (8GB/4GB) | 6코어 | ARM Cortex-A78AE | 1.5 MB |

## ⚙️ Orin에서 BPMP와 CCPLEX의 협업 흐름

1. **부팅 단계**: 전원 인가 시 BPMP가 우선 기동하여 클록 및 전원을 초기화하고, 펌웨어 로드 후 CCPLEX(Cortex-A78AE)를 깨웁니다.
2. **런타임 단계**: CCPLEX에서 OS가 구동되는 동안 메인 CPU가 주파수 변경을 요청(DVFS)하면, BPMP가 백그라운드에서 하드웨어 전압을 안전하게 조절합니다.