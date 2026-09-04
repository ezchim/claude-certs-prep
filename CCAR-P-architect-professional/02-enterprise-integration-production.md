---
title: Enterprise Integration & Production
---

# 02 — Enterprise Integration & Production

**CCAR-P condensed domain 2** 
**Official domains mapped here:** Integration (**19%**) · Evaluation, Testing & Optimization (**16%**) 
**Combined exam weight covered by this file:** ~**35%** (heaviest technical slice)

---

## Disclaimer

Original study notes only—grounded in **public** Anthropic docs (Messages API, tool use, MCP, RAG/citations, prompt caching, observability patterns) and published CCAR-P blueprint summaries. Integration details and API surfaces evolve; verify against current platform documentation. Independent pack; not affiliated with Anthropic.

---

## Overview

If Solution Design picks the *shape*, Integration makes it *real* in the enterprise, and Evaluation proves it *works*. CCAR-P weights Integration highest (19%) because architects fail in production on authZ, retrieval quality, tool boundaries, latency, and observability—not on clever prose.

This file covers:

1. Tools, MCP, agents, and enterprise connectors 
2. RAG / retrieval architecture and citations 
3. AuthN/Z, least privilege, network & data paths 
4. **CSP / delivery-route governance** (Direct API vs Bedrock vs Vertex vs Azure/Foundry) 
5. Evaluation strategies, datasets, A/B and regression—including **model judges vs code judges** 
6. Optimization (tokens, latency, cost) and observability 

Developer enablement / Claude Code org rollout lives in **`05`**. Safety/governance deep dive in **`03`**. Entry-point model notes in **`01`**; this file owns the enterprise **route-selection** decision table.

---

## Key map

| Task | Official domain | Exam signal |
| --- | --- | --- |
| Choose MCP vs REST vs CLI vs A2A | Integration 19% | Fit transport to trust boundary & ops model |
| Design RAG chunk/index/retrieve/cite | Integration 19% | Match corpus + query patterns |
| Tool allowlists & least privilege | Integration 19% | Prevent capability sprawl |
| Progressive discovery vs monolith | Integration 19% | Large tool/context sets |
| Define metrics & eval sets | Evaluation 16% | Accuracy, latency, cost, safety |
| Diagnose hallucinations / prompt fails | Evaluation 16% | Root cause → fix lever |
| Observability & optimization loops | Evaluation 16% | Logs → regressions → improve |
| Pick CSP / delivery route under constraints | Integration 19% | Auth, residency, logging, procurement—not vibes |
| Model judge vs code judge + calibration | Evaluation 16% | Deterministic first; calibrate before trust |

---

## Part A — Integration primitives

### A1. The integration stack

```
App / Orchestrator
 ├─ Claude Messages API (model I/O)
 ├─ Client tools (your code executes)
 ├─ Server tools (Anthropic-hosted, where enabled)
 ├─ MCP servers (standardized tool/data connectors)
 ├─ Retrieval system (search/index/rerank)
 └─ Enterprise IdP, secrets, network, audit
```

**Contract reminder (public tool-use model):** Claude never executes your tools by itself. It emits `tool_use`; **your runtime** (or Anthropic for server tools) executes; you return `tool_result`. Architects design that loop: timeouts, retries, idempotency, and privilege.

### A2. Client tools vs server tools

| Type | Who runs code? | Examples (illustrative) | Architect implication |
| --- | --- | --- | --- |
| Client / user-defined | Your app | CRM lookup, ticketing, internal SQL | You own authZ, logging, sandboxing |
| Anthropic server tools | Anthropic | Web search/fetch, code execution, tool search (per docs) | Enable deliberately; understand data leaving boundary |
| Anthropic-schema client tools | Your app, trained-friendly schemas | Prefer known schemas when equivalent | Often more reliable than ad-hoc twins |

### A3. Model Context Protocol (MCP)

MCP is an **open standard** for connecting assistants/agents to tools and data. Public docs emphasize:

- Local process, HTTP/SSE/streamable HTTP, and in-process transports (ecosystem evolves—check current MCP + Claude Code docs). 
- Servers expose tools/resources/prompts. 
- **Trust model:** treat third-party servers as supply-chain risk; prompt injection via fetched content is real. 
- Large tool sets → **tool search** / deferred loading so definitions do not devour the context window. 
- Channels/events (where supported) push external signals into sessions.

**When to choose MCP**

| Prefer MCP when… | Prefer direct API/SDK tools when… |
| --- | --- |
| Many systems, standardized discovery | One or two bespoke tools |
| Want reusable connectors across Claude Code + apps | Ultra-custom latency path |
| Org wants a connector catalog | Protocol overhead not worth it |
| Partners already ship MCP servers | You must stay inside a locked-down RPC fabric |

### A4. Agent-to-agent and orchestration buses

Multi-agent systems need an explicit **handoff protocol**: message schema, shared memory policy, and permission boundaries. Prefer:

- Clear supervisor or workflow owning side effects 
- Per-agent credentials 
- Trace IDs across hops 

Avoid chatty free-form agent swarms without budgets.

### A5. CLI and automation surfaces

Command-line tools (including Claude Code CLI in eng workflows—see file 05) fit CI, ops, and developer loops. For customer-facing products, prefer service APIs with authZ. Exam scenarios: pick CLI for **builder productivity**, APIs for **product runtime**—unless the product *is* a developer tool.

---

## Part B — RAG & retrieval architecture

### B1. RAG reference pipeline

```
Ingest → Clean/OCR → Chunk → Embed/Index (+ keyword) → (optional graph)
Query → Rewrite/classify → Retrieve → Rerank → Pack context → Claude → Cite → Validate
```

### B2. Chunking decisions

| Corpus trait | Chunking bias | Notes |
| --- | --- | --- |
| Policies with numbered sections | Section-aware chunks + headers | Preserve clause IDs for citations |
| Code | Symbol/function-aware | Don’t split mid-function blindly |
| Tables/financials | Table-as-unit or row groups | Often need OCR structure |
| Chat logs | Turn windows with timestamps | PII scrubbing critical |
| Slides | Per-slide + title | Low text density |

Overlap helps continuity; too much overlap wastes tokens and duplicates noise.

### B3. Hybrid retrieval

Production RAG usually combines:

- **Dense/vector** for semantic paraphrase 
- **BM25/keyword** for IDs, error codes, SKUs, legal article numbers 
- **Metadata filters** (tenant, product, locale, effective date) 
- **Reranker** for precision at low k 

Knowledge graphs help multi-hop “who owns X” questions but add ops cost—justify with eval gains.

### B4. Context packing & citations

Public Messages API patterns support **search result** content blocks and citations so answers attribute sources. Architect rules:

- Return source id/title/uri with chunks 
- Require citations in the output contract when factual grounding matters 
- If retrieval returns nothing relevant, **refuse or ask**—do not freestyle policy 
- Cap top-k by token budget; prefer fewer high-quality chunks 

### B5. Index freshness & tenancy

- Version documents; prefer effective-dating for policies 
- Tenant-isolated indexes or hard filters (never “prompt the model to only use tenant A”) 
- Tombstone deletes; GDPR erasure workflows touch indexes too 
- Measure **staleness SLOs** (e.g., policy live within 15 minutes of publish)

### B6. RAG failure modes (exam gold)

1. Wrong chunk boundaries → missed clauses 
2. Embedding drift after model change without reindex plan 
3. Cross-tenant leakage via shared index 
4. Prompt injection in retrieved HTML (“ignore policies…”) 
5. Citation theater (numbers that don’t match sources) 
6. Over-retrieval drowning the model 
7. Under-retrieval → hallucinations 

### B7. Progressive discovery applied to RAG

Don’t dump the corpus. Use:

1. Metadata route (product=X) 
2. Retrieve small k 
3. If confidence low / user asks deeper, retrieve more or browse hierarchical TOC tools 
4. Agent may call `search` then `fetch_section` tools rather than one giant context 

---

## Part C — Tools, agents, and production control

### C1. Tool design checklist

- [ ] Name/description clear enough for routing 
- [ ] JSON schema strict; enums over free text 
- [ ] Side-effect tools idempotent with keys 
- [ ] Timeouts and circuit breakers 
- [ ] AuthZ checked **in tool server**, not only in prompt 
- [ ] Arguments logged with redaction 
- [ ] Rate limits per tenant 
- [ ] Dry-run / confirm mode for destructive ops 

### C2. Least privilege patterns

| Pattern | Description |
| --- | --- |
| Stage-scoped tools | Extract stage cannot `refund_payment` |
| Role-scoped agents | Support vs finance credentials differ |
| Read/write split | Research agent read-only; executor gated |
| Confirm token | Second human or second factor for writes |
| Allowlist hosts | SSRF-safe fetch tools |

