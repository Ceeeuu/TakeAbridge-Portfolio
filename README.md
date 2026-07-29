# TakeABridge（橋得攏 · BridgeUs）

TakeABridge 是一個促進異質觀點對話的去極化對話平台。

社群演算法容易把人推進同溫層、加深對立；這個平台的做法相反——依使用者的立場向量，把立場相反的人配對在一起，透過結構化的對話（真人對真人、真人對 AI 代理人）促進理解。對話過程中會把每個人的推理脈絡畫成概念認知網路圖（CCND），對話後用量化指標衡量立場的變化，並把整理過的觀點放進知識庫。

本專案已通過國科會大專學生研究計畫（計畫編號：115-2813-C-032-043-H），由團隊分工開發。以下是我在其中負責的部分。

---

> **完整操作流程（含建議截圖）見 [CONTRIBUTIONS.md](./CONTRIBUTIONS.md)。**

## 前端畫面（React）

負責平台各頁面的介面與互動，其中最主要的是用 D3.js 把後端算出的推理脈絡資料呈現成**概念認知網路圖（CCND）**，以及首頁、成就等頁面。

![CCND 概念網路圖](assets/ccnd-graph.png)

## Godot 2D 虛擬大廳（Godot 4.7）

負責一個像素風的 2D 多人互動空間：玩家在世界中自由移動、發表議題、閱讀彼此頭頂張貼的議題。

![虛擬大廳世界](assets/godot-lobby.png)
![發表議題](assets/godot-issue-post.png)

---

## 技術

- 我用到的：React 19、Vite、React Router、D3.js、Godot 4.7
- 團隊其他部分：Django 6、DRF、Django Channels、PostgreSQL + pgvector、ChromaDB、Sentence-Transformers、Claude、Gemini、LangChain

## 團隊分工

| 成員 | 負責 |
|---|---|
| 賴則名 | 系統架構、對話模組、CCND 架構、問卷與知識庫規範 |
| 伍晨安 | 前後端整合、後端 API、測試伺服器 |
| 黃筱筑 | 問卷計分、CCND 機器學習方法、測試優化 |
| **陳彩希（我）** | **前端介面、Godot 虛擬大廳** |
| 葉錦諦 | RAG 資料、觀點知識庫 |

## 系統整體文件

理解整個平台與我的部分接在哪：
[ARCHITECTURE.md](./ARCHITECTURE.md)（系統整體）、[API.md](./API.md)、[DATABASE.md](./DATABASE.md)、[TECHNICAL_OVERVIEW.md](./TECHNICAL_OVERVIEW.md)

---

內容為系統設計文件與畫面，不含原始碼、金鑰或機密；程式碼在團隊私有 repo。
