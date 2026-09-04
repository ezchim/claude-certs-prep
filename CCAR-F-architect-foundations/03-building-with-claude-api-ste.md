---
title: Building with the Claude API — Exam Prep Study Notes — Simplified Technical English
disclaimer: Original study notes — independent and not official course content
approx_length: STE edition (ASD-STE100) — primary study
updated: 2026-08-30
---

# Building with the Claude API — Exam Prep Study Notes

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, MCP, prompting, caching, effort, p95) are exceptions and stay as written. Model IDs and prices change. Learn the decision rules. Check the current docs before the exam.

> **Disclaimer:** These are **original study notes** for exam and certification prep. They are **not** official Anthropic course material. They are **not** transcripts. They are **not** verbatim lesson content. Align the topics to the public outline of [Building with the Claude API](https://academy.claude.com/courses/building-with-the-claude-api). Check current API details in the official Anthropic docs ([platform.claude.com / docs.anthropic.com](https://docs.anthropic.com)).

**Primary sources to check (this file does not copy them):** Messages API, tool use, streaming, structured outputs, prompt caching, Message Batches, vision/files, rate limits/errors.

---

## Overview

The public course page is a production path for software engineers who already know Python and JSON. Stated outcomes include these skills. You authenticate and call the Anthropic API. You run single-turn chats and multi-turn chats. You use system prompts, temperature, streaming, and structured outputs. You evaluate prompts in a systematic way. You call tools (custom and built-in). You build RAG pipelines. You use extended features (thinking, vision, PDFs, citations, caching). You integrate MCP. You design agent workflows (parallelize, chain, route).

**Course shape (public modules):** Accessing Claude with the API → Prompt engineering → Tool use → RAG. Also, agentic search → Model Context Protocol → Anthropic apps (Claude Code, Computer Use) → Agents and workflows.

**How to use these notes:** Treat each section as a flashcard cluster. After you read a section, answer the Self-check Q&A. Do not look back. Prefer official docs for parameter names when APIs change.

**Exam posture:** Graders check if you can operate the Messages loop correctly. This includes roles, stop reasons, tool_result ids, and stream assembly. They also check if you diagnose cost and latency. Caching, batches, and model choice matter here. They check if you select workflow vs. agent patterns with intent. They do not grade marketing slogans.

---

## Key concepts (cheat sheet)

| Idea | Remember |
|------|----------|
| Messages API | Conversation turns with roles `user` / `assistant`. There is **no** `system` role in messages. Use top-level `system`. |
| Auth | API key via env / secret store. Send required version headers. Never commit keys. |
| `max_tokens` | Required upper bound on generated tokens. Truncation vs. complete answers. |
| Temperature | Higher → more varied. Lower → more deterministic. Pair with evals. |
| Streaming | SSE event stream. Assemble deltas. Special handling for tool `input_json_delta`. |
| Stop reasons | `end_turn`, `max_tokens`, `tool_use`, `stop_sequence`, `pause_turn`, `refusal`. Each needs a different next action. |
| Tools | Model emits `tool_use`. Your app runs the tool. Return `tool_result` in the next turn. |
| Parallel tools | Multiple `tool_use` blocks in one assistant turn. Return one result per id. |
| Structured outputs | Schema-constrained JSON (and/or strict tools) for reliable pipelines. |
| RAG | Retrieve relevant chunks → inject into context → generate with citations/grounds. |
| Prompt caching | Cache stable prefixes (tools → system → messages) to cut latency and cost. |
| Batches | Async Message Batches for large offline jobs. Typically discounted vs. sync. |
| Agents | Workflows vs. free-form agents. Parallel / chain / route patterns. |
| MCP | Standardize tools/resources/prompts across hosts (see companion MCP notes). |

> **Current notes (2026-08):** (1) *Temperature* — Current frontier models remove sampling controls (`temperature`/`top_p`/`top_k`). Opus 4.7+, Opus 5, Sonnet 5, and Fable 5 return a 400. They stay valid on 4.6-and-older. Determinism advice moves to schemas and structured outputs. (2) *System role* — "no `system` role in `messages`" stays the foundations-exam answer. The newest Opus/Fable-tier models do accept **mid-conversation** `{"role": "system"}` messages appended to `messages`. This is a cache-friendly operator channel. `messages[0]` still cannot be a system message.

---

## Deep notes by topic

### 1. Accessing Claude with the API

#### Authentication & setup

- Create an API key in the Anthropic Console. Store it in environment variables or a secrets manager (`ANTHROPIC_API_KEY`). Never put it in source control. Never put it in client-side apps.
- Use the official SDKs (Python / TypeScript) when you can. They handle headers, retries, and streaming helpers.
- HTTP clients must send the API key and an API version header (commonly `anthropic-version`). Treat version headers as part of the contract. Wrong or missing headers fail with a clear error.
- Typical request fields: `model`, `max_tokens`, `messages`, optional `system`, `temperature`, `tools`, `stream`, `tool_choice`, caching controls, thinking config, and similar fields.
- Select models by capability vs. cost and latency. For cert prep, know why you select a faster or cheaper model for classification or routing. Know why you pick a stronger model for hard reasoning or tool orchestration.
- **Server-side vs. client-side keys:** Browser apps must never hold long-lived API keys. Put Claude calls behind your backend.

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

Common exam error: “always use the model with the most capability.” The correct answer is to match the model size to the task. Measure with evals.

#### Messages & multi-turn context

- Each message has a `role` and `content` (string or content blocks).
- Alternate user/assistant turns. Your app owns history. Trim, summarize, or drop old turns to stay within context limits.
- Content can be multimodal: text plus images or document inputs where the API supports them.
- Common error: put a “system” message into `messages` with role `system`. The Messages API expects the top-level `system` parameter instead. *(Current note: newest Opus/Fable-tier models accept mid-conversation system messages appended to `messages`. The exam-level rule still stands. Put system instructions in top-level `system`.)*
- When you continue after tools, append the full assistant turn (including `tool_use` blocks). Then add a user turn that holds `tool_result` blocks. Do not invent fake assistant text in place of tool results.

**History hygiene patterns**

1. **Sliding window:** keep last N turns. Drop older user/assistant pairs.
2. **Summarize-and-replace:** replace early turns with a compact summary message at set intervals.
3. **Pin critical facts:** put durable policy in `system`. Put session facts in a short pinned user/assistant preface.
4. **Never silently rewrite tool transcripts** if you still need auditability. Store raw tool logs offline even if you summarize for the model.

#### System prompts

- System text sets role, rules, output format, safety boundaries, and tool-use policy.
- Keep system prompts stable when you use prompt caching. Put volatile user data in the user message.
- Prefer explicit instructions ("always return JSON that matches schema X"). Do not use vague lines (“be helpful”).
- Structure long systems with labeled sections (XML-style tags are a common pattern): `<role>`, `<rules>`, `<output_format>`, `<examples>`.
- Tool policy belongs here too: when to call tools, when to ask the user, when to refuse.

#### Temperature, max_tokens, and output control

- Temperature (and related sampling controls) trades creativity for consistency — **on models where they still exist**. Current frontier models (Opus 4.7+, Opus 5, Sonnet 5, Fable 5) have **removed** `temperature`/`top_p`/`top_k`. If you send them, you get a 400. They stay valid on 4.6-and-older models.
- For grading, extraction, and tool argument generation, prefer schemas and structured outputs as the primary consistency control. Add lower temperature only on older models that support it.
- Always set `max_tokens` with care. Too low truncates mid-JSON. Too high wastes budget if you do not need long answers.
- `max_tokens` is an **upper bound**, not a target. Models often stop earlier with `end_turn`.
- Custom `stop_sequences` can halt generation on sentinel strings. Then `stop_reason` becomes `stop_sequence`. Use them sparingly. Schemas are usually better than brittle stop strings for structured work.

#### Streaming (SSE)

- Set `stream: true` to receive server-sent events as the model generates.
- Typical event flow: `message_start` → (`content_block_start` → deltas → `content_block_stop`)* → `message_delta` → `message_stop` (plus `ping` / `error` events as applicable).
- For UX: render text deltas immediately. Buffer tool JSON until the block completes. Or follow fine-grained tool streaming rules with care.
- Exam tip: streaming does **not** change the final semantic result. It changes delivery. It changes how you assemble the message.
- Tool streaming detail: `tool_use` blocks often start with empty `input: {}`. Arguments arrive as `input_json_delta` / `partial_json` fragments. Concatenate fragments. Then parse after `content_block_stop`.
- Thinking streams (when enabled) may emit `thinking_delta` and a `signature_delta` before the thinking block closes. Treat signatures as integrity metadata. Do not treat them as user-facing prose.
- SDKs usually provide helpers (`text_stream`, `get_final_message` / `finalMessage`). You do not hand-parse SSE unless you build a raw HTTP client.
- Handle unknown event types with care (versioning may add events).

#### Stop reasons (must-know table)

| `stop_reason` | Meaning | Typical next action |
|---------------|---------|---------------------|
| `end_turn` | Natural completion | Show / use the response |
| `max_tokens` | Hit your output cap | Raise `max_tokens` or continue generation |
| `tool_use` | Model wants client tools | Execute tools. Send `tool_result`s |
| `stop_sequence` | Hit your custom stop string | Handle sentinel. Maybe continue |
| `pause_turn` | Server-side loop paused (e.g. long server tools / thinking) | Continue turn per docs (often re-send assistant content) |
| `refusal` | Policy refusal | Surface safely. Do not retry without a check with the same prompt |

Common exam error: treat every unfinished answer as a model failure. First check `stop_reason` and `usage`.

#### Structured outputs

- Use schema-constrained JSON so downstream code can parse without brittle regex.
- Strict tool schemas similarly reduce invalid tool arguments.
- Combine structured outputs with evals. Assert shape *and* semantic correctness.
- Prefer official structured-output / JSON-schema features when they are available. Do not rely on “please reply as JSON” alone.
- If schema fails in production, log raw text. Fail closed for critical paths. Fall back to a repair prompt only when that path is safe.

#### Errors, retries, and rate limits

- Expect HTTP-class failures: auth (401), bad request (400), not found (404), rate limit (429), overloaded / capacity (e.g. 529-class), 5xx.
- On 429: respect `retry-after` when it is present. Use exponential backoff + jitter. Isolate per-tenant budgets if you multi-tenant.
- Distinguish **request rate** vs. **token rate** limits (RPM vs. input/output tokens per minute). A “small” chat app can still exceed ITPM with huge RAG contexts.
- Idempotency rule: retries must not double-charge side effects. Make tool execution idempotent. Or dedupe by tool_use id.
- Streaming errors can arrive as SSE `error` events mid-stream. Your client must abort cleanly. Then decide whether to resume.

#### Message Batches (async scale)

- Message Batches API submits many Messages-style requests asynchronously. It is good for offline eval sets, bulk classification, and overnight enrichment.
- Batches typically cost less than synchronous Messages for the same tokens. You retrieve results later. This is not a live chat UX.
- Each item needs a `custom_id` so you can join results back to source rows.
- Still subject to batch-specific rate / queue limits. Requests can expire if the system does not process them in time. Docs cite long outer windows such as ~24h processing bounds. Check current numbers before exams.
- Feature parity is broad (tools, vision, caching often supported). Cache **pre-warm with `max_tokens: 0`** is a poor fit inside batches. Ephemeral cache may expire before follow-ups run.
- Design pattern: sync path for interactive users. Batch path for backfills and eval sweeps.

---

### 2. Prompt engineering & evaluation

**Techniques that show up on exams**

- **Clear directives:** say what to do, what not to do, and the output contract.
- **XML / tagged sections:** separate instructions, context, examples, and user input so the model can attend to the right block.
- **Few-shot examples:** show input→output pairs that match the desired style and edge cases.
- **Chain of thought (when appropriate):** ask for reasoned steps for hard problems. For structured extraction, prefer schemas over open-ended rambling.
- **Role prompting:** put expertise and constraints in `system`.
- **Delimiter hygiene:** never concatenate untrusted user text next to instructions without clear boundaries. This reduces instruction-injection risk.

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

**Evaluation approach**

- Do not judge prompts only by inspection in production. Build a small dataset of cases (standard path, edge cases, adversarial).
- Automate grading: exact match, rubric scores, or model-as-judge with a fixed rubric.
- Iterate: change one prompt variable at a time. Track regressions.
- Generate synthetic test cases. Then hand-spot-check for realism.
- Separate **prompt regressions** from **model version** regressions. Pin model ids in eval runs.
- Track cost and latency alongside quality. A 2% quality gain that triples tokens may be a product loss.

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
4. Append `tool_result` blocks tied to each `tool_use` id. Continue the conversation until the model answers in plain text (or calls more tools).

**Design tips**

- Descriptions are part of the prompt. Be precise about when to call the tool and what each field means.
- Prefer many small, composable tools over one oversized tool with ambiguous arguments.
- Handle parallel/batch tool calls. Execute safely (idempotency, timeouts, partial failure).
- Built-in / server tools (e.g. web search, code execution, computer-use variants that depend on the product surface) differ from client-executed tools. Know which side runs the code.
- `tool_choice` can guide or force tool use (`auto` / `any` / specific tool / none — confirm exact enum in current docs).
- For strict argument validity, enable schema-strict tool modes when the API offers them.

**Parallel tools checklist**

- One assistant message may contain multiple `tool_use` blocks.
- Return **one** `tool_result` per id in the following user message (can be same user message content array).
- Failures: still return a result with an error payload so the model can recover. Do not omit the id.
- Concurrency: only parallelize tools that are safe together (no conflicting writes).

**Client tools vs. server tools**

| Kind | Who executes? | Your job |
|------|---------------|----------|
| Client tool | Your app / MCP server / backend | Run function. Send `tool_result` |
| Server tool | Anthropic infrastructure | Often appears as server tool use + result blocks in-stream. You may still need to continue on `pause_turn` |

**Common failures**

- Forgetting to return results for every `tool_use` id.
- Executing tools the user is not allowed to run.
- Feeding huge raw tool dumps back without a summary when context is tight.
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

**Why RAG:** Context windows are finite. Private corpora change. You want grounded answers with sources.

**Pipeline building blocks**

- **Chunking:** split documents by size/structure (headings, paragraphs). Overlap can help continuity.
- **Embeddings + vector search:** semantic similarity for fuzzy queries.
- **BM25 / lexical search:** strong for exact terms, IDs, rare tokens.
- **Hybrid retrieval:** combine lexical + dense. Often add **reranking**.
- **Contextual retrieval:** enrich chunks with document-level context so retrieval is less ambiguous.
- **Multi-index:** separate indexes by corpus or modality. Route queries.

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

- Instead of one-shot retrieve→answer, let the model (or a planner) issue follow-up searches. Refine queries. Stop when evidence is enough.
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
- You retrieve relevant text, but a weak prompt does not use it well.
- No evaluation of retrieval hit-rate separately from generation quality.
- Updating docs in the CMS but forgetting to re-index.
- Do not retrieve Letting the model cite sources that (hallucinated citations).

---

### 5. Extended Claude features

- **Extended thinking / reasoning modes:** useful for hard multi-step problems. Understand cost/latency tradeoffs. Whether thinking content is separable from the final answer depends on the API surface.
- **Images & PDFs:** pass as content blocks. Ask for structured extraction. Watch token/cost implications of large documents.
- **Vision inputs:** common source types include base64, URL, or Files API references. Media types typically include jpeg/png/gif/webp. Confirm current limits.
- **Documents / PDFs:** prefer upload-once via Files API and reference by id for repeated analysis. Page limits and token costs matter for exam “what breaks at scale?” questions.
- **Citations:** when available, prefer features that ground quotes to source spans for trustworthy UX.
- **Prompt caching:** mark stable prefixes. Measure cache hit rates. Do not put per-request secrets or unique IDs in the cached prefix if that defeats reuse.

#### Prompt caching — operational detail

- Cacheable prefix order to remember: **tools → system → messages** (and included content blocks up to a cache breakpoint).
- Place `cache_control` on the **last shared block** you expect to reuse. Do not put it on a unique placeholder user message if that would key the cache poorly.
- Monitor usage fields such as cache creation vs. cache read tokens (names like `cache_creation_input_tokens` / `cache_read_input_tokens` in responses).
- TTL / ephemeral behavior: know that cache entries expire. High-churn prefixes waste money on repeated writes.
- Pre-warming exists in some forms (including `max_tokens: 0` patterns for cache population). Use it only when follow-up traffic will hit soon enough.

#### Files, vision, and documents — design checklist

1. Resize/compress huge images before upload when quality allows.
2. Prefer file ids over repeatedly shipping megabyte base64 payloads.
3. For PDFs: extract text when layout is simple. Use document/vision pathways when layout matters.
4. Ask for structured fields + page references when you build audit trails.
5. Redact secrets in screenshots before they enter logs or prompts.

---

### 6. MCP (course module — high level)

This API course also covers building MCP servers/clients so tools and resources are reusable across hosts. For depth, study the dedicated MCP notes (`04-introduction-to-mcp.md`). Exam-relevant mapping:

- Host app ↔ MCP client ↔ MCP server.
- Primitives: **tools** (actions), **resources** (read-only data), **prompts** (reusable instruction templates).
- Prefer MCP when many apps need the same integrations. Prefer inline tools for single-use prototypes.
- Lifecycle: initialize / negotiate capabilities → list primitives → call tools / read resources → stop cleanly.
- Security: user consent, least privilege, and clear boundaries still apply when tools arrive via MCP instead of hard-coded JSON schemas.

---

### 7. Anthropic apps: Claude Code & Computer Use

- **Claude Code:** agentic coding assistant in the terminal/IDE. It accelerates repo tasks. It often integrates MCP for project tools.
- **Computer Use:** model-driven UI automation (screenshots + actions). It is powerful and risky. Sandbox. Use least privilege. Get human approval for sensitive steps.
- Cert framing: know *use cases* and *safety boundaries*. Do not memorize product marketing copy.
- Map to API knowledge: Computer Use is an extreme form of tool use + vision. Claude Code is an agent host with permissions, hooks, and MCP connectors.

---

### 8. Agents and workflows

**Workflow vs. agent**

- **Workflow:** you define the graph (steps, branches). The model fills in nodes. Deterministic control. Easier to test.
- **Agent:** model decides next actions with tools until a stop condition. Flexible. Harder to evaluate.

**Core patterns**

- **Parallelization:** fan-out independent subtasks. Merge results.
- **Chaining:** output of step N becomes input to N+1 (extract → transform → validate).
- **Routing:** classify intent, then send to a specialized prompt/model/tool path.
- **Debugging:** log every tool call, intermediate state, and token usage. Add timeouts and max-iteration caps to prevent unbounded loops.

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
6. Eval suite before you enable new tools.

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

Think in layers so exam scenarios do not blur together:

1. **Transport/auth layer** — HTTPS, API key, version header, optional beta headers.
2. **Sampling layer** — model id, temperature, `max_tokens`, stop sequences, thinking config.
3. **Context layer** — `system`, prior `messages`, retrieved RAG chunks, images/files.
4. **Capability layer** — `tools` / MCP-discovered tools, `tool_choice`, structured output schema.
5. **Delivery layer** — `stream` on/off. Client event assembly. Timeouts.
6. **Control-plane layer** — your retries, rate-limit budgets, agent iteration caps, authz.

If something fails, locate the layer first. A parse error in the client is not a “model quality” problem. A `tool_use` without results is not a temperature problem.

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

Original practice cue: given a bug report (“assistant asked for weather but never answered”), map it to pattern B. Ask which hop broke. Options: model did not call tool / app did not execute / app did not return result / second model call missing.

### 11. Prompt caching worked example (conceptual)

Suppose every support ticket shares:

- the same 3 tools (CRM lookup, refund calculator, policy search),
- the same 2,000-token policy system prompt,
- but a unique ticket body.

**Good:** cache tools + system (and maybe a static policy resource). Put ticket text only in the varying user message after the breakpoint.

**Bad:** put `Ticket #{{id}}` inside the cached system string. Each ticket writes a new prefix.

**Measurement habit:** log cache-write tokens vs. cache-read tokens. A feature that "uses caching" but shows near-zero cache reads gives no real benefit.

**Invalidation intuition:** a change to tool schemas, system text, thinking/effort settings, or breakpoint placement can miss the prior cache entry. Keep cached bytes byte-stable across requests you want to hit.

### 12. Structured outputs & tools — choosing a contract

| Need | Prefer | Notes |
|------|--------|-------|
| Final answer must be parseable JSON | Structured outputs / JSON schema mode | Best for API responses to clients |
| Model must call your function | Tools + input_schema (+ strict if available) | Best for side effects |
| Both | Tools for actions + structured final text/schema | Common agent pattern |
| Soft formatting only | Prompt instructions | Fragile. Use only for prototypes |

Common exam error: “tools are only for side effects, never for JSON.” Historically, developers also used tools to force JSON shapes. Modern structured-output features may be cleaner for final answers. Know both stories. Check current docs.

### 13. Rate limits, errors, and resilience playbook

**Classify the failure**

| Signal | Likely class | First response |
|--------|--------------|----------------|
| 401/403 | Auth/permission | Fix key/scopes. Do not retry without a check |
| 400 | Request shape / params | Fix payload. Log body (redacted) |
| 404 | Bad path / missing resource | Fix endpoint or file id |
| 429 | Rate limit | Backoff. Shed load. Reduce tokens |
| 5xx / overloaded | Capacity | Retry with jitter. Fallback model if policy allows |
| SSE `error` event | Mid-stream failure | Abort UI cleanly. Optional resume strategy |

**Resilience patterns**

- Token bucket per tenant.
- Queue interactive traffic separately from batch traffic.
- Circuit breaker when error rate spikes.
- Fallback to a smaller model for non-critical paths.
- Idempotency keys around refunds/emails triggered by tools.

### 14. RAG production checklist (beyond the happy path)

1. **Corpus governance:** who can add docs? PII scrubbing?
2. **Chunking strategy recorded:** size, overlap, splitter version — so you can reproduce indexes.
3. **Embedding model version pinned.**
4. **Hybrid weights tuned** on a labeled query set, not vague preference.
5. **Rerank top-n** before prompting.
6. **Prompt forces citations** from provided ids only.
7. **Refuse when retrieval empty.**
8. **Freshness:** re-index on publish. Show doc updated-at to users when relevant.
9. **Poisoning awareness:** untrusted user-uploaded docs can instruct the model. Sandbox them. Label them.
10. **Cost guard:** cap retrieved tokens. Do not dump entire manuals.

### 15. Agent debugging scenarios (exam-style)

**Scenario 1 — Spend spike.** Logs show 40 tool calls/turn. Fix: max iterations, tool allowlist, require plan-before-act for expensive tools.

**Scenario 2 — Fails intermittently JSON.** `stop_reason=max_tokens` mid-object. Fix: raise cap, shrink schema, or continue. Validate before DB write.

**Scenario 3 — Wrong tool chosen.** Descriptions overlap (“search” vs “search_users”). Fix: rename, narrow descriptions, add “when NOT to use” notes, reduce tool count.

**Scenario 4 — Grounding misses.** Answers sound confident without sources. Fix: retrieval eval. Strengthen “sources only” system rule. Add citation feature if available.

**Scenario 5 — Cache never hits.** Breakpoint on unique user turn. Fix: move breakpoint to shared system/tools.

### 16. Security & compliance notes for API builders

- Treat prompts + tool results as **data stores**. They may contain secrets. Scrub logs.
- Separate **model trust** from **tool trust**. Even if the model asks to `delete_all_users`, your authz layer must refuse.
- Prompt injection via retrieved docs or web-tool results is real. Prefer allowlisted tools. Confirm irreversible actions.
- For Computer Use / browser tools: dedicated VM, no production credentials. Record sessions for audit when required.
- Data retention: know whether your org requires zero-retention options or regional processing. Cloud courses cover Bedrock/Vertex residency separately.

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

1. **Messages vs. Completions working model:** Claude’s public API path is Messages-centric. System is top-level.
2. **Tool loop completeness:** every `tool_use` needs a matching `tool_result` (or a deliberate abort strategy).
3. **Streaming assembly:** text deltas vs. partial JSON for tools. Do not `json.loads` mid-stream unless you know it is complete.
4. **Evals are better than vague judgment:** expect questions on datasets, graders, and regression when you change prompts.
5. **RAG is two systems:** retrieval quality and generation quality fail differently. Diagnose them separately.
6. **Caching & cost:** know *what* to cache (stable prefix) and *why* (latency + spend).
7. **Safety:** keys, tool authz, computer-use sandboxing, and PII in logs are valid exam topics for “what is wrong with this design?” questions.
8. **MCP pointer:** if a question mentions standardized reusable tools across hosts, think MCP. Do not think ad-hoc duplicate schemas.
9. **Always read `stop_reason` before retrying.**
10. **Batches ≠ streaming chat.** Different product shape.
11. **Parallel tools:** match ids, not names alone.
12. **Rate limits:** backoff + jitter. Separate RPM from token budgets.

---

## Exam traps (common wrong answers)

| Trap | Why it is wrong | Better answer |
|------|----------------|---------------|
| Put `role: "system"` inside `messages` | Wrong shape for Messages API | Top-level `system` |
| Assume stream changes answer quality | Delivery ≠ sampling semantics | Same request params → same final content intent |
| Drop failed tool ids | Model waits forever / errors | Return error `tool_result` for that id |
| Cache the unique user question as prefix | Destroys hit rate | Cache tools/system/static docs |
| One oversized `do_anything` tool | Ambiguous args, hard authz | Small tools with clear schemas |
| Only measure final answer accuracy for RAG | Hides retrieval failures | Measure retrieval and generation separately |
| No max agent iterations | Cost/unbounded risk | Hard caps + stop control |
| Put API key in a mobile app | Key theft | Backend proxy |

*(Row 1 current note: mid-conversation `{"role": "system"}` The newest Opus/Fable-tier models now accept mid-conversation system messages. "Top-level `system`" remains the correct foundations-exam answer for where system instructions live.)*

---

## Self-check Q&A

**Q1. Where do you put system instructions in the Messages API?** 
**A:** In the top-level `system` parameter — not as a message with role `system`.

**Q2. What must your app do after Claude returns `tool_use` blocks?** 
**A:** Execute authorized tools. Then continue the conversation with matching `tool_result` content blocks (by tool use id).

**Q3. Name one reason to stream responses.** 
**A:** Lower perceived latency / progressive UI updates. The final assembled message should match non-streaming semantics.

**Q4. Why combine BM25 with embeddings in RAG?** 
**A:** Lexical search catches exact tokens/IDs. Dense search catches semantic paraphrases. Hybrid plus reranking is often better than either alone.

**Q5. What is a prompt evaluation workflow in one sentence?** 
**A:** Fixed test cases + automated scoring so prompt changes are measurable and regressions are caught.

**Q6. When is a deterministic workflow better than an open-ended agent?** 
**A:** When steps are known, compliance/auditability matters, or you need reliable SLAs and cheaper debugging.

**Q7. What belongs in a prompt cache prefix?** 
**A:** Stable content reused across requests (long system prompts, tool defs, static docs). Not unique per-user payloads that destroy hit rate.

**Q8. How does MCP relate to the API course?** 
**A:** It standardizes how hosts discover and call external tools/resources/prompts instead of reinventing schemas per app.

**Q9. What does `stop_reason: "max_tokens"` imply?** 
**A:** Output hit your cap. Raise `max_tokens` or continue generation. Do not treat the truncated JSON as complete.

**Q10. What is `input_json_delta` for?** 
**A:** Streaming partial JSON fragments that assemble into a tool’s `input` object after the content block stops.

**Q11. Client tool vs. server tool—who runs the code?** 
**A:** Client tools run in your environment. Server tools run on Anthropic’s side (your app may still need to continue turns).

**Q12. Why require `max_tokens`?** 
**A:** It bounds generation cost/length. Models may stop earlier, but you always set the ceiling.

**Q13. Name two rate-limit dimensions.** 
**A:** Requests per minute and tokens per minute (input/output). 429s need backoff.

**Q14. When would you use Message Batches?** 
**A:** You do not need Large asynchronous jobs (evals, backfills) where interactive latency and discounted pricing helps.

**Q15. How do you handle parallel `tool_use` ids?** 
**A:** Execute each authorized tool. Return a `tool_result` for every id in the follow-up user turn.

**Q16. What is a safe temperature default for extraction?** 
**A:** On older models (4.6 and earlier): low temperature plus a schema. On current frontier models the API removes sampling params. The schema or structured output is the determinism control.

**Q17. How should vision images be supplied?** 
**A:** As image content blocks via supported sources (e.g. base64, URL, or Files API id). Not as pretend text paths.

**Q18. What is contextual retrieval trying to fix?** 
**A:** Ambiguous chunks that lack document-level context, which hurts embedding match quality.

**Q19. Why pin model ids in evals?** 
**A:** So prompt changes are not confounded with silent model upgrades.

---

# CCAR-F Domain 4 gap card (added 2026-08-23)

> Compact supplement for **exam guide tasks 4.3/4.4/4.6** specifics the notes above touch only lightly. Original synthesis against the official task statements.

## G1. Schema design against fabrication (task 4.3)

- **Nullable/optional fields prevent fabrication:** if a source document may lack a field, a **required** schema field forces the model to invent a value. Make it **optional/nullable** so the model can return null. This is the exam's canonical "model fabricates values" fix.
- **Extensible enums:** add `"unclear"` for ambiguous cases. Add `"other"` **plus a detail string field** for categories outside the enum. Closed enums force misclassification.
- **Strict schemas via tool_use eliminate *syntax* errors only.** They never eliminate *semantic* errors. Examples: line items that do not sum to the stated total, or correct values in wrong fields. Semantic checks are validation-layer work (G2).
- Include **format-normalization rules in the prompt** alongside strict output schemas when source formatting varies.

## G2. Validation, retry, and feedback loops (task 4.4)

- **Retry-with-error-feedback:** on validation failure, send a follow-up that contains **the original document, the failed extraction, and the specific validation errors**. The model self-corrects against named errors far better than against a bare "try again."
- **Limits of retry (tested):** retries fix **format/structural** errors. They are **ineffective when the information is absent from the source** (or lives in a document you did not provide). Diagnose which failure class you have before you use more retries.
- **Self-correction flows:** extract `calculated_total` alongside `stated_total` to flag discrepancies. Add `conflict_detected` booleans for inconsistent source data.
- **`detected_pattern` fields:** tag each finding with the code construct/pattern that triggered it. Then you can analyze dismissed findings in a systematic way for false-positive patterns (feeds the 4.1 precision loop).

## G3. Multi-instance & multi-pass review (task 4.6)

- **Self-review limitation:** a model that generated code **retains its reasoning context**. It is unlikely to question its own decisions in the same session. Neither "review your work carefully" instructions nor extended thinking substitute for independence.
- **Independent review instance:** a second Claude instance **without the generator's reasoning context** catches subtle issues self-review misses. This pairs with the CI session-isolation point in file 05's supplement.
- **Multi-pass review:** split large reviews into **per-file local passes + a cross-file integration pass**. Attention dilution, not context size, causes inconsistent single-pass reviews (see 08 §6).
- **Calibrated routing:** have the reviewer self-report **confidence alongside each finding** to enable review routing. Then calibrate those scores before you trust them (see 09 §5).

## G4. Gap-card Q&A

**GQ1.** The model fills in an invoice field the document does not contain. Schema fix?
**A.** Make the field optional/nullable. Required fields force fabrication.

**GQ2.** What errors do strict tool-use schemas eliminate, and what survives?
**A.** JSON syntax errors are eliminated. Semantic errors (wrong sums, misplaced values) survive and need validation logic.

**GQ3.** Describe retry-with-error-feedback.
**A.** Follow-up request = original document + failed extraction + the specific validation errors. This guides targeted self-correction.

**GQ4.** When will retries never fix an extraction?
**A.** When the required information is absent from the provided source. That is a data problem, not a format problem.

**GQ5.** Why is an independent instance better than "review your own code thoroughly"?
**A.** The generator retains its reasoning context and will not question its own decisions. Independence removes that bias.

**GQ6.** Purpose of a `detected_pattern` field in review findings?
**A.** Enables systematic analysis of which constructs trigger dismissed findings. This is the raw material for cutting false-positive categories.

**Q20. What is a routing workflow?** 
**A:** Classify the request. Then send it to a specialized prompt/model/tool path.

**Q21. What should you log for an agent turn?** 
**A:** Message ids, stop_reason, tool names/ids/inputs (redacted), results status, token usage, latency.

**Q22. Cache prefix order to remember?** 
**A:** Tools, then system, then messages—up to the cache breakpoint.

**Q23. What is `pause_turn` signaling?** 
**A:** A long-running server-side turn paused. Continue according to docs rather than treating it as a hard failure.

**Q24. Why are tool descriptions “prompt engineering”?** 
**A:** The model uses them to decide *when* and *how* to call tools. Vague descriptions cause wrong calls.

**Q25. What is wrong with returning a huge SQL dump as tool_result?** 
**A:** Context bloat and distraction. Summarize or page results. Keep raw data server-side.

**Q26. How do refusals differ from empty answers?** 
**A:** `stop_reason: "refusal"` shows a policy refusal. Handle with safe UX. Do not infinite-retry the same content.

**Q27. Name one Computer Use safety control.** 
**A:** Sandbox / least privilege / human approval for sensitive UI actions.

**Q28. What is the difference between chaining and parallelization?** 
**A:** Chaining is sequential dependencies. Parallelization fans out independent work then merges.

**Q29. Why separate retrieval metrics from answer metrics?** 
**A:** Bad answers may be good writing over bad chunks—or good chunks with a weak prompt. Fixes differ.

**Q30. What is the first debugging step when tool use does not complete?** 
**A:** Check whether every `tool_use` id received a `tool_result`. Inspect `stop_reason` / iteration caps.

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
|BM25. |Common lexical ranking function. |
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

Use this as a private drill list while you study official docs/SDKs:

1. **Hello Messages:** one-shot user message → print assistant text. Then add a second user turn with prior assistant message preserved.
2. **System ablations:** same user question with three different system prompts. Note tone and constraint adherence.
3. **Stream assemble:** concatenate text deltas. Assert equality with non-streamed output for the same seed/settings when possible.
4. **Tool round-trip:** fake weather tool. Verify ids match on `tool_result`. Force a parallel double call if the model emits two tools.
5. **Structured parse:** schema for `{ "sentiment": enum, "confidence": number }`. Fail the build if parse throws.
6. **Mini-RAG:** three markdown files → chunk → embed or BM25 → answer with “sources:” lines.
7. **Cache sanity:** put a long static system prompt behind cache controls. Compare latency on request 1 vs. request 2.
8. **Agent budget:** max 5 tool iterations. On loop, return “budget exceeded” instead of hanging.
9. **Stop-reason drill:** intentionally set tiny `max_tokens` and confirm you detect truncation.
10. **Batch dry-run design:** sketch 100 classification rows with `custom_id`s and a results joiner (even without calling the API).
11. **429 simulator:** write a retry helper with exponential backoff + jitter and a max-retry cap.
12. **Vision extract:** one screenshot/receipt image → structured JSON fields. Verify you did not hardcode fake OCR.

These drills reinforce exam mechanics. They are not substitutes for Anthropic’s graded quizzes.

---

## Source URLs (verify before exam day)

- Course outline: https://academy.claude.com/courses/building-with-the-claude-api
- Anthropic docs hub: https://docs.anthropic.com (also platform.claude.com/docs)
- Topics to re-open live: Messages, streaming, stop reasons, tool use, structured outputs, prompt caching, batch processing, vision/files, rate limits/errors, MCP overview

*End of notes. Cross-check parameters against current Anthropic platform docs before an exam sitting.*
