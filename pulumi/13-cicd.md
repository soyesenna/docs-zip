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

GitHub Actions는 GitHub에 내장된 CI/CD 서비스로, `.github/workflows/` 디렉터리의 YAML 파일로 워크플로우를 정의한다. `pulumi/actions` 액션으로 Pulumi CLI를 설치하고 명령을 실행하며, 어떤 언어로 작성된 Pulumi 프로그램이든 모든 클라우드 프로바이더에서 작동한다.

### Prerequisites

시작하기 전에 다음이 필요하다.

- Pulumi Cloud 계정 및 조직
- GitHub 저장소
- 해당 저장소에 커밋된 Pulumi 프로그램 (없으면 Get started 가이드 참조)

### Pulumi GitHub Actions

| Action | 목적 |
|--------|------|
| `pulumi/actions` | Pulumi CLI 설치 및 명령 실행 |
| `pulumi/setup-pulumi` | CLI만 설치 (직접 `pulumi` 명령 호출 시) |
| `pulumi/auth-actions` | GitHub OIDC 토큰을 단기 Pulumi access token으로 교환 |
| `pulumi/esc-action` | Pulumi ESC 환경 변수를 워크플로우에 주입 |
| `pulumi/esc-export-secrets-action` | GitHub Actions 시크릿을 ESC 환경으로 내보내기 |

### 저장된 access token으로 인증

워크플로우는 단일 Pulumi access token으로 Pulumi Cloud에 인증한다. `PULUMI_ACCESS_TOKEN` 환경 변수를 통해 제공된다. 개인 토큰보다 조직 또는 팀 토큰 사용을 권장하며, 워크플로우의 identity가 개인에 종속되지 않도록 한다.

**설정 방법:**

1. 저장소의 **Settings > Secrets and variables > Actions**에서 `PULUMI_ACCESS_TOKEN`이라는 이름의 encrypted secret으로 토큰을 추가한다.
2. 워크플로우에서 `secrets` 컨텍스트로 읽는다.

```yaml
env:
  PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
```

### OIDC 인증 (권장)

저장된 정적 토큰을 완전히 제거할 수 있다. GitHub Actions가 워크플로우 job에 대해 단기 OIDC 토큰을 발급하면, `pulumi/auth-actions` 액션이 이를 단기 Pulumi access token으로 교환한다. 장기 비밀이 저장소 secret으로 저장될 필요가 없다.

`pulumi/esc-action`과 함께 사용하면 클라우드 자격 증명, 시크릿, 구성을 ESC 환경에서 가져올 수 있다. 이 방식이 권장되며, 다음과 같은 이점이 있다.

| 이점 | 설명 |
|------|------|
| **Provider-agnostic** | AWS, Azure, Google Cloud 등 동일한 패턴으로 모든 클라우드에 작동 |
| **Portable** | 동일한 ESC 환경이 로컬 및 모든 CI/CD 시스템에서 작동 |
| **Centralized** | 자격 증명 구성이 ESC에 집중되고 개별 워크플로우에 흩어지지 않음 |

OIDC를 사용하려면 `id-token: write` 권한이 필요하다. PR 코멘트 작성 시 `pull-requests: write`도 추가한다.

```yaml
permissions:
  id-token: write
  contents: read
  pull-requests: write
```

**OIDC + ESC 인증 예시 (저장된 secret 없이):**

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

### Trunk-based 워크플로우 (Go)

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
      - uses: actions/setup-go@v6
        with:
          go-version: 'stable'
      - run: go mod download
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
      - uses: actions/setup-go@v6
        with:
          go-version: 'stable'
      - run: go mod download
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

### Trunk-based 워크플로우 (.NET)

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
      - uses: actions/setup-dotnet@v5
        with:
          dotnet-version: '10.0.x'
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
      - uses: actions/setup-dotnet@v5
        with:
          dotnet-version: '10.0.x'

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

> **Note:** .NET 런타임은 Pulumi 실행 중에 프로젝트를 복원하고 빌드하므로 별도의 install 단계가 필요하지 않다.

### Trunk-based 워크플로우 (Java)

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
      - uses: actions/setup-java@v5
        with:
          distribution: temurin
          java-version: '21'
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
      - uses: actions/setup-java@v5
        with:
          distribution: temurin
          java-version: '21'

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

