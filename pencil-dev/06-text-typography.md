# Pencil Dev 텍스트와 타이포그래피

> 원문: Pencil MCP 서버 텍스트 지침 (get_editor_state로 조회)

텍스트 노드는 가장 세밀한 제어가 필요한 요소입니다. 크기 결정 방식, 줄바꿈, 정렬 등의 규칙을 정확히 이해해야 합니다.

---

## 핵심 규칙

| 규칙 | 설명 |
| --- | --- |
| fill 필수 | 텍스트는 기본적으로 `fill`이 없어 보이지 않음. 반드시 설정 |
| 변수 참조 | `$` 접두사로 변수 참조: `fill: "$text-primary"` |
| 패밀리 | 모든 Google Fonts 사용 가능 |
| 크기 단위 | 픽셀 고정값 또는 변수 참조만. 퍼센트/뷰포트 단위 미지원 |

---

## textGrowth 모드

텍스트의 크기를 결정하는 3가지 모드:

### auto (기본값)

콘텐츠에 맞춰 자동 크기 조절. 줄바꿈 없이 항상 한 줄.

| 속성 | 동작 |
| --- | --- |
| `width` | 무시됨 (콘텐츠에 맞춤) |
| `height` | 무시됨 (콘텐츠에 맞춤) |
| 줄바꿈 | 안 함 |
| 용도 | 버튼 라벨, 태그, 뱃지, 짧은 텍스트 |

```javascript
// 텍스트가 콘텐츠 크기를 결정
btn = Insert(parent, { type: "frame", name: "Button", padding: 12, gap: 8 })
Insert(btn, { type: "text", name: "Label", fontFamily: "Inter", content: "Submit", fontSize: 14, fill: "$text-primary" })
```

### fixed-width

고정 너비 + 콘텐츠에 맞춘 높이. 줄바꿈 수행.

| 속성 | 동작 |
| --- | --- |
| `width` | **필수 지정** |
| `height` | 콘텐츠에 맞춰 자동 계산 |
| 줄바꿈 | 함 |
| 용도 | 제목, 설명, 문단 |

```javascript
// 부모가 너비를 결정
section = Insert(parent, { type: "frame", name: "Header", layout: "vertical", gap: 12, width: 400 })
Insert(section, { type: "text", name: "Title", textGrowth: "fixed-width", width: "fill_container", fontFamily: "Inter", content: "Dashboard", fontSize: 24, fill: "$text-primary" })
Insert(section, { type: "text", name: "Subtitle", textGrowth: "fixed-width", width: "fill_container", fontFamily: "Inter", content: "Manage your account settings", fontSize: 14, fill: "$text-secondary" })
```

### fixed-width-height

고정 너비 + 고정 높이. 줄바꿈 수행, 텍스트 콘텐츠가 수직으로 오버플로우됨.

| 속성 | 동작 |
| --- | --- |
| `width` | **필수 지정** |
| `height` | **필수 지정** |
| 줄바꿈 | 함 (텍스트 콘텐츠가 수직으로 오버플로우됨) |
| 용도 | 높이를 명시적으로 제어해야 할 때만 사용 |

> `fixed-width-height`는 높이를 오버라이드해야 할 때만 사용하세요. 대부분의 경우 `fixed-width` + `fill_container`가 더 좋습니다.

---

## 텍스트 스타일 속성

`TextStyle` 인터페이스에 정의된 텍스트 스타일 속성입니다. 모든 속성은 선택 사항이며, 미지정 시 기본값이 적용됩니다.

| 속성 | 타입 | 설명 |
| --- | --- | --- |
| `fontFamily` | `string \| Variable` | 글꼴 패밀리. 모든 Google Fonts 사용 가능 |
| `fontSize` | `number \| Variable` | 글꼴 크기 (픽셀) |
| `fontWeight` | `string \| Variable` | 글꼴 굵기. 예: `"400"`(Regular), `"600"`(Semi Bold), `"700"`(Bold) |
| `letterSpacing` | `number \| Variable` | 글자 간격 (픽셀 단위). 양수면 넓게, 음수면 좁게 |
| `fontStyle` | `string \| Variable` | 글꼴 스타일. 이탈릭 등 |
| `underline` | `boolean \| Variable` | 밑줄 표시 여부 |
| `strikethrough` | `boolean \| Variable` | 취소선(가로선) 표시 여부 |
| `href` | `string` | 텍스트에 연결된 하이퍼링크 URL |
| `lineHeight` | `number \| Variable` | 줄 간격 (fontSize의 비율). 자세한 내용은 아래 lineHeight 섹션 참조 |
| `textAlign` | `"left" \| "center" \| "right" \| "justify"` | 가로 정렬. 자세한 내용은 아래 텍스트 정렬 섹션 참조 |
| `textAlignVertical` | `"top" \| "middle" \| "bottom"` | 세로 정렬. 자세한 내용은 아래 텍스트 정렬 섹션 참조 |

