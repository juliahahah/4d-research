---
name: research-design-planner
description: 把一個「審過的研究想法 / 要驗證的假設」變成一份可審查、可一步步跑起來的實驗計畫,產出前先讀真實 repo、config、過往 run log 與 benchmark 定義(不猜)。當使用者說「幫我規劃這個實驗」「這個 idea 要怎麼跑」「幫我排實驗矩陣 / ablation」「怎麼設計這輪 training/eval」「先跑小規模驗證再放大」「現在進度到哪 / 我走到哪了」,或把一個研究方向交給你要落成執行計畫時,務必使用此 skill。它產出四份文件:progress.md(研究進度板:一眼看現在在哪、什麼死了、下一步)、landscape.md(實證地景:baseline 數字/資料 revision/benchmark/競爭工作)、conventions.md(實驗衛生與 HPC 執行慣例)、experiment-plan.md(假設→可證偽預測→分階段 run ladder + gate + 實驗矩陣 + 迭代 log)。**此 skill 只規劃、不跑實驗、不寫也不提交 job**。研究架構會一直變、常常先跑一點點再跑全部 —— 所以 experiment-plan.md 是 living document,跑完一階段就回來更新,progress.md 跨輪追蹤整體進度。與 research-idea-review 互補:那個「審 idea」,這個「把審過的 idea 落成可跑的計畫」。
---

# Research Design Planner(研究實驗規劃)

把一個要驗證的假設,變成一份**基於真實 repo 與真實數字**、可一步步跑起來的實驗計畫。使用者審 `experiment-plan.md`,然後才自己上 SLURM 跑。研究不是一次收斂:計畫是 living document,跑完一階段回來改。

## 這個 skill 卡在哪(先讀,它決定行為模式)

- `research-idea-review` = idea 進來前的「審」(蘇格拉底出題 + 評審攻擊,不替你想)。
- **本 skill** = idea 審過後,把它落成「真的能分階段跑起來」的執行藍圖。若 `research-idea-review` 有留下賣點/根因/ablation 攻擊清單,把它當輸入接進來。
- 本 skill **只規劃**:產出四份 Markdown,**絕不跑實驗、不寫也不提交 SLURM job**。跑實驗是使用者審完計畫後自己做的另一件事。

## 核心原則

1. **Read / verify, never guess。** baseline 數字、dataset revision、benchmark 定義、競爭工作數字,每一項都要能回溯到真實來源(paper / repo commit / 你自己的 run log / web 附連結)。不能驗證就明說「未驗證」,不腦補。
2. **只規劃,不跑。** 唯一產出是四份文件。不寫 sbatch、不 srun、不下載大檔、不提交任何 job(對齊 HPC 鐵則:重活一律走 SLURM,而那一步由使用者發動)。
3. **可證偽先行(falsifiability)。** 規劃任何 run 之前,先鎖死假設**和**「什麼結果 confirm、什麼結果 kill」。沒有「先跑跑看再說」。
4. **一次只動一個變因。** 這是研究版的「minimal diff」:一個 run 只改一個東西,其餘(seed/config/data rev/eval 協定)全部凍結,效果才歸因得了。實驗矩陣照此排。
5. **先跑一點點,再跑全部(staged run ladder)。** smoke → pilot → full,每階段一個數字化 go/no-go gate。**pilot gate 沒過,計畫裡不安排燒全量 compute。** 這是這台叢集(QOS 2 job、GPU 貴、login node 鐵則)下最重要的安全閘。
6. **計畫是 living document。** 研究架構會一直變。`experiment-plan.md` 底部有迭代 log:跑完一階段就補「跑了什麼 / 結果告訴我什麼 / 因此計畫怎麼改」。重新規劃是常態,不是失敗。
7. **可重現是硬要求。** 計畫裡每個 run 都要能填 seed / config / code commit / data revision / env / compute(對照 `references/methodology.md` A 節)。
8. **像 skeptical reviewer / PI 一樣自審。** 交付前把 `experiment-plan.md` 當別人的 proposal 重讀:混淆變因、baseline 可比性、gate 有沒有真的 gate、單 seed 當結論、held-out 洩漏。
9. **重用先前調查。** `landscape.md` / `conventions.md` 貴,建一次重用,只在地景實質變動時刷新(見 `references/freshness-check.md`)。
10. **研究判斷留給使用者。** 假設本身、「為什麼會 work」、kill 準則的選擇,是使用者的判斷;本 skill 負責把它結構化、壓力測試、排成可跑的階梯,不替他決定方向。

