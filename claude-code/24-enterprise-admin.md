# Claude Code 엔터프라이즈 관리

> 기업 환경 설정, 관리형 정책, 비용/사용량 모니터링, 규정 준수

**원문**: https://code.claude.com/docs/en/admin-setup | https://code.claude.com/docs/en/server-managed-settings | https://code.claude.com/docs/en/costs | https://code.claude.com/docs/en/monitoring-usage | https://code.claude.com/docs/en/analytics | https://code.claude.com/docs/en/llm-gateway | https://code.claude.com/docs/en/claude-platform-on-aws

---

## 1. 관리자 설정 — 조직 배포 결정 맵

Claude Code는 관리형 설정을 통해 조직 정책을 시행하며, 이 설정은 로컬 개발자 구성보다 우선 적용된다. 설정은 Claude 관리자 콘솔, MDM 시스템 또는 디스크 파일을 통해 전달할 수 있으며, 도구·명령어·서버·네트워크 대상을 제어한다.

### API 제공자 선택

Claude Code는 여러 API 제공자를 통해 Claude에 연결한다. 선택에 따라 요금 청구, 인증, 규정 준수 방식이 달라진다.

| 제공자 | 선택 기준 |
| --- | --- |
| Claude for Teams / Enterprise | Claude Code와 claude.ai를 하나의 시트당 구독으로 통합. 인프라 운영 불필요. 기본 권장 |
| Claude Console | API 우선 또는 사용량 기반(Pay-as-you-go) 과금 선호 시 |
| Amazon Bedrock | 기존 AWS 규정 준수 통제와 요금 청구를 상속받으려는 경우 |
| Google Vertex AI | 기존 GCP 규정 준수 통제와 요금 청구를 상속받으려는 경우 |
| Microsoft Foundry | 기존 Azure 규정 준수 통제와 요금 청구를 상속받으려는 경우 |

> Claude Platform on AWS는 AWS Marketplace 과금으로 Anthropic 운영 Claude API를 직접 사용하는 별도 배포 경로다. provider 표의 5개 제공자와는 별도 페이지로 존재하며, 자세한 구성은 아래 "Claude Platform on AWS" 섹션을 참조.

일부 Claude Code 기능은 Claude.ai 계정이 필요하다. Claude Code on the Web, Routines, Code Review, Remote Control, Chrome 확장은 Console API 키나 클라우드 제공자 자격 증명만으로는 사용할 수 없다.

### 설정 전달 방식

관리형 설정은 로컬 개발자 구성보다 우선하며, Claude Code는 다음 네 가지 위치에서 설정을 찾아 장치에서 처음 발견한 것을 사용한다.

| 메커니즘 | 전달 방식 | 우선순위 | 플랫폼 |
| --- | --- | --- | --- |
| 서버 관리형 | Claude.ai 관리자 콘솔 | 최고 | 전체 |
| plist / 레지스트리 정책 | macOS: `com.anthropic.claudecode` plist, Windows: `HKLM\SOFTWARE\Policies\ClaudeCode` | 높음 | macOS, Windows |
| 파일 기반 관리 | macOS: `/Library/Application Support/ClaudeCode/managed-settings.json`, Linux/WSL: `/etc/claude-code/managed-settings.json`, Windows: `C:\Program Files\ClaudeCode\managed-settings.json` | 중간 | 전체 |
| Windows 사용자 레지스트리 | `HKCU\SOFTWARE\Policies\ClaudeCode` | 최저 | Windows만 |

서버 관리형 설정은 인증 시점에 장치에 도달하며 활성 세션 중 매시간 새로고침된다. Claude for Teams 또는 Enterprise 플랜이 필요하다. 조직이 여러 제공자를 혼합하는 경우, Claude.ai 사용자에게는 서버 관리형 설정을 구성하고 다른 사용자에게는 파일 기반 또는 plist/레지스트리 대체 수단을 구성해야 한다.

plist와 HKLM 레지스트리 위치는 관리자 권한이 필요해 변조에 강하다. Windows 사용자 레지스트리(HKCU)는 권한 상승 없이 쓸 수 있으므로 강제 채널이 아닌 편의 기본값으로 취급한다.

기본적으로 WSL은 `/etc/claude-code`의 Linux 파일 경로만 읽는다. 같은 장치에서 Windows 레지스트리와 `C:\Program Files\ClaudeCode` 정책을 WSL로 확장하려면, admin 전용 Windows 소스 중 하나에 `wslInheritsWindowsSettings: true`를 설정한다.

어떤 메커니즘을 선택하든 관리형 값은 사용자 및 프로젝트 설정보다 우선한다. `permissions.allow`와 `permissions.deny` 같은 배열 설정은 모든 소스의 항목을 병합하므로, 개발자는 관리형 목록을 확장할 수 있지만 제거할 수는 없다.

### 강제 가능한 제어 항목

| 제어 | 역할 | 키 설정 |
| --- | --- | --- |
| 권한 규칙 | 특정 도구·명령 허용/질문/거부 | `permissions.allow`, `permissions.deny` |
| 권한 잠금 | 관리형 권한 규칙만 적용, `--dangerously-skip-permissions` 비활성화 | `allowManagedPermissionRulesOnly`, `permissions.disableBypassPermissionsMode` |
| 샌드박싱 | OS 수준 파일시스템·네트워크 격리 + 도메인 허용 목록 | `sandbox.enabled`, `sandbox.network.allowedDomains` |
| 관리형 정책 CLAUDE.md | 모든 세션에 로드되는 조직 전체 지침 | 관리형 정책 경로의 파일 |
| MCP 서버 제어 | 서버 allowlist/denylist 또는 고정 서버 세트 배포 | `allowedMcpServers`, `deniedMcpServers`, `allowManagedMcpServersOnly`, 또는 배포된 `managed-mcp.json` 파일 |
| 플러그인 마켓플레이스 제어 | 마켓플레이스 소스 제한 | `strictKnownMarketplaces`, `blockedMarketplaces` |
| 커스터마이징 잠금 | 스킬·에이전트·훅·MCP를 플러그인 또는 관리형 설정에서만 로드 | `strictPluginOnlyCustomization` |
| 훅 제한 | 관리형 훅만 로드, HTTP 훅 URL 제한 | `allowManagedHooksOnly`, `allowedHttpHookUrls` |
| 에이전트 뷰 비활성화 | `claude agents`, `--bg`, `/background`, on-demand supervisor 비활성화 | `disableAgentView` |
| 버전 하한선 | 자동 업데이트가 조직 최소 버전 미만 설치 방지 | `minimumVersion` |

### 검증 및 온보딩

관리형 설정 구성 후, 개발자가 Claude Code 내에서 `/status`를 실행하면 `Enterprise managed settings` 줄과 출처 `(remote)`, `(plist)`, `(HKLM)`, `(HKCU)`, `(file)` 중 하나가 표시된다.

---

## 2. 서버 관리형 설정

서버 관리형 설정은 Claude.ai의 웹 인터페이스를 통해 관리자가 Claude Code를 중앙에서 구성할 수 있게 한다. Claude Code 클라이언트는 사용자가 조직 자격 증명으로 인증할 때 이 설정을 자동으로 수신한다.

### 요구 사항

- Claude for Teams 또는 Claude for Enterprise 플랜
- Claude for Teams: Claude Code v2.1.38 이상, Enterprise: v2.1.30 이상
- `api.anthropic.com`에 대한 네트워크 접근

### 서버 관리형 vs 엔드포인트 관리형

| 방식 | 적합한 환경 | 보안 모델 |
| --- | --- | --- |
| 서버 관리형 | MDM이 없거나 비관리 장치의 사용자 | Anthropic 서버에서 설정 전달 |
| 엔드포인트 관리형 | MDM 또는 엔드포인트 관리 솔루션이 있는 환경 | OS 수준에서 설정 파일 보호 가능 |

### 설정 우선순위

서버 관리형과 엔드포인트 관리형 모두 Claude Code 설정 계층에서 최상위 티어를 차지한다. 관리 티어 내에서는 비어있지 않은 구성을 처음으로 전달하는 소스가 적용된다. 서버 관리형이 먼저 확인되고, 엔드포인트 관리형이 그 다음이다. 소스 간 병합은 없다.

### 설정 전달 검증

설정이 적용 중인지 확인하려면:

- 사용자에게 Claude Code를 다시 시작하도록 안내. 설정에 보안 승인 다이얼로그를 트리거하는 항목이 포함된 경우, 시작 시 관리형 설정을 설명하는 프롬프트가 표시된다.
- 관리형 권한 규칙이 활성인지 확인하려면 사용자가 `/permissions`를 실행하여 유효 권한 규칙을 확인.
- `/status`로 활성 관리 소스를 확인.

### 가져오기 및 캐싱 동작

- **첫 실행(캐시 없음)**: 설정을 비동기로 가져온다. 가져오기 실패 시 관리형 설정 없이 계속 진행
- **후속 실행(캐시 있음)**: 캐시된 설정이 즉시 적용되고, 백그라운드에서 새 설정을 가져온다

### 전달된 설정의 무효 항목 (v2.1.169+)

