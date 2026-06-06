# Codex CLI - config.toml 설정 전체

> 설정 파일 계층, config.toml 기본 구조, 주요 섹션, 세분화된 승인 정책, 인증, 환경 변수, 권한 프로필, 실행 규칙, 빠른 모드

**참조**: [developers.openai.com/codex/config-basic/](https://developers.openai.com/codex/config-basic/) | [developers.openai.com/codex/config-advanced/](https://developers.openai.com/codex/config-advanced/) | [developers.openai.com/codex/config-reference/](https://developers.openai.com/codex/config-reference/) | [developers.openai.com/codex/environment-variables/](https://developers.openai.com/codex/environment-variables/) | [developers.openai.com/codex/permissions](https://developers.openai.com/codex/permissions) | [developers.openai.com/codex/rules](https://developers.openai.com/codex/rules) | [developers.openai.com/codex/speed](https://developers.openai.com/codex/speed) | [developers.openai.com/codex/hooks](https://developers.openai.com/codex/hooks)

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
# 값: "minimal" | "low" | "medium" | "high" | "xhigh"
model_reasoning_effort = "medium"

# Reasoning 요약 모드
# 값: "auto" | "concise" | "detailed" | "none"
model_reasoning_summary = "auto"

# 출력 상세도
# 값: "low" | "medium" | "high"
model_verbosity = "medium"

# 리뷰에 사용할 모델
review_model = "gpt-5.5"

# 서비스 티어
# 값: "default" | "fast"
service_tier = "default"
```

| 파라미터 | 값 | 설명 |
| --- | --- | --- |
| `model` | 모델 슬러그 문자열 | 사용할 모델 (예: `"gpt-5.5"`, `"gpt-5.4"`) |
| `model_reasoning_effort` | `minimal`, `low`, `medium`, `high`, `xhigh` | Reasoning 모델의 추론 강도 |
| `model_reasoning_summary` | `auto`, `concise`, `detailed`, `none` | Reasoning 요약 생성 모드 |
| `model_verbosity` | `low`, `medium`, `high` | 모델 출력의 상세도 |
| `review_model` | 모델 슬러그 문자열 | 리뷰에 사용할 모델 |
| `service_tier` | `default`, `fast` | 서비스 티어 (Fast mode 활성화 시 `"fast"`) |

### model_providers - 모델 제공자 설정

`model_provider` 단일 문자열 대신 `model_providers` 테이블로 여러 제공자를 정의할 수 있습니다.

```toml
# 기본 제공자 유형
# 값: "openai" | "oss" | 기타
model_provider = "openai"

# 제공자 상세 설정
[model_providers.oss]
provider = "lmstudio"

[model_providers.custom]
provider = "ollama"
base_url = "http://localhost:11434"
```

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `model_providers.<id>.provider` | 문자열 | 제공자 유형 (예: `"lmstudio"`, `"ollama"`) |
| `model_providers.<id>.base_url` | 문자열 | 제공자 API 엔드포인트 URL |
| `model_providers.<id>.env_key` | 문자열 | 제공자 API 키를 공급하는 환경 변수명 |

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
# 값: "untrusted" | "on-request" | "never" | { reject = { sandbox_approval = bool, rules = bool, mcp_elicitations = bool } }
approval_policy = "on-request"
```

| 값 | 설명 |
| --- | --- |
| `untrusted` | 모든 명령에 승인 필요 |
| `on-request` | 샌드박스 외부 작업에만 승인 요청 |
| `never` | 승인 없이 자동 실행 |
| `{ reject = { ... } }` | 특정 프롬프트 카테고리를 자동 거부하며 나머지는 대화형으로 유지 |

> **참고**: `on-failure`는 더 이상 유효한 값이 아닙니다. 대화형 실행에는 `on-request`를, 비대화형 실행에는 `never`를 사용하세요.

#### reject 객체

`approval_policy`를 객체 형태로 설정하면 특정 프롬프트 카테고리를 자동 거부할 수 있습니다.

```toml
[approval_policy]
# 기본 승인 정책
# ...

[approval_policy.reject]
# 샌드박스 승격 승인 프롬프트 자동 거부
sandbox_approval = true

# execpolicy prompt 규칙에 의해 트리거된 승인 자동 거부
rules = true

# MCP elicitation 프롬프트 자동 거부
mcp_elicitations = true
```

| reject 필드 | 타입 | 설명 |
| --- | --- | --- |
| `sandbox_approval` | `boolean` | `true` 시 샌드박스 승격 승인 프롬프트를 자동 거부 |
| `rules` | `boolean` | `true` 시 execpolicy `prompt` 규칙에 의한 승인을 자동 거부 |
| `mcp_elicitations` | `boolean` | `true` 시 MCP elicitation 프롬프트를 자동 거부 |

### approvals_reviewer - 승인 검토자

```toml
# 승인 검토자 설정
# 값: "user" | "auto_review"
approvals_reviewer = "user"
```

| 값 | 설명 |
| --- | --- |
| `user` | 사용자가 직접 승인 |
| `auto_review` | 자동 검토 시스템이 승인/거부 |

### features - 기능 토글

실험적 기능과 추가 기능을 제어합니다. `codex features list`로 확인할 수 있습니다.

```toml
[features]
# 통합 실행기
unified_exec = true

# 셸 스냅샷
shell_snapshot = true

# 셸 도구
shell_tool = true

# 실행 취소
undo = true

# 웹 검색 요청
web_search_request = false

# apply_patch 자유형식
apply_patch_freeform = false

# 샌드박스에서 실행 정책 적용
exec_policy = true

# 원격 모델 목록 새로고침
remote_models = false

# 원격 압축 (ChatGPT 인증만)
remote_compaction = true
```

| 플래그 | 기본값 | 성숙도 | 설명 |
| --- | --- | --- | --- |
| `apply_patch_freeform` | `false` | Experimental | 자유형 `apply_patch` 도구 포함 |
| `elevated_windows_sandbox` | `false` | Experimental | 권한 상승 Windows 샌드박스 파이프라인 사용 |
| `exec_policy` | `true` | Experimental | `shell`/`unified_exec`에 규칙 검사 적용 |
| `experimental_windows_sandbox` | `false` | Experimental | Windows 제한 토큰 샌드박스 사용 |
| `remote_compaction` | `true` | Experimental | 원격 압축 활성화 (ChatGPT 인증만) |
| `remote_models` | `false` | Experimental | 준비 상태 표시 전 원격 모델 목록 새로고침 |
| `shell_snapshot` | `false` | Beta | 셸 환경 스냅샷으로 반복 명령 속도 향상 |
| `shell_tool` | `true` | Stable | 기본 `shell` 도구 활성화 |
| `unified_exec` | `false` | Beta | 통합 PTY 기반 실행 도구 사용 |
| `undo` | `true` | Stable | 턴별 git ghost 스냅샷으로 undo 활성화 |
| `web_search_request` | `false` | Stable | 모델이 웹 검색을 요청할 수 있도록 허용 |

> 생략한 기능 키는 기본값을 유지합니다.
> `config.toml`의 `[features]`에 `feature_name = true`를 추가하거나, CLI에서 `codex --enable feature_name`으로 활성화할 수 있습니다.
> 여러 기능을 동시에 활성화하려면 `codex --enable feature_a --enable feature_b`를 사용하세요.
> 기능을 비활성화하려면 `config.toml`에서 해당 키를 `false`로 설정하세요.

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
API_KEY = "<YOUR_API_KEY>"

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

라이프사이클 이벤트에 사용자 정의 스크립트를 연결합니다. 훅은 기본적으로 활성화되어 있으며, `hooks.json` 파일 또는 `config.toml` 내 인라인 `[hooks]` 테이블로 정의합니다.

```toml
# 인라인 훅 예시
[[hooks.PreToolUse]]
matcher = "^Bash$"

[[hooks.PreToolUse.hooks]]
type = "command"
command = '/usr/bin/python3 "check_bash.py"'
timeout = 30
statusMessage = "Checking Bash command"

[[hooks.PostToolUse]]
matcher = "^Bash$"

[[hooks.PostToolUse.hooks]]
type = "command"
command = '/usr/bin/python3 "review_output.py"'
timeout = 30
statusMessage = "Reviewing Bash output"
```

#### 훅 이벤트 유형 (공식 10종)

| 이벤트 | matcher 대상 | 설명 |
| --- | --- | --- |
| `SessionStart` | `source` (`startup`, `resume`, `clear`, `compact`) | 세션 시작/재개 시 |
| `SubagentStart` | `agent_type` | 서브에이전트 시작 시 |
| `PreToolUse` | `tool_name` (`Bash`, `apply_patch`, MCP 도구명 등) | 도구 사용 전 (차단/수정 가능) |
| `PermissionRequest` | `tool_name` | 권한 요청 시 (승인/거부 가능) |
| `PostToolUse` | `tool_name` | 도구 사용 후 |
| `PreCompact` | `trigger` (`manual`, `auto`) | 대화 압축 전 |
| `PostCompact` | `trigger` (`manual`, `auto`) | 대화 압축 후 |
| `UserPromptSubmit` | 미지원 | 사용자 프롬프트 제출 시 (차단 가능) |
| `SubagentStop` | `agent_type` | 서브에이전트 종료 시 (계속 진행 가능) |
| `Stop` | 미지원 | 턴 종료 시 (계속 진행 가능) |

> **참고**: `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PreCompact`, `PostCompact`, `UserPromptSubmit`, `SubagentStop`, `Stop`은 턴(turn) 스코프에서 실행됩니다. `SessionStart`와 `SubagentStart`는 스레드/서브에이전트 시작 스코프에서 실행됩니다.

#### 훅 핸들러 필드

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `type` | `"command"` | 예 | 핸들러 유형 (현재 `command`만 지원) |
| `command` | 문자열 | 예 | 실행할 명령어 |
| `timeout` | 숫자 | 아니요 | 타임아웃 (초). 기본값 `600` |
| `statusMessage` | 문자열 | 아니요 | UI에 표시할 상태 메시지 |
| `command_windows` | 문자열 | 아니요 | Windows 전용 명령어 오버라이드 |

#### 훅 신뢰 및 관리

- 관리되지 않는 커맨드 훅은 실행 전 검토 및 신뢰가 필요합니다.
- `/hooks` 명령으로 훅 소스 확인, 신뢰, 비활성화가 가능합니다.
- 관리(managed) 훅은 `requirements.toml`, 시스템, MDM, 클라우드 소스에서 로드되며 자동으로 신뢰됩니다.
- `allow_managed_hooks_only = true` 설정 시 사용자/프로젝트/세션/플러그인 훅을 건너뛰고 관리 훅만 로드합니다.

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

`approval_policy`를 `reject` 객체 형태로 설정하면 특정 프롬프트 카테고리를 자동 거부하면서 나머지는 대화형으로 유지할 수 있습니다.

```toml
# 샌드박스 승격 및 규칙 기반 승인은 자동 거부, MCP elicitation은 사용자에게 표시
approval_policy = { reject = { sandbox_approval = true, rules = true } }
```

---

## Permissions (권한 프로필)

**Beta** — 권한 프로필은 활발히 개발 중이며 변경될 수 있습니다.

권한 프로필은 Codex가 로컬에서 실행하는 명령에 최소 권한 원칙을 적용합니다. 파일시스템 규칙(읽기/쓰기)과 네트워크 규칙(도달 가능한 대상)을 결합한 명명된 정책입니다.

> **참조**: [developers.openai.com/codex/permissions](https://developers.openai.com/codex/permissions)

### 기본 제공 프로필

| 프로필 | 설명 |
| --- | --- |
| `:read-only` | 로컬 명령 실행을 읽기 전용으로 유지 |
| `:workspace` | 활성 작업공간 루트 내에서 쓰기 허용 |
| `:danger-full-access` | 로컬 샌드박스 제한 제거 (의도적인 광범위 접근 시에만 사용) |

### 프로필 정의 및 선택

```toml
default_permissions = "project-edit"

[permissions.project-edit.workspace_roots]
"~/code/app" = true
"~/code/shared-lib" = true

[permissions.project-edit.filesystem]
":minimal" = "read"

[permissions.project-edit.filesystem.":workspace_roots"]
"." = "write"
".devcontainer" = "read"
"**/*.env" = "deny"

[permissions.project-edit.network]
enabled = true

[permissions.project-edit.network.domains]
"api.openai.com" = "allow"
"objects.githubusercontent.com" = "allow"
"*.github.com" = "allow"
"tracking.example.com" = "deny"
```

### 프로필 상속 (extends)

`extends` 키워드로 기존 프로필을 확장할 수 있습니다. 기본 제공 프로필(`:read-only`, `:workspace`)을 상속하는 것을 권장합니다. `:danger-full-access`는 상속할 수 없습니다.

```toml
default_permissions = "project-edit"

[permissions.project-edit]
description = "Project editing with OpenAI API access."
extends = ":workspace"

[permissions.project-edit.filesystem.":workspace_roots"]
"**/*.env" = "deny"

[permissions.project-edit.network]
enabled = true

[permissions.project-edit.network.domains]
"api.openai.com" = "allow"
```

### 권한 프로필 설정 사양

| 항목 | 타입 / 값 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `default_permissions` | 문자열 프로필명 | 없음 | 기본으로 적용할 권한 프로필 이름 |
| `[permissions.<name>]` | 테이블 | 없음 | 프로필 정의. `default_permissions`로 선택 |
| `permissions.<name>.description` | 문자열 | 없음 | 프로필에 대한 설명 |
| `permissions.<name>.extends` | 문자열 프로필명 | 없음 | 상속할 부모 프로필. `:read-only`, `:workspace`, 또는 다른 명명된 프로필 |
| `[permissions.<name>.workspace_roots]` | 테이블 | 없음 | 프로필 정의 작업공간 루트 |
| `[permissions.<name>.filesystem]` | 테이블 | 없음 | 경로 → 접근 값 매핑 |
| `[permissions.<name>.network]` | 테이블 | 없음 | 네트워크 샌드박스 프록시 및 정책 |
| `permissions.<name>.network.enabled` | 불리언 | `false` | 네트워크 접근 활성화 |
| `[permissions.<name>.network.domains]` | 테이블 | 없음 | 호스트 패턴 → `allow`/`deny` 매핑 |
| `permissions.<name>.network.proxy_url` | URL | `http://127.0.0.1:3128` | HTTP 프록시 리스너 |
| `permissions.<name>.network.enable_socks5` | 불리언 | `true` | SOCKS5 리스너 활성화 |
| `permissions.<name>.network.allow_local_binding` | 불리언 | `false` | 로컬/사설망 가드 비활성화 |

### 파일시스템 권한

| 접근 | 의미 |
| --- | --- |
| `read` | 파일 읽기 및 디렉토리 나열 허용. 생성/수정/삭제 불가 |
| `write` | 읽기 + 파일 생성/수정/이름변경/삭제 허용 |
| `deny` | 읽기/쓰기 모두 거부. 더 넓은 권한에서 제외 구간 지정 시 사용 |

지원하는 경로 형식:

| 경로 | 의미 | 하위 경로 가능 |
| --- | --- | --- |
| `:root` | 파일시스템 루트 | `.`만 |
| `:minimal` | 일반 개발 도구에 필요한 플랫폼/런타임 경로 | `.`만 |
| `:workspace_roots` | 현재 세션 작업공간 + 프로필 정의 루트 | 예 |
| `:tmpdir` | `$TMPDIR` 위치 | `.`만 |
| `/absolute/path` | 절대 경로 | 예 |
| `~/path` | 홈 디렉토리 하위 경로 | 예 |

> **우선순위**: `deny` > `write` > `read`. 더 구체적인 경로가 더 넓은 경로를 덮어씁니다.

### 네트워크 권한

```toml
[permissions.project-edit.network]
enabled = true

[permissions.project-edit.network.domains]
"example.com" = "allow"      # 정확한 호스트
"*.example.com" = "allow"    # 서브도메인만
"**.example.com" = "allow"   # apex + 서브도메인
"ads.example.com" = "deny"   # deny가 allow보다 우선
```

> 로컬/사설 네트워크 대상은 기본적으로 차단됩니다. `localhost`, `127.0.0.1` 등을 명시적으로 허용해야 합니다.
> `deny` 항목이 `allow` 항목보다 우선합니다.

### 권한 프로필 마이그레이션

권한 프로필은 이전 `sandbox_mode` + `sandbox_workspace_write` 조합을 대체합니다. 한 세션에서 두 시스템을 혼용할 수 없습니다. `sandbox_mode`가 활성 설정 레이어에 나타나면 기존 샌드박스 설정이 대신 사용됩니다.

---

## Rules (실행 규칙)

**Experimental** — 규칙은 실험적이며 변경될 수 있습니다.

Rules는 샌드박스 외부에서 Codex가 실행할 수 있는 명령을 제어합니다. `.rules` 파일은 활성 config 레이어 옆 `rules/` 폴더에 생성합니다 (예: `~/.codex/rules/default.rules`).

> **참조**: [developers.openai.com/codex/rules](https://developers.openai.com/codex/rules)

### prefix_rule()

```
# gh pr view 명령에 대해 실행 전 프롬프트 표시
prefix_rule(
    pattern = ["gh", "pr", "view"],
    decision = "prompt",
    justification = "Viewing PRs is allowed with approval",
    match = [
        "gh pr view 7888",
        "gh pr view --repo openai/codex",
        "gh pr view 7888 --json title,body,comments",
    ],
    not_match = [
        "gh pr --repo openai/codex view 7888",
    ],
)
```

### prefix_rule 필드

| 필드 | 필수 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `pattern` | 예 | - | 명령어 접두사 정의. 각 요소는 리터럴 문자열 또는 리터럴 유니온 |
| `decision` | 아니요 | `"allow"` | 일치 시 동작. `allow`/`prompt`/`forbidden` |
| `justification` | 아니요 | - | 규칙 존재 이유. 승인 프롬프트에 표시될 수 있음 |
| `match` | 아니요 | `[]` | 일치해야 하는 예제 명령어 |
| `not_match` | 아니요 | `[]` | 일치하지 않아야 하는 예제 명령어 |

### decision 우선순위

여러 규칙이 일치하면 가장 제한적인 decision이 적용됩니다:

`forbidden` > `prompt` > `allow`

### 규칙 파일 위치

- `~/.codex/rules/` — 사용자 글로벌
- `<repo>/.codex/rules/` — 프로젝트 로컬 (신뢰된 프로젝트만)
- 팀 config 위치의 `rules/`

> `.rules` 파일은 **Starlark** 구문을 사용합니다. Python과 유사하지만 부작용 없이 안전하게 실행되도록 설계되었습니다.

### 명령어 분할

Codex는 `bash -lc`, `bash -c`, `zsh -c`, `sh -c`로 래핑된 스크립트를 특수 처리합니다:

- **안전한 분할**: 일반 단어 + 안전한 연산자(`&&`, `||`, `;`, `|`)로만 구성된 스크립트는 개별 명령어로 분할하여 규칙 적용
- **분할하지 않음**: 리다이렉션(`>`, `>>`), 치환(`$(...)`), 환경변수, 와일드카드, 제어문이 포함된 경우 전체를 단일 호출로 처리

### 정책 테스트

```shell
codex execpolicy check --pretty \
  --rules ~/.codex/rules/default.rules \
  -- gh pr view 7888 --json title,body,comments
```

---

## Speed (빠른 모드)

> **참조**: [developers.openai.com/codex/speed](https://developers.openai.com/codex/speed)

### Fast mode

지원 모델의 속도를 1.5배 향상시키는 기능입니다. 크레딧 소비율이 표준 모드보다 높습니다.

| 모델 | 속도 향상 | 크레딧 소비율 |
| --- | --- | --- |
| GPT-5.5 | 1.5x | 표준의 2.5배 |
| GPT-5.4 | 1.5x | 표준의 2배 |

CLI에서 설정:

```shell
/fast on      # 활성화
/fast off     # 비활성화
/fast status  # 현재 상태 확인
```

config.toml에서 영구 설정:

```toml
service_tier = "fast"

[features]
fast_mode = true
```

> API 키 인증 시에는 표준 API 요금이 적용되며 Fast mode 크레딧을 사용할 수 없습니다.
> Fast mode는 Codex IDE 확장, CLI, 앱에서 ChatGPT 로그인 시 사용 가능합니다.

### Codex-Spark

GPT-5.3-Codex-Spark는 빠르고 가벼운 전용 코덱스 모델입니다. Fast mode와 달리 별도의 모델 선택이며 자체 사용량 제한이 있습니다.

- Research preview 기간 동안 ChatGPT Pro 구독자에게만 제공됩니다.

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

Codex는 지속 설정에 `config.toml`을 사용합니다. 환경 변수는 셸 범위 오버라이드, 자동화 시크릿, 설치 프로그램 동작, 진단에 사용합니다.

> 이 표는 Codex가 직접 읽는 **안정적 공개 환경 변수**만 나열합니다. 내부 개발 변수, 테스트 변수, 또는 `env_key`로 직접 선택하는 제공자별 시크릿 이름은 포함하지 않습니다.

### Core locations

| 환경 변수 | 사용 주체 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `CODEX_HOME` | CLI, IDE 확장, app-server, 설치 프로그램 | `~/.codex` | 구성, 인증, 로그, 세션, 스킬, 독립 패키지 메타데이터를 포함한 Codex 상태의 루트. 설정 시 디렉토리가 이미 존재해야 함 |
| `CODEX_SQLITE_HOME` | CLI 및 app-server 상태 | `CODEX_HOME` | SQLite 기반 상태가 저장되는 위치. `sqlite_home` config 옵션이 우선. 상대 경로는 현재 작업 디렉토리에서 해석 |

### Installer variables

| 환경 변수 | 기본값 | 설명 |
| --- | --- | --- |
| `CODEX_NON_INTERACTIVE` | `false` | `1`, `true`, `yes`로 설정 시 설치 프로그램 프롬프트를 건너뜀. 프롬프트는 기본 응답을 사용하므로 스크립트 설치 및 업데이트에 사용. 최초 설정에는 부적합 |
| `CODEX_INSTALL_DIR` | macOS/Linux: `~/.local/bin`, Windows: `%LOCALAPPDATA%\Programs\OpenAI\Codex\bin` | `codex` 명령이 설치되는 위치 변경. 독립 패키지 캐시는 `CODEX_HOME/packages/standalone`에 유지 |

```shell
# 무인 설치 예시 (macOS/Linux)
curl -fsSL https://chatgpt.com/codex/install.sh | CODEX_NON_INTERACTIVE=1 sh

# 무인 설치 예시 (Windows PowerShell)
$env:CODEX_NON_INTERACTIVE=1; irm https://chatgpt.com/codex/install.ps1 | iex
```

### Authentication and network

| 환경 변수 | 사용 주체 | 설명 |
| --- | --- | --- |
| `CODEX_API_KEY` | `codex exec` | 단일 비대화형 실행을 위한 API 키. **`codex exec`에서만 지원**. 저장소 제어 코드를 실행할 때 작업 전체가 아닌 인라인으로 설정 |
| `CODEX_ACCESS_TOKEN` | CLI, app-server, 신뢰된 자동화 | ChatGPT 또는 Codex 액세스 토큰 제공. 지속 로그인을 위해 `codex login --with-access-token`으로 파이프 |
| `CODEX_CA_CERTIFICATE` | HTTPS, 로그인, WebSocket 클라이언트 | 기업 TLS 가로채기 또는 사설 루트 CA 환경에서 PEM CA 번들 경로. `SSL_CERT_FILE`보다 우선 |
| `SSL_CERT_FILE` | HTTPS, 로그인, WebSocket 클라이언트 | `CODEX_CA_CERTIFICATE`가 설정되지 않은 경우 대체 PEM CA 번들 경로 |

> 제공자 API 키의 경우 모델 제공자 구성에서 `env_key`를 설정하세요. Codex는 해당 구성으로 명명된 변수를 읽으므로, 변수 이름 자체는 고정된 Codex 환경 변수가 아닙니다.

### Diagnostics

| 환경 변수 | 사용 주체 | 설명 |
| --- | --- | --- |
| `RUST_LOG` | CLI 및 app-server | Rust 로그 필터링 및 상세도 제어. `codex exec`는 더 상세한 값을 설정하지 않는 한 `error` 출력이 기본 |

```shell
# 디버그 로깅 예시
RUST_LOG=debug codex -c log_dir=./.codex-log
tail -F ./.codex-log/codex-tui.log
```

### 환경 변수 사용 예시

```shell
# Codex exec 전용 API 키 (인라인으로 설정, 작업 전체에 설정하지 말 것)
CODEX_API_KEY="<YOUR_API_KEY>" codex exec "작업 설명"

# 액세스 토큰으로 로그인
echo "<YOUR_ACCESS_TOKEN>" | codex login --with-access-token

# SQLite 데이터베이스 경로
export CODEX_SQLITE_HOME="/data/codex/db"

# 커스텀 CA 인증서
export CODEX_CA_CERTIFICATE="/path/to/ca-cert.pem"

# Codex 홈 디렉토리 변경
export CODEX_HOME="/data/codex"

# 디버그 로깅 활성화
export RUST_LOG=codex_core=debug,codex_tui=debug
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
model_reasoning_effort = "medium"
model_reasoning_summary = "auto"
model_verbosity = "medium"
service_tier = "default"
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
[[hooks.PostToolUse]]
matcher = "^Bash$"

[[hooks.PostToolUse.hooks]]
type = "command"
command = 'echo "도구 사용 완료"'
timeout = 30

# 기능 플래그
[features]
unified_exec = true
shell_snapshot = true
shell_tool = true
undo = true

# 프로필
[profiles.fast]
model = "gpt-4.1-mini"
approval_policy = "never"
service_tier = "fast"

[profiles.fast.features]
unified_exec = true

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

> **최종 업데이트**: 2026-06-06
> **출처**: [developers.openai.com/codex/config-basic/](https://developers.openai.com/codex/config-basic/), [developers.openai.com/codex/config-advanced/](https://developers.openai.com/codex/config-advanced/), [developers.openai.com/codex/config-reference/](https://developers.openai.com/codex/config-reference/), [developers.openai.com/codex/environment-variables/](https://developers.openai.com/codex/environment-variables/), [developers.openai.com/codex/permissions](https://developers.openai.com/codex/permissions), [developers.openai.com/codex/rules](https://developers.openai.com/codex/rules), [developers.openai.com/codex/speed](https://developers.openai.com/codex/speed), [developers.openai.com/codex/hooks](https://developers.openai.com/codex/hooks)
