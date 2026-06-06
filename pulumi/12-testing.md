# Pulumi 테스팅

> https://www.pulumi.com/docs/iac/guides/testing/
> https://www.pulumi.com/docs/iac/guides/testing/unit/
> https://www.pulumi.com/docs/iac/guides/testing/integration/
> https://www.pulumi.com/docs/iac/guides/testing/integration/framework/
> https://www.pulumi.com/docs/iac/guides/testing/integration/automation-api/

Pulumi는 범용 프로그래밍 언어로 클라우드 리소스를 프로비저닝하므로, 네이티브 테스트 프레임워크를 활용하여 인프라에 대한 자동화 테스트를 수행할 수 있다. Pulumi는 세 가지 테스트 스타일을 제공한다: 모든 외부 호출을 모킹하는 **단위 테스트(Unit Tests)**, 인프라 배포 중에 리소스 수준의 단언을 실행하는 **속성 테스트(Property Tests)**, 임시 인프라를 배포하고 외부 테스트를 수행하는 **통합 테스트(Integration Tests)**.

---

## 테스트 종류 비교

| 항목 | 단위 테스트 (Unit Tests) | 속성 테스트 (Property Tests) | 통합 테스트 (Integration Tests) |
|------|-------------------------|------------------------------|--------------------------------|
| 실제 인프라 프로비저닝 | 아니오 | 예 | 예 |
| Pulumi CLI 필요 | 아니오 | 예 | 예 |
| 실행 시간 | 밀리초 | 초 | 분 |
| 사용 언어 | Pulumi 프로그램과 동일 | Node.js 또는 Python | 모든 언어 |
| 검증 대상 | 리소스 입력값(Inputs) | 리소스 입력/출력값 | 외부 엔드포인트 |
| 모킹 필요 | 예 | 아니오 | 아니오 |
| 접근 방식 | 화이트박스 | 인프라 배포 중 검사 | 블랙박스 |

---

## 단위 테스트 (Unit Testing)

> https://www.pulumi.com/docs/iac/guides/testing/unit/

단위 테스트는 코드의 동작을 격리된 환경에서 평가하며, 모든 외부 의존성을 **모킹(Mock)**으로 대체한다. 단위 테스트는 메모리 내에서 실행되며 프로세스 외부 호출이 없으므로 매우 빠르다. TDD(테스트 주도 개발)를 포함한 빠른 피드백 루프에 적합하다.

단위 테스트는 테스트 대상 Pulumi 프로그램과 동일한 언어로 작성된다. 클라우드 리소스가 실제로 생성되지 않으므로, 엔드포인트에 HTTP 요청을 보내는 등 인프라의 실제 동작을 평가하는 테스트는 작성할 수 없다.

### 작동 원리

Pulumi 프로그램이 업데이트를 실행할 때 Pulumi CLI와 통신하여 배포를 오케스트레이션한다. 단위 테스트는 이 통신 채널을 차단하고 엔진을 모킹으로 대체한다. 모킹은 동일한 OS 프로세스 내에서 Pulumi 프로그램의 호출에 응답하고 더미 데이터를 반환한다.

### 테스트 프레임워크 설치

| 언어 | 프레임워크 | 설치 명령어 |
|------|-----------|------------|
| TypeScript | Mocha | `npm install mocha @types/mocha ts-node --global --save-dev` |
| Python | unittest (내장) | 별도 설치 불필요 |
| Go | go test (내장) | 별도 설치 불필요 |
| C# | NUnit + Moq + FluentAssertions | `dotnet add package NUnit` 등 |
| Java | JUnit 5 | pom.xml에 의존성 추가 |
| YAML | 해당 없음 | YAML은 선언형이므로 모킹 기반 단위 테스트 미지원 |

### 모킹 설정

모킹은 `newResource`와 `call` 두 가지 핸들러를 구현해야 한다.

**TypeScript 모킹 예제:**