전달된 payload는 다른 관리 소스와 동일한 규칙으로 관용 파싱된다. payload에 스키마 검증에 실패하는 항목이 포함된 경우, Claude Code는 해당 항목을 제거(strip)하고 검증 에러를 표시한 뒤 나머지 유효한 설정을 모두 적용한다. Claude Code v2.1.169 이상 필요.

서버 관리형 전달은 다음 동작을 추가한다:

- `~/.claude/remote-settings.json` 캐시는 무효 항목이 제거된 salvage된 payload를 저장한다. 원시 무효 payload는 영구 저장되지 않는다.
- payload에서 salvage 가능한 필드가 하나도 없으면, Claude Code는 마지막으로 수락된 캐시 설정을 유지하고 fatal error를 기록한다.
- 보안 승인 다이얼로그는 salvage된 payload를 평가하므로, 제거된 무효 항목은 승인 대상으로 제출되거나 실행되지 않는다.

전달 문제를 디버깅하려면 `claude --debug-file <path>`를 실행하고 로그에서 `Remote settings`를 검색하라. payload 변경 사항은 조직에 롤아웃하기 전에 테스트 장비에서 `claude doctor`로 검증하라.

### 폐쇄형 강제(fail-closed) 시작

기본적으로 원격 설정 가져오기가 실패하면 CLI가 관리형 설정 없이 계속 진행된다. 이 미적용 구간이 허용되지 않는 환경에서는 `forceRemoteSettingsRefresh: true`를 설정한다.

```json
{
  "forceRemoteSettingsRefresh": true
}
```

이 설정이 활성화되면 CLI는 시작 시 원격 설정을 새로 가져올 때까지 차단한다. 가져오기가 실패하면 CLI가 종료된다. v2.1.139부터 `claude auth` 하위 명령어는 이 검사에서 제외된다.

### 보안 승인 대화상자

다음 설정은 보안 위험을 초래할 수 있어 사용자의 명시적 승인이 필요하다.

- 셸 명령어를 실행하는 설정
| 커스텀 환경 변수 (알려진 안전 허용 목록 외)
- 훅 정의

### 플랫폼 가용성

서버 관리형 설정은 `api.anthropic.com`에 직접 연결해야 하며, 다음 환경에서는 사용할 수 없다.

- Amazon Bedrock
- Google Vertex AI
- Microsoft Foundry
- `ANTHROPIC_BASE_URL` 또는 LLM 게이트웨이를 통한 커스텀 API 엔드포인트

### 감사 로깅

설정 변경에 대한 감사 로그 이벤트는 Compliance API 또는 감사 로그 내보내기를 통해 사용할 수 있다. 이벤트에는 수행된 작업 유형, 작업을 수행한 계정 및 장치, 이전 값과 새 값에 대한 참조가 포함된다.

### 현재 제약사항

서버 관리형 설정에는 다음 제약이 있다.

- 조직 내 모든 사용자에게 설정이 균일하게 적용된다. 그룹별 구성은 아직 지원되지 않는다.
- `managed-mcp.json` 파일은 서버 관리형 설정을 통해 배포할 수 없다. 대신 `allowedMcpServers`와 `deniedMcpServers` 정책 키를 사용하라.
- `policyHelper` 및 `wslInheritsWindowsSettings` 등 OS 수준 정책 소스로 제한된 설정은 서버 관리형에서 적용되지 않는다. MDM 또는 시스템 `managed-settings.json` 파일을 통해 배포하라.

### 보안 고려사항

서버 관리형 설정은 중앙 집중식 정책 시행을 제공하지만, 클라이언트 측 제어로 작동한다. 비관리 장치에서 관리자 또는 sudo 접근 권한이 있는 사용자는 Claude Code 바이너리, 파일시스템 또는 네트워크 구성을 수정할 수 있다.

| 시나리오 | 동작 |
| --- | --- |
| 사용자가 캐시된 설정 파일 편집 | 조작된 파일이 시작 시 적용되지만, 다음 서버 가져오기 시 올바른 설정으로 복원 |
| 사용자가 캐시된 설정 파일 삭제 | 첫 실행 동작 발생: 설정을 비동기로 가져오며 짧은 미적용 구간 존재 |
| API 사용 불가 | 캐시된 설정이 있으면 적용, 없으면 다음 성공적 가져오기까지 관리형 설정 미시행. `forceRemoteSettingsRefresh: true` 시 `claude auth` 하위 명령을 제외하고 CLI 종료 |
| 사용자가 다른 조직으로 인증 | 관리 조직 외부 계정에는 설정이 전달되지 않음 |
| 사용자가 타사 모델 제공자 구성 | 서버 관리형 설정이 우회됨. `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_MANTLE`, `CLAUDE_CODE_USE_VERTEX`, `CLAUDE_CODE_USE_FOUNDRY` 또는 비기본 `ANTHROPIC_BASE_URL` 설정 시 포함 |

런타임 구성 변경을 감지하려면 `ConfigChange` 훅을 사용하여 변경 사항을 로깅하거나 승인되지 않은 변경을 차단할 수 있다. 더 강력한 시행을 위해서는 MDM에 등록된 장치에서 엔드포인트 관리형 설정을 사용하라.

### 접근 제어

서버 관리형 설정을 관리할 수 있는 역할:

- **Primary Owner**
- **Owner**

---

## 3. 관리형 MCP

기본적으로 Claude Code를 실행하는 누구나 원하는 MCP 서버를 연결할 수 있다. 관리자는 조직 내에서 실행되는 서버를 제한할 수 있다.

### 제어 패턴

| 패턴 | 동작 | 구성 |
| --- | --- | --- |
| MCP 비활성화 | 어떤 서버도 로드되지 않음 | `managed-mcp.json`에 빈 서버 맵 |
| 고정 배포 | 모든 사용자가 동일한 서버 세트를 받음 | `managed-mcp.json`에 원하는 서버 지정 |
| 승인된 카탈로그 | 승인된 서버 목록 게시, 사용자가 원하는 것만 추가 | `allowedMcpServers` + `allowManagedMcpServersOnly: true` |
| 플러그인 서버만 | 플러그인에서만 서버 제공 | `strictPluginOnlyCustomization`에 `mcp` 포함 |
| 소프트 allowlist | 사용자가 자신의 설정에서 확장 가능한 allowlist | `allowedMcpServers` (allowManagedMcpServersOnly 없이) |
| Denylist만 | 알려진 위험 서버 차단 | `deniedMcpServers` |
| 제한 없음 | 사용자가 자유롭게 추가 | 관리형 MCP 구성 배포 안 함 |

### managed-mcp.json 배포

`managed-mcp.json` 파일을 배포하면 Claude Code는 해당 파일에 정의된 서버만 로드한다. 사용자는 다른 MCP 서버를 추가·수정·사용할 수 없다.

파일 경로:

| 플랫폼 | 경로 |
| --- | --- |
| macOS | `/Library/Application Support/ClaudeCode/managed-mcp.json` |
| Linux 및 WSL | `/etc/claude-code/managed-mcp.json` |
| Windows | `C:\Program Files\ClaudeCode\managed-mcp.json` |

