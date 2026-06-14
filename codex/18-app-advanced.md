# Codex App 고급 인터랙션 기능

> **출처 (대조 기준)**
> - Computer Use: https://developers.openai.com/codex/app/computer-use/
> - In-app Browser: https://developers.openai.com/codex/app/browser/
> - Chrome Extension: https://developers.openai.com/codex/app/chrome-extension/
> - Appshots: https://developers.openai.com/codex/appshots/
> - App Features (개요): https://developers.openai.com/codex/app/features/
>
> 최종 업데이트: 2026-06-15

이 문서는 Codex App이 코드 편집을 넘어 **데스크톱 GUI·브라우저·화면 캡처**를 다루는 고급 인터랙션 기능을 하나로 묶어 정리한다. 각 기능은 별도의 플러그인·권한·설정을 가지며, 공식 문서에 명시된 플랫폼 지원·요구사항·안전 권고를 표 중심으로 정리한다.

---

## 기능 한눈에 보기

| 기능 | 용도 | 지원 플랫폼 | 진입점 / 단축키 | 요구 플러그인 |
|---|---|---|---|---|
| **Computer Use** | GUI 앱·브라우저를 Codex가 직접 조작 | macOS, Windows (EEA·UK·스위스 런칭 제외) | `@Computer`, `@AppName`, 또는 프롬프트에서 "computer use" | Computer Use plugin |
| **In-app Browser** | 렌더링된 페이지 미리보기·댓글, localhost 흐름 조작 | macOS, Windows (App 전체) | 툴바, URL 클릭, `Cmd`+`Shift`+`B` / `Ctrl`+`Shift`+`B` | Browser plugin (browser use 시) |
| **Chrome Extension** | 로그인 상태가 필요한 Chrome 사이트 병렬 조작 | Chrome 설치 OS | `@Chrome` 프롬프트 | Chrome plugin + Chrome 확장 |
| **Appshots** | 최상단 Mac 앱 창(스크린샷+텍스트)을 스레드로 전송 | macOS (App 전용) | Command 키 두 번 (또는 사용자 지정 단축키) | 없음 (앱 내장) |

> **선택 시 기준 (공식 권고)**: 로컬 개발 서버·파일 기반 미리보기·공개 페이지는 In-app Browser 우선. 로그인이 필요한 사이트는 Chrome Extension. 그 외 GUI 앱·데스크톱 흐름은 Computer Use. 단발적 컨텍스트 공유는 Appshots.

---

## Computer Use

> https://developers.openai.com/codex/app/computer-use/

Computer Use는 Codex가 macOS 또는 Windows의 그래픽 사용자 인터페이스를 보고 조작할 수 있게 한다. 커맨드라인 도구나 구조적 통합으로 충분하지 않은 작업 — 데스크톱 앱 점검, 브라우저 사용, 앱 설정 변경, GUI에서만 재현되는 버그 재현 — 에 사용한다.

### 지원 환경 및 제약

| 항목 | macOS | Windows |
|---|---|---|
| 가용성 | 지원 | 지원 |
| 런칭 제외 지역 | EEA, 영국, 스위스 (런칭 시점) | EEA, 영국, 스위스 (런칭 시점) |
| 작동 방식 | 잠금 해제(Locked) 또는 전경(foreground) | 활성 데스크톱에서 **전경 전용**. 같은 세션에서 백그라운드 작동 불가 |
| 백그라운드 작업 | Locked computer use(별도 활성화 필요) | 미지원. 장치를 잠금 해제·인터넷 연결 상태로 두거나 VM 사용 권장 |
| 필수 시스템 권한 | Screen Recording, Accessibility | 대상 앱이 활성 데스크톱에 보여야 함 |

### 설치 및 권한

1. **Codex 설정 > Computer Use** 에서 **Install** 클릭 — Computer Use plugin 설치.
2. macOS: 프롬프트가 뜨면 Screen Recording(대상 앱을 볼 수 있게)과 Accessibility(클릭·타이핑·탐색) 권한 부여.
3. Windows: 작업 실행 중 대상 앱이 활성 데스크톱에 보이도록 유지.

