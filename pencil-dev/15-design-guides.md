# Pencil Dev 디자인 가이드와 스타일

> 원문: Pencil MCP 서버 가이드 8종 (Web App, Mobile App, Landing Page, Slides, Table, Tailwind, Code, Design System), https://docs.pencil.dev/troubleshooting
>
> 문제 해결 섹션 원문: https://docs.pencil.dev/troubleshooting (Last updated: June 1, 2026)

Pencil은 다양한 프로젝트 유형에 맞는 내장 디자인 가이드와 27개의 스타일 프리셋을 제공합니다.

---

## 내장 가이드

### 가이드 목록

| 가이드 | 용도 | 로드 방법 |
| --- | --- | --- |
| **Code** | .pen에서 코드 생성 | `get_guidelines({ category: "guide", name: "Code" })` |
| **Design System** | 디자인 시스템 컴포넌트 구성 | `get_guidelines({ category: "guide", name: "Design System" })` |
| **Landing Page** | 랜딩 페이지 디자인 | `get_guidelines({ category: "guide", name: "Landing Page" })` |
| **Mobile App** | 모바일 앱 화면 | `get_guidelines({ category: "guide", name: "Mobile App" })` |
| **Slides** | 프레젠테이션 슬라이드 | `get_guidelines({ category: "guide", name: "Slides" })` |
| **Table** | 테이블/대시보드 | `get_guidelines({ category: "guide", name: "Table" })` |
| **Tailwind** | Tailwind CSS v4 구현 | `get_guidelines({ category: "guide", name: "Tailwind" })` |
| **Web App** | 웹 애플리케이션 UI | `get_guidelines({ category: "guide", name: "Web App" })` |

---

## Web App 디자인 원칙

웹 애플리케이션 UI 설계 시 따라야 할 16가지 원칙:

### 핵심 원칙

| # | 원칙 | 설명 |
| --- | --- | --- |
| 1 | 목적 우선 | 모든 화면은 명확한 기본 목적을 가져야 함 |
| 2 | 지배적 영역 | 하나의 지배적 시각 영역 포함. 동일 비중 레이아웃 지양 |
| 3 | 이해 가능성 | 라벨은 명확, 액션은 인식 가능, 아이콘은 텍스트를 대체하지 않음 |
| 4 | 점진적 공개 | 복잡성을 점진적으로 드러냄. 고급 기능은 문맥적 |
| 5 | 인식 > 회상 | 관련 액션을 필요할 때 표시. 사용자가 이전 상태를 기억하게 하지 않음 |
| 6 | 시스템 상태 가시성 | Loading, Empty, Error, Success 상태를 항상 표시 |

### 구조 원칙

| # | 원칙 | 설명 |
| --- | --- | --- |
| 7 | 액션 계층 | 화면/섹션당 하나의 기본 액션. 파괴적 액션은 명확히 구분 |
| 8 | 구조적 일관성 | 유사한 문제 → 유사한 해결책. 간격은 일관된 스케일 |
| 9 | 밀도 의도성 | Compact/Medium/Airy 중 하나를 선택해 화면 내에서 혼합 금지 |
| 10 | 공간 논리 | 화면당 하나의 지배적 축. 불필요한 중첩 스크롤 회피 |
| 11 | 피드백 | 모든 사용자 액션에 즉각적 확인. 파괴적 작업은 확인 요청 |

### 확장 원칙

| # | 원칙 | 설명 |
| --- | --- | --- |
| 12 | 반응형 | 모든 중단점에서 계층 유지. 모바일은 단일 열 |
| 13 | 엔티티 무결성 | 엔티티의 이름, 상태, 메타데이터를 명확히 표시 |
| 14 | 제약 > 장식 | 네비게이션/이해/의사결정/액션에 기여하지 않는 요소 제거 |
| 15 | 확장성 | 더 많은 데이터/기능이 구조를 깨지 않아야 함 |
| 16 | 적응 논리 | 제품 유형에 따라 지배 영역, 기본 액션, 밀도를 추론 |

---

## Mobile App 가이드

### 화면 구조

