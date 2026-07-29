# Architecture · 系統架構與流程

本文件以 Mermaid 圖說明 TakeABridge 的系統架構、前後端關係、資料流、API 流程、AI 流程與使用者操作流程。
所有圖皆為**設計層級**，不含任何原始碼或機密。

---

## 1. 系統架構（Layered Overview）

```mermaid
flowchart TB
    subgraph Client["前端 / Client"]
        R["React SPA"]
        D["D3.js CCND 視覺化"]
        G["Godot 2D 虛擬大廳 (WASM)"]
    end

    subgraph Edge["同源部署 (Cloudflare Tunnel, path-routed)"]
        Nginx["單一網域<br/>/ → 前端 · /api → 後端 · /godot → 遊戲 · /ws · /godot-ws"]
    end

    subgraph Backend["後端 / Django"]
        DRF["REST (DRF)"]
        CH["WebSocket (Channels/ASGI)"]
        SVC["Services 層<br/>matching · ai_agent · semantic_tree · nlp"]
    end

    subgraph Data["資料層"]
        PG[("PostgreSQL + pgvector")]
        CR[("ChromaDB")]
        RD[("Redis")]
    end

    subgraph AI["AI / NLP"]
        EMB["Sentence-Transformers"]
        LLM["Claude / OpenAI"]
        CLS["BERT / DistilBERT"]
    end

    R --> Nginx
    G --> Nginx
    Nginx --> DRF
    Nginx --> CH
    DRF --> SVC
    CH --> SVC
    SVC --> PG
    SVC --> CR
    CH -.->|channel layer| RD
    SVC --> EMB
    SVC --> LLM
    SVC --> CLS
    R -.-> D
```

**分層原則**：前端只透過 HTTP / WebSocket 與後端溝通；後端把商業邏輯集中在 Services 層；向量資料（每則發言 embedding）與業務資料同存 PostgreSQL（pgvector），RAG 靜態知識庫獨立放 ChromaDB。

---

## 2. Frontend / Backend 關係

```mermaid
flowchart LR
    subgraph FE["前端 React SPA"]
        UI["UI 頁面 / 元件<br/>(呈現層)"]
        CLIENT["api client<br/>(axios + JWT 攔截器)"]
        WS["WebSocket client"]
    end

    subgraph GD["Godot 遊戲端"]
        WORLD["2D 世界 / 玩家 / UI"]
        BSEAM["Backend.gd (串接層)"]
    end

    subgraph BE["後端 Django"]
        REST["REST API /api/*"]
        SOCK["WS /ws/*"]
        GWS["Godot 多人 WS /godot-ws"]
    end

    UI --> CLIENT
    UI --> WS
    CLIENT -->|"Bearer JWT<br/>401 自動 refresh"| REST
    WS -->|"?token=JWT<br/>串流訊息"| SOCK
    WORLD --> BSEAM
    BSEAM -->|"HTTP + service token"| REST
    WORLD -->|"WebSocketMultiplayerPeer"| GWS

    classDef mine fill:#e3efe0,stroke:#4c7a3c,color:#234;
    classDef seam fill:#f6e3e0,stroke:#b05540,color:#234;
    class UI,WORLD mine;
    class CLIENT,WS,BSEAM seam;
```

> 🟩 綠色 = 前端 UI / Godot 世界（**我負責**）　🟥 紅色 = 前後端串接層（**非我負責**）
>
> - 前端把「呈現」與「串接」分離：頁面/元件只管畫面，所有 REST/WS 呼叫集中在 `api client`。
> - `api client` 用攔截器統一加上 JWT，遇 401 會自動呼叫 refresh、去重複並重試。
> - Godot 對後端的呼叫全部收斂在單一 `Backend.gd` 接縫；沒有後端時遊戲仍可本地遊玩。

---

## 3. Database Flow（資料如何流動）

```mermaid
flowchart TD
    A["使用者填前測問卷"] --> B["計算 stance score (1-7)<br/>+ 開放題 embedding (384d)"]
    B --> C[("UserStanceProfile<br/>stance_score · q9_embedding")]

    C --> D{"立場分類"}
    D -->|支持/反對| E["進配對佇列<br/>MatchQueueEntry"]
    D -->|中立| F["建立 AI 對話<br/>DialogueSessionRecord"]

    E --> G[("DialogueMatch<br/>兩人 + 距離指標 + room_id")]
    G --> H[("MatchMessage<br/>content · embedding")]
    F --> I[("AIConversation<br/>prompt/response · embedding")]

    H --> J["即時分析<br/>emotion / drift / stalemate"]
    I --> J
    J --> K[("MatchStanceDrift<br/>drift 時間序列")]

    H --> L["建 CCND 語意樹"]
    I --> L
    L --> M[("semantic_tree_state<br/>存於 Match.stats / SessionRecord")]

    G --> N["對話結束 (on_commit)"]
    N --> O["M6 觀點萃取 pipeline<br/>品質過濾 + pgvector 去重"]
    O --> P[("ViewpointNode<br/>知識庫 · 人工審核")]

    A2["使用者填後測問卷"] --> Q[("PostDialogueResponse<br/>s_pre · s_post · delta_s · stance_centrism")]
```

**重點**：立場向量（`q9_embedding`）、每則發言向量（`embedding`）、觀點向量（`ViewpointNode.embedding`）全是同一 384 維空間、同存 pgvector，讓「配對距離」「立場漂移」「觀點去重」都能用 SQL 端的 `CosineDistance` 直接算。

---

## 4. API Flow（一次對話的請求時序）

