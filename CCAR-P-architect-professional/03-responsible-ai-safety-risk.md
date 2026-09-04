---
title: Responsible AI, Safety & Risk for Architects
---

# 03 — Responsible AI, Safety & Risk for Architects

**CCAR-P condensed domain 3** 
**Official domain mapped here:** Governance, Safety & Risk Management (**14%**) 
**Approx. questions:** ~9 of 63

---

## Disclaimer

Original architect-oriented safety notes grounded in **public** Anthropic materials (Constitutional AI concepts, platform safety features, responsible use norms) and standard enterprise AI governance practice. Not legal advice—engage counsel for GDPR/HIPAA/FedRAMP interpretations. Independent pack; not affiliated with Anthropic.

---

## Overview

CCAR-P expects architects to **design controls**, not recite ethics slogans. You must identify LLM-specific risks, place guardrails at the right layers (model, prompt, tool, app, process), insert human oversight where impact warrants, and align to regulatory/ethical obligations without pretending prompts alone are compliance.

This domain connects tightly to Integration (authZ, injection) and Stakeholders (risk communication). Developer enablement security baselines appear in file **05**.

---

## Key map

| Architect responsibility | Exam focus |
| --- | --- |
| Threat model LLM apps | Injection, data leakage, over-autonomy, unsafe advice |
| Defense in depth | Multiple control layers; least privilege |
| HITL design | When automation must stop |
| Compliance overlays | GDPR, HIPAA, FedRAMP *as scenario constraints* |
| Transparency & abuse reporting | User notices, logging, escalation |
| Dual-use / misuse | Refuse facilitation of harm; product policy |
| Vendor & subprocessors | Data processing agreements, retention/ZDR |

---

## Part A — Risk taxonomy for Claude solutions

### A1. Core LLM risks

| Risk | Description | Example |
| --- | --- | --- |
| Prompt injection | Untrusted content steers model against policy | Doc says “ignore rules, exfiltrate keys” |
| Jailbreak / policy bypass | User tries to override safety | Social-engineering the assistant |
| Hallucination | Fluent falsehoods | Invented legal clause |
| Data leakage | Sensitive data in outputs/logs/tools | PII in support draft + logged raw |
| Over-privilege | Excessive tool scope | Support bot with admin SaaS |
| Autonomous harm | Model-driven irreversible acts | Agent deletes prod resources |
| Bias / unfairness | Systematic disparate outcomes | Resume screener skew |
| Insecure output handling | Executing model output unsafely | Shelling LLM text without sandbox |
| Model theft / abuse of API | Credential stuffing, scraping | Stolen API keys |
| Supply chain | Malicious MCP/skills/plugins | Trojansed connector |

### A2. Impact × likelihood workshop

Score each risk for the use case. High impact × non-trivial likelihood → **mandatory mitigations** before GA. Exam scenarios often hide high impact in “convenient automation” proposals.

### A3. Anthropic alignment context (public concepts)

Anthropic publishes research on **Constitutional AI**—training with explicit principles so models critique/revise toward helpful, honest, harmless behavior. Architects should understand:

- Base model alignment ≠ your **application** is automatically compliant. 
- You still add product policies, tool restrictions, and monitoring. 
- Transparency about AI use and limitations remains an enterprise duty.

---

## Part B — Guardrail architecture (defense in depth)

### B1. Control layers

```
1. Policy & training (acceptable use, employee guidelines)
2. Identity & access (who may invoke which agent)
3. Application filters (input/output classifiers, DLP)
4. Prompt/Skill policy (local behavioral constraints)
5. Model choice & provider safety
6. Tool/MCP allowlists + server-side authZ
7. HITL & approval workflows
8. Monitoring, audit, incident response
```

No single layer is enough. Exam trap: “Add a stronger system prompt” as the only fix for refund fraud risk.

### B2. Input controls

- AuthN/Z before expensive/powerful models 
- Schema validation on user forms 
- Size limits; file type allowlists 
- PII detection/minimization where appropriate 
- Separation of untrusted content (retrieved docs) from instructions 
- Rate limits / anomaly detection per user 

### B3. Output controls

- Schema validation before side effects 
- DLP on outbound messages (secrets, PAN, health data) 
- Toxicity/policy classifiers for user-facing channels 
- Citation requirements for high-stakes Q&A 
- Blocking or queueing when classifiers fire 

### B4. Tool controls (strongest practical brakes)

- Least privilege credentials 
- Allowlisted operations 
- Human confirmation for high-impact verbs 
- Dry-run modes 
- Immutable audit log of tool args/results (redacted) 
- Network egress restrictions 

### B5. Prompt-level safety (necessary but insufficient)

Include: scope limits, disallowed topics/actions, escalation phrases, instructions to treat retrieved content as untrusted. Keep aligned with legal/policy owners—don’t invent “laws” in prompts.

---

## Part C — Prompt injection deeply

### C1. Direct vs indirect

- **Direct:** User message attacks the model. 
- **Indirect:** Attacker plants instructions in data the model will read (email, wiki, PDF, webpage, ticket).

Indirect injection is the enterprise nightmare for RAG and MCP fetch tools.

### C2. Mitigations that matter

