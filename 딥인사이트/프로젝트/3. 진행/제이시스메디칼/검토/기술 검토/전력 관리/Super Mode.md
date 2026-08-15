---
title: Supported Modes and Power Efficiency (Jetson Orin)
source: https://docs.nvidia.com/jetson/archives/r36.5/DeveloperGuide/SD/PlatformPowerAndPerformance/JetsonOrinNanoSeriesJetsonOrinNxSeriesAndJetsonAgxOrinSeries.html
author: NVIDIA
published:
created: 2025-09-07
description: Jetson Orin Nano / Orin NX / AGX Orin 시리즈의 지원 전력 모드 및 전력 효율 정리
tags:
  - jetson
  - orin
  - power-management
  - nvpmodel
---

# Supported Modes and Power Efficiency

## 개요

- Jetson Orin은 고효율 **PMIC(Power Management IC)**, 전압 레귤레이터, 파워 트리로 설계되어 전력 효율을 최적화한다.
- 여러 최적화된 전력 예산(power budget) 을 지원한다 — 예: 10W, 15W, 25W, 30W 등.
- 각 전력 예산(모드)마다 온라인 CPU 코어 수와 CPU·GPU·DLA·PVA·SOC 엔진의 최대 클럭이 다르게 설정된다.
- 전력 모드는 **`nvpmodel`** 유틸리티로 전환한다.

### MAXN / MAXN_SUPER 모드
- **MAXN**: CPU·GPU·DLA·PVA·SOC 엔진의 코어 수와 클럭을 제한 없이(unconstrained) 최대로 허용하는 모드.
- ==단, 실험용(experimental) 모드로 **모든 use case에서 최고 성능을 보장하지 않는다** — 모듈 총 전력이 TDP 예산을 초과하면 하드웨어 스로틀링이 개입하기 때문.==
- **MAXN_SUPER**: "Super" 구성에서 제공되는 확장 성능 모드로, 기존 대비 더 높은 CPU/GPU/DLA 클럭을 허용한다.

## 모듈별 지원 전력 모드

### Jetson Orin Nano 4GB
| 모드 | Mode ID | Online CPU | CPU Max | GPU Max |
|------|---------|-----------|---------|---------|
| 10W (기본값) | 0 | 6 | 1510.4 MHz | 624.75 MHz |
| 7W_AI | 1 | - | - | - |
| 7W_CPU | 2 | - | - | - |
| MAXN_SUPER | - | 6 | 1728 MHz | 1020 MHz |

### Jetson Orin Nano 8GB
| 모드 | Mode ID | Online CPU | CPU Max | GPU Max |
|------|---------|-----------|---------|---------|
| 15W (기본값) | 0 | 6 | 1510.4 MHz | 624.75 MHz |
| 7W | 1 | - | - | - |
| 25W (Super) | 1 | - | - | - |
| MAXN_SUPER | 2 | 6 | 1728 MHz | 1020 MHz |

### Jetson Orin NX 8GB
| 모드 | Mode ID | Online CPU | CPU Max | GPU Max | DLA Max |
|------|---------|-----------|---------|---------|---------|
| MAXN / MAXN_SUPER | 0 | 6 | 1984 MHz | 1173 MHz | 1228.8 MHz |
| 10W | 1 | - | - | - | - |
| 15W (기본값) | 2 | 4 | 1420.8 MHz | 612 MHz | 614.4 MHz |
| 20W | 3 | - | - | - | - |

### Jetson Orin NX 16GB
| 모드 | Mode ID | Online CPU | CPU Max | GPU Max | DLA Max |
|------|---------|-----------|---------|---------|---------|
| MAXN / MAXN_SUPER | 0 | 8 | 1984 MHz | 1173 MHz | 1228.8 MHz |
| 10W | 1 | - | - | - | - |
| 15W (기본값) | 2 | 4 | 1420.8 MHz | 612 MHz | 614.4 MHz |
| 25W | 3 | - | - | - | - |
| 40W | 4 | - | - | - | - |

### Jetson AGX Orin (32GB / 64GB / Industrial)
| 모드 | Mode ID | Online CPU | CPU Max | GPU Max | DLA Max |
|------|---------|-----------|---------|---------|---------|
| MAXN | 0 | 8~12 | 1971.2~2201.6 MHz | 930.75~1301 MHz | 1408~1600 MHz |
| 15W | 1 | - | - | - | - |
| 30W (기본값) | 2 | 8 | 1728 MHz | 612 MHz | 1369.6 MHz |
| 40W / 50W / 60W | 3 | - | - | - | - |

> 모든 구성에서 Display, NVDEC, NVENC 등 일부 SOC 엔진의 최대 클럭은 모드와 무관하게 동일하게 유지된다.

> ⚠️ 표의 세부 수치(코어 수·클럭)는 문서 버전(r36.5) 및 모듈 리비전에 따라 달라질 수 있으므로, 실제 적용 전 [원문 표](https://docs.nvidia.com/jetson/archives/r36.5/DeveloperGuide/SD/PlatformPowerAndPerformance/JetsonOrinNanoSeriesJetsonOrinNxSeriesAndJetsonAgxOrinSeries.html#sd-platformpowerandperformance-supportedmodesandpowerefficiency)에서 정확한 값을 확인할 것.
