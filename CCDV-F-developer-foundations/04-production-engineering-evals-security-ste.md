---
title: Chapter 04 — Production Engineering, Evals & Security — Simplified Technical English
pack: CCDV-F Developer Foundations
disclaimer: Original study notes — independent and not official course content
approx_length: STE edition (ASD-STE100) — primary study
updated: 2026-08-23
---

# Chapter 04 — Production Engineering, Evals, and Security

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, MCP, prompting, caching, effort, p95) are exceptions and stay as written. Model IDs and prices change. Learn the decision rules. Check the current model cards before the exam.

> **Disclaimer:** These are original study notes. They use public Anthropic safety and product patterns. They also use common LLM-app eval practice. They map to the public CCDV-F domains Security and Safety (8.1%) and Eval/Testing/Debugging (2.6%). They also cover production parts of Applications and Integration (33.1%). They are independent. They are not official course content.

**Primary maps:** Security 8.1% · Eval/Testing/Debugging 2.6% · production Integration.
**Secondary:** Agents reliability · MSO justification via evals · MCP and tool hardening.

---

## 1. Overview

When you ship Claude, you solve an operations problem. You have three goals. You cannot split these goals.

1. Make the system **correct** — use evals, golden sets, and regression gates.
2. Make the system **safe** — protect secrets and PII. Block injection. Gate destructive actions. Resist abuse.
3. Make the system **operable** — set SLOs. Use tracing. Plan incident response. Control cost. Manage change.

The exam weights for Eval and Security are small. They still act as filters. They remove designs that look good in multi-domain stems. Study them as decision constraints. Do not treat them as optional chapters.

---

## 2. Key map

**Pillar: Eval** — Does behavior improve on real tasks? **Controls:** golden sets, rubrics, side-effect asserts.
**Pillar: Debug** — Why does this trace fail? **Controls:** replay, diffs, tool logs.
**Pillar: Security** — What if the model is wrong or an attacker uses it? **Controls:** deny lists, authz, redaction, HITL.
**Pillar: Safety** — Does output cause harm or break policy? **Controls:** filters, classifiers, refusals.
**Pillar: Production eng** — Will it stay up within budget? **Controls:** SLOs, queues, caps, runbooks.

---

## 3. Deep notes — Evaluation

### 3.1 Why evals are better than opinions

LLM changes are easy to ship. They are hard to notice. Evals convert opinions into measured deltas. CCDV-F narratives expect eval evidence. You need this evidence before you change model, prompt, or tools in production.

### 3.2 Eval types

**Unit-style checks:** schema validity and tool-arg shape. They detect breakage fast.
**Golden sets:** labeled cases with exact or fuzzy match criteria.
**Rubric or LLM-as-judge:** qualitative quality. You need calibration and human audits.
**Side-effect tests:** the tools that the agent called. These tests are critical for agents.
**Adversarial packs:** injections and jailbreaks. They overlap with security.
**Online methods:** shadow, A/B, canary on real traffic.
**Cost and latency budgets:** pair them with quality. Never use them alone.

### 3.3 Building a golden set

Sample real tasks after a privacy review. Label expected answers or acceptable rubrics. Include edge cases and known failures. Version the set. Freeze it during comparisons. Segment by difficulty and intent. Average scores must not hide hard-slice regressions.

### 3.4 Agent eval specifics

Assert final answer quality. Assert tools used and tools not used. Assert step count under budget. Assert no forbidden tools. Assert idempotent behavior under retry. A correct paragraph after a forbidden delete is still a failure.

### 3.5 LLM-as-judge pitfalls

Judges can share generator weaknesses. Pairwise comparisons show position bias. Vague rubrics create noise. Spot-check with humans on a schedule.

### 3.6 Gating deploys

CI runs unit schema checks plus a small golden set. Staging runs the full golden set plus an adversarial pack. Canary watches online metrics and safety monitors. Fleet rollout continues monitoring. Keep a ready rollback.

### 3.7 Debugging loop

Capture a redacted failing trace. Reproduce with pinned versions. Diff prompt, tool, model, and config. Hypothesize among context issues, tool errors, policy conflicts, and capacity problems. Fix with the smallest change. Add a regression case to the golden set. If you jump model tiers before you read the trace, you fall into a common exam error.

### 3.8 Metrics that matter

Track task success rate. Track schema fail rate. Track tool error rate. Track refusal correctness (false refuse versus false allow). Track injection resistance rate. Track p95 latency. Track cost per success. Track human escalation rate.

---

## 4. Deep notes — Security and Safety

### 4.1 Threat model themes

**Prompt injection:** direct and indirect via documents, tools, and the web.
**Data exfiltration:** secrets in context or through outbound tools.
**Unauthorized actions:** the model makes a tool act beyond its permission.
**Abuse and fraud:** at the API edge.
**Supply chain risk:** from MCP servers, plugins, and dependencies.
**Privacy issues:** PII in logs and retention.
**Policy bugs:** overrefuse or underrefuse.

### 4.2 Defense in depth

Use edge authentication, authorization, and rate limits.
Use input filtering and size limits.
Use a non-overridable system policy.
Delimit untrusted content.
Use tool allowlists plus server-side authz.
Use Claude Code permissions and hooks where they apply.
Use output filtering and PII scrubbing.
Use audit logs and anomaly detection.

No single layer is enough. A polite system prompt is never enough.

### 4.3 Secrets management

