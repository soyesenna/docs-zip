# Pulumi ESC (Environments, Secrets, and Configuration)

> **원문**
> - https://www.pulumi.com/docs/esc/
> - https://www.pulumi.com/docs/esc/get-started/
> - https://www.pulumi.com/docs/esc/concepts/
> - https://www.pulumi.com/docs/esc/concepts/how-esc-works/
> - https://www.pulumi.com/docs/esc/concepts/providers/
> - https://www.pulumi.com/docs/esc/environments/
> - https://www.pulumi.com/docs/esc/environments/imports/
> - https://www.pulumi.com/docs/esc/environments/importing-environments/
> - https://www.pulumi.com/docs/esc/environments/versioning/
> - https://www.pulumi.com/docs/esc/environments/syntax/
> - https://www.pulumi.com/docs/esc/providers/
> - https://www.pulumi.com/docs/esc/providers/login/
> - https://www.pulumi.com/docs/esc/providers/secrets/
> - https://www.pulumi.com/docs/esc/providers/rotators/
> - https://www.pulumi.com/docs/esc/cli/
> - https://www.pulumi.com/docs/esc/guides/integrate-with-pulumi-iac/
> - https://www.pulumi.com/docs/esc/guides/running-commands/

Pulumi ESC(Environments, Secrets, and Configuration)는 클라우드 인프라와 애플리케이션 전반에 걸쳐 중앙 집중식 시크릿 관리와 구성 오케스트레이션을 제공하는 서비스다. Pulumi Cloud의 관리형 서비스로 제공되며 자체 호스팅도 가능하다. 환경(Environment)이라는 YAML 문서 단위로 시크릿과 설정을 정의하고, OIDC 기반 단기 자격 증명 발급, 외부 시크릿 스토어 통합, 자동 순환(rotator), `esc run` 명령어를 통한 환경 변수 주입 등의 기능을 제공한다.

---

## 핵심 개념

### 환경(Environments)

환경은 ESC의 기본 조직 단위다. YAML 문서로 정의되며, 정적 값·동적 프로바이더 호출·다른 환경의 import를 포함할 수 있다.

| 속성 | 설명 |
| --- | --- |
| **이름 규칙** | 프로젝트 내 고유(예: `my-org/my-project/production`) |
| **버전 관리** | 변경 시마다 새 불변 revision 생성 |
| **태그** | `stable`, `v1.2.3` 등으로 버전 참조 가능(`latest` 태그 기본 제공) |
| **평가 시점** | 환경을 열(open) 때 동적 값과 import가 모두 해석됨 |
| **구성 가능** | 한 환경이 다른 환경을 import하여 상속 및 오버라이드 가능 |

### 소스(Sources)

ESC는 확장 가능한 프로바이더 시스템을 통해 다양한 소스에서 설정과 시크릿을 가져온다.

| 소스 유형 | 설명 |
| --- | --- |
| **정적 값** | 환경 YAML에 직접 정의된 키-값 쌍 |
| **OIDC 프로바이더** | AWS, Azure, GCP, Vault 등의 단기 자격 증명 생성 |
| **시크릿 프로바이더** | AWS Secrets Manager, Azure Key Vault, 1Password 등에서 값 검색 |
| **로그인 프로바이더** | 클라우드 서비스 인증 |

동적 프로바이더는 환경이 정의될 때가 아니라 열릴 때 실행되므로, 자격 증명과 시크릿이 항상 최신 상태로 유지된다.

### 대상(Targets)

ESC는 동일한 환경을 여러 대상에 동시에 출력할 수 있다.

| 대상 | 설명 |
| --- | --- |
| **환경 변수** | `esc run` 명령으로 프로세스에 주입 |
| **구성 API** | TypeScript, Python, Go SDK를 통한 프로그래밍 방식 접근 |
| **Pulumi IaC** | `Pulumi.<stack>.yaml` 파일에서 환경 참조 |
| **CLI 출력** | `esc env open`, `esc env get`으로 값 확인 |
| **Kubernetes** | ESC 오퍼레이터를 통해 시크릿 동기화 |
| **기타 도구** | Terraform, Docker, 개발 도구와 통합 |

### 관리(Management)

Pulumi Cloud에서 중앙 집중식으로 관리된다.

