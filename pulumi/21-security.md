# Pulumi 보안과 컴플라이언스

> https://www.pulumi.com/docs/iac/operations/iac-least-privileges/
> https://www.pulumi.com/docs/administration/organizations-teams/
> https://www.pulumi.com/docs/administration/security-compliance/
> https://www.pulumi.com/docs/administration/security-compliance/audit-logs/
> https://www.pulumi.com/docs/administration/security-compliance/customer-managed-keys/
> https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/
> https://www.pulumi.com/docs/administration/access-identity/rbac/roles/
> https://www.pulumi.com/docs/administration/access-identity/rbac/teams/
> https://www.pulumi.com/docs/administration/access-identity/saml/
> https://www.pulumi.com/docs/administration/access-identity/scim/
> https://www.pulumi.com/docs/administration/access-identity/access-tokens/
> https://www.pulumi.com/docs/administration/self-hosting/operations/security-hardening/
> https://www.pulumi.com/docs/iac/concepts/secrets/
> https://www.pulumi.com/docs/administration/self-hosting/
> https://www.pulumi.com/docs/iac/cli/commands/pulumi_login/
> https://www.pulumi.com/docs/reference/cloud-rest-api/oauth-token-exchange/
> https://www.pulumi.com/docs/iac/cli/commands/pulumi_stack_change-secrets-provider/

Pulumi IaC는 인프라 프로비저닝 시 최소 권한(Least Privilege) 보안 태세를 채택하여 보안과 컴플라이언스를 동시에 충족한다. Pulumi Cloud는 RBAC(Role-Based Access Control), OIDC(OpenID Connect) 인증, 시크릿 암호화, 감사 로그, SAML SSO, SCIM 사용자 프로비저닝, Customer Managed Keys 등 다계층 보안 기능을 제공하며, Business Critical 에디션에서는 셀프 호스팅을 통한 완전한 인프라 제어가 가능하다.

---

## 최소 권한(Least Privilege) 원칙

### IaC 실행의 보안 함의

Pulumi IaC 프로그램은 완전한 애플리케이션으로, 런타임 환경의 전체 권한으로 실행된다. 이는 IaC 도구의 본질적 특성이며, 개발자가 IaC 코드를 직접 실행할 수 있다면 해당 프로그램이 사용하는 시크릿에도 접근할 수 있음을 의미한다. 따라서 프로덕션 환경에서는 격리된 보안 환경에서 배포를 실행해야 한다.

### 환경별 접근 권한 전략

| 환경 | 권한 수준 | 설명 |
|---|---|---|
| 개발/테스트 | 직접 실행 허용 | `pulumi up` 직접 실행, 빠른 반복과 디버깅 가능. ESC 환경 접근 허용 |
| 프로덕션 | 직접 실행 제한 | PR 승인 프로세스 필수, 격리된 CI/CD 시스템에서 배포 |

### 최소 권한 구현 단계

1. **스택 및 ESC 권한 구성:** RBAC를 사용하여 조직 수준 기본 권한을 `None` 또는 `Read`로 설정. 필요한 경우에만 `Write`/`Admin` 부여
2. **팀 기반 권한 설정:** Settings > Teams에서 사용자를 팀으로 조직화. 스택 및 ESC 권한을 조직 역할에 맞게 명시적으로 할당
3. **프로덕션 보안 배포 접근 방식 선택:** Pulumi Deployments, OIDC 기반 CI/CD, 또는 다른 CI/CD 프로바이더 중 선택

---

## RBAC(역할 기반 접근 제어)

### 기본 조직 역할

| 역할 | 설명 |
|---|---|
| `Admin` | 조직의 모든 리소스와 설정에 대한 전체 접근 권한. 멤버·역할·조직 설정 관리 가능 |
| `Member` | 조직 리소스 조회 및 스택 작업 참여 가능. 조직 설정 수정 불가 |
| `Billing Manager` | 결제 정보 조회 및 관리만 가능. 기타 조직 설정이나 리소스 수정 불가 |

> **참고:** Teams 기능은 Pulumi Enterprise 및 Business Critical 에디션에서만 사용할 수 있다.

### 조직 기본 역할(Organization Default Role)

조직에 커스텀 역할이 있는 경우, **조직 기본 역할**을 설정할 수 있다. 기본 역할은 `Member` 조직 역할을 가지면서 명시적으로 커스텀 역할을 할당받지 않은 모든 멤버에게 자동으로 적용되는 커스텀 역할이다.

**설정 방법:** Settings > Roles에서 커스텀 역할을 열고 **Set as default role**을 선택한다.

> **참고:** 기본 역할은 Settings > Roles(접근 관리 페이지가 아님)에서 설정한다.

### 팀 기반 권한 관리

Pulumi Cloud는 팀을 통한 RBAC를 제공한다. 조직 관리자는 팀에 스택 권한을 일괄 할당할 수 있다.

**팀 생성 및 권한 할당 흐름:**

1. **Settings > Teams**에서 팀 생성 (기본적으로 조직 관리자만 생성 가능. Settings > Access Management에서 **Allow organization members to create teams** 토글을 켜면 모든 멤버가 생성 가능)
2. 스택 권한 할당: `Read`, `Write`, `Admin`
3. ESC 환경 권한 할당: `Environment reader`, `opener`, `editor`, `admin`
4. Insights 계정 권한 할당
5. 스택 초기화 시 팀 권한 부여:

```bash
pulumi stack init --teams YourTeamName
```

### 팀 내 접근 유형

| 유형 | 설명 |
|---|---|
| `Team admin` | 팀 멤버 추가 가능. Settings > Teams에서 멤버 역할 변경 가능 |
| `Team member` | 기본 역할, 팀에 할당된 권한만 행사 |

### Role-backed Teams

커스텀 역할이 활성화된 조직에서는 팀에 역할(default 또는 커스텀)을 할당할 수 있다. 이를 **Role-backed teams**라고 한다.

- 팀에 여러 역할을 할당할 수 있다(다중 역할 할당).
- 팀 멤버는 자신의 사용자 역할과 팀에 할당된 모든 역할의 권한을 합산하여 받는다(합성성, Composability).
- 역할 할당 관리에는 `role:update` 및 `team:update` 스코프를 가진 역할이 필요하다. 팀 관리자(Team admin)만으로는 충분하지 않다.

