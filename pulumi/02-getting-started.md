# Pulumi 클라우드별 시작하기

> https://www.pulumi.com/docs/get-started/
> https://www.pulumi.com/docs/iac/get-started/aws/
> https://www.pulumi.com/docs/iac/get-started/azure/
> https://www.pulumi.com/docs/iac/get-started/gcp/
> https://www.pulumi.com/docs/iac/get-started/kubernetes/

Pulumi는 모던 인프라스트럭처 as Code(IaC) 플랫폼으로, 익숙한 프로그래밍 언어와 도구를 사용해 클라우드 리소스를 안전하고 일관되게 배포, 변경, 관리할 수 있다. Pulumi IaC는 무료 오픈소스이며, Pulumi Cloud와 선택적으로 연동하여 인프라 관리를 더욱 안전하고 간편하게 만들 수 있다.

이 문서는 AWS, Azure, Google Cloud, Kubernetes 각 클라우드 환경에서 Pulumi를 시작하는 단계별 가이드를 제공한다. 공통 워크플로우를 먼저 설명한 뒤, 각 클라우드별 필수 설정과 첫 프로젝트 코드 예제를 다룬다.

---

## 사전 준비

모든 클라우드 공통으로 Pulumi CLI가 설치되어 있어야 한다. 설치 방법은 별도 설치 가이드를 참조하며, 주요 방법은 다음과 같다.

| 설치 방법 | 명령어 |
|---|---|
| Homebrew (macOS) | `brew install pulumi/tap/pulumi` |
| 설치 스크립트 (macOS/Linux) | `curl -fsSL https://get.pulumi.com \| sh` |
| MacPorts | `sudo port install pulumi` |

Pulumi CLI는 기본적으로 Pulumi Cloud에 상태를 저장한다. Pulumi Cloud는 개인 사용자에게 무료이며 학습 시 권장되는 백엔드이다. 자체 관리 백엔드(S3, Azure Blob, GCS, 로컬)를 사용하려면 별도 문서를 참조한다.

---

## 클라우드별 사전 요구사항 비교

| 항목 | AWS | Azure | Google Cloud | Kubernetes |
|---|---|---|---|---|
| **필수 계정** | AWS 계정 | Azure 구독 | Google Cloud 프로젝트 | Kubernetes 클러스터 접근 |
| **CLI 도구** | AWS CLI | Azure CLI (`az login`) | gcloud CLI (인증 완료) | kubectl (설정 완료) |
| **인증 방식** | AWS 액세스 키 / 환경 변수 | Azure CLI 로그인 / 서비스 주체 | gcloud 인증 / 서비스 계정 | kubeconfig 파일 |
| **기본 리전/위치** | `us-east-1` | `WestUS2` | `US` | 클러스터에 따라 다름 |
| **프로젝트 템플릿** | `aws-typescript` / `aws-python` | `azure-typescript` / `azure-python` | `gcp-typescript` / `gcp-python` | `kubernetes-typescript` / `kubernetes-python` |
| **기본 생성 리소스** | S3 Bucket | Resource Group + Storage Account | Storage Bucket | NGINX Deployment |

### 클라우드별 인증 설정

| 클라우드 | 인증 설정 방법 |
|---|---|
| **AWS** | AWS CLI를 사용하거나 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` 환경 변수 설정 |
| **Azure** | `az login` 실행 후 구독에 로그인. 서비스 주체를 사용할 경우 `ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_SUBSCRIPTION_ID`, `ARM_TENANT_ID` 환경 변수 설정 |
| **Google Cloud** | `gcloud auth login` 및 `gcloud config set project <PROJECT_ID>` 실행. 서비스 계정 키를 사용할 경우 `GOOGLE_CREDENTIALS` 또는 `GOOGLE_APPLICATION_CREDENTIALS` 환경 변수 설정 |
| **Kubernetes** | kubeconfig 파일이 `~/.kube/config`에 위치하거나 `KUBECONFIG` 환경 변수로 경로 지정 |

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
| Kubernetes 클러스터 | 로컬(Minikube 등) 또는 클라우드 기반 |
| kubectl | 설치 및 클러스터에 연결되도록 구성 완료 |
| 언어 런타임 | Node.js + npm, Python + pip, Go, .NET, Java 11+ + Maven 3.6.1+ 중 선택 |

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
| Pulumi ESC 체험 | 중앙 집중식 시크릿 관리 및 오케스트레이션 서비스. 환경 변수와 시크릿을 안전하게 관리 |
| 튜토리얼 진행 | 클라우드별 핵심 Pulumi 개념을 안내하는 튜토리얼 |
| 템플릿으로 새 프로젝트 시작 | 정적 웹사이트, 서버리스 애플리케이션, 가상 머신, 컨테이너 서비스, Kubernetes 클러스터 등 일반적인 아키텍처 템플릿 |
| 공식 문서 심화 학습 | 프로젝트, Stack, 설정, 시크릿, 리소스, 상태 등 핵심 개념 학습 |
| Pulumi ESC | 시크릿 스프롤 방지, RBAC 기반 접근 제어, .env 파일 대체 |
