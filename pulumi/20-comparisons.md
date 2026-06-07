# Pulumi IaC 도구 비교

> **원문**
> - https://www.pulumi.com/docs/iac/comparisons/
> - https://www.pulumi.com/docs/iac/comparisons/terraform/
> - https://www.pulumi.com/docs/iac/comparisons/cloudformation/
> - https://www.pulumi.com/docs/iac/comparisons/aws-cdk/
> - https://www.pulumi.com/docs/iac/comparisons/cdktf/
> - https://www.pulumi.com/docs/iac/comparisons/crossplane/
> - https://www.pulumi.com/docs/iac/comparisons/helm/
> - https://www.pulumi.com/docs/iac/comparisons/k8s-yaml-dsls/
> - https://www.pulumi.com/docs/iac/comparisons/serverless/
> - https://www.pulumi.com/docs/iac/comparisons/opentofu/
> - https://www.pulumi.com/docs/iac/comparisons/arm-templates/
> - https://www.pulumi.com/docs/iac/comparisons/cloud-sdks/
> - https://www.pulumi.com/docs/iac/comparisons/chef-puppet-etc/
> - https://www.pulumi.com/docs/iac/comparisons/custom/
> - https://www.pulumi.com/docs/iac/comparisons/terraform/opentofu/

Pulumi는 범용 프로그래밍 언어(Python, TypeScript, JavaScript, Go, .NET, Java)와 YAML로 클라우드 인프라를 정의하는 Infrastructure as Code(IaC) 플랫폼이다. 다양한 IaC 도구와 기능이 겹치지만, 언어 지원·실행 모델·클라우드 범위·상태 관리·비밀 처리 방식에서 유의미한 차이가 있다. 이 문서는 Pulumi와 주요 IaC 도구 13종을 공식 비교 문서에 근거하여 정리한다.

---

## 전체 비교 표

### 언어·클라우드 범위·실행 모델

| 구분 | Pulumi | Terraform | CloudFormation | AWS CDK | CDKTF | Crossplane | Helm | K8s YAML Manifests | Serverless Framework | OpenTofu | ARM Templates/Bicep | Cloud SDKs |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **언어** | Python, TypeScript, JavaScript, Go, C#, Java, YAML | HCL(HashiCorp Configuration Language) | JSON, YAML 템플릿 + 고유 함수(`Fn::Join`, `Fn::If` 등) | TypeScript, JavaScript, Python, Go, C#, Java | TypeScript, Python, Go, C#, Java | Kubernetes YAML 매니페스트; Go(프로바이더·컴포지션 함수 작성용) | YAML + Go 템플릿 문법 | YAML 또는 JSON(Kustomize로 베이스·오버레이 레이어링 가능, 루프·조건문·변수 없음) | `serverless.yml`(YAML) + 변수 시스템 | HCL | JSON 또는 Bicep(DSL → JSON 컴파일) | 클라우드 SDK가 지원하는 언어 전체(AWS Boto/Python, AWS SDK for JS/Go/Java/C# 등) |
| **클라우드 범위** | 모든 클라우드 및 SaaS(Pulumi Registry) | 모든 클라우드 및 SaaS(Terraform Registry) | AWS 전용 | AWS 전용 | Terraform 프로바이더 전용 | 모든 클라우드(Crossplane 프로바이더) | Kubernetes 전용 | Kubernetes API 객체 전용(비-K8s 클라우드 리소스는 별도 도구 필요) | AWS 전용(V4 기준) | 모든 클라우드(OpenTofu Registry) | Azure 전용 | SDK가 지원하는 클라우드 범위에 한정(멀티클라우드 시 각 SDK 개별 사용) |
| **트랜스파일** | 없음 — 호스트 언어에서 직접 실행 | 없음 — HCL을 Terraform CLI가 직접 해석 | 없음 — CloudFormation 서비스가 직접 해석 | 있음 — CloudFormation JSON/YAML으로 합성(synth) 후 CloudFormation이 배포 | 있음 — Terraform JSON으로 합성 후 Terraform CLI가 배포 | 없음 — YAML 매니페스트를 Crossplane 컨트롤러가 직접 조정(reconcile) | 없음 — Go 템플릿으로 렌더 후 `helm` CLI가 적용 | 없음 — 매니페스트를 Kubernetes API 서버에 직접 전송 | 있음 — `serverless.yml`을 CloudFormation 템플릿으로 컴파일 후 CloudFormation 배포 | 없음 — HCL을 OpenTofu CLI가 직접 해석 | Bicep → ARM JSON 컴파일 후 ARM 서비스가 배포; JSON은 직접 해석 | 해당 없음 — 클라우드 API를 직접 호출하는 명령형(imperative) 라이브러리 |
| **실행 모델** | 로컬 CLI, Automation API(프로그래밍), Pulumi Deployments(원격) | 로컬 CLI, HCP Terraform(원격) | AWS 관리 서비스(Console, CLI, SDK, Change Sets) | `cdk deploy` → CloudFormation 서비스 | `cdktf deploy` → Terraform CLI | Kubernetes 클러스터 내 컨트롤러가 지속 조정(GitOps) | 로컬 `helm` CLI | 로컬 `kubectl` CLI 또는 GitOps 컨트롤러(Argo CD, Flux); K8s 자체는 중앙 오케스트레이션 서비스 없음 | 로컬 `serverless` CLI, Dashboard | 로컬 `tofu` CLI; 원격 실행은 서드파티 필요 | Azure 관리 서비스(Portal, CLI, PowerShell, Bicep CLI, REST API) | 애플리케이션 코드에 직접 삽입(명령형 호출) |

### 상태 관리·비밀·롤백·정책

