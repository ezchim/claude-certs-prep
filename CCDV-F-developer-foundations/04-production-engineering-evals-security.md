---
title: Production Engineering, Evals & Security
pack: CCDV-F Developer Foundations
disclaimer: Original study notes — independent and not official course content
approx_length: ~6500–9500 words (primary study)
updated: 2026-08-23
---

# Chapter 04 — Production Engineering, Evals, and Security

> **Disclaimer:** Original study notes. Grounded in public Anthropic safety and product patterns, common LLM-app eval practice, and the publicly reported CCDV-F domains Security and Safety (8.1%) and Eval/Testing/Debugging (2.6%), plus production slices of Applications and Integration (33.1%). Independent; not official course content.

**Primary maps:** Security 8.1% · Eval/Testing/Debugging 2.6% · production Integration.
**Secondary:** Agents reliability · MSO justification via evals · MCP and tool hardening.

---

## 1. Overview

Shipping Claude is an operations problem with three inseparable goals:

1. Make it correct — evals, golden sets, regression gates.
2. Make it safe — secrets, PII, injection defenses, destructive-action gates, abuse resistance.
3. Make it operable — SLOs, tracing, incident response, cost controls, change management.

Eval and Security have small weights on paper, yet they act as filters that eliminate otherwise plausible designs in multi-domain stems. Study them as decision constraints, not optional chapters.

---

## 2. Key map

Pillar: Eval — Did behavior improve for real tasks? Controls: golden sets, rubrics, side-effect asserts.
Pillar: Debug — Why did this trace fail? Controls: replay, diffs, tool logs.
Pillar: Security — What if the model is wrong or attacked? Controls: deny lists, authz, redaction, HITL.
Pillar: Safety — Does output cause harm or policy breaks? Controls: filters, classifiers, refusals.
Pillar: Production eng — Will it stay up within budget? Controls: SLOs, queues, caps, runbooks.

---

## 3. Deep notes — Evaluation

### 3.1 Why evals beat vibes

LLM changes are easy to ship and hard to notice. Evals convert opinions into measured deltas. CCDV-F narratives expect eval evidence before model, prompt, or tool changes in production.

### 3.2 Eval types

Unit-ish checks: schema validity and tool-arg shape. Fast breakage detection.
Golden sets: labeled cases with exact or fuzzy match criteria.
Rubric or LLM-as-judge: qualitative quality with calibration and human audits.
Side-effect tests: which tools were called; critical for agents.
Adversarial packs: injections and jailbreaks; security overlap.
Online methods: shadow, A/B, canary on real traffic.
Cost and latency budgets paired with quality — never alone.

### 3.3 Building a golden set

Sample real tasks after privacy review. Label expected answers or acceptable rubrics. Include edge cases and known failures. Version the set and freeze it during comparisons. Segment by difficulty and intent so average scores do not hide hard-slice regressions.

### 3.4 Agent eval specifics

Assert final answer quality, tools used and not used, step count under budget, no forbidden tools, and idempotent behavior under retry. A correct paragraph after a forbidden delete is still a fail.

### 3.5 LLM-as-judge pitfalls

Judges can share generator weaknesses. Pairwise compares show position bias. Vague rubrics create noise. Spot-check with humans on a schedule.

### 3.6 Gating deploys

CI runs unit schema checks plus a mini golden set. Staging runs the full golden set plus an adversarial pack. Canary watches online metrics and safety monitors. Fleet rollout continues monitoring with a ready rollback.

### 3.7 Debugging loop

Capture a redacted failing trace. Reproduce with pinned versions. Diff prompt, tool, model, and config. Hypothesize among context issues, tool errors, policy conflicts, and capacity problems. Fix with the smallest change. Add a regression case to the golden set. Jumping model tiers before reading the trace is a classic exam trap.

### 3.8 Metrics that matter

Task success rate, schema fail rate, tool error rate, refusal correctness (false refuse versus false allow), injection resistance rate, p95 latency, cost per success, and human escalation rate.

---

## 4. Deep notes — Security and Safety

### 4.1 Threat model themes

Prompt injection, direct and indirect via documents, tools, and the web.
Data exfiltration of secrets in context or through outbound tools.
Unauthorized actions when the model talks a tool into overreach.
Abuse and fraud at the API edge.
Supply chain risk from MCP servers, plugins, and dependencies.
Privacy issues around PII in logs and retention.
Policy bugs that overrefuse or underrefuse.

### 4.2 Defense in depth

