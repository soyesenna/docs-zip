# Pulumi Deployments

> https://www.pulumi.com/docs/deployments/
>
> https://www.pulumi.com/docs/deployments/deployments/
>
> https://www.pulumi.com/docs/deployments/deployments/using/
>
> https://www.pulumi.com/docs/deployments/deployments/using/settings/
>
> https://www.pulumi.com/docs/deployments/deployments/using/triggers/

Pulumi Deployments는 인프라스트럭처 코드(IaC)를 위해 목적에 맞게 구축된 관리형 CI/CD 플랫폼이다. 관리형 컴퓨트, 안전한 시크릿 처리, 버전 관리 시스템과의 깊은 통합을 제공하여 조직 전반의 인프라 변경 사항을 안전하게 배포할 수 있다. Git Push 배포, 풀 리퀘스트 자동 프리뷰, 스케줄된 작업, Drift Detection, 임시 환경(Review Stacks, TTL Stacks) 등 다양한 배포 자동화 기능을 제공한다.

---

## 핵심 구성 요소

Pulumi Deployments는 세 가지 핵심 컴포넌트로 구성된다.

| 컴포넌트 | 설명 |
|---|---|
| **Deployment Settings** | 소스 코드 위치, 클라우드 자격 증명, OIDC 구성, 환경 변수, 빌드 요구 사항, 커스텀 Docker 이미지 등 배포에 필요한 모든 설정을 정의 |
| **Managed Compute** | 보안 격리된 컴퓨트 인스턴스에서 Pulumi 작업을 실행. 자동 스케일링, 동시 배포 처리, 상세 로그 제공. 필요 시 Customer-Managed Workflow Runners로 교체 가능 |
| **Flexible Triggers** | Git Push, 풀 리퀘스트 프리뷰, REST API, UI 버튼, 스케줄, Remote Automation API 등 다양한 배포 트리거 지원 |

---

## Deployment Triggers

배포 트리거는 Deployment를 초기화하는 방법이다. Pulumi Deployments는 다음과 같은 트리거를 지원한다.

| 트리거 | 설명 | 사용 가능한 작업 |
|---|---|---|
| **Click to Deploy** | Pulumi Cloud 콘솔 UI에서 버튼 클릭으로 주문형 실행 | Update, Preview, Refresh, Destroy, Detect drift, Remediate drift |
| **Push to Deploy** | PR 생성 시 `pulumi preview`, 병합 시 `pulumi up` 자동 실행. Git Tag Push 시에도 배포 가능 | Update, Preview |
| **Review Stacks** | PR이 열릴 때마다 임시 스택을 자동 생성하고 배포. PR 병합/종료 시 자동 삭제 | Update, Destroy |
| **Scheduled Deployments** | Cron 표현식으로 정기적으로 Pulumi 작업 실행 | Update, Preview, Refresh, Destroy |
| **TTL Stacks** | 지정된 시간이 지난 후 스택 자동 삭제. 임시 인프라 비용 및 보안 관리 | Destroy |
| **REST API** | HTTP 요청으로 프로그래밍 방식 배포 트리거. Deployment Settings를 요청 본문에 전달 가능 | Update, Preview, Refresh, Destroy |
| **Deployment Webhooks** | 다른 스택의 이벤트 발생 시 Deployment 트리거 | Update, Preview, Refresh, Destroy |

### Deployment Operations

각 Pulumi Deployment은 특정 `pulumi` CLI 명령어를 중심으로 실행된다.

| 작업 | CLI 명령어 | 설명 |
|---|---|---|
| **Update** | `pulumi up` | 스택 리소스 생성 또는 업데이트 |
| **Preview** | `pulumi preview` | 변경 사항 검토 |
| **Refresh** | `pulumi refresh` | 클라우드 제공자의 실제 상태로 스택 상태 파일 업데이트 |
| **Destroy** | `pulumi destroy` | 스택의 모든 리소스 삭제 |
| **Detect drift** | `pulumi refresh` + 변경 감지 시 실패 | 상태 파일을 새로고침하고 변경이 감지되면 실패 처리 |
| **Remediate drift** | `pulumi update --refresh` | 상태 파일 새로고침 후 선언된 상태와 일치하도록 실제 리소스 수정 |

---

## Deployment Settings

Deployment Settings는 스택별로 정의되는 전체 배포 구성이다. Pulumi Cloud UI, REST API, 또는 Pulumi Cloud Provider를 통해 코드로 관리할 수 있다.

### 설정 방법

