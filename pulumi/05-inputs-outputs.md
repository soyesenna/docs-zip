# Pulumi Inputs & Outputs

> **원문**
> - https://www.pulumi.com/docs/iac/concepts/inputs-outputs/
> - https://www.pulumi.com/docs/iac/concepts/inputs-outputs/apply/
> - https://www.pulumi.com/docs/iac/concepts/inputs-outputs/all/
> - https://www.pulumi.com/docs/iac/concepts/inputs-outputs/helpers/
> - https://www.pulumi.com/docs/iac/concepts/functions/

Pulumi 리소스는 속성을 정의하기 위해 **Input**과 **Output**이라는 특수 타입을 사용한다. 이 타입들은 일반 값(string, integer 등)을 래핑하며, Pulumi가 인프라 리소스를 **선언적**으로 관리할 수 있게 한다. Input은 리소스에 전달하는 값이고, Output은 리소스가 프로비저닝된 후에야 알 수 있는 값이다. 이 둘을 결합해 Pulumi는 리소스 간 의존성을 자동 추적하고 올바른 순서로 생성·수정·삭제를 수행한다.

---

## Input과 Output이란

### Input

Input은 리소스에 제공하는 값이다. 필수(required)이거나 선택사항(optional)일 수 있다. 예를 들어 `aws.ec2.Subnet`의 `vpcId`는 필수 Input이고, `aws.s3.Bucket`의 `forceDestroy`는 선택사항으로 기본값이 `false`다.

Input을 지정할 때는 항상 **평문(plain) 타입**을 사용할 수 있다. 예를 들어 `pulumi.Input<string>`으로 정의된 Input에 일반 `string` 값을 그대로 전달해도 된다.

```typescript
const key = new tls.PrivateKey("my-private-key", {
    algorithm: "ECDSA", // ECDSA는 평문 값
});
```

```python
key = tls.PrivateKey("my-private-key",
    algorithm="ECDSA", # ECDSA는 평문 값
)
```

### Output

Output은 리소스가 **생성된 후에야 알 수 있는 값**이다. 예를 들어 `aws.ec2.Vpc` 리소스의 VPC ID는 Output이다. 사용자가 이 값을 선택할 수 없으며, AWS에서 VPC가 생성된 후에야 알 수 있다.

Output은 Promise/Future와 유사하다. 프로비저닝이 완료되기 전에는 값을 알 수 없는 **비동기 값**을 나타낸다. 따라서 `console.log()`, `print()` 등으로 직접 출력할 수 없으며, Pulumi SDK에서 제공하는 메서드를 사용해야 한다.

> **참고**: 여기서 다루는 Output은 리소스의 출력 속성이다. 스택 출력(stack output)과는 다른 개념이며, 스택 출력은 `pulumi stack output` 명령이나 Stack Reference를 통해 프로그램 외부에서 사용하기 위한 값이다.

---

## 의존성 추적

Pulumi 프로그램에서는 한 리소스의 Output을 다른 리소스의 Input으로 전달하는 경우가 많다. Pulumi는 이 관계를 통해 **자동으로 의존성을 추적**한다.

```typescript
const password = new random.RandomPassword("password", {
    length: 16,
    special: true,
    overrideSpecial: "!#$%&*()-_=+[]{}<>:?",
});
const example = new aws.rds.Instance("example", {
    instanceClass: "db.t3.micro",
    allocatedStorage: 64,
    engine: "mysql",
    username: "someone",
    password: password.result, // Output을 Input으로 전달
});
```

```python
password = random.RandomPassword(
    "password",
    length=16,
    special=True,
    override_special="!#$%&*()-_=+[]{}<>:?"
)
example = aws.rds.Instance(
    "example",
    instance_class="db.t3.micro",
    allocated_storage=64,
    engine="mysql",
    username="someone",
    password=password.result, # Output을 Input으로 전달
)
```

Pulumi가 관리하는 동작:

| 상황 | Pulumi 동작 |
|---|---|
| `pulumi up` (최초 실행) | VPC가 생성되고 ID가 확정될 때까지 Subnet 생성 대기 |
| `pulumi up` (추가 리소스) | VPC ID가 이미 state에 저장되어 있으므로 즉시 생성 |
| `pulumi destroy` | 모든 Subnet이 삭제된 후에야 VPC 삭제 |