| 구분 | Pulumi | Terraform | CloudFormation | AWS CDK | CDKTF | Crossplane | Helm | K8s YAML Manifests | Serverless Framework | OpenTofu | ARM Templates/Bicep | Cloud SDKs |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **상태 관리** | Pulumi Cloud(기본) 또는 자체 관리(S3, Azure Blob, GCS, 로컬 등) | 로컬 파일(기본); 원격 백엔드(S3, GCS, Consul, HCP Terraform) | CloudFormation 서비스 내부 관리; 사용자 접근 불가 | CloudFormation 서비스 내부(상태 파일 없음) | Terraform 상태 모델(로컬/원격/클라우드) | Kubernetes etcd(리소스 객체의 status로 저장) | 클러스터 내 Secret 또는 ConfigMap으로 release 메타데이터 저장 | 별도 상태 파일 없음; 라이브 클러스터(etcd)가 진실의 원천, `kubectl apply`가 server-side apply로 관리 필드 추적 | CloudFormation 스택(독립 상태 저장소 없음) | 로컬 파일(기본); 원격 백엔드(S3, GCS, Azure Blob, HTTP) | ARM 서비스 내부(Azure 구독 내 배포 이력으로 관리); 사용자 접근 불가 | 없음 — 프로비저닝 상태 추적·체크포인트 기능 없음(직접 구현 필요) |
| **비밀 관리** | 전송 중·저장 시 암호화(기본), 스택별 암호화 키, 플러그형 KMS(AWS KMS, Azure Key Vault, GCP KMS, Vault) | 상태 파일 내 미암호화; HCP Terraform은 저장 시 암호화; Vault는 별도 제품 | 기본 없음; `NoEcho` 파라미터 또는 Secrets Manager/SSM 동적 참조 | 기본 없음; CloudFormation의 `NoEcho`/동적 참조 상속 | 기본 없음 | Kubernetes Secrets(클러스터 설정에 의존) | 기본 없음; `helm-secrets` 등 커뮤니티 플러그인 또는 외부 스토어 | Kubernetes Secret 객체는 base64 인코딩만(미암호화); etcd 암호화는 클러스터 수준에서 별도 구성 필요 | 기본 없음; SSM Parameter Store/Secrets Manager 변수 참조 | 상태·플랜 파일 암호화(1.7+); 개별 변수는 일급 비밀 프리미티브 아님 | 기본 없음; `secureString`/`secureObject` 파라미터 또는 Key Vault 참조 | 없음 — 비밀 처리를 직접 구현해야 함(클라우드 비밀 관리 서비스 별도 호출 가능) |
| **실패 시 롤백** | 부분 업데이트 상태 유지; 이후 `pulumi up`으로 수렴 | 부분 적용 상태 유지; 이후 `terraform apply`로 수렴 | 자동 스택 롤백(기본); 롤백 트리거·`DisableRollback` 설정 가능 | CloudFormation 자동 롤백 상속 | 자동 롤백 없음; 코드 수정 후 재실행 필요 | 트랜잭션 롤백 없음; 컨트롤러가 지속 재시도 | `helm rollback`으로 이전 리비전 복원 | `kubectl apply`는 트랜잭션 롤백 없음; 워크로드 컨트롤러는 `kubectl rollout undo`로 리비전 롤백 지원 | `serverless rollback` + CloudFormation 자동 롤백 | 자동 롤백 없음; `tofu apply` 재실행으로 수렴 | 오류 시 롤백 옵션(배포 시 활성화 가능, 마지막 성공 배포로 복원) | 없음 — 실패 시 복구 로직을 직접 작성해야 함(부분 생성 리소스 수동 정리 필요) |
| **Policy as Code** | Pulumi Policies(오픈 소스, Python/TypeScript/Rego); 상용 플랜에 중앙 관리 + 준법 팩(CIS, HITRUST, NIST, PCI DSS) | Sentinel(상용, HCP Terraform/Enterprise 전용), OPA | CloudFormation Hooks, CloudFormation Guard | CloudFormation Hooks, CloudFormation Guard(상속) | 일급 없음; OPA 또는 Sentinel(상용) | 기본 없음; OPA Gatekeeper, Kyverno 등 Kubernetes 어드미션 컨트롤러 | 기본 없음; Kyverno, OPA Gatekeeper 등 | 기본 없음; OPA Gatekeeper, Kyverno 등 어드미션 컨트롤러로 클러스터 내 시행 | 기본 없음; CloudFormation Guard, OPA 등 외부 도구 | 기본 없음; OPA, Checkov 등 외부 도구 | Azure Policy(별도 Azure 서비스, 템플릿 저작 루프와 분리) | 없음 — 정책 시행·컴플라이언스 검사 불가 |
| **프로그래밍 API** | Automation API — CLI 호출 없이 `up`, `preview`, `destroy`를 호스트 애플리케이션에 내장 가능 | 해당 없음 | 해당 없음(ARM REST API는 있지만 임베딩 SDK는 아님) | 해당 없음 | 해당 없음 | Kubernetes API 자체가 프로그래밍 인터페이스 | Go 전용 Helm SDK; 다른 언어는 CLI 셸아웃 | Kubernetes API·클라이언트 라이브러리로 개별 객체 조작 가능하나 apply/preview/destroy 수명주기 오케스트레이션 SDK는 없음 | 해당 없음 | 해당 없음 | 해당 없음 | SDK 자체가 API — 클라우드 API를 직접 호출하지만 IaC 수명주기(프리뷰·diff·상태 체크포인트)는 없음 |
| **오픈 소스 라이선스** | Apache 2.0 | Business Source License 1.1 | 폐쇄 소스(AWS 서비스) | CDK 프레임워크: Apache 2.0; 배포 엔진(CloudFormation): 폐쇄 소스 | MPL 2.0(보관됨); 배포용 Terraform CLI: BSL 1.1 | Apache 2.0 | Apache 2.0 | Apache 2.0(Kubernetes, kubectl) | V3까지 MIT; V4는 독점 라이선스 | MPL 2.0 | Bicep: MIT; ARM 배포 서비스: 폐쇄 소스 | 클라우드 SDK별 상이(AWS SDK: Apache 2.0, Azure SDK: MIT, GCP SDK: Apache 2.0 등) |
| **상용 옵션** | Pulumi Cloud | HCP Terraform / Terraform Enterprise | 없음(AWS의 일부) | 없음(AWS의 일부) | HCP Terraform | 없음(Upbound에서 관리형 컨트롤 플레인 제공) | 없음 | 없음(매니페스트는 K8s API 입력 형식; 관리형 K8s는 클라우드 프로바이더가 판매) | Serverless Subscription(V4, 수입 임계값 초과 조직 필수) | 없음(서드파티: Spacelift, env0, Scalr) | 없음(Azure의 일부) |

---

## Pulumi vs Terraform

> https://www.pulumi.com/docs/iac/comparisons/terraform/

### 개요

Pulumi와 HashiCorp Terraform은 모두 선언적 IaC 도구이다. Pulumi는 범용 언어(Python, TypeScript, JavaScript, Go, .NET, Java, YAML)로 인프라를 정의하고 Pulumi Registry를 통해 모든 클라우드/SaaS를 지원한다. Terraform은 HCL(HashiCorp Configuration Language)을 사용하며 HashiCorp 프로바이더 생태계에 의존한다. HashiCorp는 2025년 2월 IBM에 인수되었다.

### 주요 차이점

| 차이점 | Pulumi | Terraform |
| --- | --- | --- |
| **언어** | 범용 언어 6종 + YAML. 클래스, 루프, 조건문, 패키지 관리자, IDE 기능(자동완성, 타입 검사, 리팩토링), 테스트 프레임워크 활용 가능 | HCL(DSL). 프로젝트가 복잡해질수록 제어 흐름·동적 블록 가독성 저하 |
| **프로바이더** | Pulumi Registry(브릿지·네이티브·파라미터화·동적 프로바이더); Kubernetes, Azure Native, AWS Cloud Control, Google Cloud Native는 API 스키마에서 자동 생성 → 신규 API 당일 지원; 모든 Terraform 프로바이더를 Pulumi 프로바이더로 변환 가능 | Terraform Registry의 HashiCorp·커뮤니티 프로바이더 |
| **비밀 처리** | 일급 프리미티브. 전송 중·저장 시 암호화, 스택별 키, 파생 값도 자동 암호화 | 상태 파일 내 미암호화; HCP Terraform은 저장 시 암호화; Vault는 별도 제품 |
| **Automation API** | CLI 없이 프로그래밍 방식으로 배포 구동 가능(임베딩 SDK) | 해당 기능 없음 |
| **정책** | Pulumi Policies — 오픈 소스(Apache 2.0), Python/TypeScript/Rego | Sentinel — 상용, HCP Terraform/Enterprise 전용 |
| **라이선스** | Apache 2.0 | Business Source License 1.1 (v1.6부터) |
| **모듈성** | Component Resources(런타임 객체, 부모-자식 관계 명시); Pulumi Package로 한 언어에서 작성→모든 언어에서 소비 | Terraform 모듈(HCL 정적 단위, 상태에서 평면적 인스턴스) |

### 마이그레이션 경로

