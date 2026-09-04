# 04 — Stakeholder Engagement, Lifecycle & GTM-Adjacent Communication — Simplified Technical English

**CCAR-P condensed domain 4**
**Official domain mapped here:** Stakeholder Communication & Lifecycle Management (**14%**)
**Note:** Official **Developer Productivity & Operational Enablement (7%)** is in the dedicated file **`05-team-enablement-operational-productivity.md`**.
**Approx. questions from this domain:** about 9 of 63 (plus overlap with enablement scenarios)

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names Claude, MCP, ADR, GTM, GA stay as written. These notes are original study notes. They are not official sales scripts. They are not affiliated with Anthropic. They are not legal advice or sales advice.

---

## Disclaimer

These notes cover discovery, proposal framing, lifecycle and operations handoff, and value communication. This file does **not** copy official sales scripts or internal playbooks. It teaches durable architect skills: stakeholder discovery, expectation management, ADR communication, and lifecycle ownership. These notes are not affiliated with Anthropic. They are not legal advice. They are not sales advice.

---

## Overview

More than one third of CCAR-P weight sits in communication domains when you combine Governance + Stakeholders + Enablement. Candidates who study only RAG/MCP lose exam points. This file trains you to:

1. Run structured discovery across business, technical, legal, security, and operations stakeholders
2. Translate architecture into narratives for each audience
3. Frame proposals with trade-offs, risks, and success metrics
4. Own the lifecycle from pilot → GA → improve → retire
5. Communicate value without confidential partner GTM machinery

For hands-on team rollout of Claude Code, CLAUDE.md, hooks, and training curricula, use **file 05**.

---

## Key map

| Task | Official mapping | Good exam answer traits |
| --- | --- | --- |
| Discovery interviews | Stakeholder 14% | You get constraints, success metrics, and risks |
| Explain trade-offs | Stakeholder 14% | Non-technical clarity plus technical precision |
| Document architecture | Stakeholder 14% | ADRs, diagrams, and runbook pointers |
| Manage expectations | Stakeholder 14% | Phased autonomy and known limitations |
| Lifecycle handoff | Stakeholder 14% | Owners, SLOs, and feedback loops |
| Value communication | Stakeholder 14% (GTM-adjacent) | Outcomes and metrics. Do not use hype |
| Builder enablement | → File 05 (7%) | See the dedicated domain file |

---

## Part A — Stakeholder discovery

### A1. Stakeholder map

| Role | What they care about | Questions to ask |
| --- | --- | --- |
| Executive sponsor | Outcomes, risk, timeline, ROI | What decision will this change? What does success look like in 90 days? |
| Business owner | Process KPIs, UX, exceptions | Where do humans spend time? What errors hurt customers? |
| End users | Trust, speed, control | When would you reject AI help? What must never auto-send? |
| Engineering | APIs, SLOs, maintainability | What systems are the source of truth? What are the rate limits? |
| Data | Quality, tenancy, pipelines | What are the freshness SLOs? What are the PII classes? |
| Security | Threats, identity, logging | What is the authorization boundary? |
| Legal/Privacy | Basis, notices, retention | Is there regulated data? What are the transfer limits? |
| Compliance/Risk | Controls evidence | What audits apply? |
| Ops/SRE | Pages, runbooks, capacity | Who is on-call on day 1? |
| Finance | Unit economics | What is the cost ceiling per transaction? |
| Sales/CS (GTM-adjacent) | Promise vs delivery | What has already been committed externally? |

### A2. Discovery workshop agenda (90 minutes)

1. Problem framing (15) — jobs to be done, quantified pain
2. Constraints wall (15) — data, latency, compliance, budget
3. Success metrics (15) — leading and lagging KPIs
4. Journey mapping (20) — human+AI swimlane
5. Risk storming (15) — injection, hallucination, over-automation
6. Decision log (10) — open questions and owners

Output: a one-page brief that feeds architecture.

### A3. Requirements taxonomy

- **Functional:** intents, tools, languages
- **Non-functional:** latency, availability, cost, residency
- **Safety:** disallowed actions, HITL triggers
- **Operability:** logging, ownership, support hours
- **Change:** model upgrade policy

Record **non-goals** in explicit form (example: “not a fully autonomous negotiator in v1”).

### A4. Anti-patterns in discovery

