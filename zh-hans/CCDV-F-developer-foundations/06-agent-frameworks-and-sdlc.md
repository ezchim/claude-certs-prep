---
title: Agent 框架、托管模式与 SDLC 基础
pack: CCDV-F 开发者基础考试
disclaimer: 原创备考笔记 — 独立内容，非正式课程教材
approx_length: 约 2500–4000 字（P0 缺口补齐）
updated: 2026-08-23
---

# Agent 框架、托管模式与 SDLC 基础

> **免责声明：** 原创备考笔记，用来补上已公布 CCDV-F 考试蓝图的缺口：具名 agent 框架词汇、自架对 Anthropic 托管式 agent、软件工程／系统生命周期基础，以及一段简短的 WebSocket 说明。框架 API 与产品名称会漂移——**考前请对照现行官方文档查证**。

**主要对应：** Agent 与工作流程 **14.7%**（模式／框架约 4.9%；构建／托管约 5.3%）· 应用与集成 **33.1%**（软件工程基础约 7.4%；系统生命周期约 2.8%；技术基础／SDK 优先于 REST／流式传输层约占 6.1% 中的一部分）。
**次要：** Claude Code（3.1%）作为 agent 主机 · 题干出现托管与审查关卡时的评测／安全。
**配套章节：** [02](./02-production-prompting-agents-tools.md)（agent 循环、工具）· [03](./03-claude-code-mcp-integration.md)（API／流式／模型上下文协议（MCP））· [04](./04-production-engineering-evals-security.md)（持续集成（CI）关卡、审查、部署）。

---

## 1. 概览 — 为什么有这一章

第 02–04 章已经教过**模式**：工作流程对 agent、预算、验证器、Messages API、Claude Code、MCP、评测、安全。公开考试指南的措辞还期望**具名识别**，以及对应到 Claude 交付的经典**软件工程素养**：

1. 在词汇层级认得 **Strands**、**LangGraph** 和 **PydanticAI**（大致知道各自用途——不是教学课程）。
2. 用一张干脆的决策表，在**自架 Agent SDK／自订循环**与 **Anthropic 托管式 agent** 之间做选择。
3. 回答那些假设你熟悉 REST／JSON／异步、版本控制、代码审查、重构与系统生命周期阶段的集成题干——Claude Code／API 是适配点，不是软件工程教科书。
4. 知道 **WebSocket** 相对于惯用 Claude 流式路径（**服务器推送事件（SSE）**）的位置。

考试形状：挑对**控制模型**与**拥有权边界**，不是凭记忆实现一个框架。

---

## 2. 关键地图（考试识别）

| 名称／模式 | 大致工作 | 考试线索 |
| --- | --- | --- |
| **自订循环** | 你自己写 `while not done: model → tools → results` | 最大控制；每一种失败模式都由你承担 |
| **Claude Agent SDK** | Anthropic 的 harness（精神与 Claude Code 相同）跑在**你的**程序里 | Claude 优先；MCP／子代理／权限主题 |
| **Anthropic Managed Agents** | 托管的 REST 形状 agent 执行环境；Anthropic 运行 harness／沙箱／会话 | 运营快；数据驻留与执行环境费用的取舍 |
| **LangGraph** | 明确的**图**编排（节点／边、状态、检查点） | 可审计性、人在回路中（HITL）中断、确定性工作流程外皮 |
| **Strands（AWS）** | **模型驱动**的 agent 循环；与 AWS／Bedrock 高度契合 | 跨模型可移植性；OpenTelemetry（OTel）；AWS 原生部署 |
| **PydanticAI** | **类型化** agent／结构化且经验证的 I/O（Pydantic 优先） | schema 保证；较轻的编排 |
| **SSE 流式** | Claude Messages 的默认流式（经 HTTP 从服务器到客户端） | 聊天使用体验、首个 token 延迟（TTFT）、工具参数增量 |
| **WebSocket** | 双向应用传输层（你的 UI ↔ 你的后端） | 中断／HITL／语音——**不是** Claude API 的主要流式线路 |

