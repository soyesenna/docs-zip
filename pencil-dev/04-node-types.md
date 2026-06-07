# Pencil Dev 노드 타입 참조

> 원문: Pencil MCP 서버 .pen 파일 스키마 (get_editor_state로 조회)

.pen 파일의 모든 시각 요소는 노드(Node)로 표현됩니다. 각 노드는 특정 타입을 가지며, 고유한 속성 집합을 갖습니다.

---

## 노드 타입 개요

| 타입 | 용도 | 자식 가능 | 레이아웃 |
| --- | --- | --- | --- |
| `frame` | 컨테이너, 화면, 컴포넌트 | ✅ | ✅ Flexbox |
| `group` | 논리적 그룹화 | ✅ | ❌ |
| `rectangle` | 사각형 도형 | ❌ | ❌ |
| `ellipse` | 원/타원/호 | ❌ | ❌ |
| `polygon` | 정다각형 | ❌ | ❌ |
| `path` | SVG 패스 | ❌ | ❌ |
| `text` | 텍스트 | ❌ | ❌ |
| `note` | 메모/주석 | ❌ | ❌ |
| `prompt` | AI 프롬프트 | ❌ | ❌ |
| `context` | AI 컨텍스트 | ❌ | ❌ |
| `icon` | 아이콘 라이브러리 | ❌ | ❌ |
| `script` | JS 동적 생성 | ❌ | ❌ |
| `ref` | 컴포넌트 인스턴스 | ❌ | ❌ |

---

## 공통 속성 (Entity)

모든 노드가 상속하는 기본 속성:

| 속성 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 고유 식별자. `/` 포함 불가. 생략 시 자동 생성 |
| `name` | `string` | 사람이 읽을 수 있는 이름 |
| `context` | `string` | 컨텍스트 정보 |
| `reusable` | `boolean` | `true` 시 컴포넌트로 정의, `ref`로 재사용 가능 |
| `theme` | `Theme` | 테마 축→값 매핑 (예: `{ device: "phone" }`) |
| `enabled` | `BooleanOrVariable` | `false` 시 숨김 |
| `opacity` | `NumberOrVariable` | 불투명도 (0~1) |
| `flipX` | `BooleanOrVariable` | 수평 뒤집기 |
| `flipY` | `BooleanOrVariable` | 수직 뒤집기 |
| `layoutPosition` | `"auto" \| "absolute"` | 레이아웃 내 위치. 기본값 `auto` |
| `metadata` | `object` | 임의 메타데이터 |
| `rotation` | `NumberOrVariable` | 좌상단 기준 반시계 방향 회전 (도) |
| `x`, `y` | `number` | 부모 기준 위치 (Flexbox 레이아웃에서는 무시됨) |

---

## Frame

가장 핵심적인 컨테이너 노드. 화면, 섹션, 컴포넌트 등 모든 구조적 요소에 사용.

```typescript
interface Frame extends Rectangleish, CanHaveChildren, Layout {
  type: "frame";
  clip?: BooleanOrVariable;           // 오버플로우 클리핑. 기본값 false
  placeholder?: boolean;              // 작업 중 표시
  slot?: false | string[];            // 슬롯 정의
}
```

| 특성 | 기본값 |
| --- | --- |
| `layout` | `"horizontal"` |
| `width` | `fit_content` |
| `height` | `fit_content` |
| `clip` | `false` |

### Slot

`slot` 속성으로 프레임을 슬롯으로 표시할 수 있습니다. 배열의 항목은 추천되는 자식 컴포넌트 ID입니다:

```javascript
{ slot: ["buttonPrimaryId", "buttonSecondaryId"] }
```

---

## Group

자식을 논리적으로 묶는 컨테이너. 레이아웃이나 크기는 없고 효과만 적용 가능.

```typescript
interface Group extends Entity, CanHaveChildren, CanHaveEffects {
  type: "group";
}
```

