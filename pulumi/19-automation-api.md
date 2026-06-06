# Pulumi Automation API

> **원문**
> - https://www.pulumi.com/docs/iac/concepts/automation-api/
> - https://www.pulumi.com/docs/iac/guides/building-extending/automation-api/

Pulumi Automation API는 Pulumi CLI 없이 Pulumi 프로그램을 프로그래밍 방식으로 실행할 수 있는 인터페이스다. `pulumi up`, `pulumi preview`, `pulumi destroy`, `pulumi stack init` 등 CLI 기능을 강타입(strongly typed) SDK로 캡슐화하여, 셸에서 `pulumi` 명령을 호출하는 대신 자체 애플리케이션 내에서 Pulumi 엔진을 구동할 수 있다. Automation API는 Pulumi SDK 내의 네임스페이스로 배포되며, 오픈소스 제품의 일부다.

---

## 주요 사용 사례

Automation API는 인프라 프로비저닝이 더 큰 애플리케이션이나 워크플로의 일부인 시나리오에 적합하다.

| 사용 사례 | 설명 |
|-----------|------|
| **CI/CD 워크플로 내 인프라 배포** | 파이프라인에서 Automation API로 직접 배포 수행 |
| **통합 테스트** | 인프라 변경 사항에 대한 자동화된 테스트 실행 |
| **멀티 스테이지 배포** | Blue-Green 배포 패턴 등 다단계 배포 오케스트레이션 |
| **애플리케이션 코드 포함 배포** | 데이터베이스 마이그레이션 등 애플리케이션 코드와 인프라를 함께 배포 |
| **커스텀 CLI/도구 구축** | Pulumi를 기반으로 한 고수준 커스텀 도구 개발 |
| **REST/gRPC API 노출** | Pulumi를 REST 또는 gRPC API 뒤에 배치하여 서비스화 |
| **디버깅** | 인라인 프로그램으로 단일 진입점을 사용해 Pulumi 프로그램 디버깅 |

---

## 핵심 개념

### Workspace

Automation API는 런타임 커스터마이징을 위해 `Workspace` 인터페이스를 정의한다. `Workspace`는 단일 Pulumi 프로젝트, 프로그램, 하나 이상의 스택을 포함하는 실행 컨텍스트다. 플러그인 설치, 환경 설정(`$PULUMI_HOME`), 스택의 생성/삭제/목록 조회 등 실행 환경 관리 유틸리티를 제공한다.

| Workspace 유형 | 설명 |
|----------------|------|
| **LocalWorkspace** | 기본 구현체. `Pulumi.yaml`과 `Pulumi.<stack>.yaml`을 온디스크 형식으로 사용. `ProjectSettings` 수정 시 `Pulumi.yaml` 파일이 변경되고, 스택 설정 변경 시 해당 `Pulumi.<stack>.yaml` 파일이 변경됨. CLI 기반 워크스페이스와 동일한 동작 방식 |
| **RemoteWorkspace** | Pulumi Deployments로 원격에서 Pulumi 작업을 실행하기 위한 워크스페이스. 프로그램이 원격 Git 리포지토리에 위치 |

### Stack

`Stack`은 격리된, 독립적으로 구성 가능한 Pulumi 프로그램 인스턴스다. 전체 Pulumi 라이프사이클(`up`, `preview`, `refresh`, `destroy`) 메서드와 설정 관리 메서드를 노출한다. 여러 스택은 개발, 스테이징, 프로덕션 등 개발 단계나 피처 브랜치를 나타내는 데 사용된다.

| Stack 유형 | 설명 |
|------------|------|
| **Stack** | `LocalWorkspace` 기반 스택. 로컬에서 Pulumi 라이프사이클 실행 |
| **RemoteStack** | `RemoteWorkspace`에 대응하는 스택. 동일한 라이프사이클 메서드를 원격에서 실행 |

### Program 유형

Automation API는 두 가지 유형의 Pulumi 프로그램을 구동할 수 있다.

| 프로그램 유형 | 설명 |
|--------------|------|
| **로컬 프로그램 (Local Program)** | 자체 디렉토리, `Pulumi.yaml` 파일, 프로그램 정의 파일을 가진 전통적인 CLI 기반 Pulumi 프로그램. Automation API가 CLI와 동일한 방식으로 구동 |
| **인라인 프로그램 (Inline Program)** | 별도의 온디스크 패키지나 `Pulumi.yaml`이 필요 없음. Automation API 프로그램과 같은 파일에 작성하거나 다른 패키지에서 import할 수 있는 함수 형태 |

