# Pulumi 프로젝트와 스택

> https://www.pulumi.com/docs/iac/concepts/projects/
> https://www.pulumi.com/docs/iac/concepts/stacks/
> https://www.pulumi.com/docs/iac/concepts/projects/project-file/
> https://www.pulumi.com/docs/iac/guides/basics/

Pulumi의 핵심 구성 단위는 **프로젝트(Project)** 와 **스택(Stack)** 입니다. 프로젝트는 `Pulumi.yaml` 파일이 포함된 폴더로, 런타임과 메타데이터를 정의합니다. 스택은 프로젝트의 격리된 배포 인스턴스로, 개발·스테이징·프로덕션 같은 환경별로 독립적으로 구성됩니다. 이 문서는 프로젝트 파일 구조, 스택 관리 명령어, 조직 네이밍, 멀티스택 워크플로우를 정리합니다.

---

## 프로젝트(Project)

Pulumi 프로젝트는 `Pulumi.yaml` 파일을 포함하는 폴더입니다. 런타임에 가장 가까운 상위 폴더의 `Pulumi.yaml`이 현재 프로젝트를 결정합니다. 프로젝트는 `pulumi new` 명령으로 생성합니다.

### 프로젝트 파일(Pulumi.yaml)

`Pulumi.yaml`은 프로젝트의 런타임·메타데이터를 정의하는 필수 파일입니다. 파일명은 대문자 `P`로 시작해야 하며 `.yml`, `.yaml` 확장자 모두 지원됩니다.

#### 기본 구조

```yaml
# TypeScript
name: webserver
runtime: nodejs
description: A minimal Pulumi program.
```

```yaml
# Python
name: webserver
runtime: python
description: A minimal Pulumi program.
```

#### 컴파일된 언어의 사전 빌드 실행 파일

Go, .NET, Java는 `binary` 옵션으로 사전 빌드된 실행 파일을 지정할 수 있습니다. 이 경우 배포 시 빌드 단계를 건너뜁니다.

```yaml
# Go 예시
name: my-project
description: A precompiled Go Pulumi program.
runtime:
  name: go
  options:
    binary: mybinary
```

```yaml
# .NET 예시
name: my-project
description: A precompiled .NET Pulumi program.
runtime:
  name: dotnet
  options:
    binary: bin/MyInfra.dll
```

```yaml
# Java 예시
name: my-project
description: A precompiled Java Pulumi program.
runtime:
  name: java
  options:
    binary: target/my-project-1.0-SNAPSHOT-jar-with-dependencies.jar
```

#### YAML 런타임 인라인 리소스

YAML 런타임은 `Pulumi.yaml` 파일 내에 리소스를 직접 정의할 수 있습니다.

```yaml
name: my-project
runtime: yaml
resources:
  bucket:
    type: aws:s3:Bucket
```

---

## Pulumi.yaml 속성 레퍼런스

다음은 `Pulumi.yaml` 파일에서 사용 가능한 모든 속성입니다.

### 최상위 속성

| 속성 | 필수 | 설명 |
|------|------|------|
| `name` | **필수** | 프로젝트 이름. 알파벳, 하이픈, 밑줄, 마침표 사용 가능 |
| `runtime` | **필수** | 언어 런타임: `nodejs`, `python`, `go`, `dotnet`, `java`, `yaml`, `bun` |
| `description` | 선택 | 프로젝트에 대한 간단한 설명 |
| `author` | 선택 | 프로젝트 작성자 |
| `website` | 선택 | 프로젝트 웹사이트 또는 리포지토리 URL |
| `license` | 선택 | 라이선스(예: `Apache-2.0`, `MIT`) |
| `config` | 선택 | 프로젝트 수준 구성(v3.44 추가) |
| `packages` | 선택 | 프로그램에서 사용할 추가 패키지(v3.157.0 추가) |
| `main` | 선택 | Pulumi 프로그램 경로. `Pulumi.yaml` 위치 기준 상대 경로 |
| `stackConfigDir` | 선택 | 구성 디렉터리 위치. `Pulumi.yaml` 위치 기준 상대 경로 |
| `backend` | 선택 | 프로젝트의 백엔드 설정 |
| `options` | 선택 | 추가 프로젝트 옵션 |
| `template` | 선택 | `pulumi new`로 새 스택 생성 시 사용할 템플릿 구성 |
| `plugins` | 선택 | 플러그인 선택 재정의(플러그인 개발용) |
| `requiredPulumiVersion` | 선택 | 필요한 Pulumi CLI 버전 범위 |

