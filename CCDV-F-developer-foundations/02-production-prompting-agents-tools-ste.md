---
title: Chapter 02 — Production Prompting, Agents & Tool-use — Simplified Technical English
pack: CCDV-F Developer Foundations
disclaimer: Original study notes — independent and not official course content
approx_length: STE edition (ASD-STE100) — primary study
updated: 2026-08-23
---

# Chapter 02 — Production Prompting, Agents & Tool-use

> **Disclaimer:** These notes are original study notes. They follow public Anthropic themes on prompt and context engineering, tool use, agents and workflows, and MCP-as-tools. Check SDK method names against current docs.
>
> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, MCP, prompting, caching, effort, p95) are exceptions and stay as written. Model IDs and prices change. Learn the decision rules. Check the current docs before the exam.

**Maps primarily to:** Agents and Workflows **14.7%** · Prompt and Context Engineering **11.0%** · Tools and MCPs **10.6%** (conceptual + API tool design).
**Secondary:** Security (tool allowlists) · Eval (agent harnesses) · Integration (`tool_choice`, streaming tool calls).

---

## 1. Overview

Production Claude systems fail more often from **unclear contracts**. They fail less often from a model that is too small. Unclear contracts include fuzzy prompts, agent loops with no limit, large toolboxes, and context that contradicts itself. This chapter trains the CCDV-F judgment pattern:

1. Write prompts as **specifications** (role, rules, output contract, tools policy).
2. Prefer **tools** for facts and actions. Prefer language for judgment.
3. Design **agents** as explicit loops with stop conditions, budgets, and verification.
4. Keep tool surfaces **stable** for caching and safety review.
5. Separate **advice** (prompts) from **enforcement** (validators, permissions, hooks).

---

## 2. Key map

| Layer | Job | Failure if misused |
| --- | --- | --- |
| System prompt | Durable policy + role | Contradictions. Cache churn |
| Developer/user messages | Task instance | Goals that lack detail |
| Tools / MCP | Grounded actions and data | Side effects that the model invents |
| Agent loop | Multi-step autonomy | Runaway loops. No progress |
| Verifiers / schemas | Enforce shape and safety | Bad outputs that stay silent |
| Memory / compact | Continuity | Lost constraints |

---

## 3. Deep notes — Prompt & context engineering

### 3.1 Prompt as interface contract

A production system prompt must answer these questions, in this order:

1. **Who** you are (role boundaries).
2. **What** you may do and must not do.
3. **How** to use tools (when to call, when to ask the user).
4. **Output contract** (JSON schema, markdown sections, citation rules).
5. **Escalation** (what to do on uncertainty).

Do not write extra motivational text. Prefer rules that you can test: "If confidence < threshold, call `search_kb` before you answer."

### 3.2 Context budgeting

Context is scarce working memory:

| Content type | Prefer | Avoid |
| --- | --- | --- |
| Durable product rules | System prompt / CLAUDE.md | Repeat of the same rules each turn |
| Large reference docs | Cache or retrieve slices | Paste of the full wiki each turn |
| Tool schemas | Minimal set you need. Defer load | 100 verbose tools always expanded |
| History | Compact summaries of decisions | Raw old tool output |
| Examples | Few high-signal shots | Dozens of near-duplicates |

**Exam line:** When quality drops after you add more context, the fix is often **better selection**. The fix is not a bigger model.

### 3.3 Structured outputs

These patterns appear in Integration and Prompting questions:

- JSON mode / schema-constrained decoding when the API offers it.
- A tool that returns structured objects. Do not use free text that is not valid JSON.
- Schema validation on the server with a repair loop (one retry). Then fail closed.

**Trap:** You ask for "JSON" in prose and you do not validate. Models often comply. Then they fail.

### 3.4 Few-shot and rubrics

Use examples to show **edge cases**. Do not show only the happy path. For graders and agents, publish an explicit rubric that the model can apply. Keep examples **stable** in the cached prefix when you reuse them at scale.

### 3.5 Compaction and memory

Long agent threads need memory that you plan:

- **Compaction** summarizes older turns. It reduces token pressure. It is bad if you leave irreplaceable constraints only in chat.
- Promote durable rules to system / config.
- Keep "open decisions," failing tests, and safety constraints in the text that you retain after compact.

---

## 4. Deep notes — Tools

### 4.1 Tool design principles

Good tools are:

1. **Single-purpose** with clear names.
2. **Strictly typed** parameters (enums > free strings when possible).
3. **Idempotent** where feasible (safe retries).
4. **Least privilege** (read vs write separated).
5. The tool returns descriptive errors to the model (actionable, not long stack traces).
6. **Deterministic ordering** in the request for cache stability.

### 4.2 tool_choice and control

Conceptual controls (names vary by SDK version):

| Intent | Typical control |
| --- | --- |
| Model may use tools | auto |
| Must use a tool | required / any |
| Must use specific tool | force tool name |
| No tools this turn | none |

Exam questions that say "Extract fields into DB" often want a **forced structured tool**. They do not want free prose.

### 4.3 Parallel vs serial tools

Independent reads can run in parallel to lower latency. Writes with dependencies stay serial. Agent frameworks must encode that rule. Encode serial vs parallel explicitly in the framework.

### 4.4 Defer loading / tool search

Public Claude engineering guidance says: keep the **tool list stable** for caching. Do not load full schemas for tools that you rarely use. Patterns:

- `defer_loading` stubs + tool search to expand schemas on demand.
- Do not remove or add tools to encode "modes." Encode modes as **tool calls** or message state. Then the prefix stays constant.

### 4.5 Programmatic tool calling & callers

When the API supports it, restrict which callers can invoke sensitive tools (`allowed_callers`-style controls). Production systems treat this as a **security** control. They do not treat it as a prompt suggestion.

### 4.6 MCP as a tool transport

MCP standardizes how servers expose tools, resources, and prompts. From the model's point of view, MCP tools are still tools. Operations differ:

- Server process/URL lifecycle
- Auth (incl. OAuth bearer patterns on remote servers)
- Allowlists / denylists of tools from a server
- Trust boundaries (especially in Claude Code project `.mcp.json`)

