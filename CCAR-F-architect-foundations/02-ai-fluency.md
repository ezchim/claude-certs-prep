---
title: AI Fluency Framework & Foundations
source: https://academy.claude.com/courses/ai-fluency-framework-foundations
collection: https://academy.claude.com/collections/ai-fluency
disclaimer: Original study notes for exam prep — not official Anthropic material
approx_length: ~5800 words
deepened: 2026-08-23 — expanded for primary exam-study use
---

# AI Fluency: Framework & Foundations — Study Notes

> **Disclaimer:** These are original study notes for exam prep — **not** official Anthropic material. Course outline source: https://academy.claude.com/courses/ai-fluency-framework-foundations. Collection context: https://academy.claude.com/collections/ai-fluency. Framework origin credited publicly to Rick Dakan and Joseph Feller with Anthropic collaboration; terminology here is paraphrased for study, not copied lesson prose.

## Overview

**AI Fluency: Framework & Foundations** is Anthropic’s flagship fluency course (public pages: **14 lessons · 1 quiz · ~4 hours**, completion badge after assessment). It grew from collaboration between Anthropic educators and professors **Rick Dakan** (Ringling College of Art and Design) and **Joseph Feller** (University College Cork), who developed the AI Fluency Framework in 2023–2024.

The course is less about “prompt tricks of the month” and more about a durable way to collaborate with AI: **effectively, efficiently, ethically, and safely**—the four “E/S” outcomes often paired with the 4D competencies.

**Stated learning outcomes (paraphrased from the public course page)**

- Define AI Fluency and why it matters
- Apply the **4D framework**: Delegation, Description, Discernment, Diligence
- Distinguish **Automation, Augmentation, and Agency** as engagement modes
- Explain generative AI / LLM basics, strengths, and limits
- Make Delegation choices using problem awareness, platform awareness, and task split
- Communicate via Product / Process / Performance Description and solid prompting
- Critique outputs and refine through the **Description–Discernment loop**
- Practice Creation, Transparency, and Deployment Diligence

These notes are original study material for reading and Claude cert prep—not official Anthropic content and not a lesson transcript.

**How to use this file as a primary study source**

1. Memorize the Key concepts map (10–15 minutes)
2. Read Deep notes on the 4Ds and the three engagement modes twice
3. Drill Decision trees until you can label scenarios cold
4. Walk Common exam traps aloud
5. Complete Self-check closed-book, then verify
6. Night-before: Quick review checklist + glossary
7. Pair with `01-claude-101.md` to connect framework → product features

**Public diligence modeling note (concept, not a quote bank):** The course materials themselves demonstrate Transparency Diligence by disclosing how Claude assisted drafting while humans retained judgment and final responsibility. Use that as a *pattern* for your own disclosures—not as text to memorize verbatim.

---

## Related titles in the AI Fluency collection (brief)

Skim these so exam questions about “where to go next” do not surprise you. Source collection: https://academy.claude.com/collections/ai-fluency

| Title | Why it matters |
|-------|----------------|
| **The 4 Properties of AI** (short tutorial) | Fast mental model of what makes AI strong or weak |
| **AI Capabilities and Limitations** (full course) | Deeper model of next-token prediction, knowledge, memory, steerability, context limits |
| **4 Ds — Behavioral Indicators** | Concrete day-to-day behaviors tied to each D |
| Short trust tutorials | “What happens when you talk to AI?”, trust, privacy, hallucination, sycophancy, bias |
| **Writing an AI diligence statement** | Practice Transparency Diligence |
| **Tokens** / **How context affects performance & cost** | Practical cost–quality tradeoffs |
| Editions “for your world” | Builders, educators, pK–12, nonprofits, small businesses, students |
| Teach & facilitate tracks | Teaching AI Fluency; train-the-trainer; Education Report / Fluency Index guides |

Flagship first; use the collection as a map of adjacent skills.

---

## Key concepts map

