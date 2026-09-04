---
title: Domain 03 — Evaluating & Validating Claude's Output — Simplified Technical English
pack: STE edition (ASD-STE100)
updated: 2026-08-30
---

# Domain 03 — Evaluating & Validating Claude's Output
## Maps to official CCAO-F **Output Evaluation and Validation** (~21%, heaviest, ~13 questions)

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Artifacts, Skills, Connectors, Research, eval, RAG) are exceptions and stay as written.

> **Dedup note (Aug 2026):** This file is one primary-study copy. Earlier pack builds repeated the same drill blocks about 15 times. This edition removes those duplicates.

## Disclaimer

These are original CCAO-F study notes for non-developers who use claude.ai (Projects, Artifacts, Skills, Connectors, Research). They use public Anthropic Help Center and product docs, public Claude Academy (Claude 101, AI Fluency 4D), and the published CCAO-F blueprint. They are independent. They are not affiliated with Anthropic. Verify live details on support.claude.com.

---

## Overview

This is the largest CCAO-F domain by weight. Your job is to **judge outputs before they reach customers, executives, or systems of record**. Official blueprint verbs include evaluate accuracy and completeness, and identify hallucinations, inconsistencies, and biases. You must have They also include apply fact-checking, decide when human review , and edit, adapt, refine, and compare for audience. Organise information and select formats (Artifacts, inline, structured data).

Core exam truth: **Claude can be fluent and wrong.** AI Fluency maps here to Discernment (you spot what is unsafe to ship) and Diligence (you build review habits that match the risk). Eloquence is not evidence. An Artifact is not pre-validated. A citation is not proof that the linked text supports the claim.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Accuracy & completeness | evaluate, assess | Claim vs source. Missing sections. Silent omissions |
| Hallucinations / inconsistencies / biases | identify, spot | Fabrications, math/timeline drift, stereotypes, one-sidedness |
| Fact-checking & validation | verify, cross-check | Numbers, names, dates, quotes, URLs, policy quotes |
| Human review gates | determine, escalate | L1–L4 risk tiers. When peer/expert review is mandatory |
| Edit / adapt / refine / compare | revise, curate | Audience rewrite. Rubric compare. Ship/revise/reject |
| Format fitness | organise, select | Inline vs Artifact vs structured (tables/Sheets-ready) |

---

## Deep notes

### 1. What “evaluation” means on this exam

Evaluation is a **three-way compare**. (1) the ask / success criteria, (2) sources of truth (uploaded files, Project knowledge, Research citations, policy of record), and (3) the draft. Score across:

- **Correctness** — are load-bearing claims true and supported?
- **Completeness** — is anything material missing relative to the ask or outline?
- **Consistency** — do numbers, dates, owners, and table↔prose agree?
- **Calibration** — does confidence match evidence? Are unknowns labeled?
- **Audience fit** — tone, length, jargon, decision-usefulness for the reader?
- **Policy / safety** — PII, commitments, legal-effect language, refusals honored?
- **Format fit** — is delivery mode (inline / Artifact / structured) appropriate?

Fail any one of these and you may still “revise” rather than “ship.” Completeness failures hide better than wrong claims. A polished one-pager that drops the biggest customer objection is a common Domain 03 trap.

### 2. Risk tiers (proportional review)

Match review depth to consequence. Exam multiple-response items often pair a **skip-review-on-high-risk** distractor with an **over-validate-trivia** distractor. Choose the proportional set.

| Tier | Typical work | Review depth |
|---|---|---|
| **L1 Low** | Labeled brainstorms, icebreakers, internal ideation | Skim. Keep “draft / speculative” label |
| **L2 Medium** | Internal summary of a short known doc. Team status rewrite | Spot-check key claims + completeness vs outline |
| **L3 High** | Customer email with commitments. External publish. Ops runbook change | Full claim map + peer review. Quote sources of truth |
| **L4 Critical** | Legal / medical / finance / HR / safety. Employment decisions. Irreversible connector sends | Expert human review + primary sources. Claude is drafting aid only |

**General rule:** if the output creates an **external commitment**, moves **money/people/safety**, or hits a **system of record**, treat at least L3. Irreversible Connector actions (send email, update ticket, write to CRM) always need a **human gate** before execution. This is true even if the draft looks perfect.

