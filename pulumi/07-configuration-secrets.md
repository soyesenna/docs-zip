# Pulumi Configuration & Secrets

> https://www.pulumi.com/docs/iac/concepts/config/
> https://www.pulumi.com/docs/iac/concepts/secrets/

Pulumi는 스택별로 다른 설정값을 관리하기 위한 **Configuration** 시스템과, 민감한 데이터를 암호화하여 보호하는 **Secrets** 관리 기능을 제공한다. 설정값은 CLI 명령어(`pulumi config set/get`)와 프로그래밍 모델(`pulumi.Config`) 모두에서 접근할 수 있으며, 시크릿은 `--secret` 플래그 또는 코드 내 `requireSecret` 계열 API로 암호화된다. 모든 설정은 `Pulumi.<stack-name>.yaml` 스택 설정 파일에 저장되며 버전 관리에 커밋하는 것이 권장된다.

---

## Configuration 기본

### 설정 키 네임스페이스

설정 키는 `[<namespace>:]<key-name>` 형식을 사용한다. 콜론(`:`)이 없는 단순 이름을 사용하면 Pulumi가 자동으로 `Pulumi.yaml`의 프로젝트 이름을 네임스페이스로 사용한다.

| 형식 | 예시 | 설명 |
|---|---|---|
| `<namespace>:<key>` | `aws:region` | 명시적 네임스페이스 (프로바이더 등) |
| `<key>` | `name` | 프로젝트 이름이 자동으로 네임스페이스로 사용됨 |

이 네임스페이스 체계 덕분에 AWS 패키지의 `aws:region`과 다른 패키지의 `region` 키가 충돌하지 않는다.

---

## 설정 명령어

### CLI 명령어 표

| 명령어 | 설명 |
|---|---|
| `pulumi config set <key> [value]` | 설정 키-값 쌍 저장 (기존 값 덮어쓰기, 경고 없음) |
| `pulumi config get <key>` | 설정 값 조회 |
| `pulumi config` | 현재 스택의 모든 설정 나열 (`--json` 시 JSON 출력) |
| `pulumi config set --secret <key> [value]` | 값을 암호화하여 저장 |
| `pulumi config set --plaintext <key> [value]` | 값을 평문으로 명시적 저장 |
| `pulumi config set --path '<key.subkey>' [value]` | 구조화된 설정(객체/배열) 값 저장 |
| `pulumi config set --path --secret '<key.subkey>' [value]` | 구조화된 설정 내 특정 값을 시크릿으로 저장 |
| `pulumi config refresh` | 가장 최근 배포된 설정으로 스택 설정 파일 재구성 |

### 기본 사용 예시

```bash
# AWS 리전 설정 (명시적 네임스페이스)
$ pulumi config set aws:region us-west-2
$ pulumi config get aws:region
us-west-2

# 프로젝트 설정 (기본 네임스페이스, 프로젝트 이름이 broome-proj라면)
$ pulumi config set name BroomeLLC
$ pulumi config get name
BroomeLLC

# 값을 생략하면 인터랙티브 프롬프트로 입력
$ pulumi config set name

# 표준 입력으로 값 전달 (멀티라인, 이스케이프 필요 시 유용)
$ cat my_key.pub | pulumi config set publicKey
```

### `pulumi new`에서 설정 전달

```bash
# 단일 키-값 쌍
$ pulumi new template-name --config="key=value"

# 여러 키-값 쌍
$ pulumi new template-name --config="key1=value1" --config="key2=value2"

# AWS 리전 지정 예시
$ pulumi new aws-typescript --config="aws:region=us-west-2"
```

---

## 코드에서 설정 접근

### Config 객체 기본

`pulumi.Config` 객체를 생성하여 설정값에 접근한다. 생성자에 네임스페이스를 전달하지 않으면 현재 프로젝트 이름이 사용된다.

| API | TypeScript | Python | 동작 |
|---|---|---|---|
| 값 조회 (선택) | `config.get("key")` | `config.get("key")` | 값이 없으면 `undefined`/`None` 반환 |
| 값 조회 (필수) | `config.require("key")` | `config.require("key")` | 값이 없으면 예외 발생, 배포 중단 |
| 시크릿 조회 (선택) | `config.getSecret("key")` | `config.get_secret("key")` | `Output`으로 래핑된 시크릿 반환 |
| 시크릿 조회 (필수) | `config.requireSecret("key")` | `config.require_secret("key")` | 필수 시크릿, 없으면 예외 발생 |
| 정수 조회 | `config.getNumber("key")` | `config.get_int("key")` | 정수형으로 반환 |
| 객체 조회 (선택) | `config.getObject<T>("key")` | `config.get_object("key")` | 구조화된 설정을 객체로 반환 |
| 객체 조회 (필수) | `config.requireObject<T>("key")` | `config.require_object("key")` | 필수 객체, 없으면 예외 발생 |

