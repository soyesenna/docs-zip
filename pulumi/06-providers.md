# Pulumi 프로바이더

> **출처**
> - https://www.pulumi.com/docs/iac/concepts/providers/
> - https://www.pulumi.com/docs/iac/concepts/providers/dynamic-providers/
> - https://www.pulumi.com/docs/iac/concepts/providers/any-terraform-provider/
> - https://www.pulumi.com/docs/iac/concepts/plugins/
> - https://www.pulumi.com/docs/iac/concepts/functions/provider-functions/
> - https://www.pulumi.com/docs/iac/concepts/functions/
> - https://www.pulumi.com/docs/esc/
> - https://www.pulumi.com/registry/packages/aws/installation-configuration/
> - https://www.pulumi.com/registry/packages/azure-native/installation-configuration/
> - https://www.pulumi.com/registry/packages/gcp/installation-configuration/
> - https://www.pulumi.com/registry/packages/kubernetes/installation-configuration/

리소스 프로바이더는 클라우드 또는 SaaS 서비스와 통신하여 Pulumi 프로그램에서 정의한 리소스의 생성·조회·수정·삭제(CRUD)를 수행한다. 프로바이더는 두 부분으로 구성된다: 클라우드 프로바이더 API를 직접 호출하는 **실행 파일(executable)**과 Pulumi 프로그램 언어로 프로바이더를 사용할 수 있게 하는 **SDK**다. Pulumi는 프로그램을 실행할 때 언어 호스트(예: Node.js)에 코드를 전달하고, 리소스 등록 알림을 받아 원하는 상태 모델을 조립한 뒤 프로바이더 실행 파일을 호출해 실제 인프라를 프로비저닝한다.

---

## 프로바이더 유형

대부분의 Pulumi 프로바이더는 다음 두 가지 구현 방식 중 하나를 따른다.

| 유형 | 설명 | 예시 |
| - | - | - |
| **브릿지(Bridged) 프로바이더** | Terraform 또는 OpenTofu 프로바이더를 내부 의존성으로 사용하고, Pulumi Terraform Bridge를 통해 스키마를 변환 | AWS, Google Cloud(GCP) |
| **네이티브(Native) 프로바이더** | 클라우드/서비스의 API 스펙으로부터 리소스 정의와 CRUD 호출을 직접 생성. 별도의 브릿지 없이 동작 | Azure Native, Kubernetes |

추가로 다음 특수 유형이 존재한다.