Use a secret manager, environment injection, or cloud IAM. Never put secrets in prompts. Never put secrets in CLAUDE.md. Never commit secrets in settings. Redact secrets from logs and traces. Rotate on leak. Narrow scopes per environment and per tool.

### 4.4 PII handling

Minimize PII in prompts. Mask PII in tool adapters. Define retention. Use regional hosting when policy requires it. Access-control eval datasets. Prefer synthetic data when labels allow it.

### 4.5 Destructive action gates

**Reads:** allow under authz.
**Updates:** authz plus audit.
**Deletes, payments, and external email:** HITL or a strong policy engine inside the tool.
**Shell execution:** deny by default for autonomous agents. Sandbox when you need shell.

### 4.6 Guardrail patterns

Use independent classifiers for toxicity, PII, or jailbreak attempts. Know that they err both ways.
Put hard caps inside tools.
Use dual control for high-risk actions.
Use safe completion and clear refuse paths.
Never remove all guardrails to fix overrefusal without measurement.

### 4.7 MCP and plugin supply chain

Review code before you trust it. Pin versions. Use managed allowlists in enterprises. Use least-privilege tokens. Monitor egress. Treat project-delivered MCP configs like executable dependencies.

### 4.8 Security testing

Adversarial evals should include ignore-previous-instructions attacks. Include tool exfiltration attempts. Include social engineering. Include malicious tool descriptions.

### 4.9 Incidents

Contain: disable risky tools. Preserve redacted traces. Identify the vector. Patch delimiters and authz. Rotate secrets if exfiltration is possible. Notify stakeholders per policy. Add eval cases. Reopen with care.

---

## 5. Deep notes — Production engineering

### 5.1 SLOs

Examples: availability, success rate, p95 latency, cost ceiling, safety incident count. Error budgets decide whether you freeze features or keep shipping.

### 5.2 Capacity and rate limits

Queue work. Prioritize interactive over batch. Back off with jitter. Enforce tenant fairness so one customer cannot starve others.

### 5.3 Cost engineering ops

Use daily anomaly alerts. Use per-feature budgets. Use kill switches. Use cache hit dashboards. Store model and effort policy as versioned config. Do not store it as informal team knowledge.

### 5.4 Change management

Prompt, tool, and model changes need PR review, eval deltas, and canary. Use the same discipline as application code.

### 5.5 Residency and multi-region

Route by policy. Know host feature parity. Do not break residency rules for a shinier model SKU.

### 5.6 Degradation and DR

Use cached FAQ answers. Use human-only mode. Use delayed batch processing. Use feature flags to disable generative paths quickly.

### 5.7 On-call signals

Page on tool-error spikes. Page on schema fails. Page on cost burn. Page on safety classifier spikes. Page on step-count spikes that signal runaway loops.

---

## 6. Decision trees

### 6.1 Is it safe to automate?

If irreversible external effects exist, require HITL or a hard policy engine in the tool.
If credentials in the environment are broader than the task, narrow them before you raise autonomy.
If adversarial eval coverage is weak, keep close human supervision.
If the golden set is green and monitors are ready, you may consider higher autonomy. Otherwise do not enable autopilot.

### 6.2 Eval sufficiency

When you change prompts, tools, or models, you need at least an offline golden delta.
When you ship agent writes, you need side-effect assertions plus an adversarial pack.
Regulated domains also need human-reviewed samples and audit trails.

### 6.3 Suspected injection incident

Contain: disable risky tools. Preserve redacted traces. Identify the vector: a document, a tool payload, or a web fetch. Patch delimiters and authz. Rotate secrets if you need to. Add an eval case. Reopen with care.

---

## 7. Exam traps

1. Prompt-only security for hard risks.
2. Evals only on happy-path chat with no side effects.
3. Logging secrets for convenience.
4. Autopilot with shell access and production credentials.
5. Ignoring side effects when final text looks good.
6. One manual test treated as an eval suite.
7. Disabling guardrails to fix overrefusal without measurement.
8. Sharing eval data that still contains PII.
9. Trusting MCP servers merely because they resolve on internal DNS.
10. Skipping the small Eval domain because of its 2.6 percent weight.

---

## 8. Self-check Q&A (25)

**Q1.** What is the minimum eval before a production prompt change?
**A1.** Use a versioned golden comparison plus smoke tests. Do not use opinions.

Call **Q2.** The answer text is correct, but `delete_user`. Does this pass?
**A2.** Fail. Side effects still fail the case.

**Q3.** Where do API keys live?
**A3.** Put them in a secret manager, env, or IAM. Never put them in prompts or repos.

**Q4.** Give an example of indirect prompt injection.
**A4.** Malicious instructions in a fetched PDF or page.

**Q5.** What is the best control so that nobody drops production tables?
**A5.** Use a DB role without DROP, a tool allowlist, and denies. Do not use only a prompt line.

**Q6.** Judge scores rise but humans disagree. What do you do?
**A6.** Audit the judge and rubric. Do not ship on the judge alone.

**Q7.** Cost burn at 3am. What are the first moves?
**A7.** Use the kill switch or disable nonessential agents. Inspect runaway loops and cache collapse.

**Q8.** Why do you redact traces?
**A8.** Prompts and tool payloads often contain PII or secrets.

**Q9.** What is the shorthand for security versus safety?
**A9.** Security focuses on attackers and system misuse. Safety focuses on harmful or policy-violating outputs. They overlap.

