# 05 — Team Enablement & Operational Productivity — Simplified Technical English

**CCAR-P condensed domain 5 (dedicated)**
**Official domain mapped here:** Developer Productivity & Operational Enablement (**7%**)
**Approx. questions:** ~4 of 63. There are fewer items. The scenarios are concrete (org rollout, settings hierarchy, secure defaults).
**Cross-links:** Architecture patterns in `01`. MCP/tools in `02`. Safety baselines in `03`. Lifecycle/comms in `04`.

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names Claude Code, MCP, CLAUDE.md, Skills, and hooks stay as written. These notes are original enablement notes. They use themes from **public** Claude Code and Anthropic product documentation (project instructions, MCP, plugins/marketplaces, settings hierarchies, hooks). They also use standard enterprise developer-productivity practice. They are **not** official course content. Product UI names and setting keys change. Check current public Claude Code docs before a production rollout. This pack is independent. It is not affiliated with Anthropic.

---

## Disclaimer

These are original enablement notes. They use themes from **public** Claude Code and Anthropic product documentation. Those themes include project instructions, MCP, plugins/marketplaces, settings hierarchies, and hooks. They also use standard enterprise developer-productivity practice. They are **not** official course content. Product UI names and setting keys change. Check current public Claude Code docs before a production rollout. This pack is independent. It is not affiliated with Anthropic.

---

## Overview

The 7% domain has more impact than the percentage shows. Wrong answers are obvious if you know the **control plane** for teams. You roll out Claude Code (or similar builder tooling) with **enforced security**, **shared skills**, **standardized workflows**, and **operational runbooks**. You must not let every developer become an unsupervised admin of production credentials.

This file covers:

1. Team rollout strategy for Claude Code / AI coding assistants
2. Org instruction files (`CLAUDE.md` and related), Skills, hooks
3. Managed settings vs recommended settings vs local overrides
4. Plugin/marketplace distribution of approved tools
5. Productivity workflows (code, test, docs, debug)
6. Operational support patterns & runbooks
7. Training and onboarding curricula for builders

---

## Key map

| Enablement task | Exam signal |
| --- | --- |
| Enforce org security settings | Server-managed / highest-precedence controls that users cannot override |
| Distribute approved Skills/MCP/plugins | Managed marketplace / catalog—not “install anything” |
| Standardize repo guidance | CLAUDE.md / project instructions versioned in git |
| Raise velocity safely | AI for code/test/docs with review gates |
| Ops debugging with AI | Guardrailed assistants + runbooks—not unrestricted production keys |
| Train builders | Role-based onboarding + paved road |

---

## Part A — Rollout strategy

### A1. Why architects own enablement design

If platform and security teams do not make a paved road, teams adopt consumer AI. Those teams then paste secrets. Architects design:

- Identity (SSO) into approved tooling
- Data handling rules (what code/secrets you may send)
- Approved MCP/plugin set
- Secrets via vault—not chat
- Metrics for adoption **and** incidents

### A2. Phased rollout

| Phase | Population | Goal |
| --- | --- | --- |
| 0 Design | Platform + security | Settings, marketplace, logging |
| 1 Pilot | 1–2 volunteer teams | Friction log. Productivity signals |
| 2 Expand | All eng with training | Default secure config |
| 3 Optimize | Org-wide | Skills library. Inner source |
| 4 Continuous | — | Version upgrades. Access reviews |

Exit criteria per phase: you close security findings. The support queue stays manageable. You measure useful adoption (not seat count alone).

### A3. Success metrics for enablement

| Metric | Intent |
| --- | --- |
| Time-to-first-merged-PR with assistant | Onboarding efficacy |
| PR cycle time / review iterations | Velocity (check other causes) |
| Secret-scan incidents involving AI tools | Safety |
| % repos with CLAUDE.md | Standardization |
| % MCP from approved marketplace only | Supply-chain hygiene |
| Support tickets per 100 seats | Enablement quality |
| Developer satisfaction (survey) | UX of paved road |

---

## Part B — CLAUDE.md, Skills, and project instructions

### B1. What org instruction files are for

Public Claude Code docs describe project guidance files (commonly `CLAUDE.md` or equivalent project instructions). These files help the coding agent know:

- Repo structure & conventions
- Build/test commands
- Code style & review expectations
- Forbidden actions (e.g., do not touch `prod/` without a ticket)
- How to run linters/security scanners

**Architect guidance:** Treat these files as **versioned product code**. Review them in PRs. Keep secrets out. Link to deeper docs. Do not paste long documents (context budget).

### B2. Layered instructions

