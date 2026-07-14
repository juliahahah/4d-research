# 研究實驗規劃方法論(權威來源蒸餾)

這份是 `research-design-planner` 的「外部標準」參考檔 —— 相當於 project-design-planner 裡 `conventional-commits.md` 的角色,只是這裡的標準是**實驗方法論**而非 commit 格式。Phase 1(鎖契約)與 Phase 3(自審)都要對照這份。

所有條目蒸餾自下列公開來源,宣稱請回溯來源,勿憑記憶擴充:

- NeurIPS 2019 Reproducibility Program report — https://arxiv.org/abs/2003.12206
- NeurIPS Paper Checklist Guidelines — https://neurips.cc/Conferences/2022/PaperInformation/PaperChecklist
- REFORMS: Recommendations for ML-based Science (Science Advances) — https://www.science.org/doi/10.1126/sciadv.adk3452
- Preregistration for Experiments with AI Agents — https://arxiv.org/abs/2606.11217
- Pre-registration for Predictive Modeling — https://arxiv.org/abs/2311.18807
- Controlled ablation study(方法整理)— https://www.emergentmind.com/topics/controlled-ablation-study
- OSF Preregistration — https://www.cos.io/initiatives/prereg

---

## A. ML Reproducibility Checklist(NeurIPS)—每個 planned run 都要能回答

一份可信的實驗計畫,對「要跑的每個 run」都要能填出下列欄位。缺哪一項就是計畫的漏洞:

1. **模型 / 演算法描述**:數學設定、演算法、模型架構講清楚(不是「用 4D-RGPT」而已,是哪個 checkpoint、哪個 revision)。
2. **資料集規格**:用哪個 dataset、**哪個 revision / 版本**、怎麼存取、train/val/test 怎麼切、held-out 是哪個。
3. **產生報告數字的確切超參 config**:不是「差不多的設定」,是能重現該數字的完整 config。
4. **超參搜尋範圍 + 選擇準則**:掃了哪些、依什麼準則選定(避免「挑最好看的那個 seed」)。
5. **random seed 數量與聚合方式**:跑幾個 seed、結果怎麼聚合。**單一 seed 的數字不可當結論**。
6. **compute infrastructure**:GPU 型號、卡數、時數(本叢集:H100/H200,QOS 每人 2 job —— 直接影響能排幾個 run)。
7. **結果報告方式**:報中心值**加**變異(mean±std / error bar / CI)。只報單一數字 = 不可信。

## B. Ablation / 受控實驗紀律 —— 對應「一次只動一個變因」原則

1. **固定 baseline + ceteris paribus**:一次只改一個元件/變因,其餘(seed、超參、資料切分、評測協定)全部凍結,效果才歸因得了。
2. **一個 run 只拿掉/改一個東西**:同時動多個 → 無法判斷是誰造成差異。
3. **事先註記所有常數**:seed、split、超參,先寫進計畫(見 preregistration)。
4. **拿掉元件通常要重訓**:只把某層 zero-out 而不重訓,會低估它的貢獻(其他元件沒機會適應它的缺席)。
5. 每個「攻擊/懷疑」都應對應一個**明確的 ablation run**,而不是口頭討論。

## C. Preregistration 欄位 —— 對應「可證偽先行 / falsifiability」

跑之前先鎖死,避免事後 p-hacking / forking paths(挑對自己有利的分析路徑):

1. **假設(hypotheses)**:實驗前就寫下的**具體預測**(「加了 X 會讓 STI +Y 分」),不是「應該會變好」。
2. **成功 / kill 準則**:什麼結果 confirm、什麼結果直接否定這個方向。先講好,別跑完再挑標準。
3. **metric + 成功門檻**:主指標是什麼、要贏多少才算贏(考慮變異,別把噪音當進步)。
4. **實驗協定**:怎麼跑、評測怎麼算(對齊你 repo 的 eval,例如 lmms-eval 的 scoring 是否已修對)。
5. **分析計畫**:結果出來後怎麼統計、怎麼比較 baseline。
6. **stopping rules**:什麼條件下停(gate 沒過就停,別硬燒)。
7. **degrees of freedom 記錄**:所有「可以這樣也可以那樣」的決策,先記下來,避免事後合理化。

## D. Staged run ladder —— 對應「先跑一點點再跑全部」

方法論來源沒有單一標準名稱,但受控實驗 + compute 成本管理的共識是**分階段驗證、每階段設 gate**:

- **Stage 0 — smoke**:最小規模(1 sample / 1 step / tiny subset),只驗「跑得起來、不 crash、端到端接通」。在 compute node 跑,秒~分鐘級。
- **Stage 1 — pilot**:小子集、單 seed,只驗「信號方向對不對、值得不值得放大」。**Gate**:達到某個最低信號才准往上。
- **Stage 2 — full**:全量 / 多 seed / 全矩陣,產生要寫進論文/報告的數字。**只有 pilot gate 過了才跑**。

每個 gate 是明確的 go/no-go 條件(數字化),寫進計畫。gate 沒過 → 停下改假設,而不是無腦跑全量。這一層直接對應本叢集 QOS 2 jobs + GPU 貴 + login node 鐵則:**別在 pilot 都沒驗證前燒掉全量 compute。**

## E. 常見「假進步」陷阱(自審時逐條掃)

- **baseline 不可比**:兩個數字來自不同 data revision / 不同 eval scoring / 不同 prompt → 不能直接比(這是本 repo 踩過的真坑,見 lessons-learned)。
- **單 seed 當結論**:差異落在噪音範圍內。
- **挑好看的數字**:超參/seed/checkpoint 事後挑最高的。
- **benchmark 誤配**:benchmark 測的東西支撐不了你要宣稱的結論(交叉參考 research-idea-review 第 5 步)。
- **held-out 洩漏**:拿來調參的集合又拿來報成績。