Edge authentication, authorization, and rate limits.
Input filtering and size limits.
Non-overridable system policy.
Untrusted content delimiting.
Tool allowlists plus server-side authz.
Claude Code permissions and hooks where relevant.
Output filtering and PII scrubbing.
Audit logs and anomaly detection.

No single layer is enough, especially not a polite system prompt.

### 4.3 Secrets management

Secret manager, environment injection, or cloud IAM. Never prompts, never CLAUDE.md, never committed settings. Redact from logs and traces. Rotate on leak. Narrow scopes per environment and per tool.

### 4.4 PII handling

Minimize PII in prompts. Mask in tool adapters. Define retention. Use regional hosting when required. Access-control eval datasets. Prefer synthetic data when labels allow.

### 4.5 Destructive action gates

Reads: allow under authz.
Updates: authz plus audit.
Deletes, payments, and external email: HITL or a strong policy engine inside the tool.
Shell execution: deny by default for autonomous agents; sandbox when required.

### 4.6 Guardrail patterns

Independent classifiers for toxicity, PII, or jailbreak attempts — know they err both ways.
Hard caps inside tools.
Dual control for high-risk actions.
Safe completion and clear refuse paths.
Never remove all guardrails to fix overrefusal without measurement.

### 4.7 MCP and plugin supply chain

Review code before trust. Pin versions. Use managed allowlists in enterprises. Least-privilege tokens. Monitor egress. Treat project-delivered MCP configs like executable dependencies.

### 4.8 Security testing

Adversarial evals should include ignore-previous-instructions attacks, tool exfiltration attempts, social engineering, and malicious tool descriptions.

### 4.9 Incidents

Contain by disabling risky tools. Preserve redacted traces. Identify the vector. Patch delimiters and authz. Rotate secrets if exfiltration is possible. Notify stakeholders per policy. Add eval cases. Reopen carefully.

---

## 5. Deep notes — Production engineering

### 5.1 SLOs

Examples: availability, success rate, p95 latency, cost ceiling, safety incident count. Error budgets decide whether to freeze features or keep shipping.

### 5.2 Capacity and rate limits

Queue work. Prioritize interactive over batch. Back off with jitter. Enforce tenant fairness so one customer cannot starve others.

### 5.3 Cost engineering ops

Daily anomaly alerts. Per-feature budgets. Kill switches. Cache hit dashboards. Model and effort policy as versioned config, not tribal knowledge.

### 5.4 Change management

Prompt, tool, and model changes need PR review, eval deltas, and canary — the same discipline as application code.

### 5.5 Residency and multi-region

Route by policy. Understand host feature parity. Do not break residency rules for a shinier model SKU.

### 5.6 Degradation and DR

Cached FAQ answers. Human-only mode. Delayed batch processing. Feature flags to disable generative paths quickly.

### 5.7 On-call signals

Page on tool-error spikes, schema fails, cost burn, safety classifier spikes, and step-count explosions that signal runaway loops.

---

## 6. Decision trees

### 6.1 Is it safe to automate?

If irreversible external effects exist, require HITL or a hard policy engine in the tool.
If credentials in the environment are broader than the task, narrow them before raising autonomy.
If adversarial eval coverage is weak, keep close human supervision.
If the golden set is green and monitors are ready, consider higher autonomy; otherwise do not enable autopilot.

### 6.2 Eval sufficiency

Changing prompts, tools, or models requires at least an offline golden delta.
Shipping agent writes requires side-effect assertions plus an adversarial pack.
Regulated domains additionally need human-reviewed samples and audit trails.

### 6.3 Suspected injection incident

Contain by disabling risky tools. Preserve redacted traces. Identify whether the vector was a document, tool payload, or web fetch. Patch delimiters and authz. Rotate secrets if needed. Add an eval case. Reopen carefully.

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

