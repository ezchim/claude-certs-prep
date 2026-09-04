---
title: Claude Platform & Solution Design
---

# 01 — Claude Platform & Solution Design

**CCAR-P condensed domain 1** 
**Official domains mapped here:** Solution Design & Architecture (**17%**) · Claude Models, Prompting & Context Engineering (**13%**) 
**Combined exam weight covered by this file:** ~**30%**

---

## Disclaimer

These are **original study notes** for exam preparation. They are grounded in **public** Anthropic platform documentation, published CCAR-P blueprint summaries, and widely taught architecture patterns for LLM systems. They do **not** reproduce official course content. Model names, pricing, and context limits change; always reconcile numbers with current [platform.claude.com](https://platform.claude.com) docs before production decisions or exam day. This pack is independent and not affiliated with Anthropic.

---

## Overview

CCAR-P expects you to think like a **production architect**, not a prompt tinkerer. In scenarios you will:

1. Translate a messy business problem into a **Claude-powered solution shape** (inputs → processing → outputs → feedback).
2. Choose among **workflow**, **augmented LLM**, **agentic**, and **multi-agent** patterns with explicit trade-offs.
3. Select a **model tier** (capability vs latency vs cost vs context) and justify it.
4. Engineer **prompts and context** so the system is reliable under token, safety, and reuse constraints.
5. Connect design choices to **business value pillars** — per the official guide's wording: efficiency, transformation, **productivity**, cost, and performance SLAs.

This file deep-dives platform capabilities and solution design. Integration mechanics (MCP wiring, RAG pipelines, eval harnesses) appear in file `02`; governance in `03`; stakeholder/lifecycle/enablement in `04`.

---

## Key map (user domain ↔ official tasks)

| Architect task | Official domain | What “good” looks like on the exam |
| --- | --- | --- |
| Problem → solution framing | Solution Design 17% | Clear value hypothesis, constraints, success metrics |
| Pattern selection (workflow/agent/…) | Solution Design 17% | Prefer simplest pattern that meets audibility & reliability needs |
| Decomposition & orchestration | Solution Design 17% | Stages, gates, retries, human checkpoints where risk warrants |
| Model selection | Models/Prompting 13% | Match complexity, latency budget, context size, cost |
| System prompts & behavioral controls | Models/Prompting 13% | Role, constraints, output schema, refusal boundaries |
| Few-shot / structured reasoning | Models/Prompting 13% | Examples when format/edge cases matter; thinking for hard reasoning |
| Context window strategy | Models/Prompting 13% | Progressive discovery vs curated context; caching & Skills for reuse |
| Prompt caching / modular prompts / Skills | Models/Prompting 13% | Reuse static prefixes; avoid stuffing every request |

---

## Part A — Claude models for architects

### A1. How to think about the model ladder

Public docs describe a **family** of Claude models with different speed/intelligence/cost points. Exact IDs and prices move over time; for CCAR-P, memorize the **decision logic**, not a frozen price sheet:

| Decision factor | Prefer higher-capability tier when… | Prefer faster/cheaper tier when… |
| --- | --- | --- |
| Task complexity | Multi-step planning, hard coding, ambiguous requirements, high-stakes judgment | Classification, extraction, routing, short rewriting |
| Latency SLA | Offline batch, async jobs, human-reviewed drafts | Interactive UX, high QPS gatekeepers |
| Context size | Large codebases, long docs, multi-hour agent traces | Short turns, tool-heavy loops with small payloads |
| Cost envelope | Low volume / high value (contracts, architecture reviews) | High volume / low marginal value |
| Error cost | Wrong answer is expensive or unsafe | Wrong answer is cheap to catch/retry |

**Architect rule:** Default to the **smallest model that meets quality bars on your eval set**. Promote capability only when evals prove a gap. Demote when latency/cost budgets fail and quality still holds.

### A2. Capability knobs that show up in designs

Public platform features architects must place correctly:

- **Context windows:** Current frontier Claude models commonly advertise **large** windows (hundreds of thousands to **~1M tokens** on supported models/platforms). Everything in the request counts: system prompt, messages, tool defs, images/docs, tool results, and (when used) thinking tokens. Caching changes **price**, not whether tokens occupy the window.
- **Max output:** Synchronous Messages API vs Batch (batch may support extended output via documented beta headers—verify current docs).
- **Thinking / adaptive thinking / effort:** Use deeper reasoning for hard problems; control cost/latency with **effort levels** (`output_config.effort`). Adaptive thinking replaced fixed thinking budgets — `budget_tokens` is deprecated on the 4.6 generation and removed (rejected with an error) on current models. Changing thinking config invalidates message-level prompt-cache breakpoints (the tools/system prefix stays cached)—design for cache stability.
- **Vision / multimodal:** Image input where the product needs screenshots, diagrams, or scanned docs.
- **Deployment surfaces:** Claude API, Amazon Bedrock, Vertex AI, Microsoft Foundry, Claude Platform on AWS—each with ID naming and data-residency nuances. Architects pick based on enterprise procurement, region, and compliance—not brand preference alone. **Full CSP / delivery-route governance decision table lives in file `02` Part D5** (constraint elimination, ZDR themes, model-ID traps, Q&A).

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

**Exam trap:** Choosing Opus/Fable-class for every step of a deterministic ETL-like pipeline. Prefer **workflow + smaller model** with validation gates.

### A4. Multi-model architectures

Common production shapes:

1. **Router → specialist:** Cheap classifier chooses extract vs reason vs refuse. 
2. **Draft → critique:** Mid model drafts; stronger model or rubric critic reviews. 
3. **Shadow eval:** New model runs in parallel; compare offline before cutover. 
4. **Fallback:** Primary timeout → degraded model or cached answer + human queue.

Document **why** each hop exists (quality, cost, safety). Orphan hops fail reviews and fail exam scenarios that ask for simplest viable design.

---

## Part B — Prompting & behavioral control

### B1. System prompt as architecture, not poetry

Treat the system prompt as a **policy + interface contract**:

| Section | Purpose | Example content (illustrative) |
| --- | --- | --- |
| Role & scope | Bound the job | “You are a procurement assistant for freight quotes. You do not negotiate contracts.” |
| Tools policy | When to call what | “Call `get_policy` before recommending. Never invent policy clauses.” |
| Output contract | Schema / format | JSON schema, citation requirements, severity enums |
| Safety / refusal | Local guardrails | Disallowed actions; escalation phrases |
| Quality bar | How to handle uncertainty | “If confidence low, ask clarifying question or return `needs_review`.” |
| Style | Audience | Exec brief vs engineer debug |

**Prefer:** Explicit constraints + structured outputs. 
**Avoid:** Vague “be helpful and thorough” without success criteria.

### B2. Prompting techniques architects should name correctly

| Technique | When to use | Failure mode |
| --- | --- | --- |
| Zero-shot | Clear tasks, strong base capability | Ambiguous formats drift |
| Few-shot | Format, tone, edge-case handling | Bad examples become law; token bloat |
| Chain-of-thought / structured reasoning | Multi-step logic (prefer API thinking features where available) | Leaking chain into user-facing channel; cost |
| Decomposition in prompt | Single call must cover subskills | Still weaker than real workflow gates for audit |
| Tool-forced (`tool_choice`) | Retrieval or action must happen | Over-forcing when optional tools exist |
| Constitutional / principle prompts | Local safety/style overlays | Conflicts with org policy if not reviewed |

### B3. Output contracts and machine consumers

If a downstream system parses Claude’s output, design for **machines first**:

- Prefer **JSON / typed schemas** with validation before side effects. 
- Separate **user-facing prose** from **machine fields**. 
- Include explicit enums for status (`ok`, `needs_review`, `refused`). 
- Validate with code; never “hope” the model obeyed.

### B4. Skills and modular prompt reuse (public product concept)

Public Claude product docs describe **Skills** (and related modular instruction packs) as reusable specialized instructions/resources so teams do not paste giant instruction blobs into every request. Architect takeaway for CCAR-P:

- Put **stable** expertise into modular reusable units. 
- Keep **request-specific** data in the user/tool channel. 
- Version Skills like code; review for prompt-injection surfaces when Skills can pull external content.

---

## Part C — Context engineering

### C5. What counts toward the window

Everything sent counts. Architects must budget:

1. System / Skills / static policy 
2. Tool definitions (large MCP tool catalogs can dominate) 
3. Conversation history 
4. Retrieved chunks / files 
5. Tool results 
6. Thinking tokens (when enabled)

**Token counting API** (public) exists to estimate before send—use it in design reviews for worst-case prompts.

### C6. Progressive discovery vs monolithic context

| Strategy | Idea | Best for | Risk |
| --- | --- | --- | --- |
| Monolithic context | Stuff “everything possibly relevant” into the prompt | Small corpora, strong need for global consistency | Noise, cost, lost-in-the-middle, cache thrash |
| Progressive discovery | Start lean; retrieve/tools pull what is needed | Large corpora, tool ecosystems, agents | Missing retrieval → incomplete answers |
| Hybrid | Cached core policy + dynamic retrieval | Most enterprise apps | Complexity of two paths |

**Exam preference:** When corpora or tool sets are large, prefer **progressive discovery** (RAG/tools/MCP tool search) over dumping megabytes of text. When the task is short and closed-world, a curated static context can be simpler and more auditable.

### C7. Prompt caching (architect view)

Public docs: prompt caching reduces cost/latency for repeated prefixes (system prompts, large reference docs, tool defs). Cache writes cost more than ordinary input; cache **reads** are heavily discounted. TTLs commonly include short (e.g., ~5 minutes) and longer (e.g., ~1 hour) options—confirm current docs.

**Design rules:**

- Put **stable content first** (cacheable prefix). 
- Put **volatile user content last**. 
- Avoid reshuffling tool defs every request. 
- Changing thinking budgets/modes can invalidate breakpoints—stabilize configs in hot paths. 
- Remember: cached tokens **still consume** context window capacity.

### C8. Compaction / context editing

For long-running agents and chats, public platform features include **compaction** (server-side summarization near limits) and **context editing** strategies (e.g., clearing old tool results). Architects should:

- Define what must **never** be summarized away (safety policy, user entitlements). 
- Log when compaction occurs for audit. 
- Test agent quality after compaction—summary drift is a real failure mode.

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

Map each box to owners (app team, platform, security, data). Missing owners = lifecycle failure (see file 04).

### D2. Pattern catalog (know these cold)

Anthropic-style teaching (and exam scenarios) distinguish **predefined workflows** from **agents that decide their own path**.

#### 1) Augmented LLM (single call + light tools)

One Claude call, maybe with retrieval or a tool or two. 
**Use when:** Task is bounded; one reasoning pass suffices. 
**Pros:** Simple, cheap, easy to eval. 
**Cons:** Weak for multi-stage audit; limited recovery.

#### 2) Workflow / prompt chaining

Code owns the control flow: Step A → validate → Step B → … 
**Use when:** Stages are known, ordered, and need intermediate checks (procurement quote extract → policy validate → recommend). 
**Pros:** Auditable, retryable per stage, clear SLAs. 
**Cons:** More latency than one call; more engineering.

#### 3) Parallelization

Independent subtasks run concurrently; aggregator merges. 
**Use when:** No data dependencies between branches. 
**Trap:** Parallelizing dependent stages (validate before extract finishes).

#### 4) Routing

Classifier/router sends to specialized prompts, models, or workflows. 
**Use when:** Heterogeneous intents share one front door.

#### 5) Evaluator–optimizer / critique loops

Generator produces; evaluator scores against rubric; iterate N times or until pass. 
**Use when:** Quality variance is high and latency allows.

#### 6) Agentic (model-driven control flow)

Claude plans, chooses tools, and loops until stop condition. 
**Use when:** Steps cannot be fully hardcoded; environment is open-ended (research, debugging unknown codebases). 
**Pros:** Flexibility. 
**Cons:** Cost, non-determinism, harder audit—needs budgets, tool allowlists, HITL for high impact.

#### 7) Multi-agent

Specialized agents with handoffs (researcher / writer / reviewer) or supervisor pattern. 
**Use when:** Clear role separation improves quality or security boundaries. 
**Trap:** Multi-agent theater—three agents doing what one workflow could do.

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

Write **interfaces** between stages: schemas, idempotency keys, error codes. Exam scenarios reward designs that can retry stage 2 without redoing stage 1.

### D5. Business value alignment

Tie architecture to measurable outcomes:

| Value pillar | Example metric | Design lever |
| --- | --- | --- |
| Efficiency | Minutes saved per case | Automation rate + HITL only on exceptions |
| Productivity | Tasks completed per person / throughput | Copilot adoption, automated drafts, faster review loops |
| Transformation | New capability enabled | Agents + tools unlocking previously manual research |
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

**Need:** Extract quote fields → validate policy → recommend carrier; auditable. 
**Pattern:** Fixed **workflow** with three Claude (or Claude+code) stages and validators between them. 
**Not:** Single monolithic autonomous agent; not parallel independent extract/validate/recommend. 
**Why:** Dependencies + audit. This pattern appears repeatedly in public practice explanations and matches Anthropic’s workflow-vs-agent distinction.

---

## Part E — Design documentation artifacts

For lifecycle handoff (file 04), architects should produce:

1. **Context diagram** (systems Claude touches) 
2. **Sequence diagram** for the happy path + one failure path 
3. **Decision record** (pattern, model, why not alternatives) 
4. **Threat notes** (injection, data leakage—deep dive in file 03) 
5. **Eval plan** stub (metrics—deep dive in file 02) 
6. **Runbook** pointers (on-call, rollback model pin)

Exam questions may ask what to communicate to executives vs engineers—same design, different altitude.

---

## Decision trees & quick tables (exam desk reference)

### Model + pattern joint decision

| Quality need | Path predictability | Recommended shape |
| --- | --- | --- |
| High | High | Workflow + mid/high model on hard steps |
| High | Low | Agent + strong model + budgets + HITL |
| Medium | High | Workflow + mid model |
| Medium | Low | Router + agent for long tail |
| Low | Any | Fast model + simple prompt; maybe no LLM |

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
2. **Biggest model everywhere** — Overkill fails cost/latency scenarios. 
3. **Ignoring dependencies in parallel designs.** 
4. **Treating prompt caching as free context** — Still counts toward window. 
5. **Monolithic context for enterprise knowledge bases.** 
6. **No output schema** when systems consume results. 
7. **Skipping business metrics** — Architecture without value story. 
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
**A.** Later stages depend on earlier outputs; parallelization breaks dependencies.

**Q3.** Name three inputs to model selection. 
**A.** Task complexity/quality bar, latency SLA, cost/volume (also context size, error cost).

**Q4.** Do prompt-cache tokens still count toward the context window? 
**A.** Yes—caching affects billing/latency, not window occupancy.

**Q5.** What belongs in a system prompt’s “output contract”? 
**A.** Schema/format, required fields, enums, citation rules, uncertainty handling.

**Q6.** Large MCP tool catalogs hurt you how? 
**A.** Tool definitions consume context; prefer allowlists or tool search / progressive discovery.

**Q7.** What is progressive discovery? 
**A.** Start with lean context; retrieve or call tools to pull only needed information.

**Q8.** When are few-shot examples most justified? 
**A.** To lock format, tone, or rare edge-case handling—not as a substitute for retrieval of facts.

**Q9.** Give one valid use of a multi-agent design. 
**A.** Separating roles with different permissions (e.g., researcher read-only vs executor write with approval).

**Q10.** What should you do before promoting a larger model in production? 
**A.** Prove a quality gap on a representative eval set; measure cost/latency impact.

**Q11.** Why pin model IDs in production architectures? 
**A.** Avoid surprise behavior changes; manage deprecations deliberately.

**Q12.** What is an evaluator–optimizer loop? 
**A.** Generate → score against rubric → refine until pass or budget exhausted.

**Q13.** Name a business-value metric for an efficiency use case. 
**A.** Average handle time, cases auto-closed, or minutes saved per ticket.

**Q14.** Where should volatile user content sit relative to a cacheable prefix? 
**A.** After the stable prefix (end of prompt), so the cache prefix remains stable.

**Q15.** When is a single augmented LLM call enough? 
**A.** Bounded task, low need for intermediate audit, one reasoning pass suffices.

**Q16.** What risk does compaction introduce in long agents? 
**A.** Summary drift—critical constraints or facts may be lost if not protected.

**Q17.** How do you protect a p95 latency SLA in a multi-stage workflow? 
**A.** Budgets per stage, timeouts, parallel independent work, smaller models on easy stages, async where UX allows.

**Q18.** Why separate machine JSON from user prose? 
**A.** Parsers need stability; UX needs readability—mixing causes brittle integrations.

**Q19.** What is a router pattern good for? 
**A.** Heterogeneous intents behind one entrypoint with different prompts/models/workflows.

**Q20.** Give a reason to choose cloud-hosted Claude (Bedrock/Vertex/Foundry) over direct API. 
**A.** Enterprise procurement, existing cloud commit, data residency/compliance controls (scenario-dependent).

**Q21.** What should a stage interface include for retries? 
**A.** Typed schema, idempotency key, explicit error codes, and validated outputs.

**Q22.** Why might changing the thinking/effort configuration between turns be costly? 
**A.** It invalidates message-level prompt-cache breakpoints (the tools/system prefix stays cached), increasing latency and spend. (Note: fixed `budget_tokens` thinking budgets are deprecated/removed on current models — adaptive thinking plus `effort` replaced them.)

**Q23.** What is the architect’s default stance on tool permissions? 
**A.** Least privilege—only tools needed for the role/stage.

**Q24.** How do Skills help teams? 
**A.** Modular reusable instructions/resources reduce prompt sprawl and improve consistency.

**Q25.** A stakeholder wants “fully autonomous” processing of regulated approvals. What do you propose instead? 
**A.** Workflow with policy checks and human-in-the-loop for high-impact approvals; agent only for bounded research if needed.

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
- [ ] I know when multi-agent helps vs when it is theater 
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
4. **Branch:** pass → next stage; fail → repair prompt, retry with cap, or human queue. 
5. **Idempotent side effects** only after final gate.

This hybrid of LLM + code is what CCAR-P scenarios reward. Pure LLM self-checks help but are weaker than typed validators for compliance narratives.

### F2. Map-reduce over documents

For large packets (RFPs, due diligence binders):

- **Map:** Same extract prompt per chunk/doc in parallel (mid/fast model). 
- **Reduce:** Stronger model synthesizes with citations and conflict detection. 
- **Verify:** Spot-check conflicting fields; optional second-pass critique.

Architect concerns: chunk boundaries that split tables, consistent schemas across mappers, and reduce-stage token blowups.

### F3. Human-in-the-loop as an architectural component

HITL is not only a safety topic (file 03). In solution design it is a **first-class stage**:

| Trigger | UX pattern | Data to show the human |
| --- | --- | --- |
| Low model confidence | Review queue | Top uncertainties + source snippets |
| High-impact action | Explicit approve/deny | Diff of proposed write + policy cites |
| Novel intent | Specialist escalation | Conversation transcript + tools used |
| Eval regression | Shadow mode | Side-by-side old vs new outputs |

Design the queue’s SLAs and ownership or the “architecture” fails in production.

### F4. Batch vs realtime

Public Batch APIs trade latency for cost/throughput. Architects choose Batch when:

- Jobs are overnight analytics, offline evals, corpus enrichment. 
- User is not waiting on a synchronous UI. 

Keep **the same prompts and schemas** between realtime and batch so evals transfer. Document different timeout and retry policies.

### F5. Anti-patterns library (memorize names)

1. **God prompt** — one prompt owns extract, reason, act, and apologize. 
2. **Tool sprawl** — every SaaS verb exposed to one agent. 
3. **Context landfill** — entire SharePoint dump in system prompt. 
4. **Eval-free launch** — “we’ll know quality when we see it.” 
5. **Autonomy without budgets** — unbounded loops and spend. 
6. **Silent fallback** — switch models without logging/telemetry. 
7. **Schema optional** — free text into ERP. 
8. **Security bolted on later** — authZ after tools already write. 
9. **Multi-agent cosplay** — three roles, one shared over-privileged toolkit. 
10. **Latency theater** — deep thinking on a yes/no classification.

### F6. Capacity & cost modeling (back-of-envelope)

For a workflow with stages S1..Sn:

- Tokens ≈ sum(input_i + output_i) × requests × (1 + retry_rate) 
- Apply cache hit rate on stable prefixes 
- Apply router fraction that escalates to expensive models 
- Add tool/RAG infrastructure cost separately 

Exam tip: if a scenario gives volume + latency, **mention caching, routing, or batching** as cost/latency levers even if you also pick the right pattern.

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

CCAR-P blurs into lifecycle (file 04); in design answers, naming owners scores points.

---

## Part G — Prompt & context deep techniques

### G1. Separating instructions, data, and untrusted content

Architects should structurally separate:

- **Trusted instructions** (system / Skills / developer) 
- **Retrieved documents** (data; may be malicious—injection risk) 
- **User utterances** (intent; may be adversarial) 
- **Tool outputs** (semi-trusted depending on tool)

Use clear delimiters, cite sources, and instruct the model to treat retrieved content as **untrusted data**. Pair with technical controls in file 02/03 (allowlists, output encoding, privilege separation).

### G2. Structured reasoning without leaking chain-of-thought

Prefer platform **thinking** features for hidden reasoning when available. If you must show rationales to users, provide a **user-facing explanation** field that is curated—not raw chain-of-thought—especially in regulated UX.

### G3. Determinism levers (practical, not absolute)

LLM calls are stochastic. Reduce variance with:

- (Older models only) lower temperature — current Claude tiers (Opus 4.7+/Opus 5/Sonnet 5/Fable 5) reject sampling parameters, so lean on the levers below instead 
- Strict schemas 
- Few-shot anchors 
- Workflow gates 
- Ensemble/self-consistency only when cost allows 

Do not promise bit-identical outputs in customer SLAs; promise **process SLAs** and quality metrics.

### G4. Multimodal design notes

When inputs include images/PDFs:

- Decide whether vision model call vs OCR+text pipeline is better for tables. 
- Bound image resolution/count for cost. 
- Treat OCR text as untrusted retrieved content. 
- Eval with real document scans, not only clean digital PDFs.

### G5. Internationalization & tone

Enterprise designs often need multilingual support. Public Claude models are multilingual; still:

- Put language policy in the output contract (“respond in user’s language”). 
- Eval per language—quality is not uniform by fiat. 
- Beware mixed-language retrieval corpora and embedding/search gaps (file 02).

---

## Part H — Additional worked scenarios

### Scenario H1 — Internal policy Q&A

**Requirements:** Employees ask HR policy questions; answers must cite policy; wrong legal advice is high risk. 
**Design:** Progressive RAG over versioned policy corpus + citation-required output + refuse when retrieval empty + HITL for “exceptions/approvals” intents routed separately. Mid-tier model; cache system+tool defs. 
**Not:** Parametric-only answers from model memory; not fully autonomous agent with email-send tool.

### Scenario H2 — Coding assistant for a monorepo

**Requirements:** Help developers navigate a large repo, propose patches, run tests. 
**Design:** Agent pattern with repository tools, test runner, and strict write permissions to branches—not direct prod deploy. Strong coding model. Context via progressive file fetch, not whole monorepo dump. Server-managed settings / approved MCP tools for enterprise Claude Code (details in file 04). 
**Eval:** SWE-style task suites + security regression (no secret exfiltration).

### Scenario H3 — Customer support copilot

**Requirements:** Suggest replies; agents send. p95 < 2s for draft. 
**Design:** Fast model for draft; retrieval of account + macros; optional escalate to mid model for angry/VIP. Workflow not open agent. Cache macros/policies. HITL is the human agent (always-on). 
**Metrics:** Edit distance, CSAT, handle time, hallucination rate on factual fields.

### Scenario H4 — Overnight contract clause extraction

**Requirements:** 50k PDFs nightly; cost-sensitive; morning dashboard. 
**Design:** Batch API + map-reduce workflow + schema validation + exception queue. Not interactive Opus-class chat UX.

---

## Part I — Extended practice Q&A (26–35)

**Q26.** What is the difference between a process SLA and promising identical LLM strings? 
**A.** Process SLAs cover latency, availability, and quality rates; identical strings are unrealistic for generative models.

**Q27.** Why run OCR before or instead of pure vision for dense tables? 
**A.** Tables often extract more reliably via OCR+structure pipelines; validate with evals either way.

**Q28.** What belongs in a model upgrade canary plan? 
**A.** Shadow/replay metrics, percentage rollout, kill switch, prompt pin alignment, rollback owner.

**Q29.** How do you prevent “context landfill”? 
**A.** Retrieval filters, top-k caps, relevance thresholds, progressive discovery, and periodic corpus hygiene.

**Q30.** When is Batch API the wrong choice? 
**A.** When the end user needs interactive, low-latency responses.

**Q31.** Name two variance-reduction techniques for extraction. 
**A.** JSON schema validation + few-shot anchors (also deterministic post-processors). Temperature tuning is not a lever on current Claude tiers — sampling parameters are rejected there.

**Q32.** Why separate research-agent credentials from execute-agent credentials? 
**A.** Least privilege and blast-radius control if the research path is prompt-injected.

**Q33.** What should you log for an architectural incident review (at minimum)? 
**A.** Model ID, prompt/Skill versions, tool calls (redacted), retrieval IDs, latency, and decision outcome.

**Q34.** A design uses three agents that all share admin SaaS tokens. Primary issue? 
**A.** Over-privilege / missing security boundaries—multi-agent without isolation.

**Q35.** How do Skills interact with caching strategy? 
**A.** Stable Skill content can live in cacheable prefixes; version carefully so cache and behavior stay coherent.

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

Use these as timed drills. Write a 6–10 sentence decision, then check the scoring keys.

### Scenario J1 — Claims triage under a 3-day SLA

**Context:** An insurer wants Claude to cut claims cycle time from 11 days to 3. Volume is 8k claims/day. Regulators require an auditable decision trail. Payouts above $5k need a human signature. Corpus: policy PDFs + structured claim forms.

**Decide:** pattern per stage, model tiering, HITL placement, success metrics.

**Scoring key (preferred shape):**
1. **Workflow** with stages: ingest/OCR → extract fields → policy retrieve+cite → recommend → (HITL if payout ≥ $5k or low confidence). 
2. **Router:** Haiku-class for intent/confidence; Sonnet-class for extract+recommend; escalate hard disputes to higher-capability only when evals justify. 
3. Metrics: cycle time, auto-resolution rate below threshold, citation coverage, overturn rate by humans, p95 latency of auto path, $ per claim. 
4. Anti-patterns: single autonomous agent with write tools to payment systems; monolithic “paste whole policy book” context.

### Scenario J2 — Internal coding assistant for a 40-repo org

**Context:** Platform team wants Claude across IDEs/CLI. Secrets must not leave vault. Some repos are regulated. Latency for autocomplete-ish help should feel interactive; deep refactors can be async.

**Decide:** progressive vs monolithic context; Skills vs giant CLAUDE.md; model routing.

**Scoring key:** Progressive discovery via repo `CLAUDE.md` + retrieval over docs + MCP only for approved tools. Separate **interactive** mid-tier path from **deep agent** high-capability path with budgets. Regulated repos: stricter allowlists + no server web tools. Org Skills for ADR/test templates; avoid one 50k-token global prompt.

### Scenario J3 — Multilingual support copilot with CRM tools

**Context:** 12 languages, CRM write actions (case notes, refunds ≤ $50 auto). Brand tone critical. Hallucinated refunds are catastrophic.

**Decide:** pattern, output contract, tool authZ, eval focus.

**Scoring key:** Augmented LLM + tools inside a **workflow gate** for refunds (schema validation + policy engine + amount cap). Do not let the model “decide” refund eligibility alone—code enforces caps. Eval: language quality sample + tool-call correctness + refusal on out-of-policy. Citations not primary; **action correctness** is.

### Scenario J4 — Overnight batch contract clause extraction

**Context:** 200k PDFs/night; structured JSON to warehouse; 98% field accuracy target; cost sensitive.

**Decide:** Batch vs realtime API; map-reduce; model tier; validation.

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

**Rule of thumb:** Prefer the **leftmost** pattern that meets the requirement. Move right only when evals show the left fails.

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
| Context starvation | “I don’t know” wrongly | Over-aggressive filtering | Relax filters; query rewrite |
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

1. **Fixed KYC checklist with regulators watching** → Workflow + validators; not free-form agent. 
2. **Unknown-root-cause production incident research** → Agent + read-only tools + budget + HITL for remediations. 
3. **One front door for HR/IT/Facilities FAQs** → Router + per-domain RAG corpora + separate credentials. 
4. **Generate 10 independent product descriptions** → Parallel map; aggregate; schema validate. 
5. **High-stakes legal memo** → High-capability + RAG citations + HITL attorney; never auto-send.

---

## Part O — Extended Q&A (36–40)

**Q36.** A stakeholder wants “one mega agent” to replace a 7-stage underwriting checklist that auditors inspect. Best response? 
**A.** Propose a **workflow** mirroring the checklist with logged stage outputs; reserve agents for open research sub-tasks only. Auditability beats flexibility here.

**Q37.** Interactive UX needs p95 < 2s, but hard cases need deep reasoning. Best design? 
**A.** **Router**: fast path mid/fast model; async or “continue analyzing” path for deep thinking; communicate UX expectation.

**Q38.** Cache hit rate is 5% despite a huge system prompt. Likest cause? 
**A.** Volatile content or tool defs inserted **before** the stable prefix, or thinking/config changing breakpoints every call.

**Q39.** Select TWO for map-reduce over 500 contracts: (a) parallel per-doc extract (b) one call with all PDFs (c) schema validate each (d) single agent with write-to-ERP unbound. 
**A.** **(a)** and **(c)**.

**Q40.** Skills vs stuffing expertise into every user message—when Skills win? 
**A.** When expertise is **stable, reusable, versioned** across many requests; keep request-specific facts in user/tool channels.

---

## Part P — Rapid review card (Domain 1+2 ≈ 30%)

- Smallest model that passes evals; escalate with evidence. 
- Workflow when steps known; agent when not; multi-agent only with clear roles. 
- System prompt = policy + interface contract. 
- Progressive discovery for large tools/corpora; cache stable prefixes. 
- Value pillars must name levers (efficiency, transformation, productivity, cost, performance SLAs). 
- Document ADRs; reject alternatives explicitly. 
- Validate machine outputs in code—never hope. 
- Compaction/thinking/caching interactions are design concerns, not afterthoughts.

*Cross-read Integration/Evals in `02`, Safety in `03`. (Parts Q–R continue below.)*


---

## Part Q — Model routing & cost envelopes (worked numbers)

Architects need back-of-envelope math, not memorized price sheets (prices change—verify docs).

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
| Hard reason / dispute | 15% | High | Thinking on; async OK |

If hard route is 60% of cost but only 15% of volume, eval whether mid+critique is cheaper for a slice.

### When Batch API wins

High volume, latency-insensitive, structured jobs (extraction, classification, overnight reports). Not for chat UX. Pair with schema validation and quarantine queues.

### Pinning & upgrades

Pin model IDs in prod; upgrade via shadow → eval gate → canary → full. Treat prompt and model as a **coupled** release artifact.

---

## Part R — Additional failure modes & exam stems

| Stem cue | Prefer |
| --- | --- |
| “Auditors need stage evidence” | Workflow + logs |
| “Steps unknown; explore codebase” | Agent + budget + allowlist |
| “12 intents one bot” | Router |
| “Huge PDF corpus” | RAG progressive, not monolith |
| “p95 interactive” | Fast/mid + defer deep thinking |
| “$ sensitive overnight” | Batch + mid + validate |

**Q41.** Cache reads are cheap, so put the user question first in every request for readability—good idea? 
**A.** **No**—put **stable** content first for caching; volatile user content last.

**Q42.** Select TWO decomposition axes: by risk (read vs write), by fashion preference, by SLA path, by meme velocity. 
**A.** By risk and by SLA path.

**Q43.** Output must feed a payment API—what’s mandatory? 
**A.** **Schema validation in code** before side effects; enums for status; idempotency keys.

*End of file 01. Next: `02-enterprise-integration-production.md` (Integration 19% + Evaluation 16%).*
