---
title: Troubleshooting & Optimization
---

# Domain 07 — Troubleshooting & Optimization
## Maps to official CCAO-F **Troubleshooting and Optimization** (~10%, ~6 questions)

> **Dedup note (2026-08-23):** Rewritten as a single primary-study copy. Earlier builds repeated the same drill blocks ~7×; duplicates removed and content deepened to the Domain 03 standard.

## Disclaimer

Original CCAO-F study notes for non-developers using claude.ai (Projects, Artifacts, Skills, Connectors, Research). Grounded in public Anthropic Help Center & product docs, public Claude Academy (Claude 101, AI Fluency 4D), and the published CCAO-F blueprint. Independent; not affiliated with Anthropic. Verify live product details on support.claude.com.

---

## Overview

Official blueprint verbs: **identify, diagnose, and resolve** issues with underperforming prompts or poor outputs; **adjust** approach based on feedback and results; **optimize** workflows for efficiency and effectiveness. This is the integration domain: decide whether the break is **platform, inputs, configuration, prompt, context, model, evaluation, or governance** — and fix the right layer without wasting usage. Most stems reward the cheapest correct fix, not the most heroic one.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Diagnose poor outputs | identify, isolate | The diagnostic ladder; layer naming |
| Adjust from feedback | adjust, adapt | Miss taxonomy → targeted system updates |
| Optimize workflows | optimize, streamline | Levers: model, effort, feature, context, structure |

---

## Deep notes

### 1. The diagnostic ladder (run it in order)

1. **Success criteria** — was the ask vague? Maybe nothing is broken but the brief (Domain 02).
2. **Inputs** — wrong/missing files, unauthenticated connector, web search off, OCR-junk scans, capability toggles.
3. **Configuration** — stale or conflicting knowledge; contradictory instructions; wrong Project; stale **Memory entry** feeding wrong context (edit/delete it — see `05-…` §Memory).
4. **Prompt** — Description gaps: no criteria, no schema, mixed task types.
5. **Context** — thread drift; summarize and restart.
6. **Model/effort** — under- or over-powered tier; tune effort before jumping tiers.
7. **Evaluation gates** — quality "problems" that are really shipped-without-review problems (Domain 03).
8. **Limits/plan** — usage limits, Free caps, feature availability.

Discipline: change **one variable at a time**; note what fixed it so it becomes a pattern, not a lucky rerun. Two failed tightened prompt iterations = wrong layer; move up or down the ladder.

### 2. Worked cases

- **"Claude gives the wrong refund answer."** Which policy file is in the Project? Ask it to quote the section. File stale → update knowledge. File right but ignored → instructions: "Prefer `Refund-Policy-v4`; if not in knowledge, say Not found." Only then consider model escalation.
- **"Research is useless."** Web search on? Paid plan? Question scoped? Connectors authed if internal data expected? Narrow the scope before blaming the feature.
- **"The numbers in the summary are wrong."** Large-table arithmetic in plain chat is the wrong tool — rerun through **code execution** so the math is computed, not pattern-matched; then compare against source.
- **"Claude keeps bringing up my old project."** Stale memory entry — Settings → Memory, edit/delete; use separate Projects to scope context; incognito for one-offs.
- **"The team says Claude is random."** Shared Project without owner; conflicting guides; no schema. Governance + configuration fix — not a bigger model.
- **"Claude refused my legitimate request."** State the benign, authorized context plainly and rephrase within policy; refusals on disallowed content are correct behavior, and jailbreak options are always exam-wrong (Domain 06).

### 3. Optimization levers (effectiveness × efficiency)

Model tier; effort; feature choice (Research vs chat vs code execution); context size; Project-persisted context vs re-pasting; Skill reuse; schemas to cut revision cycles; batching similar tasks; pruning noisy knowledge; capability toggles; connector scope. Optimize **both** quality and cost/time: cutting review on L4 work is false efficiency; running Opus on ticket triage is false quality.

Scoreboard: median revisions-to-ship; % factual misses; usage per workflow; time-to-first-draft. Improve the worst metric with the **cheapest lever** — that phrase decides exam options.

### 4. Feedback → system updates (miss taxonomy)

