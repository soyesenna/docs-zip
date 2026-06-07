# Pencil Dev 그래픽과 효과

> 원문: Pencil MCP 서버 .pen 파일 스키마 fill/effect 섹션 (get_editor_state로 조회)

.pen 파일은 색상, 그래디언트, 이미지, 셰이더, 메시 그래디언트 등 다양한 Fill 타입과 효과를 지원합니다.

---

## Fill 타입

Fill은 단일 값 또는 배열로 지정합니다. 여러 Fill을 겹칠 수 있습니다.

### 1. Color Fill

가장 기본적인 색상 채우기:

```javascript
// 단축형
{ fill: "#3B82F6" }

// 상세형
{ fill: { type: "color", color: "#3B82F6", enabled: true, blendMode: "normal" } }
```

### 2. Gradient Fill

선형, 방사형, 각도 그래디언트:

```javascript
{ fill: {
  type: "gradient",
  gradientType: "linear",       // "linear" | "radial" | "angular"
  opacity: 1.0,
  center: { x: 0.5, y: 0.5 },  // 정규화된 중심점
  size: { width: 1, height: 1 },
  rotation: 90,                  // 도 (반시계). 0°=위, 90°=왼쪽, 180°=아래
  colors: [
    { color: "#3B82F6", position: 0 },
    { color: "#8B5CF6", position: 1 }
  ]
}}
```

| 속성 | 설명 |
| --- | --- |
| `gradientType` | `"linear"`, `"radial"`, `"angular"` |
| `center` | 정규화된 중심 (기본값 0.5, 0.5) |
| `size` | 정규화된 크기. linear=height가 길이. radial/angular=지름 |
| `rotation` | 반시계 방향 도 단위 |
| `colors` | 색상 정지점 배열. position은 0~1 |

### 3. Image Fill

이미지로 채우기. URL은 .pen 파일 기준 상대 경로:

```javascript
{ fill: {
  type: "image",
  url: "./photo.jpg",           // .pen 파일 기준 상대 경로
  mode: "fill",                 // "stretch" | "fill" | "fit"
  opacity: 1.0
}}
```

| 모드 | 설명 |
| --- | --- |
| `"stretch"` | 이미지를 박스에 맞춰 늘림 |
| `"fill"` | 박스를 채우며 자름 (비율 유지) |
| `"fit"` | 박스 안에 맞춤 (여백 가능, 비율 유지) |

> 이미지를 표시하려면 먼저 Frame이나 Rectangle을 만들고, 그 위에 `Generate` 함수로 이미지를 적용하세요. `image` 노드 타입은 존재하지 않습니다.

### 4. Shader Fill

WebGL 1.0 프래그먼트 셰이더로 복잡한 그래픽 효과:

```javascript
{ fill: {
  type: "shader",
  url: "./effect.glsl",         // 셰이더 파일 (상대 경로)
  opacity: 1.0,
  uniforms: {
    "u_size": 32,
    "u_color1": "#ffffff",
    "u_offset": [0.5, 0.3]
  }
}}
```

| 속성 | 설명 |
| --- | --- |
| `url` | WebGL 1.0 (`#version 100`) 프래그먼트 셰이더 파일 |
| `uniforms` | 유니폼 오버라이드 값. `@resolution`/`@time`은 제외 |

### 5. Mesh Gradient Fill

베지어 보간된 컬러 그리드:

```javascript
{ fill: {
  type: "mesh_gradient",
  columns: 3,
  rows: 3,
  colors: ["#FF0000", "#00FF00", "#0000FF", ...],  // 9개 (3×3)
  points: [[0,0], [0.5,0], [1,0], ...],             // 9개 정규화된 위치
  opacity: 1.0
}}
```

---

## Fill 공통 속성

| 속성 | 타입 | 설명 |
| --- | --- | --- |
| `enabled` | `BooleanOrVariable` | 활성화 여부. 기본값 `true` |
| `blendMode` | `BlendMode` | 블렌드 모드 |
| `opacity` | `NumberOrVariable` | 불투명도 (Color Fill은 hex alpha로만 제어) |

