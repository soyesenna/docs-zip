# Codex CLI - CLI 명령어 및 플래그

> CLI 기본 사용법, 플래그, 슬래시 명령어, 비대화형 모드, MCP/플러그인 CLI 명령어, Codex SDK

> **대조 기준(출처)**: [developers.openai.com/codex/cli/features](https://developers.openai.com/codex/cli/features) | [developers.openai.com/codex/cli/reference](https://developers.openai.com/codex/cli/reference) | [developers.openai.com/codex/cli/slash-commands](https://developers.openai.com/codex/cli/slash-commands) | [developers.openai.com/codex/noninteractive](https://developers.openai.com/codex/noninteractive) | [developers.openai.com/codex/sdk](https://developers.openai.com/codex/sdk) | [developers.openai.com/codex/github-action](https://developers.openai.com/codex/github-action) | [developers.openai.com/codex/app-server](https://developers.openai.com/codex/app-server) | [developers.openai.com/codex/mcp](https://developers.openai.com/codex/mcp)

---

## CLI 기본 사용법

### 대화형 모드

```shell
# 대화형 TUI 시작
codex

# 프롬프트와 함께 시작
codex "이 코드베이스를 설명해줘"

# 특정 디렉토리에서 실행
codex --cd /path/to/project

# 여러 디렉토리에 쓰기 권한 부여
codex --cd apps/frontend --add-dir ../backend --add-dir ../shared
```

### 비대화형 모드 (exec)

```shell
# 단일 실행
codex exec "CI 실패를 수정해줘"

# JSON 출력
codex exec --json "버그를 찾아줘"

# 이전 실행 재개
codex exec resume --last "찾은 경쟁 상태를 수정해줘"
codex exec resume <SESSION_ID> "계획을 구현해줘"

# Git 리포지토리 검사 건너뛰기
codex exec --skip-git-repo-check "스크립트 실행"
```

### 인증 (`codex login` / `codex logout`)

`codex login`은 ChatGPT OAuth, device auth, 또는 stdin으로 전달된 API 키로 인증합니다. 플래그 없이 실행하면 ChatGPT OAuth 브라우저 플로우를 엽니다.

| 키/서브커맨드 | 값 | 설명 |
| --- | --- | --- |
| `--with-api-key` | `boolean` | stdin에서 API 키 읽기. 예: `printenv OPENAI_API_KEY \| codex login --with-api-key` |
| `--with-access-token` | `boolean` | stdin에서 액세스 토큰 읽기 |
| `--device-auth` | `boolean` | 디바이스 인증 플로우 사용 |
| `codex login status` | 서브커맨드 | 활성 인증 모드 출력. 로그인 상태면 종료 코드 `0` |

```shell
# 기본 OAuth 로그인
codex login

# API 키로 로그인 (stdin)
printenv OPENAI_API_KEY | codex login --with-api-key

# 상태 확인 (자동화에서 유용)
codex login status

# 로그아웃 (API 키 및 ChatGPT 자격증명 모두 제거)
codex logout
```

> `codex login status`는 자격증명이 있으면 종료 코드 `0`으로 종료하여 자동화 스크립트에서 유용합니다.

### 기타 CLI 명령어

> 아래 명령어는 공식 CLI reference의 Command overview에 문서화된 항목입니다. 출처: [CLI reference](https://developers.openai.com/codex/cli/reference/).

```shell
# Cloud 태스크 diff 패치 적용 (별칭: codex a)
codex apply <TASK_ID>

# 샌드박스 관리 (macOS seatbelt / Linux landlock)
codex sandbox

# MCP 서버 관리 (list, add, remove, authenticate)
codex mcp

# Codex 자체를 stdio 기반 MCP 서버로 실행
codex mcp-server

# 로컬 App Server 시작 (개발·디버그용)
codex app-server

# 터미널에서 Codex Cloud 태스크 조회·실행
codex cloud
```

```shell
# 세션 재개
codex resume                # 세션 선택 피커 열기
codex resume --all          # 모든 세션 표시
codex resume --last         # 가장 최근 세션 재개
codex resume <SESSION_ID>   # 특정 세션 재개
```

> 참고: `codex doctor`, `codex update`, `codex remote-control`, `codex execpolicy check`, `codex debug models/app-server`, `codex fork` 등은 공식 CLI reference(developers.openai.com/codex/cli/reference)의 Command overview에 문서화되어 있지 않습니다. CLI에 존재하더라도 공식 카탈로그 기준이 아니므로 사용 전 `codex --help`로 확인하세요.

---

## CLI 플래그

### 명령어 개요

| 명령어 | 상태 | 설명 |
| --- | --- | --- |
| `codex` | stable | TUI 실행. 글로벌 플래그와 선택적 프롬프트·이미지 첨부 허용 |
| `codex exec` (별칭 `codex e`) | stable | 비대화형 실행. stdout 또는 JSONL로 결과 스트리밍, 세션 재개 지원 |
| `codex login` | stable | ChatGPT OAuth, device auth, 또는 stdin API 키로 인증 |
| `codex logout` | stable | 저장된 인증 자격증명 제거 |
| `codex resume` | stable | 이전 대화형 세션을 ID로 재개하거나 최근 세션 재개 |
| `codex apply` (별칭 `codex a`) | stable | Codex Cloud 태스크의 최신 diff를 로컬 작업 트리에 적용 |
| `codex sandbox` | platform-specific | macOS seatbelt 또는 Linux landlock 샌드박스에서 임의 명령 실행 |
| `codex completion` | stable | Bash, Zsh, Fish, PowerShell용 셸 완성 스크립트 생성 |
| `codex cloud` | stable | 터미널에서 Codex Cloud 태스크 조회·실행 |
| `codex mcp` | experimental | MCP 서버 관리 (list, add, remove, authenticate) |
| `codex mcp-server` | experimental | Codex 자체를 stdio 기반 MCP 서버로 실행 |
| `codex app-server` | 개발·디버그용 | 로컬 app-server 실행. 사전 공지 없이 변경될 수 있음 |

> 출처: [CLI reference - Command overview](https://developers.openai.com/codex/cli/reference/).

### 글로벌 플래그

글로벌 플래그는 기본 `codex` 명령에 적용되며, 별도 표기가 없는 한 서브커맨드로 전파됩니다. CLI는 대부분의 기본값을 `~/.codex/config.toml`에서 상속하며, 명령행에서 전달한 `-c key=value` 오버라이드가 해당 호출에 한해 우선합니다.

| 플래그 | 단축 | 값 | 설명 |
| --- | --- | --- | --- |
| `PROMPT` | - | `string` | 세션 시작 시 전달할 텍스트 지시. 생략하면 TUI만 실행 |
| `--image` | `-i` | `path[,path...]` | 초기 프롬프트에 이미지 첨부. 쉼표 구분 또는 플래그 반복 |
| `--model` | `-m` | `string` | 설정 모델 오버라이드 (예: `gpt-5-codex`) |
| `--oss` | - | `boolean` | 로컬 오픈소스 모델 제공자 사용 (`-c model_provider="oss"`와 동일). Ollama 실행 여부 검증 |
| `--profile` | `-p` | `string` | `~/.codex/config.toml`에서 로드할 설정 프로필 이름 |
| `--sandbox` | `-s` | `read-only \| workspace-write \| danger-full-access` | 모델 생성 셸 명령의 샌드박스 정책 |
| `--ask-for-approval` | `-a` | `untrusted \| on-failure \| on-request \| never` | 명령 실행 전 사람 승인을 요구하는 시점 제어 |
| `--full-auto` | - | `boolean` | 무인 로컬 작업용 shortcut. `--ask-for-approval on-failure`와 `--sandbox workspace-write` 설정 |
| `--dangerously-bypass-approvals-and-sandbox` | `--yolo` | `boolean` | 모든 승인·샌드박스 우회. 외부에서 격리된 환경에서만 사용 |
| `--cd` | `-C` | `path` | 에이전트 시작 전 작업 디렉토리 설정 |
| `--config` | `-c` | `key=value` | 설정 오버라이드 (반복 가능). 예: `-c approval_policy="never"` |
| `--add-dir` | - | `path` | 추가 쓰기 가능 디렉토리. `--sandbox danger-full-access`보다 권장 |
| `--search` | - | `boolean` | 라이브 웹 검색 도구 활성화 |
| `--disable` | - | `feature` | 지정한 실험적 기능 비활성화 |
| `--enable` | - | `feature` | 지정한 실험적 기능 활성화 (예: `--enable rmcp_client`) |
| `--dangerously-bypass-hook-trust` | - | `boolean` | 훅 신뢰 검사를 우회 (위험) |
| `--remote-auth-token-env` | - | `ENV_VAR` | 원격 서버 인증 토큰을 읽어올 환경 변수 |
| `--strict-config` | - | `boolean` | 설정 검증을 엄격하게 적용 |
| `--no-alt-screen` | - | `boolean` | 대체 화면 모드 비활성화. 스크롤백 보존 (Zellij 등에 유용) |
| `--remote` | - | `ws:// \| wss:// \| unix://` | 원격 TUI 서버에 연결 |
| `--version` | - | - | 버전 정보 표시 |
| `--help` | - | - | 도움말 표시 |

> 출처: [CLI reference - Global flags](https://developers.openai.com/codex/cli/reference/). `--config/-c`, `--dangerously-bypass-hook-trust`, `--disable`, `--enable`, `--remote-auth-token-env`, `--strict-config`는 reference 페이지의 "Expand to view all"에 문서화된 확장 글로벌 플래그입니다.

### 샌드박스 플래그 (`--sandbox` / `-s`)

| 값 | 설명 |
| --- | --- |
| `read-only` | 파일 읽기만 허용. 변경이나 명령 실행 불가 |
| `workspace-write` | 작업 디렉토리 내에서 파일 수정 및 명령 실행 허용 |
| `danger-full-access` | 전체 시스템 접근 허용 (위험) |

### 승인 정책 플래그 (`--ask-for-approval` / `-a`)

| 값 | 설명 |
| --- | --- |
| `untrusted` | 모든 명령에 승인 필요 |
| `on-failure` | 실패한 명령에만 승인 요청. `--full-auto` 프리셋이 이 값을 사용 |
| `on-request` | 샌드박스 외부 작업에만 승인 요청 |
| `never` | 승인 없이 자동 실행 |

> **권장 (공식 noninteractive 가이드 기준)**: 비대화형/자동화에서는 `--full-auto`(= `--ask-for-approval on-failure` + `--sandbox workspace-write`)를 우선 사용하세요. `danger-full-access`/`--yolo`는 격리된 CI 러너나 컨테이너 같은 controlled environment에서만 사용해야 합니다. 출처: [Non-interactive mode](https://developers.openai.com/codex/noninteractive/).

### 위험 플래그

| 플래그 | 설명 |
| --- | --- |
| `--dangerously-bypass-approvals-and-sandbox` (`--yolo`) | 모든 승인 및 샌드박스 우회. **매우 위험**. 외부에서 샌드박스된 환경에서만 사용 |

> `--yolo`는 `--ask-for-approval` 및 `--full-auto`와 함께 사용할 수 없습니다. `--full-auto`는 샌드박스 VM이나 전용 러너 같은 격리된 환경이 아니면 `--yolo`와 조합하지 마세요.

---

## 승인 모드 (Approval Modes)

세션 중 `/approvals` 명령으로 전환할 수 있습니다.

| 모드 | 설명 |
| --- | --- |
| **Auto** (기본값) | 작업 디렉토리 내에서 파일 읽기/수정/명령 실행. 외부 접근 시 승인 요청 |
| **Read-only** | 파일 탐색만 가능. 변경이나 명령 실행은 계획 승인 후에만 가능 |
| **Full Access** | 네트워크 포함 전체 시스템 접근. 신뢰할 수 있는 리포지토리와 작업에만 사용 |

---

## 슬래시 명령어 목록

대화형 세션에서 `/`를 입력하면 명령어 목록이 나타납니다. 작업 실행 중에 `Tab`을 누르면 다음 턴에 실행할 명령어를 큐에 추가할 수 있습니다.

### 빌트인 슬래시 명령어 (공식)

> 공식 slash-commands 페이지는 다음 명령어들을 빌트인으로 문서화합니다. 출처: [Slash commands](https://developers.openai.com/codex/cli/slash-commands/).

| 명령어 | 용도 | 언제 사용할까 |
| --- | --- | --- |
| `/approvals` | 승인 모드(Auto / Read-only / Full Access) 설정 | 세션 도중 승인 요건을 완화하거나 강화 |
| `/compact` | 대화 요약하여 토큰 확보 | 긴 작업 후 컨텍스트 윈도우 절약 |
| `/diff` | Git diff 표시 (미추적 파일 포함) | 커밋·테스트 전 편집 검토 |
| `/exit` | CLI 종료 (`/quit`과 동일) | 다른 철자; 둘 다 세션 종료 |
| `/feedback` | 로그 및 진단 정보를 유지보수자에게 전송 | 이슈 리포트, 지원에 진단 공유 |
| `/init` | `AGENTS.md` 스캐폴드 생성 | 저장소 영구 지시문 캡처 |
| `/logout` | Codex에서 로그아웃 | 공유 머신에서 자격증명 제거 |
| `/mcp` | 설정된 MCP 도구 목록 표시 | 세션에서 사용 가능한 외부 도구 확인 |
| `/mention` | 파일을 대화에 첨부 | 특정 파일·폴더 검사 지시 |
| `/model` | 활성 모델 및 reasoning effort 선택 | 작업 전 모델 전환 |
| `/fork` | 저장된 대화를 새 스레드로 포크 | 원본 보존하며 다른 접근 시도 |
| `/resume` | 저장된 세션 목록에서 대화 재개 | 이전 CLI 세션 이어서 진행 |
| `/new` | 같은 CLI 세션에서 새 대화 시작 | 컨텍스트 리셋, 같은 저장소에서 새 프롬프트 |
| `/quit` | CLI 종료 | 세션 즉시 종료 |
| `/review` | 워킹 트리 리뷰 실행 | 작업 완료 후 또는 로컬 변경사항 재검토 |
| `/status` | 세션 설정 및 토큰 사용량 표시 | 활성 모델·승인 정책·writable roots 확인 |

> `/quit`과 `/exit`은 모두 CLI를 종료합니다. 중요한 작업을 저장하거나 커밋한 뒤에 사용하세요.

### 커스텀 프롬프트

공식 빌트인 외에 `/prompts: <name>` 형태로 동작하는 **커스텀 프롬프트**를 만들 수 있습니다. 인자와 메타데이터를 갖는 재사용 가능한 프롬프트로, 새로운 슬래시 명령어처럼 동작합니다. 자세한 내용은 공식 [Custom Prompts](https://developers.openai.com/codex/cli/slash-commands/) 가이드를 참조하세요.

### 비공식 명령어 (참고용)

> 아래 명령어들은 공식 slash-commands 페이지나 CLI reference에 문서화되어 있지 않습니다. 실제 CLI에 존재할 수 있으나, 공식 카탈로그 기준이 아니므로 버전에 따라 동작이 다를 수 있습니다. 사용 전 `codex --help` 또는 세션 내 `/` 입력으로 확인하세요.

대표적으로 알려진 비공식 명령어: `/clear`, `/side`, `/btw`, `/archive`, `/fast`, `/personality`, `/approve`, `/experimental`, `/plan`, `/goal`, `/copy`, `/ide`, `/skills`, `/apps`, `/plugins`, `/hooks`, `/agent`, `/ps`, `/stop`(`/clean`), `/debug-config`, `/statusline`, `/title`, `/theme`, `/keymap`, `/vim`, `/raw`, `/sandbox-add-read-dir`, `/memories` 등.

---

## 비대화형 모드 상세

### `codex exec`

CI/CD, 스크립트, 자동화에 적합한 모드입니다. 단축형 `codex e` 지원.

```shell
# 기본 실행
codex exec "fix the CI failure"

# JSON 출력 (프로그램에서 파싱)
codex exec --json "find bugs"

# stdin에서 프롬프트 파이프
echo "요약해줘" | codex exec -

# 세션 재개
codex exec resume --last "continue fixing"
codex exec resume <SESSION_ID> "implement the plan"
```

### exec 플래그

`codex exec`는 글로벌 플래그 외에 다음을 추가로 지정할 수 있습니다.

| 플래그 | 단축 | 값 | 설명 |
| --- | --- | --- | --- |
| `PROMPT` | - | `string \| -` | 초기 작업 지시. `-`이면 stdin에서 읽음 |
| `--image` | `-i` | `path[,path...]` | 첫 메시지에 이미지 첨부 (반복 가능) |
| `--model` | `-m` | `string` | 이 실행에 한해 모델 오버라이드 |
| `--oss` | - | `boolean` | 로컬 오픈소스 제공자 사용 (Ollama 실행 필요) |
| `--sandbox` | `-s` | `read-only \| workspace-write \| danger-full-access` | 샌드박스 정책. 기본값은 설정 |
| `--profile` | `-p` | `string` | config.toml의 설정 프로필 선택 |
| `--full-auto` | - | `boolean` | `workspace-write` 샌드박스 + on-failure 승인 자동화 프리셋 |
| `--dangerously-bypass-approvals-and-sandbox` | `--yolo` | `boolean` | 승인·샌드박스 우회 (위험) |
| `--cd` | `-C` | `path` | 작업 영역 루트 설정 |
| `--skip-git-repo-check` | - | `boolean` | Git 리포지토리 외부 실행 허용 |
| `--config` | `-c` | `key=value` | 설정 오버라이드 (**반복 가능**) |
| `--ephemeral` | - | `boolean` | 세션 롤아웃 저장 없이 일회성 실행 |
| `--ignore-rules` | - | `path` | 지정한 규칙 파일을 무시 |
| `--ignore-user-config` | - | `boolean` | 사용자 수준 config 무시 |
| `--color` | - | `auto \| always \| never` | 색상 출력 제어 |

### exec resume 서브커맨드

| 키 | 값 | 설명 |
| --- | --- | --- |
| `SESSION_ID` | `uuid` | 지정 세션 재개. 생략하고 `--last` 사용 시 최근 세션 |
| `--last` | `boolean` | 피커 건너뛰고 최근 대화 자동 재개 |
| `--all` | `boolean` | 현재 작업 디렉토리 밖의 세션까지 표시 |
| `PROMPT` | `string \| -` | 재개 직후 보낼 후속 지시. `-`이면 stdin |

### exec 출력 옵션

| 플래그 | 단축 | 설명 |
| --- | --- | --- |
| `--output-last-message` | `-o` | 최종 메시지를 파일에 기록 (stdout에도 출력) |
| `--output-schema` | - | JSON Schema 경로. 최종 응답을 해당 스키마에 맞게 구조화 |
| `--json` | - | stdout을 JSONL 이벤트 스트림으로 전환 |

### 환경 변수

| 변수 | 설명 |
| --- | --- |
| `CODEX_API_KEY` | OpenAI API 키. `codex exec`에서만 지원. `OPENAI_API_KEY` 대신 사용 가능 |

### JSONL 이벤트 타입

`--json` 사용 시 stdout에 JSONL 형식으로 이벤트가 스트리밍됩니다. 이벤트 타입은 **점 표기법(dot notation)**을 사용합니다 (`thread.started`, `turn.completed` 등).

| 이벤트 타입 | 설명 |
| --- | --- |
| `thread.started` | 스레드 시작 (`thread_id` 포함) |
| `turn.started` | 턴 시작 |
| `turn.completed` | 턴 완료 (`turn.status`: `completed`, `interrupted`, `failed`) |
| `turn.failed` | 턴 실패. 별도의 `error` 이벤트 후 `turn.completed`로 종료 |
| `error` | 턴 실패 시 `{ error: { message, codexErrorInfo?, additionalDetails? } }` 전송 |
| `item.started` | 작업 항목 시작 |
| `item.completed` | 작업 항목 완료 |
| `item.agentMessage/delta` | 에이전트 메시지 스트리밍 |
| `item.commandExecution/outputDelta` | 명령 실행 stdout/stderr |
| `item/fileChange` | 파일 변경 제안 |
| `item/mcpToolCall` | MCP 도구 호출 |
| `item/webSearch` | 웹 검색 요청 |
| `item/plan/delta` | 계획 텍스트 스트리밍 |
| `item/reasoning/summaryTextDelta` | 추론 요약 |
| `thread.tokenUsage/updated` | 토큰 사용량 업데이트 |
| `turn.diff/updated` | 통합 diff 업데이트 |
| `turn.plan/updated` | 계획 상태 변경 |

> 턴 실패 시 `error` 이벤트의 `codexErrorInfo` 필드에 오류 상세가 포함됩니다: `ContextWindowExceeded`, `UsageLimitExceeded`, `HttpConnectionFailed` (4xx/5xx 업스트림), `ResponseStreamConnectionFailed`, `ResponseStreamDisconnected`, `ResponseTooManyFailedAttempts`, `BadRequest`, `Unauthorized`, `SandboxError`, `InternalServerError`, `Other`. 업스트림 HTTP 상태 코드가 있으면 `codexErrorInfo.httpStatusCode`로 전달됩니다.

샘플 JSONL 스트림 (각 줄이 하나의 JSON 객체):

```json
{"type":"thread.started","thread_id":"0199a213-81c0-7800-8aa1-bbab2a035a53"}
{"type":"turn.started"}
{"type":"item.started","item":{"id":"item_1","type":"command_execution","command":"bash -lc ls","status":"in_progress"}}
{"type":"item.completed","item":{"id":"item_3","type":"agent_message","text":"Repo contains docs, sdk, and examples directories."}}
{"type":"turn.completed","usage":{"input_tokens":24763,"cached_input_tokens":24448,"output_tokens":122}}
```

### 단일 프롬프트 모드

대화형 UI 없이 빠르게 응답을 얻을 수 있습니다:

```shell
codex "이 코드베이스를 설명해줘"
```

Codex가 작업 디렉토리를 읽고 계획을 수립한 뒤 터미널에 응답을 스트리밍하고 종료합니다. `--cd`로 특정 디렉토리를 지정하거나 `--model`로 동작을 조정할 수 있습니다.

---

## MCP CLI 명령어

### 서버 관리

```shell
# MCP 도구 목록 확인
codex mcp list

# MCP 서버 추가
codex mcp add <server-name>

# MCP 서버 추가 (원격 URL)
codex mcp add <server-name> --url <URL>

# Bearer 토큰 환경 변수 지정
codex mcp add <server-name> --bearer-token-env-var <ENV_VAR_NAME>

# MCP 서버 정보 조회
codex mcp get <server-name>

# MCP 서버 제거
codex mcp remove <server-name>

# MCP 인증 로그인
codex mcp login <server-name>

# MCP 로그아웃
codex mcp logout <server-name>
```

### 세션 내 명령어

```
/mcp                  # 설정된 MCP 도구 목록
/mcp verbose          # 서버 진단 정보 포함
```

---

## 플러그인 CLI 명령어

플러그인 관리는 로컬 설치된 플러그인(`codex plugin`)과 마켓플레이스(`codex plugin marketplace`)로 분리됩니다.

### 로컬 플러그인 관리 (`codex plugin`)

```shell
# 설치된 플러그인 목록 (--json 지원)
codex plugin list
codex plugin list --json

# 마켓플레이스에서 플러그인 추가
codex plugin add <plugin-name> --marketplace <marketplace-name>

# 플러그인 제거
codex plugin remove <plugin-name>
```

| 키 | 값 | 설명 |
| --- | --- | --- |
| `add <name>` | `-m, --marketplace <name>` | 지정 마켓플레이스에서 플러그인 추가 |
| `list` | `--json` | 설치된 플러그인 목록. `--json`으로 기계 판독 가능 |
| `remove <name>` | - | 플러그인 제거 |

### 마켓플레이스 관리 (`codex plugin marketplace`)

```shell
# 등록된 마켓플레이스의 설치 가능 플러그인 목록
codex plugin marketplace list

# 마켓플레이스에서 플러그인 설치
codex plugin marketplace add <plugin-name>

# 플러그인 업그레이드
codex plugin marketplace upgrade <plugin-name>

# 플러그인 제거
codex plugin marketplace remove <plugin-name>
```

### 세션 내 명령어

```
/plugins              # 설치된 플러그인 탐색 및 관리
```

---

## Feature 플래그 관리

> **주의**: `codex features` 서브커맨드는 공식 CLI reference(developers.openai.com/codex/cli/reference)의 Command overview에 문서화되어 있지 않습니다. 아래 내용은 실제 CLI에서 사용 가능할 수 있으나, 공식 카탈로그 기준이 아닙니다. 사용 전 `codex --help`로 확인하세요.

```shell
# 사용 가능한 기능 목록
codex features list

# 기능 활성화
codex features enable unified_exec

# 기능 비활성화
codex features disable shell_snapshot
```

> `codex features enable/disable`은 `$CODEX_HOME/config.toml`에 기록됩니다. `codex features`는 `--profile`을 지원하지 않습니다 — 설정 프로필 기반의 기능 토글은 직접 `config.toml`의 `[features]` 섹션을 편집하세요.

> 공식 문서에서 기능 플래그 관련 안내: `codex mcp login`은 RMCP 클라이언트 기능(`[features].rmcp_client = true` 또는 `codex --enable rmcp_client`)이 필요합니다. 출처: [CLI reference](https://developers.openai.com/codex/cli/reference/).

---

## Cloud 명령어

터미널에서 Codex Cloud 태스크를 조회·실행합니다. 인증은 메인 CLI와 동일합니다. 제출 실패 시 0이 아닌 종료 코드로 종료합니다.

```shell
# Cloud 태스크 대화형 피커
codex cloud

# 태스크 목록 조회 (별칭: codex cloud-tasks)
codex cloud list
codex cloud list --env ENV_ID --json --limit 50
codex cloud list --cursor <CURSOR>

# Cloud 태스크 직접 실행
codex cloud exec --env ENV_ID "버그 요약"

# Best-of-N 실행 (1-4회 시도)
codex cloud exec --env ENV_ID --attempts 3 "버그 요약"
```

### `codex cloud` 키

| 키 | 값 | 설명 |
| --- | --- | --- |
| `QUERY` | `string` | 태스크 프롬프트. 생략 시 대화형으로 요청 |
| `--env` | `ENV_ID` | 대상 Codex Cloud 환경 ID (필수) |
| `--attempts` | `1-4` | best-of-N 실행 횟수 |

### `codex cloud list` 서브커맨드

| 플래그 | 설명 |
| --- | --- |
| `--env` | 환경 ID 필터 |
| `--cursor` | 커서 기반 페이지네이션 |
| `--limit` | 페이지 크기 |
| `--json` | 기계 판독 가능한 JSON 출력 |

---

## 샌드박스 (`codex sandbox`)

Codex가 내부적으로 사용하는 동일한 정책으로 명령을 실행하는 헬퍼입니다. 플랫폼별로 동작합니다.

### macOS seatbelt

| 키 | 값 | 설명 |
| --- | --- | --- |
| `--full-auto` | `boolean` | 현재 작업공간과 `/tmp`에 승인 없이 쓰기 권한 부여 |
| `--config` | `-c` `key=value` | 샌드박스 실행에 설정 오버라이드 전달 (반복 가능) |
| `--permissions-profile` | `-P` | 권한 프로필 지정 (0.139.0 이상 alias) |
| `--cd` | `-C` `path` | 작업 디렉토리 설정 |
| `--allow-unix-socket` | `boolean` | Unix 소켓 접근 허용 (macOS) |
| `--log-denials` | `boolean` | 거부된 작업을 로깅 (macOS) |
| `COMMAND...` | `var-args` | macOS Seatbelt 하에서 실행할 셸 명령. `--` 이후 전달 |

### Linux landlock

| 키 | 값 | 설명 |
| --- | --- | --- |
| `--full-auto` | `boolean` | Landlock 샌드박스 내에서 작업공간과 `/tmp`에 쓰기 권한 부여 |
| `--config` | `-c` `key=value` | 샌드박스 실행 전 적용할 설정 오버라이드 (반복 가능) |
| `--permissions-profile` | `-P` | 권한 프로필 지정 (0.139.0 이상 alias) |
| `--cd` | `-C` `path` | 작업 디렉토리 설정 |
| `COMMAND...` | `var-args` | Landlock + seccomp 하에서 실행할 명령. `--` 이후 전달 |

> 출처: [CLI reference - codex sandbox](https://developers.openai.com/codex/cli/reference/). `--permissions-profile/-P`, `--cd/-C`, macOS의 `--allow-unix-socket`/`--log-denials`는 reference 페이지의 "Expand to view all"에 문서화되어 있습니다.

---

## App Server (`codex app-server`)

Codex가 리치 클라이언트(예: VS Code 확장)를 구동하는 데 사용하는 인터페이스입니다. 인증, 대화 기록, 승인, 스트리밍 에이전트 이벤트가 필요한 제품 내 깊은 통합에 사용합니다. CI/자동화에는 Codex SDK를 사용하세요.

MCP와 마찬가지로 `codex app-server`는 양방향 통신을 지원하며 **stdio로 JSONL을 스트리밍**합니다. 프로토콜은 JSON-RPC 2.0이지만 `"jsonrpc":"2.0"` 헤더는 생략합니다. 출처: [App Server](https://developers.openai.com/codex/app-server/).

```shell
# 로컬 App Server 시작 (stdio 대기)
codex app-server
```

서버는 stdin으로 JSONL을 기다리고 프로토콜 메시지만 stdout에 출력합니다.

### 메시지 형식

요청은 `method`, `params`, `id`를 포함합니다:

```json
{ "method": "thread/start", "id": 10, "params": { "model": "gpt-5.1-codex" } }
```

응답은 동일한 `id`와 함께 `result` 또는 `error`를 반환합니다:

```json
{ "id": 10, "result": { "thread": { "id": "thr_123" } } }
```

```json
{ "id": 10, "error": { "code": 123, "message": "Something went wrong" } }
```

알림(Notification)은 `id`를 생략하고 `method`, `params`만 사용합니다:

```json
{ "method": "turn/started", "params": { "turn": { "id": "turn_456" } } }
```

### 핵심 개념

- **Thread**: 사용자와 Codex 에이전트 간의 대화. 턴(turn)을 포함합니다.
- **Turn**: 하나의 사용자 요청과 그에 따른 에이전트 작업. 아이템(item)과 증분 업데이트를 스트리밍합니다.
- **Item**: 입력 또는 출력의 단위 (사용자 메시지, 에이전트 메시지, 명령 실행, 파일 변경, 도구 호출 등).

### 연결 절차

1. `codex app-server`로 서버 시작. stdin으로 JSONL을 대기하고 프로토콜 메시지만 출력합니다.
2. stdio로 클라이언트를 연결한 뒤 `initialize` 요청 전송, 이어서 `initialized` 알림 전송.
3. 스레드와 턴을 시작하고 stdout에서 알림을 계속 읽습니다.

> 클라이언트는 연결당 반드시 한 번 `initialize` 요청을 보내야 하며, 그 전의 요청은 거부됩니다. `initialize`에는 `clientInfo`(name/title/version)를 포함해 자신의 통합을 식별합니다. `clientInfo.name`은 OpenAI Compliance Logs Platform에서 클라이언트 식별에 사용됩니다.

### 스키마 생성

실행한 Codex 버전에 정확히 일치하는 TypeScript 스키마 또는 JSON Schema 번들을 생성할 수 있습니다.

```shell
codex app-server generate-ts --out ./schemas
codex app-server generate-json-schema --out ./schemas
```

### experimentalApi capability

일부 메서드/필드는 `experimentalApi` capability로 게이트됩니다.

- `capabilities`(또는 `experimentalApi: false`)를 생략하면 안정적인 API 표면만 사용하며, 실험적 메서드/필드는 거부됩니다.
- `capabilities.experimentalApi: true`로 설정하면 실험적 메서드와 필드가 활성화됩니다.

옵트인 없이 실험적 메서드/필드를 보내면 다음 오류로 거부됩니다:

```
<descriptor> requires experimentalApi capability
```

### 핵심 JSON-RPC 메서드

App Server는 thread/turn/item 라이프사이클 메서드를 제공합니다. 주요 메서드:

| 메서드 | 설명 |
| --- | --- |
| `initialize` / `initialized` | 연결당 1회 핸드셰이크. 이 전의 요청은 `Not initialized` 오류로 거부 |
| `thread/start` · `thread/resume` · `thread/fork` | 새 스레드 생성(공식 예시 모델 `gpt-5.1-codex`), 기존 재개, 분기 |
| `thread/list` · `thread/read` · `thread/loaded/list` | 커서 페이지네이션 목록, ID로 읽기(재개 없음), 메모리 로드 목록 |
| `thread/archive` · `thread/unarchive` | 스레드 보관 / 복원 |
| `thread/compact/start` · `thread/rollback` | 기록 압축, 마지막 N턴 롤백 |
| `turn/start` · `turn/steer` · `turn/interrupt` | 턴 시작, 진행 중 입력 추가, 취소 |
| `review/start` | 리뷰어 실행 (`enteredReviewMode`/`exitedReviewMode` 스트림) |
| `command/exec` | 스레드 없이 단일 명령을 샌드박스에서 실행 |
| `model/list` · `experimentalFeature/list` | 모델·기능 플래그 조회 |
| `skills/list` · `app/list` · `mcpServerStatus/list` | 스킬·앱·MCP 서버 상태 조회 |
| `account/read` · `account/login/start` · `account/logout` | 인증 모드 조회·로그인·로그아웃 |
| `config/read` · `config/value/write` · `config/batchWrite` | 설정 조회·단일 키 쓰기·원자적 일괄 쓰기 |

### 모델 목록 응답 예시 (`model/list`)

공식 app-server 페이지의 `model/list` 응답 예시 (모델 값은 출처 페이지 기준):

```json
{ "method": "model/list", "id": 6, "params": { "limit": 20 } }
{ "id": 6, "result": {
  "data": [{
    "id": "gpt-5.2-codex",
    "model": "gpt-5.2-codex",
    "upgrade": "gpt-5.3-codex",
    "displayName": "GPT-5.2 Codex",
    "defaultReasoningEffort": "medium",
    "reasoningEffort": [{ "effort": "low", "description": "Lower latency" }],
    "inputModalities": ["text", "image"],
    "supportsPersonality": true,
    "isDefault": true
  }],
  "nextCursor": null
} }
```

> 출처: [App Server - List models](https://developers.openai.com/codex/app-server/). `thread/start` 공식 예시 모델은 `gpt-5.1-codex`입니다.

---

## 자동완성 설치

```shell
# Bash
codex completion bash >> ~/.bashrc

# Zsh
codex completion zsh > ~/.zsh/completion/_codex
# 또는
echo 'eval "$(codex completion zsh)"' >> ~/.zshrc

# Fish
codex completion fish > ~/.config/fish/completions/codex.fish

# PowerShell
codex completion powershell

# Elvish
codex completion elvish
```

---

## 키보드 단축키

| 단축키 | 동작 |
| --- | --- |
| `Ctrl+C` | 세션 종료 |
| `Ctrl+L` | 화면 지우기 (대화 유지) |
| `Ctrl+O` | 최신 출력 복사 |
| `Ctrl+R` | 프롬프트 기록 검색 |
| `Ctrl+G` | 외부 에디터 열기 (`VISUAL` 또는 `EDITOR`) |
| `Tab` | 작업 중 다음 턴 입력 큐잉 / 파일 자동완성 |
| `Enter` (작업 중) | 현재 턴에 지시 삽입 |
| `Esc` `Esc` | 이전 사용자 메시지 편집 |
| `Up/Down` | 작성 기록 탐색 |
| `Alt+R` | Raw 출력 모드 토글 |
| `@` | 파일 퍼지 검색 열기 |

---

## Codex SDK

프로그래밍 방식으로 로컬 Codex 에이전트를 제어할 때 사용합니다. 공식 SDK 페이지는 세 가지 방법을 제공합니다.

- **TypeScript library** — JavaScript/TypeScript 서버 사이드 애플리케이션에서 Codex를 완전히 제어
- **Using Codex CLI programmatically** — 개별 작업을 Codex에 보낼 때 (`codex exec`)
- **GitHub Action** — GitHub Actions 워크플로에서 Codex를 트리거·제어

**참조**: [developers.openai.com/codex/sdk](https://developers.openai.com/codex/sdk)

> 참고: 공식 SDK 페이지는 위 세 가지 방법만 나열하며, `openai-codex` 같은 Python 라이브러리에 대한 언급이 없습니다. Python 바인딩이 필요하다면 app-server의 JSON-RPC 프로토콜을 직접 사용하세요.

### TypeScript 라이브러리 (`@openai/codex-sdk`)

Node.js 18 이상 필요. 서버 사이드에서 사용합니다.

```shell
npm install @openai/codex-sdk
```

```typescript
import { Codex } from "@openai/codex-sdk";

const codex = new Codex();
const thread = codex.startThread();
const result = await thread.run(
  "Make a plan to diagnose and fix the CI failures"
);

console.log(result);
```

같은 스레드에서 이어서 실행하거나, 이전 스레드를 재개할 수 있습니다.

```typescript
// 같은 스레드에서 계속
const result = await thread.run("Implement the plan");

// 이전 스레드 재개
const threadId = "<thread-id>";
const thread2 = codex.resumeThread(threadId);
const result2 = await thread2.run("Pick up where you left off");
```

### Codex CLI 프로그래밍 방식 사용 (`codex exec`)

개별 작업만 보내려면 라이브러리 대신 `codex exec`를 사용합니다. 진행 상황은 stderr로 스트리밍되고 최종 에이전트 메시지만 stdout에 출력됩니다.

```shell
codex exec "find any remaining TODOs and create for each TODO a detailed implementation plan markdown file in the .plans/ directory."
```

기본적으로 읽기 전용 샌드박스로 동작합니다. 편집이나 네트워크가 필요하면 플래그를 조합하세요.

```shell
# 파일 편집 허용
codex exec --full-auto "<task>"

# 편집 및 네트워크 명령 허용
codex exec --sandbox danger-full-access "<task>"
```

### 샌드박스 프리셋

| 프리셋 | 설명 |
| --- | --- |
| `read-only` | 파일 읽기만 허용, 쓰기 불가 |
| `workspace-write` | 작업 디렉토리 내에서 읽기/쓰기 허용 |
| `danger-full-access` | 파일시스템 접근 제한 없음 |

> `sandbox` 생략 시 app-server 기본 설정을 따릅니다. 출처: [SDK](https://developers.openai.com/codex/sdk/), [CLI reference](https://developers.openai.com/codex/cli/reference/).

---

## GitHub Action (`openai/codex-action@v1`)

CI/CD 파이프라인에서 Codex를 실행합니다.

**참조**: [developers.openai.com/codex/github-action](https://developers.openai.com/codex/github-action)

### 기본 워크플로 예시

```yaml
name: Codex pull request review
on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  codex:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    outputs:
      final_message: ${{ steps.run_codex.outputs.final-message }}
    steps:
      - uses: actions/checkout@v5
        with:
          ref: refs/pull/${{ github.event.pull_request.number }}/merge

      - name: Pre-fetch base and head refs
        run: |
          git fetch --no-tags origin \
            ${{ github.event.pull_request.base.ref }} \
            +refs/pull/${{ github.event.pull_request.number }}/head

      - name: Run Codex
        id: run_codex
        uses: openai/codex-action@v1
        with:
          openai-api-key: ${{ secrets.OPENAI_API_KEY }}
          prompt-file: .github/codex/prompts/review.md
          output-file: codex-output.md
          safety-strategy: drop-sudo
          sandbox: workspace-write
```

### 주요 입력값

| 입력값 | 설명 |
| --- | --- |
| `prompt` 또는 `prompt-file` | 인라인 지시 또는 프롬프트 파일 경로 (하나만 지정) |
| `codex-args` | 추가 CLI 플래그 (JSON 배열 또는 쉘 문자열) |
| `model` / `effort` | 모델 및 reasoning effort 설정 |
| `sandbox` | 샌드박스 모드 (`workspace-write`, `read-only`, `danger-full-access`) |
| `output-file` | 최종 메시지 저장 파일 경로 |
| `safety-strategy` | 안전 전략. `drop-sudo`(기본값), `unprivileged-user`, `unsafe`(Windows 필수) |
| `unprivileged-user` + `codex-user` | `safety-strategy: unprivileged-user`와 함께 특정 계정으로 Codex 실행. 저장소 체크아웃 읽기·쓰기 권한 필요 |
| `codex-home` | 공유 Codex 홈 디렉토리 경로. 여러 스텝에서 config·MCP 설정 재사용 |
| `codex-version` | CLI 버전 고정 |
| `allow-users` / `allow-bots` | 워크플로 트리거 허용 계정 |

### 권한 관리

Codex는 제한하지 않으면 GitHub 호스티드 러너에서 상당한 접근 권한을 상속합니다.

| 전략 | 설명 |
| --- | --- |
| `safety-strategy: drop-sudo` (기본값) | Codex 실행 전 `sudo` 제거. 되돌릴 수 없으며 메모리의 시크릿 보호. Windows에서는 `unsafe` 필요 |
| `safety-strategy: unprivileged-user` | `codex-user`와 조합해 특정 계정으로 Codex 실행. 읽기 전용 샌드박스에서 파일 조회만 가능하며 변경·네트워크 불가. 소유권 수정 필요 (`.cache/codex-action/examples/unprivileged-user.yml` 참조) |
| `safety-strategy: unsafe` | Windows 전용. `sudo` 제거 없이 실행. 다중 테넌트 러너에서는 사용 금지 |
| `read-only` (sandbox 전략) | Codex가 파일·네트워크 변경 불가. 단 elevated 권한으로 실행되므로 시크릿 보호만으로는 부족 |
| `sandbox` | Codex 내부에서 파일시스템·네트워크 접근 제한. 작업 완료 가능한 가장 좁은 옵션 선택 |

> `final-message` 출력값으로 후속 스텝에서 Codex 응답을 사용할 수 있습니다. 출처: [GitHub Action - Manage privileges](https://developers.openai.com/codex/github-action/).

---

> **최종 업데이트**: 2026-06-15
> **출처**: [developers.openai.com/codex/cli/features](https://developers.openai.com/codex/cli/features), [developers.openai.com/codex/cli/reference](https://developers.openai.com/codex/cli/reference), [developers.openai.com/codex/cli/slash-commands](https://developers.openai.com/codex/cli/slash-commands), [developers.openai.com/codex/noninteractive](https://developers.openai.com/codex/noninteractive), [developers.openai.com/codex/sdk](https://developers.openai.com/codex/sdk), [developers.openai.com/codex/github-action](https://developers.openai.com/codex/github-action), [developers.openai.com/codex/app-server](https://developers.openai.com/codex/app-server), [developers.openai.com/codex/mcp](https://developers.openai.com/codex/mcp)
