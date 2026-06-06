# Codex - 플러그인 및 연동

> Codex의 플러그인 시스템, 외부 서비스 연동, 그리고 Sites 호스팅에 대해 설명합니다.

**참조**: <https://developers.openai.com/codex/integrations/github>, <https://developers.openai.com/codex/integrations/slack>, <https://developers.openai.com/codex/integrations/linear>, <https://developers.openai.com/codex/plugins>, <https://developers.openai.com/codex/sites>

---

## 플러그인 및 연동 개요

Plugins는 skills, app 연동, MCP 서버를 재사용 가능한 워크플로로 묶어 Codex의 기능을 확장합니다.

플러그인이 포함할 수 있는 요소:

| 구성 요소 | 설명 |
| --- | --- |
| Skills | 특정 작업에 대한 재사용 가능한 지침. 필요할 때 Codex가 로드 |
| Apps | GitHub, Slack, Google Drive 등 외부 도구와의 연결 |
| MCP servers | 로컬 프로젝트 외부의 시스템에서 추가 도구나 공유 정보에 접근 |

### 지원 플러그인 및 연동

| 플러그인/연동 | 유형 | 설명 |
| --- | --- | --- |
| GitHub Code Review | 연동(Integration) | PR 코드 리뷰 자동화 |
| Slack | 연동(Integration) | 채널/스레드에서 작업 실행 |
| Linear | 연동(Integration) | 이슈에서 작업 위임 및 triage 자동화 |
| Sites | 플러그인 | 웹사이트/앱 생성 및 배포 호스팅 |
| Codex Security | 플러그인 | 인가된 코드 스캔 및 취약점 확인 |
| Gmail | 플러그인 | 이메일 읽기 및 관리 |
| Google Drive | 플러그인 | Drive, Docs, Sheets, Slides 연동 |

---

## 플러그인 설치 및 사용

### Codex 앱에서의 Plugin Directory

Codex 앱에서 **Plugins**를 열어 큐레이션된 플러그인을 탐색하고 설치합니다.

플러그인 디렉토리 분류:

| 카테고리 | 설명 |
| --- | --- |
| Curated by OpenAI | 모든 Codex 사용자에게 제공되는 추천 플러그인 |
| Shared with you | ChatGPT 워크스페이스 구성원이 공유한 플러그인 |
| Created by you | 직접 생성하거나 워크스페이스에 추가한 플러그인 |

### CLI에서 플러그인 관리

```
codex
/plugins
```

CLI 플러그인 브라우저는 마켓플레이스별로 그룹화됩니다. 마켓플레이스 탭으로 소스를 전환하고, 플러그인 상세를 확인하며, 설치/제거할 수 있습니다. 설치된 플러그인에서 `Space`를 눌러 활성화 상태를 토글합니다.

### 플러그인 설치 및 호출

1. 플러그인 디렉토리에서 검색 또는 탐색 후 상세를 엽니다.
2. 설치 버튼을 선택합니다. (앱: 플러스 버튼 또는 **Add to Codex**, CLI: `Install plugin`)
3. 외부 앱 연결이 필요한 경우 안내에 따라 인증합니다.
4. 설치 후 새 스레드를 시작하고 플러그인을 사용합니다.

플러그인 호출 방법:

| 방식 | 예시 | 사용 시기 |
| --- | --- | --- |
| 작업 직접 설명 | "Summarize unread Gmail threads from today" | Codex가 적절한 도구를 자동 선택하게 할 때 |
| 특정 플러그인 지정 | `@Gmail` 또는 `@Sites` | 특정 플러그인이나 skill을 명시할 때 |

### 권한 및 데이터 공유

플러그인 설치 시 워크플로가 Codex에 제공되지만, 기존 승인 설정은 그대로 유지됩니다.

- Bundled skills는 설치 즉시 사용 가능
- 플러그인에 apps가 포함된 경우 ChatGPT에서 설치 또는 로그인 필요
- MCP 서버가 포함된 경우 추가 설정이나 인증이 필요할 수 있음
- 외부 서비스를 통한 데이터 전송 시 해당 앱의 약관 및 개인정보처리방침이 적용

### 플러그인 제거 및 비활성화

플러그인 브라우저에서 다시 열어 **Uninstall plugin**을 선택합니다. 제거해도 bundled apps는 ChatGPT에서 관리할 때까지 설치된 상태로 유지됩니다.