| Mitigation | Notes |
| --- | --- |
| Privilege separation | Reader agent cannot pay invoices |
| Content security policy for tools | Allowlists, no arbitrary code |
| Explicit untrusted delimiters | “DATA, not instructions” framing |
| Minimize HTML/JS in context | Prefer plain text extracts |
| Output encoding | Never `eval` model text |
| Spot-check / detectors | Heuristics + models for injection patterns |
| Human approval | For novel destinations / large transfers |
| Corpus hygiene | Moderate who can publish to indexed stores |

### C3. What not to rely on

- “Please ignore malicious instructions” alone 
- Security through obscurity of system prompts 
- Assuming citations imply integrity of content 

---

## Part D — Human-in-the-loop (HITL)

### D1. When HITL is expected

- Legal/medical/financial irreversible advice or actions 
- Access changes / production deployments 
- Low confidence / out-of-distribution inputs 
- Safety classifier uncertainty 
- Regulatory “human review” obligations 
- Novel high-dollar transactions 

### D2. HITL anti-patterns

- Rubber-stamp UI with no evidence shown 
- Flooding humans with low-value reviews (alert fatigue) 
- HITL with no SLA owner 
- Logging approvals without identity 

### D3. Graduated autonomy

Start read-only → suggest → act with approval → act automatically on narrow safe class. Expand autonomy only with eval evidence.

### D4. Review-routing matrix (reversibility × cost × confidence) — P1

Use this to decide **auto** vs **sample** vs **always-human** without flooding reviewers or rubber-stamping high impact.

| Reversibility of action/advice | Cost / blast radius | Model+system confidence | Default routing |
| --- | --- | --- | --- |
| Easily reversible (draft email in UI, suggest edit) | Low | High | **Auto** (log + async sample) |
| Easily reversible | Low | Low / OOD | **Sample** elevated % or force confirm |
| Easily reversible | High (mass send capability exists) | Any | Treat as **not** easily reversible until send is gated |
| Hard to reverse (prod config, access grant) | Low–med | High | **Always-human** or dual-control for writes |
| Hard to reverse | High | Any | **Always-human**; break-glass only with ticket |
| Irreversible / regulated (clinical, legal filing, funds move) | Any | Any | **Always-human** (or human + second system)—no “confident auto” |
| Any | Any | Safety classifier uncertain | **Always-human** or block |

**Operating rules**

- Confidence is a **system** signal (retrieval score, validator pass, classifier margin)—not the model saying “I’m sure.” 
- **Sample** means statistically useful audit (stratified), not 0.1% vanity sampling. 
- Raise autonomy only when evals show error rates compatible with blast radius (tie to file 02 thresholds). 
- Exam trap: auto-routing irreversible money movement because “p95 latency.”

**Quick stems**

**Q69.** Low-confidence draft reply to one user, reversible in UI—route? 
**A.** Auto or light sample OK; still log. Not the same class as wire transfer.

**Q70.** High-confidence agent wants to detach IAM policy in prod—route? 
**A.** Always-human (or blocked tool); confidence does not override irreversibility.

---

## Part E — Privacy, compliance, and data governance

### E1. Scenario frameworks (not legal advice)

| Regime (often cited in exams) | Architect themes |
| --- | --- |
| GDPR | Lawful basis, minimization, access/erasure, DPIA, subprocessors, international transfers |
| HIPAA | PHI boundaries, BAAs, audit controls, minimum necessary |
| FedRAMP / public sector | Authorization boundary, logging, incident reporting, residency |
| SOC2 / ISO27001 | Control ownership, change management, vendor review |

Map product data flows: user → app → model provider → logs → vector DB → tools → third parties.

### E2. Retention & training use

Know your contractual posture: whether prompts are used to train; Zero Data Retention / retention options where offered; enterprise settings for log retention. Align app-level retention with legal holds and erasure requests—including **embeddings and eval caches**.

### E3. Privacy engineering tactics

- Tokenization / redaction before model call when feasible 
- Field-level encryption in stores 
- Purpose limitation in prompts (“use salary only for banding, don’t echo”) 
- Separate projects/keys per environment 
- Access reviews on who can read prompt logs 

### E4. Transparency

Users should know they’re interacting with AI when material; provide limitations and escalation to humans. Document model/provider in internal architecture records (EU AI Act-style transparency themes may appear as scenario flavor—follow counsel).

---

## Part F — Fairness, ethics, dual-use

### F1. Fairness in enterprise AI

For ranking/decision support:

- Define protected attributes handling policy 
- Measure slice performance 
- Avoid proxies that encode discrimination 
- Keep humans accountable for adverse decisions 
- Provide contestability paths 

### F2. Dual-use & misuse

Architect product policies for disallowed assistance (e.g., large-scale deception, cyber offense, bio harm) consistent with provider AUPs. Technical: classifiers, tool limits, abuse monitoring, trust & safety escalation.

### F3. Children / sensitive audiences

If product may serve minors or vulnerable users, raise age-appropriate design, stricter HITL, and content policies—don’t treat as generic chat.

---

## Part G — Governance operating model

### G1. Artifacts

