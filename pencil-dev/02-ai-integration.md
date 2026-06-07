# Pencil Dev AI 통합

> 원문: https://docs.pencil.dev/getting-started/ai-integration

Pencil은 MCP(Model Context Protocol)를 통한 AI 어시스턴트 통합으로 강력한 디자인 자동화와 워크플로를 제공합니다.

---

## 지원 AI 어시스턴트

Pencil은 MCP를 통해 다음 AI 도구들과 연동됩니다:

| AI 어시스턴트 | 설명 |
| --- | --- |
| **Claude Code** (CLI + IDE) | Anthropic Claude 기반 CLI 및 IDE 내장 패널 |
| **Claude Desktop** | Claude 데스크톱 애플리케이션 |
| **Cursor** | AI 기반 IDE |
| **Windsurf IDE** (Codeium) | Codeium에서 제공하는 AI IDE |
| **Codex CLI** (OpenAI) | OpenAI 기반 명령줄 도구 |
| **Antigravity IDE** | Pencil 내장 IDE |
| **OpenCode CLI** | 오픈소스 CLI 도구 |

---

## MCP (Model Context Protocol)

### MCP란?

MCP는 AI 어시스턴트가 디자인 파일과 상호작용할 수 있도록 하는 프로토콜입니다. AI가 `.pen` 파일을 프로그래밍 방식으로 읽고 수정할 수 있는 API 역할을 합니다.

### 작동 방식

| 단계 | 설명 |
| --- | --- |
| 로컬 실행 | Pencil MCP 서버는 로컬에서 실행 (클라우드 의존 없음) |
| 연결 | AI 어시스턴트가 Pencil 실행 중 MCP를 통해 연결 |
| 도구 사용 | AI가 도구를 사용해 디자인 읽기, 수정, 생성 |
| 사용자 제어 | AI가 제안하고, 사용자가 승인 |

### 보안 및 개인정보

| 항목 | 설명 |
| --- | --- |
| 로컬 전용 | MCP 서버는 사용자의 로컬 머신에서만 실행 |
| 원격 접근 불가 | 디자인 파일은 로컬에 유지 |
| 저장소 비공개 | 소스 코드는 현재 공개되지 않음 |
| 도구 검사 | IDE 설정에서 사용 가능한 도구 확인 가능 |

---

## Claude Code 연동

### 설정

**사전 요구사항:**
- Claude Code CLI 설치
- 인증: `claude`
- Pencil 실행 중
- `.pen` 파일 열림 상태

### Antigravity/VSCode에서 Claude Code 패널 사용

Antigravity IDE 또는 VSCode에서 Pencil 확장과 함께 Claude Code 패널을 사용할 수 있습니다. Pencil이 실행 중이면 Claude Code 패널이 MCP를 통해 자동으로 Pencil 도구에 접근합니다.

