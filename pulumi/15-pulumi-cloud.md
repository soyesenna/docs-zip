# Pulumi Cloud 관리

> 원문: [Pulumi Cloud](https://www.pulumi.com/docs/iac/concepts/pulumi-cloud/) | [Organizations & Teams](https://www.pulumi.com/docs/administration/organizations-teams/) | [Access & Identity](https://www.pulumi.com/docs/administration/access-identity/) | [Self-Hosting](https://www.pulumi.com/docs/administration/self-hosting/)

Pulumi Cloud는 Pulumi CLI의 기본 상태 백엔드이자, 팀이 대규모로 Pulumi를 운영하는 데 필요한 기능을 제공하는 관리형 플랫폼이다. 액세스 제어, 재사용 가능한 구성 및 시크릿, 정책 강제, 클라우드 리소스 인벤토리, 예약 드리프트 감지, 관리형 배포, AI 에이전트 등의 기능을 제공한다. Pulumi Cloud는 호스팅 SaaS와 셀프 호스팅 에디션으로 제공되며, 개인(Individual) 티어는 무료다.

---

## 조직(Organizations)

> 원문: [Organizations](https://www.pulumi.com/docs/administration/organizations-teams/organizations/)

조직은 Pulumi Cloud의 최상위 계정 단위로, 관련 프로젝트·스택·사용자를 그룹화하는 협업 공간이다. 스택의 정규화된 이름은 `<organization>/<project>/<stack>` 형식을 따른다. Pulumi Cloud 콘솔에서는 `<organization>//<stack>`(프로젝트 구분자로 슬래시 두 개) 형식으로 표시된다.

### 조직 페이지 구성

| 페이지 | 설명 |
|--------|------|
| Dashboard | 최근 업데이트된 스택, 최근 활동, 리소스 수 그래프 등 조직 개요 |
| All Stacks | 프로젝트별 그룹화 및 태그 필터링이 가능한 조직 스택 검색 목록 |
| Policies | 조직 정책 및 정책 그룹 목록. 모범 사례 및 컴플라이언스 가드레일 설정 |
| Settings | 구독/결제 정보, Billing Managers, 스택 권한, 액세스 관리 등 조직 설정 |

### 조직 역할

| 역할 | 설명 |
|------|------|
| Admin | 멤버 초대, 팀/정책 생성, 스택 권한 및 RBAC 관리, 결제 정보 수정, 조직 설정 전체 제어 |
| Member | 접근 권한이 있는 스택의 조회 및 편집, 멤버·팀 조회 가능 |
| Billing Manager | 결제 정보 수정 및 다른 Billing Managers 조회. 스택·팀·정책에 대한 읽기/쓰기 권한 없음 |

### 조직 생성 및 멤버 관리

조직 생성 시 무료 평가판이 시작되며, 평가판 종료 후 Team / Enterprise / Business Critical 에디션 중 선택할 수 있다.

**멤버 초대 방법:**

1. **Settings > Members**로 이동
2. 이메일 주소로 초대(**Invite members**) 또는 초대 링크 복사(**Copy new invite link**)
3. 초대 링크는 만료되지 않으며 1회만 사용 가능

**멤버가 되려면** 기존 조직 관리자의 초대 또는 승인이 필요하다. 서드파티 identity provider(예: GitHub Organization, GitLab Group)가 연결된 경우, 해당 서드파티 조직의 멤버여야 Pulumi 조직에 가입할 수 있다.

### Identity Provider

Pulumi 조직은 다음 identity provider 중 하나로 운영할 수 있다:

| Identity Provider | 설명 |
|-------------------|------|
| Pulumi (기본) | Pulumi 자체 계정으로 멤버십 관리 |
| GitHub Organization | GitHub OAuth app에 `read:org` 스코프 부여 필요. 소스코드 접근 권한 없음 |
| GitLab Group | GitLab 그룹 관리자가 그룹을 추가하고 멤버를 초대. 임시 멤버십 만료 시 Pulumi 접근도 상실 |
| Bitbucket Workspace | Bitbucket OAuth app에 계정 및 워크스페이스 멤버십 읽기 권한 부여 필요 |
| SAML 2.0 SSO | Enterprise / Business Critical 에디션에서 지원 (아래 SAML SSO 섹션 참조) |

Identity provider 변경은 **Settings > Access Management > Other** 탭에서 수행한다. 변경 전 모든 멤버가 새 identity provider를 개인 계정에 연결해야 하며, 그렇지 않으면 조직 접근이 차단된다.

---

## 팀(Teams)

> 원문: [Teams](https://www.pulumi.com/docs/administration/access-identity/rbac/teams/)

팀은 Enterprise 및 Business Critical 에디션에서 사용할 수 있다. 팀을 통해 조직 관리자가 사용자 그룹에 스택 권한 세트를 일괄 할당할 수 있다.

### 팀 접근 유형

| 유형 | 설명 |
|------|------|
| Team Admin | 팀에 멤버를 추가할 수 있음. Entity Access Grants 직접 관리 가능 |
| Team Member | 기본 역할. 새 멤버는 기본적으로 Team Member로 할당 |

### GitHub 기반 팀

Pulumi 조직이 GitHub에 연결된 경우, 기존 GitHub 팀을 Pulumi로 가져올 수 있다. 멤버십은 GitHub에서 관리하고, 스택 권한과 역할 할당은 Pulumi Cloud에서 관리한다.

### Team Role Assignments

조직에서 custom roles가 활성화된 경우, 팀에 여러 역할을 할당할 수 있다. 팀 멤버는 자신의 사용자 역할과 팀에 할당된 모든 역할의 **합집합** 권한을 갖는다.

역할 할당 관리는 `role:update` 및 `team:update` 스코프를 가진 사용자만 수행할 수 있다(예: 조직 관리자). Team Admin은 역할 할당 수정 권한이 없어도 Entity Access Grants는 항상 직접 관리할 수 있다.

### Team Entity Access Grants

Team Entity Access Grants를 사용하면 팀 관리자가 조직 수준 역할 관리 권한 없이도 스택, 환경, Insights 계정에 대한 팀의 접근을 직접 관리할 수 있다.

**환경 권한 값:**

| 값 | 콘솔 라벨 | 설명 |
|----|-----------|------|
| `read` | Environment reader | 환경 정의 조회. 시크릿 복호화 또는 동적 자격 증명 획득 불가 |
| `open` | Environment opener | 시크릿 복호화 및 동적 자격 증명 획득 가능 |
| `write` | Environment editor | 환경 열기 및 업데이트 |
| `admin` | Environment admin | 환경 열기, 업데이트, 삭제 |

**REST API로 환경 권한 관리:**

```bash
# 환경 권한 추가
curl -s -X PATCH \
  "https://api.pulumi.com/api/orgs/{orgName}/teams/{teamName}" \
  -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "addEnvironmentPermission": {
      "projectName": "<PROJECT_NAME>",
      "envName": "<ENV_NAME>",
      "permission": "read"
    }
  }'
```

```python
# pulumi cloud api 명령으로 동일 작업
# pulumi cloud api PATCH /orgs/{orgName}/teams/{teamName}
#   --body '{"addEnvironmentPermission":{"projectName":"<PROJECT_NAME>","envName":"<ENV_NAME>","permission":"read"}}'
```

---

## RBAC (역할 기반 접근 제어)

> 원문: [RBAC](https://www.pulumi.com/docs/administration/access-identity/rbac/) | [Roles](https://www.pulumi.com/docs/administration/access-identity/rbac/roles/) | [Permission Sets](https://www.pulumi.com/docs/administration/access-identity/rbac/permission-sets/) | [Scopes](https://www.pulumi.com/docs/administration/access-identity/rbac/scopes/)

Pulumi Cloud의 RBAC는 조직 리소스에 대한 액세스를 유연하고 안전하게 관리하는 시스템이다.

### RBAC 핵심 구성 요소

| 구성 요소 | 설명 |
|-----------|------|
| **Scopes** | 가장 세분화된 접근 제어 단위. 특정 리소스에 대한 특정 작업 (예: `stack:read`, `environment:write`) |
| **Permission Sets** | 관련 Scopes의 묶음. 일반적으로 함께 사용되는 권한을 그룹화 |
| **Roles** | Permission Sets를 리소스에 적용하고 주체(Principal)에 할당하는 컬렉션 |
| **Teams** | 역할을 할당받을 수 있는 사용자 그룹 |

### 권한 누적 구조

사용자의 최종 권한은 모든 권한 부여의 **합집합**이며, 어떤 권한도 다른 권한을 줄일 수 없다:

1. **조직 전체 설정** (Settings > Access Management): ON/OFF 토글. 활성화 시 모든 멤버에게 무조건 부여
2. **팀 역할**: 소속 팀에 할당된 모든 역할의 권한 상속
3. **조직 역할**: 각 멤버의 기본 조직 역할(Admin/Member/Billing Manager/Custom)
4. **생성자 권한(Creator Grants)**: 스택을 생성한 사용자는 해당 스택에 대해 자동으로 Stack Admin 권한 부여

### 기본 역할

| 역할 | 설명 |
|------|------|
| `Admin` | 모든 조직 리소스 및 설정에 대한 전체 액세스. 멤버·역할·조직 구성 관리 가능 |
| `Member` | 조직 리소스 조회 및 스택 작업 참여. 조직 설정 수정 불가 |
| `Billing Manager` | 결제 정보 조회 및 관리. 다른 조직 설정이나 리소스 수정 불가 |

### Custom Roles (Enterprise / Business Critical)

Custom roles는 Direct Entity Access, Global Entity Access, Tag-based(ABAC) 규칙을 조합하여 생성할 수 있다.

| 규칙 유형 | 설명 |
|-----------|------|
| **Direct Entity Access** | 개별적으로 선택한 엔티티(스택, 환경, Insights 계정)에 Permission Set 부여 |
| **Global Entity Access** | 특정 유형의 **모든** 엔티티에 Permission Set 부여 (향후 생성되는 엔티티 포함) |
| **Tag-based (ABAC)** | 리소스 태그가 정의된 조건과 일치할 때 Permission Set 부여 (예: `team=platform` 태그가 있는 모든 스택) |

### 기본 Permission Sets

**스택 Permission Sets:**

| Permission Set | 설명 | 포함 Scopes |
|----------------|------|-------------|
| `Stack Read` | 읽기 전용. Preview 실행 가능 | `stack:read`, `stack:export`, `stack:encrypt`, `stack:decrypt`, `stack_deployment:read`, `stack_deployment_settings:read`, `stack_access:read`, `stack_schedule:read` |
| `Stack Write` | 스택 업데이트 및 구성 수정 | Stack Read + `stack:import`, `stack:cancel_update`, `stack:write` 등 |
| `Stack Admin` | 스택 전체 제어 | Stack Write + `stack:delete`, `stack_access:update`, `stack:transfer`, `stack:rename` |

**환경 Permission Sets:**

| Permission Set | 설명 |
|----------------|------|
| `Environment Read` | 읽기 전용 |
| `Environment Open` | 시크릿 복호화 및 동적 자격 증명 획득 |
| `Environment Write` | 환경 수정 |
| `Environment Admin` | 환경 전체 제어 |

**Insights 계정 Permission Sets:**

| Permission Set | 설명 |
|----------------|------|
| `Account Read` | 읽기 전용 |
| `Account Write` | Insights 계정 수정 |
| `Account Admin` | Insights 계정 전체 제어 |

### 주요 Scopes (조직 수준)

| Scope | 설명 | 기본 부여 역할 |
|-------|------|----------------|
| `stack:create` | 스택 생성 | 조직 전체 설정으로 제어 |
| `stack:delete` | 스택 삭제 | 조직 전체 설정으로 제어 |
| `team:create` | 팀 생성 | 조직 전체 설정으로 제어 |
| `audit_logs:read` | 감사 로그 조회 | `Admin` |
| `audit_logs:export` | 감사 로그 내보내기 | `Admin` |
| `org_member:add` | 조직 멤버 추가 | `Admin` |
| `org_member:delete` | 조직 멤버 제거 | `Admin` |
| `environment:create` | 환경 생성 | `Member`, `Admin` |
| `policy_pack:create` | 정책 팩 생성 | `Admin` |
| `oidc_issuers:create` | OIDC 발급자 등록 | `Admin` |

---

## SAML SSO

> 원문: [SAML SSO](https://www.pulumi.com/docs/administration/access-identity/saml/)

Pulumi Cloud는 SAML 2.0 기반 identity provider를 통한 Single Sign-On을 지원한다. SAML 지원은 **Enterprise** 및 **Business Critical** 에디션에서 사용할 수 있다.

SSO 조직의 멤버는 `https://app.pulumi.com/welcome/<organization-name>/sso` URL로 접속하면 조직 이름이 자동 입력된 로그인 페이지로 이동한다.

> **참고:** Pulumi는 SCIM 애플리케이션당 하나의 Pulumi 조직만 지원한다. 여러 Pulumi 조직을 관리하는 경우 각 조직마다 별도의 SCIM 애플리케이션을 구성해야 한다.

### 지원 Identity Provider 통합 가이드

| Identity Provider | 문서 |
|-------------------|------|
| Microsoft Entra ID (구 Azure AD) | [Entra ID 가이드](https://www.pulumi.com/docs/administration/access-identity/saml/entra/) |
| Google Workspace (구 G Suite) | [Google Workspace 가이드](https://www.pulumi.com/docs/administration/access-identity/saml/gsuite/) |
| Okta | [Okta 가이드](https://www.pulumi.com/docs/administration/access-identity/saml/okta/) |
| Auth0 | [Auth0 가이드](https://www.pulumi.com/docs/administration/access-identity/saml/auth0/) |
| JumpCloud | [JumpCloud 가이드](https://www.pulumi.com/docs/administration/access-identity/saml/jumpcloud/) |
| OneLogin | [OneLogin 가이드](https://www.pulumi.com/docs/administration/access-identity/saml/onelogin/) |

> **참고:** 셀프 호스팅 Pulumi Cloud를 사용하는 경우, 먼저 셀프 호스팅 인프라에 SAML SSO를 구성(API 서비스 키 및 환경 변수)한 후 이 곳에서 IdP 구성을 완료해야 한다.

---

## SCIM 프로비저닝

> 원문: [SCIM](https://www.pulumi.com/docs/administration/access-identity/scim/)

Pulumi Cloud는 SCIM(System for Cross-domain Identity Management) 2.0 통합을 지원하여, Identity Provider(IdP)에서 사용자와 그룹을 중앙 관리하고 Pulumi Cloud로 동기화할 수 있다. SCIM은 **Business Critical** 에디션에서만 사용할 수 있다.

SCIM으로 관리되는 팀 외에도 Pulumi Cloud 내에서 로컬 팀을 직접 구성하고 관리할 수 있다.

### SCIM 지원 Identity Provider

| Identity Provider | 문서 |
|-------------------|------|
| Microsoft Entra ID | [Entra ID SCIM 가이드](https://www.pulumi.com/docs/administration/access-identity/scim/entra/) |
| Okta | [Okta SCIM 가이드](https://www.pulumi.com/docs/administration/access-identity/scim/okta/) |
| OneLogin | [OneLogin SCIM 가이드](https://www.pulumi.com/docs/administration/access-identity/scim/onelogin/) |

---

## Access Tokens

> 원문: [Access Tokens](https://www.pulumi.com/docs/administration/access-identity/access-tokens/)

Access Tokens는 CLI를 통한 Pulumi Cloud 로그인 또는 REST API를 사용한 자동화에 사용된다.

### 토큰 유형

| 유형 | 설명 | 사용 가능 에디션 |
|------|------|-----------------|
| **Personal Token** | 생성한 사용자의 권한을 상속. 모든 조직 멤버십·팀 멤버십·역할 할당 포함 | 모든 에디션 |
| **Organization Token** | 조직 자체로 인증. 감사 로그에 조직 이름으로 기록. 생성 시 RBAC 역할 할당 | Enterprise, Business Critical |
| **Team Token** | 특정 팀으로 인증. 팀에 할당된 역할의 권한 상속. 감사 로그에 팀 이름으로 기록 | Enterprise, Business Critical |

### 토큰 보안 권장 사항

- 최대 2년까지 만료 기간 설정 가능. 만료된 토큰은 갱신 또는 재활성화 불가
- 토큰 만료 설정을 권장하여 토큰 순환 장려
- Organization/Team Token은 CI/CD 파이프라인 등 비대화형 자동화에만 사용
- Organization Token에 custom role을 할당하여 최소 권한 원칙 적용

### Personal Token 생성

1. 사용자 메뉴에서 **Personal access tokens** 선택
2. **Create token** 클릭
3. 설명 입력(선택) 및 만료 기간 선택
4. **Create token** 클릭

### Organization Token

Organization Token은 조직 자체로 인증하는 머신 토큰이다. CI/CD 파이프라인, 드리프트 감지, 정책 강제, 조직 수준 보고서 등에 적합하다.

- 조직 관리자가 **Settings > Access Tokens**에서 생성/조회/삭제
- 토큰은 생성한 관리자 개인이 소유하지 않음. 관리자가 조직을 떠나도 다른 관리자가 접근 가능
- 토큰 이름은 삭제 후에도 영구 예약되어 감사 로그 무결성 보존
- 삭제 시 즉시 접근 권한 회수

### OIDC 발급 토큰

CI/CD 워크플로우(GitHub Actions, GitLab CI, Bitbucket Pipelines 등)에서 OIDC(OpenID Connect)를 통해 Pulumi Cloud에 인증할 수 있다. OIDC 인증은 장수명 시크릿(access token)을 저장하지 않고도 임시 자격 증명으로 Pulumi Cloud에 접근할 수 있는 보안 메커니즘이다.

**OIDC 토큰 교환 흐름:**

1. CI/CD 공급자가 OIDC 토큰(JWT)을 발급
2. Pulumi Cloud가 사전에 등록된 OIDC Issuer 구성의 authorization policy와 토큰의 subject claim을 매칭
3. 매칭 성공 시 Pulumi access token으로 교환

**OIDC 토큰 타입:**

| 토큰 타입 | 설명 | 사용 가능 에디션 |
|-----------|------|-----------------|
| **Personal** | 특정 사용자의 권한으로 인증 | 모든 에디션 |
| **Organization** | 조직 자체로 인증, RBAC 역할 할당 가능 | Enterprise, Business Critical |
| **Team** | 팀 권한으로 인증 | Enterprise, Business Critical |
| **Deployment Runner** | 배포 실행을 위한 전용 토큰 | Business Critical |

권한은 `scope: admin` 같은 필드로 요청하는 것이 아니라, authorization policy의 subject claim 매칭과 토큰 타입에 할당된 RBAC 역할로 결정된다. 자세한 구성 방법은 아래 OIDC Issuers 섹션을 참조하라.

---

## 스택 권한(Stack Permissions)

> 원문: [Stack Permissions](https://www.pulumi.com/docs/administration/access-identity/stack-permissions/)

스택 권한은 조직 내 개별 스택에 대한 접근을 세밀하게 제어하는 메커니즘이다. RBAC의 조직 수준 역할과는 별개로, 스택 단위로 읽기/쓰기/관리 권한을 개별 사용자나 팀에 부여할 수 있다.

### 스택 권한 수준

| 권한 | 설명 |
|------|------|
| **Read** | 스택 상태 및 리소스 조회 |
| **Write** | 스택 업데이트, 설정 변경 |
| **Admin** | 스택 삭제, 권한 관리, 이름 변경, 이전 |

### 스택 권한 부여 방식

1. **생성자 자동 권한(Creator Grants)**: 스택을 생성한 사용자는 해당 스택에 자동으로 Stack Admin 권한이 부여된다. 이는 Settings > Access Management에서 비활성화할 수 있다.
2. **팀 기반 권한**: 팀에 스택 권한을 할당하면 팀 멤버 전원이 해당 권한을 상속받는다.
3. **RBAC 역할을 통한 권한**: Custom Role의 Direct Entity Access, Global Entity Access, Tag-based 규칙으로 스택 권한을 부여할 수 있다.

### 스택 권한 관리

```bash
# 스택 권한 조회
pulumi stack permission list

# 멤버에게 스택 권한 부여
pulumi stack permission set <member-name> <permission-level>
```

REST API로도 스택 권한을 관리할 수 있다:

```bash
# REST API로 스택 권한 업데이트
curl -s -X PATCH \
  "https://api.pulumi.com/api/stacks/{org}/{project}/{stack}/permissions" \
  -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "updates": [
      {"member": "<user-or-team>", "permission": "<read|write|admin>"}
    ]
  }'
```

---

## OIDC Issuers

> 원문: [OIDC Issuers](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/)

OIDC Issuers는 CI/CD 파이프라인에서 장수명 시크릿 없이 Pulumi Cloud에 인증하기 위한 OpenID Connect 발급자를 등록하고 관리하는 기능이다. OIDC Issuer를 등록하면 GitHub Actions, GitLab CI, Bitbucket Pipelines 등의 워크플로우가 임시 자격 증명으로 Pulumi Cloud에 안전하게 접근할 수 있다.

### OIDC Issuer 등록

조직 관리자는 **Settings > Access Management > OIDC Issuers**에서 발급자를 등록할 수 있다. 등록 시 authorization policy를 정의하여 어떤 워크플로우 조건에서 토큰 교환을 허용할지 제어한다.

**지원 OIDC 공급자:**

| 공급자 | 발급자 URL | 설명 |
|--------|-----------|------|
| GitHub Actions | `https://token.actions.githubusercontent.com` | GitHub OIDC 공급자 |
| GitLab CI | `https://gitlab.com` | GitLab OIDC 공급자 |
| Bitbucket Pipelines | `https://api.bitbucket.org/2.0/workspaces/{workspace}/pipelines-config/identity/oidc` | Bitbucket OIDC 공급자 |
| Custom | 사용자 정의 | 기타 OIDC 호환 공급자 |

### 에디션별 토큰 타입

| 토큰 타입 | 설명 | 사용 가능 에디션 |
|-----------|------|-----------------|
| **Personal** | 특정 사용자 권한으로 인증 | 모든 에디션 |
| **Organization** | 조직 자체로 인증, RBAC 역할 할당 | Enterprise, Business Critical |
| **Team** | 팀 권한으로 인증 | Enterprise, Business Critical |
| **Deployment Runner** | 배포 실행 전용 | Business Critical |

### Authorization Policy

OIDC Issuer 등록 시 authorization policy를 설정한다. policy는 토큰의 subject claim과 사전에 정의된 패턴을 매칭하여 토큰 교환을 허용 또는 거부한다.

**GitHub Actions 예시:**

```yaml
# .github/workflows/pulumi.yml
jobs:
  pulumi:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # OIDC 토큰 발급에 필요
      contents: read
    steps:
      - uses: actions/checkout@v4
      - name: Pulumi 로그인 (OIDC)
        uses: pulumi/action-install-pulumi-cli@v2
      - run: pulumi login --cloud-url https://api.pulumi.com
        env:
          PULUMI_ACCESS_TOKEN: ${{ steps.pulumi-oidc.outputs.PULUMI_ACCESS_TOKEN }}
```

subject claim 매칭 예시:
- `repo:my-org/my-repo:*` — 특정 리포지토리의 모든 워크플로우 허용
- `repo:my-org/my-repo:ref:refs/heads/main` — main 브랜치에서만 허용
- `repo:my-org/my-repo:environment:production` — production 환경에서만 허용

### 썸프린트(Thumbprint)

OIDC Issuer 등록 시 공급자의 인증서 썸프린트를 제공해야 할 수 있다. 썸프린트는 OIDC 공급자의 TLS 인증서 지문(SHA-1)으로, 토큰의 출처를 검증하는 데 사용된다.

### CLI 및 REST API를 통한 토큰 교환

**CLI를 통한 OIDC 인증:**

```bash
# Pulumi CLI에서 OIDC 인증 사용
pulumi login --oidc <issuer-url>
```

**REST API를 통한 토큰 교환:**

```bash
# OIDC 토큰을 Pulumi access token으로 교환
curl -s -X POST \
  "https://api.pulumi.com/api/orgs/{orgName}/oidc/token" \
  -H "Content-Type: application/json" \
  -d '{
    "accessToken": "<oidc-jwt-token>",
    "issuer": "<issuer-url>"
  }'
```

---

## 감사 로그(Audit Logs)

> 원문: [RBAC Scopes - Organization Settings](https://www.pulumi.com/docs/administration/access-identity/rbac/scopes/org-settings/)

감사 로그는 조직 내 모든 활동에 대한 가시성을 제공한다. 감사 로그에 대한 접근은 RBAC Scopes로 제어된다.

| Scope | 설명 | 기본 부여 역할 |
|-------|------|----------------|
| `audit_logs:read` | 조직 활동의 감사 로그 조회. 시스템 이벤트 및 사용자 작업에 대한 가시성 제공 | `Admin` |
| `audit_logs:export` | 컴플라이언스 및 분석 목적으로 감사 로그 데이터 내보내기 | `Admin` |

Organization Token 및 Team Token으로 수행된 작업은 감사 로그에 해당 토큰의 이름으로 기록되어, 개인 사용자 노출 없이 작업 추적이 가능하다.

---

## 셀프 호스팅(Self-Hosted)

> 원문: [Self-Hosting](https://www.pulumi.com/docs/administration/self-hosting/)

Pulumi **Business Critical** 에디션은 조직 인프라 내에서 Pulumi Cloud를 셀프 호스팅할 수 있는 옵션을 제공한다. 데이터, 보안, 운영에 대한 완전한 제어가 가능하다.

### 배포 옵션

| 옵션 | 설명 |
|------|------|
| [Quickstart Docker Compose](https://www.pulumi.com/docs/administration/self-hosting/deployment-options/quickstart-docker-compose/) | 로컬 환경 테스트용 Docker Compose 배포 |
| [ECS-Hosted](https://www.pulumi.com/docs/administration/self-hosting/deployment-options/ecs-hosted/) | AWS ECS 프로덕션 배포 (TypeScript / Go 자동화) |
| [EKS-Hosted](https://www.pulumi.com/docs/administration/self-hosting/deployment-options/eks-hosted/) | Amazon EKS 프로덕션 배포 (TypeScript 자동화) |
| [AKS-Hosted](https://www.pulumi.com/docs/administration/self-hosting/deployment-options/aks-hosted/) | Azure Kubernetes Service 프로덕션 배포 (TypeScript 자동화) |
| [GKE-Hosted](https://www.pulumi.com/docs/administration/self-hosting/deployment-options/gke-hosted/) | Google Kubernetes Engine 프로덕션 배포 (TypeScript 자동화) |
| [BYO Infrastructure](https://www.pulumi.com/docs/administration/self-hosting/deployment-options/byo-infra-hosted/) | 자체 Kubernetes, MySQL, S3 호환 스토리지에 배포 |
| [Local-Docker](https://www.pulumi.com/docs/administration/self-hosting/deployment-options/local-docker/) | 커스텀 Docker 환경에서 MySQL 및 오브젝트 스토리지와 함께 프로덕션 배포 |

### 구성 요소

| 항목 | 설명 |
|------|------|
| [Components](https://www.pulumi.com/docs/administration/self-hosting/components/) | Pulumi Cloud 프론트엔드 UI 및 백엔드 API Docker 이미지 |
| [Network Requirements](https://www.pulumi.com/docs/administration/self-hosting/network/) | 인그레스, 이그레스, 인프라 요구 사항 |

### 운영 가이드

| 항목 | 설명 |
|------|------|
| [Operations Guide](https://www.pulumi.com/docs/administration/self-hosting/operations/) | HA, DR, 모니터링, 사이징, 보안 강화 |
| [Backup and Recovery](https://www.pulumi.com/docs/administration/self-hosting/operations/backup-recovery/) | 백업 전략, 복구 절차, RTO 목표 |
| [Monitoring and Alerting](https://www.pulumi.com/docs/administration/self-hosting/operations/monitoring/) | 3단계 알림 전략 및 주요 메트릭 |
| [Security Hardening](https://www.pulumi.com/docs/administration/self-hosting/operations/security-hardening/) | 네트워크 보안, 암호화, SMTP, 봇 보호 |

---

## 조직 설정 항목 총정리

| 설정 항목 | 경로 | 설명 |
|-----------|------|------|
| 대시보드 | 조직 홈 | 최근 활동, 리소스 수, 스택 개요 |
| 스택 관리 | All Stacks | 검색, 프로젝트별 그룹화, 태그 필터링, 스택 이동/복원 |
| 멤버 관리 | Settings > Members | 멤버 초대(이메일/링크), 초대 상태 모니터링, 역할 변경 |
| 팀 관리 | Settings > Teams | 팀 생성, 멤버 할당, 역할 할당, Entity Access Grants |
| 역할 관리 | Settings > Roles | 기본 역할 조회, Custom Role 생성/수정/삭제, 기본 역할 설정 |
| Permission Sets | Settings > Access Management > Permission Sets | 기본 Permission Sets 조회, Custom Permission Set 생성 |
| Access Management | Settings > Access Management | 조직 전체 ON/OFF 토글(스택 생성/삭제, 팀 생성 등), Identity Provider 변경, Custom Roles 활성화 |
| 스택 권한 | 스택별 Settings / REST API | 개별 스택에 대한 읽기/쓰기/관리 권한 부여 (생성자 자동 권한, 팀 기반, RBAC 역할) |
| Access Tokens | Settings > Access Tokens (Org) / 사용자 메뉴 (Personal) | Personal / Organization / Team Token 생성 및 관리 |
| 결제 및 사용량 | Settings > Billing & Usage | 결제 수단 업데이트, 사용량 확인 |
| SAML SSO | Settings > Access Management > SAML | SAML 2.0 IdP 구성 (Enterprise / Business Critical) |
| SCIM | Identity Provider 측 설정 | 사용자/그룹 프로비저닝 동기화 (Business Critical) |
| OIDC Issuers | Settings > Access Management > OIDC Issuers | OpenID Connect 인증 구성 |
| 정책 | Policies | 정책 팩, 정책 그룹 관리, 컴플라이언스 가드레일 |
| 감사 로그 | Audit Logs | 조직 활동 로그 조회 및 내보내기 (Admin 전용) |
| 조직 삭제 | Settings > Delete Organization | 조직 영구 삭제 (Admin 전용). 사전에 스택 이전 필요 |

---

## 프로그래밍 방식으로 조직 이름 가져오기

배포 중인 조직 이름을 런타임에 읽어 리소스 이름 지정이나 태깅, 다른 스택 참조 구성에 활용할 수 있다.

```typescript
const organization = pulumi.getOrganization();
```

```python
organization = pulumi.get_organization()
```
