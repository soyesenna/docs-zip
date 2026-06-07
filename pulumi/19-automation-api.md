# Pulumi Automation API

> **원문**
> - https://www.pulumi.com/docs/iac/concepts/automation-api/
> - https://www.pulumi.com/docs/iac/guides/building-extending/automation-api/

Pulumi Automation API는 Pulumi CLI 없이 Pulumi 프로그램을 프로그래밍 방식으로 실행할 수 있는 인터페이스다. `pulumi up`, `pulumi preview`, `pulumi destroy`, `pulumi stack init` 등 CLI 기능을 강타입(strongly typed) SDK로 캡슐화하여, 셸에서 `pulumi` 명령을 호출하는 대신 자체 애플리케이션 내에서 Pulumi 엔진을 구동할 수 있다. Automation API는 Pulumi SDK 내의 네임스페이스로 배포되며, 오픈소스 제품의 일부다.

> **참고:** Automation API는 내부적으로 Pulumi CLI를 구동하므로 런타임에 CLI가 반드시 필요하다. 사전 설치 후 `PATH`에 추가하거나, 프로그램 내에서 프로그래밍 방식으로 설치할 수 있다.

> **팁:** Automation API는 Pulumi 엔진 자체를 구동하여 업데이트, 프리뷰, 리프레시, 디스트로이를 실행한다. Pulumi 프로그램을 실행하지 않고 Pulumi Cloud 리소스(예: 스택 메타데이터, 액세스 토큰, Insights 데이터)를 읽거나 수정하려면 `pulumi api`를 사용한다.

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

