---
title: Claude 101 — Exam-Prep Study Notes (Primary) — Simplified Technical English
source: https://academy.claude.com/courses/claude-101
disclaimer: Original study notes for exam prep — not official Anthropic material
approx_length: ~5500 words
deepened: 2026-08-23 — expanded for primary exam-study use
---

# Claude 101 — Study Notes

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names are exceptions and stay as written. The names include Claude, Cowork, Projects, Artifacts, Skills, Connectors, Enterprise Search, Research, Claude Code, Claude Tag, Claude Design, MCP, RAG, 4D, Delegation, Description, Discernment, and Diligence. Product UIs and plan gates change. Check current Help Center articles before the exam.
>
> **Disclaimer:** These notes are original study notes for exam prep. They are **not** official Anthropic material. Course outline source: https://academy.claude.com/courses/claude-101. Public Anthropic Help Center articles provide the feature details. Product UIs and plan gates can change. Check them before exam day.

## Overview

Claude 101 is Anthropic’s starter path. It helps you bring Claude into everyday work. Public course pages describe **13 lessons and 1 quiz** (about 2.5 hours). The path ends with a completion badge. The course is for anyone with a Claude account (Free through Enterprise). A few features (for example Cowork and Enterprise Search) need a paid plan or a Team/Enterprise plan for hands-on use. The ideas still matter for exams.

**What you can do after you study this material**

- Explain what Claude is and run a useful first conversation
- Write clearer prompts, iterate when results do not meet the goal, and connect prompting to the AI Fluency 4Ds.
- Select among three “shapes of work” in the desktop app: turn-by-turn Chat, handing work off (Cowork), and building software (Code)
- Use Projects, Artifacts, and Skills to organize context and repeatable work
- Expand reach with Connectors, Enterprise Search, and Research
- Map Claude to common roles and know adjacent products (Claude Code, Claude Tag, Claude Design, Claude for Microsoft 365, Claude in Chrome)

**Typical module arc (aligned to public outlines)**

1. Meet Claude — what it is. First conversation
2. Getting better results — prompting. Chat / Cowork / Code
3. Organizing work & knowledge — Projects, Artifacts, Skills
4. Expanding reach — Connectors, Enterprise Search, Research
5. Putting it together — role use cases. Other ways to work with Claude

These notes rewrite concepts in original language for reading and Claude cert prep. They are **not** a transcript of the course. The notes keep original study wording in expanded form. Do not treat this file as Academy lesson prose.

**How to use this file as a primary study source**

1. Read the Key concepts map one time
2. Read Deep notes in order (features build on each other)
3. Practice Decision trees / comparison tables until you can select the correct tool without notes
4. Read Common exam traps
5. Answer Self-check without looking at the answers, then check answers
6. Use the Quick review checklist the night before the exam
7. Pair with `02-ai-fluency.md` — Claude 101 applies the framework. Fluency explains it

---

## Key concepts map

| Idea | One-line meaning | Fluency link |
|------|------------------|--------------|
| Prompt pattern | Give role/stage, task, and rules (context, format, constraints) | Description |
| Iteration | The first output is a draft. Refine it with specific feedback. | Description–Discernment loop |
| Chat | Turn-by-turn collaboration in conversation | Augmentation default |
| Cowork | Hand off multi-step work with files/apps (plan-gated) | Agency-leaning automation |
| Code | Software-build shape of work | Implementation agency |
| Projects | Workspace with knowledge + instructions across chats | Standing Description |
| Artifacts | Standalone outputs in a side panel (docs, code, apps) | Visible deliverable |
| Skills | Reusable *how-to* procedures Claude can run | Process Description |
| Connectors | Links to web services or local desktop tools | Platform reach |
| Enterprise Search | Company-knowledge retrieval (Team/Enterprise) | Internal grounding |
| Research | Broader, multi-source investigation. Not a single chat guess. | Multi-step search agency |
| 4D Fluency | Delegation, Description, Discernment, Diligence | Whole practice |
| RAG (projects) | Retrieve relevant chunks when knowledge grows large | Capacity without overload |
| Progressive disclosure (skills) | Load only relevant skill bits when needed | Context hygiene |

**Memory peg:** *Shape → Context → Deliverable → Procedure → Reach*

