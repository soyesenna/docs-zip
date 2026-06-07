# Pencil Dev 캔버스와 레이아웃

> 원문: Pencil MCP 서버 레이아웃 지침 (get_editor_state로 조회)

Pencil은 무한 캔버스에 Flexbox 기반 레이아웃 시스템을 사용합니다. CSS/HTML과 유사하지만 동일하지는 않습니다.

---

## 캔버스 기본 원칙

| 원칙 | 설명 |
| --- | --- |
| 무한 캔버스 | 작업 영역에 제한이 없음 |
| 중첩 계층 | 노드 안에 노드를 중첩해 복잡한 구조 생성 |
| 객체 좌표 | 모든 좌표는 부모의 좌상단 기준. X는 오른쪽, Y는 아래쪽으로 증가 |
| 회전 | 바운딩 박스의 좌상단 기준 반시계 방향 |

### 문서 루트 규칙

문서 루트(`document`)에는 다음만 직접 배치:

- 페이지/스크린 프레임
- 재사용 컴포넌트 프레임 (`reusable: true`)
- 주요 컨테이너 프레임

> 텍스트, 아이콘, 버튼, 카드, 장식용 도형 등은 절대 문서 루트에 직접 배치하지 마세요.

---

## Flexbox 레이아웃

`layout`과 `padding`은 **`frame` 타입에서만** 설정 가능합니다. 다른 노드 타입에는 설정할 수 없습니다.

> **핵심 제약:** Pencil의 Flexbox 레이아웃은 **단일 축(single-axis)만** 지원하며, 아이템 wrapping(줄바꿈)을 지원하지 않습니다. CSS의 `flex-wrap`에 해당하는 기능이 없습니다. 그리드 형태의 레이아웃이 필요하면 각 행(row)을 별도의 Frame으로 수동 생성하세요.

### Layout 인터페이스

```typescript
interface Layout {
  layout?: "none" | "vertical" | "horizontal";  // 기본값 horizontal
  gap?: NumberOrVariable;                         // 자식 간 간격. 기본값 0
  layoutIncludeStroke?: boolean;
  padding?: NumberOrVariable | [N, N] | [N, N, N, N]; // 패딩
  justifyContent?: "start" | "center" | "end" | "space_between" | "space_around";
  alignItems?: "start" | "center" | "end";
}
```

### 오버플로우 제어 (clip)

Frame의 `clip` 속성으로 자식 콘텐츠가 Frame 경계를 넘어갈 때 잘라낼지 여부를 제어합니다.

| 속성 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `clip` | `boolean` | `false` | `true`면 자식 콘텐츠가 Frame 경계를 벗어나지 못하게 잘라냄 |

> **권장:** 화면(스크린) 프레임에는 `clip: true`를 사용하여 콘텐츠 오버플로우를 방지하세요.

```javascript
// 화면 프레임에 clip 적용
screen = Insert(document, {
  type: "frame", name: "Screen",
  layout: "vertical", width: 1440, height: 900,
  clip: true  // 자식이 경계를 벗어나지 않도록 제한
})
```

### 방향

| 방향 | 설명 |
| --- | --- |
| `"horizontal"` | 기본값. 자식을 가로로 나열 |
| `"vertical"` | 자식을 세로로 나열 |
| `"none"` | 절대 배치. 자식의 x/y가 적용됨 |

### 정렬

| 속성 | 옵션 | 설명 |
| --- | --- | --- |
| `justifyContent` | `start`, `center`, `end`, `space_between`, `space_around` | 주축 정렬 |
| `alignItems` | `start`, `center`, `end` | 교차축 정렬. **`baseline`, `stretch` 지원 안 함** |

### 간격과 패딩

| 속성 | 설명 |
| --- | --- |
| `gap` | 자식 사이의 간격 (주축) |
| `padding` | 컨테이너 내부 여백 (Inside padding) |

**패딩 형식:**

```javascript
padding: 16                                    // 전면
padding: [16, 24]                              // [상하, 좌우]
padding: [16, 24, 16, 24]                      // [상, 우, 하, 좌]
```

---

## Sizing (크기 지정)

> **기본 사이징:** Frame은 항상 방향이 `horizontal`이며, 크기는 기본적으로 `fit_content`로 설정됩니다. 즉, 명시적으로 width/height를 지정하지 않으면 자식 크기 합에 맞춰 자동 조절됩니다.

### 동적 크기

| 값 | 설명 | 사용 시기 | 사전조건 |
| --- | --- | --- | --- |
| `"fit_content"` | 자식 크기 합 | 부모가 자식에 맞출 때 | 해당 노드에 layout이 설정되어 있어야 함 |
| `"fill_container"` | 부모 크기 | 자식이 부모에 꽉 찰 때 | 부모 노드에 layout이 설정되어 있어야 함 |
| `fit_content(N)` | 자식 크기 + fallback N | 자식이 없을 때 N 사용 | 해당 노드에 layout이 설정되어 있어야 함 |
| `fill_container(N)` | 부모 크기 + fallback N | 부모가 레이아웃이 아닐 때 N 사용 | 부모 노드에 layout이 설정되어 있어야 함 |

### 핵심 원칙

> **동적 크기를 하드코딩된 픽셀 값보다 선호하세요.** `fill_container`와 `fit_content`를 사용하면 디자인이 더 유지보수 가능해집니다.

### 안티패턴

