# Claude Certified Architect – Professional (CCAR-P) Study Pack

**Exam code:** CCAR-P 
**Credential:** Claude Certified Architect – Professional 
**Pack type:** Original exam-prep notes (not official course content) 
**Depth:** **Deepened for PRIMARY study** — architecture decision scenarios, trade-off tables, failure-mode analysis, production checklists, and expanded Q&A (25–40+ per domain file). Intended to serve as a main study source alongside public Anthropic docs and the official exam guide (when you have access). 
**Audience:** Solution architects designing production Claude systems 
**User domains in this pack:** **Five** condensed files (maps onto official seven) 
**Offline reading tip:** Each file is plain Markdown — convert with any MD→EPUB tool if you prefer e-reader formatting; keep headings H1–H3 for navigation.

---

## Public availability & partner gating (read this first)

| Resource | Public? | Notes |
| --- | --- | --- |
| Exam blueprint domains & weights | **Official** — Exam Guide v1.0 (effective July 2026); download from the official Pearson VUE Anthropic program page | All weights and mechanics below are taken directly from it, cross-checked (2026-08-23) against the public Pearson VUE Anthropic program page. |
| Exam registration | Official portal | Registration and proctoring run through the official exam portal (Pearson VUE delivery); check current requirements before booking. |
| Official gated prep course | Not used | This pack is original and grounded in public sources only. |
| Public Anthropic docs (platform.claude.com, model cards, MCP, tool use, safety research) | **Public** | Primary grounding for all technical notes in this pack. |
| claude.ai / Claude API / Claude Code docs | **Public** (product features may require accounts) | Use for hands-on practice. |
| GTM / partner sales playbooks | **Often partner-only** | File `04` covers stakeholder discovery, proposal framing, lifecycle handoff, and value communication in **original** language only—no confidential partner scripts. |

**Disclaimer:** This pack is an independent study aid. It is not affiliated with, endorsed by, or derived from official course materials. Exam mechanics and domain weights can change; verify against the official guide before exam day.

---

## Exam mechanics (official Exam Guide v1.0, July 2026)

| Item | Value |
| --- | --- |
| Questions | **63** items |
| Time | **120 minutes** (~1.9 min/question average) |
| Pass score | **720** on a **100–1,000** scaled score |
| Format | Multiple choice + multiple-response (“Select TWO/THREE”); many long multi-constraint scenarios |
| Delivery | Pearson VUE (online proctored or test center) |
| Validity | **12 months**; free non-proctored on-time renewal |
| Fee | **$175 USD** per attempt |
| Retakes | 14/30/90-day waits; max 4 attempts per rolling 12 months |
| Prerequisites | None mandatory; **production Claude architecture experience strongly recommended**; CCAR-F helpful but not required |

**Scoring note:** Scaled scoring means 720 is **not** a fixed % correct. Multiple-response items usually need **all** correct selections—partial credit should not be assumed.

---

## User’s 5 condensed domains ↔ official 7 domains

| # | User condensed domain | File | Official domains covered |
| --- | --- | --- | --- |
| 1 | Claude Platform & Solution Design | `01-platform-solution-design.md` | **Solution Design & Architecture (17%)** · **Claude Models, Prompting & Context Engineering (13%)** |
| 2 | Enterprise Integration & Production | `02-enterprise-integration-production.md` | **Integration (19%)** · **Evaluation, Testing & Optimization (16%)** |
| 3 | Responsible AI, Safety & Risk for Architects | `03-responsible-ai-safety-risk.md` | **Governance, Safety & Risk Management (14%)** |
| 4 | Stakeholder Engagement, Lifecycle & GTM | `04-stakeholder-lifecycle-gtm.md` | **Stakeholder Communication & Lifecycle Management (14%)** · GTM-adjacent communication (partner limits flagged) |
| 5 | Team Enablement & Operational Productivity | `05-team-enablement-operational-productivity.md` | **Developer Productivity & Operational Enablement (7%)** |

### Official weight map (study time budgeting)

| Official domain | Weight | ~Q (of 63) | Primary file |
| --- | --- | --- | --- |
| Integration | 19% | ~12 | 02 |
| Solution Design & Architecture | 17% | ~11 | 01 |
| Evaluation, Testing & Optimization | 16% | ~10 | 02 |
| Governance, Safety & Risk Management | 14% | ~9 | 03 |
| Stakeholder Communication & Lifecycle Management | 14% | ~9 | 04 |
| Claude Models, Prompting & Context Engineering | 13% | ~8 | 01 |
| Developer Productivity & Operational Enablement | 7% | ~4 | **05** |