### 프로젝트 기본 네임스페이스로 접근

```typescript
// TypeScript
let config = new pulumi.Config();
let name = config.require("name");
let lucky = config.getNumber("lucky") || 42;
let secret = config.requireSecret("secret");
```

```python
# Python
config = pulumi.Config()
name = config.require("name")
lucky = config.get_int("lucky") or 42
secret = config.require_secret("secret")
```

### 특정 네임스페이스로 접근 (프로바이더 등)

```typescript
// TypeScript - AWS 프로바이더 설정 읽기
let awsConfig = new pulumi.Config("aws");
let awsRegion = awsConfig.require("region");
```

```python
# Python - AWS 프로바이더 설정 읽기
aws_config = pulumi.Config("aws")
aws_region = aws_config.require("region")
```

> **참고:** 설정값은 프로그램 실행 중 **읽기만** 가능하다. 프로그래밍 방식으로 설정값을 쓰려면 Automation API를 사용해야 한다.

---

## 구조화된 설정 (Structured Configuration)

`--path` 플래그를 사용하면 객체와 배열 형태의 구조화된 설정을 저장할 수 있다.

```bash
$ pulumi config set --path 'data.active' true
$ pulumi config set --path 'data.nums[0]' 1
$ pulumi config set --path 'data.nums[1]' 2
$ pulumi config set --path 'data.nums[2]' 3
```

### 스택 설정 파일 결과 (`Pulumi.<stack-name>.yaml`)

```yaml
config:
  proj:data:
    active: true
    nums:
    - 1
    - 2
    - 3
```

`true`/`false`는 boolean으로, 정수 변환 가능한 값은 정수로 저장된다.

### 코드에서 구조화된 설정 접근

```typescript
// TypeScript
interface Data {
    active: boolean;
    nums: number[];
}

let config = new pulumi.Config();
let data = config.requireObject<Data>("data");
console.log(`Active: ${data.active}`);
```

```python
# Python
config = pulumi.Config()
data = config.require_object("data")
print("Active:", data.get("active"))
```

### 중첩 값 접근

`requireObject`가 반환하는 것은 일반 객체이며 `Config` 인스턴스가 아니다. 따라서 `config.require("api").require("endpoint")` 같은 체이닝은 작동하지 않고, 일반 객체 속성 접근을 사용해야 한다.

```typescript
// TypeScript
interface ApiConfig {
    endpoint: string;
    timeout: number;
    headers: { authorization: string; "content-type": string; };
}

const config = new pulumi.Config();
const apiConfig = config.requireObject<ApiConfig>("api");
const endpoint = apiConfig.endpoint;              // "https://api.example.com"
const authHeader = apiConfig.headers.authorization; // "Bearer token123"
```

```python
# Python
config = pulumi.Config()
api_config = config.require_object("api")
endpoint = api_config["endpoint"]              # "https://api.example.com"
auth_header = api_config["headers"]["authorization"]  # "Bearer token123"
```

---

## 프로젝트 수준 설정 (Project Level Configuration)

여러 스택에서 공통으로 사용하는 설정값은 `Pulumi.yaml` 파일의 `config` 섹션에 프로젝트 수준으로 정의할 수 있다.

### 설정 방법

현재 `pulumi config set` 명령어는 프로젝트 수준 설정을 지원하지 않으므로, `Pulumi.yaml` 파일에 직접 입력해야 한다. 프로젝트 수준 설정은 **평문(clear text)만** 지원한다.

> **중요:** 스택 수준 파일과 프로젝트 수준 파일은 구조화된 설정의 문법이 다르다.

| 파일 | 구조화된 설정 문법 |
|---|---|
| **프로젝트 수준** (`Pulumi.yaml`) | `key:` 아래에 `value:` 래퍼 사용. 프로젝트 이름 접두사 없음 |
| **스택 수준** (`Pulumi.<stack>.yaml`) | `projectname:key:` 형식. `value:` 래퍼 없이 직접 중첩 |