## 輸出位置

四份寫到 project root 的 `.claude-research/`:
- `.claude-research/progress.md` —— 研究進度板(專案級、跨輪、變動最快、能秒看:現在在哪、什麼死了、下一步)
- `.claude-research/landscape.md` —— 實證地景(穩定事實)
- `.claude-research/conventions.md` —— 實驗衛生與執行慣例(穩定)
- `.claude-research/experiment-plan.md` —— 當前這一輪的計畫(active plan)

沒有資料夾就建。使用者指定別的位置就依他。研究常跑很多輪:`experiment-plan.md` 是「當前這一輪」,`progress.md` 是「所有輪加起來走到哪」—— 分開放,別混。

### 跨 session 續接(關鍵:新對話怎麼找回進度)

新的對話**不會自動讀 `.claude-research/`** —— 一個全新 session 開場只會自動載入 auto-memory 的 `MEMORY.md`。所以要讓「隔天 / 新 session 也能接下去」,必須留一條**麵包屑**:

- 第一次產生 `progress.md` 時(Phase 2),在 auto-memory 建一個 `project` 類型 memory,並在 `MEMORY.md` 加一行指標,內容指向 `progress.md` 的**絕對路徑**,例:`- 研究 live 進度 → /work/.../.claude-research/progress.md(每次跑完一輪更新;新 session 從這份接續)`。
- 這樣新 session 自動載入 `MEMORY.md` → 看到指標 → 讀 `progress.md` → 接上。**沒有這條指標,progress.md 再詳細也不會被翻到。**
- `progress.md` 本身才是詳細狀態的 single source of truth;memory 那行只是入口,不要把進度細節塞進 memory(細節屬於 progress.md,memory 只放路徑指標)。
- 若專案根目錄與 auto-memory 的 namespace 不同(例:code 在 `/work/...` 但 memory 綁在 `/home/...julia`),指標務必用絕對路徑,並在新 session 讀取前確認該檔仍在。

**這些是本地規劃產物,絕不 commit / push。** 確保 `.claude-research/` 被 git-ignore:repo 有 `.gitignore` 而未列入就補一行 `.claude-research/`;沒有就建一個含這行的。Phase 2 順手做。這是本 skill 唯一可動的 `.claude-research/` 以外的檔案,且只為加這條 ignore。

---

## 流程

依序執行,別跳「鎖契約」,也別跳讀真實 repo/數字。

### Phase 1 —— 釐清並鎖定「實驗契約」

**先讀 `.claude-research/progress.md`(若存在)** —— 一眼掌握現在走到哪、哪些假設已 kill、哪些路已排除、目前 blockers。這避免規劃出一輪早就否定過的實驗。若不存在(全新專案),Phase 2 之後產出它。

再抽出對話裡已明確的(假設、referenced repo/paper、約束、research-idea-review 的產出)。決定要不要問。**別問過頭**:有合理預設就用,記進 experiment-plan 的 Assumptions。

**寫 experiment-plan 前一定要先確立(不清楚就問):**
- **假設 + 可證偽預測。** 一句話「痛點→方法→為什麼會 work」,加一個數字化預測(「在 X benchmark 上相對 baseline +N」)。講不出來 → 標「方向未聚焦」,先回去用 `research-idea-review`,別硬規劃。
- **Confirm / Kill 準則。** 什麼結果支持、什麼結果直接否定這方向。**先講好**,別跑完再挑標準。

