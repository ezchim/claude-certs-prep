---
title: MSO Foundations (Model Selection & Optimization)
pack: CCDV-F Developer Foundations
disclaimer: Original study notes — independent and not official course content
approx_length: ~6500–9000 words (primary study)
updated: 2026-08-23
---

# Chapter 01 — MSO Foundations
## Model Selection & Optimization fundamentals

> **Disclaimer:** Original exam-prep notes for focused reading. Grounded in **public** Anthropic docs on choosing models, effort, context windows, pricing concepts, and caching/batch cost levers — Model IDs and prices change; memorize *decision criteria*, then verify current cards on exam day.

**Maps primarily to:** Model Selection and Optimization (**16.8%**). 
**Also feeds:** Applications and Integration (when the stem is “pick API path for cost/latency”).

**How to study:** Learn the three-axis tradeoff (capability × latency × cost), then practice forcing a single best answer under one hard constraint. Most MSO items are constraint puzzles, not “which model is smartest.”

---

## 1. Overview

MSO is the skill of **routing work to the right intelligence budget**. On Claude that budget is expressed through:

1. **Model family / tier** (frontier vs mid vs fast/cheap).
2. **Effort / thinking configuration** (how hard a *single* model works).
3. **Context and output ceilings** (window size, max output tokens).
4. **Serving path** (interactive Messages vs streaming vs Message Batches vs cached prefixes).
5. **Operational knobs** (pinning versions, aliases vs full IDs, region/cloud host).

Public docs frame model choice as balancing **capabilities, speed, and cost**. Recent Opus/Sonnet-class models add an **effort** parameter that trades intelligence for latency/cost *within* one model — often a better first lever than jumping tiers.

**Exam posture:** Prefer the **smallest sufficient** model/effort that meets quality gates on your evals. Over-buying intelligence is a common wrong answer when the stem emphasizes cost or p95 latency.

---

## 2. Key map (flashcard)

| Concept | One-line meaning | Exam trigger words |
| --- | --- | --- |
| Model tier | Capability class (frontier / balanced / fast) | “complex refactor,” “classify at scale,” “chat UI” |
| Effort | Test-time compute within a model | “same model, better answers,” “lower latency same model” |
| Context window | How much prompt+history fits | “long repo,” “1M context,” “truncate” |
| Prompt caching | Prefix reuse for cheaper/faster repeats | “stable system+tools,” “multi-turn agent” |
| Message Batches | Async bulk at discount | “overnight eval,” “latency tolerant” |
| Streaming | Token-by-token UX / early cancel | “TTR,” “user-facing chat” |
| Pinning | Freeze model string for reproducibility | “prod regression,” “version drift” |
| Alias | Moving pointer to “latest sonnet/opus/…” | “always newest,” “dev sandbox” |

---

## 3. Deep notes

### 3.1 The three-axis decision frame

Every production choice sits on a triangle:

```text
 CAPABILITY
 /\
 / \
 / \
 / ? \
 /________\
 COST LATENCY
```

- Move toward **capability** when failure is expensive (security review, multi-hour agent, hard coding).
- Move toward **latency** when humans wait on first token or full answer (support chat, IDE inline).
- Move toward **cost** when volume dominates (classification, extraction, nightly batches).

**Rule of thumb for exams:** Identify which vertex the stem hard-constrains, then eliminate options that ignore it.

### 3.2 Model family mental model (stable even as names change)

Public choosing-a-model guidance (2026) organizes roughly as:

| Workload pattern | Prefer (conceptual) | Why |
| --- | --- | --- |
| Multihour agents, deep reasoning, large refactors, vision-heavy enterprise work | Frontier Opus-class (e.g. Claude Opus 5 family in public docs) | Highest autonomy and reasoning headroom |
| Daily coding, data analysis, content, solid agentic tool use | Sonnet-class | Best default balance for most apps |
| High-QPS routine classification, simple transforms, cheap loops | Haiku-class | Speed and unit economics |
| Longest / hardest frontier agent runs (where available) | Fable-class tier (a model family sitting above Opus; Claude Code’s `best` alias resolves interactively to the top tier) | Explicitly for hardest long-running tasks |

**Do not memorize marketing slogans.** Memorize **fit**:

- **Tool-heavy coding agents** → mid-to-frontier + appropriate effort.
- **Structured extraction with gold schema** → often mid tier + strict schema/tool; escalate only if evals fail.
- **User-facing low-latency assistants** → faster tier or same tier at lower effort; stream.

### 3.3 Effort as the intra-model dial

Public docs describe **effort** on recent Opus/Sonnet models as trading intelligence for latency and cost. Defaults are often **high**; higher settings (e.g. `xhigh`, `max` where offered) help demanding agentic/coding work; lower settings help routine tasks.

**Operational pattern:**

1. Pin a model.
2. Start at the **documented default** for that model.
3. Raise effort only when evals show systematic under-reasoning.
4. Lower effort when p95 latency/cost budgets break and quality still passes.

**Exam trap:** Switching models when the stem says “keep the same model ID for cache stability / contract” — answer is usually **change effort**, not model.

**Cache note:** Changing thinking configuration invalidates **message-level** cache breakpoints; the tools/system prefix portion stays cacheable. Matching effort/thinking between pre-warm and live traffic is still the safe operating rule — a mismatched warm can leave your message-level entries unread.

### 3.4 Context windows and output limits

Treat context as a **budget**, not a dump:

| Pressure | Symptom | MSO response |
| --- | --- | --- |
| History too long | Truncation, lost instructions | Compact summaries; move durable rules out of chat |
| Huge tool schemas | Wasted prefix tokens | Tool search / defer loading; fewer tools |
| Large docs every turn | Cost explosion | Cache stable docs; retrieve slices |
| Need long outputs | Cutoff mid-JSON | Raise `max_tokens`; structured streaming; chunk tasks |

Public materials highlight large windows (including **1M**-class windows on some models) and high max output on frontier models. Exams care that you **know when** to request a large window alias (long sessions / big repos) versus when retrieval + caching is cheaper than stuffing everything.

### 3.5 Cost levers that are still “MSO”

MSO is not only “pick Opus vs Sonnet.” Cost-aware answers often combine:

1. **Smaller model** if quality holds.
2. **Lower effort** if quality holds.
3. **Prompt caching** for stable prefixes (system, tools, reference docs).
4. **Message Batches** when latency can wait (public docs: async bulk, discounted vs sync; caching stacks but hits are best-effort).
5. **Shorter outputs** (ask for concise; schema-constrained).
6. **Avoid rewriting the cached prefix** (tool order shuffle, timestamps in system prompt, mid-session model swap).

**Batch vs realtime decision tree:**

```text
Is a human waiting on this response in < few seconds?
 YES → Sync Messages (+ stream if UX needs tokens early)
 NO → Are there many independent requests and hours-ok SLA?
 YES → Message Batches (+ shared cache_control, prefer longer TTL if available)
 NO → Sync with caching; consider queue workers
```

Public batch caveats worth remembering: no streaming results path, no sync-only speed modes, no `max_tokens: 0` pre-warm inside batches, stateful thread params generally don’t apply.

### 3.6 Latency levers

| Lever | Effect | Side effect |
| --- | --- | --- |
| Faster model tier | Lower TTFT / total time | Possible quality drop |
| Lower effort | Faster | Possible quality drop |
| Streaming | Better *perceived* latency | Same total compute roughly |
| Prompt cache hit | Faster prefill | Requires stable prefix |
| Smaller prompts | Faster | May need better retrieval |
| Parallel tool calls (where supported) | Wall-clock win | More concurrency complexity |

**Exam nuance:** Streaming fixes **UX wait**, not unit cost. If the stem is about **bill**, streaming alone is rarely the answer.

### 3.7 Quality levers (when under-performing)

Escalate in this order unless stem forbids:

1. Fix **prompt/context** (missing constraints, bad examples, noisy tools).
2. Add **tools / MCP** so the model can fetch ground truth.
3. Raise **effort**.
4. Escalate **model tier**.
5. Change **architecture** (multi-step agent, verifier model, human gate).

Jumping straight to the largest model is a frequent distractor.

### 3.8 Version pinning vs aliases

| Approach | Use when | Risk |
| --- | --- | --- |
| Full dated / explicit model ID | Production, evals, compliance | You must plan upgrades |
| Alias (`sonnet`, `opus`, `haiku`, `best`, …) | Dev, “always current” sandboxes | Silent behavior drift |
| Claude Code aliases (`opusplan`, `sonnet[1m]`, etc.) | Session workflow preferences | Still pin for CI reproducibility |

**Production checklist:** pin model string in config; record in eval reports; canary before fleet upgrade; never change model mid-conversation if you rely on prompt cache.

### 3.9 Routing patterns (architecture meets MSO)

**A. Static route:** One model for the product surface. Simple ops, limited optimization.

**B. Tiered route:** Classifier or rules send easy traffic to Haiku-class, hard to Sonnet/Opus-class.

**C. Cascade / escalate:** Try cheap path; if confidence low or schema validation fails, retry expensive path.

**D. Dual-model verify:** Generator on mid tier; critic/verifier on same or higher tier for high-risk outputs.

Exam stems that mention “90% trivial tickets, 10% novel incidents” almost always want **tiered or cascade**, not “everything on frontier.”

### 3.10 Cloud hosting is still MSO-adjacent

Public Academy topics include Bedrock and Vertex/Agent Platform. Selection differences that show up in scenarios:

- **Model ID format** differs by host (Anthropic API name vs Bedrock inference profile ARN vs Vertex version name).
- **Feature lag / parity** — confirm tool, caching, batch, and beta availability on that host.
- **Data residency / IAM** — may force a region even if another region has a newer SKU.

If the stem’s hard constraint is **residency**, the correct model choice is “the approved regional SKU,” not the absolute newest global ID.

### 3.11 Worked mini-scenarios

**Scenario A — Support chatbot, p95 < 2s, moderate quality.** 
Prefer fast/mid tier, lower-to-default effort, stream, cache system+tool prefix. Avoid frontier+max effort.

**Scenario B — Overnight evaluation of 50k prompts.** 
Message Batches; identical `cache_control` blocks; longer cache TTL if offered; mid tier unless evals need frontier.

**Scenario C — Autonomous multi-hour coding agent.** 
Frontier or strong Sonnet/Opus-class with elevated effort; large context as needed; cache tools; do not thrash model mid-run.

**Scenario D — Cost spike after “adding a timestamp to the system prompt.”** 
Cache bust. Fix prefix stability; don’t first jump to a cheaper model.

### 3.12 Metrics that prove an MSO decision

Track together or you will optimize the wrong axis:

| Metric | Asks |
| --- | --- |
| Task success / rubric score | Did quality hold? |
| Cost per successful task | Not cost per raw call |
| p50/p95 latency | Human-waiting paths |
| Cache hit rate | Prefix engineering health |
| Escalation rate | Is routing calibrated? |
| Safety incident rate | Did cheaper path skip filters? |

---

## 4. Decision trees and tables

### 4.1 Master selection tree

```text
START: What fails first if we choose wrong?
│
├─ Safety / legal / high blast radius failure
│ → Higher capability + strong guardrails; human gate as needed
│
├─ Human waiting (interactive)
│ → Prefer speed tier or lower effort; STREAM; cache prefix
│ → Escalate model only if quality evals fail
│
├─ Bulk / offline
│ → BATCHES + shared cache; right-size model via eval sample
│
└─ Long-running agentic coding
 → Capable model + adequate effort; stable tools; pin version
```

### 4.2 Effort vs model switch

| Observation | First action | Avoid |
| --- | --- | --- |
| Same model, weak multi-step plans | Raise effort | Random prompt lengthening alone |
| Cache misses after model A/B test in one thread | Don’t switch mid-thread | “Just try Opus on turn 12” |
| Easy tasks too expensive | Lower effort or smaller model | Keep frontier for everything |
| Quality cliffs on hard 5% | Route hard 5% up | Upgrade 100% of traffic |

### 4.3 Cost stack (combine carefully)

| Stack element | Typical win | Requires |
| --- | --- | --- |
| Smaller model | Large | Eval parity |
| Lower effort | Medium | Eval parity |
| Prompt cache reads | Large on agents | Stable ordered prefix |
| Batches discount | Large on bulk | Latency tolerance |
| Shorter outputs | Medium | Clear instructions / schemas |

