---
title: Agent Frameworks, Hosting Modes & SDLC Foundations
pack: CCDV-F Developer Foundations
disclaimer: Original study notes — independent and not official course content
approx_length: ~2500–4000 words (P0 gap closer)
updated: 2026-08-23
---

# Chapter 06 — Agent Frameworks, Hosting Modes & SDLC Foundations

> **Disclaimer:** Original study-oriented study notes that close published CCDV-F blueprint gaps: named agent-framework vocabulary, self-hosted vs Anthropic-hosted managed agents, Software Engineering / systems life-cycle foundations, and a brief websockets note. Framework APIs and product names drift — **verify against current official docs** before exam day.

**Maps primarily to:** Agents and Workflows **14.7%** (patterns/frameworks ~4.9%; construction/hosting ~5.3%) · Applications and Integration **33.1%** (SE foundations ~7.4%; systems life cycle ~2.8%; technical fundamentals / SDK-over-REST / streaming transports portion of ~6.1%). 
**Secondary:** Claude Code (3.1%) as an agent host · Eval/Security when hosting and review gates appear in stems. 
**Companion chapters:** [02](./02-production-prompting-agents-tools.md) (agent loops, tools) · [03](./03-claude-code-mcp-integration.md) (API/streaming/MCP) · [04](./04-production-engineering-evals-security.md) (CI gates, review, deploy).

---

## 1. Overview — why this chapter exists

Chapters 02–04 already teach **patterns**: workflow vs agent, budgets, verifiers, Messages API, Claude Code, MCP, evals, security. Public exam-guide wording also expects **named recognition** and classic **SE literacy** mapped onto Claude delivery:

1. Recognize **Strands**, **LangGraph**, and **PydanticAI** at vocabulary level (what each is roughly for — not tutorials).
2. Choose **self-hosted Agent SDK / custom loop** vs **Anthropic-hosted managed agents** with a crisp decision table.
3. Answer Integration stems that assume REST/JSON/async, VCS, code review, refactoring, and systems life-cycle phases — with Claude Code / API as fit-points, not as a SE textbook.
4. Know where **websockets** sit relative to the usual Claude streaming path (**SSE**).

Exam shape: pick the right **control model** and **ownership boundary**, not implement a framework from memory.

---

## 2. Key map (exam recognition)

| Name / mode | Rough job | Exam cue |
| --- | --- | --- |
| **Custom loop** | You write `while not done: model → tools → results` | Max control; you own every failure mode |
| **Claude Agent SDK** | Anthropic harness (same spirit as Claude Code) in **your** process | Claude-first; MCP/subagents/permissions themes |
| **Anthropic Managed Agents** | Hosted REST-shaped agent runtime; Anthropic runs harness/sandbox/session | Fast ops; residency & runtime-fee tradeoffs |
| **LangGraph** | Explicit **graph** orchestration (nodes/edges, state, checkpoints) | Auditability, HITL interrupts, deterministic workflow skin |
| **Strands (AWS)** | **Model-driven** agent loop; strong AWS/Bedrock affinity | Portability across models; OTel; AWS-native deploy |
| **PydanticAI** | **Typed** agents / structured validated I/O (Pydantic-first) | Schema guarantees; lighter orchestration |
| **SSE streaming** | Default Claude Messages stream (server → client over HTTP) | Chat UX, TTFT, tool-arg deltas |
| **WebSockets** | Bidirectional app transport (your UI ↔ your backend) | Interrupt/HITL/voice — **not** the Claude API’s primary stream wire |

---

## 3. Named frameworks vocabulary (exam recognition, not tutorials)

> **Verify-against-current-docs:** Stars, version numbers, cloud SKUs, and exact class names change. Memorize **roles and tradeoffs**, then skim live READMEs/docs before sitting.

### 3.1 How they relate to workflow-vs-agent / typed tools / graph orchestration

CCDV-F already drills **workflow vs agent** (Chapter 02): known steps → deterministic workflow/DAG; open-ended goals with tools → agent loop with budgets. Named frameworks are **implementations of those shapes**:

| Concern | Prefer recognizing… |
| --- | --- |
| Fixed or auditable multi-step process with cycles/HITL | **LangGraph** (graph orchestration) |
| “Give model + tools + prompt; let the loop run” with AWS bias | **Strands** (model-driven agent) |
| Guaranteed structured outputs / typed tool args in Python services | **PydanticAI** (typed tools & validation) |
| Claude-native coding/agent product loop in your infra | **Claude Agent SDK** |
| Minimal surface; full ownership; teaching exam loop | **Custom Messages API loop** |

They are **not mutually exclusive** in production: teams often put Pydantic validation *inside* a LangGraph node, or call Claude via Bedrock from Strands, or wrap Agent SDK sessions behind an app WebSocket. Exam stems usually ask which **primary** control model fits the constraint.

### 3.2 Strands (AWS Strands Agents) — high level

- **What it is roughly for:** A model-driven agent framework (Apache-licensed open source lineage) optimized for production loops with first-class **AWS / Bedrock** integration themes, built-in observability (commonly OpenTelemetry), and multi-agent primitives (swarm/graph/handoff-style patterns in public materials).
- **Relation to exam concepts:** Closer to an **agent loop** than to a hand-drawn workflow DAG. You declare model, tools, prompt; the framework owns much of the turn machinery.
- **Tradeoffs vs Claude Agent SDK / custom loops:**
 - **Win:** Model portability (not Claude-only); AWS deploy story; less DIY tracing glue.
 - **Cost:** Another framework surface to learn; not the same harness as Claude Code; Bedrock/IAM setup before first run is common.
 - **Exam line:** “AWS-native, swap models without rewriting the loop” → Strands-shaped answer — not “must use Claude Agent SDK.”

### 3.3 LangGraph — high level

- **What it is roughly for:** Graph-based orchestration from the LangChain ecosystem: you define **nodes**, **edges**, shared **state**, and often **checkpoints**. Strong fit when the **process** must be inspectable (compliance, long workflows, explicit human interrupts).
- **Relation to exam concepts:** Embodies **workflow / graph orchestration**. The model acts *inside* nodes; control flow is yours. Ideal when “audit every transition” beats “max autonomy.”
- **Tradeoffs vs Claude Agent SDK / custom loops:**
 - **Win:** Explicit control, HITL interrupts, mature ecosystem integrations, durable state patterns.
 - **Cost:** More orchestration code/concepts; model-agnostic (you wire Claude yourself); heavier than a thin Agent SDK query loop for simple Claude-only agents.
 - **Exam line:** “Stateful multi-step with mandatory human approval gates and replayable state” → LangGraph-shaped — not unbounded agent flail.

### 3.4 PydanticAI — high level

- **What it is roughly for:** A Python-first, **type-safe** agent/tool layer built around Pydantic models — emphasize **validated structured outputs** and typed tool parameters over heavy multi-agent choreography.
- **Relation to exam concepts:** Amplifies **typed tools** and **output contracts** (Chapters 02/03). Often a *layer* inside larger systems rather than the whole orchestration story.
- **Tradeoffs vs Claude Agent SDK / custom loops:**
 - **Win:** Schema/validation culture; catches bad tool args before side effects; low lock-in for “single agent + tools” services.
 - **Cost:** Not primarily a graph orchestrator; multi-agent/graph needs may push you to LangGraph/Strands/Agent SDK on top.
 - **Exam line:** “Python service must never write unvalidated JSON to the ledger” → PydanticAI / schema-first — still pair with permissions for writes.

### 3.5 Tradeoff card — frameworks vs Claude Agent SDK vs custom loops