설치 상태를 유지하면서 비활성화하려면 `~/.codex/config.toml`에서 설정 후 Codex를 재시작합니다.

```toml
[plugins."gmail@openai-curated"]
enabled = false
```

---

## GitHub Code Review

Codex를 사용하면 GitHub Pull Request에 대한 고품질 코드 리뷰를 수행할 수 있습니다. Codex는 PR diff를 검토하고, 리포지토리 가이드라인을 따르며, 심각한 이슈에 집중한 표준 GitHub 코드 리뷰를 게시합니다.

### 사전 준비

- 리뷰할 리포지토리에 대해 Codex Cloud가 설정되어 있어야 합니다.
- Codex code review 설정에 접근할 수 있어야 합니다.
- 리포지토리별 리뷰 가이드라인을 원하는 경우 `AGENTS.md` 파일이 필요합니다.

### 설정

1. Codex Cloud를 설정합니다.
2. Codex 설정으로 이동합니다.
3. 해당 리포지토리의 **Code review** 토글을 켭니다.

### 리뷰 요청

PR 코멘트에서 `@codex review`를 멘션합니다.

```
@codex review
```

Codex는 팀원처럼 PR에 리뷰를 게시하며, **P0 및 P1 이슈**만 플래그하여 리뷰 코멘트가 우선순위가 높은 위험에 집중되도록 합니다.

### 자동 리뷰

**Automatic Reviews**를 활성화하면 누군가 새 PR을 열 때 `@codex review` 코멘트 없이도 자동으로 리뷰가 게시됩니다.

### 커스터마이징

#### AGENTS.md 리뷰 가이드라인

리포지토리에 `AGENTS.md` 파일을 추가하여 **Review guidelines**를 정의합니다.

```markdown
## Review guidelines

- Don't log PII.
- Verify that authentication middleware wraps every route.
```

Codex는 변경된 파일에 가장 가까운 `AGENTS.md`의 가이드라인을 적용합니다. 특정 패키지에 추가 검증이 필요한 경우 더 깊은 디렉토리에 구체적인 지침을 배치할 수 있습니다.

문서 내 오타를 플래그하려면 `AGENTS.md`에 가이드라인을 추가합니다. (예: "Treat typos in docs as P1.")

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

### 기타 작업 위임

`@codex`와 함께 `review` 외의 다른 명령을 사용하면 Codex가 PR을 컨텍스트로 사용하는 클라우드 작업을 시작합니다.

```
@codex fix the CI failures
```

### Troubleshoot

Codex가 반응하지 않거나 리뷰를 게시하지 않는 경우:

- Codex 설정에서 해당 리포지토리에 **Code review**가 활성화되어 있는지 확인
- PR이 Codex Cloud가 설정된 리포지토리에 속해 있는지 확인
- PR 코멘트에 정확한 트리거 `@codex review`를 사용했는지 확인
- 자동 리뷰의 경우 **Automatic Reviews**가 활성화되어 있고 PR 이벤트가 리뷰 트리거 설정과 일치하는지 확인

---

## Slack 통합

Slack 채널과 스레드에서 `@Codex`를 멘션하여 코딩 작업을 시작할 수 있습니다. Codex가 클라우드 작업을 생성하고 결과로 답장합니다.

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

### Data usage, privacy, and security

`@Codex`를 멘션하면 Codex가 요청을 이해하고 작업을 생성하기 위해 메시지와 스레드 히스토리를 수신합니다.

- 데이터 처리는 OpenAI의 Privacy Policy, Terms of Use 및 기타 적용 가능한 정책을 따릅니다.
- 보안에 대한 자세한 내용은 Codex security documentation을 참조하십시오.
- Codex는 대규모 언어 모델을 사용하므로 실수를 할 수 있습니다. 항상 답변과 diff를 검토하십시오.

### Tips and troubleshooting

| 문제 | 해결 방법 |
| --- | --- |
| Missing connections | Slack 또는 GitHub 연결을 확인할 수 없는 경우, Codex가 재연결 링크로 답장 |
| Unexpected environment choice | 스레드에서 원하는 환경을 지정한 후 (예: `Please run this in openai/openai (applied)`) `@Codex`를 다시 멘션 |
| Long or complex threads | 최신 메시지에 핵심 내용을 요약하여 컨텍스트 누락 방지 |
| Workspace posting | 일부 Enterprise 워크스페이스는 최종 답변 게시를 제한. 이 경우 작업 링크를 열어 진행 상황과 결과 확인 |
| More help | OpenAI Help Center 참조 |