**Role-backed teams 설정 흐름:**

1. 커스텀 역할 생성 (예: 특정 스택 또는 태그 기반 규칙만 포함)
2. Settings > Teams에서 팀 생성
3. 팀의 **Access** 탭에서 **Add role**로 역할 할당
4. 멤버 추가 — 멤버는 팀의 역할 권한을 자동으로 획득

### 팀 엔티티 접근 권한 부여(Entity Access Grants)

팀 관리자는 조직 수준 역할 관리 권한 없이도 팀의 스택, 환경, Insights 계정에 대한 직접 접근 권한을 관리할 수 있다. 이를 통해 팀이 자체 엔티티 접근을 자율 관리하면서도 더 광범위한 역할 관리는 중앙 집중화된다.

### 팀 환경 권한 REST API 관리

팀 환경 권한은 Pulumi Cloud REST API를 통해 프로그래밍 방식으로 관리할 수 있다. 세 가지 작업 모두 동일한 엔드포인트를 사용한다.

```bash
# 환경 권한 추가
curl -s -X PATCH \
    "https://api.pulumi.com/api/orgs/{orgName}/teams/{teamName}" \
    -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"addEnvironmentPermission":{"projectName":"{projectName}","envName":"{envName}","permission":"read"}}'

# pulumi cloud api 명령으로 동등하게 실행
pulumi cloud api PATCH /orgs/{orgName}/teams/{teamName} \
    -- --body '{"addEnvironmentPermission":{"projectName":"{projectName}","envName":"{envName}","permission":"read"}}'

# 환경 권한 수정
curl -s -X PATCH \
    "https://api.pulumi.com/api/orgs/{orgName}/teams/{teamName}" \
    -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"editEnvironmentPermission":{"projectName":"{projectName}","envName":"{envName}","permission":"write"}}'

# 환경 권한 제거
curl -s -X PATCH \
    "https://api.pulumi.com/api/orgs/{orgName}/teams/{teamName}" \
    -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"removeEnvironment":{"projectName":"{projectName}","envName":"{envName}"}}'
```

> **참고:** 성공 시 `204 No Content`를 반환한다. `projectName`은 환경이 속한 ESC 프로젝트 이름이다(명시적 프로젝트를 지정하지 않은 경우 `default`).

### 커스텀 역할

Enterprise/Business Critical 에디션에서는 커스텀 역할을 생성하여 세분화된 접근 제어가 가능하다. 커스텀 역할은 엔티티 접근 규칙(직접, 전역, 태그 기반)과 조직 접근 수준을 조합하여 구성한다.

**생성 방법:** Settings > Roles에서 **Create custom role**을 클릭한다. 고유한 이름과 설명을 입력한 후 규칙을 추가한다.

| 규칙 유형 | 설명 |
|---|---|
| Direct entity access | 개별 엔티티(스택, 환경, Insights 계정)에 직접 권한 부여. 엔티티 유형을 선택 후 **Select specific [type]**으로 검색 가능한 목록에서 선택 |
| Global entity access | 해당 유형의 모든 엔티티에 권한 일괄 부여. 현재 및 향후 생성되는 모든 엔티티에 적용 |
| Tag-based rules (ABAC) | 태그 조건(`env=production`, `team exists` 등)에 따라 권한 부여. 대규모 조직에서 개별 리소스 나열 없이 태그로 일괄 접근 부여 |

**태그 기반 규칙(ABAC) 구성 요소:**

| 구성 요소 | 설명 |
|---|---|
| Entity type | `Stack`, `Environment`, `Insights Account` 중 선택 |
| Tag conditions | 하나 이상의 태그 키/값 조건 (예: `tag env equals production`, `tag team exists`) |
| Permission set | 조건이 일치할 때 부여할 권한 세트 |

> **참고:** 태그 기반 규칙은 Pulumi Cloud UI 및 API에서 "tag rules" 또는 "tag-based access control rules"로 표시된다. ABAC(Attribute-Based Access Control)는 업계 일반 용어이다.

### 역할 할당

역할은 조직 액세스 토큰, 사용자, 팀에 할당할 수 있다. 유효 권한은 사용자의 조직 역할과 소속 팀에 할당된 모든 역할의 합집합이다.

| 대상 | 설명 |
|---|---|
| 조직 액세스 토큰 | 하나의 역할(default 또는 커스텀)을 할당. 해당 역할의 권한으로 제한 |
| 사용자 | 각 멤버는 하나의 조직 역할(Admin, Member, Billing Manager, 또는 커스텀 역할)을 가짐. Settings > Access Management에서 **Assign custom roles to users** 활성화 시 개별 멤버에게 커스텀 역할 할당 가능 |
| 팀 | 여러 역할(default 또는 커스텀)을 할당 가능. 팀 멤버는 자신의 역할 + 팀 역할의 합산 권한 획득 |

### 스택, ESC 환경 및 Insights 계정 권한

| 권한 | 스택 | ESC 환경 | Insights 계정 |
|---|---|---|---|
| `None` | 접근 불가 | - | - |
| `Read` | 스택 상태 조회 | 환경 정의 조회 (시크릿 복호화 불가) | Insights 계정 조회 |
| `Write` | 스택 업데이트 | 환경 업데이트 | - |
| `Admin` | 전체 관리 | 환경 생성/수정/삭제 | 전체 관리 |
| `open` (`Environment opener`) | - | 시크릿 복호화 및 동적 자격 증명 조회 | - |

**ESC 환경 권한 값(API/CLI):**

| 값 | 콘솔 레이블 | 설명 |
|---|---|---|
| `read` | Environment reader | 환경 정의 조회. 시크릿 복호화 및 동적 자격 증명 조회 불가 |
| `open` | Environment opener | 시크릿 복호화 및 동적 자격 증명 조회 가능 |
| `write` | Environment editor | 환경 열기 및 업데이트 가능 |
| `admin` | Environment admin | 환경 열기, 업데이트, 삭제 가능 |

---

## OIDC(OpenID Connect) 인증

### 개요