파일 형식 예시:

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "sentry": {
      "type": "http",
      "url": "https://mcp.sentry.dev/mcp"
    },
    "company-internal": {
      "type": "stdio",
      "command": "/usr/local/bin/company-mcp-server",
      "args": ["--config", "/etc/company/mcp-config.json"],
      "env": {
        "COMPANY_API_URL": "https://internal.example.com"
      }
    }
  }
}
```

### Allowlist 및 Denylist

`allowedMcpServers`와 `deniedMcpServers`는 구성된 서버 중 로드가 허용되는 것을 필터링한다.

| 키 | 매칭 대상 | 용도 |
| --- | --- | --- |
| `serverUrl` | 원격 서버 URL, `*` 와일드카드 지원 | HTTP/SSE 서버 |
| `serverCommand` | stdio 서버를 시작하는 명령어와 인자 (정확히 일치) | Stdio 서버 |
| `serverName` | 사용자 지정 레이블, 정확한 일치만 | 두 유형 모두 |

서버 평가 순서:

1. **목록 병합**: 모든 설정 소스의 항목을 하나로 병합. `allowManagedMcpServersOnly`가 true면 관리형 allowlist만 유지
2. **Denylist 확인**: denylist 항목과 일치하면 차단. 무효화 불가
3. **Allowlist 확인**: allowlist가 설정되지 않은 경우 denylist를 통과한 모든 서버가 로드. 설정된 경우 서버 유형에 따라 매칭 필요

---

## 4. IAM (인증 및 접근 관리)

Claude Code는 설정에 따라 여러 인증 방식을 지원한다.

### 인증 방법

| 계정 유형 | 설명 |
| --- | --- |
| Claude Pro 또는 Max 구독 | Claude.ai 계정으로 로그인 |
| Claude for Teams 또는 Enterprise | 팀 관리자가 초대한 Claude.ai 계정 |
| Claude Console | Console 자격 증명. 관리자가 먼저 초대해야 함 |
| 클라우드 제공자 | Amazon Bedrock, Google Vertex AI, Microsoft Foundry — 환경 변수 설정 필요 |

### Claude for Teams vs Enterprise

- **Claude for Teams**: 셀프 서비스 플랜. 협업 기능, 관리 도구, 요금 관리 포함. 소규모 팀에 적합
- **Claude for Enterprise**: SSO, 도메인 캡처, 역할 기반 권한, Compliance API, 관리형 정책 설정 추가. 보안 및 규정 준수 요구사항이 있는 대규모 조직에 적합

### 인증 우선순위

여러 자격 증명이 있을 때 Claude Code는 다음 순서로 선택한다.

1. 클라우드 제공자 자격 증명 (`CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX`, `CLAUDE_CODE_USE_FOUNDRY` 설정 시)
2. `ANTHROPIC_AUTH_TOKEN` 환경 변수 — Bearer 헤더로 전송
3. `ANTHROPIC_API_KEY` 환경 변수 — `X-Api-Key` 헤더로 전송
4. `apiKeyHelper` 스크립트 출력 — 동적/회전 자격 증명
5. `CLAUDE_CODE_OAUTH_TOKEN` 환경 변수 — `claude setup-token`으로 생성
6. `/login`을 통한 구독 OAuth 자격 증명 (기본값)

### 자격 증명 관리

| 플랫폼 | 저장 위치 |
| --- | --- |
| macOS | 암호화된 macOS Keychain |
| Linux | `~/.claude/.credentials.json` (파일 모드 `0600`) |
| Windows | `%USERPROFILE%\.claude\.credentials.json` (사용자 프로필 디렉토리 접근 제어 상속) |

`apiKeyHelper`는 기본적으로 5분 후 또는 HTTP 401 응답 시 호출된다. `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` 환경 변수로 갱신 간격을 커스터마이즈할 수 있다.

### 장기 토큰 생성

CI 파이프라인, 스크립트 등 대화형 브라우저 로그인이 불가능한 환경에서는 `claude setup-token`으로 1년 유효 OAuth 토큰을 생성할 수 있다.

```bash
export CLAUDE_CODE_OAUTH_TOKEN=your-token
```

---

## 5. LLM 게이트웨이

LLM 게이트웨이는 Claude Code와 모델 제공자 사이의 중앙 집중식 프록시 계층을 제공한다.

### 게이트웨이 기능

- **중앙 집중식 인증** — API 키 관리의 단일 지점
- **사용량 추적** — 팀 및 프로젝트 전반의 사용량 모니터링
- **비용 통제** — 예산 및 요금 제한 구현
- **감사 로깅** — 규정 준수를 위한 모든 모델 상호작용 추적
- **모델 라우팅** — 코드 변경 없이 제공자 간 전환

### 게이트웨이 요구 사항

게이트웨이는 최소 하나 이상의 API 형식을 노출해야 한다.

| API 형식 | 엔드포인트 |
| --- | --- |
| Anthropic Messages | `/v1/messages`, `/v1/messages/count_tokens` |
| Bedrock InvokeModel | `/invoke`, `/invoke-with-response-stream` |
| Vertex rawPredict | `:rawPredict`, `:streamRawPredict`, `/count-tokens:rawPredict` |

요청 헤더에서 `anthropic-beta`, `anthropic-version`을 전달해야 한다.

### 요청 헤더

| 헤더 | 설명 |
| --- | --- |
| `X-Claude-Code-Session-Id` | 현재 세션의 고유 식별자 |
| `X-Claude-Code-Agent-Id` | 요청을 발행한 하위 에이전트/팀원의 식별자 |
| `X-Claude-Code-Parent-Agent-Id` | 요청 에이전트를 생성한 상위 에이전트의 식별자 |

두 에이전트 ID 헤더는 영구 사용자/장치 ID가 아닌 spawn마다 고유한 임시 식별자다. Claude Code는 또한 시스템 프롬프트에 클라이언트 버전과 대화에서 파생된 지문이 포함된 짧은 attribution 블록을 앞에 추가한다. Anthropic API는 처리 전 이 블록을 제거하므로 자체 prompt cache에는 영향이 없다. 게이트웨이가 전체 요청 본문에 키를 둔 자체 prompt cache를 구현하는 경우, 이 블록을 생략하려면 `CLAUDE_CODE_ATTRIBUTION_HEADER=0`을 설정하라. 추가로 `ANTHROPIC_CUSTOM_HEADERS`에 지정된 헤더들도 함께 전송된다.

### 모델 검색

`CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`을 설정하면 Claude Code가 게이트웨이의 `/v1/models` 엔드포인트를 쿼리하여 `/model` 선택기에 모델을 추가한다. v2.1.129 이상 필요. Anthropic Messages 형식에만 적용된다( Bedrock/Vertex 패스스루 엔드포인트 및 `ANTHROPIC_BASE_URL`이 없거나 `api.anthropic.com`인 경우에는 실행되지 않는다). 검색 요청은 `ANTHROPIC_AUTH_TOKEN`을 Bearer 토큰으로, 또는 `ANTHROPIC_API_KEY`를 `x-api-key` 헤더로 전송하며, `ANTHROPIC_CUSTOM_HEADERS`의 헤더도 함께 보낸다. `claude` 또는 `anthropic`으로 시작하는 모델만 선택기에 추가되며, 각 항목은 "From gateway"로 라벨링되고 응답에 `display_name` 필드가 있으면 이를 사용한다. 결과는 `~/.claude/cache/gateway-models.json`에 캐시되고 매 시작 시 새로고침된다. 요청이 실패하거나 게이트웨이가 `/v1/models`를 구현하지 않으면, 이전 시작의 캐시 목록 또는 내장 모델 목록으로 폴백한다.

### 동적 API 키 (apiKeyHelper)

회전 키 또는 사용자별 인증이 필요한 경우 `apiKeyHelper` 스크립트를 구성할 수 있다.

1. API 키 헬퍼 스크립트 작성:

```bash
#!/bin/bash
# ~/bin/get-litellm-key.sh

# 예: Vault에서 키 가져오기
vault kv get -field=api_key secret/litellm/claude-code

# 예: JWT 토큰 생성
jwt encode \
  --secret="${JWT_SECRET}" \
  --exp="+1h" \
  '{"user":"'${USER}'","team":"engineering"}'
```

2. Claude Code 설정에서 헬퍼 사용:

```json
{
  "apiKeyHelper": "~/bin/get-litellm-key.sh"
}
```

3. 토큰 갱신 간격 설정:

```bash
# 1시간마다 갱신 (3600000 ms)
export CLAUDE_CODE_API_KEY_HELPER_TTL_MS=3600000
```

헬퍼 출력은 `Authorization` 및 `X-Api-Key` 헤더로 전송된다. `apiKeyHelper`는 `ANTHROPIC_AUTH_TOKEN` 또는 `ANTHROPIC_API_KEY`보다 우선순위가 낮다.

### LiteLLM 구성 예시

```bash
# 통합 엔드포인트 (권장)
export ANTHROPIC_BASE_URL=https://litellm-server:4000

# 정적 API 키
export ANTHROPIC_AUTH_TOKEN=sk-litellm-static-key

# Bedrock 패스스루
export ANTHROPIC_BEDROCK_BASE_URL=https://litellm-server:4000/bedrock
export CLAUDE_CODE_SKIP_BEDROCK_AUTH=1
export CLAUDE_CODE_USE_BEDROCK=1

# Vertex AI 패스스루
export ANTHROPIC_VERTEX_BASE_URL=https://litellm-server:4000/vertex_ai/v1
export ANTHROPIC_VERTEX_PROJECT_ID=your-gcp-project-id
export CLAUDE_CODE_SKIP_VERTEX_AUTH=1
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=us-east5

# Claude Platform on AWS 게이트웨이 패스스루
export ANTHROPIC_AWS_BASE_URL=https://litellm-server:4000/anthropic-aws
export ANTHROPIC_AWS_WORKSPACE_ID=wrkspc_01ABCDEFGHIJKLMN
export CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH=1
export CLAUDE_CODE_USE_ANTHROPIC_AWS=1
```

> Claude Desktop 앱은 자체 호스팅 게이트웨이에 대해 자체 구성 키를 사용하는 Cowork on 3P research preview를 통해서도 게이트웨이에 연결할 수 있다.

---

## 5.5 Claude Platform on AWS

Claude Platform on AWS는 AWS 인증, IAM 접근 통제, AWS Marketplace 과금을 사용하는 Anthropic 운영 Claude API다. 요청은 Anthropic API에 직접 도달하므로 동일한 모델과 기능을 동일한 릴리스 일정으로 사용할 수 있다.

### 사전 요구 사항

- AWS Marketplace를 통한 활성 Claude Platform on AWS 구독
- AWS 연결 Anthropic 조직의 workspace와 해당 workspace ID
- Anthropic 서비스 호출 권한이 있는 IAM 보안 주체, 또는 workspace 범위 API 키
- SigV4 인증을 원하는 경우 환경, `~/.aws/credentials`, 또는 연결된 IAM 역할의 AWS 자격 증명 (AWS CLI는 SSO 로그인 플로우에만 필요)

### AWS 자격 증명 구성

두 가지 인증 방법을 지원한다.

| 방법 | 설명 |
| --- | --- |
| Option A: SigV4 | 환경 변수, `~/.aws/credentials`, IAM 역할, AWS SSO 등 표준 AWS 자격 증명 체인으로 SigV4 서명. SSO 자격 증명이 세션 중 만료되면 `settings.json`의 `awsAuthRefresh`에 로그인 명령을 지정해 재실행 후 재시도 |
| Option B: workspace API 키 | 장기 비밀인 workspace API 키를 `ANTHROPIC_AWS_API_KEY`로 설정. `x-api-key` 헤더로 전송되며 SigV4보다 우선 적용 |

```bash
# Option A: SigV4 (SSO)
aws sso login --profile my-profile
export AWS_PROFILE=my-profile