---

## 5. Exam traps

1. **Picking the smartest model** when the stem stresses cost or latency.
2. **Streaming as a cost saver** (it isn’t).
3. **Batches for chat UI** (wrong latency class).
4. **Changing tools or model mid-conversation** while relying on cache.
5. **Putting volatile data (timestamps, per-user secrets) in the cached system prefix.**
6. **Aliases in production** without a pin + canary story.
7. **Ignoring host/region constraints** (Bedrock/Vertex residency).
8. **Optimizing cost per token** instead of **cost per successful task**.
9. **Raising max_tokens** to “fix” bad structure — prefer schemas and chunking.
10. **Treating effort and model tier as identical knobs** — they’re layered.

---

## 6. Self-check Q&A (20)

**Q1.** A product needs sub-second perceived replies for FAQ chat. Quality is “good enough” on a mid-tier model. What is the best first optimization? 
**A1.** Keep the mid-tier model, enable **streaming**, ensure prompt **cache** hits on the stable system/tool prefix, and consider **lower effort** if quality evals still pass. Do not jump to frontier.

**Q2.** You must score 200k documents by tomorrow morning; no user is waiting. Best API pattern? 
**A2.** **Message Batches**, optionally with shared cached context and a longer cache TTL when available.

**Q3.** Cache hit rate collapsed after a deploy. Diff shows tool definitions sorted randomly each request. Root cause? 
**A3.** **Prefix mismatch** — non-deterministic tool order breaks prompt caching. Serialize tools deterministically.

**Q4.** Evals show the model under-plans on hard coding tasks; model ID must stay fixed for procurement. Lever? 
**A4.** Increase **effort** (and improve prompts/tools). Do not change model if the constraint forbids it.

**Q5.** Why can switching from Sonnet to Opus mid-thread hurt an agent session beyond raw price? 
**A5.** Caches are **model-scoped**; switching invalidates the cached prefix and can destabilize behavior.

**Q6.** When is Haiku-class the *wrong* answer despite high QPS? 
**A6.** When the stem requires deep multi-step autonomy, subtle policy judgment, or high-cost-of-error reasoning that evals show small models fail.

**Q7.** Alias `sonnet` in CI started failing after a silent upgrade. Prevention? 
**A7.** **Pin** explicit model IDs in CI/prod; use aliases only where drift is acceptable.

**Q8.** Batch jobs show low cache hits despite `cache_control`. Likely miss? 
**A8.** Non-identical prefixes across items, cache TTL too short for async processing, or expecting guaranteed hits (docs: **best-effort** in batches).

**Q9.** Is streaming required for batch processing? 
**A9.** No — batch results are retrieved as completed results/files, not live token streams.

**Q10.** A 1M-context option exists. Should every app enable it? 
**A10.** No — use when sessions/repos truly need it; otherwise retrieval + caching is often cheaper/clearer.

**Q11.** Cascade routing: what signal should trigger escalation? 
**A11.** Validation failure, low self-confidence score, policy risk flags, or rubric fail — not “random 10%.”

**Q12.** Cost per call dropped but user complaints rose. Which metric was missing? 
**A12.** **Task success / cost per successful task** (and possibly safety incidents).

**Q13.** Pre-warm with `max_tokens: 0` — purpose? 
**A13.** Populate prompt cache without producing an answer (sync path); not available inside batches per public batch limitations.

**Q14.** Thinking/effort on pre-warm vs live traffic differs. Symptom? 
**A14.** Message-level cache entries written under one thinking config don’t match live traffic (the tools/system prefix can still hit) → partial misses that look like “caching broken.” Align configs.

**Q15.** Bedrock-only customer in a locked region. Newest Anthropic API model not listed there. Correct action? 
**A15.** Choose the **approved regional** model/profile that meets policy; plan parity tests — don’t invent cross-region calls against policy.

**Q16.** “Make it cheaper” stem; agent uses 40 tools every turn. Strong MSO move? 
**A16.** Keep tool list stable but use **defer loading / tool search** patterns so full schemas aren’t always expanded — plus cache the stable stub prefix.

**Q17.** Dual-model verify pattern — when justified? 
**A17.** High-stakes outputs (legal/medical summaries, prod migrations) where verifier cost < error cost.

**Q18.** p95 latency bad; cache hit rate 5%. First investigation? 
**A18.** Prefix stability (system/tools ordering, volatile system text), not immediately “buy bigger instances of the same mistake.”

**Q19.** Why might raising effort *increase* cost more than expected? 
**A19.** More tokens / tool loops / thinking — bill rises with test-time compute, not just list price.

**Q20.** Map this chapter to exam weight. 
**A20.** Core of **Model Selection and Optimization 16.8%**, plus Integration items that are really cost/latency routing questions.

---

## 7. Checklist

- [ ] I can explain capability vs latency vs cost without naming a specific SKU.
- [ ] I know when to change **effort** vs **model tier**.
- [ ] I can pick sync vs stream vs batch from an SLA sentence.
- [ ] I can list three ways to accidentally bust a prompt cache.
- [ ] I pin models in prod and know alias risks.
- [ ] I track success-rate-adjusted cost, not vanity token spend.
- [ ] I have a cascade/tiered routing story for mixed workloads.
- [ ] I remember cloud host ID/parity/residency caveats.
- [ ] I escalate quality: prompt → tools → effort → model → architecture.
- [ ] I re-check current public model cards before exam day.

---

## 8. Glossary

| Term | Definition |
| --- | --- |
| MSO | Model Selection & Optimization — choosing and tuning intelligence spend |
| Effort | Intra-model dial for test-time compute / thoroughness |
| TTFT | Time to first token — key streaming UX metric |
| Prefix cache | Cached leading portion of a prompt (system/tools/shared docs) |
| Cache bust | Any change that prevents prefix match |
| Message Batches | Async bulk Messages API processing at discounted economics |
| Model pin | Fixed model identifier for reproducible prod/evals |
| Model alias | Indirection that resolves to a moving “latest” model |
| Cascade routing | Cheap attempt first, escalate on failure signals |
| Cost per successful task | Unit economics aligned to user outcomes |
| Context window | Maximum tokens of addressable prompt/history for a model |
| Host parity | Feature/model availability differences across API/Bedrock/Vertex |

