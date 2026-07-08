---
# C · 資料集與證據模態
**目的**：C1 找已帶 depth 的資料集；C2 探索 depth/flow 以外的證據模態
**日期**：2026/07/07

---

## C1 · 已帶 depth（與 flow）的資料集

### (a) 真實 RGB-D / 3D 掃描（自帶深度、幾何）
| 資料集 | 規模 | 標註 | 備註 |
|---|---|---|---|
| **ARKitScenes** [2111.08897](https://arxiv.org/pdf/2111.08897) | 5,048 RGB-D 序列 / 1,661 場景 | **雷射掃描 GT 深度** + 有向 3D bbox(17 類家具) | 比 ScanNet 大 3 倍;**有真 GT 深度** |
| **ScanNet / ScanNet++** | 1,513 掃描 / 707 場景 | 稠密 3D 標註 + CAD | ScanNet 缺 GT 深度;ScanNet++ 高保真 |
| **CO3D / MegaDepth / BlendedMVS** | 大 | 多視角深度/位姿 | 靜態為主 |

> VSI-Bench 就是建在 ScanNet / ScanNet++ / ARKitScenes 上 → **C1 與 D 的 VSI-Bench 相通**。

### (b) 同時有 optical flow + depth/scene-flow（對「flow reward」最關鍵）
| 資料集 | 提供的訊號 | 真實/合成 |
|---|---|---|
| **Spring** [spring-benchmark.org](https://spring-benchmark.org/) | 高解析 **GT 光流** + disparity(深度) | 合成、高品質 |
| **FlyingThings3D / Monkaa / Driving** [1512.02134](https://arxiv.org/pdf/1512.02134) | GT 光流 + disparity + **scene flow**,34,801 幀 | 合成(Blender) |
| **Virtual KITTI 2** | **GT scene flow** | 合成、駕駛場景 |
| **Dynamic Replica** | GT 光流 + **3D 點追蹤** | 合成、動態 |
| **PointOdyssey / Kubric** | **3D 點追蹤** | 合成、動態 |
| **TartanAir / Waymo** | 深度、位姿(Waymo 真實駕駛) | 混合 |

> 這些正是 **Flow4R / Any4D** 這類 4D 重建工作用的資料組合 → 我們的 flow-consistency reward 有現成 GT 可先驗證。

### (c) ⭐ 可行性關鍵：用基礎模型「偽標註」depth / flow（免 GT，可規模化）
- **Depth Anything V2** [GitHub](https://github.com/DepthAnything/Depth-Anything-V2)(NeurIPS'24):在 **6,200 萬張真實無標註影像**上產生深度偽標籤 → 證明「任何影片都能自動生深度」。
- **RAFT / SEA-RAFT**:光流的事實標準;**FlowSeek** [2509.05297](https://arxiv.org/abs/2509.05297)(ICCV'25)用**深度基礎模型 + 運動基底**做光流,單張消費級 GPU,跨資料集泛化超 SEA-RAFT 10–15%。
- **意義**:reward 不需人工深度/光流 GT —— 用現成基礎模型在**任何影片**上偽標,再拿偽標的一致性當 reward。這讓「連續 depth/flow reward」**可在大規模真實影片上實作**,不受標註資料稀缺限制。

---

## C2 · depth / flow 以外的其他證據模態

| 模態 | 內容 | 對 4D 的價值 | 取得方式 |
|---|---|---|---|
| ⭐ **Scene flow(3D 運動場)** | 每點的 3D 位移 | **統一 depth + flow 於單一訊號**,才是真正的「4D 證據」 | Virtual KITTI 2 / FlyingThings3D 有 GT |
| ⭐ **3D 點追蹤 / 對應** | 跨幀點軌跡 | 直接量測**時間一致性**(接 B1 弱點) | PointOdyssey / Dynamic Replica / Kubric |
| **相機位姿 (camera pose)** | 內外參、metric 座標 | 視角變化、ego-motion | 多視角幾何 / VG-LLM 類 |
| **表面法線 (surface normals)** | 每像素法向 | 幾何結構、平面 | 從深度導出 |
| **分割遮罩 (segmentation)** | 實例/語意 mask | 物件級接地(back-project→3D bbox) | SAM 類 |
| **點雲 / 體素 / mesh** | 顯式 3D 輸入 | N3D-VLM 用 RGB-D→3D bbox | 掃描 / 重建 |
| **BEV(鳥瞰圖)** | 由影片特徵重建 | 時空脈絡(GPT4Scene) | 重建 |
| **novel-view / 多視角證據** | 跨視角一致性 | 補單視角盲區 | 多視角幾何 |

### 相關方法(用「幾何基礎模型 prior」增強 VLM)
- **VG-LLM / Spatial-MLLM / VLM-3R** [2505.20279](https://arxiv.org/pdf/2505.20279):注入幾何基礎模型的 view-conditioned prior。
- **SpaceMind: Camera-Guided Modality Fusion** [2511.23075](https://arxiv.org/pdf/2511.23075):相機導向的模態融合。
- **Spa3R / N3D-VLM** [2602.21186](https://arxiv.org/pdf/2602.21186)、[2512.16561](https://arxiv.org/html/2512.16561):預測式空間場 / 原生 3D 接地。

---

## C 對方向的收斂
1. **C1 立即可用**:先用 **Spring / FlyingThings3D / Virtual KITTI 2 / Dynamic Replica**(有 GT flow+depth)驗證「flow-consistency reward」概念,再用 **ARKitScenes / VSI-Bench**(真實 + 深度)測泛化。
2. **可行性突破(寫進 method 的關鍵論點)**:**Depth Anything V2 + RAFT/FlowSeek 偽標** → reward 可在任意真實影片上計算,擺脫標註瓶頸,讓 GRPO 訓練規模化。
3. **C2 升級賣點**:比「depth+flow 分開」更強的訊號是 **scene flow(3D 運動)** 與 **3D 點追蹤(時間對應)** —— 兩者都直擊 B1 指出的弱點「整合視覺線索 + 維持時間一致性」。可考慮把 reward 訊號從「2D 光流一致性」升級為 **「scene-flow / 點追蹤一致性」**,新穎度與 4D 契合度更高。

### 待補
- [ ] 確認 Dynamic Replica / PointOdyssey 的授權與規模是否適合當訓練集。
- [ ] 評估 scene-flow 偽標(如 SceneFlow foundation model)成熟度 —— 若不成熟,先用 depth+flow 組合近似。
- [ ] 盤點「真實動態影片 + 可偽標」的來源(如 Waymo、Ego4D)供 GRPO 大規模訓練。
