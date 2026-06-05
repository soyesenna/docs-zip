# 06. 훅 시스템 (Hooks)

> **참조**: [https://docs.anthropic.com/en/docs/claude-code/hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)

---

## 1. 훅 개요

Claude Code 훅은 특정 이벤트 발생 시 자동으로 셸 명령을 실행하는 이벤트 핸들러 시스템입니다. 도구 실행 전후, 세션 시작/종료, 알림 발생 등 다양한 이벤트에 대해 커스텀 동작을 정의할 수 있습니다.

> **중요**: Claude Code 훅은 **JSON stdin/stdout** 방식을 사용합니다. 환경변수가 아닌 **stdin으로 JSON 데이터를 받고 stdout으로 JSON 결과를 반환**합니다.

### 설정 파일 위치

훅은 다음 설정 파일에서 구성할 수 있습니다 (우선순위 순):

| 설정 파일 | 범위 |
|-----------|------|
| `~/.claude/settings.json` | 사용자 전체 설정 |
| `.claude/settings.json` | 프로젝트 설정 (버전 관리 공유) |
| `.claude/settings.local.json` | 로컬 프로젝트 설정 (커밋되지 않음) |
| 엔터프라이즈 관리 정책 설정 | 조직 전체 관리 설정 |

---

## 2. 설정 구조

훅은 **matcher**와 **hooks 배열**로 구성됩니다. 각 이벤트 타입 아래에 matcher 그룹을 배치합니다.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 /path/to/validate.py",
            "timeout": 30
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint"
          }
        ]
      }
    ]
  }
}
```

### 구조 필드

| 필드 | 설명 |
|------|------|
| **matcher** | 도구 이름과 매칭할 패턴 (대소문자 구분). `PreToolUse`와 `PostToolUse`에만 적용 |
| **hooks** | 매칭 시 실행할 명령어 배열 |
| **type** | 현재 `"command"`만 지원 |
| **command** | 실행할 bash 명령어. `$CLAUDE_PROJECT_DIR` 환경변수 사용 가능 |
| **timeout** | (선택) 개별 명령어 실행 제한 시간 (초 단위). 기본 60초 |

### matcher가 필요 없는 이벤트

`UserPromptSubmit`, `Notification`, `Stop`, `SubagentStop` 등은 matcher를 사용하지 않으므로 생략할 수 있습니다.

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 $CLAUDE_PROJECT_DIR/scripts/context-loader.py"
          }
        ]
      }
    ]
  }
}
```

---

## 3. Matcher 규칙

matcher는 `PreToolUse`와 `PostToolUse` 이벤트에서 도구 이름을 필터링합니다.

| 패턴 | 설명 | 예시 |
|------|------|------|
| 단순 문자열 | 정확히 일치 | `Write` -> Write 도구만 매치 |
| 정규식 | regex 패턴 | `Edit\|Write` 또는 `Notebook.*` |
| `*` 또는 `""` | 모든 도구 매치 | 생략해도 동일 |

### 일반적인 matcher 값

| matcher | 매치되는 도구 |
|---------|--------------|
| `Task` | 서브에이전트 작업 |
| `Bash` | 셸 명령어 |
| `Glob` | 파일 패턴 매칭 |
| `Grep` | 콘텐츠 검색 |
| `Read` | 파일 읽기 |
| `Edit`, `MultiEdit` | 파일 편집 |
| `Write` | 파일 쓰기 |
| `WebFetch`, `WebSearch` | 웹 작업 |

---

## 4. 훅 이벤트 9종 전체 표