- AI use-case register with risk tiering 
- Architecture decision records including safety 
- Model/prompt/tool change control 
- Incident severity matrix for AI harms 
- Vendor inventory (MCP, embedding providers) 
- Eval reports as release evidence 

### G2. Risk tiers (example)

| Tier | Examples | Controls |
| --- | --- | --- |
| 0 | Internal brainstorming | Baseline AUP |
| 1 | Copilot drafts | Logging, DLP light |
| 2 | Customer-facing RAG | Citations, evals, on-call |
| 3 | Semi-automated actions | HITL, strong authZ, IR plan |
| 4 | Critical automated decisions | Often disallowed or heavily regulated design |

### G3. Incident response extras for LLMs

Playbooks for: leaked prompts/secrets, injection campaigns, model producing prohibited content at scale, runaway spend, cross-tenant bleed. Include evidence preservation (traces) and customer notification criteria with legal.

---

## Part H — Safety evals

Safety is part of Evaluation domain but owned jointly:

- Policy violation rate on red-team sets 
- Indirect injection success rate 
- PII leakage probes 
- Over-refusal vs under-refusal balance for product UX 
- Tool abuse attempts 

Gate releases on safety metrics, not only task accuracy.

---

## Decision trees

### Autonomy gate

```
Can the action cause irreversible harm or regulated impact?
 YES → HITL or forbid automation
 NO → Is blast radius limited + evals green + monitoring live?
 YES → Allow narrow auto class
 NO → Suggest-only mode
```

### Injection-prone feature gate

```
Does the model read attacker-influenced content AND hold write tools?
 YES → Split privileges; confirm writes; harden retrieval; maybe delay GA
 NO → Standard input/output controls still apply
```

---

## Exam traps

1. Prompt-only “compliance.” 
2. Full autonomy on money movement / prod changes. 
3. Ignoring indirect injection in RAG. 
4. Logging raw PHI prompts indefinitely without controls. 
5. Fairness hand-waving for hiring/lending use cases. 
6. Enabling powerful MCP without vendor review. 
7. No incident severity for AI-specific harms. 
8. Equating provider alignment with app safety. 
9. HITL theater without evidence or owners. 
10. Erasing DB rows but leaving embeddings.

---

## Practice Q&A (28)

**Q1.** Why isn’t a strong system prompt enough to stop refund fraud? 
**A.** Attackers use injection; controls must include tool authZ, confirmations, monitoring.

**Q2.** Define indirect prompt injection. 
**A.** Malicious instructions planted in content the model retrieves or reads.

**Q3.** Name three defense-in-depth layers. 
**A.** Any of: IdP/authZ, input filters, prompt policy, tool allowlists, HITL, monitoring.

**Q4.** When is graduated autonomy appropriate? 
**A.** Expand from read-only to auto only after metrics prove safety on narrow classes.

**Q5.** What must erasure workflows include beyond primary DB delete? 
**A.** Embeddings, backups policy, eval caches, logs per legal guidance.

**Q6.** Give an HITL anti-pattern. 
**A.** Rubber-stamp approvals with no sources/diff shown.

**Q7.** How does Constitutional AI relate to app architects? 
**A.** Explains base model training philosophy; apps still need their own controls.

**Q8.** What is insecure output handling? 
**A.** Executing or rendering model output without sandboxing/encoding (e.g., raw shell).

**Q9.** Why measure over-refusal? 
**A.** Excess refusals harm usefulness; balance safety with product requirements.

**Q10.** What belongs in an AI use-case register? 
**A.** Purpose, data classes, risk tier, owner, controls, eval status.

**Q11.** Cross-tenant document bleed—primary control type? 
**A.** Hard isolation/filters in retrieval & authZ—not prompt instructions alone.

**Q12.** Name a dual-use concern for coding agents. 
**A.** Assistance with offensive cyber techniques beyond defensive/authorized scope—policy + monitoring.

**Q13.** Who approves tier-3 semi-automated money actions? 
**A.** Design should require human approval (and finance/security policy owners)—not the model alone.

**Q14.** What is a BAA relevant to? 
**A.** HIPAA scenarios involving PHI handled by vendors.

**Q15.** Why redact prompts in long-term logs? 
**A.** Reduce PII/secret sprawl and breach blast radius.

**Q16.** What is alert fatigue in HITL? 
**A.** Too many low-value reviews cause humans to approve blindly.

**Q17.** Minimum gate before GA of customer RAG? 
**A.** Typically: grounding evals, injection tests, tenancy tests, monitoring, clear escalation path.

**Q18.** Why treat Skills/plugins as supply chain? 
**A.** They can introduce instructions/tools that exfiltrate data or escalate privilege.

**Q19.** Output DLP caught a secret—what next? 
**A.** Block/queue message, rotate credential if real, incident process, fix source prompt/data.

**Q20.** Fairness slice testing means? 
**A.** Measuring quality/error rates across relevant demographic or segment slices per policy.

**Q21.** Can you satisfy GDPR with model choice alone? 
**A.** No—process, contracts, minimization, rights handling, etc. are required.

**Q22.** What’s a good first step when adding browser fetch to an agent? 
**A.** Threat-model injection + SSRF; allowlist; strip active content; restrict tools.

