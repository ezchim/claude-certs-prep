---
title: Chapter 05 — Accelerators and IP Contribution — Simplified Technical English
pack: CCDV-F Developer Foundations
disclaimer: Original study notes — independent and not official course content
approx_length: ~5000–7500 words (primary study)
updated: 2026-08-23
---

# Chapter 05 — Accelerators and IP Contribution

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, MCP, CLAUDE.md, eval, skill) are exceptions and stay as written. These notes are original study notes. They do **not** copy official course content. They give original exam guidance on reusable agents, skills packaging, CLAUDE.md templates, eval harness reuse, and handoff documentation. The guidance uses public Claude Code, MCP, API, and engineering practice.

**Maps primarily to:** Applications and Integration (reusable configs and templates) · Agents and Workflows (packaged agents) · Eval (harness reuse).
**Secondary:** Claude Code (skills, templates) · Tools/MCP (modular servers) · Security (safe defaults in packages).

---

## 1. Overview

Accelerators are reusable components. They help the next engineer deliver a Claude solution faster. The next engineer does not copy informal chat notes.

On exams and in partner delivery, strong answers package these items:

1. A default architecture with pinned models and safe permissions.
2. Prompt and CLAUDE.md templates with clear extension points.
3. Skills or playbooks for repeated procedures.
4. MCP or tool modules with allowlists and auth patterns.
5. An eval harness and golden starter set.
6. Handoff docs: how to run, how to change, how to rollback.

IP contribution means you turn a single client success into modular assets. Other teams can adopt the assets under your organization rules. This file teaches the engineering substance. Follow your firm legal rules and partner-program rules for actual submission.

---

## 2. Key map

Asset type: **Agent template** — What it encodes: loop, budgets, tools, verifiers.
Asset type: **Skill** — What it encodes: on-demand procedure for Claude Code.
Asset type: **CLAUDE.md template** — What it encodes: lean always-on repo guidance.
Asset type: **Settings pack** — What it encodes: permissions, hooks, MCP gates.
Asset type: **MCP module** — What it encodes: domain tools with auth and errors.
Asset type: **Eval harness** — What it encodes: runner, scorers, CI gate.
Asset type: **Runbook** — What it encodes: operate, incident, upgrade steps.
Asset type: **Reference app** — What it encodes: end-to-end thin vertical slice.

Use this map as a flashcard. Name the asset type. Then name what the asset encodes. Do not mix a skill with a CLAUDE.md template. Do not mix a runbook with an eval harness.

---

## 3. Deep notes — Packaging reusable agents

### 3.1 What makes an agent reusable

An agent is reusable when you give it these properties:

- Inputs and outputs are clear.
- You document the tools.
- You pin the defaults. Override hooks exist.
- The skeleton includes budgets and stop conditions.
- The agent includes eval cases.
- Permissions are safe by default.
- The package holds **no** secrets.
- You version the releases.

If one of these items is missing, the next team cannot adopt the agent with safety. They copy a one-off build. They do not get an accelerator.

### 3.2 Configuration surface

You separate these four surfaces:

- **Policy** (safety rules, denies)
- **Product prompts** (tone, task framing)
- **Environment** (endpoints, model pins)
- **Secrets** (injected, never packaged)

Consumers override environment and product prompts more often than policy. Keep policy stable. Keep secrets out of the package. Inject secrets at run time.

### 3.3 Reference agent skeleton (conceptual)

A reusable agent skeleton includes these parts:

- Intake message schema.
- Planner optional step.
- Tool registry with allowlist.
- Loop with max steps and wall clock.
- Verifier hooks (schema, tests, policy).
- Escalation path.
- Telemetry hooks.
- Eval entrypoint.

Document each part. Give adopters a clear place to extend the skeleton. Do not hide stop conditions in code comments only.

### 3.4 Anti-patterns in agent packaging

Do not do these things when you package an agent:

- Hardcoded customer names.
- Secrets in the repo.
- Unbounded tools.
- No version.
- No evals.
- Docs that say only "run main".
- You enable bypass permissions to make the demo faster.

These anti-patterns fail exams and fail handoff. A demo that uses bypass permissions is not a safe default.

---

## 4. Deep notes — Skills packaging (Claude Code)

### 4.1 Skills versus CLAUDE.md versus hooks

**CLAUDE.md:** short always-on context.
**Skill:** a procedure that Claude Code invokes when the procedure is relevant. Skills keep always-on context lean.
**Hook or permission:** a gate that you must enforce.

Package skills for these procedures:

- Release checklist
- Add HTTP endpoint
- Write migration safely
- Triage test that fails intermittently
- Prepare PR description

Use CLAUDE.md for rules that must stay in context on every turn. Use a skill for a procedure that you need only sometimes. Use a hook when the rule must fire even if the model ignores text.

### 4.2 Skill quality bar

A skill meets the quality bar when it does these things:

- States preconditions.
- Lists steps.
- Names tools involved.
- Declares stop conditions.
- Links to tests.
- Notes dangerous variants.
- Includes examples of good and bad outcomes.

If the skill does not name stop conditions, the agent can continue past a safe point. If the skill does not link to tests, adopters cannot prove the procedure still works.

