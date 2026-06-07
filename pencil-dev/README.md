# Pencil Dev 개발자 문서

> https://pencil.dev, https://docs.pencil.dev를 기반으로 정리한 한국어 개발자 가이드입니다.
> 최종 업데이트: 2026-06-07 (공식 문서 동기화 완료)

Pencil은 IDE 내부에서 동작하는 벡터 디자인 도구입니다. "Design on canvas. Land in code." — 디자인과 코드를 같은 환경에서 작업하며, MCP를 통해 AI 어시스턴트가 디자인을 직접 조작할 수 있습니다.

---

## 문서 목록

### 시작하기

| # | 문서 | 설명 |
| --- | --- | --- |
| 00 | [Pencil Dev 개요](./00-overview.md) | 제품 소개, 핵심 기능, 에코시스템, 빠른 링크 |
| 01 | [설치 및 설정](./01-installation.md) | VS Code, Cursor, 데스크톱, Claude Code CLI, MCP 서버 설치 |
| 02 | [AI 통합](./02-ai-integration.md) | MCP 프로토콜, Claude Code, Cursor, Codex 연동, MCP 도구 목록 |

### 코어 컨셉

| # | 문서 | 설명 |
| --- | --- | --- |
| 03 | [.pen 파일 기본](./03-pen-files.md) | 파일 포맷, Document 구조, 임포트, 테마, 변수 정의 |
| 04 | [노드 타입 참조](./04-node-types.md) | Frame, Text, Shape, Path, Icon, Script, Ref 전체 노드 타입 |
| 05 | [캔버스와 레이아웃](./05-canvas-layout.md) | Flexbox 레이아웃, sizing, positioning, 레이아웃 패턴 |
| 06 | [텍스트와 타이포그래피](./06-text-typography.md) | textGrowth 모드, 정렬, lineHeight, 줄바꿈, 안티패턴 |
| 07 | [컴포넌트와 슬롯](./07-components-slots.md) | reusable, ref, descendants 오버라이드, slots, Copy/Replace |
| 08 | [변수와 테마](./08-variables-themes.md) | 변수 타입, 테마 축, 디자인 토큰 컨벤션, CSS 매핑 |
| 09 | [그래픽과 효과](./09-graphics-effects.md) | Fill 5종, Stroke, Effect 3종, SVG Path, 이미지, 아이콘 |

### API 및 개발

| # | 문서 | 설명 |
| --- | --- | --- |
| 10 | [batch_design API](./10-batch-design-api.md) | Insert, Copy, Update, Replace, Move, Delete, Generate, FindEmptySpace |
| 11 | [코드 생성](./11-code-generation.md) | Design ↔ Code, React 컴포넌트 생성, Tailwind CSS v4, SVG 추출 |
| 12 | [디자인 시스템 구성](./12-design-system.md) | Sidebar, Card, Table, Tabs, Dropdown, Pagination 조합 패턴 |
| 13 | [스크립팅과 셰이더](./13-scripting-shaders.md) | Script 노드, @input, WebGL 셰이더, 유니폼 어노테이션 |
| 14 | [가져오기/내보내기/CLI](./14-import-export.md) | Import/Export, 디자인 라이브러리, Pencil CLI, 키보드 단축키 |
| 15 | [디자인 가이드와 스타일](./15-design-guides.md) | Web App 16원칙, Mobile, Landing Page, Slides 가이드, 27 스타일 프리셋, 문제 해결 가이드 |
| 16 | [인터페이스와 단축키](./16-pencil-interface.md) | 무한 캔버스, 프레임, 레이어/속성/AI 패널, 도구 모음, 전체 키보드 단축키 |

---

## 빠른 참조

### 노드 타입 요약

```
Frame    — 컨테이너 (layout, padding, gap, clip, slot, reusable)
Group    — 논리적 그룹 (자식 포함, 레이아웃 없음)
Rectangle — 사각형 (cornerRadius)
Ellipse  — 원/타원/호 (innerRadius, startAngle, sweepAngle)
Polygon  — 정다각형 (polygonCount)
Path     — SVG 패스 (geometry, viewBox, fillRule)
Text     — 텍스트 (textGrowth, fontFamily, fontSize, fill)
Icon     — 아이콘 (library, icon, weight)
Script   — JS 동적 생성 (scriptUri, inputs)
Ref      — 컴포넌트 인스턴스 (ref, descendants)
```

### batch_design API 요약

```
Insert(parent, data) → nodeId
Copy(path, parent, data) → nodeId
Update(path, data) → void
Replace(path, data) → nodeId
Move(path, parent, index?) → void
Delete(path) → void
Generate(nodeId, "ai"|"stock", prompt) → void
FindEmptySpace({width, height, direction?, padding?}) → {x, y}
```

### Sizing 요약

| Pencil | 설명 |
| --- | --- |
| `fill_container` | 부모 크기에 맞춤 |
| `fit_content` | 자식 크기에 맞춤 |
| `fill_container(N)` | 부모 크기 + fallback N |
| `fit_content(N)` | 자식 크기 + fallback N |

### MCP 도구 요약

| 도구 | 용도 |
| --- | --- |
| `get_editor_state` | 편집기 상태 조회 |
| `batch_get` | 노드 배치 조회 |
| `batch_design` | JS로 디자인 수정 |
| `get_screenshot` | 스크린샷 캡처 |
| `export_nodes` | 이미지/PDF 내보내기 |
| `snapshot_layout` | 레이아웃 구조 확인 |
| `get_variables` | 변수/테마 조회 |
| `set_variables` | 변수/테마 업데이트 |
| `get_guidelines` | 가이드/스타일 로드 |

---

## 원문 링크

- 공식 문서: https://docs.pencil.dev
- 제품 페이지: https://pencil.dev
- Pencil YouTube 소개: https://www.youtube.com/watch?v=mduKhXvmsyA