```mermaid
sequenceDiagram
    autonumber
    participant U as 使用者 (React)
    participant API as Django REST
    participant WS as Channels WS
    participant AI as AI / NLP Services
    participant DB as PostgreSQL / pgvector

    U->>API: POST /api/token/ （登入）
    API-->>U: access / refresh JWT

    U->>API: GET /api/dialogue/topics/
    API-->>U: 可見議題清單

    U->>API: GET /api/dialogue/topics/{id}/survey/
    API-->>U: 前測問卷定義
    U->>API: POST /api/dialogue/entry/ （送出問卷）
    API->>AI: 計算 stance + embedding
    API->>DB: 寫入 UserStanceProfile
    API-->>U: route = ai 或 match

    alt 中立 → AI 對話
        U->>WS: 連 ws/dialogue/{session_id}/?token=JWT
        U->>WS: user_message
        WS->>AI: RAG 檢索 + LLM 生成
        WS-->>U: agent_stream（逐字串流）
        WS-->>U: agent_stream_end（stance_drift）
    else 立場相反 → 真人配對
        U->>API: POST /api/matching/join/
        API->>DB: 建立 DialogueMatch
        U->>WS: 連 ws/matching/rooms/{room_id}/?token=JWT
        U->>WS: match_message
        WS->>AI: 情緒/離題/僵局偵測
        WS-->>U: match_message + 必要時 AI 建議
    end

    U->>API: POST /api/post-questionnaire/
    API->>DB: 計算 delta_s / stance_centrism
    API-->>U: 去極化結算數值
```

完整端點清單見 [`API.md`](./API.md)。

---

## 5. AI Flow（去極化的 AI 管線）

```mermaid
flowchart TD
    subgraph S1["① 立場量測 + 向量化"]
        A["前測: 量表 Q1-Q8 + 開放題 Q9"] --> B["stance score 1-7<br/>(反向題校正後平均)"]
        A --> D["Sentence-Transformers<br/>MiniLM 384 維"]
        D --> E[("pgvector: q9_embedding")]
    end

    subgraph S2["② 異質配對 (最大化距離)"]
        B --> G["match score =<br/>0.6·量表距離 + 0.4·語意餘弦距離"]
        E --> G
        G --> H{"對立立場閘門<br/>找『最相反』的人"}
        H -->|配對成功| I["H-H 對話室"]
        H -->|中立/無對手| R
    end

    subgraph S3["③ RAG AI 代理人 (對立立場)"]
        R["AI 對話"] --> J["Chroma 檢索 top-k"]
        KB[("ChromaDB 知識庫<br/>離線建置")] --> J
        J --> K["脈絡 + 歷史 + 對話階段"]
        K --> L["Claude 串流生成<br/>(system prompt 快取)"]
        L --> M["逐句串流回使用者"]
    end

    subgraph S4["④ 即時 NLP 監測"]
        I --> N["情緒 (DistilBERT)"]
        M --> N
        N --> O["離題 (餘弦 vs 議題錨點)"]
        N --> P["僵局 (距離標準差)"]
        N --> Q["立場漂移 (發言均值 vs 基準)"]
    end

    subgraph S5["⑤ CCND 語意樹"]
        I --> T{"依議題分派"}
        M --> T
        T -->|一般| U["OpenAI 結構化 JSON<br/>固定 6 錨點 + 合併規則"]
        T -->|核能| V["本地兩階段 BERT<br/>macro → micro"]
        U --> W["每人一棵推理樹<br/>可時間軸回放"]
        V --> W
    end

    subgraph S6["⑥ 結算與知識庫"]
        W --> X["CCND 快照分析<br/>Jaccard 相似度 + 新穎度"]
        A2["後測問卷"] --> Y["delta_s + stance_centrism<br/>(去極化指標)"]
        W --> Z["M6 觀點萃取 → pgvector 去重"]
        Z --> KB2[("ViewpointNode 知識庫")]
    end
```

設計理由（為何本地 embedding、pgvector vs ChromaDB 分工、為何「最大化距離」）見 [`TECHNICAL_OVERVIEW.md`](./TECHNICAL_OVERVIEW.md#ai-流程與設計理由)。

---

## 6. 使用者操作流程（User Journey）

```mermaid
flowchart TD
    Start([進入平台]) --> Login["登入 / 訪客登入"]
    Login --> Home["首頁: 選議題"]
    Home --> Survey["填前測問卷 (立場量測)"]
    Survey --> Route{"系統依立場分流"}

    Route -->|中立| AIChat["與 AI 代理人對話"]
    Route -->|立場相反| Wait["配對等待"]
    Wait -->|逾時| AIChat
    Wait -->|配對成功| HHChat["與真人對話"]

    AIChat --> Tree["即時看 CCND 概念網路圖"]
    HHChat --> Tree
    Tree --> Post["填後測問卷"]
    Post --> Result["結算: 立場軌跡 + 思辨評級<br/>可匯出分享長圖"]
    Result --> KB["逛觀點知識庫"]

    Home --> Lobby["進 Godot 虛擬大廳"]
    Lobby --> Meet["遇到其他玩家 → 張貼議題 → 表情互動 / 語音"]
    Meet --> Match["就地配對進對話"]
    Match --> HHChat

    Result --> Ach["解鎖成就 / 頭銜"]
```

> 綠色路徑（前端頁面呈現、CCND 視覺化、結算畫面、成就、Godot 大廳互動）為**我負責**的部分；分流、配對、AI 生成、結算數值計算由後端負責。
