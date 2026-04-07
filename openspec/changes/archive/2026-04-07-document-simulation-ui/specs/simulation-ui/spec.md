## ADDED Requirements

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