> 시스템 권한(Screen Recording·Accessibility)은 Codex App approvals와 별개다. App approvals는 "Codex가 사용하도록 허용할 앱"을 결정하고, 시스템 권한은 "보고 조작하는 능력"을 결정한다. 파일 읽기·쓰기·셸 명령은 여전히 스레드의 sandbox·approval 설정을 따른다.

### Windows 앱 정책 (`config.toml`)

Windows에서 Computer Use는 영구 앱 결정을 `$CODEX_HOME/computer-use/config.toml` 에 저장한다. 프롬프트 없이 열 수 있는 앱(`allowed`)과 반드시 거부할 앱(`denied`)을 나열한다. `denied`가 `allowed`보다 우선한다. 어느 목록에도 없는 앱은 Codex가 매번 묻는다.

```toml
[apps]
allowed = ["mspaint.exe"]
denied  = ["calc.exe"]
```

- 식별자는 Windows Computer Use가 보고하는 값(데스크톱 앱은 실행 파일 이름, 패키지 앱은 App User Model ID).
- 이 파일은 로컬 결정 저장용. 관리자가 `requirements.toml`의 `[features].computer_use = false` 로 비활성화하는 것과는 별개.

### Locked Computer Use (macOS 전용)

Locked computer use는 Mac이 잠긴 뒤에도 연결된 기기에서 시작한 작업이 Computer Use를 계속 쓸 수 있게 한다(사용자가 명시적으로 활성화한 경우에 한함).

- 활성화하면 Codex가 macOS 잠금 해제 흐름에 참여하는 **Apple authorization plug-in**을 설치한다.
- 일반적인 원격 잠금 해제 경로가 아니며, 다른 앱이나 로컬 프로세스가 Mac을 잠금 해제하도록 허용하지 않는다.

**활성화 절차**

1. Codex 설정 > Computer Use 열기.
2. Locked computer use 활성화.
3. Mac 화면이 잠긴 뒤 연결된 기기에서 computer use 작업을 사용하는 작업 시작.

**동작 및 안전장치**

| 항목 | 동작 |
|---|---|
| 잠금 해제 시점 | 활성·신뢰할 수 있는 computer use 턴인 경우에만 임시 해제. 그 외 창 밖에서는 거부하고 수동 잠금 해제 요청 |
| 해제 중 보호 | 임시 해제 중 로컬 사용을 차단하고 잠금 화면 보호 유지, 모든 디스플레이 덮음 |
| 로컬 입력 감지 | 키보드/포인터 입력 감지 시 Mac을 다시 잠그고 수동 해제까지 자동 해제 일시정지 |
| 인가 창 | 현재 잠금 해제 시도에만 국한된 짧은 수명 |

### 좋은 활용 예시 (공식)

- macOS/Windows/iOS 시뮬레이터 등 Codex가 빌드 중인 데스크톱/모바일 앱 테스트
- 웹 브라우저가 필요한 작업 수행
- GUI에서만 나타나는 버그 재현
- UI 클릭이 필요한 앱 설정 변경
- 플러그인으로 접근 불가능한 앱·데이터 소스 점검
- (macOS) 백그라운드에서 범위가 좁은 작업을 실행하며 다른 작업 계속
- 두 개 이상의 앱에 걸친 워크플로 실행

```text
Open the app with computer use, reproduce the onboarding bug, and fix the
smallest code path that causes it. After each change, run the same UI flow
again.
```

```text
Open @Chrome and verify the checkout page still works after the latest changes.
```

> **우선순위 권고**: 대상 앱이 전용 플러그인이나 MCP 서버를 제공하면 그 구조적 통합을 데이터 접근·반복 작업에 우선 사용하고, Codex가 앱을 시각적으로 점검·조작해야 할 때만 computer use를 선택하라. 로컬 웹앱을 빌드 중이라면 In-app Browser를 먼저 사용하라.

