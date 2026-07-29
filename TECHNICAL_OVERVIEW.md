# Technical Overview（面試用技術總覽）

> 一頁看懂 TakeABridge 的目的、架構、技術選型理由，以及**我在其中負責什麼**。

---

## 1. 專案目的

社群媒體的推薦演算法把人推進同溫層，加劇社會極化。**TakeABridge** 想驗證一個假設：

> 讓立場相反的人進行**結構化的異質對話**，能量化地降低彼此的極化程度；
> 而與一個「有完整知識架構、無情緒干擾」的 **AI 代理人**對話，能達到與真人異質對話**同等甚至更穩定**的去極化效果。

因此系統要做到三件事：**準確量測立場 → 刻意把最相反的人配在一起（或給對立 AI）→ 用可量化指標與視覺化證明立場/認知確實移動了**。這是一個國科會大專學生研究計畫。

---

## 2. 系統架構

- **前端**：React SPA（參與者 + 研究者介面）、D3.js 呈現 CCND 概念網路圖、Godot 4.7 匯出的 2D 多人虛擬大廳（WASM，嵌在 iframe）。
- **後端**：Django + DRF（HTTP）+ Django Channels（WebSocket / ASGI），商業邏輯集中在 Services 層（配對、AI 代理、語意樹、NLP）。
- **資料**：PostgreSQL 16 + pgvector（業務資料 + 對話向量同庫）、ChromaDB（RAG 靜態知識庫）、Redis（Channel Layer）。
- **部署**：同源 path-routed（單一網域下 `/`、`/api`、`/godot`、`/ws`、`/godot-ws` 分流）。

系統分成 M1–M6 六大模組（Auth / Stance / Matching+AI / Dialogue / NLP+CCND / Summary）。完整圖見 [`ARCHITECTURE.md`](./ARCHITECTURE.md)。

---

## 3. 技術選型理由

| 決策 | 選擇 | 為什麼 |
|---|---|---|
| **對話向量存哪** | PostgreSQL + **pgvector** | 每則發言的 embedding 每回合都在變、要和使用者/議題 JOIN、要交易一致地寫入，還要用 SQL 端 `CosineDistance` 算漂移與去重 → 放主庫最省一致性成本 |
| **RAG 語料存哪** | **ChromaDB**（獨立）| 知識庫是**靜態、離線建置**、只被檢索 → 與動態業務向量分開，是「live vs static」而非重複 |
| **embedding 模型** | **Sentence-Transformers**（本地）| 不呼叫外部 API → 低延遲、零每則成本、多語（中英）；以 process singleton 載入、推論丟 thread pool 不擋 WebSocket 事件圈 |
| **兩個 embedding 模型** | 使用者內容用多語 MiniLM、RAG 語料用輕量 MiniLM | 使用者內容偏中文需多語模型；知識庫只在建置時編一次，用較輕模型即可，兩者皆 384 維但分屬不同 store |
| **即時通訊** | Django **Channels** + Redis | M4 對話室與 M5 CCND 即時更新都需要 WebSocket，統一技術棧 |
| **對話 LLM** | Claude（可切換 OpenAI/Gemini）| 串流體驗好；system prompt 用 prompt caching 降多輪成本；prompt 放外部檔案，迭代不動程式 |
| **CCND 節點分類** | 一般議題用 LLM 結構化 JSON、旗艦議題（核能）用**本地微調兩階段 BERT** | 有訓練資料的議題改用本地分類器 → 去除每則 LLM 成本、可重現、可調參；輸出格式與 LLM 路徑一致可直接替換 |
| **配對演算法** | **最大化**加權距離 + 對立立場硬閘門 | 研究目標是「有建設性的異議」，所以刻意配「最相反、語意最不同」的人，而不是找相似 |

---

## 4. AI 流程與設計理由

