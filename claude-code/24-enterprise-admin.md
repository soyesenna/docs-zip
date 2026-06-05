# Claude Code 엔터프라이즈 관리

> 기업 환경 설정, 관리형 정책, 비용/사용량 모니터링, 규정 준수

**원문**: https://code.claude.com/docs/en/admin-setup (및 관련 페이지)

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

일부 Claude Code 기능은 Claude.ai 계정이 필요하다. Claude Code on the Web, Routines, Code Review, Remote Control, Chrome 확장은 Console API 키나 클라우드 제공자 자격 증명만으로는 사용할 수 없다.

### 설정 전달 방식

관리형 설정은 로컬 개발자 구성보다 우선하며, Claude Code는 다음 네 가지 위치에서 설정을 찾아 장치에서 처음 발견한 것을 사용한다.

| 메커니즘 | 전달 방식 | 우선순위 | 플랫폼 |
| --- | --- | --- | --- |
| 서버 관리형 | Claude.ai 관리자 콘솔 | 최고 | 전체 |
| plist / 레지스트리 정책 | macOS: `com.anthropic.claudecode` plist, Windows: `HKLM\SOFTWARE\Policies\ClaudeCode` | 높음 | macOS, Windows |
| 파일 기반 관리 | macOS: `/Library/Application Support/ClaudeCode/managed-settings.json`, Linux/WSL: `/etc/claude-code/managed-settings.json`, Windows: `C:\Program Files\ClaudeCode\managed-settings.json` | 중간 | 전체 |
| Windows 사용자 레지스트리 | `HKCU\SOFTWARE\Policies\ClaudeCode` | 최저 | Windows만 |

서버 관리형 설정은 인증 시점에 장치에 도달하며 활성 세션 중 매시간 새로고침된다. Claude for Teams 또는 Enterprise 플랜이 필요하다.

### 강제 가능한 제어 항목

| 제어 | 역할 | 키 설정 |
| --- | --- | --- |
| 권한 규칙 | 특정 도구·명령 허용/질문/거부 | `permissions.allow`, `permissions.deny` |
| 권한 잠금 | 관리형 권한 규칙만 적용, `--dangerously-skip-permissions` 비활성화 | `allowManagedPermissionRulesOnly`, `permissions.disableBypassPermissionsMode` |
| 샌드박싱 | OS 수준 파일시스템·네트워크 격리 + 도메인 허용 목록 | `sandbox.enabled`, `sandbox.network.allowedDomains` |
| 관리형 정책 CLAUDE.md | 모든 세션에 로드되는 조직 전체 지침 | 관리형 정책 경로의 파일 |
| MCP 서버 제어 | 서버 allowlist/denylist 또는 고정 서버 세트 배포 | `allowedMcpServers`, `deniedMcpServers`, `allowManagedMcpServersOnly` |
| 플러그인 마켓플레이스 제어 | 마켓플레이스 소스 제한 | `strictKnownMarketplaces`, `blockedMarketplaces` |
| 커스터마이징 잠금 | 스킬·에이전트·훅·MCP를 플러그인 또는 관리형 설정에서만 로드 | `strictPluginOnlyCustomization` |
| 훅 제한 | 관리형 훅만 로드, HTTP 훅 URL 제한 | `allowManagedHooksOnly`, `allowedHttpHookUrls` |
| 에이전트 뷰 비활성화 | `claude agents`, `--bg`, `/background` 비활성화 | `disableAgentView` |
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

### 가져오기 및 캐싱 동작

- **첫 실행(캐시 없음)**: 설정을 비동기로 가져온다. 가져오기 실패 시 관리형 설정 없이 계속 진행
- **후속 실행(캐시 있음)**: 캐시된 설정이 즉시 적용되고, 백그라운드에서 새 설정을 가져온다

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

### 모델 검색

`CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`을 설정하면 Claude Code가 게이트웨이의 `/v1/models` 엔드포인트를 쿼리하여 `/model` 선택기에 모델을 추가한다. v2.1.129 이상 필요. Anthropic Messages 형식에만 적용된다.

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
```

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

## 7. 법률 및 규정 준수

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

## 8. 데이터 사용

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

## 9. 제로 데이터 보존 (ZDR)

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

## 10. 비용

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
- Pro, Max, Team, Enterprise에서 `/usage`는 스킬, 하위 에이전트, 플러그인, MCP 서버별 사용량 분석도 제공

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
| 컨텍스트 관리 | 작업 전환 시 `/clear` 사용, `/compact`로 맞춤 압축 지침 제공 |
| 모델 선택 | 대부분 Sonnet으로 충분. Opus는 복잡한 아키텍처 결정에 예약. `/model`로 중간 전환 |
| MCP 서버 최적화 | 사용하지 않는 서버 비활성화 (`/mcp`), CLI 도구 우선 |
| 훅 및 스킬 활용 | 대용량 파일 전처리, 도메인 지식 온디맨드 로드 |
| CLAUDE.md 최적화 | 200줄 미만 유지, 전문 지침은 스킬로 이동 |
| Extended Thinking 조정 | `/effort`로 노력 수준 조정, `MAX_THINKING_TOKENS`로 예산 제한 |
| 하위 에이전트에 위임 | 장황한 출력은 하위 에이전트 컨텍스트에 격리 |
| 구체적인 프롬프트 | "improve this codebase"보다 "auth.ts의 login 함수에 입력 검증 추가"가 효율적 |

---

## 11. 분석

Claude Code는 조직이 개발자 사용 패턴을 이해하고, 기여 메트릭을 추적하며, Claude Code가 엔지니어링 속도에 미치는 영향을 측정할 수 있는 분석 대시보드를 제공한다.

### 대시보드 접근

| 플랜 | 대시보드 URL | 포함 내용 |
| --- | --- | --- |
| Claude for Teams / Enterprise | claude.ai/analytics/claude-code | 사용량 메트릭, 기여 메트릭(GitHub 연동), 리더보드, 데이터 내보내기 |
| API (Claude Console) | platform.claude.com/claude-code | 사용량 메트릭, 지출 추적, 팀 인사이트 |

### 요약 메트릭

| 메트릭 | 설명 |
| --- | --- |
| PRs with CC | Claude Code로 작성된 코드가 1줄 이상 포함된 병합된 PR 총수 |
| Lines of code with CC | Claude Code 지원으로 작성된 코드의 총 줄 수 (유효 줄만 계산) |
| PRs with Claude Code (%) | 전체 병합 PR 중 Claude Code 지원 코드가 포함된 PR의 비율 |
| Suggestion accept rate | Edit, Write, NotebookEdit 도구 사용에 대한 수락률 |
| Lines of code accepted | Claude Code가 작성하고 사용자가 수락한 총 코드 줄 수 |

### 기여 메트릭 활성화

기여 메트릭은 GitHub 조직 연결이 필요하다. Owner 역할이 필요하며, GitHub 관리자가 GitHub 앱을 설치해야 한다. 데이터는 활성화 후 24시간 이내에 나타난다. GitHub Cloud 및 GitHub Enterprise Server를 지원한다.

### PR 속성(Attribion)

1. 병합된 PR에서 추가된 줄을 추출
2. 해당 파일을 편집한 Claude Code 세션을 시간 윈도우 내에서 식별 (병합일 기준 21일 전 ~ 2일 후)
3. 다중 전략으로 PR 줄과 Claude Code 출력을 매칭
4. AI 지원 줄 수와 총 줄 수로 메트릭 계산

**제외 파일**: lock 파일, 생성된 코드, 빌드 디렉토리, 테스트 픽스처, 1,000자 이상 줄

### API 고객 분석

Console 대시보드에는 다음이 포함된다.

- **Lines of code accepted**: 수락된 코드 줄 수
- **Suggestion accept rate**: 편집 도구 수락률
- **Activity**: 일별 활성 사용자 및 세션 차트
- **Spend**: 일별 API 비용(달러)과 사용자 수

팀 인사이트 테이블에서 사용자별 메트릭을 확인할 수 있다.

---

## 12. 사용량 모니터링 (OpenTelemetry)

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
| `OTEL_LOG_RAW_API_BODIES` | 전체 API 요청/응답 JSON 로깅 | `1` 또는 `file:<dir>` |
| `OTEL_METRIC_EXPORT_INTERVAL` | 메트릭 내보내기 간격(ms, 기본 60000) | `5000` |
| `OTEL_LOGS_EXPORT_INTERVAL` | 로그 내보내기 간격(ms, 기본 5000) | `10000` |

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
| `claude_code.tool_decision` | 도구 권한 결정 시 |
| `claude_code.mcp_server_connection` | MCP 서버 연결/해제/실패 시 |
| `claude_code.auth` | 로그인/로그아웃 시 |
| `claude_code.permission_mode_changed` | 권한 모드 변경 시 |
| `claude_code.plugin_installed` | 플러그인 설치 시 |
| `claude_code.plugin_loaded` | 세션 시작 시 활성 플러그인 |
| `claude_code.skill_activated` | 스킬 호출 시 |

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

### 백엔드 선택

| 유형 | 추천 백엔드 |
| --- | --- |
| 메트릭 | Prometheus(시계열), ClickHouse(컬럼형), Honeycomb/Datadog(통합 플랫폼) |
| 이벤트/로그 | Elasticsearch/Loki(로그 집계), ClickHouse(구조화 분석) |
| 추적 | Jaeger, Zipkin, Grafana Tempo, Honeycomb/Datadog |