### 4.3 Versioning skills

Use a semantic version or a dated changelog. Note breaking changes. Keep skills in the repo so PRs review them. Do not keep personal-only skills as team accelerators. A personal skill does not travel. A repo skill does travel.

### 4.4 Plugin-style packaging

Where teams use plugins or shared bundles, include these items:

- Settings recommendations
- Skills
- Hook scripts
- Example CLAUDE.md
- A README that explains trust boundaries

Consumers still review MCP and permissions before they enable the bundle. The bundle does not replace that review. Trust boundaries stay explicit.

---

## 5. Deep notes — CLAUDE.md templates

### 5.1 Lean template sections

A lean CLAUDE.md template uses these sections:

1. Repo purpose in three lines
2. Directory map
3. Canonical commands
4. Conventions (errors, testing, APIs)
5. Dangerous zones
6. Pointers to deep docs
7. Explicit non-goals (do not invent infra)

Keep the file short on purpose. Long files compete with task tokens. Put deep detail in linked docs.

### 5.2 Template variants

You need variants. One template does not fit all repos.

- Monorepo root template plus nested package template.
- Library template versus service template.
- Data science template with notebook and eval notes.
- Infrastructure template with plan-mode emphasis and deny lists for prod apply.

Match the variant to the repo type. A service template is not a data science template. An infrastructure template must emphasize plan mode and deny lists.

### 5.3 What not to put in templates

Do not put these items in templates:

- Secrets.
- Customer PII.
- Volatile timestamps.
- Entire architecture decision records.
- Contradictory rules from multiple authors without ownership.

Secrets and PII create incidents. Timestamps invalidate the cache and confuse agents. Full ADRs make the file too long. Contradictory rules cause teams to ignore the template.

### 5.4 Maintenance

Assign an owner. Review the template every quarter. Tie updates to broken evals or incidents. Prefer links over paste. An owner who does not review the file lets the template drift.

---

## 6. Deep notes — Eval harness reuse

### 6.1 Why harnesses are IP

The harness encodes how your organization measures Claude quality. Reuse of the harness prevents every team from inventing inconsistent scoring methods. A shared harness is high-value IP. It makes comparison honest.

### 6.2 Harness components to package

Package these components:

- Case schema.
- Fixture loaders.
- Model and prompt pin interface.
- Tool simulators.
- Scorers (exact, fuzzy, rubric, side-effect).
- Report formats.
- CI examples.
- Sample golden set (non-sensitive).

The sample golden set must hold **no** sensitive data. Adopters copy the sample. Then they add domain cases.

### 6.3 Extension points

Give adopters these extension points:

- Custom scorers.
- Custom tool mocks.
- Per-domain case packs.
- Host adapters (API, Bedrock, Vertex).

Document each extension point. Adopters must not fork the whole harness to add one scorer.

### 6.4 Governance

Keep cases frozen during comparisons. Use change control for scorers. Run a privacy review for any real-traffic sampling utilities that ship with the harness. A scorer change without a version bump makes history dishonest.

---

## 7. Deep notes — MCP and tool modules as accelerators

Ship domain MCP servers or tool libraries with these items:

- Narrow tools, not raw HTTP passthrough
- Auth examples using env secrets
- Structured errors
- Contract tests
- Allowlist recommendations
- One-page threat model summary

Version the servers. Document upgrade notes. Give a deny-by-default starter permission snippet for Claude Code.

Do not ship a raw superuser API as a convenience tool. Narrow tools are safer. Env secrets stay out of the repo. Contract tests prove schema and auth behavior without the full client app.

---

## 8. Deep notes — Documentation for handoff

### 8.1 Minimum viable handoff pack

A minimum viable handoff pack includes:

- README: what it is, when to use, when not to use.
- Quickstart: under fifteen minutes to first success.
- Architecture one-pager with diagram in text form.
- Configuration reference: pins, env vars, permissions.
- Security notes: secrets, denies, threat model summary.
- Ops: dashboards, alerts, kill switch, on-call tips.
- Evals: how to run, how to interpret, how to extend.
- Changelog and upgrade guide.
- Ownership and support channel.

If one of these files is missing, the next team cannot operate the build with safety. Handoff is part of the product. It is not extra work after the demo.

### 8.2 Handoff anti-patterns

Do not ship these handoff anti-patterns:

- Demo-only scripts that fail on a clean machine.
- Docs that need informal knowledge of last week's call.
- No rollback section.
- Screenshots of secrets.
- Undocumented model aliases in production paths.

A script that runs only on the author's laptop is not a quickstart. A screenshot of a secret is an incident. An undocumented alias in production causes surprise upgrades.

### 8.3 Audience split

Write for three readers: the engineer who implements, the security engineer who reviews, and the on-call engineer. Each reader needs different sections. One pack can serve all readers when headings are clear.

The engineer who implements needs the quickstart and configuration. The security engineer needs threat model and default denies. The on-call engineer needs dashboards, alerts, kill switch, and rollback.

---

## 9. Decision trees

### 9.1 Should this become an accelerator?

Use this tree.