| Layer | Example | Scope |
| --- | --- | --- |
| Org default | Company coding standards Skill | All repos |
| Repo `CLAUDE.md` | Service-specific architecture | One repo |
| Directory hints | `/payments` stricter rules | Subtree |
| Personal | User preferences | Local only—must not weaken security |

Security-critical rules belong in **enforced settings**. Do not put them only in markdown. A model can under-weight markdown.

### B3. Skills for teams

Skills package reusable instructions/resources for recurring tasks: “add OpenTelemetry span,” “write ADR,” “migrate endpoint.” Benefits:

- Consistency across humans
- Smaller per-turn prompts
- Reviewable artifacts

Governance: Skill review owners. Semver. Deprecate. Security scan for exfil instructions.

### B4. Hooks

Hooks are automation around agent lifecycle events (formatters, secret scanners, policy checks). Hooks enforce **deterministic** controls where prompts are soft:

- Pre-commit / pre-push style checks triggered in agent loops
- Block commits containing secrets
- Auto-run formatters
- Require tests for certain paths

Exam theme: combine **soft guidance** (CLAUDE.md) with **hard hooks/settings**.

---

## Part C — Settings hierarchy & enforcement

### C1. Precedence mental model

Public Claude Code enterprise features describe a hierarchy. **Server-managed / organization-managed settings** sit at the highest precedence. Users **cannot override** them with project, user, local, or CLI flags. Lower layers give defaults and team conventions.

| Layer (conceptual) | Override by users? | Use for |
| --- | --- | --- |
| Server / org-managed | No | Security policies, denied tools, env constraints |
| Marketplace restrictions | No/limited | Approved plugin sources |
| Shared project recommendations | Yes (unless managed) | Conventions |
| User/local | Yes | Personal DX |

**Common exam error:** “Publish a recommended settings JSON in a repo. Also, ask teams to copy it.” This is not enough when the requirement is **enforce**. Prefer managed settings + managed marketplace.

### C2. What to enforce org-wide

- Disallowed MCP sources / unchecked plugin installs
- Telemetry/logging baselines required by security
- Environment variable policies
- Command allow/deny lists for agent shell
- Network egress constraints where supported
- Requirement to use SSO-backed installs

### C3. What to leave flexible

- Editor keybindings
- Non-security model preference within approved set
- Personal prompt snippets that do not grant tools

### C4. Managed plugin marketplace

Use a central catalog to distribute approved:

- Skills
- Agents
- Hooks
- MCP servers

The catalog supports versioning, discovery, updates, and **source control**. This is better than each developer using curl on random GitHub. Pair this with policy: only marketplace installs on corp devices.

---

## Part D — MCP & credentials for builders

### D1. Separation from production runtime

Builder environments must **not** reuse production write credentials. Give:

- Dev/stage credentials with narrow scopes
- Read-only prod replicas where justified
- Break-glass prod access via existing privileged access management—not Claude config files

### D2. Approved connector set

Start small: issue tracker, code host, internal docs search, CI status. Add SaaS MCPs only after vendor review (file 03). Document the data classes that each connector may send.

### D3. Secret hygiene training (mandatory)

- Never paste API keys into prompts
- Use vault + short-lived tokens
- Assume the system may retain transcripts per policy
- Redact in screenshots for support

---

## Part E — Developer productivity workflows

### E1. High-ROI assistant use cases

| Workflow | How AI helps | Guardrail |
| --- | --- | --- |
| Codebase onboarding | Explain modules, find examples | Cite files. Verify |
| Feature scaffolding | Boilerplate + tests | Human review. Architecture fit |
| Bug investigation | Trace logs ↔ code | No prod writes |
| Test generation | Edge cases | Still run in CI |
| PR description / changelog | Summarize diffs | Author accountable |
| Doc drift fixes | Update READMEs | Review for accuracy |
| Refactors | Mechanical transforms | Tests required |

### E2. Anti-patterns

- Accept large AI diffs without a read
- Skip tests because “the model said it passes”
- Let an agent force-push / rewrite git history unsafely
- Paste customer PII into prompts for debugging
- Generate crypto/auth code without expert review

### E3. Review culture upgrades

AI raises the **volume** of diffs. Raise review standards:

- Require tests for AI-touched auth/payment paths
- Ownership annotations
- Smaller PRs
- Threat-model checklist on tool/MCP changes

### E4. Measuring productivity without false metrics

Do not use “lines of code” vanity metrics. Prefer:

- Lead time for changes
- Change fail rate
- Perceived toil reduction surveys
- Onboarding time to first meaningful PR

Check team maturity and concurrent process changes.