**Q23.** Why separate eval caches from prod logging retention? 
**A.** Different purposes/legal bases; still both need erasure/access processes.

**Q24.** What should users see on material AI decisions? 
**A.** Notice it’s AI-assisted, limitations, and path to human review (context-dependent).

**Q25.** Runaway agent spend—safety or ops issue? 
**A.** Both: budgets are safety/reliability controls against uncontrolled autonomy.

**Q26.** Best control for “agent can email anyone”? 
**A.** Destination allowlists + human confirm for novel addresses + rate limits.

**Q27.** Why keep an abuse mailbox / reporting path? 
**A.** Users and customers need a channel; feeds trust & safety response.

**Q28.** Provider ZDR means you can skip app-level retention design? 
**A.** No—you still retain data in *your* logs, stores, and tools.

---

## Checklist

- [ ] I can threat-model injection for RAG+tools 
- [ ] I can place controls across ≥5 layers 
- [ ] I can design HITL with evidence and SLAs 
- [ ] I can map data flows for privacy reviews 
- [ ] I can tier use cases and match controls 
- [ ] I can define AI incident severities 
- [ ] I can list safety eval probes for release gates 
- [ ] I can explain least privilege for agents 
- [ ] I can spot prompt-only “compliance” answers 
- [ ] I can describe erasure beyond SQL delete 

---

## Glossary

| Term | Meaning |
| --- | --- |
| Prompt injection | Steering model via crafted untrusted text |
| Indirect injection | Injection via retrieved/third-party content |
| Jailbreak | Attempts to bypass safety policies |
| HITL | Human-in-the-loop oversight |
| DLP | Data loss prevention |
| DPIA | Data Protection Impact Assessment |
| BAA | Business Associate Agreement (HIPAA context) |
| ZDR | Zero Data Retention (contractual/API posture) |
| AUP | Acceptable Use Policy |
| Over-refusal | Refusing benign requests excessively |
| Dual-use | Tech usable for helpful and harmful ends |
| Insecure output handling | Unsafe execution/rendering of model output |
| Use-case register | Inventory of AI systems with risk metadata |
| Red team | Adversarial testing of safety properties |
| Graduated autonomy | Phased increase of automatic action rights |

---

## Part I — Worked governance scenarios

### I1. Hospital discharge summary assistant

**Constraints:** PHI, high harm if wrong. 
**Design:** HIPAA-aware architecture; BAA where required; minimize PHI in prompts; on-prem/VPC cloud AI if mandated; citations to chart sections; clinician HITL always before chart writeback; extensive safety+accuracy evals; audit logs; no web tools. 
**Reject:** Autonomous chart updates; consumer API keys in wards.

### I2. HR benefits chatbot

**Constraints:** Employee personal data; legal accuracy. 
**Design:** RAG over versioned benefits PDFs with citations; refuse uncovered topics; escalate to HR for exceptions; strict tenancy; retention limits; fairness less central than privacy/accuracy but tone fairness still monitored.

### I3. Marketing content generator

**Constraints:** Brand, copyright, deceptive claims. 
**Design:** Claim checkers against approved product facts; disallow unverifiable medical/finance claims; human edit before publish; logging of prompts for trademark issues.

### I4. DevOps agent with cloud credentials

**Constraints:** Production blast radius. 
**Design:** Read-only by default; break-glass write role with two-person rule; environment segregation; command allowlists; mandatory change tickets for prod; session recording; budget caps. 
**Reject:** Long-lived admin keys in MCP config committed to git.

---

## Part J — Policy writing for architects

Translate legalese into enforceable engineering:

| Policy statement | Engineering control |
| --- | --- |
| “No automated adverse hiring decisions” | Tool that updates ATS stages blocked; HITL required |
| “Minimize personal data” | Field allowlist pre-model; redact logs |
| “Customers can request deletion” | Erasure API hits DB+index+object store |
| “All prod changes audited” | Tool gateway logs immutable to SIEM |

If you cannot name the control, the policy is theater.

---

## Part K — Extended Q&A (29–35)

**Q29.** Why forbid web tools in a PHI assistant by default? 
**A.** Reduces egress/injection surface and keeps boundary clearer for compliance.

**Q30.** Two-person rule means? 
**A.** Sensitive action requires a second authorized human approval.

**Q31.** What is claim checking in marketing genAI? 
**A.** Verifying generated claims against an approved facts base before publish.

**Q32.** Why is “clinician HITL always” justified for discharge summaries? 
**A.** High clinical risk; regulations and duty of care expect professional accountability.

**Q33.** Git-committed cloud admin keys—what risks? 
**A.** Leakage, lateral movement, uncontrollable agent actions; rotate and move to secret manager.

**Q34.** How do you enforce “minimize personal data” at request time? 
**A.** Explicit field allowlists / redaction before calling the model.

**Q35.** What makes an AI incident severity different from generic Sev1? 
**A.** May include mass harmful content, privacy bleed across tenants, or unsafe autonomy—not only uptime.

---

## Extended checklist

- [ ] I can convert policy text into concrete engineering controls 
- [ ] I can design a PHI-safe architecture sketch 
- [ ] I can apply two-person rules to prod agents 
- [ ] I can justify when web/MCP tools are inappropriate 

