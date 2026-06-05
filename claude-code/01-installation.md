# Claude Code 설치 및 설정

> 시스템 요구사항, 플랫폼별 설치, 인증, 업데이트, 제거 방법

**참고**: [Advanced Setup - Claude Code 공식 문서](https://docs.anthropic.com/en/docs/claude-code/install) | [Bedrock & Vertex - Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/bedrock-vertex)

---

## 시스템 요구사항

### 운영체제

| OS | 최소 버전 |
|----|-----------|
| macOS | 13.0+ |
| Windows | 10 1809+ 또는 Windows Server 2019+ |
| Ubuntu | 20.04+ |
| Debian | 10+ |
| Alpine Linux | 3.19+ |

### 하드웨어 및 기타

| 항목 | 요구사항 |
|------|----------|
| RAM | 4 GB 이상 |
| 프로세서 | x64 또는 ARM64 |
| 네트워크 | 인터넷 연결 필수 |
| 쉘 | Bash, Zsh, PowerShell, CMD |
| 추가 의존성 | ripgrep (보통 Claude Code에 포함됨) |

> 네이티브 Windows 설정에는 Git for Windows가 필요합니다. WSL 설정에는 필요하지 않습니다.

---

## 네이티브 설치 (권장)

### macOS / Linux / WSL

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

### Windows PowerShell

```powershell
irm https://claude.ai/install.ps1 | iex
```

### Windows CMD

```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

> `The token '&&' is not a valid statement separator` 오류가 나타나면 PowerShell이 아닌 CMD에서 실행 중인 것입니다. PowerShell 명령을 사용하세요. PowerShell 프롬프트는 `PS C:\` 형태로 표시됩니다.

### Homebrew (macOS)

```bash
# Stable 채널 (약 1주 지연, 주요 회귀 제외)
brew install --cask claude-code

# Latest 채널 (최신 버전 즉시 수신)
brew install --cask claude-code@latest
```

### WinGet (Windows)

```cmd
winget install Anthropic.ClaudeCode
```

---

## Windows 설정

Windows에서는 네이티브 또는 WSL 중 선택할 수 있습니다:

| 옵션 | 요구사항 | 샌드박싱 | 사용 시기 |
|------|----------|----------|-----------|
| 네이티브 Windows | Git for Windows | 미지원 | Windows 네이티브 프로젝트 및 도구 |
| WSL 2 | WSL 2 활성화 | 지원 | Linux 툴체인 또는 샌드박스 명령 실행 |
| WSL 1 | WSL 1 활성화 | 미지원 | WSL 2를 사용할 수 없는 경우 |

### 옵션 1: 네이티브 Windows (Git Bash)

Git for Windows를 먼저 설치한 후 PowerShell 또는 CMD에서 설치 명령을 실행합니다. Administrator 권한은 필요하지 않습니다.

설치 후 PowerShell, CMD, Git Bash 어디서든 `claude`를 실행할 수 있습니다. Claude Code는 내부적으로 Git Bash를 사용하여 명령을 실행합니다. Git Bash 경로를 찾지 못하는 경우 `settings.json`에 설정합니다:

```json
{
  "env": {
    "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"
  }
}
```

### 옵션 2: WSL

WSL 배포판을 열고 Linux 설치 명령을 실행합니다. PowerShell이나 CMD가 아닌 **WSL 터미널 내부**에서 설치 및 실행합니다.

---

## Alpine Linux 설정

Alpine 및 기타 musl/uClibc 기반 배포판에서는 `libgcc`, `libstdc++`, `ripgrep`이 필요합니다:

```bash
# 필수 패키지 설치
apk add libgcc libstdc++ ripgrep
```

`settings.json`에 다음을 추가합니다:

```json
{
  "env": {
    "USE_BUILTIN_RIPGREP": "0"
  }
}
```

---

## npm 설치 (Deprecated)

> **주의**: npm 설치는 **더 이상 권장되지 않습니다**. 네이티브 설치가 더 빠르고, 의존성이 필요 없으며, 백그라운드에서 자동 업데이트됩니다. 가능하면 네이티브 설치 방법을 사용하세요.

### npm에서 네이티브로 마이그레이션

```bash
# 네이티브 바이너리 설치
curl -fsSL https://claude.ai/install.sh | bash

# 기존 npm 설치 제거
npm uninstall -g @anthropic-ai/claude-code
```

기존 npm 설치에서 `claude install`을 실행하여 네이티브 바이너리를 나란히 설치한 후 npm 버전을 제거할 수도 있습니다.

### npm으로 설치 (호환성이 필요한 경우)

Node.js 18+가 필요합니다:

```bash
npm install -g @anthropic-ai/claude-code
```

---

## 인증

Claude Code를 사용하려면 다음 계정 중 하나가 필요합니다:

| 계정 유형 | 설명 |
|-----------|------|
| Pro | Claude.ai Pro 구독 |
| Max | Claude.ai Max 구독 |
| Team | Claude.ai Team 플랜 |
| Enterprise | Claude.ai Enterprise 플랜 |
| Console | Anthropic Console (API 사용량 기반 요금 청구) |

> 무료 Claude.ai 플랜에는 Claude Code 액세스가 포함되지 않습니다.

Amazon Bedrock, Google Vertex AI, Microsoft Foundry와 같은 서드파티 API 프로바이더를 통해서도 사용할 수 있습니다.

설치 후 `claude`를 실행하고 브라우저 프롬프트를 따라 로그인합니다.

```bash
claude
# 최초 실행 시 브라우저에서 로그인 프롬프트가 표시됩니다
```

---

## 업데이트

### 자동 업데이트

네이티브 설치의 경우 백그라운드에서 자동으로 업데이트됩니다. Claude Code는 시작 시 그리고 실행 중 주기적으로 업데이트를 확인합니다. 업데이트는 백그라운드에서 다운로드 및 설치되며, 다음 Claude Code 시작 시 적용됩니다.

> Homebrew 및 WinGet 설치는 수동 업데이트가 필요합니다.

### 릴리스 채널 구성

`autoUpdatesChannel` 설정으로 업데이트 채널을 제어합니다:

| 채널 | 설명 |
|------|------|
| `latest` (기본값) | 새 기능이 출시되는 즉시 수신 |
| `stable` | 약 1주일 뒤의 버전을 사용하며, 주요 회귀가 있는 릴리스를 건너뜀 |

```json
{
  "autoUpdatesChannel": "stable"
}
```

`/config` 메뉴의 **Auto-update channel**에서도 설정할 수 있습니다.

> Homebrew는 캐스트 이름으로 채널을 선택합니다: `claude-code`는 stable, `claude-code@latest`는 latest를 추적합니다.

### 자동 업데이트 비활성화

```json
{
  "env": {
    "DISABLE_AUTOUPDATER": "1"
  }
}
```

### 수동 업데이트

```bash
claude update
```

---

## 특정 버전 설치

### 최신 버전 설치 (기본값)

```bash
# macOS, Linux, WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# Windows CMD
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

### Stable 버전 설치

```bash
# macOS, Linux, WSL
curl -fsSL https://claude.ai/install.sh | bash -s stable

# Windows PowerShell
& ([scriptblock]::Create((irm https://claude.ai/install.ps1))) stable

# Windows CMD
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd stable && del install.cmd
```

### 특정 버전 번호 설치

```bash
# macOS, Linux, WSL
curl -fsSL https://claude.ai/install.sh | bash -s 2.1.89

# Windows PowerShell
& ([scriptblock]::Create((irm https://claude.ai/install.ps1))) 2.1.89

# Windows CMD
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd 2.1.89 && del install.cmd
```

---

## 바이너리 무결성 및 코드 서명

각 릴리스는 모든 플랫폼 바이너리의 SHA256 체크섬이 포함된 `manifest.json`을 게시합니다. 매니페스트는 Anthropic GPG 키로 서명되어 서명 확인을 통해 모든 바이너리의 무결성을 간접적으로 확인할 수 있습니다.

### 플랫폼 코드 서명

| 플랫폼 | 서명 | 확인 방법 |
|--------|------|-----------|
| macOS | "Anthropic PBC"가 서명, Apple 공증 | `codesign --verify --verbose ./claude` |
| Windows | "Anthropic, PBC"가 서명 | `Get-AuthenticodeSignature .\claude.exe` |
| Linux | 개별 코드 서명 없음 | 매니페스트 서명으로 무결성 확인 |

---

## 제거 방법

### 네이티브 설치 제거

```bash
# macOS, Linux, WSL
rm -f ~/.local/bin/claude
rm -rf ~/.local/share/claude

# Windows PowerShell
Remove-Item -Path "$env:USERPROFILE\.local\bin\claude.exe" -Force
Remove-Item -Path "$env:USERPROFILE\.local\share\claude" -Recurse -Force
```

### Homebrew 제거

```bash
# Stable 캐스트인 경우
brew uninstall --cask claude-code

# Latest 캐스트인 경우
brew uninstall --cask claude-code@latest
```

### WinGet 제거

```cmd
winget uninstall Anthropic.ClaudeCode
```

### npm 제거

```bash
npm uninstall -g @anthropic-ai/claude-code
```

### 설정 파일 완전 제거

```bash
# macOS, Linux, WSL
rm -rf ~/.claude
rm ~/.claude.json
rm -rf .claude          # 프로젝트 디렉토리에서
rm -f .mcp.json         # 프로젝트 디렉토리에서

# Windows PowerShell
Remove-Item -Path "$env:USERPROFILE\.claude" -Recurse -Force
Remove-Item -Path "$env:USERPROFILE\.claude.json" -Force
Remove-Item -Path ".claude" -Recurse -Force
Remove-Item -Path ".mcp.json" -Force
```

---

## Bedrock / Vertex AI 연동

### Amazon Bedrock 연결

```bash
# Bedrock 활성화
export CLAUDE_CODE_USE_BEDROCK=1

# 프롬프트 캐싱이 활성화되지 않은 경우 추가 설정
export DISABLE_PROMPT_CACHING=1

# 프록시 사용 시
export ANTHROPIC_BEDROCK_BASE_URL=https://your-proxy-url
```

AWS SDK 자격 증명이 필요합니다 (`~/.aws/credentials` 또는 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` 환경 변수):

```bash
# AWS 자격 증명 설정
aws configure
```

> 비용 절감 및 향상된 속도 제한을 위해 Amazon Bedrock에 프롬프트 캐싱을 활성화하세요.

### Google Vertex AI 연결

```bash
# Vertex AI 활성화
export CLAUDE_CODE_USE_VERTEX=1

# 프롬프트 캐싱이 활성화되지 않은 경우 추가 설정
export DISABLE_PROMPT_CACHING=1

# 프록시 사용 시
export ANTHROPIC_VERTEX_BASE_URL=https://your-proxy-url
```

GCP 자격 증명이 필요합니다 (google-auth-library를 통해 구성):

```bash
# GCP 자격 증명 설정
gcloud auth application-default login
```

> 최상의 경험을 위해 Google에 향상된 속도 제한을 요청하세요.

---

## 프록시 연동

LLM 프록시(예: LiteLLM)와 함께 Claude Code를 사용할 때 인증 동작을 제어할 수 있습니다.

### 환경 변수

| 변수 | 설명 |
|------|------|
| `ANTHROPIC_AUTH_TOKEN` | `Authorization` 및 `Proxy-Authorization` 헤더에 사용할 커스텀 값 (`Bearer` 접두사 자동 추가) |
| `ANTHROPIC_CUSTOM_HEADERS` | 요청에 추가할 커스텀 헤더 (`Name: Value` 형식) |
| `HTTP_PROXY` | HTTP 프록시 URL |
| `HTTPS_PROXY` | HTTPS 프록시 URL |

### 글로벌 설정을 통한 구성

환경 변수 대신 `~/.claude.json`의 `env` 객체에 추가할 수도 있습니다:

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your-token",
    "ANTHROPIC_CUSTOM_HEADERS": "X-Custom-Header: value",
    "HTTP_PROXY": "http://proxy.example.com:8080"
  }
}
```

### apiKeyHelper 설정

`apiKeyHelper`를 사용하여 API 키를 가져오는 커스텀 쉘 스크립트를 실행할 수 있습니다. 시작 시 한 번 호출되며 각 세션 동안 캐시됩니다.

```json
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh"
}
```

---

## 설치 확인

설치 후 Claude Code가 정상적으로 작동하는지 확인합니다:

```bash
claude --version
```

더 자세한 설치 및 구성 확인:

```bash
claude doctor
```