| # | 이벤트 | 설명 | matcher 적용 | 실행 시점 |
|---|--------|------|-------------|-----------|
| 1 | **PreToolUse** | 도구 실행 전 | 도구 이름 매칭 | Claude가 도구 매개변수를 생성한 후, 실행 전 |
| 2 | **PostToolUse** | 도구 실행 후 | 도구 이름 매칭 | 도구가 성공적으로 완료된 직후 |
| 3 | **Notification** | 알림 발생 시 | 없음 | Claude가 권한 요청 또는 60초 이상 입력 대기 시 |
| 4 | **UserPromptSubmit** | 프롬프트 제출 전 | 없음 | 사용자가 프롬프트를 제출하고 Claude가 처리하기 전 |
| 5 | **Stop** | 메인 에이전트 종료 시 | 없음 | 메인 Claude Code 에이전트가 응답을 마친 때. 사용자 인터럽트로 인한 종료는 제외 |
| 6 | **SubagentStop** | 서브에이전트 종료 시 | 없음 | Claude Code 서브에이전트(Task 도구 호출)가 응답을 마친 때 |
| 7 | **PreCompact** | 압축 전 | `manual` 또는 `auto` | Claude Code가 compact 작업을 실행하기 전 |
| 8 | **SessionStart** | 세션 시작/재개 | `startup`, `resume`, `clear`, `compact` | 새 세션 시작 또는 기존 세션 재개 시 |
| 9 | **SessionEnd** | 세션 종료 | 없음 | 세션이 종료될 때. `reason`: `clear`, `logout`, `prompt_input_exit`, `other` |

### 이벤트별 matcher 상세

**PreCompact matcher**:
- `manual` - `/compact`로 호출
- `auto` - 컨텍스트 윈도우가 꽉 차서 자동 압축

**SessionStart matcher**:
- `startup` - 시작 시
- `resume` - `--resume`, `--continue`, `/resume`으로 호출
- `clear` - `/clear`로 호출
- `compact` - 자동 또는 수동 compact로 호출

**SessionEnd reason**:
- `clear` - `/clear` 명령으로 세션 초기화
- `logout` - 사용자 로그아웃
- `prompt_input_exit` - 프롬프트 입력 표시 중 사용자 종료
- `other` - 기타 종료 사유

---

## 5. 훅 입력 (JSON stdin)

훅은 **stdin을 통해 JSON 데이터를 수신**합니다. 이벤트별로 다른 스키마를 가집니다.

### PreToolUse 입력

```json
{
  "session_id": "abc123",
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf /tmp/test"
  }
}
```

### PostToolUse 입력

```json
{
  "session_id": "abc123",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  },
  "tool_response": {
    "stdout": "...",
    "stderr": "",
    "exitCode": 0
  }
}
```

> `tool_input`과 `tool_response`의 정확한 스키마는 도구에 따라 다릅니다.

### Notification 입력

```json
{
  "session_id": "abc123",
  "message": "Claude needs your permission to use Bash"
}
```

### UserPromptSubmit 입력

```json
{
  "session_id": "abc123",
  "prompt": "사용자가 입력한 프롬프트 내용"
}
```

### Stop / SubagentStop 입력

```json
{
  "session_id": "abc123",
  "stop_hook_active": false,
  "transcript": "..."
}
```

> `stop_hook_active`가 `true`이면 이미 stop 훅의 결과로 Claude Code가 계속 실행 중인 것입니다. 무한 루프를 방지하려면 이 값을 확인하세요.

### PreCompact 입력

```json
{
  "session_id": "abc123",
  "trigger": "manual",
  "custom_instructions": "사용자가 /compact에 전달한 명령어"
}
```

- `manual`: `custom_instructions`은 사용자가 `/compact`에 전달한 내용
- `auto`: `custom_instructions`은 빈 문자열

### SessionStart 입력

```json
{
  "session_id": "abc123",
  "trigger": "startup"
}
```

### SessionEnd 입력

```json
{
  "session_id": "abc123",
  "reason": "clear"
}
```

---

## 6. 훅 출력 방식

훅은 두 가지 방식으로 Claude Code에 결과를 반환합니다.

### 방식 1: 종료 코드 (Simple)

