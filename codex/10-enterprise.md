# 엔터프라이즈 관리

> ChatGPT Enterprise 환경에서 Codex를 관리하기 위한 관리자 가이드입니다.

**참조**: <https://developers.openai.com/codex/enterprise/admin-setup>, <https://developers.openai.com/codex/enterprise/managed-configuration>, <https://developers.openai.com/codex/enterprise/governance>

---

## 엔터프라이즈 관리 개요

엔터프라이즈 관리자는 두 가지 방식으로 로컬 Codex 동작을 제어할 수 있습니다.

| 방식 | 설명 | 사용자 재정의 |
| --- | --- | --- |
| **Requirements** (강제 제약) | 관리자가 강제하는 보안 제약 | 불가 |
| **Managed defaults** (관리 기본값) | 시작 시 적용되는 기본값 | 세션 중 변경 가능 (재시작 시 복원) |

---

## requirements.toml 시스템

`requirements.toml`은 사용자가 재정의할 수 없는 **관리자 강제 제약**을 정의합니다.

### 우선순위

Codex는 다음 순서로 requirements를 적용합니다 (이전 레이어가 우선).

| 우선순위 | 소스 | 설명 |
| --- | --- | --- |
| 1 (최고) | 클라우드 관리 Requirements | ChatGPT Business/Enterprise에서 가져옴 |
| 2 | macOS MDM | `com.openai.codex:requirements_toml_base64` |
| 3 | 시스템 `requirements.toml` | Linux/macOS: `/etc/codex/requirements.toml`, Windows: `%ProgramData%\OpenAI\Codex\requirements.toml` |

필드별로 병합됩니다: 이전 레이어가 설정한 필드는 이후 레이어가 덮어쓸 수 없습니다.

### 제어 가능한 설정

| 카테고리 | 설정 키 | 설명 |
| --- | --- | --- |
| 승인 정책 | `allowed_approval_policies` | 허용되는 승인 정책 값 제한 |
| 샌드박스 모드 | `allowed_sandbox_modes` | 허용되는 샌드박스 모드 제한 |
| 웹 검색 | `allowed_web_search_modes` | 웹 검색 모드 제한 (`disabled`, `cached`, `live`) |
| 자동 검토 | `allowed_approvals_reviewers` | 승인 리뷰어 제한 |
| 명령 규칙 | `rules` | 제한적 명령 규칙 강제 |
| MCP 서버 | `mcp_servers` | 허용되는 MCP 서버 허용 목록 |
| 기능 플래그 | `features` | 기능 플래그 고정 |
| 파일 시스템 | `permissions.filesystem.deny_read` | 읽기 거부 경로/글로브 |
| 네트워크 | `experimental_network` | 네트워크 접근 요구사항 |
| 호스트별 샌드박스 | `remote_sandbox_config` | 호스트별 샌드박스 오버라이드 |
| 관리형 훅 | `hooks` | 관리자 강제 라이프사이클 훅 |
| 검토 정책 | `guardian_policy_config` | 자동 검토 정책 교체 |

### 예시

#### 승인 정책 및 샌드박스 제한

`--ask-for-approval never` 및 `--sandbox danger-full-access` (including `--yolo`) 차단:

```toml
allowed_approval_policies = ["untrusted", "on-request"]
allowed_sandbox_modes = ["read-only", "workspace-write"]
```

#### 파일 시스템 읽기 거부

```toml
[permissions.filesystem]
deny_read = [
  "/**/*.env",    # 절대 경로 글로브
  "~/.ssh",       # 홈 디렉토리 상대경로
]
```

#### 명령 규칙 강제

```toml
[rules]
prefix_rules = [
  { pattern = [{ token = "rm" }], decision = "forbidden", justification = "Use git clean -fd instead." },
  { pattern = [{ token = "git" }, { any_of = ["push", "commit"] }], decision = "prompt", justification = "Require review before mutating history." },
]
```

