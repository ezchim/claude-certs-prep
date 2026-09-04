---
title: Configuration & Knowledge Management
---

# Domain 05 — Configuration & Knowledge Management
## Maps to official CCAO-F **Configuration and Knowledge Management** (~12%, ~7 questions)

> **Dedup note (2026-08-23):** Rewritten as a single primary-study copy. Earlier builds repeated the same drill blocks ~7×; duplicates removed, content deepened to the Domain 03 standard, plan facts re-verified against support.claude.com, and a full **Memory** section added (it was a named exam-guide topic with near-zero coverage).

## Disclaimer

Original CCAO-F study notes for non-developers using claude.ai (Projects, Artifacts, Skills, Connectors, Research). Grounded in public Anthropic Help Center & product docs, public Claude Academy (Claude 101, AI Fluency 4D), and the published CCAO-F blueprint. Independent; not affiliated with Anthropic. Verify live product details on support.claude.com.

---

## Overview

Configure Claude so recurring work stays consistent. Official blueprint verbs: **configure** Claude Projects with instructions and knowledge sources; **manage** uploaded knowledge and connectors (the guide names Google Drive and Gmail); **create** effective system-level instructions; **inform, maintain, and update** configurations, knowledge sources, and instructions over time. This is the "make it reliable for the team" domain — and, since Memory shipped, the "manage what Claude remembers" domain too.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | What you practice |
|---|---|---|
| Configure Projects | configure, set up | Instructions + knowledge + sharing + ownership |
| Manage knowledge & connectors | manage, curate, authorize | Upload vs connect; hygiene; least privilege |
| Effective instructions | create, structure | Short, prioritized, conflict rules, schemas |
| Maintain & update | maintain, refresh, retire | Cadence, changelogs, archiving, Memory curation |

---

## Deep notes

### 1. Project setup that survives contact with a team

Checklist: purpose; owner; sensitivity label; instructions; knowledge set; sharing permissions; model-default guidance; evaluation notes. Verified plan facts (support.claude.com, Aug 2026): Projects exist on **all plans; Free caps at five projects**; **enhanced knowledge via RAG is paid-only and expands capacity up to ~10×**, activating as knowledge approaches context limits; **sharing is Team/Enterprise** with "Can view" (read-only) vs "Can edit" (modify knowledge/instructions), granted individually, in bulk, or org-wide.

### 2. Instruction engineering

Good instructions specify: role; audience defaults; output schemas; citation rules; escalation ("if not in knowledge, say **Not found**"); conflict priority (legal > brand); pointers to files by name. Bad instructions narrate the company novel, contradict themselves, or bury critical rules. Keep them **short, stable, prioritized** — instructions compete with task tokens; bulky policy belongs in knowledge files.

Test instructions with three golden tasks and three adversarial tasks (ask for out-of-knowledge answers and see whether "Not found" fires). Update when failures cluster. **Styles/preferences vs Project instructions:** global personal taste vs workspace rules — a Style can't carry a Project's legal constraints. **Skills vs instructions:** procedures vs standing rules; don't paste whole Skills into instructions.

### 3. Knowledge hygiene playbook

- **Naming:** `Policy-Refunds-v4-2026-06.pdf` beats `finalFINAL.pdf`. Version dates in filenames; ask by filename.
- **Curation:** remove superseded docs the day the new version lands — a stale file is a silent production bug (true to file, false to world). No duplicate/conflicting versions; retrieval confusion is self-inflicted.
- **Structure:** split unrelated domains (legal / HR / marketing) into separate Projects; keep a manifest note listing each document and its purpose.
- **Quality:** prefer text-extractable files; OCR-junk scans poison retrieval.
- **RAG behavior:** when knowledge is large, expect search-like retrieval — ask narrowly and name files.

### 4. Connectors: upload vs connect, and least privilege

Upload = a frozen snapshot you must refresh; connector = live data with auth and permission complexity. Choose deliberately: living documents that change weekly → connector; stable policy of record → curated upload with version discipline.

Governance basics (Domain 06 owns the deep end): least privilege — prefer **read-only** for analysis; write scopes are a governance decision with human gates. Team/Enterprise pattern: **admin enables, each user authenticates**. Document which connectors a Project expects, so "Claude can't see Drive" gets diagnosed as auth, not model failure. **Custom connectors (remote MCP) exist on every plan; Free is limited to one custom connector** (verified Aug 2026).

### 5. Memory — configuring what Claude remembers (added; named in the exam guide)

Facts verified against support.claude.com (Aug 2026 — this feature evolves; re-check before exam day):

