# 보안 및 샌드박스

> <https://developers.openai.com/codex/agent-approvals-security>
> <https://developers.openai.com/codex/concepts/sandboxing>
> <https://developers.openai.com/codex/concepts/sandboxing/auto-review>
> <https://developers.openai.com/codex/concepts/cyber-safety>
> <https://developers.openai.com/codex/codex-security>

---

## 보안 모델 개요

Codex 보안 제어는 두 가지 레이어로 구성됩니다.

| 레이어 | 역할 |
| --- | --- |
| **샌드박스 모드** | Codex가 기술적으로 수행할 수 있는 작업 (예: 쓰기 가능한 경로, 네트워크 접근) |
| **승인 정책** | Codex가 작업을 실행하기 전에 사용자 승인을 요청해야 하는 시점 |

Codex는 기본적으로 **네트워크 접근이 꺼진 상태**로 시작하며, OS 수준 샌드박스로 파일 시스템 접근을 제한합니다.

---

## Codex Security

> <https://developers.openai.com/codex/codex-security>

Codex Security는 보안 취약점을 발견하고 수정하는 전용 제품으로, 플러그인과 클라우드 두 가지 형태로 제공됩니다.

### Codex Security 플러그인

Codex Security 플러그인은 **로컬 저장소**에서 동작합니다.

- 로컬 리포지토리의 코드를 직접 스캔
- diff-review 워크플로우로 변경 사항 검사
- Codex 스레드 내에서 실행
- 심층 보안 스캔, 병합 전 코드 변경 검사, 취약점 백로그 수정 지원

### Codex Security 클라우드 (Research Preview)

Codex Security 클라우드는 **연결된 GitHub 리포지토리**를 커밋 단위로 스캔합니다.

- **작동 방식**:
  1. 리포지토리별 위협 모델 구축
  2. 실제 코드 컨텍스트를 기반으로 취약점 검사
  3. 높은 신호의 이슈를 격리 환경에서 검증 후 표시
- **결과 제공**: 순위가 매겨진 결과, 증거, 제안된 패치 옵션
- **지원 플랜**: ChatGPT Enterprise, Edu, Business, Pro

| 기능 | 플러그인 | 클라우드 |
| --- | --- | --- |
| 실행 위치 | 로컬 스레드 | Codex Web |
| 스캔 대상 | 로컬 저장소 / diff | GitHub 저장소 (커밋 단위) |
| 위협 모델 | 로컬 컨텍스트 기반 | 리포지토리별 맞춤형 |

---

## 샌드박스 (OS 수준)

Codex는 운영체제 수준의 샌드박스를 사용하여 에이전트의 행동을 제한합니다.

### 플랫폼별 샌드박스 메커니즘

| 플랫폼 | 메커니즘 | 설명 |
| --- | --- | --- |
| **macOS 12+** | Apple Seatbelt (`sandbox-exec`) | 선택한 `--sandbox` 모드에 해당하는 프로필로 파일 시스템 및 네트워크 접근 제한 |
| **Linux / WSL2** | `bwrap` + `seccomp` | `bubblewrap` 패키지 설치 필요. 첫 번째 `bwrap` 실행 파일을 사용하며, 없으면 번들 헬퍼로 폴백 (비특권 사용자 네임스페이스 필요) |
| **Windows (WSL2)** | Linux 샌드박스 (`bwrap` + `seccomp`) | WSL2에서 실행 시 Linux 샌드박스와 동일. WSL1은 Codex `0.114`까지만 지원되었으며, `0.115`부터 `bwrap` 기반으로 전환되어 WSL1 지원이 중단됨 |
| **Windows (네이티브)** | 네이티브 Windows 샌드박스 | PowerShell 등 네이티브 환경에서 실행 시 Windows 자체 샌드박스 사용. `config.toml`에서 `unelevated` / `elevated` 모드 선택 가능 |

### 샌드박스 정책 (`--sandbox` 플래그)

| 정책 | 설명 | 효과 |
| --- | --- | --- |
| `read-only` | 읽기 전용 | 파일 읽기 및 질문에 답변만 가능. 편집, 명령 실행, 네트워크 접근 불가 |
| `workspace-write` | 워크스페이스 쓰기 | 작업 디렉토리 내에서 읽기/쓰기/명령 실행 가능. 워크스페이스 밖 작업은 승인 필요 |
| `danger-full-access` | 전체 접근 | 샌드박스 없음. 모든 파일 시스템 및 네트워크 접근 허용 (권장하지 않음) |

### 쓰기 가능 루트의 보호 경로

