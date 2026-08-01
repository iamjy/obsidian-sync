---
title: Hermes Agent SKILL 매뉴얼
source: 
author:
  - "[[@_louis]]"
  - "[[스마트앤컴퍼니]]"
published:
created: 2025-09-07
description: Hermes Agent 설치 및 Skill 추가 방법 가이드
tags:
  - hermes
  - hermes-skill
---

# Hermes Agent 가이드

## 1. Hermes Agent 설치
아래 명령어를 터미널에 입력하여 에이전트를 설치합니다.

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

## 2. Agent SKILL 추가 방법
새로운 Skill을 추가하는 단계별 예제입니다 (예: 주사위 굴리기 `roll-dice`).

### Step 1: Skill 디렉토리 생성 및 파일 작성
```bash
mkdir -p ~/.agents/skills/roll-dice
vim ~/.agents/skills/roll-dice/SKILL.md
```

### Step 2: SKILL.md 내용 작성
`SKILL.md` 파일에 아래 내용을 입력합니다.

---
**[SKILL.md 내용 시작]**
```markdown
---
name: roll-dice
description: Roll dice using a random number generator. Use when asked to roll a die (d6, d20, etc.), roll dice, or generate a random dice roll.
---

To roll a die, use the following command that generates a random number from 1 to the given number of sides:

```bash
echo $((RANDOM % <sides> + 1))
```

```powershell
Get-Random -Minimum 1 -Maximum (<sides> + 1)
```

Replace `<sides>` with the number of sides on the die (e.g., 6 for a standard die, 20 for a d20).
```
**[SKILL.md 내용 끝]**
---

### Step 3: Skill 심볼릭 링크 연결
작성한 Skill을 Hermes 에이전트가 인식할 수 있도록 링크를 생성합니다.

```bash
cd ~/.hermes/skills/
ln -s ../../.agents/skills/roll-dice/ .
```