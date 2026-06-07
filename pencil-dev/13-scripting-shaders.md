# Pencil Dev 스크립팅과 셰이더

> 원문: Pencil MCP 서버 Scripting/Shader 지침 (get_editor_state로 조회), https://docs.pencil.dev/core-concepts/code-on-canvas

Pencil은 JavaScript 스크립트 노드로 동적 콘텐츠를 생성하고, WebGL 셰이더로 복잡한 그래픽 효과를 만들 수 있습니다.

---

## Script 노드

### 개요

Script 노드는 외부 JavaScript 파일을 연결해 노드의 자식을 동적으로 생성합니다.

```typescript
interface Script extends Entity, Size {
  type: "script";
  clip?: BooleanOrVariable;           // 오버플로우 클리핑. 기본값 false
  scriptUri?: string;                  // JS 파일 URI (.pen 파일 기준 상대경로)
  inputs?: { [key: string]: any };     // 입력값
}
```

### 스크립트 파일 구조

```javascript
/**
 * @schema 2.11                       // ← 필수: 현재 버전
 * @input rows: number = 3            // 입력 선언
 * @input gap: number = 4
 * @input color: color = #3B82F6
 * @input label: string = "Hello"
 * @input filled: boolean = true
 * @input layout: enum("grid", "stack", "scatter") = "grid"
 * @input target: ref
 */

const rows = Math.max(1, Math.floor(pencil.input.rows));
const cellH = (pencil.height - pencil.input.gap * (rows - 1)) / rows;

const nodes = [];
for (let r = 0; r < rows; r++) {
  nodes.push({
    type: "rectangle",
    name: "Bar " + (r + 1),
    x: 0,
    y: r * (cellH + pencil.input.gap),
    width: pencil.width,
    height: cellH * Math.random(),
    fill: pencil.input.color,
  });
}

return nodes;  // 노드 배열을 반환
```

### 필수 요소

| 요소 | 설명 |
| --- | --- |
| `@schema 2.11` | 파일 첫 줄에 필수. 누락 시 에러 |
| `pencil` 객체 | `pencil.width`, `pencil.height`, `pencil.input.<name>` |
| `return` | 노드 객체 배열을 반환해야 함 |

### 입력 타입 (@input)

| 타입 | 설명 | 예시 |
| --- | --- | --- |
| `number` | 숫자 | `@input rows: number = 3` |
| `string` | 문자열 | `@input label: string = "Hello"` |
| `boolean` | 불리언 | `@input filled: boolean = true` |
| `color` | 색상 | `@input color: color = #3B82F6` |
| `ref` | 컴포넌트 참조 | `@input target: ref` |
| `enum` | 열거형 | `@input layout: enum("grid", "stack") = "grid"` |

### pencil 객체

| 속성 | 설명 |
| --- | --- |
| `pencil.width` | Script 노드의 너비 |
| `pencil.height` | Script 노드의 높이 |
| `pencil.input.<name>` | @input으로 선언된 입력값 |

### Math.random()

`Math.random()`은 스크립트에서 결정적으로 동작하므로 프로시저럴 생성에 안전하게 사용할 수 있습니다.

---

## Script 노드 사용

### batch_design에서 삽입

```javascript
scriptNode = Insert(page, {
  type: "script",
  name: "Bar Chart",
  scriptUri: "./bar-chart.js",
  width: 400,
  height: 300,
  inputs: {
    rows: 5,
    gap: 4,
    color: "#3B82F6",
    layout: "grid"
  }
})
```

### 입력값 업데이트

```javascript
Update(scriptNode, {
  inputs: {
    rows: 8,
    color: "#EF4444"
  }
})
```

---

## Shader (셰이더)

### 개요

Shader Fill은 WebGL 1.0 프래그먼트 셰이더(`#version 100`)로 복잡한 그래픽 효과를 만듭니다. 셰이더 파일은 .pen 파일 기준 상대 경로로 참조합니다.

### 셰이더 파일 예시

```glsl
/** @resolution */
uniform vec2 u_resolution;

/** @default 32 */
uniform float u_size;

/** @color @default #ffffff */
uniform vec3 u_color1;

/** @color @default #000000 */
uniform vec3 u_color2;

void main() {
  vec2 cell = floor(gl_FragCoord.xy / u_size);
  float check = mod(cell.x + cell.y, 2.0);
  vec3 color = mix(u_color1, u_color2, check);
  gl_FragColor = vec4(color, 1.0);
}
```

### 유니폼 어노테이션

| 어노테이션 | 설명 |
| --- | --- |
| `@resolution` | 출력 해상도로 자동 설정. `vec2` 유니폼에 사용 |
| `@time` | 경과 시간(초)으로 자동 설정. 애니메이션에 사용 |
| `@mouse` | 마우스 위치(`gl_FragCoord` 공간)로 설정. 인터랙티브 효과에 사용 |
| `@color` | 컬러 피커 컨트롤 표시. `vec3`/`vec4`에 사용 |
| `@default` | 유니폼의 기본값 설정 |

### 지원 유니폼 타입

| GLSL 타입 | JavaScript 값 | 설명 |
| --- | --- | --- |
| `float`, `int` | `number` | 스칼라 |
| `vec2/3/4`, `ivec2/3/4` | `number[]` 또는 `"#RRGGBB"` | 벡터 |
| `sampler2D` | `string` (이미지 URL) | 텍스처 |

### Shader Fill 적용

```javascript
{ fill: {
  type: "shader",
  url: "./checkerboard.glsl",
  opacity: 1.0,
  uniforms: {
    "u_size": 32,
    "u_color1": "#ffffff",
    "u_color2": "#000000"
  }
}}
```

### 주의사항

| 항목 | 설명 |
| --- | --- |
| `@resolution`/`@time` | uniforms에서 제외해야 함 (자동 바인딩) |
| 버전 | WebGL 1.0 (`#version 100`)만 지원 |
| `textureSize` | 지원됨 (aspect-correct texturing용) |
| 파일 경로 | .pen 파일 기준 상대 경로 |

---

## Code on Canvas

Pencil의 "Code on Canvas" 개념은 캔버스 위에서 코드를 직접 실행하는 것을 의미합니다:

| 기능 | 설명 |
| --- | --- |
| Script 노드 | JavaScript로 동적 콘텐츠 생성 |
| Shader Fill | WebGL 셰이더로 그래픽 효과 |
| batch_design API | JavaScript 스니펫으로 디자인 조작 |

이 세 가지를 결합하면 프로그래매틱하게 디자인 요소를 생성하고, 데이터 기반 차트를 그리며, 애니메이션 효과를 만들 수 있습니다.
