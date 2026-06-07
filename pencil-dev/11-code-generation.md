# Pencil Dev 코드 생성

> 원문: https://docs.pencil.dev/design-and-code/design-to-code, Pencil MCP 서버 Code 가이드, Tailwind v4 가이드

Pencil은 .pen 디자인에서 React/TypeScript 컴포넌트와 Tailwind CSS 코드를 생성합니다. 디자인-코드 양방향 워크플로를 지원합니다.

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

## 컴포넌트 구현 워크플로

### Step 1: 컴포넌트 분석

#### 1A. 필요한 컴포넌트 식별

1. 대상 프레임/디자인을 `batch_get`으로 읽기
2. 해당 프레임에서 사용되는 `ref` 컴포넌트 식별
3. 각 컴포넌트의 사용 횟수 확인

#### 1B. 컴포넌트 정의 추출

1. `batch_get`으로 컴포넌트 구조 추출
2. 중첩 자식 포함 전체 트리 확보
3. **한 번에 하나씩** 처리:
   - 컴포넌트 추출 → React 재생성 → 검증 → 다음

#### 1C. 인스턴스 매핑

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

### Step 2: React 컴포넌트 생성

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

### Step 3: 검증

1. **시각 검증**: 디자인 컴포넌트 스크린샷과 React 컴포넌트 비교
2. **스타일 검증**: CSS 속성, 크기, 간격, 색상, 타이포그래피 일치 확인
3. **동작 검증**: fill_container 확장, fit_content 크기, 오버플로우 확인
4. **반복 수정**: 불일치 발견 시 즉시 수정 후 재검증

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
