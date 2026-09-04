---
title: Accelerators and IP Contribution
pack: CCDV-F Developer Foundations
disclaimer: Original study notes — independent and not official course content
approx_length: ~5000–7500 words (primary study)
updated: 2026-08-23
---

# Chapter 05 — Accelerators and IP Contribution

> **Disclaimer:** Original study notes. These notes do **not** reproduce official course content. Instead they provide original, exam-useful guidance on reusable agents, skills packaging, CLAUDE.md templates, eval harness reuse, and documentation for handoff — grounded in public Claude Code, MCP, API, and engineering practice.

**Maps primarily to:** Applications and Integration (reusable configs and templates) · Agents and Workflows (packaged agents) · Eval (harness reuse).
**Secondary:** Claude Code (skills, templates) · Tools/MCP (modular servers) · Security (safe defaults in packages).

---

## 1. Overview

Accelerators are **reusable building blocks** that help the next engineer ship a Claude solution faster without copying tribal Slack lore. On exams and in partner delivery, strong answers package:

1. A default architecture with pinned models and safe permissions.
2. Prompt and CLAUDE.md templates with clear extension points.
3. Skills or playbooks for repeated procedures.
4. MCP or tool modules with allowlists and auth patterns.
5. An eval harness and golden starter set.
6. Handoff docs: how to run, how to change, how to rollback.

IP contribution means turning a one-off client win into **modular assets** others can adopt under whatever licensing and contribution rules your organization uses. This file teaches the engineering substance; follow your firm's legal and partner-program rules for actual submission.

---

## 2. Key map

Asset type: Agent template — What it encodes: loop, budgets, tools, verifiers.
Asset type: Skill — What it encodes: on-demand procedure for Claude Code.
Asset type: CLAUDE.md template — What it encodes: lean always-on repo guidance.
Asset type: Settings pack — What it encodes: permissions, hooks, MCP gates.
Asset type: MCP module — What it encodes: domain tools with auth and errors.
Asset type: Eval harness — What it encodes: runner, scorers, CI gate.
Asset type: Runbook — What it encodes: operate, incident, upgrade steps.
Asset type: Reference app — What it encodes: end-to-end thin vertical slice.

---

## 3. Deep notes — Packaging reusable agents

### 3.1 What makes an agent reusable

Clear inputs and outputs. Documented tools. Pinned defaults with override hooks. Budgets and stop conditions included. Eval cases that travel with the agent. Safe-by-default permissions. No embedded secrets. Versioned releases.

### 3.2 Configuration surface

Separate:
- **Policy** (safety rules, denies)
- **Product prompts** (tone, task framing)
- **Environment** (endpoints, model pins)
- **Secrets** (injected, never packaged)

Consumers should override environment and product prompts more often than policy.

### 3.3 Reference agent skeleton (conceptual)

Intake message schema.
Planner optional step.
Tool registry with allowlist.
Loop with max steps and wall clock.
Verifier hooks (schema, tests, policy).
Escalation path.
Telemetry hooks.
Eval entrypoint.

### 3.4 Anti-patterns in agent packaging

Hardcoded customer names. Secrets in repo. Unbounded tools. No version. No evals. Docs that say only run main. Bypass permissions enabled to demo faster.

---

## 4. Deep notes — Skills packaging (Claude Code)

### 4.1 Skills versus CLAUDE.md versus hooks

CLAUDE.md: short always-on context.
Skill: procedure invoked when relevant; keeps always-on context lean.
Hook or permission: must-enforce gate.

Package skills for: release checklist, add HTTP endpoint, write migration safely, triage flaky test, prepare PR description.

### 4.2 Skill quality bar

States preconditions. Lists steps. Names tools involved. Declares stop conditions. Links to tests. Notes dangerous variants. Includes examples of good and bad outcomes.

### 4.3 Versioning skills

Semantic version or dated changelog. Note breaking changes. Keep skills in repo so PRs review them. Avoid personal-only skills for team accelerators.

### 4.4 Plugin-style packaging

Where teams use plugins or shared bundles, include: settings recommendations, skills, hook scripts, example CLAUDE.md, and a README that explains trust boundaries. Consumers should still review MCP and permissions before enabling.

---

## 5. Deep notes — CLAUDE.md templates

