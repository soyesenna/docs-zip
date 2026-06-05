# 05. 슬래시 명령어 커스텀 (Slash Commands)

> **참조**: [https://docs.anthropic.com/en/docs/claude-code/slash-commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)

---

## 1. 커스텀 슬래시 명령어 개요

커스텀 슬래시 명령어는 자주 사용하는 프롬프트를 마크다운 파일로 정의하여 Claude Code에서 실행할 수 있게 해줍니다. 명령어는 스코프(프로젝트/개인)에 따라 구성되며, 디렉토리 구조를 통한 네임스페이싱을 지원합니다.

### 기본 문법

```
/<command-name> [arguments]
```

| 파라미터 | 설명 |
|----------|------|
| `<command-name>` | 마크다운 파일명에서 `.md` 확장자를 제외한 이름 |
| `[arguments]` | 명령어에 전달할 선택적 인자 |

---

## 2. 스코프: 프로젝트 vs 개인

### 프로젝트 명령어

저장소에 저장되어 팀과 공유됩니다. `/help`에 표시될 때 설명 뒤에 "(project)"가 붙습니다.

- **위치**: `.claude/commands/`

```bash
# 프로젝트 명령어 생성
mkdir -p .claude/commands
echo "Analyze this code for performance issues and suggest optimizations:" > .claude/commands/optimize.md
```

### 개인 명령어

모든 프로젝트에서 사용할 수 있는 개인 명령어입니다. `/help`에 표시될 때 설명 뒤에 "(user)"가 붙습니다.

- **위치**: `~/.claude/commands/`

```bash
# 개인 명령어 생성
mkdir -p ~/.claude/commands
echo "Review this code for security vulnerabilities:" > ~/.claude/commands/security-review.md
```

### 네임스페이싱

서브디렉토리를 사용하여 명령어를 구성할 수 있습니다. 서브디렉토리는 명령어 이름에 영향을 주지 않고 설명에만 표시됩니다.

| 파일 경로 | 명령어 | 설명 표시 |
|-----------|--------|-----------|
| `.claude/commands/frontend/component.md` | `/component` | (project:frontend) |
| `~/.claude/commands/component.md` | `/component` | (user) |

> **참고**: 사용자 수준과 프로젝트 수준 간의 이름 충돌은 지원되지 않습니다.

---

## 3. 파일 형식

명령어 파일은 **마크다운 + YAML frontmatter** 형식입니다.

```markdown
---
description: 명령어 설명
allowed-tools: Bash(git add:*), Bash(git status:*)
argument-hint: [message]
model: claude-3-5-haiku-20241022
---

여기에 명령어 프롬프트 내용을 작성합니다.
인자를 사용하려면 $ARGUMENTS 또는 $1, $2 등을 사용하세요.
```

---

## 4. Frontmatter 필드

| 필드 | 용도 | 기본값 |
|------|------|--------|
| `description` | 명령어에 대한 간단한 설명 | 프롬프트의 첫 번째 줄 |
| `allowed-tools` | 명령어가 사용할 수 있는 도구 목록 | 대화의 설정 상속 |
| `argument-hint` | 슬래시 명령어에 예상되는 인자 힌트. 자동완성 시 사용자에게 표시 | 없음 |
| `model` | 특정 모델 문자열 지정 (Models overview 참조) | 대화의 설정 상속 |

### argument-hint 예시

```yaml
argument-hint: add [tagId] | remove [tagId] | list
```

```yaml
argument-hint: [pr-number] [priority] [assignee]
```

---

## 5. 인자 시스템

### 전체 인자: `$ARGUMENTS`

`$ARGUMENTS` 플레이스홀더는 명령어에 전달된 모든 인자를 캡처합니다.

```bash
# 명령어 정의
echo 'Fix issue #$ARGUMENTS following our coding standards' > .claude/commands/fix-issue.md

# 사용
> /fix-issue 123 high-priority
# $ARGUMENTS = "123 high-priority"
```

### 위치 인자: `$1`, `$2`, `$3` ...

셸 스크립트처럼 개별 인자에 접근할 수 있습니다.

```bash
# 명령어 정의
echo 'Review PR #$1 with priority $2 and assign to $3' > .claude/commands/review-pr.md

# 사용
> /review-pr 456 high alice
# $1 = "456", $2 = "high", $3 = "alice"
```

**위치 인자가 유용한 경우**:
- 명령어의 다른 부분에서 개별 인자 접근이 필요할 때
- 누락된 인자에 기본값을 제공할 때
- 특정 매개변수 역할이 있는 구조화된 명령어를 만들 때

---

## 6. Bash 명령어 실행

`!` 접두사와 백틱(backtick) 구문을 사용하여 명령어 실행 전에 bash 명령을 실행할 수 있습니다. 실행 결과는 명령어 컨텍스트에 포함됩니다.

> **중요**: bash 명령을 사용하려면 `allowed-tools`에 `Bash` 도구를 반드시 포함해야 합니다. 허용할 특정 bash 명령을 선택할 수 있습니다.

### 문법

```
!`command`
```

### 실전 예제: 커밋 생성 명령어

```markdown
---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
description: Create a git commit
---

## Context

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10`

## Your task

Based on the above changes, create a single git commit.
```

---

## 7. 파일 참조

`@` 접두사를 사용하여 파일 내용을 명령어에 포함할 수 있습니다.

```markdown
# 특정 파일 참조
Review the implementation in @src/utils/helpers.js

# 여러 파일 비교
Compare @src/old-version.js with @src/new-version.js
```

---

## 8. MCP 슬래시 명령어

MCP 서버는 프롬프트를 슬래시 명령어로 노출할 수 있습니다. 이 명령어들은 연결된 MCP 서버에서 동적으로 검색됩니다.

### 명령어 형식

```
/mcp__<server-name>__<prompt-name> [arguments]
```

### 사용 예시

```bash
# 인자 없이
> /mcp__github__list_prs

# 인자와 함께
> /mcp__github__pr_review 456
> /mcp__jira__create_issue "Bug title" high
```

### MCP 권한 및 와일드카드

MCP 도구에 대한 권한 설정 시 **와일드카드는 지원되지 않습니다**.

| 표기 | 지원 여부 | 설명 |
|------|-----------|------|
| `mcp__github` | 올바름 | github 서버의 모든 도구 승인 |
| `mcp__github__get_issue` | 올바름 | 특정 도구만 승인 |
| `mcp__github__*` | 잘못됨 | 와일드카드 미지원 |

MCP 서버의 모든 도구를 승인하려면 서버 이름만 사용하고, 특정 도구만 승인하려면 각 도구를 개별적으로 나열하세요.

### MCP 연결 관리

`/mcp` 명령어를 사용하여:
- 모든 구성된 MCP 서버 보기
- 연결 상태 확인
- OAuth 지원 서버 인증
- 인증 토큰 삭제
- 각 서버의 사용 가능한 도구 및 프롬프트 보기

---

## 9. 실전 예시

### 예시 1: 코드 리뷰 명령어

```markdown
---
description: Review code for quality and best practices
argument-hint: [file-path]
allowed-tools: Read, Glob, Grep
---

Review the following code thoroughly:

$ARGUMENTS

Check for:
1. Code organization and structure
2. Error handling patterns
3. Security vulnerabilities
4. Performance concerns
5. Test coverage
```

**사용**: `/code-review src/auth/login.ts`

### 예시 2: 테스트 실행 명령어

```markdown
---
description: Run tests for specific files or directories
argument-hint: [file-or-directory]
allowed-tools: Bash(npm test:*), Bash(npx jest:*)
---

Run the tests for: $ARGUMENTS

Current test configuration: !`cat jest.config.js`

Execute the appropriate test command and report:
- Total tests run
- Pass/fail counts
- Any error details
- Coverage summary if available
```

**사용**: `/run-tests src/auth/`

### 예시 3: 커밋 생성 명령어 (위치 인자 활용)

```markdown
---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
argument-hint: [type] [message]
description: Create a git commit with conventional commit format
model: claude-3-5-haiku-20241022
---

Create a git commit with type: $1 and message: $2.

Current git status: !`git status`
Current diff: !`git diff HEAD`

Follow conventional commit format:
- feat: new feature
- fix: bug fix
- docs: documentation changes
- refactor: code refactoring
- test: adding tests
- chore: maintenance tasks
```

**사용**: `/smart-commit feat "사용자 인증 기능 추가"`

### 예시 4: 이슈 해결 명령어

```markdown
---
description: Fix a GitHub issue
argument-hint: [issue-number]
allowed-tools: Bash(gh *), Read, Edit, Write, Grep, Glob
---

Fix GitHub issue #$ARGUMENTS

Issue details: !`gh issue view $ARGUMENTS`

Steps:
1. Read and understand the issue
2. Find relevant code
3. Implement the fix
4. Write or update tests
5. Create a branch and PR
```

**사용**: `/fix-issue 123`

---

## 10. 내장 슬래시 명령어 참조

| 명령어 | 용도 |
|--------|------|
| `/add-dir` | 추가 작업 디렉토리 추가 |
| `/agents` | 커스텀 AI 서브에이전트 관리 |
| `/bug` | 버그 리포트 (대화 내용이 Anthropic에 전송) |
| `/clear` | 대화 기록 삭제 |
| `/compact [instructions]` | 선택적 포커스 명령어와 함께 대화 압축 |
| `/config` | 설정 보기/수정 |
| `/cost` | 토큰 사용량 통계 |
| `/doctor` | Claude Code 설치 상태 확인 |
| `/help` | 사용 도움말 |
| `/init` | CLAUDE.md 가이드로 프로젝트 초기화 |
| `/login` | Anthropic 계정 전환 |
| `/logout` | Anthropic 계정 로그아웃 |
| `/mcp` | MCP 서버 연결 및 OAuth 인증 관리 |
| `/memory` | CLAUDE.md 메모리 파일 편집 |
| `/model` | AI 모델 선택 또는 변경 |
| `/permissions` | 권한 보기 또는 업데이트 |
| `/pr_comments` | 풀 리퀘스트 댓글 보기 |
| `/review` | 코드 리뷰 요청 |
| `/status` | 계정 및 시스템 상태 보기 |
| `/terminal-setup` | Shift+Enter 줄바꿈 키 바인딩 설치 (iTerm2, VSCode) |
| `/vim` | vim 모드 진입 |