1. **병행 사용** — Pulumi 프로그램에서 기존 Terraform 상태 파일 참조 가능
2. **Pulumi Cloud를 Terraform 상태 백엔드로 사용** — 암호화·RBAC·감사·상태 잠금 제공
3. **기존 Terraform 프로바이더 활용** — Terraform 브릿지로 모든 Terraform 프로바이더를 Pulumi에서 사용
4. **Terraform 모듈 직접 소비** — Pulumi 프로그램에서 기존 HCL 모듈을 그대로 호출
5. **HCL → Pulumi 변환** — `pulumi convert --from terraform`으로 HCL을 원하는 언어의 Pulumi 코드로 변환
6. **기존 리소스 Import** — `pulumi import`로 이미 프로비저닝된 리소스를 Pulumi 관리로 편입

---

## Pulumi vs AWS CloudFormation

> https://www.pulumi.com/docs/iac/comparisons/cloudformation/

### 개요

Pulumi와 AWS CloudFormation은 모두 AWS 인프라를 위한 선언적 IaC 도구이다. Pulumi는 범용 언어로 모든 클라우드/SaaS를 지원하지만, CloudFormation은 JSON/YAML 템플릿으로 AWS 리소스만 프로비저닝한다.

### 주요 차이점

| 차이점 | Pulumi | CloudFormation |
| --- | --- | --- |
| **언어** | 범용 언어 6종 + YAML | JSON/YAML + 고유 함수(`Fn::Join`, `Fn::If`, `Fn::ForEach` 등) |
| **클라우드** | 모든 클라우드/SaaS | AWS 전용; 서드파티는 CloudFormation Registry 확장·Lambda 기반 Custom Resources로 제한적 지원 |
| **실행** | 로컬 CLI, Automation API, Pulumi Deployments | AWS 관리 서비스(Console, CLI, SDK, Change Sets) |
| **롤백** | 부분 업데이트 후 수렴; 직접 제어 | 자동 스택 롤백(기본); 롤백 트리거 설정 가능 |
| **비밀** | 일급 암호화 프리미티브 | 기본 없음; `NoEcho` 파라미터 또는 Secrets Manager/SSM 동적 참조 |
| **모듈성** | Component Resources + Pulumi Packages(언어 간 소비 가능) | Nested Stacks, CloudFormation Modules, Cross-stack References |
| **AWS 리소스 커버리지** | AWS Classic 프로바이더(Terraform 기반) + AWS Cloud Control 프로바이더(Cloud Control API 기반, 당일 지원) | CloudFormation 자체가 AWS 리소스의 원천 |

---

## Pulumi vs AWS CDK

> https://www.pulumi.com/docs/iac/comparisons/aws-cdk/

### 개요

Pulumi와 AWS CDK는 모두 범용 언어로 인프라를 작성하지만, 실행 경로가 다르다. Pulumi는 자체 배포 엔진에서 직접 실행하고 모든 클라우드/SaaS를 지원한다. AWS CDK는 CloudFormation 템플릿으로 합성(synth)한 뒤 CloudFormation 서비스가 배포하며, AWS 전용이다.

### 주요 차이점

| 차이점 | Pulumi | AWS CDK |
| --- | --- | --- |
| **트랜스파일** | 없음 — 프로그램이 직접 실행 | `cdk synth` → CloudFormation JSON/YAML(AWS Cloud Assembly)으로 합성 |
| **클라우드** | 모든 클라우드/SaaS | AWS 전용 |
| **리소스 제한** | 스택당 리소스 수 하드 리밋 없음 | CloudFormation의 500 리소스/템플릿 제한 상속(중첩 스택으로 회피) |
| **테스트** | 프로그램의 객체 그래프 자체를 단위 테스트 | 합성된 CloudFormation 템플릿에 대한 어설션 중심 |
| **모듈성** | Component Resources; 한 언어로 작성→모든 언어에서 소비 | Constructs(jsii 기반, 다중 언어 게시 필요); Construct Hub에서 공개 |
| **마이그레이션** | Pulumi CDK Adapter로 CDK Constructs를 Pulumi 프로그램에 직접 임베드; Pulumi Neo로 자동 변환 | — |

---

## Pulumi vs CDKTF

> https://www.pulumi.com/docs/iac/comparisons/cdktf/

> **참고:** CDKTF는 2025년 12월에 deprecated 되었으며, GitHub 리포지토리가 보관(archived)되었다.

### 개요

CDKTF(HashiCorp, 2020 출시, 2025년 12월 deprecated)는 TypeScript, Python, Go, C#, Java로 인프라를 정의한 뒤 Terraform JSON으로 합성하여 Terraform CLI로 배포했다. GitHub 리포지토리(https://github.com/hashicorp/terraform-cdk)는 보관(archived)되어 더 이상 업데이트를 받지 않는다. Pulumi는 트랜스파일 없이 직접 실행하며, 동일한 Terraform 프로바이더·모듈을 그대로 소비할 수 있다. Pulumi의 가장 인기 있는 프로바이더 다수는 CDKTF가 사용하던 것과 동일한 Terraform 프로바이더 스키마에서 생성되므로 리소스 모델이 일치하는 경우가 많아 마이그레이션이 보통 아키텍처 재설계가 아닌 코드 형태 변경 수준이다.

### 주요 차이점

| 차이점 | Pulumi | CDKTF |
| --- | --- | --- |
| **상태** | 활발히 유지보수 중 | 2025년 12월부터 deprecated 및 보관(archived). 버그 수정·보안 업데이트·신규 기능 없음 |
| **트랜스파일** | 없음 — 직접 실행 | `cdktf synth` → Terraform JSON → `terraform apply` |
| **프로바이더** | Pulumi Registry + 모든 Terraform 프로바이더 + 모든 Terraform 모듈 직접 소비 | Terraform 프로바이더만(`cdktf get`으로 주문형 SDK 생성) |
| **비밀** | 일급 암호화 프리미티브 + Pulumi ESC | 기본 없음 |
| **Automation API** | 있음 | 없음 |
| **정책** | Pulumi Policies(오픈 소스, Python/TypeScript/Rego) | 일급 없음. OPA 또는 Sentinel(상용)을 합성된 Terraform JSON에 적용 |
| **라이선스** | Apache 2.0 | CDKTF 프레임워크: MPL 2.0(보관됨); 배포용 Terraform CLI: BSL 1.1 |
| **마이그레이션** | `cdktf synth` → Terraform JSON → `pulumi convert --from terraform`; `pulumi-terraform-migrate`로 상태 변환; Pulumi Neo로 자동화(권장) | — |

---

## Pulumi vs Crossplane

> https://www.pulumi.com/docs/iac/comparisons/crossplane/

### 개요

Pulumi와 Crossplane은 모든 클라우드/SaaS에 리소스를 프로비저닝하지만, 아키텍처가 다르다. Pulumi는 CLI·Automation API·Pulumi Deployments로 실행된다. Crossplane은 Kubernetes 클러스터 내에서 컨트롤러가 YAML 매니페스트를 지속 조정(reconcile)하는 컨트롤 플레인 모델이다. Crossplane은 CNCF 졸업(Graduated) 프로젝트이며, Upbound에서 개발되었다. Crossplane v2는 2025년 8월 출시되었으며, 컴포지트·매니지드 리소스의 네임스페이스 기본화, 기존 "claim" 개념 제거, patch-and-transform 컴포지션 대신 composition functions 도입, 그리고 Crossplane 관리 인프라뿐 아니라 모든 Kubernetes 리소스를 컴포지션에 포함할 수 있도록 변경되었다.

### 주요 차이점