Q1. Minimum eval before a prod prompt tweak? Versioned golden comparison plus smoke — not vibes.
Q2. Correct answer text but delete_user was called — pass? Fail on side effects.
Q3. Where do API keys live? Secret manager, env, or IAM — never prompts or repos.
Q4. Indirect prompt injection example? Malicious instructions in a fetched PDF or page.
Q5. Best control so production tables are never dropped? DB role without DROP, tool allowlist, and denies — not only a prompt line.
Q6. Judge scores rise but humans disagree — action? Audit the judge and rubric; do not ship on judge alone.
Q7. Cost burn at 3am — first moves? Kill switch or disable nonessential agents; inspect runaway loops and cache collapse.
Q8. Why redact traces? Prompts and tool payloads often contain PII or secrets.
Q9. Security versus safety shorthand? Security focuses on attackers and system misuse; safety focuses on harmful or policy-violating outputs; they overlap.
Q10. Canary versus shadow? Canary affects some real users; shadow compares without user-visible change when designed that way.
Q11. When must an eval set be frozen? During comparison of two systems.
Q12. Bypass permissions on shared CI runners? Dangerous default — avoid.
Q13. PII in golden sets — controls? Minimize, mask, ACL, retention, or synthesize.
Q14. Tool description asks to send data to an unknown host — risk? Malicious supply chain; review, pin, allowlist.
Q15. Overrefusal after tightening filters — response? Measure false refuses; tune with evals; do not remove filters blindly.
Q16. Schema failures spike — debug how? Diff recent model, prompt, and tool versions; sample traces.
Q17. What is an error budget for an LLM feature? Allowed unreliability before reliability work takes priority.
Q18. Why tenant rate limits? Fairness and abuse isolation.
Q19. Why HITL for mass email? High reputational and legal blast radius.
Q20. Security plus Eval weights? 8.1 percent plus 2.6 percent — small but decisive.
Q21. When to add a regression case? After every caught production failure.
Q22. Is temperature zero full determinism? No — still pin models; tools remain nondeterministic; evals still required.
Q23. Residency blocks a US-only model — path? Use approved regional SKU; redesign if features are missing.
Q24. Monitors for agent loops? Steps per session, duplicate tool calls, cost per session, duration.
Q25. First security question for a new tool? What authz remains if the model is fully compromised?

---

## 9. Checklist

- [ ] Golden set is versioned and segmented
- [ ] Agent side effects are tested
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

Golden set: labeled evaluation dataset.
Rubric: structured scoring criteria.
LLM-as-judge: a model that grades outputs.
Canary: limited real-user rollout.
Shadow traffic: parallel evaluation without user impact.
Prompt injection: untrusted instructions that manipulate model behavior.
Exfiltration: data leaving via model outputs or tools.
Guardrail: filter or policy layer around the model.
HITL: human-in-the-loop approval.
Redaction: removing sensitive data from logs and traces.
Error budget: allowed unreliability before prioritization shifts.
Kill switch: emergency disable for an LLM feature.
False refuse: blocking a benign allowed request.
False allow: permitting a disallowed request.

---

## 11. Extended production playbooks

### 11.1 Pre-prod review board questions

What can the tools do if abused?
What eval evidence exists?
What is the blast radius?
What is the rollback plan?
Where does PII flow?
Who is on-call?
What does degradation look like for users?

### 11.2 Adversarial pack categories

Direct jailbreaks. Indirect injection in documents. Tool-name social engineering. Policy-conflict requests such as ignore the refund cap. Obfuscation tricks. Multilingual evasion samples when relevant. Excessive agency requests such as run all shell commands.

### 11.3 Eval harness architecture

Case store. Runner that pins model, prompt, and tools. Scorers. Report publisher. CI gate. Dashboard. Build once and reuse across teams — see Chapter 05 on packaging.

### 11.4 Security control matrix samples

API keys: theft risk — secret manager and rotation.
Customer PII: leak risk — masking and retention.
Shell tools: code execution risk — deny or ask, plus sandbox.
Remote MCP: supply-chain risk — allowlist and pin.
Logs: sensitive store risk — redaction.
Batch result buckets: broad access risk — ACLs.

### 11.5 Debugging narratives

Quality dropped Monday: check Sunday deploy diffs for alias moves, prompt edits, tool schema changes, cache misses from timestamps, or MCP updates.
Safety filter blocks legitimate content: use a labeled false-refuse set; tune carefully with domain review when needed.
Agent spent a large sum overnight: missing budgets, loops, huge tool payloads, or cache disabled; use kill switch; add caps.

### 11.6 Compliance-oriented design notes

Expect audit logs, least privilege, residency, retention, and human oversight for high risk as design requirements. You do not need to recite statutes; you need to choose controls that match those themes.

### 11.7 Link to model selection

Evals decide whether cheaper models are acceptable. Cost alerts without quality metrics cause harmful cuts. Always pair economics with task success.

---

## 12. Additional Q&A (26-40)

