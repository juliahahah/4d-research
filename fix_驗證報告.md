# fix.md 驗證報告(2026/07/10)

逐項查證 fix.md 的宣稱後,已修改 `slides/TDL_7頁回答.pptx`(現為 11 頁)。以下分三類。

---

## 一、已驗證 ✅(已改進簡報)

| 宣稱 | 驗證方式 | 結果 | 改在哪 |
|---|---|---|---|
| MetaSpatial = arXiv 2503.18470 | 抓 arXiv abs 頁 | 標題/作者/內容完全吻合 | Q2 加入「已查證」 |
| GeoFlow (2605.18365) 幾何一致性 reward | 抓 abs 頁 | 屬實:用光流/深度位姿分離剛體與動態區,幾何一致性 reward + RL 微調(影片生成) | Q2 新增「影片生成側近例」bullet,結論改成兩段式 |
| Flow-GRPO 系列的 flow = flow-matching 非光流 | 與先前搜尋一致 | 屬實 | Q2 bullet |
| image.png = Flow4Agent Table 4/5 消融表 | 開圖確認 | 屬實(換 flow 模型 Overall 64.1→64.7 只差 0.6) | Q1 嵌入原表 + 消融 insight |
| TGO/MTP 全名與細節 | 讀本地 PDF method 段 | TGO=Temporal Granularity Optimization、MTP=Motion Token Pruning;Sea-RAFT(4/12 迭代)、U2-Net、homography 補償、top 50% | Q1 bullets 全面改寫 |
| ARKitScenes 授權=個人非商用 | Apple GitHub LICENSE | 屬實 | Q4 |
| Spring = CC BY 4.0 可商用 | 官網/論文 | 屬實 | Q4 |
| Virtual KITTI 2 = CC BY-NC-SA | NAVER Labs 頁 | 屬實,但版本是 **3.0**(fix.md 未寫版本) | Q4 |
| 規模不是瓶頸、授權才是 | 綜合以上 | 成立 | Q4 新 bullet |
| STI-Bench = 2503.23765、VSI-Bench = 2412.14171 | arXiv 搜尋 | 屬實 | Q8 表格加編號 |
| DA-V2 影片閃爍、輸出相對深度 | Video Depth Anything (2501.12375) 動機段 | 屬實(該論文動機即為此) | Q4「偽標風險已由文獻坐實」 |

## 二、部分驗證 ⚠️(已改,但保守寫法)

| 宣稱 | 出入 | 簡報處理 |
|---|---|---|
| AR-Drag (2510.08131) 有「motion consistency reward」 | 論文存在、確為 RL 影片生成,但**摘要用詞是 trajectory-based reward**,非該字樣(fix.md 引 ResearchGate 轉述) | Q2 寫「軌跡式 reward」,未驗證段註明用詞出入 |
| GRPO Survey (2603.06623) 列 optical-flow discriminator | 該篇確為 GRPO survey,但**摘要查不到此清單**,需讀全文 | 只寫進 Q2 未驗證段,不當論據 |

## 三、未驗證 ❌(未改進簡報)

| 宣稱 | 原因 |
|---|---|
| FlyingThings3D 96,336 幀 / VKITTI2 84,840 幀 / Spring 23,812 幀 | 與先前查到的數字(SceneFlow 全集 34,801 stereo 幀)衝突,兩邊都是二手來源 → **不放數字**,只寫「數萬幀級」 |
| Dynamic Replica 授權 | fix.md 自己也寫「需查」;未查到明確條款 → 留在 Q4 未驗證段 |
| VSTI-Bench、R4D-Bench 的獨立 arXiv 編號 | VSTI 查不到獨立編號;R4D-Bench 是 4D-RGPT 論文自建(無獨立編號)→ 表格如實標注 |

---

## 已完成的其他 fix.md 要求

1. **Q1 加消融**:image.png(Table 4/5)已嵌入,並補一條 insight:換 flow 模型只差 0.6 → 增益穩健但運動先驗邊際貢獻有限。
2. **Q8 加連結**:表格第一欄全部附 arXiv 編號(查得到的);查不到的如實標「編號未查得」。
3. **新增第 10 頁「VLM 統整」**:兩份簡報用到的模型分三類(被訓練的 base / 工具模型 / 閉源對照)。重點:8/10 篇 RL 論文 base 都是 Qwen2.5-VL-7B;judge 至少要 32B 級(Perception-R1 實測 7B judge 會被 reward hack)。

## 對 Q2 結論的重要修正(fix.md 的建議,查證後採納)

原句「flow 一致性當推理 reward 未見」→ 改為:
> **「以光流/運動一致性作為『文字推理』的 verifiable reward 未見;但作為『影片生成』的 RL/GRPO reward 已有近例(GeoFlow、AR-Drag)。」**

報告時要主動講與 GeoFlow 的差異:**它獎勵「畫得幾何一致」,我們獎勵「想得與運動證據一致」**——一個管生成品質,一個管推理正確性。
