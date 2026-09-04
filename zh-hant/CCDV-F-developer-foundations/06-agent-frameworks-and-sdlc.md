---
title: Agent 框架、託管模式與 SDLC 基礎
pack: CCDV-F 開發者基礎考試
disclaimer: 原創備考筆記 — 獨立內容，非正式課程教材
approx_length: 約 2500–4000 字（P0 缺口補齊）
updated: 2026-08-23
---

# Agent 框架、託管模式與 SDLC 基礎

> **免責聲明：** 原創備考筆記，用來補上已公布 CCDV-F 考試藍圖的缺口：具名 agent 框架詞彙、自架對 Anthropic 託管式 agent、軟體工程／系統生命週期基礎，以及一段簡短的 WebSocket 說明。框架 API 與產品名稱會漂移——**考前請對照現行官方文件查證**。

**主要對應：** Agent 與工作流程 **14.7%**（模式／框架約 4.9%；建構／託管約 5.3%）· 應用與整合 **33.1%**（軟體工程基礎約 7.4%；系統生命週期約 2.8%；技術基礎／SDK 優先於 REST／串流傳輸層約佔 6.1% 中的一部分）。
**次要：** Claude Code（3.1%）作為 agent 主機 · 題幹出現託管與審查關卡時的評測／資安。
**配套章節：** [02](./02-production-prompting-agents-tools.md)（agent 迴圈、工具）· [03](./03-claude-code-mcp-integration.md)（API／串流／模型上下文協定（MCP））· [04](./04-production-engineering-evals-security.md)（持續整合（CI）關卡、審查、部署）。

---

## 1. 概覽 — 為什麼有這一章

第 02–04 章已經教過**模式**：工作流程對 agent、預算、驗證器、Messages API、Claude Code、MCP、評測、資安。公開考試指南的措辭還期望**具名辨識**，以及對應到 Claude 交付的經典**軟體工程素養**：

1. 在詞彙層級認得 **Strands**、**LangGraph** 和 **PydanticAI**（大致知道各自用途——不是教學課程）。
2. 用一張乾脆的決策表，在**自架 Agent SDK／自訂迴圈**與 **Anthropic 託管式 agent** 之間做選擇。
3. 回答那些假設你熟悉 REST／JSON／非同步、版本控制、程式碼審查、重構與系統生命週期階段的整合題幹——Claude Code／API 是適配點，不是軟體工程教科書。
4. 知道 **WebSocket** 相對於慣用 Claude 串流路徑（**伺服器推送事件（SSE）**）的位置。

考試形狀：挑對**控制模型**與**擁有權邊界**，不是憑記憶實作一個框架。

---

## 2. 關鍵地圖（考試辨識）

| 名稱／模式 | 大致工作 | 考試線索 |
| --- | --- | --- |
| **自訂迴圈** | 你自己寫 `while not done: model → tools → results` | 最大控制；每一種失敗模式都由你承擔 |
| **Claude Agent SDK** | Anthropic 的 harness（精神與 Claude Code 相同）跑在**你的**程序裡 | Claude 優先；MCP／子代理／權限主題 |
| **Anthropic Managed Agents** | 託管的 REST 形狀 agent 執行環境；Anthropic 運行 harness／沙箱／工作階段 | 營運快；資料落地與執行環境費用的取捨 |
| **LangGraph** | 明確的**圖**編排（節點／邊、狀態、檢查點） | 可稽核性、人在迴路中（HITL）中斷、確定性工作流程外皮 |
| **Strands（AWS）** | **模型驅動**的 agent 迴圈；與 AWS／Bedrock 高度親和 | 跨模型可攜性；OpenTelemetry（OTel）；AWS 原生部署 |
| **PydanticAI** | **型別化** agent／結構化且經驗證的 I/O（Pydantic 優先） | schema 保證；較輕的編排 |
| **SSE 串流** | Claude Messages 的預設串流（經 HTTP 從伺服器到用戶端） | 聊天使用體驗、首個 token 延遲（TTFT）、工具引數增量 |
| **WebSocket** | 雙向應用傳輸層（你的 UI ↔ 你的後端） | 中斷／HITL／語音——**不是** Claude API 的主要串流線路 |

