---
title: CCDV-F Developer Foundations — Study Pack Index
cert: Claude Certified Developer – Foundations (CCDV-F)
disclaimer: Original study notes — independent and not official Anthropic material
updated: 2026-08-23
edition: Deepened for PRIMARY study
---

# Claude Certified Developer – Foundations (CCDV-F)
## original PRIMARY study pack (deepened)

> **Disclaimer:** These notes are **original study synthesis** for exam prep. They are **not** official Anthropic material and **not** a reproduction of any official course content or quizzes. Grounding sources are **public** Anthropic docs (`docs.anthropic.com` / `platform.claude.com` / `code.claude.com`), **public** Academy course *topics* (API, MCP, Claude Code), and the official CCDV-F Exam Guide v1.0 (July 2026) — download it from the official Pearson VUE Anthropic program page. Always verify live API/SDK details against current official docs before you sit the exam.

> **Edition note:** This pack has been **deepened in place** to serve as a **PRIMARY study source** (not a skim companion). Chapters emphasize Applications & Integration themes (~33.1% of the exam): API mechanics, streaming, caching, batch vs realtime, schemas, config/`CLAUDE.md`, and model pinning — while keeping the public-availability note and **5→8 domain mapping**.

---

## Public availability note (read this first)

| Resource | Publicly available? | Notes |
| --- | --- | --- |
| Anthropic API / Claude Code / MCP docs | **Yes** | Primary technical source of truth |
| Public Claude Academy catalog (API, MCP, Claude Code courses) | **Yes** | Free/public course *outlines* and learning objectives; lesson bodies evolve |
| Official gated prep courses | Not used | These notes are original and grounded in public sources only. |
| Official exam guide PDF / sample items | **Yes** — official Exam Guide v1.0 (effective July 2026); download from the official Pearson VUE Anthropic program page | The authoritative blueprint. All weights and mechanics below are taken from it, cross-checked (2026-08-23) against the public Pearson VUE Anthropic program page |
| Exam registration / proctoring | Pearson VUE | Online proctored or test center; check the official registration portal for current requirements |

**Implication for this pack:** Content is written as *developer study notes* aligned to the published domain map and public product docs — **not** as a substitute for any official course.

---

## Exam mechanics (official Exam Guide v1.0, July 2026)

Values from the official Exam Guide v1.0 (July 2026), cross-checked against the public Pearson VUE Anthropic program page:

| Item | Official value |
| --- | --- |
| Exam code | **CCDV-F** |
| Questions | **53** items |
| Time | **120 minutes** |
| Pass score | **720** on a **100–1000** scaled score |
| Formats | Single-answer MCQ + multi-response; each item states how many to select (the “~¼ multi-response” share is anecdotal, not from the guide) |
| Delivery | Pearson VUE (test center or online proctored), English |
| Fee | **$125 USD** per attempt |
| Validity | **12 months**, with free on-time renewal assessment; retakes 14/30/90-day waits, max 4 attempts per rolling 12 months |
| Recommended experience | 1–5 years engineering + ~6+ months Claude/LLM hands-on; Python and/or TypeScript |

Rough pacing: ~2.3 minutes per item. Scenario stems are long — read the **constraint** (latency, cost, safety, cache stability) before the options.

---

## Official 8 exam domains (weights from Exam Guide v1.0)

| Domain | Weight | Study intensity |
| --- | --- | --- |
| Applications and Integration | **33.1%** | Highest — API, streaming, caching, batch vs realtime, schemas, config/version pinning |
| Model Selection and Optimization | **16.8%** | High — model family, effort, cost/latency, context windows |
| Agents and Workflows | **14.7%** | High — agent loops, Agent SDK patterns, orchestration |
| Prompt and Context Engineering | **11.0%** | Medium-high — system prompts, context budgets, compaction |
| Tools and MCPs | **10.6%** | Medium-high — tool design, MCP servers/clients, connector |
| Security and Safety | **8.1%** | Medium — secrets, PII, guardrails, destructive-action gates |
| Claude Code | **3.1%** | Focused — CLAUDE.md, settings, permissions, MCP in Code |
| Eval / Testing / Debugging | **2.6%** | Focused — eval harnesses, regression sets, debug loops |

**Priority math:** Domains 1–3 alone ≈ **64.6%**. Domains 1–5 ≈ **86.2%**. Do not skip Security or Evals — they are short but appear as decisive scenario filters.

