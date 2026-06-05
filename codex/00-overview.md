# Codex CLI - 개요 및 빠른 시작

> OpenAI의 에이전트 기반 코딩 도구, Codex CLI에 대한 종합 가이드

**참조**: [developers.openai.com/codex](https://developers.openai.com/codex) | [github.com/openai/codex](https://github.com/openai/codex)

---

## Codex CLI 소개

**Codex CLI**는 OpenAI가 개발한 경량 코딩 에이전트로, 로컬 컴퓨터의 터미널에서 실행됩니다. Rust 기반(`codex-rs`)으로 작성되어 빠르고 가볍게 동작하며, 코드 작성, 디버깅, 리팩토링, 자동화 등 소프트웨어 개발 전반을 지원합니다.

ChatGPT Plus, Pro, Business, Edu, Enterprise 플랜에 Codex가 포함되어 있으며, OpenAI API 키를 통해서도 사용할 수 있습니다.

---

## 두 가지 실행 환경

### 로컬 환경

| 환경 | 설명 |
| --- | --- |
| **CLI** | 터미널에서 직접 실행하는 풀스크린 TUI. 대화형으로 코드를 작성하고 검토 |
| **IDE 확장** | VS Code, Cursor, Windsurf에서 사이드바 패널로 동작 |
| **Codex App** | macOS/Windows 데스크톱 앱 (`codex app`으로 실행 또는 별도 다운로드) |

### 클라우드 환경

| 환경 | 설명 |
| --- | --- |
| **Codex Cloud (Web)** | [chatgpt.com/codex](https://chatgpt.com/codex)에서 브라우저 기반으로 동작. GitHub 리포지토리 연동 |
| **Code Review** | GitHub PR 코멘트에서 `@codex` 태그로 코드 리뷰 위임 |
| **Slack** | Slack 연동을 통해 팀 채널에서 Codex 사용 |

---

## 빠른 시작

### 1. 설치

**macOS / Linux** (권장 설치 스크립트):

```shell
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

**Windows** (PowerShell):

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
```

**npm** (Node.js 필요):

```shell
npm install -g @openai/codex
```

**Homebrew** (macOS):

```shell
brew install --cask codex
```

**GitHub Release 바이너리** 직접 다운로드도 지원:

- macOS Apple Silicon: `codex-aarch64-apple-darwin.tar.gz`
- macOS Intel: `codex-x86_64-apple-darwin.tar.gz`
- Linux x86_64: `codex-x86_64-unknown-linux-musl.tar.gz`
- Linux arm64: `codex-aarch64-unknown-linux-musl.tar.gz`

### 2. 인증

```shell
codex
```

실행 후 **Sign in with ChatGPT**를 선택하여 ChatGPT 계정(Plus, Pro, Business, Edu, Enterprise)으로 로그인합니다. API 키를 사용하려면 환경변수를 설정합니다:

```shell
export OPENAI_API_KEY="sk-..."
```

### 3. 첫 실행

```shell
# 대화형 모드로 시작
codex

# 프롬프트와 함께 바로 시작
codex "이 코드베이스를 설명해줘"

# 특정 디렉토리에서 실행
codex --cd /path/to/project "버그를 찾아줘"
```

---

## Codex가 할 수 있는 일

| 기능 | 설명 |
| --- | --- |
| **코드 작성** | 자연어 설명으로 코드를 생성하며, 기존 프로젝트 구조와 컨벤션에 맞게 적응 |
| **코드 이해** | 복잡하거나 레거시 코드를 읽고 설명. 낯선 코드베이스 파악에 유용 |
| **코드 리뷰** | 잠재적 버그, 논리 오류, 엣지 케이스를 분석하여 보고 |
| **디버깅 및 수정** | 실패 추적, 원인 진단, 타겟팅된 수정 제안 |
| **자동화** | 리팩토링, 테스트, 마이그레이션, 설정 작업 등 반복 워크플로우 자동 실행 |
| **이미지 처리** | 스크린샷이나 디자인 스펙을 첨부하여 이미지 세부 정보를 프롬프트와 함께 활용 |
| **이미지 생성** | 아이콘, 배너, 일러스트 등 에셋을 직접 생성 (`gpt-image-2` 사용) |
| **웹 검색** | 내장 웹 검색 도구로 최신 정보를 검색하여 코드 작성에 활용 |

---

## 핵심 기능 요약

| 기능 | 설명 | 관련 문서 |
| --- | --- | --- |
| **Skills (스킬)** | 작업별 특화 동작을 정의하여 Codex 성능 향상 | [Skills 문서](https://developers.openai.com/codex/skills) |
| **Plugins (플러그인)** | 마켓플레이스에서 추가 기능을 설치하여 확장 | [Plugins 문서](https://developers.openai.com/codex/plugins) |
| **Hooks (훅)** | 라이프사이클 이벤트에 사용자 정의 스크립트 실행 | [Hooks 문서](https://developers.openai.com/codex/hooks) |
| **MCP (Model Context Protocol)** | 외부 도구 서버를 연결하여 Codex 도구 생태계 확장 | [MCP 문서](https://developers.openai.com/codex/mcp) |
| **Apps (앱 연동)** | GitHub, Slack, Linear 등 서드파티 앱과 연동 | [Integrations 문서](https://developers.openai.com/codex/integrations) |
| **Sandbox (샌드박스)** | 명령어 실행을 격리된 환경에서 수행하여 시스템 보호 | [Sandbox 문서](https://developers.openai.com/codex/sandboxing) |
| **Subagents (서브에이전트)** | 큰 작업을 병렬로 분산 처리 | [Subagents 문서](https://developers.openai.com/codex/subagents) |
| **AGENTS.md** | 프로젝트별 지속적 명령어 파일로 Codex 동작 가이드 | [AGENTS.md 문서](https://developers.openai.com/codex/agents-md) |

---

## 추천 모델

| 모델 | 용도 |
| --- | --- |
| **gpt-5.5** | 대부분의 작업에 권장되는 최신 프론티어 모델 |
| **GPT-5.3-Codex-Spark** | ChatGPT Pro 구독자용 빠른 작업 (연구 프리뷰) |
| **gpt-4.1** | 범용 코딩 작업 |
| **gpt-4.1-mini** | 가벼운 작업 |

세션 중 `/model` 명령으로 모델을 전환할 수 있습니다.

---

## 공식 문서 링크 모음

| 리소스 | URL |
| --- | --- |
| Codex 공식 문서 | [developers.openai.com/codex](https://developers.openai.com/codex) |
| GitHub 리포지토리 | [github.com/openai/codex](https://github.com/openai/codex) |
| 빠른 시작 | [developers.openai.com/codex/quickstart](https://developers.openai.com/codex/quickstart) |
| CLI 기능 | [developers.openai.com/codex/cli/features](https://developers.openai.com/codex/cli/features) |
| 슬래시 명령어 | [developers.openai.com/codex/cli/slash-commands](https://developers.openai.com/codex/cli/slash-commands) |
| 설정 가이드 | [developers.openai.com/codex/config-file/config-basics](https://developers.openai.com/codex/config-file/config-basics) |
| 인증 | [developers.openai.com/codex/authentication](https://developers.openai.com/codex/authentication) |
| 요금 | [developers.openai.com/codex/pricing](https://developers.openai.com/codex/pricing) |
| 챗지피티 Codex | [chatgpt.com/codex](https://chatgpt.com/codex) |
| GitHub 설치 가이드 | [github.com/openai/codex - docs/install.md](https://github.com/openai/codex/blob/main/docs/install.md) |
| 커뮤니티 (Discord) | [discord.gg/openai](https://discord.gg/openai) |

---

> **최종 업데이트**: 2026-06-05
> **출처**: [developers.openai.com/codex](https://developers.openai.com/codex), [github.com/openai/codex](https://github.com/openai/codex)