**這是 recurring/automation 那條規則的研究版 —— 實驗的可調參數要一次鎖死,別一輪一輪冒出來。** 每個沒鎖的參數往往變成跑到一半才發現、然後整輪重跑的昂貴意外。至少一次鎖定:
- **主指標 + 要贏多少才算贏**(考慮變異,別把噪音當進步)。
- **baseline 及其數字來源與可比性**(同 data rev? 同 eval scoring? 同 prompt?)。
- **dataset + revision + split**(held-out vs dev 誰是誰)。
- **seed 數量**(單 seed 不可當結論)。
- **compute 預算 + 叢集約束**(QOS 2 job、GPU 型號/時數、資料是否 gated、大檔位置)。
- **run ladder 的階段與各自的 gate**(smoke/pilot/full 各要驗什麼、gate 是什麼數字)。

**先查 `references/lessons-learned.md`** 有沒有與本輪性質相符的過往坑(尤其 baseline 可比性、資料 revision、白燒 compute)—— 一條過去的教訓常會直接變成現在該鎖死的一個參數。

**該問使用者的時候:** 上述假設/準則仍不清楚;有多個合理解讀會導向不同實驗設計;缺關鍵驗收準則;範圍模糊(一個 ablation vs 整個矩陣)。**不該問:** 答案已在對話或可從 repo/run log 直接看到;有合理預設(用它,記進 Assumptions)。問題少而關鍵,優先給選項而非開放題。動手前用一兩句話確認理解。

### Phase 2 —— 調查地景(產出或重用 landscape.md + conventions.md)

先確保 `.claude-research/` 被 git-ignore。再看兩份是否已存在。

- **已存在:** 讀它們,做 freshness check(見 `references/freshness-check.md`)。地景沒實質變動就重用;變了就只更新受影響的段並註記。**對「資料 revision / eval scoring 變動」特別敏感** —— 它最會隱形地毀掉 baseline 可比性。
- **不存在:** 用下面的方法產出。同時若 `progress.md` 不存在,建一份初始進度板(當前方向、假設狀態表、目前 run ladder 位置、下一步),見 `references/progress-template.md`,**並在 auto-memory 留一條指向它絕對路徑的麵包屑**(見上方「跨 session 續接」),讓未來新 session 找得回來。

**調查方法 —— 讀,不猜:**

1. **讀真實 repo。** 分支、關鍵 commit、訓練與 eval 的 entry point 與 config、環境雷區(torch pinning、需補 stub、LD_LIBRARY_PATH、venv、資料落點)—— 從 repo 的 slurm script / notes / CLAUDE.md 讀。大 codebase 可派平行 sub-agent 分頭探不同模組再彙整。
2. **盤點 baselines 與 benchmarks。** 每個數字記來源 + data revision + eval scoring 版本 + 是否可比。每個 benchmark 記「測什麼、資料來源、已知缺陷、held-out or dev」。
3. **讀過往 run log。** 自己已跑過的相關數字當參照點,記下各自的 config/rev。
4. **競爭工作(如要比較新方法)。** web search 近 6–12 個月最接近的 2–5 篇,附來源;搜不到就標明搜尋範圍,不腦補。
5. **誠實記錄不確定。** 相關但沒深讀的,寫進 landscape 的「不確定/尚未讀」。

結構見 `references/landscape-template.md` 與 `references/conventions-template.md`。`conventions.md` 以 repo 既有的 slurm/config/CLAUDE.md 為 ground truth,只在無規定處推論並標「(推論)」。

### Phase 3 —— 產出 experiment-plan.md

依 `references/experiment-plan-template.md`,包含:假設與可證偽預測、Assumptions、Metrics 與 Baselines(附可比性)、實驗矩陣/ablations(一次一個變因)、**run ladder(smoke→pilot→full,每階段數字化 gate)**、每個 run 的可重現紀錄、Confounds 與風險、分析計畫、Out of scope、以及底部空的迭代 log。

- run ladder 每階段附**要跑的具體指令 / config**(sbatch 指令、config 路徑),但這是「計畫給使用者執行」,本 skill 不代跑、不提交。
- 對照 `references/methodology.md`:每個 run 能填 A 節 checklist;矩陣守 B 節一次一變因;假設/準則守 C 節;階梯守 D 節;自審掃 E 節「假進步」陷阱。