---

## This pack’s chapter outline (PRIMARY study lengths)

Public LinkedIn / partner write-ups often frame prep as five chapters. This pack follows that **study order**, then maps back to the official eight domains below, plus **Chapter 06** closing P0 blueprint gaps (named frameworks, managed hosting, SE/SDLC, websockets). Lengths below are **deepened primary-study targets**.

| # | File | Theme | Target length |
| --- | --- | --- | --- |
| 01 | [01-mso-foundations.md](./01-mso-foundations.md) | Model Selection & Optimization fundamentals | **6500–9000** words |
| 02 | [02-production-prompting-agents-tools.md](./02-production-prompting-agents-tools.md) | Production prompting, agents & tool-use (+ Integration contracts) | **7000–10000** words |
| 03 | [03-claude-code-mcp-integration.md](./03-claude-code-mcp-integration.md) | Claude Code, MCP & **Applications/Integration** (heaviest) | **7000–10000** words |
| 04 | [04-production-engineering-evals-security.md](./04-production-engineering-evals-security.md) | Production engineering, evals, security (+ Integration ops) | **6500–9500** words |
| 05 | [05-accelerators-ip-contribution.md](./05-accelerators-ip-contribution.md) | Accelerators & reusable IP packaging | **5000–7500** words |
| 06 | [06-agent-frameworks-and-sdlc.md](./06-agent-frameworks-and-sdlc.md) | Agent frameworks, hosting modes & SDLC/SE foundations (P0 gap closer) | **2500–4000** words |

**Each chapter includes:** disclaimer · overview · key map · deep notes · decision trees/tables · exam traps · 25–40+ Q&A with answers · checklist · glossary · “if exam asks X think Y”.

Chapter 5 covers accelerators and IP contribution (packaging for reuse). The notes are original guidance on reusable agents, skills packaging, `CLAUDE.md` templates, eval harness reuse, and handoff docs.

---

## Suggested study order (primary path)

1. **Skim this README** + domain weights (30 min).
2. **Chapter 03 first if short on time** — Applications/Integration is **33.1%**; then circle back to 01/02.
3. **Chapter 01 (MSO)** — lock model/effort/cost decision trees (16.8% + Integration cost/latency stems).
4. **Chapter 02 (Prompting / Agents / Tools)** — Agents 14.7% + Prompting 11% + Tools/MCP conceptual half of 10.6% + API tool contracts.
5. **Chapter 04 (Prod eng / Evals / Security)** — Security 8.1% + Eval 2.6% + production Integration patterns.
6. **Chapter 05 (Accelerators)** — packaging & handoff (supports Integration + Agents + Evals in scenario form).
7. **Chapter 06 (Frameworks / Hosting / SDLC)** — Strands/LangGraph/PydanticAI vocabulary, self-hosted vs managed agents, SE/SDLC card, websockets vs SSE (closes P0 gaps).
8. **Timed practice:** 53 questions / 120 min — score by domain; re-read weak chapters’ “If X think Y” tables.

**Offline reading tips:** Each file is plain Markdown (ATX headings, tables, fenced `text` trees). Convert with your preferred MD→EPUB toolchain if you like e-reader formatting; keep one chapter = one file.

---

## Mapping appendix — 5 chapters → 8 official domains

| Pack chapter | Primary official domains | Secondary |
| --- | --- | --- |
| 01 MSO Foundations | Model Selection and Optimization (16.8%) | Applications and Integration (cost/latency/caching choices) |
| 02 Production Prompting, Agents & Tool-use | Agents and Workflows (14.7%); Prompt and Context Engineering (11.0%); Tools and MCPs (10.6%) | Security (tool allowlists); Eval (agent regression); Integration (tool_choice, schemas, cache-stable tools) |
| 03 Claude Code, MCP & Integration | Applications and Integration (33.1%); Tools and MCPs (10.6%); Claude Code (3.1%) | Security (MCP trust); Agents (Code as agent host) |
| 04 Production Engineering, Evals, Security | Security and Safety (8.1%); Eval/Testing/Debugging (2.6%); Applications and Integration (ops) | Agents (reliability); MSO (cost under load) |
| 05 Accelerators & IP Contribution | Applications and Integration (reusable configs); Agents and Workflows (packaged agents); Eval (harness reuse) | Claude Code (templates/skills); partner-specific packaging norms |
| 06 Agent Frameworks & SDLC | Agents and Workflows (frameworks ~4.9%; construction/hosting ~5.3%); Applications and Integration (SE foundations ~7.4%; systems life cycle ~2.8%; streaming transports) | Claude Code (CI/review host); Security/Eval (residency & gates) |

