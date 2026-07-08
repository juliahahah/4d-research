---
# 文獻調查 · A1 / A2 / A3
**目的**：支撐 B1、確認方向新穎性
**日期**：2026/07/07

---

## A1 · Flow4Agent 如何使用 flow
### Flow4Agent: Long-form Video Understanding via Motion Prior from Optical Flow
arXiv 2510.05836 · ICCV 2025 ·（**首個把光流用於 LLM 影片理解**的工作）

- **定位**：**training-free**，用光流的 motion prior 幫長影片理解，不改模型權重。
- **兩個核心模組**：
  1. **TGO (Temporal Granularity Optimization)**：用**粗粒度 flow prior** 把相似畫面分組,再用語意 prior 濾掉無關場景 → 自適應的 frame 階層。
  2. **MTP (Motion Token Pruning)**：用**細粒度光流**剪掉 frame 內高冗餘的 video token。
- **成績**：Video-MME 64.7%、MLVU 71.4%、LongVideoBench 60.4%。
- **關鍵理解(對我們最重要的一點)**：Flow4Agent 把光流當成**「冗餘壓縮 / 證據聚焦的 prior」**（決定看哪些 frame、留哪些 token），**不是**當成 reward、也不是訓練訊號。
  - → **可借用點子**：它證明了光流能有效標定「哪裡有運動證據」。我們可把同一個 motion prior 從「token 剪枝」升級為 **reward 訊號**(獎勵模型的推理與光流證據一致)—— 這正是 Flow4Agent 沒做、留給我們的縫隙。

---

## A2 · 是否已有人把 depth / flow 放進 GRPO reward？（新穎性核心）

### 結論速覽
- **Depth 當 reward → 已相當擁擠**（多篇,見下）。純「深度當 reward」不再是新穎點。
- **Optical flow 當 reward → 幾乎空白**。現有的 flow 用法多為 *輸入 / CoT / token 剪枝*,而非 *可驗證 reward*。**這是最大的新穎縫隙。**