### C3. Agent loop controls

Production agents need:

- **Max turns / max dollars / max wall time** 
- Stop conditions (`task_complete`, `needs_human`) 
- Sticky trace IDs 
- Memory policy (what persists across sessions) 
- Sandbox for code execution 

### C4. Latency budget worksheet

Example interactive UX p95 = 3.0s:

| Stage | Budget |
| --- | --- |
| Auth + ingress | 50ms |
| Retrieval | 200ms |
| Rerank | 50ms |
| Claude TTFT | 800ms |
| Claude completion | 1500ms |
| Validate + render | 100ms |
| Contingency | 300ms |

Miss budget? Shrink context, cache prefixes, smaller model, parallel retrieval, or async UX (“answer streaming + tools in background”).

### C5. Reliability patterns

- Retries with jitter on 429/5xx 
- Fallback model on timeout 
- Cached last-good answers for FAQ intents 
- Queue + worker for long tools 
- Poison-tool detection (repeated failures → disable tool, alert)

---

## Part D — Security-relevant integration (pointer + essentials)

Deep governance is file 03. Integration essentials:

- **Prompt injection** via docs/tools → isolate untrusted content; minimize write tools 
- **Secrets** in vaults; short-lived tokens to tools 
- **Egress controls** for fetch/MCP 
- **Data paths:** what leaves VPC to Anthropic/cloud AI endpoint; ZDR/retention options per contract 
- **Logging redaction** for prompts containing PII 


---

## Part D5 — CSP / delivery-route governance (P0)

Architects do **not** pick “Claude on X” because a slide looked modern. They eliminate routes that violate **identity, residency, logging/retention, procurement, and model-ID** constraints—then choose among survivors on ops fit and feature lag.

Public surfaces (verify current docs; offerings evolve):

| Route | What it is (public framing) |
| --- | --- |
| **Direct Anthropic API** | First-party Messages API (`platform.claude.com`); Anthropic-operated inference |
| **Amazon Bedrock** | Claude via AWS Bedrock; AWS-managed boundary / regional controls (Messages and/or legacy Invoke/Converse paths per docs) |
| **Google Vertex AI** | Claude via Google Cloud Vertex; GCP IAM + regional endpoints |
| **Azure / Microsoft Foundry** | Claude in Microsoft Foundry; Azure-hosted and/or Anthropic-hosted options with different deployment types |
| **Claude Platform on AWS** *(related)* | Anthropic-operated Claude on AWS Marketplace-style path—**not** identical to Bedrock; residency/inference pinning differs—do not conflate in exam stems |

### D5.1 Public-safe decision table

Treat cells as **themes to verify in the live contract/docs**, not eternal guarantees. ZDR and BAAs are **enterprise options**—confirm entitlement before designing as if they exist.

| Dimension | Direct Anthropic API | Amazon Bedrock | Google Vertex AI | Azure / Microsoft Foundry |
| --- | --- | --- | --- | --- |
| **Identity / auth** | API keys / org & workspace controls; app still owns user→tool identity | **AWS IAM** (roles, SCPs, PrivateLink patterns); CloudTrail-friendly | **GCP IAM** + Vertex permissions; org policies | **Entra ID / Azure RBAC** + Foundry project controls |
| **Data residency / inference pin** | `inference_geo` (e.g. global vs `us`) on supported models—check docs | **Regional endpoints** / geographic inference profiles; region choice is the pin | **Region / endpoint** selection governs where inference runs | **Global Standard** vs **US Data Zone Standard** (US pin analogous to `inference_geo: us` where documented); hosting option (Azure vs Anthropic) matters |
| **Logging / retention posture** | Anthropic retention & abuse-monitoring terms; **ZDR** may be available as enterprise option—**verify** | Governed largely by **AWS/Bedrock** data-protection docs + your account logging; do not assume Anthropic ZDR wording maps 1:1 | Governed by **Google Cloud / Vertex** data processing + your logging sinks | Anthropic processor terms + Azure-side retention; safety-flagged content paths may differ by hosting—**read current Foundry Claude docs** |
| **Procurement / BAA / FedRAMP themes** | Direct MSA / DPA with Anthropic; BAA/FedRAMP only if **your** deal and offering support them | Buy through **AWS Marketplace / Bedrock**; lean on existing AWS commit, AWS compliance artifacts, org SCPs—still map PHI/FedRAMP to the **specific** Bedrock Claude offering | Buy through **GCP**; reuse GCP commit & org policy; same “artifact ≠ automatic BAA” rule | Buy through **Azure Marketplace / Foundry**; reuse Microsoft EA; confirm HIPAA/FedRAMP against the **deployment type** you actually provision |
| **Model ID differences** | Anthropic model IDs (e.g. dated `claude-…` strings in Messages API) | Bedrock **model IDs / inference profile IDs / ARNs** (and Messages path IDs where enabled)—**not** copy-paste of first-party IDs | Vertex publisher model IDs / versions—**platform-specific** | Foundry **deployment names** + model versions (hosting option encoded)—app config must not hardcode another CSP’s ID |
| **Feature / API lag themes** | Often **earliest** access to new Messages features, betas, server tools | May lag or expose via Bedrock-shaped APIs; check parity before promising a feature | Similar parity/lag story on GCP | Parity depends on Azure-hosted vs Anthropic-hosted deployment—feature matrices differ |
| **When architects pick this** | Max feature velocity; simple SaaS; residency solvable via `inference_geo` + contract; team already on Anthropic billing | AWS-centric estate; IAM/PrivateLink mandates; residency via AWS regions; procurement must stay inside AWS | GCP-centric estate; Vertex is the approved AI plane; org policy blocks other egress | Azure-centric estate; Entra + Foundry is the approved AI plane; need US Data Zone or Azure-hosted boundary |

**Always true on every route:** your **app** still owns tenancy authZ, tool least privilege, prompt/log redaction, and eval gates. Switching CSP does not fix “authZ in the prompt.”

### D5.2 Constraint-elimination method (exam method)

```
1. List hard constraints from the stem (IdP, residency geo, logging/ZDR, BAA/FedRAMP, cloud commit, PrivateLink, feature X must ship next sprint).
2. Strike any route that cannot meet a hard constraint (verify assumptions—don't invent FedRAMP).
3. Among survivors, score: ops familiarity, model-ID/ops tooling fit, feature parity risk, cost predictability.
4. Document rejected routes in an ADR one-liner ("Rejected Direct API: no path to required VPC endpoint pattern in this org").
```

**Vibes (fail):** “Everyone uses Bedrock.” / “Direct API is always fastest to value so pick it.” / “Foundry because we like Copilot.” 
**Constraint elimination (pass):** “Stem requires EU processing inside existing GCP org policy → Vertex regional endpoint; Direct API global default is out.”

### D5.3 Model ID & config portability traps

- Abstract a **provider adapter**: `model_ref` in your config → resolved ID per environment. 
- Never share one hardcoded Bedrock ARN across “dev on Direct API.” 
- Pin **versioned** IDs in prod; control upgrades via flags + eval gates. 
- Residency parameters differ by route (`inference_geo` vs region URL vs Data Zone deployment)—test the pin with a deliberate misconfig in staging.

### D5.4 Logging / ZDR exam discipline

- **ZDR** = contractual/API posture that limits provider retention of prompts/completions (where offered). It is **not** a substitute for your app’s retention, SIEM, or erasure of embeddings/eval caches (see file 03). 
- Abuse-monitoring / safety-flagged egress may still exist under provider terms—design assuming “ZDR ≠ invisible to all safety systems” until counsel/docs say otherwise. 
- Cloud routes: your CloudTrail/Cloud Logging/Diagnostic logs may retain **metadata and sometimes payloads** you configured—architect the sink, not only the model vendor story.

### D5.5 Exam traps — delivery routes

1. Picking a CSP for brand preference while ignoring residency in the stem. 
2. Equating **Claude Platform on AWS** with **Bedrock** (different operators/boundaries). 
3. Assuming BAA/FedRAMP/ZDR on every route without verification. 
4. Copy-pasting Anthropic model IDs into Bedrock/Vertex/Foundry calls. 
5. Treating cloud IAM as optional because “the model is smart.” 
6. Promising day-one feature parity across all CSPs. 
7. Using Direct API global inference when the stem demands pinned geography. 
8. Believing CSP choice alone satisfies GDPR/HIPAA—data maps and app controls still required.

### D5.6 Mini ADR template (copy/adapt)

```
Decision: Deliver Claude via <route>
Constraints: <residency, IdP, procurement, feature>
Rejected: <routes + one-line why>
Auth: <IAM/API key pattern>
Residency pin: <param/region/deployment type>
Logging: <provider posture + our sinks + ZDR status: verified/not entitled>
Model IDs: <adapter mapping>
Rollback: <alternate route or prior model pin>
```