| 방법 | 설명 |
|---|---|
| **Pulumi Cloud UI** | 스택의 `Settings > Deploy` 탭에서 설정. 모든 트리거에 적용 |
| **REST API** | `PATCH /api/stacks/{org}/{project}/{stack}/deployments/settings` 엔드포인트로 설정 |
| **Pulumi Cloud Provider** | `pulumiservice.DeploymentSettings` 리소스를 사용하여 소스 제어에 설정 저장 |

> Pulumi는 스택이 자체 Deployment Settings를 정의하는 것을 권장하지 않는다. 설정 변경이 적용되려면 두 번의 배포가 필요하기 때문이다. 대신 여러 스택의 설정을 관리하는 별도의 Pulumi 프로그램을 만드는 것을 권장한다.

### Deployment Settings 코드 예제

**TypeScript:**

```typescript
import * as pulumiservice from "@pulumi/pulumiservice";

const deploymentSettings = new pulumiservice.DeploymentSettings("deploymentSettings", {
    organization: pulumi.getOrganization(),
    project: "<YOUR_PROJECT>",
    stack: "<YOUR_STACK>",
    github: {
        deployCommits: true,
        previewPullRequests: true,
        repository: "<YOUR_ORG>/<YOUR_REPO>",
    },
    sourceContext: {
        git: {
            branch: "refs/heads/main",
            repoDir: "<YOUR_REPO_DIR>",
        },
    },
});
```

**Python:**

```python
import pulumi
import pulumi_pulumiservice as pulumiservice

deployment_settings = pulumiservice.DeploymentSettings("deploymentSettings",
    organization=pulumi.get_organization(),
    project="<YOUR_PROJECT>",
    stack="<YOUR_STACK>",
    github=pulumiservice.DeploymentSettingsGithubArgs(
        deploy_commits=True,
        preview_pull_requests=True,
        repository="<YOUR_ORG>/<YOUR_REPO>",
    ),
    source_context=pulumiservice.DeploymentSettingsSourceContextArgs(
        git=pulumiservice.DeploymentSettingsGitSourceContextArgs(
            branch="refs/heads/main",
            repo_dir="<YOUR_REPO_DIR>",
        ),
    ),
)
```

---

## Path Filtering

Push to Deploy를 사용할 때, 특정 파일 변경 시에만 배포가 트리거되도록 Path Filter를 설정할 수 있다. 모노레포에서 각 스택이 자체 파일 변경 시에만 배포되도록 하는 데 특히 유용하다.

### 필터 작성 규칙

| 패턴 | 설명 |
|---|---|
| `infrastructure/**` | `infrastructure/` 디렉토리 하위의 모든 파일 포함 |
| `infrastructure/Pulumi.dev.yaml` | 해당 파일만 포함 |
| `!infrastructure/docs/**` | `infrastructure/docs/` 하위 파일 제외 |

### 필터 평가 규칙

- **필터 없음**: 모든 Push가 배포를 트리거
- **Include 필터만**: 변경된 파일 중 하나 이상이 Include 필터와 일치하면 배포 트리거
- **Exclude 필터만**: Exclude 필터에 일치하지 않는 모든 파일이 포함
- **Exclude가 항상 우선**: 파일이 Exclude 필터와 일치하면 Include 필터가 있어도 제외됨. `.gitignore`와 달리 나중에 재포함 불가

---

## Tag Filtering

Git Tag Push 시 배포를 트리거할 수 있다. 릴리즈 기반 워크플로우(예: `v1.2.0` Tag Push 시에만 배포)에 유용하다.

| 설정 | 설명 |
|---|---|
| `deployTags` | Tag Push 시 배포 활성화 여부 (boolean) |
| `tagFilters` | Tag 이름에 대한 Glob 패턴 목록 |

| 패턴 | 설명 |
|---|---|
| `v*` | `v`로 시작하는 모든 Tag 포함 |
| `!*-rc*` | RC(Release Candidate) Tag 제외 |

> Tag 삭제는 배포를 트리거하지 않는다. Tag 푸시 시 `PULUMI_CI_TAG_NAME` 환경 변수에 Tag 이름이 설정된다.

---

## Pre-Run Commands

배포 프로세스가 시작되기 전에 임의의 셸 명령을 실행할 수 있다. 각 명령줄은 별도의 셸에서 실행된다.

**사용 예시:**

```bash
# Pulumi ESC 환경과 함께 사용
pulumi env run <YOUR_ESC_ENV> -- aws s3 ls

# 환경 변수를 Pulumi 프로그램에 전달
export GOOGLE_OAUTH_ACCESS_TOKEN=$(gcloud auth print-access-token)
echo GOOGLE_OAUTH_ACCESS_TOKEN=$GOOGLE_OAUTH_ACCESS_TOKEN >> /PULUMI_ENV
```