`workspace-write` 모드에서도 다음 경로는 **읽기 전용**으로 보호됩니다.

- `<워크스페이스>/.git` (디렉토리 또는 포인터 파일이 가리키는 경로 포함)
- `<워크스페이스>/.agents`
- `<워크스페이스>/.codex`

### 샌드박스 테스트

```bash
# macOS
codex sandbox macos [--permissions-profile <name>] [--log-denials] [COMMAND]...

# Linux
codex sandbox linux [--permissions-profile <name>] [COMMAND]...

# Windows
codex sandbox windows [--permissions-profile <name>] [COMMAND]...
```

> `sandbox` 명령은 `codex debug`로도 사용 가능하며, 플랫폼 헬퍼별 alias도 제공됩니다 (예: `codex sandbox seatbelt`, `codex sandbox landlock`).

### Linux / WSL2 필수 패키지

Linux 및 WSL2에서는 `bubblewrap` 패키지를 설치해야 합니다.

```bash
sudo apt install bubblewrap
```

Codex는 `PATH`에서 첫 번째 `bwrap` 실행 파일을 찾습니다. `bwrap`이 없으면 번들 헬퍼로 폴백하지만, 이 헬퍼는 비특권 사용자 네임스페이스 생성이 필요합니다. 배포 패키지로 설치하는 것이 가장 안정적입니다.

`bwrap`이 누락되거나 필요한 사용자 네임스페이스를 생성할 수 없는 경우 Codex는 시작 시 경고를 표시합니다.

**Ubuntu AppArmor 참고**: Ubuntu 25.04에서는 `bubblewrap` 패키지만 설치하면 추가 AppArmor 설정 없이 동작합니다. Ubuntu 24.04에서는 추가 프로필 로드가 필요할 수 있습니다:

```bash
sudo apt update
sudo apt install apparmor-profiles apparmor-utils
sudo install -m 0644 \
  /usr/share/apparmor/extra-profiles/bwrap-userns-restrict \
  /etc/apparmor.d/bwrap-userns-restrict
sudo apparmor_parser -r /etc/apparmor.d/bwrap-userns-restrict
```

또는 AppArmor 비특권 사용자 네임스페이스 제한을 비활성화할 수도 있습니다:

```bash
sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=0
```

### Windows 설정

**WSL2**에서 실행 시 Linux 샌드박스(`bwrap` + `seccomp`)가 사용됩니다. WSL1은 Codex `0.114`까지만 지원되었으며, `0.115`부터 Linux 샌드박스가 `bwrap` 기반으로 전환되면서 WSL1 지원이 중단되었습니다.

IDE 확장에서 WSL2를 직접 사용하려면 VS Code 설정에 다음을 추가합니다:

```json
{
  "chatgpt.runCodexInWindowsSubsystemForLinux": true
}
```

이렇게 하면 IDE 확장이 호스트 OS가 Windows여도 Linux 샌드박스 의미 체계를 상속합니다.

**네이티브 Windows**에서는 `config.toml`로 샌드박스 모드를 선택합니다:

```toml
[windows]
sandbox = "unelevated"     # 또는 "elevated"
# sandbox_private_desktop = true   # 기본값; 호환성 문제 시에만 false
```

> Docker 등 컨테이너 환경에서 Linux를 실행하는 경우, 호스트나 컨테이너 설정이 네임스페이스·setuid `bwrap`·seccomp 작업을 차단하면 샌드박스가 동작하지 않을 수 있습니다. 이 경우 컨테이너 자체를 격리 경계로 사용하고 `--sandbox danger-full-access`로 실행하세요.

---

## 권한 승인 모드

### 승인 정책 (`--ask-for-approval` / `approval_policy`)

| 모드 | 플래그 | 설명 |
| --- | --- | --- |
| `untrusted` | (기본값) | 안전한 읽기 작업만 자동 실행. 상태 변경 명령은 승인 필요 |
| `on-request` | `--ask-for-approval on-request` | 워크스페이스 내 작업은 자동. 위험한 작업은 승인 요청 |
| `never` | `--ask-for-approval never` | 승인 없이 모든 작업 실행. 샌드박스 모드와 조합 가능 |
| — | `--yolo` (`--dangerously-bypass-approvals-and-sandbox`) | **승인 정책 `never`와 다름** — 승인 프롬프트뿐만 아니라 샌드박스 자체도 함께 비활성화. 네트워크 및 파일시스템 제한 없이 전체 접근 허용 (위험, 비권장) |

### 일반적인 조합