---

## 3. 具名框架詞彙（考試辨識，不是教學課程）

> **請對照現行文件查證：** 星數、版本號、雲端 SKU（產品型號）與確切類別名稱都會變。背下**角色與取捨**，考前再瀏覽現行 README／文件。

### 3.1 它們與工作流程對 agent／型別化工具／圖編排的關係

CCDV-F 已經反覆操練**工作流程對 agent**（第 02 章）：已知步驟 → 確定性工作流程／有向無環圖（DAG）；帶工具的開放式目標 → 帶預算的 agent 迴圈。具名框架是**那些形狀的實作**：

| 關注點 | 優先認出…… |
| --- | --- |
| 固定或可稽核的多步驟流程，帶循環／HITL | **LangGraph**（圖編排） |
| 「給模型＋工具＋提示詞，讓迴圈自己跑」，且偏 AWS | **Strands**（模型驅動 agent） |
| Python 服務裡有保證的結構化輸出／型別化工具引數 | **PydanticAI**（型別化工具與驗證） |
| 在你自己基礎設施內的 Claude 原生寫程式／agent 產品迴圈 | **Claude Agent SDK** |
| 最小介面；完全擁有權；教學考試迴圈 | **自訂 Messages API 迴圈** |

它們在生產環境中**並不互斥**：團隊常把 Pydantic 驗證放進 LangGraph 節點裡，或從 Strands 經 Bedrock 呼叫 Claude，或把 Agent SDK 工作階段包在應用程式 WebSocket 後面。考試題幹通常問的是，哪一個**主要**控制模型符合該限制。

### 3.2 Strands（AWS Strands Agents）— 高層

- **大致用途：** 模型驅動的 agent 框架（Apache 授權的開源血統），為生產迴圈最佳化，具有一流的 **AWS／Bedrock** 整合主題、內建可觀測性（常見為 OpenTelemetry），以及公開材料中的多 agent 原語（群集／圖／交接風格模式）。
- **與考試概念的關係：** 比較接近 **agent 迴圈**，而不是手繪的工作流程 DAG。你宣告模型、工具、提示詞；框架擁有大部分回合機制。
- **對 Claude Agent SDK／自訂迴圈的取捨：**
 - **贏面：** 模型可攜性（不只限 Claude）；AWS 部署故事；較少自己動手的追蹤膠水。
 - **代價：** 又多一個框架介面要學；不是與 Claude Code 同一個 harness；第一次運行之前，通常要先做 Bedrock／身分與存取管理（IAM）設定。
 - **考試句子：** 「AWS 原生、不用重寫迴圈就能換模型」→ Strands 形狀的答案——不是「必須用 Claude Agent SDK」。

### 3.3 LangGraph — 高層

- **大致用途：** 來自 LangChain 生態系的圖形編排：你定義**節點**、**邊**、共享**狀態**，通常還有**檢查點**。當**流程本身**必須可檢視時（合規、長工作流程、明確的人為中斷），適配力強。
- **與考試概念的關係：** 體現**工作流程／圖編排**。模型在節點*內部*行動；控制流是你的。當「稽核每一次轉換」勝過「最大自主程度」時，它是理想選擇。
- **對 Claude Agent SDK／自訂迴圈的取捨：**
 - **贏面：** 明確控制、HITL 中斷、成熟生態系整合、耐久狀態模式。
 - **代價：** 更多編排程式碼／概念；模型中立（你要自己接 Claude）；對簡單的純 Claude agent，比薄的 Agent SDK 查詢迴圈更重。
 - **考試句子：** 「有狀態的多步驟，加上強制人工核准關卡與可重播狀態」→ LangGraph 形狀——不是無邊界的 agent 亂衝。

### 3.4 PydanticAI — 高層