> `requirements.toml`의 규칙은 `decision`이 `prompt` 또는 `forbidden`이어야 합니다 (`allow` 불가).

#### MCP 서버 허용 목록

```toml
[mcp_servers.docs]
identity = { command = "codex-mcp" }

[mcp_servers.remote]
identity = { url = "https://example.com/mcp" }
```

`mcp_servers`가 존재하지만 비어있으면 모든 MCP 서버가 비활성화됩니다.

### 호스트별 샌드박스 오버라이드

`remote_sandbox_config`로 호스트별로 다른 샌드박스 요구사항을 적용합니다.

```toml
allowed_sandbox_modes = ["read-only"]

[[remote_sandbox_config]]
hostname_patterns = ["*.devbox.example.com", "runner-??.ci.example.com"]
allowed_sandbox_modes = ["read-only", "workspace-write"]
```

- 첫 번째로 매칭되는 항목이 우선 적용됩니다.
- 매칭이 대소문자 구분 없이 수행됩니다.
- `*`는 임의의 문자 시퀀스, `?`는 한 문자와 매칭됩니다.

---

## Managed Configuration (managed_config.toml)

관리 기본값은 Codex 시작 시 적용되며, 사용자는 세션 중 변경할 수 있지만 **재시작 시 복원**됩니다.

### 위치

| 플랫폼 | 경로 |
| --- | --- |
| Linux/macOS | `/etc/codex/managed_config.toml` |
| Windows | `~/.codex/managed_config.toml` |

### 구성 우선순위

| 우선순위 | 소스 |
| --- | --- |
| 1 (최고) | macOS 관리 환경설정 (MDM) |
| 2 | `managed_config.toml` |
| 3 | `config.toml` (사용자 기본 구성) |

CLI `--config key=value` 오버라이드는 베이스에 적용되지만, 관리 레이어가 우선합니다.

### 예시 managed_config.toml

```toml
# 보수적 기본값 설정
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

[sandbox_workspace_write]
network_access = false    # 네트워크 비활성화 유지

[otel]
environment = "prod"
exporter = "otlp-http"
log_user_prompt = false   # 프롬프트 내용 익명화
```

---

## macOS MDM 배포

macOS에서는 MDM을 통해 설정을 배포할 수 있습니다.

### 설정 방법

| 항목 | 값 |
| --- | --- |
| Preference Domain | `com.openai.codex` |
| 관리 기본값 키 | `config_toml_base64` |
| Requirements 키 | `requirements_toml_base64` |

### MDM 배포 워크플로우

1. 관리 페이로드 TOML을 작성하고 `base64`로 인코딩 (줄바꿈 없이)
2. MDM 프로필의 `com.openai.codex` 도메인에 문자열 배치
3. 프로필 푸시 후 사용자에게 Codex 재시작 요청
4. 시작 구성 요약에서 관리 값 반영 확인

### 지원 도구

Jamf Pro, Fleet, Kandji 등 표준 macOS MDM 도구 사용 가능.

---

## Managed Hooks

`requirements.toml`에서 관리형 라이프사이클 훅을 직접 정의할 수 있습니다.

### 설정

```toml
# 사용자/프로젝트/세션/플러그인 훅을 건너뛰고 관리 훅만 허용
allow_managed_hooks_only = true

[features]
hooks = true

[hooks]
managed_dir = "/enterprise/hooks"
windows_managed_dir = 'C:\enterprise\hooks'

[[hooks.PreToolUse]]
matcher = "^Bash$"

[[hooks.PreToolUse.hooks]]
type = "command"
command = "python3 /enterprise/hooks/pre_tool_use_policy.py"
command_windows = 'py -3 C:\enterprise\hooks\pre_tool_use_policy.py'
timeout = 30
statusMessage = "Checking managed Bash command"
```

### 주의사항