---

## What was deepened most (this edition)

1. **Chapter 03** — Applications & Integration catalog: Messages API mechanics, streaming, prompt caching (explicit vs automatic, pre-warm), batch vs realtime, schemas, pin bundles, CLAUDE.md vs settings enforcement, MCP trust.
2. **Chapter 02** — Production contracts spanning prompts, tools, agents, and Integration (tool_choice, streaming tools, cache-stable toolboxes).
3. **Chapter 04** — Evals/security paired with Integration ops (canaries, multi-tenant cache safety, deploy gates).
4. **Chapter 01** — MSO×Integration serving-path matrix, caching as cost lever, pinning/host matrix.
5. **Chapter 05** — Accelerator bar, kits, and handoff docs aligned to exam-usable engineering (not partner lesson dumps).
6. **Chapter 06** — P0 gap closer: named Strands/LangGraph/PydanticAI vocabulary; self-hosted vs Anthropic-hosted managed agents table; Claude-in-SDLC / SE foundations card; websockets vs SSE note.

### 2026-08-23 revision — blueprint-gap closure + dedup pass

Checked against the official exam guide (v1.0, July 2026):

- **Chapter 01 additions:** §12 LLM fundamentals primer (tokens, next-token generation, non-determinism/sampling — incl. removal of temperature/top_p/top_k on current models — and zero/single/multi-shot definitions); §13 **fast mode** (blueprint-named, previously absent: Opus 5 / 4.8 research preview, ~2.5× output speed at premium price, own rate limit, not on Bedrock/Vertex/Batches); §14 dated **model lineup card** (context windows, adaptive-thinking support, effort levels per tier). Staleness fixes: thinking-config cache semantics (message-level breakpoints vs tools/system prefix), Claude Code default effort = xhigh, Fable phrased as a model tier.
- **Chapter 03 additions:** §36 **Claude Code vocabulary card** — the exact Domain 3 terms (Rules / Skills / Commands / Agents / Agent Memory, `/init`, built-in + custom slash commands, headless vs streaming vs **auto-mode**, session `--resume`/`--continue`), verified against live Claude Code docs; §35.2 deepened for **vision** (base64 vs Files API, PDFs, injection-via-document).
- **Dedup pass (all chapters):** removed mid-file “End of Chapter” markers; resolved duplicate section numbers (02: second §28–31 → §33–36; 04: double §35 → renumbered chain; 03: colliding Q46–Q60 sets → Q61–Q75, downstream Q renumbered); merged repeated rubric/scorecard/template/checklist sections in 04–05 into single canonical versions with pointers. Q&A numbering is now unique and sequential per chapter.

---

## How these notes were made

- Blueprint weights and exam mechanics cited from the official Exam Guide v1.0 (July 2026), cross-checked against the public Pearson VUE Anthropic program page (2026-08-23).
- Technical claims aligned to **public** Anthropic documentation themes: Messages API, model choice / effort, prompt caching, batches, tool use, MCP connector, Claude Code settings/`CLAUDE.md`/MCP.
- **No** official lesson prose copied.
- Product names and model IDs evolve — treat decision *trees* as stable; re-check current model cards before exam day.

## File checklist

- [x] README.md (this file — deepened primary-study edition)
- [x] 01-mso-foundations.md
- [x] 02-production-prompting-agents-tools.md
- [x] 03-claude-code-mcp-integration.md
- [x] 04-production-engineering-evals-security.md
- [x] 05-accelerators-ip-contribution.md
- [x] 06-agent-frameworks-and-sdlc.md
- [x] PRO-TIPS.md — exam-craft companion (pacing, stem protocol, reflex table, distractor archetypes; added 2026-08-23)
- [x] Parent zip: `../CCDV-F-developer-foundations.zip`

Good luck — optimize for **Applications and Integration** first, then MSO and Agents; use Chapter 06 to lock framework/hosting/SDLC vocabulary before timed practice.
