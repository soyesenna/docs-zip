# Pencil Dev batch_design API

> 원문: Pencil MCP 서버 batch_design 문서 (get_editor_state로 조회)

`batch_design`은 JavaScript 스니펫으로 .pen 파일을 프로그래밍 방식으로 수정하는 MCP 도구입니다. 작은 단위로 나누어 여러 번 호출하는 것이 권장됩니다.

---

## API 개요

### 지원 함수

| 함수 | 설명 | 반환값 |
| --- | --- | --- |
| `Insert(parent, nodeData)` | 부모에 새 노드 삽입 | 노드 ID (string) |
| `Copy(path, parent, copyNodeData)` | 노드 복사 | 새 노드 ID (string) |
| `Update(path, updateData)` | 기존 노드 속성 업데이트 | void |
| `Replace(path, nodeData)` | 노드를 새 노드로 교체 | 새 노드 ID (string) |
| `Move(path, parent, index?)` | 노드를 다른 위치로 이동 | void |
| `Delete(path)` | 노드 삭제 | void |
| `Generate(nodeId, type, prompt)` | 이미지 Fill 생성 | void |
| `FindEmptySpace(input)` | 빈 영역 찾기 | `{ x, y, parentId? }` |

### 전역 변수

```javascript
const document: string;  // 문서 루트 ID
```

---

## Insert

부모의 자식 배열 끝에 새 노드를 삽입합니다.

```javascript
// 기본 삽입
button = Insert(page, { type: "frame", name: "Button", padding: 12, gap: 8 })

// 자식 추가
Insert(button, { type: "text", name: "Label", fontFamily: "Inter", content: "Click Me", fontSize: 14, fill: "#FFFFFF" })

// 컴포넌트 인스턴스 삽입
card = Insert(container, { type: "ref", ref: cardId, width: 480 })
```

| 규칙 | 설명 |
| --- | --- |
| 한 번에 하나 | 한 Insert 호출에 하나의 노드만 |
| name 필수 | 모든 노드에 `name` 속성 설정 |
| id 불가 | id는 자동 생성. 수동 설정 불가 |
| 반환값 | 삽입된 노드의 ID. 다음 호출에서 사용 |

---

## Copy

기존 노드를 복사합니다.

```javascript
// 기본 복사
pos = FindEmptySpace({ width: 1440, height: 1024, padding: 80 })
dashboardV2 = Copy("Xk9f2", document, { name: "Dashboard V2", x: pos.x, y: pos.y, placeholder: true })

// descendants와 함께 복사 (컴포넌트)
newCard = Copy(sourceCardId, container, {
  name: "Card V2",
  width: "fill_container",
  descendants: {
    "titleId": { content: "New Title" },
    "descId": { content: "New Description" }
  }
})
```

| 규칙 | 설명 |
| --- | --- |
| 재사용 노드 복사 | `reusable: true` 노드를 복사하면 연결된 `ref` 인스턴스 생성 |
| descendants | 복사와 동시에 하위 노드 속성 오버라이드 |
| 새 ID | 복사된 노드와 하위 노드는 모두 새 ID를 받음 |

> Copy 후에는 별도 Update로 descendants를 수정할 수 없습니다. Copy 호출에 descendants를 포함하세요.

---

## Update

기존 노드의 속성을 업데이트합니다.

```javascript
// 직접 노드 업데이트
Update(pageId, { placeholder: false })

// 인스턴스의 하위 노드 업데이트
Update(instanceId + "/childTitleId", { content: "New Title" })
Update(instanceId + "/childDescriptionId", { content: "New Description" })
```

| 규칙 | 설명 |
| --- | --- |
| children 변경 불가 | children은 Replace로만 변경 |
| id/type/ref 변경 불가 | 이 속성들은 변경할 수 없음 |
| 부분 업데이트 | 지정한 속성만 변경. 나머지는 보존 |

---

## Replace

노드를 완전히 새 노드로 교체합니다. 컴포넌트 인스턴스의 하위 노드를 교체할 때 이상적입니다.

```javascript
// 슬롯을 새 콘텐츠로 교체
customContent = Replace(instanceId + "/contentSlotId", {
  type: "frame", name: "Content", layout: "vertical", gap: 16, width: "fill_container"
})
Insert(customContent, { type: "text", content: "New Content", fontFamily: "Inter", fill: "#000000" })
```

| 규칙 | 설명 |
| --- | --- |
| 전체 교체 | x/y를 포함한 모든 속성이 교체됨 |
| 반환값 | 새 노드의 ID |
| 경로 | `instanceId/childId` 형식으로 중첩 접근 |

---

## Move

노드를 다른 위치로 이동합니다.

```javascript
// 다른 부모로 이동
Move("nodeId", newParentId)

// 특정 인덱스로 이동
Move("nodeId", parentId, 2)
```

---

## Delete

노드를 삭제합니다.

```javascript
Delete("nodeId")
```

---

## Generate (이미지)

