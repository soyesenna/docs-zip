# Claude Code CLI 명령어 및 플래그

> CLI 명령어, 플래그, 슬래시 명령어, 단축키, Vim 모드, 내장 도구 전체 참조

**참고**: [CLI Reference - Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/cli-reference) | [Interactive Mode - Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/interactive-mode) | [Slash Commands - Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/slash-commands) | [SDK - Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/sdk)

---

## 대화형 모드 vs 원샷 모드

| 모드 | 설명 | 명령어 |
|------|------|--------|
| **대화형 모드** | 터미널에서 대화형 REPL 세션 시작 | `claude` |
| **원샷 모드** | 쿼리를 실행하고 결과를 출력한 후 종료 | `claude -p "쿼리"` |

---

## CLI 명령어

| 명령어 | 설명 | 예시 |
|--------|------|------|
| `claude` | 대화형 REPL 시작 | `claude` |
| `claude "쿼리"` | 초기 프롬프트와 함께 REPL 시작 | `claude "이 프로젝트 설명해줘"` |
| `claude -p "쿼리"` | SDK를 통해 쿼리 후 종료 | `claude -p "이 함수 설명해줘"` |
| `cat file \| claude -p "쿼리"` | 파이프된 내용 처리 | `cat logs.txt \| claude -p "설명해줘"` |
| `claude -c` | 가장 최근 대화 이어서 | `claude -c` |
| `claude -c -p "쿼리"` | SDK로 대화 이어서 | `claude -c -p "타입 에러 확인해줘"` |
| `claude -r "<session-id>" "쿼리"` | 세션 ID로 특정 세션 재개 | `claude -r "abc123" "이 PR 마저 끝내줘"` |
| `claude config` | 설정 관리 | `claude config list` |
| `claude update` | 최신 버전으로 업데이트 | `claude update` |
| `claude mcp` | MCP 서버 구성 | `claude mcp` |

---

## CLI 플래그

| 플래그 | 설명 | 예시 |
|--------|------|------|
| `--print`, `-p` | 대화형 모드 없이 응답 출력 (SDK 참조) | `claude -p "쿼리"` |
| `--output-format` | 출력 형식 지정 (`text`, `json`, `stream-json`) | `claude -p "쿼리" --output-format json` |
| `--input-format` | 입력 형식 지정 (`text`, `stream-json`) | `claude -p --input-format stream-json` |
| `--verbose` | 상세 로깅 활성화, 전체 턴별 출력 표시 | `claude --verbose` |
| `--max-turns` | 비대화형 모드에서 에이전트 턴 수 제한 | `claude -p --max-turns 3 "쿼리"` |
| `--model` | 세션 모델 설정 (별칭 또는 전체 이름) | `claude --model claude-sonnet-4-20250514` |
| `--add-dir` | Claude가 접근할 추가 작업 디렉토리 | `claude --add-dir ../apps ../lib` |
| `--allowedTools` | 권한 프롬프트 없이 허용할 도구 목록 | `"Bash(git log:*)" "Bash(git diff:*)" "Read"` |
| `--disallowedTools` | 권한 프롬프트 없이 거부할 도구 목록 | `"Bash(git log:*)" "Bash(git diff:*)" "Edit"` |
| `--append-system-prompt` | 시스템 프롬프트에 추가 (`--print` 전용) | `claude --append-system-prompt "커스텀 지시사항"` |
| `--system-prompt` | 시스템 프롬프트 덮어쓰기 (`--print` 전용) | `claude --system-prompt "커스텀 지시사항"` |
| `--permission-mode` | 지정된 권한 모드로 시작 | `claude --permission-mode plan` |
| `--permission-prompt-tool` | 비대화형 모드에서 권한 프롬프트를 처리할 MCP 도구 | `claude -p --permission-prompt-tool mcp_auth_tool "쿼리"` |
| `--resume` | 세션 ID로 특정 세션 재개 | `claude --resume abc123 "쿼리"` |
| `--continue` | 현재 디렉토리의 가장 최근 대화 로드 | `claude --continue` |
| `--mcp-config` | JSON 파일에서 MCP 서버 로드 | `claude --mcp-config servers.json` |
| `--dangerously-skip-permissions` | 권한 프롬프트 건너뛰기 (주의 필요) | `claude --dangerously-skip-permissions` |

---

## Print 모드 상세 (`-p`)

### 출력 형식

