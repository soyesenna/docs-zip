# Claude Code 설정

> 설정 파일 계층, 사용 가능한 설정, 권한, 환경 변수, 글로벌 구성

**참고**: [Settings - Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/settings)

---

## 설정 파일 계층

`settings.json`은 계층적 설정을 통한 공식 구성 메커니즘입니다.

| 설정 파일 | 위치 | 설명 |
|-----------|------|------|
| **사용자 설정** | `~/.claude/settings.json` | 모든 프로젝트에 적용되는 개인 설정 |
| **공유 프로젝트 설정** | `.claude/settings.json` | 소스 컨트롤에 체크인되어 팀과 공유되는 설정 |
| **로컬 프로젝트 설정** | `.claude/settings.local.json` | 체크인되지 않는 개인 프로젝트 설정 (git이 자동으로 ignore 처리) |
| **엔터프라이즈 관리 설정** | 플랫폼별 경로 (아래 참조) | IT/DevOps가 배포, 재정의 불가 |

### 엔터프라이즈 관리 설정 파일 경로

| 플랫폼 | 경로 |
|--------|------|
| macOS | `/Library/Application Support/ClaudeCode/managed-settings.json` |
| Linux / WSL | `/etc/claude-code/managed-settings.json` |
| Windows | `C:\ProgramData\ClaudeCode\managed-settings.json` |

---

## 설정 우선순위

높은 순서에서 낮은 순서로:

| 우선순위 | 설정 소스 | 설명 |
|----------|-----------|------|
| 1 (최고) | **엔터프라이즈 관리 정책** | IT/DevOps가 배포, 재정의 불가 |
| 2 | **CLI 인수** | 특정 세션에 대한 임시 재정의 |
| 3 | **로컬 프로젝트 설정** | `.claude/settings.local.json` - 개인 프로젝트별 설정 |
| 4 | **공유 프로젝트 설정** | `.claude/settings.json` - 소스 컨트롤의 팀 공유 설정 |
| 5 (최저) | **사용자 설정** | `~/.claude/settings.json` - 개인 글로벌 설정 |

> 엔터프라이즈 보안 정책은 항상 강제되면서도 팀과 개인이 자신의 경험을 커스터마이즈할 수 있습니다.

### 구성 시스템 핵심 포인트

- **메모리 파일 (CLAUDE.md)**: Claude가 시작 시 로드하는 지침과 컨텍스트
- **설정 파일 (JSON)**: 권한, 환경 변수, 도구 동작 구성
- **슬래시 명령어**: 세션 중에 `/command-name`으로 실행 가능한 커스텀 명령
- **MCP 서버**: Claude Code를 추가 도구 및 통합으로 확장
- **상속**: 설정이 병합되며, 더 구체적인 설정이 더 포괄적인 설정에 추가되거나 재정의

---

## 사용 가능한 설정 전체

`settings.json`에서 지원하는 설정 목록입니다.