### fontWeight

글꼴 굵기를 문자열로 지정합니다. 사용할 수 있는 일반적인 값은 다음과 같습니다.

| 값 | 이름 | 설명 |
| --- | --- | --- |
| `"100"` | Thin | 가장 얇은 굵기 |
| `"200"` | Extra Light | 매우 얇은 굵기 |
| `"300"` | Light | 얇은 굵기 |
| `"400"` | Regular | 기본 굵기 |
| `"500"` | Medium | 중간 굵기 |
| `"600"` | Semi Bold | 약간 굵은 굵기 |
| `"700"` | Bold | 굵은 굵기 |
| `"800"` | Extra Bold | 매우 굵은 굵기 |
| `"900"` | Black | 가장 굵은 굵기 |

> 글꼴이 해당 굵기를 지원해야 실제로 반영됩니다. 지원하지 않는 굵기를 지정하면 가장 가까운 굵기로 대체됩니다.

```javascript
// 제목에 Semi Bold 적용
Insert(section, {
  type: "text",
  name: "Heading",
  fontFamily: "Inter",
  fontWeight: "600",
  content: "제목 텍스트",
  fontSize: 24,
  fill: "$text-primary"
})
```

### letterSpacing

글자 사이의 간격을 픽셀 단위로 설정합니다.

```javascript
// 넓은 글자 간격 (대문자 타이틀에 유용)
Insert(parent, {
  type: "text",
  fontFamily: "Inter",
  content: "WELCOME",
  fontSize: 12,
  letterSpacing: 2,
  fill: "$text-secondary"
})

// 좁은 글자 간격
Insert(parent, {
  type: "text",
  fontFamily: "Inter",
  content: "Tight text",
  fontSize: 14,
  letterSpacing: -0.5,
  fill: "$text-primary"
})
```

### fontStyle

글꼴의 스타일 변형을 지정합니다. 이탈릭 등의 스타일을 적용할 수 있습니다.

```javascript
// 이탈릭 스타일 적용
Insert(section, {
  type: "text",
  fontFamily: "Inter",
  fontStyle: "italic",
  content: "강조된 텍스트",
  fontSize: 14,
  fill: "$text-secondary"
})
```

### underline / strikethrough

텍스트 장식 속성입니다. 부울 값으로 밑줄과 취소선을 각각 독립적으로 제어합니다.

```javascript
// 밑줄이 있는 링크 텍스트
Insert(parent, {
  type: "text",
  fontFamily: "Inter",
  content: "자세히 보기",
  fontSize: 14,
  underline: true,
  href: "https://example.com",
  fill: "$accent"
})

// 취소선이 있는 할인 가격
Insert(row, {
  type: "text",
  fontFamily: "Inter",
  content: "₩29,000",
  fontSize: 14,
  strikethrough: true,
  fill: "$text-secondary"
})
```

### href

텍스트 노드에 하이퍼링크 URL을 연결합니다. 클릭 시 해당 URL로 이동합니다.

```javascript
// 링크가 있는 텍스트
Insert(nav, {
  type: "text",
  fontFamily: "Inter",
  content: "문서 보기",
  fontSize: 14,
  href: "https://docs.example.com",
  fill: "$accent",
  underline: true
})
```

---

## 안티패턴

```javascript
// ❌ 부모가 레이아웃인데 픽셀 너비 사용
Insert(container, { type: "text", content: "abc", textGrowth: "fixed-width", width: 320, fontSize: 14 })
// ✅ fill_container 사용
Insert(container, { type: "text", content: "abc", textGrowth: "fixed-width", width: "fill_container", fontSize: 14 })

// ❌ textGrowth 없이 width 지정
Insert(container, { type: "text", content: "abc", width: 100, fontSize: 14 })
// ✅ textGrowth와 함께 지정
Insert(container, { type: "text", content: "abc", textGrowth: "fixed-width", width: "fill_container", fontSize: 14 })

// ❌ 텍스트 크기 추측 (항상 레이아웃으로 해결)
Insert(container, { type: "text", content: "Long text...", width: 276, height: 42, fontSize: 14 })
```

### 순환 의존성 안티패턴

부모가 `fit_content`(기본값)인데 모든 자식이 `fill_container`를 사용하면 순환 의존성이 발생합니다. 부모는 자식 크기의 합으로 자신의 크기를 결정하려 하지만, 자식은 부모 크기에 맞추려 하므로 크기가 0으로 붕괴됩니다.