### 3. Claim maps (the Domain 03 signature skill)

A **claim map** turns fluent prose into a checklist you can verify. Method:

1. Extract every load-bearing claim (numbers, names, dates, quotes, attributions, commitments, causal statements, “per Policy X”).
2. Tag each: `SUPPORTED` (quote or cite the evidence), `UNSUPPORTED`, `CONTRADICTED`, `AMBIGUOUS`, `OUT_OF_SCOPE`.
3. Decide per claim: keep / rewrite / drop / escalate.
4. Only then polish tone.

**Priority scan order (poisoned-number drill):** numbers → names → dates → commitments/SLAs → legal or medical assertions → attributions/citations → soft opinions. Exams hide one bad figure inside otherwise fluent paragraphs.

**Claim extract script (prompt pattern you can reuse):** 
“List every factual claim as a bullet. For each: claim text | evidence needed | status (SUPPORTED/UNSUPPORTED/…) | suggested fix. Do not rewrite the memo until the map is complete.”

Self-check YES/NO checklists at the end of outputs speed human review. They **do not** replace L3/L4 review.

### 4. Hallucination types

Know the taxonomy. Stems rarely say “hallucination.” They show symptoms.

1. **Fabricated facts** — statistics, market sizes, “47% per Gartner 2026” with no real support.
2. **Fabricated or mismatched sources** — real-looking URLs that 404. Citations whose body does not support the claim. Invented study titles.
3. **Fabricated attributions** — quoting a person or policy section that does not exist (e.g., “per Policy 4.2” when 4.2 is unrelated or missing).
4. Overconfident inference: you treat a guess as a settled fact. Fake precision. (“exactly 12.47%”).
5. **Context bleed** — mixing details from another chat, Project file, or earlier example into this task.
6. **Instruction amnesia** — ignoring constraints (audience, banned claims, “quote only from file”).

**Citations are not automatic validation.** Research outputs still need **citation-support checks**: open the source, confirm the sentence actually backs the claim, confirm date/version relevance. Irrelevant but real links still fail validation.

### 5. Inconsistencies and bias

Inconsistencies: incorrect arithmetic. Timeline contradictions. Table totals ≠ prose. Constraint drift mid-answer (“no discounts” then offers 15%). Owner named in summary but absent from transcript.

**Bias patterns the exam cares about:** stereotype defaults in examples. Position bias (the output gives too much weight to the first source). Fake neutrality that hides a preferred option. One-sided analysis missing counterarguments or risks. Fix by requiring counterarguments, decision criteria, and explicit unknowns. Do not only ask “are you sure?” once.

### 6. Fact-checking workflow (ship / revise / reject / escalate)

Standard flow for consequential drafts:

1. **Risk-tier** the task (L1–L4).
2. **Extract claims** (map).
3. **Map evidence** to Project knowledge / uploaded policy / Research citations / primary systems.
4. **Spot-check** high-severity claims first (numbers, commitments, legal-effect).
5. **Completeness pass** against the original outline or ask.
6. **Decide:** ship (often labeled) / revise / reject unsupported block / escalate to human expert.
7. **Format** only after content passes—or format choice is itself part of fitness (see below).

Red-team Claude as **assistance, not authority**: Adversarial prompts ('attack this draft for unsupported commitments') help you find issues. Outsourcing final regulated judgment to the model fails Diligence.

### 7. Human-review gates (when you must stop)

Mandatory human / expert review when any of these apply:

- Legal, medical, finance, HR, safety, or employment-effect content
- External customer or public publish
- Irreversible Connector / automation actions
- PII or sensitive personal data in the draft
- Low evidence coverage on load-bearing claims
- Topic beyond your or Claude’s demonstrated competence
- High ambiguity with material downside

If you use Claude to grade Claude, you only find issues. Treating that grade as legal sign-off is an exam-wrong answer. “Are you sure?” once is **not** a validation method.

### 8. Audience edit, adapt, refine, compare

**Adapt ≠ invent.** Audience rewrite changes length, jargon, structure, and emphasis. It does not change facts. If the exec brief needs a number you do not have, mark `UNKNOWN` or escalate. Do not fabricate a crisp KPI.