**Study implication:** Integration + Solution Design + Evaluation ≈ **52%**. Governance + Stakeholders + Enablement ≈ **35%**—do not skip files 03–05.

---

## File index

1. [README.md](./README.md) — this index 
2. [01-platform-solution-design.md](./01-platform-solution-design.md) — models, prompting, context, solution architecture 
3. [02-enterprise-integration-production.md](./02-enterprise-integration-production.md) — MCP/tools/agents, RAG, evals, observability 
4. [03-responsible-ai-safety-risk.md](./03-responsible-ai-safety-risk.md) — guardrails, HITL, compliance, risk 
5. [04-stakeholder-lifecycle-gtm.md](./04-stakeholder-lifecycle-gtm.md) — stakeholders, lifecycle, GTM-adjacent communication 
6. [05-team-enablement-operational-productivity.md](./05-team-enablement-operational-productivity.md) — Claude Code rollout, CLAUDE.md/Skills/hooks, training, ops enablement 
7. [PRO-TIPS.md](./PRO-TIPS.md) — exam-craft companion: pacing math, the constraint-elimination drill, reflex table, distractor archetypes (added 2026-08-23) 

Each domain file includes: disclaimer · overview · key map · deep notes · architecture decision scenarios · trade-off tables · failure-mode analysis · decision trees · exam traps · **25–40+ Q&A** · production checklists · glossary. Deepened for primary study use.

### P0 gap-closure additions (this pack revision)

Original notes only (no official lesson prose):

| Addition | Where |
| --- | --- |
| **CSP / delivery-route governance** — public-safe decision table (Direct Anthropic API vs Amazon Bedrock vs Google Vertex AI vs Azure/Microsoft Foundry), identity/auth, residency, logging/retention & ZDR-as-verify themes, procurement/BAA/FedRAMP themes, model-ID differences, constraint-elimination vs vibes, + Q&A | `02` Part **D5** (pointer from `01` deployment surfaces) |
| **Model judges vs code judges + calibration** — when deterministic assertions win, when rubric judges are needed, human-gold calibration, thresholds/rollback **before** peeking at experiment data, mini worksheet + Q&A | `02` Part **M7** (pointer from Part E3) |
| **Review-routing matrix** (P1) — reversibility × cost × confidence → auto / sample / always-human | `03` Part **D4** |

---

## Recommended study path (4–6 weeks) — primary-source mode

1. **Week 1:** File 01 — models, prompting, context, architecture patterns; complete Part J scenarios. 
2. **Week 2:** File 02 Integration half — RAG, MCP, authZ, latency, capability bloat (heaviest weight). 
3. **Week 3:** File 02 Evaluation half + File 03 safety/governance; run failure-mode tables. 
4. **Week 4:** File 04 stakeholders/lifecycle/change/value (flag partner-only GTM limits) + File 05 enablement; mixed scenarios. 
5. **Weeks 5–6:** Timed 63Q/120m sets; weakness loops; write ADRs from the scenario banks cold.

Hands-on: one RAG path, one tool/MCP path, one eval harness, one HITL gate, one Claude Code org-settings dry-run (even if only in docs review).

**Primary-study tip:** For each official domain, be able to teach aloud: (1) decision tree, (2) one trade-off table, (3) top three failure modes, (4) production checklist items that would block GA.

---

## Public sources to ground further reading

- Anthropic Claude models overview & pricing (platform docs) 
- Messages API, tool use, server vs client tools 
- Model Context Protocol (MCP) introduction & server guides 
- Prompt caching, context windows, thinking / adaptive thinking 
- Search results / citations for RAG attribution 
- Claude Code docs (settings hierarchy, MCP, plugins—public product docs) 
- Anthropic research: Constitutional AI / Claude’s Constitution 
- Cloud deployments: Amazon Bedrock, Vertex AI, Microsoft Foundry (as applicable)

---

## EPUB conversion hint

```bash
pandoc README.md 01-*.md 02-*.md 03-*.md 04-*.md 05-*.md -o CCAR-P-study.epub \
 --metadata title="CCAR-P Study Pack" --toc --toc-depth=2
```

---

*Pack deepened as a **primary study source** across **five** user domain files (original notes only), including **P0 gap closures** (CSP routes, judge calibration, review-routing). Keep the 5↔7 domain mapping above. Verify exam mechanics against the current official CCAR-P Exam Guide before sitting the exam.*
