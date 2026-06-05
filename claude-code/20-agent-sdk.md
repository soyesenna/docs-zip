# Claude Code Agent SDK

> 공식 프로그래밍 SDK — TypeScript/Python으로 Claude Code 에이전트 빌드

**원문**: https://code.claude.com/docs/en/agent-sdk/overview (및 하위 페이지 전체)

---

## 1. 개요

Agent SDK는 Claude Code를 라이브러리로 사용하여 프로덕션급 AI 에이전트를 빌드할 수 있게 해주는 공식 SDK다. Python과 TypeScript를 지원하며, 파일 읽기, 명령어 실행, 코드 편집, 웹 검색 등을 자율적으로 수행하는 에이전트를 프로그래밍 방식으로 제어할 수 있다.

### 핵심 특징

| 특징 | 설명 |
| --- | --- |
| 내장 도구 | Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, Monitor 등 |
| 언어 지원 | TypeScript (`@anthropic-ai/claude-agent-sdk`), Python (`claude-agent-sdk`) |
| 인증 | Anthropic API Key, Amazon Bedrock, Vertex AI, Azure AI Foundry |
| 에이전트 루프 | 자율적 도구 호출 루프 (Claude Code CLI와 동일) |
| 서브에이전트 | 프로그래밍 방식으로 하위 에이전트 정의 및 실행 |
| MCP 통합 | Model Context Protocol 서버 연동 |
| 커스텀 도구 | 인프로세스 MCP 서버로 커스텀 함수 등록 |
| 권한 제어 | 세분화된 권한 모드 및 규칙 |
| 세션 관리 | 세션 생성, 재개, 포크, 외부 스토리지 영속화 |
| 훅 시스템 | 에이전트 라이프사이클 이벤트에 콜백 등록 |

### 다른 Claude 도구와의 비교

| 구분 | Agent SDK | Client SDK | Claude Code CLI | Managed Agents |
| --- | --- | --- | --- | --- |
| 실행 환경 | 사용자 프로세스 | 사용자 프로세스 | 터미널 | Anthropic 인프라 |
| 인터페이스 | Python/TypeScript 라이브러리 | API 직접 호출 | CLI | REST API |
| 도구 실행 | 자동 (내장) | 수동 구현 | 자동 (내장) | Anthropic 관리 |
| 적합한 용도 | 프로덕션 자동화, CI/CD | 저수준 API 제어 | 대화형 개발 | 호스팅 에이전트 |

---

## 2. 빠른 시작

### 전제 조건

- Node.js 18+ (TypeScript) 또는 Python 3.10+ (Python)
- Anthropic 계정 및 API Key

### 설치

| 언어 | 명령어 |
| --- | --- |
| TypeScript | `npm install @anthropic-ai/claude-agent-sdk` |
| Python | `pip install claude-agent-sdk` |

TypeScript SDK는 플랫폼별 네이티브 Claude Code 바이너리를 선택적 의존성으로 번들하므로 Claude Code를 별도로 설치할 필요가 없다.

### API Key 설정

```bash
export ANTHROPIC_API_KEY=your-api-key
```

서드파티 프로바이더 인증:

| 프로바이더 | 환경 변수 |
| --- | --- |
| Amazon Bedrock | `CLAUDE_CODE_USE_BEDROCK=1` + AWS 자격증명 |
| Vertex AI | `CLAUDE_CODE_USE_VERTEX=1` + Google Cloud 자격증명 |
| Azure AI Foundry | `CLAUDE_CODE_USE_FOUNDRY=1` + Azure 자격증명 |
| Claude Platform on AWS | `CLAUDE_CODE_USE_ANTHROPIC_AWS=1` + AWS 자격증명 |

### 첫 번째 에이전트 실행

**Python**

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    async for message in query(
        prompt="What files are in this directory?",
        options=ClaudeAgentOptions(allowed_tools=["Bash", "Glob"]),
    ):
        if hasattr(message, "result"):
            print(message.result)

asyncio.run(main())
```

**TypeScript**

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "What files are in this directory?",
  options: { allowedTools: ["Bash", "Glob"] },
})) {
  if ("result" in message) console.log(message.result);
}
```

### 권한 모드 요약

| 모드 | 동작 | 사용 사례 |
| --- | --- | --- |
| `acceptEdits` | 파일 편집 및 파일시스템 명령 자동 승인 | 신뢰된 개발 워크플로 |
| `dontAsk` | `allowedTools` 외의 모든 요청 거부 | 잠긴 헤드리스 에이전트 |
| `auto` (TS만) | 모델 분류기가 승인/거부 결정 | 안전 가드레일이 있는 자율 에이전트 |
| `bypassPermissions` | 모든 권한 프롬프트 생략 | 샌드박스 CI, 완전 신뢰 환경 |
| `default` | `canUseTool` 콜백으로 승인 처리 | 커스텀 승인 플로우 |
| `plan` | 읽기 전용 도구만 실행 | 코드 수정 없이 분석/계획 |

---

## 3. TypeScript SDK

### 설치

```bash
npm install @anthropic-ai/claude-agent-sdk
```

### 핵심 함수

