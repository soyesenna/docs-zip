# Pencil Dev 컴포넌트와 슬롯

> 원문: https://docs.pencil.dev/core-concepts/components, https://docs.pencil.dev/core-concepts/slots, Pencil MCP 서버 컴포넌트 지침

Pencil의 컴포넌트 시스템은 재사용 가능한 UI 요소를 정의하고, 인스턴스를 통해 여러 곳에서 활용하는 메커니즘입니다.

---

## 컴포넌트 개념

| 용어 | 설명 |
| --- | --- |
| **컴포넌트** | `reusable: true`가 설정된 노드. 다른 곳에서 재사용 가능 |
| **인스턴스** | `ref` 노드로 컴포넌트를 참조한 복제본 |
| **오버라이드** | 인스턴스에서 컴포넌트의 특정 속성을 재정의 |
| **슬롯** | 컴포넌트 내부에 동적 콘텐츠를 삽입하는 영역 |

---

## 컴포넌트 정의

### 기본 원칙

```javascript
// 컴포넌트 생성 (reusable: true)
metricCard = Insert(document, {
  type: "frame",
  name: "Metric",
  reusable: true,          // 이것이 컴포넌트임을 표시
  layout: "vertical",
  gap: 4,
  x: 100, y: 100
})
metricLabel = Insert(metricCard, { type: "text", name: "Label", fontFamily: "Inter", fontSize: 13, fill: "$text-secondary", content: "Label" })
metricValue = Insert(metricCard, { type: "text", name: "Value", fontFamily: "Inter", fontSize: 28, fill: "$text-primary", content: "0" })
```

| 규칙 | 설명 |
| --- | --- |
| ID 자동 생성 | 컴포넌트의 ID는 항상 무작위로 생성됨. 수동 설정 불가 |
| 캔버스 상단 배치 | 컴포넌트는 캔버스 상단에 배치, 화면은 아래쪽에 |
| 별도 batch_design | 컴포넌트 생성은 별도 batch_design 호출에서 수행 (ID를 받아야 함) |
| 작업 완료 후 placeholder 해제 | 프레임이 완성되면 즉시 `placeholder: false` |

### 컴포넌트 설계 원칙

| 원칙 | 설명 |
| --- | --- |
| 일반화 | 여러 곳에서 쓸 수 있도록 충분히 일반적으로 설계 |
| 중복 최소화 | 비슷한 컴포넌트를 여러 개 만들지 말고, 하나를 범용으로 |
| 기존 것 우선 사용 | 항상 새로 만들기 전에 문서의 기존 컴포넌트를 확인 |
| 파일 간 참조 불가 | 다른 파일의 컴포넌트는 사용할 수 없음 — 복사해서 사용 |

---

## 인스턴스 생성 (ref)

### 기본 사용

```javascript
// 컴포넌트를 인스턴스로 삽입
card = Insert(page, {
  type: "ref",
  ref: metricCard,     // 컴포넌트의 ID
  name: "Account Card"
})
```

### 인스턴스 오버라이드

#### 루트 속성 오버라이드

`ref` 객체에 직접 속성을 설정:

```javascript
card = Insert(page, {
  type: "ref",
  ref: metricCard,
  name: "Revenue Card",
  width: "fill_container"    // 루트 속성 오버라이드
})
```

#### 하위 노드 속성 오버라이드

`descendants` 맵으로 컴포넌트 내부 노드의 속성 변경:

```javascript
card = Insert(page, {
  type: "ref",
  ref: metricCard,
  name: "Revenue Card",
  descendants: {
    [metricLabel]: { content: "Revenue" },     // ID로 참조
    [metricValue]: { content: "$48.2K" }
  }
})
```

#### 중첩 인스턴스 오버라이드

중첩된 인스턴스의 하위 노드는 `instanceId/childId` 경로로 참조:

```javascript
// menu 컴포넌트 안의 button 인스턴스 안의 icon 노드
descendants: {
  "menuInstanceId/buttonInstanceId/iconNodeId": { icon: "settings" }
}
```

### 노드 숨김 (enabled: false)

인스턴스에서 특정 하위 노드를 숨길 수 있습니다:

```javascript
descendants: {
  "iconId": { enabled: false }    // 이 노드를 숨김
}
```

---

## 노드 교체 (Replace)

`descendants`에서 `type` 속성이 있으면 하위 노드를 완전히 교체:

```javascript
card = Insert(container, { type: "ref", ref: cardId, width: 480 })
Replace(card + "/headerSlotId", {
  type: "frame", layout: "vertical", gap: 4, padding: 24, width: "fill_container",
  children: [
    { type: "text", content: "Card Title", fill: "$foreground", fontFamily: "Inter", fontSize: 18, fontWeight: "600" },
    { type: "text", content: "Card description", fill: "$muted-foreground", fontFamily: "Inter", fontSize: 14 }
  ]
})
```

---

## 슬롯 (Slots)

### 슬롯 정의

컴포넌트의 특정 프레임에 `slot` 속성을 설정:

```javascript
sidebar = Insert(document, {
  type: "frame",
  name: "Sidebar",
  reusable: true,
  layout: "vertical"
})
contentSlot = Insert(sidebar, {
  type: "frame",
  name: "Content Slot",
  layout: "vertical",
  slot: ["sidebarItemId", "sidebarSectionTitleId"]  // 추천 자식 컴포넌트
})
```

### 슬롯에 콘텐츠 삽입

```javascript
// 사이드바 인스턴스의 슬롯에 항목 추가
sidebarInstance = Insert(page, { type: "ref", ref: sidebarId, height: "fill_container" })
item1 = Insert(sidebarInstance + "/contentSlotId", {
  type: "ref", ref: sidebarItemActiveId,
  descendants: { "iconId": { icon: "dashboard" }, "labelId": { content: "Dashboard" } }
})
item2 = Insert(sidebarInstance + "/contentSlotId", {
  type: "ref", ref: sidebarItemDefaultId,
  descendants: { "iconId": { icon: "settings" }, "labelId": { content: "Settings" } }
})
```

### 슬롯 숨김

인스턴스에서 특정 슬롯이 필요 없으면 `enabled: false`로 숨김:

```javascript
Update(card + "/contentSlotId", { enabled: false })
```

---

## Copy와 descendants

### Copy 시 주의사항

**Copy 후에는 별도 Update로 descendants를 수정할 수 없습니다.** Copy가 노드와 descendants에 새 ID를 부여하므로 원래 ID가 무효화됩니다.

```javascript
// ✅ 올바름: Copy 시 descendants를 함께 지정
newCard = Copy("sourceCardId", parentFrame, {
  name: "Card V2",
  descendants: {
    "titleId": { content: "New Title" },
    "descId": { content: "New Description" }
  }
})

// ❌ 잘못됨: Copy 후 원래 ID로 Update
newCard = Copy("sourceCardId", parentFrame, { name: "Card V2" })
Update("titleId", { content: "New Title" })  // 새 ID가 할당되어 실패
```

---

## 인스턴스 수정 패턴

### Update로 속성 변경

```javascript
// 인스턴스의 직접 하위 노드 수정
Update(instanceId + "/childTitleId", { content: "Account Details" })
Update(instanceId + "/childDescriptionId", { content: "Manage your settings" })
```

### Replace로 노드 교체

```javascript
// 슬롯을 새 콘텐츠로 교체
customContent = Replace(instanceId + "/contentSlotId", {
  type: "frame", name: "Content", layout: "vertical"
})
Insert(customContent, { type: "text", name: "Item 1", fontFamily: "Inter", content: "Item 1", fill: "#000000" })
```

### Insert로 슬롯에 추가

```javascript
// 슬롯에 새 자식 삽입
Insert(instanceId + "/contentSlotId", { type: "ref", ref: itemId, descendants: { "labelId": { content: "New Item" } } })
```

---

## 레이아웃 없는 환경에서의 인스턴스

인스턴스가 레이아웃을 사용하지 않는 객체 안에 있으면, x와 y를 명시적으로 설정해야 합니다:

```javascript
// 둘 다 설정하거나 아무것도 설정하지 마세요. 하나만 설정하면 안 됩니다.
Insert(parent, {
  type: "ref",
  ref: cardId,
  x: 200, y: 300,    // 항상 x, y를 함께 설정
  name: "Card"
})
```

---

## 오버라이드 전파 규칙

| 규칙 | 설명 |
| --- | --- |
| 직접 자식만 | 오버라이드는 지정된 노드에만 적용. 하위에는 전파 안 됨 |
| type 유지 | `Update`로 `id`, `type`, `ref`는 변경할 수 없음 |
| 변수 지원 | 오버라이드 값에도 변수 참조(`$var`) 사용 가능 |