---

## Part F — Operational enablement & runbooks

### F1. AI-assisted ops (safe pattern)

Use assistants to:

- Summarize alert storms
- Correlate logs (within policy)
- Draft incident timelines
- Suggest dashboard queries
- Draft postmortem outlines

Do **not** give unrestricted cloud admin to chat agents at the start.

### F2. Runbook template for AI features

For each production Claude feature (product runtime—not only IDE):

1. Symptoms
2. Dashboards
3. Feature flag / pin rollback
4. Distinguishing model vs retrieval vs tool failure
5. Safety incident fork → security IR
6. Customer comms pointer
7. Escalation graph

Enablement means on-call staff can execute. They do not page the original architect every time.

### F3. Support patterns for builder tooling

| Tier | Handler | Examples |
| --- | --- | --- |
| L1 | IT/helpdesk | Install, SSO, license |
| L2 | Platform AI enablement | Settings, MCP allowlist requests |
| L3 | Security + architect | Suspected exfil, malicious Skill |

Knowledge base: golden `CLAUDE.md` examples, FAQ, known failure modes, request form for new MCP.

### F4. Debugging productivity loops

Teach builders a standard loop:

1. Reproduce
2. Ask assistant with **minimal** sensitive context
3. Verify against tests
4. Document Skill if pattern recurs

---

## Part G — Training & onboarding curriculum

### G1. Role-based paths

| Role | Curriculum focus |
| --- | --- |
| IC developer | CLAUDE.md, prompt hygiene, PR review with AI, secrets |
| Tech lead | Skills authoring, hooks, metrics, team norms |
| SRE/ops | Runbooks, safe ops assistants, rollback |
| Security champion | MCP threat model, settings enforcement, audits |
| PM/EM | Lifecycle metrics, expectation management (file 04) |

### G2. Onboarding week outline

- Day 1: Account, SSO, policy acknowledgment
- Day 2: Tour paved road Skills. Complete lab in sandbox repo
- Day 3: Pair on real task with review checklist
- Day 4: Secret hygiene + injection awareness mini-module
- Day 5: Office hours. Feedback form

### G3. Labs (original ideas)

1. Add a Skill for “generate ADR from template”
2. Configure a deny-list hook for `.env` commits
3. Use MCP docs search to answer architecture Q with citations
4. Deliberately break tenancy in a toy RAG and fix filters (with `02`)
5. Practice rollback of a prompt pin in staging

### G4. Champions network

Volunteer champions per org unit reduce L2 load. They meet biweekly to share Skills and incidents. Architects start the first Skill library.

---

## Part H — Governance of enablement itself

- Quarterly access review of who can publish marketplace items
- Changelog for org-managed settings
- Track exceptions (teams with temporary elevated tools)
- Align with SDLC policies in file 03
- Retire unused Skills to reduce attack surface

---

## Decision trees

### Enforce vs recommend

```
Could a developer override cause data leak or prod damage?
 YES → Org-managed enforced setting / hook / deny tool
 NO → Recommended project settings / CLAUDE.md guidance
```

### New MCP request

```
Vendor review + least scopes + stage credentials + Skill docs + owner?
 ALL YES → Marketplace publish
 ELSE → Reject or sandbox spike only
```

---

## Exam traps

1. Recommended repo settings as a substitute for org-enforced policy.
2. Allowing arbitrary plugin installs then “audit later.”
3. Production admin credentials in IDE MCP configs.
4. Long CLAUDE.md files full of secrets.
5. Measuring only seat adoption, not safety or outcomes.
6. AI ops bot with unrestricted cloud IAM.
7. No support tier / knowledge base—architect as permanent L1.
8. Skills without owners or review.
9. Training only on prompts, never on hooks/settings.
10. Agents amplify a force-push culture.

---

## Practice Q&A (28)

**Q1.** How do you enforce settings developers cannot override?
**A.** Use organization/server-managed settings at highest precedence.

**Q2.** Why is “put recommended settings in git” insufficient for mandatory security?
**A.** Teams can ignore or change them. Managed settings cannot be overridden.

**Q3.** What should a managed marketplace distribute?
**A.** Approved Skills, hooks, agents, MCP servers (versioned).

**Q4.** Where do repo-specific build commands belong?
**A.** Versioned project instructions (e.g., `CLAUDE.md`), reviewed in PRs.

**Q5.** Name two hard controls beyond markdown guidance.
**A.** Hooks (secret scan/format) and org-managed deny lists / tool policies.

**Q6.** Why separate builder credentials from prod write keys?
**A.** Limit blast radius of prompt injection or mistaken agent actions.

