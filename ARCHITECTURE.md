# 技術架構

本文件說明 TakeABridge 整體的技術架構。

---

## 整體分層

![系統技術架構圖](assets/architecture.svg)

系統由上而下分為五層：

| 層級 | 內容 | 職責 |
|---|---|---|
| 客戶端 | React 19 前端（D3.js CCND 視覺化）、Godot 4.7 虛擬大廳 | 使用者介面、對話與思維圖呈現、2D 互動空間 |
| 接入層 | Django REST Framework + Simple JWT、Django Channels（ASGI） | HTTP API、身分驗證、WebSocket 雙向通訊 |
| 應用模組 | M1 使用者、M2 立場量測、M3 配對、M4 對話室、M5 NLP/CCND、M6 摘要/知識庫 | 各業務邏輯 |
| AI / NLP 服務 | Claude Sonnet、Gemini、Sentence-Transformers、DistilBERT、LangChain | 對話生成、概念擷取、語義向量、情緒偵測、RAG |
| 資料層 | PostgreSQL 16 + pgvector、ChromaDB、Redis 7 | 結構化資料 + 語義向量、RAG 知識向量、快取與訊息代理 |

前端透過 **HTTP** 打 REST API（登入、問卷、報告等一次性請求），透過 **WebSocket** 維持對話室的即時雙向連線；Godot 大廳同樣以 WebSocket 與後端同步位置與標籤互動。

---


> 使用者實際操作流程見 [USER_FLOW.md](./USER_FLOW.md)。
