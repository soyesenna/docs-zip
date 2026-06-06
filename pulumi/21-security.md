# Pulumi 보안과 컴플라이언스

> https://www.pulumi.com/docs/iac/operations/iac-least-privileges/
> https://www.pulumi.com/docs/pulumi-cloud/
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

Pulumi IaC는 인프라 프로비저닝 시 최소 권한(Least Privilege) 보안 태세를 채택하여 보안과 컴플라이언스를 동시에 충족한다. Pulumi Cloud는 RBAC(Role-Based Access Control), OIDC(OpenID Connect) 인증, 시크릿 암호화, 감사 로그, SAML SSO, SCIM 사용자 프로비저닝, Customer Managed Keys 등 다계층 보안 기능을 제공하며, Business Critical 에디션에서는 셀프 호스팅을 통한 완전한 인프라 제어가 가능하다.

---

## 최소 권한(Least Privilege) 원칙

### IaC 실행의 보안 함의

Pulumi IaC 프로그램은 완전한 애플리케이션으로, 런타임 환경의 전체 권한으로 실행된다. 이는 IaC 도구의 본질적 특성이며, 개발자가 IaC 코드를 직접 실행할 수 있다면 해당 프로그램이 사용하는 시크릿에도 접근할 수 있음을 의미한다. 따라서 프로덕션 환경에서는 격리된 보안 환경에서 배포를 실행해야 한다.

### 환경별 접근 권한 전략

| 환경 | 권한 수준 | 설명 |
|---|---|---|
| 개발/테스트 | 직접 실행 허용 | `pulumi up` 직접 실행, 빠른 반복과 디버깅 가능 |
| 프로덕션 | 직접 실행 제한 | PR 승인 프로세스 필수, 격리된 CI/CD 시스템에서 배포 |

---

## RBAC(역할 기반 접근 제어)

### 기본 조직 역할

| 역할 | 설명 |
|---|---|
| `Admin` | 조직의 모든 리소스와 설정에 대한 전체 접근 권한. 멤버·역할·조직 설정 관리 가능 |
| `Member` | 조직 리소스 조회 및 스택 작업 참여 가능. 조직 설정 수정 불가 |
| `Billing Manager` | 결제 정보 조회 및 관리만 가능. 기타 조직 설정이나 리소스 수정 불가 |

> **참고:** Teams 기능은 Pulumi Enterprise 및 Business Critical 에디션에서만 사용할 수 있다.

### 팀 기반 권한 관리

Pulumi Cloud는 팀을 통한 RBAC를 제공한다. 조직 관리자는 팀에 스택 권한을 일괄 할당할 수 있다.

**팀 생성 및 권한 할당 흐름:**

1. **Settings > Teams**에서 팀 생성
2. 스택 권한 할당: `Read`, `Write`, `Admin`
3. ESC 환경 권한 할당: `Environment reader`, `opener`, `editor`, `admin`
4. 스택 초기화 시 팀 권한 부여:

```bash
pulumi stack init --teams YourTeamName
```

### 팀 내 접근 유형

| 유형 | 설명 |
|---|---|
| `Team admin` | 팀 멤버 추가 가능 |
| `Team member` | 기본 역할, 팀에 할당된 권한만 행사 |

### 커스텀 역할

Enterprise/Business Critical 에디션에서는 커스텀 역할을 생성하여 세분화된 접근 제어가 가능하다.

| 규칙 유형 | 설명 |
|---|---|
| Direct entity access | 개별 엔티티(스택, 환경)에 직접 권한 부여 |
| Global entity access | 해당 유형의 모든 엔티티에 권한 일괄 부여 |
| Tag-based rules (ABAC) | 태그 조건(`env=production` 등)에 따라 권한 부여 |

### 스택 및 ESC 환경 권한

| 권한 | 스택 | ESC 환경 |
|---|---|---|
| `None` | 접근 불가 | - |
| `Read` | 스택 상태 조회 | 환경 정의 조회 (시크릿 복호화 불가) |
| `Write` | 스택 업데이트 | 환경 업데이트 |
| `Admin` | 전체 관리 | 환경 생성/수정/삭제 |
| `open` | - | 시크릿 복호화 및 동적 자격 증명 조회 |
| `opener` | - | 시크릿 복호화 및 동적 자격 증명 조회 |

---

## OIDC(OpenID Connect) 인증

### 개요

OIDC Issuers를 사용하면 외부 서비스가 하드코딩된 자격 증명 없이 Pulumi Cloud 액세스 토큰을 안전하게 획득할 수 있다. 장기 수명의 Pulumi 액세스 토큰을 CI 시스템에 시크릿으로 저장하는 대신, 외부 서비스를 신뢰할 수 있는 OIDC Issuer로 등록하여 단기 수명 토큰을 교환받는다.

### 토큰 교환 흐름

1. 외부 워크로드가 호스트 서비스에서 OIDC id_token 획득
2. 해당 id_token을 Pulumi Cloud에 전달하여 단기 Pulumi 액세스 토큰으로 교환
3. Pulumi 액세스 토큰으로 Pulumi 작업 수행

### 에디션별 사용 가능 토큰 유형

| 에디션 | 사용 가능 토큰 유형 |
|---|---|
| Individual | `personal` |
| Team | `personal`, `organization` |
| Enterprise | `personal`, `organization`, `team` |
| Business Critical | `personal`, `organization`, `team`, `deployment-runner` |