- **大致用途：** Python 優先、**型別安全**的 agent／工具層，圍繞 Pydantic 模型打造——強調**經驗證的結構化輸出**與型別化工具參數，勝過重型多 agent 編排。
- **與考試概念的關係：** 放大**型別化工具**與**輸出合約**（第 02／03 章）。通常是較大系統裡面的*一層*，而不是整個編排故事。
- **對 Claude Agent SDK／自訂迴圈的取捨：**
 - **贏面：** schema／驗證文化；在副作用發生前攔下壞的工具引數；「單一 agent＋工具」服務的低鎖定。
 - **代價：** 主要不是圖編排器；多 agent／圖的需求可能把你推向再疊 LangGraph／Strands／Agent SDK。
 - **考試句子：** 「Python 服務絕不能把未驗證的 JSON 寫進總帳」→ PydanticAI／schema 優先——寫入仍要搭配權限。

### 3.5 取捨卡 — 框架對 Claude Agent SDK 對自訂迴圈

| 選項 | 何時最好 | 何時弱 |
| --- | --- | --- |
| **自訂 Messages 迴圈** | 教學清晰度；微型 agent；獨特控制需求 | 你會重新發明工作階段、壓縮（上下文壓縮）、權限、追蹤 |
| **Claude Agent SDK** | 只限 Claude；想要像 Code 的 harness（工具、MCP、子代理）跑在**你的**程序裡 | 需要非 Claude 模型；無法在自己基礎設施裡執行？→ 考慮託管 |
| **Managed Agents（Anthropic 託管）** | 想讓 Anthropic 運行 harness／沙箱／工作階段；快速交付 | 嚴格資料落地／僅限虛擬私有雲（VPC）的工具執行；需要特殊迴圈控制 |
| **LangGraph** | 流程本身就是產品；檢查點；HITL 圖 | 簡單單工具問答（殺雞用牛刀） |
| **Strands** | 以 AWS／Bedrock 為中心；模型可攜的 agent 迴圈 | 純 Anthropic 技術棧、且偏好與 Code 對等的 harness |
| **PydanticAI** | Python 裡型別化 I/O 保證 | 複雜多方編排才是主要問題 |

**反陷阱：** 點名一個熱門框架永遠不夠。題幹仍然期望**預算、最小權限原則、評測與停止條件**（第 02／04 章）。

---

## 4. 自架對 Anthropic 託管的 Managed Agents — 決策表

公開的 2026 年產品劃分（請即時查證）：**Claude Agent SDK**＝**你的**程序裡的函式庫；**Claude Managed Agents**＝託管的 agent 執行環境（REST／事件 API 主題），由 **Anthropic** 運行 harness、沙箱與耐久工作階段機制。底下是同樣的 Claude 模型；不同的是**營運擁有權**。

| 維度 | 自架（Agent SDK／自訂迴圈／你的 worker） | Anthropic 託管的 Managed Agents |
| --- | --- | --- |
| **誰運行迴圈** | 你 | Anthropic |
| **工具／沙箱在哪裡執行** | 你的基礎設施（VPC、筆電、Kubernetes（k8s）） | Anthropic 管理的沙箱（VPC connector 可能存在——請查證文件） |
| **工作階段／當機復原** | 你自己建或採用現成的 | 平台的責任 |
| **資料落地壓力** | 控制更強（若你那樣設計，只有推論離開邊界） | 工作階段日誌＋沙箱在供應商那一側 → 合規審查 |
| **成本形狀** | token ＋**你的**運算／營運 | token ＋**執行環境／工作階段**費用（有公開報導；請查證） |
| **客製化深度** | 最高（自訂重試、特殊工具、實體隔離（air-gapped）模式） | 產品介面高、基礎設施自己動手較少；特殊迴圈可能不合 |
| **到生產環境的時間** | 若缺乏 harness 成熟度則較慢 | 若限制允許託管執行則較快 |
| **與 Claude Code 對等** | Agent SDK 目標是在本地／程序內提供像 Code 的 harness | 託管＝服務形狀；仍是 Claude 家族 agent |
| **典型考試選擇** | 受監管資料、自訂權限、既有機群、你已在營運的極端並行成本 | 非同步長時間運行 agent、小團隊、不想擁有沙箱 |

