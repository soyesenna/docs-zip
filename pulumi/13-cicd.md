# Pulumi CI/CD 통합 가이드

> **출처**
> - https://www.pulumi.com/docs/iac/operations/continuous-delivery/
> - https://www.pulumi.com/docs/iac/operations/continuous-delivery/github-actions/
> - https://www.pulumi.com/docs/iac/operations/continuous-delivery/gitlab-ci/
> - https://www.pulumi.com/docs/iac/operations/continuous-delivery/circleci/
> - https://www.pulumi.com/docs/iac/operations/continuous-delivery/jenkins/
> - https://www.pulumi.com/docs/iac/operations/continuous-delivery/azure-devops/
> - https://www.pulumi.com/docs/iac/operations/continuous-delivery/bitbucket/
> - https://www.pulumi.com/docs/iac/operations/continuous-delivery/buildkite/
> - https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/
> - https://www.pulumi.com/docs/iac/operations/continuous-delivery/troubleshooting/

Pulumi는 클라우드 리소스를 소스 코드로 모델링하므로, 인프라 변경 사항을 애플리케이션 코드와 동일한 파이프라인에서 검토·테스트·배포할 수 있다. Pull Request, 코드 리뷰, 자동화 테스트 기반의 trunk-based development 워크플로우를 CI/CD 시스템과 통합하는 방법을 정리한다.

---

## Trunk-based Development 워크플로우

대부분의 Pulumi CI/CD 파이프라인은 다음 세 단계의 trunk-based development 모델을 따른다.

| 단계 | 트리거 | Pulumi 명령 | 목적 |
|------|--------|-------------|------|
| PR 오픈 | Pull Request 생성/업데이트 | `pulumi preview` | 제안된 변경 사항 검토 |
| main 브랜치 병합 | `main` 브랜치 push | `pulumi up` | staging/dev 환경에 지속적 배포 |
| 프로덕션 승격 | `release-*` 태그 push | `pulumi up` | 프로덕션 환경에 의도적 배포 |

> **Warning:** CI/CD 파이프라인은 모든 Pull Request에서 `pulumi preview`를 반드시 실행해야 한다. 코드 diff만으로는 실제 인프라 변경을 파악할 수 없기 때문이다. 동일한 코드 수정도 현재 리소스 상태에 따라 완전히 다른 인프라 변경을 생성할 수 있다.

PR은 `main` 브랜치와 최신 상태를 유지해야 하며, 브랜치가 업데이트될 때마다 `pulumi preview`를 다시 실행해야 한다. 중간에 `main` 브랜치에 머지된 변경이 기존 preview 결과를 무효화할 수 있다.

---

## 인증 및 구성

### Pulumi Cloud 인증

