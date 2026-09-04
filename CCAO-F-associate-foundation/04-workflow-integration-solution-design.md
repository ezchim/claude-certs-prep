---
title: Workflow Integration & Solution Design
---

# Domain 04 — Workflow Integration & Solution Design
## Maps to official CCAO-F **Workflow Integration and Solution Design** (~16%, ~10 questions)

> **Dedup note (2026-08-23):** Rewritten as a single primary-study copy. Earlier builds repeated the same drill blocks ~7×; duplicates removed and content deepened to the Domain 03 standard.

## Disclaimer

Original CCAO-F study notes for non-developers using claude.ai (Projects, Artifacts, Skills, Connectors, Research). Grounded in public Anthropic Help Center & product docs, public Claude Academy (Claude 101, AI Fluency 4D), and the published CCAO-F blueprint. Independent; not affiliated with Anthropic. Verify live product details on support.claude.com.

---

## Overview

Second-heaviest domain. Design how Claude fits real work. Official blueprint verbs: **apply** Claude to analyze requirements and use cases; **leverage** Claude for research, planning, and process optimization; **use** Claude to support solution design, development, and iteration; **integrate** Claude into existing workflows to augment or redesign them; **communicate** Claude's value **and limitations** to stakeholders. Non-developer solutions mean Projects, Skills, Artifacts, Connectors, Research, code execution, and review gates — not API architecture.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Requirements & use cases | analyze, elicit | Job-to-be-done, constraints, use-case classes |
| Research / planning / optimization | leverage, plan | Research→validated brief; process mapping |
| Solution design & iteration | design, compose, pilot | Feature composition patterns; pilot→measure→tighten |
| Integrate: augment vs redesign | integrate, decide | Where Claude drafts/retrieves/transforms; gates |
| Stakeholder communication | communicate, disclose | Value + limits + risks + metrics |

---

## Deep notes

### 1. Requirements and use-case analysis first

Elicit before tooling: trigger event, inputs, outputs, users, frequency, latency need, sensitivity class, downstream systems, definition of done, error budget. If stakeholders can't answer, Claude can help interview and outline — but humans confirm. Premature tooling is the classic fail.

Classify the use case: **generate, transform, retrieve, decide, act.** Decide/act demand stronger gates (decisions usually stay human). Generate/transform still need Discernment proportional to stakes. Map the current workflow's steps and mark where Claude drafts, retrieves, transforms, computes, or (rarely) decides.

### 2. Solution pattern catalog

- **Pattern A — Knowledge Hub:** Project with policies/examples/instructions; team edit rights; Sonnet default; evaluation checklist inside instructions.
- **Pattern B — Landscape to Decision:** Research with citations → Domain 03 claim map → decision memo Artifact.
- **Pattern C — Intake Desk:** Connector reads a mailbox/form → Claude drafts ticket fields → human confirms → system update. The human gate before the write is the point.
- **Pattern D — Playbook Skill:** Skill encodes procedure (e.g., incident updates); Project holds severity definitions.
- **Pattern E — Org Brain:** Enterprise Search for "what did we decide" with citations back to Slack/Drive.
- **Pattern F — Data-to-Deliverable:** upload data → code execution computes (totals, trends, charts) → generates the Excel/PDF/slides deliverable → human reviews numbers against source. Use when the workflow's core is *computation or file production*, not prose. (Code execution and file creation are available on all plans — verified support.claude.com, Aug 2026.)

Compose patterns; rarely is one feature a full solution. Exam answers naming a thoughtful composition beat single-feature silver bullets.

### 3. Augment vs redesign

**Augment:** keep the process, insert Claude at draft/retrieve/compute steps. Low change-management cost; the default for regulated or high-stakes flows (plus audit trails).
**Redesign:** remove steps made unnecessary by AI + connectors. Higher payoff, but needs change management, training, and clear ownership.

Decision logic: high manual waste + low risk + willing owners → redesign candidate. Regulated, external-facing, or irreversible flows → augment first with human gates; earn redesign with pilot evidence. Never design "full autopilot" for customer-facing sends — draft + human send is the exam-correct shape.

### 4. Research, planning, and process optimization

Use Research for the external landscape during planning (validated per Domain 03 before it informs decisions); Projects to persist the plan's context; code execution to quantify ("which step consumes the hours?"). Optimization targets follow measurement, not vibes: instrument time-to-draft, revision cycles, error types first, then redesign the worst step with the cheapest lever.

### 5. Stakeholder communication (value AND limitations — blueprint-explicit)

