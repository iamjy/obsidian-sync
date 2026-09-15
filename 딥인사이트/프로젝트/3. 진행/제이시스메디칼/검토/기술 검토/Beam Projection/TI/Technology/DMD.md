---
title: How To Display Video & Control Light Using TI DLP® Technology
source:
author:
published:
created: 2025-09-07
description:
tags:
---
## 1. DMD 개요 및 물리적 특성 (Overview & Physical Properties)
- **구성**: 수백만 개의 미세한 반사형 알루미늄 거울로 구성 (최대 800만 개 이상).
- **크기**: 각 거울의 폭은 수 마이크론($\mu m$) 단위.
- **제어 방식**: 정전기적 제어를 통해 초당 수천 번 거울을 기울이거나 전환하여 빛의 방향을 스티어링(Steer).

## 2. 이미지 생성 원리 (Image Generation)
- **기본 메커니즘**: 디지털 비디오 신호와 결합하여 이미지를 생성.
- **픽셀 표현**:
	- **White Pixel**: 빛을 투사 광학계(Projection Optics) 방향으로 반사.
	- **Black Pixel**: 빛을 흡수체(Light Absorber) 방향으로 반사.
	- **Grayscale**: 각 프레임 내에서 빛이 투사 경로로 향하는 시간의 길이에 따라 명암 결정.
- **호환성**: 램프, LED, 레이저 등 다양한 광원 및 가시광선, 적외선(IR), 자외선(UV) 파장 제어 가능.

## 3. 색상 구현 방식 (Color Implementation)
### A. 컬러 휠 방식 (Color Wheel System)
- **구조**: 광원과 DMD 사이에 회전하는 컬러 휠 배치.
- **작동**: RGB 빛이 순차적으로 거울 어레이에 집중될 때, 거울의 전환 타이밍을 휠과 동기화하여 색상 표현. (예: 빨강+초록 $\rightarrow$ 노란색)

### B. 직접 발광 방식 (Direct Colored Source)
- **구조**: 컬러 휠 없이 유색 LED 또는 레이저 광원 사용.
- **특징**: 거울의 전환 속도가 매우 빠르기 때문에 가능하며, 10억 가지 이상의 색상 표현 가능.

## 4. 산업적 응용 (Industrial Applications)
DMD의 고속 픽셀 데이터 처리 능력을 활용하여 다음과 같은 솔루션에 적용:
- **3D 스캐닝**
- **3D 프린팅**
- **센싱 솔루션**

---
**(Original Text)**
Each mirror has a reflective aluminum surface that can be just a few microns wide. DMDs vary in resolution and size and can contain over 8 milion mirros. A DMD steer light by electrostatically deflecting or switching each mirror up to thousands of times per second. An image is created when a DMD is combined with a digital video signal. The technology can be used with virtually any light source, including lamps, LEDs, lasers, and laser phosphor, and the DMD can steer many types of light, including visible, infrared, and ultraviolet wavelengths, as a light source illuminates the DMD. Each mirror will either reflect light through the projection optics to display a white pixel, or towards the light absorber to display a black pixel. Grayscale shades are determined by the length of time light is steered towards the projection path during each frame. In some systems, color is introduced by placing a color wheel between the light source and the  DMD. As the color wheel spins, it focused red, green, and blue light on the mirror array. When the timing of each mirror is synchronized with the wheel, a shaded of color is displayed. For example, a yellow pixel is displayed by turninga mirror toward the projection path when red and green light is landing on it. In other systems, colored LEDs or lasers can be used to display video without a color wheel because the mirrors can switch so fast. A projection system based on DLP technology can display over 1 bilion colors. And in industrial applications, the DMDs high speed pixel data rates are used for incredibly fast. 3D scanning, printing, and sensing solutions. TI's DLP technology continues to be trusted by innovators, retailers and consumers in a wide range of display, industrial and automotive solutions all around the world.

각 거울은 폭이 불과 몇 마이크론에 불과한 반사형 알루미늄 표면을 갖추고 있습니다. DMD는 해상도와 크기가 다양하며, 800만 개 이상의 거울을 포함할 수 있습니다. DMD는 각 거울을 초당 수천 번까지 정전기적으로 기울이거나 전환함으로써 빛의 방향을 제어합니다. DMD가 디지털 비디오 신호와 결합되면 이미지가 생성됩니다. 이 기술은 램프, LED, 레이저, 레이저 형광체 등 거의 모든 광원과 함께 사용할 수 있으며, 광원이 DMD를 비출 때 가시광선, 적외선, 자외선 파장 등 다양한 유형의 빛을 제어할 수 있습니다. 각 거울은 빛을 투사 광학계로 반사하여 흰색 픽셀을 표시하거나, 빛 흡수체 쪽으로 반사하여 검은색 픽셀을 표시합니다. 그레이스케일(회색조) 명암은 각 프레임 동안 빛이 투사 경로로 향하는 시간에 따라 결정됩니다. 일부 시스템에서는 광원과 DMD 사이에 컬러 휠을 배치하여 색상을 구현합니다. 컬러 휠이 회전하면서 빨간색, 초록색, 파란색 빛을 거울 어레이에 집중시킵니다. 각 거울의 작동 타이밍을 휠의 회전과 동기화하면 특정 색상이 표시됩니다. 예를 들어, 빨간색과 초록색 빛이 거울에 비칠 때 해당 거울을 투사 경로 쪽으로 향하게 하면 노란색 픽셀이 표시됩니다. 거울의 전환 속도가 매우 빠르기 때문에, 다른 시스템에서는 컬러 휠 없이 유색 LED나 레이저를 사용하여 영상을 표시할 수도 있습니다. DLP 기술 기반의 투사 시스템은 10억 가지 이상의 색상을 표현할 수 있습니다. 또한 산업 분야에서는 DMD의 고속 픽셀 데이터 처리 능력을 활용하여 초고속 3D 스캐닝, 프린팅 및 센싱 솔루션을 구현합니다. TI의 DLP 기술은 전 세계의 다양한 디스플레이, 산업 및 자동차 솔루션 분야에서 혁신가, 소매업체, 소비자들로부터 지속적인 신뢰를 받고 있습니다.