# Pencil Dev .pen 파일 기본

> 원문: https://docs.pencil.dev/core-concepts/pen-files, https://docs.pencil.dev/core-concepts/design-as-code, https://docs.pencil.dev/for-developers/the-pen-format

.pen 파일은 Pencil의 독자적인 벡터 디자인 파일 포맷입니다. 암호화되어 있어 MCP 도구로만 접근해야 합니다.

---

## 파일 포맷 개요

| 속성 | 설명 |
| --- | --- |
| 포맷 | 암호화된 독자 포맷 |
| 기반 | JSON 스키마 기반 내부 구조 |
| 현재 버전 | `2.13` |
| 접근 방식 | Pencil MCP 도구만으로 접근 (Read/Grep 사용 금지) |

### 주의사항

> .pen 파일은 암호화되어 있습니다. Pencil MCP 도구(`batch_get`, `batch_design` 등)로만 접근하세요. 일반 `Read`나 `Grep` 도구로는 내용을 읽을 수 없습니다.

---

## Document 구조

.pen 파일의 최상위 구조는 `Document` 인터페이스를 따릅니다:

```typescript
interface Document {
  version: "2.13";                              // 파일 버전
  themes?: { [key: string]: string[] };         // 테마 축 정의
  imports?: { [key: string]: string };          // 외부 .pen 파일 임포트
  variables?: { [key: string]: VariableDef };   // 전역 변수 정의
  children: Child[];                            // 최상위 노드 배열
}
```

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `version` | `"2.13"` | 파일 포맷 버전 (고정값) |
| `themes` | `object` | 테마 축 정의. 키=축 이름, 값=축 값 배열 |
| `imports` | `object` | 외부 .pen 파일 임포트. 키=별칭, 값=상대 URI |
| `variables` | `object` | 전역 변수. 키=변수명(정규식 `[^:]+`), 값=타입별 정의 |
| `children` | `Child[]` | 문서의 최상위 자식 노드들 |

### Child 노드 타입

`children` 배열에는 다음 노드 타입들이 올 수 있습니다:

| 타입 | 용도 | 상세 문서 |
| --- | --- | --- |
| `Frame` | 컨테이너, 화면, 컴포넌트 (layout, padding, slot 지원) | [노드 타입 참조](./04-node-types.md) |
| `Group` | 논리적 그룹화 | [노드 타입 참조](./04-node-types.md) |
| `Rectangle` | 사각형 도형 | [노드 타입 참조](./04-node-types.md) |
| `Ellipse` | 원/타원/호 | [노드 타입 참조](./04-node-types.md) |
| `Polygon` | 정다각형 | [노드 타입 참조](./04-node-types.md) |
| `Path` | SVG 패스 | [노드 타입 참조](./04-node-types.md) |
| `Text` | 텍스트 (textGrowth, 폰트 스타일) | [노드 타입 참조](./04-node-types.md), [텍스트 가이드](./06-text-typography.md) |
| `Icon` | 아이콘 라이브러리 (lucide, Material 등) | [노드 타입 참조](./04-node-types.md) |
| `Script` | JavaScript 동적 콘텐츠 생성 | [스크립팅 가이드](./13-scripting-shaders.md) |
| `Ref` | 컴포넌트 인스턴스 (재사용) | [컴포넌트 가이드](./07-components-slots.md) |
| `Note` | 메모/주석 | [노드 타입 참조](./04-node-types.md) |
| `Prompt` | AI 프롬프트 정의 | [노드 타입 참조](./04-node-types.md) |
| `Context` | AI 컨텍스트 제공 | [노드 타입 참조](./04-node-types.md) |

### Entity (모든 노드의 기본 속성)

| 속성 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 고유 식별자 (`/` 포함 불가, 자동 생성) |
| `name` | `string` | 사람이 읽을 수 있는 이름 |
| `reusable` | `boolean` | `true` 시 컴포넌트로 정의, `ref`로 재사용 가능 |
| `theme` | `Theme` | 테마 축→값 매핑 |
| `enabled` | `BooleanOrVariable` | `false` 시 숨김 |
| `opacity` | `NumberOrVariable` | 불투명도 |
| `rotation` | `NumberOrVariable` | 좌상단 기준 반시계 방향 회전 (도) |
| `x`, `y` | `number` | 부모 기준 위치 (Flexbox에서는 무시) |
| `layoutPosition` | `"auto" \| "absolute"` | 레이아웃 내 위치 |

### Layout (Frame 전용)

Frame 노드는 Flexbox 레이아웃을 지원합니다. 상세한 레이아웃 규칙은 [캔버스와 레이아웃](./05-canvas-layout.md)을 참조하세요.