```
Screen (vertical layout, fit_content(844) height)
├── Status Bar — OS 크롬 (62px, Inter, 세로 중앙 정렬)
├── Content Wrapper — 좌우 패딩 16–20px, gap 24–32px
└── Tab Bar (옵션) — 하단 고정 캡슐 탭바
```

### Tab Bar (iOS "Liquid Glass" 스타일)

| 속성 | 값 |
| --- | --- |
| 위치 | 화면 가장자리에서 ~16px 들어감, 하단 ~12px |
| 높이 | ~56px |
| 모서리 | 높이의 절반 (진정한 캡슐) |
| 내부 패딩 | ~6px |
| 배경 | 프로스티드 글래스 — 70% 불투명도, 부드러운 섀도 |
| 아이템 | 둥근 아이콘(~22px) + 라벨(10px) |
| 선택됨 | 액센트 색상, 채워진 아이콘, 부드러운 캡슐 하이라이트 |

### 모바일 원칙

| 원칙 | 설명 |
| --- | --- |
| 단일 의도 | 화면당 하나의 기본 의도 |
| 도달성 | 핵심 액션은 한 손 사용을 위해 하단 절반에 |
| 터치 타겟 | 충분한 터치 영역 |
| 패딩 | 래퍼가 수평 패딩 처리 — 섹션별 패딩 추가 금지 |

---

## Landing Page 가이드

### 핵심 원칙

| 원칙 | 설명 |
| --- | --- |
| 전환 의도 | 모든 요소가 하나의 액션(가입, 구매 등)으로 이동 |
| 변환 > 기능 | 도구가 아닌 결과를 보여줌 |
| 히어로 = 전체 피치 | 첫 화면에 전체 피치를 압축 |
| 헤드라인 강도 | 변환형 > 결과형 > 혜택형 > 기능형 |

### 구조

| 요소 | 설명 |
| --- | --- |
| 정렬 | 섹션당 하나의 정렬 축. 섹션 내에서 혼합 금지 |
| 리듬 | 텍스트 중심과 시각적 섹션을 교대로. 비슷한 밀도 연속 금지 |
| 타이포그래피 | 본문 14px 이상. 2–3줄 이상 중앙 정렬 금지. 행 길이 50–75자 |
| 섹션 흐름 | 약속 → 증명 → 액션의 서사 구조 |

---

## Slides 가이드

### 핵심 규칙

| 규칙 | 설명 |
| --- | --- |
| 한 슬라이드 = 한 아이디어 | 슬라이드는 시각 보조 도구, 문서가 아님 |
| 폰트 | 본문 ≥24px, 제목 ≥40px |
| 포맷 | 16:9, 1920×1080 |
| 여백 | 콘텐츠는 가장자리에서 ≥100px |

### 레이아웃 컨트랙트

| ID | 용도 | 구조 |
| --- | --- | --- |
| L01 | Cover | 중앙: 제목(48-64) + 부제(28-32) |
| L03 | Section Break | 중앙: 라벨(24) + 제목(48-56) |
| L05 | Concept+Visual | 2col: 텍스트 | 이미지 |
| L07 | 3 Pillars | 3col: 시각+라벨+설명 |
| L09 | Single KPI | 중앙: 라벨 + 숫자(120-200) |
| L13 | Process | 행: 3-5단계 (아이콘+라벨+설명) |
| L20 | Closing | 중앙: 헤드라인 + 부제 + 연락처 |

---

## Table 가이드

### 엄격한 계층

```
Table (frame, vertical)
├── Header Row (frame, horizontal, fill_container, 고정 높이)
│   └── Cell (frame) → Text (bold, fixed-width)
├── Data Row (frame, horizontal, fill_container, 고정 높이)
│   └── Cell (frame) → Text (fixed-width)
└── ...
```

| 규칙 | 설명 |
| --- | --- |
| Cell은 항상 frame | 콘텐츠를 직접 Row에 넣지 마세요 |
| 더미 데이터 | 사용자가 데이터를 지정하지 않으면 더미 값 생성 |
| 반응형 | 넓은 테이블은 모바일에서 카드로 변환 고려 |

---

## 스타일 프리셋

Pencil은 27개의 스타일 프리셋을 제공합니다:

| 스타일 | 분위기 |
| --- | --- |
| Aerial Gravitas | 웅장, 중후 |
| Anchored Ribbon Grid | 리본 그리드 |
| Artisan Editorial | 장인 에디토리얼 |
| Blueprint Technical | 기술 청사진 |
| Centered Device Cascade | 중앙 기기 캐스케이드 |
| Centered Serif List | 중앙 세리프 리스트 |
| Cinematic Alternating | 시네마틱 교대 |
| Cinematic Device Column | 시네마틱 기기 컬럼 |
| Color Block Stack | 컬러 블록 스택 |
| Dark Centered Platform | 다크 중앙 플랫폼 |
| Editorial Landscape Stack | 에디토리얼 풍경 스택 |
| Editorial Scientific | 학술 에디토리얼 |
| Gradient Prompt Stack | 그래디언트 프롬프트 스택 |
| Illustrated Ribbon Stack | 일러스트 리본 스택 |
| Illustrated Warm | 따뜻한 일러스트 |
| Inline Friendly | 인라인 친근 |
| Modular Bento Showcase | 모듈 벤토 쇼케이스 |
| Monumental Editorial | 기념비적 에디토리얼 |
| Narrative Illustrated | 내러티브 일러스트 |
| Product Data Grid | 제품 데이터 그리드 |
| Product Demo | 제품 데모 |
| Saturated Code Bridge | 채도 코드 브릿지 |
| Soft Bento | 부드러운 벤토 |
| Spatial Plus | 공간 플러스 |
| Split Inverse Showcase | 분할 역전 쇼케이스 |
| Zigzag Bold Split | 지그재그 볼드 분할 |

### 스타일 로드

```javascript
get_guidelines({ category: "style", name: "Soft Bento" })
```

> 스타일은 변수를 저장하지 않고 참조 값만 제공합니다.

---

## 문제 해결

> 공식 문서 기준: https://docs.pencil.dev/troubleshooting (2026년 6월 1일 업데이트)

---

### Installation & Setup

#### 확장 프로그램이 설치되었으나 연결되지 않음

**증상:**

- Pencil 확장 프로그램이 설치되었으나 작동하지 않음
- 편집기에 Pencil 아이콘이 보이지 않음
- `.pen` 파일을 열 수 없음

**해결 방법:**

| 순서 | 작업 |
| --- | --- |
| 1 | Claude Code 로그인 확인: 터미널에서 `claude` 실행 |
| 2 | 활성화 프로세스 완료 (이메일에서 코드 확인) |
| 3 | Pencil MCP 서버 연결 확인 (Settings → MCP) |
| 4 | `.pen` 파일을 생성하고 열어보기 |
| 5 | IDE 재시작 |
| 6 | 문제가 지속되면 확장 프로그램 재설치 |

#### Pencil 아이콘이 보이지 않음

| 순서 | 작업 |
| --- | --- |
| 1 | `.pen` 파일 생성 (예: `test.pen`) |
| 2 | 파일 열기 |
| 3 | Extensions 패널에서 확장 버전 확인 |
| 4 | 명령 팔레트 열기 (`Cmd/Ctrl + Shift + P`) → "Pencil" 검색 |
| 5 | 확장 프로그램이 활성화되어 있는지 확인 |

---

### Authentication & Activation

#### 활성화 이메일을 받지 못함

| 해결 방법 | 설명 |
| --- | --- |
| 스팸/정크 폴더 확인 | 이메일이 스팸으로 분류되었을 수 있음 |
| 대기 | 이메일이 지연될 수 있으니 몇 분 기다림 |
| 다른 이메일 주소 사용 | 다른 주소로 시도 |
| 확장 재설치 | 활성화가 멈춘 경우 재설치 시도 |

#### "Invite for your email address was not found"

활성화 과정에서 발생할 수 있는 문제:

1. Cursor/VS Code 확장 프로그램 재설치
2. 활성화 다시 시도
3. 계속되면 고객 지원에 문의

#### 반복해서 활성화 프롬프트가 표시됨

일부 버전에서 발생하는 알려진 이슈:

1. IDE 재시작
2. 활성화를 한 번 더 완료
3. 계속되면 확장 프로그램 재설치