| Option | Best when | Weak when |
| --- | --- | --- |
| **Custom Messages loop** | Teaching clarity; tiny agents; unique control needs | You reinvent session, compaction, permissions, tracing |
| **Claude Agent SDK** | Claude-only; want Code-like harness (tools, MCP, subagents) in **your** process | Need non-Claude models; cannot run execution in your infra? → consider managed |
| **Managed Agents (Anthropic-hosted)** | Want Anthropic to run harness/sandbox/session; ship fast | Strict data residency / VPC-only tool execution; need exotic loop control |
| **LangGraph** | Process = product; checkpoints; HITL graph | Simple single-tool Q&A (overkill) |
| **Strands** | AWS/Bedrock-centric; model-portable agent loops | Pure Anthropic stack with Code-parity harness preferred |
| **PydanticAI** | Typed I/O guarantees in Python | Complex multi-party orchestration as the main problem |

**Anti-trap:** Naming a trendy framework is never enough. Stems still expect **budgets, least privilege, evals, and stop conditions** (Chapter 02/04).

---

## 4. Self-hosted vs Anthropic-hosted managed agents — decision table

Public 2026 product split (verify live): **Claude Agent SDK** = library in **your** process; **Claude Managed Agents** = hosted agent runtime (REST/event API themes) where **Anthropic** runs harness, sandbox, and durable session machinery. Same Claude models underneath; different **ops ownership**.

| Dimension | Self-hosted (Agent SDK / custom loop / your workers) | Anthropic-hosted managed agents |
| --- | --- | --- |
| **Who runs the loop** | You | Anthropic |
| **Where tools/sandbox execute** | Your infra (VPC, laptop, k8s) | Anthropic-managed sandbox (VPC connectors may exist — verify docs) |
| **Session / crash recovery** | You build or adopt | Platform concern |
| **Data residency pressure** | Stronger control (only inference leaves, if you design it that way) | Session logs + sandbox on provider side → compliance review |
| **Cost shape** | Tokens + **your** compute/ops | Tokens + **runtime/session** fees (publicly reported; verify) |
| **Customization depth** | Highest (custom retries, exotic tools, air-gapped patterns) | High product surface, less infra DIY; exotic loops may not fit |
| **Time-to-prod** | Slower if you lack harness maturity | Faster if constraints allow hosted execution |
| **Claude Code parity** | Agent SDK aims at Code-like harness locally/in-process | Managed = service shape; still Claude-family agents |
| **Typical exam pick** | Regulated data, custom permissions, existing fleet, cost at extreme concurrency you already operate | Async long-running agents, small team, prefer not to own sandboxes |

```text
Need agent loop at all?
 NO → Messages API (+ tools) is enough
 YES → Must tools/state stay in your boundary (residency, VPC, custom sandbox)?
 YES → Self-hosted Agent SDK or custom loop (+ optional LangGraph/Strands/PydanticAI)
 NO → Is shipping speed / managed sandbox worth runtime fees?
 YES → Anthropic-hosted managed agents
 NO → Self-hosted anyway (cost or control preference)

Prototype path (common public guidance theme): local Agent SDK → managed when ops pain dominates — still re-check compliance before moving secrets/PII.
```

**Exam traps**

1. Equating “uses Claude” with “must use Managed Agents.”
2. Equating “Agent SDK” with “Anthropic hosts my sandbox.”
3. Ignoring that **Message Batches ≠ managed agents** (batches = async model calls; not your local tool loop).
4. Forgetting **hooks/permissions/evals** still apply in both modes.

---

## 5. Claude in the SDLC / Software Engineering foundations (exam-weight card)

Published blueprint weights make **Software Engineering Foundations (~7.4%)** and **Systems Life Cycle (~2.8%)** material — not “assumed background.” Study as **Claude-mapped literacy**, not a SE course.

### 5.1 REST / JSON / async (technical fundamentals)

| Concept | Exam-useful Claude mapping |
| --- | --- |
| **REST** | Messages API is HTTP resource-oriented; SDKs wrap REST. Prefer official SDKs for retries/streaming helpers; know headers (`x-api-key`, API version) conceptually. |
| **JSON** | Tool args, structured outputs, MCP payloads, logs. Validate server-side; never trust model JSON alone for writes. |
| **Async** | (1) Concurrent tool calls / `asyncio` workers in your app; (2) Message Batches for offline model work; (3) Managed/long-running agent sessions. Don’t conflate the three. |
| **SDK-over-REST** | SDK is the default production choice; raw REST when debugging wire format or non-SDK languages. |

