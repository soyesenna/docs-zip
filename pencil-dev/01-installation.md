# Pencil Dev 설치 및 설정

> 원문: https://docs.pencil.dev/getting-started/installation, https://docs.pencil.dev/getting-started/authentication

Pencil은 IDE 확장과 독립 데스크톱 애플리케이션으로 제공됩니다. 워크플로에 맞는 설치 방법을 선택하세요.

---

## 설치 방법별 비교

| 방법 | 플랫폼 | AI 기능 | 비고 |
| --- | --- | --- | --- |
| VS Code 확장 | macOS, Linux, Windows | Claude Code 필요 | 가장 많이 사용 |
| Cursor 확장 | macOS, Linux, Windows | Claude Code 필요 | Cursor IDE 사용자용 |
| 데스크톱 앱 | macOS, Linux, Windows | Claude Code 필요 | 독립 실행 |

---

## VS Code 확장

### 설치

1. VS Code 실행
2. 확장 프로그램 열기 (`Cmd/Ctrl + Shift + X`)
3. "Pencil" 검색
4. Install 클릭

### 확인

1. `.pen` 확장자의 새 파일 생성 (예: `test.pen`)
2. 파일 열기
3. 에디터 우상단에 Pencil 아이콘 확인

**아이콘이 보이지 않는 경우:**

| 확인 사항 | 방법 |
| --- | --- |
| 명령 팔레트 | `Cmd/Ctrl + Shift + P` → "Pencil" 검색 |
| 확장 활성화 | 확장 패널에서 Pencil이 활성화 상태인지 확인 |

---

## Cursor 확장

### 설치

1. Cursor 실행
2. 확장 프로그램 열기
3. "Pencil" 검색
4. Install 클릭

### 확인

VS Code와 동일: `.pen` 파일 생성 후 Pencil 아이콘 확인

### 일반적인 문제

| 문제 | 해결 방법 |
| --- | --- |
| 확장이 연결되지 않음 | Claude Code 로그인 확인 → Pencil MCP 서버 연결 확인 |
| 프롬프트 편집기/박스가 보이지 않음 | 활성화/로그인 상태 확인 → Cursor 재시작 → 확장 재설치 |
| "You need Cursor Pro" 메시지 | Cursor 구독 상태 확인 |

---

## 데스크톱 애플리케이션

### macOS

**다운로드:**

- Pencil 웹사이트 또는 releases 페이지에서 최신 `.dmg` 다운로드
- Pencil을 Applications 폴더로 드래그
- Pencil 실행

**첫 실행 시:** macOS 보안 경고가 뜨면 우클릭 → 열기로 실행. 활성화 과정 완료 후 AI 기능 사용을 위해 Claude Code 로그인.

### Linux

```bash
# .deb 패키지
sudo dpkg -i pencil-*.deb

# .AppImage
chmod +x pencil-*.AppImage
./pencil-*.AppImage
```

| 환경 | 상태 |
| --- | --- |
| X11 | 안정적 |
| Wayland | 일부 UI 이슈 발생 가능 |
| Hyprland | 일부 UI 이슈 발생 가능 |

### Windows

- 데스크톱 앱 또는 VS Code/Cursor 확장 사용
- 확장 설치는 macOS/Linux와 동일

---

## Claude Code CLI

Pencil의 AI 기능을 사용하려면 Claude Code CLI 설치와 인증이 필요합니다.

### 설치

```bash
# npm으로 설치
npm install -g @anthropic-ai/claude-code-cli

# 또는 공식 설치 스크립트
curl https://claude.ai/cli/install.sh | sh
```

### 인증

```bash
# Claude Code 실행 → 브라우저 인증 흐름 진행
claude
```

인증 과정에서 다음 단계가 자동으로 진행됩니다:

1. 브라우저 열기
2. Anthropic 계정으로 로그인
3. 인증 정보를 로컬에 저장

### 확인

```bash
# CLI 버전 확인
claude --version
```

**Pencil 내에서도 확인 가능:**

| 확인 사항 | 방법 |
| --- | --- |
| 연결 경고 | Pencil 실행 후 "Claude Code not connected" 경고가 없는지 확인 |
| AI 프롬프트 패널 | `Cmd/Ctrl + K`로 AI 프롬프트 패널이 정상 작동하는지 확인 |

---

## MCP 서버

Pencil MCP 서버는 Pencil 사용 시 자동으로 실행됩니다. AI 어시스턴트에게 .pen 파일을 읽고 수정할 수 있는 도구를 제공합니다.

### 자동 설정

- Pencil 실행 시 MCP 서버 자동 시작
- 로컬에서 실행 (클라우드 의존 없음)
- 추가 설정 불필요

### 확인