```typescript
import * as pulumi from "@pulumi/pulumi";

pulumi.runtime.setMocks({
    newResource: function(args: pulumi.runtime.MockResourceArgs): {id: string, state: any} {
        switch (args.type) {
            case "aws:ec2/securityGroup:SecurityGroup":
                return {
                    id: "sg-12345678",
                    state: {
                        ...args.inputs,
                        arn: "arn:aws:ec2:us-west-2:123456789012:security-group/sg-12345678",
                        name: args.inputs.name || args.name + "-sg",
                    },
                };
            case "aws:ec2/instance:Instance":
                return {
                    id: "i-1234567890abcdef0",
                    state: {
                        ...args.inputs,
                        publicIp: "203.0.113.12",
                        instanceState: "running",
                    },
                };
            default:
                return {
                    id: args.inputs.name + "_id",
                    state: { ...args.inputs },
                };
        }
    },
    call: function(args: pulumi.runtime.MockCallArgs) {
        switch (args.token) {
            case "aws:ec2/getAmi:getAmi":
                return { "architecture": "x86_64", "id": "ami-0eb1f3cdeeb8eed2a" };
            default:
                return args.inputs;
        }
    },
}, "project", "stack", false);
```

**Python 모킹 예제:**

```python
import pulumi

class MyMocks(pulumi.runtime.Mocks):
    def new_resource(self, args: pulumi.runtime.MockResourceArgs):
        return [args.name + '_id', args.inputs]
    def call(self, args: pulumi.runtime.MockCallArgs):
        return {}

pulumi.runtime.set_mocks(
    MyMocks(),
    preview=False,
)
```

> **주의:** Python에서 `new_resource`에서 명시적 출력 속성을 반환할 때, 속성 이름은 camelCase(예: `"publicIp"`)를 사용해야 한다. Pulumi는 프로그래밍 언어와 관계없이 내부 속성 직렬화에 camelCase를 사용하기 때문이다. `"public_ip"`가 아닌 `"publicIp"`를 사용해야 한다.

### 입력 속성 vs 출력 속성

모킹 구현 시 두 가지 속성 유형을 구분하는 것이 중요하다.

| 속성 유형 | 설명 | 모킹에서의 처리 |
|-----------|------|----------------|
| 입력 속성 (Input Properties) | 코드에서 설정하는 값 (`tags`, `userData`, `ingress` 등) | `args.inputs`를 통해 자동 전달됨 |
| 출력 속성 (Output Properties) | 클라우드 제공자가 계산하는 값 (`arn`, `publicIp`, `instanceState` 등) | 모킹에서 명시적으로 반환해야 함 |

### StackReference 모킹

프로그램이 `StackReference`를 사용하는 경우, 모킹에서 `pulumi:pulumi:StackReference` 타입을 처리해야 한다.

**TypeScript 예제:**

```typescript
pulumi.runtime.setMocks({
    newResource: function(args: pulumi.runtime.MockResourceArgs): {id: string, state: any} {
        if (args.type === "pulumi:pulumi:StackReference") {
            return {
                id: args.inputs.name + "_id",
                state: {
                    ...args.inputs,
                    outputs: {
                        vpcId: "vpc-12345678",
                        subnetIds: ["subnet-11111111", "subnet-22222222"],
                        clusterName: "my-cluster",
                    },
                },
            };
        }
        return { id: args.inputs.name + "_id", state: args.inputs };
    },
    call: function(args: pulumi.runtime.MockCallArgs) { return args.inputs; },
});
```

**Python 예제:**

```python
class MyMocks(pulumi.runtime.Mocks):
    def new_resource(self, args: pulumi.runtime.MockResourceArgs):
        if args.typ == "pulumi:pulumi:StackReference":
            return [args.name + "_id", {
                **args.inputs,
                "outputs": {
                    "vpcId": "vpc-12345678",
                    "subnetIds": ["subnet-11111111", "subnet-22222222"],
                    "clusterName": "my-cluster",
                },
            }]
        return [args.name + "_id", args.inputs]
    def call(self, args: pulumi.runtime.MockCallArgs):
        return {}
```

### 테스트 작성

모킹 정의 후 프로그램을 임포트하여 리소스 속성을 검사하는 테스트를 작성한다. 모든 Pulumi 리소스 속성은 `Output`이므로 `apply` 메서드를 사용해 값에 접근해야 한다.

**TypeScript 테스트 예제:**

