# 07. MCP 통합 (Model Context Protocol)

> **참조**: [https://docs.anthropic.com/en/docs/claude-code/mcp](https://docs.anthropic.com/en/docs/claude-code/mcp)

---

## 1. MCP 개요

Claude Code는 **Model Context Protocol (MCP)** 을 통해 수백 개의 외부 도구와 데이터 소스에 연결할 수 있습니다. MCP는 AI-도구 통합을 위한 오픈소스 표준으로, MCP 서버를 통해 Claude Code가 도구, 데이터베이스, API에 접근할 수 있게 합니다.

> **중요**: Claude Code는 **stdio뿐만 아니라 HTTP, SSE 전송도 지원**합니다. 로컬 도구에는 stdio를, 원격 서비스에는 HTTP/SSE를 사용할 수 있습니다.

### MCP로 할 수 있는 일

- **이슈 트래커에서 기능 구현**: "JIRA 이슈 ENG-4521에 설명된 기능을 추가하고 GitHub에 PR 생성"
- **모니터링 데이터 분석**: "Sentry와 Statsig에서 ENG-4521 기능의 사용량 확인"
- **데이터베이스 쿼리**: "PostgreSQL에서 ENG-4521 기능을 사용한 사용자 10명의 이메일 찾기"
- **디자인 통합**: "Slack에 게시된 Figma 디자인에 따라 이메일 템플릿 업데이트"
- **워크플로우 자동화**: "10명의 사용자에게 새 기능 피드백 세션 초장하는 Gmail 초안 생성"

---

## 2. MCP 서버 설치 3가지 방법

### 방법 1: 원격 HTTP 서버

HTTP 서버는 원격 MCP 서버 연결에 권장되는 옵션입니다. 클라우드 기반 서비스에 가장 널리 지원되는 전송 방식입니다.

```bash
# 기본 문법
claude mcp add --transport http <name> <url>

# 실제 예시: Notion 연결
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Bearer 토큰 포함 예시
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### 방법 2: 원격 SSE 서버

SSE (Server-Sent Events) 전송 방식입니다.

```bash
# 기본 문법
claude mcp add --transport sse <name> <url>

# 실제 예시: Asana 연결
claude mcp add --transport sse asana https://mcp.asana.com/sse

# 인증 헤더 포함 예시
claude mcp add --transport sse private-api https://api.company.com/sse \
  --header "X-API-Key: your-key-here"
```

### 방법 3: 로컬 stdio 서버

stdio 서버는 로컬 프로세스로 실행됩니다. 시스템에 직접 접근해야 하는 도구나 커스텀 스크립트에 적합합니다.

```bash
# 기본 문법
claude mcp add --transport stdio <name> <command> [args...]

# 실제 예시: Airtable 서버
claude mcp add --transport stdio airtable --env AIRTABLE_API_KEY=YOUR_KEY \
  -- npx -y airtable-mcp-server

# 데이터베이스 서버 예시
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:password@localhost:5432/analytics"
```

---

## 3. 서버 관리 명령어

```bash
# 모든 구성된 서버 목록
claude mcp list

# 특정 서버 상세 정보
claude mcp get github

# 서버 제거
claude mcp remove github

# 프로젝트 승인 선택 초기화
claude mcp reset-project-choices

# Claude Code 내에서 서버 상태 확인
/mcp
```

---

## 4. 설치 스코프 3가지 및 우선순위

MCP 서버는 세 가지 스코프 수준에서 구성할 수 있습니다.

### Local 스코프 (기본값)

`~/.claude.json`에 저장되며, 현재 프로젝트 디렉토리에서만 접근 가능합니다. 개인 개발 서버, 실험적 구성, 민감한 자격 증명이 포함된 서버에 적합합니다.

```bash
# 로컬 스코프 서버 추가 (기본값)
claude mcp add --transport http stripe https://mcp.stripe.com

# 명시적 지정
claude mcp add --transport http stripe --scope local https://mcp.stripe.com
```

### Project 스코프

프로젝트 루트의 `.mcp.json` 파일에 저장됩니다. 버전 관리에 체크인되어 모든 팀원이 동일한 MCP 도구에 접근할 수 있습니다.

```bash
# 프로젝트 스코프 서버 추가
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp
```

### User 스코프

`~/.claude.json`에 저장되며, 모든 프로젝트에서 접근 가능합니다. 개인 유틸리티 서버, 자주 사용하는 서비스에 적합합니다.

```bash
# 사용자 스코프 서버 추가
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### 스코프 선택 가이드

| 스코프 | 적합한 경우 |
|--------|-------------|
| **Local** | 개인 서버, 실험적 구성, 민감한 자격 증명 (프로젝트별) |
| **Project** | 팀 공유 서버, 프로젝트별 도구, 협업 필수 서비스 |
| **User** | 여러 프로젝트에서 필요한 개인 유틸리티, 자주 사용하는 서비스 |

### 우선순위 (Precedence)

같은 이름의 서버가 여러 스코프에 존재할 때: **Local > Project > User** 순서로 우선순위가 적용됩니다.

---

## 5. .mcp.json 파일 형식

프로젝트 스코프 서버는 `.mcp.json` 파일로 관리됩니다.

```json
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

### HTTP 서버 예시

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

> **보안**: Claude Code는 `.mcp.json` 파일의 프로젝트 스코프 서버를 사용하기 전에 승인을 요청합니다.

---

## 6. 환경변수 확장

`.mcp.json` 파일에서 환경변수 확장을 지원합니다. 팀이 구성을 공유하면서 머신별 경로와 API 키 등을 유연하게 관리할 수 있습니다.

### 지원 문법

| 문법 | 설명 |
|------|------|
| `${VAR}` | 환경변수 `VAR`의 값으로 확장 |
| `${VAR:-default}` | `VAR`이 설정되어 있으면 그 값, 없으면 `default` 사용 |

### 확장 적용 위치

- `command` - 서버 실행 파일 경로
- `args` - 명령줄 인자
- `env` - 서버에 전달되는 환경변수
- `url` - HTTP 서버 타입의 URL
- `headers` - HTTP 서버 인증 헤더

> 필수 환경변수가 설정되지 않고 기본값도 없으면 Claude Code가 구성 파싱에 실패합니다.

---

## 7. 플러그인 제공 MCP 서버

플러그인은 MCP 서버를 번들로 포함할 수 있으며, 플러그인이 활성화되면 자동으로 도구와 통합을 제공합니다.

### `.mcp.json` 방식 (플러그인 루트)

```json
{
  "database-tools": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
    "env": {
      "DB_URL": "${DB_URL}"
    }
  }
}
```

### `plugin.json` 인라인 방식

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

### 플러그인 MCP 기능

| 기능 | 설명 |
|------|------|
| **자동 수명 주기** | 플러그인 활성화 시 서버 시작. 단, MCP 서버 변경(활성화/비활성화) 적용을 위해 Claude Code 재시작 필요 |
| **환경변수** | `${CLAUDE_PLUGIN_ROOT}`로 플러그인 상대 경로 사용 |
| **사용자 환경 접근** | 수동 구성 서버와 동일한 환경변수 접근 |
| **다중 전송 타입** | stdio, SSE, HTTP 전송 지원 |

---

## 8. Claude Code 자체를 MCP 서버로 사용

Claude Code 자체를 MCP 서버로 실행하여 다른 애플리케이션에서 연결할 수 있습니다.

```bash
# stdio MCP 서버로 Claude 시작
claude mcp serve
```

### Claude Desktop에서 사용

`claude_desktop_config.json`에 다음 설정을 추가합니다.

```json
{
  "mcpServers": {
    "claude-code": {
      "type": "stdio",
      "command": "claude",
      "args": ["mcp", "serve"],
      "env": {}
    }
  }
}
```

---

## 9. MCP 출력 제한

MCP 도구가 대량의 출력을 생성할 때 토큰 사용을 관리합니다.

| 항목 | 값 |
|------|-----|
| **출력 경고 임계값** | MCP 도구 출력이 10,000 토큰 초과 시 경고 표시 |
| **기본 최대 제한** | 25,000 토큰 |
| **설정 변수** | `MAX_MCP_OUTPUT_TOKENS` 환경변수 |

```bash
# MCP 출력 제한 증가
export MAX_MCP_OUTPUT_TOKENS=50000
claude
```

대량 출력이 발생하는 경우:
- 대규모 데이터셋 또는 데이터베이스 쿼리
- 상세한 보고서 또는 문서 생성
- 광범위한 로그 파일 또는 디버깅 정보

---

## 10. 엔터프라이즈 MCP 설정

조직에서 MCP 서버를 중앙 집중식으로 제어할 수 있습니다. IT 관리자가 승인된 MCP 서버를 배포하고, 사용자가 임의로 서버를 추가하지 못하도록 제한할 수 있습니다.

### managed-mcp.json 설정

시스템 관리자가 관리 설정 파일과 함께 엔터프라이즈 MCP 설정 파일을 배포합니다.

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
        "COMPANY_API_URL": "https://internal.company.com"
      }
    }
  }
}
```

