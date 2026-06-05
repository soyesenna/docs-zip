# 17. 출력 스타일 (Output Styles)

> **참조**: [Settings - Output styles - Anthropic](https://docs.anthropic.com/en/docs/claude-code/settings)

---

## 목차

- [출력 스타일 개요](#출력-스타일-개요)
- [사용 가능한 스타일](#사용-가능한-스타일)
- [설정 방법](#설정-방법)
- [커스텀 스타일](#커스텀-스타일)

---

## 출력 스타일 개요

출력 스타일(Output Styles)은 Claude Code의 응답 형식과 톤을 조정하는 설정입니다. `outputStyle` 설정을 통해 시스템 프롬프트를 조정하여 Claude의 응답 방식을 변경할 수 있습니다.

---

## 사용 가능한 스타일

| 스타일 | 설명 |
|--------|------|
| _(기본값)_ | 표준 응답 형식. 명확하고 간결한 코드 중심 응답 |
| `"Explanatory"` | 상세한 설명을 포함하는 응답. 각 코드 변경에 대한 이유와 배경을 설명 |

---

## 설정 방법

### settings.json을 통한 설정

```json
{
  "outputStyle": "Explanatory"
}
```

### /config 명령어를 통한 설정

```bash
> /config
# Output style → Explanatory 선택
```

### CLI 플래그를 통한 설정

```bash
claude --output-style "Explanatory"
```

---

## 커스텀 스타일

`--system-prompt` 또는 `--append-system-prompt`를 사용하여 응답 스타일을 더 세밀하게 제어할 수 있습니다.

### 예시: 한국어 응답 강제

```bash
claude --append-system-prompt "항상 한국어로 응답하세요. 코드 주석도 한국어로 작성하세요."
```

### 예시: 간결한 응답 강제

```bash
claude --append-system-prompt "응답은 최대한 간결하게 유지하세요. 불필요한 설명은 생략하고 코드에 집중하세요."
```

### 예시: 교육적 응답

```bash
claude --append-system-prompt "모든 응답에 교육적인 설명을 포함하세요. 초보 개발자도 이해할 수 있도록 단계별로 설명하세요."
```

---

## 참고

- `outputStyle`은 시스템 프롬프트를 조정하므로, `--system-prompt`로 전체 시스템 프롬프트를 덮어쓰면 무효화됩니다
- 프로젝트별로 다른 출력 스타일을 설정하려면 `.claude/settings.json`에 구성하세요
- `--append-system-prompt`는 기존 시스템 프롬프트에 추가되므로 `outputStyle`과 함께 사용 가능합니다
