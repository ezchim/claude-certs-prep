# Domain 07 — Troubleshooting & Optimization
## Maps to official CCAO-F **Troubleshooting and Optimization** (~10%, ~6 questions)

> **Note:** This edition follows the ASD-STE100 Simplified Technical English writing rules. Simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Projects, Artifacts, Skills, Connectors, Research, Memory, effort) are exceptions and stay as written.

> **Dedup note (2026-08-23):** This file is one primary-study copy. Earlier builds repeated the same drill blocks about 7 times. Those duplicates are gone. The content now matches the Domain 03 depth.

## Disclaimer

These notes are original CCAO-F study notes. They are for people who are not developers. They use claude.ai (Projects, Artifacts, Skills, Connectors, Research). The notes use public Anthropic Help Center and product docs. They also use public Claude Academy (Claude 101, AI Fluency 4D) and the published CCAO-F blueprint. The notes are independent. They are not affiliated with Anthropic. Check live product details on support.claude.com.

---

## Overview

Official blueprint verbs: **identify, diagnose, and resolve** issues with underperforming prompts or poor outputs. **Adjust** the approach based on feedback and results. **Optimize** workflows for efficiency and effectiveness. This is the integration domain. Decide whether the fault is in the platform, inputs, configuration, prompt, context, model, evaluation, or governance. Then fix the right layer. Do not waste usage. Most stems reward the cheapest correct fix. They do not reward the most dramatic one.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Diagnose poor outputs | identify, isolate | The diagnostic ladder. Layer naming |
| Adjust from feedback | adjust, adapt | Miss taxonomy → targeted system updates |
| Optimize workflows | optimize, streamline | Levers: model, effort, feature, context, structure |

---

## Deep notes

### 1. The diagnostic ladder (run it in order)

1. **Success criteria** — was the request vague? Maybe nothing is broken. The brief is the problem (Domain 02).
2. **Inputs** — wrong or missing files, unauthenticated connector, web search off, OCR-junk scans, capability toggles.
3. **Configuration** — stale or conflicting knowledge. Contradictory instructions. Wrong Project. Stale **Memory entry** that feeds wrong context (edit or delete it — see `05-…` §Memory).
4. **Prompt** — Description gaps: no criteria, no schema, mixed task types.
5. **Context** — thread drift. Summarize and restart.
6. **Model/effort** — under-powered or over-powered tier. Tune effort before you change tiers.
7. **Evaluation gates** — quality "problems" that are really shipped-without-review problems (Domain 03).
8. **Limits/plan** — usage limits, Free caps, feature availability.

Discipline: change **one variable at a time**. Record what fixed it so it becomes a pattern. Do not treat a lucky rerun as a pattern. Two failed tightened prompt iterations mean the wrong layer. Move up or down the ladder.

### 2. Worked cases

- **"Claude gives the wrong refund answer."** Which policy file is in the Project? Ask it to quote the section. If the file is stale, update knowledge. If the file is right but ignored, add instructions: "Prefer `Refund-Policy-v4`. If not in knowledge, say Not found." Only then consider model escalation.
- **"Research is useless."** Is web search on? Is the plan paid? Is the question scoped? Are connectors authenticated if you expect internal data? Narrow the scope before you blame the feature.
- **"The numbers in the summary are wrong."** Large-table arithmetic in plain chat is the wrong tool. Rerun through **code execution** so the math is computed, not pattern-matched. Then compare against the source.
- **"Claude keeps bringing up my old project."** Stale memory entry. Go to Settings → Memory. Edit or delete the entry. Use separate Projects to scope context. Use incognito for single-use questions.
- **"The team says Claude is random."** Shared Project without an owner. Conflicting guides. No schema. This is a governance and configuration fix. It is not a bigger model.
- **"Claude refused my legitimate request."** State the benign, authorized context plainly. Rephrase within policy. Refusals on disallowed content are correct behavior. Jailbreak options are always exam-wrong (Domain 06).

### 3. Optimization levers (effectiveness × efficiency)

Model tier. Effort. Feature choice (Research vs chat vs code execution). Context size. Project-persisted context vs re-pasting. Skill reuse. Schemas to cut revision cycles. Batching similar tasks. Pruning noisy knowledge. Capability toggles. Connector scope. Optimize **both** quality and cost/time. If you cut review on L4 work, that is false efficiency. If you run Opus on ticket triage, that is false quality.

Scoreboard: median revisions-to-ship. % factual misses. Usage per workflow. Time-to-first-draft. Improve the worst metric with the **cheapest lever**. That phrase decides exam options.