### PULUMI_ENV

`/PULUMI_ENV` 파일에 환경 변수를 추가하면 Pre-Run 명령과 최종 Pulumi 배포 간에 환경 변수가 유지된다.

---

## 환경 변수

Pulumi Deployments는 기본적으로 다음 환경 변수를 설정한다.

| 환경 변수 | 설명 |
|---|---|
| `GITHUB_TOKEN` | 소스가 GitHub일 때 PAT (커스텀 환경 변수로 오버라이드하지 않은 경우) |
| `PULUMI_ACCESS_TOKEN` | 배포 중인 스택에만 읽기-쓰기 권한이 있는 임시 토큰 |
| `PULUMI_DEPLOY_OIDC_CONFIG` | 클라우드 통합을 위한 OIDC 구성 |
| `PULUMI_CI_SYSTEM` | `"Pulumi Deployments"` |
| `PULUMI_CI_BUILD_ID` | 현재 Deployment ID |
| `PULUMI_CI_BUILD_NUMBER` | 현재 Deployment 버전 |
| `PULUMI_CI_BUILD_URL` | 현재 Deployment URL |
| `PULUMI_CI_ORGANIZATION` | 조직 이름 |
| `PULUMI_CI_PROJECT` | 프로젝트 이름 |
| `PULUMI_CI_STACK` | 스택 이름 |
| `PULUMI_CI_OPERATION` | 현재 작업 (`update`, `preview`, `destroy`, `refresh`, `detect-drift`, `remediate-drift`) |
| `PULUMI_CI_TAG_NAME` | Tag Push 시 Tag 이름 (예: `v1.2.0`) |

---

## OIDC 인증

> https://www.pulumi.com/docs/deployments/deployments/oidc/
>
> https://www.pulumi.com/docs/deployments/deployments/oidc/aws/
>
> https://www.pulumi.com/docs/deployments/deployments/oidc/azure/
>
> https://www.pulumi.com/docs/deployments/deployments/oidc/gcp/

Pulumi Deployments는 OpenID Connect(OIDC)를 통해 클라우드 제공자와의 인증을 지원한다. 정적 자격 증명 대신 동적이고 단기 수명의 클라우드 자격 증명을 사용할 수 있어 보안이 강화된다.

### OIDC 토큰 구조

배포가 실행될 때마다 Pulumi Cloud는 해당 실행에 특정한 새로운 OIDC 토큰을 발행한다. 토큰은 서명된 단기 수명의 JWT(JSON Web Token)이다.

**표준 클레임:**

| 클레임 | 설명 |
|---|---|
| `aud` | (Audience) 배포와 연결된 조직 이름 |
| `iss` | (Issuer) `https://api.pulumi.com/oidc` |
| `sub` | (Subject) 배포에 대한 정보를 포함. 신뢰 관계 구성에 사용 |

**Subject 형식:**

```
pulumi:deploy:org:<organization name>:project:<project name>:stack:<stack name>:operation:<operation kind>:scope:write
```

**커스텀 클레임:**

| 클레임 | 설명 |
|---|---|
| `stackId` | 배포 중인 스택의 정규화된 식별자 |
| `operation` | 배포 작업 (`preview`, `update`, `refresh`, `destroy`) |
| `org` | 조직 이름 |
| `project` | 프로젝트 이름 |
| `stack` | 스택 이름 |
| `deployment` | Deployment 버전 |
| `scope` | OIDC 토큰의 범위 (항상 `write`) |

### AWS OIDC 설정

AWS에서는 Web Identity Provider를 생성하여 IAM Role을 수임(Assume)하는 방식으로 OIDC를 구성한다.

**구성 단계:**

1. IAM 콘솔에서 Identity Provider 생성 (Provider URL: `https://api.pulumi.com/oidc`, Audience: Pulumi 조직 이름)
2. IAM Role 및 Trust Policy 구성
3. Pulumi 콘솔에서 OIDC 활성화 (Role ARN, Session Name 입력)

**신뢰 관계 제한 예시 (조직 전체):**

```json
"Condition": {
  "StringEquals": {
    "api.pulumi.com/oidc:aud": "<YOUR_ORG_NAME>"
  },
  "StringLike": {
    "api.pulumi.com/oidc:sub": "pulumi:deploy:org:<YOUR_ORG_NAME>:*"
  }
}
```

**신뢰 관계 제한 예시 (특정 프로젝트):**