- **What it is:** Claude builds **memory entries** from your chats — professional context, role, preferences, ongoing work — organized into categories and applied to later conversations. This replaces the old mental model "chats are ephemeral; nothing persists between threads."
- **Availability:** memory generation on **Free, Pro, Max** (newer experience) and **Enterprise** (legacy 24-hour synthesis experience); **chat search** (Claude retrieving your past chats) is **Pro and above**. Team plans have **no org-level memory controls** — members manage their own.
- **Project scoping:** every Project has its **own separate memory** and summary, keeping client work, confidential threads, and general chats apart. This is the configuration answer to "keep contexts from bleeding."
- **Controls (this domain's core):** Settings → Memory: **view** entries, **edit**, **delete individual entries**, or **reset all**; memory and search **toggle independently**; citations show which chat a remembered fact came from.
- **Incognito chats:** ghost icon, all plans, outside Projects; excluded from history, memory, and search. (Enterprise retains incognito chats a minimum period for safety and includes them in data exports — governance detail.)
- **Portability:** memory is included in data exports; experimental import/export can move memory between assistants.

**Management doctrine:** Memory is a *personal context layer*, not a knowledge base. Curate it like knowledge: review entries periodically; delete wrong or stale ones (a bad memory entry is configuration drift you carry into every chat); disable or use incognito for sensitive matters; and keep load-bearing team truth in **Project knowledge** — curated files with owners beat inferred per-user memories. When Claude "remembers wrong," the fix is edit/delete the entry, not a bigger model (Domain 07 bridge).

### 6. Maintenance, lifecycle, ownership

Every shared Project needs a named owner accountable for updates; without one, configuration drift is guaranteed. Cadence: review instructions monthly; refresh knowledge when policies change (replace file + announce + bump a version note in instructions); changelog in knowledge for shared Projects; archive dead Projects — on Free, the five-project cap forces it. Onboarding teammates: share the Project, explain instructions, show the evaluation checklist, confirm connector auth, supervise a first task.

### 7. Failure modes that look like model bugs

Claude "ignores policy" → superseded PDF still in knowledge. "Mixes brands" → two style guides with no priority rule. "Forgot our project context" → user relied on personal Memory instead of Project knowledge, or memory entry was deleted/scoped to another Project. "Can't see Drive" → connector not authenticated. "Remembers something wrong about me" → stale memory entry; edit or delete it. Fix configuration before escalating models.

### 8. Mini-cases (configuration audits)

**Case 1 — The mystery brand voice.** Marketing's shared Project alternates between formal and punchy copy. Audit finds `Brand-Voice-2025.pdf` and `Voice-Refresh-2026.docx` both in knowledge, no priority rule, no owner. Fix: delete the 2025 file, add "voice per `Voice-Refresh-2026.docx`" to instructions, name an owner, start a changelog. One audit, every user fixed.

**Case 2 — The consultant's bleed risk.** A solo consultant serves two competing clients from general chat and worries context will leak. Fix: one Project per client (memory is project-scoped; knowledge separated), general Memory kept for personal preferences only, periodic review of memory entries, incognito for exploratory conflict-sensitive questions.

**Case 3 — The Friday-report Project that rotted.** Built in March, praised in April, wrong by August: policy v5 shipped but v3 is still in knowledge; the builder left; nobody owns it. Classic drift. Fix: assign an owner, replace the file, add a version note to instructions, calendar a monthly review. Prevention was cheaper than the bad report that surfaced it.

**Case 4 — Onboarding gone sideways.** A new analyst gets Project access, but their first outputs cite nothing from Drive. Audit: connector admin-enabled but the analyst never authenticated; also they used their personal account, not the workspace. Fix: workspace seat, per-user auth walkthrough, supervised first task against the evaluation checklist.

### 9. Multiple-response pattern bank

Eliminate these recurring wrong options: "paste the whole policy into instructions" (bloat — knowledge's job); "keep the old file for reference" (superseded = delete or clearly archive); "memory will keep the team aligned" (per-user, not shared); "grant edit to everyone so nobody's blocked" (permission failure); "re-upload weekly instead of connecting" when the doc is living (or its mirror: "connect everything" including stable policy). Correct combinations pair a **placement** choice (instructions vs knowledge vs Skill vs connector vs Memory) with a **lifecycle** choice (owner, version note, cadence, archive).

---

## Decision trees

| Situation | Action |
|---|---|
| Recurring team context | Project (instructions + knowledge + owner) |
| Bulky reference material | Knowledge file, named and versioned — not instructions |
| Live, frequently changing docs | Connector (read-only unless writes are governed) |
| Policy updated | Replace file + announce + version note |
| Personal cross-chat preferences | Memory — review/edit entries in Settings |
| Confidential one-off chat | Incognito (excluded from memory/history/search) |
| Contexts must not bleed | Separate Projects (memory is project-scoped) |
| Free near five-project cap | Archive or merge |
| Wrong remembered fact | Edit/delete the memory entry |
| Shared Project misbehaving | Check owner, duplicates, staleness before model blame |

---

## Exam traps

1. Everything crammed into instructions (novel-length, self-contradicting)
2. Never updating knowledge; superseded PDFs left in place
3. Treating personal Memory as the team's knowledge base
4. Assuming memory crosses Project boundaries (it's scoped per Project)
5. Write-scope connectors by default for analysis-only work
6. Ignoring sharing permissions (org-wide edit on restricted knowledge)
7. Duplicate conflicting files causing retrieval confusion
8. Shared Project with no owner (guaranteed drift)
9. Styles doing a Project's legal-compliance job
10. Forgetting incognito exists for sensitive one-offs

---

## Practice Q&A (18) — scenario stems with answers and rationales

**Q1.** A team needs standing tone rules plus a 40-page policy PDF applied to weekly work. Configuration?
**A:** Project: short tone rules in instructions; the PDF in knowledge, named with version and date; instructions point to it by filename.

**Q2.** A Free-plan consultant hits the project cap starting client #6. Options?
**A:** Archive or merge an inactive project, or upgrade — Free caps at five (verified Aug 2026; re-check current caps at support.claude.com).

**Q3.** Select TWO knowledge-hygiene practices.
**A:** Version dates in filenames; removing superseded documents immediately. (Duplicates and `finalFINAL` naming are the anti-patterns.)

**Q4.** Claude cites last quarter's refund window. The v4 policy shipped yesterday. Root cause and fix?
**A:** Stale knowledge — replace v3 with v4, announce the change, bump the instructions' version note. True-to-file, false-to-world is a maintenance failure.

**Q5.** When does Project RAG matter, and who gets it?
**A:** Paid plans, when knowledge approaches context limits — retrieval expands capacity up to ~10×; help it with clear naming and narrow asks.

**Q6.** A user expects a new chat to know yesterday's project decisions automatically. What's the right configuration answer?
**A:** Put decisions in Project knowledge (team truth) — Memory may recall some context (per-user, plan-dependent, project-scoped) but is not a reliable or shared store for load-bearing facts.

**Q7.** Analysis-only Drive workflow: which connector scope?
**A:** Read-only / least privilege. Write scopes need governance approval and human gates.

**Q8.** Claude keeps addressing a manager by a project they left months ago. Fix?
**A:** Settings → Memory: edit or delete the stale entry. Memory is user-curated configuration, not fate.

**Q9.** "Can view" vs "Can edit" on a shared Project — the difference?
**A:** View = use the Project read-only; Edit = modify knowledge and instructions. Grant edit narrowly; sharing is a governance control.

**Q10.** Where do rich multi-step procedures belong: instructions or a Skill?
**A:** A Skill — procedures load when relevant; instructions carry short standing rules. Duplicating a Skill inside instructions bloats context.

**Q11.** Two conflicting style guides live in one Project's knowledge. Symptom and fix?
**A:** Inconsistent brand outputs from retrieval confusion — remove one or add an explicit priority rule in instructions.

**Q12.** An exec wants sensitive board prep with zero trace in memory or history. Recommendation?
**A:** Incognito chat (all plans, outside Projects) — excluded from history, memory, and search. For recurring board work, a dedicated Project with tight sharing beats repeated incognito.

**Q13.** Enterprise Search returns nothing for a source the team swears is connected. Checks?
**A:** Admin actually enabled the connector; the asking user authenticated; the source is in scope. Config before model blame.

**Q14.** Who maintains a shared Project, and what artifact proves it?
**A:** A named owner; a changelog note in knowledge recording file swaps and instruction versions.

**Q15.** A Free-plan user wants three custom MCP connectors. Reality?
**A:** Custom connectors exist on Free but are limited to one; multiple custom connectors need a paid plan (verified Aug 2026).

**Q16.** Personal Styles vs Project instructions — when is Styles the wrong answer?
**A:** Whenever the rule is workspace truth (legal constraints, team schemas): Styles are personal taste and don't govern a shared Project.

**Q17.** Select TWO reasons memory being *project-scoped* matters to configuration.
**A:** Client contexts can't bleed between Projects; sensitive project discussions stay out of general-chat memory. (It's also why "keep each client in its own Project" is the standing design rule.)

**Q18.** During teammate onboarding to a configured Project, what two steps prevent week-one chaos?
**A:** Confirm their connector authentication, and run a supervised first task against the evaluation checklist.

---

## Quick review checklist (pre-exam)

- [ ] Project setup: purpose, owner, sensitivity, instructions, knowledge, sharing
- [ ] Instructions: short, prioritized, conflict rules, "Not found" escalation, file pointers
- [ ] Knowledge: versioned names, no duplicates, replace superseded, manifest
- [ ] Upload vs connector; least privilege; admin-enable + user-auth
- [ ] Free caps: five projects, one custom connector; RAG paid ~10×
- [ ] Memory: plans, project-scoped, view/edit/delete/reset, independent toggles
- [ ] Incognito: all plans, no history/memory/search
- [ ] Maintenance cadence + changelog + named owner; archive dead Projects

---

## Glossary

| Term | Meaning |
|---|---|
| **Project instructions** | Standing behavioral rules scoped to a workspace |
| **Project knowledge** | Curated reference files; the team's source of truth |
| **RAG (Projects)** | Paid retrieval expanding knowledge capacity near context limits |
| **Connector** | Live integration (e.g., Drive, Gmail); admin-enabled, user-authed |
| **Least privilege** | Minimum scope that does the job; read-only default |
| **Memory** | Per-user, project-scoped remembered context; curated in Settings |
| **Memory entry** | One editable/deletable remembered fact with chat citation |
| **Incognito chat** | Chat excluded from history, memory, and search |
| **Knowledge hygiene** | Naming, versioning, dedup, and pruning discipline |
| **Configuration drift** | Slow divergence of instructions/knowledge/memory from reality |
