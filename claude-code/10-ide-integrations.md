# 10. IDE 통합 (IDE Integrations)

> **참조**: [Add Claude Code to your IDE - Anthropic](https://docs.anthropic.com/en/docs/claude-code/ide-integrations)

---

## 목차

- [지원 IDE](#지원-ide)
- [IDE 통합 기능](#ide-통합-기능)
- [설치 방법](#설치-방법)
- [사용 방법](#사용-방법)
- [설정](#설정)
- [JetBrains 플러그인 설정](#jetbrains-플러그인-설정)
- [ESC 키 설정 (JetBrains)](#esc-키- 설정-jetbrains)
- [문제 해결](#문제-해결)
- [보안 주의사항](#보안-주의사항)

---

## 지원 IDE

Claude Code는 터미널이 있는 모든 IDE에서 작동하지만, 다음 IDE에 대해서는 전용 통합 기능을 제공합니다.

### Visual Studio Code 계열

| IDE | CLI 명령어 | 비고 |
|-----|-----------|------|
| **Visual Studio Code** | `code` | 공식 지원 |
| **Cursor** | `cursor` | 포크 지원 |
| **Windsurf** | `windsurf` | 포크 지원 |
| **VSCodium** | `codium` | 포크 지원 |

### JetBrains 계열

| IDE | 비고 |
|-----|------|
| **IntelliJ IDEA** | 전체 지원 |
| **PyCharm** | 전체 지원 |
| **Android Studio** | 전체 지원 |
| **WebStorm** | 전체 지원 |
| **PhpStorm** | 전체 지원 |
| **GoLand** | 전체 지원 |

---

## IDE 통합 기능

### 전체 기능 목록

| 기능 | 설명 | 단축키 |
|------|------|--------|
| **빠른 실행** | 에디터에서 바로 Claude Code 열기 | `Cmd+Esc` (Mac) / `Ctrl+Esc` (Windows/Linux) |
| **Diff 뷰어** | 터미널 대신 IDE Diff 뷰어에 코드 변경사항 표시 | `/config`에서 설정 |
| **선택 컨텍스트** | IDE의 현재 선택/탭이 Claude Code에 자동 공유됨 | 자동 |
| **파일 참조 단축키** | 파일 참조 삽입 (예: `@File#L1-99`) | `Cmd+Option+K` (Mac) / `Alt+Ctrl+K` (Windows/Linux) |
| **진단 공유** | IDE의 린트/구문 에러가 Claude에 자동 공유됨 | 자동 |

### 기능 상세 설명

#### 1. 빠른 실행

`Cmd+Esc` (Mac) 또는 `Ctrl+Esc` (Windows/Linux)를 누르면 Claude Code가 에디터에서 바로 열립니다. 또는 UI의 Claude Code 버튼을 클릭할 수도 있습니다.

#### 2. Diff 뷰어

코드 변경사항이 터미널 대신 IDE의 내장 Diff 뷰어에 표시됩니다. 이를 통해 변경사항을 더 직관적으로 확인할 수 있습니다.

#### 3. 선택 컨텍스트

IDE에서 현재 선택한 텍스트나 열려 있는 탭의 내용이 Claude Code에 자동으로 전달됩니다. 별도의 복사/붙여넣기 없이 컨텍스트를 공유할 수 있습니다.

#### 4. 파일 참조 단축키

`Cmd+Option+K` (Mac) 또는 `Alt+Ctrl+K` (Windows/Linux)를 누르면 파일 참조를 삽입할 수 있습니다.

```
@src/components/App.tsx#L10-50
```

#### 5. 진단 공유

IDE에서 발생하는 린트 에러, 구문 에러 등의 진단 정보가 Claude에 자동으로 공유됩니다. Claude가 작업 중 실시간으로 에러를 인식할 수 있습니다.

---

## 설치 방법

### VS Code (및 포크) 설치

1. VS Code 열기
2. 통합 터미널 열기
3. `claude` 실행 - 확장이 자동으로 설치됨

```
$ claude
```

### JetBrains 설치

1. JetBrains IDE 열기
2. 통합 터미널에서 `claude` 실행
3. 플러그인이 자동으로 감지되어 설치됨

---

## 사용 방법

### IDE 내부에서 사용

IDE의 통합 터미널에서 `claude`를 실행하면 모든 기능이 활성화됩니다.

```bash
$ claude
```

### 외부 터미널에서 사용

외부 터미널에서 `/ide` 명령어를 사용하여 Claude Code를 IDE에 연결할 수 있습니다.

```bash
$ claude
> /ide
```

이렇게 하면 외부 터미널에서도 IDE의 모든 통합 기능을 사용할 수 있습니다.

> **팁**: Claude가 IDE와 동일한 파일에 접근하려면 IDE 프로젝트 루트와 같은 디렉토리에서 Claude Code를 시작하세요.

---

## 설정

### Diff 도구 설정

IDE 통합은 Claude Code의 설정 시스템과 연동됩니다.

1. `claude` 실행
2. `/config` 명령어 입력
3. Diff 도구를 `auto`로 설정 → IDE 자동 감지가 활성화됨

```
> /config
# diff tool → auto로 설정
```

---

## JetBrains 플러그인 설정

JetBrains IDE에서 **Settings > Tools > Claude Code [Beta]**로 이동하여 플러그인을 설정할 수 있습니다.

### 일반 설정

| 설정 항목 | 설명 |
|----------|------|
| **Claude command** | Claude 실행에 사용할 커스텀 명령어 지정 (예: `claude`, `/usr/local/bin/claude`, `npx @anthropic/claude`) |
| **Suppress notification for Claude command not found** | Claude 명령어를 찾을 수 없다는 알림 건너뛰기 |
| **Enable using Option+Enter for multi-line prompts** (macOS만) | Option+Enter로 Claude Code 프롬프트에 줄바꿈 삽입. Option 키가 예기치 않게 캡처되는 문제가 있으면 비활성화 (터미널 재시작 필요) |
| **Enable automatic updates** | 플러그인 업데이트 자동 확인 및 설치 (재시작 시 적용) |

---

## ESC 키 설정 (JetBrains)

JetBrains 터미널에서 ESC 키가 Claude Code 작업을 중단하지 않는 경우 다음 설정을 변경하세요.

### 해결 방법

1. **Settings > Tools > Terminal**로 이동
2. 다음 중 하나를 선택:
   - "Move focus to the editor with Escape" **체크 해제**
   - 또는 "Configure terminal keybindings" 클릭 후 "Switch focus to Editor" 단축키 **삭제**
3. 변경사항 적용

---

## 문제 해결

### VS Code 확장이 설치되지 않는 경우

| 확인 사항 | 해결 방법 |
|----------|----------|
| 터미널 확인 | VS Code의 **통합 터미널**에서 실행 중인지 확인 |
| CLI 명령어 확인 | 각 IDE의 CLI 명령어가 설치되어 있는지 확인 |
| 권한 확인 | VS Code에 확장 설치 권한이 있는지 확인 |

CLI 명령어 설치 방법:
- `Cmd+Shift+P` (Mac) 또는 `Ctrl+Shift+P` (Windows/Linux)
- "Shell Command: Install 'code' command in PATH" 검색 및 실행

### JetBrains 플러그인이 작동하지 않는 경우

| 확인 사항 | 해결 방법 |
|----------|----------|
| 작업 디렉토리 | 프로젝트 루트 디렉토리에서 실행 중인지 확인 |
| 플러그인 활성화 | IDE 설정에서 Claude Code 플러그인이 활성화되어 있는지 확인 |
| IDE 재시작 | IDE를 완전히 재시작 (여러 번 필요할 수 있음) |
| 원격 개발 | JetBrains Remote Development의 경우 원격 호스트에 플러그인이 설치되어 있는지 확인 |

---

## 보안 주의사항

### IDE 자동 편집 권한

Claude Code가 자동 편집 권한이 활성화된 IDE에서 실행 중일 때, IDE 구성 파일을 수정할 수 있으며, 이는 IDE에 의해 자동으로 실행될 수 있습니다.

### 위험 요소

- 자동 편집 모드에서 Claude Code 실행 시 위험이 증가할 수 있음
- Bash 실행에 대한 Claude Code의 권한 프롬프트를 우회할 수 있음

### 권장 보안 조치

| 조치 | 설명 |
|------|------|
| **제한 모드 활성화** | VS Code Restricted Mode와 같은 IDE 보안 기능 활성화 |
| **수동 승인 모드 사용** | 편집에 수동 승인 모드 사용 |
| **신뢰할 수 있는 프롬프트만 사용** | Claude를 신뢰할 수 있는 프롬프트와 함께 사용 |

---

## 요약

Claude Code는 VS Code 계열과 JetBrains 계열 IDE에 대한 전용 통합 기능을 제공합니다. Diff 뷰어, 선택 컨텍스트 공유, 파일 참조 단축키, 진단 공유 등의 기능을 통해 IDE 내에서 원활하게 Claude Code를 사용할 수 있습니다. IDE에서 실행할 때는 보안 설정에 주의하세요.
