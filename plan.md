# PoC App 規劃：GPX 匯入並同步到 Google 帳號（理想環境）

## 1. 目標與範圍
- 建立 Android PoC App，驗證「手機匯入 GPX 軌跡後，同步到 Google 帳號」的基本可行性。
- 本 PoC 僅在**理想環境**測試：
  - 使用者已登入 Google 帳號。
  - 網路穩定可用。
  - 不考慮離線暫存、斷線補傳、重試機制。
- 先不串接真實穿戴設備，使用 GPX 匯入模擬設備資料。

## 2. 使用者流程（MVP）
1. 使用者開啟 App 並以 Google 帳號登入。
2. 使用者選擇單一 GPX 檔案匯入。
3. App 解析 GPX（track/segment/point、時間、座標、高度）。
4. App 於畫面顯示匯入摘要（起訖時間、點數、距離）。
5. 使用者點擊「同步到 Google」。
6. App 直接呼叫 Google API 上傳本次軌跡。
7. 成功顯示 `SYNCED`；失敗顯示錯誤訊息（僅提示，不做重試排程）。

## 3. 系統架構建議（精簡）

### 3.1 模組分層
- **UI Layer**：
  - 匯入頁（選檔、解析結果）
  - 同步頁（同步按鈕、結果狀態）
- **Domain Layer**：
  - `ImportGpxUseCase`
  - `SyncTrackToGoogleUseCase`
- **Data Layer**：
  - `GpxParser`
  - `GoogleFitDataSource`（或目標 Google 運動資料 API）

### 3.2 同步策略（理想環境版）
- 同步採**前景即時執行**（使用者觸發）。
- 不使用 WorkManager 背景排程。
- 不設計離線佇列、不做指數退避、不做自動重試。
- 同步失敗僅回傳錯誤並提示使用者再次手動操作。

## 4. Google 同步方向
- PoC 以「Google 帳號下的運動資料同步」為目標，建議直接對接 **Google Fit / 對應健康資料介面**。
- 不納入 Google Drive 檔案儲存方案。

## 5. 資料模型（PoC 最小集）

### `tracks`
- `id` (UUID)
- `source`（`GPX_IMPORT`）
- `startTime`, `endTime`
- `distanceMeters`（可選）
- `status`（`IMPORTED` / `SYNCED` / `ERROR`）
- `errorMessage`（同步失敗時顯示）

### `track_points`
- `id`
- `trackId`
- `lat`, `lng`, `elevation`
- `timestamp`
- `seq`

> 不建立 `sync_tasks` 表，因為本 PoC 不做背景重試與排程。

## 6. 實作里程碑

### Milestone 1：專案骨架與登入
- 建立 Android 專案（Kotlin）。
- 接入 Google Sign-In。
- 完成基本頁面導覽。

### Milestone 2：GPX 匯入
- 實作檔案挑選（Storage Access Framework）。
- 實作 GPX parser。
- 顯示匯入摘要與路徑資料。

### Milestone 3：手動同步
- 串接 Google 運動資料 API。
- 完成「一鍵同步」流程。
- 顯示同步成功/失敗結果。

### Milestone 4：PoC 驗證
- 驗證不同 GPX 樣本可成功上傳。
- 整理 Demo 腳本與限制清單（僅理想環境可用）。

## 7. 測試案例建議（理想環境）
- 已登入 + 有網路：匯入 GPX 後可成功同步。
- GPX 格式不合法：可顯示解析錯誤。
- API 呼叫失敗：可顯示錯誤訊息，使用者可再次手動按同步。
- 同一檔案重複匯入：可提示或允許建立新紀錄（依 PoC 規則擇一）。

## 8. 風險與注意事項
- **範圍控制**：此版不處理離線與重試，Demo 時需先說明限制。
- **隱私與權限**：僅申請必要 Google 權限，清楚揭露資料用途。
- **資料一致性**：因無補償機制，失敗時由使用者手動再同步。

## 9. 建議技術清單
- Kotlin, Coroutines
- (可選) Room（只保留匯入與顯示需要的最小資料）
- Google Sign-In
- Google Fit / 對應健康資料 API

## 10. 後續擴充方向（PoC 之後）
- 新增離線佇列與背景同步（WorkManager）。
- 新增自動重試與冪等機制。
- 串接真實穿戴設備（BLE）取代 GPX 匯入。
