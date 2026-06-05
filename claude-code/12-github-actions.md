# 12. GitHub Actions 통합

> **참조**: [Claude Code GitHub Actions - Anthropic](https://docs.anthropic.com/en/docs/claude-code/github-actions)

---

## 목차

- [GitHub Actions 개요](#github-actions-개요)
- [주요 기능](#주요-기능)
- [Claude가 할 수 있는 일](#claude가-할-수-있는-일)
- [설정 방법](#설정-방법)
- [Beta에서 GA로 업그레이드](#beta에서-ga로-업그레이드)
- [Action 매개변수 v1](#action-매개변수-v1)
- [claude_args 사용법](#claude_args-사용법)
- [AWS Bedrock / Google Vertex AI 사용](#aws-bedrock--google-vertex-ai-사용)
- [보안 고려사항](#보안-고려사항)
- [비용 최적화 팁](#비용-최적화-팁)
- [문제 해결](#문제-해결)

---

## GitHub Actions 개요

Claude Code GitHub Actions는 GitHub 워크플로우에 AI 자동화를 제공합니다. PR이나 이슈에서 `@claude` 멘션만으로 Claude가 코드를 분석하고, PR을 생성하며, 기능을 구현하고, 버그를 수정할 수 있습니다.

---

## 주요 기능

| 기능 | 설명 |
|------|------|
| **즉시 PR 생성** | 원하는 것을 설명하면 Claude가 필요한 모든 변경사항이 포함된 완전한 PR 생성 |
| **자동 코드 구현** | 단일 명령으로 이슈를 작동하는 코드로 변환 |
| **표준 준수** | `CLAUDE.md` 가이드라인과 기존 코드 패턴을 따름 |
| **간단한 설정** | 설치 프로그램과 API 키로 몇 분 안에 시작 |
| **기본 보안** | 코드가 GitHub의 러너에 머물며 안전하게 처리 |

---

## Claude가 할 수 있는 일

### 이슈/PR 코멘트에서 사용

```
@claude 이 PR의 코드를 리뷰해주세요
@claude 이 버그를 수정해주세요
@claude 이 기능을 구현해주세요
@claude 테스트를 추가해주세요
```

Claude가 자동으로 컨텍스트를 분석하고 적절하게 응답합니다.

---

## 설정 방법

### 빠른 설정

가장 쉬운 설정 방법은 Claude Code 터미널에서 `/install-github-app`을 실행하는 것입니다.

```bash
$ claude
> /install-github-app
```

이 명령어가 GitHub 앱 설정과 필수 시크릿 구성을 안내합니다.

### 수동 설정

빠른 설정이 실패하거나 수동 설정을 선호하는 경우:

**1단계**: Claude GitHub 앱을 리포지토리에 설치

- [https://github.com/apps/claude](https://github.com/apps/claude) 에서 설치

**2단계**: `ANTHROPIC_API_KEY`를 리포지토리 시크릿에 추가

- GitHub 리포지토리 > Settings > Secrets and variables > Actions
- 새 시크릿 추가: 이름 `ANTHROPIC_API_KEY`

**3단계**: 워크플로우 파일 복사

```yaml
# .github/workflows/claude.yml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]

jobs:
  claude:
    if: contains(github.event.comment.body, '@claude') || (github.event_name == 'issues' && contains(github.event.issue.body, '@claude'))
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-base-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

## Beta에서 GA로 업그레이드

Beta 버전을 사용 중인 경우 GA 버전으로 업그레이드하는 것이 권장됩니다.

### 필수 변경사항

1. **액션 버전 업데이트**: `@beta`를 `@v1`로 변경
2. **모드 설정 제거**: `mode: "tag"` 또는 `mode: "agent"` 삭제 (자동 감지됨)
3. **프롬프트 입력 업데이트**: `direct_prompt`를 `prompt`로 교체
4. **CLI 옵션 이동**: `max_turns`, `model`, `custom_instructions` 등을 `claude_args`로 변환

### Breaking Changes 표

| 구 Beta 입력 | 신규 v1.0 입력 |
|-------------|---------------|
| `mode` | _(제거됨 - 자동 감지)_ |
| `direct_prompt` | `prompt` |
| `override_prompt` | `prompt` (GitHub 변수 사용) |
| `custom_instructions` | `claude_args: --system-prompt` |
| `max_turns` | `claude_args: --max-turns` |
| `model` | `claude_args: --model` |
| `allowed_tools` | `claude_args: --allowedTools` |
| `disallowed_tools` | `claude_args: --disallowedTools` |
| `claude_env` | `settings` JSON 형식 |

### 변경 전/후 예시

**Beta 버전:**

```yaml
- uses: anthropics/claude-code-base-action@beta
  with:
    mode: "agent"
    direct_prompt: "Review this PR"
    max_turns: 5
    model: "claude-sonnet-4-20250514"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**GA 버전 (v1.0):**

```yaml
- uses: anthropics/claude-code-base-action@v1
  with:
    prompt: "Review this PR"
    claude_args: "--max-turns 5 --model claude-sonnet-4-20250514"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

## Action 매개변수 v1

| 매개변수 | 설명 | 필수 |
|----------|------|------|
| `prompt` | Claude에 대한 지침 (텍스트 또는 슬래시 명령어) | 아니요* |
| `claude_args` | Claude Code에 전달할 CLI 인자 | 아니요 |
| `anthropic_api_key` | Anthropic API 키 | 예** |
| `github_token` | GitHub API 접근용 토큰 | 아니요 |
| `trigger_phrase` | 커스텀 트리거 구문 (기본값: `@claude`) | 아니요 |
| `use_bedrock` | Anthropic API 대신 AWS Bedrock 사용 | 아니요 |
| `use_vertex` | Anthropic API 대신 Google Vertex AI 사용 | 아니요 |

> *`prompt`는 선택사항입니다. 이슈/PR 코멘트에서 생략하면 Claude가 트리거 구문에 응답합니다.
>
> **Anthropic API 직접 사용 시에만 필수. Bedrock/Vertex 사용 시 불필요

---

## claude_args 사용법

`claude_args` 매개변수는 Claude Code의 모든 CLI 인자를 허용합니다.

### 일반 인자

| 인자 | 설명 | 예시 |
|------|------|------|
| `--max-turns` | 최대 대화 턴 수 (기본값: 10) | `--max-turns 5` |
| `--model` | 사용할 모델 | `--model claude-sonnet-4-20250514` |
| `--mcp-config` | MCP 설정 파일 경로 | `--mcp-config servers.json` |
| `--allowed-tools` | 허용할 도구 목록 (쉼표 구분) | `--allowed-tools "Bash,Read,Edit"` |
| `--debug` | 디버그 출력 활성화 | `--debug` |

### 사용 예시

```yaml
- uses: anthropics/claude-code-base-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: "--max-turns 5 --model claude-sonnet-4-20250514 --debug"
```

---

## AWS Bedrock / Google Vertex AI 사용

엔터프라이즈 환경에서는 자체 클라우드 인프라를 사용하여 데이터 거주지와 결제를 제어할 수 있습니다.

### Google Cloud Vertex AI 전제조건

| 요구사항 | 설명 |
|----------|------|
| Google Cloud 프로젝트 | Vertex AI 활성화 |
| Workload Identity Federation | GitHub Actions용 구성 |
| 서비스 계정 | 필요한 권한 포함 |
| GitHub App | 권장 (또는 기본 `GITHUB_TOKEN` 사용) |

### AWS Bedrock 전제조건

| 요구사항 | 설명 |
|----------|------|
| AWS 계정 | Amazon Bedrock 활성화 |
| GitHub OIDC Identity Provider | AWS에 구성 |
| IAM 역할 | Bedrock 권한 포함 |
| GitHub App | 권장 (또는 기본 `GITHUB_TOKEN` 사용) |

### 워크플로우 예시 (Bedrock)

```yaml
- uses: anthropics/claude-code-base-action@v1
  with:
    use_bedrock: true
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

---

## 보안 고려사항

### API 키 관리

항상 GitHub Secrets를 사용하여 API 키를 관리하세요.

```yaml
# 올바른 방법
anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

# 절대 하지 말 것 - 워크플로우 파일에 API 키를 직접 하드코딩 금지
anthropic_api_key: "sk-ant-xxxxx"  # 위험!
```

### 보안 체크리스트

| 항목 | 설명 |
|------|------|
| **시크릿 사용** | API 키를 `ANTHROPIC_API_KEY` 리포지토리 시크릿으로 저장 |
| **최소 권한** | 액션 권한을 필요한 것만으로 제한 |
| **변경 검토** | Claude의 제안을 병합 전에 검토 |
| **시크릿 참조** | 워크플로우에서 `${{ secrets.ANTHROPIC_API_KEY }}`로 참조 |

---

## 비용 최적화 팁

### GitHub Actions 비용

- Claude Code는 GitHub 호스팅 러너에서 실행되어 GitHub Actions 분을 소비
- 자세한 가격 및 분 제한은 GitHub 결제 문서 참조

### API 비용

- 각 Claude 상호작용은 프롬프트와 응답 길이에 따라 API 토큰 소비
- 토큰 사용량은 작업 복잡도와 코드베이스 크기에 따라 다름
- 현재 토큰 요금은 Claude 가격 페이지 참조

### 비용 절감 방법

| 방법 | 설명 |
|------|------|
| **구체적 명령 사용** | `@claude` 명령을 구체적으로 작성하여 불필요한 API 호출 감소 |
| **max-turns 설정** | `claude_args`에 적절한 `--max-turns`를 구성하여 과도한 반복 방지 |
| **타임아웃 설정** | 워크플로우 수준 타임아웃으로 장기 실행 작업 방지 |
| **동시성 제어** | GitHub의 동시성 제어를 사용하여 병렬 실행 제한 |

```yaml
jobs:
  claude:
    runs-on: ubuntu-latest
    timeout-minutes: 10  # 워크플로우 타임아웃
    concurrency:
      group: claude-${{ github.event.pull_request.number }}
      cancel-in-progress: true  # 이전 실행 취소
```

---

## 문제 해결

### Claude가 @claude 명령에 응답하지 않는 경우

| 확인 사항 | 해결 방법 |
|----------|----------|
| GitHub 앱 설치 | 앱이 올바르게 설치되어 있는지 확인 |
| 워크플로우 활성화 | 워크플로우가 활성화되어 있는지 확인 |
| API 키 | 리포지토리 시크릿에 API 키가 설정되어 있는지 확인 |
| 트리거 구문 | 코멘트에 `@claude`가 포함되어 있는지 확인 (`/claude` 아님) |

### Claude의 커밋에 CI가 실행되지 않는 경우

| 확인 사항 | 해결 방법 |
|----------|----------|
| 앱 사용 | GitHub App 또는 커스텀 앱 사용 중인지 확인 (Actions 사용자가 아님) |
| 워크플로우 트리거 | 트리거에 필요한 이벤트가 포함되어 있는지 확인 |
| 앱 권한 | CI 트리거가 포함되어 있는지 확인 |

### 인증 에러

| 확인 사항 | 해결 방법 |
|----------|----------|
| API 키 유효성 | API 키가 유효하고 충분한 권한이 있는지 확인 |
| Bedrock/Vertex 자격 증명 | 자격 증명 구성이 올바른지 확인 |
| 시크릿 이름 | 워크플로우에서 시크릿 이름이 정확한지 확인 |

---

## 요약

Claude Code GitHub Actions는 `@claude` 멘션으로 PR 생성, 코드 분석, 기능 구현, 버그 수정을 자동화합니다. 빠른 설정(`/install-github-app`)으로 몇 분 안에 시작할 수 있으며, GA 버전(v1)은 간소화된 매개변수 구조를 제공합니다. 비용 최적화와 보안 모범 사례를 준수하여 효율적으로 사용하세요.
