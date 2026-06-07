# Pencil Dev 디자인 시스템 구성 가이드

> 원문: Pencil MCP 서버 Design System 가이드 (get_guidelines로 조회)

Pencil의 Design System 가이드는 사이드바, 카드, 테이블, 탭, 드롭다운, 페이지네이션 등 일반적인 UI 패턴의 구성 방법을 제공합니다.

---

## 공통 컴포넌트 패턴

### 명명 규칙

| 패턴 | 설명 |
| --- | --- |
| `Button/*` | 버튼 변형 |
| `Input/*` 또는 `Input Group/*` | 폼 입력 |
| `Card` | 카드 컨테이너 |
| `Sidebar` | 네비게이션 사이드바 |
| `Table` 또는 `Data Table` | 테이블 요소 |
| `Alert/*` | 피드백 알림 |
| `Modal/*` 또는 `Dialog` | 모달 대화상자 |

---

## Sidebar 구성

### 구조

```
Sidebar Component
├── Header (로고, 브랜드)
├── Content Slot ← 네비게이션 항목을 여기에 삽입
└── Footer (사용자 프로필, 설정)
```

### 구현

```javascript
sidebar = Insert(page, { type: "ref", ref: sidebarId, height: "fill_container" })
// 섹션 제목
Insert(sidebar + "/contentSlotId", {
  type: "ref", ref: sidebarSectionTitleId,
  descendants: { "labelTextId": { content: "Main Menu" } }
})
// 활성 항목
Insert(sidebar + "/contentSlotId", {
  type: "ref", ref: sidebarItemActiveId,
  descendants: { "iconId": { icon: "dashboard" }, "labelId": { content: "Dashboard" } }
})
// 기본 항목들
for (const [icon, label] of [["users", "Users"], ["settings", "Settings"]]) {
  Insert(sidebar + "/contentSlotId", {
    type: "ref", ref: sidebarItemDefaultId,
    descendants: { "iconId": { icon }, "labelId": { content: label } }
  })
}
```

---

## Card 구성

### 구조

```
Card Component
├── Header Slot ← 제목, 설명
├── Content Slot ← 메인 콘텐츠
└── Actions Slot ← 버튼
```

### 구현

```javascript
card = Insert(container, { type: "ref", ref: cardId, width: 480 })
// 헤더 커스텀 교체
Replace(card + "/headerSlotId", {
  type: "frame", layout: "vertical", gap: 4, padding: 24, width: "fill_container",
  children: [
    { type: "text", content: "Card Title", fill: "$foreground", fontFamily: "$font-primary", fontSize: 18, fontWeight: "600" },
    { type: "text", content: "Card description", fill: "$muted-foreground", fontFamily: "$font-secondary", fontSize: 14 }
  ]
})
// 콘텐츠 슬롯 설정
Update(card + "/contentSlotId", { layout: "vertical", gap: 16, padding: 24 })
Insert(card + "/contentSlotId", { type: "ref", ref: inputGroupId, width: "fill_container", descendants: { "labelId": { content: "Email" } } })
// 액션 슬롯
Update(card + "/actionsSlotId", { gap: 12, justifyContent: "end", padding: 24 })
Insert(card + "/actionsSlotId", { type: "ref", ref: buttonOutlineId, descendants: { "iconId": { enabled: false }, "labelId": { content: "Cancel" } } })
Insert(card + "/actionsSlotId", { type: "ref", ref: buttonPrimaryId, descendants: { "iconId": { enabled: false }, "labelId": { content: "Save" } } })
```

---

## Table 구성

### 계층 구조

```
Table (frame, vertical layout)
├── Table Header — 검색/필터 + 액션 버튼
├── Table Wrapper
│   ├── Header Row (frame, horizontal)
│   │   └── Cell (frame)
│   │       └── Content (text, label, button 등)
│   ├── Data Row 1
│   │   └── Cell → Content
│   └── ...
└── Table Footer — 행 수 + 페이지네이션
```

| 규칙 | 설명 |
| --- | --- |
| 엄격한 계층 | Table → Row → Cell(frame) → Cell Content |
| Cell은 항상 frame | 콘텐츠를 직접 Row에 넣지 마세요 |
| 2~3행씩 분할 | 행이 많으면 batch_design을 나누어 호출 |

### 열 너비 가이드

| 열 유형 | 일반적 너비 |
| --- | --- |
| 기본 식별자 (이름) | 200–250px |
| 이메일, URL | `fill_container` |
| 상태, 뱃지 | 100–120px |
| 날짜 | 120–150px |
| 액션 | 80–100px |
| 숫자 | 80–100px |

### 데이터 행 추가

```javascript
row1 = Insert(table, { type: "ref", ref: dataTableRowId, width: "fill_container" })
nameCell = Insert(row1, { type: "ref", ref: dataTableCellId, width: "fill_container" })
Insert(nameCell, { type: "text", content: "John Doe" })
statusCell = Insert(row1, { type: "ref", ref: dataTableCellId, width: 120 })
Insert(statusCell, { type: "ref", ref: labelSuccessId, descendants: { "textId": { content: "Active" } } })
```

---

## Tabs 구성

