## 1. 驗證 Spec 與現況一致

- [ ] 1.1 對照 `simulation_ui.html` 確認所有參數欄位 ID、預設值、單位與 spec 中列出的內容一致
- [ ] 1.2 確認 `readP()` 中所有 getter 衍生屬性已寫入 spec
- [ ] 1.3 確認控制列按鈕 (`btnR`、重置、存檔/載入/預設、速度)、狀態文字 ID 與 spec 一致
- [ ] 1.4 確認右側統計卡片 ID (`s_time`、`s_uph_ss`、`s_out` …) 與 spec 一致
- [ ] 1.5 確認底部四欄區塊 (`#eLog`、`#tc`、`#ovenRodList`、`#excelTable`) 與 spec 一致

## 2. 解決 Open Questions

- [ ] 2.1 釐清 `saveParams()` / `loadParams()` 是寫入 localStorage 還是 `sim_params.json`,並把結論補進 spec 的對應 scenario
- [ ] 2.2 決定 Excel 理論比較表的欄位是否需要納入本次規格,如需要則新增 Requirement

## 3. 歸檔變更

- [ ] 3.1 執行 `openspec verify document-simulation-ui` 確認所有 artifact 通過驗證
- [ ] 3.2 執行 `openspec archive document-simulation-ui`,將 spec 同步到 `openspec/specs/simulation-ui/spec.md`
- [ ] 3.3 確認 `openspec/specs/simulation-ui/spec.md` 已建立且內容完整