```typescript
import * as pulumi from "@pulumi/pulumi";
import "mocha";

// 모킹 설정은 프로그램 임포트 전에 수행
pulumi.runtime.setMocks({ /* ... */ });

describe("Infrastructure", function() {
    let infra: typeof import("./index");

    before(async function() {
        infra = await import("./index");
    });

    // 검사 1: 인스턴스에 Name 태그가 있어야 함
    it("must have a name tag", function(done) {
        pulumi.all([infra.server.urn, infra.server.tags]).apply(([urn, tags]) => {
            if (!tags || !tags["Name"]) {
                done(new Error(`Missing a name tag on server ${urn}`));
            } else {
                done();
            }
        });
    });

    // 검사 2: userData 스크립트를 사용하지 않아야 함
    it("must not use userData (use an AMI instead)", function(done) {
        pulumi.all([infra.server.urn, infra.server.userData]).apply(([urn, userData]) => {
            if (userData) {
                done(new Error(`Illegal use of userData on server ${urn}`));
            } else {
                done();
            }
        });
    });

    // 검사 3: SSH가 인터넷에 열려 있지 않아야 함
    it("must not open port 22 (SSH) to the Internet", function(done) {
        pulumi.all([infra.group.urn, infra.group.ingress]).apply(([urn, ingress]) => {
            if (ingress.find(rule =>
                rule.fromPort === 22 && (rule.cidrBlocks || []).find(block => block === "0.0.0.0/0"))) {
                done(new Error(`Illegal SSH port 22 open to the Internet on group ${urn}`));
            } else {
                done();
            }
        });
    });
});
```

**Python 테스트 예제:**

```python
import unittest
import pulumi

# 모킹 설정 후 infra 임포트
pulumi.runtime.set_mocks(MyMocks())
import infra

class TestingWithMocks(unittest.TestCase):

    @pulumi.runtime.test
    def test_server_tags(self):
        def check_tags(args):
            urn, tags = args
            self.assertIsNotNone(tags, f'server {urn} must have tags')
            self.assertIn('Name', tags, f'server {urn} must have a name tag')
        return pulumi.Output.all(infra.server.urn, infra.server.tags).apply(check_tags)

    @pulumi.runtime.test
    def test_server_userdata(self):
        def check_user_data(args):
            urn, user_data = args
            self.assertFalse(user_data, f'illegal use of user_data on server {urn}')
        return pulumi.Output.all(infra.server.urn, infra.server.user_data).apply(check_user_data)

    @pulumi.runtime.test
    def test_security_group_rules(self):
        def check_security_group_rules(args):
            urn, ingress = args
            ssh_open = any([
                rule['from_port'] == 22 and any([block == "0.0.0.0/0" for block in rule['cidr_blocks']])
                for rule in ingress
            ])
            self.assertFalse(ssh_open, f'security group {urn} exposes port 22 to the Internet')
        return pulumi.Output.all(infra.group.urn, infra.group.ingress).apply(check_security_group_rules)
```

### 테스트 실행 명령어

| 언어 | 명령어 |
|------|--------|
| TypeScript | `mocha -r ts-node/register ec2tests.ts` |
| Python | `python -m unittest` |
| Go | `go test` |
| C# | `dotnet test` |
| Java | `mvn test` |

### 단위 테스트 핵심 포인트

- 모킹은 프로그램 임포트 **이전**에 설정해야 한다.
- 클라우드 제공자가 계산하는 출력 속성은 모킹에서 명시적으로 반환해야 한다.
- Pulumi 리소스 속성은 모두 `Output`이므로 `apply` 메서드로 값에 접근한다.
- 출력은 비동기적으로 해결되므로 프레임워크의 비동기 테스트 기능을 사용해야 한다.

---

## 통합 테스트 (Integration Testing)

> https://www.pulumi.com/docs/iac/guides/testing/integration/

통합 테스트는 Pulumi 프로그램을 실행하고 내부 구현 세부 사항을 검사하지 않고 결과를 테스트하는 방식이다. 예를 들어, HTTP 엔드포인트가 사용 가능하고 들어오는 요청에 예상된 응답을 주는지 테스트할 수 있다. 이 접근 방식은 "블랙박스 테스트"로도 알려져 있다.

### 통합 테스트가 보장하는 사항

