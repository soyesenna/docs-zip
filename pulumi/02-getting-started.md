# Pulumi 클라우드별 시작하기

> https://www.pulumi.com/docs/get-started/
> https://www.pulumi.com/docs/iac/get-started/aws/
> https://www.pulumi.com/docs/iac/get-started/aws/begin/
> https://www.pulumi.com/docs/iac/get-started/aws/configure/
> https://www.pulumi.com/docs/iac/get-started/aws/modify-program/
> https://www.pulumi.com/docs/iac/get-started/aws/create-component/
> https://www.pulumi.com/docs/iac/get-started/azure/
> https://www.pulumi.com/docs/iac/get-started/azure/configure/
> https://www.pulumi.com/docs/iac/get-started/gcp/configure/
> https://www.pulumi.com/docs/iac/get-started/kubernetes/
> https://www.pulumi.com/docs/iac/get-started/kubernetes/configure/
> https://www.pulumi.com/docs/iac/get-started/kubernetes/create-project/

Pulumi는 모던 인프라스트럭처 as Code(IaC) 플랫폼으로, 익숙한 프로그래밍 언어와 도구를 사용해 클라우드에서 실행하는 모든 것을 자동화, 보안, 관리할 수 있다. Pulumi IaC는 무료 오픈소스이며, Pulumi Cloud와 선택적으로 연동하여 인프라 관리를 안전하고 간편하게 만들 수 있다.

이 문서는 AWS, Azure, Google Cloud, Kubernetes 각 클라우드 환경에서 Pulumi를 시작하는 단계별 가이드를 제공한다. 공통 워크플로우를 먼저 설명한 뒤, 각 클라우드별 필수 설정과 첫 프로젝트 코드 예제를 다룬다.

---

## 사전 준비

모든 클라우드 공통으로 Pulumi CLI가 설치되어 있어야 한다. 설치 방법은 별도 설치 가이드를 참조하며, 주요 방법은 다음과 같다.

| 설치 방법 | 명령어 |
|---|---|
| Homebrew (macOS) | `brew install pulumi/tap/pulumi` |
| 설치 스크립트 (macOS/Linux) | `curl -fsSL https://get.pulumi.com \| sh` |
| Chocolatey (Windows) | `choco install pulumi` |

설치 후 버전 확인:

```bash
pulumi version
```

정상적으로 출력되지 않으면 터미널을 재시작하여 PATH가 반영되었는지 확인한다.

Pulumi CLI는 기본적으로 Pulumi Cloud에 상태를 저장한다. Pulumi Cloud는 개인 사용자에게 무료이며 학습 시 권장되는 백엔드이다. 자체 관리 백엔드(S3, Azure Blob, GCS, 로컬)를 사용하려면 별도 문서를 참조한다.

---

## 클라우드별 사전 요구사항 비교

| 항목 | AWS | Azure | Google Cloud | Kubernetes |
|---|---|---|---|---|
| **필수 계정** | AWS 계정 | Azure 구독 | Google Cloud 프로젝트 | Kubernetes 클러스터 접근 (로컬: Minikube, kind, Docker Desktop / 클라우드: GKE, AKS, EKS) |
| **CLI 도구** | AWS CLI | Azure CLI (`az login`) | gcloud CLI (인증 완료) | kubectl (설정 완료) |
| **Python 패키지 매니저** | pip, Poetry 또는 uv | pip, Poetry 또는 uv | pip, Poetry 또는 uv | pip, Poetry 또는 uv |
| **인증 방식** | AWS 액세스 키 / 환경 변수 | Azure CLI 로그인 / 서비스 주체 | gcloud 인증 / 서비스 계정 | kubeconfig 파일 |
| **기본 리전/위치** | `us-east-1` | `WestUS2` | `US` | 클러스터에 따라 다름 |
| **프로젝트 템플릿** | `aws-typescript` / `aws-python` | `azure-typescript` / `azure-python` | `gcp-typescript` / `gcp-python` | `kubernetes-typescript` / `kubernetes-python` |
| **기본 생성 리소스** | S3 Bucket | Resource Group + Storage Account (Azure Blob Storage 기반 웹사이트) | Storage Bucket | NGINX Deployment |

### 클라우드별 인증 설정