| 함수 | 설명 |
| --- | --- |
| `query()` | 메인 함수. AsyncGenerator로 메시지를 스트리밍 |
| `startup()` | CLI 서브프로세스를 미리 준비 (pre-warm) |
| `tool()` | 타입 안전 MCP 도구 정의 (Zod 스키마) |
| `createSdkMcpServer()` | 인프로세스 MCP 서버 생성 |
| `listSessions()` | 과거 세션 목록 조회 |
| `getSessionMessages()` | 세션 메시지 읽기 |
| `getSessionInfo()` | 단일 세션 메타데이터 조회 |
| `renameSession()` | 세션 제목 변경 |
| `tagSession()` | 세션 태그 설정 |
| `resolveSettings()` | 설정 병합 결과 조회 |

### `query()` 시그니처

```typescript
function query({
  prompt,
  options
}: {
  prompt: string | AsyncIterable<SDKUserMessage>;
  options?: Options;
}): Query;
```

### `Options` 주요 필드

| 필드 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `allowedTools` | `string[]` | `[]` | 자동 승인할 도구 목록 |
| `disallowedTools` | `string[]` | `[]` | 차단할 도구 목록 |
| `permissionMode` | `PermissionMode` | `'default'` | 권한 모드 |
| `model` | `string` | CLI 기본값 | 사용할 모델 |
| `maxTurns` | `number` | 제한 없음 | 최대 도구 사용 횟수 |
| `maxBudgetUsd` | `number` | 제한 없음 | 최대 비용 (USD) |
| `effort` | `'low'/'medium'/'high'/'xhigh'/'max'` | `'high'` | 추론 노력 수준 |
| `systemPrompt` | `string | {type:'preset', preset:'claude_code'}` | `undefined` | 시스템 프롬프트 |
| `mcpServers` | `Record<string, McpServerConfig>` | `{}` | MCP 서버 설정 |
| `agents` | `Record<string, AgentDefinition>` | `undefined` | 프로그래밍 방식 서브에이전트 |
| `hooks` | `Partial<Record<HookEvent, HookCallbackMatcher[]>>` | `{}` | 훅 콜백 |
| `settingSources` | `SettingSource[]` | CLI 기본값 (전체) | 파일시스템 설정 소스 |
| `cwd` | `string` | `process.cwd()` | 작업 디렉토리 |
| `resume` | `string` | `undefined` | 재개할 세션 ID |
| `forkSession` | `boolean` | `false` | 세션 포크 여부 |
| `sandbox` | `SandboxSettings` | `undefined` | 샌드박스 설정 |
| `plugins` | `SdkPluginConfig[]` | `[]` | 플러그인 설정 |
| `skills` | `string[] | 'all'` | `undefined` | 활성 스킬 |
| `thinking` | `ThinkingConfig` | `{type:'adaptive'}` | 확장 사고 설정 |

### `startup()` — Pre-warm

```typescript
import { startup } from "@anthropic-ai/claude-agent-sdk";

const warm = await startup({ options: { maxTurns: 3 } });
for await (const message of warm.query("What files are here?")) {
  console.log(message);
}
```

### 메시지 타입

| 타입 | 설명 |
| --- | --- |
| `SDKAssistantMessage` | Claude의 응답 (텍스트 + 도구 호출) |
| `SDKUserMessage` | 사용자 입력 / 도구 결과 |
| `SDKResultMessage` | 최종 결과 (비용, 사용량, 세션 ID) |
| `SDKSystemMessage` | 세션 초기화 메타데이터 |
| `SDKPartialAssistantMessage` | 스트리밍 부분 메시지 |
| `SDKCompactBoundaryMessage` | 컨텍스트 압축 경계 |

### 메시지 타입 확인

```typescript
// TypeScript: type 필드 확인
if (message.type === "result" && message.subtype === "success") {
  console.log(message.result);
}
```

---

## 4. Python SDK

### 설치

```bash
pip install claude-agent-sdk
```

Python 3.10 이상 필요. `No matching distribution found` 오류 시 Python 버전을 확인.

### `query()` vs `ClaudeSDKClient`

| 기능 | `query()` | `ClaudeSDKClient` |
| --- | --- | --- |
| 세션 | 호출마다 새 세션 | 동일 세션 유지 |
| 대화 | 단일 교환 | 다중 교환 (문맥 유지) |
| 인터럽트 | 미지원 | 지원 |
| 커스텀 도구 | 지원 | 지원 |
| 훅 | 지원 | 지원 |
| 적합한 용도 | 일회성 작업 | 연속 대화, 채팅 인터페이스 |

### `query()` 사용

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    options = ClaudeAgentOptions(
        system_prompt="You are an expert Python developer",
        permission_mode="acceptEdits",
        cwd="/home/user/project",
    )
    async for message in query(prompt="Create a Python web server", options=options):
        print(message)

asyncio.run(main())
```

### `ClaudeSDKClient` — 연속 대화

```python
import asyncio
from claude_agent_sdk import ClaudeSDKClient, AssistantMessage, TextBlock

