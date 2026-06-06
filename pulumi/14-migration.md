# Pulumi 마이그레이션 가이드

> 출처
> - https://www.pulumi.com/docs/iac/guides/migration/
> - https://www.pulumi.com/docs/iac/get-started/terraform/
> - https://www.pulumi.com/docs/iac/comparisons/
> - https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-terraform/
> - https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-cloudformation/
> - https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-arm/
> - https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-kubernetes/
> - https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-serverless/
> - https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-cdk/
> - https://www.pulumi.com/docs/iac/comparisons/terraform/
> - https://www.pulumi.com/docs/iac/comparisons/cloudformation/
> - https://www.pulumi.com/docs/iac/comparisons/aws-cdk/
> - https://www.pulumi.com/docs/iac/comparisons/cdktf/
> - https://www.pulumi.com/docs/iac/comparisons/arm-templates/
> - https://www.pulumi.com/docs/iac/comparisons/k8s-yaml-dsls/
> - https://www.pulumi.com/docs/iac/comparisons/serverless/

이미 실행 중인 기존 인프라가 있는 경우에도 Pulumi를 채택할 수 있다. 신규 프로젝트는 처음부터 Pulumi로 시작하면 되지만, 이미 다른 도구로 프로비저닝된 인프라도 마이그레이션이 가능하다. Pulumi는 **공존(coexistence)**, **임포트(importing)**, **변환(conversion)** 세 가지 기본 접근 방식을 제공하며, 각 소스 도구마다 지원 수준이 다르다.

---

## 마이그레이션 세 가지 핵심 개념

| 개념 | 설명 | 기존 인프라 영향 |
|------|------|------------------|
| **공존(Coexistence)** | 기존 도구로 관리되는 인프라를 그대로 두고, 새 인프라만 Pulumi로 관리 | 없음 (읽기 전용 참조) |
| **임포트(Importing)** | 이미 프로비저닝된 클라우드 리소스를 Pulumi 관리로 편입 | Pulumi가 이후 관리 권한을 가짐 |
| **변환(Conversion)** | 기존 IaC 코드를 Pulumi 코드로 자동 변환 | 코드 구조 보존, 상태도 함께 이관 가능 |

### 소스별 지원 매트릭스

| 소스 도구 | 공존 | 임포트 | 변환 |
|-----------|:----:|:------:|:----:|
| Terraform | O | O | O |
| AWS CloudFormation | O | O | O |
| Azure ARM Templates / Bicep | O | O | O |
| Kubernetes YAML / Helm | O | O | O |
| Serverless Framework | O | O | O |
| AWS CDK | O | O | O |
| CDKTF (deprecated) | O | O | O |
| 기타 (수동 프로비저닝 등) | O | O | - |

---

## 공존 패턴 상세

공존은 기존 인프라를 현재 도구로 계속 관리하면서, 새 인프라만 Pulumi로 프로비저닝하는 방식이다. 기존 인프라를 수정하거나 삭제하지 않고 읽기 전용으로 참조한다.

### 공존에 사용되는 기법

| 기법 | 설명 |
|------|------|
| **Resource Getters** | 모든 리소스에서 사용 가능한 `get` 함수로, 리소스 ID만으로 클라우드 제공자에서 상세 정보를 읽어옴 |
| **Stack References** | 다른 Pulumi 스택의 출력값을 참조하여 입력으로 사용 |
| **External State References** | Terraform state, CloudFormation 스택, ARM 배포 등 비-Pulumi 소스의 출력값을 참조 |

---

## Terraform에서 Pulumi로 마이그레이션

> 출처: https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-terraform/

### 마이그레이션 경로 선택

| 경로 | 방식 | 추천 대상 |
|------|------|-----------|
| **Pulumi Neo (권장)** | 자동 변환 + 리소스 임포트 + `pulumi preview` 검증 | 대부분의 경우 |
| **State-first 마이그레이션** | `pulumi-terraform-migrate`로 상태 이관 후 LLM으로 코드 변환 | Neo 미보유 시 |
| **공존** | `.tfstate` 파일 참조 | 점진적 전환 |
| **변환** | `pulumi convert --from terraform` | HCL 코드 변환 |
| **임포트** | `pulumi import --from terraform` | 기존 리소스 편입 |
| **Terraform Module 직접 사용** | `pulumi package add terraform-module` | 모듈 자산 보존 |

