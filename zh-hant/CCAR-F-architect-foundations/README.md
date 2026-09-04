# CCAR-F — Claude 認證架構師基礎考試（Foundations）：學習包索引

原創備考筆記。**非** Anthropic 官方材料；以作者自己的話撰寫，供專注閱讀與認證複習。

> **註記：** 本資料夾（`CCAR-F-architect-foundations`，於 2026-08-23 更名）涵蓋 **CCAR-F（Claude Certified Architect – Foundations）** — 認證名稱與代碼已對照 Pearson VUE 的 Anthropic 官方專案列表確認。獨立的 `CCAR-P-architect-professional` 目錄涵蓋 Professional 考試。

---

## 考試（出自官方考試指南 v1.0，2026 年 7 月 — 權威來源；請從 Pearson VUE 的 Anthropic 官方專案頁面下載）

| 項目 | 值 |
| --- | --- |
| 憑證 / 代碼 | Claude Certified Architect – Foundations / **CCAR-F** |
| 題數 | **60**（單選 + 複選；每題會註明應選幾個選項） |
| 結構 | **4 個情境題，從 6 題題庫中隨機抽取**（考試指南印出全部 6 題） |
| 時間 | 120 分鐘 |
| 及格 | **720** 量尺分數（100–1,000）；成績報告含各領域百分比 |
| 費用 / 效期 | **USD $125** / 12 個月（準時續證免費、免監考） |
| 重考 | 等待期 14 / 30 / 90 天；任一滾動 12 個月最多 4 次 |
| 交付 | Pearson VUE（線上監考或考場） |

### 考試藍圖（配分比重即讀書時間的鐵律）

| # | 領域 | 配分比重 | 主要檔案 |
| --- | --- | --- | --- |
| 1 | Agentic 架構與編排 | **27%** | **08**（+ 03 §agent 迴圈） |
| 2 | 工具設計與模型上下文協定（MCP）整合 | 18% | **04 + 其領域 2 補充篇**（+ 03 §工具） |
| 3 | Claude Code 設定與工作流程 | **20%** | **05 + 其領域 3 補充篇** |
| 4 | 提示工程與結構化輸出 | **20%** | **03**（schema、tool_choice、batches、few-shot） |
| 5 | 上下文管理與可靠性 | 15% | **09**（+ 05 §壓縮） |

**明確不在考試範圍（不要把備考時間花在這裡）：** 雲端供應商設定（AWS/GCP/Azure）、串流／伺服器推送事件（SSE）實作、速率限制與定價、驗證協定、提示詞快取的實作細節、Constitutional AI／人類回饋強化學習（RLHF）、向量嵌入／向量資料庫、電腦操作（computer use）、視覺、微調、MCP 伺服器託管／部署、token 計數。

### 6 個情境題（考卷會出現其中 4 個）

1. 客服解決 agent（D1、D2、D5）
2. 以 Claude Code 產生程式碼（D3、D5）
3. 多 agent 研究系統（D1、D2、D5）
4. 以 Claude 提升開發者生產力（D2、D3、D1）
5. 以 Claude Code 做持續整合（CI）（D3、D4）
6. 結構化資料萃取（D4、D5）

---

## 檔案 — 依 CCAR-F 標註優先級