| 목적 | 플래그 | 효과 |
| --- | --- | --- |
| 안전한 읽기 전용 | `--sandbox read-only --ask-for-approval on-request` | 읽기만 가능, 승인 필요 |
| CI용 비대화형 | `--sandbox read-only --ask-for-approval never` | 읽기만, 승인 요청 없음 |
| 자동 편집 (프리셋) | `--full-auto` (`--sandbox workspace-write --ask-for-approval on-request`) | 워크스페이스 내 자동, 외부는 승인 |
| YOLO (비권장) | `--dangerously-bypass-approvals-and-sandbox` (`--yolo`) | 샌드박스 없음 + 승인 없음. `--ask-for-approval never`와 달리 샌드박스 자체도 비활성화됨 |
| Auto-review 모드 | `--sandbox workspace-write --ask-for-approval on-request -c approvals_reviewer=auto_review` | 표준 on-request와 동일한 샌드박스 경계, 승인 요청을 리뷰어 에이전트가 평가 |

> **참고**: `codex exec --full-auto`는 **deprecated**입니다. 하위 호환 경로로 유지되며 실행 시 경고가 출력됩니다. 대신 `codex exec --sandbox workspace-write`를 사용하세요.

### 세부 승인 정책 (Granular)

```toml
approval_policy = { granular = {
  sandbox_approval = true,        # 샌드박스 에스컬레이션 승인
  rules = true,                   # execpolicy 규칙 승인
  mcp_elicitations = true,        # MCP 호출 승인
  request_permissions = false,    # 권한 요청 자동 거부
  skill_approval = false          # 스킬 스크립트 자동 승인
} }
```

### MCP 서버별 승인 모드

```toml
[mcp_servers.docs]
command = "docs-server"
default_tools_approval_mode = "approve"   # auto | prompt | approve

[mcp_servers.docs.tools.search]
approval_mode = "prompt"                  # 개별 도구 오버라이드
```

| 승인 모드 | 설명 |
| --- | --- |
| `auto` | 기본 동작 사용 |
| `prompt` | 도구 호출 시 사용자 승인 요청 |
| `approve` | 자동 승인 |

---

## Auto-Review (자동 검토)

Auto-Review는 승인 요청을 사용자 대신 **별도의 리뷰어 에이전트**가 평가하는 기능입니다. 메인 에이전트는 동일한 샌드박스, 동일한 승인 정책, 동일한 네트워크 및 파일시스템 제한 안에서 동작하며, 변경되는 것은 **누가 승인 요청을 검토하는가**뿐입니다.

```toml
approval_policy = "on-request"
approvals_reviewer = "auto_review"    # user | auto_review
```

> Auto-Review는 승인이 **대화형**일 때만 적용됩니다. 즉 `approval_policy = "on-request"` 또는 관련 프롬프트 카테고리가 활성화된 granular 정책에서만 동작합니다. `approval_policy = "never"`에서는 검토할 것이 없으므로 Auto-Review도 실행되지 않습니다.

### 동작 흐름

1. 메인 에이전트가 `read-only` 또는 `workspace-write` 내에서 작업
2. 샌드박스 경계를 넘어야 할 때 승인 요청
3. `approvals_reviewer = "auto_review"`인 경우, 해당 승인 요청이 사용자 대신 리뷰어 에이전트로 라우팅
4. 리뷰어가 해당 작업의 실행 여부를 판단하고 근거(rationale)를 반환
5. 승인되면 실행이 계속되고, 거부되면 메인 에이전트는 **본질적으로 더 안전한 대안**을 찾거나 사용자에게 질문

> Auto-Review는 **권한 부여가 아닌 리뷰어 교체**입니다. `writable_roots`를 확장하거나, 네트워크 접근을 활성화하거나, 보호 경로를 약화시키지 않습니다.

### 검토 대상

Auto-Review는 다음과 같은 승인 요청을 평가합니다:

- 샌드박스 권한 에스컬레이션을 요청하는 Shell/exec 도구 호출
- 현재 샌드박스 또는 정책에 의해 차단된 네트워크 요청
- 허용된 쓰기 가능 루트 밖의 파일 편집
- 도구 어노테이션 또는 구성된 승인 모드에 따라 승인이 필요한 MCP/app 도구 호출
- 새로운 웹사이트나 도메인에 대한 Browser Use 접근

샌드박스 내에서 이미 허용되는 일상적인 작업에는 Auto-Review가 실행되지 않습니다.

### 차단 기준

리뷰어는 기본적으로 다음과 같은 작업을 차단합니다:

- 신뢰할 수 없는 대상으로 개인 데이터, 비밀, 자격 증명 전송
- 자격 증명, 토큰, 쿠키, 세션 자료 탐색
- 광범위하거나 지속적인 보안 약화
- 되돌릴 수 없는 손상 위험이 있는 파괴적 행동

### 리스크 레벨

| 레벨 | 처리 |
| --- | --- |
| 낮음/중간 | 정책이 허용하면 자동 승인 |
| 높음 | 충분한 사용자 권한 및 거부 규칙 미매칭 필요 |
| 치명적 | 항상 거부 |
| 빌드/파싱 실패 | **안전 쪽으로 실패(fail closed)** — 실행 불가 |

> 타임아웃은 명시적 거부와 별도로 처리됩니다. 타임아웃만으로 해당 작업이 안전하지 않다는 증거가 되지 않는다고 메인 에이전트에 알립니다.

### 회로 차단기 (Circuit Breaker)

Auto-Review는 **턴당 거부 회로 차단기**를 적용합니다:

| 조건 | 임계값 | 효과 |
| --- | --- | --- |
| 연속 거부 | **3회** | 턴 중단 |
| 롤링 윈도우 내 거부 | **최근 50회 중 10회** | 턴 중단 |

거부 이외의 결과(승인, 타임아웃 등)가 발생하면 연속 거부 카운터가 초기화됩니다. 차단기가 트리거되면 경고를 출력하고 현재 턴을 interrupt로 중단합니다.

### `/approve` 재시도 경로

거부된 작업에 대한 명시적 오버라이드 경로가 있습니다. 현재 오픈소스 TUI에서 `/approve`를 실행하면 **Auto-review Denials** 선택기가 열리며, 최근 거부된 작업 하나를 선택하여 1회 재시도를 승인할 수 있습니다.

- Codex는 스레드당 최대 10개의 최근 거부를 기록
- 승인은 해당 **정확한 거부 작업에만** 적용되며, 유사한 향후 작업에는 적용되지 않음
- 재시도 역시 Auto-Review를 거치며, 리뷰어가 정책상 사용자가 덮어쓸 수 없는 거부 클래스면 다시 거부 가능

### Computer Use 예외

Computer Use의 앱 승인은 **사용자에게 직접 표시**되므로, Auto-Review가 이를 대체하지 않습니다.

### 정책 커스터마이징

기본 리뷰어 정책은 오픈소스 Codex 저장소의 `core/src/guardian/policy.md`에 있습니다.

**엔터프라이즈**: `guardian_policy_config`로 테넌트별 정책 교체:

```toml
# managed requirements에서 정의
guardian_policy_config = """
## Environment Profile
- Trusted internal destinations include github.com/my-org, artifacts.example.com

## Tenant Risk Taxonomy and Allow/Deny Rules
- Treat uploads to unapproved third-party file-sharing services as high risk
- Deny actions that expose credentials or private source code
"""
```

**개인 사용자**: 로컬 `config.toml`에 `[auto_review].policy` 설정 (단, managed requirements가 우선):

```toml
[auto_review]
policy = """
YOUR POLICY GOES HERE
"""
```

> 정책을 커스터마이징할 때는 기본 정책 전문을 먼저 복사한 뒤, 개별 리스크 프로필에 맞게 수정하는 것이 좋습니다.

### 모범 사례

Auto-Review는 샌드박스가 일상적인 안전한 워크플로를 이미 커버하고 있을 때 가장 잘 동작합니다. 너무 많은 일상적 작업이 검토를 필요로 한다면, 리뷰어가 노이즈가 많은 에스컬레이션을 계속 승인하도록 가르치기보다 **경계 자체를 먼저 수정**하세요.

실질적인 효과가 높은 변경:

- 스크래치 디렉토리나 인접 리포지토리에 대해 좁은 범위의 `writable_roots` 추가
- 좁은 범위의 명령어 접두사 규칙 사용 — `["cargo", "test"]`나 `["pnpm", "run", "lint"]`는 좋고, `["python"]`이나 `["curl"]`은 너무 넓음

Auto-Review 세션 기록은 기본적으로 `~/.codex/sessions`에 보존되며, 정책이나 권한을 변경하기 전에 과거 트래픽을 분석하는 데 활용할 수 있습니다.

---

## 네트워크 제어

### 클라우드 에이전트

