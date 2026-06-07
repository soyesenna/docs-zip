# Pencil Dev 가져오기, 내보내기, 디자인 라이브러리, CLI

> 원문:
> - https://docs.pencil.dev/core-concepts/import-and-export
> - https://docs.pencil.dev/core-concepts/design-libraries
> - https://docs.pencil.dev/for-developers/pencil-cli

Pencil은 다양한 소스에서 디자인을 가져오고, 여러 형식으로 내보내며, 재사용 가능한 디자인 라이브러리 시스템과 강력한 CLI 도구를 제공합니다.

---

## 가져오기 (Import)

### Complete Figma Files

전체 Figma 파일을 Pencil로 가져올 수 있습니다.

| 방법 | 설명 |
| --- | --- |
| 툴바 | Rectangle 아이콘 아래 펼침(V) 메뉴 클릭 → `Import Figma` 선택 |
| 파일 메뉴 (데스크톱 앱 전용) | `File` > `Import Image/SVG/Figma...` → Figma 파일 선택 |

### Individual Figma Layers

Figma에서 개별 레이어를 복사해서 Pencil 캔버스에 붙여넣을 수 있습니다.

| 방법 | 설명 |
| --- | --- |
| 복사/붙여넣기 | Figma에서 개별 요소를 선택하고 복사한 뒤 Pencil 캔버스에 붙여넣기 |

> **주의**: 이미지 요소의 복사/붙여넣기는 지원되지 않습니다. 전체 Figma 파일을 가져오거나 이미지를 수동으로 추가하세요.

### Images

이미지를 Pencil 캔버스에 가져오는 4가지 방법:

| 방법 | 설명 |
| --- | --- |
| 드래그 앤 드롭 | 컴퓨터에서 이미지를 Pencil 캔버스로 드래그 |
| 복사/붙여넣기 | 클립보드에 복사된 이미지를 캔버스에 붙여넣기 |
| 툴바 | Rectangle 아이콘 아래 펼침 메뉴 → `Import Image or SVG...` |
| 파일 메뉴 (데스크톱 앱 전용) | `File` > `Import Image/SVG/Figma...` → 이미지 선택 |

> **지원 형식**: PNG, JPEG, SVG

### Icons

Pencil은 내장 아이콘 라이브러리와 커스텀 SVG 아이콘 가져오기를 지원합니다.

| 유형 | 설명 |
| --- | --- |
| 내장 라이브러리 | Material Symbols (Outlined, Rounded, Sharp), Lucide Icons, Feather, Phosphor |
| 커스텀 SVG | 개별 이미지와 동일한 방식으로 SVG 아이콘 가져오기 |

---

## 내보내기 (Export)

### Design to Code (AI 기반)

AI를 활용하여 디자인을 코드로 변환합니다.

| 단축키 | 기능 |
| --- | --- |
| `Cmd/Ctrl + K` | AI 채팅 패널을 열어 디자인에서 코드 생성 요청 |

### Individual Elements (UI 기반 내보내기)

하나 이상의 요소를 PNG, JPEG, WEBP, PDF 형식으로 내보냅니다.

| 단계 | 설명 |
| --- | --- |
| 1 | 내보낼 요소를 선택합니다. |
| 2 | 속성 패널(Properties panel) 하단에서 원하는 크기와 형식을 선택합니다. |
| 3 | **Export layer** 버튼을 클릭하고 저장 위치를 지정한 후 **Save**를 클릭합니다. |

| 지원 형식 | 설명 |
| --- | --- |
| PNG | 래스터 이미지 (무손실) |
| JPEG | 래스터 이미지 (손실 압축) |
| WEBP | 차세대 웹 이미지 형식 |
| PDF | 다중 페이지 문서 (모든 노드를 단일 문서로 결합) |

### MCP 도구로 내보내기

