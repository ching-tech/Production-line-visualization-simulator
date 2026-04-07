# simulation-ui Specification

## Purpose
TBD - created by archiving change document-simulation-ui. Update Purpose after archive.
## Requirements
### Requirement: 整體版面結構

模擬器 SHALL 以單一 HTML 頁面呈現,使用 CSS Grid 將畫面切為:頂部標題列、左側參數面板、中央 Canvas 視覺區、右側統計面板、底部資訊列。Grid 設定 MUST 為 `grid-template-columns: 330px 1fr 300px` 與 `grid-template-rows: 44px 1fr 200px`,且整體高度填滿視窗 (100vh)。

#### Scenario: 開啟頁面顯示五大區塊
- **WHEN** 使用者在瀏覽器開啟 `simulation_ui.html`
- **THEN** 畫面同時顯示頂部控制列、左參數面板、中央 Canvas、右統計面板與底部面板,且不出現整頁捲軸

### Requirement: 頂部控制列

頂部控制列 SHALL 提供:標題、開始/停止按鈕 (`btnR`)、重置按鈕、速度倍率輸入 (`spdS`,範圍 1–9999,預設 10)、儲存參數、載入參數、恢復預設參數三顆按鈕,以及右側狀態文字 (`stTxt`,初始為「就緒」)。

#### Scenario: 點擊開始按鈕
- **WHEN** 使用者點擊「開始」按鈕
- **THEN** 模擬開始執行,按鈕狀態切換 (套用 `.on` 樣式)

#### Scenario: 點擊重置按鈕
- **WHEN** 使用者點擊「重置」按鈕
- **THEN** 模擬時間歸零,所有統計數值與事件記錄被清空

#### Scenario: 調整速度倍率
- **WHEN** 使用者在速度欄輸入 100
- **THEN** 模擬以 100 倍速度推進

### Requirement: 左側參數面板分組

左側參數面板 SHALL 將 ~40 個輸入欄位依產線設備分為以下群組,並依序顯示:
入料貨架、手臂A + 高空滑台1、手臂O、清洗機 (迴轉步進式)、手臂1、手臂2、強化機、預烘機、手臂3、手臂L + 烤箱、手臂B + 高空滑台2、出料貨架、模擬。

#### Scenario: 參數欄位 ID 與預設值
- **WHEN** 頁面載入完成
- **THEN** 下列欄位 ID 必須存在,且預設 `value` 與單位如下:
  - 入料貨架: `p_cart_pieces=32 片`、`p_rack_rotate_n=16 片`、`p_rack_rotate_time=30 秒`、`p_cart_in_time=120 秒`
  - 手臂A + 滑台1: `p_armA_photo=15 秒`、`p_armA_pick=40 秒`、`p_armA_photo_n=8 片`、`p_conv1_go=10 秒`、`p_conv1_back=10 秒`
  - 手臂O: `p_armO_to_wash=30 秒`、`p_armO_to_rack=30 秒`
  - 清洗機: `p_wash_clean=50 秒`、`p_wash_move=10 秒`、`p_wash_slots=13 格`
  - 手臂1: `p_arm1_move=30 秒`、`p_plat1_cap=4 片`
  - 手臂2: `p_arm2_to_str=35 秒`、`p_arm2_to_pre=30 秒`
  - 強化機: `p_str_ct=120 秒`、`p_str_n=4 台`
  - 預烘機: `p_pre_ct=10 秒`、`p_pre_slots=4 格`
  - 手臂3: `p_arm3_to_p2=35 秒`、`p_arm3_to_out=35 秒`
  - 手臂L + 烤箱: `p_armL_in=120 秒`、`p_armL_out=120 秒`、`p_batch=4 片`、`p_ovenRods=37 桿`、`p_bakeOk=120 min`、`p_bakeOver=140 min`
  - 手臂B + 滑台2: `p_conv2_go=10 秒`、`p_conv2_back=10 秒`、`p_armB_photo=15 秒`、`p_armB_pick=40 秒`、`p_armB_photo_n=8 片`
  - 出料貨架: `p_cart_out_pieces=32 片`、`p_rack_out_rotate_n=16 片`、`p_rack_out_rotate_time=30 秒`、`p_cart_out_time=120 秒`
  - 模擬: `p_simH=8 hr`、`p_extra=0 秒`