- Shape = Chat / Cowork / Code
- Context = Projects (+ instructions)
- Deliverable = Artifacts
- Procedure = Skills
- Reach = Connectors / Enterprise Search / Research

---

## Deep notes by topic

### 1. What Claude is (and is not)

Claude is Anthropic’s AI assistant for writing, analysis, coding help, research, and collaborative problem-solving. Treat Claude as a **capable collaborator**. Do not treat Claude as an oracle. Claude predicts helpful responses from patterns in training. Claude also uses what you put in the current conversation and in connected context.

**Exam-style takeaway:** Confidence in tone is not the same as correctness. High-stakes facts need independent verification, citations, or web grounding.

**Practical tip:** Start every new domain with a short “contract”: goal, audience, constraints, and what “done” looks like. That sets Description (see AI Fluency). Do this before you try clever wording.

**Pitfall:** You send a vague request ("write something about Q3") and then blame the model. Vague input gives generic output.

**Worked example (original) — first contract**

Bad: “Help with onboarding.”

Better: “You are helping a new CS manager onboard. Deliver a 1-page checklist for week 1. Audience: managers who hate filler. Must include: access requests, buddy meeting, first ticket shadow. Avoid: motivational quotes. Ask me 3 clarifying questions before drafting if anything is ambiguous.”

**If the exam asks X, think Y**

- X: “Why did Claude invent a statistic?” → Y: LLMs generate fluent continuations. If you do not ground or tools they can invent facts. Verify.
- X: “Is Claude always right when confident?” → Y: No. Tone is not the same as truth. You must have Discernment.

---

### 2. Effective first conversations

A strong first message usually includes:

1. **Who you are / role** (or who Claude should act as)
2. **What you need** (concrete deliverable)
3. **Context** (audience, stakes, prior decisions)
4. **Format and limits** (length, structure, tone, must-include / must-avoid)

Public lesson material often uses a simple pattern such as **role + task + context + format**. Another pattern is a three-part frame of stage, task, and rules. Memorize the meaning of the pattern. Do not memorize a brand slogan: **orient → specify → constrain**.

**Self-check habit:** Before you send, ask “Could a smart new hire succeed with only this message?” If the answer is no, add context.

**Anti-patterns**

| Anti-pattern | Why it fails | Repair |
|--------------|--------------|--------|
| One-word asks | No deliverable | Name the artifact |
| Hidden constraints | Wrong tone/length | State must-avoid |
| Paste of whole email threads | Noise is larger than signal | Summarize stakes + paste only key excerpts |
| “Act as expert” with no task | Role without a task | Role + concrete output |
| Never iterating | You stay with draft 1 | Precise, targeted feedback |

**Worked example (original) — email rewrite**

Prompt skeleton:

```text
Role: You are my editor for customer emails.
Task: Rewrite the draft below into a calm apology.
Context: Customer waited 6 days; we missed an SLA; refund already approved.
Format: 120–150 words; subject line + body; no legalese; end with one clear next step.
Rules: Do not blame the customer; do not invent policy; flag any claim you cannot verify from my draft.
[paste draft]
```

---

### 3. Getting better results: common fixes

When output misses the goal, diagnose the failure mode:

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Too generic | Thin context | Add audience, constraints, history |
| Wrong length | Unstated expectation | Set word/paragraph limits |
| Wrong structure | Format not shown | Give a skeleton or example |
| Wrong tone | Default “helpful professional” | Name tone. Paste a sample |
| Plausible but wrong | Hallucination risk | Verify. Ask for sources/confidence. Use search/Research |
| Over-agreeable | Sycophancy | Ask for critique / opposing view |
| Thread chaos | Too many pivots | New chat with cleaned prompt |

**Iteration mindset**

- Treat draft 1 as a starting point
- Feedback must be **precise** (“cut para 1. Make CTA specific”). Do not only say “make better”
- If the thread is mixed and unclear, **start a clean chat** with a clearer prompt

**Lightweight evals (Discernment in practice)**

For work that repeats:

1. Collect 5–10 past good examples you already trust
2. Write prompts that recreate them
3. Compare Claude’s output to your gold examples
4. Record gaps. Add missing rules into prompts, Projects, or Skills
5. Only then trust the workflow on new data

This is the same idea as a “delegation diligence loop”: test on known answers before you trust new ones.

**Worked example (original) — eval for weekly metrics blurb**

