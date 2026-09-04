---
title: Agentic Architecture & Orchestration
exam: Claude Certified Architect – Foundations (CCAR-F), Domain 1 (27% — largest domain)
disclaimer: Original study notes for exam prep — not official Anthropic material. Not a lesson transcript.
created: 2026-08-23
---

# Agentic Architecture & Orchestration — Domain 1 (27%)

> **Disclaimer:** Original exam-prep study synthesis, aligned to the **official CCAR-F Exam Guide v1.0 (July 2026)** task statements 1.1–1.7 and grounded in public Anthropic docs (Claude Agent SDK, Claude Code). Verify live SDK option names against [code.claude.com/docs/en/agent-sdk](https://code.claude.com/docs/en/agent-sdk) before exam day — names evolve.

> **Why this file exists:** Domain 1 is the **heaviest domain on the exam (27%)** and drives three of the six exam scenarios (Customer Support Resolution Agent, Multi-Agent Research System, Developer Productivity). Everything here maps to a numbered task statement in the official guide.

---

## Overview

The exam's Domain 1 tests whether you can design and reason about **agentic loops** (task 1.1), **coordinator–subagent orchestration** (1.2), **subagent spawning and context passing** (1.3), **workflow enforcement and handoff** (1.4), **Agent SDK hooks** (1.5), **task decomposition** (1.6), and **session state, resumption, and forking** (1.7). Questions are scenario-shaped: production logs show a failure pattern, and you pick the architecturally proportionate fix.

**Master mental model:** an agent = model + tools + loop. The *model* decides which tool to call next from context (model-driven decision-making); the *loop* is dumb plumbing that executes tools and feeds results back; *hooks* are the deterministic policy layer wrapped around that plumbing; *subagents* are extra isolated loops the coordinator spawns for focused work.

---

## Key map (task statement ↔ exam verbs)

| Task | You must be able to… |
| --- | --- |
| 1.1 Agentic loops | Implement stop_reason-driven control flow; name the anti-patterns |
| 1.2 Coordinator–subagent | Design hub-and-spoke; diagnose narrow decomposition; iterative refinement |
| 1.3 Spawning & context | Task tool + allowedTools; AgentDefinition; explicit context passing; parallel spawning |
| 1.4 Workflows & handoff | Programmatic prerequisites vs prompts; structured escalation handoffs |
| 1.5 Hooks | PostToolUse normalization; tool-call interception; deterministic vs probabilistic |
| 1.6 Decomposition | Prompt chaining vs adaptive decomposition; per-file + integration passes |
| 1.7 Sessions | --resume, fork_session; stale-context judgment |

---

## Deep notes

### 1. The agentic loop (task 1.1)

The lifecycle every loop implements:

1. Send the request (messages + tools) to Claude.
2. Inspect **`stop_reason`**: `"tool_use"` → Claude wants tools executed; `"end_turn"` → Claude is done.
3. Execute the requested tool(s).
4. Append the assistant message **and** the `tool_result` blocks to the conversation history.
5. Send the updated history back; repeat until `end_turn`.

```text
loop:
 response = call_claude(history, tools)
 history.append(response)
 if response.stop_reason == "tool_use":
 results = execute(response.tool_calls)
 history.append(user_message(tool_results=results))
 continue
 if response.stop_reason == "end_turn":
 break # final answer is in response content
```

**Why appending tool results matters:** the model is stateless between requests. It can only reason about a tool's output if that output is *in the conversation it receives next*. Dropping or truncating a tool_result silently changes what the model "knows."

**Model-driven vs pre-configured:** the exam distinguishes an agent (Claude reasons about which tool to call next based on accumulated context) from a decision tree or fixed tool sequence coded by you. Agents shine on high-ambiguity inputs (returns, billing disputes) where the path can't be enumerated in advance; fixed pipelines shine when the path is known (see task 1.6).

**Loop termination anti-patterns (the guide names these — memorize):**

| Anti-pattern | Why it fails | Correct signal |
| --- | --- | --- |
| Parsing natural-language signals ("I'm done!") to decide termination | Brittle string matching; models phrase completion many ways | `stop_reason == "end_turn"` |
| Arbitrary iteration cap as the **primary** stopping mechanism | Cuts off legitimate long tasks; masks real bugs | stop_reason; keep a cap only as a *safety backstop* |
| Checking whether the assistant message contains text as a completion indicator | Assistant turns can contain text *and* tool_use blocks together | stop_reason, never content shape |

### 2. Coordinator–subagent orchestration (task 1.2)

**Hub-and-spoke:** one coordinator agent owns task decomposition, delegation, result aggregation, and error handling. Subagents never talk to each other directly — **all inter-subagent communication routes through the coordinator**. Why: observability (one place to log), consistent error handling (one recovery policy), controlled information flow (no unreviewed cross-talk).

**Context isolation (heavily tested):** subagents do **not** automatically inherit the coordinator's conversation history or share memory between invocations. Each spawned subagent starts a fresh context containing only its own system prompt plus the prompt string the coordinator passed. If the synthesis subagent needs the web-search findings, the coordinator must **put those findings in the synthesis subagent's prompt** — nothing flows implicitly.

**The coordinator's job list:**

- Analyze the query and **dynamically select** which subagents to invoke — not always the full pipeline. A simple lookup shouldn't wake four specialists.
- **Partition scope** across subagents to minimize duplication (assign distinct subtopics or source types per agent).
- **Aggregate** results and evaluate the synthesis for gaps.
- **Iterative refinement loop:** if the synthesis has coverage gaps, re-delegate to search/analysis subagents with *targeted* queries, then re-invoke synthesis; repeat until coverage is sufficient.

**The signature Domain 1 failure — overly narrow decomposition** (official sample Q7): the research report on "creative industries" only covers visual arts because the coordinator decomposed the topic into digital art / graphic design / photography. Every subagent executed its assignment correctly; **the root cause is what they were assigned**. Exam reflex: when subagent logs look clean but coverage is bad, blame the coordinator's decomposition, not the downstream agents.

**Coordinator prompt design:** specify **research goals and quality criteria** ("cover every major creative-industry sector; each claim cited") rather than step-by-step procedural instructions. Goal-shaped prompts let subagents adapt; procedure-shaped prompts break on the first unanticipated case.

### 3. Spawning, AgentDefinition, and context passing (task 1.3)

**The Task tool** is the mechanism a coordinator uses to spawn subagents, and the coordinator's **`allowedTools` must include `"Task"`** or it cannot delegate at all — a classic exam distractor is a delegation failure whose root cause is a missing Task entry. *(Current-docs note: Claude Code renamed the tool to `Agent` in v2.1.63; the exam guide and appendix use `Task` — answer with the guide's terminology, know the rename exists.)*