| 클라우드 | 인증 설정 방법 |
|---|---|
| **AWS** | AWS CLI를 이미 설치 및 구성한 경우 Pulumi가 자동으로 해당 설정을 감지하여 사용한다. 별도 구성 시 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` 환경 변수 설정. AWS 프로필을 사용할 경우 `AWS_PROFILE` 환경 변수 설정 |
| **Azure** | `az login` 실행 후 구독에 로그인. Azure CLI를 이미 설치한 경우 Pulumi가 자동으로 해당 설정을 감지한다. 서비스 주체를 사용할 경우 `ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_SUBSCRIPTION_ID`, `ARM_TENANT_ID` 환경 변수 설정 |
| **Google Cloud** | gcloud CLI를 이미 설치 및 초기화한 경우 Pulumi가 자동으로 해당 설정을 감지한다. 서비스 계정 키를 사용할 경우 `GOOGLE_CREDENTIALS` 또는 `GOOGLE_APPLICATION_CREDENTIALS` 환경 변수 설정. 프로젝트를 명시하려면 `GOOGLE_PROJECT` 환경 변수 설정 |
| **Kubernetes** | Pulumi는 kubectl과 동일한 kubeconfig 파일(일반적으로 `~/.kube/config`)을 사용한다. 특정 kubeconfig 파일을 사용하려면 `KUBECONFIG` 환경 변수로 경로 지정. 컨텍스트를 Pulumi 스택 설정에 지정하려면 `pulumi config set kubernetes:context <context-name>` 실행 |

> **Pulumi ESC OIDC 추천**: 장수명(static) 자격 증명 대신 Pulumi ESC의 클라우드별 로그인 지원을 통해 OpenID Connect(OIDC) 기반의 단기 자격 증명을 사용하는 것이 보안 모범 사례이다. AWS는 `aws-login`, Azure는 `azure-login`, Google Cloud는 `gcp-login` ESC 프로바이더를 통해 OIDC 인증을 지원한다.

### AWS 인증 상세 설정

AWS CLI를 이미 설치하고 구성한 경우 Pulumi가 자동으로 해당 설정을 감지하여 사용한다. IAM 사용자 계정은 S3 버킷 배포 및 관리 권한이 있는 프로그래밍 액세스 권한이 필요하다.

**액세스 확인**:

```bash
aws sts get-caller-identity
```

사용자 ID, 계정, ARN이 출력되면 구성이 올바른 것이다.

**환경 변수 방식** (CI/CD 파이프라인에서 권장):

```bash
# macOS/Linux
export AWS_ACCESS_KEY_ID="<YOUR_ACCESS_KEY_ID>"
export AWS_SECRET_ACCESS_KEY="<YOUR_SECRET_ACCESS_KEY>"
```

```powershell
# Windows PowerShell
$env:AWS_ACCESS_KEY_ID = "<YOUR_ACCESS_KEY_ID>"
$env:AWS_SECRET_ACCESS_KEY = "<YOUR_SECRET_ACCESS_KEY>"
```

**AWS 프로필 사용**:

```bash
# macOS/Linux
export AWS_PROFILE="<YOUR_PROFILE_NAME>"
```

```powershell
# Windows PowerShell
$env:AWS_PROFILE = "<YOUR_PROFILE_NAME>"
```

### Azure 인증 상세 설정

Azure CLI를 이미 설치하고 구성한 경우 Pulumi가 자동으로 해당 설정을 감지한다. 스토리지 계정 및 Blob 컨테이너 배포/관리 권한이 있는 계정이 필요하다.

**액세스 확인**:

```bash
az account show
```

구독 정보가 출력되면 구성이 올바른 것이다.

**서비스 주체 방식** (CI/CD 파이프라인에서 권장):

```bash
# macOS/Linux
export ARM_CLIENT_ID="<YOUR_CLIENT_ID>"
export ARM_CLIENT_SECRET="<YOUR_CLIENT_SECRET>"
export ARM_TENANT_ID="<YOUR_TENANT_ID>"
export ARM_SUBSCRIPTION_ID="<YOUR_SUBSCRIPTION_ID>"
```

```powershell
# Windows PowerShell
$env:ARM_CLIENT_ID = "<YOUR_CLIENT_ID>"
$env:ARM_CLIENT_SECRET = "<YOUR_CLIENT_SECRET>"
$env:ARM_TENANT_ID = "<YOUR_TENANT_ID>"
$env:ARM_SUBSCRIPTION_ID = "<YOUR_SUBSCRIPTION_ID>"
```

### Google Cloud 인증 상세 설정

gcloud CLI를 이미 설치하고 초기화한 경우 Pulumi가 자동으로 해당 설정을 감지한다. Cloud Storage 버킷 배포/관리 권한이 있는 계정이 필요하다.

**액세스 확인**:

```bash
gcloud config list
```

활성 계정과 프로젝트가 출력되면 구성이 올바른 것이다. `gcloud auth list`로 인증 상태를 추가로 확인할 수 있다.

**서비스 계정 키 방식** (CI/CD 파이프라인에서 권장):

```bash
# macOS/Linux - JSON 키 파일 내용 직접 설정
export GOOGLE_CREDENTIALS="$(cat ~/path/to/service-account-key.json)"
# 또는 파일 경로 지정
export GOOGLE_APPLICATION_CREDENTIALS="$HOME/path/to/service-account-key.json"
```

```powershell
# Windows PowerShell
$env:GOOGLE_CREDENTIALS = (Get-Content -Path "C:\path\to\service-account-key.json" -Raw)
# 또는 파일 경로 지정
$env:GOOGLE_APPLICATION_CREDENTIALS = "C:\path\to\service-account-key.json"
```

**프로젝트 명시 설정**:

```bash
export GOOGLE_PROJECT="<YOUR_PROJECT_ID>"
```

### Kubernetes 인증 상세 설정

Pulumi는 kubectl과 동일한 kubeconfig 파일(일반적으로 `~/.kube/config`)을 사용한다. kubectl이 정상 동작하면 Pulumi도 자동으로 클러스터에 접근할 수 있다.

**액세스 확인**:

```bash
kubectl cluster-info
kubectl get nodes
```

컨트롤 플레인과 노드 정보가 출력되면 구성이 올바른 것이다.

**현재 컨텍스트 확인**:

```bash
kubectl config current-context
```

**대체 kubeconfig 지정**:

```bash
# macOS/Linux
export KUBECONFIG="$HOME/path/to/kubeconfig"
```

```powershell
# Windows PowerShell
$env:KUBECONFIG = "C:\path\to\kubeconfig"
```

**Pulumi 스택 설정으로 컨텍스트 지정**:

```bash
pulumi config set kubernetes:context my-cluster-context
```

---

## 공통 워크플로우

모든 클라우드에서 동일한 5단계 워크플로우를 따른다.

### 1단계: 프로젝트 생성

프로젝트는 클라우드 리소스를 정의하는 프로그램이다. 각 프로젝트는 자체 디렉터리에 존재한다.

```bash
# 프로젝트 디렉터리 생성 및 이동
mkdir quickstart
cd quickstart