파이프라인은 단일 [Pulumi access token](https://www.pulumi.com/docs/administration/access-identity/access-tokens/)으로 Pulumi Cloud에 인증한다.

| 인증 방식 | 설명 | 권장 여부 |
|-----------|------|-----------|
| **저장된 access token** | `PULUMI_ACCESS_TOKEN` 환경 변수에 장기 토큰 저장 | 기본 |
| **OIDC token exchange** | CI/CD 시스템의 단기 OIDC 토큰을 Pulumi access token으로 교환 | 권장 |

개인 토큰보다 조직 또는 팀 토큰 사용을 권장한다.

### OIDC(OpenID Connect) 인증

OIDC를 사용하면 장기 비밀을 저장할 필요가 없다. CI/CD 시스템이 발급하는 단기 OIDC 토큰을 Pulumi Cloud의 단기 access token으로 교환한다.

**OIDC 흐름:**

1. 외부 워크로드가 호스트 서비스에서 OIDC id_token 획득
1. id_token을 Pulumi Cloud에 전달하여 단기 Pulumi access token으로 교환
1. 교환된 토큰으로 Pulumi 작업 실행

**OIDC 발급자 관리 방법:**

| 방법 | 설명 |
|------|------|
| Pulumi Cloud UI | Settings > Access Management > OIDC Issuers |
| REST API | OIDC Issuers REST API 사용 |
| Pulumi Service Provider | `OidcIssuer` 리소스로 코드로 관리 |

**에디션별 사용 가능 토큰 타입:**

| 에디션 | 사용 가능 토큰 타입 |
|--------|---------------------|
| Individual | `personal` |
| Team | `personal`, `organization` |
| Enterprise | `personal`, `organization`, `team` |
| Business Critical | `personal`, `organization`, `team`, `deployment-runner` |

**CLI로 OIDC 토큰 교환:**

```bash
pulumi login --oidc-token <token> --oidc-org <org-name>
```

> **Warning:** `OIDC token exchange failed: Post "/api/oauth/token": unsupported protocol scheme ""` 오류가 발생하면 backend URL을 명시하라:
> `pulumi login https://api.pulumi.com --oidc-token <token> --oidc-org <org-name>`

### Pulumi ESC를 통한 시크릿 관리

[Pulumi ESC](https://www.pulumi.com/docs/esc/)(Environments, Secrets, and Configuration)는 클라우드 자격 증명, 시크릿, 구성을 파이프라인에 제공한다. ESC는 CI/CD 파이프라인과 개발자 머신 모두에 동일한 방식으로 값을 전달하므로, 하나의 환경 정의가 양쪽에서 모두 작동한다.

---

## CI/CD별 설정 가이드

### CI/CD 시스템 비교

| CI/CD 시스템 | 설정 파일 | 통합 방식 | OIDC 지원 | 공식 확장 |
|-------------|----------|----------|----------|----------|
| GitHub Actions | `.github/workflows/*.yml` | `pulumi/actions` 액션 | `pulumi/auth-actions` | Pulumi-maintained |
| GitLab CI/CD | `.gitlab-ci.yml` | CLI 직접 호출 + 공식 컨테이너 이미지 | `id_tokens` 키워드 | 공식 Docker 이미지 |
| CircleCI | `.circleci/config.yml` | Pulumi Orb | ESC 통해 간접 | Pulumi-maintained Orb |
| Jenkins | `Jenkinsfile` | `pulumi/pulumi` 컨테이너 이미지 | 커뮤니티 OIDC Provider 플러그인 필요 | 공식 Docker 이미지 |
| Azure DevOps | `azure-pipelines.yml` | Pulumi Task Extension (`Pulumi@1`) | ESC 통해 간접 | Pulumi-maintained |
| Bitbucket Pipelines | `bitbucket-pipelines.yml` | CLI 직접 호출 | `oidc: true` 스텝 설정 | 공식 Docker 이미지 |
| Buildkite | `.buildkite/pipeline.yml` | `pulumi` 플러그인 | 플러그인 `use-oidc: true` | Pulumi-maintained |

---

## GitHub Actions

GitHub Actions는 GitHub에 내장된 CI/CD 서비스로, `.github/workflows/` 디렉터리의 YAML 파일로 워크플로우를 정의한다.

### Pulumi GitHub Actions

| Action | 목적 |
|--------|------|
| `pulumi/actions` | Pulumi CLI 설치 및 명령 실행 |
| `pulumi/setup-pulumi` | CLI만 설치 (직접 `pulumi` 명령 호출 시) |
| `pulumi/auth-actions` | GitHub OIDC 토큰을 단기 Pulumi access token으로 교환 |
| `pulumi/esc-action` | Pulumi ESC 환경 변수를 워크플로우에 주입 |
| `pulumi/esc-export-secrets-action` | GitHub Actions 시크릿을 ESC 환경으로 내보내기 |

### OIDC 인증 (권장)

OIDC를 사용하려면 `id-token: write` 권한이 필요하다. PR 코멘트 작성 시 `pull-requests: write`도 추가한다.

```yaml
permissions:
  id-token: write
  contents: read
  pull-requests: write
```

**OIDC + ESC 인증 예시:**

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version-file: infra/package.json
  - name: Authenticate with Pulumi Cloud
    uses: pulumi/auth-actions@v1
    with:
      organization: acme
      requested-token-type: urn:pulumi:token-type:access_token:organization
  - name: Load the ESC environment
    uses: pulumi/esc-action@v1
    with:
      environment: acme/website/ci
  - run: npm install
    working-directory: infra
  - uses: pulumi/actions@v7
    with:
      command: preview
      stack-name: acme/website/staging
      work-dir: infra
```

### Trunk-based 워크플로우 (TypeScript)

**PR Preview (`.github/workflows/pr.yml`):**

```yaml
name: Pulumi preview
on:
  pull_request:
jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: infra/package.json
      - run: npm install
        working-directory: infra
      - uses: pulumi/actions@v7
        with:
          command: preview
          stack-name: acme/website/staging
          work-dir: infra
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
```

**배포 (`.github/workflows/main.yml`):**

```yaml
name: Pulumi deploy
on:
  push:
    branches:
      - main
    tags:
      - 'release-*'
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: infra/package.json
      - run: npm install
        working-directory: infra

      - name: Deploy to staging
        if: github.ref == 'refs/heads/main'
        uses: pulumi/actions@v7
        with:
          command: up
          stack-name: acme/website/staging
          work-dir: infra
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}

      - name: Deploy to production
        if: startsWith(github.ref, 'refs/tags/release-')
        uses: pulumi/actions@v7
        with:
          command: up
          stack-name: acme/website/production
          work-dir: infra
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
```

### Trunk-based 워크플로우 (Python)

**PR Preview:**

```yaml
name: Pulumi preview
on:
  pull_request:
jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install -r requirements.txt
        working-directory: infra
      - uses: pulumi/actions@v7
        with:
          command: preview
          stack-name: acme/website/staging
          work-dir: infra
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
```

**배포:**

```yaml
name: Pulumi deploy
on:
  push:
    branches:
      - main
    tags:
      - 'release-*'
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install -r requirements.txt
        working-directory: infra

      - name: Deploy to staging
        if: github.ref == 'refs/heads/main'
        uses: pulumi/actions@v7
        with:
          command: up
          stack-name: acme/website/staging
          work-dir: infra
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}

      - name: Deploy to production
        if: startsWith(github.ref, 'refs/tags/release-')
        uses: pulumi/actions@v7
        with:
          command: up
          stack-name: acme/website/production
          work-dir: infra
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
```

### PR에 결과 보고

**Pulumi GitHub App (권장):** Pulumi Cloud가 리소스 변경 요약을 PR에 직접 게시한다.

**Action 코멘트:** `comment-on-pr: true` 설정 시 CLI 출력을 PR에 게시한다.

```yaml
- uses: pulumi/actions@v7
  with:
    command: preview
    stack-name: acme/website/staging
    work-dir: infra
    comment-on-pr: true
    github-token: ${{ secrets.GITHUB_TOKEN }}
  env:
    PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
