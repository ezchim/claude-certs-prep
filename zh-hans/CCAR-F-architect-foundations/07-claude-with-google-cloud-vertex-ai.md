---
title: 通过 Google Cloud 的 Vertex AI 使用 Claude — 备考笔记（主要来源）
source: https://academy.claude.com/courses/claude-with-google-cloud-s-vertex-ai
disclaimer: 用于备考的原创笔记——不是 Anthropic 或 Google Cloud 官方教材。不是课堂逐字稿。
approx_length: 约 5500–7000 字
deepened: 2026-08-23
cross_check: Google Cloud Vertex AI／Model Garden Claude 公开文档（ADC、端点、模型 ID）
---

# 通过 Google Cloud 的 Vertex AI 使用 Claude — 主要备考笔记

> **免责声明：** 这些是对齐 [Claude with Google Cloud's Vertex AI](https://academy.claude.com/courses/claude-with-google-cloud-s-vertex-ai) **公开** Claude Academy 课程的**原创**备考笔记。它们**不是** Academy 课堂内容转储或课程原文。请在 Google Cloud 与 Anthropic 文档确认现行模型 ID、端点与功能矩阵。完成官方课程仍是小考的真相来源。

**适合对象：** 要通过 Google Cloud Vertex AI／Agent Platform 加入 Claude 功能的开发者。

**先修（课程）：** Python、JSON 基础、有 Vertex AI 访问权的 Google Cloud 项目。

**如何使用：** 平台专属材料（Model Garden 启用、应用程序默认凭证（ADC）、global／us／eu／区域端点、发布者模型 ID、功能对等注意事项）是回报率最高的差异点。共享的 Claude 技能与 Bedrock 路线重叠——要认得它们在 Vertex 的 Messages 形状 client 里的样子。

---

## 1. 课程地图（公开模块）

公开页面（约 66 堂课、多个小考）大致组织成：

1. **用 API 访问 Claude** — 验证、请求、多轮、系统提示词、temperature、流式、结构化输出
2. **提示工程技巧** — 策略、评测、自动化评分
3. **Claude 的工具使用** — 函数调用、多轮工具、批量工具、内置工具
4. **RAG** — 切块、向量嵌入、BM25 混合、多索引、重排序、上下文化检索
5. **模型上下文协议** — 工具、资源、提示；服务器／客户端生命周期
6. **Agent 与工作流程** — 平行化、链式、路由、调试
7. 相关主题：视觉、PDF、引用来源、提示词缓存、Claude Code／电脑操作风格应用

共享的 Claude 技能与 Bedrock 课程大量重叠。对考试来说**真正不同的**是 **GCP 设置、验证、区域／端点、模型 ID 格式，以及对上 Anthropic 与 Bedrock 的 API 差异**。

**主轴：** 在 Model Garden 启用 → ADC／gcloud → 挑端点类别（global／us／eu／区域） → `AnthropicVertex` Messages 调用 → 自己掌管历史 → 工具／RAG／评测 → 知道对上 Anthropic API 的对等落差。

---

## 2. 设置与身份验证

### 2.1 在 Model Garden 启用 Claude

1. 在 Google Cloud Console 打开 Vertex AI／Model Garden（选对项目）。
2. 搜索 **Anthropic**／Claude。
3. 为项目**启用**你需要的模型（若尚未启用）。
4. 确认该模型在你打算使用的位置／端点风格下可用。

跳过启用是经典的「文档上能用、我的项目里失败」。

### 2.2 gcloud 与应用程序默认凭证（ADC）

典型本机设置：

```bash
gcloud init
gcloud auth login
gcloud config set project YOUR_PROJECT_ID
gcloud auth application-default login
```

Anthropic **Vertex** SDK 使用 Google 验证（ADC／服务账户／工作负载身份）。你**不会**为 Vertex 后端的调用传入 Anthropic API 密钥。

生产环境：优先使用带最小权限 Vertex AI 权限的**服务账户**，而不是在服务器上用长效的用户 ADC。在 GKE／Cloud Run 上尽可能使用工作负载身份。

### 2.3 安装 SDK

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

常见涉及的环境变量：项目 id 与区域（`ANTHROPIC_VERTEX_PROJECT_ID`、`CLOUD_ML_REGION` 等，依 SDK 文档为准）。

**验证决策树：**

```text
在本机笔电做原型？
 → gcloud auth application-default login（用户 ADC）
Cloud Run／GKE／GCE 正式环境？
 → 服务帐户 + 工作负载身分／附加的服务帐户
来自 GitHub 的 CI、没有长效金钥？
 → 工作负载身分联合的模式
还想为 Vertex 设定 ANTHROPIC_API_KEY？
 → 错误的产品路径——那是 Anthropic API，不是 Vertex
```
---

## 3. Vertex 上的 Claude API 与 Anthropic API 差在哪里

**Messages** 形状几乎一样，但平台差异很重要：

| 主题 | Anthropic API | Vertex／Agent Platform 上的 Claude |
| --- | --- | --- |
| 验证 | `x-api-key`／Anthropic 密钥 | Google Cloud 凭证 |
| 模型位置 | 在请求 body 的 `model` | 原始 HTTP 常常在 **URL 路径**；SDK 仍接受 `model=` |
| 版本标记 | API 版本标头 | 原始 HTTP 上 body 字段 `anthropic_version` = `vertex-2023-10-16` |
| 端点 | `api.anthropic.com` | `…-aiplatform.googleapis.com` 或多区域／global 主机 |
| 计费／条款 | Anthropic | Google Cloud 合作伙伴模型条款 |
| 某些平台功能 | Batches、Files、某些服务器工具等 | 查「支持／不支持」清单——它们不同 |

**原始 HTTP 提醒：** 模型放在发布者路径
`/v1/projects/{PROJECT}/locations/{LOCATION}/publishers/anthropic/models/{MODEL_ID}:rawPredict`
（适用时流式用 streamPredict）
而 JSON body 包含 `anthropic_version`、`messages`、`max_tokens`、……

**SDK 提醒：** `AnthropicVertex` 让你继续用熟悉的字段写 `messages.create(...)`，由它处理 Vertex 路由。

**对等注意事项口诀：** 同一个 Claude 家族，功能面不一定相同。若题目说「在 Vertex 上」，不要假设每一个 Anthropic 专属的管理／files／batches／服务器工具功能都存在。

---

## 4. 区域与端点（高价值考试主题）

Agent Platform／Vertex 提供三种端点风格（公开 Google／Anthropic 文档）：

### 4.1 Global

- `region="global"`
- 在有容量的受支持区域之间，以可用性为目标动态路由
- 通常是**标准定价**（没有驻留溢价）
- 当数据驻留有弹性时，最好的默认
- 随用随付导向；预置吞吐量需要区域容量

概念上使用 global 的 AI Platform 端点（现行主机字符串以文档为准）。

### 4.2 多区域（`us` 或 `eu`）

- 在该地理范围内路由
- 常被引用的范例主机：`aiplatform.us.rep.googleapis.com`／`aiplatform.eu.rep.googleapis.com`
- **地理**粒度的数据驻留，可用性又比单一区域好
- 相对于 global 通常有**定价溢价**（常被引用的数字是，4.5 世代起的新模型约 10%——以定价页为准）

### 4.3 特定区域（例如 `us-east5`、`europe-west1`）

- 严格的单一区域路由
- 硬性驻留、某些企业控制、**预置吞吐量**需要它
- 相对于 global 通常也有溢价
- 模型可用性可能落后；最新模型可能先出现在 global／关键区域

**考试决策流程：**

```text
1. 落地有弹性 → global
2. 必须大致待在美国或欧盟 → us / eu
3. 必须钉住单一区域或使用布建吞吐量 → 特定区域
4. 永远核实你选的 Claude 版本在该位置已启用且可用
```

**考试陷阱：** 合规题要求欧盟驻留时用了 `global`——可用性不等于驻留。

**考试陷阱：** 以为 global 有预置吞吐量——公开数据强调区域端点上的预置容量。

---

## 5. Vertex 上的模型 ID

Vertex 发布者 ID 看起来像 Anthropic 名称，但遵循 **Google 的目录字符串**，例如（示意——永远查现行文档）：

- `claude-sonnet-4-6`
- `claude-haiku-4-5@20251001`（有些 ID 含日期后缀）
- `claude-opus-4-6`

它们**不是** Bedrock 的 `anthropic.claude-…` 或 `us.anthropic.…` 推理配置文件字符串，也可能和 Anthropic API 别名略有不同。

合作伙伴平台上的生命周期（已弃用／退役）日期，可能和 Anthropic 自家时间线不同——考试答案应说「查 Google 的 Claude 模型页面」，不要捏造日期。

**ID 比较速记：**

| 平台 | 范例样式 |
| --- | --- |
| Anthropic API | Anthropic 模型别名／带版本的 ID |
| Bedrock | `anthropic.claude-…` 或 `us.anthropic.claude-…` |
| Vertex | `claude-sonnet-4-6` 发布者 ID |

跨云端拷贝粘贴是故意设计的干扰选项。
---

## 6. 功能支持（可以期待什么）

Vertex Claude 上通常可用（按模型确认）：

- Messages API（聊天）
- 工具使用（包括几种 Anthropic 工具类型，但有注意事项）
- 提示词缓存、有提供时的思考／扩展推理
- 视觉／文档（留意**载荷大小上限**——Vertex 可能把请求大小限制在例如几十 MB）
- 有文档记载时的引用来源、结构化输出
- 经由 SDK／流式 API 的流式

常常**不是**完整 Anthropic API 功能面——公开文档陆续点名过某些 Files／URL 输入辅助、一些服务器端工具、Message Batches、Admin API，以及一些受管 agent 基础设施。若题目说「在 Vertex 上」，不要假设每一个 Anthropic 专属功能都存在。

上下文窗口：较新的 Claude 世代可能提供很大的窗口（例如列出模型上的 1M 等级）；较旧的可能较小。优先「查模型卡」，而不是背下每个数字。

**对等决策规则：**

```text
产品需要某个功能？
 → 查 Vertex／Agent Platform 对该模型的支持矩阵
 → 若缺少，重新设计、改用 Anthropic API（若政策允许），或等待功能对等
绝不要未经查核就回答：『Claude 在 Anthropic API 上能做 X，所以 Vertex 也可以』
```

---

## 7. 课程技能主题（与 Bedrock 路线共享）

### 7.1 多轮与系统提示词

用户／助理交替；保持历史连贯；把持久行为放进 `system`。HTTP 调用之间无状态——你的应用程序存历史。

### 7.2 提示工程与评测

XML 风格结构、例子、输出控制；创建**测试集**，用基于模型与基于代码的评分者评分。生产质量 = 迭代循环，不是单一聪明的提示词。

### 7.3 工具使用

JSON Schema 工具 → 模型工具调用 → 你的执行 → 工具结果回到 messages。多轮与批量模式对 agent 很重要。

### 7.4 RAG

切块 → 嵌入 → 检索（向量＋**BM25** 混合）→ 可选重排序 → 上下文化检索技巧 → 有提供时带引用来源作答。概念管线与其他 Claude 云端课程相同；存储／嵌入可能使用 GCP 服务（Vertex AI embeddings、Vector Search 等）。

### 7.5 MCP

定义工具、资源、提示模板；跑 MCP 服务器／客户端；集成进应用程序。协议知识可转移到 Anthropic API、Bedrock 与 Vertex 传输层。

### 7.6 Agent 与工作流程

要背的模式：

| 模式 | 想法 | 何时用 |
| --- | --- | --- |
| 平行化 | 把独立子任务扇出 | 极度可平行的工作 |
| 链式 | 第 N 步的输出喂给第 N＋1 步 | 有相依的管线 |
| 路由 | 先分类再送到专门提示／工具 | 混合意图流量 |
| 调试 | 记录追踪、约束工具、评测中间步骤 | 不稳定的 agent |

---

## 8. Vertex 对上 Bedrock 对上 Anthropic API（比较表）

| 维度 | Anthropic API | Bedrock | Vertex AI Claude |
| --- | --- | --- | --- |
| 云端身份 | Anthropic 账号 | AWS IAM | GCP IAM／ADC |
| 主要 SDK | `Anthropic` | AWS SDK＋Converse | `AnthropicVertex` |
| 模型 ID 样式 | Anthropic ID | `anthropic.`／地理配置文件 | 发布者 ID（`claude-…`） |
| 统一多模型聊天 API | Messages | **Converse**（Bedrock 全局） | 经由 Vertex 后端的 Messages |
| 版本标记 | Anthropic 标头 | Invoke 上 `bedrock-2023-05-31` 风格 | 原始 HTTP 上 `vertex-2023-10-16` |
| 驻留控制 | Anthropic 选项 | 区域＋推理配置文件 | global／us／eu／区域 |
| 企业包装 | Anthropic 合约 | AWS Marketplace／Bedrock | Google Cloud／Model Garden |
| 启用 | API 访问 | Bedrock 模型访问 | Model Garden 启用 |

**选 Vertex，当：** GCP 已经是控制平面、你需要 Google 计费／驻留，或组织政策强制 Vertex。
**选 Bedrock，当：** 以 AWS 为中心的 IAM、跨 Bedrock 模型的 Converse 可携性、AWS 合规边界。
**选 Anthropic API，当：** 通往最新 Anthropic 专属功能的最快路径，以及最简单的密钥验证。

---

## 9. 考试陷阱（Vertex 专属）

| 陷阱 | 现实 |
| --- | --- |
| 对 AnthropicVertex 使用 Anthropic API 密钥 | 错误验证——需要 GCP ADC／服务账户 |
| 粘贴 Bedrock 模型 ID | 错误目录 |
| 欧盟驻留强制时选 global | 错误端点类别 |
| 以为 global 有预置吞吐量 | 通常是区域的 |
| 以为完整 Anthropic 功能对等 | 查支持矩阵 |
| 忘记 Model Garden 启用 | 需要项目层启用 |
| 原始 HTTP 只把模型放在 body | 模型常常在 URL 路径；body 需要 `anthropic_version` |
| 相信 Vertex 会存聊天历史 | 无状态；应用程序重送 messages |
| 忽略 PDF／图像的请求大小上限 | Vertex 可能强制载荷上限 |
| 用 `europe-west1` 跑只在 global 上的全新模型 | 区域落后 |
---

## 10. 考试提示

- 设置顺序：启用模型 → gcloud／ADC → `AnthropicVertex(project, region)`。
- 为原始 HTTP 背下 **`anthropic_version: vertex-2023-10-16`**。
- 区域选择是**合规与可用性**问题，不是表面功夫。
- 不要把 Bedrock 模型 ID 贴进 Vertex，反之亦然。
- 无状态多轮历史仍然是你的工作。
- 对于「Vertex 上有功能 X 吗？」——优先「若 Agent Platform／该模型有列出就支持」，不是 Anthropic API 假设。
- Agent 工作流程词汇：平行化、链式、路由。
- RAG 答案应引用混合检索与评测，而不只是向量嵌入。
- us／eu／区域相对 global 的定价溢价是已知的考试周边事实——在定价页确认现行数字。
- 生产环境的服务账户胜过服务器上的用户 ADC。

---

## 11. 最小练习草图（概念）

1. 在 Model Garden 启用一个 Claude 模型。
2. 设置 gcloud 项目＋ADC。
3. 用 `region="global"` 与一则短 Messages 请求调用 `AnthropicVertex`。
4. 用 `region="us"` 再做一次，注意端点／定价意涵。
5. 附加助理输出，组出两轮聊天。
6. 加一个自定义工具，完成一次工具循环。
7. 草拟一条小 RAG 路径（切块 → 检索 → 作答）与一份含 5 个固定案例的评测评分量规。
8. 尝试一次原始 HTTP `rawPredict`，看 `anthropic_version` 与路径上的模型 ID。
9. 故意用一次 Bedrock 风格模型 ID，认清失败模式。
10. 写下你依赖哪些功能，并逐一对 Vertex 支持清单核实。

---

## 12. 自我检核问答（附答案）

**Q1.** AnthropicVertex 的本机验证通常怎么做？
**A.** `gcloud auth application-default login` 之后的 Google 应用程序默认凭证（或生产环境的服务账户）——不是 Anthropic API 密钥。

**Q2.** 原始请求 body 里，除了／加上 Anthropic 标头，放什么？
**A.** `anthropic_version` 设成 `vertex-2023-10-16`；模型常常住在 URL 路径。

**Q3.** 什么时候选 `region="eu"` 而不是 `global`？
**A.** 当你需要欧盟地理数据驻留，加上欧盟内部的多区域可用性，并接受可能的溢价定价。

**Q4.** 为什么全新的 Claude 模型可能在 `europe-west1` 失败，却在 `global` 能用？
**A.** 区域可用性落后；最新模型常常先登陆 global／精选区域。

**Q5.** 说出一个对上 Bedrock 模型 ID 的差异。
**A.** Vertex 使用像 `claude-sonnet-4-6` 这样的发布者 ID；Bedrock 使用 `anthropic.claude-…` 或 `us.anthropic.…` 配置文件。

**Q6.** Vertex 会在调用之间存储对话历史吗？
**A.** 不会——你的应用程序必须每轮重送 messages（除非你自建会话存储）。

**Q7.** 哪个 agent 模式把不同查询类型送到不同工具／提示？
**A.** 路由。

**Q8.** 即使你部署在 Vertex 上，Converse 知识仍有帮助的两个原因。
**A.** 共享的 Claude 概念（工具、系统提示词、评测）；以及云端考试的比较题常常对比 Bedrock 的 Converse 与 Vertex 在 GCP 上的 Messages 作法。

**Q9.** 什么时候需要特定区域端点？
**A.** 硬性单一区域驻留与／或预置吞吐量（依公开指引）。

**Q10.** Model Garden 的角色是什么？
**A.** 为你的 GCP 项目探索并启用合作伙伴模型（包括 Claude）。

**Q11.** 为什么生产环境优先服务账户？
**A.** 最小权限、可轮替、服务器上没有交互式用户 ADC，适合工作负载身份。

**Q12.** Vertex 上的流式——会改变工具循环正确性规则吗？
**A.** 不会——仍然要组出最终消息，并保持 tool_use／tool_result 配对正确。

**Q13.** 说出两个在 Vertex 上可能与 Anthropic API 不同的功能。
**A.** 例子：Message Batches、某些 Files／URL 辅助、一些服务器工具、Admin API——永远核对现行清单。

**Q14.** 混合 RAG 是什么意思？
**A.** 生成前结合词汇（BM25）与向量检索。

**Q15.** 平行化对上链式？
**A.** 平行化把独立工作扇出；链式把有相依的步骤排成序列。

**Q16.** 与美国多区域相关的主机样式是什么？
**A.** 常常是 `aiplatform.us.rep.googleapis.com` 风格主机（确认文档）。

**Q17.** 可以不在项目里启用就在 Vertex 上用 Claude 吗？
**A.** 不行——需要 Model Garden／项目访问的启用。

**Q18.** `max_tokens` 放哪里？
**A.** 在 Messages 请求 body（SDK 参数）——生成的必填上限。

**Q19.** 结构化提取的 temperature？
**A.** 较旧模型（4.6 及更早）：较低以得到更确定性的输出。现行前沿模型已完全移除采样参数——确定性靠 schema／结构化输出。

**Q20.** 假设 1M 上下文之前该查什么？
**A.** Vertex 上该模型的模型卡——不是所有模型共享同一个窗口。

**Q21.** 工作负载身份的目的，一句话？
**A.** 让云端工作负载取得 GCP 凭证，而不用下载长效密钥。

**Q22.** 引用来源功能——考试注意？
**A.** 在该模型／平台有文档记载时才可用；不要捏造引用来源支持。

**Q23.** 为什么 PDF／图像请求可能在验证有效时仍然失败？
**A.** Vertex 路径上的载荷大小上限，或不支持的媒体处理。

**Q24.** Bedrock `additionalModelRequestFields` 在 Vertex 上的对应物？
**A.** 不是同一套 API——在 Vertex 上你通常传 SDK／API 支持的 Anthropic Messages 字段；额外项依 Anthropic／Vertex 文档，不是 Bedrock Converse 命名。
---

## 13. 复习检查清单（考前）

- [ ] Model Garden 启用步骤
- [ ] ADC 对上服务账户对上 Anthropic API 密钥（哪个属于哪里）
- [ ] global 对上 us／eu 对上区域的决策树
- [ ] 原始 HTTP：路径上的模型 ID＋`anthropic_version: vertex-2023-10-16`
- [ ] 发布者 ID 格式对上 Bedrock ID
- [ ] 对上 Anthropic API 的功能对等注意事项
- [ ] 无状态历史＋工具循环顺序
- [ ] Agent 模式：平行化、链式、路由
- [ ] RAG 混合／上下文化检索词汇
- [ ] 评测 = 系统化代码＋模型评分
- [ ] 凭记忆画三云比较表
- [ ] 预置吞吐量绑在区域端点

---

## 14. 词汇表

- **Model Garden** — GCP 目录，用来探索／启用合作伙伴与 Google 模型。
- **Agent Platform** — Google Cloud 上 agentic／合作伙伴模型服务的接口（文档命名会演进）。
- **ADC** — Google 验证用的应用程序默认凭证。
- **AnthropicVertex** — 官方 Anthropic SDK client，用于以 Vertex 为后端的 Claude。
- **发布者模型 ID** — 例如发布者 `anthropic` 下的 `claude-sonnet-4-6`。
- **rawPredict** — Vertex 原始预测端点，接受 Anthropic 形状的 JSON body。
- **anthropic_version** — Body 字段；Vertex 原始 HTTP 使用 `vertex-2023-10-16`。
- **Global 端点** — 以可用性为目标的动态多区域路由；驻留有弹性。
- **多区域端点** — `us` 或 `eu` 地理路由，带地理驻留。
- **区域端点** — 单一区域路由；驻留钉住；预置吞吐量。
- **预置吞吐量** — 预留容量（通常是区域的）。
- **工作负载身份** — 没有静态密钥的云端工作负载验证。
- **功能对等** — Vertex 对某功能是否支持与 Anthropic API 相同的能力。
- **BM25** — 混合 RAG 里的词汇检索组件。
- **上下文化检索** — 嵌入前用上下文丰富切块。
- **路由／链式／平行化** — 核心 agent 工作流程模式。
- **基于代码的评分** — 程序化评测断言。
- **基于模型的评分** — 经由另一次模型调用依评分量规打分。

---

## 15. 走完的迷你情境

**情境 1 — 错误密钥**
工程师设置 `ANTHROPIC_API_KEY`，纳闷为什么 AnthropicVertex 在有 VPC-SC 的 GCP 项目里失败。
**答案：** 用 GCP 凭证（ADC／服务账户）。Anthropic API 密钥是不同的产品路径。

**情境 2 — 合规**
政策：在欧盟处理。代码为了较便宜的 token 使用 `region="global"`。
**答案：** 改成 `eu` 多区域或特定欧盟区域端点；适用时接受溢价；核实模型可用性。

**情境 3 — 新模型落后**
Opus 在 global 上全新；区域端点 404／不可用。
**答案：** 用 global（若驻留允许）或等待／在需要的区域启用；不要发明 Bedrock 风格配置文件前缀。

**情境 4 — 原始 HTTP 调试**
SDK 能用；手写 curl 失败。
**答案：** 检查来自 gcloud 的 bearer token、URL 位置片段、发布者路径上的模型 ID，以及 JSON 里的 `anthropic_version`。

**情境 5 — 对等缺失**
应用程序依赖 Anthropic 专属的批量 API。搬到 Vertex 就坏了。
**答案：** 查支持矩阵；用自己的队列重新设计，或在政策允许时留在 Anthropic API。

**情境 6 — Agent 设计**
混合意图：重设密码对上订单状态对上常见问题。
**答案：** 路由模式送到专门提示／工具；评测每个分支。

**情境 7 — GCP 上的 RAG**
只靠向量的搜索漏掉精确 SKU 字符串。
**答案：** 混合 BM25＋向量；重排序；上下文化检索；含那些 SKU 的固定评测案例。

**情境 8 — 生产验证加固**
把用户 ADC JSON 提交进存储库给 Cloud Run。
**答案：** 停手；用附加的服务账户／工作负载身份；轮替已暴露的凭证。

---

## 16. 更深的端点比较表

| | Global | 多区域 us／eu | 特定区域 |
| --- | --- | --- | --- |
| 驻留 | 有弹性／没有保证 | 地理层级 | 单一区域 |
| 可用性 | 最高（动态） | 地理范围内高 | 取决于单一区域 |
| 典型价格 | 基准 | 溢价（较新模型常约 10%） | 溢价 |
| 预置吞吐量 | 通常没有 | 查文档 | 有（典型所在） |
| 最适合 | 大多数没有驻留规则的应用 | 美国／欧盟合规＋高可用性 | 硬性钉住／预留容量 |

---

## 17. 读书节奏

第 1 天：设置、验证、端点、模型 ID。画驻留决策树。
第 2 天：功能对等、共享技能、对上 Bedrock／Anthropic 的比较。
第 3 天：问答闭书＋情境＋检查清单。

**口诀：** 在 Garden 启用，用 Google 验证，为驻留挑端点，用发布者 ID，假设是 Messages 不是 Converse，答应功能前先核对对等。
---

## 18. 共享 Claude 技能对应到 Vertex 请求字段

| 技能主题 | Vertex 表达 |
| --- | --- |
| 系统提示词 | Messages 顶层的 `system` |
| Temperature | `temperature` 请求字段 |
| 工具 | `tools`／工具选择字段（Anthropic 形状） |
| 多轮 | 应用程序管理的 `messages` |
| 流式 | SDK 流式辅助／streamPredict |
| 结构化输出 | Schema 提示＋验证；若有列出则用平台功能 |
| 视觉／PDF | 内容区块；留意大小上限 |
| 缓存 | Vertex 模型支持时的提示词缓存字段 |
| 思考 | 支持时的扩展思考参数 |
| Agent | 你的编排（平行／链式／路由）＋可选的 Claude Code |

对上 Bedrock 命名：Bedrock Converse 使用 `inferenceConfig`、`toolConfig`、`additionalModelRequestFields`。Vertex 经由 `AnthropicVertex` 更靠近 Anthropic Messages 名称。

---

## 19. 失败分诊树

```text
验证错误／401／403？
 → ADC 存在吗？专案对吗？服务帐户角色包含 Vertex AI User 权限吗？
 → 此专案在 Model Garden 已启用该模型吗？

404／找不到模型？
 → 发布者 ID 字符串错了？
 → 模型在所选位置不可用？
 → 不小心用了 Bedrock 风格 ID？

空的／被截断的输出？
 → max_tokens 太低；停止原因；工具循环未完成

工具循环错误？
 → 缺少 tool_result；角色顺序；schema 不匹配

落地稽核不通过？
 → 在地理强制要求下用了 global → 改到 us／eu／区域端点
```

---

## 20. 额外自我检核问答

**Q25.** Vertex 路径里的 `locations/global` 是什么意思？
**A.** 使用 global 端点位置，而不是像 `us-east5` 这样的单一区域。

**Q26.** 为什么财务可能在意 us／eu 端点？
**A.** 较新模型相对 global 的溢价定价——成本与合规都重要。

**Q27.** 链式例子，一句话？
**A.** 提取实体 → 抓取记录 → 起草答案，每一步喂给下一步。

**Q28.** MCP 是 Vertex 专属的吗？
**A.** 不是——MCP 与传输层无关；Vertex 是你可能与 Claude 一起托管或消费工具的地方之一。

**Q29.** 这门课里的电脑操作／Claude Code——重点？
**A.** Agentic 应用可以跑在以 Vertex 为后端的 Claude 上；仍然需要 GCP 验证、启用与验证纪律。

**Q30.** 没有驻留规则的新创，最好的第一个端点？
**A.** Global——可用性与基准定价，若合规出现再回头看。

**Q31.** 如何把项目 id 传给 SDK？
**A.** `AnthropicVertex(project_id=..., region=...)` 与／或文档化的环境变量。

**Q32.** 每轮只存最后一则用户消息有什么问题？
**A.** 丢掉助理／工具历史；破坏多轮连贯性与工具循环。

**Q33.** Vertex 应用的基于代码评分例子？
**A.** 在 CI 对输出断言 JSON schema、必填字段，以及禁用的个人识别信息（PII）模式。

**Q34.** 基于模型的评分例子？
**A.** 用固定裁判提示词，依评分量规在 1–5 分上打帮助性／有据程度。

**Q35.** 为什么 Vertex 课程的考试还要比较 Bedrock Converse？
**A.** 云端选择题测试你是否知道：即使 Claude 技能可转移，验证、ID 与 API 形状仍然不同。
---

## 21. 生产环境就绪迷你评分量规（Vertex）

1. 在正确项目启用模型；ID 对 Model Garden 卡片核实过
2. 端点类别匹配驻留政策（global／us／eu／区域）
3. 生产使用服务账户或工作负载身份——不是笔记本电脑用户 ADC
4. IAM 角色尽可能只给预测用的最小权限
5. 流式与非流式路径的超时／重试
6. 评测套件把关提示／模型变更
7. 日志依要求对密钥与敏感提示词脱敏
8. token 用量成本警示；理解溢价端点定价
9. 区域容量失败时文档化的回退（仅在政策允许时）
10. 功能相依对 Vertex 支持矩阵核过

---

## 22. 并排请求解剖（概念）

**Anthropic API：** 主机 `api.anthropic.com`＋API 密钥标头＋body `{model, messages, max_tokens,...}`

**Bedrock Converse：** 对 `bedrock-runtime` 的 AWS SigV4＋`{modelId, messages, inferenceConfig, toolConfig,...}`

**Vertex rawPredict：** 对 `…aiplatform…/publishers/anthropic/models/{MODEL}:rawPredict` 的 Google bearer token＋body `{anthropic_version, messages, max_tokens,...}`（模型在路径）

若你能把同一个 hello-world 请求改写成全部三种形状，你就掌握了云端比较题。

---

## 23. 团队上手一页纸（可贴进 wiki）

1. 为项目 P 在 Model Garden 启用 Claude 模型 X／Y
2. 安装 `anthropic[vertex]`；设置项目；除非合规另有规定，优先 `region=global`
3. 本机：`gcloud auth application-default login`
4. 生产：带文档化角色的服务账户
5. 永远不要提交凭证
6. 用模型卡上的发布者模型 ID——不要拷贝 Bedrock ID
7. 在我们的会话存储服务里自己掌管对话历史
8. 工具必须为每一次 tool_use 回传结果
9. 使用花哨的 Anthropic 专属功能前，先查 Vertex 支持
10. 推行提示变更前跑共享评测包

---

## 24. 速查卡

| 需要 | Vertex 动作 |
| --- | --- |
| 验证 | ADC／服务账户 |
| 启用 | Model Garden |
| 默认位置 | 驻留有弹性时 `global` |
| 美国／欧盟驻留＋高可用性 | `us`／`eu` |
| 硬性钉住／预置 | 特定区域 |
| SDK | `AnthropicVertex` |
| 原始版本字段 | `vertex-2023-10-16` |
| 模型 ID | 发布者 `claude-…` |
| 历史 | 应用程序管理 |
| 对上 Bedrock | Messages 不是 Converse；不同的 ID／验证 |

**一句话口诀：** 在 Vertex 上，在 Garden 启用，用 Google 验证，为驻留选端点，用发布者 ID 调用 Messages，答应 Anthropic API 能力前先核对功能对等。

---

## 25. 读书收尾

若你能不看笔记重画（1）三种端点类别、（2）ADC 对上 API 密钥、（3）发布者对上 Bedrock ID 例子、（4）三云比较表，你就准备好应付 Vertex 差异化考题了。然后刷新共享的 Claude 技能（工具、RAG、MCP、agent、评测）——与 Bedrock／Anthropic 路线同一套想法，Vertex 的传输层与限制。

---

*对齐 https://academy.claude.com/courses/claude-with-google-cloud-s-vertex-ai。发布前，在 Google Cloud 与 Anthropic 文档确认模型 ID、区域、定价溢价与功能矩阵。*

---

## 26. 最终闪卡操练

不看笔记大声说：

1. 启用 → ADC → 端点类别 → 发布者模型 ID → Messages 调用。
2. Global 对上 us／eu 对上区域，各一句话。
3. Vertex 与 Anthropic API 不同的三个原因（验证、版本字段／路径、功能对等）。
4. Vertex 与 Bedrock 不同的三个原因（IAM 云端、Converse 对上 Messages、ID 格式）。
5. 点名平行化、链式与路由，各附一句例子。

若任何操练失败，只重读该节的表格，然后再试。优先短闭书循环，而不是把整份文件从头读到尾。
