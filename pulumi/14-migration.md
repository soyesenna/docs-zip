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
| AWS CDK | O | O | O* |
| CDKTF (deprecated) | O | O | O |
| 기타 (수동 프로비저닝 등) | O | O | - |

> **CDK 변환(O*) 각주**: AWS CDK는 `pulumi convert --from cdk` 같은 독립적 변환 도구를 제공하지 않는다. CDK 마이그레이션은 Neo(자동 변환), CDK Adapter(공존), 수동 import+재작성 세 가지 경로만 지원된다. 매트릭스의 O*는 Neo를 통한 자동 변환만 해당됨을 의미한다. Terraform(`--from terraform`), ARM(`--from arm`), Kubernetes(`--from kubernetes`)는 실제 `pulumi convert` 대상이 존재한다.

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

> 출처: https://www.pulumi.com/docs/iac/comparisons/terraform/

### Pulumi Cloud를 Terraform State Backend로 사용 (Store Terraform State)

Terraform/OpenTofu를 계속 사용하면서도 Pulumi Cloud의 암호화 상태 관리, 업데이트 히스토리, 상태 잠금, RBAC, 감사 정책을 활용할 수 있다. 이 경로는 기존 Terraform 워크플로를 유지하면서 점진적으로 Pulumi 생태계로 전환하고자 하는 팀에 적합하다.

```bash
# Pulumi Cloud를 Terraform state backend로 구성
terraform {
  backend "pulumi" {
    # Pulumi Cloud 조직 및 프로젝트 설정
  }
}
```