**Compare drafts with a rubric table:**

| Criterion | Draft A | Draft B | Winner / merge note |
|---|---|---|---|
| Accuracy vs source | … | … | … |
| Completeness vs outline | … | … | … |
| Calibration / unknowns | … | … | … |
| Audience fit | … | … | … |
| Policy / commitments | … | … | … |

Merge strengths. Do not prefer a draft only because it is longer or more eloquent. Style-only polish after a failed claim map is a trap answer.

### 9. Format choice: Artifacts / inline / structured

Wrong format fails **fit-for-purpose** even when content is good.

| Format | Use when | Anti-pattern |
|---|---|---|
| **Inline chat** | Short answers, quick clarifications, iterative brainstorming | You send a board pack as raw chat. |
| **Artifact** | Standalone editable deliverable (doc, app, interactive table, HTML) you will revise or share | You force a two-word answer into an Artifact 'app'. |
| **Structured** (table, bullets, CSV/Sheets-ready) | Output that feeds another step. Easy claim checking. Ops handoff | Prose paragraphs when the next step is a spreadsheet |
| **Research + citations** | Multi-source investigation needing cited landscape | Research for a single already-uploaded PDF you should quote |
| **Project knowledge (after approval)** | A source of truth for recurring work | You paste unvalidated drafts into knowledge as 'truth'. |

Organise and curate: headings, claim lists, decision/evidence/risks sections for leaders. Attractive dashboards with wrong metric definitions fail evaluation. Use **semantics before aesthetics**.

### 10. Bridges to other domains

- Bad prompts create difficult eval work (02).
- Wrong features raise error rates (**01**).
- Stale Project knowledge yields true-to-file, false-to-world (**05**).
- If you ship without gates, you fail governance (06).
- Repeated eval misses become troubleshooting patterns (**07**).

Fluency without Discernment is a business risk. The **21% weight is intentional**: Anthropic certifies associates who stop fluent errors from becoming organizational facts.

### 11. Personal SOP (Diligence system)

Keep a one-page rubric. Pre-define L1–L4 rules per workflow. Store known failure examples in a Project. Record almost-shipped failures monthly. Lab loop: Research brief → map ~15 claims → decide ship/revise/reject → Artifact only after pass → write three exam stems from mistakes found.

Validation scripts that operationalize Discernment (they also serve as Domain 02 performance tools): claim extract. Adversarial attack. Audience rewrite diff. Commitment/obligation scanner. Privacy/PII redaction scanner.

---

## Decision trees

| Situation | Action |
|---|---|
| Unsourced critical external claim | Revise or reject. Do not ship |
| Internal contradiction (35 vs 28, table≠prose) | Revise for consistency |
| Missing required section vs outline | Completeness fail → revise |
| Audience mismatch, facts OK | Adapt (tone/structure), do not invent |
| Labeled low-stakes brainstorm | Ship labeled (L1) |
| Connector send / write action | Human gate before execution |
| Employment / legal / medical / finance memo | L4 expert review |
| Citation present but unsupported | Validation fail (same as no citation) |
| Safety refusal on disallowed ask | Appropriate—do not “fix” as a bug |
| Pretty Artifact, wrong metric defs | Revise semantics before ship |

**Tier path:** L1 skim → L2 spot-check → L3 full claim map + peer → L4 expert + primary sources.

---

## Timed claim-map drills (90-second stems)

Work each stem in **~90 seconds**: Extract 3 to 6 load-bearing claims, tag the status, then select ship, revise, reject, or escalate. Do **not** rewrite the whole memo in the drill. Map first. Answers follow each stem.

### Drill A — Vendor email
Stem: Draft thanks Acme and cites “47% faster close rates per Gartner 2026” linking a lifestyle blog. Offers a 60-day pilot with SLA credits “as discussed” (not in thread). 
**Map focus:** statistic + source. SLA commitment. 
**Answer:** Reject/revise unsupported Gartner claim. Strip or rewrite SLA until evidence exists. Get legal/commercial review before send (L3).

### Drill B — Meeting notes
Stem: Notes assign Jordan as owner of “migrate billing by Friday”. Transcript never names Jordan. Jordan was not in the invite. 
**Map focus:** attribution / action owner. 
**Answer:** Fail attribution. Mark `OWNER UNCLEAR`. Do not invent owners. Escalate to the facilitator.

