# Pulumi 리소스

> https://www.pulumi.com/docs/iac/concepts/resources/
> https://www.pulumi.com/docs/iac/concepts/stacks/
> https://www.pulumi.com/docs/iac/concepts/resources/names/
> https://www.pulumi.com/docs/iac/concepts/resources/options/
> https://www.pulumi.com/docs/iac/concepts/components/

Pulumi에서 **리소스(Resource)** 는 클라우드 인프라를 구성하는 기본 단위다. 컴퓨트 인스턴스, 스토리지 버킷, Kubernetes 클러스터 등 모든 인프라 요소는 리소스로 표현된다. 모든 리소스는 `Resource` 클래스의 두 하위 클래스 중 하나로 정의되며, 선언한 리소스는 **스택(Stack)** 이라는 격리된 배포 단위 내에서 관리된다.

---

## 리소스 종류

Pulumi의 모든 인프라 리소스는 다음 두 가지 하위 클래스 중 하나로 분류된다.

| 종류 | 설명 |
|---|---|
| **CustomResource** | AWS, Azure, GCP, Kubernetes 등의 [리소스 프로바이더](https://www.pulumi.com/docs/iac/concepts/providers/)가 관리하는 실제 클라우드 리소스 |
| **ComponentResource** | 여러 리소스를 논리적으로 묶어 더 높은 수준의 추상화를 제공하는 그룹 리소스. 내부 구현을 캡슐화한다 |

### Custom Resource

`CustomResource`는 클라우드 프로바이더가 직접 관리하는 실제 리소스를 나타낸다. 예를 들어 `aws.s3.Bucket`, `azure-native.compute.VirtualMachine` 등이 여기에 해당한다.

```typescript
import * as aws from "@pulumi/aws";

const bucket = new aws.s3.Bucket("my-bucket");
```

```python
import pulumi_aws as aws

bucket = aws.s3.Bucket("my-bucket")
```

### Component Resource

`ComponentResource`는 여러 관련 리소스를 하나의 리소스로 노출하는 논리적 그룹이다. 소비자는 복잡한 인프라를 간단한 인터페이스로 생성할 수 있으며, 내부 구현 세부 사항을 알 필요가 없다.

컴포넌트는 Pulumi 프로그램 내에 인라인으로 정의되거나, 언어 생태계 라이브러리를 통해 공유되거나, Pulumi 패키지의 일부로 배포될 수 있다.

```typescript
import * as awsx from "@pulumi/awsx";

const vpc = new awsx.ec2.Vpc("vpc", {
    subnetSpecs: [
        { type: awsx.ec2.SubnetType.Public, cidrMask: 22 },
        { type: awsx.ec2.SubnetType.Private, cidrMask: 20 },
    ],
}, { protect: true });
```

```python
import pulumi_awsx as awsx

vpc = awsx.ec2.Vpc("vpc",
    awsx.ec2.VpcArgs(
        subnet_specs=[
            awsx.ec2.SubnetSpecArgs(type=awsx.ec2.SubnetType.PUBLIC, cidr_mask=22),
            awsx.ec2.SubnetSpecArgs(type=awsx.ec2.SubnetType.PRIVATE, cidr_mask=20),
        ],
    ),
    opts=pulumi.ResourceOptions(protect=True),
)
```

`pulumi up` 실행 시 컴포넌트는 자식 리소스가 중첩된 트리 형태로 출력된다:

```text
Updating (dev):
     Type                            Name              Status
 +   pulumi:pulumi:Stack             my-stack          created
 +   └─ awsx:ec2:Vpc                 vpc               created
 +      └─ aws:ec2:Vpc               vpc               created
 +         ├─ aws:ec2:Subnet         vpc-private-1     created
 +         └─ aws:ec2:Subnet         vpc-public-1      created
```

---

## 리소스 등록

리소스의 원하는 상태는 리소스의 인스턴스를 생성(construct)하여 선언한다.

```typescript
let res = new Resource(name, args, options);
```

```python
res = Resource(name, args, options)
```

### name 인자

모든 리소스에는 필수 `name` 인자가 있다. 이 **논리적 이름(logical name)** 은 동일한 종류의 리소스 간에 고유해야 하며, 같은 스택 내에서 겹칠 수 없다. 논리적 이름은 클라우드 프로바이더가 할당하는 **물리적 이름(physical name)** 에 영향을 미친다. Pulumi는 기본적으로 물리적 리소스를 [자동 명명(auto-naming)](#자동-명명-autonaming)하므로 물리적 이름과 논리적 이름이 다를 수 있다.

### args 인자

`args`는 리소스를 초기화하는 데 사용되는 명명된 속성 입력 값의 집합이다. 문자열, 정수, 리스트, 맵 같은 일반 값이거나 다른 리소스의 [출력(outputs)](https://www.pulumi.com/docs/concepts/inputs-outputs/)일 수 있다. 각 리소스가 지원하는 인자는 [Pulumi Registry](https://www.pulumi.com/registry/)에서 확인할 수 있다.

### options 인자

`options`는 선택 사항이며 리소스 관리 방식을 제어한다. 명시적 종속성 지정, 커스텀 프로바이더 구성 사용, 기존 인프라 가져오기 등이 가능하다. 자세한 내용은 [Resource Options](#resource-options) 섹션을 참조하라.

---

## 리소스 이름과 식별

각 리소스는 서로 다른 목적을 가진 **네 가지 식별 형태** 를 갖는다.

| 식별 형태 | 출처 | 예시 값 | 사용 시기 |
|---|---|---|---|
| **논리적 이름** | 코드 — 생성자의 첫 번째 인자 | `"my-bucket"` | 리소스 생성자에 전달. URN 및 물리적 이름 접두사에 영향 |
| **물리적 이름** | 프로바이더 (논리적 이름과 자동 명명에 영향을 받음) | `"my-bucket-d7c3a1f"` | 프로바이더 API 호출에 사용. 프로바이더별 출력 속성으로 읽기 |
| **물리적 ID** | 프로바이더 — 리소스 생성 후 반환 | `"my-bucket-d7c3a1f"` (S3), `"vpc-0abc1234"` (VPC) | `import`, `get`, 프로바이더 API에 전달. `resource.id`로 접근 |
| **URN** | Pulumi — 프로젝트, 스택, 타입, 논리적 이름에서 파생 | `"urn:pulumi:dev::app::aws:s3/bucket:Bucket::my-bucket"` | Pulumi CLI 명령(`pulumi state`). 프로그램 코드에서는 거의 사용하지 않음. `resource.urn`으로 접근 |

> **주의:** `dependsOn` 옵션은 **리소스 참조**(변수 자체) 목록을 받으며, URN이나 ID가 아니다.

### 자동 명명 (Auto-Naming)

대부분의 물리적 리소스 이름은 기본적으로 자동 명명된다. 논리적 이름이 `my-role`이더라도 물리적 이름은 `my-role-d7c2fa0`처럼 임의의 접미사가 추가된다.

이 임의 접미사는 두 가지 목적을 갖는다:

1. **스택 간 충돌 방지** — 동일한 프로젝트의 여러 스택이 리소스 이름 충돌 없이 배포될 수 있다.
2. **무중단 업데이트** — 일부 클라우드 프로바이더는 특정 업데이트 시 리소스 교체가 필요하다. Pulumi는 먼저 새 리소스를 생성하고 기존 참조를 업데이트한 뒤 이전 리소스를 삭제한다.

명시적 이름을 지정하려면 대부분의 리소스에 `name` 속성을 사용한다:

```typescript
let role = new aws.iam.Role("my-role", {
    name: "my-role-001",
});
```

```python
role = iam.Role('my-role', name='my-role-001')
```

### 자동 명명 설정

`pulumi:autonaming` 구성 설정으로 자동 명명 동작을 커스터마이즈할 수 있다.

| 모드 | 설명 |
|---|---|
| `default` | 기본 동작. 임의 접미사 추가 |
| `verbatim` | 논리적 이름을 그대로 사용. 접미사 없음 |
| `disabled` | 모든 리소스에 명시적 이름 필수 |

**패턴 표현식:**

| 표현식 | 설명 | 예시 |
|---|---|---|
| `${name}` | 논리적 리소스 이름 | `${name}` |
| `${hex(n)}` | 길이 n의 16진수 임의 문자열 | `${name}-${hex(5)}` |
| `${alphanum(n)}` | 길이 n의 영숫자 임의 문자열 | `${name}${alphanum(4)}` |
| `${string(n)}` | 길이 n의 임의 문자 문자열 | `${name}${string(6)}` |
| `${num(n)}` | 길이 n의 임의 숫자 문자열 | `${name}${num(4)}` |
| `${uuid}` | UUID | `${uuid}` |
| `${organization}` | 조직 이름 | `${organization}_${name}` |
| `${project}` | 프로젝트 이름 | `${project}_${name}` |
| `${stack}` | 스택 이름 | `${stack}_${name}` |
| `${config.key}` | 키의 구성 값 | `${config.region}_${name}` |

```yaml
# Pulumi.<stack-name>.yaml
config:
  pulumi:autonaming:
    pattern: ${name}-${project}-${stack}
```

> **주의:** `verbatim` 모드나 임의 구성 요소가 없는 패턴을 사용하면, 교체가 필요한 리소스가 먼저 삭제된 후 새로 생성되므로 가동 중단이 발생할 수 있다.

### 리소스 타입 토큰

각 리소스는 `<package>:<module>:<typename>` 형식의 타입 토큰으로 식별된다.

| 타입 토큰 | 패키지 | 모듈 | 타입명 |
|---|---|---|---|
| `aws:s3/bucket:Bucket` | `aws` | `s3/bucket` | `Bucket` |
| `azure-native:compute:VirtualMachine` | `azure-native` | `compute` | `VirtualMachine` |
| `kubernetes:apps/v1:Deployment` | `kubernetes` | `apps/v1` | `Deployment` |
| `random:index:RandomPassword` | `random` | `index` | `RandomPassword` |

### URN (Uniform Resource Name)

각 리소스에는 전역적으로 고유한 URN이 자동으로 생성된다. URN은 프로젝트 이름, 스택 이름, 리소스 이름, 리소스 타입, 모든 부모 리소스의 타입으로 구성된다.

```text
urn:pulumi:production::acmecorp-website::custom:resources:Resource$aws:s3/bucket:Bucket::my-bucket
           ^^^^^^^^^^  ^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^^^ ^^^^^^^^^^^^^^^^^^^^  ^^^^^^^^^
           <stack-name>                 <resource-type>       <resource-name>
```

URN은 전역적으로 고유해야 한다. 동일한 이름, 타입, 부모 경로를 가진 두 리소스를 생성하면 오류가 발생한다.

> **주의:** 리소스 이름을 변경하면 새 URN이 생성되어 이전 리소스는 삭제, 새 리소스는 생성된다. 이름 변경 없이 리소스를 유지하려면 `aliases` 옵션을 사용하라.

---

## Resource Options

모든 Pulumi 리소스는 리소스 관리 방식을 커스터마이즈하는 공통 옵션 세트를 지원한다.

### 전체 옵션 목록

| 옵션 | 설명 | 적용 대상 |
|---|---|---|
| `additionalSecretOutputs` | 암호(Secret)로 암호화해야 할 출력 속성 지정 | Custom |
| `aliases` | 이름 변경 또는 리팩토링 시 리소스 교체 방지를 위한 별칭 지정 | Custom, Component |
| `customTimeouts` | 리소스 프로비저닝의 기본 재시도/타임아웃 동작 재정의 | Custom |
| `deleteBeforeReplace` | 교체 시 기존 리소스를 먼저 삭제한 후 새 리소스 생성 | Custom |
| `deletedWith` | 지정된 리소스도 삭제될 때 이 리소스의 삭제를 건너뜀 | Custom |
| `dependsOn` | 종속성 그래프 외에 명시적 종속성 지정 | Custom, Component |
| `envVarMappings` | 프로바이더 인증을 위해 환경 변수를 커스텀 키로 리매핑 | Provider |
| `hideDiffs` | CLI 출력에서 지정된 속성의 diff 표시를 간소화 | Custom |
| `hooks` | 리소스 수명주기의 특정 지점에서 커스텀 로직 실행 | Custom, Component |
| `ignoreChanges` | diff 시 지정된 속성의 변경 무시 | Custom |
| `import` | 기존 클라우드 리소스를 Pulumi 관리로 가져오기 | Custom |
| `parent` | 리소스 간 부모/자식 관계 설정 | Custom, Component |
| `protect` | 실수로 삭제되지 않도록 리소스를 보호 상태로 표시 | Custom, Component |
| `provider` | 기본 프로바이더 대신 명시적으로 구성된 프로바이더 전달 | Custom, Component |
| `providers` | 컴포넌트의 자식 리소스에 대해 명시적으로 구성된 프로바이더 전달 | Component |
| `replaceOnChanges` | 지정된 속성의 변경을 리소스 교체로 처리 | Custom |
| `replaceWith` | 지정된 리소스 중 하나가 교체될 때 이 리소스도 교체 | Custom |
| `replacementTrigger` | 지정된 트리거 값이 변경될 때마다 강제 교체 | Custom |
| `retainOnDelete` | Pulumi가 삭제 시 클라우드 프로바이더에 리소스 유지 | Custom |
| `transformations` | 리소스 속성을 동적으로 변환 (`transforms` 권장) | Custom, Component |
| `transforms` | 리소스 속성을 실시간으로 동적으로 변환 | Custom, Component |
| `version` | 리소스 작업 시 사용할 프로바이더 플러그인 버전 고정 | Custom |

### 주요 옵션 사용 예제

#### protect

```typescript
const bucket = new aws.s3.Bucket("my-bucket", {}, { protect: true });
```

```python
bucket = aws.s3.Bucket("my-bucket", opts=pulumi.ResourceOptions(protect=True))
```

#### ignoreChanges

```typescript
const instance = new aws.ec2.Instance("my-instance", {
    ami: "ami-0c55b159cbfafe1f0",
    instanceType: "t2.micro",
    tags: { Name: "my-instance" },
}, { ignoreChanges: ["tags"] });
```

```python
instance = aws.ec2.Instance("my-instance",
    ami="ami-0c55b159cbfafe1f0",
    instance_type="t2.micro",
    tags={"Name": "my-instance"},
    opts=pulumi.ResourceOptions(ignore_changes=["tags"]),
)
```

#### dependsOn

```typescript
const bucket = new aws.s3.Bucket("my-bucket");
const obj = new aws.s3.BucketObject("hello.txt", {
    bucket: bucket.id,
    key: "hello.txt",
    content: "Hello!",
}, { dependsOn: [bucket] });
```

```python
bucket = aws.s3.Bucket("my-bucket")
obj = aws.s3.BucketObject("hello.txt",
    bucket=bucket.id,
    key="hello.txt",
    content="Hello!",
    opts=pulumi.ResourceOptions(depends_on=[bucket]),
)
```

#### parent

```typescript
const parent = new awsx.ec2.Vpc("vpc", {});
const subnet = new aws.ec2.Subnet("subnet", {
    vpcId: parent.vpcId,
    cidrBlock: "10.0.1.0/24",
}, { parent: parent });
```

```python
parent = awsx.ec2.Vpc("vpc")
subnet = aws.ec2.Subnet("subnet",
    vpc_id=parent.vpc_id,
    cidr_block="10.0.1.0/24",
    opts=pulumi.ResourceOptions(parent=parent),
)
```

#### provider

```typescript
const myProvider = new aws.Provider("my-provider", { region: "us-west-2" });
const bucket = new aws.s3.Bucket("my-bucket", {}, { provider: myProvider });
```

```python
my_provider = aws.Provider("my-provider", region="us-west-2")
bucket = aws.s3.Bucket("my-bucket", opts=pulumi.ResourceOptions(provider=my_provider))
```

#### deleteBeforeReplace

```typescript
const role = new aws.iam.Role("my-role", {
    name: `my-role-${pulumi.getProject()}-${pulumi.getStack()}`,
}, { deleteBeforeReplace: true });
```

```python
role = iam.Role(
    'my-role',
    name=f'my-role-{ pulumi.get_project() }-{ pulumi.get_stack() }',
    opts=ResourceOptions(delete_before_replace=True),
)
```

#### additionalSecretOutputs

```typescript
const db = new aws.rds.Instance("my-db", {
    engine: "mysql",
    instanceClass: "db.t3.micro",
}, { additionalSecretOutputs: ["password"] });
```

```python
db = aws.rds.Instance("my-db",
    engine="mysql",
    instance_class="db.t3.micro",
    opts=pulumi.ResourceOptions(additional_secret_outputs=["password"]),
)
```

#### aliases

```typescript
const bucket = new aws.s3.Bucket("my-bucket", {}, {
    aliases: [{ name: "old-bucket-name" }],
});
```

```python
bucket = aws.s3.Bucket("my-bucket",
    opts=pulumi.ResourceOptions(aliases=[pulumi.Alias(name="old-bucket-name")]),
)
```

#### customTimeouts

```typescript
const instance = new aws.ec2.Instance("my-instance", {
    ami: "ami-0c55b159cbfafe1f0",
    instanceType: "t2.micro",
}, { customTimeouts: { create: "10m", update: "15m", delete: "10m" } });
```

```python
instance = aws.ec2.Instance("my-instance",
    ami="ami-0c55b159cbfafe1f0",
    instance_type="t2.micro",
    opts=pulumi.ResourceOptions(custom_timeouts=pulumi.CustomTimeouts(
        create="10m", update="15m", delete="10m"
    )),
)
```

#### replaceOnChanges

```typescript
const instance = new aws.ec2.Instance("my-instance", {
    ami: "ami-0c55b159cbfafe1f0",
    instanceType: "t2.micro",
}, { replaceOnChanges: ["ami"] });
```

```python
instance = aws.ec2.Instance("my-instance",
    ami="ami-0c55b159cbfafe1f0",
    instance_type="t2.micro",
    opts=pulumi.ResourceOptions(replace_on_changes=["ami"]),
)
```

### 컴포넌트 리소스와 옵션 상속

모든 리소스 옵션이 컴포넌트 리소스에 적용되는 것은 아니다. 컴포넌트는 클라우드 프로바이더가 지원하지 않는 논리적 그룹이므로 프로바이더 동작에 영향을 주는 옵션은 컴포넌트 자체에 직접적인 효과가 없다.

#### 컴포넌트에서 자식으로 상속되는 옵션

| 옵션 | 자식에게 상속 |
|---|---|
| `additionalSecretOutputs` | N/A (컴포넌트에 설정 불가) |
| `aliases` | 예 |
| `customTimeouts` | 아니오 |
| `deleteBeforeReplace` | N/A (컴포넌트에 설정 불가) |
| `deletedWith` | 예 |
| `dependsOn` | 아니오 |
| `hideDiffs` | 아니오 |
| `hooks` | 아니오 |
| `ignoreChanges` | 아니오 |
| `import` | N/A (컴포넌트에 설정 불가) |
| `parent` | N/A |
| `protect` | 예 |
| `provider` | 예 |
| `providers` | 예 |
| `replaceOnChanges` | 아니오 |
| `replaceWith` | 아니오 |
| `replacementTrigger` | 아니오 |
| `retainOnDelete` | 예 |
| `transformations` | 예 |
| `transforms` | 예 |
| `version` | 아니오 |

> **참고:** 컴포넌트에 지원되지 않는 옵션을 적용하면 직접적인 효과가 없다. 자식 리소스에 `ignoreChanges` 같은 옵션을 적용하려면 컴포넌트의 `transforms` 옵션을 사용하여 자식 리소스 등록 시 해당 옵션을 주입하라.

---

## 리소스 수명주기

Pulumi 리소스는 다음 네 가지 주요 수명주기 작업을 거친다.

| 작업 | 설명 |
|---|---|
| **Create** | 새 리소스를 클라우드 프로바이더에 생성. 프로그램에 새 리소스가 선언되면 Pulumi가 프로바이더를 통해 리소스를 생성하고 물리적 ID를 반환받는다 |
| **Update** | 기존 리소스의 속성 변경. 리소스의 입력 속성이 변경되면 Pulumi는 프로바이더를 통해 인플레이스(in-place) 업데이트를 시도한다 |
| **Delete** | 리소스를 클라우드 프로바이더에서 삭제. 프로그램에서 리소스가 제거되면 Pulumi는 프로바이더를 통해 삭제를 수행한다. `protect` 옵션이 설정된 경우 삭제가 차단된다 |
| **Replace** | 기존 리소스를 삭제하고 새 리소스를 생성. 인플레이스 업데이트가 불가능한 속성(예: EC2 인스턴스의 AMI)이 변경된 경우 발생한다. 기본적으로 Pulumi는 새 리소스를 먼저 생성하고 기존 참조를 업데이트한 뒤 이전 리소스를 삭제하여 무중단을 보장한다 |

### 교체(Replace) 동작 제어

- **기본 동작**: 새 리소스 먼저 생성 -> 기존 참조 업데이트 -> 이전 리소스 삭제 (무중단)
- **`deleteBeforeReplace: true`**: 이전 리소스 먼저 삭제 -> 새 리소스 생성 (가동 중단 발생 가능, 명시적 물리적 이름 사용 시 충돌 방지)
- **`replaceOnChanges`**: 지정된 속성이 변경되면 강제로 교체 수행

> **주의:** 리소스의 논리적 이름을 변경하면 URN이 변경되어 교체가 아닌 **create + delete** 작업이 수행된다. 이름 변경 시 기존 리소스를 유지하려면 `aliases` 옵션을 사용하라.

---

## 스택(Stack)과 리소스

모든 Pulumi 프로그램은 **스택(Stack)** 에 배포된다. 스택은 Pulumi 프로그램의 격리되고 독립적으로 구성 가능한 인스턴스다. 스택은 일반적으로 개발 단계(`development`, `staging`, `production`)나 기능 브랜치(`feature-x-dev`)를 나타내는 데 사용된다.

### 주요 스택 명령

| 명령 | 설명 |
|---|---|
| `pulumi stack init <stackName>` | 새 스택 생성 |
| `pulumi stack ls` | 프로젝트의 스택 목록 조회 |
| `pulumi stack select <stackName>` | 활성 스택 변경 |
| `pulumi stack rename <new-name>` | 스택 이름 변경 |
| `pulumi up` | 활성 스택 업데이트 |
| `pulumi preview` | 변경 사항 미리보기 |
| `pulumi destroy` | 스택의 모든 리소스 삭제 |
| `pulumi stack` | 현재 스택의 메타데이터, 리소스, 출력 속성 조회 |

### 스택 이름 형식

스택 이름은 프로젝트 내에서 고유해야 하며, 영숫자, 하이픈, 밑줄, 마침표만 포함할 수 있다.

| 형식 | 설명 |
|---|---|
| `stackName` | 현재 사용자 또는 기본 조직의 스택 |
| `orgName/stackName` | 지정된 조직의 스택 |
| `orgName/projectName/stackName` | 지정된 조직과 프로젝트의 스택 |
