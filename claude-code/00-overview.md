# Claude Code 개요 및 빠른 시작

> Anthropic의 에이전트 기반 코딩 도구, 터미널에서 동작하는 AI 코딩 어시스턴트

**참고**: [Claude Code Overview - Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/overview)

---

## Claude Code란?

Claude Code는 Anthropic이 개발한 **에이전트 기반 코딩 도구**입니다. 터미널 내에서 동작하며, 자연어 명령을 통해 코드를 작성, 수정, 탐색, 디버깅할 수 있습니다. 별도의 IDE나 채팅 창이 아닌, 개발자가 이미 사용 중인 터미널 환경에서 직접 작동합니다.

---

## 30초 빠른 시작

### 사전 요구사항

- Node.js 18 이상
- Claude.ai 계정 (권장) 또는 Anthropic Console 계정

### 설치 및 실행

```bash
# Claude Code 설치
npm install -g @anthropic-ai/claude-code

# 프로젝트 디렉토리로 이동
cd your-awesome-project

# Claude Code 시작
claude
# 최초 실행 시 로그인 프롬프트가 표시됩니다
```

> 특정 설정이 필요하거나 문제가 발생한 경우 [설치 및 설정 가이드](01-installation.md)를 참조하세요.

---

## Claude Code가 할 수 있는 일

| 기능 | 설명 |
|------|------|
| **기능 빌드** | 자연어로 설명하면 계획을 수립하고 코드를 작성하여 동작을 확인합니다 |
| **디버깅 및 수정** | 버그를 설명하거나 에러 메시지를 붙여넣으면 코드베이스를 분석하여 원인을 파악하고 수정합니다 |
| **코드베이스 탐색** | 팀의 코드베이스에 대해 질문하면 프로젝트 구조를 파악하고 답변합니다. 웹에서 최신 정보를 검색할 수 있으며, MCP를 통해 Google Drive, Figma, Slack 등 외부 데이터 소스에 접근할 수 있습니다 |
| **반복 작업 자동화** | 린트 문제 수정, 머지 충돌 해결, 릴리스 노트 작성 등을 단일 명령으로 처리합니다. CI 환경에서도 자동 실행이 가능합니다 |

---

## 개발자들이 Claude Code를 좋아하는 이유

### 터미널에서 동작
별도의 채팅 창이나 IDE가 아닙니다. 개발자가 이미 사용 중인 터미널 환경에서, 익숙한 도구와 함께 작동합니다.

### 직접 행동
파일을 직접 편집하고, 명령을 실행하며, 커밋을 생성할 수 있습니다. MCP를 통해 Google Drive의 디자인 문서를 읽거나, Jira 티켓을 업데이트하거나, 커스텀 개발 도구를 사용할 수 있습니다.

### 유닉스 철학
조합 가능하고 스크립트 가능합니다:

```bash
# 로그 스트림 모니터링
tail -f app.log | claude -p "이 로그 스트림에서 이상 징후가 발견되면 Slack으로 알려줘"

# CI에서 자동 실행
claude -p "새로운 텍스트 문자열이 있으면 프랑스어로 번역하고 @lang-fr-team 리뷰용 PR을 생성해줘"
```

### 엔터프라이즈 준비
Anthropic API를 직접 사용하거나 AWS/GCP에 호스팅할 수 있습니다. 엔터프라이즈급 보안, 프라이버시, 컴플라이언스가 기본 내장되어 있습니다.

---

## 핵심 기능 요약

| 기능 | 설명 | 관련 문서 |
|------|------|-----------|
| 대화형 모드 | 터미널에서 대화형으로 코딩 | [CLI 참조](02-cli-reference.md) |
| 원샷 모드 | `claude -p`로 비대화형 실행 | [CLI 참조](02-cli-reference.md) |
| 슬래시 명령어 | 21개 내장 명령어 + 커스텀 명령어 | [CLI 참조](02-cli-reference.md) |
| Vim 모드 | Vim 스타일 편집 지원 | [CLI 참조](02-cli-reference.md) |
| MCP 서버 | 외부 도구 및 데이터 소스 연동 | [설정](03-settings.md) |
| 권한 관리 | allow/deny/ask 세분화된 권한 제어 | [설정](03-settings.md) |
| Hooks | 도구 실행 전후 커스텀 명령 실행 | [설정](03-settings.md) |
| CLAUDE.md | 프로젝트별 컨텍스트 및 지침 | [설정](03-settings.md) |
| Bedrock/Vertex | AWS, GCP 클라우드 프로바이더 연동 | [설치 및 설정](01-installation.md) |
| SDK | TypeScript, Python 프로그래밍 통합 | [CLI 참조](02-cli-reference.md) |

---

## 공식 문서 링크 모음

| 문서 | URL |
|------|-----|
| Claude Code 개요 | https://docs.anthropic.com/en/docs/claude-code/overview |
| 설치 및 설정 | https://docs.anthropic.com/en/docs/claude-code/install |
| CLI 참조 | https://docs.anthropic.com/en/docs/claude-code/cli-reference |
| 대화형 모드 | https://docs.anthropic.com/en/docs/claude-code/interactive-mode |
| 슬래시 명령어 | https://docs.anthropic.com/en/docs/claude-code/slash-commands |
| 설정 | https://docs.anthropic.com/en/docs/claude-code/settings |
| Bedrock & Vertex | https://docs.anthropic.com/en/docs/claude-code/bedrock-vertex |
| SDK | https://docs.anthropic.com/en/docs/claude-code/sdk |
| 트러블슈팅 | https://docs.anthropic.com/en/docs/claude-code/troubleshooting |
