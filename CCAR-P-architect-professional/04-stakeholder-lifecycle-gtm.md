---
title: Stakeholder Engagement, Lifecycle & GTM-Adjacent Communication
---

# 04 — Stakeholder Engagement, Lifecycle & GTM-Adjacent Communication

**CCAR-P condensed domain 4** 
**Official domain mapped here:** Stakeholder Communication & Lifecycle Management (**14%**) 
**Note:** Official **Developer Productivity & Operational Enablement (7%)** is covered in dedicated file **`05-team-enablement-operational-productivity.md`**. 
**Approx. questions from this domain:** ~9 of 63 (plus overlap with enablement scenarios)

---

## Disclaimer

Original notes on discovery, proposal framing, lifecycle/ops handoff, and value communication. This file does **not** reproduce any official sales scripts or internal playbooks. It teaches durable architect skills: stakeholder discovery, expectation management, ADR communication, and lifecycle ownership. Not affiliated with Anthropic; not legal/sales advice.

---

## Overview

More than a third of CCAR-P weight sits in “soft” domains when you combine Governance + Stakeholders + Enablement. Candidates who only grind RAG/MCP leave points on the table. This file trains you to:

1. Run structured discovery across business, technical, legal, security, and ops stakeholders 
2. Translate architecture into audience-specific narratives 
3. Frame proposals with trade-offs, risks, and success metrics 
4. Own the lifecycle from pilot → GA → improve → retire 
5. Communicate value without confidential partner GTM machinery 

Hands-on team rollout of Claude Code, CLAUDE.md, hooks, and training curricula → **file 05**.

---

## Key map

| Task | Official mapping | Good exam answer traits |
| --- | --- | --- |
| Discovery interviews | Stakeholder 14% | Constraints, success metrics, risks elicited |
| Explain trade-offs | Stakeholder 14% | Non-technical clarity + technical precision |
| Document architecture | Stakeholder 14% | ADRs, diagrams, runbook pointers |
| Manage expectations | Stakeholder 14% | Phased autonomy, known limitations |
| Lifecycle handoff | Stakeholder 14% | Owners, SLOs, feedback loops |
| Value communication | Stakeholder 14% (GTM-adjacent) | Outcomes & metrics—not hype |
| Builder enablement | → File 05 (7%) | See dedicated domain file |

---

## Part A — Stakeholder discovery

### A1. Stakeholder map

| Role | What they care about | Questions to ask |
| --- | --- | --- |
| Executive sponsor | Outcomes, risk, timeline, ROI | What decision will this change? What does success look like in 90 days? |
| Business owner | Process KPIs, UX, exceptions | Where do humans spend time? What errors hurt customers? |
| End users | Trust, speed, control | When would you reject AI help? What must never auto-send? |
| Engineering | APIs, SLOs, maintainability | What systems are source of truth? Rate limits? |
| Data | Quality, tenancy, pipelines | Freshness SLOs? PII classes? |
| Security | Threats, identity, logging | What’s the authorization boundary? |
| Legal/Privacy | Basis, notices, retention | Any regulated data? Transfer limits? |
| Compliance/Risk | Controls evidence | What audits apply? |
| Ops/SRE | Pages, runbooks, capacity | Who is on-call day-1? |
| Finance | Unit economics | Cost ceiling per transaction? |
| Sales/CS (GTM-adjacent) | Promise vs delivery | What has already been committed externally? |

### A2. Discovery workshop agenda (90 minutes)

1. Problem framing (15) — jobs to be done, pain quant 
2. Constraints wall (15) — data, latency, compliance, budget 
3. Success metrics (15) — leading/lagging KPIs 
4. Journey mapping (20) — human+AI swimlane 
5. Risk storming (15) — injection, hallucination, over-automation 
6. Decision log (10) — open questions & owners 

Output: one-pager brief that feeds architecture.

### A3. Requirements taxonomy

- **Functional:** intents, tools, languages 
- **Non-functional:** latency, availability, cost, residency 
- **Safety:** disallowed actions, HITL triggers 
- **Operability:** logging, ownership, support hours 
- **Change:** model upgrade policy 

Capture **non-goals** explicitly (“not a fully autonomous negotiator in v1”).

### A4. Anti-patterns in discovery

- Only talking to the sponsor 
- Accepting “ChatGPT but on our data” as a requirement 
- Skipping security until late 
- Ignoring already-sold promises from GTM motions 
- No quantitative baseline (can’t prove value later)

---

## Part B — Communicating architecture

### B1. Altitude switching

| Audience | Artifacts | Language |
| --- | --- | --- |
| Exec | One-slide value + risk + ask | Outcomes, timeline, investment |
| Business | Journey + exception paths | Before/after KPIs |
| Engineers | Sequence diagrams, schemas, ADRs | Interfaces, failure modes |
| Security | Data flow + threat model | Trust boundaries, controls |
| Legal | Processing description | Purposes, retention, rights |