Output-to-Input 관계로 자동 추적되지 않는 의존성이 있으면 `dependsOn` 리소스 옵션을 사용해 명시적으로 정의할 수 있다.

---

## Input\<T\>와 Output\<T\> 타입

Input과 Output 타입은 각 언어 SDK에 정의된다.

| 언어 | Input 타입 | Output 타입 |
|---|---|---|
| TypeScript | `pulumi.Input<T>` | `pulumi.Output<T>` |
| Python | `pulumi.Input[T]` | `pulumi.Output[T]` |
| Go | `pulumi.XxxInput` 인터페이스 | `pulumi.XxxOutput` |
| C# | `Input<T>` | `Output<T>` |
| Java | `Input<T>` | `Output<T>` |

### Input\<T\>에 평문 값 전달

`pulumi.Input<string>`에는 일반 `string`을, `pulumi.Input<number>`에는 일반 `number`를 그대로 전달할 수 있다. Pulumi가 자동으로 래핑한다.

---

## apply()로 단일 Output 값 추출

`apply` 메서드는 **단일 Output**의 평문 값에 접근해 연산을 수행한다. Output이 비동기 값이므로 값이 확정될 때까지 대기한 뒤 콜백 함수를 실행한다.

### 주요 용도

- Output 값 디버그 출력
- 복합 타입의 중첩 값 접근
- Output 값을 다른 값으로 변환
- `Input<T>`를 `Output<T>`로 변환한 뒤 `apply` 호출

> **경고**: `apply` 내부에서 리소스를 생성하지 마라. `apply` 안에서 생성된 리소스는 Output 값이 이미 알려진 상태가 아니면 `pulumi preview`에 나타나지 않는다. 대신 Output을 직접 리소스의 Input으로 전달하면 Pulumi가 의존성을 자동으로 추적한다.

> **경고**: `apply` 내부에서 스택 출력(`export`, `pulumi.export()` 등)을 생성할 수 없다. Output에 의존하는 값을 내보내려면 Output을 직접 export하면 된다.

### 기본 사용법

```typescript
// EC2 인스턴스의 DNS 이름으로 HTTPS URL 생성
const url = server.publicDns.apply(dnsName => `https://${dnsName}`);
export const instanceUrl = url;
```

```python
# EC2 인스턴스의 DNS 이름으로 HTTPS URL 생성
url = server.public_dns.apply(
    lambda dns_name: "https://" + dns_name
)
pulumi.export("instanceUrl", url)
```

`apply`의 반환값은 새로운 `Output<T>`다. 원본 Output의 의존성도 그대로 유지된다.

### 출력값 디버깅

```typescript
vpc.vpcId.apply(id => console.log(`VPC ID: ${id}`));
```

```python
vpc.vpc_id.apply(lambda id: print('VPC ID:', id))
```

### 중첩 Output 값 접근과 Lifting

Output이 배열이나 객체인 경우, `apply`로 중첩 값에 접근할 수 있다.

```typescript
// apply로 중첩 값 접근
const record = new aws.route53.Record("certValidation", {
    name: cert.domainValidationOptions.apply(
        opts => opts[0].resourceRecordName),
    type: cert.domainValidationOptions.apply(
        opts => opts[0].resourceRecordType),
    zoneId: zone.zoneId,
    ttl: 60,
    records: [cert.domainValidationOptions.apply(
        opts => opts[0].resourceRecordValue)],
});
```

```python
# apply로 중첩 값 접근
cert_validation = aws.route53.Record("certValidation",
    name=certificate.domain_validation_options.apply(
        lambda opts: opts[0].resource_record_name
    ),
    type=certificate.domain_validation_options.apply(
        lambda opts: opts[0].resource_record_type
    ),
    zone_id=zone.zone_id,
    ttl=60,
    records=[certificate.domain_validation_options.apply(
        lambda opts: opts[0].resource_record_value
    )],
)
```

**Lifting**을 사용하면 `apply` 없이 직접 프로퍼티나 배열 요소에 접근할 수 있다. Pulumi 타입 시스템이 자동으로 접근을 Output 컨텍스트로 "끌어올린다".

```typescript
// Lifting으로 단순화
const record = new aws.route53.Record("certValidation", {
    name: cert.domainValidationOptions[0].resourceRecordName,
    type: cert.domainValidationOptions[0].resourceRecordType,
    zoneId: zone.zoneId,
    ttl: 60,
    records: [cert.domainValidationOptions[0].resourceRecordValue],
});
```

```python
# Lifting으로 단순화
cert_validation = aws.route53.Record("certValidation",
    name=certificate.domain_validation_options[0].resource_record_name,
    type=certificate.domain_validation_options[0].resource_record_type,
    zone_id=zone.zone_id,
    ttl=60,
    records=[certificate.domain_validation_options[0].resource_record_value],
)
```

> **참고**: Python에서 Lifting은 `__getattr__` 오버라이드로 구현된다. 따라서 리소스 Output에 `hasattr`를 사용하면 예상과 다르게 동작한다.

### apply 내부 에러 처리

`apply` 콜백에서 발생한 미처리 에러는 배포 실패로 이어진다. 에러를 우아하게 처리하려면 콜백 내부에서 try/catch를 수행하고 대체 값을 반환한다.

```typescript
const validated = name.apply(n => {
    if (!n.includes("a")) {
        throw new Error(`name "${n}" must contain the letter 'a'`);
    }
    return n;
});
```

```python
def validate(n: str) -> str:
    if "a" not in n:
        raise Exception(f'name "{n}" must contain the letter \'a\'')
    return n