- **기본**: 인터넷 접근 차단 (보안 및 프롬프트 인젝션 방지)
- **런타임 모델**: 2단계 런타임 — setup 단계에서는 네트워크 접근이 가능하여 지정된 의존성을 설치하고, agent 단계에서는 기본적으로 오프라인으로 실행 (인터넷 접근을 활성화한 환경은 예외)
- **시크릿**: 클라우드 환경에 구성된 시크릿은 setup 단계에서만 사용 가능하며, agent 단계 시작 전에 제거됨
- **설정**: 관리자가 허용 목록(allowlist)으로 도메인 및 HTTP 메서드 제어 가능

### 로컬 에이전트

- `workspace-write` 모드에서 **기본 네트워크 접근 꺼짐**
- 활성화하려면 명시적 설정 필요:

```toml
[sandbox_workspace_write]
network_access = true
```

CLI 세션에서 한 번만 적용하려면:

```bash
codex \
  -c 'features.network_proxy=true' \
  -c 'sandbox_workspace_write.network_access=true'
```

### 샌드박스 네트워크 프록시

네트워크가 이미 활성화된 경우, `network_proxy` 기능으로 트래픽을 추가로 제어할 수 있습니다.

```toml
[features.network_proxy]
enabled = true
domains = { "api.openai.com" = "allow", "example.com" = "deny" }
```

CLI에서 정책과 함께 설정:

```bash
codex \
  -c 'features.network_proxy.enabled=true' \
  -c 'features.network_proxy.domains={ "api.openai.com" = "allow", "example.com" = "deny" }' \
  -c 'sandbox_workspace_write.network_access=true'
```

> `network_proxy`는 **활성화된 네트워크 접근을 어떻게 제어할지**를 결정하는 기능이며, 그 자체로 네트워크 접근을 부여하지 않습니다.

**네트워크 상태 조합**:

| `network_access` | `network_proxy` | 효과 |
| --- | --- | --- |
| 꺼짐 | 켜짐 | 네트워크 꺼짐 유지, 프록시 동작 없음 |
| 켜짐 | 꺼짐 | 네트워크 켜짐, 제한 없는 직접 아웃바운드 |
| 켜짐 | 켜짐 | 네트워크 켜짐, 구성된 정책으로 아웃바운트 제한 |

**도메인 규칙** (allowlist-first):

| 패턴 | 매칭 |
| --- | --- |
| 정확한 호스트 | 해당 호스트만 |
| `*.example.com` | 서브도메인만 (예: `api.example.com`). apex 도메인은 제외 |
| `**.example.com` | apex + 서브도메인 모두 |
| `*` (allow only) | 모든 공개 호스트. 광범위한 네트워크 접근과 동일하므로 좁은 범위 규칙 권장 |
| `deny` | 항상 `allow`보다 우선 |

### 로컬/프라이빗 대상 차단

기본적으로 `allow_local_binding = false`이며, 루프백·링크로컬·프라이빗 대상을 차단합니다:

| 접근 방법 | 설명 |
| --- | --- |
| 특정 예외 | 정확한 로컬 IP 리터럴 또는 `localhost` allow 규칙을 추가하면 해당 로컬 대상만 허용 |
| 광범위 접근 | `allow_local_binding = true` 설정 시 의도적으로 더 넓은 로컬/프라이빗 접근 허용 |
| 와일드카드 | 와일드카드 규칙은 명시적 로컬 예외로 간주하지 않음 |
| 해석된 주소 | allowlist와 일치해도 로컬/프라이빗 IP로 해석되는 호스트명은 여전히 차단됨 |

### DNS 리바인딩 보호

Codex는 호스트명을 허용하기 전에 **최선의 노력(best-effort) DNS 및 IP 분류 검사**를 수행합니다:

| 검사 | 처리 |
| --- | --- |
| 조회 실패 또는 타임아웃 | 차단 |
| 비공개 주소로 해석되는 호스트명 | 차단 |
| 와일드카드 | 명시적 로컬 예외로 간주하지 않음 |

> 이 검사는 DNS 리바인딩 위험을 **줄이지만 완전히 제거하지는 않습니다**. 완전한 방지를 위해서는 전송 계층에서 해석된 IP를 고정해야 합니다. 적대적 DNS가 위협 모델에 포함된다면 하위 계층에서 이그레스 제어도 함께 적용하세요.

### 프록시 상세 설정

`network_proxy` 활성화 시 사용 가능한 전체 설정:

| 설정 | 기본값 | 설명 |
| --- | --- | --- |
| `enabled` | `false` | 명령 네트워크 접근이 이미 켜져 있을 때만 샌드박스 네트워킹 시작 |
| `domains` | 미설정 | allowlist 동작. `allow` 규칙을 추가할 때까지 외부 대상 없음 |
| `unix_sockets` | 미설정 | 명시적 `allow` 규칙을 추가할 때까지 Unix 소켓 대상 없음 |
| `allow_local_binding` | `false` | 로컬 및 프라이빗 네트워크 대상 차단 |
| `enable_socks5` | `true` | 정책이 허용할 때 SOCKS5 지원 노출 |
| `enable_socks5_udp` | `true` | SOCKS5 사용 가능 시 UDP over SOCKS5 허용 |
| `allow_upstream_proxy` | `true` | 환경의 업스트림 프록시를 샌드박스 네트워킹에서 활용 |
| `dangerously_allow_non_loopback_proxy` | `false` | 리스너 엔드포인트를 루프백에 유지 (`true` 시 localhost 밖으로 노출) |
| `dangerously_allow_all_unix_sockets` | `false` | Unix 소켓 접근을 allowlist 기반으로 유지 (`true` 시 보호 우회) |

> Unix 소켓 프록시가 활성화된 경우, 리스너는 `dangerously_allow_non_loopback_proxy`가 `true`여도 루프백 전용으로 유지됩니다. 샌드박스 네트워킹이 로컬 데몬으로의 원격 브릿지가 되는 것을 방지하기 위함입니다.

### 웹 검색 제어

```toml
web_search = "cached"     # 기본값: OpenAI 관리 인덱스 사용
# web_search = "live"     # 실시간 웹 검색
# web_search = "disabled" # 웹 검색 비활성화
```

> `--yolo` 또는 다른 전체 접근 샌드박스 설정을 사용하면 웹 검색이 기본적으로 `live`로 설정됩니다. `--search` 플래그 또는 `web_search = "live"`로 실시간 검색을 허용할 수 있습니다.

> 프롬프트 인젝션 공격 위험 때문에 네트워크 접근이나 웹 검색 활성화 시 주의가 필요합니다.

---

## Dev Containers

호스트에서 Linux 샌드박스를 직접 실행할 수 없거나, 조직에서 이미 컨테이너 기반 개발을 표준화한 경우 **Dev Containers**에서 Codex를 실행하여 Docker를 외부 격리 경계로 사용할 수 있습니다. Visual Studio Code Dev Containers 및 호환 도구에서 동작합니다.

> Codex 보안 devcontainer 예제(`.devcontainer`)를 참조 구현으로 사용할 수 있습니다. Codex, 일반 개발 도구, `bubblewrap`, 방화벽 기반 아웃바운드 제어가 포함되어 있습니다.

### 구성 요소

| 파일 | 역할 |
| --- | --- |
| `.devcontainer/devcontainer.secure.json` | 컨테이너 설정, 기능, 마운트, 환경변수, VS Code 확장 |
| `.devcontainer/Dockerfile.secure` | Ubuntu 기반 이미지 및 설치 도구 정의 |
| `.devcontainer/init-firewall.sh` | 아웃바운드 네트워크 정책 적용 |

### 사용 방법

1. Visual Studio Code 및 Dev Containers 확장 설치
2. Codex 예제 `.devcontainer` 설정을 리포지토리에 복사
3. VS Code에서 **Dev Containers: Open Folder in Container…** 실행 후 `.devcontainer/devcontainer.secure.json` 선택
4. 컨테이너 시작 후 터미널에서 `codex` 실행

CLI로 시작:

```bash
devcontainer up --workspace-folder . --config .devcontainer/devcontainer.secure.json
```

### 컨테이너 내 샌드박스 선택

| 모드 | 설명 |
| --- | --- |
| Linux 샌드박스 유지 | Dev Container 프로필이 `bwrap`에 필요한 기능을 부여하는 경우 내부 샌드박스를 계속 사용 |
| `--sandbox danger-full-access` | 컨테이너 자체가 보안 경계인 경우, Codex가 두 번째 샌드박스 레이어를 생성하지 않도록 설정 |

> Dev Container는 상당한 보호를 제공하지만 모든 공격을 방지하지는 못합니다. 컨테이너 내에서 `--sandbox danger-full-access` 또는 `--yolo`를 사용하면 악의적인 프로젝트가 devcontainer 내의 모든 것(자격 증명 포함)을 유출할 수 있습니다. 신뢰할 수 있는 리포지토리에서만 사용하세요.

### 참고 사항

