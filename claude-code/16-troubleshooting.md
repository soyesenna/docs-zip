# 16. 문제 해결 (Troubleshooting)

> **참조**: [Troubleshooting - Anthropic](https://docs.anthropic.com/en/docs/claude-code/troubleshooting)

---

## 목차

- [일반적인 설치 문제](#일반적인-설치-문제)
- [권한 및 인증](#권한-및-인증)
- [성능 및 안정성](#성능-및-안정성)
- [IDE 통합 문제](#ide-통합-문제)
- [마크다운 서식 문제](#마크다운-서식-문제)
- [추가 도움](#추가-도움)

---

## 일반적인 설치 문제

### Windows 설치 문제: WSL에서의 에러

**OS/플랫폼 감지 문제**: 설치 중 에러가 발생하면 WSL이 Windows `npm`을 사용하고 있을 수 있습니다.

```bash
# 해결 방법
npm config set os linux
npm install -g @anthropic-ai/claude-code --force --no-os-check
# 주의: sudo 사용 금지
```

**Node를 찾을 수 없는 에러**: `exec: node: not found`가 나타나면 WSL 환경이 Windows의 Node.js를 사용하고 있을 수 있습니다.

```bash
# 확인
which npm
which node
# /usr/로 시작해야 함 (/mnt/c/이면 Windows 버전)
```

**nvm 버전 충돌**: WSL과 Windows 양쪽에 nvm이 설치된 경우

```bash
# 쉘 설정 파일(~/.bashrc, ~/.zshrc)에 nvm 로드 확인
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

### Linux 및 Mac 설치 문제: 권한 또는 명령어 미발견 에러

**권장 해결 방법**: 네이티브 설치 사용

```bash
# macOS, Linux, WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex
```

**대안**: 기존 설치에서 마이그레이션

```bash
claude install  # ~/.claude/local/로 이동, 쉘 설정에 alias 추가
```

---

## 권한 및 인증

### 반복되는 권한 프롬프트

동일한 명령을 반복적으로 승인하는 경우 `/permissions` 명령으로 특정 도구를 허용할 수 있습니다.

### 인증 문제

```bash
# 1단계: 로그아웃
/logout

# 2단계: Claude Code 종료 후 재시작
claude

# 문제가 지속되면 인증 정보 삭제
rm -rf ~/.claude/auth
claude
```

---

## 성능 및 안정성

### 높은 CPU 또는 메모리 사용량

| 해결 방법 | 설명 |
|----------|------|
| `/compact` 사용 | 컨텍스트 크기 감소 |
| 정기적으로 재시작 | 주요 작업 사이에 Claude Code 재시작 |
| `.gitignore` 활용 | 대규모 빌드 디렉토리 제외 |

### 명령어 멈춤 또는 중단

1. `Ctrl+C`로 현재 작업 취소 시도
2. 응답이 없으면 터미널을 닫고 재시작

### 검색 및 탐색 문제

검색 도구, `@file` 멘션, 커스텀 에이전트, 커스텀 슬래시 명령어가 작동하지 않는 경우:

```bash
# 시스템 ripgrep 설치
# macOS
brew install ripgrep

# Ubuntu/Debian
sudo apt install ripgrep

# 환경변수 설정
# settings.json에 추가
{
  "env": {
    "USE_BUILTIN_RIPGREP": "0"
  }
}
```

### WSL에서 느리거나 불완전한 검색 결과

WSL에서 파일 시스템 간 작업 시 디스크 읽기 성능 저하가 발생할 수 있습니다.

| 해결 방법 | 설명 |
|----------|------|
| 더 구체적인 검색 | 디렉토리나 파일 유형 지정 |
| Linux 파일시스템 사용 | `/home/`에 프로젝트 위치 (`/mnt/c/` 대신) |
| 네이티브 Windows 사용 | WSL 대신 네이티브 Windows에서 실행 |

---

## IDE 통합 문제

### WSL2에서 JetBrains IDE 미감지

WSL2의 NAT 네트워킹 또는 Windows 방화벽이 연결을 차단할 수 있습니다.

**옵션 1: Windows 방화벽 설정 (권장)**

```bash
# WSL2 IP 주소 확인
hostname -I
```

```powershell
# PowerShell (관리자 권한)에서 방화벽 규칙 생성
New-NetFirewallRule -DisplayName "WSL2 Claude Code" -Direction Inbound -InterfaceAlias "vEthernet (WSL)" -Action Allow
```

**옵션 2: 미러드 네트워킹 전환**

`.wslconfig` 파일 (Windows 사용자 디렉토리)에 추가:

```ini
[wsl2]
networkingMode=mirrored
```

```powershell
# WSL 재시작
wsl --shutdown
```

### ESC 키가 JetBrains에서 작동하지 않는 경우

1. **Settings → Tools → Terminal**로 이동
2. 다음 중 하나 선택:
   - "Move focus to the editor with Escape" **체크 해제**
   - 또는 "Configure terminal keybindings" 클릭 후 "Switch focus to Editor" 단축키 **삭제**
3. 변경사항 적용

---

## 마크다운 서식 문제

### 코드 블록에 언어 태그 누락

<!-- 잘못됨: 언어 태그 없음 -->
```
console.log('hello')
```

<!-- 올바름: 언어 태그 포함 -->
```javascript
console.log('hello')
```

**해결 방법**:
1. Claude에게 "모든 코드 블록에 언어 태그를 추가해주세요"라고 요청
2. 후처리 훅으로 자동 감지 및 수정 설정
3. 생성 후 수동 검토

### 간격 및 서식 불일치

**해결 방법**:
1. Claude에게 서식 수정 요청
2. `prettier` 등 포매터 훅 설정
3. CLAUDE.md에 서식 요구사항 명시

### 마크다운 생성 모범 사례

- 요청에 명확하게 지정: "언어 태그가 포함된 적절한 마크다운"
- CLAUDE.md에 선호하는 마크다운 스타일 문서화
- 후처리 훅으로 자동 검증 및 수정

---

## 추가 도움

| 방법 | 설명 |
|------|------|
| `/bug` 명령어 | Claude Code 내에서 Anthropic에 직접 문제 보고 |
| GitHub 이슈 | 알려진 문제 확인 |
| `/doctor` | Claude Code 설치 상태 진단 |
| Claude에게 질문 | "Claude Code의 기능과 제한사항은 무엇인가요?" |