If the same pattern ships three or more times, yes. Make it an accelerator.
If only one unusual client needs it, maybe write a case study. Do not make that case an accelerator.
If the work embeds customer secrets or proprietary data, sanitize the work. If you cannot sanitize it, do not publish.
If evals do not travel with it, it is not ready.

A pattern with one use is not an accelerator. A pattern without evals is not ready. A pattern with secrets is not publishable.

### 9.2 Skill versus template versus reference app

Select the asset type that matches the need:

- Need always-on conventions: CLAUDE.md template.
- Need a procedure sometimes: skill.
- Need an end-to-end learning path: thin reference app plus harness.
- Need enforceables: settings and hooks pack.

Do not put a rare procedure in always-on CLAUDE.md. Do not put a hard permission gate only in prose. Do not use a skill when the learner needs a full vertical slice.

### 9.3 Safe contribution checklist tree

Ask these questions in order:

- Secrets removed?
- Evals included?
- License and partner rules checked?
- Threat model written?
- Default permissions least privilege?
- Pins documented?
- Owner named?

If any answer is no, do not contribute yet. Fix the gap. Then run the tree again.

---

## 10. Exam traps

1. You treat accelerators as prompt storage without evals.
2. You ship bypass permissions to make demos smooth.
3. You put secrets in example configs.
4. You assume official IP-contribution modules are public (they are not).
5. You write very large CLAUDE.md templates that copy unstructured wiki content.
6. You ship MCP modules that expose raw superuser APIs.
7. You ship no versioning and no changelog.
8. You write handoff docs without rollback.
9. You measure contribution by file count instead of reuse and safety.
10. You ignore security review because it is only an accelerator.

Trap 4 is important. Official IP-contribution steps may sit behind official programs. These notes cover portable engineering practice only. Do not invent private portal steps on the exam.

An accelerator is still production software. Security review still applies. File count is not a success metric.

---

## 11. Self-check Q&A (20)

**Q1.** What is the difference between a skill and CLAUDE.md?
**A1.** Skills are on-demand procedures. CLAUDE.md is lean always-on guidance.

**Q2.** Name three things that must never ship inside an accelerator package.
**A2.** Secrets, live customer PII, and bypass-permissions-as-default.

**Q3.** Why travel evals with an agent template?
**A3.** Adopters can prove the template still works after they customize it.

**Q4.** When is a reference app better than a skill alone?
**A4.** When learners need an end-to-end vertical slice. The slice includes config and tests.

**Q5.** What belongs in security notes of a handoff pack?
**A5.** Threat model summary, secret injection pattern, default denies, and review checklist.

**Q6.** How do you keep CLAUDE.md templates maintainable?
**A6.** Assign an owner. Review every quarter. Use links over paste. Keep sections lean.

**Q7.** Why separate policy config from product prompt config?
**A7.** Consumers change tone more often than safety policy. Safer defaults then stay.

**Q8.** What is a contract test for an MCP accelerator?
**A8.** Schema and auth behavior tests that run without the full client app.

**Q9.** Scope of this chapter?
**A9.** Official IP-contribution steps may exist behind official programs. These notes cover portable engineering practice only.

**Q10.** Select a good accelerator success metric.
**A10.** Number of teams that adopted it with passing evals. Do not use lines of code.

**Q11.** Why pin models in templates?
**A11.** Pins give reproducibility for adopters and for CI.

**Q12.** What is a dangerous zone section?
**A12.** An explicit callout of migrations, billing, prod apply, and similar high-damage areas.

**Q13.** How should examples handle API keys?
**A13.** Use placeholders plus secret manager instructions. Never use real keys.

**Q14.** When to nest CLAUDE.md templates in a monorepo accelerator?
**A14.** When packages have distinct commands and conventions.

**Q15.** What makes a settings pack valuable?
**A15.** Shared hooks and denies that encode safety lessons from past incidents.

**Q16.** Why include a when-not-to-use section?
**A16.** The section prevents misuse on the wrong problem classes.

**Q17.** What is modular IP in this context?
**A17.** Reusable, versioned, documented assets that others can compose.

**Q18.** How do accelerators relate to Integration domain weight?
**A18.** Reusable configs, pins, MCP modules, and handoff quality are integration competence.

**Q19.** First review question from security on a new accelerator?
**A19.** What can it do if the model is compromised?

**Q20.** Map Chapter 05 to official domains.
**A20.** Integration, Agents, Eval, Claude Code, Tools/MCP, with Security defaults.

---

## 12. Checklist

- [ ] Accelerator has a clear when-to-use and when-not-to-use
- [ ] Secrets are absent. You document placeholders
- [ ] You document model and prompt pins
- [ ] Default permissions are least privilege
- [ ] Skills are lean and test-aware
- [ ] CLAUDE.md template is short and owned
- [ ] Eval harness and sample cases ship with it
- [ ] MCP or tools include contract tests
- [ ] Handoff docs include rollback and on-call
- [ ] Version and changelog exist
- [ ] You check legal or partner contribution rules before you publish internally or externally

Run this checklist before you publish. A missing item is a block. Do not skip the legal or partner check.

---

## 13. Glossary