# 클라우드별 템플릿으로 프로젝트 초기화
# AWS 예시
pulumi new aws-typescript
# Azure 예시
pulumi new azure-typescript
# Google Cloud 예시
pulumi new gcp-typescript
# Kubernetes 예시
pulumi new kubernetes-typescript
```

`pulumi new` 명령은 프로젝트 초기화, Stack 생성, 설정을 대화형으로 진행한다. Stack은 프로젝트의 인스턴스이며 dev, staging, prod 등 서로 다른 설정을 가진 여러 Stack을 가질 수 있다.

초기화 시 클라우드별 설정 값을 입력하라는 프롬프트가 표시된다.

| 클라우드 | 설정 프롬프트 예시 | 기본값 |
|---|---|---|
| AWS | `aws:region` (배포할 AWS 리전) | `us-east-1` |
| Azure | `azure-native:location` (사용할 Azure 위치) | `WestUS2` |
| Google Cloud | `gcp:project` (배포할 Google Cloud 프로젝트 ID) | 없음 (직접 입력) |
| Kubernetes | 설정 없음 | - |

최초 실행 시 Pulumi Cloud 로그인 프롬프트가 표시된다. 이는 무료 서비스이며 상태를 안전하게 관리해 준다.

프로젝트 생성 후 디렉터리 구조는 다음과 같다.

| 파일 | 설명 |
|---|---|
| `index.ts` / `__main__.py` | 프로젝트의 메인 코드 (클라우드 리소스 선언) |
| `Pulumi.yaml` | 프로젝트 메타데이터 (이름, 런타임 등) |
| `Pulumi.<stack>.yaml` | Stack 설정 값 |

### 2단계: 코드 작성 및 pulumi up

코드를 작성한 후 `pulumi up` 명령으로 배포한다.

```bash
pulumi up
```

이 명령은 코드에 정의된 리소스를 실제 클라우드에 생성한다. 실행 전 미리보기(preview)가 표시되어 어떤 리소스가 생성/수정/삭제되는지 확인할 수 있다.

### 3단계: 수정 및 업데이트

코드를 수정한 후 다시 `pulumi up`을 실행하면 변경 사항이 클라우드에 반영된다. Pulumi는 이전 상태와 비교하여 최소한의 변경만 수행한다.

#### AWS S3 정적 웹사이트 변환 예시

AWS의 경우 기본 S3 Bucket을 정적 웹사이트로 변환하려면 세 가지 추가 리소스가 필요하다.

| 리소스 | 역할 |
|---|---|
| `BucketWebsiteConfiguration` | Bucket을 웹사이트로 구성 (`indexDocument` 설정) |
| `BucketOwnershipControls` | Bucket 접근 제어 구성 (`ObjectWriter` 소유권 설정) |
| `BucketPublicAccessBlock` | 공용 접근 허용 (기본적으로 비활성화되어 있으므로 명시적 활성화 필요) |

**TypeScript 예시**:

```typescript
// S3 Bucket을 웹사이트로 전환
const website = new aws.s3.BucketWebsiteConfiguration("website", {
    bucket: bucket.id,
    indexDocument: { suffix: "index.html" },
});