Q26. What is a smoke eval? A tiny critical-path set on every build.
Q27. Why separate safety metrics from helpfulness? Optimizing one can tank the other.
Q28. Raw prod traces as golden data? Only with privacy review and labeling pipelines.
Q29. Container concerns with bash tools? Harden sandboxes; deny dangerous namespaces; least privilege.
Q30. Regex secret filters on outputs — enough? Helpful but incomplete; prevent secrets entering context and block exfil tools.
Q31. Feature flag off versus model rollback? Flags disable features; rollbacks revert model or prompt versions.
Q32. Why hash configs in eval reports? Reproducibility and audit.
Q33. When DLP on MCP egress? When tools can send data externally; often an enterprise default.
Q34. Danger of only mocking tools in unit tests? Missing integration authz bugs — add staging contract tests.
Q35. Select THREE for write-tool hardening examples: server authz, audit log, HITL or policy cap, plus idempotency.
Q36. Chat p95 applied to overnight batches? Wrong SLO class.
Q37. Disallowed advice in outputs — layers? Policy prompt, safety systems, domain tool limits, and UX disclaimers as appropriate.
Q38. Why both known-good and known-bad security cases? Track false positives and false negatives.
Q39. Who owns eval failures on a platform? Platform owns the harness; product owns labels and rubrics.
Q40. Chapter 04 mantra? Measure, enforce outside the model, observe, and be ready to shut it off.

---

## 13. Vignette pack

Hospital chatbot: residency, PII minimization, clinical overrefusal and underrefusal evals, no raw EHR write tools without hard authz, HITL for sensitive scheduling actions.
Fintech refunds: caps inside tools, dual control over thresholds, immutable audit logs, adversarial social-engineering tests.
University coding tutors: shell sandbox, no network by default, academic integrity policy, cost caps per student.
Enterprise search: permission-aware retrieval so unauthorized documents never enter context, injection hardening for documents, DLP on answers.

Each vignette should trigger a multi-layer answer on the exam: eval evidence plus enforcement outside the model plus operable monitoring.

---

## 14. On-call checklists

Latency page: inspect cache hits, downstream tool latency, model overload, and retry storms.
Quality page: inspect deploy diffs, eval deltas, and tool error rates.
Safety page: inspect classifier spikes, consider disabling generative replies, preserve traces.
Cost page: inspect top sessions by spend, loop detectors, and disable noncritical jobs.

Write these checklists before you need them. Exams reward prepared operational judgment.

---

## 15. Production engineering patterns in depth

### 15.1 Queues and priorities

Interactive user requests should not wait behind giant batch backfills. Separate queues or priority lanes. Apply per-tenant fair sharing. Shed load explicitly with user-visible degradation rather than silent timeout storms.

### 15.2 Idempotent consumers

Workers that execute tool side effects must be idempotent. Use idempotency keys, upsert semantics, and state machines that can resume after crashes without double charging or double emailing.

### 15.3 Backpressure

When Claude or a tool host slows down, upstream admission control should reject or queue rather than opening unbounded client threads. Pair with circuit breakers.

### 15.4 Config as code

Model pins, prompt versions, tool hashes, effort defaults, and feature flags belong in reviewed config. Emergency breaks should still leave an audit trail.

### 15.5 Multi-host reality

Some customers live on Bedrock or Vertex. Maintain a feature matrix. Integration tests should run per host you claim to support. Residency may force a region with fewer SKUs — design for sufficiency, not fantasy.

### 15.6 Data lifecycle

Define how long prompts, traces, and batch outputs live. Encrypt at rest. Restrict who can replay traces. Eval datasets need the same rigor as production data when they contain sensitive content.

### 15.7 Human factors

On-call humans need clear dashboards and one-click kill switches. A brilliant eval suite fails in practice if nobody can act at 3am.

---

## 16. Security deep drills

Drill 1: List five places secrets accidentally appear (frontend, prompts, CLAUDE.md, traces, MCP config) and the fix for each.
Drill 2: Given a write tool, write server authz rules that still hold if the model is hostile.
Drill 3: Convert a prompt-only rule into a hard control.
Drill 4: Design an indirect injection test using a malicious README fetched by a tool.
Drill 5: Map autonomy level L0 through L3 to permission postures.

---

## 17. Eval deep drills

Drill 1: Split a golden set into easy, medium, and hard; explain why a single average misleads.
Drill 2: Write three side-effect assertions for a refund agent.
Drill 3: Design a canary metric set for a prompt change.
Drill 4: Explain how to calibrate an LLM judge with a human-labeled subset.
Drill 5: Turn yesterday's incident into two regression cases.

---

## 18. Combined scenario walkthroughs

