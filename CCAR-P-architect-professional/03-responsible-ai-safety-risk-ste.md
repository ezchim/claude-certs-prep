---
title: 03 — Responsible AI, Safety & Risk for Architects — Simplified Technical English
disclaimer: Original study notes — independent and not official course content
approx_length: STE edition (ASD-STE100) — primary study
updated: 2026-08-30
---

# 03 — Responsible AI, Safety & Risk for Architects

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, MCP, prompting, caching, effort, p95) are exceptions and stay as written. Model IDs and prices change. Learn the decision rules. Check the current model cards before the exam.

**CCAR-P condensed domain 3** 
**Official domain mapped here:** Governance, Safety & Risk Management (**14%**) 
**Approx. questions:** ~9 of 63

---

## Disclaimer

These notes are original architect safety notes. They use **public** Anthropic materials. These include Constitutional AI concepts, platform safety features, and responsible use norms. They also use standard enterprise AI governance practice.

This is not legal advice. Use counsel for GDPR, HIPAA, and FedRAMP interpretations. This pack is independent. It is not affiliated with Anthropic.

---

## Overview

CCAR-P expects architects to **design controls**. It does not expect you to recite ethics slogans.

You must identify LLM-specific risks. You must put guardrails at the correct layers (model, prompt, tool, app, process). You must insert human oversight where impact needs it. You must align to regulatory and ethical obligations. Do not treat prompts alone as compliance.

This domain connects tightly to Integration (authZ, injection) and Stakeholders (risk communication). Developer enablement security baselines appear in file **05**.

---

## Key map

| Architect responsibility | Exam focus |
| --- | --- |
| Threat model LLM apps | Injection, data leakage, over-autonomy, unsafe advice |
| Defense in depth | Multiple control layers. Least privilege |
| HITL design | When automation must stop |
| Compliance overlays | GDPR, HIPAA, FedRAMP *as scenario constraints* |
| Transparency & abuse reporting | User notices, logging, escalation |
| Dual-use / misuse | Refuse help that causes harm. Product policy |
| Vendor & subprocessors | Data processing agreements, retention/ZDR |

---

## Part A — Risk taxonomy for Claude solutions

### A1. Core LLM risks

| Risk | Description | Example |
| --- | --- | --- |
| Prompt injection | Untrusted content steers the model against policy | A doc says “ignore rules, exfiltrate keys” |
| Jailbreak / policy bypass | A user tries to override safety | Social-engineering the assistant |
| Hallucination | Fluent false statements | An invented legal clause |
| Data leakage | Sensitive data in outputs, logs, or tools | PII in a support draft plus a raw log |
| Over-privilege | Tool scope that is too large | A support bot with admin SaaS access |
| Autonomous harm | Model-driven acts that you cannot reverse | An agent deletes prod resources |
| Bias / unfairness | Systematic outcomes that are not equal | Resume screener skew |
| Insecure output handling | You execute model output in an unsafe way | You run LLM text in a shell without a sandbox |
| Model theft / abuse of API | Credential stuffing, scraping | Stolen API keys |
| Supply chain | Malicious MCP, skills, or plugins | A trojan connector |

### A2. Impact × likelihood workshop

Score each risk for the use case. High impact × real likelihood → mandatory mitigations before GA.

Exam scenarios often put high impact inside proposals that look like convenient automation.

### A3. Anthropic alignment context (public concepts)

Anthropic publishes research on **Constitutional AI**. Training uses explicit principles. Models critique and revise toward helpful, honest, harmless behavior. Architects need to know this:

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

No single layer is enough. Common exam error: “Add a stronger system prompt” as the only fix for refund fraud risk.

### B2. Input controls

- Use AuthN/Z before you call expensive or powerful models 
- Validate schema on user forms 
- Set size limits and file type allowlists 
- Detect and minimize PII where this is appropriate 
- Separate untrusted content (retrieved docs) from instructions 
- Use rate limits and anomaly detection per user 

### B3. Output controls

- Validate schema before side effects 
- Run DLP on outbound messages (secrets, PAN, health data) 
- Use toxicity and policy classifiers for user-facing channels 
- Require citations for high-stakes Q&A 
- Block or queue when classifiers fire 

### B4. Tool controls (strongest practical brakes)

- Use least privilege credentials 
- Allowlist operations 
- Require human confirmation for high-impact verbs 
- Provide dry-run modes 
- Keep an immutable audit log of tool args and results (redacted) 
- Restrict network egress 