**Q10.** How does canary differ from shadow?
**A10.** Canary affects some real users. Shadow compares without a user-visible change when you design it that way.

**Q11.** When must you freeze an eval set?
**A11.** Freeze it during comparison of two systems.

**Q12.** Should you bypass permissions on shared CI runners?
**A12.** That is a dangerous default. Avoid it.

**Q13.** PII is in golden sets. What controls do you use?
**A13.** Minimize, mask, ACL, retention, or synthesize.

**Q14.** A tool description asks you to send data to an unknown host. What is the risk?
**A14.** Malicious supply chain. Review, pin, and allowlist.

**Q15.** Overrefusal occurs after you tighten filters. How do you respond?
**A15.** Measure false refuses. Tune with evals. Do not remove filters without data.

**Q16.** Schema failures spike. How do you debug?
**A16.** Diff recent model, prompt, and tool versions. Sample traces.

**Q17.** What is an error budget for an LLM feature?
**A17.** Allowed unreliability before reliability work takes priority.

**Q18.** Why do you use tenant rate limits?
**A18.** Fairness and abuse isolation.

**Q19.** Why do you use HITL for mass email?
**A19.** High reputational and legal blast radius.

**Q20.** What are the Security plus Eval weights?
**A20.** 8.1 percent plus 2.6 percent. The weights are small but decisive.

**Q21.** When do you add a regression case?
**A21.** After every caught production failure.

**Q22.** Is temperature zero full determinism?
**A22.** No. Still pin models. Tools remain nondeterministic. You still need evals.

**Q23.** Residency blocks a US-only model. What is the path?
**A23.** Use an approved regional SKU. Redesign if features are missing.

**Q24.** Which monitors detect agent loops?
**A24.** Steps per session, duplicate tool calls, cost per session, and duration.

**Q25.** What is the first security question for a new tool?
**A25.** What authz remains if an attacker fully controls the model?

---

## 9. Checklist

- [ ] Golden set is versioned and segmented
- Test [ ] Agent side effects.
- [ ] Adversarial and injection cases exist
- [ ] Secrets are absent from prompts and logs
- [ ] PII is minimized and masked
- [ ] Destructive tools are gated
- [ ] Defense-in-depth layers are listed for your app
- [ ] Canary and rollback have been practiced
- [ ] Cost and safety alerts are wired
- [ ] Incident runbook includes revoke, disable, patch, and eval updates

---

## 10. Glossary

**Golden set:** labeled evaluation dataset.
**Rubric:** structured scoring criteria.
**LLM-as-judge:** a model that grades outputs.
**Canary:** limited real-user rollout.
**Shadow traffic:** parallel evaluation without user impact.
**Prompt injection:** untrusted instructions that manipulate model behavior.
**Exfiltration:** data leaving via model outputs or tools.
**Guardrail:** filter or policy layer around the model.
**HITL:** human-in-the-loop approval.
**Redaction:** removing sensitive data from logs and traces.
**Error budget:** allowed unreliability before you shift priority.
**Kill switch:** emergency disable for an LLM feature.
**False refuse:** blocking a benign allowed request.
**False allow:** permitting a disallowed request.

---

## 11. Extended production playbooks

### 11.1 Pre-prod review board questions

What can the tools do if someone abuses them?
What eval evidence exists?
What is the blast radius?
What is the rollback plan?
Where does PII flow?
Who is on-call?
What does degradation look like for users?

### 11.2 Adversarial pack categories

Direct jailbreaks. Indirect injection in documents. Tool-name social engineering. Policy-conflict requests such as ignore the refund cap. Obfuscation tricks. Multilingual evasion samples when they apply. Excessive agency requests such as run all shell commands.

### 11.3 Eval harness architecture

Case store. Runner that pins model, prompt, and tools. Scorers. Report publisher. CI gate. Dashboard. Build once and reuse across teams. See Chapter 05 on packaging.

### 11.4 Security control matrix samples

**API keys:** theft risk — secret manager and rotation.
**Customer PII:** leak risk — masking and retention.
**Shell tools:** code execution risk — deny or ask, plus sandbox.
**Remote MCP:** supply-chain risk — allowlist and pin.
**Logs:** sensitive store risk — redaction.
**Batch result buckets:** broad access risk — ACLs.

### 11.5 Debugging narratives

**Quality drop on Monday:** check Sunday deploy diffs. Look for alias moves, prompt edits, tool schema changes, cache misses from timestamps, or MCP updates.
**Safety filter blocks legitimate content:** use a labeled false-refuse set. Tune with care. Use domain review when you need it.
**Agent spends a large sum overnight:** missing budgets, loops, large tool payloads, or cache disabled. Use the kill switch. Add caps.

### 11.6 Compliance-oriented design notes

Expect audit logs, least privilege, residency, retention, and human oversight for high risk. Treat these as design requirements. You do not need to recite statutes. You need to select controls that match those themes.

### 11.7 Link to model selection

Evals decide whether cheaper models are acceptable. Cost alerts without quality metrics cause harmful cuts. Always pair economics with task success.

---

## 12. Additional Q&A (26-40)

**Q26.** What is a smoke eval?
**A26.** A small critical-path set on every build.

**Q27.** Why do you separate safety metrics from helpfulness?
**A27.** If you optimize one, you can damage the other.

**Q28.** Can you use raw production traces as golden data?
**A28.** Only with privacy review and labeling pipelines.

