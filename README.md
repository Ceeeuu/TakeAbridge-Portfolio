# TakeABridge（橋得攏 · BridgeUs）

TakeABridge 是一個促進異質觀點對話的去極化對話平台。

社群演算法容易把人推進同溫層、加深對立；這個平台的做法相反——依使用者的立場向量，把立場相反的人配對在一起，透過結構化的對話（真人對真人、真人對 AI 代理人）促進理解。對話過程中會把每個人的推理脈絡畫成概念認知網路圖（CCND），對話後用量化指標衡量立場的變化，並把整理過的觀點放進知識庫。

本專案已通過國科會大專學生研究計畫（計畫編號：115-2813-C-032-043-H），由團隊分工開發。以下是我在其中負責的部分。

---

## 前端畫面（React）

負責平台各頁面的介面與互動：

- 用 D3.js 把後端算出的推理脈絡資料呈現成概念網路圖（CCND）
- 首頁、知識庫、歷史紀錄、成就、對話結算等頁面

（截圖：`assets/ccnd-graph.png`）

## Godot 2D 虛擬大廳（Godot 4.7）

負責 2D 多人社交世界的畫面與互動：玩家相遇、在頭上張貼議題、用表情回應、聊天、語音。

（截圖：`assets/godot-lobby.png`、`assets/godot-issue.png`）

細節記錄在 [CONTRIBUTIONS.md](./CONTRIBUTIONS.md)。

---

## 技術

- 我用到的：React、Vite、React Router、D3.js、Godot 4.7
- 團隊其他部分：Django、DRF、Django Channels、PostgreSQL + pgvector、ChromaDB、Sentence-Transformers、Claude

## 分工

- 我：前端畫面呈現與 Godot 2D 世界
- 其他成員：前端與後端的串接、後端、AI / NLP、資料庫、配對演算法

## 系統整體文件

為了理解我的部分接在哪，附上整體系統的設計文件（多為團隊後端）：
[ARCHITECTURE.md](./ARCHITECTURE.md)、[API.md](./API.md)、[DATABASE.md](./DATABASE.md)、[TECHNICAL_OVERVIEW.md](./TECHNICAL_OVERVIEW.md)

---

內容為系統設計文件與畫面，不含原始碼、金鑰或機密；程式碼在團隊私有 repo。