```text
真的需要 agent 迴圈嗎？
 否 → Messages API（＋工具）就夠了
 是 → 工具／狀態必須留在你的邊界內嗎（資料落地、VPC、自訂沙箱）？
 是 → 自架 Agent SDK 或自訂迴圈（＋可選 LangGraph／Strands／PydanticAI）
 否 → 交付速度／託管沙箱值得執行環境費用嗎？
 是 → Anthropic 託管的 managed agents
 否 → 仍然自架（成本或控制偏好）

原型路徑（常見的公開指引主題）：本機 Agent SDK → 營運痛點主導時改託管——搬動機密／PII 之前仍要重新檢查合規。
```

**考試陷阱**

1. 把「用 Claude」等同於「必須用 Managed Agents」。
2. 把「Agent SDK」等同於「Anthropic 託管我的沙箱」。
3. 忽略 **Message Batches（批次 API）≠ 託管式 agent**（批次＝非同步模型呼叫；不是你本地的工具迴圈）。
4. 忘記 **hooks／權限／評測** 在兩種模式下照樣適用。

---

## 5. 軟體開發生命週期（SDLC）／軟體工程基礎中的 Claude（考試配分卡）

已公布藍圖配分讓**軟體工程基礎（約 7.4%）**與**系統生命週期（約 2.8%）**成為該讀的材料——不是「假設的背景知識」。當作**對應到 Claude 的素養**來讀，不是軟體工程課程。

### 5.1 REST／JSON／非同步（技術基礎）

| 概念 | 考試有用的 Claude 對應 |
| --- | --- |
| **REST** | 表現層狀態轉換（REST）。Messages API 是以 HTTP 資源為導向的；SDK 包裝 REST。優先用官方 SDK 取得重試／串流輔助；概念上要知道標頭（`x-api-key`、API 版本）。 |
| **JSON** | 工具引數、結構化輸出、MCP 酬載、日誌。在伺服器端驗證；絕不要只信任模型的 JSON 就做寫入。 |
| **非同步** | （1）你應用程式裡的並行工具呼叫／`asyncio` worker；（2）離線模型工作用的 Message Batches；（3）託管／長時間運行的 agent 工作階段。不要把三者混為一談。 |
| **SDK 優先於 REST** | SDK 是預設的生產選擇；除錯線路格式或沒有 SDK 的語言時才用裸 REST。 |

### 5.2 版本控制、程式碼審查、重構

| 實務 | Claude 適配何處 | 考試判斷 |
| --- | --- | --- |
| **版本控制系統（VCS，git）** | 權威狀態活在提示詞之外；分支／pull request（PR）是變更的單位 | agent 應在分支上提交；絕不要把強制推送到 `main` 當作預設自主權 |
| **程式碼審查** | Claude Code／PR agent 起草；人類（或政策機器人）擁有合併關卡 | 高風險：密鑰、授權（authZ）、遷移、不可逆腳本 → 必須人工審查 |
| **重構** | 計畫模式（plan mode）→ 測試優先 → 小差異 → 驗證 | 偏好有測試支撐的重構；無邊界的「清理整個倉庫」agent 在題幹裡失敗 |

**一句話：** Claude 加速撰寫；**版本控制＋審查＋CI** 仍然是控制平面。

### 5.3 系統生命週期階段（經典標籤 → Claude 交付）

| 階段 | Claude／CCDV 適配 |
| --- | --- |
| **需求／探索** | 待完成工作（JTBD）、限制（延遲、成本、安全）、成功指標——在挑模型之前 |
| **設計** | 工作流程對 agent；工具／MCP 邊界；託管模式；schema；威脅模型 |
| **實作** | API 應用程式、Agent SDK／Code、作為版本化構件的提示詞、pin bundle（綁定版本組） |
| **驗證／評測** | 黃金標準集、回歸評測框架、副作用失敗測試（第 04 章） |
| **部署** | 金絲雀、功能旗標、受管理設定、緊急停止開關 |
| **營運／監控** | 追蹤、每次成功成本、快取命中率、事故 runbook（維運手冊） |
| **演進／退役** | 提示詞／工具版本化、模型遷移、用樁棄用工具 |

**考試句子：** 從炫的展示直接跳到生產環境自主，**跳過**驗證與部署關卡——即使模型是 Opus 層級，答案仍是錯的。