---

## 3. 具名框架词汇（考试识别，不是教学课程）

> **请对照现行文档查证：** 星数、版本号、云端 SKU（产品型号）与确切类别名称都会变。背下**角色与取舍**，考前再浏览现行 README／文档。

### 3.1 它们与工作流程对 agent／类型化工具／图编排的关系

CCDV-F 已经反复操练**工作流程对 agent**（第 02 章）：已知步骤 → 确定性工作流程／有向无环图（DAG）；带工具的开放式目标 → 带预算的 agent 循环。具名框架是**那些形状的实现**：

| 关注点 | 优先认出…… |
| --- | --- |
| 固定或可审计的多步骤流程，带循环／HITL | **LangGraph**（图编排） |
| 「给模型＋工具＋提示词，让循环自己跑」，且偏 AWS | **Strands**（模型驱动 agent） |
| Python 服务里有保证的结构化输出／类型化工具参数 | **PydanticAI**（类型化工具与验证） |
| 在你自己基础设施内的 Claude 原生写程序／agent 产品循环 | **Claude Agent SDK** |
| 最小接口；完全拥有权；教学考试循环 | **自订 Messages API 循环** |

它们在生产环境中**并不互斥**：团队常把 Pydantic 验证放进 LangGraph 节点里，或从 Strands 经 Bedrock 调用 Claude，或把 Agent SDK 会话包在应用程序 WebSocket 后面。考试题干通常问的是，哪一个**主要**控制模型符合该限制。

### 3.2 Strands（AWS Strands Agents）— 高层

- **大致用途：** 模型驱动的 agent 框架（Apache 授权的开源血统），为生产循环优化，具有一流的 **AWS／Bedrock** 集成主题、内置可观测性（常见为 OpenTelemetry），以及公开材料中的多 agent 原语（集群／图／交接风格模式）。
- **与考试概念的关系：** 比较接近 **agent 循环**，而不是手绘的工作流程 DAG。你声明模型、工具、提示词；框架拥有大部分回合机制。
- **对 Claude Agent SDK／自订循环的取舍：**
 - **赢面：** 模型可移植性（不只限 Claude）；AWS 部署故事；较少自己动手的追踪胶水。
 - **代价：** 又多一个框架接口要学；不是与 Claude Code 同一个 harness；第一次运行之前，通常要先做 Bedrock／身份与访问管理（IAM）设置。
 - **考试句子：** 「AWS 原生、不用重写循环就能换模型」→ Strands 形状的答案——不是「必须用 Claude Agent SDK」。

### 3.3 LangGraph — 高层

- **大致用途：** 来自 LangChain 生态系统的图形编排：你定义**节点**、**边**、共享**状态**，通常还有**检查点**。当**流程本身**必须可查看时（合规、长工作流程、明确的人为中断），适配力强。
- **与考试概念的关系：** 体现**工作流程／图编排**。模型在节点*内部*行动；控制流是你的。当「审计每一次转换」胜过「最大自主程度」时，它是理想选择。
- **对 Claude Agent SDK／自订循环的取舍：**
 - **赢面：** 明确控制、HITL 中断、成熟生态系统集成、持久状态模式。
 - **代价：** 更多编排代码／概念；模型中立（你要自己接 Claude）；对简单的纯 Claude agent，比薄的 Agent SDK 查询循环更重。
 - **考试句子：** 「有状态的多步骤，加上强制人工核准关卡与可重播状态」→ LangGraph 形状——不是无边界的 agent 乱冲。

### 3.4 PydanticAI — 高层

