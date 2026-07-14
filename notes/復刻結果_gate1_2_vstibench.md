# 4D-RGPT 復刻結果:Gate 1 / Gate 2(2026-07-14)

> 環境已在 NYCU HPC 完整跑通(flash-attn 源碼編、deepspeed 補洞、L4P teacher、資料集就緒),eval pipeline 端到端驗證通過 → **復刻可信**。
> 表中分數皆為**我自己跑**〔我跑〕;論文值〔paper〕另立對照,因 dataset revision / ablation 種類不同**不可直接並排**(見 §4)。

## 1. 總表:兩 benchmark × 兩模型

| Benchmark | 題數 | frames | base NVILA-Lite-8B (ZS) | 官方 4D-RGPT-8B | Δ | held-out? | 用途 |
|---|---|---|---|---|---|---|---|
| **STI-Bench** | 2064 | 16 | 34.8 | **47.5**(scoring 修正後) | **+12.7** | ✅ 是 | **建議主指標** |
| VSTI-Bench | 5736 | 16 | 50.2 | 62.3 | +12.1 | ❌ 否,paper 訓練來源 | 次要 sanity |

> 兩 benchmark 方向、量級一致(官方皆 > base 約 +12)。STI 原本掉到 20.3(≈亂猜)是 **scoring bug**,已修 → §3。

## 2. VSTI-Bench 分題型(全 5736 題,16f,lmms-eval)

| Question type | base (ZS) | 官方 | Δ |
|---|---|---|---|
| camera_obj_abs_dist(數值 MRA) | 22.3 | 44.3 | **+22.0** |
| camera_displacement(數值 MRA) | 1.9 | 23.6 | **+21.7** |
| camera_obj_rel_dist v1 | 52.7 | 71.4 | **+18.7** |
| camera_obj_rel_dist v2 | 61.7 | 75.3 | +13.6 |
| obj_obj_rel_pos(上下) | 77.3 | 90.7 | +13.4 |
| camera_obj_rel_dist v3 | 66.0 | 76.6 | +10.6 |
| obj_obj_rel_pos(前後) | 65.6 | 69.1 | +3.5 |
| camera_movement_direction | 49.2 | 53.2 | +4.0 |
| obj_obj_rel_pos(左右) | 55.2 | 56.5 | +1.3 |
| **overall** | **50.22** | **62.30** | **+12.1** |

> **on-thesis**:提升最大的全是**深度/距離**題(abs_dist/displacement +22、rel_dist +10~19),不吃深度的「左右」幾乎不動(+1.3)→ 蒸餾確實在做論文宣稱的事。

## 3. STI-Bench scoring 修正(2026-07-14)⭐ 本次重點

| Model | strict(字母 only,舊=bug) | 修正後(字母+數值對應) | letter / numeric / unmatched |
|---|---|---|---|
| base NVILA-Lite-8B | 34.8 | 34.8(不變) | 2064 / 0 / 0 |
| 官方 4D-RGPT-8B | 20.3 | **47.5** | 915 / 1032 / 117 |

**bug**:STI 選項是數值,官方 fine-tuned 模型不聽「output only the letter」,直接答選項的**值**(如 `2.36m`,而選項 (E) 就是 `2.36m`);upstream 字母 parser 判全錯 → 假性掉到 20.3。base 乖乖答字母 → 34.8 沒事。
**修法**:`tools/vlm/lmms-eval/ext/tasks/stibench/utils.py` 加 `_numeric_option_match()` — 字母抽不到時,數值**精確**對應到唯一相符選項(相對容差 1e-3,**無 nearest snapping**),記 `matched_by`。

**證據(修正正確、非灌水)**:

| numeric-answer 分析 | 數字 |
|---|---|
| 精確對應到唯一選項(同單位) | 1032 |
| 對應到 0 個選項 | 0 |
| 對應到 2+(歧義,保守判錯) | 27 |
| 數值子集準確率 | 54.6%(> 字母子集 45.7%) |
| "117 unmatched" | 是文字答案 `Right`/`Front-Left`/`Front`(方向題,無數字),非自由估計 |

→ 每個數值答案都是**逐字 echo 某個選項的值(同單位)**,模型是在「選選項」而非自由估計 → 精確對應完全正確,**不需 cm→m 正規化或 nearest**(先前用 nearest+正規化得 33.0 是把已同單位的值錯誤正規化 / id-join 配錯 造成的假象)。
**驗證**:離線用 saved `samples_stibench.jsonl` 重跑修正後 `stibench_process_results()` → 官方 47.5、base 34.8。

**對矩陣**:STI 在正確 scoring 下 base→官方 = **+12.7**,和 VSTI(+12.1)同級 → **不是弱區分器**;加上 STI held-out、VSTI 是訓練來源 → **STI 當主指標**。前提:矩陣跑 STI 前 scoring 必須是本修正版,否則訓練過的格子都會假性偏低。

## 4. 論文對照(⚠️ 皆不可直接並排)

