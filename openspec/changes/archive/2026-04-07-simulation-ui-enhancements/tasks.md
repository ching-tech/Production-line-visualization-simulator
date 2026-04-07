## 1. 驗證主檔 `simulation_ui.html` 行為

- [ ] 1.1 確認 `p_action_gap` 欄位存在,預設 0,step 0.1,min 0
- [ ] 1.2 確認 `readP()` 用 `a()` helper 對動作型參數套用 GAP,處理時間型不套用
- [ ] 1.3 確認預烘機資料結構為 `{slots:[], phase, stepTimer}`,並有步進邏輯
- [ ] 1.4 確認預烘機觸發條件嚴格為 `slots[0]!==null && slots[last]===null`
- [ ] 1.5 確認手臂3 平衡策略 (canDrain / canFill) 已套用
- [ ] 1.6 確認 `S.utilStarted` / `S.utilStartTime` 與利用率延後起算邏輯
- [ ] 1.7 確認預設值 (`cart_in_time=60`、`bakeOver=200`、`spdS=999` 等)
- [ ] 1.8 確認 `exportSnapshot()` 函式存在且按鈕已加入控制列

## 2. 驗證變體檔 `simulation_ui - 手臂3手臂4分工.html`

- [ ] 2.1 確認檔案存在且包含 `arm4` 在 `STATION_DEFS`
- [ ] 2.2 確認 arm3 與 arm4 邏輯為兩個獨立 if 區塊(可並行)
- [ ] 2.3 確認 `EQ.arm4` 繪圖座標已加入,FLOW 連線正確
- [ ] 2.4 確認瓶頸 `armIds` 與時間軸 `ARM_ROUTES` 已包含 `arm4`
- [ ] 2.5 確認其餘行為與主檔一致(GAP、預烘機、利用率、匯出)

## 3. 測試與比對

- [ ] 3.1 主檔 GAP=0 跑完整 8 hr,記錄穩態 UPH 與瓶頸,匯出快照
- [ ] 3.2 主檔 GAP=0.5 跑完整 8 hr,確認 UPH 不再有 cliff (應接近 GAP=0 的 1–4% 內)
- [ ] 3.3 變體檔 GAP=0 跑完整 8 hr,確認手臂3/4 同時忙碌、利用率各自顯示
- [ ] 3.4 變體檔與主檔在相同參數下比對 UPH 改善幅度

## 4. Archive 流程

- [ ] 4.1 執行 `openspec verify simulation-ui-enhancements`
- [ ] 4.2 確認 `document-simulation-ui` baseline 已先 archive(本變更的 spec 應建立在其之後)
- [ ] 4.3 執行 `openspec archive simulation-ui-enhancements`
- [ ] 4.4 確認 `openspec/specs/simulation-ui/spec.md` 已包含本變更新增的 Requirement
