# Shared Core — the 10 doctrines all four exams test

> **Note:** This edition follows the ASD-STE100 Simplified Technical English writing rules. Simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Haiku, Sonnet, Opus, Fable, MCP, CLAUDE.md, prompting, caching, effort, Batch, Pearson VUE) are exceptions and stay as written.

*Learn each doctrine once. Then read its four-exam table. The principle is the same. Each exam uses a different level of detail and vocabulary. Sources: the four official Exam Guides v1.0 (July 2026) and their sample questions.*

---

## D1. Verify before it ships

Do not trust anything Claude produces until you check it. Open citations. Recompute numbers. Validate outputs. The check must match the blast radius of a wrong answer.

| Exam | How the exam tests it |
| --- | --- |
| CCAO-F | The largest domain (21%): ship, revise, or reject judgment. Check the citation before the compliance send (official sample) |
| CCAR-F | Confidence calibration, stratified sampling, claim–source provenance (Domain 5) |
| CCDV-F | Eval harnesses, regression sets, validators before side effects |
| CCAR-P | Gold + adversarial + regression eval sets. Shadow before cutover. Code judges before model judges |

## D2. A deterministic mechanism is better than a probabilistic one.

Prompts and few-shots *lower* failure rates. Gates, hooks, and code make the failure rate *zero*. Stems that contain must / never / compliance / a dollar threshold ask for the deterministic mechanism.

| Exam | The mechanism vocabulary |
| --- | --- |
| CCAO-F | Human approval gate before external/irreversible sends |
| CCAR-F | Programmatic prerequisite. PreToolUse-style interception hook (sample Q1) |
| CCDV-F | Schema validation in code. Claude Code settings/hooks enforce. CLAUDE.md advises |
| CCAR-P | Workflow gates + authZ enforced in the tool server. Caps in code, never model judgment |

## D3. Least privilege everywhere

For tools, data, permissions, and sharing: grant the minimum that is sufficient for the task. Removal of capability is a *valid first fix*. It is not an admission of failure.

| Exam | Flavor |
| --- | --- |
| CCAO-F | Sharing permissions (view vs edit), connector scope, PII minimization |
| CCAR-F | Subagent `tools` restriction lists. The reviewer gets read-only |
| CCDV-F | Tool allowlists. Injection blast-radius control |
| CCAR-P | Remove tools first (sample Q1). Per-agent credentials. authZ at the owning seam |

## D4. Smallest model that passes evals

Default to the lightest tier that meets the quality requirement. Escalate only on evidence. "Use the model with the most capability" is a distractor on all four exams.

**Shared facts (learn once — dated Aug 2026, check again in exam week):** Tiers run Haiku (fast/cheap, 200K ctx) → Sonnet (default). Then Opus (deep) → Fable-class (longest-horizon frontier). Current 4.6+ generations use **adaptive thinking**. Fixed `budget_tokens` is legacy. Current tiers remove sampling params like temperature. Effort (low to max, `xhigh` from Opus 4.7 on) controls cost and depth inside a model. Current frontier and balanced tiers have 1M-class context.

| Exam | Altitude |
| --- | --- |
| CCAO-F | Product picker by task type. The guide names only Haiku/Sonnet/Opus. Fable-maximalism is a trap |
| CCAR-F | Cheaper model per subagent (`model` override) for bulk reading |
|CCDV-F. |Eval-driven selection + effort setting + fast mode (CCDV-only fact set). |
| CCAR-P | Eval-proven promotion/demotion. Router architectures. Cost envelopes |

## D5. Prefer the simplest pattern that meets the requirement

Single call → workflow → agent → multi-agent. Move right only when evidence shows the left option fails. The left options give auditability. The right

| Exam | Framing |
| --- | --- |
| CCAO-F | Chat → Project → Skill/connector ladder. Do not build a workflow when a good prompt does the job |
| CCAR-F | Prompt chaining (steps enumerable) vs adaptive decomposition (steps emerge). Agent = model-driven tool choice |
| CCDV-F | Workflow + smaller model beats agent-everywhere. Frameworks add structure. They do not add hidden capability |
| CCAR-P | The leftmost-pattern rule, verbatim. Multi-agent without role boundaries is not a real design |

## D6. Context is a budget

Everything you send counts. More context is not more quality. Curate the context. Retrieve it in steps. Protect what you must never summarize away.

| Exam | Framing |
| --- | --- |
| CCAO-F | Project knowledge hygiene: stale or duplicate files damage every chat |
| CCAR-F | Lost-in-the-middle, case-facts blocks, per-file + integration passes for attention dilution (sample Q12) |
| CCDV-F | Context budgeting, compaction, tool-catalog size, token counting |
| CCAR-P | Progressive discovery vs monolithic dump. Compaction drift as a failure mode |

## D7. Cache-stable prefixes *(CCDV + CCAR-P only — out of scope for CCAR-F, absent from CCAO)*

Put stable content first. Put volatile content last. Render order is tools → system → messages. Use about 4 breakpoints. A byte change invalidates everything after it. A thinking-config change invalidates message-level entries. The tools/system prefix survives. Cached tokens still occupy the window. CCAR-P sample Q2 and about half of CCDV cache stems test this one paragraph.

## D8. Batch vs realtime *(CCDV + CCAR-P)*

If a human waits, use sync. Stream if you need better perceived latency. If the work is overnight, bulk, or cost-driven, use **Batch**. Batch is about 50% of sync cost. It is async. It has no streaming. Cache hits are best-effort. It has no pre-warm. Both exams' sample sets start with this trade.

## D9. MCP mental model *(CCDV + CCAR-F + CCAR-P. Concepts only in CCAO's "connectors")*

Know the host, client, and server roles. Know tools, resources, and prompts. Build an MCP server when **reuse across apps or teams** matters. Third-party servers are supply chain. Trust them and set their scope. Mechanics differ per exam (see the divergence map). CCAR-F wants config and error mechanics. CCAR-P wants authZ-in-the-server architecture. CCDV wants the connector and integration picture.

## D10. Enforced vs advisory in Claude Code *(CCDV + CCAR-F + CCAR-P)*

Settings, permissions, managed policy, and hooks **enforce**. CLAUDE.md and prompts **advise**. Org-managed settings cannot be overridden. Version control distributes project-level config. User-level config does not. That difference is a recurring diagnostic stem in all three technical exams.

---

## Exam-craft constants (identical across all four)

- **720 pass on a 100–1,000 scaled score**. This is not a fixed %. The program equates forms.
- **No penalty for wrong answers**. Never leave a blank. Flag the item and return to it.
- **Multiple-response is all-or-nothing**. Each item states how many to select.
- **Retakes:** 14/30/90-day waits. Max 4 attempts per rolling 12 months. This is relevant to your run. You cannot retry a failed exam inside this window's remaining days. Sleep well before the exam. Do not study intensively at the last minute.
- **Constraint-first reading:** Read the question sentence first. Find the stem's constraints. Remove any option that violates one constraint. Then select the simplest survivor. This is the shared solving algorithm. Only the vocabulary changes.