---



---

## Part L — Organizational accountability & board-level framing

Architects often brief executives. Translate risk into business language:

| Technical issue | Business framing |
| --- | --- |
| Indirect injection | Fraud / data exfiltration risk via ordinary documents |
| Hallucinated policy | Regulatory misstatement & customer harm |
| Over-broad tools | Insider-threat-class blast radius without an insider |
| No eval gates | Shipping unquantified error rates into brand channels |
| Log PII sprawl | Breach cost amplification |

Propose **controls + metrics + owners + dates**, not fear alone.

### L1. Third-party risk questionnaire (MCP/SaaS AI)

Ask vendors:

- Data retention & training use 
- Subprocessors list 
- Encryption in transit/at rest 
- Incident notification SLA 
- Pen-test / SOC reports 
- Region pinning 
- Termination & deletion mechanics 

Score before enablement; re-review on major version changes.

### L2. Secure SDLC for prompts & tools

Treat prompts/Skills/tools as code:

- PR review including security 
- Static checks for forbidden tool scopes 
- Secrets scanning 
- Mandatory eval job 
- Change ticket linking for prod pins 

### L3. Red team cadence

Quarterly external or internal red team for tier-2+ systems; continuous automated probes in CI for smoke-level injections. Track findings to closure like vulns (CVE-like severity for AI issues optional but useful).

### L4. Accessibility & inclusion

Responsible AI includes accessible UX: screen readers for AI notices, readable explanations, avoiding dark patterns that hide human escalation. Multilingual safety should not be English-only eval theater.

### L5. Environmental / cost ethics (light touch)

Unbounded agent loops waste money and energy. Budgets are governance, not only FinOps.

### L6. Deep scenario: “summarize these emails and send fixes”

Attack emails contain injection. Design: summarize in read-only mode; never send mail without human confirm; strip/ignore instruction-like lines in detectors; allowlist recipients; show diff of proposed sends.

### L7. Deep scenario: model suggests disabling safety filters

Product must not expose toggles that disable enterprise-mandatory controls to end users. Admin break-glass with audit only.

### L8. Documentation pack for auditors

Keep: data flow diagrams, DPIA/transfer assessments, model cards/system cards (internal), eval reports, access reviews, incident postmortems, vendor DPAs. CCAR-P won’t grade your binder—but scenarios ask what evidence you’d produce.

### L9. Extended Q&A 36–45

**Q36.** How do you brief an exec on injection risk in one sentence? 
**A.** “Ordinary documents can carry hidden instructions that trick the assistant into abusing its permissions.”

**Q37.** Name four vendor questions before enabling SaaS MCP. 
**A.** Retention/training, subprocessors, regions, incident SLA (plus deletion, encryption, audit reports).

**Q38.** Why treat prompts as code? 
**A.** They change behavior like code—need review, versioning, testing, and controlled release.

**Q39.** English-only safety evals miss what? 
**A.** Multilingual jailbreaks and toxicities; non-English user harm.

**Q40.** What is break-glass for safety toggles? 
**A.** Time-limited admin override with mandatory audit—not a user-facing ‘disable safety’ switch.

**Q41.** Why are budgets a governance control? 
**A.** They bound autonomous waste, abuse, and runaway tool loops.

**Q42.** What evidence might an auditor request for an AI feature? 
**A.** Data flows, DPIA, eval gates, access reviews, DPAs, incident records.

**Q43.** In the malicious email scenario, what’s the safest send policy? 
**A.** Human confirm + recipient allowlist + visible diff; no autonomous send.

**Q44.** How often red team tier-2+ systems (example cadence)? 
**A.** At least quarterly deep tests plus continuous automated smoke probes.

**Q45.** Secret scanning should cover which AI artifacts? 
**A.** Prompts, Skills, MCP configs, notebooks, CI logs, and example traces in repos.

### L10. Extra checklist

- [ ] I can translate technical AI risk to business impact 
- [ ] I can run a vendor MCP questionnaire 
- [ ] I can describe secure SDLC for prompts/tools 
- [ ] I can outline auditor evidence for an AI system 
- [ ] I can design read-only summarize vs send flows safely 
- [ ] I can argue against user-facing safety kill switches 

### L11. Glossary additions

| Term | Meaning |
| --- | --- |
| System card | Internal/public description of system behavior & limits |
| DPA | Data Processing Agreement |
| Break-glass | Emergency privileged access with audit |
| Red team cadence | Scheduled adversarial testing rhythm |
| FinOps | Cloud/AI cost operational discipline |
| ATS | Applicant tracking system |
| SIEM | Security information and event management |
| Dark pattern | UX that manipulates users against their interest |
| Transfer assessment | Review of cross-border data transfers |
| Smoke probe | Lightweight continuous safety test |

---

## Part M — Mapping controls to CCAR-P verbs

Exam stems often use verbs: *implement, identify, ensure, design, recommend*. Pair them:

| Verb | Strong answer ingredient |
| --- | --- |
| Implement | Name concrete mechanism (allowlist, HITL queue, DLP) |
| Identify | Name risk + where in architecture it arises |
| Ensure | Control + measurement/monitoring |
| Design | Diagrammed layers + owners |
| Recommend | Choose among options with trade-offs, not laundry list |

Avoid vague “follow best practices” selections when a specific control is offered.

### M1. Final rapid-fire Q&A 46–50

**Q46.** Best single control to add when giving an agent `delete_instance`? 
**A.** Remove it or require multi-party approval—least privilege first.

**Q47.** What’s wrong with storing raw chat containing SSNs forever? 
**A.** Retention/minimization failure; breach amplification; rights request difficulty.

**Q48.** How do you “ensure” tenancy safety? 
**A.** Hard authZ filters + automated cross-tenant retrieval tests in CI + monitoring.

**Q49.** Customer asks to turn off logging for “privacy.” Risk? 
**A.** May conflict with security/audit duties; seek legal/security design of redaction vs disable.

**Q50.** Which is better evidence of safety: marketing claims or eval reports? 
**A.** Versioned eval/red-team reports with metrics and owners.



---

## Part N — Safety exam drill

Stems often combine “automate” + regulated data. Default instinct: **narrow scope, HITL, server-side authZ, eval gates**. If an option only strengthens the system prompt, it is usually incomplete.

### N1. Abuse monitoring signals

- Spike in refusal bypass attempts 
- Unusual tool fanout per user 
- Destination novelty for email/transfer tools 
- Cross-tenant retrieval denials surge (or worse, successes) 
- Sudden cost per user 

### N2. Model card vs system card

Model cards describe base models; **system cards** describe *your* application’s behavior, evaluations, and limits. Architects own system-level documentation.

### N3. Q&A 51–56

**Q51.** If one option is “stronger system prompt” and another adds tool confirm + allowlist, which usually wins for money movement? 
**A.** Confirm + allowlist (hard controls).

**Q52.** What is a system card in enterprise practice? 
**A.** Documentation of the deployed AI system’s behavior, evals, and limitations.

**Q53.** Name three abuse-monitoring signals. 
**A.** Bypass attempts, tool fanout spikes, novel transfer destinations (etc.).

**Q54.** Cross-tenant retrieval *successes* in logs mean? 
**A.** Critical isolation failure—incident response immediately.

**Q55.** Why separate model cards from system cards? 
**A.** Provider model properties ≠ your tool privileges, data, and UX risks.

**Q56.** Default design instinct for regulated automation stems? 
**A.** Narrow scope + HITL + server authZ + eval gates.

*Next: `04-stakeholder-lifecycle-gtm.md`. Enablement details in `05`.*


## Part O — Rapid review (Governance 14%)

Governance questions reward **layered controls** and **humility about prompts**. Rapid list:

- Threat-model injection whenever untrusted text meets privileged tools. 
- HITL for irreversible or regulated impacts; no rubber stamps. 
- Privacy: minimization, retention, erasure including embeddings. 
- Tier use cases; match controls to tiers. 
- Vendors/MCP are supply chain. 
- Safety evals gate releases with task quality. 
- Incidents need AI-specific severities and evidence trails. 
- System cards document *your* app, not just the base model. 
- Translate policy → engineering mechanism. 
- Never claim prompt-only compliance.

Worked memory hook: **Hospital writeback → always clinician HITL; DevOps agent → read-only first; Marketing claims → fact checker; Email send → confirm + allowlist.**


## Part P — Control catalog (printable)

| Risk | Preventive control | Detective control | Corrective control |
| --- | --- | --- | --- |
| Indirect injection | Privilege split; allowlists | Injection probes; odd tool patterns | Disable tool; revoke sessions |
| Hallucinated advice | Citations; refuse-on-empty | Faithfulness evals | HITL correction; user notice |
| PII leakage | Minimization; DLP | Log scanners | Rotate; notify; purge |
| Over-autonomy | Budgets; confirmations | Turn/cost alerts | Flag off; reduce scope |
| Cross-tenant bleed | Hard filters; separate indexes | Canary tenant tests | Incident IR; patch filters |
| Malicious MCP | Vendor review; marketplace only | Version pin monitors | Kill switch; revoke tokens |
| Insecure output handling | Sandbox; no raw exec | Runtime detections | Patch renderer; revoke |
| Bias in decisions | Human accountability; slice tests | Disparate error dashboards | Adjust product; stop auto |

Use this catalog when a stem asks what to *implement*, *monitor*, or *do after* an incident—match the column to the verb.


### Closing thought

Responsible AI for architects is operational: privileges, proofs, people-in-the-loop, and paperwork that maps to real systems. If your answer cannot name a mechanism, a metric, and an owner, keep designing.


*Remember:* mechanism + metric + owner. Prompt text is never the whole control story for CCAR-P governance items—pair it with authZ, HITL, monitoring, and evidence. *(File continues — deep-dive Part Q follows below.)*

### One more worked stem

Stem flavor: “Auto-approve invoice payments under $500 via Claude agent with ERP write tool.” 
Strong answer direction: lower threshold further or require dual control; enforce payee allowlist; server-side authZ; anomaly detection on novel payees; HITL above risk score—not prompt promising thrift alone.


