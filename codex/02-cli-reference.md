# Codex CLI - CLI 명령어 및 플래그

> CLI 기본 사용법, 플래그, 슬래시 명령어, 비대화형 모드, MCP/플러그인 CLI 명령어

**참조**: [developers.openai.com/codex/cli/features](https://developers.openai.com/codex/cli/features) | [developers.openai.com/codex/cli/slash-commands](https://developers.openai.com/codex/cli/slash-commands)

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
```

### 세션 관리

```shell
# 세션 재개
codex resume                # 세션 선택 피커 열기
codex resume --all          # 모든 세션 표시
codex resume --last         # 가장 최근 세션 재개
codex resume <SESSION_ID>   # 특정 세션 재개

# 세션 포크
codex fork                  # 세션 선택 후 포크
codex fork --last           # 가장 최근 세션 포크
```

---

## CLI 플래그

### 주요 플래그

| 플래그 | 단축 | 설명 | 예시 |
| --- | --- | --- | --- |
| `--model` | `-m` | 사용할 모델 지정 | `codex -m gpt-5.5` |
| `--ask-for-approval` | `-a` | 승인 정책 설정 | `codex -a on-request` |
| `--sandbox` | `-s` | 샌드박스 모드 설정 | `codex -s workspace-write` |
| `--full-auto` | - | 자동 실행 편의 플래그 (`-a on-request`, `--sandbox workspace-write`) | `codex --full-auto` |
| `--cd` | `-C` | 작업 디렉토리 설정 | `codex --cd ./my-project` |
| `--image` | `-i` | 이미지 첨부 (쉼표로 구분) | `codex -i screenshot.png "설명해줘"` |
| `--profile` | `-p` | 설정 프로필 선택 | `codex -p work` |
| `--add-dir` | - | 추가 쓰기 가능 디렉토리 | `codex --add-dir ../shared` |
| `--search` | - | 라이브 웹 검색 활성화 | `codex --search "최신 API 변경사항"` |
| `--oss` | - | 로컬 오픈소스 모델 제공자 사용 | `codex --oss` |
| `--local-provider` | - | 로컬 제공자 지정 (lmstudio, ollama) | `codex --oss --local-provider ollama` |

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
| `on-request` | 샌드박스 외부 작업에만 승인 요청 |
| `never` | 승인 없이 자동 실행 |

### 위험 플래그

| 플래그 | 설명 |
| --- | --- |
| `--dangerously-bypass-approvals-and-sandbox` (`--yolo`) | 모든 승인 및 샌드박스 우회. **매우 위험**. 외부에서 샌드박스된 환경에서만 사용 |

> `--yolo`는 `--ask-for-approval` 및 `--full-auto`와 함께 사용할 수 없습니다.

### 기타 플래그

| 플래그 | 설명 |
| --- | --- |
| `--no-alt-screen` | 대체 화면 모드 비활성화. 터미널 스크롤백 보존 (Zellij 등에 유용) |
| `--remote` | 원격 TUI 서버에 연결 (`ws://`, `wss://`, `unix://`) |
| `--version` | 버전 정보 표시 |
| `--help` | 도움말 표시 |

---

## 승인 모드 (Approval Modes)

세션 중 `/permissions` 명령으로 전환할 수 있습니다.

| 모드 | 설명 |
| --- | --- |
| **Auto** (기본값) | 작업 디렉토리 내에서 파일 읽기/수정/명령 실행. 외부 접근 시 승인 요청 |
| **Read-only** | 파일 탐색만 가능. 변경이나 명령 실행은 계획 승인 후에만 가능 |
| **Full Access** | 네트워크 포함 전체 시스템 접근. 신뢰할 수 있는 리포지토리와 작업에만 사용 |

---

## 슬래시 명령어 목록

대화형 세션에서 `/`를 입력하면 명령어 목록이 나타납니다. 작업 실행 중에 `Tab`을 누르면 다음 턴에 실행할 명령어를 큐에 추가할 수 있습니다.

### 세션 관리

| 명령어 | 설명 | 작업 중 사용 |
| --- | --- | --- |
| `/new` | 같은 CLI 세션에서 새 대화 시작 | 아니요 |
| `/clear` | 터미널 지우고 새 대화 시작 | 아니요 |
| `/resume` | 저장된 세션 목록에서 대화 재개 | 아니요 |
| `/fork` | 현재 대화를 새 스레드로 포크 | 아니요 |
| `/side`, `/btw` | 임시 사이드 대화 시작 | 아니요 |
| `/rename` | 현재 스레드 이름 변경 | 가능 |
| `/quit`, `/exit` | CLI 종료 | 가능 |

### 모델 및 설정

| 명령어 | 설명 | 작업 중 사용 |
| --- | --- | --- |
| `/model` | 모델 및 reasoning effort 선택 | 아니요 |
| `/fast` | Fast 서비스 티어 토글 | 아니요 |
| `/personality` | 응답 스타일 선택 (friendly, pragmatic, none) | 아니요 |
| `/permissions` | 승인 모드 설정 | 아니요 |
| `/approve` | 자동 리뷰 거부 건 재시도 승인 | 아니요 |
| `/experimental` | 실험적 기능 토글 | 아니요 |
| `/plan` | 플랜 모드 전환 (선택적 프롬프트 포함) | 아니요 |
| `/goal` | 작업 목표 설정/조회/일시정지/해제 | 가능 |

### 코드 리뷰 및 Diff

| 명령어 | 설명 | 작업 중 사용 |
| --- | --- | --- |
| `/review` | 워킹 트리 리뷰 실행 | 아니요 |
| `/diff` | Git diff 표시 (미추적 파일 포함) | 가능 |

### 컨텍스트 및 파일

| 명령어 | 설명 | 작업 중 사용 |
| --- | --- | --- |
| `/compact` | 대화 요약하여 토큰 확보 | 아니요 |
| `/mention` | 파일을 대화에 첨부 | 가능 |
| `/ide` | IDE 컨텍스트 포함 | 가능 |
| `/copy` | 최신 완료된 출력 복사 (`Ctrl+O`와 동일) | 가능 |

### 도구 및 확장

| 명령어 | 설명 | 작업 중 사용 |
| --- | --- | --- |
| `/skills` | 스킬 탐색 및 사용 | 가능 |
| `/mcp` | 설정된 MCP 도구 목록 표시 (`verbose` 옵션) | 가능 |
| `/apps` | 앱(커넥터) 탐색 및 프롬프트에 삽입 | 가능 |
| `/plugins` | 설치된 플러그인 탐색 및 관리 | 가능 |
| `/hooks` | 라이프사이클 훅 검토 및 관리 | 가능 |

### 에이전트 및 백그라운드

| 명령어 | 설명 | 작업 중 사용 |
| --- | --- | --- |
| `/agent` | 활성 에이전트 스레드 전환 | 가능 |
| `/ps` | 백그라운드 터미널 목록 및 상태 | 가능 |
| `/stop` (`/clean`) | 모든 백그라운드 터미널 중지 | 가능 |

### UI 설정

| 명령어 | 설명 | 작업 중 사용 |
| --- | --- | --- |
| `/status` | 세션 설정 및 토큰 사용량 표시 | 가능 |
| `/debug-config` | 설정 레이어 및 요구사항 진단 | 가능 |
| `/statusline` | 상태 표시줄 항목 구성 | 아니요 |
| `/title` | 터미널 제목 항목 구성 | 가능 |
| `/theme` | 구문 강조 테마 선택 | 가능 |
| `/keymap` | 키보드 단축키 리매핑 | 가능 |
| `/vim` | Vim 모드 토글 | 가능 |
| `/raw` | Raw 스크롤백 모드 토글 | 가능 |

### 기타

| 명령어 | 설명 | 작업 중 사용 |
| --- | --- | --- |
| `/init` | `AGENTS.md` 스캐폴드 생성 | 아니요 |
| `/memories` | 메모리 사용 및 생성 설정 | 가능 |
| `/feedback` | 로그 및 진단 정보를 유지보수자에게 전송 | 가능 |
| `/logout` | Codex에서 로그아웃 | 가능 |

---

## 비대화형 모드 상세

### `codex exec`

CI/CD, 스크립트, 자동화에 적합한 모드입니다.

```shell
# 기본 실행
codex exec "fix the CI failure"

# JSON 출력 (프로그램에서 파싱)
codex exec --json "find bugs"

# 세션 재개
codex exec resume --last "continue fixing"
codex exec resume <SESSION_ID> "implement the plan"
```

### 단일 프롬프트 모드

대화형 UI 없이 빠르게 응답을 얻을 수 있습니다:

```shell
codex "이 코드베이스를 설명해줘"
```

---

## MCP CLI 명령어

### 서버 관리

```shell
# MCP 도구 목록 확인
codex mcp list

# MCP 서버 추가
codex mcp add <server-name>

# MCP 인증 로그인
codex mcp login <server-name>
```

### 세션 내 명령어

```
/mcp                  # 설정된 MCP 도구 목록
/mcp verbose          # 서버 진단 정보 포함
```

---

## 플러그인 CLI 명령어

### 마켓플레이스 관리

```shell
# 설치 가능한 플러그인 목록
codex plugin marketplace list

# 플러그인 설치
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

```shell
# 사용 가능한 기능 목록
codex features list

# 기능 활성화
codex features enable unified_exec

# 기능 비활성화
codex features disable shell_snapshot
```

> `codex features enable/disable`은 `$CODEX_HOME/config.toml`에 기록됩니다.

---

## Cloud 명령어

```shell
# Cloud 태스크 대화형 피커
codex cloud

# Cloud 태스크 직접 실행
codex cloud exec --env ENV_ID "버그 요약"

# Best-of-N 실행 (1-4회 시도)
codex cloud exec --env ENV_ID --attempts 3 "버그 요약"
```

---

## App Server (원격 TUI)

```shell
# 로컬 App Server 시작
codex app-server --listen ws://127.0.0.1:4500

# 원격에서 TUI 연결
codex --remote ws://127.0.0.1:4500

# 인증 토큰 설정
TOKEN_FILE="$HOME/.codex/app-server-token"
openssl rand -base64 32 > "$TOKEN_FILE"
chmod 600 "$TOKEN_FILE"
codex app-server --listen ws://0.0.0.0:4500 \
  --ws-auth capability-token \
  --ws-token-file "$TOKEN_FILE"

# 클라이언트 연결
export CODEX_REMOTE_TOKEN="$(cat "$TOKEN_FILE")"
codex --remote wss://remote-host:4500 \
  --remote-auth-token-env CODEX_REMOTE_TOKEN
```

### WebSocket 인증 모드

| 모드 | 서버 플래그 |
| --- | --- |
| Capability Token | `--ws-auth capability-token --ws-token-file /path` |
| Signed Bearer Token | `--ws-auth signed-bearer-token --ws-shared-secret-file /path` |

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

> **최종 업데이트**: 2026-06-05
> **출처**: [developers.openai.com/codex/cli/features](https://developers.openai.com/codex/cli/features), [developers.openai.com/codex/cli/slash-commands](https://developers.openai.com/codex/cli/slash-commands), [github.com/openai/codex - codex-rs/tui/src/cli.rs](https://github.com/openai/codex/blob/main/codex-rs/tui/src/cli.rs)