### `runtime` 옵션

`runtime` 속성은 `options` 하위 속성으로 런타임별 세부 구성을 제공합니다.

| 옵션 | 적용 런타임 | 설명 |
|------|------------|------|
| `typescript` | `nodejs` | `ts-node` 사용 여부(Boolean) |
| `tsconfig` | `nodejs` | 커스텀 `tsconfig.json` 경로 |
| `nodeargs` | `nodejs` | `node`에 전달할 인수 |
| `packagemanager` | `nodejs` | 패키지 매니저: `npm`, `pnpm`, `yarn`, `bun`. 미설정 시 lockfile에서 자동 감지 |
| `buildTarget` | `go` | 컴파일된 Go 바이너리 저장 경로. `binary`와 함께 사용 불가 |
| `binary` | `go`, `dotnet`, `java` | 사전 빌드된 실행 파일 경로 |
| `toolchain` | `python` | 가상환경 관리 도구: `pip`, `poetry`, `uv`. 미설정 시 lockfile에서 자동 감지 |
| `virtualenv` | `python`(`pip`/`uv`) | 가상환경 경로 |
| `typechecker` | `python` | 타입 체커: `mypy` 또는 `pyright` |
| `compiler` | `yaml` | 표준 출력으로 실행되는 컴파일러 실행 파일 및 인수 |

### `backend` 옵션

| 옵션 | 필수 | 설명 |
|------|------|------|
| `url` | 선택 | 백엔드 URL을 명시적으로 설정 |

### `options` 옵션

| 옵션 | 필수 | 설명 |
|------|------|------|
| `refresh` | 선택 | `always`로 설정 시 Pulumi 작업 전 상태 새로고침 |

### `packages` 옵션

프로그램이 사용하는 패키지를 선언합니다. `pulumi install` 명령이 이 패키지를 설치하고 SDK를 생성합니다.

| 속성 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `source` | string | **필수** | 패키지 소스(로컬 경로, Git 리포지토리 URL, 플러그인 이름) |
| `version` | string | 선택 | 패키지 버전 |
| `parameters` | List\<string\> | 선택 | 소스 패키지에 대한 파라미터 목록 |
| `checksums` | Map\<string, string\> | 선택 | `os-arch` 키(예: `linux-x64`)와 체크섬 매핑 |
| `pluginDownloadURL` | string | 선택 | 커스텀 플러그인 다운로드 위치 |

### `config` 옵션 (프로젝트 수준)

프로젝트 수준 `config`는 구성 속성 키를 값 또는 구조화된 선언에 매핑합니다.

**값 선언:**

| 속성 | 필수 | 설명 |
|------|------|------|
| `value` | **필수** | 구성 속성의 값 |

**스키마 선언:**

| 속성 | 필수 | 설명 |
|------|------|------|
| `type` | 선택 | 타입: `string`, `boolean`, `integer`, `array`, `object`. 생략 시 `default`에서 추론 |
| `description` | 선택 | 구성 속성 설명 |
| `secret` | 선택 | 비밀값 여부(Boolean) |
| `default` | 선택 | 기본값. 지정한 타입과 일치해야 함 |
| `items` | 조건부 | `type`이 `array`일 때 필수. 배열 항목의 타입 선언 |

### `requiredPulumiVersion`

시맨틱 버전 범위 구문을 사용하여 필요한 CLI 버전을 지정합니다.

```yaml
# 예시
requiredPulumiVersion: ">=3.0.0"          # 3.0.0 이상
requiredPulumiVersion: "!3.1.2"           # 3.1.2 제외 모든 버전
requiredPulumiVersion: ">=3.5.0 !3.7.7"   # 3.5.0 이상이되 3.7.7은 제외 (AND 결합)
requiredPulumiVersion: "<3.4.0 || >3.8.0" # 3.4.0 미만 또는 3.8.0 초과 (OR 결합)
```

