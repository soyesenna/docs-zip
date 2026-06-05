# 08. 서브에이전트 (Subagents)

> **참조**: [Create custom subagents - Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
>
> **이전 참조**: [Subagents - Anthropic](https://docs.anthropic.com/en/docs/claude-code/sub-agents)

---

## 목차

- [서브에이전트 개요](#서브에이전트-개요)
- [핵심 이점](#핵심-이점)
- [빌트인 서브에이전트](#빌트인-서브에이전트)
- [Quickstart: 첫 서브에이전트 만들기](#quickstart-첫-서브에이전트-만들기)
- [/agents 명령어로 관리](#agents-명령어로-관리)
- [서브에이전트 범위 선택](#서브에이전트-범위-선택)
- [서브에이전트 파일 작성](#서브에이전트-파일-작성)
- [Frontmatter 필드](#frontmatter-필드)
- [모델 선택](#모델-선택)
- [도구 접근 제어](#도구-접근-제어)
- [사용 불가 도구](#사용-불가-도구)
- [MCP 서버 스코프](#mcp-서버-스코프)
- [권한 모드](#권한-모드)
- [스킬 프리로드](#스킬-프리로드)
- [영구 메모리](#영구-메모리)
- [Isolation (Worktree) 모드](#isolation-worktree-모드)
- [Hooks로 조건부 제어](#hooks로-조건부-제어)
- [CLI를 통한 동적 정의](#cli를-통한-동적-정의)
- [특정 서브에이전트 비활성화](#특정-서브에이전트-비활성화)
- [명시적 서브에이전트 호출](#명시적-서브에이전트-호출)
- [서브에이전트 작업 패턴](#서브에이전트-작업-패턴)
- [시작 시 로드되는 항목](#시작-시-로드되는-항목)
- [포그라운드와 백그라운드 실행](#포그라운드와-백그라운드-실행)
- [서브에이전트 재개](#서브에이전트-재개)
- [대화 Fork](#대화-fork)
- [서브에이전트 제약사항](#서브에이전트-제약사항)
- [실전 예시](#실전-예시)
- [모범 사례](#모범-사례)

---

## 서브에이전트 개요

서브에이전트(Subagent)는 Claude Code에서 특정 작업에 특화된 AI 어시스턴트입니다. 메인 대화와 **분리된 컨텍스트 윈도우**에서 동작하며, 작업이 끝나면 결과만 메인 대화로 반환합니다. 서브에이전트는 자체 시스템 프롬프트, 특정 도구 접근, 독립적인 권한을 가지고 실행됩니다.

Claude는 작업이 서브에이전트의 `description`과 일치하면 자동으로 해당 서브에이전트에 위임합니다.

### 작동 방식

```
사용자 요청 → Claude Code → 적절한 서브에이전트에 작업 위임
                                ↓
                    서브에이전트가 독립 컨텍스트에서 작업 수행
                                ↓
                    결과를 메인 대화로 반환
```

---

## 핵심 이점

서브에이전트는 다음과 같은 이점을 제공합니다.

| 이점 | 설명 |
|------|------|
| **Preserve context** | 탐색과 구현 결과를 메인 대화 밖에 유지하여 컨텍스트 보존 |
| **Enforce constraints** | 서브에이전트가 사용할 수 있는 도구를 제한하여 제약 강제 |
| **Reuse configurations** | 사용자 레벨 서브에이전트로 여러 프로젝트에서 설정 재사용 |
| **Specialize behavior** | 특정 도메인에 집중된 시스템 프롬프트로 행동 특화 |
| **Control costs** | Haiku 같은 빠르고 저렴한 모델로 작업을 라우팅하여 비용 제어 |

---

## 빌트인 서브에이전트

Claude Code에는 Claude가 상황에 맞게 자동으로 사용하는 빌트인 서브에이전트가 포함되어 있습니다. 각 서브에이전트는 부모 대화의 권한을 상속하며, 추가적인 도구 제한이 적용됩니다.

### Explore

빠르고 읽기 전용으로 코드베이스 검색 및 분석에 최적화된 에이전트입니다.

| 항목 | 값 |
|------|-----|
| **모델** | Haiku (빠르고 저지연) |
| **도구** | 읽기 전용 도구 (Write, Edit 접근 거부) |
| **용도** | 파일 검색, 코드 탐색, 코드베이스 분석 |

Claude는 코드베이스를 변경 없이 검색하거나 이해해야 할 때 Explore에 위임합니다. 호출 시 Claude는 **thoroughness level**을 지정합니다: `quick` (타겟팅된 조회), `medium` (균형 잡힌 탐색), `very thorough` (포괄적 분석).

### Plan

Plan 모드에서 계획을 제시하기 전 컨텍스트를 수집하는 리서치 에이전트입니다.

| 항목 | 값 |
|------|-----|
| **모델** | 메인 대화에서 상속 |
| **도구** | 읽기 전용 도구 (Write, Edit 접근 거부) |
| **용도** | 계획 수립을 위한 코드베이스 리서치 |

### General-purpose

탐색과 실행이 모두 필요한 복잡한 다단계 작업을 위한 에이전트입니다.

| 항목 | 값 |
|------|-----|
| **모델** | 메인 대화에서 상속 |
| **도구** | 전체 도구 |
| **용도** | 복잡한 리서치, 다단계 작업, 코드 수정 |

### 기타 빌트인 에이전트

| 에이전트 | 모델 | 사용 시점 |
|----------|------|-----------|
| `statusline-setup` | Sonnet | `/statusline` 실행 시 |
| `claude-code-guide` | Haiku | Claude Code 기능에 대해 질문할 때 |

> **참고**: Explore와 Plan은 CLAUDE.md 파일과 부모 세션의 git status를 건너뛰어 리서치를 빠르고 저렴하게 유지합니다. 다른 모든 빌트인 및 커스텀 서브에이전트는 둘 다 로드합니다.

---

## Quickstart: 첫 서브에이전트 만들기

서브에이전트는 YAML frontmatter가 포함된 Markdown 파일로 정의됩니다. 직접 생성하거나 `/agents` 명령어를 사용할 수 있습니다.

`/agents` 명령어를 사용하면 안내된 설정으로 사용자 레벨 서브에이전트를 쉽게 만들 수 있습니다. 생성된 서브에이전트는 머신의 모든 프로젝트에서 사용할 수 있습니다.

수동으로 Markdown 파일을 직접 생성하거나, CLI 플래그로 정의하거나, 플러그인을 통해 배포할 수도 있습니다.

---

## /agents 명령어로 관리

`/agents` 명령어는 서브에이전트를 관리하는 탭 형태의 인터페이스를 엽니다.

### Running 탭

현재 실행 중인 서브에이전트를 보여주며, 열거나 중지할 수 있습니다.

### Library 탭

- 사용 가능한 모든 서브에이전트 조회 (빌트인, 사용자, 프로젝트, 플러그인)
- 안내된 설정 또는 Claude 생성으로 새 서브에이전트 생성
- 기존 서브에이전트의 설정 및 도구 접근 편집
- 커스텀 서브에이전트 삭제
- 중복 존재 시 어떤 서브에이전트가 활성 상태인지 확인

서브에이전트를 생성하고 관리하는 데 권장되는 방법입니다.

---

## 서브에이전트 범위 선택

서브에이전트는 YAML frontmatter가 포함된 Markdown 파일입니다. 범위에 따라 다른 위치에 저장합니다. 여러 서브에이전트가 같은 이름을 공유하면 높은 우선순위 위치가 우선합니다.

| 위치 | 범위 | 우선순위 | 생성 방법 |
|------|------|----------|-----------|
| Managed settings | 조직 전체 | 1 (최고) | Managed settings를 통해 배포 |
| `--agents` CLI 플래그 | 현재 세션 | 2 | Claude Code 실행 시 JSON 전달 |
| `.claude/agents/` | 현재 프로젝트 | 3 | 대화형 또는 수동 |
| `~/.claude/agents/` | 모든 프로젝트 | 4 | 대화형 또는 수동 |
| 플러그인 `agents/` 디렉터리 | 플러그인이 활성화된 곳 | 5 (최저) | 플러그인과 함께 설치 |

**프로젝트 서브에이전트** (`.claude/agents/`)는 코드베이스에 특화된 서브에이전트에 이상적입니다. 버전 관리에 체크인하면 팀 전체가 공동으로 사용하고 개선할 수 있습니다.

**사용자 서브에이전트** (`~/.claude/agents/`)는 모든 프로젝트에서 사용할 수 있는 개인 서브에이전트입니다.

Claude Code는 `.claude/agents/`와 `~/.claude/agents/`를 재귀적으로 스캔하므로, `agents/review/`나 `agents/research/` 같은 하위 폴더로 정의를 정리할 수 있습니다. 하위 디렉터리 경로는 서브에이전트 식별에 영향을 주지 않으며, 식별은 `name` frontmatter 필드로만 결정됩니다. 한 스코프 내에서 두 파일이 같은 `name`을 선언하면 Claude Code는 경고 없이 하나만 유지합니다.

---

## 서브에이전트 파일 작성

서브에이전트 파일은 설정을 위한 YAML frontmatter와 시스템 프롬프트 역할을 하는 Markdown 본문으로 구성됩니다.

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

Frontmatter는 서브에이전트의 메타데이터와 설정을 정의합니다. 본문은 서브에이전트의 행동을 안내하는 시스템 프롬프트가 됩니다. 서브에이전트는 이 시스템 프롬프트(작업 디렉터리 같은 기본 환경 정보 포함)만 수신하며, Claude Code의 전체 시스템 프롬프트는 수신하지 않습니다.

서브에이전트는 메인 대화의 현재 작업 디렉터리에서 시작합니다. 서브에이전트 내에서 `cd` 명령은 Bash 도구 호출 간에 유지되지 않으며 메인 대화의 작업 디렉터리에도 영향을 주지 않습니다. 격리된 저장소 사본을 제공하려면 `isolation: worktree`를 설정하세요.

---

## Frontmatter 필드

| 필드 | 필수 | 설명 |
|------|------|------|
| `name` | **예** | 소문자와 하이픈을 사용한 고유 식별자. Hooks에서 `agent_type`으로 이 값을 수신. 파일명과 일치할 필요 없음 |
| `description` | **예** | Claude가 언제 이 서브에이전트에 위임할지를 설명 |
| `tools` | 아니오 | 서브에이전트가 사용할 수 있는 도구. 생략시 모든 도구 상속. Skills를 컨텍스트에 프리로드하려면 `skills` 필드를 사용 |
| `disallowedTools` | 아니오 | 거부할 도구. 상속 또는 지정된 목록에서 제거 |
| `model` | 아니오 | 사용할 모델: `sonnet`, `opus`, `haiku`, 전체 모델 ID(예: `claude-opus-4-8`), 또는 `inherit`. 기본값 `inherit` |
| `permissionMode` | 아니오 | 권한 모드: `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan`. 플러그인 서브에이전트에는 무시됨 |
| `maxTurns` | 아니오 | 서브에이전트가 중단되기 전 최대 에이전트 턴 수 |
| `skills` | 아니오 | 시작 시 서브에이전트의 컨텍스트에 프리로드할 스킬. 전체 스킬 콘텐츠가 주입됨. 나열되지 않은 스킬도 Skill 도구로 호출 가능 |
| `mcpServers` | 아니오 | 이 서브에이전트가 사용할 수 있는 MCP 서버. 플러그인 서브에이전트에는 무시됨 |
| `hooks` | 아니오 | 이 서브에이전트에 스코프된 라이프사이클 훅. 플러그인 서브에이전트에는 무시됨 |
| `memory` | 아니오 | 영구 메모리 스코프: `user`, `project`, `local`. 세션 간 학습 활성화 |
| `background` | 아니오 | `true`로 설정 시 이 서브에이전트를 항상 백그라운드 작업으로 실행. 기본값: `false` |
| `effort` | 아니오 | 이 서브에이전트가 활성 상태일 때의 effort level. 세션 effort level을 오버라이드. 기본값: 세션에서 상속. 옵션: `low`, `medium`, `high`, `xhigh`, `max` |
| `isolation` | 아니오 | `worktree`로 설정 시 임시 git worktree에서 서브에이전트를 실행. 기본 브랜치에서 브랜치된 격리된 저장소 사본 제공. 변경사항이 없으면 worktree는 자동 정리됨 |
| `color` | 아니오 | 작업 목록과 트랜스크립트에서 서브에이전트의 표시 색상. `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan` |
| `initialPrompt` | 아니오 | 이 에이전트가 메인 세션 에이전트로 실행될 때(`--agent` 또는 `agent` 설정) 첫 번째 사용자 턴으로 자동 제출됨. 명령어와 스킬이 처리됨 |

---

## 모델 선택

`model` 필드는 서브에이전트가 사용할 AI 모델을 제어합니다.

| 설정 방식 | 설명 |
|-----------|------|
| **모델 별칭** | `sonnet`, `opus`, `haiku` 중 하나 사용 |
| **전체 모델 ID** | `claude-opus-4-8`, `claude-sonnet-4-6` 등. `--model` 플래그와 동일한 값 |
| **`inherit`** | 메인 대화와 동일한 모델 사용 |
| **생략** | 기본값 `inherit` (메인 대화와 동일한 모델) |

### 모델 해결 순서

Claude가 서브에이전트를 호출할 때 모델은 다음 순서로 결정됩니다.

```
1. CLAUDE_CODE_SUBAGENT_MODEL 환경 변수 (설정된 경우)
2. 호출 시 전달된 model 파라미터
3. 서브에이전트 정의의 model frontmatter
4. 메인 대화의 모델
```

---

## 도구 접근 제어

서브에이전트는 기본적으로 메인 대화에서 사용 가능한 내부 도구와 MCP 도구를 상속합니다.

### Available tools

도구 접근을 제한하려면 `tools` 필드(허용 목록) 또는 `disallowedTools` 필드(거부 목록)를 사용합니다.

**허용 목록 예시** - `tools`로 지정된 도구만 사용:

```markdown
---
name: safe-researcher
description: Research agent with restricted capabilities
tools: Read, Grep, Glob, Bash
---
```

**거부 목록 예시** - `disallowedTools`로 지정된 도구만 제외:

```markdown
---
name: no-writes
description: Inherits every tool except file writes
disallowedTools: Write, Edit
---
```

> **참고**: 두 필드가 모두 설정된 경우 `disallowedTools`가 먼저 적용된 후 `tools`가 나머지 풀에서 해결됩니다. 양쪽에 모두 나열된 도구는 제거됩니다.

### 특정 서브에이전트 스폰 제한

`claude --agent`로 메인 스레드로 실행되는 에이전트는 Agent 도구로 서브에이전트를 스폰할 수 있습니다. 스폰할 수 있는 서브에이전트 유형을 제한하려면 `tools` 필드에 `Agent(agent_type)` 구문을 사용합니다.

```markdown
---
name: coordinator
description: Coordinates work across specialized agents
tools: Agent(worker, researcher), Read, Bash
---
```

이것은 허용 목록입니다: `worker`와 `researcher` 서브에이전트만 스폰할 수 있습니다. 괄호 없이 `Agent`를 사용하면 모든 서브에이전트를 제한 없이 스폰할 수 있습니다. `tools` 목록에서 `Agent`를 완전히 생략하면 어떤 서브에이전트도 스폰할 수 없습니다.

---

## 사용 불가 도구

다음 도구는 메인 대화의 UI나 세션 상태에 의존하므로 `tools` 필드에 나열해도 서브에이전트에서 사용할 수 없습니다.

| 도구 | 비고 |
|------|------|
| `Agent` | 서브에이전트는 다른 서브에이전트를 스폰할 수 없음 |
| `AskUserQuestion` | 메인 UI에 의존 |
| `EnterPlanMode` | 메인 UI에 의존 |
| `ExitPlanMode` | `permissionMode`가 `plan`인 경우 예외적으로 사용 가능 |
| `ScheduleWakeup` | 메인 세션 스케줄러에 의존 |
| `WaitForMcpServers` | 메인 세션에 의존 |

---

## MCP 서버 스코프

`mcpServers` 필드를 사용하면 메인 대화에서 사용할 수 없는 MCP 서버를 서브에이전트에 제공할 수 있습니다. 인라인으로 정의된 서버는 서브에이전트 시작 시 연결되고 종료 시 해제됩니다. 문자열 참조는 부모 세션의 연결을 공유합니다.

```markdown
---
name: browser-tester
description: Tests features in a real browser using Playwright
mcpServers:
  # 인라인 정의: 이 서브에이전트에만 스코프
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
  # 이름으로 참조: 이미 설정된 서버 재사용
  - github
---

Use the Playwright tools to navigate, screenshot, and interact with pages.
```

인라인 정의는 `.mcp.json` 서버 항목과 동일한 스키마(`stdio`, `http`, `sse`, `ws`)를 사용합니다.

MCP 서버를 메인 대화에서 완전히 제외하고 해당 도구 설명이 컨텍스트를 소모하지 않게 하려면, `.mcp.json` 대신 여기서 인라인으로 정의하세요. 서브에이전트는 도구를 받고, 부모 대화는 받지 않습니다.

### MCP 서버 제한 사항 (v2.1.153+)

v2.1.153부터 메인 세션에 적용되는 MCP 제한이 서브에이전트 frontmatter에 선언된 서버에도 적용됩니다.

- `--strict-mcp-config` 및 `--bare`
- 엔터프라이즈 관리형 MCP 구성
- `allowedMcpServers` 및 `deniedMcpServers` 정책

이 제한들 중 하나가 서버를 차단하면 Claude Code는 해당 서버를 건너뛰고 차단된 서버 이름을 경고로 표시합니다.

**관리 설정 제한**은 서브에이전트가 어떻게 정의되었는지와 관계없이 모든 서브에이전트에 적용됩니다.

**예외**: `--strict-mcp-config`는 `--agents` 또는 SDK `agents` 옵션을 통해 인라인으로 전달된 서버는 필터링하지 않습니다. 이들은 명시적인 호출자 입력으로 간주되기 때문입니다.

---

## 권한 모드

`permissionMode` 필드는 서브에이전트가 권한 프롬프트를 처리하는 방식을 제어합니다. 서브에이전트는 메인 대화의 권한 컨텍스트를 상속하며 모드를 오버라이드할 수 있습니다.

| 모드 | 동작 |
|------|------|
| `default` | 프롬프트와 함께 표준 권한 확인 |
| `acceptEdits` | 작업 디렉터리 또는 `additionalDirectories` 경로의 파일 편집 및 일반 파일시스템 명령 자동 수락 |
| `auto` | 백그라운드 분류기가 명령과 보호 디렉터리 쓰기를 검토 |
| `dontAsk` | 권한 프롬프트 자동 거부 (명시적으로 허용된 도구는 여전히 작동) |
| `bypassPermissions` | 권한 프롬프트 건너뜀 |
| `plan` | Plan 모드 (읽기 전용 탐색) |

> **참고**: 부모가 `bypassPermissions` 또는 `acceptEdits`를 사용하면 이것이 우선하며 오버라이드할 수 없습니다. 부모가 `auto` 모드를 사용하면 서브에이전트도 `auto` 모드를 상속하며 frontmatter의 `permissionMode`는 무시됩니다.

---

## 스킬 프리로드

`skills` 필드를 사용하면 시작 시 서브에이전트의 컨텍스트에 스킬 콘텐츠를 주입할 수 있습니다.

```markdown
---
name: api-developer
description: Implement API endpoints following team conventions
skills:
  - api-conventions
  - error-handling-patterns
---

Implement API endpoints. Follow the conventions and patterns from the preloaded skills.
```

나열된 각 스킬의 전체 콘텐츠가 시작 시 컨텍스트에 주입됩니다. 이 필드는 프리로드할 스킬을 제어할 뿐, 서브에이전트가 접근할 수 있는 스킬을 제한하지는 않습니다. 나열되지 않은 스킬도 실행 중에 Skill 도구로 호출할 수 있습니다.

---

## 영구 메모리

`memory` 필드는 서브에이전트에 세션 간에 유지되는 디렉터리를 부여합니다. 서브에이전트는 이 디렉터리를 사용하여 시간이 지남에 따라 지식을 축적합니다.

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
memory: user
---

You are a code reviewer. As you review code, update your agent memory with
patterns, conventions, and recurring issues you discover.
```

| 스코프 | 위치 | 사용 시기 |
|--------|------|-----------|
| `user` | `~/.claude/agent-memory/<name-of-agent>/` | 모든 프로젝트에 걸쳐 학습 내용을 기억해야 할 때 |
| `project` | `.claude/agent-memory/<name-of-agent>/` | 프로젝트 특화 지식, 버전 관리로 공유 가능 |
| `local` | `.claude/agent-memory-local/<name-of-agent>/` | 프로젝트 특화지만 버전 관리에 체크인하지 않을 때 |

메모리가 활성화되면:
- 서브에이전트의 시스템 프롬프트에 메모리 디렉터리 읽기/쓰기 지침이 포함됨
- 메모리 디렉터리의 `MEMORY.md` 첫 200줄 또는 25KB(먼저 도달하는 쪽)가 시스템 프롬프트에 포함됨
- Read, Write, Edit 도구가 자동으로 활성화되어 메모리 파일 관리 가능

---

## Isolation (Worktree) 모드

`isolation` 필드를 `worktree`로 설정하면 서브에이전트가 임시 git worktree에서 실행됩니다. 이것은 부모 세션의 `HEAD`가 아닌 기본 브랜치에서 브랜치된 저장소의 격리된 사본을 서브에이전트에 제공합니다.

```markdown
---
name: safe-experimenter
description: Experiments with changes in isolation
isolation: worktree
tools: Read, Edit, Write, Bash
---
```

서브에이전트가 변경사항을 만들지 않으면 worktree는 자동으로 정리됩니다. 변경사항이 있는 경우 worktree가 유지됩니다.

---

## Hooks로 조건부 제어

### 서브에이전트 Frontmatter의 Hooks

서브에이전트의 Markdown 파일에 직접 hooks를 정의할 수 있습니다. 이 hooks는 해당 서브에이전트가 활성 상태일 때만 실행되며, 완료되면 정리됩니다.

| 이벤트 | Matcher 입력 | 실행 시점 |
|--------|-------------|-----------|
| `PreToolUse` | 도구 이름 | 서브에이전트가 도구를 사용하기 전 |
| `PostToolUse` | 도구 이름 | 서브에이전트가 도구를 사용한 후 |
| `Stop` | (없음) | 서브에이전트가 완료될 때 (런타임에 `SubagentStop`으로 변환) |

```markdown
---
name: code-reviewer
description: Review code changes with automatic linting
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh $TOOL_INPUT"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
---
```

### 프로젝트 레벨 서브에이전트 이벤트 Hooks

`settings.json`에서 메인 세션의 서브에이전트 라이프사이클 이벤트에 응답하는 hooks를 구성할 수 있습니다.

| 이벤트 | Matcher 입력 | 실행 시점 |
|--------|-------------|-----------|
| `SubagentStart` | 에이전트 타입 이름 | 서브에이전트가 실행을 시작할 때 |
| `SubagentStop` | 에이전트 타입 이름 | 서브에이전트가 완료될 때 |

```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [
          { "type": "command", "command": "./scripts/setup-db-connection.sh" }
        ]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [
          { "type": "command", "command": "./scripts/cleanup-db-connection.sh" }
        ]
      }
    ]
  }
}
```

---

## CLI를 통한 동적 정의

CLI를 통해 JSON 형식으로 서브에이전트를 동적으로 정의할 수 있습니다. `--agents` 플래그를 사용합니다. CLI 정의 서브에이전트는 해당 세션에만 존재하며 디스크에 저장되지 않습니다.

**macOS / Linux / WSL:**

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  },
  "debugger": {
    "description": "Debugging specialist for errors and test failures.",
    "prompt": "You are an expert debugger. Analyze errors, identify root causes, and provide fixes."
  }
}'
```

`--agents` 플래그는 파일 기반 서브에이전트와 동일한 frontmatter 필드를 JSON으로 받습니다: `description`, `prompt`, `tools`, `disallowedTools`, `model`, `permissionMode`, `mcpServers`, `hooks`, `maxTurns`, `skills`, `initialPrompt`, `memory`, `effort`, `background`, `isolation`, `color`.

---

## 특정 서브에이전트 비활성화

Claude가 특정 서브에이전트를 사용하지 못하도록 설정의 `deny` 배열에 추가할 수 있습니다. `Agent(subagent-name)` 형식을 사용합니다.

```json
{
  "permissions": {
    "deny": ["Agent(Explore)", "Agent(my-custom-agent)"]
  }
}
```

빌트인 및 커스텀 서브에이전트 모두에 작동합니다. CLI 플래그로도 가능합니다:

```bash
claude --disallowedTools "Agent(Explore)"
```

---

## 명시적 서브에이전트 호출

자동 위임만으로 충분하지 않을 때, 직접 서브에이전트를 요청할 수 있습니다. 세 가지 패턴이 있으며, 일회성 제안에서 세션 전체 기본값까지 점진적으로 escalation됩니다.

### 자연어 호출

특별한 구문 없이 서브에이전트 이름을 언급하면 Claude가 위임 여부를 결정합니다.

```
test-runner 서브에이전트로 실패한 테스트를 수정해줘
code-reviewer 서브에이전트가 내 최근 변경사항을 검토하게 해줘
debugger 서브에이전트에게 이 에러를 조사해달라고 해줘
```

### @-멘션 호출

`@`를 입력하고 타입어헤드에서 서브에이전트를 선택합니다. 파일을 @-멘션하는 것과 같은 방식입니다. 이 방식은 Claude가 아닌 특정 서브에이전트가 실행되도록 보장합니다.

```
@"code-reviewer (agent)" auth 변경사항을 검토해줘
```

전체 메시지는 여전히 Claude에게 전달되며, Claude는 요청 내용을 기반으로 서브에이전트의 작업 프롬프트를 작성합니다. @-멘션은 Claude가 어떤 서브에이전트를 호출할지 제어할 뿐, 서브에이전트가 받는 프롬프트를 제어하지는 않습니다.

**플러그인 서브에이전트**가 활성화된 경우, 타입어헤드에 스코프된 이름으로 나타납니다. 예: `my-plugin:code-reviewer` 또는 `my-plugin:review:security` (플러그인이 에이전트를 하위 폴더로 구성한 경우).

**수동 입력**도 가능합니다: 로컬 서브에이전트는 `@agent-<name>`, 플러그인 서브에이전트는 `@agent-` 뒤에 스코프된 이름을 입력합니다. 예: `@agent-my-plugin:code-reviewer`.

현재 세션에서 실행 중인 이름이 있는 백그라운드 서브에이전트도 타입어헤드에 나타나며, 이름 옆에 상태가 표시됩니다.

### 세션 전체 서브에이전트 실행

`--agent <name>` 플래그를 전달하면 메인 스레드 자체가 해당 서브에이전트의 시스템 프롬프트, 도구 제한, 모델을 사용하는 세션이 시작됩니다.

```bash
claude --agent code-reviewer
```

서브에이전트의 시스템 프롬프트가 기본 Claude Code 시스템 프롬프트를 완전히 대체합니다. `--system-prompt`와 동일한 방식입니다. `CLAUDE.md` 파일과 프로젝트 메모리는 정상적인 메시지 흐름을 통해 여전히 로드됩니다. 에이전트 이름이 시작 헤더에 `@<name>`으로 표시되어 활성 상태를 확인할 수 있습니다.

빌트인 및 커스텀 서브에이전트 모두 작동하며, 세션을 재개할 때도 선택이 유지됩니다.

**플러그인 서브에이전트**의 경우 에이전트 이름만 전달하면 Claude Code가 찾습니다:

```bash
claude --agent security-reviewer
```

여러 플러그인이 같은 이름의 에이전트를 제공하는 경우, 스코프된 이름으로 구분합니다:

```bash
claude --agent my-plugin:security-reviewer
```

플러그인이 에이전트를 `agents/` 디렉터리의 하위 폴더에 배치한 경우, 하위 폴더를 스코프된 이름에 포함합니다:

```bash
claude --agent my-plugin:review:security
```

### settings.json의 agent 설정

프로젝트의 모든 세션에 기본값으로 설정하려면 `.claude/settings.json`에 `agent`를 설정합니다:

```json
{
  "agent": "code-reviewer"
}
```

CLI 플래그와 설정이 모두 있는 경우 CLI 플래그가 우선합니다.

---

## 서브에이전트 작업 패턴

### 대용량 출력 격리

서브에이전트의 가장 효과적인 용도 중 하나는 대량의 출력을 생성하는 작업을 격리하는 것입니다. 테스트 실행, 문서 가져오기, 로그 파일 처리 등은 상당한 컨텍스트를 소모할 수 있습니다. 서브에이전트에 위임하면 장황한 출력은 서브에이전트의 컨텍스트에 남고 관련 요약만 메인 대화로 반환됩니다.

### 병렬 리서치

독립적인 조사의 경우 여러 서브에이전트를 동시에 스폰하여 병렬로 작업할 수 있습니다.

```
인증, 데이터베이스, API 모듈을 각각 별도의 서브에이전트로 병렬로 리서치하세요
```

### 서브에이전트 체이닝

다단계 워크플로우의 경우 서브에이전트를 순차적으로 연결할 수 있습니다. 각 서브에이전트가 작업을 완료하고 결과를 Claude에 반환하면, Claude가 관련 컨텍스트를 다음 서브에이전트에 전달합니다.

```
code-reviewer 서브에이전트로 성능 이슈를 찾고, optimizer 서브에이전트로 수정하세요
```

### 메인 대화 vs 서브에이전트 선택 기준

**메인 대화**를 사용할 때:
- 잦은 피드백이나 반복적인 정제가 필요한 작업
- 여러 단계가 상당한 컨텍스트를 공유하는 경우 (계획 -> 구현 -> 테스트)
- 빠르고 타겟팅된 변경
- 지연 시간이 중요한 경우

**서브에이전트**를 사용할 때:
- 메인 컨텍스트에 필요 없는 장황한 출력을 생성하는 작업
- 특정 도구 제한이나 권한을 강제해야 하는 경우
- 독립적이고 요약으로 반환할 수 있는 작업

---

## 시작 시 로드되는 항목

각 서브에이전트는 깨끗하고 격리된 컨텍스트 윈도우로 시작합니다. 대화 기록, 이미 호출한 스킬, Claude가 이미 읽은 파일을 볼 수 없습니다. Claude는 작업을 요약하는 위임 메시지를 작성하고, 서브에이전트는 그 메시지를 기반으로 작업합니다. 예외는 fork이며, fork는 새로 시작하는 대신 부모 대화를 상속합니다.

Non-fork 서브에이전트의 초기 컨텍스트에는 다음이 포함됩니다.

### 시스템 프롬프트

에이전트 자체 프롬프트와 Claude Code가 추가하는 환경 세부정보. Claude Code의 전체 시스템 프롬프트가 아닙니다. 커스텀 서브에이전트는 markdown 본문 또는 `prompt` 필드에서 자체 프롬프트를 정의합니다. 빌트인 에이전트는 미리 정의된 프롬프트를 사용합니다.

### 작업 메시지

Claude가 작업을 전달할 때 작성하는 위임 프롬프트입니다.

### CLAUDE.md 및 메모리

메인 대화가 로드하는 모든 수준의 메모리 계층이 포함됩니다: `~/.claude/CLAUDE.md`, 프로젝트 규칙, `CLAUDE.local.md`, 관리 정책 파일. **빌트인 Explore 및 Plan 에이전트는 이 항목을 건너뜁니다.**

### Git 상태

부모 세션 시작 시점의 스냅샷입니다. 작업 디렉터리가 Git 저장소가 아니거나 `includeGitInstructions`가 `false`인 경우 제외됩니다. **Explore와 Plan은 관계없이 이 항목을 건너뜁니다.**

### 프리로드된 스킬

에이전트의 `skills` 필드에 명명된 스킬의 전체 콘텐츠. 빌트인 에이전트는 스킬을 프리로드하지 않습니다.

### Explore와 Plan의 예외 처리

Explore와 Plan은 CLAUDE.md와 git status를 생략하는 유일한 서브에이전트입니다. 이 동작을 변경하는 frontmatter 필드나 에이전트별 설정은 없습니다.

**중요**: 메인 대화는 Explore와 Plan의 결과를 전체 CLAUDE.md 컨텍스트와 함께 읽으므로, 대부분의 규칙은 서브에이전트 자체에 도달할 필요가 없습니다. 만약 규칙이 서브에이전트에 반드시 전달되어야 하는 경우 (예: "`vendor/` 디렉터리는 무시"), Claude에게 작업을 위임할 때 프롬프트에 해당 규칙을 직접 명시하세요.

---

## 포그라운드와 백그라운드 실행

서브에이전트는 포그라운드(블로킹) 또는 백그라운드(동시 실행)로 실행할 수 있습니다.

| 모드 | 동작 |
|------|------|
| **포그라운드** | 메인 대화를 완료될 때까지 차단. 권한 프롬프트가 사용자에게 전달됨 |
| **백그라운드** | 작업을 계속하면서 동시에 실행. 이미 부여된 권한으로 실행되며 프롬프트가 필요한 도구 호출은 자동 거부됨 |

백그라운드 서브에이전트가 권한 부족으로 실패하면, 동일한 작업으로 새로운 포그라운드 서브에이전트를 시작하여 대화형 프롬프트로 재시도할 수 있습니다.

수동 제어:
- Claude에게 "백그라운드에서 실행해"라고 요청
- 실행 중인 작업에 **Ctrl+B**를 눌러 백그라운드로 전환

`CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` 환경 변수를 `1`로 설정하면 모든 백그라운드 작업 기능을 비활성화할 수 있습니다.

---

## 서브에이전트 재개

각 서브에이전트 호출은 새로운 컨텍스트의 새로운 인스턴스를 생성합니다. 기존 서브에이전트의 작업을 계속하려면 Claude에게 재개를 요청하세요.

재개된 서브에이전트는 이전의 모든 도구 호출, 결과, 추론을 포함한 전체 대화 기록을 유지합니다. 정확히 중단한 지점에서 이어서 작업합니다.

서브에이전트가 완료되면 Claude는 해당 에이전트 ID를 수신합니다. Claude는 `SendMessage` 도구를 사용하여 에이전트 ID를 `to` 필드에 지정하여 서브에이전트를 재개합니다.

> **중요**: `SendMessage` 도구는 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 환경 변수를 통해 에이전트 팀이 활성화된 경우에만 사용할 수 있습니다. 이 설정 없이는 재개 기능을 사용할 수 없습니다.

서브에이전트를 재개하려면 Claude에게 이전 작업을 계속하라고 요청하세요:

```
code-reviewer 서브에이전트로 인증 모듈을 리뷰하세요
[에이전트 완료]

그 코드 리뷰를 이어서 인가 로직도 분석하세요
[Claude가 이전 컨텍스트를 가진 서브에이전트를 재개]
```

중단된 서브에이전트가 `SendMessage`를 수신하면 새로운 `Agent` 호출 없이 백그라운드에서 자동 재개됩니다.

에이전트 ID를 명시적으로 참조하고 싶다면 Claude에게 요청할 수도 있고, `~/.claude/projects/{project}/{sessionId}/subagents/` 경로의 트랜스크립트 파일에서 확인할 수도 있습니다. 각 트랜스크립트는 `agent-{agentId}.jsonl` 형식으로 저장됩니다.

서브에이전트 트랜스크립트는 메인 대화와 독립적으로 유지됩니다:
- 메인 대화 압축 시 서브에이전트 트랜스크립트는 영향을 받지 않음
- 세션 내에서 지속되며, Claude Code를 재시작해도 동일한 세션을 재개하면 서브에이전트를 계속 사용할 수 있음
- `cleanupPeriodDays` 설정(기본값 30일)에 따라 자동 정리됨

### 자동 압축 (Auto-compaction)

서브에이전트는 메인 대화와 동일한 로직을 사용하여 자동 압축을 지원합니다. 기본적으로 약 95% 용량에서 자동 압축이 트리거됩니다. 더 일찍 압축을 트리거하려면 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 환경 변수를 더 낮은 백분율(예: `50`)로 설정하세요.

압축 이벤트는 서브에이전트 트랜스크립트 파일에 기록됩니다:

```json
{
  "type": "system",
  "subtype": "compact_boundary",
  "compactMetadata": {
    "trigger": "auto",
    "preTokens": 167189
  }
}
```

`preTokens` 값은 압축 발생 전 사용된 토큰 수를 나타냅니다.

---

## 대화 Fork

Fork는 새로 시작하는 대신 지금까지의 전체 대화를 상속하는 서브에이전트입니다. 이렇게 하면 서브에이전트가 일반적으로 제공하는 입력 격리가 사라집니다. Fork는 메인 세션과 동일한 시스템 프롬프트, 도구, 모델, 메시지 기록을 볼 수 있습니다.

`CLAUDE_CODE_FORK_SUBAGENT`를 설정하면 Claude Code가 두 가지 방식으로 변경됩니다:
- Claude가 일반적으로 general-purpose 서브에이전트를 사용할 때마다 대신 fork를 스폰합니다. Explore 같은 네임드 서브에이전트는 기존과 같이 스폰됩니다.
- 모든 서브에이전트 스폰이 백그라운드에서 실행됩니다.

직접 fork를 시작하려면 `/fork` 뒤에 지시문을 입력합니다:

```
/fork draft unit tests for the parser changes so far
```

### Fork vs 네임드 서브에이전트 비교

| 항목 | Fork | 네임드 서브에이전트 |
|------|------|---------------------|
| 컨텍스트 | 전체 대화 기록 | 전달된 프롬프트만으로 새로운 컨텍스트 |
| 시스템 프롬프트/도구 | 메인 세션과 동일 | 서브에이전트 정의 파일에서 가져옴 |
| 모델 | 메인 세션과 동일 | 서브에이전트의 `model` 필드에서 가져옴 |
| 권한 | 터미널에 프롬프트 표시 | 백그라운드 실행 시 자동 거부 |
| 프롬프트 캐시 | 메인 세션과 공유 | 별도 캐시 |

---

## 서브에이전트 제약사항

> **중요**: 서브에이전트는 다른 서브에이전트를 생성할 수 **없습니다**.

이 제한은 무한 중첩(Infinite Nesting)을 방지하기 위해 존재합니다. 서브에이전트는 1단계까지만 허용됩니다.

```
메인 대화 → 서브에이전트 (가능)
서브에이전트 → 또 다른 서브에이전트 (불가능)
```

---

## 실전 예시

### 코드 리뷰어

```markdown
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code is clear and readable
- Functions and variables are well-named
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Input validation implemented
- Good test coverage
- Performance considerations addressed

Provide feedback organized by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)

Include specific examples of how to fix issues.
```

### 디버거

```markdown
---
name: debugger
description: Debugging specialist for errors, test failures, and unexpected behavior. Use proactively when encountering any issues.
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate the failure location
4. Implement minimal fix
5. Verify solution works

Debugging process:
- Analyze error messages and logs
- Check recent code changes
- Form and test hypotheses
- Add strategic debug logging
- Inspect variable states

For each issue, provide:
- Root cause explanation
- Evidence supporting the diagnosis
- Specific code fix
- Testing approach
- Prevention recommendations

Focus on fixing the underlying issue, not the symptoms.
```

### 데이터 과학자

```markdown
---
name: data-scientist
description: Data analysis expert for SQL queries, BigQuery operations, and data insights. Use proactively for data analysis tasks and queries.
tools: Bash, Read, Write
model: sonnet
---

You are a data scientist specializing in SQL and BigQuery analysis.

When invoked:
1. Understand the data analysis requirement
2. Write efficient SQL queries
3. Use BigQuery command line tools (bq) when appropriate
4. Analyze and summarize results
5. Present findings clearly

Key practices:
- Write optimized SQL queries with proper filters
- Use appropriate aggregations and joins
- Include comments explaining complex logic
- Format results for readability
- Provide data-driven recommendations
```

### DB 쿼리 검증기

```markdown
---
name: db-reader
description: Execute read-only database queries. Use when analyzing data or generating reports.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database analyst with read-only access. Execute SELECT queries to answer questions about the data.
```

---

## 모범 사례

### 1. Claude로 초기 생성 후 반복 개선

처음에는 Claude를 통해 서브에이전트를 생성하고, 이를 개인화하며 발전시키세요. 이 방식이 가장 좋은 결과를 얻을 수 있습니다.

### 2. 단일 책임 원칙

한 가지 일만 하는 집중된 서브에이전트를 만드세요. 하나의 서브에이전트가 모든 것을 하려고 하면 성능과 예측 가능성이 떨어집니다.

### 3. 상세한 프롬프트 작성

구체적인 지침, 예시, 제약사항을 시스템 프롬프트에 포함하세요. 가이드가 명확할수록 성능이 향상됩니다.

### 4. 도구 접근 최소화

서브에이전트의 목적에 필요한 도구만 부여하세요. 보안이 강화되고 서브에이전트가 관련 작업에 집중할 수 있습니다.

### 5. 버전 관리

프로젝트 서브에이전트를 버전 관리에 체크인하세요. 팀 전체가 혜택을 받고 공동으로 개선할 수 있습니다.

### 서브에이전트 체이닝

복잡한 워크플로우의 경우 여러 서브에이전트를 연결할 수 있습니다.

```
> 먼저 code-analyzer 서브에이전트로 성능 이슈를 찾고,
> 그 다음 optimizer 서브에이전트로 수정하세요
```

### 성능 고려사항

| 항목 | 설명 |
|------|------|
| **컨텍스트 효율성** | 서브에이전트는 메인 컨텍스트를 보존하여 더 긴 세션을 가능하게 함 |
| **지연 시간** | 서브에이전트는 매번 깨끗한 상태에서 시작하므로 필요한 컨텍스트를 수집하는 데 지연이 추가될 수 있음 |

---

## 요약

서브에이전트는 Claude Code의 강력한 기능으로, 특정 작업에 특화된 AI 어시스턴트를 통해 효율적인 문제 해결이 가능합니다. 명확한 역할 정의, 적절한 도구 제한, 상세한 프롬프트 작성이 핵심입니다. 커스텀 서브에이전트를 만들어 프로젝트나 조직의 요구에 맞는 특화된 워크플로우를 구축할 수 있습니다.