### BlendMode

| 모드 | 설명 |
| --- | --- |
| `normal` | 기본값 |
| `darken` | 어두운 색 선택 |
| `multiply` | 곱하기 |
| `linearBurn` | 선형 번 (더 어두운 결과) |
| `colorBurn` | 컬러 번 (대비 강화) |
| `light` | 밝은 색 선택 |
| `screen` | 스크린 |
| `linearDodge` | 선형 닷지 (더 밝은 결과) |
| `colorDodge` | 컬러 닷지 (하이라이트 강화) |
| `overlay` | 오버레이 |
| `softLight` | 부드러운 빛 |
| `hardLight` | 강한 빛 |
| `difference` | 차이 |
| `exclusion` | 제외 |
| `hue` | 색상 |
| `saturation` | 채도 |
| `color` | 색 |
| `luminosity` | 명도 |

---

## Stroke (스트로크)

`stroke` 속성은 `Fills` 타입을 사용합니다. 즉, 단일 `Fill` 또는 `Fill` 배열을 지정할 수 있어 Color는 물론 Gradient, Image, Shader, Mesh Gradient 등 Fill의 모든 타입을 스트로크에 적용할 수 있습니다.

```javascript
// 단일 색상 스트로크
{
  stroke: "#000000",
  strokeWidth: 2,
  strokeLinecap: "round",
  strokeLinejoin: "miter",
  strokeAlignment: "center"
}

// 그래디언트 스트로크
{
  stroke: {
    type: "gradient",
    gradientType: "linear",
    colors: [
      { color: "#3B82F6", position: 0 },
      { color: "#8B5CF6", position: 1 }
    ]
  },
  strokeWidth: 2
}

// 다중 Fill 스트로크 (배열)
{
  stroke: [
    { type: "color", color: "#00000033" },
    { type: "gradient", gradientType: "linear", colors: [
      { color: "#FF0000", position: 0 },
      { color: "#0000FF", position: 1 }
    ]}
  ],
  strokeWidth: 3
}
```

| 속성 | 옵션 | 설명 |
| --- | --- | --- |
| `strokeWidth` | 숫자 또는 `{ top, right, bottom, left }` | 선 두께. 면별로 다르게 가능 |
| `strokeLinecap` | `"butt"`, `"round"`, `"square"` | 선 끝 모양 |
| `strokeLinejoin` | `"miter"`, `"bevel"`, `"round"` | 선 연결 모양 |
| `strokeAlignment` | `"inner"`, `"center"`, `"outer"` | 스트로크 위치 |

---

## Effect (효과)

### Blur

노드 전체를 블러 처리:

```javascript
{ effect: { type: "blur", radius: 10, enabled: true } }
```

### Background Blur

노드 뒤의 배경을 블러 처리:

```javascript
{ effect: { type: "background_blur", radius: 20, enabled: true } }
```

### Shadow

내부 또는 외부 드롭 섀도:

```javascript
{ effect: {
  type: "shadow",
  shadowType: "outer",           // "inner" | "outer"
  offset: { x: 0, y: 4 },
  spread: 0,
  blur: 12,
  color: "#00000033",
  blendMode: "normal",
  enabled: true
}}
```

| 속성 | 설명 |
| --- | --- |
| `shadowType` | `"inner"` (내부 섀도) 또는 `"outer"` (외부 섀도) |
| `offset` | `{ x, y }` 오프셋 |
| `spread` | 확산 |
| `blur` | 블러 반경 |
| `color` | 색상 (alpha 포함 가능) |

### 여러 효과 적용

Effect 배열로 여러 효과를 겹칠 수 있습니다:

```javascript
{ effect: [
  { type: "shadow", shadowType: "outer", offset: { x: 0, y: 4 }, blur: 12, color: "#00000020" },
  { type: "background_blur", radius: 8 }
]}
```

