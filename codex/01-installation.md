# Codex CLI - 설치 및 설정

> 시스템 요구사항, 설치 방법, 인증 설정, 설정 파일 위치, 트러블슈팅 가이드

**참조**: [developers.openai.com/codex/quickstart](https://developers.openai.com/codex/quickstart) | [github.com/openai/codex - docs/install.md](https://github.com/openai/codex/blob/main/docs/install.md)

---

## 시스템 요구사항

| 요구사항 | 상세 |
| --- | --- |
| **운영체제** | macOS 12+, Ubuntu 20.04+ / Debian 10+, Windows 11 (WSL2 필요) |
| **Git (권장)** | 2.23+ (내장 PR 헬퍼 사용 시) |
| **RAM** | 최소 4GB (권장 8GB) |
| **Node.js** | npm 설치 시 22+ 필요 |

> **참고**: Windows는 WSL2를 통해서만 지원됩니다. 네이티브 Windows 지원은 별도 가이드를 참조하세요.

---

## 설치 방법

### 방법 1: 공식 설치 스크립트 (권장)

**macOS / Linux**:

```shell
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

**Windows** (PowerShell):

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
```

**무인 설치** (CI/CD 환경):

```shell
# macOS / Linux
curl -fsSL https://chatgpt.com/codex/install.sh | CODEX_NON_INTERACTIVE=1 sh

# Windows PowerShell
$env:CODEX_NON_INTERACTIVE=1; irm https://chatgpt.com/codex/install.ps1 | iex
```

### 방법 2: npm

```shell
npm install -g @openai/codex
```

Node.js 22+가 필요합니다. Node.js가 설치되어 있지 않다면 [nodejs.org](https://nodejs.org)에서 설치하세요.

### 방법 3: Homebrew (macOS)

```shell
brew install --cask codex
```

### 방법 4: GitHub Release 바이너리

[최신 릴리스](https://github.com/openai/codex/releases/latest)에서 플랫폼에 맞는 바이너리를 다운로드합니다.

| 플랫폼 | 파일명 |
| --- | --- |
| macOS Apple Silicon | `codex-aarch64-apple-darwin.tar.gz` |
| macOS Intel (x86_64) | `codex-x86_64-apple-darwin.tar.gz` |
| Linux x86_64 | `codex-x86_64-unknown-linux-musl.tar.gz` |
| Linux arm64 | `codex-aarch64-unknown-linux-musl.tar.gz` |

압축 해제 후 바이너리 이름을 `codex`로 변경하여 PATH에 추가합니다.

### 방법 5: 소스에서 빌드 (Rust)

```shell
# 리포지토리 클론
git clone https://github.com/openai/codex.git
cd codex/codex-rs

# Rust 툴체인 설치
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustup component add rustfmt clippy

# 빌드 도구 설치
cargo install --locked just
cargo install --locked cargo-nextest  # 선택사항

# 빌드
cargo build

# 실행
cargo run --bin codex -- "이 코드베이스를 설명해줘"
```

---

## API 키 설정

### 방법 1: 환경변수

```shell
export OPENAI_API_KEY="sk-..."
```

영구적으로 설정하려면 셸 설정 파일(`~/.zshrc`, `~/.bashrc` 등)에 추가합니다:

```shell
echo 'export OPENAI_API_KEY="sk-..."' >> ~/.zshrc
source ~/.zshrc
```

### 방법 2: ChatGPT 계정 로그인

```shell
codex
```

실행 후 **Sign in with ChatGPT**를 선택합니다. 다음 플랜이 지원됩니다:

| 플랜 | 설명 |
| --- | --- |
| ChatGPT Plus | 개인 사용자 |
| ChatGPT Pro | 고급 사용자 |
| ChatGPT Business | 팀/비즈니스 |
| ChatGPT Edu | 교육 기관 |
| ChatGPT Enterprise | 기업 |

> API 키 로그인 시 일부 기능이 제한될 수 있습니다.

---

## 설정 파일 위치

### 글로벌 설정

| 파일 | 경로 | 설명 |
| --- | --- | --- |
| config.toml | `~/.codex/config.toml` | 사용자 글로벌 설정 |
| managed_config.toml | `~/.codex/managed_config.toml` | 엔터프라이즈 관리 설정 (우선순위 높음) |

### 프로젝트 설정

| 파일 | 경로 | 설명 |
| --- | --- | --- |
| config.toml | `.codex/config.toml` | 프로젝트별 설정 |

### 우선순위 (높은 순)

```
managed_config.toml > CLI 플래그 > .codex/config.toml > ~/.codex/config.toml
```

---

## 세션 데이터 위치

### 기본 경로

- 세션 홈: `$CODEX_HOME` (기본값: `~/.codex`)
- 세션 기록: `$CODEX_HOME/sessions/`
- 설정 파일: `$CODEX_HOME/config.toml`

### 환경변수로 변경

```shell
export CODEX_HOME="/custom/path/to/codex-home"
```

---

## 로그 및 디버깅

### 로그 위치

| 항목 | 경로 |
| --- | --- |
| TUI 로그 | `~/.codex/log/codex-tui.log` |
| 세션 기록 | `~/.codex/sessions/` |

### 로그 확인

```shell
tail -F ~/.codex/log/codex-tui.log
```

### 로그 수준 조정

`RUST_LOG` 환경변수로 제어합니다:

```shell
# 상세 로그
export RUST_LOG=codex_core=debug,codex_tui=debug

# 기본값
export RUST_LOG=codex_core=info,codex_tui=info,codex_rmcp_client=info
```

비대화형 모드(`codex exec`)의 기본값은 `RUST_LOG=error`이며, 인라인으로 출력됩니다.

### 로그 디렉토리 변경

```shell
codex -c log_dir=./.codex-log
```

---

## 셸 자동완성 설정

```shell
# Bash
codex completion bash >> ~/.bashrc

# Zsh
echo 'eval "$(codex completion zsh)"' >> ~/.zshrc
# compdef 오류 발생 시:
echo 'autoload -Uz compinit && compinit' >> ~/.zshrc

# Fish
codex completion fish > ~/.config/fish/completions/codex.fish
```

---

## 업데이트 방법

### 설치 스크립트로 설치한 경우

```shell
# 최신 버전으로 재설치
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

### npm으로 설치한 경우

```shell
npm update -g @openai/codex
```

### Homebrew로 설치한 경우

```shell
brew upgrade --cask codex
```

### 버전 확인

```shell
codex --version
```

---

## 트러블슈팅 기본

### 일반적인 문제 해결

| 문제 | 해결 방법 |
| --- | --- |
| `command not found: codex` | PATH에 설치 경로가 포함되어 있는지 확인. npm 설치 시 `-g` 플래그 사용 |
| 인증 실패 | `OPENAI_API_KEY`가 올바른지 확인. `codex` 실행 후 ChatGPT 로그인 재시도 |
| 권한 오류 | `~/.codex/` 디렉토리 권한 확인: `chmod -R 755 ~/.codex` |
| 샌드박스 오류 | `--sandbox` 플래그로 모드 변경 시도 |
| macOS 보안 경고 | `xattr -d com.apple.quarantine $(which codex)` 로 격리 속성 제거 |

### 설정 진단

```shell
# 설정 계층 및 정책 요구사항 디버그
codex --help

# 세션 내에서 설정 상태 확인
codex
> /status

# 설정 레이어 디버그
> /debug-config
```

### 로그 분석

```shell
# 최근 에러 로그 확인
grep -i error ~/.codex/log/codex-tui.log | tail -20

# 세션 기록 확인
ls -lt ~/.codex/sessions/ | head -10
```

### 세션 복구

```shell
# 이전 세션 목록 확인
codex resume

# 특정 세션 재개
codex resume <SESSION_ID>

# 가장 최근 세션 재개
codex resume --last
```

---

## macOS 로그 위치 (Native 앱)

Native macOS 앱을 사용하는 경우 로그는 다음 위치에 저장됩니다:

```
~/Library/Logs/com.openai.codex/
```

---

> **최종 업데이트**: 2026-06-05
> **출처**: [developers.openai.com/codex/quickstart](https://developers.openai.com/codex/quickstart), [github.com/openai/codex - docs/install.md](https://github.com/openai/codex/blob/main/docs/install.md)