- Gold set: last 4 Friday write-ups you already published
- Input: same raw CSV + same Project instructions
- Score: factual match, length, chart callouts, no invented drivers
- Gap found: Claude invents “seasonality” without data → add rule “Only claim drivers present in the numbers or my notes”
- Add the rule into Project instructions. Test one more time. Then use the workflow on live data.

**If the exam asks X, think Y**

- X: “Output is generic” → Y: Weak Description / missing audience
- X: “How do you improve without better ‘prompt unspecified mechanism’?” → Y: Iteration + evals + Projects/Skills

---

### 4. Three shapes of work (desktop app)

Public objectives stress three modes of collaboration:

1. **Turn by turn (Chat)** — You draft, ask, and analyze in conversation, one turn at a time. Best when you want tight control each step.
2. **Handing work off (Cowork)** — Longer sessions where Claude works with files and applications with more autonomy. Best for multi-step operational tasks. Often plan-gated (paid plans). Sessions can organize files, synthesize research, run scheduled tasks, and use projects scoped for Cowork.
3. **Building software (Code / Claude Code)** — Implementation-oriented work: repos, agents, coding workflows.

**Exam tip:** Match the *shape of the task*. Do not select a mode only because it looks new. Exploratory idea work → Chat. “Clean this folder of invoices and summarize” → Cowork-style handoff. “Ship a feature in the repo” → Code.

**Pitfall:** You use agentic handoff for a task that still needs turn-by-turn judgment (sensitive client email, legal wording).

**Decision tree — select a shape**

```text
Is the primary goal writing/judging tone sentence-by-sentence?
 YES → Chat
 NO ↓
Is the primary goal multi-step file/app ops you can leave running?
 YES → Cowork (if plan allows)
 NO ↓
Is the primary goal software in a repo / coding agent workflow?
 YES → Code
 NO → Default Chat, then escalate if the task grows
```

**Comparison — Chat vs Cowork vs Code**

| Dimension | Chat | Cowork | Code |
|-----------|------|--------|------|
| Control per step | Highest | Medium (plan then run) | Medium–high when you steer |
| Best for | Drafts, Q&A, judgment | Ops across files/apps | Implementation |
| Autonomy | Low | Higher | Higher in repo |
| Plan gates | Broadest access | Often paid | Separate coding surface |
| Risk if misused | Wasted time | Side effects without review | Bad commits / scope grows without control |
| Diligence focus | Review text | Review actions + files | Review diffs + tests |

**Anti-patterns**

- You start Cowork for a one-paragraph rewrite
- You use Code mode to “think about strategy” with no repo
- You leave Cowork without review on irreversible actions, and you set no checkpoints

---

### 5. Projects — knowledge hubs

**Projects** are dedicated workspaces with:

- Their own chat histories
- **Project knowledge** (uploaded docs Claude can reuse)
- **Project instructions** (tone, expertise, standing rules)
- Optional team sharing of the same shared context (Team/Enterprise)

Public Help Center framing: Projects are available broadly (including Free, with a small project count limit on Free). On paid plans, large knowledge bases can shift into **RAG mode**. Claude retrieves relevant chunks. Claude does not put everything into the window. This expands capacity while it keeps quality.

**Use Projects when:** You have a stable domain (product docs, brand voice, client briefs) that many chats should share.

**Do not confuse Projects with Skills:** Projects store *what* Claude should know. Skills encode *how* to run a procedure. Help Center contrast: Projects = static background knowledge loaded in that workspace. Skills = specialized procedures that activate dynamically and can work across Claude.

**Worked example (original) — “Acme Support” Project**

- Knowledge: FAQ PDF, severity matrix, refund policy, tone samples
- Instructions: “Never invent policy. Quote policy titles. Escalate legal claims to human. Tone: calm, plain.”
- Chats inside Project: ticket rewrites, macro drafts, weekly theme summaries
- Outside Project: do **not** expect the same policy grounding

**Anti-patterns**

- One Project for every client mixes client contexts.
- You upload secrets that you would not put in a shared drive
- You write no instructions, then you blame Claude: “Claude forgot our voice”
- You treat Project knowledge as always fully loaded when RAG may retrieve selectively. Ask pointed questions. Cite files.

**If the exam asks X, think Y**