### Drill C — Refund reply vs policy file
Stem: Chat answers “always refund within 90 days from general knowledge”. Uploaded `Refund_Policy_v3.pdf` says 30 days for digital goods. 
**Map focus:** policy conflict. 
**Answer:** Reject general-knowledge answer. Quote the policy file. Use an L3 customer commitment gate.

### Drill D — Research brief citation
Stem: Research claims “EU guidance banned feature X in 2024” with a citation that discusses an unrelated privacy cookie rule. 
**Map focus:** citation support. 
**Answer:** Unsupported (mismatched citation). Revise the claim or find a supporting source. Do not ship on link presence alone.

### Drill E — Dashboard Artifact
Stem: Interactive Artifact “ARR by segment” uses “bookings” field labeled as ARR. Finance dictionary defines ARR differently. 
**Map focus:** metric semantics. 
**Answer:** Revise definitions before aesthetics. Use a structured/finance owner check (L3/L4).

### Drill F — Campaign brainstorm
Stem: Clearly labeled “speculative concepts only—not approved messaging”. Playful but risky slogans. 
**Map focus:** risk tier + label. 
**Answer:** Ship as labeled L1 draft. Do not treat as brand-approved copy.

### Drill G — Exec brief completeness
Stem: Ask required Decision / Evidence / Risks / Ask. Draft has Decision and Evidence only. Omits Risks and Ask. 
**Map focus:** completeness. 
**Answer:** Completeness fail → revise. Polished half-briefs still fail.

### Drill H — Timeline inconsistency
Stem: Prose says pilot starts 15 Sep. Table says 5 Oct. Both presented as firm. 
**Map focus:** consistency. 
**Answer:** Revise. Reconcile dates against source of truth before external share.

### Drill I — Biased shortlist
Stem: The vendor comparison praises one tool with strong adjectives. No criteria table. Competitors dismissed in one line. 
**Map focus:** bias / fake neutrality. 
**Answer:** Revise to criteria-based compare + counterarguments. Do not ship as “objective analysis.”

### Drill J — Connector send
Stem: Draft support reply looks correct. User prompt: “send via Gmail connector now.” 
**Map focus:** human gate. 
**Answer:** Human gate before send. Evaluation of text is not authorization to execute.

### Drill K — Fabricated policy section
Stem: Memo says “per Employee Handbook §4.2 remote stipend is $500”. Handbook has no §4.2 stipend language. 
**Map focus:** fabricated attribution. 
**Answer:** Reject claim. Quote the real section or mark unsupported. Use HR/expert review (L4).

### Drill L — Poisoned number in fluent prose
Stem: Three paragraphs of solid summary. One sentence says “NPS rose from 42 to 71” but source table shows 42 → 47. 
**Map focus:** number priority scan. 
**Answer:** Catch on first number pass. Revise 71→47. Ship only after the fix.

*(Twelve unique stems—practice rotation, not copy-paste padding.)*

---

### 12. Mini-cases (apply the full loop)

**Case 1 — Customer success email.** Claude drafts a renewal note promising “priority roadmap influence” and citing “NPS 72 from last quarter.” Project knowledge has NPS 61. Roadmap influence was never approved. Tier L3. Claim-map both promises. Revise: quote real NPS or drop. Remove unapproved influence language. Use peer review before send. Format: inline for the email body. Optional structured bullet of approved talking points stored in Project after approval—not before.

**Case 2 — Competitive landscape Artifact.** Research produces a polished HTML Artifact comparing four vendors. Two citations are solid. One statistic traces to a press release restating the vendor’s own claim. One “analyst quote” has no matching source text. Completeness vs ask is good. Correctness is mixed. Action: keep Artifact shell. Rewrite unsupported cells to `Unverified`. Swap the press-release stat for a primary filing, or drop it. Then ship labeled “draft for review.” Semantics and evidence are more important than visual polish.

**Case 3 — HR FAQ.** Prompt asks for a friendly FAQ on parental leave. Claude invents a “six weeks fully paid for all contractors” bullet. Handbook covers employees only. Contractors are out of scope. This is L4-adjacent employment-effect content. Reject invented contractor benefit. Escalate to HR. Do not publish FAQ from chat alone. Safety/policy Diligence outweighs helpful tone.

