---
title: Claude Platform & Model Foundations
---

# Domain 01 — Claude Platform & Model Foundations
## Maps to official CCAO-F **Product and Model Selection** (~12%, ~7 questions)

> **Dedup note (2026-08-23):** Rewritten as a single primary-study copy. Earlier builds repeated the same drill blocks ~7×; duplicates removed, content deepened to the Domain 03 standard, and product facts re-verified against support.claude.com (marked below).

## Disclaimer

Original CCAO-F study notes for non-developers using claude.ai (Projects, Artifacts, Skills, Connectors, Research). Grounded in public Anthropic Help Center & product docs, public Claude Academy (Claude 101, AI Fluency 4D framework, model-choice tutorials), and the published CCAO-F blueprint. Independent; not affiliated with Anthropic. Verify live product details on support.claude.com.

---

## Overview

Pick the right Claude **feature** and **model** for the job. Official blueprint verbs: select appropriate product features (Projects, research mode, chat, artifacts); differentiate model types (**Haiku, Sonnet, Opus** — the guide names these three); align model selection with task requirements (cost, speed, quality); understand and manage context limitations and **memory considerations** (when to restart, summarize, or persist). ~7 questions, but this domain is vocabulary for the whole exam: Domains 04–07 assume you can name the right feature instantly.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | Anchors |
|---|---|---|
| Feature selection | select, recommend | Chat, Projects, Artifacts, Skills, Connectors, Research, Enterprise Search, code execution/file creation |
| Model differentiation | differentiate, compare | Haiku / Sonnet / Opus tiers (Fable exists above Opus — see note) |
| Task alignment | align, escalate | Cost, speed, quality; effort; lightest-shippable rule |
| Context & memory | restart, summarize, persist | Context window, Project knowledge, Memory feature, incognito, handoff briefs |
| Plans | identify constraints | Free / Pro / Max / Team / Enterprise caps hidden in stems |

---

## Deep notes

### 1. Three shapes of work

- **Turn-by-turn chat** — you steer every step. Best for iteration, quick answers, drafting with feedback.
- **Hand-off agentic** — you brief, Claude works multi-step (Research; long agentic tasks). Costs more usage; needs a better brief and citation checking.
- **Reusable systems** — Projects, Skills, Artifacts, Connectors encode context and procedure once so the whole team stops re-explaining.

Most exam feature stems are really asking: which shape is this job?

### 2. Feature catalog (what to reach for, and when)

**Chat.** Fast iteration on one thread. Weak for recurring shared context — that's a Project. Chats persist in your history (and, with Memory on, can inform later chats — see §5); they are not a team knowledge base.

**Projects.** Self-contained workspaces with **instructions** (standing behavior) + **knowledge** (reference files) and separate chat histories. Available on **all plans; Free accounts are capped at five projects** (verified support.claude.com, Aug 2026). **Enhanced knowledge via RAG is paid-plan only and expands knowledge capacity by up to ~10×**, activating when project knowledge approaches context limits (verified Aug 2026). Sharing is a Team/Enterprise capability: "Can view" vs "Can edit," shared individually, in bulk, or org-wide. Your chats inside a shared Project stay yours — sharing distributes knowledge and instructions, not your transcripts.

**Artifacts.** Standalone editable deliverables (docs, code, HTML/SVG, interactive apps) rendered beside the chat. Can be published/shared; AI-powered artifacts consume the **viewer's** usage limits, not the creator's. Not pre-validated — Domain 03 rules still apply.

**Skills.** Packaged task procedures loaded when relevant. Contrast: Project knowledge is always-available reference; a Skill is a *repeatable procedure*; a Connector is *live data*.

**Connectors (MCP).** Attach live systems (Drive, Gmail, Slack, custom remote MCP servers). Admin enablement plus per-user authentication is the normal pattern on Team/Enterprise. **Custom connectors are available on every plan, but Free accounts are limited to one custom connector** (verified Aug 2026). Governance lives in Domain 06.

**Research.** Agentic multi-angle investigation with citations; needs web search on; can use connectors; paid plans; burns usage faster than chat. For a single already-uploaded PDF, quote the file instead.

**Enterprise Search.** Org-internal search with citations back to connected sources (Team/Enterprise). "What did we decide?" → Enterprise Search. "What's the public landscape?" → Research.

**Code execution & file creation.** Claude has a **sandboxed computing environment** on claude.ai: it writes and runs code to analyze data, build charts, and generate downloadable files — Excel, PowerPoint, Word, PDF, CSV. **Available on all plans (Free–Enterprise) on web, Desktop, and Mobile** (verified Aug 2026); it replaced the older "analysis tool." Reach for it when the stem involves *computation on data* (totals, pivots, trends from a spreadsheet) or a *file deliverable* — pattern-matching in plain chat is unreliable for arithmetic at scale. Distinct from Artifacts: an Artifact is the interactive/editable deliverable; code execution is the compute that can produce files and verified numbers.

