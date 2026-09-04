# Domain 04 — Workflow Integration & Solution Design
## Maps to official CCAO-F **Workflow Integration and Solution Design** (~16%, ~10 questions)

> **Note:** This edition follows the ASD-STE100 Simplified Technical English writing rules. Simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Projects, Artifacts, Skills, Connectors, Research) are exceptions and stay as written.

> **Dedup note (2026-08-23):** This file is one primary-study copy. Earlier builds repeated the same drill blocks about 7 times. Those duplicates are gone. The content now matches the Domain 03 depth.

## Disclaimer

These notes are original CCAO-F study notes. They are for people who are not developers. They use claude.ai (Projects, Artifacts, Skills, Connectors, Research). The notes use public Anthropic Help Center and product docs. They also use public Claude Academy (Claude 101, AI Fluency 4D) and the published CCAO-F blueprint. The notes are independent. They are not affiliated with Anthropic. Check live product details on support.claude.com.

---

## Overview

This domain has the second-highest exam weight. You design how Claude fits actual work. Official blueprint verbs: **apply** Claude to analyze requirements and use cases. **Leverage** Claude for research, planning, and process optimization. **Use** Claude to support solution design, development, and iteration. **Integrate** Claude into existing workflows to augment or redesign them. **Communicate** Claude's value **and limitations** to stakeholders. For people who are not developers, solutions use Projects, Skills, Artifacts, Connectors, Research, code execution, and review gates. Solutions do not use API architecture.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Requirements & use cases | analyze, elicit | The work to complete, constraints, use-case classes |
| Research / planning / optimization | leverage, plan | Research that leads to a validated brief. Process maps |
| Solution design & iteration | design, compose, pilot | Feature composition patterns. Pilot, then measure, then tighten |
| Integrate: augment vs redesign | integrate, decide | Where Claude drafts, retrieves, or transforms. Gates |
| Stakeholder communication | communicate, disclose | Value + limits + risks + metrics |

---

## Deep notes

### 1. Requirements and use-case analysis first

Collect requirements before you select tools. Record the trigger event, inputs, outputs, users, frequency, latency need, sensitivity class, downstream systems, definition of done, and error budget. If stakeholders cannot answer, Claude can help you interview and outline. Humans confirm the answers. Tool selection before requirements is a common exam error.

Classify the use case: **generate, transform, retrieve, decide, act.** Decide and act need stronger gates. Decisions usually stay with humans. Generate and transform still need Discernment that matches the stakes. Map the steps of the current workflow. Mark where Claude drafts, retrieves, transforms, computes, or (rarely) decides.

### 2. Solution pattern catalog

- **Pattern A — Knowledge Hub:** A Project with policies, examples, and instructions. Team edit rights. Sonnet as the default. An evaluation checklist inside the instructions.
- **Pattern B — Landscape to Decision:** Research with citations. Then a Domain 03 claim map. Then a decision memo Artifact.
- **Pattern C — Intake Desk:** A Connector reads a mailbox or form. Claude drafts ticket fields. A human confirms. Then the system updates. The human gate before the write is the point.
- **Pattern D — Playbook Skill:** A Skill holds the procedure (for example, incident updates). The Project holds severity definitions.
- **Pattern E — Org Brain:** Enterprise Search for "what did we decide," with citations back to Slack or Drive.
- **Pattern F — Data-to-Deliverable:** You upload data. Code execution computes totals, trends, and charts. Then it generates the Excel, PDF, or slides deliverable. A human reviews numbers against the source. Use this pattern when the core of the workflow is *computation or file production*, not prose. Code execution and file creation are available on all plans (verified on support.claude.com, Aug 2026).

Compose patterns. One feature is rarely a full solution. Exam answers that name a careful composition score better than answers that name one feature as a complete solution.

### 3. Augment vs redesign

**Augment:** Keep the process. Insert Claude at draft, retrieve, or compute steps. Change-management cost stays low. This is the default for regulated or high-stakes flows. Add audit trails.
**Redesign:** Remove steps that AI and connectors make unnecessary. Payoff is higher. Redesign needs change management, training, and clear ownership.

Decision logic: High manual waste, low risk, and willing owners make a redesign candidate. For regulated, external-facing, or irreversible flows, augment first with human gates. Earn redesign with pilot evidence. Never design full autopilot for customer-facing sends. Draft plus human send is the exam-correct shape.

### 4. Research, planning, and process optimization

Use Research for the external landscape during planning. Validate Research per Domain 03 before it informs decisions. Use Projects to keep the plan context. Use code execution to quantify ("which step consumes the hours?"). Optimization targets follow measurement, not guesses. First measure time-to-draft, revision cycles, and error types. Then redesign the worst step with the smallest correct change.

### 5. Stakeholder communication (value AND limitations — blueprint-explicit)