자세한 내용은 [Pulumi Cloud as a Terraform State Backend](https://www.pulumi.com/docs/iac/get-started/terraform/terraform-state-backend/)를 참조하라.

### Terraform Provider를 Pulumi에서 직접 사용 (Any Terraform Provider)

Pulumi는 [Terraform Bridge](https://www.pulumi.com/docs/iac/concepts/providers/any-terraform-provider/)를 통해 Terraform Registry에 게시된 모든 Provider를 Pulumi Provider로 변환하여 사용할 수 있다. 이를 통해 Terraform 전용 생태계의 리소스를 Pulumi 프로그램에서 관리할 수 있다. Pulumi Registry의 많은 Provider가 이 방식으로 구축되었다.

```bash
# Terraform Provider를 Pulumi에서 사용
pulumi package add terraform-provider <provider-name>
```

이 기능은 기존 Terraform Provider 자산을 Pulumi로 즉시 활용할 수 있게 해주므로, 마이그레이션 과정에서 Provider 호환성 문제를 해결하는 핵심 경로이다.

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

Pulumi는 Kubernetes 설정을 원하는 언어로 작성할 수 있을 뿐 아니라, 기존 Kubernetes 및 Helm YAML 설정 파일을 그대로 재사용할 수도 있다. 이를 통해 기존 YAML을 재작성하거나, Pulumi 코드로 새로 작성하거나, 하이브리드 접근을 취할 수 있으며, 모든 경우에 Pulumi를 배포 오케스트레이션으로 표준화할 수 있다.

### 마이그레이션 경로 선택

| 경로 | 방식 | 설명 |
|------|------|------|
| **기존 YAML 그대로 배포** | `ConfigFile`, `ConfigGroup` (v2 API) | YAML 재작성 없이 Pulumi에서 오케스트레이션 |
| **Helm Chart 배포** | `Chart` 리소스 (클라이언트 사이드 렌더) / `Release` 리소스 (Helm SDK) | Helm Chart를 Pulumi에서 직접 관리 |
| **YAML로 렌더링** | `renderYamlToDirectory` | Pulumi로 작성하되 기존 `kubectl`/GitOps 파이프라인으로 배포 |
| **변환** | `pulumi convert --from kubernetes` | YAML을 Pulumi 프로그램 코드로 변환 |
| **임포트** | `pulumi import` | 실행 중인 클러스터 리소스를 Pulumi 관리로 편입 |

### 기존 Kubernetes YAML 그대로 배포 (ConfigFile / ConfigGroup)

Kubernetes provider의 `yaml.v2` 모듈은 두 가지 리소스 타입을 제공한다:

| 리소스 타입 | 용도 |
|-------------|------|
| `ConfigFile` | 단일 Kubernetes YAML 파일 배포 |
| `ConfigGroup` | 여러 Kubernetes YAML 파일을 묶어서 배포 |

기본적으로 리소스 이름은 그대로 사용되며, `resourcePrefix`로 이름을 재지정할 수 있다. `transforms` 콜백을 `ResourceOptions`로 전달하여 배포 전에 리소스 설정을 즉석에서 수정할 수도 있다.

#### 단일 YAML 파일 배포 (ConfigFile)

```bash
# Kubernetes Guestbook 예제 YAML 다운로드
curl -L --remote-name \
    https://raw.githubusercontent.com/kubernetes/examples/master/web/guestbook/all-in-one/guestbook-all-in-one.yaml
```

TypeScript 예제:

```typescript
import * as k8s from "@pulumi/kubernetes";

// 단일 YAML 파일에서 리소스 생성
const guestbook = new k8s.yaml.v2.ConfigFile("guestbook", {
    file: "guestbook-all-in-one.yaml",
});

// getResource로 내부 리소스에 접근
const frontend = guestbook.getResource("v1/Service", "frontend");
export const privateIp = frontend.spec.clusterIP;
```

Python 예제:

```python
from pulumi_kubernetes.yaml.v2 import ConfigFile

guestbook = ConfigFile("guestbook", file="guestbook-all-in-one.yaml")

# getResource로 내부 리소스에 접근
frontend = guestbook.get_resource("v1/Service", "frontend")
pulumi.export("private_ip", frontend.spec["cluster_ip"])
```

`getResource` 함수를 사용하면 YAML 내부의 리소스를 타입과 이름으로 조회하여 속성에 접근할 수 있다. 반환값은 리소스 타입에 따라 강타입으로 제공된다.

#### 여러 YAML 파일 배포 (ConfigGroup)

```bash
# 여러 YAML 파일 다운로드
mkdir yaml
curl -L --remote-name \
    "https://raw.githubusercontent.com/kubernetes/examples/master/web/guestbook/{frontend-deployment,frontend-service,redis-master-deployment,redis-master-service,redis-replica-deployment,redis-replica-service}.yaml"
```

TypeScript 예제:

```typescript
import * as k8s from "@pulumi/kubernetes";
import * as path from "path";

const guestbook = new k8s.yaml.v2.ConfigGroup("guestbook", {
    files: [path.join("yaml", "*.yaml")],
});

const frontend = guestbook.getResource("v1/Service", "frontend");
export const privateIp = frontend.spec.clusterIP;
```

Python 예제:

```python
from pulumi_kubernetes.yaml.v2 import ConfigGroup

guestbook = ConfigGroup("guestbook", files=["yaml/*.yaml"])
frontend = guestbook.get_resource("v1/Service", "frontend")
pulumi.export("private_ip", frontend.spec["cluster_ip"])
```

### Helm Chart 배포

Pulumi는 Helm Chart를 사용하는 두 가지 방식을 지원한다:

| 방식 | 리소스 타입 | 특징 |
|------|-------------|------|
| **Chart 리소스 (클라이언트 사이드)** | `k8s.helm.v3.Chart` | 템플릿을 렌더링하여 직접 적용. `getResource`로 내부 리소스 접근 가능 |
| **Release 리소스 (Helm SDK)** | `k8s.helm.v3.Release` | Helm SDK를 내장하여 네이티브 Helm Release 관리. `pulumi import`로 기존 Release 임포트 가능 |

#### Chart 리소스로 Helm Chart 배포

Chart 리소스는 `ConfigFile`/`ConfigGroup`과 유사하게 템플릿을 클라이언트 사이드에서 렌더링하여 직접 적용한다. 서버 사이드 컴포넌트 없이 Kubernetes 인증 설정만으로 프로비저닝이 이루어진다.

주요 옵션:
- `chart`: Chart 이름 (필수, 예: `"wordpress"`)
- `repo`: Helm 저장소 URL (예: `"https://charts.bitnami.com/bitnami"`)
- `version`: Chart 버전 (기본: 최신)
- `values`: Chart 값 설정 (키/값 딕셔너리)
- `fetchOpts`: fetch 동작 제어 옵션
- `resourcePrefix`: 리소스 이름 접두사
- `namespace`: 모든 리소스를 특정 네임스페이스에 배치
- `transformations`: 리소스 변환 콜백

TypeScript 예제 (WordPress Chart 배포):

```typescript
import * as k8s from "@pulumi/kubernetes";

const wordpress = new k8s.helm.v3.Chart("wpdev", {
    fetchOpts: {
        repo: "https://charts.bitnami.com/bitnami"
    },
    chart: "wordpress",
});

// getResource로 Service 접근
const frontend = wordpress.getResource("v1/Service", "default/wpdev-wordpress");
export const frontendIp = frontend.status.loadBalancer.ingress[0].ip;
```

Python 예제:

```python
from pulumi_kubernetes.helm.v3 import Chart, ChartOpts

wordpress = Chart('wpdev', ChartOpts(
    fetch_opts={'repo': 'https://charts.bitnami.com/bitnami'},
    chart='wordpress',
))

frontend = wordpress.get_resource('v1/Service', 'default/wpdev-wordpress')
pulumi.export('frontend_ip', frontend.status.load_balancer.ingress[0].ip)
```

#### Release 리소스로 Helm Chart 배포

Release 리소스는 Pulumi Kubernetes Provider에 내장된 Helm SDK를 사용하여 완전한 Helm Release 관리를 제공한다. `Chart` 리소스와 달리 내부 Kubernetes 리소스에 대한 참조를 포함하지 않으며, Release 상태만 Pulumi state에 저장된다.

주요 옵션:
- `chart`: Chart 이름 (필수)
- `repositoryOpts`: 저장소 URL 및 인증 정보
- `version`: Chart 버전
- `values`: Chart 값 설정
- `skipAwait`: 리소스 가용성 대기 건너뛰기 (기본: false)
- `timeout`: 대기 시간 초과

TypeScript 예제:

```typescript
import * as k8s from "@pulumi/kubernetes";
import * as pulumi from "@pulumi/pulumi";

const wordpress = new k8s.helm.v3.Release("wpdev", {
    chart: "wordpress",
    repositoryOpts: {
        repo: "https://charts.bitnami.com/bitnami",
    },
});

const svc = k8s.core.v1.Service.get("wpdev-wordpress",
    pulumi.interpolate`${wordpress.status.namespace}/${wordpress.status.name}-wordpress`);
export const frontendIp = svc.status.loadBalancer.ingress[0].ip;
```

Python 예제:

```python
from pulumi import Output
from pulumi_kubernetes.core.v1 import Service
from pulumi_kubernetes.helm.v3 import Release, ReleaseArgs, RepositoryOptsArgs

wordpress = Release("wpdev", ReleaseArgs(
    chart="wordpress",
    repository_opts=RepositoryOptsArgs(
        repo="https://charts.bitnami.com/bitnami",
    ),
))

srv = Service.get("wpdev-wordpress",
    Output.concat(wordpress.status.namespace, "/", wordpress.status.name, "-wordpress"))
pulumi.export("frontendIP", srv.status.load_balancer.ingress[0].ip)
```

Release 리소스는 `pulumi import` 명령으로 기존 Helm Release를 임포트하는 것도 지원한다.

### Configuration Transformations (transforms)

`transforms` 콜백을 사용하면 YAML 배포 시 리소스 설정을 즉석에서 수정할 수 있다. 예를 들어, 기본적으로 LoadBalancer가 없는 서비스에 `type: LoadBalancer`를 추가할 수 있다.

TypeScript 예제:

```typescript
import * as k8s from "@pulumi/kubernetes";

const guestbook = new k8s.yaml.v2.ConfigFile("guestbook", {
    file: "guestbook-all-in-one.yaml",
}, {
    transforms: [async (args) => {
        if (args.type === "kubernetes:core/v1:Service" &&
                (args.props as any)?.metadata?.name === "frontend") {
            const props = args.props as any;
            props.spec = { ...props.spec, type: "LoadBalancer" };
            return { props, opts: args.opts };
        }
    }],
});

const frontend = guestbook.getResource("v1/Service", "frontend");
export const publicIp = frontend.status.loadBalancer.ingress[0].ip;
```

Python 예제:

```python
import pulumi
from pulumi_kubernetes.yaml.v2 import ConfigFile

def make_frontend_public(args):
    if (args.type_ == "kubernetes:core/v1:Service" and
            args.props.get("metadata", {}).get("name") == "frontend"):
        props = dict(args.props)
        spec = dict(props.get("spec", {}))
        spec["type"] = "LoadBalancer"
        props["spec"] = spec
        return pulumi.ResourceTransformResult(props=props, opts=args.opts)

guestbook = ConfigFile("guestbook",
    file="guestbook-all-in-one.yaml",
    opts=pulumi.ResourceOptions(transforms=[make_frontend_public]))

frontend = guestbook.get_resource("v1/Service", "frontend")
pulumi.export("public_ip", frontend.status["load_balancer"]["ingress"][0]["ip"])
```

이 예제는 `ConfigFile`을 사용하지만, `ConfigGroup`과 Helm `Chart` 리소스 타입에서도 동일한 transform 동작이 사용 가능하다.

### YAML 렌더링 (renderYamlToDirectory)

Pulumi 프로그램을 Kubernetes YAML로 렌더링할 수 있다. 이를 통해 범용 프로그래밍 언어로 설정을 작성하면서도 기존 `kubectl`이나 GitOps 파이프라인으로 배포할 수 있다.

명시적 Kubernetes Provider 객체의 `renderYamlToDirectory` 속성을 설정하면, `pulumi up` 실행 시 클러스터에 배포하는 대신 YAML 파일로 출력한다.

TypeScript 예제:

```typescript
import * as k8s from "@pulumi/kubernetes";

// renderYamlToDirectory가 설정된 Provider 생성
const renderProvider = new k8s.Provider("k8s-yaml-renderer", {
    renderYamlToDirectory: "yaml",
});

// NGINX Deployment + LoadBalancer Service
const labels = { "app": "nginx" };
const dep = new k8s.apps.v1.Deployment("nginx-dep", {
    spec: {
        selector: { matchLabels: labels },
        replicas: 1,
        template: {
            metadata: { labels: labels },
            spec: { containers: [{ name: "nginx", image: "nginx" }] },
        },
    },
}, { provider: renderProvider });
const svc = new k8s.core.v1.Service("nginx-svc", {
    spec: {
        type: "LoadBalancer",
        selector: labels,
        ports: [{ port: 80 }],
    },
}, { provider: renderProvider });
```

Python 예제:

```python
from pulumi import ResourceOptions
from pulumi_kubernetes import Provider
from pulumi_kubernetes.apps.v1 import Deployment
from pulumi_kubernetes.core.v1 import Service

render_provider = Provider('k8s-yaml-rendered', render_yaml_to_directory='yaml')

labels = { 'app': 'nginx' }
dep = Deployment('nginx-dep',
    spec={
        'selector': { 'matchLabels': labels },
        'replicas': 1,
        'template': {
            'metadata': { 'labels': labels },
            'spec': { 'containers': [{ 'name': 'nginx', 'image': 'nginx' }] },
        },
    }, opts=ResourceOptions(provider=render_provider))
svc = Service('nginx-svc',
    spec={
        'type': 'LoadBalancer',
        'selector': labels,
        'ports': [{'port': 80}],
    }, opts=ResourceOptions(provider=render_provider))
```

`pulumi up` 실행 후 `yaml/` 디렉터리에 YAML 파일이 생성된다:

```
yaml/
├── 0-crd
└── 1-manifest
    ├── deployment-nginx-dep-xj8peqh3.yaml
    └── service-nginx-svc-nsnetbz3.yaml
```

```bash
# kubectl로 배포
kubectl apply -f "yaml/0-crd"
kubectl apply -f "yaml/1-manifest"
```

> **주의**: YAML 렌더링 시 리소스가 클러스터에 생성되지 않으므로 서버 사이드에서 계산되는 정보(예: Service의 IP 할당)는 사용할 수 없다. Secret 값은 렌더링된 매니페스트에 평문으로 나타난다.

### Kubernetes YAML 변환

```bash
pulumi convert --from kubernetes --language typescript --out <output_dir>
```

변환기에 대한 자세한 내용은 [pulumi-converter-kubernetes 저장소](https://github.com/pulumi/pulumi-converter-kubernetes)를 참조하라.

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

### 마이그레이션 예제: serverless.yml을 Pulumi 코드로 변환

다음은 Lambda 함수 + HTTP API 엔드포인트 + DynamoDB 테이블로 구성된 전형적인 `serverless.yml`과 이에 대응하는 Pulumi 프로그램 예제이다.

**serverless.yml 원본:**

```yaml
service: my-api

provider:
  name: aws
  runtime: nodejs20.x
  stage: dev
  environment:
    ORDERS_TABLE: !Ref OrdersTable

functions:
  createOrder:
    handler: src/handlers/createOrder.handler
    events:
      - httpApi:
          path: /orders
          method: post

resources:
  Resources:
    OrdersTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: ${self:service}-orders-${self:provider.stage}
        BillingMode: PAY_PER_REQUEST
        AttributeDefinitions:
          - AttributeName: id
            AttributeType: S
        KeySchema:
          - AttributeName: id
            KeyType: HASH
```

**Pulumi TypeScript 변환:**

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

const stage = pulumi.getStack();

// DynamoDB 테이블
const ordersTable = new aws.dynamodb.Table("orders-table", {
    name: `my-api-orders-${stage}`,
    billingMode: "PAY_PER_REQUEST",
    hashKey: "id",
    attributes: [{ name: "id", type: "S" }],
});

// Lambda 실행 역할
const lambdaRole = new aws.iam.Role("create-order-role", {
    assumeRolePolicy: aws.iam.assumeRolePolicyForPrincipal({
        Service: "lambda.amazonaws.com",
    }),
    managedPolicyArns: [aws.iam.ManagedPolicy.AWSLambdaBasicExecutionRole],
});

const lambdaPolicy = new aws.iam.RolePolicy("create-order-policy", {
    role: lambdaRole.id,
    policy: ordersTable.arn.apply(arn => JSON.stringify({
        Version: "2012-10-17",
        Statement: [{
            Effect: "Allow",
            Action: ["dynamodb:PutItem", "dynamodb:GetItem", "dynamodb:Query"],
            Resource: arn,
        }],
    })),
});

// Lambda 함수
const createOrderFn = new aws.lambda.Function("create-order", {
    runtime: aws.lambda.Runtime.NodeJS20dX,
    handler: "src/handlers/createOrder.handler",
    role: lambdaRole.arn,
    code: new pulumi.asset.FileArchive("./app"),
    environment: {
        variables: {
            ORDERS_TABLE: ordersTable.name,
        },
    },
});

// HTTP API (API Gateway v2)
const api = new aws.apigatewayv2.Api("api", {
    protocolType: "HTTP",
});

const integration = new aws.apigatewayv2.Integration("create-order-integration", {
    apiId: api.id,
    integrationType: "AWS_PROXY",
    integrationUri: createOrderFn.arn,
    payloadFormatVersion: "2.0",
});

const route = new aws.apigatewayv2.Route("create-order-route", {
    apiId: api.id,
    routeKey: "POST /orders",
    target: pulumi.interpolate`integrations/${integration.id}`,
});

const apiStage = new aws.apigatewayv2.Stage("api-stage", {
    apiId: api.id,
    name: "$default",
    autoDeploy: true,
});

const lambdaPermission = new aws.lambda.Permission("api-lambda-permission", {
    action: "lambda:InvokeFunction",
    function: createOrderFn.name,
    principal: "apigateway.amazonaws.com",
    sourceArn: pulumi.interpolate`${api.executionArn}/*/*`,
});

export const endpoint = api.apiEndpoint;
export const tableName = ordersTable.name;
```

**Pulumi Python 변환:**

```python
import json
import pulumi
import pulumi_aws as aws

stage = pulumi.get_stack()

# DynamoDB 테이블
orders_table = aws.dynamodb.Table("orders-table",
    name=f"my-api-orders-{stage}",
    billing_mode="PAY_PER_REQUEST",
    hash_key="id",
    attributes=[aws.dynamodb.TableAttributeArgs(name="id", type="S")],
)

# Lambda 실행 역할
lambda_role = aws.iam.Role("create-order-role",
    assume_role_policy=json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Action": "sts:AssumeRole",
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
        }],
    }),
    managed_policy_arns=[aws.iam.ManagedPolicy.AWS_LAMBDA_BASIC_EXECUTION_ROLE],
)