`export_nodes` MCP 도구를 사용하여 노드를 이미지 파일로 내보냅니다.

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
| `quality` | JPEG/WEBP 품질 | 1-100. 기본값: JPEG=95, WEBP=100 |
| `outputDir` | 출력 디렉터리 | 필수 |
| `nodeIds` | 내보낼 노드 ID 배열 | 필수. PDF는 모든 노드를 단일 문서로 결합 |

---

## 디자인 라이브러리 (Design Libraries)

### 개념

디자인 라이브러리는 재사용 가능한 컴포넌트를 모아둔 .pen 파일로, 다른 .pen 파일에 임포트할 수 있습니다. 라이브러리 파일에서 컴포넌트를 변경하면 해당 컴포넌트가 사용된 모든 곳에 변경 사항이 반영됩니다.

> "When you make changes to a component in a library file, those changes are reflected everywhere the component is used."

### 라이브러리 생성 (Create a Design Library)

| 단계 | 설명 |
| --- | --- |
| 1 | 새 .pen 파일을 생성합니다. |
| 2 | 컴포넌트를 작성합니다. |
| 3 | 왼쪽 레이어 패널에서 **Libraries** 아이콘을 클릭 → 하단의 **"Turn this file into a library"**를 클릭합니다. |

| 특징 | 설명 |
| --- | --- |
| 파일 접미사 | 디자인 라이브러리 파일은 `.lib.pen` 접미사를 사용합니다. |
| 되돌리기 불가 | 파일을 디자인 라이브러리로 지정하면 취소할 수 없습니다. |

### 라이브러리 임포트 (Import a Library Into a File)

| 단계 | 설명 |
| --- | --- |
| 1 | 왼쪽 레이어 패널에서 **Libraries** 아이콘을 클릭합니다. |
| 2 | 이 파일에 임포트할 라이브러리를 선택합니다. 기본 라이브러리 중에서 선택할 수도 있습니다. |

### 라이브러리 에셋 사용 (Use Design Library Assets)

| 단계 | 설명 |
| --- | --- |
| 1 | 왼쪽 레이어 패널에서 **Assets** 아이콘을 클릭합니다. |
| 2 | 그리드를 스크롤하여 원하는 에셋을 찾거나 이름으로 검색합니다. |
| 3 | 드래그 앤 드롭하거나 클릭하여 캔버스에 배치합니다. |

### 라이브러리 구성 예시

```
design-library.lib.pen
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

### Document imports 필드 (MCP/코드 기반)

Document의 `imports` 필드로 외부 .pen 파일을 참조할 수도 있습니다.

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

---

## Pencil CLI

### 개요

Pencil CLI는 터미널에서 `.pen` 디자인 파일을 생성하고 편집할 수 있는 독립적인 명령줄 도구입니다. 데스크톱 앱과 IDE 확장과 동일한 편집 엔진을 헤드리스(GUI 없이)로 실행합니다.

AI 에이전트에 프롬프트를 전달하여 디자인을 생성/수정하거나, 인터랙티브 셸에서 MCP 도구를 직접 호출하거나, 여러 디자인을 일괄 처리하거나, PNG/JPEG/WEBP/PDF로 내보낼 수 있습니다.

### Installation

```bash
npm install -g @pencil.dev/cli
```

설치 확인:

```bash
pencil version
```

> **요구 사항**: Node.js 18 이상 필요

### Authentication

CLI는 에이전트 작업 실행 전 인증이 필요합니다. 두 가지 방법을 지원합니다.

#### Interactive Login

```bash
pencil login
```

이메일 + 비밀번호 또는 이메일 + OTP 코드로 로그인합니다. 성공 시 세션 토큰이 `~/.pencil/session-cli.json`에 저장됩니다.

#### CLI Key (CI/CD용)

`PENCIL_CLI_KEY` 환경 변수를 설정합니다. CLI 키는 조직에 스코핑되며, Pencil 웹 앱의 조직 설정 **Developer Keys** 섹션에서 생성할 수 있습니다.

```bash
PENCIL_CLI_KEY=pencil_cli_... pencil --out design.pen --prompt "Create a form"
```

> CLI 키는 저장된 세션 토큰보다 항상 우선합니다.

#### Checking Status

```bash
pencil status
```

현재 인증 방식을 표시하고, 백엔드로 세션을 검증하며, 계정 정보를 보여줍니다.

### Quick Start

```bash
# 먼저 로그인
pencil login