| Term | Short definition | Core question |
|------|------------------|---------------|
| AI Fluency | Skillful collaboration with AI: effective, efficient, ethical, safe | Can I work *with* AI well? |
| Delegation | Decide whether/when/how to engage AI; split work | Who does which part? |
| Description | Communicate goals, process, and desired behavior | How do I say what good looks like? |
| Discernment | Critically evaluate outputs, process, and behavior | How good is this, really? |
| Diligence | Responsibility for creation, transparency, deployment | What am I accountable for? |
| Automation | AI executes specified tasks on instruction | “Do this.” |
| Augmentation | Human + AI co-create / co-think | “Work with me.” |
| Agency | You shape AI to act more independently on your behalf | “Act toward these aims.” |
| Description–Discernment loop | Describe → output → critique → re-describe | How do we improve? |
| Problem awareness | Understand real goal, risks, success criteria | What’s actually at stake? |
| Platform awareness | Know this tool’s strengths/limits | What can *this* system do? |
| Task delegation | Slice workflow into human / shared / AI-primary | Keep / share / give? |
| Product Description | Specify the deliverable | What should exist at the end? |
| Process Description | Specify how work should proceed | What steps/tools/stop points? |
| Performance Description | Specify interaction behavior | How should AI behave with me? |
| Creation Diligence | Care while building with AI | Is the quality bar right? |
| Transparency Diligence | Disclose AI assistance where it matters | Who needs to know AI helped? |
| Deployment Diligence | Care before releasing into the world | Who could be harmed if this ships? |

**Memory pegs**

- Outcomes: **Effective · Efficient · Ethical · Safe**
- Competencies: **Delegate · Describe · Discern · Diligent**
- Modes: **Automate · Augment · Agency**
- Description layers: **Product · Process · Performance**
- Delegation lenses: **Problem · Platform · Task**
- Diligence strands: **Creation · Transparency · Deployment**

---

## Deep notes by topic

### 1. Why AI Fluency (not just prompting)

Tools change quickly. Fluency is a **stable competency layer**: judgment about when to use AI, how to steer it, how to check it, and how to own the result. Without it, people either over-trust fluent nonsense or under-use valuable help.

**Exam framing:** Fluency is about *thinking with* AI, not treating it as a fancy spellchecker.

**What fluency is not**

- Memorizing model version numbers
- Collecting viral prompt templates alone
- Blind trust in polished paragraphs
- Avoiding AI entirely out of fear without learning evaluation

**Worked contrast (original)**

- Prompt-only user: pastes a clever template; ships first draft; surprises stakeholders with errors
- Fluent user: decides the split (Delegation), writes a clear contract (Description), checks against criteria (Discernment), discloses and owns the result (Diligence)

**If the exam asks X, think Y**

- X: “Why not just learn prompting?” → Y: Prompting ⊂ Description; fluency also covers whether/when, evaluation, and responsibility

---

### 2. The 4D Framework

Remember with a story arc:

1. **Delegation** — Should AI be in this work at all, and on which parts?
2. **Description** — How do I tell it what good looks like?
3. **Discernment** — How do I tell if the result (and the process) is good?
4. **Diligence** — How do I stay responsible as I create, disclose, and ship?

The Ds are **interconnected**. Weak Description breaks Discernment (“I don’t know what I asked for”). Weak Discernment poisons Delegation (“I’ll automate something I never validated”). Diligence wraps the whole practice.

#### Delegation

Thoughtfully decide what humans do, what AI does, and how they hand off. Public framing emphasizes setting goals and deciding whether, when, and how to engage with AI.

Helpful sub-skills called out in objectives:

- **Problem awareness** — Understand the real goal, constraints, risks, and success criteria
- **Platform awareness** — Know what *this* model/tool can and cannot do (and plan limits)
- **Task delegation** — Split the workflow: draft vs decide, clean vs interpret, explore vs approve

**Practical tip:** For any recurring task, write a one-sentence “keep / share / give” list: keep (must stay human), share (pair), give (AI-primary with review).

**Pitfall:** Delegating interpretation of high-stakes outcomes while only checking formatting.

**Worked example (original) — hiring scorecard**

- Keep: final hire decision, legal compliance calls
- Share: structuring rubrics, clarifying role requirements
- Give: formatting candidate comparison tables from notes you already wrote
- Checkpoint: human verifies every factual claim about a candidate against source notes—no invented experience

#### Description

Clear communication with AI systems. Public framing often splits:

- **Product Description** — What should the *output* be? (audience, structure, examples of good/bad)
- **Process Description** — How should AI *work*? (steps, tools, order, stop points)
- **Performance Description** — How should AI *behave* in the interaction? (tone, ask-before-assuming, cite uncertainty)

