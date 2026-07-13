# 4D 證據完整性 Reward — 簡報口頭稿

> 逐頁講稿，對應 `4D_evidence_reward_deep.pptx`（共 22 頁）。約 20 分鐘。

## 第 1 頁 · 開場 · 標題

各位老師、同學好。今天我要報告的是我們這個研究方向——也就是老師方向——的文獻定位。這個方向一句話講：用 GRPO、RLVR 這類強化學習的 reward，直接去獎勵『模型的答案跟推理』要跟連續的 depth、flow 這種 4D 幾何與運動證據保持一致，我們把它叫做『證據完整性 reward』。

為了把這個切入點站在哪裡講清楚，我深讀了十篇最相關的論文，橫跨 2025 到 2026 年的頂會。每一篇我都會講：它比較誰、解決前面方法什麼痛點、方法是什麼、怎麼驗證、用哪些 benchmark；如果是 RL 的論文，我還會拆它的 reward 設計、RL pipeline、RL 環境跟 RL 方法。大概二十分鐘。

## 第 2 頁 · 全景地圖 · 十篇定位表

先給大家一張全景地圖。這十篇，我依照『方法家族』跟『reward 或證據來源』排開，右邊的星等代表跟我們方向的相關程度。

最相關的三顆星有兩篇——VER 跟 4D-RGPT，等一下會重點講。二顆星大多是 RL 影片推理的家族。

請大家先看最下面這句話，這是我整份報告的核心觀察：整個家族，不管用什麼方法，reward 都落在時序、正確性、RGB 定位、或離散標註上；沒有任何一篇是用『連續的 depth、flow 4D 證據』去做『重答一致性』。這個空白，就是我們要切進去的地方。

## 第 3 頁 · 本方向定位

這頁把我們的方向拆成三塊。方法上，我們用 GRPO 這類強化學習，用 reward 去塑形推理，而不是用蒸餾。架構上，我們引入凍結的 4D 專家，抽出 depth 加 flow 的連續幾何與運動場當證據來源——這一點跟 4D-RGPT 一樣。但關鍵差在 reward 訊號：我們獎勵『重答、推理』跟這些 4D 幾何運動場的一致性。

下面這條金色帶把差異講得最白：多數 RL 影片推理用的是離散標註當 reward；唯一用 depth 加 flow 的 4D-RGPT 是蒸餾、reward 根本沒碰 4D；而我們就站在這個沒人佔的交集。

## 第 4 頁 · VER ①：問題與方法

第一篇，也是最重要的對照——VER，When Thinking Drifts，NeurIPS 2025，UT Austin。

它先講一個現象，叫 Visual Thinking Drift，視覺思維漂移：影片推理裡，CoT 想得越長，反而越容易依賴幻覺出來的事實、或不完整的片段，從語言慣性去推，而不是真的看畫面。最弔詭的是——很多錯的推理鏈在邏輯上是完美的，只是它脫錨了。他們還用貝式觀點解釋：softmax 裡語言權重遠大於視覺權重，鏈越長，視覺訊號被稀釋越嚴重。

方法上他們的洞察很關鍵：CoT 的 token 在訓練時從來沒被顯式監督過，所以會漂。他們用一個叫 inverted prompting 的技巧——餵進『問題加標準答案』，反過來要 teacher 生成『支持這個答案的視覺證據清單』，這是 question-dependent 的，不是泛用 caption。

## 第 5 頁 · VER ②：RL 深挖

VER 的 RL 這頁是重點，因為它跟我們最像。方法是 GRPO，每題 8 個 rollout，組內標準化。

核心的 Visual Evidence Reward 怎麼算——他們用一個 LLM judge，Llama-3.1-70B，去判斷模型的推理有沒有引用到相關、而且跟影片一致的視覺事實，給 0 或 1。注意這是語意層級的判斷，不是 token 比對。然後有個很重要的 gating：必須『答對、而且有引用』才拿得到這個 bonus，α 設 0.3；這樣就不會獎勵到『答錯但硬引用』。

pipeline 是兩階段，先 SFT 冷啟動、再 GRPO。結果很強：10 個 benchmark 有 9 個第一，平均比 base 進步 4%，抗幻覺的 benchmark 最高進步 9%。

跟我們的差別看最右邊：它驗的是 RGB 語意證據，不是 depth、flow 的連續 4D 場。

## 第 6 頁 · 4D-RGPT ①：P4D 蒸餾

