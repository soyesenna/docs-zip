# Pencil Dev 코드 생성

> 원문: https://docs.pencil.dev/design-and-code/design-to-code
>
> Pencil은 .pen 디자인에서 React/TypeScript 컴포넌트와 Tailwind CSS 코드를 생성합니다.
> 디자인-코드 양방향 워크플로를 지원합니다.

---

## 개요

Pencil은 디자인과 코드 사이의 양방향 워크플로를 제공합니다:

| 방향 | 설명 |
| --- | --- |
| Design → Code | Pencil 디자인에서 컴포넌트 생성 |
| Code → Design | 기존 코드베이스의 컴포넌트를 Pencil로 가져오기 |

---

## Design → Code 워크플로

### 기본 코드 내보내기

1. **Pencil에서 디자인** — 캔버스에서 화면, 레이아웃 또는 개별 UI 컴포넌트를 디자인
2. **.pen 파일 저장** — 프로젝트 워크스페이스에 `.pen` 파일 저장
3. **AI 채팅 열기** — `Cmd/Ctrl + K`
4. **Pencil에 코드 생성 요청**

### Example Prompts (예시 프롬프트)

#### Component generation (컴포넌트 생성)

| 목적 | 프롬프트 예시 |
| --- | --- |
| 단일 컴포넌트 | `Create a React component for this button` |
| 타입 생성 | `Generate TypeScript types for this form` |
| 재사용 컴포넌트 | `Export this card as a reusable component` |

#### Full pages (전체 페이지)

| 목적 | 프롬프트 예시 |
| --- | --- |
| Next.js 페이지 | `Generate a Next.js page from this design` |
| 랜딩 페이지 | `Create a landing page component with Tailwind CSS` |
| 대시보드 | `Export this dashboard as a React component` |

#### With specific libraries (특정 라이브러리 사용)

| 목적 | 프롬프트 예시 |
| --- | --- |
| shadcn/ui | `Generate code using Shadcn UI components` |
| React Hook Form | `Create this form using React Hook Form` |
| 아이콘 교체 | `Export using Lucide icons instead of Material Icons` |

---

## Code → Design 워크플로

기존 코드베이스에 컴포넌트가 있다면, Pencil이 이를 시각적으로 재현할 수 있습니다.

### 요구사항

| 항목 | 설명 |
| --- | --- |
| .pen 파일 위치 | .pen 파일을 코드와 같은 워크스페이스에 유지 |
| AI 에이전트 접근 | AI 에이전트가 디자인 파일과 코드 파일 모두에 접근 가능해야 함 |

### 워크플로 단계

1. **.pen 파일 열기**
2. **AI 채팅 열기** — `Cmd/Ctrl + K` 누름
3. **코드 가져오기 요청**

### 예시 프롬프트

```
Recreate the Button component from src/components/Button.tsx
Import the LoginForm from my codebase into this design
Add the Header component from src/layouts/Header.tsx
```

### 가져오기 결과

가져오기 시 다음 항목이 반영됩니다:

| 항목 | 설명 |
| --- | --- |
| 컴포넌트 구조 및 계층 | 컴포넌트의 전체 트리 구조 |
| 레이아웃 및 배치 | 위치, 정렬, 간격 |
| 스타일링 | 색상, 타이포그래피, 간격 |

---

## Two-Way Sync (양방향 동기화)

가장 강력한 워크플로는 양방향을 결합하는 것입니다:

1. **Start with code** — 기존 컴포넌트를 Pencil로 임포트
2. **Design improvements** — Pencil에서 시각적 변경 수행
3. **Update code** — AI에 변경사항을 코드에 적용하도록 요청
4. **Iterate** — 필요에 따라 반복

---

## Variables & Design Tokens 동기화

CSS 변수와 Pencil 변수 간 동기화된 디자인 토큰 시스템을 구축할 수 있습니다.

### Import CSS to Pencil (CSS → Pencil)

`globals.css` 또는 유사 파일에 CSS 변수가 정의되어 있어야 합니다.

**예시 프롬프트:**

```
Create Pencil variables from my globals.css
Import design tokens from src/styles/tokens.css
```

### Export Pencil to CSS (Pencil → CSS)

Pencil에서 변수를 정의한 뒤 CSS로 내보냅니다.

**예시 프롬프트:**

```
Update globals.css with these Pencil variables
Sync these design tokens to my CSS
```

---

## Best Practices

### File Organization (파일 구성)