| 기능 | 설명 |
| --- | --- |
| **RBAC** | 조직·팀·개인 수준의 읽기/쓰기/삭제 권한 제어 |
| **감사 로그** | 모든 접근과 수정 이력 추적 |
| **버전 관리** | 모든 변경이 불변 버전으로 기록 |
| **태깅** | 롤백 또는 단계적 배포를 위한 버전 레이블 |
| **암호화** | 저장 시 및 전송 중 모든 시크릿 암호화 |
| **고객 관리 키** | 자체 키로 암호화하는 옵션 제공 |

---

## 환경 정의 구문(Syntax)

ESC 환경은 YAML 문서로, 최상위 키 두 개(`imports`, `values`)를 정의한다.

### 최상위 키

| 키 | 설명 |
| --- | --- |
| `imports` | 다른 환경을 가져오는 리스트. JSON Merge Patch(RFC 7396) 의미론으로 병합 |
| `values` | 환경이 평가될 때 생성할 값을 정의 |

### 기본 예제

```yaml
imports:
  - aws-production
  - stripe-production
values:
  desiredInstanceCount: 8
  aws:region: us-west-2
```

### 보간법(Interpolation)과 참조(References)

참조는 `${property-path}` 형식으로, 다른 값을 재사용할 수 있게 한다.

| 형식 | 설명 | 예시 |
| --- | --- | --- |
| **Bare reference** | 참조된 값의 복사본으로 평가 | `${user}` |
| **Interpolation** | 문자열 내에 참조를 포함 | `Hello, ${user.name}!` |

```yaml
values:
  user:
    name: Real Name
    login: user-login
  user-copy: ${user}             # bare reference
  greeting: Hello, ${user.name}! # interpolation
```

### 내장 함수(Built-in Functions)

모든 함수 호출은 `fn::` 접두사를 가진 단일 키를 가진 객체 형태다.

| 함수 | 설명 |
| --- | --- |
| `fn::secret` | 값을 암호화된 시크릿으로 표시 |
| `fn::open` | 프로바이더를 호출하여 동적 값 검색 |
| `fn::rotate` | 로테이터를 호출하여 자격 증명 교체 |
| `fn::join` | 배열을 문자열로 결합 |
| `fn::split` | 문자열을 배열로 분할 |
| `fn::toJSON` | 값을 JSON 문자열로 직렬화 |
| `fn::fromJSON` | JSON 문자열을 값으로 파싱 |
| `fn::toBase64` | 값을 Base64로 인코딩 |
| `fn::fromBase64` | Base64 문자열을 디코딩 |
| `fn::toString` | 값을 문자열 표현으로 변환 |
| `fn::concat` | 배열을 연결 |
| `fn::final` | 값의 최종 형태를 참조 |
| `fn::validate` | 값을 검증 |

```yaml
values:
  apiKey:
    fn::secret: demo-secret-123
  greeting:
    fn::join: [" ", ["hello", "world"]]
```

### 예약 속성(Reserved Properties)

| 속성 | 소비자 | 설명 |
| --- | --- | --- |
| `environmentVariables` | `esc` CLI | 환경 변수로 노출할 키-값 맵 |
| `pulumiConfig` | `pulumi` CLI | Pulumi IaC Config API로 전달할 키-값 맵 |

---

## 환경 Import

환경은 다른 환경을 import하여 구성을 계층적으로 조직할 수 있다.

### 명시적 Import(Explicit Imports)

`imports` 리스트에 환경 이름을 나열한다. 여러 환경을 import하면 JSON Merge Patch(RFC 7396) 의미론으로 병합되며, 나중에 선언된 값이 이전 값을 덮어쓴다.

```yaml
# myorg/myapp/dev
imports:
  - aws/dev
  - stripe/dev
values:
  greeting: Hello from the myapp/dev environment!
  environmentVariables:
    AWS_ACCESS_KEY_ID: ${aws.login.accessKeyId}
    STRIPE_API_KEY: ${stripe.apiKey}
```

### 암시적 Import(Implicit Imports)

`${environments.PROJECT.ENV.VALUEPATH}` 형식으로 전체 환경을 노출하지 않고 특정 값만 참조한다.

