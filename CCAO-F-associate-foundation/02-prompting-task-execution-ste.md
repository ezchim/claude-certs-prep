# Domain 02 — Prompting & Task Execution
## Maps to official CCAO-F **Prompting and Task Execution** (~14%, ~8 questions)

> **Note:** This edition follows the ASD-STE100 Simplified Technical English writing rules. Simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Projects, Artifacts, Skills, Connectors, Research, prompting, prompt, schema, eval) are exceptions and stay as written.

> **Dedup note (2026-08-23):** This is one primary-study copy. Earlier builds repeated the same drill blocks about 10 times. Those duplicates are gone. The content now matches the Domain 03 depth.

## Disclaimer

These notes are original CCAO-F study notes. They are for people who are not developers. They use claude.ai (Projects, Artifacts, Skills, Connectors, Research). The notes use public Anthropic Help Center and product docs. They also use public Claude Academy (Claude 101, AI Fluency 4D framework) and the published CCAO-F blueprint. The notes are independent. They are not affiliated with Anthropic. Check live product details on support.claude.com.

---

## Overview

Prompting on CCAO-F equals professional briefing. Official blueprint verbs: **create** effective prompts for business and technical tasks. **Apply task decomposition** to structure complex requests. **Iterate** prompts to improve quality. **Adapt prompting strategies by task type**. The guide names *analysis, research, drafting, brainstorming*. The AI Fluency **Description** triad is Product, Process, Performance. This triad is the core structure. Product is what the deliverable is. Process is how you get there. Performance is what "good" looks like.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Effective prompts | create, structure | Prompt anatomy. The Description triad |
| Task decomposition | apply, sequence | Phased prompts with checkpoints |
| Iteration | iterate, refine | Named-defect feedback. Change one variable at a time |
| Task-type adaptation | adapt, select | Analysis / research / drafting / brainstorming playbooks |

---

## Deep notes

### 1. Prompt anatomy

A complete brief carries these parts: **role**, **goal**, **inputs**, **constraints**, **process**, **format**, **verification**.

- **Role:** who Claude acts as.
- **Goal:** the deliverable plus the decision it supports.
- **Inputs:** files by name, pasted data, connector sources.
- **Constraints:** length, tone, banned claims, audience.
- **Process:** ordered steps and checkpoints.
- **Format:** schema, sections, table.
- **Verification:** what you check before you answer.

You rarely need all seven parts. Exam-best prompts have no *missing load-bearing* part. They also have no internal contradiction.

**Description triad:** *Product* Description names the deliverable. It also names the audience decision that the deliverable supports. *Process* Description sequences tools and checkpoints. *Performance* Description sets acceptance tests, examples, and counterexamples. A missing part creates a predictable failure. No Product Description gives empty, unusable output. No Process skips steps. No Performance gives stylish wrong answers.

**Stakeholder translation** is the associate skill. "Make it pop" needs voice samples. "Keep it legal" needs disclaimer files and banned claims. "Board ready" needs a Decision/Risks/Ask structure. "Quick thoughts" must have a speculative label. Stems use stakeholder slang. Translate the slang before you select an answer.

**Contradictions** must have an explicit resolution. Example: "explain like a beginner but keep undefined internal acronyms." Unresolved contradictions give unstable outputs. People then blame the model.

**Examples are better than adjectives.** Two or three gold examples (and one counterexample) are better than paragraphs of style description. Few-shot is the strongest control a non-developer has.

**Ambiguity:** prefer *clarify* over *invent*. If you proceed, require an explicit `ASSUMPTIONS` block. Wrong guesses then stay visible. You can fix them at low cost.

### 2. Task decomposition

Break multi-deliverable or high-stakes work into phases. Put human checkpoints between phases:

- **Outline → fill:** Approve the structure before the prose.
- **Extract → analyze:** Separate facts from judgment. Then you can trace errors.
- **Options → decide:** Make alternatives without a commitment. Then select with criteria.
- **Draft → red-team:** Produce the draft. Then attack it as an adversary (this bridges Domain 03).

