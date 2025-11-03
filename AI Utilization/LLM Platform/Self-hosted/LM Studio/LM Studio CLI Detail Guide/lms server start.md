---
title: "LM Studio CLI: `lms server start` 명령어 가이드"
source: https://lmstudio.ai/docs/cli/server-start
author:
  - "[[@_louis]]"
published:
created: 2025-10-13
description: "`lms server start`는 LM Studio 로컬 서버를 실행해 HTTP API 로 모델을 호출할 수 있게 해 주는 명령입니다."
tags:
---
# LM Studio CLI `lms server start` 가이드

---

## 📌 개요
`lms server start`는 **LM Studio** 로컬 서버를 실행해 HTTP API 로 모델을 호출할 수 있게 해 주는 명령입니다.  
주요 목적은  

* 로컬에서 로드된 LLM (large language model) 에 REST API 로 접근  
* 다른 애플리케이션·VS Code Extension·웹 프론트엔드와 연동  
* 포트와 CORS 옵션을 자유롭게 지정  

아래에서는 설치부터 실행, 옵션 활용, 보안 주의사항, 흔히 겪는 오류와 해결법까지 단계별로 설명합니다.

---

## 1️⃣ 사전 준비

| 단계 | 내용 | 비고 |
|------|------|------|
| 1 | **LM Studio** 설치 (Desktop 앱) | macOS, Windows, Linux 지원 |
| 2 | **Node.js ≥ 14** (또는 npm) 설치 – CLI 패키지는 npm 또는 yarn 으로 설치 | `node -v` 로 버전 확인 |
| 3 | **CLI 설치**  `npm i -g @lmstudio/cli` | 전역 설치 (※ 관리자 권한 필요) |
| 4 | **모델 로드** – LM Studio 앱에서 원하는 모델을 *Load* 해 두세요. | 서버 시작 전 반드시 로드되어 있어야 API 호출 가능 |

> **Tip**  
> 모델을 로드하지 않으면 `GET /v1/models` 에서 빈 목록이 반환됩니다.

---

## 2️⃣ 기본 서버 실행

```bash
lms server start
```

* **동작**  
  * 이전에 사용한 포트(기본 8000)로 서버를 시작합니다.  
  * 콘솔에 `Server listening on http://127.0.0.1:8000` 와 같은 로그가 표시됩니다.  

* **API 엔드포인트** (예시)  

| 메서드 | 경로 | 설명 |
|--------|------|------|
| `GET` | `/v1/models` | 현재 로드된 모델 목록 |
| `POST`| `/v1/completions` | 텍스트 생성 요청 |
| `POST`| `/v1/chat/completions` | 챗 형식 요청 |

> **주의**  
> 기본적으로 **CORS** 가 **비활성화** 되어 있습니다. 웹 애플리케이션에서 직접 호출하려면 `--cors` 옵션을 사용해야 합니다.

---

## 3️⃣ 옵션 설명

| 옵션 | 형태 | 설명 | 예시 |
|------|------|------|------|
| `--port` | `<number>` | 서버가 바인딩할 포트 지정 (기본값: 마지막 사용 포트 또는 8000) | `lms server start --port 3000` |
| `--cors` | flag | **Cross‑Origin Resource Sharing** 허용. 웹 브라우저에서 동일 출처 정책을 우회하게 함. | `lms server start --cors` |
| `--help` | flag | 사용법 출력 | `lms server start --help` |

### 3‑1️⃣ 포트 지정 예시

```bash
# 8081 포트에서 서버 실행
lms server start --port 8081
```

### 3‑2️⃣ CORS 활성화 예시

```bash
# CORS 활성화 + 5000 포트 지정
lms server start --port 5000 --cors
```

> **보안 경고**  
> CORS 를 켜면 외부 웹 페이지가 로컬 서버에 접근할 수 있습니다.  
> - **개발 환경**에서만 사용하고, **프로덕션**에서는 차단하거나 프록시 서버 뒤에 두세요.  
> - 방화벽을 이용해 로컬 IP(127.0.0.1) 외의 접근을 제한하는 것이 안전합니다.

---

## 4️⃣ 서버 상태 확인 및 종료

| 명령 | 설명 |
|------|------|
| `lms server status` | 현재 서버가 실행 중인지, 포트, pid 등을 출력 |
| `lms server stop`   | 실행 중인 LM Studio 서버를 강제 종료 |

```bash
# 상태 확인
lms server status

# 서버 중지
lms server stop
```

---

## 5️⃣ 실전 활용 예시

### 5‑1️⃣ cURL 로 모델 호출

```bash
curl -X POST http://127.0.0.1:8000/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
        "model": "your-model-name",
        "prompt": "안녕하세요, 오늘 날씨는?",
        "max_tokens": 50
      }'
```

### 5‑2️⃣ JavaScript (Fetch) 로 호출 (CORS 활성화 필요)

```js
fetch('http://127.0.0.1:5000/v1/chat/completions', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    model: 'your-model-name',
    messages: [{role: 'user', content: '안녕!'}],
    max_tokens: 100
  })
})
.then(res => res.json())
.then(console.log)
.catch(console.error);
```

---

## 6️⃣ 흔히 발생하는 오류와 해결책

| 오류 메시지 | 원인 | 해결 방법 |
|-------------|------|-----------|
| `Error: address already in use` | 지정 포트가 이미 다른 프로세스에 의해 사용 중 | 다른 포트 지정 (`--port 3001`) 또는 기존 프로세스 종료 (`lms server stop` 또는 `kill <pid>`) |
| `CORS policy: No 'Access-Control-Allow-Origin' header` | `--cors` 옵션 미사용 | 서버를 `lms server start --cors` 로 재시작 |
| `Model not found` | LM Studio 앱에서 모델을 로드하지 않음 | LM Studio → *Model* 탭에서 원하는 모델을 **Load** |
| `Connection refused` | 서버가 실행되지 않음 | `lms server status` 로 상태 확인 후 `lms server start` 실행 |

---

## 7️⃣ 베스트 프랙티스

1. **포트 관리** – 개발 중 여러 인스턴스를 띄우는 경우 포트를 명시적으로 지정하고, `lms server status` 로 현재 사용 포트를 파악한다.  
2. **CORS 제한** – 로컬 개발이 끝나면 `--cors` 옵션을 끄고, 프록시(Nginx·Traefik 등) 로 외부 접근을 중계한다.  
3. **로그 레벨** – 필요 시 `LM_STUDIO_LOG=debug` 환경 변수를 설정해 상세 로그를 확인한다.  
4. **자동 시작** – Docker 혹은 systemd 서비스 파일에 `lms server start --port 8000 --cors` 명령을 넣어 부팅 시 자동 실행하도록 구성한다.  

```yaml
# 예시: systemd 서비스 (Ubuntu)
[Unit]
Description=LM Studio Server
After=network.target

[Service]
ExecStart=/usr/local/bin/lms server start --port 8000 --cors
Restart=always
User=your_user

[Install]
WantedBy=multi-user.target
```

---

## 8️⃣ 마무리

`lms server start` 명령은 **LM Studio** 를 로컬 API 서버로 변환해 다양한 애플리케이션과 손쉽게 연동할 수 있게 해 줍니다.  
핵심은  

* **포트와 CORS 옵션을 명확히 설정** → 보안·편의성 균형 유지  
* **모델을 미리 로드** → API 호출 시 오류 방지  
* **상태·종료 명령**을 활용해 서버 관리  

위 가이드를 따라 설정하면 빠르게 로컬 LLM 서비스를 구축하고, 원하는 프로젝트에 바로 적용할 수 있습니다. 🚀  