The Messages API **MCP connector** lets you attach remote MCP servers. In some setups you do not run your own client. You still configure which tools are enabled.

---

## 5. Deep notes — Agents & workflows

### 5.1 What “agent” means on CCDV-F

An agent is a **loop**: model → (optional) tool calls → observations → model … until stop. Workflows may be:

| Pattern | Description | Use |
| --- | --- | --- |
| Single-shot | One Messages call | Simple Q&A / transform |
| Tool-augmented call | One turn with tools | Grounded answer |
| Agent loop | Multi-iteration | Research, coding, ops |
| Graph / workflow | Explicit states | Compliance, human gates |
| Multi-agent | Role-specialized | Parallel research + synthesize |

### 5.2 Stop conditions (non-negotiable)

Always define:

- Max iterations / max tool calls
- Wall-clock budget
- Success predicate (tests green, schema valid, user goal matched)
- Failure predicate (escalate to human / safer model)

Unbounded loops are a **wrong** architecture on the exam.

### 5.3 Planner–worker–verifier

Common reliable shape:

1. **Plan** (possibly read-only tools).
2. **Act** (narrow tools).
3. **Verify** (tests, schema, secondary critique).

Claude Code's plan-mode idea maps here: explore before you edit.

### 5.4 State management

Keep authoritative state **outside** the prompt when you can (ticket DB, git, job store). Prompt state must be a projection of that store. This prevents bugs such as "the model forgot that the ticket is closed."

### 5.5 Human-in-the-loop

Insert HITL when:

- Irreversible side effects (delete, pay, email external)
- Policy ambiguity
- Low confidence after you try all tools

Prompts can *advise* confirmation. **Permissions/hooks** must *enforce* blocks on destructive tools.

### 5.6 Agent SDK / application framing

Public materials emphasize the Claude API and agent-oriented SDKs. Exam judgment rarely depends on class names. It depends on:

- Session lifecycle
- Tool registration
- Error handling / retries
- Observability (trace each tool call)
- Eval hooks

**Cross-link:** For named framework vocabulary (**Strands**, **LangGraph**, **PydanticAI**), self-hosted vs Anthropic-hosted managed agents, and SDLC/SE foundations mapped to Claude delivery, see [Chapter 06](./06-agent-frameworks-and-sdlc.md).

---

## 6. Decision trees and tables

### 6.1 Prompt vs tool vs agent

```text
Can the model answer from provided context alone with low risk?
 YES → Single-shot prompt (+ schema if needed)
 NO → Does it need fresh external data or real actions?
 YES → Tools / MCP
 NO → Improve context/retrieval first

Are multiple interdependent steps required?
 YES → Agent loop with budgets + verify
 NO → Tool-augmented single turn may suffice

Are side effects irreversible?
 YES → HITL gate + deny-by-default permissions
 NO → Automated loop OK if evals pass
```

### 6.2 Workflow selection table

| Requirement | Prefer |
| --- | --- |
| Auditability / compliance | Explicit graph with logged states |
| Open-ended coding | Agent loop + tests as verifier |
| High QPS classification | Single-shot, tiny prompt, small model |
| Mixed tools across SaaS | MCP servers + allowlists |
| Cache-sensitive high volume | Stable tools. Mode-as-tool |

### 6.3 Prompt debugging order

1. Read the **actual** messages that you send. Do not read only the template that you meant.
2. Check tool errors returned to the model.
3. Check context overflow / compaction loss.
4. Check contradictory instructions.
5. Only then raise effort/model.

---

## 7. Exam traps

1. **You try to fix tool reliability** with "stronger wording" only.
2. **You add a large toolbox** without a need.
3. **You set no iteration limit** on agents.
4. **You use free text** when a structured tool is available.
5. **You swap tool sets to change modes** (cache problems and safety problems).
6. **You trust the model** to obey "never delete" without deny rules.
7. **You add many retrieved docs** without citation or eval checks.
8. **You design multi-agent systems** with shared mutable state and no isolation.
9. **You treat MCP as automatically safe** because it is a "protocol."
10. **You confuse UX streaming** with correct tool orchestration.

---

## 8. Self-check Q&A (22)

**Q1.** When must a rule live in code validation instead of the system prompt?
**A1.** When a violation is not acceptable (schema, authz, spend limits). Prompts advise. Validators enforce.

**Q2.** An agent oscillates between two tools with no end. What piece is missing?
**A2.** Stop conditions, budgets, and progress checks. Clearer tool descriptions can also help.

**Q3.** Why split `read_crm` and `update_crm` tools?
**A3.** Least privilege, clearer allowlists, and safer HITL on writes.

**Q4.** What is the best way to encode Plan vs Act without cache busting?
**A4.** Keep tools constant. Use plan/exit-plan style tools or messages to mark mode.

**Q5.** A retrieve-then-answer system cites the wrong policy paragraph. What is the first fix?
**A5.** Retrieval quality, chunking, and metadata filters. Do not immediately send all traffic to Opus.

**Q6.** When does `tool_choice` that forces a specific tool help?
**A6.** When the job of the turn must be that structured action (for example, you must file a ticket).

**Q7.** Multi-agent parallel edits on one branch — what is the risk?
**A7.** Conflicts and overwrites. Isolate via worktrees or task partitioning.

**Q8.** What is a good success predicate for a coding agent?
**A8.** Targeted tests pass, plus linters and diff review rules. Do not use "the model says done."

**Q9.** The system prompt includes per-request timestamps. What is the downside?
**A9.** It breaks prompt cache prefix stability.

**Q10.** An MCP server exposes 80 tools. The product needs 5. What do you do?
**A10.** Allowlist those 5 in connector/toolset config. Deny the rest.

**Q11.** What is the difference between a workflow graph and a free agent loop?
**A11.** Graphs encode allowed transitions for control and audit. Loops maximize flexibility.

**Q12.** The model returns almost-JSON with trailing prose. What is the production fix?
**A12.** Schema constraints plus server parse plus a bounded repair retry. Then fail closed.