- **大致用途：** Python 优先、**类型安全**的 agent／工具层，围绕 Pydantic 模型打造——强调**经验证的结构化输出**与类型化工具参数，胜过重型多 agent 编排。
- **与考试概念的关系：** 放大**类型化工具**与**输出合约**（第 02／03 章）。通常是较大系统里面的*一层*，而不是整个编排故事。
- **对 Claude Agent SDK／自订循环的取舍：**
 - **赢面：** schema／验证文化；在副作用发生前拦下坏的工具参数；「单一 agent＋工具」服务的低锁定。
 - **代价：** 主要不是图编排器；多 agent／图的需求可能把你推向再叠 LangGraph／Strands／Agent SDK。
 - **考试句子：** 「Python 服务绝不能把未验证的 JSON 写进总账」→ PydanticAI／schema 优先——写入仍要搭配权限。

### 3.5 取舍卡 — 框架对 Claude Agent SDK 对自订循环

| 选项 | 何时最好 | 何时弱 |
| --- | --- | --- |
| **自订 Messages 循环** | 教学清晰度；微型 agent；独特控制需求 | 你会重新发明会话、压缩（上下文压缩）、权限、追踪 |
| **Claude Agent SDK** | 只限 Claude；想要像 Code 的 harness（工具、MCP、子代理）跑在**你的**程序里 | 需要非 Claude 模型；无法在自己基础设施里执行？→ 考虑托管 |
| **Managed Agents（Anthropic 托管）** | 想让 Anthropic 运行 harness／沙箱／会话；快速交付 | 严格数据驻留／仅限虚拟私有云（VPC）的工具执行；需要特殊循环控制 |
| **LangGraph** | 流程本身就是产品；检查点；HITL 图 | 简单单工具问答（杀鸡用牛刀） |
| **Strands** | 以 AWS／Bedrock 为中心；模型可移植的 agent 循环 | 纯 Anthropic 技术栈、且偏好与 Code 对等的 harness |
| **PydanticAI** | Python 里类型化 I/O 保证 | 复杂多方编排才是主要问题 |

**反陷阱：** 点名一个热门框架永远不够。题干仍然期望**预算、最小权限原则、评测与停止条件**（第 02／04 章）。

---

## 4. 自架对 Anthropic 托管的 Managed Agents — 决策表

公开的 2026 年产品划分（请即时查证）：**Claude Agent SDK**＝**你的**程序里的函数库；**Claude Managed Agents**＝托管的 agent 执行环境（REST／事件 API 主题），由 **Anthropic** 运行 harness、沙箱与持久会话机制。底下是同样的 Claude 模型；不同的是**运营拥有权**。

| 维度 | 自架（Agent SDK／自订循环／你的 worker） | Anthropic 托管的 Managed Agents |
| --- | --- | --- |
| **谁运行循环** | 你 | Anthropic |
| **工具／沙箱在哪里执行** | 你的基础设施（VPC、笔记本、Kubernetes（k8s）） | Anthropic 管理的沙箱（VPC connector 可能存在——请查证文档） |
| **会话／宕机恢复** | 你自己建或采用现成的 | 平台的责任 |
| **数据驻留压力** | 控制更强（若你那样设计，只有推理离开边界） | 会话日志＋沙箱在供应商那一侧 → 合规审查 |
| **成本形状** | token ＋**你的**运算／运营 | token ＋**执行环境／会话**费用（有公开报道；请查证） |
| **定制化深度** | 最高（自订重试、特殊工具、实体隔离（air-gapped）模式） | 产品接口高、基础设施自己动手较少；特殊循环可能不合 |
| **到生产环境的时间** | 若缺乏 harness 成熟度则较慢 | 若限制允许托管执行则较快 |
| **与 Claude Code 对等** | Agent SDK 目标是在本地／程序内提供像 Code 的 harness | 托管＝服务形状；仍是 Claude 家族 agent |
| **典型考试选择** | 受监管数据、自订权限、既有机群、你已在运营的极端并行成本 | 异步长时间运行 agent、小团队、不想拥有沙箱 |

