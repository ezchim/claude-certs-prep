# CCAR-F — Claude 认证架构师基础考试（Foundations）：学习包索引

原创备考笔记。**非** Anthropic 官方材料；以作者自己的话撰写，供专注阅读与认证复习。

> **注记：** 本文件夹（`CCAR-F-architect-foundations`，于 2026-08-23 更名）涵盖 **CCAR-F（Claude Certified Architect – Foundations）** — 认证名称与代码已对照 Pearson VUE 的 Anthropic 官方项目列表确认。独立的 `CCAR-P-architect-professional` 目录涵盖 Professional 考试。

---

## 考试（出自官方考试指南 v1.0，2026 年 7 月 — 权威来源；请从 Pearson VUE 的 Anthropic 官方项目页面下载）

| 项目 | 值 |
| --- | --- |
| 凭证 / 代码 | Claude Certified Architect – Foundations / **CCAR-F** |
| 题数 | **60**（单选 + 复选；每题会注明应选几个选项） |
| 结构 | **4 个情境题，从 6 题题库中随机抽取**（考试指南印出全部 6 题） |
| 时间 | 120 分钟 |
| 及格 | **720** 量表分数（100–1,000）；成绩报告含各领域百分比 |
| 费用 / 效期 | **USD $125** / 12 个月（准时续证免费、免监考） |
| 重考 | 等待期 14 / 30 / 90 天；任一滚动 12 个月最多 4 次 |
| 交付 | Pearson VUE（在线监考或考场） |

### 考试蓝图（配分比重即读书时间的铁律）

| # | 领域 | 配分比重 | 主要文件 |
| --- | --- | --- | --- |
| 1 | Agentic 架构与编排 | **27%** | **08**（+ 03 §agent 循环） |
| 2 | 工具设计与模型上下文协议（MCP）集成 | 18% | **04 + 其领域 2 补充篇**（+ 03 §工具） |
| 3 | Claude Code 设置与工作流程 | **20%** | **05 + 其领域 3 补充篇** |
| 4 | 提示工程与结构化输出 | **20%** | **03**（schema、tool_choice、batches、few-shot） |
| 5 | 上下文管理与可靠性 | 15% | **09**（+ 05 §压缩） |

**明确不在考试范围（不要把备考时间花在这里）：** 云端供应商设置（AWS/GCP/Azure）、流式／服务器推送事件（SSE）实现、速率限制与定价、验证协议、提示词缓存的实现细节、Constitutional AI／人类反馈强化学习（RLHF）、向量嵌入／向量数据库、电脑操作（computer use）、视觉、微调、MCP 服务器托管／部署、token 计数。

### 6 个情境题（考卷会出现其中 4 个）

1. 客服解决 agent（D1、D2、D5）
2. 以 Claude Code 产生代码（D3、D5）
3. 多 agent 研究系统（D1、D2、D5）
4. 以 Claude 提升开发者生产力（D2、D3、D1）
5. 以 Claude Code 做持续集成（CI）（D3、D4）
6. 结构化数据提取（D4、D5）

---

## 文件 — 依 CCAR-F 标注优先级