- You talk only to the sponsor
- You accept “ChatGPT but on our data” as a requirement
- You skip security until late
- You ignore promises that GTM motions already sold
- You have no quantitative baseline (you cannot prove value later)

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

The system stays the same. You show different views. Exam traps: you give executives too much token-graph detail. You give engineers only vague security statements.

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
| C Full agent | High? | Worst | High | Higher | Keep for phase 2 |

Recommend **one** option. Do not show the table without a selection.

### B4. Expectation management scripts (original, non-confidential)

- “Phase 1 suggests. Humans send. Phase 2 auto-sends only for template class T with metrics M.”
- “Model will be wrong ~X% on slice S. We route those to experts.”
- “We will not promise identical wording—only process SLAs and quality rates.”

Honesty builds trust. Overclaiming creates disappointment at incident scale.

---

## Part C — Proposal framing (GTM-adjacent, partner limits flagged)

### C1. What you *can* write originally

Architects who support pre-sales or internal funding structure proposals around:

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
- Customer logos and case details that you do not have rights to use

**Note:** Employer-provided playbooks can add firm-specific motions. This pack stays skill-oriented.

### C3. Value communication patterns

| Value story | Evidence to attach |
| --- | --- |
| Time saved | Time-motion baseline vs pilot |
| Quality lift | Error rate / edit distance |
| Risk reduction | Fewer policy breaches. Audit readiness |
| Revenue assist | Conversion lift with experiment design |
| Developer velocity | See file 05 metrics |

Avoid “AI unspecified mechanism” slides. Prefer “instrumented learning.”

### C4. Handling oversold expectations

If GTM already promised full autonomy:

1. Acknowledge the gap. Do not run a public blame session
2. Show the risk of the current promise
3. Propose phased delivery that still lands value early
4. Align the sponsor on rewritten success criteria in writing

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
| Build | Tests green. Tools contract-tested |
| Pilot | N users. Metrics vs baseline. Safety probes pass |
| GA | On-call live. Docs. Feature flags. SLOs |
| Operate | Weekly quality review. Cost within envelope |
| Improve | Prioritized backlog from feedback/evals |
| Retire | Traffic drained. Data deleted per policy. Postmortem |

### D2. Handoff package (build → ops)

- Architecture overview + diagrams
- Model/prompt/index pins
- Runbooks (latency, bad outputs, injection suspicion, cost spike)
- Dashboards & alert thresholds
- Known failure modes
- Support escalation matrix
- Change process for prompts/tools

File 05 deepens developer and operations enablement content.

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

Prefer living docs near code. Do not use slide decks as the only source of truth.

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

1. You jump to tech without discovery metrics.
2. You use one communication style for all stakeholders.
3. You have no non-goals / infinite scope.
4. You go to GA without an operations handoff.
5. You invent partner-confidential GTM claims.
6. You ignore customer promises that are already committed.
7. You track only vanity metrics (page views of a chatbot).
8. You have no retirement/EOL plan for models.
9. You skip legal/security until after build.
10. You say “we will monitor” without named owners and cadence.

---

## Practice Q&A (26)

**Q1.** Name five stakeholder groups for an enterprise Claude app.
**A.** Examples: sponsor, business owner, engineering, security, and legal. Also include operations, data, finance, and users.

**Q2.** What belongs in discovery non-goals?
**A.** Capabilities that are explicitly out of scope for the phase (example: no autonomous refunds in v1).

**Q3.** How should you present options to an exec?
**A.** Use a short outcome/risk/ask. You can add one trade-off table with a clear recommendation.

**Q4.** What is an ADR consequence section for?
**A.** You record follow-on costs and risks of the decision for future readers.

**Q5.** Pilot exit criteria example?
**A.** Target users complete tasks with metric lift, and safety probes pass.

**Q6.** What do you do if sales promised full autonomy unsafely?
**A.** Realign the sponsor on phased delivery. Rewrite success criteria in writing.

**Q7.** Why capture baseline metrics before build?
**A.** Without a baseline you cannot prove value or defend ROI.

**Q8.** Name three handoff artifacts to ops.
**A.** Runbooks, dashboards/alerts, and pin inventory (also escalation matrix and known failures).

**Q9.** Vanity metric example to avoid?
**A.** Raw chatbot message count without task success or CSAT/quality linkage.