#### Claude Code 연결 문제

**시나리오 1: "Claude Code isn't connected / not logged in"**

| 순서 | 작업 |
| --- | --- |
| 1 | 프로젝트 디렉터리에서 터미널 열기 |
| 2 | `claude` 실행 후 인증 완료 |
| 3 | Pencil로 돌아가기 |
| 4 | `.pen` 파일 열기/편집 시도 |

**시나리오 2: "Invalid API key" 또는 "Please run /login"**

| 원인 | 설명 |
| --- | --- |
| 불완전한 인증 | 인증이 완료되지 않음 |
| 인증 방식 충돌 | CLI + 환경 변수 + 커스텀 프로바이더 충돌 |
| 만료된 자격 증명 | 인증 토큰이 만료됨 |

**해결:**

1. `claude` CLI 실행 후 인증
2. 충돌 확인:
   - CLI 사용 시 `ANTHROPIC_API_KEY` 환경 변수 제거
   - 커스텀 프로바이더 설정 초기화
   - 깨끗한 로그인 세션 시도
3. 인증 후 IDE 재시작

**시나리오 3: Claude CLI는 작동하지만 Pencil이 로그인되지 않았고 표시함**

1. 서드파티/커스텀 프로바이더 충돌 확인
2. 깨끗한 Claude Code 세션 시도
3. IDE 재시작
4. 환경 변수 충돌 확인

---

### IDE-Specific Issues

#### Cursor

| 문제 | 해결 방법 |
| --- | --- |
| 프롬프트 편집기/프롬프트 박스가 보이지 않음 | 활성화/로그인 상태 확인 → Cursor 재시작 → 설정에서 MCP 연결 확인 → 필요시 확장 재설치 |
| "You need Cursor Pro" | 일부 기능은 Cursor Pro 구독이 필요할 수 있음. Cursor의 현재 플랜 제한 확인. Pro 없이 기본 기능 시도 |

#### VS Code

**확장 프로그램이 활성화되지 않음:**

| 순서 | 작업 |
| --- | --- |
| 1 | `.pen` 파일이 열려 있는지 확인 |
| 2 | 확장 프로그램이 활성화 상태인지 확인 |
| 3 | Developer Tools Console에서 오류 확인 |
| 4 | 재설치 시도 |

---

### Welcome File & Onboarding

#### 웰컴 파일을 받지 못함

1. 캔버스에서 아무 곳이나 우클릭
2. **Open Welcome File** 선택
3. 별도 창에서 열릴 수 있음

#### 새 프로젝트에 Pencil 추가하기

**권장 흐름:**

1. 워크스페이스에 `design.pen` 같은 파일 생성
2. IDE(Cursor/VS Code)에서 열기
3. Pencil 활성화
4. AI가 코드와 디자인 모두 볼 수 있도록 `.pen` 파일을 프로젝트에 함께 유지

---

### Canvas & Interface

#### 중첩 요소를 탐색할 수 없음

**키보드 단축키:**

| 단축키 | 동작 |
| --- | --- |
| `Cmd/Ctrl + Click` | 직접 선택 (가장 깊은 요소) |
| `Shift + Enter` | 부모 선택 |
| `Cmd/Ctrl + Enter` | 부모 선택 (대체) |

**레이어 패널 사용:**

- 인터페이스에서 사용 가능
- 전체 계층 구조 표시
- 클릭하여 요소 선택

#### 선택 박스 색상이 헷갈림

| 색상 | 의미 |
| --- | --- |
| **파란색 (Blue)** | 일반 요소 |
| **보라색/마젠타 (Purple/Magenta)** | 재사용 가능한 컴포넌트/심볼 |

#### 컴포넌트로 변환해도 보라색 박스가 보이지 않음

`Cmd/Ctrl + Option/Alt + K` 로 변환 후:

1. 요소 선택 해제
2. 다시 선택
3. 보라색 하이라이트가 나타나야 함

---

### Importing & Exporting

#### macOS에서 이미지 가져오기 실패

