# Freshness check —— landscape.md / conventions.md 要重用還是刷新?

`landscape.md` 與 `conventions.md` 建一次就重用。重用前做輕量檢查,判斷自上次調查以來實驗地景有沒有實質變動。目標:避免昂貴的全面重查,又不拿過期事實去規劃。

## 快速訊號

1. **Codebase drift**:repo 分支 / 關鍵 commit 是否變了?訓練或 eval 的 entry point / config 是否改過?→ 更新 landscape 的 Codebase 段。
2. **環境 drift**:torch/cuda/gcc pinning、venv、需補的 stub 是否變了?→ 刷新 conventions 的環境 pinning 段。
3. **資料 / benchmark drift(研究特有,最容易踩)**:dataset revision 是否更新?eval scoring 是否修過?held-out 是否換了?**這會讓舊 baseline 數字瞬間不可比** —— 一有變動,必須刷新 landscape 的 Baselines / Datasets 段,並在 experiment-plan 明講「新舊數字不可直接比」。
4. **新 baseline / 競爭工作**:這輪要比較的對象是否有新 paper / 新數字出來?→ 補 landscape 的相關工作段(web search，附來源)。
5. **與這輪的相關性**:即使地景大致沒變,這次要碰的**特定區域**上次可能沒深讀(看 landscape 的「不確定/尚未讀」)。若是,現在深讀並擴充。

## 決定

- **無實質變動 且 相關區域已調查過** → 兩份直接重用,在 experiment-plan 註明基於既有調查。
- **部分 drift** → 只更新受影響的段,更新「最後調查日期」,註記改了什麼。
- **重大 drift**(換 repo / 換 dataset 主版本 / 大改架構)→ 重查。

**資料 revision / eval scoring 的變動要特別敏感** —— 這是研究裡最隱形、最會毀掉可比性的 drift。有疑慮就讀 run log 與 config 確認,別默默拿舊數字當 baseline。
