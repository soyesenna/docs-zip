# Pencil Dev 인터페이스 및 단축키

> 원문: https://docs.pencil.dev/core-concepts/pencil-interface, https://docs.pencil.dev/core-concepts/keyboard-shortcuts

Pencil 인터페이스의 핵심 구성 요소와 전체 키보드 단축키를 정리한 문서입니다. 무한 캔버스 탐색부터 AI 채팅, 편집, 컴포넌트 관리까지 모든 워크플로우를 다룹니다.

---

## 무한 캔버스 (Infinite Canvas)

캔버스는 제한 없는 작업 공간을 제공하여 자유롭게 디자인을 탐색하고 개발할 수 있습니다. 전문 디자인 도구에서 이미 익숙한 원칙을 기반으로 하므로 바로 적응할 수 있습니다.

**이동**: 스페이스바를 누른 상태에서 드래그합니다.
**줌**: 트랙패드를 사용하거나 Cmd/Ctrl 키를 누른 상태에서 스크롤합니다.

### 캔버스 탐색 단축키

| 단축키 | 동작 |
| --- | --- |
| `Spacebar + Drag` | 캔버스 패닝 (이동) |
| `Shift + Scroll` | 수평 패닝 |
| `0` | 100%로 줌 |
| `1` | 모든 요소에 맞게 줌 |
| `2` | 선택 영역에 맞게 줌 |

---

## 프레임 (Frames)

프레임은 디자인의 컨테이너입니다.

- 관련 요소를 그룹화
- 화면 경계를 정의
- `Cmd/Ctrl + Option/Alt + G` 로 선택 항목에 Flex 레이아웃 적용

---

## 선택 및 하이라이트 (Selection & Highlighting)

캔버스에서 클릭 후 드래그하여 요소를 선택합니다. 선택된 요소는 타입을 나타내는 색상 경계 상자로 하이라이트됩니다.

**파란색** 경계 상자는 프레임, 도형, 텍스트 등 일반 요소에 나타납니다.

**마젠타 및 보라색** 경계 상자는 컴포넌트(재사용 요소)에 나타납니다.

- **마젠타**: 컴포넌트 오리진(원본). 이곳에서 변경하면 모든 인스턴스에 자동 반영됩니다.
- **보라색**: 컴포넌트 인스턴스.

### 선택 단축키

| 단축키 | 동작 |
| --- | --- |
| `Click` | 요소 선택 |
| `Cmd/Ctrl + Click` | 직접 선택 (가장 깊은 요소) |
| `Shift + Click` | 선택 영역에 추가 |
| `Cmd/Ctrl + A` | 전체 선택 |
| `Enter` | 현재 선택된 부모의 자식 요소 선택 |
| `Shift + Enter` | 현재 선택된 자식의 부모 요소 선택 |
| `Esc` | 선택 해제 |

---

## 레이어 패널 (Layers Panel)

레이어 패널은 화면 좌측에 위치하며 캔버스의 모든 요소를 나열합니다. 디자인 계층 구조를 명확하게 보여주어 복잡한 중첩 구조에서도 요소를 쉽게 탐색, 편집, 정리할 수 있습니다.

| 기능 | 설명 |
| --- | --- |
| 레이어 이름 변경 | 패널에서 레이어를 더블클릭 |
| 패널 토글 | "Layers" 아이콘 클릭 |

---

## 속성 패널 (Properties Panel)

속성 패널은 캔버스에서 하나 이상의 요소를 선택하면 화면 우측에 나타납니다. 정렬, 레이아웃, 외관, 채우기, 테두리, 효과 등의 속성을 확인하고 편집할 수 있습니다.

| 기능 | 설명 |
| --- | --- |
| 내보내기 | 선택 항목을 PNG, JPEG, WEBP, PDF로 내보내기 |
| 패널 최소화 | 우측 상단 아이콘 클릭 |

---

## AI 채팅 (AI Chat)