- 참조 방화벽은 시작점으로 의도된 것입니다. 도메인 allowlist를 격리 수단으로 의존하는 경우, 환경에 맞는 DNS 리바인딩 및 DNS 새로고침 보호(TTL 인식 새로고침 또는 DNS 인식 방화벽)를 구현하세요.
- 지속적 마운트로 명령어 히스토리와 Codex 구성을 보존합니다.
- `bubblewrap`이 포함되어 있어 컨테이너가 필요한 기능을 부여하면 내부 Linux 샌드박스를 사용할 수 있습니다.

---

## 엔터프라이즈 보안 기능

### Zero Data Retention (ZDR)

- Codex App, CLI, IDE에서 **코드가 개발자 환경에만 유지**
- 엔터프라이즈 데이터 학습에 사용되지 않음

### 저장 데이터 암호화

- **AES-256** 암호화로 데이터 보호

### 전송 중 암호화

- **TLS 1.2+** 로 모든 통신 암호화

### 감사 로깅

- ChatGPT Compliance API를 통한 활동 로그 내보내기
- 최대 30일 보존

### 추가 엔터프라이즈 보안

| 기능 | 설명 |
| --- | --- |
| 거주지 및 보존 정책 | ChatGPT Enterprise 정책 따름 |
| 세분화된 사용자 접근 제어 | RBAC로 접근 관리 |
| 커스텀 CA 인증서 | `CODEX_CA_CERTIFICATE` 환경변수로 엔터프라이즈 프록시 지원 |

---

## OpenTelemetry (OTel) 모니터링

Codex는 **옵트인** 방식으로 OpenTelemetry(OTel) 모니터링을 지원합니다. 팀이 사용량을 감사하고, 문제를 조사하며, 컴플라이언스 요구사항을 충족할 수 있도록 도와줍니다. 로컬 보안 기본 설정을 약화시키지 않으며, 기본적으로 꺼져 있습니다.

### 개요

- OTel 내보내기는 기본적으로 꺼져 있어 로컬 실행이 자체 완료적으로 유지됩니다.
- 활성화하면 대화, API 요청, SSE/WebSocket 스트림 활동, 사용자 프롬프트(기본적으로 리랙션됨), 도구 승인 결정, 도구 결과를 다루는 구조화된 로그 이벤트를 내보냅니다.
- 내보낸 이벤트에는 `service.name`(발신자), CLI 버전, 환경 라벨(dev/staging/prod)이 태그됩니다.

### 활성화

`config.toml`(일반적으로 `~/.codex/config.toml`)에 `[otel]` 블록을 추가합니다.

```toml
[otel]
environment = "staging"    # dev | staging | prod
exporter = "none"           # none | otlp-http | otlp-grpc
log_user_prompt = false      # 정책이 허용하지 않는 한 프롬프트 텍스트를 리랙션
```

- `exporter = "none"`: 계측은 활성 상태지만 데이터를 어디에도 전송하지 않습니다.
- 자체 수집기로 이벤트를 전송하려면:

```toml
# OTLP HTTP
[otel]
exporter = { otlp-http = {
  endpoint = "https://otel.example.com/v1/logs",
  protocol = "binary",
  headers = { "x-otlp-api-key" = "<YOUR_OTLP_TOKEN>" }
}}
```

```toml
# OTLP gRPC
[otel]
exporter = { otlp-grpc = {
  endpoint = "https://otel.example.com:4317",
  headers = { "x-otlp-meta" = "<YOUR_META_VALUE>" }
}}
```

> Codex는 이벤트를 배치 처리하고 종료 시 플러시합니다. OTel 모듈에서 생성된 원격 측정 데이터만 내보냅니다.

### 이벤트 카테고리

| 이벤트 유형 | 설명 |
| --- | --- |
| `codex.conversation_starts` | 모델, 추론 설정, 샌드박스/승인 정책 |
| `codex.api_request` | 시도, 상태/성공, 소요 시간, 오류 상세 |
| `codex.sse_event` | 스트림 이벤트 종류, 성공/실패, 소요 시간, `response.completed` 시 토큰 수 |
| `codex.websocket_request` / `codex.websocket_event` | 요청 소요 시간, 메시지별 종류/성공/오류 |
| `codex.user_prompt` | 길이 (내용은 명시적으로 활성화하지 않으면 리랙션) |
| `codex.tool_decision` | 승인/거부, 출처(구성 vs 사용자) |
| `codex.tool_result` | 소요 시간, 성공, 출력 스니펫 |

관련 OTel 메트릭(카운터 + 소요 시간 히스토그램 쌍)으로 `codex.api_request`, `codex.sse_event`, `codex.websocket.request`, `codex.websocket.event`, `codex.tool.call`(및 대응하는 `.duration_ms` 계기)이 포함됩니다.