async def main():
    async with ClaudeSDKClient() as client:
        await client.query("What's the capital of France?")
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")

        # 후속 질문 — 동일 세션 문맥 유지
        await client.query("What's the population of that city?")
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")

asyncio.run(main())
```

### `ClaudeSDKClient` 메서드

| 메서드 | 설명 |
| --- | --- |
| `connect(prompt)` | 연결 및 초기 프롬프트 전송 |
| `query(prompt)` | 새 요청 전송 |
| `receive_messages()` | 모든 메시지 수신 (AsyncIterator) |
| `receive_response()` | ResultMessage까지 수신 |
| `interrupt()` | 실행 중단 |
| `set_permission_mode(mode)` | 권한 모드 변경 |
| `set_model(model)` | 모델 변경 |
| `rewind_files(user_message_id)` | 파일 상태 복원 |
| `get_mcp_status()` | MCP 서버 상태 조회 |
| `stop_task(task_id)` | 백그라운드 작업 중단 |
| `disconnect()` | 연결 종료 |

### `ClaudeAgentOptions` 주요 필드

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `allowed_tools` | `list[str]` | 자동 승인할 도구 |
| `disallowed_tools` | `list[str]` | 차단할 도구 |
| `permission_mode` | `PermissionMode` | 권한 모드 |
| `model` | `str | None` | 사용할 모델 |
| `max_turns` | `int | None` | 최대 턴 수 |
| `max_budget_usd` | `float | None` | 최대 비용 (USD) |
| `effort` | `EffortLevel` | 추론 노력 수준 |
| `system_prompt` | `str | SystemPromptPreset` | 시스템 프롬프트 |
| `mcp_servers` | `dict[str, McpServerConfig]` | MCP 서버 설정 |
| `agents` | `dict[str, AgentDefinition]` | 서브에이전트 정의 |
| `hooks` | `dict[HookEvent, list[HookMatcher]]` | 훅 설정 |
| `setting_sources` | `list[SettingSource]` | 설정 소스 제어 |
| `cwd` | `str | Path` | 작업 디렉토리 |
| `resume` | `str | None` | 재개할 세션 ID |
| `sandbox` | `SandboxSettings` | 샌드박스 설정 |
| `thinking` | `ThinkingConfig` | 확장 사고 설정 |

### 메시지 타입 확인

```python
# Python: isinstance()로 확인
from claude_agent_sdk import ResultMessage
if isinstance(message, ResultMessage) and message.subtype == "success":
    print(message.result)
```

---

## 5. 에이전트 루프

### 아키텍처

에이전트 루프는 Claude Code CLI와 동일한 실행 루프를 따른다:

1. **프롬프트 수신** — 시스템 프롬프트, 도구 정의, 대화 기록과 함께 전달
2. **평가 및 응답** — Claude가 다음 행동 결정 (텍스트, 도구 호출 또는 둘 다)
3. **도구 실행** — SDK가 각 도구를 실행하고 결과를 수집
4. **반복** — Claude가 도구 호출이 없는 응답을 생성할 때까지 2-3 반복
5. **결과 반환** — `ResultMessage`로 최종 결과, 비용, 세션 ID 반환

### 턴과 메시지

하나의 턴은 Claude가 출력을 생성하고 도구를 호출하며, SDK가 도구를 실행하고 결과를 Claude에게 반환하는 한 번의 왕복이다. 턴은 제어권이 코드로 반환되지 않고 루프 내부에서 자동으로 반복된다.

### 루프 제어

| 옵션 | 설명 | 기본값 |
| --- | --- | --- |
| `max_turns` / `maxTurns` | 최대 도구 사용 턴 수 | 제한 없음 |
| `max_budget_usd` / `maxBudgetUsd` | 최대 비용 임계치 (USD) | 제한 없음 |
| `effort` | 추론 노력 수준 | TS: `'high'`, Py: 모델 기본값 |

### 노력 수준

| 수준 | 동작 | 적합한 용도 |
| --- | --- | --- |
| `low` | 최소 추론, 빠른 응답 | 파일 조회, 디렉토리 나열 |
| `medium` | 균형 잡힌 추론 | 일반 편집, 표준 작업 |
| `high` | 철저한 분석 | 리팩토링, 디버깅 |
| `xhigh` | 확장된 추론 깊이 | 코딩/에이전트 작업, Opus 4.7 권장 |
| `max` | 최대 추론 깊이 | 심층 분석이 필요한 다단계 문제 |

### ResultMessage 하위타입

| 하위타입 | 의미 | `result` 필드 |
| --- | --- | --- |
| `success` | 정상 완료 | 있음 |
| `error_max_turns` | `maxTurns` 한계 도달 | 없음 |
| `error_max_budget_usd` | `maxBudgetUsd` 한계 도달 | 없음 |
| `error_during_execution` | 실행 중 오류 | 없음 |
| `error_max_structured_output_retries` | 구조화된 출력 검증 실패 | 없음 |

### 컨텍스트 윈도우

컨텍스트는 세션 내 턴 간에 누적되며 초기화되지 않는다. 컨텍스트 윈도우 한계에 가까워지면 SDK가 자동으로 이전 대화를 요약하는 **compaction**을 수행한다.

컨텍스트 비용을 줄이는 전략:
- 서브에이전트를 서브태스크에 활용 (각각 새 대화로 시작)
- 도구 정의를 최소화 (`tools` 필드로 제한)
- `effort`를 낮춰 토큰 사용량 감소

---

## 6. 커스텀 도구

커스텀 도구는 SDK의 인프로세스 MCP 서버를 통해 Claude가 호출할 수 있는 사용자 정의 함수를 의미한다.

### 도구 정의 구성요소

| 구성요소 | 설명 |
| --- | --- |
| `name` | 도구의 고유 식별자 |
| `description` | Claude가 언제 호출할지 결정하는 설명 |
| `input_schema` | 입력 파라미터 스키마 (TS: Zod, Py: dict/JSON Schema) |
| `handler` | Claude가 호출 시 실행되는 async 함수 |

### 날씨 도구 예시

**Python**

```python
from claude_agent_sdk import tool, create_sdk_mcp_server, query, ClaudeAgentOptions
from typing import Any