### Walkthrough A — Support agent proposes a policy rewrite

User uploads an email saying ignore refund policy and pay immediately. Correct system behavior: treat email as untrusted; policy tools still enforce caps; possibly escalate; log the attempt; later add to adversarial pack.

### Walkthrough B — Coding agent in CI

Headless session with pinned model, denied production credentials, tests as success gate, no bypass permissions, artifacts uploaded for review. If tests fail, pipeline fails closed.

### Walkthrough C — Cost regression after a helpful prompt edit

New prompt added a per-request timestamp into the system section. Cache hit rate collapsed. Cost rose. Fix prefix hygiene first; do not immediately escalate model spend elsewhere to compensate.

### Walkthrough D — Safety false refuses

Classifier blocks benign technical content. Use false-refuse set; tune thresholds; add allow categories carefully; monitor both false refuse and false allow after change.

---

## 19. Policy-as-code examples (conceptual)

Express spend caps, allowed regions, blocked tool names, and max steps as machine-checked config. The model may read a human explanation of the policy, but enforcement must not depend on compliance with that explanation.

Example concepts to remember on exam day:
- max_refund_amount enforced in the refund tool
- max_agent_steps enforced in the orchestrator
- denied_path_patterns enforced in Claude Code permissions
- residency_route enforced in the gateway before model selection

---

## 20. Observability redaction patterns

Redact API keys, passwords, session tokens, government IDs, and free-text fields known to hold secrets.
Keep enough structure to debug: tool names, latency, error codes, hashed identifiers.
Provide a break-glass path for privileged incident responders with audited access to fuller traces.

---

## 21. Testing pyramid for Claude apps

Bottom: deterministic unit tests for validators, authz, and pure business rules.
Middle: contract tests against tool hosts and schema fixtures.
Upper-middle: golden offline evals with pinned models.
Top: canary and online monitors.

Do not invert the pyramid by relying only on expensive online experiments.

---

## 22. Autonomy promotion checklist

Before raising autonomy from ask-heavy to autopilot:
1. Side-effect evals green across segments (task success stable across two pin versions)
2. Adversarial pack green enough for risk tolerance
3. Budgets (turns, dollars, wall clock) and kill switches live
4. Progress detector prevents no-op loops
5. Destructive tools behind human confirmation
6. On-call runbooks rehearsed (incl. injection-like incidents)
7. Least-privilege credentials verified
8. Logging redaction verified; audit log of tool calls searchable
9. Rollback and feature flags verified
10. Cost dashboard annotated with the autonomy change
11. Stakeholder acceptance of residual risk recorded

---

## 23. Common stem translations

Stem says users trust the chatbot too much — add citations, uncertainty language, and escalations; maybe reduce autonomy.
Stem says auditor needs proof — immutable logs, version pins, eval reports.
Stem says model leaked an API key — rotate, redact, remove secrets from context, review traces.
Stem says agent looped all night — budgets, progress detectors, alerts.
Stem says prompt injection via search results — delimit untrusted content, harden tool authz, add adversarial cases.
Stem says cheaper model failed security cases — do not ship that route for the failing segment.

---

## 24. One-page revision

Evals: golden plus side effects plus adversarial plus deploy gates.
Security: defense in depth; secrets out of prompts; tool authz; injection hygiene.
Production: SLOs, budgets, canary, kill switch, redacted observability.
Mantra: measure, enforce outside the model, observe, and be ready to shut it off.

---

## Appendix — Chapter to official domains

Security and Safety 8.1 percent: core.
Eval, Testing, and Debugging 2.6 percent: core.
Applications and Integration 33.1 percent: ops, gates, observability.
Agents and Workflows 14.7 percent: autonomy levels and side effects.
Tools and MCPs 10.6 percent: supply chain and authz.
Model Selection and Optimization 16.8 percent: eval-justified routing.
Prompt and Context Engineering 11.0 percent: policy prompts as one layer only.
Claude Code 3.1 percent: hooks and permissions as enforcement.

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

Define without looking: canary, shadow traffic, false allow, false refuse, error budget, kill switch, idempotency, residency, DLP, supply chain, side-effect assertion, golden freeze, break-glass access, circuit breaker, fair sharing.

Teaching each term in one sentence is enough for exam recall.

---

## 27. Final security and eval pairing table