| 환경 | 확인 방법 |
| --- | --- |
| Cursor | Settings → Tools & MCP → Pencil 확인 |
| Codex CLI | Pencil 실행 후 Codex 열기 → `/mcp` 실행 → Pencil 확인 |

---

## 설치 후 설정

Pencil 설치 완료 후 다음 단계를 진행하세요:

1. **활성화 완료** — 이메일로 Pencil 활성화
2. **Claude Code 로그인** — AI 기능 사용을 위해 필요
3. **웰컴 파일 열기** — 캔버스 우클릭 → Open Welcome File
4. **첫 디자인 생성** — `.pen` 파일 생성 방법 참고
5. **AI 프롬프트 패널 열기** — `Cmd/Ctrl + K`
6. **디자인 요청** — 예: "Create a login form with email and password"

---

## 인증

Pencil은 Pencil 자체 활성화와 Claude Code 인증 두 가지가 필요합니다.

### Pencil 활성화

1. Pencil 실행 시 이메일 입력
2. 이메일로 받은 활성화 코드 입력
3. 활성화 완료

### 일반적인 인증 문제

| 문제 | 원인 | 해결 |
| --- | --- | --- |
| 활성화 이메일 미수신 | 스팸 필터, 지연 | 스팸함 확인, 다른 이메일 시도, 몇 분 대기 |
| "Invite not found" | 대기자 명단 | 확장 재설치, 지원팀 문의 |
| 반복 활성화 프롬프트 | 알려진 이슈 | IDE 재시작, 확장 재설치 |
| 마이그레이션 중 활성화 멈춤 | 확장 데이터 이관 문제 | 확장 재설치, 확장 데이터 삭제 후 재시도 |
| "Claude Code isn't connected" | CLI 인증 누락 | 터미널에서 `claude` 실행 후 인증 |
| "Invalid API key" | 인증 충돌 | `ANTHROPIC_API_KEY` 환경변수 제거, CLI 인증 사용 |
| CLI는 작동하지만 Pencil에서 미연결 | 서드파티 프로바이더 충돌 | 클린 Claude Code 세션 사용, IDE 재시작, 충돌 환경변수 확인 |

### 인증 방식

| 방식 | 설명 | 비고 |
| --- | --- | --- |
| **Claude CLI (권장)** | `claude` 실행 → 브라우저 인증 | 가장 간단하고 안정적 |
| **API 키 (대안)** | `ANTHROPIC_API_KEY` 환경변수 설정 | CLI 인증과 충돌 가능, 일반 사용 비권장 |

### 권한 및 접근

| 이슈 | 원인 | 해결 |
| --- | --- | --- |
| 폴더 접근 불가 | 폴더 권한 제한, OS 보안 설정 | 접근 프롬프트 수락, 시스템 설정에서 권한 업데이트 |
| 권한 프롬프트 미표시 | 시스템 알림 설정 | Pencil 외부에서 별도 Claude Code 세션 실행, 시스템 알림 설정 확인 |

### MCP 서버 보안

| 항목 | 설명 |
| --- | --- |
| 로컬 전용 | MCP 서버는 로컬에서만 실행 |
| 데이터 전송 | AI 기능 사용 시에만 Claude에 프롬프트 전송 |
| 저장소 | 현재 비공개 (소스 코드 미공개) |
| 도구 검사 | Cursor/VS Code 설정에서 MCP 도구 확인 가능 |

### 권장 인증 흐름

```
Pencil 설치 → Pencil 활성화(이메일) → Claude Code CLI 설치 → Claude Code 인증(claude) → .pen 파일 생성 → 작업 시작
```

---

## 업데이트

확장 프로그램은 기본적으로 자동으로 업데이트됩니다. 수동으로 업데이트하려면:

### VS Code/Cursor 확장

1. 확장 패널 열기
2. Pencil 찾기
3. 업데이트가 있으면 Update 클릭

### 데스크톱 앱

- 업데이트가 있으면 앱 내에서 알림 표시
- 웹사이트에서 최신 버전 다운로드 후 설치

---

## 설치 트러블슈팅

| 문제 | 해결 방법 |
| --- | --- |
| 확장 설치 후 연결 안 됨 | Claude Code 로그인 확인 → 활성화 과정 완료 → IDE 재시작 |
| 활성화 이메일 미수신 | 스팸함 확인 → 다른 이메일 주소 시도 → 확장 재설치 |
| "Invalid API key" 또는 "Please run /login" | `claude` CLI 실행 후 인증 완료 → 충돌하는 인증 설정 제거 (환경변수 키, 커스텀 프로바이더) |

> 더 많은 문제와 해결 방법은 [Troubleshooting 가이드](https://docs.pencil.dev/troubleshooting)를 참고하세요.