### Requirement: 參數計算衍生屬性

讀取參數時系統 SHALL 計算並提供下列衍生值:
- `armAPerPiece = armAPick + armAPhoto / armAPhotoN`
- `conv1Total = conv1Go + conv1Back`
- `armOTotal = armOToWash + armOToRack`
- `washStep = washClean + washMove` (穩態 CT 等於每步時間)
- `arm2Total = arm2ToStr + arm2ToPre`
- `arm3Total = arm3ToP2 + arm3ToOut`
- `armLTotal = armLIn + armLOut`
- `conv2Total = conv2Go + conv2Back`
- `bakeOk` 與 `bakeOver` SHALL 從分鐘轉為秒 (× 60)

#### Scenario: 預設參數下的衍生值
- **WHEN** 使用者載入頁面且未修改參數
- **THEN** `armAPerPiece` 計算為 40 + 15/8 = 41.875 秒,`bakeOk` 為 7200 秒

### Requirement: 參數持久化

系統 SHALL 提供三個按鈕呼叫 `saveParams()`、`loadParams()`、`resetParams()`,分別用於儲存目前參數、載入上次存檔、恢復 HTML 預設值。

#### Scenario: 儲存後重新載入
- **WHEN** 使用者修改參數後點擊「💾 存檔」,然後重新整理頁面並點擊「📂 載入」
- **THEN** 所有參數欄位回復為儲存時的值

#### Scenario: 恢復預設
- **WHEN** 使用者點擊「🔄 預設」
- **THEN** 所有參數欄位重設為 HTML `value` 屬性中的初始值

### Requirement: 中央視覺化 Canvas

中央區 SHALL 包含一個 `<canvas id="lc">` 撐滿整個區域,以及一個浮動 tooltip (`#tip`),供滑鼠移到設備節點時顯示明細。

#### Scenario: 滑鼠 hover 設備
- **WHEN** 使用者將滑鼠移到 Canvas 上的設備圖示
- **THEN** Tooltip 顯示該設備名稱與目前狀態

### Requirement: 右側統計面板

右側面板 SHALL 顯示以下統計卡片,每張卡片包含標題、主要數值與次要描述:
模擬時間 (`s_time` + `s_pct`)、穩態 UPH (`s_uph_ss`)、出料/入料 (`s_out` + `s_in`)、過烘 (`s_ob` + `s_ob2`)、最長烘烤 (`s_mb` + `s_mb2`)、只出不進 (`s_oo` + `s_oo2`)、換車 (`s_cc` + `s_cc2`)、烤箱 (`s_ov` + `s_ov2`)、瓶頸設備 (`s_bn` + `s_bn2`)、WIP 堆積 (`s_wip`)、各站利用率 (`s_util`),並在底部包含 UPH 走勢小圖 (`<canvas id="uc">`)。

#### Scenario: 模擬執行中更新統計
- **WHEN** 模擬執行 1 模擬小時
- **THEN** 各統計卡片數值依模擬狀態更新,模擬時間以 `H:MM:SS` 格式顯示

### Requirement: 底部面板四欄

底部面板 SHALL 以四欄等寬呈現:
1. 事件記錄 (`#eLog`):依時間順序列出事件,每列包含時間、類型 (warn/info/ok)、描述。
2. 設備即時狀態 (`<canvas id="tc">`,高 165px):以時間軸顯示各設備占用狀態。
3. 烤箱桿位明細 (`#ovenRodList`):列出每根桿目前狀態。
4. Excel 理論比較 (`#excelTable`):顯示與 Excel 計算的對照。

#### Scenario: 事件記錄新增警告
- **WHEN** 模擬中發生過烘事件
- **THEN** 事件記錄新增一筆,類型為 `warn` 並以紅色文字呈現

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