| 키 | 설명 | 예시 |
|----|------|------|
| `apiKeyHelper` | `/bin/sh`에서 실행할 커스텀 스크립트. 생성된 인증 값이 `X-Api-Key` 및 `Authorization: Bearer` 헤더로 전송됨 | `/bin/generate_temp_api_key.sh` |
| `cleanupPeriodDays` | 마지막 활동 날짜 기준으로 채팅 대화를 로컬에 보관할 기간 (기본값: 30일) | `20` |
| `env` | 모든 세션에 적용할 환경 변수 | `{"FOO": "bar"}` |
| `includeCoAuthoredBy` | git 커밋 및 PR에 `co-authored-by Claude` 바이라인 포함 여부 (기본값: `true`) | `false` |
| `permissions` | 권한 구성 (아래 권한 설정 참조) | |
| `hooks` | 도구 실행 전후에 실행할 커스텀 명령 구성 | `{"PreToolUse": {"Bash": "echo 'Running command...'"}}` |
| `disableAllHooks` | 모든 hooks 비활성화 | `true` |
| `model` | Claude Code에서 사용할 기본 모델 재정의 | `"claude-3-5-sonnet-20241022"` |
| `statusLine` | 컨텍스트를 표시할 커스텀 상태 라인 구성 | `{"type": "command", "command": "~/.claude/statusline.sh"}` |
| `outputStyle` | 시스템 프롬프트를 조정할 출력 스타일 구성 | `"Explanatory"` |
| `forceLoginMethod` | `claudeai`로 Claude.ai 계정만, `console`로 Console 계정만 로그인 제한 | `claudeai` |
| `forceLoginOrgUUID` | 로그인 시 자동으로 선택할 조직의 UUID. `forceLoginMethod` 설정 필요 | `"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"` |
| `enableAllProjectMcpServers` | 프로젝트 `.mcp.json` 파일에 정의된 모든 MCP 서버를 자동 승인 | `true` |
| `enabledMcpjsonServers` | `.mcp.json` 파일에서 승인할 특정 MCP 서버 목록 | `["memory", "github"]` |
| `disabledMcpjsonServers` | `.mcp.json` 파일에서 거부할 특정 MCP 서버 목록 | `["filesystem"]` |
| `awsAuthRefresh` | `.aws` 디렉토리를 수정하는 커스텀 스크립트 | `aws sso login --profile myprofile` |
| `awsCredentialExport` | AWS 자격 증명이 포함된 JSON을 출력하는 커스텀 스크립트 | `/bin/generate_aws_grant.sh` |
| `autoUpdatesChannel` | 자동 업데이트 채널 (`latest` 또는 `stable`) | `"stable"` |

---

## 권한 설정

`permissions` 객체 내에서 구성합니다.

| 키 | 설명 | 예시 |
|----|------|------|
| `allow` | 도구 사용을 허용하는 권한 규칙 배열. Bash 규칙은 정규식이 아닌 접두사 매칭을 사용 | `["Bash(git diff:*)"]` |
| `ask` | 도구 사용 시 확인을 요청하는 권한 규칙 배열 | `["Bash(git push:*)"]` |
| `deny` | 도구 사용을 거부하는 권한 규칙 배열. 민감한 파일을 Claude Code 접근에서 제외하는 데 사용 | `["WebFetch", "Bash(curl:*)", "Read(./.env)", "Read(./secrets/**)"]` |
| `additionalDirectories` | Claude가 접근할 수 있는 추가 작업 디렉토리 | `["../docs/"]` |
| `defaultMode` | Claude Code를 열 때의 기본 권한 모드 | `"acceptEdits"` |
| `disableBypassPermissionsMode` | `"disable"`으로 설정하면 `bypassPermissions` 모드 활성화 방지 | `"disable"` |

### 권한 예시

```json
{
  "permissions": {
    "allow": [
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Read"
    ],
    "deny": [
      "WebFetch",
      "Bash(curl:*)",
      "Read(./.env)",
      "Read(./secrets/**)"
    ],
    "additionalDirectories": ["../docs/"]
  }
}
```

> `deny` 패턴은 Bash 접두사 매칭이며 우회될 수 있습니다. 자세한 내용은 Bash 권한 제한 사항을 참조하세요.

---

## 환경 변수 전체

Claude Code의 동작을 제어하는 환경 변수 목록입니다. `settings.json`의 `env` 키에 설정하거나 쉘에서 직접 설정할 수 있습니다.

### 인증 및 API

| 변수 | 용도 |
|------|------|
| `ANTHROPIC_API_KEY` | `X-Api-Key` 헤더로 전송되는 API 키 (대화형 사용의 경우 `/login` 실행) |
| `ANTHROPIC_AUTH_TOKEN` | `Authorization` 헤더에 사용할 커스텀 값 (`Bearer` 접두사 자동 추가) |
| `ANTHROPIC_CUSTOM_HEADERS` | 요청에 추가할 커스텀 헤더 (`Name: Value` 형식) |

### 모델 구성