### 4. Feedback → system updates (miss taxonomy)

Tag every miss. Then fix the system that caused it. **Factual** → knowledge, citations, or code execution for math. **Format** → instructions or schema. **Tone** → examples in Project or Skill. **Policy** → governance plus instructions. **Incomplete** → checklist or decomposition. **Auth/access** → IT enablement. **Wrong remembered context** → memory curation. When you fix the causing system, you practice Diligence. The anti-pattern is coaching users so they compensate for a broken configuration.

### 5. Bridge summary across all domains

01 pick tools and models. 02 describe tasks. 03 validate outputs. 04 design workflows. 05 configure knowledge and memory. 06 govern risk. 07 diagnose and optimize. Real failures are usually multi-domain. Name the **primary** layer in your answer. Fix it. Then measure again.

### 6. 90-second diagnosis drills

Work each in about 90 seconds: name the layer, then the cheapest fix. Answers follow.

- **Drill A:** "Claude did our weekly report well before. Since Marta added her files, it mixes two template styles." → Configuration: conflicting templates in knowledge. Remove one, or set priority in instructions.
- **Drill B:** "Research mode grays out for one analyst." → Platform/plan: their seat, plan, or capability toggle. This is not a prompt problem.
- **Drill C:** "The extraction is perfect on 10 rows and fails on 5,000." → Tool/context: move from chat-window pasting to an uploaded file plus code execution.
- **Drill D:** "Claude answered from the March price list. April's is uploaded too." → Knowledge hygiene: a superseded file is still present. Purge it.
- **Drill E:** "Every chat, Claude assumes I am still on the Berlin project." → Memory: stale entry. Edit or delete it in Settings → Memory.
- **Drill F:** "Long thread: it now ignores the banned-words list from message 3." → Context drift: summarize and restart, with rules persisted in the Project.
- **Drill G:** "Quality is fine but we use up the day's limit by 2 p.m." → Optimization: lower the tier or effort on routine tasks. Batch. Persist context instead of re-pasting.
- **Drill H:** "It refuses to summarize our pen-test report." → Governance-aware rephrase: state the authorized defensive context. Never jailbreak.

The core skill: symptom → layer → cheapest fix, in one sentence. If your explanation needs a paragraph, you probably have not found the layer.

### 7. Multiple-response pattern bank

Recurring wrong pairs: "switch to the top model" plus "give up on the task" (status plus giving up — the ladder is the middle). Also "add three more paragraphs of instructions" plus "delete all instructions" (word count as a cure). Also "remove the review step to speed up" plus "review every trivial output" (false efficiency plus disproportion). Also "restart every chat hourly" plus "never restart" (ritual plus sunk cost). Correct combinations pair a **diagnostic** move with a **structural** fix. Diagnostic moves: check auth and toggles, quote-the-file test, isolate one variable. Structural fixes: schema, knowledge purge, memory edit, correct tier size.

---

## Decision trees

| Symptom | Likely layer → action |
|---|---|
| Wrong facts "from" files | Knowledge/pointing → name the file, purge superseded versions |
| Ignores required format | Prompt → schema/Performance constraints |
| Great early, degrades late in thread | Context → summarize + restart |
| Wrong arithmetic on data | Tool → code execution, verify vs source |
| Slow/expensive workflow | Optimization → lighter model/effort, less Research, templates |
| Feature "does not work" | Inputs/platform → plan, capability toggle, auth, web search |
| Wrong personal context recalled | Memory → edit or delete the entry. Project scoping |
| Refusal on allowed work | Governance-aware rephrase. Never jailbreak |
| Chronic misses across users | Workflow/config redesign — not a bigger model |
| Errors reaching stakeholders | Evaluation gates missing → restore Domain 03 SOP |

---

## Exam traps

1. "Escalate to the model with the most capability" as the universal fix
2. You ignore auth, web search, and capability toggles before deep diagnosis
3. You fix symptoms with ever-longer prompts (wrong layer)
4. "Optimizing" by removing human review from high-risk steps
5. You treat safety refusals as outages
6. You never restart drifted threads (sunk-cost threading)
7. You keep conflicting knowledge versions
8. You change five variables at once and learn nothing
9. You blame the model for a stale memory entry or a stale PDF
10. You optimize an unmeasured workflow

---

## Practice Q&A (16) — scenario stems with answers and rationales

**Q1.** Claude answers refund questions from general knowledge despite an uploaded policy. First two moves?
**A:** Name the file in the prompt and require quoting. Check knowledge for superseded or duplicate versions. Pointing plus hygiene is better than asking again with more force.

