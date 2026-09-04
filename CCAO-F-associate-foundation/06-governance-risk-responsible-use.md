---
title: Governance, Risk & Responsible Use
---

# Domain 06 — Governance, Risk & Responsible Use
## Maps to official CCAO-F **Governance, Risk, and Responsible Use** (~15%, ~9 questions)

> **Dedup note (2026-08-23):** Rewritten as a single primary-study copy. Earlier builds repeated the same drill blocks ~7×; duplicates removed and content deepened to the Domain 03 standard.

## Disclaimer

Original CCAO-F study notes for non-developers using claude.ai (Projects, Artifacts, Skills, Connectors, Research). Grounded in public Anthropic Usage Policy themes, privacy documentation, public Claude Academy (AI Fluency Diligence), and the published CCAO-F blueprint. Independent; not affiliated with Anthropic; nothing here is legal advice. Verify live policy and privacy details on support.claude.com and the Anthropic policy pages.

---

## Overview

Official blueprint verbs: **identify** appropriate and inappropriate use cases; **apply** data sensitivity, regulatory, and privacy considerations; **follow** organizational AI policies and governance standards; **understand** the ethical implications of AI usage. Non-lawyer practical judgment for associates: know what to refuse, what needs safeguards, what data may go where, and who owns deployed outputs. The official sample question for this domain rewards *anonymizing regulated identifiers before upload, consistent with policy* — that judgment pattern generalizes.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Use-case appropriateness | identify, refuse, escalate | Disallowed vs high-risk-with-safeguards vs ordinary |
| Data sensitivity & privacy | apply, classify, minimize | Data tiers; PII minimization; venue choice |
| Org policy compliance | follow, check, escalate | Policy owners; shadow-IT avoidance |
| Ethics | understand, disclose | Bias, transparency, ownership of outputs |

---

## Deep notes

### 1. The policy reasoning pattern (memorize the pattern, not clauses)

1. **Disallowed?** → refuse; do not look for phrasing tricks. Universal categories include violent crimes, child exploitation, scams/fraud, malicious cyber operations, and serious privacy invasions.
2. **High-risk but allowed?** (legal, financial, employment, medical-adjacent decisions affecting people) → proceed **only with safeguards**: human-in-the-loop, disclosure, evaluation gates.
3. **Ordinary business?** → still apply privacy minimization and org rules.
4. **Uncertain?** → escalate to the policy owner; don't improvise legality.

On the exam, jailbreak-flavored options ("rephrase to get around the refusal") are always wrong; a safety refusal on genuinely disallowed content is correct behavior, not a defect. Nuance the policy does honor: context matters — defensive security work with the owner's consent differs from malicious hacking; benign intent can be stated plainly (Domain 07).

### 2. Data sensitivity in practice

Classify before pasting: **public / internal / confidential / restricted.** Then:

- **Minimize:** share the minimum needed; anonymize or redact regulated identifiers (names, account numbers) before upload when policy restricts them — the official sample answer.
- **Venue:** personal Free accounts are usually the wrong venue for company confidential data; managed Team/Enterprise workspaces exist precisely to carry organizational controls.
- **Training/retention defaults:** consumer vs commercial offerings differ; commercial/organizational offerings are typically **not** trained on by default — verify current terms in the privacy docs rather than reciting folklore. "Instruct Claude not to retain it" is **not** a policy control (sample question distractor).
- **Memory & retention surface (new):** Claude's Memory stores context from chats — treat memory entries as data. Sensitive one-offs belong in **incognito chats** (excluded from history, memory, search); on Enterprise, incognito chats are still retained for a minimum safety period and appear in admin data exports — incognito is a *memory/history* control, not a way around org visibility. Memory is included in data exports; review and delete entries per policy (see `05-…` §Memory).

### 3. Connector and sharing governance

Connectors expand the blast radius: before connecting Gmail/Drive/Slack, know what Claude can **read** and whether **writes** are possible. Prefer read-only for analysis; write scopes (send email, update HRIS/CRM) are governance decisions requiring approval and human gates. Enterprise-managed auth and domain restrictions may apply. Project sharing is itself a control: "Can view" vs "Can edit"; avoid org-wide sharing of restricted knowledge; shared Artifacts can expose data — read the confirmation dialogs. Free/personal accounts are the wrong venue for company secrets.

### 4. Transparency, disclosure, and ownership (Diligence)

