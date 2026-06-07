# Pencil Dev 개요

> 원문: https://docs.pencil.dev

Pencil fundamentally increases your engineering speed by bringing designing directly into your preferred IDE. IDE 내부에서 직접 동작하는 벡터 디자인 도구입니다. "Design on canvas. Land in code."라는 모토처럼, 디자인과 코드를 같은 환경에서 작업할 수 있게 해줍니다.

---

## Pencil이란?

Pencil은 전통적인 디자인 도구(Figma, Sketch 등)와 달리 개발자의 IDE 안에서 동작하는 파워풀한 벡터 디자인 도구입니다. 별도의 애플리케이션이나 브라우저 탭을 오갈 필요 없이, 코드 옆에서 바로 디자인할 수 있습니다.

> **소개 영상**: [Pencil 소개 영상 보기 (YouTube)](https://www.youtube.com/watch?v=mduKhXvmsyA)

**핵심 특징:**

| 특징 | 설명 |
| --- | --- |
| IDE 통합 | VS Code, Cursor 확장 및 독립 데스크톱 앱 제공 |
| .pen 파일 포맷 | JSON 기반 벡터 디자인 파일 포맷 (구조화된 읽기 가능한 데이터 포맷, Git 호환) |
| MCP 서버 | AI 어시스턴트가 디자인 파일을 읽고 수정할 수 있는 로컬 MCP 서버 내장 |
| 코드 생성 | 디자인에서 React, Tailwind CSS 등의 코드를 직접 생성 |
| 디자인 시스템 | 재사용 가능한 컴포넌트, 변수, 테마 시스템 지원 |
| 무한 캔버스 | 제한 없는 작업 영역에서 자유롭게 디자인 |

---

## 왜 Pencil인가?

Pencil은 디자인과 개발 사이의 간극을 줄입니다. 디자인 컴포넌트를 만들고 코드와 동기화하며, AI 어시스턴트가 디자인과 구현 사이의 일관성을 유지하도록 도와줍니다.

**기존 워크플로의 문제점:**

| 문제 | Pencil의 해결책 |
| --- | --- |
| 디자인 도구와 IDE를 오가야 함 | IDE 내부에서 디자인과 코드를 동시 작업 |
| 디자인-코드 간 불일치 | Design ↔ Code 동기화로 일관성 유지 |
| 반복적인 컴포넌트 작성 | AI 에이전트가 디자인에서 코드를 자동 생성 |
| 디자인 시스템 관리 부담 | 변수, 테마, 재사용 컴포넌트로 체계적 관리 |

---

## 지원 AI 어시스턴트

Pencil은 MCP(Model Context Protocol)를 통해 여러 AI 도구와 연동됩니다:

| AI 도구 | 연동 방식 |
| --- | --- |
| Claude Code (CLI 및 IDE) | MCP 서버 직접 연결 |
| Claude Desktop | MCP 서버 연결 |
| Cursor (AI IDE) | 확장 프로그램 + MCP |
| Windsurf IDE (Codeium) | MCP 지원 |
| Codex CLI (OpenAI) | MCP 연결 |
| Antigravity IDE | 확장 프로그램 + MCP |
| OpenCode CLI | MCP 지원 |

---

## 주요 기능 영역

### 캔버스와 노드

무한 캔버스 위에 다양한 노드 타입(Frame, Text, Rectangle, Ellipse, Polygon, Icon, Path, Group, Note, Prompt, Context, Script, Ref 등)을 배치하고, Flexbox 기반 레이아웃 시스템으로 정밀하게 배치합니다.

| 노드 타입 | 설명 |
| --- | --- |
| Frame | 자식 노드를 포함하는 컨테이너. Flexbox 레이아웃 지원 |
| Text | 텍스트 콘텐츠 노드 |
| Rectangle / Ellipse / Polygon | 기본 도형 노드 |
| Icon | 아이콘 라이브러리(lucide, feather, Material Symbols 등)에서 아이콘 표시 |
| Path | SVG 경로 기반 벡터 노드 |
| Group | 자식 노드를 그룹화. 그래픽 속성 없이 효과만 적용 가능 |
| Note | 캔버스에 텍스트 주석을 남기는 노드 |
| Prompt | AI 모델 프롬프트를 위한 노드 (`model` 속성으로 모델 지정) |
| Context | 컨텍스트 정보를 제공하는 노드 |
| Script | JavaScript 파일로 동적 콘텐츠를 생성하는 노드 |
| Ref | 재사용 가능한 컴포넌트의 인스턴스를 참조하는 노드 |

### 컴포넌트 시스템

재사용 가능한 컴포넌트(reusable)를 정의하고, `ref` 노드로 인스턴스를 생성합니다. `descendants` 속성으로 인스턴스별 오버라이드가 가능합니다. 슬롯(slot) 메커니즘으로 컴포넌트 내부에 동적 콘텐츠를 삽입할 수 있습니다.

### 변수와 테마

타입이 있는 변수 시스템(color, number, string, boolean)으로 디자인 토큰을 관리합니다. 테마 축(theme axis)을 정의해 다크 모드, 반응형 등의 변형을 만들 수 있습니다.

### batch_design API

JavaScript 스니펫으로 프로그래매틱하게 디자인을 생성·수정하는 API입니다. Insert, Copy, Update, Replace, Move, Delete, Generate, FindEmptySpace 연산을 제공합니다.

### 코드 생성

디자인에서 React 컴포넌트를 생성하고, Tailwind CSS v4 클래스로 스타일링합니다. SVG 패스 추출, CSS 변수 매핑, 디자인 토큰 변환을 자동으로 수행합니다.

### 스크립팅

Script 노드로 JavaScript(.js) 파일을 연결해 동적 콘텐츠를 생성합니다. `@input` 지시어로 `number`, `string`, `boolean`, `color`, `enum`, `ref` 타입의 입력을 선언할 수 있습니다.

### 셰이더

.pen 파일 포맷의 fill 속성 중 Shader fill을 통해 WebGL 1.0(`#version 100`) 프래그먼트 셰이더 파일(.frag)을 URL로 참조하여 복잡한 그래픽 효과를 만들 수 있습니다.

---

## 제품 에코시스템

```
Pencil 에코시스템
├── IDE 확장
│   ├── VS Code Extension
│   ├── Cursor Extension
│   └── Antigravity IDE
├── 데스크톱 앱
│   ├── macOS (.dmg)
│   ├── Linux (.deb / .AppImage)
│   └── Windows
├── AI 연동
│   ├── MCP 서버 (로컬)
│   ├── Claude Code CLI
│   ├── Codex CLI
│   └── OpenCode CLI
├── 개발자 도구
│   ├── .pen 파일 포맷 (v2.13)
│   ├── batch_design API
│   └── Pencil CLI
└── 코드 생성
    ├── React / TypeScript
    ├── Tailwind CSS v4
    └── Next.js 통합
```

---

## 빠른 링크

| 항목 | 설명 |
| --- | --- |
| [설치 및 설정](./01-installation.md) | Your first steps with Pencil - VS Code, Cursor, 데스크톱, Claude Code CLI 설치 |
| [.pen 파일 기본](./03-pen-files.md) | Understanding .pen files and components - 파일 포맷, 구조, 변수, 컴포넌트 |
| [코드 생성](./11-code-generation.md) | Syncing design with your codebase - Design ↔ Code, React, Tailwind |
| [문제 해결](./12-troubleshooting.md) | Common issues and solutions |
| [개발자 가이드 (.pen Format)](./14-pen-format.md) | Technical information - .pen 파일 포맷 기술 명세 |

### 추가 가이드

| 항목 | 설명 |
| --- | --- |
| [AI 통합](./02-ai-integration.md) | MCP 서버, Claude Code, Cursor, Codex 연동 |
| [노드 타입 참조](./04-node-types.md) | Frame, Text, Rectangle, Ellipse, Polygon, Path, Icon, Script, Ref 등 |
| [캔버스와 레이아웃](./05-canvas-layout.md) | Flexbox, sizing, positioning |