**Q10.** Who is typically accountable for GA go/no-go safety signoff?
**A.** Security (and the sponsor overall)—per RACI. The architect helps with evidence.

**Q11.** What is altitude switching?
**A.** You adjust artifact depth and language to the audience. You do not change the facts.

**Q12.** Why schedule model EOL as a lifecycle event?
**A.** Deprecations break pins. You need migration owners and evals.

**Q13.** GTM-adjacent content you should not invent in study claims?
**A.** Confidential partner discounts, battle cards, and unpublished guarantee scripts.

**Q14.** What makes a good success review meeting?
**A.** Pre-agreed metrics, open risks, and a decision to scale, pivot, or stop.

**Q15.** How do feedback loops connect to engineering?
**A.** You triage into prompt, retrieval, UX, training, or refuse-to-automate backlog items.

**Q16.** Conflict speed vs safety—architect move?
**A.** Phase autonomy. Ship suggest-mode value early. Gate auto-send on metrics.

**Q17.** What is a limitations page for?
**A.** Transparent user and admin expectations. This reduces misuse and support load.

**Q18.** Why include CS/Sales in discovery lightly?
**A.** You learn external commitments and objection patterns early.

**Q19.** Minimum legal-oriented artifact?
**A.** A processing and data-flow description with purposes and retention.

**Q20.** What does “instrumented learning” mean in value stories?
**A.** Claims backed by measured pilot evidence, not slogans.

**Q21.** Who owns weekly quality review?
**A.** A named role (often architect + engineering + business). The owner must be explicit.

**Q22.** Escape hatch in platform paved road?
**A.** A documented way for teams to deviate with extra review. Silent forks are not allowed.

**Q23.** Why write non-functional requirements early?
**A.** Latency, residency, and cost reshape architecture. Late discovery causes rework.

**Q24.** Retire stage must include what data step?
**A.** Drain traffic and delete or retain data per policy (indexes and logs included).

**Q25.** Best response to “just make it like ChatGPT”?
**A.** Reframe into jobs-to-be-done, constraints, and measurable outcomes.

**Q26.** Where are Claude Code rollout details?
**A.** File `05-team-enablement-operational-productivity.md`.

---

## Checklist

- [ ] I can run a 90-minute discovery workshop structure
- [ ] I can switch altitude across exec/eng/security/legal
- [ ] I can write an ADR with alternatives and consequences
- [ ] I can frame a phased proposal without partner-confidential extra claims
- [ ] I can list lifecycle exit criteria including retire
- [ ] I can build an ops handoff package outline
- [ ] I can reset oversold autonomy expectations in a constructive way
- [ ] I can define operating cadences and owners
- [ ] I know GTM partner-content limits for this pack
- [ ] I know enablement deep content is in file 05

---

## Glossary

| Term | Meaning |
| --- | --- |
| Discovery | Structured collection of requirements and constraints |
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

Draft a one-paragraph internal “launch note” early: who benefits, what changes, and what will not change. If stakeholders cannot agree on that paragraph, architecture work will change without control.

### G2. Decision hygiene

- Single decision owner
- Deadline
- Inputs required
- Reversible vs irreversible classification
- Document in ADR within 48 hours

### G3. Stakeholder resistance types

| Resistance | Underlying need | Move |
| --- | --- | --- |
| Fear of job loss | Role clarity | Redesign the job to oversight and exception handling |
| Fear of liability | Accountability | HITL + audit + clear owners |
| Skepticism from past AI hype | Evidence | Small pilot with strict metrics |
| Security as a late veto | Control | Invite security as co-designer early |
| Shadow IT AI use | Speed | Paved road that is easier than shadow tools |

### G4. Comms plan for incidents involving AI

Prewrite: who speaks, what you disclose, how you pause the feature flag, and how you update customers. Pair this with file 03 IR playbooks.

### G5. Training stakeholders (light)

Business users need “how to ask / how to verify / when to escalate.” They do not need model theory. Deep builder training is in file 05.

### G6. Extended scenarios

**Scenario:** Procurement wants AI to auto-email vendors and negotiate price.
**Stakeholder move:** Convene legal, security, and procurement. Propose suggest-mode with approval. Define a savings metric. Refuse unbounded autonomy in v1.

**Scenario:** Board asks for an “AI strategy KPI.”
**Move:** Use a portfolio view. Percent of use cases with eval gates, incident count, value realized vs target, and risk tier coverage. Do not use a single vanity chatbot score.