### 5.4 Claude Code／API 如何適配 CI 與審查

```text
PR 開啟
 → CI：單元＋schema／工具合約測試＋評測子集（便宜）
 → Claude Code 無頭模式／Agent SDK 工作（釘定模型）在分支上提案或實作
 → 靜態檢查＋密鑰掃描＋權限政策 lint
 → 人工審查（破壞性／授權／資料路徑必須）
 → 合併 → 分階段部署 → 金絲雀評測 → 晉升
```

| 適配點 | 優先選 |
| --- | --- |
| 聊天使用體驗的 token | 串流 Messages（經 SDK 的 SSE） |
| 離線分類／摘要 | Message Batches |
| 倉庫本地的 agentic 編輯 | Claude Code／帶釘定＋允許清單的 Agent SDK |
| 組織政策 | 受管理設定 > 專案方便 |
| 品質真相 | CI 裡的評測框架，不是 Slack 裡的感覺 |

---

## 6. WebSocket 簡短說明（串流 SDK）

**Claude Messages 串流（供應商 → 你的後端）** 公開文件記載為 **HTTP 伺服器推送事件（SSE）**（`text/event-stream` 主題）：`message_start` → 內容區塊增量 → `message_stop`。官方 SDK 會為你解析。

**WebSocket** 在 CCDV 形狀的題幹裡，是**應用傳輸層**的選擇：

| 傳輸層 | 方向 | 與 Claude 應用的典型用途 |
| --- | --- | --- |
| **SSE（Claude API 串流）** | 伺服器 → 用戶端（單向） | token 串流、TTFT 使用體驗、細粒度工具引數預覽 |
| **SSE（你的 API → 瀏覽器）** | 伺服器 → 瀏覽器 | 映射 API 串流的簡單聊天介面 |
| **WebSocket（你的應用）** | 雙向 | 串流中途使用者中斷、同一工作階段上的 HITL 核准、語音、協作 agent |
| **輪詢** | 請求／回應 | 批次工作狀態——不是聊天 TTFT |

**考試線索**

- 「聊天要更低的 TTFT」→ 串流（SSE 路徑），不是為了 WebSocket 而用 WebSocket。
- 「核准一個破壞性工具，而又不能把第二個 HTTP POST 關聯到一條已死的 SSE」→ 在 **UI 與你的後端** 之間用 WebSocket（或精心設計的工作階段）；後端仍然經 SDK／SSE／REST 與 Claude 交談。
- 遠端 MCP 傳輸層常被描述為 HTTP／SSE 風格——不要把 MCP 線路和瀏覽器 WebSocket 搞混。

**陷阱：** 宣稱 Claude Messages API「是 WebSocket」。在辨識層級，**SSE 是預設串流**；WebSocket 通常是**你自己**產品的雙向通道。

---

## 7. 決策樹（壓縮版）

### 7.1 框架／harness 挑選

```text
主要限制？
 只要型別化 Python I/O → PydanticAI（± 薄迴圈）
 可稽核的圖＋HITL 檢查點 → LangGraph
 AWS／Bedrock＋模型可攜性 → Strands
 我們程序裡像 Claude Code 的 harness → Claude Agent SDK
 託管沙箱＋不想自己跑 harness → Managed Agents
 考試教學／獨特控制 → 自訂 Messages 迴圈
```

### 7.2 題幹裡的 SDLC 紅旗

```text
沒有成功指標 → 修需求
自主程度提高 ↑ 之前沒有評測 → 修驗證
沒有分支／PR → 修版本控制紀律
只用提示詞說「永不刪除」 → 修權限／hooks
正式環境用展示用模型 ID → 修釘定＋設定管理
```

---

## 8. 考試陷阱

1. **只點名框架而沒有停止條件**——仍然是錯的。
2. **對單次常見問答用 LangGraph**——殺雞用牛刀；偏好提示詞／工具。
3. **用 Managed Agents 來滿足實體隔離的工具執行**——通常應自架。
4. **Agent SDK＝Anthropic 託管我的 VPC**——假的。
5. **任何串流都需要 WebSocket**——假的；SSE 是 API 預設。
6. **批次取代 CI 評測**——假的。
7. **沒有測試的重構 agent**——軟體工程基礎題幹失敗。
8. **因為是 Claude 寫的就跳過程式碼審查**——審查／安全題幹失敗。

