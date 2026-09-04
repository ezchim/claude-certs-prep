---
title: Building with the Claude API
---

# Building with the Claude API — Exam Prep Study Notes

> **Disclaimer:** These are **original study notes** written for exam / certification prep. They are **not** official Anthropic course material, transcripts, or verbatim lesson content. Align topics to the public outline of [Building with the Claude API](https://academy.claude.com/courses/building-with-the-claude-api). Always verify current API details in the official Anthropic docs ([platform.claude.com / docs.anthropic.com](https://docs.anthropic.com)).

**Primary sources to cross-check (not copied here):** Messages API, tool use, streaming, structured outputs, prompt caching, Message Batches, vision/files, rate limits/errors.

---

## Overview

The public course page positions this as a production-oriented path for software engineers who already know Python and JSON. Stated outcomes include: authenticate and call the Anthropic API; run single- and multi-turn chats; use system prompts, temperature, streaming, and structured outputs; evaluate prompts systematically; call tools (custom and built-in); build RAG pipelines; use extended features (thinking, vision, PDFs, citations, caching); integrate MCP; and design agent workflows (parallelize, chain, route).

**Course shape (public modules):** Accessing Claude with the API → Prompt engineering → Tool use → RAG and agentic search → Model Context Protocol → Anthropic apps (Claude Code, Computer Use) → Agents and workflows.

**How to use these notes:** Treat each section as a flashcard cluster. After reading, answer the Self-check Q&A without looking back. Prefer official docs for parameter names when APIs change.

**Exam posture:** Graders care whether you can *operate* the Messages loop correctly (roles, stop reasons, tool_result ids, stream assembly), diagnose cost/latency (caching, batches, model choice), and choose workflow vs. agent patterns deliberately—not whether you memorized marketing slogans.

---

## Key concepts (cheat sheet)

| Idea | Remember |
|------|----------|
| Messages API | Conversation turns with roles `user` / `assistant`; **no** `system` role in messages — use top-level `system` |
| Auth | API key via env / secret store; send required version headers; never commit keys |
| `max_tokens` | Required upper bound on generated tokens; truncation vs. complete answers |
| Temperature | Higher → more varied; lower → more deterministic; pair with evals |
| Streaming | SSE event stream; assemble deltas; special handling for tool `input_json_delta` |
| Stop reasons | `end_turn`, `max_tokens`, `tool_use`, `stop_sequence`, `pause_turn`, `refusal` — each needs a different next action |
| Tools | Model emits `tool_use`; your app runs the tool; return `tool_result` in next turn |
| Parallel tools | Multiple `tool_use` blocks in one assistant turn; return one result per id |
| Structured outputs | Schema-constrained JSON (and/or strict tools) for reliable pipelines |
| RAG | Retrieve relevant chunks → inject into context → generate with citations/grounds |
| Prompt caching | Cache stable prefixes (tools → system → messages) to cut latency/cost |
| Batches | Async Message Batches for large offline jobs; typically discounted vs. sync |
| Agents | Workflows vs. free-form agents; parallel / chain / route patterns |
| MCP | Standardize tools/resources/prompts across hosts (see companion MCP notes) |

> **Currency notes (2026-08):** (1) *Temperature* — sampling knobs (`temperature`/`top_p`/`top_k`) are **removed on current frontier models** (Opus 4.7+, Opus 5, Sonnet 5, Fable 5 return a 400); they remain valid on 4.6-and-older. Determinism advice shifts to schemas/structured outputs. (2) *System role* — "no `system` role in `messages`" remains the foundations-exam answer, but the newest Opus/Fable-tier models do accept **mid-conversation** `{"role": "system"}` messages appended to `messages` (cache-friendly operator channel); `messages[0]` still can't be one.

---

## Deep notes by topic

### 1. Accessing Claude with the API

#### Authentication & setup

- Create an API key in the Anthropic Console; store it in environment variables or a secrets manager (`ANTHROPIC_API_KEY`), never in source control or client-side apps.
- Use the official SDKs (Python / TypeScript) when possible: they handle headers, retries, and streaming helpers.
- HTTP clients must send the API key and an API version header (commonly `anthropic-version`). Treat version headers as part of the contract: wrong/missing headers fail loudly.
- Typical request fields: `model`, `max_tokens`, `messages`, optional `system`, `temperature`, `tools`, `stream`, `tool_choice`, caching controls, thinking config, etc.
- Choose models by capability vs. cost/latency. For cert prep: know *why* you’d pick a faster/cheaper model for classification or routing and a stronger model for hard reasoning or tool orchestration.
- **Server-side vs. client-side keys:** browser apps must never hold long-lived API keys. Put Claude calls behind your backend.

**Original example — minimal mental request shape**

```text
POST /v1/messages
Headers: x-api-key, anthropic-version, content-type: application/json
Body: {
 model: "<current-model-id>",
 max_tokens: 1024,
 system: "You are a careful support triage bot. Reply in JSON.",
 messages: [
 { role: "user", content: "Order #4421 never arrived." }
 ]
}
```

#### Models & selection heuristics

| Workload | Prefer | Why |
|----------|--------|-----|
| Intent routing / cheap classify | Smaller / faster model | High volume, low ambiguity |
| Hard reasoning, long tool chains | Stronger model | Fewer tool mistakes, better planning |
| Batch offline labeling | Strong enough + Batches API | Cost + throughput |
| Interactive UI chat | Mid/strong + streaming | Latency perception + quality |

Exam trap: “always use the biggest model.” Correct answer is **right-size** for the task and measure with evals.

#### Messages & multi-turn context

- Each message has a `role` and `content` (string or content blocks).
- Alternate user/assistant turns. Your app owns history: trim, summarize, or drop old turns to stay within context limits.
- Content can be multimodal: text plus images or document inputs where the API supports them.
- Pitfall: stuffing a “system” message into `messages` with role `system` — Messages API expects the top-level `system` parameter instead. *(Current note: newest Opus/Fable-tier models accept mid-conversation system messages appended to `messages`; the exam-level rule — system instructions go in top-level `system` — still stands.)*
- When continuing after tools, append the full assistant turn (including `tool_use` blocks) and a user turn that holds `tool_result` blocks. Do not invent fake assistant text in place of tool results.

**History hygiene patterns**

1. **Sliding window:** keep last N turns; drop older user/assistant pairs.
2. **Summarize-and-replace:** periodically replace early turns with a compact summary message.
3. **Pin critical facts:** put durable policy in `system`; put session facts in a short pinned user/assistant preface.
4. **Never silently rewrite tool transcripts** if you still need auditability—store raw tool logs offline even if you summarize for the model.

#### System prompts

- System text sets role, rules, output format, safety boundaries, and tool-use policy.
- Keep system prompts stable when using prompt caching; put volatile user data in the user message.
- Prefer explicit instructions (“always return JSON matching schema X”) over vague vibes (“be helpful”).
- Structure long systems with labeled sections (XML-style tags are a common pattern): `<role>`, `<rules>`, `<output_format>`, `<examples>`.
- Tool policy belongs here too: when to call tools, when to ask the user, when to refuse.

#### Temperature, max_tokens, and output control

- Temperature (and related sampling knobs) trade creativity for consistency — **on models where they still exist**. Current frontier models (Opus 4.7+, Opus 5, Sonnet 5, Fable 5) have **removed** `temperature`/`top_p`/`top_k` (sending them returns a 400); they remain valid on 4.6-and-older models.
- For grading, extraction, and tool argument generation, prefer **schemas / structured outputs** as the primary consistency lever (plus lower temperature only on older models that support it).
- Always set `max_tokens` thoughtfully: too low truncates mid-JSON; too high wastes budget if you don’t need long answers.
- `max_tokens` is an **upper bound**, not a target. Models often stop earlier with `end_turn`.
- Custom `stop_sequences` can halt generation on sentinel strings; then `stop_reason` becomes `stop_sequence`. Use sparingly—schemas usually beat brittle stop strings for structured work.

#### Streaming (SSE)

- Set `stream: true` to receive server-sent events as the model generates.
- Typical event flow: `message_start` → (`content_block_start` → deltas → `content_block_stop`)* → `message_delta` → `message_stop` (plus `ping` / `error` events as applicable).
- For UX: render text deltas immediately; buffer tool JSON until the block completes (or follow fine-grained tool streaming rules carefully).
- Exam tip: streaming does **not** change the final semantic result; it changes delivery and how you assemble the message.
- Tool streaming detail: `tool_use` blocks often start with empty `input: {}`; arguments arrive as `input_json_delta` / `partial_json` fragments. Concatenate fragments, then parse after `content_block_stop`.
- Thinking streams (when enabled) may emit `thinking_delta` and a `signature_delta` before the thinking block closes—treat signatures as integrity metadata, not user-facing prose.
- SDKs usually provide helpers (`text_stream`, `get_final_message` / `finalMessage`) so you don’t hand-parse SSE unless building a raw HTTP client.
- Handle unknown event types gracefully (versioning may add events).

#### Stop reasons (must-know table)

| `stop_reason` | Meaning | Typical next action |
|---------------|---------|---------------------|
| `end_turn` | Natural completion | Show / use the response |
| `max_tokens` | Hit your output cap | Raise `max_tokens` or continue generation |
| `tool_use` | Model wants client tools | Execute tools; send `tool_result`s |
| `stop_sequence` | Hit your custom stop string | Handle sentinel; maybe continue |
| `pause_turn` | Server-side loop paused (e.g. long server tools / thinking) | Continue turn per docs (often re-send assistant content) |
| `refusal` | Policy refusal | Surface safely; don’t retry blindly with same prompt |

Exam trap: treating every unfinished answer as a model failure. First check `stop_reason` and `usage`.

#### Structured outputs

- Use schema-constrained JSON so downstream code can parse without brittle regex.
- Strict tool schemas similarly reduce invalid tool arguments.
- Combine structured outputs with evals: assert shape *and* semantic correctness.
- Prefer official structured-output / JSON-schema features when available over “please reply as JSON” alone.
- If schema fails in production, log raw text, fail closed for critical paths, and fall back to a repair prompt only when safe.

#### Errors, retries, and rate limits

- Expect HTTP-class failures: auth (401), bad request (400), not found (404), rate limit (429), overloaded / capacity (e.g. 529-class), 5xx.
- On 429: respect `retry-after` when present; use exponential backoff + jitter; isolate per-tenant budgets if you multi-tenant.
- Distinguish **request rate** vs. **token rate** limits (RPM vs. input/output tokens per minute). A “small” chat app can still blow ITPM with huge RAG contexts.
- Idempotency mindset: retries must not double-charge side effects—make tool execution idempotent or dedupe by tool_use id.
- Streaming errors can arrive as SSE `error` events mid-stream; your client must abort cleanly and decide whether to resume.

#### Message Batches (async scale)

- Message Batches API submits many Messages-style requests asynchronously—good for offline eval sets, bulk classification, overnight enrichment.
- Typically priced lower than synchronous Messages for the same tokens; results are retrieved later (not a live chat UX).
- Each item needs a `custom_id` so you can join results back to source rows.
- Still subject to batch-specific rate / queue limits; requests can expire if not processed in time (docs cite long outer windows such as ~24h processing bounds—verify current numbers before exams).
- Feature parity is broad (tools, vision, caching often supported), but cache **pre-warm with `max_tokens: 0`** is a poor fit inside batches because ephemeral cache may expire before follow-ups run.
- Design pattern: sync path for interactive users; batch path for backfills and eval sweeps.

---

### 2. Prompt engineering & evaluation

**Techniques that show up on exams**

- **Clear directives:** say what to do, what not to do, and the output contract.
- **XML / tagged sections:** separate instructions, context, examples, and user input so the model can attend to the right block.
- **Few-shot examples:** show input→output pairs that match the desired style and edge cases.
- **Chain of thought (when appropriate):** ask for reasoned steps for hard problems; for structured extraction, prefer schemas over open-ended rambling.
- **Role prompting:** put expertise and constraints in `system`.
- **Delimiter hygiene:** never concatenate untrusted user text next to instructions without clear boundaries—reduces instruction-injection risk.

**Original mini-pattern (study only)**

```text
system:
 <role>You extract invoice fields.</role>
 <rules>If a field is missing, use null. Never invent tax IDs.</rules>
 <output>JSON object with keys vendor, total, currency, due_date.</output>

user:
 <document>
... raw OCR text...
 </document>
```

**Evaluation mindset**

- Don’t “eyeball” prompts in production. Build a small dataset of cases (happy path, edge cases, adversarial).
- Automate grading: exact match, rubric scores, or model-as-judge with a fixed rubric.
- Iterate: change one prompt variable at a time; track regressions.
- Generate synthetic test cases, then hand-spot-check for realism.
- Separate **prompt regressions** from **model version** regressions—pin model ids in eval runs.
- Track cost and latency alongside quality; a 2% quality bump that triples tokens may be a product loss.

**Eval loop (memorize)**

1. Define task success criteria in writing.
2. Collect 30–200 labeled cases (start small).
3. Freeze a grader (script or rubric).
4. Run baseline → change one thing → compare.
5. Promote only if quality ↑ and cost/latency acceptable.

---

### 3. Tool use with Claude

**Core loop**

1. Send user message + tool definitions (`name`, `description`, JSON `input_schema`).
2. Model may return one or more `tool_use` blocks (id, name, input).
3. Your application executes the tools (and enforces authz!).
4. Append `tool_result` blocks tied to each `tool_use` id; continue the conversation until the model answers in plain text (or calls more tools).

**Design tips**

- Descriptions are part of the prompt: be precise about when to call the tool and what each field means.
- Prefer many small, composable tools over one mega-tool with ambiguous arguments.
- Handle parallel/batch tool calls: execute safely (idempotency, timeouts, partial failure).
- Built-in / server tools (e.g. web search, code execution, computer use variants depending on product surface) differ from client-executed tools: know which side runs the code.
- `tool_choice` can nudge or force tool use (`auto` / `any` / specific tool / none—confirm exact enum in current docs).
- For strict argument validity, enable schema-strict tool modes when offered.

**Parallel tools checklist**

- One assistant message may contain multiple `tool_use` blocks.
- Return **one** `tool_result` per id in the following user message (can be same user message content array).
- Failures: still return a result with an error payload so the model can recover—don’t omit the id.
- Concurrency: only parallelize tools that are safe together (no conflicting writes).

**Client tools vs. server tools**

| Kind | Who executes? | Your job |
|------|---------------|----------|
| Client tool | Your app / MCP server / backend | Run function; send `tool_result` |
| Server tool | Anthropic infrastructure | Often appears as server tool use + result blocks in-stream; you may still need to continue on `pause_turn` |

**Common failures**

- Forgetting to return results for every `tool_use` id.
- Executing tools the user is not allowed to run.
- Feeding huge raw tool dumps back without summarizing when context is tight.
- Treating hallucinated tool names as real — validate against your registry.
- Parsing partial JSON mid-stream before `content_block_stop`.
- Infinite tool loops with no max-iteration / budget guard.

**Original example — weather tool round-trip (conceptual)**

```text
Assistant content:
 [tool_use id=toolu_1 name=get_weather input={location:"Singapore"}]

Your next user content:
 [tool_result tool_use_id=toolu_1 content="31°C, thunderstorms"]

Then model produces end_turn text for the user.
```

---

### 4. RAG and agentic search

**Why RAG:** Context windows are finite; private corpora change; you want grounded answers with sources.

**Pipeline building blocks**

- **Chunking:** split documents by size/structure (headings, paragraphs); overlap can help continuity.
- **Embeddings + vector search:** semantic similarity for fuzzy queries.
- **BM25 / lexical search:** strong for exact terms, IDs, rare tokens.
- **Hybrid retrieval:** combine lexical + dense; often add **reranking**.
- **Contextual retrieval:** enrich chunks with document-level context so retrieval is less ambiguous.
- **Multi-index:** separate indexes by corpus or modality; route queries.

**Injection pattern**

```text
system: Answer ONLY using <sources>. If missing, say you don't know.
user:
 <question>...</question>
 <sources>
 [1]...chunk...
 [2]...chunk...
 </sources>
```

**Agentic search pattern**

- Instead of one-shot retrieve→answer, let the model (or a planner) issue follow-up searches, refine queries, and stop when evidence is enough.
- Always keep a citation trail: which chunk IDs supported which claims.
- Budget the loop: max searches, max tokens, max wall time.

**Evaluation split (exam favorite)**

| Layer | Metric ideas |
|-------|----------------|
| Retrieval | Recall@k, MRR, citation hit-rate |
| Generation | Faithfulness, answer correctness, format validity |
| System | Latency p95, cost / query, tool error rate |

**Pitfalls**

- Chunks too large (noise) or too small (lost meaning).
- Retrieving relevant text but burying it under a weak prompt (“use the context if helpful”).
- No evaluation of retrieval hit-rate separately from generation quality.
- Updating docs in the CMS but forgetting to re-index.
- Letting the model cite sources that weren’t retrieved (hallucinated citations).

---

### 5. Extended Claude features

- **Extended thinking / reasoning modes:** useful for hard multi-step problems; understand cost/latency tradeoffs and that thinking content may be separable from the final answer depending on API surface.
- **Images & PDFs:** pass as content blocks; ask for structured extraction; watch token/cost implications of large documents.
- **Vision inputs:** common source types include base64, URL, or Files API references; media types typically include jpeg/png/gif/webp—confirm current limits.
- **Documents / PDFs:** prefer upload-once via Files API and reference by id for repeated analysis; page limits and token costs matter for exam “what breaks at scale?” questions.
- **Citations:** when available, prefer features that ground quotes to source spans for trustworthy UX.
- **Prompt caching:** mark stable prefixes; measure cache hit rates; don’t put per-request secrets or unique IDs in the cached prefix if that defeats reuse.

#### Prompt caching — operational detail

- Cacheable prefix order to remember: **tools → system → messages** (and included content blocks up to a cache breakpoint).
- Place `cache_control` on the **last shared block** you expect to reuse—not on a unique placeholder user message if that would key the cache poorly.
- Monitor usage fields such as cache creation vs. cache read tokens (names like `cache_creation_input_tokens` / `cache_read_input_tokens` in responses).
- TTL / ephemeral behavior: know that cache entries expire; high-churn prefixes waste money on repeated writes.
- Pre-warming exists in some forms (including `max_tokens: 0` patterns for cache population)—use only when follow-up traffic will hit soon enough.

#### Files, vision, and documents — design checklist

1. Resize/compress huge images before upload when quality allows.
2. Prefer file ids over repeatedly shipping megabyte base64 payloads.
3. For PDFs: extract text when layout is simple; use document/vision pathways when layout matters.
4. Ask for structured fields + page references when building audit trails.
5. Redact secrets in screenshots before they enter logs or prompts.

---

### 6. MCP (course module — high level)

This API course also covers building MCP servers/clients so tools and resources are reusable across hosts. For depth, study the dedicated MCP notes (`04-introduction-to-mcp.md`). Exam-relevant mapping:

- Host app ↔ MCP client ↔ MCP server.
- Primitives: **tools** (actions), **resources** (read-only data), **prompts** (reusable instruction templates).
- Prefer MCP when many apps need the same integrations; prefer inline tools for one-off prototypes.
- Lifecycle: initialize / negotiate capabilities → list primitives → call tools / read resources → shut down cleanly.
- Security: user consent, least privilege, and clear boundaries still apply when tools arrive via MCP instead of hard-coded JSON schemas.

---

### 7. Anthropic apps: Claude Code & Computer Use

- **Claude Code:** agentic coding assistant in the terminal/IDE; accelerates repo tasks; often integrates MCP for project tools.
- **Computer Use:** model-driven UI automation (screenshots + actions); powerful and risky — sandbox, least privilege, human approval for sensitive steps.
- Cert framing: know *use cases* and *safety boundaries*, not product marketing copy.
- Map to API knowledge: Computer Use is an extreme form of tool use + vision; Claude Code is an agent host with permissions, hooks, and MCP connectors.

---

### 8. Agents and workflows

**Workflow vs. agent**

- **Workflow:** you define the graph (steps, branches); the model fills in nodes. Deterministic control, easier to test.
- **Agent:** model decides next actions with tools until a stop condition. Flexible, harder to evaluate.

**Core patterns**

- **Parallelization:** fan-out independent subtasks; merge results.
- **Chaining:** output of step N becomes input to N+1 (extract → transform → validate).
- **Routing:** classify intent, then send to specialized prompt/model/tool path.
- **Debugging:** log every tool call, intermediate state, token usage; add timeouts and max-iteration caps to prevent runaway loops.

**Comparison table**

| Pattern | Control | Best for | Failure mode |
|---------|---------|----------|--------------|
| Chain | High | ETL-like LLM steps | Error cascades if no validators |
| Parallel | Medium | Map-reduce style tasks | Merge conflicts / inconsistent schemas |
| Route | High | Multi-domain assistants | Mis-route → wrong specialist |
| Free agent | Low | Open-ended research | Loops, spend spikes, hard audits |

**Production guardrails (write these on the exam)**

1. Max tool iterations / max wall clock.
2. Per-tool authz and allowlists.
3. Human approval for irreversible actions.
4. Structured logs (request id, tool id, tokens, stop_reason).
5. Dead-letter queue for failed tool side effects.
6. Eval suite before enabling new tools.

**Original routing sketch**

```text
User message
 → cheap classifier (billing | technical | cancel)
 → billing prompt + CRM tools
 → technical prompt + docs RAG
 → cancel workflow (deterministic steps + confirmation)
```

---

### 9. End-to-end request anatomy (study diagram)

Think in layers so exam scenarios don’t blur together:

1. **Transport/auth layer** — HTTPS, API key, version header, optional beta headers.
2. **Sampling layer** — model id, temperature, `max_tokens`, stop sequences, thinking config.
3. **Context layer** — `system`, prior `messages`, retrieved RAG chunks, images/files.
4. **Capability layer** — `tools` / MCP-discovered tools, `tool_choice`, structured output schema.
5. **Delivery layer** — `stream` on/off; client event assembly; timeouts.
6. **Control-plane layer** — your retries, rate-limit budgets, agent iteration caps, authz.

If something fails, locate the layer first. A parse error in the client is not a “model quality” problem; a `tool_use` without results is not a temperature problem.

### 10. Multi-turn conversation patterns worth memorizing

**A. Plain chat**
User → Assistant (`end_turn`) → User → …

**B. Tool-interrupted turn**
User → Assistant (`tool_use`) → User(`tool_result`) → Assistant (`end_turn` or more tools)

**C. Parallel tools**
User → Assistant(`tool_use` A + `tool_use` B) → User(`tool_result` A + `tool_result` B) → …

**D. RAG-augmented turn**
Retriever(user query) → User(question + sources) → Assistant grounded answer

**E. Routed workflow**
Classifier → branch-specific system/tool set → specialized answer

**F. Batch offline**
Enqueue N independent Messages params → poll/fetch results by `custom_id`

Original practice cue: given a bug report (“assistant asked for weather but never answered”), map it to pattern B and ask which hop broke (model didn’t call tool / app didn’t execute / app didn’t return result / second model call missing).

### 11. Prompt caching worked example (conceptual)

Suppose every support ticket shares:

- the same 3 tools (CRM lookup, refund calculator, policy search),
- the same 2,000-token policy system prompt,
- but a unique ticket body.

**Good:** cache tools + system (and maybe a static policy resource). Put ticket text only in the varying user message after the breakpoint.

**Bad:** put `Ticket #{{id}}` inside the cached system string—each ticket writes a new prefix.

**Measurement habit:** log cache-write tokens vs. cache-read tokens. A feature that “uses caching” but shows near-zero reads is theater.

**Invalidation intuition:** changing tool schemas, system text, thinking/effort settings, or breakpoint placement can miss the prior cache entry. Keep cached bytes byte-stable across requests you want to hit.

### 12. Structured outputs & tools — choosing a contract

| Need | Prefer | Notes |
|------|--------|-------|
| Final answer must be parseable JSON | Structured outputs / JSON schema mode | Best for API responses to clients |
| Model must call your function | Tools + input_schema (+ strict if available) | Best for side effects |
| Both | Tools for actions + structured final text/schema | Common agent pattern |
| Soft formatting only | Prompt instructions | Fragile; use only for prototypes |

Exam trap: “tools are only for side effects, never for JSON.” Historically, tools were *also* used to force JSON shapes; modern structured-output features may be cleaner for final answers—know both stories and check current docs.

### 13. Rate limits, errors, and resilience playbook

**Classify the failure**

| Signal | Likely class | First response |
|--------|--------------|----------------|
| 401/403 | Auth/permission | Fix key/scopes; don’t retry blindly |
| 400 | Request shape / params | Fix payload; log body (redacted) |
| 404 | Bad path / missing resource | Fix endpoint or file id |
| 429 | Rate limit | Backoff; shed load; reduce tokens |
| 5xx / overloaded | Capacity | Retry with jitter; fallback model if policy allows |
| SSE `error` event | Mid-stream failure | Abort UI cleanly; optional resume strategy |

**Resilience patterns**

- Token bucket per tenant.
- Queue interactive traffic separately from batch traffic.
- Circuit breaker when error rate spikes.
- Fallback to a smaller model for non-critical paths.
- Idempotency keys around refunds/emails triggered by tools.

### 14. RAG production checklist (beyond the happy path)

1. **Corpus governance:** who can add docs? PII scrubbing?
2. **Chunking strategy recorded:** size, overlap, splitter version—so you can reproduce indexes.
3. **Embedding model version pinned.**
4. **Hybrid weights tuned** on a labeled query set, not vibes.
5. **Rerank top-n** before prompting.
6. **Prompt forces citations** from provided ids only.
7. **Refuse when retrieval empty.**
8. **Freshness:** re-index on publish; show doc updated-at to users when relevant.
9. **Poisoning awareness:** untrusted user-uploaded docs can instruct the model—sandbox and label them.
10. **Cost guard:** cap retrieved tokens; don’t dump entire manuals.

### 15. Agent debugging scenarios (exam-style)

**Scenario 1 — Spend spike.** Logs show 40 tool calls/turn. Fix: max iterations, tool allowlist, require plan-before-act for expensive tools.

**Scenario 2 — Flaky JSON.** `stop_reason=max_tokens` mid-object. Fix: raise cap, shrink schema, or continue; validate before DB write.

**Scenario 3 — Wrong tool chosen.** Descriptions overlap (“search” vs “search_users”). Fix: rename, narrow descriptions, add “when NOT to use” notes, reduce tool count.

**Scenario 4 — Grounding misses.** Answers sound confident without sources. Fix: retrieval eval; strengthen “sources only” system rule; add citation feature if available.

**Scenario 5 — Cache never hits.** Breakpoint on unique user turn. Fix: move breakpoint to shared system/tools.

### 16. Security & compliance notes for API builders

- Treat prompts + tool results as **data stores**: they may contain secrets; scrub logs.
- Separate **model trust** from **tool trust**: even if the model asks to `delete_all_users`, your authz layer must refuse.
- Prompt injection via retrieved docs or web-tool results is real—prefer allowlisted tools and confirmations for irreversible actions.
- For Computer Use / browser tools: dedicated VM, no production credentials, record sessions for audit when required.
- Data retention: know whether your org requires zero-retention options or regional processing (cloud courses cover Bedrock/Vertex residency separately).

### 17. Mapping course modules → skills you must demo

| Module | Skill to demo from memory |
|--------|---------------------------|
| Accessing API | Auth, messages, system, stream, structured out |
| Prompt engineering | Tagged prompts + eval loop |
| Tool use | Full client tool loop + parallel + server-tool awareness |
| RAG / agentic search | Hybrid retrieve + cite + iterate |
| MCP | When to externalize tools/resources/prompts |
| Claude Code / Computer Use | Use cases + safety boundaries |
| Agents / workflows | Chain / parallel / route + guardrails |


## Comparison tables (exam quick hits)

### Sync Messages vs. Batches

| | Sync Messages | Message Batches |
|--|---------------|-----------------|
| Latency | Seconds | Minutes–hours |
| UX | Interactive | Offline / jobs |
| Cost | Standard | Typically discounted |
| Streaming | Yes | No live SSE to user |
| Best for | Chat, copilots | Evals, backfills |

### Streaming vs. non-streaming

| | Non-stream | Stream SSE |
|--|------------|------------|
| Time-to-first-token | Worse | Better |
| Client complexity | Lower | Higher (assemble events) |
| Final message | Direct JSON | Accumulated equivalent |
| Tool JSON | Complete object | Often partial deltas first |

### Inline tools vs. MCP tools

| | Inline tool defs | MCP |
|--|------------------|-----|
| Setup speed | Fastest | More moving parts |
| Reuse across hosts | Copy/paste schemas | Shared servers |
| Ops | App owns everything | Server processes / remote endpoints |
| When | Single app prototype | Platform / many clients |

---




## Exam tips

1. **Messages vs. Completions mental model:** Claude’s public API path is Messages-centric; system is top-level.
2. **Tool loop completeness:** every `tool_use` needs a matching `tool_result` (or a deliberate abort strategy).
3. **Streaming assembly:** text deltas vs. partial JSON for tools — don’t `json.loads` mid-stream unless you know it’s complete.
4. **Evals beat vibes:** expect questions on datasets, graders, and regression when changing prompts.
5. **RAG is two systems:** retrieval quality and generation quality fail differently — diagnose separately.
6. **Caching & cost:** know *what* to cache (stable prefix) and *why* (latency + spend).
7. **Safety:** keys, tool authz, computer-use sandboxing, and PII in logs are fair game for “what’s wrong with this design?” questions.
8. **MCP pointer:** if a question mentions standardized reusable tools across hosts, think MCP — not ad-hoc duplicate schemas.
9. **Always read `stop_reason` before retrying.**
10. **Batches ≠ streaming chat.** Different product shape.
11. **Parallel tools:** match ids, not names alone.
12. **Rate limits:** backoff + jitter; separate RPM from token budgets.

---

## Exam traps (common wrong answers)

| Trap | Why it’s wrong | Better answer |
|------|----------------|---------------|
| Put `role: "system"` inside `messages` | Wrong shape for Messages API | Top-level `system` |
| Assume stream changes answer quality | Delivery ≠ sampling semantics | Same request params → same final content intent |
| Drop failed tool ids | Model waits forever / errors | Return error `tool_result` for that id |
| Cache the unique user question as prefix | Destroys hit rate | Cache tools/system/static docs |
| One mega-tool `do_anything` | Ambiguous args, hard authz | Small tools with clear schemas |
| Only measure final answer accuracy for RAG | Hides retrieval failures | Measure retrieval and generation separately |
| No max agent iterations | Cost/runaway risk | Hard caps + kill switch |
| Put API key in a mobile app | Key theft | Backend proxy |

*(Row 1 currency note: mid-conversation `{"role": "system"}` messages are now accepted on the newest Opus/Fable-tier models — but "top-level `system`" remains the correct foundations-exam answer for where system instructions live.)*

---

## Self-check Q&A

**Q1. Where do you put system instructions in the Messages API?** 
**A:** In the top-level `system` parameter — not as a message with role `system`.

**Q2. What must your app do after Claude returns `tool_use` blocks?** 
**A:** Execute authorized tools, then continue the conversation with matching `tool_result` content blocks (by tool use id).

**Q3. Name one reason to stream responses.** 
**A:** Lower perceived latency / progressive UI updates; final assembled message should match non-streaming semantics.

**Q4. Why combine BM25 with embeddings in RAG?** 
**A:** Lexical search catches exact tokens/IDs; dense search catches semantic paraphrases; hybrid + rerank often beats either alone.

**Q5. What’s a prompt evaluation workflow in one sentence?** 
**A:** Fixed test cases + automated scoring so prompt changes are measurable and regressions are caught.

**Q6. When is a deterministic workflow better than an open-ended agent?** 
**A:** When steps are known, compliance/auditability matters, or you need reliable SLAs and cheaper debugging.

**Q7. What belongs in a prompt cache prefix?** 
**A:** Stable content reused across requests (long system prompts, tool defs, static docs) — not unique per-user payloads that destroy hit rate.

**Q8. How does MCP relate to the API course?** 
**A:** It standardizes how hosts discover and call external tools/resources/prompts instead of reinventing schemas per app.

**Q9. What does `stop_reason: "max_tokens"` imply?** 
**A:** Output hit your cap; raise `max_tokens` or continue generation—don’t treat the truncated JSON as complete.

**Q10. What is `input_json_delta` for?** 
**A:** Streaming partial JSON fragments that assemble into a tool’s `input` object after the content block stops.

**Q11. Client tool vs. server tool—who runs the code?** 
**A:** Client tools run in your environment; server tools run on Anthropic’s side (your app may still need to continue turns).

**Q12. Why require `max_tokens`?** 
**A:** It bounds generation cost/length; models may stop earlier, but you always set the ceiling.

**Q13. Name two rate-limit dimensions.** 
**A:** Requests per minute and tokens per minute (input/output); 429s need backoff.

**Q14. When would you use Message Batches?** 
**A:** Large asynchronous jobs (evals, backfills) where interactive latency isn’t required and discounted pricing helps.

**Q15. How do you handle parallel `tool_use` ids?** 
**A:** Execute each authorized tool and return a `tool_result` for every id in the follow-up user turn.

**Q16. What’s a safe temperature default for extraction?** 
**A:** On older models (4.6 and earlier): low temperature plus a schema. On current frontier models sampling params are removed — the schema/structured output **is** the determinism lever.

**Q17. How should vision images be supplied?** 
**A:** As image content blocks via supported sources (e.g. base64, URL, or Files API id)—not as pretend text paths.

**Q18. What is contextual retrieval trying to fix?** 
**A:** Ambiguous chunks that lack document-level context, which hurts embedding match quality.

**Q19. Why pin model ids in evals?** 
**A:** So prompt changes aren’t confounded with silent model upgrades.

---

# CCAR-F Domain 4 gap card (added 2026-08-23)

> Compact supplement for **exam guide tasks 4.3/4.4/4.6** specifics the notes above touch only lightly. Original synthesis against the official task statements.

## G1. Schema design against fabrication (task 4.3)

- **Nullable/optional fields prevent fabrication:** if a source document may lack a field, a **required** schema field forces the model to invent a value to satisfy the schema. Make it **optional/nullable** so the model can return null — the exam's canonical "model fabricates values" fix.
- **Extensible enums:** add `"unclear"` for ambiguous cases and `"other"` **plus a detail string field** for categories outside the enum — closed enums force misclassification.
- **Strict schemas via tool_use eliminate *syntax* errors only** — never *semantic* errors (line items that don't sum to the stated total, correct values in wrong fields). Semantic checks are validation-layer work (G2).
- Include **format-normalization rules in the prompt** alongside strict output schemas when source formatting varies.

## G2. Validation, retry, and feedback loops (task 4.4)

- **Retry-with-error-feedback:** on validation failure, send a follow-up containing **the original document, the failed extraction, and the specific validation errors** — the model self-corrects against named errors far better than against a bare "try again."
- **Limits of retry (tested):** retries fix **format/structural** errors; they are **ineffective when the information is absent from the source** (or lives in a document you didn't provide). Diagnose *which* failure class you have before burning retries.
- **Self-correction flows:** extract `calculated_total` alongside `stated_total` to flag discrepancies; add `conflict_detected` booleans for inconsistent source data.
- **`detected_pattern` fields:** tag each finding with the code construct/pattern that triggered it, so dismissed findings can be analyzed systematically for false-positive patterns (feeds the 4.1 precision loop).

## G3. Multi-instance & multi-pass review (task 4.6)

- **Self-review limitation:** a model that generated code **retains its reasoning context** and is unlikely to question its own decisions in the same session. Neither "review your work carefully" instructions nor extended thinking substitute for independence.
- **Independent review instance:** a second Claude instance **without the generator's reasoning context** catches subtle issues self-review misses (pairs with the CI session-isolation point in file 05's supplement).
- **Multi-pass review:** split large reviews into **per-file local passes + a cross-file integration pass** — attention dilution, not context size, causes inconsistent single-pass reviews (see 08 §6).
- **Calibrated routing:** have the reviewer self-report **confidence alongside each finding** to enable review routing — then calibrate those scores before trusting them (see 09 §5).

## G4. Gap-card Q&A

**GQ1.** The model fills in an invoice field the document doesn't contain. Schema fix?
**A.** Make the field optional/nullable — required fields force fabrication.

**GQ2.** What errors do strict tool-use schemas eliminate, and what survives?
**A.** JSON syntax errors are eliminated; semantic errors (wrong sums, misplaced values) survive and need validation logic.

**GQ3.** Describe retry-with-error-feedback.
**A.** Follow-up request = original document + failed extraction + the specific validation errors, guiding targeted self-correction.

**GQ4.** When will retries never fix an extraction?
**A.** When the required information is absent from the provided source — that's a data problem, not a format problem.

**GQ5.** Why is an independent instance better than "review your own code thoroughly"?
**A.** The generator retains its reasoning context and won't question its own decisions; independence removes that bias.

**GQ6.** Purpose of a `detected_pattern` field in review findings?
**A.** Enables systematic analysis of which constructs trigger dismissed findings — the raw material for cutting false-positive categories.

**Q20. What is a routing workflow?** 
**A:** Classify the request, then send it to a specialized prompt/model/tool path.

**Q21. What should you log for an agent turn?** 
**A:** Message ids, stop_reason, tool names/ids/inputs (redacted), results status, token usage, latency.

**Q22. Cache prefix order to remember?** 
**A:** Tools, then system, then messages—up to the cache breakpoint.

**Q23. What is `pause_turn` signaling?** 
**A:** A long-running server-side turn paused; continue according to docs rather than treating it as a hard failure.

**Q24. Why are tool descriptions “prompt engineering”?** 
**A:** The model uses them to decide *when* and *how* to call tools—vague descriptions cause wrong calls.

**Q25. What’s wrong with returning a huge SQL dump as tool_result?** 
**A:** Context bloat and distraction; summarize or page results, keep raw data server-side.

**Q26. How do refusals differ from empty answers?** 
**A:** `stop_reason: "refusal"` indicates policy refusal—handle with safe UX, don’t infinite-retry the same content.

**Q27. Name one Computer Use safety control.** 
**A:** Sandbox / least privilege / human approval for sensitive UI actions.

**Q28. What’s the difference between chaining and parallelization?** 
**A:** Chaining is sequential dependencies; parallelization fans out independent work then merges.

**Q29. Why separate retrieval metrics from answer metrics?** 
**A:** Bad answers may be good writing over bad chunks—or good chunks with a weak prompt; fixes differ.

**Q30. What’s the first debugging step when tool use “hangs”?** 
**A:** Check whether every `tool_use` id received a `tool_result` and inspect `stop_reason` / iteration caps.

---

## Quick review checklist

- [ ] I can sketch a Messages request with `model`, `max_tokens`, `system`, `messages`.
- [ ] I never put `system` as a message role.
- [ ] I can name ≥4 stop reasons and the next action for each.
- [ ] I can assemble SSE text deltas and tool `partial_json` safely.
- [ ] I can complete a tool loop including parallel tools and error results.
- [ ] I can explain structured outputs vs. “please return JSON”.
- [ ] I can design a tiny eval set and grader.
- [ ] I can describe hybrid RAG + why to measure retrieval separately.
- [ ] I know what to put in a prompt-cache prefix (and what not to).
- [ ] I know when to use Batches vs. sync Messages.
- [ ] I can compare workflow vs. agent and give parallel/chain/route examples.
- [ ] I can map MCP host/client/server at a high level.
- [ ] I treat API keys, tool authz, and computer-use sandboxing as exam-critical safety topics.

---

## Glossary

| Term | Plain meaning |
|------|----------------|
| Messages API | Chat-style API with user/assistant turns and top-level system |
| Content block | Typed piece of message content (text, image, tool_use, tool_result, …) |
| SSE | Server-Sent Events stream used for incremental responses |
| `stop_reason` | Why generation stopped |
| Tool use | Model requests a function call via `tool_use` blocks |
| Server tool | Tool executed by the platform rather than your client |
| Structured outputs | Schema-constrained model output for parsers |
| Prompt caching | Reusing a stable prompt prefix to save tokens/latency |
| Message Batches | Async bulk Messages processing API |
| RAG | Retrieve documents, then generate grounded answers |
| BM25 | Classic lexical ranking function |
| Reranker | Second-stage model/function that reorders retrieved hits |
| Agentic search | Multi-step retrieve/refine loop driven by the model |
| Workflow | Developer-defined graph of LLM/tool steps |
| Agent | Model-directed loop over tools until a stop condition |
| MCP | Open protocol for reusable tools/resources/prompts across hosts |
| Computer Use | Vision + UI action tools for controlling a desktop/browser |
| ITPM / OTPM | Input/output tokens-per-minute style rate limits |
| `custom_id` | Caller-assigned id joining a batch request to its result |

---

## Quick lab checklist (practice without copying course labs)

Use this as a private drill list while studying official docs/SDKs:

1. **Hello Messages:** one-shot user message → print assistant text; then add a second user turn with prior assistant message preserved.
2. **System ablations:** same user question with three different system prompts; note tone and constraint adherence.
3. **Stream assemble:** concatenate text deltas; assert equality with non-streamed output for the same seed/settings when possible.
4. **Tool round-trip:** fake weather tool; verify ids match on `tool_result`; force a parallel double call if the model emits two tools.
5. **Structured parse:** schema for `{ "sentiment": enum, "confidence": number }`; fail the build if parse throws.
6. **Mini-RAG:** three markdown files → chunk → embed or BM25 → answer with “sources:” lines.
7. **Cache sanity:** put a long static system prompt behind cache controls; compare latency on request 1 vs. request 2.
8. **Agent budget:** max 5 tool iterations; on loop, return “budget exceeded” instead of hanging.
9. **Stop-reason drill:** intentionally set tiny `max_tokens` and confirm you detect truncation.
10. **Batch dry-run design:** sketch 100 classification rows with `custom_id`s and a results joiner (even without calling the API).
11. **429 simulator:** write a retry helper with exponential backoff + jitter and a max-retry cap.
12. **Vision extract:** one screenshot/receipt image → structured JSON fields; verify you didn’t hardcode fake OCR.

These drills reinforce exam mechanics; they are not substitutes for Anthropic’s graded quizzes.

---

## Source URLs (verify before exam day)

- Course outline: https://academy.claude.com/courses/building-with-the-claude-api
- Anthropic docs hub: https://docs.anthropic.com (also platform.claude.com/docs)
- Topics to re-open live: Messages, streaming, stop reasons, tool use, structured outputs, prompt caching, batch processing, vision/files, rate limits/errors, MCP overview

*End of notes. Cross-check parameters against current Anthropic platform docs before an exam sitting.*
