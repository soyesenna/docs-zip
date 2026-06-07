# Pulumi 언어 및 SDK

> https://www.pulumi.com/docs/iac/languages-sdks/
> https://www.pulumi.com/docs/iac/languages-sdks/javascript/
> https://www.pulumi.com/docs/iac/languages-sdks/python/
> https://www.pulumi.com/docs/iac/languages-sdks/go/
> https://www.pulumi.com/docs/iac/languages-sdks/dotnet/
> https://www.pulumi.com/docs/iac/languages-sdks/java/
> https://www.pulumi.com/docs/iac/languages-sdks/hcl/
> https://www.pulumi.com/docs/iac/languages-sdks/python/python-blocking-async/
> https://www.pulumi.com/docs/iac/languages-sdks/python/resource-identity/
> https://www.pulumi.com/docs/iac/languages-sdks/go/go-inputs-outputs/
> https://www.pulumi.com/docs/iac/languages-sdks/javascript/provider-package-versions/

Pulumi는 TypeScript/JavaScript, Python, Go, C#/.NET, Java, HCL, YAML 등 다양한 언어로 인프라를 코드로 관리할 수 있게 한다. 각 언어는 동등한 기능을 제공하며 Pulumi Registry의 모든 프로바이더 전체 영역을 지원한다. 일반 목적 언어를 사용하면 익숙한 문법, 풍부한 생태계, 기존 개발 도구 활용, 정적 타입 안전성 등의 이점이 있다.

---

## 지원 언어 개요

| 언어 | 런타임 | 패키지 레지스트리 | 프로젝트 생성 명령 | 공식 지원 상태 |
|---|---|---|---|---|
| TypeScript / JavaScript | Node.js, Bun | npm | `pulumi new typescript` | 완전 지원 |
| Python | CPython | PyPI | `pulumi new python` | 완전 지원 |
| Go | Go | Go Modules | `pulumi new go` | 완전 지원 |
| C# / F# / VB (.NET) | .NET | NuGet | `pulumi new csharp` | 완전 지원 |
| Java | JVM | Maven Central | `pulumi new java` | 완전 지원 |
| Pulumi YAML | Pulumi CLI (별도 런타임 불필요) | Pulumi Registry | `pulumi new yaml` | 완전 지원 |
| HCL | Pulumi CLI >= 3.235.0 (별도 런타임 불필요) | Pulumi Registry | N/A (직접 프로젝트 생성) | 지원 (pulumi-labs) |