**Q29.** What container concerns apply with bash tools?
**A29.** Harden sandboxes. Deny dangerous namespaces. Use least privilege.

**Q30.** Are regex secret filters on outputs enough?
**A30.** They help, but they are incomplete. Stop secrets from entering context. Block exfil tools.

**Q31.** How does a feature flag off differ from a model rollback?
**A31.** Flags disable features. Rollbacks revert model or prompt versions.

**Q32.** Why do you hash configs in eval reports?
**A32.** Reproducibility and audit.

**Q33.** When do you use DLP on MCP egress?
**A33.** When tools can send data externally. This is often an enterprise default.

**Q34.** What is the danger if you only mock tools in unit tests?
**A34.** You miss integration authz bugs. Add staging contract tests.

**Q35.** Select THREE for write-tool hardening examples.
**A35.** Server authz, audit log, HITL or policy cap, plus idempotency.

**Q36.** Do you apply chat p95 to overnight batches?
**A36.** No. That is the wrong SLO class.

**Q37.** Disallowed advice appears in outputs. Which layers do you use?
**A37.** Policy prompt, safety systems, domain tool limits, and UX disclaimers as they apply.

**Q38.** Why do you keep both known-good and known-bad security cases?
**A38.** Track false positives and false negatives.

**Q39.** Who owns eval failures on a platform?
**A39.** Platform owns the harness. Product owns labels and rubrics.

**Q40.** What is the Chapter 04 rule?
**A40.** Measure. Enforce outside the model. Observe. Be ready to shut it off.

---

## 13. Vignette pack

**Hospital chatbot:** residency, PII minimization, clinical overrefusal and underrefusal evals. No raw EHR write tools without hard authz. HITL for sensitive scheduling actions.
**Fintech refunds:** caps inside tools, dual control over thresholds, immutable audit logs, adversarial social-engineering tests.
**University coding tutors:** shell sandbox, no network by default, academic integrity policy, cost caps per student.
**Enterprise search:** permission-aware retrieval so unauthorized documents never enter context. Injection hardening for documents. DLP on answers.

Each vignette should trigger a multi-layer answer on the exam: eval evidence plus enforcement outside the model plus operable monitoring.

---

## 14. On-call checklists

**Latency page:** inspect cache hits, downstream tool latency, model overload, and retry storms.
**Quality page:** inspect deploy diffs, eval deltas, and tool error rates.
**Safety page:** inspect classifier spikes. Consider disabling generative replies. Preserve traces.
**Cost page:** inspect top sessions by spend, loop detectors, and disable noncritical jobs.

Write these checklists before you need them. Exams reward prepared operational judgment.

---

## 15. Production engineering patterns in depth

### 15.1 Queues and priorities

Interactive user requests must not wait behind large batch backfills. Separate queues or priority lanes. Apply per-tenant fair sharing. Shed load explicitly with user-visible degradation. Do not allow silent timeout storms.

### 15.2 Idempotent consumers

Workers that execute tool side effects must be idempotent. Use idempotency keys. Use upsert semantics. Use state machines that can resume after crashes. Avoid double charging or double emailing.

### 15.3 Backpressure

When Claude or a tool host slows down, upstream admission control should reject or queue. Do not open unbounded client threads. Pair this with circuit breakers.

### 15.4 Config as code

Model pins, prompt versions, tool hashes, effort defaults, and feature flags belong in reviewed config. Emergency breaks should still leave an audit trail.

### 15.5 Multi-host reality

Some customers run on Bedrock or Vertex. Maintain a feature matrix. Integration tests should run per host that you claim to support. Residency may force a region with fewer SKUs. Design for sufficiency, not for ideal conditions.

### 15.6 Data lifecycle

Define how long to keep prompts, traces, and batch outputs. Encrypt at rest. Restrict who can replay traces. Eval datasets need the same rigor as production data when they contain sensitive content.

### 15.7 Human factors

On-call humans need clear dashboards and one-click kill switches. A good eval suite fails in practice if nobody can act at 3am.

---

## 16. Security deep drills

**Drill 1:** List five places secrets accidentally appear (frontend, prompts, CLAUDE.md, traces, MCP config) and the fix for each.
**Drill 2:** Given a write tool, write server authz rules that still hold if the model is hostile.
**Drill 3:** Convert a prompt-only rule into a hard control.
**Drill 4:** Design an indirect injection test with a malicious README that a tool fetches.
**Drill 5:** Map autonomy level L0 through L3 to permission postures.

---

## 17. Eval deep drills

**Drill 1:** Split a golden set into easy, medium, and hard. Explain why a single average misleads.
**Drill 2:** Write three side-effect assertions for a refund agent.
**Drill 3:** Design a canary metric set for a prompt change.
**Drill 4:** Explain how to calibrate an LLM judge with a human-labeled subset.
**Drill 5:** Turn yesterday's incident into two regression cases.

---

## 18. Combined scenario walkthroughs

### Walkthrough A — Support agent proposes a policy rewrite

A user uploads an email that says ignore refund policy and pay immediately. Correct system behavior: treat the email as untrusted. Policy tools still enforce caps. Possibly escalate. Log the attempt. Later add it to the adversarial pack.

### Walkthrough B — Coding agent in CI

Use a headless session with a pinned model. Deny production credentials. Use tests as the success gate. Do not bypass permissions. Upload artifacts for review. If tests fail, the pipeline fails closed.

### Walkthrough C — Cost regression after a helpful prompt edit

