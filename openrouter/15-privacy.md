# OpenRouter 개인정보 및 데이터 처리 가이드

> 원문: https://openrouter.ai/docs/guides/privacy

OpenRouter를 통해 AI를 사용할 때(채팅 인터페이스 또는 API) 프롬프트와 응답은 여러 터치포인트를 거칩니다. 각 단계에서 데이터가 어떻게 처리되는지에 대한 실용적인 개요를 제공합니다.

---

## OpenRouter 내부 데이터 처리

### 프롬프트 및 응답 저장

**OpenRouter는 프롬프트나 응답을 저장하지 않습니다.** 단, 계정 설정에서 프롬프트 로깅을 명시적으로 활성화(opt-in)한 경우는 예외입니다.

### 익명 프롬프트 분류

OpenRouter는 리포팅 및 모델 랭킹을 위해 소수의 프롬프트를 샘플링하여 분류합니다.

- 프롬프트 로깅에 동의하지 않은 경우, 프롬프트 분류는 **완전히 익명**으로 저장됩니다
- 분류 결과는 계정이나 사용자 ID와 **절대 연결되지 않습니다**
- 분류는 모델 단위로 수행되며, **Zero-Data-Retention(영데이터보존)** 정책을 따릅니다

### 메타데이터 저장

OpenRouter는 각 요청에 대해 다음 메타데이터를 저장합니다:

| 저장 항목 | 설명 |
| --- | --- |
| 프롬프트 토큰 수 | 입력 토큰 수량 |
| 완성 토큰 수 | 출력 토큰 수량 |
| 지연 시간 | 요청 처리 시간 |
| 모델 정보 | 사용된 모델 식별자 |
| 비용 | 크레딧 사용량 |

**중요**: 이 메타데이터에는 프롬프트나 응답의 **내용은 포함되지 않으며**, 요청 자체에 대한 정보만 포함됩니다.

용도:
- 리포팅 및 모델 랭킹
- 활동 피드 (Activity Feed)
- 성능 통계

---

## 데이터 정책 제어

### 요청 수준

`provider` 객체의 `data_collection` 필드로 데이터 저장 정책을 제어할 수 있습니다:

```typescript
const completion = await openRouter.chat.send({
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    dataCollection: 'deny',  // 데이터를 저장하지 않는 프로바이더만 사용
  },
  stream: false,
});
```

| 값 | 설명 |
| --- | --- |
| `"allow"` (기본값) | 사용자 데이터를 비일시적으로 저장하거나 학습에 사용할 수 있는 프로바이더 허용 |
| `"deny"` | 사용자 데이터를 수집하지 않는 프로바이더만 사용 |

### 계정 수준

개인정보 설정(Privacy Settings)에서 계정 전체에 데이터 정책을 설정할 수 있습니다:
- 서드파티 모델 프로바이더의 입력 데이터 학습 사용 여부 제어
- 특정 프로바이더 허용/차단

---

## Zero Data Retention (ZDR)

### 요청 수준 ZDR

```typescript
const completion = await openRouter.chat.send({
  model: 'gpt-4',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    zdr: true,  // ZDR 엔드포인트로만 라우팅
  },
  stream: false,
});
```

`zdr: true`를 설정하면 프롬프트를 보존하지 않는 엔드포인트로만 라우팅됩니다.

### 모델 그룹별 / 계정 수준 ZDR

ZDR은 다음 수준에서도 설정할 수 있습니다:
- **모델 그룹별**: Anthropic, OpenAI, Google 및 비프론티어 모델 그룹별로 개별 설정
- **계정 수준**: 개인정보 설정에서 전체 계정에 적용
- **가드레일**: 조직 가드레일을 통해 강제 적용

요청 수준 `zdr` 파라미터는 계정 수준 및 가드레일 ZDR 설정과 **OR** 관계로 작동합니다. 즉, 어느 하나라도 활성화되면 ZDR이 적용됩니다.

---

## 프롬프트 로깅

프롬프트 로깅은 기본적으로 **비활성화**되어 있습니다.

- 활성화: 계정 설정에서 명시적으로 opt-in
- 비활성화 시: 프롬프트와 응답 내용이 OpenRouter에 저장되지 않음
- 분류: 프롬프트 로깅 여부와 관계없이 익명 분류는 수행됨 (위 참조)

---

## EU 데이터 레지던시 (Enterprise)

OpenRouter는 엔터프라이즈 고객을 위해 EU 내 라우팅을 지원합니다. 활성화하면 프롬프트와 완성이 EU 내에서만 처리됩니다.

자세한 내용은 개인정보 문서를 참조하고, 엔터프라이즈팀에 문의하려면 이 양식을 작성하세요.

---

## Distillable Text Enforcement

텍스트 증류(distillation)를 허용하는 모델로만 라우팅을 제한할 수 있습니다:

```typescript
const completion = await openRouter.chat.send({
  model: 'meta-llama/llama-3.3-70b-instruct',
  messages: [{ role: 'user', content: 'Hello' }],
  provider: {
    enforceDistillableText: true,
  },
  stream: false,
});
```

이 설정은 모델 파인튜닝이나 증류 워크플로우를 위한 데이터셋 구축 시 유용합니다.

---

## 추가 정보

- **개인정보처리방침**: OpenRouter의 공식 개인정보처리방침
- **서비스 약관**: 각 프로바이더의 서비스 약관은 모델 페이지에서 확인 가능
- **데이터 정책 태그**: 모델 페이지에 Data Policy 태그로 표시됨

> **참고**: 데이터 정책 태그는 서드파티 데이터 정책의 결정적 출처가 아니며, OpenRouter의 최선 지식을 기반으로 합니다.

---

## 관련 문서

- [API 레퍼런스](./02-api-reference.md)
- [Provider Selection](./08-provider-selection.md)
- [속도 제한](./14-rate-limits.md)
