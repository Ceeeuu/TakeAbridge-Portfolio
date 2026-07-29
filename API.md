# API Reference · REST & WebSocket

> ℹ️ **System Context（非本人作品）**：這份完整 API 對照表屬**後端團隊**的設計，放在這裡是為了呈現整體系統與我理解全貌。我負責的是**前端呈現層**，串接（實際打這些 API）由團隊其他成員負責。我的作品見 [`CONTRIBUTIONS.md`](./CONTRIBUTIONS.md)。

TakeABridge 後端以 Django REST Framework + Django Channels 提供服務。以下為**設計層級**的 API 契約對照表，不含實作、金鑰或請求範例中的真實資料。

- Base prefix：`/api/`
- 認證：JWT（SimpleJWT），access token 內含額外的 `is_researcher` claim
- 角色：**Any** 已登入者／**Researcher** 研究者群組／**Staff** 後台管理／**Service** Godot 服務 token／**Public** 免登入

> 本頁描述介面契約，用於展示系統設計；欄位為高階示意，非完整 schema。

---

## M1 — Auth / Accounts / Platform Settings

| API | Method | URL | Request | Response | 功能 |
|---|---|---|---|---|---|
| Obtain token | POST | `/api/token/` | `username`, `password` | `access`, `refresh` | 登入取得 JWT |
| Refresh token | POST | `/api/token/refresh/` | `refresh` | `access` | 換發 access token |
| Guest login | POST | `/api/guest/` | `nickname?` | `access`, `refresh`, `user_id`, `username` | 建立訪客帳號（Public）|
| Current identity | GET | `/api/me/` | — | `id`, `username`, `is_researcher`, `entry_mode` | 即時查角色 / 入口模式 |
| List / create accounts | GET, POST | `/api/accounts/` | POST: `username`, `password`, `is_researcher` | 帳號清單 | 帳號管理（Researcher）|
| Update account | PATCH | `/api/accounts/<pk>/` | `is_active`, `is_researcher` | 帳號物件 | 啟用/停用、升降權（Researcher）|
| Reset password | POST | `/api/accounts/<pk>/reset-password/` | `password` | `detail` | 研究者重設密碼 |
| Display settings | GET, PATCH | `/api/settings/display/` | `participant_entry_mode`, `researcher_entry_mode`, `match_fallback_timeout_minutes` | 平台設定 + 各議題設定 | 入口模式與逾時設定（Researcher）|
| Topic display override | PATCH | `/api/settings/display/topics/<topic_id>/` | 可見性、`support/oppose_threshold` | 議題設定列 | 各議題可見性與立場門檻（Researcher）|

## M2 — Topic & Stance

| API | Method | URL | Request | Response | 功能 |
|---|---|---|---|---|---|
| List topics | GET | `/api/dialogue/topics/` | — | `[{id, title, description, date}]` | 依角色過濾的議題清單 |
| Trending topics | GET | `/api/dialogue/topics/trending/` | — | `[{id, title, hits}]` | 依活動量排序的熱門議題 |
| Topic survey | GET | `/api/dialogue/topics/<id>/survey/` | — | 量表 / 立場規則 / 題目 | 前測問卷定義 |
| Stance profile | GET | `/api/dialogue/topics/<id>/stance-profile/` | — | `exists`, `stance_score`, `stance_category`, `survey_answers` | 檢查 / 重用既有立場 |
| Dialogue entry | POST | `/api/dialogue/entry/` | `topic_id`, `survey_answers`, `survey_open_answers` | `route` = `ai`\|`match` + payload | 單一入口：依立場分流至 AI 或真人 |
| Entry fallback | POST | `/api/dialogue/entry/fallback/` | `topic_id` | `route: "ai"` + payload | 等待過久改走 AI |

## M3 — Matching & AI Agent

| API | Method | URL | Request | Response | 功能 |
|---|---|---|---|---|---|
| Join queue | POST | `/api/matching/join/` | `topic_id`, `survey_answers`, `restart_existing_match?` | 配對狀態 payload | 加入真人配對佇列 |
| Matching status | GET | `/api/matching/status/` | `topic_id` (query) | 配對狀態 payload | 輪詢配對狀態 |
| Cancel matching | POST | `/api/matching/cancel/` | `topic_id` | 配對狀態 payload | 離開佇列 |
| Create AI session | POST | `/api/dialogue/sessions/` | `topic_id`, `survey_answers`, `user_initial_argument` | `session_id`, `dialogue_phase`, `stance_*`, `history` | 建立 AI 對話 session |
| Latest AI session | GET | `/api/dialogue/sessions/latest/` | `topic_id` (query) | session payload | 續接最近一個 session |
| AI session detail | GET | `/api/dialogue/sessions/<session_id>/` | — | session payload | 取單一 session 狀態 |
| AI session reply | POST | `/api/dialogue/sessions/<session_id>/reply/` | `message` | `reply`, `dialogue_phase`, `stance_drift`, `history` | 同步 AI 回合（WS 的 HTTP 後備）|

## M4 — Dialogue Room

| API | Method | URL | Request | Response | 功能 |
|---|---|---|---|---|---|
| Room messages | GET, POST | `/api/matching/rooms/<room_id>/messages/` | POST: `content` | `messages[]`, 對方資訊, `stance_drift` | 載入 / 送出配對室訊息 |
| Leave room | POST | `/api/matching/rooms/<room_id>/leave/` | — | 配對狀態 payload | 關閉配對室 |
| Message reactions | GET, POST | `/api/message-reactions/` | GET: `target_type`, `conversation_id`; POST: `target_type`, `target_id`, `value`(1/-1/0) | `reactions[]` | 對對方訊息按讚/倒讚（0=取消）|
| Godot match room | POST | `/api/godot/match-rooms/` | `topic_id`, `user_ids`(2) | `room_id`, `redirect_url` | 大廳兩玩家就地配對（Service）|