**Q7.** Good enablement metric beyond seats?
**A.** Time-to-first-merged-PR, secret incidents, % approved MCP only, CSAT.

**Q8.** What is a Skill useful for in teams?
**A.** Packaging recurring workflows for consistency and reuse.

**Q9.** Who should publish to the org marketplace?
**A.** Controlled publishers with review—not every developer by default.

**Q10.** AI-generated auth code requires what extra step?
**A.** Expert human review (+ tests). Do not trust the output without a check.

**Q11.** L1 vs L2 support for Claude Code?
**A.** L1 install/SSO. L2 platform settings/MCP allowlist requests.

**Q12.** What belongs in an AI feature on-call runbook?
**A.** Rollback pins/flags, dashboards, failure differentiation, escalation.

**Q13.** Why teach injection awareness to developers using MCP?
**A.** Untrusted content can steer agents that hold tools.

**Q14.** Vanity productivity metric to avoid?
**A.** Raw lines of code generated.

**Q15.** What is a champions network for?
**A.** Federated enablement, Skill sharing, reduced central ticket load.

**Q16.** Can personal settings weaken org security policy?
**A.** They must not. If you manage settings correctly, org policy wins.

**Q17.** First step when a team requests a new SaaS MCP?
**A.** Security/vendor review + scope minimization (not instant install).

**Q18.** How do hooks complement CLAUDE.md?
**A.** Hooks enforce deterministic checks. Markdown is advisory context.

**Q19.** What training do tech leads need beyond ICs?
**A.** Skills authoring, hooks, team norms, metrics.

**Q20.** Why redact customer PII before you debug with AI?
**A.** Privacy/policy compliance. You reduce retention and leak risk.

**Q21.** Break-glass prod access should go through what?
**A.** Existing PAM/privileged workflows—not long-lived keys in agent config.

**Q22.** What does % repos with CLAUDE.md show?
**A.** Standardization of agent guidance (leading enablement indicator).

**Q23.** Risk of unowned Skills?
**A.** Drift, abandonment, potential malicious stale instructions.

**Q24.** How should AI affect PR size norms?
**A.** Keep PRs reviewable. Do not merge huge unread AI changes.

**Q25.** Office hours purpose in onboarding?
**A.** Unblock pilots. Gather friction for paved-road fixes.

**Q26.** What changelog should platform keep?
**A.** Org-managed settings and marketplace item versions.

**Q27.** Is this domain only about IDE tooling?
**A.** Primarily builder productivity + ops enablement. Product runtime ops runbooks also matter.

**Q28.** Select TWO for org-wide enforcement of approved tools: managed marketplace + server-managed settings—why both?
**A.** Marketplace distributes capabilities. Managed settings enforce policy/source restrictions.

---

## Checklist

- [ ] I can explain enforced vs recommended settings
- [ ] I can design a phased Claude Code rollout with exit criteria
- [ ] I can outline CLAUDE.md + Skills + hooks together
- [ ] I can specify marketplace governance
- [ ] I can separate builder vs prod credentials
- [ ] I can name enablement metrics beyond adoption seats
- [ ] I can draft L1/L2/L3 support boundaries
- [ ] I can write a rollback-oriented runbook section
- [ ] I can sketch a 5-day onboarding plan
- [ ] I can answer “Select TWO” enforcement questions confidently

---

## Glossary

| Term | Meaning |
| --- | --- |
| CLAUDE.md | Project instruction file for Claude Code-style agents |
| Skill | Reusable instruction/resource pack for recurring tasks |
| Hook | Deterministic automation around agent/dev lifecycle events |
| Managed settings | Org-level enforced configuration (highest precedence) |
| Marketplace | Catalog for distributing approved plugins/Skills/MCP |
| Paved road | Supported standard path for teams |
| PAM | Privileged access management |
| Enablement metrics | Measures of safe productivity adoption |
| Champions network | Federated volunteer experts per team |
| Break-glass | Emergency privileged access with audit |
| Inner source | Internal open-source style sharing of Skills/tools |
| Change fail rate | DORA-style metric of deployment failures |
| Lead time | Time from commit to production |
| Telemetry baseline | Required logging/analytics for security/ops |
| Sandbox repo | Safe training environment without prod data |

---

## Part I — Deep dive: designing the paved road

### I1. Reference architecture for corp Claude Code

```
IdP (SSO) → Claude Code enterprise → Org-managed settings
 ↓
 Managed marketplace
 (Skills, hooks, MCP)
 ↓
 Dev/stage credentials (vault)
 ↓
 Optional: AI gateway for API use
```