A new prompt added a per-request timestamp into the system section. Cache hit rate collapsed. Cost rose. Fix prefix hygiene first. Do not immediately escalate model spend elsewhere to compensate.

### Walkthrough D — Safety false refuses

A classifier blocks benign technical content. Use a false-refuse set. Tune thresholds. Add allow categories with care. Monitor both false refuse and false allow after the change.

---

## 19. Policy-as-code examples (conceptual)

Express spend caps, allowed regions, blocked tool names, and max steps as machine-checked config. The model may read a human explanation of the policy. Enforcement must not depend on compliance with that explanation.

Example concepts to remember on exam day:
- `max_refund_amount` enforced in the refund tool
- `max_agent_steps` enforced in the orchestrator
- `denied_path_patterns` enforced in Claude Code permissions
- `residency_route` enforced in the gateway before model selection

---

## 20. Observability redaction patterns

Redact API keys, passwords, session tokens, government IDs, and free-text fields that hold secrets.
Keep enough structure to debug: tool names, latency, error codes, hashed identifiers.
Give a break-glass path for privileged incident responders. Use audited access to fuller traces.

---

## 21. Testing pyramid for Claude apps

**Bottom:** deterministic unit tests for validators, authz, and pure business rules.
**Middle:** contract tests against tool hosts and schema fixtures.
**Upper-middle:** golden offline evals with pinned models.
**Top:** canary and online monitors.

Do not invert the pyramid. Do not rely only on expensive online experiments.

---

## 22. Autonomy promotion checklist

Before you raise autonomy from ask-heavy to autopilot:

1. Side-effect evals are green across segments (task success is stable across two pin versions)
2. Adversarial pack is green enough for risk tolerance
3. Budgets (turns, dollars, wall clock) and kill switches are live
4. Progress detector prevents no-op loops
5. Destructive tools sit behind human confirmation
6. On-call runbooks are rehearsed (include injection-like incidents)
7. Least-privilege credentials are verified
8. Logging redaction is verified. Audit log of tool calls is searchable
9. Rollback and feature flags are verified
10. Cost dashboard is annotated with the autonomy change
11. Record Stakeholder acceptance of residual risk.

---

## 23. Common stem translations

**Stem says users trust the chatbot too much** — add citations, uncertainty language, and escalations. Maybe reduce autonomy.
**Stem says auditor needs proof** — immutable logs, version pins, eval reports.
**Stem says model leaked an API key** — rotate, redact, remove secrets from context, review traces.
**Stem says agent looped all night** — budgets, progress detectors, alerts.
**Stem says prompt injection via search results** — delimit untrusted content, harden tool authz, add adversarial cases.
**Stem says cheaper model failed security cases** — do not ship that route for the failing segment.

---

## 24. One-page revision

**Evals:** golden plus side effects plus adversarial plus deploy gates.
**Security:** defense in depth. Secrets out of prompts. Tool authz. Injection hygiene.
**Production:** SLOs, budgets, canary, kill switch, redacted observability.
**Rule:** measure, enforce outside the model, observe, and be ready to shut it off.

---

## Appendix — Chapter to official domains

**Security and Safety 8.1 percent:** core.
**Eval, Testing, and Debugging 2.6 percent:** core.
**Applications and Integration 33.1 percent:** ops, gates, observability.
**Agents and Workflows 14.7 percent:** autonomy levels and side effects.
**Tools and MCPs 10.6 percent:** supply chain and authz.
**Model Selection and Optimization 16.8 percent:** eval-justified routing.
**Prompt and Context Engineering 11.0 percent:** policy prompts as one layer only.
**Claude Code 3.1 percent:** hooks and permissions as enforcement.

---

## 25. Extra practice prompts for self-study

Write a one-paragraph threat model for an MCP-enabled coding agent with repository access.
Write five golden cases for a refund bot including boundary amounts.
Diagram defense in depth for a public-facing RAG chatbot.
List kill-switch criteria that should auto-disable generative replies.
Explain how you would prove to an auditor that a model pin did not drift last quarter.

If you can answer those without notes, Chapter 04 is ready.

---

## 26. Extended glossary drill

Define without looking: canary, shadow traffic, false allow, false refuse, error budget, kill switch, idempotency, residency. Then define: DLP, supply chain, side-effect assertion, golden freeze, break-glass access, circuit breaker, fair sharing.

Teaching each term in one sentence is enough for exam recall.

---

## 27. Final security and eval pairing table

**Change type: prompt edit** — required eval: golden delta — security note: watch injection regressions.
**Change type: tool add** — required eval: misuse and authz cases — security note: server-side authz review.
**Change type: model upgrade** — required eval: segmented golden plus safety pack — security note: compare false allow rates.
**Change type: autonomy increase** — required eval: side-effect suite — security note: HITL policy revisit.
**Change type: MCP server add** — required eval: contract plus adversarial tool-desc cases — security note: allowlist and pin.

---

## 28. Workshop — turn weak designs into strong ones

**Weak:** We told Claude not to invent refunds.
**Strong:** Refund tool enforces caps. HITL above threshold. Retrieval for policy. Adversarial evals. Audit log.

**Weak:** We tested the agent once and it looked good.
**Strong:** Versioned golden set, side-effect asserts, CI gate, canary metrics, rollback plan.

**Weak:** Logs keep full prompts forever for learning.
**Strong:** Redaction, retention limits, ACL on the trace store, privacy review before any training use.

