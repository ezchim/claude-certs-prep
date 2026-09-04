# Domain 01 — Claude Platform & Model Foundations
## Maps to official CCAO-F **Product and Model Selection** (~12%, ~7 questions)

> **Note:** This edition follows the ASD-STE100 Simplified Technical English writing rules. Simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Haiku, Sonnet, Opus, Fable, MCP, Projects, Artifacts, Skills, Connectors, Research, RAG, Memory, prompting, effort) are exceptions and stay as written.

> **Dedup note (2026-08-23):** This is one primary-study copy. Earlier builds repeated the same drill blocks about 7 times. Those duplicates are gone. The content now matches the Domain 03 depth. Check Product facts against support.claude.com (marked below).

## Disclaimer

These notes are original CCAO-F study notes. They are for people who are not developers. They use claude.ai (Projects, Artifacts, Skills, Connectors, Research). The notes use public Anthropic Help Center and product docs. They also use public Claude Academy (Claude 101, AI Fluency 4D framework, model-choice tutorials). They use the published CCAO-F blueprint. The notes are independent. They are not affiliated with Anthropic. Check live product details on support.claude.com.

---

## Overview

Select the correct Claude feature and model for the task. Official blueprint verbs follow. Select the right product features (Projects, research mode, chat, artifacts). Differentiate model types (**Haiku, Sonnet, Opus** — the guide names these three). Align model selection with task requirements (cost, speed, quality). Understand and manage context limitations and **memory considerations**. Know when to restart, summarize, or persist. This domain has about 7 questions. This domain is also vocabulary for the whole exam. Domains 04–07 assume you can name the right feature at once.

---

## Key map (objectives ↔ exam verbs)

| Official objective | Exam verbs | Anchors |
|---|---|---|
| Feature selection | select, recommend | Chat, Projects, Artifacts, Skills, Connectors, Research, Enterprise Search, code execution/file creation |
| Model differentiation | differentiate, compare | Haiku / Sonnet / Opus tiers (Fable exists above Opus — see note) |
| Task alignment | align, escalate | Cost, speed, quality. Effort. Lightest-shippable rule |
| Context & memory | restart, summarize, persist | Context window, Project knowledge, Memory feature, incognito, handoff briefs |
| Plans | identify constraints | Free / Pro / Max / Team / Enterprise caps that stems hide |

---

## Deep notes

### 1. Three shapes of work

- **Turn-by-turn chat** — you steer every step. This shape is best for iteration, quick answers, and drafts with feedback.
- **Hand-off agentic** — you give a brief. Claude then works in multiple steps (Research. Long agentic tasks). This shape costs more usage. It needs a better brief. You also need to check citations.
- **Reusable systems** — Projects, Skills, Artifacts, and Connectors hold context and procedure once. Then the whole team does not re-explain the same context.

Most exam feature stems ask this: which shape is this job?

### 2. Feature catalog (what to use, and when)

**Chat.** Use chat for fast iteration on one thread. Chat is weak for recurring shared context. That work is a Project. Chats stay in your history. With Memory on, they can inform later chats (see §5). Chats are not a team knowledge base.

**Projects.** A Project is a self-contained workspace. It has **instructions** (standing behavior) plus **knowledge** (reference files). It also has separate chat histories. Projects exist on **all plans**. **Free accounts cap at five projects** (verified support.claude.com, Aug 2026). **Enhanced knowledge via RAG is paid-plan only**. RAG expands knowledge capacity by up to about **10×**. RAG starts when project knowledge approaches context limits (verified Aug 2026). Sharing is a Team/Enterprise capability. Sharing uses "Can view" vs "Can edit." You share individually, in bulk, or org-wide. Your chats inside a shared Project stay yours. Sharing distributes knowledge and instructions. Sharing does not distribute your transcripts.

**Artifacts.** An Artifact is a standalone editable deliverable (docs, code, HTML/SVG, interactive apps). The product shows it beside the chat. You can publish or share an Artifact. AI-powered artifacts consume the **viewer's** usage limits, not the creator's. Artifacts are not pre-validated. Domain 03 rules still apply.