| 항목 | 내용 |
| --- | --- |
| 문제 | File 메뉴 드롭다운을 사용하면 실패할 수 있음 |
| 해결 | **드래그 앤 드롭** 사용. 모든 플랫폼에서 안정적으로 작동 |

#### Figma에서 이미지가 붙여넣기되지 않음

| 항목 | 내용 |
| --- | --- |
| 제한 | Figma에서 복사-붙여넣기 시 이미지가 지원되지 않음 |
| 해결 | SVG를 사용하거나 이미지를 개별적으로 다시 가져오기 |

#### Claude Code로 내보내기 실패

**"Process exited with code 1" 오류:**

| 원인 | 설명 |
| --- | --- |
| Claude Code 설정 이슈 | 구성 문제 |
| 인증 문제 | 인증이 완료되지 않음 |
| 권한 문제 | 접근 권한 부족 |

**해결:**

1. Claude Code 인증 확인: `claude` 실행
2. 권한 확인
3. 별도의 Claude Code 세션에서 내보내기 시도
4. 환경 변수 확인

#### 내보낸 결과가 캔버스와 일치하지 않음

| 항목 | 내용 |
| --- | --- |
| 상태 | 캔버스와 내보내기 간 시각적 불일치. 버그일 수 있음 |
| 해결 1 | 참조용으로 캔버스 스크린샷 촬영 |
| 해결 2 | 다시 내보내기 시도 |
| 해결 3 | 디자인을 조정 후 다시 내보내기 |
| 해결 4 | 해당 사례를 보고하여 조사 요청 |

---

### MCP & AI Integration

#### MCP 서버 이슈

**MCP 서버가 로컬인가요, 원격인가요?**

- MCP 서버는 **로컬**에서 실행됨
- 디자인 작업에 클라우드 종속성 없음

**MCP 연결 확인:**

| 환경 | 확인 방법 |
| --- | --- |
| Cursor | Settings → Tools & MCP |
| VS Code | MCP 구성 확인 |
| Codex | `/mcp` 실행 → Pencil이 나타나야 함 |

#### Codex config.toml 수정

| 항목 | 내용 |
| --- | --- |
| 문제 | Pencil이 Codex config를 수정하거나 복제할 수 있음 |
| 해결 | 최초 사용 전 `config.toml` 백업 |
| 상태 | 공식적으로 인지된 이슈이며 조사 중 |

#### AI가 폴더에 접근할 수 없음

**"Pencil / Claude can't access other folders" 해결:**

1. 폴더 권한 제한 확인
2. 접근 권한 프롬프트가 나타나면 수락
3. 시스템 설정에서 권한 업데이트
4. 적절한 권한으로 IDE/Pencil 실행

**권한 프롬프트가 나타나지 않은 경우:**

1. Pencil 외부의 별도 세션에서 작업 실행
2. 시스템 알림 설정 확인
3. IDE에 필요한 권한이 있는지 확인

---

### Saving & Version Control

#### 자동 저장 없음

| 항목 | 내용 |
| --- | --- |
| 현재 상태 | 자동 저장 미지원. 향후 릴리스 예정 |
| 해결 1 | `Cmd/Ctrl + S`로 자주 저장 |
| 해결 2 | Git 커밋으로 버전 이력 관리 |
| 해결 3 | 정기 저장 알림 설정 |

#### 제한적인 undo/redo

| 항목 | 내용 |
| --- | --- |
| 제한 | 일반 디자인 편집기보다 undo/redo가 더 제한적일 수 있음 |
| 권장 | 주요 변경 전 저장, Git에 자주 커밋, Git 이력으로 복구, 증분 변경 수행 |

#### 실시간 협업 없음

| 항목 | 내용 |
| --- | --- |
| 현재 상태 | 실시간 멀티플레이어 미지원. Git을 통한 협업만 가능 |
| 워크플로 | Git 브랜치로 병렬 작업 → Pull, Edit, Commit, Push, PR 생성 → PR에서 디자인 리뷰 |

---

### Performance & UI

#### Wayland/Hyprland UI 이슈 (Linux)

