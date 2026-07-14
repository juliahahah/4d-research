# 4D-RGPT 復刻結果:Gate 1 / Gate 2(2026-07-14)

環境已在 NYCU HPC 完整跑通(flash-attn 自源碼編、deepspeed 補洞、L4P teacher、資料集全就緒)。
eval pipeline **端到端驗證通過**,而且官方 checkpoint 的提升符合預期 → **復刻可信**。

## 結果:VSTI-Bench(全 5736 題,16 frames,lmms-eval)

| Question type | NVILA-Lite-8B (base, zero-shot) | 4D-RGPT-8B (官方, 蒸餾後) | Δ |
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

**關鍵觀察(on-thesis)**:提升最大的全是**深度/距離**題(abs_dist +22、displacement +22、rel_dist +10~19)——正是 depth/flow P4D 蒸餾該改善的地方;不吃深度的「左右」幾乎不動(+1.3)。蒸餾確實在做論文宣稱的事,矩陣結果可以信任。

## ⚠️ 版本不同:我的數字不能直接跟論文表格並排(VSTI-Bench 2026-07-13 erratum)

**上表的 50.22 / 62.30 是我自己跑出來的,不是論文數字。** 論文(4D-RGPT, arXiv 2512.17012,Table 2 / A1)官方值是:

| | 論文官方 (Table 2 / A1) | 我的復刻 (本 notes) | 差 |
|---|---|---|---|
| NVILA-Lite-8B (base) | **45.2** | 50.22 | +5.0 |
| 4D-RGPT-8B (蒸餾後) | **59.1** | 62.30 | +3.2 |
| 增益 Δ | +13.9 | +12.1 | 一致 |
| (參考) 15B base / 蒸餾 | 42.4 / 58.6 (+16.2) | — | — |

**為什麼對不上 —— 是資料集 revision,不是復刻 bug:**
- HF `Journey9ni/vstibench` 於 **2026-07-13** 發 erratum:相機位置 ground-truth 有 bug(用了 `-R.T @ t` 而非 `pose[:3,3]`)。修正後 **test 集 6042 → 5736 題**(移除 306 題並改動大量 label)。dataset card 明講「**舊 label 的結果與此版不可比**」,且用 buggy pipeline 產生的訓練資料 fine-tune 的模型也應重訓。
- 論文的 45.2 / 59.1 算在 **erratum 前的舊 6042 label**;我的 50.22 / 62.30 算在 **erratum 後的新 5736 label**(我的 `n-samples = 5736` 正是修正後數目)。兩者先天不可比。

**協定其餘部分與論文一致**(已逐項核對 `NVlabs/4D-RGPT` 官方 code):
- **frames = 16**(論文原文 "we use the same number of sampled frames, i.e., 16 frames";我的 fork 也 16)。⚠️ VSTI-Bench 上游 VLM-3R 原生腳本 `eval_vlm_3r_vstibench.sh` 用的是 **32 frames**,別跟論文的 16 搞混。
- harness = lmms-eval task `vstibench`(vendored 自 VLM-3R);generation T=0 / greedy / max_new_tokens=16;答案解析 MCA=fuzzy_matching(取第一 token 去尾點)→大小寫不敏感 exact_match、NA=MRA(.5:.95:.05);overall=各 question_type 平均再對所有 type 平均 ×100。

**結論與行動:**
- 增益量級一致(+12.1 vs +13.9)、方向一致(集中深度/距離題)→ **復刻忠實**,pipeline 沒問題。
- 想在 paper/slide 直接對標論文 59.1 那格:需抓 erratum 前的舊 revision 重跑(但官方說舊 label 有 bug、不建議)。
- 較合理做法:採用新版 5736 的數字為 baseline,並在圖表/文字**明確註明**:「Evaluated on VSTI-Bench 2026-07-13 revision (5736 items); not directly comparable to the paper's pre-erratum numbers (6042).」
- 待確認(次要):論文提到 region-level 比較用 SoM(Set-of-Marks)標註;VSTI 屬 non-region-level 應不套用,但要 100% 保險可查 lmms-eval 這條 path 沒有額外畫框。

## 注意
- 這是 **VSTI-Bench**(VLM-3R 版,HF `Journey9ni/vstibench`),**不是** notes 裡 33.8 的 **STI-Bench**。兩者不同 benchmark,數字不可直接比。33.8 那條要另跑 `stibench` task(需 STI-Bench 本地資料)。
- 每次全量 eval 約 33–35 分鐘(1×H100,normal 分區)。

## 環境備忘
- 環境雷區(flash-attn 源碼編、deepspeed 補 stub、torch `--no-deps`、LD_LIBRARY_PATH、lmms-eval cache 目錄名 ≠ task 名、compute node 有網路)已移到 **`/home/a314581027/julia/CLAUDE.md`**;可執行的 source of truth 在 repo **`scripts/slurm/nycu_*.sbatch`**。

## 下一步(Gate 3:量一格訓練 wall-clock)
- 訓練資料 `sat+vstibench+robofac+wolf` 走 gitignored 的 `llava/data/registry/datasets/*.yaml`,指向本地大型/gated 資料(NuScenes=Wolf、RoboFAC、SAT)→ **授權/容量是瓶頸,如 notes 早先預測**。
- 選項:(A) 用最小可得資料(小 subset)只為量 wall-clock + 矩陣 proof;(B) 備齊完整 mixture 做忠實復刻(要下載/申請大資料)。待決定。
