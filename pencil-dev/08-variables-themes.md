# Pencil Dev 변수와 테마

> 원문: https://docs.pencil.dev/core-concepts/variables, Pencil MCP 서버 변수 시스템

Pencil의 변수 시스템은 CSS 커스텀 프로퍼티나 디자인 토큰과 유사하게 작동합니다. HEX 코드(색상), 숫자(간격, 보더 반경, 크기), 문자열(글꼴 이름) 등을 한 번 정의하면 디자인 전체에서 재사용할 수 있으며, 변수 패널에서 값을 변경하면 해당 변수를 사용하는 모든 요소가 일괄 업데이트됩니다.

---

## 변수 기본

### 변수 타입

| 타입 | 값 형식 | 사용 예 |
| --- | --- | --- |
| `color` | `#RGB`, `#RRGGBB`, `#RRGGBBAA` | 배경색, 텍스트 색상 |
| `number` | 숫자 | 간격, 크기, 반경 |
| `string` | 문자열 | 글꼴 이름 |
| `boolean` | `true` / `false` | 표시/숨김, 활성/비활성 |

### 변수 정의

Document의 `variables` 필드에 정의:

```javascript
variables: {
  "primary": { type: "color", value: "#3B82F6" },
  "background": { type: "color", value: "#FFFFFF" },
  "foreground": { type: "color", value: "#000000" },
  "spacing-unit": { type: "number", value: 16 },
  "font-primary": { type: "string", value: "Inter" },
  "dark-mode": { type: "boolean", value: false }
}
```

### 변수 참조

속성에서 `$` 접두사로 변수를 참조:

```javascript
// fill에 색상 변수 참조
{ fill: "$primary" }

// gap에 숫자 변수 참조
{ gap: "$spacing-unit" }

// fontFamily에 문자열 변수 참조
{ fontFamily: "$font-primary" }
```

| 규칙 | 설명 |
| --- | --- |
| 참조 형식 | `"$variable-name"` ($ 접두사) |
| 변수명 | `$`로 시작하면 안 됨 (정의 시). `[^:]+` 정규식 형식 |
| 바인딩 | 속성이 변수를 참조하면 해당 변수의 값으로 자동 바인딩 |

---

## 변수 생성 방법

Pencil에서 변수를 생성하는 세 가지 방법이 있습니다.

### 수동 생성 (Manually)

Pencil 내에서 직접 변수를 정의하고 테마별 값을 설정합니다. 툴바의 변수 아이콘을 클릭하여 변수 패널을 엽니다.

| 항목 | 설명 |
| --- | --- |
| 접근 방법 | 툴바의 **변수 아이콘** 클릭 → 변수 패널 열기 |
| 정의 방식 | 변수명, 타입, 기본값을 직접 입력 |
| 테마 설정 | 동일 변수에 대해 테마별 값을 개별 지정 가능 |

### CSS에서 가져오기 (From CSS)

AI 에이전트에게 `globals.css` 파일을 기반으로 변수를 생성하도록 요청합니다. AI가 색상, 간격, 글꼴 값을 자동으로 추출합니다.

```javascript
// AI 에이전트에 프롬프트 예시
// "globals.css에서 색상, 간격, 글꼴 변수를 추출해서 Pencil 변수로 만들어줘"
```

| 항목 | 설명 |
| --- | --- |
| 입력 | `globals.css` 파일의 CSS 커스텀 프로퍼티 |
| 자동 추출 항목 | 색상(colors), 간격(spacing), 글꼴(fonts) |
| 동작 | AI 에이전트가 CSS 변수를 Pencil 변수로 자동 변환 |

### Figma에서 가져오기 (From Figma)

Figma의 변수 테이블 스크린샷을 붙여넣고 AI에게 Pencil에 설정해 달라고 요청합니다. 개별 토큰 값을 직접 복사하여 붙여넣을 수도 있습니다.

| 항목 | 설명 |
| --- | --- |
| 방법 1 | 변수 테이블 스크린샷을 붙여넣고 AI에게 설정 요청 |
| 방법 2 | Figma에서 개별 토큰 값을 직접 복사/붙여넣기 |

---

## UI에서 변수 및 테마 다루기

### 변수 패널 열기

툴바의 **변수 아이콘**을 클릭하면 변수 패널이 열립니다. 여기서 변수를 확인, 생성, 수정할 수 있습니다.

| 조작 | 위치 | 설명 |
| --- | --- | --- |
| 변수 패널 열기 | **툴바** → 변수 아이콘 | 변수 목록 확인, 생성, 수정 |
| 테마 전환 | **속성 패널** (Properties panel) | light/dark 등 테마 간 전환하여 디자인 변화 확인 |
| 테마 열 추가 | **변수 패널** | 새 열(column)을 추가하여 light, dark 등 테마 생성 |

### 변수를 요소에 적용하기

하드코딩된 값 대신 변수를 참조하면, 변수가 변경될 때 해당 변수를 사용하는 모든 요소가 자동으로 업데이트됩니다.

```javascript
// 하드코딩 (권장하지 않음)
{ fill: "#3B82F6" }

// 변수 참조 (권장)
{ fill: "$primary" }
```

---

## 테마 시스템

### 테마 축 정의

Document의 `themes` 필드에 축(axis)을 정의:

```javascript
themes: {
  "mode": ["light", "dark"],
  "device": ["desktop", "tablet", "phone"]
}
```

### 테마별 변수 값

변수의 `value`를 배열로 지정해 테마별 값을 설정:

```javascript
"background": {
  type: "color",
  value: [
    { value: "#FFFFFF", theme: { mode: "light" } },
    { value: "#1A1A1A", theme: { mode: "dark" } }
  ]
},
"spacing-base": {
  type: "number",
  value: [
    { value: 16, theme: { device: "desktop" } },
    { value: 12, theme: { device: "tablet" } },
    { value: 8, theme: { device: "phone" } }
  ]
}
```

### 노드에서 테마 설정

개별 노드의 `theme` 속성으로 해당 노드(및 하위)의 테마 값을 설정:

```javascript
// 다크 모드 화면
darkScreen = Insert(document, {
  type: "frame",
  theme: { mode: "dark" },
  width: 1440,
  height: 900,
  fill: "$background"    // 다크 모드의 background 값 사용
})
```

---

## 코드와의 양방향 동기화 (Sync with Code)

Pencil과 코드베이스 간에 변수를 동기화할 수 있습니다. AI 어시스턴트에게 Pencil 파일을 기반으로 CSS 변수를 업데이트하거나, 반대로 CSS 변경 사항을 Pencil로 다시 가져오도록 요청할 수 있습니다. 이를 통해 **양방향 디자인-코드 워크플로우**가 가능해집니다.

### Pencil → CSS 동기화

Pencil 파일에서 정의한 변수를 CSS 커스텀 프로퍼티로 내보냅니다.

```javascript
// AI 어시스턴트에 프롬프트 예시
// "Pencil 파일의 변수를 기반으로 CSS 변수를 업데이트해줘"
```

```css
/* 자동 생성된 CSS */
:root {
  --color-primary: #3b82f6;
  --color-secondary: #8b5cf6;
  --spacing-base: 16px;
  --font-primary: Inter;
}
```

### CSS → Pencil 동기화

코드베이스에서 변경된 CSS 변수를 Pencil로 다시 가져옵니다.

```javascript
// AI 어시스턴트에 프롬프트 예시
// "globals.css에서 변경된 CSS 변수를 Pencil으로 가져와줘"
```

| 방향 | 설명 |
| --- | --- |
| Pencil → CSS | Pencil 파일의 변수를 기반으로 CSS 변수 업데이트 |
| CSS → Pencil | CSS 변경 사항을 Pencil 파일의 변수로 다시 가져오기 |

---

## MCP 도구로 변수 관리

### get_variables

문서에 정의된 변수와 테마를 조회:

```javascript
// MCP 도구 호출
get_variables({ filePath: "design.pen" })
```

### set_variables

변수를 업데이트. 기본적으로 기존 변수에 병합:

```javascript
set_variables({
  filePath: "design.pen",
  variables: {
    "accent": { type: "color", value: "#A3B59A" },
    "spacing-unit": { type: "number", value: 16 },
    "font-heading": { type: "string", value: "Playfair Display" }
  }
})
```

### 테마 변수 설정

```javascript
set_variables({
  filePath: "design.pen",
  variables: {
    "background": {
      type: "color",
      value: [
        { value: "#F8F5F0", theme: { mode: "light" } },
        { value: "#1A1A1A", theme: { mode: "dark" } }
      ]
    }
  }
})
```

### 전체 교체

`replace: true`로 기존 변수를 완전히 대체:

```javascript
set_variables({
  filePath: "design.pen",
  replace: true,
  variables: { /* 전체 새 변수 세트 */ }
})
```

---

## 디자인 토큰 컨벤션

### 색상 토큰

| 토큰 | 용도 |
| --- | --- |
| `$background` | 페이지 배경 |
| `$foreground` | 기본 텍스트 |
| `$muted-foreground` | 보조 텍스트, 플레이스홀더 |
| `$card` | 카드 배경 |
| `$border` | 보더, 구분선 |
| `$primary` | 기본 액션, 브랜드 |
| `$secondary` | 보조 요소 |
| `$destructive` | 위험 액션 |

### 시맨틱 색상

| 상태 | 배경 | 전경 |
| --- | --- | --- |
| 성공 | `$color-success` | `$color-success-foreground` |
| 경고 | `$color-warning` | `$color-warning-foreground` |
| 오류 | `$color-error` | `$color-error-foreground` |
| 정보 | `$color-info` | `$color-info-foreground` |

### 타이포그래피 토큰

| 토큰 | 용도 |
| --- | --- |
| `$font-primary` | 제목, 라벨, 네비게이션 |
| `$font-secondary` | 본문 텍스트, 설명, 입력 |

### 보더 반경 토큰

| 토큰 | 용도 |
| --- | --- |
| `$radius-none` | 테이블, 날카로운 컨테이너 |
| `$radius-m` | 카드, 모달 |
| `$radius-pill` | 버튼, 입력, 뱃지 |

---

## 변수 작성 시 주의사항

| 주의사항 | 설명 |
| --- | --- |
| 기존 변수 확인 | 새 변수를 만들 때 기존 변수를 덮어쓰지 않도록 주의 |
| 하드코딩 금지 | 색상, 간격 등은 항상 변수를 사용. `fill: "#3B82F6"` 대신 `fill: "$primary"` |
| CSS 변수와의 매핑 | 코드 생성 시 CSS 변수로 자동 변환됨 |

---

## CSS 변수로 변환

코드 생성 시 Pencil 변수는 CSS 커스텀 프로퍼티로 변환됩니다:

```css
:root {
  --color-primary: #3b82f6;
  --color-secondary: #8b5cf6;
  --spacing-base: 16px;
}
```

| Pencil | CSS | 비고 |
| --- | --- | --- |
| `$primary` | `var(--color-primary)` | 자동 매핑 |
| `$spacing-unit` | `var(--spacing-base)` | 자동 매핑 |
| `$font-primary` | CSS 클래스 `.font-primary` | 폰트는 변수 대신 클래스 사용 |