第二篇三顆星，4D-RGPT，NVIDIA，CVPR 2026 highlight——這是唯一跟我們一樣用 depth 加 flow 的正面對手，而且是唯一有公開 code 的。

它要解的痛點是：MLLM 的 4D 感知跟時間理解很弱，連 GPT-4o 都很難同時做到區域追蹤、感知深度、跟感知時間進程。作者有個很強的主張：『純文字監督不足以支撐 4D 感知』。

方法叫 P4D，感知式 4D 蒸餾：teacher 是一個凍結的 4D 專家 L4P，會輸出 depth、flow、motion、camray 四種訊號；然後用兩條分支蒸餾——latent distillation 對齊中間表徵，explicit distillation 對齊輸出訊號——把這些能力灌進 MLLM 內部。重點是這些模組只有訓練時用，推論時零額外成本。時間感知則用 TPE 時間戳位置編碼解決。

## 第 7 頁 · 4D-RGPT ②：蒸餾 vs RL

這頁我特別做給我們自己看的。左邊是作者『為什麼選蒸餾、不選 RL』的論證：他們認為蒸餾能對『內部 4D 感知』做直接、逐像素的監督，而 RL 的 reward 只作用在文字答案層；而且他們觀察到 RL 方法 SpaceR，在他們的 region-level benchmark 反而輸給 Qwen，代表 RL 過擬合到非區域 VQA。

但右邊是我要反駁、也是我們的切入點：如果我們把 teacher 的 depth、flow 當成『可驗證的 reward』，其實就把 4D 訊號帶回了 reward 層，直接反駁『文字監督不足』這個論點；而且 RL 沒有 teacher 品質的天花板，還能誘發推理、補他們自己承認的『數值精度不足』。所以最有潛力的其實是蒸餾加 RL 的混合。

R4D-Bench 是他們自建的，第一個同時有動態場景加區域提示的 4D VQA。

## 第 8 頁 · Video-R1 ①：問題與方法

接下來是 RL 影片推理的『直接前身』——Video-R1，港中文，NeurIPS 2025。它是第一個系統性把 DeepSeek-R1 那套 rule-based RL 引進影片推理的。

痛點很直接：如果你把原始 GRPO 直接套到影片，沒有任何訊號鼓勵時序推理，模型會走捷徑、只看單一幀就作答；而且高品質的影片推理資料很稀缺。

他們的做法是自建兩個資料集，一個 16.5 萬做 SFT 冷啟動、一個 26 萬做 RL，而且刻意混影像資料進去——影像教通用推理，影片教時序。然後兩階段訓練。

## 第 9 頁 · Video-R1 ②：T-GRPO

Video-R1 的核心是 T-GRPO，時序感知的 GRPO。它的巧思在 reward：同一個問題，同時餵『時序正確順序』跟『隨機打亂順序』兩種 frame，如果有序組答對的比例比打亂組高，就給一個時序獎勵，α 是 0.3，而且只加到答對的回答上。

其他還有依資料型態的正確性獎勵——多選精確匹配、OCR 用字錯率、free-form 用 ROUGE——加上一個長度獎勵，鼓勵 320 到 512 token，避免想太多。

結果很亮眼：7B 在 VSI-Bench 拿 37.1%，贏過 GPT-4o 的 34。而且他們證明 RL 全面贏 SFT。對我們來說，它的 reward 只有時序加正確性，完全沒有 depth、flow 的幾何運動證據。

## 第 10 頁 · VideoChat-R1 ①：問題與方法

VideoChat-R1，上海 AI Lab，NeurIPS 2025，主打『資料高效的強化微調』。

痛點是：影片時空推理的 RL 少人做，而且之前的時序建模方法都要靠大量資料 SFT，結果犧牲了通用能力、還容易過擬合。

他們的做法是對五類任務各設規則式 reward，最後在三個時空感知任務——時間定位、追蹤、grounding QA——一起聯合訓練，總共才一萬八千筆。輸出用 think、answer，還有 clue 標籤帶時間線索。

## 第 11 頁 · VideoChat-R1 ②：規則式 RFT

它用 GRPO 做 RFT。reward 是規則式的組合：時間定位跟追蹤用 IoU reward，多選跟品質用 accuracy，字幕用一個很特別的 recall reward——把真值跟預測都拆成事件清單、算事件召回率——再加 format。

有個有趣的消融：用 GPT-3.5 還是 Qwen 72B 當評估器，結果幾乎一樣，因為 GRPO 靠的是組內相對差分，不是絕對分數。