validated = name.apply(validate)
```

---

## all()로 여러 Output 조합

`all` 함수는 여러 Output 값을 동시에 접근할 때 사용한다. 모든 Output 값이 확정될 때까지 대기한 뒤 평문 값으로 콜백에 전달한다. `all`의 결과 역시 `Output<T>`다.

> **경고**: `all` 콜백 내부에서도 리소스를 생성하지 마라. `pulumi preview`에서 누락될 수 있다. 대신 Output을 직접 리소스 Input으로 전달하라.

### 문자열 생성

```typescript
let connectionString = pulumi.all([sqlServer.name, database.name])
    .apply(([server, db]) =>
        `Server=tcp:${server}.database.windows.net;initial catalog=${db};`);
```

```python
from pulumi import Output

# 위치 인자를 사용하는 방식
connection_string = Output.all(sql_server.name, database.name) \
    .apply(lambda args:
        f"Server=tcp:{args[0]}.database.windows.net;initial catalog={args[1]};")

# 키워드 인자를 사용하는 방식
connection_string = Output.all(server=sql_server.name, db=database.name) \
    .apply(lambda args:
        f"Server=tcp:{args['server']}.database.windows.net;initial catalog={args['db']};")
```

### 새 데이터 구조 생성

```typescript
// 새로운 딕셔너리 Output 생성
let connectionDetails = pulumi.all([server.ipAddress, database.port])
    .apply(([ip, port]) => ({
        serverIp: ip,
        databasePort: port,
    }));

// 새로운 배열 Output 생성
let connectionList = pulumi.all([server.ipAddress, database.port])
    .apply(([ip, port]) => [ip, port]);
```

```python
from pulumi import Output

# 새로운 Dict Output 생성
connection_details = Output.all(sql_server.ipAddress, database.port) \
    .apply(lambda args: {
        "server_ip": args[0],
        "database_port": args[1]
    })

# 새로운 List Output 생성
connection_list = Output.all(sql_server.ipAddress, database.port) \
    .apply(lambda args: [args[0], args[1]])
```

---

## Output Helper 함수

`apply`나 `all`을 직접 호출하는 대신, 자주 쓰는 변환을 간결하게 수행하는 Helper 함수를 제공한다.

### 문자열 보간 Helper

| 언어 | Helper | 설명 |
|---|---|---|
| TypeScript | `pulumi.interpolate` | 태그드 템플릿 리터럴. `${}` 안에 Output 직접 사용 |
| TypeScript | `pulumi.concat()` | 문자열과 Output 목록을 단일 `Output<string>`으로 결합 |
| Python | `pulumi.Output.format()` | `str.format()`과 동일한 방식. 위치/키워드 플레이스홀더 |
| Python | `pulumi.Output.concat()` | 문자열과 Output 목록을 단일 `Output[str]`로 결합 |
| Go | `pulumi.Sprintf()` | `fmt.Sprintf()`와 동일한 방식 |
| C# | `Output.Format()` | C# 문자열 보간 구문 사용 |
| Java | `Output.format()` | `String.format()`과 동일한 `%s` 플레이스홀더 |
| YAML | `${...}` | YAML에서 네이티브 문자열 보간 지원 |

```typescript
const bucket = new aws.s3.Bucket("bucket");