| 형식 | 설명 | 사용 예시 |
|------|------|-----------|
| `text` | 응답 텍스트만 반환 (기본값) | `claude -p "쿼리"` |
| `json` | 메타데이터를 포함한 구조화된 데이터 반환 | `claude -p "쿼리" --output-format json` |
| `stream-json` | 메시지를 수신하는 대로 스트리밍 | `claude -p "쿼리" --output-format stream-json` |

### Text 출력 (기본값)

```bash
claude -p "이 함수의 목적을 설명해줘"
```

### JSON 출력

```bash
claude -p "쿼리" --output-format json
```

메타데이터를 포함한 구조화된 응답을 반환합니다.

### Streaming JSON 출력

```bash
claude -p "쿼리" --output-format stream-json
```

각 대화는 초기 `init` 시스템 메시지로 시작하고, 사용자 및 어시스턴트 메시지 목록이 이어지며, 통계가 포함된 최종 `result` 시스템 메시지로 끝납니다.

### Streaming JSON 입력

stdin을 통해 메시지 스트림을 제공할 수 있습니다. 각 메시지는 사용자 턴을 나타냅니다. `-p` 및 `--output-format stream-json`이 필요합니다.

```bash
claude -p --output-format stream-json --input-format stream-json
```

---

## 슬래시 명령어

대화형 세션에서 사용할 수 있는 내장 슬래시 명령어입니다.

| 명령어 | 용도 |
|--------|------|
| `/add-dir` | 추가 작업 디렉토리 연결 |
| `/agents` | 커스텀 AI 서브에이전트 관리 |
| `/bug` | 버그 리포트 (대화 내용을 Anthropic에 전송) |
| `/clear` | 대화 기록 삭제 |
| `/compact [지시사항]` | 선택적 포커스 지시사항으로 대화 압축 |
| `/config` | 설정 보기/수정 |
| `/cost` | 토큰 사용량 통계 표시 |
| `/doctor` | Claude Code 설치 상태 확인 |
| `/help` | 사용 도움말 |
| `/hooks` | 훅 설정 관리 |
| `/ide` | 외부 터미널에서 IDE에 연결 |
| `/init` | CLAUDE.md 가이드로 프로젝트 초기화 |
| `/install-github-app` | GitHub Actions용 GitHub 앱 설치 |
| `/login` | Anthropic 계정 전환 |
| `/logout` | Anthropic 계정 로그아웃 |
| `/mcp` | MCP 서버 연결 및 OAuth 인증 관리 |
| `/memory` | CLAUDE.md 메모리 파일 편집 |
| `/model` | AI 모델 선택 또는 변경 |
| `/permissions` | 권한 보기 또는 업데이트 |
| `/plugin` | 플러그인 관리 (설치, 활성화, 비활성화) |
| `/pr_comments` | 풀 리퀘스트 댓글 보기 |
| `/reload-plugins` | 플러그인 다시 로드 (수정 후 재시작 불필요) |
| `/resume` | 이전 대화 선택하여 재개 |
| `/review` | 코드 리뷰 요청 |
| `/status` | 계정 및 시스템 상태 보기 |
| `/terminal-setup` | Shift+Enter 줄바꿈 키 바인딩 설치 (iTerm2, VSCode만) |
| `/vim` | Vim 모드 진입 (삽입/명령 모드 전환) |

### 커스텀 슬래시 명령어

Markdown 파일로 커스텀 명령어를 정의할 수 있습니다.

#### 프로젝트 명령어 (팀 공유)

```bash
mkdir -p .claude/commands
echo "이 코드의 성능 문제를 분석하고 최적화를 제안해줘:" > .claude/commands/optimize.md
```

#### 개인 명령어 (모든 프로젝트에서 사용 가능)

```bash
mkdir -p ~/.claude/commands
echo "이 코드의 보안 취약점을 검토해줘:" > ~/.claude/commands/security-review.md
```

#### 인자 사용

```markdown
<!-- .claude/commands/fix-issue.md -->
Fix issue #$ARGUMENTS following our coding standards
```

```markdown
<!-- .claude/commands/review-pr.md -->
Review PR #$1 with priority $2 and assign to $3
```

#### 프론트매터

```markdown
---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
argument-hint: [message]
description: Create a git commit
model: claude-3-5-haiku-20241022
---

Create a git commit with message: $ARGUMENTS
```

---

## 특수 단축키

### 일반 제어