// 접근 제어 구성 허용
const ownershipControls = new aws.s3.BucketOwnershipControls("ownership-controls", {
    bucket: bucket.id,
    rule: { objectOwnership: "ObjectWriter" },
});

// 공용 접근 허용
const publicAccessBlock = new aws.s3.BucketPublicAccessBlock("public-access-block", {
    bucket: bucket.id,
    blockPublicAcls: false,
});
```

**Python 예시**:

```python
# S3 Bucket을 웹사이트로 전환
website = s3.BucketWebsiteConfiguration("website",
    bucket=bucket.id,
    index_document={"suffix": "index.html"},
)

# 접근 제어 구성 허용
ownership_controls = s3.BucketOwnershipControls("ownership-controls",
    bucket=bucket.id,
    rule={"object_ownership": "ObjectWriter"},
)

# 공용 접근 허용
public_access_block = s3.BucketPublicAccessBlock("public-access-block",
    bucket=bucket.id,
    block_public_acls=False,
)
```

이후 `index.html` 파일을 업로드하고 Bucket Policy를 설정하여 웹사이트를 공개한다. `pulumi up`을 실행하면 추가된 리소스가 클라우드에 반영된다.

#### index.html 파일 생성

프로젝트 디렉터리에 `index.html` 파일을 생성한다.

```html
<html>
<body>
    <h1>Hello, Pulumi!</h1>
</body>
</html>
```

#### BucketObject로 index.html 업로드

`index.html`을 S3 버킷에 업로드하는 `BucketObject` 리소스를 추가한다. 이 리소스는 Pulumi의 **에셋(Asset)** 개념을 사용하여 로컬 파일을 클라우드에 업로드한다.

**TypeScript 예시**:

```typescript
// index.html 파일을 S3 버킷에 업로드
const bucketObject = new aws.s3.BucketObject("index.html", {
    bucket: bucket.id,
    source: new pulumi.asset.FileAsset("index.html"),
    contentType: "text/html",
    acl: "public-read",
}, { dependsOn: [ownershipControls, publicAccessBlock] });
```

**Python 예시**:

```python
# index.html 파일을 S3 버킷에 업로드
bucket_object = s3.BucketObject(
    'index.html',
    bucket=bucket.id,
    source=pulumi.FileAsset('index.html'),
    content_type='text/html',
    acl='public-read',
    opts=pulumi.ResourceOptions(
        depends_on=[ownership_controls, public_access_block]
    ),
)
```

`dependsOn`을 명시하는 이유는 `ownershipControls`와 `publicAccessBlock`이 먼저 생성되어야 AWS가 `public-acl` 권한을 허용하기 때문이다. Pulumi는 일반적으로 종속성을 자동 추적하지만, 이 경우 AWS 내부 사이드 이펙트로 인해 Pulumi가 인식할 수 없는 종속성이므로 수동 지정이 필요하다.

#### 웹사이트 URL 내보내기

버킷 웹사이트의 URL을 출력으로 내보내면 배포 후 쉽게 접근할 수 있다.

**TypeScript**:

```typescript
export const url = pulumi.interpolate`http://${website.websiteEndpoint}`;
```

**Python**:

```python
pulumi.export('url', pulumi.Output.concat('http://', website.website_endpoint))
```

### 4단계: pulumi destroy

모든 리소스를 삭제하려면 다음 명령을 사용한다.

```bash
pulumi destroy
```

Stack 자체도 삭제하려면 `pulumi stack rm <stack-name>`을 추가로 실행한다.

---

## AWS 첫 프로젝트

### TypeScript

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";
import * as awsx from "@pulumi/awsx";

// Create an AWS resource (S3 Bucket)
const bucket = new aws.s3.Bucket("my-bucket");

// Export the name of the bucket
export const bucketName = bucket.id;
```

