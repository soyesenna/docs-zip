# Dynamic Workflows, Agent Teams, Agent View 및 병렬 실행

> **원문**: [Dynamic Workflows](https://code.claude.com/docs/en/workflows) | [Agent Teams](https://code.claude.com/docs/en/agent-teams) | [Agent View](https://code.claude.com/docs/en/agent-view) | [Running Agents in Parallel](https://code.claude.com/docs/en/agents) | [Goal](https://code.claude.com/docs/en/goal)

이 문서는 Claude Code의 다중 에이전트 오케스트레이션 기능을 다룹니다. Dynamic Workflows로 수십~수백 개의 에이전트를 스크립트로 조정하고, Agent Teams로 다중 세션을 협업시키며, Agent View로 모든 백그라운드 세션을 한 화면에서 관리하는 방법을 설명합니다.

---

## 병렬 실행 방식 비교

Claude Code는 네 가지 병렬 실행 접근 방식을 제공합니다. 각 방식은 조정자, 통신 방식, 격리 수준이 다릅니다.

| 접근 방식 | 정의 | 언제 사용하는가 |
| --- | --- | --- |
| **Subagents** | 세션 내에서 Claude가 생성하는 작업자 | 사이드 태스크가 메인 대화를 검색 결과나 로그로 채울 때 |
| **Agent View** | `claude agents`로 여는 백그라운드 세션 관리 화면. Research preview | 여러 독립 태스크를 위임하고 한눈에 상태를 확인할 때 |
| **Agent Teams** | 공유 태스크 목록과 메시징으로 조정되는 다중 세션. 실험적, 기본 비활성화 | Claude가 프로젝트를 분할하고 작업자 간 동기화가 필요할 때 |
| **Dynamic Workflows** | 여러 subagent를 실행하고 결과를 교차 검증하는 스크립트. Research preview | 소수의 subagent로 감당 안 되는 규모, 코드베이스 전체 감사, 500파일 마이그레이션, 교차 검증 연구 |

### 핵심 차이점 비교

|  | Subagents | Skills | Agent Teams | Workflows |
| --- | --- | --- | --- | --- |
| **정체** | Claude가 생성하는 작업자 | Claude가 따르는 지침 | 리드 에이전트가 피어 세션을 감독 | 런타임이 실행하는 스크립트 |
| **다음 단계 결정** | Claude, 턴별 | Claude, 프롬프트에 따라 | 리드 에이전트, 턴별 | 스크립트 자체 |
| **중간 결과 저장소** | Claude의 컨텍스트 윈도우 | Claude의 컨텍스트 윈도우 | 공유 태스크 목록 | 스크립트 변수 |
| **재사용 가능한 것** | 작업자 정의 | 지침 | 팀 정의 | 오케스트레이션 자체 |
| **규모** | 턴당 몇 개의 위임 | Subagent와 동일 | 소수의 장기 실행 피어 | 실행당 수십~수백 개 에이전트 |
| **중단 시** | 턴 재시작 | 턴 재시작 | 팀원은 계속 실행 | 같은 세션에서 이어서 실행 가능 |

### 선택 기준

- **누가 작업을 조정하는가?**
  - Claude가 한 대화에서 위임하고 결과를 수집: Subagents
  - 독립 태스크를 위임하고 나중에 확인: Agent View
  - Claude가 계획, 할당, 감독: Agent Teams
  - 스크립트가 판단 대신 계획을 보관: Dynamic Workflows

- **작업자 간 통신이 필요한가?** Subagent는 호출자에게만 결과를 보고, Agent View 세션은 사용자에게만 보고. Agent Teams의 팀원은 공유 태스크 목록과 직접 메시징으로 소통.

- **같은 파일을 편집하는가?** Worktree로 격리. Subagent와 직접 실행 세션은 각각 별도의 worktree 사용 가능. Agent Teams는 팀원을 worktree로 격리하지 않으므로 파일 소유를 분할해야 함.

---

## Dynamic Workflows

Dynamic Workflow는 대규모 subagent를 오케스트레이션하는 JavaScript 스크립트입니다. Claude가 태스크에 맞는 스크립트를 작성하고, 런타임이 백그라운드에서 실행하면서 세션은 응답성을 유지합니다.

### 번들 워크플로우

| 명령어 | 기능 |
| --- | --- |
| `/deep-research <question>` | 여러 관점에서 웹 검색을 확장, 소스를 가져와 교차 검증, 각 주장에 투표하고 인용된 보고서 반환. WebSearch 도구 필요 |

직접 저장한 워크플로우도 동일한 방식으로 명령어가 되어 `/` 자동완성에 나타납니다.

### 워크플로우 실행

#### 프롬프트에서 요청

프롬프트에 `ultracode` 키워드를 포함하거나 자연어로 "use a workflow"라고 요청합니다.

```
ultracode: audit every API endpoint under src/routes/ for missing auth checks
```

의도하지 않았다면 macOS에서 `Option+W`, Windows/Linux에서 `Alt+W`로 하이라이트를 해제할 수 있습니다. `/config`에서 Ultracode 키워드 트리거를 끌 수도 있습니다.

#### Ultracode 자동 결정

Ultracode는 `xhigh` reasoning effort와 자동 워크플로우 오케스트레이션을 결합한 설정입니다. 활성화하면 Claude가 모든 실질적 태스크에 대해 워크플로우를 계획합니다.

```
/effort ultracode
```

- 현재 세션에만 적용, 새 세션에서 리셋
- `xhigh` effort를 지원하는 모델에서만 사용 가능
- `/effort high`로 복귀 가능

#### 실행 전 승인

CLI에서 실행 시 계획된 단계가 표시되며 다음 옵션이 제공됩니다.

- **Yes, run it**: 실행 시작
- **Yes, and don't ask again for `<name>` in `<path>`**: 해당 프로젝트에서 이 워크플로우는 다시 묻지 않음
- **View raw script**: 스크립트 확인
- **No**: 취소

권한 모드별 승인 동작:

| 권한 모드 | 승인 프롬프트 시점 |
| --- | --- |
| Default, accept edits | 매 실행 (해당 워크플로우에서 "don't ask again"을 선택하지 않은 경우) |
| Auto | 첫 실행만. ultracode 활성 시 생략 |
| Bypass permissions, `claude -p`, Agent SDK | 승인 없이 즉시 실행 |

#### 실행 상태 모니터링

| 키 | 동작 |
| --- | --- |
| `↑` / `↓` | 단계 또는 에이전트 선택 |
| `Enter` 또는 `→` | 선택한 단계로 드릴인, 에이전트의 프롬프트·도구 호출·결과 확인 |
| `Esc` | 한 단계 뒤로 |
| `j` / `k` | 에이전트 상세 스크롤 |
| `p` | 실행 일시정지/재개 |
| `x` | 선택한 에이전트 정지, 또는 실행 중이면 전체 워크플로우 정지 |
| `r` | 선택한 실행 중인 에이전트 재시작 |
| `s` | 실행 스크립트를 명령어로 저장 |

### 워크플로우 저장 및 재사용

`/workflows`에서 원하는 실행을 선택하고 `s`를 누르면 저장 대화상자가 나타납니다. `Tab`으로 두 위치를 전환합니다.

| 저장 위치 | 설명 |
| --- | --- |
| `.claude/workflows/` (프로젝트) | 리포지토리를 클론하는 모든 사람과 공유 |
| `~/.claude/workflows/` (사용자) | 모든 프로젝트에서 사용 가능, 본인만 볼 수 있음 |

프로젝트 워크플로우와 개인 워크플로우가同名이면 프로젝트 워크플로우가 우선 실행됩니다.

### 인수 전달

저장된 워크플로우는 `args` 파라미터로 입력을 받을 수 있습니다. 스크립트 내에서 `args`라는 이름의 전역 변수로 접근합니다.

```
> Run /triage-issues on issues 1024, 1025, and 1030
```

`args`가 생략되면 스크립트 내에서 `undefined`입니다.

### 런타임 동작과 제한

| 제약 | 이유 |
| --- | --- |
| 실행 중 사용자 입력 불가 | 에이전트 권한 프롬프트만 실행을 일시정지 가능. 단계 간 승인이 필요하면 각 단계를 별도 워크플로우로 실행 |
| 워크플로우 자체의 직접 파일시스템/셸 접근 불가 | 에이전트가 읽고 쓰고 실행. 스크립트는 에이전트를 조정 |
| 최대 16개 동시 에이전트 (CPU 코어에 따라 감소) | 로컬 리소스 제한 |
| 실행당 최대 1,000개 에이전트 | 무한 루프 방지 |

- 모든 실행 스크립트는 `~/.claude/projects/` 하위 세션 디렉터리에 파일로 저장됨
- 중단 후 같은 세션 내에서 재개 가능 (완료된 에이전트는 캐시된 결과 반환)
- 세션을 종료하면 다음 세션에서 새로 시작

### 비용 관리

워크플로우는 많은 에이전트를 생성하므로 대화로 작업할 때보다 훨씬 많은 토큰을 소모합니다. 대규모 태스크 전에 작은 단위로 먼저 실행하여 비용을 가늠하십시오.

- `/model`로 대규모 실행 전 모델 확인
- 스크립트에서 단계별로 다른 모델 지정 가능

### 워크플로우 비활성화

| 방법 | 범위 |
| --- | --- |
| `/config`에서 Dynamic workflows 끄기 | 세션 간 유지 |
| `~/.claude/settings.json`에 `"disableWorkflows": true` 설정 | 세션 간 유지 |
| `CLAUDE_CODE_DISABLE_WORKFLOWS=1` 환경변수 | 시작 시 읽음 |
| 관리 설정에서 `"disableWorkflows": true` | 조직 전체 |

비활성화 시 번들 워크플로우 명령어 사용 불가, `ultracode` 키워드 트리거 안 됨, `/effort` 메뉴에서 ultracode 제거.

---

## Agent Teams

Agent Teams는 여러 Claude Code 인스턴스가 팀으로 협업하는 기능입니다. 한 세션이 팀 리드로서 작업을 조정하고, 팀원은 각자의 컨텍스트 윈도우에서 독립적으로 작업하며 서로 직접 소통합니다.

### 활성화

기본적으로 비활성화. 환경변수 또는 `settings.json`에서 활성화합니다.

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 아키텍처

| 구성 요소 | 역할 |
| --- | --- |
| **Team lead** | 팀을 생성하고 팀원을 생성하며 작업을 조정하는 메인 Claude Code 세션 |
| **Teammates** | 할당된 태스크에서 작업하는 개별 Claude Code 인스턴스 |
| **Task list** | 팀원이 할당받고 완료하는 공유 작업 항목 목록 |
| **Mailbox** | 에이전트 간 통신 메시징 시스템 |

저장 위치:

- 팀 설정: `~/.claude/teams/{team-name}/config.json`
- 태스크 목록: `~/.claude/tasks/{team-name}/`

### 디스플레이 모드

| 모드 | 설명 | 요구사항 |
| --- | --- | --- |
| **In-process** | 모든 팀원이 메인 터미널에서 실행. `Shift+Down`으로 팀원 순환 | 없음 |
| **Split panes** | 각 팀원이 별도의 창. 모든 출력을 동시에 볼 수 있음 | tmux 또는 iTerm2 |

기본값은 `"auto"` (tmux 세션 안이면 split panes, 아니면 in-process). 설정 변경:

```json
{
  "teammateMode": "in-process"
}
```

또는 세션별 플래그:

```bash
claude --teammate-mode in-process
```

### 팀원 및 모델 지정

```
Create a team with 4 teammates to refactor these modules in parallel.
Use Sonnet for each teammate.
```

팀원은 기본적으로 리드의 `/model` 설정을 상속하지 않습니다. `/config`에서 **Default teammate model**을 변경하거나, **Default (leader's model)**을 선택하여 리드의 현재 모델을 따르게 할 수 있습니다.

### Subagent 정의를 팀원으로 사용

미리 정의한 subagent 타입을 팀원으로 참조할 수 있습니다.

```
Spawn a teammate using the security-reviewer agent type to audit the auth module.
```

팀원은 해당 정의의 `tools` 허용 리스트와 `model`을 따르며, 정의의 본문은 시스템 프롬프트에 추가됩니다. `SendMessage` 및 태스크 관리 도구는 `tools` 제한에 관계없이 항상 사용 가능합니다.

### 태스크 할당 및 관리

태스크 상태: pending, in progress, completed. 태스크 간 의존성 설정 가능.

- **리드가 할당**: 리드에게 어떤 태스크를 누구에게 줄지 지시
- **자동 할당**: 팀원이 태스크를 완료하면 다음 미할당, 미차단 태스크를 자동으로 가져감
- 파일 잠금으로 동시 할당 경쟁 방지

### 품질 게이트 (Hooks)

| Hook | 실행 시점 | 종료 코드 2의 효과 |
| --- | --- | --- |
| `TeammateIdle` | 팀원이 유휴 상태로 전환 직전 | 피드백 전송, 팀원 계속 작업 |
| `TaskCreated` | 태스크 생성 중 | 생성 방지, 피드백 전송 |
| `TaskCompleted` | 태스크 완료 표시 중 | 완료 방지, 피드백 전송 |

### 계획 승인 요구

복잡하거나 위험한 태스크에서 팀원이 구현 전에 계획을 승인받도록 요구할 수 있습니다.

```
Spawn an architect teammate to refactor the authentication module.
Require plan approval before they make any changes.
```

### 컨텍스트 및 통신

- 각 팀원은 고유한 컨텍스트 윈도우를 가짐
- 리드의 대화 기록은 상속되지 않음
- CLAUDE.md, MCP 서버, 스킬은 자동 로드
- 메시지 자동 전달, 유휴 알림 자동 전송

### 제한사항

- `/resume` 및 `/rewind`로 in-process 팀원 복원 불가
- 태스크 상태가 지연될 수 있음
- 종료가 느릴 수 있음
- 리드당 한 팀만 관리 가능
- 중첩 팀 불가 (팀원이 자신의 팀을 생성할 수 없음)
- 리드 고정 (팀원을 리드로 승격 불가)
- 모든 팀원은 리드의 권한 모드로 시작
- Split pane 모드는 VS Code 통합 터미널, Windows Terminal, Ghostty에서 미지원

---

## Agent View

Agent View는 `claude agents` 명령어로 여는 모든 백그라운드 세션의 관리 화면입니다. Claude Code v2.1.139 이상 필요.

### 빠른 시작

| 단계 | 동작 |
| --- | --- |
| 1 | `claude agents` 실행 |
| 2 | 프롬프트 입력 후 `Enter`로 세션 디스패치 |
| 3 | `Space`로 피크 패널 열어서 최근 출력 확인 및 답변 |
| 4 | `Enter` 또는 `→`로 전체 대화에 첨부, `←`로 분리 |
| 5 | 기존 세션에서 `/bg` 또는 `←`로 백그라운드 전환 |

### 세션 상태

| 상태 | 아이콘 표시 | 의미 |
| --- | --- | --- |
| Working | 애니메이션 | Claude가 도구를 실행하거나 응답 생성 중 |
| Needs input | 노란색 | Claude가 질문이나 권한 결정 대기 |
| Idle | 흐림 | 다음 프롬프트 대기 중 |
| Completed | 초록색 | 태스크 성공 완료 |
| Failed | 빨간색 | 오류로 종료 |
| Stopped | 회색 | `Ctrl+X` 또는 `claude stop`으로 정지 |

아이콘 모양으로 프로세스 상태 구분:

| 모양 | 의미 |
| --- | --- |
| `✻` 또는 애니메이션 `✽` | 세션 프로세스 활성, 즉시 응답 |
| `∙` | 프로세스 종료됨. 피크, 답변, 첨부 가능하며 Claude가 이어서 재시작 |
| `✢` | `/loop` 세션이 반복 사이에 대기 중 |

### 키보드 단축키

| 단축키 | 동작 |
| --- | --- |
| `↑` / `↓` | 행 간 이동 |
| `Enter` | 선택한 세션에 첨부, 또는 입력이 있으면 디스패치 |
| `Space` | 피크 패널 열기/닫기 |
| `Shift+Enter` | 디스패치 후 즉시 첨부 |
| `→` | 선택한 세션에 첨부 |
| `Alt+1`..`Alt+9` | 해당 디렉터리의 1~9번 세션에 첨부 |
| `Tab` | 빈 입력에서 subagent 탐색, 그 외 제안 적용 |
| `Ctrl+S` | 상태/디렉터리 그룹핑 전환 |
| `Ctrl+T` | 세션 고정/해제 |
| `Ctrl+R` | 세션 이름 변경 |
| `Ctrl+G` | 에디터에서 디스패치 프롬프트 작성 |
| `Ctrl+X` | 세션 정지, 2초 내 다시 누르면 삭제 |
| `Shift+↑` / `Shift+↓` | 세션 순서 변경 |
| `Esc` | 피크 패널 닫기, 입력 지우기, 종료 |
| `?` | 모든 단축키 표시 |

### 세션 디스패치

| 입력 | 효과 |
| --- | --- |
| `<agent-name> <prompt>` | 첫 단어가 subagent 이름과 일치하면 해당 subagent로 실행 |
| `@<agent-name>` | 프롬프트 내에서 subagent 지정 |
| `@<repo>` | 하위 리포지토리에서 세션 실행 |
| `/<command>` | 스킬이나 명령어를 프롬프트로 디스패치 |
| `! <command>` | Claude 세션 대신 셸 명령을 백그라운드 잡으로 실행 |
| `#<number>` 또는 PR URL | 해당 PR을 작업 중인 세션 선택 |

셸에서 직접 디스패치:

```bash
claude --bg "investigate the flaky SettingsChangeDetector test"
claude --agent code-reviewer --bg "address review comments on PR 1234"
claude --bg --name "flaky-test-fix" "investigate the flaky test"
```

### 파일 편집 격리

모든 백그라운드 세션은 파일 편집 전 `.claude/worktrees/` 하위의 격리된 git worktree로 이동합니다.

Worktree 생략 조건:
- 이미 연결된 git worktree 안에 있는 경우
- 작업 디렉터리가 git 리포지토리가 아닌 경우
- 쓰기가 작업 디렉터리 밖인 경우

Worktree 격리 끄기 (v2.1.143+):

```json
{
  "worktree": {
    "bgIsolation": "none"
  }
}
```

### 백그라운드 세션 관리 (셸 명령)

| 명령어 | 용도 |
| --- | --- |
| `claude agents` | Agent View 열기 |
| `claude agents --cwd <path>` | 특정 디렉터리로 범위 제한 |
| `claude agents --json` | 실행 중인 세션을 JSON 배열로 출력 |
| `claude attach <id>` | 세션에 첨부 |
| `claude logs <id>` | 최근 출력 표시 |
| `claude stop <id>` | 세션 정지 |
| `claude respawn <id>` | 대화를 유지한 채 세션 재시작 |
| `claude respawn --all` | 모든 실행 중인 세션 재시작 |
| `claude rm <id>` | 세션 제거 |
| `claude daemon status` | 감독자 프로세스 상태 확인 |
| `claude daemon stop --any` | 감독자 및 백그라운드 세션 정지 |

### 상태 저장 위치

| 경로 | 내용 |
| --- | --- |
| `~/.claude/daemon.log` | 감독자 로그 |
| `~/.claude/daemon/roster.json` | 실행 중인 백그라운드 세션 목록 |
| `~/.claude/jobs/<id>/state.json` | 세션별 상태 |
| `~/.claude/jobs/<id>/tmp/` | 세션별 임시 디렉터리 |

### Agent View 비활성화

`disableAgentView` 설정을 `true`로 설정하거나 `CLAUDE_CODE_DISABLE_AGENT_VIEW` 환경변수를 설정합니다.

---

## /goal 명령어

`/goal` 명령어로 완료 조건을 설정하면 Claude가 조건이 충족될 때까지 여러 턴에 걸쳐 계속 작업합니다.

- `code.claude.com/docs/en/goal`의 메타데이터에 따르면: "Set a completion condition with `/goal` and Claude keeps working across turns until the condition is met."
- 탐색 메뉴의 Goals 섹션(Automation 카테고리)에 위치

---

## 실행 중인 작업 확인

| 접근 방식 | 확인 명령 |
| --- | --- |
| 백그라운드 세션 | `claude agents` (Agent View) |
| 현재 세션의 subagent | `/agents` (Running 탭 + Library 탭) |
| 현재 세션의 백그라운드 작업 | `/tasks` |
| Dynamic Workflows | `/workflows` |
