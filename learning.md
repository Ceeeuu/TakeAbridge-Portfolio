# learning.md — 給自己看的超詳細架構筆記

> 這份是寫給**你自己**的：把整個 TakeABridge 從頭到尾講清楚，讓你能對著它看懂系統、也能在面試時流暢地講。
> 標示規則：🟩 = **我負責**的部分　🟥 = 串接 / 後端（**不是我負責**，但要看得懂才能講整體）。

---

## 0. 一句話理解整個系統

> 「量測你的立場 → 把你和**立場最相反**的人（或一個對立 AI）配在一起對話 → 一邊對話一邊把你的推理畫成一張**概念網路圖**、偵測情緒與離題 → 對話後用數字證明你的立場/認知移動了多少 → 把好的觀點沉澱成知識庫。」

整個平台就是為了驗證一個研究假設：**跟對立 AI 對話，去極化效果能不能跟跟真人一樣好。**

---

## 1. 六大模組（先有大圖）

| 模組 | 名稱 | 白話 |
|---|---|---|
| M1 | Auth | 登入、發 JWT、帳號管理 |
| M2 | Topic & Stance | 選議題、填問卷、算出你的「立場分數」和「立場向量」 |
| M3 | Matching & AI Agent | 依立場把人配對；沒對手就生一個對立 AI |
| M4 | Dialogue Room | WebSocket 即時聊天；即時偵測情緒/離題/僵局並提示 |
| M5 | NLP & CCND | 把對話畫成概念網路圖、算立場漂移 |
| M6 | Summary & KB | 對話摘要、觀點知識庫 |

前端（🟩 我做 UI）把這些模組的畫面呈現出來；後端（🟥）實作邏輯；中間靠 REST + WebSocket 串（🟥 串接不是我做）。

---

## 2. 前端架構（React）

### 2.1 呈現層 vs 串接層（這個分界很重要，面試一定要講清楚）

前端刻意把兩件事分開：

- 🟩 **呈現層（我負責）**：頁面 (`pages/`)、元件 (`components/`) 只管「畫面長怎樣、使用者怎麼互動」，資料是「別人傳進來的 props」或「上層抓好給的」。
- 🟥 **串接層（不是我負責）**：`api/client.js`（axios 實例 + JWT 攔截器 + 401 自動 refresh）、WebSocket 連線、各頁面的資料抓取邏輯。

> 面試講法：「前端我負責的是**呈現與互動**——版面、元件、視覺化、動畫；跟後端的 API/WebSocket 串接是團隊另一位負責，我在 UI 這層是拿已經整理好的資料來畫。」

### 2.2 頁面清單（哪些是我 UI、哪些偏串接）

| 頁面 | 作用 | 我的角度 |
|---|---|---|
| `LoginPage` | 登入 | 🟥 串接（打 `/api/token/`）|
| `HomePage` | 首頁：問候 + 議題卡 + 捷徑 | 🟩 純 UI（資料由 props 傳入）|
| `TopicChat` | 核心對話頁：AI/真人對話 + 語意樹 + 讚倒讚 | 🟥 串接重鎮（REST + 兩條 WS）|
| `KnowledgeBase` / `KnowledgeBaseTopicPage` / `KnowledgeBaseConversationPage` | 觀點知識庫 | 🟥 串接 |
| `HistoryPage` | 歷史對話 | 🟥 串接 |
| `AchievementPage` | 成就 / 頭銜牆 | 🟩 純 UI（靜態資料 `achievements.data.js`）|
| `PostQuestionnairePage` → `SettlementReceipt` | 後測問卷 + 結算畫面 | 🟩 結算**視覺化**是我做（信封動畫、收據式圖表、匯出 PNG）；資料抓取是串接 |
| `DebriefingPage` / `PlatformFeedbackPage` | 匯報 / 回饋表單 | 🟥 串接 |
| `ViewpointReviewPage` | 研究者審核佇列 | 🟥 串接（後端 `IsResearcher` 把關）|
| `SettingsPage` | 帳號 / 設定 | 🟥 串接 |
| `GodotLobby` | 用 iframe 嵌入 Godot | 🟥 串接接縫（把 JWT/user 傳給 Godot）|