Tag every miss, then fix the system that caused it: **factual** → knowledge/citations/code execution for math; **format** → instructions/schema; **tone** → examples in Project/Skill; **policy** → governance + instructions; **incomplete** → checklist/decomposition; **auth/access** → IT enablement; **wrong remembered context** → memory curation. Closing the loop is Diligence; the anti-pattern is coaching users to compensate for a broken configuration.

### 5. Bridge summary across all domains

01 pick tools/models; 02 describe tasks; 03 validate outputs; 04 design workflows; 05 configure knowledge/memory; 06 govern risk; 07 diagnose and optimize. Real failures are usually multi-domain — name the **primary** layer in your answer, fix it, and re-measure.

### 6. 90-second diagnosis drills

Work each in ~90 seconds: name the layer, then the cheapest fix. Answers follow.

- **Drill A:** "Claude used to nail our weekly report; since Marta added her files it mixes two template styles." → Configuration: conflicting templates in knowledge; remove one or set priority in instructions.
- **Drill B:** "Research mode grays out for one analyst." → Platform/plan: their seat/plan or capability toggle — not a prompt problem.
- **Drill C:** "The extraction is perfect on 10 rows, garbage on 5,000." → Tool/context: move from chat-window pasting to uploaded file + code execution.
- **Drill D:** "Claude answered from the March price list; April's is uploaded too." → Knowledge hygiene: superseded file still present; purge it.
- **Drill E:** "Every chat, Claude assumes I'm still on the Berlin project." → Memory: stale entry; edit/delete in Settings → Memory.
- **Drill F:** "Long thread: it now ignores the banned-words list from message 3." → Context drift: summarize + restart with rules persisted in the Project.
- **Drill G:** "Quality is fine but we burn the day's limit by 2 p.m." → Optimization: tier/effort down on routine tasks; batch; persist context instead of re-pasting.
- **Drill H:** "It refuses to summarize our pen-test report." → Governance-aware rephrase: state the authorized defensive context; never jailbreak.

The meta-skill: symptom → layer → cheapest fix, said in one sentence. If your explanation needs a paragraph, you probably haven't found the layer.

### 7. Multiple-response pattern bank

Recurring wrong pairs: "switch to the top model" + "give up on the task" (prestige + defeatism — the ladder is the middle); "add three more paragraphs of instructions" + "delete all instructions" (wordcount as medicine); "remove the review step to speed up" + "review every trivial output" (false efficiency + disproportion); "restart every chat hourly" + "never restart" (ritual + sunk cost). Correct combinations pair a **diagnostic** move (check auth/toggles, quote-the-file test, isolate one variable) with a **structural** fix (schema, knowledge purge, memory edit, tier right-sizing).

---

## Decision trees

| Symptom | Likely layer → action |
|---|---|
| Wrong facts "from" files | Knowledge/pointing → name file, purge superseded versions |
| Ignores required format | Prompt → schema/Performance constraints |
| Great early, degrades late in thread | Context → summarize + restart |
| Wrong arithmetic on data | Tool → code execution, verify vs source |
| Slow/expensive workflow | Optimization → lighter model/effort, less Research, templates |
| Feature "doesn't work" | Inputs/platform → plan, capability toggle, auth, web search |
| Wrong personal context recalled | Memory → edit/delete entry; Project scoping |
| Refusal on allowed work | Governance-aware rephrase; never jailbreak |
| Chronic misses across users | Workflow/config redesign — not a bigger model |
| Errors reaching stakeholders | Evaluation gates missing → restore Domain 03 SOP |

---

## Exam traps

1. "Escalate to the biggest model" as the universal fix
2. Ignoring auth, web search, and capability toggles before deep diagnosis
3. Fixing symptoms with ever-longer prompts (wrong layer)
4. "Optimizing" by removing human review from high-risk steps
5. Treating safety refusals as outages
6. Never restarting drifted threads (sunk-cost threading)
7. Hoarding conflicting knowledge versions
8. Changing five variables at once, learning nothing
9. Blaming the model for a stale memory entry or stale PDF
10. Optimizing an unmeasured workflow

---

## Practice Q&A (16) — scenario stems with answers and rationales

**Q1.** Claude answers refund questions from general knowledge despite an uploaded policy. First two moves?
**A:** Name the file in the prompt and require quoting; check knowledge for superseded/duplicate versions. Pointing + hygiene beat re-asking louder.