**AgentDefinition** configures each subagent type (verified against current SDK docs — the `agents` option of `query()` maps names to definitions):

| Field | Purpose |
| --- | --- |
| `description` | When the coordinator should use this subagent — this is how Claude matches tasks to agents |
| `prompt` | The subagent's **system prompt**: role, expertise, constraints |
| `tools` | Tool restriction list — a synthesis agent gets no web tools; a reviewer gets read-only `["Read", "Grep", "Glob"]` |
| `model` | Optional per-subagent model override (e.g. cheaper model for bulk reading) |

**Explicit context passing skills the guide lists:**

- Include **complete findings from prior agents directly in the subagent's prompt** (e.g. pass search results and document analyses into the synthesis prompt).
- Use **structured data formats that separate content from metadata** (claim text vs source URL / document name / page number) so attribution survives hand-offs — arrives again in Domain 5.6.
- **Parallel spawning:** emit **multiple Task tool calls in a single coordinator response**, not one per turn. One-per-turn serializes what could run concurrently; the single-response pattern is the documented way to parallelize (official prep Exercise 4 measures the latency win).

**Fork-based exploration** belongs here conceptually (details in §7): fork a session to explore divergent approaches from a shared analysis baseline.

### 4. Multi-step workflows: enforcement and handoff (task 1.4)

**Programmatic enforcement vs prompt guidance — the single most-tested distinction in Domain 1** (official sample Q1):

> 12% of cases skip `get_customer` before `process_refund`. Fix? **A programmatic prerequisite that blocks the downstream call until the prerequisite has completed.** Prompt wording and few-shot examples are probabilistic — they lower the failure rate, never to zero. When errors have financial or compliance consequences ("identity verification before financial operations"), only deterministic gates are acceptable.

Enforcement ladder (weakest → strongest): system-prompt instruction → few-shot examples → **hook / prerequisite gate in code**. The exam rewards choosing the *proportionate* rung: precision problems in review prompts get criteria and few-shots (Domain 4); compliance-critical ordering gets code.

**Multi-concern requests:** decompose a message containing several distinct issues into items, investigate each (in parallel where independent, sharing common context like the verified customer ID), then synthesize **one unified resolution** — don't answer concern 1 and silently drop concern 3.