@tool("get_temperature", "Get temperature for a location", {"latitude": float, "longitude": float})
async def get_temperature(args: dict[str, Any]) -> dict[str, Any]:
    # 실제 API 호출 로직
    return {"content": [{"type": "text", "text": f"Temperature: 22°C"}]}

weather_server = create_sdk_mcp_server(name="weather", tools=[get_temperature])

async for message in query(
    prompt="What's the weather at latitude 37.5, longitude 127?",
    options=ClaudeAgentOptions(
        mcp_servers={"weather": weather_server},
        allowed_tools=["mcp__weather__get_temperature"],
    ),
):
    print(message)
```

**TypeScript**

```typescript
import { tool, createSdkMcpServer, query } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const getTemperature = tool(
  "get_temperature",
  "Get temperature for a location",
  { latitude: z.number(), longitude: z.number() },
  async ({ latitude, longitude }) => {
    return { content: [{ type: "text", text: `Temperature: 22°C` }] };
  }
);

const weatherServer = createSdkMcpServer({
  name: "weather",
  tools: [getTemperature],
});

for await (const message of query({
  prompt: "What's the weather at latitude 37.5, longitude 127?",
  options: {
    mcpServers: { weather: weatherServer },
    allowedTools: ["mcp__weather__get_temperature"],
  },
})) {
  console.log(message);
}
```

### 도구 어노테이션

| 필드 | 기본값 | 의미 |
| --- | --- | --- |
| `readOnlyHint` | `false` | 환경을 수정하지 않음 (병렬 실행 가능) |
| `destructiveHint` | `true` | 파괴적 업데이트 수행 가능 |
| `idempotentHint` | `false` | 동일 인자로 반복 호출해도 추가 효과 없음 |
| `openWorldHint` | `true` | 외부 시스템과 상호작용 |

### 도구 이름 형식

MCP 도구의 이름 패턴: `mcp__{server_name}__{tool_name}`

예: `weather` 서버의 `get_temperature` 도구 -> `mcp__weather__get_temperature`

### 에러 처리

| 상황 | 결과 |
| --- | --- |
| 핸들러에서 미처리 예외 발생 | 에이전트 루프 중단 |
| 핸들러에서 `isError: true` 반환 | 루프 계속 진행, Claude가 에러를 데이터로 처리 |

### 결과에서 이미지 및 리소스 반환

`content` 배열에 `text`, `image`, `resource` 블록을 혼합할 수 있다.

```typescript
return {
  content: [
    { type: "image", data: base64Data, mimeType: "image/png" }
  ],
  structuredContent: { series: "temperature", points: [62.1, 63.4] }
};
```

---

## 7. 구조화된 출력

JSON Schema로 에이전트 출력 형태를 정의하고, SDK가 출력을 검증한다.

### 설정

`outputFormat` (TS) / `output_format` (Py) 옵션 사용:

```python
options = ClaudeAgentOptions(
    output_format={
        "type": "json_schema",
        "schema": {
            "type": "object",
            "properties": {
                "name": {"type": "string"},
                "year_founded": {"type": "integer"},
                "headquarters": {"type": "string"},
            },
            "required": ["name", "year_founded", "headquarters"],
        },
    },
)
```

### 타입 안전 스키마

- **TypeScript**: Zod 스키마 + `z.toJSONSchema()`
- **Python**: Pydantic 모델 + `.model_json_schema()`

### 에러 처리

| 하위타입 | 의미 |
| --- | --- |
| `success` | 출력 생성 및 검증 성공 |
| `error_max_structured_output_retries` | 여러 시도 후에도 유효한 출력 생성 불가 |

---

## 8. 스트리밍

`include_partial_messages` (Py) / `includePartialMessages` (TS)를 `true`로 설정하면 실시간으로 텍스트와 도구 호출을 수신할 수 있다.

### 스트리밍 활성화

**Python**

```python
from claude_agent_sdk import query, ClaudeAgentOptions
from claude_agent_sdk.types import StreamEvent

