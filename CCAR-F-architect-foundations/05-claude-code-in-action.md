---
title: Claude Code in Action
source: https://academy.claude.com/courses/claude-code-in-action
disclaimer: Original study notes for exam prep — not official Anthropic material. Not a lesson transcript.
approx_length: ~5500–6500 words
deepened: 2026-08-23
---

# Claude Code in Action — Primary Study Notes

> **Disclaimer:** These are **original** exam-prep notes for Claude certification study. They align to the **public** course outline at [Claude Code in Action](https://academy.claude.com/courses/claude-code-in-action) (objectives and module themes only). They are **not** Claude Academy lesson dumps, transcripts, or verbatim slide text. Completing the official course remains the source of truth for quizzes. Verify CLI flag and UI labels in current Claude Code docs — names evolve.

**Who it is for:** Developers who already use Claude Code for single prompts and want longer, less supervised, team-wide workflows.

**Prerequisites (course):** Claude Code for one-shot tasks; basic Git and command line.

**How to use these notes:** Read one major section, then answer the self-check without scrolling back. Treat tables as flashcards. The ladder **steer → configure → automate → verify/share** is the spine of most scenario questions.

---

## 1. Course map (public objectives)

The public page frames four outcome clusters (~9 lessons, 1 quiz):

1. **Steer the work** — plan mode, directed compaction, rewind, hands-on vs autonomous goal/loop runs, safe parallelism.
2. **Configure Claude** — lean `CLAUDE.md`, skills, permission modes, hooks for rules that cannot be skipped.
3. **Automate repeat work** — routines on Anthropic infrastructure, headless mode in your pipeline, PR / GitHub Action wiring.
4. **Verify and share** — verify unsupervised runs in proportion to how little you watched them; gate on real tests; package a trusted setup as a plugin.

**Mental model:** Claude Code is an agentic coding loop with tools (file edit, bash, search, etc.). Your job on exams is not “write a longer prompt,” but **choose the right control surface** for the risk of the job.

| Layer | Question it answers | Strength |
| --- | --- | --- |
| Steering | What should we do *this session*? | Ephemeral control |
| Configuration | What should Claude *always* know / be allowed? | Persistent |
| Automation | How does this run without me typing? | Operational |
| Verification / plugins | How do we trust and reuse the setup? | Assurance + portability |

Remember: many exam-style questions ask which layer **advises** vs **enforces**.

---

## 2. Steering long sessions

### 2.1 Plan mode

For non-trivial changes, start in **plan mode**. Claude may read the repo and run exploratory commands, but it should **not edit source** until you approve a plan. Use it when:

- The codebase is unfamiliar
- The change touches many files or modules
- You need architectural decisions before code
- Blast radius includes migrations, auth, billing, or deploy scripts

Cycle modes in the CLI (commonly **Shift+Tab**). Learn the *intent* of each mode more than any single UI string — labels can change; the spectrum from “read-only exploration” to “auto-approve edits” to “bypass prompts” does not.

**Exam framing:** Plan mode is a *steering* choice, not a permanent project setting. After you approve a plan, you typically switch to a mode that allows implementation under permissions you trust.

### 2.2 Compaction

Long sessions fill the context window. Claude Code **compacts** older turns (and you can trigger compaction manually in many setups, e.g. `/compact`). Summaries drop detail.

**What survives compaction poorly:** one-off chat rules (“always use pytest”), temporary naming decisions, failed experiment paths that you still need as negative examples.

**What should survive by design:**

- Durable conventions → **`CLAUDE.md`** / `.claude/rules/`
- On-demand procedures → **skills**
- Hard stops → **hooks** / deny rules

When you *direct* compaction (keep X, drop Y), prefer keeping: approved plan, open decisions, constraints, failing test names. Drop: verbose tool dumps, abandoned rabbit holes, duplicate file listings.

**Exam trap:** “Compaction lost our coding standards” → root cause is standards living only in chat, not in CLAUDE.md.

### 2.3 Rewind

When a session drifts, **rewind** to an earlier clean point instead of stacking contradictory fixes on a polluted context. Then restate **one** clear goal.

Rewind vs compact:

| Action | Best when | Risk if misused |
| --- | --- | --- |
| Rewind | Wrong path / contradictory instructions | Lose useful later work if you rewind too far |
| Compact | Context full but direction still good | Lose detail; may reinforce bad summaries if chat was messy |

**Pattern:** Drift → rewind → single goal → (optional) plan mode again → implement.

### 2.4 Hands-on vs autonomous

| Style | Use when | Control |
| --- | --- | --- |
| Hands-on steering | Novel / high blast radius | You approve edits and watch turns |
| Goal + loop / autonomous | Trusted, well-scoped tasks | Hooks, tests, tighter permission mode |
| Parallel agents | Independent subtasks | **Git worktrees** (or separate checkouts) |

**Exam nugget:** Parallel agents sharing one dirty working tree is the wrong pattern; isolation via worktrees is the safe answer.

### 2.5 Steering vs autonomy decision tree

```text
Is the task novel or high blast radius?
 YES → Hands-on + plan mode; default/manual or accept-edits only after reviewed plan
 NO → Is the procedure repeated and already verified?
 YES → Skill + hooks; consider routine
 NO → Still interactive; encode learnings into CLAUDE.md as you go

Does it need your laptop secrets, private network, or custom piping?
 YES → Headless in YOUR CI/agent host
 NO → Routine on Anthropic infra may suffice

Must results be identical run-to-run in CI?
 YES → Bare/deterministic headless flags + pinned permissions
 NO → Interactive or auto modes OK with verification

Multiple agents useful?
 YES → Split independent work; isolate with worktrees
 NO → Single agent, clear goal

Session went sideways?
 → Rewind first; only then re-prompt
```

## 3. Configuration surfaces (advisory vs enforced)

| Surface | Nature | Strength |
| --- | --- | --- |
| Chat prompt | Temporary | Advisory |
| `CLAUDE.md` / rules | Always-on project context | Advisory but persistent |
| Skills | On-demand procedures | Advisory when loaded |
| Permission rules / modes | Tool allow / ask / deny | **Enforced by Claude Code** |
| Hooks | Lifecycle scripts | **Enforced guarantees** |

**Core distinction:** "Never edit `.env`" in `CLAUDE.md` is a polite request. A **deny** rule or **PreToolUse hook** that blocks `.env` is a guarantee. Instructions shape intent; rules and hooks change what is allowed.

### 3.1 Lean CLAUDE.md

Keep it short enough that Claude follows it. Public guidance and good exam answers converge on:

- Project purpose, stack, layout (few bullets)
- Commands that matter (test, lint, build)
- Hard do/don't for secrets, migrations, force-push
- Pointers to skills/docs for long procedures — do **not** paste a novel

Put each rule on the right surface:

| Content type | Surface |
| --- | --- |
| Always-true facts and conventions | CLAUDE.md |
| Path-scoped conventions | `.claude/rules/` (when available) |
| Multi-step / occasional workflows | Skills |
| Must-run-at-event guarantees | Hooks |
| Tool allow/deny | Permission rules |

`/init` (or equivalent) maps the repo and seeds CLAUDE.md; refine with what Claude cannot discover from code alone.

**Anti-pattern:** A 2,000-line CLAUDE.md that duplicates the README, the wiki, and every coding standard PDF. Claude will under-follow it; exams reward *lean* memory.

**CLAUDE.md maintenance loop:** After a painful session, ask: "What should never be rediscovered from scratch?" Promote that fact into CLAUDE.md. If it is a procedure, promote to a skill instead.

### 3.2 Skills

**Skills** package repeated procedures (start with a verification skill that runs real checks). Typical skill contents conceptually:

- When to apply
- Steps / commands
- Success criteria
- Optional allowed-tools grants for the invoking turn (still subject to baseline permissions)

Skills load **on demand** (invoked or auto-selected when relevant). That is why they beat stuffing every procedure into CLAUDE.md.

**Exam contrast:**

- CLAUDE.md = always in context → keep short
- Skill = load when needed → detailed OK
- Hook = runs regardless of Claude's "opinion" → use for non-negotiables

Good first skills for a team: "run verification suite," "create PR with checklist," "migrate DB safely," "security review touchpoints."

### 3.3 Plugins

**Plugins** bundle CLAUDE.md pieces + skills + hooks + related config (and sometimes MCP / subagents) so the whole team installs one trusted setup. Plugins are about **portability**, not new model capabilities.

Share when: the setup is verified, hooks gate real tests, permissions are least-privilege, and onboarding cost drops from "read a wiki" to "install plugin."

Plugin hygiene checklist:

1. Document required env vars and secrets injection points
2. Fail closed if tests cannot run
3. Namespace skills so multiple plugins coexist
4. Version the plugin; break changes need a changelog
5. Never ship bypass-by-default in a team plugin
---

## 4. Permission modes (match mode to risk)

Modes set the baseline before allow/ask/deny rules. Names evolve; know the spectrum:

| Mode (conceptual) | Behavior | Typical job |
| --- | --- | --- |
| Default / manual | Prompt on first tool use | Interactive pairing |
| Accept edits | Auto-accept common workspace edits | Trusted local feature work |
| Plan | Explore; no source edits until plan approved | Design before implement |
| Auto (classifier) | Approves routine actions; blocks risky ones | Faster interactive with guardrails |
| Don't ask | Strict; often denies unless pre-allowed | Controlled non-interactive |
| Bypass / don't ask risky variant | Skips most prompts | Isolated CI / sandbox **only** |

**Evaluation order (mental model):** deny → ask → allow; first match wins. Deny beats allow. A PreToolUse hook that exits with a blocking decision can still stop a call even when an allow rule or bypass mode would otherwise pass — hooks are the last hard gate in many designs.

**Permission decision tree:**

```text
Interactive pairing / learning codebase?
 → Default/manual or plan
Trusted local feature after reviewed plan?
 → Accept edits (or auto with classifier)
CI / headless with pinned allowlist?
 → Don't ask + explicit allows + hooks
Throwaway container, no secrets, ephemeral FS?
 → Bypass may be acceptable
Production credentials on laptop?
 → Never bypass; least privilege + deny rules
```

**Exam traps:**

- Bypass-style modes as a laptop default with production credentials → wrong
- Assuming CLAUDE.md overrides permission deny → wrong
- Using plan mode for a trusted, already-specified one-line rename → overly slow but not "unsafe"; usually accept-edits or auto is fine
- Believing auto mode replaces deny rules for `.env` → wrong; classifier is probabilistic, deny is absolute

Admins may disable bypass in team settings — know that organizational policy can remove the escape hatch.

---

## 5. Hooks: unskippable control

Hooks fire at fixed lifecycle points. Examples worth memorizing (exact event names may evolve — learn the *jobs*):

| Lifecycle job | Example use |
| --- | --- |
| Session start | Inject env facts, repo branch warnings |
| Pre tool use | Block `.env`, force ask on `rm -rf`, deny production deploys |
| Permission request | Auto-decide in non-interactive runs |
| Pre compact | Re-inject critical constraints into summary |
| Stop / after turn | Run linters; fail closed if tests red |
| File / rules loaded | Audit what entered context |

Patterns:

1. Block protected paths (`.env`, production configs, private keys)
2. Gate a turn on **real** test runners (fail closed if red)
3. Force a prompt for sensitive Bash even if Bash is broadly allowed
4. Re-inject critical context after compaction
5. Return permission-decision JSON / exit codes to allow, deny, or ask

**Hooks enforce; prompts advise.**

**Non-interactive note:** In headless `-p` runs, interactive permission prompts may not exist. Rely on PreToolUse hooks and pre-declared allow/deny rules — do not assume a human will click "allow."

**Hook design tips for exams:**

- Prefer fail-closed for safety checks (missing test runner = deny)
- Keep hooks fast; slow hooks frustrate interactive use and get disabled
- Log denials with enough detail for audit without dumping secrets
- Do not put irreversible destructive capabilities behind soft asks alone

---

## 6. Automation spectrum

Once a task is trusted, stop kicking it off by hand:

1. **Routines** — schedule prompts on Anthropic-managed infrastructure (least ops).
2. **Headless** — `-p` / `--print`: one-shot, no TUI; stdin/stdout for pipes and CI.
3. **Bare / deterministic flags** — prefer when CI needs repeatable, low-variance runs.
4. **Agent SDK** — embed Claude Code inside your TypeScript/Python product.
5. **PR / GitHub Action wiring** — managed review patterns; Claude in the pull-request loop.

**Decision rule:** start with routines; drop to headless when the job needs *your* environment or surrounding script logic; use the Agent SDK when the agent *is* part of your product.

```text
Need Anthropic-hosted schedule, little custom env?
 → Routine
Need local secrets, Docker, private packages, custom scripts?
 → Headless in your runner
Need agent embedded in product UX?
 → Agent SDK
Need PR comments / checks?
 → GitHub Action / managed review integration
```

Automation without proportional verification is incomplete — that sentence is an exam answer by itself.

**Headless specifics worth remembering:**

- Non-interactive means no human in the permission loop
- Pipe prompts via stdin or args; capture stdout for logs
- Pair with explicit permission allowlists and PreToolUse hooks
- Prefer deterministic settings when CI flakes would be costly

---

## 7. Verify unsupervised work

**The less you watched, the more you verify.**

| How unsupervised | Minimum verification |
| --- | --- |
| Interactive pair | Spot-check diffs |
| Accept-edits local run | `git diff` + smoke tests |
| Auto mode long session | Diff review + targeted tests + hook gates |
| Headless / scheduled / bypass | Mandatory automated tests, deny rules for dangerous tools, human review of high-risk paths |

Never trust Claude's claim that "tests passed" — execute the runner in a hook or CI step.

**Verification skill pattern:** Package "run unit + lint + typecheck; print summary; non-zero exit on failure" as a skill, and also wire the same commands into a Stop/PreToolUse gate for unattended runs.

**Risk-tiered review:**

- Low: formatting, comments, docs → automated checks may suffice
- Medium: app logic → tests + diff skim
- High: auth, payments, migrations, IAM → mandatory human review regardless of green CI
---

## 8. Practical end-to-end workflow (cert checklist)

1. Initialize / map the repo (`/init` or equivalent).
2. Feed relevant files / point at the subsystem before asking for a feature.
3. Plan (no code) then approve then implement.
4. Prefer test-aware loops when quality matters.
5. Encode durable rules in CLAUDE.md; procedures in skills; hard stops in hooks.
6. Choose permission mode for the job risk.
7. Automate only after verification is solid; share via plugin.
8. If drift: rewind, then one goal, then continue.
9. Before shipping plugin: prove hooks fail closed on a deliberate red test.

**Session hygiene habits:**

- One goal per stretch of autonomous work
- Commit or stash before large autonomous runs so rewind or git restore is easy
- Name branches clearly when using worktrees
- After compaction, restate the current plan in one sentence

---

## 9. Anti-patterns (high-yield exam recognition)

| Anti-pattern | Why it fails | Prefer |
| --- | --- | --- |
| Safety only in chat or CLAUDE.md | Advisory; ignored under pressure | Deny plus PreToolUse hook |
| Bypass on laptop with prod creds | Huge blast radius | Sandbox CI plus least privilege |
| Automate before tests exist | Unsupervised garbage at scale | Write gates first |
| Two agents, one dirty branch | Merge chaos or lost work | Worktrees |
| Trust model claim that tests passed | Hallucinated verification | Hook executes runner |
| Novel in CLAUDE.md | Dilution; under-followed | Skills plus short CLAUDE.md |
| Compact instead of rewind after contradiction | Bad summary locks in confusion | Rewind |
| Plan mode forever | Never ships | Approve plan then implement mode |
| Plugin with bypass default | Spreads unsafe defaults | Least privilege defaults |
| Ignoring Stop-hook failures | Greenwashed broken code | Fail closed |
---

## 10. Comparison: control surfaces deep dive

Think of Claude Code configuration as layered defense:

```text
[Chat instructions] advisory, ephemeral
 |
[CLAUDE.md / rules] advisory, durable
 |
[Skills] advisory, on-demand depth
 |
[Permission modes/rules] enforced allow/ask/deny
 |
[Hooks] enforced lifecycle guarantees
 |
[CI / human review] external verification
```

Exams love scenarios where the student picks a higher layer than needed (for example, adding a novel to CLAUDE.md when a deny rule was required) or a lower layer than needed (hoping chat alone stops force-push).

**Worked scenario A — secret file protection**

- Wrong: ask in chat not to touch secret files
- Better: CLAUDE.md note, still not enough alone
- Right: deny rule for edit/write on secret env files and a PreToolUse hook blocking those paths

**Worked scenario B — weekly dependency bump**

- Wrong: manually re-prompt every Monday with bypass on laptop
- Right: verified skill plus routine or headless CI job plus tests plus PR for human merge

**Worked scenario C — unfamiliar payment refactor**

- Wrong: auto mode overnight
- Right: plan mode, design review, accept-edits with tight scope, tests, human review of payment paths

---

## 11. Exam tips

- Map scenarios to steer / configure / automate / verify.
- Identify advisory vs enforced controls every time.
- Prefer plan mode for unfamiliar or multi-file work.
- Prefer worktrees for parallel agents.
- Prefer routines until you truly need headless or SDK.
- Compaction without CLAUDE.md loses conventions — that failure mode is testable.
- Bypass without isolation is the wrong safety answer.
- Hooks can block even when modes are permissive — know that ordering story.
- Plugins package trust; they do not replace verification.
- Headless without hooks is an incomplete answer for unattended safety.
---

## 12. Self-check Q&A (with answers)

**Q1.** CLAUDE.md says never commit secrets, but a secret env file still gets staged. What stops it? 
**A.** Deny permission rules and/or a PreToolUse hook — not the markdown sentence alone.

**Q2.** When is plan mode the right default? 
**A.** Before large or unclear edits: explore and propose without changing source until approval.

**Q3.** Session is confused after many failed attempts. Next step? 
**A.** Rewind (or carefully compact), then one clear goal — avoid contradictory stacked instructions.

**Q4.** Same prompt every Monday. First automation choice? 
**A.** A routine; drop to headless if you need local env or custom piping.

**Q5.** Why CLAUDE.md instead of only chat? 
**A.** Chat is compacted away; CLAUDE.md is the durable project instruction surface.

**Q6.** Two agents on one branch — safer pattern? 
**A.** Isolate with git worktrees / separate checkouts.

**Q7.** What does headless `-p` provide? 
**A.** Non-interactive, scriptable runs for pipelines and pipes.

**Q8.** How much verification for fully unattended runs? 
**A.** Maximum: test gates, deny rules for dangerous tools, and review policy proportional to zero human watching.

**Q9.** Skill vs CLAUDE.md for a 20-step release checklist? 
**A.** Skill (on-demand procedure); keep CLAUDE.md lean with a pointer.

**Q10.** Accept-edits vs bypass for trusted local feature work on a developer machine? 
**A.** Accept-edits (or auto with classifier). Bypass belongs in isolated sandboxes/CI, not default laptop use.

**Q11.** What should a PreCompact hook often do? 
**A.** Re-inject critical constraints so compaction summaries do not drop non-negotiables.

**Q12.** Plugin vs skill? 
**A.** Skill = one packaged procedure/knowledge unit. Plugin = installable bundle of skills, hooks, config (portability layer).
**Q13.** Auto mode still does something dangerous. Your next hardening step? 
**A.** Add an explicit deny rule and/or PreToolUse hook for that action class — do not rely on the classifier alone for non-negotiables.

**Q14.** When choose Agent SDK over headless CLI? 
**A.** When the coding agent is embedded inside your product TypeScript/Python application, not merely invoked from a shell pipeline.

**Q15.** GitHub Action runs Claude on every PR. What must exist first? 
**A.** Proportional verification: real tests, scoped permissions, and review of high-risk paths — automation without gates is incomplete.

**Q16.** Evaluation order if allow and deny both match a tool path? 
**A.** Deny wins (first-match mental model: deny before allow).

**Q17.** Why might hooks still matter in bypass mode? 
**A.** PreToolUse hooks can deny regardless of permission mode — they enforce policy users cannot skip by flipping mode.

**Q18.** Parallel agents editing the same file — correct? 
**A.** No. Split independent work; isolate trees; merge deliberately.

**Q19.** After `/init`, CLAUDE.md is huge and generic. What do you do? 
**A.** Trim to always-true facts; move procedures to skills; add only non-discoverable conventions.

**Q20.** Unattended job needs private package registry credentials. Routine or headless? 
**A.** Headless (or your CI) where secrets and network live — not a cloud routine alone unless that infra has the secrets.

**Q21.** What is the ladder order for building team Claude Code maturity? 
**A.** Steer, then configure, then automate, then verify/share.

**Q22.** Spot-check diffs enough for scheduled overnight refactors? 
**A.** No — require automated tests plus hooks plus human review of high-risk diffs.

**Q23.** Where do irreversible must-never rules belong? 
**A.** Permission deny rules and hooks — not only CLAUDE.md.

**Q24.** Directed compaction should prioritize keeping what? 
**A.** Approved plan, open decisions, constraints, and failing tests — drop noisy failed experiment dumps.

**Q25.** Why is a verification skill still valuable if CI already runs tests? 
**A.** Local interactive loops catch failures earlier; the same commands should also gate unattended Stop/PreToolUse hooks for consistency.
---

## 13. Review checklist (before exam)

- [ ] Can explain plan / compact / rewind and when to use each
- [ ] Can sort CLAUDE.md vs skills vs hooks vs permissions by advisory/enforced
- [ ] Can pick permission mode for interactive, trusted local, CI, sandbox
- [ ] Can choose routine vs headless vs Agent SDK
- [ ] Can describe worktree isolation for parallel agents
- [ ] Can state verification proportional to supervision
- [ ] Can list five anti-patterns and the fix for each
- [ ] Can describe what a plugin packages and why teams use it
- [ ] Know hooks beat permission mode for hard policy
- [ ] Know compaction loses chat-only conventions
- [ ] Can walk the steer → configure → automate → verify ladder
- [ ] Can design a fail-closed test gate for unattended runs

---

## 14. Glossary

- **Plan mode** — research/propose without editing source until approval.
- **Compaction** — summarize older context to free window space.
- **Directed compaction** — steer what the summary must keep/drop.
- **Rewind** — restore earlier session state after drift.
- **CLAUDE.md** — persistent project memory loaded each session.
- **Rules** — modular / path-scoped instruction files (when used).
- **Skill** — on-demand packaged procedure or domain knowledge.
- **Hook** — enforced lifecycle script (allow/deny/ask/side effects).
- **Permission mode** — baseline for tool approval behavior.
- **Allow / ask / deny rules** — explicit tool permission policy.
- **Routine** — scheduled managed run on Anthropic infrastructure.
- **Headless** — print/one-shot CLI (`-p`) for pipelines.
- **Agent SDK** — embed Claude Code loops in your application.
- **Plugin** — shareable Claude Code setup bundle.
- **Worktree** — separate Git working tree for isolated parallel agents.
- **Verification gate** — real test/lint command that fails closed.
- **Advisory vs enforced** — prompt/CLAUDE.md advise; permissions/hooks enforce.
- **Goal and loop** — autonomous style: clear goal, iterate until checks pass.
- **Classifier auto mode** — model-assisted approval of routine actions with risk blocks.
- **Fail closed** — if a safety check cannot run, treat as failure/deny.

---

## 15. Extra depth: subagents, MCP, and team rollout

### Subagents (conceptual)

Claude Code can spawn specialized subagents for isolated work (explore, plan, or custom). Isolation prevents bloating the main session. For exams: use subagents when the subtask does not need the full conversation; keep dangerous tools constrained; remember background subagents in non-interactive mode cannot show interactive prompts — hooks and denies decide.

### MCP inside Claude Code

MCP servers extend tools/resources/prompts available to the coding agent. Treat MCP tools like any other tools: permission rules and hooks still apply. Do not assume an MCP server is safe because it is "just config" — it can expose powerful side effects.

### Team rollout sequence (practical)

1. One repo pilot: lean CLAUDE.md + verification skill + deny secrets
2. Add PreToolUse hooks for protected paths and test gates
3. Standardize permission defaults for interactive vs CI
4. Package as plugin; install on second repo
5. Add routine/headless for one boring weekly task
6. Measure: fewer permission prompts that are noise, fewer incidents, faster PR checks
7. Only then expand autonomy (auto mode, more routines)

### Metrics that matter (not vanity)

- Time from issue → green PR
- Rate of hook denials that prevented real mistakes
- Flake rate of verification gates
- Onboarding time for a new engineer with the plugin
- Incidents involving unsupervised edits

### Study rhythm

Day 1: sections 1–5 (steer + configure). Flashcard advisory vs enforced. 
Day 2: sections 6–10 (automate + verify + anti-patterns). Decision trees aloud. 
Day 3: all Q&A closed-book, then checklist. Re-read only missed items.

---

## 16. Reading tip

Each major heading is a flashcard front; the tables are the backs. Quiz yourself on advisory vs enforced, then on routines vs headless, then on plan/compact/rewind, before re-opening the Academy outline.

---

*Aligned to public outline at https://academy.claude.com/courses/claude-code-in-action. Use for recall; complete the official course for quizzes. Confirm live CLI details in Claude Code documentation.*

---

## 17. Quick reference card (print-friendly)

| Need | Reach for |
| --- | --- |
| Unfamiliar multi-file change | Plan mode |
| Context full, direction still good | Directed `/compact` |
| Contradictory / polluted session | Rewind |
| Always-true project facts | Lean CLAUDE.md |
| Occasional multi-step procedure | Skill |
| Must never happen | Deny rule + hook |
| Weekly boring job, little custom env | Routine |
| Needs your secrets/network/scripts | Headless `-p` |
| Agent inside your product | Agent SDK |
| Team-wide portable setup | Plugin |
| Parallel independent tasks | Worktrees |
| Unsupervised trust | Real test gates + human high-risk review |

**One-sentence exam mantra:** Advise with CLAUDE.md and skills; enforce with permissions and hooks; automate only after verification matches how little you watch.

**Common distractors on multiple choice:**

- "Add it to the prompt again" when the real fix is a hook
- "Use bypass for speed" when the environment is a developer laptop
- "Compact" when the session needs rewind
- "One shared branch for two agents" when worktrees are required
- "Claude said tests passed" when CI/hooks must execute the runner

---

# CCAR-F Domain 3 mechanics supplement (added 2026-08-23)

> **Scope:** the concrete configuration mechanics the **CCAR-F exam guide Domain 3 (Claude Code Configuration & Workflows, 20%)** tests, which the course notes above cover only conceptually — task statements 3.1 (CLAUDE.md hierarchy), 3.2 (commands & skills), 3.3 (path-specific rules), 3.4 (plan mode / Explore), 3.6 (CI/CD flags). All file paths and frontmatter keys below verified against current Claude Code docs (2026-08); original synthesis, volatile details hedged.

## S1. CLAUDE.md hierarchy, @import, and /memory (task 3.1)

**Three levels the exam names:**

| Level | Location | Shared? |
| --- | --- | --- |
| **User** | `~/.claude/CLAUDE.md` | No — personal, applies to all your projects, **not** in version control |
| **Project** | `./CLAUDE.md` (root) or `./.claude/CLAUDE.md` | Yes — via version control, whole team |
| **Directory** | `CLAUDE.md` in a subdirectory | Yes — loaded **on demand** when Claude works with files in that directory |

Files **concatenate** (broader scope first, closer-to-work read last); they do not override each other. *(Current docs add a managed/org policy level above user — org-wide, cannot be excluded — and a gitignored `CLAUDE.local.md` for personal per-project notes; know they exist.)*

**The diagnostic scenario the exam likes:** a new team member's Claude ignores "the team's" instructions → the instructions live in someone's **user-level** `~/.claude/CLAUDE.md` instead of **project-level** config, so version control never delivered them.

**`@import` syntax:** reference external files from CLAUDE.md with `@path/to/file` — the imported file is expanded into context at launch. Use it to keep CLAUDE.md modular, e.g. each package's CLAUDE.md imports only the standards files relevant to it. (Details verified: relative paths resolve against the *importing file*; imports recurse to a maximum depth of **four hops** (per current official docs, verified 2026-08-23); wrap a path in backticks to *mention* it without importing.)

**`/memory`** lists the memory files (CLAUDE.md and friends) across scopes and lets you open/edit them — use it to **verify which memory files are loaded and diagnose inconsistent behavior across sessions**. *(Current docs: `/context` additionally confirms what actually loaded into this session.)*

## S2. Custom slash commands and skills (task 3.2)

**Commands — the scope rule (official sample Q4 hinges on it):**

| Location | Scope |
| --- | --- |
| **`.claude/commands/`** in the repo | Project — **version-controlled, automatically available to every developer on clone/pull** |
| **`~/.claude/commands/`** | User — personal, not shared |

A markdown file `.claude/commands/review.md` defines `/review`. Team-standard workflows → project scope; personal helpers → user scope. *(Current docs note: custom commands have merged into skills — `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both create `/deploy`; existing commands keep working. The exam-facing answer for "team-shared /review command" remains `.claude/commands/` in the project repository.)*

**Skills** live at `.claude/skills/<name>/SKILL.md` (project) or `~/.claude/skills/` (personal). The frontmatter keys the exam names, verified current:

| Frontmatter key | What it does |
| --- | --- |
| **`context: fork`** | Runs the skill in an **isolated sub-agent context**, keeping verbose skill output (codebase analysis, brainstorming) from **polluting the main conversation** |
| **`allowed-tools`** | Tools Claude may use **without permission prompts during the skill's invoking turn** — restrict to what the skill needs (e.g. file-write-only to prevent destructive actions) |
| **`argument-hint`** | Autocomplete hint showing expected arguments (e.g. `[issue-number]`) when a developer invokes the skill without them |

**Personal variants without team impact:** create your own version under `~/.claude/skills/` **with a different name**, leaving the team's project skill untouched.

**Skills vs CLAUDE.md (the exam's dividing line):** skills = **on-demand invocation for task-specific workflows** (body loads only when used); CLAUDE.md = **always-loaded universal standards**. A growing CLAUDE.md section that has become a *procedure* should move to a skill.

## S3. Path-specific rules — `.claude/rules/` (task 3.3)

Rule files are markdown in **`.claude/rules/`** with **YAML frontmatter whose `paths:` field holds glob patterns**:

```markdown
---
paths:
 - "src/api/**/*"
 - "**/*.test.tsx"
---
# API + test conventions
- All endpoints validate input
- Tests use the team fixture factory
```

- A rule **loads only when Claude works with files matching its globs** — irrelevant conventions stay out of context, saving tokens (rules *without* `paths` load every session, like CLAUDE.md).
- **Globs beat directory-level CLAUDE.md when a convention spans directories** (official sample Q6): test files sit beside sources throughout the tree, so `**/*.test.tsx` catches them all — per-directory CLAUDE.md files can't, root-CLAUDE.md-with-headers relies on inference, and skills require invocation. Rules-with-globs is the deterministic, automatic answer.
- Example from the guide's own exercises: `paths: ["terraform/**/*"]` so Terraform conventions load only when editing Terraform.

## S4. Plan mode, direct execution, and the Explore subagent (task 3.4)

Named decision rule (§2.1 above covers the workflow; the exam wants the *criteria* verbatim):

- **Plan mode** — complex tasks: large-scale changes, **multiple valid approaches**, architectural decisions, multi-file modifications (monolith→microservices; a 45-file library migration). Safe exploration and design **before committing to changes** prevents costly rework (sample Q5's correct A).
- **Direct execution** — simple, well-scoped changes: single-file bug fix with a clear stack trace; adding one validation conditional.
- **Combine them:** plan mode for investigation, then direct execution for the planned implementation.
- **The Explore subagent** isolates **verbose discovery output** and returns summaries, **preserving main-conversation context** during multi-phase work — name it when a question describes context exhaustion during codebase discovery. (Ties to Domain 5.4 subagent delegation.)

## S4b. Iterative refinement techniques (task 3.5)

- **Concrete input/output examples beat prose** when natural-language descriptions of a transformation are interpreted inconsistently — provide **2–3 example pairs** showing input and expected output.
- **Test-driven iteration:** write the test suite first (expected behavior, edge cases, performance), then iterate by **sharing test failures** to guide progressive improvement; specific failing cases with example input/expected output fix edge-case handling (e.g. null values in migration scripts).
- **The interview pattern:** in an unfamiliar domain, have Claude **ask you questions first** to surface considerations you hadn't anticipated (cache invalidation strategy, failure modes) *before* implementing.
- **Interacting vs independent issues:** fixes that **interact** → one single detailed message addressing all of them together; **independent** issues → sequential iteration is fine.

## S5. CI/CD integration flags (task 3.6)

| Mechanic | Exam fact |
| --- | --- |
| **`-p` / `--print`** | Runs Claude Code **non-interactively**: processes the prompt, writes to stdout, exits — the fix when a pipeline job "hangs waiting for interactive input" (sample Q10; distractors like `CLAUDE_HEADLESS=true` or `--batch` are **non-existent features**) |
| **`--output-format json`** | Machine-parseable output in print mode (`text`, `json`, `stream-json`) |
| **`--json-schema`** | Print-mode flag producing **validated JSON matching your schema** — structured findings ready for automated posting as PR comments |
| **CLAUDE.md in CI** | The mechanism for giving CI-invoked Claude project context: testing standards, fixture conventions, review criteria — better test generation, fewer low-value cases |
| **Session context isolation** | The session that **generated** code is less effective at **reviewing** it (it retains its own reasoning context and won't question its decisions) — run review as an **independent instance** (ties to Domain 4.6 multi-instance review) |
| **Re-review without duplicates** | Include **prior review findings** in context when re-running after new commits; instruct Claude to report only new or still-unaddressed issues |
| **Test generation without duplicates** | Provide **existing test files** in context so generated tests skip already-covered scenarios |

## S6. Supplement Q&A

**SQ1.** *(= official sample Q4)* A `/review` command must reach every developer on clone/pull. Where does the file go?
**A.** `.claude/commands/` in the project repository — version-controlled project scope. `~/.claude/commands/` is personal; CLAUDE.md holds instructions, not command definitions; `.claude/config.json` with a commands array doesn't exist.

**SQ2.** *(= official sample Q6)* React, API, and DB conventions differ; test files sit beside sources everywhere and share one convention. Most maintainable mechanism?
**A.** `.claude/rules/` files with YAML frontmatter glob patterns (`**/*.test.tsx`) — conditional, automatic, location-independent. Root-CLAUDE.md headers rely on inference; per-directory CLAUDE.md can't span the tree; skills need invocation.

**SQ3.** New teammate doesn't get "the team's" instructions. Likely cause?
**A.** Instructions live in user-level `~/.claude/CLAUDE.md` (not shared via version control) instead of project-level `CLAUDE.md`/`.claude/`.

**SQ4.** How do you keep CLAUDE.md modular per package?
**A.** `@import` syntax (`@docs/standards.md`) — each package's CLAUDE.md imports only relevant standards files.

**SQ5.** Which command verifies which memory files are loaded?
**A.** `/memory` (and `/context` confirms what loaded this session).

**SQ6.** A codebase-analysis skill floods the main conversation with output. Frontmatter fix?
**A.** `context: fork` — runs the skill in an isolated sub-agent context.

**SQ7.** Restrict a skill to file writes to prevent destructive actions — which key?
**A.** `allowed-tools` in SKILL.md frontmatter (grants scoped, prompt-free tool use for the invoking turn).

**SQ8.** Developers invoke `/deploy` without required parameters. Which key prompts them?
**A.** `argument-hint` (e.g. `[environment] [version]`).

**SQ9.** You want a personal variant of a team skill without affecting teammates.
**A.** Create it in `~/.claude/skills/` under a **different name**.

**SQ10.** Skill or CLAUDE.md: a 20-step release procedure vs "we use 2-space indent"?
**A.** Procedure → skill (on-demand); universal standard → CLAUDE.md (always loaded).

**SQ11.** When do `paths:`-scoped rules load?
**A.** Only when Claude works with files matching the globs; rules without `paths` load every session.

**SQ12.** Monolith→microservices restructuring: plan mode or direct?
**A.** Plan mode — large-scale, multiple valid approaches, architectural decisions (sample Q5). Single-file bug fix with a stack trace → direct execution.

**SQ13.** Multi-phase task exhausts context during discovery. Named remedy?
**A.** The Explore subagent — verbose discovery isolated, summaries returned.

**SQ14.** CI job hangs: "Claude Code is waiting for interactive input." Fix?
**A.** `claude -p "..."` — the `-p`/`--print` flag is the documented non-interactive mode (sample Q10; `CLAUDE_HEADLESS`/`--batch` don't exist).

**SQ15.** Structured, machine-postable review findings from CI — which flags?
**A.** `--output-format json` with `--json-schema` (print mode).

**SQ16.** Why shouldn't the session that wrote the code review it?
**A.** Session context isolation: it retains generation reasoning and won't question its own decisions — use an independent review instance.

**SQ17.** Re-running review after new commits spams duplicate comments. Fix?
**A.** Include prior findings in context and instruct: report only new or still-unaddressed issues.

**SQ18.** Generated tests duplicate existing coverage. Fix?
**A.** Provide existing test files in context so generation avoids covered scenarios.

## S7. Supplement checklist

- [ ] I can place an instruction at user / project / directory level and predict who receives it.
- [ ] I can use `@import` and explain what `/memory` shows.
- [ ] I know `.claude/commands/` vs `~/.claude/commands/` cold (sample Q4).
- [ ] I can write a `.claude/rules/` file with `paths:` globs and argue it over per-directory CLAUDE.md (sample Q6).
- [ ] I can recite `context: fork`, `allowed-tools`, `argument-hint`.
- [ ] I can classify plan mode vs direct execution and name the Explore subagent.
- [ ] I know `-p`, `--output-format json`, `--json-schema`, and the three CI context tricks (CLAUDE.md, prior findings, existing tests).