---

## Rectangle

사각형 도형. 배경, 버튼 베이스, 카드 등에 사용.

```typescript
interface Rectangle extends Rectangleish {
  type: "rectangle";
}
```

`Rectangleish`은 `Entity`, `Size`, `CanHaveGraphics`를 상속하며 `cornerRadius`를 추가:

| 속성 | 타입 | 설명 |
| --- | --- | --- |
| `cornerRadius` | `NumberOrVariable \| [N, N, N, N]` | 모서리 반경 (단일값 또는 좌상/우상/우하/좌하) |

---

## Ellipse

원, 타원, 호. 경계 사각형으로 정의.

```typescript
interface Ellipse extends Entity, Size, CanHaveGraphics {
  type: "ellipse";
  innerRadius?: NumberOrVariable;   // 0=원, 1=도넛. 기본값 0
  startAngle?: NumberOrVariable;    // 호 시작 각도 (도, 반시계, 오른쪽 기준). 기본값 0
  sweepAngle?: NumberOrVariable;    // 호 길이. +는 반시계, -는 시계. -360~360. 기본값 360
}
```

---

## Polygon

정다각형. 변 수와 모서리 반경 지정.

```typescript
interface Polygon extends Entity, Size, CanHaveGraphics {
  type: "polygon";
  polygonCount?: NumberOrVariable;   // 변 수
  cornerRadius?: NumberOrVariable;   // 모서리 반경
}
```

---

## Path

SVG 패스 데이터로 임의 형상을 정의.

```typescript
interface Path extends Entity, Size, CanHaveGraphics {
  type: "path";
  fillRule?: "nonzero" | "evenodd";  // 기본값 nonzero
  geometry?: string;                  // SVG path 데이터
  viewBox?: [number, number, number, number]; // SVG 좌표공간 [x,y,w,h]
}
```

| 규칙 | 설명 |
| --- | --- |
| `viewBox` | 항상 명시적으로 설정. SVG 좌표공간을 노드 박스에 매핑 |
| `geometry` | SVG path 명령어 (`M`, `L`, `C`, `Z` 등) |

---

## Text

텍스트 표시. 가장 세밀한 제어가 필요한 노드.

```typescript
interface Text extends Entity, Size, CanHaveGraphics, TextStyle {
  type: "text";
  content?: TextContent;                    // 텍스트 내용
  textGrowth?: "auto" | "fixed-width" | "fixed-width-height";  // 크기 결정 방식
}
```

**중요:** Text는 기본적으로 `fill`이 없어 보이지 않습니다. 반드시 `fill` 속성을 설정해야 합니다.

### TextStyle 속성

| 속성 | 타입 | 설명 |
| --- | --- | --- |
| `fontFamily` | `StringOrVariable` | 글꼴 (모든 Google Fonts 사용 가능) |
| `fontSize` | `NumberOrVariable` | 크기 (px) |
| `fontWeight` | `StringOrVariable` | 굵기 |
| `letterSpacing` | `NumberOrVariable` | 자간 |
| `fontStyle` | `StringOrVariable` | 스타일 |
| `underline` | `BooleanOrVariable` | 밑줄 |
| `lineHeight` | `NumberOrVariable` | 줄간격 (fontSize의 배수) |
| `textAlign` | `"left" \| "center" \| "right" \| "justify"` | 가로 정렬 |
| `textAlignVertical` | `"top" \| "middle" \| "bottom"` | 세로 정렬 |
| `strikethrough` | `BooleanOrVariable` | 취소선 |
| `href` | `string` | 링크 URL |

### textGrowth 모드

| 모드 | 너비 | 높이 | 줄바꿈 |
| --- | --- | --- | --- |
| `auto` (기본) | 콘텐츠에 맞춤 | 콘텐츠에 맞춤 | 안 함 (항당 한 줄) |
| `fixed-width` | **필수 지정** | 콘텐츠에 맞춤 | 함 |
| `fixed-width-height` | **필수 지정** | **필수 지정** | 함 (오버플로우 가능) |

