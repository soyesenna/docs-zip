# 11. Claude Code SDK

> **참조**: [Claude Code SDK - Anthropic](https://docs.anthropic.com/en/docs/claude-code/sdk)

---

## 목차

- [SDK 개요](#sdk-개요)
- [인증](#인증)
- [CLI 사용법](#cli-사용법)
- [TypeScript SDK](#typescript-sdk)
- [Python SDK](#python-sdk)
- [고급 사용법](#고급-사용법)
- [CLI 옵션 전체 표](#cli-옵션-전체-표)
- [출력 형식 상세](#출력-형식-상세)
- [입력 형식](#입력-형식)
- [모범 사례](#모범-사례)

---

## SDK 개요

Claude Code SDK는 Claude Code를 **서브프로세스**로 실행하여, Claude의 기능을 활용하는 AI 기반 코딩 어시스턴트와 도구를 구축할 수 있게 합니다.

### 지원 환경

| 환경 | 패키지 | 설명 |
|------|--------|------|
| **CLI** | 내장 | 명령줄에서 직접 사용 |
| **TypeScript** | `@anthropic-ai/claude-code` | NPM 패키지 |
| **Python** | `claude-code-sdk` | PyPI 패키지 |

---

## 인증

Claude Code SDK는 여러 인증 방법을 지원합니다.

### Anthropic API 키

전용 API 키 생성을 권장합니다.

**1단계**: [Anthropic Console](https://console.anthropic.com/)에서 API 키 생성

**2단계**: 환경 변수 설정

```bash
export ANTHROPIC_API_KEY="your-api-key"
```

> **보안 팁**: API 키는 GitHub Secret 등을 사용하여 안전하게 보관하세요.

### Amazon Bedrock

```bash
export CLAUDE_CODE_USE_BEDROCK=1
# AWS 자격 증명 구성 필요
```

### Google Vertex AI

```bash
export CLAUDE_CODE_USE_VERTEX=1
# Google Cloud 자격 증명 구성 필요
```

---

## CLI 사용법

### 기본 실행

```bash
# 비대화형 모드로 실행
claude -p "explain this codebase"

# JSON 형식 출력
claude -p --output-format json "list all TODO comments"

# 스트리밍 JSON 출력
claude -p --output-format stream-json "analyze the architecture"
```

### 세션 관리

```bash
# 세션 ID로 대화 재개
claude --resume abc123

# 가장 최근 대화 계속
claude --continue
```

---

## TypeScript SDK

### 설치

```bash
npm install @anthropic-ai/claude-code
```

### 기본 사용법

```typescript
import { claude } from '@anthropic-ai/claude-code';

const result = await claude({
  prompt: 'explain this codebase',
  options: {
    maxTurns: 3,
  },
});

console.log(result);
```

### 옵션 표

| 인자 | 설명 | 기본값 |
|------|------|--------|
| `abortController` | 중단 컨트롤러 | `new AbortController()` |
| `cwd` | 현재 작업 디렉토리 | `process.cwd()` |
| `executable` | JavaScript 런타임 | Node.js에서는 `node`, Bun에서는 `bun` |
| `executableArgs` | 실행 파일에 전달할 인자 | `[]` |
| `pathToClaudeCodeExecutable` | Claude Code 실행 파일 경로 | 패키지에 포함된 실행 파일 |

### 전체 옵션 예시

```typescript
import { claude } from '@anthropic-ai/claude-code';

const result = await claude({
  prompt: 'fix the failing tests',
  options: {
    maxTurns: 5,
    systemPrompt: 'You are a test fixing expert.',
    allowedTools: ['Bash', 'Read', 'Edit'],
    outputFormat: 'json',
    cwd: '/path/to/project',
  },
});
```

---

## Python SDK

### 전제조건

| 요구사항 | 버전 |
|----------|------|
| **Python** | 3.10+ |
| **Node.js** | 설치 필요 |
| **Claude Code CLI** | `npm install -g @anthropic-ai/claude-code` |

### 설치

```bash
pip install claude-code-sdk
```

### 기본 사용법

```python
from claude_code_sdk import ClaudeCode

client = ClaudeCode()

result = client.query("explain this codebase")
print(result)
```

### 옵션 사용

Python SDK는 `ClaudeCodeOptions` 클래스를 통해 CLI의 모든 인자를 지원합니다.

```python
from claude_code_sdk import ClaudeCode, ClaudeCodeOptions

options = ClaudeCodeOptions(
    max_turns=3,
    system_prompt="You are a Python expert.",
    output_format="json",
)

client = ClaudeCode(options=options)
result = client.query("analyze the code for bugs")
```

---

## 고급 사용법

### 멀티턴 대화

세션을 재개하거나 가장 최근 세션에서 계속하여 멀티턴 대화를 구현할 수 있습니다.

```bash
# 세션 ID로 재개
claude --resume <session-id>

# 최근 세션 계속
claude --continue
```

### 커스텀 시스템 프롬프트

Claude의 동작을 가이드하는 커스텀 시스템 프롬프트를 제공할 수 있습니다.

```bash
# 시스템 프롬프트 완전 교체
claude -p --system-prompt "You are a security auditor. Focus only on security issues." "review this code"

# 기본 시스템 프롬프트에 추가
claude -p --append-system-prompt "Always respond in Korean." "explain this function"
```

### MCP 설정

Model Context Protocol (MCP)을 사용하여 Claude Code를 외부 서버의 추가 도구와 리소스로 확장할 수 있습니다.

**1단계**: MCP 서버 JSON 설정 파일 생성

```json
{
  "mcpServers": {
    "database": {
      "command": "node",
      "args": ["./mcp-db-server.js"]
    },
    "api": {
      "command": "python",
      "args": ["./mcp-api-server.py"]
    }
  }
}
```

**2단계**: Claude Code에서 MCP 설정 로드

```bash
claude --mcp-config servers.json
```

### 권한 프롬프트 도구

`--permission-prompt-tool`을 사용하여 MCP 도구로 권한 확인을 커스터마이즈할 수 있습니다.

#### 작동 방식

1. 모델이 도구를 호출하면 먼저 `settings.json`, `--allowedTools`, `--disallowedTools`를 확인
2. 허용/거부가 결정되면 해당 결정에 따라 진행
3. 결정되지 않은 경우 `--permission-prompt-tool`에 지정된 MCP 도구 호출

#### 반환 값

권한 프롬프트 도구는 JSON 문자열 형태의 결과를 반환해야 합니다.

```json
// 승인
{ "behavior": "allow", "updatedInput": {...} }

// 거부
{ "behavior": "deny", "message": "User denied this action" }
}
```

#### TypeScript 구현 예시

```typescript
// MCP 권한 프롬프트 도구 구현
const permissionHandler = async (toolName: string, input: any) => {
  // 사용자에게 권한 요청 로직 구현
  const approved = await askUserPermission(toolName, input);

  if (approved) {
    return JSON.stringify({
      behavior: "allow",
      updatedInput: input, // 입력이 수정되지 않은 경우 원본 반환
    });
  }

  return JSON.stringify({
    behavior: "deny",
    message: "User denied permission",
  });
};
```

#### 사용법

```bash
# MCP 서버 추가 후 권한 프롬프트 도구 지정
claude -p --mcp-config servers.json --permission-prompt-tool mcp__auth__prompt
```

> **참고**: `updatedInput`은 권한 프롬프트가 입력을 수정한 경우에만 변경된 입력을 반환합니다. 수정되지 않은 경우 원본 `input`을 반환하세요.

---

## CLI 옵션 전체 표

| 플래그 | 설명 | 예시 |
|--------|------|------|
| `--print`, `-p` | 비대화형 모드 실행 | `claude -p "query"` |
| `--output-format` | 출력 형식 지정 (`text`, `json`, `stream-json`) | `claude -p --output-format json` |
| `--resume`, `-r` | 세션 ID로 대화 재개 | `claude --resume abc123` |
| `--continue`, `-c` | 가장 최근 대화 계속 | `claude --continue` |
| `--verbose` | 상세 로깅 활성화 | `claude --verbose` |
| `--max-turns` | 비대화형 모드에서 에이전트 턴 수 제한 | `claude --max-turns 3` |
| `--system-prompt` | 시스템 프롬프트 재정의 (`--print` 전용) | `claude --system-prompt "Custom"` |
| `--append-system-prompt` | 시스템 프롬프트에 추가 (`--print` 전용) | `claude --append-system-prompt "Extra"` |
| `--allowedTools` | 허용할 도구 목록 | `claude --allowedTools "Bash(npm install),mcp__filesystem"` |
| `--disallowedTools` | 거부할 도구 목록 | `claude --disallowedTools "Bash(git commit),mcp__github"` |
| `--mcp-config` | MCP 서버 JSON 파일 로드 | `claude --mcp-config servers.json` |
| `--permission-prompt-tool` | 권한 프롬프트용 MCP 도구 (`--print` 전용) | `claude --permission-prompt-tool mcp__auth__prompt` |

### 도구 문자열 형식

`--allowedTools`와 `--disallowedTools`는 공백으로 구분된 목록 또는 쉼표로 구분된 문자열을 지원합니다.

```bash
# 공백으로 구분
claude --allowedTools mcp__slack mcp__filesystem

# 쉼표로 구분된 문자열
claude --allowedTools "Bash(npm install),mcp__filesystem"
```

괄호 안의 인자는 특정 명령어만 허용/거부할 때 사용합니다.

---

## 출력 형식 상세

### Text 출력 (기본값)

응답 텍스트만 반환합니다.

```bash
claude -p "explain this function"
```

```
이 함수는 사용자 입력을 검증하고 데이터베이스에 저장합니다...
```

### JSON 출력

메타데이터를 포함한 구조화된 데이터를 반환합니다.

```bash
claude -p --output-format json "list all files"
```

```json
{
  "type": "result",
  "subtype": "success",
  "cost_usd": 0.003,
  "is_error": false,
  "duration_ms": 1500,
  "duration_api_ms": 1200,
  "num_turns": 2,
  "result": "다음 파일들이 발견되었습니다: ...",
  "session_id": "abc123"
}
```

### 스트리밍 JSON 출력

메시지가 수신될 때마다 스트리밍됩니다.

```bash
claude -p --output-format stream-json "analyze the code"
```

각 대화는 초기 `init` 시스템 메시지로 시작하고, 사용자 및 어시스턴트 메시지 목록이 이어지며, 통계가 포함된 최종 `result` 시스템 메시지로 끝납니다. 각 메시지는 별도의 JSON 객체로 출력됩니다.

```jsonl
{"type":"system","subtype":"init","session_id":"abc123","tools":[...]}
{"type":"user","message":{"role":"user","content":[...]}}
{"type":"assistant","message":{"role":"assistant","content":[...]}}
{"type":"system","subtype":"result","cost_usd":0.003,"duration_ms":1500,"num_turns":2}
```

---

## 입력 형식

### 텍스트 입력 (기본값)

인자로 텍스트를 직접 제공합니다.

```bash
claude -p "explain this function"
```

또는 stdin 파이프를 통해 제공합니다.

```bash
cat main.py | claude -p "explain this code"
```

### 스트리밍 JSON 입력

`stdin`을 통해 메시지 스트림을 제공합니다. 각 메시지는 사용자 턴을 나타냅니다. 이를 통해 `claude` 바이너리를 재실행하지 않고도 여러 턴의 대화를 진행할 수 있습니다.

- 각 메시지는 출력 메시지 스키마와 동일한 형식의 JSON 'User message' 객체
- JSONL 형식 (각 줄이 완전한 JSON 객체)
- `-p` 및 `--output-format stream-json` 필요

> **현재 제한사항**: 텍스트 전용 사용자 메시지만 지원됩니다.

---

## 모범 사례

### 1. JSON 출력 형식 사용

프로그래밍 방식으로 응답을 파싱하려면 JSON 출력 형식을 사용하세요.

```bash
claude -p --output-format json "list all files"
```

### 2. 에러 우아하게 처리

종료 코드와 stderr를 확인하세요.

```bash
result=$(claude -p --output-format json "query" 2>&1)
if [ $? -ne 0 ]; then
  echo "Error: $result"
  exit 1
fi
```

### 3. 세션 관리 사용

멀티턴 대화에서 컨텍스트를 유지하려면 세션 관리를 사용하세요.

```bash
# 첫 번째 턴
claude -p --output-format json "analyze this code" > session.json
SESSION_ID=$(jq -r '.session_id' session.json)

# 다음 턴
claude --resume $SESSION_ID -p "now fix the issues"
```

### 4. 타임아웃 고려

장시간 실행되는 작업에 대한 타임아웃을 설정하세요.

```typescript
const controller = new AbortController();
setTimeout(() => controller.abort(), 30000); // 30초 타임아웃

const result = await claude({
  prompt: 'long running task',
  options: {
    abortController: controller,
  },
});
```

### 5. 속도 제한 준수

여러 요청을 보낼 때는 호출 간에 지연을 추가하여 속도 제한을 준수하세요.

---

## 요약

Claude Code SDK는 CLI, TypeScript, Python 환경에서 Claude Code를 서브프로세스로 실행할 수 있게 해줍니다. 다양한 인증 방법, 유연한 입출력 형식, MCP 확장, 커스텀 권한 관리 등을 통해 강력한 AI 코딩 도구를 구축할 수 있습니다.
