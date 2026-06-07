# Pulumi CLI 레퍼런스

> **원문**
> - https://www.pulumi.com/docs/iac/cli/
> - https://www.pulumi.com/docs/iac/cli/commands/
> - https://www.pulumi.com/docs/iac/cli/environment-variables/
> - https://www.pulumi.com/docs/iac/cli/exit-codes/
> - https://www.pulumi.com/docs/iac/cli/api/
> - https://www.pulumi.com/docs/iac/cli/command-line-completion/
> - https://www.pulumi.com/docs/iac/cli/direct-resource-operations/

Pulumi는 주로 CLI(명령줄 인터페이스)를 통해 제어된다. CLI는 Pulumi Cloud와 연동하여 클라우드 앱 및 인프라에 변경 사항을 배포하며, 팀 내에서 누가 언제 무엇을 업데이트했는지에 대한 이력을 유지한다. CLI는 내부 루프(inner loop) 생산성과 CI/CD 시나리오 모두에 최적화되어 설계되었다.

이 문서는 핵심 명령어의 용법·옵션·예제, 환경변수, 종료 코드, `pulumi api` 서브커맨드, 셸 자동완성, 직접 리소스 작업(`pulumi do`), ESC CLI를 정리한다.

---

## 전체 명령어 목록

| 명령어 | 설명 |
| --- | --- |
| `pulumi about` | Pulumi 환경 정보 출력 |
| `pulumi ai` | Pulumi AI CLI 기본 명령어 |
| `pulumi api` | Pulumi Cloud REST API 엔드포인트 호출 |
| `pulumi cancel` | 스택의 현재 실행 중인 업데이트 취소 |
| `pulumi config` | 구성 관리 |
| `pulumi console` | 현재 스택을 Pulumi Console에서 열기 |
| `pulumi convert` | Pulumi 프로그램을 다른 지원 언어로 변환 |
| `pulumi deployment` | **[EXPERIMENTAL]** Pulumi Cloud 스택 배포 관리 |
| `pulumi destroy` | 스택의 모든 기존 리소스 삭제 |
| `pulumi do` | **[RESEARCH PREVIEW]** 프로젝트·프로그램 없이 직접 클라우드 리소스 작업 수행 |
| `pulumi env` | 환경 관리 |
| `pulumi gen-completion` | 셸 자동완성 스크립트 생성 |
| `pulumi import` | 기존 리소스를 스택으로 임포트 |
| `pulumi insights` | **[EXPERIMENTAL]** Pulumi Insights 리소스 및 계정 관리 |
| `pulumi install` | 현재 프로그램 또는 정책 팩의 패키지·플러그인 설치 |
| `pulumi login` | Pulumi Cloud에 로그인 |
| `pulumi logout` | Pulumi Cloud에서 로그아웃 |
| `pulumi logs` | 스택의 집계 리소스 로그 표시 |
| `pulumi neo` | 로컬 도구 실행으로 Pulumi Neo 에이전트 태스크 시작 |
| `pulumi new` | 새 Pulumi 프로젝트 생성 |
| `pulumi org` | 조직 구성 관리 |
| `pulumi package` | Pulumi 패키지 작업 |
| `pulumi plugin` | 언어 및 리소스 프로바이더 플러그인 관리 |
| `pulumi policy` | 리소스 정책 관리 |
| `pulumi preview` | 스택 리소스 업데이트 미리보기 |
| `pulumi project` | Pulumi 프로젝트 관리 |
| `pulumi refresh` | 스택 리소스 상태 새로고침 |
| `pulumi schema` | 패키지 스키마 분석 |
| `pulumi stack` | 스택 관리 및 상태 보기 |
| `pulumi state` | 현재 스택의 상태 편집 |
| `pulumi template` | Pulumi 템플릿 작업 |
| `pulumi up` | 스택의 리소스 생성 또는 업데이트 |
| `pulumi version` | Pulumi 버전 번호 출력 |
| `pulumi watch` | 스택의 리소스를 지속적으로 업데이트 |
| `pulumi whoami` | 현재 로그인한 사용자 표시 |

---

## 글로벌 옵션

모든 `pulumi` 명령어에 공통으로 적용되는 옵션이다.

| 옵션 | 설명 |
| --- | --- |
| `--color string` | 출력 색상. `always`, `never`, `raw`, `auto` (기본값 `auto`) |
| `-C, --cwd string` | 지정한 디렉터리에서 실행한 것처럼 Pulumi 실행 |
| `--disable-integrity-checking` | 체크포인트 파일 무결성 검사 비활성화 |
| `-e, --emoji` | 출력에 이모지 활성화 |
| `-Q, --fully-qualify-stack-names` | 정규화된 스택 이름 표시 |
| `-h, --help` | 도움말 표시 |
| `--logflow` | 로그 설정을 하위 프로세스(플러그인 등)로 전달 |
| `--logtostderr` | 파일 대신 stderr에 로그 출력 |
| `--memprofilerate int` | 메모리 할당 프로파일 정밀도 설정 |
| `--non-interactive` | 모든 명령어의 대화형 모드 비활성화 |
| `--otel-traces string` | OpenTelemetry 트레이스를 지정 엔드포인트로 내보내기. `file://` (로컬 JSON), `grpc://` (원격 수집기) |
| `--profiling string` | CPU/메모리 프로파일 및 실행 트레이스를 `[filename].[pid].{cpu,mem,trace}`로 출력 |
| `--tracing file:` | 지정 엔드포인트로 트레이싱 출력. `file:` 스킴으로 로컬 파일 작성 |
| `-v, --verbose int` | 상세 로깅 활성화 (예: `v=3`). 3 초과 시 매우 상세함 |

---

## 핵심 명령어 상세

---

### pulumi new

새 Pulumi 프로젝트와 스택을 템플릿에서 생성한다. 템플릿 이름(예: `aws-typescript`, `azure-python`)을 지정하면 해당 템플릿으로 프로젝트를 생성하고, 생략하면 대화형으로 템플릿 목록이 표시된다. 로컬 템플릿 경로도 지정할 수 있다.

```
pulumi new [template|url] [flags]
```

| 옵션 | 설명 |
| --- | --- |
| `--ai string` | Pulumi AI에 전달할 프롬프트 |
| `-c, --config stringArray` | 저장할 구성 값 |
| `--config-path` | 구성 키에 맵/리스트 내 속성 경로 포함 |
| `-d, --description string` | 프로젝트 설명 |
| `--dir string` | 생성할 프로젝트 위치 (기본값: 현재 디렉터리) |
| `-f, --force` | 기존 파일 변경 여부와 무관하게 강제 생성 |
| `-g, --generate-only` | 프로젝트만 생성 (스택 생성·구성 저장·의존성 설치 생략) |
| `--language pulumiAILanguage` | Pulumi AI 사용 언어 (`TypeScript`, `JavaScript`, `Python`, `Go`, `C#`, `Java`, `YAML`) |
| `-l, --list-templates` | 로컬에 설치된 템플릿 목록 출력 후 종료 |
| `-n, --name string` | 프로젝트 이름 |
| `-o, --offline` | 네트워크 요청 없이 로컬 캐시된 템플릿 사용 |
| `--runtime-options strings` | 언어 런타임에 전달할 추가 옵션 (형식: `key1=value1,key2=value2`) |
| `--secrets-provider string` | 시크릿 암호화/복호화 프로바이더 (`default`, `passphrase`, `awskms`, `azurekeyvault`, `gcpkms`, `hashivault`, 기본값 `default`) |
| `-s, --stack string` | 스택 이름 |
| `-t, --template-mode` | 템플릿 모드로 실행. AI 또는 템플릿 기능에 대한 프롬프트 건너뜀 |
| `-y, --yes` | 프롬프트 건너뛰고 기본값으로 진행 |

