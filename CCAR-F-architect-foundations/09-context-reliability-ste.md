---
title: Context Management & Reliability — CCAR-F Domain 5 Study Notes — Simplified Technical English
exam: Claude Certified Architect – Foundations (CCAR-F), Domain 5 (15%)
disclaimer: Original study notes for exam prep — not official Anthropic material. Not a lesson transcript.
created: 2026-08-23
---

# Context Management & Reliability — Domain 5 (15%)

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Claude Code, MCP, `/compact`, case-facts, scratchpad, manifest, RAG, eval, p95, TTL, JSON, QPS, SLA, Task) are exceptions and stay as written.

> **Disclaimer:** Original exam-prep study synthesis. It aligns to the **official CCAR-F Exam Guide v1.0 (July 2026)** task statements 5.1–5.6 and public Anthropic docs.

> **Why this file exists:** Domain 5 appears in **four of the six exam scenarios**. The scenarios are customer support, Claude Code sessions, multi-agent research, and structured extraction. The domain tests the judgment layer. You decide what to keep in context. You decide when to escalate. You decide how errors travel between agents. You decide how humans stay calibrated on model output.

---

## Overview

The exam has six task statements.

**5.1** preserve critical information across long interactions.

**5.2** escalation and ambiguity resolution.

**5.3** error propagation across multi-agent systems.

**5.4** context in large-codebase exploration.

**5.5** human review workflows and confidence calibration.

**5.6** information provenance and uncertainty in multi-source synthesis.

**Master mental model:** context is a lossy, position-sensitive, finite buffer.

Reliability engineering here means two things.

(a) You decide *what must never be lossy*. Those items are case facts, provenance, and error detail. You move them into **structured layers** outside free-flowing history.

(b) You never trust a single unstructured signal to make a routing decision. Unstructured signals include model confidence, sentiment, and aggregate accuracy.

---

## Key map

| Task | Core skill |
| --- | --- |
| 5.1 | Case-facts blocks. Trim verbose tool output. Use position-aware ordering. |
| 5.2 | Explicit escalation criteria. Honor human requests. Detect policy gaps. |
| 5.3 | Structured error context. Access-failure vs empty-result. Local recovery. |
| 5.4 | Scratchpad files, subagent delegation, crash-recovery manifests, `/compact` |
| 5.5 | Stratified sampling. Field-level confidence calibrated on labeled sets. |
| 5.6 | Claim–source mappings. Conflict annotation. Temporal metadata. |

---

## Deep notes

### 1. Preserving critical information in long interactions (task 5.1)

**Progressive summarization risk:** each summarization pass tends to condense **numerical values, percentages, dates, and customer-stated expectations** into vague prose. Example: "a partial refund was discussed" loses "$127.50 by Friday". Exact transactional facts are the first things that summaries destroy.

**Fix — the persistent "case facts" block:** extract transactional facts into a structured block. The facts include amounts, dates, order numbers, and statuses. Include that block **in each prompt, outside the summarized history**. Summaries can stay lossy because the facts layer is lossless. For multi-issue sessions, store structured issue data as its own context layer. That issue data includes order IDs, amounts, and per-issue status.

**Lost in the middle:** models process the **beginning and end** of long inputs with high reliability. They may omit findings from **middle sections**. Mitigations: put **key-findings summaries at the beginning** of aggregated inputs. Organize detail with **explicit section headers**. Do not put the decisive constraint at position 60 percent of a 100K-token prompt.

**Tool-result bloat:** tool results add tokens that often exceed their relevance. An order lookup returns 40+ fields when 5 fields matter. **Trim verbose tool outputs to return-relevant fields before they enter context.** In multi-agent systems, do this trim in the upstream subagents. Subagents that serve limited-context consumers downstream must return **structured data**. Return key facts, citations, and relevance scores. Do not return verbose content and reasoning chains. Also include metadata (dates, source locations, methodological context) so synthesis stays accurate.

**Conversational coherence basics:** pass the complete conversation history on subsequent API requests. The API is stateless. If "the agent forgot the last turn," the client usually does not resend the history.

### 2. Escalation and ambiguity resolution (task 5.2)