```

push 트리거 배포의 경우 `comment-on-summary: true`로 workflow run summary에 결과를 게시할 수 있다.

### Stack Outputs

`pulumi/actions` 단계에 `id`를 부여하면 stack output을 후속 단계에서 사용할 수 있다.

```yaml
- name: Deploy
  id: pulumi
  uses: pulumi/actions@v7
  with:
    command: up
    stack-name: acme/website/staging
    work-dir: infra
  env:
    PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
- name: Use a stack output
  run: echo "Deployed to ${{ steps.pulumi.outputs.url }}"
```

> **Warning:** Stack output에 민감한 값이 포함될 수 있다. `suppress-outputs: true`로 출력 값을 워크플로우 로그에서 숨기고, 시크릿은 stack output 대신 encrypted secrets에 저장하라.

### 캐시로 속도 향상

```yaml
- name: Cache Pulumi plugins and policy packs
  uses: actions/cache@v4
  with:
    path: |
      ~/.pulumi/plugins
      ~/.pulumi/policies
    key: ${{ runner.os }}-pulumi-${{ hashFiles('infra/package.json') }}
    restore-keys: |
      ${{ runner.os }}-pulumi-
```

### 동시 실행 제어

**PR Preview:** 최신 커밋 결과만 유지

```yaml
concurrency:
  group: pulumi-pr-${{ github.event.pull_request.number }}
  cancel-in-progress: true
```

**배포:** 큐잉 (중간 취소는 인프라를 불완전한 상태로 남길 수 있음)

```yaml
concurrency:
  group: pulumi-deploy
```

---

## GitLab CI/CD

GitLab CI/CD는 `.gitlab-ci.yml` 파일로 파이프라인을 정의한다. 공식 `pulumi/pulumi` 컨테이너 이미지를 사용하여 CLI를 직접 호출한다.

### OIDC 인증

GitLab의 `id_tokens` 키워드로 OIDC 토큰을 요청하고 `pulumi login --oidc-token`으로 교환한다.

```yaml
variables:
  PULUMI_ORG: acme