**예제**

```bash
# 대화형으로 AWS TypeScript 프로젝트 생성
pulumi new aws-typescript

# Pulumi AI로 프로젝트 생성
pulumi new --ai "S3 정적 웹사이트" --language TypeScript

# Git 저장소에서 템플릿으로 프로젝트 생성
pulumi new https://github.com/<USER>/<REPO>

# 비밀번호 기반 시크릿 프로바이더 사용
pulumi new --secrets-provider=passphrase
```

---

### pulumi login

Pulumi Cloud 또는 자체 호스팅 백엔드에 로그인한다. `pulumi login`만 실행하면 관리형 Pulumi Cloud에 로그인하며, 액세스 토큰을 입력하라는 프롬프트가 표시된다. `PULUMI_ACCESS_TOKEN` 환경변수로 스크립트에서 인증할 수 있다.

```
pulumi login [url] [flags]
```

| 옵션 | 설명 |
| --- | --- |
| `-c, --cloud-url string` | Pulumi Cloud API URL (예: `https://api.pulumi.com`) |
| `--default-org string` | 로그인 시 기본 조직 설정 |
| `--insecure` | SSL 사용 시 안전하지 않은 서버 연결 허용 |
| `--interactive` | 알려진 계정 기반 대화형 로그인 옵션 표시 |
| `-l, --local` | 로컬 전용 모드 (`file://~`의 별칭) |
| `--oidc-expiration string` | 클라우드 백엔드 액세스 토큰 만료 기간 (예: `15m`, `24h`) |
| `--oidc-org string` | OIDC 토큰 교환 대상 조직 |
| `--oidc-team string` | 팀 토큰 교환 시 팀 지정 |
| `--oidc-token string` | 클라우드 백엔드 액세스 토큰으로 교환할 OIDC 토큰. 원시 토큰 또는 `file://` 접두사 파일 경로 |
| `--oidc-user string` | 개인 토큰 교환 시 사용자 지정 |

**예제**

```bash
# 관리형 Pulumi Cloud에 로그인
pulumi login

# 자체 호스팅 Pulumi Cloud에 로그인
pulumi login https://api.pulumi.acmecorp.com

# 로컬 파일시스템 백엔드 사용
pulumi login --local

# 클라우드 오브젝트 스토리지 백엔드 사용
pulumi login s3://my-pulumi-state-bucket        # AWS S3
pulumi login gs://my-pulumi-state-bucket         # GCP GCS
pulumi login azblob://my-pulumi-state-bucket     # Azure Blob
pulumi login postgres://username:password@hostname:5432/database  # PostgreSQL

# 환경변수로 스크립트에서 인증
export PULUMI_ACCESS_TOKEN="<YOUR_ACCESS_TOKEN>"
pulumi login
```

---

### pulumi stack

스택을 관리하고 상태를 확인한다. 스택은 이름이 지정된 업데이트 대상이며, 단일 프로젝트에 여러 스택이 있을 수 있다. 각 스택은 자체 구성 및 업데이트 이력을 가진다.

```
pulumi stack [flags]
```

| 옵션 | 설명 |
| --- | --- |
| `--output string` | 출력 형식: `default` (사람이 읽기 쉬운 형태) 또는 `json` (기본값 `default`) |
| `-i, --show-ids` | 각 리소스의 프로바이더 할당 고유 ID 표시 |
| `--show-name` | 스택 이름만 표시 |
| `--show-secrets` | 시크릿으로 표시된 스택 출력을 평문으로 표시 |
| `-u, --show-urns` | 각 리소스의 Pulumi 할당 전역 고유 URN 표시 |
| `-s, --stack string` | 대상 스택 이름 (기본값: 현재 스택) |

**주요 서브커맨드**

| 서브커맨드 | 설명 |
| --- | --- |
| `pulumi stack ls` | 스택 목록 |
| `pulumi stack new` | 빈 스택 생성 |
| `pulumi stack select` | 현재 스택 전환 |
| `pulumi stack rm` | 스택 및 구성 제거 |
| `pulumi stack output` | 스택 출력 속성 표시 |
| `pulumi stack history` | 스택 업데이트 이력 |
| `pulumi stack export` | 스택 배포 상태를 stdout으로 내보내기 |
| `pulumi stack get` | **[EXPERIMENTAL]** 스택의 상세 정보 조회 (`pulumi stack --output=json`의 편의 별칭). 조직·프로젝트·스택 이름, 현재 버전, 태그, 활성 업데이트 작업, 로컬 리소스 스냅샷 등을 반환 |
| `pulumi stack import` | stdin에서 배포 상태를 스택으로 가져오기 |
| `pulumi stack rename` | 스택 이름 변경 |
| `pulumi stack graph` | 스택 의존성 그래프를 파일로 내보내기 |
| `pulumi stack tag` | 스택 태그 관리 |
| `pulumi stack change-secrets-provider` | 스택의 시크릿 프로바이더 변경 |
| `pulumi stack drift` | **[EXPERIMENTAL]** 스택 드리프트 감지 결과 검사 |
| `pulumi stack schedule` | **[EXPERIMENTAL]** 스택 예약 배포 액션 관리 |
| `pulumi stack webhook` | **[EXPERIMENTAL]** 스택 웹훅 관리 |
| `pulumi stack unselect` | 워크스페이스의 스택 선택 해제 |

**예제**

```bash
# 현재 스택 정보 확인
pulumi stack

# 스택 목록 조회
pulumi stack ls

# 새 스택 생성
pulumi stack new dev

# 스택 전환
pulumi stack select prod

# 스택 삭제
pulumi stack rm dev

# 스택 출력 확인 (JSON)
pulumi stack output --json
```

---

### pulumi config

스택 구성을 관리한다. 구성 값(key, region 등)을 설정, 조회, 삭제할 수 있다.

```
pulumi config [flags]
```

| 옵션 | 설명 |
| --- | --- |
| `--config-file string` | 지정한 파일의 구성 값 사용 (자동 감지 대신) |
| `-j, --json` | JSON 형식 출력 |
| `--open` | 스택 구성에 나열된 환경 열기 및 해석. `--show-secrets` 설정 시 기본 `true`, 아니면 `false` |
| `--show-secrets` | 시크릿 값을 마스킹 대신 평문으로 표시 |
| `-s, --stack string` | 대상 스택 이름 |

**주요 서브커맨드**

| 서브커맨드 | 설명 |
| --- | --- |
| `pulumi config set` | 구성 값 설정 |
| `pulumi config set-all` | 여러 구성 값 동시 설정 |
| `pulumi config get <key>` | 특정 구성 값 조회 |
| `pulumi config rm` | 구성 값 제거 |
| `pulumi config rm-all` | 여러 구성 값 동시 제거 |
| `pulumi config cp` | 다른 스택으로 구성 복사 |
| `pulumi config env` | 스택의 ESC 환경 관리 |
| `pulumi config refresh` | 스택의 최신 배포 기준으로 로컬 구성 업데이트 |

**`pulumi config set` 옵션**

| 옵션 | 설명 |
| --- | --- |
| `--path` | 구성 키에 맵/리스트 내 속성 경로 포함 |
| `--plaintext` | 값을 평문(암호화하지 않음)으로 저장 |
| `--secret` | 값을 암호화하여 시크릿으로 저장 |
| `--type string` | 값의 타입 지정. 허용값: `string`, `bool`, `int`, `float` |