> **Variables and Design Kits Tutorial** — [Vimeo 영상 보기](https://player.vimeo.com/video/1158682767)

### 기본 워크플로

1. AI 프롬프트 패널 열기: `Cmd/Ctrl + K`
2. 디자인 요청:
   - "Create a login form with email and password"
   - "Add a navigation bar to this page"
   - "Design a card component for my design system"
3. AI가 MCP 도구로 `.pen` 파일 수정
4. 캔버스에 즉시 반영

### 예시 프롬프트

**디자인 생성:**

| 프롬프트 | 설명 |
| --- | --- |
| "Design a dashboard with sidebar and main content area" | 대시보드 레이아웃 |
| "Create a pricing table with 3 tiers" | 가격표 |
| "Add a hero section with heading and CTA button" | 히어로 섹션 |

**디자인 수정:**

| 프롬프트 | 설명 |
| --- | --- |
| "Change all primary buttons to blue" | 색상 일괄 변경 |
| "Make the sidebar narrower" | 크기 조정 |
| "Add spacing between these elements" | 간격 추가 |

**디자인 시스템:**

| 프롬프트 | 설명 |
| --- | --- |
| "Create a button component with variants" | 컴포넌트 생성 |
| "Generate a color palette based on #3b82f6" | 색상 팔레트 |
| "Build a typography scale" | 타이포그래피 스케일 |

**코드 통합:**

| 프롬프트 | 설명 |
| --- | --- |
| "Generate React code for this component" | React 코드 생성 |
| "Import the Header from my codebase" | 코드 가져오기 |
| "Create Tailwind config from these variables" | Tailwind 설정 |

---

## Cursor 연동

### 설정

1. Cursor에 Pencil 확장 설치
2. 활성화 완료
3. Claude Code 인증
4. MCP 연결 확인: Settings → Tools & MCP

### Cursor에서 Pencil 확장 사용

Cursor에서 Pencil 확장을 설치하고 활성화하면 MCP를 통해 자동으로 Pencil 도구에 접근합니다.

> **Using Pencil Extension in Cursor** — [Vimeo 영상 보기](https://player.vimeo.com/video/1158019699)

### Cursor 전용 기능

| 기능 | 설명 |
| --- | --- |
| 인라인 편집 | Pencil에서 요소 선택 → Cursor AI 채팅으로 수정 → `.pen` 파일에 즉시 반영 |
| 코드베이스 인식 | Cursor가 코드와 디자인을 모두 인식 → 컴포넌트 동기화 요청 가능, 자동으로 일관성 유지 |

### 일반적인 문제

| 문제 | 해결 |
| --- | --- |
| "Need Cursor Pro" | 일부 기능은 Cursor Pro 구독이 필요할 수 있음. Cursor 가격 페이지에서 현재 제한 확인 |
| 프롬프트 패널 누락 | 활성화/로그인 확인 → Cursor 재시작 → MCP 설정에서 연결 확인 |

---

## Codex CLI 연동

### 설정

1. Pencil 먼저 실행 (데스크톱 앱 또는 IDE 확장)
2. 터미널에서 Codex 실행
3. MCP 연결 확인: `/mcp` → Pencil이 목록에 나타나야 함

### 사용 예시

```
# Codex CLI에서
> Create a button component in design.pen
> Add a hero section to the landing page
> Generate a color scheme based on blue
```

### 장점

| 장점 | 설명 |
| --- | --- |
| 명령줄 워크플로 | 터미널에서 직접 디자인 생성 및 수정 |
| 스크립트 가능한 디자인 생성 | 스크립트를 통한 반복적 디자인 자동화 |
| 빌드 도구 통합 | 빌드 파이프라인과 연동 가능 |

### 알려진 이슈

| 이슈 | 설명 |
| --- | --- |
| `config.toml` 수정 문제 | Pencil이 Codex의 `config.toml`을 수정하거나 복제할 수 있음. 공식적으로 인지된 이슈이며 조사 중. 최초 사용 전 설정 파일 백업 권장 |

---

## MCP 도구 목록

AI 어시스턴트가 Pencil MCP 서버에 연결하면 다음 도구에 접근할 수 있습니다:

### Design Tools

| 도구 | 설명 |
| --- | --- |
| `batch_design` | 디자인 요소 생성, 수정, 조작. Insert, Copy, Update, Replace, Move, Delete 작업 및 이미지 생성/배치 |
| `batch_get` | 노드 ID 또는 검색 패턴으로 디자인 컴포넌트와 계층 구조 조회. 요소 검색 및 컴포넌트 구조 검사 |

### Analysis Tools

| 도구 | 설명 |
| --- | --- |
| `get_screenshot` | 디자인 미리보기 렌더링, 시각적 출력 검증, 변경 전후 비교 |
| `snapshot_layout` | 레이아웃 구조 분석, 포지셔닝 이슈 탐지, 겹치는 요소 찾기 |
| `get_editor_state` | 현재 편집기 컨텍스트, 선택 정보, 활성 파일 상세 조회 |

### Variables & Theming

| 도구 | 설명 |
| --- | --- |
| `get_variables` / `set_variables` | 디자인 토큰 읽기, 테마 값 업데이트, CSS와 동기화 |

### And More

각 IDE에서 사용 가능한 전체 도구 목록을 확인하는 방법:

| IDE | 확인 방법 |
| --- | --- |
| Cursor | Settings → Tools & MCP |
| VS Code | MCP 설정 확인 |
| Codex | `/mcp` 실행 후 Pencil 도구 검사 |

---

## 고급 워크플로

### 자동화된 디자인 생성

**Style guides:** Ask AI to follow specific design systems:

```
"Create a dashboard using Material Design principles"
"Design a landing page with modern, minimal aesthetics"
"Build components following our design system in design-system.pen"
```

**Batch operations:**

```
"Create 5 variations of this button component"
"Generate a complete form with all input types"
"Design an entire landing page with hero, features, pricing, and footer"
```

### 디자인 시스템 관리

**일관성 유지:**

```
"Ensure all buttons use the primary color variable"
"Update all headings to use the typography scale"
"Apply 8px spacing grid to all elements"
```

**컴포넌트 라이브러리:**

```
"Create a complete button component with all variants"
"Generate form input components (text, select, checkbox, radio)"
"Build a card component with image, title, description, and actions"
```

### 코드-디자인 워크플로

**기존 앱 가져오기:**

```
"Recreate all components from src/components in Pencil"
"Import the design system from our Tailwind config"
"Analyze the codebase and create matching designs"
```

**변경 동기화:**

```
"Update all React components to match the Pencil designs"
"Apply the new color scheme to both design and code"
"Sync typography variables between CSS and Pencil"
```

---

## 모범 사례

### 효과적인 프롬프트

**구체적으로 작성 (Be specific):**

| 프롬프트 | 평가 |
| --- | --- |
| ~~"Make it better"~~ | 모호함 |
| "Increase the button padding to 16px and change color to blue" | 구체적 |

**컨텍스트 제공 (Provide context):**

| 프롬프트 | 평가 |
| --- | --- |
| ~~"Add a form"~~ | 모호함 |
| "Add a login form with email, password, remember me checkbox, and submit button" | 구체적 |

**디자인 시스템 참조 (Reference design systems):**

| 프롬프트 예시 | 설명 |
| --- | --- |
| "Use our existing button component" | 기존 컴포넌트 재사용 |
| "Follow the spacing scale from our variables" | 변수에 정의된 간격 스케일 적용 |
| "Match the style of the header component" | 헤더 컴포넌트 스타일 일치 |

### 반복적 디자인

```
1차: "Create a dashboard layout"           → 시작
→ 스크린샷 확인
2차: "Add a sidebar with navigation items"  → 보완
→ 스크린샷 확인
3차: "Style the nav items with hover states" → 디테일
→ 스크린샷 확인
4차: "Adjust spacing to match 8px grid"     → 마무리
→ 최종 확인
```

### 검증

AI가 변경을 수행한 후:

1. 캔버스에서 시각적으로 검토
2. 레이어 패널에서 구조 확인
3. 필요시 인터랙션 테스트
4. 복잡한 레이아웃은 스크린샷으로 검증 요청

---

## 문제 해결

### 연결 이슈

| 문제 | 원인 | 해결 |
| --- | --- | --- |
| "Claude Code not connected" | Claude Code 미로그인 | `claude`로 로그인 → Pencil 재시작 → 프로젝트 디렉토리에서 `claude` 실행 |
| MCP 서버 미표시 | Pencil 미실행 | Pencil 실행 확인 → IDE MCP 설정 확인 → Pencil과 AI 어시스턴트 모두 재시작 |

### 권한 이슈

| 문제 | 원인 | 해결 |
| --- | --- | --- |
| 폴더 접근 불가 | 폴더 접근 권한 없음 | 접근 프롬프트 수락 → 시스템 폴더 권한 확인 → IDE/Pencil을 올바른 권한으로 실행 |
| 권한 프롬프트 미표시 | 알림 설정 | 별도 Claude Code 세션에서 시도 → 알림 설정 확인 → IDE 권한 확인 |

### AI 출력 이슈

| 문제 | 원인 | 해결 |
| --- | --- | --- |
| "Invalid API key" | 인증 만료 | `claude`로 재인증 → 충돌하는 인증 설정 확인 → 환경 변수 초기화 |
| AI가 예상치 못한 변경 | 모호한 프롬프트 | 프롬프트를 더 구체적으로 → 적용 전 설명 요청 → 버전 관리로 필요시 되돌리기 |

---

## Example Session

실제 Pencil + AI 어시스턴트 사용 예시입니다:

```bash
# 1. Pencil과 Claude Code 시작
claude

# 2. IDE에서 design.pen 열기
# 3. Cmd + K를 눌러 디자인 시작
```

| 단계 | 사용자 입력 | AI 응답 |
| --- | --- | --- |
| **디자인 생성** | "Create a modern landing page hero section" | 헤딩, 서브헤딩, CTA 버튼이 포함된 히어로 섹션 생성 |
| **섹션 추가** | "Add a features section with 3 columns" | 히어로 아래에 피처 섹션 추가 |
| **변수 적용** | "Use our primary color variable for the CTA buttons" | 버튼에 색상 변수 적용 |
| **코드 생성** | "Generate React code for this entire page" | Tailwind CSS가 적용된 React 컴포넌트로 내보내기 |

```bash
# 4. 검토 및 보완
# 5. Git에 커밋
git add design.pen src/pages/landing.tsx
git commit -m "Add landing page design and implementation"
```