### Python

```python
import pulumi
from pulumi_aws import s3

# Create an AWS resource (S3 Bucket)
bucket = s3.Bucket("my-bucket")

# Export the name of the bucket
pulumi.export("bucket_name", bucket.id)
```

### 프로젝트 초기화 명령어

| 언어 | 명령어 |
|---|---|
| TypeScript | `pulumi new aws-typescript` |
| Python | `pulumi new aws-python` |
| Go | `pulumi new aws-go` |
| C# | `pulumi new aws-csharp` |
| Java | `pulumi new aws-java` |
| YAML | `pulumi new aws-yaml` |

---

## Azure 첫 프로젝트

### TypeScript

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as resources from "@pulumi/azure-native/resources";
import * as storage from "@pulumi/azure-native/storage";

// Create an Azure Resource Group
const resourceGroup = new resources.ResourceGroup("resourceGroup");

// Create an Azure resource (Storage Account)
const storageAccount = new storage.StorageAccount("sa", {
    resourceGroupName: resourceGroup.name,
    sku: {
        name: storage.SkuName.Standard_LRS,
    },
    kind: storage.Kind.StorageV2,
});

// Export the storage account name
export const storageAccountName = storageAccount.name;
```

### Python

```python
import pulumi
from pulumi_azure_native import storage
from pulumi_azure_native import resources

# Create an Azure Resource Group
resource_group = resources.ResourceGroup("resource_group")

# Create an Azure Storage Account
account = storage.StorageAccount(
    "sa",
    resource_group_name=resource_group.name,
    sku={
        "name": storage.SkuName.STANDARD_LRS,
    },
    kind=storage.Kind.STORAGE_V2,
)

# Export the storage account name
pulumi.export("storage_account_name", account.name)
```

### 프로젝트 초기화 명령어

| 언어 | 명령어 |
|---|---|
| TypeScript | `pulumi new azure-typescript` |
| Python | `pulumi new azure-python` |
| Go | `pulumi new azure-go` |
| C# | `pulumi new azure-csharp` |
| Java | `pulumi new azure-java` |
| YAML | `pulumi new azure-yaml` |

---

## Google Cloud 첫 프로젝트

### TypeScript

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as gcp from "@pulumi/gcp";

// Create a Google Cloud resource (Storage Bucket)
const bucket = new gcp.storage.Bucket("my-bucket", {
    location: "US",
});

// Export the DNS name of the bucket
export const bucketName = bucket.url;
```

### Python

```python
import pulumi
from pulumi_gcp import storage

# Create a Google Cloud resource (Storage Bucket)
bucket = storage.Bucket("my-bucket", location="US")

# Export the DNS name of the bucket
pulumi.export("bucket_name", bucket.url)
```

### 프로젝트 초기화 명령어

| 언어 | 명령어 |
|---|---|
| TypeScript | `pulumi new gcp-typescript` |
| Python | `pulumi new gcp-python` |
| Go | `pulumi new gcp-go` |
| C# | `pulumi new gcp-csharp` |
| Java | `pulumi new gcp-java` |
| YAML | `pulumi new gcp-yaml` |

---

## Kubernetes 첫 프로젝트

### TypeScript

```typescript
import * as k8s from "@pulumi/kubernetes";

const appLabels = { app: "nginx" };
const deployment = new k8s.apps.v1.Deployment("nginx", {
    spec: {
        selector: { matchLabels: appLabels },
        replicas: 1,
        template: {
            metadata: { labels: appLabels },
            spec: {
                containers: [{
                    name: "nginx",
                    image: "nginx",
                }],
            },
        },
    },
});

export const name = deployment.metadata.name;
```

### Python

