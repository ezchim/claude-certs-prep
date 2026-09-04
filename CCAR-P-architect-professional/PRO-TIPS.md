# CCAR-P Pro Tips — how to take this exam well

*Companion to files 01–05; distilled 2026-08-23 from the official Exam Guide
v1.0 and this pack. Exam craft only.*

**What this exam actually rewards:** disciplined *elimination under multiple
constraints*. Professional-level stems stack 3–5 requirements (latency +
residency + audit + cost + an org quirk); every option that violates even one
named constraint is dead, and what survives is usually the least exciting
architecture on the list. The pack's leftmost-pattern rule is the exam in one
sentence: **prefer the simplest pattern that meets the requirement, and move
right only when evidence says the left fails.**

---

## The numbers that drive strategy

- **63 items / 120 min → ~1.9 min each** — the tightest pacing of the four
 exams, against the longest stems. Non-negotiable technique: read the
 question sentence first, then harvest constraints from the stem as a list.
 If you're 30 seconds in and still "absorbing the scenario," you're reading
 it wrong.
- **Triage:** Integration 19% + Solution Design 17% + Evaluation 16% = 52%.
 But the soft domains (Governance 14 + Stakeholder 14 + Enablement 7 = 35%)
 are cheap points with one shared template — **mechanism + metric + owner**.
 An answer that names all three beats an answer with better adjectives.
- **720 scaled, no guessing penalty, multiple-response all-or-nothing** (and
 this exam loves Select-TWO with three defensible-sounding options — the
 discriminator is almost always a violated constraint, not quality).

## The constraint-elimination drill (use it on every long stem)

1. Question sentence first.
2. List the constraints: SLA numbers, residency/compliance words, volumes,
 budget signals, "auditors", "regulated", "multi-tenant", who owns what.
3. Cross off options violating any single constraint.
4. Among survivors: leftmost pattern, smallest model that passes evals,
 least privilege, fewest moving parts.

## If the stem says X → think Y

| Stem signal | Reflex |
| --- | --- |
| "Auditors need evidence per step" | Workflow with stage logs + validators — not an autonomous agent |
| "Steps unknown / open-ended investigation" | Agent + tool allowlist + budgets + HITL on writes |
| Agent has too many tools / does everything | Least privilege: remove tools first (official sample Q1) |
| Cache hit rate low / cost spike on stable prompts | Stable prefix first, volatile last; stop reshuffling tool defs (sample Q2) |
| RAG answers wrong despite good model | Diagnose retrieval before touching the model (sample Q3); citations or refuse |
| Choosing Direct API vs Bedrock vs Vertex vs Foundry | Constraint elimination on residency/procurement/identity — never brand vibes (file 02 D5) |
| "How do we know it's good?" | Gold + adversarial + regression evals; code judges before model judges; freeze thresholds **before** looking at experiment data (file 02 M7) |
| Refunds/payments/writes by an agent | AuthZ in the tool server + schema validation in code + caps enforced by code, not model judgment |
| Multi-agent proposal | Real role/permission boundaries or it's theater — one workflow may do |
| "Stakeholders want it fully autonomous" | Value now + unlock later + risk named + written alignment; HITL stays on high impact |
| Handoff/hypercare/lifecycle | Named DRI + end date + runbook + ADR log — ownership answers score |
| Model upgrade in prod | Pin → shadow/replay → canary + kill switch → flip; prompt and model are one release artifact |
| Business value pillars | The guide's five: efficiency, transformation, **productivity**, cost, performance SLAs |

## Distractor archetypes to expect

- **The impressive architecture:** multi-agent, biggest model, every tool —
 violating the simplest-that-works rule. Sophistication is the bait.
- **The single-constraint answer:** perfect on latency, silently violates
 residency (or vice versa). This is *the* Select-TWO killer — check every
 survivor against every constraint.
- **The prompt-as-control:** governance "solved" by prompt text. Prompts are
 never the whole control story — pair with authZ, HITL, monitoring, evidence.
- **The trust-the-model:** letting Claude decide refund eligibility, tenancy,
 or thresholds that code must enforce.
- **The eval afterthought:** shipping on demo vibes; peeking at data before
 fixing thresholds; crowning an A/B winner on 20 chats.

## Freshness watchlist (re-verify the week of your exam)

This pack deliberately teaches decision logic over SKUs, so little rots. Worth
10 minutes anyway: current model tiers + adaptive thinking/effort (thinking
budgets and sampling params are legacy — the pack is already corrected),
platform availability differences (Bedrock/Vertex/Foundry), Claude Code
org-managed settings behavior.

## The 48-hour plan

1. Re-read file 02 Parts K–M (Integration + Evals deep dives — 35% of the
 form) and file 01's Part J scenarios with the scoring keys covered.
2. Re-do files 03–05's traps + the review-routing matrix (03 D4) — the soft
 35% compresses into mechanism/metric/owner + named-DRI answers.
3. Write two ADRs cold from the scenario banks (pattern, model, rejected
 alternatives) — the exam's long stems are ADRs in disguise.
4. Timed 63Q/120min set at full pace; practice the constraint-list drill until
 it's automatic, not the content.

## Day-of (online proctored)

ID, clean desk, room scan, stable connection. Pace ruthlessly: ~1.9 min/item
means any item at 3+ minutes gets your best surviving option and a flag. The
soft-domain items are your recovery time — bank it. On the flagged-item pass,
re-check constraints, not preferences. Answer all 63.