### B5. Prompt-level safety (necessary but insufficient)

Include: scope limits, disallowed topics and actions, escalation phrases, and instructions to treat retrieved content as untrusted.

Keep this aligned with legal and policy owners. Do not invent “laws” in prompts.

---

## Part C — Prompt injection deeply

### C1. Direct vs indirect

- **Direct:** The user message attacks the model. 
- **Indirect:** An attacker plants instructions in data that the model will read (email, wiki, PDF, webpage, ticket).

Indirect injection is a severe enterprise risk for RAG and MCP fetch tools.

### C2. Mitigations that matter

| Mitigation | Notes |
| --- | --- |
| Privilege separation | A reader agent cannot pay invoices |
| Content security policy for tools | Allowlists. No arbitrary code |
| Explicit untrusted delimiters | “DATA, not instructions” framing |
| Minimize HTML/JS in context | Prefer plain text extracts |
| Output encoding | Never `eval` model text |
| Spot-check / detectors | Heuristics plus models for injection patterns |
| Human approval | For novel destinations / large transfers |
| Corpus hygiene | Control who can publish to indexed stores |

### C3. What not to rely on

- “Please ignore malicious instructions” alone 
- Security through obscurity of system prompts 
- An assumption that citations prove integrity of content 

---

## Part D — Human-in-the-loop (HITL)

### D1. When HITL is expected

- Legal, medical, or financial advice or actions that you cannot reverse 
- Access changes / production deployments 
- Low confidence / out-of-distribution inputs 
- Safety classifier uncertainty 
- Regulatory “human review” obligations 
- Novel high-dollar transactions 

### D2. HITL anti-patterns

- A UI that lets humans approve without a real review and shows no evidence 
- Too many low-value reviews for humans (alert fatigue) 
- HITL with no SLA owner 
- Logged approvals without identity 

### D3. Graduated autonomy

Start with read-only. Then suggest. Then act with approval. Then act automatically on a narrow safe class. Raise autonomy only with eval evidence.

### D4. Review-routing matrix (reversibility × cost × confidence) — P1

Use this to decide **auto** vs **sample** vs **always-human**. This prevents too many reviews for humans. This also prevents approval without review for high impact.

| Reversibility of action/advice | Cost / blast radius | Model+system confidence | Default routing |
| --- | --- | --- | --- |
| Easily reversible (draft email in UI, suggest edit) | Low | High | **Auto** (log + async sample) |
| Easily reversible | Low | Low / OOD | **Sample** at a higher % or force confirm |
| Easily reversible | High (mass send capability exists) | Any | Treat as **not** easily reversible until you gate send |
| Hard to reverse (prod config, access grant) | Low–med | High | **Always-human** or dual-control for writes |
| Hard to reverse | High | Any | **Always-human**. Break-glass only with a ticket |
| Irreversible / regulated (clinical, legal filing, funds move) | Any | Any | **Always-human** (or human + second system)—no “confident auto” |
| Any | Any | Safety classifier uncertain | **Always-human** or block |

**Operating rules**

- Confidence is a **system** signal (retrieval score, validator pass, classifier margin). It is not the model that says “I am sure.” 
- **Sample** means a statistically useful audit (stratified). It is not 0.1% sampling that is too small to be useful. 
- Raise autonomy only when evals show error rates that match the blast radius (tie to file 02 thresholds). 
- Common exam error: auto-routing of irreversible money movement because of “p95 latency.”

**Quick stems**

**Q69.** Low-confidence draft reply to one user, reversible in UI—route? 
**A.** Auto or light sample is OK. Still log. This is not the same class as a wire transfer.

**Q70.** High-confidence agent wants to detach IAM policy in prod—route? 
**A.** Always-human (or blocked tool). Confidence does not override irreversibility.

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

Know your contractual posture. Know if prompts train models. Know Zero Data Retention / retention options where the vendor offers them. Know enterprise settings for log retention.

Align app-level retention with legal holds and erasure requests. Include **embeddings and eval caches**.

### E3. Privacy engineering tactics

- Tokenize or redact before the model call when this is feasible 
- Use field-level encryption in stores 
- Limit purpose in prompts (“use salary only for banding, do not echo”) 
- Separate projects and keys per environment 
- Review access for who can read prompt logs 

### E4. Transparency

Users need to know they interact with AI when this is material. Give limitations and a path to humans.

Document the model and provider in internal architecture records. EU AI Act-style transparency themes can appear as scenario details. Follow counsel.

---

