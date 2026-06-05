# Claude Code 설치 및 설정

> 시스템 요구사항, 플랫폼별 설치, 인증, 업데이트, 제거 방법

**원문**: [Advanced Setup](https://code.claude.com/docs/en/setup) | [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock) | [Google Vertex AI](https://code.claude.com/docs/en/google-vertex-ai) | [Microsoft Foundry](https://code.claude.com/docs/en/microsoft-foundry)

**참고**: [Advanced Setup - Claude Code 공식 문서](https://docs.anthropic.com/en/docs/claude-code/setup) | [Bedrock - Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/bedrock) | [Vertex AI - Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/vertex) | [Microsoft Foundry - Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/microsoft-foundry)

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
| 위치 | Anthropic 지원 국가 |
| 추가 의존성 | ripgrep (보통 Claude Code에 포함됨) |

> 네이티브 Windows 설정에는 Git for Windows가 필요합니다. WSL 설정에는 필요하지 않습니다.

> **새 기능**: Claude Code는 Windows에서 PowerShell을 네이티브 쉘로 사용할 수 있는 **옵트인 프리뷰** 기능을 지원합니다. 자세한 내용은 PowerShell 도구 문서를 참조하세요.

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

## Linux 패키지 매니저 설치

Claude Code는 서명된 apt, dnf, apk 저장소를 제공합니다. `stable`을 `latest`로 변경하면 롤링 채널을 사용합니다. 패키지 매니저 설치는 Claude Code를 통해 자동 업데이트되지 않으며, 일반 시스템 업그레이드 프로세스를 통해 업데이트됩니다.

### apt (Debian / Ubuntu)

```bash
sudo install -d -m 0755 /etc/apt/keyrings
sudo curl -fsSL https://downloads.claude.ai/keys/claude-code.asc \
  -o /etc/apt/keyrings/claude-code.asc
echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/stable stable main" \
  | sudo tee /etc/apt/sources.list.d/claude-code.list
sudo apt update
sudo apt install claude-code
```

GPG 키 핑거프린트 확인: `gpg --show-keys /etc/apt/keyrings/claude-code.asc` 출력이 `31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE`인지 확인하세요.

업그레이드: `sudo apt update && sudo apt upgrade claude-code`

### dnf (Fedora / RHEL)

```bash
sudo tee /etc/yum.repos.d/claude-code.repo <<'EOF'
[claude-code]
name=Claude Code
baseurl=https://downloads.claude.ai/claude-code/rpm/stable
enabled=1
gpgcheck=1
gpgkey=https://downloads.claude.ai/keys/claude-code.asc
EOF
sudo dnf install claude-code
```

dnf는 첫 설치 시 키를 다운로드하고 핑거프린트 확인을 요청합니다. `31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE`와 일치하는지 확인 후 수락하세요.

업그레이드: `sudo dnf upgrade claude-code`

### apk (Alpine Linux)

```bash
wget -O /etc/apk/keys/claude-code.rsa.pub \
  https://downloads.claude.ai/keys/claude-code.rsa.pub
echo "https://downloads.claude.ai/claude-code/apk/stable" >> /etc/apk/repositories
apk add claude-code
```

키 무결성 확인: `sha256sum /etc/apk/keys/claude-code.rsa.pub` 출력이 `395759c1f7449ef4cdef305a42e820f3c766d6090d142634ebdb049f113168b6`인지 확인하세요.

업그레이드: `apk update && apk upgrade claude-code`

---

## Alpine Linux 추가 설정

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

### 최소 버전 고정 (minimumVersion)

`minimumVersion` 설정은 바닥값을 설정합니다. 백그라운드 자동 업데이트와 `claude update`는 이 값 미만의 버전 설치를 거부하므로, `stable` 채널로 전환해도 이미 사용 중인 최신 빌드에서 다운그레이드되지 않습니다.

```json
{
  "autoUpdatesChannel": "stable",
  "minimumVersion": "2.1.100"
}
```

`/config`에서 `latest`에서 `stable`로 전환하면 현재 버전을 유지할지 다운그레이드를 허용할지 선택할 수 있습니다. 유지를 선택하면 `minimumVersion`이 해당 버전으로 설정됩니다. 다시 `latest`로 전환하면 초기화됩니다.

관리 설정(managed settings)에서 조직 전체 최소 버전을 강제할 수 있으며, 사용자 및 프로젝트 설정으로는 재정의할 수 없습니다.

### 자동 업데이트 비활성화

`DISABLE_AUTOUPDATER`는 백그라운드 확인만 중지하며, `claude update` 및 `claude install`은 여전히 작동합니다.

```json
{
  "env": {
    "DISABLE_AUTOUPDATER": "1"
  }
}
```

수동 업데이트를 포함한 **모든** 업데이트 경로를 차단하려면 `DISABLE_UPDATES`를 대신 사용하세요. 자체 채널을 통해 Claude Code를 배포하고 사용자가 제공된 버전을 유지해야 하는 경우에 사용합니다.

```json
{
  "env": {
    "DISABLE_UPDATES": "1"
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

## Amazon Bedrock 연동

### 사전 요구사항

- AWS 계정에 Bedrock 액세스가 활성화되어 있어야 함
- 원하는 Claude 모델(예: Claude Sonnet 4.6)에 대한 Bedrock 액세스 권한
- AWS CLI 설치 및 구성 (선택 사항 — 다른 자격 증명 메커니즘이 있는 경우 불필요)
- 적절한 IAM 권한

### Sign-in 위저드

AWS 자격 증명이 있고 Bedrock을 통해 Claude Code를 시작하려는 경우, 로그인 위저드가 안내합니다. AWS 측 사전 요구사항은 계정당 한 번만 완료하면 되며, 위저드가 Claude Code 측 설정을 처리합니다.

```bash
claude
# Bedrock을 선택하면 위저드가 시작됩니다
```

로그인 후 `/setup-bedrock`을 실행하여 언제든 위저드를 다시 열고 자격 증명, 리전, 모델 핀을 변경할 수 있습니다.

### 수동 설정

위저드 대신 환경 변수로 Bedrock을 구성하려면(예: CI 또는 엔터프라이즈 롤아웃) 다음 단계를 따르세요.

#### 1. 유스케이스 세부 정보 제출

Anthropic 모델을 처음 사용하는 경우 모델 호출 전에 유스케이스 세부 정보를 제출해야 합니다 (AWS 계정당 한 번).

1. 올바른 IAM 권한이 있는지 확인
2. Amazon Bedrock 콘솔로 이동
3. __Model catalog__에서 Anthropic 모델 선택
4. 유스케이스 양식 작성. 제출 후 즉시 액세스가 부여됨

AWS Organizations를 사용하는 경우 관리 계정에서 `PutUseCaseForModelAccess` API를 한 번 호출하여 승인을 받으면 하위 계정에 자동으로 확장됩니다.

#### 2. AWS 자격 증명 구성

Claude Code는 기본 AWS SDK 자격 증명 체인을 사용합니다. 다음 방법 중 하나로 자격 증명을 설정하세요.

| 방법 | 설정 |
|------|------|
| **A. AWS CLI** | `aws configure` 실행 |
| **B. 환경 변수 (액세스 키)** | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` 설정 |
| **C. 환경 변수 (SSO 프로파일)** | `aws sso login --profile=<프로파일>` 후 `AWS_PROFILE` 설정 |
| **D. AWS Management Console** | `aws login` 사용 |
| **E. Bedrock API 키** | `AWS_BEARER_TOKEN_BEDROCK` 설정 (전체 AWS 자격 증명 없이 간단한 인증) |

```bash
# 방법 B 예시
export AWS_ACCESS_KEY_ID=your-access-key-id
export AWS_SECRET_ACCESS_KEY=your-secret-access-key
export AWS_SESSION_TOKEN=your-session-token

# 방법 C 예시
aws sso login --profile=your-profile-name
export AWS_PROFILE=your-profile-name

# 방법 E 예시 (Bedrock API 키)
export AWS_BEARER_TOKEN_BEDROCK=your-bedrock-api-key
```

#### 고급 자격 증명 구성 (awsAuthRefresh / awsCredentialExport)

Claude Code는 AWS SSO 및 기업 ID 공급자를 위한 자동 자격 증명 새로고침을 지원합니다. Claude Code 설정 파일(Settings 참조)에 추가합니다.

| 설정 | 동작 |
|------|------|
| `awsAuthRefresh` | 자격 증명이 만료된 것으로 감지된 경우에만 실행. `.aws` 디렉토리를 수정하는 명령에 적합 |
| `awsCredentialExport` | 세션 시작 시 및 각 자격 증명 새로고침 때마다 실행. 기본 제공자 체인과 다른 교차 계정 자격 증명이 필요한 경우 사용 |

```json
{
  "awsAuthRefresh": "aws sso login --profile myprofile",
  "env": {
    "AWS_PROFILE": "myprofile"
  }
}
```

`awsCredentialExport`를 사용하는 경우 명령은 다음 JSON 형식으로 출력해야 합니다:

```json
{
  "Credentials": {
    "AccessKeyId": "value",
    "SecretAccessKey": "value",
    "SessionToken": "value"
  }
}
```

#### 3. Claude Code 구성

```bash
# Bedrock 활성화
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1  # 또는 선호하는 리전

# 선택: 소형/고속 모델의 AWS 리전 재정의 (Bedrock 및 Mantle)
# ANTHROPIC_DEFAULT_HAIKU_MODEL 또는 ANTHROPIC_SMALL_FAST_MODEL이 설정된 경우에만 적용
export ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION=us-west-2

# 선택: 커스텀 엔드포인트 또는 게이트웨이용 Bedrock 엔드포인트 URL 재정의
# export ANTHROPIC_BEDROCK_BASE_URL=https://bedrock-runtime.us-east-1.amazonaws.com
```

**주의 사항:**

- `AWS_REGION`은 필수 환경 변수입니다. Claude Code는 이 설정을 `.aws` config 파일에서 읽지 않습니다.
- Bedrock 사용 시 `/logout` 명령을 사용할 수 없습니다 (AWS 자격 증명으로 인증 처리).
- WebSearch 도구는 Bedrock에서 사용할 수 없습니다.
- 설정 파일을 사용하여 `AWS_PROFILE` 등 다른 프로세스에 노출하고 싶지 않은 환경 변수를 관리할 수 있습니다.

#### 4. 모델 버전 고정

특정 Bedrock 모델 ID로 환경 변수를 설정합니다.

```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL='us.anthropic.claude-opus-4-8'
export ANTHROPIC_DEFAULT_SONNET_MODEL='us.anthropic.claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='us.anthropic.claude-haiku-4-5-20251001-v1:0'
```

`ANTHROPIC_DEFAULT_OPUS_MODEL`을 설정하지 않으면 `opus` 별칭은 Opus 4.6으로 해결됩니다. 최신 모델을 사용하려면 Opus 4.8 ID로 설정하세요.

**기본 모델 (고정 변수 미설정 시):**

| 모델 유형 | 기본값 |
|-----------|--------|
| Primary model | `us.anthropic.claude-sonnet-4-5-20250929-v1:0` |
| Small/fast model | Primary model과 동일 |

백그라운드 작업(예: 세션 제목 생성)은 소형/고속 모델을 사용합니다. Bedrock에서는 Haiku가 모든 계정이나 리전에서 활성화되어 있지 않을 수 있어 기본적으로 primary model로 설정됩니다. Haiku를 사용하려면 `ANTHROPIC_DEFAULT_HAIKU_MODEL`을 설정하세요.

```bash
# 추가 커스터마이징
# 추론 프로파일 ID 사용
export ANTHROPIC_MODEL='us.anthropic.claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='us.anthropic.claude-haiku-4-5-20251001-v1:0'

# 애플리케이션 추론 프로파일 ARN 사용
export ANTHROPIC_MODEL='arn:aws:bedrock:us-east-2:your-account-id:application-inference-profile/your-model-id'

# 선택: 프롬프트 캐싱 비활성화
export DISABLE_PROMPT_CACHING=1

# 선택: 5분 기본값 대신 1시간 프롬프트 캐시 TTL 요청
export ENABLE_PROMPT_CACHING_1H=1
```

> 1시간 캐시 TTL은 5분 기본값보다 높은 요금이 청구됩니다.

#### modelOverrides — 모델 버전별 추론 프로파일 매핑

조직에서 동일한 모델 패밀리의 여러 버전을 `/model` 피커에 노출해야 하는 경우, `modelOverrides` 설정을 사용하여 각 버전을 개별 애플리케이션 추론 프로파일 ARN에 매핑합니다.

```json
{
  "modelOverrides": {
    "claude-opus-4-7": "arn:aws:bedrock:us-east-2:123456789012:application-inference-profile/opus-47-prod",
    "claude-opus-4-6": "arn:aws:bedrock:us-east-2:123456789012:application-inference-profile/opus-46-prod",
    "claude-opus-4-5-20251101": "arn:aws:bedrock:us-east-2:123456789012:application-inference-profile/opus-45-prod",
    "claude-opus-4-1-20250805": "arn:aws:bedrock:us-east-2:123456789012:application-inference-profile/opus-41-prod"
  }
}
```

### 시작 모델 확인

Claude Code는 Bedrock 구성으로 시작할 때 사용하려는 모델이 계정에서 액세스 가능한지 확인합니다 (v2.1.94 이상 필요).

- 고정한 모델 버전이 Claude Code 기본값보다 오래되었고, 계정에서 최신 버전을 호출할 수 있는 경우 업데이트를 권장합니다.
- 모델을 고정하지 않았고 현재 기본값을 계정에서 사용할 수 없는 경우 이전 버전으로 폴백합니다 (현재 세션에만 적용).

### IAM 구성

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowModelAndInferenceProfileAccess",
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream",
        "bedrock:ListInferenceProfiles",
        "bedrock:GetInferenceProfile"
      ],
      "Resource": [
        "arn:aws:bedrock:*:*:inference-profile/*",
        "arn:aws:bedrock:*:*:application-inference-profile/*",
        "arn:aws:bedrock:*:*:foundation-model/*"
      ]
    },
    {
      "Sid": "AllowMarketplaceSubscription",
      "Effect": "Allow",
      "Action": [
        "aws-marketplace:ViewSubscriptions",
        "aws-marketplace:Subscribe"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:CalledViaLast": "bedrock.amazonaws.com"
        }
      }
    }
  ]
}
```

### 1M 토큰 컨텍스트 윈도우

Claude Opus 4.6 이상과 Sonnet 4.6은 Amazon Bedrock에서 1M 토큰 컨텍스트 윈도우를 지원합니다. Claude Code는 1M 모델 변형을 선택하면 자동으로 확장 컨텍스트 윈도우를 활성화합니다.

설정 위저드에서 모델을 고정할 때 1M 컨텍스트 옵션을 제공합니다. 수동으로 고정한 모델에 활성화하려면 모델 ID 끝에 `[1m]`을 추가하세요.

### 서비스 티어 (Service Tiers)

Amazon Bedrock 서비스 티어를 사용하면 비용과 지연 시간을 조절할 수 있습니다. `ANTHROPIC_BEDROCK_SERVICE_TIER`를 `default`, `flex`, 또는 `priority`로 설정합니다:

```bash
export ANTHROPIC_BEDROCK_SERVICE_TIER=priority
```

Claude Code는 이 값을 각 요청의 `X-Amzn-Bedrock-Service-Tier` 헤더로 전송합니다. 티어 가용성은 모델 및 리전에 따라 다릅니다. 예약 용량은 이 설정 대신 프로비저닝 처리량 ARN을 모델 ID로 사용합니다.

### AWS Guardrails

Amazon Bedrock Guardrails을 사용하여 Claude Code에 콘텐츠 필터링을 구현할 수 있습니다. Bedrock 콘솔에서 Guardrail을 생성하고 버전을 게시한 후, 설정 파일에 Guardrail 헤더를 추가합니다. 교차 리전 추론 프로파일을 사용하는 경우 Guardrail에서도 교차 리전 추론을 활성화하세요.

```json
{
  "env": {
    "ANTHROPIC_CUSTOM_HEADERS": "X-Amzn-Bedrock-GuardrailIdentifier: your-guardrail-id\nX-Amzn-Bedrock-GuardrailVersion: 1"
  }
}
```

### Mantle 엔드포인트

Mantle은 Amazon Bedrock 엔드포인트로, 네이티브 Anthropic API 형태로 Claude 모델을 제공합니다. 동일한 AWS 자격 증명, IAM 권한, `awsAuthRefresh` 구성을 사용합니다.

```bash
export CLAUDE_CODE_USE_MANTLE=1
export AWS_REGION=us-east-1
```

Mantle 모델 ID는 `anthropic.` 접두사를 사용하며 버전 접미사가 없습니다 (예: `anthropic.claude-haiku-4-5`).

```bash
claude --model anthropic.claude-haiku-4-5
```

Bedrock Invoke API와 Mantle을 동시에 사용하려면 두 환경 변수를 모두 설정합니다:

```bash
export CLAUDE_CODE_USE_BEDROCK=1
export CLAUDE_CODE_USE_MANTLE=1
```

Mantle 모델을 `/model` 피커에 표시하려면 설정 파일의 `availableModels`에 ID를 나열합니다:

```json
{
  "availableModels": ["opus", "sonnet", "haiku", "anthropic.claude-haiku-4-5"]
}
```

**Mantle 환경 변수:**

| 변수 | 용도 |
|------|------|
| `CLAUDE_CODE_USE_MANTLE` | Mantle 엔드포인트 활성화 (`1` 또는 `true`) |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | 기본 Mantle 엔드포인트 URL 재정의 |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | 프록시 설정에서 클라이언트 측 인증 건너뛰기 |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | Haiku급 모델의 AWS 리전 재정의 (Bedrock과 공유) |

### Bedrock 문제 해결

| 문제 | 해결 방법 |
|------|-----------|
| SSO/기업 프록시 인증 루프 | `awsAuthRefresh` 설정 제거. 수동으로 `aws sso login` 실행 |
| 리전 문제 | `aws bedrock list-inference-profiles --region your-region`으로 모델 가용성 확인. `export AWS_REGION=us-east-1`로 전환 |
| "on-demand throughput isn't supported" | 모델을 추론 프로파일 ID로 지정 |
| Mantle 403 오류 | AWS 계정에 해당 모델 액세스 권한이 없음. AWS 계정 팀에 문의 |
| Mantle 400 오류 (모델 ID 명시) | 해당 모델 ID는 Mantle에서 제공되지 않음. Mantle 형식 ID 사용 또는 두 엔드포인트 모두 활성화 |

> Claude Code는 Bedrock Invoke API를 사용하며 Converse API는 지원하지 않습니다.

---

## Google Vertex AI 연동

### 사전 요구사항

- 결제가 활성화된 Google Cloud Platform (GCP) 계정
- Vertex AI API가 활성화된 GCP 프로젝트
- 원하는 Claude 모델(예: Claude Sonnet 4.6)에 대한 액세스 권한
- Google Cloud SDK (`gcloud`) 설치 및 구성
- 원하는 GCP 리전에 할당된 할당량

### Sign-in 위저드

Google Cloud 자격 증명이 있고 Vertex AI를 통해 Claude Code를 시작하려는 경우, 로그인 위저드가 안내합니다. GCP 측 사전 요구사항은 프로젝트당 한 번만 완료하면 됩니다.

```bash
claude
# Vertex AI를 선택하면 위저드가 시작됩니다
```

로그인 후 `/setup-vertex`를 실행하여 언제든 위저드를 다시 열고 자격 증명, 프로젝트, 리전, 모델 핀을 변경할 수 있습니다.

### 리전 구성

Claude Code는 Vertex AI 글로벌, 멀티 리전, 리전 엔드포인트를 지원합니다. `CLOUD_ML_REGION`을 `global`, 멀티 리전 위치(`eu`, `us`), 또는 특정 리전(`us-east5`)으로 설정합니다.

### 수동 설정

#### 1. Vertex AI API 활성화

```bash
gcloud config set project YOUR-PROJECT-ID
gcloud services enable aiplatform.googleapis.com
```

#### 2. 모델 액세스 요청

1. Vertex AI Model Garden으로 이동
2. "Claude" 모델 검색
3. 원하는 Claude 모델의 액세스 요청
4. 승인 대기 (24-48시간 소요 가능)

#### 3. GCP 자격 증명 구성

Claude Code는 표준 Google Cloud 인증을 사용합니다.

```bash
gcloud auth application-default login
```

Claude Code v2.1.121 이상에서는 X.509 인증서 기반 Workload Identity Federation을 지원합니다. `GOOGLE_APPLICATION_CREDENTIALS`를 자격 증명 구성 파일 경로로 설정하세요.

**고급 자격 증명 새로고침 (gcpAuthRefresh):**

Claude Code는 GCP 자격 증명이 만료되었거나 로드할 수 없는 경우 `gcpAuthRefresh` 설정에 지정된 명령을 실행합니다.

```json
{
  "gcpAuthRefresh": "gcloud auth application-default login",
  "env": {
    "ANTHROPIC_VERTEX_PROJECT_ID": "your-project-id"
  }
}
```

명령의 출력은 사용자에게 표시되지만 대화형 입력은 지원되지 않습니다. 새로고침 명령은 인증이 3분 내에 완료되지 않으면 시간 초과됩니다. 프로젝트 설정(예: `.claude/settings.json`)에 설정한 경우 작업 영역 신뢰 프롬프트를 수락한 후에만 실행됩니다.

#### 4. Claude Code 구성

```bash
# Vertex AI 활성화
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=global
export ANTHROPIC_VERTEX_PROJECT_ID=YOUR-PROJECT-ID