### 5.1 Lean template sections

1. Repo purpose in three lines
2. Directory map
3. Canonical commands
4. Conventions (errors, testing, APIs)
5. Dangerous zones
6. Pointers to deep docs
7. Explicit non-goals (do not invent infra)

### 5.2 Template variants

Monorepo root template plus nested package template.
Library template versus service template.
Data science template with notebook and eval notes.
Infrastructure template with plan-mode emphasis and deny lists for prod apply.

### 5.3 What not to put in templates

Secrets. Customer PII. Volatile timestamps. Entire architecture decision records. Contradictory rules from multiple authors without ownership.

### 5.4 Maintenance

Assign an owner. Review quarterly. Tie updates to broken evals or incidents. Prefer links over paste.

---

## 6. Deep notes — Eval harness reuse

### 6.1 Why harnesses are IP

The harness encodes how your organization measures Claude quality. Reusing it prevents every team from inventing inconsistent score rituals.

### 6.2 Harness components to package

Case schema. Fixture loaders. Model and prompt pin interface. Tool simulators. Scorers (exact, fuzzy, rubric, side-effect). Report formats. CI examples. Sample golden set (non-sensitive).

### 6.3 Extension points

Custom scorers. Custom tool mocks. Per-domain case packs. Host adapters (API, Bedrock, Vertex).

### 6.4 Governance

Frozen cases during comparisons. Change control for scorers. Privacy review for any real-traffic sampling utilities that ship with the harness.

---

## 7. Deep notes — MCP and tool modules as accelerators

Ship domain MCP servers or tool libraries with:
- Narrow tools, not raw HTTP passthrough
- Auth examples using env secrets
- Structured errors
- Contract tests
- Allowlist recommendations
- Threat model one-pager

Version servers. Document upgrade notes. Provide a deny-by-default starter permission snippet for Claude Code.

---

## 8. Deep notes — Documentation for handoff

### 8.1 Minimum viable handoff pack

README: what it is, when to use, when not to use.
Quickstart: under fifteen minutes to first success.
Architecture one-pager with diagram in text form.
Configuration reference: pins, env vars, permissions.
Security notes: secrets, denies, threat model summary.
Ops: dashboards, alerts, kill switch, on-call tips.
Evals: how to run, how to interpret, how to extend.
Changelog and upgrade guide.
Ownership and support channel.

### 8.2 Handoff anti-patterns

Demo-only scripts that fail on a clean machine.
Docs that require tribal knowledge of last week's Zoom.
No rollback section.
Screenshots of secrets.
Undocumented model aliases in production paths.

### 8.3 Audience split

Write for three readers: implementing engineer, reviewing security engineer, and future you on-call. Each needs different sections, but one pack can serve all with clear headings.

---

## 9. Decision trees

### 9.1 Should this become an accelerator?

If the same pattern shipped three or more times, yes.
If only one exotic client needs it, maybe a case study instead of a productized accelerator.
If it embeds customer secrets or proprietary data, sanitize or do not publish.
If evals do not travel with it, it is not ready.

### 9.2 Skill versus template versus reference app

Need always-on conventions: CLAUDE.md template.
Need a procedure sometimes: skill.
Need an end-to-end learning path: thin reference app plus harness.
Need enforceables: settings and hooks pack.

### 9.3 Safe contribution checklist tree

Secrets removed? Evals included? License and partner rules checked? Threat model written? Default permissions least privilege? Pins documented? Owner named? If any no, do not contribute yet.

---

## 10. Exam traps

1. Treating accelerators as prompt paste bins without evals.
2. Shipping bypass permissions to make demos smooth.
3. Putting secrets in example configs.
4. Assuming official IP-contribution modules are public (they are not).
5. Giant CLAUDE.md templates that recreate wiki dumps.
6. MCP modules that expose raw superuser APIs.
7. No versioning or changelog.
8. Handoff docs without rollback.
9. Measuring contribution by file count instead of reuse and safety.
10. Ignoring security review because it is only an accelerator.

---

## 11. Self-check Q&A (20)