```yaml
values:
  environmentVariables:
    AWS_REGION: ${environments.aws.dev.aws.region}
```

| 기능 | 명시적 Import | 암시적 Import |
| --- | --- | --- |
| **구문** | `imports: [project/env]` | `${environments.project.env.path}` |
| **범위** | 전체 환경을 import | 특정 값만 참조 |
| **병합** | 전체 JSON Merge Patch | 병합 없음 |
| **용도** | 공통 구성 공유 | 전체 구조 노출 없이 선택적 참조 |

---

## 정적 값 vs 동적 값

### 정적 값

YAML에 직접 정의되며 평가 시 한 번만 해석된다.

```yaml
values:
  region: us-west-2
  apiEndpoint: https://api.example.com
```

### 동적 값

환경이 열릴 때 생성되거나 검색된다.

```yaml
values:
  aws:
    login:
      fn::open::aws-login:
        oidc:
          roleArn: arn:aws:iam::123456789012:role/my-role
  environmentVariables:
    AWS_ACCESS_KEY_ID: ${aws.login.accessKeyId}
```

---

## 프로바이더(Providers)

프로바이더와 로테이터는 ESC에 내장된 퍼스트파티 플러그인이다. 별도 설치 없이 환경 YAML에서 직접 호출한다.

### Login Providers (로그인 프로바이더)

`fn::open::<name>-login` 형식으로 호출하며, 단기 자격 증명을 발급한다. OIDC가 지원되는 경우 정적 키보다 OIDC 사용을 권장한다.

| 프로바이더 | 설명 |
| --- | --- |
| `aws-login` | OIDC 또는 정적 자격 증명으로 AWS 로그인 |
| `azure-login` | OIDC 또는 정적 자격 증명으로 Azure 로그인 |
| `gcp-login` | OIDC 또는 정적 자격 증명으로 Google Cloud 로그인 |
| `gh-login` | GitHub App 자격 증명으로 GitHub 로그인 |
| `vault-login` | OIDC 또는 정적 자격 증명으로 HashiCorp Vault 로그인 |
| `doppler-login` | OIDC로 Doppler 로그인 |
| `infisical-login` | OIDC 또는 정적 자격 증명으로 Infisical 로그인 |
| `snowflake-login` | OIDC로 Snowflake 인증 |

### Secrets and Configuration Providers (시크릿/구성 프로바이더)

`fn::open::<name>` 형식으로 호출하며, 외부 시스템에서 값을 동적으로 가져온다.

| 프로바이더 | 설명 |
| --- | --- |
| `1password-secrets` | 1Password에서 시크릿 가져오기 |
| `aws-secrets` | AWS Secrets Manager에서 시크릿 가져오기 |
| `aws-parameter-store` | AWS Systems Manager Parameter Store에서 파라미터 가져오기 |
| `azure-secrets` | Azure Key Vault에서 시크릿 가져오기 |
| `gcp-secrets` | Google Cloud Secret Manager에서 시크릿 가져오기 |
| `vault-secrets` | HashiCorp Vault에서 시크릿 가져오기 |
| `doppler-secrets` | Doppler에서 시크릿 가져오기 |
| `infisical-secrets` | Infisical에서 시크릿 가져오기 |
| `pulumi-stacks` | Pulumi 스택 출력값(Pulumi Cloud에 저장된 Terraform state 포함) 가져오기 |
| `terraform-state` | S3 또는 Terraform Cloud의 Terraform state 파일에서 출력값 가져오기 |
| `external` | 커스텀 서비스 어댑터에서 시크릿 가져오기 |

### Rotators (로테이터)

`fn::rotate::<name>` 형식으로 호출하며, 저장된 자격 증명을 새로 발급된 것으로 교체한다. 수동(`esc env rotate`) 또는 스케줄로 실행할 수 있다.

| 로테이터 | 필수 커넥터 | 설명 |
| --- | --- | --- |
| `aws-iam` | 없음 | AWS IAM 사용자 액세스 자격 증명 교체 |
| `azure-app-secret` | 없음 | Azure 앱 등록 클라이언트 시크릿 교체 |
| `mysql` | `aws-lambda`(사설망만) | MySQL 데이터베이스 사용자 자격 증명 교체 |
| `postgres` | `aws-lambda`(사설망만) | PostgreSQL 데이터베이스 사용자 자격 증명 교체 |
| `password` | 없음 | 비밀번호 생성 규칙으로 임의 키 교체 |
| `passphrase` | 없음 | 암호 가능한 패스프레이즈 생성 규칙으로 임의 키 교체 |
| `snowflake-user` | 없음 | Snowflake 사용자 RSA 키페어 교체 |
| `external` | 없음 | 커스텀 서비스 어댑터로 자격 증명 교체 |

