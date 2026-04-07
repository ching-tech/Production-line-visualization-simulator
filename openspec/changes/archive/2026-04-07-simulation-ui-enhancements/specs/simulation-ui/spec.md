## ADDED Requirements

### Requirement: 動作間隔 (GAP) 全域參數

模擬器 SHALL 提供一個全域「動作間隔」欄位 `p_action_gap`(預設 0,最小 0,步進 0.1 秒),代表每個動作型參數讀入時自動加上的緩衝秒數,以反映真實設備動作間的切換時間。

動作型參數(會套用 GAP)包含:換車、貨架旋轉、手臂 A/O/1/2/3/L/B 的所有取放與拍照、滑台 1/2 去/回程、清洗機步進移動 (`washMove`)、預烘機步進移動 (`preCT`)。

處理時間型參數(不套用 GAP)包含:`washClean`、`strCT`、`bakeOk`、`bakeOver`,以及計數型欄位(片數、格數、桿數、容量)。

#### Scenario: 預設 GAP 為 0
- **WHEN** 使用者載入頁面
- **THEN** `p_action_gap` 顯示 0,所有動作型參數等於使用者輸入值

#### Scenario: 調整 GAP 後即時生效
- **WHEN** 使用者把 `p_action_gap` 從 0 改成 0.5
- **THEN** `readP()` 立即重新執行,所有動作型參數值各自 +0.5,模擬中下一個被排程的動作會使用新值

### Requirement: 參數即時生效

任何 `PARAM_IDS` 中的欄位(除速度倍率 `spdS` 外)在 `input` 事件觸發時 SHALL 立即呼叫 `readP()`,讓參數變動可在模擬執行中即時生效;暫停後繼續 (`toggle()`) 也 SHALL 重新呼叫 `readP()`。

#### Scenario: 模擬中調整參數
- **WHEN** 模擬執行中,使用者修改 `p_armA_pick` 從 40 改為 50
- **THEN** `P.armAPick` 立即更新為 50(或 50+GAP),手臂A 的下一個動作會使用新值

### Requirement: 預烘機步進式

預烘機 SHALL 採線性步進模型,而非並行多格獨立計時:
- 內部資料結構為 `{slots:[null|物件 × N], phase:'ready'|'moving', stepTimer}`
- 入口固定為 `slots[0]`(視覺上畫在右側,靠近手臂2)
- 出口固定為 `slots[N-1]`(視覺上畫在左側,供手臂3 取料)
- `phase='moving'` 時,入口與出口皆禁止取放料
- 步進完成時,所有片往出口方向推一格,`slots[0]` 變空
- `p_pre_ct` 欄位語意為「每步移動時間」,屬於動作型參數,套用 GAP

#### Scenario: 觸發步進
- **WHEN** 預烘機處於 `ready` 階段,且 `slots[0] !== null` 且 `slots[N-1] === null`
- **THEN** `phase` 切換為 `moving`,`stepTimer` 設為 `preCT`

#### Scenario: 不會留空洞
- **WHEN** 預烘機處於 `[空, 有, 有, 空]` 狀態
- **THEN** 不觸發步進(因 `slots[0]===null`),等到 arm2 把 `slots[0]` 補滿才推進

#### Scenario: 不會丟片
- **WHEN** 預烘機處於 `[有, 有, 有, 有]` 狀態
- **THEN** 不觸發步進(因 `slots[N-1]!==null`),等到 arm3 把 `slots[N-1]` 取走才推進

#### Scenario: 移動中禁止取放
- **WHEN** 預烘機 `phase==='moving'`
- **THEN** 手臂2 不能放料到 `slots[0]`,手臂3 不能從 `slots[N-1]` 取料,兩者都顯示等待狀態

### Requirement: 手臂3 平衡策略 (drain/fill 互補)

手臂3 SHALL 在每個 simStep 計算 `canDrain`(平台2上層有料且滑台2 有空車)與 `canFill`(預烘出口有料且平台2下層未滿且非只出不進模式),並依下列優先序選擇動作:

1. 若 `canDrain` 為真 → 執行 drain (取平台2上 → 出料位)
2. 否則若 `canFill` 為真 → 執行 fill (取預烘 → 平台2下)
3. 否則 → 閒置

不得在 `plat2Upper>0` 但無 `freeCar2` 時讓手臂3 完全空轉。

#### Scenario: 滑台2 無空車時改去 fill
- **WHEN** `plat2Upper=2`、`freeCar2=null`、預烘出口有料、`plat2Lower=1`
- **THEN** 手臂3 執行 fill 而非空等

#### Scenario: 兩者皆可時優先 drain
- **WHEN** `plat2Upper>0` 且有 `freeCar2`,且預烘出口有料且 `plat2Lower<4`
- **THEN** 手臂3 優先執行 drain

### Requirement: 利用率延後起算 (排除暖機期)