Worked patterns:

- **Ops postmortem:** Timeline facts. Then tag contributing factors as Evidence vs Hypothesis. Then write actions only with named owners.
- **Marketing email:** Facts plus legal must-includes. Then subjects plus body. Then a compliance pass against banned claims.
- **HR FAQ:** Answer only from knowledge. Quote the section. Write "Not found" if the answer is absent.
- **Offsite agenda:** Brainstorm. Then cluster. Then select. Then make a timed agenda. Then do a risk check.
- **Data cleanup:** Define a schema. Validate a 20-row sample via code execution. Then scale.

**Decomposition is also governance.** It inserts human gates before irreversible or external steps. Exams test this crossover in Domains 04 and 06.

Decompose when you have:

- multiple deliverables
- high stakes
- mixed task types (never brainstorm and analyze in the same prompt)
- a need for intermediate checkpoints

Single-shot is fine when the task is small, clear, and low-stakes. Decomposition of trivia wastes time and usage.

### 3. Iteration craft

Strong iteration **names the defect and the constraint to add**. Example: "the summary invented a Q3 figure — use only numbers from `Q3-report.pdf`, mark anything else UNKNOWN." This is better than "make it better." Change **one major variable at a time** when you debug, so you learn the cause. Prefer narrow edits ("fix only the table") over a full regeneration. When a prompt fails twice with tighter wording, the problem is usually inputs, feature choice, or task framing. The problem is not word choice (Domain 07's ladder).

Keep a micro-prompt library:

- assumptions list
- FACTS vs INFERENCES
- only the table
- NULL if missing
- strongest counterargument
- meaning-preserving diff
- outline only
- claim-to-source map
- three options without picking
- compress without changing numbers

### 4. Task-type playbooks (the named blueprint objective)

| Task type | Strategy | Signature moves |
|---|---|---|
| **Analysis** | Criteria first. Separate evidence from inference. | Give a rubric. Demand FACTS vs INFERENCES sections. Require counterarguments and unknowns. |
| **Research** | Scope + sources + citations | Define the question, timeframe, and source rules. Require citation-support (Domain 03). Use the Research feature for a multi-source public landscape. |
| **Drafting** | Must-include facts + audience + format | Supply approved facts. Do not let Claude source them. Add voice samples, banned phrases, and length caps. |
| **Brainstorming** | Diverge, then converge — in separate steps | Ask for quantity and variety without judgment first. Rank with criteria in a second step. Label the output speculative. |

Mode mixing is the common error. A prompt that asks for "creative ideas but strictly accurate to policy" gets neither result. Split the phases.

### 5. Prompting as a system (team scale)

Stable rules move out of prompts. Recurring instructions go to Project instructions. Reference detail goes to knowledge files (point to them by name). Rich procedures go to Skills. Shared Projects need priority rules for conflicts (example: legal > brand). Store gold examples and counterexamples in knowledge. Teach teammates to paste the request. Do not paste vague urgency. Vague urgency without specs wastes usage limits. A prompt that you repeat weekly is configuration, not typing (Domain 05).

### 6. Timed heuristics for prompt-comparison items

When the exam shows four candidate prompts, score each on Goal, Inputs, Constraints, Process, Format, Verification. The highest complete, non-contradictory score wins. Prefer clarify over invent. Prefer phased approvals over one mega-prompt for consequential transformations. API-parameter options (temperature, fine-tuning) are distractors on this product-focused exam. Before you "escalate the model," ask this: could a careful intern succeed from this brief? If no, fix the Description first.

### 7. Worked briefs (before → after)

**Weak:** "Summarize this customer call." → **Strong:** "From `call-2026-08-12.txt`, produce a one-page summary for the account exec deciding whether to escalate. 5 bullet facts with timestamps, customer sentiment with a supporting quote, commitments made by either side (quote exactly. UNKNOWN if unclear), and recommended next step with rationale. Do not infer facts not in the transcript."

**Weak:** "Write a LinkedIn post about our launch." → **Strong:** "Draft a LinkedIn post announcing the launch for prospective mid-market buyers. Must include: the two approved claims in `launch-facts.md` (verbatim numbers), a customer-pain opening, ≤150 words, no superlatives from `banned-phrases.md`. Match the voice of the two examples in knowledge. Give two variants: safe and bold."

**Weak:** "Is this vendor contract okay?" → **Strong:** "Phase 1: extract every clause in `vendor-msa.pdf` touching liability, termination, data handling, and auto-renewal — quote each. Phase 2 (after my confirmation): assess each extracted clause against the checklist in `contract-redlines.md`, tag OK / FLAG / ESCALATE with one-line reasons. Final legal judgment stays with counsel."

The pattern: Name the file. Name the audience decision. Name the must-includes. Name the format. Name the verification rule. Phase anything consequential.

### 8. Multiple-response elimination bank

Select-TWO/THREE prompting items usually pair one correct craft move with distractors. Practice this: eliminate these options:

- "make it better" (no defect named)
- 'add temperature=0' (API parameter, incorrect approach)
- "use the model with the most capability" (Domain 01 prestige bias)
- "let Claude find the statistics" (Claude sources load-bearing facts)
- "combine brainstorm and compliance review into one prompt" (mode mixing)
- "re-paste the policy every message" (configuration failure)

Correct pairs typically combine a *structural* move with a *verification* move. Structural moves: schema, decomposition, examples. Verification moves: assumptions block, quote-the-source, checkpoint.

---

## Decision trees

| Situation | Action |
|---|---|
| Multi-deliverable or high stakes | Decompose with checkpoints |
| Extraction into a system | Define a schema. Use NULL for missing. Sample first. |
| Analysis request | Rubric plus Evidence/Unknowns separation |
| Vague stakeholder ask | Translate slang into Product/Process/Performance. Clarify. |
| Output misses style | Add examples and counterexamples. Do not add adjectives. |
| "Make it better" urge | Name the defect and the constraint instead |
| Same brief needed weekly | Move rules to Project instructions / Skill |
| Two tightened iterations still fail | Re-diagnose the layer (inputs, feature, knowledge) — Domain 07 |

---

## Exam traps

1. A very large prompt with no checkpoints for consequential work
2. A role with no success criteria ("act as a CFO" alone)
3. You let Claude source load-bearing numbers that a stakeholder must own
4. Mix of brainstorm and analysis in one prompt
5. "Make it better" / "be more professional" without concrete criteria
6. Restatement of stable policies every turn instead of Project instructions
7. Research asks with no scope, timeframe, or source rules
8. Decomposition of trivial tasks (unnecessary process)
9. Change of five prompt variables at once while you debug
10. Answers to product-exam stems with API knobs (temperature, fine-tuning)

---

## Practice Q&A (18) — scenario stems with answers and rationales

**Q1.** A PM types: "Analyze our Q4 plan." Claude returns generic strategy prose. What is the best next prompt?
**A:** Rebuild the brief. Attach the plan file. State the decision it informs. Give evaluation criteria. Require Evidence vs Unknowns. The failure is a missing Product/Performance Description. The failure is not the model.

**Q2.** Select TWO Performance-Description elements for a customer-email draft.
**A:** A gold example of an approved email. Explicit banned claims/phrases. (Adjectives like "professional" are not acceptance tests.)

**Q3.** Legal must review a 60-page contract for renewal risks. What is the strongest structure?
**A:** Decompose. Extract clauses relevant to renewal. Tag each with evidence quotes. Then assess risk with a rubric. Then do a human legal review. Extract-then-judge keeps errors easy to trace.

**Q4.** Claude still ignores the uploaded pricing PDF. What is the best iteration?
**A:** Name the file explicitly ("answer only from `Pricing-v3.pdf`. Quote the section. Say Not found otherwise"). A file pointer is better than a louder restatement of the question.

**Q5.** A team wants campaign ideas *and* a compliance-safe shortlist. How do you prompt?
**A:** Use two phases. First diverge: many labeled-speculative ideas, no judgment. Then converge: rank against criteria plus a banned-claims file. Mode mixing in one prompt lowers quality of both.

**Q6.** Where do standing tone rules for every chat belong?
**A:** Project instructions. Stable rules are configuration, not per-prompt typing.

**Q7.** A manager's feedback is "make it better." Why is this the exam's wrong option, and what is the fix?
**A:** It names no defect or constraint. Improvement is then luck. Fix: state what missed (audience, length, missing risks). Add the constraint.

**Q8.** Select TWO benefits of decomposition on consequential work.
**A:** Human checkpoints before irreversible steps. Separation of extraction from judgment so errors stay localized.

**Q9.** A rewrite for executives must not change meaning. What constraint do you add?
**A:** Meaning locks: "preserve all facts and numbers exactly. Change structure/length/jargon only. List any sentence whose meaning shifted." Adapt is not invent (Domain 03 bridge).

**Q10.** The ask touches employment-law wording. The associate is not sure what is allowed. What is the prompt move?
**A:** Require an ASSUMPTIONS block. Route the output to human legal review. Clarify or escalate. Do not invent on regulated content.

**Q11.** Claude must fill a tracker with vendor, date, amount, owner. What is the prompt pattern?
**A:** Define the schema. Use one row per record. Use NULL if missing. Then validate a sample before you scale. Schemas cut revision cycles.

**Q12.** A draft cites market stats that sound impressive. Nobody provided them. What is the iteration?
**A:** Ban unsourced figures. Supply approved numbers, or require UNKNOWN plus an Unknowns section. Drafting strategy equals must-include facts. It does not equal model-sourced facts.

**Q13.** When is single-shot prompting the right answer?
**A:** Small, clear, low-stakes tasks. Decomposition of an icebreaker request is unnecessary process.

**Q14.** "Act as a McKinsey partner and review our strategy" does not perform well. What is missing?
**A:** Success criteria and inputs. A role without a rubric and documents is a role name only, not a complete brief.

**Q15.** Select TWO analysis-prompt requirements.
**A:** An explicit rubric/criteria list. Separation of facts from inferences (with unknowns labeled).

**Q16.** The output is a long block of prose. Ops needs it in the ticket template. What is the fix?
**A:** Add a format contract: exact sections/fields, table shape, length caps. Keep it in the Project so the format stays.

**Q17.** A researcher wants "everything about competitor X." What is a better research prompt?
**A:** Scope it. Name the questions, the timeframe, and the source types. Require citations. Then use Research mode. Validate citation support afterward.

**Q18.** After two tightened iterations, extraction still fails on scanned PDFs. What is the diagnosis?
**A:** Stop more prompt edits. The input layer (OCR-poor scans) is the problem. Two failed tightened rounds signal a layer change, not more wording (Domain 07).

---

## Quick review checklist (pre-exam)

- [ ] Recite the seven anatomy parts and the PPP triad
- [ ] Decompose: outline-fill, extract-analyze, options-decide, draft-redteam
- [ ] Iterate: name the defect plus the constraint. Change one variable at a time
- [ ] Playbooks for analysis / research / drafting / brainstorming
- [ ] Examples plus counterexamples over adjectives
- [ ] ASSUMPTIONS block when you proceed under ambiguity
- [ ] Stable rules → Project instructions / Skills, not repeated prompts
- [ ] Prompt-comparison scoring: a complete, non-contradictory brief wins

---

## Glossary

| Term | Meaning |
|---|---|
| **Prompt anatomy** | Role, goal, inputs, constraints, process, format, verification |
| **Description triad** | Product / Process / Performance — what, how, and what counts as good |
| **Decomposition** | Split of work into phases with checkpoints |
| **Iteration** | Refinement that names the defect and adds a constraint |
| **Few-shot** | Control with worked examples (and counterexamples) |
| **Schema** | Fixed output structure for extraction/handoff |
| **ASSUMPTIONS block** | Explicit list of guesses made under ambiguity |
| **Meaning lock** | Rewrite constraint that keeps facts and numbers exactly |
