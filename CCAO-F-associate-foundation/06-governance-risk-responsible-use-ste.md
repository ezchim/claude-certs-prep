# Domain 06 — Governance, Risk & Responsible Use
## Maps to official CCAO-F **Governance, Risk, and Responsible Use** (~15%, ~9 questions)

> **Note:** This edition follows the ASD-STE100 Simplified Technical English writing rules. Simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Projects, Artifacts, Skills, Connectors, Research, Memory, PII, GDPR, MCP) are exceptions and stay as written.

> **Dedup note (2026-08-23):** This is one primary-study copy. Earlier builds repeated the same drill blocks about 7 times. Those duplicates are gone. The content now matches the Domain 03 depth.

## Disclaimer

These notes are original CCAO-F study notes. They are for people who are not developers. They use claude.ai (Projects, Artifacts, Skills, Connectors, Research). The notes use public Anthropic Usage Policy themes and privacy documentation. They also use public Claude Academy (AI Fluency Diligence) and the published CCAO-F blueprint. The notes are independent. They are not affiliated with Anthropic. Nothing here is legal advice. Check live policy and privacy details on support.claude.com and the Anthropic policy pages.

---

## Overview

The official blueprint verbs are: **identify** appropriate and inappropriate use cases. **Apply** data sensitivity, regulatory, and privacy considerations. **Follow** organizational AI policies and governance standards. **Understand** the ethical implications of AI usage. You need practical judgment. You are not a lawyer. Know what to refuse. Know what needs safeguards. Know which data may go where. Know who owns deployed outputs. The official sample question rewards this action: **anonymize regulated identifiers before upload, consistent with policy**. That judgment pattern applies to many questions.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Use-case appropriateness | identify, refuse, escalate | Disallowed vs high-risk-with-safeguards vs ordinary |
| Data sensitivity & privacy | apply, classify, minimize | Data tiers. PII minimization. Venue choice |
| Org policy compliance | follow, check, escalate | Policy owners. **Shadow IT** avoidance |
| Ethics | understand, disclose | Bias, transparency, ownership of outputs |

---

## Deep notes

### 1. The policy reasoning pattern (memorize the pattern, not clauses)

1. **Disallowed?** Refuse. Do not look for phrasing tricks. Universal categories include violent crimes, child exploitation, scams/fraud, malicious cyber operations, and serious privacy invasions.
2. **High-risk but allowed?** Legal, financial, employment, and medical-adjacent decisions affect people. Proceed **only with safeguards**: human-in-the-loop, disclosure, and evaluation gates.
3. **Ordinary business?** Still apply privacy minimization and org rules.
4. **Uncertain?** Escalate to the policy owner. Do not invent a legal answer.

On the exam, jailbreak-style options are always wrong. An example is "rephrase to get around the refusal." A safety refusal on genuinely disallowed content is correct behavior. It is not a defect. The policy honors context. Defensive security work with the owner's consent differs from malicious hacking. You can state benign intent plainly (Domain 07).

### 2. Data sensitivity in practice

Classify data before you paste it: **public / internal / confidential / restricted.** Then:

- **Minimize:** Share the minimum that the task needs. Anonymize or redact regulated identifiers (names, account numbers) before upload when policy restricts them. This is the official sample answer.
- **Venue:** Personal Free accounts are usually the wrong venue for company confidential data. Managed Team/Enterprise workspaces exist to carry organizational controls.
- **Training/retention defaults:** Consumer offerings and commercial offerings differ. Anthropic typically does **not** train on commercial/organizational data by default. Check current terms in the privacy docs. Do not repeat unverified claims. "Instruct Claude not to retain it" is **not** a policy control. That line is a sample-question distractor.
- **Memory & retention surface (new):** Claude's Memory stores context from chats. Treat memory entries as data. Put sensitive single-use questions in **incognito chats**. Exclude Incognito chats from history, memory, and search. On Enterprise, the org still retains incognito chats for a minimum safety period. Admin data exports include them. Incognito is a *memory/history* control. It is not a way around org visibility. Data exports include Memory. Review and delete entries per policy (see `05-…` §Memory).