Change type: prompt edit — required eval: golden delta — security note: watch injection regressions.
Change type: tool add — required eval: misuse and authz cases — security note: server-side authz review.
Change type: model upgrade — required eval: segmented golden plus safety pack — security note: compare false allow rates.
Change type: autonomy increase — required eval: side-effect suite — security note: HITL policy revisit.
Change type: MCP server add — required eval: contract plus adversarial tool-desc cases — security note: allowlist and pin.


---

## 28. Workshop — turn weak designs into strong ones

Weak: We told Claude not to invent refunds.
Strong: Refund tool enforces caps; HITL above threshold; retrieval for policy; adversarial evals; audit log.

Weak: We tested the agent once and it looked good.
Strong: Versioned golden set, side-effect asserts, CI gate, canary metrics, rollback plan.

Weak: Logs keep full prompts forever for learning.
Strong: Redaction, retention limits, ACL on trace store, privacy review before any training use.

Weak: CI uses bypass permissions so the pipeline stays green.
Strong: Pin permissions, deny secrets and prod networks, fail closed on tests, no bypass on shared runners.

Weak: One global API key in every microservice.
Strong: Per-env scoped keys in a secret manager, rotation, anomaly alerts on usage.

---

## 29. Metrics cookbook (what to put on the wall)

Quality wall: task success by segment, schema fail rate, human escalation rate.
Safety wall: false allow, false refuse, injection catch rate, high-risk tool invocations.
Reliability wall: availability, error classes, p95 latency, queue depth.
Cost wall: cost per successful task, cache hit rate, effort distribution, top expensive sessions.
Autonomy wall: steps per success, budget exhaustions, HITL acceptance rate.

Review weekly. Tie alerts to pages only when action is required.

---

## 30. Closing study note for Chapter 04

The exam will rarely ask you to recite a weight percentage. It will ask whether you ship a change with evidence, whether you enforce hard rules outside the model, and whether you can operate the system when it misbehaves. Keep those three questions taped above your desk while you revise.


---

## 31. Cross-chapter links

Chapter 01: evals justify model and effort changes; cost alerts need success metrics.
Chapter 02: agent budgets and verifiers are eval and security controls in disguise.
Chapter 03: permissions, hooks, MCP trust, and cache hygiene are production and security surfaces.
Chapter 05: package harnesses, runbooks, and CLAUDE.md templates so every team inherits these controls instead of reinventing them badly.
Chapter 06: SE/SDLC foundations, code-review/CI fit-points, and self-hosted vs managed-agent residency cues that feed security and deploy gates — see [06-agent-frameworks-and-sdlc.md](./06-agent-frameworks-and-sdlc.md).

---

## 32. Last words

If an option only adjusts wording while a hard risk remains, it is probably wrong.
If an option ships a model change without measurement, it is probably wrong.
If an option increases autonomy without least privilege and budgets, it is probably wrong.
If an option logs secrets to make debugging easier, it is definitely wrong.


---

## 33. Appendix flashcards

Card: Golden freeze — do not edit labels mid-comparison.
Card: Side-effect fail — correct text cannot excuse forbidden tools.
Card: Kill switch — faster than a debate during burn incidents.
Card: Managed deny — org policy beats project convenience.
Card: Untrusted fence — retrieved content is evidence, not instruction.
Card: Cost per success — vanity token charts mislead.
Card: Canary first — fleet-wide alias flips cause Monday outages.
Card: Redact by default — break-glass is the exception with audit.

Study these eight cards the night before the exam together with the Chapter 01 selection tree and the Chapter 03 precedence stack.


---

## 34. Ready check

You are ready for the Security and Eval slices of CCDV-F when you can explain defense in depth without notes, invent three adversarial cases for any new tool, gate a prompt change with a frozen golden set, and describe how you would stop a runaway agent at 3am without waiting for a meeting.


Keep public Anthropic safety documentation and your own runbooks close while revising; product labels change, but the enforce-outside-the-model principle does not.

When in doubt on exam stems, prefer measurement, least privilege, and fail-closed defaults over optimistic prompting alone.
---

## 35. Primary-study deepening — Production engineering, evals, security

Chapter 04 is primary study for Security and Safety (8.1%) and Eval/Testing/Debugging (2.6%), plus the operational half of Applications and Integration (33.1%).


### 35.1 Evaluation as an engineering system

Why evals beat vibes: production traffic is noisy; offline golden sets catch regressions before customers do; online metrics catch drift evals missed.

