# TakeABridge（橋得攏 · BridgeUs）

去極化對話平台 · 團隊研究專案（國科會大專生計畫）。
這個 repo 是我的**作品集**，展示我負責的部分；不含原始碼與機密。

**我負責：前端 UI / 視覺化（React · D3.js）＋ Godot 2D 虛擬大廳。**
（後端、AI/NLP、資料庫、所有前後端串接由其他成員負責。）

---

## 🎯 重點作品

### 1. CCND 概念認知網路圖 — D3.js 力導向圖
把後端算出的「推理脈絡語意樹」畫成**可縮放、有層次的力導向網路圖**，讓抽象的思考結構變成看得懂的視覺。
> 🖼️ `assets/ccnd-graph.png`

### 2. Godot 2D 虛擬大廳 — 多人社交世界（Godot 4.7）
玩家相遇、頭頂張貼議題、5 種表情互動、聊天、語音。核心架構：**RPC 掛玩家節點、UI 只做本地呈現**（多人同步靠節點路徑一致）。
> 🖼️ `assets/godot-lobby.png` · `assets/godot-issue.png`

其他：成就系統 UI、對話結算頁等前端頁面呈現。

📌 技術細節與 case study → [`CONTRIBUTIONS.md`](./CONTRIBUTIONS.md)

---

## 🧰 技術棧
- **我的**：React 19 · Vite · React Router · **D3.js** · Godot 4.7（場景/節點/UI/音訊 DSP）
- 團隊：Django · DRF · Channels · PostgreSQL+pgvector · ChromaDB · Sentence-Transformers · Claude

## 🙋 角色界線
- ✅ **我**：前端呈現/視覺化、Godot 2D 世界
- 🚫 **非我**：前端↔後端串接、Godot 的 `Backend.gd`、後端/AI/資料庫/配對演算法

## 📚 System Context（整體系統，多為團隊後端作品）
[`ARCHITECTURE.md`](./ARCHITECTURE.md)（架構/流程圖）· [`API.md`](./API.md) · [`DATABASE.md`](./DATABASE.md) · [`TECHNICAL_OVERVIEW.md`](./TECHNICAL_OVERVIEW.md)

---
純設計文件，無原始碼/金鑰/機密；程式碼在團隊私有 repo。