> **Note:** Java 런타임은 Pulumi 실행 중에 종속성을 해석하고 빌드하므로 별도의 install 단계가 필요하지 않다.

### 프로덕션 승격

릴리스를 승격하려면 `release-*` 패턴에 매칭되는 태그를 push한다.

```bash
git tag release-2026-05-20
git push origin release-2026-05-20
```

프로덕션을 자체 스택에 두고 태그로만 배포하면 각 프로덕션 업데이트가 추적 가능한 단일 Git 작업이 되며, 테스트되지 않은 커밋에서 프로덕션이 배포되지 않는다.

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

### Managing GitHub with Pulumi

GitHub 자체(저장소, 팀, branch protection 규칙, GitHub Actions secrets 등)를 Pulumi Registry의 GitHub provider로 코드로 관리할 수 있다. 이 가이드에서 설명하는 워크플로우 secret과 저장소 설정을 Pulumi 프로그램의 일부로 정의할 수 있다.

### Additional resources

- [Continuous delivery](https://www.pulumi.com/docs/iac/operations/continuous-delivery/) -- CI/CD에서 Pulumi 실행 개요
- [pulumi/actions](https://github.com/pulumi/actions) -- Pulumi GitHub Action의 전체 입력 참조
- [Pulumi GitHub App](https://www.pulumi.com/docs/iac/operations/continuous-delivery/github-actions/) -- Pulumi Cloud의 풍부한 PR 코멘트 및 커밋 체크
- [Pulumi ESC](https://www.pulumi.com/docs/esc/) -- 워크플로우와 개발자에게 일관된 자격 증명, 시크릿, 구성 제공
- [OIDC issuers](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/) -- CI/CD 시스템의 OIDC 토큰을 단기 Pulumi access token으로 교환
- [Review Stacks](https://www.pulumi.com/docs/deployments/review-stacks/) -- 각 PR에 대해 자동 생성되는 임시 환경
- [CI/CD troubleshooting](https://www.pulumi.com/docs/iac/operations/continuous-delivery/troubleshooting/) -- 파이프라인에서 발생하는 일반적인 실패 진단

---

## GitLab CI/CD

GitLab CI/CD는 `.gitlab-ci.yml` 파일로 파이프라인을 정의한다. 공식 `pulumi/pulumi` 컨테이너 이미지를 사용하여 CLI를 직접 호출한다. CLI를 직접 실행하므로 어떤 언어로 작성된 프로그램이든 모든 클라우드 프로바이더에서 작동한다.

Pulumi는 CLI와 언어 런타임이 사전 설치된 공식 컨테이너 이미지를 주간 릴리스 주기로 배포한다.

### Prerequisites

시작하기 전에 다음이 필요하다.

- Pulumi Cloud 계정 및 조직
- GitLab 프로젝트
- 해당 프로젝트에 커밋된 Pulumi 프로그램 (없으면 Get started 가이드 참조)

### 저장된 access token으로 인증

파이프라인은 단일 Pulumi access token으로 Pulumi Cloud에 인증한다. `PULUMI_ACCESS_TOKEN` 환경 변수를 통해 제공된다. 개인 토큰보다 조직 또는 팀 토큰 사용을 권장한다.

**설정 방법:**

1. 프로젝트의 **Settings > CI/CD > Variables**에서 `PULUMI_ACCESS_TOKEN`이라는 이름의 CI/CD 변수로 토큰을 추가한다.
2. **Masked**를 선택하여 job log에 토큰이 나타나지 않도록 한다.
3. Pulumi CLI가 환경 변수에서 자동으로 읽으므로 별도의 `pulumi login`이 필요 없다.

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

### Trunk-based 워크플로우 (Go)

```yaml
# .gitlab-ci.yml
stages:
  - preview
  - deploy

default:
  image:
    name: pulumi/pulumi-go:latest
    entrypoint: [""]

variables:
  PULUMI_STACK_STAGING: acme/website/staging
  PULUMI_STACK_PRODUCTION: acme/website/production

.pulumi:
  before_script:
    - cd infra
    - go mod download

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

### Trunk-based 워크플로우 (.NET)

```yaml
# .gitlab-ci.yml
stages:
  - preview
  - deploy

default:
  image:
    name: pulumi/pulumi-dotnet:latest
    entrypoint: [""]

variables:
  PULUMI_STACK_STAGING: acme/website/staging
  PULUMI_STACK_PRODUCTION: acme/website/production

# .NET 런타임은 Pulumi 실행 중에 프로젝트를 복원하고 빌드한다.
.pulumi:
  before_script:
    - cd infra

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

### Trunk-based 워크플로우 (Java)

```yaml
# .gitlab-ci.yml
stages:
  - preview
  - deploy

default:
  image:
    name: pulumi/pulumi-java:latest
    entrypoint: [""]

variables:
  PULUMI_STACK_STAGING: acme/website/staging
  PULUMI_STACK_PRODUCTION: acme/website/production

# Java 런타임은 Pulumi 실행 중에 프로젝트를 해석하고 빌드한다.
.pulumi:
  before_script:
    - cd infra

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

> **Note:** `pulumi up --yes` 플래그는 비대화형 파이프라인에서 필수인, 대화형 확인 프롬프트 없이 변경을 적용한다. `environment` 키워드는 각 배포를 GitLab environment에 기록하여 배포 이력과 원클릭 롤백을 제공한다.

### 프로덕션 승격

릴리스를 승격하려면 `release-*` 패턴에 매칭되는 태그를 push한다.

```bash
git tag release-2026-05-21
git push origin release-2026-05-21
```

### 결과 보고

파이프라인이 merge request에서 `pulumi preview`를 실행할 때, 제안된 변경을 merge request 자체에 요약하고 싶을 것이다. Pulumi GitLab 통합이 이를 수행한다. GitLab 그룹을 Pulumi Cloud에 한 번 연결하면, Pulumi가 리소스 변경 요약( Pulumi Cloud 콘솔 링크 포함)을 merge request 코멘트로 게시하고 commit 상태 체크도 함께 제공한다. 이는 Pulumi를 실행하는 CI/CD 시스템에 관계없이 그룹 내 모든 프로젝트에 작동한다.

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

같은 resource group의 job은 동시에 실행되지 않고 큐잉되며, 다른 group(여기서는 staging과 production)은 병렬로 실행된다.

### Managing GitLab with Pulumi

GitLab 자체(프로젝트, 그룹, branch protection 규칙, CI/CD 변수 등)를 Pulumi Registry의 GitLab provider로 코드로 관리할 수 있다. 이 가이드에서 설명하는 CI/CD 변수와 프로젝트 설정을 Pulumi 프로그램의 일부로 정의할 수 있다.

### Additional resources

- [Continuous delivery](https://www.pulumi.com/docs/iac/operations/continuous-delivery/) -- CI/CD에서 Pulumi 실행 개요
- [Pulumi GitLab integration](https://www.pulumi.com/docs/integrations/version-control/gitlab/) -- Pulumi Cloud의 merge request 코멘트, commit 상태 체크, review stacks
- [Configuring OpenID Connect for GitLab](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/gitlab/) -- GitLab을 신뢰할 수 있는 OIDC 발급자로 등록
- [OIDC issuers](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/) -- CI/CD 시스템의 OIDC 토큰을 단기 Pulumi access token으로 교환
- [Pulumi ESC](https://www.pulumi.com/docs/esc/) -- 파이프라인과 개발자에게 일관된 자격 증명, 시크릿, 구성 제공
- [Review Stacks](https://www.pulumi.com/docs/deployments/review-stacks/) -- 각 merge request에 대해 자동 생성되는 임시 환경
- [CI/CD troubleshooting](https://www.pulumi.com/docs/iac/operations/continuous-delivery/troubleshooting/) -- 파이프라인에서 발생하는 일반적인 실패 진단

---

## CircleCI

CircleCI는 `.circleci/config.yml`에 파이프라인을 정의하며, Pulumi Orb로 CLI를 설치하고 실행한다. Orb는 CLI 명령을 실행하므로 어떤 언어로 작성된 Pulumi 프로그램이든 모든 클라우드 프로바이더에서 작동한다.

### Prerequisites

시작하기 전에 다음이 필요하다.

- Pulumi Cloud 계정 및 조직
- Git 저장소에 연결된 CircleCI 계정과 프로젝트
- Pulumi 프로그램 (없으면 Get started 가이드 참조)

### Orb 추가

```yaml
version: 2.1

orbs:
  pulumi: pulumi/pulumi@2.2.0
```

### 인증

`PULUMI_ACCESS_TOKEN`을 CircleCI [context](https://circleci.com/docs/contexts/) 또는 프로젝트 환경 변수에 저장한다. `pulumi/login` 명령이 이 환경 변수에서 토큰을 읽는다. Context는 조직 수준에서 환경 변수를 보관하므로 동일한 토큰을 여러 프로젝트에서 사용할 수 있다. 개인 토큰보다 조직 또는 팀 토큰 사용을 권장한다.

Pulumi ESC를 사용하면 클라우드 자격 증명, 시크릿, 구성을 파이프라인에 제공할 수 있다. ESC는 CI/CD 파이프라인과 개발자 머신 모두에 동일한 방식으로 값을 전달하므로, 클라우드 프로바이더 키를 CircleCI에 별도로 저장할 필요가 없다.

### 프로그램 종속성 설치 (pulumi install)

`pulumi/login` 명령은 Pulumi CLI를 설치하지만, 프로그램의 언어 종속성은 설치하지 않는다. preview 또는 update 단계 전에 `pulumi install`을 실행하는 step을 추가해야 한다.

```yaml
- run:
    name: Install dependencies
    command: pulumi install --cwd infra/
```

`pulumi install`은 프로그램의 언어 종속성과 필수 플러그인을 함께 설치하므로, 어떤 언어로 작성된 Pulumi 프로그램에도 동일한 step이 작동한다. CircleCI executor 이미지에 프로그램 언어의 런타임이 포함되어야 한다(예: TypeScript/JavaScript는 `cimg/node`, Python은 `cimg/python`).

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

### 프로덕션 승격

릴리스를 승격하려면 `release-*` 패턴에 매칭되는 태그를 push한다. 기본적으로 CircleCI는 tag push를 무시하므로, `deploy-production` job에 명시적인 `filters.tags` 항목을 설정하고 모든 branch를 무시해야 한다.

```bash
git tag release-2026-05-20
git push origin release-2026-05-20
```

### Using with other cloud providers

AWS, Google Cloud 또는 다른 프로바이더와 함께 사용하려면 필요한 자격 증명을 CircleCI context 또는 프로젝트의 환경 변수로 제공한다. Pulumi 프로그램이 preview 및 update job에서 실행될 때 이를 읽는다. Pulumi ESC를 통해 OIDC로 단기 클라우드 자격 증명을 브로커하면 장기 프로바이더 키를 CircleCI에 저장할 필요가 없다.

### Managing CircleCI with Pulumi

CircleCI 자체(프로젝트, context, 환경 변수 등)를 Pulumi Registry의 CircleCI provider로 코드로 관리할 수 있다. 이 provider는 Terraform provider에서 브리징되었으며, 프로젝트에 추가하려면:

```bash
pulumi package add terraform-provider mrolla/circleci
```

### Additional resources

- [Continuous delivery](https://www.pulumi.com/docs/iac/operations/continuous-delivery/) -- CI/CD에서 Pulumi 실행 개요
- [Pulumi ESC](https://www.pulumi.com/docs/esc/) -- 파이프라인과 개발자에게 일관된 자격 증명, 시크릿, 구성 제공
- [OIDC issuers](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/) -- CI/CD 시스템의 OIDC 토큰을 단기 Pulumi access token으로 교환
- [Review Stacks](https://www.pulumi.com/docs/deployments/review-stacks/) -- 각 PR에 대해 자동 생성되는 임시 환경
- [CI/CD troubleshooting](https://www.pulumi.com/docs/iac/operations/continuous-delivery/troubleshooting/) -- 파이프라인에서 발생하는 일반적인 실패 진단

---

## Jenkins

Jenkins는 `Jenkinsfile`에 파이프라인을 정의하며, `pulumi/pulumi` 컨테이너 이미지 내에서 Pulumi를 실행한다. 별도 Jenkins 플러그인은 필요하지 않다. `pulumi/pulumi` 컨테이너 이미지가 통합 지점이며, CLI와 모든 언어 런타임이 포함되어 있어 어떤 언어로 작성된 프로그램이든 모든 클라우드 프로바이더에서 작동한다.

### Prerequisites

시작하기 전에 다음이 필요하다.

- Pulumi Cloud 계정 및 조직
- Docker Pipeline 및 Git 플러그인이 설치된 Jenkins 환경, Docker 컨테이너를 실행할 수 있는 agent
- Jenkins가 빌드할 수 있는 Git 저장소의 Pulumi 프로그램 (없으면 Get started 가이드 참조)

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

### 클라우드 자격 증명 제공

Pulumi가 실행될 때 프로그램은 관리하는 클라우드 프로바이더의 자격 증명도 필요하다. 두 가지 방법으로 제공할 수 있다.

| 방법 | 설명 | 권장 여부 |
|------|------|-----------|
| **Pulumi ESC** | ESC 환경을 구성하여 OIDC로 단기 클라우드 자격 증명을 브로커. 파이프라인은 Pulumi access token만 저장 | 권장 |
| **Jenkins credentials** | 프로바이더의 자격 증명을 Jenkins credentials store에 저장하고 `withCredentials`로 stage에 바인딩 | 기본 |

Jenkins는 `usernamePassword` 쌍이나 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` 등의 plain `Secret text` 항목에 대한 built-in typed binding을 제공한다. Azure Credentials plugin은 `azureServicePrincipal` 바인딩을 추가한다.

> **Warning:** 클라우드 자격 증명이나 Pulumi access token을 저장소에 절대 커밋하지 마라. Jenkins credentials store에 보관하여 값이 보호되고 접근이 감사 가능하도록 하라.

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

### PR 결과 보고

Jenkins와 독립적으로, Pulumi Cloud의 버전 관리 통합이 Pulumi를 인기 있는 버전 관리 시스템에 연결한다. 통합을 구성하면 Pulumi Cloud가 인프라 변경 요약을 PR 및 merge request에 코멘트와 상태 체크로 게시하고, 각 스택 업데이트를 해당 커밋과 PR에 연결한다. 이는 Jenkins 파이프라인만으로는 제공하지 않는 리뷰 컨텍스트를 제공한다.

### Additional resources

- [Continuous delivery](https://www.pulumi.com/docs/iac/operations/continuous-delivery/) -- CI/CD에서 Pulumi 실행 개요
- [CI/CD troubleshooting guide](https://www.pulumi.com/docs/iac/operations/continuous-delivery/troubleshooting/) -- 파이프라인에서 발생하는 일반적인 실패 진단
- [Pulumi ESC](https://www.pulumi.com/docs/esc/) -- 파이프라인과 개발자에게 일관된 자격 증명, 시크릿, 구성 제공
- [OIDC Issuers](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/) -- 단기 교환 자격 증명으로 정적 토큰 제거
- [Review Stacks](https://www.pulumi.com/docs/deployments/review-stacks/) -- PR을 위한 임시 환경
- [Version Control](https://www.pulumi.com/docs/integrations/version-control/) -- PR 보고를 위해 Pulumi Cloud를 버전 관리 시스템에 연결

---

## Azure DevOps

Azure DevOps는 [Pulumi Task Extension](https://marketplace.visualstudio.com/items?itemName=pulumi.build-and-release-task)(`Pulumi@1` 태스크)을 통해 Pulumi를 실행한다. 태스크가 CLI 명령을 실행하므로 어떤 언어로 작성된 Pulumi 프로그램이든 모든 클라우드 프로바이더에서 작동한다. Azure뿐만 아니라 모든 클라우드에 사용할 수 있다.

### Prerequisites

시작하기 전에 다음이 필요하다.

- Pulumi Cloud 계정 및 조직
- Git 저장소가 포함된 Azure DevOps 조직과 프로젝트
- Pulumi 프로그램 (없으면 Get started 가이드 참조)

### Task Extension 설치

Visual Studio Marketplace에서 Azure DevOps 조직에 Pulumi Task Extension을 설치한다. 설치 후 조직의 모든 파이프라인에서 `Pulumi@1` 태스크를 사용할 수 있다.

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

태스크의 `env` 지시어를 사용하여 추가 환경 변수를 Pulumi 프로그램에 전달할 수 있다.

### Using with other cloud providers

AWS, Google Cloud 또는 다른 프로바이더와 함께 사용하려면 필요한 자격 증명을 파이프라인 변수로 전달하거나 변수 그룹을 파이프라인에 연결한다. AWS의 예:

```yaml
- task: Pulumi@1
  inputs:
    command: up
    stack: acme/website/production
    args: --yes
  env:
    AWS_ACCESS_KEY_ID: $(awsAccessKeyId)
    AWS_SECRET_ACCESS_KEY: $(awsSecretAccessKey)
```

Pulumi ESC를 사용하여 단기 클라우드 자격 증명을 브로커하면 프로바이더 키를 파이프라인 변수로 저장할 필요가 없다.

### Managing Azure DevOps with Pulumi

Azure DevOps 자체(프로젝트, 저장소, 파이프라인, 서비스 연결 등)를 Pulumi Registry의 Azure DevOps provider로 코드로 관리할 수 있다.

### Additional resources

- [Continuous delivery](https://www.pulumi.com/docs/iac/operations/continuous-delivery/) -- CI/CD에서 Pulumi 실행 개요
- [Azure DevOps version control integration](https://www.pulumi.com/docs/integrations/version-control/azure-devops/) -- Pulumi Cloud Deployments와 연결하여 push 시 배포 및 PR preview를 자체 파이프라인 없이 실행
- [Pulumi ESC](https://www.pulumi.com/docs/esc/) -- 파이프라인과 개발자에게 일관된 자격 증명, 시크릿, 구성 제공

---

## Bitbucket Pipelines

Bitbucket Pipelines는 `bitbucket-pipelines.yml`에 파이프라인을 정의하며, CLI를 직접 호출한다. 어떤 언어로 작성된 Pulumi 프로그램이든 모든 클라우드 프로바이더에서 작동한다.

### Prerequisites

시작하기 전에 다음이 필요하다.

- Pulumi Cloud 계정 및 조직
- Pipelines가 활성화된 Bitbucket Cloud 저장소
- 해당 저장소에 커밋된 Pulumi 프로그램 (없으면 Get started 가이드 참조)

### 인증

**저장된 access token:**

`PULUMI_ACCESS_TOKEN`을 **Repository settings > Repository variables**에 **Secured**로 저장한다. 이름을 `PULUMI_ACCESS_TOKEN`으로 지정하고 **Secured** 체크박스를 선택하면 값이 암호화되고 빌드 로그에서 마스킹된다. Secured 변수는 파이프라인의 모든 단계에 환경 변수로 노출된다. 개인 토큰보다 조직 또는 팀 토큰 사용을 권장한다.

> **Warning:** 항상 민감한 값(access token, 클라우드 프로바이더 키 등)을 **Secured**로 표시하라. Secured가 아닌 변수는 일반 텍스트로 저장되고 출력된다.

**OIDC 인증:**

저장된 정적 토큰을 완전히 제거할 수 있다. `oidc: true`를 설정한 스텝은 `BITBUCKET_STEP_OIDC_TOKEN` 환경 변수로 단기 OIDC 토큰에 접근할 수 있다. Bitbucket Pipelines를 Pulumi Cloud의 신뢰할 수 있는 OIDC 발급자로 등록하면, 파이프라인이 런타임에 OIDC 토큰을 단기 Pulumi access token으로 교환할 수 있다.

Pulumi ESC를 사용하면 클라우드 자격 증명, 시크릿, 구성을 파이프라인에 제공할 수 있다. ESC는 CI/CD 파이프라인과 개발자 머신 모두에 동일한 방식으로 값을 전달하므로, 클라우드 프로바이더 키를 저장소 변수에 별도로 저장할 필요가 없다.

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

`pulumi/pulumi` Docker 이미지에 Pulumi CLI와 모든 언어 런타임이 포함되어 있어 프로그램 언어에 관계없이 동일한 파이프라인이 작동한다. `pulumi install`은 프로그램의 언어 종속성과 필수 플러그인을 함께 설치한다. `pulumi preview`는 리소스를 수정하지 않고 제안된 변경을 보고하여, 리뷰어에게 병합 결과를 요약한다.

릴리스를 승격하려면 `release-*` 패턴에 매칭되는 태그를 push한다.

```bash
git tag release-2026-05-20
git push origin release-2026-05-20
```

### Cloud provider credentials

Pulumi 프로그램은 관리하는 클라우드의 자격 증명이 필요하다. **Secured** 저장소 변수로 제공하거나, Pulumi ESC를 통해 단기 클라우드 자격 증명을 브로커하여 프로바이더 키를 저장소 변수에 전혀 저장하지 않을 수도 있다.

### Report results back to Bitbucket

Pulumi Cloud Bitbucket 버전 관리 통합은 CI/CD 시스템에 관계없이 모든 배포에 대해 PR 코멘트와 커밋 상태 체크를 게시한다. 연결하면 리뷰어가 PR에서 직접 리소스 변경 요약을 볼 수 있다.

이 통합은 수작성 파이프라인을 완전히 대체할 수도 있다. push-to-deploy 및 review stacks를 사용하면 Pulumi Cloud가 Bitbucket 이벤트에 응답하여 Pulumi 호스팅 인프라에서 업데이트를 실행하므로, `bitbucket-pipelines.yml`을 유지할 필요가 없다.

### Additional resources

- [Continuous delivery](https://www.pulumi.com/docs/iac/operations/continuous-delivery/) -- CI/CD에서 Pulumi 실행 개요
- [Pulumi ESC](https://www.pulumi.com/docs/esc/) -- 파이프라인과 개발자에게 일관된 자격 증명, 시크릿, 구성 제공
- [OIDC issuers](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/) -- CI/CD 시스템의 OIDC 토큰을 단기 Pulumi access token으로 교환
- [Bitbucket version control integration](https://www.pulumi.com/docs/integrations/version-control/bitbucket/) -- Pulumi Cloud의 PR 코멘트와 커밋 상태 체크
- [Review Stacks](https://www.pulumi.com/docs/deployments/review-stacks/) -- 각 PR에 대해 자동 생성되는 임시 환경
- [CI/CD troubleshooting](https://www.pulumi.com/docs/iac/operations/continuous-delivery/troubleshooting/) -- 파이프라인에서 발생하는 일반적인 실패 진단

---

## Buildkite

Buildkite는 `.buildkite/pipeline.yml`에 파이프라인을 정의하며, 공식 `pulumi` 플러그인으로 CLI를 설치한다. 플러그인이 CLI 명령을 실행하므로 어떤 언어로 작성된 Pulumi 프로그램이든 모든 클라우드 프로바이더에서 작동한다.

### Prerequisites

시작하기 전에 다음이 필요하다.

- Pulumi Cloud 계정 및 조직
- 클러스터와 agent가 있는 Buildkite 계정
- Buildkite 파이프라인에 연결된 Git 저장소
- Pulumi 프로그램 (없으면 Get started 가이드 참조)

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

### Dynamic pipelines

Buildkite는 빌드 시간에 파이프라인 단계를 동적으로 생성할 수 있다. 각 Pulumi 스택이나 프로젝트별로 단계를 펼치고 싶을 때 유용하며, 각각을 하드코딩하지 않아도 된다.

### Cache volumes

호스팅 agent에서 cache volumes는 빌드 간에 디렉터리를 유지한다. `~/.pulumi/plugins`와 언어 패키지(`node_modules` 등)를 캐시하면 실행 속도가 향상된다. Pulumi 플러그인 버전은 이를 사용하는 Pulumi 패키지 버전과 1:1로 연결되므로, 해당 패키지가 변경되지 않는 한 캐시가 올바르게 유지된다.

### Managing Buildkite with Pulumi

Buildkite 자체(파이프라인, 팀, agent token 등)를 Pulumiverse Buildkite provider로 코드로 관리할 수 있다. 이 provider는 커뮤니티에서 유지 관리되며 Pulumi Registry에서 사용할 수 있다.

### Additional resources

- [Continuous delivery](https://www.pulumi.com/docs/iac/operations/continuous-delivery/) -- CI/CD에서 Pulumi 실행 개요
- [Pulumi ESC](https://www.pulumi.com/docs/esc/) -- 파이프라인과 개발자에게 일관된 자격 증명, 시크릿, 구성 제공
- [OIDC issuers](https://www.pulumi.com/docs/administration/access-identity/oidc-issuers/) -- CI/CD 시스템의 OIDC 토큰을 단기 Pulumi access token으로 교환
- [Review Stacks](https://www.pulumi.com/docs/deployments/review-stacks/) -- 각 PR에 대해 자동 생성되는 임시 환경

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
