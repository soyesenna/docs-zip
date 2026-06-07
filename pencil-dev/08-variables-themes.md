# Pencil Dev 변수와 테마

> 원문: https://docs.pencil.dev/core-concepts/variables, Pencil MCP 서버 변수 시스템

Pencil의 변수 시스템은 디자인 토큰을 관리하고 테마 변형(다크 모드, 반응형 등)을 지원합니다.

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
