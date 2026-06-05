# 15. 일반 워크플로우 (Common Workflows)

> **참조**: [Common workflows - Anthropic](https://docs.anthropic.com/en/docs/claude-code/common-workflows)

---

## 목차

- [새 코드베이스 이해하기](#새-코드베이스-이해하기)
- [버그 효율적으로 수정하기](#버그-효율적으로-수정하기)
- [코드 리팩토링](#코드-리팩토링)
- [전문 서브에이전트 사용](#전문-서브에이전트-사용)
- [Plan Mode로 안전한 코드 분석](#plan-mode로-안전한-코드-분석)
- [테스트 작업](#테스트-작업)
- [풀 리퀘스트 생성](#풀-리퀘스트-생성)
- [문서화 작업](#문서화-작업)
- [이미지 작업](#이미지-작업)
- [파일 및 디렉토리 참조](#파일-및-디렉토리-참조)
- [확장된 사고(Extended Thinking) 사용](#확장된-사고extended-thinking-사용)
- [이전 대화 재개](#이전-대화-재개)
- [Git Worktree로 병렬 세션 실행](#git-worktree로-병렬-세션-실행)
- [Unix 유틸리티로 Claude 활용](#unix-유틸리티로-claude-활용)
- [커스텀 슬래시 명령어 만들기](#커스텀-슬래시-명령어-만들기)
- [Claude에게 기능 질문하기](#claude에게-기능-질문하기)

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

## 전문 서브에이전트 사용

### 사용 가능한 서브에이전트 보기

```
> /agents
```

### 자동 위임

Claude Code가 적절한 서브에이전트에 자동으로 작업을 위임합니다:

```
> 최근 코드 변경사항을 보안 문제로 리뷰해주세요
```

```
> 모든 테스트를 실행하고 실패한 것을 수정해주세요
```

### 명시적 서브에이전트 요청

```
> code-reviewer 서브에이전트로 auth 모듈을 확인해주세요
```

```
> debugger 서브에이전트로 사용자가 로그인할 수 없는 문제를 조사해주세요
```

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

**헤드리스 쿼리**:
```bash
claude --permission-mode plan -p "인증 시스템을 분석하고 개선점을 제안해주세요"
```

### Plan Mode를 기본으로 설정

```json
// .claude/settings.json
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

## 확장된 사고(Extended Thinking) 사용

복잡한 아키텍처 결정, 어려운 버그, 다단계 구현 계획에 유용합니다.

```
> OAuth2를 사용한 새 인증 시스템을 구현해야 합니다. 우리 코드베이스에 가장 적합한 접근 방식에 대해 깊이 생각해주세요.
```

```
> 이 접근 방식의 잠재적 보안 취약점에 대해 생각해주세요
```

**사고 깊이 조절**:
- "think" → 기본 확장 사고
- "keep thinking", "think more", "think longer" → 더 깊은 사고

---

## 이전 대화 재개

```bash
# 가장 최근 대화 재개
claude --continue

# 비대화형 모드로 재개
claude --continue --print "작업을 계속해주세요"

# 대화 선택기 표시
claude --resume
```

---

## Git Worktree로 병렬 세션 실행

```bash
# 새 worktree 생성
git worktree add ../project-feature-a -b feature-a

# worktree에서 Claude Code 실행
cd ../project-feature-a
claude

# 다른 worktree에서도 독립 실행
cd ../project-bugfix
claude

# worktree 관리
git worktree list
git worktree remove ../project-feature-a
```

---

## Unix 유틸리티로 Claude 활용

### 검증 프로세스에 Claude 추가

```json
// package.json
{
  "scripts": {
    "lint:claude": "claude -p '당신은 린터입니다. main과의 변경사항을 확인하고 오타와 관련된 문제를 보고해주세요.'"
  }
}
```

### 파이프 인/아웃

```bash
cat build-error.txt | claude -p '이 빌드 에러의 근본 원인을 간결하게 설명해주세요' > output.txt
```

### 출력 형식 제어

```bash
# 텍스트 (기본값)
cat data.txt | claude -p '이 데이터를 요약해주세요' --output-format text > summary.txt

# JSON
cat code.py | claude -p '이 코드의 버그를 분석해주세요' --output-format json > analysis.json

# 스트리밍 JSON
cat log.txt | claude -p '이 로그 파일에서 에러를 파싱해주세요' --output-format stream-json
```

---

## 커스텀 슬래시 명령어 만들기

### 프로젝트 명령어

```bash
mkdir -p .claude/commands
echo "이 코드의 성능을 분석하고 세 가지 구체적인 최적화를 제안해주세요:" > .claude/commands/optimize.md
```

```
> /optimize
```

### 인자가 있는 명령어

```bash
echo '이슈 #$ARGUMENTS를 찾아 수정해주세요. 다음 단계를 따르세요: 1. 티켓에 설명된 이슈 이해 2. 관련 코드 위치 파악 3. 근본 원인을 해결하는 솔루션 구현 4. 적절한 테스트 추가 5. 간결한 PR 설명 준비' > .claude/commands/fix-issue.md
```

```
> /fix-issue 123
```

### 개인 명령어

```bash
mkdir -p ~/.claude/commands
echo "이 코드의 보안 취약점을 리뷰해주세요. 다음에 집중해주세요:" > ~/.claude/commands/security-review.md
```

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

## 다음 단계

| 주제 | 설명 | 문서 |
|------|------|------|
| 서브에이전트 | 특화된 AI 어시스턴트 만들기 | [서브에이전트](08-subagents.md) |
| 출력 스타일 | 응답 스타일 커스터마이징 | [출력 스타일](17-output-styles.md) |
| GitHub Actions | CI 자동화 | [GitHub Actions](12-github-actions.md) |
| SDK | 프로그래밍 통합 | [SDK](11-sdk.md) |
