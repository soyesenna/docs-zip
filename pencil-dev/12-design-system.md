# Pencil Dev 디자인 시스템 구성 가이드

> 원문: https://docs.pencil.dev/core-concepts/components, https://docs.pencil.dev/core-concepts/slots, Pencil MCP 서버 Design System 가이드 (get_guidelines로 조회)

Pencil의 Design System 가이드는 사이드바, 카드, 테이블, 탭, 드롭다운, 페이지네이션 등 일반적인 UI 패턴의 구성 방법과 아이콘, 슬롯, 화면 레이아웃 패턴, 디자인 토큰을 제공합니다.

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

## Understanding Slots

슬롯(Slot)은 컴포넌트 내부에서 자식 컴포넌트를 삽입할 수 있는 placeholder 프레임입니다. `slot` 속성에 추천 컴포넌트 ID 배열이 포함되어 있습니다.

### 슬롯 식별

컴포넌트를 읽을 때 `slot` 속성이 있는 프레임을 찾습니다:

```json
{
  "id": "slotId",
  "name": "Content Slot",
  "slot": ["recommendedComponentId1", "recommendedComponentId2"]
}
```

| 속성 | 설명 |
| --- | --- |
| `slot` | 추천 자식 컴포넌트 ID 배열 |
| `name` | 슬롯의 용도를 나타내는 이름 (예: "Content Slot", "Header Slot") |

### 슬롯 사용법

1. **부모 컴포넌트를 Insert** 하고 바인딩을 캡처
2. **슬롯에 자식을 Insert** — 경로: `parentBinding/slotId`
3. **추천 컴포넌트**를 `slot` 배열에서 참조 (다른 콘텐츠도 삽입 가능)

```javascript
sidebar = Insert(page, { type: "ref", ref: "sidebarComponentId", height: "fill_container" })
item1 = Insert(sidebar + "/contentSlotId", { type: "ref", ref: "sidebarItemId", descendants: { ... } })
item2 = Insert(sidebar + "/contentSlotId", { type: "ref", ref: "sidebarItemId", descendants: { ... } })
```

### 불필요한 슬롯 숨김

인스턴스에서 특정 슬롯이 필요 없으면 `enabled: false`로 숨깁니다:

```javascript
Update(card + "/contentSlotId", { enabled: false })
Update(card + "/actionsSlotId", { enabled: false })
```

---

## Icons

### 아이콘 라이브러리

`icon` 타입과 함께 다음 라이브러리를 사용할 수 있습니다:

| 라이브러리 | 스타일 | 예시 아이콘명 |
| --- | --- | --- |
| `lucide` | Outline, rounded | `home`, `settings`, `user`, `search`, `plus`, `x` |
| `feather` | Outline, rounded | `home`, `settings`, `user`, `search`, `plus`, `x` |
| `Material Symbols Outlined` | Outline | `home`, `settings`, `person`, `search`, `add`, `close` |
| `Material Symbols Rounded` | Rounded | `home`, `settings`, `person`, `search`, `add`, `close` |
| `Material Symbols Sharp` | Sharp corners | `home`, `settings`, `person`, `search`, `add`, `close` |

### 독립 아이콘 삽입

```javascript
// Lucide 아이콘
icon = Insert(container, { type: "icon", library: "lucide", icon: "settings", width: 24, height: 24, fill: "$--foreground" })

// Material Symbols (weight 속성 사용)
icon = Insert(container, { type: "icon", library: "Material Symbols Rounded", icon: "dashboard", width: 24, height: 24, fill: "$--foreground", weight: 400 })
```

| 속성 | 설명 |
| --- | --- |
| `type` | `"icon"` 고정 |
| `library` | 아이콘 라이브러리 이름 |
| `icon` | 아이콘 이름 |
| `weight` | Material Symbols 전용 굵기 (예: 400) |

### 컴포넌트 내 아이콘 오버라이드

컴포넌트에 포함된 아이콘은 `descendants`로 오버라이드합니다. 키는 노드 ID/경로 또는 고유한 descendant 이름입니다:

```javascript
descendants: {
  "iconNodeId": { icon: "settings" }
}
```

### 공통 아이콘 이름 매핑