### Pulumi Neo를 사용한 자동 마이그레이션

1. **전제조건**: `.tfstate` 파일 접근 권한, Pulumi GitHub app 설치, 클라우드 자격 증명을 Pulumi ESC에 구성, Pulumi Neo 접근 권한
2. **마이그레이션 시작**: Neo에게 `"Migrate my Terraform configuration to Pulumi"` 지시
3. **Neo 수행 작업**: Terraform state를 Pulumi state로 변환, 동등한 Pulumi 코드 생성, `pulumi preview`로 변경 없음을 검증
4. **검토 및 커밋**: 생성된 Pulumi 코드 확인 후 커밋

### State-first 마이그레이션 (`pulumi-terraform-migrate`)

```bash
# 1. Pulumi 프로젝트 생성
mkdir my-pulumi-project && cd my-pulumi-project
pulumi new typescript  # 또는 python, go, csharp 등
pulumi up

# 2. Terraform state 변환
pulumi plugin run terraform-migrate -- stack \
    --from path/to/terraform-sources \
    --to path/to/pulumi-project \
    --out /tmp/pulumi-state.json \
    --plugins /tmp/required-plugins.json

# 3. 필수 플러그인 설치 및 state 임포트
pulumi plugin install resource aws 7.12.0
pulumi stack import --file /tmp/pulumi-state.json

# 4. LLM 에이전트로 코드 변환 후 검증
pulumi preview
pulumi up
```

### Terraform State 참조 (공존)

TypeScript 예제:

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as terraform from "@pulumi/terraform";
import * as eks from "@pulumi/eks";

const tfState = terraform.state.getLocalReferenceOutput({
    path: "../tf-state-ref-terraform/terraform.tfstate",
});

const vpcId = tfState.outputs["vpc_id"] as pulumi.Output<string>;
const publicSubnetIds = tfState.outputs["public_subnet_ids"] as pulumi.Output<string[]>;
const privateSubnetIds = tfState.outputs["private_subnet_ids"] as pulumi.Output<string[]>;

const cluster = new eks.Cluster("my-cluster", {
    vpcId: vpcId,
    publicSubnetIds: publicSubnetIds,
    privateSubnetIds: privateSubnetIds,
});
```

Python 예제:

```python
import pulumi
import pulumi_terraform as terraform
import pulumi_eks as eks

tf_state = terraform.state.get_local_reference_output(
    path="../tf-state-ref-terraform/terraform.tfstate"
)

vpc_id = tf_state.outputs["vpc_id"]
public_subnet_ids = tf_state.outputs["public_subnet_ids"]
private_subnet_ids = tf_state.outputs["private_subnet_ids"]

cluster = eks.Cluster("my-cluster",
    vpc_id=vpc_id,
    public_subnet_ids=public_subnet_ids,
    private_subnet_ids=private_subnet_ids
)
```

| 함수 | 용도 |
|------|------|
| `terraform.state.getLocalReference` | 로컬 `.tfstate` 파일 참조 |
| `terraform.state.getRemoteReference` | Terraform Cloud / Enterprise의 원격 state 참조 |

### HCL 코드 변환 (`pulumi convert`)

```bash
# HCL을 Pulumi TypeScript 코드로 변환
pulumi convert --from terraform --language typescript

# HCL을 Pulumi Python 코드로 변환
pulumi convert --from terraform --language python
```

**지원되는 Terraform 기능**: Variables, outputs, resources, data sources, Terraform modules (Pulumi Component로 변환), 대부분의 HCL2 표현식

변환기가 지원하지 않는 기능은 `notImplemented` 호출로 TODO가 생성되며, 대부분의 프로젝트에서 90~95%의 코드가 자동 변환된다.

### 기존 리소스 일괄 임포트

```bash
# .tfstate 파일의 모든 리소스를 Pulumi로 임포트
pulumi import --from terraform ./terraform.tfstate
```

임포트된 리소스는 안전을 위해 `protect` 속성이 설정된다.

### Terraform Module 직접 사용

```bash
# AWS VPC 모듈 추가
pulumi package add terraform-module terraform-aws-modules/vpc/aws 5.19.0 vpc
```

TypeScript 예제:

```typescript
import * as vpc from "@pulumi/vpc";

