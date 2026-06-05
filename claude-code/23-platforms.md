# 플랫폼 및 통합

> **출처**
> - https://code.claude.com/docs/en/platforms
> - https://code.claude.com/docs/en/desktop
> - https://code.claude.com/docs/en/desktop-quickstart
> - https://code.claude.com/docs/en/claude-code-on-the-web
> - https://code.claude.com/docs/en/web-quickstart
> - https://code.claude.com/docs/en/remote-control
> - https://code.claude.com/docs/en/chrome
> - https://code.claude.com/docs/en/slack
> - https://code.claude.com/docs/en/voice-dictation
> - https://code.claude.com/docs/en/computer-use

Claude Code는 동일한 엔진을 모든 환경에서 실행하지만, 각 플랫폼은 서로 다른 작업 방식에 맞게 튜닝되어 있다. 이 문서는 Claude Code를 다양한 환경에서 사용하는 방법과 외부 서비스와의 통합 방법을 정리한다.

---

## 플랫폼 비교

| 플랫폼 | 적합한 용도 | 제공 기능 |
| --- | --- | --- |
| **CLI** | 터미널 워크플로우, 스크립팅, 원격 서버 | 전체 기능, Agent SDK, macOS Computer Use(Pro/Max), 서드파티 프로바이더 |
| **Desktop** | 시각적 리뷰, 병렬 세션, 관리형 설정 | Diff 뷰어, 앱 프리뷰, Computer Use 및 Dispatch(Pro/Max) |
| **VS Code** | 터미널 전환 없이 에디터 내 작업 | 인라인 diff, 통합 터미널, 파일 컨텍스트 |
| **JetBrains** | IntelliJ, PyCharm, WebStorm 등에서 작업 | Diff 뷰어, 선택 영역 공유, 터미널 세션 |
| **Web** | 장시간 실행 작업, 오프라인 시에도 계속되는 작업 | Anthropic 관리 클라우드, 연결 끊겨도 지속 |
| **Mobile** | 컴퓨터에서 벗어났을 때 작업 시작 및 모니터링 | iOS/Android Claude 앱의 클라우드 세션, Remote Control, Dispatch |

CLI는 터미널 기반 작업에 가장 완전한 기능을 제공한다. 스크립팅과 Agent SDK는 CLI 전용이다. 서드파티 프로바이더도 VS Code에서 사용 가능하다. Desktop과 IDE 확장은 시각적 리뷰와 타이트한 에디터 통합을 위해 일부 CLI 전용 기능을 교환한다. Web은 Anthropic 클라우드에서 실행되어 연결을 끊어도 작업이 지속된다. Mobile은 클라우드 세션에 대한 thin client이거나 Remote Control을 통해 로컬 세션에 접속하며, Dispatch로 Desktop에 작업을 보낼 수 있다.

동일 프로젝트에서 여러 플랫폼을 혼합 사용할 수 있다. 설정, 프로젝트 메모리, MCP 서버는 로컬 플랫폼 간에 공유된다.

---

## Desktop App

### 개요

Claude Desktop 앱은 세 개의 탭으로 구성된다.

| 탭 | 용도 |
| --- | --- |
| **Chat** | 파일 접근 없는 일반 대화, claude.ai와 유사 |
| **Cowork** | Dispatch 및 장기 에이전트 작업을 수행하는 자율 백그라운드 에이전트, 클라우드 VM에서 자체 환경으로 실행 |
| **Code** | 로컬 파일에 직접 접근하는 인터랙티브 코딩 어시스턴트, 변경 사항을 실시간으로 리뷰 및 승인 |

Windows ARM64의 경우 ARM64 인스톨러를 다운로드해야 한다. Linux에서는 Desktop 앱을 사용할 수 없으며 CLI를 사용해야 한다.

### 세션 시작

첫 메시지를 보내기 전에 네 가지를 구성한다.

- **Environment**: Claude가 실행되는 위치. Local(로컬 머신), Remote(Anthropic 호스팅 클라우드), SSH(원격 머신)
- **Project folder**: Claude가 작업할 폴더 또는 리포지토리
- **Model**: 전송 버튼 옞 드롭다운에서 모델 선택
- **Permission mode**: Claude의 자율성 수준