### D5.7 Q&A — CSP / delivery routes (73–80)

**Q73.** Stem: “Must keep inference in US; already standardized on Azure AD + Foundry.” Best default direction? 
**A.** Azure/Foundry with a **US Data Zone** (or equivalent US-pinned) deployment type per current docs—not Direct API global by default.

**Q74.** Select TWO reasons to prefer Bedrock over Direct API in an AWS-only regulated estate. 
**A.** Reuse **IAM/SCPs/PrivateLink** patterns and keep procurement/compliance evidence inside the **AWS** boundary the org already operates.

**Q75.** Why is “we’ll use Bedrock model ID `claude-sonnet-…` from the Anthropic docs” a trap? 
**A.** Bedrock uses **platform-specific** model/inference-profile identifiers; first-party IDs are not portable.

**Q76.** ZDR is enabled on the provider—can you skip redacting PII from application logs? 
**A.** **No.** ZDR addresses provider retention posture; your logs, warehouses, and eval stores still need minimization/retention design.

**Q77.** Product needs a brand-new Messages API beta next week; estate is multi-cloud with no hard residency pin. Which route often wins and why? 
**A.** **Direct Anthropic API**—typically earliest feature access; confirm beta entitlement and still document residency/logging.

**Q78.** FedRAMP-like stem lists “authorized cloud AI service only.” What do you do first? 
**A.** **Constraint-eliminate** routes using the customer’s authorized service list / boundary docs—then design inside the survivor (logging, change control), not “add a safer prompt.”

**Q79.** Vertex vs Direct API when GCP org policy blocks egress to non-Google AI endpoints? 
**A.** **Vertex**—Direct API may be unreachable/noncompliant under that org policy regardless of model quality.

**Q80.** Pick-for-vibes vs constraint-elimination—give one exam tell for each. 
**A.** Vibes: answer cites popularity/speed alone. Constraint-elimination: answer cites a stem constraint that **disqualifies** other routes before optimizing.


---

## Part E — Evaluation, testing & optimization

### E1. What to measure

| Category | Example metrics |
| --- | --- |
| Task success | Exact match, F1, pass@k, rubric score |
| Grounding | Citation precision/recall, unsupported claim rate |
| Safety | Policy violation rate, injection success rate |
| Latency | TTFT, total, p95/p99 per route |
| Cost | $ per successful task, tokens per task |
| UX | Thumbs, edit distance (copilot), escalation rate |
| Ops | Error rate, tool failure %, cache hit rate |

**Architect rule:** Metrics must match the **business job**. Support copilots ≠ code agents ≠ RAG legal Q&A.

### E2. Dataset design

1. **Gold set** — labeled, versioned, representative 
2. **Adversarial set** — injections, jailbreaks, tricky entities 
3. **Regression set** — past production failures 
4. **Slice sets** — languages, tenants, document types 

Keep train/prompt-tuning examples **out** of the held-out eval set.

### E3. Automated vs human eval

| Method | Strength | Weakness |
| --- | --- | --- |
| Exact/schema checks | Cheap, precise for extract | Misses nuance |
| Rubric LLM-as-judge | Scales | Judge bias; calibrate |
| Human review | Gold standard for nuance | Cost/latency |
| Pairwise A/B | Good for preference | Needs traffic/sample size |
| Security probes | Catches injections | Must refresh attacks |

Hybrid is normal: automations gate every PR; humans sample weekly + review high-severity slices.

**P0 expansion:** see **Part M7 — Model judges vs code judges + calibration** for when deterministic graders beat LLM-as-judge, how to calibrate against human gold, and how to freeze thresholds **before** peeking at experiment results.

### E4. Experimentation discipline

- Change **one** major variable at a time when diagnosing 
- Record model ID, prompt hash, index version, tool versions 
- Use statistical sense on A/B (don’t crown a winner on 20 chats) 
- Shadow mode before cutover 

### E5. Diagnosing common failures

| Symptom | Likely causes | Levers |
| --- | --- | --- |
| Hallucinated facts | Weak retrieval / no refuse path | Raise recall, require citations, refuse on empty |
| Format breaks | Loose prompt | Schema + validator + few-shot |
| High latency | Fat context / big model | Cache, route, shrink k, faster model |
| Cost spike | Loops / fat tools defs | Max turns, tool search, caching |
| Inconsistent quality | Underspecified policy/contract | Clarify contract, schema + few-shot anchors (sampling knobs are rejected on current tiers) |
| Tool wrong choice | Poor descriptions / too many tools | Redescribe, allowlist, router |
| Good offline, bad prod | Distribution shift | Add prod slices to eval |

### E6. Optimization playbook (order matters)

1. **Correctness first** (eval gates) 
2. **Remove waste context** 
3. **Cache stable prefixes** 
4. **Route easy traffic to cheaper/faster models** 
5. **Parallelize independent work** 
6. **Only then** micro-tune prompt phrasing and few-shot anchors 

Premature model downgrades that break safety/quality fail exam scenarios that list compliance constraints.

### E7. Observability architecture

Minimum viable telemetry:

- Request/trace ID 
- Route / intent 
- Model + prompt/Skill versions 
- Retrieval query + doc IDs returned 
- Tool name, latency, status 
- Token usage breakdown (input/cache/output/thinking) 
- Outcome labels (auto, human-edited, failed) 
- Safety flags 

Store raw prompts carefully (PII, retention). Prefer pointers + redacted snippets for long-term storage.

Dashboards: quality weekly, latency/cost hourly, safety realtime alerts.

### E8. Continuous evaluation in CI/CD

```
PR → unit tests for tools → offline eval suite gate → staging shadow → canary → prod
```

Fail the build on statistically meaningful regressions on critical slices (e.g., citation precision, PII leak probes).

---

## Part F — Reference architectures (integration-centric)

### F1. Enterprise RAG service

Ingress → tenancy authZ → query rewrite → hybrid retrieve → rerank → Claude (cached policy) → citation validate → response. Eval harness replays queries nightly. Index updater consumes CMS webhooks.

### F2. Tool-using support agent

Workflow front door routes: FAQ RAG vs account-tool path vs human. Account tools use scoped OAuth. Writes require confirm. Traces to warehouse for weekly rubric grading.

### F3. MCP-connected builder environment

Claude Code + approved MCP marketplace (file 05) for eng productivity; **separate** from customer runtime credentials. Never reuse prod write tokens in IDE agents.

---

## Decision trees

### Integration transport picker

```
Need standardized multi-tool ecosystem across products/IDEs?
 YES → MCP (+ allowlisted servers)
 NO → Is it 1–2 tools in one service?
 YES → Native client tools via Messages API
 NO → Mix: MCP for commodity SaaS, native for core domain
```

### RAG urgency picker

```
Answers must be attributable to private docs?
 YES → RAG (+ citations) required
 Facts mostly stable public knowledge + low risk?
 YES → Maybe parametric + browse tool
 Actions on systems of record?
 YES → Tools/API, not RAG alone
```

### Eval readiness gate

```
Have gold + adversarial + regression sets with owners?
 NO → Do not ship beyond private beta
 YES → Automate on PR + weekly human sample → ship with canary
```

---

## Exam traps

1. Dumping entire KB into context instead of RAG. 
2. AuthZ “in the prompt only.” 
3. Agent with unbounded tools and no max-turn budget. 
4. Evaluating only vibe checks / cherry-picked demos. 
5. Optimizing cost before meeting accuracy/safety bars. 
6. Ignoring tenant isolation in shared vector DB. 
7. Treating MCP servers as trusted by default. 
8. No citations when the business requires grounded answers. 
9. Parallelizing dependent RAG→generate→act stages incorrectly. 
10. Missing cache hit metrics when cost is a scenario constraint. 
11. Using production write credentials inside developer MCP. 
12. Declaring victory on LLM-as-judge without calibration.
13. Picking Direct API / Bedrock / Vertex / Foundry for brand vibes instead of constraint elimination.
14. Assuming ZDR or BAA exists on a route without verifying entitlement.
15. Freezing eval pass thresholds only after peeking at A/B results.

---

## Practice Q&A (28)

**Q1.** Who executes client tool calls? 
**A.** Your application runtime—not the model process itself.

**Q2.** Why might MCP tool search matter? 
**A.** Large tool catalogs consume context; search loads only needed tools.

**Q3.** Name two retrieval methods used in hybrid search. 
**A.** Dense/vector and BM25/keyword (plus metadata filters/rerank).

**Q4.** What should happen if RAG returns no relevant chunks for a policy question? 
**A.** Refuse, ask clarifying questions, or escalate—do not invent policy.

