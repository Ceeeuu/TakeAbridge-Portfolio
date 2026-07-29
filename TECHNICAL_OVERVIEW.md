# 技術總覽

## 技術棧

| 層級 | 技術 |
|---|---|
| 前端 | React 19、Vite、React Router、axios、D3.js（CCND 力導向圖）|
| 遊戲端 | Godot 4.7（像素風 2D、TileMap、WebSocket 多人）|
| 後端 | Python、Django 6、Django REST Framework、Django Channels（ASGI／WebSocket）|
| 資料 | PostgreSQL 16 + pgvector、ChromaDB、Redis |
| AI / NLP | Sentence-Transformers（384 維語義向量）、Claude Sonnet（對話）、Gemini（CCND 概念擷取）、DistilBERT（情緒）、LangChain（RAG）|
| 部署 | Docker、Daphne + Gunicorn、Cloudflare |

## 幾個技術選型（簡述）

- **pgvector 與 ChromaDB 分工**：對話中動態產生的語義向量存 PostgreSQL（和業務資料同庫、可直接做向量查詢）；RAG 用的靜態外部知識存 ChromaDB。
- **WebSocket（Channels + Redis）**：對話與 CCND 需要即時雙向更新。
- **本地 Sentence-Transformers**：語義向量在本地算，低延遲、多語、無 API 成本。

## 團隊分工

| 成員 | 負責 |
|---|---|
| 賴則名 | 系統架構、H-AI/H-H 對話模組、CCND 架構、問卷與知識庫規範 |
| 伍晨安 | 前後端整合、後端 API、測試伺服器、除錯 |
| 黃筱筑 | 問卷計分、CCND 機器學習方法、測試優化 |
| **陳彩希（我）** | **前端介面、Godot 虛擬大廳** |
| 葉錦諦 | RAG 資料爬取、觀點知識庫架構與內容 |

> 我負責的細節與操作流程見 [CONTRIBUTIONS.md](./CONTRIBUTIONS.md)。