One-pager: problem, proposed workflow diagram, human gates, data flows, metrics, risks, ask. Demo with real messy inputs, not polished toys; showing a *caught* hallucination builds more trust than a perfect run. Always include: hallucination risk and the review SOP; data boundaries (what may/may not enter which plan/feature — Domain 06); what Claude should not own (final decisions, regulated judgment); success metrics and rollback.

Objection handling: "It will replace us" → augmentation metrics and higher-value work reallocation. "It hallucinates" → show the evaluation SOP and error rates after review. "Data leaves our systems" → plan controls, connector auth scopes, org policies. Hype without limits fails this domain; so does doom without a pilot plan.

### 6. Measurement and iteration

Track: time saved, revision cycles to ship, error types (factual, completeness, tone, policy), adoption, usage-limit consumption. Use the error taxonomy to update instructions/knowledge/Skills — not to blame users. Pilot one team; expand after metrics clear. Solutions are products: version them, changelog them, name an owner.

Integration failure modes: stale knowledge, unclear owners, gates bypassed under deadline pressure, adoption ignored ("we built it, nobody came"). Design for the real org, not the slide org.

### 7. Mini-cases (apply the full loop)

**Case 1 — Weekly exec digest.** Comms spends four hours each Friday compiling a digest from Slack, Drive docs, and two dashboards. Design: connectors read the sources; a Project holds the digest template, voice examples, and inclusion criteria; Claude drafts Friday morning; the comms lead validates claims (Domain 03) and sends. Metrics: prep time, corrections after send. Redesign candidate after four clean weeks: drop the manual dashboard screenshot step by exporting data for code execution to chart.

**Case 2 — RFP factory.** Sales answers 20 RFPs a quarter from a wiki nobody updates. Solution: curated answer library in Project knowledge (owned by sales ops, versioned); a Skill encoding the answer-assembly procedure; extraction schema for RFP questions; human review gate before submission because answers are contractual commitments. Anti-pattern rejected: "auto-fill and submit" — external commitments keep a human gate.

**Case 3 — Support triage.** Goal: cut first-response time. Connector reads the queue; Claude classifies severity and drafts responses citing the policy Project; agents review-and-send; Sev-1 routes straight to humans with a Claude-drafted context brief. The severity *decision* for high-impact tickets stays human; the drafting and context assembly is where Claude adds speed safely.

**Case 4 — Board-pack analysis.** CFO wants trend commentary from monthly exports. Data-to-Deliverable: code execution computes trends and builds charts + the PDF appendix; the controller verifies computed figures against source; Opus/Sonnet drafts commentary *from the computed numbers only*; human owns the narrative. Composition: code execution (math) + Project (voice, definitions) + human gate (regulated audience).

Each case rehearses the same loop: requirements → use-case class → pattern composition → gates → metrics → iterate.

### 8. Multiple-response pattern bank

Common wrong pairs to eliminate: "automate the send to save time" + "keep everything manual forever" (gate failure + anti-Delegation); "promise stakeholders zero errors" + "lead with everything that can go wrong" (hype + doom — both omit the SOP-and-metrics middle); "roll out org-wide immediately" + "pilot indefinitely without expansion criteria" (no evidence + no decision rule); "design a custom API microservice" (wrong track). Correct combinations pair a **composition** choice (Project + connector + gate; Research + validation; code execution + review) with a **communication or measurement** choice (limits disclosed, metric defined, owner named).

---

## Decision trees

| Situation | Action |
|---|---|
| Irreversible external action in flow | Human gate before execution — non-negotiable |
| Recurring work with shared context | Project (+ Skill if procedure-rich) |
| Multi-source public discovery | Research (validated) — internal: Enterprise Search |
| Workflow core is computation / file output | Code execution pattern F |
| Stakeholder needs interactive tool | Artifact |
| Unclear requirements | Discovery interview + process map first |
| Regulated / high-stakes process | Augment with gates + audit trail; redesign only with evidence |
| Success undefined | Define metrics before building |
| Pilot succeeded | Expand gradually; keep measuring; assign owner |

---

## Exam traps

1. Full autopilot on customer-facing sends (no human gate)
2. Skipping the limitations half of stakeholder communication
3. Selling Claude as error-free; or refusing pilots out of fear
4. Ignoring change management and training on redesigns
5. API/microservice architecture answers on a product-track exam
6. No success metrics or rollback plan
7. Single-feature silver bullets where composition is needed
8. Tool-first design with no requirements or process map
9. Optimizing steps nobody measured
10. Forgetting adoption: unowned solutions rot

---

## Practice Q&A (18) — scenario stems with answers and rationales

**Q1.** A director says "put Claude into our invoicing process" with no further detail. First step?
**A:** Requirements and use-case analysis — map the process, inputs/outputs, sensitivity, and definition of done before choosing features. Tooling before requirements is the trap.