const myVpc = new vpc.Module("my-vpc", {
    name: "pulumi-vpc",
    cidr: "10.0.0.0/16",
    azs: ["us-west-2a", "us-west-2b", "us-west-2c"],
    private_subnets: ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"],
    public_subnets: ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"],
    enable_nat_gateway: true
});

export const vpcId = myVpc.vpc_id;
```

Python 예제:

```python
import pulumi
import pulumi_vpc as vpc

my_vpc = vpc.Module("my-vpc",
    name="pulumi-vpc",
    cidr="10.0.0.0/16",
    azs=["us-west-2a", "us-west-2b", "us-west-2c"],
    private_subnets=["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"],
    public_subnets=["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"],
    enable_nat_gateway=True
)

pulumi.export("vpc_id", my_vpc.vpc_id)
```

---

## AWS CloudFormation에서 Pulumi로 마이그레이션

> 출처: https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-cloudformation/

### 마이그레이션 경로 선택

| 경로 | 방식 | 추천 대상 |
|------|------|-----------|
| **Pulumi Neo (권장)** | CloudFormation 템플릿 자동 변환 + 리소스 임포트 | 대부분의 경우 |
| **공존** | `aws.cloudformation.getStack`으로 스택 출력값 참조 | 점진적 전환 |
| **CloudFormation Stack 리소스 사용** | `aws.cloudformation.Stack`으로 기존 템플릿을 Pulumi에서 배포 | 중간 단계 |
| **임포트 + 코드 재작성** | 리소스를 개별적으로 임포트하고 코드로 재작성 | 전면 전환 |

### CloudFormation 스택 출력값 참조 (공존)

TypeScript 예제:

```typescript
import * as aws from "@pulumi/aws";

const network = aws.cloudformation.getStackOutput({
    name: "my-network-stack",
});

const subnetId = network.outputs["SubnetId"];

const web = new aws.ec2.Instance("web", {
    ami: "ami-0adc0e3ef2558cb1f",
    instanceType: "t3.micro",
    subnetId: subnetId,
});
```

Python 예제:

```python
import pulumi_aws as aws

network = aws.cloudformation.get_stack(
    name='my-network-stack'
)

subnet_id = network.outputs['SubnetId']

web = aws.ec2.Instance('web',
    ami='ami-0adc0e3ef2558cb1f',
    instance_type='t2.micro',
    subnet_id=subnet_id
)
```

### CloudFormation Stack을 Pulumi로 배포

Pulumi의 `aws.cloudformation.Stack` 리소스로 기존 CloudFormation 템플릿을 Pulumi에서 배포할 수 있다. 이 경우 Pulumi는 CloudFormation 스택을 단일 불투명 리소스로 취급하며, 내부 개별 리소스는 제어하지 않는다.

TypeScript 예제:

```typescript
import * as aws from "@pulumi/aws";

const template = `{
    "Parameters" : {
        "VPCCidr" : {
            "Type" : "String",
            "Default" : "10.0.0.0/16"
        }
    },
    "Resources": {
        "myVpc": {
            "Type" : "AWS::EC2::VPC",
            "Properties" : {
                "CidrBlock" : { "Ref" : "VPCCidr" },
                "Tags" : [{"Key": "Name", "Value": "Primary_CF_VPC"}]
            }
        }
    },
    "Outputs": {
        "VpcId": { "Value": { "Ref": "myVpc" } }
    }
}`;

const network = new aws.cloudformation.Stack("network", {
    templateBody: template,
    parameters: {
        VPCCidr: "10.0.0.0/16",
    },
});

