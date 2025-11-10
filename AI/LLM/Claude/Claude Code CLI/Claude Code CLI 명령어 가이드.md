---
title: 10배 생산성을 높이는 20가지 Claude Code CLI 명령어
source:
author:
  - "[[@_louis]]"
published:
created: 2025-09-15
description: 이 글은 단순한 명령어 목록을 넘어, 명령줄 베테랑부터 AI 개발 초보자까지 생산성을 10배 향상시키는 워크플로우와 기술을 제시하며, Claude Code CLI 마스터링과 더 효과적·효율적인 개발 도구를 제공한다.
tags:
  - claude-cli
---
## 개요
Claude Code CLI는 터미널에서 AI 기반 코딩 지원을 제공하는 강력한 도구입니다. 코드베이스를 이해하고, 명령을 실행하며, 프로젝트의 복잡성을 학습할 수 있는 에이전트 코딩 파트너 역할을 합니다.

## 1. 설치 및 기본 설정

### 설치
```bash
npm install -g @anthropic-ai/claude-code
```

### 알림 설정
```bash
claude config set --global preferredNotifChannel terminal_bell
```

## 2. 세션 관리 명령어

### 새 대화 시작
```bash
claude
```

### 최근 세션 재개
```bash
claude --continue
# 또는
claude -c
```

### 과거 세션 목록에서 선택
```bash
claude --resume
# 또는
claude -r
```

## 3. 세션 내 슬래시 명령어

### 프로젝트 초기화
```bash
/init
```
- 프로젝트 루트에 `CLAUDE.md` 파일 생성
- 프로젝트 아키텍처, 의존성, 코딩 컨벤션 정보 저장

### 대화 기록 관리
```bash
/clear      # 대화 기록 및 컨텍스트 재설정
/compact    # 대화 요약하여 토큰 수 줄이기
```

### 코드 리뷰
```bash
/review
```
- 풀 리퀘스트, 파일, 코드 블록 검토
- 버그 발견, 개선사항 제안, 스타일 가이드 준수 확인

### 기타 유용한 명령어
```bash
/help       # 사용 가능한 명령어 목록 표시
/model      # Claude 모델 변경 (Opus, Sonnet 등)
```

## 4. 프로젝트 이해를 위한 핵심 프롬프트

### 프로젝트 개요 파악
```
> summarize this project
```

### 폴더 구조 이해
```
> explain the folder structure
```

### 특정 기능 찾기
```
> find the files that handle user authentication
```

### 아키텍처 패턴 분석
```
> explain the main architecture patterns used here
```

## 5. 고급 기능

### 사용자 정의 슬래시 명령어
- `.claude/commands` 디렉토리에 마크다운 파일 생성
- 파일명이 명령어 이름이 됨
- 예시: `.claude/commands/test.md` → `/project:test`

### MCP (Model Context Protocol) 추가
```bash
claude mcp add playwright npx @playwright/mcp@latest
```

### 권한 관리
- `.claude/settings.json` 파일로 명령어 화이트리스트/블랙리스트 설정
- `permission.allow` / `permission.deny`

### 사용량 확인
```bash
npx ccusage@latest
```

### 심층 사고 활용
```
> ultrathink how to design a scalable real-time chat application
```

## 6. 에이전트 워크플로우

### TDD (테스트 주도 개발) 워크플로우
1. `> write a failing test for the new feature`
2. 테스트 실행 및 실패 확인
3. `> write the code to make the test pass`
4. 테스트 재실행 및 통과 확인
5. 리팩토링 요청

### 다중 인스턴스 활용
- 인스턴스 1: 코드 작성
- 인스턴스 2: 코드 리뷰
- 인스턴스 3: 리팩토링

## 7. 주요 생산성 팁

1. **컨텍스트 관리**: `/compact` 명령으로 긴 대화 유지
2. **프로젝트 문서화**: `CLAUDE.md` 파일로 프로젝트 정보 제공
3. **세션 재개**: `-c` 또는 `-r` 플래그로 컨텍스트 유지
4. **사용자 정의 명령**: 반복 작업 자동화
5. **모델 선택**: 작업 특성에 따른 적절한 모델 선택

## 결론
Claude Code CLI는 단순한 도구를 넘어 지능적인 코딩 파트너 역할을 하며, 개발자의 생산성을 크게 향상시킬 수 있는 혁신적인 도구입니다.
