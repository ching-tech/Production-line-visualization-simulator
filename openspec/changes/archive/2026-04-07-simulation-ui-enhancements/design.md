## Context

baseline 的 `simulation_ui.html` 用「絕對優先 drain」與「並行多格預烘機」的模型,測試發現在 GAP=0.5 秒這種小擾動下,UPH 從 ~43 暴跌到 ~30(30% 損失),屬於閾值現象 (threshold cliff),不是線性退化。同時也希望模擬結果更貼近真實機台行為(預烘機是步進輸送、預烘出口可被取走才能再步進),並且希望能匯出單次模擬的快照給使用者比對不同參數。

## Goals / Non-Goals

**Goals:**
- 修正 GAP=0.5 的非線性 UPH cliff,讓動作間隔的影響回到線性 (~1–4%)。
- 預烘機行為與真實步進輸送機台一致(線性步進、移動中不可取放、入出口固定)。
- 提供使用者「比較不同設定」的工作流(匯出 HTML 快照)。
- 提供「單臂 vs 雙臂」的對照變體,協助評估擴充手臂的投資效益。
- 既有測試案例(GAP=0、預設值)的 UPH 與瓶頸分析結果不變。

**Non-Goals:**
- 不重寫模擬引擎核心 (event loop / canvas drawing)。
- 不調整強化機、烤箱、清洗機本身的演算法。
- 不在 spec 中描述每個 JS 函式的內部實作,只描述使用者可觀察行為。

## Decisions

- **GAP 統一在 `readP()` 入口處套用**:用 helper `a()` = `g() + GAP`,避免在 simStep 內到處改。動作型參數 +GAP,處理時間型(strCT/preCT 之前/bakeOk/bakeOver/washClean)不加。
  - 替代:在每個 armStart 呼叫處 +GAP。捨棄原因:程式碼侵入性太大,易漏改。
  - 預烘機改為步進式後,`preCT` 從處理時間變成動作時間,改為套用 GAP。
- **預烘機改用線性陣列 `slots[]`**:`null`/物件 表示空/有料,`phase` 區分 `ready` / `moving`。
  - 替代:使用 circular buffer 模擬鏈條繞圈。捨棄原因:在預設參數下兩者行為等價,線性陣列實作更簡單。
- **預烘機觸發條件 = 入口有料 AND 出口空**:這個雙條件確保不會丟片(出口必空)也不會留洞(入口必滿)。
  - 替代:出口空就推。捨棄原因:會在「[空,有,有,空]」狀態時製造空洞。
- **手臂3 平衡策略**:`canDrain = upper>0 && freeCar2`、`canFill = preExit有料 && lower<4 && !ooMode`。drain 不能做就做 fill,而非空等。
- **利用率延後起算**:在「烤箱首次出料 + 手臂L 完成放到平台2」這個事件點才開始累計,並把已累計的歸零。理由:暖機期會嚴重稀釋穩態利用率,影響瓶頸判斷。
- **匯出格式選 HTML 而非 JSON/CSV**:HTML 可內嵌 PNG 截圖、用瀏覽器直接開、不需任何軟體。檔名含來源 HTML 名,方便比對是哪支程式跑出來的。
- **手臂3/4 分工放在獨立檔案而非開關**:兩種模型差異夠大,獨立檔案比 if 分支更清楚,也避免 spec 裡有條件式 Requirement。

## Risks / Trade-offs

- [預烘機 latency 變長] → 線性步進讓首片從入到出要走 4 步 = 40s,比並行多格 (10s) 多 30s。但 throughput 沒變,對穩態 UPH 無影響,只影響暖機時間 ~30 秒。
- [手臂3 平衡策略可能造成片在 plat2Upper 滯留] → drain 被延後直到 freeCar2 來,中間時間 plat2Upper 會持續有料,但因為 armL 出爐條件是「upper===0」,armL 也會跟著等。實測 UPH 改善遠大於這個副作用。
- [兩個檔案需同步維護] → 變體檔之後的修改要兩邊都改。緩解:只在差異點(arm3/arm4 邏輯與繪圖)有少量分歧,其他完全相同。
- [匯出 HTML 包含 base64 PNG,檔案較大 (~200KB–1MB)] → 對單次比對可接受;若大量匯出可能擠硬碟。緩解:檔名含時間戳便於管理。
- [參數即時 readP 可能在模擬中產生中途切換的奇怪狀態] → 已排程進行的動作仍用舊 duration,只有新動作用新值;對 UPH 統計影響極小。

## Migration Plan

1. baseline (`document-simulation-ui`) 已 archived,既有 spec 在 `openspec/specs/simulation-ui/`。
2. 本次以 delta 形式更新該 spec(MODIFIED 既有 Requirement、ADDED 新行為)。
3. 變體檔 `simulation_ui - 手臂3手臂4分工.html` 在 spec 中以 ADDED Requirement「手臂3/4 並行變體」描述,不另開 capability。
4. archive 後,`openspec/specs/simulation-ui/spec.md` 即同時涵蓋兩個檔案的行為。

## Open Questions

- 之後若有更多變體檔(例如 conv2 改 3 台車),是否仍合併在同一 capability,或拆出 `simulation-ui-variants` capability?
- 匯出 HTML 是否要加入 diff 工具的支援(例如同時匯出純文字版本以利 diff)?
