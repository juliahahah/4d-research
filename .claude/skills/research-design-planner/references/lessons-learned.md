# Lessons learned(研究實驗規劃)

使用這個 skill 時踩過的坑的流水帳 —— 讓同一個錯誤 / 被推遲的需求 / 漏掉的可比性檢查,不要在下一輪實驗重演。條目跨專案累積;這份是 skill 通用的,不放專案特定事實(那些屬於 `.claude-research/landscape.md` / `conventions.md`)。

## 何時讀

- **Phase 1(鎖契約)**:定 clarification 問題與可調參數前,掃過與本輪性質相符的條目。一條過去的教訓常會變成現在該先鎖死的一個參數,而不是跑到一半才冒出來的意外。
- **Phase 3(自審)**:像 reviewer/PI 審 experiment-plan 時,檢查有沒有哪條的「下次怎麼避免」適用於這份計畫卻還沒處理。

## 何時寫

當一輪實驗跑下去(或使用者回報結果)後,冒出 experiment-plan 沒預料到的事 —— 被推遲的設定、混淆變因、不可比的 baseline、白燒的 compute —— 就在**繼續往下之前**補一條。簡短:發生什麼、根因、什麼具體步驟能提前擋掉。最新在上。

---

## 條目模板

```
### <短標題> (<專案或情境>, <日期>)
**發生什麼:** ...
**根因:** ...
**下次怎麼避免:** ...(理想上:該回饋到哪個 Phase/步驟)
```

---

### 多節點同時死 ≠ 叢集事件 —— 先查磁碟配額;checkpoint 要進磁碟預算

**發生什麼:** 三個 job 在三個不同節點同秒被殺、重交全部 0 秒 FAILED 連 log 都沒有。第一直覺判成「叢集級事件」,實際是 **deepspeed zero3 每格 checkpoint ~120GB × 10+ 格把 /work 配額(1.5T)塞爆**:跨過配額那一瞬間所有寫入者同時死(看起來就像叢集事件),之後 prolog 建 log 檔失敗 → 0 秒死。

**根因:** 批次設計時只算了 GPU 時數,沒算 **磁碟預算**(ckpt 大小 × 格數);且「同時多點失敗」的診斷直覺偏向外因。

**下次怎麼避免:** ①批次開跑前算磁碟預算:單 ckpt 實測大小 × 格數 vs 剩餘配額;②eval 完立即自動刪 ckpt(結果已記錄、可重生)——寫進 eval wrapper 而非靠手動;③0 秒 FAILED / 無 log 的第一檢查是 `df` + `touch` 寫入測試,不是怪叢集。

### 官方 loss 權重是對「特定 head / 資料」校準的 —— 換 head 或換資料後必須重測 raw 量級

**發生什麼:** 矩陣沿用官方 per-task λ(camray=1e-5)。實測自己資料上的 unweighted loss 才發現:fork 把 camray teacher head 換成 raw-rays 版後,raw 量級從官方校準時的 ~10³ 掉到 ~0.13 —— 照官方 λ 跑,該格貢獻 ~1e-6,**等於沒蒸,整格白跑**。同時官方 ld=0.01 在新資料上會讓 LD(raw~21)壓過 SFT 3 倍。

**根因:** loss 權重是「權重 × raw 量級 ≈ 目標貢獻」的乘積校準;换 teacher head、換資料、換 loss 型式都會改 raw 量級,沿用舊 λ 就失準(太小=無效格、太大=爆掉淹沒主 loss)。

**下次怎麼避免:** 任何 sweep 開跑前,先用 1 個短 run 把**每個 loss 項的 unweighted 量級**在自己的資料/head 上實測出來(好 repo 會把 unweighted 值印進 log;沒有就加),再按 `λ = T / raw` 配權重(T ≈ 主 loss 的一半以內),並在首批 log 驗證:單項貢獻不超過總 loss 一半、主 loss 仍在降。把「實測 raw → λ」的表寫進 experiment-plan。

### 旗標「留空 ≠ 關閉」:批次開矩陣前,逐格對照 loss code 驗證旗標語意

**發生什麼:** 蒸餾矩陣 launcher 用「`--ed_weights` 傳空字串」表達「關掉 explicit-KD」。實查 loss 組裝才發現:該 repo 中 ed_weights **缺席的 task 預設權重 1.0**、`distill_tasks` 空 = 全部 task —— 空字串的真實語意是「**全模態 ED 全開、每個權重 1.0**」(比官方校準 λ 大 10~10⁵ 倍),與意圖完全相反;且 bash 未引號的空字串會被吃掉,argparse 直接 crash。兩個受影響格(feature-only、relation-only)在開跑前修正,否則那兩格數字會是「看起來合理但量錯東西」的假結果。

**根因:** 憑「空=關」的直覺寫批次設定,沒有回 loss 組裝的 code 驗證每個旗標組合的真實語意;預設值語意(default-on vs default-off)因 repo 而異。

**下次怎麼避免:** 開任何多格批次(矩陣/sweep)前,把**每格的旗標組合**逐一對照 loss/forward code 走一遍(哪個項會進 loss、權重多少),寫成「cell → flags → 驗證點」表放進 experiment-plan;「關閉」一律用顯式零值,不用留空。此 audit 應在 Phase 3 自審時做,屬「一次一變因」的落地檢查。

### 宣告「卡在缺資料」前,先 grep repo 的 registry/config —— 常已備好最小 fixture

**發生什麼:** experiment-plan 一度把 Stage 0 標成「磁碟零訓練資料 → 無法起跑」的硬 blocker。實查 registry 才發現 repo 已備 `nycu_timing`(64 筆 timing-only 集,對應 json 已存在),就是為「驗 pipeline + 量 wall-clock」而設 → Stage 0 其實現在就能跑。且該 fixture 註解同時揭露一個計畫沒抓到的汙染陷阱(用主指標的影片當訓練 → train/test 汙染,只能量 wall-clock 不能當訊號)。

**根因:** Phase 2 survey 查了 `data/` 目錄與下載的資料集,但沒查 repo 的**資料 registry / config yaml**;那裡才記錄了專案自備的最小訓練 fixture 與其用途註解。

**下次怎麼避免:** Phase 2 盤點資料時,除了看資料目錄,**一定 grep repo 的 registry / dataset config**(`*registry*`、`datasets/*.yaml`、訓練 entry 的 `--data*` 旗標)找有沒有 purpose-built 的最小/timing/debug fixture;它常已存在、且註解會揭露汙染或用途邊界。宣告任何 stage「blocked on 缺 X」前,先確認 repo 沒有現成的 X。

### Baseline 數字跨 data revision 不可比,差點做出假進步結論

**發生什麼:** 復刻某模型後拿自己跑的 eval 數字去跟論文官方數字比,一度以為復刻不忠實 / 有落差。實際上兩者算在**不同的 dataset revision**(且 eval scoring 中途修過一個 bug),根本不可直接比。

**根因:** Phase 2 盤點 baseline 時,沒把每個數字的 data revision 與 eval scoring 版本一起記下來,可比性沒被標出來。

**下次怎麼避免:** landscape.md 的 Baselines 表**強制**每個數字都填「來源 + data revision + eval scoring 版本 + 是否可比」;freshness-check 對「資料 revision / scoring 變動」特別敏感;experiment-plan 若拿不可比的數字當參照,必須紅字標明。(已編進 landscape 模板與 freshness-check,此處留作動機案例。)