- Codex는 `requirements.toml`의 훅 설정을 강제하지만, 스크립트 파일은 배포하지 않습니다
- 스크립트는 MDM 또는 디바이스 관리 도구로 별도 배포해야 합니다
- 관리 훅 명령은 구성된 관리 디렉토리 내의 절대 경로를 참조해야 합니다

---

## Governance 및 Observability

### Analytics Dashboard

ChatGPT 워크스페이스 관리자가 셀프 서비스로 채택 현황을 추적할 수 있습니다.

**제공 대시보드**:

| 대시보드 | 내용 |
| --- | --- |
| 제품별 일일 사용자 | CLI, IDE, cloud, Code Review |
| 일일 코드 리뷰 | 리뷰 수, 우선순위별 분류 |
| 일일 클라우드 작업 | 작업 수, 사용자 수 |
| 일일 CLI/VS Code 사용자 | 고유 사용자 수 |

**데이터 내보내기**: CSV 또는 JSON 형식으로 코드 리뷰, 클라우드 작업, 사용자 데이터 내보내기 가능.

### Analytics API

프로그래밍 방식으로 Codex 메트릭을 가져옵니다.

**엔드포인트**:

```
https://api.chatgpt.com/v1/analytics/codex
```

| 엔드포인트 | 설명 |
| --- | --- |
| `/workspaces/{id}/usage` | 일일 스레드, 턴, 크레딧 집계 |
| `/workspaces/{id}/code_reviews` | 코드 리뷰 완료 수, 코멘트 수, 심각도 분류 |
| `/workspaces/{id}/code_review_responses` | 코멘트 반응, 참여도 분석 |

**설정 방법**:

1. OpenAI API Platform Portal에서 전용 API 키 생성
2. `codex.enterprise.analytics.read` 스코프로 키 설정 (support@openai.com 이메일 요청)
3. `workspace_id`는 ChatGPT Admin 콘솔에서 확인

```bash
curl -H "Authorization: Bearer YOUR_PLATFORM_API_KEY" \
  "https://api.chatgpt.com/v1/analytics/codex/workspaces/WORKSPACE_ID/usage"
```

### Compliance API

감사 및 조사를 위한 활동 로그를 내보냅니다.

**엔드포인트**:

```
https://api.chatgpt.com/v1/
```

| 엔드포인트 | 설명 |
| --- | --- |
| `/compliance/workspaces/{id}/logs` | 사용 가능한 로그 파일 목록 |
| `/compliance/workspaces/{id}/logs/{file_id}` | 특정 로그 파일 다운로드 |
| `/compliance/workspaces/{id}/codex_tasks` | Codex 작업 목록 |
| `/compliance/workspaces/{id}/codex_environments` | Codex 환경 목록 |

**로그 보존**: 최대 30일

**설정 방법**:

1. OpenAI API Platform Portal에서 전용 API 키 생성 (모든 권한)
2. support@openai.com에 키 정보와 필요 스코프(`read`, `delete` 또는 둘 다) 전송
3. 승인 후 Compliance API 호출 가능

```bash
# 로그 파일 목록
curl -L -H "Authorization: Bearer YOUR_COMPLIANCE_API_KEY" \
  "https://api.chatgpt.com/v1/compliance/workspaces/WORKSPACE_ID/logs?event_type=CODEX_LOG&after=2026-03-01T00:00:00Z"

# Codex 작업 목록
curl -H "Authorization: Bearer YOUR_COMPLIANCE_API_KEY" \
  "https://api.chatgpt.com/v1/compliance/workspaces/WORKSPACE_ID/codex_tasks"
```

### 추천 거버넌스 설정

| 역할 | 담당 |
| --- | --- |
| 채택 보고 소유자 | Analytics Dashboard 및 API 모니터링 |
| 감사/컴플라이언스 소유자 | Compliance API 로그 검토 |
| 검토 주기 | 정기적인 거버넌스 리뷰 일정 수립 |
