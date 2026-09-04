---
title: 01 — Claude Platform & Solution Design — Simplified Technical English
disclaimer: Original study notes — independent and not official course content
approx_length: STE edition (ASD-STE100) — primary study
updated: 2026-08-30
---

# 01 — Claude Platform & Solution Design

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, MCP, prompting, caching, effort, p95) are exceptions and stay as written. Model IDs and prices change. Learn the decision rules. Check the current model cards before the exam.

**CCAR-P condensed domain 1**
**Official domains mapped here:** Solution Design & Architecture (**17%**) · Claude Models, Prompting & Context Engineering (**13%**)
**Combined exam weight covered by this file:** ~**30%**

---

## Disclaimer

These are **original study notes** for exam preparation. They use **public** Anthropic platform documentation. They also use published CCAR-P blueprint summaries and common architecture patterns for LLM systems. They do **not** copy official course content. Model names, prices, and context limits change. Always check numbers against current [platform.claude.com](https://platform.claude.com) docs. Do this before production decisions and before exam day. This pack is independent. It is not affiliated with Anthropic.

---

## Overview

CCAR-P expects you to think like a **production architect**. It does not expect you to only adjust prompts. In scenarios you:

1. Change a messy business problem into a **Claude-powered solution shape** (inputs → processing → outputs → feedback).
2. Select among **workflow**, **augmented LLM**, **agentic**, and **multi-agent** patterns. State the trade-offs.
3. Select a **model tier** (capability vs latency vs cost vs context) and justify it.
4. Design **prompts and context** so the system stays reliable under token, safety, and reuse constraints.
5. Connect design choices to **business value pillars**. The official guide names these: efficiency, transformation, **productivity**, cost, and performance SLAs.

This file covers platform capabilities and solution design in depth. Integration mechanics (MCP wiring, RAG pipelines, eval harnesses) appear in file `02`. Governance appears in `03`. Stakeholder, lifecycle, and enablement appear in `04`.

---

## Key map (user domain ↔ official tasks)

| Architect task | Official domain | What “good” looks like on the exam |
| --- | --- | --- |
| Problem → solution framing | Solution Design 17% | Clear value hypothesis, constraints, and success metrics |
| Pattern selection (workflow/agent/…) | Solution Design 17% | Prefer the simplest pattern that meets audit and reliability needs |
| Decomposition & orchestration | Solution Design 17% | Stages, gates, retries, and human checkpoints when risk is high |
| Model selection | Models/Prompting 13% | Match complexity, latency budget, context size, and cost |
| System prompts & behavioral controls | Models/Prompting 13% | Role, constraints, output schema, and refusal boundaries |
| Few-shot / structured reasoning | Models/Prompting 13% | Examples when format or edge cases matter. Thinking for hard reasoning |
| Context window strategy | Models/Prompting 13% | Progressive discovery vs curated context. Caching and Skills for reuse |
| Prompt caching / modular prompts / Skills | Models/Prompting 13% | Reuse static prefixes. Do not put extra content in every request |

---

## Part A — Claude models for architects

### A1. How to think about the model ladder

Public docs describe a **family** of Claude models. The models have different speed, intelligence, and cost points. Exact IDs and prices change over time. For CCAR-P, learn the **decision logic**. Do not memorize a frozen price sheet:

| Decision factor | Prefer higher-capability tier when… | Prefer faster/cheaper tier when… |
| --- | --- | --- |
| Task complexity | Multi-step planning, hard coding, ambiguous requirements, high-stakes judgment | Classification, extraction, routing, short rewriting |
| Latency SLA | Offline batch, async jobs, human-reviewed drafts | Interactive UX, high QPS gatekeepers |
| Context size | Large codebases, long docs, multi-hour agent traces | Short turns, tool-heavy loops with small payloads |
| Cost envelope | Low volume / high value (contracts, architecture reviews) | High volume / low marginal value |
| Error cost | A wrong answer is expensive or unsafe | You can catch or retry a wrong answer at low cost |

**Architect rule:** Default to the **smallest model that meets quality bars on your eval set**. Raise capability only when evals show a gap. Lower capability when latency or cost budgets fail and quality still holds.

### A2. Capability controls that appear in designs

Public platform features that you must place correctly:

- **Context windows:** Current frontier Claude models often advertise **large** windows (hundreds of thousands to **~1M tokens** on supported models/platforms). Everything in the request counts: system prompt, messages, tool defs, images/docs, tool results, and thinking tokens when you use them. Caching changes **price**. Cached tokens still occupy the window.
- **Max output:** Compare the synchronous Messages API and Batch. Batch may support extended output via documented beta headers. Check current docs.
- **Thinking / adaptive thinking / effort:** Use deeper reasoning for hard problems. Control cost and latency with **effort levels** (`output_config.effort`). Adaptive thinking replaced fixed thinking budgets. `budget_tokens` is deprecated on the 4.6 generation. Current models remove it and reject it with an error. A change to thinking config invalidates message-level prompt-cache breakpoints. The tools/system prefix stays cached. Design for cache stability.
- **Vision / multimodal:** Use image input when the product needs screenshots, diagrams, or scanned docs.
- **Deployment surfaces:** Claude API, Amazon Bedrock, Vertex AI, Microsoft Foundry, and Claude Platform on AWS. Each has ID naming and data-residency nuances. You select based on enterprise procurement, region, and compliance. Do not select from brand preference alone. **The full CSP / delivery-route governance decision table lives in file `02` Part D5** (constraint elimination, ZDR themes, model-ID traps, Q&A).

### A3. Model selection decision tree

```
START: Define task + quality bar + p95 latency + monthly volume
 │
 ├─ Need long-running agentic coding / complex enterprise judgment?
 │ └─ YES → Start with high-capability model; measure cost on realistic traces
 │
 ├─ Need interactive speed + “good enough” intelligence?
 │ └─ YES → Mid-tier (Sonnet-class); escalate hard cases via router
 │
 ├─ High QPS classify/extract/route?
 │ └─ YES → Fast tier (Haiku-class); sample escalate to mid/high on low confidence
 │
 └─ Mixed workload?
 └─ YES → Explicit router (rules or small model) + per-route SLOs
```

**Common exam error:** You select Opus/Fable-class for every step of a deterministic ETL-like pipeline. Prefer **workflow + smaller model** with validation gates.

### A4. Multi-model architectures

Common production shapes:

1. **Router → specialist:** A cheap classifier selects extract vs reason vs refuse.
2. **Draft → critique:** A mid model writes a draft. A stronger model or rubric critic reviews it.
3. **Shadow eval:** A new model runs in parallel. You compare offline before cutover.
4. **Fallback:** If the primary times out, use a degraded model or a cached answer plus a human queue.

Record **why** each hop exists (quality, cost, safety). Extra hops fail reviews. They also fail exam scenarios that ask for the simplest viable design.

---

## Part B — Prompting & behavioral control

### B1. System prompt as architecture, not poetry

Treat the system prompt as a **policy + interface contract**:

| Section | Purpose | Example content (illustrative) |
| --- | --- | --- |
| Role & scope | Bound the job | “You are a procurement assistant for freight quotes. You do not negotiate contracts.” |
| Tools policy | When to call what | “Call `get_policy` before recommending. Never invent policy clauses.” |
| Output contract | Schema / format | JSON schema, citation requirements, severity enums |
| Safety / refusal | Local guardrails | Disallowed actions. Escalation phrases |
| Quality bar | How to handle uncertainty | “If confidence low, ask clarifying question or return `needs_review`.” |
| Style | Audience | Exec brief vs engineer debug |

**Prefer:** Explicit constraints + structured outputs.
**Avoid:** Vague “be helpful and thorough” with no success criteria.

### B2. Prompting techniques architects should name correctly

| Technique | When to use | Failure mode |
| --- | --- | --- |
| Zero-shot | Clear tasks, strong base capability | Ambiguous formats drift |
| Few-shot | Format, tone, edge-case handling | Bad examples become the rule. Token bloat |
| Chain-of-thought / structured reasoning | Multi-step logic (prefer API thinking features where available) | You leak the chain into a user-facing channel. Cost |
| Decomposition in prompt | Single call must cover subskills | Still weaker than real workflow gates for audit |
| Tool-forced (`tool_choice`) | Retrieval or action must happen | You force a tool when optional tools exist |
| Constitutional / principle prompts | Local safety/style overlays | Conflicts with org policy if you do not review |

### B3. Output contracts and machine consumers

If a downstream system parses Claude’s output, design for **machines first**:

- Prefer **JSON / typed schemas**. Validate before side effects.
- Separate **user-facing prose** from **machine fields**.
- Add explicit enums for status (`ok`, `needs_review`, `refused`).
- Validate with code. Do not hope that the model obeyed.

### B4. Skills and modular prompt reuse (public product concept)

Public Claude product docs describe **Skills** (and related modular instruction packs). Skills are reusable specialized instructions and resources. Teams do not paste giant instruction blobs into every request. Architect takeaway for CCAR-P:

- Put **stable** expertise into modular reusable units.
- Keep **request-specific** data in the user/tool channel.
- Version Skills like code. Review for prompt-injection surfaces when Skills can pull external content.

---

## Part C — Context engineering

### C5. What counts toward the window

Everything that you send counts. You must budget:

1. System / Skills / static policy
2. Tool definitions (large MCP tool catalogs can dominate)
3. Conversation history
4. Retrieved chunks / files
5. Tool results
6. Thinking tokens (when enabled)

A public **Token counting API** exists. Use it to estimate before you send. Use it in design reviews for worst-case prompts.

### C6. Progressive discovery vs monolithic context

| Strategy | Idea | Best for | Risk |
| --- | --- | --- | --- |
| Monolithic context | Put “everything possibly relevant” into the prompt | Small corpora, strong need for global consistency | Noise, cost, lost-in-the-middle, cache thrash |
| Progressive discovery | Start lean. Retrieval or tools pull what you need | Large corpora, tool ecosystems, agents | Missing retrieval → incomplete answers |
| Hybrid | Cached core policy + dynamic retrieval | Most enterprise apps | Complexity of two paths |

**Exam preference:** When corpora or tool sets are large, prefer **progressive discovery** (RAG, tools, MCP tool search). Do not dump megabytes of text. When the task is short and closed-world, a curated static context can be simpler and more auditable.

### C7. Prompt caching (architect view)

Public docs: prompt caching lowers cost and latency for repeated prefixes (system prompts, large reference docs, tool defs). Cache writes cost more than ordinary input. Cache **reads** have a large discount. TTLs often include short options (for example, ~5 minutes) and longer options (for example, ~1 hour). Confirm current docs.

**Design rules:**

- Put **stable content first** (cacheable prefix).
- Put **volatile user content last**.
- Do not reshuffle tool defs on every request.
- A change to thinking budgets or modes can invalidate breakpoints. Keep configs stable in hot paths.
- Know this: cached tokens **still consume** context window capacity.

### C8. Compaction / context editing

For long-running agents and chats, public platform features include **compaction** (server-side summarization near limits). They also include **context editing** strategies (for example, clearing old tool results). You should:

- Define what you must **never** summarize away (safety policy, user entitlements).
- Record when compaction occurs, for audit.
- Test agent quality after compaction. Summary drift is a real failure mode.

### C9. Context engineering checklist

- [ ] Priority order of context sections documented
- [ ] Retrieval top-k and ranking justified
- [ ] Tool catalog size managed (tool search / allowlists)
- [ ] Cache breakpoints stable across turns
- [ ] PII/minimization rules for what enters context
- [ ] Escape hatches when context insufficient (`needs_more_data`)

---

## Part D — Solution design & architecture patterns

### D1. End-to-end solution skeleton

Every CCAR-P-worthy design can be drawn as:

```
[Actors/Channels] → [Ingress & AuthN/Z] → [Orchestration layer]
 → [Claude calls / tools / retrieval]
 → [Validators & policy gates]
 → [Side effects / UX]
 → [Telemetry + eval feedback loop]
```

Map each box to owners (app team, platform, security, data). Missing owners cause lifecycle failure (see file 04).

### D2. Pattern catalog (memorize these patterns)

Anthropic-style teaching and exam scenarios distinguish **predefined workflows** from **agents that decide their own path**.

#### 1) Augmented LLM (single call + light tools)

One Claude call, maybe with retrieval or a tool or two.
**Use when:** The task is bounded. One reasoning pass is enough.
**Pros:** Simple, cheap, easy to eval.
**Cons:** Weak for multi-stage audit. Limited recovery.

#### 2) Workflow / prompt chaining

Code owns the control flow: Step A → validate → Step B → …
**Use when:** Stages are known, ordered, and need intermediate checks (procurement quote extract → policy validate → recommend).
**Pros:** Auditable, retryable per stage, clear SLAs.
**Cons:** More latency than one call. More engineering.

#### 3) Parallelization

Independent subtasks run at the same time. An aggregator merges the results.
**Use when:** No data dependencies exist between branches.
**Trap:** You run dependent stages in parallel (validate before extract finishes).

#### 4) Routing

A classifier or router sends work to specialized prompts, models, or workflows.
**Use when:** Heterogeneous intents share one front door.

#### 5) Evaluator–optimizer / critique loops

A generator produces. An evaluator scores against a rubric. Iterate N times or until pass.
**Use when:** Quality variance is high and latency allows.

#### 6) Agentic (model-driven control flow)

Claude plans, selects tools, and loops until a stop condition.
**Use when:** You cannot fully hardcode the steps. The environment is open-ended (research, debugging unknown codebases).
**Pros:** Flexibility.
**Cons:** Cost, non-determinism, and harder audit. You need budgets, tool allowlists, and HITL for high-impact actions.

#### 7) Multi-agent

Specialized agents with handoffs (researcher / writer / reviewer) or a supervisor pattern.
**Use when:** Clear role separation improves quality or security boundaries.
**Trap:** Multi-agent appearance: three agents do work that one workflow can do.

### D3. Pattern selection table

| Situation | Prefer | Avoid |
| --- | --- | --- |
| Fixed stages + compliance audit | Workflow + gates | Fully autonomous agent |
| Open-ended investigation | Agent + tool allowlist + budget | Brittle 12-step workflow |
| Independent doc sections | Parallel map-reduce | Serial chain |
| Mixed intents | Router | One mega-prompt |
| High-stakes irreversible action | Workflow + HITL approval | Agent with write tools unbound |
| Need citations from corpus | RAG + citation blocks | Pure parametric memory |

### D4. Decomposition techniques

1. **By business stage** (intake → enrich → decide → act → explain)
2. **By risk** (read-only research vs write actions)
3. **By data domain** (HR vs finance agents with separate credentials)
4. **By SLA** (fast path vs deep path)
5. **By confidence** (auto vs human queue)

Write **interfaces** between stages: schemas, idempotency keys, error codes. Exam scenarios reward designs that can retry stage 2. You do not redo stage 1.

### D5. Business value alignment

Tie architecture to measurable outcomes:

| Value pillar | Example metric | Design lever |
| --- | --- | --- |
| Efficiency | Minutes saved per case | Automation rate + HITL only on exceptions |
| Productivity | Tasks completed per person / throughput | Copilot adoption, automated drafts, faster review loops |
| Transformation | New capability enabled | Agents + tools that unlock research that was manual before |
| Cost | $ per successful task | Caching, smaller models, batch API |
| Performance | p95 latency | Parallelism, Haiku router, smaller context |
| Quality *(not in the guide's pillar list — quality stems live in the Evaluation domain)* | Error rate vs baseline | Workflow gates, eval harness, stronger model on hard route |

If a scenario lists SLAs, your answer must **name the lever** that protects the SLA.

### D6. Non-functional requirements checklist

- Availability & degradation mode
- Latency budgets per stage
- Cost ceilings / kill switches
- Data residency & retention
- Audit logging of prompts/tool calls (with redaction)
- Secrets handling (never in prompts)
- Rate limits & backoff
- Model deprecation / pin strategy

### D7. Worked mini-scenario (freight quotes)

**Need:** Extract quote fields → validate policy → recommend carrier. The trail must be auditable.
**Pattern:** A fixed **workflow** with three Claude (or Claude+code) stages and validators between them.
**Not:** A single monolithic autonomous agent. Not parallel independent extract, validate, and recommend.
**Why:** Dependencies and audit. This pattern appears often in public practice explanations. It matches Anthropic’s workflow-vs-agent distinction.

---

## Part E — Design documentation artifacts

For lifecycle handoff (file 04), you should produce:

1. **Context diagram** (systems Claude touches)
2. **Sequence diagram** for the happy path + one failure path
3. **Decision record** (pattern, model, why not alternatives)
4. **Threat notes** (injection, data leakage—deep dive in file 03)
5. **Eval plan** stub (metrics—deep dive in file 02)
6. **Runbook** pointers (on-call, rollback model pin)

Exam questions may ask what you communicate to executives vs engineers. Use the same design. Change the altitude.

---

## Decision trees & quick tables (exam desk reference)

### Model + pattern joint decision

| Quality need | Path predictability | Recommended shape |
| --- | --- | --- |
| High | High | Workflow + mid/high model on hard steps |
| High | Low | Agent + strong model + budgets + HITL |
| Medium | High | Workflow + mid model |
| Medium | Low | Router + agent for long tail |
| Low | Any | Fast model + simple prompt. Maybe no LLM |

### Context strategy picker

| Corpus size | Query diversity | Strategy |
| --- | --- | --- |
| Tiny (< context comfortably) | Low | Curated static context + cache |
| Large | Low | Metadata filters + light RAG |
| Large | High | Hybrid search RAG + rerank |
| Tools many | High | MCP + tool search / progressive discovery |

---

## Exam traps (Platform & Solution Design)

1. **Agent everywhere** — Prefer workflow when steps are known.
2. **Model with the most capability everywhere** — More capability than necessary fails cost/latency scenarios.
3. **Ignoring dependencies in parallel designs.**
4. **Treating prompt caching as free context** — Cached tokens still count toward the window.
5. **Monolithic context for enterprise knowledge bases.**
6. **No output schema** when systems consume results.
7. **Skipping business metrics** — Architecture without a value story.
8. **Forgetting degradation** — What happens when Claude/API is down?
9. **Multi-agent without boundaries** — Shared over-privileged tools.
10. **Changing cache-critical prefixes every request.**
11. **Putting secrets in prompts or tool args logs.**
12. **Equating “citations in prose” with grounded retrieval** — Prefer structured citations/search_result patterns from public docs.

---

## Practice Q&A (25)

**Q1.** When should you prefer a fixed workflow over an autonomous agent?
**A.** When stages are known, ordered, and need intermediate validation/audit.

**Q2.** A system must extract, then validate, then recommend. Parallel independent calls are proposed. What is wrong?
**A.** Later stages depend on earlier outputs. Parallelization breaks those dependencies.

**Q3.** Name three inputs to model selection.
**A.** Task complexity/quality bar, latency SLA, cost/volume (also context size, error cost).

**Q4.** Do prompt-cache tokens still count toward the context window?
**A.** Yes. Caching affects billing and latency. It does not change window occupancy.

**Q5.** What belongs in a system prompt’s “output contract”?
**A.** Schema/format, required fields, enums, citation rules, and uncertainty handling.

**Q6.** Large MCP tool catalogs hurt you how?
**A.** Tool definitions consume context. Prefer allowlists or tool search / progressive discovery.

**Q7.** What is progressive discovery?
**A.** Start with lean context. Retrieve or call tools to pull only the information that you need.

**Q8.** When are few-shot examples most justified?
**A.** To lock format, tone, or rare edge-case handling. They are not a substitute for retrieval of facts.

**Q9.** Give one valid use of a multi-agent design.
**A.** Separate roles with different permissions (for example, researcher read-only vs executor write with approval).

**Q10.** What should you do before promoting a larger model in production?
**A.** Prove a quality gap on a representative eval set. Measure cost and latency impact.

**Q11.** Why pin model IDs in production architectures?
**A.** Avoid surprise behavior changes. Manage deprecations with a plan.

**Q12.** What is an evaluator–optimizer loop?
**A.** Generate → score against a rubric → refine until pass or the budget is exhausted.

**Q13.** Name a business-value metric for an efficiency use case.
**A.** Average handle time, cases auto-closed, or minutes saved per ticket.

**Q14.** Where should volatile user content sit relative to a cacheable prefix?
**A.** After the stable prefix (end of prompt), so the cache prefix stays stable.

**Q15.** When is a single augmented LLM call enough?
**A.** Bounded task, low need for intermediate audit, and one reasoning pass is enough.

**Q16.** What risk does compaction introduce in long agents?
**A.** Summary drift. Critical constraints or facts may be lost if you do not protect them.

**Q17.** How do you protect a p95 latency SLA in a multi-stage workflow?
**A.** Budgets per stage, timeouts, parallel independent work, smaller models on easy stages, and async where UX allows.

**Q18.** Why separate machine JSON from user prose?
**A.** Parsers need stability. UX needs readability. Mixing causes brittle integrations.

**Q19.** What is a router pattern good for?
**A.** Heterogeneous intents behind one entrypoint with different prompts, models, or workflows.

**Q20.** Give a reason to choose cloud-hosted Claude (Bedrock/Vertex/Foundry) over direct API.
**A.** Enterprise procurement, existing cloud commit, or data residency/compliance controls (scenario-dependent).

**Q21.** What should a stage interface include for retries?
**A.** Typed schema, idempotency key, explicit error codes, and validated outputs.

**Q22.** Why might changing the thinking/effort configuration between turns be costly?
**A.** It invalidates message-level prompt-cache breakpoints (the tools/system prefix stays cached). This raises latency and spend. (Note: current models deprecate and remove fixed `budget_tokens` thinking budgets — adaptive thinking plus `effort` replaced them.)

**Q23.** What is the architect’s default stance on tool permissions?
**A.** Least privilege — only the tools needed for the role/stage.

**Q24.** How do Skills help teams?
**A.** Modular reusable instructions/resources reduce prompt sprawl and improve consistency.

**Q25.** A stakeholder wants “fully autonomous” processing of regulated approvals. What do you propose instead?
**A.** A workflow with policy checks and human-in-the-loop for high-impact approvals. Use an agent only for bounded research if needed.

---

## Pre-exam checklist (Platform & Solution Design)

- [ ] I can explain workflow vs agent with one crisp example each
- [ ] I can pick a model tier from quality/latency/cost constraints
- [ ] I can sketch end-to-end: ingress → orchestrate → Claude/tools → validate → act → telemetry
- [ ] I can design a cache-friendly prompt layout
- [ ] I can justify progressive discovery vs monolithic context
- [ ] I can write an output schema and validation gate
- [ ] I can map architecture choices to business metrics and SLAs
- [ ] I can list NFRs: residency, audit, degradation, rate limits, model pinning
- [ ] I know when multi-agent helps vs when it only looks useful.
- [ ] I can spot dependency mistakes in “parallelize everything” answers

---

## Glossary

| Term | Meaning |
| --- | --- |
| Augmented LLM | Single LLM call enriched with retrieval/tools |
| Workflow / prompt chaining | Code-orchestrated sequence of LLM/tool steps |
| Agent | Model-driven control flow with planning and tool loops |
| Multi-agent | Multiple specialized roles with handoffs |
| Context engineering | Deliberate selection, ordering, and budgeting of tokens |
| Prompt caching | Reusing billed/processed stable prompt prefixes |
| Progressive discovery | Fetch context on demand via retrieval/tools |
| Skills | Modular reusable instruction/resource packs |
| Output contract | Schema and rules for machine-consumable responses |
| Router | Component that dispatches to specialized paths |
| Effort / thinking | Controls for deeper reasoning vs latency/cost |
| Model pin | Fixed model ID for reproducible production behavior |
| NFR | Non-functional requirement (latency, cost, residency, etc.) |
| HITL | Human-in-the-loop review/approval |
| SLA / SLO | Service level agreement / objective |

---



---

## Part F — Deeper solution patterns & anti-patterns

### F1. Prompt chaining with programmatic gates

A production-grade chain is not “three prompts in a row.” It is:

1. **Deterministic prep** (parse upload, normalize units, load tenant config).
2. **LLM step** with a narrow contract.
3. **Programmatic validator** (schema, business rules, allowlists).
4. **Branch:** pass → next stage. Fail → repair prompt, retry with cap, or human queue.
5. **Idempotent side effects** only after the final gate.

This hybrid of LLM + code is what CCAR-P scenarios reward. Pure LLM self-checks help. They are weaker than typed validators for compliance narratives.

### F2. Map-reduce over documents

For large packets (RFPs, due diligence binders):

- **Map:** Same extract prompt per chunk/doc in parallel (mid/fast model).
- **Reduce:** A stronger model synthesizes with citations and conflict detection.
- **Verify:** Spot-check conflicting fields. Optional second-pass critique.

Architect concerns: chunk boundaries that split tables, consistent schemas across mappers, and reduce-stage token overruns.

### F3. Human-in-the-loop as an architectural component

HITL is not only a safety topic (file 03). In solution design it is a **first-class stage**:

| Trigger | UX pattern | Data to show the human |
| --- | --- | --- |
| Low model confidence | Review queue | Top uncertainties + source snippets |
| High-impact action | Explicit approve/deny | Diff of proposed write + policy cites |
| Novel intent | Specialist escalation | Conversation transcript + tools used |
| Eval regression | Shadow mode | Side-by-side old vs new outputs |

Design the queue’s SLAs and ownership. If you do not, the “architecture” fails in production.

### F4. Batch vs realtime

Public Batch APIs trade latency for cost and throughput. You select Batch when:

- Jobs are overnight analytics, offline evals, or corpus enrichment.
- The user does not wait for a synchronous UI.

Keep **the same prompts and schemas** between realtime and batch so evals transfer. Document different timeout and retry policies.

### F5. Anti-patterns library (memorize names)

1. Single mega-prompt: one prompt does extract, reason, act, and apology.
2. **Tool sprawl** — every SaaS verb is exposed to one agent.
3. Context dump: the entire SharePoint export sits in the system prompt.
4. **Eval-free launch** — “we will know quality when we see it.”
5. **Autonomy without budgets** — unbounded loops and spend.
6. **Silent fallback** — switch models without logging/telemetry.
7. **Schema optional** — free text into ERP.
8. **Security bolted on later** — authZ after tools already write.
9. Multi-agent imitation: three roles share one over-privileged toolkit.
10. Wasted latency: deep thinking on a yes/no classification.

### F6. Capacity and cost modeling (rough estimate)

For a workflow with stages S1..Sn:

- Tokens ≈ sum(input_i + output_i) × requests × (1 + retry_rate)
- Apply cache hit rate on stable prefixes
- Apply router fraction that escalates to expensive models
- Add tool/RAG infrastructure cost separately

Exam tip: if a scenario gives volume + latency, **mention caching, routing, or batching** as cost/latency levers even if you also select the right pattern.

### F7. Migration & model upgrade playbook

1. Pin current model ID.
2. Build shadow traffic or offline replay from production logs (redacted).
3. Compare task metrics + safety metrics.
4. Canary percentage with kill switch.
5. Update runbooks and prompt snapshots.
6. Only then change the pin.

Never “just flip the alias” in regulated environments without a plan.

### F8. Interface contracts between product and platform teams

Define:

- **Prompt/Skill ownership** and review cadence
- **Tool registry** ownership and change control
- **Eval suite** ownership and release gating
- **Incident severity** definitions for model misbehavior

CCAR-P blurs into lifecycle (file 04). In design answers, naming owners scores points.

---

## Part G — Prompt & context deep techniques

### G1. Separating instructions, data, and untrusted content

You should structurally separate:

- **Trusted instructions** (system / Skills / developer)
- **Retrieved documents** (data. May be malicious—injection risk)
- **User utterances** (intent. May be adversarial)
- **Tool outputs** (semi-trusted depending on the tool)

Use clear delimiters. Cite sources. Instruct the model to treat retrieved content as **untrusted data**. Pair this with technical controls in file 02/03 (allowlists, output encoding, privilege separation).

### G2. Structured reasoning without leaking chain-of-thought

Prefer platform **thinking** features for hidden reasoning when they are available. If you must show rationales to users, give a **user-facing explanation** field that is curated. Do not show raw chain-of-thought. This is especially true in regulated UX.

### G3. Determinism levers (practical, not absolute)

LLM calls are stochastic. Reduce variance with:

(Older models only) lower temperature. Current Claude tiers (Opus 4.7+/Opus 5/Sonnet 5/Fable 5) reject sampling parameters, so lean on the levers below instead.
- Strict schemas
- Few-shot anchors
- Workflow gates
- Ensemble/self-consistency only when cost allows

Do not promise bit-identical outputs in customer SLAs. Promise **process SLAs** and quality metrics.

### G4. Multimodal design notes

When inputs include images/PDFs:

- Decide whether a vision model call vs an OCR+text pipeline is better for tables.
- Bound image resolution and count for cost.
- Treat OCR text as untrusted retrieved content.
- Eval with real document scans, not only clean digital PDFs.

### G5. Internationalization & tone

Enterprise designs often need multilingual support. Public Claude models are multilingual. Still:

- Put language policy in the output contract (“respond in user’s language”).
- Eval per language. Quality is not uniform only because you declare it.
- Beware mixed-language retrieval corpora and embedding/search gaps (file 02).

---

## Part H — Additional worked scenarios

### Scenario H1 — Internal policy Q&A

**Requirements:** Employees ask HR policy questions. Answers must cite policy. Wrong legal advice is high risk.
**Design:** Progressive RAG over a versioned policy corpus + citation-required output + refuse when retrieval is empty + HITL for “exceptions/approvals” intents routed separately. Mid-tier model. Cache system+tool defs.
**Not:** Parametric-only answers from model memory. Not a fully autonomous agent with an email-send tool.

### Scenario H2 — Coding assistant for a monorepo

**Requirements:** Help developers navigate a large repo, propose patches, and run tests.
**Design:** Agent pattern with repository tools, a test runner, and strict write permissions to branches — not direct prod deploy. Strong coding model. Context via progressive file fetch, not a whole monorepo dump. Server-managed settings / approved MCP tools for enterprise Claude Code (details in file 04).
**Eval:** SWE-style task suites + security regression (no secret exfiltration).

### Scenario H3 — Customer support copilot

**Requirements:** Suggest replies. Agents send. p95 < 2s for draft.
**Design:** Fast model for draft. Retrieval of account + macros. Optional escalate to mid model for angry/VIP. Workflow, not an open agent. Cache macros/policies. HITL is the human agent (always-on).
**Metrics:** Edit distance, CSAT, handle time, hallucination rate on factual fields.

### Scenario H4 — Overnight contract clause extraction

**Requirements:** 50k PDFs nightly. Cost-sensitive. Morning dashboard.
**Design:** Batch API + map-reduce workflow + schema validation + exception queue. Not interactive Opus-class chat UX.

---

## Part I — Extended practice Q&A (26–35)

**Q26.** What is the difference between a process SLA and promising identical LLM strings?
**A.** Process SLAs cover latency, availability, and quality rates. Identical strings are unrealistic for generative models.

**Q27.** Why run OCR before or instead of pure vision for dense tables?
**A.** Tables often extract more reliably via OCR+structure pipelines. Validate with evals either way.

**Q28.** What belongs in a model upgrade canary plan?
**A.** Shadow/replay metrics, percentage rollout, kill switch, prompt pin alignment, and rollback owner.

**Q29.** How do you prevent “context excessive content”?
**A.** Retrieval filters, top-k caps, relevance thresholds, progressive discovery, and periodic corpus hygiene.

**Q30.** When is Batch API the wrong choice?
**A.** When the end user needs interactive, low-latency responses.

**Q31.** Name two variance-reduction techniques for extraction.
**A.** JSON schema validation + few-shot anchors (also deterministic post-processors). Temperature tuning is not a lever on current Claude tiers — those tiers reject sampling parameters there.

**Q32.** Why separate research-agent credentials from execute-agent credentials?
**A.** Least privilege and blast-radius control if the research path is prompt-injected.

**Q33.** What should you log for an architectural incident review (at minimum)?
**A.** Model ID, prompt/Skill versions, tool calls (redacted), retrieval IDs, latency, and decision outcome.

**Q34.** A design uses three agents that all share admin SaaS tokens. Primary issue?
**A.** Over-privilege / missing security boundaries — multi-agent without isolation.

**Q35.** How do Skills interact with caching strategy?
**A.** Stable Skill content can live in cacheable prefixes. Version carefully so cache and behavior stay coherent.

---

## Expanded checklist additions

- [ ] I can describe map-reduce over documents with schema consistency risks
- [ ] I can place HITL triggers in a diagram as first-class stages
- [ ] I can rough-order cost with cache hit rate and escalation fraction
- [ ] I can write a model migration canary outline
- [ ] I can separate trusted instructions from untrusted retrieved text
- [ ] I can choose Batch vs realtime with a one-line rationale
- [ ] I can name five anti-patterns and their fixes
- [ ] I can design a support copilot that meets a 2s draft latency via model routing

---

## Glossary additions

| Term | Meaning |
| --- | --- |
| Programmatic gate | Code validation between LLM stages |
| Map-reduce LLM | Parallel per-doc extract + synthesize |
| Shadow traffic | Parallel scoring of a candidate model/prompt |
| Canary release | Gradual production exposure with rollback |
| God prompt | Anti-pattern: one prompt does all jobs |
| Tool sprawl | Excessive tools exposed to one agent |
| Process SLA | Operational targets without bit-identical outputs |
| Untrusted context | Retrieved/user content that may contain injections |
| Exception queue | Human review path for failed gates |
| Prompt/Skill pin | Version identifier for reproducible instructions |

---

*(Deepening passes continue below: Parts J–R.)*


---

## Part J — Primary-study deep dive: architecture decision scenarios

Use these as timed drills. Write a 6–10 sentence decision. Then check the scoring keys.

### Scenario J1 — Claims triage under a 3-day SLA

**Context:** An insurer wants Claude to cut claims cycle time from 11 days to 3. Volume is 8k claims/day. Regulators require an auditable decision trail. Payouts above $5k need a human signature. Corpus: policy PDFs + structured claim forms.

**Decide:** pattern per stage, model tiering, HITL placement, success metrics.

**Scoring key (preferred shape):**
1. **Workflow** with stages: ingest/OCR → extract fields → policy retrieve+cite → recommend → (HITL if payout ≥ $5k or low confidence).
2. **Router:** Haiku-class for intent/confidence. Sonnet-class for extract+recommend. Escalate hard disputes to higher-capability only when evals justify.
3. Metrics: cycle time, auto-resolution rate below threshold, citation coverage, overturn rate by humans, p95 latency of auto path, $ per claim.
4. Anti-patterns: a single autonomous agent with write tools to payment systems. Monolithic “paste whole policy book” context.

### Scenario J2 — Internal coding assistant for a 40-repo org

**Context:** Platform team wants Claude across IDEs/CLI. Secrets must not leave the vault. Some repos are regulated. Latency for autocomplete help must stay interactive. Deep refactors can be async.

**Decide:** progressive vs monolithic context. Skills vs giant CLAUDE.md. Model routing.

**Scoring key:** Progressive discovery via repo `CLAUDE.md` + retrieval over docs + MCP only for approved tools. Separate **interactive** mid-tier path from **deep agent** high-capability path with budgets. Regulated repos: stricter allowlists + no server web tools. Org Skills for ADR/test templates. Avoid one 50k-token global prompt.

### Scenario J3 — Multilingual support copilot with CRM tools

**Context:** 12 languages, CRM write actions (case notes, refunds ≤ $50 auto). Brand tone is critical. Hallucinated refunds are catastrophic.

**Decide:** pattern, output contract, tool authZ, eval focus.

**Scoring key:** Augmented LLM + tools inside a **workflow gate** for refunds (schema validation + policy engine + amount cap). Do not let the model “decide” refund eligibility alone — code enforces caps. Eval: language quality sample + tool-call correctness + refusal on out-of-policy. Citations are not primary. **Action correctness** is.

### Scenario J4 — Overnight batch contract clause extraction

**Context:** 200k PDFs/night. Structured JSON to warehouse. 98% field accuracy target. Cost sensitive.

**Decide:** Batch vs realtime API. Map-reduce. Model tier. Validation.

**Scoring key:** Batch API + map-reduce (per-doc extract → schema validate → quarantine fails → sample human audit). Prefer mid-tier with strong schema unless hard clauses need escalate. Cost model before GA. Not an interactive agent.

---

## Part K — Trade-off tables (desk reference)

### K1. Capability vs latency vs cost (operator view)

| Lever | Improves | Often worsens | Exam cue |
| --- | --- | --- | --- |
| Higher-capability model | Hard reasoning, coding, judgment | Latency, $ | High-stakes offline review |
| Faster/cheaper model | p95, throughput, $ | Edge-case quality | Classify/extract/route |
| Thinking / higher effort | Multi-step correctness | Tokens, latency, cache stability | Hard math/logic/planning |
| Prompt caching | $ and TTFT on stable prefixes | Design discipline (prefix order) | Large static system+tools |
| Smaller context | Latency, focus | Completeness | Progressive discovery |
| Parallel stages | Wall-clock | $ and merge complexity | Independent branches |
| Batch API | Unit economics | Interactivity | Overnight jobs |
| Stronger eval gates | Quality, safety | Time-to-ship | Regulated GA |

### K2. Pattern vs auditability

| Pattern | Auditability | Flexibility | Typical cost variance |
| --- | --- | --- | --- |
| Augmented LLM | High (one call) | Low | Low |
| Workflow | Very high (stage logs) | Medium | Medium |
| Agent | Medium (needs traces) | High | High |
| Multi-agent | Needs hop IDs | Highest | Highest |

**General rule:** prefer the leftmost pattern that meets the requirement. Move right only when evals show the left fails.

### K3. Context strategy trade-offs

| Strategy | Token efficiency | Completeness risk | Ops complexity |
| --- | --- | --- | --- |
| Monolithic dump | Poor at scale | Low (if fits) | Low initially, blows up later |
| Progressive RAG | Good | Retrieval misses | Medium |
| Tool/MCP discovery | Good | Tool selection errors | Medium–high |
| Hybrid cached core + dynamic | Best common case | Dual-path bugs | Medium |

---

## Part L — Failure-mode analysis (platform & design)

| Failure mode | Symptoms | Likely root | Design fix |
| --- | --- | --- | --- |
| Lost-in-the-middle | Ignores middle docs | Too much packed context | Rank, shrink k, progressive |
| Format drift | Downstream parse fails | Weak output contract | Schema + validator + repair loop |
| Cache thrash | Cost spikes, slow TTFT | Volatile prefix / thinking flips | Stabilize prefix & configs |
| Over-agentification | Unpredictable writes | Agent where workflow fits | Convert to staged workflow |
| Model mismatch | Great demos, prod misses | Eval on toy data | Production-like eval set |
| Context starvation | “I do not know” wrongly | Over-aggressive filtering | Relax filters. Query rewrite |
| Skill/prompt conflict | Contradictory behavior | Multiple instruction layers | Precedence doc + tests |
| Multimodal OCR fail | Bad extracts from scans | Vision without validation | Confidence + human queue |
| Router collapse | Everything escalates | Bad confidence calibration | Threshold tuning + metrics |
| Compaction drift | Long agents forget rules | Summary dropped policy | Pin never-compact sections |

---

## Part M — Production checklist (Platform & Solution Design)

### M1. Before architecture review

- [ ] Problem statement, users, and “non-goals” written
- [ ] Value pillars mapped to metrics and owners
- [ ] Pattern choice with rejected alternatives documented (ADR)
- [ ] Model tier per stage with cost envelope
- [ ] Context budget diagram (static vs dynamic vs tools vs thinking)
- [ ] Output schemas and validation ownership named
- [ ] Degradation mode if Claude or tools unavailable
- [ ] Eval stub attached (even if thin)

### M2. Before pilot

- [ ] Golden set of ≥50–100 realistic cases
- [ ] Latency budget per stage signed by product
- [ ] Cost kill-switch / max tokens per request
- [ ] Logging redaction policy for prompts/tool results
- [ ] Model pin / upgrade policy draft
- [ ] HITL queues staffed for expected exception rate

### M3. Before GA

- [ ] Shadow or A/B vs baseline complete
- [ ] Failure modes above reviewed with security/ops
- [ ] Runbook + on-call named
- [ ] Stakeholder sign-off on known limitations (file 04)
- [ ] Continuous eval in CI for prompts/tools/schemas

---

## Part N — Additional architecture decision drills (short)

1. **Fixed KYC checklist with regulators watching** → Workflow + validators. Not a free-form agent.
2. **Unknown-root-cause production incident research** → Agent + read-only tools + budget + HITL for remediations.
3. **One front door for HR/IT/Facilities FAQs** → Router + per-domain RAG corpora + separate credentials.
4. **Generate 10 independent product descriptions** → Parallel map. Aggregate. Schema validate.
5. **High-stakes legal memo** → High-capability + RAG citations + HITL attorney. Never auto-send.

---

## Part O — Extended Q&A (36–40)

**Q36.** A stakeholder wants “one mega agent” to replace a 7-stage underwriting checklist that auditors inspect. Best response?
**A.** Propose a **workflow** that mirrors the checklist with logged stage outputs. Reserve agents for open research sub-tasks only. Auditability beats flexibility here.

**Q37.** Interactive UX needs p95 < 2s, but hard cases need deep reasoning. Best design?
**A.** **Router**: fast path mid/fast model. Async or “continue analyzing” path for deep thinking. Communicate UX expectation.

**Q38.** Cache hit rate is 5% despite a huge system prompt. Most likely cause?
**A.** Volatile content or tool defs inserted **before** the stable prefix, or thinking/config changes breakpoints every call.

**Q39.** Select TWO for map-reduce over 500 contracts. Option (a) is parallel per-doc extract. Option (b) is one call with all PDFs. Option (c) is schema validate each. Option (d) is single agent with write-to-ERP unbound.
**A.** **(a)** and **(c)**.

**Q40.** Skills vs stuffing expertise into every user message—when Skills win?
**A.** When expertise is **stable, reusable, versioned** across many requests. Keep request-specific facts in user/tool channels.

---

## Part P — Rapid review card (Domain 1+2 ≈ 30%)

- Smallest model that passes evals. Escalate with evidence.
- Workflow when steps are known. Agent when they are not. Multi-agent only with clear roles.
- System prompt = policy + interface contract.
- Progressive discovery for large tools/corpora. Cache stable prefixes.
- Value pillars must name levers (efficiency, transformation, productivity, cost, performance SLAs).
- Document ADRs. Reject alternatives explicitly.
- Validate machine outputs in code—never hope.
- Compaction/thinking/caching interactions are design concerns, not afterthoughts.

*Cross-read Integration/Evals in `02`, Safety in `03`. (Parts Q–R continue below.)*


---

## Part Q — Model routing & cost envelopes (worked numbers)

You need rough estimate math. You do not need memorized price sheets (prices change—verify docs).

### Cost envelope worksheet

```
Monthly tasks T
× tokens_in_avg × ($/M input) 
+ tokens_out_avg × ($/M output)
+ cache_write share × write_rate + cache_read share × read_rate
+ tool_overhead_tokens
= monthly model $
```

Add: retrieval infra, HITL labor, observability storage. Compare to value per successful task.

### Routing example (illustrative proportions)

| Route | % traffic | Model class | Notes |
| --- | --- | --- | --- |
| Classify/refuse | 40% | Fast | Gatekeeper |
| Standard answer | 45% | Mid | Cached policy prefix |
| Hard reason / dispute | 15% | High | Thinking on. Async OK |

If the hard route is 60% of cost but only 15% of volume, eval whether mid+critique is cheaper for a slice.

### When Batch API wins

High volume, latency-insensitive, structured jobs (extraction, classification, overnight reports). Not for chat UX. Pair with schema validation and quarantine queues.

### Pinning & upgrades

Pin model IDs in prod. Upgrade via shadow → eval gate → canary → full. Treat prompt and model as a **coupled** release artifact.

---

## Part R — Additional failure modes & exam stems

| Stem cue | Prefer |
| --- | --- |
| “Auditors need stage evidence” | Workflow + logs |
| “Steps unknown. Explore codebase” | Agent + budget + allowlist |
| “12 intents one bot” | Router |
| “Huge PDF corpus” | RAG progressive, not monolith |
| “p95 interactive” | Fast/mid + defer deep thinking |
| “$ sensitive overnight” | Batch + mid + validate |

**Q41.** Cache reads are cheap, so put the user question first in every request for readability—good idea?
**A.** **No**—put **stable** content first for caching. Volatile user content last.

**Q42.** Select TWO decomposition axes: by risk (read vs write), by style preference, by SLA path.
**A.** By risk and by SLA path.

**Q43.** Output must feed a payment API—what is mandatory?
**A.** **Schema validation in code** before side effects. Enums for status. Idempotency keys.

*End of file 01. Next: `02-enterprise-integration-production.md` (Integration 19% + Evaluation 16%).*
