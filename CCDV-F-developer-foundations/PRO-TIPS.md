# CCDV-F Pro Tips — how to take this exam well

*Companion to the study chapters; distilled 2026-08-23 from the official Exam
Guide v1.0 and this pack. This file is exam craft, not content — the content
lives in chapters 01–06.*

**What this exam actually rewards:** picking the *operationally correct* API
mechanism under a named constraint. Almost every stem hides one binding
constraint (latency, cost, cache stability, safety, auditability). Find it
before reading the options — the right answer serves the constraint; the
distractors serve the scenario's vibe.

---

## The numbers that drive strategy

- **53 items / 120 min → ~2.26 min each.** Long scenario stems mean the real
 budget is ~90 seconds of reading + ~45 of deciding. Bank time on the short
 MSO/definition items to spend on Integration scenarios.
- **720 scaled to pass, no penalty for wrong answers** — never leave a blank.
 Flag and move on; a first-pass sweep of all 53 beats perfecting the first 30.
- **Domain weight is triage law:** Integration 33.1% + MSO 16.8% + Agents
 14.7% ≈ 65% of the form. A shaky Integration answer costs you three times a
 shaky Evals answer.
- **Multiple-response is all-or-nothing.** Each item states how many to select
 — select exactly that many. Pick the N options you can *defend*, not the N
 that "sound related."

## Stem-reading protocol

1. Read the last sentence first (the actual question).
2. Scan the stem for the **constraint keyword**: "p95", "overnight", "cost per
 task", "compliance", "multi-tenant", "cache hit rate", "user is waiting".
3. Kill every option that violates the constraint — usually 2 of 4 die here.
4. Between survivors, prefer the **simplest mechanism that fully solves it**.

## If the stem says X → think Y (the reflexes)

| Stem signal | Reflex |
| --- | --- |
| Overnight / bulk / "by morning" / cost-sensitive | **Batch API** (50% cost, no streaming, async) — official sample Q1 pattern |
| User waiting / interactive / p95 | Streaming, faster tier, smaller context — never Batch |
| "Cache hit rate is low" | Volatile content before the stable prefix; changed tool defs; thinking-config flips. Order: tools → system → messages |
| Latency-critical AND needs frontier quality AND budget allows | **Fast mode** (Opus 5/4.8, `speed:"fast"`, premium price, own rate limit, not Batch/Bedrock/Vertex) |
| "Reuse across teams/apps" for a tool/data source | **MCP server** (sample Q3 pattern) — not copy-pasted client code |
| Untrusted content (web, docs, user uploads) meets tools | Injection isolation: delimiters, least-privilege tools, treat retrieved text as data (sample Q2 pattern) |
| "Which model should they use" | Smallest model that passes their evals; escalate on evidence, never by default |
| "Identical outputs" / determinism | Trap — sampling params are gone on current models and never guaranteed determinism anyway; answer = schemas + validation |
| Team behavior must be *guaranteed* in Claude Code | Settings/permissions/hooks (enforced) — CLAUDE.md only *advises* |
| Version surprises in prod | Pin model IDs; dateless IDs are pinned snapshots, not auto-upgrading |

## Distractor archetypes to expect

- **The bigger hammer:** "use the largest model / larger context window" for a
 problem that is structural (decomposition, caching, retrieval). Weight/size
 never fixes structure.
- **The impossible combo:** Batch + streaming; cache pre-warm inside Batch;
 temperature tuning on current tiers. Kill on sight — these are checkable
 facts, and chapters 01/03 list them.
- **The vibe answer:** restates the goal ("improve the prompt to be more
 reliable") without a mechanism. If you can't name the API feature it uses,
 it's not the answer.
- **The over-engineered agent:** an autonomous agent where a 3-step workflow
 with validators meets the stated audit requirement.

## Freshness watchlist (re-verify the week of your exam)

Fast mode status/models · current model lineup + adaptive-thinking support
(chapter 01 §14 card is dated) · Claude Code default effort · PDF/vision caps.
Ten minutes on the official docs the night before covers all of it.

## The 48-hour plan

1. Re-read chapter 03's Integration catalog + chapter 01's decision trees (the
 65% core), then chapter 06's framework/hosting vocabulary card.
2. Re-do every chapter's "exam traps" table and the two "if X think Y" tables.
3. One timed 53Q/120min simulation; score by domain; re-read only the weak
 domain's chapter sections.
4. Skim README's domain-weight table one last time — triage order in your head
 beats any last fact memorized.

## Day-of (online proctored)

Government ID ready, clean desk, room scan, stable connection, external
monitor off. Check in early — Pearson VUE lets you start up to 30 min ahead.
When flagged items come back around, trust your first constraint-based read
unless you find a fact you misread — vibes-based answer changes lose points.