### (a) Depth / 3D 幾何當 reward —— 已有先例
| 論文 | reward 訊號 | 與 GRPO 關係 |
|---|---|---|
| **DepthLM: Metric Depth from VLMs** [researchgate](https://www.researchgate.net/publication/396048771) | 度量深度用**回歸型 metric 當 reward**(答案是數字) | GRPO + format reward;自動生成 3D 空間 VQA(規模達 20 億例) |
| ⭐ **Smooth Operator: Smooth Verifiable Reward Activates Spatial Reasoning** [arXiv 2601.07695](https://arxiv.org/abs/2601.07695) | **3D 物理約束**當可驗證訊號;用 **SNRA(動態 Sigmoid)**把 near-miss 誤差轉成**連續稠密 reward** | 提出 **AP-GRPO**(保留絕對數值梯度),Numerical3D-50k,達到與大規模監督相當 |
| **3D-R1 / 3D VLM 空間接地** [arXiv 2507.23478](https://arxiv.org/html/2507.23478v1) | format + **perception reward(物件定位)** + semantic reward | GRPO;ScanQA CIDEr 97.95→106.45 |
| **DepthRL(弱監督單目深度)** [Springer](https://link.springer.com/article/10.1007/s40747-025-01967-w) | **scale-invariant 深度誤差 + edge-aware 平滑**的負和當 reward | deep RL(非 VLM),無需稠密標註 |
| **MetaSpatial** | 渲染的 **depth / IoU reward** | GRPO;AR/VR 場景生成的空間推理 |
| **N3D-VLM** [arXiv 2512.16561](https://arxiv.org/html/2512.16561v1) | 用 RGB-D 輸入做原生 3D 接地(reward 導向定位) | 空間推理 |
| **SpaceVista / SpatialVLM** | 全尺度空間推理資料與訓練 | 空間推理資料生成 |

### (b) Optical flow / 運動一致性當 reward —— 幾乎沒有(縫隙所在)
| 論文 | flow 的角色 | 是否當 reward？ |
|---|---|---|
| **Flow4Agent**(見 A1) | motion prior 做 frame 選擇 / token 剪枝 | ✗ training-free,不是 reward |
| **Enhancing Spatial Reasoning via CoT + RL** [arXiv 2507.13362](https://arxiv.org/pdf/2507.13362) | **Optical Flow CoT**(用光流描述推理步驟) | ✗ 用於 CoT 推理鏈,非 reward 訊號 |
| **Video-R2 / STVG-R1 / VideoRLVR** [2511.23478](https://arxiv.org/pdf/2511.23478)、[2602.11730](https://arxiv.org/pdf/2602.11730)、[2605.15458](https://arxiv.org/html/2605.15458) | **時間一致性 / 時間對齊 reward(TAR)**、grounding reward | △ 有「時間一致性」reward,但**非以光流一致性為顯式可驗證訊號**;偏影片生成/接地 |
| 影片風格轉換 RL | 用 **RAFT 光流 warp** 量測 frame 間一致性 | △ 有,但屬影片生成領域,非 VLM 4D 推理 reward |

> **A2 的賣點結論**：把 **optical-flow 一致性當成 VLM 4D 推理的可驗證 reward** 目前基本是空白。可站在 Smooth Operator 的「連續可驗證 reward + AP-GRPO」肩膀上,把訊號從深度換/擴到 **光流一致性**,再套 MoCA 式的模態信用分配 → 三重差異化。

---

## A3 · Reward 設計提升能力（含視覺以外）
- **RLVR 總覽**（RL with Verifiable Reward）[Reinforcement Learning in Vision: A Survey, arXiv 2508.08189](https://arxiv.org/html/2508.08189v1)
  - 用**可程式化 / 自動可驗證**準則當 reward,降低對主觀人類偏好的依賴、緩解 reward hacking。
  - 典型:拿模型輸出對 ground-truth 做**二元 rule-based reward**,標註成本低。
- **現有 RLVR 的痛點(我們可攻擊的點)**：
  - 多數只用 **outcome-only** reward → 訊號**極度稀疏**,難以約束中間推理步驟。
  - 有研究指出 **RLVR 不會誘發全新推理模式**(RL 潛力 vs 實效有落差)[OpenReview 4OsgYD7em5](https://openreview.net/forum?id=4OsgYD7em5)。
- **過程 / 稠密 reward 方向(對接 B1 的 MoCA)**：
  - **Perception-centric Process Reward Models**(2604.24583):PRM 監督中間步驟 + token 級 advantage 重分配 → 解稀疏。
  - **From Faithfulness to Correctness: Generative Reward Models**(2509.25409)、**Efficient Reasoning via Reward Model**(2511.09158)。
- **連續 reward 手法**：Smooth Operator 的 **SNRA(Sigmoid 稠密化)** 是把稀疏 near-miss 轉稠密的可借範式(直接呼應我們的「連續 depth/flow 證據」)。

---

## 三題對 B1 / 方向的收斂

1. **A1 → 動機補強(接 E2)**：Flow4Agent 證明光流能有效定位運動證據,但只用在 training-free 的壓縮/聚焦。**把 motion prior 從壓縮升級為 reward** 是清楚的下一步。
2. **A2 → 新穎性定位(接 E1 賣點)**：
   - depth-as-reward **已擁擠** → 不能只賣這個。
   - **flow-consistency-as-verifiable-reward 幾乎空白** → **主打賣點**。
   - 技術地基:**Smooth Operator / AP-GRPO** 的連續可驗證 reward + **MoCA** 的模態信用分配。
3. **A3 → 方法正當性**：RLVR 的痛點(outcome-only、稀疏、不誘發新推理)正好被「連續 4D 證據 reward + 過程級信用分配」對症解決。

### 待補(讀全文)
- [ ] Smooth Operator(2601.07695)全文:SNRA / AP-GRPO 公式細節 —— 評估能否直接換成光流一致性。
- [ ] DepthLM 全文:回歸型 depth reward 的具體 metric 與資料生成流程。
- [ ] 確認 Video-R2 / VideoRLVR 的「時間一致性 reward」是否已隱含光流(避免撞題)。