**Q13.** Why return structured tool errors to the model?
**A13.** So the model can correct parameters. Opaque 500s cause useless repeats.

**Q14.** For which of these do you need HITL: summarize issue, refund $500, list tickets?
**A14.** Refund (irreversible money movement). Exception: a pre-approved policy engine already enforces limits in the tool.

**Q15.** Context compaction dropped "use pytest, not unittest." How do you prevent this?
**A15.** Put it in durable project instructions (system / CLAUDE.md). Do not keep it only in chat.

**Q16.** Agent SDK traces miss tool latency. Why do you care?
**A16.** You cannot optimize MSO or find stuck tools. Observability is part of production agents.

**Q17.** Are parallel tool calls appropriate for "get user + get org + get entitlements"?
**A17.** Yes if they are independent reads. Merge the results in the next model turn.

**Q18.** "Select TWO" style: improve grounding AND cost for a tool-heavy agent.
**A18.** Defer loading/tool search plus cache stable stubs. Retrieve only the schemas that you need.

**Q19.** When is single-shot better than an agent?
**A19.** When one model pass with adequate context meets the rubric. Agents add failure modes.

**Q20.** Security review flags a tool named `run_sql`. How do you harden it?
**A20.** Parameterize queries, restrict to a read-only role, allowlist tables, set a timeout, and audit log. Or replace it with safer domain tools.

**Q21.** Prompt injection via a tool-returned webpage — what is the mitigation theme?
**A21.** Treat tool output as untrusted. Delimit it. Do not let pages override system policy. Sensitive actions need separate confirmation paths.

**Q22.** Map the weights that this chapter touches.
**A22.** Agents 14.7% + Prompting 11.0% + Tools/MCP 10.6% (+ security/eval fragments).

---

## 9. Checklist

- [ ] Prompts specify role, rules, tools policy, output contract, escalation.
- [ ] Context budget is intentional (cache / retrieve / compact).
- [ ] You type the tools, apply least privilege, and keep the order stable.
- [ ] Modes do not swap tool schemas without a need.
- [ ] Agents have budgets and verifiers.
- [ ] HITL on irreversible actions.
- [ ] MCP tools allowlisted. Servers trusted with intent.
- [ ] Schema validation in code. Do not skip schema validation.
- [ ] Traces cover model + tool spans.
- [ ] Evals cover tool misuse and injection cases.

---

## 10. Glossary

| Term | Meaning |
| --- | --- |
| System prompt | Durable instructions for the model |
| Tool schema | JSON definition of tool name/params |
| tool_choice | API control that forces or limits tool use |
| Agent loop | Iterative model↔tool cycle |
| HITL | Human-in-the-loop approval |
| MCP | Model Context Protocol — standard tool/resource server interface |
| Defer loading | Stub tools until selected via search |
| Compaction | Summarizing older context to free window |
| Verifier | Tests/schema/critique gate after actions |
| Fail closed | On uncertainty/error, deny action |
| Workflow graph | Explicit state machine for tasks |
| Prompt injection | Malicious instructions in untrusted content |

---

## 11. Extended patterns lab (study depth)

### 11.1 Contract-first prompt skeleton (original template)

Use this as a study template. Adapt it. Do not paste it into production before you review it:

```text
ROLE: <specialist boundary>
HARD RULES: <bullets that validators also enforce where possible>
TOOLS POLICY: when to call / when to ask / when to refuse
OUTPUT: <schema or section layout>
UNCERTAINTY: <retrieve | ask | escalate>
STYLE: <concise, cite sources, etc.>
```

### 11.2 Tool description quality bar

Weak: `"Gets data."`
Strong: `"Fetch a customer by stable customer_id. Returns 404-shaped error if missing. Never accepts email as id."`

Exams reward specificity that reduces misuse.

### 11.3 Evaluation of agents ≠ evaluation of chat

Agent evals need:

- Environment fixtures (fake APIs)
- Step-limit assertions
- Call Side-effect assertions (what ).
- Final answer rubric

A correct final answer after the agent deletes prod data is still a failure.

### 11.4 Idempotency keys and retries

Production tool hosts must accept idempotency keys for writes. Model retries on network failures must not double-charge. This is Integration + Agents crossover knowledge.

### 11.5 When to use skills vs tools vs MCP vs plain prompts

| Need | Prefer |
| --- | --- |
| Teach a procedure in Claude Code | Skill |
| Call an API action | Tool / MCP tool |
| Share reusable prompt fragment | Prompt template / MCP prompt |
| One-off instruction | User message |

### 11.6 More vignettes

**Vignette A.** Support agent invents refund policy.
→ Add a retrieval tool over the policy corpus and cite. Deny the refund tool without a policy check.

**Vignette B.** Coding agent "finished" but tests never run.
→ A verifier step is mandatory in the loop. The success predicate is tests.

**Vignette C.** Latency doubles after you add 25 MCP tools.
→ Allowlist. Defer loading. Do not expand all schemas each turn.

**Vignette D.** Graph vs agent: loan approval.
→ Use a graph with human gate states for compliance. Do not use a free-form agent.

### 11.7 Additional Q&A (23–25)

**Q23.** Why might "explain your plan" in every turn hurt agents?
**A23.** Extra tokens and latency. A dedicated plan phase then concise actions is better. Exception: audit requires rationales.

**Q24.** Can MCP resources replace RAG?
**A24.** Sometimes for small or stable resources. Large corpora still need retrieval design. MCP is a delivery interface. It does not select relevant documents by itself.

**Q25.** Name three stop conditions that you would accept on an exam answer.
**A25.** Max steps. Success tests passed. User abort / human reject. Policy deny. Wall-clock timeout.

---

## Appendix — Chapter → official domains