Foundational prompting lives here: role/context, task clarity, constraints, examples, and iteration instructions.

**Pitfall:** Only describing the product (“write a strategy”) with no process or performance rules—then being surprised by confident overreach.

**Mini templates (original wording)**

```text
PRODUCT: Deliverable, audience, length, must-include, must-avoid, example of good.
PROCESS: Steps to follow, tools to use, when to ask me questions, what to cite.
PERFORMANCE: Tone, uncertainty handling, challenge my assumptions if weak.
```

#### Discernment

Evaluate:

- **Outputs** — accuracy, completeness, usefulness, tone fit
- **Processes** — Did the approach make sense? Missing steps? Invented data?
- **Behaviors** — sycophancy, overconfidence, bias patterns, refusal quality

Build eval habits: compare to known-good work; spot-check facts; ask the model to show reasoning or sources *then verify*.

**Pitfall:** Mistaking “I like how this sounds” for “this is correct.”

#### Diligence

Responsible use, often taught as three strands:

- **Creation Diligence** — Care in how you build with AI (quality bar, appropriate use)
- **Transparency Diligence** — Disclose AI assistance where it matters (clients, academia, teammates)
- **Deployment Diligence** — Care before releasing AI-touched work into the world (safety, fairness, compliance)

**If the exam asks X, think Y**

- X: “Which D is missing if I automate before validating?” → Y: Discernment (and poor Delegation)
- X: “Which D is disclosure?” → Y: Transparency Diligence
- X: “Pretty but wrong citations” → Y: Discernment failure (+ Diligence if published)

---

### 3. Three ways of engaging: Automation, Augmentation, Agency

| Mode | Human role | AI role | Example |
|------|------------|---------|---------|
| **Automation** | Specify task; review | Execute defined work | “Format these rows into a table” |
| **Augmentation** | Co-think, co-edit | Partner on ideas and drafts | Strategy brainstorm + critique |
| **Agency** | Set goals, knowledge, guardrails | Work more independently toward aims | Ongoing research agent with standing instructions |

**Exam tip:** Agency is not “set and forget forever.” You still shape knowledge and behavior, then apply Discernment and Diligence on outcomes. More autonomy → **more** need for clear Description and stronger checks—not less.

**Pitfall:** Calling something “automation” when you actually need augmentation (judgment-heavy creative work).

**Decision tree — pick a mode**

```text
Is the task fully specified with a clear success check?
 YES → Automation (+ review)
 NO ↓
Do you need ongoing co-thinking and judgment tradeoffs?
 YES → Augmentation
 NO ↓
Are you configuring standing goals/tools so AI can pursue work with less turn-by-turn steering?
 YES → Agency (with stronger guardrails + monitoring)
 ELSE → Start in Augmentation; tighten into Automation once stable
```

**Anti-patterns by mode**

| Mode | Anti-pattern | Better |
|------|--------------|--------|
| Automation | Vague task, no acceptance test | Specify + sample output |
| Augmentation | Accepting flattery as collaboration | Demand critique criteria |
| Agency | Unbounded autonomy on email/send | Guardrails, approvals, logs |

---

### 4. Generative AI & LLM fundamentals (mental model)

You do not need a PhD for the quiz; you need a **usable mental model**:

- Modern assistants are built on **large language models** trained to predict useful next tokens in context
- They combine **learned patterns** with **current context** (your messages, files, tool results)
- Strengths often include: drafting, transforming, summarizing, coding assistance, structured brainstorming, explaining
- Limits often include: inventing plausible falsehoods, weak live knowledge without tools, context window bounds, uneven niche expertise, sensitivity to framing

Related collection pieces deepen: next-token prediction, working memory, steerability, tokens, and cost/quality tradeoffs.

**Delegation link:** Platform awareness means matching task type to these strengths/limits.

**Trust topics to recognize (collection)**

- **Hallucination** — Fluent but false content; spot-check, ground with sources/tools
- **Sycophancy** — Agreeing with you because you seem to want it; ask for critique explicitly
- **Bias** — Skewed associations from data/systems; watch stereotypes and skewed recommendations
- **Privacy** — What you paste may be processed; minimize sensitive data; follow org policy

**Tokens & context (exam-aware, light)**

