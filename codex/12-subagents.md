# Codex CLI - 서브에이전트 및 멀티 에이전트

> 서브에이전트 워크플로우로 Codex를 병렬 작업에 활용하는 방법, 커스텀 에이전트 정의, 팀 구성 패턴, 모델 선택 가이드

**참조**: [developers.openai.com/codex/subagents](https://developers.openai.com/codex/subagents) | [developers.openai.com/codex/concepts/subagents](https://developers.openai.com/codex/concepts/subagents) | [developers.openai.com/codex/multi-agent](https://developers.openai.com/codex/multi-agent) | [developers.openai.com/codex/concepts/multi-agents](https://developers.openai.com/codex/concepts/multi-agents)

---

## 서브에이전트 개요

Codex는 전문화된 에이전트를 병렬로 실행(spawn)하고 그 결과를 하나의 응답으로 수집하는 서브에이전트 워크플로우를 지원합니다. 이는 코드베이스 탐색, 다단계 기능 구현 등 병렬성이 높은 복잡한 작업에 특히 유용합니다.

서브에이전트 워크플로우를 사용하면 작업에 따라 서로 다른 모델 구성과 명령어를 가진 커스텀 에이전트도 정의할 수 있습니다.

### 핵심 용어

| 용어 | 정의 |
| --- | --- |
| **Subagent workflow** | Codex가 병렬 에이전트를 실행하고 결과를 결합하는 워크플로우 |
| **Subagent** | 특정 작업을 처리하기 위해 Codex가 시작하는 위임된 에이전트 |
| **Agent thread** | 에이전트의 CLI 스레드. `/agent`로 전환 및 검사 가능 |

### 왜 서브에이전트가 필요한가

컨텍스트 윈도우가 크더라도 모델에는 한계가 있습니다. 메인 대화(요구사항, 제약사항, 결정을 정의하는 곳)에 탐색 노트, 테스트 로그, 스택 트레이스, 명령어 출력 등 노이즈가 많은 중간 결과물이 쌓이면 세션의 신뢰성이 점진적으로 저하됩니다.

| 문제 | 설명 |
| --- | --- |
| **Context pollution** | 유용한 정보가 노이즈가 많은 중간 출력물 아래 묻힘 |
| **Context rot** | 덜 관련된 세부사항으로 대화가 채워지면서 성능 저하 |

서브에이전트 워크플로우는 노이즈가 많은 작업을 메인 스레드에서 분리합니다.

- **메인 에이전트**: 요구사항, 결정, 최종 출력에 집중
- **서브에이전트**: 탐색, 테스트, 로그 분석 등을 병렬로 실행
- **요약 반환**: 원시 중간 출력 대신 요약을 메인 스레드에 반환

---

## 활성화 및 기본 동작

### 서브에이전트 (현재 릴리스)

현재 Codex 릴리스에서는 서브에이전트 워크플로우가 **기본적으로 활성화**되어 있습니다.

서브에이전트 활동은 현재 **Codex App**과 **CLI**에서 표시됩니다. IDE 확장에서의 가시성은 곧 지원될 예정입니다.

### 멀티 에이전트 (실험적)

멀티 에이전트 워크플로우는 현재 **실험적(experimental)**이며 명시적으로 활성화해야 합니다.

CLI에서 `/experimental`을 입력한 뒤 **Multi-agents**를 활성화하고 Codex를 재시작합니다.

또는 설정 파일(`~/.codex/config.toml`)에 직접 추가합니다.

```toml
[features]
multi_agent = true
```

### 자동 스폰 여부 비교

| 버전 | 자동 스폰 | 비고 |
| --- | --- | --- |
| **서브에이전트 (현재)** | 사용자가 명시적으로 요청할 때만 | "spawn two agents", "delegate in parallel" 등 직접 지시 필요 |
| **멀티 에이전트 (실험적)** | Codex가 자동으로 판단하거나 사용자가 명시적 요청 | 장시간 실행 명령/폴링 워크플로우에 `monitor` 역할 활용 |

---

## 서브에이전트 설정 — `[agents]` 섹션

### 글로벌 설정

글로벌 서브에이전트 설정은 `config.toml`의 `[agents]` 섹션에 정의합니다.

| 필드 | 타입 | 필수 | 기본값 | 설명 |
| --- | --- | --- | --- | --- |
| `agents.max_threads` | number | No | `6` | 동시에 열 수 있는 에이전트 스레드 상한 |
| `agents.max_depth` | number | No | `1` | 에이전트 중첩 깊이. 루트 세션은 0부터 시작 |
| `agents.job_max_runtime_seconds` | number | No | `1800` | `spawn_agents_on_csv` 작업의 워커별 기본 타임아웃(초) |

**주의사항**:

- `max_depth`의 기본값 `1`은 직계 자식 에이전트가 하위 에이전트를 실행(spawn)하는 것은 허용하지만 더 깊은 중첩은 방지합니다. 기본값을 유지하는 것을 권장합니다. 이 값을 높이면 광범위한 위임 지시가 반복적인 팬아웃(fan-out)으로 이어져 토큰 사용량, 지연 시간, 로컬 리소스 소비가 증가합니다.
- `max_threads`는 동시 스레드 수를 제한하지만 깊은 재귀의 비용과 예측 가능성 위험은 제거하지 않습니다.
- `job_max_runtime_seconds`를 설정하지 않으면 `spawn_agents_on_csv`는 워커당 1800초(30분)의 기본 타임아웃을 사용합니다. 개별 호출 시 `max_runtime_seconds`를 지정하면 글로벌 설정보다 우선합니다.
- 커스텀 에이전트 이름이 빌트인 에이전트(예: `explorer`)와 일치하면 커스텀 에이전트가 우선합니다.

### 기본 설정 예시

```toml
# .codex/config.toml
[agents]
max_threads = 6
max_depth = 1
```

---

## 서브에이전트 생성 및 구성

### 빌트인 에이전트

Codex는 다음 빌트인 에이전트를 제공합니다.

| 에이전트 | 설명 |
| --- | --- |
| `default` | 범용 폴백 에이전트 |
| `worker` | 구현 및 수정에 집중하는 실행 중심 에이전트 |
| `explorer` | 읽기 중심 코드베이스 탐색 에이전트 |
| `monitor` | 장시간 실행 명령/작업 모니터링 역할 (대기/폴링에 최적화). 멀티 에이전트(실험적)에서만 제공 |

### 커스텀 에이전트 파일

커스텀 에이전트를 정의하려면 독립 실행형 TOML 파일을 작성합니다.

| 범위 | 경로 |
| --- | --- |
| 개인용 | `~/.codex/agents/` |
| 프로젝트 범위 | `.codex/agents/` |

각 파일은 하나의 커스텀 에이전트를 정의합니다. Codex는 이 파일들을 스폰된 세션의 구성 레이어로 로드하므로, 일반 Codex 세션 config와 동일한 설정을 오버라이드할 수 있습니다.

### 커스텀 에이전트 파일 스키마

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `name` | string | Yes | Codex가 에이전트를 실행(spawn)하거나 참조할 때 사용하는 이름 |
| `description` | string | Yes | Codex가 이 에이전트를 언제 사용해야 하는지에 대한 가이드 |
| `developer_instructions` | string | Yes | 에이전트의 동작을 정의하는 핵심 명령어 |
| `nickname_candidates` | string[] | No | 스폰된 에이전트에 할당할 표시용 닉네임 풀 |

생략한 선택 필드(`model`, `model_reasoning_effort`, `sandbox_mode`, `mcp_servers`, `skills.config` 등)는 부모 세션에서 상속합니다.

Codex는 `name` 필드로 커스텀 에이전트를 식별합니다. 파일명을 에이전트 이름과 일치시키는 것이 가장 간단한 규칙이지만, `name` 필드가 실제 식별자입니다.

### 표시 닉네임

`nickname_candidates`를 사용하면 동일한 커스텀 에이전트의 여러 인스턴스를 실행할 때 UI에 고유한 레이블을 표시할 수 있습니다.

- 닉네임은 표시 전용입니다. Codex는 여전히 `name`으로 에이전트를 식별하고 실행(spawn)합니다.
- 닉네임 후보는 비어있지 않은 고유한 이름 목록이어야 합니다.
- 각 닉네임은 ASCII 문자, 숫자, 공백, 하이픈, 밑줄을 사용할 수 있습니다.

```toml
name = "reviewer"
description = "PR reviewer focused on correctness, security, and missing tests."
developer_instructions = """
Review code like an owner.
Prioritize correctness, security, behavior regressions, and missing test coverage.
"""
nickname_candidates = ["Atlas", "Delta", "Echo"]
```

---

## 멀티 에이전트 아키텍처 — 역할 기반 구성 (실험적)

멀티 에이전트 모드에서는 `[agents]` 섹션에서 역할(role) 기반으로 에이전트를 구성합니다. 로컬 설정(`~/.codex/config.toml`) 또는 프로젝트별 설정(`.codex/config.toml`)에 정의합니다.

### 역할 스키마

각 역할은 Codex가 이 에이전트를 언제 사용할지에 대한 가이드(`description`)를 제공하고, 선택적으로 역할별 config 파일(`config_file`)을 로드합니다.

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `[agents.<name>]` | table | No | 역할 선언. `<name>`이 에이전트 실행(spawn) 시 `agent_type`이 됨 |
| `agents.<name>.description` | string | No | Codex가 역할을 선택할 때 참고하는 가이드 |
| `agents.<name>.config_file` | string (path) | No | 해당 역할로 실행(spawn)된 에이전트에 적용할 TOML config 레이어 경로 |

**주의사항**:

- Codex는 `[agents.<name>]`의 알 수 없는 필드를 거부합니다.
- 상대적 `config_file` 경로는 역할을 정의한 `config.toml` 기준으로 해석됩니다.
- Codex는 config 로드 시 `agents.<name>.config_file`을 검증하며, 기존 파일을 가리켜야 합니다.
- 역할 이름이 빌트인 역할(예: `explorer`)과 일치하면 사용자 정의 역할이 우선합니다.
- config 파일을 로드할 수 없으면 파일을 수정할 때까지 에이전트 실행(spawn)이 실패할 수 있습니다.
- 역할에서 설정하지 않은 구성은 부모 세션에서 상속됩니다.

### 역할 정의 예시

```toml
# .codex/config.toml
[agents]
max_threads = 6
max_depth = 1

[agents.explorer]
description = "Read-only codebase explorer for gathering evidence before changes are proposed."
config_file = "agents/explorer.toml"

[agents.reviewer]
description = "PR reviewer focused on correctness, security, and missing tests."
config_file = "agents/reviewer.toml"

[agents.docs_researcher]
description = "Documentation specialist that uses the docs MCP server to verify APIs and framework behavior."
config_file = "agents/docs-researcher.toml"
```

역할별 config 파일 예시 (`agents/reviewer.toml`):

```toml
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "read-only"
developer_instructions = """
Review code like an owner.
Prioritize correctness, security, behavior regressions, and missing test coverage.
Lead with concrete findings, include reproduction steps when possible, and avoid style-only comments unless they hide a real bug.
"""
```

---

## 팀 구성 패턴 — Building AI Teams

최고의 에이전트 정의는 좁고 명확한(opinionated) 것입니다. 각 에이전트에 하나의 명확한 작업, 그 작업에 맞는 도구 표면, 인접 작업으로 번지지 않도록 하는 명령어를 부여합니다.

### 패턴 1: PR 리뷰 팀

리뷰를 세 가지 집중된 역할로 분할합니다.

| 역할 | 책임 | 모델 | reasoning | sandbox |
| --- | --- | --- | --- | --- |
| `pr_explorer` / `explorer` | 코드베이스 매핑, 증거 수집 | `gpt-5.3-codex-spark` 또는 `gpt-5.4-mini` | `medium` | `read-only` |
| `reviewer` | 정확성, 보안, 테스트 리스크 탐색 | `gpt-5.3-codex` 또는 `gpt-5.4` | `high` | `read-only` |
| `docs_researcher` | 프레임워크/API 문서 확인 (MCP 서버 활용) | `gpt-5.4-mini` | `medium` | `read-only` |

서브에이전트 방식의 커스텀 에이전트 파일 예시:

```toml
# .codex/agents/pr-explorer.toml
name = "pr_explorer"
description = "Read-only codebase explorer for gathering evidence before changes are proposed."
model = "gpt-5.4-mini"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"
developer_instructions = """
Stay in exploration mode.
Trace the real execution path, cite files and symbols, and avoid proposing fixes unless the parent agent asks for them.
Prefer fast search and targeted file reads over broad scans.
"""
```

```toml
# .codex/agents/reviewer.toml
name = "reviewer"
description = "PR reviewer focused on correctness, security, and missing tests."
model = "gpt-5.4"
model_reasoning_effort = "high"
sandbox_mode = "read-only"
developer_instructions = """
Review code like an owner.
Prioritize correctness, security, behavior regressions, and missing test coverage.
Lead with concrete findings, include reproduction steps when possible, and avoid style-only comments unless they hide a real bug.
"""
```

```toml
# .codex/agents/docs-researcher.toml
name = "docs_researcher"
description = "Documentation specialist that uses the docs MCP server to verify APIs and framework behavior."
model = "gpt-5.4-mini"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"
developer_instructions = """
Use the docs MCP server to confirm APIs, options, and version-specific behavior.
Return concise answers with links or exact references when available.
Do not make code changes.
"""

[mcp_servers.openaiDeveloperDocs]
url = "https://developers.openai.com/mcp"
```

실행 프롬프트:

```
Review this branch against main. Have pr_explorer map the affected code paths, reviewer find real risks, and docs_researcher verify the framework APIs that the patch relies on.
```

### 패턴 2: 프론트엔드 통합 디버깅 팀

UI 회귀, 불안정한 브라우저 플로우, 또는 애플리케이션 코드와 실행 중인 제품을 교차하는 통합 버그에 유용합니다.

| 역할 | 책임 | 모델 | reasoning | sandbox |
| --- | --- | --- | --- | --- |
| `code_mapper` / `explorer` | 프론트엔드/백엔드 코드 경로 탐색 | `gpt-5.4-mini` | `medium` | `read-only` |
| `browser_debugger` | 브라우저 도구로 이슈 재현 및 증거 수집 | `gpt-5.4` | `high` | `workspace-write` |
| `ui_fixer` / `worker` | 문제 파악 후 최소 수정 구현 | `gpt-5.3-codex-spark` | `medium` | 기본 상속 |

```toml
# .codex/agents/browser-debugger.toml
name = "browser_debugger"
description = "UI debugger that uses browser tooling to reproduce issues and capture evidence."
model = "gpt-5.4"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"
developer_instructions = """
Reproduce the issue in the browser, capture exact steps, and report what the UI actually does.
Use browser tooling for screenshots, console output, and network evidence.
Do not edit application code.
"""

[mcp_servers.chrome_devtools]
url = "http://localhost:3000/mcp"
startup_timeout_sec = 20
```

```toml
# .codex/agents/ui-fixer.toml
name = "ui_fixer"
description = "Implementation-focused agent for small, targeted fixes after the issue is understood."
model = "gpt-5.3-codex-spark"
model_reasoning_effort = "medium"
developer_instructions = """
Own the fix once the issue is reproduced.
Make the smallest defensible change, keep unrelated files untouched, and validate only the behavior you changed.
"""

[[skills.config]]
path = "/Users/me/.agents/skills/docs-editor/SKILL.md"
enabled = false
```

실행 프롬프트:

```
Investigate why the settings modal fails to save. Have browser_debugger reproduce it, code_mapper trace the responsible code path, and ui_fixer implement the smallest fix once the failure mode is clear.
```

---

## CSV 배치 작업 — `spawn_agents_on_csv`

유사한 작업이 여러 개 있고 각 행이 하나의 작업 항목에 매핑될 때 `spawn_agents_on_csv`를 사용합니다. Codex가 CSV를 읽고, 행당 하나의 워커 서브에이전트를 실행(spawn)하며, 전체 배치가 완료될 때까지 대기한 후 결합된 결과를 CSV로 내보냅니다.

### 적합한 사용 사례

- 파일, 패키지, 서비스별 리뷰
- 인시던트, PR, 마이그레이션 대상 목록 확인
- 유사한 입력에 대한 구조화된 요약 생성

### 도구 파라미터

| 파라미터 | 설명 |
| --- | --- |
| `csv_path` | 소스 CSV 경로 |
| `instruction` | 워커 프롬프트 템플릿. `{column_name}` 플레이스홀더 사용 |
| `id_column` | 안정적인 항목 ID로 사용할 특정 열 |
| `output_schema` | 각 워커가 반환할 JSON 객체의 고정된 형태 |
| `output_csv_path` | 결과 CSV 출력 경로 |
| `max_concurrency` | 최대 동시 실행 수 |
| `max_runtime_seconds` | 개별 호출 타임아웃. 글로벌 `job_max_runtime_seconds`보다 우선 |

각 워커는 `report_agent_job_result`를 **정확히 한 번** 호출해야 합니다. 결과를 보고하지 않고 종료하면 해당 행은 에러로 표시됩니다.

### 실행 예시

```
Create /tmp/components.csv with columns path,owner and one row per frontend component.

Then call spawn_agents_on_csv with:
- csv_path: /tmp/components.csv
- id_column: path
- instruction: "Review {path} owned by {owner}. Return JSON with keys path, risk, summary, and follow_up via report_agent_job_result."
- output_csv_path: /tmp/components-review.csv
- output_schema: an object with required string fields path, risk, summary, and follow_up
```

`codex exec`로 실행하면 배치 실행 중 `stderr`에 단일 행 진행 상태가 표시됩니다. 내보낸 CSV에는 원본 행 데이터와 함께 `job_id`, `item_id`, `status`, `last_error`, `result_json` 메타데이터가 포함됩니다.

### 관련 런타임 설정

| 설정 | 설명 |
| --- | --- |
| `agents.max_threads` | 동시 열린 에이전트 스레드 수 상한 |
| `agents.job_max_runtime_seconds` | CSV 팬아웃 작업의 워커별 기본 타임아웃 |
| `sqlite_home` | 에이전트 작업 및 내보낸 결과의 SQLite 상태 저장 위치 |

---

## 서브에이전트 라이프사이클

### 오케스트레이션

Codex는 에이전트 간 오케스트레이션을 처리합니다.

1. 새 서브에이전트 실행(spawning)
2. 후속 명령어 라우팅
3. 결과 대기
4. 에이전트 스레드 종료

여러 에이전트가 실행 중일 때 Codex는 요청된 모든 결과가 준비될 때까지 대기한 후 통합 응답을 반환합니다.

### 샌드박스 및 권한 상속

서브에이전트는 **현재 샌드박스 정책을 상속**합니다.

- 대화형 CLI 세션에서 비활성 에이전트 스레드의 승인 요청이 메인 스레드를 보는 동안 표시될 수 있습니다. 승인 오버레이에 소스 스레드 레이블이 표시되며, `o`를 눌러 해당 스레드를 열 수 있습니다.
- 비대화형 플로우에서는 새 승인이 필요한 동작이 실패하고 에러가 부모 워크플로우에 전달됩니다.
- Codex는 자식을 실행(spawn)할 때 부모 턴의 라이브 런타임 오버라이드를 다시 적용합니다. 여기에는 세션 중 설정한 샌드박스 및 승인 선택(`/permissions` 변경, `--yolo` 등)이 포함됩니다. 커스텀 에이전트 파일에 다른 기본값이 있어도 무시합니다.
- 개별 커스텀 에이전트의 샌드박스 구성을 오버라이드할 수 있습니다 (예: 읽기 전용 모드 명시).

### 제어 흐름

```
┌─────────────────────────────────────┐
│  메인 에이전트 (사용자 대화)         │
│  - 요구사항 정의                     │
│  - 결정 사항 관리                    │
│  - 최종 출력 생성                    │
├──────────────┬──────────────────────┤
│  서브에이전트 1  │  서브에이전트 2  │  서브에이전트 N  │
│  (병렬 실행)     │  (병렬 실행)     │  (병렬 실행)     │
│  - 탐색/분석     │  - 테스트/검증    │  - 문서 확인     │
│  - 요약 반환     │  - 요약 반환      │  - 요약 반환     │
└──────────────┴──────────────────────┘
         │                │               │
         ▼                ▼               ▼
   ┌─────────────────────────────────────┐
   │  통합 응답 (Consolidated Response)   │
   └─────────────────────────────────────┘
```

---

## CLI 명령어

### `/agent` — 에이전트 스레드 전환

CLI에서 활성 에이전트 스레드 간 전환하고 진행 중인 스레드를 검사합니다.

### `/experimental` — 실험적 기능 활성화

멀티 에이전트 워크플로우 등 실험적 기능을 켭니다. **Multi-agents**를 활성화한 후 Codex를 재시작해야 합니다.

### `/ps` — 실행 중인 에이전트 확인

현재 실행 중인 에이전트 스레드를 확인합니다.

### `/stop` — 에이전트 중지

실행 중인 서브에이전트를 중지합니다. Codex에 직접 요청하여 실행 중인 서브에이전트를 조종(steer), 중지(stop), 또는 완료된 에이전트 스레드를 닫을 수도 있습니다.

### `wait` 도구

장시간 실행 명령이나 폴링 워크플로우에 사용합니다. 호출당 최대 1시간까지 폴링 윈도우를 지원합니다. (멀티 에이전트 실험 모드에서 제공)

---

## 모델 및 추론 설정 가이드

서브에이전트와 멀티 에이전트는 동일한 모델 선택 철학을 공유합니다. 모델이나 `model_reasoning_effort`를 고정하지 않으면 Codex가 작업에 맞게 균형을 잡아 선택합니다.

### 모델 선택

| 모델 | 용도 | 특징 |
| --- | --- | --- |
| `gpt-5.5` | 가장 까다로운 에이전트에 사용 | 모호하고 다단계적인 작업에 강점. 계획, 도구 사용, 검증, 후속 작업에 뛰어남 |
| `gpt-5.4` | GPT-5.4에 고정된 워크플로우 | 강력한 코딩, 추론, 도구 사용 |
| `gpt-5.4-mini` | 속도와 효율성 우선 | 탐색, 읽기 중심 스캔, 대용량 파일 리뷰, 문서 처리. 병렬 워커에 적합 |
| `gpt-5.3-codex-spark` | 거의 즉각적인 텍스트 전용 반복 | ChatGPT Pro 필요 (research preview). 지연 시간이 중요할 때 사용 |

### 추론 노력 (`model_reasoning_effort`)

| 값 | 용도 |
| --- | --- |
| `high` | 복잡한 논리 추적, 가정 검증, 엣지 케이스 처리 (리뷰어, 보안 에이전트 등) |
| `medium` | 대부분의 에이전트에 대한 균형 잡힌 기본값 |
| `low` | 간단한 작업에서 속도가 가장 중요할 때 |

추론 노력이 높을수록 응답 시간과 토큰 사용량이 증가하지만 복잡한 작업의 품질이 향상될 수 있습니다.

### 작업 유형별 권장 조합

| 작업 유형 | 추천 모델 | 추론 노력 | sandbox_mode |
| --- | --- | --- | --- |
| 코드 리뷰 / 보안 분석 | `gpt-5.5` 또는 `gpt-5.4` | `high` | `read-only` |
| 코드베이스 탐색 | `gpt-5.4-mini` | `medium` | `read-only` |
| 문서 확인 (MCP) | `gpt-5.4-mini` | `medium` | `read-only` |
| 구현 / 수정 | `gpt-5.4` | `medium` | `workspace-write` |
| 빠른 요약 / 분류 | `gpt-5.4-mini` | `low` | `read-only` |
| 장시간 모니터링 | `gpt-5.4-mini` | `low` | 기본 상속 |

---

## 사용 패턴과 모범 사례

### 서브에이전트 프롬프트 작성

좋은 서브에이전트 프롬프트는 다음을 설명해야 합니다.

1. 작업을 어떻게 나눌 것인지
2. 모든 에이전트가 완료될 때까지 대기할 것인지
3. 어떤 요약이나 출력을 반환할 것인지

```
Review this branch with parallel subagents. Spawn one subagent for security risks,
one for test gaps, and one for maintainability. Wait for all three, then summarize
the findings by category with file references.
```

### 읽기 vs 쓰기 작업

| 작업 유형 | 병렬 안전성 | 비고 |
| --- | --- | --- |
| 읽기 중심 (탐색, 테스트, 분류, 요약) | 권장 | 병렬 에이전트의 이상적인 출발점 |
| 쓰기 중심 (코드 편집) | 주의 필요 | 여러 에이전트가 동시에 코드를 편집하면 충돌 및 조정 오버헤드 증가 |

### 토큰 비용 고려

서브에이전트 워크플로우는 각 서브에이전트가 자체적으로 모델 및 도구 작업을 수행하므로 동등한 단일 에이전트 실행보다 **더 많은 토큰을 소비**합니다.

### 커스텀 에이전트 설계 원칙

1. **좁고 명확하게(opinionated)**: 각 에이전트에 하나의 명확한 작업을 부여
2. **도구 표면 일치**: 작업에 맞는 도구만 노출
3. **인접 작업 방지**: 에이전트가 다른 작업으로 번지지 않도록 명령어 설정
4. **역할 분리**: 탐색은 탐색 전용, 수정은 수정 전용으로 분리
5. **최소 권한**: 읽기만 필요한 에이전트는 `sandbox_mode = "read-only"`로 설정

### `spawn_agents_on_csv` 활용 팁

- 각 워커가 `report_agent_job_result`를 정확히 한 번 호출하도록 `instruction`에 명시
- `output_schema`를 지정하면 결과의 일관성 보장
- `max_concurrency`로 동시 실행 수 제어
- `sqlite_home`으로 상태 저장 위치 관리

---

## 서브에이전트 vs 멀티 에이전트 비교

현재 Codex 문서에서 서브에이전트(subagents)와 멀티 에이전트(multi-agents)는 동일한 기능의 다른 버전으로 보입니다. 주요 차이점은 다음과 같습니다.

| 구분 | 서브에이전트 (Subagents) | 멀티 에이전트 (Multi-agents) |
| --- | --- | --- |
| 상태 | 현재 릴리스, 기본 활성화 | 실험적, 명시적 활성화 필요 |
| 활성화 방법 | 기본 활성화 | `/experimental` 또는 `[features] multi_agent = true` |
| 에이전트 정의 방식 | 독립 TOML 파일 (`~/.codex/agents/` 또는 `.codex/agents/`) | `[agents.<name>]` 역할 테이블 + `config_file` |
| 필수 필드 | `name`, `description`, `developer_instructions` | `description` (선택), `config_file` (선택) |
| 자동 스폰 | 사용자 명시적 요청만 | Codex 자동 판단 또는 사용자 명시적 요청 |
| `monitor` 역할 | 제공 안 함 | 장시간 실행/폴링에 최적화된 `monitor` 역할 제공 |
| `wait` 도구 | 명시되지 않음 | 호출당 최대 1시간 폴링 지원 |
| UI 지원 | Codex App + CLI | CLI만 (App/IDE는 예정) |
| 용어 | subagent, agent thread | multi-agent, sub-agent, agent thread |
| 모델 | `gpt-5.5`, `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.3-codex-spark` | `gpt-5.3-codex`, `gpt-5.3-codex-spark` |

> **참고**: 두 문서는 동일한 기능의 다른 버전에 대한 것으로 보입니다. 서브에이전트 문서가 최신 버전을, 멀티 에이전트 문서가 이전 실험 버전을 설명할 가능성이 높습니다. 프로젝트에서는 서브에이전트 방식(독립 TOML 파일)을 우선 사용하는 것을 권장합니다.