| # | 文件 | 涵盖 | CCAR-F 优先级 |
| --- | --- | --- | --- |
| 08 | [08-agentic-orchestration-agent-sdk.md](./08-agentic-orchestration-agent-sdk.md) | **领域 1（27%）**：agentic 循环、协调者–子代理、Task tool／AgentDefinition、hooks、任务拆解、sessions／forking | **主要 — 先读** |
| 09 | [09-context-reliability.md](./09-context-reliability.md) | **领域 5（15%）**：案例事实、中段遗失、升级、错误传播、草稿区／清单文件、校准、出处可追溯性 | **主要** |
| 05 | [05-claude-code-in-action.md](./05-claude-code-in-action.md) + 领域 3 补充篇 | **领域 3（20%）**：引导／设置概念 + CLAUDE.md 层级、@import、.claude/rules/、commands、SKILL.md frontmatter、-p／--output-format／--json-schema | **主要** |
| 03 | [03-building-with-claude-api.md](./03-building-with-claude-api.md) | **领域 4（20%）** + D1／D2 的 API 半部：tool_use、tool_choice、schema、batches、agent 循环 | **主要** |
| 04 | [04-introduction-to-mcp.md](./04-introduction-to-mcp.md) + 领域 2 补充篇 | **领域 2（18%）**：MCP 概念 + isError、错误分类法、.mcp.json vs ~/.claude.json、内置工具 | **主要** |
| 01 | [01-claude-101.md](./01-claude-101.md) | claude.ai 产品基础 | 仅背景 — 属 CCAO-F 范围，CCAR-F 产出低 |
| 02 | [02-ai-fluency.md](./02-ai-fluency.md) | 4D 流畅度框架 | 仅背景 — CCAR-F 产出低 |
| 06 | [06-claude-with-amazon-bedrock.md](./06-claude-with-amazon-bedrock.md) | Bedrock 访问、身份与访问管理（IAM）、Converse | **CCAR-F 不在考试范围**（考试指南排除云端供应商设置）— 平台工作可留，本考试请跳过 |
| 07 | [07-claude-with-google-cloud-vertex-ai.md](./07-claude-with-google-cloud-vertex-ai.md) | Vertex AI 设置、应用程序默认凭证（ADC）、端点 | **CCAR-F 不在考试范围** — 同 06 |

*（文件 01–07 源自 Claude Academy 课程笔记 — Claude 101、AI Fluency、Building with the Claude API、Intro to MCP、Claude Code in Action、Bedrock、Vertex。文件 08–09 以及 04／05 的补充篇于 2026-08-23 添加，用以补齐本包对考试指南蓝图的缺口。）*

**应试技巧**（配速、六情境策略、反射表、干扰选项模式）：[PRO-TIPS.md](./PRO-TIPS.md) — 读完内容文件后再看。

---

## 建议研读顺序（由配分比重驱动）

1. **文件 08** — 领域 1 占 27%，且驱动 6 个情境中的 3 个；没有其他文件报酬更高。
2. **文件 05 + 其领域 3 补充篇** — 20%；补充篇收录精确机制（rules 的 glob、commands 作用范围、CI 标志），十二题样例题中有两题考这些。
3. **文件 03** — 领域 4（20%）：tool_use schema、tool_choice 的 auto／any／forced、nullable 字段、带反馈重试的主题、Batches（50%／24h／custom_id／不支持多轮工具调用）。
4. **文件 04 + 其领域 2 补充篇** — 18%：描述、错误分类法、isError、作用范围。
5. **文件 09** — 领域 5（15%），然后重扫 08 的交接小节（两章互相扣连）。
6. 把考试指南 PDF 里的 **12 题样例题**全部冷答一遍；每一题都能从文件 03／04／05／08／09 作答 — 答错会告诉你该重读哪份文件。
7. 时间允许时亲手做指南的 **4 个准备练习**（与情境题库一对一对应）。

## 这些笔记是怎么做成的

- 文件 01–07：公开课程页面用于标题、模块清单与载明的学习目标；内文为原创综合整理。
- 文件 08–09 + 补充篇：直接对照官方考试指南的任务陈述撰写，SDK／CLI 机制已于 2026-08 对照当时公开的 Claude Code／Agent SDK 文档验证。名称会漂移 — 考试前请对照在线文档核对 `--resume`、SKILL.md frontmatter，以及 Agent SDK 选项拼法。特别注意：考试指南写 **Task tool**；现行 Claude Code 已改名为 **Agent** — 考试作答用指南的用语。
- 时效修正 2026-08-23：现行前沿模型移除 temperature／采样参数（03、06、07）、自适应思考对上已弃用的思考预算（06）、对话中途系统消息注记（03）。

## 相关（仅指标）

- 进阶 MCP（读完 `04-…` 的入门笔记之后）：[Model Context Protocol: Advanced Topics](https://academy.claude.com/courses/model-context-protocol-advanced-topics)