| 검증 항목 | 설명 |
|-----------|------|
| 코드 구문 검사 | 프로젝트 코드가 구문적으로 올바르고 오류 없이 실행됨 |
| 구성 검증 | 스택의 구성(Config)과 시크릿(Secrets)이 올바르게 해석됨 |
| 배포 성공 | 선택한 클라우드 제공자에 성공적으로 배포 가능함 |
| 리소스 형태 | 원하는 형태의 리소스가 성공적으로 생성됨 |
| 인프라 동작 | 헬스체크 엔드포인트가 유효한 HTML을 반환하는 등 인프라가 예상대로 동작함 |
| 업데이트 성공 | 시작 상태에서 다른 상태로 성공적으로 업데이트 가능함 |
| 삭제 성공 | 클라우드 제공자에서 성공적으로 삭제 및 제거 가능함 |

### 통합 테스트 접근 방식 비교

| 항목 | Integration Testing Framework | Automation API | DIY (셸 스크립트) |
|------|-------------------------------|----------------|-------------------|
| 테스트 언어 | Go만 가능 | Node.js, Python, .NET, Go, Java | 모든 언어 |
| YAML 지원 | 예 | 아니오 | 예 |
| 목적 | Pulumi 프로그램 테스트 전용 | 범용 프로그래밍 인터페이스 | 범용 |
| 생명주기 테스트 | 배포, 업데이트, 삭제 전체 | `up()`, `destroy()` 직접 제어 | 수동 |
| 타입 안전성 | 높음 | 높음 | 낮음 |
| 추가 의존성 | Go 패키지 | Pulumi SDK | Pulumi CLI만 |
| 업데이트 경로 테스트 | `EditDirs`로 시퀀스 테스트 | 직접 구현 필요 | 직접 구현 필요 |

---

## Integration Testing Framework

> https://www.pulumi.com/docs/iac/guides/testing/integration/framework/

Pulumi는 Go로 작성된 전용 통합 테스트 프레임워크를 제공한다. 이 프레임워크는 Pulumi CLI 및 프로바이더의 핵심 기능을 내부적으로 검증하는 데 사용된다. 테스트 대상 Pulumi 프로그램의 언어에 관계없이 Go로 테스트를 작성할 수 있다.

### 기본 통합 테스트

```go
package test

import (
    "os"
    "path"
    "testing"

    "github.com/pulumi/pulumi/pkg/v3/testing/integration"
)

func TestExamples(t *testing.T) {
    awsRegion := os.Getenv("AWS_REGION")
    if awsRegion == "" {
        awsRegion = "us-west-1"
    }
    cwd, _ := os.Getwd()
    integration.ProgramTest(t, &integration.ProgramTestOptions{
        Quick:       true,
        SkipRefresh: true,
        Dir:         path.Join(cwd, "..", "..", "aws-js-s3-folder"),
        Config: map[string]string{
            "aws:region": awsRegion,
        },
    })
}
```

이 테스트는 스택 생성, 업데이트, 삭제의 기본 생명주기를 실행하며 약 1분 정도 소요된다.

### 주요 구성 옵션 (ProgramTestOptions)

| 옵션 | 설명 |
|------|------|
| `Quick` | 빠른 테스트 모드 활성화 |
| `SkipRefresh` | 새로고침 단계 건너뛰기 |
| `Dir` | 테스트할 Pulumi 프로그램 디렉터리 경로 |
| `Config` | 스택 구성 맵 (`map[string]string`) |
| `Tracing` | Jaeger 엔드포인트 구성 |
| `ExpectFailure` | 부정 테스트(Negative Testing)를 위한 실패 예상 플래그 |
| `EditDirs` | 업데이트 상태 전환 시퀀스를 위한 프로그램 수정 목록 |
| `ExtraRuntimeValidation` | 배포 후 추가 검증 콜백 |

