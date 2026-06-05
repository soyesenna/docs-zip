# 13. 보안 (Security)

> **원문**: [Security](https://code.claude.com/docs/en/security) | [Sandboxing](https://code.claude.com/docs/en/sandboxing) | [Sandbox Environments](https://code.claude.com/docs/en/sandbox-environments)
>
> **참조(구)**: [Security - Anthropic](https://docs.anthropic.com/en/docs/claude-code/security)

---

## 목차

- [보안 기본 원칙](#보안-기본-원칙)
- [권한 기반 아키텍처](#권한-기반-아키텍처)
- [내장 보호 기능](#내장-보호-기능)
- [프롬프트 인젝션 방지](#프롬프트-인젝션-방지)
- [개인정보 보호 조치](#개인정보-보호-조치)
- [MCP 보안](#mcp-보안)
- [IDE 보안](#ide-보안)
- [클라우드 실행 보안](#클라우드-실행-보안)
- [샌드박싱](#샌드박싱)
- [샌드박스 환경 비교](#샌드박스-환경-비교)
- [보안 모범 사례](#보안-모범-사례)

---

## 보안 기본 원칙

Claude Code는 보안을 핵심으로 설계되었습니다. Anthropic의 포괄적인 보안 프로그램에 따라 개발되었습니다.

### 인증 및 규정 준수

| 인증 | 설명 |
|------|------|
| **SOC 2 Type 2** | 서비스 조직 통제 보고서 |
| **ISO 27001** | 정보보호관리체계 인증 |

자세한 내용은 [Anthropic Trust Center](https://www.anthropic.com/trust-center)에서 확인할 수 있습니다.

---

## 권한 기반 아키텍처

Claude Code는 **기본적으로 읽기 전용** 권한을 사용합니다.

### 핵심 원칙

| 원칙 | 설명 |
|------|------|
| **기본 읽기 전용** | 추가 작업이 필요한 경우에만 쓰기 권한 요청 |
| **명시적 권한 요청** | 파일 편집, 테스트 실행, 명령 실행 시 반드시 권한 요청 |
| **사용자 제어** | 사용자가 동작을 한 번 승인할지, 자동으로 허용할지 결정 |
| **Bash 승인** | Bash 명령은 실행 전 반드시 승인 필요 |

### 투명한 설계

Claude Code는 투명하고 안전하게 설계되었습니다. 예를 들어, Bash 명령은 실행 전 승인을 요구하여 사용자가 직접 제어할 수 있습니다.

### 권한 구성

사용자와 조직은 권한을 직접 구성할 수 있습니다. 자세한 권한 구성은 Identity and Access Management 문서를 참조하세요.

---

## 내장 보호 기능

에이전트 시스템의 위험을 완화하기 위한 내장 보호 기능입니다.

### Sandboxed Bash Tool

Bash 명령을 파일시스템 및 네트워크 격리와 함께 샌드박스에서 실행하여, 권한 프롬프트를 줄이면서 보안을 유지합니다. `/sandbox` 명령으로 활성화하며, Claude Code가 자율적으로 작업할 수 있는 경계를 정의합니다. 자세한 구성은 [샌드박싱](#샌드박싱) 섹션을 참조하세요.

### 쓰기 접근 제한

| 항목 | 설명 |
|------|------|
| **쓰기 범위** | Claude Code는 시작된 폴더와 하위 폴더에만 쓰기 가능 |
| **상위 디렉토리** | 명시적 권한 없이 상위 디렉토리의 파일 수정 불가 |
| **읽기 범위** | 작업 디렉토리 외부의 파일은 읽기 가능 (시스템 라이브러리, 종속성 접근에 유용) |
| **보안 경계** | 쓰기 작업은 프로젝트 범위로 엄격히 제한되어 명확한 보안 경계 생성 |

### 프롬프트 피로 완화

자주 사용하는 안전한 명령을 사용자별, 코드베이스별, 조직별로 허용 목록(Allowlist)에 등록할 수 있습니다.

### Accept Edits 모드

파일 편집과 작업 디렉토리 내 경로에 대한 고정된 파일시스템 Bash 명령(`mkdir`, `touch`, `rm`, `mv`, `cp`, `sed` 등)을 자동 승인합니다. 다른 Bash 명령과 범위 밖 경로는 여전히 권한 프롬프트를 표시합니다.

### 안전한 자격 증명 저장소

API 키와 토큰은 암호화되어 저장됩니다. 자세한 내용은 Credential Management 문서를 참조하세요.

### 사용자 책임

Claude Code는 사용자가 부여한 권한만 가집니다. 승인 전에 제안된 코드와 명령의 안전성을 검토하는 것은 **사용자의 책임**입니다.

---

## 프롬프트 인젝션 방지

프롬프트 인젝션은 공격자가 악의적인 텍스트를 삽입하여 AI 어시스턴트의 지침을 재정의하거나 조작하려는 기술입니다.

### 핵심 보호 기능

| 보호 기능 | 설명 |
|----------|------|
| **권한 시스템** | 민감한 작업에는 명시적 승인 필요 |
| **컨텍스트 인식 분석** | 전체 요청을 분석하여 잠재적으로 유해한 지침 감지 |
| **입력 정제** | 사용자 입력을 처리하여 명령 인젝션 방지 |
| **명령 블록리스트** | `curl`, `wget` 등 웹에서 임의 콘텐츠를 가져오는 위험한 명령 차단 |

### 추가 보호 기능

| 보호 기능 | 설명 |
|----------|------|
| **네트워크 요청 승인** | 네트워크 요청을 수행하는 도구는 기본적으로 사용자 승인 필요 |
| **격리된 컨텍스트 윈도우** | Web Fetch는 별도의 컨텍스트 윈도우를 사용하여 악의적인 프롬프트 주입 방지 |
| **신뢰 검증** | 처음 실행하는 코드베이스와 새 MCP 서버는 신뢰 검증 필요 |
| **명령 인젝션 감지** | 의심스러운 Bash 명령은 이전에 허용 목록에 있어도 수동 승인 필요 |
| **Fail-closed 매칭** | 일치하지 않는 명령은 수동 승인 필요 (기본 거부) |
| **자연어 설명** | 복잡한 Bash 명령에 사용자 이해를 돕는 설명이 포함됨 |
| **안전한 자격 증명 저장소** | API 키와 토큰은 암호화되어 저장됨. Credential Management 참조 |

> **참고**: `-p` 플래그로 비대화형 모드로 실행할 때는 신뢰 검증이 비활성화됩니다. 단, `--worktree`는 예외로, 해당 디렉토리에 대해 신뢰가 이미 수락되어 있어야 합니다.
>
> **참고**: 홈 디렉토리에서 Claude Code를 직접 시작하면 신뢰 수락이 현재 세션에만 유지되며 디스크에 기록되지 않아, 실행할 때마다 프롬프트가 다시 나타납니다. 이 동작을 영구적으로 변경하는 설정은 없습니다. 대신 프로젝트 하위 디렉토리에서 시작하면 디렉토리별로 신뢰 수락이 저장됩니다.

### 신뢰할 수 없는 콘텐츠 작업 모범 사례

1. 승인 전에 제안된 명령을 검토
2. 신뢰할 수 없는 콘텐츠를 Claude에 직접 파이프하지 않기
3. 중요 파일에 대한 변경사항 확인
4. 특히 외부 웹 서비스와 상호작용할 때 가상 머신(VM)에서 스크립트 및 도구 호출 실행
5. 의심스러운 동작은 `/feedback`으로 보고

---

## 개인정보 보호 조치

| 조치 | 설명 |
|------|------|
| **민감 정보 보유 기간 제한** | 민감한 정보의 보유 기간이 제한되어 있음 |
| **세션 데이터 접근 제한** | 사용자 세션 데이터에 대한 접근이 제한됨 |
| **데이터 학습 선호도 제어** | 사용자가 데이터 학습 여부를 제어할 수 있음. 소비자 사용자는 언제든지 개인정보 설정 변경 가능 |

자세한 내용은 Anthropic의 이용약관 및 개인정보 보호정책을 참조하세요.

---

## MCP 보안

### MCP 서버 구성

| 항목 | 설명 |
|------|------|
| **구성 위치** | 허용된 MCP 서버 목록은 소스 코드의 Claude Code 설정에 구성 |
| **버전 관리** | 설정 파일을 소스 컨트롤에 체크인 |
| **권한 구성** | MCP 서버에 대한 Claude Code 권한을 구성할 수 있음 |

### 보안 권장사항

| 권장사항 | 설명 |
|----------|------|
| **직접 작성 또는 신뢰할 수 있는 제공자 사용** | MCP 서버는 직접 작성하거나 신뢰할 수 있는 제공자의 것만 사용 |
| **Anthropic은 MCP 서버를 관리/감사하지 않음** | MCP 서버의 보안은 사용자의 책임 |
| **권한 구성** | MCP 서버에 대한 권한을 적절히 설정 |

---

## IDE 보안

IDE에서 Claude Code를 실행하는 방법에 대한 자세한 내용은 VS Code 보안 및 개인정보 보호 문서를 참조하세요.

> **참고**: IDE 보안에 대한 자세한 내용은 [VS Code security and privacy](https://code.claude.com/docs/en/ide-security)를 참조하세요.

---

## 클라우드 실행 보안

Claude Code on the web을 사용할 때 추가적인 보안 제어가 적용됩니다.

### 클라우드 보안 제어

| 제어 | 설명 |
|------|------|
| **격리된 가상 머신** | 각 클라우드 세션은 격리된 Anthropic 관리 VM에서 실행 |
| **네트워크 접근 제어** | 기본적으로 네트워크 접근이 제한되며, 비활성화하거나 특정 도메인만 허용하도록 구성 가능 |
| **자격 증명 보호** | 인증은 보안 프록시를 통해 처리되며, 샌드박스 내부에서는 스코프가 제한된 자격 증명을 사용하고 이를 실제 GitHub 인증 토큰으로 변환 |
| **브랜치 제한** | Git push 작업은 현재 작업 브랜치로만 제한됨 |
| **감사 로깅** | 클라우드 환경의 모든 작업이 규정 준수 및 감사 목적으로 로깅됨 |
| **자동 정리** | 세션 완료 후 클라우드 환경이 자동으로 종료됨 |

### Remote Control 세션

Remote Control 세션은 다르게 동작합니다. 웹 인터페이스가 로컬 머신에서 실행 중인 Claude Code 프로세스에 연결됩니다. 모든 코드 실행과 파일 접근은 로컬에 유지되며, 로컬 Claude Code 세션 중에 흐르는 것과 동일한 데이터가 TLS를 통해 Anthropic API로 전송됩니다. 클라우드 VM이나 샌드박싱은 관여하지 않습니다. 연결은 여러 개의 수명이 짧고 스코프가 좁은 자격 증명을 사용하며, 각각 특정 목적으로 제한되고 독립적으로 만료되어 단일 자격 증명이 손상될 경우의 영향 범위를 제한합니다.

> 자세한 내용은 Claude Code on the web 문서를 참조하세요.

---

## 샌드박싱

Bash 샌드박스는 Claude가 대부분의 셸 명령을 권한 요청 없이 실행할 수 있게 합니다. 대신 명령이 접근할 수 있는 파일과 네트워크 도메인을 정의하면, 운영체제가 모든 Bash 명령과 그 자식 프로세스에 대해 해당 경계를 강제합니다.

### 시작하기

샌드박스는 Claude Code에 내장되어 있으며 macOS, Linux, WSL2에서 실행됩니다. Native Windows는 지원되지 않으며, Windows에서는 WSL2 배포 내에서 실행해야 합니다.

| 플랫폼 | 프레임워크 | 추가 설치 |
|--------|-----------|-----------|
| **macOS** | Seatbelt (내장) | 없음 |
| **Linux / WSL2** | bubblewrap + socat | 패키지 매니저로 설치 필요 |

Linux/WSL2 설치:

```bash
# Ubuntu/Debian
sudo apt-get install bubblewrap socat

# Fedora
sudo dnf install bubblewrap socat
```

`/sandbox` 명령으로 패널을 열어 모드를 선택합니다. 선택한 모드는 프로젝트 로컬 설정(`.claude/settings.local.json`)에 기록됩니다. 모든 프로젝트에 적용하려면 사용자 설정(`~/.claude/settings.json`)에서 `sandbox.enabled`를 `true`로 설정하세요.

### 샌드박스 모드

| 모드 | 동작 |
|------|------|
| **Auto-allow** | Bash 명령이 샌드박스 내에서 자동으로 실행되며 권한 없이 허용됨. 샌드박스 불가 명령은 일반 권한 흐름으로 폴백 |
| **Regular permissions** | 샌드박스된 명령도 포함하여 모든 Bash 명령이 일반 권한 흐름을 거침 |

> Auto-allow 모드에서도 명시적 거부 규칙은 항상 존중되며, `/`, 홈 디렉토리 등 중요 시스템 경로를 대상으로 하는 `rm`/`rmdir`은 여전히 권한 프롬프트를 트리거합니다.

샌드박스에서 실행할 수 없는 명령(호환되지 않는 도구, 허용되지 않은 호스트 필요 등)은 `dangerouslyDisableSandbox` 매개변수로 재시도할 수 있습니다. 이 이스케이프 해치를 비활성화하려면 `"allowUnsandboxedCommands": false`를 설정하세요(Strict sandbox mode).

### 구성

`settings.json`에서 샌드박스 동작을 커스터마이즈합니다.

#### 파일시스템 격리

```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": ["~/.kube", "/tmp/build"],
      "denyWrite": ["~/secrets"],
      "denyRead": ["~/"],
      "allowRead": ["."]
    }
  }
}
```

| 설정 | 설명 |
|------|------|
| `sandbox.filesystem.allowWrite` | 작업 디렉토리 외부 경로에 대한 쓰기 접근 허용 |
| `sandbox.filesystem.denyWrite` | 특정 경로에 대한 쓰기 차단 |
| `sandbox.filesystem.denyRead` | 특정 경로에 대한 읽기 차단 |
| `sandbox.filesystem.allowRead` | `denyRead` 영역 내에서 특정 경로 재허용 |

경로 접두사 규칙:

| 접두사 | 의미 | 예시 |
|--------|------|------|
| `/` | 파일시스템 루트의 절대 경로 | `/tmp/build` |
| `~/` | 홈 디렉토리 상대 | `~/.kube` |
| `./` 또는 없음 | 프로젝트 루트 상대 (프로젝트 설정) 또는 `~/.claude` 상대 (사용자 설정) | `./output` |

> 기본적으로 샌드박스는 작업 디렉토리로 읽기/쓰기를 허용하고 전체 컴퓨터를 읽을 수 있습니다. 단, `~/.aws/credentials`, `~/.ssh/` 등 자격 증명 파일은 기본적으로 읽기 허용되므로 `denyRead`로 차단해야 합니다.

#### 네트워크 격리

```json
{
  "sandbox": {
    "enabled": true,
    "network": {
      "allowedDomains": ["api.github.com", "registry.npmjs.org"],
      "deniedDomains": ["evil.example.com"],
      "httpProxyPort": 8080,
      "socksProxyPort": 8081
    }
  }
}
```

| 설정 | 설명 |
|------|------|
| `sandbox.network.allowedDomains` | Bash 명령이 접근할 수 있는 도메인 사전 허용 |
| `sandbox.network.deniedDomains` | `allowedDomains` 와일드카드보다 우선하여 차단할 도메인 |
| `sandbox.network.httpProxyPort` | 커스텀 HTTP 프록시 포트 |
| `sandbox.network.socksProxyPort` | 커스텀 SOCKS 프록시 포트 |

> `allowManagedDomainsOnly`를 관리 설정에 지정하면, 관리 설정의 `allowedDomains`만 적용되고 프롬프트 대신 자동으로 차단됩니다.

#### 제외 명령

```json
{
  "sandbox": {
    "enabled": true,
    "excludedCommands": ["docker *", "terraform *"]
  }
}
```

`excludedCommands`에 나열된 명령은 샌드박스 외부에서 실행됩니다.

### 권한 규칙과의 관계

| 계층 | 제어 대상 |
|------|----------|
| **권한 규칙** | Claude Code가 사용할 수 있는 도구 제어. 모든 도구(Bash, Read, Edit, WebFetch, MCP 등)에 적용. 명령 실행 전 평가 |
| **샌드박싱** | Bash 명령이 파일시스템/네트워크 수준에서 접근할 수 있는 대상 제한. OS 수준 강제. Bash 명령과 자식 프로세스에만 적용 |

두 계층의 파일시스템/네트워크 경로는 병합되어 최종 샌드박스 구성을 형성합니다.

### 조직 전체 샌드박스 강제

관리 설정을 통해 모든 개발자에게 샌드박스를 강제할 수 있습니다.

```json
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "allowUnsandboxedCommands": false
  }
}
```

| 키 | 설명 |
|-----|------|
| `failIfUnavailable` | 샌드박스 의존성(예: Linux의 bubblewrap) 누락 시 Claude Code 시작 차단 |
| `allowUnsandboxedCommands: false` | `dangerouslyDisableSandbox` 이스케이프 해치 무시 |

> 개발자가 정책을 확장하는 것을 방지하려면 `allowManagedReadPathsOnly`와 `allowManagedDomainsOnly`를 사용하세요.

### 트러블슈팅

| 문제 | 해결 방법 |
|------|----------|
| host-not-allowed 오류 | 프롬프트에서 권한을 부여하면 호스트가 허용 목록에 추가됨 |
| `jest` 중단/실패 | `watchman`이 샌드박스와 호환되지 않음. `jest --no-watchman` 사용 |
| Go 기반 CLI TLS 실패 (macOS) | `gh`, `gcloud`, `terraform` 등을 `excludedCommands`에 추가하거나 `enableWeakerNetworkIsolation: true` 설정 |
| `docker` 명령 실패 | `excludedCommands`에 `docker *` 추가 |
| 컨테이너 내 bubblewrap 실패 | `enableWeakerNestedSandbox: true` 설정 |
| seccomp 필터 누락 (Linux) | `npm install -g @anthropic-ai/sandbox-runtime`으로 설치 |

### 제한 사항

- **네트워크 필터링**: 내장 프록시는 TLS 검사를 수행하지 않으므로 암호화된 연결의 내용은 검사되지 않습니다. 신뢰할 수 있는 도메인만 허용하세요.
- **Unix 소켓 권한 상승**: `allowUnixSockets` 설정은 강력한 시스템 서비스에 접근을 허용할 수 있어 샌드박스 우회로 이어질 수 있습니다. 예: `/var/run/docker.sock` 허용 시 호스트 시스템 접근 가능.
- **파일시스템 권한 상승**: `$PATH` 내 실행 파일 디렉토리, 시스템 구성 디렉토리, 셸 구성 파일(`.bashrc`, `.zshrc`)에 대한 광범위한 쓰기 권한은 권한 상승 공격을 가능하게 합니다.
- **설정 파일 보호**: 샌드박스는 Claude Code의 `settings.json` 파일(모든 스코프) 및 관리 설정 디렉토리에 대한 쓰기를 자동으로 차단합니다.
- **환경 변수**: 샌드박스된 Bash 명령은 기본적으로 부모 프로세스의 환경 변수를 상속하며, 여기에는 자격 증명이 포함될 수 있습니다. Anthropic 및 클라우드 제공자의 자격 증명을 하위 프로세스에서 제거하려면 `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` 환경 변수를 설정하세요.
- **하위 에이전트**: 하위 에이전트는 부모 세션과 동일한 프로세스에서 실행되며 동일한 샌드박스 구성을 사용합니다. 샌드박싱이 부모 세션에서 활성화된 경우 하위 에이전트 내의 Bash 명령도 샌드박스됩니다.
- **적용 범위**: 내장 파일 도구(Read, Edit, Write)는 권한 시스템을 통해 제어되며 샌드박스를 거치지 않습니다. MCP 서버와 훅은 샌드박스 외부에서 실행됩니다.
- **플랫폼**: macOS, Linux, WSL2 지원. WSL1 및 Native Windows는 미지원.

---

## 샌드박스 환경 비교

Claude Code를 격리하는 방법은 경량 per-command 샌드박스부터 완전한 가상 머신까지 여러 가지가 있습니다.

### 접근 방식 비교

| 접근 방식 | 격리 대상 | Docker 필요 | 설정 난이도 |
|-----------|----------|-------------|------------|
| **Sandboxed Bash tool** | Bash 명령과 자식 프로세스 | 아니요 | macOS: 최소, Linux/WSL2: 낮음 |
| **Sandbox runtime** | Claude Code 전체 프로세스 (파일 도구, MCP 서버, 훅 포함) | 아니요 | 낮음 |
| **Dev container** | 전체 개발 환경 | 예 | 중간 |
| **Custom container** | 전체 개발 환경 | 예 | 중간~높음 |
| **가상 머신** | 전체 운영체제 | 아니요 | 높음 |
| **Claude Code on the web** | 전체 운영체제 (Anthropic 호스팅) | 아니요 | 없음 (Claude 구독 및 GitHub 필요) |

> Sandboxed Bash tool은 Bash 명령만 제한합니다. 내장 파일 도구, MCP 서버, 훅은 호스트에서 직접 실행됩니다. 나머지 방식은 Claude Code 전체 프로세스를 격리 경계 내에 배치합니다.

### 목적별 선택 가이드

| 목적 | 추천 방식 |
|------|----------|
| 일상 작업 중 권한 프롬프트 감소 | Sandboxed Bash tool (`/sandbox`로 활성화) |
| `--dangerously-skip-permissions` 또는 auto mode로 무인 실행 | Dev container, 컨테이너, VM, 또는 sandbox runtime |
| Docker 없이 MCP 서버와 훅까지 격리 | Sandbox runtime |
| 신뢰할 수 없는 리포지토리 작업 | 전용 가상 머신 또는 Claude Code on the web |
| 팀 전체에 샌드박스 환경 표준화 | Preconfigured dev container를 리포지토리에 커밋 |
| 로컬 설정 없는 기기에서 사용 | Claude Code on the web |
| 조직 내 모든 개발자에게 격리 강제 | 관리 설정으로 샌드박스 강제 |

### 격리와 권한 모드의 관계

권한 모드는 도구 호출이 실행되는지와 사전 프롬프트 여부를 결정합니다. 격리는 명령이 실행된 후 접근할 수 있는 대상을 제한합니다. 두 가지는 함께 작동합니다.

| | 제어 대상 | 프롬프트 대체 |
|---|----------|-------------|
| `/sandbox` | Bash 명령이 실행 후 접근할 수 있는 대상 | 샌드박스 경계 자체 (auto-allow 모드) |
| Auto mode | 각 도구 호출의 실행 여부 | 동작을 검토하는 분류기 |
| `--dangerously-skip-permissions` | 각 도구 호출의 실행 여부 | 없음. 보호 경로 검사도 생략 |

> `--dangerously-skip-permissions`를 사용할 때는 반드시 컨테이너, VM, 또는 sandbox runtime 내에서 실행하세요.

### Sandbox Runtime

`@anthropic-ai/sandbox-runtime` 패키지는 전체 프로세스를 Seatbelt 또는 bubblewrap 격리로 래핑합니다. Bash뿐만 아니라 모든 도구, 훅, MCP 서버를 제한합니다.

```bash
npx @anthropic-ai/sandbox-runtime claude
```

`~/.srt-settings.json`에서 최소한 프로젝트 디렉토리와 Claude Code 구성 경로(`~/.claude`, `~/.claude.json`)에 쓰기 접근을 허용하고, `api.anthropic.com` 등 필요한 네트워크 도메인을 허용하세요.

### Dev Container

Dev container는 Docker 컨테이너 내부에서 Claude Code를 실행하며, 프로젝트가 마운트됩니다. `.devcontainer/` 디렉토리로 정의합니다. claude-code 리포지토리는 default-deny iptables 방화벽이 포함된 예제 dev container를 제공합니다.

### 가상 머신

전용 가상 머신은 자체 커널과 가상화된 하드웨어로 가장 강력한 분리를 제공합니다. 클라우드 인스턴스, 로컬 하이퍼바이저, Firecracker 같은 microVM 등이 포함됩니다. 신뢰할 수 없는 코드를 평가하거나 커널 수준 분리가 필요한 경우에 사용하세요.

---

## 보안 모범 사례

### 민감한 코드 작업 시

| 실천 사항 | 설명 |
|----------|------|
| **변경사항 검토** | 승인 전 모든 제안된 변경사항 검토 |
| **프로젝트별 권한 설정** | 민감한 리포지토리에 프로젝트별 권한 설정 사용 |
| **DevContainer 사용** | 추가 격리를 위해 DevContainer 사용 고려 |
| **권한 정기 감사** | `/permissions`로 권한 설정을 정기적으로 감사 |

### 팀 보안

| 실천 사항 | 설명 |
|----------|------|
| **엔터프라이즈 관리 정책** | 조직 표준을 강제하는 엔터프라이즈 관리 정책 사용 |
| **권한 구성 공유** | 버전 관리를 통해 승인된 권한 구성 공유 |
| **팀원 교육** | 보안 모범 사례에 대한 팀원 교육 |
| **사용량 모니터링** | OpenTelemetry 메트릭을 통해 Claude Code 사용량 모니터링 |
| **설정 변경 감사** | `ConfigChange` 훅으로 세션 중 설정 변경을 감사하거나 차단 |

### 취약점 보고

Claude Code에서 보안 취약점을 발견한 경우:

| 단계 | 행동 |
|------|------|
| **1** | 공개적으로 공개하지 않기 |
| **2** | HackerOne 프로그램을 통해 보고 |
| **3** | 상세한 재현 단계 포함 |
| **4** | 공개 공개 전 해결 시간 허용 |

---

## 요약

Claude Code는 권한 기반 아키텍처, 내장 보호 기능, 다층 프롬프트 인젝션 방지 체계를 통해 강력한 보안을 제공합니다. 사용자는 기본적으로 읽기 전용 권한으로 시작하며, 모든 쓰기 작업과 명령 실행은 명시적 승인이 필요합니다. Sandboxed Bash Tool, 샌드박스 런타임, Dev Container, 가상 머신 등 다양한 격리 수준을 위협 모델에 맞게 선택할 수 있습니다. 클라우드 실행 시 격리된 VM, 네트워크 제어, 자격 증명 보호 등 추가 보안 제어가 적용됩니다. 보안 모범 사례를 준수하고 정기적으로 권한을 감사하여 안전하게 사용하세요.