# Option B: workspace API 키
export ANTHROPIC_AWS_API_KEY=sk-ant-xxxxx
```

`awsAuthRefresh` 설정 예시:

```json
{
  "awsAuthRefresh": "aws sso login --profile my-profile"
}
```

### Claude Code 라우팅 구성

```bash
export CLAUDE_CODE_USE_ANTHROPIC_AWS=1
export ANTHROPIC_AWS_WORKSPACE_ID=wrkspc_01ABCDEFGHIJKLMN
export AWS_REGION=us-east-1
```

`ANTHROPIC_AWS_WORKSPACE_ID`는 필수이며 모든 요청에 `anthropic-workspace-id` 헤더로 전송된다. base URL은 `AWS_REGION`에서 `https://aws-external-anthropic.{region}.api.aws`로 자동 계산된다. URL을 직접 재정의하려면 `ANTHROPIC_AWS_BASE_URL`을 설정한다.

Claude Platform on AWS는 AWS 자격 증명이 환경에 있어도 opt-in이다. Bedrock과 Foundry가 제공자 라우팅에서 우선순위를 가지므로, 설정되어 있다면 `CLAUDE_CODE_USE_BEDROCK`과 `CLAUDE_CODE_USE_FOUNDRY`를 해제해야 한다.

### 모델 버전 고정

Claude Platform on AWS는 직접 Claude API와 동일한 모델 ID를 사용한다. 기본 별칭 `fable`, `opus`, `sonnet`, `haiku`는 Claude Code의 내장 기본값으로 해결되며, `ANTHROPIC_DEFAULT_OPUS_MODEL` 없이 `opus` 별칭은 Opus 4.7로 해결된다. 팀 배포 시 새 릴리스가 모두를 한 번에 이동시키지 않도록 모델 ID를 명시적으로 고정하라.

```bash
export ANTHROPIC_DEFAULT_FABLE_MODEL=claude-fable-5
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-7
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-6
export ANTHROPIC_DEFAULT_HAIKU_MODEL=claude-haiku-4-5
```

Prompt caching은 자동으로 활성화된다. 5분 기본값 대신 1시간 캐시 TTL을 요청하려면 `ENABLE_PROMPT_CACHING_1H=1`을 설정한다. API는 1시간 캐시 쓰기를 더 높은 요율로 청구한다.

### Agent SDK

Agent SDK는 CLI와 동일한 환경 변수를 읽으므로, Claude Code 하위 프로세스를 spawn하는 프로그램은 호출 전에 `CLAUDE_CODE_USE_ANTHROPIC_AWS`, `ANTHROPIC_AWS_WORKSPACE_ID`, 그리고 `ANTHROPIC_AWS_API_KEY` 또는 AWS 자격 증명을 export하여 Claude Platform on AWS를 대상으로 지정할 수 있다.

### 프록시 라우팅

`ANTHROPIC_AWS_BASE_URL`을 프록시 주소로 설정해 트래픽을 라우팅한다. 게이트웨이가 자체적으로 요청에 서명하는 경우 `CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH=1`을 설정해 Claude Code가 미서명 요청을 보내게 하고, 게이트웨이가 자체 토큰을 요구하면 `ANTHROPIC_AUTH_TOKEN`에 설정한다.

### /status 트러블슈팅

`/status`를 실행하여 해결된 제공자와 명시적으로 구성된 workspace ID, region, base URL 재정의, auth-skip 설정을 확인한다.

| 증상 | 원인 및 해결 |
| --- | --- |
| 모든 요청에서 `403 Forbidden` 또는 `AccessDenied` | IAM 보안 주체에 workspace의 Anthropic 서비스 호출 권한이 없음. 역할 확인. `ANTHROPIC_AWS_API_KEY`를 설정한 경우 키가 SigV4보다 우선하므로 만료 키가 동일한 에러를 유발 — 키 재생성 또는 변수 해제 |
| missing-workspace 에러 | `ANTHROPIC_AWS_WORKSPACE_ID`가 unset/empty. AWS 자격 증명으로부터 추론되지 않음. AWS Console의 Workspaces에서 ID 확인 후 export |
| 요청이 여전히 `api.anthropic.com`으로 전송 | `CLAUDE_CODE_USE_ANTHROPIC_AWS`가 unset이거나 truthy로 파싱되지 않음. `1`로 설정 후 `/status`로 확인. `CLAUDE_CODE_USE_BEDROCK` 또는 `CLAUDE_CODE_USE_FOUNDRY`도 설정된 경우 Claude Platform on AWS보다 우선 |

---

## 6. 네트워크 설정

Claude Code는 환경 변수를 통해 다양한 엔터프라이즈 네트워크 및 보안 구성을 지원한다.

### 프록시 구성

```bash
# HTTPS 프록시 (권장)
export HTTPS_PROXY=https://proxy.example.com:8080

# HTTP 프록시
export HTTP_PROXY=http://proxy.example.com:8080

# 프록시 우회 — 공백 또는 콤마 구분
export NO_PROXY="localhost 192.168.1.1 example.com .example.com"

# 모든 요청 우회
export NO_PROXY="*"

# 기본 인증
export HTTPS_PROXY=http://username:password@proxy.example.com:8080
```

### CA 인증서

기본적으로 Claude Code는 번들된 Mozilla CA 인증서와 OS 인증서 저장소를 모두 신뢰한다. `CLAUDE_CODE_CERT_STORE`로 제어한다.

```bash
# 커스텀 CA 인증서
export NODE_EXTRA_CA_CERTS=/path/to/ca-cert.pem

# 번들만 신뢰
export CLAUDE_CODE_CERT_STORE=bundled

# OS 저장소만 신뢰
export CLAUDE_CODE_CERT_STORE=system
```

### mTLS 인증

```bash
export CLAUDE_CODE_CLIENT_CERT=/path/to/client-cert.pem
export CLAUDE_CODE_CLIENT_KEY=/path/to/client-key.pem
export CLAUDE_CODE_CLIENT_KEY_PASSPHRASE="your-passphrase"
```

### 네트워크 접근 요구 사항

| URL | 용도 |
| --- | --- |
| `api.anthropic.com` | Claude API 요청 |
| `claude.ai` | claude.ai 계정 인증 |
| `platform.claude.com` | Anthropic Console 계정 인증 |
| `downloads.claude.ai` | 플러그인 실행 파일 다운로드, 네이티브 설치/자동 업데이트 |
| `storage.googleapis.com` | v2.1.116 이전 버전의 네이티브 설치/자동 업데이트 |
| `bridge.claudeusercontent.com` | Chrome 확장 WebSocket 브릿지 |
| `raw.githubusercontent.com` | `/release-notes` 체인지로그 피드, 플러그인 마켓플레이스 설치 수 |

Bedrock, Vertex AI, Foundry를 사용할 때는 모델 트래픽과 인증이 해당 제공자로 향한다.

---

## 7. 엔터프라이즈 배포 개요

조직은 Anthropic 직접 또는 클라우드 제공자를 통해 Claude Code를 배포할 수 있다.

### 배포 옵션 비교

대부분의 조직에서 Claude for Teams 또는 Claude for Enterprise가 최적의 경험이다. 팀원은 단일 구독으로 Claude Code와 Claude on the Web에 모두 접근할 수 있으며, 중앙 집중식 청구에 인프라 설정이 필요 없다.

| 기능 | Teams/Enterprise | Console | Bedrock | Claude Platform on AWS | Vertex AI | Foundry |
| --- | --- | --- | --- | --- | --- | --- |
| 적합 대상 | 대부분 조직 (권장) | 개인 개발자 | AWS 네이티브 | AWS Marketplace + Claude API | GCP 네이티브 | Azure 네이티브 |
| 과금 | 시트당 / 연락처 | 사용량 기반 | AWS 사용량 기반 | AWS Marketplace 사용량 기반 | GCP 사용량 기반 | Azure 사용량 기반 |
| 인증 | SSO 또는 이메일 | API 키 | API 키 또는 AWS 자격 | API 키 또는 AWS 자격 | GCP 자격 | API 키 또는 Entra ID |
| Claude on web 포함 | 예 | 아니오 | 아니오 | 아니오 | 아니오 | 아니오 |
| 엔터프라이즈 기능 | 팀 관리, SSO, 모니터링 | 없음 | IAM, CloudTrail | IAM, CloudTrail | IAM, Cloud Audit Logs | RBAC, Azure Monitor |

### 프록시 및 게이트웨이 구성