- X: “How do I reuse brand voice across chats?” → Y: Project knowledge + instructions
- X: “Knowledge got huge—what happens conceptually?” → Y: Retrieval of relevant chunks (RAG-style), especially on paid plans

---

### 6. Artifacts — standalone outputs

**Artifacts** appear in a dedicated panel beside chat. Claude does not hide a long HTML, code, or document output in the thread. Claude renders something you can preview, copy, download, or iterate on.

Claude tends to create an artifact when content is significant, self-contained, editable/reusable, and likely to be referenced later. The content is often longer or more structured than a short chat reply.

Common types (conceptually):

- Documents (markdown/text reports, plans, posts)
- Code snippets ready to copy
- Interactive / visual pieces (simple apps, charts, calculators, SVG/diagrams, React components)

**Tips**

- Ask explicitly: “Create this as an artifact” if the content stays stuck in chat
- Toggle preview vs underlying code when you debug
- Use Artifacts for shareable deliverables. Keep chat for negotiation and critique
- Version selector helps you explore directions without losing prior work
- Publishing/sharing and advanced behaviors (AI-powered artifacts, MCP inside artifacts, persistent storage) depend on plan and settings. Know that they exist. Check live docs for exam specifics
- Code execution / file creation capabilities are often required for modern artifact workflows

**Worked example (original)**

Ask for a “pricing comparison calculator” as an artifact. Iterate on UI in the panel. Download or share when the artifact is stable. Keep negotiation (“should we include annual discount?”) in chat.

**Anti-patterns**

- You leave a 200-line HTML app only in chat bubbles
- You paste the whole file again every time when you edit. You do not target the artifact.
- You publish without a check of personal vs shared storage (privacy)

---

### 7. Skills — procedural machines

**Skills** are reusable instruction sets (often folders of instructions, scripts, and resources) for multi-step workflows. Examples include brand process, PDF packaging, call-preparation procedure, and blog pipeline. Skills improve consistency via **progressive disclosure**. Claude loads what is relevant. Claude does not load every skill into context.

Types to recognize at a high level:

- Anthropic-maintained skills (e.g., stronger document creation flows)
- Custom skills you author (Markdown instructions. Optional scripts)
- Organization-provisioned skills (Team/Enterprise)
- Partner skills in a directory (often paired with MCP connectors)

**Projects vs Skills (exam favorite)**

| | Projects | Skills |
|--|----------|--------|
| Purpose | Store knowledge | Define processes |
| Best for | Long-term context, team reference | Repeatable methodology |
| Persistence | Across chats in the project | When the skill is invoked / relevant |
| Metaphor | Library + standing rules | Playbook / SOP |
| Loading | Background in that workspace | Dynamic / on-demand |

They combine well. A “customer call prep” Skill can pull profiles from a Project’s knowledge base.

**Skills vs Connectors/MCP (short)**

- Connectors/MCP = *access* to tools and data
- Skills = *how* to use those tools for a workflow
- Use both: MCP opens access. Skills teach the workflow.

**Skills vs custom instructions**

- Custom instructions = broad, always-on preferences
- Skills = task-specific, loaded when relevant

**Pitfall:** You encode single-use trivia as a Skill. Or you put a giant SOP into a Project with no procedure.

**Worked example (original) — “Weekly status” Skill**

Procedure steps: pull themes from notes → group by risk → draft bullets → ask a human to confirm blockers → format for Slack. Attach the Skill only if the procedure repeats weekly.

**If the exam asks X, think Y**

- X: “Same procedure every time across chats” → Y: Skill
- X: “Same docs and voice every time in a workspace” → Y: Project
- X: “Need live Asana data” → Y: Connector/MCP (+ Skill for the workflow)

---

### 8. Connectors — expanding reach

**Connectors** link Claude to external tools.

- **Web connectors** — Cloud apps (Drive, Notion, Slack, Asana, Gmail, etc.), often via MCP-style integrations
- **Desktop extensions** — Local files and native apps via the desktop app

Setup pattern: find in directory → connect → authenticate → grant least privilege → test with “Can you see X?”

**Diligence angle:** Install only trusted connectors. Permissions are revocable. Treat connected data as in-scope for privacy reviews.

**Anti-patterns**

- You connect everything without a current need
- You ask Claude to email externally without an approval gate
- You assume connector access equals unlimited org-wide visibility (auth and scopes matter)

---

### 9. Enterprise Search and Research