---

## Appendix — Chapter → official domains

| Official domain | How Chapter 01 shows up |
| --- | --- |
| Model Selection and Optimization 16.8% | Direct coverage |
| Applications and Integration 33.1% | Batch/stream/cache/pinning choices |
| Agents and Workflows 14.7% | Effort and model for long-horizon agents |
| Eval/Testing/Debugging 2.6% | Using evals to justify MSO changes |
| Others | Light touch |


---

## 9. Expanded deep dive — production MSO playbooks

### 9.1 Building an MSO policy document (what seniors actually ship)

A team that “picks models ad hoc in PRs” will fail Integration-style scenarios. Encode a short **MSO policy**:

1. **Default model** for interactive product surfaces.
2. **Batch model** for offline scoring.
3. **Agent model** for tool-loop coding / ops agents.
4. **Escalation model** for failures / high severity.
5. **Effort defaults** per surface.
6. **Pinning rule** (full IDs in prod; aliases allowed only in personal sandboxes).
7. **Eval gate** — no fleet model change without golden-set delta report.
8. **Cache contract** — ordered tools, frozen system sections, owners for prefix changes.

Exams love answers that sound like **governance**, not vibes: “update the pinned config + canary + eval report,” not “swap to Opus in the chat UI.”

### 9.2 Token economics without memorizing price tables

You do not need exact dollar charts. You need **relative** reasoning:

- Output tokens are usually priced higher than input tokens on public Anthropic pricing pages — so **verbosity** hurts.
- Cache **writes** cost more than cache **reads** (conceptually: paying to populate, saving on reuse).
- Batch discounts apply to eligible bulk work; they do not magically fix a bloated prompt.
- Tool results re-enter the context — chatty tools are a hidden cost amplifier.

**Practical exam heuristic:** If two options both “work,” pick the one that reduces **repeated uncached prefill** or **unnecessary output**, not the one that merely changes adjectives in the system prompt.

### 9.3 Long-context strategy matrix

| Strategy | Best for | Failure mode |
| --- | --- | --- |
| Stuff full docs into prompt | Tiny, stable reference | Cost + distraction |
| Prompt cache stable corpus | Many requests sharing docs | Bust on edits |
| RAG / retrieval slices | Large changing corpora | Bad chunking → wrong answers |
| Summary memories | Long multi-day agents | Summary drift |
| 1M-class window | Truly huge single sessions | Cost; still not an excuse for junk context |

**Teaching point:** A bigger window is not a substitute for **context engineering**. Public prompt-and-context themes on the exam treat “what belongs in context” as a first-class skill adjacent to MSO.

### 9.4 Agent-loop cost control

Agentic sessions multiply turns. Cost drivers:

1. **Tool schema size × turns** (why defer-loading / tool search matter).
2. **Re-reading large files** every turn without caching strategy.
3. **Unbounded loops** without step budgets or stop conditions.
4. **Over-high effort** on every micro-step (classify, then plan, then edit — not all need max effort).

**Pattern:** Use a capable model for **planning** turns; consider a cheaper model for **narrow mechanical** turns *only if* you accept the complexity — many teams keep one model for cache stability. Exams often prefer **one stable model + effort/prompt discipline** over clever multi-model thrashing unless the stem explicitly asks for routing.

### 9.5 When evals disagree with intuition

Suppose online users “like” the smarter model but offline rubric scores are flat and cost is +40%. CCDV-F-style judgment:

- Prefer **measured** quality on a labeled set tied to business outcomes.
- Segment evals (easy/hard). Maybe only hard segment deserves frontier.
- Watch **safety** metrics separately — a fun model that leaks PII is not an upgrade.

### 9.6 Migration playbook (model upgrade)

```text
1. Freeze golden eval set + production traffic shadow sample
2. Pin candidate model in a canary config
3. Compare: success, latency, cost/success, safety flags, cache hit rate
4. If effort defaults differ, tune effort before blaming the model
5. Roll forward with rollback ID ready
6. Update runbooks and CLAUDE.md / service config docs
```

Wrong answer pattern: “Change the alias globally on Friday.”

### 9.7 Interactive vs analytical vs agentic profiles (cheat sheet)

| Profile | Latency | Cost sensitivity | Typical knobs |
| --- | --- | --- | --- |
| Interactive UX | Critical | Medium | Stream, fast tier/effort, tight prompts |
| Analytical bulk | Low | Critical | Batches, cache, right-size model |
| Agentic tools | Medium | Medium-high | Stable tools, cache, step budgets, capable model |
| IDE / Claude Code session | Medium | Medium | Aliases for humans OK; pin in CI headless |

### 9.8 Common stem phrases → intended lever

| Stem phrase | Likely lever |
| --- | --- |
| “overnight,” “backfill,” “evaluate corpus” | Batches |
| “users see typing delay” | Streaming (+ faster path) |
| “bill spiked after adding debug timestamp to system” | Cache bust / prefix hygiene |
| “procurement locked model ID” | Effort / prompt / tools |
| “must stay in eu-region” | Regional host SKU |
| “1% ultra-hard tickets” | Tiered/cascade routing |
| “deterministic CI” | Pin IDs, controlled effort, golden tests |
| “tool list changes every mode switch” | Mode-as-tool / defer_loading, don’t swap schemas |

### 9.9 Anti-patterns gallery

1. **God model for everything** — destroys margin; hides poor prompting.
2. **Premature multi-model mesh** — operational complexity without eval proof.
3. **Cache optionalism** — treating caching as a nice-to-have for agents.
4. **Eval-free swaps** — classic production outage starter pack.
5. **Latency theater** — micro-optimizing TTFT while the agent does 30 needless tool calls.
6. **Context hoarding** — pasting entire wikis “just in case.”
7. **Alias roulette in prod** — unexplained Monday regressions.

### 9.10 Linking MSO to the other four pack chapters

- **Chapter 02:** Agents and prompts determine how much intelligence you *need*.
- **Chapter 03:** Claude Code aliases and MCP tool payload sizes change real cost.
- **Chapter 04:** Evals justify MSO changes; security may forbid certain hosts/models.
- **Chapter 05:** Packaged accelerators should ship with a default MSO policy, not tribal knowledge.

