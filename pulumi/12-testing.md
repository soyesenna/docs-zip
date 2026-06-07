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

### 속성 테스트와 Policy as Code

속성 테스트(Property Tests)는 **Policy as Code** 기반으로 동작한다. Pulumi의 정책 팩(Policy Pack)은 TypeScript/JavaScript(Node.js)와 Python으로 작성할 수 있으며, 각 정책은 테스트가 평가하고 단언하는 **속성(property), 즉 불변식(invariant)**이 된다. 예를 들어 "모든 S3 버킷은 암호화되어야 한다", "보안 그룹은 22번 포트를 인터넷에 공개해서는 안 된다" 등의 규칙을 정책으로 정의하고, `pulumi up` 실행 시 자동으로 검사한다. 속성 테스트는 Pulumi CLI 내부에서 인프라 프로비저닝 전후에 실행되며, 단위 테스트와 달리 클라우드 제공자가 반환하는 실제 값을 평가할 수 있다. 모의(mock) 값이 아닌 실제 값을 검증할 수 있으므로 단위 테스트보다 높은 신뢰도를 제공한다. 속성 테스트는 모든 클라우드 환경에서 실행할 수 있다: 지속적인 "수용(acceptance)" 스택, 각 pull request마다 생성되는 임시 클라우드 환경, 또는 이들의 조합 등. 자세한 내용은 [Property Testing 가이드](https://www.pulumi.com/docs/iac/guides/testing/property-testing/) 및 [Policy as Code 가이드](https://www.pulumi.com/docs/insights/policy/policy-packs/authoring/)를 참조하라.

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
                        arn: "arn:aws:ec2:us-west-2:123456789012:instance/i-1234567890abcdef0",
                        instanceState: "running",
                        primaryNetworkInterfaceId: "eni-12345678",
                        privateDns: "ip-10-0-1-17.ec2.internal",
                        publicDns: "ec2-203-0-113-12.compute-1.amazonaws.com",
                        publicIp: "203.0.113.12",
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
// 인수 설명: "project" → 프로젝트 이름, "stack" → 스택 이름,
// false → dryRun 플래그 (pulumi가 preview 모드로 실행 중인지 여부)
```

모킹 인터페이스의 정의는 [runtime API 참조 페이지](https://www.pulumi.com/docs/reference/pkg/nodejs/pulumi/pulumi/runtime/#Mocks)에서 확인할 수 있다.

### 모킹의 제한 사항 (Limitations)

모킹 서버는 전체 Pulumi 엔진을 구현하지 않으므로 다음 기능이 모킹 기반 단위 테스트에서는 실행되지 않는다.

- **Lifecycle Hooks**: 리소스의 `onCreate`, `onUpdate`, `onDelete` 등 생명주기 훅이 실행되지 않는다.
- **Resource Transforms**: 리소스 변환(transform)이 적용되지 않는다.

이러한 제한을 회피하려면 다음 방법을 고려할 수 있다.

| 회피 방법 | 설명 |
|-----------|------|
| 로직 분리 | lifecycle hook이나 transform 내부의 로직을 별도 함수로 분리하여 독립적으로 단위 테스트 |
| 모킹에서 기대 결과 반환 | hook이나 transform이 처리했을 것으로 예상되는 결과를 모킹에서 직접 반환 |
| 통합 테스트 사용 | lifecycle hook과 resource transform의 실제 동작은 통합 테스트로 검증 |

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

**Go 모킹 예제:**

```go
import (
    "github.com/pulumi/pulumi/sdk/v3/go/common/resource"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

type mocks int

func (mocks) NewResource(args pulumi.MockResourceArgs) (string, resource.PropertyMap, error) {
    return args.Name + "_id", args.Inputs, nil
}

func (mocks) Call(args pulumi.MockCallArgs) (resource.PropertyMap, error) {
    return args.Args, nil
}
```

**C# 모킹 예제:**

```csharp
using System.Collections.Immutable;
using System.Threading.Tasks;
using Pulumi;
using Pulumi.Testing;

namespace UnitTesting
{
    class Mocks : IMocks
    {
        public Task<(string? id, object state)> NewResourceAsync(MockResourceArgs args)
        {
            var outputs = ImmutableDictionary.CreateBuilder<string, object>();
            outputs.AddRange(args.Inputs);

            if (args.Type == "aws:ec2/instance:Instance")
            {
                outputs.Add("publicIp", "203.0.113.12");
                outputs.Add("publicDns", "ec2-203-0-113-12.compute-1.amazonaws.com");
            }

            args.Id ??= $"{args.Name}_id";
            return Task.FromResult<(string? id, object state)>((args.Id, (object)outputs));
        }

        public Task<object> CallAsync(MockCallArgs args)
        {
            var outputs = ImmutableDictionary.CreateBuilder<string, object>();

            if (args.Token == "aws:index/getAmi:getAmi")
            {
                outputs.Add("architecture", "x86_64");
                outputs.Add("id", "ami-0eb1f3cdeeb8eed2a");
            }

            return Task.FromResult((object)outputs);
        }
    }

    public static class Testing
    {
        public static Task<ImmutableArray<Resource>> RunAsync<T>() where T : Stack, new()
        {
            return Deployment.TestAsync<T>(new Mocks(), new TestOptions { IsPreview = false });
        }

        public static Task<T> GetValueAsync<T>(this Output<T> output)
        {
            var tcs = new TaskCompletionSource<T>();
            output.Apply(v =>
            {
                tcs.SetResult(v);
                return v;
            });
            return tcs.Task;
        }
    }
}
```

**Java 모킹 예제:**

```java
package myproject;

import com.pulumi.test.Mocks;
import com.pulumi.test.Mocks.CallArgs;
import com.pulumi.test.Mocks.ResourceArgs;
import com.pulumi.test.Mocks.ResourceResult;
import java.util.HashMap;
import java.util.Map;
import java.util.Optional;
import java.util.concurrent.CompletableFuture;

class MyMocks implements Mocks {
    @Override
    public CompletableFuture<ResourceResult> newResourceAsync(ResourceArgs args) {
        var state = new HashMap<>(args.inputs);
        return CompletableFuture.completedFuture(
            ResourceResult.of(Optional.of(args.name + "_id"), state)
        );
    }

    @Override
    public CompletableFuture<Map<String, Object>> callAsync(CallArgs args) {
        return CompletableFuture.completedFuture(Map.of());
    }
}
```

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

**Go 예제:**

```go
type mocks int

func (mocks) NewResource(args pulumi.MockResourceArgs) (string, resource.PropertyMap, error) {
    // Handle StackReference resources
    if args.TypeToken == "pulumi:pulumi:StackReference" {
        outputs := resource.NewPropertyMapFromMap(map[string]interface{}{
            "vpcId":       "vpc-12345678",
            "subnetIds":   []interface{}{"subnet-11111111", "subnet-22222222"},
            "clusterName": "my-cluster",
        })
        state := args.Inputs.Copy()
        state["outputs"] = resource.NewObjectProperty(outputs)
        return args.Name + "_id", state, nil
    }
    return args.Name + "_id", args.Inputs, nil
}

func (mocks) Call(args pulumi.MockCallArgs) (resource.PropertyMap, error) {
    return args.Args, nil
}
```

프로그램에서는 다음과 같이 `StackReference`를 사용할 수 있다:

```go
networkStack, err := pulumi.NewStackReference(ctx, "organization/network/prod", nil)
if err != nil {
    return err
}
vpcId := networkStack.GetStringOutput(pulumi.String("vpcId"))
// 테스트에서 vpcId는 "vpc-12345678"로 해결된다
```

**C# 예제:**

```csharp
class Mocks : IMocks
{
    public Task<(string? id, object state)> NewResourceAsync(MockResourceArgs args)
    {
        var outputs = ImmutableDictionary.CreateBuilder<string, object>();
        outputs.AddRange(args.Inputs);

        if (args.Type == "pulumi:pulumi:StackReference")
        {
            outputs.Add("outputs", new Dictionary<string, object>
            {
                { "vpcId", "vpc-12345678" },
                { "subnetIds", new[] { "subnet-11111111", "subnet-22222222" } },
                { "clusterName", "my-cluster" },
            });
        }

        args.Id ??= $"{args.Name}_id";
        return Task.FromResult<(string? id, object state)>((args.Id, (object)outputs));
    }

    public Task<object> CallAsync(MockCallArgs args)
    {
        return Task.FromResult((object)ImmutableDictionary<string, object>.Empty);
    }
}
```

프로그램에서는 다음과 같이 `StackReference`를 사용할 수 있다:

```csharp
var networkStack = new StackReference("organization/network/prod");
var vpcId = networkStack.GetOutput("vpcId");
// 테스트에서 vpcId는 "vpc-12345678"로 해결된다
```

**Java 예제:**

```java
import java.util.List;

class MyMocks implements Mocks {
    @Override
    public CompletableFuture<ResourceResult> newResourceAsync(ResourceArgs args) {
        var state = new HashMap<>(args.inputs);
        if ("pulumi:pulumi:StackReference".equals(args.type)) {
            state.put("outputs", Map.of(
                "vpcId", "vpc-12345678",
                "subnetIds", List.of("subnet-11111111", "subnet-22222222"),
                "clusterName", "my-cluster"
            ));
        }
        return CompletableFuture.completedFuture(
            ResourceResult.of(Optional.of(args.name + "_id"), state)
        );
    }

    @Override
    public CompletableFuture<Map<String, Object>> callAsync(CallArgs args) {
        return CompletableFuture.completedFuture(Map.of());
    }
}
```

프로그램에서는 다음과 같이 `StackReference`를 사용할 수 있다:

```java
var networkStack = new StackReference("organization/network/prod",
    StackReferenceArgs.builder().build());
var vpcId = networkStack.getOutput(Output.of("vpcId"));
// 테스트에서 vpcId는 "vpc-12345678"로 해결된다
```

> **참고:** YAML 프로그램은 선언형이므로 단위 테스트 모킹에서 StackReference를 지원하지 않는다. StackReference를 사용하는 프로그램의 테스트는 [통합 테스트](https://www.pulumi.com/docs/iac/guides/testing/integration/)를 참조하라.

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

**Go 테스트 예제:**

```go
package main

import (
    "sync"
    "testing"

    "github.com/pulumi/pulumi-aws/sdk/v4/go/aws/ec2"
    "github.com/pulumi/pulumi/sdk/v3/go/common/resource"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
    "github.com/stretchr/testify/assert"
)

// ... mocks as shown above

func TestInfrastructure(t *testing.T) {
    err := pulumi.RunErr(func(ctx *pulumi.Context) error {
        infra, err := createInfrastructure(ctx)
        assert.NoError(t, err)

        var wg sync.WaitGroup
        wg.Add(3)

        // 검사 1: 인스턴스에 Name 태그가 있어야 함
        pulumi.All(infra.server.URN(), infra.server.Tags).ApplyT(func(all []interface{}) error {
            urn := all[0].(pulumi.URN)
            tags := all[1].(map[string]string)
            assert.Containsf(t, tags, "Name", "missing a Name tag on server %v", urn)
            wg.Done()
            return nil
        })

        // 검사 2: userData 스크립트를 사용하지 않아야 함
        pulumi.All(infra.server.URN(), infra.server.UserData).ApplyT(func(all []interface{}) error {
            urn := all[0].(pulumi.URN)
            userData := all[1].(string)
            assert.Emptyf(t, userData, "illegal use of userData on server %v", urn)
            wg.Done()
            return nil
        })

        // 검사 3: SSH가 인터넷에 열려 있지 않아야 함
        pulumi.All(infra.group.URN(), infra.group.Ingress).ApplyT(func(all []interface{}) error {
            urn := all[0].(pulumi.URN)
            ingress := all[1].([]ec2.SecurityGroupIngress)
            for _, i := range ingress {
                openToInternet := false
                for _, b := range i.CidrBlocks {
                    if b == "0.0.0.0/0" {
                        openToInternet = true
                        break
                    }
                }
                assert.Falsef(t, i.FromPort == 22 && openToInternet,
                    "illegal SSH port 22 open to the Internet (CIDR 0.0.0.0/0) on group %v", urn)
            }
            wg.Done()
            return nil
        })

        wg.Wait()
        return nil
    }, pulumi.WithMocks("project", "stack", mocks(0)))
    assert.NoError(t, err)
}
```

**C# 테스트 예제:**

```csharp
using System.Linq;
using System.Threading.Tasks;
using FluentAssertions;
using NUnit.Framework;
using Pulumi.Aws.Ec2;

namespace UnitTesting
{
    [TestFixture]
    public class WebserverStackTests
    {
        // 검사 1: 인스턴스에 Name 태그가 있어야 함
        [Test]
        public async Task InstanceHasNameTag()
        {
            var resources = await Testing.RunAsync<WebserverStack>();

            var instance = resources.OfType<Instance>().FirstOrDefault();
            instance.Should().NotBeNull("EC2 Instance not found");

            var tags = await instance.Tags.GetValueAsync();
            tags.Should().NotBeNull("Tags are not defined");
            tags.Should().ContainKey("Name");
        }

        // 검사 2: userData 스크립트를 사용하지 않아야 함
        [Test]
        public async Task InstanceMustNotUseInlineUserData()
        {
            var resources = await Testing.RunAsync<WebserverStack>();

            var instance = resources.OfType<Instance>().FirstOrDefault();
            instance.Should().NotBeNull("EC2 Instance not found");

            var userData = await instance.UserData.GetValueAsync();
            userData.Should().BeNull();
        }

        // 검사 3: SSH가 인터넷에 열려 있지 않아야 함
        [Test]
        public async Task SecurityGroupMustNotHaveSshPortsOpenToInternet()
        {
            var resources = await Testing.RunAsync<WebserverStack>();

            foreach (var securityGroup in resources.OfType<SecurityGroup>())
            {
                var urn = await securityGroup.Urn.GetValueAsync();
                var ingress = await securityGroup.Ingress.GetValueAsync();
                foreach (var rule in ingress)
                {
                    (rule.FromPort == 22 && rule.CidrBlocks.Any(b => b == "0.0.0.0/0"))
                        .Should().BeFalse($"Illegal SSH port 22 open to the Internet (CIDR 0.0.0.0/0) on group {urn}");
                }
            }
        }
    }
}
```

**Java 테스트 예제:**

```java
package myproject;

import com.pulumi.test.PulumiTest;
import com.pulumi.test.TestOptions;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class Ec2Tests {
    @AfterEach
    void cleanup() {
        PulumiTest.cleanup();
    }

    // 검사 1: 인스턴스에 Name 태그가 있어야 함
    @Test
    void instanceMustHaveNameTag() {
        var result = PulumiTest
            .withMocks(new MyMocks())
            .withOptions(TestOptions.builder()
                .projectName("project").stackName("stack").preview(false)
                .build())
            .runTest(App::stack);

        var instances = result.resources().stream()
            .filter(r -> r instanceof Instance)
            .map(r -> (Instance) r)
            .toList();

        assertFalse(instances.isEmpty(), "EC2 Instance not found");
        for (var instance : instances) {
            var urn = PulumiTest.extractValue(instance.urn());
            var tags = PulumiTest.extractValue(instance.tags());
            assertNotNull(tags, "Server " + urn + " must have tags");
            assertTrue(tags.containsKey("Name"), "Server " + urn + " must have a Name tag");
        }
    }

    // 검사 2: userData 스크립트를 사용하지 않아야 함
    @Test
    void instanceMustNotUseInlineUserData() {
        var result = PulumiTest
            .withMocks(new MyMocks())
            .withOptions(TestOptions.builder()
                .projectName("project").stackName("stack").preview(false)
                .build())
            .runTest(App::stack);

        var instance = result.resources().stream()
            .filter(r -> r instanceof Instance)
            .map(r -> (Instance) r)
            .findFirst().orElse(null);

        assertNotNull(instance, "EC2 Instance not found");
        var urn = PulumiTest.extractValue(instance.urn());
        var userData = PulumiTest.extractValue(instance.userData());
        assertNull(userData, "Illegal use of userData on server " + urn);
    }

    // 검사 3: SSH가 인터넷에 열려 있지 않아야 함
    @Test
    void securityGroupMustNotHaveSshOpenToInternet() {
        var result = PulumiTest
            .withMocks(new MyMocks())
            .withOptions(TestOptions.builder()
                .projectName("project").stackName("stack").preview(false)
                .build())
            .runTest(App::stack);

        for (var resource : result.resources()) {
            if (resource instanceof SecurityGroup group) {
                var urn = PulumiTest.extractValue(group.urn());
                var ingress = PulumiTest.extractValue(group.ingress());
                if (ingress != null) {
                    for (var rule : ingress) {
                        var fromPort = PulumiTest.extractValue(rule.fromPort());
                        var cidrBlocks = PulumiTest.extractValue(rule.cidrBlocks());
                        boolean sshOpen = fromPort != null && fromPort == 22
                            && cidrBlocks != null && cidrBlocks.contains("0.0.0.0/0");
                        assertFalse(sshOpen, "Illegal SSH port 22 open to the Internet "
                            + "(CIDR 0.0.0.0/0) on group " + urn);
                    }
                }
            }
        }
    }
}
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

통합 테스트는 클라우드 리소스를 모킹하는 단위 테스트나 특정 리소스의 기대값을 검증하는 Policy as Code와 달리, 실제 인프라를 배포하여 코드의 동작을 종단 간으로 검증할 수 있게 한다.

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

### DIY 옵션

Automation API 외에도 Pulumi CLI 명령을 셸에서 직접 실행하여 통합 테스트를 작성할 수 있다. 셸 스크립트나 선호하는 언어에서 셸 명령을 실행하는 방식이다. Automation API가 언어 네이티브 API를 제공하여 이 과정을 더 쉽게 만들어주지만, 필요하다면 셸 스크립트로도 유사한 결과를 얻을 수 있다.

| 장점 | 단점 |
|------|------|
| 모든 프로그래밍 언어에서 사용 가능 | 수동 오류 처리 필요 |
| 이해와 구현이 간단 | Automation API보다 타입 안전성이 낮음 |
| Pulumi CLI 외에 추가 의존성 없음 | 리소스 속성 추출 및 검증이 어려움 |

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

### 추가 리소스

- [Integration Testing in Go 예제](https://github.com/pulumi/examples/tree/master/testing-integration) - Pulumi Go 통합 테스트 프레임워크를 사용한 최소 예제
- [Pulumi AWS 프로바이더 테스트](https://github.com/pulumi/pulumi-aws/tree/master/examples) - AWS 프로바이더의 포괄적인 통합 테스트 예제
- [Pulumi 예제 테스트 스위트](https://github.com/pulumi/examples/blob/master/misc/test/examples_test.go) - 추가 예제 및 패턴

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

`localProgram-tsnode-mochatests` 예제는 Automation API로 스택을 설정하고 런타임 검증을 수행한 후 테스트의 일환으로 스택을 정리하는 방법을 보여준다. [localProgram-tsnode-mochatests 예제](https://github.com/pulumi/automation-api-examples/tree/main/nodejs/localProgram-tsnode-mochatests)에서 확인할 수 있다.

---

## 테스트 전략과 모범 사례

### 테스트 피라미드 적용

Pulumi는 세 가지 테스트 스타일을 모두 시도해 보고, 품질 목표, 개발 실천 방식, 애플리케이션 스타일에 가장 적합한 것을 선택할 것을 권장한다. 많은 팀에서는 이 세 가지 접근 방식을 **조합**하여 사용하는 것이 합리적이다.

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
- Automation API를 사용하면 선호하는 언어와 테스트 프레임워크로 통합 테스트를 작성할 수 있다.
- 셸 스크립트 기반 DIY 접근 방식도 가능하지만, Automation API가 더 나은 타입 안전성과 오류 처리를 제공한다.
- 통합 테스트에서 Pulumi 서비스(Pulumi Cloud)와의 상호작용을 테스트하려면 Automation API로 스택 생성 시 `projectName`과 `stackName`을 지정하여 격리된 테스트 스택을 사용하라.

### 언어별 테스트 지원 요약

| 언어 | 단위 테스트 | 속성 테스트 | Integration Framework | Automation API |
|------|-----------|-----------|----------------------|----------------|
| TypeScript | 예 | 예 | 예 (Go로 작성) | 예 |
| Python | 예 | 예 | 예 (Go로 작성) | 예 |
| Go | 예 | 아니오 | 예 | 예 |
| C# | 예 | 아니오 | 예 (Go로 작성) | 예 |
| Java | 예 | 아니오 | 예 (Go로 작성) | 예 |
| YAML | 아니오 | 아니오 | 예 (Go로 작성) | 아니오 |

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