**`pulumi config set-all` 옵션**

| 옵션 | 설명 |
| --- | --- |
| `--plaintext stringArray` | 평문(암호화하지 않음)으로 저장할 `key=value` 쌍 |
| `--secret stringArray` | 암호화하여 저장할 `key=value` 쌍 |
| `--json string` | `pulumi config --json` 형식의 JSON 문자열에서 값 읽기. `--plaintext`, `--secret`, `--path`와 함께 사용 불가 |
| `--path` | 키를 맵/리스트 내 경로로 파싱 |

**예제**

```bash
# 구성 값 설정
pulumi config set aws:region us-west-2

# 시크릿 구성 설정
pulumi config set dbPassword <YOUR_SECRET> --secret

# 구성 목록 확인
pulumi config

# 특정 키 값 조회
pulumi config get aws:region

# JSON으로 전체 구성 조회
pulumi config --json
```

---

### pulumi up

스택의 리소스를 생성하거나 업데이트한다. 현재 Pulumi 프로그램을 실행하여 리소스 그래프를 생성하고, 이를 기존 상태와 비교하여 생성·읽기·업데이트·삭제 작업을 결정한다. 완료 후 스택의 새 상태를 트랜잭션 스냅샷으로 기록한다.

```
pulumi up [template|url] [flags]
```

| 옵션 | 설명 |
| --- | --- |
| `--attach-debugger stringArray[=program]` | 프로그램 및 소스 기반 플러그인에 디버거 연결. `program`, `plugins`, `plugin:<name>`, `all`로 제한 가능 |
| `-c, --config stringArray` | 업데이트 중 사용할 구성 값 (스택 구성 파일에 저장) |
| `--config-file string` | 지정한 파일의 구성 값 사용 |
| `--config-path` | 구성 키에 맵/리스트 내 속성 경로 포함 |
| `--continue-on-error` | 오류 발생 시에도 리소스 업데이트 계속 (`PULUMI_CONTINUE_ON_ERROR` 환경변수로도 설정 가능) |
| `-d, --debug` | 리소스 작업 중 상세 디버깅 출력 |
| `--diff` | 전체 변경 사항을 rich diff로 표시 |
| `--exclude stringArray` | 무시할 리소스 URN. 와일드카드(`*`, `**`) 지원 |
| `--exclude-dependents` | `--exclude`에 명시되지 않은 종속 리소스도 무시 |
| `--expect-no-changes` | 변경 발생 시 오류 반환 (업데이트 적용 후 검사) |
| `-j, --json` | 업데이트 diff, 작업, 전체 출력을 JSON으로 직렬화 |
| `-m, --message string` | 업데이트 작업에 연결할 메시지 |
| `--neo` | Neo(AI 어시스턴트)를 활성화하여 CLI 경험 개선 (`PULUMI_NEO` 환경변수로도 설정 가능) |
| `--neo-task-on-failure` | 작업 실패 시 Neo 디버그 태스크 자동 시작 |
| `-p, --parallel int32` | 병렬로 실행할 리소스 작업 수 (기본값 16) |
| `--plan string` | **[EXPERIMENTAL]** 업데이트에 사용할 계획 파일 경로 |
| `--policy-pack strings` | 업데이트의 일부로 실행할 정책 팩 |
| `--policy-pack-config strings` | 정책 팩 설정 JSON 파일 경로 (해당 `--policy-pack` 플래그와 대응) |
| `-r, --refresh` | 업데이트 전 스택 리소스 상태 새로고침 |
| `--remote` | **[EXPERIMENTAL]** 원격으로 작업 실행 |
| `--remote-agent-pool-id string` | 원격 배포에 사용할 에이전트 풀 ID |
| `--remote-env stringArray` | 원격 배포에 전달할 환경변수 (`키=값` 형식, 반복 가능) |
| `--remote-env-secret stringArray` | 원격 배포에 전달할 시크릿 환경변수 (`키=값` 형식, 반복 가능) |
| `--remote-executor-image string` | 원격 배포에 사용할 실행기(executor) 컨테이너 이미지 |
| `--remote-executor-image-password string` | 원격 실행기 이미지 레지스트리 비밀번호 |
| `--remote-executor-image-username string` | 원격 실행기 이미지 레지스트리 사용자명 |
| `--remote-git-auth-access-token string` | 원격 배포 Git 인증용 액세스 토큰 |
| `--remote-git-auth-password string` | 원격 배포 Git 인증용 비밀번호 |
| `--remote-git-auth-ssh-private-key string` | 원격 배포 Git 인증용 SSH 개인 키 |
| `--remote-git-auth-ssh-private-key-path string` | 원격 배포 Git 인증용 SSH 개인 키 파일 경로 |
| `--remote-git-auth-username string` | 원격 배포 Git 인증용 사용자명 |
| `--remote-git-branch string` | 원격 배포에 사용할 Git 브랜치 |
| `--remote-git-commit string` | 원격 배포에 사용할 Git 커밋 해시 |
| `--remote-git-repo-dir string` | 원격 배포에서 Git 저장소 내 작업 디렉터리 경로 |
| `--remote-inherit-settings` | 원격 배포에서 조직/스택 설정 상속 |
| `--remote-pre-run-command stringArray` | 원격 배포 실행 전에 실행할 명령어 (반복 가능) |
| `--remote-skip-install-dependencies` | 원격 배포에서 의존성 설치 건너뛰기 |
| `--replace stringArray` | 교체할 리소스 URN. 와일드카드 지원 |
| `--run-program` | `--refresh` 설정 시 프로그램을 실행하여 최신 상태 확인 |
| `--secrets-provider string` | 시크릿 암호화/복호화 프로바이더. 템플릿에서 새 스택 생성 시에만 사용 (기본값 `default`) |
| `--show-config` | 구성 키와 변수 표시 |
| `--show-full-output` | 입력 및 출력의 전체 길이 표시 |
| `--show-policy-remediations` | 리소스별 정책 수정(remediation) 상세 표시 |
| `--show-reads` | 직접 관리되는 리소스와 함께 읽기(read) 리소스 표시 |
| `--show-replacement-steps` | 리소스 교체 생성 및 삭제를 단일 단계 대신 상세 단계로 표시 |
| `--show-sames` | 변경되지 않은 리소스도 함께 표시 |
| `--show-secrets` | 시크릿 출력을 CLI에 평문으로 표시 |
| `--skip-plugin-pre-install` | 사전 프로바이더 플러그인 설치 건너뛰기. 누락된 플러그인은 엔진에서 지연 설치 |
| `-f, --skip-preview` | 미리보기 없이 바로 업데이트 수행 |
| `-s, --stack string` | 대상 스택 이름 (기본값: 현재 스택) |
| `--strict` | **[EXPERIMENTAL]** strict plan 동작 활성화. `--skip-preview`와 함께 사용 불가 |
| `--suppress-outputs` | 스택 출력 표시 억제 |
| `--suppress-permalink` | 상태 퍼마링크 표시 억제 |
| `--suppress-progress` | 주기적 진행 점(progress dots) 표시 억제 |
| `--suppress-stream-logs` | **[EXPERIMENTAL]** 배포 작업의 로그 스트리밍 억제 |
| `-t, --target stringArray` | 업데이트할 리소스 URN. 와일드카드 지원 |
| `--target-dependents` | `--target`에 명시되지 않은 종속 리소스도 업데이트 |
| `--target-replace stringArray` | 교체할 리소스 URN (`--target urn --replace urn`의 약식) |
| `--urns` | 짧은 리소스 이름 대신 전체 URN 표시 |
| `-y, --yes` | 미리보기 후 자동 승인 및 업데이트 수행 |