**Q5.** Where must authorization be enforced for a `create_refund` tool? 
**A.** In the tool server / backend policy layer (least privilege), not only in prose instructions.

**Q6.** Give three agent loop controls. 
**A.** Max turns, max cost, max wall time (also stop conditions, allowlists).

**Q7.** What is citation precision measuring roughly? 
**A.** Whether cited sources actually support the claims made.

**Q8.** Why keep an adversarial eval set? 
**A.** To catch injections, jailbreaks, and tricky entities before attackers/customers do.

**Q9.** Offline eval good, prod bad—first suspicion? 
**A.** Distribution shift; add production slices to the eval suite.

**Q10.** How does prompt caching help integration architectures? 
**A.** Reduces cost/latency for stable system prompts, tool defs, and policy prefixes.

**Q11.** What is a poison-tool pattern response? 
**A.** Detect repeated failures, disable/circuit-break the tool, alert operators.

**Q12.** Why isolate tenants in the index? 
**A.** Prevent cross-tenant retrieval leakage; prompt instructions alone are insufficient.

**Q13.** When are server tools a design concern? 
**A.** When data egress / trust boundary to Anthropic-hosted execution matters to compliance.

**Q14.** Order optimization correctly at high level. 
**A.** Correctness/safety → shrink waste → cache → route cheaper → parallelize.

**Q15.** What telemetry fields are essential for debugging a bad tool choice? 
**A.** Trace ID, tool descriptions version, model ID, conversation snippet, tool_use args/results (redacted).

**Q16.** Map-reduce RAG reduce stage blows tokens—what do you do? 
**A.** Cap mapper outputs, hierarchical reduce, or summarize intermediate notes with schema.

**Q17.** Why version the index alongside prompts? 
**A.** So evals and incidents can reproduce behavior; retrieval changes alter answers.

**Q18.** What is SSRF risk in fetch tools? 
**A.** Model/tool induced requests to internal IPs/metadata endpoints—mitigate with allowlists.

**Q19.** When is CLI integration appropriate? 
**A.** Developer/ops automation; usually not end-customer product path unless the product is a CLI.

**Q20.** Define successful A/B for a prompt change. 
**A.** Pre-registered metrics, adequate sample, no other simultaneous changes, rollback plan.

**Q21.** How do you stop citation theater? 
**A.** Validator checks cite IDs exist in provided chunks; rubrics penalize unsupported claims.

**Q22.** What is progressive discovery in a tool-rich agent? 
**A.** Start with few tools/context; fetch more via search/browse as needed.

**Q23.** Why separate builder MCP credentials from runtime product credentials? 
**A.** Blast radius—IDE agents should not hold prod write power.

**Q24.** Name a grounding metric besides citation precision. 
**A.** Unsupported claim rate / faithfulness score / retrieval recall@k.

**Q25.** What does a schema validator prevent in tool args? 
**A.** Malformed calls, unexpected enums, missing required fields—before side effects.

**Q26.** Latency p95 fails only on tool-heavy intents—where to look? 
**A.** Tool fanout, serial tool chains, cold MCP connections, over-retrieval before tools.

**Q27.** What belongs in a nightly eval job? 
**A.** Gold+adversarial suites, regression cases, cost/latency snapshots, alert on deltas.

**Q28.** Can you rely on the model to “only call safe tools”? 
**A.** No—enforce allowlists and server-side authZ; prompts are not controls.

---

## Checklist

- [ ] I can diagram client tool loops vs server tools 
- [ ] I can argue MCP vs native tools with trade-offs 
- [ ] I can design hybrid RAG with tenancy filters and citations 
- [ ] I can set agent budgets and least-privilege tool scopes 
- [ ] I can build gold/adversarial/regression eval sets 
- [ ] I can diagnose hallucination vs format vs latency failures 
- [ ] I can list essential observability fields 
- [ ] I can place eval gates in CI/CD 
- [ ] I can optimize cost without violating safety bars 
- [ ] I can explain progressive discovery for tools and RAG 
- [ ] I can run CSP route selection by constraint elimination (auth, residency, logging/ZDR, procurement, model IDs) 
- [ ] I can choose code vs model judges, calibrate to human gold, and pre-register thresholds/rollback 

---

## Glossary

| Term | Meaning |
| --- | --- |
| MCP | Model Context Protocol—standard for tool/data connectors |
| Client tool | Tool executed by your runtime after `tool_use` |
| Server tool | Provider-hosted tool execution path |
| Hybrid search | Vector + keyword (+ filters/rerank) |
| Reranker | Model/stage that reorders retrieved candidates |
| Faithfulness | Answer supported by provided evidence |
| TTFT | Time to first token |
| Shadow mode | Candidate system scores in parallel without user impact |
| Circuit breaker | Stop calling failing dependencies temporarily |
| Tenant isolation | Hard separation of customer data in retrieve/act paths |
| Tool search | Deferred discovery of tools to save context |
| Idempotency key | Ensures safe retries for side-effecting tools |
| Gold set | Labeled evaluation dataset |
| LLM-as-judge | Using a model to score outputs against a rubric |
| SSRF | Server-side request forgery via fetch-like tools |
| CSP (delivery) | Cloud/service path that serves Claude (Direct API, Bedrock, Vertex, Foundry, …) |
| ZDR | Zero Data Retention—provider retention posture to verify contractually |
| Inference pin | Control that restricts where inference may run (geo param, region, data zone) |
| Code judge | Deterministic assertions/graders (schema, cite-ID, policy engine) |
| Judge calibration | Measuring LLM-as-judge agreement vs human gold before trusting scores |
| Pre-registration | Fixing metrics/thresholds/rollback before seeing experiment outcomes |

---

## Part G — Deep dive: enterprise auth, networking, and data planes

### G1. Identity propagation

Enterprise integrations fail when the model path “becomes the user” with a god token. Prefer:

1. User authenticates to your app (OIDC/SAML). 
2. App obtains **downstream tokens** constrained to that user’s scopes (OAuth on-behalf-of, token exchange). 
3. Tools call APIs with those tokens. 
4. Audit logs attribute actions to the human + service principal.

Service accounts are for batch jobs with separate controls—not as a shortcut for interactive agents.

### G2. Network patterns

| Pattern | Use |
| --- | --- |
| Public Claude API egress | Simplest; control via proxy/DLP |
| Cloud AI in your tenant (Bedrock/Vertex/Foundry) | Data plane nearer to existing cloud controls |
| PrivateLink / VPC endpoints | Reduce public internet exposure |
| Reverse proxy with allowlists | Central inspect/log/block |

Document **where prompts and embeddings live** (vector DB region, log store region). Residency questions appear in architect scenarios.

### G3. Synchronous product vs async worker

Interactive user waits → tight tool timeouts, streaming text, defer slow tools to “still working” jobs. Back-office automation → queue workers, Batch API, longer tool budgets. Do not force both into one latency class.

### G4. Contract testing for tools

For every tool:

- Schema unit tests 
- AuthZ negative tests (user without role must fail) 
- Idempotency tests 
- Timeout behavior tests 
- Recording of example traces for eval replay 

Treat tools as microservices—the LLM is just another client.

### G5. Retrieval evaluation specifics

Measure beyond “answer quality”:

- **Recall@k** of relevant docs 
- **nDCG** for ranking 
- **Chunk utilization** (did Claude use what you retrieved?) 
- **Freshness lag** 
- **Filter correctness** (tenant/product)

Many “model problems” are retrieval problems. Fix index before swapping to a giant model.

### G6. Online evaluation & feedback

Capture implicit signals: edits, regenerations, thumbs, escalation. Route low-confidence or high-severity traffic to human labels. Close the loop into regression sets weekly. Without this, offline gold sets rot.

### G7. Cost anomaly detection

Alert when:

- Tokens/task jumps >N% week over week 
- Agent turn count distribution shifts 
- Cache hit rate collapses (prefix thrash) 
- One tenant dominates spend (runaway loop)

Tie alerts to runbooks: freeze agent max-turns, revert prompt, disable tool.

### G8. Multi-region active-active concerns

Pin model versions consistently; replicate indexes with conflict rules; sticky sessions for conversation state; understand cloud endpoint global vs regional routing trade-offs from public docs. Fail closed on auth, fail soft on optional enrichment tools.

### G9. Additional scenarios

**Scenario:** Manufacturing firm wants Claude to query SCADA historian. 
**Answer shape:** Never raw plant network from SaaS agent. Use DMZ read API, allowlisted metrics, human approval for actuation—prefer no write tools initially. RAG over manuals + read tools for telemetry summaries.

