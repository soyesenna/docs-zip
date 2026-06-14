# Routines 및 예약 작업 가이드

> **원문**: [Routines](https://code.claude.com/docs/en/routines) | [Scheduled Tasks](https://code.claude.com/docs/en/scheduled-tasks) | [Desktop Scheduled Tasks](https://code.claude.com/docs/en/desktop-scheduled-tasks)

> Routines는 현재 **research preview** 상태다. 동작, 한도, API 표면이 변경될 수 있다 ("Routines are in research preview. Behavior, limits, and the API surface may change.").

Claude Code는 반복 작업과 예약 실행을 위한 세 가지 메커니즘을 제공한다. 클라우드 인프라에서 실행되는 Routines, 데스크톱 앱의 로컬 예약 작업, 그리고 세션 내 `/loop` 명령어다. 각각의 실행 환경, 지속성, 기능 범위가 다르므로 용도에 맞게 선택해야 한다.

---

## 1. 스케줄링 옵션 비교

| 항목 | Cloud (Routines) | Desktop (예약 작업) | `/loop` (세션 내) |
|---|---|---|---|
| 실행 환경 | Anthropic 클라우드 | 로컬 머신 | 로컬 머신 |
| 머신 켜져 있어야 함 | 아니요 | 예 | 예 |
| 열린 세션 필요 | 아니요 | 아니요 | 예 |
| 재시작 후 유지 | 예 | 예 | `--resume` 시 복원(미만료분) |
| 로컬 파일 접근 | 아니요 (fresh clone) | 예 | 예 |
| MCP 서버 | 작업별 connector 설정 | config 파일 및 connector | 세션에서 상속 |
| 권한 프롬프트 | 없음 (자율 실행) | 작업별 설정 가능 | 세션에서 상속 |
| 커스텀 스케줄 | CLI `/schedule` | 예 | 예 |
| 최소 간격 | 1시간 | 1분 | 1분 |

---

## 2. Routines 개요

Routine은 저장된 Claude Code 설정 -- prompt, 하나 이상의 repository, connector 집합 -- 을 패키징하여 자동 실행하는 기능이다. Anthropic 관리 클라우드 인프라에서 실행되므로 노트북을 닫아도 계속 동작한다.

### 2.1 트리거 유형

각 Routine에는 하나 이상의 트리거를 연결할 수 있다.

| 트리거 | 설명 |
|---|---|
| **Scheduled** | 시간별, 일별, 주별 등 반복 주기 또는 특정 시점 1회 실행 |
| **API** | HTTP POST 요청으로 온디맨드 트리거. bearer token 인증 |
| **GitHub** | pull request 또는 release 이벤트에 반응하여 자동 실행 |

하나의 Routine에 여러 트리거를 조합할 수 있다. 예를 들어 PR review routine을 매일 밤 실행하면서, deploy 스크립트에서 API로도 호출하고, 새 PR 생성 시에도 반응하도록 설정할 수 있다.

### 2.2 요금제 및 접근 권한

- Routines은 Pro, Max, Team, Enterprise 요금제에서 Claude Code on the web 활성화 시 사용 가능
- claude.ai/code/routines 또는 CLI `/schedule`로 생성 및 관리
- Team/Enterprise 관리자는 claude.ai/admin-settings/claude-code에서 Routines 토글로 전체 비활성화 가능
- 비활성화 시 기존 Routine 실행이 중지되고 신규 생성 불가

### 2.3 대표 활용 사례

| 사례 | 트리거 | 설명 |
|---|---|---|
| 백로그 정비 | Scheduled (평일 야간) | connector로 issue tracker 읽기, 라벨/담당자 지정, Slack 요약 |
| 알림 분류 | API | 모니터링 도구에서 임계값 초과 시 트리거, stack trace 분석 후 fix PR 초안 생성 |
| 맞춤 코드 리뷰 | GitHub (`pull_request.opened`) | 팀 리뷰 체크리스트 적용, 인라인 코멘트로 보안/성능/스타일 피드백 |
| 배포 검증 | API | CD 파이프라인 후 실행, smoke test 및 에러 로그 스캔, go/no-go 게시 |
| 문서 동기화 | Scheduled (주간) | 병합된 PR 스캔, 변경된 API 참조 문서 플래그, docs repo에 업데이트 PR |
| 라이브러리 포팅 | GitHub (`pull_request.closed`) | 한 SDK repo의 병합된 PR을 다른 언어 SDK로 포팅하여 PR 생성 |

---

## 3. Routine 생성 및 관리

### 3.1 생성 방법

| 방법 | 설명 |
|---|---|
| **웹** | claude.ai/code/routines에서 폼으로 생성 |
| **Desktop 앱** | 사이드바 Routines > New routine > **Remote** 선택 |
| **CLI** | `/schedule` 명령어로 대화형 생성 |

세 가지 경로 모두 동일한 클라우드 계정에 기록되므로 한 곳에서 만든 Routine이 다른 곳에도 즉시 나타난다.

> **모델 선택**: 웹 폼의 prompt 입력에는 model selector가 포함된다. Claude는 매 실행마다 선택한 모델을 사용한다 ("The prompt input includes a model selector. Claude uses the selected model on every run.").

#### CLI에서 생성

```
/schedule daily PR review at 9am
```

```
/schedule clean up feature flag in one week
```

`/schedule`은 Scheduled 트리거만 생성한다. API 또는 GitHub 트리거를 추가하려면 웹에서 편집해야 한다.

CLI 관리 명령어:

| 명령어 | 기능 |
|---|---|
| `/schedule list` | 모든 Routine 나열 |
| `/schedule update` | Routine 변경 |
| `/schedule run` | 즉시 실행 |

### 3.2 Schedule 트리거 설정

| 프리셋 | 설명 |
|---|---|
| Hourly | 매 시간 |
| Daily | 매일 (시간 선택, 기본 오전 9시) |
| Weekdays | 평일만 |
| Weekly | 주 단위 (요일+시간 선택) |

커스텀 간격(예: 2시간마다, 매월 1일)은 웹에서 가장 가까운 프리셋을 선택한 후 CLI `/schedule update`로 cron 표현식을 직접 설정한다. 최소 간격은 1시간이며, 1시간 미만 표현식은 거부된다.

실행 시간은 로컬 timezone으로 입력하며 자동으로 UTC 변환된다. 실제 시작은 스케줄 시간으로부터 몇 분 지연될 수 있으며(stagger), 지연 오프셋은 Routine마다 일관하게 유지된다.

#### 1회성 실행 (One-off)

특정 시점에 한 번만 실행하고 자동 비활성화되는 스케줄. 실행 후 웹 UI에 **Ran**으로 표시된다.

```
/schedule tomorrow at 9am, summarize yesterday's merged PRs
```

```
/schedule in 2 weeks, open a cleanup PR that removes the feature flag
```

1회성 실행은 일일 Routine 실행 한도(cap)에 포함되지 않는다. 정기 구독 사용량만 소비한다.

### 3.3 API 트리거 설정

웹 UI에서만 추가 가능. 각 Routine에 전용 HTTP endpoint와 bearer token이 부여된다.

#### 트리거 요청 예시

```bash
curl -X POST https://api.anthropic.com/v1/claude_code/routines/trig_01ABCDEFGHJKLMNOPQRSTUVW/fire \
  -H "Authorization: Bearer sk-ant-oat01-xxxxx" \
  -H "anthropic-beta: experimental-cc-routine-2026-04-01" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"text": "Sentry alert SEN-4521 fired in prod. Stack trace attached."}'
```

요청 body의 `text` 필드는 선택 사항이며, 실행 시점의 컨텍스트(알림 본문, 실패 로그 등)를 전달한다. JSON이나 다른 구조화된 페이로드도 그대로 문자열로 전달된다.

#### 응답 예시

```json
{
  "type": "routine_fire",
  "claude_code_session_id": "session_01HJKLMNOPQRSTUVWXYZ",
  "claude_code_session_url": "https://claude.ai/code/session_01HJKLMNOPQRSTUVWXYZ"
}
```

Token은 Routine별로 고유하며, 해당 Routine 트리거 용도로만 scope이 제한된다. Rotate/Revoke는 웹 UI의 동일 모달에서 수행한다.

> `/fire` endpoint는 `experimental-cc-routine-2026-04-01` beta header를 사용한다. Breaking changes는 새로운 날짜가 부여된 beta header 버전 뒤에 배포되며, 호출자가 마이그레이션할 시간을 가질 수 있도록 **가장 최근의 이전 header 버전 두 개**가 계속 작동한다 ("Breaking changes ship behind new dated beta header versions, and the two most recent previous header versions continue to work so that callers have time to migrate.").

#### 공식 API 참조

모든 error response, validation 규칙, 필드 한도를 포함한 전체 API 참조는 Claude Platform 문서의 **Trigger a routine via API**를 참조한다. `/fire` endpoint는 claude.ai 사용자만 사용할 수 있으며 Claude Platform API 표면의 일부가 아니다 ("The `/fire` endpoint is available to claude.ai users only and is not part of the Claude Platform API surface.").

### 3.4 GitHub 트리거 설정

웹 UI에서만 설정 가능. 연결된 repository의 이벤트가 발생하면 자동으로 새 세션이 시작된다.

> **Claude GitHub App 설치 필요**: GitHub 트리거를 사용하려면 구독하려는 repository에 **Claude GitHub App**이 설치되어 있어야 한다 ("The Claude GitHub App must be installed on the repository you want to subscribe to."). CLI에서 `/web-setup`을 실행하면 clone을 위한 repository 접근 권한은 부여되지만, **Claude GitHub App을 설치하지는 않으며 webhook 전달도 활성화되지 않는다**. GitHub 트리거는 Claude GitHub App 설치가 필수다 ("Running `/web-setup` in the CLI grants repository access for cloning, but it does not install the Claude GitHub App and does not enable webhook delivery. GitHub triggers require installing the Claude GitHub App.").

> **Research preview 한도**: Research preview 기간 동안 GitHub webhook 이벤트에는 routine별, 계정별 시간당 한도가 적용된다. 한도를 초과한 이벤트는 window가 reset될 때까지 폐기된다. 현재 한도는 claude.ai/code/routines에서 확인할 수 있다 ("During the research preview, GitHub webhook events are subject to per-routine and per-account hourly caps. Events beyond the limit are dropped until the window resets. See your current limits at claude.ai/code/routines.").

#### 지원 이벤트

| 이벤트 | 트리거 조건 |
|---|---|
| Pull request | PR의 opened, closed, assigned, labeled, synchronized 등 모든 업데이트 |
| Release | release의 created, published, edited, deleted |

#### PR 필터

| 필터 | 매치 대상 |
|---|---|
| Author | PR 작성자 GitHub username |
| Title | PR 제목 텍스트 |
| Body | PR 설명 텍스트 |
| Base branch | PR이 향하는 브랜치 |
| Head branch | PR이 출발하는 브랜치 |
| Labels | PR에 적용된 라벨 |
| Is draft | PR draft 상태 여부 |
| Is merged | PR 병합 여부 |

각 필터에 연산자(equals, contains, starts with, is one of, is not one of, matches regex)를 조합한다. `matches regex`는 전체 필드 값을 대상으로 테스트하므로 부분 일치를 원하면 `.*hotfix.*`처럼 작성해야 한다. 모든 필터 조건이 일치해야 트리거가 발생한다.

세션-이벤트 매핑: 매칭된 GitHub 이벤트마다 독립적인 새 세션이 시작된다. 이벤트 간 세션 재사용은 지원되지 않는다.

### 3.5 Repositories 및 Branch 권한

- 각 실행 시 repository가 clone된다. 기본 브랜치에서 시작하며 prompt에서 지정하면 다른 브랜치도 가능
- 기본적으로 `claude/` prefix 브랜치에만 push 가능. 보호 브랜치 실수 수정 방지
- **Allow unrestricted branch pushes** 활성화 시 특정 repository에 대해 제한 해제 가능

### 3.6 Connectors

Routine은 claude.ai 계정에 연결된 MCP connector를 사용할 수 있다.

| 구분 | 설명 |
|---|---|
| connector 추가 | claude.ai/customize/connectors 또는 Routine 폼에서 직접 추가 |
| 로컬 MCP 서버 | CLI `claude mcp add`로 추가한 서버는 로컬 저장이므로 Routine에서 사용 불가. 대신 claude.ai connector로 추가하거나 repo에 `.mcp.json` 커밋 |
| 기본 동작 | 생성 시 현재 연결된 모든 connector가 포함됨. 불필요한 것은 제거 |

### 3.7 Environment 및 Network 접근

각 Routine은 network 접근, environment 변수, setup script를 제어하는 클라우드 환경에서 실행된다. Routine은 매 실행마다 해당 환경의 network 정책을 상속한다. Cloud environment는 세 가지를 제어한다:

| 항목 | 설명 |
|---|---|
| Network access | 허용된 아웃바운드 호스트 결정 |
| Environment variables | API key, token, secret 등 비밀값 주입 |
| Setup script | 의존성/도구 설치. 결과가 캐시되어 매 세션마다 재실행되지 않는다 |
| Default 환경 | **Trusted** network access 사용. 기본 allowlist(package registry, cloud API 등)만 접근 가능. 허용되지 않은 호스트로의 아웃바운드 요청은 `403` 응답과 `x-deny-reason: host_not_allowed`로 실패한다 ("Outbound requests to other hosts fail with `403` and `x-deny-reason: host_not_allowed`.") |
| 추가 도메인 허용 | 환경 설정에서 **Allowed domains**에 추가 |
| MCP connector 트래픽 | Anthropic 서버를 경유하므로 별도 도메인 허용 불필요 |

### 3.8 Usage 및 한도

- Routine은 대화형 세션과 동일하게 구독 사용량을 소비
- 계정당 일일 실행 횟수 cap 존재. 현재 소비량과 잔여량은 claude.ai/code/routines 또는 claude.ai/settings/usage에서 확인
- Usage credits 활성 시 cap 초과 후에도 종량 과금으로 계속 실행 가능
- 1회성 실행은 일일 cap에서 제외, 정기 구독 사용량만 소비

### 3.9 Routine 관리

상세 페이지에서 수행할 수 있는 작업:

| 작업 | 설명 |
|---|---|
| Run now | 다음 스케줄 대기 없이 즉시 실행 |
| Toggle (Repeats) | 일시정지/재개. 설정은 유지 |
| Edit | 이름, prompt, repo, 환경, connector, 트리거 변경 |
| Delete | Routine 제거. 과거 실행 세션은 유지 |

### 3.10 트러블슈팅

| 문제 | 원인 및 해결 |
|---|---|
| `/schedule`이 "Unknown command" 반환 | (1) Console API key 또는 Bedrock/Vertex/Foundry 인증 사용 중. `/schedule`은 claude.ai 구독 로그인 필요. `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, `apiKeyHelper` 제거 (2) `DISABLE_TELEMETRY`, `DO_NOT_TRACK`, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`, `DISABLE_GROWTHBOOK` 환경변수가 설정됨. Feature flag 비활성화됨 (3) Claude Code on the web 세션 안에 있음. 웹 UI 사용 (4) CLI가 v2.1.81 미만. `claude update` 실행 |
| "Routines are disabled by your organization's policy" | Team/Enterprise 관리자가 Routines 토글을 끔. 서버 측 설정이므로 로컬에서 재정의 불가. 관리자에게 요청 |

---

## 4. 세션 내 예약 작업 (Scheduled Tasks, `/loop`)

세션 범위의 예약 작업. Claude가 prompt를 자동 반복 실행한다. 배포 상태 폴링, PR 감시, 빌드 체크, 세션 내 리마인더에 활용한다.

> 예약 작업(scheduled tasks)은 **Claude Code v2.1.72 이상**이 필요하다. 버전은 `claude --version`으로 확인한다 ("Scheduled tasks require Claude Code v2.1.72 or later. Check your version with `claude --version`.").

> 세션 범위이므로 새 대화를 시작하면 모든 작업이 사라진다. `--resume` 또는 `--continue`로 재개 시 미만료 작업은 복원된다.

### 4.1 `/loop` 명령어

| 입력 형태 | 예시 | 동작 |
|---|---|---|
| 간격 + prompt | `/loop 5m check the deploy` | 고정 간격으로 prompt 실행 |
| prompt만 | `/loop check the deploy` | Claude가 매 반복마다 간격을 동적 선택 |
| 간격만 또는 없음 | `/loop` | 내장 maintenance prompt 실행 (또는 `loop.md`가 있으면 그것 사용) |

다른 명령어를 prompt로 전달할 수도 있다: `/loop 20m /review-pr 1234`

#### 간격 단위

| 단위 | 의미 |
|---|---|
| `s` | 초 (1분으로 올림) |
| `m` | 분 |
| `h` | 시간 |
| `d` | 일 |

cron이 분 단위 정밀도이므로 초는 가장 가까운 분으로 올림된다. 깔끔한 cron step에 매핑되지 않는 간격(예: 7m, 90m)은 가장 가까운 간격으로 반올림된다.

### 4.2 동적 간격 모드

간격을 생략하면 Claude가 매 반복 후 관찰 결과에 따라 1분~1시간 사이의 대기를 선택한다. 빌드가 끝나가거나 PR이 활성이면 짧게, 대기 중인 것이 없으면 길게 기다린다.

동적 모드에서 Claude는 Monitor 도구를 직접 사용할 수도 있다. 이는 폴링보다 토큰 효율이 좋고 반응성이 높다.

> **Bedrock, Vertex AI, Microsoft Foundry 제약**: 간격이 없는 prompt는 동적 스케줄 대신 고정 10분 주기로 실행된다 ("On Bedrock, Vertex AI, and Microsoft Foundry, a prompt with no interval runs on a fixed 10-minute schedule instead.").

동적 스케줄 loop도 예약 작업 목록에 나타나므로 동일하게 목록 조회/취소가 가능하다. 7일 만료 규칙이 적용된다.

### 4.3 내장 Maintenance Prompt

prompt를 생략하면 Claude가 다음을 순서대로 수행한다:

1. 대화에서 완료되지 않은 작업 계속
2. 현재 브랜치의 PR 관리: 리뷰 코멘트, 실패한 CI, merge conflict
3. 대기 중인 것이 없으면 bug hunt, simplification 등 정리 pass

새로운 initiative를 시작하지 않으며, push/delete 등 되돌릴 수 없는 작업은 이미 승인된 내용에 한해서만 진행한다.

> **Bedrock, Vertex AI, Microsoft Foundry 제약**: prompt 없이 `/loop`를 실행하면 maintenance prompt 대신 usage 메시지가 출력된다 ("On Bedrock, Vertex AI, and Microsoft Foundry, `/loop` with no prompt prints the usage message instead of running the maintenance prompt.").

### 4.4 `loop.md` 커스터마이징

`loop.md` 파일로 내장 prompt를 대체할 수 있다. 이 파일은 별개의 예약 작업 목록이 아니라 bare `/loop`를 위한 **단일 기본 prompt**를 정의하며, 명령줄에 prompt를 제공하면 무시된다. 추가 prompt를 함께 예약하려면 `/loop <prompt>`를 사용하거나 Claude에게 직접 요청한다 ("It defines a single default prompt for bare `/loop`, not a list of separate scheduled tasks, and is ignored whenever you supply a prompt on the command line. To schedule additional prompts alongside it, use `/loop <prompt>` or ask Claude directly.").

> **Bedrock, Vertex AI, Microsoft Foundry 제약**: 이 환경에서는 `loop.md`가 읽히지 않으며, prompt 없이 `/loop`를 실행하면 usage 메시지가 출력된다 ("On Bedrock, Vertex AI, and Microsoft Foundry, `loop.md` isn't read and `/loop` with no prompt prints the usage message instead.").

| 경로 | 범위 |
|---|---|
| `.claude/loop.md` | 프로젝트 수준. 두 파일이 모두 있으면 우선 적용 |
| `~/.claude/loop.md` | 사용자 수준. 프로젝트에 `loop.md`가 없을 때 적용 |

파일은 일반 Markdown이며, `/loop` prompt를 직접 입력하는 것처럼 작성한다. 실행 중에도 편집이 가능하며 다음 반복에 반영된다. 내용이 25,000 bytes를 초과하면 잘린다.

### 4.5 Loop 중지

- **Esc 키**: 대기 중인 다음 반복을 취소. loop가 다시 실행되지 않음
- Claude가 자율적으로 완료 판단 시 스스로 종료 가능(동적 모드)
- 고정 간격 loop는 중지하거나 7일이 경과할 때까지 계속 실행

> **Esc 적용 범위**: Claude에게 직접 요청하여 예약한 작업(자연어 예약)은 Esc의 영향을 받지 않으며, 삭제하기 전까지 유지된다 ("Tasks you scheduled by asking Claude directly are not affected by Esc and stay in place until you delete them."). Esc는 `/loop`로 만든 대기 중인 wakeup에만 적용된다.

### 4.6 1회성 리마인더

자연어로 시간을 지정하여 한 번만 실행되는 작업을 생성할 수 있다.

```
remind me at 3pm to push the release branch
```

```
in 45 minutes, check whether the integration tests passed
```

Claude가 cron 표현식으로 변환하여 실행 시점을 확정하고 알려준다. 실행 후 자동 삭제된다.

### 4.7 Cron 도구

| 도구 | 기능 |
|---|---|
| `CronCreate` | 새 작업 예약. 5필드 cron 표현식, prompt, 반복/1회 여부 입력 |
| `CronList` | 모든 예약 작업의 ID, 스케줄, prompt 나열 |
| `CronDelete` | ID로 작업 취소 |

각 작업에는 8자리 ID가 부여된다. 세션당 최대 50개 작업을 보유할 수 있다.

### 4.8 실행 메커니즘

- 스케줄러가 매 초마다 만료 작업을 확인하여 낮은 우선순위로 큐에 추가
- 예약 prompt는 Claude의 응답 턴 사이에 실행됨(응답 중에는 대기)
- 모든 시간은 로컬 timezone 기준

#### Jitter

모든 세션이 동일한 시각에 API를 호출하는 것을 방지하기 위한 deterministic offset:

| 유형 | 지연 범위 |
|---|---|
| 반복 작업 | 최대 30분 (또는 간격의 절반, 1시간 미만 간격의 경우) |
| 1회성 작업 | 정각/30분 기준 최대 90초 조기 fire |

정확한 시간이 중요하면 `:00`이나 `:30`이 아닌 분을 선택(예: `3 9 * * *`). offset은 작업 ID에서 파생되므로 동일 작업은 항상 같은 offset을 갖는다.

#### 7일 만료

반복 작업은 생성 후 7일이 지나면 자동 만료되어 삭제된다. 잊혀진 loop의 실행 기간을 제한하는 안전장치다. 더 오래 실행해야 하면 만료 전에 재생성하거나 Routines/Desktop 예약 작업을 사용한다.

### 4.9 Cron 표현식 참조

표준 5필드 형식: `minute hour day-of-month month day-of-week`

모든 필드는 wildcard(`*`), 단일 값, step(`*/15`), range(`1-5`), 콤마 구분 목록(`1,15,30`)을 지원한다.

| 표현식 | 의미 |
|---|---|
| `*/5 * * * *` | 5분마다 |
| `0 * * * *` | 매 정각 |
| `7 * * * *` | 매 시 7분 |
| `0 9 * * *` | 매일 오전 9시 (로컬) |
| `0 9 * * 1-5` | 평일 오전 9시 |
| `30 14 15 3 *` | 3월 15일 오후 2:30 |

day-of-week는 `0` 또는 `7`이 일요일, `6`이 토요일. `L`, `W`, `?`, `MON`, `JAN` 등 확장 문법은 지원하지 않는다. day-of-month와 day-of-week가 모두 제약되면 둘 중 하나라도 일치하면 매치된다 (vixie-cron semantics).

### 4.10 비활성화

`CLAUDE_CODE_DISABLE_CRON=1` 환경변수를 설정하면 스케줄러가 완전히 비활성화된다. cron 도구와 `/loop`를 사용할 수 없으며, 기존 예약 작업도 실행되지 않는다.

### 4.11 제한 사항

| 제한 | 설명 |
|---|---|
| 세션 필요 | Claude Code가 실행 중이고 idle 상태일 때만 fire |
| 누락된 실행의 보정 없음 | Claude가 긴 요청을 처리 중이면 idle이 된 후 한 번만 실행 |
| 새 대화 시 초기화 | 모든 세션 범위 작업이 삭제됨. `--resume`/`--continue`로 미만료 작업 복원 가능하지만 Background Bash/Monitor 작업은 복원되지 않음 |

---

## 5. Desktop 예약 작업 (Desktop Scheduled Tasks)

Desktop 앱의 Routines 페이지에서 생성하는 로컬 예약 작업. 앱이 열려 있고 컴퓨터가 깨어 있을 때만 실행된다.

### 5.1 생성

Desktop 앱 사이드바에서 **Routines** > **New routine** > **Local**을 선택한다. (Remote를 선택하면 클라우드 Routine이 생성됨.)

| 필드 | 설명 |
|---|---|
| Name | 작업 식별자. lowercase kebab-case로 변환되어 폴더명으로 사용. 전체 작업에서 고유해야 함 |
| Description | 작업 목록에 표시되는 짧은 설명 |
| Instructions | Claude가 실행할 내용. prompt 입력과 동일하게 작성. Permission mode 및 model picker 포함, 작업 폴더 및 worktree 격리 여부 선택 |
| Schedule | 실행 주기. 아래 옵션 참조 |

폴더가 지정되어야 저장할 수 있다. 아직 신뢰하지 않은 폴더면 저장 전 신뢰(trust) 프롬프트가 표시된다.

> **Worktree 기본 동작**: 기본적으로 예약 작업은 커밋되지 않은 변경사항을 포함한 작업 디렉터리의 현재 상태 그대로 실행된다. 작업 생성 시 **worktree 토글**을 켜면 병렬 세션과 동일한 방식으로 각 실행에 격리된 Git worktree를 부여한다 ("By default, scheduled tasks run against whatever state your working directory is in, including uncommitted changes. Enable the worktree toggle when creating the task to give each run its own isolated Git worktree, the same way parallel sessions work.").

세션 내에서 자연어로도 생성 가능: "set up a daily code review that runs every morning at 9am"

### 5.2 Schedule 옵션

| 옵션 | 설명 |
|---|---|
| Manual | 스케줄 없음. Run now 클릭 시에만 실행 |
| Hourly | 매 시간 |
| Daily | 시간 선택 가능 (기본 오전 9시) |
| Weekdays | 평일만 (토/일 제외) |
| Weekly | 요일+시간 선택 |

picker에서 제공하지 않는 간격(예: 15분마다, 매월 1일, 특정 시간 1회)은 Desktop 세션에서 Claude에게 자연어로 요청하면 설정할 수 있다.

### 5.3 실행 동작

- Desktop 앱이 열려 있는 동안 매 분마다 스케줄을 확인
- 작업이 만료되면 새 세션이 시작됨 (수동 세션과 독립)
- API 트래픽 분산을 위해 각 작업에 몇 분의 deterministic delay가 추가됨
- 실행 시 Desktop 알림이 표시되고 사이드바 **Scheduled** 섹션에 세션이 나타남
- 세션에서 파일 편집, 명령 실행, commit, PR 생성 등 모든 작업 가능

### 5.4 컴퓨터 절전 시

- 작업은 Desktop 앱이 실행 중이고 컴퓨터가 깨어 있을 때만 실행됨
- 컴퓨터가 절전 모드이면 해당 시간의 실행이 건너뜀
- Settings > Desktop app > General에서 **Keep computer awake**를 활성화하면 유휴 절전 방지 가능 (단, 노트북 덮개를 닫으면 여전히 절전)
- 컴퓨터가 꺼져 있을 때도 실행해야 하면 Remote routine 사용

### 5.5 누락된 실행 보정 (Missed Runs)

앱 시작 또는 컴퓨터가 깨어날 때 Desktop이 지난 7일간의 누락된 실행을 확인한다. 누락이 있으면 가장 최근 시간에 대해 정확히 **1회** catch-up 실행을 하고, 그 이전의 것은 폐기한다. 예를 들어 6일치를 놓친 daily 작업은 깨어날 때 1회만 실행된다 ("A daily task that missed six days runs once on wake."). Desktop은 catch-up 실행이 시작될 때 알림을 표시한다 ("Desktop shows a notification when a catch-up run starts.").

시간이 중요한 prompt에는 가드레일을 추가하는 것이 좋다:

```
Only review today's commits. If it's after 5pm,
skip the review and just post a summary of what was missed.
```

### 5.6 권한

| 항목 | 설명 |
|---|---|
| Permission mode | 각 작업마다 개별 설정 |
| 글로벌 설정 | `~/.claude/settings.json`의 Allow 규칙도 적용 |
| Ask mode에서 권한 부족 | 실행이 멈추고 승인 대기. 사이드바 세션에서 나중에 응답 가능 |
| 권한 사전 승인 | Run now로 테스트 실행 후 "always allow" 선택하면 이후 실행에서 자동 승인 |

### 5.7 작업 관리

상세 페이지에서 수행:

| 작업 | 설명 |
|---|---|
| Run now | 즉시 실행 |
| Status | Active/Paused 토글 |
| Edit | Instructions, schedule, folder 등 변경 |
| Review history | 과거 실행 이력 확인. 건너뛴 실행은 hover로 사유 확인 (절전, 이전 실행 진행 중, 다른 작업 실행 중 등). 이전 항목을 불러오려면 **Show more** 클릭 |
| Always allowed panel | 승인된 권한 확인 및 철회 |
| Delete | 작업 제거 및 세션 보관. "Also delete files on disk" 체크 시 `~/.claude/scheduled-tasks/` 데이터도 삭제. 삭제는 상세 페이지의 **Delete** 버튼으로만 가능 |

세션 내 자연어로도 관리 가능(list/create/edit/pause): "pause my dependency-audit task", "show me my scheduled tasks". 단 **삭제는 자연어로 불가**하며 상세 페이지의 Delete 버튼을 사용해야 한다 ("To delete a task, use the Delete button on its detail page.").

### 5.8 SKILL.md로 온디스크 편집

`~/.claude/scheduled-tasks/<task-name>/SKILL.md`를 직접 편집할 수 있다. `CLAUDE_CONFIG_DIR`가 설정된 경우 해당 경로 하위를 사용한다 ("open `~/.claude/scheduled-tasks/<task-name>/SKILL.md` (or under `CLAUDE_CONFIG_DIR` if set)."). YAML frontmatter(`name`, `description`)와 prompt body로 구성되며, 다음 실행부터 반영된다. Schedule, folder, model, enabled 상태는 이 파일에 없으므로 Edit 폼이나 Claude에게 요청해야 한다.

### 5.9 동적 스케줄 변경

실행 중인 세션 내에서 `update_scheduled_task` MCP 도구를 사용하면 작업이 자신의 스케줄이나 prompt를 수정할 수 있다. 예를 들어 release branch가 생성된 것을 감지하면 코드 리뷰 작업을 더 이른 시간으로 재스케줄할 수 있다.

---

## 6. 지원 플랫폼별 차이점

| 기능 | Cloud Routines | Desktop 예약 작업 | CLI `/loop` |
|---|---|---|---|
| 웹 UI 관리 | claude.ai/code/routines | Desktop 앱 Routines > Local | 세션 내 자연어 |
| CLI 관리 | `/schedule` | 세션 내 자연어 | 자연어 + Cron 도구 |
| 트리거 유형 | Schedule, API, GitHub | Schedule | Schedule (고정/동적) |
| GitHub 이벤트 | 지원 | 미지원 | 미지원 |
| API 트리거 | 지원 | 미지원 | 미지원 |
| 실행 환경 | Anthropic 클라우드 (fresh clone) | 로컬 머신 | 로컬 머신 (현재 세션) |
| 컴퓨터 상태 무관 | 예 | 아니요 | 아니요 |
| 세션 필요 | 아니요 | 아니요 | 예 |

---

## 7. 모범 사례

### 선택 기준

| 상황 | 권장 방식 |
|---|---|
| 24/7 무인 실행, GitHub 이벤트/API 연동 | Cloud Routines |
| 로컬 파일 접근 필요, 컴퓨터가 항상 켜져 있음 | Desktop 예약 작업 |
| 임시 폴링, 세션 내 단기 반복 | `/loop` |
| 특정 시점 1회 실행 (세션 내) | 자연어 리마인더 |
| 특정 시점 1회 실행 (독립적) | Cloud Routine one-off 또는 Desktop 예약 |

### Routine 설계 팁

| 팁 | 설명 |
|---|---|
| 최소 권한 원칙 | connector, network 접근, branch push 권한을 작업에 필요한 것만 부여 |
| prompt에 가드레일 추가 | 시간이 지난 실행이 catch-up으로 늦게 돌아올 경우를 대비해 조건부 건너뛰기 로직 작성 |
| 테스트 후 배포 | Run now로 1회 실행해 권한 프롬프트를 미리 해결 |
| 사용량 모니터링 | claude.ai/settings/usage에서 일일 cap 및 사용량 확인 |
| 1회성 vs 반복 | 1회성 실행은 일일 cap에서 제외되므로 단발성 작업에 적극 활용 |

### 세션 내 스케줄링 팁

| 팁 | 설명 |
|---|---|
| 동적 간격 활용 | 폴링 간격을 모르겠으면 prompt만 제공하고 간격은 Claude가 결정하게 하라 |
| `loop.md`로 프로젝트 커스터마이징 | 팀별 maintenance routine을 `.claude/loop.md`에 정의 |
| Esc로 즉시 중지 | 대기 중인 loop는 Esc로 깔끔하게 중단 |
| 7일 만료 주의 | 장기 실행이 필요하면 만료 전 재생성하거나 Routines으로 전환 |
| 정확한 시간이 필요하면 `:00`/`:30` 피하기 | Jitter가 적용되지 않는 분을 선택 |