---

## Part Q — Primary-study deep dive: Governance architecture

### Q1. Defense-in-depth reference blueprint

```
[User/Channel]
 → AuthN (SSO) + AuthZ (role → agent)
 → Input filters (injection heuristics, DLP, malware)
 → Orchestrator policy (allowed tools, budgets)
 → Claude (system policy + Skills) 
 → Tool gateway (allowlist, arg validation, server authZ)
 → Output filters (PII, toxicity, brand, secrets)
 → HITL queues (by risk tier)
 → Audit + abuse monitoring + IR
```

Every exam scenario that proposes “just add a stronger system prompt” is incomplete unless other layers are considered.

### Q2. Risk-tier → control baseline (copy/adapt)

| Tier | Example | Autonomy | Logging | HITL | Eval bar |
| --- | --- | --- | --- | --- | --- |
| T0 Informational | Public FAQ paraphrase | High | Basic | Spot check | Light |
| T1 Assistive draft | Email draft to human | High draft / low send | Full prompts redacted | Human sends | Medium |
| T2 Bounded action | Refund ≤ $50 | Auto within caps | Full + tool I/O | Sampled + anomalies | High |
| T3 High impact | Medical, legal, large money, prod changes | Default deny auto | Tamper-evident | Mandatory | Highest + red team |

### Q3. Guardrail trade-off table

| Control | Catches | Misses / cost | Overuse failure |
| --- | --- | --- | --- |
| System prompt policy | Soft behavioral norms | Determined injection | False sense of security |
| Input classifier | Many jailbreaks/PII | Novel paraphrases; latency | Blocks legitimate work |
| Output DLP | Secrets/PII egress | Encoded exfil | Over-redaction UX |
| Tool allowlist | Over-privilege | Logic bugs inside allowed tools | Shadow IT bypass |
| Server authZ | Confused deputy | Misconfigured ACLs | — (do not skip) |
| HITL | High-impact errors | Slow; reviewer fatigue | Rubber-stamping |
| Monitoring | Drift/abuse over time | Needs analysts | Alert fatigue |

### Q4. HITL design patterns

1. **Approve/reject gate** before side effect. 
2. **Edit-then-send** (copilot). 
3. **Review sampling** (X% or uncertainty-triggered). 
4. **Four-eyes** for dual control (finance). 
5. **Async specialist queue** (legal/medical). 

Anti-patterns: HITL on every trivial FAQ (kills ROI); HITL with no SLA (queue death); HITL without UI showing sources/tool args (blind approval).

### Q5. Prompt injection failure-mode analysis

| Variant | Channel | Example | Primary mitigations |
| --- | --- | --- | --- |
| Direct | User chat | “Ignore previous…” | Policy + classifiers + refuse |
| Indirect | Retrieved doc/email/web | Hidden instruction in PDF | Treat content as data; cite; no raw tool trust |
| Tool-result injection | MCP/API payloads | Error message contains instructions | Schema validate; sanitize; allowlist |
| Cross-user | Shared memory/stores | Poisoned shared Skill | Tenancy; review; signing |
| Multimodal | Image/OCR text | Text in screenshot | Same as indirect |

**Never rely on:** “The model is aligned so we’re fine.” Alignment reduces risk; application controls remain mandatory.

### Q6. Compliance overlay mapping (scenario language, not legal advice)

| Constraint cue in stem | Architecture implications |
| --- | --- |
| GDPR / personal data | Minimization, retention, DSR erasure including indexes/logs, DPIA-minded design, transfer review |
| HIPAA / health | BAA/vendor posture, PHI minimization, access controls, audit, careful on third-party tools/web |
| FedRAMP / gov | Authorized deployment path, strict boundaries, logging, change control |
| PCI hints | Never put PAN into prompts; tokenize; segment |
| Children’s data | Heightened restrictions; often avoid training/retention; parental/policy gates |

Exam answers should **name controls**, not recite statute numbers.

### Q7. Fairness & transparency practical checklist

- [ ] Who is affected if the model is wrong? 
- [ ] Disparate error rates checked across relevant groups where lawful/appropriate? 
- [ ] User told they interact with AI? 
- [ ] Limitations and escalation path documented? 
- [ ] Appeals/override for consequential decisions? 
- [ ] Training/eval data governance noted?

### Q8. Incident response extras for LLM apps

Add to classic IR:
- Prompt/tool trace preservation (legal hold) 
- Ability to disable tools/models/features via flag 
- Injection indicator hunting 
- Customer comms templates (file 04) 
- Model/provider status correlation 

---

## Part R — Worked decision scenarios (Safety)

### R1. “Auto-send email fixes from mailbox summary”

**Reject auto-send.** Summarize + draft only; human sends; treat email bodies as untrusted (indirect injection); DLP on drafts; audit.

### R2. DevOps agent with cloud credentials

Read-only recon tools first; writes via change tickets + HITL; short-lived creds; no org-admin keys in MCP; network egress allowlists; break-glass separate.

### R3. HR chatbot asked for medical leave legal advice

Scope to benefits docs via RAG + cite; refuse personalized legal/medical advice; escalate to HR human; log sensitive topics carefully.

