# Pencil Dev 코드 생성

> 원문: https://docs.pencil.dev/design-and-code/design-to-code
>
> Pencil은 .pen 디자인에서 React/TypeScript 컴포넌트와 Tailwind CSS 코드를 생성합니다.
> 디자인-코드 양방향 워크플로를 지원합니다.

---

## Design ↔ Code 개념

Pencil은 디자인과 코드 사이의 일관성을 유지하는 워크플로를 제공합니다:

| 방향 | 설명 |
| --- | --- |
| Design → Code | .pen 파일에서 React/Tailwind 컴포넌트 생성 |
| Code → Design | 기존 코드베이스에서 컴포넌트를 가져와 디자인에 반영 |
| 양방향 동기화 | 변경사항을 양쪽에 반영 |

---

## 코드 생성 핵심 원칙

| 원칙 | 설명 |
| --- | --- |
| 프로젝트 프레임워크 준수 | React 프로젝트면 React 코드 생성. Vue면 Vue |
| CSS 라이브러리 활용 | Tailwind 사용 중이면 Tailwind 클래스로 스타일링 |
| 버전 확인 | 설치된 버전에 맞는 API 사용 |
| 디자인 충실도 | 디자인의 텍스트 라벨, 아이콘, 간격을 동일하게 유지 |
| 기존 요소 우선 | 이미 코드베이스에 있는 요소는 새로 만들지 않고 업데이트 |
| 기능 보존 | 기존 컴포넌트 수정 시 기능이 깨지지 않게 |

---

## Design → Code 워크플로

### 기본 코드 내보내기

1. **Pencil에서 디자인** — 캔버스에서 화면, 레이아웃 또는 개별 UI 컴포넌트를 디자인
2. **.pen 파일 저장** — 프로젝트 워크스페이스에 .pen 파일 저장
3. **AI 채팅 열기** — `Cmd/Ctrl + K` 누름
4. **Pencil에 코드 생성 요청**

### 컴포넌트 구현 상세 워크플로

#### Step 1: 컴포넌트 분석

##### 1A. 필요한 컴포넌트 식별

1. 대상 프레임/디자인을 `batch_get`으로 읽기
2. 해당 프레임에서 사용되는 `ref` 컴포넌트 식별
3. 각 컴포넌트의 사용 횟수 확인

##### 1B. 컴포넌트 정의 추출

1. `batch_get`으로 컴포넌트 구조 추출
2. 중첩 자식 포함 전체 트리 확보
3. **한 번에 하나씩** 처리:
   - 컴포넌트 추출 → React 재생성 → 검증 → 다음

##### 1C. 인스턴스 매핑

각 인스턴스의 오버라이드를 문서화:

| 분석 항목 | 설명 |
| --- | --- |
| 인스턴스 ID/위치 | 어디에 있는지 |
| descendants 맵 | 어떤 속성이 오버라이드되는지 |
| 중첩 컴포넌트 | 항상 렌더링되는지 또는 조건부인지 |

**중첩 컴포넌트 결정 규칙:**

| 조건 | 결과 |
| --- | --- |
| 모든 인스턴스가 중첩 컴포넌트를 포함 | 필수(required) — 항상 렌더링 |
| 하나라도 오버라이드로 숨김 | 옵셔널(optional) — 조건부 렌더링 |

#### Step 2: React 컴포넌트 생성

```typescript
// src/components/Button.tsx
interface ButtonProps {
  label: string;
  variant?: "primary" | "secondary" | "outline";
  icon?: string;
  disabled?: boolean;
  onClick?: () => void;
}

export function Button({ label, variant = "primary", icon, disabled, onClick }: ButtonProps) {
  return (
    <button
      className={`flex items-center gap-2 px-4 py-2.5 rounded-lg font-medium text-sm
        ${variant === "primary" ? "bg-[var(--primary)] text-white" : ""}
        ${variant === "secondary" ? "bg-[var(--secondary)] text-white" : ""}
        ${variant === "outline" ? "border border-[var(--border)] text-[var(--foreground)]" : ""}
        ${disabled ? "opacity-50 cursor-not-allowed" : "cursor-pointer"}`}
      disabled={disabled}
      onClick={onClick}
    >
      {icon && <span className="w-4 h-4">{icon}</span>}
      {label}
    </button>
  );
}
```

#### Step 3: 검증

1. **시각 검증**: 디자인 컴포넌트 스크린샷과 React 컴포넌트 비교
2. **스타일 검증**: CSS 속성, 크기, 간격, 색상, 타이포그래피 일치 확인
3. **동작 검증**: fill_container 확장, fit_content 크기, 오버플로우 확인
4. **반복 수정**: 불일치 발견 시 즉시 수정 후 재검증

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

### 구체적 워크플로 단계

| 단계 | 방향 | 설명 |
| --- | --- | --- |
| 1. 기존 코드 가져오기 | Code → Design | 기존 컴포넌트를 Pencil로 임포트 |
| 2. 디자인 개선 | Design 측 | Pencil에서 시각적 변경 수행 |
| 3. 코드 업데이트 | Design → Code | AI에 변경사항을 코드에 적용하도록 요청 |
| 4. 반복 | 양방향 | 필요에 따라 반복 |

