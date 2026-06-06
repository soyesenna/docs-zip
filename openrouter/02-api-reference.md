# OpenRouter API 레퍼런스

> 원문: https://openrouter.ai/docs/api-reference/overview
> https://openrouter.ai/docs/api-reference
> https://openrouter.ai/docs/requests

OpenRouter의 요청/응답 스키마는 OpenAI Chat API와 매우 유사합니다. **OpenRouter는 모델과 프로바이더 간 스키마를 정규화**하므로 하나의 스키마만 학습하면 됩니다.

---

## OpenAPI 스펙

완전한 API는 OpenAPI 스펙으로 문서화되어 있습니다 (YAML/JSON 형식 모두 제공). Swagger UI, Postman 또는 OpenAPI 호환 코드 생성기에 사용할 수 있습니다.

---

## 요청 (Request)

### Completions 요청 포맷

`POST /api/v1/chat/completions` 엔드포인트의 요청 본문 TypeScript 타입 정의:

```typescript
// Definitions of subtypes are below
type Request = {
  // "messages" 또는 "prompt" 중 하나 필수
  messages?: Message[];
  prompt?: string;

  // "model"이 지정되지 않으면 사용자 기본값 사용
  model?: string;

  // 특정 출력 포맷 강제 (Structured Outputs)
  response_format?: ResponseFormat;

  stop?: string | string[];
  stream?: boolean; // 스트리밍 활성화

  // 모델 기능 확장 플러그인
  plugins?: Plugin[];

  // LLM 파라미터
  max_tokens?: number;       // 범위: [1, context_length)
  temperature?: number;      // 범위: [0.0, 2.0], 기본값: 1.0

  // 툴 콜링
  tools?: Tool[];
  tool_choice?: ToolChoice;

  // 샘플링 파라미터
  seed?: number;                     // 정수만 (재현 가능한 결과)
  top_p?: number;                    // 범위: (0, 1], 기본값: 1.0
  top_k?: number;                    // 범위: [1, Infinity), OpenAI 모델은 미지원
  frequency_penalty?: number;        // 범위: [-2.0, 2.0], 기본값: 0.0
  presence_penalty?: number;         // 범위: [-2.0, 2.0], 기본값: 0.0
  repetition_penalty?: number;       // 범위: (0, 2], 기본값: 1.0
  min_p?: number;                    // 범위: [0, 1], 기본값: 0.0
  top_a?: number;                    // 범위: [0, 1], 기본값: 0.0
  logit_bias?: { [key: number]: number };
  top_logprobs: number;              // 정수만
  logprobs?: boolean;                // 출력 토큰의 로그 확률 반환 여부

  // 지연 시간 최적화 (predicted output)
  prediction?: { type: 'content'; content: string };

  // OpenRouter 전용 파라미터
  transforms?: string[];       // 프롬프트 변환 (예: 메시지 정규화)
  models?: string[];           // 모델 폴백 배열
  route?: 'fallback';
  provider?: ProviderPreferences; // 프로바이더 라우팅 설정
  user?: string;               // 사용자 식별자 (남용 방지용)

  // 디버그 옵션 (스트리밍 전용)
  debug?: {
    echo_upstream_body?: boolean; // true면 프로바이더에 전송된 변환된 요청 본문 반환
  };
};
```

### 서브타입 정의

```typescript
type TextContent = {
  type: 'text';
  text: string;
};

type ImageContentPart = {
  type: 'image_url';
  image_url: {
    url: string;    // URL 또는 base64 인코딩 이미지 데이터
    detail?: string; // 기본값: "auto"
  };
};

type ContentPart = TextContent | ImageContentPart;

type Message =
  | {
      role: 'user' | 'assistant' | 'system';
      content: string | ContentPart[]; // ContentPart는 "user" 역할만
      name?: string;
    }
  | {
      role: 'tool';
      content: string;
      tool_call_id: string;
      name?: string;
    };

type FunctionDescription = {
  description?: string;
  name: string;
  parameters: object; // JSON Schema 객체
};

type Tool = {
  type: 'function';
  function: FunctionDescription;
};

type ToolChoice =
  | 'none'
  | 'auto'
  // 참고: 공식 parameters 문서에는 'required'도 지원된다고 명시되어 있으나
  // 공식 TypeScript 타입 정의에는 포함되지 않음
  | {
      type: 'function';
      function: {
        name: string;
      };
    };

type ResponseFormat =
  | { type: 'json_object' }
  | {
      type: 'json_schema';
      json_schema: {
        name: string;
        strict?: boolean;
        schema: object; // JSON Schema 객체
      };
    };

type Plugin = {
  id: string; // 'web', 'file-parser', 'response-healing', 'context-compression'
  enabled?: boolean;
  [key: string]: unknown; // 플러그인별 추가 옵션
};
```

---

## Structured Outputs

`response_format` 파라미터로 모델의 JSON 응답을 강제할 수 있습니다:

- `{ type: 'json_object' }`: 기본 JSON 모드 -- 모델이 유효한 JSON을 반환
- `{ type: 'json_schema', json_schema: { ... } }`: 엄격한 스키마 모드 -- 지정한 스키마에 정확히 일치하는 JSON 반환

자세한 사용법은 Structured Outputs 문서를 참조하세요. 구조화된 출력을 지원하는 모델은 모델 페이지에서 확인할 수 있습니다.

---

## Plugins

플러그인은 요청의 `plugins` 배열로 활성화합니다:

```json
{
  "plugins": [
    { "id": "web" },
    { "id": "response-healing" }
  ]
}
```

사용 가능한 플러그인:

| 플러그인 ID | 기능 |
| --- | --- |
| `web` | 실시간 웹 검색 |
| `file-parser` | PDF 처리 |
| `response-healing` | JSON 자동 수리 |
| `context-compression` | 중간 프롬프트 압축 |

자세한 설정 옵션은 Plugins 문서를 참조하세요.

---

## Assistant Prefill

모델에 부분 응답을 완성하도록 요청할 수 있습니다. `messages` 배열 끝에 `role: "assistant"` 메시지를 추가합니다:

```javascript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-5.2',
    messages: [
      { role: 'user', content: 'What is the meaning of life?' },
      { role: 'assistant', content: "I'm not sure, but my best guess is" },
    ],
  }),
});
```

---

## 응답 (Response)

### CompletionsResponse 포맷

OpenRouter는 모델/프로바이더 간 스키마를 OpenAI Chat API에 맞게 정규화합니다. `choices`는 항상 배열이며, 스트리밍 요청 시 `delta` 속성을, 비스트리밍 요청 시 `message` 속성을 포함합니다.

```typescript
type Response = {
  id: string;
  choices: (NonStreamingChoice | StreamingChoice | NonChatChoice)[];
  created: number;           // Unix 타임스탬프
  model: string;
  object: 'chat.completion' | 'chat.completion.chunk';
  system_fingerprint?: string; // 프로바이더가 지원하는 경우에만 존재
  usage?: ResponseUsage;
};
```

### Usage 정보

```typescript
type ResponseUsage = {
  prompt_tokens: number;       // 이미지, 오디오, 툴 포함
  completion_tokens: number;   // 생성된 토큰
  total_tokens: number;        // 위 두 값의 합

  prompt_tokens_details?: {
    cached_tokens: number;
    cache_write_tokens?: number;
    audio_tokens?: number;
    video_tokens?: number;
  };

  completion_tokens_details?: {
    reasoning_tokens?: number;
    audio_tokens?: number;
    image_tokens?: number;
  };

  cost?: number;               // 크레딧 비용
  is_byok?: boolean;           // BYOK 사용 여부
  cost_details?: {
    upstream_inference_cost?: number;
    upstream_inference_prompt_cost: number;
    upstream_inference_completions_cost: number;
  };

  server_tool_use?: {
    web_search_requests?: number;
  };
};
```

### Choice 서브타입

```typescript
type NonStreamingChoice = {
  finish_reason: string | null;
  native_finish_reason: string | null;
  message: {
    content: string | null;
    role: string;
    tool_calls?: ToolCall[];
  };
  error?: ErrorResponse;
};

type StreamingChoice = {
  finish_reason: string | null;
  native_finish_reason: string | null;
  delta: {
    content: string | null;
    role?: string;
    tool_calls?: ToolCall[];
  };
  error?: ErrorResponse;
};

type NonChatChoice = {
  finish_reason: string | null;
  text: string;
  error?: ErrorResponse;
};

type ToolCall = {
  id: string;
  type: 'function';
  function: FunctionCall;
};

type ErrorResponse = {
  code: number;
  message: string;
  metadata?: Record<string, unknown>; // 프로바이더 세부 정보, 원본 에러 메시지 등
};
```

---

## Finish Reason 정규화

OpenRouter는 각 모델의 `finish_reason`을 다음 값 중 하나로 정규화합니다:

| 정규화된 값 | 의미 |
| --- | --- |
| `tool_calls` | 툴 콜이 요청됨 |
| `stop` | 정상 종료 |
| `length` | 최대 토큰 도달 |
| `content_filter` | 콘텐츠 필터링 |
| `error` | 오류 발생 |

일부 모델/프로바이더는 추가 finish reason을 가질 수 있습니다. 원래의 `finish_reason`은 `native_finish_reason` 속성에서 확인할 수 있습니다.

---

## 비용 및 통계 조회

반환된 `id`를 사용하여 요청 완료 후 `/api/v1/generation` 엔드포인트로 통계를 조회할 수 있습니다. 이는 과거 사용량 감사나 비동기 통계 조회에 유용합니다.

> **참고**: 비스트리밍 완성의 경우 토큰 수는 응답 본문의 `usage` 필드에서도 확인할 수 있습니다. 토큰 수는 모델의 네이티브 토크나이저를 사용하여 계산됩니다.

```javascript
const generation = await fetch(
  'https://openrouter.ai/api/v1/generation?id=$GENERATION_ID',
  { headers },
);
const stats = await generation.json();
```

