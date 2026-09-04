# Claude Certified Associate – Foundations (CCAO-F)
## Original Exam-Prep Study Pack (Non-Developer / claude.ai Product Focus)

> **Note:** This edition follows the ASD-STE100 Simplified Technical English writing rules. Simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Projects, Artifacts, Skills, Connectors, Research, Memory, MCP, RAG, Haiku, Sonnet, Opus, Fable, Pearson VUE) are exceptions and stay as written. Product UIs and plan gates change. Check current Help Center articles before the exam.

**Exam code:** CCAO-F 
**Audience:** Business professionals who use Claude as a productivity tool (ops, marketing, PM, education, communications, knowledge work). This pack is **not** for API or coding tracks. 
**Pack type:** Original study notes only. They use **public** Anthropic product docs, Help Center, public Claude Academy courses, and the published CCAO-F blueprint. This pack is **not** affiliated with Anthropic. Anthropic does not endorse or sponsor this pack.

---

## Public availability note

| Source | Public? | How this pack uses it |
| --- | --- | --- |
| Published CCAO-F exam blueprint / exam guide (weights, mechanics, domain statements) | Yes — official Exam Guide v1.0 (July 2026). Download it from the official Pearson VUE Anthropic program page. | Domain weights, exam mechanics, skill statements. The pack checked these (2026-08-23) against the public Pearson VUE Anthropic program page |
| Claude Academy public courses (e.g. **Claude 101**, **AI Fluency: Framework & Foundations**, model-choice tutorials) | Yes — free on academy.claude.com / claude.com/resources | Prompting, 4D fluency, product feature mental models |
| Anthropic Help Center & product docs (Projects, Artifacts, Research, Connectors, Skills, privacy) | Yes | Feature behavior, plan differences, configuration facts |
| Official gated prep courses | Not used | These notes are original. They use public sources only. |

**Registration note:** Pearson VUE delivers the exams (online proctored or test center). Academy **course completion badges** (Claude 101, AI Fluency, and similar) are **not** the same as the paid, proctored CCAO-F credential.

---

## Exam mechanics (from published guide)

| Item | Detail |
| --- | --- |
| Questions | **60** (all scored) |
| Time | **120 minutes** (~2 min/question average) |
| Formats | Single-answer MCQ + multiple-response. Each item states how many to select. The “~¼ multiple-response” share is anecdotal. It is not from the guide |
| Pass score | **720** on a **100–1000** scaled score |
| Language | English |
| Delivery | Pearson VUE — online proctored or test center |
| Fee | ~USD $99 per attempt (check this at registration) |
| Validity | **12 months**. Free on-time renewal assessment. After a lapse, you take the full exam |
| Retakes | Common waiting periods: 14 / 30 / 90 days. Attempt caps apply in a rolling year |
| Score report | Pass/fail + scaled score. Domain % correct is informational |

**Tip:** Study domain files in order of **weight** (below). Do not study by file number. File numbers follow the user’s 7-domain map. Official blueprint order uses slightly different numbers. Both maps cover the same seven topics.

---

## Domain map (user order ↔ official CCAO-F)

| File | Your domain | Official CCAO-F name | Weight | ~Q |
| --- | --- | --- | --- | --- |
| `01-platform-model-foundations.md` | Claude Platform & Model Foundations | Product and Model Selection | **~12%** | ~7 |
| `02-prompting-task-execution.md` | Prompting & Task Execution | Prompting and Task Execution | **~14%** | ~8 |
| `03-evaluating-validating-output.md` | Evaluating & Validating Claude's Output | Output Evaluation and Validation | **~21%** (heaviest) | ~13 |
| `04-workflow-integration-solution-design.md` | Workflow Integration & Solution Design | Workflow Integration and Solution Design | **~16%** | ~10 |
| `05-configuration-knowledge-management.md` | Configuration & Knowledge Management | Configuration and Knowledge Management | **~12%** | ~7 |
| `06-governance-risk-responsible-use.md` | Governance, Risk & Responsible Use | Governance, Risk, and Responsible Use | **~15%** | ~9 |
| `07-troubleshooting-optimization.md` | Troubleshooting & Optimization | Troubleshooting and Optimization | **~10%** | ~6 |

