# Pulumi Insights & IDP

> https://www.pulumi.com/docs/insights/
> https://www.pulumi.com/docs/insights/discovery/
> https://www.pulumi.com/docs/insights/policy/
> https://www.pulumi.com/docs/idp/

Pulumi Insights & Governance는 클라우드 인프라 전체에 대한 가시성과 정책 실행을 제공한다. 리소스가 Pulumi, Terraform, CloudFormation 또는 수동으로 생성되었는지와 무관하게 모든 인프라를 검색하고 컴플라이언스를 보장한다. Pulumi IDP(Internal Developer Platform)는 플랫폼 팀이 재사용 가능한 컴포넌트, 템플릿, 골든 패스를 제공하여 개발자가 Day 0부터 Day 2까지 셀프 서비스 인프라 워크플로를 구축할 수 있게 한다.

---

## Insights & Governance 개요

Pulumi Cloud은 리소스 프로비저닝 도구에 관계없이 클라우드 인프라에 대한 전체 가시성과 제어를 제공한다.

| 시작 경로 | 설명 |
|---|---|
| **Discovery** 먼저 시작 | 기존 인프라를 스캔하여 전수 조사한 뒤 Policy 실행 추가 |
| **Pulumi IaC 이미 사용 중** | [Policy](#insights-policy)를 추가하여 프로덕션 배포 전 컴플라이언스 보장 |

---

## Insights Discovery

> https://www.pulumi.com/docs/insights/discovery/

Pulumi Insights Discovery는 클라우드 프로바이더 어카운트를 스캔하여 리소스 생성 방법이나 관리 상태와 무관하게 모든 리소스의 종합 인벤토리를 구축한다.

### 작동 방식

Discovery는 [Pulumi ESC](https://www.pulumi.com/docs/esc/)와 통합하여 자격 증명을 안전하게 관리하고 클라우드 인프라를 스캔한다.

| 단계 | 설명 |
|---|---|
| **어카운트 관리** | Pulumi Cloud의 Accounts 페이지에서 어카운트를 생성하고 구성. 스캔 상태, 진행 상황, 설정 관리 가능 |
| **리소스 스캔** | ESC의 읽기 전용 자격 증명으로 클라우드 프로바이더에 인증. 리소스 식별, 메타데이터 수집, 관계 기록, Insights 슈퍼그래프 업데이트 |
| **탐색** | [Resource Search](#resource-search)를 통해 인프라 탐색. 풍부한 필터링, 그룹화, [Pulumi Neo](https://www.pulumi.com/docs/ai/)를 통한 자연어 쿼리 지원 |
| **임포트** | [Visual Import](https://www.pulumi.com/docs/insights/discovery/visual-import/)로 검색된 리소스를 Pulumi IaC 코드로 변환 |

### 어카운트 계층 구조

Discovery는 필요시 자동으로 자식 어카운트를 생성한다. AWS의 경우 선택한 각 리전이 메인 부모 어카운트 아래의 자식 어카운트가 된다.

| 계층 예시 | 설명 |
|---|---|
| `my-aws-account` | 부모 어카운트 |
| `my-aws-account/us-east-1` | 리전 자식 어카운트 |
| `my-aws-account/us-east-1/my-cluster` | K8s 클러스터 하위 자식 (향후 지원 예정) |

- 부모 어카운트 작업(스캔, 삭제)은 모든 자식에 전파
- 개별 자식 어카운트는 독립적으로 관리 가능
- 자식 어카운트는 부모의 ESC 자격 증명 및 설정을 상속

### 어카운트 생성

**사전 요구 사항:**
- Pulumi 조직의 관리자 권한
- 스캔할 프로바이더 어카운트 내 자격 증명 생성 권한

**생성 절차:**

| 단계 | 설명 |
|---|---|
| 1 | Pulumi Cloud Console의 **Accounts** 탭에서 **Create Account** 클릭 |
| 2 | 프로바이더 선택 |
| 3 | 올바른 자격 증명을 가진 ESC 환경 선택 또는 생성 |
| 4 | 고유한 어카운트 이름 입력 (이름에 `/` 포함 불가, 자식 어카운트는 Pulumi가 자동으로 `/` 사용) |
| 5 | 프로바이더별 구성 추가 (AWS의 경우 파티션 선택, 제외 리전 지정) |
| 6 | 예약 스캔 활성화 여부 선택 (활성화 시 24시간마다 자동 스캔) |
| 7 | Create 클릭 |

### 지원 프로바이더 및 ESC 자격 증명 구성

| 프로바이더 | 인증 방식 |
|---|---|
| **AWS** | ESC를 통한 OIDC. IAM Role에 `ReadOnlyAccess` 관리형 정책 권장. 모든 AWS 파티션 지원 (Standard, GovCloud, ISO, ISOB, ISOF, ISOE, European Sovereign Cloud, China) |
| **Azure** | ESC를 통한 OIDC(권장) 또는 클라이언트 시크릿. Microsoft Entra 앱 구성 필요 |
| **Oracle Cloud (OCI)** | API 키 인증. `OCI_TENANCY_OCID`, `OCI_USER_OCID`, `OCI_FINGERPRINT`, `OCI_REGION`, `OCI_PRIVATE_KEY_PATH` 필요 |
| **Google Cloud** | ESC를 통한 OIDC. Service Account에 Viewer 역할 권장. Workload Identity Federation 구성 필요 |
| **Kubernetes** | kubeconfig 기반. `get` 및 `list` 권한이 있는 ServiceAccount + ClusterRole 권장. client-go credential plugin 미지원 |

### Resource Search

Resource Search 인터페이스에서 제공하는 기능:

| 기능 | 설명 |
|---|---|
| **고급 검색** | 이름, 유형, 스택, 프로젝트, 속성 등으로 리소스 쿼리 |
| **필터링 및 그룹화** | 컬럼 헤더를 드래그하여 커스텀 뷰 생성 |
| **컬럼 커스터마이징** | 필요한 정보만 표시/숨기기 |
| **즐겨찾기** | 커스텀 뷰를 팀과 공유 |
| **AI 보조** | 자연어 쿼리로 리소스 검색 (예: "How many VPCs do I have?") |

- 페이지당 최대 10,000개 결과 표시
- 대규모 데이터셋은 [Data Export](https://www.pulumi.com/docs/insights/discovery/data-export/) 또는 [REST API](https://www.pulumi.com/docs/pulumi-cloud/cloud-rest-api#resource-search) 사용

### 리소스 관계 및 통합

| 기능 | 설명 |
|---|---|
| **리소스 관계** | S3 버킷과 버킷 정책, VM과 스토리지, 네트워크 인터페이스와 보안 그룹 간의 관계를 그래프로 유지. Resource Explorer에서 확인 가능 |
| **통합 리소스** | 동일 리소스가 여러 소스(IaC 스택, Discovery 스캔)에 존재하면 검색 결과에서 통합하여 표시 (spoke 아이콘으로 표시) |
| **Managed By 속성** | `Pulumi`(Pulumi 스택으로 관리) 또는 `Other`(Discovery에서 감지되었으나 Pulumi 스택으로 관리되지 않음)로 분류 |

### 접근 제어

| 역할 | 권한 |
|---|---|
| 조직 관리자 | 모든 리소스 접근 가능 |
| 기본 권한이 read/write인 조직 | 모든 사용자가 모든 리소스 쿼리 가능 |
| 기본 권한이 없는 조직 | Stack 또는 Team 권한으로 접근 권한이 있는 리소스만 쿼리 가능 |

---

## Insights Policy

> https://www.pulumi.com/docs/insights/policy/

Pulumi Policies를 사용하면 코드로 정책(Policy as Code)을 구현하여 전체 클라우드 인프라에 적용할 수 있다. 비즈니스 및 보안 규칙을 코드로 작성하여 자동화된 컴플라이언스 보호를 제공한다.

### Policy as Code 개념

Policy as Code는 소프트웨어 엔지니어링 실천을 인프라 정책에 적용하는 것이다. 프로그래밍 언어로 정책을 작성하고 인프라 코드와 함께 관리한다.

| 혜택 | 설명 |
|---|---|
| **비용 제어** | 리소스 가격 기반 정책으로 비싼 배포를 사전에 방지. 지출 한도 설정, 미사용 리소스 식별, 태깅 강제 |
| **컴플라이언스 및 보안** | 공개 S3 버킷, 노출된 데이터베이스, 과도하게 관대한 보안 그룹 등의 일반적인 오설정 방지 |
| **조기 검증** | `pulumi preview` 시 정책 위반을 포착하여 리소스 생성 전에 차단 |
| **모범 사례의 코드화** | 조직 표준과 클라우드 프로바이더 모범 사례를 버전 관리되고 테스트 가능한 정책으로 인코딩 |
| **클라우드 네이티브 도구 통합** | AWS IAM Access Analyzer, AWS Organizations Tag Policies 등과 함께 작동 |

Policy as Code는 [analyzer plugin](https://www.pulumi.com/docs/iac/concepts/plugins/#analyzer-plugins)을 통해 구현되며, Pulumi CLI와 함께 자동 설치된다.

### 정책 계층 구조

| 구성 요소 | 설명 |
|---|---|
| **Policy** | 인프라 구성을 검증하는 개별 규칙 (예: "S3 버킷은 비공개여야 함", "VM은 승인된 인스턴스 유형만 사용") |
| **Policy Pack** | 관련 정책의 버전 관리된 컬렉션. [사전 구축된 팩](#사전-구축된-policy-packs) 사용 또는 TypeScript, JavaScript, Python, OPA(Rego)로 [커스텀 팩 작성](https://www.pulumi.com/docs/insights/policy/policy-packs/authoring/) 가능 |
| **Policy Group** | 특정 스택이나 클라우드 어카운트에 policy pack을 적용. 프로덕션에 더 엄격한 정책, 개발에 더 관대한 정책 적용 가능 |

### 실행 모드

| 모드 | 설명 | 차단 여부 |
|---|---|---|
| **Preventative** | `pulumi preview` 및 `pulumi up` 시 Pulumi 스택 리소스를 검증. 위반 감지 시 배포 차단 | **예** (`mandatory` 시) |
| **Audit** | [Insights Discovery](#insights-discovery)를 통해 발견된 리소스를 지속적으로 스캔. Terraform, CloudFormation, 수동 생성 리소스 포함. 차단 없이 가시성만 제공 | **아니오** |

### Policy Group 비교

| 항목 | Preventative Policy Group | Audit Policy Group |
|---|---|---|
| **대상** | Pulumi 스택 | Pulumi 스택 및 클라우드 어카운트 |
| **실행 시점** | `pulumi up` / `pulumi preview` 시 | 스택 업데이트 시 및 예약 스캔 |
| **주요 목표** | 비준수 배포 방지 | 기존 비준수 감지 및 모니터링 |
| **범위** | Pulumi 관리 리소스 | 스택 상태 및 모든 클라우드 어카운트 리소스 |
| **배포 차단** | **예** (`mandatory` 시) | **아니오** |

### 실행 수준

| 수준 | 설명 |
|---|---|
| **Advisory** | 경고만 표시하고 배포는 진행. 새 정책 테스트 또는 정보 제공에 유용 |
| **Mandatory** | 위반 감지 시 배포 차단. 중요한 보안, 컴플라이언스, 비용 정책에 사용 |

### 로컬 실행 vs Pulumi Cloud

| 항목 | 로컬 실행 | Pulumi Cloud |
|---|---|---|
| **Preventative** | `--policy-pack` 플래그로 적용. 모든 백엔드 사용 가능 | 중앙 관리, 자동 다운로드, 버전 제어, 콘솔에서 결과 확인 |
| **Audit** | 미지원 | Discovery 리소스 지속 스캔. 모든 인프라 위반 식별. [Policy Findings](#policy-findings) 대시보드에서 확인 |
| **제약** | Policy pack이 로컬 디스크에 있어야 함 | Pulumi Cloud 전용 (Self-managed 백엔드에서는 Audit 사용 불가) |

### 지원 언어

| 언어 | 상태 |
|---|---|
| **TypeScript/JavaScript** (Node.js) | Stable |
| **Python** | Stable |
| **Open Policy Agent (OPA)** (Rego) | Stable |
| **.NET** | Future |
| **Go** | Future |

### 사전 구축된 Policy Packs

> https://www.pulumi.com/docs/insights/policy/policy-packs/pre-built-packs/

Pulumi Cloud은 일반적인 보안 및 컴플라이언스 프레임워크에 대한 사전 구축된 policy pack을 제공한다.

| 프레임워크 | 지원 클라우드 프로바이더 | 설명 |
|---|---|---|
| **CIS 8.1** | AWS, Azure, Google Cloud | CIS 8.1 제어를 실행하여 업계 인정 보안 모범 사례 적용 |
| **CIS Kubernetes** | AWS (EKS), Azure (AKS), Google Cloud (GKE) | 관리형 Kubernetes 서비스에 CIS Kubernetes Benchmark 제어 실행 |
| **HITRUST CSF 11.5** | AWS, Azure, Google Cloud | HITRUST CSF 요구사항에 맞춘 사전 정의된 제어 제공 |
| **NIST SP 800-53** | AWS | NIST SP 800-53 rev. 5 보안 및 프라이버시 제어 실행 |
| **PCI DSS v4.0.1** | AWS | PCI DSS v4.0.1 컴플라이언스 제어 실행 |
| **Pulumi Best Practices** | AWS, Azure, Google Cloud | 기본적인 거버넌스 및 보안 제어 권장 세트 |
| **AWS Organizations Tag Policies** | AWS, AWS-Native | AWS Organizations Tag Policies와 통합하여 필수 태그 검증 |

- 사전 구축 팩은 시맨틱 버전링을 따름
- 커스텀 policy pack과 동일한 Policy Group에 함께 추가 가능

### Policy Findings

> https://www.pulumi.com/docs/insights/policy/policy-findings/

Policy Findings는 클라우드 인프라 전체의 컴플라이언스를 관리하는 중앙 뷰를 제공한다.

| 탭 | 기능 |
|---|---|
| **Overview** | 조직의 보안 및 컴플라이언스 상태 개요. Policy 준수 점수와 Resource 준수 점수 표시. 히트맵으로 스택/어카운트별 정책 팩 컴플라이언스 시각화 |
| **Compliance** | 정책 중심 뷰. 개별 정책별로 결과를 그룹화. CIS, NIST, PCI DSS 등 프레임워크 감사 시 유용 |
| **Issues** | 정책 결과를 작업 항목으로 관리. 분류, 할당, 우선순위 지정, 해결 추적 |

**준수 점수:**

| 점수 | 계산 방식 |
|---|---|
| **Policy 준수 점수** | 조직 전체에서 현재 통과 중인 활성화된 정책 규칙의 백분율 |
| **Resource 준수 점수** | 모든 적용 가능한 정책을 완전히 준수하는 관리 대상 리소스의 백분율 |

**Issue 관리:**

| 속성 | 값 |
|---|---|
| **Status** | Open, In Progress, Ignored |
| **Priority** | P0 (critical) ~ P4 (low) |
| **Assignment** | 조직 내 특정 팀 멤버에게 할당 |
| **AI Remediation** | Pulumi Neo 사용 시 여러 이슈를 선택해 AI 기반 자동 수정 작업 생성 가능 |

Policy Findings는 [Pulumi Cloud REST API](https://www.pulumi.com/docs/reference/cloud-rest-api/policy-results/)를 통해서도 접근 가능하다.

---

## Internal Developer Platform (IDP)

> https://www.pulumi.com/docs/idp/
> https://www.pulumi.com/docs/idp/concepts/
> https://www.pulumi.com/docs/idp/guides/

Pulumi IDP는 플랫폼 팀이 인프라 빌딩 블록을 게시하고 개발자가 이를 소비하는 셀프 서비스 인프라 워크플로를 제공한다. Internal Developer Portal과 달리 Pulumi IDP는 정보 소비가 아닌 구체적인 결과를 도출한다.

### Day 0-2 라이프사이클

| 단계 | 설명 | 관련 기능 |
|---|---|---|
| **Day 0** | 골든 패스의 중앙 소스 오브 트루스 구축. 플랫폼 엔지니어가 컴포넌트와 템플릿을 작성하여 Private Registry에 게시 | Private Registry, Components, Templates, ESC, Policies |
| **Day 1** | 유연한 워크플로로 인프라 프로비저닝. Pulumi YAML, 코드 작성, 또는 [New Project Wizard](https://www.pulumi.com/docs/idp/concepts/new-project-wizard/)를 통한 노코드 배포 | Private Registry, No-code Stacks, Deployments |
| **Day 2** | 인프라 유지보수 및 확장. Services를 사용한 인프라 모델링, 구성 조정 및 재배포 | No-code Stacks, Services |

### IDP 주요 구성 요소

| 구성 요소 | 설명 |
|---|---|
| **Private Registry** | 조직의 템플릿, 컴포넌트, 인프라 빌딩 블록을 위한 중앙 레지스트리 |
| **Organization Templates** | 조직의 표준과 모범 사례가 인코딩된 템플릿에서 새 프로젝트 스캐폴딩 |
| **No-code Stacks** | 코드 작성 없이 New Project Wizard와 템플릿으로 인프라 배포 |
| **Services** | 스택, 환경, 리소스의 논리적 그룹화로 인프라 모델링 |
| **New Project Wizard** | 브라우저에서 템플릿으로 새 프로젝트를 직접 생성하고 자동 배포 설정 |
| **Backstage Plugin** | 기존 개발자 포털과 Pulumi 통합 |

### Four Factors 프레임워크

Pulumi IDP 모범 사례의 핵심은 네 가지 요소(Templates, Components, Environments, Policies)의 조합이다.

| 요소 | 역할 |
|---|---|
| **Templates** | 새 프로젝트 스캐폴딩을 위한 골든 패스 템플릿 |
| **Components** | 재사용 가능한 인프라 빌딩 블록. 보안, 컴플라이언스, 운영 요구사항이 캡슐화됨 |
| **Environments** (ESC) | 비밀, 구성, 자격 증명 관리. 서비스/팀/수명 주기 단계별로 환경 구성 |
| **Policies** | 코드로 정책을 정의하여 비용, 보안, 컴플라이언스 규칙 강제 |

---

## IDP 패턴

> https://www.pulumi.com/docs/idp/guides/best-practices/patterns/

### 환경 패턴

| 패턴 | 설명 |
|---|---|
| **서비스별 ESC 환경** | 각 서비스에 전용 ESC 환경을 할당하여 구성 격리 |
| **팀별 ESC 환경** | 팀 단위로 ESC 환경을 구성하여 팀 공유 설정 관리 |
| **수명 주기 단계별 ESC 환경** | dev, staging, prod 등 수명 주기 단계별로 환경 구성 |
| **조합형 환경 (Composable Environments)** | 여러 ESC 환경을 계층적으로 조합. 부모 환경의 설정을 자식이 상속받고 오버라이드 |

### 조합형 환경 예시

```yaml
# monitoring-base ESC environment
values:
  monitoring:
    datadog:
      apiKey:
        fn::secret: <YOUR_DD_API_KEY>
      appKey:
        fn::secret: <YOUR_DD_APP_KEY>
    alerting:
      slackChannel: "#alerts"
      pagerDutyService: "platform-team"
  environmentVariables:
    DD_API_KEY: ${monitoring.datadog.apiKey}
    DD_APP_KEY: ${monitoring.datadog.appKey}
```

```yaml
# monitoring-production ESC environment (monitoring-base 상속)
imports:
  - monitoring-base
values:
  monitoring:
    environment: "production"
    alerting:
      errorThreshold: 0.01
      responseTimeThreshold: 500
      slackChannel: "#production-alerts"
  pulumiConfig:
    monitoring:environment: ${monitoring.environment}
    monitoring:errorThreshold: ${monitoring.alerting.errorThreshold}
```

### 거버넌스 패턴

| 패턴 | 설명 |
|---|---|
| **Policies as tests** | 정책을 테스트처럼 사용하여 인프라 변경 시 자동으로 검증 |
| **Policy functions로 Component 입력 검증** | 공유 검증 로직을 policy function으로 구현하여 컴포넌트 입력값 검증 |
| **비용 제어** | 컴포넌트, 정책, 제한된 입력 타입을 조합하여 인프라 비용 통제 |

### 비용 제어 패턴 예시

제한된 입력 타입을 정의하여 허용되는 구성만 선택 가능하게 한다.

**TypeScript:**

```typescript
// types/cost-controls.ts
export type InstanceSize = "small" | "medium" | "large";
export type Environment = "dev" | "staging" | "prod";

export const INSTANCE_CONFIGS = {
  small: { instanceClass: "db.t3.micro", maxStorage: 100 },
  medium: { instanceClass: "db.t3.small", maxStorage: 500 },
  large: { instanceClass: "db.t3.medium", maxStorage: 1000 },
} as const;

export const ENVIRONMENT_LIMITS = {
  dev: { maxInstances: 2, allowedSizes: ["small"] as InstanceSize[] },
  staging: { maxInstances: 3, allowedSizes: ["small", "medium"] as InstanceSize[] },
  prod: { maxInstances: 10, allowedSizes: ["small", "medium", "large"] as InstanceSize[] },
} as const;
```

비용 제어 컴포넌트를 생성하여 제한된 옵션만 허용한다.

```typescript
// components/cost-controlled-database.ts
import { InstanceSize, Environment, INSTANCE_CONFIGS, validateCostConstraints } from "../types/cost-controls";

export interface CostControlledDatabaseArgs {
  size: InstanceSize;
  environment: Environment;
  storage: number;
}

export class CostControlledDatabase extends ComponentResource {
  constructor(name: string, args: CostControlledDatabaseArgs, opts?: ComponentResourceOptions) {
    super("acme:components:CostControlledDatabase", name, {}, opts);

    const errors = validateCostConstraints(args.size, args.environment);
    if (errors.length > 0) {
      throw new Error(`Cost constraint violations:\n${errors.join('\n')}`);
    }

    const config = INSTANCE_CONFIGS[args.size];
    if (args.storage > config.maxStorage) {
      throw new Error(`${args.size} size allows maximum ${config.maxStorage} GB storage`);
    }

    const db = new aws.rds.Instance(name, {
      instanceClass: config.instanceClass,
      allocatedStorage: args.storage,
      tags: {
        Environment: args.environment,
        Size: args.size,
        CostControlled: "true",
      },
    }, { parent: this });
  }
}
```

조직 전체에 비용 정책을 강제한다.

```typescript
// policies/cost-control-policy.ts
import { PolicyPack, validateResourceOfType } from "@pulumi/policy";
import { validateCostConstraints } from "../types/cost-controls";
import { aws } from "@pulumi/aws";

new PolicyPack("cost-control-policies", {
  policies: [{
    name: "database-cost-control",
    description: "Enforce cost controls on database instances",
    enforcementLevel: "mandatory",
    validateResource: validateResourceOfType(aws.rds.Instance, (instance, args, reportViolation) => {
      const size = instance.tags?.Size as InstanceSize;
      const environment = instance.tags?.Environment as Environment;

      if (size && environment) {
        const errors = validateCostConstraints(size, environment);
        errors.forEach(error => reportViolation(error));
      }
    }),
  }],
});
```

### 보안 업데이트 패턴

플랫폼 팀이 컴포넌트 내에 보안 구성을 중앙 집중화하고, 컴포넌트 버전을 업데이트하여 모든 애플리케이션에 보안 패치를 일괄 전파한다.

**TypeScript:**

```typescript
// components/secure-database.ts
export class SecureDatabase extends ComponentResource {
  constructor(name: string, args: SecureDatabaseArgs, opts?: ComponentResourceOptions) {
    super("acme:security:SecureDatabase", name, {}, opts);

    const db = new aws.rds.Instance(name, {
      instanceClass: args.instanceClass,
      allocatedStorage: args.storage,

      // 플랫폼 팀이 중앙 관리하는 보안 기본값
      storageEncrypted: true,
      kmsKeyId: args.kmsKeyId,
      enabledCloudwatchLogsExports: ["postgresql", "upgrade"],
      monitoringInterval: 60,
      performanceInsightsEnabled: true,
      backupRetentionPeriod: args.backupRetentionDays,

      tags: {
        SecurityCompliant: "true",
        ComponentVersion: "1.2.3",
        LastSecurityUpdate: "2024-01-15",
      },
    }, { parent: this });
  }
}
```

정책으로 최소 컴포넌트 버전을 강제한다.

```typescript
// policies/component-version-policy.ts
const MINIMUM_COMPONENT_VERSIONS = {
  "SecureDatabase": "1.2.0",
  "SecureNetwork": "2.1.0",
  "SecureStorage": "1.5.0",
};

new PolicyPack("component-version-enforcement", {
  policies: [{
    name: "enforce-minimum-component-versions",
    description: "Ensure all components are using minimum required versions for security",
    enforcementLevel: "mandatory",
    validateResource: validateResourceOfType(aws.rds.Instance, (instance, args, reportViolation) => {
      const componentVersion = instance.tags?.ComponentVersion;
      const componentName = instance.tags?.ComponentName;

      if (componentName && componentVersion) {
        const minimumVersion = MINIMUM_COMPONENT_VERSIONS[componentName];
        if (minimumVersion && isVersionOutdated(componentVersion, minimumVersion)) {
          reportViolation(
            `Component ${componentName} version ${componentVersion} is outdated. ` +
            `Minimum required version is ${minimumVersion} for security compliance.`
          );
        }
      }
    }),
  }],
});
```

### 컴포넌트 패턴

| 패턴 | 설명 |
|---|---|
| **Components using other Components** | 컴포넌트가 다른 컴포넌트를 참조하여 계층적 인프라 빌딩 블록 구성 |
| **Security Updates using Components** | 중앙 집중식 보안 구성 관리. 컴포넌트 버전 업데이트로 전체 조직에 보안 패치 전파 |

### 애플리케이션 패턴

| 패턴 | 설명 |
|---|---|
| **Multiple workloads on shared infrastructure** | 공유 인프라에 여러 워크로드를 배치하는 패턴 |

---

## 기능 비교표

### Insights Discovery vs Policy vs IDP 기능 비교

| 항목 | Discovery | Policy | IDP |
|---|---|---|---|
| **목적** | 기존 인프라 전수 검색 및 가시성 | 정책 실행 및 컴플라이언스 | 셀프 서비스 인프라 워크플로 |
| **대상** | 클라우드 어카운트 리소스 | Pulumi 스택 + 클라우드 어카운트 | 전체 인프라 라이프사이클 |
| **프로바이더 지원** | AWS, Azure, OCI, Google Cloud, Kubernetes | 모든 Pulumi 프로바이더 | 모든 Pulumi 프로바이더 |
| **실행 시점** | 예약 스캔 (24시간) 또는 수동 | 배포 시 (Preventative) / 지속적 (Audit) | Day 0-2 전체 |
| **차단 기능** | 아니오 | 예 (Preventative + Mandatory) | 정책을 통해 간접적 |
| **Private Registry** | 해당 없음 | 해당 없음 | 핵심 기능 |
| **Templates** | 해당 없음 | 해당 없음 | 핵심 기능 |
| **No-code** | Visual Import | 해당 없음 | No-code Stacks |

### Self-hosted Insights

Pulumi Insights는 Self-hosted 환경에서도 실행 가능하다. 자체 관리하는 워크플로 러너를 사용하여 고객 환경 내에서 Discovery 스캔과 Policy 평가를 수행할 수 있다. 자세한 내용은 [Self-hosted Insights](https://www.pulumi.com/docs/insights/self-hosted/)를 참조.

---

## 모범 사례 요약

| 영역 | 권장 사항 |
|---|---|
| **정책 도입** | Audit 모드로 시작하여 영향을 파악한 뒤 Preventative로 승격 |
| **실행 계층화** | 필수 규칙은 Preventative + Mandatory, 모니터링은 Audit로 분리 |
| **위험 수준별 분리** | 고위험 정책(보안, 컴플라이언스)과 저위험 정책(최적화, 모범 사례)을 별도 그룹으로 관리 |
| **컴포넌트 보안** | 보안 구성을 컴포넌트에 캡슐화하고 버전 관리로 일괄 업데이트 |
| **비용 통제** | 제한된 입력 타입 + 정책 조합으로 허용된 구성만 배포 가능하게 제한 |
| **환경 관리** | ESC 환경을 조합형(composable)으로 구성하여 공통 설정은 상속, 환경별 설정은 오버라이드 |
| **ESC 참조** | Policy Group에서 ESC 환경 참조 시 버전/태그 핀닝으로 구성 변경 시점 제어 |
