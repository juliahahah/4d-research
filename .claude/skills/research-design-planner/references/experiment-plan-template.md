# experiment-plan.md 模板

這是使用者審查、也是這輪實驗真正的執行藍圖。它是 project-design-planner 的 `design.md` 的研究版,但有兩個根本差異:

1. **不是一次收斂,是 living document**:底部有「迭代 log」,每跑完一階段就更新(跑了什麼 / 結果告訴我什麼 / 因此計畫怎麼改)。重新規劃是常態,不是失敗。
2. **核心不是 before/after diff,而是「假設 → 可證偽預測 → 分階段 run ladder + gate」**。

用以下結構:

```markdown
# 實驗計畫:<這輪要驗證的東西,一句話>

_日期:<date>。基於本資料夾的 landscape.md 與 conventions.md。_

## 假設與可證偽預測(先於任何 run)
- **假設**:一句話「痛點 → 方法 → 為什麼會 work」(接 research-idea-review 的賣點,如果有)。
- **具體預測**:實驗前寫死的數字化預測,例:「在 STI-Bench held-out 上,加了 X 相對 baseline(來源:landscape.md)+≥N 分」。
- **Confirm 準則**:什麼結果算支持假設。
- **Kill 準則**:什麼結果直接否定 / 讓這方向不值得再投 compute。**先講好。**

## Assumptions(用預設取代發問的每一項)
每個沒問使用者、自己選的預設。讀者不同意時知道該改哪。

## Metrics 與 Baselines
- 主指標 + 要贏多少才算贏(考慮變異,別把噪音當進步)。
- 對照的 baseline 及其數字**來源與可比性**(同 data rev? 同 scoring?)—— 引用 landscape.md。
- held-out vs dev 的分工。

## 實驗矩陣 / Ablations(一次只動一個變因)
| Run 名 | 動的變因 | 凍結的常數(seed/config/data rev/eval) | 目的 / 對應的懷疑 |
|---|---|---|---|
- 每個「攻擊/懷疑」對應一個 run。ceteris paribus:一次一個變因。

## Run ladder(先跑一點點 → 全部)
分階段,每階段一個明確 go/no-go **gate**:

| 階段 | 規模 | 目的 | Gate(數字化 go/no-go) | 估計 compute(GPU 時數 / job 數) |
|---|---|---|---|---|
| Stage 0 smoke | 1 sample / 1 step | 跑得起來、不 crash | 端到端跑完、輸出合理 | ~分鐘,1 job |
| Stage 1 pilot | 小子集 / 單 seed | 信號方向對不對 | ≥ 某最低信號才往上 | ... |
| Stage 2 full | 全量 / 多 seed / 全矩陣 | 產生報告數字 | —(只有 pilot gate 過才跑) | ... |

- 每階段附**要跑的具體指令 / config**(sbatch 指令、config 路徑)—— 這是計畫,使用者執行,本 skill 不代跑。
- gate 沒過 → 停,回去改假設,別硬燒全量。

## 每個 run 的可重現紀錄(對照 methodology.md A 節)
要跑的每個 run 都能填:模型/checkpoint+rev、dataset+rev+split、完整 config、超參搜尋範圍與選法、seed 數、compute、結果報 mean±std。

## Confounds 與風險
- 可能讓結果不可信的混淆(baseline 不可比、單 seed、held-out 洩漏、benchmark 誤配 —— 對照 methodology.md E 節)。
- 環境 / compute 風險(QOS 2 job 排程、資料 gated、大檔位置)。
- 使用者該在跑之前拍板的事。

## 分析計畫
結果出來後怎麼比、怎麼判 confirm/kill。事先寫好,避免事後挑標準。

## Out of scope
這輪計畫刻意不做的事(留給後續迭代)。

## 迭代 log(living —— 每跑完一階段就補一條,最新在上)
```
### <日期> Stage <X>:<跑了什麼>
- 結果:<數字 / 觀察>
- 對照 gate:<過 / 沒過>
- 因此計畫怎麼改:<改了哪個假設 / 加了哪個 ablation / kill 掉哪條路>
```
```

讓「假設 → gate → 迭代 log」成為文件的主角:使用者靠這條線就能信任這輪該不該往下燒 compute,不必自己重跑一遍。