Pencil의 AI 채팅은 바이브 디자이닝(vibe-designing)을 위한 인터페이스입니다. 처음부터 디자인하거나 캔버스의 기존 디자인을 편집하도록 요청할 수 있습니다.

데스크톱 앱에서는 채팅 패널이 내장되어 있습니다. IDE에서 Pencil 확장을 사용할 때는 IDE의 내장 채팅을 통해 AI 에이전트와 작업합니다.

- 캔버스에서 선택한 항목은 자동으로 컨텍스트에 추가됩니다.
- "New Agent"를 클릭하면 새로 시작하며 컨텍스트 윈도우가 초기화됩니다.

### AI 채팅 단축키

| 단축키 | 동작 |
| --- | --- |
| `Cmd/Ctrl + K` | AI 채팅 토글 |
| `Cmd/Ctrl + T` | 새 Agent 탭 열기 |
| `Ctrl + Tab` | 다음 채팅 탭으로 이동 |
| `Ctrl + Shift + Tab` | 이전 채팅 탭으로 이동 |
| `↑` / `↓` | 프롬프트 히스토리 탐색 |

---

## 도구 모음 (Toolbar)

캔버스 상단의 도구 모음에서 디자인 도구를 선택할 수 있습니다.

| 단축키 | 도구 | 설명 |
| --- | --- | --- |
| `V` | Move | 이동 도구 |
| `H` | Hand | 핸드 도구 (캔버스 이동) |
| `R` | Rectangle | 사각형 도구 |
| `O` | Ellipse | 타원 도구 |
| `A` / `F` | Frame | 프레임 도구 |
| `T` | Text | 텍스트 도구 |
| `N` | Sticky note | 스티커 메모 도구 |
| `P` | Path | 패스 도구 |

---

## 편집 단축키 (Editing)

| 단축키 | 동작 |
| --- | --- |
| `Cmd/Ctrl + C` | 클립보드에 복사 |
| `Cmd/Ctrl + V` | 클립보드에서 붙여넣기 |
| `Cmd/Ctrl + D` | 선택 항목 복제 |
| `Cmd/Ctrl + X` | 잘라내기 |
| `Delete` / `Backspace` | 삭제 |
| `Arrow keys` | 1px 이동 / Flex 레이아웃에서 순서 교환 |
| `Shift + Arrow keys` | 10px 이동 |
| `Cmd/Ctrl + G` | 그룹화 |
| `Cmd/Ctrl + Shift + G` | 그룹 해제 |
| `Cmd/Ctrl + Option/Alt + G` | 선택 항목에 Flex 레이아웃 적용 |
| `Cmd/Ctrl + [` | 뒤로 보내기 (Send backward) |
| `Cmd/Ctrl + ]` | 앞으로 가져오기 (Bring forward) |
| `[` | 맨 뒤로 보내기 (Send to back) |
| `]` | 맨 앞으로 가져오기 (Bring to front) |

---

## 실행 취소 / 다시 실행 (Undo / Redo)

| 단축키 | 동작 |
| --- | --- |
| `Cmd/Ctrl + Z` | 실행 취소 (Undo) |
| `Cmd/Ctrl + Shift + Z` | 다시 실행 (Redo) |

> 실행 취소 및 다시 실행은 일반 디자인 에디터보다 제한적일 수 있습니다.

### 모범 사례

- 주요 변경 전 자주 저장하고 Git 커밋을 활용하세요.
- `Cmd/Ctrl + S` 로 자주 저장하세요.
- 필요 시 Git 히스토리로 되돌리세요.

---

## 컴포넌트 단축키 (Components)

| 단축키 | 동작 |
| --- | --- |
| `Cmd/Ctrl + Option/Alt + K` | 요소를 컴포넌트로 변환 또는 컴포넌트를 일반 요소로 되돌리기 |
| `Cmd/Ctrl + Option/Alt + X` | 컴포넌트 인스턴스 분리 (Detach) |

---