**元件（`components/`）幾乎都是 🟩 純 UI**：`Navbar`、`Sidebar`、`IssueCard`、`ActionCard`、`ConversationTreePanel`（D3 語意樹視覺化）、`AchievementToast`、`SurveyModal`…

### 2.3 我在前端最能拿出來講的三個東西

1. **CCND 概念網路圖（D3.js）**：把後端算好的「語意樹」資料，用力導向圖畫成可縮放、有層次的網路圖 —— 抽象的「推理脈絡」變成看得懂的圖。
2. **對話結算畫面**：後測送出後跳出一個**信封**，點擊有**開封動畫**，滑出「收據 / 明細」風格的結算：立場軌跡（1–7 刻度尺前→後）、思辨投入雷達圖、讚倒讚甜甜圖，並能一鍵**匯出整份長圖 PNG** 分享。全部圖表**手刻 inline SVG**（不裝圖表套件，配色過無障礙檢查）。
3. **成就系統 UI**：解鎖 toast、頭銜、成就牆。

### 2.4 路由與狀態（能看懂就好）
- `react-router-dom`；**未登入時所有路徑都渲染 LoginPage**，登入後才掛出完整路由（結構性 auth gate）。
- 狀態管理**沒有用 Redux**，就是 `useState`/`useEffect` + props 往下傳；JWT 和 user 存 `localStorage`。（這是 🟥 串接層的設計，不是我主導，但要看得懂。）

---

## 3. Godot 2D 虛擬大廳（🟩 我負責遊戲/世界；🟥 Backend.gd 串接不是我）

### 3.1 這是什麼
Godot 4.7 做的 2D 多人社交世界：兩個玩家在同一張地圖相遇 → 各自在頭上張貼一個「議題」→ 走近讀對方議題 → 用表情回應 → 發起 1 對 1 聊天。整個多人靠 **Godot 自己的多人連線（WebSocketMultiplayerPeer）**。

### 3.2 最核心的架構決定：RPC 掛在 player node、UI 只做本地呈現
- 每個玩家節點命名為 `str(peer_id)`，位在**每個 peer 上都相同的路徑** `/root/Game/<peer_id>` —— 這個「路徑一致」是 `rpc_id` 能精準送到對的人身上的關鍵。
- **所有跨 peer 的 RPC 都掛在 player node 上**；**UI（`game_ui.gd`，一個螢幕空間 CanvasLayer）完全不含 RPC、只做本地畫面**。UI 要做網路動作就呼叫本地 player，player 收到別人的事件再回呼 UI 顯示。
- 面試講法：「我把**網路邏輯**放在玩家節點、**畫面呈現**放在 UI 層，兩者分離；因為多人同步靠節點路徑一致，把 RPC 誤放到 UI 會讓跨端定位壞掉。」

### 3.3 議題 + 表情回應系統（我做的社交玩法）
- 張貼議題 → `apply_issue` 廣播到所有 peer → 每個人頭上泡泡更新、旁邊的人即時看到。
- 表情回應：讀議題時選 5 種 emoji 之一；作者端用一個 `{sender_id: emoji}` 字典保存「每位讀者一個表情」（可覆蓋），再廣播讓所有人重畫頭頂的表情列。
- **晚加入的人**：一連上就 `request_issue_sync`，每個作者把自己的議題和表情補送給他（順序有講究：先議題後表情）。

### 3.4 語音聊天（本地音訊 DSP，我做的）
麥克風 → 動態建立的音訊匯流排 → 變聲 / 殘響 / 破音（UI 滑桿即時調）→ 擷取（要傳的音框）→ 頻譜分析（UI 波形）。設計上**與網路解耦**：擷取和視覺化都拿「變聲後」的音訊。（先前把變聲範圍從 0.5–2.0 收窄到 0.7–1.5，就是為了讓變聲後仍聽得懂。）