```text
真的需要 agent 循环吗？
 否 → Messages API（＋工具）就够了
 是 → 工具／状态必须留在你的边界内吗（数据驻留、VPC、自订沙箱）？
 是 → 自架 Agent SDK 或自订循环（＋可选 LangGraph／Strands／PydanticAI）
 否 → 交付速度／托管沙箱值得执行环境费用吗？
 是 → Anthropic 托管的 managed agents
 否 → 仍然自架（成本或控制偏好）

原型路径（常见的公开指引主题）：本机 Agent SDK → 营运痛点主导时改托管——搬动机密／PII 之前仍要重新检查合规。
```

**考试陷阱**

1. 把「用 Claude」等同于「必须用 Managed Agents」。
2. 把「Agent SDK」等同于「Anthropic 托管我的沙箱」。
3. 忽略 **Message Batches（批量 API）≠ 托管式 agent**（批量＝异步模型调用；不是你本地的工具循环）。
4. 忘记 **hooks／权限／评测** 在两种模式下照样适用。

---

## 5. 软件开发生命周期（SDLC）／软件工程基础中的 Claude（考试配分卡）

已公布蓝图配分让**软件工程基础（约 7.4%）**与**系统生命周期（约 2.8%）**成为该读的材料——不是「假设的背景知识」。当作**对应到 Claude 的素养**来读，不是软件工程课程。

### 5.1 REST／JSON／异步（技术基础）

| 概念 | 考试有用的 Claude 对应 |
| --- | --- |
| **REST** | 表现层状态转换（REST）。Messages API 是以 HTTP 资源为导向的；SDK 包装 REST。优先用官方 SDK 取得重试／流式辅助；概念上要知道标头（`x-api-key`、API 版本）。 |
| **JSON** | 工具参数、结构化输出、MCP 载荷、日志。在服务器端验证；绝不要只信任模型的 JSON 就做写入。 |
| **异步** | （1）你应用程序里的并行工具调用／`asyncio` worker；（2）脱机模型工作用的 Message Batches；（3）托管／长时间运行的 agent 会话。不要把三者混为一谈。 |
| **SDK 优先于 REST** | SDK 是默认的生产选择；调试线路格式或没有 SDK 的语言时才用裸 REST。 |

### 5.2 版本控制、代码审查、重构

| 实践 | Claude 适配何处 | 考试判断 |
| --- | --- | --- |
| **版本控制系统（VCS，git）** | 权威状态活在提示词之外；分支／pull request（PR）是变更的单位 | agent 应在分支上提交；绝不要把强制推送到 `main` 当作默认自主权 |
| **代码审查** | Claude Code／PR agent 起草；人类（或政策机器人）拥有合并关卡 | 高风险：密钥、授权（authZ）、迁移、不可逆脚本 → 必须人工审查 |
| **重构** | 计划模式（plan mode）→ 测试优先 → 小差异 → 验证 | 偏好有测试支撑的重构；无边界的「清理整个仓库」agent 在题干里失败 |

**一句话：** Claude 加速撰写；**版本控制＋审查＋CI** 仍然是控制平面。

### 5.3 系统生命周期阶段（经典标签 → Claude 交付）

| 阶段 | Claude／CCDV 适配 |
| --- | --- |
| **需求／探索** | 待完成工作（JTBD）、限制（延迟、成本、安全）、成功指标——在挑模型之前 |
| **设计** | 工作流程对 agent；工具／MCP 边界；托管模式；schema；威胁模型 |
| **实现** | API 应用程序、Agent SDK／Code、作为版本化构件的提示词、pin bundle（绑定版本组） |
| **验证／评测** | 黄金标准集、回归评测框架、副作用失败测试（第 04 章） |
| **部署** | 金丝雀、功能标志、受管理设置、紧急停止开关 |
| **运营／监控** | 追踪、每次成功成本、缓存命中率、事故 runbook（运维手册） |
| **演进／退役** | 提示词／工具版本化、模型迁移、用桩弃用工具 |

**考试句子：** 从炫的展示直接跳到生产环境自主，**跳过**验证与部署关卡——即使模型是 Opus 层级，答案仍是错的。

