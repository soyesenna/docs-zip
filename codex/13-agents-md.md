# Codex CLI - AGENTS.md 가이드

> **원문**
> - Custom instructions with AGENTS.md: https://developers.openai.com/codex/guides/agents-md/
> - Customization: https://developers.openai.com/codex/concepts/customization
> - Memories: https://developers.openai.com/codex/memories

---

## 목차

1. [AGENTS.md란](#1-agentsmd란)
2. [포함할 내용](#2-포함할-내용)
3. [계층 구조와 우선순위](#3-계층-구조와-우선순위)
4. [글로벌 파일 설정](#4-글로벌-파일-설정)
5. [리포지토리 파일 설정](#5-리포지토리-파일-설정)
6. [Override 파일](#6-override-파일)
7. [Fallback 파일명](#7-fallback-파일명)
8. [AGENTS.md 업데이트 시점](#8-agentsmd-업데이트-시점)
9. [유지 관리 팁](#9-유지-관리-팁)
10. [고급 활용](#10-고급-활용)
11. [Memories와의 관계](#11-memories와의-관계)
12. [커스터마이제이션 빌드 순서](#12-커스터마이제이션-빌드-순서)
13. [문제 해결](#13-문제-해결)

---

## 1. AGENTS.md란

`AGENTS.md`는 Codex가 작업을 시작하기 전에 읽는 **지속적인 프로젝트 가이드**입니다. 리포지토리와 함께 이동하며, 에이전트가 어떤 작업을 수행하든 항상 동일한 규칙을 따르도록 보장합니다.

| 특성 | 설명 |
|------|------|
| 로드 시점 | Codex가 작업을 시작하기 전 (런당 한 번, TUI에서는 세션당 한 번) |
| 목적 | 에이전트가 매번 따라야 할 규칙 제공 |
| 위치 | 글로벌(`~/.codex/`), 리포지토리 루트, 하위 디렉토리 |
| 크기 제한 | 기본 32 KiB (`project_doc_max_bytes`로 변경 가능) |
| 파일 형식 | Markdown |

`AGENTS.md`는 에이전트에게 **README** 역할을 합니다. Codex는 작업을 시작할 때마다 자동으로 해당 파일들을 발견하고 로드하므로, 매번 수동으로 컨텍스트를 전달할 필요가 없습니다.

---

## 2. 포함할 내용

공식 문서에서 권장하는 `AGENTS.md`의 핵심 내용은 다음과 같습니다.

| 항목 | 설명 | 예시 |
|------|------|------|
| 빌드 및 테스트 명령어 | 프로젝트에서 사용하는 빌드/테스트 명령 | `npm run lint`, `make test-payments` |
| 리뷰 기대사항 | PR 리뷰 시 따라야 할 규칙 | "PR을 열기 전 lint 통과 필수" |
| 리포지토리 관례 | 코드 스타일, 네이밍, 디렉토리 구조 | "공개 유틸리티는 `docs/`에 문서화" |
| 디렉토리별 지침 | 특정 팀/모듈에만 적용되는 규칙 | "결제 서비스는 `make test-payments` 사용" |
| 의존성 정책 | 새 의존성 추가 시 규칙 | "새 프로덕션 의존성 추가 전 확인 요청" |

### 예시: 글로벌 AGENTS.md

```markdown
# ~/.codex/AGENTS.md

## Working agreements

- Always run `npm test` after modifying JavaScript files.
- Prefer `pnpm` when installing dependencies.
- Ask for confirmation before adding new production dependencies.
```

### 예시: 리포지토리 AGENTS.md

```markdown
# AGENTS.md

## Repository expectations

- Run `npm run lint` before opening a pull request.
- Document public utilities in `docs/` when you change behavior.
```

### 예시: 하위 디렉토리 Override

```markdown
# services/payments/AGENTS.override.md

## Payments service rules

- Use `make test-payments` instead of `npm test`.
- Never rotate API keys without notifying the security channel.
```

---

## 3. 계층 구조와 우선순위

Codex는 명령어 체인을 구성할 때 다음 순서로 파일을 탐색하고 병합합니다.

### 탐색 순서

```
1. 글로벌 스코프 (~/.codex/)
   ├── AGENTS.override.md  (존재하면 우선 사용)
   └── AGENTS.md            (override가 없으면 사용)

2. 프로젝트 스코프 (Git 루트 → 현재 디렉토리)
   ├── <repo-root>/AGENTS.override.md  또는 AGENTS.md
   ├── <repo-root>/subdir/AGENTS.override.md  또는 AGENTS.md
   └── <cwd>/AGENTS.override.md  또는 AGENTS.md
```

### 병합 규칙

| 규칙 | 설명 |
|------|------|
| 병합 방향 | 루트에서 현재 디렉토리 방향으로 연결 |
| 우선순위 | 현재 디렉토리에 가까울수록 뒤에 나타나므로 우선순위가 높음 |
| 디렉토리당 최대 파일 | 1개 (`AGENTS.override.md` > `AGENTS.md` > fallback 파일명 순) |
| 크기 제한 | 합산 크기가 `project_doc_max_bytes` (기본 32 KiB) 에 도달하면 중단 |
| 빈 파일 | 건너뜀 |

### 탐색 상세 로직

1. **글로벌 스코프**: Codex 홈 디렉토리(기본 `~/.codex`, `CODEX_HOME`으로 변경 가능)에서 `AGENTS.override.md`를 먼저 확인. 존재하면 사용, 없으면 `AGENTS.md` 사용. 이 레벨에서는 첫 번째로 비어있지 않은 파일 하나만 사용.
2. **프로젝트 스코프**: 프로젝트 루트(일반적으로 Git 루트)에서 시작하여 현재 작업 디렉토리까지 하향 탐색. 각 디렉토리에서 `AGENTS.override.md` → `AGENTS.md` → fallback 파일명 순서로 확인. 디렉토리당 최대 1개 파일만 포함.
3. **병합 순서**: 루트부터 아래 방향으로 빈 줄로 연결. 현재 디렉토리에 가까울수록 오버라이드 권한이 높음(프롬프트에서 뒤에 나타나기 때문).

### 파일 트리 예시

```
AGENTS.md              ← 리포지토리 기대사항
services/
  payments/
    AGENTS.md          ← override가 있으므로 무시됨
    AGENTS.override.md ← 결제 서비스 규칙 (활성)
    README.md
  search/
    AGENTS.md          ← 검색 서비스 규칙
    ...
```

---

## 4. 글로벌 파일 설정

글로벌 파일은 모든 리포지토리에 상속되는 개인 작업 규약을 정의합니다.

### 설정 방법

```bash
# 1. 디렉토리 확인
mkdir -p ~/.codex

# 2. 글로벌 AGENTS.md 생성
cat > ~/.codex/AGENTS.md << 'EOF'
# ~/.codex/AGENTS.md

## Working agreements

- Always run `npm test` after modifying JavaScript files.
- Prefer `pnpm` when installing dependencies.
- Ask for confirmation before adding new production dependencies.
EOF

# 3. 로드 확인
codex --ask-for-approval never "Summarize the current instructions."
```

### 글로벌 vs 리포지토리 파일 용도 구분

| 레이어 | 글로벌 (`~/.codex/AGENTS.md`) | 리포지토리 (루트 `AGENTS.md`) |
|--------|------|------|
| 대상 | 개인 (개발자 본인) | 팀 전체 |
| 목적 | Codex와의 소통 방식(리뷰 스타일, 상세도, 기본값) | 코드베이스 규칙, 팀 컨벤션 |
| 버전 관리 | 로컬만 | Git으로 관리 공유 |
| 예시 | "항상 이모지로 응답", "git 명령은 명시적 요청 시만" | "PR 전 lint 필수", "빌드는 `make build`" |

### 글로벌 Override

`~/.codex/AGENTS.override.md`를 사용하면 기본 글로벌 파일을 삭제하지 않고 임시로 다른 규칙을 적용할 수 있습니다.

```bash
# 임시 override 생성
echo "# 실험 모드\n- 더 간결한 응답 사용" > ~/.codex/AGENTS.override.md

# 복구하려면 override 삭제
rm ~/.codex/AGENTS.override.md
```

---

## 5. 리포지토리 파일 설정

리포지토리 레벨의 파일은 팀 전체가 공유하는 프로젝트 규칙을 담습니다. 글로벌 기본값을 그대로 상속하면서 프로젝트 고유의 규칙을 추가합니다.

### 설정 방법

```bash
# 리포지토리 루트에 생성
cat > AGENTS.md << 'EOF'
# AGENTS.md

## Repository expectations

- Run `npm run lint` before opening a pull request.
- Document public utilities in `docs/` when you change behavior.
EOF
```

### 하위 디렉토리에 규칙 추가

특정 팀이나 모듈에 다른 규칙이 필요한 경우, 해당 디렉토리에 override 파일을 생성합니다.

```bash
# 결제 서비스 전용 규칙
mkdir -p services/payments
cat > services/payments/AGENTS.override.md << 'EOF'
# services/payments/AGENTS.override.md

## Payments service rules

- Use `make test-payments` instead of `npm test`.
- Never rotate API keys without notifying the security channel.
EOF

# 특정 디렉토리에서 Codex 실행하여 확인
codex --cd services/payments --ask-for-approval never \
  "List the instruction sources you loaded."
```

---

## 6. Override 파일

`AGENTS.override.md`는 같은 디렉토리에 있는 `AGENTS.md` 대신 사용되는 파일입니다.

| 동작 | 설명 |
|------|------|
| 파일명 | `AGENTS.override.md` |
| 적용 조건 | 같은 디렉토리에 존재하면 `AGENTS.md` 대신 로드 |
| 용도 | 기존 `AGENTS.md`를 수정하지 않고 임시/특수 규칙 적용 |
| 복구 | override 파일 삭제만으로 원래 규칙으로 복귀 |

### Override 동작 예시

```
services/payments/
  AGENTS.md            ← override가 있으므로 무시됨
  AGENTS.override.md   ← 이 파일이 대신 로드됨
```

---

## 7. Fallback 파일명

리포지토리에서 이미 다른 파일명(예: `TEAM_GUIDE.md`)을 사용 중인 경우, Codex 설정에 fallback 파일명을 추가하여 `AGENTS.md`처럼 취급할 수 있습니다.

### 설정

```toml
# ~/.codex/config.toml
project_doc_fallback_filenames = ["TEAM_GUIDE.md", ".agents.md"]
project_doc_max_bytes = 65536
```

### 탐색 순서 (fallback 포함)

각 디렉토리에서 다음 순서로 파일을 확인합니다.

| 순서 | 파일명 |
|------|--------|
| 1 | `AGENTS.override.md` |
| 2 | `AGENTS.md` |
| 3 | `TEAM_GUIDE.md` (fallback) |
| 4 | `.agents.md` (fallback) |

이 목록에 없는 파일명은 명령어 탐색에서 무시됩니다.

### Fallback 적용 예시

```
TEAM_GUIDE.md       ← fallback 목록에서 감지됨
.agents.md          ← 루트의 fallback 파일
support/
  AGENTS.override.md ← fallback 가이드를 오버라이드
  playbooks/
    ...
```

---

## 8. AGENTS.md 업데이트 시점

공식 문서에서 권장하는 `AGENTS.md` 업데이트 상황입니다.

| 상황 | 설명 | 액션 |
|------|------|------|
| **반복 실수** | 에이전트가 같은 실수를 반복할 때 | 규칙 추가 |
| **과도한 파일 읽기** | 올바른 파일을 찾지만 너무 많은 문서를 읽을 때 | 라우팅 가이드 추가 (우선순위 디렉토리/파일 지정) |
| **반복되는 PR 피드백** | 같은 피드백을 두 번 이상 남길 때 | 피드백을 규칙으로 명문화 |
| **GitHub에서** | PR 코멘트에서 `@codex` 태그로 요청 | `@codex add this to AGENTS.md` |
| **드리프트 자동 감지** | 자동화를 활용한 정기 체크 | 일일 자동화로 가이드 갭 탐지 |

---

## 9. 유지 관리 팁

### 기본 원칙

- **작고 정확하게 유지**: 처음부터 모든 것을 담지 말고, 중요한 지침만 포함
- **피드백 루프로 활용**: 에이전트가 코드베이스에 대해 잘못된 가정을 하면 `AGENTS.md`에서 수정하고, 에이전트에게 `AGENTS.md` 업데이트를 요청하여 수정 사항이 지속되도록 함
- **반복 리뷰 피드백 명문화**: 자주 발생하는 리뷰 피드백은 규칙으로 작성
- **가장 가까운 디렉토리에 배치**: 지침은 적용되는 가장 가까운 디렉토리에 작성

### 검증 명령어

```bash
# 현재 로드된 지침 확인
codex --ask-for-approval never "Summarize the current instructions."

# 특정 하위 디렉토리에서 로드된 파일 확인
codex --cd subdir --ask-for-approval never \
  "Show which instruction files are active."
```

### 로그 확인

세션 후 어떤 지침 파일이 로드되었는지 로그에서 확인할 수 있습니다.

```bash
# TUI 로그 확인
cat ~/.codex/log/codex-tui.log

# 세션 로깅을 활성화한 경우
# 가장 최근 session-*.jsonl 파일 확인
```

Codex는 매 실행 시 (그리고 각 TUI 세션 시작 시) 명령어 체인을 재구성하므로, 수동으로 지울 캐시가 없습니다. 지침이 오래된 것처럼 보이면 해당 디렉토리에서 Codex를 다시 시작하십시오.

---

## 10. 고급 활용

### @codex를 통한 PR에서 업데이트

GitHub PR 코멘트에서 `@codex`를 태그하여 `AGENTS.md` 업데이트를 클라우드 작업으로 위임할 수 있습니다.

```
@codex add this to AGENTS.md
```

이 방식을 사용하면 로컬 환경에 접속하지 않고도 원격에서 `AGENTS.md`를 갱신할 수 있습니다.

### 자동화를 활용한 드리프트 체크

자동화(automations)를 사용하여 정기적으로(예: 매일) 가이드 갭을 탐지하고 `AGENTS.md`에 추가할 내용을 제안하도록 설정할 수 있습니다.

### 프리커밋 훅과 페어링

`AGENTS.md`의 규칙을 강제하는 인프라와 함께 사용하는 것이 좋습니다.

| 도구 | 역할 |
|------|------|
| Pre-commit hooks | 커밋 전에 규칙 위반 감지 |
| Linters | 코드 스타일 규칙 자동 검사 |
| Type checkers | 타입 오류 사전 방지 |

이러한 도구들은 사용자가 보기 전에 문제를 포착하므로, 시스템이 반복되는 실수를 예방하는 데 더욱 똑똑해집니다.

### CODEX_HOME 환경변수

`CODEX_HOME` 환경변수를 설정하면 다른 프로필(예: 프로젝트별 자동화 사용자)을 사용할 수 있습니다.

```bash
CODEX_HOME=$(pwd)/.codex codex exec "List active instruction sources"
```

---

## 11. Memories와의 관계

Memories는 Codex가 이전 스레드에서 학습한 유용한 컨텍스트를 미래 작업으로 전달하는 기능입니다. `AGENTS.md`와 보완 관계에 있습니다.

| 구분 | AGENTS.md | Memories |
|------|-----------|----------|
| 성격 | 명시적 규칙 | 학습된 컨텍스트 |
| 관리 방식 | 수동 작성 및 편집 | Codex가 자동 생성 |
| 저장 위치 | 리포지토리 내 파일 | `~/.codex/memories/` |
| 버전 관리 | Git 가능 | 생성된 상태 파일 |
| 필수 팀 규칙 | 여기에 보관 | 보조적 회상 레이어 |
| 가용성 | 항상 활성 | 비활성화 가능 |

### Memories 설정

```toml
# ~/.codex/config.toml
[features]
memories = true
```

> Memories는 기본적으로 비활성화되어 있으며, 유럽 경제 지역, 영국, 스위스에서는 출시 시점에 사용할 수 없습니다.

### Memories 주요 설정

| 설정 | 설명 |
|------|------|
| `memories.generate_memories` | 새 스레드를 메모리 생성 입력으로 저장할지 여부 |
| `memories.use_memories` | 기존 메모리를 미래 세션에 주입할지 여부 |
| `memories.disable_on_external_context` | MCP/웹 검색을 사용한 스레드를 메모리 생성에서 제외 |
| `memories.min_rate_limit_remaining_percent` | 메모리 생성 시작 전 최소 남은 rate-limit 비율 |
| `memories.extract_model` | 스레드별 메모리 추출에 사용할 모델 오버라이드 |
| `memories.consolidation_model` | 전역 메모리 통합에 사용할 모델 오버라이드 |

공식 문서에서는 **필수 팀 지침은 `AGENTS.md`나 체크인된 문서에 보관**하고, memories는 유용한 로컬 회상 레이어로 활용할 것을 권장합니다.

---

## 12. 커스터마이제이션 빌드 순서

공식 문서에서 권장하는 커스터마이제이션 적용 순서입니다.

| 단계 | 작업 | 설명 |
|------|------|------|
| 1 | `AGENTS.md`로 지침 설정 | 리포지토리 관례를 따르도록 설정. Pre-commit 훅과 linter를 추가하여 규칙 강제 |
| 2 | 플러그인 설치 또는 Skill 생성 | 재사용 가능한 워크플로가 이미 존재하면 플러그인 설치. 없으면 Skill으로 작성 후 플러그인으로 패키징 |
| 3 | MCP 연결 | 워크플로에 외부 시스템(Linear, GitHub, docs 서버, 디자인 도구)이 필요할 때 추가 |
| 4 | Subagents 활용 | 노이즈가 많거나 특수한 작업을 Subagent에 위임 |

### 커스터마이제이션 레이어 비교

| 레이어 | 글로벌 | 리포지토리 |
|--------|--------|------|
| AGENTS | `~/.codex/AGENTS.md` | 리포지토리 루트 또는 하위 디렉토리의 `AGENTS.md` |
| Skills | `$HOME/.agents/skills` | 리포지토리 내 `.agents/skills` |

---

## 13. 문제 해결

| 문제 | 원인 | 해결 방법 |
|------|------|------|
| 아무것도 로드되지 않음 | 잘못된 디렉토리이거나 파일이 비어 있음 | 올바른 리포지토리에 있는지 확인. `codex status`로 워크스페이스 루트 확인. 파일에 내용이 있는지 확인 |
| 잘못된 가이드가 나타남 | 상위 디렉토리나 Codex 홈에 `AGENTS.override.md`가 있음 | override 파일을 찾아 이름 변경 또는 삭제 |
| Fallback 파일명이 무시됨 | 설정에 오타가 있거나 Codex를 재시작하지 않음 | `project_doc_fallback_filenames`에 오타가 없는지 확인 후 Codex 재시작 |
| 지침이 잘림 | 합산 크기가 제한 초과 | `project_doc_max_bytes`를 늘리거나 큰 파일을 하위 디렉토리로 분리 |
| 프로필 혼란 | `CODEX_HOME`이 예상과 다름 | `echo $CODEX_HOME`으로 확인. 비기본값이면 편집한 디렉토리와 다를 수 있음 |
| 지침이 오래된 것처럼 보임 | 캐시 문제가 아님 | Codex를 해당 디렉토리에서 재시작 (캐시가 없으므로 재시작으로 해결) |

---

## 참고

- 공식 문서에서 `AGENTS.md`를 **작게 유지(Keep it small)** 할 것을 권장합니다.
- `AGENTS.md`는 피드백 루프로 활용해야 합니다. 에이전트가 잘못된 가정을 하면 `AGENTS.md`에서 수정하고, 에이전트에게 업데이트를 요청하십시오.
- Memories는 보조적 회상 레이어이며, 필수 팀 규칙의 유일한 출처로 사용해서는 안 됩니다.