The Diligence triad: **Creation** (quality of AI-assisted work), **Transparency** (honesty about AI's role), **Deployment** (you own what you release). Disclose AI assistance when policy, high-risk context, or customer-facing use requires it — customer-facing chatbots disclose that they're AI. "Claude said it's fine" is never a compliance defense; the deployer owns the output. Fabricating compliance claims ("this is GDPR-approved") is itself a violation.

### 5. Ethics scenarios that recur

- **Hiring/employment assist:** ranking candidates demands bias scrutiny, human decision, and often disclosure — fully automated hire/reject is the canonical inappropriate use.
- **Finance/legal/medical-sounding advice to customers:** licensed-human review + disclaimers.
- **Bias:** stereotype defaults, skewed examples, one-sided analyses — evaluate impacts on people, require counterarguments and criteria (Domain 03 machinery serving Domain 06 goals).
- **Synthetic media/deception:** non-consensual deepfakes and impersonation are disallowed territory.
- **Copyright/confidentiality:** don't paste third-party secrets or bulk copyrighted text into unmanaged contexts.
- **Accessibility & inclusion:** examples and outputs shouldn't quietly exclude.

### 6. Following org policy without becoming shadow IT

Help teams adopt Claude inside the lines: publish allowed tools and venues, data tiers, review requirements, example prompts, escalation paths. Training beats scare posters. Metrics should count policy incidents, not only speed wins. When org policy and personal convenience conflict, policy wins on the exam every time; when policy is silent, escalate to its owner rather than improvising. Report harmful or broken outputs through product channels when warranted.

### 7. Mini-cases (judgment reps)

**Case 1 — The helpful shortcut.** An ops manager pastes the full customer database extract (names, emails, account balances) into chat "to find churn patterns." Policy classifies this restricted. Correct path: aggregate or anonymize first (drop direct identifiers, keep behavioral columns), run in the managed workspace, and involve the data owner. The analysis was legitimate; the handling wasn't.

**Case 2 — The quiet chatbot.** A team ships a Claude-backed FAQ widget with no AI disclosure and no human escalation path. Two failures: transparency (customer-facing AI discloses) and deployment ownership (no route for wrong answers to reach humans). Fix before scale, not after the first complaint.

**Case 3 — The eager connector.** An admin grants a write-scope HRIS connector so Claude can "tidy records." No governance review, no gate. Even with good intent, this is the blast-radius trap: people-data writes need the system owner's approval, least privilege, and human confirmation per change. Read-only reporting would have delivered 90% of the value at 10% of the risk.

**Case 4 — The confident compliance memo.** Claude drafts "our data flows are GDPR-compliant" for a client email. Nobody with authority reviewed it. Fabricated compliance claims are a governance violation even when plausible; regulated assertions ship only with the accountable owner's sign-off (Domain 03 L4 gate serving Domain 06).

**Case 5 — The screening spreadsheet.** A recruiter asks Claude to score 400 resumes and auto-email rejections to the bottom half. Split it: Claude may structure and summarize applications; ranking feeds a human decision with bias checks; rejection *decisions and sends* stay human. High-risk-with-safeguards, decomposed correctly.

### 8. Multiple-response pattern bank

Recurring wrong pairs: "rephrase until it answers" + "report the refusal as an outage" (jailbreak + refusal-as-bug); "upload as-is, it's internal" + "never analyze customer data at all" (policy breach + paralysis — anonymization is the middle); "Claude reviewed it, ship it" + "humans must draft everything" (fake sign-off + anti-Delegation); "ban connectors" + "connect with write access everywhere" (absolutes). Correct combinations pair a **data-handling** move (classify, minimize, right venue) with an **oversight** move (human gate, disclosure, escalation to the policy owner).

---

## Decision trees

| Situation | Action |
|---|---|
| Disallowed harmful request | Refuse; no rephrasing games |
| High-risk decisioning (employment/finance/legal) | Human oversight + disclosure + evaluation gates |
| Regulated identifiers in a file to analyze | Anonymize/redact first, per policy |
| Confidential data, personal Free account | Wrong venue — use the managed workspace |
| Customer-facing AI | Disclosure norms apply |
| Sensitive one-off question | Incognito chat; on Enterprise, remember admin export/retention still applies |
| Write-scope connector on a sensitive system | Governance approval + human gate |
| Uncertain legality/policy | Escalate to policy owner |
| Safety refusal on allowed work | Clarify benign intent within policy (Domain 07); never jailbreak |

---

## Exam traps

1. "Claude said it's fine" as legal/compliance cover
2. Pasting restricted data into a personal account against policy
3. "Instruct Claude not to retain it" as a privacy control
4. Jailbreak options dressed as resourcefulness
5. Skipping disclosure on customer-facing or high-risk use
6. Treating safety refusals as bugs to fix
7. Fully automated employment/credit decisions
8. Write connectors on sensitive systems without approval
9. Believing incognito hides activity from Enterprise governance (it's a memory/history control)
10. Fabricated compliance claims ("certified GDPR-safe output")

---

## Practice Q&A (18) — scenario stems with answers and rationales

**Q1.** A user asks Claude for malware to "test" a competitor's network. Response pattern?
**A:** Refuse — malicious cyber operations are disallowed; no consent, no authorization. Defensive work with the system owner's consent is the different, legitimate case.

**Q2.** A PM wants to upload a spreadsheet with customer names and account numbers; policy restricts regulated personal data. Most appropriate action?
**A:** Remove or anonymize the identifiers before upload, consistent with policy — the analysis proceeds without exposing protected data (official sample answer pattern).

**Q3.** Select TWO valid privacy moves for routine work.
**A:** Minimize PII to what the task needs; follow the org's data classification for venue and handling. ("Tell Claude to forget it" is not a control.)

**Q4.** HR wants Claude to auto-reject bottom-scored resumes with no human step. Verdict?
**A:** Inappropriate — employment decisions are high-risk and need human oversight, bias scrutiny, and disclosure. Claude may assist (summarize, structure), not decide.

**Q5.** A customer-facing support bot built on Claude — what governance requirement applies to the interface?
**A:** Disclose that customers are talking to AI, with escalation to humans available — transparency norms for customer-facing use.

**Q6.** An analyst uses their personal Free account for board-confidential financials "to move fast." Problem?
**A:** Wrong venue: personal accounts lack org controls; policy on confidential data likely prohibits it. Use the managed workspace — convenience never outranks classification.

**Q7.** Does commercial/organizational Claude usage train the model on your data by default?
**A:** Typically not by default for commercial offerings — but verify current privacy documentation rather than reciting folklore; "verify, then rely" is the exam-safe posture.

**Q8.** Ops wants a connector that can write to the HRIS to "save clicks." Governance shape?
**A:** Approval by the system's governance owner, least-privilege scope, and a human gate before writes. Reads for analysis are the easier grant; writes move people-data.

**Q9.** A hiring-assist workflow shows the model favoring one university in rankings. Required response?
**A:** Bias scrutiny: test with counterfactual candidates, add criteria, keep humans deciding, document the check. Shipping it unexamined fails ethics and possibly law.

**Q10.** Claude refuses a request that seems clearly legitimate. Wrong answer vs right answer?
**A:** Wrong: jailbreak/rephrase-to-trick. Right: state the benign, authorized context plainly within policy; escalate if it persists (Domain 07).

**Q11.** When must AI assistance be disclosed in deliverables?
**A:** When org policy, high-risk context, or customer-facing use requires it — Transparency is one third of Diligence, and the deployer owns the call.

**Q12.** A teammate asks Claude whether their data-sharing plan is legal in the EU and acts on the answer. Failure?
**A:** Treating Claude as counsel — uncertain legality escalates to the compliance/policy owner. Claude can draft the question well; it cannot own the answer.

**Q13.** Select TWO facts about incognito chats relevant to governance.
**A:** They're excluded from history, memory, and search; on Enterprise they're still retained for a minimum safety period and included in data exports — not invisible to the org.

**Q14.** Someone shares a Project holding restricted contracts org-wide with edit rights. Assessment?
**A:** Permission failure twice over: restricted knowledge to an org-wide audience, and edit (not view) rights. Sharing scope is a governance control.

**Q15.** Marketing wants Claude to fabricate "9 out of 10 dentists" style stats for copy. Call?
**A:** Refuse the fabrication — deceptive claims are inappropriate use; supply real sourced figures or drop the claim (Domain 03 enforcement of a Domain 06 rule).

**Q16.** A near-miss: an associate almost emailed AI-drafted financial advice to a client unreviewed. What controls were missing?
**A:** High-risk safeguards: licensed-human review, disclosure, and a human gate on the send. High-risk allowed ≠ high-risk unsupervised.

**Q17.** What belongs in a team's responsible-use enablement pack?
**A:** Allowed tools/venues, data tiers with examples, review requirements by risk, example prompts, escalation paths — adoption inside the lines beats shadow IT.

**Q18.** Select TWO cases that are disallowed outright (refuse, don't safeguard).
**A:** Non-consensual deepfake/impersonation content; instructions enabling scams or malicious cyber operations. (Employment decisioning is high-risk-with-safeguards, not refuse-outright.)

---

## Quick review checklist (pre-exam)

- [ ] Policy pattern: disallowed → refuse; high-risk → safeguards; ordinary → minimize; uncertain → escalate
- [ ] Data tiers + anonymize regulated identifiers before upload
- [ ] Venue discipline: personal Free ≠ confidential work
- [ ] Training/retention: commercial typically not by default — verify current docs
- [ ] Incognito = memory/history control, not governance invisibility
- [ ] Connectors: read-only default; writes = approval + human gate
- [ ] Disclosure for customer-facing and high-risk use; deployer owns outputs
- [ ] Refusals on disallowed content are correct behavior

---

## Glossary

| Term | Meaning |
|---|---|
| **Usage Policy** | Anthropic's rules: universal disallowed categories + high-risk requirements |
| **High-risk use case** | Allowed use affecting people's rights/money/health — needs safeguards |
| **Human-in-the-loop** | Required human review/decision inside the workflow |
| **Data classification** | Public / internal / confidential / restricted handling tiers |
| **PII minimization** | Sharing the least personal data the task needs; redact/anonymize |
| **Disclosure** | Being honest about AI's role where it matters |
| **Diligence triad** | Creation, Transparency, Deployment — you own what you ship |
| **Shadow IT** | Unsanctioned tool use around policy; fix with enablement, not fear |
| **Constitutional AI** | High-level: principles-based training guiding safer model behavior |