## Part F — Fairness, ethics, dual-use

### F1. Fairness in enterprise AI

For ranking and decision support:

- Define a policy for protected attributes 
- Measure slice performance 
- Avoid proxies that encode discrimination 
- Keep humans accountable for adverse decisions 
- Provide contestability paths 

### F2. Dual-use & misuse

Architect product policies for disallowed assistance. Examples: large-scale deception, cyber offense, bio harm. Keep these consistent with provider AUPs.

Technical controls: classifiers, tool limits, abuse monitoring, and trust and safety escalation.

### F3. Children / sensitive audiences

If the product can serve minors or vulnerable users, raise age-appropriate design, stricter HITL, and content policies. Do not treat this as generic chat.

---

## Part G — Governance operating model

### G1. Artifacts

- AI use-case register with risk tiering 
- Architecture decision records that include safety 
- Model, prompt, and tool change control 
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
| 4 | Critical automated decisions | Often disallowed or a heavily regulated design |

### G3. Incident response extras for LLMs

Playbooks for: leaked prompts and secrets, injection campaigns, a model that produces prohibited content at scale, runaway spend, and cross-tenant bleed.

Include evidence preservation (traces). Include customer notification criteria with legal.

---

## Part H — Safety evals

Safety is part of the Evaluation domain. Safety is also a joint ownership:

- Policy violation rate on red-team sets 
- Indirect injection success rate 
- PII leakage probes 
- Over-refusal vs under-refusal balance for product UX 
- Tool abuse attempts 

Gate releases on safety metrics. Do not use task accuracy alone.

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
3. You ignore indirect injection in RAG. 
4. You log raw PHI prompts with no time limit and no controls. 
5. Fairness statements with no method for hiring/lending use cases. 
6. You enable powerful MCP without vendor review. 
7. No incident severity for AI-specific harms. 
8. You treat provider alignment as app safety. 
9. HITL that only looks like review, with no evidence or owners. 
10. You erase DB rows but you leave embeddings.

---

## Practice Q&A (28)

**Q1.** Why is a strong system prompt not enough to stop refund fraud? 
**A.** Attackers use injection. Controls must include tool authZ, confirmations, and monitoring.

**Q2.** Define indirect prompt injection. 
**A.** Malicious instructions that an attacker plants in content the model retrieves or reads.

**Q3.** Name three defense-in-depth layers. 
**A.** Any of: IdP/authZ, input filters, prompt policy, tool allowlists, HITL, monitoring.

**Q4.** When is graduated autonomy appropriate? 
**A.** Move from read-only to auto only after metrics prove safety on narrow classes.

**Q5.** What must erasure workflows include beyond primary DB delete? 
**A.** Embeddings, backups policy, eval caches, and logs per legal guidance.

**Q6.** Give an HITL anti-pattern. 
**A.** Approvals with no sources or diff shown. Humans approve without a real review.

**Q7.** How does Constitutional AI relate to app architects? 
**A.** It explains base model training philosophy. Apps still need their own controls.

**Q8.** What is insecure output handling? 
**A.** You execute or render model output without a sandbox or encoding (for example, a raw shell).

**Q9.** Why measure over-refusal? 
**A.** Excess refusals harm usefulness. Balance safety with product requirements.

**Q10.** What belongs in an AI use-case register? 
**A.** Purpose, data classes, risk tier, owner, controls, eval status.

**Q11.** Cross-tenant document bleed—primary control type? 
**A.** Hard isolation and filters in retrieval and authZ. Do not use prompt instructions alone.

**Q12.** Name a dual-use concern for coding agents. 
**A.** Assistance with offensive cyber techniques beyond defensive or authorized scope. Use policy plus monitoring.

**Q13.** Who approves tier-3 semi-automated money actions? 
**A.** Design must require human approval (and finance/security policy owners). The model alone is not enough.

**Q14.** What is a BAA relevant to? 
**A.** HIPAA scenarios where vendors handle PHI.

**Q15.** Why redact prompts in long-term logs? 
**A.** You reduce PII/secret sprawl and the blast radius of a breach.

**Q16.** What is alert fatigue in HITL? 
**A.** Too many low-value reviews cause humans to approve without a check.

**Q17.** Minimum gate before GA of customer RAG? 
**A.** Typically: grounding evals, injection tests, tenancy tests, monitoring, and a clear escalation path.

**Q18.** Why treat Skills/plugins as supply chain? 
**A.** They can add instructions or tools that exfiltrate data or raise privilege.

