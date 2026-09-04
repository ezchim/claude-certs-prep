# Shared Core — the 10 doctrines all four exams test

*Learn each doctrine once, then read its four-exam table: same principle,
different altitude and vocabulary per exam. Sources: the four official Exam
Guides v1.0 (July 2026) and their sample questions.*

---

## D1. Verify before it ships

Nothing Claude produces is trusted until checked — citations opened, numbers
recomputed, outputs validated — and the check is proportionate to the blast
radius of being wrong.

| Exam | How it's tested |
| --- | --- |
| CCAO-F | The 21% mega-domain: ship/revise/reject judgment; verify the citation before the compliance send (official sample) |
| CCAR-F | Confidence calibration, stratified sampling, claim–source provenance (Domain 5) |
| CCDV-F | Eval harnesses, regression sets, validators before side effects |
| CCAR-P | Gold + adversarial + regression eval sets; shadow before cutover; code judges before model judges |

## D2. Deterministic beats probabilistic

Prompts and few-shots *lower* failure rates; gates, hooks, and code make them
*zero*. Stems containing must / never / compliance / a dollar threshold are
asking for the deterministic mechanism.

| Exam | The mechanism vocabulary |
| --- | --- |
| CCAO-F | Human approval gate before external/irreversible sends |
| CCAR-F | Programmatic prerequisite; PreToolUse-style interception hook (sample Q1) |
| CCDV-F | Schema validation in code; Claude Code settings/hooks enforce, CLAUDE.md advises |
| CCAR-P | Workflow gates + authZ enforced in the tool server; caps in code, never model judgment |

## D3. Least privilege everywhere

Tools, data, permissions, sharing: grant the minimum that does the job, and
removing capability is a *first-class fix*, not an admission of failure.

| Exam | Flavor |
| --- | --- |
| CCAO-F | Sharing permissions (view vs edit), connector scope, PII minimization |
| CCAR-F | Subagent `tools` restriction lists; reviewer gets read-only |
| CCDV-F | Tool allowlists; injection blast-radius control |
| CCAR-P | Remove tools first (sample Q1); per-agent credentials; authZ at the owning seam |

## D4. Smallest model that passes evals

Default to the lightest tier that meets the quality bar; escalate only on
evidence; "use the biggest model" is a planted distractor on all four exams.

**Shared facts (learn once — dated Aug 2026, re-verify exam week):** tiers
run Haiku (fast/cheap, 200K ctx) → Sonnet (default) → Opus (deep) → Fable-class
(longest-horizon frontier); current 4.6+ generations use **adaptive thinking**
(fixed `budget_tokens` is legacy; sampling params like temperature are removed
on current tiers); **effort** (low→max, `xhigh` from Opus 4.7 on) is the
intra-model cost/depth dial; 1M-class context on current frontier/balanced
tiers.

| Exam | Altitude |
| --- | --- |
| CCAO-F | Product picker by task type — guide names only Haiku/Sonnet/Opus; Fable-maximalism is a trap |
| CCAR-F | Cheaper model per subagent (`model` override) for bulk reading |
| CCDV-F | Eval-driven selection + effort dial + fast mode (CCDV-only fact set) |
| CCAR-P | Eval-proven promotion/demotion; router architectures; cost envelopes |

## D5. Prefer the simplest pattern that meets the requirement

Single call → workflow → agent → multi-agent: move right only when evidence
says the left fails. Auditability lives left; flexibility lives right.

| Exam | Framing |
| --- | --- |
| CCAO-F | Chat → Project → Skill/connector ladder; don't build a workflow where a good prompt does |
| CCAR-F | Prompt chaining (steps enumerable) vs adaptive decomposition (steps emerge); agent = model-driven tool choice |
| CCDV-F | Workflow + smaller model beats agent-everywhere; frameworks add structure, not magic |
| CCAR-P | The leftmost-pattern rule verbatim; multi-agent without role boundaries is theater |

## D6. Context is a budget

Everything sent counts; more context is not more quality. Curate, retrieve
progressively, and protect what must never be summarized away.

| Exam | Framing |
| --- | --- |
| CCAO-F | Project knowledge hygiene: stale/duplicate files poison every chat |
| CCAR-F | Lost-in-the-middle, case-facts blocks, per-file + integration passes for attention dilution (sample Q12) |
| CCDV-F | Context budgeting, compaction, tool-catalog size, token counting |
| CCAR-P | Progressive discovery vs monolithic dump; compaction drift as failure mode |

## D7. Cache-stable prefixes *(CCDV + CCAR-P only — out of scope for CCAR-F, absent from CCAO)*

Stable content first, volatile last; render order tools → system → messages;
~4 breakpoints; byte changes invalidate everything after them; thinking-config
changes invalidate message-level entries (tools/system prefix survives);
cached tokens still occupy the window. CCAR-P sample Q2 and half of CCDV's
cache stems are this one paragraph.

## D8. Batch vs realtime *(CCDV + CCAR-P)*

Human waiting → sync (stream for perceived latency). Overnight/bulk/cost →
**Batch** (~50% cost, async, no streaming, best-effort cache hits, no
pre-warm). Both exams' sample sets open with this trade.

## D9. MCP mental model *(CCDV + CCAR-F + CCAR-P; concepts only in CCAO's "connectors")*

Host/client/server roles; tools/resources/prompts; build an MCP server when
**reuse across apps/teams** matters; third-party servers are supply chain —
trust and scope them. Mechanics diverge per exam (see divergence map):
CCAR-F wants config/error mechanics, CCAR-P wants authZ-in-the-server
architecture, CCDV wants the connector/integration picture.

## D10. Enforced vs advisory in Claude Code *(CCDV + CCAR-F + CCAR-P)*

Settings, permissions, managed policy, and hooks **enforce**; CLAUDE.md and
prompts **advise**. Org-managed settings are non-overridable; project-level
config travels via version control, user-level config doesn't — that
difference is a recurring diagnostic stem in all three technical exams.

---

## Exam-craft constants (identical across all four)

- **720 pass on a 100–1,000 scaled score** — not a fixed %; forms are equated.
- **No penalty for wrong answers** — never leave a blank; flag and return.
- **Multiple-response is all-or-nothing**; each item states how many to select.
- **Retakes:** 14/30/90-day waits, max 4 attempts per rolling 12 months —
 relevant to your run: a failed exam cannot be retried inside this window's
 remaining days, so protect sleep over cramming.
- **Constraint-first reading:** question sentence first, harvest the stem's
 constraints, kill options that violate any one, then pick the simplest
 survivor. This is the shared solving algorithm; only the vocabulary changes.
