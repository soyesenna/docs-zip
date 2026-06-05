# Codex CLI - config.toml 설정 전체

> 설정 파일 계층, config.toml 기본 구조, 주요 섹션, 세분화된 승인 정책, 인증, 환경 변수

**참조**: [developers.openai.com/codex/config-file/config-basics](https://developers.openai.com/codex/config-file/config-basics) | [developers.openai.com/codex/config-file/config-reference](https://developers.openai.com/codex/config-file/config-reference) | [developers.openai.com/codex/config-file/environment-variables](https://developers.openai.com/codex/config-file/environment-variables)

---

## 설정 파일 계층

Codex는 여러 계층의 설정 파일을 지원하며, 높은 우선순위의 설정이 낮은 것을 덮어씁니다.

```
┌──────────────────────────────────────────────┐
│  managed_config.toml (엔터프라이즈 관리)      │ ← 최고 우선순위
├──────────────────────────────────────────────┤
│  CLI 플래그 (-m, -a, -s 등)                  │
├──────────────────────────────────────────────┤
│  .codex/config.toml (프로젝트별)              │
├──────────────────────────────────────────────┤
│  ~/.codex/config.toml (사용자 글로벌)         │ ← 최저 우선순위
└──────────────────────────────────────────────┘
```

### 설정 파일 위치

| 파일 | 경로 | 설명 |
| --- | --- | --- |
| 글로벌 config.toml | `~/.codex/config.toml` | 사용자 전체 설정 |
| 프로젝트 config.toml | `.codex/config.toml` | 프로젝트별 설정 |
| 관리 config.toml | `~/.codex/managed_config.toml` | 엔터프라이즈 관리 설정 |

> 하위 계층에서 설정하지 않은 필드는 상위 계층의 값으로 채워집니다.
> `managed_config.toml`은 `approval_policy`와 `sandbox_mode`를 요구사항으로 해석하여 해당 값만 허용합니다.

### config.schema.json

설정 파일의 JSON 스키마가 제공됩니다. 자세한 필드 정의는 `config.schema.json`을 참조하세요.

---

## config.toml 기본 구조

```toml
# ~/.codex/config.toml 예시

# 모델 설정
model = "gpt-5.5"

# 승인 정책
approval_policy = "on-request"

# 샌드박스 모드
sandbox_mode = "workspace-write"

# 웹 검색 설정
web_search = "live"

# 리뷰 모델
review_model = "gpt-5.5"
```

---

## 주요 섹션

### model - 모델 설정

모델 선택 및 동작을 제어합니다.

```toml
# 모델 슬러그
model = "gpt-5.5"

# Reasoning effort (reasoning 모델에만 적용)
# 값: "low" | "medium" | "high"
reasoning_effort = "medium"

# Reasoning 요약 생성 여부
reasoning_summary = true

# 출력 상세도
# 값: "low" | "medium" | "high"
verbosity = "medium"

# 리뷰에 사용할 모델
review_model = "gpt-5.5"

# Fast 서비스 티어
fast = false
```

### model_provider - 모델 제공자 설정

```toml
# 제공자 유형
# 값: "openai" | "oss" | 기타
model_provider = "openai"

# OSS 로컬 제공자 (lmstudio, ollama)
[model_provider.oss]
provider = "lmstudio"
```

### sandbox_mode - 샌드박스 모드

```toml
# 샌드박스 정책
# 값: "read-only" | "workspace-write" | "danger-full-access"
sandbox_mode = "workspace-write"
```

| 값 | 설명 |
| --- | --- |
| `read-only` | 파일 읽기만 허용 |
| `workspace-write` | 작업 디렉토리 내에서 수정 및 실행 허용 |
| `danger-full-access` | 전체 시스템 접근 허용 |

### approval_policy - 승인 정책

```toml
# 승인 정책
# 값: "untrusted" | "on-request" | "never" | { granular 설정 }
approval_policy = "on-request"
```

| 값 | 설명 |
| --- | --- |
| `untrusted` | 모든 명령에 승인 필요 |
| `on-request` | 샌드박스 외부 작업에만 승인 요청 |
| `never` | 승인 없이 자동 실행 |
| `granular` | 세분화된 승인 규칙 적용 (객체 형태) |

### approvals_reviewer - 승인 검토자

```toml
# 승인 검토자 설정
# 값: "user" | "auto_review" | "guardian_subagent"
approvals_reviewer = "user"
```

| 값 | 설명 |
| --- | --- |
| `user` | 사용자가 직접 승인 |
| `auto_review` | 자동 검토 시스템이 승인/거부 |
| `guardian_subagent` | Guardian 서브에이전트가 검토 |

### features - 기능 토글

실험적 기능과 추가 기능을 제어합니다. `codex features list`로 확인할 수 있습니다.

```toml
[features]
# 통합 실행기
unified_exec = true

# 셸 스냅샷
shell_snapshot = true

# 하위 에이전트
subagents = false

# 스마트 승인
smart_approvals = false

# 앱
apps = false
```

> 사용 가능한 기능 플래그는 버전에 따라 다를 수 있습니다.
> `codex features list` 명령으로 현재 사용 가능한 기능을 확인하세요.

### tools - 도구 설정

```toml
[tools]
# 웹 검색
# 값: "disabled" | "cached" | "live"
web_search = "cached"

# 이미지 생성
image_generation = true

# 셸 실행
shell = true

# 컴퓨터 사용
computer_use = false
```

### tui - TUI 설정

터미널 UI의 모양과 동작을 제어합니다.

```toml
[tui]
# 구문 강조 테마
theme = "dracula"

# Vim 모드 기본값
vim_mode_default = false

# Raw 출력 모드 기본값
raw_output_mode = false

# 상태 표시줄 항목
# 값: model, model+reasoning, context_stats, rate_limits, git_branch,
#     token_counters, session_id, current_dir, codex_version
status_line = ["model", "git_branch", "token_counters"]

# 터미널 제목 항목
# 값: app_name, project, spinner, status, thread, git_branch, model, task_progress
terminal_title = ["project", "model"]

# 키맵 커스텀
[tui.keymap]
# 글로벌 단축키
[tui.keymap.global]
# 컨텍스트별 단축키
```

#### 키 바인딩 형식

```
ctrl-a, shift-enter, page-down, alt-r 등
```

### mcp_servers - MCP 서버 설정

Model Context Protocol 서버를 연결합니다.

```toml
# STDIO MCP 서버 예시
[[mcp_servers]]
name = "my-mcp-server"
command = "npx"
args = ["-y", "@my-org/mcp-server"]

# 환경 변수
[mcp_servers.env]
API_KEY = "sk-..."

# HTTP 스트리밍 MCP 서버 예시
[[mcp_servers]]
name = "remote-mcp"
url = "https://mcp.example.com/stream"

# 인증
[mcp_servers.auth]
type = "bearer"
token_env = "MCP_AUTH_TOKEN"
```

### plugins - 플러그인 관리

```toml
[plugins]
# 플러그인 캐시 경로
# 기본값: ~/.codex/plugins/cache/

# 플러그인 활성화/비활성화
[plugins.state]
my-plugin = true
another-plugin = false
```

### hooks - 훅 설정

라이프사이클 이벤트에 사용자 정의 스크립트를 연결합니다.

```toml
# 인라인 훅 예시
[[hooks]]
event = "pre_tool_use"
command = "echo '도구 사용 전'"

[[hooks]]
event = "post_tool_use"
command = "echo '도구 사용 후'"

# 훅 이벤트 유형
# - pre_tool_use: 도구 사용 전
# - post_tool_use: 도구 사용 후
# - pre_response: 응답 생성 전
# - post_response: 응답 생성 후
# - on_error: 에러 발생 시
```

### skills - 스킬 설정

```toml
[skills]
# 스킬 디렉토리 (글로벌)
dirs = ["~/.codex/skills"]

# 활성화된 스킬
enabled = ["my-custom-skill"]
```

### apps - 앱 설정

```toml
[apps]
# 앱 연동 설정
[apps.github]
enabled = true

[apps.slack]
enabled = false

[apps.linear]
enabled = false
```

### analytics - 분석 설정

```toml
[analytics]
# 사용 분석 수집
enabled = true
```

---

## 세분화된 승인 정책 (GranularApprovalConfig)

`approval_policy`를 객체 형태로 설정하면 세분화된 제어가 가능합니다.

```toml
[approval_policy]
type = "granular"

# MCP 도구 호출 승인
[approval_policy.mcp_elicitations]
# 특정 MCP 서버의 도구에 대한 승인 규칙

# 규칙 기반 승인
[[approval_policy.rules]]
# 패턴 매칭 규칙
pattern = "npm test"
allowed = true

[[approval_policy.rules]]
pattern = "rm -rf"
allowed = false

# 샌드박스 승인
[approval_policy.sandbox_approval]
# 샌드박스 내 작업에 대한 승인 정책

# 스킬 승인
[approval_policy.skill_approval]
# 스킬 사용에 대한 승인 정책

# 권한 요청
[approval_policy.request_permissions]
# 특정 권한 요청에 대한 승인 정책
```

---

## 인증 저장 모드

Codex는 여러 인증 저장 방식을 지원합니다.

```toml
# 인증 저장 모드
# 값: "file" | "keyring" | "auto" | "ephemeral"
auth_storage = "auto"
```

| 모드 | 설명 |
| --- | --- |
| `file` | 파일에 인증 정보 저장 |
| `keyring` | OS 키링 (macOS Keychain, Linux Secret Service 등) 사용 |
| `auto` | 자동 선택 (기본값) |
| `ephemeral` | 메모리에만 저장 (세션 종료 시 삭제) |

---

## 프로필 (Profiles)

`config.toml`에서 이름 있는 프로필을 정의하여 CLI에서 `--profile` / `-p`로 선택할 수 있습니다.

```toml
# 기본 설정
model = "gpt-5.5"
approval_policy = "on-request"

# "fast" 프로필
[profiles.fast]
model = "gpt-4.1-mini"
approval_policy = "never"

# "safe" 프로필
[profiles.safe]
model = "gpt-5.5"
approval_policy = "untrusted"
sandbox_mode = "read-only"
```

```shell
# 프로필 사용
codex -p fast
codex --profile safe "리팩토링해줘"
```

---

## 환경 변수 목록

| 환경 변수 | 설명 | 기본값 |
| --- | --- | --- |
| `OPENAI_API_KEY` | OpenAI API 키 | - |
| `CODEX_HOME` | Codex 홈 디렉토리 | `~/.codex` |
| `CODEX_NON_INTERACTIVE` | 무인 설치 모드 | `0` |
| `CODEX_REMOTE_TOKEN` | 원격 연결 인증 토큰 | - |
| `RUST_LOG` | Rust 로깅 수준 | `codex_core=info,codex_tui=info,codex_rmcp_client=info` |
| `VISUAL` | 외부 에디터 경로 (`Ctrl+G`에서 사용) | - |
| `EDITOR` | 외부 에디터 경로 (`VISUAL` 미설정 시 사용) | - |
| `PATH` | 시스템 PATH (Codex가 명령어 탐색에 사용) | - |

### 환경 변수 사용 예시

```shell
# API 키 설정
export OPENAI_API_KEY="sk-..."

# Codex 홈 디렉토리 변경
export CODEX_HOME="/data/codex"

# 디버그 로깅 활성화
export RUST_LOG=codex_core=debug,codex_tui=debug

# 원격 연결
export CODEX_REMOTE_TOKEN="$(cat ~/.codex/app-server-token)"
codex --remote wss://remote-host:4500 --remote-auth-token-env CODEX_REMOTE_TOKEN
```

---

## CLI에서 설정 오버라이드

`-c` 플래그로 config.toml 필드를 직접 오버라이드할 수 있습니다.

```shell
# 로그 디렉토리 변경
codex -c log_dir=./.codex-log

# 모델 변경
codex -c model=gpt-4.1-mini
```

---

## 설정 진단

세션 내에서 설정 상태를 확인할 수 있습니다.

```
/status          # 세션 설정 및 토큰 사용량
/debug-config    # 설정 레이어 계층 및 정책 요구사항
```

### `/debug-config` 출력 항목

- 설정 레이어 순서 (낮은 우선순위부터)
- 각 레이어의 활성/비활성 상태
- 정책 소스:
  - `allowed_approval_policies`
  - `allowed_sandbox_modes`
  - `mcp_servers`
  - `rules`
  - `enforce_residency`
  - `experimental_network`

---

## 샘플 config.toml

```toml
# ~/.codex/config.toml - 전체 예시

# 모델 설정
model = "gpt-5.5"
reasoning_effort = "medium"
verbosity = "medium"
review_model = "gpt-5.5"

# 승인 및 샌드박스
approval_policy = "on-request"
sandbox_mode = "workspace-write"
approvals_reviewer = "user"

# 웹 검색
web_search = "cached"

# TUI 설정
[tui]
theme = "dracula"
vim_mode_default = false
raw_output_mode = false
status_line = ["model", "git_branch", "token_counters"]
terminal_title = ["project", "model"]

# MCP 서버
[[mcp_servers]]
name = "filesystem"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]

# 훅
[[hooks]]
event = "post_tool_use"
command = "echo '도구 사용 완료'"

# 기능 플래그
[features]
unified_exec = true
shell_snapshot = true

# 프로필
[profiles.fast]
model = "gpt-4.1-mini"
approval_policy = "never"

[profiles.safe]
approval_policy = "untrusted"
sandbox_mode = "read-only"

# 분석
[analytics]
enabled = true
```

---

## 엔터프라이즈 관리 설정 (managed_config.toml)

엔터프라이즈 관리자는 `managed_config.toml`을 통해 조직 전체의 정책을 강제할 수 있습니다.

```toml
# ~/.codex/managed_config.toml

# 허용되는 승인 정책 (이 값만 허용)
approval_policy = "on-request"

# 허용되는 샌드박스 모드
sandbox_mode = "workspace-write"

# 허용되는 승인 정책 목록
allowed_approval_policies = ["on-request", "untrusted"]

# 허용되는 샌드박스 모드 목록
allowed_sandbox_modes = ["read-only", "workspace-write"]

# MCP 서버 (관리자가 승인한 서버만)
[[mcp_servers]]
name = "approved-server"
command = "npx"
args = ["-y", "@company/mcp-server"]

# 규칙
rules = ["no-network", "no-file-deletion"]

# 거주지 강제 (데이터 지역성)
enforce_residency = "us"

# 실험적 네트워크 설정
experimental_network = false
```

> `managed_config.toml`의 `approval_policy`와 `sandbox_mode`는 **요구사항**으로 해석됩니다.
> 사용자가 다른 값을 설정해도 관리 설정이 우선합니다.

---

> **최종 업데이트**: 2026-06-05
> **출처**: [developers.openai.com/codex/config-file/config-basics](https://developers.openai.com/codex/config-file/config-basics), [developers.openai.com/codex/config-file/config-reference](https://developers.openai.com/codex/config-file/config-reference), [developers.openai.com/codex/config-file/environment-variables](https://developers.openai.com/codex/config-file/environment-variables)