### 제한 및 안전 권고

- 터미널 앱과 Codex 자체는 자동화할 수 없다(Codex 보안 정책 우회 방지).
- 관리자로 인증하거나 컴퓨터의 보안·개인정보 보호 권한 프롬프트를 승인할 수 없다.
- 데스크톱 앱으로 만든 변경사항은 디스크에 저장되어 프로젝트가 추적하기 전까지 review pane에 나타나지 않을 수 있다.
- 민감한 흐름(계정·보안·개인정보·네트워크·결제·자격증명)에서는 항상 머물며 각 단계를 승인하라.
- Codex가 브라우저를 사용하면 로그인된 페이지와 상호작용할 수 있다 — 본인이 직접 취하는 행동처럼 검토하라. 웹 페이지는 악의적·오해 유발 콘텐츠를 포함할 수 있고, 사이트는 승인된 클릭·폼 제출·로그인된 행동을 사용자 계정에서 온 것으로 취급할 수 있다.
- ChatGPT 데이터 통제가 computer use가 찍은 스크린샷을 포함해 Codex가 처리하는 콘텐츠에 적용된다.

---

## In-app Browser

> https://developers.openai.com/codex/app/browser/

스레드 안에서 렌더링된 웹 페이지를 사용자와 Codex가 함께 볼 수 있는 공유 뷰. 웹앱을 빌드/디버그할 때 페이지를 미리보고 시각적 댓글을 붙이는 데 사용한다.

### 지원 및 진입

| 항목 | 내용 |
|---|---|
| 용도 | 로컬 개발 서버, 파일 기반 미리보기, 로그인 불필요한 공개 페이지 |
| 미지원 | 인증 흐름, 로그인된 페이지, 일반 브라우저 프로필, 쿠키, 확장, 기존 탭 |
| 진입 | 툴바, URL 클릭, 브라우저에서 직접 이동, `Cmd`+`Shift`+`B` (Windows: `Ctrl`+`Shift`+`B`) |

> 페이지 콘텐츠는 신뢰할 수 없는 컨텍스트로 취급. 브라우저 흐름에 시크릿을 붙여넣지 말 것.

### Browser use (Browser plugin)

Browser use는 Codex가 in-app 브라우저를 직접 조작하게 한다. Browser plugin을 설치·활성화한 뒤 프롬프트에서 브라우저 사용을 요청하거나 `@Browser`로 직접 참조한다.

| 수행 가능 동작 |
|---|
| 클릭·타이핑·렌더링된 상태 점검 |
| 스크린샷 캡처 |
| 페이지 에셋 다운로드 |
| 읽기 전용 페이지 점검 JavaScript 실행 |
| 페이지에서 수정 검증 |

- Codex는 허용 목록에 없는 사이트는 사용 전에 묻는다. 허용 목록에서 사이트를 제거하면 다시 묻고, 차단 목록에서 제거하면 차단 대신 다시 물을 수 있다.
- 허용·차단 웹사이트는 설정에서 관리.

```text
Use the browser to open http://localhost:3000/settings, reproduce the layout
bug, and fix only the overflowing controls.
```

### 미리보기 워크플로우

1. 통합 터미널 또는 local environment action으로 개발 서버 시작.
2. 인증 불필요한 로컬 라우트·파일 기반 페이지·공개 페이지를 URL 클릭 또는 직접 이동으로 열기.
3. 코드 diff와 함께 렌더링 상태 검토.
4. 변경이 필요한 요소·영역에 browser comments.
5. Codex에 댓글 처리를 요청하되 범위를 좁게 유지.

### 페이지 댓글 (Annotation mode)

- Annotation mode를 켜고 요소/영역 선택 후 댓글 제출.
- Annotation mode에서 `Shift`+클릭으로 영역 선택.
- `Cmd`+클릭으로 댓글 즉시 전송.