**Weak:** CI uses bypass permissions so the pipeline stays green.
**Strong:** Pin permissions. Deny secrets and prod networks. Fail closed on tests. No bypass on shared runners.

**Weak:** One global API key in every microservice.
**Strong:** Per-env scoped keys in a secret manager, rotation, anomaly alerts on usage.

---

## 29. Metrics guide (what to put on dashboards)

**Quality wall:** task success by segment, schema fail rate, human escalation rate.
**Safety wall:** false allow, false refuse, injection catch rate, high-risk tool invocations.
**Reliability wall:** availability, error classes, p95 latency, queue depth.
**Cost wall:** cost per successful task, cache hit rate, effort distribution, top expensive sessions.
**Autonomy wall:** steps per success, budget exhaustions, HITL acceptance rate.

Review weekly. You must have Tie alerts to pages only when action.

---

## 30. Closing study note for Chapter 04

The exam rarely asks you to recite a weight percentage. It will ask whether you ship a change with evidence. It will ask whether you enforce hard rules outside the model. It will ask whether you can operate the system when it misbehaves. Remember these three questions while you revise.

---

## 31. Cross-chapter links

**Chapter 01:** evals justify model and effort changes. Cost alerts need success metrics.
**Chapter 02:** agent budgets and verifiers also act as eval and security controls.
**Chapter 03:** permissions, hooks, MCP trust, and cache hygiene are production and security surfaces.
**Chapter 05:** package harnesses, runbooks, and CLAUDE.md templates so every team inherits these controls. Do not reinvent them badly.
**Chapter 06:** SE/SDLC foundations, code-review/CI fit-points, and self-hosted vs managed-agent residency cues. These feed security and deploy gates — see [06-agent-frameworks-and-sdlc.md](./06-agent-frameworks-and-sdlc.md).

---

## 32. Last words

If an option only adjusts wording while a hard risk remains, it is probably wrong.
If an option ships a model change without measurement, it is probably wrong.
If an option increases autonomy without least privilege and budgets, it is probably wrong.
If an option logs secrets to make debugging easier, it is definitely wrong.

---

## 33. Appendix flashcards

**Card: Golden freeze** — do not edit labels mid-comparison.
**Card: Side-effect fail** — correct text cannot excuse forbidden tools.
**Card: Kill switch** — faster than a debate during burn incidents.
**Card: Managed deny** — org policy takes priority over project convenience.
**Card: Untrusted fence** — retrieved content is evidence, not instruction.
**Card: Cost per success** — Token charts without a decision mislead.
**Card: Canary first** — fleet-wide alias flips cause Monday outages.
**Card: Redact by default** — break-glass is the exception with audit.

Study these eight cards the night before the exam. Use them with the Chapter 01 selection tree and the Chapter 03 precedence stack.

---

## 34. Ready check

You are ready for the Security and Eval slices of CCDV-F when you can explain defense in depth without notes. You can invent three adversarial cases for any new tool. You can gate a prompt change with a frozen golden set. You can describe how you would stop a runaway agent at 3am without waiting for a meeting.

Keep public Anthropic safety documentation and your own runbooks close while you revise. Product labels change. The enforce-outside-the-model principle does not.

When you are in doubt on exam stems, prefer measurement, least privilege, and fail-closed defaults. Do not prefer optimistic prompting alone.

---

## 35. Primary-study deepening — Production engineering, evals, security

Chapter 04 is primary study for Security and Safety (8.1%) and Eval/Testing/Debugging (2.6%). It is also primary study for the operational half of Applications and Integration (33.1%).

### 35.1 Evaluation as an engineering system

Why evals are better than opinions: production traffic is noisy. Offline golden sets catch regressions before customers do. Online metrics catch drift that evals missed.

| Type | Measures | Weakness |
| --- | --- | --- |
| Unit-style golden prompts | Exact or rubric score | Narrow |
| Schema/validator tests | Structural correctness | Not semantics |
| Tool/agent task success | End-to-end goals | Costly to build |
| LLM-as-judge | Nuanced quality | Bias. Needs calibration |
| Adversarial/red-team | Safety | Never complete |
| Online A/B or shadow | Real distribution | Risk if unsafe |

**Golden set craft:** cover happy path, boundary, adversarial, multilingual if relevant, and tool-failure paths. Freeze inputs. Version expected behaviors. Keep slices per product surface. Sample hard negatives from production incidents (redacted).

**Agent eval specifics:** score task success, not eloquence. Include environment state (repo fixtures, mock APIs). Bound steps. Fail on budget exceed. Assert safety invariants such as never called `delete_prod`.

**LLM-as-judge pitfalls:** the judge sees leaked answers or planner chain-of-thought and becomes biased. Uncalibrated rubrics inflate scores. Use pairwise or reference-anchored rubrics. Spot-check with humans.

**Deploy gates:** must_pass schema tests, safety denials, critical golden slice. should_pass quality rubrics within epsilon of baseline. Monitor cost, latency, cache hit, escalation rate.

**Debugging loop:** reproduce with a pinned bundle. Inspect transcript plus `stop_reason` plus tool trace. Classify prompt / tool / model / orchestration / data. Fix one layer. Add a regression case. Only then consider a model upgrade.

### 35.2 Security and Safety deep notes

**Threat themes:** prompt injection via user, retrieved docs, or tool results. Confused deputy when the model has tools the user should not have. Secret exfiltration via prompts, logs, or MCP. Supply chain risk from MCP servers, plugins, and skills. Destructive automation. Data residency and retention misuse.