**Structured handoff protocol for escalation:** the human agent receiving an escalation typically **cannot see the conversation transcript**. A compliant handoff summary carries: customer ID, the concern(s), **root cause analysis so far**, amounts/order numbers involved, what was already attempted, and the **recommended action**. "Escalating to human" with no structured payload forces the human to restart the investigation — an anti-pattern the scenario stems punish.

### 5. Agent SDK hooks (task 1.5)

Hooks are callbacks the SDK fires at fixed lifecycle points; they are the **deterministic guarantee layer**, in contrast to prompt instructions' **probabilistic compliance** — the same advisory-vs-enforced axis as file 05's Claude Code section, now inside the SDK.

The two hook patterns the exam names:

1. **PostToolUse — result transformation before the model sees it.** Different MCP backends return heterogeneous formats: one tool emits Unix epoch timestamps, another ISO 8601, a third numeric status codes. A PostToolUse hook **normalizes tool results into one canonical format before the agent processes them**, so the model never has to juggle three date formats mid-reasoning. *(Current SDK mechanics, verified: hooks are registered via the `hooks` option keyed by event name with matchers; a PostToolUse callback can replace the tool's output — `updatedToolOutput` — or append `additionalContext`.)*
2. **Tool-call interception — enforcing compliance rules on outgoing calls.** Intercept the call *before execution*; if it violates policy (refund > $500), **block it and redirect to an alternative workflow** (human escalation). *(Current SDK mechanics, verified: a PreToolUse callback returns `permissionDecision: "deny"` with a `permissionDecisionReason` the model sees, or `updatedInput` to rewrite arguments; deny always wins when multiple hooks fire.)*

**When hooks, when prompts:** business rules that require **guaranteed** compliance → hooks. Style, tone, soft preferences → prompts. If a question says "must never," "compliance," "regulatory," or names a dollar threshold — the answer is a hook or gate, not a better sentence in the system prompt.

### 6. Task decomposition strategies (task 1.6)

Two families:

| Strategy | Shape | Use when |
| --- | --- | --- |
| **Prompt chaining** (fixed sequential pipeline) | Predefined step 1 → step 2 → step 3 | Predictable multi-aspect work: the steps are known in advance |
| **Dynamic adaptive decomposition** | Plan emerges from intermediate findings | Open-ended investigation: what to do next depends on what you discover |

**Prompt-chaining exemplar (also official sample Q12 / Domain 4.6):** a 14-file PR reviewed in one pass produces inconsistent depth and contradictory findings — **attention dilution**. Fix: **per-file local analysis passes** plus a **separate cross-file integration pass**. Splitting is a decomposition answer, not a bigger-context-window answer (option C in Q12 is wrong precisely because larger windows don't fix attention dilution).

**Adaptive exemplar:** "add comprehensive tests to a legacy codebase" — you can't enumerate steps upfront. First **map the structure**, identify **high-impact areas**, create a **prioritized plan that adapts as dependencies are discovered**. The plan is a living artifact, regenerated as findings arrive.

**Choosing:** ask "could I write the full step list before starting?" Yes → chain. No → adaptive. Hybrid is common: adaptive outer plan, chained inner passes.

### 7. Session state, resumption, and forking (task 1.7)

- **`--resume <session-name-or-id>`** continues a specific prior conversation with its full context (files read, analyses done, decisions made). Verified current CLI: `--resume` accepts a session **ID or name** (or shows a picker); `--continue` grabs the most recent session in the directory. In the SDK, pass `resume=<session_id>` (Python) / `resume: sessionId` (TypeScript) in the query options.
- **`fork_session`** creates an **independent branch from a shared analysis baseline**: the fork copies the original's history and gets its own session ID; **the original stays unchanged**. Use it to explore divergent approaches — e.g. compare two testing strategies or two refactoring designs from one expensive codebase analysis, one fork each. Verified SDK spelling: `fork_session=True` (Python) / `forkSession: true` (TypeScript), passed *together with* `resume`.
- **Resuming after code changed:** a resumed session's tool results describe the files *as they were*. **Tell the agent which files changed** so it re-analyzes those specifically instead of trusting stale reads — targeted re-analysis beats full re-exploration.
- **Resume vs fresh-with-summary:** if prior context is *mostly valid* → resume (cheap, keeps nuance). If prior tool results are *substantially stale* → **start a new session and inject a structured summary** of the still-valid conclusions. A fresh session with clean, current facts is more reliable than a resumed session dragging contradicted file reads the model may still trust.

---

## Decision trees

**"The agent didn't do X" triage:**

```text
Did it skip a REQUIRED step (compliance/order)?
 → Programmatic prerequisite or PreToolUse-style interception hook (1.4/1.5)
Did it pick the WRONG TOOL between similar tools?
 → Tool descriptions first (Domain 2.1), few-shots second
Did it produce INCOMPLETE COVERAGE with clean subagent logs?
 → Coordinator decomposition too narrow (1.2)
Did it FAIL TO DELEGATE at all?
 → Is "Task" in the coordinator's allowedTools? (1.3)
Did it terminate early / loop forever?
 → Check stop_reason handling; remove NL-parsing / text-presence checks (1.1)
```

**Session continuation:**

```text
Prior context still valid?
 YES → --resume (add a note listing any changed files)
 MOSTLY, want to try two directions → resume + fork_session per direction
 NO (stale tool results) → new session + injected structured summary
```

---

## Exam traps

| Trap | Fix |
| --- | --- |
| "Strengthen the system prompt" for a compliance-critical ordering bug | Programmatic prerequisite / hook — prompts are probabilistic |
| Assuming subagents see the coordinator's conversation | They see only their system prompt + the prompt passed at spawn |
| Blaming downstream agents for coverage gaps | Check the coordinator's decomposition first (sample Q7) |
| Spawning parallel subagents across separate turns | Emit multiple Task calls in one response |
| Iteration cap as the primary loop terminator | stop_reason `end_turn`; cap only as backstop |
| Escalating with a bare "needs human" flag | Structured handoff: ID, root cause, amounts, recommended action |
| Resuming a session after heavy code changes without saying so | Name the changed files, or start fresh with a summary |
| Fork mutates the original session | It doesn't — fork copies history into a new session ID |
| Using hooks for tone/style preferences | Hooks for guarantees; prompts for preferences |
| One giant review pass "because the context window fits it" | Attention dilution — split per-file + integration pass |

---

## Self-check Q&A

**Q1.** The loop should continue when `stop_reason` is ___ and terminate when it is ___.
**A.** `"tool_use"`; `"end_turn"`.

**Q2.** Why must tool results be appended to history before the next request?
**A.** The model is stateless per request — it can only incorporate results present in the conversation it receives.

**Q3.** Name the three loop-termination anti-patterns the exam guide lists.
**A.** Parsing natural-language completion signals; arbitrary iteration caps as the primary stop; checking for assistant text content as a completion indicator.

**Q4.** Agent vs pre-configured pipeline — the defining difference?
**A.** Model-driven decision-making: the agent reasons about the next tool from context; a pipeline follows a coded sequence.

**Q5.** In hub-and-spoke, who talks to whom?
**A.** All subagent communication routes through the coordinator — for observability, consistent error handling, and controlled information flow.

**Q6.** Research report covers only a slice of the topic; every subagent's log looks correct. Root cause?
**A.** Coordinator's task decomposition was too narrow — subagent assignments didn't cover the topic (sample Q7 pattern).

**Q7.** What must be true of `allowedTools` for a coordinator to spawn subagents?
**A.** It must include `"Task"` (the subagent-spawning tool; renamed `Agent` in current Claude Code — exam uses Task).

**Q8.** What context does a freshly spawned subagent start with?
**A.** Its own system prompt (AgentDefinition `prompt`) plus the prompt string the coordinator passed — no parent history, no shared memory.

**Q9.** The synthesis agent needs the search agents' findings. How does it get them?
**A.** The coordinator includes the complete findings directly in the synthesis agent's prompt, ideally as structured data separating content from source metadata.

**Q10.** How do you run three subagents in parallel?
**A.** The coordinator emits three Task tool calls in a single response.

**Q11.** AgentDefinition: name its two required fields and what each does.
**A.** `description` (tells the coordinator when to use this agent type) and `prompt` (the subagent's system prompt); `tools` and `model` are optional restrictions/overrides.

**Q12.** 12% of refunds skip identity verification. Best fix?
**A.** Programmatic prerequisite blocking `process_refund` until `get_customer` returns a verified ID — deterministic, unlike prompt/few-shot fixes (sample Q1).

**Q13.** What belongs in a structured escalation handoff?
**A.** Customer ID, concern(s), root-cause analysis, amounts/order numbers, what was attempted, recommended action — the human lacks transcript access.

**Q14.** Three MCP tools return dates as epoch, ISO 8601, and strings. Where do you normalize?
**A.** A PostToolUse hook that transforms tool results into one canonical format before the model processes them.

**Q15.** Refunds over $500 must never execute automatically. Mechanism?
**A.** A tool-call interception (PreToolUse-style) hook that blocks the call and redirects to human escalation — hooks give deterministic guarantees; prompts are probabilistic.

**Q16.** Prompt chaining vs adaptive decomposition — one-line rule?
**A.** Steps enumerable in advance → chain; plan depends on discoveries → adaptive.

**Q17.** Large multi-file review produces contradictory, inconsistent findings. Restructure how?
**A.** Per-file local analysis passes plus a separate cross-file integration pass (attention dilution fix; sample Q12).

**Q18.** "Add comprehensive tests to a legacy codebase" — first three moves?
**A.** Map the structure, identify high-impact areas, build a prioritized plan that adapts as dependencies surface.

**Q19.** fork_session — what does the fork share with, and change about, the original?
**A.** It starts with a copy of the original's history under a new session ID; the original session is untouched.

**Q20.** When is fresh-session-plus-summary better than --resume?
**A.** When prior tool results are stale (code changed heavily) — injected summaries of valid conclusions beat resumed contexts containing contradicted reads.

**Q21.** You resume an investigation session after editing two files. What should your first message include?
**A.** Which files changed, so the agent re-analyzes those specifically rather than trusting stale reads or re-exploring everything.

**Q22.** Coordinator prompts should specify ___ rather than ___.
**A.** Research goals and quality criteria; step-by-step procedural instructions (adaptability).

**Q23.** When may an iteration cap legitimately exist in a loop?
**A.** As a safety backstop against runaways — never as the primary termination mechanism.

**Q24.** The coordinator routes every query through all four subagents, even trivial ones. Critique?
**A.** Coordinators should dynamically select subagents by query complexity — full-pipeline routing wastes latency/cost and adds failure surface.

**Q25.** Iterative refinement loop — describe it.
**A.** Coordinator evaluates synthesis output for gaps → re-delegates targeted queries to search/analysis agents → re-invokes synthesis → repeats until coverage is sufficient.

---

## Pre-exam checklist

- [ ] I can write the stop_reason loop from memory and name all three termination anti-patterns.
- [ ] I can explain why subagent context isolation forces explicit context passing.
- [ ] I know Task must appear in allowedTools (and about the Task→Agent rename).
- [ ] I can list AgentDefinition's fields and what tool restriction is for.
- [ ] I default to programmatic gates for compliance ordering, prompts for preferences.
- [ ] I can spec a structured escalation handoff.
- [ ] I can pick PostToolUse (normalize results) vs interception (block policy violations).
- [ ] I can choose prompt chaining vs adaptive decomposition and justify per-file + integration passes.
- [ ] I can choose between --resume, fork_session, and fresh-with-summary.

---

## Glossary

| Term | Meaning |
| --- | --- |
| Agentic loop | Request → stop_reason check → execute tools → append results → repeat |
| stop_reason | API field driving loop control: `tool_use` continue, `end_turn` stop |
| Coordinator | Hub agent owning decomposition, delegation, aggregation, error handling |
| Hub-and-spoke | Topology where all subagent communication passes through the coordinator |
| Task tool | The subagent-spawning tool (renamed Agent in current Claude Code) |
| allowedTools | SDK option auto-approving listed tools; must include Task to delegate |
| AgentDefinition | Per-subagent config: description, prompt, tools, model |
| Context isolation | Subagents start fresh; only the spawn prompt carries context in |
| Programmatic prerequisite | Code gate blocking a downstream tool until a prerequisite completed |
| PostToolUse hook | Callback transforming/normalizing a tool result before the model sees it |
| Tool-call interception | Pre-execution hook blocking or rewriting policy-violating calls |
| Prompt chaining | Fixed sequential pipeline of focused passes |
| Adaptive decomposition | Plan generated and revised from intermediate findings |
| Attention dilution | Quality degradation when one pass covers too many files/aspects |
| --resume | CLI/SDK option continuing a named/ID'd session with full context |
| fork_session | Branch a session's history into a new independent session ID |
| Structured handoff | Escalation payload: ID, root cause, amounts, attempts, recommendation |

---

*Companion files: 09 (Domain 5 context & reliability), 04 supplement (Domain 2 MCP mechanics), 05 supplement (Domain 3 Claude Code mechanics). Verify SDK spellings at code.claude.com/docs/en/agent-sdk before exam day.*