- Usage is measured in tokens; longer context can raise cost and sometimes dilute attention
- Everything Claude “knows” for your answer is either baked into training or present in current context/tools
- Be intentional: more context is not always better—relevance beats dump

**Worked example (original) — platform awareness**

Task: “What’s our CEO’s exact quote from yesterday’s all-hands?”
- Without connectors/transcripts: high hallucination risk → don’t Delegate as factual recall
- With meeting notes in a Project or Enterprise Search: better platform fit
- Still Discern: quote must match source text

---

### 5. Strategic Delegation in projects

When planning a project with AI:

1. Clarify the **problem** (who hurts if this is wrong?)
2. Inventory **platform** capabilities (chat only? tools? company search? coding agent?)
3. Slice **tasks** into AI-suitable vs human-required
4. Define checkpoints where Discernment is mandatory
5. Write Diligence rules (disclosure, data handling, approval gates)

**Worked micro-example (original):** Publishing a customer FAQ update.

- Human keeps: legal claims, pricing promises, escalation policy
- AI drafts: rewrites from ticket themes; proposes structure
- Checkpoint: human verifies every factual claim against source docs
- Disclosure: internal note that AI assisted drafting

**Second worked example (original):** Student literature survey

- Problem awareness: academic integrity rules; originality required
- Platform awareness: model may invent citations
- Task split: AI helps outline and explain papers you already found; human reads sources and writes claims
- Diligence: follow school disclosure policy; never cite unread papers

**Keep / share / give worksheet (original)**

| Step in workflow | Keep | Share | Give | Checkpoint |
|------------------|------|-------|------|------------|
| Define success | ✓ | | | Stakeholder sign-off |
| Gather sources | | ✓ | | Source list review |
| Draft synthesis | | ✓ | | Fact audit |
| Format tables | | | ✓ | Spot-check |
| Publish | ✓ | | | Final human approve |

---

### 6. Description in practice & prompting techniques

Translate Product / Process / Performance into a prompt skeleton (original wording):

```text
PRODUCT: Deliverable, audience, length, must-include, must-avoid, example of good.
PROCESS: Steps to follow, tools to use, when to ask me questions, what to cite.
PERFORMANCE: Tone, uncertainty handling, challenge my assumptions if weak.
```

Then iterate. Description is not a one-shot ritual; it is a **conversation craft**.

**Effective habits**

- Show, don’t only tell (examples beat adjectives)
- Separate “generate options” from “pick a winner”
- Ask for assumptions listed explicitly
- Prefer precise feedback over restart-everything rage (unless the thread is lost)
- Put evaluation criteria in the prompt when possible (“score yourself against this rubric, then improve”)

**Anti-patterns**

| Anti-pattern | Failure | Fix |
|--------------|---------|-----|
| Adjective pile (“brilliant, epic, viral”) | Vague product | Concrete audience + example |
| No stop points | Overconfident completion | “Ask before assuming X” |
| Hidden stakes | Wrong risk posture | State who gets hurt if wrong |
| One giant mega-prompt forever | Brittle | Iterate via Discernment loop |

**If the exam asks X, think Y**

- X: “Only product described” → Y: Add process + performance
- X: “Model never asks clarifying questions” → Y: Performance/process rules that require questions when ambiguous

---

### 7. The Description–Discernment loop

Core cycle:

1. Describe what you want (and how)
2. Receive output / observe behavior
3. Discern gaps against criteria
4. Update Description (or Delegation decision)
5. Repeat until the bar is met—or decide AI is the wrong tool

This loop is the antidote to both blind acceptance and aimless re-prompting.

**Exam phrase to remember:** Better Description is often the *result* of Discernment, not only its input.

**Worked loop (original) — landing-page hero**

1. Describe: “Hero copy for privacy-first notes app; 12 words max.”
2. Output: clever but implies “military-grade encryption” you don’t offer
3. Discern: marketing claim risk
4. Re-describe: “No security claims beyond ‘encrypted on device’; friendly tone; 12 words.”
5. Output passes claim check → proceed; still human-approves before ship (Deployment Diligence)

**Loop failure modes**

- Skipping criteria → endless taste debates
- Changing the goal every turn → no learning
- Never changing Delegation when the tool is wrong → wasted loops

---

### 8. Diligence in the real world