**스타일 피드백**: 주석 입력 옆의 설정 아이콘을 누르면 font, text, spacing, color 값을 변경하고 페이지에서 직접 결과를 미리보고 주석을 보낼 수 있어 Codex에게 더 명확한 변경 목표를 전달한다.

좋은 댓글 예시:

```text
This button overflows on mobile. Keep the label on one line if it fits,
otherwise wrap it without changing the card height.
```

```text
This tooltip covers the data point under the cursor. Reposition the tooltip so
it stays inside the chart bounds.
```

### 작업 범위 유지 권고

- 페이지·라우트·로컬 URL 이름 명시.
- 관심 있는 시각적 상태(loading, empty, error, success) 명시.
- 정확한 요소·영역에 댓글.
- Codex가 코드를 바꾼 뒤 업데이트된 라우트 재검토.
- 브라우저를 쓰기 전에 dev server 시작·점검을 Codex에게 요청.

### Developer mode (CDP)

Developer mode는 Chrome의 Browser use와 Codex in-app browser에서 동작한다. Chrome DevTools Protocol(CDP)에 대한 통제된 접근을 Codex에게 제공한다.

| 용도 |
|---|
| JavaScript 프로파일링 |
| console 출력·네트워크 트래픽 점검 |
| DOM·적용된 스타일 등 페이지 상태 점검 |
| 라이브 브라우저에서 이슈 직접 진단 |

**활성화**: Settings > Browser 의 Developer mode에서 **Enable full CDP access** 켜기. 조직이 이 설정을 비활성화했다면 로컬에서 켤 수 없다(관리자가 `requirements.toml`의 `[features]` 아래 `browser_use_full_cdp_access = false` 로 설정).

> Full CDP access는 데이터를 위험에 빠뜨릴 수 있는 민감한 브라우저 내부를 Codex가 점검·제어하게 한다. Codex는 full CDP를 사용해 웹사이트를 점검하기 전에 명시적 승인을 요청한다. 사이트·작업·요청된 접근을 승인 전에 검토하라.

```text
This app is slow. Use @Browser to capture a performance trace and inspect
network traffic, then identify the bottleneck.
```

- in-app browser는 `@Browser`, Chrome에서 Developer mode를 쓰려면 Chrome 확장을 설정하고 `@Chrome` 사용.

---

## Chrome Extension

> https://developers.openai.com/codex/app/chrome-extension/

Codex Chrome extension은 로그인된 브라우저 상태가 필요한 Chrome 작업에 Codex가 Chrome을 사용하게 한다. LinkedIn, Salesforce, Gmail, 내부 도구 등 로그인이 필요한 사이트를 읽거나 조작할 때 사용한다. 로컬 개발 서버·파일 기반 미리보기·로그인 불필요 공개 페이지는 in-app browser가 우선이다.

> 페이지 콘텐츠는 신뢰할 수 없는 컨텍스트로 취급. Codex가 계속하기 전에 웹사이트를 검토하라.

### 설정

1. Codex에서 **Plugins** 열기.
2. **Chrome** plugin 추가.
3. 설정 흐름을 따라 Codex Chrome extension 설치 및 Chrome 권한 프롬프트 승인.
4. Chrome을 열고 Codex extension이 **Connected**로 표시되는지 확인.

```text
@Chrome open Salesforce and update the account from these call notes.
```

- Chrome이 열려 있지 않으면 Codex가 열 수 있다.
- Chrome 브라우저 작업은 스레드별 작업이 함께 묶이도록 Chrome tab groups에서 실행.

### 웹사이트 승인 (호스트 기준)

기본적으로 Codex는 새 웹사이트와 상호작용 전에 묻는다. 프롬프트는 웹사이트 호스트(예: `example.com`)를 기준으로 한다.

| 옵션 | 의미 |
|---|---|
| Allow for current chat | 현재 채팅에만 허용 |
| Always allow host | 해당 웹사이트를 다시 묻지 않고 사용 |
| Decline | 거부 |