# 새 디자인 생성
pencil --out design.pen --prompt "Create a login page with email and password fields"

# 기존 디자인 수정
pencil --in existing.pen --out modified.pen --prompt "Add a blue submit button"

# 디자인을 PNG로 내보내기
pencil --in design.pen --export design.png

# 인터랙티브 셸 시작
pencil interactive -o design.pen

# 사용 가능한 모델 목록
pencil --list-models
```

### Commands

| 명령어 | 설명 |
| --- | --- |
| `pencil login` | 이메일 + 비밀번호 또는 이메일 + OTP로 대화형 로그인 |
| `pencil status` | 인증 상태 확인 및 계정 정보 표시 |
| `pencil version` | 설치된 CLI 버전 출력 |
| `pencil interactive` | 인터랙티브 MCP 도구 셸 시작 |

### Agent Mode

AI 에이전트에 프롬프트를 전달하여 디자인을 생성하거나 수정합니다.

```bash
pencil [options]
```

| 옵션 | 설명 |
| --- | --- |
| `--in, -i <path>` | 입력 `.pen` 파일 (선택 — 생략 시 빈 캔버스로 시작) |
| `--out, -o <path>` | 출력 `.pen` 파일 경로 (`--export` 사용 시 외에 필수) |
| `--prompt, -p <text>` | AI 에이전트에 전달할 프롬프트 |
| `--model, -m <id>` | 사용할 모델 (기본값: `claude-opus-4-6`) |
| `--custom, -c` | 커스텀 Claude 모델 설정 사용 (예: AWS Bedrock, Vertex AI) |
| `--list-models` | 사용 가능한 모델 목록 출력 후 종료 |
| `--tasks, -t <path>` | 배치 작업용 JSON 파일 |
| `--workspace, -w <path>` | 에이전트의 워크스페이스 폴더 경로 |
| `--export, -e <path>` | 최종 결과를 이미지로 내보내기 |
| `--export-scale <n>` | 내보내기 스케일 팩터 (기본값: 1) |
| `--export-type <type>` | 내보내기 형식: `png`, `jpeg`, `webp`, `pdf` (기본값: `png`) |
| `--verbose-mcp` | MCP 도구 오류 상세 정보를 콘솔에 출력 |
| `--help, -h` | 도움말 표시 |

#### Agent Mode 예시

**새 디자인 생성:**

```bash
pencil --out login.pen --prompt "Create a modern login page with:
- Email input field
- Password input field
- Sign In button
- Forgot password link
- Social login options (Google, GitHub)"
```

**기존 디자인 수정:**

```bash
pencil --in dashboard.pen --out dashboard-v2.pen --prompt "Add a sidebar navigation with:
- Dashboard link (active)
- Users link
- Settings link
- Logout button at bottom"
```

**특정 모델 사용:**

```bash
# 빠르고 간단한 작업에 Haiku 모델 사용
pencil --out simple.pen \
  --model claude-haiku-4-5 \
  --prompt "Create a simple 404 error page"
