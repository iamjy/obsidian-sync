---
title: "LM Studio CLI: `lms server start` 명령어 가이드"
source: https://lmstudio.ai/docs/cli/server-start
author:
  - "[[@_louis]]"
published:
created: 2025-10-13
description: "`lms server start`는 LM Studio 로컬 서버를 실행하여 HTTP API를 통해 모델과 상호작용할 수 있도록 하는 CLI 명령어입니다."
tags:
  - lm-studio
  - lm-studio-cli
---
# LM Studio CLI: `lms server start` 명령어 가이드

## 개요
`lms server start`는 LM Studio 로컬 서버를 실행하여 HTTP API를 통해 모델과 상호작용할 수 있도록 하는 CLI 명령어입니다. 서버는 다음 옵션을 통해 설정할 수 있습니다:

---

### 주요 기능

- 로컬 모델 서버 실행 및 관리
- 사용자 정의 포트 설정 지원
- CORS (Cross-Origin Resource Sharing) 활성화 옵션 제공
- 서버 상태 확인 기능 연동
  
---

## **사용법**
```bash
lms server start [옵션]
```

---

## **옵션**
| 옵션       | 설명                                                     |
| -------- | ------------------------------------------------------ |
| `--port` | 서버가 실행될 포트 번호를 지정 (값 미기재 시 마지막 사용 포트 사용)               |
| `--cors` | 웹 애플리케이션개발을 위한 CORS(Cross-Origin Resource Sharing) 활성화 |

---

## **시작 예제**
### 1. 기본 설정으로 서버 시작
```bash
lms server start
```

### 2. 특정 포트 사용  
3000 포트에서 서버 실행:
```bash
lms server start --port 3000
```

### 3. CORS 활성화  
웹 확장 프로그램 활용을 위해 CORS 허용:
```bash
lms server start --cors
```
> ⚠️ CORS 활성화 시 보안 위험이 있을 수 있으므로 **필요할 경우에만 사용** 권장됩니다.

---

## **추가 가이드**
- **서버 상태 확인**: `lms server status` 명령어 참조 ([링크](https://lmstudio.ai/docs/cli/server-status) 참조).
- **서버 정지**: `lms server stop` 명령어 사용.

---

## **참고**
- 본 문서는 [LM Studio 공식 문서](https://github.com/lmstudio-ai/docs) 기반으로 작성되었습니다.
- 보안 설정 및 CORS 처리를 주의적으로 설정하여 서버의 안전한 사용이 권장됩니다.