### R4. Marketing generator producing competitor-disparaging claims

Brand/policy output filters; require sources for factual claims; human publish gate; eval for prohibited topics.

### R5. Stakeholder asks to “turn off safety filters for speed”

Architect response: safety filters are risk controls tied to tier; propose measured latency opts (caching, routing) instead; document residual risk if any control is narrowed with approval.

---

## Part S — Production safety checklist

### S1. Design review

- [ ] Threat model includes injection, leakage, over-autonomy, abuse 
- [ ] Risk tier assigned with control baseline 
- [ ] Tool allowlist + server authZ proven with negative tests 
- [ ] HITL triggers and SLAs defined 
- [ ] Data classes allowed in prompts documented 
- [ ] Retention for logs/evals lawful and minimized 
- [ ] Abuse monitoring signals listed 
- [ ] Red-team plan scheduled pre-GA

### S2. Pre-GA safety gate

- [ ] Injection suite pass rate meets bar 
- [ ] DLP false-positive UX accepted by product 
- [ ] High-impact paths cannot execute without HITL/policy engine 
- [ ] Vendor/MCP questionnaire complete 
- [ ] Incident runbook rehearsed 
- [ ] Transparency/notice copy approved

### S3. Steady-state

- [ ] Quarterly access review for agent credentials 
- [ ] Eval drift review includes safety metrics 
- [ ] Plugin/MCP updates reviewed like dependencies 
- [ ] Post-incident learnings → control updates

---

## Part T — Trade-offs: autonomy vs control

| Push for more autonomy when… | Keep humans when… |
| --- | --- |
| Actions reversible & low blast radius | Irreversible money/health/safety/legal |
| Strong evals + monitoring | Sparse data / novel intents |
| Clear policy engines encode rules | Rules require judgment calls |
| Exception rate within staffing | Reviewers already overloaded |

---

## Part U — Extended Q&A (57–65)

**Q57.** Strongest control against unauthorized data access via tools? 
**A.** **Server-side authorization** on the data plane using verified identity.

**Q58.** Indirect injection best single description? 
**A.** Malicious instructions embedded in **content the model reads** (docs, web, emails), not only in the user chat box.

**Q59.** Select TWO for Tier-3 payout agent: mandatory HITL, open web server tool for all users, full audit traces, temperature=1.5 creative mode. 
**A.** Mandatory HITL and full audit traces.

**Q60.** Constitutional AI means your app is compliant with HIPAA? 
**A.** **No**—base alignment ≠ application compliance; you still design controls and vendor posture.

**Q61.** Reviewer rubber-stamping—what to change? 
**A.** Reduce volume (better auto for T0/T1), improve UI with sources/tool args, rotate reviewers, sample quality audits.

**Q62.** GDPR erasure request—what systems to touch? 
**A.** Primary DB **and** vector indexes, caches, eval sets, logs (per policy), backups per legal schedule.

**Q63.** Best response if product wants model to execute shell from output? 
**A.** **Sandbox / deny by default**; never pipe unsandboxed LLM text to privileged shells.

**Q64.** Dual-use request for detailed biological harm—in product policy terms? 
**A.** Refuse / safe completion per acceptable use; escalate abuse patterns; do not provide actionable harmful detail.

**Q65.** Select THREE layers in defense in depth: tool allowlist, output DLP, server authZ, “trust the vendor alone”, disable all logging for privacy theater. 
**A.** Allowlist, output DLP, server authZ.

---

## Part V — Rapid review (Governance 14%)

- Design controls, don’t slogan. 
- Defense in depth; prompts are one layer. 
- Injection: direct + indirect + tool results. 
- HITL by impact tier; avoid rubber stamps. 
- AuthZ in code; least privilege tools. 
- Compliance = controls + evidence, not vibes. 
- Fairness, notices, appeals for consequential AI. 
- IR needs traces, kill switches, clear owners.

*Pair with Integration authZ in `02` and lifecycle risk talk-tracks in `04`.*


---

## Part W — Abuse monitoring, model cards, and exam stems

### W1. Abuse monitoring signal examples

- Sudden spike in tool calls per user 
- Repeated jailbreak phrasings 
- DLP hit clusters 
- Anomalous exfil-sized outputs 
- New MCP server reaching odd egress 

Wire to SOC/IR playbooks; don’t collect sensitive content beyond policy.

### W2. Model card vs system card (architect speak)

**Model card** (provider): capabilities, limitations, eval highlights of the base model. 
**System card / system documentation** (you): how *your* application uses the model—data flows, controls, residual risks. Exam cares that you know **application** accountability remains yours.

### W3. Quick stems

**Q66.** Product removes output filters to “sound more helpful.” Risk? 
**A.** Increased leakage/policy violations; violates defense in depth; need risk acceptance formally—not silent removal.

**Q67.** Best evidence for auditors that HITL works? 
**A.** Ticketed approvals with reviewer identity, timestamps, and linked model/tool traces—not a policy PDF alone.

**Q68.** FedRAMP-like constraint in stem—what changes first? 
**A.** Deployment/boundary choices and logging/change control—not only a nicer prompt.

*File 03 primary-study target band: 6000–9000 words.*