전체 옵션은 [`ProgramTestOptions` 데이터 구조](https://pkg.go.dev/github.com/pulumi/pulumi/pkg/v3/testing/integration#ProgramTestOptions)에서 확인할 수 있다.

### 리소스 속성 검증

`ExtraRuntimeValidation` 옵션으로 배포 후 상태를 검사할 수 있다. Pulumi가 캡처한 스택 상태의 전체 스냅샷에 접근할 수 있다.

**S3 버킷 개수 검증 예제:**

```go
integration.ProgramTest(t, &integration.ProgramTestOptions{
    // 기본 옵션 ...
    ExtraRuntimeValidation: func(t *testing.T, stack integration.RuntimeValidationStackInfo) {
        var foundBuckets int
        for _, res := range stack.Deployment.Resources {
            if res.Type == "aws:s3/bucket:Bucket" {
                foundBuckets++
            }
        }
        assert.Equal(t, 1, foundBuckets, "Expected to find a single AWS S3 Bucket")
    },
})
```

### 스택 출력값을 활용한 런타임 검증

`ExtraRuntimeValidation` 콜백에서 스택 출력값을 사용하여 프로비저닝된 인프라가 실제로 작동하는지 검증할 수 있다. VM IP 주소, URL 등 인프라 정보에 접근할 수 있다.

**웹사이트 엔드포인트 검증 예제:**

```go
integration.ProgramTest(t, &integration.ProgramTestOptions{
    // 기본 옵션 ...
    ExtraRuntimeValidation: func(t *testing.T, stack integration.RuntimeValidationStackInfo) {
        url := "http://" + stack.Outputs["websiteUrl"].(string)
        resp, err := http.Get(url)
        if !assert.NoError(t, err) {
            return
        }
        if !assert.Equal(t, 200, resp.StatusCode) {
            return
        }
        defer resp.Body.Close()
        body, err := ioutil.ReadAll(resp.Body)
        if !assert.NoError(t, err) {
            return
        }
        assert.Contains(t, string(body), "Hello, Pulumi!")
    },
})
```

---

## Automation API를 활용한 통합 테스트

> https://www.pulumi.com/docs/iac/guides/testing/integration/automation-api/

Automation API는 통합 테스트를 위한 프로그래밍 인터페이스를 제공한다. 전용 테스트 프레임워크는 아니지만, 동일한 기본 정확성 테스트, 리소스 검증, 런타임 테스트 목적을 달성할 수 있다. YAML을 제외한 모든 Pulumi 언어에서 사용할 수 있다.

### 테스트 워크플로우

1. Automation API로 스택 생성
2. 스택 구성(Config) 및 시크릿(Secrets) 설정
3. `up()`으로 스택 배포
4. 배포 상태를 검사하여 리소스 검증
5. 배포된 인프라와 상호작용하여 런타임 검사
6. `destroy()`로 스택 삭제 및 리소스 정리

### 리소스 및 스택 속성 검증

올바른 리소스가 예상된 속성으로 생성되었는지 확인하려면 스택을 내보내고 결과 리소스를 검사한다.

```typescript
export async function getDeployment(): Promise<Deployment> {
  const stack = await LocalWorkspace.createOrSelectStack(args);
  return stack.exportStack();
}
```

`Deployment` 객체를 순회하며 예상된 리소스와 속성 값이 있는지 확인할 수 있다.

### 런타임 검증

배포된 인프라와 상호작용하여 올바르게 작동하는지 검증한다. Automation API는 스택 출력에 대한 전체 접근 권한을 제공한다.

| 검증 유형 | 설명 |
|-----------|------|
| HTTP 요청 | API 엔드포인트 유효성 검사 |
| 데이터베이스 쿼리 | 데이터가 올바르게 생성되었는지 확인 |
| 오브젝트 스토리지 | 파일 존재 여부 확인 |
| VM 네트워크 | VM이 네트워크 요청에 응답하는지 확인 |

### 추가 리소스

Automation API로 작성된 통합 테스트 예제:

| 언어 | 저장소 경로 |
|------|------------|
| Node.js | [pulumi/sdk/nodejs/tests/automation/localWorkspace.spec.ts](https://github.com/pulumi/pulumi/blob/master/sdk/nodejs/tests/automation/localWorkspace.spec.ts) |
| Go | [pulumi/sdk/go/auto/local_workspace_test.go](https://github.com/pulumi/pulumi/blob/master/sdk/go/auto/local_workspace_test.go) |
| Python | [pulumi/sdk/python/lib/test/automation/test_local_workspace.py](https://github.com/pulumi/pulumi/blob/master/sdk/python/lib/test/automation/test_local_workspace.py) |
| C# | [pulumi-dotnet/sdk/Pulumi.Automation.Tests/LocalWorkspaceTests.cs](https://github.com/pulumi/pulumi-dotnet/blob/main/sdk/Pulumi.Automation.Tests/LocalWorkspaceTests.cs) |
| Java | [automation-api-examples/java/localProgram/](https://github.com/pulumi/automation-api-examples/blob/main/java/localProgram/) |

---

## 테스트 전략과 모범 사례

### 테스트 피라미드 적용

많은 팀에서 세 가지 테스트 접근 방식을 조합하여 사용하는 것이 합리적이다.

| 테스트 유형 | 역할 | 실행 빈도 |
|------------|------|----------|
| 단위 테스트 | 프로그램 로직을 빠르게 검증 | 모든 커밋 |
| 속성 테스트 | 핵심 정확성 불변식 검증 | PR/병합 시 |
| 통합 테스트 | 인프라 구성 요소의 종단 간 상호작용 테스트 | 주기적/배포 전 |

### 단위 테스트 모범 사례

- 모킹은 프로그램 임포트 **이전**에 설정해야 한다.
- 클라우드 제공자가 계산하는 출력 속성(output properties)은 모킹에서 명시적으로 반환해야 한다. 그렇지 않으면 테스트에서 `undefined`가 된다.
- Python에서 `new_resource`의 출력 속성 이름은 항상 camelCase를 사용해야 한다 (`public_ip`가 아닌 `publicIp`).
- Pulumi 리소스 속성은 모두 `Output`이므로 `apply` 메서드를 통해 값에 접근해야 한다.

### 통합 테스트 모범 사례

- 통합 테스트는 임시(ephemeral) 환경에 인프라를 배포하여 테스트 후 삭제해야 한다.
- 리소스 유형과 테스트 빈도에 따라 단기 임시 환경도 클라우드 제공자로부터 상당한 비용이 발생할 수 있으므로 적절히 계획해야 한다.
- `ExtraRuntimeValidation`을 사용하여 배포된 리소스의 실제 동작을 검증한다.
- `EditDirs`를 사용하면 업데이트/업그레이드 경로를 테스트할 수 있다.

### 언어별 테스트 지원 요약

| 언어 | 단위 테스트 | 속성 테스트 | Integration Framework | Automation API |
|------|-----------|-----------|----------------------|----------------|
| TypeScript | 예 | 예 | 예 (Go로 작성) | 예 |
| Python | 예 | 예 | 예 (Go로 작성) | 예 |
| Go | 예 | 예 | 예 | 예 |
| C# | 예 | 예 | 예 (Go로 작성) | 예 |
| Java | 예 | 아니오 | 예 (Go로 작성) | 예 |
| YAML | 아니오 | 예 | 예 (Go로 작성) | 아니오 |

### 예제 저장소

Pulumi 공식 예제 저장소에서 완전한 실행 가능한 테스트를 확인할 수 있다.

| 예제 | 설명 |
|------|------|
| [Unit Tests in TypeScript](https://github.com/pulumi/examples/tree/74db62a03d013c2854d2cf933c074ea0a3bbf69d/testing-unit-ts) | TypeScript 모킹 기반 단위 테스트 |
| [Unit Tests in Python](https://github.com/pulumi/examples/tree/74db62a03d013c2854d2cf933c074ea0a3bbf69d/testing-unit-py) | Python 모킹 기반 단위 테스트 |
| [Unit Tests in Go](https://github.com/pulumi/examples/tree/74db62a03d013c2854d2cf933c074ea0a3bbf69d/testing-unit-go) | Go 모킹 기반 단위 테스트 |
| [Unit Tests in C#](https://github.com/pulumi/examples/tree/74db62a03d013c2854d2cf933c074ea0a3bbf69d/testing-unit-cs) | C# 모킹 기반 단위 테스트 |
| [Property Testing in TypeScript](https://github.com/pulumi/examples/tree/74db62a03d013c2854d2cf933c074ea0a3bbf69d/testing-pac-ts) | Policy-as-Code 기반 속성 테스트 |
| [Integration Testing in Go](https://github.com/pulumi/examples/tree/31056c3480cc445e5d4d3a8a0a86977adce2bc5e/testing-integration) | Deploy-check-destroy 통합 테스트 |