- **Enterprise Search** — Reach into company knowledge (typically Team/Enterprise). It is often a dedicated org knowledge workspace (for example, an “Ask Your Org”-style project). It has guided connectors and search-optimized behavior. Use it when the answer is in internal docs, not the open web. It is optimized for relatively quick internal retrieval with citations.
- **Research** — Multi-step, agentic investigation across sources when a single chat guess is not enough. Research is good for literature-style or competitive scans with grounding. It is typically on a paid plan. Web search must be available. Research can use web plus connected internal context. It produces thorough answers with checkable citations. It can consume usage limits faster.

**Compare (exam favorite)**

| | Enterprise Search | Research | Ordinary web search in chat |
|--|-------------------|----------|-----------------------------|
| Main job | Find internal knowledge fast | Deep multi-step synthesis | Quick external facts |
| Typical sources | Connected company tools | Web + integrations, many steps | Web |
| Depth | Targeted retrieval | Broad investigation | Light |
| Plan notes | Team/Enterprise | Paid plans common | Depends on settings |

**Exam tip:** Use Research/Search when freshness or source grounding matters. Still apply Discernment. Sources can be incomplete or conflicting.

**Worked example (original)**

- “Where is our PTO policy?” → Enterprise Search / internal connectors
- “Compare three competitors’ pricing pages and summarize changes since last quarter” → Research
- “What time is the Singapore GP this year?” → Web search in chat may suffice

**If the exam asks X, think Y**

- X: “Internal Slack + Drive answer” → Y: Enterprise Search / connectors
- X: “Multi-angle report with citations in minutes” → Y: Research
- X: “One quick public fact” → Y: Web search, not full Research

---

### 10. Role-based use and product tour

Course wrap-ups typically show Claude across roles (marketing calendars, follow-ups, analysis, ops). They also point to specialized surfaces:

- **Claude Code** — coding agents / software work
- **Claude Tag / Design** — tagged or design-oriented workflows (product naming may evolve. Know they exist as adjacent tools)
- **Claude for Microsoft 365** — work inside M365 surfaces
- **Claude in Chrome** — browser-context assistance

For exams: know **when** each shape helps. You do not need every UI click.

**Role mapping cheat sheet (original)**

| Role need | Prefer |
|-----------|--------|
| Draft + edit copy | Chat + Artifacts |
| Recurring brand SOP | Skill (+ Project knowledge) |
| Clean folders / multi-app ops | Cowork |
| Ship code | Code |
| Company wiki answers | Enterprise Search |
| Competitive landscape brief | Research |
| Browser page help | Claude in Chrome |
| Outlook/Teams embedded help | Claude for Microsoft 365 |

---

### 11. Mapping Claude 101 → AI Fluency 4Ds

| Feature / habit | Strongest D | Why |
|-----------------|-------------|-----|
| Select Chat vs Cowork vs Code | Delegation | Task split + platform awareness |
| Prompt contracts, Projects, Skills | Description | Goals, process, behavior |
| Evals, citation checks, “fix with Claude” | Discernment | Critique outputs/process |
| Connector least privilege, disclosure, human owns the final | Diligence | Responsibility |

Remember: Features are **infrastructure**. They are not a substitute for judgment.

---

## Decision trees / comparison tables

### Master chooser — “I need Claude to…”

```text
Need reusable domain knowledge across many chats?
 → Project (+ instructions)
Need a repeatable multi-step method?
 → Skill
Need a tangible deliverable to preview/share?
 → Artifact
Need live external/app data?
 → Connector / MCP
Need internal company answer quickly?
 → Enterprise Search
Need deep multi-source investigation?
 → Research
Need tight editorial control?
 → Chat
Need hands-off multi-step ops?
 → Cowork
Need repo/implementation work?
 → Code
```

### Projects vs Skills vs Artifacts vs Connectors

| Need | Reach for |
|------|-----------|
| What Claude should know here | Project |
| How Claude should run a procedure | Skill |
| What Claude produced as a thing | Artifact |
| What systems Claude can touch | Connector |
| Where company answers are kept | Enterprise Search |
| How to investigate broadly | Research |

### Failure → feature map