**Skills.** A Skill is a packaged task procedure. The product loads it when it is relevant. Contrast the three. Project knowledge is always-available reference. A Skill is a *repeatable procedure*. A Connector is *live data*.

**Connectors (MCP).** Attach live systems (Drive, Gmail, Slack, custom remote MCP servers). On Team/Enterprise, an admin enables the connector. Then each user authenticates. That is the normal pattern. **Custom connectors exist on every plan**. **Free accounts are limited to one custom connector** (verified Aug 2026). Domain 06 covers governance.

**Research.** Research is agentic multi-angle investigation with citations. It needs web search on. It can use connectors. It needs a paid plan. It costs more usage than chat. For a single already-uploaded PDF, quote the file instead.

**Enterprise Search.** Enterprise Search is org-internal search. It cites connected sources (Team/Enterprise). "What did we decide?" → Enterprise Search. "What is the public landscape?" → Research.

**Code execution & file creation.** Claude has a **sandboxed computing environment** on claude.ai. Claude writes and runs code. It analyzes data. It builds charts. It generates downloadable files: Excel, PowerPoint, Word, PDF, CSV. **This feature exists on all plans (Free–Enterprise) on web, Desktop, and Mobile** (verified Aug 2026). It replaced the older "analysis tool." Use it when the stem involves *computation on data* (totals, pivots, trends from a spreadsheet) or a *file deliverable*. Pattern-matching in plain chat is not reliable for arithmetic at scale. This is distinct from Artifacts. An Artifact is the interactive/editable deliverable. Code execution is the compute. It can produce files and verified numbers.

### 3. Model ladder and task alignment

The exam guide names three tiers. Differentiate them clearly:

| Tier | Character | Reach for |
|---|---|---|
| **Haiku** | Fastest, lightest, cheapest | Extraction, classification, short summaries, high-volume simple tasks |
| **Sonnet** | Balanced daily default | Most drafts, analysis, everyday knowledge work |
| **Opus** | Deepest reasoning | Hard multi-constraint strategy, subtle analysis — *after* a lighter model or a better brief fails |

*Note:* The current product also offers a tier above Opus (Fable-class). Use it for the longest-horizon, most complex work. The guide's objective language is Haiku/Sonnet/Opus. Answer in those terms. "Escalate past Opus" appears in stems only as the prestige-bias distractor.

Rules that decide exam answers:

- **Select the lightest model that you can ship**. Evaluate from lighter models upward. Stop at the cheapest output that you would actually ship. The official sample question rewards a "faster, lower-cost model for straightforward, high-volume tasks."
- **Escalate on demonstrated failure, not prestige.** "Use the model with the most capability to maximize quality" is the standard wrong option. It wastes cost and latency budget.
- **Effort / extended thinking tune depth *inside* a model.** Use lower effort for quick lookups. Use higher effort for genuinely hard problems. Sometimes the right fix is more effort on Sonnet. It is not a jump to Opus.
- **Match the failure to the remedy.** Haiku misses nuance → move up. Sonnet is shallow on adversarial strategy → Opus or a better decomposition. Opus "fails" on cost, latency, or limits → move *down* or restructure. Bad brief → Fix the Description (Domain 02) before you change the model selection.
- **Try-two-models method.** Run the same known task on Haiku, then on Sonnet. Compare substance, not length. If Haiku already shows the claims you care about, stay with the light model.

### 4. Context limitations (restart / summarize / persist)

The context window is finite. Long threads degrade. You get drift, contradictions, and forgotten constraints.

- **Restart** when contradictions accumulate, roles drift, or quality collapses. A fresh thread plus a tight brief is better than a damaged thread.
- **Summarize** when you need decision continuity without the draft noise. Ask for a handoff brief (decisions, open questions, constraints). Start a new chat from it.
- **Persist** when reuse is predictable. Put standing rules in Project instructions. Put reference files in Project knowledge. Do not use a long first message every Monday.