**예제**

```bash
# 대화형 미리보기 후 배포
pulumi up

# 자동 승인하여 배포
pulumi up --yes

# JSON 출력으로 배포
pulumi up --json

# 특정 스택에 배포
pulumi up --stack prod

# 특정 리소스만 업데이트
pulumi up --target "urn:pulumi:dev::myapp::aws:s3/bucket:Bucket::my-bucket"

# 배포 전 상태 새로고침
pulumi up --refresh --yes

# 메시지와 함께 배포
pulumi up --yes -m "v1.2.3 배포"
```

---

### pulumi preview

스택 리소스의 업데이트 미리보기를 표시한다. Pulumi 프로그램을 실행하여 새 desired state를 계산하고 기존 상태와 비교하지만, 실제 변경은 수행하지 않는다.

```
pulumi preview [url] [flags]
```

| 옵션 | 설명 |
| --- | --- |
| `-c, --config stringArray` | 미리보기에 사용할 구성 값 |
| `--diff` | 전체 변경 사항을 rich diff로 표시 |
| `-x, --exclude stringArray` | 무시할 리소스 URN. 와일드카드 지원 |
| `--expect-no-changes` | 변경이 제안되면 오류 반환 |
| `--import-file string` | 미리보기 중 감지된 생성 리소스를 임포트 파일로 저장 |
| `-j, --json` | 미리보기 diff, 작업, 전체 출력을 JSON으로 직렬화 |
| `-m, --message string` | 미리보기 작업에 연결할 메시지 |
| `-p, --parallel int32` | 병렬로 실행할 리소스 작업 수 (기본값 16) |
| `--policy-pack strings` | 정책 팩 실행 |
| `-r, --refresh` | 미리보기 전 스택 리소스 상태 새로고침 |
| `--save-plan string` | **[PREVIEW]** 미리보기에서 제안된 작업을 계획 파일로 저장 |
| `--show-secrets` | 시크릿을 평문으로 표시 |
| `-t, --target stringArray` | 미리보기할 리소스 URN |
| `--target-dependents` | 종속 리소스도 포함 |

**예제**

```bash
# 변경 사항 미리보기
pulumi preview

# diff와 함께 미리보기
pulumi preview --diff

# JSON 출력으로 미리보기
pulumi preview --json

# 변경이 없어야 하는지 검증 (CI에서 유용)
pulumi preview --expect-no-changes
```

---

### pulumi destroy

스택의 모든 기존 리소스를 삭제한다. 스택 자체는 삭제되지 않으며, 스택까지 삭제하려면 `pulumi stack rm` 또는 `--remove` 플래그를 사용해야 한다. 이 명령어는 일반적으로 되돌릴 수 없으므로 주의해서 사용해야 한다.

```
pulumi destroy [url] [flags]
```

| 옵션 | 설명 |
| --- | --- |
| `--continue-on-error` | 오류 발생 시에도 삭제 계속 (`PULUMI_CONTINUE_ON_ERROR`로도 설정 가능) |
| `--exclude stringArray` | 무시할 리소스 URN. 와일드카드 지원 |
| `--exclude-protected` | 보호된 리소스는 삭제하지 않음 |
| `-j, --json` | 삭제 diff, 작업, 전체 출력을 JSON으로 직렬화 |
| `-m, --message string` | 삭제 작업에 연결할 메시지 |
| `-p, --parallel int32` | 병렬로 실행할 리소스 작업 수 (기본값 16) |
| `--preview-only` | 삭제 미리보기만 표시 (실제 삭제는 수행하지 않음) |
| `-r, --refresh` | 삭제 전 스택 리소스 상태 새로고침 |
| `--remove` | 모든 리소스 삭제 후 스택과 구성 파일도 제거 |
| `--run-program` | 프로그램을 실행하여 최신 상태 확인 후 삭제 |
| `-f, --skip-preview` | 미리보기 없이 바로 삭제 |
| `-t, --target stringArray` | 삭제할 리소스 URN. 와일드카드 지원 |
| `--target-dependents` | 종속 리소스도 삭제 |
| `-y, --yes` | 미리보기 후 자동 승인 및 삭제 수행 |

**예제**

```bash
# 대화형으로 리소스 삭제
pulumi destroy

# 자동 승인하여 삭제
pulumi destroy --yes

# 스택까지 함께 삭제
pulumi destroy --yes --remove

# 특정 리소스만 삭제
pulumi destroy --target "urn:pulumi:dev::myapp::aws:s3/bucket:Bucket::my-bucket"

# 삭제 미리보기만
pulumi destroy --preview-only
```

---

### pulumi refresh

스택 리소스의 상태를 실제 클라우드 프로바이더의 상태와 비교하여 동기화한다. 클라우드 측에서 발생한 변경 사항을 현재 스택 상태에 반영한다. 프로그램 텍스트를 그에 맞게 업데이트하지 않으면 후속 업데이트에서 클라우드 프로바이더의 실제 상태와 불일치가 발생할 수 있다.

```
pulumi refresh [url] [flags]
```

| 옵션 | 설명 |
| --- | --- |
| `--clear-pending-creates` | 모든 pending create를 삭제하여 상태에서 제거 |
| `--exclude stringArray` | 새로고침에서 제외할 리소스 URN. 와일드카드 지원 |
| `--expect-no-changes` | 변경 발생 시 오류 반환 |
| `-j, --json` | 새로고침 diff, 작업, 전체 출력을 JSON으로 직렬화 |
| `-m, --message string` | 작업에 연결할 메시지 |
| `-p, --parallel int32` | 병렬로 실행할 리소스 작업 수 (기본값 16) |
| `--preview-only` | 새로고침 미리보기만 표시 |
| `--run-program` | 프로그램을 실행하여 최신 상태 확인 |
| `-f, --skip-preview` | 미리보기 없이 바로 새로고침 수행 |
| `-t, --target stringArray` | 새로고침할 리소스 URN |
| `--target-dependents` | 종속 리소스도 포함 |
| `-y, --yes` | 미리보기 후 자동 승인 |

**예제**

```bash
# 스택 상태 새로고침
pulumi refresh

# 자동 승인하여 새로고침
pulumi refresh --yes

# JSON 출력으로 새로고침 결과 확인
pulumi refresh --json

# 특정 리소스만 새로고침
pulumi refresh --target "urn:pulumi:dev::myapp::aws:ec2/instance:Instance::my-server"
```

---

### pulumi import

Pulumi로 관리되지 않는 기존 리소스를 Pulumi 스택으로 임포트한다. 각 리소스에 대한 정의가 프로젝트 언어로 stdout에 출력되며, 이를 Pulumi 프로그램에 추가해야 한다. 임포트된 리소스는 기본적으로 삭제로부터 보호된다.

```
pulumi import [arg]... [flags]
```

| 옵션 | 설명 |
| --- | --- |
| `-f, --file string` | 임포트할 리소스 목록이 포함된 JSON 파일 경로 |
| `--from string` | 리소스 임포트에 컨버터 호출 |
| `--generate-code` | 임포트된 리소스의 코드 생성 (기본값 `true`) |
| `-j, --json` | 임포트 diff, 작업, 전체 출력을 JSON으로 직렬화 |
| `-o, --out string` | 생성된 리소스 선언을 저장할 파일 경로 |
| `-p, --parallel int32` | 병렬로 실행할 리소스 작업 수 (기본값 16) |
| `--parent string` | 부모 리소스의 이름과 URN (`name=urn` 형식) |
| `--preview-only` | 임포트 미리보기만 표시 |
| `--properties strings` | 임포트에 사용할 속성 이름 (`name1,name2` 형식) |
| `--protect` | 삭제 보호 활성화 상태로 임포트 (기본값 `true`) |
| `--provider string` | 임포트에 사용할 프로바이더의 이름과 URN (`name=urn` 형식) |