// concat: 여러 값을 하나의 Output으로 결합
export const s3Url1 = pulumi.concat("s3://", bucket.bucket, "/", file.key);

// interpolate: 템플릿 리터럴에서 Output 직접 전개
export const s3Url2 = pulumi.interpolate`s3://${bucket.bucket}/${file.key}`;
```

```python
bucket = aws.s3.Bucket("bucket")

# concat: 여러 값을 하나의 Output으로 결합
s3Url1 = pulumi.Output.concat("s3://", bucket.bucket, "/", file.key)

# format: 포맷 문자열과 Output 결합
s3Url2 = pulumi.Output.format("s3://{0}/{1}", bucket.bucket, file.key)
```

### JSON Helper

| 기능 | TypeScript | Python | Go | C# |
|---|---|---|---|---|
| JSON 직렬화 | `pulumi.jsonStringify()` | `pulumi.Output.json_dumps()` | `pulumi.JSONMarshal()` | `Output.JsonSerialize()` |
| JSON 역직렬화 | `pulumi.jsonParse()` | `pulumi.Output.json_loads()` | (전용 Helper 없음, `ApplyT`+`json.Unmarshal` 사용) | `Output.JsonDeserialize<T>()` |

```typescript
const accountID = aws.getCallerIdentityOutput().accountId;
const bucket = new aws.s3.Bucket("my-bucket");

const policy = new aws.s3.BucketPolicy("my-bucket-policy", {
    bucket: bucket.id,
    policy: pulumi.jsonStringify({
        Version: "2012-10-17",
        Statement: [{
            Effect: "Allow",
            Principal: {
                AWS: pulumi.interpolate`arn:aws:iam::${accountID}:root`,
            },
            Action: "s3:ListBucket",
            Resource: bucket.arn,
        }],
    }),
});
```

```python
account_id = aws.get_caller_identity_output().apply(
    lambda identity: identity.account_id
)
bucket = aws.s3.Bucket("my-bucket")

policy = aws.s3.BucketPolicy(
    "my-bucket-policy",
    bucket=bucket.id,
    policy=pulumi.Output.json_dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {
                "AWS": pulumi.Output.format("arn:aws:iam::{0}:root", account_id)
            },
            "Action": "s3:ListBucket",
            "Resource": bucket.arn,
        }],
    }),
)
```

---

## Input\<T\>를 Output\<T\>로 변환

`Input<T>` 값을 확실한 `Output<T>`로 변환해야 하는 경우가 있다. 주로 컴포넌트 리소스나 `Input<T>`를 받는 유틸리티 함수 내부에서 `apply`를 호출해야 할 때 필요하다.

| 언어 | 변환 함수 |
|---|---|
| TypeScript | `pulumi.output(value)` |
| Python | `pulumi.Output.from_input(value)` |
| Go | `input.ToXxxOutput()` (예: `ToStringOutput()`) |
| C# | `Output.Create(value)` |
| Java | `Output.of(value)` |

값이 이미 `Output<T>`면 변환 함수는 그대로 반환하고, 평문 값이면 즉시 확정되는 새 Output으로 래핑한다.

```typescript
function buildUrl(host: pulumi.Input<string>): pulumi.Output<string> {
    return pulumi.output(host).apply(h => `https://${h}`);
}

const fromPlain = buildUrl("example.com");       // 평문 값
const fromOutput = buildUrl(bucket.websiteEndpoint); // Output 값
```

```python
def build_url(host: pulumi.Input[str]) -> pulumi.Output[str]:
    return pulumi.Output.from_input(host).apply(lambda h: f"https://{h}")

from_plain = build_url("example.com")            # 평문 값
from_output = build_url(bucket.website_endpoint)  # Output 값
```

---

## Output과 조건문 / 반복문

### 조건부 리소스 생성

Output 값에 따라 리소스를 조건부로 생성해야 할 때, **Provider Function의 직접 형식(direct form)**을 사용하면 `apply` 없이 일반 조건문을 사용할 수 있다.

직접 형식은 평문 값을 반환하므로 일반 `if` 문을 사용할 수 있다:

```typescript
// 직접 형식은 Promise를 반환하므로 await로 평문 값 획득
const candidates = await aws.ec2.getAmiIds({
    owners: ["amazon"],
    filters: [{ name: "name", values: ["amzn2-ami-hvm-*"] }],
});