**Scenario:** Legal wants “chat with all contracts.” 
**Answer shape:** Tenant-isolated hybrid RAG, clause-aware chunking, mandatory citations, matter-based ACL filters, DLP on logs, eval on hallucination of obligations, HITL for redlines.

**Scenario:** Cost blowup after MCP rollout. 
**Answer shape:** Inspect tool definition token share, enable tool search, shrink allowlists, cache system+tool prefixes, add max turns, fix loops calling search repeatedly.

### G10. Extended Q&A (29–36)

**Q29.** Why is on-behalf-of token exchange better than a shared god token? 
**A.** Preserves user-scoped authZ and attributable audit trails.

**Q30.** Name a retrieval metric that can be high while user answers still fail. 
**A.** Recall@k can look fine while chunk packing/citation/prompting still breaks answers—measure end-task metrics too.

**Q31.** What is chunk utilization telling you? 
**A.** Whether retrieved evidence is actually influencing the answer or just wasting tokens.

**Q32.** How do you test authZ for tools? 
**A.** Negative tests: identities lacking roles must be denied by the tool server.

**Q33.** Why might cache hit rate collapse after a “small prompt tweak”? 
**A.** Prefix bytes changed (or tool order/thinking config), invalidating breakpoints.

**Q34.** What is fail-soft for optional enrichment? 
**A.** Return core answer if optional tool/RAG times out, with explicit partial-status—not a hard 500.

**Q35.** Where should embeddings for regulated HR data reside? 
**A.** In approved regions/systems under the same residency controls as the source HR data.

**Q36.** What is the first integration question for any write tool? 
**A.** Who is allowed, how is it audited, and what is the blast radius if prompt-injected?

---

## Extended checklist

- [ ] I can describe OBO/token exchange vs god tokens 
- [ ] I can pick VPC/private egress vs public API with rationale 
- [ ] I can list retrieval-only metrics vs end-task metrics 
- [ ] I can design contract tests for side-effecting tools 
- [ ] I can write a cost-anomaly runbook trigger 
- [ ] I can explain global vs regional endpoint trade-offs at a high level 

---

## Glossary additions

| Term | Meaning |
| --- | --- |
| OBO | On-behalf-of delegated authorization |
| nDCG | Ranking quality metric |
| Recall@k | Fraction of relevant docs retrieved in top k |
| DLP | Data loss prevention controls on egress/logs |
| DMZ | Network zone buffering trusted and untrusted systems |
| Active-active | Multi-region serving with replication |
| Prefix thrash | Frequent cache invalidation from unstable prefixes |
| Online eval | Production feedback-based measurement |
| Contract test | Tests that enforce tool API/auth behavior |
| Fail-soft | Degrade gracefully when non-critical deps fail |

---



---

## Part H — Production hardening narratives (exam-style)

### H1. Idempotency and exactly-once *effects*

LLMs retry; users double-click; queues redeliver. Side-effecting tools must be **at-least-once safe**:

- Client supplies idempotency key (ticket_id + action + hash). 
- Tool server stores key → result for TTL. 
- Replays return original result without charging twice / double-shipping.

Exam wrong answer: “Claude will remember it already refunded.”

### H2. Conversation state stores

Choose explicitly:

| Store | Pros | Cons |
| --- | --- | --- |
| Client-owned history replay | Simple, transparent | Large prompts; PII sprawl |
| Server session store | Truncation/compaction control | Consistency & retention duties |
| Summary + recent window | Token efficient | Summary drift |

Pair with compaction features from the platform when sessions are long-running (see file 01). Always define retention aligned to legal holds.

### H3. Feature flags for AI paths

Flag dimensions that save incidents:

- Model pin 
- Prompt/Skill version 
- Retrieval index version 
- Tool allowlist set 
- Agent max-turn cap 
- HITL required yes/no 

Rollback is a flag flip—not a hope.

### H4. SLOs for AI features

Write SLOs users feel:

- Availability of `/assist` 
- p95 draft latency 
- Grounded-answer rate 
- Harmless-response rate for safety probes 
- Cost per 1000 successful tasks 

Page on-call for availability/latency/safety; review quality in business hours unless severity high.

### H5. Partner / vendor MCP supply chain

Before enabling a vendor MCP server:

1. Threat model (what data it can read/write) 
2. Auth method (OAuth scopes minimized) 
3. Changelog & version pin policy 
4. Sandbox trial on synthetic data 
5. Legal review if data leaves boundary 
6. Kill switch in config 

“It was in a directory” is not a control.

### H6. Streaming UX + tools

Stream text for perceived performance; still gate **actions** until tool results validate. Never execute a write based on a partial streamed plan without a completed, validated tool call sequence.

### H7. Eval for multi-turn agents

Single-turn gold sets under-test agents. Add:

- Scripted multi-turn goals 
- Distractor tools 
- Mid-dialog policy changes 
- Budget exhaustion behavior 
- Recovery after tool errors 

Score **task completion**, not eloquence.

### H8. Data labeling workflow

Who labels? With what rubric? Calibration sessions? Dual annotation on hard slices? Inter-annotator agreement tracked? Architects who ignore labeling quality ship false confidence.

### H9. Cross-functional RACI snippet

| Activity | Architect | Eng | Data | Security | Ops |
| --- | --- | --- | --- | --- | --- |
| Tool authZ model | A/C | R | C | A | I |
| Eval gate thresholds | A | R | R | C | I |
| Index tenancy | C | R | A | A | I |
| On-call runbooks | C | C | I | C | R/A |

(RACI: R=responsible, A=accountable, C=consulted, I=informed)

### H10. More traps

13. Assuming streamed partial output is safe to act on. 
14. No idempotency on payment/ticket tools. 
15. Eval only single-turn when product is agentic. 
16. Enabling vendor MCP without scope review. 
17. Quality paging that burns out on-call with non-actionable fluffs.

### H11. Q&A 37–45

**Q37.** Why can’t Claude “remember” a refund across retries by itself? 
**A.** Stateless tool side effects need server-side idempotency keys.

**Q38.** What flag dimensions matter for AI rollback? 
**A.** Model, prompt/Skill, index, tool allowlist, budgets, HITL requirements.

**Q39.** Name two multi-turn agent eval additions. 
**A.** Distractor tools; recovery after tool errors (also budget exhaustion, policy mid-changes).

**Q40.** What is summary drift? 
**A.** Compacted/summarized history omits or distorts constraints/facts.

**Q41.** Who should be accountable for index tenancy controls? 
**A.** Typically Data + Security accountable; Eng responsible for implementation—per RACI.

**Q42.** Why pin MCP server versions? 
**A.** Prevent surprise tool schema/behavior changes breaking agents/evals.

**Q43.** What SLO belongs on a safety probe suite? 
**A.** Harmless-response / attack-blocked rate with alert thresholds.

**Q44.** Stream tokens to UI but when commit a database write? 
**A.** Only after validated completed tool flow—not on partial plan text.

**Q45.** What is a labeling calibration session? 
**A.** Annotators align on rubric with examples to raise agreement before scoring.

---

*End of expanded integration/eval notes.*



---

## Part I — Integration patterns cheat-sheets

### I1. API gateway in front of Claude

Place an AI gateway for: auth, quotas, prompt size limits, PII redaction hooks, model routing, and centralized traces. Application teams call the gateway, not raw provider keys. Rotate keys centrally; attribute cost by team tags.

### I2. Event-driven tool results

Long tools (report generation) should acknowledge immediately, push completion events, and resume the agent/workflow with `tool_result` when ready. Avoid holding HTTP requests open for minutes without a product reason.

### I3. Embedding model change protocol

1. Freeze writes or dual-write. 
2. Reindex offline. 
3. Shadow queries compare old/new. 
4. Cut traffic. 
5. Keep old index until rollback window ends.

Never silently change embedding models under a production RAG system.

### I4. Q&A 46–50

**Q46.** Why put an AI gateway in front of provider APIs? 
**A.** Centralize auth, quotas, redaction, routing, and cost attribution.

**Q47.** How should a 10-minute report tool integrate with an agent? 
**A.** Async job + event/callback resume—not a single blocking HTTP call without design.

**Q48.** What is dual-write during reindex? 
**A.** Writing embeddings to old and new indexes during migration to allow cutover/rollback.

**Q49.** Name three gateway tags useful for FinOps. 
**A.** Team, product, environment (also use-case, model route).

**Q50.** Silent embedding model change risk? 
**A.** Retrieval quality shifts with no reproducible version—breaks evals and incidents.



---

## Part J — Integration exam drill narratives

When a stem lists latency + tenancy + citations, your answer must touch **all three**. Practice composing multi-constraint answers:

1. Name the pattern (RAG workflow vs agent). 
2. Name the control (tenant filter server-side). 
3. Name the proof (citation validator + eval gate). 
4. Name the perf lever (cache, k, model route). 

### J1. Tool description quality rubric

Poor: `search` — “searches stuff.” 
Good: `search_policies` — “Semantic+keyword search over HR policies for the caller’s tenant. Args: query (string), product (enum), top_k (1–20). Returns clause_id, title, snippet. Does not search other tenants.”

### J2. Conversation memory vs system of record

Never treat chat memory as source of truth for balances, entitlements, or inventory—**tool-fetch** at decision time. Memory may hold preferences and task state only.

### J3. Backpressure

If tool or model latency rises, shed optional enrichment, shrink k, degrade to FAQ templates, or queue with user-visible status. Define these modes in design reviews.

### J4. Q&A 51–55

**Q51.** What four ingredients appear in multi-constraint integration answers? 
**A.** Pattern, control, proof/eval, performance lever.

**Q52.** Why refetch balances instead of trusting chat memory? 
**A.** Memory is stale/hallucinable; systems of record are authoritative.

**Q53.** Example of backpressure when retrieval slows? 
**A.** Reduce top-k, skip rerank, or serve cached FAQ with partial status.

**Q54.** What makes a tool description “good”? 
**A.** Clear scope, args, returns, and explicit non-goals (e.g., tenancy).

**Q55.** Citation validator checks what minimally? 
**A.** That cited IDs exist in the retrieved set provided to the model.

*(File continues — closing summary and deep-dive Parts K–T follow.)*


## Closing integration summary (review)

Integration (19%) asks whether you can connect Claude to enterprise reality without leaking tenancy, without unbounded agents, and without flying blind on quality. Evaluation (16%) asks whether you can prove behavior with datasets, experiments, and telemetry—and optimize in the right order. If you remember only five slogans, remember these:

1. **AuthZ in the tool server.** 
2. **RAG with citations or refuse.** 
3. **Budgets on every agent loop.** 
4. **Gold + adversarial + regression evals.** 
5. **Observability with versions (model, prompt, index, tools).** 

Re-read Parts B–E the night before the exam; skim decision trees once.


### Final integration self-check

Before exam day, ensure you can whiteboard: hybrid RAG with tenant filters; an MCP allowlist story; an agent with max-turn and dollar budgets; a CI eval gate; and a cost-anomaly runbook. If any whiteboard feels fuzzy, reread the matching part above and redo two Q&As aloud.


*(Primary-study deep-dive Parts K–T follow below.)*


---

## Part K — Primary-study deep dive: Integration (19%)

Integration is the heaviest official domain. Treat every enterprise connection as a **trust boundary** plus a **latency budget** plus an **observability surface**.

### K1. Transport decision matrix (expand)

| Mechanism | Best fit | AuthZ story | Failure mode if misused |
| --- | --- | --- | --- |
| Direct Messages API + client tools | Product runtime with 1–N bespoke tools | App identity + per-tool server checks | Ad-hoc tools sprawl without catalog |
| MCP servers | Shared connectors across apps/Claude Code | Server enforces; client still scopes | Unvetted third-party servers = supply chain |
| CLI / local automation | Builder & CI loops | Local creds / OIDC device flows | Using CLI secrets in customer SaaS path |
| Agent-to-agent (A2A) | Clear role split across services | Per-agent credentials + hop audit | Chatty swarm without budgets or owners |
| Server tools (Anthropic-hosted) | Web fetch/search/code where allowed | Understand egress & data handling | Enabling “because cool” in regulated tenants |
| Event bus + workers | Long tools / human queues | Queue ACLs + idempotency keys | Dual-writes without correlation IDs |

**Exam heuristic:** Prefer **MCP** when reuse and discovery matter; **direct tools** when latency and ultra-custom control dominate; **A2A** only with a supervisor and schemas; **CLI** for builders, not end-customer prod paths (unless the product *is* a CLI).

### K2. Capability bloat analysis (exam favorite)

Symptoms of tool/agent bloat:
- Model picks wrong tool among near-duplicates 
- Context window dominated by tool JSON schemas 
- Latency rises before first useful action 
- Security review cannot enumerate capabilities 

Mitigations:
1. **Allowlists** per agent role (support ≠ admin). 
2. **Collapse** overlapping tools (`get_order` not five variants). 
3. **Progressive discovery / tool search** so definitions load on demand. 
4. **Description quality rubric**: name, when-to-use, when-not-to-use, side effects, idempotency. 
5. **Eval** tool-selection accuracy on realistic transcripts.

### K3. AuthN/Z gap checklist (walk every design)

- [ ] User identity propagated (not a shared god service account for all tenants) 
- [ ] Tool execution checks **resource-level** authZ in *your* code—not “Claude pinky-promise” 
- [ ] Tenant ID from verified token, never from model-produced arguments alone 
- [ ] Secrets in vault/env—never in prompts, Skills, or MCP URLs committed to git 
- [ ] Separate credentials for read vs write tools 
- [ ] Break-glass admin path is human + ticketed, not an agent skill 
- [ ] Audit log: who, which tool, which resource, correlation ID, success/fail 

### K4. Accuracy–latency configuration trade-offs

| Knob | Accuracy ↑? | Latency ↑? | Notes |
| --- | --- | --- | --- |
| Higher-capability model | Often | Yes | Measure on hard slice |
| More retrieval k | Sometimes | Yes | Noise risk |
| Reranker | Often precision↑ | Mild | Worth it for FAQ/policy |
| Thinking/effort | Hard tasks↑ | Yes | Cache interactions |
| Extra critique loop | Yes | Yes | Cap iterations |
| Streaming | UX feel↑ | TTFB↓, total≈ | Tools still block |
| Parallel tool calls | Wall-clock↓ | Fan-out cost | Only if independent |

Justify each knob with **metric + budget**. Blindly maxing all knobs fails scenario questions.

### K5. Observability at scale

Minimum viable AI observability:
1. **Request trace**: ingress → model → tools → validators → egress 
2. **Token & $**: input/output/cache/thinking per route 
3. **Quality proxies**: thumbs, edit distance, human overturn, citation coverage 
4. **Safety**: injection hits, DLP blocks, refusal rates 
5. **Saturation**: queue depth, tool p95, rate-limit 429s 

Without traces, “hallucination” tickets become unactionable.

---

## Part L — RAG pipeline design lab (Integration)

### L1. End-to-end decision worksheet

Fill for any corpus:

| Decision | Options | Your pick | Why |
| --- | --- | --- | --- |
| Chunk unit | tokens / section / symbol / slide | | |
| Overlap | 0–20% | | |
| Sparse index | BM25 yes/no | | |
| Dense embeddings | model + dim | | |
| Metadata filters | tenant, locale, version, ACL | | |
| k + rerank | | | |
| Citation required? | yes/no | | |
| Freshness SLO | | | |
| Empty-retrieval behavior | refuse / clarify / broaden | | |

### L2. Query-pattern → retrieval strategy

| Query shape | Strategy |
| --- | --- |
| Exact ID / error code / SKU | Keyword/BM25 heavy; metadata |
| Paraphrased “how do I…” | Dense + rerank |
| Multi-hop “who owns X that depends on Y” | Graph or multi-retrieve plan; maybe agentic retrieve |
| Time-scoped “policy as of March” | Effective-date filters mandatory |
| Tenant-specific | Hard ACL filter pre-retrieve—not prompt-only |

### L3. RAG failure-mode deep table

| Failure | Detect with | Fix lever |
| --- | --- | --- |
| Wrong chunk retrieved | Retrieval@k labels | Chunking, hybrid, rerank, query rewrite |
| Right chunk, wrong answer | Groundedness eval | Prompt: “answer only from sources”; cite |
| Stale policy cited | Doc version tests | Freshness pipeline + tombstones |
| Cross-tenant leak | Red-team tenant tests | Index isolation / filter correctness |
| Citation theater | Cite-but-unused checks | Require quote spans / ids |
| OCR garbage | Field validators | Human quarantine; vision quality gates |
| Over-retrieve | Token & latency dashboards | Lower k; pack better |

### L4. Progressive discovery for RAG + tools

Monolithic: load 80 tools + 40 docs every turn → cost and confusion. 
Progressive: classify intent → fetch 2–5 tools → retrieve top chunks → call → maybe second retrieve. 
**Stop conditions:** answerable with citations; need clarify; refuse; escalate.

---

## Part M — Evaluation, Testing & Optimization (16%) deep dive

### M1. Metric portfolio (define before building)