---

## Note, Prompt, Context

AI 및 메모 관련 노드:

| 타입 | 용도 | 속성 |
| --- | --- | --- |
| `note` | 디자인 메모/주석 | `content`, TextStyle 속성 |
| `prompt` | AI 프롬프트 정의 | `content`, `model`, TextStyle 속성 |
| `context` | AI 컨텍스트 제공 | `content`, TextStyle 속성 |

---

## Icon

아이콘 라이브러리의 아이콘을 표시. 너비/높이에 맞게 스케일됨.

```typescript
interface Icon extends Entity, Size, CanHaveEffects {
  type: "icon";
  library?: StringOrVariable;   // 아이콘 라이브러리
  icon?: StringOrVariable;      // 아이콘 이름
  weight?: NumberOrVariable;    | // 가중치 100-700
  fill?: Fills;                 // 색상
}
```

### 지원 라이브러리

| 라이브러리 | 스타일 | 예시 아이콘 |
| --- | --- | --- |
| `lucide` | 아웃라인, 둥근 모서리 | `home`, `settings`, `user`, `plus`, `x` |
| `feather` | 아웃라인, 둥근 모서리 | `home`, `settings`, `user`, `plus`, `x` |
| `Material Symbols Outlined` | 아웃라인 | `home`, `settings`, `person`, `search` |
| `Material Symbols Rounded` | 둥근 모서리 | `home`, `settings`, `person`, `search` |
| `Material Symbols Sharp` | 날카로운 모서리 | `home`, `settings`, `person`, `search` |
| `phosphor` | 다양한 변형 | `home`, `settings`, `user` |

---

## Script

JavaScript 파일로 노드의 자식을 동적 생성.

```typescript
interface Script extends Entity, Size {
  type: "script";
  clip?: BooleanOrVariable;
  scriptUri?: string;           // JS 파일 URI (.pen 파일 기준 상대경로)
  inputs?: { [key: string]: any }; // 입력값
}
```

---

## Ref

다른 노드(컴포넌트)를 재사용하는 인스턴스.

```typescript
interface Ref extends Entity {
  type: "ref";
  ref: string;                   // 참조할 컴포넌트의 ID
  descendants?: { [key: string]: object }; // 하위 노드 오버라이드
}
```

| 속성 | 설명 |
| --- | --- |
| `ref` | 재사용할 컴포넌트의 `id` |
| `descendants` | 키=하위 노드의 ID/경로/고유 이름, 값=오버라이드 속성 |

---

## 크기 (Size)

`width`와 `height`를 지정하는 노드(Frame, Rectangle, Ellipse, Polygon, Path, Text, Icon, Script):

| 값 | 설명 |
| --- | --- |
| 숫자 | 고정 픽셀 값 |
| `Variable` | `$variable-name` 형태의 변수 참조 |
| `"fit_content"` | 자식 크기 합. fallback 가능: `fit_content(100)` |
| `"fill_container"` | 부모 크기. fallback 가능: `fill_container(900)` |

---

## 그래픽 속성 (CanHaveGraphics)

`fill`, `stroke`, `effect`를 가질 수 있는 노드:

| 속성 | 설명 |
| --- | --- |
| `fill` | 단일 Fill 또는 Fill 배열 |
| `stroke` | 단일 Fill 또는 Fill 배열 (스트로크에 적용) |
| `strokeWidth` | 숫자 또는 `{ top, right, bottom, left }` |
| `strokeLinecap` | `"butt"`, `"round"`, `"square"` |
| `strokeLinejoin` | `"miter"`, `"bevel"`, `"round"` |
| `strokeAlignment` | `"inner"`, `"center"`, `"outer"` |
| `effect` | 단일 Effect 또는 Effect 배열 |