---

## 11. 허용/차단 목록 상세

관리 설정 파일에서 `allowedMcpServers`와 `deniedMcpServers`를 사용하여 사용자가 구성할 수 있는 MCP 서버를 제어합니다.

### 제한 옵션

각 항목은 두 가지 방식으로 서버를 제한할 수 있습니다:

| 방식 | 필드 | 설명 |
|------|------|------|
| 서버 이름 | `serverName` | 구성된 서버 이름과 매칭 |
| 명령어 | `serverCommand` | stdio 서버 시작에 사용된 정확한 명령어 및 인자와 매칭 |

> **중요**: 각 항목은 `serverName` 또는 `serverCommand` 중 **하나만** 가져야 합니다.

### 설정 예시

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverName": "sentry" },
    { "serverCommand": ["npx", "-y", "@modelcontextprotocol/server-filesystem"] },
    { "serverCommand": ["python", "/usr/local/bin/approved-server.py"] }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" },
    { "serverCommand": ["npx", "-y", "unapproved-package"] }
  ]
}
```

### 명령어 기반 제한 작동 방식

**정확한 매칭**:
- 명령어 배열은 명령어와 모든 인자가 정확히 순서대로 일치해야 함
- 예: `["npx", "-y", "server"]`은 `["npx", "server"]` 또는 `["npx", "-y", "server", "--flag"]`와 매치되지 않음

**stdio 서버 동작**:
- 허용 목록에 `serverCommand` 항목이 있으면 stdio 서버는 반드시 그 중 하나와 매치되어야 함
- 명령어 제한이 있을 때 이름만으로는 통과할 수 없음

**비-stdio 서버 동작**:
- 원격 서버 (HTTP, SSE, WebSocket)는 이름으로만 매칭
- 명령어 제한은 원격 서버에 적용되지 않음

### 허용 목록 동작 (`allowedMcpServers`)

| 값 | 동작 |
|----|------|
| `undefined` (기본값) | 제한 없음 - 모든 MCP 서버 구성 가능 |
| 빈 배열 `[]` | 전체 잠금 - 어떤 MCP 서버도 구성 불가 |
| 항목 목록 | 이름 또는 명령어와 매치되는 서버만 구성 가능 |

### 차단 목록 동작 (`deniedMcpServers`)

| 값 | 동작 |
|----|------|
| `undefined` (기본값) | 차단되는 서버 없음 |
| 빈 배열 `[]` | 차단되는 서버 없음 |
| 항목 목록 | 지정된 서버가 모든 스코프에서 명시적으로 차단됨 |

### 우선순위 규칙

- 모든 스코프에 적용: user, project, local, 엔터프라이즈 `managed-mcp.json` 포함
- **차단 목록이 절대 우선**: 서버가 차단 목록 항목과 매치되면 허용 목록에 있어도 차단됨
- 이름 기반과 명령어 기반 제한은 함께 작동: 서버는 이름 항목 **또는** 명령어 항목 중 하나와 매치되면 통과 (차단 목록에 없는 한)

---

## 12. MCP 서버 개발 기본

### Node.js MCP 서버 예제

```javascript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({
  name: "my-mcp-server",
  version: "1.0.0",
});

