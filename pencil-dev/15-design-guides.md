# Pencil Dev 디자인 가이드와 스타일

> 원문: Pencil MCP 서버 가이드 8종 (Web App, Mobile App, Landing Page, Slides, Table, Tailwind, Code, Design System), https://docs.pencil.dev/troubleshooting

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

### 일반적인 문제

| 문제 | 해결 |
| --- | --- |
| 확장이 연결되지 않음 | Claude Code 로그인 확인 → MCP 서버 연결 확인 → 확장 재설치 |
| 디자인 변경이 저장되지 않음 | .pen 파일이 열려 있는지 확인 → Pencil 재시작 |
| AI가 응답하지 않음 | Claude Code 인증 확인 → `claude` CLI로 로그인 → Pencil 재시동 |
| 내보내기 이미지가 비어 있음 | 노드에 실제 콘텐츠가 있는지 확인 |
| 캔버스가 느림 | 노드 수가 너무 많은지 확인. 사용하지 않는 화면 삭제 |
| 컴포넌트가 업데이트 안 됨 | 인스턴스가 올바른 ref를 가리키는지 확인 |

### MCP 연결 문제

| 환경 | 확인 방법 |
| --- | --- |
| Cursor | Settings → Tools & MCP → Pencil이 목록에 있는지 확인 |
| VS Code | 명령 팔레튼 → "Pencil" 검색 |
| Codex CLI | `/mcp` 실행 → Pencil 확인 |