| 차이점 | Pulumi | Crossplane |
| --- | --- | --- |
| **실행 모델** | 온디맨드(CLI, Automation API, Deployments) | 상시 실행(Kubernetes 클러스터 내 컨트롤러, 지속 조정) |
| **클러스터 필요** | 불필요 | 필수(Kubernetes 클러스터가 운영 공간에 포함) |
| **드리프트** | `pulumi preview`/`pulumi refresh`로 감지(요청 시) | 컨트롤러가 지속 감지·자동 수정 |
| **비밀** | 일급 암호화 프리미티브 | Kubernetes Secrets(클러스터 설정에 의존) |
| **모듈성** | Component Resources + Pulumi Packages | Composite Resource Definitions(XRD) + Compositions + Composition Functions(v2에서 patch-and-transform 대신 composition functions가 기본) |
| **정책** | Pulumi Policies(오픈 소스) | 기본 없음; OPA Gatekeeper, Kyverno 등 Kubernetes 어드미션 컨트롤러 사용 |
| **프로바이더** | Pulumi Registry + 모든 Terraform 프로바이더 | Crossplane 프로바이더 — 클러스터 내에 패키지로 설치(installed into cluster); 공식 AWS, Azure, Google Cloud 프로바이더 제공; 다수의 커뮤니티 프로바이더는 Upjet 프레임워크로 Terraform 프로바이더에서 생성 |
| **공존 패턴** | Pulumi Kubernetes 프로바이더로 Crossplane 설치·관리 가능 | — |

### Pulumi vs Crossplane 선택 기준

**Pulumi를 선택해야 하는 경우:**

1. 범용 언어의 테스트 프레임워크, 패키지 매니저, IDE 도구를 활용해 인프라를 작성하고 싶을 때
2. 인프라 관리를 위해 Kubernetes 클러스터를 선행 요구사항으로 운영하고 싶지 않을 때
3. 호스트 애플리케이션에서 배포를 구동하는 임베딩 SDK(Automation API)가 필요할 때 — 내부 개발자 플랫폼, SaaS 제품, PR별 임시 환경 등
4. 플러그형 KMS 제공자와 스택별 암호화 키를 갖춘 일급 암호화 비밀이 필요할 때

**Crossplane을 선택해야 하는 경우:**

1. Kubernetes를 컨트롤 플레인으로 표준화하고 애플리케이션과 동일한 API, RBAC, GitOps 워크플로로 인프라를 관리하고 싶을 때
2. 누군가 명령을 실행하지 않아도 컨트롤러가 지속적으로 조정하고 드리프트를 자동 수정하기를 원할 때
3. YAML로 인프라를 선언적으로 정의하고 재사용 가능한 플랫폼 API를 Kubernetes 리소스로 소비하는 것을 선호할 때

두 도구는 공존할 수도 있다. 자세한 내용은 아래 채택 섹션을 참조하라.

### 채택: 공존, 변환, Import

Pulumi를 Crossplane과 함께 또는 그 대신 도입하는 일반적인 경로는 다음과 같으며, 조합하여 사용할 수 있다:

1. **Crossplane과 Pulumi 병행 사용** — Crossplane과 그 리소스는 모두 Kubernetes 객체이므로, Pulumi의 Kubernetes 프로바이더로 Crossplane을 설치하고 프로바이더를 설치하며 Crossplane 매니페스트를 적용할 수 있다. 예를 들어 `ConfigGroup`이나 `ConfigFile`로 기존 YAML을 적용하거나, `crd2pulumi`로 Crossplane CRD에서 타입 안전 SDK를 생성할 수 있다. 팀은 Pulumi로 클러스터를 관리하고 Crossplane을 부트스트랩하면서 Crossplane 관리 리소스는 그대로 유지할 수 있다.
2. **`pulumi convert`로 YAML 변환** — `pulumi convert --from kubernetes`는 Crossplane 리소스를 포함한 Kubernetes YAML 매니페스트를 원하는 언어의 Pulumi 프로그램으로 번역한다.
3. **기존 리소스 Import** — `pulumi import`와 `import` 리소스 옵션으로 이미 프로비저닝된 클라우드 리소스를 Pulumi 관리로 편입하고 선택한 언어로 코드를 생성할 수 있다.

### 자주 묻는 질문

**Crossplane은 Terraform과 같은가?**

다르다. 관련은 있지만 별개의 도구이다. 많은 Crossplane 프로바이더가 Upjet 프레임워크로 Terraform 프로바이더에서 생성되므로 유사한 클라우드 API를 커버하지만, 실행 모델이 다르다 — Terraform은 CLI를 통해 온디맨드로 실행되고, Crossplane은 Kubernetes 클러스터 내에서 컨트롤러가 지속 조정한다. Crossplane은 HCL과 Terraform CLI 대신 Kubernetes YAML과 Kubernetes API를 사용한다.

**Pulumi는 Kubernetes가 필요한가?**

필요하지 않다. Pulumi는 CLI나 Automation API를 통해 실행되며 인프라 관리를 위해 Kubernetes 클러스터가 필요하지 않다. 반면 Crossplane은 Kubernetes 클러스터 내에서 컨트롤러를 실행하므로 클러스터가 운영 공간의 일부이다. Pulumi는 Kubernetes 리소스를 관리하기 위한 일급 Kubernetes 프로바이더를 제공하지만 — Crossplane 자체 포함 — 선택 사항이다.

**Pulumi에서 Crossplane 리소스를 배포하고 관리할 수 있는가?**

가능하다. Crossplane, 프로바이더, 그리고 Crossplane이 정의하는 커스텀 리소스는 모두 Kubernetes 객체이므로 Pulumi의 Kubernetes 프로바이더로 Crossplane을 설치하고 Crossplane 매니페스트를 적용할 수 있다. `ConfigGroup`이나 `ConfigFile`로 기존 YAML을 적용하거나, `crd2pulumi`로 Crossplane CRD에서 타입 안전 SDK를 생성할 수 있다.

**Crossplane에서 Pulumi로 마이그레이션하려면?**

조합할 수 있는 옵션이 있다: `pulumi convert --from kubernetes`로 Crossplane YAML 매니페스트 변환, `pulumi import`로 이미 프로비저닝된 클라우드 리소스를 Pulumi 관리로 편입, 또는 두 도구를 병행 운영하며 점진적으로 마이그레이션. 자세한 안내는 Kubernetes에서 Pulumi로 마이그레이션 가이드를 참조하라.

**Pulumi는 Crossplane처럼 무료인가?**

Pulumi CLI와 SDK는 Apache 2.0 라이선스의 오픈 소스로 무료다. Pulumi Cloud는 무료 Individual 티어와 조직 규모 실행을 위한 유료 플랜(관리형 상태, RBAC, 감사 로그, 정책 관리 등)을 제공한다. Crossplane 자체는 Apache 2.0으로 무료이며, 관리형 컨트롤 플레인과 엔터프라이즈 배포판은 Upbound에서 별도 판매한다.

---

## Pulumi vs Helm

> https://www.pulumi.com/docs/iac/comparisons/helm/

### 개요

Pulumi와 Helm은 서로 다른 문제를 해결하지만 같은 워크플로에서 자주 함께 사용된다. Pulumi는 모든 클라우드/SaaS 리소스를 프로비저닝하는 IaC 플랫폼이다. Helm은 Kubernetes 애플리케이션을 템플릿 기반 YAML 차트로 설치하는 패키지 매니저이다.

### 주요 차이점

