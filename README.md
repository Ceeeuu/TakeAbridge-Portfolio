# TakeABridge（橋得攏 · BridgeUs）

TakeABridge 是一個促進異質觀點對話的去極化對話平台。

社群演算法容易把人推進同溫層、加深對立；這個平台的做法相反——依使用者的立場向量，把立場相反的人配對在一起，透過結構化的對話（真人對真人、真人對 AI 代理人）促進理解。對話過程中會把每個人的推理脈絡畫成概念認知網路圖（CCND），對話後用量化指標衡量立場的變化，並把整理過的觀點放進知識庫。

本專案已通過國科會大專學生研究計畫（計畫編號：115-2813-C-032-043-H），由團隊分工開發。以下是我在其中負責的部分。

---

> **完整操作流程（含建議截圖）見 [CONTRIBUTIONS.md](./CONTRIBUTIONS.md)。**

## 前端畫面（React）

負責平台各頁面的介面與互動，其中最主要的是用 D3.js 把後端算出的推理脈絡資料呈現成**概念認知網路圖（CCND）**，以及首頁、成就等頁面。

`[截圖：CCND 概念網路圖，對話進行到一半、已長出多個節點、看得出層次的狀態]`

## Godot 2D 虛擬大廳（Godot 4）

負責一個像素風的 2D 多人互動空間：玩家在世界中自由移動、發表議題、閱讀彼此頭頂張貼的議題。

`[截圖：虛擬大廳世界全景，看得到地圖與角色]`
`[截圖：角色頭頂的議題泡泡]`

---

## 技術

- 我用到的：React 19、Vite、React Router、D3.js、Godot 4
- 團隊其他部分：Django 6、DRF、Django Channels、PostgreSQL + pgvector、ChromaDB、Sentence-Transformers、Claude、Gemini、LangChain

## 分工

- 我：前端畫面呈現與 Godot 2D 世界
- 其他成員：前端與後端的串接、後端、AI / NLP、資料庫、配對演算法

## 系統整體文件

為了理解我的部分接在哪，附上整體系統的設計文件（多為團隊後端）：
[ARCHITECTURE.md](./ARCHITECTURE.md)、[API.md](./API.md)、[DATABASE.md](./DATABASE.md)、[TECHNICAL_OVERVIEW.md](./TECHNICAL_OVERVIEW.md)

---

內容為系統設計文件與畫面，不含原始碼、金鑰或機密；程式碼在團隊私有 repo。
