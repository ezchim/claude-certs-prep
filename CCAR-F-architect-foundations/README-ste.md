# CCAR-F — Claude Certified Architect – Foundations: Study Pack Index

> **Note:** This edition follows the ASD-STE100 Simplified Technical English writing rules. Simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Claude Code, MCP, Task, AgentDefinition, SKILL.md, Bedrock, Vertex, Pearson VUE) are exceptions and stay as written. Names and CLI flags change. Check current public docs before the exam.

Original exam-prep study notes. These notes are **not** official Anthropic material. The author writes them in original words for focused reading and certification review.

> **Note:** This folder (`CCAR-F-architect-foundations`, renamed 2026-08-23) covers **CCAR-F (Claude Certified Architect – Foundations)**. The cert name and code match the official Pearson VUE Anthropic program listing. The separate `CCAR-P-architect-professional` directory covers the Professional exam.

---

## The exam (from the official Exam Guide v1.0, July 2026 — authoritative. Download it from the Pearson VUE Anthropic program page)

| Item | Value |
| --- | --- |
| Credential / code | Claude Certified Architect – Foundations / **CCAR-F** |
| Items | **60** (MC + multiple-response). Each item states how many to select |
| Structure | **4 scenarios drawn at random from a bank of 6**. The guide prints all 6 |
| Time | 120 minutes |
| Pass | **720** scaled (100–1,000). The score report includes per-domain % |
| Fee / validity | **$125 USD** / 12 months (free non-proctored on-time renewal) |
| Retakes | 14 / 30 / 90-day waits. Max 4 attempts per rolling 12 months |
| Delivery | Pearson VUE (online proctored or test center) |

### Blueprint (weights are study-time law)

| # | Domain | Weight | Primary file(s) |
| --- | --- | --- | --- |
| 1 | Agentic Architecture & Orchestration | **27%** | **08** (+ 03 §agent loop) |
| 2 | Tool Design & MCP Integration | 18% | **04 + its Domain 2 supplement** (+ 03 §tools) |
| 3 | Claude Code Configuration & Workflows | **20%** | **05 + its Domain 3 supplement** |
| 4 | Prompt Engineering & Structured Output | **20%** | **03** (schemas, tool_choice, batches, few-shot) |
| 5 | Context Management & Reliability | 15% | **09** (+ 05 §compaction) |

**Explicitly out of scope (do not spend exam-prep time here):** cloud-provider configurations (AWS/GCP/Azure). streaming/SSE implementation, rate limits & pricing, auth protocols, prompt-caching implementation detail. Also out of scope: Constitutional AI/RLHF, embeddings/vector DBs, computer use, vision, fine-tuning, MCP server hosting/deployment, token counting.

### The 6 scenarios (4 appear on your form)

1. Customer Support Resolution Agent (D1, D2, D5)
2. Code Generation with Claude Code (D3, D5)
3. Multi-Agent Research System (D1, D2, D5)
4. Developer Productivity with Claude (D2, D3, D1)
5. Claude Code for CI (D3, D4)
6. Structured Data Extraction (D4, D5)

---

## Files — priority-labeled for CCAR-F