Same system, different cuts. Exam traps: drowning execs in token graphs; hand-waving security to engineers.

### B2. Architecture Decision Records (ADRs)

Template:

- Context 
- Decision 
- Alternatives considered 
- Consequences (positive/negative) 
- Status & date 
- Related eval evidence 

Example decision: “Workflow over agent for quote validation because auditability > flexibility.”

### B3. Trade-off narratives

Use **option A/B/C** tables:

| Option | Quality | Latency | Cost | Risk | Notes |
| --- | --- | --- | --- | --- | --- |
| A Monolith prompt | Med | Best | Low | Audit weak | Reject for compliance |
| B Workflow | High | Med | Med | Low | Recommended |
| C Full agent | High? | Worst | High | Higher | Park for phase 2 |

Recommend **one**, don’t dump the table without a pick.

### B4. Expectation management scripts (original, non-confidential)

- “Phase 1 suggests; humans send. Phase 2 auto-sends only for template class T with metrics M.” 
- “Model will be wrong ~X% on slice S; we route those to experts.” 
- “We will not promise identical wording—only process SLAs and quality rates.” 

Honesty builds trust; overclaiming creates incident-class disappointment.

---

## Part C — Proposal framing (GTM-adjacent, partner limits flagged)

### C1. What you *can* write originally

Architects supporting pre-sales or internal funding should structure proposals around:

1. **Problem & baseline metrics** 
2. **Target outcomes** (efficiency, quality, risk reduction) 
3. **Solution sketch** (pattern, integrations, controls) 
4. **Phased roadmap** (pilot → GA) 
5. **Investment** (people, infra, licenses—at high level) 
6. **Risks & mitigations** 
7. **Success review date** 

### C2. What you should *not* invent

- Confidential partner discount schedules 
- Internal vendor battle cards 
- Guaranteed win rates or unpublished benchmarks as marketing claims 
- Customer logos/case details you don’t have rights to 

**Note:** Employer-provided playbooks may add firm-specific motions; this pack stays skill-oriented.

### C3. Value communication patterns

| Value story | Evidence to attach |
| --- | --- |
| Time saved | Time-motion baseline vs pilot |
| Quality lift | Error rate / edit distance |
| Risk reduction | Fewer policy breaches; audit readiness |
| Revenue assist | Conversion lift with experiment design |
| Developer velocity | See file 05 metrics |

Avoid “AI magic” slides. Prefer “instrumented learning.”

### C4. Handling oversold expectations

If GTM already promised full autonomy:

1. Acknowledge the gap without blame theater 
2. Show risk of current promise 
3. Propose phased delivery that still lands value early 
4. Align sponsor on rewritten success criteria in writing 

Architects protect long-term trust.

---

## Part D — Lifecycle management

### D1. Lifecycle stages

```
Discover → Design → Build → Pilot → GA → Operate → Improve → Retire
```

Each stage needs **exit criteria**.

| Stage | Exit criteria examples |
| --- | --- |
| Discover | Brief + metrics + risk tier signed |
| Design | ADR + threat model + eval plan |
| Build | Tests green; tools contract-tested |
| Pilot | N users; metrics vs baseline; safety probes pass |
| GA | On-call live; docs; feature flags; SLOs |
| Operate | Weekly quality review; cost within envelope |
| Improve | Prioritized backlog from feedback/evals |
| Retire | Traffic drained; data deleted per policy; postmortem |

### D2. Handoff package (build → ops)

- Architecture overview + diagrams 
- Model/prompt/index pins 
- Runbooks (latency, bad outputs, injection suspicion, cost spike) 
- Dashboards & alert thresholds 
- Known failure modes 
- Support escalation matrix 
- Change process for prompts/tools 

File 05 deepens developer/ops enablement content.

### D3. Feedback loops

Sources: user thumbs, edit distance, tickets, sales objections, security findings, eval regressions. Triage into: prompt fix, retrieval fix, product UX, training, refuse-to-automate.

### D4. Operating cadence

- Daily: error/cost/safety alerts 
- Weekly: quality slice review 
- Monthly: roadmap & risk tier revisit 
- Quarterly: red team / access review / vendor review 

### D5. Deprecation & model end-of-life

Provider model deprecations are lifecycle events. Maintain:

- Inventory of pins 
- Migration owners 
- Compat eval suite 
- Comms plan to stakeholders 

---

## Part E — Change & conflict management

### E1. Common conflicts

| Conflict | Resolution approach |
| --- | --- |
| Speed vs safety | Phased autonomy + metrics gates |
| Cost vs quality | Routing & caching before quality cuts |
| Sales promise vs eng reality | Written scope reset with sponsor |
| Central platform vs domain teams | Clear paved road + escape hatches |
| Privacy vs analytics | Aggregation/redaction design |

### E2. RACI for lifecycle (example)

