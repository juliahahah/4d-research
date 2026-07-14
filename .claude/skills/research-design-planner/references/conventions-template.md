# conventions.md 模板

專案的**實驗衛生與執行慣例**。這是 project-design-planner 的 `style.md` 的研究版:不是 coding style,而是「這個 repo / 這台叢集怎麼跑實驗才不會亂、才可重現」。

**已寫死的規則優先於推論**:repo 的 slurm script、config、CLAUDE.md、既有 run 命名若已規定,那就是 ground truth,直接引用;只有在沒有規定處才從既有 run 推論,並標「(推論)」。

用以下結構:

```markdown
# 實驗慣例(Conventions)

_源自 repo 的 slurm/config/CLAUDE.md 與既有 run;無規定處為推論(已標註)。_

## 叢集 / 提交慣例(硬規則)
- 重活一律走 SLURM 到 compute node(login node 只做輕量事)。account / partition / QOS 上限。
- 一個 run 怎麼提交(sbatch 哪支、srun 互動指令)、時限、GPU 型號。
- 引用來源:CLAUDE.md / scripts/slurm/*.sbatch。

## 環境 pinning(重現的前提)
- torch / cuda / gcc 版本綁定、venv 位置、需補的 stub、LD_LIBRARY_PATH。
- HF cache / TMPDIR / uv cache 導向(勿落 /home)。
- 引用來源。

## Run 命名與組織
- run / 實驗目錄怎麼命名(能一眼看出改了哪個變因)。
- config 放哪、怎麼版本化(檔案 or git)。
- 結果 / log / checkpoint 落點,怎麼跟 config 對應。
- (有 wandb / tensorboard 就寫怎麼用；沒有就標。)

## Seed / config / 可重現紀錄
- seed 政策(固定值?幾個 seed?)。
- 每個 run 該記什麼:seed、config hash、code commit、data revision、SLURM job id、GPU 時數。
- (對照 methodology.md A 節 checklist。)

## 評測協定
- 用哪支 eval、scoring 怎麼算(已知的 scoring bug 是否已修 —— 例:數值答案→選項對應)。
- 主指標、held-out vs dev 的分工。
- lmms-eval / 其他框架的踩雷點(例:cache 目錄名不可與 task 名相同)。

## Sources
實際讀過的 config、slurm script、CLAUDE.md、既有 run 目錄。
```

偏好具體、可檢查的規則,而非空泛描述。這份決定「未來每個 run 要長什麼樣才合格」。