```json
"Condition": {
  "StringEquals": {
    "api.pulumi.com/oidc:aud": "<YOUR_ORG_NAME>"
  },
  "StringLike": {
    "api.pulumi.com/oidc:sub": "pulumi:deploy:org:<YOUR_ORG_NAME>:project:<YOUR_PROJECT_NAME>:*"
  }
}
```

**Session Name 템플릿 변수:**

| 변수 | 설명 |
|---|---|
| `${organization.name}` | Pulumi 조직 이름 |
| `${project.name}` | 프로젝트 이름 |
| `${stack.name}` | 스택 이름 |
| `${deployment.operation}` | 작업 유형 (예: `update`, `preview`) |
| `${deployment.version}` | Deployment 버전 번호 |
| `${deployment.id}` | Deployment UUID |

> AWS `RoleSessionName`은 최대 64자. 초과 시 `organization.name`, `project.name`, `stack.name`이 끝에서부터 잘린다. `deployment.operation`, `deployment.version`, `deployment.id`는 보호되어 잘리지 않는다.

**설정 후 환경 변수:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`. 원본 OIDC 토큰은 `PULUMI_OIDC_TOKEN` 환경 변수와 `/mnt/pulumi/pulumi.oidc` 파일에서도 사용 가능.

### Azure OIDC 설정

Azure에서는 Microsoft Entra App Registration과 Workload Identity Federation을 사용하여 OIDC를 구성한다.

**구성 단계:**

1. Microsoft Entra 콘솔에서 App Registration 생성
2. Federated Credential 추가 (Issuer: `https://api.pulumi.com/oidc`, Audience: Pulumi 조직 이름)
3. Service Principal 생성 및 역할 할당
4. Pulumi 콘솔에서 OIDC 활성화 (Client ID, Tenant ID, Subscription ID 입력)

**Subject Claim 예시 (모든 작업 허용):**

```
pulumi:deploy:org:contoso:project:core:stack:dev:operation:preview:scope:write
pulumi:deploy:org:contoso:project:core:stack:dev:operation:update:scope:write
pulumi:deploy:org:contoso:project:core:stack:dev:operation:refresh:scope:write
pulumi:deploy:org:contoso:project:core:stack:dev:operation:destroy:scope:write
```

> Azure의 Federated Credential은 Subject 식별자가 OIDC 토큰의 Subject Claim과 정확히 일치해야 하므로, 각 작업 유형별로 별도의 Credential을 생성해야 한다.

**설정 후 환경 변수:** `ARM_CLIENT_ID`, `ARM_TENANT_ID`, `ARM_SUBSCRIPTION_ID`

### GCP OIDC 설정

Google Cloud에서는 Workload Identity Federation을 사용하여 OIDC를 구성한다.

**구성 단계:**

1. Workload Identity Pool 및 Provider 생성 (Issuer: `https://api.pulumi.com/oidc`, Audience: Pulumi 조직 이름)
2. Service Account 생성 및 역할 할당
3. Pool에 Service Account 액세스 권한 부여 (Subject 조건 필터 사용)
4. Pulumi 콘솔에서 OIDC 활성화 (Project Number, Workload Pool ID, Identity Provider ID, Service Account Email 입력)

> GCP 역시 Azure와 마찬가지로 Subject Claim에 와일드카드를 허용하지 않으므로, 각 작업 유형별로 별도의 Credential을 생성해야 한다.

**설정 후 환경 변수:** `GOOGLE_CREDENTIALS` (Credential Configuration 형식)

---

## Cloud Credentials

> https://www.pulumi.com/docs/deployments/deployments/cloud-credentials/

Pulumi Deployments에 클라우드 자격 증명을 제공하는 두 가지 접근 방식이 있다.

| 방식 | 설명 |
|---|---|
| **Pulumi Deployments OIDC** | AWS, Azure, GCP 지원. 나머지 필요한 시크릿은 Deployment 환경 변수에 저장 |
| **Pulumi ESC** | ESC Environment를 정의하고 스택에 Import. 더 이식성이 높고 모듈식이며 구성 가능 |

### Pulumi ESC 권장 사항

Pulumi는 Deployments OIDC보다 ESC Environments 사용을 권장한다.

| 장점 | 설명 |
|---|---|
| **이식성** | 로컬에서 성공하면 Deployments에서도 성공할 신뢰도가 높음 |
| **모듈성** | 스택별 OIDC 구성 반복이 불필요. 중앙에서 정의 후 여러 스택에 Import |
| **더 많은 네이티브 통합** | OIDC, 관리형 시크릿 서비스, Kubernetes, Docker 등 다양한 통합 |
| **구성 가능성** | Environment가 다른 Environment를 Import 가능 |
| **버전 관리** | Environment 변경을 제어된 방식으로 롤아웃 가능 |