### G7. Q&A 27–35

**Q27.** Why draft an internal launch paragraph early?
**A.** It forces agreement on who benefits and what will not change, before build work changes without control.

**Q28.** Reversible vs irreversible decision—example of irreversible?
**A.** Autonomously emailing legally binding commitments. Migrating production data without rollback.

**Q29.** How do you respond to security-as-late-gate?
**A.** Bring security into discovery and design. Share threat model drafts early.

**Q30.** Shadow IT AI—platform response?
**A.** Make the paved road faster and safer than unsanctioned tools. Educate. Monitor.

**Q31.** Board-level AI KPI examples?
**A.** Eval-gated use-case percent, realized value vs target, AI incident rate, and risk-tier coverage.

**Q32.** What training do business users need first?
**A.** Ask, verify, and escalate behaviors, plus limitations. Not transformer internals.

**Q33.** Who speaks in a public AI content incident?
**A.** A pre-assigned comms owner with legal and security input. Not an ad-hoc engineer tweet.

**Q34.** Why classify decisions by reversibility?
**A.** Irreversible ones need more evidence, HITL, and higher altitude approval.

**Q35.** Procurement auto-negotiate email—first workshop output?
**A.** A written non-goal of autonomy, plus an approval workflow, plus a savings metric.

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

CS needs: limitation cards, escalation matrix, “how to verify answers,” and when to pull engineers. Do not invent partner discount motions.

### H4. Roadmap communication

Show **capability unlocks** tied to controls: “Citations+eval gate unlocks external FAQ”. “HITL refunds unlock finance pilot.” Avoid a roadmap of model brand names alone.

### H5. Q&A 36–42

**Q36.** What are the six parts of an exec one-pager?
**A.** Problem, proposal, value, risk, ask, checkpoint.

**Q37.** Why tie roadmap unlocks to controls?
**A.** This makes safety and eval investment visible as value enablers, not blockers.

**Q38.** What does CS need besides feature lists?
**A.** Limitation cards, verification guidance, and escalation paths.

**Q39.** Injection storyboard purpose in security reviews?
**A.** Make indirect injection concrete with a step-by-step abuse path.

**Q40.** What should not appear in this pack’s GTM claims?
**A.** Confidential partner playbooks, unpublished guarantees, and unauthorized case studies.

**Q41.** Capability unlock example?
**A.** External FAQ only after the citation precision gate passes.

**Q42.** Why include a checkpoint date in the ask?
**A.** It forces decision hygiene and prevents a pilot that never ends.

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

Fail any critical safety or privacy item → no GA, regardless of average.

### I3. Stakeholder update email (original template)

Subject: [Pilot] Week N outcomes
Body: metric table vs baseline. Top incidents. Decision needed. Next experiment. Keep the email to one screen.

### I4. Workshop facilitation tips

- Parking lot for out-of-scope items
- Timebox architects who explain transformers
- Capture verbatim user “never auto-…” quotes—they become HITL requirements
- End with owners and dates, not vague feelings

### I5. Cross-cultural / multi-region stakeholders

Clarify residency early. Use local champions. Use language eval slices. Do not assume US-only legal frames for global rollouts.

### I6. Q&A 43–50

**Q43.** Name four pilot charter fields.
**A.** Hypothesis, metrics/thresholds, safety stops, roles (and related fields).

**Q44.** Can high quality scores override failed safety probes for GA?
**A.** No—critical safety or privacy fails block GA.

**Q45.** What belongs in a weekly pilot email?
**A.** Metrics vs baseline, incidents, decision ask, and next experiment.

**Q46.** Why capture “never auto” user quotes?
**A.** They translate directly into HITL and product rules.

**Q47.** Multi-region discovery must ask what early?
**A.** Data residency and local regulatory constraints.

**Q48.** Parking lot technique prevents what?
**A.** Scope explosion that derails the workshop agenda.

**Q49.** Who is DRI in a pilot charter?
**A.** The directly responsible individual for day-to-day delivery.

**Q50.** Why include support enablement in GA scorecard?
**A.** Without macros and training, production creates chaos and shadow process.

*Enablement, Claude Code org rollout, runbooks-as-training → `05`.*


## Part J — Extended lifecycle playbooks

### J1. From discovery notes to backlog