**예제**

```bash
# 단일 리소스 임포트
pulumi import 'aws:iam/user:User' my-user <USER_NAME>

# 보호 없이 임포트
pulumi import 'aws:s3/bucket:Bucket' my-bucket <BUCKET_NAME> --protect=false

# JSON 파일로 다중 리소스 임포트
pulumi import --file import.json

# 부모 및 프로바이더 지정
pulumi import 'aws:iam/user:User' name id --parent 'parent=<URN>' --provider 'admin=<URN>'
```

임포트 JSON 파일 형식:

```json
{
  "resources": [
    {
      "type": "aws:ec2/vpc:Vpc",
      "name": "application-vpc",
      "id": "vpc-0ad77710973388316"
    }
  ]
}
```

`pulumi preview --import-file import.json`을 실행하면 미리보기에서 생성이 필요한 리소스의 임포트 파일을 생성할 수 있다.

---

## 환경변수

### 핵심 환경변수

| 환경변수 | 설명 | 예시 |
| --- | --- | --- |
| `PULUMI_ACCESS_TOKEN` | Pulumi Cloud 인증용 액세스 토큰. `pulumi login` 시 토큰 프롬프트 우회 | `PULUMI_ACCESS_TOKEN="<YOUR_TOKEN>"` |
| `PULUMI_BACKEND_URL` | 기본 백엔드 대신 사용할 백엔드 URL 지정 | `PULUMI_BACKEND_URL="s3://<BUCKET>"` |
| `PULUMI_STACK` | 선택된 스택을 오버라이드. 우선순위: `--stack` 플래그 > `PULUMI_STACK` > `pulumi stack select` | `PULUMI_STACK="prod"` |
| `PULUMI_CONFIG` | 단위 테스트용 구성(JSON 형식). 일반 작업(`up`, `preview` 등)에서는 무시됨 | `PULUMI_CONFIG='{"key":"val"}'` |
| `PULUMI_CONFIG_PASSPHRASE` | 시크릿 암호화/복호화에 사용되는 passphrase. `AES-256-GCM` 기반 | `PULUMI_CONFIG_PASSPHRASE="<YOUR_PASSPHRASE>"` |
| `PULUMI_CONFIG_PASSPHRASE_FILE` | passphrase가 저장된 파일 경로 (`PULUMI_CONFIG_PASSPHRASE`의 대안) | `PULUMI_CONFIG_PASSPHRASE_FILE="/tmp/passphrase.txt"` |
| `PULUMI_HOME` | Pulumi CLI가 플러그인, 워크스페이스, 템플릿, 자격 증명 파일을 저장하는 경로 (기본값 `~/.pulumi`) | `PULUMI_HOME="/path/to/artifacts"` |
| `PULUMI_EXPERIMENTAL` | 실험적 옵션 및 명령어 활성화 | `PULUMI_EXPERIMENTAL=true` |

### 실행 제어 환경변수

| 환경변수 | 설명 |
| --- | --- |
| `PULUMI_CONTINUE_ON_ERROR` | 오류 발생 시에도 업데이트/삭제 작업 계속 |
| `PULUMI_PARALLEL` | 병렬로 실행할 리소스 작업 수 (1이면 병렬 없음) |
| `PULUMI_PARALLEL_DIFF` | diff 계산을 병렬로 실행 |
| `PULUMI_SKIP_CONFIRMATIONS` | 비대화형 모드에서 명시적 확인 수행 (v2.0.0 이상) |
| `PULUMI_SKIP_UPDATE_CHECK` | Pulumi 버전 업데이트 확인 건너뛰기 (v0.17.9 이상) |
| `PULUMI_RUN_PROGRAM` | refresh/destroy 작업 시 프로그램 실행 (`--run-program=true`와 동일) |
| `PULUMI_NON_INTERACTIVE` | 모든 명령어의 대화형 모드 비활성화 |

### 플러그인 및 프로바이더 환경변수

| 환경변수 | 설명 |
| --- | --- |
| `PULUMI_DISABLE_AUTOMATIC_PLUGIN_ACQUISITION` | 누락된 플러그인 자동 설치 비활성화 |
| `PULUMI_IGNORE_AMBIENT_PLUGINS` | `$PATH` 검사를 통한 추가 플러그인 발견 비활성화 |
| `PULUMI_PLUGIN_DOWNLOAD_URL_OVERRIDES` | 플러그인 다운로드 URL 오버라이드. 형식: `regexp=URL` (쉼표로 구분) |

### 언어 런타임 환경변수

| 환경변수 | 설명 |
| --- | --- |
| `PULUMI_PREFER_YARN` | Node.js 의존성 설치 시 `npm` 대신 `yarn` 사용 |
| `PULUMI_PYTHON_CMD` | Python 프로그램 실행에 사용할 바이너리 (v0.16.6 이상, 기본값 `python3`) |

### 상태·백엔드 관련 환경변수

| 환경변수 | 설명 |
| --- | --- |
| `PULUMI_SKIP_CHECKPOINTS` | 상태 체크포인트 저장을 건너뛰고 최종 배포만 저장 (v3.40.1 이상, 실험적) |
| `PULUMI_DISABLE_SECRET_CACHE` | 변경되지 않은 스택 시크릿에 대한 캐시 암호화 작업 비활성화 |
| `PULUMI_FALLBACK_TO_STATE_SECRETS_MANAGER` | `true` 시 스택 구성이 누락되면 상태에 저장된 시크릿 매니저를 폴백으로 사용 |
| `PULUMI_DIY_BACKEND_DISABLE_CHECKPOINT_BACKUPS` | 체크포인트 백업을 백업 폴더에 기록하지 않음 |
| `PULUMI_DIY_BACKEND_GZIP` | 상태 파일 작성 시 gzip 압축 활성화 |
| `PULUMI_DIY_BACKEND_LEGACY_LAYOUT` | 새 버킷에 레거시 레이아웃 사용 |
| `PULUMI_DIY_BACKEND_NO_LEGACY_WARNING` | 레거시 스택 파일과 프로젝트 범위 스택 파일 혼합 경고 비활성화 |
| `PULUMI_DIY_BACKEND_PARALLEL` | DIY 백엔드에서 스택/리소스 조회 시 병렬 작업 수 |
| `PULUMI_DIY_BACKEND_RETAIN_CHECKPOINTS` | 모든 체크포인트를 타임스탬프 파일에 복제 |

### 디버깅·개발 환경변수

| 환경변수 | 설명 |
| --- | --- |
| `PULUMI_DEBUG_COMMANDS` | Pulumi 자체 디버깅에 유용한 명령어 나열 |
| `PULUMI_DEBUG_GRPC` | gRPC 내부 디버그 트레이싱 활성화 (로그 파일 경로 지정) |
| `PULUMI_DEBUG_PROMISE_LEAKS` | Promise 누수 시 상세 오류 메시지 표시 (v0.12.2 이상) |
| `PULUMI_DEV` | Pulumi 자체 개발용 기능 활성화 |

### 레거시 호환성 환경변수

