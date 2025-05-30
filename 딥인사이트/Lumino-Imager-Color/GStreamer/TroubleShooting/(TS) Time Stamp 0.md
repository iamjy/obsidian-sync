---
title: Time stamp trouble shooting
source: 
author: 
published: 
created: 
description: 
tags:
---
---

`self.encoder.set_property("key-int-max", 30)` 설정은 **최대 30프레임마다 꼭 키프레임이 생성**되도록 하여,  

- **키프레임 간격이 짧아져**
- `splitmuxsink`가 **적절한 위치에서 분할할 수 있는 지점을 더 자주 확보**하게 되며
- 결과적으로 **파일 경계 인근에서 프레임 손실 가능성을 최소화**

하지만,

---

### 주의  
- 이것만으로 **"완벽한 무손실"을 *보장*하지는 않습니다.**
- 이유:  
  - 인코더 구현 세부에 따라 강제하지 않을 수 있고  
  - **하드웨어 인코더(예: `v4l2h264enc`)**는 `key-int-max` 무시하거나 다르게 작동할 수 있음  
  - 시스템 버퍼, 디스크 I/O 속도 문제  
  - GStreamer 내부 `splitmuxsink` 정책 상 약간의 지연 누적 등 때문
- **내부 버퍼 처리, IO 예외 및 키프레임 인터벌 이외의 변수**가 항상 존재  

---

### 결론
- `key-int-max=30` 설정은  
  **프레임 누락 *가능성은 대폭 줄이지만***,  
  **절대적인 완벽 무손실을 보장하지는 않음.**
- **분할 손실 최소화를 위한 중요한 조치임은 맞음.**

---

### 권장
- 무손실에 가깝게 하려면  
  - `send-keyframe-requests` 활성화
  - `tune=zerolatency`  
  - `key-int-max` 값 낮추기
- 그리고 최종적으로 **녹화 파일 타임스탬프와 내용물 비교, 검증 과정이 필요**합니다.