Q1. What is the difference between a skill and CLAUDE.md? Skills are on-demand procedures; CLAUDE.md is lean always-on guidance.
Q2. Name three things that must never ship inside an accelerator package. Secrets, live customer PII, bypass-permissions-as-default.
Q3. Why travel evals with an agent template? So adopters can prove the template still works after they customize it.
Q4. When is a reference app better than a skill alone? When learners need an end-to-end vertical slice including config and tests.
Q5. What belongs in security notes of a handoff pack? Threat model summary, secret injection pattern, default denies, and review checklist.
Q6. How do you keep CLAUDE.md templates maintainable? Owner, quarterly review, links over paste, lean sections.
Q7. Why separate policy config from product prompt config? Consumers change tone more often than safety policy; safer defaults stick.
Q8. What is a contract test for an MCP accelerator? Schema and auth behavior tests that run without the full client app.
Q9. Scope of this chapter? Official IP-contribution steps may exist behind official programs; these notes cover portable engineering practice only.
Q10. Select a good accelerator success metric. Number of teams that adopted it with passing evals — not lines of code.
Q11. Why pin models in templates? Reproducibility for adopters and CI.
Q12. What is a dangerous zone section? Explicit callout of migrations, billing, prod apply, and similar high blast-radius areas.
Q13. How should examples handle API keys? Placeholders plus secret manager instructions — never real keys.
Q14. When to nest CLAUDE.md templates in a monorepo accelerator? When packages have distinct commands and conventions.
Q15. What makes a settings pack valuable? Shared hooks and denies that encode hard-won safety lessons.
Q16. Why include a when-not-to-use section? Prevents misuse on wrong problem classes.
Q17. What is modular IP in this context? Reusable, versioned, documented assets others can compose.
Q18. How do accelerators relate to Integration domain weight? Reusable configs, pins, MCP modules, and handoff quality are integration competence.
Q19. First review question from security on a new accelerator? What can it do if the model is compromised?
Q20. Map Chapter 05 to official domains. Integration, Agents, Eval, Claude Code, Tools/MCP, with Security defaults.

---

## 12. Checklist

- [ ] Accelerator has a clear when-to-use and when-not-to-use
- [ ] Secrets are absent; placeholders documented
- [ ] Model and prompt pins documented
- [ ] Default permissions are least privilege
- [ ] Skills are lean and test-aware
- [ ] CLAUDE.md template is short and owned
- [ ] Eval harness and sample cases ship with it
- [ ] MCP or tools include contract tests
- [ ] Handoff docs include rollback and on-call
- [ ] Version and changelog exist
- [ ] Legal or partner contribution rules checked before publishing internally or externally

---

## 13. Glossary

Accelerator: reusable solution kit that speeds delivery.
Modular IP: versioned assets designed for composition and contribution.
Skill: on-demand procedure package for Claude Code.
Reference app: thin end-to-end example application.
Handoff pack: docs and assets that let another team operate the build.
Harness: reusable evaluation runner and scoring system.
Golden starter set: small non-sensitive labeled cases shipping with a template.
Settings pack: shared Claude Code settings, hooks, and permission starters.
Contribution: submitting reusable assets under org or program rules.
Extension point: documented place for adopters to customize safely.
Safe default: configuration that fails closed until deliberately loosened.
Vertical slice: minimal path exercising real integration points.

---

## 14. Worked packaging examples (original)

### Example A — Support copilot accelerator

Includes: agent loop template, versioned system prompts, policy retrieval tool interface (adapter port), refund tool with caps, human publish gate for policy-facing content, CLAUDE.md for the support repo layout, skill for add-new-intent, eval pack with injection and refund boundary cases (golden tickets), cost dashboard stub, runbook with kill switch, README with when-not-to-use for clinical or legal advice domains.

### Example B — Claude Code monorepo starter

Includes: root and nested CLAUDE.md templates, settings with deny for secrets and prod kube contexts, hooks for format and secret scan, skills for create-package and cut-release, sample.mcp.json with comments and disabled-by-default servers, CI headless workflow with pins.

### Example C — Eval platform starter

Includes: case schema, CLI runner, JSON and markdown reports, example scorers, CI snippet, privacy guide for sampling, adapters for Anthropic API and a cloud host stub.

### Example D — MCP SaaS connector kit

Includes: server skeleton, OAuth env pattern, three task-shaped tools, contract tests, Claude Code permission snippets, threat model, versioning guide.

