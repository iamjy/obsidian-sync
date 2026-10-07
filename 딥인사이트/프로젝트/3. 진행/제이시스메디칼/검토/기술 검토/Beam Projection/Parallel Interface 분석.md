---
title: 
source: TI DLPC347x Datasheet
author: 
published: 
created: 2025-09-07
description: Parallel Interface Frame Timing Requirements 검토
tags: #TI #DLPC347x #TimingAnalysis
---
![[Pasted image 20261007131012.png]]
![[Pasted image 20261007131041.png]]
![[Pasted image 20261007131104.png]]

![[Pasted image 20261007135637.png]]

## 1. $t_{p\_tvb}$ 정의 및 최소값 계산 공식
TI DLPC347x 데이터시트 Section 5.12 (Parallel Interface Frame Timing Requirements) 기준입니다.
- **정의**: $t_{p\_tvb} = t_{p\_vbp} + t_{p\_vfp}$ (Vertical Back Porch와 Vertical Front Porch의 합, 단위: lines)
- **최소값 산출 수식**:
    $$t_{p\_tvb}(\text{min}) = 6 + \left[ 8 \times \max\left(1, \frac{\text{SOURCE\_ALPF}}{\text{DMD\_ALPF}}\right) \right] \text{ lines}$$
    - $\text{SOURCE\_ALPF}$: 프레임당 입력 소스의 유효 라인 수 (Input source active lines per frame)
    - $\text{DMD\_ALPF}$: 프레임당 실제 사용되는 DMD의 유효 라인 수 (Actual DMD used lines per frame supported)

## 2. 해상도별 계산 예시 및 검증

### Case 1: 1:1 매칭 해상도 (예: 720p 입력 $\rightarrow$ 720p DMD)
- **조건**: $\text{SOURCE\_ALPF} = 720$, $\text{DMD\_ALPF} = 720$
- **계산 과정**:
    1. 비율 계산: $720 / 720 = 1$
    2. $\max(1, 1) = 1$
    3. 공식 대입: $t_{p\_tvb}(\text{min}) = 6 + (8 \times 1) = \mathbf{14 \text{ lines}}$
- **결과**: 전체 수직 블랭킹 합은 최소 14 라인 이상이어야 함.

### Case 2: 다운스케일링 입력 (예: 1080p 입력 $\rightarrow$ 720p DMD)
- **조건**: $\text{SOURCE\_ALPF} = 1080$, $\text{DMD\_ALPF} = 720$
- **계산 과정**:
    1. 비율 계산: $1080 / 720 = 1.5$
    2. $\max(1, 1.5) = 1.5$
    3. 공식 대입: $t_{p\_tvb}(\text{min}) = 6 + (8 \times 1.5) = 6 + 12 = \mathbf{18 \text{ lines}}$
- **결과**: 전체 수직 블랭킹 합은 최소 18 라인 이상이어야 함.

## 3. 타이밍 파라미터 제약 조건 및 분배 예시

### 개별 항목 최소 규격
- $t_{p\_vbp}$ (Vertical Back Porch): $\ge 2 \text{ lines}$
- $t_{p\_vfp}$ (Vertical Front Porch): $\ge 1 \text{ lines}$
- $t_{p\_vsw}$ (VSYNC Pulse Duration): $\ge 1 \text{ lines}$

### 적용 예시 (Case 1, 총 14 lines 기준)
- **설정값**: $t_{p\_vbp} = 10 \text{ lines}$, $t_{p\_vfp} = 4 \text{ lines}$
- **검증**:
    1. $t_{p\_vbp} (10) \ge 2$ $\rightarrow$ 충족
    2. $t_{p\_vfp} (4) \ge 1$ $\rightarrow$ 충족
    3. $t_{p\_tvb} (10 + 4 = 14) \ge 14$ $\rightarrow$ 충족

> **주의사항**: Figure 5-7에 따라 $t_{p\_vbp}$는 `VSYNC_WE` 시작부터 첫 액티브 라인의 `HSYNC_CS`까지의 구간입니다. 따라서 $t_{p\_vbp}$ 내에 VSYNC 펄스 폭($t_{p\_vsw}$)이 포함되어야 하므로, 반드시 $t_{p\_vbp} \ge t_{p\_vsw}$ 관계가 성립해야 합니다.

## 참고 자료
- **TI DLPC347x Datasheet**: Section 5.12 Parallel Interface Frame Timing Requirements & Figure 5-7.

![[Pasted image 20261007140429.png]]

## 4. PCLK 클록 지터(Clock Jitter) 분석
TI DLPC347x 데이터시트 Section 5.13 (Parallel Interface General Timing Requirements) 기준입니다.
### 4.1 계산 공식 및 파라미터 정의
- **계산 공식**:

    $$\text{Jitter} = \left[ \frac{1}{f_{clock}} - 5.76\text{ ns} \right]$$

    - $f_{clock}$: 입력 PCLK의 동작 주파수 (Operating Frequency)

    - $\frac{1}{f_{clock}}$: 입력 PCLK의 1주기 시간($t_{p\_clkper}$, 단위: ns)

    - $5.76\text{ ns}$: 내부 회로의 펄스 폭 및 셋업/홀드 동작 보장을 위해 차감되는 고정 오프셋 시간 상수
- **추가 제약 조건**: 지터 발생 시에도 데이터 신호의 셋업 타임($t_{p\_su} \ge 0.9\text{ ns}$)과 홀드 타임($t_{p\_h} \ge 0.9\text{ ns}$) 조건을 반드시 만족해야 합니다.
### 4.2 구체적인 계산 예시
#### Case 1: 최대 지원 주파수 동작 시 ($f_{clock} = 155.0\text{ MHz}$)
1. **클록 주기($T$) 계산**: $T = \frac{1}{155.0 \times 10^6\text{ Hz}} \approx 6.4516\text{ ns}$
2. **공식 대입**: $\text{Jitter} = 6.4516\text{ ns} - 5.76\text{ ns} = \mathbf{0.6916\text{ ns}} \approx \mathbf{0.69\text{ ns}}$ (약 692 ps)
- **결과**: $155\text{ MHz}$ 구동 시 허용 지터는 최대 약 0.69 ns 이하로 엄격하게 제한됩니다.
#### Case 2: 일반적인 720p 60Hz 동작 시 ($f_{clock} = 74.25\text{ MHz}$)
1. **클록 주기($T$) 계산**: $T = \frac{1}{74.25 \times 10^6\text{ Hz}} \approx 13.468\text{ ns}$
2. **공식 대입**: $\text{Jitter} = 13.468\text{ ns} - 5.76\text{ ns} = \mathbf{7.708\text{ ns}} \approx \mathbf{7.71\text{ ns}}$
- **결과**: 주파수가 낮아짐에 따라 타이밍 마진이 확보되어, 허용 지터 상한선은 최대 약 7.71 ns로 완화됩니다.
### 4.3 타이밍 설계 시 고려사항
- **펄스 폭 마진 확인**: PCLK의 High 및 Low 펄스 폭($t_{p\_wh}, t_{p\_wl}$)은 각각 최소 $2.43\text{ ns}$ 이상을 유지해야 합니다. 지터로 인해 어느 한쪽 반주기가 $2.43\text{ ns}$ 미만으로 떨어질 경우 비정상 래치(Latch) 에러가 발생할 가능성이 있습니다.
### 추가 참고 자료
- **TI DLPC347x Datasheet**: Section 5.13 Parallel Interface General Timing Requirements (PCLK 동작 범위 1.0~155.0 MHz, 셋업/홀드 시간 0.9 ns, 각주 (1) 지터 계산 공식).