| 변수 | 용도 |
|------|------|
| `ANTHROPIC_MODEL` | 사용할 모델 이름 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Haiku 클래스 모델 이름 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Sonnet 클래스 모델 이름 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Opus 클래스 모델 이름 |
| `ANTHROPIC_SMALL_FAST_MODEL` | [DEPRECATED] 백그라운드 작업용 Haiku 클래스 모델 이름 |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | Bedrock 사용 시 Haiku 클래스 모델의 AWS 리전 재정의 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 서브에이전트 모델 구성 |

### AWS Bedrock

| 변수 | 용도 |
|------|------|
| `CLAUDE_CODE_USE_BEDROCK` | Bedrock 사용 |
| `AWS_BEARER_TOKEN_BEDROCK` | Bedrock 인증용 API 키 |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | Bedrock에 대한 AWS 인증 건너뛰기 (예: LLM 게이트웨이 사용 시) |
| `ANTHROPIC_BEDROCK_BASE_URL` | Bedrock 프록시 URL |

### Google Vertex AI

| 변수 | 용도 |
|------|------|
| `CLAUDE_CODE_USE_VERTEX` | Vertex AI 사용 |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | Vertex에 대한 Google 인증 건너뛰기 (예: LLM 게이트웨이 사용 시) |
| `ANTHROPIC_VERTEX_BASE_URL` | Vertex AI 프록시 URL |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | Claude 3.5 Haiku 리전 재정의 |
| `VERTEX_REGION_CLAUDE_3_5_SONNET` | Claude Sonnet 3.5 리전 재정의 |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | Claude 3.7 Sonnet 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | Claude 4.0 Opus 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | Claude 4.0 Sonnet 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | Claude 4.1 Opus 리전 재정의 |

### Bash 및 명령 실행

| 변수 | 용도 |
|------|------|
| `BASH_DEFAULT_TIMEOUT_MS` | 장시간 실행되는 bash 명령의 기본 타임아웃 |
| `BASH_MAX_TIMEOUT_MS` | 모델이 설정할 수 있는 최대 bash 타임아웃 |
| `BASH_MAX_OUTPUT_LENGTH` | bash 출력이 중간 잘림되기 전의 최대 문자 수 |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | 각 Bash 명령 후 원래 작업 디렉토리로 복귀 |

### 출력 및 토큰

| 변수 | 용도 |
|------|------|
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | 대부분의 요청에 대한 최대 출력 토큰 수 |
| `MAX_THINKING_TOKENS` | 모델의 thinking 예산 강제 설정 |
| `MAX_MCP_OUTPUT_TOKENS` | MCP 도구 응답에 허용되는 최대 토큰 수 (기본값: 25000) |

### MCP (Model Context Protocol)

| 변수 | 용도 |
|------|------|
| `MCP_TIMEOUT` | MCP 서버 시작 타임아웃 (밀리초) |
| `MCP_TOOL_TIMEOUT` | MCP 도구 실행 타임아웃 (밀리초) |

### 네트워크 및 프록시

| 변수 | 용도 |
|------|------|
| `HTTP_PROXY` | HTTP 프록시 서버 지정 |
| `HTTPS_PROXY` | HTTPS 프록시 서버 지정 |
| `NO_PROXY` | 프록시를 우회하여 직접 요청할 도메인 및 IP 목록 |

### mTLS 인증

| 변수 | 용도 |
|------|------|
| `CLAUDE_CODE_CLIENT_CERT` | mTLS 인증용 클라이언트 인증서 파일 경로 |
| `CLAUDE_CODE_CLIENT_KEY` | mTLS 인증용 클라이언트 개인 키 파일 경로 |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | 암호화된 CLAUDE_CODE_CLIENT_KEY의 암호 (선택) |

### 기능 제어

| 변수 | 용도 |
|------|------|
| `DISABLE_AUTOUPDATER` | `1`로 설정 시 자동 업데이트 비활성화 (`autoUpdates` 설정보다 우선) |
| `DISABLE_BUG_COMMAND` | `1`로 설정 시 `/bug` 명령 비활성화 |
| `DISABLE_COST_WARNINGS` | `1`로 설정 시 비용 경고 메시지 비활성화 |
| `DISABLE_ERROR_REPORTING` | `1`로 설정 시 Sentry 오류 보고 옵트아웃 |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | `1`로 설정 시 중요하지 않은 경로의 모델 호출 비활성화 |
| `DISABLE_TELEMETRY` | `1`로 설정 시 Statsig 원격 측정 옵트아웃 (코드, 파일 경로, bash 명령은 포함되지 않음) |