---

## Linear 통합

Linear 이슈에서 작업을 위임할 수 있습니다. 이슈를 Codex에 할당하거나 코멘트에서 `@Codex`를 멘션하면 Codex가 클라우드 작업을 생성하고 진행 상황과 결과를 답장합니다.

> Codex in Linear는 유료 플랜에서 사용 가능합니다 (Pricing 참조). Enterprise 플랜의 경우 ChatGPT 워크스페이스 관리자에게 Codex cloud tasks를 활성화하고 커넥터 설정에서 **Codex for Linear**를 켜달라고 요청하십시오.

### 설정

1. Codex에서 GitHub를 연결하고 작업할 리포지토리에 대한 환경을 생성하여 Codex Cloud 작업을 설정합니다.
2. Codex 설정에서 **Codex for Linear**를 워크스페이스에 설치합니다.
3. Linear 이슈의 코멘트 스레드에서 `@Codex`를 멘션하여 Linear 계정을 연결합니다.

### 이슈 할당

설치 후 이슈를 팀원에게 할당하듯이 Codex에게 할당할 수 있습니다. Codex가 작업을 시작하고 이슈에 업데이트를 게시합니다.

코멘트 스레드에서 `@Codex`를 멘션하여 작업을 위임하거나 질문할 수도 있습니다. Codex가 답장한 후 스레드에서 후속 질문을 하면 같은 세션이 계속됩니다.

특정 리포지토리를 지정하려면 코멘트에 포함합니다. (예: `@Codex fix this in openai/codex`)

진행 상황 추적:

- 이슈의 **Activity**에서 진행 업데이트 확인
- 작업 링크를 열어 상세 진행 상황 확인

작업이 완료되면 Codex가 요약과 완료된 작업 링크를 게시하여 PR을 생성할 수 있습니다.

### 환경 선택 방식

| 조건 | 동작 |
| --- | --- |
| 이슈 컨텍스트 기반 | Linear가 이슈 컨텍스트를 기반으로 리포지토리를 제안하고 Codex가 매칭되는 환경 선택 |
| 모호한 요청 | 사용자가 가장 최근에 사용한 환경으로 폴백 |
| 기본 리포지토리 | 환경의 리포지토리 맵에서 첫 번째 리포지토리의 기본 브랜치 사용 |
| 적합한 환경 없음 | Linear에서 해결 방법 안내 후 재시도 |

### Triage rules

Triage rules를 사용하여 이슈를 Codex에 자동 할당할 수 있습니다.

1. Linear에서 **Settings**로 이동합니다.
2. **Your teams**에서 팀을 선택합니다.
3. 워크플로 설정에서 **Triage**를 열고 활성화합니다.
4. **Triage rules**에서 규칙을 생성하고 **Delegate** > **Codex**를 선택합니다 (및 기타 설정할 속성).

Triage가 활성화되면 새 이슈가 triage에 들어올 때 자동으로 Codex에 할당됩니다. Triage rules를 사용하면 Codex가 이슈 생성자의 계정으로 작업을 실행합니다.

### MCP 서버 로컬 연결

Codex 앱, CLI, 또는 IDE Extension에서 로컬로 Linear 이슈에 접근하려면 Linear MCP(Model Context Protocol) 서버를 구성합니다.

#### CLI 사용 (권장)

```
codex mcp add linear --url https://mcp.linear.app/mcp
```

Linear 계정으로 로그인하고 Codex에 연결하라는 안내가 표시됩니다.

#### 수동 구성

1. 편집기에서 `~/.codex/config.toml`을 엽니다.
2. 다음을 추가합니다.

```toml
[mcp_servers.linear]
url = "https://mcp.linear.app/mcp"
```

3. `codex mcp login linear`를 실행하여 로그인합니다.

---

## Sites (호스팅)

Sites 플러그인을 사용하면 Codex가 웹사이트, 웹 앱, 게임을 생성, 저장, 배포, 검사할 수 있습니다. 별도의 배포 워크플로를 설정하지 않고도 프롬프트 또는 호환 가능한 기존 프로젝트를 호스팅된 사이트로 전환할 수 있습니다.