### 전체 속성 예시

```yaml
name: Example Pulumi project file with all possible attributes
runtime:
  name: yaml
  options:
    compiler: cue export
description: An example project with all attributes
author: Your Name
website: https://example.com
license: Apache-2.0
main: example-project/
stackConfigDir: config/
backend:
  url: https://pulumi.example.com
options:
  refresh: always
template:
  displayName: Example Template
  description: An example template
  quickstart: Run npm install to install dependencies, then pulumi up to deploy.
  important: false
  config:
    aws:region:
      description: The AWS region to deploy into
      default: us-east-1
      secret: true
  metadata:
    cloud: aws
plugins:
  providers:
    - name: aws
      path: ../../bin
  languages:
    - name: yaml
      path: ../../../pulumi-yaml/bin
      version: 1.2.3
requiredPulumiVersion: ">=3.0.0"
```

---

## 엔트리포인트 규칙

각 언어는 프로그램 엔트리포인트를 찾는 규칙이 다릅니다.

| 언어 | 엔트리포인트 규칙 |
|------|-----------------|
| TypeScript | `package.json`이 가리키는 파일(예: `index.ts`). `main` 속성으로 `.ts`/`.js` 파일 직접 지정 가능 |
| Python | `__main__.py` 또는 `setup.py` 존재 여부. `main`으로 모듈 파일(예: `example.py`) 지정 가능 |
| Go | Go 빌드 도구 규칙. `main`으로 `.go` 파일의 디렉터리 지정 가능 |
| .NET | .NET 빌드 도구 규칙. `main`으로 `.csproj` 파일 지정 시 `dotnet run`으로 실행 |
| Java | Java 빌드 도구 규칙 |
| YAML | `Pulumi.yaml` 자체 또는 `main`으로 지정한 다른 YAML 파일(`Main.yaml`) |

`main` 속성은 모든 언어에서 디렉터리를 가리켜 프로그램 로드 위치를 변경할 수 있습니다. `Pulumi.yaml`과 `package.json`(Node.js)에 모두 `main`이 있으면 `Pulumi.yaml` 값이 우선합니다.

---

## 스택(Stack)

모든 Pulumi 프로그램은 **스택**에 배포됩니다. 스택은 Pulumi 프로그램의 격리된 독립 구성 인스턴스입니다. 주로 개발 환경(development, staging, production)이나 기능 브랜치(feature-x-dev)를 구분하는 데 사용됩니다.

프로젝트에 필요한 만큼 스택을 생성할 수 있습니다. `pulumi new` 명령으로 새 프로젝트를 시작하면 기본적으로 스택이 하나 생성됩니다.

---

## 스택 이름 형식

스택 이름은 프로젝트 내에서 고유해야 합니다. 알파벳, 하이픈, 밑줄, 마침표만 사용할 수 있습니다.

스택 이름은 다음 세 가지 형식 중 하나로 지정합니다.

| 형식 | 설명 |
|------|------|
| `stackName` | 현재 사용자 계정 또는 기본 조직의 스택. 프로젝트는 가장 가까운 `Pulumi.yaml`로 결정 |
| `orgName/stackName` | 조직 `orgName`의 스택. 프로젝트는 가장 가까운 `Pulumi.yaml`로 결정 |
| `orgName/projectName/stackName` | 조직 `orgName`의 프로젝트 `projectName`에 속한 스택. `projectName`은 `Pulumi.yaml`의 name과 일치해야 함 |

> **참고:** DIY 백엔드를 사용하는 경우 `orgName`은 항상 상수 값 `organization`이어야 합니다.

동일한 스택을 여러 형식으로 참조할 수 있습니다. 현재 조직이 `my-org`이고 현재 프로젝트가 `my-project`인 경우 다음은 모두 동일합니다.

```
my-org/my-project/dev
my-org/dev
dev
```

---

## 스택 관리 명령어

### 스택 생성

```bash
$ pulumi stack init staging
```

빈 스택을 생성하고 활성 스택으로 설정합니다.

> **참고:** `pulumi stack init`는 `Pulumi.<stack-name>.yaml` 파일을 생성하지 않습니다. 스택 구성 파일은 `pulumi config` 명령으로 생성 및 관리됩니다.