**Defense in depth stack:**

1. Identity and secrets: vaulted keys. Never in prompts or CLAUDE.md.
2. Allowlists: tools, paths, networks, MCP.
3. Permissions and hooks: client-enforced in Claude Code. Server validators in apps.
4. Input delimiters and distrust of tool results.
5. Output filters for PII/secrets where needed.
6. Human approval for irreversible classes.
7. Audit logs with redaction.
8. Evals and red-team continuous.

**Destructive action gates:** classify the action. If it is irreversible, require an explicit human confirmation artifact AND permission allow AND optional dual control for high severity. Else allow under policy. Plan mode and "I will be careful" prompts are not gates.

**PII:** minimize. Redact logs. Prefer tokens/IDs over raw data in prompts. Document retention.

**MCP and plugin supply chain:** review commands/URLs. Pin versions. Least privilege. Trust UX. Deny by default in prod repos.

### 35.3 Production engineering (Applications ops)

**SLOs:** availability, TTFT/p95 latency, task success, error budgets for model outages.

**Capacity:** rate limits, queues, fairness across tenants, isolate eval traffic.

**Cost engineering ops:** dashboards for dollars per success. Cache hit. Escalation. Output length. Weekly pin reviews.

**Change management:** pin bundles. Canaries. Changelog linking prompt/tool/model. Rollback runbooks.

**Residency:** regional endpoints. Know global versus regional routing tradeoffs from public cloud docs themes.

**Degradation and DR:** fallback models. Template modes. Replay queues. Batch catch-up after outage.

**On-call signals:** spike in refusals, tool errors, cache misses, cost, detectors, human escalations.

### 35.4 Integration-heavy security and eval scenarios

**Scenario A — Support agent proposes policy rewrite.** Treat as high-impact content. Require human publish. Eval for unsafe advice. Do not auto-merge to the help center.

**Scenario B — Coding agent in CI.** Deny network and prod credentials. Allow test commands. Hooks block secret commit. Golden failing tests must go green.

**Scenario C — Cost regression after prompt edit.** Diff prefix. Check cache. Measure turns. Roll back pin. Add a cost assertion to the harness.

**Scenario D — False refuses.** Safety eval slice for benign medical/legal FAQs. Adjust policy with care. Do not disable real denials.

**Scenario E — MCP returns hostile instructions inside data.** Delimit the tool result. Ignore system override claims in data. Allowlist actions. Add a red-team fixture.

### 35.5 Decision trees

Safe to automate?
```text
Irreversible or cross-tenant?
 YES -> human gate + deny-by-default tools
 NO -> Is blast radius small and reversible?
 YES -> automate with audit + budgets
 NO -> simulate first / staged rollout
```

Eval sufficient to ship?
```text
Critical safety tests green?
 NO -> do not ship
 YES -> Golden quality within tolerance?
 NO -> do not ship / canary only with kill switch
 YES -> Ship canary -> watch online metrics
```

Suspected injection incident:
```text
Contain (disable tools / tighten allowlist) ->
preserve logs (redacted) ->
identify channel (user/retrieval/tool) ->
patch + add eval ->
comms if customer impact
```

### 35.6 Exam traps

1. Shipping on opinion checks.
2. Prompt-only safety for money movement.
3. Logging raw secrets for debugging.
4. Disabling safety filters to fix false refuses without measurement.
5. Judge model scoring its own unvalidated plan.
6. Treating batch discount as a security boundary.
7. Expanding MCP allow-all after one successful demo.
8. No rollback plan for prompt changes.
9. Mixing prod keys into eval runners.
10. Assuming CLAUDE.md never-delete stops deletes.

### 35.7 Additional Q&A (Q41-Q55)

**Q41.** What is the minimum bar to change a production system prompt?
**A41.** Version bump, golden eval pass, canary, rollback note. Not "looks better in playground."

**Q42.** How do you eval an agent that spends money?
**A42.** Mock ledger. Assert the tool is never called in the dry-run suite. Use a separate staged integration with caps.

**Q43.** Rate limit errors occur during a bulk marketing email feature. What is the architecture fix?
**A43.** Queue plus backoff. Do not raise concurrency without data. Consider Message Batches if the work is offline.

**Q44.** What is the difference between a guardrail and a permission?
**A44.** A guardrail is often a model/filter layer. A permission is hard client/server enforcement. You need both for defense in depth.

**Q45.** Why keep adversarial evals small but permanent?
**A45.** They catch safety regressions quickly. People ignore large noisy sets. Curate high-signal attacks.

**Q46.** An online metric moves before offline evals. Which do you trust?
**A46.** Investigate both. Online can have confounders. Offline may be stale. Do not without a check ship or revert without attribution.

**Q47.** Where should API keys live in Claude Code setups?
**A47.** Environment or secret manager. Not CLAUDE.md. Not committed settings. Not chat logs.

**Q48.** What is a success metric for a refactor agent?
**A48.** Tests pass plus lint clean plus scoped diff plus no forbidden path touches. Not lines changed.

**Q49.** Can caching create a security issue?
**A49.** Caching does not grant tools. Caching secrets in prefixes or logs is a data handling bug. Multi-tenant systems must not share prefixes that embed tenant secrets.

**Q50.** What is the multi-tenant prompt cache design rule?
**A50.** Shared prefix equals non-sensitive global instructions/tools only. Put tenant data in uncached turns.