| # | 檔案 | 涵蓋 | CCAR-F 優先級 |
| --- | --- | --- | --- |
| 08 | [08-agentic-orchestration-agent-sdk.md](./08-agentic-orchestration-agent-sdk.md) | **領域 1（27%）**：agentic 迴圈、協調者–子代理、Task tool／AgentDefinition、hooks、任務拆解、sessions／forking | **主要 — 先讀** |
| 09 | [09-context-reliability.md](./09-context-reliability.md) | **領域 5（15%）**：案例事實、中段遺失、升級、錯誤傳播、草稿區／清單檔、校準、出處可追溯性 | **主要** |
| 05 | [05-claude-code-in-action.md](./05-claude-code-in-action.md) + 領域 3 補充篇 | **領域 3（20%）**：引導／設定概念 + CLAUDE.md 階層、@import、.claude/rules/、commands、SKILL.md frontmatter、-p／--output-format／--json-schema | **主要** |
| 03 | [03-building-with-claude-api.md](./03-building-with-claude-api.md) | **領域 4（20%）** + D1／D2 的 API 半部：tool_use、tool_choice、schema、batches、agent 迴圈 | **主要** |
| 04 | [04-introduction-to-mcp.md](./04-introduction-to-mcp.md) + 領域 2 補充篇 | **領域 2（18%）**：MCP 概念 + isError、錯誤分類法、.mcp.json vs ~/.claude.json、內建工具 | **主要** |
| 01 | [01-claude-101.md](./01-claude-101.md) | claude.ai 產品基礎 | 僅背景 — 屬 CCAO-F 範圍，CCAR-F 產出低 |
| 02 | [02-ai-fluency.md](./02-ai-fluency.md) | 4D 流暢度框架 | 僅背景 — CCAR-F 產出低 |
| 06 | [06-claude-with-amazon-bedrock.md](./06-claude-with-amazon-bedrock.md) | Bedrock 存取、身分與存取管理（IAM）、Converse | **CCAR-F 不在考試範圍**（考試指南排除雲端供應商設定）— 平台工作可留，本考試請跳過 |
| 07 | [07-claude-with-google-cloud-vertex-ai.md](./07-claude-with-google-cloud-vertex-ai.md) | Vertex AI 設定、應用程式預設憑證（ADC）、端點 | **CCAR-F 不在考試範圍** — 同 06 |

*（檔案 01–07 源自 Claude Academy 課程筆記 — Claude 101、AI Fluency、Building with the Claude API、Intro to MCP、Claude Code in Action、Bedrock、Vertex。檔案 08–09 以及 04／05 的補充篇於 2026-08-23 新增，用以補齊本包對考試指南藍圖的缺口。）*

**應試技巧**（配速、六情境策略、反射表、干擾選項模式）：[PRO-TIPS.md](./PRO-TIPS.md) — 讀完內容檔後再看。

---

## 建議研讀順序（由配分比重驅動）

1. **檔案 08** — 領域 1 佔 27%，且驅動 6 個情境中的 3 個；沒有其他檔案報酬更高。
2. **檔案 05 + 其領域 3 補充篇** — 20%；補充篇收錄精確機制（rules 的 glob、commands 作用範圍、CI 旗標），十二題範例題中有兩題考這些。
3. **檔案 03** — 領域 4（20%）：tool_use schema、tool_choice 的 auto／any／forced、nullable 欄位、帶回饋重試的主題、Batches（50%／24h／custom_id／不支援多輪工具呼叫）。
4. **檔案 04 + 其領域 2 補充篇** — 18%：描述、錯誤分類法、isError、作用範圍。
5. **檔案 09** — 領域 5（15%），然後重掃 08 的交接小節（兩章互相扣連）。
6. 把考試指南 PDF 裡的 **12 題範例題**全部冷答一遍；每一題都能從檔案 03／04／05／08／09 作答 — 答錯會告訴你該重讀哪一檔。
7. 時間允許時親手做指南的 **4 個準備練習**（與情境題庫一對一對應）。

## 這些筆記是怎麼做成的

- 檔案 01–07：公開課程頁面用於標題、模組清單與載明的學習目標；內文為原創綜合整理。
- 檔案 08–09 + 補充篇：直接對照官方考試指南的任務陳述撰寫，SDK／CLI 機制已於 2026-08 對照當時公開的 Claude Code／Agent SDK 文件驗證。名稱會漂移 — 考試前請對照線上文件核對 `--resume`、SKILL.md frontmatter，以及 Agent SDK 選項拼法。特別注意：考試指南寫 **Task tool**；現行 Claude Code 已改名為 **Agent** — 考試作答用指南的用語。
- 時效修正 2026-08-23：現行前沿模型移除 temperature／取樣參數（03、06、07）、自適應思考對上已棄用的思考預算（06）、對話中途系統訊息註記（03）。

## 相關（僅指標）

- 進階 MCP（讀完 `04-…` 的入門筆記之後）：[Model Context Protocol: Advanced Topics](https://academy.claude.com/courses/model-context-protocol-advanced-topics)