---

## 9. 自我檢核問答（18）

**Q1.** 考試列出 Strands、LangGraph、PydanticAI——每一個各自負責什麼辨識工作？
**A1.** Strands ≈ 模型驅動（常為 AWS 原生）的 agent 迴圈；LangGraph ≈ 明確的圖／狀態編排；PydanticAI ≈ 型別化／經驗證的結構化 I/O agent。

**Q2.** 什麼時候 LangGraph 比自由 agent 迴圈是更好的心智模型？
**A2.** 當轉換必須可稽核、可設檢查點，或可由人類中斷，作為流程定義的一部分。

**Q3.** 什麼時候你會挑 Strands 而不是 Claude Agent SDK？
**A3.** 需要模型可攜性，以及／或 AWS／Bedrock 原生部署／可觀測性，勝過與 Claude Code 對等的 harness。

**Q4.** PydanticAI 對「只在提示詞裡用 JSON 模式」？
**A4.** PydanticAI／schema 驗證在程式碼裡強制型別；只靠提示詞的 JSON，失敗即關閉的可靠性較低。

**Q5.** 自架 Agent SDK 對 Managed Agents——第一個決定性問題？
**A5.** 工具執行與工作階段構件，必須留在我們的資料落地／控制邊界之內嗎？

**Q6.** 用 Managed Agents 會消除評測與允許清單的需求嗎？
**A6.** 不會——託管改變營運擁有權，不改變安全或品質義務。

**Q7.** 自訂迴圈對 Agent SDK——考試一句話？
**A7.** 自訂＝最大控制／最大責任；Agent SDK＝Claude 導向的 harness，讓你不用重建工作階段／工具／權限的基本件。

**Q8.** 哪一項 REST／JSON 素養最可能出現在 CCDV-F？
**A8.** 建構 Messages 風格的請求、驗證 JSON 工具引數、用 SDK 而非脆弱的臨時 HTTP、處理串流事件。

**Q9.** 在生產環境題幹裡，agent 應該怎麼和 git 互動？
**A9.** 分支＋PR；尊重受保護的 `main`；不提交密鑰；有風險的差異在合併前要審查。

**Q10.** 程式碼審查關卡相對於 Claude Code 坐在哪裡？
**A10.** 在 agent 編輯之後、合併／部署之前——尤其授權、資料、破壞性操作；Claude 起草，政策＋人類把關。

**Q11.** 把「系統生命週期」對應到一次 Claude 功能發布。
**A11.** 需求 → 設計（agent 對工作流程、託管）→ 實作（釘定、工具）→ 評測 → 金絲雀部署 → 監控 → 版本化／退役。

**Q12.** Claude API 串流的預設線路格式？
**A12.** HTTP 上的 SSE；SDK 抽象掉事件解析。

**Q13.** 什麼時候 Claude 應用裡用 WebSocket 是合理的？
**A13.** 雙向需求：串流中途取消／中斷、單一工作階段上的 HITL、語音——在用戶端與**你的**後端之間。

**Q14.** Message Batches 對託管的長時間運行 agent？
**A14.** 批次＝帶限制的非同步模型推論；託管式 agent＝託管的 agent harness／工作階段，帶工具／沙箱主題。

**Q15.** 重構題幹：一個 agent 一夜之間重寫了半個 monorepo（單一儲存庫）。哪裡錯了？
**A15.** 缺少 SDLC 控制——範圍、測試、小 PR、審查；沒有驗證的自主。

**Q16.** 軟體工程基礎裡，「非同步」的三種不同意思？
**A16.** 應用程式並行；Message Batches；長時間運行的 agent 工作階段／worker——不要把答案混在一起。

**Q17.** 能把 PydanticAI 型別和 LangGraph 編排結合嗎？
**A17.** 可以，概念上——型別化驗證活在圖節點內；考試仍可能問哪個關注點是主要的。