### 스택 목록 조회

```bash
$ pulumi stack ls
NAME                                      LAST UPDATE              RESOURCE COUNT
jane-dev                                  4 hours ago              97
staging*                                  n/a                      n/a
broomellc/test                            2 weeks ago              121
```

`*` 표시가 현재 활성 스택을 나타냅니다. 조직이나 프로젝트가 다른 스택은 부분 정규화된 이름으로 표시됩니다.

### 스택 선택

`pulumi config`, `pulumi preview`, `pulumi up`, `pulumi destroy`는 모두 활성 스택에서 동작합니다.

```bash
# 개인 스택 선택
$ pulumi stack select jane-dev

# 조직 스택 선택
$ pulumi stack select mycompany/prod
```

### 스택 이름 변경

```bash
# 단순 이름 변경
$ pulumi stack rename production

# 정규화된 이름으로 변경
$ pulumi stack rename myorg/myproject/production
```

> **주의:** 스택 이름 변경 시 프로그램에서 반환되는 스택 이름 값이 변경됩니다(예: TypeScript `pulumi.getStack()`, Python `pulumi.get_stack()`). 이 값으로 리소스 이름을 지정한 경우, 다음 `pulumi up`에서 리소스 교체가 시도됩니다.

### 스택 갱신(Update)

```bash
$ pulumi up
```

현재 선택된 스택을 업데이트합니다. 미리보기에서 저장한 계획이 있으면 `--plan` 옵션으로 해당 계획만 실행하도록 제한할 수 있습니다.

```bash
$ pulumi up --plan=plan.json
```

### 스택 내보내기/가져오기(Export/Import)

스택의 원시 데이터를 내보내고 가져올 수 있습니다. Pulumi가 인식하지 못하는 클라우드 플랫폼 변경을 수동 적용해야 할 때 유용합니다.

```bash
# 스택 상태를 파일로 내보내기
$ pulumi stack export --file stack.json

# 수정한 스택 상태 가져오기
$ pulumi stack import --file stack.json
```

> **주의:** 이 기능은 Pulumi의 일반적인 리소스 관리 방식을 우회합니다. 잘못된 스택 사양을 가져오면 클라우드 리소스가 고아 상태가 되거나 이후 업데이트가 불가능해질 수 있습니다.

### 스택 파괴(Destroy)

스택에 연결된 리소스가 남아 있는 경우, 스택 삭제 전에 먼저 리소스를 삭제해야 합니다.

```bash
$ pulumi destroy
```

이 명령은 마지막 배포 시 사용된 구성이 아닌 최신 구성 값을 사용합니다.

### 스택 삭제(Delete)

리소스가 없는 스택을 삭제합니다. 스택의 모든 기록이 제거되고 `Pulumi.<stack-name>.yaml` 구성 파일도 삭제됩니다.

```bash
# 빈 스택 삭제
$ pulumi stack rm

# 리소스가 있는 스택 강제 삭제(리소스가 고아 상태가 될 수 있음)
$ pulumi stack rm --force
```

> **참고:** 실수로 삭제한 스택은 조직 관리자가 Pulumi Cloud 콘솔에서 제한된 시간 내에 복원할 수 있습니다.

---

## 스택 명령어 요약

| 명령어 | 설명 |
|--------|------|
| `pulumi stack init <name>` | 새 스택 생성 및 활성 스택으로 설정 |
| `pulumi stack ls` | 현재 프로젝트의 스택 목록 조회 |
| `pulumi stack select <name>` | 활성 스택 변경 |
| `pulumi stack rename <new-name>` | 스택 이름 변경 |
| `pulumi stack` | 현재 스택의 메타데이터·리소스·출력 조회 |
| `pulumi stack output <name>` | 특정 스택 출력값 조회 |
| `pulumi stack output --json` | 전체 출력값 JSON 형식 조회 |
| `pulumi stack output --show-secrets` | 비밀 출력값 평문 조회 |
| `pulumi stack export --file <path>` | 스택 상태를 파일로 내보내기 |
| `pulumi stack import --file <path>` | 파일에서 스택 상태 가져오기 |
| `pulumi stack tag ls` | 스택 태그 목록 조회 |
| `pulumi stack tag set <name> <value>` | 커스텀 태그 설정 |
| `pulumi stack tag rm <name>` | 태그 삭제 |
| `pulumi destroy` | 스택의 모든 리소스 삭제 |
| `pulumi stack rm` | 빈 스택 삭제(구성 파일 포함) |
| `pulumi stack rm --force` | 리소스가 있는 스택 강제 삭제 |

