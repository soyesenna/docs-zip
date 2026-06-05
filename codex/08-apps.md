# 앱 연동

> 사전 구축된 도구 커넥터를 통해 Codex를 외부 서비스와 연결합니다.

**참조**: <https://developers.openai.com/codex/integrations/github>, <https://developers.openai.com/codex/integrations/slack>, <https://developers.openai.com/codex/config-reference>

---

## 앱 연동 개요

Codex는 사전 구축된 도구 커넥터를 통해 다양한 외부 서비스와 연동됩니다. 컴포저에서 `$`를 입력하면 접근 가능한 앱 목록이 표시되며, `/apps` 명령어로 사용 가능 및 설치된 앱을 확인할 수 있습니다. 연결된 앱은 목록 상단에 표시되고, 설치 가능한 앱은 별도로 표시됩니다.

### 지원 앱

| 앱 | 설명 |
| --- | --- |
| GitHub Code Review | PR 코드 리뷰 자동화 |
| Slack | 채널/스레드에서 작업 실행 |
| Linear | 프로젝트 관리 연동 |
| Google Drive | 파일 접근 |
| Gmail | 이메일 연동 |
| Figma | 디자인 파일 연동 |
| Sentry | 오류 추적 연동 |

---

## GitHub Code Review

Codex를 사용하면 GitHub Pull Request에 대한 고품질 코드 리뷰를 자동으로 수행할 수 있습니다.

### 설정

1. **Codex Cloud 설정**: 리포지토리에 대해 Codex Cloud를 설정합니다.
2. **Code Review 활성화**: Codex 설정에서 해당 리포지토리의 **Code review** 토글을 켭니다.
3. **Automatic Reviews 활성화** (선택): 모든 PR에 대해 자동 리뷰를 원하는 경우 **Automatic reviews** 토글을 켭니다.

### 리뷰 요청

PR 코멘트에서 `@codex review`를 멘션하면 Codex가 리뷰를 게시합니다.

```
@codex review
```

Codex는 팀원처럼 PR에 리뷰를 게시하며, **P0 및 P1 이슈**만 플래그하여 리뷰 코멘트가 우선순위가 높은 위험에 집중되도록 합니다.

### 자동 리뷰

**Automatic Reviews**가 활성화되면 새 PR이 열릴 때 `@codex review` 코멘트 없이도 자동으로 리뷰가 게시됩니다. 리뷰 트리거 설정으로 어떤 PR 이벤트에서 리뷰를 시작할지 제어할 수 있습니다.

### 커스터마이징

#### AGENTS.md 리뷰 가이드라인

리포지토리에 `AGENTS.md` 파일을 추가하여 Codex가 따를 **Review guidelines**를 정의할 수 있습니다.

```markdown
## Review guidelines

- PII를 로깅하지 않습니다.
- 모든 라우트에 인증 미들웨어가 적용되었는지 확인합니다.
- SQL 쿼리에 파라미터화가 사용되었는지 검증합니다.
```

Codex는 변경된 파일에 가장 가까운 `AGENTS.md`의 가이드라인을 적용합니다. 특정 패키지에 추가 검증이 필요한 경우 더 깊은 디렉토리에 구체적인 지침을 배치할 수 있습니다.

#### 일회성 포커스

PR 코멘트에 특정 초점을 추가할 수도 있습니다.

```
@codex review for security regressions
```

### 리뷰 후 수정

리뷰가 게시된 후, 같은 PR에서 코멘트를 통해 이슈 수정을 요청할 수 있습니다.

```
@codex fix the P1 issue
```

Codex는 PR을 컨텍스트로 사용하여 클라우드 작업을 시작하고, 권한이 있는 경우 브랜치에 수정 사항을 푸시할 수 있습니다.

> `@codex`와 함께 `review` 외의 다른 명령을 사용하면 Codex가 PR을 컨텍스트로 사용하는 클라우드 작업을 시작합니다. (예: `@codex fix the CI failures`)

---

## Slack 통합

Slack 채널과 스레드에서 Codex를 사용하여 코딩 작업을 시작할 수 있습니다.

### 설정

