## Context

`simulation_ui.html` 是單一 HTML 檔的產線模擬器,內含 CSS、HTML 版面與 JavaScript 模擬引擎 (參數讀取、事件迴圈、Canvas 繪圖、統計計算)。本變更僅為文件化,不修改程式碼。目標是把現有可觀察的 UI 行為與參數欄位忠實記錄到 spec,作為 baseline。

## Goals / Non-Goals

**Goals:**
- 由 `simulation_ui.html` 反向萃取出 UI 規格 (版面、輸入欄位、控制按鈕、統計卡片、底部面板)。
- 規格內容可作為未來變更比對的基準,讓任何 UI/參數調整都能以 OpenSpec delta 形式呈現。
- 規格層級停留在「使用者可見行為」,而非 JS 內部演算法。

**Non-Goals:**
- 不重寫或重構 `simulation_ui.html`。
- 不規範模擬引擎內部演算法 (排程邏輯、Canvas 繪圖細節)。
- 不涵蓋 `sim_test.js`、`sim_params.json` 內部結構 (僅在參數持久化處概略提及)。

## Decisions

- **單一 capability `simulation-ui`**:整個 UI 做為一個 capability,內部以多個 Requirement 區分版面、參數、控制、統計、底部面板。
  - 替代方案:拆成 `simulation-params`、`simulation-stats`、`simulation-controls` 等多個 capability。捨棄原因:目前只有單一檔案,過度切分反而增加維護成本。
- **參數欄位以表格列出**:在 spec 中以條列方式記錄欄位 ID、群組、預設值、單位,而非把每個欄位寫成獨立 Requirement。
  - 原因:~40 個欄位若每個都是 Requirement 會稀釋 spec 重點;以單一 Requirement + 表格 scenario 更易閱讀。
- **數值預設值以現況為準**:預設值直接抄錄目前 HTML 中的 `value=` 屬性,作為 baseline。

## Risks / Trade-offs

- [規格與程式碼漂移] → 後續任何 HTML 變更必須走 OpenSpec delta 流程,否則 spec 會過時。
- [預設值硬編碼於 spec] → 若預設值頻繁調整,spec 會不斷被修改;接受此成本以換取明確 baseline。
- [統計計算公式未進入 spec] → spec 只描述「顯示什麼」不描述「怎麼算」,計算正確性需仰賴 `sim_test.js`。

## Migration Plan

1. 建立 `openspec/specs/simulation-ui/spec.md` (透過 archive change 流程)。
2. 後續對 `simulation_ui.html` 的 UI/參數變更,皆建立新的 OpenSpec change 並產生 delta spec。

## Open Questions

- 參數持久化目前是 localStorage 還是寫入 `sim_params.json`?需在 specs 階段確認後再寫入 scenario。
- Excel 理論比較表的欄位內容尚未列入本次規格,是否要在後續變更中補上?