```typescript
// 워크플로 예시: 헤더 컴포넌트 개선

// 1단계: 기존 Header 컴포넌트를 Pencil으로 임포트
// 프롬프트: "Import src/components/Header.tsx into this design"

// 2단계: Pencil에서 레이아웃 조정, 색상 변경, 간격 수정

// 3단계: 변경사항을 코드에 동기화
// 프롬프트: "Update src/components/Header.tsx to match the updated design"

// 4단계: 필요시 반복
```

---

## Example Prompts (예시 프롬프트)

### Design → Code — 컴포넌트 생성

| 목적 | 프롬프트 예시 |
| --- | --- |
| 단일 컴포넌트 | `Create a React component for this button` |
| 타입 생성 | `Generate TypeScript types for this form` |
| 재사용 컴포넌트 | `Export this card as a reusable component` |

### Design → Code — 전체 페이지

| 목적 | 프롬프트 예시 |
| --- | --- |
| Next.js 페이지 | `Generate a Next.js page from this design` |
| 랜딩 페이지 | `Create a landing page component with Tailwind CSS` |
| 대시보드 | `Export this dashboard as a React component` |

### Design → Code — 특정 라이브러리 사용

| 목적 | 프롬프트 예시 |
| --- | --- |
| shadcn/ui | `Generate code using Shadcn UI components` |
| React Hook Form | `Create this form using React Hook Form` |
| 아이콘 교체 | `Export using Lucide icons instead of Material Icons` |

---

## Tailwind CSS v4 구현

### globals.css 구조

```css
@import "tailwindcss";

:root {
  /* Pencil 디자인 변수를 CSS 커스텀 프로퍼티로 매핑 */
  --color-primary: #3b82f6;
  --color-secondary: #8b5cf6;
  --spacing-base: 16px;
  /* 폰트 스택은 저장하지 않음 */
}

@layer base {
  html, body {
    height: 100%;
  }

  /* 폰트 유틸리티 클래스 */
  .font-primary {
    font-family: "Inter", sans-serif;
  }

  .font-secondary {
    font-family: "JetBrains Mono", monospace;
  }
}
```

### Tailwind v4 주의사항

| 항목 | 올바름 | 잘못됨 |
| --- | --- | --- |
| 임포트 | `@import "tailwindcss";` | `@tailwind base; @tailwind components; @tailwind utilities;` |
| 폰트 | `@layer base` 유틸리티 클래스 | `font-[var(--font-name)]` |
| 리셋 | `@import`에 Preflight 자동 포함 | `* { margin: 0; }` 수동 리셋 |
| 테마 변수 | `:root` 블록에 저장 | `@theme`에 저장 |

### fill_container 변환

| Pencil | Tailwind | 조건 |
| --- | --- | --- |
| `fill_container` (flex 안) | `flex-1` | 부모가 flex 컨테이너 |
| `fill_container` (너비) | `w-full` | 명시적 크기 |
| `fill_container` (높이) | `h-full` | 부모 체인에 높이 설정 필요 |
| `fit_content` | `w-fit` / `h-fit` | — |

### 크기 지정 규칙

```typescript
// ✅ 올바름: Tailwind 클래스만 사용
<div className="w-[280px] h-[48px] p-4 bg-[var(--color-primary)] rounded-lg">

// ❌ 잘못됨: 인라인 스타일
<div style={{ width: 280, height: 48, padding: 16, backgroundColor: 'var(--color-primary)' }}>
```

---

## SVG Path 구현

### 추출 워크플로

1. `batch_get`에 `includePathGeometry: true`로 정확한 geometry 추출
2. `geometry` → SVG `<path>`의 `d` 속성으로 사용
3. `viewBox="0 0 {width} {height}"` 설정

```typescript
// Pencil에서 추출한 SVG
<svg className="w-6 h-6 fill-[var(--icon-primary)]" viewBox="0 0 24 24">
  <path d="M12 2L2 7l10 5 10-5-10-5z" />
</svg>
```

| 규칙 | 설명 |
| --- | --- |
| 정확한 geometry | 근사치 금지. `batch_get`에서 추출한 정확한 값 사용 |
| 변수 변환 | `$primary` → `var(--primary)` |
| Tailwind 스타일 | `fill-[var(--color)]`, `stroke-[var(--color)]` |
| 브랜드 에셋 | 복잡한 로고도 전체 geometry 유지 |

---

## Next.js 통합

### 폰트 로딩

```tsx
// layout.tsx
import { JetBrains_Mono } from "next/font/google";

const jetbrainsMono = JetBrains_Mono({
  variable: "--font-jetbrains-mono",
  subsets: ["latin"],
});

export default function RootLayout({ children }) {
  return (
    <html>
      <body className={jetbrainsMono.variable}>
        {children}
      </body>
    </html>
  );
}
```

```css
/* globals.css — ❌ next/font 변수를 :root에 재래핑 금지 */
@layer base {
  .font-primary {
    font-family: var(--font-jetbrains-mono), "JetBrains Mono", monospace;
  }
}
```

---

## 변수 매핑

Pencil 변수 → CSS 커스텀 프로퍼티 변환:

| Pencil 변수 | CSS | Tailwind 사용 |
| --- | --- | --- |
| `$primary` | `--color-primary` | `bg-[var(--color-primary)]` |
| `$text-secondary` | `--color-text-secondary` | `text-[var(--color-text-secondary)]` |
| `$spacing-unit` | `--spacing-base` | `gap-[var(--spacing-base)]` |
| `$font-primary` | CSS 클래스 `.font-primary` | `className="font-primary"` |

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