Logging sinks to SIEM per security baseline. You track exceptions in ticketing.

### I2. Sample org policies (illustrative)

1. Only marketplace MCP servers on managed devices.
2. No production write scopes in developer agents.
3. All repos owning services must include `CLAUDE.md` with test commands.
4. Skills that touch auth/payment require security co-review.
5. Agent shell deny: `terraform apply` in prod dirs without ticket ID env.

### I3. Migration from shadow ChatGPT usage

- Provide equivalent convenience (docs MCP, issue MCP)
- Clear data-handling comparison sheet
- Amnesty window to rotate pasted secrets
- Monitor outbound to unsanctioned AI domains if DLP supports

### I4. Multi-repo monorepo notes

Monorepos need directory-scoped guidance. Hooks must fire only on touched paths for performance. Shared Skills for language ecosystems (Go/Java/TS) reduce duplication.

### I5. Compliance evidence from enablement

Auditors may ask how you control AI coding tools. Keep: settings screenshots/exports, marketplace ACL lists, training completion records, exception register, incident tickets.

### I6. Extended scenarios

**Scenario:** Org must prevent developers from overriding security settings. The org must also distribute approved workflows.
**Answer shape:** Server-managed settings + managed plugin marketplace (common Select TWO). Not periodic audits of local installs alone. Not repo-only recommendations.

**Scenario:** SRE wants Claude to auto-remediate kube pods in prod.
**Answer shape:** Start with read-only diagnose + suggested kubectl. Use human approval. Later, use narrow automations with PAM. Never put a broad admin kubeconfig in the IDE.

**Scenario:** New hire floods support with “how do I connect Jira MCP.”
**Answer shape:** Improve onboarding lab + L1 KB article + champions. Do not custom-configure each laptop manually as a permanent process.

### I7. Q&A 29–40

**Q29.** Draw the control plane pieces for corp Claude Code.
**A.** SSO, org-managed settings, marketplace, vault credentials, logging/SIEM.

**Q30.** Why directory-scoped hooks in monorepos?
**A.** Performance and relevance—do not run unrelated policy on every edit.

**Q31.** Amnesty window means?
**A.** Period for rotating secrets previously pasted into unsanctioned tools. Do not punish people in this period.

**Q32.** Select TWO common enforcement actions?
**A.** Managed settings + managed marketplace (per common practice-exam pattern).

**Q33.** Why co-review Skills for payments?
**A.** Higher abuse impact. Instructions could coax unsafe code patterns.

**Q34.** What evidence shows training worked?
**A.** Lab completion, fewer L1 tickets, fewer secret incidents, survey confidence.

**Q35.** Auto-remediate prod—first architecture?
**A.** Suggest-only with human approve. PAM. Narrow later.

**Q36.** What belongs in an exception register?
**A.** Team, elevated tool, expiry, approver, compensating controls.

**Q37.** How does an AI gateway help builders calling Messages API?
**A.** Central keys, quotas, redaction, attribution—aligned with file 02.

**Q38.** Why not measure success only by plugin install counts?
**A.** Installs ≠ safe or valuable use. Track outcomes and incidents.

**Q39.** What is inner source of Skills?
**A.** Teams contribute reusable Skills via review to a shared catalog.

**Q40.** Who is accountable for org-managed settings changes?
**A.** Platform/security owners with change tickets. Do not use ad-hoc admin clicks that you do not document.

---

## Extended checklist

- [ ] I can sketch corp Claude Code control plane
- [ ] I can write five illustrative org AI-coding policies
- [ ] I can plan shadow-IT migration with amnesty
- [ ] I can answer Select-TWO enforcement questions
- [ ] I can design suggest-only SRE automation first
- [ ] I can list auditor evidence for AI coding controls

---

## Glossary additions

| Term | Meaning |
| --- | --- |
| Control plane | Settings/marketplace/IAM governing assistants |
| Exception register | Tracked temporary policy deviations |
| Amnesty window | Coordinated secret rotation period |
| Directory-scoped | Rules applying to repo subtrees |
| Suggest-only | Assistant proposes. Human executes |
| SIEM sink | Security log destination |
| Co-review | Dual ownership review (e.g., eng+security) |
| DORA metrics | Elite engineering performance metrics set |
| KB | Knowledge base for support |
| Federated enablement | Central standards + local champions |

---



---

## Part J — Enablement exam drill & labs detail

### J1. “Select TWO” pattern family

Whenever the stem says *enforce* + *distribute approved tools*, look for:

- Org/server-managed settings
- Managed marketplace / approved catalog