### 3. Connector and sharing governance

Connectors expand the range of harm. Before you connect Gmail, Drive, or Slack, know what Claude can **read**. Know whether **writes** are possible. Prefer read-only for analysis. Write scopes (send email, update HRIS/CRM) are governance decisions. They need approval and human gates. Enterprise-managed auth and domain restrictions may apply. Project sharing is itself a control: "Can view" vs "Can edit". Do not share restricted knowledge org-wide. Shared Artifacts can expose data. Read the confirmation dialogs. Free/personal accounts are the wrong venue for company secrets.

### 4. Transparency, disclosure, and ownership (Diligence)

The Diligence triad is **Creation** (quality of AI-assisted work), **Transparency** (honesty about AI's role), and **Deployment** (you own what you release). Disclose AI assistance when policy, high-risk context, or customer-facing use requires it. Customer-facing chatbots disclose that they are AI. "Claude said it is fine" is never a compliance defense. The deployer owns the output. Fabricated compliance claims are a violation. An example is "this is GDPR-approved."

### 5. Ethics scenarios that recur

- **Hiring/employment assist:** Ranking of candidates needs bias scrutiny, a human decision, and often disclosure. Fully automated hire/reject is the canonical inappropriate use.
- **Finance/legal/medical-sounding advice to customers:** Licensed-human review plus disclaimers.
- **Bias:** Stereotype defaults, skewed examples, and one-sided analyses. Evaluate impacts on people. Require counterarguments and criteria (Domain 03 methods that serve Domain 06 goals).
- **Synthetic media/deception:** Non-consensual deepfakes and impersonation are disallowed.
- **Copyright/confidentiality:** Do not paste third-party secrets or bulk copyrighted text into unmanaged contexts.
- **Accessibility & inclusion:** Examples and outputs must not leave people out without notice.

### 6. Following org policy without becoming shadow IT

Help teams adopt Claude **inside policy**. Publish allowed tools and venues. Publish data tiers, review requirements, example prompts, and escalation paths. Training works better than warnings that cause fear. Metrics must count policy incidents, not only speed wins. When org policy and personal convenience conflict, policy wins on the exam every time. When policy is silent, escalate to its owner. Do not invent a rule. Report harmful or broken outputs through product channels when the case warrants it.

### 7. Mini-cases (judgment reps)

**Case 1 — The helpful shortcut.** An ops manager pastes the full customer database extract into chat "to find churn patterns." The extract includes names, emails, and account balances. Policy classifies this data as restricted. Correct path: aggregate or anonymize first. Drop direct identifiers. Keep behavioral columns. Run the work in the managed workspace. Involve the data owner. The analysis is legitimate. The handling is not.

**Case 2 — The quiet chatbot.** A team ships a Claude-backed FAQ widget. There is no AI disclosure. There is no human escalation path. Two failures exist. Transparency: customer-facing AI discloses. Deployment ownership: there is no route for wrong answers to reach humans. Fix this before you scale. Do not wait for the first complaint.

**Case 3 — The eager connector.** An admin grants a write-scope HRIS connector so Claude can "tidy records." There is no governance review. There is no gate. Even with good intent, this is the range-of-harm trap. Writes of people-data need the system owner's approval. They need least privilege. They need human confirmation per change. Read-only reporting delivers 90% of the value at 10% of the risk.

**Case 4 — The confident compliance memo.** Claude drafts "our data flows are GDPR-compliant" for a client email. Nobody with authority reviews it. Fabricated compliance claims are a governance violation even when they sound plausible. Regulated assertions ship only with the accountable owner's sign-off (the Domain 03 L4 gate that serves Domain 06).

**Case 5 — The screening spreadsheet.** A recruiter asks Claude to score 400 resumes and auto-email rejections to the bottom half. Split the work. Claude may structure and summarize applications. Ranking feeds a human decision with bias checks. Rejection *decisions and sends* stay human. This is high-risk-with-safeguards, split correctly.

### 8. Multiple-response pattern bank

Wrong pairs that recur:

- "Rephrase until it answers" + "report the refusal as an outage" (jailbreak + refusal-as-bug).
- "Upload as-is, it is internal" + "never analyze customer data at all" (policy breach + paralysis). Anonymization is the middle path.
- "Claude reviewed it, ship it" + "humans must draft everything" (fake sign-off + anti-Delegation).
- "Ban connectors" + "connect with write access everywhere" (absolutes).

Correct combinations pair a **data-handling** move (classify, minimize, right venue) with an **oversight** move (human gate, disclosure, escalation to the policy owner).

---

## Decision trees

| Situation | Action |
|---|---|
| Disallowed harmful request | Refuse. Do not rephrase to bypass the refusal. |
| High-risk decisioning (employment/finance/legal) | Human oversight + disclosure + evaluation gates |
| Regulated identifiers in a file to analyze | Anonymize or redact first, per policy |
| Confidential data, personal Free account | Wrong venue. Use the managed workspace. |
| Customer-facing AI | Disclosure norms apply |
| Sensitive one-off question | Incognito chat. On Enterprise, admin export/retention still applies. |
| Write-scope connector on a sensitive system | Governance approval + human gate |
| Uncertain legality/policy | Escalate to the policy owner |
| Safety refusal on allowed work | Clarify benign intent within policy (Domain 07). Never jailbreak. |

---

## Exam traps

1. "Claude said it is fine" as legal/compliance cover
2. You paste restricted data into a personal account against policy
3. "Instruct Claude not to retain it" as a privacy control
4. Jailbreak options presented as cleverness
5. You skip disclosure on customer-facing or high-risk use
6. You treat safety refusals as bugs to fix
7. Fully automated employment/credit decisions
8. Write connectors on sensitive systems without approval
9. You believe incognito hides activity from Enterprise governance (it is a memory/history control)
10. Fabricated compliance claims ("certified GDPR-safe output")

---

## Practice Q&A (18) — scenario stems with answers and rationales

**Q1.** A user asks Claude for malware to "test" a competitor's network. Response pattern?
**A:** Refuse — malicious cyber operations are disallowed. There is no consent and no authorization. Defensive work with the system owner's consent is the different, legitimate case.

**Q2.** A PM wants to upload a spreadsheet with customer names and account numbers. Policy restricts regulated personal data. Most appropriate action?
**A:** Remove or anonymize the identifiers before upload, consistent with policy. The analysis proceeds without exposure of protected data (official sample answer pattern).

**Q3.** Select TWO valid privacy moves for routine work.
**A:** Minimize PII to what the task needs. Follow the org's data classification for venue and handling. ("Tell Claude to forget it" is not a control.)

**Q4.** HR wants Claude to auto-reject bottom-scored resumes with no human step. Verdict?
**A:** Inappropriate. Employment decisions are high-risk. They need human oversight, bias scrutiny, and disclosure. Claude may assist (summarize, structure). Claude may not decide.

**Q5.** A customer-facing support bot built on Claude — what governance requirement applies to the interface?
**A:** Disclose that customers talk to AI. Keep escalation to humans available. Transparency norms apply to customer-facing use.

**Q6.** An analyst uses their personal Free account for board-confidential financials "to save time." Problem?
**A:** Wrong venue. Personal accounts lack org controls. Policy on confidential data likely prohibits this. Use the managed workspace. Convenience never outranks classification.

**Q7.** Does commercial/organizational Claude usage train the model on your data by default?
**A:** Typically not by default for commercial offerings. Check current privacy documentation. Do not recite folklore. "Verify, then rely" is the exam-safe posture.

**Q8.** Ops wants a connector that can write to the HRIS to "save clicks." Governance shape?
**A:** Approval by the system's governance owner. Least-privilege scope. A human gate before writes. Reads for analysis are the easier grant. Writes move people-data.

**Q9.** A hiring-assist workflow shows the model favoring one university in rankings. Required response?
**A:** Bias scrutiny. Test with counterfactual candidates. Add criteria. Keep humans deciding. Document the check. If you ship it without a check, you fail ethics and possibly law.

**Q10.** Claude refuses a request that seems clearly legitimate. Wrong answer vs right answer?
**A:** Wrong: jailbreak or rephrase-to-trick. Right: state the benign, authorized context plainly within policy. Escalate if it persists (Domain 07).

**Q11.** When must AI assistance be disclosed in deliverables?
**A:** When org policy, high-risk context, or customer-facing use requires it. Transparency is one third of Diligence. The deployer owns the call.

**Q12.** A teammate asks Claude whether their data-sharing plan is legal in the EU and acts on the answer. Failure?
**A:** The teammate treats Claude as counsel. Escalate uncertain legality to the compliance/policy owner. Claude can draft the question well. Claude cannot own the answer.

**Q13.** Select TWO facts about incognito chats relevant to governance.
**A:** Exclude They from history, memory, and search. On Enterprise the org still retains them for a minimum safety period. Data exports include them. They are not invisible to the org.

**Q14.** Someone shares a Project holding restricted contracts org-wide with edit rights. Assessment?
**A:** This is a permission failure two times. Restricted knowledge goes to an org-wide audience. The share grants edit rights, not view. Sharing scope is a governance control.

**Q15.** Marketing wants Claude to fabricate "9 out of 10 dentists" style stats for copy. Call?
**A:** Refuse the fabrication. Deceptive claims are inappropriate use. Supply real sourced figures or drop the claim (Domain 03 enforcement of a Domain 06 rule).

**Q16.** A near-miss: an associate almost emailed AI-drafted financial advice to a client unreviewed. What controls were missing?
**A:** High-risk safeguards: licensed-human review, disclosure, and a human gate on the send. High-risk allowed does not equal high-risk unsupervised.

**Q17.** What belongs in a team's responsible-use enablement pack?
**A:** Allowed tools/venues. Data tiers with examples. Review requirements by risk. Example prompts. Escalation paths. Adoption **inside policy** beats **Shadow IT**.

**Q18.** Select TWO cases that are disallowed outright (refuse. Do not safeguard).
**A:** Non-consensual deepfake/impersonation content. Instructions that enable scams or malicious cyber operations. (Employment decisioning is high-risk-with-safeguards, not refuse-outright.)

---

## Quick review checklist (pre-exam)

- [ ] Policy pattern: disallowed → refuse. High-risk → safeguards. Ordinary → minimize. Uncertain → escalate
- [ ] Data tiers + anonymize regulated identifiers before upload
- [ ] Venue discipline: personal Free ≠ confidential work
- [ ] Training/retention: commercial typically not by default — verify current docs
- [ ] Incognito = memory/history control, not governance invisibility
- [ ] Connectors: read-only default. Writes = approval + human gate
- [ ] Disclosure for customer-facing and high-risk use. Deployer owns outputs
- [ ] Refusals on disallowed content are correct behavior

---

## Glossary

| Term | Meaning |
|---|---|
| **Usage Policy** | Anthropic's rules: universal disallowed categories + high-risk requirements |
| **High-risk use case** | Allowed use that affects people's rights, money, or health — needs safeguards |
| **Human-in-the-loop** | Required human review/decision inside the workflow |
| **Data classification** | Public / internal / confidential / restricted handling tiers |
| **PII minimization** | Share the least personal data the task needs. Redact or anonymize |
| **Disclosure** | Be honest about AI's role where it matters |
| **Diligence triad** | Creation, Transparency, Deployment — you own what you ship |
| **Shadow IT** | Unsanctioned tool use around policy. Fix with enablement, not fear |
| **Constitutional AI** | High-level: principles-based training that guides safer model behavior |