---

## 15. Quality rubric for contributions (score 0-2 each)

Clarity of purpose. Pin defaults. Safety defaults. Eval completeness. Docs completeness. Extensibility. Operational readiness. Version discipline. Secret hygiene. Reuse evidence / ownership clarity.

**Ship threshold:** at least 14/20 overall, with **zero zeros** on safety or secret hygiene.

---

## 16. Mapping appendix for Chapter 05

Applications and Integration 33.1 percent: templates, pins, MCP modules, handoff quality.
Agents and Workflows 14.7 percent: packaged agent loops and budgets.
Eval 2.6 percent: harness reuse and starter goldens.
Claude Code 3.1 percent: skills, CLAUDE.md, settings packs.
Tools and MCPs 10.6 percent: modular servers and allowlists.
Security 8.1 percent: safe defaults and threat notes in every package.
Prompting 11.0 percent: prompt templates with extension points.
MSO 16.8 percent: documented default model and effort policy in accelerators.

---

## 17. Scope note (read carefully)

Official tracks may teach IP-contribution workflows, portals, and naming standards that are not public. This chapter intentionally stops at portable engineering practice; if your organization provides official modules on accelerators, use those for process compliance and this file for technical substance.

---

## 18. Building an accelerator roadmap inside a team

Phase 1: Identify repeated delivery patterns from the last few projects.
Phase 2: Extract one thin vertical slice with evals.
Phase 3: Add safe defaults and handoff docs.
Phase 4: Pilot with a second team; gather friction notes.
Phase 5: Version 1.0 with changelog and owner.
Phase 6: Only then advertise widely or contribute to a partner catalog under your rules.

Skipping straight to a large framework usually creates shelfware.

---

## 19. Documentation templates (original outlines)

### README outline

Title and one-sentence purpose.
Audience.
When to use / when not to use (non-goals).
Architecture snapshot.
Quickstart.
Configuration (incl. pin defaults).
Security.
Evals (how to run the smoke set).
Ops.
FAQ.
Owners / support channel.

### SECURITY.md outline

Assets and data flows.
Trust boundaries.
Default denies.
Secret injection.
Known residual risks.
How to report issues.

### OPERATIONS.md outline

Dashboards.
Alerts.
Kill switch steps.
Common failures.
Rate limits and host matrix.
Upgrade procedure.
Rollback procedure.
On-call owner.

### EVALS.md outline

How to run.
How to read reports.
How to add cases.
Frozen-set rules and must-pass gates.
Scoring version policy.
Privacy constraints.

---

## 20. Reusable prompt template packaging

Store prompts as versioned files, not only in a UI.
Include metadata: version, owner, intended model, eval suite id.
Provide a changelog.
Provide extension comments that mark safe edit zones versus do-not-edit policy zones.
Keep examples free of customer data.

---

## 21. Additional Q&A (21-30)

Q21. Why pilot with a second team before catalog publication? Discovers missing docs and unsafe defaults under real reuse.
Q22. Can an accelerator include a Bedrock and API dual adapter? Yes if the feature matrix and pins are explicit per host.
Q23. What is shelfware? An unused internal framework that fails operationally or docs-wise.
Q24. Should demos enable all MCP servers? No — demos should model least privilege.
Q25. How do you prove reuse? Adoption count, eval passes on adopter forks, issue tracker feedback.
Q26. What belongs in a changelog entry for a skill? Behavior changes, breaking steps, required permission updates.
Q27. Why include when-not-to-use? Misapplication causes incidents and erodes trust in the catalog.
Q28. Is a screenshot-heavy wiki enough handoff? No — need runnable quickstart and rollback text.
Q29. How do accelerators help with CCDV-F Integration questions? They encode pinning, MCP trust, and operable configs as first-class artifacts.
Q30. Final Chapter 05 mantra? Package safe defaults, evals, and handoff docs — not just clever prompts.

---

## 22. Scenario lab

Scenario: Your team built a great incident-triage agent for one client. Leadership wants it as IP.
Steps: sanitize data and brand; extract config; add budgets and denies; write threat model; create golden cases from synthetic incidents; write handoff pack; pilot with another squad; version; contribute under org rules.