// 평문 값이므로 일반 조건문 사용 가능
if (candidates.ids.length > 0) {
    new aws.ec2.Instance("web", {
        ami: candidates.ids[0],
        instanceType: "t3.micro",
    });
}
```

```python
# 직접 형식은 평문 결과를 반환
candidates = aws.ec2.get_ami_ids(
    owners=["amazon"],
    filters=[{"name": "name", "values": ["amzn2-ami-hvm-*"]}],
)

# 평문 값이므로 일반 조건문 사용 가능
if candidates.ids:
    aws.ec2.Instance(
        "web",
        ami=candidates.ids[0],
        instance_type="t3.micro",
    )
```

### Output 형식(output form)을 사용해야 하는 경우

리소스가 먼저 생성된 후 함수를 실행해야 하면 **Output 형식**을 사용한다. Output 형식은 Input을 인자로 받으므로 `apply` 없이 Output 값을 직접 전달할 수 있다.

```typescript
const amiNameFilter = config.requireSecret("amiNameFilter");

// Output 형식은 Input을 받으므로 Secret Output을 apply 없이 직접 전달
const latestAmi = aws.ec2.getAmiOutput({
    owners: ["amazon"],
    mostRecent: true,
    filters: [{ name: "name", values: [amiNameFilter] }],
});

new aws.ec2.Instance("web", {
    ami: latestAmi.imageId,
    instanceType: "t3.micro",
});
```

```python
ami_name_filter = config.require_secret("amiNameFilter")

# Output 형식은 Input을 받으므로 Secret Output을 apply 없이 직접 전달
latest_ami = aws.ec2.get_ami_output(
    owners=["amazon"],
    most_recent=True,
    filters=[{"name": "name", "values": [ami_name_filter]}],
)

aws.ec2.Instance(
    "web",
    ami=latest_ami.image_id,
    instance_type="t3.micro",
)
```

> Pulumi는 특별한 이유가 없는 한 Output 형식 사용을 권장한다. 하나의 프로그래밍 모델(Input/Output)만 사용하면 되고, 직접 형식은 언어별 반환 타입(Promise, Task 등)도 함께 관리해야 하기 때문이다.

---

## Provider Functions (fn::invoke)

Pulumi는 클라우드 리소스와 상호작용하고 데이터를 조회하는 세 가지 함수 타입을 제공한다.

| 함수 타입 | 용도 | 특징 |
|---|---|---|
| **Provider Functions** | 클라우드 API를 쿼리해 리소스가 아닌 데이터 조회 | 예: 최신 AMI ID 조회 |
| **Get Functions** | Pulumi가 관리하지 않는 기존 리소스 참조 | 읽기 전용, 업데이트/삭제 불가 |
| **Resource Methods** | Pulumi가 관리하는 리소스에서 파생된 값 반환 | 리소스 인스턴스에서 호출 |

### Provider Functions 예제

```typescript
const latestAmi = await aws.ec2.getAmi({
    owners: ["amazon"],
    mostRecent: true,
    filters: [
        { name: "name", values: ["amzn2-ami-hvm-*"] },
        { name: "architecture", values: ["x86_64"] },
    ],
});

new aws.ec2.Instance("web-server", {
    ami: latestAmi.imageId,
    instanceType: "t3.micro",
});
```

```python
latest_ami = aws.ec2.get_ami(
    owners=["amazon"],
    most_recent=True,
    filters=[
        {"name": "name", "values": ["amzn2-ami-hvm-*"]},
        {"name": "architecture", "values": ["x86_64"]},
    ],
)