```python
import pulumi
from pulumi_kubernetes.apps.v1 import Deployment

app_labels = {"app": "nginx"}

deployment = Deployment(
    "nginx",
    spec={
        "selector": {"match_labels": app_labels},
        "replicas": 1,
        "template": {
            "metadata": {"labels": app_labels},
            "spec": {
                "containers": [{
                    "name": "nginx",
                    "image": "nginx",
                }],
            },
        },
    })

pulumi.export("name", deployment.metadata["name"])
```

### Go

```go
package main

import (
    "github.com/pulumi/pulumi-kubernetes/sdk/v4/go/kubernetes/apps/v1"
    "github.com/pulumi/pulumi-kubernetes/sdk/v4/go/kubernetes/core/v1"
    metav1 "github.com/pulumi/pulumi-kubernetes/sdk/v4/go/kubernetes/meta/v1"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        appLabels := pulumi.StringMap{
            "app": pulumi.String("nginx"),
        }
        _, err := v1.NewDeployment(ctx, "nginx", &v1.DeploymentArgs{
            Spec: &v1.DeploymentSpecArgs{
                Selector: &metav1.LabelSelectorArgs{
                    MatchLabels: appLabels,
                },
                Replicas: pulumi.Int(1),
                Template: &core.PodTemplateSpecArgs{
                    Metadata: &metav1.ObjectMetaArgs{
                        Labels: appLabels,
                    },
                    Spec: &core.PodSpecArgs{
                        Containers: core.ContainerArray{
                            &core.ContainerArgs{
                                Name:  pulumi.String("nginx"),
                                Image: pulumi.String("nginx"),
                            },
                        },
                    },
                },
            },
        })
        if err != nil {
            return err
        }
        return nil
    })
}
```

### C#

```csharp
using Pulumi;
using Pulumi.Kubernetes.Apps.V1;
using Pulumi.Kubernetes.Types.Inputs.Apps.V1;
using Pulumi.Kubernetes.Types.Inputs.Core.V1;
using Pulumi.Kubernetes.Types.Inputs.Meta.V1;

return await Deployment.RunAsync(() =>
{
    var appLabels = new InputMap<string>
    {
        { "app", "nginx" },
    };

    var deployment = new Pulumi.Kubernetes.Apps.V1.Deployment("nginx", new DeploymentArgs
    {
        Spec = new DeploymentSpecArgs
        {
            Selector = new LabelSelectorArgs
            {
                MatchLabels = appLabels,
            },
            Replicas = 1,
            Template = new PodTemplateSpecArgs
            {
                Metadata = new ObjectMetaArgs
                {
                    Labels = appLabels,
                },
                Spec = new PodSpecArgs
                {
                    Containers = new ContainerArgs
                    {
                        Name = "nginx",
                        Image = "nginx",
                    },
                },
            },
        },
    });
});
```

### Java

```java
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.kubernetes.apps.v1.Deployment;
import com.pulumi.kubernetes.apps.v1.DeploymentArgs;
import com.pulumi.kubernetes.meta.v1.inputs.LabelSelectorArgs;
import com.pulumi.kubernetes.meta.v1.inputs.ObjectMetaArgs;
import com.pulumi.kubernetes.core.v1.inputs.ContainerArgs;
import com.pulumi.kubernetes.core.v1.inputs.PodSpecArgs;
import com.pulumi.kubernetes.core.v1.inputs.PodTemplateSpecArgs;
import java.util.Map;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {
            var appLabels = Map.of("app", "nginx");

            var deployment = new Deployment("nginx",
                DeploymentArgs.builder()
                    .spec(DeploymentSpecArgs.builder()
                        .selector(LabelSelectorArgs.builder()
                            .matchLabels(appLabels)
                            .build())
                        .replicas(1)
                        .template(PodTemplateSpecArgs.builder()
                            .metadata(ObjectMetaArgs.builder()
                                .labels(appLabels)
                                .build())
                            .spec(PodSpecArgs.builder()
                                .containers(ContainerArgs.builder()
                                    .name("nginx")
                                    .image("nginx")
                                    .build())
                                .build())
                            .build())
                        .build())
                    .build());

            ctx.export("name", deployment.metadata().applyValue(m -> m.name()));
        });
    }
}
```

### YAML

```yaml
name: quickstart
runtime: yaml
resources:
  nginx:
    type: kubernetes:apps/v1:Deployment
    properties:
      spec:
        selector:
          matchLabels:
            app: nginx
        replicas: 1
        template:
          metadata:
            labels:
              app: nginx
          spec:
            containers:
              - name: nginx
                image: nginx
outputs:
  name: ${nginx.metadata.name}
```