Distractors: periodic audits, repo-only recommendations, project-level optional configs, unconstrained installs.

### J2. Lab write-up: secrets hook

Goal: agent cannot commit `.env`.
Steps: add a hook that scans staged files. Deny on pattern. Document in CLAUDE.md as a soft warning too. Prove with an attempted commit in sandbox. Record an evidence screenshot for the training KB.

### J3. Lab write-up: Skill for ADR

Skill provides ADR template + “ask for context/decision/alternatives/consequences”. It outputs markdown path `/docs/adr/XXXX.md`. CI checks ADR filename pattern on certain PRs.

### J4. Productivity appearance vs real gains

If cycle time drops but change-fail rate spikes, AI may accelerate bad changes. Tighten review on sensitive paths. Enablement owners watch **joint** metrics.

### J5. Q&A 41–48

**Q41.** Common distractor for enforcement questions?
**A.** “Audit local configs periodically” but you do not prevent bad installs.

**Q42.** What twin controls appear in Select TWO org rollout stems?
**A.** Managed settings + managed marketplace.

**Q43.** Why track change-fail rate with AI adoption?
**A.** Velocity without quality may increase incidents.

**Q44.** ADR Skill should produce what artifact property?
**A.** Versioned markdown under a known docs path with required sections.

**Q45.** Soft warning in CLAUDE.md vs hook—difference?
**A.** Warning is advisory. A hook can block deterministically.

**Q46.** Who maintains the training KB articles for Jira MCP?
**A.** Enablement/platform L2—not each squad reinventing.

**Q47.** Sensitive path review policy example?
**A.** Extra human review (+ tests) for auth/payments touched by AI diffs.

**Q48.** What evidence closes a secrets-hook lab?
**A.** Documented failed commit attempt + hook config reviewed into the paved road.

*This completes the five-file user domain set. Return to README for the official-7 mapping.*


## Part K — Rapid review (Enablement 7%)

Even with ~4 questions, missed points still cost you. Drill:

1. **Enforce** → org/server-managed settings (not git recommendations alone).
2. **Distribute approved tools** → managed marketplace.
3. **CLAUDE.md** → versioned guidance. **hooks** → hard checks.
4. **No prod write keys** in builder MCP.
5. **Metrics** beyond seats. Watch fail rate with velocity.
6. **Support tiers** + KB + champions.
7. **Training** role-based with labs and secret hygiene.
8. **Select TWO** stems often pair managed settings + marketplace.

If a scenario says “developers must not override,” remove any answer that relies on goodwill or periodic audits as the primary control.


## Part L — Sample paved-road checklist (copy/adapt)

- [ ] SSO enforced for Claude Code seats
- [ ] Org-managed settings documented & change-controlled
- [ ] Marketplace ACL: publishers listed. Consumers read-only install
- [ ] Default deny for non-marketplace MCP
- [ ] Vault pattern doc for dev credentials
- [ ] Golden CLAUDE.md template in platform repo
- [ ] Secret-scan hook enabled by default
- [ ] Onboarding lab in sandbox repo
- [ ] L1/L2 runbooks published
- [ ] Quarterly access review scheduled
- [ ] Exception register exists with expiry
- [ ] Metrics dashboard: adoption, incidents, cycle time, fail rate

This completed list is a stronger “enablement architecture” than any single prompt tip. When you are not sure on CCAR-P, select the answer that builds this control plane.


*(File continues — deep-dive Part M follows below.)*


---

## Part M — Primary-study deep dive: Paved-road enablement

Domain weight is only **7%**, but items are concrete: settings precedence, secure defaults, CLAUDE.md/Skills/hooks, approved MCP/plugins, and ops debugging patterns. Wrong answers usually violate **least privilege**. They also confuse **builder tooling** with **production runtime**.

### M1. Control-plane mental model

```
Org managed settings (non-overridable security)
 → Team / enterprise defaults
 → Repo CLAUDE.md + Skills (versioned)
 → User local settings (where allowed)
 → Session prompts
```

**Exam rule:** Security-relevant controls belong at the **highest precedence that users cannot bypass**. You can recommend productivity niceties as defaults.

### M2. What to enforce vs recommend (expanded)

| Control | Enforce org-wide | Recommend |
| --- | --- | --- |
| Secret exfiltration guards / hooks | Yes | — |
| Disallow unapproved MCP/marketplaces | Yes | — |
| Block pasting `.env` / keys patterns | Yes | — |
| Require SSO | Yes | — |
| Default model for interactive | Often | Allow power users if policy OK |
| Commit message style Skill | — | Yes |
| Aggressive auto-run of shell | Deny by default | Enable per trusted repo carefully |
| Telemetry / audit of tool use | Yes where lawful | — |