Convert workshop outputs into epics:

- Epic: Retrieval tenancy hardening
- Epic: Citation UX
- Epic: HITL queue v1
- Epic: Eval harness CI
- Epic: On-call runbooks

Each epic links to a metric and a risk tier control. This prevents “random prompt tweaks” that pretend to be delivery.

### J2. Steering committee pack

Monthly 30 minutes: KPI dashboard, top risks, decision requests, budget burn. Pre-read only. The meeting is for decisions. Architects who speak for 25 minutes do not pass the committee.

### J3. Internal FAQ for skeptical leaders

**Q:** Will this replace my team?
**A:** Design targets toil reduction and exception-centered roles. Staffing decisions belong to leadership. Architecture must make oversight workable.

**Q:** Can we trust answers?
**A:** You measure trust with citation rates, eval slices, and HITL on critical paths. Do not assume trust.

**Q:** Why not ship the demo?
**A:** Demos lack tenancy tests, on-call, and abuse cases. The GA scorecard exists for a reason.

### J4. Partner-limit reminder box

> This pack stops at universal architect skills: discovery, framing, lifecycle, and honest value communication. Do not treat anything here as official sales doctrine.

### J5. Post-GA value review (day 60)

Return to baseline metrics. Publish a short “value realized” memo. Decide invest, maintain, or sunset. Without this, unused systems stay in production.

### J6. Additional Q&A 51–58

**Q51.** How do workshop notes become engineering work?
**A.** Epics linked to metrics and controls—not orphan prompt tweaks.

**Q52.** What is a steering pack for?
**A.** Decision-making with pre-read KPIs and risks. Not architecture lectures.

**Q53.** Best answer to “ship the demo”?
**A.** Demo is not GA. Cite missing gates on the scorecard.

**Q54.** Day-60 review purpose?
**A.** Measure realized value. Invest, maintain, or sunset with intent.

**Q55.** How to answer replacement fears?
**A.** Focus on toil and exceptions. Avoid speculative HR promises.

**Q56.** Why link epics to risk tier controls?
**A.** This keeps safety work visible in the same backlog as features.

**Q57.** What is an abandoned AI app?
**A.** A running system without owners, metrics, or value review.

**Q58.** Where is developer enablement detailed?
**A.** File 05. Do not mix stakeholder answers with IDE settings unless the question asks.

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
- “Phase 1 will not auto-send. That unlocks after metric X ≥ T for two weeks.”
- “Cost target requires cache hit rate ≥ H and top-k ≤ K.”

### K3. Lifecycle RACI quiz yourself

Select a random activity—prompt change, index rebuild, customer apology, MCP add—and assign RACI. Do not look at a key first. Then check against your organization’s real names. Exam scenarios expect you to know *which function* is accountable even if names differ.

### K4. End-card

Stakeholder domain = discovery quality + altitude switching + phased honesty + lifecycle ownership. GTM-adjacent work stays non-confidential: value, risks, phases, metrics. Enablement mechanics live in file 05.


## Part L — Full discovery questionnaire bank (condensed)

**Business:** What breaks if the AI is wrong? What is the cost of delay today? Which step is pure toil?
**Users:** When would you ignore the suggestion? What must remain human-branded?
**Eng:** Rate limits? Idempotency already in APIs? Sync vs async constraints?
**Data:** Freshness? Label quality? Known poison documents?
**Security:** Highest-value systems in scope? Existing DLP?
**Legal:** Automated decisioning rules? Retention schedule?
**Ops:** Who pages at 2am? What is the rollback procedure that people already know?
**Finance:** Unit-cost ceiling? CapEx vs OpEx preferences?
**GTM/CS:** What was already promised in writing? Top objections?

If you run even half of these, you prevent the common “solution looking for a problem” failure. CCAR-P scenarios punish that failure.


## Part M — Handoff email template (original)

Subject: Handoff — <feature> to operations
Attach: diagrams, ADR index, pin list, runbooks, dashboard links, known issues, escalation matrix.
Body checklist: SLO summary. Feature flags. How to roll back model, prompt, and index. Where eval reports live. Next quality review date. DRI during hypercare (usually two weeks).

Hypercare means the build team stays in the support path while operations gains practice. Then the build team steps back per agreement. If you skip hypercare, lifecycle work fails without a clear signal.

### Closing card for file 04