자세한 응답 형태는 Generation API 레퍼런스를 참조하세요.

---

## Reasoning (추론) 파라미터

> 원문: https://openrouter.ai/docs/guides/reasoning-tokens

추론 토큰을 지원하는 모델에서 `reasoning` 객체로 추론 동작을 제어할 수 있습니다. 추론 토큰은 출력 토큰으로 간주되어 해당 비용이 청구됩니다.

추론 토큰은 모델이 출력하기로 결정한 경우 기본적으로 응답에 포함되며, `exclude`를 설정하지 않는 한 각 메시지의 `reasoning` 필드에 나타납니다.

> **참고**: 일부 추론 모델(예: OpenAI o-시리즈)은 추론 토큰을 반환하지 않습니다.

```json
{
  "model": "your-model",
  "messages": [],
  "reasoning": {
    "effort": "high",
    "max_tokens": 2000,
    "exclude": false
  }
}
```

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `reasoning.enabled` | `boolean` | 추론 기능 활성화 (기본값: `effort` 또는 `max_tokens`에서 추론) |
| `reasoning.effort` | `string` | 추론 노력 수준 (OpenAI 스타일): `xhigh`, `high`, `medium`, `low`, `minimal`, `none` |
| `reasoning.max_tokens` | `number` | 최대 추론 토큰 수 (Anthropic/Gemini/Qwen 스타일) |
| `reasoning.exclude` | `boolean` | `true`면 응답에서 추론 토큰 제외 (기본값: `false`) |

### effort별 토큰 할당 비율

| effort | 할당 비율 (max_tokens 대비) |
| --- | --- |
| `xhigh` | 약 95% |
| `high` | 약 80% |
| `medium` | 약 50% |
| `low` | 약 20% |
| `minimal` | 약 10% |
| `none` | 추론 비활성화 |

### Reasoning Details (응답)

추론 모델의 응답에는 `reasoning_details` 배열이 포함됩니다:

- **비스트리밍**: `choices[].message.reasoning_details`
- **스트리밍**: `choices[].delta.reasoning_details`

각 reasoning detail 객체의 타입:

| 타입 | 필드 | 설명 |
| --- | --- | --- |
| `reasoning.summary` | `summary` | 추론 과정의 요약 |
| `reasoning.encrypted` | `data` | 암호화된 추론 데이터 |
| `reasoning.text` | `text`, `signature` | 원시 텍스트 추론 (선택적 서명) |

모든 객체는 공통으로 `id`, `format`, `index` 필드를 포함합니다.

### 레거시 파라미터

하위 호환성을 위해 다음 레거시 파라미터가 지원됩니다:

| 필드 | 설명 |
| --- | --- |
| `include_reasoning: true` | `reasoning: {}`와 동일 |
| `include_reasoning: false` | `reasoning: { exclude: true }`와 동일 |

새로운 통합 `reasoning` 파라미터 사용을 권장합니다.

---

## Verbosity (상세도)

모델 응답의 상세도를 제어합니다:

```json
{
  "verbosity": "medium"
}
```

| 값 | 설명 |
| --- | --- |
| `low` | 간결한 응답 |
| `medium` | 일반적인 상세도 (기본값) |
| `high` | 상세한 응답 |

---

## 스트리밍

`stream: true`를 설정하면 Server-Sent Events (SSE)로 **모든 모델**에 대해 스트리밍이 지원됩니다. SSE 스트림에 연결 타임아웃 방지용 주석 페이로드가 포함될 수 있으며, 이는 무시해야 합니다.

### 스트림 취소

스트리밍 요청은 연결을 중단하여 취소할 수 있습니다. 지원되는 프로바이더의 경우 모델 처리와 요금 청구가 즉시 중지됩니다.

---

## 비표준 파라미터

선택한 모델이 요청 파라미터를 지원하지 않는 경우 (예: 비OpenAI 모델의 `logit_bias`, OpenAI 모델의 `top_k`) 해당 파라미터는 무시됩니다. 나머지 파라미터는 기본 모델 API로 전달됩니다.

---

## 모델 라우팅

`model` 파라미터를 생략하면 사용자(또는 결제자)의 기본 모델이 사용됩니다. `models` 배열과 `route: 'fallback'`을 사용하여 폴백 체인을 구성할 수 있습니다. OpenRouter는 비용이 가장 낮고 성능이 가장 좋은 GPU를 선택하여 요청을 처리하며, 5xx 응답 코드를 수신하거나 요청 제한에 걸린 경우 다른 프로바이더나 GPU로 폴백합니다.

---

## 관련 문서

- [Quickstart](./01-quickstart.md)
- [툴 콜링 가이드](./03-tool-calling.md)
- [Structured Outputs](./05-structured-outputs.md)
- [플러그인](./09-plugins.md)
- [Reasoning Tokens 가이드](https://openrouter.ai/docs/guides/reasoning-tokens)
- [API Parameters](https://openrouter.ai/docs/api-reference/parameters)
- [API Streaming](https://openrouter.ai/docs/api-reference/streaming)