**Q19.** Output DLP caught a secret—what next? 
**A.** Block or queue the message. Rotate the credential if it is real. Run the incident process. Fix the source prompt or data.

**Q20.** Fairness slice testing means? 
**A.** You measure quality and error rates across relevant demographic or segment slices per policy.

**Q21.** Can you satisfy GDPR with model choice alone? 
**A.** No. You also need process, contracts, minimization, rights handling, and more.

**Q22.** What is a good first step when you add browser fetch to an agent? 
**A.** Threat-model injection and SSRF. Allowlist. Strip active content. Restrict tools.

**Q23.** Why separate eval caches from prod logging retention? 
**A.** They have different purposes and legal bases. Both still need erasure and access processes.

**Q24.** What should users see on material AI decisions? 
**A.** Notice that the system is AI-assisted. Show limitations. Give a path to human review (this depends on context).

**Q25.** Runaway agent spend—safety or ops issue? 
**A.** Both. Budgets are safety and reliability controls against uncontrolled autonomy.

**Q26.** Best control for “agent can email anyone”? 
**A.** Destination allowlists plus human confirm for novel addresses plus rate limits.

**Q27.** Why keep an abuse mailbox / reporting path? 
**A.** Users and customers need a channel. This feeds trust and safety response.

**Q28.** Provider ZDR means you can skip app-level retention design? 
**A.** No. You still retain data in *your* logs, stores, and tools.

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
- [ ] I can find prompt-only “compliance” answers 
- [ ] I can describe erasure beyond SQL delete 

---

## Glossary

| Term | Meaning |
| --- | --- |
| Prompt injection | Steering a model via crafted untrusted text |
| Indirect injection | Injection via retrieved or third-party content |
| Jailbreak | Attempts to bypass safety policies |
| HITL | Human-in-the-loop oversight |
| DLP | Data loss prevention |
| DPIA | Data Protection Impact Assessment |
| BAA | Business Associate Agreement (HIPAA context) |
| ZDR | Zero Data Retention (contractual/API posture) |
| AUP | Acceptable Use Policy |
| Over-refusal | Refusal of benign requests in excess |
| Dual-use | Tech that people can use for helpful and harmful ends |
| Insecure output handling | Unsafe execution or rendering of model output |
| Use-case register | Inventory of AI systems with risk metadata |
| Red team | Adversarial testing of safety properties |
| Graduated autonomy | Phased increase of automatic action rights |

---

## Part I — Worked governance scenarios

### I1. Hospital discharge summary assistant

**Constraints:** PHI. Harm is high if the output is wrong. 
**Design:** HIPAA-aware architecture. Use a BAA where required. Minimize PHI in prompts. Use on-prem or VPC cloud AI if mandated. Cite chart sections. Require clinician HITL always before chart writeback. Run extensive safety and accuracy evals. Keep audit logs. Do not add web tools. 
**Reject:** Autonomous chart updates. Consumer API keys in wards.

### I2. HR benefits chatbot

**Constraints:** Employee personal data. Legal accuracy. 
**Design:** RAG over versioned benefits PDFs with citations. Refuse uncovered topics. Escalate to HR for exceptions. Use strict tenancy. Set retention limits. Fairness is less central than privacy and accuracy. Still monitor tone fairness.

### I3. Marketing content generator

**Constraints:** Brand, copyright, deceptive claims. 
**Design:** Claim checkers against approved product facts. Disallow unverifiable medical and finance claims. Require human edit before publish. Log prompts for trademark issues.

### I4. DevOps agent with cloud credentials

**Constraints:** Production blast radius. 
**Design:** Read-only by default. Break-glass write role with a two-person rule. Environment segregation. Command allowlists. Mandatory change tickets for prod. Session recording. Budget caps. 
**Reject:** Long-lived admin keys in MCP config that people commit to git.

---

## Part J — Policy writing for architects

Translate legal language into engineering that you can enforce:

| Policy statement | Engineering control |
| --- | --- |
| “No automated adverse hiring decisions” | Block the tool that updates ATS stages. Require HITL. |
| “Minimize personal data” | Field allowlist before the model. Redact logs. |
| “Customers can request deletion” | Erasure API hits DB + index + object store |
| “All prod changes audited” | Tool gateway logs that are immutable to SIEM |

If you cannot name the control, the policy has no real effect.

---

## Part K — Extended Q&A (29–35)

**Q29.** Why forbid web tools in a PHI assistant by default? 
**A.** This reduces egress and injection surface. It also keeps the boundary clearer for compliance.