> Sites는 preview 상태이며 현재 ChatGPT Business 및 Enterprise 워크스페이스에서 사용 가능합니다. 추가 플랜은 순차적으로 지원될 예정입니다. ChatGPT Enterprise 워크스페이스의 경우 관리자가 역할 기반 접근 제어(RBAC)를 통해 활성화해야 합니다.

### 설정

1. **Enterprise 워크스페이스 활성화**: ChatGPT Enterprise를 사용하는 경우 워크스페이스 관리자에게 ChatGPT 관리자 설정의 RBAC 컨트롤에서 Sites를 켜달라고 요청합니다. ChatGPT Business 워크스페이스는 기본적으로 활성화되어 있으므로 이 단계를 건너뛸 수 있습니다.
2. **Sites 플러그인 추가**: Sites가 이미 사용 가능하지 않은 경우 Codex 앱에서 **Plugins**를 열고 **Sites**를 찾아 추가합니다. 플러그인 설치 후 새 스레드를 시작합니다.
3. **Sites 작업 시작**: 스레드에서 생성 또는 게시할 사이트를 설명합니다. 특히 호스팅 배포로 끝나야 하는 작업의 경우 `@Sites`로 명시적으로 지정할 수 있습니다.

### 프롬프트 예시

새 웹사이트, 대시보드, 내부 도구의 경우:

```
@Sites Build a project request dashboard for my operations team. Let team
members submit requests, see who owns each one, update the status, and filter
the list. Require people to sign in with their workspace account, and keep the
request data saved between visits.
```

기존 프로젝트 배포:

```
@Sites Deploy this project. Check whether it is compatible with Sites, make any
required changes, and give me the deployment URL.
```

지속적 애플리케이션 데이터 또는 업로드 파일이 필요한 경우:

```
@Sites Add persistent player scores and avatar uploads to this game. Use
the appropriate Sites storage and deploy the updated game.
```

### .openai/hosting.json

Sites 프로젝트는 로컬 소스 프로젝트를 Sites를 통해 관리되는 호스팅에 연결합니다. 이 연결 정보와 선택적 스토리지 바인딩 이름은 `.openai/hosting.json`에 저장됩니다.

새로 생성된 로컬 스타터는 `project_id` 없이 시작할 수 있으며, Sites가 호스팅 프로젝트를 프로비저닝한 후 추가합니다.

예시 - 관계형 데이터베이스 바인딩을 사용하고 파일 스토리지가 없는 프로비저닝된 사이트:

```json
{
  "project_id": "<project-id>",
  "d1": "DB",
  "r2": null
}
```

### 게시 단계

Sites 게시는 두 가지 단계로 구분됩니다.

| 단계 | 설명 |
| --- | --- |
| Save a version | Codex가 배포 가능한 사이트를 빌드하고 해당 버전을 빌드에 사용된 소스 Git 커밋과 연결. 검토 가능한 배포 후보가 필요할 때 사용 |
| Deploy a version | 저장된 버전을 게시하고 배포 성공 시 프로덕션 URL을 보고. 선택한 대상이 사이트에 접근할 의도가 있을 때만 사용 |

이전 배포 후보를 확인하려면 Codex에게 저장된 버전을 나열하거나 검사하도록 요청합니다.

### 사이트 유형 및 스토리지

Sites는 Cloudflare Worker 호환 출력을 ES 모듈로 빌드하는 프로젝트를 호스팅합니다.

| 사이트 요구사항 | 요청 내용 |
| --- | --- |
| 콘텐츠 중심 웹사이트 또는 랜딩 페이지 | 경험에 필요하지 않은 한 영구 애플리케이션 상태가 없는 사이트 |
| 저장된 레코드, 사용자 진행 상황, 게임 점수 | D1 - 내구성 있는 구조화된 데이터를 위한 관계형 데이터베이스 |
| 이미지, 문서, 오디오, 비디오 등 업로드 파일 | R2 - 파일용 오브젝트 스토리지 |
| 검색 가능한 메타데이터가 있는 업로드 파일 | 메타데이터용 D1 + 파일 콘텐츠용 R2 |
| 현재 워크스페이스 사용자의 신원이 필요한 내부 사이트 | Workspace-authenticated user identity |
| 공개 로그인 또는 외부 인증 제공자 | Authentication-enabled Sites 프로젝트 |