| 차이점 | Pulumi | Helm |
| --- | --- | --- |
| **목적** | 모든 클라우드/SaaS 인프라 | Kubernetes 애플리케이션 패키지 매니저 |
| **범위** | AWS, Azure, GCP, Kubernetes, SaaS(Datadog, Auth0, GitHub, Cloudflare 등) | Kubernetes 리소스만 |
| **언어** | 범용 언어 6종 + YAML(타입 안전한 입출력) | YAML + Go 템플릿(텍스트 템플릿) |
| **비밀** | 일급 암호화 프리미티브 | 기본 없음; `helm-secrets` 플러그인 또는 외부 스토어 |
| **롤백** | 부분 업데이트 후 수렴 | `helm rollback`으로 이전 리비전 복원 |
| **SDK** | 모든 지원 언어에서 Automation API | Go 전용 Helm SDK; 다른 언어는 CLI 셸아웃 |

### Pulumi에서 Helm 사용하기

Pulumi와 Helm은 대체재보다 보완재 관계가 일반적이다. Pulumi Kubernetes 프로바이더는 두 가지 Helm 리소스를 제공한다.

| 리소스 | 설명 |
| --- | --- |
| `helm.v4.Chart` | 차트 템플릿을 로컬에서 렌더하고, 각 Kubernetes 객체를 개별 Pulumi 리소스로 등록. 플랜 출력·드리프트 감지·상태에서 모든 자식 리소스가 보임 |
| `helm.v3.Release` | 내장 Helm SDK로 실제 Helm Release를 클러스터에 생성. Chart Hooks, Post-rendering, Release History 등 Helm 전용 동작 보존 |

---

## Pulumi vs Serverless Framework

> https://www.pulumi.com/docs/iac/comparisons/serverless/

### 개요

Pulumi는 모든 클라우드/SaaS 리소스를 프로비저닝하는 범용 IaC 플랫폼이다. Serverless Framework는 `serverless.yml`로 AWS Lambda 함수와 관련 리소스를 배포하는 AWS 중심 도구이다. V4부터 비-공식 소스 라이선스로 전환되었고, AWS 외 프로바이더 지원이 deprecated 되었다.

### 주요 차이점

| 차이점 | Pulumi | Serverless Framework |
| --- | --- | --- |
| **범위** | 범용 IaC(서버리스·비서버리스 모두) | 서버리스 애플리케이션(함수 중심 모델) |
| **클라우드** | 모든 클라우드/SaaS | AWS 전용(V4); 비-AWS는 deprecated |
| **언어** | 범용 언어 6종 + YAML | `serverless.yml`(YAML) |
| **트랜스파일** | 없음 | `serverless.yml` → CloudFormation 템플릿 |
| **비함수 인프라** | 데이터베이스, VPC, DNS, Kubernetes 등 모두 일급 리소스 | `resources` 블록에 원시 CloudFormation으로 표현(프레임워크 추상화 이탈) |
| **라이선스** | Apache 2.0 | V3까지 MIT; V4는 독점 라이선스 |

---

## Pulumi vs OpenTofu

> https://www.pulumi.com/docs/iac/comparisons/opentofu/

### 개요

OpenTofu는 Terraform 1.6에서 포크된 Linux Foundation 관리의 오픈 소스 IaC 도구이다. HCL을 사용하며 Terraform과 동일한 프로바이더 생태계를 공유한다. Pulumi와 OpenTofu는 모든 클라우드/SaaS를 지원하지만, 언어·실행·비밀 처리에서 차이가 있다.

### 주요 차이점

| 차이점 | Pulumi | OpenTofu |
| --- | --- | --- |
| **언어** | 범용 언어 6종 + YAML | HCL(고정 함수 집합, `for_each`, `count`, `dynamic` 메타 인수) |
| **프로바이더** | Pulumi Registry + Any Terraform Provider 기능으로 OpenTofu/Terraform 레지스트리의 모든 프로바이더에서 타입 안전 SDK 생성(`pulumi package add terraform-provider <name>` 실행으로 로컬 SDK 생성) | OpenTofu Registry + Terraform Registry; `required_providers` 블록으로 설치·고정 |
| **비밀** | 일급 프리미티브(전송 중·저장 시 암호화, 스택별 키) | 상태·플랜 파일 암호화(1.7+, 전체 파일 단위); 개별 민감 변수는 일급 프리미티브 아님 |
| **Automation API** | 있음 | 없음(`tofu` CLI 호출로만 오케스트레이션) |
| **상용** | Pulumi Cloud(단일 벤더) | 프로젝트 자체는 상용 없음; 서드파티(Spacelift, env0, Scalr)에서 관리형 서비스 제공 |
| **라이선스** | Apache 2.0 | MPL 2.0 |
| **모듈 소비** | OpenTofu 모듈을 Pulumi에서 직접 실행(OpenTofu 자동 설치·호출) | — |
| **마이그레이션** | `pulumi convert --from terraform`(OpenTofu HCL 동일하게 처리, 별도 `--from opentofu` 불필요); Pulumi에서 OpenTofu 상태 참조 가능 | — |

### Pulumi vs OpenTofu 선택 기준

**Pulumi를 선택해야 하는 경우:**

1. 범용 언어의 테스트 프레임워크, 패키지 매니저, IDE 도구를 활용해 인프라를 작성하고 싶을 때
2. 호스트 애플리케이션에서 배포를 구동하는 임베딩 SDK(Automation API)가 필요할 때 — 내부 개발자 플랫폼, SaaS 제품, PR별 임시 환경 등
3. 플러그형 KMS 제공자와 스택별 암호화 키를 갖춘 일급 암호화 비밀이 필요할 때
4. 상태, RBAC, 감사 로그, 정책 관리, 원격 실행을 모두 포함하는 단일 관리 제품(Pulumi Cloud)을 원할 때

**OpenTofu를 선택해야 하는 경우:**

1. 프로젝트 자체의 상용 제품이 없는, 완전한 오픈 소스·벤더 중립 IaC 도구를 원할 때
2. HCL 구성, Terraform/OpenTofu 모듈, 팀 전문 지식에 대한 대규모 기존 투자를 마이그레이션하고 싶지 않을 때
3. 단일 벤더에서 구매하는 대신 서드파티 서비스(Spacelift, env0, Scalr)에서 관리형 상태·협업 기능을 조립하는 것을 선호할 때

두 도구는 공존할 수도 있다. 자세한 내용은 아래 채택 섹션을 참조하라.

### 채택: 공존, 변환, Import

Pulumi를 OpenTofu와 함께 또는 그 대신 도입하는 일반적인 경로는 다음과 같으며, 조합하여 사용할 수 있다:

1. **OpenTofu와 Pulumi 병행 사용** — Pulumi 프로그램에서 기존 OpenTofu 상태 파일을 참조하여 출력값을 읽을 수 있다(`terraform.state.getLocalReference`, `terraform.state.getRemoteReference`). Pulumi는 기존 OpenTofu 모듈도 직접 실행할 수 있으며, 모듈을 실행하기 위해 OpenTofu를 자동 설치·호출한다.
2. **`pulumi convert`로 HCL 변환** — `pulumi convert --from terraform`은 HCL을 원하는 언어의 Pulumi 프로그램으로 번역한다. 동일한 플래그가 Terraform과 OpenTofu HCL 모두를 처리하며, 구성 언어가 동일하므로 별도의 `--from opentofu` 플래그는 없다.
3. **기존 리소스 Import** — `pulumi import`와 `import` 리소스 옵션으로 이미 프로비저닝된 클라우드 리소스를 Pulumi 관리로 편입하고 선택한 언어로 코드를 생성할 수 있다.

