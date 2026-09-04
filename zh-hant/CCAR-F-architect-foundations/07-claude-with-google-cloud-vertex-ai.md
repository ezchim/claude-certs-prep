---
title: 透過 Google Cloud 的 Vertex AI 使用 Claude — 備考筆記（主要來源）
source: https://academy.claude.com/courses/claude-with-google-cloud-s-vertex-ai
disclaimer: 用於備考的原創筆記——不是 Anthropic 或 Google Cloud 官方教材。不是課堂逐字稿。
approx_length: 約 5500–7000 字
deepened: 2026-08-23
cross_check: Google Cloud Vertex AI／Model Garden Claude 公開文件（ADC、端點、模型 ID）
---

# 透過 Google Cloud 的 Vertex AI 使用 Claude — 主要備考筆記

> **免責聲明：** 這些是對齊 [Claude with Google Cloud's Vertex AI](https://academy.claude.com/courses/claude-with-google-cloud-s-vertex-ai) **公開** Claude Academy 課程的**原創**備考筆記。它們**不是** Academy 課堂傾倒或課程原文。請在 Google Cloud 與 Anthropic 文件確認現行模型 ID、端點與功能矩陣。完成官方課程仍是小考的真相來源。

**適合對象：** 要透過 Google Cloud Vertex AI／Agent Platform 加入 Claude 功能的開發者。

**先修（課程）：** Python、JSON 基礎、有 Vertex AI 存取權的 Google Cloud 專案。

**如何使用：** 平台專屬材料（Model Garden 啟用、應用程式預設憑證（ADC）、global／us／eu／區域端點、發布者模型 ID、功能對等注意事項）是報酬率最高的差異點。共享的 Claude 技能與 Bedrock 路線重疊——要認得它們在 Vertex 的 Messages 形狀 client 裡的樣子。

---

## 1. 課程地圖（公開模組）

公開頁面（約 66 堂課、多個小考）大致組織成：

1. **用 API 存取 Claude** — 驗證、請求、多輪、系統提示詞、temperature、串流、結構化輸出
2. **提示工程技巧** — 策略、評測、自動化評分
3. **Claude 的工具使用** — 函式呼叫、多輪工具、批次工具、內建工具
4. **RAG** — 切塊、向量嵌入、BM25 混合、多索引、重排序、脈絡化檢索
5. **模型上下文協定** — 工具、資源、提示；伺服器／用戶端生命週期
6. **Agent 與工作流程** — 平行化、串接、路由、除錯
7. 相關主題：視覺、PDF、引用來源、提示詞快取、Claude Code／電腦操作風格應用

共享的 Claude 技能與 Bedrock 課程大量重疊。對考試來說**真正不同的**是 **GCP 設定、驗證、區域／端點、模型 ID 格式，以及對上 Anthropic 與 Bedrock 的 API 差異**。

**主軸：** 在 Model Garden 啟用 → ADC／gcloud → 挑端點類別（global／us／eu／區域） → `AnthropicVertex` Messages 呼叫 → 自己掌管歷史 → 工具／RAG／評測 → 知道對上 Anthropic API 的對等落差。

---

## 2. 設定與身分驗證

### 2.1 在 Model Garden 啟用 Claude

1. 在 Google Cloud Console 開啟 Vertex AI／Model Garden（選對專案）。
2. 搜尋 **Anthropic**／Claude。
3. 為專案**啟用**你需要的模型（若尚未啟用）。
4. 確認該模型在你打算使用的位置／端點風格下可用。

跳過啟用是經典的「文件上能用、我的專案裡失敗」。

### 2.2 gcloud 與應用程式預設憑證（ADC）

典型本機設定：

```bash
gcloud init
gcloud auth login
gcloud config set project YOUR_PROJECT_ID
gcloud auth application-default login
```

Anthropic **Vertex** SDK 使用 Google 驗證（ADC／服務帳戶／工作負載身分）。你**不會**為 Vertex 後端的呼叫傳入 Anthropic API 金鑰。

生產環境：優先使用帶最小權限 Vertex AI 權限的**服務帳戶**，而不是在伺服器上用長效的使用者 ADC。在 GKE／Cloud Run 上盡可能使用工作負載身分。

### 2.3 安裝 SDK

Python 例子：`pip install -U "anthropic[vertex]"`。
TypeScript：`@anthropic-ai/vertex-sdk`。

Client 模式：

```python
from anthropic import AnthropicVertex

client = AnthropicVertex(project_id="YOUR_PROJECT_ID", region="global")
message = client.messages.create(
 model="claude-sonnet-4-6", # example — check current IDs
 max_tokens=1024,
 messages=[{"role": "user", "content": "Hello"}],
)
```

常見涉及的環境變數：專案 id 與區域（`ANTHROPIC_VERTEX_PROJECT_ID`、`CLOUD_ML_REGION` 等，依 SDK 文件為準）。

**驗證決策樹：**

```text
在本機筆電做原型？
 → gcloud auth application-default login（使用者 ADC）
Cloud Run／GKE／GCE 正式環境？
 → 服務帳戶 + 工作負載身分／附加的服務帳戶
來自 GitHub 的 CI、沒有長效金鑰？
 → 工作負載身分聯合的模式
還想為 Vertex 設定 ANTHROPIC_API_KEY？
 → 錯誤的產品路徑——那是 Anthropic API，不是 Vertex
```
---

## 3. Vertex 上的 Claude API 與 Anthropic API 差在哪裡

**Messages** 形狀幾乎一樣，但平台差異很重要：

| 主題 | Anthropic API | Vertex／Agent Platform 上的 Claude |
| --- | --- | --- |
| 驗證 | `x-api-key`／Anthropic 金鑰 | Google Cloud 憑證 |
| 模型位置 | 在請求 body 的 `model` | 原始 HTTP 常常在 **URL 路徑**；SDK 仍接受 `model=` |
| 版本標記 | API 版本標頭 | 原始 HTTP 上 body 欄位 `anthropic_version` = `vertex-2023-10-16` |
| 端點 | `api.anthropic.com` | `…-aiplatform.googleapis.com` 或多區域／global 主機 |
| 計費／條款 | Anthropic | Google Cloud 合作夥伴模型條款 |
| 某些平台功能 | Batches、Files、某些伺服器工具等 | 查「支援／不支援」清單——它們不同 |

**原始 HTTP 提醒：** 模型放在發布者路徑
`/v1/projects/{PROJECT}/locations/{LOCATION}/publishers/anthropic/models/{MODEL_ID}:rawPredict`
（適用時串流用 streamPredict）
而 JSON body 包含 `anthropic_version`、`messages`、`max_tokens`、……

**SDK 提醒：** `AnthropicVertex` 讓你繼續用熟悉的欄位寫 `messages.create(...)`，由它處理 Vertex 路由。

**對等注意事項口訣：** 同一個 Claude 家族，功能面不一定相同。若題目說「在 Vertex 上」，不要假設每一個 Anthropic 專屬的管理／files／batches／伺服器工具功能都存在。

---

## 4. 區域與端點（高價值考試主題）

Agent Platform／Vertex 提供三種端點風格（公開 Google／Anthropic 文件）：

### 4.1 Global

- `region="global"`
- 在有容量的受支援區域之間，以可用性為目標動態路由
- 通常是**標準定價**（沒有落地溢價）
- 當資料落地有彈性時，最好的預設
- 隨用隨付導向；佈建吞吐量需要區域容量

概念上使用 global 的 AI Platform 端點（現行主機字串以文件為準）。

### 4.2 多區域（`us` 或 `eu`）

- 在該地理範圍內路由
- 常被引用的範例主機：`aiplatform.us.rep.googleapis.com`／`aiplatform.eu.rep.googleapis.com`
- **地理**粒度的資料落地，可用性又比單一區域好
- 相對於 global 通常有**定價溢價**（常被引用的數字是，4.5 世代起的新模型約 10%——以定價頁為準）

### 4.3 特定區域（例如 `us-east5`、`europe-west1`）

- 嚴格的單一區域路由
- 硬性落地、某些企業控制、**佈建吞吐量**需要它
- 相對於 global 通常也有溢價
- 模型可用性可能落後；最新模型可能先出現在 global／關鍵區域

**考試決策流程：**

```text
1. 落地有彈性 → global
2. 必須大致待在美國或歐盟 → us / eu
3. 必須釘住單一區域或使用佈建吞吐量 → 特定區域
4. 永遠核實你選的 Claude 版本在該位置已啟用且可用
```

**考試陷阱：** 合規題要求歐盟落地時用了 `global`——可用性不等於落地。

**考試陷阱：** 以為 global 有佈建吞吐量——公開資料強調區域端點上的佈建容量。

---

## 5. Vertex 上的模型 ID

Vertex 發布者 ID 看起來像 Anthropic 名稱，但遵循 **Google 的目錄字串**，例如（示意——永遠查現行文件）：

- `claude-sonnet-4-6`
- `claude-haiku-4-5@20251001`（有些 ID 含日期後綴）
- `claude-opus-4-6`

它們**不是** Bedrock 的 `anthropic.claude-…` 或 `us.anthropic.…` 推論設定檔字串，也可能和 Anthropic API 別名略有不同。

合作夥伴平台上的生命週期（已棄用／退役）日期，可能和 Anthropic 自家時程不同——考試答案應說「查 Google 的 Claude 模型頁面」，不要捏造日期。

**ID 比較速記：**

| 平台 | 範例風味 |
| --- | --- |
| Anthropic API | Anthropic 模型別名／帶版本的 ID |
| Bedrock | `anthropic.claude-…` 或 `us.anthropic.claude-…` |
| Vertex | `claude-sonnet-4-6` 發布者 ID |

跨雲端複製貼上是故意設計的干擾選項。
---

## 6. 功能支援（可以期待什麼）

Vertex Claude 上通常可用（按模型確認）：

- Messages API（聊天）
- 工具使用（包括幾種 Anthropic 工具類型，但有注意事項）
- 提示詞快取、有提供時的思考／延伸推理
- 視覺／文件（留意**酬載大小上限**——Vertex 可能把請求大小限制在例如幾十 MB）
- 有文件記載時的引用來源、結構化輸出
- 經由 SDK／串流 API 的串流

常常**不是**完整 Anthropic API 功能面——公開文件陸續點名過某些 Files／URL 輸入輔助、一些伺服器端工具、Message Batches、Admin API，以及一些受管 agent 基礎設施。若題目說「在 Vertex 上」，不要假設每一個 Anthropic 專屬功能都存在。

上下文視窗：較新的 Claude 世代可能提供很大的視窗（例如列出模型上的 1M 等級）；較舊的可能較小。優先「查模型卡」，而不是背下每個數字。

**對等決策規則：**

```text
產品需要某個功能？
 → 查 Vertex／Agent Platform 對該模型的支援矩陣
 → 若缺少，重新設計、改用 Anthropic API（若政策允許），或等待功能對等
絕不要未經查核就回答：『Claude 在 Anthropic API 上能做 X，所以 Vertex 也可以』
```

---

## 7. 課程技能主題（與 Bedrock 路線共享）

### 7.1 多輪與系統提示詞

使用者／助理交替；保持歷史連貫；把持久行為放進 `system`。HTTP 呼叫之間無狀態——你的應用程式存歷史。

### 7.2 提示工程與評測

XML 風格結構、例子、輸出控制；建立**測試集**，用基於模型與基於程式碼的評分者評分。生產品質 = 迭代迴圈，不是單一聰明的提示詞。

### 7.3 工具使用

JSON Schema 工具 → 模型工具呼叫 → 你的執行 → 工具結果回到 messages。多輪與批次模式對 agent 很重要。

### 7.4 RAG

切塊 → 嵌入 → 檢索（向量＋**BM25** 混合）→ 可選重排序 → 脈絡化檢索技巧 → 有提供時帶引用來源作答。概念管線與其他 Claude 雲端課程相同；儲存／嵌入可能使用 GCP 服務（Vertex AI embeddings、Vector Search 等）。

### 7.5 MCP

定義工具、資源、提示模板；跑 MCP 伺服器／用戶端；整合進應用程式。協定知識可轉移到 Anthropic API、Bedrock 與 Vertex 傳輸層。

### 7.6 Agent 與工作流程

要背的模式：

| 模式 | 想法 | 何時用 |
| --- | --- | --- |
| 平行化 | 把獨立子任務扇出 | 極度可平行的工作 |
| 串接 | 第 N 步的輸出餵給第 N＋1 步 | 有相依的管線 |
| 路由 | 先分類再送到專門提示／工具 | 混合意圖流量 |
| 除錯 | 記錄追蹤、約束工具、評測中間步驟 | 不穩定的 agent |

---

## 8. Vertex 對上 Bedrock 對上 Anthropic API（比較表）

| 維度 | Anthropic API | Bedrock | Vertex AI Claude |
| --- | --- | --- | --- |
| 雲端身分 | Anthropic 帳號 | AWS IAM | GCP IAM／ADC |
| 主要 SDK | `Anthropic` | AWS SDK＋Converse | `AnthropicVertex` |
| 模型 ID 風味 | Anthropic ID | `anthropic.`／地理設定檔 | 發布者 ID（`claude-…`） |
| 統一多模型聊天 API | Messages | **Converse**（Bedrock 全域） | 經由 Vertex 後端的 Messages |
| 版本標記 | Anthropic 標頭 | Invoke 上 `bedrock-2023-05-31` 風格 | 原始 HTTP 上 `vertex-2023-10-16` |
| 落地控制 | Anthropic 選項 | 區域＋推論設定檔 | global／us／eu／區域 |
| 企業包裝 | Anthropic 合約 | AWS Marketplace／Bedrock | Google Cloud／Model Garden |
| 啟用 | API 存取 | Bedrock 模型存取 | Model Garden 啟用 |

**選 Vertex，當：** GCP 已經是控制平面、你需要 Google 計費／落地，或組織政策強制 Vertex。
**選 Bedrock，當：** 以 AWS 為中心的 IAM、跨 Bedrock 模型的 Converse 可攜性、AWS 合規邊界。
**選 Anthropic API，當：** 通往最新 Anthropic 專屬功能的最快路徑，以及最簡單的金鑰驗證。

---

## 9. 考試陷阱（Vertex 專屬）

| 陷阱 | 現實 |
| --- | --- |
| 對 AnthropicVertex 使用 Anthropic API 金鑰 | 錯誤驗證——需要 GCP ADC／服務帳戶 |
| 貼上 Bedrock 模型 ID | 錯誤目錄 |
| 歐盟落地強制時選 global | 錯誤端點類別 |
| 以為 global 有佈建吞吐量 | 通常是區域的 |
| 以為完整 Anthropic 功能對等 | 查支援矩陣 |
| 忘記 Model Garden 啟用 | 需要專案層啟用 |
| 原始 HTTP 只把模型放在 body | 模型常常在 URL 路徑；body 需要 `anthropic_version` |
| 相信 Vertex 會存聊天歷史 | 無狀態；應用程式重送 messages |
| 忽略 PDF／影像的請求大小上限 | Vertex 可能強制酬載上限 |
| 用 `europe-west1` 跑只在 global 上的全新模型 | 區域落後 |
---

## 10. 考試提示

- 設定順序：啟用模型 → gcloud／ADC → `AnthropicVertex(project, region)`。
- 為原始 HTTP 背下 **`anthropic_version: vertex-2023-10-16`**。
- 區域選擇是**合規與可用性**問題，不是化妝。
- 不要把 Bedrock 模型 ID 貼進 Vertex，反之亦然。
- 無狀態多輪歷史仍然是你的工作。
- 對於「Vertex 上有功能 X 嗎？」——優先「若 Agent Platform／該模型有列出就支援」，不是 Anthropic API 假設。
- Agent 工作流程詞彙：平行化、串接、路由。
- RAG 答案應引用混合檢索與評測，而不只是向量嵌入。
- us／eu／區域相對 global 的定價溢價是已知的考試周邊事實——在定價頁確認現行數字。
- 生產環境的服務帳戶勝過伺服器上的使用者 ADC。

---

## 11. 最小練習草圖（概念）

1. 在 Model Garden 啟用一個 Claude 模型。
2. 設定 gcloud 專案＋ADC。
3. 用 `region="global"` 與一則短 Messages 請求呼叫 `AnthropicVertex`。
4. 用 `region="us"` 再做一次，注意端點／定價意涵。
5. 附加助理輸出，組出兩輪聊天。
6. 加一個自訂工具，完成一次工具迴圈。
7. 草擬一條小 RAG 路徑（切塊 → 檢索 → 作答）與一份含 5 個固定案例的評測評分量規。
8. 嘗試一次原始 HTTP `rawPredict`，看 `anthropic_version` 與路徑上的模型 ID。
9. 故意用一次 Bedrock 風格模型 ID，認清失敗模式。
10. 寫下你依賴哪些功能，並逐一對 Vertex 支援清單核實。

---

## 12. 自我檢核問答（附答案）

**Q1.** AnthropicVertex 的本機驗證通常怎麼做？
**A.** `gcloud auth application-default login` 之後的 Google 應用程式預設憑證（或生產環境的服務帳戶）——不是 Anthropic API 金鑰。

**Q2.** 原始請求 body 裡，除了／加上 Anthropic 標頭，放什麼？
**A.** `anthropic_version` 設成 `vertex-2023-10-16`；模型常常住在 URL 路徑。

**Q3.** 什麼時候選 `region="eu"` 而不是 `global`？
**A.** 當你需要歐盟地理資料落地，加上歐盟內部的多區域可用性，並接受可能的溢價定價。

**Q4.** 為什麼全新的 Claude 模型可能在 `europe-west1` 失敗，卻在 `global` 能用？
**A.** 區域可用性落後；最新模型常常先登陸 global／精選區域。

**Q5.** 說出一個對上 Bedrock 模型 ID 的差異。
**A.** Vertex 使用像 `claude-sonnet-4-6` 這樣的發布者 ID；Bedrock 使用 `anthropic.claude-…` 或 `us.anthropic.…` 設定檔。

**Q6.** Vertex 會在呼叫之間儲存對話歷史嗎？
**A.** 不會——你的應用程式必須每輪重送 messages（除非你自建工作階段儲存）。

**Q7.** 哪個 agent 模式把不同查詢類型送到不同工具／提示？
**A.** 路由。

**Q8.** 即使你部署在 Vertex 上，Converse 知識仍有幫助的兩個原因。
**A.** 共享的 Claude 概念（工具、系統提示詞、評測）；以及雲端考試的比較題常常對比 Bedrock 的 Converse 與 Vertex 在 GCP 上的 Messages 作法。

**Q9.** 什麼時候需要特定區域端點？
**A.** 硬性單一區域落地與／或佈建吞吐量（依公開指引）。

**Q10.** Model Garden 的角色是什麼？
**A.** 為你的 GCP 專案探索並啟用合作夥伴模型（包括 Claude）。

**Q11.** 為什麼生產環境優先服務帳戶？
**A.** 最小權限、可輪替、伺服器上沒有互動式使用者 ADC，適合工作負載身分。

**Q12.** Vertex 上的串流——會改變工具迴圈正確性規則嗎？
**A.** 不會——仍然要組出最終訊息，並保持 tool_use／tool_result 配對正確。

**Q13.** 說出兩個在 Vertex 上可能與 Anthropic API 不同的功能。
**A.** 例子：Message Batches、某些 Files／URL 輔助、一些伺服器工具、Admin API——永遠核對現行清單。

**Q14.** 混合 RAG 是什麼意思？
**A.** 生成前結合詞彙（BM25）與向量檢索。

**Q15.** 平行化對上串接？
**A.** 平行化把獨立工作扇出；串接把有相依的步驟排成序列。

**Q16.** 與美國多區域相關的主機風味是什麼？
**A.** 常常是 `aiplatform.us.rep.googleapis.com` 風格主機（確認文件）。

**Q17.** 可以不在專案裡啟用就在 Vertex 上用 Claude 嗎？
**A.** 不行——需要 Model Garden／專案存取的啟用。

**Q18.** `max_tokens` 放哪裡？
**A.** 在 Messages 請求 body（SDK 參數）——生成的必填上限。

**Q19.** 結構化萃取的 temperature？
**A.** 較舊模型（4.6 及更早）：較低以得到更確定性的輸出。現行前沿模型已完全移除取樣參數——確定性靠 schema／結構化輸出。

**Q20.** 假設 1M 上下文之前該查什麼？
**A.** Vertex 上該模型的模型卡——不是所有模型共享同一個視窗。

**Q21.** 工作負載身分的目的，一句話？
**A.** 讓雲端工作負載取得 GCP 憑證，而不用下載長效金鑰。

**Q22.** 引用來源功能——考試注意？
**A.** 在該模型／平台有文件記載時才可用；不要捏造引用來源支援。

**Q23.** 為什麼 PDF／影像請求可能在驗證有效時仍然失敗？
**A.** Vertex 路徑上的酬載大小上限，或不支援的媒體處理。

**Q24.** Bedrock `additionalModelRequestFields` 在 Vertex 上的類比？
**A.** 不是同一套 API——在 Vertex 上你通常傳 SDK／API 支援的 Anthropic Messages 欄位；額外項依 Anthropic／Vertex 文件，不是 Bedrock Converse 命名。
---

## 13. 複習檢查清單（考前）

- [ ] Model Garden 啟用步驟
- [ ] ADC 對上服務帳戶對上 Anthropic API 金鑰（哪個屬於哪裡）
- [ ] global 對上 us／eu 對上區域的決策樹
- [ ] 原始 HTTP：路徑上的模型 ID＋`anthropic_version: vertex-2023-10-16`
- [ ] 發布者 ID 格式對上 Bedrock ID
- [ ] 對上 Anthropic API 的功能對等注意事項
- [ ] 無狀態歷史＋工具迴圈順序
- [ ] Agent 模式：平行化、串接、路由
- [ ] RAG 混合／脈絡化檢索詞彙
- [ ] 評測 = 系統化程式碼＋模型評分
- [ ] 憑記憶畫三雲比較表
- [ ] 佈建吞吐量綁在區域端點

---

## 14. 詞彙表

- **Model Garden** — GCP 目錄，用來探索／啟用合作夥伴與 Google 模型。
- **Agent Platform** — Google Cloud 上 agentic／合作夥伴模型服務的介面（文件命名會演進）。
- **ADC** — Google 驗證用的應用程式預設憑證。
- **AnthropicVertex** — 官方 Anthropic SDK client，用於以 Vertex 為後端的 Claude。
- **發布者模型 ID** — 例如發布者 `anthropic` 下的 `claude-sonnet-4-6`。
- **rawPredict** — Vertex 原始預測端點，接受 Anthropic 形狀的 JSON body。
- **anthropic_version** — Body 欄位；Vertex 原始 HTTP 使用 `vertex-2023-10-16`。
- **Global 端點** — 以可用性為目標的動態多區域路由；落地有彈性。
- **多區域端點** — `us` 或 `eu` 地理路由，帶地理落地。
- **區域端點** — 單一區域路由；落地釘住；佈建吞吐量。
- **佈建吞吐量** — 預留容量（通常是區域的）。
- **工作負載身分** — 沒有靜態金鑰的雲端工作負載驗證。
- **功能對等** — Vertex 對某功能是否支援與 Anthropic API 相同的能力。
- **BM25** — 混合 RAG 裡的詞彙檢索元件。
- **脈絡化檢索** — 嵌入前用脈絡豐富切塊。
- **路由／串接／平行化** — 核心 agent 工作流程模式。
- **基於程式碼的評分** — 程式化評測斷言。
- **基於模型的評分** — 經由另一次模型呼叫依評分量規打分。

---

## 15. 走完的迷你情境

**情境 1 — 錯誤金鑰**
工程師設定 `ANTHROPIC_API_KEY`，納悶為什麼 AnthropicVertex 在有 VPC-SC 的 GCP 專案裡失敗。
**答案：** 用 GCP 憑證（ADC／服務帳戶）。Anthropic API 金鑰是不同的產品路徑。

**情境 2 — 合規**
政策：在歐盟處理。程式碼為了較便宜的 token 使用 `region="global"`。
**答案：** 改成 `eu` 多區域或特定歐盟區域端點；適用時接受溢價；核實模型可用性。

**情境 3 — 新模型落後**
Opus 在 global 上全新；區域端點 404／不可用。
**答案：** 用 global（若落地允許）或等待／在需要的區域啟用；不要發明 Bedrock 風格設定檔前綴。

**情境 4 — 原始 HTTP 除錯**
SDK 能用；手寫 curl 失敗。
**答案：** 檢查來自 gcloud 的 bearer token、URL 位置片段、發布者路徑上的模型 ID，以及 JSON 裡的 `anthropic_version`。

**情境 5 — 對等缺失**
應用程式依賴 Anthropic 專屬的批次 API。搬到 Vertex 就壞了。
**答案：** 查支援矩陣；用自己的佇列重新設計，或在政策允許時留在 Anthropic API。

**情境 6 — Agent 設計**
混合意圖：重設密碼對上訂單狀態對上常見問題。
**答案：** 路由模式送到專門提示／工具；評測每個分支。

**情境 7 — GCP 上的 RAG**
只靠向量的搜尋漏掉精確 SKU 字串。
**答案：** 混合 BM25＋向量；重排序；脈絡化檢索；含那些 SKU 的固定評測案例。

**情境 8 — 生產驗證加固**
把使用者 ADC JSON 提交進儲存庫給 Cloud Run。
**答案：** 停手；用附加的服務帳戶／工作負載身分；輪替已暴露的憑證。

---

## 16. 更深的端點比較表

| | Global | 多區域 us／eu | 特定區域 |
| --- | --- | --- | --- |
| 落地 | 有彈性／沒有保證 | 地理層級 | 單一區域 |
| 可用性 | 最高（動態） | 地理範圍內高 | 取決於單一區域 |
| 典型價格 | 基準 | 溢價（較新模型常約 10%） | 溢價 |
| 佈建吞吐量 | 通常沒有 | 查文件 | 有（典型所在） |
| 最適合 | 大多數沒有落地規則的應用 | 美國／歐盟合規＋高可用性 | 硬性釘住／預留容量 |

---

## 17. 讀書節奏

第 1 天：設定、驗證、端點、模型 ID。畫落地決策樹。
第 2 天：功能對等、共享技能、對上 Bedrock／Anthropic 的比較。
第 3 天：問答閉書＋情境＋檢查清單。

**口訣：** 在 Garden 啟用，用 Google 驗證，為落地挑端點，用發布者 ID，假設是 Messages 不是 Converse，答應功能前先核對對等。
---

## 18. 共享 Claude 技能對應到 Vertex 請求欄位

| 技能主題 | Vertex 表達 |
| --- | --- |
| 系統提示詞 | Messages 頂層的 `system` |
| Temperature | `temperature` 請求欄位 |
| 工具 | `tools`／工具選擇欄位（Anthropic 形狀） |
| 多輪 | 應用程式管理的 `messages` |
| 串流 | SDK 串流輔助／streamPredict |
| 結構化輸出 | Schema 提示＋驗證；若有列出則用平台功能 |
| 視覺／PDF | 內容區塊；留意大小上限 |
| 快取 | Vertex 模型支援時的提示詞快取欄位 |
| 思考 | 支援時的延伸思考參數 |
| Agent | 你的編排（平行／串接／路由）＋可選的 Claude Code |

對上 Bedrock 命名：Bedrock Converse 使用 `inferenceConfig`、`toolConfig`、`additionalModelRequestFields`。Vertex 經由 `AnthropicVertex` 更靠近 Anthropic Messages 名稱。

---

## 19. 失敗分診樹

```text
驗證錯誤／401／403？
 → ADC 存在嗎？專案對嗎？服務帳戶角色包含 Vertex AI User 權限嗎？
 → 此專案在 Model Garden 已啟用該模型嗎？

404／找不到模型？
 → 發布者 ID 字串錯了？
 → 模型在所選位置不可用？
 → 不小心用了 Bedrock 風格 ID？

空的／被截斷的輸出？
 → max_tokens 太低；停止原因；工具迴圈未完成

工具迴圈錯誤？
 → 缺少 tool_result；角色順序；schema 不匹配

落地稽核不通過？
 → 在地理強制要求下用了 global → 改到 us／eu／區域端點
```

---

## 20. 額外自我檢核問答

**Q25.** Vertex 路徑裡的 `locations/global` 是什麼意思？
**A.** 使用 global 端點位置，而不是像 `us-east5` 這樣的單一區域。

**Q26.** 為什麼財務可能在意 us／eu 端點？
**A.** 較新模型相對 global 的溢價定價——成本與合規都重要。

**Q27.** 串接例子，一句話？
**A.** 擷取實體 → 抓紀錄 → 起草答案，每一步餵給下一步。

**Q28.** MCP 是 Vertex 專屬的嗎？
**A.** 不是——MCP 與傳輸層無關；Vertex 是你可能與 Claude 一起託管或消費工具的地方之一。

**Q29.** 這門課裡的電腦操作／Claude Code——重點？
**A.** Agentic 應用可以跑在以 Vertex 為後端的 Claude 上；仍然需要 GCP 驗證、啟用與驗證紀律。

**Q30.** 沒有落地規則的新創，最好的第一個端點？
**A.** Global——可用性與基準定價，若合規出現再回頭看。

**Q31.** 如何把專案 id 傳給 SDK？
**A.** `AnthropicVertex(project_id=..., region=...)` 與／或文件化的環境變數。

**Q32.** 每輪只存最後一則使用者訊息有什麼問題？
**A.** 丟掉助理／工具歷史；破壞多輪連貫性與工具迴圈。

**Q33.** Vertex 應用的基於程式碼評分例子？
**A.** 在 CI 對輸出斷言 JSON schema、必填欄位，以及禁用的個人識別資訊（PII）模式。

**Q34.** 基於模型的評分例子？
**A.** 用固定裁判提示詞，依評分量規在 1–5 分上打幫助性／有據程度。

**Q35.** 為什麼 Vertex 課程的考試還要比較 Bedrock Converse？
**A.** 雲端選擇題測試你是否知道：即使 Claude 技能可轉移，驗證、ID 與 API 形狀仍然不同。
---

## 21. 正式環境就緒迷你評分量規（Vertex）

1. 在正確專案啟用模型；ID 對 Model Garden 卡片核實過
2. 端點類別匹配落地政策（global／us／eu／區域）
3. 生產使用服務帳戶或工作負載身分——不是筆電使用者 ADC
4. IAM 角色盡可能只給預測用的最小權限
5. 串流與非串流路徑的逾時／重試
6. 評測套件把關提示／模型變更
7. 日誌依要求遮蔽密鑰與敏感提示詞
8. token 用量成本警示；理解溢價端點定價
9. 區域容量失敗時文件化的後援（僅在政策允許時）
10. 功能相依對 Vertex 支援矩陣核過

---

## 22. 並排請求解剖（概念）

**Anthropic API：** 主機 `api.anthropic.com`＋API 金鑰標頭＋body `{model, messages, max_tokens,...}`

**Bedrock Converse：** 對 `bedrock-runtime` 的 AWS SigV4＋`{modelId, messages, inferenceConfig, toolConfig,...}`

**Vertex rawPredict：** 對 `…aiplatform…/publishers/anthropic/models/{MODEL}:rawPredict` 的 Google bearer token＋body `{anthropic_version, messages, max_tokens,...}`（模型在路徑）

若你能把同一個 hello-world 請求改寫成全部三種形狀，你就掌握了雲端比較題。

---

## 23. 團隊上手一頁紙（可貼進 wiki）

1. 為專案 P 在 Model Garden 啟用 Claude 模型 X／Y
2. 安裝 `anthropic[vertex]`；設定專案；除非合規另有規定，優先 `region=global`
3. 本機：`gcloud auth application-default login`
4. 生產：帶文件化角色的服務帳戶
5. 永遠不要提交憑證
6. 用模型卡上的發布者模型 ID——不要複製 Bedrock ID
7. 在我們的工作階段儲存服務裡自己掌管對話歷史
8. 工具必須為每一次 tool_use 回傳結果
9. 使用花俏的 Anthropic 專屬功能前，先查 Vertex 支援
10. 推行提示變更前跑共享評測包

---

## 24. 速查卡

| 需要 | Vertex 動作 |
| --- | --- |
| 驗證 | ADC／服務帳戶 |
| 啟用 | Model Garden |
| 預設位置 | 落地有彈性時 `global` |
| 美國／歐盟落地＋高可用性 | `us`／`eu` |
| 硬性釘住／佈建 | 特定區域 |
| SDK | `AnthropicVertex` |
| 原始版本欄位 | `vertex-2023-10-16` |
| 模型 ID | 發布者 `claude-…` |
| 歷史 | 應用程式管理 |
| 對上 Bedrock | Messages 不是 Converse；不同的 ID／驗證 |

**一句話口訣：** 在 Vertex 上，在 Garden 啟用，用 Google 驗證，為落地選端點，用發布者 ID 呼叫 Messages，答應 Anthropic API 能力前先核對功能對等。

---

## 25. 讀書收束

若你能不看筆記重畫（1）三種端點類別、（2）ADC 對上 API 金鑰、（3）發布者對上 Bedrock ID 例子、（4）三雲比較表，你就準備好應付 Vertex 差異化考題了。然後刷新共享的 Claude 技能（工具、RAG、MCP、agent、評測）——與 Bedrock／Anthropic 路線同一套想法，Vertex 的傳輸層與限制。

---

*對齊 https://academy.claude.com/courses/claude-with-google-cloud-s-vertex-ai。出貨前，在 Google Cloud 與 Anthropic 文件確認模型 ID、區域、定價溢價與功能矩陣。*

---

## 26. 最終閃卡操練

不看筆記大聲說：

1. 啟用 → ADC → 端點類別 → 發布者模型 ID → Messages 呼叫。
2. Global 對上 us／eu 對上區域，各一句話。
3. Vertex 與 Anthropic API 不同的三個原因（驗證、版本欄位／路徑、功能對等）。
4. Vertex 與 Bedrock 不同的三個原因（IAM 雲端、Converse 對上 Messages、ID 格式）。
5. 點名平行化、串接與路由，各附一句例子。

若任何操練失敗，只重讀該節的表格，然後再試。優先短閉書迴圈，而不是把整份檔從頭讀到尾。
