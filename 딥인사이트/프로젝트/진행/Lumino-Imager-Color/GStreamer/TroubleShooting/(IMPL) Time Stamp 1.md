---
title: 
source: 
author: 
published: 
created: 
description: 
tags:
---
---

`on_format_location()`에서 **현재 시간을 직접 사용하면 GStreamer 내부 전환 시점보다 늦은 시간으로 저장**되어 **최대 약 10초 오차 발생**하는 것이 맞습니다.

### 해결방안
콜백에서 "**현재 시스템 시간**" 대신,  
**현재 스트림의 `"마지막 처리된 버퍼의 PTS (Presentation Timestamp)"** 를 사용하면 됩니다.

### 방법 요약
- 전역 또는 클래스 내 변수에 `"가장 최근 버퍼 pts"` 저장
  - `handoff`, `pad probe` 등에서 `buffer.pts` 읽어 업데이트
- `on_format_location()` 호출 시, 그 값을 기반으로 파일명 생성  
  (예: `base_time + 누적분` 계산 가능)
  
### 간단한 가이드
1. 클래스에 `self.last_buffer_pts` 변수 추가
2. `handoff` 콜백 내 혹은 적절한 패드 프로브 내에서:
```python
buffer = ... # 현재 Gst.Buffer
if buffer and buffer.pts != Gst.CLOCK_TIME_NONE:
    self.last_buffer_pts = buffer.pts
```
3. `on_format_location()`에서:
```python
from datetime import datetime, timedelta

timestamp_sec = self.last_buffer_pts / Gst.SECOND
ts_datetime = datetime.fromtimestamp(timestamp_sec)

new_filename = f"video_{ts_datetime.strftime('%Y-%m-%d_%H-%M-%S')}.mp4"
return new_filename
```

이렇게 하면 **실제 영상 캡처 시점**의 시간이 `파일명`으로 정확히 반영되어  
10초 시간차 문제를 해결할 수 있습니다.