| 유형 | 용도 | 구성 변수 |
| --- | --- | --- |
| 기업 프록시 | 모든 아웃바운드 트래픽을 보안 모니터링/규정 준수를 위해 프록시로 라우팅 | `HTTPS_PROXY`, `HTTP_PROXY` |
| LLM 게이트웨이 | 중앙 집중식 사용량 추적, 커스텀 요금 제한, 인증 관리 | `ANTHROPIC_BASE_URL`, `ANTHROPIC_BEDROCK_BASE_URL`, `ANTHROPIC_AWS_BASE_URL`, `ANTHROPIC_VERTEX_BASE_URL` |

### 모델 버전 고정 권장

Bedrock, Vertex AI, Foundry 또는 Claude Platform on AWS로 배포하는 경우, 특정 모델 버전을 고정하라. 고정하지 않으면 모델 별칭이 최신 버전으로 해결되며, Anthropic 업데이트 시 계정에 아직 활성화되지 않은 버전일 수 있다. Claude Platform on AWS에서는 `ANTHROPIC_DEFAULT_OPUS_MODEL` 없이 `opus` 별칭이 Opus 4.7로 해결된다.

| 환경 변수 | 역할 |
| --- | --- |
| `ANTHROPIC_DEFAULT_FABLE_MODEL` | `fable` 별칭 고정 (예: `claude-fable-5`). 현재 기본값은 opus 별칭 → Opus 4.7 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | `opus` 별칭 고정 (예: `claude-opus-4-7`) |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | `sonnet` 별칭 고정 (예: `claude-sonnet-4-6`) |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | `haiku` 별칭 고정 (예: `claude-haiku-4-5`) |

> Fable 5는 항상 extended thinking을 사용하므로 thinking 비활성화를 사용할 수 없다.

---

## 8. 법률 및 규정 준수

### 라이선스

| 사용자 유형 | 적용 약관 |
| --- | --- |
| Team, Enterprise, Claude API | Commercial Terms |
| Free, Pro, Max | Consumer Terms of Service |

### 상업 계약

Claude API 직접(1P) 또는 Amazon Bedrock/Google Vertex(3P)를 통해 접근하는 경우, 별도 합의가 없는 한 기존 상업 계약이 Claude Code 사용에도 적용된다.

### 헬스케어 규정 준수 (BAA)

BAA(Business Associate Agreement)를 체결한 고객이 Claude Code를 사용하려면, BAA가 체결되어 있고 ZDR(Zero Data Retention)이 활성화된 경우 BAA가 자동으로 Claude Code를 포함하도록 확장된다.

### 인증 및 자격 증명 사용 정책

- **OAuth 인증**: Claude Free, Pro, Max, Team, Enterprise 구독 구매자 전용
- **API 키 인증**: Claude Console 또는 지원 클라우드 제공자를 통한 개발자용. 제3자 개발자가 Claude.ai 로그인을 제공하거나 Free/Pro/Max 플랜 자격 증명을 통해 사용자를 대신해 요청을 라우팅하는 것은 허용되지 않는다.

### 보안 및 신뢰

- Anthropic Trust Center 및 Transparency Hub에서 자세한 정보를 확인할 수 있다
- 보안 취약점은 HackerOne을 통해 관리된다

---

## 9. 데이터 사용

### 데이터 훈련 정책

| 사용자 유형 | 정책 |
| --- | --- |
| 소비자(Free, Pro, Max) | 데이터 사용 설정이 켜져 있을 때 모델 훈련에 사용 |
| 상업(Team, Enterprise, API, 타사 플랫폼) | 코드나 프롬프트를 모델 훈련에 사용하지 않음 (별도 옵트인 제외) |

### 데이터 보존

| 사용자 유형 | 보존 기간 |
| --- | --- |
| 소비자 + 데이터 사용 허용 | 5년 |
| 소비자 + 데이터 사용 거부 | 30일 |
| 상업 표준 | 30일 |
| 상업 ZDR | 요청 완료 후 서버 측 보존 없음 |
| 로컬 캐시 | `~/.claude/projects/`에 30일간 일반 텍스트로 저장 (세션 재개용). `cleanupPeriodDays`로 조정 가능 |

### 암호화

| 제공자 | 저장 시 암호화 |
| --- | --- |
| Anthropic API | 인프라 수준 디스크 암호화(AES-256). ZDR 활성화 시 서버 측 지속성 없음 |
| Amazon Bedrock | AWS 관리 키로 AES-256. AWS KMS를 통해 고객 관리 키 사용 가능 |
| Google Vertex AI | Google 관리 암호화 키. CMEK 사용 가능 |
| Microsoft Foundry | Anthropic 인프라로 라우팅, AES-256 디스크 암호화 |

### 원격 측정(Telemetry) 서비스

| 서비스 | 기본 동작 (Claude API) | 비활성화 환경 변수 |
| --- | --- | --- |
| 운영 메트릭 (Anthropic) | 활성 | `DISABLE_TELEMETRY=1` |
| 오류 로깅 (Sentry) | 활성 | `DISABLE_ERROR_REPORTING=1` |
| `/feedback` 명령 | 활성 | `DISABLE_FEEDBACK_COMMAND=1` |
| 세션 품질 설문조사 | 활성 | `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY=1` |

Bedrock, Vertex, Foundry, Claude Platform on AWS를 사용할 때는 오류 보고, 원격 측정, 버그 보고가 기본적으로 비활성화된다. `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`으로 모든 비필수 트래픽을 한번에 비활성화할 수 있다.

### WebFetch 도메인 안전 확인

WebFetch 도구는 URL을 가져오기 전에 요청된 호스트명을 `api.anthropic.com`으로 보내 안전 차단 목록을 확인한다. 사용 중인 모델 제공자와 관계없이 실행되며, `skipWebFetchPreflight: true`로 비활성화할 수 있다.

---

## 10. 제로 데이터 보존 (ZDR)

ZDR은 Claude for Enterprise를 통해 Claude Code에 사용할 수 있다. ZDR이 활성화되면 Claude Code 세션 중 생성된 프롬프트와 모델 응답은 실시간으로 처리되며, 법적 준수나 오용 방지에 필요한 경우를 제외하고 Anthropic에 저장되지 않는다.

### ZDR 범위

**포함되는 항목**: Claude for Enterprise의 Claude Code 모델 추론 호출. 사용하는 Claude 모델에 관계없이 적용.

**포함되지 않는 항목**:

| 항목 | 상세 |
| --- | --- |
| claude.ai 채팅 | 웹 인터페이스 대화는 ZDR 미적용 |
| Cowork | Cowork 세션은 ZDR 미적용 |
| Claude Code Analytics | 프롬프트나 모델 응답을 저장하지 않으나, 계정 이메일과 사용 통계 등 생산성 메타데이터 수집. 기여 메트릭 사용 불가 |
| 사용자/시트 관리 | 관리 데이터는 표준 정책에 따라 보존 |
| 타사 통합 | MCP 서버 등 외부 도구에서 처리되는 데이터는 ZDR 범위 외 |

### ZDR 활성화 시 비활성화되는 기능

| 기능 | 사유 |
| --- | --- |
| Claude Code on the Web | 대화 기록의 서버 측 저장 필요 |
| Desktop 앱의 원격 세션 | 프롬프트와 응답을 포함하는 지속적 세션 데이터 필요 |
| 피드백 제출 (`/feedback`) | 대화 데이터를 Anthropic에 전송 |

### 정책 위반 데이터 보존

ZDR이 활성화되어도 Anthropic은 법적 요구나 사용 정책 위반을 해결하기 위해 데이터를 보존할 수 있다. 정책 위반으로 플래그된 세션의 입력과 출력은 최대 2년간 보존될 수 있다.

### ZDR 요청

Claude for Enterprise에서 ZDR을 요청하려면 영업팀이나 Anthropic 계정 팀에 연락한다. 계정 팀이 내부적으로 요청을 제출하고, Anthropic이 자격을 확인한 후 조직에 ZDR을 활성화한다.

---

## 11. 비용

Claude Code는 API 토큰 소비량에 따라 요금이 청구된다. 구독 플랜 가격은 claude.com/pricing을 참조한다.

### 비용 개요

| 지표 | 값 |
| --- | --- |
| 개발자당 일평균 비용 | 약 $13 |
| 개발자당 월평균 비용 | $150-250 |
| 90% 사용자 일일 상한 | $30 미만 |

### 비용 추적

- `/usage` 명령: 현재 세션의 토큰 사용량, 비용 추정치, 기간 표시
- Claude Console의 Usage 페이지에서 공식 청구 확인 가능
- Pro, Max, Team, Enterprise에서 `/usage`는 스킬, 하위 에이전트, 플러그인, MCP 서버별 사용량 분석도 제공. `d` 또는 `w`를 눌러 최근 24시간과 최근 7일 간 전환. 수치는 이 장비의 로컬 세션 기록에서 근사 계산되므로 다른 장치나 claude.ai의 사용량은 포함되지 않는다.
- VS Code 확장에서는 동일한 분석이 Account & usage 다이얼로그의 Day/Week 토글로 표시된다. Claude Code v2.1.174 이상 필요.

### 팀 비용 관리