.pulumi:
  id_tokens:
    PULUMI_OIDC_TOKEN:
      aud: urn:pulumi:org:$PULUMI_ORG
  before_script:
    - pulumi login --oidc-token "$PULUMI_OIDC_TOKEN" --oidc-org "$PULUMI_ORG"
    - cd infra
    - npm ci
```

### Trunk-based 워크플로우 (TypeScript)

```yaml
# .gitlab-ci.yml
stages:
  - preview
  - deploy

default:
  image:
    name: pulumi/pulumi-nodejs:latest
    entrypoint: [""]

variables:
  PULUMI_STACK_STAGING: acme/website/staging
  PULUMI_STACK_PRODUCTION: acme/website/production

.pulumi:
  before_script:
    - cd infra
    - npm ci

preview:
  extends: .pulumi
  stage: preview
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  script:
    - pulumi preview --stack "$PULUMI_STACK_STAGING"

deploy-staging:
  extends: .pulumi
  stage: deploy
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  environment:
    name: staging
  script:
    - pulumi up --yes --stack "$PULUMI_STACK_STAGING"

deploy-production:
  extends: .pulumi
  stage: deploy
  rules:
    - if: $CI_COMMIT_TAG =~ /^release-/
  environment:
    name: production
  script:
    - pulumi up --yes --stack "$PULUMI_STACK_PRODUCTION"
```

### Trunk-based 워크플로우 (Python)

```yaml
# .gitlab-ci.yml
stages:
  - preview
  - deploy

default:
  image:
    name: pulumi/pulumi-python:latest
    entrypoint: [""]

variables:
  PULUMI_STACK_STAGING: acme/website/staging
  PULUMI_STACK_PRODUCTION: acme/website/production

.pulumi:
  before_script:
    - cd infra
    - pip install -r requirements.txt

preview:
  extends: .pulumi
  stage: preview
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  script:
    - pulumi preview --stack "$PULUMI_STACK_STAGING"

deploy-staging:
  extends: .pulumi
  stage: deploy
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  environment:
    name: staging
  script:
    - pulumi up --yes --stack "$PULUMI_STACK_STAGING"

deploy-production:
  extends: .pulumi
  stage: deploy
  rules:
    - if: $CI_COMMIT_TAG =~ /^release-/
  environment:
    name: production
  script:
    - pulumi up --yes --stack "$PULUMI_STACK_PRODUCTION"
```

### 캐시 설정

```yaml
variables:
  PULUMI_HOME: $CI_PROJECT_DIR/.pulumi

cache:
  key:
    files:
      - infra/package-lock.json
  paths:
    - .pulumi/plugins
    - .pulumi/policies
```

### 배포 직렬화

동시 `pulumi up` 실행으로 인한 업데이트 충돌을 방지하려면 `resource_group`을 사용한다.

```yaml
deploy-staging:
  resource_group: staging

deploy-production:
  resource_group: production
```

---

## CircleCI

CircleCI는 `.circleci/config.yml`에 파이프라인을 정의하며, Pulumi Orb로 CLI를 설치하고 실행한다.

### Orb 추가

```yaml
version: 2.1

orbs:
  pulumi: pulumi/pulumi@2.2.0
```

### 인증

`PULUMI_ACCESS_TOKEN`을 CircleCI [context](https://circleci.com/docs/contexts/) 또는 프로젝트 환경 변수에 저장한다.

### Trunk-based 워크플로우

```yaml
version: 2.1

orbs:
  pulumi: pulumi/pulumi@2.2.0

jobs:
  preview:
    docker:
      - image: cimg/node:lts
    steps:
      - checkout
      - pulumi/login
      - run:
          name: Install dependencies
          command: pulumi install --cwd infra/
      - pulumi/preview:
          stack: acme/website/staging
          working_directory: infra/

  deploy-staging:
    docker:
      - image: cimg/node:lts
    steps:
      - checkout
      - pulumi/login
      - run:
          name: Install dependencies
          command: pulumi install --cwd infra/
      - pulumi/update:
          stack: acme/website/staging
          working_directory: infra/

  deploy-production:
    docker:
      - image: cimg/node:lts
    steps:
      - checkout
      - pulumi/login
      - run:
          name: Install dependencies
          command: pulumi install --cwd infra/
      - pulumi/update:
          stack: acme/website/production
          working_directory: infra/