**Accelerator:** a reusable solution kit. It helps you deliver faster.
**Modular IP:** versioned assets. You design them for composition and contribution.
**Skill:** on-demand procedure package for Claude Code.
**Reference app:** thin end-to-end example application.
**Handoff pack:** docs and assets that let another team operate the build.
**Harness:** reusable evaluation runner and scoring system.
**Golden starter set:** small non-sensitive labeled cases that ship with a template.
**Settings pack:** shared Claude Code settings, hooks, and permission starters.
**Contribution:** you submit reusable assets under org or program rules.
**Extension point:** documented place for adopters to customize with safety.
**Safe default:** configuration that fails closed until you loosen it on purpose.
**Vertical slice:** minimal path that exercises real integration points.

Learn these terms. Exam stems often use them. Do not confuse a skill with a reference app. Do not confuse a harness with a golden starter set.

---

## 14. Worked packaging examples (original)

### Example A — Support copilot accelerator

This accelerator includes:

- Agent loop template
- Versioned system prompts
- Policy retrieval tool interface (adapter port)
- Refund tool with caps
- Human publish gate for policy-facing content
- CLAUDE.md for the support repo layout
- Skill for add-new-intent
- Eval pack with injection and refund boundary cases (golden cases)
- Cost dashboard stub
- Runbook with kill switch
- README with when-not-to-use for clinical or legal advice domains

You must include the when-not-to-use section. Do not use this copilot for clinical advice. Do not use it for legal advice. The refund tool has caps. The human publish gate covers policy-facing content.

### Example B — Claude Code monorepo starter

This accelerator includes:

- Root and nested CLAUDE.md templates
- Settings with deny for secrets and prod kube contexts
- Hooks for format and secret scan
- Skills for create-package and cut-release
- sample.mcp.json with comments and disabled-by-default servers
- CI headless workflow with pins

Servers in sample.mcp.json start disabled. Adopters enable only the servers they need. The deny list covers secrets and prod kube contexts. CI uses pins, not aliases.

### Example C — Eval platform starter

This accelerator includes:

- Case schema
- CLI runner
- JSON and markdown reports
- Example scorers
- CI snippet
- Privacy guide for sampling
- Adapters for Anthropic API and a cloud host stub

The privacy guide is part of the product. Do not sample real traffic until privacy review passes. Host adapters keep the runner portable.

### Example D — MCP SaaS connector kit

This accelerator includes:

- Server skeleton
- OAuth env pattern
- Three task-shaped tools
- Contract tests
- Claude Code permission snippets
- Threat model
- Versioning guide

Tools are task-shaped. They are not raw HTTP passthrough. OAuth uses env vars. Contract tests travel with the kit.

---

## 15. Quality rubric for contributions (score 0-2 each)

Score each item from 0 to 2:

- Clarity of purpose.
- Pin defaults.
- Safety defaults.
- Eval completeness.
- Docs completeness.
- Extensibility.
- Operational readiness.
- Version discipline.
- Secret hygiene.
- Reuse evidence / ownership clarity.

**Ship threshold:** at least 14/20 overall, with **zero zeros** on safety or secret hygiene.

A high total with a zero on safety does not pass. A high total with a zero on secret hygiene does not pass. You must fix those zeros before you ship.

---

## 16. Mapping appendix for Chapter 05

Applications and Integration **33.1 percent:** templates, pins, MCP modules, handoff quality.
Agents and Workflows **14.7 percent:** packaged agent loops and budgets.
Eval **2.6 percent:** harness reuse and starter goldens.
Claude Code **3.1 percent:** skills, CLAUDE.md, settings packs.
Tools and MCPs **10.6 percent:** modular servers and allowlists.
Security **8.1 percent:** safe defaults and threat notes in every package.
Prompting **11.0 percent:** prompt templates with extension points.
MSO **16.8 percent:** documented default model and effort policy in accelerators.

Chapter 05 is not a private partner-process quiz. It is engineering substance across these domains. Integration has the largest weight. Package reusable configs, pins, MCP modules, and handoff quality.

---

## 17. Scope note (read carefully)

Official tracks may teach IP-contribution workflows, portals, and naming standards that are not public. This chapter stops at portable engineering practice on purpose.

If your organization gives official modules on accelerators, use those modules for process compliance. Use this file for technical substance.

Do not invent private portal steps on the exam. Do not claim gated official curriculum as public. Study pins, safe defaults, evals, and handoff docs.

---

## 18. Building an accelerator roadmap inside a team

Run the roadmap in this order:

**Phase 1:** Identify repeated delivery patterns from the last few projects.
**Phase 2:** Extract one thin vertical slice with evals.
**Phase 3:** Add safe defaults and handoff docs.
**Phase 4:** Pilot with a second team. Gather friction notes.
**Phase 5:** Version 1.0 with changelog and owner.
**Phase 6:** Only then advertise widely, or contribute to a partner catalog under your rules.

If you skip to a large framework first, you usually make unused software. Start thin. Add evals. Then add docs. Then pilot. Then version. Then advertise.

A first version without a pilot hides missing docs and unsafe defaults. A catalog item without an owner becomes drift.

---

## 19. Documentation templates (original outlines)

### README outline

Use this outline:

- Title and one-sentence purpose.
- Audience.
- When to use / when not to use (non-goals).
- Architecture snapshot.
- Quickstart.
- Configuration (incl. pin defaults).
- Security.
- Evals (how to run the smoke set).
- Ops.
- FAQ.
- Owners / support channel.

The when-not-to-use section is not optional. The quickstart must run on a clean machine.

### SECURITY.md outline

Use this outline:

- Assets and data flows.
- Trust boundaries.
- Default denies.
- Secret injection.
- Known residual risks.
- How to report issues.

Security reviewers open this file first. Keep default denies explicit. Keep secret injection explicit.

### OPERATIONS.md outline

Use this outline:

- Dashboards.
- Alerts.
- Kill switch steps.
- Common failures.
- Rate limits and host matrix.
- Upgrade procedure.
- Rollback procedure.
- On-call owner.

Rollback is a required section. Kill switch steps must be copyable. Name the on-call owner.

### EVALS.md outline

Use this outline:

- How to run.
- How to read reports.
- How to add cases.
- Frozen-set rules and must-pass gates.
- Scoring version policy.
- Privacy constraints.

Frozen-set rules keep comparisons honest. Privacy constraints block real customer transcripts without review.

---

## 20. Reusable prompt template packaging

Store prompts as versioned files. Do not store them only in a UI.

Include metadata: version, owner, intended model, eval suite id.
Provide a changelog.
Provide extension comments. The comments mark safe edit zones versus do-not-edit policy zones.
Keep examples free of customer data.

You cannot compare a prompt without a version. You cannot prove a prompt without an eval suite id. A UI-only prompt does not travel.

---

## 21. Additional Q&A (21-30)

**Q21.** Why pilot with a second team before catalog publication?
**A21.** The pilot finds missing docs and unsafe defaults under real reuse.

**Q22.** Can an accelerator include a Bedrock and API dual adapter?
**A22.** Yes, if the feature matrix and pins are explicit per host.

**Q23.** What is unused catalog software?
**A23.** An unused internal framework that fails in operations or in docs.

**Q24.** Should demos enable all MCP servers?
**A24.** No. Demos must model least privilege.

**Q25.** How do you prove reuse?
**A25.** Adoption count, eval passes on adopter forks, and issue tracker feedback.

**Q26.** What belongs in a changelog entry for a skill?
**A26.** Behavior changes, breaking steps, and required permission updates.

**Q27.** Why include when-not-to-use?
**A27.** Wrong use causes incidents. Incidents reduce trust in the catalog.

**Q28.** Is a screenshot-heavy wiki enough handoff?
**A28.** No. You need a runnable quickstart and rollback text.

**Q29.** How do accelerators help with CCDV-F Integration questions?
**A29.** They encode pinning, MCP trust, and operable configs as primary artifacts.

**Q30.** What is the final Chapter 05 rule?
**A30.** Package safe defaults, evals, and handoff docs. Do not package only clever prompts.

---

## 22. Scenario lab

**Scenario:** Your team builds a strong incident-triage agent for one client. Leadership wants it as IP.

Steps:
1. Sanitize data and brand.
2. Extract config.
3. Add budgets and denies.
4. Write a threat model.
5. Create golden cases from synthetic incidents.
6. Write the handoff pack.
7. Pilot with another team.
8. Version the package.
9. Contribute under org rules.

Do not publish client data. Use synthetic incidents for golden cases. Pilot before you contribute.

**Scenario:** A CLAUDE.md template is 4,000 lines because people paste ADRs.

Steps: cut to lean summary. Move procedures to skills. Link ADRs. Add owner. Measure whether agent performance improves with less noise.

A 4,000-line template is unstructured wiki content. Skills hold procedures. Links hold ADRs. Measure performance after the cut.

**Scenario:** An MCP accelerator uses a shared admin token in the example compose file.

Steps: stop distribution. Rotate token. Replace with env placeholders. Add pre-commit secret scan. Add SECURITY.md. Republish.

A shared admin token in an example is an incident. Stop distribution first. Rotate the token. Then fix the package.

---

## 23. One-page revision

Accelerators encode repeatable Claude delivery: agents, skills, templates, MCP modules, eval harnesses, and handoff docs.
Safe defaults are more important than impressive demos.
Evals travel with the asset.
Partner IP process may be private. Engineering substance here is public-practice oriented.
Success is reuse with safety, not repository size.

Remember the order: safe defaults, evals, handoff docs, then contribution. Do not reverse that order.

---

## 24. Detailed CLAUDE.md starter (illustrative, original)

Section Purpose: This repository implements X for Y users. Agents should prefer existing libraries in /packages/core before they invent new utilities.

Section Layout: /apps for deployable services, /packages for shared libraries, /evals for golden sets, /.claude for settings and skills.

Section Commands: Use the task runner documented in CONTRIBUTING for test, lint, and typecheck. Do not invent alternate commands. If those commands fail, document why.

Section Conventions: Prefer typed interfaces. Add tests beside code. Match existing error taxonomy. Do not widen permissions to make a task easier.

Section Dangerous zones: Do not edit database migrations without plan mode and human review. Do not change billing calculations without tests and HITL. Do not apply production infrastructure changes from an agent session.