async for message in query(
    prompt="Explain this codebase",
    options=ClaudeAgentOptions(include_partial_messages=True),
):
    if isinstance(message, StreamEvent):
        event = message.event
        if event.get("type") == "content_block_delta":
            delta = event.get("delta", {})
            if delta.get("type") == "text_delta":
                print(delta["text"], end="", flush=True)
```

**TypeScript**

```typescript
for await (const message of query({
  prompt: "Explain this codebase",
  options: { includePartialMessages: true },
})) {
  if (message.type === "stream_event") {
    const event = message.event;
    if (event.type === "content_block_delta" && event.delta?.type === "text_delta") {
      process.stdout.write(event.delta.text);
    }
  }
}
```

### 스트리밍 이벤트 흐름

```
StreamEvent (message_start)
StreamEvent (content_block_start) - 텍스트 블록
StreamEvent (content_block_delta) - 텍스트 청크...
StreamEvent (content_block_stop)
StreamEvent (content_block_start) - tool_use 블록
StreamEvent (content_block_delta) - 도구 입력 청크...
StreamEvent (content_block_stop)
StreamEvent (message_delta)
StreamEvent (message_stop)
AssistantMessage - 완전한 메시지
ResultMessage - 최종 결과
```

### 제한 사항

- 확장 사고(`max_thinking_tokens`) 활성화 시 `StreamEvent` 미발생
- 구조화된 출력은 `ResultMessage.structured_output`에서만 확인 가능

---

## 9. 마이그레이션 가이드

### 이름 변경

| 항목 | 기존 | 신규 |
| --- | --- | --- |
| TypeScript 패키지 | `@anthropic-ai/claude-code` | `@anthropic-ai/claude-agent-sdk` |
| Python 패키지 | `claude-code-sdk` | `claude-agent-sdk` |
| Python 옵션 타입 | `ClaudeCodeOptions` | `ClaudeAgentOptions` |

### TypeScript 마이그레이션

```bash
npm uninstall @anthropic-ai/claude-code
npm install @anthropic-ai/claude-agent-sdk
```

```typescript
// 변경 전
import { query } from "@anthropic-ai/claude-code";
// 변경 후
import { query } from "@anthropic-ai/claude-agent-sdk";
```

### Python 마이그레이션

```bash
pip uninstall claude-code-sdk
pip install claude-agent-sdk
```

```python
# 변경 전
from claude_code_sdk import query, ClaudeCodeOptions
# 변경 후
from claude_agent_sdk import query, ClaudeAgentOptions
```

### 주요 변경 사항

- 시스템 프롬프트가 더 이상 기본으로 로드되지 않음
- `settingSources` 생략 시 CLI와 동일하게 파일시스템 설정 로드 (기본 동작)
- 격리하려면 `settingSources: []` / `setting_sources=[]` 전달

---

## 10. 세션 관리

### 세션 기본

각 `query()` 호출은 세션을 생성하거나 계속한다. `ResultMessage.session_id`에서 세션 ID를 획득할 수 있다.

### 세션 재개

**Python**

```python
from claude_agent_sdk import query, ClaudeAgentOptions, SystemMessage, ResultMessage

session_id = None

# 첫 번째 쿼리: 세션 ID 획득
async for message in query(
    prompt="Read the authentication module",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob"]),
):
    if isinstance(message, SystemMessage) and message.subtype == "init":
        session_id = message.data["session_id"]

# 동일 세션 재개
async for message in query(
    prompt="Now find all places that call it",
    options=ClaudeAgentOptions(resume=session_id),
):
    if isinstance(message, ResultMessage):
        print(message.result)
```

**TypeScript**

```typescript
let sessionId: string | undefined;

for await (const message of query({
  prompt: "Read the authentication module",
  options: { allowedTools: ["Read", "Glob"] },
})) {
  if (message.type === "system" && message.subtype === "init") {
    sessionId = message.session_id;
  }
}

for await (const message of query({
  prompt: "Now find all places that call it",
  options: { resume: sessionId },
})) {
  if (message.type === "result") console.log(message.result);
}
```

### 세션 포크

`forkSession: true` (TS) / `fork_session=True` (Py)를 사용하면 원본 세션을 그대로 두고 새 세션 ID로 분기할 수 있다.

### 세션 유틸리티 함수

| 함수 | 설명 |
| --- | --- |
| `listSessions()` | 과거 세션 목록 조회 |
| `getSessionMessages()` | 세션 메시지 읽기 |
| `getSessionInfo()` | 단일 세션 메타데이터 |
| `renameSession()` | 세션 제목 변경 |
| `tagSession()` | 세션 태그 설정/해제 |

### 외부 스토리지 영속화

`sessionStore` (TS) / `session_store` (Py) 옵션으로 세션 트랜스크립트를 외부 백엔드(S3, Redis, Postgres 등)에 미러링할 수 있다.

---

## 11. MCP 통합

### MCP 서버 추가

**코드로 추가 (stdio)**

```python
options = ClaudeAgentOptions(
    mcp_servers={
        "playwright": {"command": "npx", "args": ["@playwright/mcp@latest"]}
    }
)
```

**코드로 추가 (HTTP)**

```typescript
options: {
  mcpServers: {
    "remote-api": {
      type: "http",
      url: "https://api.example.com/mcp",
      headers: { "Authorization": "Bearer ${API_TOKEN}" }
    }
  }
}
```

**`.mcp.json` 파일로 추가**

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
    }
  }
}
```