| 환경변수 | 설명 |
| --- | --- |
| `PULUMI_ENABLE_LEGACY_APPLY` | v1.0.0-beta1 이상에서 `preview` 시 입력 속성이 누락된 출력 속성으로 전파되는 이전 동작 활성화 |
| `PULUMI_ENABLE_LEGACY_DIFF` | v0.17.23 이상에서 레거시 diff 동작 활성화 |
| `PULUMI_ENABLE_LEGACY_PLUGIN_SEARCH` | v0.16.18 이상에서 레거시 플러그인 로드 동작 활성화 |
| `PULUMI_ENABLE_LEGACY_REFRESH_DIFF` | 레거시 새로고침 diff 동작 사용 (출력 변경만 보고, desired state와의 변경은 계산하지 않음) |
| `PULUMI_DISABLE_PROVIDER_PREVIEW` | 프로바이더 미리보기 비활성화, 이전 보수적 미리보기 동작 사용 |
| `PULUMI_DISABLE_VALIDATION` | 시스템 입력 형식 검증 비활성화 (현재 스택 이름 검증만 해당) |

### 기타 환경변수

| 환경변수 | 설명 |
| --- | --- |
| `PULUMI_CONSOLE_DOMAIN` | Pulumi Console 링크 생성 시 도메인 오버라이드 |
| `PULUMI_COPILOT` | Neo 도움말 및 링크를 CLI 출력에 표시 |
| `PULUMI_SUPPRESS_COPILOT_LINK` | Neo의 `explainFailure` 링크 표시 억제 |
| `PULUMI_ERROR_ON_DEPENDENCY_CYCLES` | 의존성 순환 감지 시 오류 보고 활성화 |
| `PULUMI_ERROR_OUTPUT_STRING` | Output을 문자열로 변환 시 문자열 반환 대신 오류 발생 |
| `PULUMI_AUTOMATION_API_SKIP_VERSION_CHECK` | Automation API의 최소 CLI 버전 검사 건너뛰기 (권장하지 않음) |
| `PULUMI_GITSSH_PASSPHRASE` | SSH를 사용하는 Git 작업에 사용할 passphrase |
| `NO_COLOR` | 값과 무관하게 터미널 출력에서 색상 ANSI 코드 제거 ([no-color.org](https://no-color.org/) 참조) |

### CLI 인수를 환경변수로 설정

v3.208.0부터 모든 Pulumi CLI 인수를 환경변수로 설정할 수 있다. 명명 규칙은 `PULUMI_OPTION_` + 대문자 스네이크 케이스 인수 이름이다.

| CLI 인수 예시 | 환경변수 예시 |
| --- | --- |
| `pulumi up --parallel 1` | `PULUMI_OPTION_PARALLEL=1 pulumi up` |
| `pulumi up --refresh` | `PULUMI_OPTION_REFRESH=true pulumi up` |
| `pulumi up --yes` | `PULUMI_OPTION_YES=1 pulumi up` |
| `pulumi up --target foo --target bar` | `PULUMI_OPTION_TARGET=foo,bar pulumi up` |

> 불리언 인수는 `true`/`false` 또는 `1`/`0`으로 지정할 수 있다.

---

## 종료 코드

Pulumi CLI는 명령어의 결과를 나타내는 숫자 종료 코드를 반환한다. 스크립트, CI/CD 시스템, 도구에서 이 코드를 사용하여 다양한 유형의 실패를 구분할 수 있다.

> **참고:** 이 페이지에 설명된 글로벌 CLI 종료 코드 매핑은 Pulumi CLI **v3.226.1**에 도입되었다. 이전 버전은 동일한 매핑을 보장하지 않는다.

0은 **성공**, 0이 아닌 값은 **실패**를 의미한다.

| 종료 코드 | 범주 | 발생 시점 |
| ---:| --- | --- |
| 0 | 성공 (Success) | 명령이 성공적으로 완료됨 |
| 1 | 일반 오류 (Generic error) | 더 구체적인 범주에 맞지 않는 실패 |
| 2 | 구성·검증 오류 (Configuration and validation error) | 잘못되거나 누락된 인수, 구성, 스키마 검증 오류. 비대화형 모드에서 `--yes` 등 필수 확인 플래그 미제공 |
| 3 | 인증·권한 오류 (Authentication or authorization error) | 로그인 필요, 자격 증명 누락, 접근 금지 |
| 4 | 리소스·배포 오류 (Resource or deployment error) | 배포, 미리보기, 새로고침, 삭제, 임포트 중 리소스 작업 실패. 프로바이더 및 클라우드 플랫폼 오류 포함 |
| 5 | 정책 위반 (Policy failure) | 프로그램과 프로바이더는 성공했으나 정책 팩 또는 조직 정책이 작업을 차단 |
| 6 | 스택 없음 (Stack not found) | 요청한 스택이 존재하지 않거나, 찾을 수 없거나, 선택된 스택이 없음 |
| 7 | 변경 없음 위반 (No changes) | `--expect-no-changes` 사용 시 변경이 감지되는 등, 변경에 대한 기대가 충족되지 않음 |
| 8 | 실행 취소 (Canceled run) | `pulumi cancel`, 사용자 인터럽트(Ctrl+C), Automation API를 통한 취소로 인해 완료 전 작업이 취소됨 |
| 9 | 시간 초과 (Timeout) | 예상 시간 내에 작업이 완료되지 않음 |
| 255 | 내부 CLI 오류 (Internal CLI error) | 다른 범주에 맞지 않는 Pulumi CLI 내부의 예상치 못한 상태 |

---

## pulumi api 서브커맨드

`pulumi api` 명령어를 사용하면 CLI에서 Pulumi Cloud REST API 엔드포인트를 직접 호출할 수 있다. **[EXPERIMENTAL]** 비대화형으로 실행되어 스크립팅에 안전하며, `pulumi login`에서 이미 사용 중인 자격 증명을 재사용하므로 별도의 토큰 관리가 필요 없다. `gh api`를 모델로 설계되었다. 위치 인수는 경로(템플릿 변수 포함, 예: `/api/orgs/{orgName}/members`), Operation ID(예: `ListOrganizationMembers`), 또는 붙여넣기 가능한 행(예: `GET /api/...`)이 될 수 있다.

### 서브커맨드 구조

| 서브커맨드 | 설명 |
| --- | --- |
| `pulumi api` | 지정된 엔드포인트에 단일 HTTP 요청 발행 |
| `pulumi api list` (별칭: `ls`) | Pulumi Cloud OpenAPI 사양에 노출된 모든 엔드포인트 나열. 기본 TTY 친화적 테이블 출력, `--output=json`으로 안정적인 JSON 봉투 제공 |
| `pulumi api describe` | 단일 작업의 파라미터, 요청 본문, 응답 스키마 출력. 기본 사람이 읽기 쉬운 텍스트, `--output=markdown` 또는 `--output=json` 사용 가능 |

### 인증

`pulumi api`는 Pulumi CLI의 나머지 부분과 동일한 액세스 토큰을 사용한다. 개인, 조직, 팀 액세스 토큰, `PULUMI_ACCESS_TOKEN`, 또는 OIDC 발급 토큰 등 `pulumi login`을 인증하는 모든 것이 API 호출도 인증한다. 인증되지 않은 상태에서 요청하면 종료 코드 `3`과 함께 오류가 반환된다.

### 경로 템플릿 치환

대부분의 Pulumi Cloud 엔드포인트는 URL에 조직, 프로젝트, 스택을 포함한다 (예: `/api/orgs/{orgName}/members`). 템플릿 변수는 다음 순서로 해석된다:

1. URL에 직접 입력된 리터럴 값
2. 일치하는 `-F` 또는 `-f` 필드 (경로에 사용되고 쿼리/본문 파라미터로는 전달되지 않음)
3. 현재 Pulumi 프로젝트 컨텍스트: 선택된 스택의 조직, `Pulumi.yaml`의 프로젝트 이름, 선택된 스택 이름

### 요청 플래그

| 플래그 | 용도 |
| --- | --- |
| `-X` / `--method` | HTTP 메서드. 기본값 `GET`, 본문 필드·`--body`·`--input`이 있으면 `POST` |
| `-F` / `--field` | 형식화된 `key=value`. 숫자, 불리언, `null` 자동 감지; JSON 객체/배열 리터럴 파싱; `@file`은 파일에서, `@-`는 stdin에서 읽기. `GET`/`HEAD`에서는 쿼리 파라미터, 그 외에는 JSON 본문 필드로 전달 |
| `-f` / `--raw-field` | 타입 강제 변환 없는 문자열 `key=value`. 있는 그대로 전송 |
| `-H` / `--header` | 커스텀 HTTP 헤더 `Key: Value` (반복 가능). 사용자 헤더가 기본값(`Accept`, `Content-Type`)보다 우선 |
| `--body` | 인라인 요청 본문. `--input`과 상호 배타적 |
| `--input` | 파일에서 요청 본문 읽기; `-`는 stdin에서 읽기 |
| `--paginate` | 연속 커서를 따라 단일 결합 JSON 봉투를 출력. 최대 1000페이지까지 처리하며, 잘림·네트워크 오류·취소 발생 시 부분 결과라도 플러시하여 호출자가 항상 결과를 받을 수 있음 |
| `-i` / `--include` | 출력에 HTTP 상태 라인 및 응답 헤더 포함 |
| `--silent` | 성공 시 응답 본문 억제 (오류는 여전히 출력됨) |
| `--verbose` | 해석된 요청과 전체 응답을 stderr에 출력 |
| `--dry-run` | 요청을 전송하지 않고 해석된 요청만 출력 |
| `--output` | 콘텐츠 협상 구동. 기본값은 작업의 기본 응답 콘텐츠 타입. `json`, `markdown`, `raw` 사용 가능 |
| `--refresh-spec` | OpenAPI 스펙을 강제로 다시 가져옴 |

### 종료 코드

`pulumi api`는 표준 Pulumi CLI 종료 코드 매핑을 사용한다:

| 종료 코드 | 의미 |
| ---:| --- |
| 0 | 성공 |
| 1 | 호출자 오류: 잘못된 인수, 일치하는 작업 없음, 템플릿 변수 누락, 부분 페이지네이션 |
| 2 | 잘못된 플래그 조합 (예: `--body`와 `--input` 동시 사용) |
| 3 | 인증/권한 실패 (자격 증명 누락, `401`, `403`) |
| 8 | 작업 취소 (`SIGINT` 또는 `SIGTERM`) |
| 255 | 내부 CLI 오류 |

실패 시 stderr에 안정적인 `code` 필드가 포함된 단일 줄 JSON 봉투로 오류가 출력된다.

### OpenAPI 스펙 캐싱

CLI는 `/api/openapi/pulumi-spec.json`에서 OpenAPI 스펙을 한 번 가져와 `$PULUMI_HOME/cloud-api-cache/<host>/spec.json`에 24시간 캐시한다. 캐시가 TTL을 초과하고 새로고침이 실패하면(네트워크 오류, 5xx 응답) 만료된 캐시를 stderr 경고와 함께 반환하여 일시적 중단이 `list` 및 `describe`를 중단시키지 않는다.

### 예제

```bash
# 현재 인증된 사용자 조회
pulumi api /api/user

# URL 경로로 호출 (-F로 템플릿 변수 채움)
pulumi api /api/orgs/{orgName}/members -F orgName=acme

# Operation ID로 호출 (orgName은 현재 프로젝트에서 자동 추출)
pulumi api ListOrganizationMembers

# 프로젝트 컨텍스트 없이 명시적으로 변수 전달
pulumi api GetStack -F orgName=acme -F projectName=web -F stackName=prod

# POST로 리소스 생성 (본문 필드 자동 감지)
pulumi api CreateOrgToken -F orgName=acme \
  -F name=ci-bot -F description="CI deploy token" \
  -F admin=false -F expires=0

# 중첩 JSON 본문 전송
pulumi api CreateStack -F orgName=acme -F projectName=web \
  -F stackName=prod -F 'tags={"env":"prod","team":"platform"}'

# 인라인 본문으로 요청
pulumi api UpdateStackTags -F orgName=acme -F projectName=web -F stackName=prod \
  --body '{"env":"prod","team":"platform"}'

# 파일에서 본문 읽기
pulumi api UpdateStackTags --input ./tags.json

# stdin에서 본문 읽기
cat tags.json | pulumi api UpdateStackTags --input -

# jq로 JSON 응답 필터링
pulumi api /api/user --output=json | jq '.githubLogin'

# 페이지네이션
pulumi api ListUserStacks --paginate | jq -r '.stacks[].stackName'

# API 탐색
pulumi api list --output=json | jq '.operations[] | select(.tag == "Stacks")'

# 특정 작업 스키마 확인
pulumi api describe CreateOrgToken --output=markdown

# dry-run으로 요청 미리보기
pulumi api CreateOrgToken -F orgName=acme \
  -F name=ci-bot -F description="CI" -F admin=false -F expires=0 --dry-run
```

---

## 셸 자동완성

> **원문**: https://www.pulumi.com/docs/iac/cli/command-line-completion/

Pulumi CLI는 Bash, Zsh, Fish용 명령줄 자동완성 스크립트를 생성할 수 있다. 모든 명령어, 서브커맨드, 플래그에 대해 탭 완성이 제공된다.

```bash
pulumi gen-completion <shell>
```

### Bash

bash-completion이 설치되어 있어야 한다.

- **Linux**: 대부분의 배포판에 기본 포함
- **macOS**: `brew install bash-completion`으로 설치

```bash
# 자동완성 스크립트 저장
## Linux
pulumi gen-completion bash > /etc/bash_completion.d/pulumi
## macOS
pulumi gen-completion bash > /usr/local/etc/bash_completion.d/pulumi

# ~/.bash_profile에 추가
## Linux
if [ -f /etc/bash_completion ]; then
  . /etc/bash_completion
fi
## macOS
if [ -f /usr/local/etc/bash_completion ]; then
  . /usr/local/etc/bash_completion
fi

# 적용 (새 터미널을 열거나 프로파일 다시 로드)
. ~/.bash_profile
```

### Zsh

```bash
# $fpath 디렉터리 확인
echo $fpath

# 자동완성 스크립트를 $fpath 디렉터리 중 하나에 저장
pulumi gen-completion zsh > "${fpath[1]}/_pulumi"

# 또는 임의 디렉터리 사용 (~/.zsh/completion/)
fpath=(~/.zsh/completion $fpath)

# ~/.zshrc에 compinit 로드
autoload -Uz compinit && compinit -i

# 셸 재시작
exec $SHELL -l
```

### Fish

```bash
# 현재 세션에서 즉시 사용
pulumi gen-completion fish | source

# 영구적으로 사용하려면 파일로 저장
pulumi gen-completion fish > ~/.config/fish/completions/pulumi.fish

# 새 터미널을 열면 적용됨
```

---

## 직접 리소스 작업 (pulumi do)

> **원문**: https://www.pulumi.com/docs/iac/cli/direct-resource-operations/

> **참고**: `pulumi do`는 **research preview** 상태이다. 명령 인터페이스는 피드백에 따라 변경될 수 있다.

`pulumi do` 명령어는 프로젝트, 프로그램 또는 상태 파일 없이 Pulumi CLI를 통해 클라우드 리소스에 대한 직접 작업을 수행한다. 전체 Pulumi 프로바이더 생태계를 CLI로 노출하며, 각 프로바이더의 스키마에서 동적으로 명령어가 생성된다.

### 명령 구문

```bash
# 프로바이더 함수 (읽기 전용)
pulumi do <package:module:function> [flags]

# 리소스 작업 (Create, Read, Patch, Delete)
pulumi do <package:module:type> <operation> [<id>] [flags]
```

명령 트리의 어떤 수준에서든 `--help`를 전달하여 사용 가능한 서브커맨드를 탐색할 수 있다.

### pulumi do vs pulumi up

| 시나리오 | `pulumi do` | `pulumi up` |
| --- | --- | --- |
| 클라우드 API 조회 | O | X |
| 개별 리소스 생성/수정 | O | O |
| 프로바이더 기능 탐색 | O | X |
| 에이전트 기반 임시 작업 | O | 반복 워크플로에 적합 |
| 프로덕션 인프라 관리 | X | O |
| 상태 추적 및 드리프트 감지 | X (상태 비저장) | O |
| 다중 리소스 의존성 그래프 | X | O |
| 정책 강제 및 컴플라이언스 | X | O |
| 반복 가능하고 검토 가능한 배포 | X | O |

### 프로바이더 함수

프로바이더 함수는 Pulumi의 프로바이더 계층을 통해 클라우드 API를 쿼리하는 읽기 전용 작업이다.

```bash
pulumi do <package:module:function> --input-file <path>
```

입력 파일은 함수의 인수를 포함하며, 출력은 stdout으로 JSON이 작성된다.

**예제: VPC 조회**

입력 파일 `query.pcl`:
```
tags = {
    "Name" = "production"
}
```

```bash
pulumi do aws:ec2:getVpc --input-file query.pcl
```

출력:
```json
{
    "arn": "arn:aws:ec2:us-west-2:123456789:vpc/vpc-abc123",
    "cidrBlock": "10.0.0.0/16",
    "id": "vpc-abc123",
    "tags": {"Name": "production"}
}
```

**입력 파일 형식**: PCL(기본값). 최상위 할당이 함수 파라미터에 매핑되며, 실행 전 함수 스키마에 대해 전체 타입 검사가 수행된다.

### 리소스 작업

| 작업 | 설명 | 구문 |
| --- | --- | --- |
| `create` | 새 클라우드 리소스 생성. 확인 프롬프트 후 생성 | `pulumi do <pkg:mod:type> create --input-file <path>` |
| `read` | 클라우드 프로바이더 ID로 기존 리소스의 현재 상태 읽기 | `pulumi do <pkg:mod:type> read <provider-id>` |
| `patch` | 기존 리소스 업데이트. 현재 상태를 읽고 변경 사항을 병합하여 diff 표시 후 확인 프롬프트 | `pulumi do <pkg:mod:type> patch <provider-id> --input-file <path>` |
| `delete` | 리소스 삭제. 확인 프롬프트 후 삭제 | `pulumi do <pkg:mod:type> delete <provider-id>` |

### 플래그

| 플래그 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--input-file` | string | - | 함수 또는 리소스 입력이 포함된 파일 경로 |
| `--input` | - | pcl | 입력 파일 형식 |
| `--provider-file` | string | - | 프로바이더 구성이 포함된 파일 경로 |
| `--provider-format` | string | pcl | 프로바이더 구성 파일 형식 |
| `--dry-run` | bool | false | 미리보기 모드로 실행 (프로바이더가 플레이스홀더 값 반환) |
| `--show-secrets` | bool | false | 출력에 시크릿 값 표시 |
| `--yes` | bool | false | 확인 프롬프트 자동 승인 |

### 출력 형식

모든 `pulumi do` 작업은 stdout에 구조화된 JSON을 작성한다. 진행 메시지와 프롬프트는 stderr로 전송되어 파이핑과 스크립팅이 깔끔하게 이루어진다.

```bash
# 함수 출력을 jq로 파이핑
pulumi do aws:ec2:getVpc --input-file query.pcl | jq '.cidrBlock'

# 리소스 출력을 파일로 리다이렉트 (진행 상태는 stderr에 표시)
pulumi do aws:s3:Bucket read my-bucket > result.json
```

시크릿은 기본적으로 출력에 `[secret]`으로 표시된다. `--show-secrets`로 노출할 수 있다.

### 프로바이더 구성

프로바이더는 작동하기 위해 자격 증명과 구성이 필요하다. `pulumi do`는 다음을 통해 프로바이더 구성을 해석한다:

- **앰비언트 자격 증명**: 셸에 이미 존재하는 환경변수 및 자격 증명 파일 (예: `AWS_ACCESS_KEY_ID`, `~/.aws/credentials`)
- **프로바이더 구성 파일**: `--provider-file` 플래그로 PCL 파일을 통해 프로바이더 구성 제공

```bash
pulumi do aws:ec2:getVpc --input-file query.pcl \
  --provider-file aws-config.pcl
```

---

## pulumi state 명령어 그룹

> **원문**: https://www.pulumi.com/docs/iac/cli/commands/pulumi_state/

현재 스택의 상태를 편집한다. 스택 문제 해결 또는 수동 편집이 필요한 특정 작업에 유용하다.

| 서브커맨드 | 설명 |
| --- | --- |
| `pulumi state delete` | 스택 상태에서 하나 이상의 리소스 삭제 |
| `pulumi state edit` | 편집기(`$EDITOR`)에서 현재 스택의 상태 편집 |
| `pulumi state move` | 리소스를 한 스택에서 다른 스택으로 이동 |
| `pulumi state protect` | 스택 상태의 리소스를 보호 상태로 설정 |
| `pulumi state rename` | 스택 상태의 리소스 이름 변경 |
| `pulumi state repair` | 유효하지 않은 상태 복구 |
| `pulumi state taint` | 스택 상태의 하나 이상의 리소스를 테인트(taint) 처리 |
| `pulumi state unprotect` | 스택 상태의 리소스 보호 해제 |
| `pulumi state untaint` | 스택 상태의 하나 이상의 리소스 테인트 해제 |
| `pulumi state upgrade` | 현재 백엔드를 최신 지원 버전으로 마이그레이션 |

---

## ESC CLI (pulumi esc)

> **원문**: https://www.pulumi.com/docs/esc/cli/commands/

Pulumi ESC(Environment, Secrets, and Configuration)는 별도의 CLI(`esc`)를 통해 환경을 관리한다. ESC CLI >= 0.10.0부터 ESC Projects를 사용할 수 있다.

### 주요 명령어

| 명령어 | 설명 |
| --- | --- |
| `esc env` | 환경 관리 |
| `esc env diff` | 환경 버전 간 차이 표시 |
| `esc env edit` | 환경 정의 편집 |
| `esc env get` | 환경 내 특정 값 조회 |
| `esc env init` | 빈 환경 생성 |
| `esc env ls` | 환경 목록 조회 |
| `esc env rm` | 환경 또는 환경 내 특정 값 제거 |
| `esc env set` | 환경 내 값 설정 |
| `esc env version` | 환경 버전 관리 |
| `esc env version rollback` | 환경 정의를 특정 버전으로 롤백 |
| `esc env version tag` | 태그가 지정된 버전 관리 |
| `esc login` | Pulumi Cloud에 로그인 |
| `esc open` | 지정한 이름의 환경 열기 |
| `esc run` | 지정한 이름의 환경을 열고 명령어 실행 |
| `esc version` | ESC 버전 번호 출력 |