```

**이미지로 내보내기:**

```bash
pencil --in design.pen --export hero.png --export-scale 2
```

### Available Models

| 모델 ID | 설명 | 기본값 |
| --- | --- | --- |
| `claude-opus-4-6` | 가장 강력한 모델, 높은 비용 | 기본값 |
| `claude-sonnet-4-6` | 빠르고 균형 잡힌 성능 | |
| `claude-haiku-4-5` | 가장 빠르고 저렴한 모델 | |

```bash
pencil --list-models
```

### Interactive Mode

인터랙티브 셸에서 `.pen` 파일에 대해 MCP 도구를 직접 호출할 수 있습니다. 스크립팅, 디버깅, 세밀한 제어가 필요한 에이전트 워크플로에 유용합니다.

```bash
pencil interactive [options]
```

| 옵션 | 설명 |
| --- | --- |
| `--app, -a <name>` | 실행 중인 Pencil 앱에 연결 (예: `desktop`, `vscode`) |
| `--in, -i <path>` | 입력 `.pen` 파일 (선택 — 생략 시 빈 캔버스) |
| `--out, -o <path>` | 출력 `.pen` 파일 (헤드리스 모드에서 필수) |
| `--help, -h` | 상세 도구 참조 표시 |

#### App Mode

실행 중인 Pencil 데스크톱 앱 또는 확장에 WebSocket으로 연결합니다. 변경 사항이 실시간으로 적용됩니다.

```bash
pencil interactive -a desktop -i my-design.pen
```

#### Headless Mode

GUI 없이 로컬 편집기를 실행합니다. `save()`로 출력 파일에 저장합니다.

```bash
# 빈 캔버스로 시작
pencil interactive -o output.pen

# 기존 파일 편집
pencil interactive -i input.pen -o output.pen
```

#### Shell Commands

| 명령 | 설명 |
| --- | --- |
| `tool_name({ key: value })` | 인자와 함께 MCP 도구 호출 |
| `tool_name()` | 인자 없이 MCP 도구 호출 |
| `save()` | 문서를 디스크에 저장 |
| `exit()` | 셸 종료 |

#### Example Session

```
pencil > get_editor_state({ include_schema: true })
pencil > get_guidelines()
pencil > get_guidelines({ category: "guide", name: "Landing Page" })
pencil > batch_design({ operations: 'hero=I(document,{type:"frame",name:"Hero",x:0,y:0,width:1440,height:900,fill:"#0A0A0A"})' })
pencil > get_screenshot({ nodeId: "hero" })
pencil > save()
pencil > exit()
```

> `pencil interactive --help`로 전체 도구 참조와 매개변수 타입, 설명을 확인할 수 있습니다.

### Batch Processing

`--tasks` 옵션으로 JSON 파일에 정의된 여러 디자인을 순차적으로 처리합니다. 각 작업은独立的인 편집기 인스턴스에서 실행됩니다.

```bash
pencil --tasks batch.json
```

**batch.json 예시:**

```json
{
  "tasks": [
    {
      "out": "landing-page.pen",
      "prompt": "Create a SaaS landing page with hero, features, and pricing sections"
    },
    {
      "in": "existing-app.pen",
      "out": "existing-app-v2.pen",
      "prompt": "Add a dark mode toggle to the header"
    },
    {
      "out": "mobile-menu.pen",
      "model": "claude-haiku-4-5",
      "prompt": "Create a mobile hamburger menu component"
    }
  ]
}
```

| 필드 | 필수 | 설명 |
| --- | --- | --- |
| `out` | 예 | 출력 `.pen` 파일 경로 |
| `prompt` | 예 | AI 프롬프트 |
| `in` | 아니오 | 입력 `.pen` 파일 |
| `model` | 아니오 | 모델 오버라이드 |

### Supported MCP Tools

CLI는 데스크톱 앱과 IDE 확장과 동일한 MCP 도구를 지원합니다.

#### Design Operations

| 도구 | 설명 |
| --- | --- |
| `batch_design` | 노드 삽입, 업데이트, 삭제, 이동, 복사, 교체 |
| `batch_get` | 패턴 또는 ID로 노드 검색 및 읽기 |
| `get_variables` | 디자인 변수 읽기 |
| `set_variables` | 디자인 변수 업데이트 |
| `get_editor_state` | 문서 메타데이터 및 구조 가져오기 |
| `snapshot_layout` | 계산된 경계로 문서 구조 가져오기 |

#### Visual Operations

| 도구 | 설명 |
| --- | --- |
| `get_screenshot` | 노드를 PNG 이미지로 렌더링 |
| `export_nodes` | 노드를 PNG/JPEG/WEBP/PDF로 내보내기 |

#### Image Generation

`batch_design`의 `G()` 연산은 AI 생성 이미지와 스톡 이미지를 지원합니다.

| 타입 | 설명 |
| --- | --- |
| `G(nodeId, "ai", prompt)` | 텍스트 프롬프트로 AI 이미지 생성 |
| `G(nodeId, "stock", keywords)` | Unsplash에서 스톡 사진 검색 |

#### Style & Guidelines

| 도구 | 설명 |
| --- | --- |
| `get_guidelines` | .pen 파일 작업을 위한 가이드 및 스타일 로드 |

### CI/CD Usage

CLI 키와 Anthropic API 키를 사용하여 자동화 파이프라인에서 Pencil을 실행합니다.

```bash
export PENCIL_CLI_KEY=pencil_cli_...
export ANTHROPIC_API_KEY=sk-ant-...