### 9.11 Additional practice vignettes

**Vignette 1.** A retrieval bot caches a 20k-token policy manual. Legal updates the manual hourly. Hits fall. 
**Answer shape:** Hourly corpus changes bust cache — retrieve section-level chunks or version the manual and accept controlled write rates; don’t first switch to Haiku.

**Vignette 2.** An internal agent uses frontier+max effort to format JSON. 
**Answer shape:** Overkill — schema-constrained mid tier; reserve frontier for planning/exceptions.

**Vignette 3.** Product wants “one endpoint” for chat and batch scoring. 
**Answer shape:** One facade OK, but **internally** route sync vs batch and possibly different models/effort.

**Vignette 4.** Shadow traffic shows Opus +5% quality, +90% cost. 
**Answer shape:** Segment; upgrade only slices where +5% matters; keep default tier elsewhere.

### 9.12 Quick reference — public doc themes to re-read before exam

- Choosing a model (capabilities, speed, cost; effort dial)
- Prompt caching (prefix matching, breakpoints, invalidation, TTLs)
- Message Batches (async economics, limitations vs sync)
- Model config in Claude Code (aliases vs names, effort settings)
- Pricing page skim for *relative* input/output/cache/batch relationships (not memorizing every cell)

### 9.13 Five more Q&A

**Q21.** Why might `opusplan`-style aliasing appear in Claude Code but not in your backend API service? 
**A21.** It encodes a **session workflow** (plan with one model, execute with another). Backend services usually need explicit pinned IDs and deliberate routing logic instead of interactive aliases.

**Q22.** Your sync path uses automatic caching; your pre-warm used a placeholder user message with top-level auto cache. Hits fail. Why? 
**A22.** Automatic caching may key off the **last** block (the placeholder). Pre-warm should use an **explicit** breakpoint on the shared system/tools prefix, matching live traffic.

**Q23.** Does reducing `max_tokens` always reduce cost? 
**A23.** It caps **potential** output cost and prevents runaways, but if the model still emits near the cap quality may suffer; pair with “be concise” and schemas. It does not reduce input/cache costs.

**Q24.** A stakeholder asks for the “most powerful model” for a classifier with 99.5% already on Haiku-class. Response? 
**A24.** Keep Haiku-class; invest in data/rules for the 0.5% errors or cascade only those cases — MSO is sufficiency under constraints.

**Q25.** Name two reasons feature parity across Bedrock and Anthropic API matters for MSO. 
**A25.** (1) A caching/batch/tool feature may be unavailable, changing the optimal design; (2) model SKU names differ, so pins and runbooks must be host-specific.

---

## 10. One-page revision sheet

**Remember:** Constraint first → sufficient model → effort → serving path → cache hygiene → pin → measure success-adjusted cost.

**Forget:** Brand worship, streaming-as-savings, batches-for-chat, mid-thread model hopping, alias-only production.

**Drill daily:** 5 stems where you only underline the constraint word (cost / latency / residency / bulk / autonomy) before looking at options.


---

## 11. Primary-study deepening — MSO for Applications & Integration stems

Applications and Integration is **33.1%** of CCDV-F. Many of those items are *MSO decisions dressed as API design*: which model string to pin, whether to stream, whether to batch, how to keep a cache hot, and how to avoid configuration drift. This section trains that overlap so Chapter 01 is a primary study source, not a skim.

### 11.1 Model identity mechanics (exam-stable concepts)

Public Anthropic model docs emphasize:

| Idea | What it means for builders | Exam trigger |
| --- | --- | --- |
| Pinned snapshot ID | A specific model release you can reproduce | “prod regression,” “no surprise upgrades” |
| Convenience alias (legacy pattern) | Pointer that may resolve to a dated ID | “always latest,” “sandbox only” |
| Dateless IDs on newer generations | Still a **pinned snapshot**, not an evergreen moving target | Don’t assume dateless = auto-upgrade |
| Host-specific SKUs | Bedrock / Vertex / Foundry IDs differ from Claude API | Multi-cloud customer; residency |
| Models API introspection | Query `max_input_tokens`, `max_tokens`, capabilities | Capability gating without hardcoding |

**Primary study rule:** Production pins an explicit model identifier in config. Aliases and “latest” marketing names belong in labs and interactive Claude Code sessions unless you deliberately accept drift.

### 11.2 Effort, adaptive thinking, and extended thinking (conceptual map)

Recent public docs distinguish:

1. **Effort** (`output_config.effort` / related surfaces) — dial how hard a model works; the API default is **high**, while Claude Code defaults to **xhigh** (its sweet spot for coding/agentic work).
2. **Adaptive thinking** — model decides when to think (newer Opus/Sonnet/Fable lines).
3. **Extended thinking** (`thinking.type: "enabled"`) — older/explicit thinking control still present on some models (e.g. Haiku 4.5 in public comparison tables).

**Decision discipline:**

```text
Need better answers on SAME model ID?
 → Raise effort / allow more thinking budget first
Need lower latency/cost on SAME quality bar?
 → Lower effort, then consider smaller tier if still green on evals
Need a different capability class (vision edge, autonomy, cheapest QPS)?
 → Change model tier — and treat as a migration (pins, caches, evals)
```

**Cache coupling:** Thinking-config changes invalidate **message-level** cache breakpoints (the tools/system prefix still caches). Keep pre-warm and live thinking/effort matched so every layer of the cache actually gets read.

### 11.3 Serving-path matrix (Integration × MSO)

| Path | Human waiting? | Cost posture | Streaming? | Typical use |
| --- | --- | --- | --- | --- |
| Sync Messages | Yes / near-real-time | Full sync rates | Optional | Chat, agents, online tools |
| Sync + stream | Yes — UX cares about TTFT | Same compute class | Yes | User-facing assistants |
| Sync + prompt cache | Yes, repeated prefixes | Cheaper after warmup | Optional | Agents, apps with stable system+tools |
| Message Batches | No (hours OK) | Discounted async | No | Evals, classification, bulk gen |
| Batch + cache | No | Discounts can **stack**; hits best-effort | No | Nightly jobs with shared prefix |

**Public batch caveats to memorize as judgment, not trivia:**

