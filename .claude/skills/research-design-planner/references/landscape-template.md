# landscape.md 模板

實驗的**實證地景**調查:靠讀 repo / config / 過往 run log / benchmark 定義 / 相關工作得到,不靠猜。每個數字(尤其 baseline、SOTA)都要能回溯到來源(paper / repo commit / 你自己的 run log)。無法驗證的就明說「未驗證」。

這是 project-design-planner 的 `knowledge.md` 的研究版:除了讀 repo,還要盤點「跟誰比、比什麼、資料是哪版」。

用以下結構:

```markdown
# 實驗地景(Landscape)

_最後調查日期:<date>。基於下方「Sources」列出的檔案與來源。_

## 研究目標與這輪要驗證的東西
一段話:這個研究方向要解什麼、這一輪實驗要驗證哪個假設(接 research-idea-review 的賣點/根因,如果有)。

## Codebase 現況
- 要跑的 repo / 路徑 / 分支 / 關鍵 commit
- 訓練與評測的 entry point(哪個 script、哪個 config)
- 環境雷區(torch 版本綁定、需補的 stub、LD_LIBRARY_PATH、venv 位置等 —— 從 repo 的 slurm/notes 讀,不猜)
- 資料落點:HF cache / TMPDIR / checkpoint / 結果輸出目錄

## Baselines(要比較的對象)
| 方法 / 模型 | 數字 | 來源(paper/repo/自己的 run log) | 是否可比(同 data rev? 同 eval scoring?) |
|---|---|---|---|
- **可比性是重點**:標明每個數字的 data revision、eval scoring 版本、prompt/setting。不可比就紅字標。

## Datasets / Benchmarks
| 名稱 | revision/版本 | 測什麼 | 已知缺陷 | held-out or dev? |
|---|---|---|---|---|
- 每個 benchmark「測什麼、資料來源、已知坑」對照(對齊 research-idea-review 第 5 步)。

## 相關 / 競爭工作(如與新方法比較)
近 6–12 個月最接近的 2–5 篇:做了什麼、跟本方法差異、數字。**一律附來源連結**,搜不到就標「搜尋範圍 X,未找到」。

## 過往 run 的既有結果
自己已經跑過的相關數字(從 run log / wandb / 你 memory),當這輪的參照點。標明各自的 config/rev。

## 不確定 / 尚未讀
相關但沒深讀的部分。誠實列出,讓計畫知道哪些是未驗證的。

## Sources
實際讀過的檔案、run log、以及 web 來源連結。
```

保持事實、可回溯。這份存在的目的是讓後續實驗規劃不必每次重新盤點地景。