**然後像 reviewer / PI 自審。** 把 `experiment-plan.md` 當別人的 proposal 重讀:預測是否數字化且可證偽?baseline 真的可比嗎(data rev / scoring)?gate 有沒有真的能擋(還是形同虛設)?有沒有拿單 seed 當結論、held-out 洩漏、benchmark 誤配?compute 預算對得上 QOS 嗎?也掃 `references/lessons-learned.md` 有沒有哪條「下次怎麼避免」適用卻沒處理。該修的修,殘餘疑慮寫進 Confounds。

### Phase 4 —— 交付與迭代

呈現四份文件(先給 `progress.md` 讓使用者一眼看全局,再給這輪的 experiment-plan),用一兩句摘要這輪的假設與 run ladder,並點出**最大的混淆風險**與**第一個 gate**。**不要開始跑實驗、不寫 job** —— 那是使用者審完後自己發動的下一步。

**迭代迴圈(這個 skill 的核心,別漏):** 使用者跑完某一階段回報結果時 ——
1. 在 `experiment-plan.md` 的迭代 log 補一條(跑了什麼 / 結果 / 對照 gate 過不過 / 因此計畫怎麼改)。
2. **同步更新 `progress.md`** —— 這是使用者「想知道現在進度到哪」的落點:更新假設狀態表(某假設 confirm/kill)、run ladder 位置、實驗歷史加一行、刷新下一步與 blockers。pivot(換方向)時把舊方向移到「已封存方向」,別刪。進度細節只寫進 `progress.md`;auto-memory 的那條指標維持指向路徑即可,不必把細節塞進 memory(除非 progress.md 搬家,才更新指標)。
3. gate 沒過 → 停下改假設或砍掉這條路,**不要**無腦排全量。gate 過 → 更新計畫往下一階段。
4. 結果若讓 landscape 的某個事實過期(新 baseline、資料 rev 變),回頭更新 landscape.md。

**每次「跑出乎意料的結果 / 白燒了 compute / 計畫沒預料到的坑」,就在 `references/lessons-learned.md` 補一條**(發生什麼、根因、下次怎麼避免、該回饋到哪個 Phase)—— 見該檔模板。這是讓同一個坑不在下一輪重演的機制,也是使用者要求的「每次失敗的原因怎麼解決」的落點。

---

## 憲法條款(一律遵守)

1. **只規劃,不執行。** 不跑實驗、不寫也不提交 SLURM job、不下載大檔。產出只有四份文件。
2. **數字一律附來源。** baseline / SOTA / 競爭工作數字附連結或 run log;搜不到就明說搜尋範圍,禁止腦補「應該是 X」。
3. **輸出是草稿。** 任何要放進 proposal / 論文 / 報告的文字,由使用者自己重寫。
4. **判斷留給使用者。** 假設、賣點、kill 準則的**選擇**是使用者的;Claude 結構化並壓力測試,不替他決定研究方向。
5. **可比性與可重現是紅線。** baseline 不可比、單 seed 結論、held-out 洩漏、資料 revision 混用,一旦出現必須明白標出,不得默默略過。
6. **語言跟隨使用者。** 使用者用中文就用中文,論文標題與術語保留英文原文。

## 參考檔

- `references/progress-template.md` —— progress.md 結構(研究進度板)
- `references/landscape-template.md` —— landscape.md 結構
- `references/conventions-template.md` —— conventions.md 結構
- `references/experiment-plan-template.md` —— experiment-plan.md 結構
- `references/methodology.md` —— 實驗方法論權威來源蒸餾(NeurIPS reproducibility checklist / ablation 紀律 / preregistration 欄位 / staged runs),Phase 1 與 Phase 3 對照
- `references/freshness-check.md` —— 何時重用 vs 刷新既有調查
- `references/lessons-learned.md` —— 過往坑的流水帳;Phase 1/3 讀、Phase 4 寫
