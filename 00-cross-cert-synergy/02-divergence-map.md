# Divergence Map — where shared knowledge must NOT cross exams

*The anti-contamination file. Read the relevant exam's fences for 20 minutes
each exam morning. When this file and instinct disagree, this file wins —
each exam is graded against its own guide's scope and vocabulary.*

---

## 1. Scope fences (the same topic, in one exam and out of another)

| Topic | CCAO-F | CCAR-F | CCDV-F | CCAR-P |
| --- | --- | --- | --- | --- |
| Prompt-caching mechanics | absent | **explicitly OUT of scope** | **core** (Integration 33.1%) | **core** (sample Q2) |
| Cloud providers (Bedrock/Vertex/Foundry) | absent | **explicitly OUT** | in (deployment surfaces, hosting) | **in** (CSP route governance, file 02 D5) |
| Streaming / SSE vs websockets | absent | **explicitly OUT** | in (module 06) | light (latency lever) |
| Pricing / rate limits | absent | **explicitly OUT** | concepts (cost levers, batch 50%) | cost envelopes, no price sheets |
| Agent SDK specifics (Task tool, AgentDefinition, hooks, fork_session) | absent | **CCAR-F ONLY — 27% domain** | generic agent loops only | pattern-level only |
| claude.ai product (Projects, Artifacts, Memory, Research, incognito) | **core — the whole exam** | absent | absent | absent |
| Fast mode | absent | out (pricing-adjacent) | **named in the guide** (MSO) | not named |
| Embeddings / vector DBs | absent | **explicitly OUT** | light (RAG context) | in (RAG architecture) |
| Vision / multimodal | absent | **explicitly OUT** | in (API mechanics) | light (multimodal design) |
| GTM / stakeholder / lifecycle | absent | absent | absent | **CCAR-P ONLY — ~21%** |
| 4D AI Fluency framework | **in** (Delegation, Description, Discernment, Diligence) | absent | absent | absent |
| Eval harness engineering | judgment-level only | calibration/sampling flavor | light (2.6%) | **deep — 16%** |

**Usage rule:** an answer option resting on a topic marked OUT for that day's
exam is a distractor by definition — kill it without deliberation (this is a
weapon on CCAR-F especially).

## 2. Terminology deltas (graded against each guide's own words)

| Concept | Say this per exam |
| --- | --- |
| Subagent spawning tool | CCAR-F: **"Task" tool** (the guide's vocabulary — know the current-docs rename to Agent, answer with Task) |
| Value pillars | CCAR-P: **efficiency, transformation, productivity, cost, performance SLAs** — the guide's exact five |
| Model ladder | CCAO-F: **Haiku / Sonnet / Opus** — the guide names three tiers only; Fable appears in options as an over-buy trap |
| Prompting technique names | CCDV-F: zero-shot / single-shot / multi-shot defined exactly; CCAO-F uses plain-language framing |
| "Memory" | CCAO-F: the claude.ai Memory feature (entries, controls, incognito). CCAR-F/CCDV: Claude Code auto memory / `/memory`. Different products — never mix |

## 3. Same stem, different correct answer (the dangerous ones)

**"The agent skipped a required compliance step."**
- CCAR-F → programmatic prerequisite or PreToolUse-style interception hook.
- CCAR-P → workflow gate + authZ enforced in the tool server (architecture altitude).
- CCAO-F → human approval gate before the action leaves the building.
Same doctrine (D2); the graded answer names *that exam's* mechanism.

**"Improve extraction reliability."**
- CCDV-F → JSON schema + strict validation + few-shot anchors.
- CCAR-F → retry-with-error-feedback loops, nullable fields to prevent fabrication.
- CCAR-P → validators in code before side effects, inside a workflow stage.
(And never "lower temperature" — gone on current tiers, all packs corrected.)

**"Which model should they use?"**
- CCAO-F → task-type picker (high-volume simple → Haiku-class; deep → Opus-class).
- CCDV-F → smallest that passes evals + effort dial; fast mode only if latency-critical AND premium justified.
- CCAR-P → eval-proven promotion with router/canary architecture around it.

**"Claude keeps getting things wrong for the team."**
- CCAO-F → Project knowledge hygiene / Memory entry cleanup / configuration first.
- CCDV-F / CCAR-F → project-level vs user-level Claude Code config; enforce vs advise.
- CCAR-P → eval-diagnose first (retrieval before model — sample Q3), then config.

**"Handle this long/complex conversation context."**
- CCAO-F → start a fresh chat with a summary; Project knowledge for durable facts.
- CCAR-F → scratchpad files, case-facts blocks, fresh-session-with-summary vs --resume.
- CCDV-F / CCAR-P → compaction/context editing features + cache-stable design.

## 4. Per-exam unique mass (what ONLY that pack teaches — protect time for it)

| Exam | Unique, non-transferable content | Where |
| --- | --- | --- |
| CCAO-F | Product feature judgment (Projects/Artifacts/Memory/Research/incognito/code execution), plan facts, 4D fluency, high-risk-use governance | whole pack, esp. 03/04/06 |
| CCAR-F | Agent SDK mechanics (stop_reason loop anti-patterns, AgentDefinition, allowedTools, hooks, fork_session), Claude Code config mechanics (@import, rules globs, SKILL.md keys, --output-format json), the **6 published scenarios** | 08, 09, 04/05 supplements |
| CCDV-F | Integration API detail (streaming, caching mechanics, batch mechanics, schemas, pinning), fast mode, LLM fundamentals, frameworks (Strands/LangGraph/PydanticAI), SDLC/websockets | 03, 01, 06 |
| CCAR-P | Soft domains (~35%): stakeholder/GTM, lifecycle/hypercare/DRI, enablement; CSP route governance; judge calibration; review-routing matrix | 02 D5/M7, 03 D4, 04, 05 |

## 5. Morning-of checklists (20 minutes, that exam's column only)

- **Before CCAO-F:** fences (no API content will be graded); Haiku/Sonnet/Opus
 framing; verify → gate → own; Memory = claude.ai feature today.
- **Before CCAR-F:** the OUT list is your distractor-killer; "Task" vocabulary;
 deterministic > probabilistic; blame the coordinator; walk the 6 scenarios.
- **Before CCDV-F:** caching/batch/fast-mode facts are in scope; the
 constraint keyword decides; check the freshness watchlist once.
- **Before CCAR-P:** everything technical is in scope PLUS the
 soft 35% — mechanism + metric + owner; leftmost pattern; official pillar
 wording; constraint-elimination drill at 1.9 min/item.