### 자주 묻는 질문

**OpenTofu는 Terraform과 같은가?**

OpenTofu는 HashiCorp가 Terraform의 라이선스를 MPL 2.0에서 Business Source License로 변경한 후 2023년에 Terraform 1.6에서 포크되었다. 두 프로젝트는 HCL 구성 언어와 대부분 겹치는 프로바이더 생태계를 공유하지만, 이후 분기되었다. OpenTofu는 Linux Foundation이 관리하며 Terraform에는 없는 상태 암호화 등의 기능을 추가했고, Terraform도 자체적인 새 기능을 계속 출시하고 있다. 대부분의 기존 구성에서 OpenTofu는 즉시 사용 가능한 대체제이지만, 두 프로젝트는 동일하지 않다.

**Pulumi에서 기존 OpenTofu 프로바이더와 모듈을 사용할 수 있는가?**

가능하다. Any Terraform Provider 기능으로 OpenTofu 또는 Terraform 레지스트리의 모든 프로바이더에서 `pulumi package add terraform-provider <name>` 실행만으로 타입 안전 Pulumi SDK를 생성할 수 있다. Pulumi는 기존 OpenTofu 모듈도 Pulumi 프로그램 내 컴포넌트로 직접 실행할 수 있다 — Pulumi가 OpenTofu를 자동 설치하고 모듈 실행을 위해 호출한다.

**OpenTofu에서 Pulumi로 마이그레이션하려면?**

세 가지 옵션을 조합할 수 있다: `pulumi convert --from terraform`로 HCL 변환(OpenTofu HCL도 처리하며 별도 `--from opentofu` 플래그 불필요), `pulumi import`로 이미 프로비저닝된 리소스를 Pulumi 관리로 편입, 또는 전환할 준비가 될 때까지 두 도구를 병행 운영. 전체 안내는 마이그레이션 가이드를 참조하라.

**Pulumi는 OpenTofu 상태 파일을 지원하는가?**

지원한다. Pulumi 프로그램은 `terraform.state.getLocalReference`(로컬 상태) 및 `terraform.state.getRemoteReference`(원격 백엔드)를 통해 OpenTofu 상태 파일의 출력값을 읽을 수 있다. 두 함수 모두 `terraform` 패키지에 있으며, 두 도구가 상태 파일 형식을 공유하므로 OpenTofu 상태에서도 작동한다.

**Pulumi는 OpenTofu처럼 무료인가?**

Pulumi CLI와 SDK는 Apache 2.0 라이선스의 오픈 소스로 무료다. Pulumi Cloud는 무료 Individual 티어와 조직 규모 실행을 위한 유료 플랜(관리형 상태, RBAC, 감사 로그, 정책 관리 등)을 제공한다. OpenTofu 자체는 MPL 2.0으로 무료이며, 상용 관리형 상태·협업 도구는 Spacelift, env0, Scalr 등 서드파티에서 별도 판매한다.

**마이그레이션 중 Pulumi와 OpenTofu를 병행 실행할 수 있는가?**

가능하며, 가장 일반적인 도입 패턴 중 하나이다. Pulumi는 OpenTofu 상태 파일에서 출력값을 읽고 OpenTofu 모듈을 직접 실행할 수 있으므로, 팀은 일반적으로 기존 인프라를 OpenTofu로 유지하면서 새 작업에 Pulumi를 사용한 후 프로젝트 상황에 맞게 점진적으로 변환하거나 Import한다.

---

## Pulumi vs ARM Templates / Bicep

> https://www.pulumi.com/docs/iac/comparisons/arm-templates/

### 개요

Pulumi와 ARM Templates(Azure Resource Manager)은 모두 Azure 인프라를 위한 선언적 IaC 도구이다. Pulumi는 범용 언어로 모든 클라우드/SaaS를 지원한다. ARM Templates은 JSON으로, Bicep은 더 깔끔한 DSL 문법으로 Azure 리소스만 프로비저닝하며 Azure 관리 서비스에서 실행된다. Bicep은 Microsoft가 만든 DSL로 ARM JSON으로 컴파일되며, MIT 라이선스의 오픈 소스 CLI를 갖추고 있지만 배포 엔진(ARM 서비스)은 폐쇄 소스이다.

### 주요 차이점

| 차이점 | Pulumi | ARM Templates / Bicep |
| --- | --- | --- |
| **언어** | 범용 언어 6종 + YAML | JSON 템플릿 또는 Bicep(DSL, ARM JSON으로 컴파일). template functions로 제한적 동적 로직 제공 |
| **클라우드** | 모든 클라우드/SaaS | Azure 전용; 서드파티는 Custom Resource Provider 또는 Deployment Script로 제한 |
| **Azure 리소스 커버리지** | Azure Native 프로바이더: ARM REST API 스펙에서 직접 생성 → 신규 리소스 당일 지원(ARM Templates과 동일) | ARM이 Azure 리소스의 원천 |
| **실행** | 로컬 CLI, Automation API, Pulumi Deployments | Azure 관리 서비스(Portal, Azure CLI, PowerShell, Bicep CLI, REST API) |
| **롤백** | 부분 업데이트 후 수렴 | 오류 시 마지막 성공 배포로 롤백(옵션, 배포 시 활성화 필요) |
| **비밀** | 일급 암호화 프리미티브 | 기본 없음; `secureString`/`secureObject` 파라미터 또는 Key Vault 참조 |
| **모듈성** | Component Resources + Pulumi Packages(언어 간 소비 가능) | Linked Templates, Bicep modules, template specs, Bicep 공개 모듈 레지스트리 |
| **Import** | `pulumi import` + `import` 리소스 옵션(코드 자동 생성) | 일급 Import 없음; Bicep `existing` 키워드로 읽기 전용 참조만 |
| **정책** | Pulumi Policies(오픈 소스, Python/TypeScript/Rego) | Azure Policy(별도 Azure 서비스, 템플릿 저작 루프와 분리) |
| **마이그레이션** | `pulumi convert --from arm` / `--from bicep`; Pulumi Neo(AI 보조); `pulumi import` | — |

---

## Pulumi vs Cloud SDKs (AWS Boto 등)

> https://www.pulumi.com/docs/iac/comparisons/cloud-sdks/

### 개요

클라우드 프로바이더는 다양한 언어로 SDK를 제공한다. 대표적으로 AWS Boto(Python)가 있으며, JavaScript, C#, Java, Go 등을 위한 수많은 라이브러리도 존재한다. 이 밖에도 클라우드별 또는 멀티클라우드 시나리오를 추상화하는 커뮤니티 주도 라이브러리도 많다. 이러한 SDK는 클라우드 API에 대한 직접 접근을 제공하며 리소스 프로비저닝에도 사용할 수 있지만, **안정적인 관리를 프로그래머에게 전적으로 맡긴다**. 이들은 명령형(imperative) 라이브러리이므로, 이를 사용해 프로비저닝과 업데이트를 직접 코드로 작성하는 것은 오류가 발생하기 쉬우며, 결국 [자체 제작 프로비저닝 시스템](https://www.pulumi.com/docs/concepts/vs/custom/)을 구축하는 결과로 이어지는 경우가 많다.