### 5.2 VCS, code review, refactoring

| Practice | Where Claude fits | Exam judgment |
| --- | --- | --- |
| **VCS (git)** | Authoritative state outside the prompt; branch/PR as the unit of change | Agents should commit on branches; never force-push `main` as default autonomy |
| **Code review** | Claude Code / PR agents draft; humans (or policy bots) own merge gates | High-risk: secrets, authZ, migrations, irreversible scripts → human review required |
| **Refactoring** | Plan mode → tests first → small diffs → verify | Prefer test-backed refactors; unbounded “clean the repo” agents fail stems |

**One-liner:** Claude accelerates authoring; **VCS + review + CI** remain the control plane.

### 5.3 Systems life-cycle phases (classic labels → Claude delivery)

| Phase | Claude / CCDV fit |
| --- | --- |
| **Requirements / discovery** | JTBD, constraints (latency, cost, safety), success metrics before model pick |
| **Design** | Workflow vs agent; tool/MCP boundaries; hosting mode; schemas; threat model |
| **Implementation** | API app, Agent SDK/Code, prompts-as-versioned artifacts, pin bundles |
| **Verification / eval** | Golden sets, regression harness, side-effect fail tests (Chapter 04) |
| **Deployment** | Canaries, feature flags, managed settings, kill switches |
| **Operate / monitor** | Traces, cost-per-success, cache hit rate, incident runbooks |
| **Evolve / retire** | Prompt/tool versioning, model migrations, deprecate tools with stubs |

**Exam line:** Jumping from a cool demo to production autonomy **skips** verification and deploy gates — wrong even if the model is Opus-tier.

### 5.4 How Claude Code / API fit CI and review

```text
PR opened
 → CI: unit + schema/tool contract tests + eval subset (cheap)
 → Claude Code headless / Agent SDK job (pinned model) proposes or implements on branch
 → Static checks + secret scan + permission policy lint
 → Human review (required for destructive / auth / data paths)
 → Merge → staged deploy → canary eval → promote
```

| Fit-point | Prefer |
| --- | --- |
| Chat UX tokens | Streaming Messages (SSE via SDK) |
| Offline classify/summarize | Message Batches |
| Repo-local agentic edits | Claude Code / Agent SDK with pins + allowlists |
| Org policy | Managed settings > project convenience |
| Quality truth | Eval harness in CI, not vibes in Slack |

---

## 6. Brief websockets note (streaming SDKs)

**Claude Messages streaming (provider → your backend)** is publicly documented as **HTTP Server-Sent Events (SSE)** (`text/event-stream` themes): `message_start` → content block deltas → `message_stop`. Official SDKs parse this for you.

**WebSockets** appear in CCDV-shaped stems as an **application transport** choice:

| Transport | Direction | Typical use with Claude apps |
| --- | --- | --- |
| **SSE (Claude API stream)** | Server → client (one-way) | Token streaming, TTFT UX, fine-grained tool-arg previews |
| **SSE (your API → browser)** | Server → browser | Simple chat UIs mirroring the API stream |
| **WebSocket (your app)** | Bidirectional | User interrupt mid-stream, HITL approvals on same session, voice, collaborative agents |
| **Polling** | Request/response | Batch job status — not chat TTFT |

**Exam cues**

- “Lower TTFT for chat” → stream (SSE path), not websockets-for-its-own-sake.
- “Approve a destructive tool without correlating a second HTTP POST to a dead SSE” → WebSocket (or carefully designed session) between **UI and your backend**; the backend still talks to Claude via SDK/SSE/REST.
- MCP remote transports are often described as HTTP/SSE-style — don’t confuse MCP wire with browser WebSockets.

**Trap:** Claiming the Claude Messages API “is WebSockets.” At recognition level, **SSE is the default stream**; websockets are usually **your** product’s bidirectional channel.

