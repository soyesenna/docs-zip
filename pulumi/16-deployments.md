# Pulumi Deployments

> https://www.pulumi.com/docs/deployments/
>
> https://www.pulumi.com/docs/deployments/deployments/
>
> https://www.pulumi.com/docs/deployments/deployments/get-started/
>
> https://www.pulumi.com/docs/deployments/deployments/using/
>
> https://www.pulumi.com/docs/deployments/deployments/using/settings/
>
> https://www.pulumi.com/docs/deployments/deployments/using/triggers/
>
> https://www.pulumi.com/docs/deployments/deployments/using/post-automation/
>
> https://www.pulumi.com/docs/deployments/deployments/permissions/
>
> https://www.pulumi.com/docs/deployments/deployments/oidc/
>
> https://www.pulumi.com/docs/deployments/deployments/oidc/aws/
>
> https://www.pulumi.com/docs/deployments/deployments/oidc/azure/
>
> https://www.pulumi.com/docs/deployments/deployments/oidc/gcp/
>
> https://www.pulumi.com/docs/deployments/deployments/cloud-credentials/
>
> https://www.pulumi.com/docs/deployments/deployments/drift/
>
> https://www.pulumi.com/docs/deployments/deployments/schedules/
>
> https://www.pulumi.com/docs/deployments/deployments/ttl/
>
> https://www.pulumi.com/docs/deployments/deployments/review-stacks/
>
> https://www.pulumi.com/docs/deployments/deployments/runs/
>
> https://www.pulumi.com/docs/deployments/deployments/runs/images/
>
> https://www.pulumi.com/docs/deployments/deployments/runs/customer-managed-agents/
>
> https://www.pulumi.com/docs/deployments/webhooks/
>
> https://www.pulumi.com/docs/deployments/pulumi-button/

Pulumi Deployments & Workflows는 인프라스트럭처 프로젝트 관리, 배포 자동화, 워크플로 통합을 위한 운영 도구 모음이다. **Deployments**는 IaC를 위해 목적에 맞게 구축된 관리형 CI/CD 플랫폼으로, 관리형 컴퓨트, 안전한 시크릿 처리, 버전 관리 시스템과의 깊은 통합을 제공한다. **Webhooks**는 스택 업데이트, 배포, Drift Detection, 정책 위반 등에 대응하여 외부 시스템과 워크플로를 트리거한다. **Deploy with Pulumi Button**은 GitHub 리포지토리, Gist 또는 웹 페이지에서 원클릭 인프라 배포를 가능하게 하는 임베디드 배포 버튼이다.

### 주요 기능

#### Managed Infrastructure CI/CD

- **Zero Touch CI/CD**: 앱 팀이 Pulumi Cloud의 New Project Wizard에서 템플릿을 선택하여 몇 분 안에 인프라 배포
- **Git 통합**: PR 생성 시 자동 `pulumi preview`, 병합 시 `pulumi up` 실행
- **Live Preview Environments**: 각 PR이 실제 인프라가 포함된 Review Stack을 자동 생성하여 병합 전 변경 사항 검증
- **Secure by Default**: Pulumi ESC 통합으로 시크릿과 클라우드 자격 증명을 안전하게 처리

#### Beyond CI/CD

- **Drift Detection**: 인프라가 원하는 상태에서 벗어났을 때 자동 감지
- **Scheduled Operations**: Cron 표현식으로 정기적인 Pulumi 작업 실행
- **Temporary Infrastructure**: TTL Stacks를 통해 개발/테스트 환경 자동 해제
- **Custom Compute**: Customer-Managed Workflow Runners로 자체 인프라에서 Pulumi 작업, Insights Discovery 스캔, 정책 평가 실행

#### Platform Engineering

- **REST API**: 인프라 작업을 프로그래밍 방식으로 자동화하여 커스텀 워크플로 및 셀프 서비스 플랫폼 구축
- **Deployment Settings**: 소스 코드, 자격 증명, 환경 변수 등 모든 배포 요구 사항을 단일 구성으로 정의 (UI 또는 Pulumi Cloud Provider로 관리)
- **Multiple Triggers**: Git Push, REST API, UI 버튼, 스케줄 등 다양한 방식으로 배포 트리거
- **Best Practices Built-in**: 대규모 인프라 자동화를 위한 Deployment 패턴 제공

### 핵심 구성 요소

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
| **Push to Deploy** | PR 생성 시 `pulumi preview`, 병합 시 `pulumi up` 자동 실행. Git Tag Push 시에도 배포 가능. VCS 통합 필수 | Update, Preview |
| **Review Stacks** | PR이 열릴 때마다 임시 스택을 자동 생성하고 배포. PR 병합/종료 시 자동 삭제 | Update, Destroy |
| **Scheduled Deployments** | Cron 표현식으로 정기적으로 Pulumi 작업 실행 | Update, Preview, Refresh, Destroy |
| **TTL Stacks** | 지정된 시간이 지난 후 스택 자동 삭제. 임시 인프라 비용 및 보안 관리 | Destroy |
| **REST API** | HTTP 요청으로 프로그래밍 방식 배포 트리거. Deployment Settings를 요청 본문에 전달 가능 | Update, Preview, Refresh, Destroy |
| **Deployment Webhooks** | 다른 스택의 이벤트 발생 시 Deployment 트리거 | Update, Preview, Refresh, Destroy |

> 모든 트리거가 모든 작업을 지원하지는 않는다.

### Push to Deploy 상세

Push to Deploy는 조직에 VCS(Version Control System) 통합이 구성되어 있어야 한다. 모든 Pulumi VCS 통합(GitHub, GitLab, Bitbucket, Azure DevOps, Custom VCS)에서 지원된다. 통합은 리포지토리에 대한 읽기 권한이 필요하며, Pulumi 프로그램을 클론하고 merge 커밋을 수신하여 git push 시 자동으로 배포를 트리거한다.

- **PR Preview**: 특정 git 브랜치(예: `main`)에 대한 PR이 열리면 스택(예: `dev`)에 대해 `pulumi preview` 실행. VCS 통합이 PR에 preview 결과를 댓글로 게시
- **Merge Deploy**: 특정 브랜치에 변경 사항이 병합되면 `pulumi up` 실행. 공유 개발/QA 환경의 지속적 배포에 유용

### Deploying on Git Tags

브랜치 Push 외에도 git tag가 push될 때 `pulumi up`을 실행할 수 있다. 이는 릴리즈 기반 워크플로우에 적합하다. 예를 들어 `v1.2.0`과 같은 버전 tag를 push할 때만 배포한다.

| 설정 | 설명 |
|---|---|
| `deployTags` | Tag Push 시 배포 활성화 여부 (boolean) |
| `tagFilters` | Tag 이름에 대한 Glob 패턴 목록 |

Tag trigger는 모든 VCS 통합에서 지원된다. Tag 삭제는 배포를 트리거하지 않는다. Tag push로 배포가 트리거되면 `PULUMI_CI_TAG_NAME` 환경 변수에 Tag 이름이 설정된다.