### MCP 도구 권한

```typescript
allowedTools: [
  "mcp__github__*",           // github 서버의 모든 도구
  "mcp__db__query",           // db 서버의 query 도구만
  "mcp__slack__send_message"  // slack 서버의 특정 도구
]
```

### 전송 타입

| 타입 | 사용 시나리오 |
| --- | --- |
| `stdio` | 로컬 프로세스 (npx, python 등) |
| `http` | 클라우드 호스팅 MCP 서버 |
| `sse` | SSE 기반 원격 API |
| `sdk` | 인프로세스 SDK MCP 서버 |

### 도구 검색 (Tool Search)

MCP 도구가 많을 경우 컨텍스트 윈도우를 절약하기 위해 기본으로 활성화된다. Claude가 필요할 때만 도구 정의를 로드한다.

---

## 12. 스킬

### 개요

스킬은 Claude가 상황에 맞게 자동으로 호출하는 전문화된 기능이다. `SKILL.md` 파일로 정의되며 파일시스템에서 로드된다.

### SDK에서 스킬 사용

```python
options = ClaudeAgentOptions(
    skills=["all"],  # 모든 발견된 스킬 활성화
    # 또는 특정 스킬만: skills=["code-review", "security-scan"]
)
```

### 스킬 위치

| 위치 | 설명 |
| --- | --- |
| `.claude/skills/` | 프로젝트 스킬 (팀 공유, git) |
| `~/.claude/skills/` | 사용자 스킬 (모든 프로젝트) |
| 플러그인 번들 | 설치된 플러그인에 포함된 스킬 |

### 스킬 구조

```
.claude/skills/processing-pdfs/
└── SKILL.md
```

`SKILL.md`는 YAML 프론트매터와 Markdown 본문으로 구성된다.

---

## 13. 서브에이전트

### 프로그래밍 방식 정의

**Python**

```python
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition

options = ClaudeAgentOptions(
    allowed_tools=["Read", "Glob", "Grep", "Agent"],
    agents={
        "code-reviewer": AgentDefinition(
            description="Expert code reviewer for quality and security reviews.",
            prompt="Analyze code quality and suggest improvements.",
            tools=["Read", "Glob", "Grep"],
        )
    },
)

async for message in query(
    prompt="Use the code-reviewer agent to review this codebase",
    options=options,
):
    print(message)
```

**TypeScript**

```typescript
const options: Options = {
  allowedTools: ["Read", "Glob", "Grep", "Agent"],
  agents: {
    "code-reviewer": {
      description: "Expert code reviewer for quality and security reviews.",
      prompt: "Analyze code quality and suggest improvements.",
      tools: ["Read", "Glob", "Grep"],
    },
  },
};
```

### AgentDefinition 필드

| 필드 | 필수 | 설명 |
| --- | --- | --- |
| `description` | Yes | 에이전트 사용 시기를 설명하는 자연어 설명 |
| `prompt` | Yes | 에이전트의 시스템 프롬프트 |
| `tools` | No | 허용된 도구 목록 (생략 시 부모 도구 상속) |
| `model` | No | 모델 오버라이드 (`sonnet`, `opus`, `haiku`, `inherit`) |
| `maxTurns` | No | 최대 턴 수 |
| `background` | No | 백그라운드 작업으로 실행 |
| `effort` | No | 추론 노력 수준 |
| `permissionMode` | No | 권한 모드 |
| `skills` | No | 프리로드할 스킬 이름 목록 |

### 서브에이전트가 상속하는 것

| 상속 | 미상속 |
| --- | --- |
| 자체 시스템 프롬프트 + Agent 도구의 프롬프트 | 부모의 대화 기록/도구 결과 |
| 프로젝트 CLAUDE.md | 부모의 시스템 프롬프트 |
| 도구 정의 (상속 또는 `tools`로 제한) | 프리로드된 스킬 콘텐츠 (`skills`에 명시된 경우만) |

### 서브에이전트 재개

서브에이전트가 완료되면 Agent 도구 결과에서 `agentId`를 얻을 수 있다. 동일 세션을 재개하면 서브에이전트의 대화 기록이 그대로 복원된다.

---

## 14. 시스템 프롬프트 수정

### 커스텀 프롬프트

```python
options = ClaudeAgentOptions(
    system_prompt="You are an expert code reviewer. Focus on security."
)
```

### Claude Code 프리셋 사용

```python
options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": "Always add detailed comments to code changes.",
    }
)
```

### 프리셋 옵션

| 필드 | 설명 |
| --- | --- |
| `type` | `"preset"` |
| `preset` | `"claude_code"` |
| `append` | 프리셋에 추가할 지시사항 |
| `exclude_dynamic_sections` | 동적 섹션을 첫 사용자 메시지로 이동 (프롬프트 캐시 개선) |

---

## 15. 비용 추적

