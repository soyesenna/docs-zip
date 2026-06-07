# Pulumi 개발자 문서

> Pulumi 공식 문서(https://www.pulumi.com/docs/)를 기반으로 정리한 한국어 개발자 가이드입니다.
> 최종 업데이트: 2026-06-07

Pulumi는 TypeScript, Python, Go, C#, Java 등 범용 프로그래밍 언어로 인프라를 정의하고 관리하는 Infrastructure as Code (IaC) 프레임워크입니다. AWS, Azure, GCP, Kubernetes 등 100개 이상의 클라우드/서비스 프로바이더를 지원하며, Pulumi Cloud를 통해 상태 관리, 배포 자동화, 시크릿 관리, 정책 강제 등의 기능을 제공합니다. Pulumi Neo(AI 에이전트)를 통해 자연어로 인프라를 생성하고 관리할 수 있습니다.

---

## 문서 목록

### 기본

| # | 문서 | 설명 |
| --- | --- | --- |
| 00 | [Pulumi 개요](./00-overview.md) | IaC 프레임워크 소개, 핵심 특징, 아키텍처, Pulumi Cloud vs OSS |
| 01 | [설치 가이드](./01-installation.md) | macOS/Linux/Windows 설치, Docker, 버전 관리, 초기 설정 |
| 02 | [클라우드별 시작하기](./02-getting-started.md) | AWS, Azure, GCP, Kubernetes 퀵스타트, 공통 워크플로우 |

### 핵심 개념

| # | 문서 | 설명 |
| --- | --- | --- |
| 03 | [프로젝트와 스택](./03-projects-stacks.md) | Pulumi.yaml, 스택 관리, 조직 구조, 멀티스택 워크플로우 |
| 04 | [리소스](./04-resources.md) | Custom/Component Resource, Resource Options, 수명주기, URN |
| 05 | [Inputs & Outputs](./05-inputs-outputs.md) | Input/Output 타입, apply, 의존성 추적, Provider Functions |
| 06 | [프로바이더](./06-providers.md) | 클라우드 프로바이더, Terraform 프로바이더 호환, 커스텀 프로바이더 |
| 07 | [설정과 시크릿](./07-configuration-secrets.md) | pulumi config, 시크릿 암호화, Stack Settings, passphrase/KMS |
| 08 | [상태와 백엔드](./08-state-backends.md) | 상태 관리, Pulumi Cloud/셀프 매니지드 백엔드, 상태 잠금, 복구 |
| 09 | [컴포넌트와 패키지](./09-components-packages.md) | Component Resource, Packages, Plugins, Converters, Stash |

### CLI & 언어

| # | 문서 | 설명 |
| --- | --- | --- |
| 10 | [CLI 레퍼런스](./10-cli-reference.md) | 주요 명령어(up/preview/destroy/stack 등), 환경변수, 종료 코드, pulumi api |
| 11 | [언어 및 SDK](./11-languages.md) | TypeScript, Python, Go, C#, Java SDK 비교, YAML/HCL 지원 |

### 운영

| # | 문서 | 설명 |
| --- | --- | --- |
| 12 | [테스팅](./12-testing.md) | 단위 테스트, 통합 테스트, Automation API 테스트, 모킹 |
| 13 | [CI/CD 통합](./13-cicd.md) | GitHub Actions, GitLab CI, Jenkins, Azure DevOps 등 CI/CD 설정 |
| 14 | [마이그레이션](./14-migration.md) | Terraform, CloudFormation, ARM, CDK, CDKTF, K8s YAML에서 전환 |

### 플랫폼

| # | 문서 | 설명 |
| --- | --- | --- |
| 15 | [Pulumi Cloud 관리](./15-pulumi-cloud.md) | 조직, RBAC, SAML SSO, SCIM, 감사 로그, Self-Hosted 배포 |
| 16 | [Pulumi Deployments](./16-deployments.md) | 배포 자동화, 트리거, OIDC, Drift Detection, Runner |
| 17 | [Pulumi ESC](./17-esc.md) | 환경·시크릿·설정 관리, 프로바이더, 로테이션, CLI |
| 18 | [Insights & IDP](./18-insights-idp.md) | 리소스 Discovery, Policy, Internal Developer Platform |
| 22 | [Pulumi AI](./22-pulumi-ai.md) | Pulumi Neo AI 에이전트, AI Skills, Automations, IDE/CLI 통합 |

### 고급

| # | 문서 | 설명 |
| --- | --- | --- |
| 19 | [Automation API](./19-automation-api.md) | 프로그래밍 방식 인프라 관리, Workspace, 커스텀 배포 도구 |
| 20 | [IaC 비교](./20-comparisons.md) | vs Terraform, CloudFormation, CDK, Crossplane, Helm, OpenTofu |
| 21 | [보안과 컴플라이언스](./21-security.md) | 최소 권한, OIDC, 시크릿 관리, 감사 로그, SOC 2/HIPAA |

---

## 빠른 참조

### 설치

```bash
# macOS
brew install pulumi/tap/pulumi

# Linux
curl -fsSL https://get.pulumi.com | sh

# Windows
choco install pulumi
```

### 기본 워크플로우

```bash
pulumi new aws-typescript    # 프로젝트 생성
pulumi up                    # 인프라 배포
pulumi preview               # 변경 사항 미리보기
pulumi destroy               # 인프라 삭제
pulumi stack ls              # 스택 목록
```

### 핵심 파일

| 파일 | 설명 |
| --- | --- |
| `Pulumi.yaml` | 프로젝트 설정 (이름, 런타임, 설명) |
| `Pulumi.<stack>.yaml` | 스택별 설정 (config, 시크릿) |
| `index.ts` / `__main__.py` | 인프라 정의 진입점 |

### 지원 언어

| 언어 | 상태 | 설치 패키지 |
| --- | --- | --- |
| TypeScript/JavaScript | 정식 지원 | `@pulumi/pulumi` |
| Python | 정식 지원 | `pulumi` |
| Go | 정식 지원 | `github.com/pulumi/pulumi/sdk/v3/go/pulumi` |
| C#/.NET | 정식 지원 | `Pulumi` |
| Java | 정식 지원 | `com.pulumi` |
| YAML | 정식 지원 | 별도 설치 불요 |
| HCL | 실험적 | `pulumi convert` |

### 주요 클라우드 프로바이더

| 프로바이더 | 패키지 | 설치 |
| --- | --- | --- |
| AWS | `@pulumi/aws` | `npm install @pulumi/aws` |
| Azure | `@pulumi/azure-native` | `npm install @pulumi/azure-native` |
| GCP | `@pulumi/gcp` | `npm install @pulumi/gcp` |
| Kubernetes | `@pulumi/kubernetes` | `npm install @pulumi/kubernetes` |

---

## 원문 링크

- 공식 문서: https://www.pulumi.com/docs/
- Pulumi Registry: https://www.pulumi.com/registry/
- API 레퍼런스: https://www.pulumi.com/docs/reference/
- GitHub: https://github.com/pulumi/pulumi
- 튜토리얼: https://www.pulumi.com/tutorials/
- 블로그: https://www.pulumi.com/blog/