### 5.4 Claude Code／API 如何适配 CI 与审查

```text
PR 开启
 → CI：单元＋schema／工具合约测试＋评测子集（便宜）
 → Claude Code 无头模式／Agent SDK 工作（钉定模型）在分支上提案或实作
 → 静态检查＋密钥扫描＋权限政策 lint
 → 人工审查（破坏性／授权／数据路径必须）
 → 合并 → 分阶段部署 → 金丝雀评测 → 晋升
```

| 适配点 | 优先选 |
| --- | --- |
| 聊天使用体验的 token | 流式 Messages（经 SDK 的 SSE） |
| 脱机分类／摘要 | Message Batches |
| 仓库本地的 agentic 编辑 | Claude Code／带钉定＋允许清单的 Agent SDK |
| 组织政策 | 受管理设置 > 项目方便 |
| 质量真相 | CI 里的评测框架，不是 Slack 里的感觉 |

---

## 6. WebSocket 简短说明（流式 SDK）

**Claude Messages 流式（供应商 → 你的后端）** 公开文档记载为 **HTTP 服务器推送事件（SSE）**（`text/event-stream` 主题）：`message_start` → 内容区块增量 → `message_stop`。官方 SDK 会为你解析。

**WebSocket** 在 CCDV 形状的题干里，是**应用传输层**的选择：

| 传输层 | 方向 | 与 Claude 应用的典型用途 |
| --- | --- | --- |
| **SSE（Claude API 流式）** | 服务器 → 客户端（单向） | token 流式、TTFT 使用体验、细粒度工具参数预览 |
| **SSE（你的 API → 浏览器）** | 服务器 → 浏览器 | 映射 API 流式的简单聊天接口 |
| **WebSocket（你的应用）** | 双向 | 流式中途用户中断、同一会话上的 HITL 核准、语音、协作 agent |
| **轮询** | 请求／响应 | 批量工作状态——不是聊天 TTFT |

**考试线索**

- 「聊天要更低的 TTFT」→ 流式（SSE 路径），不是为了 WebSocket 而用 WebSocket。
- 「核准一个破坏性工具，而又不能把第二个 HTTP POST 关联到一条已死的 SSE」→ 在 **UI 与你的后端** 之间用 WebSocket（或精心设计的会话）；后端仍然经 SDK／SSE／REST 与 Claude 交谈。
- 远程 MCP 传输层常被描述为 HTTP／SSE 风格——不要把 MCP 线路和浏览器 WebSocket 搞混。

**陷阱：** 宣称 Claude Messages API「是 WebSocket」。在识别层级，**SSE 是默认流式**；WebSocket 通常是**你自己**产品的双向通道。

---

## 7. 决策树（压缩版）

### 7.1 框架／harness 挑选

```text
主要限制？
 只要型别化 Python I/O → PydanticAI（± 薄循环）
 可稽核的图＋HITL 检查点 → LangGraph
 AWS／Bedrock＋模型可携性 → Strands
 我们进程里像 Claude Code 的 harness → Claude Agent SDK
 托管沙箱＋不想自己跑 harness → Managed Agents
 考试教学／独特控制 → 自订 Messages 循环
```

### 7.2 题干里的 SDLC 红旗

```text
没有成功指标 → 修需求
自主程度提高 ↑ 之前没有评测 → 修验证
没有分支／PR → 修版本控制纪律
只用提示词说「永不删除」 → 修权限／hooks
正式环境用展示用模型 ID → 修钉定＋设定管理
```

---

## 8. 考试陷阱

1. **只点名框架而没有停止条件**——仍然是错的。
2. **对单次常见问答用 LangGraph**——杀鸡用牛刀；偏好提示词／工具。
3. **用 Managed Agents 来满足实体隔离的工具执行**——通常应自架。
4. **Agent SDK＝Anthropic 托管我的 VPC**——假的。
5. **任何流式都需要 WebSocket**——假的；SSE 是 API 默认。
6. **批量取代 CI 评测**——假的。
7. **没有测试的重构 agent**——软件工程基础题干失败。
8. **因为是 Claude 写的就跳过代码审查**——审查／安全题干失败。