```javascript
// ❌ 잘못됨: 퍼센트 값 지원 안 함
{ width: "100%", height: "50%" }

// ❌ 잘못됨: 텍스트에 패딩 없음 (래핑 프레임 필요)
{ type: "text", content: "text", fontSize: 12, padding: 12 }

// ❌ 잘못됨: 순환 의존. 부모=fit_content, 자식=fill_container
badParent = Insert(screen, { type: "frame", layout: "vertical" });
Insert(badParent, { type: "text", textGrowth: "fixed-width", width: "fill_container" });
```

### 자식을 균등 분배

여러 자식이 같은 부모 크기를 공유해야 할 때, 각각에 `fill_container`를 사용:

```javascript
row = Insert(page, { type: "frame", layout: "horizontal", gap: 16, width: "fill_container" })
Insert(row, { type: "ref", ref: cardId, width: "fill_container" })  // 균등 분배
Insert(row, { type: "ref", ref: cardId, width: "fill_container" })
Insert(row, { type: "ref", ref: cardId, width: "fill_container" })
```

---

## Positioning (위치 지정)

### Flexbox 레이아웃에서

부모가 Flexbox 레이아웃을 사용하면 **자식의 x/y 속성은 무시됩니다.** x/y를 설정하지 마세요.

### 절대 배치

`layout: "none"`인 부모 안에서 자식의 x/y로 위치 지정:

```javascript
container = Insert(page, { type: "frame", layout: "none", width: 400, height: 300 })
Insert(container, { type: "text", x: 20, y: 20, content: "제목" })
Insert(container, { type: "rectangle", x: 0, y: 100, width: 400, height: 2, fill: "#ccc" })
```

### layoutPosition: "absolute"

Flexbox 레이아웃 안에서 특정 자식만 절대 배치할 수 있습니다.

| 값 | 설명 |
| --- | --- |
| `"auto"` | 기본값. 부모의 레이아웃 흐름에 따라 자동 배치됨 |
| `"absolute"` | 부모의 레이아웃 흐름에서 분리되어 x/y 좌표로 절대 배치됨 |

```javascript
// 부모는 flex 레이아웃이지만 이 자식만 절대 위치
Insert(parent, {
  type: "text",
  layoutPosition: "absolute",
  x: 10, y: 10,
  content: "배지"
})
```

---

## 레이아웃 패턴

### Sidebar + Content (대시보드)

```
┌──────────┬────────────────────────────────┐
│          │                                │
│ Sidebar  │     Main Content Area          │
│  280px   │      fill_container            │
│          │                                │
└──────────┴────────────────────────────────┘
```

```javascript
screen = Insert(document, {
  type: "frame", name: "Dashboard",
  layout: "horizontal", width: 1440, height: "fit_content(900)",
  fill: "$background", placeholder: true
})
sidebar = Insert(screen, { type: "ref", ref: sidebarId, height: "fill_container" })
main = Insert(screen, { type: "frame", layout: "vertical", width: "fill_container", height: "fill_container(900)", padding: 32, gap: 24 })
```

### Header + Content

```
┌────────────────────────────────────────────┐
│              Header Bar (64px)             │
├────────────────────────────────────────────┤
│            Content Area (scrollable)       │
└────────────────────────────────────────────┘
```

```javascript
screen = Insert(document, {
  type: "frame", layout: "vertical", width: 1200, height: "fit_content(800)"
})
header = Insert(screen, {
  type: "frame", layout: "horizontal", width: "fill_container", height: 64,
  padding: [0, 24], alignItems: "center", justifyContent: "space_between"
})
content = Insert(screen, { type: "frame", layout: "vertical", width: "fill_container", height: "fit_content(736)", padding: 32, gap: 24 })
```

### 2컬럼 레이아웃

```
┌─────────────────────┬─────────────┐
│                     │             │
│    Main (2/3)       │  Side (1/3) │
│   fill_container    │   360px     │
└─────────────────────┴─────────────┘
```

```javascript
columns = Insert(content, { type: "frame", layout: "horizontal", width: "fill_container", gap: 24 })
mainCol = Insert(columns, { type: "frame", layout: "vertical", width: "fill_container", gap: 24 })
sideCol = Insert(columns, { type: "frame", layout: "vertical", width: 360, gap: 24 })
```

---

## CSS/HTML과의 차이점

| Pencil | CSS/HTML | 비고 |
| --- | --- | --- |
| `layout: "vertical"` | `flex-direction: column` | 동일 개념, 다른 이름 |
| `layout: "none"` | `position: relative` + 자식 `position: absolute` | 절대 배치 |
| `fill_container` | `flex: 1` 또는 `width: 100%` | Pencil은 퍼센트 미지원 |
| `fit_content` | `width: fit-content` | 유사 |
| `alignItems` | `align-items` | `baseline`, `stretch` 미지원 |
| `padding` (frame만) | `padding` (모든 요소) | Pencil은 frame만 패딩 가능 |
| `clip` (frame) | `overflow: hidden` | Frame 전용. 기본값 `false` |
| `gap` | `gap` | 동일 |
| ❌ `margin` | `margin` | Pencil은 margin 미지원 |
| ❌ `flex-wrap` | `flex-wrap` | Pencil은 wrapping 미지원. 행별 별도 Frame 생성 필요 |
| ❌ `%` 크기 | `%` 크기 | Pencil은 퍼센트 미지원 |

> Pencil의 속성이 CSS와 유사해 보이지만, 실제로는 다르게 동작합니다. 스키마에 명시된 속성만 사용하고, 없는 속성은 다른 방법으로 구현하세요.
