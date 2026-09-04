---
title: Context Management & Reliability
exam: Claude Certified Architect – Foundations (CCAR-F), Domain 5 (15%)
disclaimer: Original study notes for exam prep — not official Anthropic material. Not a lesson transcript.
created: 2026-08-23
---

# Context Management & Reliability — Domain 5 (15%)

> **Disclaimer:** Original exam-prep study synthesis, aligned to the **official CCAR-F Exam Guide v1.0 (July 2026)** task statements 5.1–5.6 and public Anthropic docs.

> **Why this file exists:** Domain 5 threads through **four of the six exam scenarios** (customer support, Claude Code sessions, multi-agent research, structured extraction). It tests the judgment layer: what to keep in context, when to escalate, how errors travel between agents, and how humans stay calibrated on model output.

---

## Overview

Six task statements: **5.1** preserve critical information across long interactions; **5.2** escalation and ambiguity resolution; **5.3** error propagation across multi-agent systems; **5.4** context in large-codebase exploration; **5.5** human review workflows and confidence calibration; **5.6** information provenance and uncertainty in multi-source synthesis.

**Master mental model:** context is a lossy, position-sensitive, finite buffer. Reliability engineering here means (a) deciding *what must never be lossy* (case facts, provenance, error detail) and moving it into **structured layers** outside free-flowing history, and (b) never trusting a single unstructured signal (model confidence, sentiment, aggregate accuracy) to make a routing decision.

---

## Key map

| Task | Core skill |
| --- | --- |
| 5.1 | Case-facts blocks, trimming verbose tool output, position-aware ordering |
| 5.2 | Explicit escalation criteria; honor human requests; policy-gap detection |
| 5.3 | Structured error context; access-failure vs empty-result; local recovery |
| 5.4 | Scratchpad files, subagent delegation, crash-recovery manifests, /compact |
| 5.5 | Stratified sampling, field-level confidence calibrated on labeled sets |
| 5.6 | Claim–source mappings, conflict annotation, temporal metadata |

---

## Deep notes

### 1. Preserving critical information in long interactions (task 5.1)

**Progressive summarization risk:** every summarization pass tends to condense **numerical values, percentages, dates, and customer-stated expectations** into vague prose ("a partial refund was discussed" losing "$127.50 by Friday"). Exact transactional facts are precisely the things summaries destroy first.

**Fix — the persistent "case facts" block:** extract transactional facts (amounts, dates, order numbers, statuses) into a structured block that is included **in each prompt, outside the summarized history**. Summaries can stay lossy because the facts layer is lossless. For multi-issue sessions, persist structured issue data (order IDs, amounts, per-issue status) as its own context layer.

**Lost in the middle:** models reliably process the **beginning and end** of long inputs but may omit findings from **middle sections**. Mitigations: put **key-findings summaries at the beginning** of aggregated inputs; organize detail with **explicit section headers**; don't bury the decisive constraint at position 60% of a 100K-token prompt.

**Tool-result bloat:** tool results accumulate tokens disproportionately to their relevance — an order lookup returns 40+ fields when 5 matter. **Trim verbose tool outputs to return-relevant fields before they enter context.** In multi-agent systems, push this upstream: subagents with limited-context consumers downstream should return **structured data (key facts, citations, relevance scores), not verbose content and reasoning chains**, and include metadata (dates, source locations, methodological context) for accurate synthesis.

**Conversational coherence basics:** pass the complete conversation history on subsequent API requests — the API is stateless; "the agent forgot the last turn" usually means the client didn't resend it.

### 2. Escalation and ambiguity resolution (task 5.2)