**Creation:** Don’t ship first draft vibes on regulated or reputation-critical work. Choose appropriate involvement mode; maintain a quality bar.

**Transparency:** Diligence statements, footnotes, or client notes that AI helped—proportionate to context (school policies and enterprise rules differ). Collection tutorial “Writing an AI diligence statement” is practical homework.

**Deployment:** Consider who is affected, failure modes, and monitoring after release (especially for agentic workflows).

**Sample diligence statement pattern (original, short)**

```text
AI assistance: Claude helped outline and edit sections 2–3.
Human responsibility: I verified all facts against sources dated ____,
removed unsupported claims, and approve this version for [audience].
Data note: No customer PII was pasted into the chat.
```

**If the exam asks X, think Y**

- X: “Course discloses AI help in making materials” → Y: Models Transparency Diligence
- X: “Agent sends emails unsupervised” → Y: Deployment Diligence (+ Delegation/Agency risk)

---

### 9. How the 4Ds show up across modes

- **Automation:** Strong Description of the task; Discernment on outputs; Diligence on data you feed in
- **Augmentation:** All four, with heavy Description–Discernment looping
- **Agency:** Heavy upfront Delegation + standing Description; continuous Discernment of autonomous steps; Diligence before and after deployment

Claude product features (Projects, Skills, Connectors—see Claude 101 notes) are **infrastructure for Description and Delegation**, not a substitute for judgment.

| Claude 101 feature | Fluency job |
|--------------------|-------------|
| Projects | Standing Description (knowledge + rules) |
| Skills | Process Description reusable |
| Artifacts | Make Product visible for Discernment |
| Connectors | Expand platform options for Delegation |
| Research / Enterprise Search | Grounding aids Discernment—still verify |
| Chat vs Cowork vs Code | Mode / Delegation choice |

---

### 10. Behavioral indicators (study translation)

Public collection includes “4 Ds — Behavioral Indicators.” For exams, remember *examples of behaviors*, not a memorized official list:

| D | Looks like in daily work |
|---|--------------------------|
| Delegation | Pausing to ask “should AI do this?”; writing keep/share/give |
| Description | Adding examples, constraints, success criteria; clarifying ambiguity |
| Discernment | Spot-checking facts; rejecting sycophancy; comparing to gold outputs |
| Diligence | Disclosing assistance; protecting data; refusing to ship unverified claims |

**Anti-behaviors**

- Delegation: “Let AI decide the legal call”
- Description: “You know what I mean”
- Discernment: “Sounds smart, ship it”
- Diligence: “Nobody needs to know AI wrote this” when policy requires disclosure

---

## Decision trees / comparison tables

### Scenario → primary D

| Scenario | Start with |
|----------|------------|
| Should we use AI at all on this? | Delegation |
| Output misses the brief | Description (then loop) |
| Output is fluent but wrong | Discernment |
| About to publish / client-send | Diligence |
| Model only agrees with you | Discernment + Performance Description |
| Choosing Automation vs Agency | Delegation (+ platform awareness) |

### Automation vs Augmentation vs Agency

| Question | Automation | Augmentation | Agency |
|----------|------------|--------------|--------|
| Who steers moment-to-moment? | Human spec | Both | Standing config + monitoring |
| Typical risk | Silent wrong execution | Collaborative drift | Unwatched side effects |
| Description weight | Task clarity | Ongoing dialogue | Goals + guardrails |
| Discernment timing | After output | Continuously | During and after autonomous steps |

### Product vs Process vs Performance

| Layer | Asks | Example ingredient |
|-------|------|--------------------|
| Product | What is the thing? | “1-page FAQ; audience = new users” |
| Process | How to make it? | “Use only attached policy; list assumptions first” |
| Performance | How to behave? | “If unsure, ask; never invent policy” |

### Problem vs Platform vs Task (Delegation lenses)

| Lens | Failure if skipped | Fix |
|------|--------------------|-----|
| Problem | Solve the wrong pain | Restate success + harm |
| Platform | Ask impossible recall / ignore tools | Match task to capabilities |
| Task | AI decides what humans must own | Keep/share/give split |

### Trust failure → response

| Failure | Response mix |
|---------|--------------|
| Hallucination | Grounding tools + verify sources + Discernment |
| Sycophancy | Ask for critique / steelman opposition |
| Bias | Diversify examples; inspect recommendations for stereotypes |
| Privacy leak risk | Minimize data; follow policy; prefer approved environments |