- No streaming of batch results as they generate.
- Cache hits in batches are **best-effort** (concurrent async workers).
- Pre-warm with `max_tokens: 0` is a **sync** pattern — not something to expect inside batches.
- Some models support extended batch max-output via beta headers (verify live docs); don’t invent headers on the exam — know that sync and batch ceilings can differ.

### 11.4 Prompt caching as an MSO lever (depth)

Caching does not shrink context; it discounts **repeated prefix tokens** and often improves TTFT on hits.

**Prefix order (conceptual):** tools → system → messages, up through the `cache_control` breakpoint.

**TTL themes (public):**

- Default ephemeral lifetime commonly discussed as **~5 minutes**, refreshed on reads.
- Longer TTL (e.g. **1-hour**) available at additional cost when gaps between hits are long.
- Lifetime is measured from the request that writes/reads — long streaming responses consume TTL wall-clock.

**What invalidates or misses caches (study list):**

- Any change to the cached prefix bytes (system text, tool JSON order/names, inserted timestamps).
- Model ID change mid-thread.
- Thinking/effort config mismatch vs the entry you wrote (invalidates message-level breakpoints; tools/system prefix unaffected).
- Automatic caching keyed on a warmup placeholder user turn (use **explicit** breakpoint on shared system/tools for pre-warm).
- Toolset reshuffles used as a “mode switch.”

**MSO exam line:** If the stem says costs spiked after “we added a timestamp to the system prompt for debugging,” the fix is remove volatile fields from the cached prefix — not buy a cheaper model.

### 11.5 Context window strategy as optimization

Public model cards show **1M**-class windows on current Opus/Sonnet/Fable lines and smaller windows on some fast models. Exams care about *strategy*:

| Situation | Prefer |
| --- | --- |
| Multi-hour agent with large repo | Large-window model + compaction + retrieval; don’t dump entire monorepo every turn |
| Stable policies + docs | Cache them; retrieve volatile slices |
| 50+ tools | Tool search / defer loading + cache the stub list |
| Cheap high-QPS extraction | Smaller/faster model + strict schema; large window unused is wasted money only if you fill it |

**Rule:** A bigger window is permission to hold more — not a requirement to send more.

### 11.6 Routing architectures that show up as MSO+Integration

1. **Static route:** One pinned model for a product surface.
2. **Cascade:** Haiku/Sonnet attempt → escalate to Opus on low confidence / validator fail.
3. **Segmented route:** Different models per tenant tier, locale, or risk class.
4. **Plan/execute split:** Stronger model plans; faster model executes tool calls (Claude Code aliases encode this interactively; services implement it explicitly).
5. **Offline/online split:** Batches for analytics; sync for UX.

**Trap:** Cascades without budgets become accidental Opus-everywhere systems. Always cap escalations and log why.

### 11.7 Cost accounting that exam writers like

Think in **unit economics**, not invoice screenshots:

```text
success_adjusted_cost = (input + output + cache_write + cache_read + tool_overhead) / successful_task
```

- Cache **writes** cost more than base input; they pay back on hits.
- Batch discounts apply to eligible tokens; stacking with cache is allowed but probabilistic on hits.
- Agent loops multiply turns — a “cheap” model with 3× turns can lose to a stronger model with 1× turns.
- Output tokens are often priced higher than input — schemas and “be concise” are MSO controls.

### 11.8 Pinning, deprecations, and migrations

Public docs maintain deprecation/migration guidance. Primary study habits:

1. Pin model IDs in environment/config, not in scattered source files.
2. Keep a **compatibility matrix** per host (API vs Bedrock vs Vertex feature flags).
3. Migrate with: golden evals → shadow traffic → canary pin → full cutover → delete old pin.
4. Never change model and prompt and tools in the same release without a bisect plan.

### 11.9 Worked Integration×MSO scenarios (original)

**Scenario A — Support chat, p95 TTFT SLA.** 
Stream; prefer Sonnet- or Haiku-class; moderate/low effort if quality holds; cache stable system+tools; do **not** batch.

**Scenario B — Nightly PII-redacted classification of 2M tickets.** 
Message Batches; smallest sufficient model; shared `cache_control` on instructions+label taxonomy; accept best-effort cache hits; no streaming.

**Scenario C — Coding agent with flaky hard bugs.** 
Pin Opus-class; raise effort before jumping to rarer top-tier models; keep tool schemas stable for cache; compact history; verify with tests (tool), not vibes.

**Scenario D — Multi-cloud enterprise, EU residency.** 
MSO includes **where** inference runs. Pick regional endpoints that satisfy policy even if global routing is faster; confirm feature parity (caching/batch/tools) on that host.

**Scenario E — Cache miss storm after “helpful” refactor.** 
Diff the request prefix; look for tool reordering, dynamic dates, experiment flags interpolated into system text, or effort default changes after SDK upgrade.

### 11.10 Metrics dashboard for MSO owners

| Metric | Why |
| --- | --- |
| Task success rate (eval + online) | Quality gate |
| p50/p95 TTFT and total latency | UX / SLA |
| Cache hit rate | Prefix hygiene |
| Escalation rate (cascades) | Hidden cost |
| $/successful task | True MSO KPI |
| Stop-reason distribution | Truncation / tool loops |
| Model pin version | Drift detection |

### 11.11 Additional Q&A (Q26–Q35)

**Q26.** A product manager wants one model for chat and overnight evals “for consistency.” What’s the MSO rebuttal? 
**A26.** Consistency of *quality bar* can use shared rubrics and overlapping evals; serving paths differ. Chat needs sync/stream; overnight should batch. You may still pin the same model ID on both paths if quality requires it — but don’t force one serving mode.

**Q27.** When do you choose a 1-hour cache TTL over the default short TTL? 
**A27.** When requests that share a prefix arrive with gaps longer than the short TTL (spiky traffic, business-hours tools) and the extra write/storage cost beats repeated full prefill misses.

**Q28.** Your Bedrock deployment lacks a beta tool feature you used on the Claude API. Correct response? 
**A28.** Redesign for the host’s feature matrix (or run that slice on a host that supports it). Don’t assume SKU name parity implies feature parity.

