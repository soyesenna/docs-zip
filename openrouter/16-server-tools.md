# OpenRouter Server Tools 가이드

> 원문: https://openrouter.ai/docs/guides/features/server-tools
>
> **Beta**: Server Tools는 현재 베타 상태입니다. API와 동작이 변경될 수 있습니다.

Server Tools는 OpenRouter가 운영하는 특수 도구로, 요청 중 **모든 모델**이 호출할 수 있습니다. 모델이 서버 툴을 사용하기로 결정하면, OpenRouter가 서버 측에서 실행하고 결과를 모델에 반환합니다 — **클라이언트 측 구현이 필요하지 않습니다.**

---

## Server Tools vs Plugins vs User-Defined Tools

| 구분 | Server Tools | Plugins | User-Defined Tools |
| --- | --- | --- | --- |
| **실행 결정** | 모델이 결정 | 항상 실행 | 모델이 결정 |
| **실행 주체** | OpenRouter | OpenRouter | 사용자 애플리케이션 |
| **호출 빈도** | 요청당 0~N회 | 요청당 1회 | 요청당 0~N회 |
| **지정 방식** | `tools` 배열 | `plugins` 배열 | `tools` 배열 |
| **타입 접두사** | `openrouter:*` | N/A | `function` |

**Server Tools**: 모델이 요청 중 0회 이상 호출할 수 있는 툴. OpenRouter가 투명하게 실행을 처리합니다.

**Plugins**: 요청이나 응답을 주입/변형하여 기능을 추가(예: 응답 복구, PDF 파싱). 활성화 시 항상 한 번 실행됩니다.

**User-Defined Tools**: 표준 함수 콜링 툴. 모델이 호출을 제안하고, 사용자 애플리케이션이 실행합니다.

---

## 사용 가능한 Server Tools

| 툴 | 타입 | 설명 |
| --- | --- | --- |
| **Web Search** | `openrouter:web_search` | 웹에서 최신 정보 검색 |
| **Datetime** | `openrouter:datetime` | 현재 날짜와 시간 조회 |
| **Image Generation** | `openrouter:image_generation` | 텍스트 프롬프트로 이미지 생성 |

---

## 작동 방식

1. API 요청의 `tools` 배열에 하나 이상의 서버 툴을 포함합니다
2. 모델이 사용자 프롬프트를 기반으로 각 서버 툴을 호출할지 결정합니다
3. OpenRouter가 툴 콜을 가로채어 서버 측에서 실행하고, 결과를 모델에 반환합니다
4. 모델이 결과를 사용하여 응답을 작성합니다. 필요한 경우 툴을 다시 호출할 수도 있습니다

서버 툴은 사용자 정의 툴과 함께 사용할 수 있습니다 — 같은 요청에 두 가지를 모두 포함할 수 있습니다.

---

## Quick Start

서버 툴을 `tools` 배열에 `openrouter:` 타입 접두사를 사용하여 추가합니다:

```typescript
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <YOUR_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-5.2',
    messages: [
      {
        role: 'user',
        content: 'What are the latest developments in AI?',
      },
    ],
    tools: [
      { type: 'openrouter:web_search' },
      { type: 'openrouter:datetime' },
    ],
  }),
});

const data = await response.json();
console.log(data.choices[0].message.content);
```

---

## User-Defined Tools와 결합

서버 툴과 사용자 정의 툴을 같은 요청에 사용할 수 있습니다:

```json
{
  "model": "openai/gpt-5.2",
  "messages": ["..."],
  "tools": [
    { "type": "openrouter:web_search", "parameters": { "max_results": 3 } },
    { "type": "openrouter:datetime" },
    {
      "type": "function",
      "function": {
        "name": "get_stock_price",
        "description": "Get the current stock price for a ticker symbol",
        "parameters": {
          "type": "object",
          "properties": {
            "ticker": { "type": "string" }
          },
          "required": ["ticker"]
        }
      }
    }
  ]
}
```

모델은 서버 툴과 사용자 정의 툴의 조합을 자유롭게 호출할 수 있습니다. OpenRouter가 서버 툴을 자동으로 실행하고, 사용자 정의 툴 콜은 애플리케이션이 평소처럼 처리합니다.

---

## Web Search (`openrouter:web_search`)

웹에서 실시간 정보를 검색합니다. Web Search **플러그인**(`{ "id": "web" }`)과 달리, 서버 툴은 모델이 필요에 따라 0~N번 검색을 수행할 수 있습니다.

### 파라미터

| 파라미터 | 타입 | 설명 |
| --- | --- | --- |
| `max_results` | number | 최대 검색 결과 수 |

### 플러그인과의 비교

| 구분 | Server Tool (`openrouter:web_search`) | Plugin (`{ "id": "web" }`) |
| --- | --- | --- |
| 호출 횟수 | 0~N회 (모델이 결정) | 항상 1회 |
| 컨트롤 | 모델이 필요시에만 검색 | 모든 요청에서 항상 검색 |
| 비용 효율성 | 검색이 필요 없으면 비용 발생 안함 | 모든 요청에서 검색 비용 발생 |

> **권장**: Web Search 플러그인은 deprecated 되었습니다. 대신 `openrouter:web_search` 서버 툴 사용을 권장합니다.

---

## Datetime (`openrouter:datetime`)

현재 날짜와 시간을 조회합니다. 모델이 시간 관련 질문에 답변할 때 유용합니다.

```json
{
  "tools": [
    { "type": "openrouter:datetime" }
  ]
}
```

---

## Image Generation (`openrouter:image_generation`)

텍스트 프롬프트로 이미지를 생성합니다.

```json
{
  "tools": [
    { "type": "openrouter:image_generation" }
  ]
}
```

---

## 사용량 추적

서버 툴 사용량은 응답의 `usage` 객체에서 추적할 수 있습니다:

```json
{
  "usage": {
    "prompt_tokens": 105,
    "completion_tokens": 250,
    "server_tool_use": {
      "web_search_requests": 2
    }
  }
}
```

---

## 관련 문서

- [툴 콜링 (Function Calling)](./03-tool-calling.md)
- [웹 검색](./10-web-search.md)
- [플러그인](./09-plugins.md)
- [API 레퍼런스](./02-api-reference.md)
