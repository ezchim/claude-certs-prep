# CCAO-F Pro Tips — how to take this exam well

*Companion to domain files 01–07; distilled 2026-08-23 from the official Exam
Guide v1.0 and this pack. Exam craft only — content lives in the domain files.*

**What this exam actually rewards:** *judgment as a professional user*, not
technical knowledge. The heaviest domain by far is Output Evaluation (21%),
and the recurring question underneath most items is: **"what should this
person do before trusting or shipping Claude's output?"** When in doubt,
the answer that adds a verification step, a human gate, or a configuration
fix beats the answer that adds a bigger model or a longer prompt.

---

## The numbers that drive strategy

- **60 items / 120 min → 2 min each.** These stems are shorter than the
 developer exams' — the risk isn't time, it's overthinking. First defensible
 read wins; flag genuine coin-flips and move on.
- **720 scaled, no guessing penalty** — answer everything.
- **Triage by weight:** Evaluation 21% + Workflow Design 16% + Governance 15%
 = 52%. These are all judgment domains — they share one instinct (verify,
 gate, own), so drilling one strengthens all three.
- **Multiple-response is all-or-nothing**; each item states how many to select.

## The four master instincts (most items reduce to one of these)

1. **Verify before it leaves the building.** Citation → open it. Number →
 recompute or spot-check. Compliance/external send → human review first.
 (The guide's own sample: verify the citation before the compliance send.)
2. **Right-size the model.** High-volume, simple, repetitive → lighter/cheaper
 tier. Deep analysis, high-stakes → stronger tier. "Always use the biggest"
 is always wrong (the pack even plants Fable-maximalism as a trap).
3. **Fix the configuration before blaming the model.** Weak output in a shared
 Project → check instructions, stale knowledge files, missing owner, wrong
 placement — *then* consider model/prompt changes (Domain 07's whole logic).
4. **Protect the data.** Regulated/PII content → anonymize before upload, use
 incognito for sensitive one-offs, respect plan-level controls. (Sample
 question: anonymize regulated PII first.)

## If the stem says X → think Y

| Stem signal | Reflex |
| --- | --- |
| "Should they send/publish/submit it?" | Not yet — name the verification step first |
| Claude cited a source / gave a statistic | Check the source exists and says that; citations can be wrong |
| Same task, hundreds of times, simple | Lighter model + a reusable prompt/Project, not Opus-by-default |
| Team keeps getting inconsistent outputs from a shared Project | Knowledge hygiene: duplicate/stale files, no owner, no version note (file 05's mystery-brand-voice case) |
| Where should this knowledge live? | Pair a **placement** (instructions vs knowledge vs Skill vs connector vs Memory) with a **lifecycle** (owner, cadence, archive) |
| "Claude remembers wrong" / personal context issue | Edit/delete the Memory entry; Memory is per-user — it never aligns a team (Project knowledge does) |
| Sensitive matter, shouldn't persist | Incognito chat (excluded from history/memory/search) |
| Data analysis, file outputs, charts | Code execution / file creation — not "ask Claude to imagine a spreadsheet" |
| Live document that keeps changing | Connector, not weekly re-upload; stable policy docs → upload is fine |
| "High-risk use" (legal, medical, employment decisions) | Human accountability stays; Claude drafts/supports, a person decides |

## Distractor archetypes to expect

- **The trust answer:** ship it because it "looks right" or "Claude is usually
 accurate." Never the answer in a 21%-weight evaluation domain.
- **The bigger-model answer** for a workflow/config problem.
- **The permission overshare:** "grant edit to everyone so nobody's blocked" /
 org-wide sharing of restricted knowledge. Least privilege wins here too.
- **The memory confusion:** using personal Memory where shared Project
 knowledge is needed, or vice versa.
- **The plan-fact bluff:** options hinging on invented plan limits. The real
 anchors you know: Free = 5 Projects, sharing = Team/Enterprise, paid RAG
 expands knowledge capacity — anything more exotic, judge by principle.

## Freshness watchlist (re-verify the week of your exam)

Plan caps and feature availability move fastest here: Projects caps, Memory
availability/controls, Research/connector plan gating — 10 minutes on
support.claude.com. The pack's verified facts are dated Aug 2026.

## The 48-hour plan

1. Re-read file 03 in full (21% + it trains the master instinct), then files
 04 and 06.
2. Re-do every file's traps table + the multiple-response pattern banks.
3. Live practice beats re-reading here: evaluate three real Claude outputs
 with a written ship/revise/reject note; build one Project with instructions
 + knowledge + a sharing decision.
4. Timed 60Q simulation; review only what you missed.

## Day-of (online proctored)

ID, clean desk, room scan, stable connection. Two-minute rhythm: read the
question sentence, find who's at risk / what's about to leave the building,
pick the verify/gate/own answer. On review passes, change answers only for
misread facts — your calibrated instinct is the product of the drills.