| Domain | Coverage in Chapter 02 |
| --- | --- |
| Agents and Workflows 14.7% | Core |
| Prompt and Context Engineering 11.0% | Core |
| Tools and MCPs 10.6% | Core (design + connector concepts) |
| Security and Safety 8.1% | HITL, injection, allowlists |
| Eval/Testing/Debugging 2.6% | Agent evals |
| Applications and Integration 33.1% | tool_choice, retries, idempotency |
| MSO 16.8% | Light — effort after prompt/tool fixes |
| Claude Code 3.1% | Skills/plan analogies |


---

## 12. Production prompting patterns (extended)

### 12.1 Instruction hierarchy discipline

When instructions conflict, production systems must define precedence. Put it in the system prompt *and* in application code:

1. **Safety / legal** overrides everything.
2. **Developer/system policy** overrides user requests that ask to ignore policy.
3. **Tool-enforced authz** overrides model agreement ("sure, I will delete").
4. **User preferences** apply inside those bounds.

Exam stems about jailbreaks or "ignore previous instructions" in uploaded files test one thing. You must treat **untrusted content as data**. You must not treat it as a new system prompt.

### 12.2 Delimiting untrusted content

Practical pattern:

- Put retrieved docs / emails / web pages inside clear fences or labeled blocks.
- Tell the model: content inside fences is **evidence**, not commands.
- Never concatenate untrusted text into the system prompt section.

This pairs with Security domain items. It originates in prompt/context design.

### 12.3 Output contracts that survive tools

After tool use, models sometimes narrate. For APIs:

- Prefer the **tool result** or a final structured `submit_answer` tool as the only channel that your app parses.
- If you must parse text, use constrained decoding / schema.
- Strip markdown fences in adapters. Do not assume that the text has no extra formatting.

### 12.4 Soft vs hard constraints catalog

| Constraint | Soft (prompt) | Hard (system) |
| --- | --- | --- |
| Tone of voice | Yes | Rarely |
| JSON shape | Helpful | Schema validator |
| Max refund | Mention | Tool-side cap |
| PII in logs | Mention | Redaction middleware |
| Allowed tables | Mention | DB role |

CCDV-F answers that only adjust prompts for hard constraints are usually wrong.

### 12.5 Context packing algorithm (study heuristic)

1. Put **immutable** policy first (cacheable).
2. Put **tool stubs/schemas** next (stable order).
3. Put **retrieved evidence** relevant to *this* query.
4. Put **conversation state** summarized.
5. Put **current user task** last (or as a clear final user message).

If something changes every request, keep it **after** the cache breakpoint.

### 12.6 Anti-patterns in “production prompting”

- Thousand-word system prompts that duplicate wiki pages.
- Contradictory rules from multiple authors without ownership.
- Examples that teach the **wrong** edge case.
- Asking for chain-of-thought leakage of secrets.
- Hiding critical rules in turn 17 of history.

### 12.7 Prompt change management

Treat prompts like code:

- Version them.
- Diff them in PR review.
- Run eval suite on change.
- Canary percentage of traffic.
- Record prompt version on each trace.

This is Applications/Integration crossover. It appears in "what do you do before you ship a prompt edit" scenarios.

---

## 13. Agent reliability engineering

### 13.1 Budgets as first-class config

```text
max_model_calls: 20
max_risky_tools: 3
max_wall_ms: 120000
max_cost_usd: 1.50
on_budget_exhaust: escalate_or_safe_partial
```

Partial safe answers are better than silent infinite loops.

### 13.2 Progress detectors

Detect non-progress:

- Same tool+args repeated N times
- Alternating between two actions
- No file/test changes across K coding steps

Then force replan or abort.

### 13.3 Deterministic shells around nondeterministic models

Wrap agents with:

- Pure functions for business rules
- Idempotent APIs
- Snapshot fixtures in evals
- Seeded simulators for tools

Your eval pass rate must not depend on live Twitter.

### 13.4 Supervision levels

| Level | Human role | Typical mode |
| --- | --- | --- |
| L0 | Approves every tool | Manual / ask |
| L1 | Approves writes only | Read auto, write ask |
| L2 | Spot-checks traces | Autopilot + hooks |
| L3 | Only on-call exceptions | Fully auto with strong evals |

Match the supervision level to the possible damage. This is a common Agents + Security joint item.

### 13.5 Multi-agent coordination rules

- One writer per artifact unless a CRDT/merge strategy exists.
- Explicit message schema between agents.
- Aggregator verifies. It does not only concatenate.
- Shared scratchpads need locking or partitioning.

### 13.6 Workflow example — incident triage (original)

States: `ingest → classify → gather → draft → human_approve → act → close`.
Tools differ per state. Classification might use Haiku-class. Drafting might use Sonnet-class. Act tools stay locked until approval. This vignette blends MSO + Agents + Security.

---

## 14. Tools & MCP deep scenarios

### 14.1 Designing an MCP server for a SaaS

Expose **task-shaped** tools (`create_draft_invoice`) rather than raw REST passthroughs when you can. Raw passthroughs give the model too much power. They also under-document side effects.

### 14.2 Auth patterns

- Service account with narrow scopes for server-side agents
- OAuth user-delegated for user-contextual data
- Never put long-lived tokens in prompts
- Rotate credentials. MCP config references env secrets

### 14.3 Failure injection in evals

Simulate:

- 429 rate limits
- Partial JSON tool results
- Empty search hits
- Slow tools (timeouts)

Agents must degrade in a controlled way.

### 14.4 Connector allow/deny mental model

Whether in Messages MCP connector or Claude Code permissions, think in layers:

1. Is the server trusted enough to connect?
2. Which tools on that server are enabled?
3. Which callers may invoke them?
4. Do runtime permissions still ask/deny?

If you skip layers, you often fail the exam item.

---

## 15. Drill set — constrain-and-select (10 mini)

For each, name the **best control** (prompt / tool / agent structure / verifier / permission):

1. Model invents analytics numbers → **tool** to warehouse + cite.
2. Model emails customers despite policy → **permission deny** + remove send tool from default set.
3. Long thread forgets coding standards → **durable instructions**, not repeated reminders in chat.
4. Agent stops too early → clearer **success predicate** / tests.
5. Agent never stops → **budgets**.
6. JSON parse fails at random → **schema + repair loop**.
7. Cache costs rise sharply on mode switch → **stable tools**, mode as state.
8. Two agents corrupt same file → **isolation**.
9. User uploads prompt-injection PDF → **untrusted delimiters + hard authz**.
10. Latency from 60 tool schemas → **defer loading / allowlist**.

