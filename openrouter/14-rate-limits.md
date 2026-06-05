# OpenRouter Rate Limits (속도 제한) 가이드

> 원문: https://openrouter.ai/docs (다양한 페이지에 분산된 정보 통합)

OpenRouter는 전역 용량 관리 방식으로 속도 제한을 운영합니다. 추가 계정이나 API 키를 생성해도 속도 제한에 영향을 주지 않습니다.

---

## API 키 정보 조회

`GET /api/v1/key` 엔드포인트로 API 키의 크레딧 잔액과 사용량을 조회할 수 있습니다.

### 요청

```bash
curl https://openrouter.ai/api/v1/key \
  -H "Authorization: Bearer $OPENROUTER_API_KEY"
```

### 응답 타입 (TypeScript)

```typescript
type Key = {
  label: string;               // API 키 라벨
  limit: number | null;        // 사용 한도 (null이면 무제한)
  limit_remaining: number;     // 남은 한도
  usage: number;               // 총 사용량
  usage_daily: number;         // 일일 사용량
  usage_weekly: number;        // 주간 사용량
  usage_monthly: number;       // 월간 사용량

  // BYOK (Bring Your Own Key) 사용량
  byok_usage: number;
  byok_usage_daily: number;
  byok_usage_weekly: number;
  byok_usage_monthly: number;

  is_free_tier: boolean;       // 무료 티어 여부
  rate_limit: number;          // 분당 요청 수 제한
  rate_limit_per_model: Record<string, number>; // 모델별 요청 수 제한
  usage_explicit_api_key: boolean;  // 명시적 API 키 사용량 포함 여부
  usage_all_api_keys: boolean;      // 모든 API 키 사용량 포함 여부
  include_byok_in_limit: boolean;   // BYOK 사용량을 한도에 포함할지 여부
};
```

---

## 무료 모델 제한

`:free` 변형 모델을 사용할 때 다음 제한이 적용됩니다:

| 조건 | 제한 |
| --- | --- |
| 분당 최대 요청 수 | 20 req/min |
| 일일 요청 수 (크레딧 < $10) | 50 req/day |
| 일일 요청 수 (크레딧 ≥ $10) | 1,000 req/day |

무료 모델의 사용량도 크레딧 잔액에 영향을 줄 수 있습니다.

---

## DDoS 보호

OpenRouter는 Cloudflare를 통해 DDoS 보호를 제공합니다. 비정상적으로 높은 요청 빈도가 감지되면 일시적으로 요청이 차단될 수 있습니다.

---

## 음수 잔액 처리

크레딧 잔액이 음수가 되면 다음과 같이 처리됩니다:

- **402 Payment Required** 오류가 반환됩니다
- 무료 모델(`:free`)을 포함한 모든 요청이 거부됩니다
- 크레딧을 충전해야 다시 API를 사용할 수 있습니다

---

## 속도 제한 관리 팁

### 1. 모델 폴백 활용

```typescript
const completion = await openRouter.chat.send({
  models: [
    'anthropic/claude-sonnet-4.5',
    'openai/gpt-5-mini',
    'meta-llama/llama-3.3-70b-instruct',
  ],
  messages: [{ role: 'user', content: 'Hello' }],
  stream: false,
});
```

하나의 모델이 속도 제한에 걸리면 자동으로 다음 모델로 폴백됩니다.

### 2. Provider 정렬로 부하 분산

```typescript
const completion = await openRouter.chat.send({
  model: 'meta-llama/llama-3.3-70b-instruct',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    sort: 'throughput',
  },
  stream: false,
});
```

### 3. 사용량 모니터링

정기적으로 `/api/v1/key` 엔드포인트를 호출하여 크레딧 잔액과 사용량을 확인하세요.

---

## 주의사항

1. **추가 계정 무효**: 추가 계정이나 API 키를 생성해도 속도 제한이 증가하지 않습니다
2. **글로벌 관리**: 속도 제한은 전역 용량에 기반하여 관리됩니다
3. **크레딧 유지**: 음수 잔액을 방지하기 위해 크레딧을 충분히 유지하세요

---

## 관련 문서

- [API 레퍼런스](./02-api-reference.md)
- [모델 라우팅](./07-model-routing.md)
- [Provider Selection](./08-provider-selection.md)
- [개인정보 및 데이터 처리](./15-privacy.md)
