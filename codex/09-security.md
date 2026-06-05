# 보안 및 샌드박스

> Codex가 코드와 데이터를 보호하는 방법과 샌드박스, 승인 정책, 네트워크 제어에 대한 가이드입니다.

**참조**: <https://developers.openai.com/codex/agent-approvals-security>, <https://developers.openai.com/codex/security>, <https://raw.githubusercontent.com/openai/codex/main/docs/sandbox.md>

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
| **Linux** | `bwrap` + `seccomp` | Landlock/seccomp API를 사용하여 동일한 보장을 제공. 커널 지원 필요 |
| **Windows** | Restricted Token (AppContainer) | AppContainer 프로필에서 파생된 제한된 토큰으로 실행. 실험적 |

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
codex sandbox macos [--full-auto] [COMMAND]...

# Linux
codex sandbox linux [--full-auto] [COMMAND]...

# Windows
codex sandbox windows [--full-auto] [COMMAND]...
```

---

## 권한 승인 모드

### 승인 정책 (`--ask-for-approval` / `approval_policy`)

| 모드 | 플래그 | 설명 |
| --- | --- | --- |
| `untrusted` | (기본값) | 안전한 읽기 작업만 자동 실행. 상태 변경 명령은 승인 필요 |
| `on-request` | `--ask-for-approval on-request` | 워크스페이스 내 작업은 자동. 위험한 작업은 승인 요청 |
| `never` | `--ask-for-approval never` 또는 `--yolo` | 승인 없이 모든 작업 실행. 샌드박스 모드와 조합 가능 |

### 일반적인 조합

| 목적 | 플래그 | 효과 |
| --- | --- | --- |
| 안전한 읽기 전용 | `--sandbox read-only --ask-for-approval on-request` | 읽기만 가능, 승인 필요 |
| CI용 비대화형 | `--sandbox read-only --ask-for-approval never` | 읽기만, 승인 요청 없음 |
| 자동 편집 (프리셋) | `--full-auto` (`--sandbox workspace-write --ask-for-approval on-request`) | 워크스페이스 내 자동, 외부는 승인 |
| YOLO (비권장) | `--dangerously-bypass-approvals-and-sandbox` (`--yolo`) | 샌드박스 없음, 승인 없음 |

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

Auto-Review는 승인 요청을 사용자 대신 리뷰어 에이전트가 평가하는 기능입니다.

```toml
approval_policy = "on-request"
approvals_reviewer = "auto_review"    # user | auto_review
```

- **동작**: 샌드박스 에스컬레이션, 네트워크 요청, 파괴적 MCP 도구 호출 등 승인이 필요한 작업을 자동으로 검토
- **검토 내용**: 데이터 유출, 자격 증명 탐색, 보안 약화, 파괴적 행동 확인
- **리스크 레벨**: 낮음/중간은 정책에 따라 자동 승인, 높음은 충분한 사용자 권한 필요, 치명적은 항상 거부
- **커스터마이징**: `guardian_policy_config`로 테넌트별 검토 정책 교체 가능

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

---

## 네트워크 제어

### 클라우드 에이전트

- **기본**: 인터넷 접근 차단 (보안 및 프롬프트 인젝션 방지)
- **설정**: 관리자가 허용 목록(allowlist)으로 도메인 및 HTTP 메서드 제어 가능

### 로컬 에이전트

- `workspace-write` 모드에서 **기본 네트워크 접근 꺼짐**
- 활성화하려면 명시적 설정 필요:

```toml
[sandbox_workspace_write]
network_access = true
```

### 샌드박스 네트워크 프록시

네트워크가 활성화된 경우, `network_proxy` 기능으로 트래픽을 제어할 수 있습니다.

```toml
[features.network_proxy]
enabled = true
domains = { "api.openai.com" = "allow", "example.com" = "deny" }
```

**도메인 규칙**:

| 패턴 | 매칭 |
| --- | --- |
| 정확한 호스트 | 해당 호스트만 |
| `*.example.com` | 서브도메인만 |
| `**.example.com` | apex + 서브도메인 |
| `*` (allow only) | 모든 공개 호스트 |
| `deny` | 항상 `allow`보다 우선 |

### 웹 검색 제어

```toml
web_search = "cached"     # 기본값: OpenAI 관리 인덱스 사용
# web_search = "live"     # 실시간 웹 검색
# web_search = "disabled" # 웹 검색 비활성화
```

> 프롬프트 인젝션 공격 위험 때문에 네트워크 접근이나 웹 검색 활성화 시 주의가 필요합니다.

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