**Q30.** Two-person rule means? 
**A.** A sensitive action requires a second authorized human approval.

**Q31.** What is claim checking in marketing genAI? 
**A.** You verify generated claims against an approved facts base before publish.

**Q32.** Why is “clinician HITL always” justified for discharge summaries? 
**A.** Clinical risk is high. Regulations and duty of care expect professional accountability.

**Q33.** Git-committed cloud admin keys—what risks? 
**A.** Leakage, lateral movement, and agent actions you cannot control. Rotate keys. Move them to a secret manager.

**Q34.** How do you enforce “minimize personal data” at request time? 
**A.** Use explicit field allowlists or redaction before you call the model.

**Q35.** What makes an AI incident severity different from generic Sev1? 
**A.** It can include mass harmful content, privacy bleed across tenants, or unsafe autonomy. Uptime is not the only factor.

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
| Hallucinated policy | Regulatory misstatement and customer harm |
| Over-broad tools | Blast radius like an insider threat, but with no insider |
| No eval gates | You ship unquantified error rates into brand channels |
| Log PII sprawl | Breach cost amplification |

Propose **controls + metrics + owners + dates**. Do not use fear alone.

### L1. Third-party risk questionnaire (MCP/SaaS AI)

Ask vendors:

- Data retention and training use 
- Subprocessors list 
- Encryption in transit and at rest 
- Incident notification SLA 
- Pen-test / SOC reports 
- Region pinning 
- Termination and deletion mechanics 

Score before enablement. Review again on major version changes.

### L2. Secure SDLC for prompts & tools

Treat prompts, Skills, and tools as code:

- PR review that includes security 
- Static checks for forbidden tool scopes 
- Secrets scanning 
- Mandatory eval job 
- Change ticket linking for prod pins 

### L3. Red team cadence

Run a quarterly external or internal red team for tier-2+ systems. Run continuous automated probes in CI for smoke-level injections.

Track findings to closure like vulnerabilities. CVE-like severity for AI issues is optional but useful.

### L4. Accessibility & inclusion

Responsible AI includes accessible UX. Use screen readers for AI notices. Give readable explanations. Do not hide human escalation with dark patterns.

Multilingual safety must not be English-only evals that only look complete.

### L5. Environmental / cost ethics (light touch)

Unbounded agent loops waste money and energy. Budgets are governance. They are not only FinOps.

### L6. Deep scenario: “summarize these emails and send fixes”

Attack emails contain injection. Design: summarize in read-only mode. Never send mail without human confirm. Strip or ignore instruction-like lines in detectors. Allowlist recipients. Show a diff of proposed sends.

### L7. Deep scenario: model suggests disabling safety filters

The product must not expose toggles that disable enterprise-mandatory controls to end users. Admin break-glass with audit only.

### L8. Documentation pack for auditors

Keep: data flow diagrams, DPIA/transfer assessments, model cards/system cards (internal). The list also includes eval reports, access reviews, incident postmortems, and vendor DPAs.

CCAR-P does not score a document binder. Scenarios still ask what evidence you would produce.

### L9. Extended Q&A 36–45

**Q36.** How do you brief an exec on injection risk in one sentence? 
**A.** “Ordinary documents can hold hidden instructions that trick the assistant into abuse of its permissions.”

**Q37.** Name four vendor questions before enabling SaaS MCP. 
**A.** Retention/training, subprocessors, regions, incident SLA (plus deletion, encryption, audit reports).

**Q38.** Why treat prompts as code? 
**A.** They change behavior like code. They need review, versioning, testing, and controlled release.

**Q39.** English-only safety evals miss what? 
**A.** Multilingual jailbreaks and toxicities. Harm to users who do not use English.

**Q40.** What is break-glass for safety toggles? 
**A.** Time-limited admin override with mandatory audit. It is not a user-facing ‘disable safety’ switch.

**Q41.** Why are budgets a governance control? 
**A.** They bound autonomous waste, abuse, and runaway tool loops.

**Q42.** What evidence might an auditor request for an AI feature? 
**A.** Data flows, DPIA, eval gates, access reviews, DPAs, incident records.

**Q43.** In the malicious email scenario, what is the safest send policy? 
**A.** Human confirm + recipient allowlist + visible diff. No autonomous send.

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
- [ ] I can argue against user-facing switches that disable safety 

### L11. Glossary additions