### 3. Model ladder and task alignment

The exam guide names three tiers — differentiate them cleanly:

| Tier | Character | Reach for |
|---|---|---|
| **Haiku** | Fastest, lightest, cheapest | Extraction, classification, short summaries, high-volume simple tasks |
| **Sonnet** | Balanced daily default | Most drafting, analysis, everyday knowledge work |
| **Opus** | Deepest reasoning | Hard multi-constraint strategy, subtle analysis — *after* a lighter model or better brief has failed |

*Note:* the current product also offers a tier above Opus (Fable-class) for the longest-horizon, most complex work. The guide's objective language is Haiku/Sonnet/Opus — answer in those terms; "escalate past Opus" appears in stems only as the prestige-bias distractor.

Rules that decide exam answers:

- **Lightest shippable model wins.** Evaluate from lighter models upward; stop at the cheapest output you would actually ship. The official sample question rewards "faster, lower-cost model for straightforward, high-volume tasks."
- **Escalate on demonstrated failure, not prestige.** "Use the biggest model to maximize quality" is the canonical wrong option — it wastes cost and latency budget.
- **Effort / extended thinking tune depth *inside* a model.** Lower effort for quick lookups; higher for genuinely hard problems. Sometimes the right fix is more effort on Sonnet, not a jump to Opus.
- **Match the failure to the remedy.** Haiku missing nuance → move up. Sonnet shallow on adversarial strategy → Opus or a better decomposition. Opus "failing" on cost/latency/limits → move *down* or restructure. Bad brief → fix Description (Domain 02) before touching the model picker.
- **Try-two-models method.** Run the same known task on Haiku then Sonnet; compare substance, not length. If Haiku already surfaces the claims you care about, stay light.

### 4. Context limitations (restart / summarize / persist)

The context window is finite; long threads degrade — drift, contradictions, forgotten constraints.

- **Restart** when contradictions accumulate, roles drift, or quality collapses. Fresh thread + a tight brief beats nursing a polluted one.
- **Summarize** when you need decision continuity without the draft noise: ask for a handoff brief (decisions, open questions, constraints) and start a new chat from it.
- **Persist** when reuse is predictable: put standing rules in Project instructions and reference files in Project knowledge — not in a heroic first message every Monday.

Instructions compete with task tokens: keep them short and stable; put bulky policy in knowledge files with clear names, and ask by filename.

### 5. Memory (the persist layer built into the product)

claude.ai now has a first-class **Memory** feature (verified support.claude.com, Aug 2026 — details evolve; re-check before exam day):

- Claude builds **memory entries** from your chats (professional context, preferences, ongoing work) and applies them in later conversations. Memory generation is available on **Free, Pro, and Max** (newer experience) and **Enterprise** (legacy synthesis experience); **chat search** (retrieving past chats) is Pro and above.
- **Project-scoped:** each Project gets its own separate memory, so client A's context doesn't bleed into client B's — a direct answer to the exam's "memory considerations" objective.
- **You control it:** view, edit, delete individual entries, or reset all in Settings; search and memory toggle independently.
- **Incognito chats** (ghost icon, all plans, outside Projects) are excluded from history, memory, and search — use them for one-off sensitive queries.