---

## 7. Decision trees (compressed)

### 7.1 Framework / harness pick

```text
Primary constraint?
 Typed Python I/O only → PydanticAI (± thin loop)
 Auditable graph + HITL checkpoints → LangGraph
 AWS/Bedrock + model portability → Strands
 Claude Code-like harness in our process → Claude Agent SDK
 Hosted sandbox + don’t want to run harness → Managed Agents
 Exam teaching / unique control → Custom Messages loop
```

### 7.2 SDLC red flags in stems

```text
No success metric → fix requirements
No evals before autonomy ↑ → fix verification
No branch/PR → fix VCS discipline
Prompt-only “never delete” → fix permissions/hooks
Demo model ID in prod → fix pinning + config management
```

---

## 8. Exam traps

1. **Framework name-drop without stop conditions** — still wrong.
2. **LangGraph for single-shot FAQ** — overkill; prefer prompt/tools.
3. **Managed Agents to satisfy air-gapped tool execution** — usually self-hosted.
4. **Agent SDK = Anthropic hosts my VPC** — false.
5. **WebSockets required for any streaming** — false; SSE is the API default.
6. **Batches replace CI evals** — false.
7. **Refactor agent without tests** — fails SE foundations stems.
8. **Skipping code review because Claude wrote it** — fails review/safety stems.

---

## 9. Self-check Q&A (18)

**Q1.** Exam lists Strands, LangGraph, PydanticAI — what recognition job does each own? 
**A1.** Strands ≈ model-driven (often AWS-native) agent loop; LangGraph ≈ explicit graph/state orchestration; PydanticAI ≈ typed/validated structured I/O agents.

**Q2.** When is LangGraph a better mental model than a free agent loop? 
**A2.** When transitions must be auditable, checkpointed, or interrupted by humans as part of the process definition.

**Q3.** When might you pick Strands over Claude Agent SDK? 
**A3.** Need model portability and/or AWS/Bedrock-native deploy/observability more than Claude-Code-parity harness.

**Q4.** PydanticAI vs “JSON mode in the prompt only”? 
**A4.** PydanticAI/schema validation enforces types in code; prompt-only JSON fails closed less reliably.

**Q5.** Self-hosted Agent SDK vs Managed Agents — first deciding question? 
**A5.** Must tool execution and session artifacts remain under our residency/control boundary?

**Q6.** Does using Managed Agents remove the need for evals and allowlists? 
**A6.** No — hosting changes ops ownership, not safety or quality obligations.

**Q7.** Custom loop vs Agent SDK — exam one-liner? 
**A7.** Custom = maximum control/responsibility; Agent SDK = Claude-oriented harness so you don’t rebuild session/tool/permission basics.

**Q8.** REST/JSON literacy item most likely on CCDV-F? 
**A8.** Constructing Messages-style requests, validating JSON tool args, using SDKs over fragile ad-hoc HTTP, handling stream events.

**Q9.** How should an agent interact with git in production stems? 
**A9.** Branch + PR; respect protected main; no secret commits; review before merge for risky diffs.

**Q10.** Where do code review gates sit relative to Claude Code? 
**A10.** After agent edits, before merge/deploy — especially auth, data, destructive ops; Claude drafts, policy+humans gate.

**Q11.** Map “systems life cycle” to a Claude feature launch. 
**A11.** Requirements → design (agent vs workflow, hosting) → implement (pins, tools) → eval → canary deploy → monitor → version/retire.

**Q12.** Claude API streaming default wire format? 
**A12.** SSE over HTTP; SDKs abstract event parsing.

**Q13.** When are WebSockets justified in a Claude app? 
**A13.** Bidirectional needs: mid-stream cancel/interrupt, HITL on one session, voice — between clients and **your** backend.

**Q14.** Message Batches vs managed long-running agents? 
**A14.** Batches = async model inferences with constraints; managed agents = hosted agent harness/sessions with tools/sandbox themes.