### M3. CLAUDE.md design guide

Good org/repo instruction files:
- Architecture map and service boundaries
- Test commands and “definition of done”
- Security never-dos (prod creds, raw PII)
- How to propose MCP/Skill additions
- Links to runbooks—not secrets

Bad files:
- Entire style guides pasted twice
- Credentials
- Contradictions vs org managed policy
- Unmaintained guidance from 18 months ago

**Layering:** Org baseline → repo specifics → personal preferences (if allowed). Test precedence with a deliberate conflict in staging.

### M4. Skills for teams

| Skill type | Example | Governance |
| --- | --- | --- |
| ADR writer | Structured decision record | Reviewed like docs |
| Test generator | Framework-specific | No prod network tools |
| Threat-model stub | STRIDE prompts | Security co-owned |
| Migrate-langX | Internal frameworks | Versioned. Owners |

Skills that fetch remote content inherit **prompt-injection** risk. Treat them like MCP.

### M5. Hooks that give real value

High-ROI hooks (illustrative patterns from public product themes):
- Pre-tool: block commands matching secret/exfil patterns
- Pre-commit assistant: remind tests
- Stop: summarize session for handoff notes

Do not add many hooks that slow every keystroke without security value.

### M6. MCP for builders ≠ MCP for production agents

| Dimension | Builder (Claude Code) | Production agent |
| --- | --- | --- |
| Creds | Dev/sandbox roles | Scoped service roles |
| Blast radius | Repo / dev accounts | Customer data / money |
| Change speed | Fast inner loop | Change-controlled |
| Approval | Marketplace allowlist | Architecture + security review |

Never reuse unrestricted production credentials in developer MCP “for convenience.”

---

## Part N — Productivity workflows that pass review

### N1. High-ROI loops (with review gates)

1. **Explain → propose → patch → test → PR summary** with human review.
2. **Failing test → root cause hypotheses → minimal fix.**
3. **Legacy module → characterization tests before refactor.**
4. **Incident stub → timeline draft from logs (redacted) → human edit.**

### N2. Anti-patterns

| Anti-pattern | Why bad | Replace with |
| --- | --- | --- |
| Accept all diffs without a read | Ships vulns/secrets | Mandatory review culture |
| Paste prod dumps into chat | Data leakage | Redacted fixtures |
| Metric = lines generated | False signal | Cycle time and change fail rate |
| Untracked Skills with tools | Supply chain | Managed marketplace |
| One shared API key on laptop | Attribution/abuse | SSO + per-user |

### N3. Measuring productivity with true metrics

Prefer: PR cycle time, review iterations, escaped defects, secret-scan incidents, developer satisfaction, time-to-first-productive-PR for new hires.
Distrust: raw acceptance rate, token spend as “value,” lines of AI code.

### N4. Debugging & operational enablement

Safe pattern: AI assists on **read-only** diagnostics. Remediations go through existing change systems. Provide runbooks the assistant can retrieve. Do not grant cloud-admin MCP to every engineer’s laptop agent.

Runbook sections: symptoms → gather (commands) → hypothesize → mitigate → verify → escalate → postmortem links.

---

## Part O — Training curriculum (original outline)

### O1. Role paths

| Role | Must learn |
| --- | --- |
| IC engineer | CLAUDE.md, safe MCP, PR review with AI diffs |
| Tech lead | Skills governance, eval mindset for assistants |
| SRE/ops | Guardrailed debug loops, no unrestricted production keys |
| Security | Hook policies, marketplace review, DLP |
| PM/adjacent | What AI can/cannot promise. How to file good bugs |

### O2. Onboarding week (example)

Day 1: SSO + policy + secret hygiene lab.
Day 2: Repo CLAUDE.md tour + first assisted PR.
Day 3: MCP approved set only. Request process.
Day 4: Incident tabletop with AI assist (read-only).
Day 5: Office hours. Feedback into paved road backlog.

### O3. Champions network

Volunteers per org unit. Office hours. Inner-source Skills. Escalate friction to platform—not shadow SaaS.

---

## Part P — Architecture decision scenarios (Enablement)

### P1. Shadow consumer ChatGPT usage discovered

**Response:** Do not only ban. **Make a paved road**: SSO Claude tooling, clear data rules, migrate use cases, measure adoption, security exceptions time-boxed.

### P2. Team wants “install any MCP from the internet”

**Response:** Managed marketplace / allowlist. Threat review. Network egress controls. Owners. Versions pinned.

