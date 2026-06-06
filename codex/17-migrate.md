# 17. 마이그레이션 가이드

> **출처**
> - [Migrate to Codex](https://developers.openai.com/codex/migrate) — 마이그레이션 개요, import flow, 항목별 매핑
> - [Configuration Reference](https://developers.openai.com/codex/config-reference) — `config.toml` / `requirements.toml` 전체 스키마
> - [MCP](https://developers.openai.com/codex/mcp) — MCP 서버 설정
> - [Hooks](https://developers.openai.com/codex/hooks) — 훅 설정 이전
> - [Skills](https://developers.openai.com/codex/skills) — 스킬 변환
> - [Subagents](https://developers.openai.com/codex/subagents) — 서브에이전트 / 커스텀 에이전트 설정

---

## 1. 마이그레이션 개요

Codex는 다른 에이전트 도구에서 사용하던 설정을 자동으로 감지하고 가져오는 **import flow**를 제공한다. 사용자 수준(user-level) 설정과 프로젝트 수준(project-level) 설정을 모두 검사하며, 1:1 매핑이 가능한 항목은 자동 변환하고, 나머지는 후속 스레드에서 도움을 받을 수 있다.

### 마이그레이션 실행 절차

1. Codex 앱에서 **Settings** 열기
2. **General** 페이지에서 **Import other agent setup** 찾기
3. **Import** 또는 **Import again** 선택
4. Codex가 감지한 항목을 검토 후 가져올 항목 선택 → **Import**
5. 완료 후 **View imported files**로 결과 확인

### 마이그레이션 동작 흐름

| 단계 | 설명 |
|------|------|
| 1 | 사용자 및 프로젝트의 기존 설정을 자동 감지 |
| 2 | 선택한 항목을 Codex 형식으로 직접 변환 |
| 3 | 변환 완료 후 다시 한번 미변환 항목이 있는지 확인 |
| 4 | 남은 항목은 `migrate-to-codex` 스킬이 포함된 새 스레드에서 후속 작업 제안 |

---

## 2. 항목별 변환 매핑

Codex가 감지하는 외부 설정과 해당 Codex 목적지는 다음과 같다.

| 감지된 설정 | Codex 목적지 | 비고 |
|---|---|---|
| Instruction files (`CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md` 등) | `AGENTS.md` | 프로젝트 루트에 생성 |
| `settings.json` | `config.toml` | `~/.codex/config.toml` (사용자) / `.codex/config.toml` (프로젝트) |
| Skills | Codex skills | `.agents/skills/` 디렉터리에 `SKILL.md` 기반 스킬 생성 |
| 최근 30일 세션 | Codex threads & projects | 세션 기록을 Codex 스레드로 가져옴 |
| MCP server 설정 | Codex MCP configuration | `config.toml`의 `[mcp_servers.*]` 테이블로 변환 |
| Hooks | Codex hooks | `hooks.json` 또는 `config.toml` 인라인 `[hooks]` |
| Slash commands | Codex skills | 슬래시 명령어를 스킬로 변환 |
| Subagents | Codex agents | `.codex/agents/*.toml` 파일로 변환 |

---

## 3. Instruction 파일 변환 (→ `AGENTS.md`)

### 변환 대상 파일

| 원본 에이전트 | Instruction 파일 | Codex 목적지 |
|---|---|---|
| Claude Code | `CLAUDE.md` | `AGENTS.md` |
| Cursor | `.cursorrules` | `AGENTS.md` |
| GitHub Copilot | `.github/copilot-instructions.md` | `AGENTS.md` |
| Windsurf | `.windsurfrules` | `AGENTS.md` |
| Aider | `.aider.conf.yml` (convention 섹션) | `AGENTS.md` |

### 변환 예시

**원본** — `CLAUDE.md`:

```markdown
# Project Conventions

- Use TypeScript strict mode
- All API calls go through `src/api/` layer
- Test files colocated: `Component.test.tsx`
```

**변환 결과** — `AGENTS.md` (프로젝트 루트):

```markdown
# Project Conventions

- Use TypeScript strict mode
- All API calls go through `src/api/` layer
- Test files colocated: `Component.test.tsx`
```

> **참고**: 내용 자체는 대부분 그대로 이전된다. Codex는 `AGENTS.md`를 프로젝트 문서로 자동 로드하며, `project_doc_max_bytes` 설정으로 최대 읽기 크기를 제한할 수 있다. `AGENTS.md`가 없는 경우 `project_doc_fallback_filenames`에 대체 파일명을 지정할 수 있다.

---

## 4. 설정 마이그레이션 (→ `config.toml`)

### 핵심 경로

| 범위 | Codex 경로 | 설명 |
|---|---|---|
| 사용자 수준 | `~/.codex/config.toml` | 전역 설정 |
| 프로젝트 수준 | `.codex/config.toml` | 프로젝트별 오버라이드 (신뢰된 프로젝트만 로드) |
| 관리자 강제 | `requirements.toml` | 사용자가 변경할 수 없는 보안 설정 |

### 주요 설정 대응표

| 원본 (`settings.json`) | Codex (`config.toml`) | 설명 |
|---|---|---|
| `model` | `model` | 모델 지정 (예: `model = "gpt-5.5"`) |
| `permissions.allow` | `approval_policy` | `"untrusted"`, `"on-request"`, `"never"` 또는 `{ granular = { ... } }` |
| `sandbox` | `sandbox_mode` | `"read-only"`, `"workspace-write"`, `"danger-full-access"` |
| `theme` | `tui.theme` | TUI 테마 (kebab-case) |
| `file_opener` | `file_opener` | `"vscode"`, `"cursor"`, `"windsurf"`, `"none"` |
| `web_search` | `web_search` | `"disabled"`, `"cached"`, `"live"` |
| `hooks` | `[hooks]` 인라인 또는 `hooks.json` | 동일한 이벤트 스키마 사용 |
| `mcpServers` | `[mcp_servers.*]` | MCP 서버 설정 테이블 |
| `instructions` | `developer_instructions` 또는 `AGENTS.md` | 세션에 주입되는 추가 지침 |
| N/A | `model_reasoning_effort` | `"minimal"`, `"low"`, `"medium"`, `"high"`, `"xhigh"` |
| N/A | `features.hooks` | `true`/`false`로 훅 활성화 (기본값: 활성) |
| N/A | `features.memories` | Memories 기능 (기본값: 비활성) |

### `config.toml` 작성 예시

```toml
#:schema https://developers.openai.com/codex/config-schema.json

model = "gpt-5.5"
sandbox_mode = "workspace-write"
web_search = "cached"
file_opener = "vscode"

[features]
hooks = true
memories = false
multi_agent = true

[sandbox_workspace_write]
network_access = true
writable_roots = ["/tmp/my-project"]
```

### 프로젝트 수준 설정 제한

프로젝트 수준 `.codex/config.toml`에서는 다음 키가 **무시**된다 (사용자 수준에만 설정 가능):

- `openai_base_url`, `chatgpt_base_url`
- `model_provider`, `model_providers`
- `notify`
- `profile`, `profiles`
- `otel` (OpenTelemetry)

---

## 5. 스킬 변환

### Codex 스킬 구조

```
my-skill/
  SKILL.md          # 필수: 지침 + 메타데이터
  scripts/          # 선택: 실행 스크립트
  references/       # 선택: 문서
  assets/           # 선택: 템플릿, 리소스
  agents/
    openai.yaml     # 선택: UI 메타데이터, 정책, 의존성
```

### `SKILL.md` 최소 형식

```markdown
---
name: my-skill-name
description: 이 스킬이 언제 트리거되고 무엇을 하는지 설명.
---

스킬 지침 본문. Codex가 이 스킬을 선택했을 때 따라야 할 절차.
```

### 스킬 검색 경로

| 범위 | 경로 | 용도 |
|---|---|---|
| `REPO` | `$CWD/.agents/skills` | 현재 작업 디렉터리 기준 |
| `REPO` | `$REPO_ROOT/.agents/skills` | 저장소 최상단 |
| `USER` | `$HOME/.agents/skills` | 사용자 전역 |
| `ADMIN` | `/etc/codex/skills` | 머신 전체 |
| `SYSTEM` | Codex 번들 | OpenAI 제공 기본 스킬 |

### 스킬 비활성화

삭제하지 않고 `config.toml`에서 비활성화할 수 있다.

```toml
[[skills.config]]
path = "/path/to/skill/SKILL.md"
enabled = false
```

### `openai.yaml` 메타데이터 예시

```yaml
interface:
  display_name: "PR Review"
  short_description: "PR 리뷰 자동화"
  icon_small: "./assets/small-logo.svg"
  brand_color: "#3B82F6"

policy:
  allow_implicit_invocation: false

dependencies:
  tools:
    - type: "mcp"
      value: "openaiDeveloperDocs"
      transport: "streamable_http"
      url: "https://developers.openai.com/mcp"
```

---

## 6. MCP 서버 설정 이전

### CLI로 MCP 서버 추가

```bash
# STDIO 서버
codex mcp add context7 -- npx -y @upstash/context7-mcp

# 환경 변수 포함
codex mcp add <server-name> --env VAR1=VALUE1 --env VAR2=VALUE2 -- <command>
```

### `config.toml` STDIO 서버 예시

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
env_vars = ["LOCAL_TOKEN"]

[mcp_servers.context7.env]
MY_ENV_VAR = "MY_ENV_VALUE"
```

### `config.toml` Streamable HTTP 서버 예시

```toml
[mcp_servers.figma]
url = "https://mcp.figma.com/mcp"
bearer_token_env_var = "FIGMA_OAUTH_TOKEN"
http_headers = { "X-Figma-Region" = "us-east-1" }
```

### 주요 MCP 설정 필드

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `command` | `string` | STDIO | 서버 시작 명령 |
| `args` | `array<string>` | 아니요 | 명령 인자 |
| `url` | `string` | HTTP | 서버 주소 |
| `env` | `map<string,string>` | 아니요 | 서버에 전달할 환경 변수 |
| `env_vars` | `array` | 아니요 | 허용할 환경 변수 목록 |
| `startup_timeout_sec` | `number` | 아니요 | 시작 타임아웃 (기본: 10초) |
| `tool_timeout_sec` | `number` | 아니요 | 툴 실행 타임아웃 (기본: 60초) |
| `enabled` | `boolean` | 아니요 | `false`면 삭제 없이 비활성화 |
| `required` | `boolean` | 아니요 | `true`면 초기화 실패 시 시작 중단 |
| `enabled_tools` | `array<string>` | 아니요 | 툴 허용 목록 |
| `disabled_tools` | `array<string>` | 아니요 | 툴 차단 목록 (`enabled_tools` 이후 적용) |
| `default_tools_approval_mode` | `string` | 아니요 | `"auto"`, `"prompt"`, `"approve"` |
| `bearer_token_env_var` | `string` | 아니요 | Bearer 토큰 환경 변수명 |
| `http_headers` | `map` | 아니요 | 정적 HTTP 헤더 |
| `env_http_headers` | `map` | 아니요 | 환경 변수 기반 HTTP 헤더 |

---

## 7. 훅 설정 이전

### 훅 파일 위치

| 위치 | 파일 형식 | 비고 |
|---|---|---|
| `~/.codex/hooks.json` | JSON | 사용자 수준 |
| `~/.codex/config.toml` | TOML 인라인 `[hooks]` | 사용자 수준 |
| `.codex/hooks.json` | JSON | 프로젝트 수준 (신뢰된 프로젝트) |
| `.codex/config.toml` | TOML 인라인 `[hooks]` | 프로젝트 수준 |

> 하나의 설정 레이어에 `hooks.json`과 인라인 `[hooks]`가 모두 있으면 Codex가 병합하며 시작 시 경고를 표시한다. 레이어당 하나의 표현을 사용하는 것을 권장한다.

### 훅 비활성화

```toml
[features]
hooks = false
```

### JSON 형식 예시

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.codex/hooks/pre_tool_use_policy.py",
            "statusMessage": "Bash 명령 검사 중"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.codex/hooks/stop_continue.py",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### 인라인 TOML 형식 예시

```toml
[[hooks.PreToolUse]]
matcher = "^Bash$"

[[hooks.PreToolUse.hooks]]
type = "command"
command = 'python3 "$(git rev-parse --show-toplevel)/.codex/hooks/pre_tool_use_policy.py"'
timeout = 30
statusMessage = "Bash 명령 검사 중"

[[hooks.PostToolUse]]
matcher = "^Bash$"

[[hooks.PostToolUse.hooks]]
type = "command"
command = 'python3 "$(git rev-parse --show-toplevel)/.codex/hooks/post_tool_use_review.py"'
timeout = 30
statusMessage = "Bash 출력 리뷰 중"
```

### 지원되는 훅 이벤트

| 이벤트 | `matcher` 필터 대상 | 비고 |
|---|---|---|
| `PreToolUse` | tool name | Bash, `apply_patch`, MCP 툴 이름 |
| `PermissionRequest` | tool name | 승인 전 자동 허용/거부 가능 |
| `PostToolUse` | tool name | 툴 실행 후 컨텍스트 추가 가능 |
| `PreCompact` | compaction trigger | `"manual"` / `"auto"` |
| `PostCompact` | compaction trigger | `"manual"` / `"auto"` |
| `SessionStart` | start source | `"startup"`, `"resume"`, `"clear"`, `"compact"` |
| `SubagentStart` | subagent type | 서브에이전트 시작 시 |
| `SubagentStop` | subagent type | 서브에이전트 종료 시 |
| `UserPromptSubmit` | 미지원 | `matcher` 무시됨 |
| `Stop` | 미지원 | `matcher` 무시됨 |

---

## 8. 슬래시 명령어 이전

Codex는 슬래시 명령어를 **스킬**로 변환한다. 변환된 스킬은 `.agents/skills/` 디렉터리에 `SKILL.md` 파일로 저장된다.

### 변환 흐름

```
외부 에이전트 슬래시 명령어
  → Codex 스킬 (SKILL.md)
  → .agents/skills/<skill-name>/SKILL.md
```

### 사용 방법

- **명시적 호출**: CLI/IDE에서 `/skills` 실행 후 선택, 또는 `$`로 스킬 멘션
- **암시적 호출**: 작업 설명이 스킬 `description`과 일치하면 Codex가 자동으로 선택

---

## 9. 서브에이전트 설정 이전

### 커스텀 에이전트 파일 위치

| 범위 | 경로 |
|---|---|
| 사용자 수준 | `~/.codex/agents/*.toml` |
| 프로젝트 수준 | `.codex/agents/*.toml` |

### 커스텀 에이전트 파일 스키마

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `name` | `string` | 예 | Codex가 에이전트를 식별/참조할 때 사용하는 이름 |
| `description` | `string` | 예 | Codex가 이 에이전트를 언제 사용할지 판단하는 지침 |
| `developer_instructions` | `string` | 예 | 에이전트의 핵심 동작을 정의하는 지침 |
| `nickname_candidates` | `array<string>` | 아니요 | UI에 표시할 표시명 후보 |

선택 필드로 `model`, `model_reasoning_effort`, `sandbox_mode`, `mcp_servers`, `skills.config`를 포함할 수 있다. 생략하면 부모 세션에서 상속한다.

### 전역 에이전트 설정 (`config.toml`)

| 필드 | 타입 | 기본값 | 설명 |
|---|---|---|---|
| `agents.max_threads` | `number` | `6` | 동시에 열 수 있는 에이전트 스레드 상한 |
| `agents.max_depth` | `number` | `1` | 중첩 깊이 (루트 = 0). 깊이를 높이면 토큰 사용량·지연·리소스 소모 증가 |
| `agents.job_max_runtime_seconds` | `number` | `1800` | `spawn_agents_on_csv` 작업의 기본 워커 타임아웃 |

### 커스텀 에이전트 예시 — PR 리뷰

`.codex/agents/reviewer.toml`:

```toml
name = "reviewer"
description = "PR reviewer focused on correctness, security, and missing tests."
model = "gpt-5.4"
model_reasoning_effort = "high"
sandbox_mode = "read-only"
developer_instructions = """
Review code like an owner.
Prioritize correctness, security, behavior regressions, and missing test coverage.
Lead with concrete findings, include reproduction steps when possible.
"""
```

`.codex/agents/pr-explorer.toml`:

```toml
name = "pr_explorer"
description = "Read-only codebase explorer for gathering evidence before changes are proposed."
model = "gpt-5.3-codex-spark"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"
developer_instructions = """
Stay in exploration mode.
Trace the real execution path, cite files and symbols, and avoid proposing fixes unless the parent agent asks for them.
"""
```

### 프로젝트 설정 (`config.toml`)

```toml
[agents]
max_threads = 6
max_depth = 1
```

---

## 10. 마이그레이션 후 검토 항목

import가 완료된 후, 실제로 사용하기 전에 반드시 다음 항목을 점검한다.

| 검토 항목 | 이유 |
|---|---|
| 가져온 스킬/에이전트의 툴 권한 | 권한 모델이 다를 수 있음 |
| MCP 서버의 인증, 헤더, 환경 변수, 전송 방식 | 커스텀 인증이나 특수 설정이 누락될 수 있음 |
| 훅의 동작 차이 | 이벤트 스키마나 실행 타이밍이 다를 수 있음 |
| 플러그인, 마켓플레이스 등 수동 설치 항목 | 자동 변환 불가 |
| 프롬프트 템플릿의 셸 보간, 파일 경로 placeholder | 인자 전달 방식이 다를 수 있음 |

---

## 11. 마이그레이션 후 다음 단계

1. 가져온 프로젝트 중 하나를 열고 기존 작업을 이어서 진행
2. Codex가 처음이라면 Quickstart 가이드를 참고
3. 후속 작업이 필요한 항목은 `migrate-to-codex` 스킬이 포함된 스레드에서 계속 진행

> 마이그레이션 후 Codex는 사용자 수준 설정과 프로젝트 수준 설정을 분리해서 표시하므로, 각 항목이 어디에 속하는지 확인할 수 있다.