| Activity | Sponsor | Architect | Eng | Security | Ops | GTM/CS |
| --- | --- | --- | --- | --- | --- | --- |
| Success metrics | A | R | C | I | I | C |
| Threat model | I | R | C | A | C | I |
| GA go/no-go | A | R | R | A | A | C |
| Customer messaging | C | C | I | C | I | R/A |

---

## Part F — Documentation standards

Minimum viable docs:

1. README for service 
2. ADR log 
3. Data flow (happy + abuse) 
4. Eval report template 
5. User-facing limitations page 
6. On-call runbooks 

Prefer living docs near code over slide decks as sole source of truth.

---

## Decision trees

### Audience pitch picker

```
Is audience accountable for budget/risk?
 YES → Lead with outcomes, risk, ask, timeline
Is audience implementing?
 YES → Lead with interfaces, failure modes, ADRs
Is audience signing legal?
 YES → Lead with data flows, retention, subprocessors
```

### Pilot vs GA gate

```
Safety probes + tenancy tests green?
Owners & on-call named?
SLOs & dashboards live?
Support macros ready?
 ALL YES → GA candidate
 ELSE → Extend pilot with explicit gaps list
```

---

## Exam traps

1. Jumping to tech without discovery metrics. 
2. One-size communication for all stakeholders. 
3. No non-goals / infinite scope. 
4. GA without ops handoff. 
5. Inventing partner-confidential GTM claims. 
6. Ignoring already-committed customer promises. 
7. Metrics only vanity (page views of chatbot). 
8. No retirement/EOL plan for models. 
9. Skipping legal/security until after build. 
10. “We’ll monitor” without named owners/cadence.

---

## Practice Q&A (26)

**Q1.** Name five stakeholder groups for an enterprise Claude app. 
**A.** e.g., sponsor, business owner, eng, security, legal (also ops, data, finance, users).

**Q2.** What belongs in discovery non-goals? 
**A.** Explicitly out-of-scope capabilities for the phase (e.g., no autonomous refunds in v1).

**Q3.** How should you present options to an exec? 
**A.** Short outcome/risk/ask; optional one trade-off table with a clear recommendation.

**Q4.** What is an ADR consequence section for? 
**A.** Recording follow-on costs/risks of the decision for future readers.

**Q5.** Pilot exit criteria example? 
**A.** Target users completed tasks with metric lift and safety probes passing.

**Q6.** What do you do if sales promised full autonomy unsafely? 
**A.** Realign sponsor on phased delivery and rewrite success criteria in writing.

**Q7.** Why capture baseline metrics before build? 
**A.** Without baseline you cannot prove value or defend ROI.

**Q8.** Name three handoff artifacts to ops. 
**A.** Runbooks, dashboards/alerts, pin inventory (also escalation matrix, known failures).

**Q9.** Vanity metric example to avoid? 
**A.** Raw chatbot message count without task success or CSAT/quality linkage.

**Q10.** Who is typically accountable for GA go/no-go safety signoff? 
**A.** Security (and sponsor overall)—per RACI; architect facilitates evidence.

**Q11.** What is altitude switching? 
**A.** Adjusting artifact depth/language to audience without changing facts.

**Q12.** Why schedule model EOL as a lifecycle event? 
**A.** Deprecations break pins; need migration owners and evals.

**Q13.** GTM-adjacent content you should not invent in study claims? 
**A.** Confidential partner discounts, battle cards, unpublished guarantee scripts.

**Q14.** What makes a good success review meeting? 
**A.** Pre-agreed metrics, open risks, decision to scale/pivot/stop.

**Q15.** How do feedback loops connect to engineering? 
**A.** Triage into prompt/retrieval/UX/training/refuse-to-automate backlog items.

**Q16.** Conflict speed vs safety—architect move? 
**A.** Phase autonomy; ship suggest-mode value early; gate auto on metrics.

**Q17.** What is a limitations page for? 
**A.** Transparent user/admin expectations; reduces misuse and support load.

**Q18.** Why include CS/Sales in discovery lightly? 
**A.** Learn external commitments and objection patterns early.

**Q19.** Minimum legal-oriented artifact? 
**A.** Processing/data-flow description with purposes and retention.

**Q20.** What does “instrumented learning” mean in value stories? 
**A.** Claims backed by measured pilot evidence, not slogans.

**Q21.** Who owns weekly quality review? 
**A.** Named role (often architect + eng + business)—must be explicit.

**Q22.** Escape hatch in platform paved road? 
**A.** Documented way for teams to deviate with extra review—not silent forks.

**Q23.** Why write non-functional requirements early? 
**A.** Latency/residency/cost reshape architecture; late discovery causes rework.

**Q24.** Retire stage must include what data step? 
**A.** Drain traffic and delete/retain data per policy (indexes/logs included).

**Q25.** Best response to “just make it like ChatGPT”? 
**A.** Reframe into jobs-to-be-done, constraints, and measurable outcomes.

**Q26.** Where are Claude Code rollout details? 
**A.** File `05-team-enablement-operational-productivity.md`.

