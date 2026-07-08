---
# D · 4D / 時空推理 Benchmark 盤點
**目的**：了解常見 benchmark 及彼此差異（會議記錄 D1）
**日期**：2026/07/07

---

## 對照表

| Benchmark | 規模 | 資料來源 | 任務焦點 | 靜/動態 | 與我們的關係 |
|---|---|---|---|---|---|
| **VLM4D** [2508.02095](https://arxiv.org/abs/2508.02095) (ICCV'25) | curated QA | 真實第三人稱 + 第一人稱 ego + 合成影片 | 平移/旋轉運動、視角覺察、運動連續性、計數、false-positive | **動態為主** | ⭐ 主要評測場;人類 98.8% vs 最佳 62.0% |
| **STI-Bench** | 300+ 真實影片、2000+ QA | 真實世界影片 | **靜態 + 動態**空間任務(精確量測、方向、速度) | 兩者 | 會議提到的 STI;強調「精確、可量化」 |
| **Spatial4D-Bench** [2601.00092](https://arxiv.org/html/2601.00092) | ~40,000 QA、18 任務 | 室內外多樣場景 | 最全面的 4D 空間智能(多物件/動作/場景) | 兩者 | 目前**規模最大**;可當廣覆蓋評測 |
| **VSI-Bench** | 5,000 QA、8 任務 | ScanNet / ScanNet++ / ARKitScenes | 3D 空間認知:配置、量測估計、時空推理 | 偏 3D 靜態 + 部分時空 | 3D 場景導向,depth 標註完整(對接 C1) |
| **DSI-Bench** [2510.18873](https://arxiv.org/html/2510.18873) | — | — | **動態空間智能**(Dynamic Spatial Intelligence) | 動態 | 專攻動態,與我們 flow 訴求最貼 |
| **VSI-SUPER** | — | — | 大規模時空推理 | 動態 | 資料多樣性/規模仍受限(據 survey) |
| **3DSRBench** | — | — | 3D 空間推理 | 靜態 3D | 細節待補 |

---

## 關鍵差異軸（寫簡報時的分類維度）

1. **靜態 vs 動態**：VSI-Bench / 3DSRBench 偏靜態 3D;**DSI-Bench / VLM4D / STI-Bench 才真正測動態(運動、時間一致性)** → 我們的 flow reward 應主打動態這批。
2. **真實 vs 合成 vs 掃描重建**：STI/VLM4D 用真實影片;VSI-Bench 用 ScanNet 等**掃描重建(自帶深度/3D)**;Spatial4D-Bench 混合。
3. **定性 vs 定量**：STI-Bench 強調**精確可量化**(速度、距離),適合當「連續 reward」的評測;許多 benchmark 仍是選擇題式。
4. **規模**：Spatial4D-Bench(~40k)> VSI-Bench(5k)> STI-Bench(2k)。

---

## 相關「方法」（非 benchmark,但同場競爭 / 可借鏡）
- **4DThinker** [2605.05997](https://arxiv.org/pdf/2605.05997):用 4D imagery 做動態空間理解的「思考」機制。
- **Video-STR** [2510.10976](https://arxiv.org/pdf/2510.10976):用 **relation graph** 強化 MLLM 的影片時空推理(RL)。
- **VLM-3R** [2505.20279](https://arxiv.org/pdf/2505.20279):用指令對齊的 3D 重建增強 VLM。
- **4D-VLA** [2506.22242](https://arxiv.org/html/2506.22242):時空 VLA 預訓練(跨場景校準)。

---

## D 對方向的收斂
- **主評測**:VLM4D(動態、與賣點對齊)+ **STI-Bench**(精確量化,適合展示連續 reward 的優勢)。
- **廣覆蓋**:Spatial4D-Bench(規模大)、DSI-Bench(純動態)。
- **depth 對照**:VSI-Bench(ScanNet 系,自帶深度)→ 也接上 C1(找已帶 depth 的資料集)。
- **敘事切點**:強調現有 benchmark **大多用「聚合 2D 特徵跨時間」** → 正是需要 depth/flow 這類 4D 訊號的地方(VLM4D 原文指出這點)。

### 待補（讀全文補精確數字）
- [ ] STI-Bench：確認任務類別與 metric(是否有速度/距離的數值題)。
- [ ] DSI-Bench：規模、任務、SOTA 差距。
- [ ] 3DSRBench：規模與任務定義。
- [ ] 各 benchmark 的 **SOTA vs 人類差距**一覽(強化「感知不足」論述,接 B1)。