Write a one-page brief: problem, proposed workflow diagram, human gates, data flows, metrics, risks, and the request. Demonstrate with real messy inputs, not polished sample data. A *caught* hallucination builds more trust than a perfect run. Always include hallucination risk and the review SOP. Include data boundaries (what may enter which plan or feature, and what may not — Domain 06). Include what Claude should not own (final decisions, regulated judgment). Include success metrics and rollback.

Objection handling: "It will replace us" → show augmentation metrics and how you move hours to higher-value work. "It hallucinates" → show the evaluation SOP and error rates after review. "Data leaves our systems" → show plan controls, connector auth scopes, and org policies. Claims of value without limits fail this domain. Fear without a pilot plan also fails this domain.

### 6. Measurement and iteration

Track time saved, revision cycles to ship, error types (factual, completeness, tone, policy), adoption, and usage-limit consumption. Use the error taxonomy to update instructions, knowledge, and Skills. Do not use it to blame users. Pilot one team. Expand after metrics pass. Treat solutions as products: version them, write a changelog, and name an owner.

Integration failure modes: stale knowledge, unclear owners, gates skipped under deadline pressure, and ignored adoption ("nobody uses the solution"). Design for the actual organization. Do not design for the organization on the slide.

### 7. Mini-cases (apply the full loop)

**Case 1 — Weekly exec digest.** Communications spends four hours each Friday to compile a digest from Slack, Drive docs, and two dashboards. Design: Connectors read the sources. A Project holds the digest template, voice examples, and inclusion criteria. Claude drafts on Friday morning. The communications lead validates claims (Domain 03) and sends. Metrics: prep time, and corrections after send. After four clean weeks, this is a redesign candidate. Remove the manual dashboard screenshot step. Export the data so code execution can chart it.

**Case 2 — RFP factory.** Sales answers 20 RFPs a quarter from a wiki that nobody updates. Solution: a curated answer library in Project knowledge (owned by sales ops, versioned). A Skill holds the answer-assembly procedure. An extraction schema covers RFP questions. A human review gate sits before submission, because answers are contractual commitments. Reject the anti-pattern "auto-fill and submit." External commitments keep a human gate.

**Case 3 — Support triage.** Goal: cut first-response time. A Connector reads the queue. Claude classifies severity and drafts responses that cite the policy Project. Agents review and send. Sev-1 routes straight to humans with a Claude-drafted context brief. The severity *decision* for high-impact tickets stays with humans. Drafting and context assembly is where Claude adds speed with safety.

**Case 4 — Board-pack analysis.** The CFO wants trend commentary from monthly exports. Use Data-to-Deliverable. Code execution computes trends and builds charts plus the PDF appendix. The controller verifies computed figures against the source. Opus or Sonnet drafts commentary *from the computed numbers only*. A human owns the narrative. Composition: code execution (math) + Project (voice, definitions) + human gate (regulated audience).

Each case practices the same loop: requirements → use-case class → pattern composition → gates → metrics → iterate.

### 8. Multiple-response pattern bank

Common wrong pairs to remove: "automate the send to save time" plus "keep everything manual forever" (gate failure plus anti-Delegation). Also "promise stakeholders zero errors" plus "lead with everything that can go wrong" (over-claim plus fear — both omit the SOP-and-metrics middle). Also "roll out org-wide immediately" plus "pilot indefinitely without expansion criteria" (no evidence plus no decision rule). Also "design a custom API microservice" (wrong track). Correct combinations pair a **composition** choice with a **communication or measurement** choice. Composition examples: Project + connector + gate. Research + validation. Code execution + review. Communication or measurement examples: you disclose limits, you define a metric, and you name an owner.

---

## Decision trees

| Situation | Action |
|---|---|
| Irreversible external action in flow | Human gate before execution. This is not optional. |
| Recurring work with shared context | Project (+ Skill if the procedure is rich) |
| Multi-source public discovery | Research (validated). For internal sources: Enterprise Search |
| Workflow core is computation / file output | Code execution pattern F |
| Stakeholder needs interactive tool | Artifact |
| Unclear requirements | Do a discovery interview and a process map first |
| Regulated / high-stakes process | Augment with gates and an audit trail. Redesign only with evidence |
| Success undefined | Define metrics before you build |
| Pilot succeeded | Expand in steps. Keep measuring. Assign an owner |

---

## Exam traps

1. Full autopilot on customer-facing sends (no human gate)
2. You skip the limitations half of stakeholder communication
3. You sell Claude as error-free, or you refuse pilots out of fear
4. You ignore change management and training on redesigns
5. API or microservice architecture answers on a product-track exam
6. No success metrics or rollback plan
7. One-feature answers where composition is needed
8. Tool-first design with no requirements or process map
9. You optimize steps that nobody measured
10. You forget adoption: solutions without an owner decay

---

## Practice Q&A (18) — scenario stems with answers and rationales