# 선택: 커스텀 엔드포인트 또는 게이트웨이용 Vertex 엔드포인트 URL 재정의
# export ANTHROPIC_VERTEX_BASE_URL=https://aiplatform.googleapis.com

# 선택: 프롬프트 캐싱 비활성화
export DISABLE_PROMPT_CACHING=1

# 선택: 5분 기본값 대신 1시간 프롬프트 캐시 TTL 요청
export ENABLE_PROMPT_CACHING_1H=1

# CLOUD_ML_REGION=global인 경우, 글로벌 엔드포인트를 지원하지 않는 모델의 리전 재정의
export VERTEX_REGION_CLAUDE_HAIKU_4_5=us-east5
export VERTEX_REGION_CLAUDE_4_6_SONNET=europe-west1
```

대부분의 모델 버전에는 해당하는 `VERTEX_REGION_CLAUDE_*` 변수가 있습니다. Vertex Model Garden에서 글로벌 엔드포인트를 지원하는 모델과 리전 전용 모델을 확인하세요.

> Vertex AI 사용 시 `/logout` 명령을 사용할 수 없습니다.
>
> Claude Code는 Vertex AI에서 MCP 도구 검색을 기본적으로 비활성화합니다. Claude Sonnet 4.5 이상 및 Claude Opus 4.5 이상에서는 `ENABLE_TOOL_SEARCH=true`로 활성화할 수 있습니다.

#### 5. 모델 버전 고정

```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8'
export ANTHROPIC_DEFAULT_SONNET_MODEL='claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='claude-haiku-4-5@20251001'
```

`ANTHROPIC_DEFAULT_OPUS_MODEL`을 설정하지 않으면 `opus` 별칭은 Opus 4.6으로 해결됩니다. 최신 모델을 사용하려면 Opus 4.8 ID로 설정하세요.

**기본 모델 (고정 변수 미설정 시):**

| 모델 유형 | 기본값 |
|-----------|--------|
| Primary model | `claude-sonnet-4-5@20250929` |
| Small/fast model | Primary model과 동일 |

```bash
# 추가 커스터마이징
export ANTHROPIC_MODEL='claude-opus-4-8'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='claude-haiku-4-5@20251001'
```

### 시작 모델 확인

Claude Code는 Vertex AI 구성으로 시작할 때 모델이 프로젝트에서 액세스 가능한지 확인합니다 (v2.1.98 이상 필요).

### IAM 구성

`roles/aiplatform.user` 역할에 필요한 권한이 포함되어 있습니다:

- `aiplatform.endpoints.predict` — 모델 호출 및 토큰 계산에 필요

더 제한적인 권한이 필요한 경우 위 권한만 포함된 커스텀 역할을 생성하세요.

### 1M 토큰 컨텍스트 윈도우

Claude Opus 4.6 이상과 Sonnet 4.6은 Vertex AI에서 1M 토큰 컨텍스트 윈도우를 지원합니다. Claude Code는 1M 모델 변형을 선택하면 자동으로 확장 컨텍스트 윈도우를 활성화합니다. 수동으로 고정한 모델에 활성화하려면 모델 ID 끝에 `[1m]`을 추가하세요.

### Vertex AI 문제 해결

| 문제 | 해결 방법 |
|------|-----------|
| "Could not load the default credentials" | `gcloud auth application-default login` 실행 또는 `GOOGLE_APPLICATION_CREDENTIALS`를 서비스 계정 키 파일로 설정 |
| 할당량 문제 | Cloud Console에서 할당량 확인 또는 증가 요청 |
| "model not found" 404 | Model Garden에서 모델이 활성화되어 있는지 확인. 지정한 위치에서 모델이 제공되는지 확인 |
| 글로벌 엔드포인트 미지원 모델 404 | `ANTHROPIC_MODEL`로 지원 모델 지정 또는 `VERTEX_REGION_*` 변수로 리전 설정 |
| 429 오류 | `CLOUD_ML_REGION=global`로 전환하여 가용성 향상 고려 |

---

## Microsoft Foundry 연동

### 사전 요구사항

- Microsoft Foundry에 액세스할 수 있는 Azure 구독
- Microsoft Foundry 리소스 및 배포를 생성할 RBAC 권한
- Azure CLI 설치 및 구성 (선택 사항 — 다른 자격 증명 메커니즘이 있는 경우 불필요)

### 1. Microsoft Foundry 리소스 프로비저닝

1. Microsoft Foundry 포털로 이동
2. 새 리소스를 생성하고 리소스 이름을 기록
3. Claude 모델용 배포 생성:
   - Claude Opus
   - Claude Sonnet
   - Claude Haiku

### 2. Azure 자격 증명 구성

두 가지 인증 방법을 지원합니다.

**옵션 A: API 키 인증**

1. Microsoft Foundry 포털에서 리소스로 이동
2. __Endpoints and keys__ 섹션으로 이동
3. __API Key__ 복사
4. 환경 변수 설정:

```bash
export ANTHROPIC_FOUNDRY_API_KEY=your-azure-api-key
```

**옵션 B: Microsoft Entra ID 인증**

`ANTHROPIC_FOUNDRY_API_KEY`가 설정되지 않은 경우, Claude Code가 자동으로 Azure SDK 기본 자격 증명 체인을 사용합니다. 로컬 환경에서는 일반적으로 Azure CLI를 사용합니다:

```bash
az login
```

### 3. Claude Code 구성

```bash
# Microsoft Foundry 통합 활성화
export CLAUDE_CODE_USE_FOUNDRY=1