- Claude API: 워크스페이스 지출 한도 설정 가능
- Pro 및 Max: `/usage-credits` 명령으로 월간 지출 한도 설정
- Bedrock, Vertex, Foundry: LiteLLM 등 서드파티 도구로 키별 지출 추적

### 권장 요금 제한

| 팀 규모 | 사용자당 TPM | 사용자당 RPM |
| --- | --- | --- |
| 1-5 | 200k-300k | 5-7 |
| 5-20 | 100k-150k | 2.5-3.5 |
| 20-50 | 50k-75k | 1.25-1.75 |
| 50-100 | 25k-35k | 0.62-0.87 |
| 100-500 | 15k-20k | 0.37-0.47 |
| 500+ | 10k-15k | 0.25-0.35 |

### 토큰 사용량 절감 전략

| 전략 | 방법 |
| --- | --- |
| 컨텍스트 관리 | 작업 전환 시 `/clear` 사용, `/compact`로 맞춤 압축 지침 제공. 세션을 나중에 쉽게 찾도록 `/rename`으로 이름 지정 후 clear하고 `/resume`으로 복귀 |
| 모델 선택 | 대부분 Sonnet으로 충분. Opus는 복잡한 아키텍처 결정에 예약. `/model`로 중간 전환. 단순 하위 에이전트 작업에는 subagent 구성에서 `model: haiku` 지정 |
| MCP 서버 최적화 | 사용하지 않는 서버 비활성화 (`/mcp`), CLI 도구 우선 |
| 타입 언어 코드 인텔리전스 플러그인 | LSP 기반 정밀 심볼 네비게이션(go-to-definition 등)으로 불필요한 파일 읽기를 줄임. 단일 go-to-definition 호출이 grep + 후보 파일 다수 읽기를 대체. 설치된 언어 서버가 편집 후 타입 에러도 자동 보고 |
| 훅 및 스킬 활용 | 대용량 파일 전처리, 도메인 지식 온디맨드 로드 |
| CLAUDE.md 최적화 | 200줄 미만 유지, 전문 지침은 스킬로 이동 |
| Extended Thinking 조정 | `/effort`로 노력 수준 조정, `MAX_THINKING_TOKENS`로 예산 제한. 단, Fable 5는 항상 extended thinking을 사용하므로 비활성화 불가 |
| 하위 에이전트에 위임 | 장황한 출력은 하위 에이전트 컨텍스트에 격리 |
| 구체적인 프롬프트 | "improve this codebase"보다 "auth.ts의 login 함수에 입력 검증 추가"가 효율적 |
| 복잡 작업 효율적 수행 | Shift+Tab으로 plan 모드 진입 후 구현, 잘못된 방향이면 Escape로 즉시 중단, `/rewind` 또는 Escape 두 번 눌러 체크포인트 복원, 검증 타겟(테스트 케이스·스크린샷·예상 출력) 제공, 파일 하나 작성 후 테스트하는 증분 접근 |

### 에이전트 팀 토큰 비용

에이전트 팀은 여러 Claude Code 인스턴스를 생성하며, 각각 자체 컨텍스트 윈도우를 갖는다. 토큰 사용량은 활성 팀원 수와 각 실행 시간에 비례한다. 팀원이 plan 모드로 실행될 때 표준 세션의 약 7배 토큰을 사용한다.

비용 관리를 위한 권장 사항:

- 팀원에 Sonnet 사용. 조정 작업에 적합한 성능-비용 균형
- 팀 규모를 작게 유지. 각 팀원이 자체 컨텍스트를 실행하므로 토큰은 팀 규모에 비례
- spawn 프롬프트를 집중적으로 유지. 팀원은 CLAUDE.md, MCP 서버, 스킬을 자동 로드하지만 spawn 프롬프트의 모든 내용이 초기 컨텍스트에 추가됨
- 작업 완료 시 팀 정리. 활성 팀원은 유휴 상태에서도 토큰을 계속 소비
- 에이전트 팀은 기본적으로 비활성화됨. settings.json 또는 환경에서 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 설정 필요

### 백그라운드 토큰 사용량

Claude Code는 유휴 상태에서도 일부 백그라운드 기능에 토큰을 사용한다.

- **대화 요약**: `claude --resume` 기능을 위한 이전 대화 요약 백그라운드 작업
- **명령 처리**: `/usage` 등 일부 명령은 상태 확인을 위해 요청을 생성할 수 있음

이러한 백그라운드 프로세스는 활성 상호작용 없이도 세션당 약 $0.04 미만의 적은 토큰을 소비한다.

### Claude Code 동작 변경 이해

Claude Code는 비용 보고를 포함하여 기능 작동 방식을 변경할 수 있는 업데이트를 정기적으로 수신한다. 현재 버전은 `claude --version`으로 확인한다. 특정 청구 관련 질의는 Console 계정을 통해 Anthropic 지원에 문의하라.

---

## 12. 분석

Claude Code는 조직이 개발자 사용 패턴을 이해하고, 기여 메트릭을 추적하며, Claude Code가 엔지니어링 속도에 미치는 영향을 측정할 수 있는 분석 대시보드를 제공한다.

### 대시보드 접근

| 플랜 | 대시보드 URL | 포함 내용 | 접근 권한 |
| --- | --- | --- | --- |
| Claude for Teams / Enterprise | claude.ai/analytics/claude-code | 사용량 메트릭, 기여 메트릭(GitHub 연동), 리더보드, 데이터 내보내기 | Admin, Owner |
| API (Claude Console) | platform.claude.com/claude-code | 사용량 메트릭, 지출 추적, 팀 인사이트 | `UsageView` 권한 필요 (Developer, Billing, Admin, Owner, Primary Owner 역할에 부여) |

### 요약 메트릭

| 메트릭 | 설명 |
| --- | --- |
| PRs with CC | Claude Code로 작성된 코드가 1줄 이상 포함된 병합된 PR 총수 |
| Lines of code with CC | Claude Code 지원으로 작성된 코드의 총 줄 수. 정규화 후 3자 이상이고 빈 줄·괄호/단순 구두점만 있는 줄을 제외한 "effective lines"만 계산 |
| PRs with Claude Code (%) | 전체 병합 PR 중 Claude Code 지원 코드가 포함된 PR의 비율 |
| Suggestion accept rate | Edit, Write, NotebookEdit 도구 사용에 대한 수락률 |
| Lines of code accepted | Claude Code가 작성하고 사용자가 수락한 총 코드 줄 수. 거부된 제안은 제외되며 이후 삭제는 추적하지 않음 |

### 기여 메트릭 활성화

기여 메트릭은 GitHub 조직 연결이 필요하다. Owner 역할이 필요하며, GitHub 관리자가 GitHub 앱을 설치해야 한다. 데이터는 활성화 후 24시간 이내에 나타난다. GitHub Cloud 및 GitHub Enterprise Server를 지원한다.

### PR 속성(Attribution)

1. 병합된 PR에서 추가된 줄을 추출
2. 해당 파일을 편집한 Claude Code 세션을 시간 윈도우 내에서 식별 (병합일 기준 21일 전 ~ 2일 후)
3. 다중 전략으로 PR 줄과 Claude Code 출력을 매칭
4. AI 지원 줄 수와 총 줄 수로 메트릭 계산

비교 전 줄은 정규화된다: 공백을 trim하고, 다중 공백을 축소하며, 따옴표를 표준화하고, 소문자로 변환한다. Claude Code 지원 줄이 포함된 병합 PR은 GitHub에서 `claude-code-assisted` 라벨로 표시된다.

**제외 파일**: lock 파일, 생성된 코드, 빌드 디렉토리, 테스트 픽스처, 1,000자 이상 줄

**속성 참고 사항**:

- 개발자가 20% 이상 차이로 코드를 실질적으로 재작성한 경우 Claude Code로 귀속되지 않는다
- 21일 윈도우 외부의 세션은 고려되지 않는다
- 알고리즘은 속성 수행 시 PR 소스/대상 브랜치를 고려하지 않는다

### API 고객 분석

Console 대시보드에는 다음이 포함된다.

- **Lines of code accepted**: 수락된 코드 줄 수
- **Suggestion accept rate**: 편집 도구 수락률
- **Activity**: 일별 활성 사용자 및 세션 차트
- **Spend**: 일별 API 비용(달러)과 사용자 수

팀 인사이트 테이블에서 사용자별 메트릭을 확인할 수 있다.

---

## 13. 사용량 모니터링 (OpenTelemetry)

Claude Code는 OpenTelemetry(OTel)를 통해 원격 측정 데이터를 내보내어 조직 전반의 사용량, 비용, 도구 활동을 추적할 수 있다.

### 빠른 시작

```bash
# 1. 원격 측정 활성화
export CLAUDE_CODE_ENABLE_TELEMETRY=1

# 2. 익스포터 선택
export OTEL_METRICS_EXPORTER=otlp       # otlp, prometheus, console, none
export OTEL_LOGS_EXPORTER=otlp          # otlp, console, none

# 3. OTLP 엔드포인트 구성
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317

# 4. 인증 (필요 시)
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer your-token"
```

### 관리자 구성 예시

```json
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.example.com:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer example-token"
  }
}
```