> 중요한 차이: Pulumi Deployments OIDC는 Pre-Run 명령을 포함한 전체 배포 프로세스에 적용되지만, Pulumi ESC 환경은 Pulumi IaC 작업(예: `pulumi up`)에만 적용된다. Pre-Run 명령에서 ESC를 사용하려면 `pulumi env run` 명령을 접두사로 사용해야 한다.

---

## Drift Detection

> https://www.pulumi.com/docs/deployments/deployments/drift/

Drift Detection은 클라우드 환경의 실제 상태가 Pulumi Cloud에 저장된 예상 상태와 다른 경우 이를 식별하는 프로세스다. 수동 변경, 스크립트의 의도치 않은 결과, 또는 무단 변경 등으로 인해 발생할 수 있다.

### Drift Detection 실행 방법

| 방법 | 설명 |
|---|---|
| **CLI** | `pulumi refresh --preview-only` 또는 `pulumi refresh` 실행 |
| **Click to Deploy** | 스택의 Click to Deploy 옵션에서 ad hoc 실행 |
| **스케줄** | Cron 표현식으로 정기 실행. 자동 수정(Auto-Remediation) 옵션 사용 가능 |
| **REST API** | `/api/stacks/{org}/{project}/{stack}/deployments/drift/schedules` 엔드포인트로 설정 |
| **Pulumi Service Provider** | `pulumiservice.DriftSchedule` 리소스로 코드로 관리 |

### Drift Detection 이벤트

| 이벤트 | 설명 |
|---|---|
| Drift detected | Drift 실행에서 Drift가 감지됨 |
| Drift detection succeeded | Drift 실행이 성공 (Drift 감지 여부와 무관) |
| Drift detection failed | Drift 실행이 실패 |
| Drift remediation succeeded | Drift 수정 실행이 성공 |
| Drift remediation failed | Drift 수정 실행이 실패 |

### Drift 스케줄 설정 코드 예제

**TypeScript:**

```typescript
import * as pulumiservice from "@pulumi/pulumiservice";

const driftSchedule = new pulumiservice.DriftSchedule("driftDetectionSchedule", {
    organization: "<YOUR_ORG>",
    project: "<YOUR_PROJECT>",
    stack: "<YOUR_STACK>",
    scheduleCron: "0 0 * * *", // 매일 자정
    autoRemediate: true,
});
```

**Python:**

```python
import pulumi_pulumiservice as pulumiservice

drift_schedule = pulumiservice.DriftSchedule("driftDetectionSchedule",
    organization="<YOUR_ORG>",
    project="<YOUR_PROJECT>",
    stack="<YOUR_STACK>",
    schedule_cron="0 0 * * *",  # 매일 자정
    auto_remediate=True)
```

---

## Scheduled Deployments

> https://www.pulumi.com/docs/deployments/deployments/schedules/

Scheduled Deployments는 Cron 표현식을 사용하여 정기적으로 클라우드 작업을 자동화하는 기능이다. Pulumi Deployments 동시성 제한이 적용되며, 스택 배포 일시 중지 시 예약된 배포도 대기열에 추가된다.

### 스케줄 설정 코드 예제

**TypeScript:**

```typescript
import * as pulumiservice from "@pulumi/pulumiservice";

const rawSchedule = new pulumiservice.DeploymentSchedule("rawSchedule", {
    organization: "<YOUR_ORG>",
    project: "<YOUR_PROJECT>",
    stack: "<YOUR_STACK>",
    scheduleCron: "0 0 * * *",
    pulumiOperation: pulumiservice.PulumiOperation.update,
});
```

**Python:**

```python
import pulumi_pulumiservice as pulumiservice

raw_schedule = pulumiservice.DeploymentSchedule("rawSchedule",
    organization="<YOUR_ORG>",
    project="<YOUR_PROJECT>",
    stack="<YOUR_STACK>",
    schedule_cron="0 0 * * *",
    pulumi_operation=pulumiservice.PulumiOperation.update)
```

---

## TTL Stacks

> https://www.pulumi.com/docs/deployments/deployments/ttl/

TTL(Time-to-Live) Stacks은 지정된 날짜/시간 이후에 스택을 자동으로 삭제하는 기능이다. 개발 환경 등 임시 클라우드 리소스가 자동으로 해제되어 비용을 절감하고 보안 태세를 개선할 수 있다.

### TTL 설정 코드 예제

**TypeScript:**

