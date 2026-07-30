---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
안녕하세요? 정영현상무님,

딥인사이트 김형철입니다.

 

아래 전문에서 언급하신 2번에 대한 협조에 대한 협의가 완료되었는지 궁금합니다.

 

2. [핵심 요청] 텔레칩스의 카메라 브링업(Bring-up) 직접 수행 및 결과물 이관 -> Camera 2Ch. 브링업이 7월까지 해결되지 않으면, 2Ch. KPI는 과제 기한 내 불가능함.

칩 벤더사인 텔레칩스에서 직접 핵심 기능에 대한 브링업을 완료한 후 검증된 결과물(이미지 및 파이프라인)을 당사에 이관하는 방식으로 변경 요청

텔레칩스 직접 검증 및 완료 필수 항목:
GPU 안정성 확보: ARM Mali GPU 초기화가 커널/시스템 레벨에서 정상 로드되며, GUI 멈춤 등 기존 이슈 없이 100% 구동되는 환경
GStreamer 2Ch 구동 확보: 멀티 인스턴스 및 VPU 하드웨어 가속이 정상 적용된 Dual Preview 및 Recording 파이프라인의 직접 구동 및 검증 완료
위 내용에 대한 명확한 Guide 문서 제공
 

텔레칩스에서 향후 과제 수행을 위한 구체적인 계획이나, 실행 방안에 대한 회신을 받지 못했습니다.

신규 패치된 SDK를 텔레칩스에서 보내왔는데, 추가 수정사항이 있을 수 있다고 합니다.

당사에서는 텔레칩스의 “한번해보고 안되면, 다시해보는”방식의 개발을 더 이상 지원하기 어려워 실무진에게 개발 중단을 지시했습니다.

㈜딥인사이트는 텔레칩스의 칩 디버깅을 진행하는 협력사가 아니라, 해당 과제를 수행하는 공동연구기관임을 명확히 하고 싶습니다.

 

주관기관의 명확한 답변 기다리겠습니다.

감사합니다.

 

From: 정영현 <yhjeong@mcnex.com>
Sent: Monday, July 6, 2026 4:28 PM
To: 김형철(Hyungcheol KIM) <hyungcheol.kim@dinsight.ai>; '권순재' <sjkwon@mcnex.com>
Cc: '정혜진' <hjjung@mcnex.com>; 'MCNEX_서장미 팀장님' <jmseo@mcnex.com>; '이해린' <hrlee@mcnex.com>; 신윤섭(Yunsup SHIN) <yunsup.shin@dinsight.ai>; 지인찬(Inchan JI) <iji@dinsight.ai>; 박진영(Louis) <louis@dinsight.ai>
Subject: RE: Main core 통합 SDK 디버깅 건 공동기관 업무분장 재조정 요청의 건

 

안녕하세요 본부장님. 진행 내용에 대한 회신 감사합니다.

 

제가 아래 내용을 보고, 정리 한다면,  결국 2CH에 대한 기능 요구사양을 만족 하려면,

현재 방법 중에 최선은 2번이 가장 빠른 방법이라 이해 됩니다.

 

텔레칩스에 아래 2번에 대한 협조를 별도 채널로 우선적으로 요청 하겠습니다.

 

의견 부탁드립니다.

 

From: 김형철(Hyungcheol KIM) <hyungcheol.kim@dinsight.ai>
Sent: Monday, July 6, 2026 2:51 PM
To: '정영현' <yhjeong@mcnex.com>
Cc: '정혜진' <hjjung@mcnex.com>; 'MCNEX_서장미 팀장님' <jmseo@mcnex.com>; 이해린 <hrlee@mcnex.com>; '권순재' <sjkwon@mcnex.com>; 신윤섭(Yunsup SHIN) <yunsup.shin@dinsight.ai>; 지인찬(Inchan JI) <iji@dinsight.ai>; 박진영(Louis) <louis@dinsight.ai>
Subject: Main core 통합 SDK 디버깅 건 공동기관 업무분장 재조정 요청의 건

 

안녕하세요? 정영현상무님,

딥인사이트 김형철입니다.

 

당사는 지난 회의에서 텔레칩스 측이 공유해 주신 내용을 바탕으로 Main Core SDK에 대한 내부 검토를 착수하였습니다.

그러나 현재 배포된 SDK의 빌드 옵션(Build Option) 상에 비디오 인코더(Video Encoder)가 비활성화되어 있고,

관련 가이드 문서 또한 아직 전달받지 못해 부득이하게 당사의 검토 작업을 잠정 중단하게 되었습니다.

 

아울러 첨부해 드린 실무진 간의 소통 이력(이미지 참조)에서 확인하실 수 있듯,

현재 Main Core SDK 환경에서 본 프로젝트의 핵심인 '2Ch 동시 녹화' 기능에 대한 자체 검증이 아직 이루어지지 않았다는 점을 확인하였습니다.

이로 인해 당사에서 선제적으로 SDK를 검토하고 디버깅을 진행하기에는 현실적인 어려움이 있는 상황입니다.



1. 현 개발 프로세스의 한계 및 실무진 리스크

업무 효율성 급감: 미 검증된 SDK 기반에서 당사가 이슈를 리포트하고, 1주일 이상 텔레칩스의 가이드를 대기하여 재테스트하는 비효율적인 핑퐁 구조가 반복되고 있습니다.
실무진 피로도 한계 도달: 불완전한 상태의 환경에서 당사가 직접 가이드에 의존해 디버깅을 시도함에 따라 파생 이슈 추적에 과도한 공수가 낭비되고 있습니다.
 

2. [핵심 요청] 텔레칩스의 카메라 브링업(Bring-up) 직접 수행 및 결과물 이관 -> Camera 2Ch. 브링업이 7월까지 해결되지 않으면, 2Ch. KPI는 과제 기한 내 불가능함.

칩 벤더사인 텔레칩스에서 직접 핵심 기능에 대한 브링업을 완료한 후 검증된 결과물(이미지 및 파이프라인)을 당사에 이관하는 방식으로 변경 요청

텔레칩스 직접 검증 및 완료 필수 항목:
GPU 안정성 확보: ARM Mali GPU 초기화가 커널/시스템 레벨에서 정상 로드되며, GUI 멈춤 등 기존 이슈 없이 100% 구동되는 환경
GStreamer 2Ch 구동 확보: 멀티 인스턴스 및 VPU 하드웨어 가속이 정상 적용된 Dual Preview 및 Recording 파이프라인의 직접 구동 및 검증 완료
위 내용에 대한 명확한 Guide 문서 제공
 

3. 주관사(MCNEX)의 명확한 R&R 재정립 요청

본 과제에서 각 참여 기관의 의미와 효율성을 극대화하기 위해서는 초기 기획된 바와 같이 명확한 R&R이 준수되어야 합니다.

카메라 및 칩 제조사: 하드웨어 및 카메라 브링업 환경 제공
당사: 제공된 안정적 환경 위에서 Application 및 AI 기능 개발
 

현 시점에서 프로젝트를 정상 궤도에 올리기 위해서는 주관사인 MCNEX의 적극적인 중재가 반드시 필요합니다.

본 과제가 본래의 취지와 R&R에 맞게 운영될 수 있도록, 조속한 일정 내에 업무 조율 및 방향성에 대한 조정 부탁드립니다.

 

감사합니다.

 





Deep-In-Sight Logo

김형철(김형철)

MDS사업본부 / 본부장 / PO

Deep-In-Sight Co., Ltd. (www.dinsight.ai)

M +82.10.2331.6590

E hyungcheol.kim@dinsight.ai

 

 