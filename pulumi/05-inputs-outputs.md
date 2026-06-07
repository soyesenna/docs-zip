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

## Input과 Output이 필요한 이유: 선언형 vs 명령형

Pulumi의 Input/Output 시스템은 **명령형(imperative) 프로그래밍 언어**로 작성된 프로그램에서 **선언형(declarative)** 인프라 관리를 가능하게 하는 핵심 메커니즘이다.

**명령형** 프로그래밍에서는 단계별로 컴퓨터에 "VPC를 먼저 생성하고, 완료될 때까지 대기한 다음, VPC ID를 가져와서 Subnet을 생성하라"고 지시한다.

**선언형** 프로그래밍에서는 "VPC와 그 안에 있는 Subnet을 원한다"고 원하는 상태만 기술하면, 시스템이 자동으로 VPC를 먼저 만들고 Subnet을 그 뒤에 만드는 순서를 결정한다.

| 구분 | 명령형 | 선언형 (Pulumi) |
|---|---|---|
| 접근 방식 | 단계별 실행 순서를 직접 제어 | 원하는 상태를 선언, 시스템이 순서 결정 |
| 의존성 관리 | 개발자가 명시적으로 대기/순서 제어 | Output을 Input으로 전달하면 자동 추적 |
| 대표 예시 | "VPC 생성 → 대기 → ID 획득 → Subnet 생성" | "VPC의 `vpcId`를 Subnet의 `vpcId`에 전달" |

Pulumi의 Input/Output은 한 리소스의 Output을 다른 리소스의 Input으로 전달할 때 의존성을 **자동으로 기록**하고, 리소스가 올바른 순서로 생성·수정·삭제되도록 보장한다. 개발자가 명시적인 순서 제어 로직을 작성할 필요가 없다.

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

### Python의 객체 타입 Input: Args vs ArgsDict

Python에서 여러 값을 그룹화하는 객체 타입 Input은 **클래스(class)** 또는 **딕셔너리 리터럴(dictionary literal)** 두 가지 방식으로 표현할 수 있다.

| 표현 방식 | 타입 접미사 | 특징 |
|---|---|---|
| 클래스 | `Args` | 타입 힌트 및 IDE 자동완성 지원 |
| 딕셔너리 리터럴 | `ArgsDict` | 더 간결한 표기 가능 |

> `ArgsDict` 접미사 타입은 2024년 7월에 도입되었다. 아직 업데이트되지 않은 Provider에서는 기존 딕셔너리 리터럴을 그대로 사용할 수 있지만, 새 타입의 타입 검사 혜택을 받을 수 없다.

```python
import pulumi_aws as aws

# 딕셔너리 리터럴 사용 (ArgsDict)
repo1 = aws.ecr.Repository("repo1-with-dictionary-literals",
    image_tag_mutability="MUTABLE",
    image_scanning_configuration={
        "scan_on_push": True,
    })

# 클래스 사용 (Args)
repo2 = aws.ecr.Repository("repo2-with-args",
    image_tag_mutability="MUTABLE",
    image_scanning_configuration=aws.ecr.RepositoryImageScanningConfigurationArgs(
        scan_on_push=True
    ))
```

### 리소스 식별자(Identity)와 Input

Pulumi 리소스는 네 가지 식별 형태를 가진다. 각 형태는 용도가 다르며, 잘못된 형태를 인자에 전달하는 것은 Python 프로그램에서 타입 불일치 에러의 가장 흔한 원인이다.

| 식별 형태 | 설명 | 접근 방식 | 타입 |
|---|---|---|---|
| Logical name | Pulumi가 상태 추적·URN 생성에 사용 | 생성자 첫 번째 인자 | `str` (평문) |
| Physical name | 클라우드 공급자가 할당한 이름 | 리소스 속성 | `Output[str]` |
| Physical ID | 클라우드 공급자가 할당한 ID (예: `vpc-0abc123`) | `resource.id` | `Output[str]` |
| URN | Pulumi 내부 식별자 (`urn:pulumi:<stack>::<project>::<type>::<name>`) | `resource.urn` | `Output[str]` |