---

## Checklist

- [ ] I can run a 90-minute discovery workshop structure 
- [ ] I can switch altitude across exec/eng/security/legal 
- [ ] I can write an ADR with alternatives and consequences 
- [ ] I can frame a phased proposal without partner-confidential fluff 
- [ ] I can list lifecycle exit criteria including retire 
- [ ] I can build an ops handoff package outline 
- [ ] I can reset oversold autonomy expectations constructively 
- [ ] I can define operating cadences and owners 
- [ ] I know GTM partner-content limits for this pack 
- [ ] I know enablement deep content is in file 05 

---

## Glossary

| Term | Meaning |
| --- | --- |
| Discovery | Structured requirements & constraint elicitation |
| ADR | Architecture Decision Record |
| Non-goals | Explicit out-of-scope items |
| Baseline | Pre-AI metric for comparison |
| Pilot | Limited release to learn under controls |
| GA | General availability |
| Handoff package | Artifacts transferring build → operate |
| Paved road | Supported standard platform path |
| Escape hatch | Approved deviation process |
| Value story | Outcome narrative backed by evidence |
| RACI | Responsibility assignment matrix |
| EOL | End of life (e.g., model deprecation) |
| GTM | Go-to-market |
| Expectation management | Aligning promises to feasible delivery |
| Instrumented learning | Measurement-driven iteration |

---

## Part G — Meeting patterns & facilitation

### G1. Working backward from the press release (internal)

Draft a one-paragraph internal “launch note” early: who benefits, what changes, what won’t. If stakeholders cannot agree on that paragraph, architecture will thrash.

### G2. Decision hygiene

- Single decision owner 
- Deadline 
- Inputs required 
- Reversible vs irreversible classification 
- Document in ADR within 48 hours 

### G3. Stakeholder resistance types

| Resistance | Underlying need | Move |
| --- | --- | --- |
| Fear of job loss | Role clarity | Redesign job to oversight/exception handling |
| Fear of liability | Accountability | HITL + audit + clear owners |
| Skepticism from past AI hype | Evidence | Small pilot with ruthless metrics |
| Security veto culture | Control | Invite security as co-designer early |
| Shadow IT AI use | Speed | Paved road that’s easier than shadow tools |

### G4. Comms plan for incidents involving AI

Prewrite: who speaks, what we disclose, how we pause the feature flag, how we update customers. Pair with file 03 IR playbooks.

### G5. Training stakeholders (light)

Business users need “how to ask / how to verify / when to escalate,” not model theory. Deep builder training → file 05.

### G6. Extended scenarios

**Scenario:** Procurement wants AI to auto-email vendors negotiating price. 
**Stakeholder move:** Convene legal+security+procurement; propose suggest-mode with approval; define savings metric; refuse unbounded autonomy in v1.

**Scenario:** Board asks for “AI strategy KPI.” 
**Move:** Portfolio view—% use cases with eval gates, incident count, value realized vs target, risk tier coverage—not a single vanity chatbot score.

### G7. Q&A 27–35

**Q27.** Why draft an internal launch paragraph early? 
**A.** Forces agreement on who benefits and what won’t change before build thrash.

**Q28.** Reversible vs irreversible decision—example of irreversible? 
**A.** Autonomously emailing legally binding commitments; migrating prod data without rollback.

**Q29.** How do you respond to security-as-late-gate? 
**A.** Bring security into discovery/design; share threat model drafts early.

**Q30.** Shadow IT AI—platform response? 
**A.** Make paved road faster/safer than unsanctioned tools; educate; monitor.

**Q31.** Board-level AI KPI examples? 
**A.** Eval-gated use-case %, realized value vs target, AI incident rate, risk-tier coverage.

**Q32.** What training do business users need first? 
**A.** Ask/verify/escalate behaviors and limitations—not transformer internals.

**Q33.** Who speaks in a public AI content incident? 
**A.** Pre-assigned comms owner with legal/security input—not an ad-hoc engineer tweet.

**Q34.** Why classify decisions by reversibility? 
**A.** Irreversible ones need more evidence, HITL, and higher altitude approval.

**Q35.** Procurement auto-negotiate email—first workshop output? 
**A.** Written non-goal of autonomy + approval workflow + savings metric.

---



---

## Part H — Narrative kits (original, reusable)

### H1. Executive one-pager skeleton

1. **Problem:** quantified pain 
2. **Proposal:** pattern in one sentence 
3. **Value:** 2–3 metrics with targets 
4. **Risk:** top 3 with mitigations 
5. **Ask:** funding/people/decision 
6. **Checkpoint:** date for go/no-go 

### H2. Security review talking points

- Data classes in prompts/logs/indexes 
- Tool privilege diagram 
- Injection storyboard 
- Eval gates for safety 
- Incident severity mapping 
- Vendor/MCP list 

### H3. Customer success enablement (non-confidential)