> GitLab 통합이 이 기능 이전에 생성된 경우, Tag push 이벤트를 활성화하려면 기존 GitLab 그룹 웹훅에서 Tag push events를 활성화해야 한다.

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

> 위 "Deploying on Git Tags" 섹션 참조. Tag 필터링은 tag trigger와 함께 사용되어 어떤 tag 이름이 배포를 트리거할지 제어한다.

| 패턴 | 설명 |
|---|---|
| `v*` | `v`로 시작하는 모든 Tag 포함 |
| `v1.0.0` | 해당 Tag만 정확히 일치 |
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

## Deployment Runner Pools

Pulumi Deployments를 사용할 때 워크플로가 실행되는 위치를 선택할 수 있다.

| 풀 유형 | 설명 |
|---|---|
| **Pulumi Hosted Pool** | Pulumi가 관리하며 모든 Pulumi Cloud 고객이 사용 가능 |
| **Customer-Managed Workflow Runners** | 자체 호스팅 러너로 프라이빗 네트워크와 리소스에 접근 가능. 배포, Insights Discovery 스캔, 정책 평가 지원 |

스택에 명시적으로 풀이 구성되지 않은 경우 해석 순서는 다음과 같다.

1. 스택, 계정, 또는 정책 그룹에 명시적으로 구성된 풀
2. 조직 기본 풀 (설정된 경우)
3. Pulumi Hosted Pool