### 프로젝트 초기화 명령어

| 언어 | 명령어 |
|---|---|
| TypeScript | `pulumi new kubernetes-typescript` |
| Python | `pulumi new kubernetes-python` |
| Go | `pulumi new kubernetes-go` |
| C# | `pulumi new kubernetes-csharp` |
| Java | `pulumi new kubernetes-java` |
| YAML | `pulumi new kubernetes-yaml` |

### 사전 요구사항

Kubernetes 튜토리얼을 시작하기 전에 다음이 필요하다.

| 요구사항 | 설명 |
|---|---|
| Kubernetes 클러스터 | 로컬 클러스터(Minikube, kind, Docker Desktop) 또는 클라우드 관리 클러스터(GKE, AKS, EKS) |
| kubectl | 설치 및 클러스터에 연결되도록 구성 완료 |
| 언어 런타임 | Node.js + npm, Python + pip, Go, .NET, Java 11+ + Maven 3.6.1+ 중 선택 |

클러스터 접근을 테스트하려면 다음 명령을 실행한다.

```bash
kubectl cluster-info
kubectl get nodes
```

Pulumi는 kubectl과 동일한 kubeconfig 파일(일반적으로 `~/.kube/config`)을 사용하므로, kubectl이 정상 동작하면 Pulumi도 자동으로 클러스터에 접근할 수 있다.

---

## 컴포넌트(Component)로 재사용 가능한 인프라 추상화

컴포넌트는 인프라 복잡성을 캡슐화하고 공유 및 재사용을 가능하게 하는 추상화이다. 공통 패턴을 복사-붙여넣기 대신 컴포넌트로 인코딩할 수 있다.

### AWS S3 웹사이트 컴포넌트 예시

#### TypeScript

```typescript
// website.ts
import * as aws from "@pulumi/aws";
import * as pulumi from "@pulumi/pulumi";

// Arguments for the AWS S3 hosted static website component.
export interface AwsS3WebsiteArgs {
    files: string[]; // a list of files to serve.
}

// A component that encapsulates creating an AWS S3 hosted static website.
export class AwsS3Website extends pulumi.ComponentResource {
    public readonly url: pulumi.Output<string>; // the S3 website url.

    constructor(name: string, args: AwsS3WebsiteArgs, opts?: pulumi.ComponentResourceOptions) {
        super("quickstart:index:AwsS3Website", name, args, opts);
        // Component initialization will go here...
        this.registerOutputs({}); // Signal component completion.
    }
}
```

#### Python

```python
# website.py
import pulumi
from pulumi_aws import s3
from typing import List

# A component that encapsulates creating an AWS S3 hosted static website.
class AwsS3Website(pulumi.ComponentResource):
    def __init__(self, name: str, files: List[str] = None, opts=None):
        super().__init__('quickstart:index:AwsS3Website', name, {'files': files}, opts)
        # Component initialization will go here...
        self.register_outputs({})  # Signal component completion.
```

### 컴포넌트 사용 예시

```typescript
// TypeScript
const website = new AwsS3Website("my-website", { files: ["index.html"] });
```

```python
# Python
website = AwsS3Website("my-website", files=["index.html"])
```

### 컴포넌트에 리소스 추가 (리팩토링)

컴포넌트의 생성자에 모든 S3 웹사이트 리소스를 이동시킨다. 다음 네 가지 변경을 수행한다.

1. `index.ts`의 모든 리소스를 컴포넌트 생성자로 이동
2. 각 리소스의 `parent`를 컴포넌트로 설정
3. `files` 배열을 루프하여 BucketObject를 동적으로 생성
4. 웹사이트 URL을 컴포넌트의 `url` 속성에 할당

#### 완성된 TypeScript 컴포넌트