---

## 9. 自我检核问答（18）

**Q1.** 考试列出 Strands、LangGraph、PydanticAI——每一个各自负责什么识别工作？
**A1.** Strands ≈ 模型驱动（常为 AWS 原生）的 agent 循环；LangGraph ≈ 明确的图／状态编排；PydanticAI ≈ 类型化／经验证的结构化 I/O agent。

**Q2.** 什么时候 LangGraph 比自由 agent 循环是更好的心智模型？
**A2.** 当转换必须可审计、可设检查点，或可由人类中断，作为流程定义的一部分。

**Q3.** 什么时候你会挑 Strands 而不是 Claude Agent SDK？
**A3.** 需要模型可移植性，以及／或 AWS／Bedrock 原生部署／可观测性，胜过与 Claude Code 对等的 harness。

**Q4.** PydanticAI 对「只在提示词里用 JSON 模式」？
**A4.** PydanticAI／schema 验证在代码里强制类型；只靠提示词的 JSON，失败即关闭的可靠性较低。

**Q5.** 自架 Agent SDK 对 Managed Agents——第一个决定性问题？
**A5.** 工具执行与会话构件，必须留在我们的数据驻留／控制边界之内吗？

**Q6.** 用 Managed Agents 会消除评测与允许清单的需求吗？
**A6.** 不会——托管改变运营拥有权，不改变安全或质量义务。

**Q7.** 自订循环对 Agent SDK——考试一句话？
**A7.** 自订＝最大控制／最大责任；Agent SDK＝Claude 导向的 harness，让你不用重建会话／工具／权限的基本件。

**Q8.** 哪一项 REST／JSON 素养最可能出现在 CCDV-F？
**A8.** 构建 Messages 风格的请求、验证 JSON 工具参数、用 SDK 而非脆弱的临时 HTTP、处理流式事件。

**Q9.** 在生产环境题干里，agent 应该怎么和 git 交互？
**A9.** 分支＋PR；尊重受保护的 `main`；不提交密钥；有风险的差异在合并前要审查。

**Q10.** 代码审查关卡相对于 Claude Code 位于哪里？
**A10.** 在 agent 编辑之后、合并／部署之前——尤其授权、数据、破坏性操作；Claude 起草，政策＋人类把关。

**Q11.** 把「系统生命周期」对应到一次 Claude 功能发布。
**A11.** 需求 → 设计（agent 对工作流程、托管）→ 实现（钉定、工具）→ 评测 → 金丝雀部署 → 监控 → 版本化／退役。

**Q12.** Claude API 流式的默认线路格式？
**A12.** HTTP 上的 SSE；SDK 抽象掉事件解析。

**Q13.** 什么时候 Claude 应用里用 WebSocket 是合理的？
**A13.** 双向需求：流式中途取消／中断、单一会话上的 HITL、语音——在客户端与**你的**后端之间。

**Q14.** Message Batches 对托管的长时间运行 agent？
**A14.** 批量＝带限制的异步模型推理；托管式 agent＝托管的 agent harness／会话，带工具／沙箱主题。

**Q15.** 重构题干：一个 agent 一夜之间重写了半个 monorepo（单一存储库）。哪里错了？
**A15.** 缺少 SDLC 控制——范围、测试、小 PR、审查；没有验证的自主。

**Q16.** 软件工程基础里，「异步」的三种不同意思？
**A16.** 应用程序并行；Message Batches；长时间运行的 agent 会话／worker——不要把答案混在一起。

**Q17.** 能把 PydanticAI 类型和 LangGraph 编排结合吗？
**A17.** 可以，概念上——类型化验证活在图节点内；考试仍可能问哪个关注点是主要的。