| Failure | First fix |
|---------|-----------|
| Generic answers | Better prompt / Project context |
| Inconsistent weekly procedure | Skill |
| Lost long HTML in scrollback | Artifact |
| Cannot see Drive/Slack | Connector |
| Invented market numbers | Research/web + verify |
| Wrong tool for autonomy level | Select Chat/Cowork/Code again |

---

## Common exam traps

1. **Projects = Skills** — False. Knowledge/workspace vs procedure.
2. **Artifacts are just pretty chat** — False. Artifacts are standalone panel deliverables.
3. **More autonomy = less oversight** — False. Cowork/Code/Research need *more* Discernment and Diligence.
4. **Confident tone = verified fact** — False.
5. **Free plan can do every hands-on demo** — False. Cowork / Enterprise Search / some Research behaviors are plan-gated. Definitions are still testable.
6. **Connectors replace Skills** — False. Access is not the same as procedure.
7. **Enterprise Search = Research** — False. Quick internal retrieval vs deep multi-step investigation.
8. **Custom instructions = Skills** — False. Always-on broad prefs vs task-specific loaded procedures.
9. **RAG means Claude read every file fully every time** — Misleading. Retrieval pulls relevant chunks when knowledge is large.
10. **One Project for all clients is fine** — Risky. Client contexts mix. Prefer separate Projects for sensitive matters.
11. **“Make it better” is good feedback** — Weak. Give precise, targeted feedback.
12. **Publishing an artifact has no privacy angle** — False. Shared storage / org sharing rules matter.
13. **Fluency is only prompt tricks** — False. Fluency is a 4D collaboration skill (see course 02).
14. **You can skip human accountability if Claude drafted it** — False. You own the work product.

---

## Expanded self-check (Q&A)

**Q1.** A teammate says Projects and Skills are the same because both “give Claude context.” How do you correct them?

**A1.** Projects mainly hold reusable *knowledge and standing instructions* across chats in a workspace. Skills encode *repeatable procedures*. Claude invokes or loads a Skill when it is relevant. Knowledge is not the same as process. They complement each other.

**Q2.** You need a one-page client apology email and must control every sentence. Which desktop “shape of work” fits best and why?

**A2.** Chat (turn-by-turn). High judgment and tone control favor conversational iteration. Do not use a long handoff.

**Q3.** Claude’s market stats look polished but you cannot verify them. What should you do?

**A3.** Treat the stats as possibly wrong. Verify them independently. Ask for sources and confidence. Enable web or Research grounding for current facts.

**Q4.** Name one symptom of weak Description and one fix.

**A4.** Example: generic output → add audience, constraints, and a format example.

**Q5.** What is a lightweight eval for a weekly metrics write-up?

**A5.** Take last month’s known-good write-ups. Prompt Claude on the same raw inputs. Compare the output to your originals. Then add missing rules into the prompt, Project, or Skill. Do this before you use the workflow on new data.

**Q6.** When project knowledge gets huge, what conceptual shift helps capacity?

**A6.** Shift from loading all files into context to retrieving only relevant chunks (RAG-style). Answers stay grounded. The window does not overflow. Paid plans emphasize this.

**Q7.** Web connector vs desktop extension—key difference?

**A7.** Web connectors link cloud services. Desktop extensions run locally through the desktop app for files and native apps.

**Q8.** Map “I am accountable for the final report and will disclose AI help if asked” to which Fluency D?

**A8.** Diligence (responsibility, transparency, ethical use).

**Q9.** When should you prefer Research over a single web-search answer?

**A9.** Prefer Research when you need multi-step, multi-source synthesis with systematic exploration and citations. Do not use full Research for one quick fact.

**Q10.** How does Enterprise Search differ from a normal Project?

**A10.** Enterprise Search is specialized for org knowledge retrieval. It is often auto-provisioned, search-optimized, and paired with guided connectors. Normal Projects are general-purpose workspaces. You can customize their instructions and knowledge.

**Q11.** Give an example of a wrong use of Cowork.

**A11.** You hand off a sensitive legal letter that still needs sentence-level human judgment, and you set no tight checkpoints. Or you run irreversible file operations with no review.

**Q12.** Skills vs MCP connectors in one sentence each.

**A12.** Skills teach *how* to perform a workflow. MCP/connectors provide *access* to external tools and data.

**Q13.** What should you put in Project instructions vs Project knowledge?

**A13.** Instructions = standing rules/tone/role behavior. Knowledge = reference files and facts Claude should draw from.

