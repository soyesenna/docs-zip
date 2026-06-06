# Pulumi 개요

> **원문**
> - https://www.pulumi.com/docs/
> - https://www.pulumi.com/docs/iac/
> - https://www.pulumi.com/docs/iac/concepts/
> - https://www.pulumi.com/docs/iac/guides/basics/how-pulumi-works/
> - https://www.pulumi.com/docs/iac/concepts/pulumi-cloud/
> - https://www.pulumi.com/docs/iac/guides/basics/pulumi-cloud-vs-oss/
> - https://www.pulumi.com/docs/iac/concepts/state-and-backends/

Pulumi는 TypeScript, Python, Go, .NET, Java, YAML 같은 범용 프로그래밍 언어와 마크업 언어를 사용하여 클라우드 인프라를 정의·배포·관리하는 오픈 소스 Infrastructure as Code(IaC) 플랫폼이다. CLI, 런타임, SDK, 리소스 프로바이더로 구성되며, 선택적으로 Pulumi Cloud(관리형 서비스)와 연동하여 팀 협업·접근 제어·정책 시행·드리프트 감지 등을 제공한다.

---

## 핵심 특징

| 특징 | 설명 |
| --- | --- |
| **다중 언어 지원** | TypeScript, JavaScript(Node.js), Python, Go, C#/VB/F#(.NET), Java, Pulumi YAML 지원. 모든 언어가 모든 주요 클라우드의 리소스를 동등하게 프로비저닝 가능 |
| **클라우드 독립적** | AWS, Azure, Google Cloud, Kubernetes 등 150개 이상의 클라우드 프로바이더 및 서비스 지원 ([Registry](https://www.pulumi.com/registry/)에서 검색) |
| **오픈 소스** | [GitHub](https://github.com/pulumi/pulumi)에서 호스팅되는 오픈 소스 CLI, 배포 엔진, 언어 SDK, 리소스 프로바이더 포함 |
| **상태 관리** | Pulumi Cloud(관리형) 또는 DIY 백엔드(AWS S3, Azure Blob, GCS, PostgreSQL, 로컬 파일시스템) 중 선택 |
| **선언적 desired-state 모델** | 프로그램이 원하는 상태(desired state)를 선언하면, 배포 엔진이 마지막 배포 상태와 비교하여 최소 변경셋을 계산 |
| **자동 의존성 해석** | 리소스 간 속성 참조를 통해 의존성을 자동으로 파악하고, 독립적인 리소스는 병렬로 생성 |

---

## 지원 언어 및 런타임

| 언어 | 런타임 | 진입점 규칙 |
| --- | --- | --- |
| TypeScript / JavaScript | Node.js | `package.json` → `index.ts` 등 |
| Python | Python | `__main__.py` 또는 `setup.py` |
| Go | Go | Go 빌드 도구 규칙 (`main.go`) |
| C# / VB / F# | .NET | .NET 빌드 도구 규칙 |
| Java | JVM | Maven/Gradle 빌드 산출물 |
| Pulumi YAML | YAML | `Pulumi.yaml` 내에 리소스를 인라인 정의 |

> 모든 Pulumi 지원 언어는 모든 주요 클라우드의 리소스를 프로비저닝하고 관리하는 데 동등한 능력을 갖는다. 일부 기능이 특정 언어에 아직 구현되지 않았을 수 있다.

---

## 아키텍처 개요

Pulumi 플랫폼은 다음 핵심 컴포넌트로 구성된다.

### 주요 컴포넌트

| 컴포넌트 | 역할 |
| --- | --- |
| **SDK (Software Development Kit)** | 각 리소스 타입에 대한 바인딩을 제공. 클라우드 및 프로바이더의 리소스를 정의·관리하는 데 필요한 도구와 라이브러리 포함 |
| **CLI (Command-Line Interface)** | Pulumi를 제어하는 기본 인터페이스. `pulumi up`, `pulumi preview`, `pulumi destroy` 등의 명령 제공. 로컬 개발 내부 루프(inner loop)와 CI/CD 시나리오 모두에 최적화 |
| **Deployment Engine (배포 엔진)** | 현재 인프라 상태를 프로그램이 선언한 desired state로 변환하기 위해 필요한 작업 집합을 계산. 리소스 간 의존성을 이해하여 병렬성을 극대화하고 올바른 순서를 보장 |
| **Language Host (언어 호스트)** | 프로그램을 실행하는 언어별 런타임(Node.js, Python, Go, .NET, JVM, YAML). 리소스 등록 요청을 배포 엔진으로 전송 |
| **Resource Provider (리소스 프로바이더)** | 특정 클라우드(AWS, Azure, GCP 등)의 리소스를 실제로 CRUD하는 플러그인. 엔진이 직접 클라우드와 통신하지 않고 프로바이더에 위임. 새 리소스 타입 추가 시 CLI 업데이트 없이 프로바이더 버전만 업데이트하면 됨 |

### 실행 흐름

1. `pulumi up` 실행 시 CLI가 Language Host를 시작
2. Language Host가 프로그램을 실행하며 리소스 등록 요청을 배포 엔진에 전송
3. 엔진은 마지막 배포 상태와 비교하여 필요한 작업(Create/Update/Delete)을 결정
4. 각 작업을 해당 Resource Provider에 위임하여 실제 클라우드 리소스 조작
5. 작업 완료 후 새 상태를 checkpoint로 백엔드에 기록
6. 독립적인 리소스는 병렬로 처리하여 성능 극대화

```
+-------------+     리소스 등록      +-------------------+
| Language    |  ------------------> |  Deployment       |
| Host        |                      |  Engine           |
| (Node/Py/   |  <------------------ |                   |
|  Go/.NET/   |    작업 결과         +-------------------+
|  JVM/YAML)  |                           |
+-------------+                           |
                                    +-----------+-----------+
                                    |           |           |
                                    v           v           v
                              +----------+ +----------+ +----------+
                              | Resource | | Resource | | Resource |
                              | Provider | | Provider | | Provider |
                              | (AWS)    | | (Azure)  | | (GCP)    |
                              +----------+ +----------+ +----------+
                                    |           |           |
                                    v           v           v
                                클라우드 API (실제 리소스 조작)
```

---

## 핵심 개념: 프로젝트와 스택

### 프로젝트

| 항목 | 설명 |
| --- | --- |
| 정의 | `Pulumi.yaml` 파일을 포함하는 디렉터리 |
| 역할 | 사용할 런타임, 프로그램 진입점 위치, 프로젝트 메타데이터를 지정 |
| 생성 | `pulumi new` 명령으로 생성 |
| 설정 파일 | `Pulumi.yaml` (필수, 대문자 P 필수, `.yml`/`.yaml` 모두 가능) |

**Pulumi.yaml 예시:**

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

### 스택

| 항목 | 설명 |
| --- | --- |
| 정의 | Pulumi 프로그램의 격리된, 독립적으로 구성 가능한 인스턴스 |
| 용도 | 개발 환경 구분(`development`, `staging`, `production`) 또는 기능 브랜치(`feature-x-dev`) |
| 이름 형식 | `stackName` / `orgName/stackName` / `orgName/projectName/stackName` |
| 생성 | `pulumi stack init <stackName>` |
| 제약 | 스택 이름은 프로젝트 내에서 고유해야 하며, 알파벳·숫자·하이픈·밑줄·마침표만 사용 가능 |

### 스택 출력 (Stack Outputs)

배포된 인프라의 중요 값(리소스 ID, IP 주소, DNS 이름 등)을 스택에서 내보낼 수 있다.

```typescript
// TypeScript
export const publicIp = server.publicIp;
export const publicDns = server.publicDns;
```

```python
# Python
pulumi.export("public_ip", server.public_ip)
pulumi.export("public_dns", server.public_dns)
```

---

## 기본 예제: AWS EC2 인스턴스 + 보안 그룹

```typescript
// TypeScript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

const group = new aws.ec2.SecurityGroup("web-sg", {
    description: "Enable HTTP access",
    ingress: [{
        protocol: "tcp",
        fromPort: 80,
        toPort: 80,
        cidrBlocks: ["0.0.0.0/0"],
    }],
});

const server = new aws.ec2.Instance("web-server", {
    ami: "ami-0319ef1a70c93d5c8",
    instanceType: "t2.micro",
    vpcSecurityGroupIds: [group.id],
});

export const publicIp = server.publicIp;
export const publicDns = server.publicDns;
```

```python
# Python
import pulumi
import pulumi_aws as aws

group = aws.ec2.SecurityGroup(
    "web-sg",
    description="Enable HTTP access",
    ingress=[{
        "protocol": "tcp",
        "from_port": 80,
        "to_port": 80,
        "cidr_blocks": ["0.0.0.0/0"],
    }],
)

server = aws.ec2.Instance(
    "web-server",
    ami="ami-0319ef1a70c93d5c8",
    instance_type="t2.micro",
    vpc_security_group_ids=[group.id],
)

pulumi.export("public_ip", server.public_ip)
pulumi.export("public_dns", server.public_dns)
```

---

## 상태 관리 및 백엔드

Pulumi는 인프라 메타데이터를 **상태(state)** 로 관리하며, 각 스택은 자체 상태를 가진다. 상태는 **백엔드**에 저장된다.

### 상태란?

- Pulumi가 클라우드 리소스를 언제, 어떻게 생성/읽기/삭제/수정할지 파악하기 위해 사용하는 메타데이터
- 체크포인트(checkpoint)라는 트랜잭션 스냅샷 형태로 저장
- 클라우드 자격 증명(credential)은 상태에 포함되지 않으며, CLI가 실행되는 클라이언트에 로컬로 유지

### 백엔드 옵션

| 백엔드 유형 | 옵션 | 특징 |
| --- | --- | --- |
| **Pulumi Cloud** (기본) | SaaS 또는 Self-hosted | 관리형 트랜잭션 REST API. 접근 제어·백업·가용성 모니터링 자동 처리. 삭제된 스택 복구 가능 |
| **DIY** - AWS S3 | `s3://<bucket>` | 클라우드 IAM으로 접근 제어 직접 관리 |
| **DIY** - Azure Blob | `azblob://<container>` | Azure IAM으로 접근 제어 직접 관리 |
| **DIY** - Google Cloud Storage | `gs://<bucket>` | GCP IAM으로 접근 제어 직접 관리 |
| **DIY** - S3 호환 서버 | Minio, Ceph 등 | 자체 관리 스토리지 |
| **DIY** - PostgreSQL | PostgreSQL 데이터베이스 | 관계형 DB 기반 상태 저장 |
| **DIY** - 로컬 파일시스템 | `file://<path>` | 단일 머신에서만 사용 |

### 로그인/로그아웃

```bash
# Pulumi Cloud에 로그인 (기본)
pulumi login

# 특정 백엔드에 로그인
pulumi login <backend-url>

# 현재 로그인 정보 확인
pulumi whoami -v

# 로그아웃
pulumi logout
```

### 상태 새로고침 (Refresh)

Pulumi는 매 실행 시 클라우드 리소스의 실시간 상태를 조회하지 않고 마지막 배포 상태를 기준으로 작업한다. 외부에서 변경된 사항을 반영하려면 수동으로 refresh를 수행해야 한다.

```bash
# 상태만 새로고침 (인프라 변경 없음)
pulumi refresh

# 새로고침 후 업데이트를 한 번에 수행
pulumi up --refresh
```

> Pulumi Cloud를 사용하면 예약된 드리프트 감지 및 자동 수정 기능을 사용할 수 있다.

---

## Pulumi Cloud vs OSS 비교

Pulumi의 IaC 도구는 오픈 소스이며, Pulumi Cloud는 이를 보완하는 관리형 서비스이다.

### 기능 비교 표

| 기능 | 오픈 소스 Pulumi | Pulumi Cloud |
| --- | --- | --- |
| **상태 백엔드** | DIY(객체 스토리지, PostgreSQL, 로컬 파일시스템) | 관리형 트랜잭션 백엔드. Terraform 상태 저장도 가능 |
| **배포 이력** | 스택별 체크포인트 이력 | 조직 전체 배포 이력 |
| **접근 제어** | 직접 관리(예: 클라우드 IAM) | 빌트인 RBAC + SAML/SSO 연동 |
| **시크릿 암호화** | 패스프레이즈 또는 자체 관리 KMS 키 | 관리형 암호화(기본). 별도 암호화 서비스 사용도 가능 |
| **설정/시크릿 관리** | 스택별 설정 파일만 | 스택별 설정 파일 + Pulumi ESC(중앙 관리, 재사용 가능한 환경) |
| **Policy as Code** | 디스크에 보관 후 CLI 인수로 전달 | 중앙 관리 자동 시행 + 사전 구축된 정책 팩 |
| **클라우드 리소스 인벤토리** | 미포함 | Pulumi Insights가 Pulumi 관리 외 리소스까지 스캔 |
| **드리프트 감지** | `pulumi refresh` 수동 실행 | 예약된 드리프트 감지 및 자동 수정 |
| **AI 지원** | CLI 및 VS Code 확장 내 AI 통합 | Pulumi Neo AI 에이전트(디버그, 코드 작성, 질문 응답) |
| **임시 환경** | 미포함 | Review Stacks, TTL Stacks |
| **관리형 배포** | 자동화 직접 구축 | Pulumi Deployments 관리형 서비스(Git push 트리거 등) |
| **REST API / Webhooks** | 미포함 | 문서화된 REST API 및 Webhooks |
| **컴플라이언스** | 해당 없음 | 연간 SOC 2 Type II 감사. 내보내기 가능한 감사 추적 |
| **지원** | 커뮤니티 지원. 상업 지원 가능 | 커뮤니티 지원. 엔터프라이즈 고객용 지원 플랜 |

### 비용 관점

| 항목 | 오픈 소스 | Pulumi Cloud |
| --- | --- | --- |
| 직접 비용 | 무료(객체 스토리지 비용은 저렴) | 무료 티어 + 유료 플랜 |
| 운영 부담 | 직접 보안·백업·가용성·접근 관리 수행 | 관리형 서비스가 운영 작업 대행 |
| 추가 기능 | RBAC, 리소스 인벤토리, 중앙 정책 시행 등은 직접 구축하거나 생략 | 플랫폼에 내장 |

> Pulumi Cloud는 호스팅된 SaaS와 자체 환경에서 실행할 수 있는 [Self-hosted](https://www.pulumi.com/docs/pulumi-cloud/self-hosted/) 에디션을 모두 제공한다. Individual 티어는 무료이다.

---

## Pulumi Cloud 추가 기능

Pulumi Cloud는 상태 백엔드 외에도 다음 기능을 제공한다.

| 기능 | 설명 |
| --- | --- |
| **조직(Organization)** | 프로젝트, 스택, 팀을 그룹화하는 최상위 계정 단위. 스택 정규화된 이름은 `<organization>/<project>/<stack>` 형식 |
| **팀 및 RBAC** | 팀 기반 접근 제어. SAML/SSO 연동. 세밀한 권한의 액세스 토큰 |
| **Pulumi ESC** | 중앙 관리, 구성 가능(composable), 버전 관리되는 환경. 설정과 시크릿을 여러 스택과 프로젝트에서 재사용 |
| **Pulumi Insights** | 클라우드 계정을 스캔하여 관리형/비관리형 리소스의 검색 가능한 인벤토리 구축 |
| **드리프트 감지** | 예약된 `pulumi refresh` 실행. 알림 또는 자동 수정 |
| **Pulumi Deployments** | Git push 등의 이벤트에 응답하여 원격으로 Pulumi 작업 실행 |
| **Pulumi Neo** | 배포 실패 디버그, 코드 작성, 인프라 질문 응답을 지원하는 AI 에이전트 |
| **Webhooks** | 배포, 리소스 업데이트, 정책 위반 등의 이벤트에 대한 Webhook 발송 |

---

## Pulumi 플랫폼 전체 기능 영역

공식 문서에 따른 Pulumi 플랫폼의 주요 기능 영역은 다음과 같다.

| 기능 영역 | 설명 |
| --- | --- |
| **Infrastructure as Code** | TypeScript, Python, Go, .NET, Java, YAML로 클라우드 인프라 정의 및 관리 |
| **Deployments & Workflows** | 클라우드 호스팅 배포, 드리프트 감지, 상태 관리, 자동화 |
| **Secrets & Configuration (ESC)** | 중앙 집중식 시크릿 및 설정 관리 |
| **Insights & Governance** | 클라우드 인프라 검색, 컴플라이언스, 정책 시행 |
| **Version Control** | GitHub, GitLab, Azure DevOps와의 통합 |
| **Internal Developer Platform (IDP)** | 템플릿, 가드레일, 개발자 포털을 통한 셀프서비스 인프라 |
| **Infrastructure AI** | Pulumi Neo 및 자연어 지원을 통한 인프라 자동화 |

---

## 주요 CLI 명령어

| 명령어 | 설명 |
| --- | --- |
| `pulumi new` | 새 프로젝트 생성 |
| `pulumi stack init <name>` | 새 스택 생성 |
| `pulumi stack ls` | 프로젝트의 스택 목록 조회 |
| `pulumi stack select <name>` | 활성 스택 변경 |
| `pulumi up` | 스택 배포(업데이트) |
| `pulumi preview` | 변경 사항 미리보기(실제 배포 없음) |
| `pulumi refresh` | 상태를 실제 클라우드 리소스와 동기화 |
| `pulumi destroy` | 스택의 모든 리소스 삭제 |
| `pulumi login` | 백엔드에 로그인 |
| `pulumi logout` | 현재 백엔드에서 로그아웃 |
| `pulumi whoami` | 현재 로그인된 사용자 확인 |
| `pulumi config set` | 스택 설정 값 구성 |
| `pulumi stack export` | 스택 상태를 파일로 내보내기 |
| `pulumi stack import` | 파일에서 스택 상태 가져오기 |