pencil --out onboarding.pen --prompt "Create a 3-step onboarding flow"
```

### Environment Variables

| 변수 | 설명 |
| --- | --- |
| `PENCIL_CLI_KEY` | CI/CD용 CLI API 키 (저장된 세션보다 우선) |
| `ANTHROPIC_API_KEY` | Anthropic API 키 |
| `PENCIL_API_BASE` | 백엔드 API 베이스 URL (기본값: `https://api.pencil.dev`) |
| `DEBUG` | 디버그 로깅 활성화 |

### Token Storage

| 파일 | 용도 |
| --- | --- |
| `~/.pencil/session-cli.json` | `pencil login`으로 생성된 세션 토큰 |

> CLI는 데스크톱 앱과 별도의 세션 파일을 사용하여 백엔드가 어떤 클라이언트를 사용 중인지 구분할 수 있습니다.

---

## 키보드 단축키

Pencil 인터페이스에서 사용하는 주요 단축키:

### 일반

| 단축키 | 기능 |
| --- | --- |
| `Cmd/Ctrl + K` | AI 프롬프트 패널 열기 (Design to Code) |
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
| Figma 가져오기 실패 | 파일 형식 미지원 | 전체 Figma 파일로 다시 시도 |
| 이미지 붙여넣기 안 됨 | Figma 레이어의 이미지 요소 | 전체 파일 가져오기 또는 수동 이미지 추가 |
| 내보내기 빈 화면 | 노드에 콘텐츠 없음 | 내보내기 전 노드 내용 확인 |
| 라이브러리 변경 미반영 | 라이브러리 임포트 안 됨 | Libraries 패널에서 라이브러리 임포트 확인 |
| CLI 인증 실패 | 세션 만료 | `pencil login`으로 재로그인 |
| CLI 노드 버전 오류 | Node.js 18 미만 | Node.js 18+로 업그레이드 |
| MCP 서버 연결 불가 | Pencil 미실행 | Pencil 먼저 실행 |

---

## 가져오기/내보내기 워크플로

### 디자인 시스템 구축

```
1. design-library.lib.pen 생성
2. 공통 컴포넌트 정의 (reusable: true)
3. 변수와 테마 설정
4. Libraries 아이콘 → "Turn this file into a library" 클릭
5. 다른 .pen 파일에서 Libraries 패널로 임포트
6. Assets 패널에서 컴포넌트를 드래그 앤 드롭으로 사용
```

### 디자인 핸드오프

```
1. .pen 파일에서 화면 노드를 export_nodes로 PNG 내보내기
2. 개발자가 Code 가이드에 따라 React 컴포넌트 생성
3. Tailwind CSS로 스타일링
4. 변수 → CSS 커스텀 프로퍼티 매핑
```

### CLI 자동화

```
1. pencil login으로 인증
2. pencil --out design.pen --prompt "..."로 디자인 생성
3. pencil --in design.pen --export output.png로 이미지 내보내기
4. CI/CD에서 PENCIL_CLI_KEY로 자동 실행
```