端到端六階段（詳見 [`ARCHITECTURE.md` §5](./ARCHITECTURE.md#5-ai-flow去極化的-ai-管線)）：

1. **立場量測**：量表反向題校正後平均 → 1–7 分；開放題編成 384 維向量存 pgvector。
2. **異質配對**：`match_score = 0.6 × 量表距離 + 0.4 × 語意餘弦距離`，在「對立立場」硬閘門下**最大化**此分數；中立者無真人對手 → 轉 AI。
3. **RAG AI 代理人**：從 ChromaDB 議題知識庫檢索 top-k 當依據，扮演**對立立場**，用對話階段（engagement→confrontation→convergence）調整策略，Claude 逐字串流。
4. **即時 NLP 監測**：情緒（DistilBERT）、離題（餘弦 vs 議題錨點）、僵局（配對距離標準差）、立場漂移（發言均值 vs 基準向量），全部 thread-pool 化避免阻塞。
5. **CCND 語意樹**：每人一棵「root → 6 固定錨點 → 觀點節點」的推理樹，每則訊息最多貢獻 2 個節點以維持可讀性；可依訊息時點回放。
6. **結算與知識庫**：後測算 `Δs`（立場移動）與 `stance_centrism`（<0 = 去極化）；CCND 快照分析算概念相似度與新穎度；M6 用 pgvector 去重把高品質觀點沉澱成知識庫。

**設計精神**：配對「最大化距離」+ 每人獨立的 CCND 樹，讓「真人 vs AI」兩組能一個人一個人地公平比較 —— 這正是研究成立的關鍵。

---

## 5. API Flow

單一入口 `POST /api/dialogue/entry/` 依立場把使用者分流到 AI 或真人；即時對話走 WebSocket（AI 逐字串流 / 真人配對室含 NLP 介入），對話後 `POST /api/post-questionnaire/` 回傳去極化結算數值。完整端點與時序見 [`API.md`](./API.md) 與 [`ARCHITECTURE.md` §4](./ARCHITECTURE.md#4-api-flow一次對話的請求時序)。

---

## 6. Database

PostgreSQL + pgvector 單庫存業務資料與對話向量；核心表圍繞 `UserStanceProfile`（立場輸入）、`DialogueMatch`/`MatchMessage`（真人對話）、`AIConversation`（AI 對話）、`PostDialogueResponse`（結算）、`ViewpointNode`（知識庫）。向量欄位統一 384 維，跨表可比。ER 圖與各表用途見 [`DATABASE.md`](./DATABASE.md)。

---

## 7. 我的負責內容

> 團隊研究專案。以下是**我實際負責**的範圍，界線清楚以免誤導。

### ✅ 我做的

**前端 UI / UX / 視覺化（React，呈現層）**
- 參與者 / 研究者介面的版面、元件庫、視覺設計與互動
- **CCND 概念網路圖的 D3.js 力導向圖**呈現
- 成就系統介面、對話**結算畫面**（信封開封動畫 + 收據式立場軌跡/雷達/甜甜圈，可匯出分享 PNG）
- 首頁、知識庫、歷史紀錄等頁面的呈現層

**Godot 2D 虛擬大廳（遊戲端）**
- 2D 多人世界場景、玩家與介面（player node ↔ UI CanvasLayer 分層架構）
- 議題張貼 + 5 種表情回應系統、頭頂泡泡、邀請/聊天互動
- 語音聊天本地音訊 DSP（變聲 / 殘響 / 頻譜視覺化）與大廳 UX

### 🚫 我沒做的（由其他成員 / 後端負責）
- **前端 ↔ 後端串接**（API client、JWT、WebSocket 接線、資料抓取）
- **Godot ↔ 後端串接**（`Backend.gd`）
- 後端、AI / RAG / NLP、資料庫、配對演算法

### 我從中學到 / 能談的
- 把「呈現」與「串接」解耦的前端分層思維（元件只管畫面，資料由上層注入）
- 用 D3 力導向圖呈現階層/網路資料、把抽象的「立場移動」轉成使用者看得懂的結算視覺
- Godot 多人架構中「RPC 掛在 player node、UI 只做本地呈現」的路徑解析設計，以及把 UI/呈現與網路/串接分離的重要性