You now have discovery banks, altitude switching, proposal framing within partner-limit honesty, lifecycle gates, and handoff discipline. Pair this with file 05 when questions mention Claude Code, CLAUDE.md, hooks, or org-wide developer tooling.

### One more worked stem

Stem example: "Sponsor wants GA next month after an impressive demo. Security has not reviewed. No on-call."
Strong answer: present GA scorecard gaps. Propose a dated pilot extension. Set security review and a runbook drill as exit criteria. Reset external promises with the sponsor. Do not ship in silence.

### Expectation-reset paragraph (memorize structure)

“We can deliver measurable value by date D in suggest-mode with controls C. Full autonomy remains a phase-2 unlock after metrics M hold for period P. If you ship autonomy now, you accept risk R without compensating controls. Please confirm the revised success criteria in writing so engineering, security, and GTM share one plan.”

That structure—value now, unlock later, risk named, written alignment—answers most “pressure to over-automate” stakeholder questions. You do not need partner-specific playbooks.

*(File continues — deep-dive Part N follows below.)*

Hypercare without a named DRI is not real. Name the person and the end date when you hand off.

Keep a living decision log. When stakeholders revisit settled choices, point to the ADR date and evidence. Do not restart the debate from memory.

---

## Part N — Primary-study deep dive: Discovery that still works in real conditions

> **Scope note (read first):** This pack teaches **original** discovery, lifecycle, change management, and value framing skills only. It does not invent pricing, confidential win themes, or official lesson text. If an exam question looks like a “sales motion,” answer with durable architect behaviors: clarify commitments, align SLAs, document trade-offs, and phase delivery.

### N1. Discovery depth model (five passes)

1. **Problem pass** — jobs-to-be-done, quantified pain, current workaround.
2. **Constraint pass** — data, latency, residency, budget, unions/works councils, brand.
3. **Success pass** — leading and lagging metrics, decision rights, time box.
4. **Risk pass** — safety, compliance, operational blast radius, reputational.
5. **Change pass** — who loses power or status, training load, process redesign.

If you skip the change pass, technically complete pilots still fail.

### N2. Stakeholder interview bank (condensed, original)

**Executive:** What decision improves if this works? What would make you stop the program?
**Business owner:** Which step burns minutes? What error is unacceptable?
**Users:** When do you distrust AI? What must stay manual?
**Security:** What is the identity boundary? What is your nightmare ticket?
**Legal/Privacy:** Data classes? Transfer? Retention? Notices?
**Ops:** Who pages at 2am? What is the degrade mode?
**Finance:** Fully loaded cost ceiling per transaction?
**Sales/CS (GTM-adjacent):** What was already promised externally?

### N3. Requirement quality bar

A requirement is exam-ready when it has: owner, metric, constraint, priority, and test idea.
“Make it smart” is not a requirement. “p95 < 3s for FAQ path; HITL for refunds > $50. GDPR erasure within policy SLO” is.

### N4. Anti-patterns (discovery)

| Anti-pattern | Why it fails | Fix |
| --- | --- | --- |
| Single stakeholder proxy | Hidden vetoes later | Map power + impacted users |
| Solutioneering first | Misses real job | Problem framing before pattern |
| Ignoring shadow process | Users bypass system | Observe real work |
| No non-goals | Scope grows without control | Explicit out-of-scope list |
| Verbal-only decisions | People forget the decision | Decision log / ADR |

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
- “Citations reduce hallucination risk. They do not eliminate it.”

### O4. Oversell recovery

If Sales promised “fully autonomous legal advice”:
1. Document the external commitment as a fact.
2. Reframe to compliant scope (draft + attorney HITL).
3. Offer a phased roadmap with exit criteria.
4. Escalate the sponsor decision on residual risk. Do not ship unsafe scope in silence.

---

## Part P — Lifecycle & change management deep dive

### P1. Stage exit criteria (expand)

| Stage | Exit when… |
| --- | --- |
| Discovery | Brief + metrics + constraints signed |
| Design | ADR + threat notes + eval plan reviewed |
| Build | Contracts tested. Tracing on |
| Pilot | Success criteria measured. Risks logged |
| GA | Scorecard green. On-call ready. Docs handed off |
| Operate | SLO reviews running. Feedback loop owned |
| Iterate | Backlog prioritized from traces + users |
| Retire | Migration + model deprecation complete |

