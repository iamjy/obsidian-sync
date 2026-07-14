---
title: Cloudflare Tunnel 설정 가이드
source: 
author: 
published: 
created: 2025-09-07
description: 로컬 서버를 외부 도메인에 안전하게 연결하는 Cloudflare Tunnel(대시보드 관리형) 구축 방법
tags: [Cloudflare, Tunnel, Network, Devops]
---
Cloudflare Tunnel은 로컬 서버의 포트를 외부에 개방하지 않고도, Outbound 커넥션을 통해 외부 도메인으로 안전하게 접속할 수 있게 해주는 서비스입니다. 본 문서는 관리가 용이한 **대시보드 관리형(Dashboard-managed)** 방식을 기준으로 설명합니다.

## 🚀 설정 단계 (4 Step)

### 1. 도메인을 Cloudflare DNS에 등록
터널 사용을 위해 소유한 도메인이 Cloudflare에서 활성화되어 있어야 합니다.
1. **사이트 추가:** Cloudflare 로그인 후 `[Add a Site]` 버튼으로 도메인 추가.
2. **네임서버 변경:** 도메인 구매 업체(가비아, GoDaddy 등) 설정에서 네임서버를 Cloudflare 제공 주소로 변경.
3. **활성화 대기:** 대시보드에서 도메인 상태가 'Active'가 될 때까지 대기.

### 2. Zero Trust 대시보드에서 Tunnel 생성
네트워크 통제를 위한 가상 터널 입구를 생성합니다.
1. **메뉴 이동:** `[Zero Trust]` $\rightarrow$ `[Networks]` $\rightarrow$ `[Tunnels]`.
2. **터널 생성:** `[Create a tunnel]` 클릭.
3. **설정:** Connector 타입을 `Cloudflared`로 선택하고, 식별 가능한 이름(예: `my-local-web`) 입력 후 저장.

### 3. 로컬 서버에 커넥터 설치 및 실행
에이전트 프로그램을 통해 실제 로컬 서버와 Cloudflare를 연결합니다.
1. **OS 선택:** 자신의 환경(Windows, macOS, Linux 등)을 선택하여 제공되는 명령어를 복사합니다.
2. **명령어 실행:** 로컬 터미널에서 복사한 설치 및 토큰 실행 명령어를 입력합니다.

**예시 (Ubuntu/Debian):**
```bash
# cloudflared 패키지 다운로드 및 설치
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb

# 서비스 등록 및 실행 (YOUR_UNIQUE_TOKEN 부분에 실제 토큰 입력)
sudo cloudflared service install YOUR_UNIQUE_TOKEN_HERE
```
3. **상태 확인:** 대시보드 하단의 `Connectors` 상태가 **Active**로 변경되었는지 확인합니다.

### 4. 공개 도메인과 로컬 포트 매핑 (Public Hostname)
외부 주소와 내부 서비스의 연결 규칙을 정의합니다.
1. **탭 이동:** `[Public Hostname]` 탭 클릭.
2. **호스트네임 설정:**
   - **Subdomain:** 사용할 서브도메인 입력 (예: `test`, `api`)
   - **Domain:** 등록한 내 도메인 선택.
3. **서비스 연결 정보 입력:**
   - **Type:** `HTTP` (기본)
   - **URL:** `localhost:8080` (실제 로컬 서버 포트 번호)
4. **저장:** `[Save hostname]` 클릭.

---

## ✅ 최종 확인
웹 브라우저 주소창에 `https://test.yourdomain.com`을 입력하여 접속합니다. 별도의 SSL 설정 없이 HTTPS 보안 연결이 적용된 로컬 서버 화면이 출력되면 정상적으로 완료된 것입니다.