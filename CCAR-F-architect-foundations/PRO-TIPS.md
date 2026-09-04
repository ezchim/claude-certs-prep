# CCAR-F Pro Tips — how to take this exam well

*Companion to files 03–05 (+ supplements), 08, 09; distilled 2026-08-23 from
the official Exam Guide v1.0 and this pack. Exam craft only.*

**What this exam actually rewards:** diagnosing agentic-system failures from
production evidence and choosing the *proportionate* fix. The recurring axis
is **deterministic vs probabilistic**: prompts and few-shots lower failure
rates; gates, hooks, and code make them zero. Every time a stem mentions
compliance, ordering, money, or "must never," the graded answer lives on the
deterministic side.

---

## The numbers that drive strategy

- **60 items / 120 min → 2 min each**, but the items cluster into **4 scenarios
 drawn from a bank of 6 — and all 6 are printed in your exam guide.** This is
 the biggest legal advantage on any of the four exams: study the scenarios
 themselves (Customer Support Resolution Agent, Code Generation with Claude
 Code, Multi-Agent Research System, Developer Productivity, Claude Code for
 CI, Structured Data Extraction). Walk into the room already knowing the
 four-ish systems you'll be debugging.
- **Domain 1 is 27%** — the Agent SDK orchestration file (08) carries more
 weight than any two other files combined. D3 + D4 are 20% each.
- **720 scaled, no guessing penalty, multiple-response all-or-nothing.**
- **The out-of-scope list is a weapon:** cloud-provider config, streaming
 implementation, pricing/rate limits, embeddings, prompt-caching internals,
 computer use, vision are all excluded. An option resting on one of these is
 a distractor by definition — kill it without deliberation.

## If the stem says X → think Y

| Stem signal | Reflex |
| --- | --- |
| Loop won't stop / stops early / parses "I'm done!" | `stop_reason` (`tool_use` continue, `end_turn` stop) — never string matching, never an iteration cap as primary |
| Report has clean subagent logs but bad coverage | **Coordinator decomposition too narrow** — blame the assignments, not the workers (sample Q7) |
| Agent "can't delegate" | `"Task"` missing from the coordinator's `allowedTools` |
| Subagent "doesn't know" something the coordinator learned | Context isolation — findings must be passed **in the spawn prompt**; nothing flows implicitly |
| Steps skipped in a compliance/financial sequence | **Programmatic prerequisite / PreToolUse-style gate**, never "strengthen the prompt" (sample Q1) |
| Heterogeneous tool-result formats confusing the agent | **PostToolUse hook** normalizing results before the model sees them |
| "Must never execute above $N" | Tool-call interception → block + redirect to human escalation |
| Inconsistent depth / contradictions in a big multi-file review | Attention dilution → per-file passes + separate integration pass — NOT a bigger context window (sample Q12) |
| Resume after heavy code changes | Tell it which files changed, or fresh session + injected summary — stale tool results are poison |
| Explore two designs from one expensive analysis | `fork_session` per direction; the original is untouched |
| Team's Claude Code behavior differs per developer | Project-level config (`.claude/commands/`, project CLAUDE.md, rules) vs someone's user-level file — version control is the tell |
| Convention spans scattered files | `.claude/rules/` with `paths:` globs — beats per-directory CLAUDE.md and inference from headers |
| Escalating to a human | Structured handoff payload (ID, root cause, amounts, attempts, recommendation) — the human can't see the transcript |

## Terminology discipline

The guide (and therefore the item bank) says **Task tool** — answer with the
guide's vocabulary even though current Claude Code renamed it Agent. Same
principle everywhere: this exam is graded against Exam Guide v1.0 terms; file
08 flags the current-docs deltas so you're never surprised, but don't
"correct" the exam.

## Distractor archetypes to expect

- **The prompt patch** for a determinism problem ("add to the system prompt
 that it must always…"). Lowers, never eliminates — wrong when the stem says
 must/never/compliance.
- **The bigger window** for attention dilution or coverage problems.
- **The out-of-scope lure:** Bedrock IAM, pricing tiers, embedding stores —
 plausible-sounding, explicitly excluded.
- **The blame-shift:** fixing the subagent when the coordinator's
 decomposition or the missing context hand-off is the root cause.
- **The transcript assumption:** answers that assume a human or subagent can
 see conversation history it was never given.

## Freshness watchlist (re-verify the week of your exam)

Exact SDK option spellings (`agents`, `fork_session`/`forkSession`, hook
callback fields) at code.claude.com/docs/en/agent-sdk — file 08's footer
lists them. Five minutes; the concepts won't move, the spellings might.

## The 48-hour plan

1. Re-read 08 end-to-end, then 09 (together = 42% of the form).
2. Re-read the two supplements (04's Domain-2 mechanics, 05's Domain-3
 mechanics) — these are dense fact sections; recency helps.
3. Walk each of the 6 printed scenarios and predict its failure modes from the
 reflex table above — this is the closest thing to seeing the paper early.
4. Re-do the 12 sample questions cold; every one should now feel mechanical.
5. Skip files 06–07 entirely (out of scope) and 01–02 (background).

## Day-of (online proctored)

ID, clean desk, room scan, stable connection. When a scenario block opens,
spend 30 seconds identifying which of the 6 published scenarios it is — your
pre-walked failure-mode map then prices every question in it. Deterministic
beats probabilistic; coordinator before workers; guide terminology over
current docs. Answer everything.