| Type | Measures | Weakness |
| --- | --- | --- |
| Unit-style golden prompts | Exact or rubric score | Narrow |
| Schema/validator tests | Structural correctness | Not semantics |
| Tool/agent task success | End-to-end goals | Costly to build |
| LLM-as-judge | Nuanced quality | Bias; needs calibration |
| Adversarial/red-team | Safety | Never complete |
| Online A/B or shadow | Real distribution | Risk if unsafe |

Golden set craft: cover happy path, boundary, adversarial, multilingual if relevant, and tool-failure paths. Freeze inputs; version expected behaviors. Keep slices per product surface. Sample hard negatives from production incidents (redacted).

Agent eval specifics: score task success, not eloquence. Include environment state (repo fixtures, mock APIs). Bound steps; fail on budget exceed. Assert safety invariants such as never called delete_prod.

LLM-as-judge pitfalls: judge sees leaked answers or planner chain-of-thought and becomes biased. Uncalibrated rubrics inflate scores. Use pairwise or reference-anchored rubrics; spot-check with humans.

Deploy gates: must_pass schema tests, safety denials, critical golden slice; should_pass quality rubrics within epsilon of baseline; monitor cost, latency, cache hit, escalation rate.

Debugging loop: reproduce with pinned bundle; inspect transcript plus stop_reason plus tool trace; classify prompt / tool / model / orchestration / data; fix one layer; add regression case; only then consider model upgrade.

### 35.2 Security and Safety deep notes

Threat themes: prompt injection via user, retrieved docs, or tool results; confused deputy when the model has tools the user should not; secret exfiltration via prompts, logs, or MCP; supply chain risk from MCP servers, plugins, and skills; destructive automation; data residency and retention misuse.

Defense in depth stack:
1. Identity and secrets: vaulted keys; never in prompts or CLAUDE.md.
2. Allowlists: tools, paths, networks, MCP.
3. Permissions and hooks: client-enforced in Claude Code; server validators in apps.
4. Input delimiters and distrust of tool results.
5. Output filters for PII/secrets where needed.
6. Human approval for irreversible classes.
7. Audit logs with redaction.
8. Evals and red-team continuous.

Destructive action gates: classify action; if irreversible require explicit human confirmation artifact AND permission allow AND optional dual control for high severity; else allow under policy. Plan mode and "I will be careful" prompts are not gates.

PII: minimize; redact logs; prefer tokens/IDs over raw data in prompts; document retention.

MCP and plugin supply chain: review commands/URLs; pin versions; least privilege; trust UX; deny by default in prod repos.

### 35.3 Production engineering (Applications ops)

SLOs: availability, TTFT/p95 latency, task success, error budgets for model outages.

Capacity: rate limits, queues, fairness across tenants, isolate eval traffic.

Cost engineering ops: dashboards for dollars per success; cache hit; escalation; output length; weekly pin reviews.

Change management: pin bundles; canaries; changelog linking prompt/tool/model; rollback runbooks.

Residency: regional endpoints; know global versus regional routing tradeoffs from public cloud docs themes.

Degradation and DR: fallback models; template modes; replay queues; batch catch-up after outage.

On-call signals: spike in refusals, tool errors, cache misses, cost, detectors, human escalations.

### 35.4 Integration-heavy security and eval scenarios

Scenario A — Support agent proposes policy rewrite. Treat as high-impact content; require human publish; eval for unsafe advice; do not auto-merge to help center.

Scenario B — Coding agent in CI. Deny network and prod credentials; allow test commands; hooks block secret commit; golden failing tests must go green.

Scenario C — Cost regression after prompt edit. Diff prefix; check cache; measure turns; roll back pin; add cost assertion to harness.

Scenario D — False refuses. Safety eval slice for benign medical/legal FAQs; adjust policy carefully without disabling real denials.

Scenario E — MCP returns hostile instructions inside data. Tool result delimited; ignore system override claims in data; allowlist actions; add red-team fixture.

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

1. Shipping on vibe checks.
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
**A41.** Version bump, golden eval pass, canary, rollback note — not looks better in playground.

**Q42.** How do you eval an agent that spends money? 
**A42.** Mock ledger; assert tool never called in dry-run suite; separate staged integration with caps.

**Q43.** Rate limit errors during a marketing email blast feature — architecture fix? 
**A43.** Queue plus backoff; do not raise concurrency blindly; consider Message Batches if offline.

**Q44.** Difference between guardrail and permission? 
**A44.** Guardrail often model/filter layer; permission is hard client/server enforcement. Need both for defense in depth.

**Q45.** Why keep adversarial evals small but permanent? 
**A45.** They catch safety regressions quickly; large noisy sets get ignored — curate high-signal attacks.