### Provider vs Rotator 차이

| 항목 | Provider | Rotator |
| --- | --- | --- |
| **호출 방식** | `values:` 아래 `fn::open::<name>` | `rotators:` 아래 `fn::rotate::<name>` |
| **상태** | Stateless(항상 새 값 생성) | Stateful(발급한 자격 증명을 환경에 저장) |
| **실행 시점** | 환경 열릴 때마다 | 수동 또는 스케줄 트리거 시 |
| **용도** | 단기 자격 증명 발급, 시크릿 검색 | 저장된 자격 증명의 주기적 교체 |

---

## 주요 Login Provider 상세

### aws-login

| 입력 속성 | 타입 | 설명 |
| --- | --- | --- |
| `oidc.roleArn` | string | Assume할 IAM 역할 ARN |
| `oidc.sessionName` | string | 역할 세션 이름 |
| `oidc.duration` | string | 세션 지속 시간(기본 2시간) |
| `oidc.policyArns` | string[] | 추가 정책 ARN |
| `oidc.subjectAttributes` | string[] | OIDC 토큰에 포함할 subject 속성 |
| `static.accessKeyId` | string | 정적 AWS 액세스 키 ID |
| `static.secretAccessKey` | string | 정적 AWS 시크릿 액세스 키 |
| `static.sessionToken` | string | 정적 세션 토큰(선택) |

```yaml
values:
  aws:
    login:
      fn::open::aws-login:
        oidc:
          duration: 1h
          roleArn: arn:aws:iam::012345678912:role/role-abcd123
          sessionName: pulumi-esc
  environmentVariables:
    AWS_ACCESS_KEY_ID: ${aws.login.accessKeyId}
    AWS_SECRET_ACCESS_KEY: ${aws.login.secretAccessKey}
    AWS_SESSION_TOKEN: ${aws.login.sessionToken}
    AWS_REGION: us-west-2
```

### azure-login

| 입력 속성 | 타입 | 설명 |
| --- | --- | --- |
| `clientId` | string | 클라이언트 ID |
| `tenantId` | string | 테넌트 ID |
| `subscriptionId` | string | 구독 ID |
| `clientSecret` | string | 클라이언트 시크릿(선택) |
| `oidc` | bool | OIDC 사용 여부(기본 `false`) |
| `subjectAttributes` | string[] | OIDC 토큰에 포함할 subject 속성 |

```yaml
values:
  azure:
    login:
      fn::open::azure-login:
        clientId: <YOUR_CLIENT_ID>
        tenantId: <YOUR_TENANT_ID>
        subscriptionId: /subscriptions/<YOUR_SUBSCRIPTION_ID>
        oidc: true
  environmentVariables:
    ARM_USE_OIDC: "true"
    ARM_CLIENT_ID: ${azure.login.clientId}
    ARM_TENANT_ID: ${azure.login.tenantId}
    ARM_SUBSCRIPTION_ID: ${azure.login.subscriptionId}
    ARM_OIDC_TOKEN: ${azure.login.oidc.token}
```

### gcp-login

| 입력 속성 | 타입 | 설명 |
| --- | --- | --- |
| `project` | number | GCP 프로젝트 숫자 ID |
| `oidc.workloadPoolId` | string | 워크로드 풀 ID |
| `oidc.providerId` | string | 아이덴티티 프로바이더 ID |
| `oidc.serviceAccount` | string | 서비스 계정 이메일 |
| `oidc.region` | string | 워크로드 아이덴티티 풀 위치(기본 `global`) |
| `oidc.tokenLifetime` | string | 임시 자격 증명 수명 |
| `accessToken.accessToken` | string | 액세스 토큰(정적 인증 시) |