1. **Codex Cloud 작업 설정**: Plus, Pro, Business, Enterprise, 또는 Edu 플랜이 필요하며, 연결된 GitHub 계정과 최소 하나의 환경이 필요합니다.
2. **Slack 앱 설치**: Codex 설정에서 Slack 앱을 워크스페이스에 설치합니다. Slack 워크스페이스 정책에 따라 관리자 승인이 필요할 수 있습니다.
3. **채널에 @Codex 추가**: 채널에 `@Codex`를 추가합니다. 아직 추가하지 않은 경우 멘션 시 Slack이 추가를 안내합니다.

### 작업 시작

1. 채널이나 스레드에서 `@Codex`를 멘션하고 프롬프트를 포함합니다. Codex는 스레드의 이전 메시지를 참조할 수 있으므로 컨텍스트를 다시 설명할 필요가 없습니다.
2. (선택) 프롬프트에 환경이나 리포지토리를 지정합니다. (예: `@Codex fix the above in openai/codex`)
3. Codex가 반응(👀)하고 작업 링크로 답장합니다. 완료되면 결과를 스레드에 게시합니다.

### 환경 선택 방식

| 조건 | 동작 |
| --- | --- |
| 명시적 지정 | 프롬프트에 지정된 환경/리포지토리 사용 |
| 모호한 요청 | 사용자가 가장 최근에 사용한 환경으로 폴백 |
| 기본 리포지토리 | 환경의 리포지토리 맵에서 첫 번째 리포지토리의 기본 브랜치 사용 |
| 적합한 환경 없음 | Slack에서 해결 방법 안내 후 재시도 |

### 엔터프라이즈 데이터 제어

기본적으로 Codex는 작업 완료 시 스레드에 전체 답변을 게시합니다. 엔터프라이즈 관리자는 ChatGPT 워크스페이스 설정에서 **Allow Codex Slack app to post answers on task completion**을 해제하여 이를 비활성화할 수 있습니다. 비활성화하면 Codex는 작업 링크만 게시합니다.

---

## 앱 설정

`config.toml`의 `[apps]` 섹션에서 앱(커넥터)의 동작을 세부적으로 제어할 수 있습니다.

### AppConfig (개별 앱 설정)

```toml
# 특정 앱 활성화/비활성화
[apps.my_app]
enabled = true                          # 앱 활성화 여부 (기본값: true)
default_tools_enabled = true            # 앱 내 도구 기본 활성화 여부
default_tools_approval_mode = "auto"    # 도구 승인 모드: auto | prompt | approve
destructive_enabled = false             # destructive_hint 도구 허용 여부
open_world_enabled = false              # open_world_hint 도구 허용 여부

# 개별 도구 설정
[apps.my_app.tools.specific_tool]
enabled = true                          # 특정 도구 활성화/비활성화
approval_mode = "prompt"                # 도구별 승인 모드 오버라이드
```

### 승인 모드 (Approval Mode)

| 모드 | 설명 |
| --- | --- |
| `auto` | 기본 동작 사용 |
| `prompt` | 도구 호출 시 사용자 승인 요청 |
| `approve` | 자동 승인 |

### AppsDefaultConfig (앱 기본 설정)

모든 앱에 적용되는 기본값을 `_default` 키로 설정할 수 있습니다.

```toml
[apps._default]
enabled = true                    # 모든 앱의 기본 활성화 상태
destructive_enabled = false       # destructive 도구 기본 차단
open_world_enabled = false        # open_world 도구 기본 차단
```

### 도구 제안 비활성화

특정 커넥터나 플러그인의 도구 제안을 비활성화할 수 있습니다.

```toml
[tool_suggest]
disabled_tools = [
  { type = "plugin", id = "slack@openai-curated" },
  { type = "connector", id = "connector_google_calendar" },
]
```

---

## 요약

| 기능 | 방법 |
| --- | --- |
| 앱 목록 보기 | `/apps` 명령어 |
| 앱 삽입 | 컴포저에서 `$` 입력 |
| GitHub 코드 리뷰 | `@codex review` |
| Slack 작업 | `@Codex` 멘션 |
| 앱 설정 | `config.toml`의 `[apps]` 섹션 |