workflows:
  pulumi:
    jobs:
      - preview:
          context: pulumi
          filters:
            branches:
              ignore: main
      - deploy-staging:
          context: pulumi
          filters:
            branches:
              only: main
      - deploy-production:
          context: pulumi
          filters:
            tags:
              only: /^release-.*/
            branches:
              ignore: /.*/
```

### Orb 명령어 참조

| 명령어 | 설명 | 주요 파라미터 |
|--------|------|-------------|
| `pulumi/login` | Pulumi CLI 설치 및 로그인 | `version`, `cloud-url`, `access-token` |
| `pulumi/preview` | `pulumi preview` 실행 | `stack` |
| `pulumi/update` | `pulumi up` 실행 | `stack`, `skip-preview` |
| `pulumi/refresh` | `pulumi refresh` 실행 | `stack`, `expect_no_changes`, `skip-preview` |
| `pulumi/destroy` | `pulumi destroy` 실행 | `stack`, `skip-preview` |
| `pulumi/stack_init` | 신규 스택 생성 | `stack`, `secrets_provider`, `copy` |
| `pulumi/stack_rm` | 스택 제거 | `stack`, `force` |
| `pulumi/stack_output` | Stack output을 환경 변수로 읽기 | `stack`, `property_name`, `env_var`, `show_secrets` |

---

## Jenkins

Jenkins는 `Jenkinsfile`에 파이프라인을 정의하며, `pulumi/pulumi` 컨테이너 이미지 내에서 Pulumi를 실행한다. 별도 Jenkins 플러그인은 필요하지 않다.

### 인증

**저장된 access token:**

```groovy
environment {
    PULUMI_ACCESS_TOKEN = credentials("pulumi-access-token")
}
```

**OIDC token exchange:** Jenkins 자체적으로 OIDC 토큰을 발급하지 않으므로 커뮤니티 [OpenID Connect Provider 플러그인](https://plugins.jenkins.io/oidc-provider/)이 필요하다.

```bash
pulumi login --oidc-token "$OIDC_TOKEN" --oidc-org "your-org"
```

### 기본 파이프라인

```groovy
pipeline {
    agent {
        docker { image "pulumi/pulumi" }
    }

    environment {
        PULUMI_ACCESS_TOKEN = credentials("pulumi-access-token")
    }

    stages {
        stage("Deploy") {
            steps {
                dir("infra") {
                    sh "pulumi install"
                    sh "pulumi stack select acme/website/staging"
                    sh "pulumi up --yes"
                }
            }
        }
    }
}
```

### Trunk-based 워크플로우

```groovy
// PR: preview
stage("Preview") {
    when { changeRequest() }
    steps {
        dir("infra") {
            sh "pulumi install"
            sh "pulumi stack select acme/website/staging"
            sh "pulumi preview"
        }
    }
}

// main: staging 배포
stage("Deploy to staging") {
    when { branch "main" }
    steps {
        dir("infra") {
            sh "pulumi install"
            sh "pulumi stack select acme/website/staging"
            sh "pulumi up --yes"
        }
    }
}

// 태그: 프로덕션 배포
stage("Deploy to production") {
    when { buildingTag() }
    steps {
        dir("infra") {
            sh "pulumi install"
            sh "pulumi stack select acme/website/production"
            sh "pulumi up --yes"
        }
    }
}
```

> **Note:** Jenkinsfile에서 인자는 항상 큰따옴표로 감싸야 한다. 작은따옴표는 Groovy 문자열 보간을 억제하여 명령이 조용히 실패할 수 있다.

### 캐시: 커스텀 빌더 이미지

```dockerfile
FROM pulumi/pulumi
RUN pulumi plugin install resource aws <version> \
    && pulumi plugin install resource random <version>