**Q15.** Refactoring stem: agent rewrites half the monorepo overnight. What’s wrong? 
**A15.** Missing SDLC controls — scope, tests, small PRs, review; autonomy without verification.

**Q16.** Async in SE foundations — three distinct meanings? 
**A16.** App concurrency; Message Batches; long-running agent sessions/workers — don’t mix answers.

**Q17.** Can you combine PydanticAI typing with LangGraph orchestration? 
**A17.** Yes conceptually — typed validation inside graph nodes; exam may still ask which concern is primary.

**Q18.** Prototype on Agent SDK then move to Managed Agents — what’s the compliance catch? 
**A18.** Tool/sandbox/session data may change residency; re-check policy before promoting PII workloads.

---

## 10. Checklist

- [ ] I can one-line Strands vs LangGraph vs PydanticAI without code.
- [ ] I can choose self-hosted Agent SDK/custom vs Anthropic managed agents from residency, cost, and control cues.
- [ ] I know custom loop / Agent SDK / managed / graph framework are different ownership models.
- [ ] I map REST/JSON/async to Messages, tools, batches, and workers correctly.
- [ ] I treat git branch/PR + code review as mandatory control plane around Claude edits.
- [ ] I can walk systems life-cycle phases with Claude fit-points (design → eval → canary → operate).
- [ ] I place Claude Code/API jobs in CI without skipping human gates on risky changes.
- [ ] I distinguish SSE (API/default stream) from WebSockets (bidirectional app channel).
- [ ] I still apply budgets, allowlists, hooks, and evals regardless of framework brand.
- [ ] I will verify framework and Managed Agents details on current official docs before exam day.

---

## 11. Glossary

| Term | Study meaning |
| --- | --- |
| **Model-driven agent** | Framework owns the tool loop; you supply model/tools/prompt (Strands-shaped) |
| **Graph orchestration** | Explicit nodes/edges/state (LangGraph-shaped) |
| **Typed tools** | Schema-validated arguments/results (PydanticAI-shaped / JSON Schema tools) |
| **Self-hosted agent** | Loop and tool execution in your process/infra |
| **Managed agents** | Provider-hosted harness/sandbox/session runtime |
| **SSE** | Server-Sent Events — usual Claude stream transport |
| **WebSocket** | Full-duplex app transport for interrupt/HITL/voice patterns |
| **SDLC** | Systems/software life cycle — requirements through retire |
| **SE foundations** | REST/JSON/async, VCS, review, refactor — exam-weighted literacy |
| **Pin bundle** | Locked model/SDK/prompt/tool versions for CI and prod |

---

## 12. If the exam asks X → think Y

| If the exam asks… | Think… |
| --- | --- |
| Named framework for auditable multi-step HITL | LangGraph |
| AWS Bedrock-centric portable agent loop | Strands |
| Strict Python structured outputs / typed tools | PydanticAI |
| Same harness spirit as Claude Code in our VPC | Agent SDK (self-hosted) |
| Don’t want to operate sandboxes; OK with hosted session | Managed Agents |
| Lowest-level clarity / special control | Custom Messages loop |
| Chat typing effect / TTFT | SSE streaming via SDK |
| Approve tool on same interactive session | WebSocket (UI↔backend) + server-side gates |
| Claude changed code — merge now? | PR + review + CI evals first |
| Which life-cycle step missing? | Usually evals or deploy gates |

---

## Appendix — Chapter → official domains

| Domain | Coverage in Chapter 06 |
| --- | --- |
| Agents and Workflows (14.7%) | Named frameworks; hosting modes; harness tradeoffs |
| Applications and Integration (33.1%) | SE foundations; SDLC; REST/JSON/async; CI/review fit; streaming transports |
| Claude Code (3.1%) | Headless/CI agent host; pins; review workflow |
| Tools and MCPs (10.6%) | Typed tools overlap; MCP still orthogonal to framework brand |
| Security / Eval | Residency hosting cues; review/CI gates (pointers to Chapter 04) |

*End of Chapter 06.*