Scenario: A CLAUDE.md template is 4,000 lines after people pasted ADRs.
Steps: cut to lean summary; move procedures to skills; link ADRs; add owner; measure whether agent performance improves with less noise.

Scenario: An MCP accelerator uses a shared admin token in the example compose file.
Steps: stop distribution; rotate token; replace with env placeholders; add pre-commit secret scan; add SECURITY.md; republish.

---

## 23. One-page revision

Accelerators encode repeatable Claude delivery: agents, skills, templates, MCP modules, eval harnesses, and handoff docs.
Safe defaults beat flashy demos.
Evals travel with the asset.
Partner IP process may be private; engineering substance here is public-practice oriented.
Success is reuse with safety, not repository size.


---

## 24. Detailed CLAUDE.md starter (illustrative, original)

Section Purpose: This repository implements X for Y users. Agents should prefer existing libraries in /packages/core before inventing new utilities.

Section Layout: /apps for deployable services, /packages for shared libraries, /evals for golden sets, /.claude for settings and skills.

Section Commands: Use the task runner documented in CONTRIBUTING for test, lint, and typecheck. Do not invent alternate commands unless those fail and you document why.

Section Conventions: Prefer typed interfaces. Add tests beside code. Match existing error taxonomy. Do not widen permissions to make a task easier.

Section Dangerous zones: Do not edit database migrations without plan mode and human review. Do not change billing calculations without tests and HITL. Do not apply infrastructure changes to production from an agent session.

Section Pointers: Architecture details live in /docs/architecture. API contracts live in /docs/openapi. Keep this file short on purpose.

---

## 25. Skill starter outline (illustrative)

Name: cut-release
Preconditions: clean main, CI green, version bump agreed.
Steps: update changelog, bump version files, run release tests, open PR with summary, stop for human merge tag push.
Stops if: secrets detected, tests fail, unexpected dirty files outside release paths.
Notes: never force push; never bypass hooks.

---

## 26. Eval harness adopter guide (short)

Install harness. Copy sample cases. Set model pin via env. Run make eval. Read report. Before changing prompts, duplicate the case pack and keep the frozen baseline for delta comparison. Do not commit real customer transcripts without privacy review.

---

## 27. Contribution readiness scorecard

Score each item yes or no: purpose clear; when-not-to-use written; secrets scanned clean; pins documented; least privilege defaults (no allow-all MCP sample left uncommented); smoke evals green on a clean checkout; docs runnable on a clean machine; SECURITY.md present; rollback written; owner named; legal/license or partner checklist done. All must be yes before catalog submission.


---

## 28. Operating model for an internal accelerator catalog

Maintainers review submissions weekly against the scorecard. Security has veto on defaults. Each asset has an owner and a support channel. Deprecated assets get a sunset date and migration notes ("deprecate loudly"). Adoption metrics appear on a dashboard — including adopter time-to-first-safe-deploy. Run office hours; never accept secret-bearing PRs. Broken adopters can file issues that gate the next release.

This operating model matters on exams when stems ask how a partner or platform team scales Claude delivery quality — the answer is packaged defaults plus evals plus ownership, not more slide decks.

---

## 29. Cross-links back to the five-chapter path

After Chapter 01 you should know what defaults to pin inside accelerators.
After Chapter 02 you should know how to package agent loops and tool contracts.
After Chapter 03 you should know how to package Claude Code and MCP safely.
After Chapter 04 you should refuse to publish without evals and security notes.
Chapter 05 turns those lessons into reusable IP-shaped artifacts.

---

## 30. Final drills

Drill A: Convert a one-off agent into a template checklist in ten bullets.
Drill B: Trim a bloated CLAUDE.md to seven lean sections.
Drill C: Write a when-not-to-use paragraph for a refund accelerator.
Drill D: List five scorecard failures that should block contribution.
Drill E: Explain in two sentences what this chapter covers without inventing private lesson content.

---

## 31. Closing

Accelerators are how good Claude engineering becomes organizational memory. Keep them lean, safe, evaluated, versioned, and documented for handoff. That is the public-practice core of Chapter 05, whether or not your official program adds private submission rituals on top.


---

## 32. Appendix — quick artifact matrix