OIDC Issuers를 사용하면 외부 서비스가 하드코딩된 자격 증명 없이 Pulumi Cloud 액세스 토큰을 안전하게 획득할 수 있다. 장기 수명의 Pulumi 액세스 토큰을 CI 시스템에 시크릿으로 저장하는 대신, 외부 서비스를 신뢰할 수 있는 OIDC Issuer로 등록하여 단기 수명 토큰을 교환받는다.

> **방향성:** OIDC Issuers는 외부 서비스의 인바운드 토큰을 Pulumi 토큰으로 변환하는 방식을 구성한다. Pulumi Cloud에서 다른 서비스로 토큰을 발급하는 데는 사용되지 않는다.

### 토큰 교환 흐름

1. 외부 워크로드가 호스트 서비스에서 OIDC id_token 획득
2. 해당 id_token을 Pulumi Cloud에 전달하여 단기 Pulumi 액세스 토큰으로 교환
3. Pulumi 액세스 토큰으로 Pulumi 작업 수행

> **OIDC 토큰의 admin 권한:** OIDC를 통해 발급된 토큰은 기본적으로 admin 권한을 갖지 않는다. 스택 생성/삭제 등 상위 권한이 필요한 작업을 수행하려면 토큰 교환 시 `scope` 파라미터에 `admin`을 명시해야 한다. 예: `scope: "admin"` 또는 `scope: "team:my-team admin"`. admin scope 없이 발급된 OIDC 토큰으로 스택 생성/삭제를 시도하면 권한 부족 오류가 발생한다.

### 에디션별 사용 가능 토큰 유형

| 에디션 | 사용 가능 토큰 유형 |
|---|---|
| Individual | `personal` |
| Team | `personal`, `organization` |
| Enterprise | `personal`, `organization`, `team` |
| Business Critical | `personal`, `organization`, `team`, `deployment-runner` |

### OIDC Issuer 등록 및 관리

OIDC Issuer는 세 가지 방식으로 관리할 수 있다:

| 관리 방식 | 설명 |
|---|---|
| Pulumi Cloud UI | Settings > Access Management > OIDC Issuers에서 구성 |
| REST API | OIDC Issuers REST API 참조 |
| Pulumi Service Provider | `OidcIssuer` 리소스를 사용하여 코드로 관리 |

**UI 등록 필드:**

| 필드 | 설명 |
|---|---|
| Name | Issuer 식별용 레이블 |
| URL | Issuer URL (`/.well-known/openid-configuration`이 자동 추가됨) |
| Max expiration | 발급되는 토큰의 최대 만료 시간 (기본값: 25시간) |
| Thumbprints (선택) | TLS 인증서의 SHA-256 지문 |

### 인증 정책(Authorization Policies)

새 OIDC Issuer를 등록하면 기본적으로 모든 토큰 교환을 거부하는 정책이 생성된다. 명시적인 **Allow** 정책을 추가해야 한다.

| 정책 필드 | 설명 |
|---|---|
| Token type | `Organization`, `Team`, `Personal`, `Deployment Runner` 중 선택 |
| Audience (`aud`) | 토큰 대상 검증 (예: `urn:pulumi:org:<org-name>`) |
| Subject (`sub`) | 토큰 주체 검증 (예: `repo:<organization>/<repo>:*`) |
| Decision | `Allow`로 설정해야 토큰 교환 허용 |

**클레임 값 와일드카드 지원:**

| 패턴 | 의미 |
|---|---|
| `*` | 0개 이상의 문자 일치 |
| `?` | 0개 또는 1개의 문자 일치 |
| `.` | 정확히 1개의 문자 일치 |

예: `runner-*`는 `runner-`로 시작하는 모든 파드 이름과 일치.

**중첩 클레임 경로:** JSON 토큰 페이로드의 중첩된 객체를 타겟팅하려면 클레임 경로를 정의한다. 예: `"kubernetes.io".pod.name`

**Thumbprint 구성(선택):** 기본적으로 Pulumi Cloud는 등록 시점의 인증서 지문을 저장한다. 프로바이더가 여러 인증서를 사용하거나 인증서 교체를 지원해야 하는 경우 수동으로 구성한다.

```bash
# 인증서 지문 계산
openssl s_client -servername example.com -showcerts -connect example.com:443
# 첫 번째 인증서를 certificate.crt로 저장 후
openssl x509 -in certificate.crt -fingerprint -sha256 -noout > sha256
# 콜론 제거 후 Thumbprints 필드에 입력
```

### CLI를 통한 OIDC 로그인

Pulumi CLI는 `pulumi login`을 통해 OIDC 토큰 교환을 기본 지원한다. 대부분의 사용 사례에 권장되는 방식이다.

**주요 OIDC 로그인 플래그:**

| 플래그 | 설명 |
|---|---|
| `--oidc-token` | OIDC 토큰. 원시 토큰 값 또는 `file://` 접두사가 있는 파일 경로 |
| `--oidc-org` | 토큰 교환 audience에 사용할 조직 |
| `--oidc-team` | 팀 토큰 교환 시 대상 팀 |
| `--oidc-user` | 개인 토큰 교환 시 대상 사용자 |
| `--oidc-expiration` | 클라우드 백엔드 액세스 토큰 만료 시간. duration 형식 사용 (예: `15m`, `24h`) |

```bash
# 기본 OIDC 로그인
pulumi login --oidc-token <token> --oidc-org <org-name>

# 파일 경로에서 토큰 읽기 (file:// 접두사 사용)
pulumi login --oidc-token file:///path/to/token --oidc-org <org-name>

# 추가 옵션: 팀, 사용자, 만료 시간 지정
pulumi login --oidc-token <token> --oidc-org <org-name> \
    --oidc-team <team-name> \
    --oidc-expiration <duration>

# 개인 토큰 교환 시 사용자 지정
pulumi login --oidc-token <token> --oidc-org <org-name> \
    --oidc-user <user-login>

# 만료 시간은 duration 형식 사용 (예: 15m, 24h)
pulumi login --oidc-token <token> --oidc-org <org-name> \
    --oidc-expiration 15m
```

> **주의:** `OIDC token exchange failed` 오류 발생 시 백엔드 URL을 명시적으로 포함해야 한다:
> `pulumi login https://api.pulumi.com --oidc-token <token> --oidc-org <org-name>`

### REST API를 통한 토큰 교환

