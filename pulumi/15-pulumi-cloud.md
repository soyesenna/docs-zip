# Pulumi Cloud 관리

> 원문: [Pulumi Cloud](https://www.pulumi.com/docs/iac/concepts/pulumi-cloud/) | [Pulumi Cloud vs. OSS](https://www.pulumi.com/docs/iac/guides/basics/pulumi-cloud-vs-oss/) | [Administration](https://www.pulumi.com/docs/administration/) | [Organizations](https://www.pulumi.com/docs/administration/organizations-teams/organizations/) | [RBAC](https://www.pulumi.com/docs/administration/access-identity/rbac/) | [Roles](https://www.pulumi.com/docs/administration/access-identity/rbac/roles/) | [Teams](https://www.pulumi.com/docs/administration/access-identity/rbac/teams/) | [Permission Sets](https://www.pulumi.com/docs/administration/access-identity/rbac/permission-sets/) | [Scopes](https://www.pulumi.com/docs/administration/access-identity/rbac/scopes/) | [Access Tokens](https://www.pulumi.com/docs/administration/access-identity/access-tokens/) | [OIDC Issuers](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/) | [OIDC EKS](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/kubernetes-eks/) | [OIDC GKE](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/kubernetes-gke/) | [Audit Logs](https://www.pulumi.com/docs/administration/security-compliance/audit-logs/) | [Customer Managed Keys](https://www.pulumi.com/docs/administration/security-compliance/customer-managed-keys/) | [Self-Hosting](https://www.pulumi.com/docs/administration/self-hosting/) | [Setting Up for Success](https://www.pulumi.com/docs/administration/onboarding-guide/) | [Infrastructure AI](https://www.pulumi.com/docs/ai/) | [Tasks](https://www.pulumi.com/docs/ai/tasks/) | [Previews](https://www.pulumi.com/docs/ai/running-previews/) | [Automations](https://www.pulumi.com/docs/ai/automations/) | [`pulumi neo` CLI](https://www.pulumi.com/docs/iac/cli/commands/pulumi_neo/)

Pulumi Cloud는 Pulumi CLI의 기본 상태 백엔드이자, 팀이 대규모로 Pulumi를 운영하는 데 필요한 기능을 제공하는 관리형 플랫폼이다. 액세스 제어, 재사용 가능한 구성 및 시크릿, 정책 강제, 클라우드 리소스 인벤토리, 예약 드리프트 감지, 관리형 배포, **Pulumi Neo**(AI 에이전트), **Ephemeral Environments**(Review Stacks, TTL Stacks) 등의 기능을 제공한다. Pulumi Cloud는 호스팅 SaaS와 셀프 호스팅 에디션으로 제공되며, 개인(Individual) 티어는 무료다. Pulumi Cloud와 오픈소스 Pulumi의 기능별 상세 비교는 [Pulumi Cloud vs. OSS](https://www.pulumi.com/docs/iac/guides/basics/pulumi-cloud-vs-oss/)를 참조하라.

### Pulumi Cloud 주요 기능

| 기능 | 설명 |
|------|------|
| [RBAC](https://www.pulumi.com/docs/administration/organizations-teams/teams/) | SAML/SSO 통합 및 세분화된 액세스 토큰을 통한 역할 기반 접근 제어 |
| [Pulumi ESC](https://www.pulumi.com/docs/esc/) | 재사용 가능한 구성 및 시크릿 관리. 환경을 한 번 정의하여 여러 스택에서 사용 |
| [Policy as Code](https://www.pulumi.com/docs/insights/policy/) | 모든 업데이트에 중앙 집중식 정책 강제. 보안, 컴플라이언스, 비용 규칙용 사전 구축 정책 팩 제공 |
| [Cloud Resource Inventory](https://www.pulumi.com/docs/insights/) | 클라우드 계정 전체의 리소스 검색. Pulumi 관리 대상이 아닌 리소스 포함 |
| [Drift Detection](https://www.pulumi.com/docs/deployments/deployments/drift/) | 예약 드리프트 감지. 배포된 인프라가 선언 상태와 다를 경우 알림 또는 자동 수정 |
| [Managed Deployments](https://www.pulumi.com/docs/deployments/deployments/) | Git push 등에 응답하여 Pulumi 작업을 원격 실행. 웹훅을 통한 이벤트 기반 워크플로우 |
| [Pulumi Neo](https://www.pulumi.com/docs/ai/) | AI 에이전트. 배포 디버깅, IaC 작성, 환경 질문 응답 지원 |
| [Ephemeral Environments](https://www.pulumi.com/docs/deployments/deployments/review-stacks/) | Review Stacks(PR 기반 단기 환경) 및 TTL Stacks(수명 제한 스택) |

### Pulumi Cloud vs. OSS 비교

> 원문: [Pulumi Cloud vs. OSS](https://www.pulumi.com/docs/iac/guides/basics/pulumi-cloud-vs-oss/)

| 기능 | 오픈소스 Pulumi | Pulumi Cloud |
|------|----------------|--------------|
| 상태 백엔드 | DIY 오브젝트 스토리지, PostgreSQL, 로컬 파일시스템 | 관리형 트랜잭션 상태 백엔드. Terraform 상태도 저장 가능 |
| 배포 기록 | 백엔드별 체크포인트 기록 | 조직 전체 배포 기록 |
| 액세스 제어 | 직접 관리 (예: 클라우드 IAM) | 빌트인 RBAC, SAML/SSO 통합 |
| 시크릿 암호화 | 패스프레이즈 또는 자체 관리 KMS | 기본 관리 암호화. 별도 암호화 서비스 사용도 가능 |
| 시크릿/구성 관리 | 스택별 구성 파일만 | 스택별 구성 + 중앙 관리 재사용 가능 Pulumi ESC 환경 |
| 정책 코드화 | 디스크의 정책 팩을 CLI 인수로 전달 | 중앙 관리 강제 적용 + 사전 구축 및 커스텀 정책 팩 |
| 클라우드 리소스 인벤토리 | 미포함 | Pulumi Insights가 Pulumi 관리 외 리소스도 탐색 |
| 드리프트 감지 | `pulumi refresh` 수동 실행 | 예약 드리프트 감지 및 자동 수정 |
| AI 지원 | Pulumi CLI 및 편집기 통합 | Pulumi Neo AI 에이전트가 플랫폼 전반에 통합 |
| 임시 환경 | 미포함 | Review Stacks 및 TTL Stacks |
| 관리형 배포 | 자체 자동화 구성 | Pulumi Deployments 관리 서비스 |
| REST API 및 웹훅 | 미포함 | 문서화된 REST API 및 웹훅 |
| 컴플라이언스 | 해당 없음 | 연간 SOC 2 Type II 감사; 내보내기 가능한 감사 추적 |
| 지원 | 커뮤니티 지원; 상업 지원 가능 | 커뮤니티 지원; 엔터프라이즈 고객용 지원 플랜 |

---

## 조직(Organizations)

> 원문: [Organizations](https://www.pulumi.com/docs/administration/organizations-teams/organizations/)

조직은 Pulumi Cloud의 최상위 계정 단위로, 관련 프로젝트·스택·사용자를 그룹화하는 협업 공간이다. 조직은 팀, RBAC, 결제, 공유 Pulumi ESC 환경이 모두 속하는 기본 단위다. 스택의 정규화된 이름은 `<organization>//<stack>` 형식을 따른다. 조직 이름은 스택 정규화 이름의 첫 번째 세그먼트다. Pulumi Cloud에 가입하면 사용자 이름과 동일한 이름의 **개인 조직**(personal organization)이 자동으로 생성된다. 추가로 팀용 조직을 생성하거나 기존 조직에 초대받을 수 있으며, 여러 조직의 멤버로 동시에 속할 수 있다. 조직 간 전환은 Pulumi Cloud 콘솔 상단 탐색 메뉴의 조직 메뉴 또는 `pulumi org` CLI 명령으로 수행한다.

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

> **Organization default role:** 조직에서 custom roles가 활성화된 경우, Member 조직 역할을 가진 모든 멤버에게 적용할 **조직 기본 역할(organization default role)**을 설정할 수 있다. 기본 역할은 **Settings > Roles**에서 custom role을 열고 **Set as default role**을 선택하여 설정한다. 명시적으로 custom role이 할당되지 않은 Member 사용자는 이 기본 역할의 권한을 상속받는다.

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

### Identity Provider 변경 절차

1. **Settings > Access Management**로 이동
2. **Other** 탭 선택
3. **Membership Requirements** 섹션에서 **Change requirements** 선택
4. 새 identity provider 선택

### Identity Provider 연결 해제(Disconnect)

Identity provider를 연결 해제하려면 **다른 identity provider를 먼저 선택**해야 한다. SAML SSO 설정을 제거하려는 경우도 마찬가지로 새 identity provider를 선택해야 한다. 조직 멤버는 조직 identity provider를 변경하기 전에 먼저 새 identity provider를 개인 계정에 추가해야 하며, 그렇지 않으면 조직 접근이 차단된다.

1. **Settings > Access Management**로 이동
2. **Other** 탭 선택
3. **Membership Requirements** 섹션에서 **Change requirements** 선택
4. 새 identity provider 선택

### 스택 이전(Transferring Stacks)

스택 관리자는 개인 계정과 조직 간, 또는 조직 간에 개별 스택을 이전할 수 있다. 조직 관리자는 스택을 일괄 이전할 수 있다. 스택 이전에는 두 가지 권한이 필요하다: 현재 소유자로부터 스택을 이전할 권한과 대상 조직에서 스택을 생성할 권한. 둘 모두 조직의 액세스 제어를 통해 구성된다.

**개별 스택 이전:**

1. 스택으로 이동 후 **Settings** 선택
2. **Transfer stack** 선택
3. 대상 개인 계정 또는 조직 이름 입력 후 **Transfer** 선택

**일괄 스택 이전 (조직 관리자):**

1. **Stacks** 페이지로 이동
2. **Create project** 옆 점 세 개 메뉴 선택
3. 드롭다운에서 **Transfer stacks** 선택
4. 드롭다운에서 **Transfer destination** 선택
5. 이전할 스택 선택 (최대 15개) 후 **Transfer stacks** 선택

### 삭제된 스택 복원(Restoring Deleted Stacks)

삭제된 스택 복원은 **Pulumi Cloud** 기능이다. DIY 백엔드를 사용하는 경우 이 방식으로 복원할 수 없다. 복원하면 이전에 삭제된 스택과 해당 업데이트 기록이 함께 복구된다. 조직 내에서 가장 최근에 삭제된 25개 스택을 조직 관리자가 복원할 수 있다.

1. **Stacks** 페이지로 이동
2. **Create project** 옆 점 세 개 메뉴 선택
3. 드롭다운에서 **Restore deleted stacks** 선택
4. 복원할 스택의 점 세 개 메뉴에서 **Restore stack** 선택

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

> **Role-backed teams:** 팀을 생성하고 custom role을 할당한 뒤(예: 특정 스택 또는 [tag-based 규칙](https://www.pulumi.com/docs/administration/access-identity/rbac/roles/#tag-based-abac-rules)으로 접근 제한) 멤버를 추가하면, 해당 멤버는 자신의 사용자 역할에 더해 팀의 역할 권한을 추가로 획득한다. 이를 통해 역할 기반으로 팀 멤버십을 구성할 수 있다.

역할 할당 관리는 팀의 **Access** 탭에서 수행한다. **Role assignments** 섹션에서 현재 할당된 역할을 확인하고 **Add role**로 추가 역할을 할당할 수 있다.

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

세 가지 작업(추가, 수정, 제거) 모두 동일한 `PATCH` 엔드포인트를 사용한다:

```bash
# 환경 권한 추가
curl -s -X PATCH \
  "https://api.pulumi.com/api/orgs/{orgName}/teams/{teamName}" \
  -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "addEnvironmentPermission": {
      "projectName": "{projectName}",
      "envName": "{envName}",
      "permission": "read"
    }
  }'
```

```bash
# 기존 환경 권한 수정 (editEnvironmentPermission)
curl -s -X PATCH \
  "https://api.pulumi.com/api/orgs/{orgName}/teams/{teamName}" \
  -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "editEnvironmentPermission": {
      "projectName": "{projectName}",
      "envName": "{envName}",
      "permission": "write"
    }
  }'
```

```bash
# 환경 권한 제거 (removeEnvironment)
curl -s -X PATCH \
  "https://api.pulumi.com/api/orgs/{orgName}/teams/{teamName}" \
  -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "removeEnvironment": {
      "projectName": "{projectName}",
      "envName": "{envName}"
    }
  }'
```

성공 시 `204 No Content`를 반환한다. `projectName` 필드는 환경이 속한 ESC 프로젝트를 가리킨다 (예: 명시적 프로젝트를 지정하지 않은 경우 `default`).

```python
# pulumi cloud api 명령으로 동일 작업 (권한 추가 예시)
# pulumi cloud api PATCH /orgs/{orgName}/teams/{teamName}
#   --body '{"addEnvironmentPermission":{"projectName":"{projectName}","envName":"{envName}","permission":"read"}}'
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

### Principals (주체)

역할은 Pulumi Cloud의 세 가지 종류의 주체(Principal)에 할당할 수 있다:

| Principal | 설명 |
|-----------|------|
| **Users** | 조직 멤버는 기본 조직 역할(Admin, Member, Billing Manager, custom role)을 가진다. 조직에서 custom roles가 활성화되고 "Assign custom roles to users"가 켜진 경우, 관리자가 개별 멤버에게 custom role을 할당할 수 있다 |
| **Teams** | 팀에 하나 이상의 역할을 할당할 수 있다. 팀 멤버는 팀의 역할과 자신의 사용자 역할의 합집합 권한을 받는다. Custom roles 활성화 시 사용 가능 |
| **Organization access tokens** | 머신 토큰에 조직 전체 권한을 정의하는 하나의 역할을 할당할 수 있다 |

### Where roles apply (역할 적용 위치)

| 적용 위치 | 설명 |
|-----------|------|
| **User role** | 각 멤버는 하나의 기본 조직 역할(Admin, Member, Billing Manager, custom role)을 가진다. "Assign custom roles to users"가 활성화된 경우 관리자가 사용자별로 재정의 가능 |
| **Organization default role** | 조직에서 custom roles가 활성화된 경우, Member 조직 역할을 가진 멤버에게 적용할 기본 역할을 설정할 수 있다. 명시적으로 custom role이 할당되지 않은 Member 사용자가 이 기본 역할을 상속받는다 |
| **Team roles** | 팀에 여러 역할을 할당할 수 있다. 팀 멤버는 해당 역할들과 자신의 사용자 역할의 합집합 권한을 획득한다 |

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

#### Custom Role 생성 절차

Custom role 생성은 조직 관리자만 수행할 수 있다.

1. **Settings > Roles** 페이지 방문
2. **Create custom role** 클릭
3. 고유한 이름과 (선택) 설명 입력
4. Entity access 규칙(Direct/Global/Tag-based)과 조직 액세스 수준을 조합하여 구성
5. **Create role** 클릭

개별 사용자에게 custom role을 할당하려면 **Settings > Access Management**에서 **Assign custom roles to users** 설정이 활성화되어야 한다.

#### Tag-based 규칙(ABAC) 상세

Tag-based 규칙은 다음 요소로 구성된다:

- **Entity type** — Stack, Environment, Insights account
- **Tag conditions** — 리소스 태그에 대한 하나 이상의 조건 (예: 태그 `env` equals `production`, 태그 `team` exists)
- **Permission set** — 조건이 일치할 때 부여할 Permission set

접근 평가 시 Pulumi Cloud는 사용자의 역할(및 소속 팀의 역할)을 확인하고, 각 tag 규칙에 대해 리소스의 태그를 조건과 비교한다. 일치하면 해당 Permission set이 리소스에 적용된다. 대규모 조직에서 리소스를 개별 나열하지 않고 태그로 일괄 접근 권한을 부여할 때 유용하다.

#### Role Assignment (역할 할당)

역할은 조직 액세스 토큰, 사용자, 팀에 할당할 수 있다. 유효 권한은 사용자의 조직 역할과 소속 팀에 할당된 모든 역할의 **합집합**이다.

| Principal | 할당 방식 |
|-----------|-----------|
| **Organization access tokens** | 생성 시 하나의 역할(default 또는 custom)을 할당 |
| **Users** | "Assign custom roles to users"가 활성화된 경우 관리자가 개별 멤버에게 custom role 할당 가능 |
| **Teams** | 여러 역할 할당 가능. 팀 멤버는 자신의 사용자 역할 + 팀 역할의 합집합 권한 획득 |

### 기본 Permission Sets

**Permission Sets의 Entity Types:**

Permission Set은 특정 Entity type에 속해야 하며, 동일한 Entity type의 Scope만 포함할 수 있다.

| Entity type | 설명 | 포함 범위 |
|-------------|------|-----------|
| **Stacks** | 스택 관련 모든 작업 | 스택 업데이트, 구성, 배포 설정, 태그/어노테이션, 웹훅, 스케줄 |
| **Environments** | 환경 관련 모든 작업 | 환경 구성, 시크릿, 스케줄, 웹훅, 버전 |
| **Insights accounts** | Insights 계정 관련 작업 | 계정, 정책 평가, 스캔 구성, 결과/리포트 |
| **Organization settings** | 조직 수준 작업 | 조직 설정, 멤버 관리, 결제/사용량, 감사 로그, 통합 구성 |

> **Organization settings Permission Sets vs. 조직 전체 토글:** Organization settings Entity type은 역할을 통해 부여하는 RBAC scope(예: `stack:create`, `team:create`)를 다룬다. 이는 **Settings > Access Management**의 조직 전체 ON/OFF 토글(예: "Members can create stacks")과 별개다. 조직 전체 설정은 활성화 시 모든 멤버에게 무조건 부여되며 역할과 무관하다.

**스택 Permission Sets:**

| Permission Set | 설명 | 포함 Scopes |
|----------------|------|-------------|
| `Stack Read` | 읽기 전용. Preview 실행 가능 | `stack:read`, `stack:export`, `stack:encrypt`, `stack:decrypt`, `stack_deployment:read`, `stack_deployment_settings:read`, `stack_access:read`, `stack_schedule:read` |
| `Stack Write` | 스택 업데이트 및 구성 수정 | Stack Read + `stack:import`, `stack:cancel_update`, `stack:write` 등 |
| `Stack Admin` | 스택 전체 제어 | Stack Write + `stack:delete`, `stack_access:update`, `stack:transfer`, `stack:rename` |

**환경 Permission Sets:**

| Permission Set | 설명 | 포함 Scopes |
|----------------|------|-------------|
| `Environment Read` | 읽기 전용 | `environment:read`, `environment:rotate_history`, `environment_version:read`, `environment_schedule:read`, `environment_tag:read` |
| `Environment Open` | 시크릿 복호화 및 동적 자격 증명 획득 | Environment Read + `environment:open`, `environment:clone`, `environment_version:open`, `environment_version:read` |
| `Environment Write` | 환경 수정 | Environment Open + `environment:write`, `environment:rotate`, `environment_version:create/update/delete/retract`, `environment_tag:create/update/delete`, `environment_schedule:create/update/pause/resume/delete`, `environment_webhook:read/create/update/delete`, `change_gate:create/update/delete` |
| `Environment Admin` | 환경 전체 제어 | Environment Write + `environment:delete` |

**Insights 계정 Permission Sets:**

| Permission Set | 설명 | 포함 Scopes |
|----------------|------|-------------|
| `Account Read` | 읽기 전용 | `insights_account:read`, `insights_account_scan:read`, `insights_account_access:read` |
| `Account Write` | Insights 계정 수정 | Account Read + `insights_account:update`, `insights_account:scan`, `insights_account_scan:update/cancel/pause/resume` |
| `Account Admin` | Insights 계정 전체 제어 | Account Write + `insights_account:delete`, `insights_account_access:update` |

### Custom Permission Sets (Enterprise / Business Critical)

Custom Permission Sets는 조직 관리자만 생성할 수 있다.

**생성 절차:**

1. **Settings > Access Management**로 이동 후 **Permission Sets** 탭 선택
2. 해당 Entity type 그룹 내에서 **Create custom permission set** 클릭
3. 고유한 이름 입력 (권장: 설명도 함께 입력)
4. 포함할 Scopes 선택
5. **Create permission set** 클릭
6. 생성된 Custom Permission Set를 조직 내 역할에 할당 가능

### 주요 Scopes (조직 수준)

**스택 관련 Scopes:**

| Scope | 설명 | 기본 부여 Permission Set |
|-------|------|--------------------------|
| `stack:read` | 스택 구성 및 설정 조회 | `Stack Read` |
| `stack:write` | 스택 구성 및 설정 수정 | `Stack Write` |
| `stack:delete` | 스택 삭제 | `Stack Admin` |
| `stack:import` | 스택으로 리소스 가져오기 | `Stack Write` |
| `stack:export` | 스택 데이터 내보내기 | `Stack Read` |
| `stack:rename` | 스택 이름 변경 | `Stack Admin` |
| `stack:transfer` | 스택 소유권 이전 | `Stack Admin` |
| `stack:cancel_update` | 진행 중인 스택 업데이트 취소 | `Stack Write` |
| `stack:encrypt` / `stack:decrypt` | 스택 데이터 암호화/복호화 | `Stack Read` |
| `stack_deployment:create` | 스택 배포 생성 | `Stack Write` |
| `stack_deployment_settings:read/write` | 배포 설정 조회/수정 | `Stack Read` / `Stack Write` |
| `stack_schedule:create/update/delete` | 배포 스케줄 관리 | `Stack Write` |
| `stack_tags:update` | 스택 태그 수정 | `Stack Write` |
| `stack_webhook:create/update/delete` | 스택 웹훅 관리 | `Stack Write` |

**조직 설정 Scopes:**

| Scope | 설명 | 기본 부여 역할 |
|-------|------|----------------|
| `stack:create` | 스택 생성 | 조직 전체 설정으로 제어 |
| `team:create` / `team:delete` / `team:update` | 팀 생성/삭제/수정 | `Admin` |
| `environment:create` | 환경 생성 | `Member`, `Admin` |
| `environment:restore_deleted` | 삭제된 환경 복원 | `Admin` |
| `insights_account:create` | Insights 계정 생성 | `Admin` |
| `audit_logs:read` | 감사 로그 조회 | `Admin` |
| `audit_logs:export` | 감사 로그 내보내기 | `Admin` |
| `org_member:add/delete/update` | 조직 멤버 관리 | `Admin` |
| `org_member:set_admin` | 관리자 권한 부여/회수 | `Admin` |
| `invites:create/read` | 조직 초대 관리 | `Admin` |
| `policy_pack:create/delete/update` | 정책 팩 관리 | `Admin` |
| `policy_groups:create/delete/update` | 정책 그룹 관리 | `Admin` |
| `oidc_issuers:create/delete/update` | OIDC 발급자 관리 | `Admin` |
| `deployments:pause/resume` | 조직 전체 배포 일시정지/재개 | `Admin` |
| `deployments:read_usage` | 배포 사용량 조회 | `Member`, `Admin`, `Billing Manager` |
| `agent_pool:create/delete/update` | 에이전트 풀 관리 | `Admin` |
| `change_gate:create/update/delete` | 변경 승인 규칙(Change Gates) 관리 | `Admin` |

### Scopes vs. 조직 전체 설정

Scopes는 **Settings > Access Management**에 있는 조직 전체 설정(예: "Members can create stacks", "Members can delete stacks", "Members can create teams")과 별개의 시스템이다. 조직 전체 설정이 **활성화**되면 해당 기능이 모든 멤버에게 무조건 부여되며 역할과 무관하다. **비활성화**되면 해당 Scope가 역할에 명시적으로 포함된 멤버만 해당 기능을 사용할 수 있다. Scopes는 `object:action` 명명 패턴을 따르며 (예: `stack:read`, `environment:write`, `team:create`), 항상 특정 Entity type과 연관되어 동일 Entity type의 Permission Set 내에서만 사용할 수 있다.

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

> **참고:** 셀프 호스팅 Pulumi Cloud를 사용하는 경우, 먼저 [셀프 호스팅 인프라에 SAML SSO를 구성](https://www.pulumi.com/docs/administration/self-hosting/saml-sso/)(API 서비스 키 및 환경 변수)한 후 이 곳에서 IdP 구성을 완료해야 한다.

---

## SCIM 프로비저닝

> 원문: [SCIM](https://www.pulumi.com/docs/administration/access-identity/scim/)

Pulumi Cloud는 SCIM(System for Cross-domain Identity Management) 2.0 통합을 지원하여, Identity Provider(IdP)에서 사용자와 그룹을 중앙 관리하고 Pulumi Cloud로 동기화할 수 있다. SCIM은 **Business Critical** 에디션에서만 사용할 수 있다.

SCIM으로 관리되는 팀 외에도 Pulumi Cloud 내에서 로컬 팀을 직접 구성하고 관리할 수 있다.

> **참고:** Pulumi는 SCIM 애플리케이션당 하나의 Pulumi 조직만 지원한다. 여러 Pulumi 조직을 관리하는 경우 각 조직마다 별도의 SCIM 애플리케이션을 구성해야 한다.

### SCIM 지원 Identity Provider

| Identity Provider | 문서 |
|-------------------|------|
| Microsoft Entra ID (구 Azure AD) | [Entra ID SCIM 가이드](https://www.pulumi.com/docs/administration/access-identity/scim/entra/) |
| Okta | [Okta SCIM 가이드](https://www.pulumi.com/docs/administration/access-identity/scim/okta/) |
| OneLogin | [OneLogin SCIM 가이드](https://www.pulumi.com/docs/administration/access-identity/scim/onelogin/) |

SCIM FAQ는 [SCIM FAQ](https://www.pulumi.com/docs/administration/access-identity/scim/faq/)를 참조하라.

---

## Access Tokens

> 원문: [Access Tokens](https://www.pulumi.com/docs/administration/access-identity/access-tokens/)

Access Tokens는 CLI를 통한 Pulumi Cloud 로그인 또는 REST API를 사용한 자동화에 사용된다. `pulumi login`에 사용한 토큰은 `pulumi api` 명령에도 사용되며, 이 명령은 `Authorization` 헤더를 직접 설정하지 않고도 CLI에서 REST API 엔드포인트를 직접 호출할 수 있다.

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
- Access token은 조직의 액세스 관리 설정이 모든 멤버의 스택 생성을 허용하거나, 토큰에 할당된 역할에 `stack:create` scope가 포함된 경우 스택을 생성할 수 있다. Admin organization token은 항상 이 기능을 갖는다. 스택 생성자는 자동으로 소유자가 되어 삭제 권한을 포함한 모든 스택 권한을 갖는다

### 토큰 권한 작동 방식

**Personal Token 권한:** Personal access token은 생성한 사용자와 동일한 권한을 가진다. 사용자가 속한 모든 Pulumi Cloud 조직의 멤버십, 팀 멤버십, 역할 할당이 모두 포함된다.

**Organization Token 권한:** Organization token은 조직 자체로 인증한다. 고정된 권한 집합이 아니라 **생성 시 할당된 RBAC 역할**에서 권한을 파생한다. 역할을 할당하지 않으면 조직의 기본 멤버 역할을 받는다. Personal token과 달리 단일 조직으로 제한되며, 팀원 가입/퇴사에 영향을 받지 않는다.

**Team Token 권한:** Team token은 특정 팀의 권한으로 인증한다. 유효 권한은 **각 요청 평가 시점에 팀에 할당된 역할**에 의해 결정된다. 팀의 역할 할당이 변경되면 토큰의 권한도 즉시 반영된다. Team token은 장기 실행 자동화에서 접근 요구사항이 변할 수 있는 경우에 적합하다.

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

### Team Token

Team Token은 특정 팀의 리소스와 권한으로 제한된 머신 토큰이다. CI/CD 파이프라인 등 특정 팀이 관리하는 인프라에만 접근해야 하는 자동화 프로세스에 적합하다. 개별 팀 멤버의 Personal token을 사용할 필요가 없다.

- 조직 관리자와 팀 관리자가 생성/삭제 가능. **Teams > 팀 선택 > Access Tokens**에서 관리
- 토큰의 유효 권한은 팀에 할당된 역할의 합집합이며, 요청 시점에 평가된다
- 팀의 역할 할당이 변경되면 토큰 권한도 자동 반영. 토큰 재생성 불필요
- 감사 로그에 토큰 이름으로 기록되어 개인 사용자 노출 없이 추적 가능
- 토큰 이름은 삭제 후에도 영구 예약되어 감사 로그 무결성 보존
- 삭제 시 즉시 접근 권한 회수

### Legacy Organization Token Types

역할 할당이 도입되기 전, Organization Token은 두 가지 고정 권한 수준 중 하나로 생성되었다:

| Legacy 유형 | 설명 | 현재 대응 |
|-------------|------|-----------|
| **Standard** | 멤버 수준 권한. 읽기/쓰기 가능하지만 멤버 관리, 조직 설정, 관리자 작업 불가 | Built-in Member 역할 할당에 해당 |
| **Admin** | 전체 관리자 수준 권한. 다른 Organization Token 생성/삭제 제외 모든 작업 가능 | Built-in Admin 역할 할당에 해당 |

두 유형 모두 계속 작동하며, admin/standard 구분은 현재 RBAC 시스템의 Built-in Admin/Member 역할에 직접 매핑된다. 새 자동화에는 Custom Role을 할당하여 최소 권한 원칙을 따르는 것을 권장한다.

CI/CD 워크플로우(GitHub Actions, GitLab CI, Bitbucket Pipelines, AWS EKS, Google GKE 등)에서 OIDC(OpenID Connect)를 통해 Pulumi Cloud에 인증할 수 있다. OIDC 인증은 장수명 시크릿(access token)을 저장하지 않고도 임시 자격 증명으로 Pulumi Cloud에 접근할 수 있는 보안 메커니즘이다.

**OIDC 토큰 교환 흐름:**

1. 외부 워크로드가 호스트 서비스로부터 OIDC id_token을 획득
2. 워크로드가 id_token을 Pulumi Cloud에 제출하여 단기 Pulumi access token으로 교환
3. 워크로드가 Pulumi access token으로 Pulumi 작업 실행

**OIDC 토큰 타입:**

| 토큰 타입 | 설명 | 사용 가능 에디션 |
|-----------|------|-----------------|
| **Personal** | 특정 사용자의 권한으로 인증 | Individual: personal만 |
| **Organization** | 조직 자체로 인증, RBAC 역할 할당 가능 | Team 이상 (Team: personal + organization) |
| **Team** | 팀 권한으로 인증 | Enterprise, Business Critical |
| **Deployment Runner** | 배포 실행을 위한 전용 토큰 | Business Critical |

에디션별 사용 가능 토큰 타입: **Individual** = personal만, **Team** = personal + organization, **Enterprise** = personal + organization + team, **Business Critical** = personal + organization + team + deployment-runner.

**OIDC 토큰의 관리자 권한:** OIDC 발급 액세스 토큰은 기본적으로 관리자 권한을 받지 않는다. 스택 생성이나 삭제와 같이 상승된 권한이 필요한 작업을 수행하려면, OIDC 토큰 교환 시 명시적으로 `admin` scope를 요청해야 한다. 예시:

```json
{
  "provider": "github",
  "audience": "pulumi",
  "scope": "admin"
}
```

이 `scope` 필드는 OIDC 토큰 교환 시 관리자 권한을 요청하는 데 사용된다. 한편 REST API 직접 호출 시 `scope` 파라미터는 `team:{TEAM_NAME}` 또는 `user:{USER_LOGIN}` 형식으로 대상 팀/사용자를 지정하는 용도로 사용되며, authorization policy의 subject claim 매칭과 토큰 타입에 할당된 RBAC 역할로 권한이 결정된다. 자세한 구성 방법은 아래 OIDC Issuers 섹션을 참조하라.

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

OIDC Issuers는 CI/CD 파이프라인에서 장수명 시크릿 없이 Pulumi Cloud에 인증하기 위한 OpenID Connect 발급자를 등록하고 관리하는 기능이다. OIDC Issuer를 등록하면 GitHub Actions, GitLab CI, AWS EKS, Google GKE, Bitbucket Pipelines 등의 워크플로우가 임시 자격 증명으로 Pulumi Cloud에 안전하게 접근할 수 있다. OIDC Issuer는 외부 서비스에서 Pulumi Cloud로의 **인바운드** 토큰 수락을 구성하는 것이며, Pulumi Cloud에서 다른 서비스로 토큰을 발급하는 데 사용되지 않는다.

### OIDC Issuer 관리 방법

| 방법 | 설명 |
|------|------|
| **Pulumi Cloud UI** | Settings > Access Management > OIDC Issuers에서 구성 |
| **REST API** | [OIDC Issuers REST API reference](https://www.pulumi.com/docs/reference/service-rest-api/) 사용 |
| **Pulumi Service Provider** | `OidcIssuer` 리소스로 코드로 관리 |

### OIDC Issuer 등록

조직 관리자는 **Settings > Access Management > OIDC Issuers**에서 **Register issuer**를 선택하여 발급자를 등록할 수 있다. 등록 시 다음 필드를 제공한다:

| 필드 | 설명 |
|------|------|
| **Name** | 발급자의 라벨 (조직 내 고유 이름). 감사 로그 및 정책 관리에서 식별에 사용 |
| **URL** | 발급자 URL. Pulumi Cloud는 이 URL에 `/.well-known/openid-configuration`을 추가하여 OpenID 구성 메타데이터를 가져온다 |
| **Max expiration** | 이 신뢰 관계를 통해 발급되는 Pulumi access token의 최대 지속 시간. 기본값 25시간 |
| **Thumbprints** (선택) | 발급자가 OpenID 구성을 서비스하는 데 사용하는 TLS 인증서의 SHA-256 지문. 기본적으로 등록 시점의 인증서 지문이 저장됨. 여러 인증서를 사용하거나 인증서 순환이 필요한 경우 수동 구성 |

**지원 OIDC 공급자 및 공급자별 설정 가이드:**

| 공급자 | 발급자 URL | 설명 |
|--------|-----------|------|
| [GitHub Actions](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/github/) | `https://token.actions.githubusercontent.com` | GitHub OIDC 공급자. `pulumi/auth-actions` 액션 사용 |
| [GitLab CI](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/gitlab/) | `https://gitlab.com` | GitLab OIDC 공급자 |
| [Amazon EKS](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/kubernetes-eks/) | EKS 클러스터 OIDC 발급자 URL | EKS 파드 서비스 계정 기반 인증. `kubernetes.io` 네임스페이스 클레임 사용 |
| [Google GKE](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/kubernetes-gke/) | GKE 클러스터 OIDC 발급자 URL | GKE 워크로드 아이덴티티 기반 인증 |
| Custom | 사용자 정의 | 기타 OIDC 호환 공급자 |

### 에디션별 토큰 타입

OIDC 토큰 타입의 에디션별 가용성은 다음과 같다:

- **Individual**: personal
- **Team**: personal, organization
- **Enterprise**: personal, organization, team
- **Business Critical**: personal, organization, team, deployment-runner

Authorization policy와 토큰 요청 시, 해당 에디션에서 사용 가능한 토큰 타입을 선택해야 한다.

### OIDC Issuer 관리 방법

| 방법 | 설명 |
|------|------|
| **Pulumi Cloud UI** | Settings > Access Management > OIDC Issuers에서 구성 |
| **REST API** | [OIDC Issuers REST API reference](https://www.pulumi.com/docs/reference/cloud-rest-api/oidc-issuers/) 사용 |
| **Pulumi Service Provider** | `OidcIssuer` 리소스로 코드로 관리 ([레지스트리](https://www.pulumi.com/registry/packages/pulumiservice/api-docs/oidcissuer/) 참조) |

새 OIDC Issuer를 등록하면 Pulumi Cloud는 기본적으로 모든 토큰 교환을 거부하는 기본 authorization policy를 프로비저닝한다. 토큰 교환 전에 명시적인 **Allow** 정책을 추가해야 한다.

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
      - name: OIDC 토큰 발급
        id: pulumi-oidc
        uses: pulumi/auth-actions@v1
      - name: Pulumi CLI 설치
        uses: pulumi/action-install-pulumi-cli@v2
      - name: Pulumi 로그인 (OIDC)
        run: pulumi login --oidc-token ${{ steps.pulumi-oidc.outputs.PULUMI_ACCESS_TOKEN }} --oidc-org <org-name>
```

subject claim 매칭 예시:
- `repo:my-org/my-repo:*` — 특정 리포지토리의 모든 워크플로우 허용
- `repo:my-org/my-repo:ref:refs/heads/main` — main 브랜치에서만 허용
- `repo:my-org/my-repo:environment:production` — production 환경에서만 허용

**Nested claims 지원:** 점으로 중첩된 claim 경로를 정의할 수 있다. EKS/GKE 등 Kubernetes OIDC 발급자에서 `kubernetes.io` 네임스페이스의 클레임을 타겟팅할 때 특히 유용하다. 예를 들어 다음과 같은 토큰 페이로드가 있을 때:

```json
{
  "kubernetes.io": {
    "namespace": "production",
    "pod": {
      "name": "runner-ddfaa34e-dfrjh",
      "uid": "b99b58df-cce5-405a-a33d-49a4cf8cf7bd"
    },
    "serviceaccount": {
      "name": "pulumi-sa",
      "uid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
    }
  }
}
```

claim 경로 `"kubernetes.io".pod.name`으로 pod 이름을, `"kubernetes.io".namespace`로 네임스페이스를, `"kubernetes.io".serviceaccount.name`으로 서비스 계정 이름을 타겟팅할 수 있다. 점이 포함된 객체 키는 따옴표로 감싼다.

**Wildcard 지원:** Claim 값과 team scope에 다음 와일드카드를 사용할 수 있다:

| 패턴 | 의미 |
|------|------|
| `*` | 0개 이상의 문자 매칭 |
| `?` | 0개 또는 1개의 문자 매칭 |
| `.` | 정확히 1개의 문자 매칭 |

예: `runner-*`는 `runner-`로 시작하는 모든 pod 이름과 매칭된다.

### 썸프린트(Thumbprint)

OIDC Issuer 등록 시 공급자의 인증서 썸프린트를 제공해야 할 수 있다. 썸프린트는 OIDC 공급자의 TLS 인증서 지문(**SHA-256**)으로, 토큰의 출처를 검증하는 데 사용된다. 기본적으로 Pulumi Cloud는 등록 시 사용된 인증서의 썸프린트를 저장한다. 공급자가 여러 인증서를 사용하거나 인증서 순환이 필요한 경우 수동으로 구성한다.

**썸프린트 계산 방법:**

```bash
# 1. 발급자의 인증서 가져오기 (example.com을 발급자 호스트명으로 교체)
openssl s_client -servername example.com -showcerts -connect example.com:443

# 2. 출력에서 첫 번째 인증서를 certificate.crt 파일로 저장

# 3. SHA-256 썸프린트 계산
openssl x509 -in certificate.crt -fingerprint -sha256 -noout
# > sha256 Fingerprint=2B:60:30:08:8E:8D:08:FC:D6:1B:8B:89:70:19:F2:D9:9F:4B:9A:0F:7B:46:5B:06:5C:2B:90:E1:C5:3B:C0:7D

# 4. 콜론 제거
# 2B6030088E8D08FCD61B8B897019F2D99F4B9A0F7B465B065C2B90E1C53BC07D
```

여러 인증서를 사용하는 공급자의 경우 각 인증서에 대해 썸프린트를 추가해야 한다.

### CLI 및 REST API를 통한 토큰 교환

**CLI를 통한 OIDC 인증:**

Pulumi CLI는 `pulumi login` 명령으로 OIDC 토큰 교환을 기본 지원한다. 대부분의 사용 사례에서 권장되는 방식이다.

```bash
# Pulumi CLI에서 OIDC 인증 사용 (권장)
pulumi login --oidc-token <token> --oidc-org <org-name>
```

`--oidc-token` 플래그는 원시 토큰 문자열 또는 `file://` 접두사가 있는 파일 경로를 허용한다. 추가로 `--oidc-team`, `--oidc-user`, `--oidc-expiration` 플래그를 사용할 수 있다. 셀프 호스팅 백엔드를 사용하는 경우 백엔드 URL을 명시적으로 지정해야 한다:

```bash
pulumi login https://api.pulumi.com --oidc-token <token> --oidc-org <org-name>
```

**REST API를 통한 토큰 교환:**

고급 시나리오에서 토큰 교환 과정을 직접 제어해야 하는 경우, OAuth 2.0 token-exchange 그랜트 타입으로 토큰 엔드포인트를 호출한다. 엔드포인트는 `application/json`과 `application/x-www-form-urlencoded` 콘텐츠 타입을 모두 지원한다.

**파라미터:**

| 파라미터 | 설명 |
|----------|------|
| `audience` | `urn:pulumi:org:{ORG_NAME}` |
| `grant_type` | `urn:ietf:params:oauth:grant-type:token-exchange` |
| `subject_token_type` | `urn:ietf:params:oauth:token-type:id_token` |
| `requested_token_type` | Organization: `urn:pulumi:token-type:access_token:organization`, Team: `urn:pulumi:token-type:access_token:team`, Personal: `urn:pulumi:token-type:access_token:personal` |
| `scope` | Team 또는 Personal 토큰 요청 시 대상 지정. `team:{TEAM_NAME}` 또는 `user:{USER_LOGIN}` 형식 |
| `expiration` | 토큰 만료 시간(초). 기본값 2시간 |
| `subject_token` | OIDC 공급자가 발급한 id_token |

```bash
# OIDC 토큰을 Pulumi access token으로 교환 (OAuth 2.0 token-exchange)
curl -X POST \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'audience=urn:pulumi:org:{ORG_NAME}' \
  -d 'grant_type=urn:ietf:params:oauth:grant-type:token-exchange' \
  -d 'subject_token_type=urn:ietf:params:oauth:token-type:id_token' \
  -d 'requested_token_type=urn:pulumi:token-type:access_token:organization' \
  -d 'subject_token=<oidc-id-token>' \
  https://api.pulumi.com/api/oauth/token
```

응답 예시:

```json
{
  "access_token": "...",
  "issued_token_type": "urn:pulumi:token-type:access_token:organization",
  "token_type": "token",
  "expires_in": 7200,
  "scope": ""
}
```

---

## 감사 로그(Audit Logs)

> 원문: [Audit Logs](https://www.pulumi.com/docs/administration/security-compliance/audit-logs/)

감사 로그는 **Enterprise** 및 **Business Critical** 에디션에서 사용할 수 있다. 조직 관리자만 감사 로그를 조회할 수 있다.

감사 로그는 조직 내 사용자 활동을 추적한다. 사용자가 수행한 작업, 수행 시점, 출처 IP 등을 기록하며, 로그는 변경 불가능(immutable)하다.

### 감사 로그 조회

1. 조직의 **Settings**로 이동
2. **Audit Logs** 선택

최근 이벤트가 내림차순으로 표시되며, 특정 사용자의 프로필 사진을 선택하여 필터링할 수 있다.

### 감사 로그 내보내기

**수동 내보내기:**

- 콘솔: **Settings > Audit Logs > Download**
- REST API: `/api/orgs/{orgName}/auditlogs` 엔드포인트 사용. 자세한 내용은 [Pulumi Cloud REST API](https://www.pulumi.com/docs/reference/service-rest-api#audit-logs) 참조

**자동 내보내기 (Business Critical 전용):**

| 대상 | 설명 |
|------|------|
| [AWS S3](https://www.pulumi.com/docs/administration/security-compliance/audit-logs/aws-s3/) | Amazon S3 버킷으로 감사 로그 지속적 내보내기 |
| [Microsoft Sentinel](https://www.pulumi.com/docs/administration/security-compliance/audit-logs/azure-sentinel/) | Microsoft Sentinel로 SIEM 분석용 감사 로그 내보내기 |

### 지원 감사 로그 형식

**JSON 형식:**

| 필드 | 설명 |
|------|------|
| `timestamp` | 이벤트가 기록된 UNIX 타임스탬프 |
| `sourceIP` | 요청을 발생시킨 클라이언트 IP 주소 |
| `event` | 이벤트 이름 |
| `description` | 발생한 이벤트의 상세 설명 |
| `user` | 이벤트를 호출한 사용자 정보(login, name, avatar URL) |

**CSV 형식:**

| 필드 | 설명 |
|------|------|
| `Timestamp` | 이벤트가 기록된 UNIX 타임스탬프 |
| `Name` | 이벤트를 호출한 사용자 이름 |
| `Login` | 사용자 로그인명 |
| `Event` | 이벤트 이름 |
| `Description` | 이벤트 상세 설명 |
| `SourceIP` | 요청을 발생시킨 클라이언트 IP |
| `RequireOrgAdmin` | 조직 관리자 권한 필요 여부 (`true`/`false`) |
| `RequireStackAdmin` | 스택 관리자 권한 필요 여부 (`true`/`false`) |
| `AuthenticationFailure` | 인증 실패로 인한 이벤트 여부 (`true`/`false`) |

**CEF (Common Event Format):** SIEM 시스템에서 널리 지원하는 표준 감사 및 로깅 이벤트 형식. CEF 표준 사전 정의 키(`dvchost`, `rt`, `src`, `suser`)와 Pulumi 커스텀 키(`orgID`, `userID`, `requireOrgAdmin`, `requireStackAdmin`, `authenticationFailure`)를 포함한다.

### 주요 감사 로그 이벤트

| 이벤트 | 설명 |
|--------|------|
| Auth Failure Organization Role | 사용자가 필요한 조직 역할 없이 작업을 시도함 |
| Auth Failure SCIM Access Token | SCIM 지원 요청에 잘못된 인증 토큰이 사용됨 |
| Auth Failure Stack Permission | 사용자가 필요한 스택 권한 없이 작업을 시도함 |
| Member Added | 조직 멤버 추가 |
| Member Removed | 조직 멤버 제거 |
| Member Role Changed | 멤버 역할 변경 |
| Organization Settings Changed | 조직 설정 변경 |
| Stack Created | 스택 생성 |
| Stack Deleted | 스택 삭제 |
| Stack Renamed | 스택 이름 변경 |
| Stack Created From Template | 템플릿으로부터 스택 생성 |
| Stack Update Started | 스택 업데이트 시작 |
| Stack Update Completed | 스택 업데이트 완료 |
| Stack Update Canceled | 스택 업데이트 취소 |
| Stack Exported | 스택 내보내기 |
| Stack Imported | 스택 가져오기 |
| Stack Transferred to Organization | 스택 조직 간 이전 |
| Team Created | 팀 생성 |
| Team Deleted | 팀 삭제 |
| Team Updated | 팀 업데이트 |
| Secret Decrypted | 시크릿 복호화 |
| Stack Collaborator Added | 스택 협업자 추가 |
| Stack Collaborator Removed | 스택 협업자 제거 |
| Stack Collaborator Permissions Changed | 스택 협업자 권한 변경 |
| Policy Pack Created | 정책 팩 생성 |
| Policy Pack Deleted | 정책 팩 삭제 |
| Policy Pack Enabled | 정책 팩 활성화 |
| Policy Pack Disabled | 정책 팩 비활성화 |
| Policy Group Created | 정책 그룹 생성 |
| Policy Group Deleted | 정책 그룹 삭제 |
| Policy Group Updated | 정책 그룹 업데이트 |
| SAML Configuration Updated | SAML 구성 업데이트 |
| User Login | 사용자 로그인 성공 |
| User Login Failed | 사용자 로그인 실패 |
| User Added New Identity to Their Account | 사용자가 새 identity를 계정에 연결 |
| Environment Created | 환경 생성 |
| Environment Updated | 환경 업데이트 |
| Environment Deleted | 환경 삭제 |
| Environment Open | 환경 열기 |
| Environment Read | 환경 읽기 |
| Environment Read Open | 환경 열기 및 읽기 |
| Environment Unauthorized Open | 권한 없는 환경 열기 시도 |
| Environment Tag Created | 환경 태그 생성 |
| Environment Tag Updated | 환경 태그 수정 |
| Environment Tag Deleted | 환경 태그 삭제 |
| Environment Version Retracted | 환경 버전 철회 |
| Environment Version Tag Open | 환경 버전 태그 열기 |
| Environment Version Tag Created | 환경 버전 태그 생성 |
| Environment Version Tag Read | 환경 버전 태그 읽기 |
| Environment Version Tag Update | 환경 버전 태그 수정 |
| Environment Version Tag Delete | 환경 버전 태그 삭제 |
| Environment Decrypted | 환경 복호화 |
| Environment Clone | 환경 복제 |
| Environment Restored | 환경 복원 |
| Environment Schedule Created | 환경 스케줄 생성 |
| Environment Schedule Updated | 환경 스케줄 수정 |
| Environment Schedule Deleted | 환경 스케줄 삭제 |
| Environment Rotated | 환경 시크릿 순환 |
| Stack Provider Open | 환경 내 스택 공급자 열기 |
| Customer Managed Key Added | 고객 관리 키 추가 |
| Customer Managed Key Set Default | 고객 관리 키 기본 설정 |
| Customer Managed Key Disabled | 고객 관리 키 비활성화 |
| Customer Managed Key Disabled All | 모든 고객 관리 키 비활성화 |

---

## Security & Compliance (보안 및 컴플라이언스)

> 원문: [Security & Compliance](https://www.pulumi.com/docs/administration/security-compliance/)

보안 제어, 컴플라이언스 모니터링, 감사 로깅을 구성한다.

### Customer Managed Keys (CMK)

> 원문: [Customer Managed Keys](https://www.pulumi.com/docs/administration/security-compliance/customer-managed-keys/)

Customer Managed Keys는 **Enterprise** 및 **Business Critical** 에디션에서 사용할 수 있다. 자체 암호화 키를 사용하여 Pulumi Cloud의 민감 데이터를 보호할 수 있다.

CMK는 외부 Key Management System(KMS)을 통해 데이터 키를 암호화하며, 첫 CMK 추가 시 기존 Pulumi 관리 키로 암호화된 모든 데이터 키가 자동으로 새 CMK로 재암호화된다. 암호화된 데이터 자체는 변경되지 않는다.

> **참고:** 현재 Customer Managed Keys는 Pulumi ESC 데이터 암호화에만 사용되며, AWS KMS만 지원한다. 추가 KMS 공급자 및 Pulumi 제품 확장 지원은 개발 중이다.

**CMK 관리 (조직 관리자만):**

1. **Settings > Organization > Customer Managed Keys** 탭에서 관리
2. **Add Customer Managed Key**: AWS IAM Role ARN 및 KMS Key ARN 입력 (Alias ARN도 지원)
3. **Disable**: 기존 데이터 키 재암호화용 키를 선택 후 비활성화. 기본 키는 비활성화 불가
4. **Disable all**: 모든 CMK를 비활성화하고 Pulumi 관리 키로 재암호화

### AWS KMS 설정 단계

1. AWS IAM에 역할을 생성하고 AWS KMS에 키를 생성한다. 자세한 설정 방법은 [AWS KMS 설정 가이드](https://www.pulumi.com/docs/administration/security-compliance/customer-managed-keys/aws-kms/)를 참조하라
2. Pulumi Cloud의 **Customer Managed Keys** 설정 페이지로 이동
3. **Add Customer Managed Key** 클릭
4. 키의 고유 이름 입력
5. AWS KMS 키에 접근 권한이 있는 **Role ARN** 입력
6. AWS KMS 키의 **Key ARN** 입력 (Alias ARN도 지원)

**CMK 페이지에 표시되는 정보:**

| 항목 | 설명 |
|------|------|
| Name | 관리자가 제공한 키의 고유 이름 |
| Type | 암호화 키 유형 (예: AWS KMS) |
| Default | 조직의 기본 암호화 키 여부. 새 데이터 키가 이 키로 암호화됨 |
| Set as default | 기본 키로 설정 버튼. 이미 기본이거나 재암호화 중인 키는 비활성 |
| Disable | 키 비활성화 버튼. 기본 키 또는 재암호화 중인 키는 비활성 |

---

## Ephemeral Environments (임시 환경)

> 원문: [Review Stacks](https://www.pulumi.com/docs/deployments/deployments/review-stacks/) | [TTL Stacks](https://www.pulumi.com/docs/deployments/deployments/ttl/)

Ephemeral Environments는 임시 클라우드 환경을 자동으로 생성하고 제거하는 기능으로, 비용 절감, 보안 강화, 운영 오버헤드 감소에 기여한다.

### Review Stacks

Review Stacks는 Pulumi Deployments로 구동되는 전용 클라우드 환경이다. Pull Request가 열리면 자동으로 생성되고, 새 커밋마다 업데이트되며, PR이 병합되거나 닫히면 자동으로 삭제된다.

**지원 VCS 통합:** GitHub, GitLab, Azure DevOps, Bitbucket (Custom VCS는 미지원)

**구성 단계:**

1. 관례적으로 `pr`라는 새 스택 및 `Pulumi.pr.yaml` 구성 파일 생성
2. 스택에 [Deployment Settings](https://www.pulumi.com/docs/deployments/deployments/reference/#deployment-settings) 구성
3. `pullRequestTemplate` Deployment Setting을 `true`로 설정

```bash
# REST API로 Review Stack 활성화
curl -i -XPOST -H "Content-Type: application/json" \
  -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
  --location "https://api.pulumi.com/api/stacks/org/project/stack/deployments/settings" \
  -d '{
    "gitHub": {
      "pullRequestTemplate": true
    }
  }'
```

**일반적인 패턴:**

| 패턴 | 설명 |
|------|------|
| 단일 스택 | 동일 스택에 push-to-deploy, PR preview, review stacks 모두 구성. 가장 간단하지만 동일 클라우드 계정 사용 |
| 분리 스택 | Review stack용 별도 스택 및 구성. 다른 클라우드 계정에 배포 가능 |
| 다중 Pulumi 프로그램 | 공유 Kubernetes 클러스터 등을 활용한 복합 구성 |
| Path 필터 | 코드 변경 경로에 따라 다른 review stack 템플릿 선택 |
| GitHub Label 게이트 | `reviewStackLabels` 설정으로 특정 레이블이 있는 PR에만 review stack 생성 |

### TTL (Time-to-Live) Stacks

TTL Stacks는 지정한 날짜/시간 이후 스택을 자동으로 제거(destroy)하는 수명 관리 기능이다. 플랫폼 팀이 비용 통제, 보안 태세 개선, 운영 오버헤드 감소를 달성할 수 있게 한다.

**구성 방법:**

| 방법 | 설명 |
|------|------|
| Pulumi Cloud UI | Stack > Settings > Schedules > Time-to-Live 선택 후 cron 표현식으로 설정 |
| REST API | `POST /api/stacks/{org}/{project}/{stack}/deployments/ttl/schedules` 엔드포인트 사용 |
| Pulumi Cloud Service Provider | `pulumiservice.TtlSchedule` 리소스로 소스 제어에서 관리 |

```bash
# REST API로 TTL 스케줄 설정
curl -H "Accept: application/vnd.pulumi+json" \
     -H "Content-Type: application/json" \
     -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
     --request POST \
     --data '{"timestamp":"2024-12-31T23:59:59Z","deleteAfterDestroy":true}' \
     https://api.pulumi.com/api/stacks/{organization}/{project}/{stack}/deployments/ttl/schedules
```

```typescript
// Pulumi Cloud Service Provider로 TTL 설정
import * as pulumiservice from "@pulumi/pulumiservice";

const ttlSchedule = new pulumiservice.TtlSchedule("ttlSchedule", {
  organization: "my-org",
  project: "my-project",
  stack: "temp-stack",
  timestamp: "2024-01-01T00:00:00Z",
});
```

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

| 항목 | Docker 리포지토리 | 설명 |
|------|-------------------|------|
| [API](https://www.pulumi.com/docs/administration/self-hosting/components/api/) | `pulumi/service` | Pulumi Cloud 백엔드 API |
| [Web Console](https://www.pulumi.com/docs/administration/self-hosting/components/console/) | `pulumi/console` | Pulumi Cloud 프론트엔드 UI |
| Migrations | `pulumi/migrations` | 데이터베이스 마이그레이션 |
| [Search](https://www.pulumi.com/docs/administration/self-hosting/components/search/) | - | 검색 서비스 |
| [Deployments](https://www.pulumi.com/docs/administration/self-hosting/components/deployments/) | - | 배포 실행 서비스 |
| [Network Requirements](https://www.pulumi.com/docs/administration/self-hosting/network/) | - | 인그레스, 이그레스, 인프라 요구 사항 |

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
| 감사 로그 | Audit Logs | 조직 활동 로그 조회 및 내보내기 (Enterprise / Business Critical) |
| Customer Managed Keys | Settings > Organization > Customer Managed Keys | 자체 암호화 키 관리 (Enterprise / Business Critical) |
| Review Stacks | Stack > Deployment Settings | PR 기반 임시 환경 자동 생성/삭제 |
| TTL Stacks | Stack > Settings > Schedules | 수명 제한 스택 자동 제거 |
| Neo (AI) | Settings > Neo Settings > General | Pulumi Neo AI 에이전트 활성화/비활성화 |
| Neo CLI | 터미널에서 `pulumi neo` 실행 | 대화형/비대화형 Neo 세션. 로컬 툴 실행 모드 |
| Neo Tasks | Neo > Agent Tasks | AI 작업(Tasks), Automations, PR 관리 |
| Neo Context | Task 시작 시 스택/리포지토리 설정 | 작업 범위 지정. 공유 및 기록 관리 |
| Neo Previews | Task 중 Preview 실행 요청 | PR 생성 전 인프라 변경 검증. ESC 환경 지원 |
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

```go
organization := ctx.Organization()
```

```csharp
var organization = Deployment.Instance.OrganizationName;
```

```java
var organization = ctx.organizationName();
```

```yaml
variables:
  organization: ${pulumi.organization}
```

---

## Pulumi Neo (Infrastructure AI)

> 원문: [Infrastructure AI](https://www.pulumi.com/docs/ai/) | [Get Started](https://www.pulumi.com/docs/ai/get-started/) | [Tasks](https://www.pulumi.com/docs/ai/tasks/) | [Pull Requests](https://www.pulumi.com/docs/ai/pull-requests/) | [Automations](https://www.pulumi.com/docs/ai/automations/) | [Integrations](https://www.pulumi.com/docs/ai/integrations/) | [`pulumi neo` CLI](https://www.pulumi.com/docs/iac/cli/commands/pulumi_neo/)

**Pulumi Neo**는 자연어 기반의 AI 인프라 자동화 에이전트다. 배포 디버깅, IaC 작성, 환경 질문 응답 등 플랫폼 엔지니어링 작업을 지원한다. Neo는 현재 **Public Preview** 상태이며 무료로 사용할 수 있다.

### `pulumi neo` CLI 세션

> 원문: [`pulumi neo`](https://www.pulumi.com/docs/iac/cli/commands/pulumi_neo/)

`pulumi neo` CLI 명령으로 터미널에서 대화형 Neo 세션을 시작할 수 있다. CLI 모드에서는 파일시스템 및 셸 툴 호출이 클라우드 에이전트 컨테이너 대신 **로컬 머신**의 작업 디렉토리에서 실행된다.

```bash
# 대화형 Neo 세션 시작
pulumi neo

# 프롬프트와 함께 바로 시작
pulumi neo "VPC 구성을 검토해줘"

# 특정 스택 컨텍스트로 시작
pulumi neo --stack my-org/my-project/dev "리소스 사용량을 분석해줘"

# 비대화형 모드: 결과를 stdout에 출력하고 종료 (다른 AI 에이전트/스크립트용)
pulumi neo -p "현재 스택의 출력 값을 나열해줘"
```

**주요 플래그:**

| 플래그 | 설명 |
|--------|------|
| `--approval-mode` | 툴 호출 승인 모드: `manual`(모든 호출 시 확인), `balanced`(저위험 자동 승인), `auto`(모두 자동 실행). 기본값 `manual` |
| `--permission-mode` | 권한 모드: `default`(역할 기반 전체 권한), `read-only`(상태 변경 차단). 기본값 `default` |
| `--cwd` | 로컬 툴 실행 작업 디렉토리. 기본값은 현재 디렉토리 |
| `--org` | Neo 태스크를 소유할 조직. 기본값은 사용자의 기본 조직 |
| `-s`, `--stack` | Neo 태스크에 연결할 스택 이름 |
| `-p`, `--print` | 비대화형 모드. 단일 프롬프트 실행 후 결과를 stdout에 출력하고 종료 |

### 활성화 및 비활성화

Neo는 기본적으로 활성화되어 있다. 비활성화하려면 **Settings > Neo Settings > General**에서 설정한다.

Neo에 접근하려면 Pulumi Cloud 콘솔의 왼쪽 탐색 메뉴에서 **Neo** 섹션 내 **Agent Tasks**를 선택한다.

### Neo 권한 모델

Neo는 대화하는 사용자의 [RBAC 권한](https://www.pulumi.com/docs/administration/access-identity/rbac/) 범위 내에서 작동하며, 사용자가 수행할 수 없는 작업은 Neo도 수행할 수 없다. 권한 에스컬레이션 위험이나 특별한 관리자 접근이 필요하지 않다.

| 권한 수준 | 설명 |
|-----------|------|
| **Use my permissions** | 전체 접근 (기본 동작) |
| **Read-only** | 읽기, Preview, PR 생성만 가능. 인프라 변경(mutation) 불가 |

Read-only 모드에서도 인프라 상태 읽기, Preview 실행, 코드 작성/리팩토링, 브랜치 생성, PR 열기는 가능하다. 배포 트리거나 Pulumi Cloud 직접 쓰기 작업만 제한된다.

### Tasks (작업)

Task는 Neo의 기본 작업 단위이다. 각 Task는 사용자가 목표를 설명하는 대화이며, Neo가 인프라 변경을 처리한다.

**Task 시작 시 Neo의 동작:**

1. 수행할 단계를 개요로 제시
2. 사용자의 승인에 따라 단계별 실행
3. 코드 수정 시 Preview 실행으로 변경 검증
4. 만족하면 [Pull Request](https://www.pulumi.com/docs/ai/pull-requests/) 생성 제안

#### Plan Mode

복잡한 작업의 경우 Plan Mode를 활성화하면, Neo가 실행 전에 환경을 심층 조사하고 계획을 수립한다.

Plan Mode 활성화 시 Neo의 동작:

1. 기존 인프라, 코드, 의존성 조사
2. 발견한 내용을 종합하여 계획 수립
3. 대화를 통해 가정 검증, 대안 요청 가능
4. 명시적 승인 전까지 실행하지 않음

#### Task Modes

Task modes는 실행 중 Neo의 자율성 수준을 제어한다. 작업 중 언제든 변경할 수 있다.

| 모드 | 설명 |
|------|------|
| **Review mode** (기본) | `pulumi preview`, `pulumi up`, PR 열기 모두 승인 필요 |
| **Balanced mode** | `pulumi up` 실행 전에만 승인 요청 |
| **Auto mode** | 어떤 승인도 요청하지 않음 |

Plan Mode와 Task modes는 독립적이다. 예를 들어 Plan Mode + Auto Mode 조합으로 사전에 접근법을 철저히 검토한 후 Neo가 자율 실행하도록 할 수 있다.

### Context, Sharing, History

**Entity context 설정:** Task 시작 시 스택과 리포지토리 컨텍스트를 설정할 수 있어, Neo가 작업 범위를 정확히 파악할 수 있다.

**소유권 및 공유:** 각 Task는 생성한 사용자에게 속한다. 기본적으로 비공개이지만, 조직 내 다른 사람과 읽기 전용 링크를 생성하여 공유할 수 있다. 공유된 Task는 대화 전체(Neo의 추론, 수행한 작업, 결과)를 볼 수 있다.

공유 시 보안 경계가 유지된다:
- 뷰어는 대화를 볼 수 있지만 작업을 트리거할 수 없다
- 공유 Task 내의 스택/리소스 링크는 뷰어의 기존 RBAC 권한을 그대로 적용한다
- 원래 Task 소유자가 전체 제어권을 유지한다

**중단 및 재개:** 브라우저를 닫거나 다른 페이지로 이동해도 Task는 계속 실행된다. Neo가 작업을 완료하거나 승인이 필요한 상황에 도달할 때까지 작동한다. 돌아오면 부재 중 진행 상황을 보여준다.

**Task 기록:** Neo Task는 Pulumi Cloud의 Agent Tasks 페이지에 저장되어 언제든 전체 기록에 접근할 수 있다. 단, 에이전트가 1시간 이상 유휴 상태인 경우 Task 캐시가 손실될 수 있다.

### Pull Requests

Neo가 제안하는 모든 변경은 PR을 통해 이루어진다. PR에는 변경 제목, 문제 설명, 수정 리소스 목록, Preview 출력 요약, Neo Task 링크가 포함된다.

**VCS 통합 필요:** Neo가 코드를 읽고 PR을 생성하려면 [버전 제어 통합](https://www.pulumi.com/docs/integrations/version-control/)이 필요하다. GitHub, Azure DevOps, GitLab을 지원한다.

### Previews

> 원문: [Running Previews](https://www.pulumi.com/docs/ai/running-previews/)

Neo는 Pulumi Cloud에서 직접 `pulumi preview`를 실행하여 제안된 인프라 변경을 PR 생성 전에 검증할 수 있다. 이를 통해 예상치 못한 리소스 변경이나 정책 위반을 사전에 발견할 수 있다.

**사전 요구 사항:**
- 적용 가능한 클라우드 공급자 자격 증명 (AWS, Azure, GCP)
- Pulumi 프로그램에 필요한 추가 구성
- 자격 증명은 **Stack config** 또는 **ESC 환경**(권장)을 통해 제공. ESC는 OIDC 통합이 가능한 가장 유연한 방식

**Preview 워크플로우:**
1. Neo가 Task 해결책에 도달하면 preview 실행을 요청
2. 사용자가 승인 또는 거부
3. Preview 실행 후 Neo가 PR 열기를 제안
4. 사용자가 승인하거나 변경 요청 가능

**Preview 출력:** 수정될 리소스 수, 각 리소스의 구체적 변경 사항, 오류/경고, 정책 위반 사항(스택에 Pulumi IaC 정책이 연결된 경우)이 표시된다.

### Automations (자동화)

Automations는 Neo Task를 반복 작업으로 전환한다. 프롬프트를 정의하고 주기를 설정하면, Neo가 해당 간격으로 Task를 실행한다. 변경이 발생하면 PR을 연다. PR은 조직의 일반 리뷰 프로세스를 거치므로 branch protection 규칙과 필수 리뷰어가 여전히 적용된다.

| 기본 설정 | 설명 |
|-----------|------|
| 승인 모드 | **Auto** (각 단계에서 사람 승인 대기 없음) |
| 권한 모드 | **Read-only** (직접 인프라 쓰기 불가, PR으로만 제안) |

**설정 적용 우선순위:** (1) 자동화별 설정 > (2) 조직 수준 기본값 > (3) Auto 승인 + Read-only 권한 (기본 폴백)

**자동화 생성:** Neo Tasks > Automations 탭 > New automation. 템플릿(provider freshness check, encryption audit, backup audit, activity digest) 또는 빈 캔버스에서 시작. 이름, 프롬프트, 빈도(hourly, daily, weekdays, weekly)를 설정한다.

**권한:** 예약된 Task는 **예약한 사용자의 RBAC 권한**으로 실행되며, 실행 시점에 평가된다. 사용자의 권한이 예약과 실행 사이에 변경된 경우 새 권한이 적용된다.

**통합 상속:** Automations는 Neo의 컨텍스트 모델을 상속한다. 조직 및 프로젝트 수준의 Custom Instructions가 예약 Task에도 적용된다. MCP 통합은 구성한 사용자의 자격 증명을 사용하고, CLI 통합은 설정 시 구성된 자격 증명을 사용한다.

### Integrations (통합)

| 통합 유형 | 설명 |
|-----------|------|
| [MCP Integrations](https://www.pulumi.com/docs/ai/integrations/mcp/) | 이슈 트래커, 관측 플랫폼, 런북 위키, 온콜 도구에서 컨텍스트 가져오기 |
| [CLI Integrations](https://www.pulumi.com/docs/ai/integrations/cli/) | Pulumi ESC에서 관리하는 자격 증명으로 클라우드 공급자 CLI 호출 |
| [GitHub](https://www.pulumi.com/docs/ai/integrations/github/) | PR 스레드에서 Neo를 멘션하여 Task 시작 |
| [Slack](https://www.pulumi.com/docs/ai/integrations/slack/) | Slack 채널에서 Neo를 멘션하여 Task 시작 |

모든 통합은 조직 수준에서 관리자가 구성하며, 활성화되면 조직의 모든 Neo Task에서 사용할 수 있다.

### 제한 사항

- Neo는 코드를 통해서만 인프라를 수정할 수 있음. API나 UI 작업(배포 구성, 스택 설정 업데이트, 환경 관리 등)은 불가
- 새 리포지토리 생성 또는 Git 초기화 불가. 기존 리포지토리 내에서만 작업
- 새 Pulumi 프로젝트 초기화 불가. 이미 설정된 기존 프로젝트 내에서만 작업

---

## Onboarding Guide (조직 온보딩 가이드)

> 원문: [Onboarding Guide](https://www.pulumi.com/docs/administration/onboarding-guide/) | [Choose Subscription](https://www.pulumi.com/docs/administration/onboarding-guide/choose-subscription/) | [Ways of Working](https://www.pulumi.com/docs/administration/onboarding-guide/ways-of-working/) | [Setting Up for Success](https://www.pulumi.com/docs/administration/onboarding-guide/)

조직에 Pulumi를 도입하기 위한 종합 가이드다. 설정부터 권장 사용 패턴과 실천 방법까지 다룬다.

### 구독 티어 선택

| 티어 | 대상 | 지원 |
|------|------|------|
| **Individual / Team** | 소규모 팀 또는 시작 단계. [Pulumi Neo](https://www.pulumi.com/product/neo/), [Pulumi Registry](https://www.pulumi.com/registry/) 문서, [examples repo](https://github.com/pulumi/examples) 활용 | GitHub Discussions, Issues, Community Slack, 무료 워크숍 |
| **Enterprise** | 대규모 조직의 미션 크리티컬 워크로드 | 12x5 프리미엄 지원, 티켓팅, 보장 SLA, 프라이빗 Slack 채널 |
| **Business Critical** | 규제/에어갭 환경, 셀프 호스팅 필요 | 24x7 프리미엄 지원, 전담 계정 매니저 및 아키텍트, 우선 버그/기능 요청 |

### 배포 모델 선택

| 모델 | 설명 |
|------|------|
| **SaaS (권장)** | 대부분의 조직에 적합. 고가용성, 재해 복구, 지리적 복제 기본 제공. [보안 백서](https://www.pulumi.com/security/pulumi-cloud-security-whitepaper) 참조 |
| **Self-hosted** | Business Critical 전용. 에어갭 환경이나 격리된 Pulumi 플랫폼이 필요한 경우. [셀프 호스팅 가이드](https://www.pulumi.com/docs/administration/self-hosting/) 참조 |

> **참고:** 셀프 호스팅 부트스트랩 시 Pulumi Cloud를 아직 사용할 수 없으므로 [DIY backend](https://www.pulumi.com/docs/iac/operations/stack-management/using-a-diy-backend/)로 상태를 관리해야 한다.

### 결제 방식

| 방식 | 설명 |
|------|------|
| **월간 결제** | 신용카드로 매월 결제. 유연하며 변동 사용량에 적합 |
| **연간 약정** | 선불 송장 결제. 예측 가능한 사용량에 비용 절감 효과 |

### 프로젝트 구성 권장 사항

대부분의 팀은 작고 목적별로 구성된 프로젝트를 선호한다.

**일반적인 IaC 프로젝트 구조:**

| 프로젝트 유형 | 예시 |
|---------------|------|
| 보안 레이어 | IAM roles, KMS keys |
| 네트워크 기반 레이어 | VPCs, subnets |
| Kubernetes 클러스터 | 클러스터 정의 및 구성 |
| 데이터베이스 프로젝트 | Data lakes, MySQL 인프라 |
| 애플리케이션 프로젝트 | 컨테이너, VM, 서버리스 |

**언어 전략:**

| 전략 | 설명 |
|------|------|
| 단일 언어 (권장) | 도구, 패턴, 실천 방식의 표준화 용이. 팀이 이미 사용하는 언어 선택 |
| 다중 언어 | 대규모 조직에서 일반적. [Pulumi Packages](https://www.pulumi.com/docs/iac/concepts/packages/)로 한 언어로 작성하고 다른 언어에서 사용 가능 |

### 개발자 경험 패턴

| 패턴 | 설명 |
|------|------|
| 개별 개발자 스택 | 각 개발자에게 고유한 완전한 환경 제공 |
| 공유 인프라 | 데이터베이스 등 비용이 높은 리소스를 공유하면서 애플리케이션 인스턴스는 개별 제공 |
| [Review Stacks](https://www.pulumi.com/docs/deployments/deployments/review-stacks/) | 각 PR에 대해 단기 스택을 자동 생성/삭제 |
| [TTL Stacks](https://www.pulumi.com/docs/deployments/deployments/ttl/) | 지정 시간 후 자동 삭제되어 클라우드 비용 낭비 방지 |

### 개발자 셀프 서비스 모델

| 모델 | 설명 |
|------|------|
| **직접 Pulumi IaC 접근** | 별도 팀과 다른 RBAC 설정으로 개발자가 Pulumi 직접 사용. 컴포넌트, 템플릿, 정책으로 가드레일 적용 |
| **YAML 인터페이스** | 개발자가 YAML 파일을 Git에 체크인하고 CI/CD가 나머지 처리 |
| **UI 기반 경험** | 포털에 로그인하여 그래픽 인터페이스로 인프라 프로비저닝. [Pulumi IDP](https://www.pulumi.com/docs/idp/), [Backstage 플러그인](https://www.pulumi.com/blog/pulumi-backstage-plugin/), [Automation API](https://www.pulumi.com/docs/iac/concepts/automation-api/) 기반 커스텀 포털 |

### CI/CD 배포

| 옵션 | 설명 |
|------|------|
| [Pulumi Deployments](https://www.pulumi.com/docs/deployments/deployments/) (권장) | IaC 배포에 목적 구축된 Pulumi Cloud 통합 |
| [Pulumi Kubernetes Operator](https://www.pulumi.com/docs/integrations/clouds/kubernetes/pulumi-kubernetes-operator/) | Kubernetes 클러스터 내에서 배포 트리거 |
| 기존 CI/CD | GitHub Actions, GitLab CI, Octopus Deploy 등 |

> **Pro tip:** 조직 전체에 Pulumi를 빠르게 도입하려면 "Pulumi champions" 그룹을 식별하여 다른 팀원들이 바로 시작하도록 장려하라.