**리소스 입력에 Physical ID 전달**: 한 리소스가 다른 리소스를 참조할 때(예: Subnet을 VPC 안에 배치), 상위 리소스의 `id` Output을 하위 리소스의 Input으로 전달한다.

```python
subnet = aws.ec2.Subnet("main-subnet",
    vpc_id=vpc.id,          # Output[str] — AWS가 할당한 VPC ID
    cidr_block="10.0.1.0/24",
    availability_zone="us-east-1a",
)
```

**ResourceOptions에는 리소스 객체 자체를 전달**: `parent`, `depends_on`, `provider`, `deleted_with` 등 `ResourceOptions` 필드는 URN이나 ID가 아닌 **리소스 객체 자체**를 받는다.

```python
# 올바른 예: 리소스 객체 전달
subnet = aws.ec2.Subnet("main-subnet",
    vpc_id=vpc.id,
    cidr_block="10.0.1.0/24",
    opts=pulumi.ResourceOptions(
        parent=vpc,          # 리소스 객체 — vpc.urn, vpc.id 아님
        depends_on=[vpc],    # 리소스 객체 리스트
    ),
)
```

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

### apply 체이닝 패턴

`apply`의 반환값은 새로운 `Output<T>`이므로, 연속해서 `apply`를 호출하는 **체이닝**이 가능하다. 각 단계의 의존성은 자동으로 유지된다.

```typescript
// 체이닝: DNS 이름 → URL → 대문자 변환
const upperUrl = server.publicDns
    .apply(dns => `https://${dns}`)
    .apply(url => url.toUpperCase());

export const result = upperUrl;
```

```python
# 체이닝: DNS 이름 → URL → 대문자 변환
upper_url = server.public_dns \
    .apply(lambda dns: f"https://{dns}") \
    .apply(lambda url: url.upper())

pulumi.export("result", upper_url)
```

### Output과 Secret

Output이 Secret(비밀)으로 표시되면 Pulumi 엔진은 state 파일과 모든 전파 경로에서 해당 값을 **암호화**한다. Secret Output은 다음 방법으로 생성된다.

| 방법 | 설명 |
|---|---|
| `pulumi.secret()` / `Output.secret()` | 기존 값을 Secret Output으로 래핑 |
| `config.requireSecret()` | Secret 구성 값 읽기 |
| `additionalSecretOutputs` 옵션 | 리소스의 특정 Output 속성을 Secret으로 표시 |
| `apply` 또는 `all`에서 Secret 값 사용 | Secret Output과의 연산 결과도 자동으로 Secret |

`apply` 또는 `Output.all`의 콜백 내부에서는 Secret이 **평문으로 복호화**되어 전달된다. 이 평문 값을 신뢰할 수 없는 코드에 전달하지 않도록 주의해야 한다.

#### additionalSecretOutputs 리소스 옵션

`additionalSecretOutputs` 리소스 옵션은 특정 Output 속성 이름 목록을 받아, 해당 속성을 Secret으로 처리한다. Pulumi가 Secret Input을 통해 자동 감지하는 값 목록에 추가된다.

> **커스텀 리소스에만 적용**된다. TypeScript, C#, Java SDK에서는 `CustomResourceOptions`에 정의되어 있어 컴포넌트 리소스에 전달하면 컴파일 에러가 발생한다. Python과 Go SDK는 단일 리소스 옵션 타입을 노출하므로 컴파일 타임에는 허용되지만 컴포넌트 리소스에 적용해도 직접적인 효과는 없다.

최상위(top-level) 리소스 속성만 Secret으로 지정할 수 있다. 민감한 데이터가 중첩된 속성 내부에 있으면 전체 최상위 Output 속성을 Secret으로 표시해야 한다.

```typescript
let db = new Database("db", { /*...*/ }, {
    additionalSecretOutputs: ["password"],
});
```

```python
db = Database("db",
    opts=pulumi.ResourceOptions(additional_secret_outputs=["password"]),
)
```

```go
db, err := NewDatabase(ctx, "db", &DatabaseArgs{/*...*/},
    pulumi.AdditionalSecretOutputs([]string{"password"}))
```

```csharp
var db = new Database("db", new DatabaseArgs(),
    new CustomResourceOptions { AdditionalSecretOutputs = { "password" } });