**Q51.** Why isolate red-team runs?
**A51.** Aggressive prompts and tool attempts should not share prod quotas or prod credentials.

**Q52.** Hook versus permission for blocking git push?
**A52.** Either can work. Permissions deny/ask is declarative. Hooks allow custom logic/auditing. Often combine.

**Q53.** An eval with a live web tool gives unstable results. What is the fix?
**A53.** Mock the network in CI golden runs. Keep a small live smoke elsewhere.

**Q54.** What is an error budget for model quality?
**A54.** Allowed regression tolerance before rollback. For example, max X percent drop on a critical slice.

**Q55.** Name three Integration artifacts in a security review.
**A55.** Tool allowlists, MCP server inventory, pin bundle plus logging redaction policy.

### 35.8 If exam asks X, think Y (Chapter 04)

| If exam asks | Think |
| --- | --- |
| Did quality improve? | Evals plus online, not anecdotes |
| Stop dangerous action | Permissions/hooks/validators/human gate |
| Secret in transcript | Vault plus redaction plus rotate |
| Injection | Distrust inputs/tool results plus allowlists |
| Cost/latency regression | Pin diff plus metrics triad |
| Ship/no-ship | Safety gates first |
| CI agent | Least privilege plus mocks |
| Multi-tenant cache | No tenant secrets in shared prefix |

### 35.9 Glossary addendum

| Term | Meaning |
| --- | --- |
| Error budget | Allowed failure/regression before action |
| Shadow traffic | Run new pin without user-visible effect |
| Canary | Partial live rollout |
| Dead-letter | Failed item quarantine |
| Red-team set | Adversarial eval pack |
| Dual control | Two-party approval |
| Confused deputy | Privileged tool misused via injection |
| Fail closed | Deny on uncertainty |

### 35.10 Primary-study checklist (Chapter 04)

- [ ] I can design a golden set with safety plus quality slices.
- [ ] I can gate deploys with must-pass versus monitor metrics.
- [ ] I can stack defenses for destructive tools.
- [ ] I can respond to injection with contain, patch, eval.
- [ ] I can explain multi-tenant cache safety.
- [ ] I can list on-call signals for Claude apps.
- [ ] I never rely on CLAUDE.md alone for safety.

### 35.11 Applications ops patterns (extra depth)

**Queues and priorities:** interactive user traffic must not starve behind bulk eval jobs. Use separate queues or weighted fair sharing.

**Idempotent consumers:** workers that call tools must work correctly with at-least-once delivery. Carry idempotency keys through refunds, ticket updates, and email sends.

**Backpressure:** when the model API returns 429, slow producers. Do not send rapid retries that increase the overload.

**Config as code:** pin bundles in git. Promote via PR. Forbid silent playground-to-prod copy.

**Multi-host reality:** the same app may call Anthropic API and Bedrock. Abstract the client. Keep explicit host pins and feature flags.

**Data lifecycle:** define retention for prompts, tool traces, and eval fixtures. Purge or anonymize on schedule.

**Human factors:** on-call runbooks must say which pin to roll back to. They must not only say which dashboard to open.

### 35.12 Metrics one-liner (full wall layout: §29)

One-line recall version of the five metric groups in §29: task success rate, p95 TTFT, cache hit rate, dollars per successful task. Also track: tool error rate, human escalation rate, safety deny rate, canary delta versus baseline.

### 35.13 Closing — Chapter 04 as primary study

Evals tell truth. Security limits blast radius. Production engineering keeps Integration reliable under load. Even at smaller blueprint percentages, these domains convert borderline Application scenarios into clear correct answers.

### 35.14 Worked production review board questions (original)

1. What is the pin bundle (model, effort, prompt hash, tool schema, host, region)?
2. Which irreversible tools exist and what hard gate blocks them?
3. What must-pass evals gate this change?
4. What is the rollback command and who owns it?
5. What is the expected cache hit rate and how do we detect prefix drift?
6. Did the team inventory, pin, and allowlist the MCP servers?
7. Where do logs store transcripts, and how does the system redact PII?
8. What is the degrade path if the primary model or host fails?
9. How do multi-tenant prefixes avoid embedding tenant secrets?
10. What online signals page on-call within fifteen minutes of a bad canary?

### 35.15 Autonomy promotion checklist

Merged into §22 (single canonical checklist — eleven gates from evals through stakeholder sign-off).

### 35.16 Timed revision block (20 minutes)

*(Single canonical drill — merges an earlier duplicate.)*

**Minutes 1-5:** redraw the defense-in-depth stack from memory.
**Minutes 6-10:** list the eval types and when each is mandatory. Then write a must-pass versus monitor gate for a support agent.
**Minutes 11-15:** outline an incident runbook for key leak + injection. Write a multi-tenant cache prefix policy in five bullets.
**Minutes 16-20:** answer five Q41-Q55 items without looking. Then four fresh stems: underline the constraint word first (cost, residency, irreversible write, offline bulk).

### 35.17 Cross-chapter links

See §31 for the full map (Chapters 01, 02, 03, 05, and 06). Quick recall: cost regressions start as MSO pin mistakes (01). Budgets/allowlists are reliability *and* security controls (02). Streaming cancel safety and MCP trust are Integration favorites (03).
- Chapter 05: ship eval harnesses and safe defaults as accelerators so every team inherits Chapter 04 discipline.