---

## 스택 구성 파일(Pulumi.\<stack-name\>.yaml)

`pulumi stack init`로 스택을 생성할 때 구성 파일은 자동 생성되지 않습니다. 구성은 `pulumi config` 명령으로 관리합니다.

```bash
# 구성 값 설정
$ pulumi config set aws:region us-west-2

# 비밀 구성 값 설정
$ pulumi config set --secret db-password <YOUR_VALUE>
```

구성 키는 프로젝트 이름으로 범위가 지정됩니다. 예를 들어 `database`라는 키는 `my-project:database`로 저장됩니다.

---

## 스택 출력(Outputs)

스택은 값을 출력으로 내보낼 수 있습니다. 출력은 업데이트 중 표시되고, CLI에서 쉽게 조회할 수 있으며, Pulumi Cloud에도 표시됩니다.

### 출력 정의

```typescript
// TypeScript
export const url = resource.url;
```

```python
# Python
pulumi.export("url", resource.url)
```

### 출력 조회

```bash
# 단일 출력값
$ pulumi stack output url
hello

# 복합 출력값
$ pulumi stack output o
{"num": 42}

# 전체 출력(JSON)
$ pulumi stack output --json
{
  "x": "hello",
  "o": {
      "num": 42
  }
}
```

출력값은 JSON 직렬화되며, 문자열 출력 시 따옴표는 제거됩니다. 비밀값이 포함된 경우 기본적으로 평문이 표시되지 않으며 `--show-secrets` 플래그로 확인할 수 있습니다.

---

## 스택 태그

스택에는 메타데이터 태그가 연결됩니다. `pulumi:project`, `pulumi:runtime`, `pulumi:description`, `gitHub:owner`, `gitHub:repo`, `vcs:owner`, `vcs:repo`, `vcs:kind` 등의 내장 태그가 업데이트 시 자동 할당됩니다.

커스텀 태그를 사용하면 Pulumi Cloud에서 스택을 환경별로 그룹화할 수 있습니다.

```bash
# 환경 태그 설정
$ pulumi stack tag set environment production

# 태그 조회
$ pulumi stack tag ls

# 태그 삭제
$ pulumi stack tag rm environment
```

> **참고:** 스택 태그는 Pulumi Cloud 백엔드에서만 지원됩니다. 커스텀 태그에 `pulumi:`, `gitHub:`, `vcs:` 접두어를 사용하면 내장 태그와 충돌할 수 있습니다.

---

## 현재 스택 이름 프로그래밍 방식 조회

프로그램 내에서 현재 스택 이름을 가져올 수 있습니다. 리소스 이름 지정, 태깅, 리소스 접근에 유용합니다.

```typescript
// TypeScript
let stack = pulumi.getStack();
```

```python
# Python
stack = pulumi.get_stack()
```

---

## 스택 참조(StackReference)

스택 참조를 사용하면 한 스택에서 다른 스택의 출력값에 접근할 수 있습니다. 인프라 프로젝트와 서비스 프로젝트를 분리하는 등 멀티스택 아키텍처에서 핵심 기능입니다.

### 기본 사용법

```typescript
// TypeScript
import * as pulumi from "@pulumi/pulumi";
const other = new pulumi.StackReference("acmecorp/infra/other");
const otherOutput = other.getOutput("x");
```

```python
# Python
from pulumi import StackReference

other = StackReference("acmecorp/infra/other")
other_output = other.get_output("x")
```

스택 참조 이름은 `<organization>/<project>/<stack>` 형식의 정규화된 이름이어야 합니다.

### 출력 읽기 메서드