**Legitimate escalation triggers (the guide's list):**

1. The customer **explicitly requests a human** — escalate **immediately**, without first attempting investigation. (If the issue is simple, you may acknowledge frustration and *offer* the fix, but escalate if they reiterate.)
2. **Policy exceptions or gaps** — the policy is ambiguous or silent on the specific request (competitor price-matching when policy only covers own-site adjustments). Note: this is about *policy coverage*, not case difficulty.
3. **Inability to make meaningful progress.**

**Unreliable escalation signals (tested as wrong answers):**

- **Self-reported confidence scores** — poorly calibrated; the agent is confidently wrong exactly on the hard cases (official sample Q3's wrong option B).
- **Sentiment analysis** — frustration doesn't correlate with case complexity (Q3's wrong option D).
- The proportionate first fix for bad escalation calibration is **explicit escalation criteria with few-shot examples in the system prompt** demonstrating escalate-vs-resolve (Q3's correct A) — before any classifier infrastructure.

**Ambiguity resolution:** when a lookup returns **multiple customer matches**, instruct the agent to **request additional identifiers** — never pick heuristically ("probably the most recent account").

### 3. Error propagation across multi-agent systems (task 5.3)

**Structured error context** is what lets a coordinator make intelligent recovery decisions. On failure, a subagent returns: **failure type, the attempted query, any partial results, and potential alternative approaches** (official sample Q8's correct A).

**The two anti-patterns (both named by the guide):**

- **Silent suppression** — returning empty results marked as success hides the failure and risks incomplete output presented as complete.
- **Terminating the whole workflow on a single failure** — kills recovery strategies that could have succeeded (retry-with-modified-query, alternative source, proceed-with-partial).

**Access failure vs valid empty result:** a timeout ("couldn't reach the source" → retry decision needed) is fundamentally different from a successful query with no matches (a real answer). Error reporting must distinguish them, or the coordinator retries empty results and accepts outages.

**Local recovery first:** subagents handle **transient** failures themselves (retry with backoff); they propagate only errors they **cannot resolve locally**, including what was attempted and partial results. Generic statuses ("search unavailable") hide the context recovery needs.

**Coverage annotation:** when synthesis proceeds despite gaps, the output must **annotate which findings are well-supported and which topic areas have gaps due to unavailable sources** — degraded coverage must be visible, not silent.

### 4. Context in large-codebase exploration (task 5.4)

**Degradation symptoms:** in extended sessions the model starts giving **inconsistent answers** and referencing "typical patterns" instead of the **specific classes it discovered earlier** — generic-knowledge fallback is the tell that discovered facts have been diluted or evicted.

**Countermeasures (all four are exam-listed skills):**

1. **Scratchpad files** — the agent maintains a file of key findings and **references it for subsequent questions**, persisting knowledge across context boundaries. Files are lossless; context is not.
2. **Subagent delegation** — spawn subagents for verbose investigations ("find all test files," "trace refund-flow dependencies"); exploration output stays in the subagent's context, the main agent keeps high-level coordination clean.
3. **Phase summaries** — summarize key findings from one exploration phase before spawning the next phase's subagents, and **inject the summary into their initial context**.
4. **`/compact`** — reduce context during extended sessions when it fills with verbose discovery output (in Claude Code, pairs with directed compaction — see file 05).

**Crash recovery with manifests:** structured state persistence = each agent **exports its state to a known location**; on resume, the coordinator **loads a manifest and injects it into agent prompts**. A crash then costs the tail of one phase, not the whole run. (Same files-outrank-context principle as scratchpads, applied to whole-system recovery.)

### 5. Human review workflows and confidence calibration (task 5.5)

**Aggregate metrics mask segment failure:** "97% overall accuracy" can hide 60% accuracy on one document type or one field. **Validate accuracy by document type and field segment before automating** high-confidence extractions.

**Stratified random sampling:** even extractions the model marks high-confidence get **ongoing random sampling per stratum** (document type × field) to measure true error rates and catch **novel error patterns**. Without it, drift is invisible until an incident.

**Calibrated field-level confidence:** have the model output **per-field confidence scores**, then **calibrate review thresholds using labeled validation sets** — raw self-reported confidence is not trustworthy (same lesson as 5.2); calibration against labels is what makes it usable for routing.

**Routing rule:** low-confidence extractions and **ambiguous/contradictory source documents** go to human review, prioritizing limited reviewer capacity where error probability is genuinely highest.

### 6. Provenance and uncertainty in multi-source synthesis (task 5.6)

**Attribution dies in summarization:** when findings are compressed without preserving **claim–source mappings**, the final report can't say which source supports which claim. Fix: subagents output **structured claim–source mappings (claim, source URL/document name, relevant excerpt)** that downstream agents **preserve and merge through synthesis** — provenance is a data contract, not a prose habit.

**Conflicting statistics from credible sources:** **annotate the conflict with source attribution** — never arbitrarily pick one value, and never average. Document analysis completes **with conflicting values included and explicitly annotated**, letting the coordinator decide reconciliation before synthesis.

**Temporal metadata:** require **publication/collection dates in structured outputs** so a 2024 figure and a 2026 figure read as a time series, not a contradiction.

**Report structure:** explicit sections distinguishing **well-established findings from contested ones**, preserving original source characterizations and methodological context. Render content appropriately — **financial data as tables, news as prose, technical findings as structured lists** — instead of flattening everything to uniform prose.

---

## Decision trees

**"Where should this information live?"**

```text
Exact transactional fact (amount, date, ID)?
 → Case-facts block in every prompt, outside summarized history
Discovered codebase/system fact needed later?
 → Scratchpad file (re-read on demand)
Verbose raw tool output?
 → Trim to relevant fields BEFORE it enters context
Cross-phase / cross-crash state?
 → Structured state export + coordinator-loaded manifest
Who-said-what across sources?
 → Claim–source mapping preserved through every hand-off
```

**"Escalate or resolve?"**

```text
Customer explicitly asks for a human → escalate NOW (no investigation first)
Policy silent/ambiguous on the request → escalate (policy gap)
No meaningful progress possible → escalate with structured handoff (file 08 §4)
Merely complex but in-policy and progressing → keep resolving
Multiple identity matches → ask for more identifiers, never guess
```

**"A subagent failed — what flows upward?"**

```text
Transient? → local retry first; nothing propagates if recovered
Unrecoverable? → failure type + attempted query + partial results + alternatives
Empty result set? → report as SUCCESS with zero matches (not an error)
Synthesis proceeding anyway? → coverage-gap annotations in the output
```

---

## Exam traps

| Trap | Fix |
| --- | --- |
| Summarize everything to save tokens | Numbers/dates/IDs go in a lossless case-facts layer first |
| Key constraint buried mid-prompt | Beginning/end placement + section headers (lost-in-the-middle) |
| Full 40-field tool payloads in history | Trim to relevant fields upstream |
| Route escalation on sentiment or self-confidence | Explicit criteria + few-shots; calibrate confidence on labels before trusting it |
| Investigate first when the customer demanded a human | Escalate immediately |
| "Search unavailable" as the error message | Structured error context (type, attempt, partials, alternatives) |
| Empty result treated as failure (or failure as empty success) | Distinguish access failure from valid empty result |
| One subagent timeout kills the run | Local recovery, then partial-results propagation + coverage notes |
| Trusting "97% accurate" to automate | Segment by doc type × field; stratified sampling of high-confidence strata |
| Averaging or picking between conflicting source stats | Annotate both with sources; coordinator reconciles |
| Dates dropped from findings | Publication/collection dates required in structured outputs |
| Session cites "typical patterns" not discovered classes | Context degradation → scratchpad + subagent delegation + /compact |

---

## Self-check Q&A

**Q1.** What kinds of information does progressive summarization destroy first?
**A.** Exact numbers, percentages, dates, order numbers, and customer-stated expectations — condensed into vague prose.

**Q2.** Define the case-facts pattern.
**A.** Extract transactional facts into a structured block included in every prompt outside summarized history — a lossless layer beneath lossy summaries.

**Q3.** What is lost-in-the-middle and two mitigations?
**A.** Reliable processing at beginning/end of long inputs but omissions from the middle; put key-findings summaries first and use explicit section headers.

**Q4.** An order lookup returns 40+ fields, 5 relevant. What do you do and when?
**A.** Trim to return-relevant fields *before* the result enters context.

**Q5.** What should upstream subagents return when downstream context is tight?
**A.** Structured data — key facts, citations, relevance scores, plus dates/source metadata — not verbose content and reasoning chains.

**Q6.** The three legitimate escalation triggers?
**A.** Explicit customer request for a human; policy exceptions/gaps; inability to make meaningful progress.

**Q7.** Customer demands a human on a trivially fixable issue. Sequence?
**A.** Honor it immediately; at most acknowledge and offer the fix once, escalating if they reiterate — never investigate first against their request.

**Q8.** Why are sentiment and self-reported confidence bad escalation routers?
**A.** Neither correlates with case complexity; confidence is uncalibrated — the agent is confidently wrong on hard cases (sample Q3).

**Q9.** Tool returns three customers for "John Smith." Correct behavior?
**A.** Request additional identifiers; never select heuristically.

**Q10.** A search subagent times out. What does the coordinator need to recover intelligently?
**A.** Structured error context: failure type, attempted query, partial results, alternative approaches (sample Q8).

**Q11.** Name the two error-handling anti-patterns and the harm of each.
**A.** Silent suppression (failure dressed as empty success → incomplete output looks complete) and whole-workflow termination on one failure (kills viable recovery).

**Q12.** Access failure vs valid empty result — why distinguish?
**A.** One needs a retry decision, the other is a real answer; conflating them causes wrong retries and silently accepted outages.

**Q13.** When does an error leave a subagent?
**A.** Only when local recovery (e.g. retry for transients) failed — propagated with what was attempted and partial results.

**Q14.** Synthesis ran with one source down. What must the report contain?
**A.** Coverage annotations: which findings are well-supported, which topic areas have gaps from unavailable sources.

**Q15.** Symptom that a long exploration session's context has degraded?
**A.** Inconsistent answers and appeals to "typical patterns" instead of the specific classes discovered earlier.

**Q16.** Four countermeasures for large-codebase context management?
**A.** Scratchpad files; subagent delegation for verbose investigation; phase summaries injected into next-phase context; /compact.

**Q17.** Design crash recovery for a multi-agent run.
**A.** Each agent exports structured state to a known location; on resume the coordinator loads the manifest and injects it into agent prompts.

**Q18.** Why is 97% overall accuracy insufficient to automate?
**A.** Aggregates mask segment failure — accuracy must be validated per document type and field before reducing human review.

**Q19.** Role of stratified random sampling in a review workflow?
**A.** Ongoing error-rate measurement of high-confidence extractions per stratum, catching novel error patterns and drift.

**Q20.** How does model confidence become usable for review routing?
**A.** Model outputs field-level scores; thresholds are calibrated against labeled validation sets — raw self-report is not trusted.

**Q21.** Two credible sources give different market sizes. Handling?
**A.** Include both, annotated with source attribution; the coordinator decides reconciliation — never arbitrary selection or averaging.

**Q22.** Why require publication dates in structured outputs?
**A.** So temporal differences read as change over time rather than contradiction.

**Q23.** What keeps attribution alive through multi-agent synthesis?
**A.** Structured claim–source mappings (claim, source, excerpt) that every downstream agent preserves and merges.

**Q24.** How should a synthesis report treat contested findings?
**A.** Separate explicit sections for well-established vs contested, keeping original source characterizations and methodology context.

**Q25.** Financial data, news, and technical findings in one report — format?
**A.** Tables, prose, and structured lists respectively — render per content type, don't flatten.

---

## Pre-exam checklist

- [ ] I can spec a case-facts block and say what summarization destroys.
- [ ] I can state lost-in-the-middle and its two mitigations.
- [ ] I trim tool outputs upstream and know what subagents should return downstream.
- [ ] I can recite the three escalation triggers and the two unreliable signals.
- [ ] I can spec structured error context and both error anti-patterns.
- [ ] I distinguish access failures from empty results on sight.
- [ ] I can name all four codebase-context countermeasures + manifest recovery.
- [ ] I reflexively segment accuracy and stratify sampling before automating.
- [ ] I preserve claim–source mappings and annotate conflicts with dates.

---

## Glossary

| Term | Meaning |
| --- | --- |
| Case-facts block | Structured lossless facts layer included in every prompt outside history |
| Progressive summarization | Repeated compression of history; destroys exact values first |
| Lost in the middle | Position effect: middle of long inputs gets omitted |
| Tool-result trimming | Reducing payloads to relevant fields before context entry |
| Escalation trigger | Human request, policy gap, or no-progress — not sentiment/confidence |
| Policy gap | Request the policy is silent/ambiguous on → escalate |
| Structured error context | Failure type + attempted query + partials + alternatives |
| Silent suppression | Failure returned as empty success — anti-pattern |
| Local recovery | Subagent retries transients before propagating |
| Coverage annotation | Marking well-supported vs gap areas in synthesis output |
| Scratchpad file | Agent-maintained findings file persisting across context limits |
| Manifest | Coordinator-loaded structured state export enabling crash resume |
| Stratified sampling | Random QA sampling per doc-type/field stratum, incl. high-confidence |
| Calibration | Mapping model confidence to real error rates via labeled sets |
| Claim–source mapping | Structured claim + source + excerpt preserved through synthesis |
| Temporal annotation | Publication/collection dates attached to findings |

---

*Companion files: 08 (Domain 1 orchestration — handoff structure, subagent mechanics), 05 (+ supplement) for /compact and scratchpad practice in Claude Code.*