```yaml
# Pulumi.yaml (프로젝트 수준)
config:
  aws:region: us-east-1
  name: BroomeLLC
  data:
    value:              # value: 래퍼 필요
      active: true
      nums:
      - 10
      - 20
      - 30
```

```yaml
# Pulumi.dev.yaml (스택 수준, 프로젝트 이름이 myproject인 경우)
config:
  aws:region: us-east-1
  myproject:name: BroomeLLC
  myproject:data:               # 프로젝트 이름 접두사, value: 래퍼 없음
    active: true
    nums:
    - 10
    - 20
    - 30
```

### 우선순위

스택 수준 설정이 프로젝트 수준 설정보다 **우선**한다. 같은 키가 양쪽에 있으면 스택 수준 값이 사용된다.

### 강타입 설정 스키마 정의

`Pulumi.yaml`에서 설정값의 타입, 설명, 기본값을 지정할 수 있다. `pulumi preview` 실행 시 타입 불일치가 감지되면 에러가 발생한다.

```yaml
# Pulumi.yaml
config:
    name:
        type: string
        description: Base name to use for resources.
        default: BroomeLLC
    subnets:
        type: array
        description: Array of subnets to create.
        items:
            type: string
```

현재 설정 스펙은 구조화된 설정(structured configuration)에는 지원되지 않는다.

---

## Pulumi 전용 설정 키

### `pulumi:disable-default-providers`

기본 프로바이더 생성을 비활성화할 패키지 목록. `*`를 사용하면 모든 패키지의 기본 프로바이더를 비활성화한다.

```yaml
config:
  pulumi:disable-default-providers:
    - aws
    - kubernetes
```

### `pulumi:tags`

`pulumi up` 또는 `pulumi refresh` 시 스택에 자동 적용할 태그 목록. config에 있는 태그만 생성/수정되며, config에서 제거한 태그는 Pulumi Cloud에서 수동으로 삭제해야 한다.

```yaml
config:
  pulumi:tags:
    company: "Some LLC"
    team: Ops
```

---

## Secrets (시크릿 관리)

모든 리소스 입력/출력 값은 스택 상태(state)로 기록되어 Pulumi Cloud, 상태 파일, 또는 자체 백엔드에 저장된다. Pulumi Cloud는 상태 파일 전체를 안전하게 전송하고 저장하지만, 추가 보호를 위해 개별 값을 **시크릿**으로 암호화할 수도 있다.

> **참고:** Pulumi CLI는 클라우드 자격 증명을 Pulumi Cloud로 전송하지 않는다.

### 시크릿 생성 방법

| 방법 | TypeScript | Python | 설명 |
|---|---|---|---|
| 설정에서 읽기 | `config.getSecret("key")` | `config.get_secret("key")` | 설정값을 시크릿 Output으로 반환 |
| 설정에서 필수 읽기 | `config.requireSecret("key")` | `config.require_secret("key")` | 필수 시크릿, 없으면 예외 |
| 값에서 생성 | `pulumi.secret(value)` | `Output.secret(value)` | 기존 값을 시크릿으로 래핑 |

### `--secret` 플래그로 시크릿 설정

```bash
# 시크릿으로 설정
$ pulumi config set --secret dbPassword S3cr37

# 확인: 평문 값이 출력되지 않음
$ pulumi config
KEY                        VALUE
aws:region                 us-west-1
dbPassword                 [secret]
```

프로그램에서 출력해도 마스킹된다.

```typescript
// TypeScript
const config = new pulumi.Config();
console.log(`Password: ${config.require("dbPassword")}`);
// 출력: Password: [secret]
```

```python
# Python
config = pulumi.Config()
print('Password: {}'.format(config.require('dbPassword')))
# 출력: Password: [secret]
```

> 시크릿 값에 `$`, `!`, `@`, `#` 등 특수문자가 포함된 경우 셸 해석에 주의해야 한다. 따옴표 사용, 이스케이프, 또는 파일 리다이렉션을 활용할 것.

### `--plaintext` 플래그

Pulumi CLI는 API 토큰이나 비밀번호처럼 보이는 문자열을 감지하면 자동으로 시크릿으로 처리하려 시도한다. 의도적으로 평문으로 저장하려면 `--plaintext`를 사용한다.

```bash
$ pulumi config set --plaintext aws:region us-west-2
```