**Q29.** Is streaming cheaper? 
**A29.** Not inherently. It improves perceived latency and allows cancel; token billing still applies to generated output.

**Q30.** Effort default changed after an SDK upgrade and costs rose. First check? 
**A30.** Explicitly set effort in config to the previously validated value; re-run evals; confirm cache hit rates (mismatch can also miss caches).

**Q31.** Large context window but Haiku-class model — when is that still wrong? 
**A31.** When the task needs frontier reasoning/autonomy that evals show Haiku fails — window size doesn’t substitute for capability tier.

**Q32.** What’s the first MSO move when agent loops burn budget? 
**A32.** Add stop conditions, tool budgets, and progress checks; only then consider a stronger model that finishes in fewer turns.

**Q33.** Why pin differently in Claude Code vs a backend service? 
**A33.** Code sessions may use workflow aliases and user `/model` switches; backends need deterministic pins for SLOs, caches, and audit.

**Q34.** Cache write markup vs hit savings — when is caching still correct on day one? 
**A34.** When the same prefix will be reused promptly (agents, multi-turn apps, shared tool schemas). One-off unique prompts may not pay back.

**Q35.** Name three Integration features that change which model you pick. 
**A35.** Need for streaming UX; need for batch economics; need for tool/MCP features only stable on certain model lines — plus host/region constraints.

### 11.12 If exam asks X, think Y (MSO)

| If exam asks… | Think… |
| --- | --- |
| Lower cost, quality OK | Smaller model → lower effort → cache → batch → shorter outputs |
| Lower latency, human waiting | Faster tier / lower effort + stream + cache hits; not batches |
| Same model, better hard cases | Raise effort / thinking; improve tools/prompts before new tier |
| Reproducible prod | Pin model ID; freeze prompt/tool versions; record effort |
| Bulk offline | Message Batches (± cache); no streaming expectation |
| Cost spike after tiny prompt edit | Cache invalidation / prefix volatility |
| Multi-cloud | Host SKU + feature matrix + residency |
| Agent too expensive | Turns × tokens; budgets and stop conditions first |

### 11.13 Primary-study checklist addendum

- [ ] I can explain pin vs alias without relying on a specific marketing name.
- [ ] I can draw sync vs stream vs batch with constraints.
- [ ] I can list five cache invalidators.
- [ ] I can compute success-adjusted cost conceptually.
- [ ] I know effort/thinking must match between warm and live traffic.
- [ ] I treat host/region as an MSO input, not an afterthought.
- [ ] I escalate model tier only after evals fail at current tier+effort.

### 11.14 Glossary addendum

| Term | Definition |
| --- | --- |
| Success-adjusted cost | Total inference cost divided by successful outcomes |
| Cascade route | Try cheaper/faster path, escalate on failure signals |
| Cache breakpoint | Content block marked with `cache_control` ending the cached prefix |
| Best-effort cache hit | Batch/async paths may miss even with identical prefixes |
| Feature matrix | Per-host capability table used for design |
| Dateless pinned ID | Newer ID format that is still a fixed snapshot |
| TTFT | Time to first token (streaming UX) |
| Effort pin | Explicit effort setting stored with the model pin |

---

## 12. LLM fundamentals primer (blueprint skill 5.2%)

*Added 2026-08-23 — the official LLM Fundamentals statement tests “tokens, context windows, sampling, non-determinism, next-token generation … and fundamental prompting techniques (zero-shot, single-shot, multi-shot).” This primer closes that gap.*

### 12.1 How the model actually generates

- **Tokens** are the unit of everything: models read and write tokens (subword chunks — roughly ¾ of an English word each on average), and billing, context windows, and `max_tokens` are all counted in them. Code, JSON, and non-English text tokenize less efficiently than plain English prose.
- **Next-token generation:** the model produces output one token at a time, each choice conditioned on the entire visible context (prompt + everything generated so far). There is no lookahead “plan” outside the tokens; long structured outputs fail at the end because each step compounds.
- **Sampling and non-determinism:** at each step the model has a probability distribution over possible next tokens and *samples* from it — which is why two identical requests can produce different answers. Historically, `temperature` / `top_p` / `top_k` shaped that distribution. **Current state:** on Opus 4.7+, Opus 5, Sonnet 5, and Fable-class models these sampling parameters are **removed** (requests carrying them are rejected); they remain only on older models (4.6-generation, Haiku 4.5).
- **The determinism trap:** `temperature: 0` never guaranteed byte-identical outputs, and on current models the knob doesn’t exist at all. If a stem needs repeatable structure, the correct levers are **schemas/structured outputs, validators, and few-shot anchors** — not sampling settings.

### 12.2 Shot-count terminology (define these exactly)

| Term | Meaning | Use when |
| --- | --- | --- |
| **Zero-shot** | Instructions only; no worked examples | Task is common/easy for the model; keeps prompts short and cacheable |
| **Single-shot (one-shot)** | Exactly one worked example | Format is unusual but consistent; one example anchors shape cheaply |
| **Multi-shot (few-shot)** | Several worked examples | Edge cases, label boundaries, or house style matter; pick diverse, high-signal examples (edge cases > happy paths) |

**MSO angle:** examples live in the prompt, so every shot costs tokens on every call — keep the example block **stable** inside the cached prefix, and prune examples that stopped earning their tokens.

### 12.3 Self-check Q&A (Q36–Q40)

**Q36.** Why do two identical API calls return different wording? 
**A36.** Outputs are **sampled** from a next-token probability distribution — non-determinism is inherent, not a bug.

**Q37.** A teammate sets `temperature: 0` on a current Opus-class model “for deterministic JSON.” Two problems? 
**A37.** Sampling params are removed on current models (the request errors), and even historically temp 0 didn’t guarantee identical output. Use schemas + validation.

**Q38.** Classification prompt fails on ambiguous boundary cases only. Zero-shot fix that isn’t a model upgrade? 
**A38.** Go **multi-shot**: add a few examples that sit exactly on the confusing boundaries.

**Q39.** Why can a 2,000-word English prompt cost fewer tokens than 500 lines of JSON? 
**A39.** Tokenization efficiency differs — prose compresses into tokens better than dense structured text.

**Q40.** Where should few-shot examples sit for a high-volume cached endpoint? 
**A40.** In the stable cached prefix (with system/tools), unchanged between requests — editing an example busts the cache.