結果：時間定位進步 31.8、追蹤進步 31.2，而且通用能力還保住了；對應的 SFT 反而在 VideoMME 上過擬合到不能作答。它一樣是純 task reward，沒有證據完整性、也沒有 4D 訊號。

## 第 12 頁 · Open-o3-Video ①：問題與方法

Open-o3-Video，北大加字節，投 ICML 2026。它把 OpenAI o3 那套『thinking with images』第一次延伸到影片。

痛點是：多數影片推理只給純文字 CoT，不講證據在『何時、何地』，沒辦法追溯、驗證；之前頂多做粗粒度的時序定位。

他們的方法是讓模型在推理鏈裡直接嵌入明確的時空證據——物件、bounding box、加上時間戳。他們遇到一個關鍵問題叫 spatial collapse：空間 reward 依賴正確的時間戳，早期時序不準的時候，空間 reward 幾乎是零、學不動。他們用兩個機制解：adaptive temporal proximity，早期放寬、後期收緊；還有 temporal gating，只有時間對得夠準才計空間分。

## 第 13 頁 · Open-o3-Video ②：GSPO

它用的不是 GRPO，是 GSPO——在序列層級做 importance ratio 跟 clipping，把整條含時間戳跟 bbox 的軌跡當成一個原子單位優化，避免 CoT 崩潰。

reward 三塊：accuracy 依任務用 IoU 或 ROUGE；thinking reward 拆時序項跟空間項，時序用高斯 exp 負 Δt 平方，σ 從 4 退火到 1；空間項有 gating，錯的時間戳就不給分；再加 format。

結果 V-STAR 的 mAM 進步 14.4%、mLGM 進步 24%，贏 GPT-4o。它的證據是 RGB 的 bbox 跟 timestamp，是離散標註，不是我們的連續 4D 場重答一致性。

## 第 14 頁 · Perception-R1 ①：問題與方法

Perception-R1，中國科大，ICLR 2026。它問了一個很尖銳的問題：多模態推理其實分感知跟邏輯，感知是前提。但他們發現，只用『答案正確』的 RLVR，只提升邏輯、不提升感知。

他們用 McNemar 統計檢定證明：accuracy-only 的 RLVR 訓練前後，感知能力沒有顯著差異，p 值 0.22、0.69，都遠大於 0.05；而且七成以上的錯誤來自感知錯誤。

方法是加一個視覺感知獎勵：先用 Gemini 生 CoT、只留答對的，再用 Qwen 32B 抽出原子級的視覺事實當參考標註。

## 第 15 頁 · Perception-R1 ②：Visual Perception Reward

它用 GRPO，而且刻意不用 reward model、還移除了 KL，避免 reward hacking。

核心的 visual perception reward 就是：訓練時用一個 judging LLM 去判斷模型的 rollout 有沒有命中每一條視覺標註，算命中率。完整 reward 是 format、accuracy、感知、加重複懲罰，權重 0.1、0.9、0.7。

最驚人的是資料效率：只用 1442 筆，比 Vision-R1 的 20 萬少一百倍以上，多數 benchmark 還贏。而且他們對自己的模型再做一次 McNemar，p 值 0.04，證明感知真的顯著改善了。它對齊的是文字化的視覺標註命中率，不是連續場的重答一致性。

## 第 16 頁 · Video-STR ①：問題與方法

Video-STR，字節，ICLR 2026。痛點是：像素級定位只在影像平面，推不出真實物理空間的佈局；而 2D 認知地圖在鏡頭視角一變，就不是旋轉不變的，物件分佈就估錯。

他們的方法是讓模型在 think 的過程裡『想像』出一張物件關係圖：節點是物件位置，邊帶距離跟角度。關鍵洞察是——物件跟物件的關係，對 ego 視角的變動不敏感，所以是旋轉不變的，他們還用定理證明。這張圖不是外部模組，是模型自己在推理裡生成，再由圖獎勵去監督它對不對。

## 第 17 頁 · Video-STR ②：graph-based GRPO

它叫 graph-based GRPO：在標準 GRPO 之外，加一個顯式監督圖拓撲的獎勵。這個 graph reward 分節點跟邊：節點用物件數量加權、算預測跟真值中心點的距離；邊算距離跟角度的一致性，影片的話就對選出的幀取平均。再加答案獎勵、format、跟長度獎勵。