**Q18.** 在 Agent SDK 上做原型，然後搬到 Managed Agents——合規的陷阱是什麼？
**A18.** 工具／沙箱／工作階段資料的落地可能改變；在晉升承載個人識別資訊（PII）的工作負載之前，重新檢查政策。

---

## 10. 檢查清單

- [ ] 我能不用程式碼，一句話說清 Strands 對 LangGraph 對 PydanticAI。
- [ ] 我能依資料落地、成本與控制線索，在自架 Agent SDK／自訂迴圈與 Anthropic Managed Agents 之間做選擇。
- [ ] 我知道自訂迴圈／Agent SDK／託管／圖框架是不同的擁有權模型。
- [ ] 我能正確把 REST／JSON／非同步對應到 Messages、工具、批次與 worker。
- [ ] 我把 git 分支／PR＋程式碼審查，當作圍繞 Claude 編輯的強制控制平面。
- [ ] 我能帶著 Claude 適配點走完系統生命週期階段（設計 → 評測 → 金絲雀 → 營運）。
- [ ] 我能把 Claude Code／API 工作放進 CI，而不跳過風險變更的人工關卡。
- [ ] 我能區分 SSE（API／預設串流）與 WebSocket（雙向應用通道）。
- [ ] 無論框架品牌為何，我仍然套用預算、允許清單、hooks 與評測。
- [ ] 考前我會在現行官方文件上查證框架與 Managed Agents 細節。

---

## 11. 詞彙表

| 術語 | 備考含義 |
| --- | --- |
| **模型驅動 agent** | 框架擁有工具迴圈；你提供模型／工具／提示詞（Strands 形狀） |
| **圖編排** | 明確的節點／邊／狀態（LangGraph 形狀） |
| **型別化工具** | 經 schema 驗證的引數／結果（PydanticAI 形狀／JSON Schema 工具） |
| **自架 agent** | 迴圈與工具執行在你的程序／基礎設施裡 |
| **託管式 agent** | 供應商託管的 harness／沙箱／工作階段執行環境 |
| **SSE** | 伺服器推送事件——Claude 慣用串流傳輸層 |
| **WebSocket** | 全雙工應用傳輸層，用於中斷／HITL／語音模式 |
| **SDLC** | 軟體開發生命週期／系統生命週期——從需求到退役 |
| **軟體工程基礎** | REST／JSON／非同步、VCS、審查、重構——有考試配分的素養 |
| **pin bundle** | 為 CI 與生產鎖定的模型／SDK／提示詞／工具版本 |

---

## 12. 若考試問 X → 想 Y

| 若考試問…… | 想…… |
| --- | --- |
| 可稽核多步驟 HITL 的具名框架 | LangGraph |
| 以 AWS Bedrock 為中心、可攜的 agent 迴圈 | Strands |
| 嚴格的 Python 結構化輸出／型別化工具 | PydanticAI |
| 在我們 VPC 裡、精神與 Claude Code 相同的 harness | Agent SDK（自架） |
| 不想營運沙箱；可接受託管工作階段 | Managed Agents |
| 最低層清晰度／特殊控制 | 自訂 Messages 迴圈 |
| 聊天打字效果／TTFT | 經 SDK 的 SSE 串流 |
| 在同一互動工作階段核准工具 | WebSocket（UI↔後端）＋伺服器端關卡 |
| Claude 改了程式碼——現在合併？ | 先 PR＋審查＋CI 評測 |
| 缺了哪個生命週期步驟？ | 通常是評測或部署關卡 |

---

## 附錄 — 章節 → 官方領域

| 領域 | 第 06 章涵蓋 |
| --- | --- |
| Agent 與工作流程（14.7%） | 具名框架；託管模式；harness 取捨 |
| 應用與整合（33.1%） | 軟體工程基礎；SDLC；REST／JSON／非同步；CI／審查適配；串流傳輸層 |
| Claude Code（3.1%） | 無頭模式／CI 的 agent 主機；釘定；審查工作流程 |
| 工具與 MCP（10.6%） | 型別化工具重疊；MCP 仍與框架品牌正交 |
| 資安／評測 | 資料落地託管線索；審查／CI 關卡（指向第 04 章） |

*第 06 章結束。*
