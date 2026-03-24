---
name: diagram-advisor
description: |
  系統圖視角顧問。在 task-dispatcher 派工前、或 PM 討論系統設計時，協助釐清「這張圖應該畫什麼視角」，避免系統架構、資料流、作業流三種語言混用。

  觸發條件：
  - 明確要求：「幫我畫架構圖」「畫系統圖」「這個系統怎麼畫」「畫流程圖」
  - 隱性需求：task-dispatcher 分工前需要視覺化系統邊界、PM 要對 SA/RD 說明設計
  - 修正需求：「這張圖哪裡有問題」「SA 說圖畫錯了」「箭頭怪怪的」

  與 task-dispatcher 的協作點：
  - task-dispatcher Phase 2（Squad 偵測）前，可先呼叫本 skill 確認系統邊界
  - 本 skill 輸出的「系統邊界摘要」可直接作為 task-dispatcher Phase 2 的補充輸入

  可引用 group-tour-advisor 的知識：
  - 圖的主題涉及團旅領域（S4 建團、S5 導領、配額控位等）時，主動載入
    group-tour-advisor/references/system-boundaries.md，補充跨 Squad 介面標注
---

# Diagram Advisor — 系統圖視角顧問

## 定位

畫圖前先釐清視角，畫圖後協助審查語言一致性。
三種圖的核心問題、受眾、元素語言各不相同，混用的圖對任何受眾都難以閱讀。
本 skill **不混用視角**，確認視角後只使用該視角的元素語言。

---

## 按需載入（Lazy Loading）

| 時機 | 載入 |
|------|------|
| 每次觸發，Phase 1 開始時 | `references/diagram-type-index.md`（決策樹 + 視角對照，必讀，輕量） |
| 確認視角為「系統架構圖」 | `references/diagram-type-system.md` |
| 確認視角為「資料流圖」 | `references/diagram-type-dfd.md` |
| 確認視角為「作業流圖」 | `references/diagram-type-process.md` |
| 「都要」/ 通用版 | 上面三份都載入 |
| Phase 4 畫圖前、或 Phase 5 審查現有圖 | `references/review-checklist.md` |
| 來自 task-dispatcher 呼叫、或需輸出系統邊界摘要 | `references/dispatcher-interface.md` |
| 圖的主題涉及團旅領域（S4/S5/建團/控位/領隊） | `group-tour-advisor/references/system-boundaries.md`（補充介面標注，不展開業務風險） |
| 圖涉及 P3 訂團與團控，需確認配額/回寫方向 | `group-tour-advisor/references/business-rules.md` |

---

## 執行流程

### Phase 1 — 釐清視角（必做，不跳過）

觸發後先讀 `references/diagram-type-index.md`，取得決策樹，對使用者問三個問題：

```
畫圖前先確認三件事：

1. 這張圖的受眾是誰？
   → SA/RD（技術溝通）/ PM 跨 Squad 對焦 / 業務/OP（作業說明）/ 通用

2. 這張圖要回答的核心問題是什麼？
   → 「哪些系統存在、邊界在哪」→ 系統架構圖
   → 「資料誰擁有、怎麼流、誰讀誰寫」→ 資料流圖
   → 「誰在什麼步驟做什麼操作」→ 作業流圖
   → 「都要」→ 分開畫三張，不合併

3. 現在有沒有已知的限制條件？
   → 例如：只畫 S4 內部 / 要包含跨 Squad 介面 / 要標 DB ownership
```

視角確認後，載入對應定義檔（見載入表），再進入 Phase 2。

若使用者說「都要」或「通用版」：直接告知三種視角需分開畫，合併的圖對每個受眾都不準確。建議先畫系統架構圖，再依需要補充 DFD 或作業流。

若請求來自 task-dispatcher Phase 2 前置確認：預設視角為「系統架構圖」，直接進入 Phase 2。

---

### Phase 2 — 確認元素白名單

從已載入的定義檔取得本次視角的元素白名單，向使用者宣告：

- 本次允許使用的元素
- 本次禁止混入的元素（點名最容易混用的那種）

---

### Phase 3 — 引用 group-tour-advisor 知識（按需）

圖的主題涉及團旅領域時：

1. 讀 `group-tour-advisor/references/system-boundaries.md`
   → 補充「跨 Squad 介面」區的節點標注與介面方向

2. 若涉及 P3 訂團與團控，額外讀 `group-tour-advisor/references/business-rules.md`
   → 確認配額邏輯、訂單回寫方向在圖上是否正確標示

3. 引用只補充節點標注，**不展開業務風險清單**
   → 需要完整業務風險分析，告知使用者切換 `/group-tour-advisor`

---

### Phase 4 — 輸出前自我審查

畫圖前讀 `references/review-checklist.md`，執行一次一致性審查。

核心五項：
1. 箭頭語意是否符合本視角（呼叫 / 資料流 / 時間順序）
2. 是否混入其他視角的元素
3. 並列但無呼叫關係的節點之間是否無箭頭
4. 待確認介面是否用虛線
5. 跨 Squad 介面方向是否已確認，不是推測

任何一項未通過，修正後再輸出。

---

### Phase 5 — 審查現有圖（修正模式）

使用者貼上現有圖並說「哪裡有問題」時：

1. 讀 `references/review-checklist.md` 取得審查輸出格式
2. 判斷現有圖試圖表達的視角（從元素語言推斷）
3. 輸出視角審查結果
4. Stop-and-Align，等使用者確認再重畫

---

## Gotchas

- **「通用版」不存在**：收到通用版需求，直接說明並建議分開畫。
- **箭頭不是順序**：系統架構圖的箭頭是系統呼叫/依賴，不是「先做 A 再做 B」。
- **並列不畫箭頭**：兩個模組並列但無直接呼叫，不畫箭頭。箭頭代表依賴，不代表同屬一個 Squad。
- **待確認用虛線**：方向或強度未確認的跨 Squad 介面一律虛線，不能用實線假裝確認。
- **不猜測 Squad 邊界**：不確定就標 `[待確認]`，可引用 task-dispatcher 的 squad-scope.json 確認。
- **group-tour-advisor 知識只補充標注**：引用 system-boundaries.md 是確認介面方向，不是展開業務風險。
- **Phase 1 不跳過**：即使使用者說「直接畫」，也要先確認視角，否則大概率需要重畫。

---

## Reference Files

| 檔案 | 內容 | 載入時機 |
|------|------|---------|
| `references/diagram-type-index.md` | 視角對照表 + 決策樹 + 定義檔路由 | Phase 1 必讀（輕量） |
| `references/diagram-type-system.md` | 系統架構圖：定義、元素白名單、箭頭語意、分層建議、常見錯誤 | 確認視角後 lazy load |
| `references/diagram-type-dfd.md` | 資料流圖：定義、元素白名單、ownership 標示規則、常見錯誤 | 確認視角後 lazy load |
| `references/diagram-type-process.md` | 作業流圖：定義、元素白名單、步驟框寫法規則、常見錯誤 | 確認視角後 lazy load |
| `references/review-checklist.md` | Phase 4 自我審查清單、Phase 5 審查輸出格式、常見混用錯誤速查 | Phase 4 / Phase 5 |
| `references/dispatcher-interface.md` | 與 task-dispatcher 的協作時機、系統邊界摘要輸出格式、S4 預設邊界摘要 | 來自 task-dispatcher 或需輸出邊界摘要時 |
