# Pencil Dev 가져오기, 내보내기, CLI

> 원문: https://docs.pencil.dev/core-concepts/import-and-export, https://docs.pencil.dev/core-concepts/design-libraries, https://docs.pencil.dev/for-developers/pencil-cli

Pencil은 다양한 형식으로 가져오기/내보내기를 지원하고, 디자인 라이브러리 시스템과 CLI 도구를 제공합니다.

---

## 가져오기 (Import)

### 지원 형식

| 형식 | 설명 |
| --- | --- |
| SVG | 벡터 그래픽을 Pencil 패스로 변환 |
| 이미지 (PNG, JPG) | 이미지를 Frame의 Fill로 가져오기 |
| 다른 .pen 파일 | `imports` 필드로 외부 .pen 파일 참조 |

### 파일 임포트

Document의 `imports` 필드로 다른 .pen 파일을 참조:

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
| 컴포넌트 | **파일 간 컴포넌트 참조 불가** — 복사해서 사용해야 함 |

---

## 내보내기 (Export)

### MCP 도구로 내보내기

`export_nodes` 도구로 노드를 이미지 파일로 내보내기:

```javascript
export_nodes({
  filePath: "design.pen",
  nodeIds: ["screen1Id", "screen2Id"],
  outputDir: "/path/to/output",
  format: "png",          // "png" | "jpeg" | "webp" | "pdf"
  scale: 2,               // 기본값 2x
  quality: 95              // JPEG/WEBP용 (1-100)
})
```

| 매개변수 | 설명 | 옵션 |
| --- | --- | --- |
| `format` | 출력 형식 | `png`(기본), `jpeg`, `webp`, `pdf` |
| `scale` | 스케일 팩터 | 기본값 2 (고해상도) |
| `quality` | JPEG/WEBP 품질 | 1–100. 기본값: JPEG=95, WEBP=100 |
| `outputDir` | 출력 디렉터리 | 필수 |
| `nodeIds` | 내보낼 노드 ID 배열 | 필수. PDF는 모든 노드를 단일 문서로 결합 |

### PDF 내보내기

PDF 형식은 모든 노드를 하나의 다중 페이지 문서로 결합합니다.

---

## 디자인 라이브러리

### 개념

디자인 라이브러리는 재사용 가능한 컴포넌트, 변수, 스타일을 모아둔 .pen 파일입니다.

| 특징 | 설명 |
| --- | --- |
| 독립 파일 | 별도의 .pen 파일로 관리 |
| 임포트 | 다른 .pen 파일에서 `imports`로 참조 |
| 컴포넌트 공유 | 버튼, 카드, 폼 등의 공통 UI 요소 |
| 변수 공유 | 색상, 간격, 타이포그래피 토큰 |
| 버전 관리 | Git으로 라이브러리 버전 관리 가능 |

### 라이브러리 구성 예시

```
design-library.pen
├── 컴포넌트
│   ├── Button Primary
│   ├── Button Secondary
│   ├── Button Outline
│   ├── Input Group
│   ├── Card
│   ├── Sidebar
│   ├── Status Label
│   └── ...
├── 변수
│   ├── 색상 토큰 ($primary, $foreground, $background, ...)
│   ├── 간격 토큰 ($spacing-unit, ...)
│   ├── 타이포그래피 ($font-primary, $font-secondary, ...)
│   └── 반경 ($radius-m, $radius-pill, ...)
└── 테마
    └── mode: [light, dark]
```

---

## Pencil CLI

### 개요

Pencil CLI는 명령줄에서 .pen 파일을 조작할 수 있는 도구입니다.

### 기능

| 기능 | 설명 |
| --- | --- |
| 파일 변환 | .pen 파일을 다른 형식으로 변환 |
| 일괄 처리 | 여러 .pen 파일을 한 번에 처리 |
| 자동화 | CI/CD 파이프라인에 통합 |

> Pencil CLI는 docs.pencil.dev/for-developers/pencil-cli에서 확인할 수 있습니다.

---

## 키보드 단축키

Pencil 인터페이스에서 사용하는 주요 단축키:

### 일반

| 단축키 | 기능 |
| --- | --- |
| `Cmd/Ctrl + K` | AI 프롬프트 패널 열기 |
| `Cmd/Ctrl + Z` | 실행 취소 |
| `Cmd/Ctrl + Shift + Z` | 다시 실행 |

### 선택 및 편집

| 단축키 | 기능 |
| --- | --- |
| `V` | 선택 도구 |
| `R` | 사각형 도구 |
| `T` | 텍스트 도구 |
| `O` | 타원 도구 |
| `P` | 패스 도구 |
| `Delete/Backspace` | 선택 항목 삭제 |
| `Cmd/Ctrl + D` | 복제 |

### 네비게이션

| 단축키 | 기능 |
| --- | --- |
| `Space + 드래그` | 캔버스 이동 |
| `Cmd/Ctrl + =` | 확대 |
| `Cmd/Ctrl + -` | 축소 |
| `Cmd/Ctrl + 0` | 화면에 맞추기 |

---

## 문제 해결

| 문제 | 원인 | 해결 |
| --- | --- | --- |
| 내보내기 빈 화면 | 노드에 콘텐츠 없음 | 내보내기 전 노드 내용 확인 |
| 임포트 실패 | 파일 경로 오류 | 상대 경로 확인 |
| 컴포넌트 연결 끊김 | 파일 간 참조 | 같은 파일 내에 컴포넌트 복사 |
| CLI 인식 불가 | 설치 미완료 | Pencil CLI 설치 확인 |
| MCP 서버 연결 불가 | Pencil 미실행 | Pencil 먼저 실행 |

---

## 가져오기/내보내기 워크플로

### 디자인 시스템 구축

```
1. design-library.pen 생성
2. 공통 컴포넌트 정의 (reusable: true)
3. 변수와 테마 설정
4. 프로젝트 .pen 파일에서 imports로 참조
5. 필요한 컴포넌트를 로컬에 복사하여 사용
```

### 디자인 핸드오프

```
1. .pen 파일에서 화면 노드를 export_nodes로 PNG 내보내기
2. 개발자가 Code 가이드에 따라 React 컴포넌트 생성
3. Tailwind CSS로 스타일링
4. 변수 → CSS 커스텀 프로퍼티 매핑
```
