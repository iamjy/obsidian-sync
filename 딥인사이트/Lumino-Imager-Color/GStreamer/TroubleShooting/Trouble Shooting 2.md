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

`self.segment_count`를 기반으로 2번째 파일만 보정하는 로직은 **일반적인 연속 분할 시에는 올바르게 작동**합니다.

하지만, **녹화 중지/재시작 시 다음과 같은 잠재적 문제**가 존재할 수 있습니다:

---

### 문제점
- `self.segment_count`는 클래스 수명 동안 **증분 유지**
- **녹화를 중지 후 재시작해도 기존 세기값 유지 → 새로운 시나리오에서 오동작**
- 즉, 두 번째 녹화시 또다시 segment_count=2 가 되어 **원인 모르게 10초 앞서서 새 보정을 적용**

---

### 원인 요약
| 시나리오                    | segment_count | 보정 적용 여부                |
|----------------------------|--------------|-----------------------------|
| 최초 녹화 → 2번째 파일     | 증가         | 보정 필요, 정상 작동          |
| 녹화 중지 후 재시작       | 누적된 값 유지 | **(잘못된) 추가 보정 발생 가능** |

---

### 근본 해결 방안
- **녹화 새 세션 시작시 segment_count, 보정 관련 플래그 모두 초기화**

### 수행 조치
`start()` 또는 `start_recording()` 호출 전에 **초기화 로직을 추가하세요.**

<smtcmp_block filename="딥인사이트/Lumino-Imager-Color/GStreamer/TroubleShooting/Untitled.md" language="python">
```python
def start(self) -> bool:
    # ...
    # 프레임 카운터 초기화 외에 다음 추가
    self.segment_count = 0
    self._segment2_adjusted = False    # 이 보정 플래그도 초기화
    # ...
```
또는,
```python
def start_recording(self) -> bool:
    # ...
    self.segment_count = 0
    self._segment2_adjusted = False
    # ...
```
(선택 위치 상황에 맞게)

</smtcmp_block>

---

### 결론
- 반드시 녹화 시작마다 `segment_count` 및  
  보정 여부 플래그 (`_segment2_adjusted`)를 초기화하세요.
- 그렇게 하면 **중지 후 재시작 시 예상치 못한 보정 누적 또는 미적용 문제 없이 안정 작동**합니다.