| Term | Meaning |
| --- | --- |
| System card | Internal or public description of system behavior and limits |
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
| Implement | Name a concrete mechanism (allowlist, HITL queue, DLP) |
| Identify | Name the risk and where it arises in the architecture |
| Ensure | Control plus measurement/monitoring |
| Design | Diagrammed layers plus owners |
| Recommend | Choose among options with trade-offs. Do not list everything. |

Avoid vague “follow best practices” selections when a specific control is offered.

### M1. Final rapid-fire Q&A 46–50

**Q46.** Best single control to add when you give an agent `delete_instance`? 
**A.** Remove it or require multi-party approval. Least privilege first.

**Q47.** What is wrong with storing raw chat that contains SSNs with no time limit? 
**A.** Retention and minimization fail. A breach gets worse. Rights requests become hard.

**Q48.** How do you “ensure” tenancy safety? 
**A.** Hard authZ filters plus automated cross-tenant retrieval tests in CI plus monitoring.

**Q49.** Customer asks to turn off logging for “privacy.” Risk? 
**A.** This can conflict with security and audit duties. Seek legal and security design of redaction vs disable.

**Q50.** Which is better evidence of safety: marketing claims or eval reports? 
**A.** Versioned eval/red-team reports with metrics and owners.



---

## Part N — Safety exam drill

Stems often combine “automate” and regulated data. Default instinct: **narrow scope, HITL, server-side authZ, eval gates**.

If an option only strengthens the system prompt, it is usually incomplete.

### N1. Abuse monitoring signals

- Spike in refusal bypass attempts 
- Unusual tool fanout per user 
- Destination novelty for email/transfer tools 
- Cross-tenant retrieval denials surge (or worse, successes) 
- Sudden cost per user 

### N2. Model card vs system card

Model cards describe base models. **System cards** describe *your* application’s behavior, evaluations, and limits. Architects own system-level documentation.

### N3. Q&A 51–56

**Q51.** If one option is “stronger system prompt” and another adds tool confirm + allowlist, which usually wins for money movement? 
**A.** Confirm + allowlist (hard controls).

**Q52.** What is a system card in enterprise practice? 
**A.** Documentation of the deployed AI system’s behavior, evals, and limitations.

**Q53.** Name three abuse-monitoring signals. 
**A.** Bypass attempts, tool fanout spikes, novel transfer destinations (and similar signals).

**Q54.** Cross-tenant retrieval *successes* in logs mean? 
**A.** Critical isolation failure. Start incident response immediately.

**Q55.** Why separate model cards from system cards? 
**A.** Provider model properties ≠ your tool privileges, data, and UX risks.

**Q56.** Default design instinct for regulated automation stems? 
**A.** Narrow scope + HITL + server authZ + eval gates.

*Next: `04-stakeholder-lifecycle-gtm.md`. Enablement details in `05`.*


## Part O — Rapid review (Governance 14%)

Governance questions reward **layered controls** and a modest view of prompts. Rapid list:

- Threat-model injection whenever untrusted text meets privileged tools. 
- HITL for irreversible or regulated impacts. Do not use approvals without review. 
- Privacy: minimization, retention, erasure including embeddings. 
- Tier use cases. Match controls to tiers. 
- Vendors/MCP are supply chain. 
- Safety evals gate releases with task quality. 
- Incidents need AI-specific severities and evidence trails. 
- System cards document *your* app, not only the base model. 
- Translate policy → engineering mechanism. 
- Never claim prompt-only compliance.

Memory aid: **Hospital writeback → always clinician HITL. DevOps agent → read-only first. Marketing claims → fact checker. Email send → confirm + allowlist.**


## Part P — Control catalog (printable)

| Risk | Preventive control | Detective control | Corrective control |
| --- | --- | --- | --- |
| Indirect injection | Privilege split. Allowlists | Injection probes. Odd tool patterns | Disable tool. Revoke sessions |
| Hallucinated advice | Citations. Refuse-on-empty | Faithfulness evals | HITL correction. User notice |
| PII leakage | Minimization. DLP | Log scanners | Rotate. Notify. Purge |
| Over-autonomy | Budgets. Confirmations | Turn/cost alerts | Flag off. Reduce scope |
| Cross-tenant bleed | Hard filters. Separate indexes | Canary tenant tests | Incident IR. Patch filters |
| Malicious MCP | Vendor review. Marketplace only | Version pin monitors | Kill switch. Revoke tokens |
| Insecure output handling | Sandbox. No raw exec | Runtime detections | Patch renderer. Revoke |
| Bias in decisions | Human accountability. Slice tests | Disparate error dashboards | Adjust product. Stop auto |

