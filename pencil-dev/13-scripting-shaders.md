# Pencil Dev 스크립팅과 셰이더

> 원문: https://docs.pencil.dev/core-concepts/code-on-canvas
>
> Pencil MCP 서버 Scripting/Shader 지침 (get_editor_state로 조회)

Pencil은 JavaScript 스크립트 노드로 동적 콘텐츠를 생성하고, WebGL 셰이더로 복잡한 그래픽 효과를 만들 수 있습니다.

---

## Code on Canvas

Pencil은 **Script 노드**를 사용해 코드를 캔버스로 가져옵니다. 캔버스에 배치하고 `.js` 파일을 지정하면, 스크립트의 출력이 중첩 레이어로 렌더링됩니다 -- 파이 차트, 그리드, 장식 패턴 등 코드로 설명할 수 있는 모든 것을 만들 수 있습니다.

데이터에 의해 구동되거나, 변형을 반복하거나, 몇 가지 노브로 파라미터화하여 인터랙티브하게 조정하고 싶은 구조가 필요할 때마다 사용하세요.

| 기능 | 설명 |
| --- | --- |
| Script 노드 | JavaScript로 동적 콘텐츠 생성 |
| Shader Fill | WebGL 셰이더로 그래픽 효과 |
| batch_design API | JavaScript 스니펫으로 디자인 조작 |

이 세 가지를 결합하면 프로그래매틱하게 디자인 요소를 생성하고, 데이터 기반 차트를 그리며, 애니메이션 효과를 만들 수 있습니다.

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
 * @schema 2.13                       // ← 필수: 현재 버전
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
| `@schema 2.13` | 파일 첫 줄에 필수. 누락 시 에러. 현재 스키마 버전은 `2.13` |
| `pencil` 객체 | `pencil.width`, `pencil.height`, `pencil.input.<name>` |
| `return` | 노드 객체 배열을 반환해야 함 |

### 입력 타입 (@input)

| 타입 | 예시 | 비고 |
| --- | --- | --- |
| `number` | `@input size: number(min=0, max=100) = 10` | 선택적 `min` / `max` named args로 스크립트 실행 전 값 클램핑 |
| `string` | `@input label: string = "Hello"` | 패널에서 여러 줄 텍스트 입력 |
| `boolean` | `@input filled: boolean = true` | 체크박스로 렌더링 |
| `color` | `@input fill: color = #3B82F6` | Hex 리터럴 또는 따옴표 문자열 |
| `enum` | `@input layout: enum("grid", "stack") = "grid"` | 따옴표로 묶은 옵션의 위치 인수 목록, 드롭다운으로 렌더링 |
| `ref` | `@input target: ref` | 재사용 컴포넌트 참조, 썸네일 그리드에서 선택 |

### pencil 객체

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `pencil.width` | `number` | Script 노드의 현재 너비 |
| `pencil.height` | `number` | Script 노드의 현재 높이 |
| `pencil.input.<name>` | @input 타입과 일치 | 해당 입력의 현재 값 (이미 클램핑 및 검증됨) |

### Math.random()

`Math.random()`은 결정적으로 동작합니다. 매 실행 시마다 리시드되므로, 동일한 입력은 항상 동일한 출력을 생성합니다. 프로시저럴 생성에 안전하게 사용할 수 있습니다.

---

## Script 노드 사용법

### 스크립트 노드 만들기

1. 툴바에서 **Shape 도구 드롭다운**을 엽니다.
2. **Script** (`</>` 아이콘)를 선택합니다.
3. 캔버스 아무 곳이나 클릭하여 200x200 스크립트 노드를 배치합니다.

새 노드는 `.js` 파일을 지정할 때까지 녹색 **"No script file selected"** 플레이스홀더를 표시합니다.

### .js 파일 연결

**속성 패널**에서 script 필드에 경로를 입력합니다. 경로는 `.pen` 파일 기준 상대 경로로 해석되므로, 같은 폴더의 파일은 `chart.js`로 지정하면 됩니다. JS 파일을 `.pen` 파일과 같은 폴더(또는 하위 폴더)에 저장하고 참조하세요.

스크립트를 직접 작성할 필요는 없습니다. 두 가지 쉬운 시작 방법:

| 방법 | 설명 |
| --- | --- |
| **기성 스크립트 사용** | [highagency/pencil-scripts](https://github.com/highagency/pencil-scripts)에서 Pencil 팀이 관리하는 예제 스크립트(차트, 그리드, 패턴 등)를 다운로드하여 `.pen` 파일 옆에 배치 |
| **AI 에이전트 활용** | 에이전트 패널을 열고 원하는 내용을 설명하면, `.js` 파일을 생성하고 선택된 스크립트 노드에 연결 |

### 캔버스에서 조정

| 작업 | 설명 |
| --- | --- |
| **노드 크기 조절** | 핸들을 드래그하거나 속성 패널에서 크기 변경. 스크립트가 재실행되며 레이아웃이 실시간으로 갱신 |
| **입력 컨트롤 사용** | 스크립트 헤더의 모든 `@input` 줄이 속성 패널의 컨트롤이 됨. 숫자, 문자열, 불리언, 색상, enum 드롭다운, 기존 컴포넌트 참조 모두 지원. 컨트롤 편집 시 스크립트 즉시 재실행 |
| **.js 파일 편집** | Pencil이 디스크의 파일을 감시. 에디터에서 저장하면 캔버스가 새로고침 없이 업데이트 |
| **레이어로 변환 (Convert to layers)** | 생성된 콘텐츠를 직접 편집해야 할 때, 속성 패널 하단의 버튼을 클릭. 스크립트 노드가 생성된 레이어의 스냅샷을 포함한 일반 프레임으로 교체됨 |

> **팁:** 스크립트 입력을 변수(예: `$primary`)에 바인딩하면 테마 변경을 자동으로 감지합니다.

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

## 스크립트 실행 환경

### 작동 방식

| 항목 | 설명 |
| --- | --- |
| **외부 파일** | `.js` 파일은 `.pen` 파일 외부에 존재. 스크립트 노드는 경로만 저장 |
| **다중 노드 참조** | 여러 스크립트 노드가 같은 파일을 가리킬 수 있음. 각 노드마다 고유한 크기와 입력값을 가짐 |
| **헤더에서 입력 선언** | 헤더 주석의 `@input` 줄이 속성 패널에 렌더링할 컨트롤을 결정. 파일에서 입력을 이름 변경하거나 제거하면 패널이 업데이트됨 |
| **노드 배열 반환** | 각 반환 객체는 Pencil의 나머지 부분과 동일한 스키마를 사용하며, 스크립트 노드의 자식으로 삽입됨 |

### 제한 사항

| 항목 | 설명 |
| --- | --- |
| **샌드박스 실행** | DOM, 네트워크, 파일시스템에 접근할 수 없으며, 반드시 동기적으로 실행해야 함 (async, setTimeout 사용 불가) |
| **노드 수 제한** | 스크립트는 최대 **1,000개**의 노드를 반환할 수 있음 |
| **실행 시간 제한** | 스크립트는 **2초** 이내에 완료해야 함 |
| **파생 상태 (Derived state)** | 생성된 자식은 스크립트가 실행될 때마다 재렌더링되며, 실행 취소(undo) 기록에 포함되지 않음. 편집 가능한 레이어로 변환하려면 **레이어로 변환** 사용 |

### 에러 처리

에러와 누락된 파일은 노드에 직접 표시되며, 실패한 코드 줄이 하이라이트됩니다. 콘솔을 열 필요가 없습니다.

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
