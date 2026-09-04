---
title: Prompting & Task Execution
---

# Domain 02 — Prompting & Task Execution
## Maps to official CCAO-F **Prompting and Task Execution** (~14%, ~8 questions)

> **Dedup note (2026-08-23):** Rewritten as a single primary-study copy. Earlier builds repeated the same drill blocks ~10×; duplicates removed and content deepened to the Domain 03 standard.

## Disclaimer

Original CCAO-F study notes for non-developers using claude.ai (Projects, Artifacts, Skills, Connectors, Research). Grounded in public Anthropic Help Center & product docs, public Claude Academy (Claude 101, AI Fluency 4D framework), and the published CCAO-F blueprint. Independent; not affiliated with Anthropic. Verify live product details on support.claude.com.

---

## Overview

Prompting on CCAO-F equals professional briefing. Official blueprint verbs: **create** effective prompts for business and technical tasks; **apply task decomposition** to structure complex requests; **iterate** prompts to improve quality; **adapt prompting strategies by task type** — the guide names *analysis, research, drafting, brainstorming*. The AI Fluency **Description** triad — Product, Process, Performance — is the spine: what the deliverable is, how to get there, and what "good" looks like.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Effective prompts | create, structure | Prompt anatomy; the Description triad |
| Task decomposition | apply, sequence | Phased prompts with checkpoints |
| Iteration | iterate, refine | Named-defect feedback; one variable at a time |
| Task-type adaptation | adapt, select | Analysis / research / drafting / brainstorming playbooks |

---

## Deep notes

### 1. Prompt anatomy

A complete brief carries: **role** (who Claude acts as), **goal** (deliverable + the decision it supports), **inputs** (files by name, pasted data, connector sources), **constraints** (length, tone, banned claims, audience), **process** (ordered steps, checkpoints), **format** (schema, sections, table), **verification** (what to check before answering). You rarely need all seven — but exam-best prompts are the ones with no *missing load-bearing* part and no internal contradiction.

**Description triad:** *Product* Description names the deliverable and audience decision it supports. *Process* Description sequences tools and checkpoints. *Performance* Description sets acceptance tests, examples, and counterexamples. Missing one creates a predictable failure: fluff (no Product), skipped steps (no Process), or stylish wrongness (no Performance).

**Stakeholder translation** is the associate skill: "make it pop" needs voice samples; "keep it legal" needs disclaimer files and banned claims; "board ready" needs Decision/Risks/Ask structure; "quick thoughts" must be labeled speculative. Stems use stakeholder slang — translate before picking an answer.

**Contradictions** ("explain like a beginner but keep undefined internal acronyms") must be resolved explicitly. Unresolved contradictions yield flaky outputs blamed on the model.

**Examples beat adjectives.** Two or three gold examples (and one counterexample) outperform paragraphs of style description. Few-shot is the highest-leverage lever a non-developer has.

**Ambiguity:** prefer *clarify* over *invent*. If proceeding, require an explicit `ASSUMPTIONS` block so wrong guesses are visible and cheap to fix.

### 2. Task decomposition

Break multi-deliverable or high-stakes work into phases with human checkpoints:

- **Outline → fill:** approve structure before prose.
- **Extract → analyze:** separate facts from judgment so errors are traceable.
- **Options → decide:** generate alternatives without commitment, then choose with criteria.
- **Draft → red-team:** produce, then adversarially attack (bridges Domain 03).

Worked patterns: ops postmortem (timeline facts → contributing factors tagged Evidence vs Hypothesis → actions only with named owners); marketing email (facts + legal must-includes → subjects + body → compliance pass against banned claims); HR FAQ (answer only from knowledge; quote section; "Not found" if absent); offsite agenda (brainstorm → cluster → pick → timed agenda → risk check); data cleanup (define schema → validate a 20-row sample via code execution → scale).

**Decomposition is also governance:** it inserts human gates before irreversible or external steps — the crossover tested in Domains 04 and 06. Decompose when: multiple deliverables, high stakes, mixed task types (never brainstorm and analyze in one breath), or when you need intermediate checkpoints. Single-shot is fine when the task is small, clear, and low-stakes — decomposing trivia wastes time and usage.

### 3. Iteration craft