```javascript
// ❌ 부모가 fit_content인데 자식이 모두 fill_container → 0 너비로 붕괴
badParent = Insert(screen, { type: "frame", layout: "vertical" })
Insert(badParent, { type: "text", textGrowth: "fixed-width", width: "fill_container", content: "텍스트", fontSize: 14, fill: "#000" })
// 결과: badParent의 너비가 0이 되고, 자식 텍스트도 보이지 않음

// ✅ 부모에 명시적 너비를 지정하여 순환 해결
goodParent = Insert(screen, { type: "frame", layout: "vertical", width: 400 })
Insert(goodParent, { type: "text", textGrowth: "fixed-width", width: "fill_container", content: "텍스트", fontSize: 14, fill: "#000" })

// ✅ 또는 부모를 fill_container로 설정하여 상위에서 크기를 물려받기
outerFrame = Insert(screen, { type: "frame", layout: "vertical", width: 400 })
innerFrame = Insert(outerFrame, { type: "frame", layout: "vertical", width: "fill_container" })
Insert(innerFrame, { type: "text", textGrowth: "fixed-width", width: "fill_container", content: "텍스트", fontSize: 14, fill: "#000" })
```

---

## 텍스트 정렬

### 가로 정렬 (textAlign)

`textGrowth`가 `"fixed-width"` 또는 `"fixed-width-height"`일 때만 효과:

| 값 | 설명 |
| --- | --- |
| `"left"` | 좌측 정렬 |
| `"center"` | 중앙 정렬 |
| `"right"` | 우측 정렬 |
| `"justify"` | 양쪽 정렬 |

> `textAlign`은 텍스트 박스의 위치를 바꾸지 않습니다. 박스 안에서의 텍스트 정렬만 담당. 박스 자체의 위치는 Flexbox 레이아웃으로 제어.

### 세로 정렬 (textAlignVertical)

`textGrowth`가 `"fixed-width-height"`일 때만 효과:

| 값 | 설명 |
| --- | --- |
| `"top"` | 상단 정렬 |
| `"middle"` | 중앙 정렬 |
| `"bottom"` | 하단 정렬 |

---

## lineHeight

`lineHeight`는 fontSize의 **비율**로 작동:

```javascript
// lineHeight = 1.0 → fontSize의 100%
// lineHeight = 1.5 → fontSize의 150%
// lineHeight = 0.0 → fontSize의 0%
```

| 값 | 효과 |
| --- | --- |
| `1.0` | 100% (글자 크기와 동일) |
| `1.5` | 150% (여유 있는 줄간격) |
| `2.0` | 200% (넓은 준간격) |

미지정 시 글꼴의 기본 lineHeight가 적용됩니다.

---

## 줄바꿈이 필요할 때

줄바꿈을 하려면 **반드시** `textGrowth`를 `"fixed-width"` 또는 `"fixed-width-height"`로 설정:

```javascript
// 줄바꿈이 필요한 텍스트
Insert(card, {
  type: "text",
  content: "이것은 긴 설명 텍스트입니다. 여러 줄로 표시되어야 합니다.",
  textGrowth: "fixed-width",
  width: "fill_container",
  fontSize: 14,
  fill: "$text-secondary",
  fontFamily: "Inter"
})
```

---

## 이모지

이모지도 일반 텍스트와 동일하게 `fill` 설정이 필요합니다:

```javascript
Insert(container, { type: "text", content: "🎉", fontSize: 24, fill: "#000000" })
```

---

## 텍스트 패턴 모음

### 부모가 크기를 결정 (제목, 설명)

```javascript
section = Insert(parent, { type: "frame", name: "Section", layout: "vertical", gap: 8, width: "fill_container" })
Insert(section, { type: "text", name: "Heading", textGrowth: "fixed-width", width: "fill_container", fontFamily: "Inter", content: "제목", fontSize: 24, fontWeight: "600", fill: "$text-primary" })
Insert(section, { type: "text", name: "Body", textGrowth: "fixed-width", width: "fill_container", fontFamily: "Inter", content: "설명 텍스트", fontSize: 14, fill: "$text-secondary" })
```

### 텍스트가 크기를 결정 (버튼 라벨, 뱃지)

```javascript
row = Insert(parent, { type: "frame", layout: "horizontal", gap: 8, alignItems: "center" })
Insert(row, { type: "icon", library: "lucide", icon: "check", width: 16, height: 16, fill: "$text-secondary" })
Insert(row, { type: "text", fontFamily: "Inter", content: "완료", fontSize: 14, fill: "$text-primary" })
```

### 중앙 정렬 텍스트

```javascript
// 부모에서 justifyContent와 alignItems로 중앙 배치
Insert(parent, {
  type: "frame", layout: "vertical", justifyContent: "center", alignItems: "center",
  width: "fill_container", height: "fill_container"
})
Insert(parent, { type: "text", fontFamily: "Inter", content: "중앙 텍스트", fontSize: 18, fill: "$text-primary" })
```
