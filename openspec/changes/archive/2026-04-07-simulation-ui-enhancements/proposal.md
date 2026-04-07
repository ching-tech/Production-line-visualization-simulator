## Why

在 baseline (`document-simulation-ui`) 之後,`simulation_ui.html` 已新增多項行為強化(預烘機步進式、手臂3 平衡策略、動作間隔 GAP、利用率延後起算、匯出 HTML 快照等),並衍生出一個變體檔 `simulation_ui - 手臂3手臂4分工.html` 把手臂3 拆成兩支可並行作業。需要把這些變更納入 OpenSpec,作為後續比對與維護的基準。

## What Changes

- **新增「動作間隔 GAP」全域參數** (`p_action_gap`):每個動作型參數讀入時自動 +GAP 秒。GAP 預設 0,可由使用者即時調整。
- **參數即時生效**:任何輸入欄位變動立即 `readP()`,模擬執行中下一個動作就套用新值。
- **預烘機改為步進式**:資料結構從「多格並行(timer/has/done)」改為「線性步進(slots[]/phase/stepTimer)」。入口 `slots[0]`、出口 `slots[last]`,移動時兩端皆不可取放。觸發條件 = 入口有料 AND 出口空。視覺上入口畫在右側(靠手臂2)、出口畫在左側。`p_pre_ct` 欄位語意從「每片 CT」改為「每步移動時間」並套用 GAP。
- **手臂3 平衡策略**:從「絕對優先 drain」改為「drain 不能執行時改去 fill,不空等」,避免在無 freeCar2 時 arm3 完全閒置造成 plat2Lower 永遠湊不滿 4 的非線性 throughput cliff。
- **利用率延後起算**:在「烤箱首次出料 + 手臂L 放到平台2 完成」之前不累計 busy/idle,觸發時把所有 actionLog 歸零。確保「各站利用率」反映穩態而非暖機。
- **參數預設值更新**:`p_cart_in_time` 120→60、`p_bakeOver` 140→200、`p_cart_out_time` 120→60、`p_action_gap` 預設 0、`spdS` 速度倍率預設 999。
- **匯出 HTML 快照按鈕** (`exportSnapshot`):匯出當下參數、統計、利用率、WIP、事件記錄與主畫面/時間軸 PNG 截圖,封裝成單一 HTML 檔(檔名含來源 HTML 名與時間戳),不需任何軟體即可在瀏覽器開啟比對。
- **新檔案 `simulation_ui - 手臂3手臂4分工.html`**:複製主檔並把手臂3 拆成兩支:
  - **手臂3**:只負責入料(預烘出口 → 平台2 下層)
  - **手臂4**:只負責出料(平台2 上層 → 高空滑台2)
  - 兩支手臂可同時並行作業,獨立計算利用率
  - 新增 `STATION_DEFS` 中 `arm4` 站、繪圖座標、瓶頸分析、時間軸路線

## Capabilities

### New Capabilities
(無)

### Modified Capabilities
- `simulation-ui`: 新增 GAP 參數、預烘機改步進式、手臂3 平衡策略、利用率延後起算、匯出按鈕、預設值更新;新增變體檔的手臂3/4 並行分工。

## Impact

- 程式碼: `simulation_ui.html` 多處修改;新增 `simulation_ui - 手臂3手臂4分工.html` 變體。
- 規格: `openspec/specs/simulation-ui/spec.md` 需以 delta 方式更新多個 Requirement(`MODIFIED` 預烘機、手臂3、參數欄位、控制列;`ADDED` 動作間隔 GAP、利用率起算條件、匯出按鈕、變體分工)。
- 行為差異: 預設參數下 UPH 不變;高 GAP 設定下,新策略避免吞吐量 cliff,UPH 從 ~30 回升到 ~50。
