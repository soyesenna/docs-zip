# 18. 모델 구성 (Model Configuration)

> **참조**: [Model configuration - Anthropic](https://docs.anthropic.com/en/docs/claude-code/settings) | [Bedrock & Vertex - Anthropic](https://docs.anthropic.com/en/docs/claude-code/bedrock-vertex)

---

## 목차

- [모델 구성 개요](#모델-구성-개요)
- [모델 선택](#모델-선택)
- [CLI를 통한 모델 변경](#cli를-통한-모델-변경)
- [설정 파일을 통한 모델 구성](#설정-파일을-통한-모델-구성)
- [환경변수를 통한 모델 구성](#환경변수를-통한-모델-구성)
- [서브에이전트 모델 구성](#서브에이전트-모델-구성)
- [클라우드 프로바이더별 모델 구성](#클라우드-프로바이더별-모델-구성)

---

## 모델 구성 개요

Claude Code는 사용할 AI 모델을 다양한 방법으로 구성할 수 있습니다. 모델 선택은 응답 품질, 속도, 비용에 직접적인 영향을 미칩니다.

---

## 모델 선택

| 모델 클래스 | 별칭 | 설명 | 적합한 용도 |
|------------|------|------|-------------|
| **Opus** | `opus` | 가장 강력한 모델 | 복잡한 분석, 아키텍처 설계, 심도 있는 코드 리뷰 |
| **Sonnet** | `sonnet` | 균형 잡힌 성능 | 일반적인 코딩 작업, 기능 구현, 디버깅 |
| **Haiku** | `haiku` | 빠르고 경제적 | 간단한 탐색, 빠른 질문, 경량 작업 |

> 기본 모델은 Claude Sonnet 클래스입니다. 대부분의 코딩 작업에 최적화되어 있습니다.

---

## CLI를 통한 모델 변경

### 대화형 모드에서 변경

```
> /model
# 사용 가능한 모델 목록에서 선택
```

### CLI 플래그로 변경

```bash
# 별칭 사용
claude --model sonnet
claude --model opus

# 전체 모델 이름 사용
claude --model claude-sonnet-4-20250514
```

---

## 설정 파일을 통한 모델 구성

### 글로벌 기본 모델 설정

```json
// ~/.claude/settings.json
{
  "model": "claude-sonnet-4-20250514"
}
```

### 프로젝트별 모델 설정

```json
// .claude/settings.json
{
  "model": "claude-opus-4-20250514"
}
```

---

## 환경변수를 통한 모델 구성

| 변수 | 용도 |
|------|------|
| `ANTHROPIC_MODEL` | 메인 세션에 사용할 모델 이름 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Haiku 클래스 모델 이름 재정의 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Sonnet 클래스 모델 이름 재정의 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Opus 클래스 모델 이름 재정의 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 서브에이전트에 사용할 모델 |

### 예시

```bash
# 메인 모델을 Opus로 설정
export ANTHROPIC_MODEL="claude-opus-4-20250514"

# 서브에이전트 모델을 Sonnet으로 설정
export CLAUDE_CODE_SUBAGENT_MODEL="claude-sonnet-4-20250514"

# Haiku 클래스 모델 재정의
export ANTHROPIC_DEFAULT_HAIKU_MODEL="claude-haiku-4-5-20251001"
```

---

## 서브에이전트 모델 구성

서브에이전트는 frontmatter에서 `model` 필드로 모델을 지정할 수 있습니다.

```markdown
---
name: security-reviewer
description: 보안 전문가 리뷰어
model: opus
tools: Read, Grep, Glob
---
```

| 설정값 | 설명 |
|--------|------|
| `sonnet` | 빠르고 효율적인 기본 모델 (기본값) |
| `opus` | 복잡한 분석에 적합 |
| `haiku` | 빠른 탐색 작업에 적합 |
| `inherit` | 메인 대화와 동일한 모델 사용 |

---

## 클라우드 프로바이더별 모델 구성

### Amazon Bedrock

```bash
export CLAUDE_CODE_USE_BEDROCK=1
# 모델별 리전 재정의는 Vertex 환경변수 참조
```

### Google Vertex AI

```bash
export CLAUDE_CODE_USE_VERTEX=1

# 모델별 리전 재정의
export VERTEX_REGION_CLAUDE_4_0_SONNET="us-east5"
export VERTEX_REGION_CLAUDE_4_0_OPUS="europe-west1"
export VERTEX_REGION_CLAUDE_4_1_OPUS="us-east5"
```

### 기본 모델 재정의

기본적으로 Claude Code는 `claude-3-7-sonnet-20250219`을 사용합니다. 다른 모델을 사용하려면:

```bash
# 글로벌 설정
claude config set -g model claude-sonnet-4-20250514

# 또는 환경변수
export ANTHROPIC_MODEL=claude-sonnet-4-20250514
```

> **참고**: Bedrock 및 Vertex AI에서 사용 가능한 모델은 프로바이더의 지역 및 설정에 따라 다를 수 있습니다. 각 프로바이더의 문서에서 지원 모델을 확인하세요.