```typescript
// website.ts
import * as aws from "@pulumi/aws";
import * as pulumi from "@pulumi/pulumi";

export interface AwsS3WebsiteArgs {
    files: string[];
}

export class AwsS3Website extends pulumi.ComponentResource {
    public readonly url: pulumi.Output<string>;

    constructor(name: string, args: AwsS3WebsiteArgs, opts?: pulumi.ComponentResourceOptions) {
        super("quickstart:index:AwsS3Website", name, args, opts);

        // S3 Bucket 생성
        const bucket = new aws.s3.Bucket("my-bucket", {}, { parent: this });

        // Bucket을 웹사이트로 전환
        const website = new aws.s3.BucketWebsiteConfiguration("website", {
            bucket: bucket.id,
            indexDocument: { suffix: "index.html" },
        }, { parent: this });

        // 접근 제어 구성 허용
        const ownershipControls = new aws.s3.BucketOwnershipControls("ownership-controls", {
            bucket: bucket.id,
            rule: { objectOwnership: "ObjectWriter" },
        }, { parent: this });

        // 공용 접근 허용
        const publicAccessBlock = new aws.s3.BucketPublicAccessBlock("public-access-block", {
            bucket: bucket.id,
            blockPublicAcls: false,
        }, { parent: this });

        // files 배열의 각 파일을 BucketObject로 업로드
        for (const file of args.files) {
            new aws.s3.BucketObject(file, {
                bucket: bucket.id,
                source: new pulumi.asset.FileAsset(file),
                contentType: "text/html",
                acl: "public-read",
            }, { dependsOn: [ownershipControls, publicAccessBlock], parent: this });
        }

        // URL 출력
        this.url = pulumi.interpolate`http://${website.websiteEndpoint}`;
        this.registerOutputs({ url: this.url });
    }
}
```

#### 완성된 Python 컴포넌트

```python
# website.py
import pulumi
from pulumi_aws import s3
from typing import List

class AwsS3Website(pulumi.ComponentResource):
    def __init__(self, name: str, files: List[str] = None, opts=None):
        super().__init__('quickstart:index:AwsS3Website', name, {'files': files}, opts)

        # S3 Bucket 생성
        bucket = s3.Bucket('my-bucket',
            opts=pulumi.ResourceOptions(parent=self),
        )

        # Bucket을 웹사이트로 전환
        website = s3.BucketWebsiteConfiguration("website",
            bucket=bucket.id,
            index_document={"suffix": "index.html"},
            opts=pulumi.ResourceOptions(parent=self),
        )

        # 접근 제어 구성 허용
        ownership_controls = s3.BucketOwnershipControls('ownership-controls',
            bucket=bucket.id,
            rule={"object_ownership": "ObjectWriter"},
            opts=pulumi.ResourceOptions(parent=self),
        )

        # 공용 접근 허용
        public_access_block = s3.BucketPublicAccessBlock('public-access-block',
            bucket=bucket.id,
            block_public_acls=False,
            opts=pulumi.ResourceOptions(parent=self),
        )

        # files 배열의 각 파일을 BucketObject로 업로드
        for file in files:
            s3.BucketObject(file,
                bucket=bucket.id,
                source=pulumi.FileAsset(file),
                content_type='text/html',
                acl='public-read',
                opts=pulumi.ResourceOptions(
                    depends_on=[ownership_controls, public_access_block],
                    parent=self,
                ),
            )

        # URL 출력
        self.url = pulumi.Output.concat('http://', website.website_endpoint)
        self.register_outputs({'url': self.url})
```

> YAML은 컴포넌트를 작성하기 위한 언어 기능이 부족하므로 컴포넌트 섹션은 건너뛸 수 있다.

---

## 완료 후 다음 단계

Pulumi로 클라우드 리소스를 프로비저닝하는 기본 워크플로우를 완료했다면 다음을 수행했다.

- 새 Pulumi 프로젝트 생성
- 클라우드 리소스 프로비저닝 (S3 Bucket, Storage Account, Storage Bucket, NGINX Deployment 등)
- 정적 웹사이트로 구성
- 재사용 가능한 컴포넌트 생성
- 프로비저닝한 모든 리소스 삭제

### 추천 다음 단계

| 단계 | 설명 |
|---|---|
| Pulumi ESC 체험 | 중앙 집중식 시크릿 관리 및 오케스트레이션 서비스. 환경 변수와 시크릿을 안전하게 관리. 시크릿 스프롤 방지, RBAC 기반 접근 제어, .env 파일 대체 |
| 튜토리얼 진행 | 클라우드별 핵심 Pulumi 개념을 안내하는 튜토리얼 |
| 템플릿으로 새 프로젝트 시작 | 정적 웹사이트, 서버리스 애플리케이션, 가상 머신, 컨테이너 서비스, Kubernetes 클러스터 등 일반적인 아키텍처 템플릿 |
| 공식 문서 심화 학습 | 프로젝트, Stack, 설정, 시크릿, 리소스, 상태 등 핵심 개념 학습 |