export const vpcId = network.outputs["VpcId"];
```

### CloudFormation 리소스를 Pulumi 코드로 마이그레이션

전면 마이그레이션 절차:

1. CloudFormation 템플릿에서 대상 리소스에 `DeletionPolicy: Retain` 설정 후 `pulumi up`
2. Pulumi 코드에서 `import` 옵션으로 리소스를 편입
3. `pulumi up` 실행 후 `import` 지시어 제거
4. CloudFormation 스택에서 리소스 제거

TypeScript 임포트 예제:

```typescript
import * as aws from "@pulumi/aws";

const vpc = new aws.ec2.Vpc("myVpc", {
    cidrBlock: "10.0.0.0/16",
    tags: { Name: "Primary_CF_VPC" },
}, { import: "vpc-0e1a74859af1da17f" });

export const vpcId = vpc.id;
```

Python 임포트 예제:

```python
import pulumi
import pulumi_aws as aws

vpc = aws.ec2.Vpc('myVpc',
    cidr_block='10.0.0.0/16',
    tags={ 'Name': 'Primary_CF_VPC' },
    opts=pulumi.ResourceOptions(import_='vpc-0e1a74859af1da17f')
)

pulumi.export('vpc_id', vpc.id)
```

> **주의**: Pulumi의 AWS 리소스 모델이 CloudFormation의 리소스 프로젝션과 정확히 일치하지 않으므로, 현재 이 변환을 자동화하는 도구는 없다. CloudFormation 템플릿 정의를 복사한 후 수동으로 변환하는 방식이 권장된다.

---

## Azure ARM Templates / Bicep에서 Pulumi로 마이그레이션

> 출처: https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-arm/

### 마이그레이션 경로 선택

| 경로 | 방식 | 추천 대상 |
|------|------|-----------|
| **Pulumi Neo (권장)** | ARM/Bicep 템플릿 자동 변환 + 리소스 임포트 | 대부분의 경우 |
| **공존** | `resources.Deployment.get()`으로 ARM 배포 출력값 참조 | 점진적 전환 |
| **변환** | `pulumi convert --from arm` 또는 `--from bicep` | 코드 변환 |
| **임포트** | `pulumi import` + `import` 리소스 옵션 | 리소스 편입 |

### ARM vs Pulumi 비교

| 항목 | ARM Templates | Pulumi |
|------|---------------|--------|
| 언어 | JSON (Bicep은 DSL) | C#, Python, TypeScript, Go, Java, YAML |
| 클라우드 | Azure 전용 | 모든 클라우드 + SaaS |
| 재사용 | 제한적 (복사/붙여넣기) | 함수, 클래스, 모듈 |
| 논리/반복 | 복잡한 표현식 | `if` / `for` / `switch` |
| 타입 안전성 | 없음 | 컴파일 타임 타입 검사 |
| 테스팅 | 수동 또는 없음 | xUnit, NUnit, pytest 등 |
| 상태 파일 | 없음 (Azure가 관리) | 암호화된 상태 파일 |

### ARM 배포 출력값 참조 (공존)

TypeScript 예제:

```typescript
import * as resources from "@pulumi/azure-native/resources";
import * as storage from "@pulumi/azure-native/storage";
import * as pulumi from "@pulumi/pulumi";

const deployment = resources.Deployment.get("myStorageDeployment",
    "/subscriptions/<YOUR-SUBSCRIPTION-ID>/resourceGroups/myrg/providers/Microsoft.Resources/deployments/myStorageDeployment");
const storageAccountName = deployment.properties.outputs["storageAccountName"].value;

const blob = new storage.Blob("zip", {
    resourceGroupName: "myrg",
    accountName: storageAccountName,
    containerName: new storage.BlobContainer("myStorageContainer", {
        resourceGroupName: "myrg",
        accountName: storageAccountName,
        containerName: "files",
    }).name,
    source: new pulumi.asset.FileArchive("wwwroot"),
});

