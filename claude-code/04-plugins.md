# 04. 플러그인 시스템 (Plugins)

> **참조**: [https://docs.anthropic.com/en/docs/claude-code/plugins](https://docs.anthropic.com/en/docs/claude-code/plugins)
> **참조**: [https://code.claude.com/docs/en/plugins](https://code.claude.com/docs/en/plugins)

---

## 1. 플러그인 개요

Claude Code 플러그인은 커스텀 commands, agents, skills, hooks, MCP 서버를 **하나의 패키지**로 묶어 프로젝트와 팀 간에 공유할 수 있게 해주는 시스템입니다. 마켓플레이스를 통해 설치하거나 직접 만들 수 있습니다.

### 독립형 설정 vs 플러그인

| 구분 | 스킬 이름 | 적합한 경우 |
|------|-----------|-------------|
| **독립형** (`.claude/` 디렉토리) | `/hello` | 개인 워크플로우, 프로젝트별 커스터마이징, 빠른 실험 |
| **플러그인** (`.claude-plugin/plugin.json` 포함) | `/plugin-name:hello` | 팀 공유, 커뮤니티 배포, 버전 관리, 다중 프로젝트 재사용 |

---

## 2. 플러그인 디렉토리 구조

모든 플러그인은 `.claude-plugin/plugin.json` 매니페스트 파일을 반드시 포함해야 합니다. 나머지 컴포넌트는 선택 사항입니다.

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # 필수: 플러그인 메타데이터
├── commands/                 # 선택: 커스텀 슬래시 명령어 (마크다운 파일)
│   └── hello.md
├── agents/                   # 선택: 커스텀 에이전트 정의
│   └── helper.md
├── skills/                   # 선택: 에이전트 스킬 (SKILL.md 디렉토리)
│   └── my-skill/
│       └── SKILL.md
├── hooks/                    # 선택: 이벤트 핸들러
│   └── hooks.json
├── .mcp.json                 # 선택: MCP 서버 설정
├── .lsp.json                 # 선택: LSP 서버 설정
├── monitors/                 # 선택: 백그라운드 모니터
│   └── monitors.json
├── bin/                      # 선택: 실행 파일 (PATH에 추가됨)
├── settings.json             # 선택: 플러그인 활성화 시 적용할 기본 설정
└── README.md                 # 권장: 문서화
```

### 디렉토리 구성 요소 상세

| 디렉토리 | 위치 | 용도 |
|----------|------|------|
| `.claude-plugin/` | 플러그인 루트 | `plugin.json` 매니페스트 포함 |
| `skills/` | 플러그인 루트 | `<name>/SKILL.md` 형태의 스킬 |
| `commands/` | 플러그인 루트 | 마크다운 형식의 스킬 (신규는 `skills/` 권장) |
| `agents/` | 플러그인 루트 | 커스텀 에이전트 정의 |
| `hooks/` | 플러그인 루트 | `hooks.json` 이벤트 핸들러 |
| `.mcp.json` | 플러그인 루트 | MCP 서버 설정 |
| `.lsp.json` | 플러그인 루트 | LSP 서버 설정 (코드 인텔리전스) |
| `monitors/` | 플러그인 루트 | `monitors.json` 백그라운드 모니터 |
| `bin/` | 플러그인 루트 | 활성화 시 Bash 도구 PATH에 추가 |
| `settings.json` | 플러그인 루트 | 플러그인 활성화 시 기본 설정 적용 |

---

## 3. plugin.json 매니페스트 형식

`.claude-plugin/plugin.json`은 플러그인의 핵심 메타데이터를 정의합니다.

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "플러그인에 대한 설명",
  "author": "작성자 이름"
}
```

### 매니페스트 주요 필드

- **name**: 플러그인 식별자
- **version**: 시맨틱 버전 (선택)
- **description**: 플러그인 설명
- **author**: 작성자 정보

### 인라인 MCP 서버 정의

`plugin.json` 내에 MCP 서버를 직접 정의할 수도 있습니다.

```json
{
  "name": "my-plugin",
  "mcpServers": {
    "plugin-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server",
      "args": ["--port", "8080"]
    }
  }
}
```

### 플러그인 기본 에이전트 설정

`settings.json`을 통해 플러그인 활성화 시 기본 에이전트를 지정할 수 있습니다.

```json
{
  "agent": "security-reviewer"
}
```

---

## 4. 마켓플레이스 관리 명령어

### 마켓플레이스 추가

```bash
/plugin marketplace add your-org/claude-plugins
```

### 플러그인 설치

```bash
# 특정 플러그인 설치
/plugin install formatter@your-org
```

### 플러그인 활성화

```bash
/plugin enable plugin-name@marketplace-name
```

### 플러그인 비활성화 (삭제하지 않음)

```bash
/plugin disable plugin-name@marketplace-name
```

### 플러그인 완전 제거

```bash
/plugin uninstall plugin-name@marketplace-name
```

### 설치 확인

```bash
# 사용 가능한 명령어 확인
/help

# 플러그인 관리 인터페이스
/plugin
# "Browse Plugins" 선택
```

### 공식 마켓플레이스

Anthropic은 두 개의 공개 마켓플레이스를 운영합니다.

| 마켓플레이스 | 설명 |
|-------------|------|
| `claude-plugins-official` | Anthropic이 큐레이션한 공식 플러그인. 모든 Claude Code 설치에 자동 포함 |
| `claude-community` | 커뮤니티 제출 플러그인. `/plugin marketplace add anthropics/claude-plugins-community`로 추가 |

---

## 5. 팀 플러그인 워크플로우

### 개요

플러그인을 저장소 수준에서 설정하여 팀 전체에 동일한 도구를 일관되게 제공할 수 있습니다. 팀원이 저장소를 신뢰(trust)하면 Claude Code가 자동으로 마켓플레이스와 플러그인을 설치합니다.

### 설정 방법

**1단계**: 저장소의 `.claude/settings.json`에 마켓플레이스와 플러그인 설정 추가

```json
{
  "plugins": {
    "marketplaces": ["your-org/claude-plugins"],
    "enabled": ["formatter@your-org", "linter@your-org"]
  }
}
```

**2단계**: 팀원이 저장소 폴더를 신뢰(trust)

**3단계**: 플러그인이 모든 팀원에게 자동 설치됨

### 독립형 vs 플러그인 마이그레이션 비교

| 항목 | 독립형 (`.claude/`) | 플러그인 |
|------|---------------------|----------|
| 가용성 | 하나의 프로젝트에서만 | 마켓플레이스를 통해 공유 가능 |
| 파일 위치 | `.claude/commands/` | `plugin-name/commands/` |
| 훅 설정 | `settings.json` | `hooks/hooks.json` |
| 배포 | 수동 복사 | `/plugin install` |

---

## 6. 로컬 테스트

### `--plugin-dir` 플래그 사용

```bash
# 디렉토리에서 직접 로드
claude --plugin-dir ./my-plugin

# .zip 아카이브에서 로드 (v2.1.128+)
claude --plugin-dir ./my-plugin.zip
```

### `--plugin-url` 플래그 사용 (원격 아카이브)

```bash
# 단일 플러그인
claude --plugin-url https://example.com/my-plugin.zip

# 다중 플러그인
claude --plugin-url https://example.com/my-plugin.zip --plugin-url https://example.com/other.zip
```

### 실시간 리로드

플러그인 수정 후 재시작 없이 변경사항을 반영하려면:

```bash
/reload-plugins
```

이 명령은 플러그인, 스킬, 에이전트, 훅, 플러그인 MCP/LSP 서버를 모두 다시 로드합니다.

### skills 디렉토리에서 개발

```bash
# 스킬 디렉토리에 플러그인 스캐폴딩
claude plugin init my-tool
```

이 명령은 `~/.claude/skills/my-tool/` 디렉토리에 `.claude-plugin/plugin.json`과 `SKILL.md`를 생성합니다. 다음 세션부터 `my-tool@skills-dir`로 자동 로드됩니다.

---

## 7. 컴포넌트 타입 개요

플러그인은 다음 5가지(+) 컴포넌트 타입을 포함할 수 있습니다.

| 컴포넌트 | 설명 | 자세한 문서 |
|----------|------|-------------|
| **Commands** | 커스텀 슬래시 명령어. 마크다운 파일로 정의 | [슬래시 명령어 문서](./05-slash-commands.md) |
| **Agents** | 특정 작업에 특화된 커스텀 AI 서브에이전트 | Subagents 문서 참조 |
| **Skills** | 모델이 자동으로 호출하는 확장 기능. `SKILL.md`로 정의 | Agent Skills 문서 참조 |
| **Hooks** | 이벤트 기반 자동화 핸들러. `hooks/hooks.json`으로 정의 | [훅 시스템 문서](./06-hooks.md) |
| **MCP 서버** | 외부 도구/데이터 접근 통합. `.mcp.json` 또는 `plugin.json` 인라인 | [MCP 통합 문서](./07-mcp.md) |
| **LSP 서버** | 언어 서버 프로토콜 기반 코드 인텔리전스. `.lsp.json`으로 정의 | LSP 서버 문서 참조 |
| **Monitors** | 백그라운드 로그/파일 감시. `monitors/monitors.json`으로 정의 | Monitors 문서 참조 |

---

## 8. 디버깅 및 문제 해결

### 일반적인 문제 해결

1. **구조 확인**: 디렉토리가 `.claude-plugin/` 내부가 아닌 플러그인 루트에 있는지 확인
2. **개별 테스트**: 각 명령어, 에이전트, 훅을 개별적으로 확인
3. **검증 도구 사용**: `claude plugin validate` 실행

### 커뮤니티 제출

플러그인을 커뮤니티 마켓플레이스에 제출하려면:

1. 로컬에서 `claude plugin validate` 실행
2. 제출 폼 사용:
   - Claude.ai: claude.ai/settings/plugins/submit
   - Console: platform.claude.com/plugins/submit