```

```groovy
agent {
    docker { image "your-registry/pulumi-builder" }
}
```

---

## Azure DevOps

Azure DevOps는 [Pulumi Task Extension](https://marketplace.visualstudio.com/items?itemName=pulumi.build-and-release-task)(`Pulumi@1` 태스크)을 통해 Pulumi를 실행한다.

### 인증

`pulumi.access.token` 파이프라인 변수를 자동으로 `PULUMI_ACCESS_TOKEN` 환경 변수에 매핑한다. 시크릿 파이프라인 변수 또는 변수 그룹에 저장하라.

### Trunk-based 워크플로우

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
  tags:
    include:
      - release-*

pr:
  branches:
    include:
      - main

pool:
  vmImage: ubuntu-latest

steps:
  - task: Pulumi@1
    displayName: Preview changes
    condition: eq(variables['Build.Reason'], 'PullRequest')
    inputs:
      command: preview
      stack: acme/website/staging
      cwd: infra/
      createPrComment: true

  - task: Pulumi@1
    displayName: Deploy to staging
    condition: and(eq(variables['Build.SourceBranch'], 'refs/heads/main'), ne(variables['Build.Reason'], 'PullRequest'))
    inputs:
      command: up
      args: --yes
      stack: acme/website/staging
      cwd: infra/

  - task: Pulumi@1
    displayName: Deploy to production
    condition: startsWith(variables['Build.SourceBranch'], 'refs/tags/release-')
    inputs:
      command: up
      args: --yes
      stack: acme/website/production
      cwd: infra/
```

### PR 코멘트

`createPrComment: true` 설정 시 빌드 출력을 PR에 게시한다. 이를 위해서는 빌드 서비스 사용자에게 **Contribute to pull requests** 권한이 필요하다.

> **Note:** PR 코멘트는 Azure DevOps 호스팅 저장소에서만 지원된다.

### Pulumi@1 태스크 파라미터

| 파라미터 | 필수 | 설명 |
|----------|------|------|
| `stack` | Yes | 스택 이름 (`ORG/STACK` 또는 `ORG/PROJECT/STACK`) |
| `command` | No | 실행할 Pulumi CLI 명령 (`preview`, `up`, `destroy` 등) |
| `args` | No | 명령에 전달할 플래그 (예: `--yes`) |
| `cwd` | No | 작업 디렉터리 |
| `azureSubscription` | No | Azure 서비스 연결 참조 |
| `versionSpec` | No | Pulumi CLI 버전 (기본값: 최신) |
| `createStack` | No | 스택이 없으면 생성 (기본값: `false`) |
| `createPrComment` | No | PR에 파이프라인 출력 게시 (기본값: `false`) |
| `useThreadedPrComments` | No | 기존 코멘트 스레드에 추가 (기본값: `true`) |

---

## Bitbucket Pipelines

Bitbucket Pipelines는 `bitbucket-pipelines.yml`에 파이프라인을 정의하며, CLI를 직접 호출한다.

### 인증

`PULUMI_ACCESS_TOKEN`을 **Repository settings > Repository variables**에 **Secured**로 저장한다.

### OIDC 인증

스텝에 `oidc: true`를 설정하면 `BITBUCKET_STEP_OIDC_TOKEN` 환경 변수로 OIDC 토큰에 접근할 수 있다.

### Trunk-based 워크플로우

```yaml
# bitbucket-pipelines.yml
image: pulumi/pulumi:latest

pipelines:
  pull-requests:
    '**':
      - step:
          name: Preview infrastructure changes
          script:
            - cd infra
            - pulumi install
            - pulumi stack select acme/website/staging
            - pulumi preview

  branches:
    main:
      - step:
          name: Deploy to staging
          script:
            - cd infra
            - pulumi install
            - pulumi stack select acme/website/staging
            - pulumi up --yes

  tags:
    'release-*':
      - step:
          name: Deploy to production
          script:
            - cd infra
            - pulumi install
            - pulumi stack select acme/website/production
            - pulumi up --yes
```

> **Note:** 항상 민감한 값을 **Secured**로 표시하라. Secured가 아닌 변수는 일반 텍스트로 저장되고 출력된다.

---

## Buildkite

Buildkite는 `.buildkite/pipeline.yml`에 파이프라인을 정의하며, 공식 `pulumi` 플러그인으로 CLI를 설치한다.

### 플러그인 설정

```yaml
steps:
  - label: ":pulumi: Preview"
    commands:
      - cd infra && npm install
      - pulumi preview --stack acme/website/dev
    plugins:
      - pulumi#v1.0.0:
          version: 3.143.0
```