## M5 — NLP / Semantic Tree & CCND

| API | Method | URL | Request | Response | 功能 |
|---|---|---|---|---|---|
| AI session tree | GET | `/api/dialogue/sessions/<session_id>/semantic-tree/` | — | 語意樹 payload | AI session 的概念樹 |
| Analyze AI tree | POST | `/api/dialogue/sessions/<session_id>/semantic-tree/analyze/` | — | 語意樹 payload | 對未分析的 AI 回合跑 NLP |
| Room tree | GET | `/api/matching/rooms/<room_id>/semantic-tree/` | — | 語意樹 payload | 配對室的概念樹 |
| Room tree timeline | GET | `/api/matching/rooms/<room_id>/semantic-tree/timeline/` | `as_of_message_id` | 時間軸快照 | 指定訊息時點的樹（CCND 門檻）|
| Analyze room tree | POST | `/api/matching/rooms/<room_id>/semantic-tree/analyze/` | — | 語意樹 payload | 對未分析訊息跑 NLP |
| History list | GET | `/api/history/conversations/` | `type` (query) | `results[]` | 過往 AI + 真人對話清單 |
| History detail | GET | `/api/history/conversations/<kind>/<id>/` | — | 摘要 + 訊息 + 語意樹 | 單一過往對話 |
| History timeline | GET | `/api/history/conversations/<kind>/<id>/semantic-tree/timeline/` | `as_of_message_id` | 時間軸快照 | 回放樹（CCND 門檻）|
| CCND insights | GET | `/api/history/conversations/<kind>/<id>/ccnd-insights/` | — | `summary`, `novelty`, `similarity`, `snapshots[]` | 個人概念擴張洞察（僅本人資料）|
| CCND snapshot analysis | GET | `/api/history/conversations/<kind>/<id>/semantic-tree/snapshot-analysis/` | `segments` | 各受試者 CCND 指標 | 研究用（Staff）|

## M6 — Summary / Knowledge Base

| API | Method | URL | Request | Response | 功能 |
|---|---|---|---|---|---|
| Viewpoint review list | GET | `/api/summary/viewpoints/` | `status`, `topic_id` | 待審觀點列 | 人工審核佇列（Researcher）|
| Review decision | POST | `/api/summary/viewpoints/<pk>/review/` | `action`(approve/reject), `notes` | 更新後觀點 | 核准 / 退回候選觀點（Researcher）|
| KB highlights | GET | `/api/summary/viewpoints/highlights/` | `topic_id`, `limit` | 精選觀點卡 | 首頁熱門觀點（已審核）|
| KB browse | GET | `/api/summary/viewpoints/browse/` | `topic_id`, `page` | 分頁觀點列 | 各議題完整觀點清單 |
| KB conversation detail | GET | `/api/summary/viewpoints/<pk>/conversation/` | — | 摘要 + 立場 + 品質分數 | 觀點背後的對話摘要 |
| Video recommendations | GET | `/api/summary/videos/` | `topic_id` | 影片清單 | 議題精選影片 |

## 研究工具（問卷 / 回饋）

| API | Method | URL | Request | Response | 功能 |
|---|---|---|---|---|---|
| Post questionnaire | POST | `/api/post-questionnaire/` | 後測量表 (C1–E)、`experiment_condition` | `s_pre`, `s_post`, `delta_s`, `stance_centrism` | 送出後測 + 計算去極化指標 |
| Consent update | PATCH | `/api/post-questionnaire/<id>/consent/` | `consent_confirmed` | 同意狀態 | 記錄 / 撤回匯報同意 |
| Platform feedback | POST | `/api/platform-feedback/` | UX 量表、`nps_score` | `mean_ux`, `nps_category` | 平台體驗回饋（Part F）|

## Godot 大廳 / 社交

| API | Method | URL | Request | Response | 功能 |
|---|---|---|---|---|---|
| My titles | GET, POST | `/api/titles/me/` | POST: `title_id`, `color` | `owned[]`, `selected_id` | 檢視 / 選用頭銜 |
| Issues | GET, POST | `/api/issues/` | POST: `title`, `body` | Issue 物件 | 社群議題板 |
| Issue reactions | GET, POST | `/api/issues/<id>/reactions/` | `emoji_index`(0–4) | `counts`, `mine` | 議題表情回應 |

---

## WebSocket Routes（Django Channels）

兩條路由皆以 `?token=<access_jwt>` query 認證；關閉碼：`4001` 未授權、`4004` 找不到 session/room、`4000` 房間已關。

| 路由 | URL pattern | Client → Server | Server → Client | 功能 |
|---|---|---|---|---|
| AI 對話串流 | `ws/dialogue/<session_id>/` | `user_message` | `agent_stream`（逐字）、`agent_stream_end`（stance_drift）、`error` | 逐字串流的 AI 對話 |
| 真人配對室 | `ws/matching/rooms/<room_id>/` | `match_message`、建議回應（accept/modify/ignore） | `match_message`、`match_ai_suggestion`、`match_system_prompt`、`match_stance_drift`、`error` | 即時真人對話 + NLP 介入 |

> `semantic-tree/timeline/` 與 `ccnd-insights/` 有 CCND「時間軸門檻」（未答對應問卷前回 403）；`snapshot-analysis/` 為研究者專用；`ccnd-insights/` 只回傳本人資料、移除對方資訊。