### P2. Change management levers

- **Comms:** why change, what stays human, how to get help
- **Training:** role-based (users ≠ reviewers ≠ engineers)
- **Incentives:** do not reward raw automation rate if quality collapses
- **Champions:** early adopters coach peers
- **Feedback:** visible “fix this answer” with responses

### P3. Conflict patterns & resolutions

| Conflict | Typical resolution |
| --- | --- |
| Security vs speed | Paved road + time-boxed exceptions |
| Product vs legal | Scoped MVP. Counsel on notices |
| Ops vs constant new features | Error budgets limit releases |
| Users vs automation | Copilot first. Expand autonomy after evidence |
| Sponsor vs reality | Data-backed reset. Options A/B/C |

### P4. Handoff package (must-not-forget)

Include architecture diagrams, ADRs, tool inventory, and the authZ model. Include eval dashboards, runbooks, the HITL staffing model, and the data retention map. Include the model pin/upgrade policy, known limitations, RACI, and the support escalation matrix.

### P5. Monitoring & iteration cadence

- Daily: SLO burns that page on-call
- Weekly: eval slice + top failure themes
- Monthly: cost unit economics + roadmap
- Quarterly: risk tier revisit + access review

---

## Part Q — Architecture communication drills

### Q1. Altitude switching exercise

Same decision (“workflow not agent”):
- **Exec:** “We chose a predictable staged process so auditors can see each check—faster to trust, slightly less flexible.”
- **Engineer:** “Deterministic DAG with schema validators between extract→policy→recommend. Agent only for open research subtask.”
- **Risk:** “Side effects gated. HITL above threshold. Full tool audit.”

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
If a question asks how to “position Claude vs competitor X using partner kit,” select answers about customer outcomes, constraints, and proof via evals. Do not invent competitive intel.

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
| Pilot with no exit | No GA criteria | Scorecard + date + owner |
| Shadow commitments | Security surprised | Early risk stakeholders |
| Metrics for show | Vanity dashboards | Tie to decisions/money/risk |
| Doc drift | Ops cannot page | Runbook tests in drills |
| Users reject the change | Users bypass AI | Copilot UX + champions |
| Model EOL with no plan | Break in prod | Pin + upgrade playbook |

---

## Part U — Extended Q&A (59–68)

**Q59.** First workshop output that feeds architecture?
**A.** A **one-pager brief** with problem, constraints, metrics, and risks—not a model choice debate.

**Q60.** Exec asks “when will it be fully autonomous?” Best answer shape?
**A.** Autonomy **gated by eval evidence and risk tier**, with phased milestones. Not a blind calendar promise.

**Q61.** Select TWO handoff must-haves: eval dashboards, on-call RACI, partner discount sheet, informal chat channel.
**A.** Eval dashboards and on-call RACI.

**Q62.** Sales promised unsafe scope. Architect should…
**A.** Document, reframe to compliant phased scope, escalate sponsor—**not** ship in silence.

**Q63.** Why include ops in discovery?
**A.** Degrade modes, paging, and capacity—and to prevent unsupportable GA.

**Q64.** ADR alternative rejected field matters because…
**A.** Exam and real reviews test whether you consider agents vs workflows and similar options.

**Q65.** Change management missing—likely outcome?
**A.** Users bypass the system. ROI never materializes.

**Q66.** No official GTM playbook available. What still works on CCAR-P?
**A.** Structured discovery, trade-off communication, lifecycle artifacts, and value metrics.

**Q67.** GA scorecard red on HITL staffing—ship anyway?
**A.** **No**—staff first, or reduce autonomy scope first.

**Q68.** Select THREE discovery anti-patterns: solutioneering first, single proxy stakeholder, no non-goals, writing ADRs, measuring baselines.
**A.** Solutioneering first, single proxy, no non-goals.

---

## Part V — Rapid review (Stakeholder 14%)

- Discovery before design. Change pass included.
- Requirements need metrics and owners.
- Altitude-switch communications. Keep ADRs.
- Value = hypothesis + evidence plan.
- Lifecycle exit criteria are better than vague feelings.
- Flag partner-only GTM limits. Stay original.
- Handoff = docs + owners + evals + runbooks.
- Reset oversells. Do not ship unsafe promises.

*Enablement mechanics live in `05`. Safety talk-tracks pair with `03`.*
