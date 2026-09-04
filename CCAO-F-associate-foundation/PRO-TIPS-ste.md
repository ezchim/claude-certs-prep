---
title: CCAO-F Pro Tips — how to take this exam well — Simplified Technical English
disclaimer: Original study notes — independent and not official course content
approx_length: STE edition (ASD-STE100) — exam craft
updated: 2026-08-23
---

# CCAO-F Pro Tips — how to take this exam well

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Haiku, Sonnet, Opus, Fable, Projects, Skills, Connectors, Memory, RAG, PII) are exceptions and stay as written.

*Companion to domain files 01–07. Distilled 2026-08-23 from the official Exam Guide v1.0 and this pack. Exam craft only. Content lives in the domain files.*

**What this exam actually rewards:** *judgment as a professional user*, not technical knowledge. The heaviest domain by far is Output Evaluation (21%). The recurring question under most items is: **"what should this person do before they trust or ship Claude's output?"**
When you have doubt, select the answer that adds a verification step, a human gate, or a configuration fix. That answer beats the answer that adds a bigger model or a longer prompt.

---

## The numbers that drive strategy

- **60 items / 120 min → 2 min each.** These stems are shorter than the developer exams'. The risk is not time. The risk is that you think too long. The first defensible read is enough. Flag genuine 50/50 cases and move on.
- **720 scaled, no guessing penalty** — answer everything.
- **Triage by weight:** Evaluation 21% + Workflow Design 16% + Governance 15% = 52%. These are all judgment domains. They share one instinct (verify, gate, own), so a drill of one strengthens all three.
- Multiple-response items score only when you select every correct option. Each item states how many to select.

## The four master instincts (most items reduce to one of these)

1. **Verify before the output goes out.** Citation → open it. Number → recompute or spot-check. Compliance/external send → human review first. (The guide's own sample: verify the citation before the compliance send.)
2. **Right-size the model.** High-volume, simple, repetitive → lighter/cheaper tier. Deep analysis, high-stakes → stronger tier. "Always use the biggest" is always wrong (the pack also sets "always use the model with the most capability" as a trap).
3. **Fix the configuration before you blame the model.** Weak output in a shared Project: check instructions, stale knowledge files, missing owner, and wrong placement. *Then* consider model/prompt changes (Domain 07's whole logic).
4. **Protect the data.** Regulated/PII content → anonymize before upload, use incognito for sensitive single-use questions, respect plan-level controls. (Sample question: anonymize regulated PII first.)

## If the stem says X → think Y

| Stem signal | Reflex |
| --- | --- |
| "Should they send/publish/submit it?" | Not yet — name the verification step first |
| Claude cited a source / gave a statistic | Check the source exists and says that. Citations can be wrong |
| Same task, hundreds of times, simple | Lighter model + a reusable prompt/Project, not Opus-by-default |
| Team keeps getting inconsistent outputs from a shared Project | Knowledge hygiene: duplicate/stale files, no owner, no version note (file 05's mystery-brand-voice case) |
| Where should this knowledge live? | Pair a **placement** (instructions vs knowledge vs Skill vs connector vs Memory) with a **lifecycle** (owner, cadence, archive) |
| "Claude remembers wrong" / personal context issue | Edit/delete the Memory entry. Memory is per-user — it never aligns a team (Project knowledge does) |
| Sensitive matter, must not persist | Incognito chat (excluded from history/memory/search) |
| Data analysis, file outputs, charts | Code execution / file creation — not "ask Claude to imagine a spreadsheet" |
| Live document that keeps changing | Connector, not weekly re-upload. Stable policy docs → upload is fine |
| "High-risk use" (legal, medical, employment decisions) | Human accountability stays. Claude drafts/supports, a person decides |

## Distractor archetypes to expect

- **The trust answer:** ship it because it "looks right" or "Claude is usually accurate." Never the answer in a 21%-weight evaluation domain.
- **The bigger-model answer** for a workflow/config problem.
- **The permission overshare:** "grant edit to everyone so nobody's blocked" / org-wide sharing of restricted knowledge. Least privilege is correct here too.
- **The memory confusion:** you use personal Memory where shared Project knowledge is needed, or the reverse.
- **The plan-fact error:** options that depend on invented plan limits. The real anchors you know: Free = 5 Projects, sharing = Team/Enterprise, paid RAG expands knowledge capacity. For anything more unusual, judge by principle.

## Freshness watchlist (re-verify the week of your exam)

Plan caps and feature availability change fastest here. Check Projects caps, Memory availability/controls, and Research/connector plan gating. Use 10 minutes on support.claude.com. The pack's verified facts are dated Aug 2026.

## The 48-hour plan

1. Re-read file 03 in full (21% + it trains the master instinct), then files 04 and 06.
2. Re-do every file's traps table + the multiple-response pattern banks.
3. Live practice is better than another read here. Evaluate three real Claude outputs with a written ship/revise/reject note. Build one Project with instructions + knowledge + a sharing decision.
4. Timed 60Q simulation. Review only what you missed.

## Day-of (online proctored)

ID, clean desk, room scan, stable connection. Two-minute rhythm: read the question sentence. Find who is at risk / what is about to leave the organization. Select the verify/gate/own answer. On review passes, change answers only for misread facts. Your calibrated instinct comes from the drills.
