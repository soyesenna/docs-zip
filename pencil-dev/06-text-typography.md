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

고정 너비 + 고정 높이. 줄바꿈 수행, 오버플로우 가능.

| 속성 | 동작 |
| --- | --- |
| `width` | **필수 지정** |
| `height` | **필수 지정** |
| 줄바꿈 | 함 (오버플로우 가능) |
| 용도 | 높이를 명시적으로 제어해야 할 때만 사용 |

> `fixed-width-height`는 높이를 오버라이드해야 할 때만 사용하세요. 대부분의 경우 `fixed-width` + `fill_container`가 더 좋습니다.

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
| `2.0` | 200% (넓은 줄간격) |

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