**Q2.** Select TWO optimizations for a high-volume extraction workflow burning limits.
**A:** Drop to Haiku (or lower effort); persist the schema and rules in a Project instead of re-pasting. (Removing review is the trap option.)

**Q3.** Research returns nothing. Fastest diagnostic?
**A:** Check web search is enabled and the plan supports Research — capability toggles before feature blame.

**Q4.** A 200-message thread now contradicts itself. Fix?
**A:** Handoff summary (decisions, constraints, open items) → fresh chat. Drifted context is replaced, not argued with.

**Q5.** Monthly variance memo has subtle arithmetic errors. Layer and fix?
**A:** Tool layer: route the math through code execution on the uploaded data, then verify totals against source. Chat prose is not a calculator.

**Q6.** Outputs ignore the required five-section format intermittently. Fix?
**A:** Put a hard schema in Project instructions (sections, order, "omit nothing; write N/A if empty"). Format misses are Performance-constraint gaps, not model failures.

**Q7.** Drive connector "sees nothing" for one user but works for others. Diagnosis?
**A:** That user hasn't authenticated the connector — admin enablement is org-wide, auth is per-user.

**Q8.** Claude keeps referencing a client the associate stopped serving last year. Fix?
**A:** Memory curation: delete/edit the stale entry in Settings → Memory; keep current clients in their own Projects so scoping does the work.

**Q9.** Chronic tone misses across the whole team's outputs. Cheapest durable fix?
**A:** Add gold tone examples (and a counterexample) to the shared Project/Skill — one config change fixes every user, unlike per-chat coaching.

**Q10.** Select TWO checks *before* concluding "the model is too weak" on an analysis task.
**A:** Was a rubric/criteria provided (prompt layer)? Is the knowledge current and conflict-free (config layer)? Escalate tier only after both pass.

**Q11.** The team hits usage limits mid-afternoon. Select TWO adjustments.
**A:** Route routine tasks to lighter models/lower effort; cut Research use where a quoted file or single search suffices. (Also batching similar tasks; upgrading is a business call, not a reflex.)

**Q12.** A stakeholder demands "faster outputs — just buy the top model." Better lever set?
**A:** Templates/Skills for recurring structure, persisted Project context, right-sized model per task — speed usually lives in workflow structure, not tier.

**Q13.** Claude refuses to analyze the company's own phishing simulation results. Move?
**A:** Restate the authorized, defensive context plainly and rephrase within policy — legitimate security work with consent is fine to clarify; jailbreak framing is auto-wrong.

**Q14.** Two conflicting SOP versions live in knowledge; answers alternate between them. Fix?
**A:** Delete the superseded version (or add explicit priority in instructions). Retrieval confusion is configuration debt.

**Q15.** How do you prove an optimization worked?
**A:** Before/after on the scoreboard: revisions-to-ship, factual-miss %, usage per workflow, time-to-first-draft. Unmeasured optimization is anecdote.

**Q16.** After an incident, someone proposes removing the human review step to "streamline." Verdict?
**A:** Reject: review on consequential outputs is effectiveness, not overhead — restore the gate and optimize elsewhere (lighter model, schema, batching). False efficiency is the trap.

---

## Quick review checklist (pre-exam)

- [ ] Ladder order: criteria → inputs → config (incl. memory) → prompt → context → model/effort → gates → limits
- [ ] One variable at a time; two failed tightened iterations = change layer
- [ ] Wrong math → code execution; wrong facts → knowledge; wrong format → schema
- [ ] Stale memory entries are configuration bugs — curate them
- [ ] Cheapest lever on the worst measured metric
- [ ] Miss taxonomy → fix the causing system
- [ ] Never optimize away human gates on high-risk work
- [ ] Refusal ≠ outage; clarify within policy

---

## Glossary

| Term | Meaning |
|---|---|
| **Diagnostic ladder** | Ordered layer checks from criteria to limits |
| **Thread drift** | Long-conversation degradation; fixed by summarize + restart |
| **Miss taxonomy** | Factual / format / tone / policy / incomplete / auth classification |
| **Cheapest lever** | Smallest change that moves the worst metric |
| **Configuration drift** | Stale knowledge, instructions, or memory diverging from reality |
| **False efficiency** | Speed gained by deleting necessary review |
| **Effort control** | Depth dial inside a model tier — tune before switching tiers |
| **Scoreboard** | Revisions-to-ship, miss %, usage, time-to-first-draft |