Use this catalog when a stem asks what to *implement*, *monitor*, or *do after* an incident. Match the column to the verb.


### Closing thought

Responsible AI for architects is operational. It is privileges, proofs, people in the loop, and paperwork that maps to real systems.

If your answer cannot name a mechanism, a metric, and an owner, continue to design.


*Remember:* mechanism + metric + owner. Prompt text is never the complete control answer for CCAR-P governance items. Pair it with authZ, HITL, monitoring, and evidence. *(File continues — deep-dive Part Q follows below.)*

### One more worked stem

Stem example: "Auto-approve invoice payments under $500 via Claude agent with ERP write tool."

Strong answer direction: lower the threshold further or require dual control. Enforce a payee allowlist. Use server-side authZ. Detect anomalies on novel payees. Use HITL above a risk score. Do not rely on a prompt that only promises thrift.


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

Every exam scenario that proposes “just add a stronger system prompt” is incomplete unless you consider other layers.

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
| System prompt policy | Soft behavioral norms | Determined injection | You think you are safe when you are not |
| Input classifier | Many jailbreaks/PII | Novel paraphrases. Latency | Blocks legitimate work |
| Output DLP | Secrets/PII egress | Encoded exfil | Over-redaction UX |
| Tool allowlist | Over-privilege | Logic bugs inside allowed tools | Shadow IT bypass |
| Server authZ | Confused deputy | Misconfigured ACLs | — (do not skip) |
| HITL | High-impact errors | Slow. Reviewer fatigue | Approval without review |
| Monitoring | Drift/abuse over time | Needs analysts | Alert fatigue |

### Q4. HITL design patterns

1. **Approve/reject gate** before a side effect. 
2. **Edit-then-send** (copilot). 
3. **Review sampling** (X% or uncertainty-triggered). 
4. **Four-eyes** for dual control (finance). 
5. **Async specialist queue** (legal/medical). 

Anti-patterns: HITL on every trivial FAQ (this makes the product too costly). HITL with no SLA (the queue never completes). HITL without UI that shows sources/tool args (blind approval).

### Q5. Prompt injection failure-mode analysis

| Variant | Channel | Example | Primary mitigations |
| --- | --- | --- | --- |
| Direct | User chat | “Ignore previous…” | Policy + classifiers + refuse |
| Indirect | Retrieved doc/email/web | Hidden instruction in PDF | Treat content as data. Cite. No raw tool trust |
| Tool-result injection | MCP/API payloads | Error message contains instructions | Schema validate. Sanitize. Allowlist |
| Cross-user | Shared memory/stores | Poisoned shared Skill | Tenancy. Review. Signing |
| Multimodal | Image/OCR text | Text in screenshot | Same as indirect |

**Never rely on:** “The model is aligned so we are fine.” Alignment reduces risk. Application controls remain mandatory.

### Q6. Compliance overlay mapping (scenario language, not legal advice)

| Constraint cue in stem | Architecture implications |
| --- | --- |
| GDPR / personal data | Minimization, retention, DSR erasure including indexes/logs, DPIA-minded design, transfer review |
| HIPAA / health | BAA/vendor posture, PHI minimization, access controls, audit, careful on third-party tools/web |
| FedRAMP / gov | Authorized deployment path, strict boundaries, logging, change control |
| PCI hints | Never put PAN into prompts. Tokenize. Segment |
| Children’s data | Heightened restrictions. Often avoid training/retention. Parental/policy gates |

Exam answers must **name controls**. Do not recite statute numbers.

### Q7. Fairness & transparency practical checklist

- [ ] Who is affected if the model is wrong? 
- [ ] Disparate error rates checked across relevant groups where lawful/appropriate? 
- [ ] User told they interact with AI? 
- [ ] Limitations and escalation path documented? 
- [ ] Appeals/override for consequential decisions? 
- [ ] Training/eval data governance noted?

### Q8. Incident response extras for LLM apps

Add to common IR:
- Prompt/tool trace preservation (legal hold) 
- Ability to disable tools/models/features via flag 
- Search for injection indicators 
- Customer comms templates (file 04) 
- Model/provider status correlation 

---

## Part R — Worked decision scenarios (Safety)

### R1. “Auto-send email fixes from mailbox summary”

**Reject auto-send.** Summarize and draft only. A human sends. Treat email bodies as untrusted (indirect injection). Run DLP on drafts. Audit.