### OIDC Issuer 등록

**Settings > Access Management > OIDC Issuers**에서 등록한다.

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

### CLI를 통한 OIDC 로그인

```bash
# TypeScript (GitHub Actions 예시)
pulumi login --oidc-token <token> --oidc-org <org-name>
```

```python
# Python - OIDC 토큰 파일 경로 사용
# pulumi login --oidc-token file:///path/to/token --oidc-org <org-name>
```

> **주의:** `OIDC token exchange failed` 오류 발생 시 백엔드 URL을 명시적으로 포함해야 한다:
> `pulumi login https://api.pulumi.com --oidc-token <token> --oidc-org <org-name>`

### REST API를 통한 토큰 교환

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

### GitHub Actions OIDC 설정 예시

```yaml
# TypeScript/공통 워크플로우
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

### 지원 OIDC 프로바이더

| 프로바이더 | 문서 경로 |
|---|---|
| GitHub Actions | `/docs/administration/access-identity/oidc-issuers/github/` |
| GitLab CI | `/docs/administration/access-identity/oidc-issuers/gitlab/` |
| Amazon EKS | `/docs/administration/access-identity/oidc-issuers/kubernetes-eks/` |
| Google GKE | `/docs/administration/access-identity/oidc-issuers/kubernetes-gke/` |

---

## 액세스 토큰 관리

### 토큰 유형

| 유형 | 설명 | 가용 에디션 |
|---|---|---|
| Personal token | 생성한 사용자와 동일한 권한. 모든 조직 멤버십 포함 | 모든 에디션 |
| Organization token | 조직 자체로 인증. 감사 로그에 조직으로 기록 | Enterprise, Business Critical |
| Team token | 특정 팀으로 인증. 팀의 역할에 따른 권한 | Enterprise, Business Critical |

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

### AWS KMS 고급 옵션

```bash
# AWS SDK v2 + 프로필 지정
awskms://<key-id>?region=us-east-1&awssdk=v2&profile=dev

# Encryption Context 지정 (CLI v3.41.1+)
awskms://<key-id>?region=us-east-1&awssdk=v2&profile=dev&context_project=myproject&context_environment=staging
```

Encryption Context는 IAM 정책 조건 및 CloudTrail 로그에 표시된다.

### Customer Managed Keys (CMK)

> **참고:** CMK는 Enterprise 및 Business Critical 에디션에서 사용 가능. 현재 Pulumi ESC 데이터 암호화에만 지원되며, AWS KMS만 지원.

CMK를 사용하면 Pulumi Cloud의 데이터 암호화에 자체 KMS 키를 사용할 수 있다.

| 작업 | 설명 |
|---|---|
| 키 추가 | 첫 CMK 추가 시 기존 데이터 키가 자동으로 새 CMK로 재암호화됨 |
| 기본 키 설정 | 모든 신규 데이터 키가 이 키로 암호화됨 |
| 키 비활성화 | 새 데이터 키 생성에 사용 불가. 기존 데이터 키는 재암호화 키 필요 |
| 전체 비활성화 | 모든 CMK 비활성화 후 Pulumi 관리 키로 재암호화 |

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
| REST API | `/docs/reference/service-rest-api#audit-logs` 참조. 간헐적 사용에만 적합 |

### 지원 포맷

| 포맷 | 필드 |
|---|---|
| JSON | `timestamp`, `sourceIP`, `event`, `description`, `user`(login, name, avatar URL) |
| CSV | `Timestamp`, `Name`, `Login`, `Event`, `Description`, `SourceIP`, `RequireOrgAdmin`, `RequireStackAdmin`, `AuthenticationFailure` |
| CEF (Common Event Format) | 표준 헤더 + 확장 필드(`dvchost`, `rt`, `src`, `suser`, `orgID`, `userID`, `requireOrgAdmin`, `requireStackAdmin`, `authenticationFailure`) |

### 주요 감사 로그 이벤트

| 이벤트 | 설명 |
|---|---|
| `User Login` / `User Login Failed` | 사용자 로그인 성공/실패 |
| `Member Added` / `Member Removed` / `Member Role Changed` | 조직 멤버 변경 |
| `Stack Created` / `Stack Deleted` / `Stack Update Started/Completed/Canceled` | 스택 수명주기 |
| `Secret Decrypted` | 시크릿 값 복호화 |
| `Environment Created` / `Environment Open` / `Environment Decrypted` | ESC 환경 작업 |
| `Customer Managed Key Added` / `Set Default` / `Disabled` | CMK 변경 |
| `SAML Configuration Updated` | SAML 설정 변경 |
| `Auth Failure Organization Role` / `Auth Failure Stack Permission` | 권한 부족 인증 실패 |
| `Team Created` / `Team Updated` / `Team Deleted` | 팀 관리 |
| `Policy Pack Created` / `Enabled` / `Disabled` | 정책 팩 관리 |
| `Stack Collaborator Added` / `Permissions Changed` / `Removed` | 스택 협업자 변경 |

---

## SAML SSO 및 SCIM

### SAML SSO

> **참고:** SAML 지원은 Enterprise 및 Business Critical 에디션 필요.

Pulumi Cloud는 SAML 2.0 Identity Provider와 연동할 수 있다.

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