| 종료 코드 | 의미 | 동작 |
|-----------|------|------|
| **0** | 성공 | `stdout`이 사용자에게 표시됨 (transcript 모드, Ctrl-R). 단, `UserPromptSubmit`과 `SessionStart`에서는 stdout이 컨텍스트에 추가됨 |
| **2** | 차단 에러 | `stderr`가 Claude에게 전달되어 자동 처리됨 |
| **기타** | 비차단 에러 | `stderr`가 사용자에게 표시되고 실행 계속 |

#### 종료 코드 2 이벤트별 동작

| 이벤트 | 종료 코드 2 동작 |
|--------|-----------------|
| `PreToolUse` | 도구 호출 차단, stderr를 Claude에 표시 |
| `PostToolUse` | stderr를 Claude에 표시 (도구는 이미 실행됨) |
| `Notification` | 해당 없음, stderr를 사용자에게만 표시 |
| `UserPromptSubmit` | 프롬프트 처리 차단, 프롬프트 삭제, stderr를 사용자에게만 표시 |
| `Stop` | 종료 차단, stderr를 Claude에 표시 |
| `SubagentStop` | 종료 차단, stderr를 Claude 서브에이전트에 표시 |
| `PreCompact` | 해당 없음, stderr를 사용자에게만 표시 |
| `SessionStart` | 해당 없음, stderr를 사용자에게만 표시 |
| `SessionEnd` | 해당 없음, stderr를 사용자에게만 표시 |

### 방식 2: JSON 출력 (Advanced)

훅은 `stdout`에 구조화된 JSON을 반환하여 더 정교한 제어가 가능합니다.

#### 공통 JSON 필드

모든 훅 타입에 공통적으로 사용할 수 있는 필드:

```json
{
  "continue": true,
  "stopReason": "사용자에게 표시할 이유 (Claude에게는 표시되지 않음)"
}
```

- `continue`가 `false`이면 훅 실행 후 Claude가 처리를 중지합니다.
- 모든 경우 `"continue": false`가 `"decision": "block"`보다 우선합니다.

#### PreToolUse Decision Control

```json
{
  "decision": "allow",
  "permissionDecisionReason": "이유"
}
```

| decision | 설명 |
|----------|------|
| `"allow"` | 권한 시스템을 우회하여 승인. `permissionDecisionReason`은 사용자에게만 표시 |
| `"deny"` | 도구 호출 실행 차단. `permissionDecisionReason`이 Claude에게 표시 |
| `"ask"` | 사용자에게 확인 요청. `permissionDecisionReason`은 사용자에게만 표시 |

#### PostToolUse Decision Control

```json
{
  "decision": "block",
  "reason": "Claude에게 전달할 피드백"
}
```

| decision | 설명 |
|----------|------|
| `"block"` | Claude에게 `reason`을 자동 프롬프트로 전달 |
| `undefined` | 아무 동작 없음. `reason` 무시 |

추가 필드: `"hookSpecificOutput": { "additionalContext": "..." }`로 Claude에 컨텍스트 추가 가능

#### UserPromptSubmit Decision Control

```json
{
  "decision": "block",
  "reason": "사용자에게만 표시되는 이유"
}
```

| decision | 설명 |
|----------|------|
| `"block"` | 프롬프트 처리 차단. 제출된 프롬프트가 컨텍스트에서 삭제됨. `reason`은 사용자에게만 표시 |
| `undefined` | 프롬프트 정상 처리. `reason` 무시 |

#### Stop / SubagentStop Decision Control

```json
{
  "decision": "block",
  "reason": "Claude가 계속 진행해야 하는 이유"
}
```

| decision | 설명 |
|----------|------|
| `"block"` | Claude 종료 방지. Claude가 진행 방법을 알 수 있도록 `reason` 필수 |
| `undefined` | Claude가 정상 종료. `reason` 무시 |

#### SessionStart Decision Control

```json
{
  "hookSpecificOutput": {
    "additionalContext": "세션 시작 시 로드할 컨텍스트"
  }
}
```

- `additionalContext`가 문자열로 컨텍스트에 추가됩니다.
- 여러 훅의 `additionalContext` 값은 연결(concatenate)됩니다.

