> 원문 출처
>
> - https://www.pulumi.com/ai/
> - https://www.pulumi.com/docs/ai/
> - https://www.pulumi.com/docs/ai/get-started/
> - https://www.pulumi.com/docs/ai/tasks/
> - https://www.pulumi.com/docs/ai/pull-requests/
> - https://www.pulumi.com/docs/ai/running-previews/
> - https://www.pulumi.com/docs/ai/automations/
> - https://www.pulumi.com/docs/ai/integrations/
> - https://www.pulumi.com/docs/ai/integrations/mcp/
> - https://www.pulumi.com/docs/ai/integrations/cli/
> - https://www.pulumi.com/docs/ai/integrations/github/
> - https://www.pulumi.com/docs/ai/integrations/slack/
> - https://www.pulumi.com/docs/ai/settings/
> - https://www.pulumi.com/docs/ai/skills/
> - https://www.pulumi.com/docs/ai/pulumi-cli/

# Pulumi AI — Infrastructure AI (Pulumi Neo)

Pulumi Neo는 자연어 기반 AI 인프라 자동화 에이전트다. 대화형 인터페이스로 인프라 코드 생성, 변경 제안, 프리뷰 실행, PR 생성까지 전 과정을 수행하며, Pulumi Cloud 콘솔·CLI·Slack·GitHub 어디서든 접근할 수 있다.

현재 **Public Preview** 단계이며 무료로 사용 가능하다. `pulumi preview` 실행 시 워크플로우 분(workflow minutes)이 소비된다.

---

## 목차

