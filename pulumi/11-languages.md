# Pulumi 언어 및 SDK

> https://www.pulumi.com/docs/iac/languages-sdks/
> https://www.pulumi.com/docs/iac/languages-sdks/javascript/
> https://www.pulumi.com/docs/iac/languages-sdks/python/
> https://www.pulumi.com/docs/iac/languages-sdks/go/
> https://www.pulumi.com/docs/iac/languages-sdks/dotnet/
> https://www.pulumi.com/docs/iac/languages-sdks/java/

Pulumi는 TypeScript/JavaScript, Python, Go, C#/.NET, Java, YAML 등 다양한 언어로 인프라를 코드로 관리할 수 있게 한다. 각 언어는 동등한 기능을 제공하며 Pulumi Registry의 모든 프로바이더 전체 영역을 지원한다. 일반 목적 언어를 사용하면 익숙한 문법, 풍부한 생태계, 기존 개발 도구 활용, 정적 타입 안전성 등의 이점이 있다.

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

> 즐겨 쓰는 언어가 목록에 없다면, Pulumi는 오픈 소스이므로 [직접 언어를 추가](https://github.com/pulumi/pulumi)할 수 있다.

---

## 언어별 설치 요구사항

### TypeScript / JavaScript

| 항목 | 내용 |
|---|---|
| 런타임 | Node.js Current, Active, Maintenance LTS 버전 (최신 LTS 권장). Bun 3.227.0 이상도 지원 |
| `Pulumi.yaml` 설정 | `runtime: nodejs` 또는 `runtime: bun` |
| 패키지 매니저 | npm (기본), Yarn 1 Classic, pnpm, Bun |
| SDK 패키지 | `@pulumi/pulumi` (npm) |

**Bun 런타임 제한사항**: Function serialization과 dynamic provider는 Bun에서 지원되지 않는다. Bun이 Node.js v8/inspector API를 아직 완전히 구현하지 않았기 때문이다.

**패키지 매니저 감지**: Pulumi는 `yarn.lock`이 있거나 `PULUMI_PREFER_YARN=true` 환경변수가 설정되면 Yarn을 사용한다. pnpm은 `pnpm-lock.yaml`, Bun은 `bun.lock`(Bun >= 1.2) 또는 `bun.lockb`(구버전) 파일로 감지한다. Yarn Plug'n'Play는 지원하지 않는다.

**TypeScript 버전**: Pulumi는 하위 호환을 위해 TypeScript 3.8.3을 번들로 포함한다. 하지만 로컬 `node_modules`에 TypeScript가 있으면 그 버전을 우선 사용한다. TypeScript 3.8 이상 모든 버전(최신 TypeScript 6 포함)을 지원한다.

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

| 항목 | TypeScript / JS | Python | Go | .NET | Java |
|---|---|---|---|---|---|
| Policy SDK 작성 | 지원 (`@pulumi/policy`) | 지원 (`pulumi_policy`) | 미지원 | 미지원 | 미지원 |
| Automation API | 지원 | 지원 | 지원 | 지원 (`Pulumi.Automation`) | 지원 |
| Dev 빌드 설치 | `npm add @pulumi/pulumi@dev` | `pip install --pre` | `go get ...@master` | `dotnet add package --prerelease` | 미제공 (Maven Central 릴리스만) |
| 동적 프로바이더 | 지원 (Node.js), 미지원 (Bun) | 지원 | 지원 | 지원 | 지원 |
| Function serialization | 지원 (Node.js), 미지원 (Bun) | 미지원 | 미지원 | 미지원 | 미지원 |
| 패키지 수 (프로바이더) | 100+ | 100+ | 100+ | 100+ | 100+ |

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

### HCL (Terraform)

Pulumi는 Terraform HCL 설정을 Pulumi 프로젝트로 변환하는 도구(`pulumi convert --from terraform`)를 제공한다. HCL을 직접 실행하는 것은 아니며, 변환 후 지원 언어 중 하나로 관리하게 된다.

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

> Dev 빌드는 아직 릴리스되지 않은 수정 사항이 필요한 경우에만 사용한다. 자세한 내용은 [Using dev builds for unreleased fixes](https://www.pulumi.com/docs/iac/operations/debugging/using-dev-builds/)를 참고한다.