### 인증

**시크릿 + 플러그인:**

```yaml
plugins:
  - secrets#v1.0.0:
      variables:
        PULUMI_ACCESS_TOKEN: PULUMI_ACCESS_TOKEN
  - pulumi#v1.0.0: ~
```

**OIDC:**

```yaml
plugins:
  - pulumi#v1.0.0:
      use-oidc: true
      audience: "urn:pulumi:org:ACME_ORG"
```

### Trunk-based 워크플로우

```yaml
# .buildkite/pipeline.yml
steps:
  - label: ":pulumi: Preview"
    if: build.pull_request.id != null
    commands:
      - cd infra && npm install
      - pulumi preview --stack acme/website/staging 2>&1 | tee preview.txt
      - printf '```\n%s\n```\n' "$(cat preview.txt)" | buildkite-agent annotate --style info
    plugins:
      - secrets#v1.0.0:
          variables:
            PULUMI_ACCESS_TOKEN: PULUMI_ACCESS_TOKEN
      - pulumi#v1.0.0: ~

  - label: ":pulumi: Deploy to staging"
    if: build.branch == "main" && build.pull_request.id == null
    commands:
      - cd infra && npm install
      - pulumi up --yes --stack acme/website/staging
    plugins:
      - secrets#v1.0.0:
          variables:
            PULUMI_ACCESS_TOKEN: PULUMI_ACCESS_TOKEN
      - pulumi#v1.0.0: ~

  - label: ":pulumi: Deploy to production"
    if: build.tag != null
    commands:
      - cd infra && npm install
      - pulumi up --yes --stack acme/website/production
    plugins:
      - secrets#v1.0.0:
          variables:
            PULUMI_ACCESS_TOKEN: PULUMI_ACCESS_TOKEN
      - pulumi#v1.0.0: ~
```

### Docker 컨테이너 대안

```yaml
plugins:
  - docker#v5.9.0:
      image: pulumi/pulumi-nodejs
      mount-buildkite-agent: true
      environment:
        - PULUMI_ACCESS_TOKEN
```

---

## CI/CD 메타데이터 자동 감지

Pulumi CLI는 실행 환경을 자동으로 감지하여 빌드 메타데이터를 캡처한다. 감지되는 메타데이터는 다음과 같다.

- CI 시스템 이름
- Build ID, Build number, Build type, Build URL
- Commit SHA, Branch name, Commit message
- PR number

**자동 감지되는 CI/CD 시스템:**

AppVeyor, AWS CodeBuild, Atlassian Bamboo, Atlassian Bitbucket Pipelines, Azure Pipelines, Buildkite, CircleCI, Codefresh, Codeship, Drone, GitHub Actions, GitLab CI/CD, GoCD, Hudson, Jenkins, Magnum CI, Semaphore, Spacelift, TaskCluster, TeamCity, Travis CI

### 커스텀 CI/CD 시스템 메타데이터

목록에 없는 시스템은 다음 환경 변수를 설정하여 메타데이터를 전달할 수 있다.

| 환경 변수 | 설명 |
|-----------|------|
| `PULUMI_CI_SYSTEM` | CI 시스템 이름 |
| `PULUMI_CI_BRANCH_NAME` | 브랜치 이름 (Git detached HEAD 시에만 사용) |
| `PULUMI_CI_BUILD_ID` | 빌드 ID |
| `PULUMI_CI_BUILD_NUMBER` | 빌드 번호 |
| `PULUMI_CI_BUILD_TYPE` | 빌드 타입 |
| `PULUMI_CI_BUILD_URL` | 빌드 URL |
| `PULUMI_CI_PULL_REQUEST_SHA` | PR 커밋 SHA |
| `PULUMI_COMMIT_MESSAGE` | 커밋 메시지 (detached HEAD 시에만 사용) |
| `PULUMI_PR_NUMBER` | PR 번호 |

> **Note:** `PULUMI_CI_BRANCH_NAME`과 `PULUMI_COMMIT_MESSAGE`는 fallback으로 동작한다. 정상(non-detached) HEAD의 Git 저장소에서는 Git에서 읽은 값이 우선한다.

---

## 트러블슈팅

