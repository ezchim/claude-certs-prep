---
title: Domain 05 — Configuration & Knowledge Management — Simplified Technical English
pack: STE edition (ASD-STE100)
updated: 2026-08-30
---

# Domain 05 — Configuration & Knowledge Management
## Maps to official CCAO-F **Configuration and Knowledge Management** (~12%, ~7 questions)

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, MCP, Projects, RAG, Memory) are exceptions and stay as written. This is one primary-study copy. Earlier builds repeated the same drill blocks about 7 times. Those duplicates are gone. The content now matches the Domain 03 depth. Check Plan facts against support.claude.com. This edition adds a full **Memory** section. Memory is a named exam-guide topic. Coverage was near zero before.

## Disclaimer

These notes are original CCAO-F study notes. They are for people who are not developers. They use claude.ai (Projects, Artifacts, Skills, Connectors, Research). The notes use public Anthropic Help Center and product docs. They also use public Claude Academy (Claude 101, AI Fluency 4D) and the published CCAO-F blueprint. The notes are independent. They are not affiliated with Anthropic. Check live product details on support.claude.com.

---

## Overview

Configure Claude so that recurring work stays consistent. Official blueprint verbs: **configure** Claude Projects with instructions and knowledge sources. **Manage** uploaded knowledge and connectors (the guide names Google Drive and Gmail). **Create** effective system-level instructions. **Inform, maintain, and update** configurations, knowledge sources, and instructions over time. This domain helps you make the setup reliable for the team. Memory is now part of this domain. You also manage what Claude remembers.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Configure Projects | configure, set up | Instructions + knowledge + sharing + ownership |
| Manage knowledge & connectors | manage, curate, authorize | Upload vs connect. Hygiene. Least privilege |
| Effective instructions | create, structure | Short, prioritized, conflict rules, schemas |
| Maintain & update | maintain, refresh, retire | Cadence, changelogs, archiving, Memory curation |

---

## Deep notes

### 1. Project setup that survives contact with a team

Checklist: purpose. Owner. Sensitivity label. Instructions. Knowledge set. Sharing permissions. Model-default guidance. Evaluation notes.

Verified plan facts (support.claude.com, Aug 2026): Projects exist on **all plans**. **Free** caps at five projects. **Enhanced knowledge via RAG is paid-only**. RAG expands capacity up to about **10×**. RAG starts when knowledge approaches context limits. **Sharing is on Team and Enterprise**. Read "Can view" -only. "Can edit" lets you change knowledge and instructions. You grant access to one person, to a group, or to the whole org.

### 2. Instruction engineering

Good instructions specify role, audience defaults, output schemas, and citation rules. Add escalation, conflict priority, and file pointers by name.

Bad instructions tell the company story. They contradict each other. They hide critical rules.

Keep instructions **short, stable, and prioritized**. Instructions compete with task tokens. Put bulky policy in knowledge files.

Test instructions with three golden tasks and three adversarial tasks. In the adversarial tasks, ask for answers that are not in knowledge. Check that Claude answers "Not found". Update the instructions when failures cluster.

**Styles/preferences vs Project instructions:** Styles hold global personal taste. Project instructions hold workspace rules. A Style cannot carry a Project's legal constraints.

**Skills vs instructions:** Skills hold procedures. Instructions hold standing rules. Do not paste whole Skills into instructions.

### 3. Knowledge hygiene playbook

- **Naming:** Use `Policy-Refunds-v4-2026-06.pdf`. Do not use `finalFINAL.pdf`. Put version dates in filenames. Ask by filename.
- **Curation:** Remove superseded docs on the day the new version arrives. A stale file is a silent production bug. The answer agrees with the stale file but disagrees with current policy. Do not keep duplicate or conflicting versions. Retrieval confusion is a problem you create.
- **Structure:** Split unrelated domains (legal / HR / marketing) into separate Projects. Keep a manifest note. The note lists each document and its purpose.
- **Quality:** Prefer files that yield extractable text. Scans with unreadable text make retrieval fail.
- **RAG behavior:** When knowledge is large, expect search-like retrieval. Ask with a narrow question. Name files.