```yaml
values:
  gcp:
    login:
      fn::open::gcp-login:
        project: <YOUR_PROJECT_NUMBER>
        oidc:
          workloadPoolId: pulumi-esc
          providerId: pulumi-esc
          serviceAccount: pulumi-esc@<YOUR_PROJECT>.iam.gserviceaccount.com
  environmentVariables:
    GOOGLE_CLOUD_PROJECT: ${gcp.login.project}
    GOOGLE_OAUTH_ACCESS_TOKEN: ${gcp.login.accessToken}
```

### gh-login

| 입력 속성 | 타입 | 설명 |
| --- | --- | --- |
| `appId` | number | GitHub App ID |
| `privateKey` | string | GitHub App 개인 키(PEM 형식, `fn::secret` 권장) |
| `owner` | string | 설치 액세스 토큰을 발급할 GitHub 계정 |
| `repositories` | string[] | 접근을 허용할 리포지토리 목록(선택) |
| `permissions` | object | 토큰 권한 맵(선택) |
| `ghe.host` | string | GitHub Enterprise Server 호스트명(선택) |

```yaml
values:
  gh:
    fn::open::gh-login:
      appId: <YOUR_APP_ID>
      privateKey:
        fn::secret: |
          -----BEGIN RSA PRIVATE KEY-----
          <YOUR_PRIVATE_KEY>
          -----END RSA PRIVATE KEY-----
      owner: <YOUR_GITHUB_OWNER>
      permissions:
        contents: read
        pull_requests: write
  environmentVariables:
    GH_TOKEN: ${gh.accessToken}
```

---

## 버전 관리(Versioning)

환경이 변경될 때마다 새 불변 revision이 생성된다.

### 주요 기능

| 기능 | 설명 |
| --- | --- |
| **revision 이력** | `esc env version history <env>`로 확인 |
| **revision 비교** | `esc env diff <env>@3 <env>@2`로 차이 확인 |
| **태그** | `esc env version tag <env>@prod @3`로 revision에 태그 지정 |
| **롤백** | 특정 revision이나 태그로 되돌리기 가능 |
| **Import 핀** | `@tag` 또는 `@revision`으로 특정 버전 고정 |

### 버전 핀 예제

```yaml
# Pulumi.dev.yaml
environment:
  - my-project/common@production
  - my-project/dev@3
```

```yaml
# ESC 환경 정의에서 import 핀
imports:
  - test@prod
```

---

## ESC CLI 주요 명령어

ESC CLI는 독립 실행형 CLI로, Pulumi IaC CLI가 이미 설치된 경우 `pulumi env` 명령으로 대체 가능하다.

### 설치

```bash
# macOS
brew update && brew install pulumi/tap/esc

# Linux
curl -fsSL https://get.pulumi.com/esc/install.sh | sh

# Windows
# https://get.pulumi.com/esc/releases/esc-v0.25.0-windows-x64.zip 다운로드
```

### 주요 명령어

| 명령어 | 설명 |
| --- | --- |
| `esc login` | Pulumi Cloud에 로그인 |
| `esc env init <name>` | 빈 환경 생성 |
| `esc env ls` | 환경 목록 조회 |
| `esc env edit <name>` | 환경 정의 편집 |
| `esc env get <name> <path>` | 환경 내 특정 값 조회 |
| `esc env set <name> <path> <value>` | 환경에 값 설정 |
| `esc env rm <name> [path]` | 환경 또는 값 삭제 |
| `esc env open <name>` | 환경 열기(모든 값 해석 후 JSON 출력) |
| `esc env diff <name>@n <name>@m` | 버전 간 차이 비교 |
| `esc env version history <name>` | revision 이력 조회 |
| `esc env version tag <name>@tag @n` | revision에 태그 지정 |
| `esc env version rollback <name>@n` | 특정 버전으로 롤백 |
| `esc env rotate <name>` | 환경 내 시크릿 순환 실행 |
| `esc run <name> -- <command>` | 환경 변수를 주입하여 명령 실행 |
| `esc version` | ESC 버전 출력 |

### esc run 상세

환경을 열고 `environmentVariables` 블록의 값을 환경 변수로 주입하여 명령을 실행한다.