`ResultMessage`에서 비용 및 토큰 사용량을 확인할 수 있다.

### 주요 필드

| 필드 | 설명 |
| --- | --- |
| `total_cost_usd` | 총 비용 (USD, 클라이언트 측 추정치) |
| `usage` | 토큰 사용량 (input/output/cache) |
| `model_usage` / `modelUsage` | 모델별 상세 사용량 |
| `num_turns` | 총 턴 수 |
| `duration_ms` | 총 실행 시간 (ms) |

### 모델별 사용량 (TypeScript)

```typescript
// modelUsage: { [modelName: string]: ModelUsage }
// ModelUsage 필드: inputTokens, outputTokens, cacheReadInputTokens,
//   cacheCreationInputTokens, webSearchRequests, costUSD, contextWindow, maxOutputTokens
```

### 비용 제한

```python
options = ClaudeAgentOptions(max_budget_usd=1.0)  # $1에서 중단
```

---

## 16. 슬래시 명령어

SDK를 통해 `/`로 시작하는 명령어를 전송할 수 있다. 프롬프트 문자열에 직접 포함하면 된다.

### 주요 내장 명령어

| 명령어 | 설명 |
| --- | --- |
| `/compact` | 대화 기록 압축 (오래된 메시지 요약) |
| `/clear` | 대화 컨텍스트 초기화 |

### 커스텀 명령어

`.claude/commands/` 디렉토리에 Markdown 파일로 정의:

```markdown
---
allowed-tools: Read, Grep, Glob
description: Run security vulnerability scan
model: claude-opus-4-7
---

Analyze the codebase for security vulnerabilities including:
- SQL injection risks
- XSS vulnerabilities
- Exposed credentials
```

---

## 17. 권한

### 권한 평가 순서

1. `disallowed_tools`의 bare 이름 -> 도구 정의를 요청에서 제거
2. `disallowed_tools`의 범위 지정 규칙 -> 매칭되는 호출 거부
3. `allowed_tools` -> 나열된 도구 자동 승인
4. 훅 (`PreToolUse`) -> 커스텀 로직으로 승인/거부/수정
5. 권한 모드 -> 나머지 도구의 처리 방식 결정
6. `canUseTool` 콜백 -> 런타임 승인 프롬프트

### 권한 모드 상세

| 모드 | 동작 |
| --- | --- |
| `default` | 자동 승인 없음; 미매칭 도구는 `canUseTool` 콜백 트리거 |
| `dontAsk` | `allowed_tools` 외의 모든 요청 거부; `canUseTool` 호출 안 함 |
| `acceptEdits` | 파일 편집 및 파일시스템 명령 자동 승인; 기타는 일반 권한 규칙 |
| `bypassPermissions` | 모든 권한 프롬프트 생략; 훅은 여전히 실행 |
| `plan` | 읽기 전용 도구만 실행; 편집 불가 |
| `auto` (TS만) | 모델 분류기가 각 도구 호출을 승인/거부 |

### `canUseTool` 콜백

```typescript
const options: Options = {
  permissionMode: "default",
  canUseTool: async (toolName, input, { signal, suggestions }) => {
    if (toolName === "Bash" && input.command?.includes("rm -rf")) {
      return { behavior: "deny", message: "Dangerous command blocked" };
    }
    return { behavior: "allow", updatedInput: input };
  },
};
```

---

## 18. 훅

훅은 에이전트 루프의 특정 지점에서 실행되는 콜백 함수다. 에이전트 컨텍스트 외부에서 실행되므로 컨텍스트를 소비하지 않는다.

### 주요 훅 이벤트

| 이벤트 | 발생 시점 | 주요 용도 |
| --- | --- | --- |
| `PreToolUse` | 도구 실행 전 | 입력 검증, 위험 명령 차단 |
| `PostToolUse` | 도구 실행 후 | 출력 감사, 사이드 이펙트 트리거 |
| `UserPromptSubmit` | 프롬프트 전송 시 | 프롬프트에 추가 컨텍스트 주입 |
| `Stop` | 에이전트 종료 시 | 결과 검증, 세션 상태 저장 |
| `SubagentStart` / `SubagentStop` | 서브에이전트 시작/완료 | 병렬 작업 추적 |
| `PreCompact` | 컨텍스트 압축 전 | 전체 트랜스크립트 아카이빙 |
| `SessionStart` / `SessionEnd` | 세션 시작/종료 | 세션 초기화, 정리 |

### 훅 예시 — 파일 변경 감사

**Python**

```python
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher
from datetime import datetime

async def log_file_change(input_data, tool_use_id, context):
    file_path = input_data.get("tool_input", {}).get("file_path", "unknown")
    with open("./audit.log", "a") as f:
        f.write(f"{datetime.now()}: modified {file_path}\n")
    return {}

options = ClaudeAgentOptions(
    permission_mode="acceptEdits",
    hooks={
        "PostToolUse": [
            HookMatcher(matcher="Edit|Write", hooks=[log_file_change])
        ],
    },
)
```

**TypeScript**