#### SessionEnd Decision Control

- 세션 종료를 차단할 수 없습니다.
- 정리(cleanup) 작업만 수행 가능합니다.

---

## 7. MCP 도구와 훅

MCP 서버가 제공하는 도구는 특수한 네이밍 패턴을 가지며, 훅에서 매칭할 수 있습니다.

### MCP 도구 네이밍

```
mcp__<server>__<tool>
```

예시:
- `mcp__memory__create_entities` - Memory 서버의 엔티티 생성 도구
- `mcp__filesystem__read_file` - Filesystem 서버의 파일 읽기 도구
- `mcp__github__search_repositories` - GitHub 서버의 검색 도구

### MCP 도구에 대한 훅 설정

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__filesystem__write_file",
        "hooks": [
          {
            "type": "command",
            "command": "python3 $CLAUDE_PROJECT_DIR/scripts/validate-file-write.py"
          }
        ]
      }
    ]
  }
}
```

특정 MCP 도구 또는 전체 MCP 서버를 타겟으로 할 수 있습니다.

---

## 8. 훅 실행 세부사항

| 항목 | 설명 |
|------|------|
| **타임아웃** | 기본 60초. 명령어별로 `timeout` 필드로 설정 가능. 개별 명령의 타임아웃은 다른 명령에 영향 없음 |
| **병렬 실행** | 매칭되는 모든 훅이 **병렬로** 실행됨 |
| **중복 제거** | 동일한 훅 명령어가 여러 개 있으면 자동으로 중복 제거됨 |
| **환경** | Claude Code의 현재 디렉토리와 환경에서 실행. `CLAUDE_PROJECT_DIR` 환경변수 사용 가능 (프로젝트 루트의 절대 경로) |
| **입력** | JSON 데이터를 **stdin**으로 수신 |
| **출력** | 이벤트에 따라 다름 (아래 표 참조) |

### 이벤트별 출력 처리

| 이벤트 | 출력 처리 |
|--------|-----------|
| PreToolUse / PostToolUse / Stop / SubagentStop | transcript (Ctrl-R)에 진행 상황 표시 |
| Notification / SessionEnd | 디버그 로그에만 기록 (`--debug`) |
| UserPromptSubmit / SessionStart | stdout이 Claude의 컨텍스트에 추가됨 |

---

## 9. 설정 안전

설정 파일의 훅 직접 수정은 즉시 적용되지 않습니다. Claude Code는:

1. **시작 시 스냅샷 캡처**: 세션 시작 시 훅 설정을 스냅샷으로 저장
2. **세션 중 스냅샷 사용**: 세션 전체에서 이 스냅샷을 사용
3. **외부 변경 감지 시 경고**: 훅이 외부에서 수정되면 경고 표시
4. **변경 적용 시 확인 필요**: `/hooks` 메뉴에서 변경 검토 후 적용 필요

이를 통해 악의적인 훅 수정이 현재 세션에 영향을 주는 것을 방지합니다.

---

## 10. 실전 예제

### 예제 1: Bash 명령어 검증 (종료 코드 방식)

Bash 명령어가 실행되기 전에 위험한 명령을 차단합니다.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json,sys; d=json.load(sys.stdin); cmd=d.get('tool_input',{}).get('command',''); sys.exit(2 if 'rm -rf' in cmd else 0)\"",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### 예제 2: 세션 시작 시 컨텍스트 로드 (JSON 출력)

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json; print(json.dumps({'hookSpecificOutput': {'additionalContext': '현재 미해결 이슈:\\n' + open('/tmp/issues.txt').read()}}))\"",
            "timeout": 15
          }
        ]
      }
    ]
  }
}
```

### 예제 3: 파일 편집 후 린트 실행

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "cd $CLAUDE_PROJECT_DIR && npx eslint --fix $(git diff --name-only HEAD 2>/dev/null | grep '\\.ts$' | head -5) 2>&1 || true",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### 예제 4: 사용자 프롬프트 검증 및 컨텍스트 추가 (JSON 출력)

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 $CLAUDE_PROJECT_DIR/scripts/prompt-handler.py",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