### 3.5 網頁嵌入（顯示是我調的；串接是 Backend.gd）
- Godot 匯出成 **Web (WASM) build**，放在前端 `public/godot/`，用 iframe 嵌入。
- 顯示要點（我踩過的坑）：`stretch/aspect = expand` 讓畫面填滿不留 16:9 黑邊、`default_clear_color` 設灰色讓地圖外不是黑、canvas 撐滿 iframe。
- 🟥 **`Backend.gd` 是 Godot 對 Django 的唯一接縫（不是我負責）**：`guest_login`、`submit_issue`、`request_topic_match`（帶 service token 打 `/godot/match-rooms/`）。沒有後端時遊戲仍可本地玩，只是不持久化。

---

## 4. 後端架構（🟥 不是我負責，但要看得懂）

- **Django + DRF**：HTTP REST API（`/api/*`）。
- **Django Channels（ASGI）**：WebSocket（`/ws/*`），M4 對話、M5 即時樹更新用。
- **Services 層**：商業邏輯集中在 `apps/matching/services/`（配對、AI 代理、語意樹、NLP）等，views 只做薄薄的接口。
- **認證**：JWT（SimpleJWT），access token 內嵌 `is_researcher` 判斷研究者權限。

---

## 5. 資料庫與向量（🟥 後端；但這是系統的靈魂之一）

- 主庫 **PostgreSQL + pgvector**：業務資料和「對話語意向量」同一個庫。
- **為什麼向量放主庫**：每則發言的 embedding 每回合都在變、要和使用者/議題 JOIN、要交易一致地寫、還要用 SQL 端 `CosineDistance` 算「立場漂移」和「觀點去重」→ 放主庫最省事。
- **ChromaDB** 只放 RAG 靜態知識庫（離線建好、只被檢索）。這就是「動態向量 vs 靜態語料」的分工。
- 核心表：`UserStanceProfile`（立場輸入）、`DialogueMatch` + `MatchMessage`（真人）、`AIConversation`（AI）、`PostDialogueResponse`（結算）、`ViewpointNode`（知識庫）。所有 embedding 都 384 維、跨表可比。（細節見 `DATABASE.md`。）

---

## 6. AI / NLP 管線（🟥 後端；面試常被追問，講得出流程就贏一半）

六階段（細節見 `ARCHITECTURE.md §5`）：

1. **立場量測**：量表反向題校正後平均 → 1–7 分；開放題編成 384 維向量。
2. **異質配對**：`match_score = 0.6×量表距離 + 0.4×語意餘弦距離`，在「對立立場」硬閘門下**最大化**（刻意配最相反的人）；中立者 → 轉 AI。
3. **RAG AI 代理人**：從 ChromaDB 檢索 top-k 當依據，扮演**對立立場**，Claude 逐字串流（system prompt 快取省成本）。
4. **即時 NLP**：情緒（DistilBERT）、離題（餘弦 vs 議題錨點）、僵局（配對距離標準差）、立場漂移（發言均值 vs 基準向量）。
5. **CCND 語意樹**：每人一棵「root → 6 固定錨點 → 觀點節點」的推理樹；一般議題用 LLM 結構化 JSON、核能用本地兩階段 BERT；可時間軸回放。
6. **結算 + 知識庫**：後測算 `Δs`（立場移動）、`stance_centrism`（<0 = 去極化）；CCND 快照算相似度/新穎度；M6 用 pgvector 去重把好觀點寫進知識庫。

**一句話設計精神**：配對「最大化距離」+ 每人一棵獨立 CCND 樹，讓「真人 vs AI」兩組能一個人一個人公平比較。

---

## 7. 一次完整對話的資料旅程（把上面串起來）