// 도구 등록
server.tool(
  "get_data",
  "데이터를 조회합니다",
  { query: { type: "string", description: "검색 쿼리" } },
  async ({ query }) => {
    return {
      content: [{ type: "text", text: `검색 결과: ${query}` }],
    };
  }
);

// 서버 시작
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Claude Code에 등록

```bash
claude mcp add --transport stdio my-server -- node /path/to/server.js
```

---

## 13. 실전 예제

### 예제 1: 파일시스템 MCP 서버

```bash
# 파일시스템 서버 추가
claude mcp add --transport stdio filesystem \
  -- npx -y @modelcontextprotocol/server-filesystem /home/user/projects

# Claude Code에서 사용
> "프로젝트 디렉토리의 모든 TypeScript 파일을 나열하세요"
> "README.md 파일을 읽고 요약하세요"
```

### 예제 2: Sentry로 에러 모니터링

```bash
# 1. Sentry MCP 서버 추가
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. 인증
> /mcp

# 3. 프로덕션 이슈 디버깅
> "최근 24시간 동안 가장 많이 발생한 에러는 무엇인가요?"
> "에러 ID abc123의 스택 트레이스를 보여주세요"
> "어떤 배포에서 이 에러들이 새로 발생했나요?"
```

### 예제 3: GitHub으로 코드 리뷰