.pen 파일을 리포지토리에 보관하면 AI 에이전트가 디자인과 코드를 모두 볼 수 있어 동기화가 용이합니다.

```
my-project/
├── src/
│   ├── components/
│   └── styles/
├── design.pen           ← 디자인 파일
└── package.json
```

**이점:**

| 이점 | 설명 |
| --- | --- |
| AI 에이전트 접근 | 디자인과 코드를 동시에 확인 가능 |
| 버전 관리 | 두 파일의 변경 이력을 함께 추적 |
| 동기화 용이 | 디자인-코드 간 쉽게 동기화 |

### Workflow Recommendations (워크플로 권장사항)

#### 새 기능 시작

| 단계 | 설명 |
| --- | --- |
| 1. Pencil에서 디자인 | 시각적 디자인 먼저 수행 |
| 2. 초기 코드 생성 | AI로 초기 컴포넌트 코드 생성 |
| 3. 코드 구현 다듬기 | 생성된 코드를 실제 프로젝트에 맞게 수정 |
| 4. 필요시 디자인 업데이트 | 코드 변경사항을 디자인에 반영 |

#### 기존 기능 업데이트

| 단계 | 설명 |
| --- | --- |
| 1. 컴포넌트 임포트 | 기존 컴포넌트를 Pencil으로 가져오기 |
| 2. 디자인 변경 | Pencil에서 시각적 수정 |
| 3. 코드 동기화 | 변경사항을 코드에 다시 반영 |

#### 디자인 시스템 유지관리

| 단계 | 설명 |
| --- | --- |
| 1. Pencil에서 변수 정의 | 색상, 간격, 타이포그래피 등 토큰 정의 |
| 2. CSS로 동기화 | Pencil 변수를 CSS 커스텀 프로퍼티로 동기화 |
| 3. 양쪽에서 변수 사용 | 디자인과 코드 모두에서 동일한 토큰 사용 |
| 4. 한 번 업데이트, 전체 반영 | 변수 수정 시 디자인과 코드에 동시 적용 |

---

## Popular Stacks & Libraries

Pencil은 특정 프레임워크에 제한되지 않습니다. AI에 원하는 스택을 지정하면 해당 기술에 맞는 코드를 생성합니다.

### Frameworks (프레임워크)

| 프레임워크 | 설명 |
| --- | --- |
| React | JavaScript 또는 TypeScript |
| Next.js | App Router, Pages Router 모두 지원 |
| Vue | Vue 3 + Composition API |
| Svelte | SvelteKit 포함 |
| HTML/CSS | 바닐라 마크업 |

### Styling (스타일링)

| 라이브러리 | 설명 |
| --- | --- |
| Tailwind CSS | 유틸리티 우선 CSS 프레임워크 |
| CSS Modules | 컴포넌트 스코프 CSS |
| Styled Components | CSS-in-JS |
| Plain CSS | 순수 CSS |

### Component Libraries (컴포넌트 라이브러리)

| 라이브러리 | 설명 |
| --- | --- |
| shadcn/ui | Radix 기반 복사-붙여넣기 컴포넌트 |
| Radix UI | 접근성 우선 헤드리스 UI |
| Chakra UI | 모듈형 UI 컴포넌트 |
| Material UI | Google Material Design 구현 |
| Custom | 자체 커스텀 컴포넌트 |

### 스택 지정 방법

프롬프트에 선호하는 기술을 언급하면 해당 프로젝트에 맞는 코드를 생성합니다:

```
Generate Next.js 14 code with Tailwind CSS
Create a Vue component using TypeScript
Use shadcn/ui components for this layout
```

---

## Icon Libraries

### Built-in vs Code Libraries (내장 vs 코드 라이브러리)

#### Pencil 내장 아이콘

Pencil은 다음 내장 아이콘 라이브러리를 포함합니다:

| 라이브러리 | 스타일 |
| --- | --- |
| Material Symbols | Outlined, Rounded, Sharp |
| Lucide Icons | — |
| Feather | — |
| Phosphor | — |

개별 SVG 아이콘을 이미지와 동일한 방식으로 가져올 수도 있습니다.

#### 코드 생성 시 아이콘 지정

| 항목 | 설명 |
| --- | --- |
| 프롬프트에서 지정 | 선호하는 아이콘 라이브러리를 프롬프트에 명시 |
| 주요 옵션 | Lucide, Heroicons, FontAwesome, React Icons |

**예시 프롬프트:**

```
Generate this design using Lucide icons
Replace Material Icons with Heroicons in the code
```