### R2. DevOps agent with cloud credentials

Read-only recon tools first. Writes via change tickets + HITL. Short-lived creds. No org-admin keys in MCP. Network egress allowlists. Separate break-glass.

### R3. HR chatbot asked for medical leave legal advice

Scope to benefits docs via RAG + cite. Refuse personalized legal/medical advice. Escalate to an HR human. Log sensitive topics with care.

### R4. Marketing generator producing competitor-disparaging claims

Brand/policy output filters. Require sources for factual claims. Human publish gate. Eval for prohibited topics.

### R5. Stakeholder asks to “turn off safety filters for speed”

Architect response: safety filters are risk controls tied to tier. Propose measured latency options (caching, routing) instead. Document residual risk if any control is narrowed with approval.

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
| Actions are reversible and blast radius is low | Money, health, safety, or legal impact that you cannot reverse |
| Strong evals + monitoring | Sparse data / novel intents |
| Clear policy engines encode rules | Rules require judgment calls |
| Exception rate is within staffing | Reviewers already have too much work |

---

## Part U — Extended Q&A (57–65)

**Q57.** Strongest control against unauthorized data access via tools? 
**A.** **Server-side authorization** on the data plane using verified identity.

**Q58.** Indirect injection best single description? 
**A.** Malicious instructions embedded in **content the model reads** (docs, web, emails). This is not only in the user chat box.

**Q59.** Select TWO for Tier-3 payout agent. The list also includes mandatory HITL, open web server tool for all users, full audit traces, temperature=1.5 creative mode.
**A.** Mandatory HITL and full audit traces.

**Q60.** Constitutional AI means your app is compliant with HIPAA? 
**A.** **No**—base alignment ≠ application compliance. You still design controls and vendor posture.

**Q61.** Reviewers approve without a real check—what do you change? 
**A.** Reduce volume (better auto for T0/T1). Improve UI with sources/tool args. Rotate reviewers. Sample quality audits.

**Q62.** GDPR erasure request—what systems to touch? 
**A.** Primary DB **and** vector indexes, caches, eval sets, logs (per policy), backups per legal schedule.

**Q63.** Best response if product wants the model to execute shell from output? 
**A.** **Sandbox / deny by default**. Never pipe unsandboxed LLM text to privileged shells.

**Q64.** Dual-use request for detailed biological harm—in product policy terms? 
**A.** Refuse or give a safe completion per acceptable use. Escalate abuse patterns. Do not provide actionable harmful detail.

**Q65.** Select THREE layers in defense in depth. Tool allowlist, output DLP, server authZ, “trust the vendor alone”, disable all logging for ineffective privacy control.
**A.** Allowlist, output DLP, server authZ.

---

## Part V — Rapid review (Governance 14%)

- Design controls. Do not use slogans. 
- Defense in depth. Prompts are one layer. 
- Injection: direct + indirect + tool results. 
- HITL by impact tier. Avoid approvals without review. 
- AuthZ in code. Least privilege tools. 
- Compliance = controls + evidence. Not slogans. 
- Fairness, notices, appeals for consequential AI. 
- IR needs traces, kill switches, and clear owners.

*Pair with Integration authZ in `02` and lifecycle risk talk-tracks in `04`.*


---

## Part W — Abuse monitoring, model cards, and exam stems

### W1. Abuse monitoring signal examples

- Sudden spike in tool calls per user 
- Repeated jailbreak phrasings 
- DLP hit clusters 
- Anomalous exfil-sized outputs 
- New MCP server reaching odd egress 

Wire to SOC/IR playbooks. Do not collect sensitive content beyond policy.

### W2. Model card vs system card (architect speak)

**Model card** (provider): capabilities, limitations, eval highlights of the base model. 
**System card / system documentation** (you): how *your* application uses the model—data flows, controls, residual risks.

The exam cares that you know **application** accountability remains yours.

### W3. Quick stems

**Q66.** Product removes output filters to “sound more helpful.” Risk? 
**A.** Increased leakage/policy violations. This violates defense in depth. You need formal risk acceptance. Do not remove filters in silence.

**Q67.** Best evidence for auditors that HITL works? 
**A.** Ticketed approvals with reviewer identity, timestamps, and linked model/tool traces. A policy PDF alone is not enough.

**Q68.** FedRAMP-like constraint in stem—what changes first? 
**A.** Deployment/boundary choices and logging/change control. A nicer prompt is not enough.

*File 03 primary-study target band: 6000–9000 words.*

 