| 단축키 | 설명 |
|--------|------|
| `Ctrl+C` | 현재 입력 또는 생성 취소 |
| `Ctrl+D` | Claude Code 세션 종료 |
| `Ctrl+L` | 터미널 화면 지우기 (대화 기록 유지) |
| `Up/Down 화살표` | 명령 기록 탐색 |
| `Esc` + `Esc` | 이전 메시지 편집 |
| `Shift+Tab` | 권한 모드 전환 (Auto-Accept, Plan, 일반) |

### 여러 줄 입력

| 방법 | 단축키 | 컨텍스트 |
|------|--------|----------|
| 빠른 이스케이프 | `\` + `Enter` | 모든 터미널에서 동작 |
| macOS 기본 | `Option+Enter` | macOS 기본 설정 |
| 터미널 설정 | `Shift+Enter` | `/terminal-setup` 실행 후 |
| 제어 시퀀스 | `Ctrl+J` | 줄바꿈 문자 |
| 붙여넣기 모드 | 직접 붙여넣기 | 코드 블록, 로그 등 |

### 빠른 명령

| 단축키 | 설명 |
|--------|------|
| `#` (시작 시) | 메모리 단축키 - CLAUDE.md에 추가 |
| `/` (시작 시) | 슬래시 명령어 |
| `!` (시작 시) | Bash 모드 - 명령을 직접 실행하고 결과를 세션에 추가 |

---

## Vim 모드

`/vim` 명령어로 활성화하거나 `/config`에서 영구적으로 설정합니다.

### 모드 전환

| 명령 | 동작 | 현재 모드 |
|------|------|-----------|
| `Esc` | NORMAL 모드 진입 | INSERT |
| `i` | 커서 앞에 삽입 | NORMAL |
| `I` | 줄 시작에 삽입 | NORMAL |
| `a` | 커서 뒤에 삽입 | NORMAL |
| `A` | 줄 끝에 삽입 | NORMAL |
| `o` | 아래에 새 줄 열기 | NORMAL |
| `O` | 위에 새 줄 열기 | NORMAL |

### 탐색 (NORMAL 모드)

| 명령 | 동작 |
|------|------|
| `h`/`j`/`k`/`l` | 좌/하/상/우 이동 |
| `w` | 다음 단어 |
| `e` | 단어 끝 |
| `b` | 이전 단어 |
| `0` | 줄 시작 |
| `$` | 줄 끝 |
| `^` | 첫 번째 비공백 문자 |
| `gg` | 입력 시작 |
| `G` | 입력 끝 |

### 편집 (NORMAL 모드)

| 명령 | 동작 |
|------|------|
| `x` | 문자 삭제 |
| `dd` | 줄 삭제 |
| `D` | 줄 끝까지 삭제 |
| `dw`/`de`/`db` | 단어/끝까지/뒤로 삭제 |
| `cc` | 줄 변경 |
| `C` | 줄 끝까지 변경 |
| `cw`/`ce`/`cb` | 단어/끝까지/뒤로 변경 |
| `.` | 마지막 변경 반복 |

---

## 내장 도구

Claude Code가 코드베이스를 이해하고 수정하는 데 사용하는 도구 목록입니다.

| 도구 | 설명 | 권한 필요 |
|------|------|-----------|
| **Bash** | 환경에서 쉘 명령 실행 | 예 |
| **Edit** | 특정 파일에 대한 타겟 편집 | 예 |
| **MultiEdit** | 단일 파일에 여러 편집을 원자적으로 수행 | 예 |
| **Write** | 파일 생성 또는 덮어쓰기 | 예 |
| **Read** | 파일 내용 읽기 | 아니요 |
| **Glob** | 패턴 매칭으로 파일 찾기 | 아니요 |
| **Grep** | 파일 내용에서 패턴 검색 | 아니요 |
| **NotebookRead** | Jupyter 노트북 내용 읽기 및 표시 | 아니요 |
| **NotebookEdit** | Jupyter 노트북 셀 수정 | 예 |
| **Task** | 복잡한 다단계 작업을 처리하는 서브 에이전트 실행 | 아니요 |
| **TodoWrite** | 구조화된 작업 목록 생성 및 관리 | 아니요 |
| **WebFetch** | 지정된 URL에서 콘텐츠 가져오기 | 예 |
| **WebSearch** | 도메인 필터링이 있는 웹 검색 | 예 |

> 권한 규칙은 `/allowed-tools` 또는 [권한 설정](03-settings.md)에서 구성할 수 있습니다.