**Q2.** Select TWO optimizations for a high-volume extraction workflow burning limits.
**A:** Drop to Haiku (or lower effort). Persist the schema and rules in a Project instead of re-pasting. (Removing review is the trap option.)

**Q3.** Research returns nothing. Fastest diagnostic?
**A:** Check that web search is enabled and that the plan supports Research. Capability toggles come before feature blame.

**Q4.** A 200-message thread now contradicts itself. Fix?
**A:** Write a handoff summary (decisions, constraints, open items). Start a fresh chat. Replace drifted context. Do not argue with it.

**Q5.** Monthly variance memo has subtle arithmetic errors. Layer and fix?
**A:** Tool layer: route the math through code execution on the uploaded data. Then verify totals against the source. Chat prose is not a calculator.

**Q6.** Outputs ignore the required five-section format intermittently. Fix?
**A:** Put a hard schema in Project instructions (sections, order, "omit nothing. Write N/A if empty"). Format misses are Performance-constraint gaps, not model failures.

**Q7.** Drive connector "sees nothing" for one user but works for others. Diagnosis?
**A:** That user has not authenticated the connector. Admin enablement is org-wide. Auth is per-user.

**Q8.** Claude keeps referencing a client the associate stopped serving last year. Fix?
**A:** Memory curation: delete or edit the stale entry in Settings → Memory. Keep current clients in their own Projects so scoping does the work.

**Q9.** Chronic tone misses across the whole team's outputs. Cheapest durable fix?
**A:** Add gold tone examples (and a counterexample) to the shared Project or Skill. One config change fixes every user. Per-chat coaching does not.

**Q10.** Select TWO checks *before* concluding "the model is too weak" on an analysis task.
**A:** Was a rubric or criteria provided (prompt layer)? Is the knowledge current and conflict-free (config layer)? Escalate the tier only after both pass.

**Q11.** The team hits usage limits mid-afternoon. Select TWO adjustments.
**A:** Route routine tasks to lighter models or lower effort. Cut Research use where a quoted file or a single search is enough. (Also batch similar tasks. An upgrade is a business call, not a reflex.)

**Q12.** A stakeholder demands "faster outputs — just buy the top model." Better lever set?
**A:** Templates and Skills for recurring structure, persisted Project context, and a right-sized model per task. Speed usually lives in workflow structure, not in tier.

**Q13.** Claude refuses to analyze the company's own phishing simulation results. Move?
**A:** Restate the authorized, defensive context plainly and rephrase within policy. Legitimate security work with consent is fine to clarify. Jailbreak framing is always wrong on the exam.

**Q14.** Two conflicting SOP versions live in knowledge. Answers alternate between them. Fix?
**A:** Delete the superseded version (or add explicit priority in instructions). Retrieval confusion is configuration debt.

**Q15.** How do you prove an optimization worked?
**A:** Before/after on the scoreboard: revisions-to-ship, factual-miss %, usage per workflow, time-to-first-draft. Unmeasured optimization is a story, not proof.

**Q16.** After an incident, someone proposes removing the human review step to "streamline." Verdict?
**A:** Reject: review on consequential outputs is effectiveness, not overhead. Restore the gate and optimize elsewhere (lighter model, schema, batching). False efficiency is the trap.

---

## Quick review checklist (pre-exam)

- [ ] Ladder order: criteria → inputs → config (incl. memory) → prompt → context → model/effort → gates → limits
- [ ] One variable at a time. Two failed tightened iterations = change layer
- [ ] Wrong math → code execution. Wrong facts → knowledge. Wrong format → schema
- [ ] Stale memory entries are configuration bugs — curate them
- [ ] Cheapest lever on the worst measured metric
- [ ] Miss taxonomy → fix the causing system
- [ ] Never optimize away human gates on high-risk work
- [ ] Refusal ≠ outage. Clarify within policy

---

## Glossary

| Term | Meaning |
|---|---|
| **Diagnostic ladder** | Ordered layer checks from criteria to limits |
| **Thread drift** | Long-conversation degradation. Fix it with summarize + restart. |
| **Miss taxonomy** | Factual / format / tone / policy / incomplete / auth classification |
| **Cheapest lever** | Smallest change that moves the worst metric |
| **Configuration drift** | Stale knowledge, instructions, or memory that diverges from reality |
| **False efficiency** | Speed you gain when you delete necessary review |
|**Effort control**. |Depth setting inside a model tier — tune before you switch tiers. |
| **Scoreboard** | Revisions-to-ship, miss %, usage, time-to-first-draft |