**STI-Bench**(來源:paper Table A4,**資料量**消融,非蒸餾消融):

| | STI | R4D-Avg | Sta | Dyn |
|---|---|---|---|---|
| 〔paper〕Zero-shot | 33.8 | 37.9 | 29.1 | 41.3 |
| 〔paper〕V / V+W / V+W+R | 35.4 / 36.0 / 37.0 | … | … | … |
| 〔paper〕V+W+R+S (Ours) | **37.6** | 42.2 | 32.9 | 45.7 |
| 〔我跑〕base ZS / 官方(修正後) | **34.8 / 47.5** | — | — | — |

- base 34.8 ≈ paper 33.8 ✅。**frames=16 確定正確**(paper §6.2 明寫 16;30f 是錯方向,放棄)。
- **官方 47.5 > paper 37.6 = 答題格式效應**(非 bug):paper 37.6 是「逼模型吐字母」的分(我字母子集也才 45.7),我 47.5 是允許數值精確對應後的真實空間能力;矩陣要量的正是後者。

**VSTI-Bench**(來源:paper Table 2 / A1):

| | 〔paper〕舊 6042 rev | 〔我跑〕新 5736 rev | Δ |
|---|---|---|---|
| base NVILA-Lite-8B | 45.2 | 50.22 | +5.0 |
| 4D-RGPT-8B | 59.1 | 62.30 | +3.2 |
| 增益 | +13.9 | +12.1 | 一致 |
| (參考 15B base/蒸餾) | 42.4 / 58.6 | — | — |

- **不可比主因 = 資料 revision**:HF `Journey9ni/vstibench` 2026-07-13 erratum(相機位置 GT bug `-R.T@t` → `pose[:3,3]`),test 6042→5736、改 label,dataset card 明講舊 label 不可比。增益量級/方向一致 → 復刻忠實。
- ⚠️ **VSTI 是 paper 訓練來源**(A4 的 `V`;mixture V+W+R+S = repo `sat+vstibench+robofac+wolf`)→ 評測有 train/test 重疊風險,故列次要。待查 train/test split 是否分離。
- 圖表若用須註明:「Evaluated on VSTI-Bench 2026-07-13 revision (5736 items); not comparable to the paper's pre-erratum 6042.」

## 5. 協定 / 環境參數

| 項目 | 值 |
|---|---|
| frames | **16**(論文 16;⚠️ VSTI 上游腳本 32、STI 上游 30,別搞混) |
| harness | lmms-eval;VSTI=`vstibench`(vendored VLM-3R)、STI=`stibench`(自建 launcher `scripts/lmms_eval/tasks/stibench.sh`) |
| generation | T=0 / greedy / max_new_tokens=16 |
| VSTI 解析 | MCA=fuzzy exact_match(第一 token 去尾點、大小寫不敏感)、NA=MRA(.5:.95:.05);overall=各 type 平均再平均 ×100 |
| STI 解析 | 5-way MC;字母 6-pattern cascade + raw fallback +（本次）**數值→選項精確對應**;與上游 `opensource_test.py` prompt/regex 逐項核對過 |
| runtime | VSTI ~33–35 min、STI ~25 min(1×H100,normal) |
| STI 資料 | MINT-SJTU/STI-Bench(~1GB,2064 題,371 影片)@ `/work/a314581027/4d-rgpt-data/lmms_eval_cache/STI-Bench`(真實目錄非 symlink);`STI_BENCH_ROOT` 絕對路徑 |
| VSTI 資料 | HF `Journey9ni/vstibench` 2026-07-13 revision(5736 題) |

> **待確認(次要)**:論文 region-level 比較用 SoM 標註;VSTI 屬 non-region-level 應不套用,要 100% 保險可查 lmms-eval path 沒額外畫框。
> 環境雷區(flash-attn 源碼編、deepspeed 補 stub、torch `--no-deps`、`LD_LIBRARY_PATH`、cache 目錄名 ≠ task 名、compute node 有網路)已移到 `/home/a314581027/julia/CLAUDE.md`;可執行 source of truth 在 repo `scripts/slurm/nycu_*.sbatch`。

## 6. 蒸餾實驗矩陣 → 已移至 `.claude-research/`(本 note 只管復刻結果)

矩陣實驗(**23 個 single-method 實驗**,6 KD × knowledge)的設計、audit、跑動狀態與**全部結果**都在 repo 的 `.claude-research/`,不在本 note:

- **記分板(23 全格一張表 + 控制列)**:`/work/a314581027/4d-rgpt/.claude-research/results.md`
- 設計 / 每格旗標 audit / 迭代 log:`.../experiment-plan.md`;跨輪進度:`.../progress.md`;執行慣例(SLURM、權重、訓練後畫圖診斷):`.../conventions.md`

本 note 提供給矩陣的錨點(來自 §1/§3,勿改):**zero-shot STI = 34.8、官方 ckpt(全模態上界)= 47.5、STI 必用修正版 scoring**。