**Exam framing:** "restart, summarize, persist" is still the skill. Memory reduces re-explaining across chats, but it is *not* a team knowledge base (it's per-user), *not* a substitute for Project knowledge (curated files beat inferred memories for policy questions), and *not* guaranteed complete — load-bearing context still belongs in Projects. Configuration and governance of Memory live in Domain 05 (see `05-configuration-knowledge-management.md` §Memory).

### 6. Plans and hidden constraints

Exam stems hide plan constraints in subordinate clauses. Solve the constrained world in the stem, not the ideal world in your head:

- Free: five-project cap; one custom connector; no Research.
- Research needs a paid plan **and** web search enabled.
- Enterprise Search needs Team/Enterprise with admin-connected sources.
- Team member ≠ admin: enablement is admin's; authentication is usually per-user.
- Artifact viewers burn their own limits; capabilities toggled off block artifact/code features.

### 7. Fluency bridge (4D)

**Delegation** is this whole domain: choosing automation vs augmentation vs agency, and which feature/model carries it. **Description** includes choosing Research vs chat, not just wording. **Discernment** applies hardest to heavy models — long fluent Opus output still needs Domain 03 evaluation. **Diligence** treats publishing an Artifact or enabling a connector as a deployment act.

Wrong feature masquerades as a prompting failure; wrong model masquerades as "Claude is dumb." Domain 07 will ask which layer broke — keep a one-pager per recurring workflow: Feature | Model | Context strategy.

### 8. Mini-cases (feature + model in one breath)

**PMO Friday status:** Project with template instructions + Drive connector; Sonnet default. **Board strategy interrogation:** Opus (with Domain 03 evaluation still mandatory). **Support refund question:** answer from Project policy or Enterprise Search — never general model lore. **Workshop icebreakers:** chat + Haiku/Sonnet; skip Research. **RFP compare:** rubric in Project, public landscape via Research, computed scoring via code execution, output as table/Artifact. **Quarter-end deck:** data upload → code execution for figures → Sonnet drafts commentary → human validates.

Anti-patterns: one mega-Project mixing legal + HR + marketing; Research for a single uploaded PDF; write-capable connectors for read-only analysis; the top tier as a creativity dial; assuming an Artifact's viewers use the creator's credentials or limits; relying on Memory to carry a team's policy truth.

### 9. Multiple-response pattern bank

Select-TWO/THREE stems here pair one right feature/model move with distractors. Rehearse eliminating: "use Opus for everything important" (prestige bias); "Projects share your chat transcripts" (false mechanics); "Research works on Free" / "Research without web search" (plan/capability misses); "Memory replaces Project knowledge" (persist-layer confusion); "have chat compute the 10,000-row totals" (wrong tool — code execution). Correct combinations usually pair a **feature-fit** choice (Project / Research / code execution / Artifact) with a **constraint-respecting** choice (plan cap, auth step, lightest-shippable model).

---

## Decision trees

**Feature:** Reusable/recurring context → Project. Cited multi-source public investigation → Research. Org-internal "what did we decide" → Enterprise Search. Standalone interactive deliverable → Artifact. Repeatable procedure → Skill. Live system of record → Connector. Computation on data / file deliverable → code execution. One-off question → chat.

**Model:** Instant extract/classify → Haiku. Default knowledge work → Sonnet. Deep reasoning after a lighter attempt failed → Opus. Tune effort before jumping tiers.

**Context:** Drifting thread → restart. Need continuity, not noise → summarize into a handoff. Predictable reuse → persist in Project. Cross-chat personal context → Memory (verify entries; sensitive one-offs → incognito).

---

## Exam traps

1. Biggest model "to be safe" (prestige bias — the official sample question punishes it)
2. Believing Project sharing shares your chat transcripts (it shares knowledge + instructions)
3. Research without web search enabled, or Research for one already-uploaded PDF
4. "Enabled = access": forgetting per-user connector authentication
5. "Free has no Projects" (false — capped at five) / ignoring the Free one-custom-connector cap
6. Treating Artifacts as pre-validated because they look like products
7. Enterprise Search ≠ Research (internal cited lookup vs public agentic investigation)
8. Asking chat to "calculate" large-table arithmetic instead of using code execution
9. Treating Memory as a shared team brain or as a policy source of truth
10. Escalating the model when the brief, knowledge, or feature choice is what's broken

---

## Practice Q&A (18) — scenario stems with answers and rationales

**Q1.** A marketing lead rebuilds the same brand-voice briefing every Monday by pasting the style guide into a new chat. What should they set up?
**A:** A Project with the style guide in knowledge and standing rules in instructions. Recurring context belongs in a persistent workspace, not weekly heroic pasting.

**Q2.** An ops team must pull totals and a pivot by region from a 40,000-row CSV and deliver an Excel file. Best approach?
**A:** Upload the file and have Claude use code execution to compute and generate the spreadsheet. Chat-only "reading" of large tables risks arithmetic errors; code execution runs real code and produces the downloadable file.

**Q3.** A Free-plan user wants a sixth project for a new client. What's the reality?
**A:** Free accounts cap at five projects — archive or merge one, or upgrade. (Plan caps change; verify at support.claude.com.)

**Q4.** An associate runs Research and gets nothing useful; web search is toggled off. First fix?
**A:** Enable web search — Research is an agentic web investigation and depends on it. Diagnose capability toggles before blaming the model.

**Q5.** A consultant wants Claude to remember their reporting format across chats, but client-confidential details must never carry over between two clients. What product behavior helps?
**A:** Memory is project-scoped — each Project keeps separate memory. Keep each client in its own Project; general preferences can live in account-level memory, and entries can be viewed/edited/deleted in Settings.

**Q6.** A teammate asks a one-off sensitive HR question and doesn't want it in history or memory. What do you recommend?
**A:** An incognito chat (ghost icon) — excluded from history, memory, and search; available on all plans, outside Projects.

**Q7.** Select TWO tasks that fit Haiku.
**A:** Classifying inbound tickets by type; extracting fields from invoices. High-volume, well-specified, low-nuance work is the light-model sweet spot.

**Q8.** Sonnet's competitive analysis feels shallow on a genuinely multi-constraint strategy question after two solid iterations. Next move?
**A:** Escalate to Opus (or raise effort) — a demonstrated failure on hard reasoning is the escalation trigger, not prestige.

**Q9.** Legal needs "what did we decide about data retention in Q2?" answered from Slack and Drive. Feature?
**A:** Enterprise Search — org-internal cited lookup across connected sources. Research is for the public landscape.

**Q10.** A 300-message thread now contradicts its own constraints. Best context move?
**A:** Ask for a handoff summary (decisions, constraints, open questions) and restart a fresh chat from it. Summarize-then-restart beats pushing a drifted thread.

**Q11.** An exec wants an interactive ROI calculator the sales team can use. Feature?
**A:** An Artifact (interactive app). Note viewers consume their own usage limits, and the calculator's formulas still need Domain 03 validation.

**Q12.** A Team-plan user shares a Project org-wide and worries colleagues will read their draft chats. What's true?
**A:** Sharing distributes knowledge and instructions with view/edit permissions; their own chats in the Project are not handed to every member as transcripts. (Verify current sharing behavior on support.claude.com.)

**Q13.** Select TWO reasons the "always use the most capable model" policy fails.
**A:** It wastes cost/latency on tasks a light model ships; it substitutes prestige for the evaluate-upward method. (Quality on simple tasks does not improve enough to justify it.)

**Q14.** The team repeats a 12-step QBR-prep procedure monthly and keeps re-teaching it in chat. Feature?
**A:** A Skill — packaged, reusable procedure. Project knowledge holds the reference material; the Skill holds the *how*.

**Q15.** A paid-plan Project's knowledge has grown near context limits. What keeps it usable?
**A:** Enhanced knowledge with RAG (paid plans) — retrieval expands capacity up to ~10×; help it by naming files clearly and asking narrowly.

**Q16.** A user says "Claude forgot everything from yesterday" in a fresh chat on a Pro plan. Diagnosis?
**A:** Check whether Memory/chat search is enabled and whether yesterday's chat was incognito or in a different Project's scope. Then decide: persist standing context in a Project rather than relying on recall.

**Q17.** A director asks whether Claude can draft board-pack prose "with zero hallucination risk if we buy Opus." Correct framing?
**A:** No tier eliminates hallucination — model choice trades cost/speed/quality, while validation (Domain 03) manages accuracy. Opus + skipped review is worse than Sonnet + claim map.

**Q18.** Select TWO correct pairings of need → surface.
**A:** Live Drive data in answers → Connector; downloadable PowerPoint from analyzed data → code execution/file creation. (Research is not for single uploaded files; Memory is not a team knowledge base.)

---

## Quick review checklist (pre-exam)

- [ ] Define every feature in one sentence, including code execution/file creation and Memory
- [ ] Free caps: five projects, one custom connector; RAG = paid, ~10×
- [ ] Model ladder Haiku/Sonnet/Opus + lightest-shippable + escalate-on-failure
- [ ] Effort tunes depth inside a model
- [ ] Restart / summarize / persist — and where Memory does and doesn't help
- [ ] Incognito = out of history, memory, search
- [ ] Research needs paid plan + web search; Enterprise Search needs admin-connected sources
- [ ] Connector pattern: admin enables, user authenticates

---

## Glossary

| Term | Meaning |
|---|---|
| **Project** | Workspace with instructions + knowledge + its own chats |
| **RAG (in Projects)** | Paid-plan retrieval that expands knowledge capacity (~10×) near context limits |
| **Artifact** | Standalone editable/interactive deliverable beside the chat |
| **Skill** | Packaged reusable task procedure |
| **Connector (MCP)** | Live integration to an external system; admin-enabled, user-authed |
| **Research** | Paid agentic multi-source investigation with citations |
| **Enterprise Search** | Org-internal cited search (Team/Enterprise) |
| **Code execution / file creation** | Sandboxed environment where Claude runs code and generates files (all plans) |
| **Memory** | Per-user, project-scoped remembered context; view/edit/delete in Settings |
| **Incognito chat** | Ghost-icon chat excluded from history, memory, and search |
| **Effort** | Depth control inside a model tier |
| **Context window** | Finite working memory of a conversation |
