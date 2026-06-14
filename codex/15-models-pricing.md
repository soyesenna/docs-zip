# Codex CLI - 모델 및 요금

> **원문**
> - Models: https://developers.openai.com/codex/models/
> - Pricing: https://developers.openai.com/codex/pricing/
> - Speed: https://developers.openai.com/codex/speed/
> - Feature Maturity: https://developers.openai.com/codex/feature-maturity/

> 최종 업데이트: 2026-06-15

---

## 목차

1. [지원 모델 목록](#1-지원-모델-목록)
2. [모델별 특징 및 권장 용도](#2-모델별-특징-및-권장-용도)
3. [Other models (Chat Completions API deprecated)](#3-other-models-chat-completions-api-deprecated)
4. [Fast mode](#4-fast-mode)
5. [Codex-Spark](#5-codex-spark)
6. [대체 모델 (Alternative models)](#6-대체-모델-alternative-models)
7. [요금제](#7-요금제)
8. [크레딧 레이트 카드](#8-크레딧-레이트-카드)
9. [플랜별 사용량 한도](#9-플랜별-사용량-한도)
10. [세션 중 모델 전환](#10-세션-중-모델-전환)
11. [Feature Maturity (기능 성숙도)](#11-feature-maturity-기능-성숙도)
12. [사용량 한도 연장 팁](#12-사용량-한도-연장-팁)

---

## 1. 지원 모델 목록

### Recommended models (권장 모델)

| 모델 | 유형 | CLI & SDK | App & IDE | Cloud | ChatGPT Credits | API Access |
|---|---|---|---|---|---|---|
| `gpt-5.4` | Flagship 프론티어 | O | O | O | O | O |
| `gpt-5.3-codex` | Industry-leading 코딩 | O | O | O | O | O |
| `gpt-5.3-codex-spark` | 실시간 코딩 (research preview) | O | O | O | O | -- |

> Codex의 대부분의 작업은 `gpt-5.4`에서 시작하는 것을 권장한다. `gpt-5.4`는 강력한 코딩, 추론, native computer use, 그리고 더 넓은 범위의 프로페셔널 워크플로우를 하나의 모델로 결합한다.

> 사이드바 "Latest" 표시 모델: GPT-5.4. `config.toml`의 기본값은 `model = "gpt-5.4"`.

---

## 2. 모델별 특징 및 권장 용도

### gpt-5.4

프로페셔널 작업용 플래그십 프론티어 모델이다. GPT-5.3-Codex의 industry-leading 코딩 능력에 더 강력한 추론, 도구 사용, 그리고 에이전트 워크플로우를 결합한 모델이다.

```bash
codex -m gpt-5.4
```

- **권장 용도**: 대부분의 Codex 작업의 시작점. 전문적인 코딩, 추론, native computer use, 도구 사용이 많은 에이전트 워크플로우
- **특징**: 강력한 코딩/추론/도구 사용/에이전트 워크플로우를 하나로 결합
- **Codex 인증**: ChatGPT 로그인과 API Key 인증 모두 지원. ChatGPT 로그인 시 ChatGPT Credits로 사용량이 계산되고, API Key 인증 시 표준 API 가격이 적용된다(Fast mode 사용 불가).

### gpt-5.3-codex

복잡한 소프트웨어 엔지니어링을 위한 industry-leading 코딩 모델이다. 이 코딩 능력은 현재 GPT-5.4에도 동일하게 적용된다.

```bash
codex -m gpt-5.3-codex
```

- **권장 용도**: 복잡한 소프트웨어 엔지니어링, 고난도 코딩 작업
- **특징**: GPT-5.4의 코딩 능력을 동일하게 탑재한 코딩 특화 모델

### gpt-5.3-codex-spark

텍스트 전용 research preview 모델로, 거의 즉각적인 실시간 코딩 반복(near-instant, real-time coding iteration)에 최적화되어 있다. ChatGPT Pro 사용자만 사용할 수 있다.

```bash
codex -m gpt-5.3-codex-spark
```

- **권장 용도**: 실시간 코딩 반복, 빠른 피드백 루프
- **특징**: 텍스트 전용, 초고속 응답, ChatGPT Pro 전용, 전용 저지연 하드웨어
- **제한**: research preview 상태로, 수요에 따라 사용량 한도가 조정될 수 있음. 런칭 시점에 API 미지원

---

## 3. Other models (Chat Completions API deprecated)

Codex는 위에 나열한 모델들과 함께 가장 잘 동작한다. 그 외에도 특정 사용 사례에 맞게 **Chat Completions** 또는 **Responses API**를 지원하는 모든 모델 및 provider를 Codex에 지정할 수 있다.

> **참고**: Chat Completions API 지원은 **deprecated** 상태이며, 향후 Codex 릴리스에서 제거될 예정이다.

## 4. Fast mode

Fast mode는 지원 모델의 속도를 높이는 기능으로, 크레딧 소비율이 증가한다. 모델 자체는 변경하지 않고 동일한 모델을 더 빠르게 실행한다.

### 지원 모델 및 효과

| 모델 | 속도 향상 | 크레딧 소비율 (Standard 대비) |
|---|---|---|
| `gpt-5.4` | 1.5x | 2.0x |

> Fast mode는 현재 **GPT-5.4**에서만 지원된다. 활성화 시 속도가 1.5배 증가하고, 크레딧이 2배율로 소비된다.

### 활성화 및 가용성

- 활성화 명령어: `/fast`
- 지원 환경: Codex IDE extension, Codex CLI, Codex app — ChatGPT 로그인 시 사용 가능
- **API Key 인증에서는 Fast mode를 사용할 수 없다.** API Key 인증 시 표준 API 가격이 적용된다.

> Fast mode뿐 아니라 모든 속도 설정(speed configurations)은 해당 모델의 크레딧 소비를 증가시키며, 포함 사용량 한도(included limits)도 더 빨리 소진된다.

---

## 5. Codex-Spark

GPT-5.3-Codex-Spark는 Fast mode와 달리 **별도의 모델**이다. Fast mode가 GPT-5.4의 속도를 더 높은 크레딧 소비율로 끌어올리는 기능이라면, Codex-Spark는 자체적인 모델 선택이며 자체 사용량 한도를 가진다.

| 항목 | 내용 |
|---|---|
| 모델명 | `gpt-5.3-codex-spark` |
| 유형 | 텍스트 전용 (research preview) |
| 최적화 | 거의 즉각적인 실시간 코딩 반복 (near-instant, real-time coding iteration) |
| 가용성 | ChatGPT Pro 구독자 전용 |
| API | 런칭 시점에 API 미지원 |
| 하드웨어 | 전문 저지연 하드웨어(specialized low-latency hardware)에서 실행 |
| 사용량 | 수요에 따라 조정될 수 있는 별도 한도 적용 |

> research preview 기간 중 Codex-Spark는 ChatGPT Pro 사용자 전용이며, 런칭 시점에 API에서는 사용할 수 없다. 전문 저지연 하드웨어에서 실행되므로 사용량은 수요에 따라 조정될 수 있는 별도의 한도로 관리된다.

---

## 6. 대체 모델 (Alternative models)

권장 모델 외에, 이전 세대 모델들을 대체(alternative) 모델로 사용할 수 있다. 각 모델은 더 최신 모델로 계슡(succeeded)되었다.

| 모델 | 설명 | 계승 모델 |
|---|---|---|
| `gpt-5.2-codex` | 실제 엔지니어링을 위한 고급 코딩 모델 | GPT-5.3-Codex |
| `gpt-5.2` | 산업/도메인 전반의 코딩·에이전트 작업용 이전 범용 모델 | GPT-5.4 |
| `gpt-5.1-codex-max` | 장기 실행·에이전트 코딩 작업에 최적화 | -- |
| `gpt-5.1` | 도메인 전반의 코딩·에이전트 작업에 적합 | GPT-5.2 |
| `gpt-5.1-codex` | 장기 실행·에이전트 코딩 작업에 최적화 | GPT-5.1-Codex-Max |
| `gpt-5-codex` | 장기 실행·에이전트 코딩 작업에 튜닝된 GPT-5 버전 | GPT-5.1-Codex |
| `gpt-5-codex-mini` | GPT-5-Codex의 더 작고 비용 효율적인 버전 | GPT-5.1-Codex-Mini |
| `gpt-5` | 도메인 전반의 코딩·에이전트 작업용 추론 모델 | GPT-5.1 |

> 크레딧 평균 비용은 권장 모델과 동일하게 legacy 모델(GPT-5.2, GPT-5.2-Codex, GPT-5.1, GPT-5.1-Codex-Max, GPT-5, GPT-5-Codex, GPT-5-Codex-Mini)에도 동일하게 적용된다(자세한 내용은 섹션 8 참고).

---

## 7. 요금제

Codex는 ChatGPT **Plus, Pro, Business, Edu, Enterprise** 플랜에 포함되어 있다. 한정 기간 동안 **ChatGPT Free 및 Go에서도 무료로 Codex를 체험**할 수 있으며, **Plus, Pro, Business, Enterprise 구독 시 2배의 Codex rate limit**을 제공한다.

### 플랜 개요

| 플랜 | 요금 | 설명 |
|---|---|---|
| **Free** | $0 | 한정 기간 동안 Codex 무료 체험 |
| **Go** | $8/month | 한정 기간 동안 Codex 무료 체험 |
| **Plus** | $20/month | 주당 몇 차례 집중 코딩 세션 운영 |
| **Pro** | $100/month~ | Codex 기반 일일 풀타임 개발 |
| **Business** | 종량제 | 스타트업 및 성장 기업용 |
| **Enterprise & Edu** | 영업팀 문의 | 조직 전체를 위한 엔터프라이즈급 기능 |
| **API Key** | 토큰 사용량 기반 | CI 등 공유 환경의 자동화에 적합 |

### Plus 플랜 세부 내용

- Codex web, CLI, IDE extension, iOS에서 사용
- 자동 코드 리뷰, Slack 연동 등 클라우드 기반 통합
- 최신 모델 포함: **GPT-5.4**, **GPT-5.3-Codex**
- **GPT-5.1-Codex-Mini**로 로컬 메시지 최대 **4배** 높은 사용량 한도
- ChatGPT credits로 유연하게 사용량 확장
- Plus 플랜의 다른 ChatGPT 기능 포함

### Pro 플랜 세부 내용

- 우선 요청 처리(priority request processing)
- **GPT-5.3-Codex-Spark (research preview)** 접근 권한 — 일상적 코딩 작업을 위한 빠른 Codex 모델
- 로컬·클라우드 작업에 대해 **6배** 높은 사용량 한도
- 클라우드 기반 코드 리뷰 **10배** 더 많음
- Pro 플랜의 다른 ChatGPT 기능 포함

### Business 플랜 세부 내용

- 클라우드 작업을 더 빠르게 실행하는 **더 큰 가상 머신**
- ChatGPT credits로 유연한 사용량 확장
- SAML SSO, MFA를 갖춘 안전한 전용 워크스페이스 및 필수 관리 제어
- 비즈니스 데이터에 대한 기본 학습 미사용
- Business 플랜의 다른 ChatGPT 기능 포함

### Enterprise & Edu 플랜 세부 내용

Business의 모든 기능과 더불어:

- 우선 요청 처리
- SCIM, EKM, 사용자 분석, 도메인 인증, RBAC 포함 엔터프라이즈급 보안 및 제어
- Compliance API를 통한 감사 로그 및 사용량 모니터링
- 데이터 보존 및 데이터 거주지 제어
- Enterprise 플랜의 다른 ChatGPT 기능 포함

### API Key 플랜 세부 내용

- Codex CLI, SDK, IDE extension에서 사용
- 클라우드 기반 기능 미포함 (GitHub 코드 리뷰, Slack 등)
- **신규 모델(GPT-5.3-Codex, GPT-5.3-Codex-Spark 등)에 대한 접근은 지연(Delayed access)**: ChatGPT 플랜보다 늦게 API Key에 공급되며, 가용 모델은 해당 API Key에 할당된 API 모델을 따른다.
- 사용한 토큰에 대해서만 API 가격으로 결제

---

## 8. 크레딧 레이트 카드

크레딧은 포함 사용량 한도에 도달한 이후에도 Codex를 계속 사용할 수 있게 해준다. 사용량은 사용하는 모델·기능에 따라 사용 가능한 크레딧에서 차감된다. 메시지당 크레딧 비용은 모델, 작업 크기·복잡도, 필요한 추론에 따라 달라진다.

### GPT-5.4 — 메시지당 평균 크레딧 비용

| 항목 | 단위 | 평균 크레딧 비용 |
|---|---|---|
| Local Tasks | 1 메시지 | ~7 credits |
| Cloud Tasks | 1 메시지 | ~34 credits |
| Code Review | 1 pull request | ~34 credits |

> 위 평균은 legacy 모델(GPT-5.2, GPT-5.2-Codex, GPT-5.1, GPT-5.1-Codex-Max, GPT-5, GPT-5-Codex, GPT-5-Codex-Mini)에도 동일하게 적용된다. 평균 비율은 새 기능이 도입됨에 따라 시간이 지나며 변할 수 있다.

> 현재 공식 Pricing 페이지의 rate card 섹션에 명시된 크레딧 비용 항목은 **GPT-5.4**(Local Tasks ~7 / Cloud Tasks ~34 / Code Review ~34 credits)뿐이다. **GPT-5.3-Codex-Spark**는 research preview로서 credit cost 항목이 아니며, **GPT-Image-2**는 Pricing 페이지에 언급되지 않는다.

### 토큰 기반 rate card (Help Center)

> 2026-04-02 이후 Plus, Pro, Business, Enterprise/Edu/Gov/Health 플랜은 메시지당 평균이 아닌 **토큰 사용량 기반(token-based pricing)** rate card로 마이그레이션되었다. 아래 토큰 기반 rate card는 Pricing 페이지가 아닌 **OpenAI Help Center의 별도 문서**에 게시되어 있다.

> 출처: https://help.openai.com/en/articles/20001106-codex-rate-card

| 모델 | 입력 토큰 (credits/1M) | 캐시된 입력 토큰 (credits/1M) | 출력 토큰 (credits/1M) |
|---|---|---|---|
| GPT-5.5 | 125 | 12.50 | 750 |
| GPT-5.5 Cyber | 500 | 50 | 3,000 |
| GPT-5.4 | 62.50 | 6.250 | 375 |
| GPT-5.4-Mini | 18.75 | 1.875 | 113 |
| GPT-5.3-Codex | 43.75 | 4.375 | 350 |
| GPT-5.2 | 43.75 | 4.375 | 350 |
| GPT-5.3-Codex-Spark | _research preview_ | _research preview_ | _research preview_ |
| GPT-Image-2.0 (image) | 200 | 50 | 750 |
| GPT-Image-2.0 (text) | 125 | 31.25 | 250 |

> Help Center rate card는 위 표의 모델을 게시하며, 그중 **GPT-5.5 Cyber**는 OpenAI Daybreak/Trusted Access for Cyber 프로그램의 일부로 제공된다. **GPT-5.3-Codex-Spark**의 credit rate는 research preview 상태로 확정되지 않았다. Fast mode는 지원 모델에 대해 더 높은 비율로 크레딧을 소비한다(Speed 문서 참고). Code review는 GPT-5.3-Codex를 사용한다.

### 참고 사항

- 속도 설정(speed configurations)은 해당 모델의 크레딧 소비를 증가시키며, 포함 사용량 한도도 더 빨리 소진한다. 자세한 내용은 섹션 4(Fast mode)를 참고.
- **Cloud Tasks 및 Code Reviews는 현재 모든 플랜·모델에서 Not available** 상태다.
- 이미지 생성은 유사한 턴보다 평균적으로 포함 한도를 더 빨리 소비한다.

### Fast mode 크레딧 배율

| 모델 | Standard 대비 크레딧 소비율 |
|---|---|
| GPT-5.4 | 2.0x |

---

## 9. 플랜별 사용량 한도

보낼 수 있는 Codex 메시지 수는 사용하는 모델, 작업 크기·복잡도, 로컬/클라우드 실행 여부에 따라 달라진다. 작은 스크립트나 루틴 함수는 한도의 일부만 소비하지만, 더 큰 코드베이스·장기 실행 작업·많은 컨텍스트가 필요한 세션은 메시지당 훨씬 더 많이 소비한다.

### GPT-5.4 — 사용량 한도

|  | Local Messages* / 5h | Cloud Tasks* / 5h | Code Reviews / week |
|---|---|---|---|
| ChatGPT Plus | 33-168 | Not available | Not available |
| ChatGPT Pro | 223-1120 | Not available | Not available |
| ChatGPT Business | 33-168 | Not available | Not available |
| ChatGPT Enterprise & Edu | 고정 한도 없음 — 사용량이 크레딧에 따라 확장 | | |
| API Key | Usage-based | Not available | Not available |

> \* 로컬 메시지와 Cloud Tasks의 사용량 한도는 **5시간 윈도우**를 공유한다. 추가 주간 한도가 적용될 수 있다.

> 속도 설정(speed configurations)은 해당 모델의 크레딧 소비를 증가시키므로, 포함 사용량 한도도 더 빨리 소진한다.

> 유연한 가격제(flexible pricing)가 없는 Enterprise 및 Edu 플랜은 대부분의 기능에서 **Plus**와 동일한 좌석당 사용량 한도를 가진다.

> **GPT-5.3-Codex-Spark**는 ChatGPT Pro 사용자 전용 research preview이며, 런칭 시점에 API에서는 사용할 수 없다. 전문 저지연 하드웨어에서 실행되므로 사용량은 수요에 따라 조정될 수 있는 **별도의 사용량 한도**로 관리된다.

### 사용량 한도 도달 시

- **Plus/Pro** 사용자가 한도에 도달하면, 플랜을 업그레이드하지 않고도 추가 크레딧을 구매해 계속 작업할 수 있다.
- **Business/Edu/Enterprise**(유연한 가격제)는 워크스페이스 크레딧을 추가 구매해 Codex를 계속 사용할 수 있다.
- 한도에 근접한 경우 **GPT-5.1-Codex-Mini** 모델로 전환하면 사용량 한도를 더 오래 유지할 수 있다.
- 모든 사용자는 API Key로 추가 로컬 작업을 실행할 수 있으며, 사용량은 표준 API 요금으로 청구된다.

### 기능 가용성

| 기능 | Plus | Pro | Business | Enterprise / Edu | API Key |
|---|---|---|---|---|---|
| **접근 및 서비스** | | | | | |
| Codex web | O | O | O | O | -- |
| Codex app (로컬 작업) | O | O | O | O | O |
| Codex CLI | O | O | O | O | O |
| IDE extension | O | O | O | O | O |
| Codex SDK / `codex exec` | O | O | O | O | O |
| 자동화 액세스 토큰 | -- | -- | O | O | -- |
| **모델 및 멀티모달** | | | | | |
| Fast mode | O | O | O | O | -- |
| Codex-Spark research preview | -- | O | -- | -- | -- |
| 이미지 생성 및 편집 | O | O | O | O | O |
| 음성 받아쓰기 | O | O | O | O | -- |
| 웹 검색 | O | O | O | O | O |
| **로컬 기능** | | | | | |
| `/review` 로컬 코드 리뷰 | O | O | O | O | O |
| 승인 요청 자동 리뷰 | O | O | O | O | O |
| 샌드박싱 및 권한 제어 | O | O | O | O | O |
| 프로젝트 자동화 | O | O | O | O | O |
| Automations | O | O | O | O | O |
| Worktrees 및 내장 Git 도구 | O | O | O | O | O |
| 로컬 환경 및 반복 가능한 작업 | O | O | O | O | O |
| Appshots | O | O | O | -- | O |
| **브라우저 및 원격 제어** | | | | | |
| 인앱 브라우저 미리보기 | O | O | O | O | O |
| Browser Use 자동화 | 제한* | 제한* | 제한* | 제한* | 제한* |
| Chrome extension 브라우저 제어 | 제한* | 제한* | 제한* | 제한* | 제한* |
| Computer Use | 제한* | 제한* | 제한* | 제한* | 제한* |
| SSH 원격 연결 | O | O | O | O | O |
| 모바일 원격 제어 | O | O | O | O | -- |
| **커스터마이징 및 확장** | | | | | |
| `AGENTS.md` 커스텀 명령 | O | O | O | O | O |
| Skills | O | O | O | O | O |
| Plugins | O | O | O | O | 제한* |
| Plugin 공유 | O | O | O | O | -- |
| App connectors | O | O | O | O | -- |
| MCP | O | O | O | O | O |
| Subagents 및 커스텀 에이전트 | O | O | O | O | O |
| Memories | 제한* | 제한* | 제한* | 제한* | 제한* |
| Chronicle | -- | 제한* | -- | -- | -- |
| **클라우드 및 통합** | | | | | |
| Codex cloud tasks | O | O | O | O | -- |
| Cloud 환경 및 설정 스크립트 | O | O | O | O | -- |
| Cloud 에이전트 인터넷 액세스 제어 | O | O | O | O | -- |
| GitHub `@codex` 이슈/PR 위임 | O | O | O | O | -- |
| GitHub 코드 리뷰 및 자동 PR 리뷰 | O | O | O | O | -- |
| Slack 클라우드 통합 | O | O | O | O | -- |
| Linear 클라우드 통합 | O | O | O | O | -- |
| **관리, 보안 및 분석** | | | | | |
| SAML SSO, MFA, 워크스페이스 관리 | -- | -- | O | O | -- |
| `requirements.toml` 관리형 구성 | O | O | O | O | O |
| Cloud 관리 구성 정책 | -- | -- | O | O | -- |
| Codex RBAC 및 커스텀 역할 | -- | -- | -- | O | -- |
| SCIM, EKM, 도메인 인증 | -- | -- | -- | O | -- |
| 엔터프라이즈 보존 및 거주지 제어 | -- | -- | -- | O | -- |
| API/비즈니스 데이터 학습 미사용 | -- | -- | O | O | O |
| Analytics 대시보드 | -- | -- | -- | O | -- |
| Analytics API | -- | -- | -- | O | -- |
| Compliance API 및 감사 로그 | -- | -- | -- | O | -- |
| Codex Security (GitHub 연결) | -- | -- | -- | O | -- |

> \* 제한* 표시는 특정 지역으로 제한되는 기능이다. 개별 기능 문서에서 지리적 제한에 대한 자세한 내용을 확인하라. Plugins의 경우 일부 자사(first-party) 플러그인이 API Key에서 사용 불가하다.

---

## 10. 세션 중 모델 전환

### `/model` 명령어

CLI에서 활성 스레드 중에 모델을 변경할 수 있다.

```
/model gpt-5.4
```

### `--model` / `-m` 플래그

새 CLI 스레드를 특정 모델로 시작하거나 `codex exec`의 모델을 지정한다.

```bash
codex -m gpt-5.4
codex exec --model gpt-5.4 "lint 오류를 수정해줘"
```

### config.toml 기본 모델 설정

```toml
model = "gpt-5.4"
```

구성 파일에 `model` 항목을 추가하여 기본 모델을 지정한다. 지정하지 않으면 Codex가 권장 모델을 기본값으로 사용한다.

### IDE Extension에서 모델 선택

IDE extension에서는 입력 상자 아래의 모델 선택기를 사용하여 모델을 선택할 수 있다.

### Cloud Tasks의 모델

현재 Codex Cloud Tasks에서는 기본 모델을 변경할 수 없다.

---

## 11. Feature Maturity (기능 성숙도)

일부 Codex 기능은 maturity(성숙도) 레이블과 함께 제공된다. 각 기능이 얼마나 신뢰할 수 있는지, 무엇이 변경될 수 있는지, 어떤 수준의 지원을 기대할 수 있는지 이해할 수 있도록 돕는다.

| Maturity | 의미 | 가이던스 |
|---|---|---|
| **Under development** | 사용 준비가 되지 않음 | 사용하지 말 것 |
| **Experimental** | 불안정하며 OpenAI가 제거하거나 변경할 수 있음 | 본인 책임하에 사용 |
| **Beta** | 광범위한 테스트에 준비됨; 대부분 완료되었으나 일부는 사용자 피드백에 따라 변경될 수 있음 | 대부분의 평가 및 파일럿에 적합; 작은 변경 예상 |
| **Stable** | 완전히 지원되고 문서화되며 광범위한 사용 준비 완료; 동작과 구성이 시간이 지나도 일관되게 유지됨 | 프로덕션 사용에 안전; 제거는 일반적으로 deprecation 절차를 거침 |

> Codex-Spark는 현재 **research preview** 상태로, 위 maturity 체계와 별개의 레이블로 운영된다.

---

## 12. 사용량 한도 연장 팁

사용량 한도를 오래 유지하기 위한 방법:

- **프롬프트 크기 제어**: 불필요한 컨텍스트를 제거하고 정확한 지시만 포함
- **`AGENTS.md` 크기 축소**: 리포지토리 내에 중첩하여 주입 컨텍스트 양 제어
- **MCP 서버 수 제한**: 사용하지 않는 MCP 서버 비활성화
- **소형 모델로 전환**: 루틴 작업에 **GPT-5.1-Codex-Mini**를 사용하면 사용량 한도를 약 **4배** 연장 가능