Pulumi는 진정한 Infrastructure as Code 엔진으로 리소스의 견고한 관리를 보장한다. 특히 업데이트가 실패하더라도 시스템은 항상 명확하게 정의된 복구 가능한 상태를 유지한다. 사실 Pulumi는 내부적으로 이러한 클라우드 SDK를 사용하여 각 클라우드 프로바이더와 통신한다. Pulumi를 다른 것은 언어 런타임, 업데이트 diff 기능, 동시성 관리, 상태의 견고한 체크포인트 등 IaC 엔진으로서의 기능이다. 반복 가능하고 견고한 배포를 팀 환경에서 구현하려면 이러한 기능을 직접 구축해야 한다. 멀티클라우드 지원은 상당한 노력이 필요하며, 결국 Pulumi를 수동으로 다시 만드는 것과 같다.

### 주요 차이점

| 차이점 | Pulumi | Cloud SDKs |
| --- | --- | --- |
| **패러다임** | 선언적 IaC — 원하는 상태를 정의하면 엔진이 diff·생성·업데이트·삭제를 처리 | 명령형(imperative) — API 호출을 직접 코드로 작성, 프로비저닝 로직 전부 수동 관리 |
| **상태 관리** | Pulumi Cloud(기본) 또는 자체 관리 백엔드. 체크포인트 기반 상태 추적 | 없음. 프로비저닝 상태 추적·체크포인트 기능 없음(직접 구현 필요) |
| **diff·플랜** | `pulumi preview`로 변경 사항을 실행 전에 미리 확인 | 없음. 어떤 변경이 발생할지 직접 추적해야 함 |
| **실패 복구** | 실패 후에도 명확한 상태 유지; 이후 `pulumi up`으로 수렴 | 실패 시 복구 로직을 직접 작성해야 함(부분 생성 리소스 수동 정리 필요) |
| **멀티클라우드** | 단일 프로그램에서 AWS, Azure, GCP, Kubernetes, SaaS 모두 관리 | 각 클라우드 SDK를 개별적으로 사용; 멀티클라우드 조율은 전적으로 수동 |
| **비밀 처리** | 일급 암호화 프리미티브 | 없음. 비밀 처리를 직접 구현(클라우드 비밀 관리 서비스 별도 호출 가능) |
| **동시성 관리** | Pulumi 엔진이 종속성 그래프 기반으로 자동 처리 | 직접 구현(API 호출 순서·의존성·재시도 로직) |
| **재사용·공유** | Component Resources + Pulumi Packages로 언어 간 공유 | 범용 언어의 패키지 매니저로 코드 공유는 가능하나 IaC 추상화는 직접 설계 |

---

## Pulumi vs Kubernetes YAML Manifests

> https://www.pulumi.com/docs/iac/comparisons/k8s-yaml-dsls/

### 개요

Pulumi와 Kubernetes YAML 매니페스트는 모두 인프라의 원하는 상태를 선언적으로 정의하는 방식이다. Pulumi는 범용 언어로 모든 클라우드/SaaS 리소스를 정의하며, Kubernetes YAML 매니페스트는 Kubernetes API의 네이티브 설정 형식으로 Kubernetes 객체만 기술한다.

### 주요 차이점

| 차이점 | Pulumi | Kubernetes YAML Manifests |
| --- | --- | --- |
| **언어** | 범용 언어 6종 + YAML. 루프, 조건문, 클래스, 패키지 관리, IDE 기능 활용 가능 | YAML 또는 JSON. 루프·조건문·변수 없음. Kustomize로 베이스·오버레이 레이어링 가능하지만 여전히 선언적 YAML |
| **클라우드** | 모든 클라우드/SaaS | Kubernetes API 객체만. 비-K8s 클라우드 리소스는 별도 도구 또는 AWS Controllers for Kubernetes, Crossplane 등 인-클러스터 오퍼레이터 필요 |
| **상태** | Pulumi Cloud(기본) 또는 자체 관리 백엔드 | 별도 상태 파일 없음. 라이브 클러스터(etcd)가 진실의 원천이며 `kubectl apply`가 server-side apply로 관리 필드 추적 |
| **비밀** | 일급 암호화 프리미티브 | Kubernetes Secret 객체는 base64 인코딩만(미암호화). etcd 암호화는 클러스터 수준에서 별도 구성, Sealed Secrets 등 외부 도구 필요 |
| **롤백** | 부분 업데이트 후 수렴 | `kubectl apply`는 트랜잭션 롤백 없음. 워크로드 컨트롤러는 `kubectl rollout undo`로 리비전 롤백 지원 |
| **재사용** | Component Resources + Pulumi Packages(언어 간 소비 가능) | YAML 복사 또는 Kustomize 베이스/오버레이. 패키지 매니저·타입 인터페이스 없음 |
| **프로그래밍 API** | Automation API | Kubernetes API·클라이언트 라이브러리로 개별 객체 조작은 가능하나 apply/preview/destroy 수명주기 오케스트레이션 SDK는 없음 |
| **정책** | Pulumi Policies(오픈 소스, Python/TypeScript/Rego) | 기본 없음. OPA Gatekeeper, Kyverno 등 어드미션 컨트롤러로 클러스터 내 시행 |

### 마이그레이션 경로

1. **기존 YAML 그대로 소비** — Kubernetes 프로바이더의 `ConfigFile`, `ConfigGroup` 리소스로 기존 매니페스트를 변경 없이 배포
2. **렌더링으로 공존** — Pulumi 프로그램을 Kubernetes YAML로 렌더링하여 기존 `kubectl` 또는 GitOps 파이프라인으로 배포
3. **매니페스트 변환** — `pulumi convert --from kubernetes`로 기존 매니페스트를 원하는 언어의 Pulumi 코드로 변환
4. **기존 리소스 Import** — `pulumi import`로 클러스터 내 이미 실행 중인 리소스를 Pulumi 관리로 편입

---

## Pulumi vs Chef & Puppet

> https://www.pulumi.com/docs/iac/comparisons/chef-puppet-etc/

### 개요

Chef, Puppet, Ansible, Salt는 모두 인기 있는 *구성 관리 도구(configuration management tools)*이다. 이 도구들은 기존 클라우드 인프라에 소프트웨어를 설치하고 관리하는 데 도움을 주며, 가상 머신 부트스트래핑이나 패치에 사용된다. 하지만 인프라, 컨테이너, 서버리스 리소스의 프로비저닝이나 업데이트 문제는 해결하지 않는다.

Pulumi는 이 도구들과 근본적으로 다르며, 함께 사용하면 훌륭하게 작동한다. Pulumi는 가상 머신, 컨테이너, 데이터베이스, 호스팅된 클라우드 서비스, 서버리스 함수를 포함하여 클라우드 인프라와 애플리케이션의 프로비저닝과 업데이트를 관리한다.

Pulumi는 일부 구성 관리 도구와 마찬가지로 표현력 있는 범용 언어를 사용하여 클라우드 요구 사항과 원하는 인프라 상태를 정의한다. 그런 다음 Pulumi 도구가 이러한 설명을 받아 여러 환경을 관리하여 CLI나 호스팅된 CI/CD 시나리오를 통해 인프라가 항상 최신 상태로 유지되도록 한다.

Pulumi는 부트스트래핑과 패치가 불필요한 최신 "불변 인프라(immutable infrastructure)" 아키텍처에서 잘 작동한다. 이러한 경우 구성 관리는 일반적인 의미로 필요하지 않다. 그러나 Pulumi는 종종 가상 머신을 수반하는 클래식 인프라 접근 방식에서도 잘 작동하며, 구성 도구를 계속 함께 사용하는 것이 쉽다. 어느 경우든 Pulumi는 클라우드 네이티브 아키텍처로의 전환을 지원한다.

### 핵심 차이

