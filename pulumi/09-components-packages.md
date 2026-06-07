# Pulumi Components, Packages, Plugins & Converters

> **원문**
> - https://www.pulumi.com/docs/iac/concepts/components/
> - https://www.pulumi.com/docs/iac/concepts/packages/
> - https://www.pulumi.com/docs/iac/concepts/plugins/
> - https://www.pulumi.com/docs/iac/concepts/converters/
> - https://www.pulumi.com/docs/iac/concepts/stash/

Pulumi는 인프라를 코드로 관리하기 위한 확장성 높은 아키텍처를 제공한다. 핵심 개념으로 **Component Resource**(여러 리소스를 하나로 패키징), **Pulumi Packages**(한 번 작성하면 모든 언어에서 사용 가능), **Plugins**(언어/프로바이더/변환기 등의 확장 메커니즘), **Converters**(Terraform/ARM/Bicep/Kubernetes YAML을 Pulumi 코드로 변환), **Stash**(임시 값을 스택 상태에 저장)가 있다. 이 문서에서는 각 개념의 원리, 사용법, 배포 방식을 정리한다.

---

## Component Resource

### 개요

Component Resource는 여러 Pulumi 리소스를 논리적으로 그룹화하여 단일 Pulumi 리소스로 노출하는 기능이다. 관련 리소스와 설정을 캡슐화하여, 소비자가 복잡한 인프라를 간단한 인터페이스로 생성할 수 있게 한다. 구현 세부사항을 알 필요가 없다.

Component는 코드가 존재하는 어디에나 정의할 수 있다:
- Pulumi 프로그램 내에 인라인으로 정의
- 언어 생태계 라이브러리로 공유
- Pulumi Package의 일부로 배포 (모든 Pulumi 언어에서 소비 가능)

> Component는 패키지의 일부일 필요가 없다. 패키지 없이도 로컬에서 정의·사용할 수 있다.