各設備的利用率 (busy / (busy+idle)) SHALL 從「烤箱首次出料 + 手臂L 完成把整桿放到平台2 上層」這個事件開始累計,觸發前所有 `actionLog.busy` 與 `actionLog.idle` 不累計。觸發時系統 SHALL 將所有設備的累計值歸零並記錄事件 `info: 利用率起算`。

#### Scenario: 暖機期不累計
- **WHEN** 模擬剛啟動,首片尚未經過烤箱
- **THEN** 「各站利用率」卡片顯示 `--`,所有 `actionLog.busy/idle` 均為 0

#### Scenario: 首次出料後起算
- **WHEN** 手臂L 第一次完成「桿→上層」動作
- **THEN** `S.utilStarted` 設為 true,`S.utilStartTime` 記錄當下時間,事件記錄新增「利用率起算 @ X.X min」

### Requirement: 預設參數值更新

HTML 內建預設值 SHALL 反映實際使用情境:

| 欄位 | 預設值 |
|---|---|
| `p_cart_in_time` | 60 |
| `p_cart_out_time` | 60 |
| `p_bakeOver` | 200 |
| `p_action_gap` | 0 |
| `spdS` (速度倍率) | 999 |

其餘欄位維持 baseline 預設值。

#### Scenario: 重新整理頁面採用新預設
- **WHEN** 使用者按下「🔄 預設」或重新整理頁面
- **THEN** 各欄位顯示上述新預設值

### Requirement: 匯出 HTML 快照按鈕

控制列 SHALL 提供「📊 匯出」按鈕 (`exportSnapshot`),點擊後產生一份**自包含 HTML 檔**並觸發下載,內容包含:
- 來源 HTML 檔名(透過 `location.pathname` 取得)
- 匯出時間、模擬時間、利用率起算時刻
- 主畫面 Canvas (`#lc`) 與時間軸 Canvas (`#tc`) 的 PNG 截圖(base64 嵌入 `<img>`)
- 全部參數欄位的中文名、ID、值、單位
- 關鍵統計表(模擬時間、UPH、出/入料、過烘、烤箱、瓶頸 等)
- 各站利用率(busy 秒、idle 秒、百分比、長條圖)
- WIP 堆積數值
- 事件記錄最後 50 筆

檔名格式 SHALL 為 `snapshot_{原始 HTML 檔名}_{ISO 時間戳}.html`。

#### Scenario: 匯出後可直接以瀏覽器開啟
- **WHEN** 使用者按下「📊 匯出」
- **THEN** 瀏覽器下載一份 HTML 檔,雙擊即可在任何瀏覽器開啟,顯示截圖與所有表格,不需要任何額外軟體

#### Scenario: 檔名包含來源檔識別
- **WHEN** 使用者在 `simulation_ui - 手臂3手臂4分工.html` 中點匯出
- **THEN** 下載檔名以 `snapshot_simulation_ui - 手臂3手臂4分工_` 開頭,可用於區分不同程式檔的快照

### Requirement: 手臂3 / 手臂4 並行變體 (`simulation_ui - 手臂3手臂4分工.html`)

專案 SHALL 提供一個變體檔案 `simulation_ui - 手臂3手臂4分工.html`,將原本單一手臂3 拆成兩支可並行作業的手臂:

- **手臂3 (入料專責)**:只執行「取預烘出口 → 放平台2 下層」(`p_arm3_to_p2`)
- **手臂4 (出料專責)**:只執行「取平台2 上層 → 放滑台2 出料位」(`p_arm3_to_out`)
- 兩支手臂的決策邏輯為兩個獨立的 `if(!armBusy(...))` 區塊,可在同一個 simStep 內各自動作
- `STATION_DEFS` 中於 `platform2` 與 `armL` 之間插入 `{id:'arm4', name:'手臂4'}`
- `S.arms`、瓶頸分析 `armIds`、時間軸 `ARM_ROUTES` 均加入 `arm4`
- 視覺上 arm4 畫在 platform2 上方(靠近 conv2),arm3 畫在下方(靠近預烘機)

除上述差異外,變體檔的所有其他行為(預烘機步進、GAP、利用率起算、匯出按鈕、預設值)與主檔 `simulation_ui.html` **完全相同**。

#### Scenario: 手臂3 與手臂4 同時動作
- **WHEN** `plat2Upper=4`、有 `freeCar2`,且預烘出口有料、`plat2Lower<4`
- **THEN** 手臂3 開始 fill、手臂4 同時開始 drain,兩支手臂在時間軸上有重疊的 busy 區間

#### Scenario: 各自獨立計算利用率
- **WHEN** 模擬執行至穩態
- **THEN** 「各站利用率」卡片同時顯示「手臂3」與「手臂4」兩行,且兩者數值彼此獨立

#### Scenario: 變體檔的其他行為與主檔相同
- **WHEN** 在變體檔調整 `p_action_gap` 從 0 改為 0.5
- **THEN** 動作型參數套用 GAP 的規則、預烘機步進邏輯、利用率起算條件,皆與主檔行為一致