**Q14.** Name three common artifact content types.

**A14.** Any three: documents, code snippets, HTML pages, SVG/diagrams, interactive components/calculators.

**Q15.** Why start a new chat instead of a messy thread that continues?

**A15.** Mixed, unclear context causes drift. A clean prompt with clarified goals often works better than endless patches.

**Q16.** Map Artifacts to a Fluency idea.

**A16.** Artifacts make the *product* of Description visible and editable. This supports Discernment on a concrete deliverable.

**Q17.** Free-plan common exam error: which ideas still matter even if you cannot click every feature?

**A17.** Definitions and fit-for-task reasoning for Cowork, Enterprise Search, Research, and similar features. The exam can still test these ideas.

**Q18.** What is progressive disclosure in Skills?

**A18.** Claude loads only relevant skill information for the task. This reduces context overload.

**Q19.** You need Claude to follow the same PDF packaging steps every Friday. Project or Skill—primary choice?

**A19.** Skill (procedure). Optionally keep brand assets in a Project.

**Q20.** “AI wrote it, so I’m not responsible.” Which D fails?

**A20.** Diligence—humans remain accountable for AI-assisted work.

**Q21.** Pick a tool: summarize themes across connected Slack + Drive for “what did we decide about pricing?”

**A21.** Enterprise Search / org knowledge connectors (internal). Use Research if you also need deep external competitive synthesis.

**Q22.** What feedback is better than “make it better”?

**A22.** Precise, targeted instructions: what to cut, what to add, tone target, length, must-keep facts.

**Q23.** Why separate client matters into different Projects?

**A23.** Reduce context bleed. Keep histories, files, and instructions separate for accuracy and confidentiality.

**Q24.** Code vs Chat for “explain this stack trace and propose a patch,” when you are not in a repo agent workflow?

**A24.** Chat can analyze and propose. Switch to Code when you want agentic implementation in the actual repository.

**Q25.** How do you combine Project + Skill + Artifact in one workflow?

**A25.** Example: the Project holds product docs. The Skill runs the “release notes ritual.” The Artifact is the final markdown notes that you preview and share.

---

## Quick review checklist

- [ ] I can explain Claude as a collaborator, not an oracle
- [ ] I can write orient → specify → constrain first messages
- [ ] I can diagnose generic / wrong tone / hallucination and fix them
- [ ] I can select Chat vs Cowork vs Code with a reason
- [ ] Projects = knowledge + instructions. Skills = procedures. Artifacts = deliverables
- [ ] Connectors = access. Skills = how. Do not mix these ideas
- [ ] Enterprise Search (internal quick) ≠ Research (deep multi-step)
- [ ] RAG / progressive disclosure = capacity without putting every file into context
- [ ] Plan gates exist. Definitions still matter on exams
- [ ] Map features to Delegation / Description / Discernment / Diligence
- [ ] Human owns the final work product
- [ ] I reviewed anti-patterns and exam traps once aloud

---

## Glossary

| Term | Short definition |
|------|------------------|
| Artifact | Standalone, previewable output in a side panel |
| Chat | Turn-by-turn conversational shape of work |
| Connector | Integration that links Claude to external or local tools |
| Cowork | Hands-off / multi-step operational shape of work (often paid) |
| Code (shape) | Software-build collaboration surface |
| Custom instructions | Broad always-on preferences |
| Enterprise Search | Org-knowledge retrieval workspace/capability |
| Hallucination | Fluent but false content |
| MCP | Model Context Protocol — tool/data connection standard |
| Progressive disclosure | Load only relevant instructions/resources |
| Project | Workspace with chats, knowledge, instructions |
| Prompt contract | Goal, audience, constraints, definition of done |
| RAG | Retrieval Augmented Generation — fetch relevant chunks |
| Research | Agentic multi-step investigation with citations |
| Skill | Reusable procedural how-to package |
| Sycophancy | You agree too much instead of seeking truth |
| 4Ds | Delegation, Description, Discernment, Diligence |

---

## Reading tips

- Read the Key concepts map before Deep notes
- After each deep topic, answer one self-check aloud
- Practice the master chooser decision tree from memory
- Pair with the AI Fluency notes (`02-ai-fluency.md`)—Claude 101 applies the framework. Fluency explains it
- Night before: checklist + glossary only

*End of Claude 101 study notes.*