**Q46.** Online metric moves before offline evals — trust which? 
**A46.** Investigate both; online can be confounders; offline may be stale. Do not blindly ship or revert without attribution.

**Q47.** Where should API keys live in Claude Code setups? 
**A47.** Environment or secret manager — not CLAUDE.md, not committed settings, not chat logs.

**Q48.** What is a success metric for a refactor agent? 
**A48.** Tests pass plus lint clean plus scoped diff plus no forbidden path touches — not lines changed.

**Q49.** Can caching create a security issue? 
**A49.** Caching does not grant tools, but caching secrets in prefixes or logs is a data handling bug; multi-tenant systems must not share prefixes that embed tenant secrets.

**Q50.** Multi-tenant prompt cache design rule? 
**A50.** Shared prefix equals non-sensitive global instructions/tools only; tenant data in uncached turns.

**Q51.** Why isolate red-team runs? 
**A51.** Aggressive prompts and tool attempts should not share prod quotas or prod credentials.

**Q52.** Hook versus permission for blocking git push? 
**A52.** Either can work; permissions deny/ask is declarative; hooks allow custom logic/auditing. Often combine.

**Q53.** Eval flake from live web tool — fix? 
**A53.** Mock network in CI golden runs; keep a small live smoke elsewhere.

**Q54.** What is an error budget for model quality? 
**A54.** Allowed regression tolerance before rollback — for example max X percent drop on critical slice.

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

Queues and priorities: interactive user traffic should not starve behind bulk eval jobs. Use separate queues or weighted fair sharing.

Idempotent consumers: toolful workers must survive at-least-once delivery. Carry idempotency keys through refunds, ticket updates, and email sends.

Backpressure: when the model API returns 429, slow producers; do not spin hot retries that amplify the storm.

Config as code: pin bundles in git; promote via PR; forbid silent playground-to-prod copy.

Multi-host reality: the same app may call Anthropic API and Bedrock; abstract the client but keep explicit host pins and feature flags.

Data lifecycle: define retention for prompts, tool traces, and eval fixtures; purge or anonymize on schedule.

Human factors: on-call runbooks must say which pin to roll back to, not only which dashboard to open.

### 35.12 Metrics one-liner (full wall layout: §29)

One-line recall version of §29’s five walls: task success rate, p95 TTFT, cache hit rate, dollars per successful task, tool error rate, human escalation rate, safety deny rate, canary delta versus baseline.

### 35.13 Closing — Chapter 04 as primary study

Evals tell truth; security limits blast radius; production engineering keeps Integration reliable under load. Even at smaller blueprint percentages, these domains convert borderline Application scenarios into clear correct answers.

### 35.14 Worked production review board questions (original)

1. What is the pin bundle (model, effort, prompt hash, tool schema, host, region)?
2. Which irreversible tools exist and what hard gate blocks them?
3. What must-pass evals gate this change?
4. What is the rollback command and who owns it?
5. What is the expected cache hit rate and how do we detect prefix drift?
6. Are MCP servers inventoried, version-pinned, and allowlisted?
7. Where do logs store transcripts and how is PII redacted?
8. What is the degrade path if the primary model or host fails?
9. How do multi-tenant prefixes avoid embedding tenant secrets?
10. What online signals page on-call within fifteen minutes of a bad canary?

### 35.15 Autonomy promotion checklist

Merged into §22 (single canonical checklist — eleven gates from evals through stakeholder sign-off).

### 35.16 Timed revision block (20 minutes)

*(Single canonical drill — merges an earlier duplicate.)*

Minutes 1-5: redraw the defense-in-depth stack from memory. 
Minutes 6-10: list the eval types and when each is mandatory, then write a must-pass versus monitor gate for a support agent. 
Minutes 11-15: outline an incident runbook for key leak + injection, and a multi-tenant cache prefix policy in five bullets. 
Minutes 16-20: answer five Q41-Q55 items without looking, then four fresh stems by underlining the constraint word first (cost, residency, irreversible write, offline bulk).

### 35.17 Cross-chapter links

See §31 for the full map (Chapters 01, 02, 03, 05, and 06). Quick recall: cost regressions start as MSO pin mistakes (01); budgets/allowlists are reliability *and* security controls (02); streaming cancel safety and MCP trust are Integration favorites (03).
- Chapter 05: ship eval harnesses and safe defaults as accelerators so every team inherits Chapter 04 discipline.
