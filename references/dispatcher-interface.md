# Dispatcher Interface — 與 task-dispatcher 的協作介面

## 呼叫關係說明

```
task-dispatcher Phase 2
        ↓ 若任務涉及系統設計/架構確認
  diagram-advisor（確認系統邊界）
        ↓ 輸出「系統邊界摘要」
task-dispatcher Phase 2（繼續 Squad 偵測）
```

diagram-advisor 是 task-dispatcher Phase 2 的**前置補充**，不是替代。
Squad 偵測與成員指派仍由 task-dispatcher 負責。

---

## task-dispatcher 應呼叫本 skill 的時機

task-dispatcher Phase 2 Squad 偵測時，若任務描述出現以下任一關鍵字，應提示使用者先跑 diagram-advisor：

- 「系統設計」「架構確認」「畫架構圖」
- 「跨 Squad 介面」「API 串接」「資料流向」
- 「哪個系統負責」「邊界在哪」「誰擁有這個資料」

提示語（task-dispatcher 輸出）：
```
這批任務涉及系統邊界確認，建議先用 /diagram-advisor 確認視角與邊界後再分工。
確認完成後，將「系統邊界摘要」貼回，task-dispatcher 會直接從摘要取 Squad 邊界，不需要重新偵測。
```

---

## diagram-advisor 輸出給 task-dispatcher 的格式

視角確認完成、圖的系統邊界釐清後，輸出「系統邊界摘要」：

```markdown
## 系統邊界摘要（for task-dispatcher Phase 2）

確認視角：系統架構圖
主體 Squad：[Sx 名稱]

### 內部系統節點
- [系統/後台名稱]（[功能說明]）
- [系統/後台名稱]（[功能說明]）

### 對外介面（已確認）
- → [Sx]：[資料/服務描述]（強相關）
- → [Sy]：[資料/服務描述]（強相關）
- ← [Sz]：[資料/服務描述]（上游依賴）

### 對外介面（待確認）
- ↔ [Sa]：[描述]（方向未確認）
- ↔ [Sb]：[描述]（強弱相關未確認）

### 上游依賴
- [Sx]：[依賴內容]（強 / 弱依賴）
- [Sy]：[依賴內容]（強 / 弱依賴）

### 注意事項（若有）
- [需要跨 Squad 確認的介面、或 task-dispatcher 分工時需注意的邊界]
```

task-dispatcher 收到此摘要後：
- 「內部系統節點」→ 確認任務歸屬 Squad 內部哪個系統
- 「對外介面（已確認）」→ 直接作為跨 Squad 依賴標注來源
- 「對外介面（待確認）」→ 標 `[待釐清]`，列入跨 Squad 協調清單
- **不需要重新做 Squad 偵測**，直接進入成員指派

---

## S4 團體系統的預設邊界摘要（快速參照）

當任務明確屬於 S4、且不需要重新畫圖時，可直接引用此預設摘要：

```markdown
## 系統邊界摘要（S4 團體系統，2026-03-23 版）

確認視角：系統架構圖
主體 Squad：S4 團體系統

### 內部系統節點
- SERP 後台（開團 / 線控 / 上架設定）
- 證照系統（辦證 → 送件 → 送簽 → 送回，獨立入口）
- CMS 內容後台（POI 景點 / 影音素材管理）
- 核心服務層：團體主檔服務（LionGroupRPM）、btour、PAK/PCM、JWI、LionGroupPDM
- 領隊庫存查詢（唯讀介面，資料主權在 S5）

### 對外介面（已確認，強相關）
- → S6：團體訂單拋轉（下單後由 S6 接手）
- → S5：導領原料（行程內容 + 旅客名單，S4 為上游）
- → S7：產品展示資料（NormGroupID + GroupID，觸發 S7 渲染）
- → S8：搜尋索引更新（開團後觸發）

### 對外介面（待確認）
- ↔ S13：AI 報價方向未確認（S4 提供 or S13 直接抓？）
- ↔ S16：零件化依賴方向未確認（S16 依賴 S4 的程度？）
- ↔ S17：採購觸發方式未確認（S4 是否主動觸發？）

### 上游依賴
- S17 SCM：供應商產品原料（強依賴）
- S6 訂單：配額回寫（強依賴）
- S5 導領：領隊主檔唯讀查詢（弱依賴）
- S1 機票：估價成本參考（弱依賴，手工帶入）
- S2 訂房：估價成本參考（弱依賴，手工帶入）

### DB 系統
LionGroupRPM · rpitin00 · gitpcm00 · LionGroupCMS · LionGroupPDM · PCM · btour · JWI
```

> 此摘要為 2026-03-23 版 squad-scope.json 推導值 + 本次 PM 訪談修正結果。
> S13/S16/S17 的介面方向需後續確認後更新。