instance = aws.ec2.Instance(
    "web-server",
    ami=latest_ami.image_id,
    instance_type="t3.micro",
)
```

### 직접 형식 vs Output 형식

Provider Functions는 두 가지 형식으로 노출된다.

| 구분 | 직접 형식 (Direct Form) | Output 형식 (Output Form) |
|---|---|---|
| **인자** | 평문 값 | Pulumi Input (또는 평문 값) |
| **반환** | Promise / 평문 값 | `Output<T>` |
| **실행 시점** | 프로그램 평가 중 즉시 실행 | Pulumi 엔진이 의존성 그래프 내에서 관리 |
| **이름 규칙** | `getX()` (TS), `get_x()` (Py) | `getXOutput()` (TS), `get_x_output()` (Py) |
| **`dependsOn`** | 사용 불가 | 사용 가능 |
| **적합한 경우** | 조건부 리소스 생성 여부 결정 | Output/Input 값 전달, 의존성 추적 필요 |

### Invoke Options

Provider Functions는 리소스 옵션과 유사한 **invoke options**를 지원한다.

| 옵션 | 설명 |
|---|---|
| `dependsOn` | 이 함수가 의존하는 리소스 배열. Output 형식만 지원 |
| `parent` | 부모 리소스 지정. Provider 결정 시 참조 |
| `provider` | 명시적으로 구성된 Provider 사용 |

다음 옵션은 더 이상 사용이 권장되지 않는다.

| 옵션 | 설명 |
|---|---|
| `pluginDownloadURL` | Provider 플러그인 다운로드 URL |
| `version` | Provider 플러그인 버전 |
| `async` | 더 이상 사용되지 않음 (deprecated) |

---

## Output을 직접 조작하지 말아야 할 패턴

### 1. apply 내부에서 리소스 생성 금지

```typescript
// 잘못된 패턴: apply 안에서 리소스 생성
bucket.arn.apply(arn => {
    new aws.s3.BucketPolicy("policy", { /* ... */ }); // preview에서 누락될 수 있음
});

// 올바른 패턴: Output을 직접 Input으로 전달
const policy = new aws.s3.BucketPolicy("policy", {
    bucket: bucket.id,
    policy: pulumi.jsonStringify({ /* ... */ }),
});
```

```python
# 잘못된 패턴
bucket.arn.apply(lambda arn: aws.s3.BucketPolicy("policy", ...))

# 올바른 패턴: Output을 직접 Input으로 전달
policy = aws.s3.BucketPolicy("policy",
    bucket=bucket.id,
    policy=pulumi.Output.json_dumps({ /* ... */ }),
)
```

### 2. all 콜백 내부에서 리소스 생성 금지

```typescript
// 잘못된 패턴: all 안에서 리소스 생성
pulumi.all([vpc.vpcId, subnet.cidrBlock]).apply(([vpcId, cidr]) => {
    new aws.ec2.SecurityGroup("sg", { vpcId, /* ... */ }); // preview에서 누락될 수 있음
});

// 올바른 패턴: Output을 직접 Input으로 전달
const sg = new aws.ec2.SecurityGroup("sg", {
    vpcId: vpc.vpcId,
    // ...
});
```

### 3. apply 내부에서 스택 출력(export) 생성 금지

```typescript
// 잘못된 패턴
server.publicDns.apply(dns => {
    export const url = dns; // 에러 발생
});

// 올바른 패턴: Output을 직접 export
export const url = server.publicDns;
```

```python
# 잘못된 패턴
server.public_dns.apply(lambda dns: pulumi.export("url", dns))

# 올바른 패턴: Output을 직접 export
pulumi.export("url", server.public_dns)
```

### 4. Output을 평문처럼 직접 조작 금지

```typescript
// 잘못된 패턴
console.log(vpc.vpcId);  // Output 객체 자체가 출력됨

// 올바른 패턴
vpc.vpcId.apply(id => console.log(`VPC ID: ${id}`));
```

```python
# 잘못된 패턴
print(vpc.vpc_id)  # "Calling __str__ on an Output[T] is not supported" 에러

# 올바른 패턴
vpc.vpc_id.apply(lambda id: print('VPC ID:', id))
```

### 5. 문자열 결합 시 apply 대신 Helper 사용

```typescript
// 가능하지만 장황함
const url = server.publicDns.apply(dns => `https://${dns}`);

// Helper 사용이 더 간결
const url = pulumi.interpolate`https://${server.publicDns}`;
```

```python
# 가능하지만 장황함
url = server.public_dns.apply(lambda dns: "https://" + dns)

# Helper 사용이 더 간결
url = pulumi.Output.format("https://{0}", server.public_dns)
```
