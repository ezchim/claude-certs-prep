---
title: 通过 Amazon Bedrock 使用 Claude — 备考笔记（主要来源）
source: https://academy.claude.com/courses/claude-with-amazon-bedrock
disclaimer: 用于备考的原创笔记——不是 Anthropic 或 AWS 官方教材。不是课堂逐字稿。
approx_length: 约 5500–7000 字
deepened: 2026-08-23
cross_check: AWS Bedrock 公开文档（Converse、InvokeModel、推理配置文件、IAM）
---

# 通过 Amazon Bedrock 使用 Claude — 主要备考笔记

> **免责声明：** 这些是对齐 [Claude with Amazon Bedrock](https://academy.claude.com/courses/claude-with-amazon-bedrock) **公开** Claude Academy 课程大纲的**原创**备考笔记。它们**不是** Academy 课堂内容转储或课程原文。请搭配现行 AWS Bedrock 与 Anthropic 文档，核对现行模型 ID、区域与 API 字段。完成官方课程仍是小考的权威依据。

**适合对象：** 要通过 Amazon Bedrock 把 Claude 功能加进应用程序的开发者。

**先修（课程）：** Python、JSON 基础、已启用 Bedrock 模型访问的 AWS 账号。

**如何使用：** 平台专属章节（启用、ID、身份与访问管理（IAM）、Converse 对上 Invoke、驻留）是相对于 Anthropic API 与 Vertex 路线、回报率最高的差异点。共享的 Claude 技能（提示、工具、检索增强生成（RAG）、模型上下文协议（MCP）、agent）仍然会出现——要认得它们在 Bedrock 请求形状里的样子。

---

## 1. 课程地图（公开模块）

公开页面（约 65 堂课、多个小考）大致把主题分成：

1. **使用 API** — 验证、基本请求、对话管理、系统提示词、结构化输出
2. **提示工程** — 策略、评测框架、系统化测试
3. **工具使用** — JSON Schema 工具、多轮工具循环、批量工具调用、内置工具
4. **RAG** — 切块、向量嵌入（embedding）、BM25 混合搜索、多索引、重排序、上下文化检索
5. **模型上下文协议（MCP）** — 工具、资源、提示；服务器与客户端
6. **Agent** — 创建在 Bedrock 访问之上的 Claude Code／电脑操作风格自动化模式
7. **进阶 Claude 功能** — 扩展思考、视觉、提示词缓存、流式、temperature、结构化提取

以认证考试的角度准备，优先学**访问模式、模型 ID、IAM、区域、Converse 对上 InvokeModel**，然后弄清楚当传输层是 Bedrock 时，共享的 Claude 技能是什么样子。

**情境题主轴：** 启用模型 → 选区域／配置文件 → IAM 最小权限原则 → Converse（默认）或 Invoke（原生） → 历史自己管理 → 视需要加入工具／RAG／评测 → 考虑驻留的配置文件选择。

---

## 2. Bedrock 上的访问模式

### 2.1 启用与凭证

1. 在 Bedrock console（按账号／区域）**请求／启用**你需要的 Anthropic Claude 模型。
2. 用标准 AWS 凭证链（环境变量、共享设置、实例角色、SSO 等）调用 **Bedrock Runtime**。
3. 在你的区域创建 runtime client，例如 `boto3.client("bedrock-runtime", region_name="us-east-1")`。

经典的 Bedrock Runtime 调用，你**不会**发送 Anthropic API 密钥。计费与租户挂在 AWS 上。（独立产品如 AWS 上的 Claude Platform 可能不同——要看清楚题目点名的是哪个产品。）

**常见的启用失败：**

- 模型在 A 区域启用了，client 指向 B 区域
- 模型启用了，但在需要推理配置文件时调用原始基础模型 ID
- 凭证对 AWS 整体有效，但角色在正确的 Amazon 资源名称（ARN）上缺少 Bedrock invoke

### 2.2 模型 ID 与推理配置文件

Bedrock 用这类 ID 识别 Claude：

- 基础模型式：`anthropic.claude-…`
- **跨区域推理配置文件**：通常带地理前缀，例如 `us.anthropic.claude-…`、`eu.anthropic.…`、`global.anthropic.…`
- **应用程序推理配置文件**：客户自建的配置文件（常用于成本归属或钉在单一区域）

**考试关键事实：**

- 不是每个模型在每个区域都存在。调用不存在的模型，会得到令人困惑的「不存在」／不支持按需那一类错误。
- 较新的 Claude 模型经常要求**推理配置文件 ID**，而不只是原始基础模型 ID。
- 推理配置文件会路由到模型所在的区域。确切 ID 要在 Bedrock console 的跨区域推理／模型详细页面上查。
- IAM 必须允许你实际调用的资源（两者都用上时，基础模型 ARN **以及**推理配置文件 ARN）。范例政策里 ARN 的区域位置用通配符很常见——仍要坚持最小权限原则。

永远在 AWS 模型卡片上确认目前的 ID——字符串会随新 Claude 版本推出而改变。

**决策树——该传哪个 ID：**

```text
该模型在随需推论时是否需要推论设定档？
 是 → 使用系统地理设定档（us./eu./global.）或你的应用程序设定档 ARN／ID
 否 → 基础模型 ID 在该区域也许可用（仍要验证）

需要单一区域落地？
 → 不要使用多区域地理设定档；使用指向单一区域基础模型 ARN 的应用程序设定档
 （或 AWS 为该区域记载的其他区域内选项）

需要按团队／应用程序做成本归属？
 → 带标签的应用程序推论设定档
```
---

## 3. Converse API 对上 InvokeModel（高回报率）

| | **Converse／ConverseStream** | **InvokeModel／InvokeModelWithResponseStream** |
| --- | --- | --- |
| 形状 | 统一的 Bedrock 消息 schema | 模型原生 JSON body |
| 可携性 | 许多 Bedrock 聊天模型共用同一代码形状 | 供应商专属 payload |
| 多轮／工具 | 一等的 `messages`、`system`、`toolConfig` | 你自己组出 Anthropic Messages 风格的 body |
| Claude 额外项 | Converse 没浮出来的旋钮，用 `additionalModelRequestFields` | 完整的原生 Anthropic 请求字段 |
| 何时用 | 聊天、agent、工具的默认 | 向量嵌入／图像生成、旧有用途，或只有原生才有的功能 |

**考试默认：** 在 Bedrock 上跑对话式 Claude，优先用 **Converse**。当你需要原生 Anthropic Messages payload（某些工具类型／字段）或非聊天模态时，才降到 **InvokeModel**。

AWS 公开文档强调：Converse 为支持消息的 Bedrock 模型提供一致的 API；独特参数仍然通过模型专属结构传入。

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

从类似 `response["output"]["message"]["content"][i]["text"]` 的路径读文本（确切索引取决于内容区块）。

**无状态：** Bedrock 不会替你记住先前的轮次。每次调用，你都必须重送完整的用户／助理交替历史。

### 3.2 InvokeModel 心智模型（Claude）

Body 通常包含：

- `anthropic_version`（Bedrock Claude Messages 的值，例如 `bedrock-2023-05-31`——确认现行文档）
- `max_tokens`（原生 Messages 风格里必填）
- `messages` 数组
- 可选的 `system`、`temperature`、tools 等

你要自己解析 Anthropic 形状的响应。内容类型通常是 `application/json`；在 boto3 里 body 是 bytes（`json.dumps(...).encode()`）。

### 3.3 流式

| 非流式 | 流式 |
| --- | --- |
| `converse` | `converse_stream` |
| `invoke_model` | `invoke_model_with_response_stream` |

与非流式同一 IAM 家族，再加上**流式专属**动作。只授予 `bedrock:InvokeModel`，是非流式正常、流式却以 AccessDenied 失败的经典原因。

流式响应以事件序列送达，由你的代码重组（内容增量、消息停止等）。流式改变的是递送方式，不是对工具／历史正确性的需求。

### 3.4 当 Converse 不够用时

选 InvokeModel，当：

- 你需要向量嵌入或图像生成模型（不是聊天型 Converse）
- 某个 Claude 字段在原生 Messages 形式里比较容易，或只有那里有文档
- 调试需要看到确切的 Anthropic 形状 body
- 迁移现有 Anthropic API 代码，想做最少对应（仍然要改验证、版本字段、端点）

选 Converse，当：

- 创建横跨模型家族、可携的 Bedrock 聊天／agent
- 你想要一等的 `toolConfig` 与 Guardrails 集成模式
- 团队标准化在单一 Bedrock 对话 API 上

---

## 4. IAM 要点（最小权限原则）

Converse 与 Invoke 最终都用 Bedrock 推理动作授权：

- `bedrock:InvokeModel` — `InvokeModel` **以及** `Converse` 都需要
- `bedrock:InvokeModelWithResponseStream` — 流式变体需要

**出人意料的考试事实：** `Converse` 上的 `AccessDeniedException`，修复方法常常是授予 `bedrock:InvokeModel`，不是经典政策里那个不存在的独立 `bedrock:Converse` 权限。AWS 文档载明 Converse 需要 `bedrock:InvokeModel`；ConverseStream 需要 `bedrock:InvokeModelWithResponseStream`。

使用配置文件／在 console 选择时，还要授予：

- `bedrock:GetInferenceProfile` — 以推理配置文件执行推理
- `bedrock:ListInferenceProfiles` — 在 console 里选择配置文件
- 对正确**模型／推理配置文件 ARN** 的访问
- 任何 Guardrails／知识库动作，如果题目包含那些功能

**推理配置文件 IAM 陷阱：** 使用跨区域配置文件时，政策必须允许对以下项目 invoke：

1. 推理配置文件 ARN（入口）
2. 配置文件可能路由到的**每个目的地区域**里，底层的基础模型 ARN

只允许配置文件 ARN，是 Lambda 角色常见的失败模式。

最小权限检查清单：

- 尽可能按账号与区域缩小范围
- 列出特定模型家族 ARN，而不是 `*`
- 生产环境与实验使用不同角色
- 拒绝你的应用程序不需要的危险相邻动作（模型客制化、marketplace 购买等），除非真的需要
- 驻留方面：在政策要求时，把配置文件选择与 IAM 区域条件结合起来
---

## 5. 区域与数据驻留

- 为 runtime client 选一个支持你访问模式的**来源区域**。
- 当你需要跨地理区的可用性，用**地理推理配置文件**（`us.`、`eu.`……）。
- 当被允许且驻留有弹性时，用 **global** 配置文件。
- **单一区域驻留**，遵循 AWS 目前指引：应用程序推理配置文件指向单一区域的基础模型 ARN，或适用时的其他区域内选项。**不要**假设跨区域配置文件会把数据留在单一可用区／区域。

合规题：让配置文件地理对上政策，胜过「教程里碰巧可用的任何模型 ID」。

**驻留决策树：**

```text
落地有弹性、要极大化可用性？
 → global 推论设定档（若模型支持）

必须大致待在美国或欧盟？
 → us. 或 eu. 地理设定档

必须待在单一区域？
 → 钉在该区域基础模型 ARN 的应用程序推论设定档
 （＋你组织要求的 IAM 区域条件）

不确定模型是否可用？
 → 查 Bedrock console 模型卡／ListFoundationModels／设定档文档
```

**Bedrock 上的 Claude Code 提醒（概念）：** 若一个 agentic 程序工具默认使用跨区域配置文件，单一区域驻留可能需要创建应用程序配置文件，并把工具指向那个配置文件 ID——AWS 公开博客把这点列为常见的企业陷阱。

---

## 6. 通过 Bedrock 使用 Claude 功能（课程主题）

### 6.1 对话与系统提示词

与 Anthropic Messages API 同一套想法：角色交替；系统提示词设置持久行为；保持历史整洁。Bedrock Converse 使用像 `system` 清单与 `inferenceConfig` 这类字段名称。

**历史卫生：** 修剪旧轮次，需要时做摘要，永远不要在循环中途丢掉不成对的工具结果。

### 6.2 提示工程与评测

课程重点：结构化提示、系统化测试集、**基于模型的评分**与**基于代码的评分**。考试：评测是一个工作流程（案例 → 执行 → 评分 → 迭代），不是一次性的感觉检查。

| 评分风格 | 它检查什么 | 强项 |
| --- | --- | --- |
| 基于代码 | 精确匹配、正则表达式、JSON schema、单元断言 | 确定性、便宜 |
| 基于模型 | 由另一次模型调用依评分量规打分 | 对开放答案有弹性 |
| 人工抽查 | 样本审查 | 校准 |

### 6.3 工具使用

用 JSON Schema 定义工具；Claude 回传工具调用；你的应用程序执行，并在下一轮回传工具结果。支持多轮与批量工具调用模式。在 Converse 上，工具住在 `toolConfig` 下；只有原生才有的 Anthropic 工具变体，可能需要 InvokeModel／额外字段。

**工具循环正确性：**

1. 模型回传带工具使用区块的助理消息
2. 应用程序跑工具
3. 应用程序以要求的角色／形状附加工具结果
4. 带完整历史再次调用模型
5. 重复直到最终文本答案或停止条件

永远不要捏造工具结果。永远不要随手重排 tool_use／tool_result 配对。

### 6.4 RAG

大纲里的生产环境 RAG 主题：

- 切块策略
- 向量嵌入（常常是经由 **Invoke**、而不是 Converse，调用另一个 Bedrock embedding 模型）
- 词汇搜索（**BM25**）＋向量混合
- 多索引架构、重排序
- **上下文化检索**（嵌入／索引前，用上下文丰富切块）

知道*为什么*混合搜索存在（精确关键字对上语义匹配），胜过任何单一函数库。

Bedrock 知识库可能出现在以 AWS 为中心的题目——仍然对应同一套 RAG 概念：摄取、切块、嵌入、检索、生成、引用来源。

### 6.5 进阶功能

- **扩展思考／推理** — 额外 token 与成本。现行模型使用**自适应思考**（模型决定何时／想多深；深度用 **effort 等级**调节）——固定的 `budget_tokens` 思考预算在 4.6 **已弃用**，在较新的前沿模型上**已移除（400 错误）**；使用任一种形状前，核对 Bedrock 上该模型的模型卡片
- **视觉** — 图像内容区块
- **提示词缓存** — 当模型／平台支持时，为稳定前缀降低成本／延迟
- **流式** — 长答案有更好的用户体验
- **结构化提取** — 约束输出（schema／仔细的提示）；temperature **只适用于较旧模型**——现行前沿模型已移除采样参数，所以 schema 才是确定性杠杆

### 6.6 MCP 与 agent

MCP 跨客户端／服务器标准化工具、资源与提示。Agent 一节把以 Bedrock 为后端的 Claude 接到自动化模式（Claude Code、电脑操作）。备考：MCP 是**模块化暴露工具／资源的协议**，不是 Bedrock 专属 API。

Agent 模式仍然适用：平行化、链式、路由，用追踪调试。
---

## 7. 比较快照：Bedrock 对上 Anthropic API 对上 Vertex

| 主题 | Anthropic API | Amazon Bedrock | Google Vertex（Claude） |
| --- | --- | --- | --- |
| 验证 | Anthropic API 密钥 | AWS IAM／凭证 | GCP ADC／服务账户 |
| Client | `Anthropic` | `bedrock-runtime`（Converse／Invoke） | `AnthropicVertex` |
| 模型 ID 风格 | Anthropic ID | `anthropic.…`／`us.anthropic.…` | Vertex 发布者 ID（例如 `claude-sonnet-4-6`） |
| 版本字段 | 标头／SDK | Invoke body 里的 `anthropic_version`；Converse 抽象掉很多 | 原始 HTTP 上的 `anthropic_version: vertex-2023-10-16` |
| 统一多模型聊天 | Messages | **Converse**（Bedrock 全局） | 经由 Vertex 后端的 Messages |
| 计费／合约 | Anthropic | AWS | Google Cloud |
| 驻留旋钮 | Anthropic 政策 | 区域＋推理配置文件 | global／us／eu／区域端点 |
| 启用 | API 密钥访问 | 按账号／区域的 Bedrock 模型访问 | Model Garden 启用 |

**选 Bedrock，当：** 以 AWS 为中心的 IAM、跨 Bedrock 模型的 Converse 可携性、AWS 合规边界、既有的 AWS 网络／VPC 模式。

**选 Vertex，当：** GCP 是控制平面、Google 计费／驻留、Model Garden 合作伙伴路径。

**选 Anthropic API，当：** 通往最新 Anthropic 专属功能的最快路径，以及最简单的密钥验证。

**不要跨云端混用 ID 格式**——光这一点就能回答好几道考试干扰选项。

---

## 8. 考试陷阱（Bedrock 专属）

| 陷阱 | 现实 |
| --- | --- |
| 找 `bedrock:Converse` 权限 | Converse 用 `bedrock:InvokeModel` |
| 需要配置文件时用基础模型 ID | 改成推理配置文件 ID／ARN |
| 只授予配置文件 ARN | 地理配置文件还要授予目的地 FM ARN |
| 假设历史被存储 | 无状态；重送 messages |
| 用 Converse 做向量嵌入 | 分开 Invoke embedding 模型 |
| 单一区域政策却用跨区域配置文件 | 错误的驻留工具；用应用程序配置文件钉住 |
| 把 Vertex 模型 ID 拷贝进 Bedrock | 不同的目录字符串 |
| 已授予 Invoke，流式却 AccessDenied | 缺少 `InvokeModelWithResponseStream` |
| 「全局启用一次」 | 启用是按账号／区域模式 |
| 题目要求 Guardrails 时把它当可选 | 被问及时，在 Converse 上接上 Guardrails 设置 |

---

## 9. 考试提示

- **同时**启用模型访问，并使用模型存在的区域／配置文件。
- 除非题目强迫用原生 Invoke，否则优先 **Converse**。
- 记住**无状态**历史管理。
- Converse 的 IAM ≈ `bedrock:InvokeModel`（＋流式变体）。
- 较新模型 → 查**推理配置文件** ID。
- RAG 答案应提到切块＋检索质量，而不是生产环境「就把 PDF 塞进提示词」。
- 评测 = 系统化评分，不是单一轶事提示。
- 区分建议式提示与工具／MCP 管线。
- 驻留方面，明确说出配置文件策略。
- 工具循环方面，坚持正确的消息顺序。

---

## 10. 最小练习草图（概念）

1. 在你的账号／区域，于 Bedrock console 启用 Claude。
2. 用 AWS 凭证创建 runtime client。
3. 用推理配置文件模型 ID 与一则用户消息调用 `converse`。
4. 附加先前的助理输出，加上第二轮。
5. 经由 `toolConfig` 加一个小工具（例如计算器），完成一次工具循环。
6. 用 `converse_stream` 流式同一个提示词。
7. 故意用一次错误的区域／模型 ID，认清失败模式。
8. 草拟一份包含配置文件＋目的地 FM ARN 的 IAM 政策。
9. 写五个评测案例，用代码断言＋一份模型评分量规评分。
---

## 11. 自我检核问答（附答案）

**Q1.** Converse 调用以 AccessDenied 失败，但昨天同一个角色的 Invoke 是成功的。可能原因？
**A.** 缺少 `bedrock:InvokeModel`（或新模型／配置文件的资源 ARN）——Converse 使用同一 invoke 权限家族。若只有流式失败，查流式动作。

**Q2.** 目录里的模型 ID 在文档上可用，在你的区域却失败。下一步？
**A.** 确认区域可用性；改成正确的**推理配置文件** ID 做跨区域路由。

**Q3.** 为什么 Claude 在 Bedrock 上不记得上一轮？
**A.** Runtime 是无状态的；你必须重送完整消息历史。

**Q4.** 什么时候为 Claude 选 InvokeModel 而不是 Converse？
**A.** 需要 Converse 没有干净暴露的原生 Anthropic 请求／响应字段或功能；或非聊天模型（向量嵌入等）。

**Q5.** Converse 的 `inferenceConfig` 里放什么？
**A.** 共享旋钮如 `maxTokens`、`temperature`、`topP`——Claude 专属额外项常常放进 `additionalModelRequestFields`。

**Q6.** 说出两个超越天真 top-k embedding 搜索的 RAG 改善。
**A.** BM25 混合搜索；重排序；上下文化检索；更好的切块／多索引。

**Q7.** MCP 是什么，一句话？
**A.** 一套协议，用来以模块化、可重用的方式，向 AI 客户端暴露工具、资源与提示。

**Q8.** 如何保持多轮工具使用正确？
**A.** 依序附加助理工具调用消息与用户／工具结果消息；永远不要在循环中途丢掉结果。

**Q9.** 跨区域配置文件 invoke：IAM 只允许配置文件 ARN。仍然 AccessDenied。为什么？
**A.** 还需要对配置文件路由到的每个目的地区域里的基础模型 ARN 有 invoke 权限。

**Q10.** ConverseStream 需要哪个 IAM 动作？
**A.** `bedrock:InvokeModelWithResponseStream`（不只 `InvokeModel`）。

**Q11.** 较新、需要配置文件的 Claude 模型，如何追求单一区域驻留？
**A.** 创建指向单一区域基础模型 ARN 的应用程序推理配置文件；避免多区域地理前缀。

**Q12.** 为什么同一个应用程序里，向量嵌入用 Invoke、聊天用 Converse？
**A.** 向量嵌入不是对话式 Converse 工作负载；Invoke 才是对的模态 API。

**Q13.** Bedrock Claude Invoke body 的版本字段通常设成什么？
**A.** `anthropic_version`，例如 `bedrock-2023-05-31`（确认现行文档）。

**Q14.** 基于代码对上基于模型的评测——何时优先基于代码？
**A.** 当成功可以用客观方式检查时（JSON schema、精确字段、单元断言）。

**Q15.** 说出与 Vertex Claude 的三个差异。
**A.** AWS IAM 对上 GCP ADC；`anthropic.`／地理配置文件 ID 对上发布者 ID；Converse 对上 Vertex 上的 Messages。

**Q16.** Converse 上的 Guardrails——概念角色？
**A.** 设置后，套用在模型调用周围的平台安全／过滤——不是应用程序授权的替代品。

**Q17.** 批量工具调用在高层是什么意思？
**A.** 模型可以在一轮请求多个工具；应用程序在下一次模型调用之前回传多个结果。

**Q18.** 提示词缓存在什么时候帮助最大？
**A.** 跨调用重用的稳定长前缀（系统、工具、大型文档）——当该模型／平台支持时。

**Q19.** 你在 us-east-1 启用了 Claude，却调用 eu-west-1。会怎样？
**A.** 失败或模型不可用——启用与可用性对区域敏感；改区域或使用适当配置文件。

**Q20.** 上下文化检索加了什么？
**A.** 在嵌入／索引前，用周围文档上下文丰富切块，以改善检索质量。

**Q21.** 先分类再送到专门提示／工具的 agent 模式？
**A.** 路由。

**Q22.** 为什么生成工具参数时 temperature 通常较低？
**A.** 需要更确定性的结构化参数，才能可靠执行。

**Q23.** 应用程序推理配置文件的主要好处？
**A.** 成本归属／标签，与／或钉住路由行为（包括单一区域来源），对上系统地理配置文件。

**Q24.** 经典 Bedrock Runtime 是用 Anthropic API 密钥计费吗？
**A.** 不是——经典 Bedrock Runtime 的 Claude 调用用 AWS 凭证与 AWS 计费。
---

## 12. 复习检查清单（考前）

- [ ] 启用对上凭证对上正确区域，三者都分得清
- [ ] 基础模型 ID 对上地理配置文件对上应用程序配置文件
- [ ] 背下 Converse 对上 Invoke 的决策标准
- [ ] IAM：Converse 用 InvokeModel；流式用流式动作
- [ ] 配置文件 IAM 包含目的地 FM ARN
- [ ] 无状态多轮历史
- [ ] toolConfig 循环顺序
- [ ] RAG 混合／重排序／上下文化检索词汇
- [ ] 评测 = 系统化案例＋代码／模型评分
- [ ] 驻留决策树（global／地理／单一区域钉住）
- [ ] Bedrock 对上 Vertex 对上 Anthropic API 比较表
- [ ] 流式事件重组的概念理解

---

## 13. 词汇表

- **Bedrock Runtime** — 调用基础模型的 API 面（`bedrock-runtime` client）。
- **Converse** — 跨 Bedrock 聊天模型的统一对话推理 API。
- **ConverseStream** — Converse 的流式变体。
- **InvokeModel** — 模型原生 body 的调用 API。
- **推理配置文件** — 路由推理的资源（系统地理或应用程序）。
- **基础模型（FM）ARN** — 某个区域里底层的模型资源。
- **应用程序推理配置文件** — 客户自建、用于归属与／或钉住路由的配置文件。
- **地理配置文件前缀** — 例如配置文件 ID 上的 `us.`、`eu.`、`global.`。
- **additionalModelRequestFields** — Converse 上模型专属参数的逃生口。
- **inferenceConfig** — Converse 上共享的采样／max token 旋钮。
- **toolConfig** — Converse 的工具定义与工具选择设置。
- **无状态推理** — 调用之间没有服务器端聊天记忆。
- **BM25** — 混合 RAG 里使用的词汇排序函数。
- **上下文化检索** — 嵌入／索引前把上下文加在切块前面。
- **MCP** — 工具／资源／提示的模型上下文协议。
- **Guardrails** — 可设置在调用周围的 Bedrock 安全过滤。
- **知识库（Knowledge Base）** — AWS Bedrock 生态系统里受管的 RAG 检索组件。
- **基于模型的评分** — 用模型依评分量规为输出打分。
- **基于代码的评分** — 对输出做程序化断言。
- **AccessDeniedException** — Bedrock API 的 IAM 授权失败。

---

## 14. 更深的决策树

### 14.1 API 形状选择器

```text
工作负载是带选配工具的对话式消息吗？
 是 → Converse / ConverseStream
 否 → 是矢量嵌入／影像／仅原生才有的 payload 吗？
 是 → InvokeModel / 串流变体
 否 → 重新查阅模态文档
```

### 14.2 失败分诊

```text
AccessDenied？
 → 检查动作（Invoke 对上 InvokeWithResponseStream）
 → 检查资源 ARN（设定档 + 目的地基础模型）
 → 检查账号的模型启用

验证错误／模型不存在／不支持随需？
 → 区域错误，或需要推论设定档 ID

奇怪的空答案／部分答案？
 → maxTokens 太低；停止原因；工具循环未完成

工具循环空转？
 → 工具结果缺少／格式错误；max 轮次无上限
```

### 14.3 生产环境就绪迷你评分量规

1. 驻留与可用性用对配置文件
2. 用拒绝案例测过的最小权限 IAM
3. 流式与非流式的超时／重试
4. 提示微调前，有固定案例的评测套件
5. 日志不泄漏密钥
6. 对 token 与配置文件标签的成本监控
7. 文档化的回退模型或区域策略

---

## 15. 共享技能主题对应到 Bedrock 字段

| Claude 技能主题 | Bedrock 表达 |
| --- | --- |
| 系统提示词 | Converse 上的 `system` |
| Temperature | `inferenceConfig.temperature` |
| 工具 | `toolConfig`（或 Invoke 上的原生工具） |
| 多轮 | 应用程序管理的 `messages` 数组 |
| 流式 | `converse_stream`／invoke 流式 |
| 结构化输出 | 仔细的提示＋schema 验证；平台有提供时用平台功能 |
| 视觉 | messages 里的图像内容区块 |
| 缓存 | 可用时的模型／平台专属缓存字段 |
| Agent | 你的编排＋可选的、以 Bedrock 为后端的 Claude Code |

---

## 16. 读书节奏

第 1 天：第 1–5 节（访问、Converse／Invoke、IAM、驻留）。凭记忆画比较表。
第 2 天：第 6–10 节（功能、陷阱、练习草图）。大声走失败分诊。
第 3 天：全部问答闭书；检查清单；只扫词汇表缺口。

**口诀：** 启用、配置文件、IAM、默认 Converse、历史是你的、驻留是故意的。

---

*对齐 https://academy.claude.com/courses/claude-with-amazon-bedrock。正式使用前，在 AWS Bedrock 文档核对现行模型 ID、区域与 IAM 例子。*
---

## 17. 走完的迷你情境（考试风格）

**情境 1 — 生产环境 Lambda 里的新 Sonnet**
团队把旧的基础模型 ID 拷贝进 `converse`。错误说不支持按需吞吐量／请使用推理配置文件。
**答题路径：** 查该 Claude 版本的系统或应用程序推理配置文件 ID；更新 `modelId`；把 IAM 扩到配置文件＋目的地 FM ARN；在 Lambda 区域重测。

**情境 2 — 欧盟数据政策**
法务要求在欧盟处理。工程师用 `global.…` 配置文件，因为教程里「刚好能用」。
**答题路径：** 错。欧盟多区域驻留优先用 `eu.` 地理配置文件；若政策要求单一区域，用单一区域的欧盟应用程序配置文件。写清楚你实际拥有哪一种保证。

**情境 3 — 流式聊天小工具**
非流式聊天可用；流式以 AccessDenied 失败。
**答题路径：** 在相同资源上加上 `bedrock:InvokeModelWithResponseStream`；继续在客户端重组流式事件。

**情境 4 — 使用工具的客服 agent**
模型要两个工具；应用程序只回一个结果；下一轮角色混乱。
**答题路径：** 正确配对回传所有工具结果；保住助理的工具使用消息；然后继续。

**情境 5 — RAG「幻觉出来的政策」**
天真的 top-k embedding 检索漏掉精确条款标识符。
**答题路径：** 加上 BM25／混合，考虑重排序，改善切块，试上下文化检索；用含那些条款标识符的固定案例评测。

**情境 6 — 多云迁移**
从 Anthropic API 搬到 Bedrock。工程师把 `x-api-key` 与 Anthropic base URL 贴进 AWS 代码。
**答题路径：** 用 AWS 凭证＋Bedrock Runtime；把 Messages 字段对应到 Converse，或用 Bedrock 的 `anthropic_version` 保住 Invoke；把模型 ID 改成 Bedrock 目录形式。

**情境 7 — 成本归属**
财务看不出哪条产品线花了 Bedrock Claude token。
**答题路径：** 按产品加上标签的应用程序推理配置文件；经由那些配置文件 ARN 调用；按标签做仪表板。

**情境 8 — 评测不一致**
提示词在三个精挑细选的例子上看起来很好，在生产环境却默默失败。
**答题路径：** 创建留出评测集；把 schema 的基于代码检查，与质量的基于模型评分量规结合；用评测门槛把关部署。

---

## 18. 速查卡

| 需要 | Bedrock 动作 |
| --- | --- |
| 默认聊天／agent | Converse |
| 流式 token | ConverseStream＋流式 IAM |
| 原生 Anthropic body／向量嵌入 | InvokeModel |
| 跨区域可用性 | `us.`／`eu.`／`global.` 配置文件 |
| 单一区域钉住 | 应用程序配置文件 → 一个 FM ARN |
| 验证 | AWS 凭证链，不是 Anthropic 密钥 |
| 工具 | `toolConfig`（＋循环） |
| 只有 Claude 才有的旋钮 | `additionalModelRequestFields` |
| 对上 Vertex 的可携 | ID 与验证完全不同 |

**一句话口诀：** 在 Bedrock 上，启用模型，用 Invoke 家族 IAM 调用正确配置文件，优先 Converse，自己掌管历史，并且故意挑选驻留。

---

## 19. 额外问答（延伸）

**Q25.** 同一份 boto3 Converse 代码能给 Claude 和另一个 Bedrock 聊天模型用吗？
**A.** 对消息形状常常可以——那份可携性就是 Converse 的卖点——但模型专属字段与能力仍然不同。

**Q26.** 原生 Invoke Messages 风格的 Claude 漏掉 `max_tokens` 会怎样？
**A.** 请求验证错误——原生 Messages 风格 body 里 `max_tokens` 是必填。

**Q27.** 系统提示词在 Converse 上放哪里，对上误把 `role: system` 放进 messages？
**A.** 用 Converse 的 `system` 参数（系统区块清单）；不要像某些其他 API 那样，在 messages 里发明一个 system 角色。

**Q28.** 为什么有些 IAM 政策列出 `GetInferenceProfile`？
**A.** 依 AWS 推理先决条件文档，以推理配置文件执行推理时需要它。

**Q29.** 混合搜索，一句话？
**A.** 结合词汇（BM25）与向量检索，让精确 token 与语义匹配都能浮出来。

**Q30.** 这门课里的电脑操作／Claude Code 主题——该记住什么？
**A.** 它们是可以跑在以 Bedrock 为后端的 Claude 上的 agentic 自动化模式；仍然需要 AWS 验证、模型访问，以及验证纪律。

---

## 20. 读书收尾

若你能不看笔记重画（1）Converse 对上 Invoke、（2）流式／非流式的 IAM 动作、（3）驻留用的配置文件类型、（4）三云比较表，你就准备好应付 Bedrock 差异化考题了。然后用与 Anthropic API、Vertex 路线相同的词汇，刷新共享的 Claude 技能（工具、RAG、MCP、评测）——改变的只有传输层与 ID 格式。