### 구조화된 설정 내 시크릿

```bash
$ pulumi config set --path endpoints[0].url https://example.com
$ pulumi config set --path --secret endpoints[0].token accesstokenvalue
```

스택 설정 파일 결과:

```yaml
config:
  project-name:endpoints:
    - token:
        secure: AAABALsgfFnV0KbGLybu5f+oTqUFmPl2l7oer5EACw15g7rE6GMYQqGgMoZ07QgT
      url: https://example.com
```

> **경고:** 시크릿 값은 런타임에 복호화되어 평문으로 프로그램에 제공된다. 일반 `get`/`require`로도 읽을 수 있지만, 시크릿 변형 API(`getSecret`/`requireSecret`)를 사용해야 전이적(transitive) 사용도 시크릿으로 유지된다.

---

## 시크릿과 Output의 관계

시크릿은 `Output<T>` 타입을 가지며, 일반 Output과 동일하지만 내부적으로 암호화 필요 플래그가 설정되어 있다. `apply`나 `Output.all`로 시크릿 Output을 조합하면 결과도 자동으로 시크릿이 된다.

Output이 시크릿으로 마킹되는 경우:

| 방법 | 설명 |
|---|---|
| `Config.getSecret` / `requireSecret` | 설정에서 시크릿 읽기 |
| `pulumi.secret(value)` / `Output.secret(value)` | 값에서 시크릿 생성 |
| `additionalSecretOutputs` | 리소스 출력 속성을 시크릿으로 마킹 |
| `apply` / `Output.all`과 시크릿 결합 | 파생된 출력도 자동 시크릿 |

> **주의:** `apply` 콜백 내에서 평문 시크릿 값이 전달된다. Pulumi는 반환 값을 시크릿으로 마킹하지만, 콜백 자체가 값을 노출하지 않도록 프로그램이 보장해야 한다.

---

## 시크릿 암호화 프로바이더

### 기본 동작

Pulumi Cloud는 스택별 암호화 키를 자동으로 관리한다. `--secret` 또는 코드 내 시크릿 래핑 사용 시, CLI와 Pulumi Cloud 간에 보안 프로토콜이 사용되어 전송 중, 저장 중, 물리적 저장소 모두에서 암호화된다.

### 대체 암호화 프로바이더가 필요한 경우

1. Pulumi Cloud 없이 로컬 모드 또는 S3/Blob Storage 등 DIY 백엔드를 사용할 때
2. 팀에서 이미 특정 클라우드 암호화 프로바이더를 사용 중일 때

### 스택 초기화 시 암호화 프로바이더 지정

```bash
$ pulumi stack init <name> --secrets-provider="<provider>://<key>"
```

### 지원 프로바이더

| 프로바이더 | 형식 | 설명 |
|---|---|---|
| `awskms` | `awskms://<key-id>?region=<region>` | AWS Key Management Service |
| `azurekeyvault` | `azurekeyvault://<vault>.vault.azure.net/keys/<key>` | Azure Key Vault |
| `gcpkms` | `gcpkms://projects/<proj>/locations/<loc>/keyRings/<ring>/cryptoKeys/<key>` | Google Cloud KMS |
| `hashivault` | `hashivault://<key-name>` | HashiCorp Vault Transit |

### AWS KMS

키를 세 가지 방식으로 지정할 수 있다.

| 지정 방식 | 형식 |
|---|---|
| 키 ID | `awskms://1234abcd-12ab-34cd-56ef-1234567890ab?region=us-east-1` |
| 별칭 | `awskms://alias/ExampleAlias?region=us-east-1` |
| ARN | `awskms:///arn:aws:kms:us-east-1:111122223333:key/1234abcd-...?region=us-east-1` |

```bash
$ pulumi stack init my-stack \
    --secrets-provider="awskms://1234abcd-12ab-34cd-56ef-1234567890ab?region=us-east-1"
```

Pulumi CLI v3.33.1 이상에서는 `awssdk=v2` 및 `profile=<name>`을 쿼리 문자열에 추가할 수 있다. v3.41.1 이상에서는 `context_<key>=<value>`로 암호화 컨텍스트를 설정할 수 있다.

```bash
# 프로필 및 암호화 컨텍스트 포함
awskms://1234abcd-12ab-34cd-56ef-1234567890ab?region=us-east-1&awssdk=v2&profile=dev&context_project=myproject&context_environment=staging
```

### Azure Key Vault