플랫폼 팀은 Component를 사용하여 인프라 모범 사례, 보안 정책, 규정 준수 요구 사항을 재사용 가능한 빌딩 블록으로 정형화할 수 있다. [Pulumi IDP Private Registry](https://www.pulumi.com/docs/idp/concepts/private-registry/)에 게시하면 조직 전체에서 패키지가 검색 가능해지고, 기반 구현을 이해할 필요 없이 모든 팀이 소비할 수 있다.

Component는 Terraform 모듈, AWS CDK Construct와 유사한 개념이다. Pulumi는 Terraform 모듈과 CDK Construct를 Pulumi 프로그램에서 직접 사용하는 것도 지원한다.

### Component 예시 (AWSx 패키지)

| Component | 설명 |
|-----------|------|
| `awsx.ec2.Vpc` | 서브넷, 라우트 테이블, 게이트웨이가 AWS 모범 사례에 맞게 사전 구성된 완전한 VPC 생성 |
| `awsx.ecs.FargateService` | 로드 밸런서와 필요한 네트워킹이 포함된 ECS 서비스 생성 |
| `awsx.ecr.Repository` | 이미지 스캐닝과 라이프사이클 정책이 포함된 ECR 리포지토리 생성 |

### Component 소비 방식

| 배포 형태 | 설치 방법 | 비고 |
|-----------|----------|------|
| **로컬 Component** | 언어의 표준 import 메커니즘 사용 | 추가 설치 불필요 |
| **네이티브 언어 패키지** | `npm install`, `pip install` 등 언어 패키지 매니저 사용 | 작성된 언어에서만 사용 가능 |
| **Pulumi Package (SDK 미사전발행)** | `pulumi package add` 명령으로 즉석 SDK 생성 | 모든 언어에서 사용 가능, 런타임 요구사항 존재 |
| **Pulumi Package (SDK 사전발행)** | 언어 패키지 매니저로 직접 설치 | 추가 런타임 불필요, 사전 컴파일된 SDK |

> **Pulumi IDP Private Registry**는 조직의 Pulumi Cloud 조직에서 사용 중인 모든 패키지를 검색할 수 있는 갤러리를 제공하며, Component가 포함된 패키지도 포함된다.

### pulumi package add 사용법

SDK가 사전 발행되지 않은 Pulumi Package를 소비할 때 사용한다. Pulumi가 스키마에서 SDK를 즉석 생성한다.

```bash
# Git 리포지토리에서 패키지 추가
pulumi package add github.com/my-org/my-component@v1.0.0

# 로컬 디렉토리에서 패키지 추가 (모노레포, 빠른 반복에 유용)
pulumi package add /path/to/local/secure-s3-component
```

이 패턴은 조직이 Git 리포지토리나 Pulumi IDP Private Registry를 통해 내부 소비용으로 게시한 Component에 흔히 사용된다. source-based plugin 패키지의 Component를 여러 언어에서 소비하는 방식이기도 하다. SDK는 Component가 작성된 언어와 관계없이 프로그램의 언어로 생성된다.

> **런타임 요구사항**: `pulumi package add`는 라이브 플러그인에서 SDK를 생성하므로, 플러그인이 실행 가능해야 한다. 각 언어 SDK 문서에서 요구하는 툴체인이 필요하다.

### Component 소비 예제

`awsx.ec2.Vpc` Component는 Component Resource의 좋은 예시이다. 서브넷, 라우트 테이블, 인터넷 게이트웨이를 포함한 완전한 VPC를 간단한 인터페이스 뒤에 생성한다.

**TypeScript:**

```typescript
import * as awsx from "@pulumi/awsx";

const vpc = new awsx.ec2.Vpc("vpc", {
    subnetSpecs: [
        { type: awsx.ec2.SubnetType.Public, cidrMask: 22 },
        { type: awsx.ec2.SubnetType.Private, cidrMask: 20 },
    ],
}, { protect: true });

export const vpcId = vpc.vpcId;
export const privateSubnetIds = vpc.privateSubnetIds;
export const publicSubnetIds = vpc.publicSubnetIds;
```

**Python:**

```python
import pulumi
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

pulumi.export("vpcId", vpc.vpc_id)
pulumi.export("privateSubnetIds", vpc.private_subnet_ids)
pulumi.export("publicSubnetIds", vpc.public_subnet_ids)
```

**Go:**

```go
package main

import (
    "github.com/pulumi/pulumi-awsx/sdk/v3/go/awsx/ec2"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        vpc, err := ec2.NewVpc(ctx, "vpc", &ec2.VpcArgs{
            SubnetSpecs: []ec2.SubnetSpecArgs{
                {Type: ec2.SubnetTypePublic, CidrMask: pulumi.IntRef(22)},
                {Type: ec2.SubnetTypePrivate, CidrMask: pulumi.IntRef(20)},
            },
        }, pulumi.Protect(true))
        if err != nil {
            return err
        }

        ctx.Export("vpcId", vpc.VpcId)
        ctx.Export("privateSubnetIds", vpc.PrivateSubnetIds)
        ctx.Export("publicSubnetIds", vpc.PublicSubnetIds)
        return nil
    })
}
```

**C#:**

```csharp
using Pulumi;
using System.Collections.Generic;
using Pulumi.Awsx.Ec2.Inputs;
using Ec2 = Pulumi.Awsx.Ec2;

return await Deployment.RunAsync(() =>
{
    var vpc = new Ec2.Vpc("vpc", new()
    {
        SubnetSpecs =
        {
            new SubnetSpecArgs { Type = Ec2.SubnetType.Public, CidrMask = 22 },
            new SubnetSpecArgs { Type = Ec2.SubnetType.Private, CidrMask = 20 },
        },
    }, new ComponentResourceOptions { Protect = true });

    return new Dictionary<string, object?>
    {
        ["vpcId"] = vpc.VpcId,
        ["privateSubnetIds"] = vpc.PrivateSubnetIds,
        ["publicSubnetIds"] = vpc.PublicSubnetIds,
    };
});
```

**Java:**

```java
package myproject;

import java.util.Arrays;
import com.pulumi.Pulumi;
import com.pulumi.awsx.ec2.Vpc;
import com.pulumi.awsx.ec2.VpcArgs;
import com.pulumi.awsx.ec2.enums.SubnetType;
import com.pulumi.awsx.ec2.inputs.SubnetSpecArgs;
import com.pulumi.resources.ComponentResourceOptions;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {
            var vpc = new Vpc("vpc",
                VpcArgs.builder()
                    .subnetSpecs(Arrays.asList(
                        SubnetSpecArgs.builder().type(SubnetType.Public).cidrMask(22).build(),
                        SubnetSpecArgs.builder().type(SubnetType.Private).cidrMask(20).build()
                    ))
                    .build(),
                ComponentResourceOptions.builder().protect(true).build());

            ctx.export("vpcId", vpc.vpcId());
            ctx.export("privateSubnetIds", vpc.privateSubnetIds());
            ctx.export("publicSubnetIds", vpc.publicSubnetIds());
        });
    }
}
```

Component는 다른 Pulumi 리소스와 동일하게 이름, Component 작성자가 정의한 인수, `protect`와 같은 Resource Options를 전달하여 인스턴스화한다.

### Components in `pulumi up` output

`pulumi up`을 실행하면 Component는 CLI 출력에 리소스 트리로 나타난다. 하위 리소스는 부모 Component 아래에 중첩되어 표시되어, Component가 대신 생성한 모든 것을 확인할 수 있다.

```output
Updating (dev):
     Type                            Name              Status
 +   pulumi:pulumi:Stack             my-stack          created
 +   └─ awsx:ec2:Vpc                 vpc               created
 +      └─ aws:ec2:Vpc               vpc               created
 +         ├─ aws:ec2:Subnet         vpc-private-1     created
 +         │  └─ aws:ec2:RouteTable  vpc-private-1     created
 +         │     ├─ aws:ec2:Route    vpc-private-1     created
 +         │     └─ aws:ec2:RouteTableAssociation  vpc-private-1  created
 +         └─ aws:ec2:Subnet         vpc-public-1      created
 +            └─ aws:ec2:RouteTable  vpc-public-1      created
 +               ├─ aws:ec2:Route    vpc-public-1      created
 +               └─ aws:ec2:RouteTableAssociation  vpc-public-1   created
```

이 트리는 단일 `awsx:ec2:Vpc` Component가 개별적으로 정의하고 관리해야 하는 여러 AWS 리소스(서브넷, 라우트 테이블, 라우트, 연결)를 캡슐화하고 있음을 보여준다.

### Resource options and component resources

Component Resource에 전달되는 Resource Options는 Custom Resource와 항상 동일하게 작동하지 않는다. 예를 들어 `provider` 옵션은 Component에 아무 효과가 없으며, 대신 `providers`를 사용하여 Component의 하위 리소스에 명시적 Provider 구성을 전달해야 한다. Component Resource에 적용되는 옵션의 전체 목록은 Resource options 문서를 참조하라.

### Authoring components

Component는 `ComponentResource` 클래스를 확장하여 작성한다. 다음 가이드에서 빌드, 패키징, 테스트 방법을 다룬다:

| 가이드 | 설명 |
|--------|------|
| **Build a Component** | 클래스 정의, 인수 구조화, 하위 리소스 생성, 출력 등록, Provider 상속 구성 |
| **Packaging Components** | 세 가지 배포 옵션 비교 및 공유용 Component 패키징 |
| **Testing Components** | Component Resource 테스트 작성 |
| **Pulumi IDP Private Registry** | 조직 내 Component 게시 및 검색 |

---

## Pulumi Packages

### 패키지 구조

Pulumi Package는 **한 번 작성하면 모든 Pulumi 언어에서 사용**할 수 있게 하는 핵심 기술이다. 패키지는 두 부분으로 구성된다:

| 구성 요소 | 설명 |
|-----------|------|
| **Provider Plugin** | Pulumi 코드가 포함된 실행 파일. 모든 Pulumi 지원 언어로 작성 가능. Custom Resource(CRUD 연산), Function(클라우드 조회), Component(재사용 추상화)를 포함 |
| **SDK** | Provider의 스키마 파일에서 생성된 언어별 SDK. npm/PyPI 등에 사전 발행되거나, `pulumi package add`로 로컬 생성 가능 |

Provider Plugin은 다음을 포함한다:
- **Custom Resource**: 인프라 리소스에 대한 CRUD 연산 정의
- **Function**: 클라우드 프로바이더에서 리소스 데이터 조회
- **Component**: Custom Resource나 다른 Component를 재사용 가능한 추상화로 캡슐화

SDK는 Provider의 스키마 파일에서 생성된다. 패키지 피드(npm, PyPI 등)에 게시·호스팅되거나, 패키지를 Pulumi 프로그램에 추가할 때 Pulumi CLI가 패키지 스키마와 조합하여 로컬에서 생성할 수 있다.

### 패키지 설치 방법

| 방식 | 명령 | 설명 |
|------|------|------|
| **사전 발행 SDK** | `npm install @pulumi/aws` / `pip install pulumi-aws` 등 | 대부분의 Pulumi Registry 패키지. 언어 패키지 매니저 사용 |
| **로컬 패키지** | `pulumi package add <source>` | SDK를 로컬에서 생성. `Pulumi.yaml`에 등록됨 |
| **전체 설치** | `pulumi install` | `package.json`, `requirements.txt` 등의 의존성과 `Pulumi.yaml`의 로컬 패키지를 모두 설치 |

`pulumi install`은 표준 패키지 매니저 의존성과 `Pulumi.yaml`에 정의된 로컬 패키지를 단일 명령으로 처리하므로 권장된다. 다음 상황에서 실행하라:
- 소스 컨트롤에서 클론 후 프로젝트 설정 시
- 패키지 추가 또는 업데이트 시
- 모든 팀원이 동일한 의존성을 갖도록 보장 시

### 로컬 패키지 추가 예제

```bash
# Terraform Provider 추가
pulumi package add terraform-provider hashicorp/random

# Git 리포지토리에서 Component 추가
pulumi package add example.com/org/repo.git/path@version

# 특정 버전의 Terraform Provider 추가
pulumi package add terraform-provider hashicorp/random 3.7.1
```

로컬 패키지 추가 후 `pulumi install`을 실행하여 설치를 완료하라.

> `pulumi package add` 실행 후 `Pulumi.yaml` 파일이 업데이트된다. 이 파일을 소스 컨트롤에 커밋하면 팀원들이 `pulumi install`로 동일한 패키지를 설치할 수 있다.

로컬 패키지의 일반적인 사용 사례:

| 사용 사례 | 설명 |
|-----------|------|
| **Any Terraform Provider** | Terraform Provider에 대한 로컬 SDK 생성. 모든 Terraform Provider를 Pulumi 프로그램에서 소비 가능 |
| **Azure Native Provider** | 특정 버전의 Azure API에 대한 로컬 SDK 생성 |
| **Pulumi IDP Component** | Pulumi IDP에 게시된 Component 또는 Git 참조를 통한 직접 소비 |

### 런타임 요구사항

패키지의 작성 언어에 따라 런타임 요구사항이 다르다:

| 패키지 작성 언어 | 런타임 요구사항 |
|------------------|----------------|
| TypeScript | NodeJS 런타임 필요 |
| Python | Python 인터프리터 필요 |
| Go | 컴파일된 경우 불필요. 소스 참조 시(예: Pulumi IDP에 게시된 Component) 호환되는 Go 언어 버전 필요 |
| .NET | 런타임 포함 바이너리(권장)인 경우 불필요. 런타임 종속 바이너리는 런타임 필요 |
| Java | JVM 런타임 필요 |
| YAML | 특정 런타임 불필요 |

> Pulumi Registry의 패키지는 대부분 Go로 작성되어 컴파일되므로 런타임이 필요 없다. 주요 클라우드/SaaS 프로바이더 패키지가 모두 여기에 해당한다.

### 패키지 업그레이드

| 패키지 유형 | 업그레이드 방법 |
|-------------|----------------|
| **사전 발행 SDK** | `npm install @pulumi/aws@latest`, `pip install --upgrade pulumi-aws` 등 언어 패키지 매니저 사용 |
| **로컬 패키지** | `pulumi package add`를 원하는 버전으로 재실행. `Pulumi.yaml`의 `packages` 섹션에서 현재 버전 확인 가능 |

로컬 패키지 업그레이드 후 `pulumi install`을 실행하여 업데이트된 의존성을 설치하라.

### The Pulumi Registry

Pulumi Registry는 인기 있는 Pulumi 패키지 목록을 제공한다. 각 패키지의 Installation & Configuration 페이지에 Provider SDK 설치 방법이 언어별로 안내되어 있다. 대부분의 패키지는 사전 발행된 SDK가 있으며, 사전 발행 SDK가 없는 패키지는 메인 페이지에 SDK 생성 방법이 표시된다.

### 패키지 저작

패키지 저작의 두 가지 일반적인 시나리오:

1. **Component 배포용**: 팀/조직 내 또는 커뮤니티 공유용 Component 작성
2. **Provider 저작**: 클라우드/SaaS 프로바이더의 리소스 관리용 Provider 작성

#### Component 배포 시 로컬 패키지 vs 사전 발행 SDK

**대부분의 Component 작성자는 소비자가 로컬 패키지를 사용하도록 권장해야 한다.** 이유는 다음과 같다:

| 항목 | 로컬 패키지 | 사전 발행 SDK |
|------|------------|--------------|
| 복잡도 | 낮음 | 높음 (모든 언어 SDK 생성 + 패키지 피드 호스팅 필요) |
| 다중 언어 지원 | `pulumi package add`로 자동 생성 | CI/CD에서 각 언어별 SDK 빌드/발행 필요 |
| Schema 요구사항 | 불필요 | Pulumi Provider SDK(Go)로 스키마 생성 필요 |
| 적합한 경우 | 대부분의 Component | 보안 정책으로 소프트웨어 설치 제한 시, Pulumi Registry 공개 배포 시 |

사전 발행 SDK가 더 나은 경우:
- 조직에 개발자 기기에 소프트웨어 설치를 제한하는 보안 정책이 있는 경우
- Pulumi Registry에 공개 배포할 의도가 있는 경우
- 팀이 Go 코드 작성 및 유지보수에 익숙한 경우
- 조직에 모든 언어의 SDK를 호스팅할 패키지 피드가 이미 있는 경우

#### Pulumi Provider 저작

새로운 클라우드 또는 SaaS 프로바이더의 리소스를 관리하는 Pulumi Provider를 저작하려면, Pulumi Provider SDK(Go)를 사용하여 Go로 작성해야 한다.

---

## Plugins

### 개요

Plugin은 Pulumi의 핵심 확장 메커니즘이다. Pulumi 엔진이 다양한 언어, 리소스 프로바이더, 기타 도구와 통일된 방식으로 통신할 수 있게 한다. Plugin은 항상 별도 프로세스로 실행되며, 대부분 gRPC로 Pulumi 엔진과 통신한다.

> 대부분의 사용자는 Plugin을 깊이 이해할 필요가 없다. Plugin 설치와 관리는 자동으로 처리된다. Pulumi 내부에 대해 더 알고 싶다면 Pulumi Developer Documentation을 참조하라.

### Plugin 유형

| 유형 | 설명 | 설치 방식 |
|------|------|----------|
| **Resource Plugin (Provider)** | 클라우드 리소스 관리를 위한 표준화된 인터페이스. Pulumi Package로 배포. 자체 Component를 Resource Plugin으로 배포하여 모든 언어에서 소비 가능. Pulumi IDP에 게시하여 조직 내 검색 가능 | `pulumi up` 시 자동 설치 |
| **Language Plugin (Language Host)** | 특정 언어로 작성된 프로그램을 호스트. 언어 실행기(`pulumi-language-<name>`) + 언어 SDK로 구성 | Pulumi CLI와 함께 자동 설치 |
| **Analyzer Plugin** | Pulumi 프로그램을 스캔하여 잠재적 이슈 검사. Policy as Code 구동. 규정 준수, 보안, 모범 사례 강제 | Pulumi CLI와 함께 자동 설치 |
| **Converter Plugin** | Terraform, ARM, Bicep, Kubernetes YAML 등을 Pulumi 프로그램으로 변환 | `pulumi convert` 실행 시 자동 설치 |
| **Tool Plugin** | Pulumi와 외부 도구의 통합 지원. 기존 워크플로 및 툴체인과의 연동 확장 | - |

### Plugin 배포 방식

| 방식 | 설명 |
|------|------|
| **실행 파일** | `pulumi-<kind>-<name>` 네이밍 컨벤션 (예: `pulumi-resource-aws`) |
| **소스 기반** | `PulumiPlugin.yaml` 설정 파일 포함. 엔진이 지정된 런타임을 통해 언어 플러그인 인터페이스로 실행 |

### Plugin 설치 및 관리

**자동 설치:**
- Resource Plugin: `pulumi preview` 또는 `pulumi up` 최초 실행 시 플러그인 캐시에 없으면 자동 설치
- Language Plugin: Pulumi CLI와 함께 설치
- Policy Plugin: Pulumi CLI와 함께 설치
- Converter Plugin: `pulumi convert` 실행 시 자동 설치
- Pulumi CLI는 프로그램 실행 시 Resource Plugin이 존재하는지 확인

**수동 설치 (특수 상황에서 유용):**
- CI/CD 파이프라인 속도 향상을 위한 커스텀 Docker 이미지에 사전 로드
- 폐쇄망(air-gapped) 환경
- 프리릴리스 버전 테스트
- 커뮤니티 유지보수 Plugin 설치

### Plugin 저장 위치

| 위치 | 대상 |
|------|------|
| `~/.pulumi/bin` | Pulumi CLI에 포함된 모든 Plugin (Language Plugin, Policy Plugin 등) |
| `~/.pulumi/plugins` | 사용자가 설치한 Plugin (자동/수동 모두) |

### Plugin 관리 명령어

| 명령어 | 설명 |
|--------|------|
| `pulumi plugin ls` | 설치된 Plugin 나열 |
| `pulumi plugin rm` | 캐시된 Plugin 제거 |
| `pulumi plugin install` | 수동으로 Plugin 설치 |

### PulumiPlugin.yaml 참조

소스 기반 Plugin이 사용하는 설정 파일이다. 파일명은 대소문자 구분이 있으며 정확히 `PulumiPlugin.yaml` 또는 `PulumiPlugin.yml`이어야 한다. 커스텀 Component나 Provider를 Plugin으로 배포할 때 사용한다. 표준 프로그램을 사용하는 대부분의 사용자는 이 파일을 생성하거나 수정할 필요가 없다.

**속성:**

| 속성 | 필수 | 설명 | 옵션 |
|------|------|------|------|
| `runtime` | 필수 | Plugin 실행에 사용할 언어 런타임: `nodejs`, `python`, `go`, `dotnet`, `java`, `yaml`, `bun` | runtime options 참조 |
| `packages` | 선택 | Plugin에서 사용할 추가 패키지. `Pulumi.yaml`의 `packages`와 동일 | - |
| `requiredPulumiVersion` | 선택 | 필요한 Pulumi CLI 버전 범위. `Pulumi.yaml`의 `requiredPulumiVersion`과 동일 | - |

**런타임 옵션:**

단순 문자열 또는 옵션 객체로 지정 가능:

```yaml
# 단순 지정
runtime: nodejs

# 옵션 포함 지정
runtime:
  name: nodejs
  options:
    packagemanager: yarn
```

> `Pulumi.yaml`의 모든 런타임 옵션이 Plugin에서 지원되는 것은 아니다.

**언어별 런타임 옵션:**

| 런타임 | 옵션 | 설명 |
|--------|------|------|
| **Node.js** | `nodeargs` | Node 실행 시 전달할 인자 |
| | `packagemanager` | 패키지 매니저: `npm`(기본), `pnpm`, `yarn`, `bun` |
| **Python** | `toolchain` | 가상환경 관리 툴체인: `pip`(기본), `poetry`, `uv` |
| | `virtualenv` | 가상환경 경로 (`pip` 또는 `uv` 툴체인에만 적용) |
| **Go** | - | 런타임 옵션 미지원 |
| **.NET** | `binary` | 사전 빌드된 실행 파일 경로 |
| | `use-executor` | dotnet 바이너리 경로 오버라이드 (예: `dotnet.exe`, `/opt/homebrew/bin/dotnet`) |
| **Java** | `binary` | 사전 빌드된 실행 파일 경로 |
| | `use-executor` | Java 실행 방식 오버라이드. `gradle`, `mvn` 등 설정 가능 |
| **Bun** | - | 런타임 옵션 미지원 |
| **YAML** | - | 런타임 옵션 미지원 |

**PulumiPlugin.yaml 예제:**

```yaml
# Python + uv 툴체인
runtime:
  name: python
  options:
    toolchain: uv
    virtualenv: venv

# .NET + 커스텀 dotnet 경로
runtime:
  name: dotnet
  options:
    use-executor: /opt/homebrew/bin/dotnet

# 버전 요구사항 포함
runtime: nodejs
requiredPulumiVersion: ">=3.100.0"
```

---

## Converters

### 개요

Converter Plugin은 다른 도구로 작성된 Infrastructure as Code를 Pulumi 프로그램으로 결정론적(deterministically) 변환한다. 변환 대상은 모든 Pulumi 지원 언어로 출력할 수 있다.

> 대부분의 변환 작업에 LLM 사용을 권장한다. Pulumi Neo(Pulumi Cloud의 일부)는 AI를 사용해 인프라 코드를 변환할 수 있다. Terraform의 경우 Neo에 특화된 마이그레이션 스킬이 있다.

### 지원 Converter

| Converter | `--from` 값 | 설명 |
|-----------|-------------|------|
| **Terraform** | `terraform` | Terraform HCL을 Pulumi로 변환 |
| **Azure Resource Manager (ARM)** | `arm` | ARM JSON 템플릿을 Pulumi로 변환 |
| **Azure Bicep** | `bicep` | Bicep 템플릿을 Pulumi로 변환 |
| **Kubernetes** | `kubernetes` | Kubernetes YAML 매니페스트를 Pulumi로 변환 |

### 변환 실행

```bash
# Terraform HCL을 TypeScript로 변환
pulumi convert --from terraform --language typescript

# ARM 템플릿을 Python으로 변환
pulumi convert --from arm --language python

# Bicep을 Go로 변환
pulumi convert --from bicep --language go

# Kubernetes YAML을 C#으로 변환
pulumi convert --from kubernetes --language csharp
```

Converter Plugin은 `pulumi convert` 실행 시 자동으로 설치된다. 자세한 사용법과 옵션은 `pulumi convert` CLI 참조를 확인하라. 각 소스 도구별 단계별 마이그레이션 가이드는 Migrating to Pulumi 문서를 참조하라.

### 수동 설치

```bash
pulumi plugin install converter terraform
pulumi plugin install converter arm
pulumi plugin install converter bicep
pulumi plugin install converter kubernetes
```

수동 설치는 CI/CD 파이프라인 사전 로드나 폐쇄망 환경에서 유용하다.

---

## Stash

### 개요

Stash는 Pulumi에 내장된 리소스로, 값을 스택의 상태(state)에 저장했다가 나중에 조회할 수 있게 한다. 단일 입력 값을 받아 상태에 저장하며, 출력 속성으로 사용할 수 있다.

주요 용도:
- 계산된 값을 영속화
- 프로그램 실행 간 데이터 전달
- 나중에 접근해야 하는 중간 결과 저장

> Stash는 일반 리소스와 동일하게 프로그램 전체에서 고유한 이름을 가져야 한다.

### 속성

| 속성 | 방향 | 설명 |
|------|------|------|
| `input` | 입력 | 저장할 값. 모든 타입 허용 (문자열, 객체, 배열 등) |
| `output` | 출력 | 상태에 저장된 값. 입력이 업데이트되어도 원래 입력 값을 유지 |

> **주의**: `output`은 상태에 저장된 원래 입력 값을 반환한다. 현재 입력 값을 참조하려면 `input` 출력 속성을 사용한다.

### 기본 사용법

**TypeScript:**

```typescript
import * as pulumi from "@pulumi/pulumi";

const myStash = new pulumi.Stash("myStash", {
    input: "Hello, World!",
});

export const stashedValue = myStash.output;
```

**Python:**

```python
import pulumi

my_stash = pulumi.Stash("myStash", input="Hello, World!")
pulumi.export("stashedValue", my_stash.output)
```

**Go:**

```go
package main

import (
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        myStash, err := pulumi.NewStash(ctx, "myStash", &pulumi.StashArgs{
            Input: pulumi.String("Hello, World!"),
        })
        if err != nil {
            return err
        }
        ctx.Export("stashedValue", myStash.Output)
        return nil
    })
}
```

**C#:**

```csharp
using Pulumi;

return await Deployment.RunAsync(() =>
{
    var myStash = new Stash("myStash", new StashArgs
    {
        Input = "Hello, World!",
    });

    return new Dictionary<string, object?>
    {
        ["stashedValue"] = myStash.Output,
    };
});
```

**Java:**

```java
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.core.Output;
import com.pulumi.resources.Stash;
import com.pulumi.resources.StashArgs;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {
            var myStash = new Stash("myStash", StashArgs.builder()
                .input("Hello, World!")
                .build());

            ctx.export("stashedValue", myStash.output());
        });
    }
}
```

**YAML:**

```yaml
resources:
  myStash:
    type: pulumi:pulumi:Stash
    properties:
      input: "Hello, World!"

outputs:
  stashedValue: ${myStash.output}
```

### 복합 값 저장

`input` 속성은 복잡한 객체, 배열, 중첩 구조 등 모든 값을 받을 수 있다. 값은 상태에 저장될 때 Pulumi 속성 값으로 직렬화된다.

**TypeScript:**

```typescript
const configStash = new pulumi.Stash("configStash", {
    input: {
        region: "us-west-2",
        instanceType: "t3.micro",
        tags: {
            Environment: "production",
            Team: "platform",
        },
    },
});
```

**Python:**

```python
config_stash = pulumi.Stash("configStash", input={
    "region": "us-west-2",
    "instanceType": "t3.micro",
    "tags": {
        "Environment": "production",
        "Team": "platform",
    },
})
```

**Go:**

```go
configStash, err := pulumi.NewStash(ctx, "configStash", &pulumi.StashArgs{
    Input: pulumi.Map{
        "region":       pulumi.String("us-west-2"),
        "instanceType": pulumi.String("t3.micro"),
        "tags": pulumi.Map{
            "Environment": pulumi.String("production"),
            "Team":        pulumi.String("platform"),
        },
    },
})
```

**C#:**

```csharp
var configStash = new Stash("configStash", new StashArgs
{
    Input = new Dictionary<string, object>
    {
        ["region"] = "us-west-2",
        ["instanceType"] = "t3.micro",
        ["tags"] = new Dictionary<string, object>
        {
            ["Environment"] = "production",
            ["Team"] = "platform",
        },
    },
});
```

**Java:**

```java
var configStash = new Stash("configStash", StashArgs.builder()
    .input(Map.of(
        "region", "us-west-2",
        "instanceType", "t3.micro",
        "tags", Map.of(
            "Environment", "production",
            "Team", "platform"
        )
    ))
    .build());
```

**YAML:**

```yaml
resources:
  configStash:
    type: pulumi:pulumi:Stash
    properties:
      input:
        region: us-west-2
        instanceType: t3.micro
        tags:
          Environment: production
          Team: platform
```

### Secret 값 저장

입력 값이 secret으로 표시되면 출력도 secret이 되며, 스택 상태에서 암호화된다.

**TypeScript:**

```typescript
const apiKeyStash = new pulumi.Stash("apiKeyStash", {
    input: pulumi.secret("my-secret-api-key"),
});

// output도 secret으로 표시됨
export const apiKey = apiKeyStash.output;
```

**Python:**

```python
api_key_stash = pulumi.Stash("apiKeyStash",
    input=pulumi.Output.secret("my-secret-api-key"))

# output도 secret으로 표시됨
pulumi.export("apiKey", api_key_stash.output)
```

**Go:**

```go
apiKeyStash, err := pulumi.NewStash(ctx, "apiKeyStash", &pulumi.StashArgs{
    Input: pulumi.ToSecret(pulumi.String("my-secret-api-key")),
})
if err != nil {
    return err
}

// output도 secret으로 표시됨
ctx.Export("apiKey", apiKeyStash.Output)
```

**C#:**

```csharp
var apiKeyStash = new Stash("apiKeyStash", new StashArgs
{
    Input = Output.CreateSecret("my-secret-api-key"),
});

// output도 secret으로 표시됨
return new Dictionary<string, object?>
{
    ["apiKey"] = apiKeyStash.Output,
};
```

**Java:**

```java
var apiKeyStash = new Stash("apiKeyStash", StashArgs.builder()
    .input(Output.secret("my-secret-api-key"))
    .build());

// output도 secret으로 표시됨
ctx.export("apiKey", apiKeyStash.output());
```

**YAML:**

```yaml
resources:
  apiKeyStash:
    type: pulumi:pulumi:Stash
    properties:
      input:
        fn::secret: my-secret-api-key

outputs:
  apiKey: ${apiKeyStash.output}
```

Stash된 secret 값을 조회할 때 평문 내용은 기본적으로 표시되지 않고 CLI에서 `[secret]`으로 표시된다. 평문 값을 확인하려면 실행 명령에 `--show-secrets`를 전달하라.

### Stash 값 업데이트

Stash에 저장된 값을 업데이트하려면 교체(replace)해야 한다. 방법은 다음과 같다:

| 방법 | 설명 |
|------|------|
| `--target-replace` | `pulumi up`에 전달하여 엔진에 교체 지시 |
| `pulumi state taint` | 다음 배포 시 리소스가 교체되도록 표시 |
| `TriggerReplacement` | 값 변경 시 리소스 교체를 트리거하는 Resource Option |

교체 없이 `input` 속성을 변경하면 `input` 출력 속성에 반영되지만, `output` 속성은 변경되지 않는다. `output`은 Stash가 생성될 때의 원래 값을 계속 반환한다.

### Stash 삭제

Stash 리소스를 삭제하려면 프로그램에서 제거하고 `pulumi up`을 실행하라. Pulumi는 업데이트 중에 스택 상태에서 Stash를 제거한다.

### 일반 사용 사례

#### 첫 배포 사용자 기록

인프라를 처음 배포한 사용자를 기록해야 할 때:

**TypeScript:**

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as os from "os";

const firstDeployer = new pulumi.Stash("firstDeployer", {
    input: os.userInfo().username,
});

// output은 항상 원래 배포자를 반환
export const originalDeployer = firstDeployer.output;
```

**Python:**

```python
import pulumi
import getpass

first_deployer = pulumi.Stash("firstDeployer",
    input=getpass.getuser())

# output은 항상 원래 배포자를 반환
pulumi.export("originalDeployer", first_deployer.output)
```

**Go:**

```go
import (
    "os"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        firstDeployer, err := pulumi.NewStash(ctx, "firstDeployer", &pulumi.StashArgs{
            Input: pulumi.String(os.Getenv("USER")),
        })
        if err != nil {
            return err
        }
        ctx.Export("originalDeployer", firstDeployer.Output)
        return nil
    })
}
```

**C#:**

```csharp
using Pulumi;
using System;

return await Deployment.RunAsync(() =>
{
    var firstDeployer = new Stash("firstDeployer", new StashArgs
    {
        Input = Environment.UserName,
    });

    return new Dictionary<string, object?>
    {
        ["originalDeployer"] = firstDeployer.Output,
    };
});
```

#### 최초 생성 시간 기록

인프라가 처음 생성된 타임스탬프를 영속화해야 할 때:

**TypeScript:**

```typescript
import * as pulumi from "@pulumi/pulumi";

const creationTime = new pulumi.Stash("creationTime", {
    input: new Date().toISOString(),
});

// 항상 원래 생성 시간 반환
export const firstDeployed = creationTime.output;
```

**Python:**

```python
import pulumi
from datetime import datetime

creation_time = pulumi.Stash("creationTime",
    input=datetime.now().isoformat())

# 항상 원래 생성 시간 반환
pulumi.export("firstDeployed", creation_time.output)
```