Instructions compete with task tokens. Keep them short and stable. Put bulky policy in knowledge files with clear names. Ask by filename.

### 5. Memory (the persist layer built into the product)

claude.ai now has a first-class **Memory** feature (verified support.claude.com, Aug 2026). Details evolve. Re-check before exam day:

- Claude builds **memory entries** from your chats. Entries hold professional context, preferences, and ongoing work. Claude applies them in later conversations. Memory generation exists on **Free, Pro, and Max** (newer experience) and **Enterprise** (legacy synthesis experience). **Chat search** (Claude retrieves your past chats) is Pro and above.
- **Project-scoped:** each Project gets its own separate memory. Context from client A does not enter the memory of client B. This is a direct answer to the exam's "memory considerations" objective.
- **You control it:** view, edit, or delete individual entries, or reset all in Settings. Search and Memory toggle independently.
- Exclude **Incognito chats** (ghost icon, all plans, outside Projects) from history, memory, and search. Use them for one-off sensitive queries.

**Exam framing:** "restart, summarize, persist" is still the skill. Memory reduces how often you re-explain across chats. Memory is *not* a team knowledge base (it is per-user). Memory is *not* a substitute for Project knowledge. Curated files are better than inferred memories for policy questions. Memory is *not* guaranteed complete. Load-bearing context still belongs in Projects. Configuration and governance of Memory live in Domain 05 (see `05-configuration-knowledge-management.md` §Memory).

### 6. Plans and hidden constraints

Exam stems hide plan constraints in subordinate clauses. Solve the constrained world in the stem. Do not solve an unconstrained world that the stem does not give:

- Free: five-project cap. One custom connector. No Research.
- Research needs a paid plan **and** web search enabled.
- Enterprise Search needs Team/Enterprise with admin-connected sources.
- Team member ≠ admin: the admin enables. Authentication is usually per-user.
- Artifact viewers consume their own limits. Capabilities that you toggle off block artifact/code features.

### 7. Fluency bridge (4D)

**Delegation** is this whole domain. You select automation vs augmentation vs agency. You also select which feature and model carries it. **Description** includes the choice of Research vs chat, not just wording. **Discernment** applies hardest to heavy models. Long fluent Opus output still needs Domain 03 evaluation. **Diligence** treats these acts as a deployment: you publish an Artifact, or you enable a connector.

Wrong feature looks like a prompting failure. Wrong model looks like "Claude is dumb." Domain 07 asks which layer failed. Keep a one-page note per recurring workflow: Feature | Model | Context strategy.

### 8. Mini-cases (feature and model together)

**PMO Friday status:** Use a Project with template instructions and a Drive connector. Use Sonnet as the default.

**Board strategy interrogation:** Use Opus. Domain 03 evaluation is still mandatory.

**Support refund question:** Answer from Project policy or Enterprise Search. Never use general model knowledge.

**Workshop icebreakers:** Use chat plus Haiku or Sonnet. Skip Research.

**RFP compare:** Put the rubric in a Project. Use Research for the public landscape. Use code execution for computed scoring. Output as a table or Artifact.

**Quarter-end deck:** Upload the data. Use code execution for figures. Sonnet drafts commentary. A human validates.

Anti-patterns follow. Do not mix legal, HR, and marketing in one mega-Project. Do not use Research for a single uploaded PDF. Do not use write-capable connectors for read-only analysis. Do not use the top tier to increase creativity. Do not assume Artifact viewers use the creator's credentials or limits. Do not use Memory to hold a team's policy truth.

### 9. Multiple-response pattern bank

Select-TWO/THREE stems pair one right feature/model move with distractors. Practice this. Remove these wrong options:

- "Use Opus for everything important" (prestige bias)
- "Projects share your chat transcripts" (false mechanics)
- "Research works on Free" / "Research without web search" (plan/capability misses)
- "Memory replaces Project knowledge" (persist-layer confusion)
- "Have chat compute the 10,000-row totals" (wrong tool — code execution)