구성 관리 도구(Chef, Puppet, Ansible, Salt)와 Pulumi는 서로 다른 문제를 해결한다:

| 구분 | Pulumi | 구성 관리 도구 |
| --- | --- | --- |
| **목적** | 클라우드 인프라 프로비저닝 및 관리 | 기존 서버의 소프트웨어 설치 및 구성 관리 |
| **접근 방식** | 선언적 IaC — 원하는 상태를 정의하면 엔진이 처리 | 명령형/선언형 혼합 — 주로 구성 관리에 초점 |
| **리소스 유형** | 가상 머신, 컨테이너, 데이터베이스, 서버리스, 클라우드 서비스 등 모든 클라우드 리소스 | 기존 인프라 위의 소프트웨어 패키지, 파일, 서비스 |
| **클라우드 범위** | 모든 클라우드/SaaS | 클라우드에 구애받지 않으나 프로비저닝이 아닌 구성에 초점 |
| **관계** | 보완재 — Pulumi가 인프라를 프로비저닝하고 구성 관리 도구가 소프트웨어를 구성 | 보완재 — 구성 관리 도구로 소프트웨어를 구성하고 Pulumi로 인프라를 프로비저닝 |

---

## Pulumi vs Custom Solutions

> https://www.pulumi.com/docs/iac/comparisons/custom/

### 개요

많은 조직이 수동으로 인프라를 관리하는 것으로 시작한다. 일반적으로 클라우드 웹 콘솔에서 포인트앤드클릭으로 시작하여, 점차 프로비저닝과 로직을 간단한 스크립트(Bash, Python 등)로 인코딩하는 방식으로 발전한다.

이 접근 방식은 초기에는 작동하지만, 시간이 지남에 따라 자체 구축한 솔루션은 다음과 같은 문제로 점점 무너진다:

- 애플리케이션의 여러 복사본을 프로비저닝하는 것이 어렵거나 불가능해진다
- 스크립트의 로직이 인프라 복잡성에 비례하여 비선형적으로 증가한다
- 장애 모드에서 환경 복구를 위해 수동 개입이 필요하며, 종종 다운타임으로 이어진다
- 클라우드 프로바이더가 진화함에 따라 최신 혁신을 따라잡고 채택하기가 어려워진다

이러한 솔루션의 유지 보수 비용은 종종 여러 팀 구성원을 소모하며, 실제로 최고의 엔지니어들이 묶이게 된다. 많은 팀이 자체 구축한 인프라 스크립트를 관리하는 데 비즈니스 로직을 작성하는 것보다 더 많은 시간을 소비하면서도 원하는 속도로 움직이지 못한다는 것을 발견한다.

### Pulumi가 해결하는 문제

Pulumi는 이러한 문제를 즉시 해결하도록 설계되었다. 친숙한 언어로 클라우드 프로그래밍 모델을 제공하여 클라우드 인프라와 애플리케이션 요구 사항을 표현할 수 있다. Pulumi로 표현하면 팀 환경에서 작동하는 견고하고 반복 가능한 배포 모델을 얻을 수 있다. Pulumi는 기존 시스템의 새 리전, 새 환경(스테이징, 프로덕션, 테스트), 테스트 용이성 등 인프라 또는 애플리케이션의 여러 복사본 프로비저닝을 간단하게 만든다.

또한 Pulumi는 [오픈 소스](https://github.com/pulumi/pulumi)이며 커뮤니티 주도이다. 따라서 맞춤 요구 사항이 있는 경우 자체 구축 솔루션과 마찬가지로 필요한 맞춤 설정을 할 수 있다. 대부분의 고객에게 Pulumi는 맞춤 설정 없이 기존 맞춤 솔루션을 대체하지만, 이것이 옵션이라는 점을 알면 Pulumi가 장기적으로 사용 가능하고 성장하는 요구를 충족하기 위해 진화할 수 있다는 것을 알 수 있다.

---

## Pulumi의 장단점 요약

### 장점

| 장점 | 설명 |
| --- | --- |
| **범용 언어** | Python, TypeScript, Go, C#, Java, YAML로 작성. 기존 IDE, 테스트 프레임워크, 패키지 매니저 생태계 그대로 활용 |
| **모든 클라우드/SaaS** | Pulumi Registry를 통해 AWS, Azure, GCP, Kubernetes, 150개 이상의 클라우드·SaaS 프로바이더 지원 |
| **네이티브 프로바이더** | Kubernetes, Azure Native, AWS Cloud Control, Google Cloud Native은 업스트림 API 스키마에서 자동 생성 → 신규 기능 당일 지원 |
| **Terraform 생태계 호환** | 모든 Terraform 프로바이더를 Pulumi 프로바이더로 변환; Terraform 모듈을 직접 소비; HCL을 `pulumi convert`로 변환 |
| **일급 비밀 처리** | 전송 중·저장 시 암호화, 스택별 암호화 키, 비밀에서 파생된 값도 자동 암호화, 플러그형 KMS |
| **Automation API** | CLI 없이 프로그래밍 방식으로 배포를 호스트 애플리케이션에 내장. IDP, SaaS, PR별 임시 환경 등에 활용 |
| **언어 간 패키지 공유** | Component Resources를 한 언어로 작성하여 Pulumi Package로 게시하면 모든 지원 언어에서 소비 가능 |
| **오픈 소스 정책** | Policy as Code가 오픈 소스(Apache 2.0). 상용 플랜에 CIS, HITRUST, NIST, PCI DSS 준법 팩 포함 |
| **Apache 2.0 라이선스** | CLI, SDK가 완전한 오픈 소스. Terraform(BSL 1.1)과 달리 사용 제약 없음 |

### 단점 및 고려사항

| 단점 | 설명 |
| --- | --- |
| **자동 롤백 없음** | 실패 시 부분 업데이트 상태 유지. 수동으로 코드 수정 후 `pulumi up` 재실행 필요. CloudFormation/ARM의 자동 롤백과 대비됨 |
| **학습 곡선** | 범용 언어를 사용하므로 언어 자체의 학습이 필요할 수 있음. 단순 구성에는 HCL이나 YAML이 더 간단할 수 있음 |
| **Pulumi Cloud 의존도** | 관리형 상태, RBAC, 감사 로그, 정책 관리 등 엔터프라이즈 기능은 Pulumi Cloud(상용) 필요. 자체 관리도 가능하지만 운영 부담 |
| **상대적으로 작은 커뮤니티** | Terraform에 비해 커뮤니티 규모가 작고, 레퍼런스·예제·서드파티 도구가 적음 |
| **범용 언어의 복잡성** | 언어의 자유도가 높은 만큼, 팀 내 코딩 컨벤션·아키텍처 가이드 없이는 코드 베이스가 불균일해질 위험 |

---

## Pulumi YAML 지원

> https://www.pulumi.com/docs/iac/languages-sdks/yaml/

> **참고:** Pulumi YAML은 독립된 비교 문서가 존재하지 않으며, YAML은 Pulumi가 지원하는 언어 중 하나이다.

Pulumi는 Python, TypeScript, JavaScript, Go, C#, Java 외에도 YAML을 지원한다. YAML은 마크업 포맷을 선호하는 사용자를 위한 옵션으로, HCL, CloudFormation, Bicep, Kustomize 등의 선언적 YAML 형식에 익숙한 사용자에게 적합하다. Pulumi YAML은 타입 안전한 입출력을 제공하며, 다른 Pulumi 언어로 작성된 Component Resources를 Pulumi Package로 게시하면 YAML에서도 소비할 수 있다.