1. [아키텍처 및 핵심 개념](#1-아키텍처-및-핵심-개념)
2. [시작하기 (Get Started)](#2-시작하기-get-started)
3. [태스크 (Tasks)](#3-태스크-tasks)
4. [풀 리퀘스트 (Pull Requests)](#4-풀-리퀘스트-pull-requests)
5. [프리뷰 (Previews)](#5-프리뷰-previews)
6. [오토메이션 (Automations)](#6-오토메이션-automations)
7. [인테그레이션 (Integrations)](#7-인테그레이션-integrations)
8. [Pulumi CLI (`pulumi neo`)](#8-pulumi-cli-pulumi-neo)
9. [설정 (Settings)](#9-설정-settings)
10. [에이전트 스킬 (Agent Skills)](#10-에이전트-스킬-agent-skills)

---

## 1. 아키텍처 및 핵심 개념

### Neo가 수행하는 작업

| 기능 | 설명 |
|------|------|
| 인프라 코드 생성 | 자연어 요청을 Pulumi 코드(TypeScript, Python, Go, C#, Java, YAML)로 변환 |
| 코드 리뷰 | 기존 코드 분석 및 개선 제안 |
| 에러 디버깅 | 실패한 프리뷰/배포 원인 분석 및 해결 |
| 프리뷰 실행 | `pulumi preview`를 통해 변경 사항 검증 |
| PR 생성 | 코드 변경을 PR로 제안하여 기존 리뷰 프로세스 통과 |
| 반복 작업 자동화 | 오토메이션을 통해 정기적 인프라 감사·점검 수행 |

### 접근 채널

| 채널 | 설명 |
|------|------|
| Pulumi Cloud 콘솔 | **Agent Tasks** 메뉴에서 Neo 태스크 시작 |
| Pulumi CLI | `pulumi neo` 명령으로 터미널에서 대화형 세션 시작 |
| Slack | 채널에서 `@Neo` 멘션으로 태스크 시작 |
| GitHub | PR·이슈에서 `@pulumi-neo` 멘션으로 태스크 시작 |

### 권한 모델

Neo는 **대화 중인 사용자의 RBAC 권한** 내에서만 동작한다. 사용자가 할 수 없는 작업은 Neo도 수행할 수 없다.

- 권한 상승 위험 없음
- 특별한 관리자 권한 불필요
- 사용자 간 대화 격리

### 제한사항

| 항목 | 설명 |
|------|------|
| 코드 변경만 가능 | API·UI 작업(배포 설정, 스택 구성, 환경 관리 등)은 불가 |
| 리포지토리 생성 불가 | 기존 리포지토리 내에서만 작업 |
| 프로젝트 초기화 불가 | 기존 Pulumi 프로젝트 내에서만 작업 |

---

## 2. 시작하기 (Get Started)

### 선행 조건

| 조건 | 설명 |
|------|------|
| Pulumi Cloud 조직 | Neo가 활성화된 조직 필요 (기본 활성화) |
| VCS 연동 (권장) | GitHub, Azure DevOps, GitLab 연동 시 코드 읽기·PR 생성 가능 |
| 클라우드 자격 증명 | 프리뷰 실행 시 ESC 환경 또는 스택 구성을 통해 자격 증명 제공 |

VCS 연동이 없어도 코드 변경 제안은 가능하지만, PR 생성 및 코드 컨텍스트 파악이 제한된다.

### Neo 활성화/비활성화

Neo는 기본 활성화 상태다. 비활성화하려면:

1. **Settings > Neo Settings > General** 이동
2. **Enable Neo for organization** 토글

### 읽기 전용 모드

태스크 생성 시 두 가지 권한 수준 선택 가능:

| 옵션 | Neo가 할 수 있는 작업 |
|------|----------------------|
| **Use my permissions** | 전체 접근 (기본) |
| **Read-only** | 읽기, 프리뷰, PR 생성. 인프라 변경(mutation) 불가 |

읽기 전용 모드에서도 스택 상태 조회, 프리뷰 실행, 코드 작성·리팩터링, 브랜치 생성, PR 오픈은 가능하다.

### 첫 번째 태스크 빠른 시작

1. Pulumi Cloud에서 **Agent Tasks** 메뉴 클릭
2. 읽기 전용 분석 태스크로 시작:

```
Which of my resources are not using their latest major version?
```

3. 후속 질문:

```
Which version issue should I address first?
```

4. 코드 변경 제안 요청:

```
Can you propose the necessary code change to address the highest priority item?
```

5. Neo가 코드 수정 → 프리뷰 → PR 생성 순서로 진행

---

## 3. 태스크 (Tasks)

태스크는 Neo의 기본 작업 단위다. 각 태스크는 사용자와 Neo 간의 대화 형태로 진행된다.

### 태스크 실행 흐름

1. 사용자가 자연어로 목표 설명
2. Neo가 수행 단계 아웃라인 제시
3. 단계별 실행 (승인 모드에 따라 사용자 승인 요청)
4. 코드 변경 시 프리뷰 실행
5. PR 생성 제안

### Plan Mode

복잡한 태스크의 경우 **Plan Mode**를 활성화하면 실행 전 심층 조사·계획 수립이 이루어진다.

| 단계 | 설명 |
|------|------|
| 조사 | 기존 인프라 검사, 코드 읽기, 의존성 확인, 패턴 연구 |
| 종합 | 발견한 내용을 기반으로 구체적 계획 수립 |
| 반복 | 대화를 통해 가정 검증, 대안 요청, 세부 사항 조정 |
| 승인 대기 | 실행 전 사용자의 명시적 승인 필요 |

**Plan Mode 활용 시점:**

- 다중 스택 복합 작업
- 낯선 인프라 환경
- 자율 실행 (계획 승인이 주요 통제 지점인 경우)

**Plan Mode가 불필요한 경우:** 한 문장으로 설명 가능한 단순 작업

### 태스크 모드 (Task Modes)

실행 중 Neo의 자율성을 제어하는 설정이다.

| 모드 | 승인 요구 사항 |
|------|----------------|
| **Review** (기본) | `pulumi preview`, `pulumi up`, PR 오픈 모두 승인 필요 |
| **Balanced** | `pulumi up` 실행 전에만 승인 요청 |
| **Auto** | 어떤 승인도 요청하지 않음 |

태스크 모드는 Plan Mode와 독립적이다. 조합 가능: 예를 들어 Plan Mode + Auto Mode로 사전에 철저히 검토한 뒤 자율 실행.

### 컨텍스트·공유·이력

| 항목 | 설명 |
|------|------|
| 엔티티 컨텍스트 | 태스크 시작 시 스택·리포지토리 지정 가능 |
| 소유권 | 태스크는 생성한 사용자 소유 |
| 공유 | 조직 내 읽기 전용 링크로 공유 가능 (보안 경계 유지) |
| 중단·재개 | 브라우저를 닫아도 Neo는 계속 작업. 복귀 시 진행 상황 표시 |
| 태스크 이력 | Agent Tasks 페이지에서 전체 이력 조회 가능. 단, 에이전트가 1시간 이상 유휴 시 캐시 손실 가능 |

---

## 4. 풀 리퀘스트 (Pull Requests)

Neo의 모든 코드 변경은 PR을 통해 제안된다. 기존 리뷰 프로세스, CI/CD 파이프라인, 감사 추적이 그대로 적용된다.

### 선행 조건

| 조건 | 설명 |
|------|------|
| VCS 연동 | GitHub, Azure DevOps, GitLab 연동 필요 |
| 코드 접근 | 스택 태그를 통해 Git 리포지토리 위치 파악 |

### PR 내용

Neo가 생성하는 PR에는 다음이 포함된다:

- 변경 사항 설명 제목
- 해결하는 문제 설명
- 수정된 리소스 목록
- 프리뷰 출력 요약
 Neo 태스크로의 링크

### 변경 요청

PR 리뷰 중 후속 프롬프트로 수정을 요청할 수 있다. Neo는 PR 컨텍스트와 대화 이력을 모두 이해한다.

### CI/CD 연동

Neo의 PR은 기존 CI/CD 워크플로우를 자동으로 트리거한다. Pulumi 프리뷰, 보안 스캔, 정책 검사, 테스트가 일반 PR과 동일하게 실행된다. 워크플로우 실패 시 Neo에게 특정 이슈 해결을 요청하면 동일 PR에 수정을 푸시한다.

지원 CI/CD:
- GitHub Actions
- Azure DevOps Pipelines
- GitLab CI

---

## 5. 프리뷰 (Previews)

Neo는 Pulumi Cloud에서 직접 `pulumi preview`를 실행하여 제안된 인프라 변경을 PR 생성 전에 검증할 수 있다.

### 자격 증명 구성

| 방식 | 설명 |
|------|------|
| 스택 구성 (Stack Config) | CLI, 구성 파일, API로 직접 정의 |
| ESC 환경 (권장) | ESC를 통한 유연한 구성·시크릿 관리. OIDC 연동 지원 |

필요 자격 증명:

| 클라우드 | 참고 문서 |
|----------|-----------|
| AWS | 환경 변수 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` |
| Azure | 환경 변수 `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID` |
| GCP | 서비스 계정 키 파일 |

### 프리뷰 워크플로우

1. Neo가 해결책 도달 → `preview` 실행 요청
2. 사용자 승인 또는 거부
3. 프리뷰 실행 결과 확인:
   - 수정될 리소스 수
   - 각 리소스별 구체적 변경
   - 에러·경고
   - 정책 위반 (있는 경우)
4. PR 오픈 제안 또는 추가 수정 요청

### 정책 준수 검사

스택에 Pulumi IaC 정책이 연결된 경우, 프리뷰 과정에서 정책이 자동으로 검사된다.

### 보안 고려사항

- 요청한 사용자의 권한으로 제한
- ESC 자격 증명에 연결된 IAM 역할로 제한
- 조직 수준 정책·제한 적용

---

## 6. 오토메이션 (Automations)

오토메이션은 Neo 태스크를 반복 작업으로 변환한다. 프롬프트를 정의하고 주기를 설정하면, Neo가 해당 간격으로 태스크를 실행한다. 변경 사항이 발생하면 PR을 연다.

### 대표 활용 사례

| 유형 | 설명 |
|------|------|
| Provider freshness check | 프로바이더 버전 최신화 점검 |
| Encryption audit | 암호화 설정 감사 |
| Backup audit | 백업 구성 감사 |
| Activity digest | 활동 요약 리포트 |

### 기본 설정

오토메이션 태스크는 두 가지 기본값이 적용된다:

| 설정 | 기본값 | 설명 |
|------|--------|------|
| Approval mode | **Auto** | 각 단계마다 사람 승인 대기 없이 진행 |
| Permission mode | **Read-only** | 상태 읽기 및 PR 제안은 가능, 직접 인프라 변경은 불가 |

### 설정 적용 우선순위

1. 개별 오토메이션 설정 (있는 경우)
2. 조직 수준 기본값
3. Auto 승인 + Read-only 권한 (내장 폴백)

### 오토메이션 생성

1. **Neo Tasks** > **Automations** 탭 > **New automation**
2. 템플릿 선택 또는 빈 캔버스에서 시작
3. 이름, 프롬프트(인테그레이션 선택기 포함), 주기 설정

| 주기 옵션 | 설명 |
|-----------|------|
| Hourly | 매시간 |
| Daily | 매일 |
| Weekdays | 평일 |
| Weekly | 매주 |

### 오토메이션과 Neo의 상호작용

- 조직·프로젝트 수준 **Custom Instructions**가 예약 태스크에도 적용
- **MCP 인테그레이션**은 구성한 사용자의 자격 증명 사용
- **CLI 인테그레이션**은 설정 시 구성한 자격 증명 사용

### 권한 동작

예약 태스크는 **예약한 사용자의 RBAC 권한**으로 실행되며, 실행 시점에 권한이 재평가된다.

### 제한사항

현재 예약 태스크(scheduled tasks)만 지원된다.

---

## 7. 인테그레이션 (Integrations)

인테그레이션은 Neo를 외부 시스템에 연결하여 컨텍스트를 확장하고, 사용자가 이미 작업 중인 도구에서 Neo에 접근할 수 있게 한다.

### 인테그레이션 분류

| 유형 | 설명 | 문서 |
|------|------|------|
| **MCP 인테그레이션** | MCP 서버를 통해 외부 서비스 연결. 이슈 트래커, 관측 플랫폼, 런북, 온콜 도구 | [/docs/ai/integrations/mcp/](https://www.pulumi.com/docs/ai/integrations/mcp/) |
| **CLI 인테그레이션** | `aws`, `gcloud`, `az`, `kubectl` CLI를 ESC 자격 증명으로 실행 | [/docs/ai/integrations/cli/](https://www.pulumi.com/docs/ai/integrations/cli/) |
| **GitHub** | PR·이슈에서 `@pulumi-neo` 멘션으로 태스크 시작 | [/docs/ai/integrations/github/](https://www.pulumi.com/docs/ai/integrations/github/) |
| **Slack** | 채널에서 `@Neo` 멘션으로 태스크 시작 | [/docs/ai/integrations/slack/](https://www.pulumi.com/docs/ai/integrations/slack/) |

모든 인테그레이션은 **조직 수준**에서 관리자가 구성하며, 활성화하면 조직 내 모든 Neo 태스크에서 사용 가능하다.

### MCP 인테그레이션 상세

MCP(Model Context Protocol) 서버를 통해 외부 서비스를 Neo에 연결한다.

#### 지원 서비스

| 서비스 | 기능 |
|--------|------|
| **Atlassian** (Jira, Confluence) | 이슈 추적, 위키 조회 |
| **Datadog** | 메트릭·트레이스·로그 조회 |
| **Honeycomb** | 관측 데이터 분석 |
| **Linear** | 이슈 관리 |
| **PagerDuty** | 인시던트·온콜 관리 |
| **Supabase** | 데이터베이스·프로젝트 관리 |

#### 자격 증명 관리

- 인테그레이션 자격 증명은 **조직별 암호화 키**로 암호화되어 Pulumi Cloud에 저장
- 태스크 실행 시에만 복호화되어 사용
- 자격 증명은 언어 모델에 노출되지 않으며, 태스크 상태에 포함되지 않음

#### 태스크별 제어

기본적으로 모든 태스크가 조직의 활성 인테그레이션을 상속받는다. 태스크 작성기에서 개별 인테그레이션을 끌 수 있다.

#### 인테그레이션 장애 처리

인테그레이션은 각 메시지 시작 시 독립적으로 해결된다. 하나의 인테그레이션이 실패해도 나머지 인테그레이션으로 태스크가 계속 진행된다.

#### 서비스별 구성 요약

| 서비스 | 필수 자격 증명 | 비고 |
|--------|----------------|------|
| Atlassian | Service Account API Token, Site URL | `admin.atlassian.com`에서 Rovo MCP 활성화 필요 |
| Datadog | Site 코드, API Key, App Key | 지원 코드: `us1`, `us3`, `us5`, `eu1`, `ap1`, `ap2` |
| Honeycomb | Key ID, Key Secret | Management API Key에 MCP + Environments 스코프 필요 |
| Linear | API Key | 개인 토큰이므로 모든 작업이 토큰 생성자 귀속 |
| PagerDuty | User API Token | 개인 계정 연결. 공유용 전용 사용자(예: `pulumi-bot`) 권장 |
| Supabase | Access Token | 개인 계정 연결. 공유용 전용 사용자 권장 |

### CLI 인테그레이션 상세

Neo가 클라우드 인프라를 직접 조회할 수 있도록 CLI 도구를 연결한다. 자격 증명은 Pulumi ESC 환경에서 관리된다.

#### 지원 CLI

| CLI | 도구 | 필수 환경 변수 | 선택 사항 |
|-----|------|-----------------|-----------|
| `aws` | AWS CLI (`aws`) | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | `AWS_SESSION_TOKEN`, `AWS_REGION` |
| `gcp` | Google Cloud CLI (`gcloud`) | `GOOGLE_APPLICATION_CREDENTIALS` | `CLOUDSDK_CORE_PROJECT` |
| `azure` | Azure CLI (`az`) | `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID` | `AZURE_SUBSCRIPTION_ID` |
| `kubernetes` | Kubernetes CLI (`kubectl`) | `KUBECONFIG` | 없음 |

#### 자격 증명 설정 방식

| 방식 | 설명 | 적용 |
|------|------|------|
| **Login Provider Setup 마법사** (권장) | Pulumi Cloud에서 OIDC 트러스트 설정 + ESC 환경 생성을 자동화 | AWS, GCP, Azure |
| 수동 설정 | 클라우드 측 OIDC 트러스트 관계를 수동으로 구성 후 ESC 환경 작성 | AWS, GCP, Azure |
| Kubernetes 전용 | 스택 출력 kubeconfig 또는 정적 kubeconfig를 ESC에 구성 | Kubernetes |

**Pulumi Cloud는 CLI 인테그레이션용 클라우드 자격 증명을 저장하지 않는다.** ESC가 자격 증명을 소유하고 태스크 실행 시에만 복호화한다.

#### CLI 인테그레이션 연결

1. ESC 환경 생성 (CLI가 필요한 자격 증명 출력)
2. **Neo Settings > Integrations > CLI tools**에서 CLI 선택 후 **Connect**
3. 이름, 설명(Notes), ESC 환경 지정

동일 CLI의 여러 인스턴스 연결 가능 (예: `production-aws`, `staging-aws`).

#### 감사

연결/해제 이벤트가 조직 감사 로그에 기록된다. Neo의 CLI 호출은 태스크 트랜스크립트에서 확인 가능하다.

### GitHub 인테그레이션 상세

PR 설명, 리뷰 코멘트, 이슈에서 `@pulumi-neo`를 멘션하면 Neo가 동일 스레드에 응답한다.

#### 설정

1. [Pulumi GitHub App](https://www.pulumi.com/docs/integrations/version-control/github-app/) 설치 (GitHub 조직 관리자)
2. Pulumi 사용자를 GitHub에 연결 (GitHub로 가입한 경우 자동 연결)
3. 리포지토리에서 `@pulumi-neo` 멘션

#### 권한

GitHub에서 시작된 태스크는 해당 Pulumi Cloud 사용자의 RBAC 권한으로 실행된다.

#### 제한사항

- GitHub.com만 지원. GitHub Enterprise Server 미지원.

### Slack 인테그레이션 상세

채널에서 `@Neo`를 멘션하면 Neo가 스레드 내에서 응답한다.

#### 설정

1. [Pulumi Neo Slack 앱](https://api.pulumi.com/api/slack/neo/install) 설치 (Slack 워크스페이스 관리자)
2. Pulumi Cloud 계정 설정 > Neo 설정에서 Slack ID 연결
3. 채널에서 `@Neo` 멘션

#### 권한

Slack에서 시작된 태스크는 연결된 Pulumi Cloud 사용자의 RBAC 권한으로 실행된다.

#### 제한사항

- 다이렉트 메시지(DM)에서는 Neo 대화 시작 불가
- 스레드당 하나의 태스크

---

## 8. Pulumi CLI (`pulumi neo`)

`pulumi neo`는 터미널에서 Neo를 사용하는 인터랙티브 명령이다. 프로젝트 디렉토리에서 실행하면 로컬 환경의 CLIs, 환경 변수, kubeconfig, 프로젝트 컨텍스트를 그대로 활용한다.

### 로컬 실행의 장점

| 항목 | 설명 |
|------|------|
| 로컬 인증 | 이미 인증된 CLIs를 그대로 사용 |
| 환경 변수 | 구성된 환경 변수·kubeconfig 활용 |
| 프로젝트 컨텍스트 | 현재 편집 중인 프로젝트에 직접 접근 |
| 인터랙티브 | 후속 메시지로 실시간 작업 조정 |

### 제어

Pulumi Cloud의 제어가 터미널에도 동일하게 적용된다:

- **승인 모드** (manual, balanced, auto)
- **권한 모드** (default, read-only)
- **Plan Mode** (Shift+Tab으로 전환)

### 인테그레이션

Pulumi Cloud에서 구성한 GitHub, Slack, MCP 인테그레이션이 터미널에서도 동일하게 동작한다.

### 에이전트 핸드오프

[Neo handoff skill](https://github.com/pulumi/agent-skills/tree/main/delegation)을 통해 다른 AI 에이전트(Claude Code, Cursor 등)가 `pulumi neo`를 내부적으로 사용하여 인프라 작업을 Neo에 위임할 수 있다.

### 권한

`pulumi neo`는 `pulumi login`으로 인증된 사용자의 RBAC 권한을 사용한다. 인증, RBAC, 감사가 모두 Pulumi Cloud 로그인을 통해 처리된다.

### 시작

```bash
pulumi login
pulumi neo   # Pulumi.yaml이 있는 디렉토리에서 실행
```

---

## 9. 설정 (Settings)

Neo는 조직, 리포지토리, 사용자 세 수준에서 구성된다.

### 설정 계층 (우선순위 오름차순)

| 우선순위 | 설정 | 적용 범위 |
|----------|------|-----------|
| 1 | Custom Instructions | 조직 전체 태스크에 적용되는 기본값 |
| 2 | Repository Instructions (`AGENTS.md`) | 리포지토리 진입 시 프로젝트별 규칙 |
| 3 | Conversation Instructions | 태스크 대화에서 직접 제공한 지시 |

상위 계층이 하위 계층의 설정을 덮어쓴다.

### Neo 접근 제어

1. Pulumi Cloud 좌측 탐색에서 **Neo Settings** 선택
2. **General** 탭 선택
3. **Enable Neo for organization** 토글 on/off

### Custom Instructions

조직의 표준, 선호사항, 요구사항을 Neo에 미리 정의하여 모든 태스크에 자동 적용한다.

#### 포함 가능 항목

| 항목 | 예시 |
|------|------|
| 명명 규칙 | `{service}-{environment}-{region}` |
| 규정 준수 요구사항 | 필수 태그, 라벨, 구성 |
| 기술 선호 | 선호 언어, 프레임워크, 클라우드 서비스 |
| 비용 가이드라인 | 예산 고려사항, 비용 최적화 선호 |
| 자동 동작 | PR에 월간 예상 비용 포함 등 |

#### 예시

```text
All AWS resources must follow these standards:
- Naming convention: {service}-{environment}-{region}
- Required tags: environment and owner
- Use encryption at rest for all storage resources
- Always include estimated monthly costs in pull requests when proposing new infrastructure
```

#### 구성 위치

**Neo Settings > Organization instructions** 탭

### Repository Instructions (`AGENTS.md`)

리포지토리 루트에 `AGENTS.md` 파일을 배치하면 Neo가 리포지토리 진입 시 해당 파일을 읽고 지시를 따른다.

#### 일반적 활용

| 항목 | 설명 |
|------|------|
| 빌드/테스트 명령 | 의존성 설치, 테스트 실행, 린트 방법 |
| 환경 설정 | 필요한 도구 버전, 환경 변수, 런타임 구성 |
| Git 훅 | 훅 경로, 커밋 전 실행 명령 |
| 코딩 표준 | 언어 선호, 명명 규칙, 스타일 요구사항 |

#### 서브디렉토리 지원

`AGENTS.md`를 서브디렉토리에도 배치할 수 있다. Neo가 서브디렉토리에서 작업 시 가장 가까운 `AGENTS.md`를 읽으며, 서브디렉토리 파일이 상위 파일보다 우선한다.

#### 부트스트래핑 예시

Custom Instructions에 다음을 추가하면 Neo가 리포지토리 진입 후 즉시 `AGENTS.md`를 처리한다:

```text
When working on a task, immediately clone the associated repository
(and any additional repositories involved) and look for an AGENTS.md
file. Follow any setup instructions it contains before proceeding
with the task.
```

### Slash Commands

검증된 프롬프트를 단축 명령으로 등록하여 팀 전체가 사용할 수 있다. 대화에서 `/`를 입력하면 사용 가능한 명령이 표시된다.

#### 내장 명령

| 명령 | 설명 |
|------|------|
| `/get-started` | Neo 기능 및 효과적인 요청 작성법 안내 |
| `/policy-issues-report` | 가장 심각한 정책 위반 목록 |
| `/component-version-report` | 프라이빗 레지스트리에서 구버전 컴포넌트 목록 |
| `/provider-version-report` | 구버전 프로바이더 목록 |

#### 커스텀 명령 생성

**Neo Settings > General > Slash commands**에서 추가.

### Task Modes

태스크 모드는 Neo가 자동으로 수행할 수 있는 작업을 제어하는 프리셋이다.

| 모드 | 설명 |
|------|------|
| **Auto** | 모든 요청을 사용자 개입 없이 자동 승인 |
| **Balanced** | `pulumi up`을 제외한 요청은 자동 승인 |
| **Review** | 모든 요청에 수동 승인 필요 |

기본 태스크 모드는 조직 관리자가 설정한다. 사용자는 개별 태스크에서 기본값을 재정의할 수 있다.

### 알림 (Notifications)

사용자 수준 설정으로, 각 사용자가 자신의 계정 설정에서 독립적으로 구성한다. Pulumi Cloud 탭이 백그라운드에 있을 때 Neo의 주의가 필요한 경우 알림이 발생한다.

| 알림 유형 | 설명 | 기본값 |
|-----------|------|--------|
| 브라우저 알림 | OS 알림 영역에 표시 | Off |
| 오디오 알림 | 짧은 차임 소리 재생 | Off |

알림은 Neo가 응답을 완료하거나 승인을 요청할 때 발생한다. 사용자가 Pulumi Cloud 탭을 보고 있을 때는 알림이 억제된다.

---

## 10. 에이전트 스킬 (Agent Skills)

Pulumi Agent Skills는 AI 코딩 어시스턴트에게 Pulumi 특화 워크플로우를 교육하는 **지식 패키지**다. 인프라 마이그레이션, 시크릿 관리, 코드 변환 등 숙련된 실무자 수준의 Pulumi 작업 방식을 에이전트에 제공한다.

### 지원 AI 어시스턴트

| 어시스턴트 | 비고 |
|-----------|------|
| Claude Code | Anthropic 공식 CLI |
| OpenAI Codex | OpenAI 코딩 에이전트 |
| Cursor | AI 코드 에디터 |
| GitHub Copilot | GitHub AI 어시스턴트 |
| Google Gemini | Google CLI |
| JetBrains Junie | JetBrains AI 어시스턴트 |

Agent Skills는 [agentskills.io](https://agentskills.io) 오픈 표준을 따른다.

### 스킬 그룹

#### Migration Plugin — 타 도구에서 Pulumi로 마이그레이션

| 스킬 | 설명 |
|------|------|
| `pulumi-terraform-to-pulumi` | Terraform 프로젝트를 Pulumi로 마이그레이션 |
| `pulumi-cdk-to-pulumi` | AWS CDK 애플리케이션을 Pulumi로 마이그레이션 |
| `cloudformation-to-pulumi` | AWS CloudFormation 스택/템플릿을 Pulumi로 마이그레이션 |
| `pulumi-arm-to-pulumi` | Azure ARM 템플릿 및 Bicep을 Pulumi로 마이그레이션 |

#### Pulumi Plugin — Pulumi 인프라 작성 및 운영

| 스킬 | 설명 |
|------|------|
| `pulumi-overview` | `pulumi do`, IaC 프로젝트, Pulumi Cloud 전반의 엔트리 포인트. 전문 스킬로 라우팅 |
| `pulumi-best-practices` | 신뢰할 수 있는 Pulumi 프로그램 작성 모범 사례 |
| `pulumi-component` | `ComponentResource` 클래스 작성 가이드 |
| `pulumi-automation-api` | Pulumi Automation API 사용 모범 사례 |
| `pulumi-esc` | Pulumi ESC (Environments, Secrets, Configuration) 작업 가이드 |
| `package-usage` | 조직 전체 스택의 패키지 사용 및 버전 감사 |
| `provider-upgrade` | Pulumi 프로바이더 안전 업그레이드 및 diff 조정 |
| `pulumi-upgrade-provider` | `upgrade-provider` 도구로 프로바이더 리포 업그레이드 자동화 |
| `upstream-patches` | 프로바이더 리포의 업스트림 Terraform 패치 스택 관리 |

#### Delegation Plugin — 작업을 Neo에 위임

| 스킬 | 설명 |
|------|------|
| `pulumi-neo-handoff` | 현재 작업을 Pulumi Neo 태스크로 이관. 목표, 리포지토리 포인터, 대화 요약 포함 |

### 설치

#### Claude Code 플러그인 시스템

```bash
claude plugin marketplace add pulumi/agent-skills
claude plugin install pulumi-migration      # 마이그레이션 스킬
claude plugin install pulumi                # Pulumi 스킬 (overview + 전문)
claude plugin install pulumi-delegation     # 위임 스킬 (Neo handoff)
```

#### OpenAI Codex

```bash
codex plugin marketplace add pulumi/agent-skills
```

마켓플레이스 등록 후 `codex` 실행 → 플러그인 마켓플레이스에서 `pulumi-migration`, `pulumi`, `pulumi-delegation` 선택.

#### 범용 설치 (모든 AI 어시스턴트)

```bash
# 전체 스킬 설치
npx skills add pulumi/agent-skills --skill '*'

# 개별 플러그인 그룹 설치
npx skills add pulumi/agent-skills/migration --skill '*'      # 마이그레이션 4개
npx skills add pulumi/agent-skills/pulumi --skill '*'         # Pulumi 9개
npx skills add pulumi/agent-skills/delegation --skill '*'     # 위임 1개

# 특정 에이전트 지정
npx skills add pulumi/agent-skills --skill '*' --agent junie
```

### 사용 예시

```text
# 일반 인프라 생성
Create an S3 bucket and a Cloudflare DNS record

# Terraform → Pulumi 마이그레이션
Convert this Terraform configuration to Pulumi TypeScript

# CDK → Pulumi 마이그레이션
Help me migrate my CDK application to Pulumi

# ESC로 시크릿 관리
Set up AWS OIDC credentials using Pulumi ESC

# 컴포넌트 작성
Help me create a reusable Pulumi component for a web service

# 프로바이더 업그레이드
Help me upgrade the Pulumi AWS provider safely without changing real infrastructure

# Neo에 작업 위임
Hand this off to Neo to apply the staging migration in production
```

### 기여

[agent-skills 리포지토리](https://github.com/pulumi/agent-skills)에서 새 스킬 작성, 기존 스킬 개선, 이슈 리포트가 가능하다. 기여 가이드라인은 [CONTRIBUTING.md](https://github.com/pulumi/agent-skills/blob/main/CONTRIBUTING.md) 참조.