| 메서드 | 설명 |
|--------|------|
| `requireOutput` | 지정된 출력값을 `Output`으로 반환. 출력이 존재하지 않으면 배포 시 실패. **권장 메서드** |
| `getOutput` | 지정된 출력값을 `Output`으로 반환. 출력이 없으면 `undefined`/`None` 래핑. 출력 부재가 예상된 경우에만 사용 |
| `getOutputDetails` | `OutputDetails` 객체를 반환하여 `Output` 래핑을 우회. 비밀 여부 확인이나 프로그램 로직에서 원시값 필요 시 사용 |

### 멀티스택 워크플로우 예시

인프라 프로젝트(`infra`)가 Kubernetes 클러스터를 정의하고 서비스 프로젝트(`services`)가 해당 클러스터에 서비스를 배포하는 시나리오:

```
mycompany/infra/production  <-- mycompany/services/production
mycompany/infra/staging     <-- mycompany/services/staging
mycompany/infra/testing     <-- mycompany/services/testing
```

인프라 스택에서 kubeConfig를 출력하고, 서비스 스택에서 StackReference로 참조합니다.

```typescript
// TypeScript - services 프로젝트
import * as k8s from "@pulumi/kubernetes";
import * as pulumi from "@pulumi/pulumi";
const env = pulumi.getStack();
const infra = new pulumi.StackReference(`mycompany/infra/${env}`);
const provider = new k8s.Provider("k8s", { kubeconfig: infra.getOutput("kubeConfig") });
const service = new k8s.core.v1.Service(..., { provider: provider });
```

```python
# Python - services 프로젝트
from pulumi import get_stack, ResourceOptions, StackReference
from pulumi_kubernetes import Provider, core

env = get_stack()
infra = StackReference(f"mycompany/infra/{env}")
provider = Provider("k8s", kubeconfig=infra.get_output("kubeConfig"))
service = core.v1.Service(..., ResourceOptions(provider=provider))
```

---

## 조직 구조

Pulumi의 스택 네이밍은 `orgName/projectName/stackName` 계층 구조를 따릅니다.

```
mycompany/                   # 조직(organization)
  ├── infra/                 # 프로젝트(project)
  │   ├── production         # 스택(stack)
  │   ├── staging
  │   └── testing
  └── services/              # 프로젝트(project)
      ├── production         # 스택(stack)
      ├── staging
      └── testing
```

### 스택 이름 변경 시 주의사항(프로젝트 간)

스택 이름을 다른 프로젝트로 변경할 경우(예: `myorg/old-project/prod` -> `myorg/new-project/prod`) 두 가지 추가 수동 작업이 필요합니다.

1. **`Pulumi.yaml`의 `name` 필드 업데이트** - `name` 필드가 정규화된 스택 이름의 프로젝트 부분과 일치해야 합니다.

```yaml
name: new-project  # was: old-project
runtime: nodejs
```

2. **프로젝트 네임스페이스 구성 키 업데이트** - 구성 키는 프로젝트 이름으로 범위가 지정됩니다. `Pulumi.<stack>.yaml`에서 이전 프로젝트 접두어를 새 프로젝트로 변경해야 합니다.

```yaml
# 변경 전
config:
  old-project:database: mydb
  old-project:region: us-west-2
  aws:region: us-west-2

# 변경 후
config:
  new-project:database: mydb
  new-project:region: us-west-2
  aws:region: us-west-2
```

> **경고:** 프로젝트 간 이름 변경 후 구성 키를 업데이트하지 않으면, 프로그램이 해당 구성값을 설정되지 않은 것으로 조용히 간주하여 런타임 오류나 예상치 못한 기본값 사용이 발생할 수 있습니다.

---

## 파일 레퍼런스 요약

| 파일 | 위치 | 용도 |
|------|------|------|
| `Pulumi.yaml` | 프로젝트 루트 | 프로젝트 메타데이터·런타임·구성 정의(필수) |
| `Pulumi.<stack-name>.yaml` | 프로젝트 루트 | 스택별 구성 값. `pulumi config`로 관리 |
| `Pulumi.README.md` | 프로젝트 루트(선택) | Pulumi Cloud 스택 README 템플릿 |
| `package.json` | 프로젝트 루트(Node.js) | 엔트리포인트, 의존성 정의 |
| `__main__.py` / `setup.py` | 프로젝트 루트(Python) | Python 엔트리포인트 |