```

```java
var db = new Database("db",
    DatabaseArgs.Empty,
    CustomResourceOptions.builder()
        .additionalSecretOutputs("password")
        .build());
```

#### Output.isSecret

Output 객체는 `isSecret` 속성을 통해 해당 Output이 Secret인지 확인할 수 있는 정보를 제공한다. 공식 문서에서는 Output을 직접 출력했을 때 나타나는 디버그 정보에서 `isSecret: Promise { <pending> }` 형태로 확인할 수 있으며, 이는 Output이 비동기적으로 Secret 여부를 추적함을 보여준다.

```
// TypeScript에서 Output 직접 출력 시 (권장하지 않음)
OutputImpl {
    __pulumiOutput: true,
    resources: [Function (anonymous)],
    allResources: [Function (anonymous)],
    isKnown: Promise { <pending> },
    isSecret: Promise { <pending> },
    promise: [Function (anonymous)],
    toString: [Function (anonymous)],
    toJSON: [Function (anonymous)]
}
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

```go
connectionString := pulumi.All(sqlServer.Name, database.Name).ApplyT(
    func(args []interface{}) pulumi.StringOutput {
        server := args[0]
        db := args[1]
        return pulumi.Sprintf(
            "Server=tcp:%s.database.windows.net;initial catalog=%s;",
            server, db,
        )
    },
)
```

```csharp
// Output.All: 모든 입력이 같은 타입일 때 사용, ImmutableArray 생성
var connectionString = Output.All(sqlServer.Name, database.Name)
    .Apply(t => $"Server=tcp:{t[0]}.database.windows.net;initial catalog={t[1]};");

// Output.Tuple: 각 값의 타입을 개별적으로 보존
var connectionString2 = Output.Tuple(sqlServer.Name, database.Name)
    .Apply(t => $"Server=tcp:{t.Item1}.database.windows.net;initial catalog={t.Item2};");
```

```java
// Output.all: 모든 입력이 같은 타입일 때 사용
var connectionString = Output.all(sqlServer.name(), database.name())
    .applyValue(t -> String.format(
        "Server=tcp:%s.database.windows.net;initial catalog=%s;",
        t.get(0), t.get(1)));

// Output.tuple: 각 값의 타입을 개별적으로 보존
var connectionString2 = Output.tuple(sqlServer.name, database.name())
    .applyValue(t -> String.format(
        "Server=tcp:%s.database.windows.net;initial catalog=%s;",
        t.t1, t.t2));
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

### Get Functions

Get Functions는 Pulumi가 관리하지 않는 **기존 리소스를 참조**하는 데 사용하는 패키지 수준 함수다. `pulumi import` 명령이 리소스를 Pulumi 관리로 가져오는 것과 달리, Get Functions는 기존 리소스의 속성을 **읽기 전용**으로 조회한다.

| 특징 | 설명 |
|---|---|
| 용도 | Pulumi가 관리하지 않는 기존 리소스 참조 |
| 권한 | 읽기 전용, 업데이트/삭제 불가 |
| 접근 방식 | 리소스 타입의 `get:` 스탠자 또는 정적 메서드 사용 |
| 사용 시점 | 관리되지 않는 리소스의 ID를 알고 있고, 해당 속성을 Pulumi 관리 리소스에서 참조해야 할 때 |

```typescript
// ID로 기존 VPC를 조회하여 속성 참조
const existingVpc = aws.ec2.Vpc.get("imported-vpc", "vpc-0abc123def456789");
```

```python
# ID로 기존 VPC를 조회하여 속성 참조
existing_vpc = aws.ec2.Vpc.get("imported-vpc", id="vpc-0abc123def456789")
```

> Get Functions로 접근한 리소스는 Pulumi에 의해 업데이트되거나 삭제되지 않는다.

### Resource Methods

Resource Methods는 Pulumi가 관리 중인 **특정 리소스 타입에 연결된 함수**로, 해당 리소스 인스턴스에서 호출하여 파생된 값을 반환한다. 예를 들어 관리형 Kubernetes 클러스터 리소스는 Kubeconfig 파일을 반환하는 리소스 메서드를 제공한다.

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