Section Pointers: Architecture details live in /docs/architecture. API contracts live in /docs/openapi. Keep this file short on purpose.

This starter is illustrative. It shows lean sections. It does not copy the whole wiki. Dangerous zones are explicit. Pointers replace paste.

---

## 25. Skill starter outline (illustrative)

Name: cut-release
Preconditions: clean main, CI green, and a version bump that the team approves.
Steps: update changelog, bump version files, run release tests, open PR with summary, stop for human merge tag push.
Stops if: secrets detected, tests fail, unexpected dirty files outside release paths.
Notes: never force push. Never bypass hooks.

This outline meets the skill quality bar. Preconditions are explicit. Steps are listed. Stop conditions are named. Do not use dangerous variants (force push, hook bypass).

---

## 26. Eval harness adopter guide (short)

Install the harness. Copy sample cases. Set the model pin via env. Run make eval. Read the report.

Before you change prompts, duplicate the case pack. Keep the frozen baseline for delta comparison.

Do not commit real customer transcripts without privacy review.

The frozen baseline is the comparison anchor. A prompt change without a frozen baseline is not a measured change.

---

## 27. Contribution readiness scorecard

Score each item yes or no:

- purpose clear
- when-not-to-use written
- secrets scanned clean
- pins documented
- least privilege defaults (no allow-all MCP sample remains uncommented)
- smoke evals green on a clean checkout
- docs runnable on a clean machine
- SECURITY.md present
- rollback written
- owner named
- legal/license or partner checklist done

All items must be yes before catalog submission. One no is a block. An uncommented allow-all MCP sample fails.

---

## 28. Operating model for an internal accelerator catalog

Maintainers review submissions weekly against the scorecard. Security has veto on defaults. Each asset has an owner and a support channel.

Deprecated assets get a retirement date and migration notes. Announce deprecation clearly. Do not hide the sunset.

Adoption metrics appear on a dashboard. Include adopter time-to-first-safe-deploy. Run scheduled support hours. Never accept PRs that contain secrets. Adopters with broken builds can file issues that block the next release.

This operating model matters on exams when stems ask how a partner or platform team scales Claude delivery quality. The answer is packaged defaults plus evals plus ownership. The answer is not more presentations.

---

## 29. Cross-links back to the five-chapter path

After Chapter 01 you must know what defaults to pin inside accelerators.
After Chapter 02 you must know how to package agent loops and tool contracts.
After Chapter 03 you must know how to package Claude Code and MCP with safety.
After Chapter 04 you must refuse to publish without evals and security notes.
Chapter 05 turns those lessons into reusable IP-shaped artifacts.

Chapter 05 does not replace Chapters 01–04. It packages their judgment. Pins, loops, MCP trust, evals, and security notes all travel inside the accelerator.

---

## 30. Final drills

**Drill A:** Convert a one-off agent into a template checklist in ten bullets.
**Drill B:** Trim an overlong CLAUDE.md to seven lean sections.
**Drill C:** Write a when-not-to-use paragraph for a refund accelerator.
**Drill D:** List five scorecard failures that should block contribution.
**Drill E:** Explain in two sentences what this chapter covers without inventing private lesson content.

Do these drills in writing. Drill E is a scope check. Do not invent official portal steps. Stay on engineering substance.

---

## 31. Closing

Accelerators are how good Claude engineering becomes organizational memory. Keep them lean, safe, evaluated, versioned, and documented for handoff. That is the public-practice core of Chapter 05. Your official program may add private submission steps on top. Those rituals are out of scope here.

---

## 32. Appendix — quick artifact matrix

**Agent template:** ships loop, budgets, tools, evals.
**Skill:** ships procedure steps and stop rules.
**CLAUDE.md template:** ships lean always-on guidance.
**Settings pack:** ships hooks and permission starters.
**MCP module:** ships tools, auth pattern, contract tests.
**Eval harness:** ships runner, scorers, sample cases.
**Handoff pack:** ships README, security, ops, rollback.
**Reference app:** ships a thin vertical slice that ties them together.

If your catalog item cannot sit in this matrix, it is probably an essay, not an accelerator. Rewrite until it becomes an operable artifact. Another engineer must run it on a clean machine in under fifteen minutes. That engineer must not need private chat history.

*Use this file alongside Chapters 01-04 and the README mapping appendix for exam preparation without access to official courses. The §33 primary-study deepening continues below.*

Remember: reusable is more important than clever. Safe defaults are more important than demos. Evals are part of the product you contribute.

Package the judgment, not only the prompts.

Stay modular.
---

## 33. Primary-study deepening — Accelerators and reusable IP

Chapter 05 covers packaging and contribution themes that may not be fully public. These notes remain original study material on reusable agents, skills, CLAUDE.md templates, eval harnesses, MCP kits, and handoff docs. Those skills still appear inside Applications, Agents, Claude Code, and Eval scenarios.

### 33.1 What accelerator means here

An accelerator is a versioned, documented, evaluable package. It helps another engineer ship a Claude capability faster. That engineer does not copy secrets or one-off unique configs.