### 허용·차단 목록 관리

Computer Use 설정에서 도메인 allowlist·blocklist 관리.
- allowlist: 다시 묻지 않고 사용할 도메인.
- blocklist: Codex가 사용하지 않아야 할 도메인.
- allowlist 제거 → 다시 묻기. blocklist 제거 → 차단 대신 다시 물을 수 있음.

**Elevated Risk 설정**

| 설정 | 동작·위험 |
|---|---|
| Always allow browser content | 켜면 웹사이트 사용 전 확인을 더 이상 요청하지 않음 |
| Browser history | 민감한 telemetry·내부 URL·검색어·활동 포함 가능. Codex가 접근을 요청할 때만 사용되며, always-allow 옵션은 없음 |

### Chrome extension 권한 프롬프트

설치 시 Chrome이 요청할 수 있는 권한:
- 페이지 디버거 접근
- 모든 웹사이트의 모든 데이터 읽기 및 변경
- 로그인된 모든 기기의 브라우저 기록 읽기 및 변경
- 알림 표시
- 북마크 읽기 및 변경
- 다운로드 관리
| 협력 네이티브 애플리케이션과 통신
- 탭 그룹 보기 및 관리

이 권한들이 확장의 브라우저 워크플로 능력을 뒷받침하지만, 작업 중 웹사이트·기록 사용 전에 Codex는 자체 확인·설정·allowlist·blocklist를 따른다.

### Memories · 데이터 보관

- Browser use는 Codex Memories 설정을 따른다. Memories가 켜져 있으면 저장된 memory를 Chrome 작업에 사용할 수 있고, 꺼져 있으면 사용하지 않는다.
- OpenAI는 확장으로부터의 Chrome 행동에 대한 별도의 완전한 기록을 저장하지 않는다. 브라우저 활동은 Codex 컨텍스트의 일부가 될 때만(페이지 텍스트, 스크린샷, tool call, 요약, 메시지 등) 보관된다.
- ChatGPT·Codex 데이터 통제가 컨텍스트로 처리된 콘텐츠에 적용된다.

### 파일 업로드

Chrome 작업이 컴퓨터의 파일을 업로드해야 하면 Codex 확장이 Chrome에서 file URL에 접근하도록 허용:
1. Chrome 툴바 확장 아이콘 > **Manage Extensions**.
2. Codex 확장 카드 > **Details**.
3. **Allow access to file URLs** 켜기.

변경 후 Chrome 작업을 다시 시작.

### 연결 문제 해결

1. 설정의 blocklist에서 대상 웹사이트가 차단되어 있지 않은지 확인.
2. 툴바/확장 메뉴에서 Codex 확장이 **Connected**인지 확인. disconnected이거나 native host 누락 메시지가 뜨면 Plugins에서 Chrome plugin을 제거 후 다시 추가하고 설정 흐름 반복.
3. Plugins에서 Chrome plugin이 켜져 있는지 확인.
4. 확장이 설치된 것과 같은 Chrome 프로필을 사용 중인지 확인.
5. 새 Codex 스레드에서 다시 시도(스레드별 연결 상태 초기화).
6. Chrome과 Codex를 재시작 후 재시도. 그래도 안 되면 확장 제거, Chrome plugin 재추가, 설정 흐름 재실행. 연결은 되나 작동하지 않으면 `/feedback` 실행 후 thread ID를 포함해 지원 요청.

---

## Appshots

> https://developers.openai.com/codex/appshots/

Appshots는 최상단(frontmost) 앱 창을 Codex 스레드로 보낸다. 다른 Mac 앱에서 작업 중이며 현재 컨텍스트를 Codex에 제공하고 싶을 때 사용한다. **macOS Codex app에서만** 사용할 수 있다.

### 캡처 내용

appshot은 최상단 창만 캡처하며 다음을 포함할 수 있다:
- 보이는 창의 이미지.
- 해당 창에서 사용 가능한 텍스트(보이는 텍스트 + 스크롤 영역 밖의 텍스트 포함).

