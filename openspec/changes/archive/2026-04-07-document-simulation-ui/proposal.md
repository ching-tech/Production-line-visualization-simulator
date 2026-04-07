## Why

`simulation_ui.html` 是強化機花型高空流道產線的單檔模擬器,目前僅以程式碼與註解的形式存在,缺乏正式規格文件。為了後續維護、驗證計算邏輯與新增功能,需要把現有 UI 的版面、參數、模擬輸出與互動行為整理為 OpenSpec 規格,作為未來變更的基準 (baseline)。

## What Changes

- 建立新的 capability `simulation-ui`,以 spec 形式記錄目前 `simulation_ui.html` 的功能:
  - **版面結構**: 3 欄 × 3 列 grid (header / 左參數面板 / 中央 canvas / 右統計面板 / 底部事件記錄、設備狀態、烤箱明細、Excel 比較)。
  - **參數面板**: 入料貨架、手臂 A/O/1/2/3/L/B、高空滑台 1/2、清洗機、強化機、預烘機、烤箱、出料貨架、模擬設定等共 ~40 個輸入欄位的 ID、預設值、單位與所屬群組。
  - **控制列**: 開始/停止、重置、速度倍率 (1–9999×)、存檔 / 載入 / 預設參數三顆按鈕、狀態文字。
  - **參數持久化**: 透過 `saveParams` / `loadParams` / `resetParams` 將參數存入 `sim_params.json` 對應的儲存機制 (localStorage 或檔案)。
  - **統計指標**: 模擬時間、穩態 UPH、出/入料數、過烘片數、最長烘烤、只出不進時間、換車次數/耗時、烤箱桿數/片數/最老桿齡、瓶頸設備、WIP 堆積、各站利用率、UPH 走勢圖。
  - **底部面板**: 事件記錄 (時間 + 類型 warn/info/ok + 描述)、設備即時狀態時間軸、烤箱桿位明細、Excel 理論比較表。
  - **計算衍生屬性**: 例如 `armAPerPiece = armAPick + armAPhoto/armAPhotoN`、`washStep = washClean + washMove`、`bakeOk/bakeOver` 由分鐘換算為秒等。
- 不變更任何程式碼或行為,僅新增規格文件。

## Capabilities

### New Capabilities
- `simulation-ui`: 強化機花型高空流道產線視覺化模擬器的 UI 版面、參數欄位、控制操作、統計輸出與底部面板規格。

### Modified Capabilities
(無)

## Impact

- 文件:
  - 新增 `openspec/specs/simulation-ui/spec.md` (在 specs 階段建立)。
- 程式碼: 無變更。
- 後續變更: 之後對 `simulation_ui.html` 的任何 UI/參數/統計變更,皆以此 spec 作為 delta 比對的基準。
