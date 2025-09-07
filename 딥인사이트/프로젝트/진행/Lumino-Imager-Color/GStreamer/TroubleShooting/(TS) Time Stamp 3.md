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

# 요청
현재 녹화 시 1분단위로 파일을 분할하도록 되어 있는데 녹화 중지 후 다시 시작할 때 초기화에서 사용한 값들이 전혀 적용되어 있지 않는거 같아. 왜 그럴까?

# 답변
문제: 녹화 중지 후 재시작 시, 초기화 시점에 설정한 splitmuxsink 관련 값들(`max-size-time`, `location`, `reset-muxer` 등)이 적용되지 않는 현상

가능한 원인:

1. **splitmuxsink 인스턴스가 파이프라인 내에 그대로 유지되고, 재생성되지 않기 때문**
    - `_initialize_pipeline()`은 파이프라인 새로 생성 시점에만 호출
    - 녹화 중지 후 재시작 시에는 기존 파이프라인과 splitmuxsink를 그대로 재사용
2. splitmuxsink는 내부적으로 muxer를 유지하며, **중간에 속성 변경이 일부 무시되거나, 초기화 시점의 설정이 유지되지 않음**
3. 특히 `max-size-time`과 같은 속성은 **READY 상태에서 변경해야 적용되는데**, 재시작 시점에 이미 splitmuxsink가 PLAYING 또는 PAUSED 상태일 수 있어 변경이 무시될 수 있음
4. `reset-muxer=True` 속성도 초기화 시점에만 유효하며, 이후 동적으로 변경해도 효과가 없을 수 있음
5. `location` 역시 녹화 재시작 시 새로 지정하지 않으면, 이전 값이 유지되어 새 파일이 생성되지 않거나, 세그먼트 분할이 일어나지 않음

핵심 원인:

- **녹화 중지 후 재시작 시 splitmuxsink 인스턴스를 재생성하지 않고, 기존 인스턴스를 그대로 재사용하기 때문**
- splitmuxsink는 내부 상태를 유지하므로, 초기화 시점에 설정한 값들이 재시작 시점에 제대로 반영되지 않음
- 특히, muxer 내부 상태가 초기화되지 않아, 새 세그먼트 분할이 일어나지 않거나, 새 파일명이 적용되지 않는 문제 발생

해결책:

- **녹화 중지 후 재시작 시, splitmuxsink를 반드시 새로 생성하고, 초기화 시점과 동일한 속성으로 재설정**
- 즉, `_initialize_pipeline()`을 호출하여 파이프라인 전체를 재구성하거나,
- 최소한 splitmuxsink 인스턴스를 파이프라인에서 제거 후 새로 생성하여 다시 연결

요약:

- splitmuxsink는 내부 상태를 강하게 유지하는 특성이 있어, **중간에 속성만 변경하는 것으로는 초기화 효과를 기대하기 어렵다**
- 따라서, **녹화 재시작 시 splitmuxsink를 새로 생성하고, 초기 설정값을 다시 적용하는 것이 가장 확실한 방법**

최종 보고

Task Completed

녹화 중지 후 다시 시작할 때 초기화 시점에 설정한 splitmuxsink 관련 값들이 제대로 적용되지 않는 이유는, **splitmuxsink 인스턴스가 그대로 유지되기 때문**이다.

splitmuxsink는 내부적으로 muxer 상태와 세그먼트 정보를 유지하므로,

# 요청
녹화 중지 후 다시 시작할 때 초기화 시점에 설정한 splimtxsink 관련 값들이 제대로 적용되지 않는 이유가 splimuxsink 인스턴스가 그대로 유지되어 발생하는 것으로 추측되는데 확인해주고, 이 분석 결과를 바탕으로 splitmuxsink 를 반드시 새로 생성하고 초기화 시점과 동일한 속성으로 재설정해줘.

# 답변
당신의 분석이 정확합니다. 현재 splitmuxsink 문제는 상태 변경(READY→PLAYING)만으로는 설정값들이 제대로 유지되지 않기 때문입니다. 이 문제를 해결하기 위해 splitmuxsink를 완전히 제거하고 새로 생성하는 방식으로 접근하겠습니다.