**Case 4 — Ops runbook.** Structured table of on-call steps looks consistent, but step 4 says “page Secondary if no ack in 5 minutes” while prose intro says 15 minutes. Inconsistency fail. Revise to one SLA from the source runbook. Keep structured format because ops will execute from the table.

These cases rehearse the same loop: tier → map → evidence → completeness/consistency → ship/revise/reject/escalate → format.

### 13. Multiple-response pattern bank

Expect Select TWO/THREE items that mix a correct proportional action with distractors. Common wrong pairs to eliminate:

- Use “Skip review because Sonnet/Opus ” + “Run a full legal review on a labeled brainstorm” (prestige bias + over-validation).
- “Ship because there are citations” + “Delete all citations to reduce clutter” (blind trust + anti-evidence)
- “Autosend via Connector to save time” + “Never use Connectors” (gate failure + absolute ban)
- “Ask Claude if it is sure” as sole control + “Only humans may ever draft” (fake validation + anti-Delegation)

Correct patterns usually combine **claim-level verification** with **risk-proportional human gates** and **format fit**.

## Exam traps

1. Eloquence = accuracy 
2. Confidence / fake precision = evidence 
3. Skipping completeness because the prose “looks finished” 
4. Blind trust in citations or Research footnotes 
5. Self-grade / “are you sure?” as legal or HR sign-off 
6. Autosend via Connectors without a human gate 
7. Ignoring audience while “fixing facts later” 
8. You treat safety refusals as defects that you must not defeat. 
9. Style-only polish after a failed claim map 
10. Assuming Artifacts are pre-validated because they look like products 
11. Over-validating L1 trivia while skipping L3 commitments (disproportionality) 
12. Inventing KPIs during an “audience adapt” rewrite 

---

## Practice Q&A (25) — answers with brief rationales

**Q1.** Heaviest CCAO-F domain by weight? 
**A:** Output Evaluation and Validation (~21%). Judgment dominates the associate exam.

**Q2.** Research footnote URL returns 404. First validation call? 
**A:** Validation fail for that claim. Treat as unsupported until a working supporting source exists.

**Q3.** Select TWO strong hallucination signals. 
**A:** Specific numbers with no verifiable source. Attributions to missing people/policy sections. (Fluent tone is not a signal of truth.)

**Q4.** Which output most clearly needs human expert review before use? 
**A:** Employment / HR memo with legal-effect language (L4).

**Q5.** First step when you evaluate a consequential customer email draft? 
**A:** Build a claim map (then evidence tags). Do this before tone edits.

**Q6.** Summary says 35 open issues. Table totals 28. Primary failure type? 
**A:** Inconsistency (internal contradiction).

**Q7.** Select TWO completeness checks. 
**A:** Compare against required outline/sections. Confirm required disclaimers/risks/asks are present.

**Q8.** Why is a Research brief still not auto-approved? 
**A:** Citations need support checks. Links can be real yet irrelevant or mismatched.

**Q9.** Best way to choose between Draft A and Draft B? 
**A:** Criterion rubric table (accuracy, completeness, calibration, audience, policy). Merge strengths.

**Q10.** Output must feed a Google Sheet next. Best format? 
**A:** Structured table / Sheets-ready data. Do not use a long prose Artifact.

**Q11.** Claude refuses a disallowed medical dosing request. Correct interpretation? 
**A:** Appropriate safety behavior. It is not a defect to “fix.”

**Q12.** Select TWO high-risk claim classes. 
**A:** Customer commitments/SLAs. Legal-effect or employment claims.

**Q13.** Memo cites “Policy 4.2” but 4.2 does not exist. Failure? 
**A:** Fabricated attribution / unsupported claim → reject or rewrite from real source.

**Q14.** Adapting a technical note for executives means? 
**A:** Change structure/jargon/length. Do not invent missing metrics.

**Q15.** Role of red-team / adversarial critique prompts? 
**A:** Assistance to surface issues. Not final regulated authority.

**Q16.** Draft labels three items `Unknown—needs finance`. Good or bad? 
**A:** Good calibration. Prefer labeled unknowns over fake precision.