---

## Common exam traps

1. **Fluency = prompt engineering** — Too narrow. Prompting supports Description; fluency includes all 4Ds + modes.
2. **Agency means no oversight** — Opposite. More autonomy needs more Description, Discernment, Diligence.
3. **Automation is always faster and better** — Only when the task is well-specified and checked.
4. **Discernment is only fact-checking outputs** — Also process and behavior (sycophancy, bias, refusals).
5. **Diligence is only “be nice / ethical vibes”** — Includes creation care, disclosure, and deployment checks.
6. **Product Description alone is enough** — Missing process/performance causes overreach.
7. **Platform awareness is optional if you’re a power user** — Experts still mismatch tools to tasks.
8. **If AI helped, you can deny responsibility** — False. Humans own outcomes.
9. **Description–Discernment loop is optional polish** — It’s the core improvement engine.
10. **Hallucination means the model is “broken” only** — It’s an expected failure mode of generative systems; mitigate, don’t mythologize.
11. **Sycophancy is the same as hallucination** — Related trust issues; sycophancy is over-agreement/people-pleasing.
12. **Collection electives don’t matter** — Exams may ask where to deepen (capabilities course, diligence statement, tokens).
13. **4Ds are a strict waterfall once** — They interact continuously.
14. **Effective automatically means ethical** — Four outcomes; you can be effective at the wrong thing.
15. **Dakan–Feller framework is “Anthropic-only IP trivia”** — Know high-level origin; focus on applying competencies.

---

## Expanded self-check (Q&A)

**Q1.** Define AI Fluency in one sentence using the four outcome words.

**A1.** AI Fluency is the ability to engage with AI systems in ways that are effective, efficient, ethical, and safe.

**Q2.** List the 4Ds and give one verb for each.

**A2.** Delegation (decide/split), Description (communicate/specify), Discernment (evaluate/critique), Diligence (own/disclose/protect).

**Q3.** How does Augmentation differ from Automation?

**A3.** Automation has AI execute a specified task; Augmentation is ongoing partnership on thinking and doing. Automation is “do this”; Augmentation is “work with me on this.”

**Q4.** You ask AI to “make my argument stronger,” and it only agrees louder. Which failure mode is likely, and what Description fix helps?

**A4.** Sycophancy. Ask it to steelman the opposing view, list weaknesses, and only then suggest improvements—Performance Description that rewards critique.

**Q5.** Name the three Description types and one prompt ingredient for each.

**A5.** Product (deliverable + example); Process (steps + when to ask questions); Performance (tone + uncertainty rules).

**Q6.** What is the Description–Discernment loop?

**A6.** Iterative cycle: describe needs → review results critically → refine description (or change delegation) → repeat.

**Q7.** Give an example of Transparency Diligence.

**A7.** Adding a short note that AI assisted drafting of a report section, per school or company policy.

**Q8.** Why does platform awareness matter for Delegation?

**A8.** You cannot wisely assign work without knowing tool limits (no live web, context size, hallucination risk, missing connectors, etc.).

**Q9.** Agency increases autonomy. Does that reduce the need for Discernment?

**A9.** No—it usually increases the need for clearer guardrails and stronger evaluation of independent actions.

**Q10.** Name two related collection resources beyond this flagship course.

**A10.** Any two: AI Capabilities and Limitations; 4 Properties of AI; hallucination/sycophancy/bias tutorials; tokens/context tutorials; diligence statement tutorial; role-specific Fluency editions.

**Q11.** Map “I automated grading comments without sampling accuracy first” to the broken Ds.

**A11.** Weak Discernment before Delegation into Automation; Diligence risk if deployed to students.

**Q12.** What are the three Delegation lenses?

**A12.** Problem awareness, platform awareness, and task delegation (split).

**Q13.** Creation vs Deployment Diligence—difference?

**A13.** Creation = care while producing with AI; Deployment = care before/as you release into real-world impact (users, public, systems).

**Q14.** Give a keep/share/give split for writing a public incident report.

**A14.** Keep: final severity language and legal claims. Share: structure and clarity edits. Give: formatting timeline tables from approved facts. Checkpoint: every factual sentence traced to an approved source.

