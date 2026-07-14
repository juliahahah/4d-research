# progress.md 模板

**研究進度板** —— 一眼看「現在在哪、什麼死了、什麼還活著、下一步做什麼」。這是專案級、跨輪的狀態,變動最快,所以獨立成一份短檔,別跟 `landscape.md`(穩定事實)或單一輪的 `experiment-plan.md` 混在一起。

用途:
- **Phase 1 開新一輪前先讀它** —— 不必翻一堆舊 plan 就知道走到哪、哪些路已排除。
- **Phase 4 每跑完一輪 / 每次 pivot 回來更新它。**
- 研究會 pivot;pivot 時把舊方向移到「已封存方向」,不要刪(踩過的坑是資產)。

保持短、可秒看。細節留在對應的 experiment-plan;這裡只放彙總與指標。

用以下結構:

```markdown
# 研究進度板

_最後更新:<date>。當前 active plan:<指向哪份 experiment-plan / 哪一輪>。_

## 當前方向(一句話)
痛點 → 方法 → 為什麼會 work。加一句「現在最想證明/否定的假設」。
_自 <date> 起走這個方向(前一個方向見「已封存方向」)。_

## 現在卡在哪(run ladder 位置)
- 目前階段:Stage 0 smoke / Stage 1 pilot / Stage 2 full —— 哪個 run 正在跑或待跑。
- 最近一個 gate:過 / 沒過 / 待判。

## 假設狀態表
| 假設 / 子問題 | 狀態 | 證據(指向哪輪 / run log) |
|---|---|---|
| ... | ✅ 已驗證 / ❌ 已否定(kill) / 🔄 驗證中 / ⏳ 待驗證 | ... |

## 實驗歷史(跨輪彙總,最新在上)
每一輪一行:跑了什麼 → 結論(confirm / kill / inconclusive)→ 對方向的影響。細節連到該輪的 experiment-plan 迭代 log。
- `<date>` 輪 N:<摘要> → <結論> → <影響>

## Blockers
現在擋住往下的東西:gated / 缺的資料、compute 排隊(QOS 2 job)、環境沒建好、待使用者拍板的決策。每條標「擋住什麼 + 需要什麼才解」。

## 下一步(近期 next actions)
最多 3–5 條具體可執行動作。壞例子:「多想 novelty」;好例子:「跑 Stage 1 pilot 驗證 X ablation,gate = STI ≥ N」。

## 已封存方向(pivot 歷史)
被放棄 / 取代的方向,一行一個:方向 → 為什麼放棄 → 何時。這是資產:避免哪天又繞回同一條死路。

## Sources / 相關文件
指向本輪 experiment-plan、相關 landscape 段落、外部 run log(wandb 等)。
```

這份的價值在於:隔一週回來、或換人接手,讀這一份就知道整個研究走到哪、為什麼、下一步該幹嘛,不必重跑一遍腦內脈絡。