**Q2.** Design a customer-support email workflow with Claude. Exam-correct shape?
**A:** Claude drafts from Project policy knowledge → human reviews and sends. Customer-facing sends keep a human gate.

**Q3.** Select TWO components for a weekly competitive brief solution.
**A:** A Project holding the brief template/instructions; Research (with citation validation) for the landscape sweep.

**Q4.** Your rollout slide only lists benefits. What must be added to pass this domain's bar?
**A:** Limitations: hallucination risk + review SOP, data boundaries, what stays human, metrics and rollback. Communicating limits is blueprint-explicit.

**Q5.** Finance wants monthly variance analysis from exported CSVs, delivered as Excel. Which pattern?
**A:** Data-to-Deliverable: code execution computes variances and generates the workbook; a human checks numbers against source before circulation.

**Q6.** When does redesign beat augmentation?
**A:** High manual waste, low risk, willing owners, and pilot evidence — remove steps AI makes unnecessary. Regulated flows start as augmentation with audit trails.

**Q7.** Which metric pair best evidences workflow value?
**A:** Time-to-shipped-draft plus error rate after review. Speed without quality (or the reverse) is half a case.

**Q8.** Ops wants a shared interactive SOP checker teammates can click through. Feature?
**A:** An Artifact — standalone interactive deliverable; viewers consume their own limits; content still validated first.

**Q9.** A pilot's outputs show recurring tone misses and two factual errors traced to an outdated policy PDF. Iteration?
**A:** Update the knowledge file (staleness) and add tone examples to instructions/Skill — fix the system that caused each miss type.

**Q10.** An exec asks "will this replace the team?" Best response frame?
**A:** Reframe to augmentation: show which steps Claude absorbs, the human judgment retained, metrics, and where freed hours go.

**Q11.** Select TWO risks that must be disclosed to stakeholders in a solution proposal.
**A:** Hallucination/error risk with its mitigation; data-sensitivity boundaries and handling rules.

**Q12.** The team wants to skip a pilot and roll out org-wide "to move fast." Why is this the wrong option?
**A:** No error taxonomy, no adoption evidence, no rollback; pilots localize failure while instructions/knowledge are still raw.

**Q13.** Intake emails should become CRM tickets. Where's the gate?
**A:** Claude drafts ticket fields from the connector-read email; a human confirms before the CRM write. Reads can be liberal; writes are gated.

**Q14.** "What did we agree with vendor X last quarter?" keeps hitting chat dead-ends. Solution?
**A:** Enterprise Search over connected Slack/Drive — org-internal cited lookup, not Research and not bigger models.

**Q15.** A recurring RFP-response procedure (12 steps, strict order) is being re-explained weekly. Solution component?
**A:** A Skill encoding the procedure, with the boilerplate/source material in Project knowledge.

**Q16.** How do you present a demo that builds durable trust?
**A:** Real messy inputs, show the evaluation step, and show a caught error with its fix — proves the safety net, not just the magic.

**Q17.** Post-rollout, usage is high but revision cycles doubled. Diagnosis path?
**A:** Segment the error taxonomy: which miss types grew, which workflow steps produce them; tighten instructions/knowledge for those steps. Measure → target → cheapest lever.

**Q18.** Select TWO signs a proposed solution is over-automated for its risk level.
**A:** Irreversible external actions execute without human confirmation; regulated judgments (HR/legal/finance) are delegated to Claude's output directly.

---

## Quick review checklist (pre-exam)

- [ ] Elicit requirements + classify use case (generate/transform/retrieve/decide/act) first
- [ ] Six solution patterns, including Data-to-Deliverable via code execution
- [ ] Augment vs redesign logic + change management
- [ ] Human gates on irreversible/external actions — always
- [ ] Stakeholder comms = value + limits + risks + metrics + rollback
- [ ] Pilot → error taxonomy → tighten → expand; named owner
- [ ] No API-architecture answers; compose product features

---

## Glossary

| Term | Meaning |
|---|---|
| **Use case classes** | Generate, transform, retrieve, decide, act — gates scale with the last two |
| **Augmentation** | Claude inserted into an existing process at draft/retrieve/compute steps |
| **Redesign** | Removing process steps made unnecessary by AI + connectors |
| **Human gate** | Required human confirmation before irreversible/external actions |
| **Pilot** | Bounded rollout that generates the error taxonomy before scaling |
| **Error taxonomy** | Factual / completeness / tone / policy miss classification driving fixes |
| **Success metric** | Predefined measure (time-to-draft, post-review error rate, adoption) |
| **Change management** | Training, ownership, and communication that make redesigns stick |