> 자세한 내용은 [Customer-Managed Workflow Runners](#customer-managed-workflow-runners) 섹션을 참조한다.

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

> OIDC 외에도 Pulumi ESC를 통한 클라우드 자격 증명 제공 방식이 있다. 자세한 비교는 [Cloud Credentials](#cloud-credentials) 섹션을 참조한다.

### 신뢰 관계 구성

클라우드 제공자는 OIDC 토큰의 클레임을 신뢰 관계 조건과 대조하여 자격 증명 교환을 승인한다. 신뢰 관계 구성은 클라우드 제공자마다 다르지만, 일반적으로 최소한 Audience, Subject, Issuer 클레임을 사용한다.

| 클레임 | 구성 용도 |
|---|---|
| **Issuer** | 토큰이 올바르게 서명되었는지 검증. 발급자의 공개 서명 키를 가져와 토큰 서명 확인 |
| **Audience** | 배포와 연결된 조직 이름. 특정 조직으로 자격 증명 제한에 사용 |
| **Subject** | 배포에 대한 다양한 정보 포함. 특정 조직, 프로젝트, 스택 등으로 자격 증명 제한에 사용. Subject와 커스텀 클레임은 세밀한 조건 설정에 특히 유용 |
| **Custom Claims** | Subject와 동일한 정보를 개별 필드로 제공. 클라우드 제공자가 커스텀 클레임 기반 신뢰 관계를 지원하는 경우 사용 |

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
3. Pulumi 콘솔에서 OIDC 활성화 (Role ARN, Session Name, 선택적으로 Session Duration 및 Policy ARNs 입력)

**IAM 권한 권장 사항:**

많은 Pulumi 프로그램이 IAM Role, Policy, Instance Profile(예: Lambda, ECS, EKS용)을 생성하므로 IAM 권한이 필요하다. `AdministratorAccess`는 IAM을 포함한 모든 서비스를 포함하는 가장 간단한 정책이지만, 대부분의 워크로드에 필요 이상으로 광범위하다.

| 권한 접근 방식 | 설명 |
|---|---|
| **PowerUserAccess + IAMFullAccess** | 광범위한 서비스 액세스와 IAM 권한을 제공하지만 AWS Organizations 관리는 제외 |
| **커스텀 최소 권한 정책** | Pulumi 프로그램이 사용하는 AWS 서비스와 IAM 작업(예: `ec2:*`, `s3:*`, `iam:CreateRole`, `iam:AttachRolePolicy`, `iam:PassRole`)으로만 범위 제한 |
| **권한 경계(Permissions Boundary)** | 더 광범위한 역할 정책을 사용하되 IAM 권한 경계를 연결하여 Pulumi가 생성하는 IAM 엔터티의 권한을 제한. 권한 에스컬레이션 방지 |

> Pulumi Deployments OIDC 구성의 Policy ARNs 필드를 사용하여 세션의 권한을 추가로 제한할 수도 있다.

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

**Truncate 예시:**

템플릿 `${organization.name}-${project.name}-${stack.name}-${deployment.id}`에서 `organization.name = "pulumi-local"`, `project.name = "test-nocode-rtct-3"`, `stack.name = "dev"`, `deployment.id = "806bf21f-444f-4825-a80c-afd12cd2526a"`인 경우, 전체 길이는 72자가 된다. Pulumi는 세 가지 이름 변수를 잘라 64자에 맞추며, deployment UUID는 완전히 보존한다.

```
pulumi-loca-test-nocode-dev-806bf21f-444f-4825-a80c-afd12cd2526a
```

결과는 정확히 64자이다. `${organization.name}`과 `${project.name}`은 각각 11자로 잘리고, `${stack.name}`은 3자로 그대로 유지되며, deployment UUID는 전체가 보존된다.

**설정 후 환경 변수:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`. 원본 OIDC 토큰은 `PULUMI_OIDC_TOKEN` 환경 변수와 `/mnt/pulumi/pulumi.oidc` 파일에서도 사용 가능.

> AWS OIDC 구성 시 **Session Duration** 필드에 `"XhYmZs"` 형식으로 수임(Role Session) 기간을 제한할 수 있다. 예: `1h30m`. 또한 **Policy ARNs** 필드를 통해 세션 권한을 추가로 제한할 수도 있다.

### Azure OIDC 설정

Azure에서는 Microsoft Entra App Registration과 Workload Identity Federation을 사용하여 OIDC를 구성한다.

**Prerequisites:**

- Azure Portal에서 Microsoft Entra App Registration을 생성하고 구성할 수 있는 권한이 필요하다.
- 이 가이드는 공식 제공자 문서를 기반으로 한 단계별 지침을 제공한다. 최신 정확한 정보는 항상 [공식 Azure 문서](https://learn.microsoft.com/en-us/entra/identity-platform/howto-create-service-principal-portal)를 참조한다.

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

**설정 후 환경 변수:** `ARM_CLIENT_ID`, `ARM_TENANT_ID`, `ARM_SUBSCRIPTION_ID`. 원본 OIDC 토큰은 `PULUMI_OIDC_TOKEN` 환경 변수와 `/mnt/pulumi/pulumi.oidc` 파일에서도 사용 가능.

### GCP OIDC 설정

Google Cloud에서는 Workload Identity Federation을 사용하여 OIDC를 구성한다.

**Prerequisites:**

- [필요 API가 활성화된 Google Cloud 프로젝트](https://cloud.google.com/iam/docs/workload-identity-federation-with-other-providers#configure)가 필요하다.
- 이 가이드는 공식 제공자 문서를 기반으로 한 단계별 지침을 제공한다. 최신 정확한 정보는 항상 [공식 Google Cloud 문서](https://cloud.google.com/iam/docs/workload-identity-federation-with-other-providers)를 참조한다.

**구성 단계:**

1. Workload Identity Pool 및 Provider 생성 (Issuer: `https://api.pulumi.com/oidc`, Audience: Pulumi 조직 이름)
2. Service Account 생성 및 역할 할당
3. Pool에 Service Account 액세스 권한 부여 (Subject 조건 필터 사용)
4. Pulumi 콘솔에서 OIDC 활성화 (Project Number, Workload Pool ID, Identity Provider ID, Service Account Email 입력)
5. (선택) 스택의 Google Cloud Region 입력 (보통 불필요)
6. (선택) Session Duration 필드에 `"XhYmZs"` 형식으로 임시 Google Cloud 자격 증명 기간 제한. 예: `1h30m`

> GCP 역시 Azure와 마찬가지로 Subject Claim에 와일드카드를 허용하지 않으므로, 각 작업 유형별로 별도의 Credential을 생성해야 한다.

**설정 후 환경 변수:** `GOOGLE_CREDENTIALS` (Credential Configuration 형식). 원본 OIDC 토큰은 `PULUMI_OIDC_TOKEN` 환경 변수와 `/mnt/pulumi/pulumi.oidc` 파일에서도 사용 가능.

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
| **버전 관리** | Environment Import를 특정 버전에 고정(pin)하여 변경 사항을 제어된 방식으로 롤아웃 가능 |

> 중요한 차이: Pulumi Deployments OIDC는 Pre-Run 명령을 포함한 전체 배포 프로세스에 적용되지만, Pulumi ESC 환경은 Pulumi IaC 작업(예: `pulumi up`)에만 적용된다. Pre-Run 명령에서 ESC를 사용하려면 `pulumi env run` 명령을 접두사로 사용해야 한다. 예를 들어 프라이빗 리포지토리에서 패키지를 설치하려면 다음과 같이 실행한다.
>
> ```bash
> pulumi env run my-esc-project/my-esc-environment -- npm install
> ```

---

## Drift Detection

> https://www.pulumi.com/docs/deployments/deployments/drift/

Drift Detection은 클라우드 환경의 실제 상태가 Pulumi Cloud에 저장된 예상 상태와 다른 경우 이를 식별하는 프로세스다. 수동 변경, 스크립트의 의도치 않은 결과, 또는 무단 변경 등으로 인해 발생할 수 있다.

Pulumi Cloud는 Drift가 감지된 경우 자동으로 수정하는 기능도 제공한다. 수정(Remediation)은 예상 상태와 일치하도록 필요한 변경을 적용하여 인프라를 일관되고 예측 가능하게 유지한다.

> Drift Detection 및 Remediation을 사용하려면 먼저 스택의 Deployment Settings를 구성해야 한다.

### Drift Tab UI

Pulumi Cloud 스택 페이지에 **Drift** 탭이 추가되어 있다. 스택이 현재 Drift 상태인 경우 탭에 경고 벨 아이콘이 표시된다. Drift 실행이 어떤 방식으로 시작되었든 모든 결과가 이 탭에 표시된다. 실행에서 Drift가 감지된 경우 변경 사항의 diff 요약이 정보 카드에 포함된다.

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

### Drift Detection 스케줄 구성 (UI)

Pulumi Cloud 콘솔에서 다음 단계로 구성한다.

1. 스택의 Deployment Settings가 구성되어 있는지 확인
2. **Stack > Settings > Schedules** 로 이동
3. **Drift** 선택
4. (선택) 자동 수정(auto-remediation) 활성화
5. Cron 표현식으로 스케줄 설정
6. 스케줄 저장

### Drift 알림 구성

Slack, MS Teams 등에 Drift 알림을 통합할 수 있다. Pulumi Webhooks 통합을 사용한다.

1. **Stack > Settings > Webhooks** 로 이동
2. 원하는 Webhook 형식 선택
3. Display name, 대상 URL, 선택적 Secret 입력
4. 모든 이벤트에 Drift 이벤트가 포함되며, Drift 이벤트만 필터링할 수도 있다.

### CLI에서 Drift Detection

`pulumi refresh --preview-only` 또는 `pulumi refresh`를 실행하면 Pulumi Cloud에서 Drift 실행으로 간주된다. 실행 완료 후 스택의 Drift 탭에서 결과를 확인할 수 있다.

### REST API로 Drift 설정

Drift Detection과 Remediation은 REST API를 통해 프로그래밍 방식으로 구성할 수 있다. 사용 가능한 엔드포인트는 다음과 같다.

| 작업 | 엔드포인트 | 메서드 |
|---|---|---|
| Drift 스케줄 생성 | `/api/stacks/{org}/{project}/{stack}/deployments/drift/schedules` | POST |
| Drift 스케줄 조회 | `/api/stacks/{org}/{project}/{stack}/deployments/drift/schedules` | GET |
| Drift 스케줄 수정 또는 삭제 | `/api/stacks/{org}/{project}/{stack}/deployments/drift/schedules/{scheduleId}` | PATCH / DELETE |
| Drift 스케줄 일시 중지 또는 재개 | `/api/stacks/{org}/{project}/{stack}/deployments/drift/schedules/{scheduleId}` | PATCH |
| 조직의 모든 스케줄 나열 | `/api/orgs/{org}/schedules` | GET |

**Drift Detection 및 Remediation 스케줄 생성 예시:**

```bash
curl -H "Accept: application/vnd.pulumi+json" \
     -H "Content-Type: application/json" \
     -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
     --request POST \
     --data '{"scheduleCron":"0 0 * * *","autoRemediate":true}' \
     https://api.pulumi.com/api/stacks/{organization}/{project}/{stack}/deployments/drift/schedules
```

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

**Go:**

```go
driftDetectionSchedule, err := pulumiservice.NewDriftSchedule(ctx, "driftDetectionSchedule", &pulumiservice.DriftScheduleArgs{
    Organization: pulumi.String("<YOUR_ORG>"),
    Project:      pulumi.String("<YOUR_PROJECT>"),
    Stack:        pulumi.String("<YOUR_STACK>"),
    ScheduleCron: pulumi.String("0 0 * * *"),
    AutoRemediate: pulumi.Bool(true),
})
```

**C#:**

```csharp
var driftSchedule = new PulumiService.DriftSchedule("driftDetectionSchedule", new PulumiService.DriftScheduleArgs
{
    Organization = "<YOUR_ORG>",
    Project = "<YOUR_PROJECT>",
    Stack = "<YOUR_STACK>",
    ScheduleCron = "0 0 * * *",
    AutoRemediate = true,
});
```

**Java:**

```java
var driftSchedule = new Webhook("driftDetectionSchedule", WebhookArgs.builder()
    .organization("<YOUR_ORG>")
    .project("<YOUR_PROJECT>")
    .stack("<YOUR_STACK>")
    .scheduleCron("0 0 * * *")
    .autoRemediate(true)
    .build());
```

**YAML:**

```yaml
resources:
  driftDetectionSchedule:
    type: pulumiservice:index:DriftSchedule
    properties:
      organization: <YOUR_ORG>
      project: <YOUR_PROJECT>
      stack: <YOUR_STACK>
      scheduleCron: "0 0 * * *"
      autoRemediate: true
```

---

## Scheduled Deployments

> https://www.pulumi.com/docs/deployments/deployments/schedules/

Scheduled Deployments는 Cron 표현식을 사용하여 정기적으로 클라우드 작업을 자동화하는 기능이다. Pulumi Deployments 동시성 제한이 적용되며, 스택 배포 일시 중지 시 예약된 배포도 대기열에 추가된다.

### REST API로 스케줄 설정

| 작업 | 엔드포인트 | 메서드 |
|---|---|---|
| 스케줄 생성 | `/api/stacks/{org}/{project}/{stack}/deployments/schedules` | POST |
| 스택의 스케줄 조회 | `/api/stacks/{org}/{project}/{stack}/deployments/schedules` | GET |
| 스케줄 수정 또는 삭제 | `/api/stacks/{org}/{project}/{stack}/deployments/schedules/{scheduleId}` | PATCH / DELETE |
| 스케줄 일시 중지 또는 재개 | `/api/stacks/{org}/{project}/{stack}/deployments/schedules/{scheduleId}` | PATCH |
| 조직의 모든 스케줄 나열 | `/api/orgs/{org}/schedules` | GET |

**스케줄 생성 예시:**

```bash
curl \
  -H "Accept: application/vnd.pulumi+8" \
  -H "Content-Type: application/json" \
  -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
  --request POST \
  --data '{ "scheduleCron":"0 0 * * *", "request": { "operation": "update" } }' \
  https://api.pulumi.com/api/stacks/{organization}/{project}/{stack}/deployments/schedules
```

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

**Go:**

```go
rawSchedule, err := pulumiservice.NewDeploymentSchedule(ctx, "rawSchedule", &pulumiservice.DeploymentScheduleArgs{
    Organization:    pulumi.String("<YOUR_ORG>"),
    Project:         pulumi.String("<YOUR_PROJECT>"),
    Stack:           pulumi.String("<YOUR_STACK>"),
    ScheduleCron:    pulumi.String("0 0 * * *"),
    PulumiOperation: pulumiservice.PulumiOperationUpdate,
})
```

**C#:**

```csharp
var rawSchedule = new PulumiService.DeploymentSchedule("rawSchedule", new PulumiService.DeploymentScheduleArgs
{
    Organization = "<YOUR_ORG>",
    Project = "<YOUR_PROJECT>",
    Stack = "<YOUR_STACK>",
    ScheduleCron = "0 0 * * *",
    PulumiOperation = PulumiService.PulumiOperation.Update,
});
```

**Java:**

```java
var rawSchedule = new DeploymentSchedule("rawSchedule", DeploymentScheduleArgs.builder()
    .organization("<YOUR_ORG>")
    .project("<YOUR_PROJECT>")
    .stack("<YOUR_STACK>")
    .scheduleCron("0 0 * * *")
    .pulumiOperation(com.pulumi.pulumiservice.PulumiOperation.update())
    .build());
```

**YAML:**

```yaml
resources:
  rawSchedule:
    type: pulumiservice:index:DeploymentSchedule
    properties:
      organization: <YOUR_ORG>
      project: <YOUR_PROJECT>
      stack: <YOUR_STACK>
      scheduleCron: "0 0 * * *"
      pulumiOperation: update
```

---

## TTL Stacks

> https://www.pulumi.com/docs/deployments/deployments/ttl/

TTL(Time-to-Live) Stacks은 지정된 날짜/시간 이후에 스택을 자동으로 삭제하는 기능이다. 개발 환경 등 임시 클라우드 리소스가 자동으로 해제되어 비용을 절감하고 보안 태세를 개선할 수 있다.

### Delete After Destroy

TTL 스케줄 구성 시 "Delete After Destroy" 옵션을 활성화하면 `pulumi destroy` 실행 후 스택 자체도 완전히 삭제된다. 이 옵션은 임시 환경을 완전히 정리할 때 유용하다.

### REST API로 TTL 설정

| 작업 | 엔드포인트 | 메서드 |
|---|---|---|
| TTL 스케줄 생성 | `/api/stacks/{org}/{project}/{stack}/deployments/ttl/schedules` | POST |
| 스택의 TTL 스케줄 조회 | `/api/stacks/{org}/{project}/{stack}/deployments/ttl/schedules` | GET |
| TTL 스케줄 수정 또는 삭제 | `/api/stacks/{org}/{project}/{stack}/deployments/ttl/schedules/{scheduleId}` | PATCH / DELETE |
| TTL 스케줄 일시 중지 또는 재개 | `/api/stacks/{org}/{project}/{stack}/deployments/ttl/schedules/{scheduleId}` | PATCH |
| 조직의 모든 스케줄 나열 | `/api/orgs/{org}/schedules` | GET |

**TTL 스케줄 생성 예시:**

```bash
curl -H "Accept: application/vnd.pulumi+json" \
     -H "Content-Type: application/json" \
     -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
     --request POST \
     --data '{"timestamp":"2024-12-31T23:59:59Z","deleteAfterDestroy":true}' \
     https://api.pulumi.com/api/stacks/{organization}/{project}/{stack}/deployments/ttl/schedules
```

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

**Go:**

```go
ttlSchedule, err := pulumiservice.NewTtlSchedule(ctx, "ttlSchedule", &pulumiservice.TtlScheduleArgs{
    Organization: pulumi.String("<YOUR_ORG>"),
    Project:      pulumi.String("<YOUR_PROJECT>"),
    Stack:        pulumi.String("<YOUR_STACK>"),
    Timestamp:    pulumi.String("2024-01-01T00:00:00Z"),
})
```

**C#:**

```csharp
var ttlSchedule = new PulumiService.TtlSchedule("ttlSchedule", new PulumiService.TtlScheduleArgs
{
    Organization = "<YOUR_ORG>",
    Project = "<YOUR_PROJECT>",
    Stack = "<YOUR_STACK>",
    Timestamp = "2024-01-01T00:00:00Z",
});
```

**Java:**

```java
var ttlSchedule = new TtlSchedule("ttlSchedule", TtlScheduleArgs.builder()
    .organization("<YOUR_ORG>")
    .project("<YOUR_PROJECT>")
    .stack("<YOUR_STACK>")
    .timestamp("2024-01-01T00:00:00Z")
    .build());
```

**YAML:**

```yaml
resources:
  ttlSchedule:
    type: pulumiservice:index:TtlSchedule
    properties:
      organization: <YOUR_ORG>
      project: <YOUR_PROJECT>
      stack: <YOUR_STACK>
      timestamp: "2024-01-01T00:00:00Z"
```

---

## Review Stacks

> https://www.pulumi.com/docs/deployments/deployments/review-stacks/

Review Stacks는 풀 리퀘스트가 열릴 때 자동으로 생성되고, 각 커밋마다 업데이트되며, PR이 병합되거나 닫힐 때 삭제되는 임시 클라우드 환경이다. GitHub, GitLab, Azure DevOps, Bitbucket 통합에서 모두 작동한다. (Custom VCS 통합에서는 사용할 수 없다.)

### 구성 단계

1. 관례적으로 `pr`라는 이름의 새 스택을 만들고 `Pulumi.pr.yaml` 설정 파일 생성
2. 스택에 Deployment Settings 구성
3. `pullRequestTemplate` Deployment Setting을 `true`로 설정

### REST API로 구성

Deployments REST API를 사용하여 프로그래밍 방식으로 Review Stacks를 구성할 수 있다.

```bash
curl -i -XPOST -H "Content-Type: application/json" -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
--location "https://api.pulumi.com/api/stacks/org/project/stack/deployments/settings" \
-d '{
  "gitHub":{
    "pullRequestTemplate": true
  }
}'
```

### Config 활용 패턴

각 PR 템플릿 스택에는 해당하는 Pulumi 설정 파일이 있으며, 소스 제어에 체크인할 수 있다. 관례적으로 `Pulumi.pr.yaml`이라고 한다. PR의 일부로 Review Stack 설정 값을 수정할 수 있으며, 수정된 설정이 Review Stack 배포에 사용된다.

예를 들어 프로덕션 설정(`Pulumi.production.yaml`)을 다음과 같이 구성하고:

```yaml
config:
  aws:region: us-west-2
  webserver:apiServiceDesiredCount: "32"
  webserver:clusterInstanceType: m6g.xlarge
  webserver:clusterNumInstances: "16"
```

Review Stack 설정(`Pulumi.pr.yaml`)에서 클라우드 비용 절감을 위해 축소할 수 있다:

```yaml
config:
  aws:region: us-west-2
  webserver:apiServiceDesiredCount: "2"
  webserver:clusterInstanceType: t3.large
  webserver:clusterNumInstances: "1"
```

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

### Path Filter 기반 템플릿 선택

변경된 코드의 종류에 따라 Review Stack의 동작을 다르게 하고 싶을 수 있다. 예를 들어 `migrations` 폴더의 변경은 마이그레이션 컨테이너를 빌드하고 실행해야 하지만, 그 외의 경우 이 단계를 건너뛰어 배포 시간을 단축할 수 있다. Path Filter와 여러 Review Stack 템플릿을 조합하여 이를 구현할 수 있다. PR이 열리면 Pulumi Deployments가 코드 변경을 평가하고 Path Filter 일치 여부에 따라 사용할 템플릿을 선택한다.

```typescript
// 마이그레이션 변경이 없는 경우 사용할 템플릿
const prSettings = new pulumiservice.DeploymentSettings("prSettings", {
    organization: pulumi.getOrganization(),
    project: "<YOUR_PROJECT>",
    stack: "pr",
    github: {
        pullRequestTemplate: true,
        repository: "<YOUR_ORG>/<YOUR_REPO>",
        paths: ["!migrations/*"], // 마이그레이션 미변경 시
    },
    sourceContext: {
        git: {
            branch: "refs/heads/main",
            repoDir: "<YOUR_REPO_DIR>",
        },
    },
});

// 마이그레이션 변경이 있는 경우 사용할 템플릿
const prMigrationSettings = new pulumiservice.DeploymentSettings("prMigrationSettings", {
    organization: pulumi.getOrganization(),
    project: "<YOUR_PROJECT>",
    stack: "pr-migrations",
    github: {
        pullRequestTemplate: true,
        repository: "<YOUR_ORG>/<YOUR_REPO>",
        paths: ["migrations/*"], // 마이그레이션 변경 시
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

## Deployment Runs

> https://www.pulumi.com/docs/deployments/deployments/runs/

모든 Pulumi Deployment은 워크플로 러너의 컨테이너 이미지에서 실행된다. 실행 방식은 두 가지 설정으로 제어된다.

| 설정 | 설명 |
|---|---|
| **Image** | 기본적으로 Pulumi 관리 Linux 이미지. 추가 도구가 필요한 경우 커스텀 이미지 사용 |
| **Runner** | 기본적으로 Pulumi 호스팅. 자체 인프라에서 실행하려면 Customer-Managed |

### 하드웨어 및 운영체제 (Pulumi 호스팅 러너)

Pulumi 호스팅 워크플로 러너에서 실행 시 다음 리소스가 할당된다.

| 리소스 | 사양 |
|---|---|
| **vCPU** | 2 |
| **Memory** | 8 GB |
| **Disk** | 32 GB 볼륨 (Executor Image 및 의존성 캐시 제외 후 약 절반 사용 가능) |

기본 Executor Image를 사용할 경우 컨테이너의 운영체제는 호스트 OS와 관계없이 **Debian**이다. 커스텀 Executor Image를 제공하면 해당 이미지의 OS가 사용된다. 특정 OS, 패키지 매니저, 시스템 라이브러리에 의존하는 경우 사용하는 이미지와 일치시켜야 한다.

> 위 사양은 Pulumi 호스팅 워크플로 러너에 적용된다. Customer-Managed Workflow Runners는 자체 프로비저닝한 인프라에서 실행되므로 하드웨어와 OS를 직접 구성한다.

---

## Deployment Permissions

> https://www.pulumi.com/docs/deployments/deployments/permissions/

Deployment Permissions는 Pulumi Cloud 내에서 Deployment가 수행할 수 있는 작업(예: ESC 환경 열기, Stack References 액세스)을 제어한다. 클라우드 제공자의 리소스 관리 권한은 Cloud Credentials 및 OIDC 설정에서 다룬다.

### 기본 Deployment Permissions

기본적으로 Deployment에 부여되는 권한은 실행 방식에 따라 다르다.

| 실행 방식 | 권한 |
|---|---|
| **REST API / UI Actions 버튼** | 작업을 실행한 사용자의 권한을 상속 |
| **Git Push / Pull Request** | 스택 자체에 대해서만 admin 권한을 가진 임시 스택 토큰 사용. 다른 리소스에는 접근 불가 |

### 권한 모델 영향

| 조건 | 영향 |
|---|---|
| 조직 기본 스택 권한이 `NONE` | Git push/PR으로 생성된 Deployment는 Stack References에 접근할 수 없으며, 시도 시 실패 |
| 조직 기본 환경 권한이 `NONE` | Git push/PR으로 생성된 Deployment는 스택 설정 파일에 나열된 ESC Environments에 접근할 수 없음 |

### 추가 권한 부여

**Role Assignment (권장):**

스택의 Deployment Settings에서 **Settings > Deploy** 의 Role assignment 섹션에서 조직 역할을 선택할 수 있다. 역할이 할당되면 Deployment의 스택 토큰이 해당 역할의 권한을 상속하여 Stack References, ESC Environments, 조직 리소스에 접근할 수 있다. 세분화된 접근 제어를 위해 배포가 수행해야 하는 작업에 맞춰 특정 권한을 가진 **커스텀 역할(custom roles)**을 생성할 수도 있다. 조직 역할은 Roles 섹션에서 관리한다.

**PULUMI_ACCESS_TOKEN 환경 변수:**

또는 `PULUMI_ACCESS_TOKEN` 환경 변수를 개인, 팀, 또는 조직 토큰으로 설정할 수 있다. 이 환경 변수가 설정되면 Deployment 생성 방식(Rest API, Git Push 등)과 관계없이 해당 토큰의 권한이 사용된다.

**토큰 최소 필요 권한:**

| 권한 | 대상 |
|---|---|
| **WRITE** | 배포 중인 스택 |
| **READ** | Stack References를 사용하는 모든 스택 |
| **OPEN** | 스택 설정 파일에 나열된 모든 ESC Environment (전이적으로 Import된 환경 포함) |

### Pulumi ESC와 Deployments 통합

Pulumi ESC를 Deployments와 함께 사용하면 보안이 강화되고 자격 증명 관리가 간소화된다.

| 장점 | 설명 |
|---|---|
| **동적 단기 자격 증명** | 장기 정적 자격 증명 대신 동적 단기 자격 증명 사용 |
| **세분화된 제어** | 자격 증명 범위 및 권한에 대한 세분화된 제어 |
| **중앙 집중식 관리** | 자격 증명의 중앙 집중식 관리 |
| **노출 위험 감소** | 자격 증명 노출 위험 감소 |
| **완전한 감사 추적** | 자격 증명 사용에 대한 완전한 감사 추적 |

**ESC 구성 단계:**

1. 적절한 클라우드 제공자 자격 증명으로 ESC 환경 생성
2. Pulumi 스택 설정에서 해당 환경 참조
3. 적절한 권한을 가진 Pulumi 액세스 토큰 생성
4. Deployment Settings의 `PULUMI_ACCESS_TOKEN` 환경 변수에 토큰 추가

**Pre-Run 명령에서 ESC 사용:**

```bash
pulumi env run my-aws-env -- aws s3 ls
```

> 중요한 차이: Pulumi Deployments OIDC는 Pre-Run 명령을 포함한 전체 배포 프로세스에 적용되지만, Pulumi ESC 환경은 Pulumi IaC 작업(예: `pulumi up`)에만 적용된다. Pre-Run 명령에서 ESC를 사용하려면 `pulumi env run` 명령을 접두사로 사용해야 한다.

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
# Pulumi CLI 버전을 명시적으로 고정. :latest와 같은 부동 태그는
# 이미지와 로컬 CLI가 배포 간에 서로 달라질 수 있음.
FROM pulumi/pulumi-base:3.236.0

RUN apt-get update \
    && apt-get install -y --no-install-recommends gh \
    && rm -rf /var/lib/apt/lists/*
```

> 기본 이미지는 Pulumi 호스트 워크플로 러너에 미리 캐시되어 있지만, 커스텀 이미지는 그렇지 않다. 콜드 스타트 시간을 줄이려면 작은 베이스 이미지를 사용하고, Pulumi 호스트 러너는 AWS `us-west-2`에서 실행되므로 동일 리전의 ECR을 사용하는 것이 좋다. 또한 digest를 고정하면 러너가 태그 대신 레이어 캐시를 활용하여 재 fetch를 방지할 수 있다.

### 정적 자격 증명만 지원 (Static Credentials Only)

커스텀 이미지가 프라이빗 레지스트리에 있는 경우 Deployment Settings에 정적 사용자 이름과 비밀번호 자격 증명을 제공해야 한다. 커스텀 Executor Image에 대해서는 OIDC 및 IAM 역할 기반 Pull이 지원되지 않는다. 보안 모델이 단기 수명 레지스트리 자격 증명을 요구하는 경우, Customer-Managed Workflow Runners를 사용하여 자체 인프라에서 원하는 Pull 메커니즘을 구성할 수 있다.

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

### 복구 동작

Pulumi Cloud는 각 대기 중인 작업을 정확히 하나의 러너에 할당한다. 여러 러너가 동시에 같은 풀을 폴링하면 서비스가 각 대기 작업을 단일 러너에 전달하므로 동일한 작업이 두 러너에서 동시에 처리되지 않는다. 복구 동작은 워크플로 유형에 따라 다르다.

| 워크플로 유형 | 복구 동작 |
|---|---|
| **Insights 스캔 및 정책 평가** | 리스(Lease) 기반. 러너가 충돌하거나 연결이 끊어지면 리스가 만료되어 풀의 다른 러너가 작업을 이어받음 |
| **배포(Deployments)** | 재전달되지 않음. 러너가 배포 중 10분 동안 하트비트를 중지하면 다른 러너에 전달되지 않고 실패로 표시됨 |

### 조직 기본 풀

풀을 조직 기본값으로 지정할 수 있다. 해상 순서는 다음과 같다.

1. 스택, 계정, 또는 정책 그룹에 명시적으로 구성된 풀
2. 조직 기본 풀 (설정된 경우)
3. Pulumi Hosted Pool

### 구성 파일 참조 (`pulumi-workflow-agent.yaml`)

모든 설정은 `PULUMI_AGENT_` 접두사와 대문자 키 이름으로 환경 변수로도 제공할 수 있다. (예: `token` -> `PULUMI_AGENT_TOKEN`). 환경 변수가 구성 파일 값보다 우선한다. Duration 값은 Go Duration 문법을 사용한다 (`300ms`, `30s`, `1m`, `1h30m` 등).

**필수 설정:**

| 설정 | 기본값 | 설명 |
|---|---|---|
| `token` | (필수) | 워크플로 러너 풀 생성 시 제공되는 Pulumi 토큰. OIDC 사용 시 불필요 |

**선택 설정:**

| 설정 | 기본값 | 설명 |
|---|---|---|
| `service_url` | `https://api.pulumi.com` | Self-Hosted Pulumi 사용 시 API 도메인 |
| `working_directory` | 바이너리 위치 | Runner 바이너리를 로드할 기본 경로 |
| `shared_volume_directory` | `""` | 러너 컨테이너에 마운트할 임시 디렉토리를 생성하는 호스트 디렉토리. 비워두면 OS 기본 임시 위치 사용 |
| `deploy_target` | `docker` | 실행 환경 (`docker` 또는 `kubernetes`) |
| `single_run` | `false` | `true` 시 단일 작업 완료 후 종료. Kubernetes Job 등 일회성 러너에 유용 |
| `pull_image` | `true` | 매 작업마다 이미지를 레지스트리에서 Pull (Docker 전용) |
| `enabled_workflow_types` | 모든 유형 | 처리할 워크플로 유형 제한 (`deployment`, `insights_scan`, `policy_evaluation`). 환경 변수는 쉼표 구분 |
| `env_forward_allowlist` | `[]` | 호스트에서 러너 컨테이너로 전달할 환경 변수 목록. `DOCKER_HOST`는 항상 전달됨. 환경 변수는 공백 구분 |

**OIDC 설정:**

| 설정 | 기본값 | 설명 |
|---|---|---|
| `oidc_token_file` | `""` | OIDC 토큰이 포함된 파일 경로. Pulumi 토큰 만료 시마다 다시 읽음. OIDC 사용 시 `organization_name`, `runner_pool_id` 필수이며 `token` 불필요 |
| `organization_name` | `""` | Pulumi 조직 이름. OIDC 사용 시 필수 |
| `runner_pool_id` | `""` | 러너가 연결할 풀 ID. OIDC 사용 시 필수 |
| `token_expiration` | `""` | OIDC 교환을 통해 발급되는 토큰 수명 (Go Duration 문법, 예: `1h`) |

**폴링 및 재시도 설정:**

| 설정 | 기본값 | 설명 |
|---|---|---|
| `polling_interval` | `1m` | 새 작업 확인 폴링 간격. 서버가 Retry-After 힌트를 반환하면 해당 값이 우선 |
| `polling_interval_override` | `false` | `true` 시 서버의 Retry-After 헤더를 무시하고 항상 `polling_interval` 사용 |
| `job_status_loop_interval` | `30s` | 진행 중인 작업 상태 확인 간격 (취소 감지용) |
| `request_timeout` | `30s` | Pulumi Cloud API 요청 타임아웃 |
| `request_retry_count` | `2` | Rate Limit 또는 일시적 실패 시 최대 재시도 횟수 |
| `request_retry_wait` | `20s` | 재시도 간 초기 대기 시간 |
| `request_retry_max_wait` | `2m` | 재시도 간 대기 시간 상한 |
| `circuit_breaker_failures` | `2` | 연속 API 실패 횟수가 이 값에 도달하면 서킷 브레이커가 동작하여 폴링 일시 중지 |
| `circuit_breaker_timeout` | `10m` | 서킷 브레이커 동작 후 대기 시간 |

**Health 및 관측 설정:**

| 설정 | 기본값 | 설명 |
|---|---|---|
| `http_server_port` | `8080` | Health Check 엔드포인트 포트 |
| `health_threshold` | 자동 | 러너가 진행 없이 지날 수 있는 최대 시간. 이 시간 초과 시 health 엔드포인트가 unhealthy 보고. 기본값은 `polling_interval`과 `job_status_loop_interval` 중 긴 값의 2배 |
| `syslog` | `false` | `true` 시 stderr 대신 syslog에 로그 출력 |

### OIDC 인증 활용

정적 토큰 대신 OpenID 인증을 사용하여 Pulumi Pool 토큰을 동적으로 가져올 수 있다. 먼저 Pulumi 계정에 OpenID 제공자를 신뢰할 수 있는 OIDC Issuer로 등록해야 한다. Kubernetes 클러스터에서 실행하는 경우 EKS 또는 GKE에 대한 클러스터별 가이드를 참조한다.

등록 후 워크플로 러너에 다음 정보가 필요하다.

- `organization_name`: Pulumi 조직 이름
- `runner_pool_id`: 러너가 연결할 풀 ID
- `token_expiration` (선택): 러너가 요청하는 토큰 수명
- `oidc_token_file`: OIDC 토큰이 기록되는 파일 위치

워크플로 러너는 `oidc_token_file`에서 새 OIDC 토큰을 읽어 Pulumi 토큰이 만료될 때마다 자동으로 교환한다.

### Credentials 제공 방법

워크플로 러너에 클라우드 제공자 자격 증명을 제공하는 두 가지 방법이 있다.

1. **OIDC를 통한 자격 증명 생성**: OpenID Connect를 사용하여 자격 증명을 동적으로 생성
2. **환경 변수를 통한 직접 제공**: 호스트에서 환경 변수를 구성하거나 바이너리 실행 시 전달

```bash
VARIABLE=value customer-managed-workflow-agent run
```

`pulumi-workflow-agent.yaml`의 `env_forward_allowlist` 설정에 전달할 환경 변수를 지정해야 한다.

```yaml
token: pul-d2d2....
version: v0.0.5
env_forward_allowlist:
    - key_one
    - key_two
    - key_three
```

### Kubernetes-managed Workflow Runners

Kubernetes 환경에서는 구성 값을 환경 변수로 설정하거나 구성 파일을 Pod에 마운트하여 워크플로 러너를 실행할 수 있다. Kubernetes 전용 설정 옵션:

```yaml
# Kubernetes 이미지 Pull 정책
PULUMI_AGENT_IMAGE_PULL_POLICY: IfNotPresent
```

> Kubernetes 네이티브 설치 시 워크플로 러너 구성은 러너를 실행하는 Kubernetes Deployment에 설정된다。

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

## Private Repositories and Packages

Pulumi Deployments에서 프라이빗 Git 리포지토리에 접근해야 하는 경우가 있다.

### 프라이빗 의존성 패키지

Pulumi Deployment이 프라이빗 GitHub 리포지토리에 접근해야 하는 경우(예: 프라이빗 Go 모듈 사용), 필요한 리포지토리에 접근할 수 있는 SSH 키를 구성해야 한다. 올바른 구성 없이는 Deployment가 프라이빗 아티팩트에 접근할 수 없어 배포가 실패할 수 있다.

**구성 방법:**

1. Pre-run Commands에 다음 코드를 추가하고 Advanced Settings에서 "Skip automatic dependency installation step"을 활성화한다.

```bash
mkdir /root/.ssh && printf -- "$SSHKEY" > /root/.ssh/id_ed25519
chmod 600 /root/.ssh/id_ed25519
ssh-keyscan github.com >> ~/.ssh/known_hosts
cd .. && git config --global --add url."git@github.com:".insteadOf "https://github.com"
```

2. `$SSHKEY` 필드를 시크릿 환경 변수로 추가한다.

---

## GitHub App 통합

Pulumi Deployments는 GitHub와의 깊은 통합을 제공한다. GitHub App을 설치하면 Push to Deploy, Pull Request Preview, Review Stacks 등의 기능을 GitHub 리포지토리에서 직접 사용할 수 있다.

- **PR 자동 Preview**: PR이 열리거나 업데이트될 때 자동으로 `pulumi preview` 실행 후 결과를 PR 댓글에 게시
- **Merge 자동 배포**: PR이 병합되면 자동으로 `pulumi up` 실행
- **Review Stacks**: PR에서 임시 인프라 환경 자동 생성/삭제
- **Git Tag 배포**: Tag Push 시 자동 배포 트리거

> GitHub App은 Pulumi Cloud 조직 설정의 Integrations에서 구성할 수 있다.

---

## 기타 설정 옵션

### 의존성 설치 건너뛰기

기본적으로 배포 실행기가 언어별 기본 의존성 매니저(`npm`, `virtualenv` 등)로 의존성을 설치한다. `yarn`이나 다른 버전의 `node`를 사용하려면 기본 설치 단계를 건너뛰고(Under Advanced Settings) Pre-Run 명령으로 직접 설치할 수 있다.

### 중간 배포 건너뛰기

여러 배포가 Push될 때 기본적으로 순차적으로 실행된다. `Skip intermediate deployments` 설정을 활성화하면 동일 유형의 중간 배포를 모두 건너뛰고 가장 최근 배포만 실행한다.

### Execution Mode

Deployment 실행 모드를 통해 배포가 어떻게 실행되는지 제어할 수 있다. 기본적으로 모든 배포는 Pulumi 호스팅 러너에서 실행되지만, Customer-Managed Workflow Runners를 사용하면 자체 인프라에서 실행할 수 있다.

### Deployment 승인 (Approvals)

Deployment 실행 전에 승인(Approval)을 요구하도록 구성할 수 있다. 이를 통해 중요한 인프라 변경이 배포되기 전에 지정된 승인자의 검토를 거치도록 할 수 있다.

승인 설정은 Deployment Settings에서 구성하며, 조직의 컴플라이언스 요구 사항에 따라 프로덕션 스택에만 선택적으로 적용할 수 있다.

### Role Assignment

Deployment Settings에서 스택 토큰에 조직 역할을 할당할 수 있다. 역할이 선택되지 않으면 배포는 해당 스택에만 액세스할 수 있다. 다른 스택 참조, 환경 액세스, 조직 리소스 관리가 필요한 경우 적절한 역할을 선택해야 한다.

---

## Post-Deployment Automation

> https://www.pulumi.com/docs/deployments/deployments/using/post-automation/

배포가 완료된 후 추가 작업이나 배포를 트리거하는 두 가지 방법이 있다. 두 방법 모두 스택에 [Deployment Settings](#deployment-settings)가 구성되어 있어야 한다.

### Deployment Webhook Destinations

특정 이벤트(예: `update succeeded`) 발생 시 다른 스택의 Create Deployment API로 이벤트를 자동 전달한다.

**TypeScript 예제:**

```typescript
import * as pulumiservice from "@pulumi/pulumiservice";

const databaseWebhook = new pulumiservice.Webhook("databaseWebhook", {
    organizationName: "org",
    projectName: "network",
    stackName: "prod",
    format: pulumiservice.WebhookFormat.PulumiDeployments,
    payloadUrl: "database/prod",
    active: true,
    displayName: "deploy-database",
    filters: [pulumiservice.WebhookFilters.UpdateSucceeded],
});

const computeWebhook = new pulumiservice.Webhook("computeWebhook", {
    organizationName: "org",
    projectName: "database",
    stackName: "prod",
    format: pulumiservice.WebhookFormat.PulumiDeployments,
    payloadUrl: "compute/prod",
    active: true,
    displayName: "deploy-compute",
    filters: [pulumiservice.WebhookFilters.UpdateSucceeded],
});
```

**Python 예제:**

```python
import pulumi_pulumiservice as pulumiservice

database_webhook = pulumiservice.Webhook("databaseWebhook",
    organization_name="org",
    project_name="network",
    stack_name="prod",
    format=pulumiservice.WebhookFormat.PULUMI_DEPLOYMENTS,
    payload_url="database/prod",
    active=True,
    display_name="deploy-database",
    filters=[pulumiservice.WebhookFilters.UPDATE_SUCCEEDED])

compute_webhook = pulumiservice.Webhook("computeWebhook",
    organization_name="org",
    project_name="database",
    stack_name="prod",
    format=pulumiservice.WebhookFormat.PULUMI_DEPLOYMENTS,
    payload_url="compute/prod",
    active=True,
    display_name="deploy-compute",
    filters=[pulumiservice.WebhookFilters.UPDATE_SUCCEEDED])
```

**Go 예제:**

```go
_, err := pulumiservice.NewWebhook(ctx, "databaseWebhook", &pulumiservice.WebhookArgs{
    OrganizationName: pulumi.String("org"),
    ProjectName:      pulumi.String("network"),
    StackName:        pulumi.String("prod"),
    Format:           pulumiservice.WebhookFormatPulumiDeployments,
    PayloadUrl:       pulumi.String("database/prod"),
    Active:           pulumi.Bool(true),
    DisplayName:      pulumi.String("deploy-database"),
    Filters: pulumiservice.WebhookFiltersArray{
        pulumiservice.WebhookFiltersUpdateSucceeded,
    },
})
```

**C# 예제:**

```csharp
var databaseWebhook = new PulumiService.Webhook("databaseWebhook", new()
{
    OrganizationName = "org",
    ProjectName = "network",
    StackName = "prod",
    Format = PulumiService.WebhookFormat.PulumiDeployments,
    PayloadUrl = "database/prod",
    Active = true,
    DisplayName = "deploy-database",
    Filters = new[] { PulumiService.WebhookFilters.UpdateSucceeded },
});
```

**YAML 예제:**

```yaml
resources:
  databaseWebhook:
    type: pulumiservice:Webhook
    properties:
      organizationName: org
      projectName: network
      stackName: prod
      format: pulumi_deployments
      payloadUrl: database/prod
      active: true
      displayName: deploy-database
      filters:
        - update_succeeded
```

### Pulumi Auto Deploy Package (Preview)

`@pulumi/auto-deploy` 패키지를 사용하여 스택 간의 의존성을 선언적으로 표현하고, 필요한 Deployment Webhook을 자동으로 관리할 수 있다.

다음 예제는 아래와 같은 의존성 그래프를 구성한다.

```
a
├── b
│   ├── d
│   ├── e
│   └── f
└── c
```

그래프의 노드가 업데이트되면 모든 하위 노드가 Webhook을 통해 Pulumi Deployments에 의해 자동으로 업데이트된다.

**TypeScript 예제:**

```typescript
import * as autodeploy from "@pulumi/auto-deploy";
import * as pulumi from "@pulumi/pulumi";

const organization = pulumi.getOrganization();
const project = "dependency-example";

export const f = new autodeploy.AutoDeployer("auto-deployer-f", {
    organization,
    project,
    stack: "f",
    downstreamRefs: [],
});

export const e = new autodeploy.AutoDeployer("auto-deployer-e", {
    organization,
    project,
    stack: "e",
    downstreamRefs: [],
});

export const d = new autodeploy.AutoDeployer("auto-deployer-d", {
    organization,
    project,
    stack: "d",
    downstreamRefs: [],
});

export const c = new autodeploy.AutoDeployer("auto-deployer-c", {
    organization,
    project,
    stack: "c",
    downstreamRefs: [],
});

export const b = new autodeploy.AutoDeployer("auto-deployer-b", {
    organization,
    project,
    stack: "b",
    downstreamRefs: [d.ref, e.ref, f.ref],
});

export const a = new autodeploy.AutoDeployer("auto-deployer-a", {
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

f = auto_deploy.AutoDeployer("f",
    organization=organization,
    project=project,
    stack="f",
    downstream_refs=[])

e = auto_deploy.AutoDeployer("e",
    organization=organization,
    project=project,
    stack="e",
    downstream_refs=[])

d = auto_deploy.AutoDeployer("d",
    organization=organization,
    project=project,
    stack="d",
    downstream_refs=[])

c = auto_deploy.AutoDeployer("c",
    organization=organization,
    project=project,
    stack="c",
    downstream_refs=[])

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

**C# 예제:**

```csharp
var f = new AutoDeploy.AutoDeployer("f", new()
{
    Organization = organization,
    Project = projectVar,
    Stack = "f",
    DownstreamRefs = new[] {},
});

var b = new AutoDeploy.AutoDeployer("b", new()
{
    Organization = organization,
    Project = projectVar,
    Stack = "b",
    DownstreamRefs = new[] { d.Ref, e.Ref, f.Ref },
});

var a = new AutoDeploy.AutoDeployer("a", new()
{
    Organization = organization,
    Project = projectVar,
    Stack = "a",
    DownstreamRefs = new[] { b.Ref, c.Ref },
});
```