```typescript
import * as pulumiservice from "@pulumi/pulumiservice";

const ttlSchedule = new pulumiservice.TtlSchedule("ttlSchedule", {
    organization: "<YOUR_ORG>",
    project: "<YOUR_PROJECT>",
    stack: "<YOUR_STACK>",
    timestamp: "2024-01-01T00:00:00Z",
});
```

**Python:**

```python
import pulumi_pulumiservice as pulumiservice

ttl_schedule = pulumiservice.TtlSchedule("ttlSchedule",
    organization="<YOUR_ORG>",
    project="<YOUR_PROJECT>",
    stack="<YOUR_STACK>",
    timestamp="2024-01-01T00:00:00Z")
```

---

## Review Stacks

> https://www.pulumi.com/docs/deployments/deployments/review-stacks/

Review Stacks는 풀 리퀘스트가 열릴 때 자동으로 생성되고, 각 커밋마다 업데이트되며, PR이 병합되거나 닫힐 때 삭제되는 임시 클라우드 환경이다. GitHub, GitLab, Azure DevOps, Bitbucket 통합에서 모두 작동한다. (Custom VCS 통합에서는 사용할 수 없다.)

### 구성 단계

1. 관례적으로 `pr`라는 이름의 새 스택을 만들고 `Pulumi.pr.yaml` 설정 파일 생성
2. 스택에 Deployment Settings 구성
3. `pullRequestTemplate` Deployment Setting을 `true`로 설정

### Review Stacks 패턴

| 패턴 | 설명 |
|---|---|
| **단일 스택** | Push to Deploy, PR Preview, Review Stacks를 하나의 스택으로 구성. 가장 간단하지만 동일 클라우드 계정 사용 |
| **분리 스택** | Review Stacks와 프로덕션 스택을 분리하여 다른 클라우드 계정이나 설정 사용 가능 |
| **다중 Pulumi 프로그램** | Review Stack에 별도의 Pulumi 프로그램을 사용하여 공유 클러스터 등 다중 테넌트 개발 인프라 활용 |
| **Path Filter** | `paths` 설정으로 변경된 파일 경로에 따라 다른 Review Stack 템플릿 선택 |
| **GitHub Label 제어** | `reviewStackLabels` 설정으로 특정 Label이 있는 PR에서만 Review Stack 생성 |

### GitHub Label 제한 예제

```typescript
const reviewSettings = new pulumiservice.DeploymentSettings("reviewSettings", {
    organization: pulumi.getOrganization(),
    project: "<YOUR_PROJECT>",
    stack: "pr",
    github: {
        pullRequestTemplate: true,
        repository: "<YOUR_ORG>/<YOUR_REPO>",
        reviewStackLabels: ["review-stack", "reviewStack", "rs"],
    },
    sourceContext: {
        git: {
            branch: "refs/heads/main",
            repoDir: "<YOUR_REPO_DIR>",
        },
    },
});
```

---

## Deployment Runner Images

> https://www.pulumi.com/docs/deployments/deployments/runs/images/

Pulumi Cloud는 Pulumi 프로그램을 *Deployment Executor Image*라는 컨테이너 이미지 내에서 실행한다.

### 기본 Executor Image

기본적으로 모든 배포는 `pulumi/pulumi` 이미지(Debian 기반)에서 실행된다.

| 포함 항목 | 설명 |
|---|---|
| `pulumi` CLI | `$PATH`에 설치됨 |
| 언어 런타임 | Node.js, Python, Go, .NET, Java의 LTS 버전 |
| 빌드 도구 | `git`, `curl` 및 각 런타임의 패키지 매니저 |

### 커스텀 접근 방식 선택

| 접근 방식 | 적합한 경우 | 비용 |
|---|---|---|
| **Pre-Run Commands** | 일회성 도구, 스크립트, 빠른 실험 | 매 배포마다 설치 실행, 설정 시간 추가 |
| **Custom Executor Image** | 매 배포마다 사용하는 안정적인 도구 세트 | 새 Runner에서 콜드 스타트가 느림, 이미지 유지 보수 필요 |

### 베이스 이미지 선택

| 베이스 이미지 | 크기 (약) | 포함 항목 | 적합한 경우 |
|---|---|---|---|
| `pulumi/pulumi-base` | ~200 MB | `pulumi` CLI만 | 자체 언어 런타임 설치, 단일 언어 버전 사용 |
| `pulumi/pulumi-nodejs` 등 | 200~400 MB | `pulumi` + 1개 언어 런타임 | 단일 언어 프로젝트 |
| `pulumi/pulumi` | ~2 GB | `pulumi` + 모든 언어 런타임 | 다중 언어 프로젝트 |

### 커스텀 이미지 요구 사항

