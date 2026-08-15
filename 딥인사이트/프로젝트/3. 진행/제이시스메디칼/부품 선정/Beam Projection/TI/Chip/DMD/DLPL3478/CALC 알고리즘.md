---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---

CAIC(Content Adaptive Illumination Control) 알고리즘은 **입력 영상 콘텐츠를 실시간 분석해 RGB LED의 전류를 자동 조절**하여 화질을 유지하면서 전력을 절감하는 기술이다. 주로 프로젝터 및 디스플레이 시스템의 전력 관리 제어기로 구동된다. [[1](https://www.ti.com/lit/pdf/dlpa063), [2](https://www.ti.com/lit/DLPA119), [3](https://www.embeddedrelated.com/Documents/Pico_Design.pdf), [4](https://docs.ampnuts.ru/ti.com.datasheet/DLPC3439/User_guide_DLPU035.PDF)]

주요 동작 원리

- **프레임별 분석**: 입력되는 영상의 밝기 및 콘텐츠 특성을 프레임 단위로 실시간 감지한다.

- **전류 변조(Modulation)**: 디스플레이 컨트롤러가 PMIC로 보내는 전류 제어(IDAC) 값 자동 조절.

- **모드 선택**: 일정한 밝기를 유지하며 전력 소비를 줄이거나, 일정한 전력을 유지하며 화질을 극대화하도록 설정 가능. [[1](https://www.ti.com/lit/DLPA119), [2](https://www.embeddedrelated.com/Documents/Pico_Design.pdf)]

주요 이점

- **전력 소비 절감**: 어두운 영역이나 평균 화화면 밝기(APL)가 낮은 콘텐츠에서 LED 구동 전력을 크게 낮춤.

- **대비(Contrast) 개선**: 영상의 전체적인 명암비와 Full-On/Full-Off 성능 향상.

- **자동화**: 수동으로 설정하던 LED 전류 제어 방식을 콘텐츠 적응형으로 자동화. [[1](https://www.ti.com/lit/DLPA119)]