```bash
# 기본 사용
esc run my-org/my-project/dev -- node app.js

# 대화형 셸
esc run my-org/my-project/dev -- bash

# AWS CLI 명령 실행
esc run my-org/my-project/aws-prod -- aws s3 ls

# 테스트 실행
esc run my-org/my-project/test -- pytest
```

| 옵션 | 설명 |
| --- | --- |
| `-i`, `--interactive` | 대화형 명령으로 처리(출력 필터 비활성화) |
| `-l`, `--lifetime` | 열린 환경의 수명(기본 2시간) |
| `--draft` | draft 환경 열기(change-request-id 지정) |

---

## Pulumi IaC와 통합

### 스택 구성에서 ESC 환경 참조

`Pulumi.<stack-name>.yaml` 파일에 `environment` 블록을 추가한다.

```yaml
# Pulumi.dev.yaml
environment:
  - my-project/dev
```

여러 환경을 참조하면 선언 순서대로 병합된다(나중 값이 이전 값을 덮어씀).

```yaml
environment:
  - my-project/common
  - my-project/dev
```

### ESC 환경에서 Pulumi Config 제공

ESC 환경 정의의 `pulumiConfig` 블록이 Pulumi IaC Config API와 연결된다.

```yaml
values:
  pulumiConfig:
    region: us-west-2
    apiKey:
      fn::secret: demo-secret-123
```

### 코드에서 설정 접근

**TypeScript:**

```typescript
import * as pulumi from "@pulumi/pulumi";

const config = new pulumi.Config();
const region = config.get("region");
const apiKey = config.getSecret("apiKey");

export const Region = region;
export const ApiKey = apiKey;
```

**Python:**

```python
import pulumi

config = pulumi.Config()
region = config.get("region")
api_key = config.get_secret("apiKey")

pulumi.export("Region", region)
pulumi.export("ApiKey", api_key)
```

### 동적 클라우드 자격 증명 공유 패턴

여러 스택에서 AWS OIDC 자격 증명을 공유하는 예:

```yaml
values:
  aws:
    login:
      fn::open::aws-login:
        oidc:
          roleArn: arn:aws:iam::123456789012:role/pulumi-deployment-role
          sessionName: pulumi-session
  pulumiConfig:
    aws:region: ${aws.login.region}
  environmentVariables:
    AWS_ACCESS_KEY_ID: ${aws.login.accessKeyId}
    AWS_SECRET_ACCESS_KEY: ${aws.login.secretAccessKey}
    AWS_SESSION_TOKEN: ${aws.login.sessionToken}
```

### 환경별 구성 패턴

공통 구성을 공유하면서 환경별 오버라이드:

```yaml
# common 환경
values:
  pulumiConfig:
    myApp:instanceType: t3.micro
    myApp:replicas: 1
```

```yaml
# production 환경
imports:
  - common
values:
  pulumiConfig:
    myApp:instanceType: t3.large
    myApp:replicas: 3
```

### 기존 스택 구성을 ESC 환경으로 변환

```bash
pulumi config env init
```

### Automation API 지원

Node.js, Go, Python Automation API에서 환경 관리 메서드를 제공한다.

| 메서드 | 설명 |
| --- | --- |
| `addEnvironments(...)` | 스택의 import 목록에 환경 추가 |
| `listEnvironments()` | 스택에 현재 import된 환경 목록 조회 |
| `removeEnvironment(env)` | 스택의 import 목록에서 특정 환경 제거 |

---

## 보안 고려사항

| 항목 | 설명 |
| --- | --- |
| **시크릿 암호화** | `fn::secret`으로 표시된 값은 저장 시 자동 암호화. Pulumi Cloud 콘솔에서 `[secret]`으로 표시 |
| **단기 자격 증명** | OIDC 기반 Login Provider는 환경 열릴 때마다 새 자격 증명을 발급하므로 유효 기간이 짧음 |
| **환경 변수 격리** | `esc run`으로 주입된 환경 변수는 명령 실행 동안에만 존재하며, 종료 후에는 사용 불가 |
| **출력 필터링** | `esc run`은 기본적으로 시크릿 값을 출력에서 마스킹함. `-i` 플래그 시 필터 비활성화 |
| **RBAC** | 환경별로 읽기/쓰기/삭제 권한을 조직·팀·개인 수준으로 제어 |
| **감사 로그** | 모든 환경 접근과 변경 이력이 기록됨 |
| **고객 관리 키** | 자체 암호화 키 사용 옵션 제공 |

