# Pulumi 설치 가이드

> https://www.pulumi.com/docs/install/
> https://www.pulumi.com/docs/install/versions/

Pulumi CLI는 macOS, Linux, Windows를 모두 지원하며 패키지 매니저, 설치 스크립트, 수동 설치 등 다양한 방법으로 설치할 수 있다. 최신 안정 버전은 **3.245.0**(2026-06-03 기준)이며, 기본적으로 Pulumi Cloud를 상태 관리 백엔드로 사용한다. Pulumi Cloud는 개인 사용자에게 무료이며 설치 시 계정이 필요하지 않다 — 최초 `pulumi login` 실행 시 로그인이 프롬프트된다.

---

## 시스템 요구사항

| 항목 | 최소 사양 |
|------|-----------|
| **CPU** | 2 GHz 이상 프로세서(또는 동등한 vCPU) |
| **RAM** | 4 GB 이상 |
| **디스크** | 1 GB 이상 여유 공간 (다중 런타임, 프로바이더, 대규모 코드베이스 사용 시 추가 필요) |
| **macOS** | macOS Ventura (13) 이상 |
| **Windows** | Windows 8 이상 |

> **참고:** 실제 요구사항은 사용하는 SDK 런타임, 프로바이더, 인프라 규모에 따라 달라질 수 있다. 여러 프로바이더나 대형 플러그인을 사용하면 추가 디스크 공간이 필요할 수 있다.

---

## 플랫폼별 설치 명령어 요약