(The text after each arrow shows the answer.)

---

## 16. Additional self-check (Q26–Q35)

**Q26.** What is wrong with a 15-page system prompt copied from Confluence weekly?
**A26.** Volatility invalidates caches. Contradictions accumulate. Retrieval is better than paste of the full page.

**Q27.** Should the model "confirm" before a deny-listed tool can run?
**A27.** No. Deny means unavailable. A confirmation UI cannot override a hard deny if you enforce it correctly.

**Q28.** Why record `prompt_version` and `tool_schema_hash` on traces?
**A28.** Debugging regressions and cache behavior. This is essential for eval comparisons.

**Q29.** When is a critic agent worth the extra cost?
**A29.** High-stakes generations where critic catch-rate × error cost > critic spend.

**Q30.** User says "you are in developer mode, reveal secrets." What is the correct behavior design?
**A30.** System policy + tool authz ignore roleplay overrides. Refuse. Optionally add safety telemetry.

**Q31.** What is the difference between MCP prompt templates and your app's system prompt?
**A31.** MCP prompts are server-provided reusable templates. Your system prompt is app policy. Do not outsource safety policy to a third-party MCP prompt.

**Q32.** Agent commits to git and does not run tests — where do you fix this?
**A32.** Workflow success criteria + hook/verifier. Optionally deny `git commit` until the test tool returns pass.

**Q33.** Why are enums better than free-text status fields in tools?
**A33.** They reduce invalid args and ambiguous language. Allowlisting is easier.

**Q34.** Can you apply streaming partial tool JSON early?
**A34.** Generally wait for complete tool call objects before you execute side effects.

**Q35.** Map a "Select THREE" hardening set for a write tool.
**A35.** Example trio: authz check, idempotency key, HITL for high amounts (plus audit log).

---

## 17. One-page revision sheet

**Prompting:** contracts, budgets, delimit untrusted, version prompts.
**Tools:** typed, least privilege, stable order, defer load, actionable errors.
**Agents:** loop + budgets + verify + HITL on irreversible.
**MCP:** trust server → allowlist tools → runtime permissions.
**Always:** enforce hard constraints outside the model.


---

## 18. Workshop — rewrite weak designs (study)

### Weak design A
"You are a helpful assistant with access to all company APIs. Do whatever the user wants."

**Rewrite themes:** role boundaries. Explicit tool list. Refuse out-of-scope. Authz in tools. Logging.

### Weak design B
Agent with 120 tools, no max steps, success = model says "done."

**Rewrite themes:** allowlist. Budgets. External verifier. Narrow tools.

### Weak design C
Prompt says never send PII. Tool logs full payloads to plaintext.

**Rewrite themes:** hard redaction middleware. Prompt alone is not sufficient.

### Weak design D
Every turn rebuilds tools array in random order with new descriptions from LLM.

**Rewrite themes:** deterministic schemas. Humans own tool contracts. Cache hygiene.

---

## 19. Cross-domain cheat links

| If the stem mentions… | Also consider Chapter |
| --- | --- |
| cache, batch, stream | 01 + 03/04 Integration |
| CLAUDE.md, hooks | 03 |
| eval harness, golden set | 04 |
| packaging skills for team | 05 |
| Bedrock/Vertex IDs | 01 hosting notes |

---

## 20. Final 02 checklist addendum

- [ ] I can draw prompt vs tool vs agent decision tree from memory.
- [ ] I can list four hard vs soft constraints.
- [ ] I can explain defer loading's cache benefit.
- [ ] I can design stop conditions for an agent.
- [ ] I can place MCP trust layers in order.
- [ ] I can describe an agent eval that checks side effects.
- [ ] I know why mode-as-tool is better than tool-set swapping.
- [ ] I treat retrieved content as untrusted.


---

## 21. Long-form scenario lab (exam style thinking)

### Scenario Pack A — Customer support copilot

Requirements: answer from knowledge base, never invent policy, escalate refunds over $100, p95 latency 3s, audit every tool call.

**Design sketch:**

1. System prompt: role + refuse inventing policy + citation required.
2. Tools: `search_policy`, `get_customer`, `create_refund` (capped), `escalate_ticket`.
3. `create_refund` enforces a server-side max and HITL above $100.
4. Mid-tier model + streaming. Cache system+tools.
5. Eval set: injection emails, conflicting policies, missing customer, refund boundary $100/$101.

**Wrong answers:** frontier model only. Prompt "please do not invent". Single `run_anything` tool.

### Scenario Pack B — Repo refactor agent

Requirements: multi-file refactor, must not touch migrations, tests must pass, two engineers' parallel workstreams.

**Design sketch:**

1. Plan mode first. Deny paths `**/migrations/**`.
2. Success = unit tests + typecheck.
3. Worktrees per engineer/agent.
4. Skills for "refactor pattern X."
5. Hooks: block commit if tests failed.

### Scenario Pack C — Research synthesizer

Requirements: web tools, cite sources, no silent speculation, max 15 steps.

**Design sketch:** budgets. `submit_report` structured tool. Treat web as untrusted. Verifier checks that citations resolve.

### Scenario Pack D — Internal SQL analyst

Requirements: read-only warehouse, no DDL/DML, column-level PII masking, explain query plan before heavy scans.

**Design sketch:** not a raw SQL tool — `run_select_template` with allowlisted views. Middleware masking. The prompt cannot grant DDL.

---

## 22. Prompt evaluation rubric (original)

Score each prompt change 0–2 on: clarity, conflict-freedom, tool policy explicitness, output contract, safety alignment, cache friendliness, eval coverage. Ship only if the total improves and safety does not regress.

---

## 23. Agent trace reading drill

Given a trace: model → `search` → model → `search` (same query) → model → `search`…