### 권한 모드

| 모드 | 설정 키 | 동작 |
| --- | --- | --- |
| **Ask permissions** | `default` | 파일 편집 및 명령 실행 전 매번 승인 요청 |
| **Auto accept edits** | `acceptEdits` | 파일 편집과 `mkdir`, `touch`, `mv` 등은 자동 승인, 나머지 터미널 명령은 승인 요청 |
| **Plan mode** | `plan` | 파일은 읽고 명령은 실행하지만, 소스 코드 편집 없이 계획만 제시 |
| **Auto** | `auto` | 백그라운드 안전 검사로 요청과의 정렬을 검증하며 모든 액션 실행 |
| **Bypass permissions** | `bypassPermissions` | 모든 권한 프롬프트 없이 실행, 샌드박스 컨테이너에서만 사용 |

### 핵심 기능

**Diff 뷰**: 파일 변경 시 `+12 -1` 형태의 인디케이터가 나타나며, 클릭하면 파일별 diff 뷰어가 열린다. 특정 라인을 클릭해 코멘트를 남길 수 있다. macOS에서 `Cmd+Enter`, Windows에서 `Ctrl+Enter`로 제출.

**앱 프리뷰**: Claude가 개발 서버를 시작하고 내장 브라우저에서 변경 사항을 검증한다. 스크린샷, DOM 검사, 요소 클릭, 폼 작성 등을 자동으로 수행한다. 설정 파일은 `.claude/launch.json`에 저장된다.

**병렬 세션**: 사이드바에서 `Cmd+N`(macOS) / `Ctrl+N`(Windows)으로 여러 세션을 병렬 실행할 수 있다. Git 리포지토리에서는 각 세션이 고유한 Git worktree를 가진다.

**사이드 챗**: `Cmd+;`(macOS) / `Ctrl+;`(Windows)로 메인 세션을 벗어나지 않고 질문 가능.

**PR 모니터링**: PR 오픈 후 CI 상태 바가 나타나며, Auto-fix(실패 시 자동 수정 시도), Auto-merge(모든 체크 통과 시 squash merge) 토글을 제공한다.

**클라우드 세션**: 장시간 실행 작업은 Remote 환경에서 Anthropic 클라우드 인프라에서 실행된다. 앱을 닫아도 계속 실행되며, claude.ai/code 또는 Claude 모바일 앱에서 모니터링할 수 있다.

**SSH 세션**: 원격 머신에 SSH로 연결하여 Claude Code를 실행할 수 있다. 원격 머신은 Linux 또는 macOS여야 하며, Desktop이 자동으로 Claude Code를 설치한다.

### Computer Use (Desktop)

Computer Use는 Claude가 앱을 열고 화면을 제어하며 사용자처럼 작업할 수 있게 해준다. 기본적으로 비활성화되어 있으며, 설정에서 활성화해야 한다. macOS에서는 접근성 및 화면 녹화 권한도 부여해야 한다.

Claude는 가장 정밀한 도구를 먼저 사용한다: Connector > Bash > Chrome > Computer Use 순서.

| 티어 | Claude가 할 수 있는 작업 | 적용 대상 |
| --- | --- | --- |
| View only | 스크린샷에서 앱 보기 | 브라우저, 트레이딩 플랫폼 |
| Click only | 클릭 및 스크롤 (타이핑 및 단축키 불가) | 터미널, IDE |
| Full control | 클릭, 타이핑, 드래그, 단축키 | 나머지 모든 앱 |

### Dispatch

Dispatch는 Cowork 탭에 있는 지속적인 대화로, 작업을 전달하면 Claude가 처리 방식을 결정한다. 개발 작업으로 판단되면 자동으로 Code 세션을 생성한다. Pro 또는 Max 플랜이 필요하다.

### 키보드 단축키