export const blobUrl = blob.url;
```

ARM 배포 ID 형식: `/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RG-NAME>/providers/Microsoft.Resources/deployments/<DEPLOYMENT-NAME>`

### ARM 템플릿 변환 (`pulumi convert`)

```bash
# ARM JSON 템플릿을 Pulumi 코드로 변환
pulumi convert --from arm --language typescript

# Bicep 파일을 Pulumi 코드로 변환
pulumi convert --from bicep --language python
```

변환된 코드는 Azure Native provider를 사용하며, 이 provider는 Azure Resource Manager REST API 사양에서 직접 생성되어 Azure 리소스에 대한 동일한 날(same-day) 지원 범위를 제공한다.

### Azure 리소스 임포트

임포트 시 Azure 리소스의 **정규화된 리소스 ID**를 사용해야 한다.

TypeScript 예제:

```typescript
import * as azure_native from "@pulumi/azure-native";

const storagecreatedbyarm = new azure_native.storage.StorageAccount("storagecreatedbyarm", {
    accountName: "storagecreatedbyarm",
    kind: "StorageV2",
    location: "westeurope",
    resourceGroupName: "existing-rg",
    sku: {
        name: "Standard_LRS",
    },
}, { import: "/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/existing-rg/providers/Microsoft.Storage/storageAccounts/storagecreatedbyarm" });
```

> **주의**: 임포트 시 Azure가 자동으로 설정한 속성(`accessTier`, `enableHttpsTrafficOnly`, `encryption`, `networkRuleSet` 등)으로 인해 경고가 발생할 수 있다. `ignoreChanges` 옵션으로 해당 속성을 제외하면 된다.

---

## AWS CDK에서 Pulumi로 마이그레이션

> 출처: https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-cdk/

### 마이그레이션 경로 선택

| 경로 | 방식 | 추천 대상 |
|------|------|-----------|
| **Pulumi Neo (권장)** | CDK 코드 자동 변환 + CloudFormation 리소스 임포트 | 대부분의 경우 |
| **Pulumi CDK Adapter** | 기존 CDK Construct를 Pulumi 프로그램 내에서 직접 사용 | 최소 코드 변경 |
| **수동 마이그레이션** | `pulumi import`로 리소스를 개별 편입 후 코드 재작성 | 전면 전환 |

### Pulumi CDK Adapter를 사용한 공존

Pulumi CDK Adapter(`pulumi-cdk`)를 사용하면 기존 CDK Construct를 Pulumi 프로그램 내부에 직접 포함할 수 있다. CDK Construct의 출력값은 다른 Pulumi 리소스의 입력으로 사용할 수 있고, 그 반대도 가능하다.

### CDK에서 Pulumi로의 전면 전환

1. CDK Construct를 점진적으로 Pulumi 네이티브 코드로 교체
2. AWS CDK의 CloudFormation 기반 실행 모델을 Pulumi의 직접 실행 모델로 전환
3. CDK의 L1/L2/L3 Construct 계층을 Pulumi Component로 재구성

---

## CDKTF에서 Pulumi로 마이그레이션

> 출처: https://www.pulumi.com/docs/iac/comparisons/cdktf/
>
> **참고**: CDKTF는 2025년 12월에 deprecated 되었으며, GitHub 저장소가 아카이브되었다.

### 마이그레이션 경로

| 경로 | 명령/도구 | 설명 |
|------|-----------|------|
| **Pulumi Neo (권장)** | Neo | 코드 변환 + 상태 이관 자동화 |
| **CDKTF 합성 후 변환** | `cdktf synth --hcl` 후 `pulumi convert --from terraform` | HCL로 합성 후 Pulumi로 변환 |
| **상태 이관** | `pulumi-terraform-migrate` | CDKTF의 Terraform state를 Pulumi state로 변환 |
| **임포트** | `pulumi import` | 리소스 단위 편입 |
| **Terraform Module 직접 사용** | `pulumi package add terraform-module` | 기존 모듈 자산 보존 |

CDKTF 마이그레이션 절차:

```bash
# 1. CDKTF를 HCL로 합성
cdktf synth --hcl

# 2. HCL을 Pulumi 코드로 변환
pulumi convert --from terraform --language typescript