CS needs: limitation cards, escalation matrix, “how to verify answers,” when to pull engineers. Do not invent partner discount motions.

### H4. Roadmap communication

Show **capability unlocks** tied to controls: “Citations+eval gate unlocks external FAQ”; “HITL refunds unlock finance pilot.” Avoid roadmap of model brand names alone.

### H5. Q&A 36–42

**Q36.** What are the six parts of an exec one-pager? 
**A.** Problem, proposal, value, risk, ask, checkpoint.

**Q37.** Why tie roadmap unlocks to controls? 
**A.** Makes safety/eval investment visible as value enablers, not blockers.

**Q38.** What does CS need besides feature lists? 
**A.** Limitation cards, verification guidance, escalation paths.

**Q39.** Injection storyboard purpose in security reviews? 
**A.** Make indirect injection concrete with a step-by-step abuse path.

**Q40.** What should not appear in this pack’s GTM claims? 
**A.** Confidential partner playbooks, unpublished guarantees, unauthorized case studies.

**Q41.** Capability unlock example? 
**A.** External FAQ only after citation precision gate passes.

**Q42.** Why include a checkpoint date in the ask? 
**A.** Forces decision hygiene and prevents endless pilot limbo.



---

## Part I — Lifecycle artifacts library (expand)

### I1. Pilot charter fields

- Hypothesis 
- Population & exclusions 
- Duration 
- Metrics & thresholds to scale 
- Safety stop conditions 
- Roles (sponsor, DRI, security contact) 
- Communication plan 

### I2. GA readiness scorecard (example weights)

| Area | Weight | Evidence |
| --- | --- | --- |
| Quality evals | 25% | Report ≥ thresholds |
| Safety probes | 25% | Red-team closed or accepted |
| Ops readiness | 20% | Runbooks + on-call drill |
| Privacy/legal | 15% | Signoffs |
| Support enablement | 10% | Macros + training |
| Cost envelope | 5% | Forecast vs budget |

Fail any critical safety/privacy item → no GA regardless of average.

### I3. Stakeholder update email (original template)

Subject: [Pilot] Week N outcomes 
Body: metric table vs baseline; top incidents; decision needed; next experiment. Keep to one screen.

### I4. Workshop facilitation tips

- Parking lot for out-of-scope 
- Timebox architects explaining transformers 
- Capture verbatim user “never auto-…” quotes—they become HITL requirements 
- End with owners + dates, not vibes 

### I5. Cross-cultural / multi-region stakeholders

Clarify residency early; local champions; language eval slices; don’t assume US-only legal frames for global rollouts.

### I6. Q&A 43–50

**Q43.** Name four pilot charter fields. 
**A.** Hypothesis, metrics/thresholds, safety stops, roles (etc.).

**Q44.** Can high quality scores override failed safety probes for GA? 
**A.** No—critical safety/privacy fails block GA.

**Q45.** What belongs in a weekly pilot email? 
**A.** Metrics vs baseline, incidents, decision ask, next experiment.

**Q46.** Why capture “never auto” user quotes? 
**A.** They translate directly into HITL/product rules.

**Q47.** Multi-region discovery must ask what early? 
**A.** Data residency and local regulatory constraints.

**Q48.** Parking lot technique prevents what? 
**A.** Scope explosion derailing the workshop agenda.

**Q49.** Who is DRI in a pilot charter? 
**A.** Directly responsible individual for day-to-day delivery.

**Q50.** Why include support enablement in GA scorecard? 
**A.** Without macros/training, production creates chaos and shadow process.

*Enablement, Claude Code org rollout, runbooks-as-training → `05`.*


## Part J — Extended lifecycle playbooks

### J1. From discovery notes to backlog

Convert workshop outputs into epics:

- Epic: Retrieval tenancy hardening 
- Epic: Citation UX 
- Epic: HITL queue v1 
- Epic: Eval harness CI 
- Epic: On-call runbooks 

Each epic links to a metric and a risk tier control. This prevents “random prompt tweaks” masquerading as delivery.

### J2. Steering committee pack

Monthly 30 minutes: KPI dashboard, top risks, decision requests, budget burn. Pre-read only; meeting is for decisions. Architects who narrate for 25 minutes fail the committee.

### J3. Internal FAQ for skeptical leaders

**Q:** Will this replace my team? 
**A:** Design targets toil reduction and exception-centered roles; staffing decisions are leadership’s, but architecture should make oversight workable.

**Q:** Can we trust answers? 
**A:** Trust is measured—citation rates, eval slices, HITL on critical paths—not assumed.

**Q:** Why not ship the demo? 
**A:** Demos lack tenancy tests, on-call, and abuse cases; GA scorecard exists for a reason.

### J4. Partner-limit reminder box

> This pack deliberately stops at universal architect skills: discovery, framing, lifecycle, and honest value communication. Do not treat anything here as official sales doctrine.

### J5. Post-GA value review (day 60)