### P3. Conflict: repo CLAUDE.md says auto-run shell. Org policy forbids

**Response:** Org managed settings win. Fix the repo file. Educate. Alert on violations.

### P4. Vanity-metrics dashboard

**Response:** Replace lines-generated with quality/velocity/safety metrics. Use qualitative interviews.

---

## Part Q — Failure-mode analysis (enablement)

| Failure | Signal | Mitigate |
| --- | --- | --- |
| Settings bypass | Local overrides weaken DLP | Enforce managed non-overridable controls |
| Secret in PR via AI | Scan hits | Hooks + training + vault |
| Skill supply-chain | Unexpected network tool | Signing/review. Marketplace only |
| Support overload | Tickets per seat ↑ | Better docs. Champions. UX fixes |
| Repo instruction drift | Contradictory guidance | Owners + CI lint for CLAUDE.md |
| Prod creds in dev MCP | Audit findings | Separate IdP apps / roles |

---

## Part R — Production checklists

### R1. Before org pilot

- [ ] Managed security settings defined
- [ ] SSO live
- [ ] Marketplace policy live
- [ ] Secret hygiene training mandatory
- [ ] Support channel + SLA
- [ ] Success metrics agreed (incl. safety)

### R2. Before org-wide expand

- [ ] Pilot friction log closed or accepted
- [ ] Reference CLAUDE.md templates published
- [ ] Hooks validated (false positive rate OK)
- [ ] Access review process scheduled
- [ ] Incident tabletop done

### R3. Quarterly

- [ ] Plugin/MCP inventory review
- [ ] Settings drift audit
- [ ] Curriculum update for new model/tooling
- [ ] Productivity + safety metrics review

---

## Part S — Trade-off tables

### S1. Speed of enablement vs control

| Approach | Speed | Control | When |
| --- | --- | --- | --- |
| Uncontrolled local tools | Fast | Poor | Never for enterprise |
| Recommended defaults only | Medium | Medium | Low-risk orgs |
| Managed enforce + paved Skills | Medium | High | Default CCAR-P answer |
| Lockdown with no paved road | Slow adoption | High on paper | Creates shadow IT |

### S2. Central platform vs federated team Skills

Centralize security and core Skills. Federate domain Skills with review. Pure central work becomes a bottleneck. Pure federated work becomes chaos. Hybrid wins.

---

## Part T — Extended Q&A (49–60)

**Q49.** Highest-precedence place for “deny unapproved MCP”?
**A.** **Org managed / non-overridable** settings—not a hopeful CLAUDE.md line alone.

**Q50.** Select TWO enablement success metrics: secret-scan incidents, seat count only, time-to-first-merged-PR, raw tokens burned.
**A.** Secret-scan incidents and time-to-first-merged-PR.

**Q51.** Builder MCP with prod database admin—acceptable?
**A.** **No**—use sandbox/read-only roles. Prod changes go via controlled runtime.

**Q52.** Hooks vs Skills—security blocking of exfil commands?
**A.** Prefer **hooks/enforced controls**. Skills are instructions, not hard enforcement.

**Q53.** Shadow AI usage—best architect move?
**A.** Paved road + policy + migrate, not ban-only.

**Q54.** CLAUDE.md contains API keys—action?
**A.** Rotate keys, purge history, teach vault. Add secret scanning.

**Q55.** Select THREE rollout phases: design controls, pilot, expand, skip security until GA, email passwords to team.
**A.** Design, pilot, expand.

**Q56.** Why separate production agent creds from Claude Code MCP?
**A.** Different blast radius, change control, and attribution needs.

**Q57.** Productivity ineffective activity example?
**A.** You celebrate lines of AI-generated code while change-fail rate rises.

**Q58.** Managed plugin marketplace purpose?
**A.** Distribute **approved** tools/Skills. Reduce supply-chain risk.

**Q59.** Ops wants AI to “just fix prod.” Safe pattern?
**A.** Read-only diagnosis + human/change-managed remediation.

**Q60.** Repo instruction conflicts with org deny-list—who wins?
**A.** **Org enforced settings**. Fix the repo file.

---

## Part U — Rapid review (Enablement 7%)

- Enforce security high in the settings hierarchy.
- CLAUDE.md/Skills versioned. No secrets.
- Managed marketplace for MCP/plugins.
- Builder tooling ≠ production runtime creds.
- Measure velocity + safety, not vanity.
- Hooks for hard stops. Skills for guidance.
- Paved road beats shadow AI.
- Train by role. Champions sustain adoption.

*Cross-link: MCP deep dive `02`. Safety `03`. Rollout comms `04`.*
