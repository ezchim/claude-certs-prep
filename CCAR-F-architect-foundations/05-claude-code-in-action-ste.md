---
title: Claude Code in Action — Exam-Prep Study Notes (Primary Source) — Simplified Technical English
source: https://academy.claude.com/courses/claude-code-in-action
disclaimer: Original study notes for exam prep — not official Anthropic material. Not a lesson transcript.
approx_length: STE edition (ASD-STE100) — primary study
deepened: 2026-08-23
---

# Claude Code in Action — Primary Study Notes

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Claude Code, MCP, CLAUDE.md, hooks, plan mode) are exceptions and stay as written. These notes are **original** exam-prep notes for Claude certification study. They are **not** official Anthropic material. They are **not** a lesson transcript.

> **Disclaimer:** These notes align to the **public** course outline at [Claude Code in Action](https://academy.claude.com/courses/claude-code-in-action) (objectives and module themes only). They are not Claude Academy lesson transcripts or verbatim slide text. Completing the official course remains the source of truth for quizzes. Check CLI flags and UI labels in current Claude Code docs. Names change.

**Who it is for:** Developers who already use Claude Code for single prompts and want longer, less supervised, team-wide workflows.

**Prerequisites (course):** Claude Code for one-shot tasks. Basic Git and command line.

**How to use these notes:** Read one major section. Then answer the self-check. Do not scroll back. Treat tables as flashcards. The ladder **steer → configure → automate → verify/share** is the main structure of most scenario questions.

---

## 1. Course map (public objectives)

The public page frames four outcome clusters (about 9 lessons, 1 quiz):

1. **Steer the work** — plan mode, directed compaction, rewind, hands-on vs autonomous goal/loop runs, safe parallelism.
2. **Configure Claude** — lean `CLAUDE.md`, skills, permission modes, hooks for rules that you cannot skip.
3. **Automate repeat work** — routines on Anthropic infrastructure, headless mode in your pipeline, PR / GitHub Action wiring.
4. **Verify and share** — verify unsupervised runs in proportion to how little you watch them. Gate on real tests. Package a trusted setup as a plugin.

**Mental model:** Claude Code is an agentic coding loop with tools (file edit, bash, search, and others). Your job on exams is not "write a longer prompt." Your job is to **select the correct control surface** for the risk of the job.

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

For changes that are not small, start in **plan mode**. Claude may read the repo and run exploratory commands. Claude does **not edit source** until you approve a plan. Use it when:

- The codebase is unfamiliar
- The change touches many files or modules
- You need architectural decisions before code
- The scope of harm includes migrations, auth, billing, or deploy scripts

Cycle modes in the CLI (commonly **Shift+Tab**). Learn the *intent* of each mode more than any single UI string. Labels can change. The spectrum from "read-only exploration" to "auto-approve edits" to "bypass prompts" does not change.

**Exam view:** Plan mode is a *steering* choice. It is not a permanent project setting. After you approve a plan, you typically switch to a mode that allows implementation. Use permissions that you trust.

### 2.2 Compaction

Long sessions fill the context window. Claude Code **compacts** older turns. You can also start compaction by hand in many setups (for example `/compact`). Summaries drop detail.

**What survives compaction poorly:** single-use chat rules ("always use pytest"), temporary naming decisions, and failed experiment paths that you still need as negative examples.

**What should survive by design:**

- Durable conventions → **`CLAUDE.md`** / `.claude/rules/`
- On-demand procedures → **skills**
- Hard stops → **hooks** / deny rules

When you *direct* compaction (keep X, drop Y), prefer to keep: approved plan, open decisions, constraints, names of tests that fail. Drop verbose tool output, abandoned experiment paths, and duplicate file listings.

**Common exam error:** "Compaction lost our coding standards" → The root cause is standards that exist only in chat, not in CLAUDE.md.

### 2.3 Rewind

When a session drifts, **rewind** to an earlier clean point. Do not stack contradictory fixes on a mixed context. Then restate **one** clear goal.

Rewind vs compact:

| Action | Best when | Risk if misused |
| --- | --- | --- |
| Rewind | Wrong path / contradictory instructions | You lose useful later work if you rewind too far |
| Compact | Context full but direction still good | You lose detail. A messy chat may reinforce bad summaries |

**Pattern:** Drift → rewind → single goal → (optional) plan mode again → implement.

### 2.4 Hands-on vs autonomous

| Style | Use when | Control |
| --- | --- | --- |
| Hands-on steering | Novel / high scope of harm | You approve edits and watch turns |
| Goal + loop / autonomous | Trusted, well-scoped tasks | Hooks, tests, tighter permission mode |
| Parallel agents | Independent subtasks | **Git worktrees** (or separate checkouts) |

**Exam fact:** Parallel agents that share one dirty working tree are the wrong pattern. Isolation via worktrees is the safe answer.

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

**Core distinction:** "Never edit `.env`" in `CLAUDE.md` is a polite request. A **deny** rule or **PreToolUse hook** that blocks `.env` is a guarantee. Instructions shape intent. Rules and hooks change what is allowed.

### 3.1 Lean CLAUDE.md

Keep it short enough that Claude follows it. Public guidance and good exam answers agree on this:

- Project purpose, stack, layout (few bullets)
- Commands that matter (test, lint, build)
- Hard do/do not for secrets, migrations, force-push
- Pointers to skills/docs for long procedures — do **not** paste a very long document

Put each rule on the right surface:

| Content type | Surface |
| --- | --- |
| Always-true facts and conventions | CLAUDE.md |
| Path-scoped conventions | `.claude/rules/` (when available) |
| Multi-step / occasional workflows | Skills |
| Must-run-at-event guarantees | Hooks |
| Tool allow/deny | Permission rules |

`/init` (or equivalent) maps the repo and starts CLAUDE.md. Then refine with what Claude cannot find from code alone.

**Anti-pattern:** A 2,000-line CLAUDE.md that duplicates the README, the wiki, and every coding standard PDF. Claude will not follow it fully. Exams reward *lean* memory.

**CLAUDE.md maintenance loop:** After a difficult session, ask: "What should never be found from scratch again?" Put that fact into CLAUDE.md. If it is a procedure, put it in a skill instead.

### 3.2 Skills

**Skills** package repeated procedures (start with a verification skill that runs real checks). Typical skill contents conceptually:

- When to apply
- Steps / commands
- Success criteria
- Optional allowed-tools grants for the invoking turn (still subject to baseline permissions)

Skills load **on demand** (invoked or auto-selected when relevant). That is why they are better than putting every procedure into CLAUDE.md.

**Exam contrast:**

- CLAUDE.md = always in context → keep short
- Skill = load when needed → detailed OK
- Hook = runs regardless of Claude's "opinion" → use for rules you cannot skip

Good first skills for a team: "run verification suite," "create PR with checklist," "migrate DB safely," "security review touchpoints."

### 3.3 Plugins

**Plugins** bundle CLAUDE.md pieces + skills + hooks + related config (and sometimes MCP / subagents). The whole team installs one trusted setup. Plugins are about **portability**. They are not new model capabilities.

Share the plugin when the setup is verified. Hooks must gate real tests. Permissions must be least-privilege. Onboarding then changes from "read a wiki" to "install plugin."

Plugin hygiene checklist:

1. Document required env vars and secrets injection points
2. Fail closed if tests cannot run
3. Namespace skills so multiple plugins coexist
4. Version the plugin. Break changes need a changelog
5. Never include bypass-by-default in a team plugin
---

## 4. Permission modes (match mode to risk)

Modes set the baseline before allow/ask/deny rules. Names change. Know the spectrum:

| Mode (conceptual) | Behavior | Typical job |
| --- | --- | --- |
| Default / manual | Prompt on first tool use | Interactive pairing |
| Accept edits | Auto-accept common workspace edits | Trusted local feature work |
| Plan | Explore. No source edits until plan approved | Design before implement |
| Auto (classifier) | Approves routine actions. Blocks risky ones | Faster interactive with guardrails |
| Do not ask | Strict. Often denies unless pre-allowed | Controlled non-interactive |
| Bypass / do not ask risky variant | Skips most prompts | Isolated CI / sandbox **only** |

**Evaluation order (mental model):** deny → ask → allow. First match wins. Deny wins over allow. A PreToolUse hook that exits with a blocking decision can still stop a call. This is true even when an allow rule or bypass mode would otherwise pass. Hooks are the last hard gate in many designs.

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
- Assuming CLAUDE.md overrides a permission deny → wrong
- Using plan mode for a trusted, already-specified one-line rename → overly slow but not "unsafe". Usually accept-edits or auto is fine
- Believing auto mode replaces deny rules for `.env` → wrong. The classifier is probabilistic. Deny is absolute

Admins may disable bypass in team settings. Know that organizational policy can remove the bypass option.

---

## 5. Hooks: unskippable control

Hooks fire at fixed lifecycle points. Examples worth memorizing (exact event names may change — learn the *jobs*):

| Lifecycle job | Example use |
| --- | --- |
| Session start | Inject env facts, repo branch warnings |
| Pre tool use | Block `.env`, force ask on `rm -rf`, deny production deploys |
| Permission request | Auto-decide in non-interactive runs |
| Pre compact | Re-inject critical constraints into the summary |
| Stop / after turn | Run linters. Fail closed if tests are red |
| File / rules loaded | Audit what entered context |

Patterns:

1. Block protected paths (`.env`, production configs, private keys)
2. Gate a turn on **real** test runners (fail closed if red)
3. Force a prompt for sensitive Bash even if Bash is broadly allowed
4. Re-inject critical context after compaction
5. Return permission-decision JSON / exit codes to allow, deny, or ask

**Hooks enforce. Prompts advise.**

**Non-interactive note:** In headless `-p` runs, interactive permission prompts may not exist. Rely on PreToolUse hooks and pre-declared allow/deny rules. Do not assume a human will click "allow."

**Hook design tips for exams:**

- Prefer fail-closed for safety checks (missing test runner = deny)
- Keep hooks fast. Slow hooks make interactive use painful, and people disable them.
- Log denials with enough detail for audit without dumping secrets
- Do not put irreversible destructive capabilities behind soft asks alone

---

## 6. Automation spectrum

When you trust a task, stop starting it by hand:

1. **Routines** — schedule prompts on Anthropic-managed infrastructure (lowest operations effort).
2. **Headless** — `-p` / `--print`: one-shot, no TUI. Stdin/stdout for pipes and CI.
3. **Bare / deterministic flags** — prefer when CI needs repeatable, low-variance runs.
4. **Agent SDK** — embed Claude Code inside your TypeScript/Python product.
5. **PR / GitHub Action wiring** — managed review patterns. Claude in the pull-request loop.

**Decision rule:** start with routines. Change to headless when the job needs *your* environment or surrounding script logic. Use the Agent SDK when the agent *is* part of your product.

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

Automation without proportional verification is incomplete. That sentence is an exam answer by itself.

**Headless specifics worth remembering:**

- Non-interactive means no human in the permission loop
- Pipe prompts via stdin or args. Capture stdout for logs
- Pair with explicit permission allowlists and PreToolUse hooks
- Prefer deterministic settings when CI flakes would cost a lot

---

## 7. Verify unsupervised work

**When you watch less, you verify more.**

| How unsupervised | Minimum verification |
| --- | --- |
| Interactive pair | Spot-check diffs |
| Accept-edits local run | `git diff` + smoke tests |
| Auto mode long session | Diff review + targeted tests + hook gates |
| Headless / scheduled / bypass | Mandatory automated tests, deny rules for dangerous tools, human review of high-risk paths |

Do not trust Claude's claim that "tests passed." Execute the runner in a hook or CI step.

**Verification skill pattern:** Package "run unit + lint + typecheck. Print summary. Non-zero exit on failure" as a skill. Also wire the same commands into a Stop/PreToolUse gate for unattended runs.

**Risk-tiered review:**

- Low: formatting, comments, docs → automated checks may suffice
- Medium: app logic → tests + a quick read of the diff
- High: auth, payments, migrations, IAM → mandatory human review regardless of green CI
---

## 8. Practical end-to-end workflow (cert checklist)

1. Initialize / map the repo (`/init` or equivalent).
2. Feed relevant files / point at the subsystem before you ask for a feature.
3. Plan (no code) then approve then implement.
4. Prefer test-aware loops when quality matters.
5. Encode durable rules in CLAUDE.md. Procedures in skills. Hard stops in hooks.
6. Choose permission mode for the job risk.
7. Automate only after verification is reliable. Share via plugin.
8. If drift: rewind, then one goal, then continue.
9. Before you ship plugin: prove hooks fail closed on a deliberate red test.

**Session hygiene habits:**

- One goal per period of autonomous work
- Commit or stash before large autonomous runs so rewind or git restore is easy
- Name branches clearly when you use worktrees
- After compaction, restate the current plan in one sentence

---

## 9. Anti-patterns (high-yield exam recognition)

| Anti-pattern | Why it fails | Prefer |
| --- | --- | --- |
| Safety only in chat or CLAUDE.md | Advisory. Ignored under pressure | Deny plus PreToolUse hook |
| Bypass on laptop with prod creds | Huge scope of harm | Sandbox CI plus least privilege |
| Automate before tests exist | Unsupervised bad output at scale | Write gates first |
| Two agents, one dirty branch | Merge chaos or lost work | Worktrees |
| Trust model claim that tests passed | Hallucinated verification | Hook executes runner |
| Very long text in CLAUDE.md | Too much text. Claude does not follow it fully | Skills plus short CLAUDE.md |
| Compact instead of rewind after contradiction | A bad summary keeps the confusion | Rewind |
| Plan mode forever | The work never ships | Approve plan then implement mode |
| Plugin with bypass default | Spreads unsafe defaults | Least privilege defaults |
| Ignoring Stop-hook failures | Broken code shown as correct | Fail closed |
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

Exams often use scenarios where you select a higher layer than needed. You must have Example: you add a very long document to CLAUDE.md when a deny rule. Exams also use scenarios where you pick a lower layer than needed. Example: you rely on chat alone to stop force-push.

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
- Compaction without CLAUDE.md loses conventions. That failure mode is testable.
- Bypass without isolation is the wrong safety answer.
- Hooks can block even when modes are permissive. Know the evaluation order.
- Plugins package trust. They do not replace verification.
- Headless without hooks is an incomplete answer for unattended safety.
---

## 12. Self-check Q&A (with answers)

**Q1.** CLAUDE.md says never commit secrets, but Claude Code still stages a secret env file. What stops it?
**A1.** Deny permission rules and/or a PreToolUse hook — not the markdown sentence alone.

**Q2.** When is plan mode the right default?
**A2.** Before large or unclear edits: explore and propose without changing source until approval.

**Q3.** Session is confused after many failed attempts. Next step?
**A3.** Rewind (or carefully compact), then one clear goal. Avoid contradictory stacked instructions.

**Q4.** Same prompt every Monday. First automation choice?
**A4.** A routine. Change to headless if you need local env or custom piping.

**Q5.** Why CLAUDE.md instead of only chat?
**A5.** Claude Code compacts chat content away. CLAUDE.md is the durable project instruction surface.

**Q6.** Two agents on one branch — safer pattern?
**A6.** Isolate with git worktrees / separate checkouts.

**Q7.** What does headless `-p` provide?
**A7.** Non-interactive, scriptable runs for pipelines and pipes.

**Q8.** How much verification for fully unattended runs?
**A8.** Maximum: test gates, deny rules for dangerous tools, and review policy. Match the policy to zero human watching.

**Q9.** Skill vs CLAUDE.md for a 20-step release checklist?
**A9.** Skill (on-demand procedure). Keep CLAUDE.md lean with a pointer.

**Q10.** Accept-edits vs bypass for trusted local feature work on a developer machine?
**A10.** Accept-edits (or auto with classifier). Bypass belongs in isolated sandboxes/CI, not default laptop use.

**Q11.** What should a PreCompact hook often do?
**A11.** Re-inject critical constraints so compaction summaries do not drop rules you cannot skip.

**Q12.** Plugin vs skill?
**A12.** Skill = one packaged procedure/knowledge unit. Plugin = installable bundle of skills, hooks, config (portability layer).

**Q13.** Auto mode still does something dangerous. Your next hardening step?
**A13.** Add an explicit deny rule and/or PreToolUse hook for that action class. Do not rely on the classifier alone for rules you cannot skip.

**Q14.** When choose Agent SDK over headless CLI?
**A14.** When the coding agent is embedded inside your product TypeScript/Python application. Do not use it only as a call from a shell pipeline.

**Q15.** GitHub Action runs Claude on every PR. What must exist first?
**A15.** Proportional verification: real tests, scoped permissions, and review of high-risk paths. Automation without gates is incomplete.

**Q16.** Evaluation order if allow and deny both match a tool path?
**A16.** Deny wins (first-match mental model: deny before allow).

**Q17.** Why might hooks still matter in bypass mode?
**A17.** PreToolUse hooks can deny regardless of permission mode. They enforce policy. Users cannot skip this policy by flipping mode.

**Q18.** Parallel agents editing the same file — correct?
**A18.** No. Split independent work. Isolate trees. Merge with intent.

**Q19.** After `/init`, CLAUDE.md is huge and generic. What do you do?
**A19.** Trim to always-true facts. Move procedures to skills. Add only conventions that Claude cannot find from code.

**Q20.** Unattended job needs private package registry credentials. Routine or headless?
**A20.** Headless (or your CI), where the secrets and network are located. Do not use a cloud routine alone unless that infra has the secrets.

**Q21.** What is the ladder order for building team Claude Code maturity?
**A21.** Steer, then configure, then automate, then verify/share.

**Q22.** Spot-check diffs enough for scheduled overnight refactors?
**A22.** No. Require automated tests plus hooks plus human review of high-risk diffs.

**Q23.** Where do irreversible must-never rules belong?
**A23.** Permission deny rules and hooks — not only CLAUDE.md.

**Q24.** Directed compaction should prioritize keeping what?
**A24.** Approved plan, open decisions, constraints, and failing tests. Drop noisy output of failed experiments.

**Q25.** Why is a verification skill still valuable if CI already runs tests?
**A25.** Local interactive loops catch failures earlier. The same commands should also gate unattended Stop/PreToolUse hooks for consistency.
---

## 13. Review checklist (before exam)

- [ ] Can explain plan / compact / rewind and when to use each
- [ ] Can sort CLAUDE.md vs skills vs hooks vs permissions by advisory/enforced
- [ ] Can select permission mode for interactive, trusted local, CI, sandbox
- [ ] Can choose routine vs headless vs Agent SDK
- [ ] Can describe worktree isolation for parallel agents
- [ ] Can state verification proportional to supervision
- [ ] Can list five anti-patterns and the fix for each
- [ ] Can describe what a plugin packages and why teams use it
- [ ] Know hooks override permission mode for hard policy
- [ ] Know compaction loses chat-only conventions
- [ ] Can go through the steer → configure → automate → verify ladder
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
- **Advisory vs enforced** — prompt/CLAUDE.md advise. Permissions/hooks enforce.
- **Goal and loop** — autonomous style: clear goal, iterate until checks pass.
- **Classifier auto mode** — model-assisted approval of routine actions with risk blocks.
- **Fail closed** — if a safety check cannot run, treat as failure/deny.

---

## 15. Extra depth: subagents, MCP, and team rollout

### Subagents (conceptual)

Claude Code can spawn specialized subagents for isolated work (explore, plan, or custom). Isolation prevents the main session from growing too large. For exams: use subagents when the subtask does not need the full conversation. Keep dangerous tools constrained. Remember: background subagents in non-interactive mode cannot show interactive prompts. Hooks and denies decide.

### MCP inside Claude Code

MCP servers extend tools/resources/prompts available to the coding agent. Treat MCP tools like any other tools: permission rules and hooks still apply. Do not assume an MCP server is safe because it is "just config." It can expose powerful side effects.

### Team rollout sequence (practical)

1. One repo pilot: lean CLAUDE.md + verification skill + deny secrets
2. Add PreToolUse hooks for protected paths and test gates
3. Standardize permission defaults for interactive vs CI
4. Package as plugin. Install on second repo
5. Add routine/headless for one routine weekly task
6. Measure: fewer permission prompts that are noise, fewer incidents, faster PR checks
7. Only then expand autonomy (auto mode, more routines)

### Metrics that matter (not empty metrics)

- Time from issue → green PR
- Rate of hook denials that prevented real mistakes
- Flake rate of verification gates
- Onboarding time for a new engineer with the plugin
- Incidents involving unsupervised edits

### Study rhythm

Day 1: sections 1–5 (steer + configure). Flashcard advisory vs enforced.
Day 2: sections 6–10 (automate + verify + anti-patterns). Speak the decision trees aloud.
Day 3: all Q&A closed-book, then checklist. Re-read only missed items.

---

## 16. Reading tip

Each major heading is a flashcard front. The tables are the backs. Quiz yourself on advisory vs enforced. Then quiz on routines vs headless. Then quiz on plan/compact/rewind. Do this before you open the Academy outline again.

---

*Aligned to public outline at https://academy.claude.com/courses/claude-code-in-action. Use for recall. Complete the official course for quizzes. Confirm live CLI details in Claude Code documentation.*

---

## 17. Quick reference card (print-friendly)

| Need | Reach for |
| --- | --- |
| Unfamiliar multi-file change | Plan mode |
| Context full, direction still good | Directed `/compact` |
| Contradictory / mixed session | Rewind |
| Always-true project facts | Lean CLAUDE.md |
| Occasional multi-step procedure | Skill |
| Must never happen | Deny rule + hook |
| Weekly routine job, little custom env | Routine |
| Needs your secrets/network/scripts | Headless `-p` |
| Agent inside your product | Agent SDK |
| Team-wide portable setup | Plugin |
| Parallel independent tasks | Worktrees |
| Unsupervised trust | Real test gates + human high-risk review |

**One-sentence exam rule:** Advise with CLAUDE.md and skills. Enforce with permissions and hooks. Automate only after verification matches how little you watch.

**Common distractors on multiple choice:**

- "Add it to the prompt again" when the real fix is a hook
- "Use bypass for speed" when the environment is a developer laptop
- "Compact" when the session needs rewind
- You must have "One shared branch for two agents" when worktrees.
- "Claude said tests passed" when CI/hooks must execute the runner

---

# CCAR-F Domain 3 mechanics supplement (added 2026-08-23)

> **Scope:** the concrete configuration mechanics that the **CCAR-F exam guide Domain 3 (Claude Code Configuration & Workflows, 20%)** tests. The course notes above cover these only conceptually. Task statements: 3.1 (CLAUDE.md hierarchy), 3.2 (commands & skills), 3.3 (path-specific rules), 3.4 (plan mode / Explore), 3.6 (CI/CD flags). The author verified all file paths and frontmatter keys below against current Claude Code docs (2026-08). This is original synthesis. Volatile details are marked as possibly changing.

## S1. CLAUDE.md hierarchy, @import, and /memory (task 3.1)

**Three levels the exam names:**

| Level | Location | Shared? |
| --- | --- | --- |
| **User** | `~/.claude/CLAUDE.md` | No — personal, applies to all your projects, **not** in version control |
| **Project** | `./CLAUDE.md` (root) or `./.claude/CLAUDE.md` | Yes — via version control, whole team |
| **Directory** | `CLAUDE.md` in a subdirectory | Yes — loaded **on demand** when Claude works with files in that directory |

Files **concatenate** (broader scope first, closer-to-work read last). They do not override each other. *(Current docs add a managed/org policy level above user. It is org-wide. You cannot exclude it. Current docs also add a gitignored `CLAUDE.local.md` for personal per-project notes. Know that these exist.)*

**The diagnostic scenario that appears often on the exam:** a new team member's Claude ignores "the team's" instructions. The instructions are in someone's user-level `~/.claude/CLAUDE.md` instead of **project-level** config. Version control never delivers them to teammates.

**`@import` syntax:** reference external files from CLAUDE.md with `@path/to/file`. Claude Code expands the imported file into context at launch. Use it to keep CLAUDE.md modular. Example: each package's CLAUDE.md imports only the standards files that are relevant to it. (Details verified: relative paths resolve against the *importing file*. Imports recurse to a maximum depth of **four hops** (per current official docs, verified 2026-08-23). Wrap a path in backticks to *mention* it without importing.)

**`/memory`** lists the memory files (CLAUDE.md and related memory files) across scopes and lets you open/edit them. Load Use it to **verify which memory files and diagnose inconsistent behavior across sessions**. *(Current docs: `/context` additionally confirms what actually loaded into this session.)*

## S2. Custom slash commands and skills (task 3.2)

**Commands — the scope rule (official sample Q4 depends on it):**

| Location | Scope |
| --- | --- |
| **`.claude/commands/`** in the repo | Project — **version-controlled, automatically available to every developer on clone/pull** |
| **`~/.claude/commands/`** | User — personal, not shared |

A markdown file `.claude/commands/review.md` defines `/review`. Team-standard workflows → project scope. Personal helpers → user scope. *(Current docs note: Current docs merge custom commands into skills. `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both create `/deploy`. Existing commands keep working. The exam-facing answer for "team-shared /review command" remains `.claude/commands/` in the project repository.)*

Skills are located at `.claude/skills/<name>/SKILL.md` (project) or `~/.claude/skills/` (personal). The frontmatter keys the exam names, verified current:

| Frontmatter key | What it does |
| --- | --- |
| **`context: fork`** | Runs the skill in an **isolated sub-agent context**. This keeps verbose skill output (codebase analysis, brainstorming) from **filling the main conversation** |
| **`allowed-tools`** | Tools Claude may use **without permission prompts during the skill's invoking turn**. Restrict to what the skill needs (e.g. file-write-only to prevent destructive actions) |
| **`argument-hint`** | Autocomplete hint that shows expected arguments (e.g. `[issue-number]`) when a developer invokes the skill without them |

**Personal variants without team impact:** create your own version under `~/.claude/skills/` **with a different name**. Leave the team's project skill untouched.

**Skills vs CLAUDE.md (the exam's dividing line):** skills = **on-demand invocation for task-specific workflows** (body loads only when used). CLAUDE.md = **always-loaded universal standards**. A CLAUDE.md section that becomes a procedure should move to a skill.

## S3. Path-specific rules — `.claude/rules/` (task 3.3)

Rule files are markdown in **`.claude/rules/`**. The YAML frontmatter `paths:` field holds glob patterns:

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

- A rule **loads only when Claude works with files that match its globs**. Irrelevant conventions stay out of context. This saves tokens. (Rules *without* `paths` load every session, like CLAUDE.md.)
- **Globs are better than directory-level CLAUDE.md when a convention spans directories** (official sample Q6). Test files sit beside sources throughout the tree, so `**/*.test.tsx` catches them all. Per-directory CLAUDE.md files cannot do this. Root-CLAUDE.md-with-headers relies on inference. Skills require invocation. Rules-with-globs is the deterministic, automatic answer.
- Example from the guide's own exercises: `paths: ["terraform/**/*"]` so Terraform conventions load only when you edit Terraform.

## S4. Plan mode, direct execution, and the Explore subagent (task 3.4)

Named decision rule (§2.1 above covers the workflow. The exam wants the *criteria* as written):

- **Plan mode** — use it for complex tasks: large-scale changes, **multiple valid approaches**, architectural decisions, multi-file modifications (monolith→microservices. A 45-file library migration). Safe exploration and design before you commit to changes prevent costly rework (sample Q5's correct A).
- **Direct execution** — simple, well-scoped changes: single-file bug fix with a clear stack trace. Adding one validation conditional.
- **Combine them:** plan mode for investigation, then direct execution for the planned implementation.
- **The Explore subagent** isolates **verbose discovery output** and returns summaries. It **preserves main-conversation context** during multi-phase work. Name it when a question describes context exhaustion during codebase discovery. (Ties to Domain 5.4 subagent delegation.)

## S4b. Iterative refinement techniques (task 3.5)

- **Concrete input/output examples are better than prose** when the model interprets natural-language descriptions of a transformation in different ways. Provide **2–3 example pairs** showing input and expected output.
- **Test-driven iteration:** write the test suite first (expected behavior, edge cases, performance). Then iterate: **share test failures** to guide progressive improvement. Specific failing cases with example input/expected output fix edge-case handling (e.g. null values in migration scripts).
- **The interview pattern:** in an unfamiliar domain, have Claude **ask you questions first**. This shows considerations you did not anticipate (cache invalidation strategy, failure modes) *before* you implement.
- **Interacting vs independent issues:** fixes that **interact** → one single detailed message that addresses all of them together. **Independent** issues → sequential iteration is fine.

## S5. CI/CD integration flags (task 3.6)

| Mechanic | Exam fact |
| --- | --- |
| **`-p` / `--print`** | Runs Claude Code **non-interactively**: processes the prompt, writes to stdout, exits. This is the fix when a pipeline job "hangs waiting for interactive input" (sample Q10. Distractors like `CLAUDE_HEADLESS=true` or `--batch` are **features that do not exist**) |
| **`--output-format json`** | Machine-parseable output in print mode (`text`, `json`, `stream-json`) |
| **`--json-schema`** | Print-mode flag that produces validated JSON which matches your schema. Structured findings are ready for automated posting as PR comments |
| **CLAUDE.md in CI** | The mechanism for giving CI-invoked Claude project context: testing standards, fixture conventions, review criteria. Result: better test generation, fewer low-value cases |
| **Session context isolation** | The session that **generated** code is less effective at **reviewing** it. It retains its own reasoning context and will not question its decisions. Run review as an **independent instance** (ties to Domain 4.6 multi-instance review) |
| **Re-review without duplicates** | Include **prior review findings** in context when you re-run after new commits. Instruct Claude to report only new or still-unaddressed issues |
| **Test generation without duplicates** | Provide **existing test files** in context so generated tests skip already-covered scenarios |

## S6. Supplement Q&A

**SQ1.** *(= official sample Q4)* A `/review` command must reach every developer on clone/pull. Where does the file go?
**A1.** `.claude/commands/` in the project repository — version-controlled project scope. `~/.claude/commands/` is personal. CLAUDE.md holds instructions, not command definitions. `.claude/config.json` with a commands array does not exist.

**SQ2.** *(= official sample Q6)* React, API, and DB conventions differ. Test files sit beside sources everywhere and share one convention. Most maintainable mechanism?
**A2.** `.claude/rules/` files with YAML frontmatter glob patterns (`**/*.test.tsx`) — conditional, automatic, location-independent. Root-CLAUDE.md headers rely on inference. Per-directory CLAUDE.md cannot span the tree. Skills need invocation.

**SQ3.** New teammate does not get "the team's" instructions. Likely cause?
**A3.** Instructions are in user-level `~/.claude/CLAUDE.md` (not shared via version control) instead of project-level `CLAUDE.md`/`.claude/`.

**SQ4.** How do you keep CLAUDE.md modular per package?
**A4.** `@import` syntax (`@docs/standards.md`). Each package's CLAUDE.md imports only relevant standards files.

Load **SQ5.** Which command verifies which memory files?
**A5.** `/memory` (and `/context` confirms what loaded this session).

**SQ6.** A codebase-analysis skill fills the main conversation with output. Frontmatter fix?
**A6.** `context: fork` — runs the skill in an isolated sub-agent context.

**SQ7.** Restrict a skill to file writes to prevent destructive actions — which key?
**A7.** `allowed-tools` in SKILL.md frontmatter (grants scoped, prompt-free tool use for the invoking turn).

**SQ8.** Developers invoke `/deploy` without required parameters. Which key prompts them?
**A8.** `argument-hint` (e.g. `[environment] [version]`).

**SQ9.** You want a personal variant of a team skill without affecting teammates.
**A9.** Create it in `~/.claude/skills/` under a **different name**.

**SQ10.** Skill or CLAUDE.md: a 20-step release procedure vs "we use 2-space indent"?
**A10.** Procedure → skill (on-demand). Universal standard → CLAUDE.md (always loaded).

**SQ11.** When do `paths:`-scoped rules load?
**A11.** Only when Claude works with files matching the globs. Rules without `paths` load every session.

**SQ12.** Monolith→microservices restructuring: plan mode or direct?
**A12.** Plan mode — large-scale, multiple valid approaches, architectural decisions (sample Q5). Single-file bug fix with a stack trace → direct execution.

**SQ13.** Multi-phase task exhausts context during discovery. Named remedy?
**A13.** The Explore subagent — verbose discovery isolated, summaries returned.

**SQ14.** CI job hangs: "Claude Code is waiting for interactive input." Fix?
**A14.** `claude -p "..."` — the `-p`/`--print` flag is the documented non-interactive mode (sample Q10. `CLAUDE_HEADLESS`/`--batch` do not exist).

**SQ15.** Structured, machine-postable review findings from CI — which flags?
**A15.** `--output-format json` with `--json-schema` (print mode).

**SQ16.** Why should the session that wrote the code not review it?
**A16.** Session context isolation: it retains generation reasoning and will not question its own decisions. Use an independent review instance.

**SQ17.** Re-running review after new commits posts the same comments again. Fix?
**A17.** Include prior findings in context and instruct: report only new or still-unaddressed issues.

**SQ18.** Generated tests duplicate existing coverage. Fix?
**A18.** Provide existing test files in context so generation avoids covered scenarios.

## S7. Supplement checklist

- [ ] I can place an instruction at user / project / directory level and predict who receives it.
- [ ] I can use `@import` and explain what `/memory` shows.
- [ ] I know `.claude/commands/` vs `~/.claude/commands/` without notes (sample Q4).
- [ ] I can write a `.claude/rules/` file with `paths:` globs and argue it over per-directory CLAUDE.md (sample Q6).
- [ ] I can recite `context: fork`, `allowed-tools`, `argument-hint`.
- [ ] I can classify plan mode vs direct execution and name the Explore subagent.
- [ ] I know `-p`, `--output-format json`, `--json-schema`, and the three CI context tricks (CLAUDE.md, prior findings, existing tests).