Return to baseline metrics; publish a short “value realized” memo; decide invest/maintain/sunset. Without this, zombies accumulate.

### J6. Additional Q&A 51–58

**Q51.** How do workshop notes become engineering work? 
**A.** Epics linked to metrics and controls—not orphan prompt tweaks.

**Q52.** What is a steering pack for? 
**A.** Decision-making with pre-read KPIs/risks—not architecture lectures.

**Q53.** Best answer to “ship the demo”? 
**A.** Demo ≠ GA; cite missing gates on the scorecard.

**Q54.** Day-60 review purpose? 
**A.** Measure realized value; invest/maintain/sunset deliberately.

**Q55.** How to answer replacement fears? 
**A.** Focus on toil/exceptions; avoid speculative HR promises.

**Q56.** Why link epics to risk tier controls? 
**A.** Keeps safety work visible in the same backlog as features.

**Q57.** What is a zombie AI app? 
**A.** Running system without owners, metrics, or value review.

**Q58.** Where is developer enablement detailed? 
**A.** File 05—do not dilute stakeholder answers with IDE settings unless asked.


## Part K — Communication worksheets

### K1. Constraint wall prompts (use in workshops)

- Hard deadline? What happens if we miss? 
- Data that must never leave region X? 
- Actions that always need a human name on them? 
- Maximum acceptable wrong-answer rate for slice S? 
- Support hours and language coverage? 
- Existing contractual SLAs with customers? 

### K2. Trade-off sentence stems (practice saying these)

- “We recommend B because auditability outweighs flexibility given constraint C.” 
- “We can hit latency L only if we route easy traffic to model M and reserve N for hard cases.” 
- “Phase 1 will not auto-send; that unlocks after metric X ≥ T for two weeks.” 
- “Cost target requires cache hit rate ≥ H and top-k ≤ K.” 

### K3. Lifecycle RACI quiz yourself

Pick a random activity—prompt change, index rebuild, customer apology, MCP add—and assign RACI without looking. Then check against your organization’s real names. Exam scenarios expect you to know *which function* is accountable even if names differ.

### K4. End-card

Stakeholder domain = discovery quality + altitude switching + phased honesty + lifecycle ownership. GTM-adjacent work stays non-confidential: value, risks, phases, metrics. Enablement mechanics live in file 05.


## Part L — Full discovery questionnaire bank (condensed)

**Business:** What breaks if the AI is wrong? What is the cost of delay today? Which step is pure toil? 
**Users:** When would you ignore the suggestion? What must remain human-branded? 
**Eng:** Rate limits? Idempotency already in APIs? Sync vs async constraints? 
**Data:** Freshness? Label quality? Known poison documents? 
**Security:** Crown-jewel systems in scope? Existing DLP? 
**Legal:** Automated decisioning rules? Retention schedule? 
**Ops:** Who pages at 2am? What’s the rollback muscle memory? 
**Finance:** Unit-cost ceiling? CapEx vs OpEx preferences? 
**GTM/CS:** What was already promised in writing? Top objections?

Running even half of these prevents the classic “solution looking for a problem” failure mode that CCAR-P scenarios punish.


## Part M — Handoff email template (original)

Subject: Handoff — <feature> to operations 
Attach: diagrams, ADR index, pin list, runbooks, dashboard links, known issues, escalation matrix. 
Body checklist: SLO summary; feature flags; how to roll back model/prompt/index; where eval reports live; next quality review date; DRI during hypercare (usually two weeks). 

Hypercare means the build team stays in the support path while ops gains reps—then steps back per agreement. Skipping hypercare is how “lifecycle” fails silently.

### Closing card for file 04

You now have discovery banks, altitude switching, proposal framing within partner-limit honesty, lifecycle gates, and handoff discipline. Pair with file 05 when stems mention Claude Code, CLAUDE.md, hooks, or org-wide developer tooling.

### One more worked stem

Stem flavor: “Sponsor wants GA next month after a flashy demo; security has not reviewed; no on-call.” 
Strong answer: present GA scorecard gaps; propose dated pilot extension; secure review + runbook drill as exit criteria; reset external promises with sponsor—not silent shipping.

### Expectation-reset paragraph (memorize structure)

“We can deliver measurable value by date D in suggest-mode with controls C. Full autonomy remains a phase-2 unlock after metrics M hold for period P. Shipping autonomy now would accept risk R without compensating controls. Please confirm the revised success criteria in writing so engineering, security, and GTM share one plan.”

That structure—value now, unlock later, risk named, written alignment—answers most “pressure to over-automate” stakeholder stems without partner-specific playbooks.

*(File continues — deep-dive Part N follows below.)*

Hypercare without a named DRI is theater; name the person and the end date when you hand off.

Keep a living decision log; when stakeholders revisit settled choices, point to the ADR date and evidence rather than re-litigating from memory.


---

## Part N — Primary-study deep dive: Discovery that survives contact with reality