| 단축키 | 동작 |
| --- | --- |
| `Cmd` `/` | 키보드 단축키 표시 |
| `Cmd` `N` | 새 세션 |
| `Cmd` `W` | 세션 닫기 |
| `Ctrl` `Tab` | 다음/이전 세션 |
| `Esc` | Claude 응답 중지 |
| `Cmd` `Shift` `D` | Diff 패널 토글 |
| `Cmd` `Shift` `P` | 프리뷰 패널 토글 |
| `Ctrl` `` ` `` | 터미널 패널 토글 |
| `Cmd` `\` | 포커스된 패널 닫기 |
| `Cmd` `;` | 사이드 챗 열기 |
| `Ctrl` `O` | 뷰 모드 순환 |

---

## Claude Code on the Web

### 개요

Claude Code on the Web은 claude.ai/code에서 Anthropic 관리 클라우드 인프라에서 작업을 실행한다. 브라우저를 닫아도 세션이 유지되며, Claude 모바일 앱에서 모니터링할 수 있다.

### GitHub 인증

클라우드 세션은 GitHub 리포지토리에 접근해야 한다.

| 방법 | 작동 방식 | 적합한 경우 |
| --- | --- | --- |
| **GitHub App** | 웹 온보딩 중 Claude GitHub App 승인 | 브라우저 온보딩, Auto-fix가 필요한 팀 |
| **`/web-setup`** | 터미널에서 로컬 `gh` CLI 토큰을 Claude 계정에 동기화 | 이미 `gh`를 사용하는 개발자 |

### 클라우드 환경

각 세션은 새로운 Anthropic 관리 VM에서 리포지토리를 클론하여 실행된다.

**클라우드 세션에서 사용 가능한 것**

| 항목 | 사용 가능 여부 | 이유 |
| --- | --- | --- |
| 리포지토리의 `CLAUDE.md` | 가능 | 클론에 포함 |
| `.claude/settings.json` hooks | 가능 | 클론에 포함 |
| `.mcp.json` MCP 서버 | 가능 | 클론에 포함 |
| `.claude/rules/`, skills, agents, commands | 가능 | 클론에 포함 |
| 사용자 `~/.claude/CLAUDE.md` | 불가 | 로컬 머신에 있음 |
| 사용자 설정의 플러그인 | 불가 | 리포지토리 설정에 선언 필요 |
| `claude mcp add`로 추가한 MCP 서버 | 불가 | `.mcp.json`에 선언 필요 |
| 정적 API 토큰 및 자격 증명 | 불가 | 전용 시크릿 스토어 미지원 |

**사전 설치된 도구**

| 카테고리 | 포함 항목 |
| --- | --- |
| Python | 3.x, pip, poetry, uv, black, mypy, pytest, ruff |
| Node.js | 20, 21, 22 (nvm), npm, yarn, pnpm, bun |
| Ruby | 3.1, 3.2, 3.3, gem, bundler |
| PHP | 8.4, Composer |
| Java | OpenJDK 21, Maven, Gradle |
| Go | 최신 안정 버전 |
| Rust | rustc, cargo |
| Docker | docker, dockerd, docker compose |
| Databases | PostgreSQL 16, Redis 7.0 |
| Utilities | git, jq, yq, ripgrep, tmux, vim |

### 네트워크 액세스

| 수준 | 아웃바운드 연결 |
| --- | --- |
| **None** | 아웃바운드 네트워크 접근 없음 |
| **Trusted** | 허용된 도메인만: 패키지 레지스트리, GitHub, 클라우드 SDK |
| **Full** | 모든 도메인 |
| **Custom** | 사용자 정의 허용 목록, 기본 목록 포함 옵션 |

### 세션 간 이동

**터미널에서 Web으로**: `--remote` 플래그로 클라우드 세션 시작

```
claude --remote "Fix the authentication bug in src/auth/login.ts"
```

**Web에서 터미널로**: `--teleport`로 클라우드 세션을 로컬로 가져오기

```
claude --teleport
```

### 리소스 한계

- 4 vCPUs
- 16 GB RAM
- 30 GB 디스크

### 실행 환경 비교

|  | Web | Remote Control | CLI | Desktop |
| --- | --- | --- | --- | --- |
| **코드 실행 위치** | Anthropic 클라우드 VM | 로컬 머신 | 로컬 머신 | 로컬 또는 클라우드 VM |
| **채팅 위치** | claude.ai 또는 모바일 앱 | claude.ai 또는 모바일 앱 | 터미널 | Desktop UI |
| **로컬 설정 사용** | 아니오 (리포지토리만) | 예 | 예 | 로컬은 예, 클라우드는 아니오 |
| **GitHub 필요** | 예 | 아니오 | 아니오 | 클라우드 세션만 |
| **연결 끊김 후에도 지속** | 예 | 터미널이 열려 있는 동안 | 아니오 | 세션 유형에 따라 다름 |

---

## Remote Control

### 개요

Remote Control은 claude.ai/code 또는 Claude 모바일 앱(iOS/Android)을 로컬 머신에서 실행 중인 Claude Code 세션에 연결한다. 데스크에서 작업을 시작하고, 소파에서 폰으로, 다른 컴퓨터의 브라우저에서 이어서 작업할 수 있다.

### 요구 사항

- **구독**: Pro, Max, Team, Enterprise 플랜에서 사용 가능. API 키는 지원되지 않음
- **인증**: claude.ai를 통해 `/login`으로 로그인 필요
- **워크스페이스 신뢰**: 프로젝트 디렉토리에서 최소 한 번 `claude`를 실행하여 신뢰 대화상자 수락

### 세션 시작

**서버 모드**: 프로젝트 디렉토리에서 실행

```
claude remote-control
```

**인터랙티브 세션**: `--remote-control` 또는 `--rc` 플래그 사용

```
claude --remote-control "My Project"
```

**기존 세션에서**: `/remote-control` 또는 `/rc` 명령 사용

**VS Code**: 프롬프트 상자에서 `/remote-control` 또는 `/rc` 입력 (v2.1.79 이상)

### 서버 모드 플래그

| 플래그 | 설명 |
| --- | --- |
| `--name "My Project"` | claude.ai/code 세션 목록에 표시될 세션 제목 |
| `--spawn <mode>` | `same-dir`(기본값), `worktree`, `session` |
| `--capacity <N>` | 최대 동시 세션 수 (기본값 32) |
| `--verbose` | 상세 연결 및 세션 로그 |
| `--sandbox` / `--no-sandbox` | 샌드박싱 활성화/비활성화 |

### 연결 방법

- 세션 URL을 브라우저에서 열기
| QR 코드 스윈 (모바일 앱에서)
| claude.ai/code 또는 Claude 앱의 세션 목록에서 이름으로 찾기

### 모바일 푸시 알림

Remote Control 활성 시 Claude가 폰으로 푸시 알림을 보낼 수 있다. 장시간 실행 작업이 완료되거나 결정이 필요할 때 주로 전송된다. 프롬프트에서 `notify me when the tests finish`와 같이 직접 요청할 수도 있다.

---

## Chrome 확장

### 개요

Claude Code는 Claude in Chrome 브라우저 확장과 통합되어 CLI 또는 VS Code 확장에서 브라우저 자동화 기능을 제공한다. 코드를 빌드하고 컨텍스트 전환 없이 브라우저에서 테스트 및 디버그할 수 있다.

Claude는 브라우저 작업을 위해 새 탭을 열고, 로그인 상태를 공유하여 이미 로그인된 사이트에 접근할 수 있다.

### 기능

- **라이브 디버깅**: 콘솔 오류와 DOM 상태를 직접 읽고 원인 코드 수정
- **디자인 검증**: Figma 목업에서 UI를 빌드한 후 브라우저에서 일치 여부 확인
- **웹 앱 테스트**: 폼 유효성 검사, 시각적 회귀, 사용자 플로우 확인
- **인증된 웹 앱**: Google Docs, Gmail, Notion 등 API 커넥터 없이 상호작용
- **데이터 추출**: 웹 페이지에서 구조화된 정보 추출
- **작업 자동화**: 데이터 입력, 폼 작성, 다중 사이트 워크플로우 자동화
- **세션 녹화**: 브라우저 상호작용을 GIF로 녹화

### 사전 요구 사항

- Google Chrome 또는 Microsoft Edge
- Claude in Chrome 확장 v1.0.36 이상
- Claude Code v2.0.73 이상
- 직접 Anthropic 플랜 (Pro, Max, Team, Enterprise)

### 사용법

CLI에서 `/chrome`을 실행하여 연결 상태 확인, 권한 관리, 확장 재연결, 브라우저 선택이 가능하다. 기본 활성화는 `/chrome`에서 "Enabled by default" 선택.

---

## Slack 통합

### 개요

Claude Code in Slack은 Slack 워크스페이스에서 직접 Claude Code의 기능을 사용할 수 있게 해준다. 채널에서 `@Claude`를 멘션하여 코딩 작업을 요청하면, Claude가 자동으로 코딩 의도를 감지하고 Claude Code on the web에서 세션을 생성한다.

### 사용 사례

- 버그 조사 및 수정
| 빠른 코드 리뷰 및 수정
| 협업 디버깅
| 병렬 작업 실행

### 사전 요구 사항

| 요구 사항 | 세부 정보 |
| --- | --- |
| Claude 플랜 | Pro, Max, Team, Enterprise (Claude Code 액세스 포함) |
| Claude Code on the web | 활성화되어야 함 |
| GitHub 계정 | Claude Code on the web에 연결, 최소 하나의 리포지토리 인증 |
| Slack 인증 | Claude 앱을 통해 Claude 계정과 Slack 계정 연결 |

### 작동 방식

1. `@Claude` 멘션으로 코딩 요청
2. Claude가 메시지를 분석하여 코딩 의도 감지
3. claude.ai/code에 새 Claude Code 세션 생성
4. 작업 진행에 따라 Slack 스레드에 상태 업데이트 게시
5. 완료 시 요약 및 액션 버튼과 함께 `@Claude` 멘션으로 알림
6. "View Session" 또는 "Create PR" 버튼으로 결과 확인

### 액세스 제어

Claude는 설치 후 자동으로 채널에 추가되지 않는다. `/invite @Claude`로 명시적으로 초대해야 한다. 채널 멤버십을 통해 액세스가 제어되며, 공개 및 비공개 채널 모두 지원한다.

---

## 음성 입력 (Voice Dictation)

### 개요

Claude Code CLI에서 프롬프트를 타이핑 대신 말로 입력할 수 있다. 음성이 실시간으로 프롬프트 입력에 전사되어, 동일한 메시지에서 음성과 타이핑을 혼합할 수 있다.

### 요구 사항

- claude.ai 계정 인증 필요 (API 키, Bedrock, Vertex AI, Foundry에서는 사용 불가)
- HIPAA 규정이 활성화된 조직에서는 사용 불가
- 로컬 마이크 접근 필요 (Web, SSH 세션에서는 사용 불가)
- 전사는 메시지나 토큰을 소비하지 않음

### 활성화

```
/voice
```

| 명령 | 효과 |
| --- | --- |
| `/voice` | 켜기/끄기 토글, 현재 모드 유지 |
| `/voice hold` | Hold 모드 활성화 |
| `/voice tap` | Tap 모드 활성화 |
| `/voice off` | 비활성화 |

### 두 가지 모드

**Hold 모드** (기본값): `Space`를 누르고 있는 동안 녹음. 놓으면 녹음 중지.

**Tap 모드**: `Space`를 한 번 눌러 녹음 시작, 다시 눌러 전송. 프롬프트가 비어 있을 때만 첫 탭이 녹음을 시작한다. 15초 침묵 또는 2분 총 녹음 시 자동 중지.

### 지원 언어

| 언어 | 코드 | 언어 | 코드 |
| --- | --- | --- | --- |
| Czech | `cs` | Korean | `ko` |
| Danish | `da` | Norwegian | `no` |
| Dutch | `nl` | Polish | `pl` |
| English | `en` | Portuguese | `pt` |
| French | `fr` | Russian | `ru` |
| German | `de` | Spanish | `es` |
| Greek | `el` | Swedish | `sv` |
| Hindi | `hi` | Turkish | `tr` |
| Indonesian | `id` | Ukrainian | `uk` |
| Italian | `it` | Japanese | `ja` |

---

## Computer Use (CLI)

### 개요

Computer Use는 Claude가 앱을 열고, 화면을 제어하며, 사용자처럼 직접 작업할 수 있게 해준다. CLI에서 Claude는 Swift 앱을 컴파일하고, 실행하고, 모든 버튼을 클릭하고, 결과를 스크린샷할 수 있다.

### 기능

- 네이티브 앱 빌드 및 검증
- 엔드투엔드 UI 테스트
| 시각적 및 레이아웃 버그 디버깅
| GUI 전용 도구 제어 (iOS 시뮬레이터, 디자인 도구 등)

### 활성화

Computer Use는 `computer-use`라는 내장 MCP 서버로 제공된다. 기본적으로 비활성화되어 있으며 `/mcp`에서 활성화할 수 있다. CLI에서는 macOS만 지원된다 (Windows에서는 Desktop 앱 사용).

### 앱 승인

세션 내에서 Claude가 특정 앱을 처음 사용할 때 터미널에 프롬프트가 나타난다. "Allow for this session" 또는 "Deny"를 선택한다.

| 경고 | 적용 대상 |
| --- | --- |
| Equivalent to shell access | Terminal, iTerm, VS Code, Warp 등 |
| Can read or write any file | Finder |
| Can change system settings | System Settings |

### CLI vs Desktop 비교

| 기능 | Desktop | CLI |
| --- | --- | --- |
| 플랫폼 | macOS, Windows | macOS만 |
| 활성화 | Settings 토글 | `/mcp`에서 `computer-use` 활성화 |
| Denied apps 목록 | 설정에서 구성 가능 | 아직 미지원 |
| Auto-unhide 토글 | 옵션 | 항상 켜짐 |

---

## 원격 접근 방식 비교

터미널에서 벗어났을 때 Claude Code를 사용하는 여러 방식을 비교한다.

| 방식 | 트리거 | Claude 실행 위치 | 설정 | 적합한 경우 |
| --- | --- | --- | --- | --- |
| **Dispatch** | 모바일 앱에서 작업 메시지 | 로컬 머신 (Desktop) | 모바일 앱과 Desktop 페어링 | 자리 비움 중 작업 위임 |
| **Remote Control** | claude.ai/code 또는 모바일 앱에서 세션 제어 | 로컬 머신 (CLI/VS Code) | `claude remote-control` 실행 | 다른 기기에서 진행 중인 작업 조정 |
| **Channels** | Telegram, Discord 등에서 이벤트 푸시 | 로컬 머신 (CLI) | 채널 플러그인 설치 | CI 실패 등 외부 이벤트 대응 |
| **Slack** | 팀 채널에서 `@Claude` 멘션 | Anthropic 클라우드 | Slack 앱 설치 + Claude Code on the web 활성화 | 팀 채팅에서 PR 및 리뷰 |
| **Scheduled tasks** | 일정 설정 | CLI, Desktop, 클라우드 | 빈도 선택 | 일일 리뷰 등 반복 자동화 |

---

## 통합

| 통합 | 기능 | 용도 |
| --- | --- | --- |
| **Chrome** | 브라우저를 로그인된 세션으로 제어 | 웹 앱 테스트, 폼 작성, API 없는 사이트 자동화 |
| **GitHub Actions** | CI 파이프라인에서 Claude 실행 | 자동 PR 리뷰, 이슈 분류, 정기 유지보수 |
| **GitLab CI/CD** | GitLab용 GitHub Actions 동일 기능 | GitLab CI 기반 자동화 |
| **Code Review** | 모든 PR을 자동으로 리뷰 | 휴먼 리뷰 전 버그 포착 |
| **Slack** | 채널에서 `@Claude` 멘션에 응답 | 팀 채팅에서 버그 리포트를 PR로 전환 |

MCP 서버와 커넥터를 통해 위에 나열되지 않은 통합도 가능하다: Linear, Notion, Google Drive 또는 자체 내부 API 등.