**Q17.** Select TWO correct format matches. 
**A:** Artifact for interactive standalone deliverable. Inline for short iterative answers.

**Q18.** User asks Claude to send the approved-looking reply via email Connector. Required control? 
**A:** Human gate before irreversible send.

**Q19.** One-sided vendor write-up. Fix? 
**A:** Add explicit criteria, counterarguments, and evidence tags. Do not add more adjectives.

**Q20.** Board brief omits Risks and Ask sections required by template. Result? 
**A:** Completeness fail → revise even if Decision prose is eloquent.

**Q21.** Which AI Fluency pair is central to this domain? 
**A:** Discernment + Diligence.

**Q22.** End-of-output YES/NO self-check finds two unmet constraints. Next? 
**A:** Revise to meet constraints. The checklist aids humans. It does not replace L3/L4 review.

**Q23.** Select TWO risks of equating eloquence with accuracy. 
**A:** Shipping fluent false stats. Skipping claim maps because the draft “sounds right.”

**Q24.** Claim tagged `UNSUPPORTED` in an external customer commitment email. Action? 
**A:** Revise or reject that claim before send. Escalate if commercially sensitive.

**Q25.** Leader asks for a one-page decision brief. Best curation pattern? 
**A:** Decision / Evidence / Risks / Ask—organised for action, after claim validation.

---

## Quick review checklist (pre-exam)

- [ ] Risk-tier every consequential task (L1–L4) 
- [ ] Claim-map numbers, names, dates, commitments first 
- [ ] Verify citation **support**, not citation **presence** 
- [ ] Consistency pass: math, timelines, table↔prose 
- [ ] Completeness vs outline / required sections 
- [ ] Human gates for legal/HR/finance/safety, external publish, Connector writes, PII 
- [ ] Audience adapt without inventing facts 
- [ ] Format: inline vs Artifact vs structured—on purpose 
- [ ] Honor safety refusals. Do not treat as bugs 
- [ ] Second-pass / red-team Claude as aid, not sign-off 
- [ ] Proportional review: neither skip L3 nor over-police L1 

---

## Glossary

| Term | Meaning |
|---|---|
| **Hallucination** | Fluent false or unsupported content (facts, sources, attributions, overconfident inference) |
| **Claim mapping** | Extract → evidence-tag → decide keep/rewrite/drop/escalate |
| **Calibration** | Confidence matched to evidence. Unknowns labeled |
| **Completeness** | Material requirements present vs ask/outline—not merely “correct sentences” |
| **Consistency** | Internal agreement across prose, tables, numbers, timelines |
| **Human-in-the-loop** | Required human/expert gate before high-consequence use or irreversible action |
| **Red-team** | Adversarial critique to surface failures. Assistance only |
| **Fit for purpose** | Content **and** format suitable for audience and next workflow step |
| **Unsupported** | Claim lacks adequate evidence from an allowed source of truth |
| **Risk tier (L1–L4)** | Proportional review depth from labeled brainstorm to critical expert review |

---

## Worked evaluations (unique examples — once)

1. **Vendor email** invents “47% per Gartner 2026” with unrelated blog citation → reject claim. Rewrite with allowed proof. Legal/commercial review. 
2. **Meeting notes** invent action owner absent from transcript → fail attribution. Enforce `OWNER UNCLEAR`. 
3. **Speculative campaign** clearly labeled → ship as draft only (L1). 
4. **Refund answer** from general knowledge conflicts uploaded policy → reject. Quote policy file. 
5. An attractive Artifact dashboard uses metric definitions that differ from the finance dictionary → revise semantics before aesthetics.

**Poisoned-number drill:** Check numbers, names, dates, and commitments before you check tone. Exams hide one bad figure in fluent prose.

## Validation scripts (once)

Claim extract. Adversarial attack. Audience rewrite diff. Commitment/obligation scanner. Privacy/PII redaction scanner. Self-check YES/NO lists speed review. They do not replace L3/L4 gates.

## Personal SOP lab (once)

Research brief → map ~15 claims → decide → Artifact only after pass → write three exam stems from mistakes found. Store failure examples in a Project. Log near-misses monthly as organizational Diligence.