### 기타

| 변수 | 용도 |
|------|------|
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | `apiKeyHelper` 사용 시 자격 증명 새로고침 간격 (밀리초) |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | `DISABLE_AUTOUPDATER`, `DISABLE_BUG_COMMAND`, `DISABLE_ERROR_REPORTING`, `DISABLE_TELEMETRY`를 모두 설정하는 것과 동일 |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | `1`로 설정 시 대화 컨텍스트 기반 터미널 제목 자동 업데이트 비활성화 |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | IDE 확장 자동 설치 건너뛰기 |
| `USE_BUILTIN_RIPGREP` | `0`으로 설정 시 Claude Code 내장 `rg` 대신 시스템 설치된 `rg` 사용 |

---

## 글로벌 구성 설정

`claude config set -g <키> <값>`으로 설정합니다.

| 키 | 설명 | 예시 값 |
|----|------|---------|
| `autoUpdates` | [DEPRECATED] `DISABLE_AUTOUPDATER` 환경 변수 사용 권장 | `false` |
| `preferredNotifChannel` | 알림 수신 채널 (기본값: `iterm2`) | `iterm2`, `iterm2_with_bell`, `terminal_bell`, `notifications_disabled` |
| `theme` | 색상 테마 | `dark`, `light`, `light-daltonized`, `dark-daltonized` |
| `verbose` | 전체 bash 및 명령 출력 표시 여부 (기본값: `false`) | `true` |

---

## 설정 관리 명령어

기본적으로 `config`는 프로젝트 설정을 변경합니다. 글로벌 설정을 관리하려면 `--global` (또는 `-g`) 플래그를 사용합니다.

| 명령어 | 설명 | 예시 |
|--------|------|------|
| `claude config list` | 설정 나열 | `claude config list` |
| `claude config get <키>` | 특정 설정 확인 | `claude config get theme` |
| `claude config set <키> <값>` | 설정 변경 | `claude config set theme dark` |
| `claude config add <키> <값>` | 리스트 설정에 추가 | `claude config add permissions.allow "Bash(git log:*)"` |
| `claude config remove <키> <값>` | 리스트 설정에서 제거 | `claude config remove permissions.allow "Bash(git log:*)"` |

### 글로벌 설정 예시

```bash
# 테마 변경
claude config set -g theme dark

# 알림 채널 변경
claude config set -g preferredNotifChannel terminal_bell

# 상세 모드 활성화
claude config set -g verbose true
```

---

## 민감한 파일 제외 방법

API 키, 시크릿, 환경 파일 등 민감한 정보가 포함된 파일에 Claude Code가 접근하지 못하도록 하려면 `.claude/settings.json`의 `permissions.deny` 설정을 사용합니다.

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.local)",
      "Read(./secrets/**)",
      "Read(./**/credentials.json)",
      "Read(./**/*.pem)"
    ]
  }
}
```

이 패턴과 일치하는 파일은 Claude Code에서 완전히 보이지 않으며, 민감한 데이터의 우발적 노출을 방지합니다.

> 이 설정은 더 이상 사용되지 않는 `ignorePatterns` 구성을 대체합니다.

---

## 서브에이전트 구성

Claude Code는 사용자 및 프로젝트 수준에서 구성할 수 있는 커스텀 AI 서브에이전트를 지원합니다. 서브에이전트는 YAML 프론트매터가 포함된 Markdown 파일로 저장됩니다.

| 유형 | 위치 | 설명 |
|------|------|------|
| 사용자 서브에이전트 | `~/.claude/agents/` | 모든 프로젝트에서 사용 가능 |
| 프로젝트 서브에이전트 | `.claude/agents/` | 프로젝트별, 팀과 공유 가능 |