---

## SVG Path

Path 노드로 임의의 SVG 형상을 그립니다:

```javascript
{
  type: "path",
  geometry: "M10 10 L90 10 L90 90 L10 90 Z",
  viewBox: [0, 0, 100, 100],
  fill: "#3B82F6",
  width: 100,
  height: 100
}
```

| 규칙 | 설명 |
| --- | --- |
| `viewBox` | 항상 명시적으로 설정. `[x, y, width, height]` |
| `geometry` | SVG path 명령어 |
| 매핑 | viewBox 영역이 노드의 width/height에 스트레치됨 |

---

## 이미지 생성 (Generate)

`batch_design`의 `Generate` 함수로 AI 또는 스톡 이미지를 생성:

```javascript
// AI 생성 이미지
rect = Insert(page, { type: "rectangle", width: 400, height: 300 })
Generate(rect, "ai", "A modern dashboard interface with dark theme")

// 스톡 이미지 (Unsplash)
rect = Insert(page, { type: "rectangle", width: 400, height: 300 })
Generate(rect, "stock", "mountain landscape")
```

| 타입 | 프롬프트 | 설명 |
| --- | --- | --- |
| `"ai"` | 상세한 설명 프롬프트 | AI 생성 이미지 |
| `"stock"` | 1~3개 키워드 | Unsplash 스톡 이미지 |

> `image` 노드 타입은 없습니다. 이미지는 항상 Frame이나 Rectangle의 Fill로 적용됩니다.

---

## 아이콘

Icon 노드로 라이브러리 아이콘 표시. 6개 라이브러리를 지원합니다:

```javascript
// Lucide 아이콘
{ type: "icon", library: "lucide", icon: "settings", width: 24, height: 24, fill: "$foreground" }

// Material Symbols Rounded (weight 지정)
{ type: "icon", library: "Material Symbols Rounded", icon: "dashboard", width: 24, height: 24, fill: "$foreground", weight: 400 }
```

### 지원 라이브러리

| 라이브러리 | 스타일 | 예시 아이콘 |
| --- | --- | --- |
| `lucide` | 라인 아이콘, 균일한 선 두께 | `home`, `settings`, `search`, `plus`, `heart` |
| `feather` | 라인 아이콘, 심플한 디자인 | `home`, `settings`, `search`, `plus-circle`, `star` |
| `Material Symbols Outlined` | Google Material, 아웃라인 스타일 | `home`, `settings`, `search`, `add`, `favorite_border` |
| `Material Symbols Rounded` | Google Material, 둥근 스타일 | `home`, `settings`, `search`, `add`, `favorite` |
| `Material Symbols Sharp` | Google Material, 날카로운 스타일 | `home`, `settings`, `search`, `add`, `favorite` |
| `phosphor` | 6가지 변형(Six styles) 지원 | `house`, `gear`, `magnifying-glass`, `plus`, `heart` |

> `weight` 속성(100~700)은 라이브러리가 지원하는 경우에만 사용할 수 있습니다.

### 자주 쓰는 아이콘

| 액션 | Lucide | Feather | Material Symbols | Phosphor |
| --- | --- | --- | --- | --- |
| 홈 | `home` | `home` | `home` | `house` |
| 설정 | `settings` | `settings` | `settings` | `gear` |
| 검색 | `search` | `search` | `search` | `magnifying-glass` |
| 추가 | `plus` | `plus` | `add` | `plus` |
| 닫기 | `x` | `x` | `close` | `x` |
| 편집 | `edit`, `pencil` | `edit-2` | `edit` | `pencil-simple` |
| 삭제 | `trash`, `trash-2` | `trash-2` | `delete` | `trash` |
| 확인 | `check` | `check` | `check` | `check` |
| 메뉴 | `menu` | `menu` | `menu` | `list` |
| 알림 | `bell` | `bell` | `notifications` | `bell` |