# Azure 리소스 이름 ({resource}를 리소스 이름으로 교체)
export ANTHROPIC_FOUNDRY_RESOURCE={resource}
# 또는 전체 base URL 제공:
# export ANTHROPIC_FOUNDRY_BASE_URL=https://{resource}.services.ai.azure.com/anthropic
```

> Bedrock 및 Vertex AI와 달리 Foundry에는 대화형 설정 위저드가 없으므로 위 환경 변수가 유일한 구성 방법입니다.

### 4. 모델 버전 고정

```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8'
export ANTHROPIC_DEFAULT_SONNET_MODEL='claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='claude-haiku-4-5'
```

`ANTHROPIC_DEFAULT_OPUS_MODEL`을 설정하지 않으면 `opus` 별칭은 Opus 4.6으로 해결됩니다. 최신 모델을 사용하려면 Opus 4.8 ID로 설정하세요.

백그라운드 작업(예: 세션 제목 생성)은 소형/고속 모델을 사용합니다. Foundry에서는 기본적으로 primary model로 설정되며, Haiku 배포를 사용하려면 `ANTHROPIC_DEFAULT_HAIKU_MODEL`을 설정하세요.

```bash
# 선택: 5분 기본값 대신 1시간 프롬프트 캐시 TTL 요청
export ENABLE_PROMPT_CACHING_1H=1
```

> 1시간 캐시 TTL은 5분 기본값보다 높은 요금이 청구됩니다.

### 5. Claude Code 실행

환경 변수 설정 후 프로젝트 디렉토리에서 Claude Code를 시작합니다:

```bash
claude
```

### Azure RBAC 구성

`Azure AI User` 및 `Cognitive Services User` 기본 역할에 필요한 모든 권한이 포함되어 있습니다. 더 제한적인 권한이 필요한 경우 다음 권한을 포함한 커스텀 역할을 생성하세요:

```json
{
  "permissions": [
    {
      "dataActions": [
        "Microsoft.CognitiveServices/accounts/providers/*"
      ]
    }
  ]
}
```

### Foundry 문제 해결

| 문제 | 해결 방법 |
|------|-----------|
| "Failed to get token from azureADTokenProvider: ChainedTokenCredential authentication failed" | 환경에 Entra ID를 구성하거나 `ANTHROPIC_FOUNDRY_API_KEY` 설정 |

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
