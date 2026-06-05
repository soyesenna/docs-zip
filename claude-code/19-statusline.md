# 19. 상태 라인 구성 (Status Line Configuration)

> **참조**: [Status line configuration - Anthropic](https://docs.anthropic.com/en/docs/claude-code/settings)

---

## 목차

- [상태 라인 개요](#상태-라인-개요)
- [구성 방법](#구성-방법)
- [명령어 기반 상태 라인](#명령어-기반-상태-라인)
- [예시 스크립트](#예시-스크립트)

---

## 상태 라인 개요

상태 라인(Status Line)은 Claude Code 터미널 인터페이스 하단에 컨텍스트 정보를 표시하는 커스터마이징 가능한 영역입니다. 현재 브랜치, 프로젝트 상태, Git 정보 등을 실시간으로 표시할 수 있습니다.

---

## 구성 방법

### settings.json을 통한 설정

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh"
  }
}
```

### /config를 통한 설정

```bash
> /config
# Status line → command로 설정 후 스크립트 경로 입력
```

---

## 명령어 기반 상태 라인

`type: "command"` 설정은 지정된 쉘 명령어를 주기적으로 실행하여 그 출력을 상태 라인에 표시합니다.

### 구성 필드

| 필드 | 설명 |
|------|------|
| `type` | `"command"`로 설정 |
| `command` | 실행할 쉘 명령어 또는 스크립트 경로 |

---

## 예시 스크립트

### Git 브랜치 및 상태 표시

```bash
#!/bin/bash
# ~/.claude/statusline.sh

BRANCH=$(git branch --show-current 2>/dev/null || echo "not a git repo")
CHANGES=$(git status --porcelain 2>/dev/null | wc -l | tr -d ' ')

if [ "$CHANGES" -gt 0 ]; then
  echo "📁 ${BRANCH} ✏️ ${CHANGES}개 변경"
else
  echo "📁 ${BRANCH} ✅ 깨끗함"
fi
```

### 프로젝트 컨텍스트 표시

```bash
#!/bin/bash
# ~/.claude/statusline.sh

PROJECT=$(basename "$PWD")
BRANCH=$(git branch --show-current 2>/dev/null || echo "N/A")
TIME=$(date +"%H:%M")

echo "🖥️ ${PROJECT} | 🔀 ${BRANCH} | 🕐 ${TIME}"
```

### 테스트 상태 표시

```bash
#!/bin/bash
# ~/.claude/statusline.sh

if [ -f "package.json" ]; then
  BRANCH=$(git branch --show-current 2>/dev/null || echo "N/A")
  echo "📦 Node.js 프로젝트 | 🔀 ${BRANCH}"
elif [ -f "requirements.txt" ]; then
  BRANCH=$(git branch --show-current 2>/dev/null || echo "N/A")
  echo "🐍 Python 프로젝트 | 🔀 ${BRANCH}"
else
  echo "📂 $(basename "$PWD")"
fi
```

---

## 설정 가이드

1. 스크립트 파일을 생성하고 실행 권한 부여: `chmod +x ~/.claude/statusline.sh`
2. `settings.json`에 상태 라인 설정 추가
3. Claude Code 재시작 후 상태 라인 확인

### 주의사항

- 스크립트는 빠르게 실행되어야 합니다 (긴 실행 시간은 UI 지연 유발)
- 스크립트 출력은 한 줄로 유지하는 것이 좋습니다
- 출력에 ANSI 색상 코드는 지원되지 않습니다