```javascript
tabs = Insert(container, { type: "ref", ref: tabsId, width: "fit_content" })
Insert(tabs, { type: "ref", ref: tabItemActiveId, descendants: { "labelId": { content: "General" } } })
Insert(tabs, { type: "ref", ref: tabItemInactiveId, descendants: { "labelId": { content: "Security" } } })
Insert(tabs, { type: "ref", ref: tabItemInactiveId, descendants: { "labelId": { content: "Billing" } } })
```

---

## Dropdown 구성

```javascript
dropdown = Insert(container, { type: "ref", ref: dropdownId, height: "fit_content" })
Insert(dropdown, { type: "ref", ref: searchBoxId })                 // 검색 (옵션)
Insert(dropdown, { type: "ref", ref: listDividerId })               // 구분선
Insert(dropdown, { type: "ref", ref: listTitleId, descendants: { "labelId": { content: "Actions" } } }) // 섹션 제목
Insert(dropdown, { type: "ref", ref: listItemCheckedId, descendants: { "labelId": { content: "Option A" } } })
Insert(dropdown, { type: "ref", ref: listItemUncheckedId, descendants: { "labelId": { content: "Option B" } } })
```

---

## Pagination 구성

```javascript
pagination = Insert(container, { type: "ref", ref: paginationId })
Insert(pagination + "/pageNumbersSlotId", { type: "ref", ref: paginationItemActiveId, descendants: { "labelId": { content: "1" } } })
Insert(pagination + "/pageNumbersSlotId", { type: "ref", ref: paginationItemDefaultId, descendants: { "labelId": { content: "2" } } })
Insert(pagination + "/pageNumbersSlotId", { type: "ref", ref: paginationItemDefaultId, descendants: { "labelId": { content: "3" } } })
Insert(pagination + "/pageNumbersSlotId", { type: "ref", ref: paginationItemEllipsisId })
Insert(pagination + "/pageNumbersSlotId", { type: "ref", ref: paginationItemDefaultId, descendants: { "labelId": { content: "10" } } })
```

---

## Metric Cards

```javascript
metrics = Insert(content, { type: "frame", layout: "horizontal", gap: 16, width: "fill_container" })
const textBase = { type: "text", fontFamily: "$font-primary" }
for (const metric of [
  { label: "Total Users", value: "12,543" },
  { label: "Revenue", value: "$48.2K" },
  { label: "Conversion", value: "7.4%" }
]) {
  metricCard = Insert(metrics, { type: "ref", ref: cardId, width: "fill_container" })
  header = Replace(metricCard + "/headerSlotId", { type: "frame", layout: "vertical", gap: 4, padding: 24, width: "fill_container" })
  Insert(header, { ...textBase, content: metric.label, fill: "$muted-foreground", fontSize: 14 })
  Insert(header, { ...textBase, content: metric.value, fill: "$foreground", fontSize: 32, fontWeight: "600" })
  Update(metricCard + "/contentSlotId", { enabled: false })
  Update(metricCard + "/actionsSlotId", { enabled: false })
}
```

---

## 간격 참조

| 컨텍스트 | Gap | Padding |
| --- | --- | --- |
| 화면 섹션 | 24–32 | — |
| 카드 그리드 | 16–24 | — |
| 폼 필드 (세로) | 16 | — |
| 폼 행 (가로) | 16 | — |
| 버튼 그룹 | 12 | — |
| 카드 내부 | — | 24 |
| 버튼 내부 | — | [10, 16] |
| 입력 내부 | — | [8, 16] |
| 페이지 콘텐츠 | — | 32 |
| 사이드바 항목 | 0 | [12, 16] |

---

## 버튼 계층

| 우선순위 | 변형 | 용도 |
| --- | --- | --- |
| 1 | Primary/Default | 기본 액션 (Save, Submit, Create) |
| 2 | Secondary | 대안 액션 |
| 3 | Outline | 3순위, Cancel, Back |
| 4 | Ghost | 인라인 액션, 네비게이션 |
| 5 | Destructive | Delete, Remove |

### 액션 정렬 컨벤션

| 위치 | 정렬 |
| --- | --- |
| 카드/모달 | 오른쪽 정렬 (`justifyContent: "end"`) |
| 폼 | 제출 버튼 오른쪽 |
| 툴바 | 기본 액션 왼쪽, 보조 액션 오른쪽 |
| 파괴적 + 취소 | 취소 왼쪽, 파괴적 오른쪽 |

---

## 디자인 원칙

### 시각적 계층

- 섹션당 하나의 명확한 초점
- 크기, 굵기, 색상으로 중요도 설정
- 기본 액션이 시각적으로 가장 두드러지게

### 정렬과 그리드

- 암묵적 그리드에 맞춰 정렬
- 컨테이너 내에서 일관된 가장자리 정렬
- 떠 있는 요소 피하기

### 색상 사용

- 항상 `$--variable` 토큰 사용. 하드코딩 금지
- 텍스트 가독성을 위한 충분한 대비
- 시맨틱 색상은 해당 목적으로만 사용

### 콘텐츠 밀도

- 과밀하지 않게 여백 확보
- 카드는 하나의 핵심 아이디어
- 테이블은 적절한 열 수 (일반적으로 4–7)
