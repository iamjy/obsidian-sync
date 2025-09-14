---
title: Claude Desktop MCP Obsidian 연동 가이드
source:
author:
  - "[[@_louis]]"
published:
created: 2025-09-07
description: 이 가이드는 윈도우 환경에서 Claude Desktop과 Obsidian을 MCP(Model Context Protocol)를 통해 연동하는 절차를 다룹니다. 설치, API 키 설정, Python uv 설치, MCP 설정 파일 편집, 연동 확인 방법까지 단계별로 안내합니다.
tags:
  - claude-desktop-windows
  - mcp-obsidian-windows
---
## 개요
MCP(Model Context Protocol)를 통해 Claude Desktop과 Obsidian을 연동하여 LLM이 옵시디언 볼트의 노트를 참고할 수 있게 하는 방법입니다.

## 필요 사항
- Claude Desktop 설치
- Obsidian 설치
- Python uv 패키지 관리자

## 1. Claude Desktop 설치
[Claude Desktop](https://claude.ai/download)을 다운로드하여 로컬 PC에 설치합니다.

## 2. Obsidian REST API 플러그인 설치

### 2.1 Local REST API 커뮤니티 플러그인 설치
1. Obsidian에서 `Options` → `Community plugins` → `Browse` 버튼 클릭
2. "Local REST API" 검색하여 설치
3. **반드시 플러그인을 활성화**
   
   ![[Claude Desktop MCP Obsidian 연동 가이드 0.png]]
   
### 2.2 API Key 복사
1. 플러그인 활성화 버튼 옆 ⚙️ 버튼 클릭
2. 설정에서 API Key를 복사하여 보관 (Claude MCP 설정에 필요)
   
   ![[Claude Desktop MCP Obsidian 연동 가이드 1.png]]

## 3. Python uv 설치

### 3.1 설치 명령어
**Windows PowerShell:**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### 3.2 설치 확인
```bash
uv --version
```

## 4. Claude MCP 설정

### 4.1 설정 파일 열기
1. Claude Desktop 실행
2. 좌측 상단 햄버거 버튼 → `File` → `Settings`
3. `Developer` 탭 선택
4. `Edit Config` 버튼 클릭하여 `claude_desktop_config.json` 파일 열기

### 4.2 설정 파일 작성
다음 JSON을 `claude_desktop_config.json` 파일에 추가:

```json
{
  "mcpServers": {
    "mcp-obsidian": {
      "command": "uvx",
      "args": [
        "mcp-obsidian"
      ],
      "env": {
        "OBSIDIAN_API_KEY": "<YOUR_OBSIDIAN_API_KEY>"
      }
    }
  }
}
```

**중요:** `<YOUR_OBSIDIAN_API_KEY>` 부분을 2단계에서 복사한 실제 API Key로 교체해야 합니다.

## 5. 연동 확인

### 5.1 성공 확인 방법
Claude Desktop을 재시작한 후, Claude 입력창 하단에 새로운 🔨 아이콘이 나타나면 연동 성공입니다.

### 5.2 테스트 방법
Claude에게 옵시디언 볼트 내의 정보를 참고하는 질문을 해보세요. 더 정확한 답변을 위해 참고할 파일명이나 경로를 함께 제공하는 것을 권장합니다.

## 참고
- 원본 가이드: [https://tiaz.dev/ai/1](https://tiaz.dev/ai/1)
- MCP server for Obsidian: [GitHub 링크](https://github.com/MarkusPfundstein/mcp-obsidian)
- Python uv 설치 가이드: [Installing uv](https://docs.astral.sh/uv/getting-started/installation/)
