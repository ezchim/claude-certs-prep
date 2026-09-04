---
title: CCAR-F Pro Tips — how to take this exam well — Simplified Technical English
disclaimer: Original study notes — independent and not official course content
approx_length: STE edition (ASD-STE100) — exam craft
updated: 2026-08-23
---

# CCAR-F Pro Tips — how to take this exam well

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Claude Code, MCP, prompting, caching, Task tool, Agent SDK, PreToolUse, PostToolUse, CLAUDE.md) are exceptions and stay as written.

*Companion to files 03–05 (+ supplements), 08, 09. Distilled 2026-08-23 from the official Exam Guide v1.0 and this pack. Exam craft only.*

**What this exam actually rewards:** you diagnose agentic-system failures from production evidence. Then you select the *proportionate* fix. The recurring difference is deterministic versus probabilistic. Prompts and few-shots lower failure rates. Gates, hooks, and code make them zero. Every time a stem mentions compliance, ordering, money, or "must never," the graded answer is on the deterministic side.

---

## The numbers that drive strategy

- **60 items / 120 min → 2 min each.** The items cluster into 4 scenarios. The exam draws them from a bank of 6. All 6 are printed in your exam guide. This is the biggest legal advantage on any of the four exams. Study the scenarios themselves. The six names are Customer Support Resolution Agent, Code Generation with Claude Code, and Multi-Agent Research System. The other names are Developer Productivity, Claude Code for CI, and Structured Data Extraction. Enter the room with knowledge of about four of the six systems you will debug.
- **Domain 1 is 27%** — the Agent SDK orchestration file (08) has more weight than any two other files combined. D3 + D4 are 20% each.
- **720 scaled, no guessing penalty, multiple-response all-or-nothing.**
- **The out-of-scope list is a tool you can use.** The exam excludes these topics: cloud-provider config, streaming implementation, pricing and rate limits, embeddings. Also excluded: prompt-caching internals, computer use, vision. An option that uses one of these is a distractor by definition. Remove it at once. Do not spend time on it.

## If the stem says X → think Y

| Stem signal | Reflex |
| --- | --- |
| Loop will not stop. It stops early or parses a completion phrase | `stop_reason` (`tool_use` continue, `end_turn` stop) — never string matching, never an iteration cap as primary |
| Report has clean subagent logs but bad coverage | **Coordinator decomposition too narrow** — the cause is the assignments, not the workers (sample Q7) |
| Agent "cannot delegate" | `"Task"` missing from the coordinator's `allowedTools` |
| Subagent "does not know" something the coordinator learned | Context isolation — you must pass findings **in the spawn prompt**. Nothing flows implicitly |
| Steps skipped in a compliance/financial sequence | **Programmatic prerequisite / PreToolUse-style gate**, never "strengthen the prompt" (sample Q1) |
| Heterogeneous tool-result formats that confuse the agent | **PostToolUse hook** that normalizes results before the model sees them |
| "Must never execute above $N" | Tool-call interception → block + redirect to human escalation |
| Inconsistent depth / contradictions in a big multi-file review | Attention dilution → per-file passes + separate integration pass — NOT a bigger context window (sample Q12) |
| Resume after heavy code changes | Tell it which files changed, or fresh session + injected summary — stale tool results are dangerous |
| Explore two designs from one expensive analysis | `fork_session` per direction. The original is untouched |
| Team's Claude Code behavior differs per developer | Project-level config (`.claude/commands/`, project CLAUDE.md, rules) vs someone's user-level file — version control is the signal |
| Convention spans scattered files | `.claude/rules/` with paths globs is better than per-directory CLAUDE.md files and header inference |
| Escalating to a human | Structured handoff payload (ID, root cause, amounts, attempts, recommendation) — the human cannot see the transcript |

## Terminology discipline

The guide (and therefore the item bank) says **Task tool**. Answer with the guide's vocabulary even though current Claude Code names it Agent. Same principle everywhere. Graders use Exam Guide v1.0 terms. File 08 flags the current-docs deltas so you are never surprised. Do not "correct" the exam.

## Distractor archetypes to expect

- **The prompt patch** for a determinism problem ("add to the system prompt that it must always…"). This lowers the failure rate. It never eliminates the failure. It is wrong when the stem says must/never/compliance.
- **The bigger window** for attention dilution or coverage problems.
- **The out-of-scope distractor:** Bedrock IAM, pricing tiers, and embedding stores sound plausible, but the exam excludes them.
- **The blame-shift:** you fix the subagent when the coordinator's decomposition or the missing context hand-off is the root cause.
- **The transcript assumption:** answers that assume a human or subagent can see conversation history that no one gave it.

## Freshness watchlist (re-verify the week of your exam)

Exact SDK option spellings (`agents`, `fork_session`/`forkSession`, hook callback fields) at code.claude.com/docs/en/agent-sdk — file 08's footer lists them. Five minutes. The concepts will not change. The spellings might.

## The 48-hour plan

1. Re-read 08 end-to-end, then 09 (together = 42% of the form).
2. Re-read the two supplements (04's Domain-2 mechanics, 05's Domain-3 mechanics). These are dense fact sections. Recent reading helps.
3. Go through each of the 6 printed scenarios and predict its failure modes from the reflex table above. This is almost a preview of the exam.
4. Re-do the 12 sample questions without notes. Every one should now feel like a routine procedure.
5. Skip files 06–07 entirely (out of scope) and 01–02 (background).

## Day-of (online proctored)

ID, clean desk, room scan, stable connection. When a scenario block opens, spend 30 seconds to identify which of the 6 published scenarios it is. Your failure-mode map from practice then helps you judge every question in it. Deterministic answers are better than probabilistic answers. Coordinator before workers. Guide terminology over current docs. Answer everything.