# 3. 상태 이관
pulumi-terraform-migrate  # 별도 도구
```

---

## Kubernetes YAML / Helm에서 Pulumi로 마이그레이션

> 출처: https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-kubernetes/

### 마이그레이션 경로 선택

| 경로 | 방식 | 설명 |
|------|------|------|
| **기존 YAML 그대로 배포** | `ConfigFile`, `ConfigGroup` | YAML 재작성 없이 Pulumi에서 오케스트레이션 |
| **YAML로 렌더링** | Pulumi에서 YAML 출력 | Pulumi로 작성하되 기존 `kubectl`/GitOps 파이프라인으로 배포 |
| **변환** | `pulumi convert --from kubernetes` | YAML을 Pulumi 프로그램 코드로 변환 |
| **임포트** | `pulumi import` | 실행 중인 클러스터 리소스를 Pulumi 관리로 편입 |

### 기존 Kubernetes YAML 그대로 배포

Kubernetes provider의 `yaml` 모듈은 두 가지 리소스 타입을 제공한다:

| 리소스 타입 | 용도 |
|-------------|------|
| `ConfigFile` | 단일 Kubernetes YAML 파일 배포 |
| `ConfigGroup` | 여러 Kubernetes YAML 파일을 묶어서 배포 |

### YAML 렌더링 (역방향 호환)

Pulumi 프로그램을 Kubernetes YAML로 렌더링할 수도 있다. 이를 통해 범용 프로그래밍 언어로 설정을 작성하면서도 기존 `kubectl`이나 GitOps 파이프라인으로 배포할 수 있다.

### Kubernetes YAML 변환

```bash
pulumi convert --from kubernetes --language typescript
```

---

## Serverless Framework에서 Pulumi로 마이그레이션

> 출처: https://www.pulumi.com/docs/iac/guides/migration/migrating-to-pulumi/from-serverless/

### Serverless Framework와 CloudFormation의 관계

Serverless Framework는 `sls deploy` 실행 시 CloudFormation 템플릿을 생성하여 `{service}-{stage}` 이름 패턴의 CloudFormation 스택으로 배포한다. 따라서 CloudFormation 마이그레이션 전략이 그대로 적용된다.

### 마이그레이션 경로

| 경로 | 방식 | 추천 대상 |
|------|------|-----------|
| **Pulumi Neo (권장)** | CloudFormation 스택 자동 변환 | 대부분의 경우 |
| **공존** | `aws.cloudformation.getStack`으로 스택 출력값 참조 | 점진적 전환 |
| **임포트** | `pulumi import` | 기존 리소스 편입 |
| **재작성** | `serverless.yml`을 Pulumi 코드로 직접 변환 | 전면 전환 |

### Serverless Framework 리소스 매핑 표

| serverless.yml 항목 | Pulumi AWS 리소스 |
|---------------------|-------------------|
| `functions.[name]` | `aws.lambda.Function` |
| `functions.[name].events[].httpApi` | `aws.apigatewayv2.Api` |
| `functions.[name].events[].http` | `aws.apigateway.RestApi` + 관련 리소스 |
| `functions.[name].events[].sqs` | `aws.sqs.Queue` + `aws.lambda.EventSourceMapping` |
| `functions.[name].events[].sns` | `aws.sns.Topic` + `aws.sns.TopicSubscription` |
| `functions.[name].events[].s3` | `aws.s3.BucketNotification` |
| `functions.[name].events[].schedule` | `aws.cloudwatch.EventRule` + `aws.cloudwatch.EventTarget` |
| `functions.[name].events[].eventBridge` | `aws.cloudwatch.EventRule` + `aws.cloudwatch.EventTarget` |
| `provider.iam.role.statements` | `aws.iam.Role` + `aws.iam.RolePolicy` |
| `provider.environment` | `aws.lambda.Function`의 `environment` 인자 |
| `resources.Resources` (DynamoDB) | `aws.dynamodb.Table` |
| `resources.Resources` (S3) | `aws.s3.BucketV2` |
| `resources.Resources` (SES) | `aws.ses.DomainIdentity`, `aws.ses.EmailIdentity` |
| Stages (`--stage dev`) | Pulumi stacks (`pulumi stack select dev`) |

### Serverless Framework 스택 출력값 참조 (공존)

TypeScript 예제:

```typescript
import * as aws from "@pulumi/aws";