Correct combinations usually pair a **feature-fit** choice (Project / Research / code execution / Artifact) with a **constraint-respecting** choice (plan cap, auth step, lightest-shippable model).

---

## Decision trees

**Feature:** Reusable/recurring context → Project. Cited multi-source public investigation → Research. Org-internal "what did we decide" → Enterprise Search. Standalone interactive deliverable → Artifact. Repeatable procedure → Skill. Live system of record → Connector. Computation on data / file deliverable → code execution. One-off question → chat.

**Model:** Instant extract/classify → Haiku. Default knowledge work → Sonnet. Deep reasoning after a lighter attempt fails → Opus. Adjust effort before you move to a higher tier.

**Context:** A thread that drifts → restart. Need continuity, not noise → summarize into a handoff. Predictable reuse → persist in Project. Cross-chat personal context → Memory (check entries. Sensitive one-offs → incognito).

---

## Exam traps

1. Model with the most capability "to be safe" (prestige bias. The official sample question marks it wrong)
2. The belief that Project sharing shares your chat transcripts (it shares knowledge + instructions)
3. Research without web search enabled, or Research for one already-uploaded PDF
4. "Enabled = access": you forget per-user connector authentication
5. "Free has no Projects" (false — capped at five) / you ignore the Free one-custom-connector cap
6. You treat Artifacts as pre-validated because they look like products
7. Enterprise Search ≠ Research (internal cited lookup vs public agentic investigation)
8. You ask chat to "calculate" large-table arithmetic instead of code execution
9. You treat Memory as a shared team knowledge base or as a policy source of truth
10. You escalate the model when the brief, knowledge, or feature choice is what failed

---

## Practice Q&A (18) — scenario stems with answers and rationales

**Q1.** A marketing lead rebuilds the same brand-voice briefing every Monday. They paste the style guide into a new chat. What should they set up?
**A:** A Project with the style guide in knowledge and standing rules in instructions. Recurring context belongs in a persistent workspace. Do not paste it every week.

**Q2.** An ops team must pull totals and a pivot by region from a 40,000-row CSV. They must deliver an Excel file. What is the best approach?
**A:** Upload the file. Have Claude use code execution to compute the numbers and generate the spreadsheet. Chat-only "reading" of large tables risks arithmetic errors. Code execution runs real code. It produces the downloadable file.

**Q3.** A Free-plan user wants a sixth project for a new client. What is the reality?
**A:** Free accounts cap at five projects. Archive or merge one, or upgrade. (Plan caps change. Check at support.claude.com.)

**Q4.** An associate runs Research and gets nothing useful. Web search is toggled off. What is the first fix?
**A:** Enable web search. Research is an agentic web investigation and depends on it. Diagnose capability toggles before you blame the model.

**Q5.** A consultant wants Claude to remember their reporting format across chats. Client-confidential details must never carry over between two clients. What product behavior helps?
**A:** Memory is project-scoped. Each Project keeps separate memory. Keep each client in its own Project. General preferences can live in account-level memory. You can view, edit, or delete entries in Settings.

**Q6.** A teammate asks a one-off sensitive HR question. They do not want it in history or memory. What do you recommend?
**A:** An incognito chat (ghost icon). Exclude It from history, memory, and search. It is available on all plans, outside Projects.

**Q7.** Select TWO tasks that fit Haiku.
**A:** Classify inbound tickets by type. Extract fields from invoices. High-volume, well-specified, low-nuance work is the light-model best fit.

**Q8.** Sonnet's competitive analysis feels shallow on a genuinely multi-constraint strategy question after two solid iterations. What is the next move?
**A:** Escalate to Opus (or raise effort). A demonstrated failure on hard reasoning is the escalation trigger. Prestige is not the trigger.

**Q9.** Legal needs "what did we decide about data retention in Q2?" answered from Slack and Drive. What feature?
**A:** Enterprise Search — org-internal cited lookup across connected sources. Research is for the public landscape.