> 즐겨 쓰는 언어가 목록에 없다면, Pulumi는 오픈 소스이므로 [직접 언어를 추가](https://github.com/pulumi/pulumi)할 수 있다.

---

## 언어별 설치 요구사항

### TypeScript / JavaScript

| 항목 | 내용 |
|---|---|
| 런타임 | Node.js Current, Active, Maintenance LTS 버전 (최신 LTS 권장). Pulumi 3.227.0부터 Bun 런타임 지원 |
| `Pulumi.yaml` 설정 | `runtime: nodejs` 또는 `runtime: bun` |
| 패키지 매니저 | npm (기본), Yarn 1 Classic, pnpm, Bun |
| SDK 패키지 | `@pulumi/pulumi` (npm) |

**Bun 런타임 제한사항**: Function serialization과 dynamic provider는 Bun에서 지원되지 않는다. Bun이 Node.js v8/inspector API를 아직 완전히 구현하지 않았기 때문이다.

**패키지 매니저 감지**: Pulumi는 `yarn.lock`이 있거나 `PULUMI_PREFER_YARN=true` 환경변수가 설정되면 Yarn을 사용한다. pnpm은 `pnpm-lock.yaml`, Bun은 `bun.lock`(Bun >= 1.2) 또는 `bun.lockb`(구버전) 파일로 감지한다. Yarn Plug'n'Play는 지원하지 않는다.

**Dynamic providers 패키지 매니저 호환성**: Dynamic providers가 모든 패키지 매니저에서 올바르게 동작하지 않을 수 있다. 문제가 발생하면 npm 또는 Yarn 1 Classic을 사용한다.

**Provider package version management**: 프로바이더 패키지 버전 관리에 대한 자세한 내용은 [Provider package version management](https://www.pulumi.com/docs/iac/languages-sdks/javascript/provider-package-versions/)를 참고한다.

**TypeScript 버전**: Pulumi는 하위 호환을 위해 TypeScript 3.8.3을 번들로 포함한다. 하지만 로컬 `node_modules`에 TypeScript가 있으면 그 버전을 우선 사용한다. TypeScript 3.8 이상 모든 버전(최신 TypeScript 6 포함)을 지원한다.

**Enabling async support**: CommonJS 프로젝트에서 비동기 작업을 수행해야 하는 경우, 진입점 함수를 `async`로 만들어 Pulumi가 이를 감지하도록 할 수 있다. `index.ts`(또는 `package.json`의 `main`이 가리키는 파일)에서 다음 패턴을 사용한다:

```typescript
module.exports = async () => {
    // 이 함수 내에서 await 사용 가능
    const azs = await aws.getAvailabilityZones({ state: "available" });
    // 리소스 선언...
};
```

이 패턴은 CommonJS 프로젝트에서 top-level `await`를 사용할 수 없을 때 유용하다. Pulumi는 `module.exports`가 `async` 함수인 경우 이를 감지하여 Promise 해결을 기다린다.

---

### Python

| 항목 | 내용 |
|---|---|
| 런타임 | 현재 지원되는 모든 Python 버전 (최신 릴리스 권장) |
| `Pulumi.yaml` 설정 | `runtime: python` |
| 패키지 매니저 | pip (기본), Poetry (>= 1.8.0), uv |
| 가상환경 | 권장. pip는 `virtualenv` 옵션 필수, uv는 `.venv` 기본값 |
| SDK 패키지 | `pulumi` (PyPI) |

**패키지 매니저 설정 예시**:

```yaml
# pip
runtime:
  name: python
  options:
    toolchain: pip
    virtualenv: venv

# uv
runtime:
  name: python
  options:
    toolchain: uv
    virtualenv: .venv

# poetry
runtime:
  name: python
  options:
    toolchain: poetry
```

**타입 검사**: Pulumi Python 라이브러리는 타입 힌트를 포함한다. `mypy` 및 `pyright`를 first-class로 지원하며, `Pulumi.yaml`에서 `typechecker` 옵션으로 실행 전 자동 검사를 활성화할 수 있다.

```yaml
runtime:
  name: python
  options:
    typechecker: mypy
```

**Resource identity**: 모든 Pulumi 리소스는 네 가지 형태의 식별 정보(logical name, physical name, physical ID, URN)를 노출한다. 각각 다른 컨텍스트에서 적합하며, 잘못된 형태를 전달하는 것이 Python 프로그램에서 타입 오류의 가장 흔한 원인이다. 예를 들어 `ResourceOptions`의 `parent`, `depends_on` 필드는 리소스 객체 자체를 기대하지만, 대부분의 리소스 입력 속성(`vpc_id` 등)은 업스트림 리소스의 `id` 출력(`Output[str]`)을 기대한다. 자세한 내용은 [Resource identity in Python](https://www.pulumi.com/docs/iac/languages-sdks/python/resource-identity/)을 참고한다.

**Blocking & async**: Pulumi Python 프로그램은 싱글 스레드에서 이벤트 루프를 사용해 실행된다. 비동기 처리 방식과 blocking 호출 간의 상호작용에 대한 자세한 내용은 [Blocking & async](https://www.pulumi.com/docs/iac/languages-sdks/python/python-blocking-async/)를 참고한다.

**Self-managed virtual environments**: pip를 사용할 때는 `virtualenv` 옵션이 필수이며, uv는 기본적으로 `.venv` 디렉터리를 사용한다. Poetry는 자체 가상환경 관리를 수행한다. 가상환경을 수동으로 관리하려면 프로젝트 디렉터리에서 `python -m venv venv`를 실행한 후 활성화한다.

**가상환경 없이 pip 사용**: 별도의 가상환경을 사용하지 않으려면 `venv` 디렉터리를 삭제하고 `Pulumi.yaml`에서 `virtualenv` 옵션을 제거하면 된다. 이 경우 이미 활성화된 가상환경 셸에서 `pulumi up`을 실행하거나, Pipenv 같은 도구를 사용해 `pipenv run pulumi ...` 명령으로 실행할 수 있다.

---

### Go

| 항목 | 내용 |
|---|---|
| 런타임 | 현재 지원되는 모든 Go 버전 (최신 릴리스 권장) |
| `Pulumi.yaml` 설정 | `runtime: go` |
| 의존성 관리 | Go Modules (`go.mod`) |
| SDK 패키지 | `github.com/pulumi/pulumi/sdk/v3` |
| 클라우드별 템플릿 | `aws-go`, `azure-go`, `gcp-go` |

**사전 빌드 바이너리**: 직접 빌드한 실행 파일을 사용하려면 `binary` 옵션을 설정한다.

```yaml
runtime:
  name: go
  options:
    binary: ./bin/my-program
```

**Inputs & outputs**: Go에서 Input/Output 타입, `ApplyT`, `All`, output lifting 등의 작동 방식에 대한 자세한 내용은 [Inputs & outputs in Go](https://www.pulumi.com/docs/iac/languages-sdks/go/go-inputs-outputs/)를 참고한다.

---

### C# / F# / Visual Basic (.NET)

| 항목 | 내용 |
|---|---|
| 런타임 | 현재 지원되는 모든 .NET 버전 (Pulumi는 .NET 8, 9, 10에서 테스트) |
| `Pulumi.yaml` 설정 | `runtime: dotnet` |
| 패키지 레지스트리 | NuGet |
| SDK 패키지 | `Pulumi` (NuGet). F#은 `Pulumi.FSharp` 추가 가능 |
| 클라우드별 템플릿 | `aws-csharp`, `azure-csharp`, `gcp-csharp` (F#: `aws-fsharp` 등) |

**지원 .NET 언어**: C#, F#, Visual Basic 모두 완전 지원. 이 외에도 .NET 런타임을 대상으로 하는 언어는 사용 가능하지만, `pulumi new` 템플릿은 C#, F#, VB만 제공된다.

**프로그램 진입점**: `Deployment.RunAsync`를 호출하는 .NET 콘솔 애플리케이션. 최신 C# 템플릿은 top-level statements를 사용한다.

**사전 빌드 어셈블리**: 직접 빌드한 .dll을 사용하려면 `binary` 옵션에 경로를 지정한다.

```yaml
runtime:
  name: dotnet
  options:
    binary: bin/MyInfra.dll
```

**F# 사용자**: F#에서 Pulumi를 사용할 때 [`Pulumi.FSharp`](https://www.pulumi.com/docs/reference/pkg/dotnet/pulumi.fsharp/pulumi.fsharp.html) NuGet 패키지를 추가하면 F# 친화적인 관용적 헬퍼(idiomatic helpers)를 사용할 수 있다. Automation API를 활용하려면 [`Pulumi.Automation`](https://www.pulumi.com/docs/reference/pkg/dotnet/pulumi.automation/pulumi.automation.html) NuGet 패키지를 사용한다.

**Troubleshooting .NET 버전**: .NET 버전 관련 문제가 발생하는 경우, Pulumi는 .NET 8, 9, 10에서 테스트한다. 지원되는 .NET 버전을 사용 중인지 확인하고, 문제가 지속되면 [Troubleshooting](https://www.pulumi.com/docs/iac/troubleshooting/) 문서를 참고한다.

---

### Java

| 항목 | 내용 |
|---|---|
| 런타임 | Java 11 이상 (지원되는 모든 버전) |
| `Pulumi.yaml` 설정 | `runtime: java` |
| 빌드 도구 | Apache Maven (>= 3.6.1, 기본), Gradle, 사전 빌드 JAR |
| SDK 패키지 | `com.pulumi` (Maven Central) |
| 클라우드별 템플릿 | `aws-java`, `azure-java`, `gcp-java` |

**JVM 언어**: 공식적으로는 Java만 지원·문서화되지만, Kotlin, Scala, Groovy 등 다른 JVM 언어에서도 Pulumi Java SDK를 사용할 수 있다.

**자동 플러그인 감지**: Pulumi는 프로그램에서 참조하는 프로바이더 패키지를 자동으로 감지하여 필요한 플러그인을 다운로드한다. 별도로 `pulumi plugin install`을 실행할 필요가 없다.

**빌드 도구별 설정**:

| 빌드 도구 | 템플릿 | 진입점 설정 |
|---|---|---|
| Maven | `pulumi new java` | `pom.xml`의 `mainClass` 속성 |
| Gradle | `pulumi new java-gradle` | `build.gradle`의 `application.mainClass` |
| JAR | N/A | `runtime.options.binary`에 JAR 경로 지정 |

---

## 프로젝트 구조 및 진입점

### 언어별 기본 진입점 비교

| 언어 | 기본 진입점 파일 | 진입점 설정 방법 | Stack Outputs 방식 |
|---|---|---|---|
| TypeScript / JS | `index.ts` / `index.js` | `package.json`의 `main` 필드 | `export const out = ...` |
| Python | `__main__.py` | `Pulumi.yaml`의 `main` 속성 | `pulumi.export("out", ...)` |
| Go | `main` 패키지의 `main()` 함수 | `Pulumi.yaml`의 `main` 속성 | `ctx.Export("out", ...)` |
| .NET | `Program.cs` (top-level statements) | `Pulumi.yaml`의 `main` 속성 (`.csproj` 경로) | `Deployment.RunAsync` 반환값 또는 `[Output]` 속성 |
| Java | `App.java` | `pom.xml`의 `mainClass` 또는 Gradle `application.mainClass` | `ctx.export("out", ...)` |

### Pulumi.yaml 진입점 오버라이드 예시

```yaml
# Python
name: my-project
runtime: python
main: app.py

# Go
name: my-project
runtime: go
main: ./infra
```

---

## 주요 프로그래밍 패턴

### 리소스 선언 패턴 비교

| 언어 | 리소스 생성 | Inputs & Outputs | 비동기 패턴 |
|---|---|---|---|
| TypeScript | `new aws.s3.Bucket("my-bucket")` | `Input<T>` / `Output<T>` | `async` 함수 또는 top-level `await` (ESM) |
| Python | `aws.s3.Bucket("my-bucket")` | `Input[T]` / `Output[T]` | 싱글 스레드 + 이벤트 루프 |
| Go | `s3.NewBucket(ctx, "my-bucket", nil)` | `Input` / `Output` (`ApplyT`, `All`) | `pulumi.Run` 내에서 처리 |
| .NET | `new Bucket("my-bucket")` | `Input<T>` / `Output<T>` | `Deployment.RunAsync` |
| Java | `new Bucket("my-bucket", BucketArgs.builder().build())` | `Output<T>` (별도 `Input<T>` 타입 없음, `Args` 빌더가 오버로드 제공) | `Pulumi.run` |
| HCL | `resource "aws_s3_bucket" "my_bucket" { ... }` | 변수 참조(`var.x`) 및 속성 접근(`resource.attr`) | N/A (선언적) |

### Java의 Input/Output 특징

Java는 다른 언어와 달리 `Input<T> = T | Output<T>` 타입이 없다. 대신 `Args` 빌더가 일반 `T` 값과 `Output<T>` 값을 모두 받는 오버로드를 제공한다.

프로바이더 함수는 두 가지 형태를 제공한다:
- **output form** (`getAmi()`): `Input` 값을 받고 `Output<T>` 반환
- **direct form** (`getAmiPlain()`): 일반 인자를 받고 `CompletableFuture<T>` 반환

---

## TypeScript/JavaScript 고급 기능

### Native ESM 지원

기본 템플릿은 TypeScript를 CommonJS로 컴파일한다. Native ESM을 사용하려면 `package.json`에 `"type": "module"`을 설정하고 `tsconfig.json`에서 `module`과 `moduleResolution`을 `nodenext`로 지정한다.

```json
{
    "type": "module"
}
```

```json
{
    "compilerOptions": {
        "target": "ES2022",
        "module": "nodenext",
        "moduleResolution": "nodenext"
    }
}
```

**ts-node 및 tsx 설정**: Native ESM 환경에서 TypeScript를 실행하려면 `ts-node`(>= 10) 또는 `tsx`가 필요하다. 설치 방법:

```bash
# ts-node 사용 시
npm install typescript@^5
npm install ts-node@^10

# tsx 사용 시
npm install tsx
```

**참고**: `@pulumi/pulumi` 3.183.0 이전 버전에서는 `Pulumi.yaml`의 `nodeargs`에 `--loader ts-node/esm --no-warnings`를 수동으로 설정해야 했다:

```yaml
runtime:
  name: nodejs
  options:
    nodeargs: "--loader ts-node/esm --no-warnings"
```

최신 버전에서는 Pulumi가 ESM 로더를 자동으로 설정한다. 단, `Pulumi.yaml`에서 `--loader`, `--import`, `--require` 인자를 직접 지정하면 자동 ESM 로더 설정이 비활성화되므로 주의한다.

### Using ESM-only modules with CommonJS Pulumi templates

CommonJS 기반 Pulumi 템플릿에서 ESM 전용 모듈을 `require()`로 로드하려고 하면 `ERR_REQUIRE_ESM` 오류가 발생한다. 이 문제를 해결하는 두 가지 방법이 있다:

1. **프로젝트를 ESM으로 전환**: `package.json`에 `"type": "module"`을 추가하고 `tsconfig.json`을 ESM 설정으로 변경한다. 위의 Native ESM 설정을 참고한다.
2. **Node.js 버전 업그레이드**: Node.js v20.19.0 또는 v22.12.0 이상에서는 CommonJS 컨텍스트에서도 ESM 모듈의 동적 `import()`를 통한 로드가 개선되었다.

### Top-level await

Native ESM을 사용하면 top-level `await`를 사용할 수 있다. `tsconfig.json`에서 `target`을 `ES2022` 이상으로 설정하면 된다.

```typescript
import * as aws from "@pulumi/aws";

const azs: aws.GetAvailabilityZonesResult = await aws.getAvailabilityZones({ state: "available" });

const buckets: aws.s3.Bucket[] = azs.names.map(az =>
    new aws.s3.Bucket(`my-bucket-${az}`)
);

export const bucketNames = buckets.map(b => b.id);
```

### TypeScript 내장 지원 비활성화

직접 컴파일하는 경우 내장 TypeScript 지원을 끌 수 있다.

```yaml
runtime:
  name: nodejs
  options:
    typescript: false
```

---

## 언어별 제한사항 및 차이점

| 항목 | TypeScript / JS | Python | Go | .NET | Java | HCL |
|---|---|---|---|---|---|---|
| Policy SDK 작성 | 지원 (`@pulumi/policy`) | 지원 (`pulumi_policy`) | 미지원 | 미지원 | 미지원 | 미지원 |
| Automation API | 지원 | 지원 | 지원 | 지원 (`Pulumi.Automation`) | 지원 | 미지원 |
| Dev 빌드 설치 | `npm add @pulumi/pulumi@dev` | `pip install --pre` | `go get ...@master` | `dotnet add package --prerelease` | 미제공 (Maven Central 릴리스만) | N/A |
| 동적 프로바이더 | 지원 (Node.js), 미지원 (Bun) | 지원 | 지원 | 지원 | 지원 | 미지원 |
| Function serialization | 지원 (Node.js), 미지원 (Bun) | 미지원 | 미지원 | 미지원 | 미지원 | 미지원 |
| 패키지 수 (프로바이더) | 100+ | 100+ | 100+ | 100+ | 100+ | 100+ |

> **Policy as Code**: TypeScript/JavaScript(`@pulumi/policy`), Python(`pulumi_policy`), 그리고 OPA(Rego)로 Policy를 작성할 수 있다. 작성된 정책은 모든 언어로 된 Pulumi 프로그램에 적용할 수 있다. 자세한 내용은 [Policy as Code 문서](https://www.pulumi.com/docs/insights/policy/#languages)를 참고한다.

---

## YAML 및 HCL 지원

### Pulumi YAML

Pulumi YAML은 인프라를 최대한 단순하게 기술하기 위해 설계된 구성 언어이다. 별도 런타임 설치 없이 Pulumi CLI만으로 실행할 수 있다.

```yaml
name: simple-yaml
runtime: yaml
resources:
  my-bucket:
    type: aws:s3:Bucket
outputs:
  bucketName: ${my-bucket.id}
```

- 템플릿: `pulumi new aws-yaml`, `pulumi new azure-yaml`, `pulumi new gcp-yaml`, `pulumi new kubernetes-yaml`
- Compiler 지원: `runtime.options.compiler` 설정을 통해 CUE 등 YAML/JSON으로 컴파일되는 언어를 사용할 수 있다.

```yaml
runtime:
  name: yaml
  options:
    compiler: cue export
```

### HCL (Pulumi HCL)

Pulumi HCL은 Terraform 형태의 HCL 구문으로 Pulumi 프로그램을 작성할 수 있게 하는 언어 플러그인이다. 친숙한 HCL 블록, 표현식, 내장 함수를 그대로 사용하면서 Pulumi의 상태 관리, 시크릿 처리, 배포 엔진을 활용할 수 있다. [pulumi-labs/pulumi-hcl](https://github.com/pulumi-labs/pulumi-hcl) 저장소에서 개발된다.

**사전 요구사항**: Pulumi CLI 버전 3.235.0 이상. HCL 언어 및 컨버터 플러그인이 CLI에 포함되어 있다.

**프로젝트 구조**: `Pulumi.yaml`에 `runtime: hcl`을 설정하고, 프로젝트 디렉터리에 하나 이상의 `.hcl` 파일을 둔다.

```yaml
# Pulumi.yaml
name: simple-hcl
runtime: hcl
description: A simple Pulumi HCL project
```

```hcl
# main.hcl
pulumi {
  required_providers {
    random = {
      source  = "pulumi/random"
      version = ">= 4.0.0"
    }
  }
}

variable "prefix" {
  type    = string
  default = "test"
}

resource "random_pet" "my_pet" {
  prefix = var.prefix
  length = 2
}

output "pet_name" {
  value = random_pet.my_pet.id
}
```

**프로바이더 소스**: 프로바이더 소스는 반드시 `pulumi/` 네임스페이스를 사용해야 한다(예: `pulumi/aws`). `hashicorp/` 네임스페이스는 사용할 수 없다.

**Terraform 호환성**: Pulumi HCL은 Terraform 형태의 HCL 구문과 광범위하게 호환된다. 리소스, 데이터 소스, 변수, 로컬, 출력, 모듈, 표현식, 대부분의 내장 함수가 HashiCorp 문서대로 동작한다. 주요 차이점은 다음과 같다:

| 항목 | 설명 |
|---|---|
| 프로바이더 소스 | `pulumi/` 네임스페이스 사용 (not `hashicorp/`) |
| 리소스 교체 순서 | 새 리소스를 먼저 생성한 후 기존 리소스를 삭제 (Terraform과 반대) |
| 교체 순서 변경 | `lifecycle` 블록에 `create_before_destroy = false` 설정 시 삭제 우선 동작 |

**HCL 패키지**: Pulumi Registry에서 100개 이상의 패키지를 HCL 프로그램에서 사용할 수 있으며, `pulumi required_providers` 블록에 선언하여 사용한다. Pulumi Registry에 없는 Terraform/OpenTofu 프로바이더는 [Any Terraform Provider](https://www.pulumi.com/docs/iac/concepts/resources/providers/any-terraform-provider/)를 사용하여 즉석에서 로컬 패키지를 생성할 수 있다.

**추가 정보**: 더 많은 예제는 [Pulumi HCL GitHub 저장소](https://github.com/pulumi-labs/pulumi-hcl)에서, 전체 사양은 [Pulumi HCL Reference](https://www.pulumi.com/docs/iac/languages-sdks/hcl/hcl-language-reference/)에서 확인할 수 있다.

---

## 언어 선택 가이드

| 기준 | 추천 언어 | 이유 |
|---|---|---|
| 웹 개발 경험이 있는 경우 | TypeScript / JavaScript | npm 생태계, 널리 쓰이는 예제, 가장 풍부한 커뮤니티 자료 |
| 데이터/ML 엔지니어 | Python | PyPI 생태계, pytest 등 익숙한 도구 |
| 시스템/플랫폼 엔지니어 | Go | 빠른 컴파일, 정적 타입, 단일 바이너리 |
| 엔터프라이즈 .NET 환경 | C# / F# / VB | NuGet, Visual Studio, 기존 .NET 도구 활용 |
| 엔터프라이즈 Java 환경 | Java | Maven Central, Gradle, 기존 JVM 도구 활용 |
| 프로그래밍 경험이 없는 경우 | Pulumi YAML | 별도 언어 학습 없이 단일 파일로 인프라 정의 |
| 기존 Terraform 사용자 | HCL | 친숙한 HCL 구문 그대로 사용, Pulumi 상태 관리·시크릿 처리 활용 |
| Policy as Code 작성 필요 | TypeScript 또는 Python | Policy SDK 지원 언어 |
| Automation API 내장 필요 | TypeScript, Python, Go, .NET, Java 모두 지원 | 사용 언어와 동일한 언어로 내장 가능 |

---

## Hello World 코드 예제

### TypeScript

```typescript
import * as aws from "@pulumi/aws";

const bucket = new aws.s3.Bucket("my-bucket");

export const bucketName = bucket.id;
```

### Python

```python
import pulumi
import pulumi_aws as aws

bucket = aws.s3.Bucket("my-bucket")

pulumi.export("bucket_name", bucket.id)
```

### Go

```go
package main

import (
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
    "github.com/pulumi/pulumi-aws/sdk/v6/go/aws/s3"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        bucket, err := s3.NewBucket(ctx, "my-bucket", nil)
        if err != nil {
            return err
        }
        ctx.Export("bucketName", bucket.ID())
        return nil
    })
}
```

### C# (.NET)

```csharp
using Pulumi;
using Aws = Pulumi.Aws;

return await Deployment.RunAsync(() =>
{
    var bucket = new Aws.S3.Bucket("my-bucket");

    return new Dictionary<string, object?>
    {
        ["bucketName"] = bucket.Id,
    };
});
```

### Java

```java
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.aws.s3.Bucket;
import com.pulumi.aws.s3.BucketArgs;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {
            var bucket = new Bucket("my-bucket", BucketArgs.builder().build());
            ctx.export("bucketName", bucket.id());
        });
    }
}
```

### Pulumi YAML

```yaml
name: hello-world
runtime: yaml
resources:
  my-bucket:
    type: aws:s3:Bucket
outputs:
  bucketName: ${my-bucket.id}
```

### HCL

```hcl
# Pulumi.yaml
name: hello-world
runtime: hcl

# main.hcl
pulumi {
  required_providers {
    aws = {
      source  = "pulumi/aws"
      version = ">= 6.0.0"
    }
  }
}

resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-bucket"
}

output "bucket_name" {
  value = aws_s3_bucket.my_bucket.id
}
```

---

## 의존성 추가 방법

| 언어 | 명령어 |
|---|---|
| TypeScript / JS | `npm install @pulumi/aws` |
| Python (pip) | `pip install pulumi_aws` (또는 `requirements.txt`에 추가 후 `pip install -r requirements.txt`) |
| Python (Poetry) | `poetry add pulumi_aws` |
| Python (uv) | `uv add pulumi_aws` |
| Go | `go get github.com/pulumi/pulumi-aws/sdk/v6` |
| .NET | `dotnet add package Pulumi.Aws` |
| Java (Maven) | `pom.xml`에 의존성 추가 |
| Java (Gradle) | `build.gradle`에 의존성 추가 |
| HCL | `pulumi required_providers` 블록에 프로바이더 선언 |

---

## Dev 빌드 (사전 릴리스 버전)

| 언어 | Dev 빌드 설치 명령어 |
|---|---|
| TypeScript / JS | `npm add @pulumi/pulumi@dev` 또는 `npm add @pulumi/aws@dev` |
| Python (pip) | `pip install --pre -r requirements.txt` |
| Python (Poetry) | `poetry add --allow-prereleases <PACKAGE_NAME>` |
| Python (uv) | `uv add --prerelease=allow <PACKAGE_NAME>` |
| Go | `go get github.com/pulumi/pulumi/sdk/v3@master` |
| .NET | `dotnet add package Pulumi --prerelease` |
| Java | 미제공 (Maven Central 릴리스 버전만 사용) |
| HCL | N/A (CLI에 플러그인 내장) |

> Dev 빌드는 아직 릴리스되지 않은 수정 사항이 필요한 경우에만 사용한다. 자세한 내용은 [Using dev builds for unreleased fixes](https://www.pulumi.com/docs/iac/operations/debugging/using-dev-builds/)를 참고한다.