const serverlessStack = aws.cloudformation.getStackOutput({
    name: "my-api-dev",
});

const apiEndpoint = serverlessStack.outputs["ServiceEndpoint"];
const processOrderArn = serverlessStack.outputs["ProcessOrderLambdaFunctionQualifiedArn"];

const queue = new aws.sqs.Queue("new-queue");

export const endpoint = apiEndpoint;
export const orderFunctionArn = processOrderArn;
```

Python 예제:

```python
import pulumi
import pulumi_aws as aws

serverless_stack = aws.cloudformation.get_stack(
    name="my-api-dev"
)

api_endpoint = serverless_stack.outputs["ServiceEndpoint"]
process_order_arn = serverless_stack.outputs["ProcessOrderLambdaFunctionQualifiedArn"]

queue = aws.sqs.Queue("new-queue")

pulumi.export("endpoint", api_endpoint)
pulumi.export("order_function_arn", process_order_arn)
```

### Serverless Framework 리소스 임포트 절차

1. **리소스 식별**:

```bash
aws cloudformation list-stack-resources --stack-name my-api-dev \
    --query "StackResourceSummaries[].{Type:ResourceType,LogicalId:LogicalResourceId,PhysicalId:PhysicalResourceId}" \
    --output table
```

2. **Pulumi로 임포트**:

```bash
# Lambda 함수 임포트
pulumi import aws:lambda/function:Function create-order my-api-dev-createOrder

# DynamoDB 테이블 임포트
pulumi import aws:dynamodb/table:Table orders-table my-api-orders-dev

# API Gateway v2 임포트
pulumi import aws:apigatewayv2/api:Api api abc123def
```

3. **CloudFormation에서 리소스 분리**: `DeletionPolicy: Retain` 설정 후 스택 삭제

### Stage 관리

Serverless Framework의 `--stage`는 Pulumi의 stack으로 직접 매핑된다:

```bash
pulumi stack init dev
pulumi stack init staging
pulumi stack init prod