```bash
# 1. GitHub MCP 서버 추가
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 2. 인증
> /mcp

# 3. GitHub 작업
> "PR #456을 리뷰하고 개선점을 제안해주세요"
> "방금 발견한 버그에 대한 이슈를 생성해주세요"
> "나에게 할당된 열린 PR을 모두 보여주세요"
```

### 예제 4: PostgreSQL 데이터베이스 쿼리

```bash
# 1. 데이터베이스 서버 추가
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:password@localhost:5432/analytics"

# 2. 자연어로 쿼리
> "이번 달 총 수익은 얼마인가요?"
> "orders 테이블의 스키마를 보여주세요"
> "90일 동안 구매하지 않은 고객을 찾아주세요"
```

### 예제 5: 원격 API 통합 (HTTP)

```bash
# Bearer 토큰 인증이 필요한 API 서버
claude mcp add --transport http my-api https://api.mycompany.com/mcp \
  --header "Authorization: Bearer ${API_TOKEN}"

# 헤더 인증이 필요한 SSE 서버
claude mcp add --transport sse monitoring https://monitoring.mycompany.com/sse \
  --header "X-API-Key: ${MONITORING_KEY}"
```

---

## 14. MCP 리소스 및 프롬프트

### MCP 리소스 참조

MCP 서버가 노출하는 리소스를 `@` 멘션으로 참조할 수 있습니다. 파일 참조와 유사한 방식입니다.

### MCP 프롬프트를 슬래시 명령어로 사용

MCP 서버가 노출하는 프롬프트는 Claude Code에서 슬래시 명령어로 사용할 수 있습니다.

```
/mcp__<server-name>__<prompt-name> [arguments]
```

자세한 내용은 [슬래시 명령어 문서](./05-slash-commands.md)를 참조하세요.