### 4. Connectors: upload vs connect, and least privilege

Upload = a frozen snapshot. You must refresh it. Connector = live data with auth and permission complexity.

Select with intent. Living documents that change weekly → connector. Stable policy of record → curated upload with version discipline.

Governance basics (Domain 06 covers this topic in depth): use least privilege. Prefer **read-only** for analysis. Write scopes are a governance decision. Write scopes need human gates. Team/Enterprise pattern: **admin enables, each user authenticates**. Document which connectors a Project expects. Then "Claude cannot see Drive" is an auth problem. It is not a model failure. **Custom connectors (remote MCP) exist on every plan. Free is limited to one custom connector** (verified Aug 2026).

### 5. Memory — configuring what Claude remembers (added. Named in the exam guide)

Facts verified against support.claude.com (Aug 2026 — this feature evolves. Re-check before exam day):

- **What it is:** Claude builds **memory entries** from your chats. The entries hold professional context, role, preferences, and ongoing work. Claude organizes them into categories. Claude applies them to later conversations. This replaces the old mental model "chats are ephemeral. Nothing persists between threads."
- **Availability:** Memory generation exists on **Free, Pro, Max** (newer experience) and **Enterprise** (legacy 24-hour synthesis experience). **Chat search** (Claude retrieves your past chats) is **Pro and above**. Team plans have **no org-level memory controls**. Members manage their own memory.
- **Project scoping:** Every Project has its **own separate memory** and summary. This keeps client work, confidential threads, and general chats apart. This is the configuration answer for keeping contexts separate.
- **Controls (this domain's core):** Settings → Memory: **view** entries, **edit**, **delete individual entries**, or **reset all**. Memory and search **toggle independently**. Citations show which chat a remembered fact came from.
- **Incognito chats:** Ghost icon. All plans. Outside Projects. Excluded from history, memory, and search. (Enterprise retains incognito chats a minimum period for safety and includes them in data exports — this is a governance detail.)
- Include **Portability:** Memory in data exports. Experimental import/export can move memory between assistants.

**Management doctrine:** Memory is a *personal context layer*. It is not a knowledge base. Curate it like knowledge. Review entries on a schedule. Delete wrong or stale entries. A bad memory entry is configuration drift. You carry that drift into every chat. Disable memory or use incognito for sensitive matters. Keep important team facts in **Project knowledge**. Curated files with owners beat inferred per-user memories. When Claude "remembers wrong," edit or delete the entry. Do not select a bigger model (Domain 07 bridge).

### 6. Maintenance, lifecycle, ownership

Every shared Project needs a named owner. That owner is accountable for updates. Without an owner, configuration drift is guaranteed.

Cadence: review instructions monthly. Refresh knowledge when policies change. Replace the file. Announce the change. Update a version note in the instructions. Keep a changelog in knowledge for shared Projects. Archive dead Projects. On Free, the five-project cap forces this.

Onboarding teammates: share the Project. Explain instructions. Show the evaluation checklist. Confirm connector auth. Supervise a first task.

### 7. Failure modes that look like model bugs

Claude "ignores policy." A superseded PDF is still in knowledge.

Claude "mixes brands." Two style guides exist. There is no priority rule.

Claude "forgot our project context." The user relied on personal Memory instead of Project knowledge. Or the memory entry was deleted. Or the memory is scoped to another Project.

Claude "cannot see Drive." The connector is not authenticated.

Claude "remembers something wrong about me." The memory entry is stale. Edit or delete it.

Fix configuration before you escalate models.

### 8. Mini-cases (configuration audits)

**Case 1 — The mystery brand voice.** Marketing's shared Project alternates between formal copy and short, direct copy. Audit finds `Brand-Voice-2025.pdf` and `Voice-Refresh-2026.docx` both in knowledge. There is no priority rule. There is no owner. Fix: delete the 2025 file. Add "voice per `Voice-Refresh-2026.docx`" to instructions. Name an owner. Start a changelog. One audit fixes every user.

**Case 2 — The consultant's bleed risk.** A solo consultant serves two competing clients from general chat. The consultant worries that context leaks. Fix: one Project per client (memory is project-scoped. Knowledge is separated). Keep general Memory for personal preferences only. Review memory entries on a schedule. Use incognito for exploratory conflict-sensitive questions.

**Case 3 — The Friday-report Project that rotted.** Built in March. Praised in April. Wrong by August. Policy v5 shipped. v3 is still in knowledge. The builder left. Nobody owns it. This is common drift. Fix: assign an owner. Replace the file. Add a version note to instructions. Put a monthly review on the calendar. Prevention costs less than the bad report that showed the problem.

**Case 4 — Onboarding that failed.** A new analyst gets Project access. Their first outputs cite nothing from Drive. Audit: the connector is admin-enabled. The analyst never authenticated. Also they used their personal account, not the workspace. Fix: workspace seat. Per-user auth walkthrough. Supervised first task against the evaluation checklist.

### 9. Multiple-response pattern bank

Remove these recurring wrong options:

- "Paste the whole policy into instructions" (bloat — that is knowledge's job).
- "Keep the old file for reference" (superseded = delete or archive with a clear label).
- "Memory will keep the team aligned" (per-user, not shared).
- "Grant edit to everyone so nobody is blocked" (this is a permission failure).
- "Re-upload weekly instead of connecting" when the doc is living (or its mirror: "connect everything," including stable policy).

Correct combinations pair a **placement** choice (instructions vs knowledge vs Skill vs connector vs Memory) with a **lifecycle** choice (owner, version note, cadence, archive).

---

## Decision trees

| Situation | Action |
|---|---|
| Recurring team context | Project (instructions + knowledge + owner) |
| Bulky reference material | Knowledge file, named and versioned — not instructions |
| Live, frequently changing docs | Connector (read-only unless writes are governed) |
| Policy updated | Replace file + announce + version note |
| Personal cross-chat preferences | Memory — review/edit entries in Settings |
| Confidential single-use chat | Incognito (excluded from memory/history/search) |
| Contexts must not bleed | Separate Projects (memory is project-scoped) |
| Free near five-project cap | Archive or merge |
| Wrong remembered fact | Edit/delete the memory entry |
| Shared Project misbehaving | Check owner, duplicates, staleness before model blame |

---

## Exam traps

1. Everything crammed into instructions (novel-length, self-contradicting)
2. Never updating knowledge. Superseded PDFs left in place
3. Treating personal Memory as the team's knowledge base
4. Assuming memory crosses Project boundaries (it is scoped per Project)
5. Write-scope connectors by default for analysis-only work
6. Ignoring sharing permissions (org-wide edit on restricted knowledge)
7. Duplicate conflicting files causing retrieval confusion
8. Shared Project with no owner (guaranteed drift)
9. Styles doing a Project's legal-compliance job
10. Forgetting incognito exists for sensitive one-offs

---

## Practice Q&A (18) — scenario stems with answers and rationales

**Q1.** A team needs standing tone rules plus a 40-page policy PDF applied to weekly work. Configuration?
**A:** Project: short tone rules in instructions. The PDF in knowledge, named with version and date. Instructions point to it by filename.

**Q2.** A Free-plan consultant hits the project cap when the consultant starts client #6. Options?
**A:** Archive or merge an inactive project, or upgrade. Free caps at five (verified Aug 2026. Re-check current caps at support.claude.com).

**Q3.** Select TWO knowledge-hygiene practices.
**A:** Version dates in filenames. Removing superseded documents immediately. (Duplicates and `finalFINAL` naming are the anti-patterns.)

**Q4.** Claude cites last quarter's refund window. The v4 policy shipped yesterday. Root cause and fix?
**A:** Stale knowledge. Replace v3 with v4. Announce the change. Bump the version note in the instructions. True-to-file, false-to-world is a maintenance failure.

**Q5.** When does Project RAG matter, and who gets it?
**A:** Paid plans, when knowledge approaches context limits. Retrieval expands capacity up to about 10×. Help it with clear naming and narrow questions.

**Q6.** A user expects a new chat to know yesterday's project decisions automatically. What is the right configuration answer?
**A:** Put decisions in Project knowledge (team truth). Memory may recall some context (per-user, plan-dependent, project-scoped). Memory is not a reliable or shared store for load-bearing facts.

**Q7.** Analysis-only Drive workflow: which connector scope?
**A:** Read-only / least privilege. Write scopes need governance approval and human gates.

**Q8.** Claude keeps addressing a manager by a project they left months ago. Fix?
**A:** Settings → Memory: edit or delete the stale entry. Memory is user-curated configuration, not fate.

**Q9.** "Can view" vs "Can edit" on a shared Project — the difference?
**A:** View = use the Project read-only. Edit = modify knowledge and instructions. Grant edit to a small set of people. Sharing is a governance control.

**Q10.** Where do rich multi-step procedures belong: instructions or a Skill?
**A:** A Skill. Procedures load when they are relevant. Instructions carry short standing rules. If you copy a Skill into instructions, you bloat context.

**Q11.** Two conflicting style guides live in one Project's knowledge. Symptom and fix?
**A:** Inconsistent brand outputs from retrieval confusion. Remove one guide, or add an explicit priority rule in instructions.

**Q12.** An exec wants sensitive board prep with zero trace in memory or history. Recommendation?
**A:** Incognito chat (all plans, outside Projects). Exclude It from history, memory, and search. For recurring board work, a dedicated Project with tight sharing beats repeated incognito.

**Q13.** Enterprise Search returns nothing for a source the team swears is connected. Checks?
**A:** Confirm the admin enabled the connector. Confirm that the user who asks has authenticated. Confirm the source is in scope. Check config before you blame the model.

**Q14.** Who maintains a shared Project, and what artifact proves it?
**A:** A named owner. A changelog note in knowledge that records file swaps and instruction versions.

**Q15.** A Free-plan user wants three custom MCP connectors. Reality?
**A:** Custom connectors exist on Free but are limited to one. Multiple custom connectors need a paid plan (verified Aug 2026).

**Q16.** Personal Styles vs Project instructions — when is Styles the wrong answer?
**A:** Whenever the rule is workspace truth (legal constraints, team schemas). Styles are personal taste. Styles do not govern a shared Project.

**Q17.** Select TWO reasons memory being *project-scoped* matters to configuration.
**A:** Client contexts cannot bleed between Projects. Sensitive project discussions stay out of general-chat memory. (This is also why "keep each client in its own Project" is the standing design rule.)

**Q18.** During teammate onboarding to a configured Project, what two steps prevent week-one chaos?
**A:** Confirm their connector authentication. Run a supervised first task against the evaluation checklist.

---

## Quick review checklist (pre-exam)

- [ ] Project setup: purpose, owner, sensitivity, instructions, knowledge, sharing
- [ ] Instructions: short, prioritized, conflict rules, "Not found" escalation, file pointers
- [ ] Knowledge: versioned names, no duplicates, replace superseded, manifest
- [ ] Upload vs connector. Least privilege. Admin-enable + user-auth
- [ ] Free caps: five projects, one custom connector. RAG paid ~10×
- [ ] Memory: plans, project-scoped, view/edit/delete/reset, independent toggles
- [ ] Incognito: all plans, no history/memory/search
- [ ] Maintenance cadence + changelog + named owner. Archive dead Projects

---

## Glossary

| Term | Meaning |
|---|---|
| **Project instructions** | Standing behavioral rules scoped to a workspace |
| **Project knowledge** | Curated reference files. The team's source of truth |
| **RAG (Projects)** | Paid retrieval that expands knowledge capacity near context limits |
| **Connector** | Live integration (e.g., Drive, Gmail). Admin-enabled, user-authenticated |
| **Least privilege** | Minimum scope that does the job. Read-only default |
| **Memory** | Per-user, project-scoped remembered context. Curated in Settings |
| **Memory entry** | One editable/deletable remembered fact with chat citation |
| **Incognito chat** | Chat excluded from history, memory, and search |
| **Knowledge hygiene** | Naming, versioning, duplicate removal, and pruning discipline |
| **Configuration drift** | Slow divergence of instructions/knowledge/memory from reality |