**Q18.** 在 Agent SDK 上做原型，然后搬到 Managed Agents——合规的陷阱是什么？
**A18.** 工具／沙箱／会话数据的落地可能改变；在晋升承载个人识别信息（PII）的工作负载之前，重新检查政策。

---

## 10. 检查清单

- [ ] 我能不用代码，一句话说清 Strands 对 LangGraph 对 PydanticAI。
- [ ] 我能依数据驻留、成本与控制线索，在自架 Agent SDK／自订循环与 Anthropic Managed Agents 之间做选择。
- [ ] 我知道自订循环／Agent SDK／托管／图框架是不同的拥有权模型。
- [ ] 我能正确把 REST／JSON／异步对应到 Messages、工具、批量与 worker。
- [ ] 我把 git 分支／PR＋代码审查，当作围绕 Claude 编辑的强制控制平面。
- [ ] 我能带着 Claude 适配点走完系统生命周期阶段（设计 → 评测 → 金丝雀 → 运营）。
- [ ] 我能把 Claude Code／API 工作放进 CI，而不跳过风险变更的人工关卡。
- [ ] 我能区分 SSE（API／默认流式）与 WebSocket（双向应用通道）。
- [ ] 无论框架品牌为何，我仍然应用预算、允许清单、hooks 与评测。
- [ ] 考前我会在现行官方文档上查证框架与 Managed Agents 细节。

---

## 11. 词汇表

| 术语 | 备考含义 |
| --- | --- |
| **模型驱动 agent** | 框架拥有工具循环；你提供模型／工具／提示词（Strands 形状） |
| **图编排** | 明确的节点／边／状态（LangGraph 形状） |
| **类型化工具** | 经 schema 验证的参数／结果（PydanticAI 形状／JSON Schema 工具） |
| **自架 agent** | 循环与工具执行在你的程序／基础设施里 |
| **托管式 agent** | 供应商托管的 harness／沙箱／会话执行环境 |
| **SSE** | 服务器推送事件——Claude 惯用流式传输层 |
| **WebSocket** | 全双工应用传输层，用于中断／HITL／语音模式 |
| **SDLC** | 软件开发生命周期／系统生命周期——从需求到退役 |
| **软件工程基础** | REST／JSON／异步、VCS、审查、重构——有考试配分的素养 |
| **pin bundle** | 为 CI 与生产锁定的模型／SDK／提示词／工具版本 |

---

## 12. 若考试问 X → 想 Y

| 若考试问…… | 想…… |
| --- | --- |
| 可审计多步骤 HITL 的具名框架 | LangGraph |
| 以 AWS Bedrock 为中心、可移植的 agent 循环 | Strands |
| 严格的 Python 结构化输出／类型化工具 | PydanticAI |
| 在我们 VPC 里、精神与 Claude Code 相同的 harness | Agent SDK（自架） |
| 不想运营沙箱；可接受托管会话 | Managed Agents |
| 最低层清晰度／特殊控制 | 自订 Messages 循环 |
| 聊天打字效果／TTFT | 经 SDK 的 SSE 流式 |
| 在同一交互会话核准工具 | WebSocket（UI↔后端）＋服务器端关卡 |
| Claude 改了代码——现在合并？ | 先 PR＋审查＋CI 评测 |
| 缺了哪个生命周期步骤？ | 通常是评测或部署关卡 |

---

## 附录 — 章节 → 官方领域

| 领域 | 第 06 章涵盖 |
| --- | --- |
| Agent 与工作流程（14.7%） | 具名框架；托管模式；harness 取舍 |
| 应用与集成（33.1%） | 软件工程基础；SDLC；REST／JSON／异步；CI／审查适配；流式传输层 |
| Claude Code（3.1%） | 无头模式／CI 的 agent 主机；钉定；审查工作流程 |
| 工具与 MCP（10.6%） | 类型化工具重叠；MCP 仍与框架品牌正交 |
| 安全／评测 | 数据驻留托管线索；审查／CI 关卡（指向第 04 章） |

*第 06 章结束。*