모든 CI/CD 시스템에서 Pulumi 실행 실패는 공통된 몇 가지 범주로 나뉜다.

### 전체 요구 사항

`pulumi preview` 또는 `pulumi up`을 실행하려면 빌드 에이전트에 다음이 필요하다.

| 요구 사항 | 설명 |
|-----------|------|
| Pulumi access token | `PULUMI_ACCESS_TOKEN` 환경 변수로 인증 |
| 스택 | 사전에 생성된 스택 (`pulumi stack init`) |
| Pulumi CLI | 시스템 PATH에 설치 |
| 언어 런타임 | 프로그램 언어의 빌드 도구 |
| 종속성 | 언어 종속성 및 리소스 프로바이더 플러그인 |
| 클라우드 자격 증명 | 리소스 관리 권한이 있는 프로바이더 자격 증명 |

### 주요 실패 원인별 해결 방법

| 실패 원인 | 확인 사항 |
|-----------|----------|
| **인증 실패** | `PULUMI_ACCESS_TOKEN`이 정확한 이름의 환경 변수로 노출되어 있는지, 올바른 job/step에 범위 지정되어 있는지 확인 |
| **스택 없음** | 스택이 이미 존재하는지, 정규화된 이름(`ORG/PROJECT/STACK`) 사용 여부 확인 |
| **CLI 없음** | CLI가 동일 step에서 설치 및 실행되는지 확인. 컨테이너 기반 파이프라인에서는 `pulumi/pulumi` 이미지 사용 |
| **빌드 도구 없음** | 언어 런타임과 패키지 매니저가 설치되어 있는지 확인 |
| **종속성 누락** | `pulumi install`로 언어 종속성과 플러그인을 함께 설치. 캐시 시 플러그인도 함께 캐시 |
| **클라우드 자격 증명** | 환경 변수 이름에 오타가 없는지, 올바른 계정/권한인지 확인 |

---

## Pulumi 네이티브 지속적 배포

CI/CD 시스템 외에도 Pulumi 자체의 지속적 배포 메커니즘이 있다.

| 기능 | 설명 |
|------|------|
| [Pulumi Deployments](https://www.pulumi.com/docs/deployments/) | Pulumi 호스팅 인프라에서 Pulumi 작업 실행. git push, CLI, API로 트리거 |
| [Review Stacks](https://www.pulumi.com/docs/deployments/deployments/review-stacks/) | 각 PR에 대해 임시 환경을 자동 생성하고 PR 종료 시 삭제 |
| [Pulumi Kubernetes Operator](https://www.pulumi.com/docs/integrations/clouds/kubernetes/pulumi-kubernetes-operator/) | Kubernetes 클러스터 내에서 Custom Resource로 스택 관리 |

---

## 지원되는 모든 CI/CD 시스템

Pulumi는 다음 CI/CD 시스템에 대한 가이드를 제공한다.

| CI/CD 시스템 | 가이드 경로 |
|-------------|------------|
| ArgoCD | `/docs/iac/operations/continuous-delivery/argocd/` |
| AWS Code Services | `/docs/iac/operations/continuous-delivery/aws-code-services/` |
| Azure DevOps | `/docs/iac/operations/continuous-delivery/azure-devops/` |
| Bitbucket Pipelines | `/docs/iac/operations/continuous-delivery/bitbucket/` |
| Buildkite | `/docs/iac/operations/continuous-delivery/buildkite/` |
| CircleCI | `/docs/iac/operations/continuous-delivery/circleci/` |
| Codefresh | `/docs/iac/operations/continuous-delivery/codefresh/` |
| GitHub Actions | `/docs/iac/operations/continuous-delivery/github-actions/` |
| GitLab CI/CD | `/docs/iac/operations/continuous-delivery/gitlab-ci/` |
| Google Cloud Build | `/docs/iac/operations/continuous-delivery/google-cloud-build/` |
| Harness | `/docs/iac/operations/continuous-delivery/harness/` |
| Jenkins | `/docs/iac/operations/continuous-delivery/jenkins/` |
| Octopus Deploy | `/docs/iac/operations/continuous-delivery/octopus-deploy/` |
| TeamCity | `/docs/iac/operations/continuous-delivery/teamcity/` |
| Travis CI | `/docs/iac/operations/continuous-delivery/travis/` |