Azure Key 객체 식별자를 사용한다.

```bash
$ pulumi stack init my-stack \
    --secrets-provider="azurekeyvault://acmecorpsec.vault.azure.net/keys/payroll"
```

인증은 `DefaultAzureCredential`의 여러 방식을 자동 시도한다.

### Google Cloud KMS

키 리소스 ID를 사용한다. 키의 용도(purpose)는 `ENCRYPT_DECRYPT`여야 한다.

```bash
$ pulumi stack init my-stack \
    --secrets-provider="gcpkms://projects/acmecorpsec/locations/us-west1/keyRings/prod/cryptoKeys/payroll"
```

Google Cloud Application Default Credentials를 사용한다.

### HashiCorp Vault

Vault Transit Secrets Engine의 키 이름만 전달하면 된다. 서버 주소와 토큰은 환경 변수로 제공한다.

```bash
$ pulumi stack init my-stack \
    --secrets-provider="hashivault://payroll"
```

| 환경 변수 | 설명 |
|---|---|
| `VAULT_SERVER_URL` | Vault 서버 엔드포인트 |
| `VAULT_SERVER_TOKEN` | 인증 토큰 |

---

## 시크릿 프로바이더 변경

기존 스택의 시크릿 프로바이더를 변경하려면 `pulumi stack change-secrets-provider` 명령어를 사용한다.

```bash
$ pulumi stack change-secrets-provider "<secrets-provider>"
```

지원 프로바이더: `default`, `passphrase`, `awskms`, `azurekeyvault`, `gcpkms`, `hashivault`

변경 후 `pulumi preview`를 실행하여 변경 사항이 없음을 확인할 수 있다.

---

## 스택 설정 파일 구조

### 파일 형식

스택 설정 파일은 자동으로 `Pulumi.<stack-name>.yaml`이라는 이름으로 생성된다.

```yaml
# Pulumi.dev.yaml
config:
  myStack:somePlainTextItem: somePlainText
  myStack:someSecretItem:
    secure: AAABAIIlW0ewSuZ1FJxw/+Rpw6BNqTUvGJ30O8WkpL2hB4aPyS7UU68=
```

평문 값은 문자열 그대로 저장되고, 시크릿 값은 `secure:` 아래에 암호문(ciphertext)으로 저장된다.

### 버전 관리 권장

시크릿이 포함된 스택 설정 파일은 버전 관리에 커밋하는 것이 안전하고 권장되는 방법이다. 암호문을 복호화하려면 원래 암호화에 사용된 키가 필요하며:

- **Pulumi Cloud 관리형:** 스택 읽기 권한이 있는 사용자만 키를 자동 획득
- **DIY 백엔드:** passphrase 또는 암호화 프로바이더 인증 필요

`encryptionSalt`(passphrase 프로바이더)나 `encryptedKey`(다른 프로바이더)도 함께 커밋해야 한다.

---

## 프로바이더 설정 방식

프로바이더 설정은 세 가지 방식으로 구성할 수 있다.

| 방식 | 예시 | 비고 |
|---|---|---|
| 스택 설정 파일 | `pulumi config set aws:region us-west-2` | 기본 프로바이더에만 적용됨 |
| 환경 변수 | `AWS_REGION=us-west-2` | 프로바이더별 환경 변수 |
| SDK 생성자 인수 | `new aws.Provider("my-provider", {region: "us-west-2"})` | 명시적 프로바이더 객체 |

> 스택 설정 파일의 값은 **기본 프로바이더**에서만 사용된다. 프로바이더 객체를 직접 생성하면 스택 설정 파일의 값을 읽지 않는다. 우선순위는 프로바이더마다 다를 수 있으므로 해당 프로바이더 문서를 참조할 것.

---

## Pulumi ESC를 통한 설정/시크릿 관리

Pulumi ESC(Environments, Secrets, and Configuration)를 사용하면 여러 스택 설정 파일에서 중복되는 공통 설정과 시크릿을 중앙에서 관리할 수 있다.

```yaml
# Pulumi.<stack>.yaml
environment:
  - test        # ESC 환경 가져오기
config:
    # 일반 Pulumi 설정
```

ESC 환경은 AWS Secrets Manager, Azure Key Vault, GCP Secret Manager 등 다양한 시크릿 매니저를 단일 진입점으로 통합할 수 있으며, RBAC 및 감사 제어를 통한 보안을 제공한다.