| Attribute | Bar |
| --- | --- |
| Clear problem statement | Who it is for / not for |
| Pin bundle defaults | Model/effort/prompt/tool versions |
| Safety defaults | Deny-by-default dangerous tools |
| Eval smoke | At least a tiny golden slice |
| Handoff docs | README + SECURITY + how to roll back |
| Extension points | Documented, not fork-only |

Meet every row of this bar. A package that fails one row is not ready.

### 33.2 Packaging reusable agents

Make the agent reusable with these choices:

- Config over code for model pins, budgets, and tool allowlists.
- Interfaces for tools with reference MCP/server adapters.
- Explicit stop conditions and human-gate hooks.
- Telemetry hooks (redacted).

Anti-patterns: hardcoded prod URLs/keys. Works in our monorepo only without adapters. No evals. Skills that embed customer data.

Reference skeleton:

```text
/agent
 README.md
 SECURITY.md
 EVALS.md
 config/pin.defaults.yaml
 prompts/system.vN.md
 tools/schema.vN.json
 harness/golden/
 src/runtime/
```

Copy this layout. Put pins in config. Put prompts in versioned files. Put golden cases in harness/golden/. Do not put secrets in this tree.

### 33.3 Skills packaging (Claude Code)

| Artifact | Use when |
| --- | --- |
| CLAUDE.md | Always-on team guidance |
| Skill | Invoked workflow such as /review-pr |
| Hook | Hard enforce around events |
| Plugin | Distribute bundle of skills/hooks/MCP |
| settings.json | Permissions/model defaults |

Skill quality bar: single purpose. Inputs clear. Examples. Failure modes. No secrets. Links to evals if it changes code. Version skills like APIs. Breaking changes bump major. Keep a changelog.

### 33.4 CLAUDE.md templates as IP

Lean sections: purpose, build/test commands, code style, security invariants, review checklist, ask humans when.

Variants: service repo, data/ML repo, infra repo, docs repo.

Do not put: API keys, customer PII, unverified rumors, contradictory absolutes better enforced in settings.

Maintenance: owners. Review quarterly. Tie to incidents.

A template is IP when it is lean, owned, and versioned. A template is harm when it is long, contradictory, or it pretends to enforce controls.

### 33.5 Eval harness reuse

Harnesses are high-leverage IP. They include:

- Runner plus scoring plus report format
- Fixture layout conventions
- Safety slice always on
- Plug-in scorers (schema, task success, judge)
- Cost/latency capture for Integration regressions

Governance: changes to harness scoring need version bumps. Historical comparisons then stay honest.

### 33.6 MCP and tool modules as accelerators

Ship these items:

- Server stub with auth patterns
- Tool schemas plus deny list examples
- Contract tests
- Threat model paragraph
- Allowlist snippets for Claude Code settings

Trap: you publish a server that over-scopes OAuth scopes for convenience. Do not do that.

### 33.7 Documentation for handoff

Minimum pack: README (quickstart), SECURITY, OPERATIONS (pins, dashboards, rollback), EVALS, CODEOWNERS/support channel.

Anti-patterns: screenshot-only setup. Informal chat knowledge. Undocumented breakages. "contact Bob".

Audience split: adopter engineer, security reviewer, on-call.

Do not name one person as the only path. Name a channel. Write rollback in text.

### 33.8 Decision trees

Should this become an accelerator?

```text
Used successfully >=2 times in different contexts?
 NO -> keep as example
 YES -> Can you remove secrets and snowflakes?
 NO -> extract patterns only
 YES -> package + evals + docs
```

Skill versus template versus reference app:

```text
Always-on guidance -> CLAUDE.md template
Invoked workflow -> Skill
Full runtime + tools -> Reference app/agent kit
```

Use the first tree before you invest in packaging. Use the second tree to select the asset type. Do not skip the secret-removal question.

### 33.9 Exam traps (packaging-flavored)

1. You ship accelerators with embedded secrets.
2. You give no safety defaults (you assume adopters tighten later).
3. You claim gated official curriculum as public.
4. You ship skills that silently change permissions.
5. You ship evals that only check tone.
6. You ship MCP kits with allow-all samples.
7. You ship templates that contradict enforced settings.
8. You ship no versioning — silent drift across teams.

Adopters do not tighten defaults later. Tone-only evals miss task success. Silent permission changes are a security failure. Versioning prevents silent drift.

### 33.10 Additional Q&A (Q31-Q45)

**Q31.** What is the first file a security reviewer opens in an accelerator?
**A31.** SECURITY.md / threat model and the default allowlists — then schemas.

**Q32.** Why include a tiny golden eval in every agent kit?
**A32.** It proves the pin bundle runs. It catches adopter misconfig. It anchors regressions.

**Q33.** When is a CLAUDE.md template harmful?
**A33.** When it is so long or contradictory that teams ignore it, or when it pretends to enforce controls.

**Q34.** How do you distribute a skill safely?
**A34.** Code review, no secrets, pinned versions, and clear required permissions listed. No silent escalation.

**Q35.** Accelerator success metric?
**A35.** Time-to-first-safe-deploy for adopters plus eval pass rate. Not popularity scores (stars).