노드에 AI 또는 스톡 이미지를 Fill로 적용합니다.

```javascript
// 이미지를 담을 컨테이너 먼저 생성
photoFrame = Insert(page, { type: "frame", name: "Photo", width: 400, height: 300 })

// AI 이미지 생성
Generate(photoFrame, "ai", "A modern office workspace with natural lighting")

// 스톡 이미지 (Unsplash)
Generate(photoFrame, "stock", "mountain sunset")
```

| 매개변수 | 설명 |
| --- | --- |
| `nodeId` | 이미지를 적용할 노드 ID |
| `type` | `"ai"` 또는 `"stock"` |
| `prompt` | AI: 상세 설명. Stock: 1~3개 키워드 |

> `image` 노드 타입은 없습니다. 이미지는 항상 Fill로 적용됩니다.

---

## FindEmptySpace

문서 루트에서 빈 영역을 찾습니다.

```javascript
// 기본 사용
pos = FindEmptySpace({ width: 1440, height: 1024, padding: 80 })
// pos = { x: 1600, y: 200 }

// 방향 지정
pos = FindEmptySpace({ width: 400, height: 800, direction: "right", padding: 80 })

// 기준 노드 지정 (체인)
pos = FindEmptySpace({ width: 1440, height: 1024, nodeId: previousScreenId })
```

| 매개변수 | 설명 | 기본값 |
| --- | --- | --- |
| `width` | 필요한 너비 | (필수) |
| `height` | 필요한 높이 | (필수) |
| `direction` | 검색 방향: `"top"`, `"right"`, `"bottom"`, `"left"` | `"right"` |
| `padding` | 다른 객체와의 최소 거리 | `0` |
| `nodeId` | 기준 노드 (체인 배치) | (없음) |

---

## batch_design 작성 규칙

### 스코프와 변수

```javascript
// 각 batch_design은 독립 스코프. 지역 변수는 공유 안 됨
// 변수를 유지하려면 const/let 대신 일반 할당 사용
myNode = Insert(document, { type: "frame", name: "My Frame" })
// myNode는 다음 batch_design 호출에서도 유지됨
```

### JavaScript 활용

```javascript
// 루프로 반복 구조 생성
nav = Insert(page, { type: "frame", name: "Nav", gap: 32, alignItems: "center", width: "fill_container" })
for (const label of ["Home", "About", "Contact"]) {
  Insert(nav, { type: "text", name: label, fontFamily: "Inter", fontSize: 14, fill: "$text-secondary", content: label })
}

// 스프레드로 스타일 재사용
const navItem = { type: "text", fontFamily: "Inter", fontSize: 14, fill: "$text-secondary" }
Insert(nav, { ...navItem, name: "Home", content: "Home" })
Insert(nav, { ...navItem, name: "About", content: "About" })
```

### 에러 처리

에러 발생 시 해당 batch_design의 모든 수정사항과 생성된 전역 변수가 롤백됩니다. 경고는 다음 batch_design 호출에서 수정하세요.

---

## 작업 흐름 예시

### 1. 컴포넌트 생성

```javascript
pos = FindEmptySpace({ width: 240, height: 96, direction: "top", padding: 80 })
metricCard = Insert(document, { type: "frame", name: "Metric", x: pos.x, y: pos.y, reusable: true, layout: "vertical", gap: 4, placeholder: true })
metricLabel = Insert(metricCard, { type: "text", name: "Label", fontFamily: "Inter", fontSize: 13, fill: "$text-secondary", content: "Label" })
metricValue = Insert(metricCard, { type: "text", name: "Value", fontFamily: "Inter", fontSize: 28, fill: "$text-primary", content: "0" })
Update(metricCard, { placeholder: false })
```

### 2. 화면 생성 + 컴포넌트 사용

```javascript
pos = FindEmptySpace({ width: 1440, height: 1024, padding: 80 })
page = Insert(document, { type: "frame", name: "Dashboard", x: pos.x, y: pos.y, layout: "vertical", width: 1440, padding: 40, gap: 24, clip: true, placeholder: true })
row = Insert(page, { type: "frame", name: "Metrics", gap: 16, width: "fill_container" })
for (const [label, value] of [["Orders", "1,284"], ["Revenue", "$48.2K"], ["Customers", "9,431"]]) {
  Insert(row, { type: "ref", ref: metricCard, name: label, width: "fill_container", descendants: { [metricLabel]: { content: label }, [metricValue]: { content: value } } })
}
Update(page, { placeholder: false })
```

### 3. 기존 화면 복사 + 수정

```javascript
pos = FindEmptySpace({ width: 1440, height: 1024, padding: 80 })
dashboardV2 = Copy("Xk9f2", document, { name: "Dashboard V2", x: pos.x, y: pos.y, placeholder: true, descendants: { "Jd6Ru": { fill: "#0F172A" }, "Pc2Ny/Gh9Kf": { content: "Reports" } } })
Update(dashboardV2, { placeholder: false })
Delete("Vn4kP")
```