```typescript
const logFileChange: HookCallback = async (input, toolUseId, { signal }) => {
  const filePath = input.tool_input?.file_path ?? "unknown";
  fs.appendFileSync("./audit.log", `${new Date()}: modified ${filePath}\n`);
  return {};
};

const options: Options = {
  permissionMode: "acceptEdits",
  hooks: {
    PostToolUse: [{ matcher: "Edit|Write", hooks: [logFileChange] }],
  },
};
```

---

## 19. 보안 배포

### 서브프로세스 모델

Agent SDK는 `query()` 호출 시 `claude` CLI 서브프로세스를 생성하고 stdio로 통신한다. 하나의 에이전트 세션 = 하나의 서브프로세스.

### 로컬 디스크 상태

| 상태 | 기본 위치 |
| --- | --- |
| 세션 트랜스크립트 | `~/.claude/projects/` |
| CLAUDE.md 메모리 | `~/.claude/CLAUDE.md` 및 작업 디렉토리 |
| 작업 디렉토리 산출물 | 세션의 작업 디렉토리 |

### 세션 패턴

| 패턴 | 설명 | 적합한 워크로드 |
| --- | --- | --- |
| 임시 세션 | 작업 완료 후 컨테이너 파기 | 버그 수정, 문서 번역 |
| 장기 세션 | 지속적 컨테이너, 다중 세션 | 이메일 에이전트, 챗봇 |
| 하이브리드 세션 | `SessionStore`에서 세션 복원/저장 | 심층 리서치, 간헐적 체크인 |
| 멀티 에이전트 컨테이너 | 하나의 컨테이너에 여러 서브프로세스 | 멀티 에이전트 시뮬레이션 |

### 리소스 권장사항

| 리소스 | 시작점 |
| --- | --- |
| RAM | 1 GiB/에이전트 |
| 디스크 | 5 GiB |
| CPU | 1 코어/에이전트 |

### 관측 가능성 (OpenTelemetry)

```bash
CLAUDE_CODE_ENABLE_TELEMETRY=1
CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1
OTEL_TRACES_EXPORTER=otlp
OTEL_METRICS_EXPORTER=otlp
OTEL_EXPORTER_OTLP_ENDPOINT=http://collector.example.com:4318
```

### 멀티테넌트 격리

```python
options = ClaudeAgentOptions(
    setting_sources=[],           # 파일시스템 설정 로드 안 함
    cwd=f"/tenants/{tenant_id}",  # 테넌트별 작업 디렉토리
    env={
        "CLAUDE_CODE_DISABLE_AUTO_MEMORY": "1",
        "CLAUDE_CONFIG_DIR": f"/tenants/{tenant_id}/.claude",
    },
)
```

---

## 20. 플러그인

SDK에서 로컬 플러그인을 로드할 수 있다. 플러그인은 스킬, 에이전트, 훅, MCP 서버를 패키지로 제공한다.

```typescript
plugins: [
  { type: "local", path: "./my-plugin" },
  { type: "local", path: "/absolute/path/to/plugin" },
]
```

```python
options = ClaudeAgentOptions(
    plugins=[
        {"type": "local", "path": "./my-plugin"},
    ]
)
```

---

## 21. Claude Code 기능

SDK는 Claude Code의 파일시스템 기반 설정을 지원한다. `settingSources` / `setting_sources`로 로드 소스를 제어할 수 있다.

| 기능 | 설명 | 위치 |
| --- | --- | --- |
| Skills | Claude가 자동/수동으로 호출하는 전문 기능 | `.claude/skills/*/SKILL.md` |
| Commands | 커스텀 명령어 (레거시, 신규는 스킬 사용) | `.claude/commands/*.md` |
| Memory | 프로젝트 컨텍스트 및 지시사항 | `CLAUDE.md` 또는 `.claude/CLAUDE.md` |
| Plugins | 스킬, 에이전트, 훅, MCP 서버 확장 | 프로그래밍 방식 `plugins` 옵션 |

### 설정 소스 우선순위 (높은 순)

1. Local 설정 (`.claude/settings.local.json`)
2. Project 설정 (`.claude/settings.json`)
3. User 설정 (`~/.claude/settings.json`)
4. 프로그래밍 옵션 (`agents`, `allowedTools` 등)
5. 관리 정책 설정

### 내장 도구

| 카테고리 | 도구 | 설명 |
| --- | --- | --- |
| 파일 작업 | `Read`, `Edit`, `Write` | 파일 읽기, 수정, 생성 |
| 검색 | `Glob`, `Grep` | 패턴으로 파일 찾기, 정규식으로 콘텐츠 검색 |
| 실행 | `Bash` | 셸 명령어, 스크립트, git 작업 |
| 모니터링 | `Monitor` | 백그라운드 스크립트 감시 |
| 웹 | `WebSearch`, `WebFetch` | 웹 검색, 페이지 콘텐츠 가져오기 |
| 오케스트레이션 | `Agent`, `Skill`, `AskUserQuestion`, `TaskCreate`, `TaskUpdate` | 서브에이전트, 스킬, 사용자 질문, 작업 추적 |
| 발견 | `ToolSearch` | 도구를 온디맨드로 동적 로드 |
