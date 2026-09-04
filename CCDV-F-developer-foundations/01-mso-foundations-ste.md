---
title: Chapter 01 — MSO Foundations (Model Selection & Optimization) — Simplified Technical English
pack: CCDV-F Developer Foundations
disclaimer: Original study notes — independent and not official course content
approx_length: STE edition (ASD-STE100) — primary study
updated: 2026-08-29
---

# Chapter 01 — MSO Foundations
## Model Selection & Optimization fundamentals (Simplified Technical English)

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, MCP, prompting, caching, effort, p95) are exceptions and stay as written. Model IDs and prices change. Learn the decision rules. Check the current model cards before the exam.

**Maps primarily to:** Model Selection and Optimization (**16.8%**). 
**Also covers:** Applications and Integration (when the question says "pick API path for cost/latency").

**How to study:** Learn the three-axis tradeoff (capability × latency × cost). Then practice this: read a question, find the one hard constraint, and select the one answer that respects it. Most MSO questions are constraint problems. They do not ask "which model is the most capable."

---

## 1. Overview

MSO is the skill of **matching work to the correct intelligence budget**. On Claude, you express this budget through:

1. **Model family / tier** (frontier vs mid vs fast/cheap).
2. **Effort / thinking configuration** (how hard one model works).
3. **Context and output limits** (window size, max output tokens).
4. **Serving path** (interactive Messages vs streaming vs Message Batches vs cached prefixes).
5. **Operational settings** (pinning versions, aliases vs full IDs, region/cloud host).

Public docs describe model choice as a balance of capabilities, speed, and cost. Recent Opus/Sonnet-class models add an **effort** parameter. Effort trades intelligence for latency and cost *within* one model. Often you change effort first. This is better than a switch to another tier.

**Exam rule:** Select the **smallest model and effort that is sufficient**. The model must pass your quality gates on your evals. When the question emphasizes cost or p95 latency, an answer that selects too much intelligence is a common wrong answer.

---

## 2. Key map (flashcard)

| Concept | One-line meaning | Exam trigger words |
| --- | --- | --- |
| Model tier | Capability class (frontier / balanced / fast) | "complex refactor," "classify at scale," "chat UI" |
| Effort | Test-time compute within a model | "same model, better answers," "lower latency same model" |
| Context window | How much prompt+history fits | "long repo," "1M context," "truncate" |
| Prompt caching | Prefix reuse for cheaper/faster repeats | "stable system+tools," "multi-turn agent" |
| Message Batches | Async bulk at discount | "overnight eval," "latency tolerant" |
| Streaming | Token-by-token UX / early cancel | "TTR," "user-facing chat" |
| Pinning | Freeze model string for reproducibility | "prod regression," "version drift" |
| Alias | Moving pointer to "latest sonnet/opus/…" | "always newest," "dev sandbox" |

---

## 3. Deep notes

### 3.1 The three-axis decision frame

Every production choice balances three axes:

```text
 CAPABILITY
 /\
 / \
 / ? \
 /________\
 COST LATENCY
```

- Move toward **capability** when the cost of failure is high (security review, multi-hour agent, hard coding).
- Move toward **latency** when humans wait for the first token or the full answer (support chat, IDE inline).
- Move toward **cost** when volume dominates (classification, extraction, nightly batches).

**Exam rule:** Find the vertex that the question limits most. Then remove the options that ignore it.

### 3.2 Model family mental model (stable even as names change)

Public choosing-a-model guidance (2026) organizes roughly as:

| Workload pattern | Prefer (conceptual) | Why |
| --- | --- | --- |
| Multihour agents, deep reasoning, large refactors, vision-heavy enterprise work | Frontier Opus-class (e.g. Claude Opus 5 family in public docs) | Highest autonomy and reasoning headroom |
| Daily coding, data analysis, content, solid agentic tool use | Sonnet-class | Best default balance for most apps |
| High-QPS routine classification, simple transforms, cheap loops | Haiku-class | Speed and unit economics |
| Longest / hardest frontier agent runs (where available) | Fable-class tier (a model family above Opus. Claude Code's `best` alias resolves interactively to the top tier) | Explicitly for hardest long-running tasks |

**Do not memorize marketing slogans.** Memorize **fit**:

- **Tool-heavy coding agents** → mid-to-frontier + appropriate effort.
- **Structured extraction with gold schema** → often mid tier + strict schema/tool. Escalate only if evals fail.
- **User-facing low-latency assistants** → faster tier, or the same tier at lower effort. Stream.

### 3.3 Effort as the intra-model setting

Public docs describe **effort** on recent Opus/Sonnet models as a trade: intelligence for latency and cost. Defaults are often **high**. Higher settings (`xhigh`, `max`, where offered) help demanding agentic and coding work. Lower settings help routine tasks.

**Operational pattern:**

1. Pin a model.
2. Start at the **documented default** for that model.
3. Raise effort only when evals show systematic under-reasoning.
4. Lower effort when the p95 latency/cost budget is exceeded and quality still passes.

**Common exam error:** The question says "keep the same model ID for cache stability / contract." Then the answer is usually **change effort**, not change model.

**Cache note:** A change to the thinking configuration invalidates **message-level** cache breakpoints. The tools/system prefix stays cacheable. Keep effort/thinking the same between pre-warm and live traffic. A pre-warm with a different thinking configuration can leave your message-level cache entries unread.

### 3.4 Context windows and output limits

Treat context as a **budget**. Do not treat it as unbounded storage:

| Pressure | Symptom | MSO response |
| --- | --- | --- |
| History too long | Truncation, lost instructions | Compact summaries. Move durable rules out of chat |
| Huge tool schemas | Wasted prefix tokens | Tool search / defer loading. Fewer tools |
| Large docs every turn | Rapid cost increase | Cache stable docs. Retrieve slices |
| Need long outputs | Cutoff mid-JSON | Raise `max_tokens`. Structured streaming. Chunk tasks |

Public materials show large windows (some models have **1M**-class windows) and high max output on frontier models. Exams test one judgment. Do you know when to request a large window (long sessions, large repositories)? Do you know when retrieval and caching cost less than a full window?

### 3.5 Cost controls that are still "MSO"

MSO is not only "pick Opus vs Sonnet." Cost-aware answers often combine:

1. **Smaller model** if quality holds.
2. **Lower effort** if quality holds.
3. **Prompt caching** for stable prefixes (system, tools, reference docs).
4. **Message Batches** when latency can wait (public docs: async bulk, discounted vs sync. Caching combines with the discount, but hits are best-effort).
5. **Shorter outputs** (ask for concise. Schema-constrained).
6. **Do not rewrite the cached prefix** (tool order changes, timestamps in system prompt, mid-session model change).

**Batch vs realtime decision tree:**

```text
Is a human waiting on this response in < few seconds?
 YES → Sync Messages (+ stream if UX needs tokens early)
 NO → Are there many independent requests and hours-ok SLA?
 YES → Message Batches (+ shared cache_control, prefer longer TTL if available)
 NO → Sync with caching; consider queue workers
```

Public batch caveats: no streaming of results, no sync-only speed modes, no `max_tokens: 0` pre-warm inside batches, and stateful thread params generally do not apply.

### 3.6 Latency controls

| Lever | Effect | Side effect |
| --- | --- | --- |
| Faster model tier | Lower TTFT / total time | Possible quality drop |
| Lower effort | Faster | Possible quality drop |
| Streaming | Better *perceived* latency | Same total compute roughly |
| Prompt cache hit | Faster prefill | Requires stable prefix |
| Smaller prompts | Faster | May need better retrieval |
| Parallel tool calls (where supported) | Less wall-clock time | More concurrency complexity |

**Exam nuance:** Streaming fixes the **UX wait**. It does not fix unit cost. If the question is about cost, streaming alone is rarely the answer.

### 3.7 Quality controls (when the output under-performs)

Escalate in this order, unless the question forbids it:

1. Fix **prompt/context** (missing constraints, bad examples, noisy tools).
2. Add **tools / MCP**, so the model can fetch ground truth.
3. Raise **effort**.
4. Escalate the **model tier**.
5. Change the **architecture** (multi-step agent, verifier model, human gate).

A direct switch to the largest model is a frequent wrong answer.

### 3.8 Version pinning vs aliases

| Approach | Use when | Risk |
| --- | --- | --- |
| Full dated / explicit model ID | Production, evals, compliance | You must plan upgrades |
| Alias (`sonnet`, `opus`, `haiku`, `best`, …) | Dev, "always current" sandboxes | Silent behavior drift |
| Claude Code aliases (`opusplan`, `sonnet[1m]`, etc.) | Session workflow preferences | Still pin for CI reproducibility |

**Production checklist:** Pin the model string in config. Record it in eval reports. Run a canary before you upgrade the fleet. Do not change the model mid-conversation when you rely on the prompt cache.

### 3.9 Routing patterns (architecture and MSO)

**A. Static route:** One model serves the product surface. Operations stay simple. Optimization stays limited.

**B. Tiered route:** A classifier or rules send easy traffic to Haiku-class and hard traffic to Sonnet/Opus-class.

**C. Cascade / escalate:** Try the cheap path first. If confidence is low or schema validation fails, retry on the expensive path.

**D. Dual-model verify:** A mid-tier model generates. A same-tier or higher-tier model checks high-risk outputs.

Exam questions that mention "90% trivial tickets, 10% novel incidents" almost always require tiered or cascade routing. They rarely want "everything on frontier."

### 3.10 Cloud hosting is still MSO-adjacent

Public topics include Bedrock and Vertex/Agent Platform. These selection differences appear in scenarios:

- The **model ID format** differs by host (Anthropic API name vs Bedrock inference profile ARN vs Vertex version name).
- **Feature lag / parity** — confirm tool, caching, batch, and beta availability on that host.
- **Data residency / IAM** — policy may force a region, even when another region has a newer SKU.

If the hard constraint is **residency**, the correct model choice is "the approved regional SKU." It is not the newest global ID.

### 3.11 Worked mini-scenarios

**Scenario A — Support chatbot, p95 < 2s, moderate quality.** 
Select a fast or mid tier. Set effort at default or lower. Stream. Cache the system+tool prefix. Do not select frontier with max effort.

**Scenario B — Overnight evaluation of 50k prompts.** 
Use Message Batches. Use identical `cache_control` blocks. Use a longer cache TTL if offered. Use a mid tier, unless evals need frontier.

**Scenario C — Autonomous multi-hour coding agent.** 
Use a frontier or strong Sonnet/Opus-class model with elevated effort. Use large context as needed. Cache the tools. Do not change the model mid-run.

**Scenario D — Cost spike after "adding a timestamp to the system prompt."** 
The timestamp invalidated the cache. Fix prefix stability. Do not first change to a cheaper model.

### 3.12 Metrics that prove an MSO decision

Track these together. If you track one alone, you optimize the wrong axis:

| Metric | Asks |
| --- | --- |
| Task success / rubric score | Did quality hold? |
| Cost per successful task | Not cost per raw call |
| p50/p95 latency | Human-waiting paths |
| Cache hit rate | Prefix engineering health |
| Escalation rate | Is routing calibrated? |
| Safety incident rate | Did the cheaper path skip filters? |

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
| Cache misses after model A/B test in one thread | Do not switch mid-thread | "Just try Opus on turn 12" |
| Easy tasks too expensive | Lower effort or smaller model | Keep frontier for everything |
| Sharp quality drops on the hard 5% | Route the hard 5% up | Upgrade 100% of traffic |

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

1. **You select the most capable model** when the question stresses cost or latency.
2. **You treat streaming as a cost saver** (it is not).
3. **You use Batches for a chat UI** (wrong latency class).
4. **You change tools or model mid-conversation** while you rely on the cache.
5. **You put volatile data (timestamps, per-user secrets) in the cached system prefix.**
6. **You use aliases in production** without a pin + canary plan.
7. **You ignore host/region constraints** (Bedrock/Vertex residency).
8. **You optimize cost per token** instead of **cost per successful task**.
9. **You raise max_tokens** to "fix" bad structure — prefer schemas and chunking.
10. **You treat effort and model tier as the same setting** — they are layers.

---

## 6. Self-check Q&A (20)

**Q1.** A product needs sub-second perceived replies for FAQ chat. Quality is "good enough" on a mid-tier model. What is the best first optimization? 
**A1.** Keep the mid-tier model. Enable **streaming**. Make prompt **cache** hits happen on the stable system/tool prefix. Consider **lower effort** if quality evals still pass. Do not change to frontier.

**Q2.** You must score 200k documents by tomorrow morning. No user waits. Which API pattern is best? 
**A2.** **Message Batches**, optionally with shared cached context and a longer cache TTL when available.

**Q3.** The cache hit rate collapsed after a deploy. The diff shows tool definitions in random order each request. What is the root cause? 
**A3.** **Prefix mismatch** — non-deterministic tool order breaks prompt caching. Serialize the tools deterministically.

**Q4.** Evals show the model under-plans on hard coding tasks. The model ID must stay fixed for procurement. Which control do you use? 
**A4.** Increase **effort** (and improve prompts/tools). Do not change the model when the constraint forbids it.

**Q5.** Why does a switch from Sonnet to Opus mid-thread hurt an agent session, beyond price? 
**A5.** Caches are **model-scoped**. The switch invalidates the cached prefix and can destabilize behavior.

**Q6.** When is Haiku-class the wrong answer, despite high QPS? 
**A6.** When the question requires deep multi-step autonomy, subtle policy judgment, or high-cost-of-error reasoning that evals show small models fail.

**Q7.** The alias `sonnet` in CI started to fail after a silent upgrade. How do you prevent this? 
**A7.** **Pin** explicit model IDs in CI/prod. Use aliases only where drift is acceptable.

**Q8.** Batch jobs show low cache hits despite `cache_control`. What do people usually miss here? 
**A8.** Non-identical prefixes across items, a cache TTL too short for async processing, or an expectation of guaranteed hits (docs: **best-effort** in batches).

**Q9.** Does batch processing require streaming? 
**A9.** No. You retrieve batch results as completed results/files, not as live token streams.

**Q10.** A 1M-context option exists. Should every app enable it? 
**A10.** No. Use it when sessions/repos truly need it. Otherwise retrieval + caching is often cheaper and clearer.

**Q11.** Cascade routing: which signal triggers escalation? 
**A11.** Validation failure, low self-confidence score, policy risk flags, or a rubric failure — not "random 10%."

**Q12.** Cost per call dropped, but user complaints rose. Which metric was missing? 
**A12.** **Task success / cost per successful task** (and possibly safety incidents).

**Q13.** Why does pre-warm with `max_tokens: 0` exist? 
**A13.** It populates the prompt cache without an answer (sync path). It is not available inside batches, per public batch limitations.

**Q14.** The thinking/effort of the pre-warm differs from live traffic. What is the symptom? 
**A14.** Message-level cache entries written under one thinking config do not match live traffic (the tools/system prefix can still hit). The result is partial misses that look like "caching broken." Align the configs.

**Q15.** A Bedrock-only customer sits in a locked region. The newest Anthropic API model is not listed there. What is the correct action? 
**A15.** Select the **approved regional** model/profile that meets policy. Plan parity tests. Do not make cross-region calls against policy.

**Q16.** The question says "make it cheaper." The agent uses 40 tools every turn. What is a strong MSO move? 
**A16.** Keep the tool list stable. Use **defer loading / tool search** patterns, so the full schemas are not always expanded. Cache the stable stub prefix.

**Q17.** When is the dual-model verify pattern justified? 
**A17.** High-stakes outputs (legal/medical summaries, prod migrations), where the verifier costs less than the error.

**Q18.** p95 latency is bad. Cache hit rate is 5%. What do you investigate first? 
**A18.** Prefix stability (system/tools ordering, volatile system text). Do not first increase capacity for the same faulty setup.

**Q19.** Why can a raise in effort increase cost more than expected? 
**A19.** More tokens, tool loops, and thinking raise the cost with test-time compute — not only the list price.

**Q20.** Map this chapter to the exam weight. 
**A20.** Core of **Model Selection and Optimization 16.8%**, plus Integration items that are really cost/latency routing questions.

---

## 7. Checklist

- [ ] I can explain capability vs latency vs cost without naming a specific SKU.
- [ ] I know when to change **effort** vs **model tier**.
- [ ] I can select sync vs stream vs batch from an SLA sentence.
- [ ] I can list three ways to accidentally invalidate a prompt cache.
- [ ] I pin models in prod and I know the alias risks.
- [ ] I track success-rate-adjusted cost, not token spend without outcomes.
- [ ] I have a cascade/tiered routing plan for mixed workloads.
- [ ] I remember the cloud host ID/parity/residency caveats.
- [ ] I escalate quality in this order: prompt → tools → effort → model → architecture.
- [ ] I re-check the current public model cards before exam day.

---

## 8. Glossary

| Term | Definition |
| --- | --- |
| MSO | Model Selection & Optimization — choosing and tuning intelligence spend |
| Effort | Intra-model setting for test-time compute / thoroughness |
| TTFT | Time to first token — key streaming UX metric |
| Prefix cache | Cached leading portion of a prompt (system/tools/shared docs) |
| Cache invalidation | Any change that prevents prefix match |
| Message Batches | Async bulk Messages API processing at discounted economics |
| Model pin | Fixed model identifier for reproducible prod/evals |
| Model alias | Indirection that resolves to a moving "latest" model |
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
| Others | Small coverage |

---

## 9. Expanded deep dive — production MSO playbooks

### 9.1 Building an MSO policy document

A team that selects models without a policy in pull requests fails Integration-style scenarios. Write a short **MSO policy**:

1. The **default model** for interactive product surfaces.
2. The **batch model** for offline scoring.
3. The **agent model** for tool-loop coding / ops agents.
4. The **escalation model** for failures / high severity.
5. The **effort defaults** per surface.
6. The **pinning rule** (full IDs in prod. Aliases only in personal sandboxes).
7. The **eval gate** — no fleet model change without a golden-set delta report.
8. The **cache contract** — ordered tools, frozen system sections, owners for prefix changes.

Exam answers that look like **governance** score. "Update the pinned config + canary + eval report" is a good answer pattern. "Swap to Opus in the chat UI" is not.

### 9.2 Token economics without memorizing price tables

You do not need exact dollar charts. You need **relative** reasoning:

- On public Anthropic pricing pages, output tokens usually cost more than input tokens. So **verbosity** increases cost.
- Cache **writes** cost more than cache **reads** (you pay to fill the cache, and you save on reuse).
- Batch discounts apply to eligible bulk work. They do not fix a prompt that is too large.
- Tool results re-enter the context. Tools that return many tokens are a hidden cost amplifier.

**Exam heuristic:** If two options both "work," select the one that reduces **repeated uncached prefill** or **unnecessary output**. Do not select the one that only changes adjectives in the system prompt.

### 9.3 Long-context strategy matrix

| Strategy | Best for | Failure mode |
| --- | --- | --- |
| Put full documents in the prompt | Tiny, stable reference | Cost + distraction |
| Prompt cache stable corpus | Many requests sharing docs | Invalidation on edits |
| RAG / retrieval slices | Large changing corpora | Bad chunking → wrong answers |
| Summary memories | Long multi-day agents | Summary drift |
| 1M-class window | Truly huge single sessions | Cost. Do not use it for low-value context |

**Teaching point:** A bigger window does not replace **context engineering**. Public exam themes treat "what belongs in context" as a core skill next to MSO.

### 9.4 Agent-loop cost control

Agentic sessions multiply turns. The cost drivers:

1. **Tool schema size × turns** (why defer-loading / tool search matter).
2. **Re-reading large files** every turn, without a caching strategy.
3. **Unbounded loops** without step budgets or stop conditions.
4. **Too-high effort** on every micro-step (classify, then plan, then edit — not all need max effort).

**Pattern:** Use a capable model for **planning** turns. Consider a cheaper model for **narrow mechanical** turns only if you accept the complexity. Many teams keep one model for cache stability. Exam answers usually prefer **one stable model + effort/prompt discipline**, not clever multi-model switching — unless the question explicitly asks for routing.

### 9.5 When evals disagree with intuition

Online users "like" the smarter model. Offline rubric scores are flat. Cost is +40%. The CCDV-F judgment:

- Prefer **measured** quality on a labeled set tied to business outcomes.
- Segment the evals (easy/hard). Possibly only the hard segment needs frontier.
- Watch **safety** metrics separately. A model that users like but that leaks PII is not an upgrade.

### 9.6 Migration playbook (model upgrade)

```text
1. Freeze golden eval set + production traffic shadow sample
2. Pin candidate model in a canary config
3. Compare: success, latency, cost/success, safety flags, cache hit rate
4. If effort defaults differ, tune effort before blaming the model
5. Roll forward with rollback ID ready
6. Update runbooks and CLAUDE.md / service config docs
```

Wrong answer pattern: "Change the alias globally on Friday."

### 9.7 Interactive vs analytical vs agentic profiles (cheat sheet)

| Profile | Latency | Cost sensitivity | Typical settings |
| --- | --- | --- | --- |
| Interactive UX | Critical | Medium | Stream, fast tier/effort, tight prompts |
| Analytical bulk | Low | Critical | Batches, cache, right-size model |
| Agentic tools | Medium | Medium-high | Stable tools, cache, step budgets, capable model |
| IDE / Claude Code session | Medium | Medium | Aliases for humans are acceptable. Pin in CI headless |

### 9.8 Common stem phrases → intended control

| Stem phrase | Likely lever |
| --- | --- |
| "overnight," "backfill," "evaluate corpus" | Batches |
| "users see typing delay" | Streaming (+ faster path) |
| "bill spiked after adding debug timestamp to system" | Cache invalidation / prefix stability |
| "procurement locked model ID" | Effort / prompt / tools |
| "must stay in eu-region" | Regional host SKU |
| "1% ultra-hard tickets" | Tiered/cascade routing |
| "deterministic CI" | Pin IDs, controlled effort, golden tests |
| "tool list changes every mode switch" | Mode-as-tool / defer_loading, do not swap schemas |

### 9.9 Anti-patterns list

1. **One maximum-capability model for everything** — destroys margin. Hides poor prompting.
2. **Premature multi-model mesh** — operational complexity without eval proof.
3. **Cache optionalism** — treating caching as optional for agents.
4. **Eval-free swaps** — a common start of production outages.
5. **Cosmetic latency work** — micro-optimizing TTFT while the agent makes 30 needless tool calls.
6. **Unneeded context** — pasting entire wikis without a need.
7. **Unmanaged aliases in prod** — unexplained regressions after upgrades.

### 9.10 Linking MSO to the other four pack chapters

- **Chapter 02:** Agents and prompts determine how much intelligence you *need*.
- **Chapter 03:** Claude Code aliases and MCP tool payload sizes change real cost.
- **Chapter 04:** Evals justify MSO changes. Security may forbid certain hosts/models.
- **Chapter 05:** Packaged accelerators should include a default MSO policy, not undocumented knowledge.

### 9.11 Additional practice vignettes

**Vignette 1.** A retrieval bot caches a 20k-token policy manual. The legal team updates the manual hourly. Hits fall. 
**Answer pattern:** Hourly corpus changes invalidate the cache. Retrieve section-level chunks, or version the manual and accept controlled write rates. Do not first switch to Haiku.

**Vignette 2.** An internal agent uses frontier+max effort to format JSON. 
**Answer pattern:** More capability than necessary. Use a schema-constrained mid tier. Reserve frontier for planning/exceptions.

**Vignette 3.** Product wants "one endpoint" for chat and batch scoring. 
**Answer shape:** One facade is acceptable, but **inside** you route sync vs batch, and possibly different models/effort.

**Vignette 4.** Shadow traffic shows Opus +5% quality, +90% cost. 
**Answer shape:** Segment. Upgrade only the slices where +5% matters. Keep the default tier elsewhere.

### 9.12 Quick reference — public doc themes to re-read before exam

- Choosing a model (capabilities, speed, cost. Effort setting)
- Prompt caching (prefix matching, breakpoints, invalidation, TTLs)
- Message Batches (async economics, limitations vs sync)
- Model config in Claude Code (aliases vs names, effort settings)
- Pricing page review of *relative* input/output/cache/batch relationships (do not memorize every cell)

### 9.13 Five more Q&A

**Q21.** Why does `opusplan`-style aliasing appear in Claude Code but not in your backend API service? 
**A21.** It encodes a **session workflow** (plan with one model, execute with another). Backend services usually need explicit pinned IDs and deliberate routing logic, not interactive aliases.

**Q22.** Your sync path uses automatic caching. Your pre-warm used a placeholder user message with top-level auto cache. Hits fail. Why? 
**A22.** Automatic caching may use the **last** block (the placeholder) for the cache key. The pre-warm should set an **explicit** breakpoint on the shared system/tools prefix, so it matches live traffic.

**Q23.** Does reducing `max_tokens` always reduce cost? 
**A23.** No. It caps **potential** output cost and prevents runaways. But if the model still emits near the cap, quality may suffer. Pair it with "be concise" and schemas. It does not reduce input/cache costs.

**Q24.** A stakeholder asks for the "most powerful model" for a classifier that already scores 99.5% on Haiku-class. Your response? 
**A24.** Keep Haiku-class. Invest in data/rules for the 0.5% errors, or cascade only those cases. MSO is sufficiency under constraints.

**Q25.** Name two reasons feature parity across Bedrock and Anthropic API matters for MSO. 
**A25.** (1) A caching/batch/tool feature may be unavailable, and that changes the optimal design. (2) Model SKU names differ, so pins and runbooks must be host-specific.

---

## 10. One-page revision sheet

**Remember, in order:** Constraint first → sufficient model → effort → serving path → cache discipline → pin → measure success-adjusted cost.

**Forget:** Brand loyalty. Streaming-as-savings. Batches-for-chat. Mid-thread model switching. Alias-only production.

**Daily drill:** Take 5 exam stems. Underline only the constraint word (cost / latency / residency / bulk / autonomy). Then look at the options.

---

## 11. Primary-study deepening — MSO for Applications & Integration stems

Applications and Integration is **33.1%** of CCDV-F. Many of those items are MSO decisions inside API-design questions. This model string to pin, whether to stream, whether to batch, how to keep the cache populated, and how to avoid configuration drift. This section covers that overlap, so Chapter 01 is a primary study source.

### 11.1 Model identity mechanics (exam-stable concepts)

Public Anthropic model docs emphasize:

| Idea | What it means for builders | Exam trigger |
| --- | --- | --- |
| Pinned snapshot ID | A specific model release you can reproduce | "prod regression," "no surprise upgrades" |
| Convenience alias (legacy pattern) | Pointer that may resolve to a dated ID | "always latest," "sandbox only" |
| Dateless IDs on newer generations | Still a **pinned snapshot**, not a permanently moving target | Do not assume dateless = auto-upgrade |
| Host-specific SKUs | Bedrock / Vertex / Foundry IDs differ from Claude API | Multi-cloud customer. Residency |
| Models API introspection | Query `max_input_tokens`, `max_tokens`, capabilities | Capability gating without hardcoding |

**Primary study rule:** Production pins an explicit model identifier in config. Aliases and "latest" marketing names belong in labs and interactive Claude Code sessions — unless you deliberately accept drift.

### 11.2 Effort, adaptive thinking, and extended thinking (conceptual map)

Recent public docs distinguish:

1. **Effort** (`output_config.effort` / related surfaces) — a setting for how hard a model works. The API default is **high**. Claude Code defaults to **xhigh** (it gives the best results for coding/agentic work).
2. **Adaptive thinking** — the model decides when to think (newer Opus/Sonnet/Fable lines).
3. **Extended thinking** (`thinking.type: "enabled"`) — the older, explicit thinking control. Some models still have it (e.g. Haiku 4.5 in public comparison tables).

**Decision discipline:**

```text
Need better answers on SAME model ID?
 → Raise effort / allow more thinking budget first
Need lower latency/cost on SAME quality bar?
 → Lower effort, then consider smaller tier if still green on evals
Need a different capability class (vision edge, autonomy, cheapest QPS)?
 → Change model tier — and treat as a migration (pins, caches, evals)
```

**Cache coupling:** A change to the thinking config invalidates **message-level** cache breakpoints (the tools/system prefix still caches). Keep pre-warm and live thinking/effort matched, so every cache layer gets read.

### 11.3 Serving-path matrix (Integration × MSO)

| Path | Human waiting? | Cost posture | Streaming? | Typical use |
| --- | --- | --- | --- | --- |
| Sync Messages | Yes / near-real-time | Full sync rates | Optional | Chat, agents, online tools |
| Sync + stream | Yes — UX cares about TTFT | Same compute class | Yes | User-facing assistants |
| Sync + prompt cache | Yes, repeated prefixes | Cheaper after warmup | Optional | Agents, apps with stable system+tools |
| Message Batches | No (hours OK) | Discounted async | No | Evals, classification, bulk gen |
| Batch + cache | No | Discounts can **combine**. Hits best-effort | No | Nightly jobs with shared prefix |

**Public batch caveats — learn them as judgment, not trivia:**

- No streaming of batch results while they generate.
- Cache hits in batches are **best-effort** (concurrent async workers).
- Pre-warm with `max_tokens: 0` is a **sync** pattern. Do not expect it inside batches.
- Some models support extended batch max-output via beta headers (verify live docs). Do not invent headers in the exam. Know that sync and batch limits can differ.

### 11.4 Prompt caching as an MSO lever (depth)

Caching does not shrink context. It discounts **repeated prefix tokens**, and on hits it often improves TTFT.

**Prefix order (conceptual):** tools → system → messages, up through the `cache_control` breakpoint.

**TTL themes (public):**

- Public docs commonly discuss a default ephemeral lifetime of **~5 minutes**, refreshed on reads.
- A longer TTL (e.g. **1-hour**) is available at extra cost, for long gaps between hits.
- The lifetime starts at the request that writes or reads. Long streaming responses consume TTL wall-clock.

**What invalidates or misses caches (study list):**

- Any change to the cached prefix bytes (system text, tool JSON order/names, inserted timestamps).
- A model ID change mid-thread.
- A thinking/effort config mismatch vs the entry you wrote (invalidates message-level breakpoints. Tools/system prefix unaffected).
- Automatic caching keyed on a warmup placeholder user turn (use an **explicit** breakpoint on the shared system/tools for pre-warm).
- Changes to the tool order used as a "mode switch."

**MSO exam line:** The question says costs spiked after "we added a timestamp to the system prompt for debugging." The fix: remove volatile fields from the cached prefix. Do not select a cheaper model.

### 11.5 Context window strategy as optimization

Public model cards show **1M**-class windows on current Opus/Sonnet/Fable lines and smaller windows on some fast models. Exams test the *strategy*:

| Situation | Prefer |
| --- | --- |
| Multi-hour agent with large repo | Large-window model + compaction + retrieval. Do not send the entire monorepo every turn |
| Stable policies + docs | Cache them. Retrieve volatile slices |
| 50+ tools | Tool search / defer loading + cache the stub list |
| Cheap high-QPS extraction | Smaller/faster model + strict schema. A large window costs money only if you fill it |

**Rule:** A bigger window lets you hold more. You do not need to send more.

### 11.6 Routing architectures that show up as MSO+Integration

1. **Static route:** One pinned model serves a product surface.
2. **Cascade:** A Haiku/Sonnet attempt escalates to Opus on low confidence or validator failure.
3. **Segmented route:** Different models per tenant tier, locale, or risk class.
4. **Plan/execute split:** A stronger model plans. A faster model executes tool calls (Claude Code aliases encode this interactively. Services implement it explicitly).
5. **Offline/online split:** Batches for analytics. Sync for UX.

**Common error:** Cascades without budgets become accidental Opus-everywhere systems. Always cap escalations. Log the reason.

### 11.7 Cost accounting that exam writers like

Think in **unit economics**, not in total invoice values:

```text
success_adjusted_cost = (input + output + cache_write + cache_read + tool_overhead) / successful_task
```

- Cache **writes** cost more than base input. They pay back on hits.
- Batch discounts apply to eligible tokens. Combining with cache is allowed, but hits are probabilistic.
- Agent loops multiply turns. A "cheap" model with 3× turns can cost more than a stronger model with 1× turns.
- Output tokens often cost more than input. Schemas and "be concise" are MSO controls.

### 11.8 Pinning, deprecations, and migrations

Public docs keep deprecation/migration guidance. Primary study habits:

1. Pin model IDs in environment/config, not in scattered source files.
2. Keep a **compatibility matrix** per host (API vs Bedrock vs Vertex feature flags).
3. Migrate in this order: golden evals → shadow traffic → canary pin → full cutover → delete old pin.
4. Never change model, prompt, and tools in one release without a bisect plan.

### 11.9 Worked Integration×MSO scenarios (original)

**Scenario A — Support chat, p95 TTFT SLA.** 
Stream. Prefer Sonnet- or Haiku-class. Use moderate/low effort if quality holds. Cache the stable system+tools. Do **not** batch.

**Scenario B — Nightly PII-redacted classification of 2M tickets.** 
Use Message Batches. Use the smallest sufficient model. Share `cache_control` on instructions+label taxonomy. Accept best-effort cache hits. Do not stream.

**Scenario C — Coding agent with hard that fails intermittently bugs.**
Pin Opus-class. Raise effort before you change to rarer top-tier models. Keep tool schemas stable for cache. Compact the history. Verify with tests (a tool), not opinions.

**Scenario D — Multi-cloud enterprise, EU residency.** 
MSO includes **where** inference runs. Select regional endpoints that satisfy policy, even if global routing is faster. Confirm feature parity (caching/batch/tools) on that host.

**Scenario E — Cache miss storm after a "helpful" refactor.** 
Diff the request prefix. Look for tool reordering, dynamic dates, experiment flags in system text, or effort default changes after an SDK upgrade.

### 11.10 Metrics dashboard for MSO owners

| Metric | Why |
| --- | --- |
| Task success rate (eval + online) | Quality gate |
| p50/p95 TTFT and total latency | UX / SLA |
| Cache hit rate | Prefix quality |
| Escalation rate (cascades) | Hidden cost |
| $/successful task | True MSO KPI |
| Stop-reason distribution | Truncation / tool loops |
| Model pin version | Drift detection |

### 11.11 Additional Q&A (Q26–Q35)

**Q26.** A product manager wants one model for chat and overnight evals, "for consistency." What is the MSO rebuttal? 
**A26.** The quality *requirement* can stay consistent through shared rubrics and overlapping evals. The serving paths still differ. Chat needs sync/stream. Overnight work should batch. You may pin the same model ID on both paths if quality requires it — but do not force one serving mode.

**Q27.** When do you select a 1-hour cache TTL over the default short TTL? 
**A27.** When requests that share a prefix arrive with gaps longer than the short TTL (spiky traffic, business-hours tools). Also when the extra write/storage cost is lower than the cost of repeated full prefill misses.

**Q28.** Your Bedrock deployment lacks a beta tool feature you used on the Claude API. What is the correct response? 
**A28.** Redesign for the host's feature matrix (or run that slice on a host that supports it). Do not assume SKU name parity implies feature parity.

**Q29.** Is streaming cheaper? 
**A29.** Not inherently. It improves perceived latency and allows cancel. Token billing still applies to generated output.

**Q30.** An SDK upgrade changed the effort default, and costs rose. What do you check first? 
**A30.** Set effort explicitly in config to the previously validated value. Re-run evals. Confirm cache hit rates (a mismatch can also miss caches).

**Q31.** A large context window exists on a Haiku-class model. When is that still the wrong choice? 
**A31.** When the task needs frontier reasoning/autonomy that evals show Haiku fails. Window size does not replace the capability tier.

**Q32.** Agent loops burn budget. What is the first MSO move? 
**A32.** Add stop conditions, tool budgets, and progress checks. Only then consider a stronger model that finishes in fewer turns.

**Q33.** Why pin differently in Claude Code vs a backend service? 
**A33.** Code sessions may use workflow aliases and user `/model` switches. Backends need deterministic pins for SLOs, caches, and audit.

**Q34.** Cache write markup vs hit savings — when is caching correct on day one? 
**A34.** When the same prefix will be reused soon (agents, multi-turn apps, shared tool schemas). One-off unique prompts may not pay back.

**Q35.** Name three Integration features that change which model you pick. 
**A35.** The need for streaming UX. The need for batch economics. The need for tool/MCP features that are only stable on certain model lines. Also consider host/region constraints.

### 11.12 If exam asks X, think Y (MSO)

| If exam asks… | Think… |
| --- | --- |
| Lower cost, quality OK | Smaller model → lower effort → cache → batch → shorter outputs |
| Lower latency, human waiting | Faster tier / lower effort + stream + cache hits. Not batches |
| Same model, better hard cases | Raise effort / thinking. Improve tools/prompts before new tier |
| Reproducible prod | Pin model ID. Freeze prompt/tool versions. Record effort |
| Bulk offline | Message Batches (± cache). No streaming expectation |
| Cost spike after tiny prompt edit | Cache invalidation / prefix volatility |
| Multi-cloud | Host SKU + feature matrix + residency |
| Agent too expensive | Turns × tokens. Budgets and stop conditions first |

### 11.13 Primary-study checklist addendum

- [ ] I can explain pin vs alias without relying on a specific marketing name.
- [ ] I can draw sync vs stream vs batch with constraints.
- [ ] I can list five cache invalidators.
- [ ] I can compute success-adjusted cost conceptually.
- [ ] I know effort/thinking must match between warm and live traffic.
- [ ] I treat host/region as an MSO input, not an afterthought.
- [ ] I escalate the model tier only after evals fail at the current tier+effort.

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

*Added 2026-08-23 — the official LLM Fundamentals statement tests "tokens, context windows, sampling, non-determinism, next-token generation … and fundamental prompting techniques (zero-shot, single-shot, multi-shot)." This primer covers that gap.*

### 12.1 How the model generates

- **Tokens** are the unit for everything. Models read and write tokens (subword chunks — roughly ¾ of an English word each, on average). Billing, context windows, and `max_tokens` all count in tokens. Code, JSON, and non-English text tokenize less efficiently than plain English prose.
- **Next-token generation:** the model produces output one token at a time. Each choice depends on the entire visible context (the prompt + everything generated so far). There is no lookahead "plan" outside the tokens. Long structured outputs fail at the end, because each step compounds.
- **Sampling and non-determinism:** at each step, the model holds a probability distribution over possible next tokens, and it *samples* from that distribution. This is why two identical requests can produce different answers. Historically, `temperature` / `top_p` / `top_k` shaped that distribution. **Current state:** Opus 4.7+, Opus 5, Sonnet 5, and Fable-class models remove support for these sampling parameters (the API rejects requests that carry them). They remain only on older models (4.6-generation, Haiku 4.5).
- **The determinism problem:** `temperature: 0` never guaranteed byte-identical outputs, and on current models the setting does not exist. If a question needs repeatable structure, the correct controls are **schemas/structured outputs, validators, and few-shot anchors** — not sampling settings.

### 12.2 Shot-count terminology (define these exactly)

| Term | Meaning | Use when |
| --- | --- | --- |
| **Zero-shot** | Instructions only. No worked examples | Task is common/easy for the model. Keeps prompts short and cacheable |
| **Single-shot (one-shot)** | Exactly one worked example | Format is unusual but consistent. One example anchors shape cheaply |
| **Multi-shot (few-shot)** | Several worked examples | Edge cases, label boundaries, or house style matter. Select diverse, high-signal examples (edge cases > happy paths) |

**MSO angle:** Examples are in the prompt. Every shot costs tokens on every call. Keep the example block **stable** inside the cached prefix. Remove examples that no longer justify their tokens.

### 12.3 Self-check Q&A (Q36–Q40)

**Q36.** Why do two identical API calls return different wording? 
**A36.** Outputs are **sampled** from a next-token probability distribution. The non-determinism is inherent, not a bug.

**Q37.** A teammate sets `temperature: 0` on a current Opus-class model, "for deterministic JSON." Name two problems. 
**A37.** Current models do not support sampling params (the request errors). And even historically, temp 0 did not guarantee identical output. Use schemas + validation.

**Q38.** A classification prompt fails only on ambiguous boundary cases. What zero-shot-free fix avoids a model upgrade? 
**A38.** Go **multi-shot**: add a few examples that sit exactly on the confusing boundaries.

**Q39.** Why can a 2,000-word English prompt cost fewer tokens than 500 lines of JSON? 
**A39.** Tokenization efficiency differs. Prose compresses into tokens better than dense structured text.

**Q40.** Where should few-shot examples sit for a high-volume cached endpoint? 
**A40.** In the stable cached prefix (with system/tools), unchanged between requests. An edit to an example invalidates the cache.

---

## 13. Fast mode (blueprint-named model option — know it exists and when)

*Added 2026-08-23. The official LLM Fundamentals statement names **fast mode** next to extended thinking, adaptive thinking, and effort. This pack omitted it before. Facts below cached 2026-08 — a research-preview feature, so verify current docs before exam day.*

**What it is:** the same model runs at up to ~2.5× higher output tokens/second, at premium pricing. For example, Opus 5 fast costs roughly 2× the standard Opus 5 rate per MTok. It is a *speed/price* trade on identical intelligence — not a smaller model, not a quality change.

**Mechanics (conceptual):** available on **Claude Opus 5 and Opus 4.8 only**, as a research preview. Each request selects fast mode: beta messages endpoint + a fast-mode beta flag + a top-level `speed: "fast"` parameter. The response usage reports which speed served it. Fast mode has its **own rate limit**, separate from standard traffic.

**Where it is NOT available (high-signal exam facts):**

- Not on Bedrock / Vertex / Foundry — Claude API (and managed-agent sessions) only.
- Not with the **Batch API** (batches are the latency-tolerant path. Fast mode is the latency-critical one).
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

**When fast mode is better than a tier downgrade:** The task needs frontier quality, and the downgrade fails evals. Users feel the generation time. Examples: long code reviews or drafted documents streamed live to a waiting expert.

### 13.1 Self-check Q&A (Q41–Q45)

**Q41.** Fast mode vs a switch to Haiku-class — what is the core difference? 
**A41.** Fast mode keeps the **same model's quality** at higher output speed and higher price. A tier switch trades quality for speed/cost.

**Q42.** Can you run your overnight 200k-document batch in fast mode, "to finish sooner"? 
**A42.** No. Fast mode is not available with the Batch API, and batch work is latency-tolerant by definition. It is the wrong control.

**Q43.** A Bedrock-only enterprise asks for fast mode. Your response? 
**A43.** Not available there (Claude API research preview, Opus 5 / Opus 4.8 only). Offer streaming, caching, effort tuning, or tier choices on their host instead.

**Q44.** Fast-mode requests start to return 429 while standard Opus traffic is fine. Why is that possible? 
**A44.** Fast mode has its **own separate rate limit**. Retry after the indicated delay, or drop to standard speed (and accept the cache invalidation a speed switch causes).

**Q45.** The question says "reduce the bill." Is fast mode ever the answer? 
**A45.** No. Fast mode raises unit price for speed. Cost questions require smaller models, lower effort, caching, batches, and shorter outputs.

---

## 14. Current model selection card (dated snapshot — verify before exam day)

*Cached **2026-08**. This pack's doctrine stays "criteria over SKUs," but the official Model Selection statement asks about "Opus vs. Sonnet vs. Haiku use cases, **adaptive thinking support**." So learn the concrete selection once, then reason with the decision trees. Model cards change. Re-check the public docs in the week before your exam.*

| Tier | Model | Context | Thinking | Effort levels | Relative price (in/out per MTok) |
| --- | --- | --- | --- | --- | --- |
| Top (above Opus) | Claude Fable 5 | 1M | **Always on** (adaptive. Cannot be disabled) | low → xhigh, max | ~$10 / $50 |
| Frontier | Claude Opus 5 | 1M | Adaptive **on by default** (omit = thinking) | low → xhigh, max | ~$5 / $25 |
| Frontier | Claude Opus 4.8 / 4.7 | 1M | Adaptive available — **opt in** (omit = no thinking) | low → xhigh, max | ~$5 / $25 |
| Frontier (older) | Claude Opus 4.6 | 1M | Adaptive recommended. Legacy `budget_tokens` deprecated | low/medium/high/max (no xhigh) | ~$5 / $25 |
| Balanced | Claude Sonnet 5 | 1M | Adaptive (omit = adaptive) | low → xhigh, max | ~$3 / $15 |
| Balanced (older) | Claude Sonnet 4.6 | 1M | Adaptive recommended. `budget_tokens` deprecated | low/medium/high/max | ~$3 / $15 |
| Fast/cheap | Claude Haiku 4.5 | **200K** | **Extended thinking** style (`type:"enabled"` + `budget_tokens`) | No effort parameter | ~$1 / $5 |

**Card-reading rules (the part exams test):**

1. **Adaptive thinking support** = the 4.6-and-later Opus/Sonnet lines plus Fable-class. The newest tier: thinking is always-on. Opus 5: default-on. 4.7/4.8: you must select it. Haiku 4.5 still uses the older fixed-budget extended-thinking style.
2. The fixed **`budget_tokens`** thinking budget is a **legacy** control: deprecated on the 4.6 generation, removed on newer lines. "Adaptive thinking + effort" is the current answer when a question mentions thinking budgets.
3. **`xhigh` effort** exists from Opus 4.7 onward (and Sonnet 5 / Fable-class). It is the default in Claude Code. 4.6-generation models stop at high/max. Haiku 4.5 has no effort setting.
4. Context: 1M-class on frontier/balanced lines. Haiku-class is smaller (200K). Long-repo questions eliminate Haiku on window alone.
5. Prices are **relative anchors**, not memorization targets. Reason in ratios: Fable ≈ 2× Opus. Opus ≈ 1.7× Sonnet. Sonnet ≈ 3× Haiku (input). Never quote absolute prices in prod docs without a check of the live pricing page.

---

## 15. Closing — Chapter 01 as primary study

MSO is 16.8% on its own, and it is the decision engine inside much of Applications and Integration. If you can select **sufficient intelligence**, the right **serving path**. Also, good **cache/pin discipline** under a stated constraint, you practice the highest-frequency judgment pattern on CCDV-F.