Strong iteration **names the defect and the constraint to add**: "the summary invented a Q3 figure — use only numbers from `Q3-report.pdf`, mark anything else UNKNOWN" beats "make it better." Change **one major variable at a time** when debugging so you learn causality. Prefer surgical edits ("fix only the table") over full regeneration. When a prompt fails twice with tightened wording, the problem is usually inputs, feature choice, or task framing — not word choice (Domain 07's ladder).

Keep a micro-prompt library: assumptions list; FACTS vs INFERENCES; only the table; NULL if missing; strongest counterargument; meaning-preserving diff; outline only; claim-to-source map; three options without picking; compress without changing numbers.

### 4. Task-type playbooks (the named blueprint objective)

| Task type | Strategy | Signature moves |
|---|---|---|
| **Analysis** | Criteria first, evidence separated from inference | Provide a rubric; demand FACTS vs INFERENCES sections; require counterarguments and unknowns |
| **Research** | Scope + sources + citations | Define question, timeframe, source rules; require citation-support (Domain 03); Research feature for multi-source public landscape |
| **Drafting** | Must-include facts + audience + format | Supply approved facts (don't let Claude source them); voice samples; banned phrases; length caps |
| **Brainstorming** | Diverge, then converge — separately | Ask for quantity/variety without judgment first; rank with criteria in a second step; label output speculative |

Mixing modes is the classic error: asking for "creative ideas but strictly accurate to policy" in one prompt gets neither. Split the phases.

### 5. Prompting as a system (team scale)

Stable rules migrate out of prompts: recurring instructions → Project instructions; reference detail → knowledge files (pointed to by name); rich procedures → Skills. Shared Projects need priority rules for conflicts (e.g., legal > brand). Store gold examples and counterexamples in knowledge. Teach teammates to paste the ask, not their anxiety — vague urgency without specs wastes usage limits. A prompt worth repeating weekly is configuration, not typing (Domain 05).

### 6. Timed heuristics for prompt-comparison items

When four candidate prompts are shown, score each on Goal, Inputs, Constraints, Process, Format, Verification; highest complete, non-contradictory score wins. Prefer clarify over invent; prefer phased approvals over one mega-prompt for consequential transformations. API-parameter options (temperature, fine-tuning) are distractors on this product-focused exam. Before "escalate the model," ask: could a careful intern succeed from this brief? If no, fix the Description first.

### 7. Worked briefs (before → after)

**Weak:** "Summarize this customer call." → **Strong:** "From `call-2026-08-12.txt`, produce a one-page summary for the account exec deciding whether to escalate: 5 bullet facts with timestamps, customer sentiment with a supporting quote, commitments made by either side (quote exactly; UNKNOWN if unclear), and recommended next step with rationale. Do not infer facts not in the transcript."

**Weak:** "Write a LinkedIn post about our launch." → **Strong:** "Draft a LinkedIn post announcing the launch for prospective mid-market buyers. Must include: the two approved claims in `launch-facts.md` (verbatim numbers), a customer-pain opening, ≤150 words, no superlatives from `banned-phrases.md`. Match the voice of the two examples in knowledge. Give two variants: safe and bold."

**Weak:** "Is this vendor contract okay?" → **Strong:** "Phase 1: extract every clause in `vendor-msa.pdf` touching liability, termination, data handling, and auto-renewal — quote each. Phase 2 (after my confirmation): assess each extracted clause against the checklist in `contract-redlines.md`, tag OK / FLAG / ESCALATE with one-line reasons. Final legal judgment stays with counsel."

The pattern: name the file, the audience decision, the must-includes, the format, the verification rule — and phase anything consequential.

### 8. Multiple-response elimination bank

Select-TWO/THREE prompting items usually pair one correct craft move with distractors. Practice eliminating: "make it better" (no defect named); "add temperature=0" (API knob, wrong track); "use the biggest model" (Domain 01 prestige bias); "let Claude find the statistics" (sourcing load-bearing facts); "combine brainstorm and compliance review into one prompt" (mode mixing); "re-paste the policy every message" (configuration failure). Correct pairs typically combine a *structural* move (schema, decomposition, examples) with a *verification* move (assumptions block, quote-the-source, checkpoint).

---

## Decision trees

| Situation | Action |
|---|---|
| Multi-deliverable or high stakes | Decompose with checkpoints |
| Extraction into a system | Define a schema; NULL for missing; sample first |
| Analysis request | Rubric + Evidence/Unknowns separation |
| Vague stakeholder ask | Translate slang into Product/Process/Performance; clarify |
| Output misses style | Add examples + counterexamples, not adjectives |
| "Make it better" urge | Name the defect and the constraint instead |
| Same brief needed weekly | Move rules to Project instructions / Skill |
| Two tightened iterations still fail | Re-diagnose layer (inputs, feature, knowledge) — Domain 07 |

---

## Exam traps

1. Mega-prompt with no checkpoints for consequential work
2. Role assigned but no success criteria ("act as a CFO" alone)
3. Letting Claude source load-bearing numbers a stakeholder must own
4. Mixing brainstorm and analysis in one prompt
5. "Make it better" / "be more professional" without concrete criteria
6. Restating stable policies every turn instead of Project instructions
7. Research asks with no scope, timeframe, or source rules
8. Decomposing trivial tasks (process theater)
9. Changing five prompt variables at once while "debugging"
10. Answering product-exam stems with API knobs (temperature, fine-tuning)

---

## Practice Q&A (18) — scenario stems with answers and rationales

**Q1.** A PM types: "Analyze our Q4 plan." Claude returns generic strategy prose. Best next prompt?
**A:** Rebuild the brief: attach the plan file, state the decision it informs, give evaluation criteria, require Evidence vs Unknowns. The failure is a missing Product/Performance Description, not the model.

**Q2.** Select TWO Performance-Description elements for a customer-email draft.
**A:** A gold example of an approved email; explicit banned claims/phrases. (Adjectives like "professional" are not acceptance tests.)

**Q3.** Legal must review a 60-page contract for renewal risks. Strongest structure?
**A:** Decompose: extract clauses relevant to renewal → tag each with evidence quotes → then assess risk with a rubric → human legal review. Extract-then-judge keeps errors traceable.

**Q4.** Claude keeps ignoring the uploaded pricing PDF. Best iteration?
**A:** Name the file explicitly ("answer only from `Pricing-v3.pdf`; quote the section; say Not found otherwise"). Pointing beats repeating the question louder.

**Q5.** A team wants campaign ideas *and* a compliance-safe shortlist. How do you prompt?
**A:** Two phases: diverge (many labeled-speculative ideas, no judgment), then converge (rank against criteria + banned-claims file). Mixing modes in one prompt degrades both.

**Q6.** Where do standing tone rules used in every chat belong?
**A:** Project instructions — stable rules are configuration, not per-prompt typing.

**Q7.** A manager's feedback is "make it better." Why is this the exam's wrong option, and the fix?
**A:** It names no defect or constraint, so improvement is luck. Fix: state what missed (audience, length, missing risks) and add the constraint.

**Q8.** Select TWO benefits of decomposition on consequential work.
**A:** Human checkpoints before irreversible steps; separating extraction from judgment so errors are localized.

**Q9.** A rewrite for executives must not change meaning. What constraint do you add?
**A:** Meaning locks: "preserve all facts and numbers exactly; change structure/length/jargon only; list any sentence whose meaning shifted." Adapt ≠ invent (Domain 03 bridge).

**Q10.** The ask touches employment-law wording and the associate isn't sure what's allowed. Prompt move?
**A:** Require an ASSUMPTIONS block and route the output to human legal review — clarify/escalate beats invention on regulated content.

**Q11.** Claude must fill a tracker with vendor, date, amount, owner. Prompt pattern?
**A:** Define the schema, one row per record, NULL if missing, then validate a sample before scaling. Schemas cut revision cycles.

**Q12.** A draft cites impressive-sounding market stats nobody provided. Iteration?
**A:** Ban unsourced figures; supply approved numbers or require UNKNOWN + an Unknowns section. Drafting strategy = must-include facts, not model-sourced facts.

**Q13.** When is single-shot prompting the right answer?
**A:** Small, clear, low-stakes tasks — decomposing an icebreaker request is process theater.

**Q14.** "Act as a McKinsey partner and review our strategy" underperforms. Missing?
**A:** Success criteria and inputs. Role without rubric and documents is costume, not brief.

**Q15.** Select TWO analysis-prompt requirements.
**A:** An explicit rubric/criteria list; separation of facts from inferences (with unknowns labeled).

**Q16.** Output is a wall of prose but ops needs it in the ticket template. Fix?
**A:** Add format contract: exact sections/fields, table shape, length caps — and keep it in the Project so it sticks.

**Q17.** A researcher wants "everything about competitor X." Better research prompt?
**A:** Scope it: which questions, which timeframe, which source types, citations required — then Research mode; validate citation support afterward.

**Q18.** After two tightened iterations, extraction still fails on scanned PDFs. Diagnosis?
**A:** Stop prompt-tweaking — the input layer (OCR-poor scans) is the problem. Two failed tightened rounds signal a layer change, not more wording (Domain 07).

---

## Quick review checklist (pre-exam)

- [ ] Recite the seven anatomy parts and the PPP triad
- [ ] Decompose: outline-fill, extract-analyze, options-decide, draft-redteam
- [ ] Iterate by naming defect + constraint; one variable at a time
- [ ] Playbooks for analysis / research / drafting / brainstorming
- [ ] Examples + counterexamples over adjectives
- [ ] ASSUMPTIONS block when proceeding under ambiguity
- [ ] Stable rules → Project instructions / Skills, not repeated prompts
- [ ] Prompt-comparison scoring: complete, non-contradictory brief wins

---

## Glossary

| Term | Meaning |
|---|---|
| **Prompt anatomy** | Role, goal, inputs, constraints, process, format, verification |
| **Description triad** | Product / Process / Performance — what, how, what counts as good |
| **Decomposition** | Splitting work into phases with checkpoints |
| **Iteration** | Refinement that names the defect and adds a constraint |
| **Few-shot** | Steering with worked examples (and counterexamples) |
| **Schema** | Fixed output structure for extraction/handoff |
| **ASSUMPTIONS block** | Explicit list of guesses made under ambiguity |
| **Meaning lock** | Rewrite constraint preserving facts and numbers exactly |