> **Scope note (read first):** This pack teaches **original** discovery, lifecycle, change management, and value framing skills only — no invented pricing, confidential win themes, or official lesson text. If an exam stem smells like a “sales motion,” answer with durable architect behaviors: clarify commitments, align SLAs, document trade-offs, phase delivery.

### N1. Discovery depth model (five passes)

1. **Problem pass** — jobs-to-be-done, pain quant, current workaround. 
2. **Constraint pass** — data, latency, residency, budget, unions/works councils, brand. 
3. **Success pass** — leading/lagging metrics, decision rights, time box. 
4. **Risk pass** — safety, compliance, operational blast radius, reputational. 
5. **Change pass** — who loses power/status, training load, process redesign.

Skipping the change pass is why “technically perfect” pilots die.

### N2. Stakeholder interview bank (condensed, original)

**Executive:** What decision improves if this works? What would make you stop the program? 
**Business owner:** Which step burns minutes? What error is unacceptable? 
**Users:** When do you distrust AI? What must stay manual? 
**Security:** What’s the identity boundary? What’s your nightmare ticket? 
**Legal/Privacy:** Data classes? Transfer? Retention? Notices? 
**Ops:** Who pages at 2am? What’s the degrade mode? 
**Finance:** Fully loaded cost ceiling per transaction? 
**Sales/CS (GTM-adjacent):** What was already promised externally?

### N3. Requirement quality bar

A requirement is exam-ready when it has: owner, metric, constraint, priority, and test idea. 
“Make it smart” is not a requirement. “p95 < 3s for FAQ path; HITL for refunds > $50; GDPR erasure within policy SLO” is.

### N4. Anti-patterns (discovery)

| Anti-pattern | Why it fails | Fix |
| --- | --- | --- |
| Single stakeholder proxy | Hidden vetoes later | Map power + impacted users |
| Solutioneering first | Misses real job | Problem framing before pattern |
| Ignoring shadow process | Users bypass system | Observe real work |
| No non-goals | Scope cream | Explicit out-of-scope list |
| Verbal-only decisions | Amnesia | Decision log / ADR |

---

## Part O — Value framing without partner playbooks

### O1. Value hypothesis template (original)

```
For [user segment] who [problem],
the Claude solution will [capability]
so that [outcome metric] moves from [baseline] to [target]
within [time], under [constraints],
with residual risks [list] accepted by [owner].
```

### O2. Value pillars → evidence

| Pillar | Claim example | Evidence you must plan |
| --- | --- | --- |
| Efficiency | Handle time −30% | Time-motion + production A/B |
| Quality | Error rate −40% | Golden set + human audit |
| Transformation | New self-serve channel | Adoption + containment rate |
| Cost | $ / ticket −25% | Unit economics dashboard |
| Performance | p95 SLA met | Latency SLO burn alerts |

### O3. Expectation management (durable scripts)

- “Pilot proves learning rate, not final accuracy.” 
- “Autonomy expands with eval evidence, not calendar dates alone.” 
- “Model upgrades are changes—they need regression gates.” 
- “Citations reduce hallucination risk; they do not eliminate it.” 

### O4. Oversell recovery

If Sales promised “fully autonomous legal advice”:
1. Document the external commitment factually. 
2. Reframe to compliant scope (draft + attorney HITL). 
3. Offer phased roadmap with exit criteria. 
4. Escalate sponsor decision on residual risk—do not quietly ship unsafe scope.

---

## Part P — Lifecycle & change management deep dive

### P1. Stage exit criteria (expand)

| Stage | Exit when… |
| --- | --- |
| Discovery | Brief + metrics + constraints signed |
| Design | ADR + threat notes + eval plan reviewed |
| Build | Contracts tested; tracing on |
| Pilot | Success criteria measured; risks logged |
| GA | Scorecard green; on-call ready; docs handed off |
| Operate | SLO reviews running; feedback loop owned |
| Iterate | Backlog prioritized from traces + users |
| Retire | Migration + model deprecation complete |

### P2. Change management levers

- **Comms:** why change, what stays human, how to get help 
- **Training:** role-based (users ≠ reviewers ≠ engineers) 
- **Incentives:** don’t reward raw automation rate if quality collapses 
- **Champions:** early adopters coach peers 
- **Feedback:** visible “fix this answer” with responses 

### P3. Conflict patterns & resolutions

| Conflict | Typical resolution |
| --- | --- |
| Security vs speed | Paved road + time-boxed exceptions |
| Product vs legal | Scoped MVP; counsel on notices |
| Ops vs feature factory | Error budgets bind releases |
| Users vs automation | Copilot first; earn autonomy |
| Sponsor vs reality | Data-backed reset; options A/B/C |

### P4. Handoff package (must-not-forget)

Architecture diagrams, ADRs, tool inventory + authZ model, eval dashboards, runbooks, HITL staffing model, data retention map, model pin/upgrade policy, known limitations, RACI, support escalation matrix.

### P5. Monitoring & iteration cadence