### 주요 환경 변수

| 변수 | 설명 | 예시 |
| --- | --- | --- |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | 원격 측정 활성화 (필수) | `1` |
| `OTEL_METRICS_EXPORTER` | 메트릭 익스포터 | `console`, `otlp`, `prometheus`, `none` |
| `OTEL_LOGS_EXPORTER` | 로그/이벤트 익스포터 | `console`, `otlp`, `none` |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | OTLP 프로토콜 | `grpc`, `http/json`, `http/protobuf` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | OTLP 수집기 엔드포인트 | `http://localhost:4317` |
| `OTEL_EXPORTER_OTLP_HEADERS` | 인증 헤더 | `Authorization=Bearer token` |
| `OTEL_LOG_USER_PROMPTS` | 사용자 프롬프트 내용 로깅 | `1` |
| `OTEL_LOG_TOOL_DETAILS` | 도구 매개변수 및 MCP 서버/도구 이름 로깅 | `1` |
| `OTEL_LOG_TOOL_CONTENT` | 도구 입출력 내용 로깅 (추적 필요, 60KB에서 잘림) | `1` |
| `OTEL_LOG_RAW_API_BODIES` | 전체 API 요청/응답 JSON 로깅. `=1`은 60KB 잘림 인라인, `=file:<dir>`은 디스크에 무잘림 저장 후 `body_ref` 포인터 | `1` 또는 `file:<dir>` |
| `OTEL_METRIC_EXPORT_INTERVAL` | 메트릭 내보내기 간격(ms, 기본 60000) | `5000` |
| `OTEL_LOGS_EXPORT_INTERVAL` | 로그 내보내기 간격(ms, 기본 5000) | `10000` |
| `OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE` | 메트릭 시간성 기본 설정 (기본 `delta`). 백엔드가 누적을 기대하면 `cumulative`로 설정 | `delta`, `cumulative` |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | 동적 헤더 갱신 간격 (기본 1740000ms / 29분) | `900000` |

### mTLS 인증

OTLP 익스포터의 클라이언트 인증서 구성은 해당 신호에 사용 중인 OTLP 프로토콜에 따라 다르다.

| 프로토콜 | 클라이언트 인증서 변수 | CA 신뢰 |
| --- | --- | --- |
| `http/protobuf`, `http/json` | `CLAUDE_CODE_CLIENT_CERT`, `CLAUDE_CODE_CLIENT_KEY`, `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | `NODE_EXTRA_CA_CERTS` |
| `grpc` | `OTEL_EXPORTER_OTLP_CLIENT_KEY`, `OTEL_EXPORTER_OTLP_CLIENT_CERTIFICATE` (또는 신호별 변형) | `OTEL_EXPORTER_OTLP_CERTIFICATE` |

### 메트릭 카디널리티 제어

| 변수 | 설명 | 기본값 |
| --- | --- | --- |
| `OTEL_METRICS_INCLUDE_SESSION_ID` | `session.id` 속성 포함 | `true` |
| `OTEL_METRICS_INCLUDE_VERSION` | `app.version` 속성 포함 | `false` |
| `OTEL_METRICS_INCLUDE_ACCOUNT_UUID` | `user.account_uuid` 및 `user.account_id` 속성 포함 | `true` |
| `OTEL_METRICS_INCLUDE_ENTRYPOINT` | `app.entrypoint` 속성 포함 | `false` |
| `OTEL_METRICS_INCLUDE_RESOURCE_ATTRIBUTES` | `OTEL_RESOURCE_ATTRIBUTES`의 키를 metric datapoint 속성으로 포함 | `true` |

### 다중 팀 조직 지원

여러 팀이나 부서가 있는 조직은 `OTEL_RESOURCE_ATTRIBUTES` 환경 변수로 커스텀 속성을 추가해 그룹을 구분할 수 있다.

```bash
export OTEL_RESOURCE_ATTRIBUTES="department=engineering,team.id=platform,cost_center=eng-123"
```

이 커스텀 속성은 모든 메트릭과 이벤트에 포함되어 팀/부서별 필터링, 비용 센터별 비용 추적, 팀별 대시보드와 알림을 지원한다. Claude Code는 이 값을 OTLP resource 블록으로 전송하는 것과 별도로 모든 metric datapoint와 event record에 속성으로 첨부한다. 커스텀 키는 표준 속성(`user.id`, `session.id` 등)을 덮어쓰지 않으며, 충돌 시 내장 값을 유지한다. resource 블록에만 전송하고 datapoint 라벨에서 생략하려면 `OTEL_METRICS_INCLUDE_RESOURCE_ATTRIBUTES=false`를 설정한다.

### 동적 헤더 헬퍼

엔터프라이즈 환경에서 동적 인증이 필요한 경우, 헤더를 생성하는 스크립트를 구성할 수 있다. `http/protobuf` 및 `http/json` 프로토콜에만 적용된다.

```json
{
  "otelHeadersHelper": "/bin/generate_opentelemetry_headers.sh"
}
```

스크립트는 HTTP 헤더를 나타내는 문자열 키-값 쌍의 유효한 JSON을 출력해야 한다. 기본적으로 29분마다 실행되며, `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS`로 간격 조정이 가능하다.

### 사용 가능한 메트릭

| 메트릭명 | 설명 | 단위 |
| --- | --- | --- |
| `claude_code.session.count` | CLI 세션 시작 수 | count |
| `claude_code.lines_of_code.count` | 수정된 코드 줄 수 | count |
| `claude_code.pull_request.count` | 생성된 PR 수 | count |
| `claude_code.commit.count` | 생성된 git 커밋 수 | count |
| `claude_code.cost.usage` | 세션 비용 | USD |
| `claude_code.token.usage` | 사용된 토큰 수 | tokens |
| `claude_code.code_edit_tool.decision` | 코드 편집 도구 권한 결정 수 | count |
| `claude_code.active_time.total` | 총 활성 시간 | s |

### 주요 이벤트

| 이벤트 | 설명 |
| --- | --- |
| `claude_code.user_prompt` | 사용자 프롬프트 제출 시 |
| `claude_code.tool_result` | 도구 실행 완료 시 |
| `claude_code.api_request` | Claude API 요청 시 |
| `claude_code.api_error` | API 요청 실패 시 |
| `claude_code.api_refusal` | API 요청이 `stop_reason: "refusal"`을 반환할 때. refusal은 성공 응답 스트림에 도달하므로 `api_error` 이벤트가 발생하지 않아 이 이벤트로 빈도를 추적 |
| `claude_code.api_request_body` | `OTEL_LOG_RAW_API_BODIES` 설정 시 API 요청 본문 (인라인 모드: 60KB 잘림, 파일 모드: `body_ref` 경로) |
| `claude_code.api_response_body` | `OTEL_LOG_RAW_API_BODIES` 설정 시 API 응답 본문 |
| `claude_code.tool_decision` | 도구 권한 결정 시 |
| `claude_code.mcp_server_connection` | MCP 서버 연결/해제/실패 시 |
| `claude_code.auth` | 로그인/로그아웃 시 |
| `claude_code.permission_mode_changed` | 권한 모드 변경 시 |
| `claude_code.plugin_installed` | 플러그인 설치 시 |
| `claude_code.plugin_loaded` | 세션 시작 시 활성 플러그인 |
| `claude_code.skill_activated` | 스킬 호출 시 |
| `claude_code.at_mention` | `@`-멘션 해결 시 |
| `claude_code.api_retries_exhausted` | API 재시도 모두 소진 시 |
| `claude_code.hook_registered` | 세션 시작 시 구성된 훅 |
| `claude_code.hook_execution_start` | 훅 실행 시작 시 |
| `claude_code.hook_execution_complete` | 훅 실행 완료 시 |
| `claude_code.hook_plugin_metrics` | 공식 마켓플레이스 플러그인 훅 메트릭 |
| `claude_code.compaction` | 대화 압축 완료 시 |
| `claude_code.feedback_survey` | 세션 품질 설문조사 표시/응답 시 |
| `claude_code.internal_error` | 예기치 않은 내부 오류 포착 시 |

### 분산 추적 (베타)

추적을 활성화하려면 `CLAUDE_CODE_ENABLE_TELEMETRY=1`과 `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`을 모두 설정하고, `OTEL_TRACES_EXPORTER`를 구성한다.

스팬 계층 구조:

```
claude_code.interaction
  +-- claude_code.llm_request
  +-- claude_code.hook
  +-- claude_code.tool
      +-- claude_code.tool.blocked_on_user
      +-- claude_code.tool.execution
      +-- (Agent tool) subagent claude_code.llm_request / claude_code.tool spans