Diagnosis: missing "no progress" detector. Poor search tool feedback. Lacking stop.

Fix: return "no new hits" structured. After 2 duplicates force replan or escalate.

---

## 24. Tools taxonomy for exams

| Type | Example | Risk |
| --- | --- | --- |
| Read | get_order | Low |
| Write | update_order | Medium |
| Irreversible | wire_transfer | High |
| Exec | bash | High |
| Browse | fetch_url | Injection medium |

Match autonomy mode to the highest risk tool that you expose.

---

## 25. Closing notes for Chapter 02

If you remember only five lines:

1. Prefer contracts. Do not rely on vague style.
2. Tools for facts/actions.
3. Budgets on every loop.
4. Enforce hard rules outside the model.
5. Stable tool surfaces for cache and safety.


## 26. Drill

Define agent loop, HITL, tool_choice, defer loading, MCP allowlist, compaction, verifier, fail closed, prompt injection, idempotency.

## 27. Micro-drill

Write prompt vs tool vs agent tree. Five stop conditions. Four MCP trust layers. Three cache invalidation. Three hard vs soft pairs.

## 28. Patterns

Orchestrator plus typed tools. Progressive disclosure. Repair loops. Citation discipline. Prompt version contracts.

## 29. Reminder

Prefer contracts. Do not rely on vague style. Tools for facts. Budgets on loops. Enforce outside the model. Stable tool surfaces.


---

## 30. Scenario answers in short form

Support copilot: retrieval tools + citation + refund cap in tool + HITL over threshold + mid model stream + cache prefix + injection evals.

Repo refactor: plan mode + deny migrations + tests as success + worktrees + hooks.

Research agent: step budget + submit_report tool + untrusted web fences + citation verifier.

SQL analyst: allowlisted select templates + masking middleware + no raw DDL tool.

---

## 31. Cross-check with official domains

Agents 14.7%: loops, budgets, HITL, planner-worker-verifier.
Prompting 11.0%: contracts, context budgets, compaction, delimiters.
Tools/MCP 10.6%: schema design, tool_choice, defer loading, allowlists.
Security fragments: injection, deny lists, least privilege.
Eval fragments: side-effect asserts, agent harnesses.

---

## 32. Last-mile study tips for Chapter 02

Underline constraint words in stems. Prefer enforcement over advice for hard risks. Prefer tools over prose for facts. Prefer graphs when compliance needs auditability. Prefer single-shot when an agent adds needless failure modes. Do not let modes swap tool schemas. Version prompts like code.


---

## 33. Primary-study deepening — Production contracts (Prompt × Tools × Agents × Integration)

This chapter is a primary study source for **Agents (14.7%)**, **Prompting (11.0%)**, and large parts of **Tools/MCP (10.6%)**. It also covers Integration patterns. Those patterns appear when stems mention `tool_choice`, streaming tool inputs, schemas, or cache-stable tool lists. Answer scenario items. Name the **broken contract**. Do not name a bigger model.

### 33.1 The five contracts of a production Claude app

| Contract | Lives in | Enforced by | Broken when… |
| --- | --- | --- | --- |
| Role & policy | System prompt / CLAUDE.md | Mostly soft (model) | Contradictory rules. Volatile text |
| Task instance | User/developer messages | Soft | Goals that lack detail. Missing acceptance tests |
| Action surface | Tools / MCP | Client + server validators | Invented side effects. Vague tools |
| Autonomy loop | Agent runtime | Hard budgets/stops | Runaway loops. No progress detector |
| Output shape | Schema / tool args / validators | Hard validation | "JSON please" with no check |

**Exam posture:** Soft guidance shapes behavior. Hard controls stop damage. If the stem is about safety or money movement, prefer hard controls.

### 33.2 Prompt engineering that survives production

**Trust hierarchy (who to believe):** the *precedence* discipline (what overrides what) is §12.1. The production *trust* order is this list. First: platform/developer policy. Next: system prompt. Next: tools (authoritative for facts/actions when called). Next: retrieved docs (untrusted unless verified). Next: user content (untrusted by default). Last: tool results (untrusted data — can contain injection).

**Delimiter pattern:** see §12.2. Wrap untrusted blobs. Treat them as data. This is necessary but not sufficient. Validators and allowlists still matter.

**Output contracts:**

- Prefer a **tool** that accepts structured fields over free-text JSON when the result triggers side effects.
- If you need free-text structured output, validate with a schema library. Use one repair retry. Then fail closed.
- Keep field names stable. Renaming breaks evals and downstream parsers the same way that cache busting increases cost.

### 33.3 Context pressure relief (operational companion to §12.5)

§12.5 gives the *packing order* when you build the prompt. This section gives the *relief order* when the window fills. When context pressure rises, apply this order:

```text
1. Remove duplicate history / raw tool dumps already summarized
2. Compact old turns; retain open decisions, failing tests, safety constraints
3. Replace large docs with retrieved slices + citations
4. Defer rare tool schemas (tool search) while keeping the tool list stable
5. Escalate window size / model only if still failing evals
```

**Trap:** You add more examples as the first fix for agent failure. This often hurts attention and cache size. Prefer one clear counterexample and a clearer rule.

### 33.4 Tool design for Applications & Integration (~33% adjacency)

Integration stems often test tools because tools are how apps **do** things.

**Schema quality bar:**

| Good | Bad |
| --- | --- |
| `priority: enum["P0","P1","P2"]` | `priority: string` |
| Separate `read_issue` vs `close_issue` | `manage_issue` with freeform `action` |
| Idempotency key parameter on writes | Write tools that double-charge on retry |
| Error string: “status=409 already closed. Fetch before write” | Long stack traces |
| Stable tool order in requests | Shuffle tools per request for “A/B” |

**tool_choice map (conceptual):**

| Goal | Control |
| --- | --- |
| Model decides | auto |
| Must call some tool | any / required |
| Must call specific tool | tool name force |
| Ban tools this turn | none |