`prompt-handler.py` 예시:

```python
#!/usr/bin/env python3
import json
import sys

data = json.load(sys.stdin)
prompt = data.get("prompt", "")

# 특정 키워드 차단
if "drop table" in prompt.lower():
    print(json.dumps({
        "decision": "block",
        "reason": "SQL 파괴 명령이 감지되었습니다."
    }))
    sys.exit(0)

# 컨텍스트 추가
print(json.dumps({
    "hookSpecificOutput": {
        "additionalContext": "현재 브랜치: main\n최근 커밋: abc1234"
    }
}))
```

### 예제 5: Stop 훅으로 자동 계속 진행

Claude가 작업을 완료한 후 추가 작업이 필요한 경우 계속 진행시킵니다.

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json,sys; d=json.load(sys.stdin); t=d.get('transcript',''); print(json.dumps({'decision':'block','reason':'모든 테스트가 통과했는지 확인하고, 실패한 테스트가 있으면 수정하세요.'})) if 'test' in t.lower() else print(json.dumps({}))\"",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### 예제 6: 프로젝트 전용 스크립트 참조

`CLAUDE_PROJECT_DIR` 환경변수를 사용하여 프로젝트 내 스크립트를 참조합니다.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/scripts/validate-write.py",
            "timeout": 15
          }
        ]
      }
    ]
  }
}
```

---

## 11. 디버깅

### 기본 문제 해결

1. **설정 확인**: `/hooks` 실행하여 훅이 등록되어 있는지 확인
2. **문법 검증**: JSON 설정 파일이 유효한지 확인
3. **명령어 수동 테스트**: 훅 명령을 직접 실행하여 정상 동작 확인
4. **권한 확인**: 스크립트가 실행 가능한지 확인 (`chmod +x`)
5. **로그 확인**: `claude --debug`로 훅 실행 상세 보기

### 일반적인 문제

| 문제 | 원인 | 해결 |
|------|------|------|
| 따옴표 이스케이프 | JSON 문자열 내 `"` 이스케이프 누락 | `\\"` 사용 |
| 잘못된 matcher | 도구 이름 대소문자 불일치 | 대소문자 정확히 일치해야 함 |
| 명령어 미발견 | 상대 경로 사용 | 절대 경로 또는 `$CLAUDE_PROJECT_DIR` 사용 |

### 상세 디버깅

`claude --debug`를 사용하면 훅 실행 상세를 볼 수 있습니다.

Transcript 모드 (Ctrl-R)에서 다음 정보가 표시됩니다:
- 실행 중인 훅
- 실행되는 명령어
- 성공/실패 상태
- 출력 또는 에러 메시지

---

## 12. 보안 고려사항

### 면책 조항

훅은 시스템에서 임의의 셸 명령을 자동으로 실행합니다. 훅을 사용하면 다음 사항에 동의하는 것입니다:

- 구성한 명령에 대한 책임은 본인에게 있습니다
- 훅은 사용자 계정이 접근할 수 있는 모든 파일을 수정, 삭제 또는 접근할 수 있습니다
- 악의적이거나 잘못 작성된 훅은 데이터 손실이나 시스템 손상을 일으킬 수 있습니다
- 프로덕션 사용 전 안전한 환경에서 철저히 테스트하세요

### 보안 모범 사례

1. **입력 검증 및 정제**: 입력 데이터를 무조건 신뢰하지 마세요
2. **셸 변수 인용**: `$VAR` 대신 `"$VAR"` 사용
3. **경로 순회 차단**: 파일 경로에 `..`가 있는지 확인
4. **절대 경로 사용**: 스크립트에 전체 경로 지정. `$CLAUDE_PROJECT_DIR` 활용
5. **민감 파일 건너뛰기**: `.env`, `.git/`, 키 파일 등 피하기