lambda_policy = aws.iam.RolePolicy("create-order-policy",
    role=lambda_role.id,
    policy=orders_table.arn.apply(lambda arn: json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Action": ["dynamodb:PutItem", "dynamodb:GetItem", "dynamodb:Query"],
            "Resource": arn,
        }],
    })),
)

# Lambda 함수
create_order_fn = aws.lambda_.Function("create-order",
    runtime=aws.lambda_.Runtime.NODE_JS20D_X,
    handler="src/handlers/createOrder.handler",
    role=lambda_role.arn,
    code=pulumi.FileArchive("./app"),
    environment=aws.lambda_.FunctionEnvironmentArgs(
        variables={"ORDERS_TABLE": orders_table.name},
    ),
)

# HTTP API (API Gateway v2)
api = aws.apigatewayv2.Api("api", protocol_type="HTTP")

integration = aws.apigatewayv2.Integration("create-order-integration",
    api_id=api.id,
    integration_type="AWS_PROXY",
    integration_uri=create_order_fn.arn,
    payload_format_version="2.0",
)

route = aws.apigatewayv2.Route("create-order-route",
    api_id=api.id,
    route_key="POST /orders",
    target=integration.id.apply(lambda id: f"integrations/{id}"),
)

api_stage = aws.apigatewayv2.Stage("api-stage",
    api_id=api.id,
    name="$default",
    auto_deploy=True,
)

lambda_permission = aws.lambda_.Permission("api-lambda-permission",
    action="lambda:InvokeFunction",
    function=create_order_fn.name,
    principal="apigateway.amazonaws.com",
    source_arn=api.execution_arn.apply(lambda arn: f"{arn}/*/*"),
)

pulumi.export("endpoint", api.api_endpoint)
pulumi.export("table_name", orders_table.name)
```

이 예제는 serverless.yml의 Lambda 함수, IAM 역할, DynamoDB 테이블, API Gateway v2 설정이 Pulumi에서 어떻게 표현되는지를 보여준다. Pulumi에서는 프로그래밍 언어의 모든 기능(재사용 가능한 함수, 루프, 조건문, 테스트 등)을 활용할 수 있다.

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