| # | File | Covers | CCAR-F priority |
| --- | --- | --- | --- |
| 08 | [08-agentic-orchestration-agent-sdk.md](./08-agentic-orchestration-agent-sdk.md) | **Domain 1 (27%)**: agentic loops, coordinator–subagent, Task tool/AgentDefinition, hooks, decomposition, sessions/forking | **PRIMARY — study first** |
| 09 | [09-context-reliability.md](./09-context-reliability.md) | **Domain 5 (15%)**: case facts, lost-in-the-middle, escalation, error propagation, scratchpads/manifests, calibration, provenance | **PRIMARY** |
| 05 | [05-claude-code-in-action.md](./05-claude-code-in-action.md) + Domain 3 supplement | **Domain 3 (20%)**: steering/config concepts + CLAUDE.md hierarchy, @import, .claude/rules/, commands, SKILL.md frontmatter, -p/--output-format/--json-schema | **PRIMARY** |
| 03 | [03-building-with-claude-api.md](./03-building-with-claude-api.md) | **Domain 4 (20%)** + D1/D2 API halves: tool_use, tool_choice, schemas, batches, agent loop | **PRIMARY** |
| 04 | [04-introduction-to-mcp.md](./04-introduction-to-mcp.md) + Domain 2 supplement | **Domain 2 (18%)**: MCP concepts + isError, error taxonomy, .mcp.json vs ~/.claude.json, built-in tools | **PRIMARY** |
| 01 | [01-claude-101.md](./01-claude-101.md) | claude.ai product basics | Background only — CCAO-F material. Low CCAR-F exam value |
| 02 | [02-ai-fluency.md](./02-ai-fluency.md) | 4D fluency framework | Background only — low CCAR-F exam value |
| 06 | [06-claude-with-amazon-bedrock.md](./06-claude-with-amazon-bedrock.md) | Bedrock access, IAM, Converse | **OUT OF SCOPE for CCAR-F** (the guide excludes cloud-provider config). Keep it for platform work. Skip it for this exam |
| 07 | [07-claude-with-google-cloud-vertex-ai.md](./07-claude-with-google-cloud-vertex-ai.md) | Vertex AI setup, ADC, endpoints | **OUT OF SCOPE for CCAR-F** — same as 06 |

*(Files 01–07 come from Claude Academy course notes. Those courses are Claude 101, AI Fluency, Building with the Claude API, Intro to MCP, Claude Code in Action, Bedrock, and Vertex. The pack adds files 08–09 and the supplements in 04/05 on 2026-08-23. They close the pack's gaps against the exam-guide blueprint.)*

**Exam craft** (pacing, the 6-scenario strategy, reflex table, distractor patterns): [PRO-TIPS.md](./PRO-TIPS.md). Read it after the content files.

---

## Suggested study order (weight-driven)

1. **File 08** — Domain 1 is 27% and drives 3 of 6 scenarios. No other file gives more exam value.
2. **File 05 + its Domain 3 supplement** — 20%. The supplement holds the exact mechanics (rules globs, commands scope, CI flags). Two of the twelve sample questions test those mechanics.
3. **File 03** — Domain 4 (20%): tool_use schemas, tool_choice auto/any/forced, nullable fields, retry-with-feedback themes. Also Batches (50% / 24h / custom_id / no multi-turn tool calling).
4. **File 04 + its Domain 2 supplement** — 18%: descriptions, error taxonomy, isError, scoping.
5. **File 09** — Domain 5 (15%). Then re-read 08 §handoff. The two sections interlock.
6. Walk all **12 sample questions** in the exam guide PDF without notes. You can answer every one from files 03/04/05/08/09. A miss tells you which file to re-read.
7. Do the guide's **4 preparation exercises** hands-on if time allows. They map 1:1 to the scenario bank.

## Make How these notes.

- Files 01–07: Public course pages supply titles, module lists, and stated learning objectives. The prose is original synthesis.
- Files 08–09 + supplements: These files follow the official exam-guide task statements. The pack checks SDK/CLI mechanics against current public Claude Code / Agent SDK docs (2026-08). Names change. Check `--resume`, SKILL.md frontmatter, and Agent SDK option spellings in live docs before exam day. Note: the exam guide says **Task tool**. Current Claude Code renames it Agent. Answer exams with the guide's term.
- Currency fixes 2026-08-23: temperature/sampling-param removal on current frontier models (03, 06, 07). Also adaptive thinking vs deprecated thinking budgets (06), and a mid-conversation system messages note (03).

## Related (pointer only)

- Advanced MCP (after intro notes in `04-…`): [Model Context Protocol: Advanced Topics](https://academy.claude.com/courses/model-context-protocol-advanced-topics)
