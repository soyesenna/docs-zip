# Pulumi 상태와 백엔드

> https://www.pulumi.com/docs/iac/concepts/state-and-backends/
> https://www.pulumi.com/docs/iac/guides/basics/
> https://www.pulumi.com/docs/iac/operations/stack-management/using-a-diy-backend/
> https://www.pulumi.com/docs/iac/cli/commands/pulumi_state/

Pulumi는 클라우드 리소스를 관리하기 위해 인프라 메타데이터를 저장하며, 이 메타데이터를 **상태(state)**라고 한다. 각 [stack](https://www.pulumi.com/docs/concepts/stack/)은 자체 상태를 가지며, Pulumi는 이 상태를 기반으로 리소스를 생성·읽기·삭제·갱신할지 결정한다. 상태는 사용자가 선택한 **백엔드(backend)**에 저장된다. 백엔드는 CLI가 업데이트를 조정하고 스택 상태를 읽고 쓰는 데 사용하는 API 및 스토리지 엔드포인트다.

기본값은 관리형 **Pulumi Cloud** 백엔드이며, AWS S3, Azure Blob Storage, Google Cloud Storage, S3 호환 서버(Minio, Ceph 등), PostgreSQL, 로컬 파일시스템 등의 **DIY 백엔드**도 지원한다.

---

## 상태의 역할

Pulumi 상태는 다음 핵심 정보를 포함한다:

| 정보 | 설명 |
|------|------|
| 리소스 매핑 | 프로그램에 선언된 리소스와 실제 클라우드 리소스 간의 대응 관계 |
| 리소스 속성 | 마지막 배포 이후의 리소스 속성값 |
| 의존성 | 리소스 간의 의존 관계 및 부모-자식 관계 |
| 보안 설정 | 암호화된 비밀값 (secrets) |

> Pulumi 상태에는 클라우드 자격 증명(credential)이 포함되지 않는다. 자격 증명은 CLI가 실행되는 로컬 클라이언트에만 보관되며, Pulumi Cloud 백엔드를 사용하더라도 원격으로 전송되지 않는다.

---

## 상태 체크포인트

Pulumi 상태는 일반적으로 **체크포인트(checkpoint)**라는 트랜잭션 스냅샷으로 저장된다. Pulumi는 실행 중에 체크포인트를 조기·자주 기록하여 데이터베이스 트랜잭션처럼 안정적으로 동작한다. 체크포인트를 통해:

- 프로그램의 목표 상태와 마지막 업데이트를 비교(diff)
- 장애 발생 시 복구
- 정확한 리소스 삭제(cleanup)

Pulumi Cloud 백엔드는 트랜잭션 API를 통해 모든 체크포인트를 기록하며, 네트워크 중단 같은 비정상 장애 시나리오에서도 복구가 가능하다. DIY 백엔드도 체크포인트 히스토리(`.pulumi/history/` 디렉터리)를 유지하지만, blob 스토리지 기반 백엔드는 부분 장애 복구에 더 어려움이 있을 수 있다.

---

## 백엔드 선택

Pulumi는 두 가지 유형의 상태 백엔드를 지원한다:

| 분류 | 백엔드 | 설명 |
|------|--------|------|
| **Pulumi Cloud** | 관리형 백엔드 | 온라인 또는 자체 호스팅 Pulumi Cloud 애플리케이션. 기본 백엔드이며 CLI 설치 후 추가 설정 불필요 |
| **DIY 백엔드** | 직접 관리 | AWS S3, Azure Blob Storage, Google Cloud Storage, S3 호환 서버, PostgreSQL, 로컬 파일시스템에 상태 저장 |

### Pulumi Cloud vs DIY 백엔드 비교

| 항목 | DIY 백엔드 (OSS) | Pulumi Cloud |
|------|------------------|--------------|
| 상태 저장 | 객체 스토리지, PostgreSQL, 로컬 파일시스템 | 관리형 트랜잭션 상태 백엔드; Terraform 상태도 저장 가능 |
| 배포 히스토리 | 스택별 체크포인트 히스토리 | 조직 전체 배포 히스토리 |
| 접근 제어 | 직접 관리 (예: 클라우드 IAM) | SAML/SSO 연동 내장 RBAC |
| 비밀 암호화 | 패스프레이즈 또는 자체 관리 KMS 키 | 기본 관리형 암호화; 별도 암호화 서비스 사용 가능 |
| 정책 코드화 | 디스크에 보관, CLI 인수로 전달 | 중앙 관리형 강제 적용, 사전 구축된 정책 팩 제공 |
| 드리프트 감지 | 수동 `pulumi refresh` 실행 | 예약된 드리프트 감지 및 자동 수정 |
| 상태 잠금 | 기본 파일 기반 잠금 시스템 | 트랜잭션 API 기반 |
| 백업/복구 | 직접 구현 필요 | 자동 처리; 삭제된 스택도 관리자가 복원 가능 |

---

## 백엔드 로그인/로그아웃

### 로그인

```sh
# Pulumi Cloud (기본)
pulumi login

# 특정 백엔드 URL 지정
pulumi login <backend-url>

# 로컬 파일시스템
pulumi login --local
```

로그인 시 액세스 토큰을 입력하거나 `<ENTER>` 키를 눌러 브라우저를 통해 인증할 수 있다.

### 환경 변수 또는 프로젝트 설정으로 백엔드 지정

`PULUMI_BACKEND_URL` 환경 변수를 설정하거나 `Pulumi.yaml`에 `backend` 속성을 추가할 수 있다:

```yaml
# Pulumi.yaml
backend:
  url: <backend-url>
```

### 로그아웃

```sh
# 현재 백엔드에서 로그아웃
pulumi logout

# 모든 백엔드의 자격 증명 제거
pulumi logout --all
```

자격 증명은 `~/.pulumi/credentials.json`에 저장된다.

### 현재 로그인 상태 확인

```sh
pulumi whoami -v
# User: <your-username>
# Backend URL: https://app.pulumi.com/<your-username>
```

---

## Pulumi Cloud 백엔드

`pulumi login`을 인수 없이 실행하면 기본 Pulumi Cloud 백엔드에 로그인된다. 모든 원격 통신은 TLS로 이루어지고 데이터는 저장 시 암호화(at rest)된다.

```sh
# Pulumi Cloud (호스팅 서비스)
pulumi login

# 자체 호스팅 Pulumi Cloud
pulumi login https://pulumi.acmecorp.com
```

Pulumi Cloud은 클라우드 자격 증명을 필요로 하지 않는 구조로 설계되었다. 자세한 내용은 [Pulumi Cloud architecture](https://www.pulumi.com/docs/iac/guides/basics/how-pulumi-works/#pulumi-cloud-architecture)를 참조.

---

## DIY 백엔드 설정

DIY 백엔드는 상태를 사용자가 관리하는 객체 스토리지 또는 로컬 머신에 저장한다. 기본 파일 기반 잠금 시스템이 모든 DIY 백엔드에 활성화되어 있다.

### DIY 백엔드의 디렉터리 구조

체크포인트 파일은 상대 경로 `.pulumi` 디렉터리에 저장된다:

| 경로 | 설명 |
|------|------|
| `.pulumi/meta.yaml` | 백엔드 자체의 메타데이터 (스택 정보는 포함하지 않음) |
| `.pulumi/stacks/` | 각 스택의 활성 상태 파일 (예: `dev.json` 또는 `proj/dev.json`) |
| `.pulumi/locks/` | Pulumi 작업이 수행 중인 스택의 잠금 파일 |
| `.pulumi/history/` | 각 스택의 히스토리 파일 (타임스탬프 포함) |

### 백엔드별 설정 방법

| 백엔드 | 로그인 명령어 | 인증 방식 | 비고 |
|--------|-------------|-----------|------|
| **AWS S3** | `pulumi login 's3://<bucket-name>'` | [AWS Session](https://docs.aws.amazon.com/sdk-for-go/api/aws/session/) (환경 변수, 프로파일 등) | S3 호환 서버(Minio, Ceph, SeaweedFS) 지원. `region`, `endpoint`, `disableSSL`, `s3ForcePathStyle` 쿼리스트링 사용 가능 |
| **Azure Blob** | `pulumi login azblob://<container-path>` | `AZURE_STORAGE_KEY`, `AZURE_STORAGE_SAS_TOKEN`, 또는 `DefaultAzureCredential` (관리 ID, 서비스 주체 등) | `storage_account` 쿼리스트링으로 스토리지 계정 지정 가능 (CLI v3.41.1+). `Storage Blob Data Contributor` 역할 필요 |
| **Google Cloud Storage** | `pulumi login gs://<bucket-path>` | [Application Default Credentials](https://cloud.google.com/docs/authentication/production) | |
| **PostgreSQL** | `pulumi login 'postgres://<username>:@<hostname>:/<database>'` | 연결 문자열에 포함 | 자격 증명을 명령어에 직접 포함하지 말고 환경 변수 등 사용 권장 |
| **로컬 파일시스템** | `pulumi login --local` 또는 `pulumi login file://<path>` | 로컬 파일 권한 | 기본 경로는 `~/.pulumi`. 상대 경로는 현재 작업 디렉터리 기준 |

### AWS S3 상세 설정

```sh
# 기본
pulumi login 's3://<bucket-name>'

# 리전 및 프로파일 지정 (CLI v3.33.1+)
pulumi login 's3://<bucket-name>?region=us-east-1&awssdk=v2&profile=<profile>'

# S3 호환 서버 (Minio 등)
pulumi login 's3://<bucket-name>?endpoint=my.minio.local:8080&disableSSL=true&s3ForcePathStyle=true'

# 버킷 내 폴더 경로 지정
pulumi login 's3://my-bucket/app/project1'
```

### Azure Blob Storage 상세 설정

```sh
# 기본
pulumi login azblob://<container-path>

# 스토리지 계정 직접 지정 (CLI v3.41.1+)
pulumi login 'azblob://<container-path>?storage_account=<account_name>'
```

인증 환경 변수:

| 환경 변수 | 설명 |
|-----------|------|
| `AZURE_STORAGE_ACCOUNT` | (필수) Azure 스토리지 계정 이름 |
| `AZURE_STORAGE_KEY` | 스토리지 계정 액세스 키 |
| `AZURE_STORAGE_SAS_TOKEN` | 공유 액세스 서명 토큰 |
| `AZURE_TENANT_ID` | 서비스 주체 인증용 테넌트 ID |
| `AZURE_CLIENT_ID` | 서비스 주체 인증용 클라이언트 ID |
| `AZURE_CLIENT_SECRET` | 서비스 주체 인증용 클라이언트 시크릿 |

> DIY Azure 백엔드는 Pulumi Azure 프로바이더의 인증 메커니즘이 아닌 Azure SDK for Go를 사용하여 인증한다. `ARM_TENANT_ID`, `ARM_CLIENT_ID` 등은 지원되지 않는다.

### 프로젝트 범위 스택 (Scoping)

Pulumi v3.61.0 이전 버전에서는 DIY 백엔드의 스택이 전역 네임스페이스에 배치되었다. v3.61.0 이후에는 Pulumi Cloud 백엔드와 동일하게 프로젝트별로 스택이 범위 지정된다. 기존 DIY 백엔드를 업그레이드하려면 `pulumi state upgrade` 명령어를 사용한다. 이는 단방향 작업으로, 다운그레이드할 수 없다.

---

## 상태 잠금

모든 DIY 백엔드는 기본적으로 **파일 기반 잠금 시스템**을 사용한다. 스택이 Pulumi 작업 중일 때 `.pulumi/locks/` 디렉터리에 잠금 파일이 생성되어 동시 수정을 방지한다.

Pulumi Cloud 백엔드는 **트랜잭션 REST API**를 사용하여 상태를 증분 기록하고 부분 장애(예: 업데이트 중 네트워크 중단)에서도 깔끔하게 복구할 수 있다. blob 스토리지 기반 DIY 백엔드보다 더 강력한 일관성 보장을 제공한다.

---

## 상태 일관성과 새로고침

### 상태 새로고침 개념

Pulumi 상태는 마지막 `pulumi up` 또는 `pulumi refresh` 이후의 인프라 상태를 기록한다. `pulumi preview`나 `pulumi up`을 실행할 때 Pulumi는 이 기록된 상태를 프로그램에 선언된 구성과 비교하여 필요한 변경 사항을 결정한다. 클라우드 제공자의 각 리소스를 직접 조회하지 않는다.

따라서 누군가 Pulumi 외부에서 리소스를 수정한 경우(클라우드 콘솔, 제공자 CLI 등), 그 변경 사항은 Pulumi 상태에 자동으로 반영되지 않는다.

### Pulumi가 자동으로 새로고침하지 않는 이유

| 이유 | 설명 |
|------|------|
| **성능** | 수백~수천 개의 리소스를 가진 대규모 스택에서 모든 리소스의 실시간 상태를 조회하는 것은 상당한 지연 추가 |
| **명시적 제어** | 프로그램을 원하는 상태의 원천(source of truth)으로 취급. 자동 조정은 의도치 않은 변경 보존 위험 |
| **예측 가능성** | 새로고침을 명시적으로 유지하여 각 `pulumi up`이 수행할 작업에 대한 확신 제공 |

### `pulumi refresh` 실행

```sh
# 상태만 새로고침 (인프라 변경 없음)
pulumi refresh
```

이 명령어는 클라우드 제공자에서 스택의 각 리소스를 조회하고 차이점이 있으면 상태 파일을 업데이트한다. 리소스가 Pulumi 외부에서 삭제된 경우 상태에서 제거하고, 속성이 변경된 경우 상태를 일치시킨다.

### 업데이트와 함께 새로고침

```sh
# 상태 새로고침 후 프로그램 적용을 한 번에
pulumi up --refresh

# 새로고침 후 미리보기
pulumi preview --refresh
```

### 자동 드리프트 감지

Pulumi Cloud은 예약된 **드리프트 감지 및 수정** 기능을 제공한다. 구성된 경우 Pulumi Cloud이 주기적으로 스택에 대해 `pulumi refresh`를 실행하고 실제 인프라 상태가 기록된 상태와 다를 경우 알림을 보내거나 자동으로 수정한다.

---

## pulumi state 명령어

`pulumi state` 명령어는 스택 상태를 직접 편집할 때 사용한다. 문제 해결이나 수동 편집이 필요한 고급 시나리오에서 유용하다.

### 명령어 요약

| 명령어 | 설명 | 주요 옵션 |
|--------|------|-----------|
| `pulumi state delete [urn]...` | 상태에서 리소스 삭제 | `--force` (보호된 리소스 강제 삭제), `--all`, `--target-dependents` |
| `pulumi state rename [urn] [new-name]` | 리소스 이름 변경 | `--stack`, `-y` |
| `pulumi state move [urn]...` | 다른 스택으로 리소스 이동 | `--source`, `--dest`, `--include-parents`, `-y` |
| `pulumi state edit` | EDITOR에서 상태 파일 편집 (**EXPERIMENTAL**) | `--stack` |
| `pulumi state protect [urn]...` | 리소스 보호 설정 (삭제 방지) | `--all`, `--stack`, `-y` |
| `pulumi state unprotect [urn]...` | 리소스 보호 해제 | `--all`, `--stack`, `-y` |
| `pulumi state taint [urn]...` | 리소스에 taint 표시 (다음 `pulumi up` 시 삭제 후 재생성) | `--stack`, `-y` |
| `pulumi state untaint [urn]...` | 리소스 taint 해제 | `--stack`, `-y` |
| `pulumi state repair` | 유효하지 않은 상태 파일 복구 (순서 정렬, 누락 참조 제거) | `--stack`, `-y` |
| `pulumi state upgrade` | DIY 백엔드를 최신 버전으로 마이그레이션 (프로젝트 범위 스택으로 업그레이드) | `-y` |

> 모든 명령어에서 리소스는 **URN(Uniform Resource Name)**으로 지정한다. URN 목록을 확인하려면 `pulumi stack --show-urns`를 실행한다. URN은 작은따옴표로 감싸 쉘 해석을 방지해야 한다.

### 사용 예시

```sh
# 리소스 삭제
pulumi state delete 'urn:pulumi:stage::demo::pkg:index:Type::res-a'

# 여러 리소스 동시 삭제
pulumi state delete 'urn:pulumi:stage::demo::pkg:index:Type::res-a' 'urn:pulumi:stage::demo::pkg:index:Type::res-b'

# 리소스 이름 변경
pulumi state rename 'urn:pulumi:stage::demo::eks:index:Cluster$pulumi:providers:kubernetes::eks-provider' new-name-here

# 스택 간 리소스 이동
pulumi state move <urn> --source <source-stack> --dest <dest-stack>

# 모든 리소스 보호 설정
pulumi state protect --all

# taint로 리소스 강제 재생성 표시
pulumi state taint 'urn:pulumi:stage::demo::pkg:index:Type::res-a'
```

> `pulumi state protect`는 저수준 작업이다. 프로그램에서 `protect` 리소스 옵션을 설정하지 않으면 다음 `pulumi up` 시 보호가 해제된다.

---

## 상태 내보내기와 가져오기

### pulumi stack export / import

`pulumi stack export`와 `pulumi stack import` 명령어는 스택 상태를 내보내고 가져오는 데 사용한다.

```sh
# 상태를 파일로 내보내기
pulumi stack export --show-secrets --file my-stack.stack.json

# 특정 버전의 상태 내보내기
pulumi stack export --version <version> --file my-stack.stack.json

# 상태 파일 가져오기
pulumi stack import --file my-stack.stack.json
```

| 명령어 | 주요 옵션 | 설명 |
|--------|-----------|------|
| `pulumi stack export` | `--file`, `--show-secrets`, `--version`, `--stack` | 스택 배포 상태를 stdout(또는 파일)으로 내보냄 |
| `pulumi stack import` | `--file`, `--force`, `--stack` | 내보낸 상태 파일을 기존 스택으로 가져옴 |

이 명령어들은 상태 검사, 수동 편집, 실패한 배포로 인한 불일치 수정 등 고급 용도로 사용된다.

---

## 백엔드 간 상태 마이그레이션

한 백엔드에서 다른 백엔드로 스택을 마이그레이션할 수 있다. 예를 들어 DIY 백엔드에서 Pulumi Cloud으로 전환하는 경우가 많다.

스택의 상태에는 백엔드 정보와 암호화 제공자 등의 고유 정보가 포함되므로, 단순히 상태 파일을 복사하는 것만으로는 충분하지 않다. `pulumi stack export`와 `pulumi stack import` 명령어가 필요한 변환을 수행한다.

### 마이그레이션 절차 예시 (DIY -> Pulumi Cloud)

```sh
# 1. 소스 백엔드/스택으로 전환
pulumi login --local
pulumi stack select my-app-production

# 2. 상태를 로컬 파일로 내보내기
pulumi stack export --show-secrets --file my-app-production.stack.json

# 3. 새 백엔드로 로그인
pulumi logout
pulumi login   # 기본값: Pulumi Cloud

# 4. 동일한 이름으로 새 스택 생성
pulumi stack init my-app-production

# 5. 상태 가져오기
pulumi stack import --file my-app-production.stack.json
```

> 마이그레이션 후에도 스택은 동일한 secrets provider를 계속 사용한다. 필요시 별도로 [secrets provider 변경](https://www.pulumi.com/docs/concepts/secrets#changing-the-secrets-provider-for-a-stack)이 가능하다.

---

## 상태 암호화

### 전송 중 암호화

- **Pulumi Cloud**: 모든 원격 통신에 TLS 사용
- **DIY 백엔드**: 스토리지 제공자의 설정에 따름

### 저장 시 암호화 (at rest)

- **Pulumi Cloud**: 저장 시 데이터 암호화 제공
- **DIY 백엔드**: 스토리지 제공자의 암호화 설정에 따름

### 비밀값 (Secrets)

Pulumi secret은 비밀번호, 클라우드 토큰 등 민감한 구성 값을 안전하게 저장한다. secret 값은 스택의 선택한 암호화 제공자로 암호화되며, 상태에서 secret이 사용되는 모든 곳에 암호화가 전이적으로 적용된다.

```sh
# secret 설정
pulumi config set --secret <key> <value>
```

암호화 제공자 옵션:

| 제공자 | 설명 |
|--------|------|
| Pulumi Cloud 기본 | 서버 측 HSM 키 사용 |
| 패스프레이즈 | 사용자 지정 암호구 |
| AWS KMS | Amazon Key Management Service |
| Azure Key Vault | Microsoft Azure 키 자격 증명 모음 |
| Google Cloud KMS | Google Cloud Key Management Service |
| HashiCorp Vault | Vault Transit Secrets Engine |

---

## 상태 복구

### 수동 상태 편집

일반적으로 Pulumi는 상태 관리를 추상화하지만, 치명적인 장애 시나리오나 리소스 추가·삭제·이름 변경 등의 고급 작업에서는 수동 편집이 필요할 수 있다.

```sh
# EDITOR로 상태 편집 (EXPERIMENTAL)
pulumi state edit

# 상태 복구 (잘못된 순서 정렬, 누락된 참조 제거)
pulumi state repair
```

상태 파일은 비교적 이해하기 쉬운 JSON 형식을 사용한다. 정확한 JSON 형식은 문서화되어 있지 않지만 [APIType 소스 코드](https://github.com/pulumi/pulumi/tree/master/sdk/go/common/apitype/)에서 확인할 수 있다.

### 기존 리소스 가져오기

Pulumi 외부에서 생성된 리소스(클라우드 콘솔, CLI, 다른 IaC 도구 등)를 Pulumi로 가져올 수 있다. 리소스 메타데이터가 Pulumi 상태로 가져오기되고, 해당 상태와 일치하는 소스 코드가 선택한 언어로 생성된다.

### DIY 백엔드 문제 해결

`.pulumi/meta.yaml` 읽기 오류는 CLI가 스토리지 제공자에 연결했지만 인증에 실패했거나 구성이 잘못되었음을 의미한다:

| 원인 | 해결 방법 |
|------|-----------|
| 만료된 자격 증명 | 임시 자격 증명(AWS SSO 등) 갱신 후 재시도 |
| 권한 부족 | 읽기/쓰기/삭제 권한 확인 (Azure: `Storage Blob Data Contributor` 역할 필요) |
| 리전 누락 (AWS S3) | `?region=us-east-1` 쿼리스트링 추가 또는 `AWS_REGION` 환경 변수 설정 |
| 인증 환경 변수 누락 | 각 백엔드에 해당하는 환경 변수 확인 |

진단 시 CLI 상세 로깅을 활성화하려면 `-v` 플래그를 사용한다.