**Q10.** A 300-message thread now contradicts its own constraints. What is the best context move?
**A:** Ask for a handoff summary (decisions, constraints, open questions). Restart a fresh chat from it. Summarize, then restart. Do not continue a drifted thread.

**Q11.** An exec wants an interactive ROI calculator the sales team can use. What feature?
**A:** An Artifact (interactive app). Viewers consume their own usage limits. The calculator's formulas still need Domain 03 validation.

**Q12.** A Team-plan user shares a Project org-wide. They worry colleagues will read their draft chats. What is true?
**A:** Sharing distributes knowledge and instructions with view/edit permissions. Do not give Their own chats in the Project to every member as transcripts. (Check current sharing behavior on support.claude.com.)

**Q13.** Select TWO reasons the "always use the most capable model" policy fails.
**A:** It wastes cost and latency on tasks a light model ships. It substitutes prestige for the evaluate-upward method. (Quality on simple tasks does not improve enough to justify it.)

**Q14.** The team repeats a 12-step QBR-prep procedure monthly. They teach it again in chat each month. What feature?
**A:** A Skill — a packaged, reusable procedure. Project knowledge holds the reference material. The Skill holds the *how*.

**Q15.** A paid-plan Project's knowledge has grown near context limits. What keeps it usable?
**A:** Enhanced knowledge with RAG (paid plans). Retrieval expands capacity up to about 10×. Help it: name files clearly and ask with a narrow question.

**Q16.** A user says "Claude forgot everything from yesterday" in a fresh chat on a Pro plan. What is the diagnosis?
**A:** Check whether Memory/chat search is enabled. Check whether yesterday's chat was incognito or in a different Project's scope. Then decide. Persist standing context in a Project. Do not rely on recall.

**Q17.** A director asks whether Claude can draft board-pack prose "with zero hallucination risk if we buy Opus." What is the correct framing?
**A:** No tier eliminates hallucination. Model choice trades cost, speed, and quality. Validation (Domain 03) manages accuracy. Opus plus skipped review is worse than Sonnet plus a claim map.

**Q18.** Select TWO correct pairings of need → surface.
**A:** Live Drive data in answers → Connector. Downloadable PowerPoint from analyzed data → code execution/file creation. (Research is not for single uploaded files. Memory is not a team knowledge base.)

---

## Quick review checklist (pre-exam)

- [ ] Define every feature in one sentence, including code execution/file creation and Memory
- [ ] Free caps: five projects, one custom connector. RAG = paid, ~10×
- [ ] Model ladder Haiku/Sonnet/Opus + lightest-shippable + escalate-on-failure
- [ ] Effort tunes depth inside a model
- [ ] Restart / summarize / persist — and where Memory does and does not help
- [ ] Incognito = out of history, memory, search
- [ ] Research needs paid plan + web search. Enterprise Search needs admin-connected sources
- [ ] Connector pattern: admin enables, user authenticates

---

## Glossary

| Term | Meaning |
|---|---|
| **Project** | A workspace with instructions, knowledge, and its own chats |
| **RAG (in Projects)** | Paid-plan retrieval that expands knowledge capacity (~10×) near context limits |
| **Artifact** | A standalone editable/interactive deliverable beside the chat |
| **Skill** | A packaged reusable task procedure |
| **Connector (MCP)** | A live integration to an external system. Admin-enabled, user-authenticated |
| **Research** | Paid agentic multi-source investigation with citations |
| **Enterprise Search** | Org-internal cited search (Team/Enterprise) |
| **Code execution / file creation** | A sandboxed environment where Claude runs code and generates files (all plans) |
| **Memory** | Per-user, project-scoped remembered context. View/edit/delete in Settings |
| **Incognito chat** | Ghost-icon chat excluded from history, memory, and search |
| **Effort** | Depth control inside a model tier |
| **Context window** | Finite working memory of a conversation |