임시 표시 상태(테마 선택, 배너 닫기 등)에는 내구성 있는 스토리지를 요청하지 마십시오. 호스팅된 사이트가 기억해야 할 제품 데이터에만 요청하십시오.

### 접근 제어

배포된 URL을 공유하기 전에 대상을 설정합니다. 새 사이트의 경우 콘텐츠, 데이터 처리, 예상 대상을 검토할 때까지 접근을 소유자 및 워크스페이스 관리자로 제한합니다.

| 접근 모드 | 접근 가능한 사용자 |
| --- | --- |
| Owner and admins (`admins_only`) | 사이트 소유자 및 워크스페이스 관리자 |
| Workspace (`workspace_all`) | 워크스페이스의 모든 활성 사용자 |
| Custom (`custom`) | 지정한 특정 활성 사용자 또는 워크스페이스 그룹 (소유자는 항상 허용) |

```
@Sites Change this deployed site's access to everyone in my workspace after
showing me the current site and confirming the deployment URL.
```

### 런타임 환경 변수 구성

Codex 앱 사이드바에서 **Sites**를 열고 프로젝트를 선택하여 Sites 패널에서 호스팅된 환경 변수 및 시크릿을 추가, 업데이트 또는 제거합니다. 이 값은 `.openai/hosting.json`에 저장하지 마십시오. 로컬 `.env` 및 `.env.example` 파일을 로컬 개발에 필요한 키와 정렬하고, 시크릿 값을 커밋하지 마십시오.

환경 값을 추가, 업데이트 또는 제거한 후 Codex에게 승인된 저장 버전을 재배포하도록 요청하여 다음 배포가 업데이트된 구성을 사용하도록 합니다.

### 배포 전 체크리스트

- Codex 리뷰 창에서 소스 변경 사항 및 데이터베이스 마이그레이션을 검토
- 빌드가 성공했고 선택한 저장 버전이 게시하려는 버전인지 확인
- 의도한 대상만 사이트에 접근할 수 있는지 확인
- 런타임 시크릿 값을 Sites를 통해 구성했고 소스 파일에 커밋하지 않았는지 확인
- 배포 후 Codex에게 배포 상태와 프로덕션 URL을 확인하도록 요청한 후 공유

---

## 앱 설정

앱 및 커넥터의 세부 설정은 `config.toml`의 `[apps]` 섹션에서 제어합니다. 상세한 설정 항목은 Config Reference 문서를 참조하십сти오.

### 주요 설정 항목

| 설정 | 설명 |
| --- | --- |
| `enabled` | 앱 활성화/비활성화 |
| `default_tools_enabled` | 앱 내 도구 기본 활성화 여부 |
| `default_tools_approval_mode` | 도구 승인 모드: `auto`, `prompt`, `approve` |
| `destructive_enabled` | destructive_hint 도구 허용 여부 |
| `open_world_enabled` | open_world_hint 도구 허용 여부 |

### 승인 모드 (Approval Mode)

| 모드 | 설명 |
| --- | --- |
| `auto` | 기본 동작 사용 |
| `prompt` | 도구 호출 시 사용자 승인 요청 |
| `approve` | 자동 승인 |

---

## 요약

| 기능 | 방법 |
| --- | --- |
| 플러그인 목록 보기 | Codex 앱: **Plugins**, CLI: `/plugins` |
| 플러그인 호출 | `@` 입력 후 플러그인명 지정 또는 작업 직접 설명 |
| GitHub 코드 리뷰 | PR 코멘트에서 `@codex review` |
| Slack 작업 | 채널/스레드에서 `@Codex` 멘션 |
| Linear 작업 위임 | 이슈를 Codex에 할당 또는 `@Codex` 멘션 |
| Linear triage 자동화 | Linear Settings > Triage rules > Delegate > Codex |
| Linear MCP 로컬 연결 | `codex mcp add linear --url https://mcp.linear.app/mcp` |
| Sites 배포 | `@Sites`로 사이트 생성 후 save > deploy |
| 앱 설정 | `~/.codex/config.toml`의 `[apps]` 섹션 |
| 플러그인 비활성화 | `~/.codex/config.toml`에서 `enabled = false` 설정 |
