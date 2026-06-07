# Pencil Dev AI 통합

> 원문: https://docs.pencil.dev/getting-started/ai-integration

Pencil은 MCP(Model Context Protocol)를 통한 AI 어시스턴트 통합으로 강력한 디자인 자동화를 제공합니다.

---

## MCP (Model Context Protocol)

### MCP란?

MCP는 AI 어시스턴트가 디자인 파일과 상호작용할 수 있도록 하는 프로토콜입니다. AI가 .pen 파일을 프로그래밍 방식으로 읽고 수정할 수 있는 API 역할을 합니다.

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
- Claude Code CLI 설치 및 인증
- Pencil 실행 중
- `.pen` 파일 열림 상태

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

### Cursor 전용 기능

| 기능 | 설명 |
| --- | --- |
| 인라인 편집 | Pencil에서 요소 선택 → Cursor AI 채팅으로 수정 → .pen 파일에 즉시 반영 |
| 코드베이스 인식 | Cursor가 코드와 디자인을 모두 인식 → 컴포넌트 동기화 요청 가능 |

### 일반적인 문제

| 문제 | 해결 |
| --- | --- |
| "Need Cursor Pro" | Cursor Pro 구독 필요 가능성 |
| 프롬프트 패널 누락 | 활성화/로그인 확인 → Cursor 재시작 → MCP 설정 확인 |

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

---

## MCP 도구 목록

Pencil MCP 서버가 AI 어시스턴트에게 제공하는 도구:

### 디자인 도구

| 도구 | 설명 |
| --- | --- |
| `get_editor_state` | 현재 활성 편집기, 선택 상태, 디자인 정보 조회 |
| `batch_get` | 노드 ID 또는 검색 패턴으로 노드 배치 조회 |
| `batch_design` | JavaScript 스니펫으로 디자인 생성/수정 |
| `get_screenshot` | 특정 노드의 스크린샷 캡처 |
| `export_nodes` | 노드를 PNG/JPEG/WEBP/PDF로 내보내기 |
| `snapshot_layout` | 레이아웃 구조 확인 (클리핑, 겹침 등 문제 탐지) |

### 변수 및 테마 도구

| 도구 | 설명 |
| --- | --- |
| `get_variables` | 문서에 정의된 변수와 테마 조회 |
| `set_variables` | 변수와 테마 업데이트 |

### 가이드 도구

| 도구 | 설명 |
| --- | --- |
| `get_guidelines` | 작업별 가이드와 스타일 프리셋 로드 |

---

## 고급 워크플로

### 자동화된 디자인 생성

AI에게 전체 화면을 설명하면 batch_design API를 통해 자동 생성:

```
"Create a web app dashboard with:
 - Sidebar navigation with 5 items
 - Main content area with 3 metric cards
 - Recent activity table with 5 rows
 - Use the Design System guide"
```

### 디자인 시스템 관리

```
"Create a complete design system with:
 - Button variants (primary, secondary, outline, ghost, destructive)
 - Input fields
 - Cards with header, content, and actions slots
 - Sidebar with navigation items
 - Use semantic color tokens"
```

### 코드-디자인 워크플로

```
"Read the React components in src/components/
 and create matching .pen design components
 with proper variants and design tokens"
```

---

## 모범 사례

### 효과적인 프롬프트

| 원칙 | 설명 |
| --- | --- |
| 구체적으로 | "버튼"보다 "파란색 기본 버튼, 둥근 모서리, 흰색 텍스트" |
| 반복적 디자인 | 한 번에 완벽한 디자인을 요구하지 말고, 점진적으로 개선 |
| 검증 | 작업 후 스크린샷으로 결과 확인 |

### 반복적 디자인

```
1차: "Create a login page"
→ 스크린샷 확인
2차: "Add more spacing between fields and make the button wider"
→ 스크린샷 확인
3차: "Change the color scheme to dark mode"
→ 최종 확인
```

---

## 문제 해결

| 문제 | 원인 | 해결 |
| --- | --- | --- |
| 연결 불가 | Pencil 미실행 | Pencil 먼저 실행 후 AI 도구 사용 |
| 권한 거부 | 폴더 접근 권한 | 접근 프롬프트 수락, 시스템 설정에서 권한 업데이트 |
| AI 출력 품질 저하 | 모호한 프롬프트 | 구체적인 지시, 반복적 접근 |

---

## 핵심 제약사항

AI 어시스턴트를 통해 Pencil을 사용할 때 반드시 알아야 할 제약사항:

### CSS/HTML과의 차이

| 제약 | 설명 |
| --- | --- |
| 커스텀 포맷 | Pencil은 자체 포맷 사용. CSS/HTML 속성이나 동작을 사용/생각하지 마세요 |
| 퍼센트 미지원 | `width: "100%"`, `height: "50%"` 등 사용 불가 |
| margin 미지원 | margin 대신 부모 frame의 padding이나 gap 사용 |
| alignItems 제한 | `baseline`, `stretch` 미지원. `start`, `center`, `end`만 사용 |
| layout/padding | `frame` 타입에서만 설정 가능. 다른 노드에는 설정 불가 |
| 스키마 준수 | 스키마에 없는 속성은 지원되지 않으며 에러 발생 |

### 필수 규칙

| 규칙 | 설명 |
| --- | --- |
| Text fill 필수 | 텍스트 노드는 기본적으로 fill이 없어 보이지 않음. 반드시 fill 설정 |
| placeholder 관리 | 새로 생성/수정 중인 루트 프레임은 `placeholder: true` 유지, 완료 후 즉시 `false` |
| 컴포넌트 참조 | `ref` 노드로 컴포넌트를 재사용. 파일 간 참조 불가 — 복사해서 사용 |
| 노드 ID | `reusable` 노드의 ID는 자동 생성. 수동 설정 불가 |
| batch_design 분할 | 큰 변경은 여러 batch_design 호출로 분할. 에러 시 전체 롤백됨 |

### batch_design API 참조

batch_design은 JavaScript 스니펫으로 디자인을 프로그래밍 방식으로 수정하는 핵심 API입니다:

| 함수 | 설명 | 반환 |
| --- | --- | --- |
| `Insert(parent, data)` | 새 노드 삽입 | 노드 ID |
| `Copy(path, parent, data)` | 노드 복사 | 새 ID |
| `Update(path, data)` | 속성 업데이트 | void |
| `Replace(path, data)` | 노드 교체 | 새 ID |
| `Move(path, parent, idx?)` | 위치 이동 | void |
| `Delete(path)` | 삭제 | void |
| `Generate(nodeId, type, prompt)` | 이미지 생성 | void |
| `FindEmptySpace({w,h,...})` | 빈 영역 찾기 | `{x,y}` |

> 상세한 API 문서는 [batch_design API](./10-batch-design-api.md)를 참조하세요.
