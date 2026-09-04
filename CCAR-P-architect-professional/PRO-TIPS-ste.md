---
title: CCAR-P Pro Tips — how to take this exam well — Simplified Technical English
disclaimer: Original study notes — independent and not official course content
approx_length: STE edition (ASD-STE100) — exam craft
updated: 2026-08-23
---

# CCAR-P Pro Tips — how to take this exam well

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, MCP, prompting, caching, effort, RAG, HITL, Bedrock, Vertex, Foundry, eval, SLA) are exceptions and stay as written.

*Companion to files 01–05. Distilled 2026-08-23 from the official Exam Guide v1.0 and this pack. Exam craft only.*

**What this exam actually rewards:** disciplined *elimination under multiple constraints*. Professional-level stems stack 3–5 requirements (latency + residency + audit + cost + an organization exception). Every option that violates even one named constraint is wrong. The option that survives is usually the simplest architecture on the list. The pack's leftmost-pattern rule summarizes the exam. **Prefer the simplest pattern that meets the requirement. Move right only when evidence says the left fails.**

---

## The numbers that drive strategy

- **63 items / 120 min → ~1.9 min each** — the shortest time per item of the four exams, against the longest stems. Required technique: read the question sentence first. Then collect constraints from the stem as a list. If 30 seconds pass and you still only read the story, you read the question incorrectly.
- **Triage:** Integration 19% + Solution Design 17% + Evaluation 16% = 52%. The communication domains (Governance 14 + Stakeholder 14 + Enablement 7 = 35%) are easier points with one shared template — **mechanism + metric + owner**. An answer that names all three beats an answer with better adjectives.
- **720 scaled, no guessing penalty, multiple-response all-or-nothing.** This exam often uses Select-TWO with three options that sound defensible. The difference is almost always a violated constraint, not quality.

## The constraint-elimination drill (use it on every long stem)

1. Question sentence first.
2. List the constraints: SLA numbers, residency/compliance words, volumes, budget signals, "auditors", "regulated", "multi-tenant", who owns what.
3. Cross off options that violate any single constraint.
4. Among survivors: leftmost pattern, smallest model that passes evals, least privilege, fewest components.

## If the stem says X → think Y

| Stem signal | Reflex |
| --- | --- |
| "Auditors need evidence per step" | Workflow with stage logs + validators — not an autonomous agent |
| "Steps unknown / open-ended investigation" | Agent + tool allowlist + budgets + HITL on writes |
| Agent has too many tools / does everything | Least privilege: remove tools first (official sample Q1) |
| Cache hit rate low / cost spike on stable prompts | Stable prefix first, volatile last. Stop a change of tool definition order (sample Q2) |
| RAG answers wrong despite good model | Diagnose retrieval before you touch the model (sample Q3). Citations or refuse |
| Choosing Direct API vs Bedrock vs Vertex vs Foundry | Constraint elimination on residency/procurement/identity — never brand preference (file 02 D5) |
| "How do we know it is good?" | Gold + adversarial + regression evals. Code judges before model judges. Freeze thresholds **before** you look at experiment data (file 02 M7) |
| Refunds/payments/writes by an agent | AuthZ in the tool server + schema validation in code + caps enforced by code, not model judgment |
| Multi-agent proposal | Real role and permission boundaries, or the design only looks like multi-agent work — one workflow can be enough |
| "Stakeholders want it fully autonomous" | Value now + unlock later + risk named + written alignment. HITL stays on high impact |
| Handoff/hypercare/lifecycle | Named DRI + end date + runbook + ADR log — answers about ownership get points |
| Model upgrade in prod | Pin → shadow/replay → canary + kill switch → flip. Prompt and model are one release artifact |
| Business value pillars | The guide's five: efficiency, transformation, **productivity**, cost, performance SLAs |

## Distractor archetypes to expect

- **The impressive architecture:** multi-agent, model with the most capability, every tool — this violates the simplest-that-works rule. Complexity is the trap.
- **The single-constraint answer:** perfect on latency, silently violates residency (or the reverse). This is the main Select-TWO trap. Check every survivor against every constraint.
- **The prompt-as-control:** governance "solved" by prompt text. Prompts are never the complete control answer. Pair them with authZ, HITL, monitoring, and evidence.
- **The trust-the-model:** you let Claude decide refund eligibility, tenancy, or thresholds that code must enforce.
- **The eval afterthought:** you ship after a demo only. You look at experiment data before you freeze thresholds. You select an A/B winner on 20 chats.

## Freshness watchlist (re-verify the week of your exam)

This pack teaches decision logic over SKUs on purpose. Little becomes outdated. Still use 10 minutes. Check current model tiers and adaptive thinking/effort. Thinking budgets and sampling params are legacy. The pack already has the correction. Also check platform availability differences (Bedrock/Vertex/Foundry). Check Claude Code org-managed settings behavior.

## The 48-hour plan

1. Re-read file 02 Parts K–M (Integration + Evals deep dives — 35% of the form). Also re-read file 01's Part J scenarios with the scoring keys covered.
2. Re-do files 03–05's traps + the review-routing matrix (03 D4). The communication-domain 35% compresses into mechanism, metric, owner, and named-DRI answers.
3. Write two ADRs without notes from the scenario banks (pattern, model, rejected alternatives). The long stems on the exam are ADRs in another form.
4. Timed 63Q/120min set at full pace. Practice the constraint-list drill until the drill is automatic. The content is not the main drill.

## Day-of (online proctored)

ID, clean desk, room scan, stable connection. Pace without delay: ~1.9 min/item. Any item at 3+ minutes gets your best surviving option and a flag. The soft-domain items are your recovery time. Keep that extra time. On the flagged-item pass, re-check constraints, not preferences. Answer all 63.