## 캔버스 설정 (Canvas Settings)

| 단축키 | 동작 |
| --- | --- |
| `Cmd/Ctrl + '` | 픽셀 그리드 토글 |
| `Cmd/Ctrl + Shift + '` | 픽셀 그리드 스냅 토글 |

---

## 파일 작업 (File Operations)

| 단축키 | 동작 |
| --- | --- |
| `Cmd/Ctrl + S` | 파일 저장 |
| `Cmd/Ctrl + N` | 새 파일 만들기 |
| `Cmd/Ctrl + O` | 파일 열기 |

---

## 전체 단축키 빠른 참조

아래는 모든 단축키를 카테고리별로 한눈에 볼 수 있는 종합 표입니다.

| 카테고리 | 단축키 | 동작 |
| --- | --- | --- |
| **AI 채팅** | `Cmd/Ctrl + K` | AI 채팅 토글 |
| | `Cmd/Ctrl + T` | 새 Agent 탭 |
| | `Ctrl + Tab` | 다음 채팅 탭 |
| | `Ctrl + Shift + Tab` | 이전 채팅 탭 |
| | `↑` / `↓` | 프롬프트 히스토리 |
| **도구** | `V` | 이동 도구 |
| | `H` | 핸드 도구 |
| | `R` | 사각형 도구 |
| | `O` | 타원 도구 |
| | `A` / `F` | 프레임 도구 |
| | `T` | 텍스트 도구 |
| | `N` | 스티커 메모 도구 |
| **편집** | `Cmd/Ctrl + C` | 복사 |
| | `Cmd/Ctrl + V` | 붙여넣기 |
| | `Cmd/Ctrl + D` | 복제 |
| | `Cmd/Ctrl + X` | 잘라내기 |
| | `Delete` / `Backspace` | 삭제 |
| | `Arrow keys` | 1px 이동 / Flex 순서 교환 |
| | `Shift + Arrow keys` | 10px 이동 |
| | `Cmd/Ctrl + G` | 그룹화 |
| | `Cmd/Ctrl + Shift + G` | 그룹 해제 |
| | `Cmd/Ctrl + Option/Alt + G` | Flex 레이아웃 적용 |
| | `Cmd/Ctrl + [` | 뒤로 보내기 |
| | `Cmd/Ctrl + ]` | 앞으로 가져오기 |
| | `[` | 맨 뒤로 보내기 |
| | `]` | 맨 앞으로 가져오기 |
| **선택** | `Cmd/Ctrl + A` | 전체 선택 |
| | `Cmd/Ctrl + Click` | 직접 선택 |
| | `Shift + Click` | 선택 추가 |
| | `Enter` | 자식 요소 선택 |
| | `Shift + Enter` | 부모 요소 선택 |
| | `Esc` | 선택 해제 |
| **컴포넌트** | `Cmd/Ctrl + Option/Alt + K` | 컴포넌트 변환/되돌리기 |
| | `Cmd/Ctrl + Option/Alt + X` | 인스턴스 분리 |
| **캔버스 탐색** | `Cmd/Ctrl + Scroll` | 줌 |
| | `Spacebar + Drag` | 패닝 |
| | `Shift + Scroll` | 수평 패닝 |
| | `=` | 줌 인 |
| | `-` | 줌 아웃 |
| | `0` | 100% 줌 |
| | `1` | 전체 맞춤 줌 |
| | `2` | 선택 영역 줌 |
| **캔버스 설정** | `Cmd/Ctrl + '` | 픽셀 그리드 토글 |
| | `Cmd/Ctrl + Shift + '` | 스냅 토글 |
| **파일** | `Cmd/Ctrl + S` | 저장 |
| | `Cmd/Ctrl + N` | 새 파일 |
| | `Cmd/Ctrl + O` | 파일 열기 |
| **되돌리기** | `Cmd/Ctrl + Z` | 실행 취소 |
| | `Cmd/Ctrl + Shift + Z` | 다시 실행 |