資料自建了 STV-205k，從 TAO、ScanNet、KITTI 來。結果 STI-Bench 相對進步 13%、VSI-Bench 從 33 拉到 46.5。

他們有句話很好——RLVR generalizes, SFT memorizes，他們的 SFT 版在多任務會退化。但它用的是離散的關係圖，不是連續 depth、flow 場，而且需要物件級標註。

## 第 18 頁 · STVG-R1 ①：問題與方法

STVG-R1，西電加 BIGAI，投 ICLR 2026，是第一個把 RL 用到時空影片定位 STVG 的框架。

痛點是座標幻覺：模型常輸出超界的時間戳跟座標，密集逐幀 bbox 更嚴重。之前的對齊派要輸出逐幀座標、多物件很難處理；解碼器派又輸出隱式、泛化受限。

他們的巧思是物件中心的視覺提示，而且是 training-free 的：給每個物件一個唯一、時序一致的數字 ID，疊到畫面上，把『預測逐幀座標』重構成『辨識物件 ID』這種緊湊又可解釋的問題。資料 pipeline 用 YOLO 加 SAM2 加週期性 re-detection 自動生成。

## 第 19 頁 · STVG-R1 ②：ID 一致性 reward

它用 GRPO，每題 8 個候選。reward 三塊相加：時間用 tIoU；空間一致性是一個稀疏的 0 或 1——只有『預測 ID 等於真值 ID，而且這個 ID 出現在定位的時段內』才給 1，他們證明這比連續或耦合的空間獎勵更好；再加 format。

結果 HCSTVG-v2 進步 20.9%，而且零樣本遷移到 MeViS 這種多物件分割還拿 SOTA。消融顯示視覺提示主要提升空間、GRPO 主要提升時間。

對我們來說，它做的是定位不是 QA，視覺提示也是 RGB 疊 ID，不是 4D 證據。

## 第 20 頁 · Flow4Agent（非 RL · 佐證）

接下來兩篇是非 RL，但對我們的動機很重要。Flow4Agent，北大深圳加鵬城實驗室，ICCV 2025，是第一個把光流的『運動先驗』引進長影片理解，而且是 training-free。

痛點是長影片時空冗餘太大：統一取樣會漏資訊，語意先驗又依賴 query 細節、還會被 CLIP 的錯誤扭曲。它兩層去冗餘：TGO 在跨幀層用 HSV 加光流切事件、再用假設檢定選事件；MTP 在幀內層做相機運動補償加顯著性，剪掉運動幅值低的 token。

結果 VideoMME 64.7%，7B 就贏 GPT-4V。對我們來說，它把 flow 當『效率工具』做 token 剪枝，不是 RL 的證據 reward——但它正好佐證了光流運動先驗是有價值的。

## 第 21 頁 · PixFoundation 2.0（非 RL · 動機）

最後一篇是佐證、不是對手：PixFoundation 2.0，NeurIPS 2025。它問一個診斷性的問題：影片 MLLM 在做像素級定位的時候，到底有沒有真的用到運動？

他們發現既有的 motion 基準有漏洞——很多『運動』的指代表達式，其實光靠單一靜態幀的外觀線索就能定位。他們設計了四種 motion-centric 探測、還做了 MoCentric-Bench。

最關鍵的發現是：一個完全不含時序、只用單幀的 baseline，居然就能匹敵甚至超越舊方法；而一旦加上真正的運動探測，所有模型效能大概腰斬。這直接證明了——現在的影片 MLLM 根本沒真正用運動、也就是 4D 證據。這正是我們方向的動機。

## 第 22 頁 · 定位總結

總結一下我們的定位。整個 4D-VLM 版圖上有三條軸：

第一，我們用 RL 而不是蒸餾——4D-RGPT 有 depth、flow 卻是純蒸餾，受 teacher 天花板限制。

第二，我們用連續 4D 場而不是離散標註——Open-o3、Video-STR、STVG-R1 的證據都是 bbox、timestamp、關係圖或 ID。

第三，我們做 4D 證據完整性 reward——VER 跟 Perception-R1 只驗 RGB 語意證據。

把這三條交在一起：用 RL、用連續 depth flow 4D 證據、做重答一致性 reward，這十篇裡沒有任何一篇同時佔據。這個空白就是我們的切入點。而 PixFoundation 已經證明現有模型沒真用運動、Flow4Agent 也證明運動先驗有價值，所以動機是站得住的。謝謝，以上是我的報告。
