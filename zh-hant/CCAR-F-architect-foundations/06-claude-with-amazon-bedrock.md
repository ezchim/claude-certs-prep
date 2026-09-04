---
title: 透過 Amazon Bedrock 使用 Claude — 備考筆記（主要來源）
source: https://academy.claude.com/courses/claude-with-amazon-bedrock
disclaimer: 用於備考的原創筆記——不是 Anthropic 或 AWS 官方教材。不是課堂逐字稿。
approx_length: 約 5500–7000 字
deepened: 2026-08-23
cross_check: AWS Bedrock 公開文件（Converse、InvokeModel、推論設定檔、IAM）
---

# 透過 Amazon Bedrock 使用 Claude — 主要備考筆記

> **免責聲明：** 這些是對齊 [Claude with Amazon Bedrock](https://academy.claude.com/courses/claude-with-amazon-bedrock) **公開** Claude Academy 課程大綱的**原創**備考筆記。它們**不是** Academy 課堂傾倒或課程原文。請搭配現行 AWS Bedrock 與 Anthropic 文件，核對現行模型 ID、區域與 API 欄位。完成官方課程仍是小考的權威依據。

**適合對象：** 要透過 Amazon Bedrock 把 Claude 功能加進應用程式的開發者。

**先修（課程）：** Python、JSON 基礎、已啟用 Bedrock 模型存取的 AWS 帳號。

**如何使用：** 平台專屬章節（啟用、ID、身分與存取管理（IAM）、Converse 對上 Invoke、落地）是相對於 Anthropic API 與 Vertex 路線、報酬率最高的差異點。共享的 Claude 技能（提示、工具、檢索增強生成（RAG）、模型上下文協定（MCP）、agent）仍然會出現——要認得它們在 Bedrock 請求形狀裡的樣子。

---

## 1. 課程地圖（公開模組）

公開頁面（約 65 堂課、多個小考）大致把主題分成：

1. **使用 API** — 驗證、基本請求、對話管理、系統提示詞、結構化輸出
2. **提示工程** — 策略、評測框架、系統化測試
3. **工具使用** — JSON Schema 工具、多輪工具迴圈、批次工具呼叫、內建工具
4. **RAG** — 切塊、向量嵌入（embedding）、BM25 混合搜尋、多索引、重排序、脈絡化檢索
5. **模型上下文協定（MCP）** — 工具、資源、提示；伺服器與用戶端
6. **Agent** — 建立在 Bedrock 存取之上的 Claude Code／電腦操作風格自動化模式
7. **進階 Claude 功能** — 延伸思考、視覺、提示詞快取、串流、temperature、結構化萃取

以認證考試的角度準備，優先學**存取模式、模型 ID、IAM、區域、Converse 對上 InvokeModel**，然後弄清楚當傳輸層是 Bedrock 時，共享的 Claude 技能是什麼樣子。

**情境題主軸：** 啟用模型 → 選區域／設定檔 → IAM 最小權限原則 → Converse（預設）或 Invoke（原生） → 歷史自己管理 → 視需要加入工具／RAG／評測 → 考慮落地的設定檔選擇。

---

## 2. Bedrock 上的存取模式

### 2.1 啟用與憑證

1. 在 Bedrock console（按帳號／區域）**請求／啟用**你需要的 Anthropic Claude 模型。
2. 用標準 AWS 憑證鏈（環境變數、共享設定、執行個體角色、SSO 等）呼叫 **Bedrock Runtime**。
3. 在你的區域建立 runtime client，例如 `boto3.client("bedrock-runtime", region_name="us-east-1")`。

經典的 Bedrock Runtime 呼叫，你**不會**送 Anthropic API 金鑰。計費與租戶掛在 AWS 上。（獨立產品如 AWS 上的 Claude Platform 可能不同——要看清楚題目點名的是哪個產品。）

**常見的啟用失敗：**

- 模型在 A 區域啟用了，client 指向 B 區域
- 模型啟用了，但在需要推論設定檔時呼叫原始基礎模型 ID
- 憑證對 AWS 整體有效，但角色在正確的 Amazon 資源名稱（ARN）上缺少 Bedrock invoke

### 2.2 模型 ID 與推論設定檔

Bedrock 用這類 ID 識別 Claude：

- 基礎模型式：`anthropic.claude-…`
- **跨區域推論設定檔**：通常帶地理前綴，例如 `us.anthropic.claude-…`、`eu.anthropic.…`、`global.anthropic.…`
- **應用程式推論設定檔**：客戶自建的設定檔（常用於成本歸屬或釘在單一區域）

**考試關鍵事實：**

- 不是每個模型在每個區域都存在。呼叫不存在的模型，會得到令人困惑的「不存在」／不支援隨需那一類錯誤。
- 較新的 Claude 模型經常要求**推論設定檔 ID**，而不只是原始基礎模型 ID。
- 推論設定檔會路由到模型所在的區域。確切 ID 要在 Bedrock console 的跨區域推論／模型詳細頁面上查。
- IAM 必須允許你實際呼叫的資源（兩者都用上時，基礎模型 ARN **以及**推論設定檔 ARN）。範例政策裡 ARN 的區域位置用萬用字元很常見——仍要堅持最小權限原則。

永遠在 AWS 模型卡片上確認目前的 ID——字串會隨新 Claude 版本推出而改變。

**決策樹——該傳哪個 ID：**

```text
該模型在隨需推論時是否需要推論設定檔？
 是 → 使用系統地理設定檔（us./eu./global.）或你的應用程式設定檔 ARN／ID
 否 → 基礎模型 ID 在該區域也許可用（仍要驗證）

需要單一區域落地？
 → 不要使用多區域地理設定檔；使用指向單一區域基礎模型 ARN 的應用程式設定檔
 （或 AWS 為該區域記載的其他區域內選項）

需要按團隊／應用程式做成本歸屬？
 → 帶標籤的應用程式推論設定檔
```
---

## 3. Converse API 對上 InvokeModel（高報酬率）

| | **Converse／ConverseStream** | **InvokeModel／InvokeModelWithResponseStream** |
| --- | --- | --- |
| 形狀 | 統一的 Bedrock 訊息 schema | 模型原生 JSON body |
| 可攜性 | 許多 Bedrock 聊天模型共用同一程式碼形狀 | 供應商專屬 payload |
| 多輪／工具 | 一等的 `messages`、`system`、`toolConfig` | 你自己組出 Anthropic Messages 風格的 body |
| Claude 額外項 | Converse 沒浮出來的旋鈕，用 `additionalModelRequestFields` | 完整的原生 Anthropic 請求欄位 |
| 何時用 | 聊天、agent、工具的預設 | 向量嵌入／影像生成、舊有用途，或只有原生才有的功能 |

**考試預設：** 在 Bedrock 上跑對話式 Claude，優先用 **Converse**。當你需要原生 Anthropic Messages payload（某些工具類型／欄位）或非聊天模態時，才降到 **InvokeModel**。

AWS 公開文件強調：Converse 為支援訊息的 Bedrock 模型提供一致的 API；獨特參數仍然透過模型專屬結構傳入。

### 3.1 Converse 心智模型

```text
client.converse(
 modelId=...,
 messages=[{role, content:[{text|...}]}],
 system=[...], # optional
 inferenceConfig={maxTokens, temperature, topP,...},
 toolConfig={...}, # optional
 additionalModelRequestFields={...} # Claude-specific extras
)
```

從類似 `response["output"]["message"]["content"][i]["text"]` 的路徑讀文字（確切索引取決於內容區塊）。

**無狀態：** Bedrock 不會替你記住先前的輪次。每次呼叫，你都必須重送完整的使用者／助理交替歷史。

### 3.2 InvokeModel 心智模型（Claude）

Body 通常包含：

- `anthropic_version`（Bedrock Claude Messages 的值，例如 `bedrock-2023-05-31`——確認現行文件）
- `max_tokens`（原生 Messages 風格裡必填）
- `messages` 陣列
- 可選的 `system`、`temperature`、tools 等

你要自己解析 Anthropic 形狀的回應。內容類型通常是 `application/json`；在 boto3 裡 body 是 bytes（`json.dumps(...).encode()`）。

### 3.3 串流

| 非串流 | 串流 |
| --- | --- |
| `converse` | `converse_stream` |
| `invoke_model` | `invoke_model_with_response_stream` |

與非串流同一 IAM 家族，再加上**串流專屬**動作。只授與 `bedrock:InvokeModel`，是非串流正常、串流卻以 AccessDenied 失敗的經典原因。

串流回應以事件序列送達，由你的程式碼重組（內容增量、訊息停止等）。串流改變的是遞送方式，不是對工具／歷史正確性的需求。

### 3.4 當 Converse 不夠用時

選 InvokeModel，當：

- 你需要向量嵌入或影像生成模型（不是聊天型 Converse）
- 某個 Claude 欄位在原生 Messages 形式裡比較容易，或只有那裡有文件
- 除錯需要看到確切的 Anthropic 形狀 body
- 遷移現有 Anthropic API 程式碼，想做最少對應（仍然要改驗證、版本欄位、端點）

選 Converse，當：

- 建立橫跨模型家族、可攜的 Bedrock 聊天／agent
- 你想要一等的 `toolConfig` 與 Guardrails 整合模式
- 團隊標準化在單一 Bedrock 對話 API 上

---

## 4. IAM 要點（最小權限原則）

Converse 與 Invoke 最終都用 Bedrock 推論動作授權：

- `bedrock:InvokeModel` — `InvokeModel` **以及** `Converse` 都需要
- `bedrock:InvokeModelWithResponseStream` — 串流變體需要

**出人意料的考試事實：** `Converse` 上的 `AccessDeniedException`，修復方法常常是授與 `bedrock:InvokeModel`，不是經典政策裡那個不存在的獨立 `bedrock:Converse` 權限。AWS 文件載明 Converse 需要 `bedrock:InvokeModel`；ConverseStream 需要 `bedrock:InvokeModelWithResponseStream`。

使用設定檔／在 console 選擇時，還要授與：

- `bedrock:GetInferenceProfile` — 以推論設定檔執行推論
- `bedrock:ListInferenceProfiles` — 在 console 裡選擇設定檔
- 對正確**模型／推論設定檔 ARN** 的存取
- 任何 Guardrails／知識庫動作，如果題目包含那些功能

**推論設定檔 IAM 陷阱：** 使用跨區域設定檔時，政策必須允許對以下項目 invoke：

1. 推論設定檔 ARN（入口）
2. 設定檔可能路由到的**每個目的地區域**裡，底層的基礎模型 ARN

只允許設定檔 ARN，是 Lambda 角色常見的失敗模式。

最小權限檢查清單：

- 盡可能按帳號與區域縮小範圍
- 列出特定模型家族 ARN，而不是 `*`
- 正式環境與實驗使用不同角色
- 拒絕你的應用程式不需要的危險相鄰動作（模型客製化、marketplace 購買等），除非真的需要
- 落地方面：在政策要求時，把設定檔選擇與 IAM 區域條件結合起來
---

## 5. 區域與資料落地

- 為 runtime client 選一個支援你存取模式的**來源區域**。
- 當你需要跨地理區的可用性，用**地理推論設定檔**（`us.`、`eu.`……）。
- 當被允許且落地有彈性時，用 **global** 設定檔。
- **單一區域落地**，遵循 AWS 目前指引：應用程式推論設定檔指向單一區域的基礎模型 ARN，或適用時的其他區域內選項。**不要**假設跨區域設定檔會把資料留在單一可用區／區域。

合規題：讓設定檔地理對上政策，勝過「教程裡碰巧可用的任何模型 ID」。

**落地決策樹：**

```text
落地有彈性、要極大化可用性？
 → global 推論設定檔（若模型支援）

必須大致待在美國或歐盟？
 → us. 或 eu. 地理設定檔

必須待在單一區域？
 → 釘在該區域基礎模型 ARN 的應用程式推論設定檔
 （＋你組織要求的 IAM 區域條件）

不確定模型是否可用？
 → 查 Bedrock console 模型卡／ListFoundationModels／設定檔文件
```

**Bedrock 上的 Claude Code 提醒（概念）：** 若一個 agentic 程式工具預設使用跨區域設定檔，單一區域落地可能需要建立應用程式設定檔，並把工具指向那個設定檔 ID——AWS 公開部落格把這點列為常見的企業陷阱。

---

## 6. 透過 Bedrock 使用 Claude 功能（課程主題）

### 6.1 對話與系統提示詞

與 Anthropic Messages API 同一套想法：角色交替；系統提示詞設定持久行為；保持歷史整潔。Bedrock Converse 使用像 `system` 清單與 `inferenceConfig` 這類欄位名稱。

**歷史衛生：** 修剪舊輪次，需要時做摘要，永遠不要在迴圈中途丟掉不成對的工具結果。

### 6.2 提示工程與評測

課程重點：結構化提示、系統化測試集、**基於模型的評分**與**基於程式碼的評分**。考試：評測是一個工作流程（案例 → 執行 → 評分 → 迭代），不是一次性的感覺檢查。

| 評分風格 | 它檢查什麼 | 強項 |
| --- | --- | --- |
| 基於程式碼 | 精確比對、正規表示式、JSON schema、單元斷言 | 確定性、便宜 |
| 基於模型 | 由另一次模型呼叫依評分量規打分 | 對開放答案有彈性 |
| 人工抽查 | 樣本審查 | 校準 |

### 6.3 工具使用

用 JSON Schema 定義工具；Claude 回傳工具呼叫；你的應用程式執行，並在下一輪回傳工具結果。支援多輪與批次工具呼叫模式。在 Converse 上，工具住在 `toolConfig` 下；只有原生才有的 Anthropic 工具風味，可能需要 InvokeModel／額外欄位。

**工具迴圈正確性：**

1. 模型回傳帶工具使用區塊的助理訊息
2. 應用程式跑工具
3. 應用程式以要求的角色／形狀附加工具結果
4. 帶完整歷史再次呼叫模型
5. 重複直到最終文字答案或停止條件

永遠不要捏造工具結果。永遠不要隨手重排 tool_use／tool_result 配對。

### 6.4 RAG

大綱裡的正式環境 RAG 主題：

- 切塊策略
- 向量嵌入（常常是經由 **Invoke**、而不是 Converse，呼叫另一個 Bedrock embedding 模型）
- 詞彙搜尋（**BM25**）＋向量混合
- 多索引架構、重排序
- **脈絡化檢索**（嵌入／索引前，用脈絡豐富切塊）

知道*為什麼*混合搜尋存在（精確關鍵字對上語意符合），勝過任何單一函式庫。

Bedrock 知識庫可能出現在以 AWS 為中心的題目——仍然對應同一套 RAG 概念：擷取、切塊、嵌入、檢索、生成、引用來源。

### 6.5 進階功能

- **延伸思考／推理** — 額外 token 與成本。現行模型使用**自適應思考**（模型決定何時／想多深；深度用 **effort 等級**調節）——固定的 `budget_tokens` 思考預算在 4.6 **已棄用**，在較新的前沿模型上**已移除（400 錯誤）**；使用任一種形狀前，核對 Bedrock 上該模型的模型卡片
- **視覺** — 影像內容區塊
- **提示詞快取** — 當模型／平台支援時，為穩定前綴降低成本／延遲
- **串流** — 長答案有更好的使用者體驗
- **結構化萃取** — 約束輸出（schema／仔細的提示）；temperature **只適用於較舊模型**——現行前沿模型已移除取樣參數，所以 schema 才是確定性槓桿

### 6.6 MCP 與 agent

MCP 跨用戶端／伺服器標準化工具、資源與提示。Agent 一節把以 Bedrock 為後端的 Claude 接到自動化模式（Claude Code、電腦操作）。備考：MCP 是**模組化暴露工具／資源的協定**，不是 Bedrock 專屬 API。

Agent 模式仍然適用：平行化、串接、路由，用追蹤除錯。
---

## 7. 比較快照：Bedrock 對上 Anthropic API 對上 Vertex

| 主題 | Anthropic API | Amazon Bedrock | Google Vertex（Claude） |
| --- | --- | --- | --- |
| 驗證 | Anthropic API 金鑰 | AWS IAM／憑證 | GCP ADC／服務帳戶 |
| Client | `Anthropic` | `bedrock-runtime`（Converse／Invoke） | `AnthropicVertex` |
| 模型 ID 風格 | Anthropic ID | `anthropic.…`／`us.anthropic.…` | Vertex 發布者 ID（例如 `claude-sonnet-4-6`） |
| 版本欄位 | 標頭／SDK | Invoke body 裡的 `anthropic_version`；Converse 抽象掉很多 | 原始 HTTP 上的 `anthropic_version: vertex-2023-10-16` |
| 統一多模型聊天 | Messages | **Converse**（Bedrock 全域） | 經由 Vertex 後端的 Messages |
| 計費／合約 | Anthropic | AWS | Google Cloud |
| 落地旋鈕 | Anthropic 政策 | 區域＋推論設定檔 | global／us／eu／區域端點 |
| 啟用 | API 金鑰存取 | 按帳號／區域的 Bedrock 模型存取 | Model Garden 啟用 |

**選 Bedrock，當：** 以 AWS 為中心的 IAM、跨 Bedrock 模型的 Converse 可攜性、AWS 合規邊界、既有的 AWS 網路／VPC 模式。

**選 Vertex，當：** GCP 是控制平面、Google 計費／落地、Model Garden 合作夥伴路徑。

**選 Anthropic API，當：** 通往最新 Anthropic 專屬功能的最快路徑，以及最簡單的金鑰驗證。

**不要跨雲端混用 ID 格式**——光這一點就能回答好幾道考試干擾選項。

---

## 8. 考試陷阱（Bedrock 專屬）

| 陷阱 | 現實 |
| --- | --- |
| 找 `bedrock:Converse` 權限 | Converse 用 `bedrock:InvokeModel` |
| 需要設定檔時用基礎模型 ID | 改成推論設定檔 ID／ARN |
| 只授與設定檔 ARN | 地理設定檔還要授與目的地 FM ARN |
| 假設歷史被儲存 | 無狀態；重送 messages |
| 用 Converse 做向量嵌入 | 分開 Invoke embedding 模型 |
| 單一區域政策卻用跨區域設定檔 | 錯誤的落地工具；用應用程式設定檔釘住 |
| 把 Vertex 模型 ID 複製進 Bedrock | 不同的目錄字串 |
| 已授與 Invoke，串流卻 AccessDenied | 缺少 `InvokeModelWithResponseStream` |
| 「全域啟用一次」 | 啟用是按帳號／區域模式 |
| 題目要求 Guardrails 時把它當可選 | 被問及時，在 Converse 上接上 Guardrails 設定 |

---

## 9. 考試提示

- **同時**啟用模型存取，並使用模型存在的區域／設定檔。
- 除非題目強迫用原生 Invoke，否則優先 **Converse**。
- 記住**無狀態**歷史管理。
- Converse 的 IAM ≈ `bedrock:InvokeModel`（＋串流變體）。
- 較新模型 → 查**推論設定檔** ID。
- RAG 答案應提到切塊＋檢索品質，而不是正式環境「就把 PDF 塞進提示詞」。
- 評測 = 系統化評分，不是單一軼事提示。
- 區分建議式提示與工具／MCP 管線。
- 落地方面，明確說出設定檔策略。
- 工具迴圈方面，堅持正確的訊息順序。

---

## 10. 最小練習草圖（概念）

1. 在你的帳號／區域，於 Bedrock console 啟用 Claude。
2. 用 AWS 憑證建立 runtime client。
3. 用推論設定檔模型 ID 與一則使用者訊息呼叫 `converse`。
4. 附加先前的助理輸出，加上第二輪。
5. 經由 `toolConfig` 加一個小工具（例如計算機），完成一次工具迴圈。
6. 用 `converse_stream` 串流同一個提示詞。
7. 故意用一次錯誤的區域／模型 ID，認清失敗模式。
8. 草擬一份包含設定檔＋目的地 FM ARN 的 IAM 政策。
9. 寫五個評測案例，用程式碼斷言＋一份模型評分量規評分。
---

## 11. 自我檢核問答（附答案）

**Q1.** Converse 呼叫以 AccessDenied 失敗，但昨天同一個角色的 Invoke 是成功的。可能原因？
**A.** 缺少 `bedrock:InvokeModel`（或新模型／設定檔的資源 ARN）——Converse 使用同一 invoke 權限家族。若只有串流失敗，查串流動作。

**Q2.** 目錄裡的模型 ID 在文件上可用，在你的區域卻失敗。下一步？
**A.** 確認區域可用性；改成正確的**推論設定檔** ID 做跨區域路由。

**Q3.** 為什麼 Claude 在 Bedrock 上不記得上一輪？
**A.** Runtime 是無狀態的；你必須重送完整訊息歷史。

**Q4.** 什麼時候為 Claude 選 InvokeModel 而不是 Converse？
**A.** 需要 Converse 沒有乾淨暴露的原生 Anthropic 請求／回應欄位或功能；或非聊天模型（向量嵌入等）。

**Q5.** Converse 的 `inferenceConfig` 裡放什麼？
**A.** 共享旋鈕如 `maxTokens`、`temperature`、`topP`——Claude 專屬額外項常常放進 `additionalModelRequestFields`。

**Q6.** 說出兩個超越天真 top-k embedding 搜尋的 RAG 改善。
**A.** BM25 混合搜尋；重排序；脈絡化檢索；更好的切塊／多索引。

**Q7.** MCP 是什麼，一句話？
**A.** 一套協定，用來以模組化、可重用的方式，向 AI 用戶端暴露工具、資源與提示。

**Q8.** 如何保持多輪工具使用正確？
**A.** 依序附加助理工具呼叫訊息與使用者／工具結果訊息；永遠不要在迴圈中途丟掉結果。

**Q9.** 跨區域設定檔 invoke：IAM 只允許設定檔 ARN。仍然 AccessDenied。為什麼？
**A.** 還需要對設定檔路由到的每個目的地區域裡的基礎模型 ARN 有 invoke 權限。

**Q10.** ConverseStream 需要哪個 IAM 動作？
**A.** `bedrock:InvokeModelWithResponseStream`（不只 `InvokeModel`）。

**Q11.** 較新、需要設定檔的 Claude 模型，如何追求單一區域落地？
**A.** 建立指向單一區域基礎模型 ARN 的應用程式推論設定檔；避免多區域地理前綴。

**Q12.** 為什麼同一個應用程式裡，向量嵌入用 Invoke、聊天用 Converse？
**A.** 向量嵌入不是對話式 Converse 工作負載；Invoke 才是對的模態 API。

**Q13.** Bedrock Claude Invoke body 的版本欄位通常設成什麼？
**A.** `anthropic_version`，例如 `bedrock-2023-05-31`（確認現行文件）。

**Q14.** 基於程式碼對上基於模型的評測——何時優先基於程式碼？
**A.** 當成功可以用客觀方式檢查時（JSON schema、精確欄位、單元斷言）。

**Q15.** 說出與 Vertex Claude 的三個差異。
**A.** AWS IAM 對上 GCP ADC；`anthropic.`／地理設定檔 ID 對上發布者 ID；Converse 對上 Vertex 上的 Messages。

**Q16.** Converse 上的 Guardrails——概念角色？
**A.** 設定後，套用在模型呼叫周圍的平台安全／過濾——不是應用程式授權的替代品。

**Q17.** 批次工具呼叫在高層是什麼意思？
**A.** 模型可以在一輪請求多個工具；應用程式在下一次模型呼叫之前回傳多個結果。

**Q18.** 提示詞快取在什麼時候幫助最大？
**A.** 跨呼叫重用的穩定長前綴（系統、工具、大型文件）——當該模型／平台支援時。

**Q19.** 你在 us-east-1 啟用了 Claude，卻呼叫 eu-west-1。會怎樣？
**A.** 失敗或模型不可用——啟用與可用性對區域敏感；改區域或使用適當設定檔。

**Q20.** 脈絡化檢索加了什麼？
**A.** 在嵌入／索引前，用周圍文件脈絡豐富切塊，以改善檢索品質。

**Q21.** 先分類再送到專門提示／工具的 agent 模式？
**A.** 路由。

**Q22.** 為什麼產生工具引數時 temperature 通常較低？
**A.** 需要更確定性的結構化引數，才能可靠執行。

**Q23.** 應用程式推論設定檔的主要好處？
**A.** 成本歸屬／標籤，與／或釘住路由行為（包括單一區域來源），對上系統地理設定檔。

**Q24.** 經典 Bedrock Runtime 是用 Anthropic API 金鑰計費嗎？
**A.** 不是——經典 Bedrock Runtime 的 Claude 呼叫用 AWS 憑證與 AWS 計費。
---

## 12. 複習檢查清單（考前）

- [ ] 啟用對上憑證對上正確區域，三者都分得清
- [ ] 基礎模型 ID 對上地理設定檔對上應用程式設定檔
- [ ] 背下 Converse 對上 Invoke 的決策標準
- [ ] IAM：Converse 用 InvokeModel；串流用串流動作
- [ ] 設定檔 IAM 包含目的地 FM ARN
- [ ] 無狀態多輪歷史
- [ ] toolConfig 迴圈順序
- [ ] RAG 混合／重排序／脈絡化檢索詞彙
- [ ] 評測 = 系統化案例＋程式碼／模型評分
- [ ] 落地決策樹（global／地理／單一區域釘住）
- [ ] Bedrock 對上 Vertex 對上 Anthropic API 比較表
- [ ] 串流事件重組的概念理解

---

## 13. 詞彙表

- **Bedrock Runtime** — 呼叫基礎模型的 API 面（`bedrock-runtime` client）。
- **Converse** — 跨 Bedrock 聊天模型的統一對話推論 API。
- **ConverseStream** — Converse 的串流變體。
- **InvokeModel** — 模型原生 body 的呼叫 API。
- **推論設定檔** — 路由推論的資源（系統地理或應用程式）。
- **基礎模型（FM）ARN** — 某個區域裡底層的模型資源。
- **應用程式推論設定檔** — 客戶自建、用於歸屬與／或釘住路由的設定檔。
- **地理設定檔前綴** — 例如設定檔 ID 上的 `us.`、`eu.`、`global.`。
- **additionalModelRequestFields** — Converse 上模型專屬參數的逃生口。
- **inferenceConfig** — Converse 上共享的取樣／max token 旋鈕。
- **toolConfig** — Converse 的工具定義與工具選擇設定。
- **無狀態推論** — 呼叫之間沒有伺服器端聊天記憶。
- **BM25** — 混合 RAG 裡使用的詞彙排序函式。
- **脈絡化檢索** — 嵌入／索引前把脈絡加在切塊前面。
- **MCP** — 工具／資源／提示的模型上下文協定。
- **Guardrails** — 可設定在呼叫周圍的 Bedrock 安全過濾。
- **知識庫（Knowledge Base）** — AWS Bedrock 生態系裡受管的 RAG 檢索元件。
- **基於模型的評分** — 用模型依評分量規為輸出打分。
- **基於程式碼的評分** — 對輸出做程式化斷言。
- **AccessDeniedException** — Bedrock API 的 IAM 授權失敗。

---

## 14. 更深的決策樹

### 14.1 API 形狀選擇器

```text
工作負載是帶選配工具的對話式訊息嗎？
 是 → Converse / ConverseStream
 否 → 是向量嵌入／影像／僅原生才有的 payload 嗎？
 是 → InvokeModel / 串流變體
 否 → 重新查閱模態文件
```

### 14.2 失敗分診

```text
AccessDenied？
 → 檢查動作（Invoke 對上 InvokeWithResponseStream）
 → 檢查資源 ARN（設定檔 + 目的地基礎模型）
 → 檢查帳號的模型啟用

驗證錯誤／模型不存在／不支援隨需？
 → 區域錯誤，或需要推論設定檔 ID

奇怪的空答案／部分答案？
 → maxTokens 太低；停止原因；工具迴圈未完成

工具迴圈空轉？
 → 工具結果缺少／格式錯誤；max 輪次無上限
```

### 14.3 正式環境就緒迷你評分量規

1. 落地與可用性用對設定檔
2. 用拒絕案例測過的最小權限 IAM
3. 串流與非串流的逾時／重試
4. 提示微調前，有固定案例的評測套件
5. 日誌不洩漏密鑰
6. 對 token 與設定檔標籤的成本監控
7. 文件化的後援模型或區域策略

---

## 15. 共享技能主題對應到 Bedrock 欄位

| Claude 技能主題 | Bedrock 表達 |
| --- | --- |
| 系統提示詞 | Converse 上的 `system` |
| Temperature | `inferenceConfig.temperature` |
| 工具 | `toolConfig`（或 Invoke 上的原生工具） |
| 多輪 | 應用程式管理的 `messages` 陣列 |
| 串流 | `converse_stream`／invoke 串流 |
| 結構化輸出 | 仔細的提示＋schema 驗證；平台有提供時用平台功能 |
| 視覺 | messages 裡的影像內容區塊 |
| 快取 | 可用時的模型／平台專屬快取欄位 |
| Agent | 你的編排＋可選的、以 Bedrock 為後端的 Claude Code |

---

## 16. 讀書節奏

第 1 天：第 1–5 節（存取、Converse／Invoke、IAM、落地）。憑記憶畫比較表。
第 2 天：第 6–10 節（功能、陷阱、練習草圖）。大聲走失敗分診。
第 3 天：全部問答閉書；檢查清單；只掃詞彙表缺口。

**口訣：** 啟用、設定檔、IAM、預設 Converse、歷史是你的、落地是故意的。

---

*對齊 https://academy.claude.com/courses/claude-with-amazon-bedrock。正式使用前，在 AWS Bedrock 文件核對現行模型 ID、區域與 IAM 例子。*
---

## 17. 走完的迷你情境（考試風格）

**情境 1 — 正式環境 Lambda 裡的新 Sonnet**
團隊把舊的基礎模型 ID 複製進 `converse`。錯誤說不支援隨需吞吐量／請使用推論設定檔。
**答題路徑：** 查該 Claude 版本的系統或應用程式推論設定檔 ID；更新 `modelId`；把 IAM 擴到設定檔＋目的地 FM ARN；在 Lambda 區域重測。

**情境 2 — 歐盟資料政策**
法務要求在歐盟處理。工程師用 `global.…` 設定檔，因為教程裡「剛好能用」。
**答題路徑：** 錯。歐盟多區域落地優先用 `eu.` 地理設定檔；若政策要求單一區域，用單一區域的歐盟應用程式設定檔。寫清楚你實際擁有哪一種保證。

**情境 3 — 串流聊天小工具**
非串流聊天可用；串流以 AccessDenied 失敗。
**答題路徑：** 在相同資源上加上 `bedrock:InvokeModelWithResponseStream`；繼續在用戶端重組串流事件。

**情境 4 — 使用工具的客服 agent**
模型要兩個工具；應用程式只回一個結果；下一輪角色混亂。
**答題路徑：** 正確配對回傳所有工具結果；保住助理的工具使用訊息；然後繼續。

**情境 5 — RAG「幻覺出來的政策」**
天真的 top-k embedding 檢索漏掉精確條款識別碼。
**答題路徑：** 加上 BM25／混合，考慮重排序，改善切塊，試脈絡化檢索；用含那些條款識別碼的固定案例評測。

**情境 6 — 多雲遷移**
從 Anthropic API 搬到 Bedrock。工程師把 `x-api-key` 與 Anthropic base URL 貼進 AWS 程式碼。
**答題路徑：** 用 AWS 憑證＋Bedrock Runtime；把 Messages 欄位對應到 Converse，或用 Bedrock 的 `anthropic_version` 保住 Invoke；把模型 ID 改成 Bedrock 目錄形式。

**情境 7 — 成本歸屬**
財務看不出哪條產品線花了 Bedrock Claude token。
**答題路徑：** 按產品加上標籤的應用程式推論設定檔；經由那些設定檔 ARN 呼叫；按標籤做儀表板。

**情境 8 — 評測不一致**
提示詞在三個精挑細選的例子上看起來很好，在正式環境卻默默失敗。
**答題路徑：** 建立留出評測集；把 schema 的基於程式碼檢查，與品質的基於模型評分量規結合；用評測門檻把關部署。

---

## 18. 速查卡

| 需要 | Bedrock 動作 |
| --- | --- |
| 預設聊天／agent | Converse |
| 串流 token | ConverseStream＋串流 IAM |
| 原生 Anthropic body／向量嵌入 | InvokeModel |
| 跨區域可用性 | `us.`／`eu.`／`global.` 設定檔 |
| 單一區域釘住 | 應用程式設定檔 → 一個 FM ARN |
| 驗證 | AWS 憑證鏈，不是 Anthropic 金鑰 |
| 工具 | `toolConfig`（＋迴圈） |
| 只有 Claude 才有的旋鈕 | `additionalModelRequestFields` |
| 對上 Vertex 的可攜 | ID 與驗證完全不同 |

**一句話口訣：** 在 Bedrock 上，啟用模型，用 Invoke 家族 IAM 呼叫正確設定檔，優先 Converse，自己掌管歷史，並且故意挑選落地。

---

## 19. 額外問答（延伸）

**Q25.** 同一份 boto3 Converse 程式碼能給 Claude 和另一個 Bedrock 聊天模型用嗎？
**A.** 對訊息形狀常常可以——那份可攜性就是 Converse 的賣點——但模型專屬欄位與能力仍然不同。

**Q26.** 原生 Invoke Messages 風格的 Claude 漏掉 `max_tokens` 會怎樣？
**A.** 請求驗證錯誤——原生 Messages 風格 body 裡 `max_tokens` 是必填。

**Q27.** 系統提示詞在 Converse 上放哪裡，對上誤把 `role: system` 放進 messages？
**A.** 用 Converse 的 `system` 參數（系統區塊清單）；不要像某些其他 API 那樣，在 messages 裡發明一個 system 角色。

**Q28.** 為什麼有些 IAM 政策列出 `GetInferenceProfile`？
**A.** 依 AWS 推論先決條件文件，以推論設定檔執行推論時需要它。

**Q29.** 混合搜尋，一句話？
**A.** 結合詞彙（BM25）與向量檢索，讓精確 token 與語意符合都能浮出來。

**Q30.** 這門課裡的電腦操作／Claude Code 主題——該記住什麼？
**A.** 它們是可以跑在以 Bedrock 為後端的 Claude 上的 agentic 自動化模式；仍然需要 AWS 驗證、模型存取，以及驗證紀律。

---

## 20. 讀書收束

若你能不看筆記重畫（1）Converse 對上 Invoke、（2）串流／非串流的 IAM 動作、（3）落地用的設定檔類型、（4）三雲比較表，你就準備好應付 Bedrock 差異化考題了。然後用與 Anthropic API、Vertex 路線相同的詞彙，刷新共享的 Claude 技能（工具、RAG、MCP、評測）——改變的只有傳輸層與 ID 格式。