---

## 프로바이더 전체 목록

### Login Providers

| 프로바이더 | 호출 방식 | 지원 인증 | 설명 |
| --- | --- | --- | --- |
| `aws-login` | `fn::open::aws-login` | OIDC, Static | AWS 자격 증명 발급 |
| `azure-login` | `fn::open::azure-login` | OIDC, Static | Azure 자격 증명 발급 |
| `gcp-login` | `fn::open::gcp-login` | OIDC, Static | Google Cloud 자격 증명 발급 |
| `gh-login` | `fn::open::gh-login` | App Credentials | GitHub 설치 액세스 토큰 발급(1시간 만료) |
| `vault-login` | `fn::open::vault-login` | OIDC, Static | HashiCorp Vault 로그인 |
| `doppler-login` | `fn::open::doppler-login` | OIDC | Doppler 로그인 |
| `infisical-login` | `fn::open::infisical-login` | OIDC, Static | Infisical 로그인 |
| `snowflake-login` | `fn::open::snowflake-login` | OIDC | Snowflake 인증 |

### Secrets and Configuration Providers

| 프로바이더 | 호출 방식 | 설명 |
| --- | --- | --- |
| `1password-secrets` | `fn::open::1password-secrets` | 1Password 시크릿 가져오기 |
| `aws-secrets` | `fn::open::aws-secrets` | AWS Secrets Manager 시크릿 가져오기 |
| `aws-parameter-store` | `fn::open::aws-parameter-store` | AWS SSM Parameter Store 파라미터 가져오기 |
| `azure-secrets` | `fn::open::azure-secrets` | Azure Key Vault 시크릿 가져오기 |
| `gcp-secrets` | `fn::open::gcp-secrets` | Google Cloud Secret Manager 시크릿 가져오기 |
| `vault-secrets` | `fn::open::vault-secrets` | HashiCorp Vault 시크릿 가져오기 |
| `doppler-secrets` | `fn::open::doppler-secrets` | Doppler 시크릿 가져오기 |
| `infisical-secrets` | `fn::open::infisical-secrets` | Infisical 시크릿 가져오기 |
| `pulumi-stacks` | `fn::open::pulumi-stacks` | Pulumi 스택 출력값 가져오기 |
| `terraform-state` | `fn::open::terraform-state` | Terraform state 파일 출력값 가져오기 |
| `external` | `fn::open::external` | 커스텀 서비스 어댑터 |

### Rotators

| 로테이터 | 호출 방식 | 필수 커넥터 | 설명 |
| --- | --- | --- | --- |
| `aws-iam` | `fn::rotate::aws-iam` | 없음 | AWS IAM 사용자 자격 증명 교체 |
| `azure-app-secret` | `fn::rotate::azure-app-secret` | 없음 | Azure 앱 등록 클라이언트 시크릿 교체 |
| `mysql` | `fn::rotate::mysql` | `aws-lambda`(사설망) | MySQL 사용자 자격 증명 교체 |
| `postgres` | `fn::rotate::postgres` | `aws-lambda`(사설망) | PostgreSQL 사용자 자격 증명 교체 |
| `password` | `fn::rotate::password` | 없음 | 비밀번호 규칙 기반 자격 증명 교체 |
| `passphrase` | `fn::rotate::passphrase` | 없음 | 패스프레이즈 규칙 기반 자격 증명 교체 |
| `snowflake-user` | `fn::rotate::snowflake-user` | 없음 | Snowflake 사용자 RSA 키페어 교체 |
| `external` | `fn::rotate::external` | 없음 | 커스텀 서비스 어댑터 교체 |

---

## 언어 및 SDK

| 언어 | 문서 |
| --- | --- |
| Node.js (TypeScript/JavaScript) | `/docs/esc/languages-sdks/javascript/` |
| Python | `/docs/esc/languages-sdks/python/` |
| Go | `/docs/esc/languages-sdks/go/` |
| .NET | `/docs/esc/languages-sdks/dotnet/` |

SDK를 사용하면 애플리케이션 런타임에서 환경 값을 검색하고, 프로그래밍 방식으로 환경을 관리할 수 있다.