| 동작 | Lucide / Feather | Material Symbols |
| --- | --- | --- |
| Home | `home` | `home` |
| Settings | `settings` | `settings` |
| User | `user` | `person` |
| Search | `search` | `search` |
| Add | `plus` | `add` |
| Close | `x` | `close` |
| Edit | `edit`, `pencil` | `edit` |
| Delete | `trash`, `trash-2` | `delete` |
| Check | `check` | `check` |
| Arrow right | `arrow-right` | `arrow_forward` |
| Chevron down | `chevron-down` | `expand_more` |
| Menu | `menu` | `menu` |
| Dashboard | `layout-dashboard` | `dashboard` |
| Folder | `folder` | `folder` |
| File | `file` | `description` |
| Calendar | `calendar` | `calendar_today` |
| Mail | `mail` | `mail` |
| Bell | `bell` | `notifications` |

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
    { type: "text", content: "Card Title", fill: "$--foreground", fontFamily: "$--font-primary", fontSize: 18, fontWeight: "600" },
    { type: "text", content: "Card description", fill: "$--muted-foreground", fontFamily: "$--font-secondary", fontSize: 14 }
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
const textBase = { type: "text", fontFamily: "$--font-primary" }
for (const metric of [
  { label: "Total Users", value: "12,543" },
  { label: "Revenue", value: "$48.2K" },
  { label: "Conversion", value: "7.4%" }
]) {
  metricCard = Insert(metrics, { type: "ref", ref: cardId, width: "fill_container" })
  header = Replace(metricCard + "/headerSlotId", { type: "frame", layout: "vertical", gap: 4, padding: 24, width: "fill_container" })
  Insert(header, { ...textBase, content: metric.label, fill: "$--muted-foreground", fontSize: 14 })
  Insert(header, { ...textBase, content: metric.value, fill: "$--foreground", fontSize: 32, fontWeight: "600" })
  Update(metricCard + "/contentSlotId", { enabled: false })
  Update(metricCard + "/actionsSlotId", { enabled: false })
}
```

---

## Screen Layout Patterns

일반적인 화면 레이아웃 구조입니다. 각 패턴은 보통 하나의 `batch_design` 호출에 해당합니다. `batch_design` 입력은 JavaScript이므로 배열, 루프, 객체 스프레드, 헬퍼 객체, 템플릿 리터럴을 사용하여 반복 노드를 생성할 수 있습니다.

### Pattern A: Sidebar + Content (Dashboard)

```
┌──────────┬────────────────────────────────┐
│          │                                │
│ Sidebar  │     Main Content Area          │
│  280px   │      fill_container            │
│          │                                │
└──────────┴────────────────────────────────┘
```

```javascript
screen = Insert(document, { type: "frame", name: "Dashboard", layout: "horizontal", width: 1440, height: "fit_content(900)", fill: "$--background", placeholder: true })
sidebar = Insert(screen, { type: "ref", ref: "sidebarId", height: "fill_container" })
main = Insert(screen, { type: "frame", layout: "vertical", width: "fill_container", height: "fill_container(900)", padding: 32, gap: 24 })
```

### Pattern B: Header + Content

```
┌────────────────────────────────────────────┐
│              Header Bar (64px)             │
├────────────────────────────────────────────┤
│            Content Area                    │
└────────────────────────────────────────────┘
```

고정 헤더 + 스크롤 가능한 콘텐츠:

```javascript
screen = Insert(document, { type: "frame", layout: "vertical", width: 1200, height: "fit_content(800)", fill: "$--background", placeholder: true })
header = Insert(screen, { type: "frame", layout: "horizontal", width: "fill_container", height: 64, padding: [0, 24], alignItems: "center", justifyContent: "space_between", strokeAlignment: "inner", stroke: "$--border", strokeWidth: { bottom: 1 } })
content = Insert(screen, { type: "frame", layout: "vertical", width: "fill_container", height: "fit_content(736)", padding: 32, gap: 24 })
```

### Pattern C: Two-Column Layout

```
┌─────────────────────┬─────────────┐
│                     │             │
│    Main (2/3)       │  Side (1/3) │
│   fill_container    │   360px     │
│                     │             │
└─────────────────────┴─────────────┘
```

메인 컬럼(flexible) + 사이드 컬럼(fixed):

```javascript
columns = Insert(content, { type: "frame", layout: "horizontal", width: "fill_container", height: "fill_container(900)", gap: 24 })
mainCol = Insert(columns, { type: "frame", layout: "vertical", width: "fill_container", height: "fit_content(900)", gap: 24 })
sideCol = Insert(columns, { type: "frame", layout: "vertical", width: 360, height: "fit_content(900)", gap: 24 })
```

### Pattern D: Card Grid

```
┌──────────┐ ┌──────────┐ ┌──────────┐
│  Card 1  │ │  Card 2  │ │  Card 3  │
└──────────┘ └──────────┘ └──────────┘
```

```javascript
cardGrid = Insert(container, { type: "frame", layout: "horizontal", width: "fill_container", gap: 16 })
for (const label of ["Overview", "Activity", "Revenue"]) {
  card = Insert(cardGrid, { type: "ref", ref: "cardId", width: "fill_container" })
  Update(card + "/titleId", { content: label })
}
```

---

## Common Compositions

화면 레이아웃 패턴과 결합하거나, 초기 구조 생성 후 독립 `batch_design` 호출로 사용할 수 있는 스니펫입니다. 반복 항목이 있는 구성에는 간결한 JavaScript 생성을 권장합니다.

### Page Header with Breadcrumbs + Actions

왼쪽에 브레드크럼, 오른쪽에 액션 버튼:

```javascript
pageHeader = Insert(main, { type: "frame", layout: "horizontal", width: "fill_container", justifyContent: "space_between", alignItems: "center" })
breadcrumbs = Insert(pageHeader, { type: "frame", layout: "horizontal", gap: 0, alignItems: "center" })
const crumbs = ["Dashboard", "Users"]
for (const [index, label] of crumbs.entries()) {
  if (index > 0) Insert(breadcrumbs, { type: "ref", ref: "breadcrumbSeparatorId" })
  Insert(breadcrumbs, { type: "ref", ref: index === crumbs.length - 1 ? "breadcrumbItemActiveId" : "breadcrumbItemId", descendants: { "labelId": { content: label } } })
}
actions = Insert(pageHeader, { type: "frame", layout: "horizontal", gap: 12 })
for (const [ref, label] of [["buttonOutlineId", "Export"], ["buttonPrimaryId", "Add User"]]) {
  Insert(actions, { type: "ref", ref, descendants: { "iconId": { enabled: false }, "labelId": { content: label } } })
}
```

### Form Layout

한 행에 두 필드, 이후 전체 너비 필드:

```javascript
card = Insert(container, { type: "ref", ref: "cardId", width: "fill_container" })
form = Insert(card + "/contentSlotId", { type: "frame", layout: "vertical", gap: 16, width: "fill_container" })
row = Insert(form, { type: "frame", layout: "horizontal", gap: 16, width: "fill_container" })
for (const label of ["First Name", "Last Name"]) {
  Insert(row, { type: "ref", ref: "inputGroupId", width: "fill_container", descendants: { "labelId": { content: label } } })
}
for (const field of [
  { ref: "inputGroupId", label: "Email" },
  { ref: "textareaGroupId", label: "Message" }
]) {
  Insert(form, { type: "ref", ref: field.ref, width: "fill_container", descendants: { "labelId": { content: field.label } } })
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

## Design Tokens

디자인 토큰 변수를 사용하면 일관성을 유지할 수 있습니다.

### Colors

| 토큰 | 용도 |
| --- | --- |
| `$--background` | 페이지 배경 |
| `$--foreground` | 기본 텍스트 |
| `$--muted-foreground` | 보조 텍스트, placeholder |
| `$--card` | 카드 배경 |
| `$--border` | 테두리, 구분선 |
| `$--primary` | 기본 액션, 브랜드 |
| `$--secondary` | 보조 요소 |
| `$--destructive` | 위험 액션 |

### Semantic Colors

| 상태 | 배경 | 전경 |
| --- | --- | --- |
| Success | `$--color-success` | `$--color-success-foreground` |
| Warning | `$--color-warning` | `$--color-warning-foreground` |
| Error | `$--color-error` | `$--color-error-foreground` |
| Info | `$--color-info` | `$--color-info-foreground` |

### Typography

| 토큰 | 용도 |
| --- | --- |
| `$--font-primary` | 제목, 레이블, 네비게이션 |
| `$--font-secondary` | 본문 텍스트, 설명, 입력 |

### Border Radius

| 토큰 | 용도 |
| --- | --- |
| `$--radius-none` | 테이블, 각진 컨테이너 |
| `$--radius-m` | 카드, 모달 |
| `$--radius-pill` | 버튼, 입력, 배지 |

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

### Spacing Consistency

- 항상 디자인 시스템의 기존 gap/padding 값 사용 (간격 참조 표 참고)
- 임의의 간격 값을 혼용하지 않기 — 확립된 스케일에서 선택
- 섹션 간에 일관된 수직 리듬 유지

### 색상 사용

- 항상 `$--variable` 토큰 사용. 하드코딩 금지
- 텍스트 가독성을 위한 충분한 대비
- 시맨틱 색상은 해당 목적으로만 사용

### 콘텐츠 밀도

- 과밀하지 않게 여백 확보
- 카드는 하나의 핵심 아이디어
- 테이블은 적절한 열 수 (일반적으로 4–7)

### Grounding Rules

- `get_editor_state`로 컴포넌트 목록을 확인한 후, 필요한 특정 컴포넌트만 `batch_get`으로 가져오기
- 주요 디자인 작업 후 `get_screenshot`으로 결과 확인
- 커스텀 프레임을 새로 만들기 전에 기존 컴포넌트를 먼저 사용