---

## Recommended study order (by exam weight)

1. **Domain 03 — Evaluating & Validating Output (21%)** — start here. Judgment questions dominate. 
2. **Domain 04 — Workflow Integration & Solution Design (16%)** — scenarios, stakeholders, redesign. 
3. **Domain 06 — Governance, Risk & Responsible Use (15%)** — policy, privacy, high-risk use. 
4. **Domain 02 — Prompting & Task Execution (14%)** — craft, decompose, iterate. 
5. **Domain 01 — Platform & Model Foundations (12%)** — features + Haiku/Sonnet/Opus(/Fable). 
6. **Domain 05 — Configuration & Knowledge Management (12%)** — Projects, instructions, connectors. 
7. **Domain 07 — Troubleshooting & Optimization (10%)** — diagnose weak outputs and workflows.

**Hands-on (public product):** Practice on claude.ai / Claude Desktop. Use Projects, Artifacts, Skills, Connectors, and Research. Paid plans give Research / RAG capacity. Pair this study with public Academy: Claude 101 + AI Fluency (4D: Delegation, Description, Discernment, Diligence).

---

## Product scope (what this pack covers)

**In scope (non-developer):** claude.ai chat, Projects (instructions + knowledge + RAG), Artifacts, Skills, Connectors / MCP, Research, Enterprise Search (Team/Enterprise). Also in scope: **Memory & incognito chats**, **code execution & file creation**, model picker, Styles / preferences, usage limits, responsible use.

**Out of scope:** API coding, Claude Code as a developer tool track, Bedrock/Vertex implementation, writing exploits or leaked exam material.

---

## Pack maintenance (updated 2026-08-23)

- **The pack now holds one clean primary-study copy per domain file**. The earlier padding pattern is gone. That pattern pasted the same drill blocks 7–15 times per file. Files 01, 02, 04, 05, 06, and 07 now match the clean Domain 03 rewrite. Every file now has deep notes, decision trees, exam traps, and 15–25 real scenario Q&A with rationales.
- **Coverage gaps closed:** claude.ai **Memory** is in the official exam guide's prep list and Domain 3 objectives. Its main treatment is in `05-…`. A selection-side section is in `01-…`. **Code execution & file creation** is also in the guide. Coverage is in `01-…` (feature) and in `04-…`/`07-…` (workflow and troubleshooting use).
- **`PRO-TIPS.md` added (2026-08-23):** exam-craft companion. It holds the four master instincts, a reflex table, distractor archetypes, and a 48-hour plan. Read it after the domain files.
- **Plan facts checked again against support.claude.com (Aug 2026):** Free = max five Projects and one custom connector. Project RAG = paid plans, up to ~10× knowledge capacity. Memory generation is on Free/Pro/Max (Enterprise legacy experience). Chat search is on Pro+. Incognito chats are on all plans. Plan caps change. Check them again before exam day.

## How each domain file is structured

1. Disclaimer 
2. Overview 
3. Key map (objectives ↔ exam verbs) 
4. Deep notes 
5. Decision trees / tables 
6. Exam traps 
7. Practice Q&A (15–25) with brief rationales 
8. Pre-exam checklist 
9. Glossary 

---

## Suggested 3-week cadence

| Week | Focus | Practice |
| --- | --- | --- |
| 1 | Domains 03 + 04 + a quick read of 06 | Daily: evaluate 3 Claude outputs. Write a “ship / revise / reject” note for each |
| 2 | Domains 02 + 01 + 05 | Build one Project. One Artifact workflow. One Research brief with citations |
| 3 | Domain 07 + full review | Timed 60Q simulation. Drill multiple-response. Re-read traps |

---

## Disclaimer

This is an **independent original study aid**. Product UI and model names change. Always check current [support.claude.com](https://support.claude.com) and [academy.claude.com](https://academy.claude.com). Exam facts come from the official Exam Guide v1.0 (July 2026). The pack checks them against the public Pearson VUE Anthropic program page. Anthropic may publish updated guide versions.