**Q1.** A director says "put Claude into our invoicing process" with no further detail. First step?
**A:** Requirements and use-case analysis. Map the process, inputs and outputs, sensitivity, and definition of done before you select features. Tool selection before requirements is the trap.

**Q2.** Design a customer-support email workflow with Claude. Exam-correct shape?
**A:** Claude drafts from Project policy knowledge. A human reviews and sends. Customer-facing sends keep a human gate.

**Q3.** Select TWO components for a weekly competitive brief solution.
**A:** A Project that holds the brief template and instructions. Research (with citation validation) for the landscape sweep.

**Q4.** Your rollout slide only lists benefits. Add What must to pass this domain's bar?
**A:** Limitations: hallucination risk and review SOP, data boundaries, what stays with humans, metrics, and rollback. Communication of limits is explicit in the blueprint.

**Q5.** Finance wants monthly variance analysis from exported CSVs, delivered as Excel. Which pattern?
**A:** Data-to-Deliverable: code execution computes variances and generates the workbook. A human checks numbers against the source before circulation.

**Q6.** When does redesign beat augmentation?
**A:** High manual waste, low risk, willing owners, and pilot evidence. Remove steps that AI makes unnecessary. Regulated flows start as augmentation with audit trails.

**Q7.** Which metric pair best shows workflow value?
**A:** Time-to-shipped-draft plus error rate after review. Speed without quality (or quality without speed) is an incomplete case.

**Q8.** Ops wants a shared interactive SOP checker teammates can click through. Feature?
**A:** An Artifact. It is a standalone interactive deliverable. Viewers consume their own limits. You still validate content first.

**Q9.** A pilot's outputs show recurring tone misses and two factual errors traced to an outdated policy PDF. Iteration?
**A:** Update the knowledge file (staleness). Add tone examples to instructions or the Skill. Fix the system that caused each miss type.

**Q10.** An exec asks "will this replace the team?" Best response frame?
**A:** Change the frame to augmentation. Show which steps Claude absorbs, the human judgment you keep, the metrics, and where the freed hours go.

**Q11.** Select TWO risks that must be disclosed to stakeholders in a solution proposal.
**A:** Hallucination and error risk with its mitigation. Data-sensitivity boundaries and handling rules.

**Q12.** The team wants to skip a pilot and roll out org-wide "to move fast." Why is this the wrong option?
**A:** There is no error taxonomy, no adoption evidence, and no rollback. Pilots keep failure local while instructions and knowledge are still raw.

**Q13.** Intake emails should become CRM tickets. Where is the gate?
**A:** Claude drafts ticket fields from the email that the connector reads. A human confirms before the CRM write. Reads can be broad. Writes are gated.

**Q14.** "What did we agree with vendor X last quarter?" keeps failing in chat. Solution?
**A:** Enterprise Search over connected Slack and Drive. This is org-internal cited lookup. It is not Research. It is not a bigger model.

**Q15.** The team re-explains a recurring RFP-response procedure (12 steps, strict order) every week. Solution component?
**A:** A Skill that holds the procedure, with the boilerplate and source material in Project knowledge.

**Q16.** How do you present a demo that builds durable trust?
**A:** Use real messy inputs. Show the evaluation step. Show a caught error with its fix. This proves the error-catching step, not only the impressive output.

**Q17.** Post-rollout, usage is high but revision cycles doubled. Diagnosis path?
**A:** Segment the error taxonomy. Find which miss types grew and which workflow steps produce them. Tighten instructions and knowledge for those steps. Measure → target → cheapest lever.

**Q18.** Select TWO signs a proposed solution is over-automated for its risk level.
**A:** Irreversible external actions execute without human confirmation. Regulated judgments (HR, legal, finance) go to Claude's output directly.

---

## Quick review checklist (pre-exam)

- [ ] Collect requirements and classify the use case (generate/transform/retrieve/decide/act) first
- [ ] Six solution patterns, including Data-to-Deliverable via code execution
- [ ] Augment vs redesign logic and change management
- [ ] Human gates on irreversible and external actions — always
- [ ] Stakeholder communication = value + limits + risks + metrics + rollback
- [ ] Pilot → error taxonomy → tighten → expand. Named owner
- [ ] No API-architecture answers. Compose product features

---

## Glossary

| Term | Meaning |
|---|---|
| **Use case classes** | Generate, transform, retrieve, decide, act. Gates scale with the last two. |
| **Augmentation** | You insert Claude into an existing process at draft, retrieve, or compute steps. |
| **Redesign** | You remove process steps that AI and connectors make unnecessary. |
| **Human gate** | Required human confirmation before irreversible or external actions. |
| **Pilot** | A bounded rollout that generates the error taxonomy before you scale. |
| **Error taxonomy** | Factual / completeness / tone / policy miss classification that drives fixes. |
| **Success metric** | A predefined measure (time-to-draft, post-review error rate, adoption). |
| **Change management** | Training, ownership, and communication that make redesigns stay in place. |