| Family | Examples | Owner |
| --- | --- | --- |
| Task quality | Exact match, F1, rubric score, groundedness | Applied science / product |
| Latency | TTFB, p95 end-to-end, tool p95 | Platform |
| Cost | $ / successful task, cache hit rate | Platform + FinOps |
| Safety | Injection ASR, policy violation rate | Security/AI risk |
| Business | CSAT, handle time, conversion | Product |
| Ops | Error rate, 429s, queue age | SRE |

**Exam trap:** Optimizing BLEU-like n-gram scores for a tool-using agent. Prefer **task success** and **tool correctness**.

### M2. Dataset design methodology

1. **Production sampling** (privacy-reviewed) + synthetic edge cases. 
2. **Stratify** by intent, language, tenant size, difficulty. 
3. **Gold labels** with inter-annotator agreement checks. 
4. **Hold out** a frozen regression set; never train prompts on the whole pool without a holdout. 
5. **Refresh** monthly for drift; version datasets like code.

Mixed methods: automated graders + LLM-as-judge (with known biases) + human expert panels for high-stakes.

### M3. Experiment discipline

- One change per experiment when possible (prompt XOR model XOR retrieval). 
- Pre-register success thresholds. 
- Use paired tests on the same cases. 
- Ship behind flags; shadow traffic before cutover. 
- Record negative results—avoid thrashing.

### M4. Diagnosis tree (hallucination vs prompt vs model vs retrieval)

```
Bad answer reported
 ├─ Sources missing/wrong? → Retrieval path
 ├─ Sources good, answer invents? → Grounding prompt + decode constraints + cite required
 ├─ Format broken? → Schema/prompt/examples
 ├─ Tool args wrong? → Tool descriptions + authZ + few-shot tool use
 ├─ Only hard cases fail? → Model tier / thinking
 └─ Intermittent? → Temperature/nondeterminism, timeouts, partial tool results
```

### M5. Optimization order (do not shuffle casually)

1. Fix correctness bugs (wrong tools, bad retrieval). 
2. Shrink context (better retrieve, fewer tools). 
3. Cache stable prefixes. 
4. Route easy traffic to cheaper/faster models. 
5. Then consider Batch, parallelism, distillation-like patterns.

Optimizing cost before correctness creates silent wrong systems.

### M6. CI/CD for AI systems

| Gate | Blocks merge if… |
| --- | --- |
| Unit: schema/tool contract tests | Breaking tool JSON |
| Golden set regression | Score drop > threshold |
| Safety pack | Injection/DLP cases fail |
| Latency budget smoke | p95 exceeds budget on canary size |
| Cost estimate | Projected $ / 1k req above cap |


### M7. Model judges vs code judges + calibration (P0)

LLM-as-judge is a **measurement instrument**, not a vibe oracle. Architects choose the grader the way they choose a unit-test framework: prefer **deterministic checks** when the contract is machine-checkable; use **model judges** when quality is open-ended—and **calibrate** before the score drives ship/no-ship.

#### When code / deterministic assertions beat model judges

| Signal that code wins | Examples |
| --- | --- |
| Exact or structured output | JSON schema valid; required fields present; enum membership |
| Grounding mechanics | Citation IDs ⊆ retrieved chunk IDs; URL allowlist; quota math |
| Tool correctness | Correct tool name chosen; args pass schema; idempotency key present |
| Policy engines already exist | Refund ≤ cap; entitlement check boolean; PII regex/DLP hit |
| Safety probes with known strings | Forbidden substring; secret pattern; SSRF to link-local IP |
| Regression of formatting SLAs | Word/char limits; language code; “must include disclaimer X” |

**Rule:** If a junior engineer could write a pytest without reading the essay, **don’t** burn tokens on a judge.

#### When model judges are needed

| Signal that a rubric judge helps | Examples |
| --- | --- |
| Open-ended quality | Empathy, clarity, teaching quality, summary usefulness |
| Multi-factor rubrics | “Helpful + harmless + on-brand” with partial credit |
| Paraphrase equivalence | Many correct phrasings; exact match too brittle |
| Comparative preference | A/B pairwise “which draft is better for this persona” |
| Nuanced faithfulness | Soft check that claims are supported when citations are prose-y |

Even then: combine with code gates (schema, cite-ID ⊆ retrieval) so the judge never scores garbage JSON as “beautiful.”

#### Calibration against human gold (before trust)

```
1. Sample N items stratified by intent/difficulty (start ~50–200; size up for high stakes).
2. ≥2 qualified humans score with the SAME rubric; measure agreement (e.g., Cohen’s κ / % exact).
3. Run the candidate judge (model + prompt + effort/thinking config + reference policy) blind on the same items.
4. Compare judge vs human: correlation, systematic bias (leniency/severity), slice failures.
5. Only promote the judge if agreement clears a pre-set bar; otherwise fix rubric/examples or keep humans.
6. Re-calibrate on model upgrades, rubric edits, or domain drift—version the judge like code.
```

**Exam trap:** “Judge score went up 8% after prompt tweak → ship.” Without calibration, you may have taught the judge to love verbosity.

#### Thresholds & rollback — freeze BEFORE seeing experiment data

| Pre-register (write down first) | Why |
| --- | --- |
| Primary metric + direction | Stops metric shopping |
| Minimum lift / maximum allowed drop | Ship bar |
| Guardrail metrics (safety, latency, cost) | Prevents winning on quality while failing compliance |
| Sample size / evaluation set ID | Reproducibility |
| Rollback criteria | e.g. “any safety probe fail” or “p95 +20%” → auto-revert flag |

Peeking at results then choosing thresholds is **p-hacking**. Architects treat eval configs as controlled artifacts in git.

#### Mini worksheet — grader design (fill per feature)

```
Feature / route: _______________________
Business job: _______________________
Machine-checkable contracts (code judge):
 - [ ] _______________________
 - [ ] _______________________
Open-ended dimensions (model judge):
 - [ ] _______________________
Human gold plan: N=___ · raters=___ · agreement bar=___
Judge model + prompt hash: _______________________
Calibration result: pass / fail · date: ___
Pre-registered ship threshold: _______________________
Pre-registered rollback triggers: _______________________
Owner: _______________________
```

#### M7 exam traps

1. LLM-as-judge for schema validity. 
2. No human calibration set. 
3. Changing success thresholds after seeing the A/B dashboard. 
4. Single vague “quality 1–5” score with no rubric anchors. 
5. Judge model identical to generator without bias analysis (shared blind spots). 
6. Ignoring slice failures (“great average, fails on medical intents”). 
7. Using judge scores as sole prod gate with no code assertions.

#### M7 Q&A (81–88)

**Q81.** Refund tool args must be `{order_id, amount_cents}` with amount ≤ policy cap. Prefer code or model judge? 
**A.** **Code** (schema + policy engine). Model judge is optional narrative gloss, not the control.

**Q82.** Why calibrate LLM-as-judge against humans before CI gating? 
**A.** To measure bias/agreement; uncalibrated judges can reward style over correctness and create false ship signals.

**Q83.** Select TWO that belong in a pre-registered eval plan. 
**A.** Primary metric with threshold **and** rollback/guardrail criteria—written before looking at experiment outcomes.

**Q84.** Summaries graded only by exact string match keep failing good paraphrases. What do you do? 
**A.** Keep code checks for must-include facts/citations; add a **calibrated** rubric judge (or human sample) for paraphrase quality—don’t abandon factual assertions.

**Q85.** Judge scores rise after you switch the judge to the same model family as the generator. Concern? 
**A.** **Shared bias / self-preference**; re-calibrate vs humans and consider a different judge model or pairwise protocol.

**Q86.** What is a sound rollback criterion example? 
**A.** “If adversarial injection ASR > X% or citation-ID validation fail rate > Y% on canary, auto-disable flag”—set **before** the canary.

**Q87.** Offline gold looks great; prod thumbs-down spike. First eval move? 
**A.** Suspect **distribution shift**; sample prod failures into a new slice; don’t only retune the judge prompt to match thumbs.

**Q88.** When are human panels still required even with a calibrated judge? 
**A.** High-stakes / irreversible domains, novel failure modes, and periodic audit of the judge itself—automation supplements, not replaces, accountability.


---

## Part N — Architecture decision scenarios (Integration + Evals)

### N1. MCP vs direct API for CRM + billing + knowledge base

**Prefer MCP** if three+ apps (support bot, Claude Code, internal agent) need the same connectors and you will maintain a catalog. 
**Prefer direct tools** if only one latency-critical service needs two tightly customized tools. 
**Hybrid** is common: MCP for long-tail systems; direct for the hottest path.

### N2. Support agent with refunds and policy RAG

Workflow: retrieve policy → draft → **policy engine validates refund** → tool executes. 
Evals: policy citation groundedness + refund decision exactness + latency. 
AuthZ: refund tool capped; amount from code, not free-form model float without check.

### N3. Observability missing after GA