| 언어 | 상태 |
|------|------|
| TypeScript/JavaScript | Stable |
| Python | Stable |
| Go | Stable |
| .NET (C#) | Stable |
| Java | Stable |

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

### Go

```go
import "github.com/pulumi/pulumi/sdk/v3/go/auto"

ctx := context.Background()

// SDK 버전과 일치하는 CLI를 ~/.pulumi/versions/<version>에 설치
pulumiCmd, err := auto.InstallPulumiCommand(ctx, &auto.PulumiCommandOptions{})
if err != nil {
    fmt.Printf("Failed to install the Pulumi CLI: %v\n", err)
    os.Exit(1)
}

// 워크스페이스에 auto.Pulumi 옵션으로 전달
s, err := auto.UpsertStackInlineSource(ctx, stackName, projectName, deployFunc, auto.Pulumi(pulumiCmd))
```

### .NET (C#)

```csharp
using Pulumi.Automation;
using Pulumi.Automation.Commands;

// SDK 버전과 일치하는 CLI를 ~/.pulumi/versions/<version>에 설치
var pulumiCommand = await LocalPulumiCommand.Install();

// 워크스페이스에 PulumiCommand 속성으로 전달
var stackArgs = new InlineProgramArgs(projectName, stackName, program)
{
    PulumiCommand = pulumiCommand,
};
var stack = await LocalWorkspace.CreateOrSelectStackAsync(stackArgs);
```

### Java

> Java SDK는 CLI 프로그래밍 방식 설치를 지원하지 않는다. 수동으로 Pulumi CLI를 설치하여 `PATH`에서 사용 가능하도록 설정해야 한다.

---

## 인라인 프로그램 정의

인라인 프로그램은 일반 Pulumi 프로그램과 동일한 구조를 가지는 함수로 정의한다. 아래 예제는 AWS S3 정적 웹사이트를 생성하는 인라인 프로그램이다.

### TypeScript

```typescript
import * as aws from "@pulumi/aws";
import * as s3 from "@pulumi/aws/s3";

const pulumiProgram = async () => {
    const siteBucket = new s3.Bucket("s3-website-bucket", {});

    const indexContent = `<html><head>
<title>Hello S3</title><meta charset="UTF-8">
</head>
<body>Hello, world!Made with ❤️ with <a href="https://pulumi.com">Pulumi</a>
</body></html>`;

    const ownershipControls = new aws.s3.BucketOwnershipControls("ownership-controls", {
        bucket: siteBucket.id,
        rule: {
            objectOwnership: "ObjectWriter",
        },
    });

    const publicAccessBlock = new aws.s3.BucketPublicAccessBlock("public-access-block", {
        bucket: siteBucket.id,
        blockPublicAcls: false,
    });

    const website = new aws.s3.BucketWebsiteConfigurationV2("website", {
        bucket: siteBucket.id,
        indexDocument: {
            suffix: "index.html",
        },
    });

    const object = new s3.BucketObject("index", {
        bucket: siteBucket.id,
        content: indexContent,
        contentType: "text/html; charset=utf-8",
        key: "index.html",
        acl: "public-read",
    }, {
        dependsOn: [
            publicAccessBlock,
            ownershipControls,
            website,
        ],
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
        <body>
            Hello, world!
            Made with ❤️ with <a href="https://pulumi.com">Pulumi</a>
        </body>
    </html>
    """

    ownership_controls = s3.BucketOwnershipControls("ownership-controls",
        bucket=site_bucket.id,
        rule={
            "object_ownership": "ObjectWriter",
        })

    public_access_block = s3.BucketPublicAccessBlock("public-access-block",
        bucket=site_bucket.id,
        block_public_acls=False)

    website = aws.s3.BucketWebsiteConfigurationV2("website",
        bucket=site_bucket.id,
        index_document={
            "suffix": "index.html",
        })

    s3.BucketObject("index",
        bucket=site_bucket.id,
        content=index_content,
        acl="public-read",
        key="index.html",
        content_type="text/html; charset=utf-8",
        opts=pulumi.ResourceOptions(depends_on=[
            public_access_block,
            ownership_controls,
            website,
        ]))

    pulumi.export("website_url", website.website_endpoint)
```

### Go

```go
deployFunc := func(ctx *pulumi.Context) error {
    siteBucket, err := s3.NewBucket(ctx, "s3-website-bucket", nil)
    if err != nil {
        return err
    }

    indexContent := `<html><head>
<title>Hello S3</title><meta charset="UTF-8">
</head>
<body>Hello, world!Made with ❤️ with <a href="https://pulumi.com">Pulumi</a>
</body></html>`

    ownershipControls, err := s3.NewBucketOwnershipControls(ctx, "ownership-controls", &s3.BucketOwnershipControlsArgs{
        Bucket: siteBucket.ID(),
        Rule: &s3.BucketOwnershipControlsRuleArgs{
            ObjectOwnership: pulumi.String("ObjectWriter"),
        },
    })
    if err != nil {
        return err
    }

    publicAccessBlock, err := s3.NewBucketPublicAccessBlock(ctx, "public-access-block", &s3.BucketPublicAccessBlockArgs{
        Bucket:          siteBucket.ID(),
        BlockPublicAcls: pulumi.Bool(false),
    })
    if err != nil {
        return err
    }

    website, err := s3.NewBucketWebsiteConfigurationV2(ctx, "website", &s3.BucketWebsiteConfigurationV2Args{
        Bucket: siteBucket.ID(),
        IndexDocument: &s3.BucketWebsiteConfigurationV2IndexDocumentArgs{
            Suffix: pulumi.String("index.html"),
        },
    })
    if err != nil {
        return err
    }

    if _, err := s3.NewBucketObject(ctx, "index", &s3.BucketObjectArgs{
        Bucket:      siteBucket.ID(),
        Content:     pulumi.String(indexContent),
        Acl:         pulumi.String("public-read"),
        Key:         pulumi.String("index.html"),
        ContentType: pulumi.String("text/html; charset=utf-8"),
    }, pulumi.DependsOn([]pulumi.Resource{
        publicAccessBlock,
        ownershipControls,
        website,
    })); err != nil {
        return err
    }

    ctx.Export("websiteUrl", website.WebsiteEndpoint)
    return nil
}
```

### .NET (C#)

```csharp
var program = PulumiFn.Create(() =>
{
    var siteBucket = new Pulumi.Aws.S3.Bucket("s3-website-bucket");

    const string indexContent = @"
<html>
    <head><title>Hello S3</title><meta charset=""UTF-8""></head>
    <body>
        Hello, world!
        Made with ❤️ with <a href=""https://pulumi.com"">Pulumi</a>
    </body>
</html>";

    var ownershipControls = new Aws.S3.BucketOwnershipControls("ownership-controls", new()
    {
        Bucket = siteBucket.Id,
        Rule = new Aws.S3.Inputs.BucketOwnershipControlsRuleArgs
        {
            ObjectOwnership = "ObjectWriter",
        },
    });

    var publicAccessBlock = new Aws.S3.BucketPublicAccessBlock("public-access-block", new()
    {
        Bucket = siteBucket.Id,
        BlockPublicAcls = false,
    });

    var website = new Aws.S3.BucketWebsiteConfigurationV2("website", new()
    {
        Bucket = siteBucket.Id,
        IndexDocument = new Aws.S3.Inputs.BucketWebsiteConfigurationV2IndexDocumentArgs
        {
            Suffix = "index.html",
        },
    });

    var @object = new Pulumi.Aws.S3.BucketObject("index", new Pulumi.Aws.S3.BucketObjectArgs
    {
        Bucket = siteBucket.BucketName,
        Content = indexContent,
        Acl = "public-read",
        Key = "index.html",
        ContentType = "text/html; charset=utf-8",
    }, new CustomResourceOptions
    {
        DependsOn =
        {
            publicAccessBlock,
            ownershipControls,
            website,
        },
    });

    return new Dictionary<string, object?>
    {
        ["website_url"] = website.WebsiteEndpoint
    };
});
```

### Java

```java
private static void pulumiProgram(Context ctx) {
    var siteBucket = new Bucket("s3-website-bucket");

    var website = new BucketWebsiteConfigurationV2("website",
        BucketWebsiteConfigurationV2Args.builder()
            .bucket(siteBucket.id())
            .indexDocument(BucketWebsiteConfigurationV2IndexDocumentArgs.builder()
                .suffix("index.html")
                .build())
            .build());

    var ownershipControls = new BucketOwnershipControls("ownershipControls",
        BucketOwnershipControlsArgs.builder()
            .bucket(siteBucket.id())
            .rule(BucketOwnershipControlsRuleArgs.builder()
                .objectOwnership("ObjectWriter")
                .build())
            .build());

    var publicAccessBlock = new BucketPublicAccessBlock("publicAccessBlock",
        BucketPublicAccessBlockArgs.builder()
            .bucket(siteBucket.id())
            .blockPublicAcls(false)
            .build());

    String indexContent = """
            <html>
                <head><title>Hello S3</title><meta charset="UTF-8"></head>
                <body>
                    Hello, world!
                    Made with ❤️ with <a href="https://pulumi.com">Pulumi</a>
                </body>
            </html>
            """;

    var indexHtml = new BucketObject("index.html",
        BucketObjectArgs.builder()
            .bucket(siteBucket.id())
            .content(indexContent)
            .contentType("text/html")
            .acl("public-read")
            .build(),
        CustomResourceOptions.builder()
            .dependsOn(
                publicAccessBlock,
                ownershipControls,
                website)
            .build());

    ctx.export("website_url",
        website.websiteEndpoint().applyValue(
            websiteEndpoint -> String.format("http://%s", websiteEndpoint)));
}
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

### Go

```go
projectName := "inlineS3Project"
stackName := "dev"

s, err := auto.UpsertStackInlineSource(ctx, stackName, projectName, deployFunc)
```

### .NET (C#)

```csharp
var projectName = "inline_s3_project";
var stackName = "dev";

var stackArgs = new InlineProgramArgs(projectName, stackName, program);
var stack = await LocalWorkspace.CreateOrSelectStackAsync(stackArgs);
```

### Java

```java
var projectName = "inline_s3_project_java";
var stackName = "dev";

var stack = LocalWorkspace.createOrSelectStack(projectName, stackName, App::pulumiProgram);
```

`Stack` 객체는 `Workspace` 컨텍스트 내에서 동작한다. `Workspace`는 단일 Pulumi 프로젝트, 프로그램, 여러 스택을 포함하는 실행 컨텍스트로, 플러그인 설치, 환경 설정(`$PULUMI_HOME`), 스택의 생성/삭제/목록 조회 등을 관리한다.

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

### Go

```go
err = w.InstallPlugin(ctx, "aws", "v4.0.0")
if err != nil {
    fmt.Printf("Failed to install program plugins: %v\n", err)
    os.Exit(1)
}

s.SetConfig(ctx, "aws:region", auto.ConfigValue{Value: "us-west-2"})
```

### .NET (C#)

```csharp
await stack.Workspace.InstallPluginAsync("aws", "v4.0.0");
await stack.SetConfigAsync("aws:region", new ConfigValue("us-west-2"));
```

### Java

```java
stack.getWorkspace().installPlugin("aws", "v5.41.0");
stack.setConfig("aws:region", new ConfigValue("us-west-2"));
```

---

## Stack 명령 실행

Stack 객체에 대해 `up`, `preview`, `refresh`, `destroy`, `import`, `export` 등의 명령을 실행할 수 있다.

### up (배포)

#### TypeScript

```typescript
const upRes = await stack.up({ onOutput: console.info });
console.log(`Website URL: ${upRes.outputs.websiteUrl.value}`);
```

#### Python

```python
up_res = stack.up(on_output=print)
print(f"Website URL: {up_res.outputs['website_url'].value}")
```

#### Go

```go
res, err := s.Up(ctx, stdoutStreamer)
if err != nil {
    fmt.Printf("Failed to update stack: %v\n", err)
    os.Exit(1)
}
```

#### .NET (C#)

```csharp
var result = await stack.UpAsync(new UpOptions { OnStandardOutput = Console.WriteLine });
```

#### Java

```java
var result = stack.up(UpOptions.builder()
    .onStandardOutput(System.out::println)
    .build());
```

`up` 결과는 스택 출력값(outputs)과 변경 요약(summary)을 포함한다. 이를 활용해 프로그램 내에서 조건부 로직을 구현할 수 있다. 예를 들어, 업데이트된 리소스가 없을 때 다른 동작을 수행하거나, 출력값을 사용해 동일한 Automation 프로그램 내에서 다른 Pulumi 프로그램을 구동할 수 있다. 표준 출력에 대한 콜백 함수를 선택적으로 지정할 수도 있다.

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

### Go 예시

```go
package main

import (
    "context"
    "fmt"
    "os"

    "github.com/pulumi/pulumi/sdk/v3/go/auto"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
    "github.com/pulumi/pulumi-random/sdk/v4/go/random"
)

func main() {
    ctx := context.Background()

    program := func(ctx *pulumi.Context) error {
        pet, err := random.NewPet(ctx, "my-pet", &random.PetArgs{
            Length: pulumi.Float64(2),
        })
        if err != nil {
            return err
        }
        ctx.Export("petName", pet.ID())
        return nil
    }

    s, err := auto.UpsertStackInlineSource(ctx, "dev", "myProject", program)
    if err != nil {
        fmt.Printf("Failed to create stack: %v\n", err)
        os.Exit(1)
    }

    w := s.Workspace()
    err = w.InstallPlugin(ctx, "terraform-provider", "v1.0.2")
    if err != nil {
        fmt.Printf("Failed to install plugin: %v\n", err)
        os.Exit(1)
    }

    res, err := s.Up(ctx)
    if err != nil {
        fmt.Printf("Failed to update stack: %v\n", err)
        os.Exit(1)
    }
    fmt.Printf("Pet name: %v\n", res.Outputs["petName"].Value)
}
```

### .NET (C#) 예시

```csharp
using System;
using System.Collections.Generic;
using Pulumi.Automation;
using Pulumi.Random;

var program = PulumiFn.Create(() =>
{
    var pet = new RandomPet("my-pet", new RandomPetArgs { Length = 2 });
    return new Dictionary<string, object?>
    {
        ["petName"] = pet.Id
    };
});

var stackArgs = new InlineProgramArgs("myProject", "dev", program);
var stack = await LocalWorkspace.CreateOrSelectStackAsync(stackArgs);

// 로컬 패키지의 플러그인 설치
await stack.Workspace.InstallPluginAsync("terraform-provider", "v1.0.2");

// 배포 실행
var result = await stack.UpAsync(new UpOptions { OnStandardOutput = Console.WriteLine });
```

### Java 예시

```java
import com.pulumi.Context;
import com.pulumi.Pulumi;
import com.pulumi.automation.*;
import com.pulumi.random.RandomPet;
import com.pulumi.random.RandomPetArgs;

public class App {
    public static void main(String[] args) {
        var projectName = "myProject";
        var stackName = "dev";

        var program = (Context ctx) -> {
            var pet = new RandomPet("my-pet",
                RandomPetArgs.builder().length(2).build());
            ctx.export("petName", pet.id());
        };

        try {
            var stack = LocalWorkspace.createOrSelectStack(
                LocalProgramArgs.builder()
                    .stackName(stackName)
                    .projectName(projectName)
                    .program(program)
                    .build());

            // 로컬 패키지의 플러그인 설치
            stack.workspace().installPlugin("terraform-provider", "v1.0.2");

            // 배포 실행
            var result = stack.up(UpOptions.builder()
                .onOutput(System.out::println)
                .build());

            System.out.printf("Pet name: %s%n",
                result.outputs().get("petName").value());
        } catch (Exception e) {
            System.err.println("Error: " + e.getMessage());
        }
    }
}
```

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

## Inline Automation API (LocalWorkspace) 가이드

인라인 프로그램은 별도의 온디스크 패키지나 `Pulumi.yaml` 없이 Automation API 프로그램과 같은 파일에 Pulumi 프로그램을 정의하는 방식이다. `LocalWorkspace`를 사용하면 CLI 기반 워크스페이스와 동일한 동작 방식으로 인라인 프로그램을 실행할 수 있다.

### LocalWorkspace 핵심 워크플로

인라인 프로그램을 `LocalWorkspace`로 실행하는 전체 워크플로는 다음과 같다.

| 단계 | 작업 | 설명 |
|------|------|------|
| 1 | **프로그램 정의** | 인라인 함수로 Pulumi 리소스 정의 |
| 2 | **Stack 연결** | `createOrSelectStack`으로 스택 생성 또는 선택 |
| 3 | **플러그인 설치** | 워크스페이스에 필요한 프로바이더 플러그인 설치 |
| 4 | **설정 구성** | 스택 설정(리전 등) 구성 |
| 5 | **배포 실행** | `stack.up()`으로 배포 |
| 6 | **출력값 활용** | 배포 결과의 출력값을 프로그램 내에서 사용 |

### 전체 인라인 프로그램 예시

다음은 인라인 프로그램을 정의하고, 스택에 연결하고, 플러그인을 설치하고, 배포하는 전체 예시다.

#### TypeScript

```typescript
import * as aws from "@pulumi/aws";
import * as s3 from "@pulumi/aws/s3";
import { LocalWorkspace, InlineProgramArgs } from "@pulumi/pulumi/automation";

// 1. 인라인 프로그램 정의
const pulumiProgram = async () => {
    const siteBucket = new s3.Bucket("s3-website-bucket", {});
    const website = new aws.s3.BucketWebsiteConfigurationV2("website", {
        bucket: siteBucket.id,
        indexDocument: { suffix: "index.html" },
    });
    return { websiteUrl: website.websiteEndpoint };
};

// 2. Stack 연결
const args: InlineProgramArgs = {
    stackName: "dev",
    projectName: "inlineS3Project",
    program: pulumiProgram
};
const stack = await LocalWorkspace.createOrSelectStack(args);

// 3. 플러그인 설치
await stack.workspace.installPlugin("aws", "v4.0.0");

// 4. 설정 구성
await stack.setConfig("aws:region", { value: "us-west-2" });

// 5. 배포 실행
const upRes = await stack.up({ onOutput: console.info });

// 6. 출력값 활용
console.log(`Website URL: ${upRes.outputs.websiteUrl.value}`);
```

#### Python

```python
import pulumi
import pulumi_aws as aws
from pulumi_aws import s3
from pulumi import automation as auto

# 1. 인라인 프로그램 정의
def pulumi_program():
    site_bucket = s3.Bucket("s3-website-bucket")
    website = aws.s3.BucketWebsiteConfigurationV2("website",
        bucket=site_bucket.id,
        index_document={"suffix": "index.html"})
    pulumi.export("website_url", website.website_endpoint)

# 2. Stack 연결
stack = auto.create_or_select_stack(
    stack_name="dev",
    project_name="inline_s3_project",
    program=pulumi_program
)

# 3. 플러그인 설치
stack.workspace.install_plugin("aws", "v4.0.0")

# 4. 설정 구성
stack.set_config("aws:region", auto.ConfigValue(value="us-west-2"))

# 5. 배포 실행
up_res = stack.up(on_output=print)

# 6. 출력값 활용
print(f"Website URL: {up_res.outputs['website_url'].value}")
```

#### Go

```go
package main

import (
    "context"
    "fmt"
    "os"

    "github.com/pulumi/pulumi-aws/sdk/v6/go/aws/s3"
    "github.com/pulumi/pulumi/sdk/v3/go/auto"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    ctx := context.Background()

    // 1. 인라인 프로그램 정의
    deployFunc := func(ctx *pulumi.Context) error {
        siteBucket, err := s3.NewBucket(ctx, "s3-website-bucket", nil)
        if err != nil {
            return err
        }
        website, err := s3.NewBucketWebsiteConfigurationV2(ctx, "website", &s3.BucketWebsiteConfigurationV2Args{
            Bucket: siteBucket.ID(),
            IndexDocument: &s3.BucketWebsiteConfigurationV2IndexDocumentArgs{
                Suffix: pulumi.String("index.html"),
            },
        })
        if err != nil {
            return err
        }
        ctx.Export("websiteUrl", website.WebsiteEndpoint)
        return nil
    }

    // 2. Stack 연결
    s, err := auto.UpsertStackInlineSource(ctx, "dev", "inlineS3Project", deployFunc)
    if err != nil {
        os.Exit(1)
    }

    // 3. 플러그인 설치
    w := s.Workspace()
    err = w.InstallPlugin(ctx, "aws", "v4.0.0")
    if err != nil {
        os.Exit(1)
    }

    // 4. 설정 구성
    s.SetConfig(ctx, "aws:region", auto.ConfigValue{Value: "us-west-2"})

    // 5. 배포 실행
    res, err := s.Up(ctx)
    if err != nil {
        os.Exit(1)
    }

    // 6. 출력값 활용
    fmt.Printf("Website URL: %v\n", res.Outputs["websiteUrl"].Value)
}
```

#### .NET (C#)

```csharp
using System;
using System.Collections.Generic;
using Pulumi;
using Pulumi.Automation;
using Aws = Pulumi.Aws;

// 1. 인라인 프로그램 정의
var program = PulumiFn.Create(() =>
{
    var siteBucket = new Aws.S3.Bucket("s3-website-bucket");
    var website = new Aws.S3.BucketWebsiteConfigurationV2("website", new()
    {
        Bucket = siteBucket.Id,
        IndexDocument = new Aws.S3.Inputs.BucketWebsiteConfigurationV2IndexDocumentArgs
        {
            Suffix = "index.html",
        },
    });
    return new Dictionary<string, object?>
    {
        ["website_url"] = website.WebsiteEndpoint
    };
});

// 2. Stack 연결
var stackArgs = new InlineProgramArgs("inlineS3Project", "dev", program);
var stack = await LocalWorkspace.CreateOrSelectStackAsync(stackArgs);

// 3. 플러그인 설치
await stack.Workspace.InstallPluginAsync("aws", "v4.0.0");

// 4. 설정 구성
await stack.SetConfigAsync("aws:region", new ConfigValue("us-west-2"));

// 5. 배포 실행
var result = await stack.UpAsync(new UpOptions { OnStandardOutput = Console.WriteLine });

// 6. 출력값 활용
Console.WriteLine($"Website URL: {result.Outputs["website_url"].Value}");
```

#### Java

```java
import com.pulumi.Context;
import com.pulumi.automation.*;
import com.pulumi.aws.s3.Bucket;
import com.pulumi.aws.s3.BucketWebsiteConfigurationV2;
import com.pulumi.aws.s3.BucketWebsiteConfigurationV2Args;
import com.pulumi.aws.s3.inputs.BucketWebsiteConfigurationV2IndexDocumentArgs;

public class App {
    public static void main(String[] args) {
        var projectName = "inlineS3ProjectJava";
        var stackName = "dev";

        // 1. 인라인 프로그램 정의 & 2. Stack 연결
        var stack = LocalWorkspace.createOrSelectStack(projectName, stackName, ctx -> {
            var siteBucket = new Bucket("s3-website-bucket");
            var website = new BucketWebsiteConfigurationV2("website",
                BucketWebsiteConfigurationV2Args.builder()
                    .bucket(siteBucket.id())
                    .indexDocument(BucketWebsiteConfigurationV2IndexDocumentArgs.builder()
                        .suffix("index.html")
                        .build())
                    .build());
            ctx.export("website_url", website.websiteEndpoint());
        });

        // 3. 플러그인 설치
        stack.workspace().installPlugin("aws", "v5.41.0");

        // 4. 설정 구성
        stack.setConfig("aws:region", new ConfigValue("us-west-2"));

        // 5. 배포 실행
        var result = stack.up(UpOptions.builder()
            .onStandardOutput(System.out::println)
            .build());

        // 6. 출력값 활용
        System.out.printf("Website URL: %s%n",
            result.outputs().get("website_url").value());
    }
}
```

---

## 커스텀 배포 도구 구축 패턴

Automation API를 활용하면 Pulumi를 기반으로 한 고수준 커스텀 배포 도구를 구축할 수 있다. 대표적인 패턴은 다음과 같다.

### Pulumi Over HTTP

Pulumi를 REST API 뒤에 배치하여, HTTP 요청으로 인프라를 프로비저닝하는 패턴이다. 멀티 테넌트 SaaS 플랫폼, 셀프 서비스 인프라 포털 등에 활용된다.

| 구성 요소 | 설명 |
|-----------|------|
| **HTTP 핸들러** | 요청을 수신하여 스택 이름, 설정, 프로그램 매개변수를 추출 |
| **Automation API 스택** | 요청별로 동적으로 `LocalWorkspace.createOrSelectStack` 호출 |
| **출력값 반환** | `stack.up()` 결과의 출력값을 HTTP 응답으로 직렬화 |

### 멀티 스택 오케스트레이션

여러 스택을 순차적 또는 병렬로 배포하는 패턴이다. 마이크로서비스 아키텍처, 멀티 리전 배포 등에 활용된다.

| 구성 요소 | 설명 |
|-----------|------|
| **스택 순서 정의** | 의존 관계에 따라 스택 배포 순서를 정의 |
| **출력값 전달** | 선행 스택의 출력값을 후행 스택의 설정으로 전달 |
| **에러 처리** | 스택 배포 실패 시 롤백 또는 알림 처리 |

### 데이터베이스 마이그레이션 포함 배포

인프라 배포와 함께 애플리케이션 수준의 마이그레이션(예: 데이터베이스 스키마 변경)을 실행하는 패턴이다.

| 구성 요소 | 설명 |
|-----------|------|
| **인라인 프로그램** | Pulumi 리소스 생성과 함께 마이그레이션 로직을 인라인 함수에 포함 |
| **dependsOn** | 마이그레이션이 해당 리소스 생성 후 실행되도록 보장 |
| **출력값** | 마이그레이션 결과를 출력값으로 노출 |

### 크로스 언어 프로그램

Automation API 프로그램과 관리 대상 Pulumi 프로그램을 다른 언어로 작성하는 패턴이다. 예를 들어 Go로 오케스트레이터를 작성하고 Python으로 Pulumi 프로그램을 작성할 수 있다.

| 구성 요소 | 설명 |
|-----------|------|
| **오케스트레이터** | 한 언어로 Automation API를 사용해 워크스페이스를 제어 |
| **로컬 프로그램** | 다른 언어로 작성된 Pulumi 프로그램 디렉토리를 `LocalProgramArgs`로 참조 |
| **크로스 언어 지원** | `LocalWorkspace`가 모든 Pulumi 지원 언어의 프로그램을 구동 |

---

## 예시 리포지토리

공식 [`automation-api-examples`](https://github.com/pulumi/automation-api-examples) 리포지토리에서 모든 지원 언어의 실행 가능한 예시를 확인할 수 있다.

| 패턴 | TypeScript | Python | Go | .NET | Java |
|------|-----------|--------|-----|------|------|
| **Inline Program** | [inlineProgram-tsnode](https://github.com/pulumi/automation-api-examples/blob/main/nodejs/inlineProgram-tsnode) | [inline_program](https://github.com/pulumi/automation-api-examples/blob/main/python/inline_program) | [inline_program](https://github.com/pulumi/automation-api-examples/blob/main/go/inline_program) | [InlineProgram](https://github.com/pulumi/automation-api-examples/blob/main/dotnet/InlineProgram) | [inlineProgram](https://github.com/pulumi/automation-api-examples/blob/main/java/inlineProgram) |
| **Local Program** | [localProgram-tsnode](https://github.com/pulumi/automation-api-examples/blob/main/nodejs/localProgram-tsnode) | [local_program](https://github.com/pulumi/automation-api-examples/blob/main/python/local_program) | [local_program](https://github.com/pulumi/automation-api-examples/blob/main/go/local_program) | [LocalProgram](https://github.com/pulumi/automation-api-examples/blob/main/dotnet/LocalProgram) | [localProgram](https://github.com/pulumi/automation-api-examples/blob/main/java/localProgram) |
| **Cross-Language** | [crossLanguage-tsnode](https://github.com/pulumi/automation-api-examples/blob/main/nodejs/crossLanguage-tsnode) | [cross_language](https://github.com/pulumi/automation-api-examples/blob/main/python/cross_language) | -- | [CrossLanguage](https://github.com/pulumi/automation-api-examples/blob/main/dotnet/CrossLanguage) | -- |
| **Pulumi Over HTTP** | [pulumiOverHttp-ts](https://github.com/pulumi/automation-api-examples/blob/main/nodejs/pulumiOverHttp-ts) | [pulumi_over_http](https://github.com/pulumi/automation-api-examples/blob/main/python/pulumi_over_http) | [pulumi_over_http](https://github.com/pulumi/automation-api-examples/blob/main/go/pulumi_over_http) | -- | -- |
| **Database Migration** | [databaseMigration-ts](https://github.com/pulumi/automation-api-examples/blob/main/nodejs/databaseMigration-ts) | [database_migration](https://github.com/pulumi/automation-api-examples/blob/main/python/database_migration) | [database_migration](https://github.com/pulumi/automation-api-examples/blob/main/go/database_migration) | [DatabaseMigration](https://github.com/pulumi/automation-api-examples/blob/main/dotnet/DatabaseMigration) | [databaseMigration](https://github.com/pulumi/automation-api-examples/blob/main/java/databaseMigration) |
| **Remote Deployment** | [remoteDeployment-tsnode](https://github.com/pulumi/automation-api-examples/blob/main/nodejs/remoteDeployment-tsnode) | [remote_deployment](https://github.com/pulumi/automation-api-examples/blob/main/python/remote_deployment) | [remote_deployment](https://github.com/pulumi/automation-api-examples/blob/main/go/remote_deployment) | [RemoteDeployment](https://github.com/pulumi/automation-api-examples/blob/main/dotnet/RemoteDeployment) | -- |
| **Inline/Local Hybrid** | -- | -- | [inline_local_hybrid](https://github.com/pulumi/automation-api-examples/blob/main/go/inline_local_hybrid) | -- | -- |
| **Multi-Stack Orchestration** | -- | -- | [multi_stack_orchestration](https://github.com/pulumi/automation-api-examples/blob/main/go/multi_stack_orchestration) | -- | -- |
| **Git Repo Program** | -- | -- | [git_repo_program](https://github.com/pulumi/automation-api-examples/blob/main/go/git_repo_program) | -- | -- |
| **Jupyter Notebook** | -- | [pulumi_via_jupyter](https://github.com/pulumi/automation-api-examples/blob/main/python/pulumi_via_jupyter) | -- | -- | -- |

---

## 참고

- Automation API는 내부적으로 Pulumi CLI를 구동하므로 런타임에 CLI가 반드시 필요하다.
- 인라인 프로그램의 모든 리소스 생성과 구성은 전달된 함수 범위 내에서 수행되어야 한다.
- Pulumi Cloud 리소스(스택 메타데이터, 토큰 등)를 읽거나 수정하려면 Automation API 대신 `pulumi api`를 사용한다.
- 피드백은 [GitHub 이슈](https://github.com/pulumi/pulumi/issues/new?assignees=&labels=needs-triage&template=bug_report.md&title=)로 제출할 수 있다.