Agent template: ships loop, budgets, tools, evals.
Skill: ships procedure steps and stop rules.
CLAUDE.md template: ships lean always-on guidance.
Settings pack: ships hooks and permission starters.
MCP module: ships tools, auth pattern, contract tests.
Eval harness: ships runner, scorers, sample cases.
Handoff pack: ships README, security, ops, rollback.
Reference app: ships a thin vertical slice tying them together.

If your catalog item cannot be placed in this matrix, it is probably an essay, not an accelerator. Rewrite until it becomes an operable artifact another engineer can run on a clean machine in under fifteen minutes without private Slack history.


*Use this file alongside Chapters 01-04 and the README mapping appendix for exam prep without access to official courses. The §33 primary-study deepening continues below.*

Remember: reusable beats clever, safe defaults beat demos, and evals are part of the product you contribute.

Package the judgment, not only the prompts.

Stay modular.
---

## 33. Primary-study deepening — Accelerators and reusable IP

Chapter 05 covers packaging and contribution themes that may not be fully public. These notes remain original study material on reusable agents, skills, CLAUDE.md templates, eval harnesses, MCP kits, and handoff docs — skills that still show up inside Applications, Agents, Claude Code, and Eval scenarios.


### 33.1 What accelerator means here

An accelerator is a versioned, documented, evaluable package that helps another engineer ship a Claude capability faster without copying secrets or one-off snowflake configs.

| Attribute | Bar |
| --- | --- |
| Clear problem statement | Who it is for / not for |
| Pin bundle defaults | Model/effort/prompt/tool versions |
| Safety defaults | Deny-by-default dangerous tools |
| Eval smoke | At least a tiny golden slice |
| Handoff docs | README + SECURITY + how to roll back |
| Extension points | Documented, not fork-only |

### 33.2 Packaging reusable agents

Make reusable: config over code for model pins, budgets, tool allowlists; interfaces for tools with reference MCP/server adapters; explicit stop conditions and human-gate hooks; telemetry hooks (redacted).

Anti-patterns: hardcoded prod URLs/keys; works in our monorepo only without adapters; no evals; skills that embed customer data.

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

### 33.3 Skills packaging (Claude Code)

| Artifact | Use when |
| --- | --- |
| CLAUDE.md | Always-on team guidance |
| Skill | Invoked workflow such as /review-pr |
| Hook | Hard enforce around events |
| Plugin | Distribute bundle of skills/hooks/MCP |
| settings.json | Permissions/model defaults |

Skill quality bar: single purpose; inputs clear; examples; failure modes; no secrets; links to evals if it changes code. Version skills like APIs — breaking changes bump major; keep changelog.

### 33.4 CLAUDE.md templates as IP

Lean sections: purpose, build/test commands, code style, security invariants, review checklist, ask humans when.

Variants: service repo, data/ML repo, infra repo, docs repo.

Do not put: API keys, customer PII, unverified rumors, contradictory absolutes better enforced in settings.

Maintenance: owners; review quarterly; tie to incidents.

### 33.5 Eval harness reuse

Harnesses are high-leverage IP: runner plus scoring plus report format; fixture layout conventions; safety slice always on; plug-in scorers (schema, task success, judge); cost/latency capture for Integration regressions. Governance: changes to harness scoring need version bumps so historical comparisons stay honest.

### 33.6 MCP and tool modules as accelerators

Ship: server stub with auth patterns; tool schemas plus deny list examples; contract tests; threat model paragraph; allowlist snippets for Claude Code settings. Trap: publishing a server that over-scopes OAuth scopes for convenience.

### 33.7 Documentation for handoff

Minimum pack: README (quickstart), SECURITY, OPERATIONS (pins, dashboards, rollback), EVALS, CODEOWNERS/support channel.

Anti-patterns: screenshot-only setup; Slack lore; undocumented breakages; contact Bob.

Audience split: adopter engineer, security reviewer, on-call.

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

### 33.9 Exam traps (packaging-flavored)

1. Shipping accelerators with embedded secrets.
2. No safety defaults (adopters will tighten later).
3. Claiming gated official curriculum as public.
4. Skills that silently change permissions.
5. Evals that only check tone.
6. MCP kits with allow-all samples.
7. Templates that contradict enforced settings.
8. No versioning — silent drift across teams.

