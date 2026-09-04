---
title: CCDV-F Pro Tips — how to take this exam well — Simplified Technical English
disclaimer: Original study notes — independent and not official course content
approx_length: STE edition (ASD-STE100) — exam craft
updated: 2026-08-23
---

# CCDV-F Pro Tips — how to take this exam well

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, MCP, prompting, caching, effort, p95, Batch API, fast mode, CLAUDE.md) are exceptions and stay as written.

*Companion to the study chapters. Distilled 2026-08-23 from the official Exam Guide v1.0 and this pack. This file is exam craft, not content. The content lives in chapters 01–06.*

**What this exam actually rewards:** you select the *operationally correct* API mechanism under a named constraint. Almost every stem hides one binding constraint (latency, cost, cache stability, safety, auditability). Find it before you read the options. The right answer serves the constraint. The distractors serve the scenario's tone.

---

## The numbers that drive strategy

- **53 items / 120 min → ~2.26 min each.** Long scenario stems mean the real budget is ~90 seconds of reading + ~45 of deciding. Save time on the short MSO/definition items. Spend that time on Integration scenarios.
- **720 scaled to pass, no penalty for wrong answers** — never leave a blank. Flag and move on. A first-pass sweep of all 53 is better than a perfect first 30.
- Domain weight sets the triage order. Integration 33.1% + MSO 16.8% + Agents 14.7% ≈ 65% of the form. A weak Integration answer costs you three times a weak Evals answer.
- **Multiple-response is all-or-nothing.** Each item states how many to select. Select exactly that many. Select the N options you can defend, not the N that "sound related."

## Stem-reading protocol

1. Read the last sentence first (the actual question).
2. Scan the stem for the **constraint keyword**: "p95", "overnight", "cost per task", "compliance", "multi-tenant", "cache hit rate", "user is waiting".
3. Remove every option that violates the constraint — usually 2 of 4 fail here.
4. Between survivors, prefer the **simplest mechanism that fully solves it**.

## If the stem says X → think Y (the reflexes)

| Stem signal | Reflex |
| --- | --- |
| Overnight / bulk / "by morning" / cost-sensitive | **Batch API** (50% cost, no streaming, async) — official sample Q1 pattern |
| User waiting / interactive / p95 | Streaming, faster tier, smaller context — never Batch |
| "Cache hit rate is low" | Volatile content before the stable prefix. Changed tool defs. Thinking-config flips. Order: tools → system → messages |
| Latency-critical AND needs frontier quality AND budget allows | **Fast mode** (Opus 5/4.8, `speed:"fast"`, premium price, own rate limit, not Batch/Bedrock/Vertex) |
| "Reuse across teams/apps" for a tool/data source | **MCP server** (sample Q3 pattern) — not copy-pasted client code |
| Untrusted content (web, docs, user uploads) meets tools | Injection isolation: delimiters, least-privilege tools, treat retrieved text as data (sample Q2 pattern) |
| "Which model should they use" | Smallest model that passes their evals. Escalate on evidence, never by default |
| "Identical outputs" / determinism | Trap — sampling params are gone on current models. They never guaranteed determinism. Answer = schemas + validation |
| Team behavior must be *guaranteed* in Claude Code | Settings/permissions/hooks (enforced) — CLAUDE.md only *advises* |
| Version surprises in prod | Pin model IDs. Dateless IDs are pinned snapshots, not auto-upgrading |

## Distractor archetypes to expect

- **The oversized fix:** "use the largest model / larger context window" for a problem that is structural (decomposition, caching, retrieval). Weight/size never fixes structure.
- **The impossible combo:** Batch + streaming. Cache pre-warm inside Batch. Temperature tuning on current tiers. Remove these at once. These are checkable facts, and chapters 01/03 list them.
- **The tone-only answer:** restates the goal ("improve the prompt to be more reliable") without a mechanism. If you cannot name the API feature it uses, it is not the answer.
- **The over-engineered agent:** an autonomous agent where a 3-step workflow with validators meets the stated audit requirement.

## Freshness watchlist (re-verify the week of your exam)

Check fast mode status/models. Check the current model selection and adaptive-thinking support (chapter 01 §14 card is dated). Check Claude Code default effort. Check PDF/vision caps. Ten minutes on the official docs the night before covers all of it.

## The 48-hour plan

1. Re-read chapter 03's Integration catalog + chapter 01's decision trees (the 65% core), then chapter 06's framework/hosting vocabulary card.
2. Re-do every chapter's "exam traps" table and the two "if X think Y" tables.
3. One timed 53Q/120min simulation. Score by domain. Re-read only the weak domain's chapter sections.
4. Skim README's domain-weight table one last time. Mental triage order is better than any last fact you memorize.

## Day-of (online proctored)

Government ID ready, clean desk, room scan, stable connection, external monitor off. Check in early — Pearson VUE lets you start up to 30 min ahead. When you return to flagged items, trust your first constraint-based read unless you find a fact you misread. Preference-based answer changes lose points.