| 유형 | 설명 |
| - | - |
| **파라미터화(Parameterized) 프로바이더** | 설치 시 파라미터를 받아 로컬 SDK를 생성. [Any Terraform Provider](https://www.pulumi.com/registry/packages/terraform-provider/)가 대표적이며, Azure Native도 특정 API 버전을 타겟하는 SDK 생성에 파라미터를 지원 |
| **다이나믹(Dynamic) 프로바이더** | TypeScript, Python에서만 지원. 별도 프로바이더 패키지 없이 Pulumi 프로그램 내에 커스텀 리소스 로직을 인라인으로 선언 |

[Pulumi Registry](https://www.pulumi.com/registry)는 공개 사용 가능한 프로바이더의 카탈로그다. 일부 조직은 자체 프라이빗 프로바이더를 유지하기도 하지만 비교적 드문 경우다. 더 일반적으로는 Pulumi를 사용하는 조직이 하나 이상의 프로바이더를 소비하는 컴포넌트를 만들어 재사용 가능한 추상화로 구성하고, 이를 **Pulumi IDP**(Internal Developer Platform)를 통해 조직 내에 배포하여 검색 가능하게 공유한다.

---

## 프로바이더 설치

프로바이더를 설치하고 사용하는 방법은 두 가지다.

### 패키지 매니저를 통한 설치

가장 일반적인 방법이다. 각 언어의 패키지 관리 도구(npm, PyPI, Go modules, NuGet, Maven)를 사용해 SDK를 설치한다. 최초 `pulumi preview` 또는 `pulumi up` 실행 시 Pulumi CLI가 플러그인 캐시에 없는 필수 프로바이더를 자동 설치한다.

| 프로바이더 | TypeScript | Python | Go | .NET | Java |
| - | - | - | - | - | - |
| **AWS** | `@pulumi/aws` | `pulumi-aws` | `github.com/pulumi/pulumi-aws/sdk/go/aws` | `Pulumi.Aws` | `com.pulumi.aws` |
| **Azure Native** | `@pulumi/azure-native` | `pulumi-azure-native` | `github.com/pulumi/pulumi-azure-native-sdk` | `Pulumi.AzureNative` | `com.pulumi.azurenative` |
| **GCP** | `@pulumi/gcp` | `pulumi-gcp` | `github.com/pulumi/pulumi-gcp/sdk/v7/go/gcp` | `Pulumi.Gcp` | `com.pulumi.gcp` |
| **Kubernetes** | `@pulumi/kubernetes` | `pulumi-kubernetes` | `github.com/pulumi/pulumi-kubernetes/sdk/v4/go/kubernetes` | `Pulumi.Kubernetes` | `com.pulumi/kubernetes` |

**TypeScript 설치 예시:**

```bash
npm install @pulumi/aws
```

**Python 설치 예시:**

```bash
pip install pulumi_aws
```

### `pulumi package add`를 통한 설치 (파라미터화 프로바이더)

Pulumi Registry에 사전 빌드된 SDK가 없거나 OpenTofu 레지스트리에만 존재하는 프로바이더를 사용할 때 활용한다. `pulumi package add` 명령은 로컬 SDK를 생성하고 `Pulumi.yaml`의 `packages` 키에 자동 추가한다.

```bash
# 기본: hashicorp/random 프로바이더 추가
pulumi package add terraform-provider hashicorp/random

# 버전 지정
pulumi package add terraform-provider hashicorp/random 3.7.1

# 로컬 프로바이더 바이너리 사용
pulumi package add terraform-provider /path/to/my/terraform-provider-binary
```

`Pulumi.yaml`에 자동 추가되는 항목:

```yaml
packages:
  random:
    source: terraform-provider
    version: 0.10.0
    parameters:
      - hashicorp/random
      - 3.7.1
```

`pulumi install` 명령으로 `Pulumi.yaml`에 정의된 모든 패키지를 설치할 수 있다.

```bash
pulumi install
```

> **참고:** 생성된 SDK에는 `.gitignore`가 포함되어 있어 SDK 코드를 버전 관리에 커밋해도 의존성은 제외된다. 프로바이더 바이너리는 프로젝트 디렉토리 외부의 공유 위치에 캐시되므로 한 번만 다운로드된다.

---

## 기본 프로바이더와 명시적 프로바이더

| 항목 | 기본(Default) 프로바이더 | 명시적(Explicit) 프로바이더 |
| - | - | - |
| **선언** | 프로그램에 선언하지 않음 | 프로그램에 명시적으로 선언 (Pulumi 리소스 자체) |
| **구성** | 환경 변수, 스택 설정 키(예: `aws:region`)에서 자동 읽기 | 선언 시 명시적으로 구성 값 전달 |
| **리소스 할당** | `provider` 옵션이 설정되지 않으면 자동 적용 | `provider` 리소스 옵션으로 명시적 할당 |
| **비활성화 가능** | 가능 | 불가능 |
| **적합한 경우** | 단일 프로바이더만 필요한 간단한 프로그램 | 다중 리전/환경 배포, 런타임에 생성된 클러스터에 배포 |

### 기본 프로바이더 구성

스택 설정에서 `pulumi config set`으로 기본 프로바이더를 구성한다. 구성 키 패턴은 `<프로바이더명>:<설정이름>`이다.

```bash
pulumi config set aws:region us-west-2
```

기본 프로바이더 구성은 항상 환경 변수 등 암시적 구성보다 우선한다.

**TypeScript 예시:**

```typescript
const aws = require("@pulumi/aws");

const instance = new aws.ec2.Instance("myInstance", {
    instanceType: "t2.micro",
    ami: "myAMI",
});
```

**Python 예시:**

```python
from pulumi_aws import ec2

instance = ec2.Instance("myInstance", instance_type="t2.micro", ami="myAMI")
```

**Go 예시:**

```go
import (
    "github.com/pulumi/pulumi-aws/sdk/v6/go/aws/ec2"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

instance, err := ec2.NewInstance(ctx, "myInstance", &ec2.InstanceArgs{
    InstanceType: pulumi.String("t2.micro"),
    Ami:          pulumi.String("myAMI"),
})
```

**.NET (C#) 예시:**

```csharp
using Pulumi;
using Aws = Pulumi.Aws;

var instance = new Aws.Ec2.Instance("myInstance", new Aws.Ec2.InstanceArgs
{
    InstanceType = "t2.micro",
    Ami = "myAMI",
});
```

**Java 예시:**

```java
import com.pulumi.Pulumi;
import com.pulumi.aws.ec2.Instance;
import com.pulumi.aws.ec2.InstanceArgs;

var instance = new Instance("myInstance", InstanceArgs.builder()
    .instanceType("t2.micro")
    .ami("myAMI")
    .build());
```

**YAML 예시:**

```yaml
resources:
  myInstance:
    type: aws:ec2:Instance
    properties:
      instanceType: t2.micro
      ami: myAMI
```

> **참고:** 시크릿 값은 반드시 `--secret` 플래그를 사용해 설정해야 한다: `pulumi config set --secret aws:secretKey <VALUE>`

### 명시적 프로바이더 구성

명시적 프로바이더는 Pulumi 리소스이므로 Pulumi 입력값(Input)을 구성 값으로 받는다. 이를 통해 런타임에 생성된 리소스의 출력값(Output)을 다른 프로바이더의 입력으로 전달하는 시나리오가 가능하다. 예를 들어 Kubernetes 클러스터를 생성한 직후 그 클러스터에 리소스를 배포하는 것이 가능하다.

**다중 리전 배포 예시 (us-east-1에 ACM 인증서 생성, 기본 리전에 ALB 리스너 생성):**

```bash
pulumi config set aws:region us-west-2
```

```typescript
const pulumi = require("@pulumi/pulumi");
const aws = require("@pulumi/aws");

// us-east-1 리전용 AWS 프로바이더 생성
const useast1 = new aws.Provider("useast1", { region: "us-east-1" });

// us-east-1에 ACM 인증서 생성
const cert = new aws.acm.Certificate("cert", {
    domainName: "foo.com",
    validationMethod: "EMAIL",
}, { provider: useast1 });

// 기본 리전(us-west-2)에 ALB 리스너 생성
const listener = new aws.lb.Listener("listener", {
    loadBalancerArn: loadBalancerArn,
    port: 443,
    protocol: "HTTPS",
    sslPolicy: "ELBSecurityPolicy-2016-08",
    certificateArn: cert.arn,
    defaultAction: {
        targetGroupArn: targetGroupArn,
        type: "forward",
    },
});
```

```python
import pulumi
import pulumi_aws as aws

# us-east-1 리전용 AWS 프로바이더 생성
useast1 = aws.Provider("useast1", region="us-east-1")

# us-east-1에 ACM 인증서 생성
cert = aws.acm.Certificate("cert",
    domain_name="foo.com",
    validation_method="EMAIL",
    opts=pulumi.ResourceOptions(provider=useast1))

# 기본 리전(us-west-2)에 ALB 리스너 생성
listener = aws.lb.Listener("listener",
    load_balancer_arn=load_balancer_arn,
    port=443,
    protocol="HTTPS",
    ssl_policy="ELBSecurityPolicy-2016-08",
    certificate_arn=cert.arn,
    default_action={
        "target_group_arn": target_group_arn,
        "type": "forward",
    })
```

**Go 예시:**

```go
import (
    "github.com/pulumi/pulumi-aws/sdk/v6/go/aws"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

// us-east-1 리전용 AWS 프로바이더 생성
useast1, err := aws.NewProvider(ctx, "useast1", &aws.ProviderArgs{
    Region: pulumi.String("us-east-1"),
})

// us-east-1에 ACM 인증서 생성
cert, err := aws.acm.NewCertificate(ctx, "cert", &aws.acm.CertificateArgs{
    DomainName:       pulumi.String("foo.com"),
    ValidationMethod: pulumi.String("EMAIL"),
}, pulumi.Provider(useast1))
```

**.NET (C#) 예시:**

```csharp
using Pulumi;
using Aws = Pulumi.Aws;

// us-east-1 리전용 AWS 프로바이더 생성
var useast1 = new Aws.Provider("useast1", new Aws.ProviderArgs
{
    Region = "us-east-1",
});

// us-east-1에 ACM 인증서 생성
var cert = new Aws.Acm.Certificate("cert", new Aws.Acm.CertificateArgs
{
    DomainName = "foo.com",
    ValidationMethod = "EMAIL",
}, new CustomResourceOptions { Provider = useast1 });
```

**YAML 예시:**

```yaml
resources:
  useast1:
    type: pulumi:providers:aws
    properties:
      region: us-east-1
  cert:
    type: aws:acm:Certificate
    properties:
      domainName: foo.com
      validationMethod: EMAIL
    options:
      provider: ${useast1}
```

> **참고:** Azure는 기본 리전과 무관하게 모든 리전 리소스에 접근 가능하므로, 다중 리전 배포 시 명시적 프로바이더가 필요하지 않다. 리소스 정의 자체에 리전을 지정하면 된다.

### 컴포넌트 리소스에 프로바이더 전달

컴포넌트 리소스는 `providers` 옵션으로 자식 리소스에 사용할 프로바이더 집합을 지정할 수 있다.

```typescript
class MyResource extends pulumi.ComponentResource {
    constructor(name, opts) {
        const instance = new aws.ec2.Instance("instance", { /* ... */ }, { parent: this });
        const pod = new kubernetes.core.v1.Pod("pod", { /* ... */ }, { parent: this });
    }
}

const useast1 = new aws.Provider("useast1", { region: "us-east-1" });
const myk8s = new kubernetes.Provider("myk8s", { context: "test-ci" });
const myResource = new MyResource("myResource", { providers: { aws: useast1, kubernetes: myk8s } });
```

```python
class MyResource(pulumi.ComponentResource):
    def __init__(self, name, opts):
        instance = aws.ec2.Instance("instance", opts=pulumi.ResourceOptions(parent=self))
        pod = kubernetes.core.v1.Pod("pod", opts=pulumi.ResourceOptions(parent=self))

useast1 = aws.Provider("useast1", region="us-east-1")
myk8s = kubernetes.Provider("myk8s", context="test-ci")
my_resource = MyResource("myResource", pulumi.ResourceOptions(providers={
    "aws": useast1,
    "kubernetes": myk8s,
}))
```

---

## 기본 프로바이더 비활성화

기본 프로바이더를 비활성화하면 모든 리소스에 명시적으로 `provider` 옵션을 설정해야 한다. 잘못된 환경에 리소스가 배포되는 것을 방지할 수 있다.

```bash
# AWS 기본 프로바이더 비활성화
pulumi config set --path 'pulumi:disable-default-providers[0]' aws

# Kubernetes도 추가 비활성화
pulumi config set --path 'pulumi:disable-default-providers[1]' kubernetes

# 모든 기본 프로바이더 비활성화
pulumi config set --path 'pulumi:disable-default-providers[0]' '*'
```

**Automation API를 사용한 비활성화:**

```typescript
await stack.setConfig("pulumi:disable-default-providers[0]", { value: "*" }, { path: true });
```

---

## Terraform 프로바이더 사용 (Any Terraform Provider)

Pulumi는 Terraform 또는 OpenTofu 레지스트리의 모든 프로바이더를 Pulumi 프로그램에서 사용할 수 있게 한다. 기본적으로 [OpenTofu 레지스트리](https://search.opentofu.org)에서 프로바이더를 가져오며, 이는 Terraform 레지스트리와 API 호환이다.

### 사용 가능한 시나리오

- Pulumi Registry에 없는 프로바이더 사용
- 조직 내부 커스텀 Terraform 프로바이더 사용
- Registry에 게시된 것과 다른 버전의 프로바이더 사용
- 프로바이더 도입 전 평가

### 워크플로우

```bash
# 1. 프로바이더 추가 (로컬 SDK 생성)
pulumi package add terraform-provider honeycombio/honeycombio

# 2. SDK 설치
pulumi install

# 3. 코드에서 사용 후 배포
pulumi up
```

**TypeScript 사용 예시:**

```typescript
import * as honeycombio from "@pulumi/honeycombio";

const marker = new honeycombio.Marker("deployment-marker", {
    message: "Deployed via Pulumi",
    dataset: "my-dataset",
});

export const markerId = marker.id;
```

**Python 사용 예시:**

```python
import pulumi_honeycombio as honeycombio

marker = honeycombio.Marker(
    "deployment-marker",
    message="Deployed via Pulumi",
    dataset="my-dataset",
)

pulumi.export("marker_id", marker.id)
```

**Go 사용 예시:**

```go
import (
    "github.com/pulumi/pulumi-terraform-provider/sdks/go/honeycombio"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        marker, err := honeycombio.NewMarker(ctx, "deployment-marker", &honeycombio.MarkerArgs{
            Message: pulumi.String("Deployed via Pulumi"),
            Dataset: pulumi.String("my-dataset"),
        })
        if err != nil {
            return err
        }
        ctx.Export("markerId", marker.ID())
        return nil
    })
}
```

**.NET (C#) 사용 예시:**

```csharp
using Pulumi;
using Pulumi.Honeycombio;

return await Deployment.RunAsync(() =>
{
    var marker = new Marker("deployment-marker", new MarkerArgs
    {
        Message = "Deployed via Pulumi",
        Dataset = "my-dataset",
    });

    return new Dictionary<string, object?>
    {
        ["markerId"] = marker.Id,
    };
});
```

**Java 사용 예시:**

```java
import com.pulumi.Pulumi;
import com.pulumi.honeycombio.Marker;
import com.pulumi.honeycombio.MarkerArgs;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {
            var marker = new Marker("deployment-marker", MarkerArgs.builder()
                .message("Deployed via Pulumi")
                .dataset("my-dataset")
                .build());

            ctx.export("markerId", marker.id());
        });
    }
}
```

**YAML 사용 예시:**

```yaml
name: honeycomb-example
runtime: yaml
resources:
  marker:
    type: honeycombio:Marker
    properties:
      message: Deployed via Pulumi
      dataset: my-dataset
outputs:
  markerId: ${marker.id}
packages:
  honeycombio:
    source: terraform-provider
    version: 0.10.0
    parameters:
      - honeycombio/honeycombio
```

### 버전 관리 권장 사항

- **버전 고정:** 재현 가능한 빌드를 위해 항상 버전을 명시하라.
- **`pulumi install` 사용:** 표준 패키지 매니저 의존성과 로컬 패키지를 모두 처리한다.

### 버전 관리 고려 사항 (Version Control Considerations)

생성된 SDK 디렉토리를 버전 관리에 커밋할지 여부를 선택할 수 있다.

| 접근 방식 | 장점 | 단점 |
| - | - | - |
| **SDK 디렉토리 커밋** | 팀원 및 CI/CD 파이프라인의 빠른 설정. 생성된 SDK에는 의존성을 제외하는 `.gitignore`가 포함됨 | 저장소 크기 증가 |
| **SDK 디렉토리 미커밋** | 저장소 크기 최소화 | 팀원이 로컬에서 SDK를 직접 생성해야 함 |

프로바이더 바이너리는 항상 프로젝트 디렉토리 외부의 공유 위치에 다운로드되어 캐시되므로, 머신당 한 번만 다운로드된다.

팀원이 저장소를 클론한 후 실행해야 할 명령:

```bash
pulumi install
```

> **참고:** `pulumi install`로 프로젝트 파일에 정의된 패키지를 설치하는 경우, 생성된 SDK 파일을 버전 관리에서 제거하고 SDK 디렉토리를 `.gitignore`에 추가해야 한다. 그렇지 않으면 생성된 파일이 여전히 버전 관리 대상으로 남게 된다.

---

## 다이나믹 프로바이더

다이나믹 프로바이더는 TypeScript와 Python에서만 지원되며, 별도 프로바이더 패키지를 작성하거나 설치하지 않고도 커스텀 리소스를 정의할 수 있는 가벼운 메커니즘이다.

> **작성 전 결정 가이드:** 다이나믹 프로바이더가 항상 최적의 도구는 아니다. 기존 프로바이더, [Any Terraform Provider](https://www.pulumi.com/docs/iac/concepts/providers/any-terraform-provider/), 또는 [Command Provider](https://www.pulumi.com/registry/packages/command/)가 더 적합한 경우가 많다. 작성 전에 공식 결정 다이어그램을 참조하여 실제로 다이나믹 프로바이더가 필요한지 확인하라.

### 다이나믹 프로바이더 vs 대안

| 접근 방식 | 적합한 경우 | 지원 언어 |
| - | - | - |
| **기존 프로바이더** | Pulumi Registry에 원하는 클라우드/SaaS 프로바이더가 이미 존재 | 모든 언어 |
| **Any Terraform Provider** | Terraform/OpenTofu 레지스트리에만 프로바이더가 존재 | 모든 언어 |
| **Command Provider** | 로컬 명령어 실행이나 간단한 스크립트로 리소스를 관리하고자 할 때 | 모든 언어 |
| **다이나믹 프로바이더** | 위 어느 것도 해당하지 않고, 간단한 API를 위한 로컬 커스텀 리소스가 필요할 때 | TypeScript, Python |

### 지원 메서드

| 메서드 | 필수 | 설명 |
| - | - | - |
| `create` | **필수** | 리소스 생성 |
| `diff` | 선택 | 변경 사항 감지 및 대체(replacement) 필요 여부 판단 |
| `update` | 선택 | 기존 리소스 업데이트 |
| `delete` | 선택 | 리소스 삭제 |
| `read` | 지원 안 함 | 현재 기능하지 않음 (`pulumi import` 불가). [pulumi/pulumi#16175](https://github.com/pulumi/pulumi/issues/16175)에서 추적 중 |
| `check` | 선택 | 리소스 인수 유효성 검증 |

### 작동 방식

다이나믹 프로바이더는 배포 프로세스에 임의의 코드를 직접 포함할 수 있는 유연하고 저수준의 메커니즘이다. Pulumi 프로그램의 대부분의 코드는 리소스 그래프가 구성되는 동안(즉, 원하는 상태가 정의될 때) 실행되지만, 다이나믹 프로바이더 구현 내부의 코드(`create`, `update` 등)는 리소스 프로비저닝 중, 즉 리소스 그래프가 클라우드 프로바이더에 대해 예약된 CRUD 작업 세트로 변환될 때 실행된다.

이 두 실행 단계는 **별도의 프로세스**에서 실행된다. `new MyResource()` 구성은 Pulumi 프로그램 내의 언어별 프로세스에서 발생한다. 반면 `create`나 `update`의 구현은 `pulumi-resource-pulumi-nodejs`와 같은 특수 리소스 프로바이더 바이너리에 의해 실행되며, 이 바이너리는 Pulumi 리소스 프로바이더 gRPC 인터페이스를 실제로 구현하고 Pulumi 엔진과 직접 통신한다.

리소스 프로바이더 인터페이스 구현은 다른 프로세스에서, 잠재적으로 다른 시점에 사용되어야 하므로, 다이나믹 프로바이더는 AWS Lambda나 Google Cloud Functions로 콜백을 전환하는 데 사용되는 것과 동일한 **함수 직렬화** 위에 구축된다. 이 직렬화로 인해 리소스 프로바이더 인터페이스 구현 내부에서 수행할 수 있는 작업에 제한이 있다.

### 최소 구현 예시

**TypeScript:**

```typescript
const myProvider: pulumi.dynamic.ResourceProvider = {
    async create(inputs) {
        return { id: "foo", outs: {} };
    }
};

class MyResource extends pulumi.dynamic.Resource {
    constructor(name: string, props: {}, opts?: pulumi.CustomResourceOptions) {
        super(myProvider, name, props, opts);
    }
}
```

**Python:**

```python
from pulumi.dynamic import ResourceProvider, CreateResult, Resource
from pulumi import ResourceOptions
from typing import Any, Optional

class MyProvider(ResourceProvider):
    def create(self, inputs):
        return CreateResult(id_="foo", outs={})

class MyResource(Resource):
    def __init__(self, name: str, props: Any, opts: Optional[ResourceOptions] = None):
        super().__init__(MyProvider(), name, props, opts)
```

### 제한 사항

| 제한 | 내용 |
| - | - |
| 단일 언어 | 다이나믹 프로바이더는 작성된 언어와 동일한 언어의 프로그램에서만 사용 가능 |
| `read` 미지원 | `pulumi import` 및 `get` 정적 메서드 사용 불가 |
| 함수 직렬화 제한 | 프로바이더 메서드는 별도 프로세스에서 실행되므로 캡처할 수 있는 코드에 제한 있음 |
| pnpm 미지원 (TypeScript) | pnpm 패키지 매니저와 호환되지 않음. npm 또는 yarn 사용 |
| Bun 런타임 미지원 (TypeScript) | Bun은 Node.js v8/inspector API를 완전히 구현하지 않아 함수 직렬화 불가 |
| 정책 작성의 어려움 | 모든 다이나믹 리소스가 동일한 타입(`pulumi-nodejs:dynamic:Resource` 또는 `pulumi-python:dynamic:Resource`)을 공유 |

---

## 프로바이더 함수 (Provider Functions / Data Sources)

> **출처:** https://www.pulumi.com/docs/iac/concepts/functions/provider-functions/

프로바이더는 리소스 타입뿐 아니라 **프로바이더 함수(provider functions)**도 SDK에 노출할 수 있다. 프로바이더 함수는 클라우드 API를 호출하여 리소스의 일부가 아닌 값을 가져올 때 사용한다. 브릿지 프로바이더의 경우, 업스트림 Terraform 프로바이더의 데이터 소스(data sources)가 해당 Pulumi 프로바이더의 프로바이더 함수로 노출된다.

### 함수 유형 개요

Pulumi는 세 가지 함수 유형을 제공한다.

| 유형 | 설명 | 사용 사례 |
| - | - | - |
| **프로바이더 함수** | 클라우드 프로바이더 API를 쿼리하여 리소스가 아닌 데이터를 조회 | 최신 AMI ID 조회,可用 가용 영역 목록 확인 |
| **Get 함수** | Pulumi로 관리되지 않는 기존 리소스의 속성을 참조 | 기존 VNet ID로 CIDR 블록 조회 |
| **리소스 메서드** | Pulumi로 관리 중인 리소스 인스턴스에서 파생된 값 반환 | 관리형 Kubernetes 클러스터의 kubeconfig 생성 |

### 직접 형식(Direct Form)과 출력 형식(Output Form)

프로바이더 함수는 각 언어에서 두 가지 형식으로 노출된다. 대부분의 언어에서는 단일 함수의 오버로드가 아닌 별도의 이름을 가진 두 함수로 제공된다.

| 언어 | 직접 형식 (Plain) | 출력 형식 (Output) |
| - | - | - |
| **TypeScript** | `getX()` — `Promise<T>` 반환 | `getXOutput()` — `Output<T>` 반환 |
| **Python** | `get_x()` — 동기 결과 반환 | `get_x_output()` — `Output[T]` 반환 |
| **Go** | `LookupX()` / `GetX()` — `(T, error)` 반환 | `LookupXOutput()` / `GetXOutput()` 반환 |
| **.NET** | `GetX.InvokeAsync()` — `Task<T>` 반환 | `GetX.Invoke()` — `Output<T>` 반환 |
| **Java** | `ModuleFunctions.getXPlain()` — `CompletableFuture<T>` 반환 | `ModuleFunctions.getX()` — `Output<T>` 반환 |
| **YAML** | `fn::invoke` (두 형식 모두 동일) | `fn::invoke` (런타임이 자동 처리) |

### Invoke 옵션

프로바이더 함수는 함수 인수 외에도 리소스 옵션과 유사한 **invoke 옵션**을 받는다.

| 옵션 | 설명 | 사용 가능한 형식 |
| - | - | - |
| `dependsOn` | 이 함수가 의존하는 리소스 배열. 선행 리소스가 준비된 후에 실행 보장 | 출력 형식만 |
| `parent` | 함수 호출의 부모 리소스. provider 결정 시 참조 | 두 형식 모두 |
| `provider` | 기본 프로바이더 대신 사용할 명시적으로 구성된 프로바이더. 예: 여러 AWS 리전에서 함수를 호출할 때 | 두 형식 모두 |

### 사용 예시

**TypeScript — 최신 AMI 조회:**

```typescript
import * as aws from "@pulumi/aws";

(async () => {
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
})();
```

**Python — 최신 AMI 조회:**

```python
import pulumi_aws as aws

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

**Go — 최신 AMI 조회:**

```go
package main

import (
    "github.com/pulumi/pulumi-aws/sdk/v6/go/aws/ec2"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        latestAmi, err := ec2.LookupAmi(ctx, &ec2.LookupAmiArgs{
            Owners:     []string{"amazon"},
            MostRecent: pulumi.BoolRef(true),
            Filters: []ec2.GetAmiFilter{
                {Name: "name", Values: []string{"amzn2-ami-hvm-*"}},
                {Name: "architecture", Values: []string{"x86_64"]},
            },
        }, nil)
        if err != nil {
            return err
        }

        _, err = ec2.NewInstance(ctx, "web-server", &ec2.InstanceArgs{
            Ami:          pulumi.String(latestAmi.ImageId),
            InstanceType: pulumi.String("t3.micro"),
        })
        return err
    })
}
```

**YAML — 최신 AMI 조회:**

```yaml
variables:
  latestAmi:
    fn::invoke:
      function: aws:ec2/getAmi:getAmi
      arguments:
        filters:
          - name: name
            values: ["amzn2-ami-hvm-*"]
          - name: architecture
            values: ["x86_64"]
        owners: ["amazon"]
        mostRecent: true

resources:
  myInstance:
    type: aws:ec2:Instance
    properties:
      ami: ${latestAmi.imageId}
      instanceType: "t3.micro"
```

---

## Pulumi ESC를 통한 프로바이더 자격 증명 관리

> **출처:** https://www.pulumi.com/docs/esc/

**Pulumi ESC**(Environments, Secrets, and Configuration)는 프로바이더 자격 증명을 중앙 집중으로 관리하는 메커니즘이다. ESC를 사용하면 로컬에 자격 증명 파일을 보관할 필요 없이 OIDC를 통해 단기 자격 증명을 동적으로 생성할 수 있다.

### 지원 클라우드

| 클라우드 | ESC 로그인 프로바이더 | 설명 |
| - | - | - |
| **AWS** | OIDC → `sts:AssumeRoleWithWebIdentity` | GitHub, GitLab 등에서 AWS 임시 자격 증명 획득 |
| **Azure** | OIDC → Service Principal | GitHub, GitLab, Azure DevOps 등에서 인증 |
| **GCP** | OIDC → 서비스 계정 가장 | 동적 자격 증명 생성 |
| **기타** | AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, HashiCorp Vault, 1Password | 외부 시크릿 스토어에서 시크릿 풀 및 동기화 |

### ESC 환경 파일 예시

```yaml
# ESC 환경 파일 (예: my-org/aws-production)
values:
  aws:
    login:
      fn::open::aws-login:
        oidc:
          durationHours: 1
          sessionName: pulumi-session
          roleArn: arn:aws:iam::123456789012:role/PulumiOIDCRole
  environmentVariables:
    AWS_ACCESS_KEY_ID: ${aws.login.accessKeyId}
    AWS_SECRET_ACCESS_KEY: ${aws.login.secretAccessKey}
    AWS_SESSION_TOKEN: ${aws.login.sessionToken}
    AWS_REGION: us-west-2
```

### Pulumi IaC에서 ESC 환경 사용

`Pulumi.yaml`에서 ESC 환경을 참조하면 프로바이더 구성 값을 자동으로 주입받을 수 있다.

```yaml
# Pulumi.yaml
name: my-project
runtime: nodejs
environment:
  - my-org/aws-production
```

ESC SDK(Node.js, Python, Go, .NET)를 사용하여 런타임에 환경 값을 검색할 수도 있다. 단, Pulumi IaC 프로그램에서 환경을 소비할 때는 ESC SDK 대신 `config`를 사용한다.

---

## 플러그인 관리

프로바이더는 Pulumi 플러그인의 한 종류(리소스 플러그인)다. 대부분의 사용자는 플러그인을 직접 관리할 필요가 없다.

플러그인은 Pulumi의 핵심 확장 메커니즘이며, Pulumi 엔진이 다양한 언어, 리소스 프로바이더, 기타 도구와 균일한 방식으로 통신할 수 있게 한다. 플러그인은 항상 별도의 프로세스로 실행되며, 주로 gRPC를 사용하여 Pulumi 엔진과 통신한다.

Pulumi Registry의 패키지 외에도 자체 컴포넌트를 작성하여 리소스 플러그인으로 배포할 수 있으며, 이를 통해 모든 Pulumi 언어에서 소비할 수 있다. 컴포넌트는 조직 내 검색 가능하도록 **Pulumi IDP**에 게시하거나 Git 참조를 통해 직접 공유할 수 있다.

### 플러그인 유형

| 유형 | 설명 | 설치 방식 |
| - | - | - |
| **리소스 플러그인** | 클라우드 리소스 관리 (프로바이더) | `pulumi preview`/`pulumi up` 시 자동 설치 |
| **언어 플러그인** | 특정 언어로 작성된 Pulumi 프로그램 실행 | Pulumi CLI와 함께 자동 설치 |
| **분석기 플러그인** | 정책 검사 및 Policy as Code | Pulumi CLI와 함께 자동 설치 |
| **변환기 플러그인** | Terraform, CloudFormation 등을 Pulumi로 변환 | `pulumi convert` 실행 시 자동 설치 |
| **도구 플러그인** | 외부 도구와의 통합 | 개별 설치 |

### 플러그인 저장 위치

| 위치 | 내용 |
| - | - |
| `~/.pulumi/bin` | Pulumi CLI에 포함된 언어 플러그인, 정책 플러그인 |
| `~/.pulumi/plugins` | 자동 또는 수동으로 설치된 모든 플러그인 캐시 |

### 플러그인 관리 CLI 명령

| 명령 | 설명 |
| - | - |
| `pulumi plugin ls` | 설치된 플러그인 목록 |
| `pulumi plugin rm` | 캐시된 플러그인 제거 |
| `pulumi plugin install` | 플러그인 수동 설치 |

수동 설치는 CI/CD Docker 이미지 사전 로딩, 에어갭 환경, 프리릴리스 테스트 등에 유용하다.

### 플러그인 구현 방식

플러그인은 두 가지 접근 방식으로 배포된다.

| 방식 | 설명 |
| - | - |
| **실행 파일(Executable)** | `pulumi-<kind>-<name>` 명명 규칙을 따르는 바이너리 (예: `pulumi-resource-aws`) |
| **소스 기반(Source-based)** | `PulumiPlugin.yaml` 구성 파일을 포함하며, 엔진이 지정된 런타임을 사용하여 언어 플러그인 인터페이스를 통해 플러그인을 실행 |

### PulumiPlugin.yaml 참조

소스 기반 플러그인은 `PulumiPlugin.yaml`(또는 `PulumiPlugin.yml`) 파일로 플러그인 실행 방식을 구성한다. 파일명은 대소문자를 구분한다.

| 속성 | 필수 | 설명 |
| - | - | - |
| `runtime` | 필수 | 플러그인 실행에 사용할 언어 런타임: `nodejs`, `python`, `go`, `dotnet`, `java`, `yaml`, `bun` |
| `packages` | 선택 | 플러그인에서 사용할 추가 패키지. `Pulumi.yaml`의 `packages`와 동일 |
| `requiredPulumiVersion` | 선택 | 이 플러그인이 요구하는 Pulumi CLI 버전 범위. `Pulumi.yaml`의 `requiredPulumiVersion`과 동일 |

**런타임 옵션:**

`runtime`은 문자열 또는 추가 옵션이 있는 객체로 지정할 수 있다.

```yaml
# 단순 지정
runtime: nodejs

# 옵션 포함 지정
runtime:
  name: nodejs
  options:
    packagemanager: yarn
```

| 런타임 | 옵션 | 설명 |
| - | - | - |
| **Node.js** | `nodeargs` | node 실행 시 전달할 인수 |
| | `packagemanager` | 패키지 매니저: `npm`(기본값), `pnpm`, `yarn`, `bun` |
| **Python** | `toolchain` | 가상환경 관리 도구: `pip`(기본값), `poetry`, `uv` |
| | `virtualenv` | 가상환경 경로 (`pip` 또는 `uv` toolchain에만 적용) |
| **.NET** | `binary` | 사전 빌드된 실행 파일 경로 |
| | `use-executor` | dotnet 바이너리 경로 오버라이드 |
| **Java** | `binary` | 사전 빌드된 실행 파일 경로 |
| | `use-executor` | Java 실행 방식 오버라이드 (`gradle`, `mvn` 등) |
| **Go / Bun / YAML** | — | 추가 런타임 옵션 없음 |

**예시:**

```yaml
# Python + uv toolchain
runtime:
  name: python
  options:
    toolchain: uv
    virtualenv: venv

# .NET + 커스텀 바이너리 경로
runtime:
  name: dotnet
  options:
    use-executor: /opt/homebrew/bin/dotnet

# 버전 요구사항 포함
runtime: nodejs
requiredPulumiVersion: ">=3.100.0"
```

---

## 프로바이더별 인증 방법

### AWS 인증

| 방법 | 환경 변수 / 설정 | 설명 |
| - | - | - |
| **환경 변수** | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` | CI/CD 등에서 빠른 오버라이드에 적합 |
| **공유 자격 증명 파일** | `~/.aws/credentials` | `aws configure`로 생성. 다중 프로파일 지원 |
| **EC2 인스턴스 메타데이터** | `aws:skipMetadataApiCheck false` | EC2 인스턴스에서 IAM 역할 자동 사용 |
| **WebIdentity / OIDC** | `aws:assumeRoleWithWebIdentity` | GitHub, GitLab 등에서 OIDC로 임시 자격 증명 획득 |
| **Pulumi ESC (동적 자격 증명)** | ESC 환경 파일 + OIDC | 자격 증명을 중앙 집중 관리. 로컬 자격 증명 불필요 |

**환경 변수 예시:**

```bash
export AWS_ACCESS_KEY_ID=<YOUR_ACCESS_KEY_ID>
export AWS_SECRET_ACCESS_KEY=<YOUR_SECRET_ACCESS_KEY>
export AWS_REGION=us-west-2
```

**스택 설정 예시:**

```bash
pulumi config set aws:region us-west-2
pulumi config set aws:profile <YOUR_PROFILE_NAME>
```

### Azure Native 인증

| 방법 | 구성 키 / 환경 변수 | 설명 |
| - | - | - |
| **Azure CLI** | `az login` | 로컬 개발 시 권장. Pulumi가 자동으로 자격 증명 사용 |
| **Default Azure Credential** | `azure-native:useDefaultAzureCredential` / `ARM_USE_DEFAULT_AZURE_CREDENTIAL` | 다양한 환경의 자격 증명 체인 자동 탐지. 팀/CI 환경에 권장 |
| **Service Principal (클라이언트 시크릿)** | `azure-native:clientId`, `azure-native:clientSecret`, `azure-native:tenantId`, `azure-native:subscriptionId` / `ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_TENANT_ID`, `ARM_SUBSCRIPTION_ID` | 비대화형 환경에서 사용 |
| **Service Principal (인증서)** | `clientCertPath` / `ARM_CLIENT_CERTIFICATE_PATH` | 클라이언트 시크릿 대신 인증서 사용 |
| **OIDC** | `azure-native:useOidc` / `ARM_USE_OIDC` | GitHub, GitLab, Azure DevOps 등에서 OIDC로 인증 |
| **Managed Service Identity (MSI)** | `azure-native:useMsi` / `ARM_USE_MSI` | Azure 호스팅 환경에서 관리 ID 사용 |
| **Pulumi ESC (동적 자격 증명)** | ESC 환경 파일 + OIDC | 자격 증명 중앙 집중 관리 |

**Service Principal 스택 설정 예시:**

```bash
pulumi config set azure-native:clientId <CLIENT_ID>
pulumi config set azure-native:clientSecret <CLIENT_SECRET> --secret
pulumi config set azure-native:tenantId <TENANT_ID>
pulumi config set azure-native:subscriptionId <SUBSCRIPTION_ID>
pulumi config set azure-native:location koreacentral
```

### GCP 인증

| 방법 | 환경 변수 / 설정 | 설명 |
| - | - | - |
| **Google Cloud CLI** | `gcloud auth application-default login` | 로컬 개발 시 권장 |
| **서비스 계정 키** | `gcp:credentials` / `GOOGLE_APPLICATION_CREDENTIALS` | CI/CD 등 비대화형 환경에서 사용 |
| **액세스 토큰** | `gcp:accessToken` / `GOOGLE_OAUTH_ACCESS_TOKEN` | 임시 OAuth 2.0 액세스 토큰 |
| **OIDC / Pulumi ESC** | ESC 환경 파일 + OIDC | 동적 자격 증명 생성. 로컬 자격 증명 불필요 |
| **서비스 계정 가장** | `gcp:impersonateServiceAccount` / `GOOGLE_IMPERSONATE_SERVICE_ACCOUNT` | 다른 서비스 계정 권한으로 작업 |

**CLI 인증 예시:**

```bash
gcloud auth application-default login
pulumi config set gcp:project your-gcp-project-id
```

**환경 변수 예시:**

```bash
export GOOGLE_PROJECT=your-gcp-project-id
export GOOGLE_REGION=asia-northeast3
```

### Kubernetes 인증

| 방법 | 구성 키 / 환경 변수 | 설명 |
| - | - | - |
| **기본 kubeconfig** | `$KUBECONFIG` 또는 `~/.kube/config` | `kubectl`과 동일한 방식. 기본적으로 자동 감지 |
| **컨텍스트 지정** | `kubernetes:context` | kubeconfig 내 특정 컨텍스트 선택 |
| **명시적 kubeconfig 전달** | `new kubernetes.Provider`의 `kubeconfig` 인수 | 런타임에 생성된 클러스터에 배포 시 사용 |

**컨텍스트 설정 예시:**

```bash
pulumi config set kubernetes:context my-context
```

**명시적 프로바이더로 kubeconfig 전달 (TypeScript):**

```typescript
const k8sProvider = new kubernetes.Provider("k8s", {
    kubeconfig: cluster.kubeconfig,  // 다른 리소스의 Output
});
```

**명시적 프로바이더로 kubeconfig 전달 (Python):**

```python
k8s_provider = kubernetes.Provider("k8s",
    kubeconfig=cluster.kubeconfig,
)
```

**명시적 프로바이더로 kubeconfig 전달 (Go):**

```go
k8sProvider, err := kubernetes.NewProvider(ctx, "k8s", &kubernetes.ProviderArgs{
    Kubeconfig: cluster.Kubeconfig,
})
```

**명시적 프로바이더로 kubeconfig 전달 (.NET):**

```csharp
var k8sProvider = new Kubernetes.Provider("k8s", new Kubernetes.ProviderArgs
{
    Kubeconfig = cluster.Kubeconfig,
});
```

**명시적 프로바이더로 kubeconfig 전달 (YAML):**

```yaml
resources:
  k8s:
    type: pulumi:providers:kubernetes
    properties:
      kubeconfig: ${cluster.kubeconfig}
```

> **참고:** Pulumi는 인증 시크릿이나 자격 증명을 절대 Pulumi Cloud로 전송하지 않는다. 모든 인증은 로컬에서 각 클라우드 SDK를 통해 이루어진다.

---

## 주요 프로바이더 구성 옵션

### AWS 구성 옵션

| 옵션 | 필수 | 설명 |
| - | - | - |
| `region` | 필수 | AWS 리전 (예: `us-east-1`, `ap-northeast-2`) |
| `accessKey` | 선택 | 액세스 키 |
| `secretKey` | 선택 | 시크릿 키 |
| `token` | 선택 | MFA 토큰 / 세션 토큰 (`AWS_SESSION_TOKEN`) |
| `profile` | 선택 | `aws configure`로 생성한 프로파일명 |
| `assumeRoles` | 선택 | IAM 역할 체인 위임 (JSON 배열) |
| `assumeRoleWithWebIdentity` | 선택 | OIDC 기반 웹 아이덴티티 역할 위임 |
| `defaultTags` | 선택 | 모든 리소스에 적용할 기본 태그 |
| `ignoreTags` | 선택 | 무시할 태그 키/접두사 |
| `maxRetries` | 선택 | API 요청 최대 재시도 횟수 |
| `retryMode` | 선택 | 재시도 방식 (`standard`, `adaptive`) |
| `allowedAccountIds` | 선택 | 허용된 AWS 계정 ID 목록 (잘못된 계정 배포 방지) |
| `forbiddenAccountIds` | 선택 | 금지된 AWS 계정 ID 목록 |
| `insecure` | 선택 | SSL 검증 비활성화 (기본값: `false`) |
| `skipCredentialsValidation` | 선택 | STS를 통한 자격 증명 검증 건너뛰기 |
| `skipMetadataApiCheck` | 선택 | 메타데이터 API 확인 건너뛰기 |
| `skipRegionValidation` | 선택 | 리전 이름 정적 검증 건너뛰기 |
| `sharedCredentialsFile` | 선택 | 공유 자격 증명 파일 경로 (기본: `~/.aws/credentials`) |
| `s3ForcePathStyle` | 선택 | S3 path-style 강제 사용 |
| `dynamodbEndpoint` | 선택 | DynamoDB 엔드포인트 오버라이드 |
| `kinesisEndpoint` | 선택 | Kinesis 엔드포인트 오버라이드 |

### GCP 구성 옵션

| 옵션 | 필수 | 설명 |
| - | - | - |
| `project` | 필수 | GCP 프로젝트 ID |
| `region` | 선택 | 기본 리전 |
| `zone` | 선택 | 기본 존 |
| `credentials` | 선택 | 서비스 계정 키 JSON 파일 경로 또는 내용 |
| `accessToken` | 선택 | 임시 OAuth 2.0 액세스 토큰 |
| `scopes` | 선택 | OAuth 2.0 스코프 목록 |
| `impersonateServiceAccount` | 선택 | 가장할 서비스 계정 이메일 |

### Azure Native 구성 옵션

| 옵션 | 설명 |
| - | - |
| `subscriptionId` | 구독 ID |
| `tenantId` | 테넌트 ID (OIDC/Service Principal) |
| `clientId` | 클라이언트 ID (OIDC/Service Principal/MSI) |
| `clientSecret` | 클라이언트 시크릿 (Service Principal) |
| `clientCertificatePath` | 클라이언트 인증서 경로 (Service Principal) |
| `clientCertificatePassword` | 인증서 비밀번호 |
| `location` | 기본 위치(리전) |
| `environment` | 클라우드 환경 (`public`, `usgovernment`, `china`) |
| `useOidc` | OIDC 인증 사용 여부 |
| `useMsi` | MSI 인증 사용 여부 |
| `useDefaultAzureCredential` | Default Azure Credential 사용 여부 |
| `oidcToken` | OIDC 토큰 |
| `oidcTokenFilePath` | OIDC 토큰 파일 경로 |
| `auxiliaryTenantIds` | 추가 테넌트 ID |

> 모든 Azure Native 구성 옵션은 선택 사항이며, 해당 환경 변수로도 설정 가능하다 (예: `ARM_SUBSCRIPTION_ID`, `ARM_CLIENT_ID` 등).