Symptom: rising “wrong answer” tickets, no traces. 
Fix: add correlation IDs, store redacted prompts/tool I/O, build replay harness, freeze a regression set from tickets.

### N4. Accuracy up, latency broken

You added critique loop + k=20 retrieve + opus-class. 
Rollback to baseline; A/B each knob; keep only levers with proven lift per +100ms.

---

## Part O — Production checklists

### O1. Integration go-live

- [ ] Transport choice documented with alternatives rejected 
- [ ] Tool inventory ≤ agreed max; descriptions reviewed 
- [ ] Progressive discovery plan for >N tools 
- [ ] AuthZ tests including negative tenant cases 
- [ ] Timeouts, retries, idempotency on all side-effecting tools 
- [ ] Rate-limit and backpressure behavior defined 
- [ ] MCP servers allowlisted; owners named; versions pinned 
- [ ] Network egress reviewed for server tools

### O2. RAG go-live

- [ ] Chunking eval vs baseline 
- [ ] Hybrid retrieval justified 
- [ ] ACL filters tested 
- [ ] Freshness SLO monitored 
- [ ] Empty-hit behavior coded 
- [ ] Citation contract enforced when required

### O3. Eval go-live

- [ ] Metrics tied to business + safety 
- [ ] Golden set versioned 
- [ ] CI regression gate on 
- [ ] Online feedback sampled 
- [ ] Cost/latency dashboards live 
- [ ] Owner for weekly eval review

---

## Part P — Failure-mode catalog (ops)

| Mode | Signal | Mitigate |
| --- | --- | --- |
| Tool timeout storms | p95 tool ↑, partial answers | Bulkheads, circuit breakers, degraded UX |
| Prompt injection via retrieved HTML | Odd tool calls | Sanitize; ignore instruction-like content; allowlists |
| Embedding model change | Sudden quality drop | Dual-embed migration; reindex protocol |
| Cache stampedes | Billing spike | Stabilized prefixes; monitor hit rate |
| Eval overfitting | Great CI, bad prod | Fresh prod samples; adversarial set |
| Non-idempotent refund double-charge | Duplicate tool_use | Idempotency keys + server dedupe |
| Conversation state loss | Amnesia mid-task | Durable state store; don’t rely on client only |
| MCP server compromise | Unexpected exfil tools | Supply-chain review; network egress allowlists |

---

## Part Q — Extended Q&A (56–65) Integration + Evals

**Q56.** Select TWO symptoms of capability bloat: (a) wrong tool selection (b) faster p95 (c) huge tool defs in context (d) higher cache hit rate. 
**A.** **(a)** and **(c)**.

**Q57.** Best place to enforce “user can only see their org’s tickets”? 
**A.** **Server-side authZ** on the tool/database query using verified identity—not the system prompt alone.

**Q58.** RAG returns excellent chunks but answers invent clauses. First lever? 
**A.** Strengthen **grounding instructions + required citations + refuse if unsupported**; add groundedness eval.

**Q59.** When is agent-to-agent justified over one workflow? 
**A.** When specialized roles need **separate credentials/contexts** and a supervisor owns side effects—not for cosmetic “multi-agent” demos.

**Q60.** Order of optimization after quality is acceptable? 
**A.** Context shrink → caching → routing to cheaper models → batch/parallel where safe.

**Q61.** Multiple-response: CI should block on (select THREE): schema break, golden regression drop, safety pack fail, vanity n-gram uptick alone, CSS color change. 
**A.** Schema break, golden drop, safety fail.

**Q62.** Progressive discovery vs monolithic—large MCP catalog? 
**A.** **Progressive / tool search**; load definitions as needed.

**Q63.** Hybrid retrieval helps most when… 
**A.** Queries mix **semantic paraphrases** and **exact tokens** (IDs, codes, clause numbers).

**Q64.** Online eval signal for support copilot? 
**A.** Human edit rate / overturn rate / CSAT linked to trace IDs—not only offline BLEU.

**Q65.** Server tool web fetch in a HIPAA workspace—architect action? 
**A.** **Disable or tightly gate** with DLP and data-handling review; prefer private retrieval.

---

## Part R — Rapid review (Integration 19% + Evals 16% ≈ 35%)

- Integration = trust boundary + latency + observability. 
- MCP for reusable connectors; direct tools for bespoke hot paths. 
- Least privilege + resource authZ in code. 
- Fight tool bloat; progressive discovery at scale. 
- RAG: chunk→hybrid retrieve→rerank→cite→validate empty hits. 
- Evals before GA; optimize correctness then cost. 
- Traces or it didn’t happen. 
- Idempotency for side effects; pin MCP/embedding versions.

*Heaviest file by design. Pair with `01` patterns and `03` guardrails.*


---

## Part S — End-to-end reference scenarios (Integration + Evals heaviest drill)

### S1. Enterprise policy assistant (full stack)

**Requirements:** 40k employees, 12k policy docs, multi-locale, union-sensitive HR policies, p95 < 4s for standard Q&A, citations mandatory, no cross-region HR data mixing.

**Architecture sketch:**
1. Ingest with section-aware chunking + effective dates + locale + ACL tags. 
2. Hybrid retrieve + rerank; hard filter ACL+locale. 
3. Claude mid-tier with cached HR answer policy; require citations. 
4. Empty retrieve → clarify/refuse. 
5. Eval: groundedness, citation precision, ACL red-team, latency, cost. 
6. Observability: trace_id, doc versions cited, cache hit rate.

**Reject:** Monolithic context of all policies; shared god credentials; agent with write access to HRIS on day one.

### S2. Tool-using IT support agent

**Tools:** ticket read/write, password reset (sensitive), knowledge RAG, calendar read. 
**Controls:** password reset = HITL or step-up auth; ticket write scoped; progressive tool discovery; eval tool-selection + harmful action attempts.

### S3. Eval harness blueprint (copy/adapt)

```
datasets/
 golden_vN.jsonl
 safety_injection.jsonl
 acl_redteam.jsonl
runners/
 offline_score.py
 latency_budget.py
ci/
 gate.yaml # thresholds
dashboards/
 online_feedback.md
```

Thresholds example: groundedness ≥ 0.85; injection ASR ≤ 0.05; p95 ≤ budget; $ / 1k req ≤ cap.

### S4. Trade-off table — retrieval quality vs latency

| Config | Quality (typical) | p95 impact | Cost impact |
| --- | --- | --- | --- |
| k=3 dense only | OK semantic | Low | Low |
| k=10 hybrid + rerank | Better precision | Medium | Medium |
| k=30 + critique loop | Marginal gains | High | High |
| Agentic multi-hop retrieve | Multi-hop↑ | High variance | High |

Pick the **minimal** config that hits the quality bar on the stratified eval set.

### S5. Failure injection tests (production mindset)

- Tool 500s and timeouts 
- Empty retrieval 
- Partial JSON tool results 
- Stale index (old policy version) 
- Cross-tenant IDOR attempts 
- Prompt injection in HTML fetch 
- Duplicate tool_use (idempotency)

### S6. Extended Q&A (66–72)

**Q66.** Citation required but model cites docs not in retrieved set—what gate? 
**A.** **Verifier** that citation IDs ⊆ retrieved IDs; else regenerate or refuse.

**Q67.** Select TWO for multi-region HR assistant: region-local indexes, single global index without filters, verify region from token, trust model-stated region. 
**A.** Region-local indexes (or hard filters) and verify region from token.

**Q68.** Best CI signal that retrieval regressed after chunker change? 
**A.** Drop in **retrieval recall@k / groundedness** on frozen set.

**Q69.** Event-driven tool pattern helps when… 
**A.** Tools are long-running; user gets progress via channel; correlation IDs tie results back.

**Q70.** Embedding model upgrade protocol—select TWO: dual-write/reindex plan, flip overnight without eval, version indexes, delete old index same second as deploy. 
**A.** Dual-write/reindex plan and version indexes.

**Q71.** Observability “three pillars” for AI paths? 
**A.** Traces (tool+model), metrics (latency/cost/quality proxies), structured logs (redacted).

**Q72.** Capability bloat fix order? 
**A.** Inventory → dedupe → allowlist by role → progressive load → eval tool pick accuracy.

---

## Part T — Desk sheet (35% combined)

Integration 19%: transport, RAG, authZ gaps, bloat, progressive discovery, latency knobs, monitoring, **CSP/route governance**. 
Evals 16%: metrics, datasets, A/B, diagnosis, optimize order, CI gates, **code vs model judges + calibration**. 
Together: prove it works under enterprise constraints—or it is not an architecture.

*Heaviest technical file by blueprint weight. P0 gap closures: Part D5 (CSP routes), Part M7 (judges).*