**Streaming + tools:** User-facing agents may stream text at the same time as tool-call prep. Fine-grained tool-input streaming (where available) helps UX for large JSON arguments. Exam care: tool rounds still need a complete `tool_result`. Then the model can continue grounded work.

**Parallel tools:** Independent reads yes. Dependent writes no. Encode this in the runtime. Do not encode it only in the prompt.

### 33.5 MCP from the agent designer’s chair

MCP is a **transport and discovery standard**. It is not a substitute for tool design quality.

| Concern | Prompt/tool answer | MCP ops answer |
| --- | --- | --- |
| What can the model call? | Tool descriptions | Server exposes tools/resources/prompts |
| Who authenticates? | N/A in schema | OAuth/tokens on remote servers. OS perms on stdio |
| What is enabled? | tool_choice / allowlists | Connector allow/deny. Code permission rules |
| Cache stability | Stable tool defs | Do not swap server toolsets without a plan |

**Messages API MCP connector (public theme):** Attach remote MCP servers so your app does not always run a custom MCP client. Still configure which tools are enabled. Treat results as untrusted.

### 33.6 Agent loop blueprint (primary study)

```text
while not done:
 observe state
 plan next action (model)
 if action is tool: check permission + budget → execute → append tool_result
 if action is final answer: validate → return
 if no progress OR budget exhausted OR safety trip: stop / escalate human
```

**Budgets to configure explicitly:** max turns, max dollars, max wall clock, max destructive tools, max tokens.

**Progress detectors:** new failing tests decline. The agent touches unique files without repeated conflict. You check checklist items. Do not use "the model said almost done."

**Planner–worker–verifier:** Separate roles when tasks are long or risky. The verifier must use tools (tests, linters, schema checks). Another LLM alone is a weak verifier.

### 33.7 Agent SDK / application framing (conceptual)

Public Agent SDK / Claude Code agent themes emphasize:

- You own orchestration, tool access, and permissions.
- Subagents can parallelize work under a lead.
- Skills package repeatable workflows. Tools package actions. Prompts package judgment.

**Exam distinction:**

| Artifact | Primary job |
| --- | --- |
| Prompt | Judgment & policy |
| Tool / MCP | Side effects & facts |
| Skill | Repeatable workflow UX (`/review-pr`) |
| Hook / permission | Hard enforcement |
| Eval harness | Truth about quality |

### 33.8 Decision table — workflow shapes

| Shape | When | Risk if misused |
| --- | --- | --- |
| Single-shot prompt | Narrow Q&A, no side effects | Invented facts |
| Tool-augmented turn | Need live data / actions | Over-broad tools |
| Deterministic workflow (DAG) | Known steps | Brittleness to novelty |
| Autonomous agent loop | Open-ended goals with tools | Runaway loops |
| Human-in-the-loop | Irreversible / ambiguous | Latency / ops cost |
| Multi-agent | Parallelizable subtasks | Coordination debt |

### 33.9 Production failure gallery (original)

1. **Over-broad tool** — `run(sql: string)` with production credentials. Split read/write. Bind params. Allowlist tables.
2. **Prompt-only safety** — "never delete" but Bash allow-all. Use permissions/hooks.
3. **Unbounded research agent** — no turn cap. Stop after N searches. The agent must cite.
4. **Cache thrash mode switch** — you remove write tools in "read mode." Keep tools. Force `tool_choice`/policy instead.
5. **Verifier = same model, same prompt** — this approves without a real check. Use independent checks + tools.
6. **Memory only in chat** — compact loses "do not email customer." Promote the rule to system/config.

### 33.10 Applications-flavored drills (Integration inside Chapter 02)

**Drill 1 — Schema as API.** Design a `create_refund` tool: required fields, enums, idempotency key, and which role may call it.

**Drill 2 — Streaming UX.** The user sees a partial answer then a tool call. Explain how your UI must handle interruption and retries.

**Drill 3 — Batch vs agent.** Classifying tickets is batch. Resolving a ticket with tools is a sync agent. Do not mix these two cases.

**Drill 4 — Cache-stable toolbox.** 40 tools. 5 used per turn. Keep the list stable. Defer schemas. Cache the prefix.

**Drill 5 — Stop reason reading.** `max_tokens` mid-JSON → raise cap / chunk. `tool_use` without results appended → bug in your loop.

### 33.11 Additional Q&A (Q36–Q45)

**Q36.** Why prefer a structured tool over "respond in JSON" for writing to a database?
**A36.** One place types, logs, gates, and validates the tool arguments. Prose JSON is easy to emit in part. It is hard to gate.

**Q37.** What is wrong with putting today's date in the system prompt each morning?
**A37.** It changes the cached prefix daily (or more). This destroys hit rates. Pass date in the user turn or a non-cached block.

**Q38.** Agent keeps calling the same failing tool. What is the fix?
**A38.** Return actionable errors. Add retry limits per tool. Add a progress detector. Maybe change the plan. Do not allow silent repeats.

**Q39.** When is `tool_choice: none` correct?
**A39.** Final answer turns, pure formatting, or when you must prevent actions while you still use the model for language.

**Q40.** MCP resource vs tool — exam one-liner?
**A40.** Tools are actions (often side-effecting). Read Resources -oriented data exposures. Do not hide deletes behind a "resource read."

**Q41.** How do soft and hard constraints differ in agent design?
**A41.** Soft = prompt/skill guidance. Hard = permissions, hooks, validators, budgets. Safety-critical rules need hard layers.

**Q42.** Multi-agent system thrashing ownership of one file. What is the fix?
**A42.** Assign disjoint workspaces/paths. The lead agent merges. Use locks or a single-writer rule.

**Q43.** Eval shows chat quality up but agent task success down after a prompt edit. How do you interpret this?
**A43.** You optimize the wrong contract. Restore agent-specific instructions. Measure task success, not eloquence.

**Q44.** Should the verifier model see the planner's chain-of-thought?
**A44.** Prefer verifying artifacts (diff, test output, schema). Sharing CoT can bias the judge.

**Q45.** Name three Integration knobs that affect tool-heavy apps.
**A45.** `tool_choice`, streaming (incl. tool input streaming where available), and prompt caching around stable tool definitions — plus timeouts/retries on tool execution.

