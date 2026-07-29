# TakeABridge（橋得攏 · BridgeUs）

**TakeABridge 是一個對抗同溫層、促進異質觀點對話的去極化對話平台。**

現在的社群演算法把人不斷推進同溫層、加劇對立；這個平台反其道而行——依使用者的**立場向量**，刻意把**立場相反**的人配對在一起，透過結構化的對話（真人—真人、真人—AI 代理人）促進彼此理解。對話過程中，系統會即時把每個人的推理脈絡畫成**概念認知網路圖（CCND）**，並在對話後用量化指標衡量立場與認知移動了多少，最後把高品質的觀點沉澱成可瀏覽的知識庫。

核心研究假設：**與一個具備完整知識架構、無情緒干擾的 AI 對話，能達到與真人異質對話同等甚至更穩定的去極化效果。** 本專案為國科會（NSTC）大專學生研究計畫，由團隊分工開發。

以下整理我在這個專案中負責的**前端畫面**與 **Godot 2D 虛擬大廳**。
（後端、AI/NLP、資料庫、以及所有前後端串接由其他成員負責。）

---

## 前端畫面（React）

負責平台各頁面的 UI 呈現與互動：
- **CCND 概念網路圖的畫面** —— 用 D3.js 力導向圖呈現後端算出的推理脈絡
- 首頁、知識庫、歷史紀錄、成就、對話結算等頁面
> 🖼️ `assets/ccnd-graph.png`

## Godot 2D 虛擬大廳（Godot 4.7）

負責 2D 多人社交世界：玩家相遇、頭頂張貼議題、5 種表情互動、聊天、語音。
核心架構：**RPC 掛在玩家節點、UI 只做本地呈現**（多人同步靠節點路徑一致）。
> 🖼️ `assets/godot-lobby.png` · `assets/godot-issue.png`

📌 技術細節與 case study → [`CONTRIBUTIONS.md`](./CONTRIBUTIONS.md)

---

## 🧰 技術棧
- **我用到的**：React 19 · Vite · React Router · **D3.js** · Godot 4.7（場景/節點/UI/音訊 DSP）
- 團隊：Django · DRF · Channels · PostgreSQL+pgvector · ChromaDB · Sentence-Transformers · Claude

## 🙋 角色界線
- ✅ **我**：前端畫面呈現/視覺化、Godot 2D 世界
- 🚫 **非我**：前端↔後端串接、Godot 的 `Backend.gd`、後端/AI/資料庫/配對演算法

## 📚 System Context（整體系統，多為團隊後端）
[`ARCHITECTURE.md`](./ARCHITECTURE.md)（架構/流程圖）· [`API.md`](./API.md) · [`DATABASE.md`](./DATABASE.md) · [`TECHNICAL_OVERVIEW.md`](./TECHNICAL_OVERVIEW.md)

---
內容為系統設計文件與畫面，不含原始碼、金鑰或機密；程式碼在團隊私有 repo。