- Linux 이미지여야 함
- `curl`이 `$PATH`에 포함되어야 함
- `pulumi` CLI가 `$PATH`에 포함되어야 함
- 관련 언어 런타임이 포함되어야 함
- glibc 기반 libc 사용 (Alpine 등 musl-only 이미지는 미지원)

### 커스텀 이미지 예제

```dockerfile
FROM pulumi/pulumi-base:3.236.0

RUN apt-get update \
    && apt-get install -y --no-install-recommends gh \
    && rm -rf /var/lib/apt/lists/*
```

> 기본 이미지는 Pulumi 호스트 워크플로 러너에 미리 캐시되어 있지만, 커스텀 이미지는 그렇지 않다. 콜드 스타트 시간을 줄이려면 작은 베이스 이미지를 사용하고, Pulumi 호스트 러너는 AWS `us-west-2`에서 실행되므로 동일 리전의 ECR을 사용하는 것이 좋다.

---

## Customer-Managed Workflow Runners

> https://www.pulumi.com/docs/deployments/deployments/runs/customer-managed-agents/

Customer-Managed Workflow Runners를 사용하면 워크플로 러너를 자체 호스팅할 수 있다. Business Critical 에디션에서 사용 가능하다.

### 이점

| 이점 | 설명 |
|---|---|
| **어디서나 호스팅** | 완전히 프라이빗한 VPC 내에서도 인프라 관리 가능 |
| **임의 하드웨어/환경** | 원하는 하드웨어와 환경 구성 가능 (현재 Linux, macOS 지원) |
| **혼합 사용** | 개발 스택은 Pulumi 호스팅, 프라이빗 네트워크 인프라는 자체 호스팅 혼합 가능 |
| **다중 풀** | 여러 워크플로 러너 풀 설정, 스택을 특정 풀에 할당, 동적 스케일링 (최대 150개 동시 워크플로) |
| **컴플라이언스** | 클라우드 제공자 자격 증명이 프라이빗 네트워크를 떠나지 않음 |

### 지원 워크플로 유형

워크플로 러너는 기본적으로 모든 유형이 활성화되어 있다. `enabled_workflow_types`로 특정 유형만 처리하도록 제한할 수 있다.

| 유형 | 설명 |
|---|---|
| `deployment` | Pulumi 배포 작업 |
| `insights_scan` | Insights Discovery 스캔 |
| `policy_evaluation` | 정책 평가 |

### 스케일링

각 워크플로 러너 프로세스는 한 번에 **하나의 배포**와 선택적으로 **하나의 Insights 스캔 또는 정책 평가**를 병렬로 실행한다. 병렬 처리 능력을 높이려면 풀에 더 많은 러너 인스턴스를 추가해야 한다.

| 패턴 | 설명 |
|---|---|
| **Long-running runners** | 여러 인스턴스를 지속적으로 실행 (예: Kubernetes Deployment 복제본) |
| **Ephemeral runners** | `single_run: true` 설정 후 Kubernetes Job/CronJob으로 작업별 러너 시작 |
| **Specialized pools** | `enabled_workflow_types`로 배포 전용, 스캔 전용 등 분리 |

### 조직 기본 풀

풀을 조직 기본값으로 지정할 수 있다. 해상 순서는 다음과 같다.

1. 스택, 계정, 또는 정책 그룹에 명시적으로 구성된 풀
2. 조직 기본 풀 (설정된 경우)
3. Pulumi Hosted Pool

### 구성 파일 참조 (`pulumi-workflow-agent.yaml`)

| 설정 | 기본값 | 설명 |
|---|---|---|
| `token` | (필수) | 워크플로 러너 풀 생성 시 제공되는 Pulumi 토큰. OIDC 사용 시 불필요 |
| `service_url` | `https://api.pulumi.com` | Self-Hosted Pulumi 사용 시 API 도메인 |
| `working_directory` | 바이너리 위치 | Runner 바이너리를 로드할 기본 경로 |
| `deploy_target` | `docker` | 실행 환경 (`docker` 또는 `kubernetes`) |
| `single_run` | `false` | `true` 시 단일 작업 완료 후 종료 |
| `pull_image` | `true` | 매 작업마다 이미지를 레지스트리에서 Pull (Docker 전용) |
| `enabled_workflow_types` | 모든 유형 | 처리할 워크플로 유형 제한 (`deployment`, `insights_scan`, `policy_evaluation`) |
| `env_forward_allowlist` | `[]` | 호스트에서 러너 컨테이너로 전달할 환경 변수 목록 |
| `polling_interval` | `1m` | 새 작업 확인 폴링 간격 (Go Duration 문법) |
| `request_timeout` | `30s` | Pulumi Cloud API 요청 타임아웃 |
| `request_retry_count` | `2` | Rate Limit 또는 일시적 실패 시 최대 재시도 횟수 |
| `http_server_port` | `8080` | Health Check 엔드포인트 포트 |