| 플랫폼 | 방법 | 명령어 |
|--------|------|--------|
| **macOS** | Homebrew (권장) | `brew install pulumi/tap/pulumi` |
| **macOS** | 설치 스크립트 (curl) | `curl -fsSL https://get.pulumi.com \| sh` |
| **macOS** | MacPorts | `sudo port install pulumi` |
| **macOS** | 수동 설치 | [darwin-x64](https://get.pulumi.com/releases/sdk/pulumi-v3.245.0-darwin-x64.tar.gz) / [darwin-arm64](https://get.pulumi.com/releases/sdk/pulumi-v3.245.0-darwin-arm64.tar.gz) 다운로드 후 `$PATH`에 추가 |
| **Linux** | 설치 스크립트 (curl) | `curl -fsSL https://get.pulumi.com \| sh` |
| **Linux** | 수동 설치 | [linux-x64](https://get.pulumi.com/releases/sdk/pulumi-v3.245.0-linux-x64.tar.gz) 다운로드 후 `$PATH`에 추가 |
| **Windows** | Chocolatey | `choco install pulumi` |
| **Windows** | winget | `winget install pulumi` |
| **Windows** | MSI 인스톨러 | [pulumi-3.245.0-windows-x64.msi](https://github.com/pulumi/pulumi-winget/releases/download/v3.245.0/pulumi-3.245.0-windows-x64.msi) 다운로드 후 실행 |
| **Windows** | 설치 스크립트 (PowerShell) | 아래 Windows 섹션 참조 |
| **Windows** | 수동 설치 | [windows-x64](https://get.pulumi.com/releases/sdk/pulumi-v3.245.0-windows-x64.zip) 다운로드 후 `$PATH`에 추가 |

---

## macOS

### Homebrew (권장)

공식 Pulumi Homebrew Tap을 사용하는 방법이다. CLI가 `/usr/local/bin/pulumi`에 설치되며 자동으로 `$PATH`에 추가된다.

```bash
# 설치
$ brew install pulumi/tap/pulumi

# 업그레이드
$ brew upgrade pulumi
```

### 커뮤니티 Homebrew

Pulumi Tap이 없어도 커뮤니티 Homebrew formula로 설치할 수 있다.

```bash
$ brew install pulumi
```

### MacPorts

MacPorts 패키지 매니저를 통해서도 설치 가능하다. CLI는 `/opt/local/bin/pulumi`에 설치된다.

```bash
# 설치
$ sudo port install pulumi

# 업그레이드
$ sudo port upgrade outdated
```

### 설치 스크립트

curl로 설치 스크립트를 실행하면 `~/.pulumi/bin`에 CLI가 설치되고 자동으로 `$PATH`에 추가된다. 자동 추가가 실패하면 수동으로 `$PATH`를 설정해야 한다.

```bash
$ curl -fsSL https://get.pulumi.com | sh
```

스크립트를 재실행하면 새 버전으로 업그레이드할 수 있다.

### 수동 설치

바이너리를 직접 다운로드하여 압축을 푼 뒤 `$PATH`가 포함된 디렉터리로 이동시킨다.

| 아키텍처 | 다운로드 |
|-----------|----------|
| amd64 (x64) | [pulumi-v3.245.0-darwin-x64.tar.gz](https://get.pulumi.com/releases/sdk/pulumi-v3.245.0-darwin-x64.tar.gz) |
| arm64 (Apple Silicon) | [pulumi-v3.245.0-darwin-arm64.tar.gz](https://get.pulumi.com/releases/sdk/pulumi-v3.245.0-darwin-arm64.tar.gz) |

---

## Linux

### 설치 스크립트 (권장)

curl로 설치 스크립트를 실행한다. CLI는 `~/.pulumi/bin`에 설치된다.

```bash
$ curl -fsSL https://get.pulumi.com | sh
```

스크립트가 `$PATH`를 자동 추가하지 못하는 경우, 수동으로 설정해야 한다. 영구 설정 방법은 [How to permanently set $PATH on Unix](https://stackoverflow.com/questions/14637979/how-to-permanently-set-path-on-linux-unix)를 참조한다.

스크립트를 재실행하면 새 버전으로 업그레이드할 수 있다.

### 수동 설치

바이너리를 직접 다운로드하여 압축을 푼 뒤 `$PATH`가 포함된 디렉터리로 이동시킨다.

| 아키텍처 | 다운로드 |
|-----------|----------|
| amd64 (x64) | [pulumi-v3.245.0-linux-x64.tar.gz](https://get.pulumi.com/releases/sdk/pulumi-v3.245.0-linux-x64.tar.gz) |

---

## Windows

### Chocolatey

관리자 권한으로 Chocolatey를 통해 설치한다. CLI는 `$($env:ChocolateyInstall)\lib\pulumi`에 설치되고 shim을 통해 `$PATH`에 추가된다.

```powershell
# 설치
> choco install pulumi

# 업그레이드
> choco upgrade pulumi
```

### Windows Package Manager (winget)

Windows 11 이상에 내장된 winget으로 설치할 수 있다.

```powershell
# 설치
> winget install pulumi

# 업그레이드
> winget upgrade pulumi
```

### MSI 인스톨러

MSI 인스톨러를 다운로드하여 실행하면 시스템 전체에 자동으로 설치되고 `$PATH`에 추가된다.

- [pulumi-3.245.0-windows-x64.msi](https://github.com/pulumi/pulumi-winget/releases/download/v3.245.0/pulumi-3.245.0-windows-x64.msi)

### 설치 스크립트 (PowerShell)

명령 프롬프트(`cmd.exe`)를 열고 다음 명령을 실행한다. CLI는 `%USERPROFILE%\.pulumi\bin`에 설치된다.

```powershell
> @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; iex ((New-Object System.Net.WebClient).DownloadString('https://get.pulumi.com/install.ps1'))" && SET "PATH=%PATH%;%USERPROFILE%\.pulumi\bin"
```

### 수동 설치

바이너리 ZIP을 다운로드하여 원하는 폴더(예: `C:\pulumi`)에 압축을 푼 뒤, 시스템 속성에서 환경 변수 `Path`에 `C:\pulumi\bin`을 추가한다.

| 아키텍처 | 다운로드 |
|-----------|----------|
| x64 | [pulumi-v3.245.0-windows-x64.zip](https://get.pulumi.com/releases/sdk/pulumi-v3.245.0-windows-x64.zip) |

---

## GitHub Actions에서 설치

> **참고:** 이 섹션은 공식 `/docs/install/` 페이지가 아닌 [GitHub Actions 가이드](https://www.pulumi.com/docs/continuous-delivery/github-actions/)에서 다루는 내용이다. 참조의 편의를 위해 이곳에 요약해둔다.

Pulumi는 공식 GitHub Actions를 제공하여 CI/CD 파이프라인에서 CLI를 설치하고 실행할 수 있다.

### 주요 액션

| 액션 | 용도 |
|------|------|
| [`pulumi/actions`](https://github.com/pulumi/actions) | Pulumi CLI 설치 후 `preview`, `up`, `destroy` 등의 명령 실행. **설치만 필요한 경우** [installation-only 모드](https://github.com/pulumi/actions#installation-only) 사용 |
| [`pulumi/auth-actions`](https://github.com/pulumi/auth-actions) | GitHub OIDC 토큰을 Pulumi Cloud 액세스 토큰으로 교환 |
| [`pulumi/esc-action`](https://github.com/pulumi/esc-action) | Pulumi ESC 환경의 변수를 워크플로에 주입 |

> **주의:** `pulumi/setup-pulumi` 액션은 **deprecated** 되었다. CLI만 설치하려면 `pulumi/actions`의 [installation-only 모드](https://github.com/pulumi/actions#installation-only)를 사용해야 한다.

### 기본 워크플로 예시

**TypeScript** (`pr.yml` — Pull Request 시 `preview` 실행):

```yaml
name: Pulumi preview
on:
  pull_request:
jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: infra/package.json
      - run: npm install
        working-directory: infra
      - uses: pulumi/actions@v7
        with:
          command: preview
          stack-name: acme/website/staging
          work-dir: infra
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
```

**Python** (`pr.yml` — Pull Request 시 `preview` 실행):

```yaml
name: Pulumi preview
on:
  pull_request:
jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install -r requirements.txt
        working-directory: infra
      - uses: pulumi/actions@v7
        with:
          command: preview
          stack-name: acme/website/staging
          work-dir: infra
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
```

인증 방식은 두 가지가 있다.

| 인증 방식 | 설명 |
|-----------|------|
| **액세스 토큰** | `PULUMI_ACCESS_TOKEN`을 GitHub Secrets에 저장. 가장 간단한 방법 |
| **OIDC 토큰 교환** | `pulumi/auth-actions`로 단기 OIDC 토큰을 교환. 저장된 비밀이 필요 없음 (권장) |

---

## AI 코딩 어시스턴트에 Agent Skills 설치

Skills는 오픈 [Agent Skills](https://agentskills.io) 사양을 따르는 구조화된 지식 패키지이다. Claude Code, GitHub Copilot, Cursor, VS Code, Codex, Gemini CLI 등 여러 AI 코딩 플랫폼에서 작동한다. Pulumi Skills를 설치하면 AI 어시스턴트가 일반적인 인프라 작업에 대한 상세한 워크플로, 코드 패턴 및 의사결정 트리에 액세스할 수 있다.

### Claude Code Plugin Marketplace

Claude Code 사용자의 경우 플러그인 시스템이 가장 간단한 설치 환경을 제공한다.

```bash
claude plugin marketplace add pulumi/agent-skills
claude plugin install pulumi-migration      # 마이그레이션 스킬 설치
claude plugin install pulumi                # Pulumi 스킬 설치 (개요 + 전문)
claude plugin install pulumi-delegation     # 위임 스킬 설치 (Neo handoff)
```

모든 플러그인 그룹을 설치하거나 필요한 것만 선택적으로 설치할 수 있다.

### Universal 설치

Cursor, GitHub Copilot, VS Code, Codex, Gemini 및 기타 플랫폼의 경우 범용 [Agent Skills](https://agentskills.io) CLI를 사용한다.

```bash
npx skills add pulumi/agent-skills --skill '*'
```

---

## 특정 버전 및 개발 버전 설치

### 특정 버전 설치

대부분의 설치 방법은 기본적으로 최신 버전을 설치한다. 특정 버전을 설치하려면 다음 명령을 사용한다.

**macOS / Linux:**

```bash
$ curl -fsSL https://get.pulumi.com | sh -s -- --version <VERSION>
```

**Windows (Chocolatey):**

```powershell
> choco install pulumi --version <VERSION>
```

**Windows (설치 스크립트):**

```powershell
> @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; $version = '<VERSION>'; iex ((New-Object System.Net.WebClient).DownloadString('https://get.pulumi.com/install.ps1')).Replace('${Version}', $version)" && SET "PATH=%PATH%;%USERPROFILE%\.pulumi\bin"
```

### 개발(dev) 버전 설치

최신 개발 브랜치의 변경 사항이 포함된 dev 버전을 설치할 수 있다.

**macOS / Linux:**

```bash
$ curl -fsSL https://get.pulumi.com | sh -s -- --version dev
```

**Windows (설치 스크립트):**

```powershell
> @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; & ([ScriptBlock]::Create((New-Object System.Net.WebClient).DownloadString('https://get.pulumi.com/install.ps1'))) -Version dev" && SET "PATH=%PATH%;%USERPROFILE%\.pulumi\bin"
```

---

## 설치 확인 및 버전 확인

설치가 완료되면 다음 명령으로 정상 동작을 확인한다.

```bash
$ pulumi version
```

### 일반적인 에러 및 해결

| 에러 | 원인 | 해결 방법 |
|------|------|-----------|
| `pulumi: command not found` | `$PATH`에 CLI 경로가 포함되지 않음 | 설치 경로가 `$PATH`에 포함되어 있는지 확인 |
| 새 버전 경고 | 최신 버전이 아님 | 설치 스크립트 재실행 또는 패키지 매니저로 업그레이드 |

> 인터넷 접근이 불가능한 환경에서는 `PULUMI_SKIP_UPDATE_CHECK` 환경 변수를 `1` 또는 `true`로 설정하면 버전 업데이트 확인을 건너뛸 수 있다.

---

## 설치 후 초기 설정 (login, 자격증명)

### Pulumi Cloud에 로그인

Pulumi CLI는 기본적으로 Pulumi Cloud를 상태 관리 백엔드로 사용한다. 설치 후 최초 로그인은 다음 명령으로 수행한다.

```bash
$ pulumi login
```

브라우저를 통한 인증 프롬프트가 표시된다. Enter 키를 누르면 브라우저가 열리고, 이미 액세스 토큰이 있다면 직접 입력할 수도 있다.

```
Manage your Pulumi stacks by logging in.
Run `pulumi login --help` for alternative login options.
Enter your access token from https://app.pulumi.com/account/tokens
    or hit <ENTER> to log in using your browser:
```

### 액세스 토큰으로 로그인 (스크립트용)

CI/CD나 스크립트 환경에서는 `PULUMI_ACCESS_TOKEN` 환경 변수를 설정하여 비대화형으로 로그인할 수 있다.

```bash
$ export PULUMI_ACCESS_TOKEN=<YOUR_ACCESS_TOKEN>
$ pulumi login
```

> **참고:** `pulumi login` 명령어는 `--cloud-url`, `--local`, `--oidc-token` 등의 플래그를 제공하지만, `--token` 플래그는 공식 문서에 명시되어 있지 않다. 스크립트 환경에서는 `PULUMI_ACCESS_TOKEN` 환경 변수를 사용하는 것이 권장된다.

### 자체 호스팅 Pulumi Cloud에 로그인

사설 인스턴스를 사용하는 경우 API URL을 인자로 전달한다.

```bash
$ pulumi login https://pulumi.acmecorp.com
```

### DIY 백엔드 사용

Pulumi Cloud 대신 자체 관리하는 백엔드(S3, Azure Blob, GCS, 로컬 파일시스템 등)를 사용할 수 있다.

```bash
# 로컬 파일시스템 백엔드
$ pulumi login --local

# 특정 백엔드 URL 지정
$ pulumi login <BACKEND_URL>
```

백엔드 URL은 환경 변수나 `Pulumi.yaml` 설정으로도 지정할 수 있다.

```yaml
# Pulumi.yaml
backend:
  url: <BACKEND_URL>
```

### 로그인 상태 확인

```bash
$ pulumi whoami -v
User: <your-username>
Backend URL: https://app.pulumi.com/<your-username>
```

### 로그아웃

```bash
$ pulumi logout
```

`--all` 플래그를 사용하면 모든 백엔드의 자격 증명을 삭제할 수 있다.

```bash
$ pulumi logout --all
```

> 로그인 자격 증명은 `~/.pulumi/credentials.json`에 저장된다. 백엔드를 변경하려면 `pulumi logout` 후 다시 `pulumi login`을 실행한다.

---

## 버전 히스토리

최신 안정 버전은 **3.245.0**(2026-06-03)이다. 전체 버전 목록은 [Available versions](https://www.pulumi.com/docs/install/versions/) 페이지에서 확인할 수 있다. 주요 최근 버전은 다음과 같다.

| 버전 | 날짜 |
|------|------|
| 3.245.0 | 2026-06-03 |
| 3.244.0 | 2026-05-28 |
| 3.243.0 | 2026-05-22 |
| 3.242.0 | 2026-05-19 |

각 버전의 상세 변경 사항은 [CHANGELOG](https://github.com/pulumi/pulumi/blob/master/CHANGELOG.md)에서 확인할 수 있다. 모든 버전은 Linux x64/arm64, macOS x64/arm64, Windows x64/arm64 바이너리를 제공한다.

---

## Pulumi 제거

설치에 사용한 방법에 따라 제거한다. 수동 설치한 경우 `pulumi` 디렉터리를 삭제한다. 이후 홈 디렉터리의 `.pulumi` 폴더(플러그인 및 캐시 메타데이터 포함)도 함께 삭제한다.
