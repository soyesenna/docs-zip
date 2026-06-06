# 15. 일반 워크플로우 (Common Workflows)

> **원문**: [Common workflows](https://code.claude.com/docs/en/common-workflows) | [Best practices](https://code.claude.com/docs/en/best-practices) | [Large codebases](https://code.claude.com/docs/en/large-codebases)
>
> **기존 문서**: [Common workflows - Anthropic](https://docs.anthropic.com/en/docs/claude-code/common-workflows)

---

## 목차

- [새 코드베이스 이해하기](#새-코드베이스-이해하기)
- [버그 효율적으로 수정하기](#버그-효율적으로-수정하기)
- [코드 리팩토링](#코드-리팩토링)
- [Plan Mode로 안전한 코드 분석](#plan-mode로-안전한-코드-분석)
- [테스트 작업](#테스트-작업)
- [풀 리퀘스트 생성](#풀-리퀘스트-생성)
- [문서화 작업](#문서화-작업)
- [노트 및 비코드 폴더에서 작업하기](#노트-및-비코드-폴더에서-작업하기)
- [이미지 작업](#이미지-작업)
- [파일 및 디렉토리 참조](#파일-및-디렉토리-참조)
- [스케줄에 따라 Claude 실행하기](#스케줄에-따라-claude-실행하기)
- [Claude에게 기능 질문하기](#claude에게-기능-질문하기)
- [이전 대화 재개](#이전-대화-재개)
- [Git Worktree로 병렬 세션 실행](#git-worktree로-병렬-세션-실행)
- [편집 전에 계획하기](#편집-전에-계획하기)
- [서브에이전트에 연구 위임하기](#서브에이전트에-연구-위임하기)
- [스크립트에 Claude 파이프하기 (Non-interactive Mode)](#스크립트에-claude-파이프하기-non-interactive-mode)
- [베스트 프랙티스](#베스트-프랙티스)
- [효과적으로 소통하기](#효과적으로-소통하기)
- [커스텀 슬래시 명령어 (Skills)](#커스텀-슬래시-명령어-skills)
- [대화 관리](#대화-관리)
- [모노레포 및 대규모 코드베이스](#모노레포-및-대규모-코드베이스)

---

## 새 코드베이스 이해하기

### 빠른 코드베이스 개요

```bash
cd /path/to/project
claude
```

```
> 이 코드베이스에 대해 개요를 알려주세요
```

```
> 여기서 사용된 주요 아키텍처 패턴을 설명해주세요
```

```
> 핵심 데이터 모델은 무엇인가요?
```

```
> 인증은 어떻게 처리되나요?
```

**팁**:
- 넓은 질문으로 시작한 뒤 특정 영역으로 좁히기
- 프로젝트 코딩 컨벤션과 패턴에 대해 질문
- 프로젝트 전문 용어 사전 요청

### 관련 코드 찾기

```
> 사용자 인증을 처리하는 파일을 찾아주세요
```

```
> 이 인증 파일들이 어떻게 함께 작동하나요?
```

```
> 프론트엔드에서 데이터베이스까지의 로그인 프로세스를 추적해주세요
```

---

## 버그 효율적으로 수정하기

```
> npm test를 실행하면 에러가 나와요
```

```
> user.ts의 @ts-ignore를 수정할 몇 가지 방법을 제안해주세요
```

```
> user.ts에 제안하신 null 체크를 추가해주세요
```

**팁**:
- 문제 재현 명령과 스택 트레이스 전달
- 재현 단계 명시
- 에러가 간헐적인지 일관적인지 언급

---

## 코드 리팩토링

```
> 코드베이스에서 deprecated API 사용을 찾아주세요
```

```
> utils.js를 최신 JavaScript 기능으로 리팩토링하는 방법을 제안해주세요
```

```
> utils.js를 ES2024 기능으로 리팩토링하되 동일한 동작을 유지해주세요
```

```
> 리팩토링된 코드의 테스트를 실행해주세요
```

**팁**:
- 최신 방식의 이점 설명 요청
- 필요시 하위 호환성 유지 요청
- 작고 테스트 가능한 단위로 리팩토링

---

## Plan Mode로 안전한 코드 분석

### Plan Mode 사용 시기

| 상황 | 설명 |
|------|------|
| **다단계 구현** | 여러 파일을 편집해야 하는 기능 |
| **코드 탐색** | 변경 전 코드베이스 철저히 조사 |
| **대화형 개발** | Claude와 방향을 반복적으로 조정 |

### Plan Mode 활성화

**세션 중 전환**: `Shift+Tab`으로 권한 모드 순환

**새 세션에서 시작**:
```bash
claude --permission-mode plan
```

**기본 설정** (`.claude/settings.json`):
```json
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

---

## 테스트 작업

```
> NotificationsService.swift에서 테스트되지 않은 함수를 찾아주세요
```

```
> 알림 서비스에 대한 테스트를 추가해주세요
```

```
> 알림 서비스의 엣지 케이스에 대한 테스트 케이스를 추가해주세요
```

```
> 새 테스트를 실행하고 실패한 것을 수정해주세요
```

---

## 풀 리퀘스트 생성

```
> 인증 모듈에 대한 변경사항을 요약해주세요
```

```
> PR을 생성해주세요
```

```
> PR 설명에 보안 개선에 대한 컨텍스트를 추가해주세요
```

---

## 문서화 작업

```
> auth 모듈에서 적절한 JSDoc 주석이 없는 함수를 찾아주세요
```

```
> auth.js의 문서화되지 않은 함수에 JSDoc 주석을 추가해주세요
```

---

## 노트 및 비코드 폴더에서 작업하기

Claude Code는 모든 디렉토리에서 동작합니다. 노트 보관함, 문서 폴더, 마크다운 파일 모음 등에서도 검색, 편집, 재구성이 가능합니다. `.claude/`와 `CLAUDE.md`는 다른 도구 설정과 충돌 없이 공존하며, Claude는 매 tool 호출 시 파일을 새로 읽습니다.

---

## 이미지 작업

이미지를 대화에 추가하는 방법:
1. Claude Code 창에 이미지 드래그 앤 드롭
2. 이미지를 복사하고 `ctrl+v`로 붙여넣기 (`cmd+v` 아님)
3. 경로 제공: "/path/to/image.png를 분석해주세요"

```
> 이 이미지는 무엇을 보여주나요?
```

```
> 이 디자인 목업에 맞는 CSS를 생성해주세요
```

**UI 변경 시 이미지를 검증에 활용**:

```
> [스크린샷 붙여넣기] 이 디자인을 구현해주세요.
> 결과의 스크린샷을 찍고 원본과 비교해주세요.
> 차이점을 나열하고 수정해주세요.
```

이미지는 설계 참조, 스크린샷 비교, 시각적 버그 리포트 등에 활용할 수 있습니다.

---

## 파일 및 디렉토리 참조

`@` 기호로 파일이나 디렉토리를 빠르게 참조:

```
> @src/utils/auth.js의 로직을 설명해주세요
```

```
> @src/components의 구조는 어떻게 되나요?
```

```
> @github:repos/owner/repo/issues의 데이터를 보여주세요
```

---

## 스케줄에 따라 Claude 실행하기

매일 아침 열린 PR을 검토하거나, 주간으로 종속성을 감사하거나, 밤새 CI 실패를 확인하는 등 Claude가 반복 작업을 자동으로 처리하게 할 수 있습니다.

작업 실행 위치에 따라 스케줄링 옵션을 선택하세요:

| 옵션 | 실행 위치 | 적합한 경우 |
|------|-----------|-------------|
| **Routines** | Anthropic 관리 인프라 | 컴퓨터가 꺼져 있을 때도 실행해야 하는 작업. API 호출이나 GitHub 이벤트에도 트리거 가능. claude.ai/code/routines에서 구성. |
| **Desktop scheduled tasks** | 내 컴퓨터 (데스크톱 앱) | 로컬 파일, 도구, 커밋되지 않은 변경사항에 직접 접근이 필요한 작업. |
| **GitHub Actions** | CI 파이프라인 | PR 오픈 등 리포지토리 이벤트에 연결되거나, 워크플로우 설정과 함께 cron 스케줄로 실행해야 하는 작업. |
| **`/loop`** | 현재 CLI 세션 | 세션이 열려 있는 동안 빠르게 폴링. 새 대화를 시작하면 작업이 중지되며, `--resume`과 `--continue`로 만료되지 않은 루프를 복원 가능. |

---

## 이전 대화 재개

작업이 여러 번에 걸쳐 진행될 때, 컨텍스트를 다시 설명할 필요 없이 이전 상태부터 이어서 작업할 수 있습니다. Claude Code는 모든 대화를 로컬에 저장합니다.

```bash
# 현재 디렉토리에서 가장 최근 대화 재개
claude --continue
```

대화가 아직 없으면 `No conversation found to continue`가 출력되고 종료됩니다. `claude --resume`으로 목록에서 선택하거나, 실행 중인 세션에서 `/resume`을 사용하세요.

---

## Git Worktree로 병렬 세션 실행

한 터미널에서는 기능을 개발하고 다른 터미널에서는 Claude가 버그를 수정하면서 편집이 충돌하지 않도록 할 수 있습니다. 각 worktree는 자체 브랜치의 별도 체크아웃입니다.

```bash
claude --worktree feature-auth
```

두 번째 터미널에서 다른 이름으로 같은 명령을 실행하면 격리된 병렬 세션이 시작됩니다. 정리, `.worktreeinclude`, 비-git VCS 지원은 Worktrees를 참고하세요. 별도의 터미널 대신 한 화면에서 병렬 세션을 모니터링하려면 background agents를 참고하세요.

### 병렬 접근 방식 선택

원하는 조정 수준에 따라 병렬 방식을 선택하세요:

| 방식 | 설명 |
|------|------|
| **Worktrees** | 격리된 git checkout에서 별도 CLI 세션을 실행하여 편집 충돌 방지 |
| **Desktop app** | 여러 로컬 세션을 시각적으로 관리, 각각 자체 worktree에서 실행 |
| **Claude Code on the web** | Anthropic 관리 클라우드 인프라에서 격리된 VM으로 세션 실행 |
| **Agent teams** | 공유 작업, 메시징, 팀 리드로 여러 세션을 자동 조정 |

### Writer/Reviewer 패턴

새로운 컨텍스트에서 리뷰하면 Claude가 자신이 작성한 코드에 편향되지 않아 코드 리뷰 품질이 향상됩니다.

| 세션 A (Writer) | 세션 B (Reviewer) |
|-----------------|-------------------|
| `API 엔드포인트에 rate limiter를 구현해주세요` | |
| | `@src/middleware/rateLimiter.ts를 리뷰해주세요. 엣지 케이스, race condition, 기존 패턴 일관성 확인` |
| `리뷰 피드백: [세션 B 출력]. 이 문제들을 해결해주세요` | |

---

## 편집 전에 계획하기

디스크에 변경사항이 반영되기 전에 검토하려면 plan mode로 전환하세요. Claude는 파일을 읽고 계획을 제안하지만, 승인하기 전까지는 편집하지 않습니다.

```bash
claude --permission-mode plan
```

세션 중에 `Shift+Tab`을 눌러 plan mode로 전환할 수도 있습니다. 승인 흐름과 텍스트 에디터에서 계획 편집에 대한 자세한 내용은 Plan mode를 참고하세요.

### 중간 계획(Scratchpad) 활용

복잡한 작업에서는 계획을 파일에 기록하면 Claude가 방향을 잃지 않습니다. Claude가 계획을 문서화하게 한 후, 그 계획에 따라 실행하게 하세요:

```
PLAN.md에 인증 시스템 마이그레이션 계획을 작성해주세요.
각 단계에 필요한 파일, 변경 내용, 검증 방법을 포함해주세요.
```

계획이 정리되면 새 세션에서 실행하세요. 새 세션은 구현에만 집중하는 깨끗한 컨텍스트를 가지며, 참조할 수 있는 문서화된 계획이 있습니다.

---

## 서브에이전트에 연구 위임하기

컨텍스트는 근본적인 제약사항입니다. 서브에이전트는 별도 컨텍스트 창에서 실행되어 요약만 보고하므로 메인 대화를 깔끔하게 유지합니다.

**조사에 서브에이전트 사용**:

```
서브에이전트를 사용해 인증 시스템이 token refresh를 어떻게 처리하는지 조사해주세요.
```

**검증에 서브에이전트 사용**:

```
서브에이전트를 사용해 이 코드의 엣지 케이스를 리뷰해주세요
```

**커스텀 서브에이전트 정의** (`.claude/agents/security-reviewer.md`):

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep, Glob, Bash
model: opus
---
You are a senior security engineer. Review code for:
- Injection vulnerabilities (SQL, XSS, command injection)
- Authentication and authorization flaws
- Secrets or credentials in code
Provide specific line references and suggested fixes.
```

커스텀 에이전트 정의에 대한 자세한 내용은 Subagents를 참고하세요.

---

## 스크립트에 Claude 파이프하기 (Non-interactive Mode)

CI, pre-commit hook, 배치 처리를 위해 비대화형으로 Claude를 실행합니다. Stdin과 stdout이 일반 Unix 도구처럼 동작합니다.

```bash
# 단발성 쿼리
claude -p "이 프로젝트가 무엇을 하는지 설명해주세요"

# 스크립트를 위한 구조화된 출력
claude -p "모든 API 엔드포인트를 나열해주세요" --output-format json

# 실시간 처리를 위한 스트리밍
claude -p "이 로그 파일을 분석해주세요" --output-format stream-json --verbose
```

**파이프 활용 예시**:

```bash
git log --oneline -20 | claude -p "summarize these recent commits"
```

**기존 데이터/처리 파이프라인에 통합**:

```bash
claude -p "<your prompt>" --output-format json | your_command
```

개발 중에는 `--verbose`를 사용하고 프로덕션에서는 끄세요.

### Auto mode로 자율 실행

중단 없는 실행과 백그라운드 안전 검사를 위해 auto mode를 사용하세요. classifier 모델이 명령을 실행 전에 검토하여 범위 확장, 알 수 없는 인프라, 적대적 콘텐츠 기반 액션은 차단하고 일상적인 작업은 프롬프트 없이 진행합니다.

```bash
claude --permission-mode auto -p "fix all lint errors"
```

`-p` 플래그를 사용한 비대화형 실행에서 auto mode는 classifier가 반복적으로 액션을 차단하면 중단됩니다. 사용자가 폴백할 수 없기 때문입니다. 자세한 내용은 auto mode fallback 임계값을 참고하세요.

출력 형식, 권한 플래그, fan-out 패턴에 대한 자세한 내용은 Non-interactive mode를 참고하세요.

---

## Claude에게 기능 질문하기

Claude는 자체 문서에 내장 액세스 권한이 있어 기능과 제한사항에 대해 답변할 수 있습니다.

```
> Claude Code는 풀 리퀘스트를 생성할 수 있나요?
```

```
> Claude Code는 권한을 어떻게 처리하나요?
```

```
> MCP를 Claude Code와 함께 사용하려면 어떻게 하나요?
```

```
> Claude Code를 Amazon Bedrock으로 구성하려면 어떻게 하나요?
```

> Claude는 항상 사용 중인 버전에 관계없이 최신 Claude Code 문서에 액세스할 수 있습니다.

---

## 베스트 프랙티스

### 탐색 먼저, 그다음 계획, 그다음 코드

Claude가 바로 코딩에 들어가면 잘못된 문제를 해결하는 코드가 나올 수 있습니다. plan mode를 사용해 탐색과 실행을 분리하세요.

**권장 4단계 워크플로우**:

| 단계 | 설명 |
|------|------|
| **1. 탐색(Explore)** | Claude가 코드베이스를 읽고 구조를 파악합니다 |
| **2. 계획(Plan)** | plan mode에서 변경 계획을 제안합니다. 디스크에 아무것도 쓰지 않습니다 |
| **3. 구현(Code)** | 승인 후 Claude가 실제 편집을 수행합니다 |
| **4. 검증(Verify)** | 테스트 실행, 빌드 확인, 결과 검증 |

```
> 이 코드베이스에서 인증이 어떻게 동작하는지 조사해주세요
```

```
> /plan 인증 시스템을 OAuth2로 마이그레이션하는 계획을 세워주세요
```

```
> 계획을 승인합니다. 구현해주세요
```

```
> 테스트를 실행하고 모든 것이 통과하는지 확인해주세요
```

### 프롬프트에 구체적인 컨텍스트 제공

Claude는 의도를 추론할 수 있지만, 마음을 읽을 수는 없습니다. 구체적인 파일, 제약사항, 패턴 예시를 참조하세요.

| 전략 | Before | After |
|------|--------|-------|
| **작업 범위 지정** — 파일, 시나리오, 테스트 선호도 명시 | _"foo.py에 테스트 추가해주세요"_ | _"foo.py에 로그아웃된 사용자 엣지 케이스 테스트를 작성해주세요. mock은 사용하지 마세요"_ |
| **소스 지정** — 질문에 답할 수 있는 소스를 Claude에게 제시 | _"ExecutionFactory의 API가 왜 이상한가요?"_ | _"ExecutionFactory의 git history를 살펴보고 API가 어떻게 형성되었는지 요약해주세요"_ |
| **기존 패턴 참조** — 코드베이스의 패턴을 Claude에게 제시 | _"캘린더 위젯 추가해주세요"_ | _"홈페이지의 기존 위젯 구현 패턴을 참고하세요. HotDogWidget.php가 좋은 예시입니다. 이 패턴을 따라 새 캘린더 위젯을 구현해주세요"_ |
| **증상 설명** — 증상, 위치, "수정 완료"의 기준 제공 | _"로그인 버그 수정해주세요"_ | _"세션 타임아웃 후 로그인이 실패한다는 보고가 있습니다. src/auth/의 인증 플로우를 확인해주세요. 특히 token refresh 부분입니다. 실패하는 테스트를 먼저 작성한 후 수정해주세요"_ |

모호한 프롬프트는 탐색 중이고 방향을 수정할 여유가 있을 때 유용할 수 있습니다. `"이 파일에서 개선할 점은?"` 같은 프롬프트는 생각하지 못했던 문제를 발견하는 데 도움이 됩니다.

### 풍부한 콘텐츠 제공

Claude에 다양한 방식으로 데이터를 제공할 수 있습니다:

| 방식 | 설명 |
|------|------|
| **`@`로 파일 참조** | 코드 위치를 설명하는 대신 `@src/auth.ts`처럼 직접 참조. Claude가 응답 전 파일을 읽습니다 |
| **이미지 직접 붙여넣기** | 프롬프트에 복사/붙여넣기 또는 드래그 앤 드롭 |
| **URL 제공** | 문서와 API 레퍼런스의 URL 전달. `/permissions`로 자주 사용하는 도메인을 allowlist에 추가 가능 |
| **데이터 파이프** | `cat error.log \| claude`로 파일 내용을 직접 전송 |
| **Claude가 직접 가져오게 하기** | Bash 명령, MCP 도구, 파일 읽기를 통해 Claude가 스스로 컨텍스트를 가져오도록 지시 |

### 효과적인 CLAUDE.md 작성하기

CLAUDE.md는 Claude가 매 대화 시작 시 읽는 특수 파일로, Bash 명령, 코드 스타일, 워크플로우 규칙을 포함하면 영구적 컨텍스트를 제공합니다. `/init` 명령으로 코드베이스를 분석하여 기반을 생성할 수 있습니다.

| 포함할 내용 | 제외할 내용 |
|-------------|-------------|
| Claude가 추측할 수 없는 Bash 명령 | 코드를 읽어서 파악 가능한 것 |
| 기본값과 다른 코드 스타일 규칙 | Claude가 이미 아는 표준 언어 규칙 |
| 테스트 지침 및 선호하는 테스트 러너 | 상세한 API 문서 (대신 링크 제공) |
| 리포지토리 에티켓 (브랜치 명명, PR 컨벤션) | 자주 변경되는 정보 |
| 프로젝트 특유의 아키텍처 결정 | 긴 설명이나 튜토리얼 |
| 개발 환경 특이사항 (필수 환경변수) | 파일별 코드베이스 설명 |
| 자주 겪는 문제나 비직관적 동작 | "깔끔한 코드 작성" 같은 자명한 지침 |

CLAUDE.md는 매 세션마다 로드되므로 광범위하게 적용되는 내용만 포함하세요. 도메인 지식이나 가끔만 관련 있는 워크플로우에는 skills를 대신 사용하세요. Claude는 필요할 때만 로드하므로 모든 대화를 부풀리지 않습니다.

간결하게 유지하세요. 각 줄에 대해 _"이것을 제거하면 Claude가 실수를 할까?"_라고 자문하세요. 그렇지 않다면 삭제하세요. CLAUDE.md가 너무 길면 Claude가 실제 지시를 무시하게 됩니다.

CLAUDE.md 파일은 `@path/to/import` 구문으로 추가 파일을 임포트할 수 있습니다:

```
See @README.md for project overview and @package.json for available npm commands.

# Additional Instructions
- Git workflow: @docs/git-instructions.md
- Personal overrides: @~/.claude/my-project-instructions.md
```

CLAUDE.md 파일은 여러 위치에 배치할 수 있습니다:

| 위치 | 설명 |
|------|------|
| 홈 폴더 (`~/.claude/CLAUDE.md`) | 모든 Claude 세션에 적용 |
| 프로젝트 루트 (`./CLAUDE.md`) | git에 커밋하여 팀과 공유 |
| 프로젝트 루트 (`./CLAUDE.local.md`) | 개인 프로젝트별 노트; `.gitignore`에 추가 |
| 상위 디렉토리 | 모노레포에서 `root/CLAUDE.md`와 `root/foo/CLAUDE.md`가 자동으로 로드 |
| 하위 디렉토리 | Claude가 해당 디렉토리의 파일을 읽을 때 온디맨드로 로드 |

### 권한 구성

기본적으로 Claude Code는 파일 쓰기, Bash 명령, MCP 도구 등 시스템을 수정할 수 있는 작업에 대해 권한을 요청합니다. 안전하지만 번거로울 수 있습니다. 세 가지 방법으로 중단을 줄일 수 있습니다:

- **Auto mode**: 별도의 classifier 모델이 명령을 검토하고 위험한 것만 차단합니다
- **권한 allowlist**: 안전하다고 알려진 특정 도구를 허용합니다 (예: `npm run lint`, `git commit`)
- **샌드박싱**: OS 수준 격리를 활성화하여 파일 시스템과 네트워크 접근을 제한합니다

### Hooks 설정

Hooks는 Claude의 워크플로우 특정 지점에서 스크립트를 자동으로 실행합니다. CLAUDE.md 지시는 권고 사항이지만, hooks는 결정론적이며 액션을 보장합니다.

Claude가 hooks를 작성하도록 할 수 있습니다. _"Write a hook that runs eslint after every file edit"_ 또는 _"Write a hook that blocks writes to the migrations folder"_ 같은 프롬프트를 시도해 보세요. 직접 `.claude/settings.json`을 편집하여 hooks를 구성하고, `/hooks`로 설정된 내용을 확인할 수 있습니다.

### 컨텍스트 관리

대부분의 베스트 프랙티스는 하나의 제약에서 비롯됩니다: **Claude의 컨텍스트 창이 빠르게 채워지며, 채워질수록 성능이 저하된다**는 점입니다. 컨텍스트 창은 전체 대화, 모든 파일 읽기, 모든 명령 출력을 포함합니다. 단일 디버깅 세션이나 코드베이스 탐색만으로도 수만 토큰을 생성하고 소비할 수 있습니다.

컨텍스트 사용량을 지속적으로 추적하려면 custom status line을 사용하세요. 자세한 컨텍스트 관리 전략은 [대화 관리](#대화-관리) 섹션을 참고하세요.

### 검증 가능한 결과 제공

Claude는 작업이 완료되어 보일 때 멈춥니다. pass 또는 fail을 반환하는 검증을 제공하면 루프가 자동으로 닫힙니다. 검증은 테스트 스위트, 빌드 종료 코드, 린터, 스크립트 비교, 브라우저 스크린샷 등 Claude가 읽을 수 있는 신호를 반환하는 어떤 것이든 됩니다.

| 전략 | Before | After |
|------|--------|-------|
| **검증 기준 제공** | _"이메일 주소를 검증하는 함수를 구현해주세요"_ | _"validateEmail 함수를 작성해주세요. 테스트 케이스: user@example.com은 true, invalid는 false, user@.com은 false. 구현 후 테스트를 실행해주세요"_ |
| **UI 변경 시각적 검증** | _"대시보드를 더 좋게 만들어주세요"_ | _"[스크린샷 붙여넣기] 이 디자인을 구현해주세요. 결과의 스크린샷을 찍고 원본과 비교해주세요. 차이점을 나열하고 수정해주세요"_ |
| **근본 원인 해결** | _"빌드가 실패해요"_ | _"빌드가 이 에러로 실패합니다: [에러 붙여넣기]. 수정하고 빌드가 성공하는지 확인해주세요. 에러를 억누르지 말고 근본 원인을 해결해주세요"_ |

검증이 준비되면, 얼마나 엄격하게 종료를 제어할지 결정하세요:

| 방식 | 설명 |
|------|------|
| **단일 프롬프트** | 같은 메시지에서 Claude가 검증을 실행하고 반복하도록 요청 |
| **세션 전반** | `/goal` 조건으로 설정. 별도 evaluator가 매 턴 후 재검사하며 조건이 충족될 때까지 Claude가 계속 작업 |
| **결정론적 게이트** | Stop hook이 스크립트로 검증을 실행하고 통과할 때까지 턴 종료를 차단. Claude Code는 8번 연속 차단 후 훅을 무시하고 턴을 종료 |
| **제2의 의견** | 검증 서브에이전트나 동적 워크플로우가 별도의 모델로 결과를 반박하므로, 작업을 수행한 에이전트가 자신의 결과를 평가하지 않음 |

Claude가 성공을 주장하는 대신 증거를 보여주도록 하세요: 테스트 출력, 실행한 명령과 반환값, 결과 스크린샷. 증거를 검토하는 것이 직접 검증을 다시 실행하는 것보다 빠르며, 사용자가 지켜보지 않았던 세션에도 적용됩니다.

### 적대적 리뷰(Adversarial Review) 추가

Claude가 무인으로 오래 작업할수록 독립적 검증이 중요해집니다. 새 서브에이전트 컨텍스트의 리뷰어는 diff와 기준만 보고 결과를 평가합니다.

```
서브에이전트를 사용해 rate limiter diff를 PLAN.md와 대조하여 리뷰해주세요.
모든 요구사항이 구현되었는지, 엣지 케이스에 테스트가 있는지,
범위 밖 변경이 없는지 확인해주세요.
```

### 일반적인 실패 패턴 피하기

| 패턴 | 해결 방법 |
|------|-----------|
| **모든 것을 넣는 세션** — 한 작업으로 시작했다 관련 없는 질문을 하고 다시 돌아옴 | 관련 없는 작업 사이에 `/clear` 사용 |
| **반복적 수정** — 고치고, 틀리고, 다시 고치고, 또 틀림 | 2회 이상 실패한 후 `/clear`하고 배운 점을 반영한 더 나은 프롬프트로 다시 시작 |
| **과도한 CLAUDE.md** — 너무 길면 중요한 규칙이 노이즈에 묻힘 | 무자비하게 가지치기. Claude가 이미 올바르게 하는 것은 삭제 |
| **신뢰 후 검증 갭** — 그럴듯해 보이지만 엣지 케이스를 처리하지 않는 구현 | 항상 검증(테스트, 스크립트, 스크린샷) 제공 |
| **무한 탐색** — 범위를 정하지 않고 "조사"를 요청하여 수백 개 파일을 읽음 | 탐색 범위를 좁히거나 subagent를 사용 |

---

## 효과적으로 소통하기

### 코드베이스 질문하기

새 코드베이스에 온보딩할 때 다른 엔지니어에게 물어볼 것과 같은 질문을 하세요:

- 로깅은 어떻게 동작하나요?
- 새 API 엔드포인트를 만들려면 어떻게 하나요?
- `foo.rs` 134번째 줄의 `async move { ... }`는 무슨 의미인가요?
- 이 코드는 333번째 줄에서 `bar()` 대신 `foo()`를 호출하는 이유가 무엇인가요?

특별한 프롬프트 없이 직접 질문하세요.

### Claude가 인터뷰하게 하기

Claude는 기술적 구현, UI/UX, 엣지 케이스, 트레이드오프 등 간과할 수 있는 부분에 대해 질문합니다.

```
[간단한 설명]을 구현하고 싶습니다. AskUserQuestion 도구를 사용해 자세히 인터뷰해주세요.
기술적 구현, UI/UX, 엣지 케이스, 트레이드오프에 대해 질문해주세요.
모든 것을 다룰 때까지 인터뷰를 계속한 후, 완전한 스펙을 SPEC.md에 작성해주세요.
```

스펙이 완성되면 새 세션에서 실행하세요. 깨끗한 컨텍스트로 구현에 집중할 수 있습니다.

### 코스 수정은 일찍, 자주

| 단축키/명령 | 설명 |
|-------------|------|
| `Esc` | Claude의 동작을 중간에 중지. 컨텍스트 보존으로 방향 전환 가능 |
| `Esc + Esc` 또는 `/rewind` | 이전 대화와 코드 상태 복원 또는 선택 메시지에서 요약 |
| `"Undo that"` | Claude가 변경을 되돌리도록 지시 |
| `/clear` | 관련 없는 작업 간 컨텍스트 초기화 |

같은 문제로 두 번 이상 수정했다면 `/clear` 후 배운 점을 반영한 프롬프트로 새로 시작하세요.

---

## 커스텀 슬래시 명령어 (Skills)

Skills로 프로젝트 특화 지식을 Claude에 확장하세요. `/skill-name`으로 직접 호출하거나 Claude가 관련 시점에 자동 적용합니다.

**지식 skill 예시** (`.claude/skills/api-conventions/SKILL.md`):

```markdown
---
name: api-conventions
description: REST API design conventions for our services
---
# API Conventions
- URL 경로에는 kebab-case 사용
- JSON 속성에는 camelCase 사용
- 목록 엔드포인트에는 항상 pagination 포함
```

**반복 워크플로우 skill 예시** (`.claude/skills/fix-issue/SKILL.md`):

```markdown
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---
GitHub 이슈를 분석하고 수정합니다: $ARGUMENTS.
1. `gh issue view`로 이슈 세부사항 가져오기
2. 코드베이스에서 관련 파일 검색
3. 변경사항 구현 및 테스트 작성/실행
4. Push 및 PR 생성
```

`/fix-issue 1234`로 실행하세요. 수동 트리거만 원하면 `disable-model-invocation: true`를 사용하세요.

---

## 대화 관리

### Checkpoint로 되돌리기

Claude는 각 변경 전에 파일을 자동으로 스냅샷합니다. `Esc + Esc` 또는 `/rewind`로 rewind 메뉴를 열어 대화/코드 복원 또는 요약을 선택할 수 있습니다. Checkpoint는 세션 간에 지속되므로 터미널을 닫아도 나중에 되돌릴 수 있습니다.

### 컨텍스트 적극적으로 관리

Claude Code는 컨텍스트 한계에 가까워지면 대화 기록을 자동으로 압축합니다.

- 작업 간에 `/clear`를 자주 사용하여 컨텍스트 초기화
- 더 세밀한 제어를 위해 `/compact <instructions>` 실행 (예: `/compact Focus on the API changes`)
- `Esc + Esc` 또는 `/rewind`로 checkpoint를 선택한 후 **Summarize from here** 또는 **Summarize up to here**로 부분 압축
- CLAUDE.md에 `"When compacting, always preserve the full list of modified files and any test commands"` 같은 지시를 추가하여 중요 컨텍스트 보존
- 빠른 질문에는 `/btw`를 사용 — 답변이 대화 기록에 들어가지 않음

---

## 모노레포 및 대규모 코드베이스

대규모 코드베이스에서는 작은 프로젝트용 기본 설정이 컨텍스트 창을 관련 없는 지시와 파일 읽기로 채울 수 있습니다. 자세한 내용은 [Large codebases 가이드](https://code.claude.com/docs/en/large-codebases)를 참고하세요. 아래는 핵심 설정 요약입니다.

### 설정 옵션 요약

| 목적 | 사용 도구 |
|------|-----------|
| 작업 중인 코드의 컨벤션만 로드 | 디렉토리별 CLAUDE.md 파일 |
| 작업하지 않는 패키지의 CLAUDE.md 제외 | `claudeMdExcludes` |
| 빌드 출력, 생성 코드, vendor 종속성 읽기 차단 | `permissions.deny`의 `Read` 거부 규칙 |
| 파일 스캔 대신 언어 서버로 심볼 탐색 | Code intelligence plugin |
| worktree에서 작업에 필요한 디렉토리만 체크아웃 | `worktree.sparsePaths` |
| 형제 패키지나 다른 리포지토리에 접근 | `--add-dir` 또는 `additionalDirectories` |
| 특정 영역에만 관련된 절차를 온디맨드로 로드 | 디렉토리별 skills |

### 중첩 CLAUDE.md 파일

대규모 코드베이스에서는 루트 CLAUDE.md가 비대해지거나 너무 일반적이 됩니다. 디렉토리별로 지시를 분리하면 전체 규칙과 현재 작업 중인 코드의 컨벤션만 로드됩니다.

```
monorepo/
  CLAUDE.md                     # 루트 지시 (코딩 표준, 커밋 컨벤션)
  packages/
    api/
      CLAUDE.md                 # API 전용 지시
    web/
      CLAUDE.md                 # 프론트엔드 전용 지시
    shared/
      CLAUDE.md                 # 공유 라이브러리 지시
```

### Sparse Worktrees

`--worktree` 플래그는 새 git worktree에서 세션을 시작하여 변경사항이 메인 체크아웃과 격리됩니다. 기본적으로 전체 리포지토리를 체크아웃하지만, 대규모 리포지토리에서는 `worktree.sparsePaths` 설정으로 지정된 디렉토리와 루트 수준 파일만 디스크에 기록합니다.

```json
{
  "worktree": {
    "sparsePaths": [
      ".claude",
      "packages/api",
      "packages/shared"
    ],
    "symlinkDirectories": [
      "node_modules"
    ]
  }
}
```

`node_modules` 같은 대형 디렉토리가 worktree 간에 중복되는 것을 피하려면 `symlinkDirectories`를 함께 사용하세요. 각 worktree의 `node_modules/`가 메인 리포지토리의 복사본으로 심볼릭 링크됩니다.

### 패키지 간 접근 권한 부여

하위 디렉토리에서 Claude를 시작할 때, 작업이 여러 패키지에 걸쳐 있으면 형제 디렉토리에 대한 접근 권한이 필요합니다.

```json
{
  "permissions": {
    "additionalDirectories": [
      "../shared",
      "../web"
    ]
  }
}
```

또는 런타임에 `--add-dir` 플래그를 사용할 수도 있습니다:

```bash
claude --add-dir ../shared
```

### 읽기 제한 설정

`.gitignore`에 이미 나열된 경로(`node_modules/`, `dist/`, `build/`)는 기본적으로 검색에서 제외됩니다. 체크인된 경로(예: vendor SDK, 커밋된 생성 코드)의 경우 `permissions.deny`에 `Read` 거부 규칙을 추가합니다:

```json
{
  "permissions": {
    "deny": [
      "Read(./**/dist/**)",
      "Read(./**/build/**)",
      "Read(./**/*.generated.*)",
      "Read(./vendor/**)"
    ]
  }
}
```

---

## 다음 단계

| 주제 | 설명 | 문서 |
|------|------|------|
| Claude Code 작동 방식 | 에이전트 루프, 도구, 컨텍스트 관리 | [How Claude Code works (원문)](https://code.claude.com/docs/en/how-it-works) |
| Claude Code 확장 | skills, hooks, MCP, 서브에이전트, 플러그인 | [Extend Claude Code (원문)](https://code.claude.com/docs/en/extensions) |
| 서브에이전트 | 특화된 AI 어시스턴트 만들기 | [서브에이전트](08-subagents.md) |
| GitHub Actions | CI 자동화 | [GitHub Actions](12-github-actions.md) |
| SDK | 프로그래밍 통합 | [SDK](11-sdk.md) |
| 대규모 코드베이스 | 모노레포 및 대규모 리포지토리 설정 | [Large codebases (원문)](https://code.claude.com/docs/en/large-codebases) |