**Q36.** Why separate pin.defaults from pin.prod examples?
**A36.** Defaults are safe starters. Prod pins are environment-specific. Do not copy them without a check across hosts.

**Q37.** What belongs in OPERATIONS.md?
**A37.** Dashboards, alerts, rollback, rate limit contacts, host matrix.

**Q38.** Can Chapter 05 content appear on CCDV-F?
**A38.** Yes, as Integration/Agents/Code/Eval judgment about reusable configs. Not as partner IP process trivia. Study the engineering substance.

**Q39.** Hook packaged in a plugin deletes node_modules on every edit — problem?
**A39.** Yes. That is a dangerous default. Hooks must be least privilege. Destructive actions must be opt-in.

**Q40.** What is a good extension point?
**A40.** Bring your own search_kb tool adapter. Do not fork the whole agent.

**Q41.** How do you version prompts in a kit?
**A41.** system.vN.md plus pin file reference plus changelog. Never overwrite silently.

**Q42.** Why include a deny-list example for MCP?
**A42.** It teaches adopters that they configure each connector separately.

**Q43.** Reference agent versus production service?
**A43.** Reference teaches patterns. Production adds tenancy, SLOs, compliance. Do not confuse demos with prod.

**Q44.** What is a contribution readiness scorecard item?
**A44.** Secrets scanned, evals green, docs complete, owners listed, license clear.

**Q45.** Map accelerator work to official domains.
**A45.** Applications (reusable config/pins), Agents (packaged loops), Claude Code (skills/templates), Eval (harness reuse), Security (safe defaults).

### 33.11 If exam asks X, think Y (Chapter 05)

| If exam asks | Think |
| --- | --- |
| Scale best practice | Templates + skills + harnesses with safe defaults |
| Share MCP | Reviewed kit + allowlists + tests |
| Team onboarding to Code | CLAUDE.md + shared settings + trust model |
| Prove reuse quality | Golden evals + version pins |
| Handoff | README/SECURITY/OPS/EVALS |
| Partner packaging process | Engineering substance only. Flag non-public process |

Read the left column as the stem. Answer from the right column. For partner packaging process, stay on engineering substance. Flag non-public process. Do not invent portal steps.

### 33.12 Glossary addendum

| Term | Meaning |
| --- | --- |
| Accelerator | Reusable, documented, evaluable package |
| Pin defaults | Starter versions for adopters |
| Extension point | Documented swap-out interface |
| Handoff pack | Docs set for another team |
| Safe default | Deny/least privilege by default |
| Smoke eval | Tiny critical golden slice |
| Reference agent | Teaching implementation, not prod tenant system |
| Contribution bar | Checklist before publishing internally |

Use this addendum with §13. A reference agent is a teaching implementation. It is not a production tenant system. A smoke eval is a tiny critical golden slice. It is not the full suite.

### 33.13 Primary-study checklist (Chapter 05)

- [ ] I can list the minimum accelerator bar.
- [ ] I can select CLAUDE.md versus skill versus hook versus reference app.
- [ ] I can sketch an agent kit directory without secrets.
- [ ] I can explain why evals ship with kits.
- [ ] I can write a one-page SECURITY section for an MCP kit.
- [ ] I know Chapter 05 may exceed public curriculum — I study transferable engineering.

Tick each box only when you can perform the item. The last box is a scope reminder. Study transferable engineering. Do not study private portal trivia.

### 33.14 Building an internal accelerator catalog

Merged into §28 (operating model) and §18 (roadmap phases). Roadmap theme recall: starter CLAUDE.md set. Eval harness core. Support-copilot agent kit. MCP SaaS connector kit. Claude Code monorepo settings template.

Use that recall list when you plan a catalog. Start with CLAUDE.md. Add the eval harness. Then add one agent kit and one MCP kit. Then add monorepo settings.

### 33.15 Quality rubric for contributions

Merged into §15 (canonical rubric with the 14/20 + zero-zeros-on-safety threshold).

Recall the ship rule: at least 14/20 overall. Zero zeros on safety. Zero zeros on secret hygiene.

### 33.16 Worked packaging examples

Merged into §14 (canonical detailed Examples A–D).

Recall the four kits: support copilot, Claude Code monorepo starter, eval platform starter, MCP SaaS connector kit.

### 33.17 Closing — Chapter 05 as primary study

Packaging is how Integration quality multiplies. Official packaging courses may or may not be available to you. The public-doc-aligned skills still remain exam-relevant and job-relevant. Those skills are pins, safe defaults, evals, CLAUDE.md/skills/hooks, and MCP kits.

### 33.18 Documentation templates

Merged into §19 (canonical README / SECURITY / OPERATIONS / EVALS outlines).

Use those four outlines as the handoff pack skeleton. Do not replace them with a screenshot wiki.

### 33.19 Contribution readiness scorecard

Merged into §27 (canonical pass/fail scorecard).

All scorecard items must be yes before catalog submission. One no is a block.

### 33.20 Final reminder

Treat Chapter 05 as primary study for reusable Integration quality: pins, safe defaults, evals, and handoff docs. Re-check public Claude Code and MCP docs before exam day. Gated packaging process details are out of scope for this pack.