---

## 13. Fast mode (blueprint-named model option — know it exists and when)

*Added 2026-08-23. The official LLM Fundamentals statement names **fast mode** alongside extended thinking, adaptive thinking, and effort; this pack previously omitted it. Facts below cached 2026-08 — a research-preview feature, so verify current docs before exam day.*

**What it is:** the **same model** running at up to ~2.5× higher output tokens/second, at **premium pricing** (e.g. Opus 5 fast at roughly 2× the standard Opus 5 rate per MTok). It is a *speed/price* trade on identical intelligence — not a smaller model, not a quality change.

**Mechanics (conceptual):** available on **Claude Opus 5 and Opus 4.8 only**, as a research preview. Requests opt in per call: beta messages endpoint + a fast-mode beta flag + a top-level `speed: "fast"` parameter; the response usage reports which speed served it. Fast mode has its **own rate limit**, separate from standard traffic.

**Where it is NOT available (high-signal exam facts):**

- Not on Bedrock / Vertex / Foundry — Claude API (and managed-agent sessions) only.
- Not with the **Batch API** (batches are the latency-tolerant path; fast mode is the latency-critical one).
- Not combined with priority-capacity tiers.

**Operational judgment:**

```text
Is a human waiting AND output length dominates latency AND budget allows premium?
 YES → fast mode candidate (same model, ~2.5× output speed, ~2× price)
 NO → cheaper levers first: streaming (perceived latency), cache hits (TTFT),
 lower effort, faster tier
On fast-mode 429: retry after the indicated delay, or fall back to standard speed —
 and remember a speed switch invalidates the prompt cache entry.
```

**When it beats a tier downgrade:** the task needs frontier quality (downgrade fails evals) but users feel the generation time — e.g. long code reviews or drafted documents streamed live to a waiting expert.

### 13.1 Self-check Q&A (Q41–Q45)

**Q41.** Fast mode vs switching Haiku-class — what’s the core difference? 
**A41.** Fast mode keeps the **same model’s quality** at higher output speed and higher price; a tier switch trades quality for speed/cost.

**Q42.** Can you run your overnight 200k-document batch in fast mode to “finish sooner”? 
**A42.** No — fast mode isn’t available with the Batch API, and batch work is latency-tolerant by definition; it’s the wrong lever.

**Q43.** A Bedrock-only enterprise asks for fast mode. Response? 
**A43.** Not available there (Claude API research preview, Opus 5 / Opus 4.8 only) — offer streaming, caching, effort tuning, or tier choices on their host instead.

**Q44.** Fast-mode requests start returning 429 while standard Opus traffic is fine. Why is that possible? 
**A44.** Fast mode has its **own separate rate limit**; retry after the indicated delay or drop to standard speed (accepting the cache invalidation a speed switch causes).

**Q45.** Stem says “reduce the bill” — is fast mode ever the answer? 
**A45.** No. Fast mode raises unit price for speed. Cost stems want smaller models, lower effort, caching, batches, shorter outputs.

---

## 14. Current model lineup card (dated snapshot — verify before exam day)

*Cached **2026-08**. This pack’s doctrine stays “criteria over SKUs,” but the official Model Selection statement asks about “Opus vs. Sonnet vs. Haiku use cases, **adaptive thinking support**” — so anchor the concrete lineup once, then reason with the decision trees. Model cards change; re-check the public docs the week you sit.*

| Tier | Model | Context | Thinking | Effort levels | Relative price (in/out per MTok) |
| --- | --- | --- | --- | --- | --- |
| Top (above Opus) | Claude Fable 5 | 1M | **Always on** (adaptive; cannot be disabled) | low → xhigh, max | ~$10 / $50 |
| Frontier | Claude Opus 5 | 1M | Adaptive **on by default** (omit = thinking) | low → xhigh, max | ~$5 / $25 |
| Frontier | Claude Opus 4.8 / 4.7 | 1M | Adaptive available — **opt in** (omit = no thinking) | low → xhigh, max | ~$5 / $25 |
| Frontier (older) | Claude Opus 4.6 | 1M | Adaptive recommended; legacy `budget_tokens` deprecated | low/medium/high/max (no xhigh) | ~$5 / $25 |
| Balanced | Claude Sonnet 5 | 1M | Adaptive (omit = adaptive) | low → xhigh, max | ~$3 / $15 |
| Balanced (older) | Claude Sonnet 4.6 | 1M | Adaptive recommended; `budget_tokens` deprecated | low/medium/high/max | ~$3 / $15 |
| Fast/cheap | Claude Haiku 4.5 | **200K** | **Extended thinking** style (`type:"enabled"` + `budget_tokens`) | No effort parameter | ~$1 / $5 |

**Card-reading rules (the part exams test):**

1. **Adaptive thinking support** = the 4.6-and-later Opus/Sonnet lines plus Fable-class; on the newest tier thinking is always-on, on Opus 5 it’s default-on, on 4.7/4.8 you must opt in, and Haiku 4.5 still uses the older fixed-budget extended-thinking style.
2. The fixed **`budget_tokens`** thinking budget is a **legacy** control: deprecated on the 4.6 generation and removed on newer lines — “adaptive thinking + effort” is the current answer when a stem mentions thinking budgets.
3. **`xhigh` effort** exists from Opus 4.7 onward (and Sonnet 5 / Fable-class) — it’s the default in Claude Code; 4.6-generation models cap at high/max without xhigh; Haiku 4.5 has no effort dial.
4. Context: 1M-class on frontier/balanced lines; Haiku-class is smaller (200K) — long-repo stems eliminate Haiku on window alone.
5. Prices are **relative anchors**, not memorization targets — reason in ratios: Fable ≈ 2× Opus; Opus ≈ 1.7× Sonnet; Sonnet ≈ 3× Haiku (input). Never quote absolute prices in prod docs without checking the live pricing page.

---

## 15. Closing — Chapter 01 as primary study

MSO is 16.8% on its own and the decision engine inside much of Applications and Integration. If you can pick **sufficient intelligence**, **serving path**, and **cache/pin hygiene** under a stated constraint, you are practicing the highest-frequency judgment pattern on CCDV-F.