pulumi config set --stack dev aws:region us-east-1
pulumi config set --stack prod aws:region us-west-2
```

---

## 소스별 마이그레이션 단계 요약 및 주의사항

### Terraform

| 단계 | 명령/작업 | 주의사항 |
|------|-----------|----------|
| 1. 경로 선택 | Neo / state-first / 공존 / 변환 | Neo 우선 고려 |
| 2. 상태 이관 | `pulumi-terraform-migrate` 또는 `pulumi import --from terraform` | 스택별로 반복 수행 |
| 3. 코드 변환 | `pulumi convert --from terraform` | 미지원 기능은 TODO로 표시됨 (90~95% 자동 변환) |
| 4. 검증 | `pulumi preview` | 변경 사항 없음 확인 |
| 5. 적용 | `pulumi up` | 이후 기존 Terraform 관리 중단 |

### AWS CloudFormation

| 단계 | 명령/작업 | 주의사항 |
|------|-----------|----------|
| 1. 경로 선택 | Neo / 공존 / Stack 배포 / 임포트 | 대규모 스택은 점진적 전환 권장 |
| 2. Retain 정책 설정 | 템플릿에 `DeletionPolicy: Retain` 추가 | 삭제 방지 필수 |
| 3. Pulumi 코드 작성 | CloudFormation 템플릿을 코드로 재작성 | 자동 변환 도구 없음, 수동 변환 필요 |
| 4. 임포트 | `{ import: "<RESOURCE_ID>" }` | 리소스 ID는 AWS 콘솔/CLI에서 확인 |
| 5. 검증 및 적용 | `pulumi preview` 후 `pulumi up` | `ignoreChanges`로 속성 차이 해결 |

### Azure ARM / Bicep

| 단계 | 명령/작업 | 주의사항 |
|------|-----------|----------|
| 1. 경로 선택 | Neo / 공존 / 변환 / 임포트 | Bicep도 `pulumi convert --from bicep` 지원 |
| 2. 변환 | `pulumi convert --from arm` 또는 `--from bicep` | Azure Native provider 사용 |
| 3. 임포트 | `{ import: "<FULL_RESOURCE_ID>" }` | 정규화된 Azure 리소스 ID 필요 |
| 4. 검증 | `pulumi preview` | Azure 자동 설정 속성으로 인한 경고에 `ignoreChanges` 사용 |

### AWS CDK

| 단계 | 명령/작업 | 주의사항 |
|------|-----------|----------|
| 1. 경로 선택 | Neo / CDK Adapter / 수동 | Neo 권장, Adapter는 공존용 |
| 2. 코드 변환 | Neo 자동 변환 또는 수동 재작성 | CDK Construct를 Pulumi Component로 재구성 |
| 3. 리소스 임포트 | CloudFormation 기반 임포트 | CloudFormation 마이그레이션 전략과 동일 |

### CDKTF (deprecated)

| 단계 | 명령/작업 | 주의사항 |
|------|-----------|----------|
| 1. HCL 합성 | `cdktf synth --hcl` | 스택별 HCL 파일 생성 |
| 2. 변환 | `pulumi convert --from terraform` | Terraform 변환 경로와 동일 |
| 3. 상태 이관 | `pulumi-terraform-migrate` | Terraform state와 동일 방식 |

### Kubernetes YAML / Helm

| 단계 | 명령/작업 | 주의사항 |
|------|-----------|----------|
| 1. 경로 선택 | ConfigFile/ConfigGroup / 렌더링 / 변환 / 임포트 | 기존 YAML 재사용이 가장 간단 |
| 2. YAML 배포 | `ConfigFile`, `ConfigGroup` 리소스 사용 | YAML 변경 없이 Pulumi로 오케스트레이션 |
| 3. 변환 | `pulumi convert --from kubernetes` | 필요시 Pulumi 코드로 전환 |
| 4. YAML 렌더링 | Pulumi를 YAML로 출력 | 기존 `kubectl`/GitOps 파이프라인 유지 가능 |

### Serverless Framework

| 단계 | 명령/작업 | 주의사항 |
|------|-----------|----------|
| 1. CloudFormation 스택 식별 | `sls info --stage dev` 또는 AWS CLI | `{service}-{stage}` 이름 패턴 |
| 2. 공존 또는 임포트 | `aws.cloudformation.getStack` 또는 `pulumi import` | CloudFormation 마이그레이션 전략 적용 |
| 3. 코드 재작성 | 리소스 매핑 표 참조 | Stage는 Pulumi stack으로 매핑 |
| 4. Retain 정책 설정 | CloudFormation 템플릿에 `DeletionPolicy: Retain` | 자동 생성 리소스는 `jq`로 템플릿 수정 필요 |

---

## 모든 소스에 공통으로 적용되는 임포트 가이드

> 출처: https://www.pulumi.com/docs/iac/guides/migration/

### `pulumi import` CLI

```bash
# 단일 리소스 임포트
pulumi import <TYPE> <NAME> <ID>

# 예: AWS EC2 인스턴스
pulumi import aws:ec2/instance:Instance my-server i-0adc0e3ef2558cb1f
```

### `import` 리소스 옵션 (코드 내)

TypeScript:

```typescript
const resource = new aws.ec2.Instance("my-server", {
    // ... 리소스 속성
}, { import: "i-0adc0e3ef2558cb1f" });
```

Python:

```python
resource = aws.ec2.Instance("my-server",
    # ... 리소스 속성
    opts=pulumi.ResourceOptions(import_='i-0adc0e3ef2558cb1f')
)
```

### 임포트 후 필수 작업

1. `pulumi preview`로 변경 사항 없음을 확인
2. `pulumi up`으로 임포트 완료
3. 코드에서 `import` 옵션 제거
4. 이후 Pulumi로 리소스 관리