- Daily: page-worthy SLO burns 
- Weekly: eval slice + top failure themes 
- Monthly: cost unit economics + roadmap 
- Quarterly: risk tier revisit + access review 

---

## Part Q — Architecture communication drills

### Q1. Altitude switching exercise

Same decision (“workflow not agent”):
- **Exec:** “We chose a predictable staged process so auditors can see each check—faster to trust, slightly less flexible.” 
- **Engineer:** “Deterministic DAG with schema validators between extract→policy→recommend; agent only for open research subtask.” 
- **Risk:** “Side effects gated; HITL above threshold; full tool audit.”

### Q2. ADR minimum fields

Context, decision, alternatives rejected, consequences, status, date, owners, review trigger (e.g., model deprecation).

### Q3. SLA negotiation worksheet

| Path | Latency | Quality | Autonomy | Support hours |
| --- | --- | --- | --- | --- |
| FAQ auto | | | | |
| Complex assist | | | | |
| High-impact | | | | |

Never accept one global SLA for mixed paths.

---

## Part R — GTM-adjacent boundaries (exam hygiene)

**In scope (original skills):** discovery, value hypotheses, phased proposals, risk talk-tracks, lifecycle ownership, stakeholder updates. 
**Out of scope for this pack:** confidential discounting, internal battle cards, non-public packaging, fabricated partnership tiers. 
If a question asks how to “position Claude vs competitor X using partner kit,” choose answers about **customer outcomes, constraints, and proof via evals**—not invented competitive intel.

---

## Part S — Production checklists (lifecycle)

### S1. Discovery complete?

- [ ] Stakeholder map with veto powers 
- [ ] Metrics with baselines 
- [ ] Constraints wall photographed/documented 
- [ ] Non-goals listed 
- [ ] Risk storm captured 
- [ ] Decision log started

### S2. Ready to propose?

- [ ] Value hypothesis filled 
- [ ] Pattern + model rationale 
- [ ] Phased autonomy plan 
- [ ] Cost envelope 
- [ ] Residual risks + owners 
- [ ] Partner-limit: no confidential GTM claims invented

### S3. Ready to hand off?

- [ ] Handoff package complete 
- [ ] On-call named 
- [ ] Feedback loop tooled 
- [ ] Training delivered 
- [ ] GA scorecard archived

---

## Part T — Failure modes in stakeholder/lifecycle work

| Failure | Symptom | Fix |
| --- | --- | --- |
| Pilot purgatory | No GA criteria | Scorecard + date + owner |
| Shadow commitments | Security surprised | Early risk stakeholders |
| Metric theater | Vanity dashboards | Tie to decisions/money/risk |
| Doc drift | Ops can’t page | Runbook tests in drills |
| Change revolt | Users bypass AI | Copilot UX + champions |
| Model EOL panic | Break in prod | Pin + upgrade playbook |

---

## Part U — Extended Q&A (59–68)

**Q59.** First workshop output that feeds architecture? 
**A.** A **one-pager brief** with problem, constraints, metrics, risks—not a model choice debate.

**Q60.** Exec asks “when will it be fully autonomous?” Best answer shape? 
**A.** Autonomy **gated by eval evidence and risk tier**, with phased milestones—not a blind calendar promise.

**Q61.** Select TWO handoff must-haves: eval dashboards, on-call RACI, partner discount sheet, meme channel. 
**A.** Eval dashboards and on-call RACI.

**Q62.** Sales promised unsafe scope. Architect should… 
**A.** Document, reframe to compliant phased scope, escalate sponsor—**not** silently ship.

**Q63.** Why include ops in discovery? 
**A.** Degrade modes, paging, capacity—and to prevent unsupportable GA.

**Q64.** ADR alternative rejected field matters because… 
**A.** Exam and real reviews test whether you **considered** agents vs workflows etc.

**Q65.** Change management missing—likely outcome? 
**A.** Users bypass the system; ROI never materializes.

**Q66.** No official GTM playbook available. What still works on CCAR-P? 
**A.** Structured discovery, trade-off communication, lifecycle artifacts, value metrics.

**Q67.** GA scorecard red on HITL staffing—ship anyway? 
**A.** **No**—staff or reduce autonomy scope first.

**Q68.** Select THREE discovery anti-patterns: solutioneering first, single proxy stakeholder, no non-goals, writing ADRs, measuring baselines. 
**A.** Solutioneering first, single proxy, no non-goals.

---

## Part V — Rapid review (Stakeholder 14%)

- Discovery before design; change pass included. 
- Requirements need metrics and owners. 
- Altitude-switch communications; keep ADRs. 
- Value = hypothesis + evidence plan. 
- Lifecycle exit criteria beat vibes. 
- Flag partner-only GTM limits; stay original. 
- Handoff = docs + owners + evals + runbooks. 
- Reset oversells; don’t ship unsafe promises.

*Enablement mechanics live in `05`. Safety talk-tracks pair with `03`.*