```

`llm_request`, `tool.execution`, `hook` 스팬은 실패 기록 시 OpenTelemetry 상태 `ERROR`를 설정한다. 기본적으로 사용자 프롬프트 텍스트, 도구 입력 세부 정보, 도구 내용은 제거(redact)된다. 포함하려면 `OTEL_LOG_USER_PROMPTS=1`, `OTEL_LOG_TOOL_DETAILS=1`, `OTEL_LOG_TOOL_CONTENT=1`을 설정하라.

`claude_code.hook` 스팬은 detailed beta tracing이 활성일 때만 내보내지며, 추적 익스포터 구성에 더해 `ENABLE_BETA_TRACING_DETAILED=1`과 `BETA_TRACING_ENDPOINT`가 필요하다. 대화형 CLI 세션에서는 조직이 이 기능에 대해 allowlist에 등록되어 있어야 한다. Agent SDK 및 비대화형 `-p` 세션은 이 게이트가 없다. `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA`만 설정된 경우에는 발생하지 않는다.

### 분산 추적 TRACEPARENT 전파

추적 활성 시 Bash 및 PowerShell 하위 프로세스는 활성 도구 실행 스팬의 W3C trace context를 포함하는 `TRACEPARENT` 환경 변수를 자동으로 상속한다. Agent SDK 및 `-p` 비대화형 세션에서는 `TRACEPARENT` 및 `TRACESTATE`를 자체 환경에서 읽어 외부 프로세스의 분산 추적을 Claude Code 스팬 아래에 배치할 수 있다. 대화형 세션은 CI 또는 컨테이너 환경의 앰비언트 값을 우연히 상속하지 않도록 인바운드 `TRACEPARENT`를 무시한다.

Claude Code가 Anthropic API에 직접 연결된 경우, 각 모델 요청은 `claude_code.llm_request` 스팬 컨텍스트로 설정된 W3C `traceparent` 헤더를 전송하며, API의 `traceresponse` 헤더는 span link로 기록된다. 이 둘이 함께 Claude Code의 클라이언트 측 스팬을 규정 준수 매개체를 통한 서버 측 추적과 연결한다. 아웃바운드 HTTP MCP 요청도 동일하게 `traceparent`를 전송한다. 헤더는 서드파티 제공자에게는 전송되지 않는다.

기본적으로 모델 및 HTTP MCP 요청의 `traceparent` 헤더는 `ANTHROPIC_BASE_URL`이 unset이거나 Anthropic API를 가리킬 때만 전송된다(일부 프록시가 인식하지 못하는 헤더를 거부하기 때문). 하위 프로세스 `TRACEPARENT` 변수도 동일한 스위치로 제어된다. 커스텀 `ANTHROPIC_BASE_URL` 프록시로 Claude Code를 실행하면서 trace context를 전파하려면 `CLAUDE_CODE_PROPAGATE_TRACEPARENT=1`을 설정한다.

### 권한 모드 변경 이벤트 상세

`permission_mode_changed` 이벤트는 다음 속성을 포함한다:

- `from_mode`: 이전 권한 모드 (예: `default`, `plan`, `acceptEdits`, `auto`, `bypassPermissions`)
- `to_mode`: 새 권한 모드
- `trigger`: 변경 원인. `shift_tab`, `exit_plan_mode`, `auto_gate_denied`, `auto_opt_in` 중 하나. SDK/bridge에서 발생한 전환 시 absent

### 비용/토큰 카운터 속성

`claude_code.cost.usage`와 `claude_code.token.usage` 메트릭은 다음 컨텍스트 속성을 포함한다:

| 속성 | 설명 |
| --- | --- |
| `effort` | 요청에 적용된 노력 수준: `low`, `medium`, `high`, `xhigh`, `max`. 모델이 effort를 지원하지 않으면 absent |
| `agent.name` | 요청을 발행한 하위 에이전트 유형. 내장 에이전트명과 공식 마켓플레이스 플러그인 에이전트명은 그대로 표시. 그 외 사용자 정의는 `custom`으로 대체 |
| `skill.name` | 요청에 활성인 스킬. 내장/번들/사용자 정의/공식 마켓플레이스 스킬명은 그대로, 서드파티 플러그인 스킬은 `third-party`로 대체 |
| `plugin.name` | 활성 스킬/하위 에이전트를 제공하는 플러그인. 공식 마켓플레이스는 그대로, 서드파티는 `third-party` |
| `marketplace.name` | 소유 플러그인이 설치된 마켓플레이스. 공식 마켓플레이스 플러그인만 해당 |
| `mcp_server.name` | 해당 턴에 MCP 도구를 실행한 서버. 내장/claude.ai 프록시/공식 레지스트리는 그대로, 사용자 구성은 `custom` |
| `mcp_tool.name` | 해당 턴에 실행된 MCP 도구. `mcp_server.name`과 동일한 마스킹 적용 |

### 플러그인/훅 이벤트 속성

`plugin_loaded`, `hook_registered`, `hook_execution_start`, `hook_execution_complete` 이벤트는 다음 속성을 포함한다:

| 속성 | 설명 |
| --- | --- |
| `safe_mode` | 세션이 `--safe-mode`로 시작된 경우 `true`, 그 외 `false`. Claude Code v2.1.169 이상 필요 |
| `host_owned_mcp` | SDK 호스트가 플러그인의 MCP 연결을 관리하고 Claude Code가 플러그인의 MCP 서버 구성 읽기를 건너뛴 경우 `true`. v2.1.172 이상 필요 (plugin_loaded만) |

`skill_activated` 이벤트는 워크플로 스킬의 경우 `skill.kind: "workflow"` 속성을 포함한다(해당하지 않으면 absent).

### 보안 감사

OpenTelemetry 이벤트는 Claude Code 활동에 대한 감사 데이터 소스이다. 모든 이벤트는 도구 호출, MCP 활동, 권한 결정을 해당 사용자에게 연결하는 ID 속성을 포함한다.

SIEM으로 이벤트를 전송하려면 `OTEL_EXPORTER_OTLP_LOGS_ENDPOINT`를 SIEM의 OTLP 수신기 또는 OpenTelemetry Collector로 설정한다.

```json
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_LOG_TOOL_DETAILS": "1",
    "OTEL_EXPORTER_OTLP_LOGS_PROTOCOL": "http/protobuf",
    "OTEL_EXPORTER_OTLP_LOGS_ENDPOINT": "https://siem.example.com:4318/v1/logs",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer your-siem-token"
  }
}
```

### 보안 및 개인정보 보호

- OpenTelemetry 내보내기는 옵트인이며 명시적 구성이 필요하다. Anthropic의 별도 운영 원격 측정 및 비활성화 방법은 Data usage 참조
- 원시 파일 내용과 코드 스니펫은 메트릭이나 이벤트에 포함되지 않는다. 추적 스팬은 별도 데이터 경로이며 아래 `OTEL_LOG_TOOL_CONTENT` 참조
- OAuth 인증 시 `user.email`이 원격 측정 속성에 포함된다. 조직에서 우려되는 경우 원격 측정 백엔드에서 해당 필드를 필터링 또는 수정하라
- 사용자 프롬프트 내용은 기본적으로 수집되지 않는다. 프롬프트 길이만 기록. 포함하려면 `OTEL_LOG_USER_PROMPTS=1` 설정
- 도구 입력 인수 및 매개변수는 기본적으로 로깅되지 않는다. 포함하려면 `OTEL_LOG_TOOL_DETAILS=1` 설정. 이 데이터는 구성한 OTEL 엔드포인트로만 전송되며 Anthropic에는 전송되지 않는다
- **`OTEL_LOG_TOOL_CONTENT`**: 추적 스팬에서 도구 입출력 내용 기본 비활성화. `=1` 설정 시 스팬 이벤트에 스팬당 60KB 잘림으로 전체 도구 입출력 포함. Read 도구 결과의 원시 파일 내용과 Bash 명령 출력이 포함될 수 있으므로 원격 측정 백엔드에서 필터링 구성 필요
- **`OTEL_LOG_RAW_API_BODIES`**: Anthropic Messages API 요청/응답 본문 기본 비활성화. `=1`은 60KB 잘림 인라인, `=file:<dir>`은 디스크에 무잘림 저장. 두 모드 모두 전체 대화 기록(시스템 프롬프트, 모든 사용자/어시스턴트 턴, 도구 결과)이 포함되므로, 다른 `OTEL_LOG_*` 콘텐츠 플래그가 공개하는 모든 것에 동의하는 것임. Claude의 확장 사고 내용은 다른 설정과 관계없이 항상 제거됨

### ROI 측정 자료

Claude Code의 투자 대비 수익(ROI) 측정에 대한 포괄적인 가이드(원격 측정 설정, 비용 분석, 생산성 메트릭, 자동화된 보고 포함)는 Claude Code ROI Measurement Guide를 참조하라. 이 저장소는 Docker Compose 구성, Prometheus 및 OpenTelemetry 설정, Linear 등 도구와 통합된 생산성 보고서 생성 템플릿을 제공한다.

### Amazon Bedrock에서 Claude Code 모니터링

Amazon Bedrock에서 Claude Code 사용량 모니터링에 대한 자세한 지침은 Claude Code Monitoring Implementation (Bedrock)을 참조하라.

### 백엔드 선택

| 유형 | 추천 백엔드 |
| --- | --- |
| 메트릭 | Prometheus(시계열), ClickHouse(컬럼형), Honeycomb/Datadog(통합 플랫폼) |
| 이벤트/로그 | Elasticsearch/Loki(로그 집계), ClickHouse(구조화 분석) |
| 추적 | Jaeger, Zipkin, Grafana Tempo, Honeycomb/Datadog |