스레드에 추가된 appshot은 일반 첨부 파일처럼 동작한다. 세션 파일에 로컬로 저장된다.

### 활용 예시

- API 레퍼런스 페이지를 공유하고 그것을 사용하는 스크립트 작성 요청.
- 이메일/캘린더 뷰를 공유하고 다음 단계 초안 요청.
- 이미지 에디터/디자인/미리보기 창을 공유하고 관련 에셋·코드 수정 요청.
- 설명보다 보여주기 쉬운 에러/설정 패널/앱 상태 공유.

### appshot 찍기

1. Mac에서 Codex app 열기.
2. 공유할 앱·창 열기.
3. **Command 키 두 번** 또는 Codex 설정에서 구성한 사용자 지정 단축키 누르기.
4. macOS 권한 요청 시 허용.
5. appshot으로 작업을 Codex에 요청.

**스레드 동작**:
- 기본적으로 새 스레드를 시작.
- 최근 60초 이내에 Codex 스레드와 상호작용했다면 해당 최근 스레드에 appshot 추가.
- 연속 appshot은 같은 스레드에 추가.
- Appshots 단축키는 Codex 설정에서 변경 가능.

### 권한 및 안전

| 권한 | 목적 |
|---|---|
| Screen & System Audio Recording | 최상단 창의 이미지 캡처 |
| Accessibility | 최상단 창의 사용 가능한 텍스트 읽기 |

> appshot을 찍으면 캡처된 이미지·사용 가능한 텍스트가 Codex에 공유된다. 작업에 필요하지 않은 한 민감한 콘텐츠의 appshot을 피하라. 스크린샷·문서를 Codex와 공유할 때와 같은 방식으로 검토하라.

### 제한 및 문제 해결

- Appshots는 Codex app 기능. macOS Codex app에서만 생성 가능.
- CLI에서 appshot이 이미 포함된 스레드를 재개하면 첨부는 스레드 기록의 일부지만 CLI는 새 appshot을 만들 수 없다.
- Google Docs, Gmail, Google Sheets, Google Slides 등 일부 앱·웹사이트에서는 보이는 스크린샷만 받고 전체 문서/화면 밖 텍스트는 받지 못할 수 있다. 매칭 플러그인이 설치되어 있으면 Codex가 해당 플러그인으로 관련 앱 콘텐츠에 접근할 수 있다.

**작동하지 않을 때**:
1. System Settings > Privacy & Security 열기.
2. Codex Computer Use의 Screen & System Audio Recording, Accessibility 확인.
3. Codex 재시작 후 재시도.

---

## 부록: 단축키·설정 위치 요약

| 동작 | 단축키 / 위치 |
|---|---|
| In-app browser 열기 | `Cmd`+`Shift`+`B` (Win: `Ctrl`+`Shift`+`B`) |
| Appshot 찍기 | Command 키 두 번 (macOS, 사용자 지정 가능) |
| 통합 터미널 토글 | `Cmd`+`J` |
| 음성 프롬프트 | `Ctrl`+`M` 길게 누르며 말하기 |
| Computer Use plugin 설치 | Codex 설정 > Computer Use > Install |
| Browser plugin / Developer mode | Settings > Browser |
| Chrome plugin | Plugins > Chrome |
| Windows Computer Use 앱 정책 | `$CODEX_HOME/computer-use/config.toml` (`[apps]`) |
| 관리자 기능 비활성화 | `requirements.toml` — `[features].computer_use`, `[features].browser_use_full_cdp_access` |

---

## 출처

- Computer Use — https://developers.openai.com/codex/app/computer-use/
- In-app Browser — https://developers.openai.com/codex/app/browser/
- Chrome Extension — https://developers.openai.com/codex/app/chrome-extension/
- Appshots — https://developers.openai.com/codex/appshots/
- Codex App Features (개요) — https://developers.openai.com/codex/app/features/