### 보안 및 프라이버시 가이드

| 권장 사항 | 설명 |
| --- | --- |
| `log_user_prompt = false` 유지 | 정책에서 명시적으로 허용하지 않는 한 프롬프트 내용(소스 코드, 민감 데이터 포함)을 저장하지 마세요 |
| 자체 수집기로만 라우팅 | 컴플라이언스 요구사항에 맞춰 보존 한도 및 접근 제어 적용 |
| 도구 인자 및 출력은 민감 정보 | 가능하면 수집기 또는 SIEM에서 리랙션 권장 |
| 로컬 데이터 보존 설정 검토 | `history.persistence` / `history.max_bytes` 확인 |
| 네트워크 오프 시 내보내기 불가 | CLI에서 네트워크 접근이 꺼져 있으면 OTel 수집기에 도달할 수 없음 |
| 이벤트 주기적 검토 | 승인/샌드박스 변경 및 예상치 못한 도구 실행 확인 |

> OTel은 선택 사항이며, 위에서 설명한 샌드박스 및 승인 보호를 **보완**하기 위한 것이지 대체하는 것이 아닙니다.

---

## Cyber Safety

> <https://developers.openai.com/codex/concepts/cyber-safety>

GPT-5.3-Codex는 OpenAI Preparedness Framework에서 **High 사이버 보안 역량**으로 분류된 최초의 모델입니다. 이에 따라 자격 증명 탈취와 같은 명백히 악의적인 요청 거부 등의 추가 안전 장치가 적용됩니다.

### 배경

최근 몇 달間 사이버 보안 작업에서 모델 성능이 의미 있게 향상되었습니다. 이는 개발자와 보안 전문가 모두에게 이점이 되지만, 취약점 발견 등의 사이버 보안 역량은 본질적으로 **이중 사용(dual-use)** 성격을 가집니다. 침투 테스트, 취약점 연구, 대규모 스캐닝, 멀웨어 분석, 위협 인텔리전스 등 방어적 작업에 사용되는 동일한 지식과 기술이 실제 피해를 유발하는 데에도 사용될 수 있습니다.

### 작동 방식

1. **안전 훈련**: 모델이 명백히 악의적인 요청(예: 자격 증명 탈취)을 거부하도록 훈련
2. **자동 분류기 기반 모니터**: 의심스러운 사이버 활동 신호를 감지하여 고위험 트래픽을 덜 강력한 사이버 모델(GPT-5.2)로 라우팅
3. **영향 범위**: 매우 적은 비율의 트래픽만 완화 조치의 영향을 받으며, 정책·분류기·제품 내 알림을 지속적으로 개선 중

| 구성 요소 | 설명 |
| --- | --- |
| GPT-5.3-Codex 분류 | Preparedness Framework 하에서 **High** 사이버 보안 역량 |
| 자동 분류기 모니터 | 의심스러운 사이버 활동 신호 감지 → 고위험 트래픽을 GPT-5.2로 라우팅 |
| 안전 훈련 | 명백히 악의적인 요청(자격 증명 탈취 등)에 대한 거부 |
| 알림 | 최신 Codex CLI alpha 버전에서 제품 내 메시지 지원 (모든 클라이언트로 확장 예정) |

### Trusted Access for Cyber

합법적인 방어적 작업을 위해 고위험 사이버 보안 작업에 대한 모델 접근을 유지하는 파일럿 프로그램입니다.

| 참여 방법 | 설명 |
| --- | --- |
| 개인 사용자 | [chatgpt.com/cyber](https://chatgpt.com/cyber)에서 신원 확인 |
| 엔터프라이즈 | OpenAI 담당자를 통해 팀 전체에 기본 Trusted Access 요청 |
| 초청 전용 프로그램 | 더 강력한 사이버 역량 모델 접근이 필요한 보안 연구자/팀은 별도 신청 |

> Trusted Access 사용자도 OpenAI 사용 정책(Usage Policies) 및 이용 약관(Terms of Use)을 준수해야 합니다.

### 오탐 (False Positives)

합법적이거나 비사이버 보안 활동이 가끔 플래그될 수 있습니다. 리라우팅 발생 시:
- API 요청 로그 및 CLI 제품 내 알림에서 응답 모델 확인 가능
- 잘못된 리라우팅으로 판단되면 `/feedback`으로 오탐 보고

> OpenAI는 대부분의 경우 계정 수준 검사에서 **요청 수준 검사**로 전환하여 완화 조치를 확장하고 사이버 복원력을 강화할 계획입니다.