**Q15.** Why are the 4Ds described as interconnected?

**A15.** Failures cascade: bad Description undermines Discernment; bad Discernment leads to reckless Delegation; Diligence fails if the other three are sloppy.

**Q16.** What is a usable LLM mental model for this course?

**A16.** Next-token / pattern prediction plus current context/tools; strong at draft/transform/explain; weak at guaranteed truth without grounding.

**Q17.** When should you exit the Description–Discernment loop and change tools or stop?

**A17.** When criteria cannot be met reliably (wrong platform, missing data, stakes too high for generative drafting)—change Delegation.

**Q18.** How do Projects/Skills from Claude 101 support Fluency?

**A18.** They encode standing Description (knowledge/process), improving Delegation quality—not replacing Discernment or Diligence.

**Q19.** Name the three Diligence strands.

**A19.** Creation, Transparency, Deployment.

**Q20.** “Sounds confident and cites DOIs.” Enough to trust?

**A20.** No. Discernment requires verification; citations can be invented or mismatched.

**Q21.** Efficient but unethical—fluency?

**A21.** Incomplete. Fluency requires effective, efficient, ethical, *and* safe.

**Q22.** Who developed the AI Fluency Framework (high level)?

**A22.** Professors Rick Dakan and Joseph Feller (publicly credited), later partnered with Anthropic for the Academy course.

**Q23.** Badge path on the public course page?

**A23.** Finish lessons → final assessment → completion badge opportunity.

**Q24.** Bias mitigation habit (one)?

**A24.** Inspect outputs for stereotyped assumptions; ask for alternative framings; diversify examples in Description.

**Q25.** Privacy diligence micro-habit?

**A25.** Minimize sensitive data pasted into chats; follow org/school rules; prefer approved enterprise controls when required.

---

## Quick review checklist

- [ ] Fluency = effective, efficient, ethical, safe collaboration
- [ ] 4Ds recited with one example behavior each
- [ ] Automation / Augmentation / Agency differences cold
- [ ] Product / Process / Performance Description ingredients
- [ ] Problem / Platform / Task Delegation lenses
- [ ] Description–Discernment loop steps
- [ ] Creation / Transparency / Deployment Diligence
- [ ] Hallucination, sycophancy, bias, privacy—define + mitigate
- [ ] More agency → more oversight, not less
- [ ] Features (Projects/Skills/etc.) support Description/Delegation; judgment remains human
- [ ] Know collection “where next” at title level
- [ ] Human accountability never disappears

---

## Glossary

| Term | Short definition |
|------|------------------|
| Agency | Configuring AI to pursue goals more independently under your guardrails |
| AI Fluency | Effective, efficient, ethical, safe human–AI collaboration skill |
| Augmentation | Human–AI partnership on thinking and doing |
| Automation | AI executes a specified task on instruction |
| Bias | Skewed patterns/associations that distort outputs |
| Creation Diligence | Care and quality bar while building with AI |
| Delegation | Deciding whether/when/how to use AI and how to split work |
| Deployment Diligence | Responsibility checks before/as work enters the world |
| Description | Communicating product, process, and performance clearly |
| Description–Discernment loop | Iterate specify → critique → respecify |
| Discernment | Critical evaluation of outputs, processes, behaviors |
| Diligence | Taking responsibility for AI-assisted work |
| Hallucination | Fluent falsehood |
| Platform awareness | Knowing a given tool’s capabilities and limits |
| Problem awareness | Understanding goals, stakes, constraints, success |
| Performance Description | Desired interaction behavior |
| Process Description | Desired method/steps/tools |
| Product Description | Desired deliverable |
| Sycophancy | Over-agreeable, people-pleasing behavior |
| Task delegation | Keep / share / give workflow splitting |
| Transparency Diligence | Appropriate disclosure of AI assistance |
| Tokens | Units of text processing/cost/context |

---

## Study plan

1. Memorize Key concepts map (10–15 minutes)
2. Read Deep notes on 4Ds + modes twice
3. Skim fundamentals + trust topics
4. Rehearse decision trees from memory
5. Drill self-check without peeking
6. Optional: skim collection titles so pathways feel familiar
7. Pair with `01-claude-101.md` to connect framework → product features
8. Night before: checklist + glossary only

*End of AI Fluency study notes.*