> 모든 설정은 `PULUMI_AGENT_` 접두사와 대문자 키 이름으로 환경 변수로도 제공할 수 있다. (예: `token` -> `PULUMI_AGENT_TOKEN`). 환경 변수가 구성 파일 값보다 우선한다.

---

## Dependency Caching

Pulumi 관리 에이전트를 사용할 때 Dependency Caching으로 배포 속도를 높일 수 있다. 첫 배포 시 Lock File 기반으로 의존성을 감지하여 압축 저장하고, 이후 배포에서 아카이브를 가져와 압축 해제한다.

### 지원 런타임

| 런타임 | 요구 사항 |
|---|---|
| .NET | 특별한 구성 불필요 |
| Python | `requirements.txt` 필요 |
| Go | `go.mod` 및 `go.sum` 필요 |
| Node.js (npm) | `package-lock.json` 필요 |
| Node.js (yarn) | `yarn.lock` 필요 |

> 캐시는 프로젝트 수준에서 공유되며, 고객 간에 절대 공유되지 않는다. Customer-Managed Agent Pool에서는 사용할 수 없다.

---

## 기타 설정 옵션

### 의존성 설치 건너뛰기

기본적으로 배포 실행기가 언어별 기본 의존성 매니저(`npm`, `virtualenv` 등)로 의존성을 설치한다. `yarn`이나 다른 버전의 `node`를 사용하려면 기본 설치 단계를 건너뛰고(Under Advanced Settings) Pre-Run 명령으로 직접 설치할 수 있다.

### 중간 배포 건너뛰기

여러 배포가 Push될 때 기본적으로 순차적으로 실행된다. `Skip intermediate deployments` 설정을 활성화하면 동일 유형의 중간 배포를 모두 건너뛰고 가장 최근 배포만 실행한다.

### Role Assignment

Deployment Settings에서 스택 토큰에 조직 역할을 할당할 수 있다. 역할이 선택되지 않으면 배포는 해당 스택에만 액세스할 수 있다. 다른 스택 참조, 환경 액세스, 조직 리소스 관리가 필요한 경우 적절한 역할을 선택해야 한다.

---

## Post-Deployment Automation

> https://www.pulumi.com/docs/deployments/deployments/using/post-automation/

배포가 완료된 후 추가 작업이나 배포를 트리거하는 두 가지 방법이 있다.

### Deployment Webhook Destinations

특정 이벤트(예: `update succeeded`) 발생 시 다른 스택의 Create Deployment API로 이벤트를 자동 전달한다.

**TypeScript 예제:**

```typescript
import * as pulumiservice from "@pulumi/pulumiservice";

const databaseWebhook = new pulumiservice.Webhook("databaseWebhook", {
    organizationName: "<YOUR_ORG>",
    projectName: "network",
    stackName: "prod",
    format: pulumiservice.WebhookFormat.PulumiDeployments,
    payloadUrl: "database/prod",
    active: true,
    displayName: "deploy-database",
    filters: [pulumiservice.WebhookFilters.UpdateSucceeded],
});
```

### Pulumi Auto Deploy Package (Preview)

`@pulumi/auto-deploy` 패키지를 사용하여 스택 간의 의존성을 선언적으로 표현하고, 필요한 Deployment Webhook을 자동으로 관리할 수 있다.

**TypeScript 예제:**

```typescript
import * as autodeploy from "@pulumi/auto-deploy";
import * as pulumi from "@pulumi/pulumi";

const organization = pulumi.getOrganization();
const project = "dependency-example";

const b = new autodeploy.AutoDeployer("auto-deployer-b", {
    organization,
    project,
    stack: "b",
    downstreamRefs: [d.ref, e.ref, f.ref],
});

const a = new autodeploy.AutoDeployer("auto-deployer-a", {
    organization,
    project,
    stack: "a",
    downstreamRefs: [b.ref, c.ref],
});
```

**Python 예제:**

```python
import pulumi
import pulumi_auto_deploy as auto_deploy

organization = pulumi.get_organization()
project = "dependency-example"

b = auto_deploy.AutoDeployer("b",
    organization=organization,
    project=project,
    stack="b",
    downstream_refs=[d.ref, e.ref, f.ref])

a = auto_deploy.AutoDeployer("a",
    organization=organization,
    project=project,
    stack="a",
    downstream_refs=[b.ref, c.ref])
```