**Legitimate escalation triggers (the guide's list):**

1. The customer **explicitly requests a human**. Escalate **immediately**. Do not investigate first. If the issue is simple, you may acknowledge frustration and *offer* the fix. Escalate if they repeat the request.
2. **Policy exceptions or gaps.** The policy is ambiguous or silent on the specific request. Example: competitor price-matching when policy only covers own-site adjustments. Note: this is about *policy coverage*, not case difficulty.
3. **Inability to make meaningful progress.**

**Unreliable escalation signals (tested as wrong answers):**

- **Self-reported confidence scores.** These scores are poorly calibrated. The agent is confidently wrong on the hard cases (official sample Q3's wrong option B).
- **Sentiment analysis.** Frustration does not correlate with case complexity (Q3's wrong option D).
- The first proportionate fix for bad escalation calibration is **explicit escalation criteria with few-shot examples in the system prompt**. The examples show escalate-vs-resolve (Q3's correct A). Apply this fix before any classifier infrastructure.

**Ambiguity resolution:** when a lookup returns **multiple customer matches**, instruct the agent to **request additional identifiers**. Never select by heuristic (probably the most recent account).

### 3. Error propagation across multi-agent systems (task 5.3)

**Structured error context** lets a coordinator make intelligent recovery decisions. On failure, a subagent returns **failure type, the attempted query, any partial results, and potential alternative approaches** (official sample Q8's correct A).

**The two anti-patterns (both named by the guide):**

- **Silent suppression.** You return empty results that you mark as success. This hides the failure. Incomplete output can then appear as complete.
- **Terminating the whole workflow on a single failure.** This stops recovery strategies that can still succeed. Those strategies include retry-with-modified-query, alternative source, and proceed-with-partial.

**Access failure vs valid empty result:** a timeout ("could not reach the source") needs a retry decision. A successful query with no matches is a real answer. These two cases are different. Error reporting must distinguish them. If you do not distinguish them, the coordinator retries empty results and accepts outages.

**Local recovery first:** subagents handle **transient** failures themselves (retry with backoff). They propagate only errors they **cannot resolve locally**. The propagated error includes what they attempted and any partial results. Generic statuses ("search unavailable") hide the context that recovery needs.

**Coverage annotation:** when synthesis proceeds despite gaps, the output must **annotate which findings are well-supported**. It must also annotate which topic areas have gaps due to unavailable sources. Degraded coverage must be visible. Do not hide it.

### 4. Context in large-codebase exploration (task 5.4)

**Degradation symptoms:** in extended sessions the model starts to give **inconsistent answers**. It refers to "typical patterns" instead of the **specific classes from earlier in the session**. Generic-knowledge fallback is the sign that the session diluted or evicted the discovered facts.

**Countermeasures (all four are exam-listed skills):**

1. **Scratchpad files.** The agent maintains a file of key findings. It **references that file for subsequent questions**. This stores knowledge across context boundaries. Files are lossless. Context is not.
2. **Subagent delegation.** Spawn subagents for verbose investigations ("find all test files," "trace refund-flow dependencies"). Exploration output stays in the subagent's context. The main agent keeps high-level coordination clean.
3. **Phase summaries.** Summarize key findings from one exploration phase before you spawn the next phase's subagents. **Inject the summary into their initial context**.
4. **`/compact`.** Reduce context during extended sessions when it fills with verbose discovery output. In Claude Code, this pairs with directed compaction — see file 05.

**Crash recovery with manifests:** use structured state persistence. Each agent **exports its state to a known location**. On resume, the coordinator **loads a manifest and injects it into agent prompts**. A crash then loses only the last part of one phase, not the whole run. This is the same files-outrank-context principle as scratchpads. You apply it to whole-system recovery.

### 5. Human review workflows and confidence calibration (task 5.5)

**Aggregate metrics mask segment failure:** "97% overall accuracy" can hide 60% accuracy on one document type or one field. **Validate accuracy by document type and field segment before you automate** high-confidence extractions.

**Stratified random sampling:** you still apply **ongoing random sampling per stratum** (document type × field) even to extractions the model marks high-confidence. This measures true error rates. It also catches **novel error patterns**. Without this sampling, drift stays invisible until an incident.

**Calibrated field-level confidence:** have the model output **per-field confidence scores**. Then **calibrate review thresholds on labeled validation sets**. Raw self-reported confidence is not trustworthy (same lesson as 5.2). Calibration against labels is what makes the scores usable for routing.

**Routing rule:** send low-confidence extractions and **ambiguous/contradictory source documents** to human review. Put limited reviewer capacity where error probability is genuinely highest.

### 6. Provenance and uncertainty in multi-source synthesis (task 5.6)

**Summarization destroys attribution:** if you compress findings without **claim–source mappings**, the final report cannot say which source supports which claim. Fix: subagents output **structured claim–source mappings**. Each mapping holds the claim, the source URL or document name, and the relevant excerpt. Downstream agents **preserve and merge those mappings through synthesis**. Provenance is a data contract, not a prose habit.

**Conflicting statistics from credible sources:** **annotate the conflict with source attribution**. Never select one value arbitrarily. Never average. Document analysis completes **with conflicting values included and explicitly annotated**. The coordinator then decides reconciliation before synthesis.

**Temporal metadata:** require **publication/collection dates in structured outputs**. Then a 2024 figure and a 2026 figure read as a time series, not a contradiction.

**Report structure:** use explicit sections that distinguish **well-established findings from contested ones**. Preserve original source characterizations and methodological context. Render content by type. Use **tables for financial data, prose for news, and structured lists for technical findings**. Do not flatten everything to uniform prose.

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
| Summarize everything to save tokens | Put numbers, dates, and IDs in a lossless case-facts layer first |
| Key constraint buried mid-prompt | Place it at the beginning or end. Use section headers (lost-in-the-middle) |
| Full 40-field tool payloads in history | Trim to relevant fields upstream |
| Route escalation on sentiment or self-confidence | Use explicit criteria plus few-shots. Calibrate confidence on labels before you trust it |
| Investigate first when the customer demanded a human | Escalate immediately |
| "Search unavailable" as the error message | Use structured error context (type, attempt, partials, alternatives) |
| Empty result treated as failure (or failure as empty success) | Distinguish access failure from valid empty result |
| One subagent timeout stops the run | Use local recovery, then partial-results propagation plus coverage notes |
| Trust "97% accurate" and automate | Segment by doc type × field. Use stratified sampling of high-confidence strata |
| Average or pick between conflicting source stats | Annotate both with sources. The coordinator reconciles |
| Dates dropped from findings | Require publication/collection dates in structured outputs |
| Session cites "typical patterns" not discovered classes | Context degradation → scratchpad + subagent delegation + `/compact` |

---

## Self-check Q&A

**Q1.** What kinds of information does progressive summarization destroy first?
**A.** Exact numbers, percentages, dates, order numbers, and customer-stated expectations. Summaries condense these into vague prose.

**Q2.** Define the case-facts pattern.
**A.** Extract transactional facts into a structured block. Include the block in every prompt outside summarized history. This is a lossless layer beneath lossy summaries.

**Q3.** What is lost-in-the-middle and two mitigations?
**A.** Models process the beginning and end of long inputs with high reliability. They omit content from the middle. Put key-findings summaries first. Use explicit section headers.

**Q4.** An order lookup returns 40+ fields, 5 relevant. What do you do and when?
**A.** Trim to return-relevant fields *before* the result enters context.

**Q5.** What should upstream subagents return when downstream context is tight?
**A.** Structured data: key facts, citations, relevance scores, plus dates and source metadata. Do not return verbose content and reasoning chains.

**Q6.** The three legitimate escalation triggers?
**A.** Explicit customer request for a human. Policy exceptions/gaps. Inability to make meaningful progress.

**Q7.** Customer demands a human on a trivially fixable issue. Sequence?
**A.** Honor the request immediately. At most, acknowledge and offer the fix once. Escalate if they repeat the request. Never investigate first against their request.

**Q8.** Why are sentiment and self-reported confidence bad escalation routers?
**A.** Neither correlates with case complexity. Confidence is uncalibrated. The agent is confidently wrong on hard cases (sample Q3).

**Q9.** Tool returns three customers for "John Smith." Correct behavior?
**A.** Request additional identifiers. Never select by heuristic.

**Q10.** A search subagent times out. What does the coordinator need to recover intelligently?
**A.** Structured error context: failure type, attempted query, partial results, alternative approaches (sample Q8).

**Q11.** Name the two error-handling anti-patterns and the harm of each.
**A.** Silent suppression (failure presented as empty success → incomplete output looks complete). Whole-workflow termination on one failure (stops viable recovery).

**Q12.** Access failure vs valid empty result — why distinguish?
**A.** One needs a retry decision. The other is a real answer. If you mix them, you get wrong retries and silently accepted outages.

**Q13.** When does an error leave a subagent?
**A.** Only when local recovery (for example retry for transients) fails. Propagate the error with what the subagent attempted and any partial results.

**Q14.** Synthesis ran with one source down. What must the report contain?
**A.** Coverage annotations: which findings are well-supported, and which topic areas have gaps from unavailable sources.

**Q15.** Symptom that a long exploration session's context has degraded?
**A.** Inconsistent answers. The model refers to "typical patterns" instead of the specific classes from earlier in the session.

**Q16.** Four countermeasures for large-codebase context management?
**A.** Scratchpad files. Subagent delegation for verbose investigation. Phase summaries injected into next-phase context. `/compact`.

**Q17.** Design crash recovery for a multi-agent run.
**A.** Each agent exports structured state to a known location. On resume the coordinator loads the manifest and injects it into agent prompts.

**Q18.** Why is 97% overall accuracy insufficient to automate?
**A.** Aggregates mask segment failure. Validate accuracy per document type and field before you reduce human review.

**Q19.** Role of stratified random sampling in a review workflow?
**A.** Ongoing error-rate measurement of high-confidence extractions per stratum. This catches novel error patterns and drift.

**Q20.** How does model confidence become usable for review routing?
**A.** The model outputs field-level scores. You calibrate thresholds against labeled validation sets. Do not trust raw self-report.

**Q21.** Two credible sources give different market sizes. Handling?
**A.** Include both, annotated with source attribution. The coordinator decides reconciliation. Never use arbitrary selection or averaging.

**Q22.** Why require publication dates in structured outputs?
**A.** So temporal differences read as change over time rather than contradiction.

**Q23.** What keeps attribution alive through multi-agent synthesis?
**A.** Structured claim–source mappings (claim, source, excerpt) that every downstream agent preserves and merges.

**Q24.** How should a synthesis report treat contested findings?
**A.** Separate explicit sections for well-established vs contested. Keep original source characterizations and methodology context.

**Q25.** Financial data, news, and technical findings in one report — format?
**A.** Tables, prose, and structured lists respectively. Render per content type. Do not flatten.

---

## Pre-exam checklist

- [ ] I can specify a case-facts block and say what summarization destroys.
- [ ] I can state lost-in-the-middle and its two mitigations.
- [ ] I trim tool outputs upstream. I know what subagents should return downstream.
- [ ] I can recite the three escalation triggers and the two unreliable signals.
- [ ] I can spec structured error context and both error anti-patterns.
- [ ] I distinguish access failures from empty results immediately.
- [ ] I can name all four codebase-context countermeasures plus manifest recovery.
- [ ] As a first habit, I segment accuracy and stratify sampling before I automate.
- [ ] I preserve claim–source mappings and annotate conflicts with dates.

---

## Glossary

| Term | Meaning |
| --- | --- |
| Case-facts block | Structured lossless facts layer included in every prompt outside history |
| Progressive summarization | Repeated compression of history. It destroys exact values first |
| Lost in the middle | Position effect: the middle of long inputs gets omitted |
| Tool-result trimming | Reduce payloads to relevant fields before they enter context |
| Escalation trigger | Human request, policy gap, or no-progress — not sentiment or confidence |
| Policy gap | Request the policy is silent or ambiguous on → escalate |
| Structured error context | Failure type + attempted query + partials + alternatives |
| Silent suppression | Failure returned as empty success — anti-pattern |
| Local recovery | Subagent retries transients before it propagates the error |
| Coverage annotation | Mark well-supported vs gap areas in synthesis output |
| Scratchpad file | Agent-maintained findings file that stores knowledge across context limits |
| Manifest | Coordinator-loaded structured state export that enables crash resume |
| Stratified sampling | Random QA sampling per doc-type/field stratum, including high-confidence |
| Calibration | Map model confidence to real error rates via labeled sets |
| Claim–source mapping | Structured claim + source + excerpt preserved through synthesis |
| Temporal annotation | Publication/collection dates attached to findings |

---

*Companion files: 08 (Domain 1 orchestration — handoff structure, subagent mechanics), 05 (+ supplement) for /compact and scratchpad practice in Claude Code.*