| 항목 | 내용 |
| --- | --- |
| 상태 | Wayland/Hyprland에서 일부 UI 이슈 발생 |
| 해결 1 | 가능하면 X11 환경 사용 |
| 해결 2 | 특정 이슈를 보고하여 조사 요청 |
| 해결 3 | 데스크톱 앱이 더 나은 호환성을 보일 수 있음 |

---

### Platform-Specific

#### Windows 데스크톱 앱

| 항목 | 내용 |
| --- | --- |
| 상태 | 현재 사용 불가 |
| 대안 | VS Code 확장 프로그램 또는 Cursor 확장 프로그램 설치. macOS/Linux와 동일하게 작동 |

#### Linux 데스크톱 앱

| 항목 | 내용 |
| --- | --- |
| 상태 | Linux 지원 가능 |
| 참고 | 일부 Wayland/Hyprland 이슈 발생 가능 |
| 권장 | X11 환경이 더 안정적. 데스크톱 앱을 먼저 시도, 필요시 IDE 확장으로 전환 |

---

### Common Error Messages

| 에러 메시지 | 주요 원인 | 해결 방법 |
| --- | --- | --- |
| "Claude Code not connected" | 로그인되지 않음 | `claude` CLI 실행 |
| "Invalid API key" | 인증 충돌 | 환경 변수 제거 후 재인증 |
| "Process exited with code 1" | 내보내기/인증 이슈 | Claude Code 설정 확인 |
| "Need Cursor Pro" | 플랜 제한 | Cursor 구독 확인 |
| "Invite not found" | 활성화 이슈 | 확장 프로그램 재설치 |
| "Can't access folder" | 권한 문제 | 폴더 권한 업데이트 |

---

### Prevention Tips

공통 이슈 예방 수칙:

| 항목 | 설명 |
| --- | --- |
| 자주 저장 | 자동 저장이 아직 없으므로 수동 저장 습관화 |
| Claude Code 인증 유지 | 인증 상태를 정기적으로 확인 |
| `.pen` 파일을 프로젝트에 포함 | AI가 코드와 디자인 모두 볼 수 있도록 유지 |
| Git에 정기적으로 커밋 | 버전 관리로 안전망 확보 |
| 정기 업데이트 | Pencil과 Claude Code를 최신 버전으로 유지 |
| 권한 프롬프트 수락 | 시스템 접근 권한 요청 시 수락 |
| 지원되는 워크플로 사용 | `.pen` 파일을 수동으로 편집하지 말 것 |

> 대부분의 문제는 Claude Code가 올바르게 인증되어 있고 Pencil이 최신 버전인지 확인하는 것으로 해결됩니다. 의심스러울 때는 IDE를 재시작하고 Claude Code를 재인증해 보세요.

---

### Reporting Bugs & Getting More Help

#### 버그 보고 시 포함 사항

**환경 정보:**

| 항목 | 내용 |
| --- | --- |
| OS 및 버전 | 예: macOS 15.4, Windows 11, Ubuntu 24.04 |
| IDE 및 버전 | VS Code / Cursor / Desktop 앱과 버전 |
| Pencil 버전 | 확장 패널에서 확인 |
| Claude Code CLI 버전 | `claude --version`으로 확인 |

**이슈 상세:**

- 시도했던 작업
- 실제 발생한 결과
- 정확한 에러 메시지
- 재현 단계

**추가 정보:**

- 스크린샷
- 콘솔 로그 (해당되는 경우)
- 이슈를 재현하는 최소 `.pen` 파일

#### 여전히 문제가 해결되지 않는 경우

| 단계 | 작업 |
| --- | --- |
| 1 | 관련 문서 검토: [Installation](https://docs.pencil.dev/getting-started/installation), [Authentication](https://docs.pencil.dev/getting-started/authentication), [AI Integration](https://docs.pencil.dev/getting-started/ai-integration) |
| 2 | 모든 것을 최신으로 업데이트: Pencil 최신 버전, Claude Code CLI 최신 버전, IDE 최신 버전 |
| 3 | 재설치 시도: 확장 제거 → 확장 데이터 초기화 → 새로 설치 |
| 4 | 지원팀에 연락: 플랫폼(macOS/Windows/Linux), Pencil 버전, IDE 및 버전, 정확한 에러 메시지, 재현 단계 포함 |