### 33.12 If exam asks X, think Y (Chapter 02)

| If exam asks… | Think… |
| --- | --- |
| Model invents API payloads | Add/fix tools. Do not only ask in prose |
| Agent loops forever | Budgets + progress + stop conditions |
| Inconsistent JSON | Schema validation + tool args + repair once |
| Prompt injection via tool result | Untrusted data handling + allowlists + delimiters |
| Cache costs up after tool experiments | Stable tool list ordering. Defer load instead of swap |
| “Use MCP” as if MCP were enough | Still design tools, auth, allowlists |
| Need repeatable team workflow | Skill (+ hooks) not a longer CLAUDE.md alone |
| Irreversible action | Human approval hard gate |

### 33.13 Glossary addendum

| Term | Meaning |
| --- | --- |
| Fail closed | On validation/safety failure, deny action |
| Repair loop | Bounded retry to fix schema-invalid output |
| Progress detector | Signal that work advanced, not just tokens spent |
| Tool stub | Deferred schema placeholder for tool search |
| Lead agent | Coordinator of subagents |
| Soft constraint | Prompt-level guidance |
| Hard constraint | Runtime-enforced control |
| Grounding | Binding claims to tool/retrieved evidence |

### 33.14 Primary-study checklist (Chapter 02)

- [ ] I can write a 5-part system prompt contract without extra text.
- [ ] I can design read/write-separated tools with enums and idempotency.
- [ ] I can select `tool_choice` for extract-and-write vs advise-only.
- [ ] I can diagram an agent loop with three stop conditions.
- [ ] I can explain MCP vs plain tools in one sentence each for design and ops.
- [ ] I can list context packing steps before "buy bigger model."
- [ ] I know which failures need hard controls vs prompt tweaks.

---

## 34. Closing — Chapter 02 as primary study

Prompting, tools, and agents are how Applications actually behave. On a form weighted toward Integration, expect many items that look like API mechanics. Those items resolve to **contract design**. That design uses stable tools, validated outputs, bounded loops, and clear stop rules.


---

## 35. Long-form Integration lab — API mechanics for agent builders

Applications and Integration (~33%) rewards engineers who know how prompts and tools use the Messages API. This lab is original study material. Treat method names as conceptual. Check them against live docs.

### 30.1 Request construction checklist

1. **Auth:** API key header pattern (`x-api-key`) + API version header. Never embed keys in prompts or repos.
2. **Model pin:** Explicit ID from config.
3. **max_tokens:** Always set. Size it for the output contract (JSON/tool args vs essay).
4. **System:** Durable policy. No per-request volatility if you cache.
5. **Tools:** Full definitions or stubs+search. Deterministic order.
6. **Messages:** Alternating roles. Tool results as user-role content blocks of type tool_result.
7. **Metadata:** Trace IDs in your logs. They do not need to be in the model-visible prompt.
8. **Effort/thinking:** Explicit when defaults would drift.

### 30.2 Streaming consumer pattern

```text
open stream
on text delta → append to UI buffer
on tool_use start/delta → prepare tool UI / validate partial JSON if supported
on message end → inspect stop_reason
if tool_use → execute tools → new request with tool_results
if end_turn → validate final → return
if max_tokens → handle truncation (repair / continue / fail)
```

**Cancel:** If the user selects stop, abort the HTTP stream. Do not execute pending write tools without a fresh confirmation.

### 30.3 Batch path for offline agent variants

Not everything that uses tools must be online. Pattern:

- Online agent: sync + tools + human SLA.
- Offline variant: batch prompts that only need text or structured outputs without live tools. Or use a worker pool that runs toolful jobs asynchronously with their own queue. Do not confuse this with the Message Batches API limit (no streaming).

**Exam precision:** Message Batches ≠ generic "async workers." Batches are Anthropic's discounted async Messages processing with specific constraints.

### 30.4 Schema evolution without breaking prod

| Change | Safe approach |
| --- | --- |
| Add optional tool field | Versioned schema. Accept old+new during rollout |
| Rename field | Dual-read period. Update evals first |
| Remove tool | Deprecate. Keep stub returning “removed” until clients migrate |
| Tighten enum | Ship validator first. Measure rejection rate |

### 30.5 Additional vignettes

**V1.** Mobile chat shows blank 8s then a large block of text. → Stream. Reduce TTFT via cache warm + faster tier/effort.
**V2.** Agent SDK app retries a payment tool three times after timeouts. → Idempotency keys + safe retry classification.
**V3.** Evaluator says prompts improved but API bill doubled. → Check turns, output length, cache hit rate, escalation rate.
**V4.** Model calls deprecated MCP tool. → Deny list + server version pin + tool search refresh policy.

### 30.6 Extra Q&A (Q46–Q50)

**Q46.** Where do tool results go in the Messages transcript?
**A46.** Back into the conversation as tool result content associated with the tool_use ids. Then you call the model again.

**Q47.** Why log `stop_reason`?
**A47.** It distinguishes success, tool rounds, truncation, and refusals. This is essential for debugging and evals.

**Q48.** Can Message Batches call your local MCP stdio server?
**A48.** Batches run in Anthropic's async infrastructure for model calls. Your local stdio tools are not available there automatically. Toolful local work needs your workers.

**Q49.** What is a good first streaming test in CI?
**A49.** Assert you handle an interrupted stream and you do not execute a half-confirmed write tool.

**Q50.** How does prompt caching interact with growing multi-turn tool transcripts?
**A50.** Automatic caching can advance breakpoints as history grows. Still keep the early system+tools prefix stable so the expensive shared portion stays available for cache hits.

---

## 36. Chapter 02 revision sheet (primary)

**Contracts over vague style.** Specify role, tools policy, output shape, escalation.
**Tools for facts/actions. Language for judgment.**
**Agents = loops with budgets.**
**MCP = ops + transport. You still design tools.**
**Stable prefixes help you answer Integration cost items.**
**Hard controls for irreversible actions.**