### 33.10 Additional Q&A (Q31-Q45)

**Q31.** What is the first file a security reviewer opens in an accelerator? 
**A31.** SECURITY.md / threat model and the default allowlists — then schemas.

**Q32.** Why include a tiny golden eval in every agent kit? 
**A32.** Proves the pin bundle runs; catches adopter misconfig; anchors regressions.

**Q33.** When is a CLAUDE.md template harmful? 
**A33.** When it is so long or contradictory that teams ignore it, or when it pretends to enforce controls.

**Q34.** How do you distribute a skill safely? 
**A34.** Code review, no secrets, pinned versions, clear required permissions listed — not silent escalation.

**Q35.** Accelerator success metric? 
**A35.** Time-to-first-safe-deploy for adopters plus eval pass rate — not stars.

**Q36.** Why separate pin.defaults from pin.prod examples? 
**A36.** Defaults are safe starters; prod pins are environment-specific and must not be copied blindly across hosts.

**Q37.** What belongs in OPERATIONS.md? 
**A37.** Dashboards, alerts, rollback, rate limit contacts, host matrix.

**Q38.** Can Chapter 05 content appear on CCDV-F? 
**A38.** As Integration/Agents/Code/Eval judgment about reusable configs — not as partner IP process trivia. Study the engineering substance.

**Q39.** Hook packaged in a plugin deletes node_modules on every edit — problem? 
**A39.** Dangerous default; hooks must be least privilege and opt-in for destructive actions.

**Q40.** What is a good extension point? 
**A40.** Bring your own search_kb tool adapter rather than forking the whole agent.

**Q41.** How do you version prompts in a kit? 
**A41.** system.vN.md plus pin file reference plus changelog; never overwrite silently.

**Q42.** Why include a deny-list example for MCP? 
**A42.** Teaches adopters that connectors are not all-or-nothing.

**Q43.** Reference agent versus production service? 
**A43.** Reference teaches patterns; production adds tenancy, SLOs, compliance — do not confuse demos with prod.

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
| Partner packaging process | Engineering substance only; flag non-public process |

### 33.12 Glossary addendum

| Term | Meaning |
| --- | --- |
| Accelerator | Reusable, documented, evaluable package |
| Pin defaults | Starter versions for adopters |
| Extension point | Documented swap-out interface |
| Handoff pack | Docs set for another team |
| Safe default | Deny/least privilege out of the box |
| Smoke eval | Tiny critical golden slice |
| Reference agent | Teaching implementation, not prod tenant system |
| Contribution bar | Checklist before publishing internally |

### 33.13 Primary-study checklist (Chapter 05)

- [ ] I can list the minimum accelerator bar.
- [ ] I can choose CLAUDE.md versus skill versus hook versus reference app.
- [ ] I can sketch an agent kit directory without secrets.
- [ ] I can explain why evals ship with kits.
- [ ] I can write a one-page SECURITY section for an MCP kit.
- [ ] I know Chapter 05 may exceed public curriculum — I study transferable engineering.

### 33.14 Building an internal accelerator catalog

Merged into §28 (operating model) and §18 (roadmap phases). Roadmap theme recall: starter CLAUDE.md set; eval harness core; support-copilot agent kit; MCP SaaS connector kit; Claude Code monorepo settings template.

### 33.15 Quality rubric for contributions

Merged into §15 (canonical rubric with the 14/20 + zero-zeros-on-safety threshold).

### 33.16 Worked packaging examples

Merged into §14 (canonical detailed Examples A–D).

### 33.17 Closing — Chapter 05 as primary study

Packaging is how Integration quality multiplies. Whether or not official packaging courses are available to you, the public-doc-aligned skills — pins, safe defaults, evals, CLAUDE.md/skills/hooks, MCP kits — remain exam-relevant and job-relevant.

### 33.18 Documentation templates

Merged into §19 (canonical README / SECURITY / OPERATIONS / EVALS outlines).

### 33.19 Contribution readiness scorecard

Merged into §27 (canonical pass/fail scorecard).

### 33.20 Final reminder

Treat Chapter 05 as primary study for reusable Integration quality: pins, safe defaults, evals, and handoff docs. Re-check public Claude Code and MCP docs before exam day; gated packaging process details are out of scope for this pack.