> **주의:** 인라인 프로그램의 라이프사이클은 인라인 프로그램으로 전달된 함수/콜백/클로저 내에 완전히 포함되어야 한다. 범위 밖에서 작업을 수행하는 것은 안전하지 않으며 예측할 수 없는 동작을 초래할 수 있다.

---

## 지원 언어

Automation API는 Pulumi와 동일하게 여러 언어를 지원하며, 크로스 언어 사용도 지원한다(관리 대상 Pulumi 프로그램과 다른 언어로 작성된 프로그램에서 Automation API 실행).

| 언어 | 상태 | SDK 참조 |
|------|------|----------|
| TypeScript | Stable | [@pulumi/pulumi/automation](/docs/reference/pkg/nodejs/pulumi/pulumi/automation/) |
| JavaScript | Stable | [@pulumi/pulumi/automation](/docs/reference/pkg/nodejs/pulumi/pulumi/automation/) |
| Python | Stable | [pulumi.automation](/docs/reference/pkg/python/pulumi/#module-pulumi.automation) |
| .NET | Stable | [Pulumi.Automation](/docs/reference/pkg/dotnet/pulumi.automation/pulumi.automation.html) |
| Go | Stable | [auto 패키지](https://pkg.go.dev/github.com/pulumi/pulumi/sdk/v3/go/auto?tab=doc) |
| Java | Stable | [com.pulumi.automation](/docs/reference/pkg/java/com/pulumi/automation/package-summary.html) |

---

## CLI 프로그래밍 방식 설치

Automation API는 내부적으로 Pulumi CLI를 구동하므로, 런타임에 CLI가 사용 가능해야 한다. 사전 설치 후 `PATH`에 추가하거나, 프로그램 내에서 프로그래밍 방식으로 설치할 수 있다. TypeScript, Python, Go, .NET SDK는 SDK 버전과 일치하는 CLI를 `~/.pulumi/versions/<version>`에 다운로드하는 `install` 메서드를 제공한다.

### TypeScript

```typescript
import { LocalWorkspace, PulumiCommand } from "@pulumi/pulumi/automation";

// SDK 버전과 일치하는 CLI를 ~/.pulumi/versions/<version>에 설치
const pulumiCommand = await PulumiCommand.install();

// 워크스페이스에 pulumiCommand 옵션으로 전달
const stack = await LocalWorkspace.createOrSelectStack(args, { pulumiCommand });
```

### Python

```python
from pulumi import automation as auto

# SDK 버전과 일치하는 CLI를 ~/.pulumi/versions/<version>에 설치
pulumi_command = auto.PulumiCommand.install()

# 워크스페이스에 pulumi_command 옵션으로 전달
stack = auto.create_or_select_stack(
    stack_name="<YOUR_STACK_NAME>",
    project_name="<YOUR_PROJECT_NAME>",
    program=pulumi_program,
    opts=auto.LocalWorkspaceOptions(pulumi_command=pulumi_command)
)
```

> Java SDK는 CLI 프로그래밍 방식 설치를 지원하지 않는다. 수동으로 Pulumi CLI를 설치하여 `PATH`에서 사용 가능하도록 설정해야 한다.

---

## 인라인 프로그램 정의

인라인 프로그램은 일반 Pulumi 프로그램과 동일한 구조를 가지는 함수로 정의한다.

### TypeScript

```typescript
import * as aws from "@pulumi/aws";
import * as s3 from "@pulumi/aws/s3";

const pulumiProgram = async () => {
    const siteBucket = new s3.Bucket("s3-website-bucket", {});

    const indexContent = `<html><head>
<title>Hello S3</title><meta charset="UTF-8">
</head>
<body>Hello, world!</body></html>`;

    const ownershipControls = new aws.s3.BucketOwnershipControls("ownership-controls", {
        bucket: siteBucket.id,
        rule: { objectOwnership: "ObjectWriter" },
    });

    const publicAccessBlock = new aws.s3.BucketPublicAccessBlock("public-access-block", {
        bucket: siteBucket.id,
        blockPublicAcls: false,
    });

    const website = new aws.s3.BucketWebsiteConfigurationV2("website", {
        bucket: siteBucket.id,
        indexDocument: { suffix: "index.html" },
    });

    const object = new s3.BucketObject("index", {
        bucket: siteBucket.id,
        content: indexContent,
        contentType: "text/html; charset=utf-8",
        key: "index.html",
        acl: "public-read",
    }, {
        dependsOn: [publicAccessBlock, ownershipControls, website],
    });

    return {
        websiteUrl: website.websiteEndpoint,
    };
};
```

### Python

```python
import pulumi
import pulumi_aws as aws
from pulumi_aws import s3

def pulumi_program():
    site_bucket = s3.Bucket("s3-website-bucket")

    index_content = """
    <html>
        <head><title>Hello S3</title><meta charset="UTF-8"></head>
        <body>Hello, world!</body>
    </html>
    """

    ownership_controls = s3.BucketOwnershipControls("ownership-controls",
        bucket=site_bucket.id,
        rule={"object_ownership": "ObjectWriter"})

    public_access_block = s3.BucketPublicAccessBlock("public-access-block",
        bucket=site_bucket.id,
        block_public_acls=False)

    website = aws.s3.BucketWebsiteConfigurationV2("website",
        bucket=site_bucket.id,
        index_document={"suffix": "index.html"})

    s3.BucketObject("index",
        bucket=site_bucket.id,
        content=index_content,
        acl="public-read",
        key="index.html",
        content_type="text/html; charset=utf-8",
        opts=pulumi.ResourceOptions(depends_on=[
            public_access_block, ownership_controls, website,
        ]))

    pulumi.export("website_url", website.website_endpoint)
```

---

## Stack 연결

프로그램을 정의한 후 `Stack`과 연결해야 한다. Automation API는 스택을 선택하거나 존재하지 않으면 생성하는 편의 메서드를 제공한다.

### TypeScript

```typescript
import { LocalWorkspace, InlineProgramArgs } from "@pulumi/pulumi/automation";

const args: InlineProgramArgs = {
    stackName: "dev",
    projectName: "inlineNode",
    program: pulumiProgram
};

const stack = await LocalWorkspace.createOrSelectStack(args);
```

### Python

```python
from pulumi import automation as auto

project_name = "inline_s3_project"
stack_name = "dev"

stack = auto.create_or_select_stack(
    stack_name=stack_name,
    project_name=project_name,
    program=pulumi_program
)
```

---

## 프로바이더 플러그인 구성

AWS 등 클라우드 리소스를 배포하려면 워크스페이스에 프로바이더 플러그인을 설치하고, 스택 설정으로 리전 등을 구성해야 한다.

### TypeScript

```typescript
await stack.workspace.installPlugin("aws", "v4.0.0");
await stack.setConfig("aws:region", { value: "us-west-2" });
```

### Python

```python
stack.workspace.install_plugin("aws", "v4.0.0")
stack.set_config("aws:region", auto.ConfigValue(value="us-west-2"))
```

---

## Stack 명령 실행

Stack 객체에 대해 `up`, `preview`, `refresh`, `destroy`, `import`, `export` 등의 명령을 실행할 수 있다.

### up (배포)

### TypeScript

```typescript
// 표준 출력을 콜백으로 수신하며 배포 실행
const upRes = await stack.up({ onOutput: console.info });
console.log(`Website URL: ${upRes.outputs.websiteUrl.value}`);
```

### Python

```python
# 표준 출력을 콜백으로 수신하며 배포 실행
up_res = stack.up(on_output=print)
print(f"Website URL: {up_res.outputs['website_url'].value}")
```

`up` 결과는 스택 출력값(outputs)과 변경 요약(summary)을 포함한다. 이를 활용해 프로그램 내에서 조건부 로직을 구현할 수 있다. 예를 들어, 업데이트된 리소스가 없을 때 다른 동작을 수행하거나, 출력값을 사용해 동일한 Automation 프로그램 내에서 다른 Pulumi 프로그램을 구동할 수 있다.

---

## 로컬 패키지 사용

Pulumi Registry에 게시되지 않은 로컬 패키지를 Automation API에서 사용할 수 있다. 이는 `terraform-provider` 같은 파라미터화된 프로바이더나 커스텀 프로바이더 개발 시 일반적이다.

### 로컬 패키지 사용 절차

1. `pulumi package add`로 대상 언어용 SDK 생성
2. 워크스페이스에 플러그인(프로바이더가 아닌 플러그인)을 수동으로 설치
3. Automation API 프로그램에서 생성된 SDK를 참조

> **주의:** 로컬 패키지의 경우 **플러그인(plugin)**을 설치해야 하며, 프로바이더가 아니다. 플러그인은 배포 중 Pulumi 엔진이 리소스와 통신하는 데 사용된다.

### TypeScript 예시

```typescript
import * as automation from "@pulumi/pulumi/automation";
import * as random from "@pulumi/random";

const stack = await automation.LocalWorkspace.createOrSelectStack({
    stackName: "dev",
    projectName: "myProject",
    program: async () => {
        const pet = new random.Pet("my-pet", { length: 2 });
        return { petName: pet.id };
    }
});

// 로컬 패키지의 플러그인 설치
await stack.workspace.installPlugin("terraform-provider", "v1.0.2");

// 배포 실행
const result = await stack.up({ onOutput: console.info });
```

### Python 예시

```python
import pulumi
import pulumi_random as random
from pulumi import automation as auto

def pulumi_program():
    pet = random.Pet("my-pet", length=2)
    pulumi.export("pet_name", pet.id)

stack = auto.create_or_select_stack(
    stack_name="dev",
    project_name="myProject",
    program=pulumi_program
)

# 로컬 패키지의 플러그인 설치
stack.workspace.install_plugin("terraform-provider", "v1.0.2")

# 배포 실행
result = stack.up(on_output=print)
```

---

## 사전 요구 사항

Automation API를 사용하기 전에 다음을 준비해야 한다.

| 요구 사항 | 설명 |
|-----------|------|
| **Pulumi CLI** | 설치하여 `PATH`에서 사용 가능해야 함. 또는 프로그램 내에서 프로그래밍 방식으로 설치 가능 |
| **언어 런타임** | Node.js, Python, Go, .NET, Java 중 선택한 언어의 런타임 설치 |
| **Pulumi 액세스 토큰** | Pulumi Cloud에 상태를 저장하기 위한 토큰. `pulumi login`으로 인증하거나 `PULUMI_ACCESS_TOKEN` 환경변수 설정 |
| **클라우드 자격증명** | 배포 대상 클라우드(AWS 등)의 자격증명 구성 |

---

## Automation API vs pulumi api

Automation API와 `pulumi api` CLI 명령은 서로 다른 목적을 가진다.

| 구분 | Automation API | `pulumi api` |
|------|---------------|-------------|
| **목적** | Pulumi 엔진 자체를 구동 (업데이트, 프리뷰, 리프레시, 디스트로이) | Pulumi Cloud REST API를 직접 호출 |
| **대상** | 인프라 프로비저닝 및 관리 | 스택 메타데이터, 액세스 토큰, Insights 데이터 등 Pulumi Cloud 리소스 |
| **Pulumi 프로그램 실행** | 실행함 | 실행하지 않음 |
| **사용 방식** | SDK (TypeScript, Python, Go, .NET, Java) | CLI 명령 |

---

## 예시 리포지토리

공식 [`automation-api-examples`](https://github.com/pulumi/automation-api-examples) 리포지토리에서 모든 지원 언어의 실행 가능한 예시를 확인할 수 있다.

| 패턴 | 설명 |
|------|------|
| Inline Program | 인라인 프로그램으로 Pulumi 실행 |
| Local Program | 기존 디렉토리의 로컬 프로그램 구동 |
| Cross-Language Program | 다른 언어로 작성된 Pulumi 프로그램을 Automation API로 실행 |
| Pulumi Over HTTP | Pulumi를 HTTP API 뒤에 배치 |
| Database Migration | 데이터베이스 마이그레이션을 포함한 배포 |
| Remote Deployment | RemoteWorkspace를 통한 원격 배포 |
| Multi-Stack Orchestration | 여러 스택을 순차적으로 오케스트레이션 (Go) |
| Pulumi via Jupyter Notebook | Jupyter Notebook에서 Pulumi 실행 (Python) |

---

## 참고

- Automation API는 내부적으로 Pulumi CLI를 구동하므로 런타임에 CLI가 반드시 필요하다.
- 인라인 프로그램의 모든 리소스 생성과 구성은 전달된 함수 범위 내에서 수행되어야 한다.
- Pulumi Cloud 리소스(스택 메타데이터, 토큰 등)를 읽거나 수정하려면 Automation API 대신 `pulumi api`를 사용한다.
- 피드백은 [GitHub 이슈](https://github.com/pulumi/pulumi/issues/new?assignees=&labels=needs-triage&template=bug_report.md&title=)로 제출할 수 있다.