```
登入 (M1)
  → 選議題、填前測問卷 (M2)
  → 後端算 stance score + q9 向量，存 UserStanceProfile
  → 單一入口 /api/dialogue/entry/ 依立場分流：
      ├─ 中立 → 建 AI session → WS 逐字串流對話 (M3/M4)
      └─ 立場相反 → 進配對佇列 → 配到人 → WS 配對室對話 (M3/M4)
  → 對話中：即時 NLP（情緒/離題/僵局/漂移）+ 建 CCND 樹 (M5)
     （前端 🟩 用 D3 把樹畫出來）
  → 對話結束 → 填後測 → 算 delta_s / stance_centrism (M6)
     （前端 🟩 用信封 + 收據結算畫面呈現）
  → 好觀點被萃取、去重、審核 → 進知識庫 (M6)
```

Godot 大廳是另一條入口：在 2D 世界遇到人 → 就地配對 → 一樣進對話流程。

---

## 8. 面試可能被問 + 怎麼答（都聚焦在我做的）

**Q：你在這個專案負責什麼？**
> 前端的 UI/UX 與視覺化，還有 Godot 的 2D 虛擬大廳。前端我負責頁面呈現、元件、CCND 的 D3 視覺化、對話結算畫面（含動畫和匯出 PNG）；Godot 我負責 2D 多人世界的場景、玩家與 UI、議題與表情互動、語音 DSP。**跟後端的 API/WebSocket 串接不是我負責**，我在 UI 層是拿整理好的資料來畫。

**Q：CCND 那個圖怎麼做的？**
> 後端把每個人的推理脈絡整理成一棵「語意樹」（root → 錨點 → 觀點節點）；我用 D3.js 的力導向圖把它畫成可縮放、有層次的網路圖，讓抽象的推理結構變成使用者一眼看得懂的視覺。

**Q：結算畫面為什麼這樣設計？**
> 教授希望有「樂趣」但又不能誤導實驗。所以評級只看「思辨投入」不看立場方向（避免獎勵使用者為分數改立場而污染數據）；視覺上用信封開封 + 收據明細（立場軌跡、雷達圖、甜甜圈），圖表全手刻 SVG 方便匯出成一張長圖分享。

**Q：Godot 多人架構有什麼要注意的？**
> 關鍵是「RPC 掛在玩家節點、UI 只做本地呈現」。因為多人同步靠節點在每端路徑一致，RPC 要放在 player node 才能用 `rpc_id` 精準送達；UI 只負責畫面，把網路和呈現分離。

**Q：前端狀態怎麼管？（若被問到串接）**
> 誠實講：串接層（api client、JWT、WebSocket）是團隊另一位負責的；我這邊是呈現層，用 props/局部 state 管畫面。但我看得懂整體：JWT 存 localStorage、401 會自動 refresh、對話走兩條 WebSocket。

---

## 9. 名詞速查

| 名詞 | 意思 |
|---|---|
| **CCND** | Conceptual Cognitive Network Diagram，概念認知網路圖，把一個人的推理脈絡畫成樹/網路 |
| **stance score** | 立場分數，1–7，4 為中立 |
| **delta_s** | 立場移動量 = 後測 − 前測 |
| **stance_centrism** | 去極化指標 = \|後−4\| − \|前−4\|，< 0 代表更靠近中立（去極化）|
| **embedding** | 把文字轉成的 384 維向量，用來算語意距離 |
| **pgvector** | PostgreSQL 的向量擴充，讓向量和業務資料同庫 |
| **RAG** | Retrieval-Augmented Generation，先檢索知識再讓 LLM 生成 |
| **異質配對** | 刻意把立場相反的人配在一起 |
| **RPC** | Remote Procedure Call，Godot 多人裡跨端呼叫函式 |

---

_這份筆記與 `README` / `ARCHITECTURE` / `API` / `DATABASE` / `TECHNICAL_OVERVIEW` 互相對應；想看圖去 ARCHITECTURE，想看端點去 API，想看表去 DATABASE，面試前掃這份 learning 就好。_