| 속성 | 옵션 | 설명 |
| --- | --- | --- |
| `layout` | `"none"`, `"vertical"`, `"horizontal"` | 레이아웃 방향 |
| `gap` | `NumberOrVariable` | 자식 간 간격 |
| `padding` | `NumberOrVariable \| [N,N] \| [N,N,N,N]` | 안쪽 여백 |
| `justifyContent` | `start`, `center`, `end`, `space_between`, `space_around` | 주축 정렬 |
| `alignItems` | `start`, `center`, `end` | 교차축 정렬 |

### Sizing (동적 크기)

| 값 | 설명 |
| --- | --- |
| `"fit_content"` | 자식 크기에 맞춤. fallback 가능: `fit_content(100)` |
| `"fill_container"` | 부모 크기에 맞춤. fallback 가능: `fill_container(900)` |
| 숫자 | 고정 픽셀 값 |

> **순환 의존 주의:** 부모가 `fit_content`인데 모든 자식이 `fill_container`이면 순환 의존이 발생합니다.

---

## Design as Code 개념

Pencil은 "Design as Code" 철학을 따릅니다:

| 개념 | 설명 |
| --- | --- |
| 구조적 디자인 | JSON 스키마로 디자인의 모든 속성이 정의됨 |
| 프로그래매틱 접근 | batch_design API로 JavaScript 코드로 디자인 생성/수정 |
| 버전 관리 | .pen 파일을 Git으로 관리 가능 |
| AI 친화적 | MCP를 통해 AI 에이전트가 직접 디자인 수정 |

---

## 파일 임포트

.pen 파일은 다른 .pen 파일을 임포트할 수 있습니다:

```typescript
imports: {
  "design-system": "./design-system.pen",
  "icons": "./icons.pen"
}
```

| 규칙 | 설명 |
| --- | --- |
| 경로 | .pen 파일 기준 상대 URI |
| 키 | 임포트된 파일의 짧은 별칭 |
| 제한 | 컴포넌트는 파일 간 참조 불가 — 복사해서 사용해야 함 |

---

## 테마 시스템

테마는 축(axis)과 값으로 정의됩니다:

```typescript
themes: {
  "mode": ["light", "dark"],           // 라이트/다크 모드
  "device": ["desktop", "tablet", "phone"]  // 디바이스 변형
}
```

| 규칙 | 설명 |
| --- | --- |
| 축 이름 | 정규식 `[^:]+` 형식 |
| 축 값 | 문자열 배열 |
| 변수 연동 | 변수 값에 테마별 값을 지정할 수 있음 |

---

## 변수 정의

변수는 4가지 타입을 지원합니다:

```typescript
variables: {
  "primary-color": {
    type: "color",
    value: "#3B82F6"                        // 단일 값
  },
  "background": {
    type: "color",
    value: [                                 // 테마별 값
      { value: "#FFFFFF", theme: { mode: "light" } },
      { value: "#1A1A1A", theme: { mode: "dark" } }
    ]
  },
  "spacing-unit": {
    type: "number",
    value: 16
  },
  "font-heading": {
    type: "string",
    value: "Inter"
  },
  "dark-mode": {
    type: "boolean",
    value: false
  }
}
```

| 타입 | 값 형식 | 참조 방식 |
| --- | --- | --- |
| `color` | `ColorOrVariable` 또는 테마 배열 | `"$primary-color"` |
| `number` | `NumberOrVariable` 또는 테마 배열 | `"$spacing-unit"` |
| `string` | `StringOrVariable` 또는 테마 배열 | `"$font-heading"` |
| `boolean` | `BooleanOrVariable` 또는 테마 배열 | `"$dark-mode"` |

### 변수 참조 규칙

- 변수 참조는 `$` 접두사 사용: `fill: "$primary-color"`, `gap: "$spacing-small"`
- 변수명은 `$`로 시작하면 안 됨 (참조 시에만 `$` 사용)
- 속성에서 변수를 참조하면 해당 변수의 값으로 바인딩됨

---

## 파일 작업 모범 사례

### 문서 루트 정리

문서 루트에는 다음만 직접 배치:

| 노드 | 설명 |
| --- | --- |
| 페이지/스크린 프레임 | 각 화면을 나타내는 최상위 프레임 |
| 재사용 컴포넌트 프레임 | `reusable: true` 설정된 컴포넌트 |
| 주요 컨테이너 프레임 | 대규모 컨테이너 |

> 텍스트, 아이콘, 버튼, 카드 등의 요소는 문서 루트에 직접 배치하지 마세요. 반드시 프레임 안에 넣어야 합니다.

### 플레이스홀더

새로 생성하거나 수정 중인 루트 프레임은 작업이 완료될 때까지 `placeholder: true`를 유지해야 합니다. 작업이 끝나면 즉시 `placeholder: false`로 설정하세요.

### 새 화면 배치

문서 루트에 새 객체를 배치할 때는 `FindEmptySpace`를 사용해 빈 영역을 찾으세요. 루트 객체는 절대 겹치지 않게 합니다.