고급 시나리오에서 직접 토큰 교환을 제어해야 하는 경우 OAuth 2.0 토큰 엔드포인트를 호출한다. `application/json`과 `application/x-www-form-urlencoded` 모두 지원한다. 자세한 API 레퍼런스는 [OAuth Token Exchange](https://www.pulumi.com/docs/reference/cloud-rest-api/oauth-token-exchange/) 문서를 참조한다.

**파라미터:**

| 파라미터 | 설명 |
|---|---|
| `audience` | `urn:pulumi:org:{ORG_NAME}` |
| `grant_type` | `urn:ietf:params:oauth:grant-type:token-exchange` |
| `subject_token_type` | `urn:ietf:params:oauth:token-type:id_token` |
| `requested_token_type` | 조직 토큰: `urn:pulumi:token-type:access_token:organization`, 팀 토큰: `urn:pulumi:token-type:access_token:team`, 개인 토큰: `urn:pulumi:token-type:access_token:personal`, Deployment Runner 토큰: `urn:pulumi:token-type:access_token:runner` (Business Critical 에디션 전용) |
| `scope` | 팀 또는 개인 토큰 요청 시 필요. 형식: `team:{TEAM_NAME}` 또는 `user:{USER_LOGIN}` |
| `expiration` | 토큰 만료 시간(초). 기본값: 2시간 |
| `subject_token` | OIDC 프로바이더가 발급한 id_token |

```bash
curl -X POST \
    -H 'Content-Type: application/x-www-form-urlencoded' \
    -d 'audience=urn:pulumi:org:<ORG_NAME>' \
    -d 'grant_type=urn:ietf:params:oauth:grant-type:token-exchange' \
    -d 'subject_token_type=urn:ietf:params:oauth:token-type:id_token' \
    -d 'requested_token_type=urn:pulumi:token-type:access_token:organization' \
    -d 'subject_token=<YOUR_ID_TOKEN>' \
    https://api.pulumi.com/api/oauth/token
```

**응답 예시:**

```json
{
    "access_token": "...",
    "issued_token_type": "urn:pulumi:token-type:access_token:organization",
    "token_type": "token",
    "expires_in": 7200,
    "scope": ""
}
```

### 프로덕션 보안 배포 접근 방식

프로덕션과 같은 민감한 환경에서는 다음 세 가지 배포 접근 방식 중 하나를 선택해야 한다.

#### Option A: Pulumi Deployments (관리형 자동 배포)

Pulumi Deployments는 자동화되고 관리되며 안전한 인프라 배포를 제공한다.

| 기능 | 설명 |
|---|---|
| 자동 GitHub 연동 | PR에 대해 `pulumi preview` 자동 실행, PR 병합 시 `pulumi up` 자동 실행 |
| REST API | 커스텀 워크플로우나 서드파티 CI/CD 시스템에서 프로그래밍 방식으로 배포 트리거 |

**GitHub 연동 설정 단계:**

1. **Pulumi GitHub App 설치:** Pulumi Cloud 콘솔 → Management > Version control에서 Add account 선택 후 GitHub 선택, GitHub 리포지토리 접근 권한 부여
2. **배포 트리거 구성:** Pulumi Cloud 콘솔에서 스택 → Stack Settings > Deploy에서 배포 트리거 설정 (예: main 브랜치로 PR 병합 시)

> **참고:** 자세한 내용은 [Pulumi Deployments GitHub 연동](https://www.pulumi.com/docs/deployments/github-integration/) 및 [Pulumi Deployments REST API](https://www.pulumi.com/docs/deployments/api/) 문서를 참조.

#### Option B: GitHub Actions + OIDC 인증

GitHub Actions에서 Pulumi의 OIDC 연동을 사용하여 토큰 없는 안전한 배포를 수행한다. Pulumi Cloud가 GitHub의 OIDC 프로바이더를 신뢰하도록 구성하고, 팀 범위의 Pulumi 액세스 토큰을 획득하여 배포를 실행한다.

```yaml
# TypeScript/공통 워크플로우 (Team-Scoped)
name: Pulumi Deployment

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Authenticate with Pulumi (Team-scoped)
        uses: pulumi/auth-actions@v1
        with:
          organization: <YOUR_ORG>
          requested-token-type: urn:pulumi:token-type:access_token:team
          team: <YOUR_TEAM>

      - name: Deploy Infrastructure
        uses: pulumi/actions@v6
        with:
          command: up
          stack-name: <YOUR_ORG>/<YOUR_STACK>
```

```python
# Python 프로젝트의 경우 pulumi/actions 단계에서 동일하게 사용
# main 브랜치에 Push 시 자동 배포
```

> **참고:** `pulumi/auth-actions`는 OIDC 토큰을 팀 범위의 Pulumi 액세스 토큰으로 교환하며, `pulumi/actions`는 할당된 팀 권한에 따라 `pulumi up`을 실행한다. 자세한 내용은 [GitHub OIDC 설정](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/github/) 및 [Pulumi GitHub Actions](https://www.pulumi.com/docs/using-pulumi/continuous-delivery/github-actions/) 문서를 참조.

#### Option C: 다른 CI/CD 프로바이더

Pulumi는 GitHub Actions 외에도 다음과 같은 주요 CI/CD 플랫폼과 연동된다.

| 프로바이더 | 설명 |
|---|---|
| GitLab CI/CD | GitLab 파이프라인에서 Pulumi 배포 실행 |
| Azure DevOps Pipelines | Azure DevOps에서 Pulumi 배포 자동화 |
| Jenkins | Jenkins 파이프라인과 Pulumi 연동 |
| CircleCI | CircleCI 워크플로우에서 Pulumi 실행 |

이들 플랫폼도 보안 OIDC 인증 또는 토큰 기반 워크플로우를 활용할 수 있으며, CI/CD 프로바이더의 `sub`, `aud`, 커스텀 클레임을 사용하여 특정 파이프라인에 대한 접근을 제한할 수 있다.

> **참고:** 각 프로바이더별 상세 설정 가이드는 [Continuous Delivery 문서](https://www.pulumi.com/docs/using-pulumi/continuous-delivery/)를 참조.

### 지원 OIDC 프로바이더

| 프로바이더 | 문서 경로 |
|---|---|
| GitHub Actions | `/docs/administration/access-identity/oidc-issuers/github/` |
| GitLab CI | `/docs/administration/access-identity/oidc-issuers/gitlab/` |
| Amazon EKS | `/docs/administration/access-identity/oidc-issuers/kubernetes-eks/` |
| Google Kubernetes Engine | `/docs/administration/access-identity/oidc-issuers/kubernetes-gke/` |

> **참고:** 위에 나열된 프로바이더 외에도 OIDC id_token을 발급할 수 있는 모든 서드파티 서비스를 OIDC Issuer로 등록할 수 있다.

---

## 액세스 토큰 관리

### 토큰 유형

| 유형 | 설명 | 가용 에디션 |
|---|---|---|
| Personal token | 생성한 사용자와 동일한 권한. 모든 조직 멤버십 포함 | 모든 에디션 |
| Organization token | 조직 자체로 인증. 감사 로그에 조직으로 기록 | Enterprise, Business Critical |
| Team token | 특정 팀으로 인증. 팀의 역할에 따른 권한 | Enterprise, Business Critical |

> **레거시 조직 토큰(Legacy Organization Tokens):** RBAC가 도입되기 전에 생성된 조직 토큰은 **Standard**(현재 RBAC의 `Member` 역할에 해당) 또는 **Admin** 고정 권한을 가진다. 이 레거시 토큰들은 여전히 작동하지만, 새로운 자동화에는 RBAC 기반 커스텀 역할을 할당한 조직/팀 토큰을 사용하는 것이 권장된다. 기존 Standard/Admin 토큰을 사용 중인 경우, RBAC 마이그레이션을 위해 해당 토큰을 삭제하고 커스텀 역할이 할당된 새 조직 토큰으로 교체해야 한다.

### 토큰별 권한(RBAC)

| 토큰 유형 | 권한 결정 방식 |
|---|---|
| Personal token | 생성한 사용자의 모든 조직 멤버십, 팀 멤버십, 역할 할당을 그대로 상속. 사용자가 속한 모든 Pulumi Cloud 조직의 권한 포함 |
| Organization token | 생성 시 할당된 RBAC 역할에 따라 권한 결정. 명시적 역할이 없으면 조직의 기본 멤버 역할(default member role)을 받음. 개인 토큰과 달리 토큰이 생성된 단일 조직으로만 접근이 제한됨. 역할을 통해 읽기 전용 접근부터 전체 관리 제어까지 세밀하게 조정 가능 |
| Team token | 토큰이 속한 팀에 할당된 역할에 따라 실시간으로 권한 결정. 팀의 역할 할당이 변경되면 토큰의 권한도 즉시 반영됨. 특정 팀이 관리하는 리소스에만 접근을 제한하는 CI/CD 파이프라인에 적합 |

> **참고:** 액세스 토큰은 조직의 접근 관리 설정에서 모든 멤버의 스택 생성을 허용하거나, 토큰에 할당된 역할에 `stack:create` 스코프가 포함된 경우 스택을 생성할 수 있다. Admin 조직 토큰은 항상 이 기능을 갖는다. 스택 생성자는 자동으로 소유자가 되며 삭제 권한을 포함한 모든 스택 권한을 갖는다.

### 토큰 보안 모범 사례

| 권장 사항 | 설명 |
|---|---|
| 만료 기간 설정 | 최대 2년까지 만료 기간 설정 권장. 만료된 토큰은 갱신 불가 |
| Organization/Team 토큰 사용 | CI/CD 파이프라인에서는 개인 토큰 대신 조직/팀 토큰 사용 |
| 최소 권한 역할 할당 | 커스텀 역할을 통해 자동화에 필요한 최소 권한만 부여 |
| 즉시 삭제 | 유출 의심 시 즉시 삭제. 삭제된 토큰 이름은 감사 로그 보존을 위해 영구 예약됨 |

---

## 시크릿 관리

### 시크릿 암호화 개요

Pulumi Cloud는 전체 상태 파일을 안전하게 전송 및 저장하며, 개별 값을 시크릿으로 추가 암호화하는 기능도 제공한다. 기본적으로 Pulumi Cloud가 관리하는 스택별 자동 암호화 키를 사용하며, 자체 암호화 프로바이더도 선택할 수 있다.

> **참고:** Pulumi CLI는 클라우드 자격 증명을 Pulumi Cloud로 전송하지 않는다.

### 프로그래밍 방식 시크릿 생성

**TypeScript:**

```typescript
const cfg = new pulumi.Config();
const param = new aws.ssm.Parameter("a-secret-param", {
    type: "SecureString",
    value: cfg.requireSecret("my-secret-value"),
});
```

**Python:**

```python
cfg = pulumi.Config()
param = ssm.Parameter("a-secret-param",
    type="SecureString",
    value=cfg.require_secret("my-secret-value"))
```

### 시크릿 전파(Transitiv Secret Tracking)

Pulumi는 시크릿의 전이적 사용을 추적한다. 시크릿 입력에서 파생된 데이터도 자동으로 시크릿으로 표시되며, 시크릿이 포함된 모든 리소스 속성이 암호화된다. `apply`나 `Output.all`로 시크릿을 결합하면 결과 출력도 시크릿으로 표시된다.

### CLI 시크릿 설정

```bash
# 시크릿 값 설정
pulumi config set --secret dbPassword S3cr37

# 구조화된 설정 내 시크릿
pulumi config set --path --secret endpoints[0].token <YOUR_TOKEN>
```

### 시크릿 암호화 프로바이더

| 프로바이더 | 형식 | 인증 방식 |
|---|---|---|
| `awskms` | `awskms://<key-id>?region=<region>` | AWS CLI 자격 증명 또는 `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY` |
| `azurekeyvault` | `azurekeyvault://<vault>.vault.azure.net/keys/<key>` | Azure DefaultAzureCredential 체인 |
| `gcpkms` | `gcpkms://projects/<proj>/locations/<loc>/keyRings/<ring>/cryptoKeys/<key>` | Google Cloud Application Default Credentials |
| `hashivault` | `hashivault://<key-name>` | `VAULT_SERVER_URL` + `VAULT_SERVER_TOKEN` 환경 변수 |

**스택 초기화 시 암호화 프로바이더 지정:**

```bash
# AWS KMS 예시
pulumi stack init my-stack \
    --secrets-provider="awskms://1234abcd-12ab-34cd-56ef-1234567890ab?region=us-east-1"

# Azure Key Vault 예시
pulumi stack init my-stack \
    --secrets-provider="azurekeyvault://<YOUR_VAULT>.vault.azure.net/keys/<YOUR_KEY>"

# Google Cloud KMS 예시
pulumi stack init my-stack \
    --secrets-provider="gcpkms://projects/<YOUR_PROJECT>/locations/<LOCATION>/keyRings/<KEYRING>/cryptoKeys/<KEY>"

# HashiCorp Vault 예시
pulumi stack init my-stack \
    --secrets-provider="hashivault://<YOUR_KEY>"
```

**기존 스택의 암호화 프로바이더 변경:**

```bash
pulumi stack change-secrets-provider "<secrets-provider>"
```

지원 프로바이더: `default`, `passphrase`, `awskms`, `azurekeyvault`, `gcpkms`, `hashivault`

**`pulumi stack change-secrets-provider` CLI 옵션:**

| 플래그 | 설명 |
|---|---|
| `-s`, `--stack` | 대상 스택 이름. 생략 시 현재 스택 사용 |
| `-h`, `--help` | 도움말 |

자세한 CLI 레퍼런스는 [pulumi stack change-secrets-provider](https://www.pulumi.com/docs/iac/cli/commands/pulumi_stack_change-secrets-provider/) 문서를 참조한다.

### AWS KMS 고급 옵션

```bash
# AWS SDK v2 + 프로필 지정
awskms://<key-id>?region=us-east-1&awssdk=v2&profile=dev

# Encryption Context 지정 (CLI v3.41.1+)
awskms://<key-id>?region=us-east-1&awssdk=v2&profile=dev&context_project=myproject&context_environment=staging
```

Encryption Context는 IAM 정책 조건 및 CloudTrail 로그에 표시된다.

### Customer Managed Keys (CMK)

> **참고:** CMK는 Enterprise 및 Business Critical 에디션에서 사용 가능. 현재 Pulumi ESC 데이터 암호화에만 지원되며, AWS KMS만 지원. 추가 KMS 프로바이더 및 다른 Pulumi 제품으로의 확장을 진행 중이다.

CMK를 사용하면 Pulumi Cloud의 데이터 암호화에 자체 KMS 키를 사용할 수 있다. CMK는 데이터 키를 암호화하며, 암호화된 데이터 키가 Pulumi Cloud의 실제 데이터를 암호화한다. 조직 관리자만 CMK를 관리할 수 있다.

**CMK 페이지 확인:** Settings > Organization > Customer Managed Keys 탭에서 각 키의 Name, Type, Default 여부를 확인할 수 있다.

#### AWS KMS 키 추가

1. AWS IAM에 역할을 설정하고 AWS KMS에 키를 생성
2. Pulumi Cloud의 Customer Managed Keys 설정 페이지에서 **Add Customer Managed Key** 클릭
3. 고유한 키 이름 입력
4. Role ARN 입력 (AWS KMS 키에 접근 권한이 있는 IAM 역할)
5. Key ARN 입력 (별칭 ARN도 지원)

| 작업 | 설명 |
|---|---|
| 키 추가 | 첫 CMK 추가 시 기존 데이터 키가 자동으로 새 CMK로 재암호화됨. 암호화된 데이터 자체는 변경되지 않음 |
| 기본 키 설정 | 모든 신규 데이터 키가 이 키로 암호화됨. 이미 기본 키이거나 재암호화 진행 중인 키는 설정 불가 |
| 키 비활성화 | 새 데이터 키 생성에 사용 불가. 기존 데이터 키는 재암호화 키를 지정해야 함. 기본 키 또는 재암호화 진행 중인 키는 비활성화 불가 |
| 전체 비활성화 | 모든 CMK 비활성화 후 Pulumi 관리 키로 재암호화. 설정 페이지 우측 상단 휠 버튼에서 실행 |

---

## 감사 로그(Audit Logs)

> **참고:** 감사 로그는 Enterprise 및 Business Critical 에디션에서 사용 가능. 조직 관리자만 조회 가능.

### 개요

감사 로그는 조직 내 사용자 활동을 추적한다. 변경 불가능한(immutable) 로그로, 모든 사용자 작업을 기록한다. 이벤트 발생 시간(UNIX 타임스탬프), 작업 수행 사용자, 발생한 이벤트, 호출 소스 IP를 캡처한다.

### 감사 로그 조회

**Settings > Audit Logs**에서 최신 이벤트를 내림차순으로 확인할 수 있다. 특정 사용자로 필터링도 가능하다.

### 자동 내보내기

> **참고:** 자동 내보내기는 Business Critical 에디션 전용.

| 대상 | 설명 |
|---|---|
| AWS S3 | `/docs/administration/security-compliance/audit-logs/aws-s3/` 참조 |
| Microsoft Sentinel | `/docs/administration/security-compliance/audit-logs/azure-sentinel/` 참조 |

### 수동 내보내기

| 방식 | 설명 |
|---|---|
| 콘솔 다운로드 | Settings > Audit Logs > Download |
| REST API | `/docs/reference/service-rest-api#audit-logs` 참조. 빈도가 높은 사용에는 부적합하며, 자동 내보내기 사용 권장 |

### 지원 포맷

| 포맷 | 필드 |
|---|---|
| JSON | `timestamp`, `sourceIP`, `event`, `description`, `user`(login, name, avatar URL) |
| CSV | `Timestamp`, `Name`, `Login`, `Event`, `Description`, `SourceIP`, `RequireOrgAdmin`, `RequireStackAdmin`, `AuthenticationFailure` |
| CEF (Common Event Format) | 표준 헤더 + 확장 필드(`dvchost`, `rt`, `src`, `suser`, `orgID`, `userID`, `requireOrgAdmin`, `requireStackAdmin`, `authenticationFailure`) |

### 주요 감사 로그 이벤트

| 이벤트 | 설명 |
|---|---|
| `User Login` | 사용자 로그인 성공 |
| `User Login Failed` | 사용자 로그인 실패 |
| `User Added New Identity to Their Account` | 사용자가 Pulumi 계정에 새 아이덴티티 연결 |
| `Member Added` | 조직에 멤버 추가 |
| `Member Removed` | 조직에서 멤버 제거 |
| `Member Role Changed` | 멤버 역할 변경 |
| `Organization Settings Changed` | 조직 설정 변경 |
| `Stack Created` | 스택 생성 |
| `Stack Created From Template` | 템플릿으로 스택 생성 |
| `Stack Deleted` | 스택 삭제 |
| `Stack Renamed` | 스택 이름 변경 |
| `Stack Exported` | 스택 내보내기 |
| `Stack Imported` | 스택 가져오기 |
| `Stack Transferred to Organization` | 스택을 다른 조직으로 이전 |
| `Stack Update Started` | 스택 업데이트 시작 |
| `Stack Update Completed` | 스택 업데이트 완료 |
| `Stack Update Canceled` | 스택 업데이트 취소 |
| `Stack Collaborator Added` | 스택 협업자 추가 |
| `Stack Collaborator Permissions Changed` | 스택 협업자 권한 변경 |
| `Stack Collaborator Removed` | 스택 협업자 제거 |
| `Stack Provider Open` | 환경 내 스택 프로바이더 열기 |
| `Secret Decrypted` | 시크릿 값 복호화 |
| `Team Created` | 팀 생성 |
| `Team Updated` | 팀 업데이트 |
| `Team Deleted` | 팀 삭제 |
| `Policy Pack Created` | 정책 팩 생성 |
| `Policy Pack Deleted` | 정책 팩 삭제 |
| `Policy Pack Enabled` | 정책 팩 활성화 |
| `Policy Pack Disabled` | 정책 팩 비활성화 |
| `Policy Group Created` | 정책 그룹 생성 |
| `Policy Group Deleted` | 정책 그룹 삭제 |
| `Policy Group Updated` | 정책 그룹 업데이트 |
| `Environment Created` | ESC 환경 생성 |
| `Environment Updated` | ESC 환경 업데이트 |
| `Environment Deleted` | ESC 환경 삭제 |
| `Environment Open` | ESC 환경 열기 |
| `Environment Read` | 열린 환경 읽기 |
| `Environment Read Open` | 환경 열기 및 읽기 |
| `Environment Unauthorized Open` | 권한 없는 환경 열기 시도 |
| `Environment Decrypted` | ESC 환경 복호화 |
| `Environment Clone` | ESC 환경 복제 |
| `Environment Restored` | ESC 환경 복원 |
| `Environment Rotated` | ESC 환경 시크릿 교체 |
| `Environment Tag Created` | 환경 태그 생성 |
| `Environment Tag Updated` | 환경 태그 업데이트 |
| `Environment Tag Deleted` | 환경 태그 삭제 |
| `Environment Version Retracted` | 환경 버전 철회 |
| `Environment Version Tag Open` | 특정 버전 태그에서 환경 열기 |
| `Environment Version Tag Created` | 환경 버전 태그 생성 |
| `Environment Version Tag Read` | 환경 버전 태그 읽기 |
| `Environment Version Tag Update` | 환경 버전 태그 업데이트 |
| `Environment Version Tag Delete` | 환경 버전 태그 삭제 |
| `Environment Schedule Created` | 환경 스케줄 생성 |
| `Environment Schedule Updated` | 환경 스케줄 업데이트 |
| `Environment Schedule Deleted` | 환경 스케줄 삭제 |
| `Customer Managed Key Added` | CMK 추가 |
| `Customer Managed Key Set Default` | CMK 기본 키 설정 |
| `Customer Managed Key Disabled` | CMK 비활성화 |
| `Customer Managed Key Disabled All` | 모든 CMK 비활성화 |
| `SAML Configuration Updated` | SAML 설정 변경 |
| `Auth Failure Organization Role` | 조직 역할 권한 부족으로 인한 인증 실패 |
| `Auth Failure Stack Permission` | 스택 권한 부족으로 인한 인증 실패 |
| `Auth Failure SCIM Access Token` | SCIM 액세스 토큰 인증 실패 |

---

## SAML SSO 및 SCIM

### SAML SSO

> **참고:** SAML 지원은 Enterprise 및 Business Critical 에디션 필요.

Pulumi Cloud는 SAML 2.0 Identity Provider와 연동할 수 있다. SAML 기반 조직의 멤버는 Single Sign-On을 통해 로그인할 수 있다.

**셀프 호스팅 Pulumi Cloud 사용 시:** 먼저 셀프 호스팅 인프라를 SAML SSO에 맞게 구성(API 서비스 키 및 환경 변수)한 후 IdP 설정을 완료해야 한다.

| 지원 IdP | 문서 경로 |
|---|---|
| Microsoft Entra ID (구 Azure AD) | `/docs/administration/access-identity/saml/entra/` |
| Google Workspace (구 G Suite) | `/docs/administration/access-identity/saml/gsuite/` |
| JumpCloud | `/docs/administration/access-identity/saml/jumpcloud/` |
| Okta | `/docs/administration/access-identity/saml/okta/` |
| Auth0 | `/docs/administration/access-identity/saml/auth0/` |
| OneLogin | `/docs/administration/access-identity/saml/onelogin/` |

> **참고:** Pulumi는 SCIM 애플리케이션당 하나의 조직만 지원한다. 여러 조직을 관리하는 경우 각 조직별로 별도의 SCIM 애플리케이션을 구성해야 한다.

### SCIM 사용자 프로비저닝

> **참고:** SCIM은 Business Critical 에디션 전용.

SCIM 2.0을 통해 IdP에서 사용자와 그룹을 Pulumi Cloud로 동기화할 수 있다. SCIM 관리 팀 외에도 Pulumi Cloud 내에서 로컬 팀을 추가로 구성할 수 있다.

| 지원 IdP | 문서 경로 |
|---|---|
| Microsoft Entra ID | `/docs/administration/access-identity/scim/entra/` |
| Okta | `/docs/administration/access-identity/scim/okta/` |
| OneLogin | `/docs/administration/access-identity/scim/onelogin/` |

---

## 셀프 호스팅 보안 강화

> **참고:** 셀프 호스팅은 Pulumi Business Critical 에디션 전용.

### 배포 옵션

| 옵션 | 설명 |
|---|---|
| Docker Compose (Quickstart) | 로컬 환경 테스트용 |
| ECS-Hosted | AWS ECS 배포 |
| EKS-Hosted | Amazon EKS 배포 |
| AKS-Hosted | Azure Kubernetes Service 배포 |
| GKE-Hosted | Google Kubernetes Engine 배포 |
| BYO Infrastructure | 자체 Kubernetes, MySQL, S3 호환 스토리지 |
| Local-Docker | 커스텀 Docker 환경 |

### 네트워크 보안

| 권장 사항 | 설명 |
|---|---|
| 프라이빗 서브넷 | 데이터베이스와 애플리케이션 컨테이너를 직접 인터넷 접근이 없는 프라이빗 서브넷에 배치 |
| 보안 그룹/네트워크 정책 | 티어 간 트래픽 제한 |
| Ingress Allowlist | `ingressAllowList` 설정으로 IP 대역 기반 접근 제한 |

### 암호화

| 계층 | 권장 사항 |
|---|---|
| 저장 시(At Rest) | 데이터베이스 클러스터 및 오브젝트 스토리지 버킷에서 스토리지 암호화 활성화 |
| 전송 시(In Transit) | 모든 연결에 TLS 적용 |
| 시크릿 | 라이선스 키, TLS 인증서, SMTP 자격 증명, DB 비밀번호 등은 `pulumi config set --secret`으로 저장 |

### TLS 구성 (3계층 홉)

| 홉 | 구간 | 환경 변수 | 기본 포트 |
|---|---|---|---|
| 1 | 클라이언트 → Console | `CONSOLE_TLS_CERTIFICATE`, `CONSOLE_TLS_PRIVATE_KEY`, `CONSOLE_MIN_TLS_VERSION` | HTTP 3000 → HTTPS 3443 |
| 2 | 클라이언트/Console → API | `API_TLS_CERTIFICATE`, `API_TLS_PRIVATE_KEY`, `API_MIN_TLS_VERSION` | HTTP 8080 → HTTPS 8443 |
| 3 | API → MySQL | `DATABASE_CA_CERTIFICATE`, `DATABASE_MIN_TLS_VERSION` | MySQL over TLS |

> 자체 서명 또는 내부 CA 인증서 사용 시 Console 컨테이너에 `NODE_EXTRA_CA_CERTS`를 CA PEM 파일 경로로 설정해야 한다.

### SMTP 및 이메일

| 기능 | 설명 |
|---|---|
| 사용자 초대 워크플로우 | SMTP 필요 |
| 조직 알림 | SMTP 필요 |
| 비밀번호 재설정 | SMTP 필요 (SAML SSO 미사용 시) |
| SAML SSO 전용 환경 | SMTP 선택 사항 |

### 봇 보호

Cloudflare Turnstile를 사용하여 가입 보호를 구성할 수 있다.

| 설정 키 | 설명 |
|---|---|
| `recaptchaSiteKey` | Turnstile 사이트 키 |
| `recaptchaSecretKey` | Turnstile 시크릿 키 |

### 프로덕션 준비 체크리스트

**사전 배포:**

- 배포 플랫폼 선택 (ECS, EKS, AKS, GKE, BYO)
- Pulumi Cloud 라이선스 키 확보
- API 및 Console 도메인 이름 정의
- SMTP 서버 자격 증명 준비
- Cloudflare Turnstile 설정 (공개 접근 설치 시 권장)

**인프라:**

- 데이터베이스 Multi-AZ 구성
- 오브젝트 스토리지 버전 관리 활성화
- 퍼블릭/프라이빗 서브넷 2개 이상의 AZ에 구성
- 보안 그룹으로 티어 간 트래픽 제한

**애플리케이션:**

- API 서비스 및 Console 2개 이상의 레플리카 배포
- API 및 Console 도메인에 DNS 레코드 구성
- 모든 서비스 Health Check 통과

**운영:**

- 모니터링 및 알림 구성 (CPU, 메모리, 에러율, 스토리지)
- 데이터베이스 백업 일정 구성 (교차 리전 복사 포함)
- 오브젝트 스토리지 복제 구성 (멀티 리전 시)
- 복구 절차 문서화 및 테스트
- Ingress Allowlist 구성
- 데이터베이스 및 로드 밸런서에 삭제 보호 활성화

---

## Pulumi ESC를 통한 시크릿 관리

### 다중 팀 시크릿 공유

Pulumi ESC(Environments, Secrets, and Config)를 사용하면 여러 팀이 관리하는 시크릿을 중앙에서 관리할 수 있다. 팀별로 AWS Secrets Manager 등 다른 시크릿 저장소를 사용하더라도, ESC 환경 `imports`를 통해 단일 접근 지점으로 통합할 수 있다.

### 크로스 클라우드 시크릿 통합

ESC를 사용하면 GCP Secrets Manager와 Azure KeyVault 등 여러 클라우드의 시크릿을 하나의 환경에서 통합 관리할 수 있다.

```bash
# 환경 열기
pulumi env open myorg/cross_cloud
```

---

## 보안 모범 사례 요약

| 영역 | 권장 사항 |
|---|---|
| 최소 권한 | 조직 기본 권한을 `None` 또는 `Read`로 설정. 필요한 경우에만 `Write`/`Admin` 부여 |
| 프로덕션 배포 | 직접 IaC 실행 제한. PR 승인 프로세스 + 격리된 CI/CD 사용 |
| 인증 | 장기 토큰 대신 OIDC 기반 토큰 없는 인증 선호 |
| 시크릿 | `--secret` 플래그 또는 `requireSecret` API 사용. 소스 제어에 커밋해도 안전 |
| 암호화 프로바이더 | 규정 요구 시 자체 KMS 키 사용 (AWS KMS, Azure Key Vault, GCP KMS, HashiCorp Vault) |
| 토큰 관리 | 만료 기간 설정. CI/CD에서는 조직/팀 토큰 사용 |
| 감사 | 정기적으로 액세스 및 배포 로그 검토. ESC 로그에서 시크릿 접근 패턴 모니터링 |
| RBAC | 팀 기반 권한으로 세분화된 접근 제어. 태그 기반(ABAC) 규칙으로 대규모 조직 관리 |
| 셀프 호스팅 | 모든 홉에 TLS 적용. 프라이빗 서브넷 사용. 스토리지 암호화 활성화 |
