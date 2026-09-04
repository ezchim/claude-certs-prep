---
title: Introduction to Model Context Protocol
---

# Introduction to Model Context Protocol — Exam Prep Study Notes

> **Disclaimer:** These are **original study notes** for exam / certification prep. They are **not** official Anthropic course material or verbatim lesson text. Topics follow the public outline of [Introduction to Model Context Protocol](https://academy.claude.com/courses/introduction-to-model-context-protocol). 
> **Advanced follow-on (pointer only):** [Model Context Protocol: Advanced Topics](https://academy.claude.com/courses/model-context-protocol-advanced-topics) covers sampling, progress/logging, roots, JSON-RPC message details, STDIO handshake depth, Streamable HTTP / SSE, stateful vs. stateless scaling — take that after this intro course.

**Cross-check live docs:** [modelcontextprotocol.io](https://modelcontextprotocol.io) (architecture, primitives, transports, lifecycle). Spec versions evolve—prefer concepts over memorizing a single date stamp.

---

## Overview

MCP (Model Context Protocol) is an open standard for connecting AI hosts (Claude Desktop, IDEs, custom agents) to external tools and data **without** hand-rolling a unique JSON tool schema for every app×service pair. The intro course’s public learning goals emphasize: architecture (shifting tool definition/execution to specialized servers); transport-agnostic messaging; the end-to-end request flow; building servers with the Python SDK (decorators vs. raw JSON schemas); document-style tools; the MCP Inspector; **resources** (direct + templated); client-side resource reading and MIME types; **prompts** as reusable workflows; choosing among tools / resources / prompts; and integration patterns like autocomplete and context injection.

**Public module shape:** Introduction → Hands-on MCP servers → Connecting with MCP clients.

**Prereqs called out publicly:** basic Python, async/await, API literacy.

**How these notes relate to the API course:** Building with the Claude API teaches the Messages/tool loop inside *one* app. MCP teaches how many apps can share the same external capabilities through a common protocol. On exams, if you see “reuse across hosts” or “Inspector,” think MCP.

---

## Key concepts (cheat sheet)

| Concept | One-liner |
|---------|-----------|
| **Host** | The AI application the user sees (orchestrates clients, UI, model calls) |
| **Client** | In-host connector; **1:1** with a single MCP server |
| **Server** | Process/service exposing tools, resources, and/or prompts |
| **Tools** | Model-controlled actions (side effects allowed; like function calling) |
| **Resources** | App-controlled, mostly read-only data via URIs |
| **Prompts** | User-controlled, pre-crafted instruction templates |
| **Transport** | How JSON-RPC messages move (commonly **STDIO** locally; Streamable HTTP / SSE remotely) |
| **Data layer** | JSON-RPC methods, lifecycle, primitives, notifications |
| **Inspector** | Browser-based helper to list/call tools and debug a server |
| **Capability negotiation** | Initialize handshake declares what each side supports |

**Mental model:** Host asks the model what to do → model may select a tool → client invokes the MCP server → server talks to real systems → results flow back into the model context.

---

## Deep notes by topic

### 1. Why MCP exists

Without MCP, every AI app duplicates integration code for GitHub, Slack, databases, filesystems, etc. With MCP:

- **N servers** (one per system or capability domain) + **M hosts/clients** beat **M×N** bespoke adapters.
- Servers own auth to the backend, input validation, and domain logic.
- Hosts stay focused on UX, policy, and model orchestration.

Exam phrase to remember: MCP **shifts** tool definition and execution burden from your monolithic app server toward **specialized MCP servers**.

**Analogy:** USB standard vs. proprietary cables. Hosts speak MCP; servers speak MCP; backends stay behind the server adapter.

**What MCP is not**

- Not a replacement for the Claude Messages API (the host still calls the model).
- Not a database or an embedding store by itself.
- Not a guarantee of safety—poorly scoped servers are still dangerous.
- Not “only for Anthropic”; it is an open protocol many hosts can implement.

### 2. Architecture: hosts, clients, servers

```
User ↔ Host (LLM app) ↔ Client₁ ↔ Server₁ (e.g. docs)
 ↔ Client₂ ↔ Server₂ (e.g. calendar)
```

- **Host responsibilities:** spawn/manage clients; present prompts/resources to users; decide what context reaches the model; enforce user consent and security policy; run or call the LLM.
- **Client responsibilities:** capability negotiation / session lifecycle; translate host needs into MCP requests; isolate one server from another.
- **Server responsibilities:** advertise and implement primitives; stay focused (single domain is easier to secure and test).

**1:1 rule:** each client talks to exactly one server. A host that uses three servers runs three clients. This isolation is an exam favorite.

**Local vs. remote servers**

| | Local (often STDIO) | Remote (often Streamable HTTP) |
|--|---------------------|--------------------------------|
| Process | Host spawns subprocess | Independent service |
| Typical fan-out | One client per process | Many clients to one service |
| Auth story | OS user privileges | HTTP auth / OAuth / tokens |
| Failure mode | Crash kills that server | Network partitions, version skew |

Security boundary reminder: a compromised or overly powerful server is dangerous — least privilege, clear roots/paths, and user approval for sensitive tools matter even in “local” STDIO setups.

### 3. Layers: data vs. transport

Official architecture splits MCP into:

1. **Data layer** — JSON-RPC 2.0 message semantics: lifecycle, tools/resources/prompts, notifications, client features like sampling/elicitation (advanced).
2. **Transport layer** — how bytes move: STDIO vs. Streamable HTTP (with optional SSE), framing, and transport auth.

Intro exam bar: you can explain that the **same** JSON-RPC methods ride on different transports. Advanced course goes deeper on production HTTP scaling.

### 4. Message flow (request → Claude → tools → back)

A typical turn:

1. User asks a question in the host.
2. Host assembles context (chat history, selected resources, chosen prompt template).
3. Host calls Claude with available tool definitions discovered from MCP servers.
4. Claude returns a tool call (name + arguments).
5. Host/client sends an MCP tools/call to the right server.
6. Server executes (read file, edit doc, query API) and returns structured results.
7. Host appends tool results into the conversation and calls Claude again.
8. Claude produces a user-visible answer (or more tool calls).

Know the difference between **request/result** pairs (RPC calls) and **notifications** (one-way events). Deeper message taxonomy appears in the Advanced Topics course.

**Original sequence diagram (text)**

```text
User → Host: "Summarize Q3 plan doc"
Host → Client(docs): resources/read docs://q3-plan
Client → Server: resources/read
Server → Client: text/markdown body
Host → Claude: messages + tool defs + injected resource
Claude → Host: tool_use edit_section(...)
Host → Client: tools/call edit_section
Server → Client: ok
Host → Claude: tool_result
Claude → Host: final summary for user
```

### 5. Lifecycle (intro depth)

MCP sessions are not “fire random methods first.” Conceptual lifecycle:

1. **Connect** transport (spawn STDIO process or open HTTP session).
2. **Initialize** — exchange protocol version, `clientInfo` / `serverInfo`, and capabilities.
3. **Initialized notification** — client signals it is ready.
4. **Operate** — `tools/list`, `tools/call`, `resources/list`, `resources/read`, `prompts/list`, `prompts/get`, etc.
5. **Shut down** — close streams / terminate subprocess; don’t leak child processes.

**Capability negotiation intuition**

- Server may declare support for tools, resources, prompts, and whether lists can change (`listChanged`).
- Client may declare support for features servers might request later (sampling, elicitation—advanced).
- Calling an unsupported method is a protocol smell—discover first.

Exam tip: if asked “what happens before tools/list?”, answer **initialize / negotiate**, not “immediately call the tool.”

### 6. Building MCP servers (Python SDK mindset)

Intro course emphasis: use the SDK so you **don’t** hand-author giant JSON schemas.

- Decorators / type hints / `Field` descriptions become tool metadata the model sees.
- Good descriptions dramatically improve tool selection — treat them like prompt engineering.
- Example domain from the public outline: **document management** — tools to read and edit documents.
- Keep tools narrow: `read_document(id)` and `update_section(id, section, text)` beat a vague `do_anything(payload)`.

**Original decorator-style sketch (illustrative, not an SDK dump)**

```python
# Conceptual only — verify current SDK APIs before coding
@server.tool()
def read_document(doc_id: str) -> str:
 """Return the full text of a document by id. Use when the user asks what's inside a doc."""
...

@server.tool()
def update_section(doc_id: str, section: str, text: str) -> str:
 """Replace one named section. Prefer this over rewriting the entire document."""
...
```

**Description writing checklist**

- What the tool does in one sentence.
- When to use it / when not to.
- Units, id formats, side effects.
- Failure modes the model should expect.

**Inspector:** run the server and open the MCP Inspector in a browser to list tools, fire calls, and inspect payloads before wiring a full host. Great for exam “how do you debug a server?” answers.

### 7. Resources: direct vs. templated

**Resources** expose read-oriented data the *application* (not the model alone) decides to fetch and inject.

- **Direct resources:** static URI, e.g. `docs://index` or `config://app-settings`.
- **Templated resources:** URI templates with parameters, e.g. `docs://{doc_id}` — client fills params.

Client reading checklist:

- Request resource by URI.
- Handle **MIME types** (e.g. `application/json` vs. `text/plain`) so you parse correctly.
- Inject content into the model context deliberately (whole doc vs. excerpt).

**When resource vs. tool?** If it’s primarily “fetch data for context” with no meaningful side effect → resource. If it changes state or performs an action → tool.

**URI design tips**

- Use a clear scheme (`docs://`, `db://`, `config://`) per server domain.
- Keep ids URL-safe; document template parameters.
- Prefer immutable snapshots for auditability when injecting into prompts.

**MIME handling examples**

| MIME | Client behavior |
|------|-----------------|
| `text/plain` | Inject as text; maybe truncate |
| `text/markdown` | Same, preserve headings if possible |
| `application/json` | Parse; maybe pretty-print selected fields |
| `image/png` | Host may route to vision-capable message blocks |

Exam trap: ignoring MIME and always `json.loads`—binary or plain text will blow up.

### 8. Prompts as MCP primitives

**Prompts** are reusable, high-quality instruction templates for common workflows (the course publicly mentions examples like document formatting).

- **User-controlled:** typically selected from a menu (“Format selection as RFC-style memo”) rather than silently invented by the model.
- Can parameterize slots (title, tone, audience).
- Help teams share best-practice instructions across hosts.

**Original prompt template sketch**

```text
Name: format_as_adr
Args: title, status, context
Body:
 Write an Architecture Decision Record titled {title}.
 Status: {status}.
 Context: {context}.
 Use sections: Context, Decision, Consequences.
```

Host UX: user picks prompt → fills args → host expands → sends to model (optionally with related resources).

### 9. Choosing tools vs. resources vs. prompts

| Primitive | Who steers? | Typical use | Side effects? |
|-----------|-------------|-------------|---------------|
| Tool | Model | Search, edit, send, compute | Often yes |
| Resource | Application / host logic | Load file, config, dataset into context | Generally no |
| Prompt | User | Start a known workflow with vetted instructions | No (instructions only) |

Exam trap: calling everything a “tool.” Graders look for the control-plane distinction above.

**Sorting drill (memorize)**

- “Load README into context” → **resource**
- “Create GitHub issue” → **tool**
- “Apply ADR template” → **prompt**
- “Search docs then edit” → **tools** (+ maybe resources for known URIs)
- “Show company style guide” → **resource** or **prompt** depending on whether it’s raw data vs. a workflow template

### 10. Transports (intro-level)

MCP is **transport-agnostic** at the message layer (JSON-RPC style messages). Common options:

- **STDIO:** client spawns server as a subprocess; communicate over stdin/stdout. Ideal for local desktop/IDE integrations. Requires a correct **initialization handshake** before ordinary calls (details stressed more in Advanced Topics).
- **HTTP + SSE / Streamable HTTP:** for remote/networked servers; servers can push events; think sessions, auth, and scaling tradeoffs. Modern docs emphasize **Streamable HTTP** (HTTP POST client→server, optional SSE for streaming) and often recommend OAuth for tokens.

Intro exam bar: name the transports and when you’d pick local STDIO vs. remote HTTP. Leave dual-connection SSE minutiae and stateless horizontal scaling to the advanced course.

**Transport choice memo (original)**

Use STDIO when the server must touch the local filesystem or a developer laptop secret store and the host can manage a child process. Use Streamable HTTP when multiple hosts across a company should share one managed integration (with real auth, audit logs, and central updates).

### 11. Practical integration patterns

- **Context injection:** host reads resources → attaches relevant text to the model request.
- **Autocomplete / discovery:** list tools, resources, and prompts so UIs can suggest actions.
- **Lifecycle:** connect → initialize/negotiate → operate → shut down cleanly (don’t leak subprocesses).
- **Composition:** one host, many servers (docs + issue tracker + DB), each behind its own client.
- **Notifications:** if a server declares list-changed capability, it may notify clients to refresh `tools/list` (and similar)—hosts should update the model’s available tools.
- **Error surfacing:** tool failures should become structured errors the model can see, not silent host crashes.

### 12. Client features (pointer — advanced)

Intro awareness only (depth in Advanced Topics):

- **Sampling:** server asks the host/client to run an LLM completion (model-agnostic servers).
- **Elicitation:** server asks the host to collect user input / confirmation.
- **Logging / progress:** server emits logs or progress for long work.

Don’t overclaim these on an intro quiz—know they exist and which side exposes them.

### 13. Security considerations (intro must-pass)

1. **User consent:** hosts should approve sensitive tool calls, especially first use.
2. **Least privilege:** filesystem servers get a directory root, not `$HOME` entire disk, when possible.
3. **Secret handling:** servers should use OS keychains / env / secret managers—not hardcode tokens in tool responses.
4. **Confused deputy:** a malicious document could try to coerce the model into calling `delete_repo`—policy gates still required.
5. **Supply chain:** treat third-party MCP servers like installing software; review permissions.
6. **Transport auth:** remote servers need real authentication; STDIO inherits the user’s OS privileges—still not “safe by default.”
7. **Audit:** log tool name, args (redacted), actor, timestamp.

### 14. End-to-end document-management mental lab

Match the course’s public project theme without copying lab prose:

1. Server exposes `list_docs`, `read_document`, `update_section` tools.
2. Server exposes `docs://{doc_id}` templated resource and `docs://index` direct resource.
3. Server exposes a `format_document` prompt.
4. Use Inspector to call `read_document` before wiring Claude.
5. Host lists prompts for a “Format” menu; on select, expand prompt + optionally inject resource.
6. Model may still call `update_section` as a tool when the user asks to edit.

This single domain exercises **all three primitives**.

### 15. MCP vs. plain Messages tool defs

| Concern | Plain tool JSON in Messages | MCP |
|---------|------------------------------|-----|
| Discovery | Hardcoded in app | `*/list` from servers |
| Reuse | Copy schemas | Shared servers across hosts |
| Resources / prompts | Roll your own | First-class primitives |
| Ops | In-process functions | Local processes or remote services |
| Best when | One app, few tools | Ecosystem / many hosts |

Bridge insight: hosts often **translate** MCP tool schemas into the model provider’s tool format under the hood. MCP doesn’t remove the Claude tool loop—it standardizes the outer integration.

### 16. JSON-RPC mental model (lightweight)

Enough for intro exams:

- **Request:** has `id`, `method`, `params` → expects **result** or **error** with same `id`.
- **Notification:** has `method` (and maybe params) but **no `id`** → no response.
- Methods look like `initialize`, `tools/list`, `tools/call`, `resources/read`, `prompts/get`.
- Unknown methods / bad params → structured JSON-RPC errors, not HTTP HTML pages (even on HTTP transports, the payload is still RPC-shaped).

### 17. Common failure modes & debugging

| Symptom | Likely cause | Fix direction |
|---------|--------------|---------------|
| Host shows no tools | Init failed / capabilities missing / list not called | Check initialize + tools/list in Inspector |
| Tool call 404-ish / unknown tool | Stale list after server change | Handle list_changed; refresh |
| MIME parse error | Assumed JSON for text resource | Branch on MIME |
| Hung STDIO server | Handshake incomplete / stdout polluted with logs | Keep protocol on stdout; log to stderr |
| Remote 401 | Missing bearer/OAuth | Fix transport auth |
| Model never calls your tool | Weak description / too many overlapping tools | Rewrite descriptions; reduce set |
| Subprocess zombies | Host didn’t close lifecycle | Tear down clients on exit |

---



### 18. Host implementation checklist (what “good” looks like)

When you build or review a host that speaks MCP, graders (and production reviewers) look for more than “it connected once.”

1. **Config surface:** user can add/remove servers (command + args for STDIO; URL + auth for HTTP).
2. **Consent UX:** first-time tool use prompts; distinguish read-only vs. mutating tools.
3. **Unified registry:** merge tools from all clients with namespaced collision handling (`docs.read` vs. `db.read`).
4. **Model translation:** map MCP tool schemas into the provider tool format your LLM API expects.
5. **Result hygiene:** truncate huge tool outputs; preserve enough detail for the model to continue.
6. **Cancellation:** user can abort long tool calls; client cancels in-flight RPC when supported.
7. **Observability:** per-server latency, error rates, and which prompts/resources were injected.
8. **Graceful degradation:** if one server dies, other servers keep working.

### 19. Server design patterns (document domain + beyond)

**Pattern A — Thin adapter** 
MCP server wraps an existing REST API. Tools ≈ endpoints. Resources ≈ GETs that are safe to inject.

**Pattern B — Domain facade** 
Fewer, smarter tools (`search_and_summarize_locally`) that hide backend chaos. Easier for models; harder to keep descriptions honest.

**Pattern C — Read/write split** 
Expose reads as resources; writes as tools requiring confirmation. Matches the control-plane triad cleanly.

**Pattern D — Prompt pack** 
Server mostly ships prompts + a few resources (style guides). Great for teams standardizing workflows without giving the model write power.

**Anti-pattern — Kitchen sink** 
One server that shells out arbitrarily, reads all files, and sends email. Fails least-privilege and frightens host permission UIs.

### 20. Mapping Claude Academy modules → study actions

| Public module | What to practice tonight |
|---------------|--------------------------|
| Introduction | Draw architecture; recite triad |
| Hands-on servers | Build 2–3 tools; open Inspector |
| Connecting with clients | Read resource + MIME branch; list prompts; inject context |

Tie-back to Building with the Claude API: after MCP returns tool output, you still perform the Messages `tool_result` continuation—MCP didn’t erase that loop.

### 21. Worked story: autocomplete + context injection

**Autocomplete:** Host calls `prompts/list` and `tools/list` on idle. UI shows “Format as ADR”, “Search docs”, “Create issue”. User picks “Format as ADR” → host calls `prompts/get` with args → expands text into the composer.

**Context injection:** User @-mentions a doc. Host resolves `docs://{id}` resource, checks MIME, injects a truncated body into the next user message inside tagged `<doc>` delimiters, and *separately* keeps write tools available if the user asks to edit.

Exam insight: autocomplete is **discovery UX**; injection is **context assembly**. Both are host responsibilities even though data comes from servers.

### 22. Versioning & compatibility mindset

- Protocol versions appear during initialize; incompatible versions should fail fast.
- Servers and hosts upgrade independently—capability flags exist so optional features don’t hard-break.
- For exams: prefer answering “negotiate capabilities and degrade gracefully” over inventing a single forever-version number.


## Comparison tables (exam quick hits)

### Control plane triad

| Question | Tools | Resources | Prompts |
|----------|-------|-----------|---------|
| Who decides to use it? | Model | App/host | User |
| Primary verb | Call / execute | Read / inject | Select / expand |
| Typical risk | Side effects | Data exfil via context | Instruction exfil / social engineering |
| Discovery method family | `tools/*` | `resources/*` | `prompts/*` |

### STDIO vs Streamable HTTP

| Dimension | STDIO | Streamable HTTP |
|-----------|-------|-----------------|
| Where it shines | Desktop, IDE, local files | Team/shared integrations |
| Auth | OS user | Tokens / OAuth / headers |
| Scaling | Per-user processes | Central service |
| Ops complexity | Process mgmt | Network + auth + versioning |

### Host vs Client vs Server (blame chart)

| Bug | First place to look |
|-----|---------------------|
| Wrong tool chosen | Tool descriptions + model prompt (host) |
| Tool not listed | Client discovery / server capabilities |
| Wrong file edited | Server implementation / authz roots |
| UI missing prompt menu | Host didn’t list/present prompts |
| MIME crash | Client resource reader |

---

## Exam tips

1. Draw the **Host → Client → Server** triangle from memory; mark 1:1 client–server.
2. Memorize the triad: **tools = model-controlled**, **resources = app-controlled**, **prompts = user-controlled**.
3. SDK decorators exist to avoid brittle hand-written schemas — say that out loud once.
4. Inspector = first debugging stop for servers.
5. MIME type handling on resource reads is an easy “gotcha” question.
6. Know that an **Advanced Topics** course exists for sampling, roots, progress, and production transports — don’t overclaim intro-course depth on those.
7. Security: local STDIO is not “no security”; it still runs with user privileges.
8. Initialize before list/call.
9. Notifications ≠ requests (no response id).
10. MCP complements Messages tool use; it does not delete the tool_result loop.

---

## Exam traps

| Trap | Fix |
|------|-----|
| “Client can talk to many servers” | No—**host** has many clients; each client is 1:1 |
| Everything is a tool | Use resource/prompt when control plane differs |
| Skip Inspector, debug only in prod host | Inspector first |
| Ignore MIME | Always branch |
| Assume STDIO is sandboxed | It inherits user privileges |
| Memorize one HTTP dialect forever | Concepts: Streamable HTTP / SSE exist; verify current spec |
| Put secrets in resource bodies casually | Redact; use secure stores |
| Forget shutdown | Reap subprocesses |

---

## Self-check Q&A

**Q1. What are the three MCP roles?** 
**A:** Host (LLM app), Client (1:1 connector), Server (exposes primitives).

**Q2. Who typically decides to invoke a tool?** 
**A:** The model (with host policy gates); tools are model-controlled actions.

**Q3. Who typically selects a prompt template?** 
**A:** The user (user-controlled primitive), often via UI.

**Q4. Direct vs. templated resource?** 
**A:** Direct = fixed URI; templated = URI pattern with parameters filled by the client.

**Q5. Why use the Python SDK decorators?** 
**A:** Derive tool schemas from types/Field descriptions instead of maintaining raw JSON schemas by hand.

**Q6. What is the MCP Inspector for?** 
**A:** Browser-based testing/debugging of server tools/resources before full host integration.

**Q7. Name two transports and a use case for each.** 
**A:** STDIO for local subprocess servers; Streamable HTTP (or HTTP/SSE family) for remote servers.

**Q8. Where do you go next after this intro?** 
**A:** [MCP Advanced Topics](https://academy.claude.com/courses/model-context-protocol-advanced-topics) for sampling, progress/roots, and production transport/scaling details.

**Q9. What problem does MCP reduce mathematically?** 
**A:** M×N custom integrations toward M hosts + N specialized servers.

**Q10. What are the two MCP layers?** 
**A:** Data layer (JSON-RPC semantics/primitives) and transport layer (STDIO / HTTP).

**Q11. What must happen before ordinary tools/list calls?** 
**A:** Transport connect + initialize capability negotiation (+ initialized signal).

**Q12. Who owns backend API keys for a GitHub MCP server?** 
**A:** Typically the server (or its secret store)—not every host hardcoding tokens.

**Q13. How should a client react to tools/list_changed?** 
**A:** Refresh tool lists and update what the model can call.

**Q14. Resource or tool: “fetch schema for context”?** 
**A:** Resource (read-oriented, app-controlled injection).

**Q15. Resource or tool: “insert a row”?** 
**A:** Tool (side effect / action).

**Q16. Why keep stdout clean on STDIO servers?** 
**A:** Protocol messages travel on stdio—logging noise can break framing; use stderr for logs.

**Q17. What is context injection?** 
**A:** Host reads resources (or other data) and attaches them into the model request deliberately.

**Q18. What is autocomplete/discovery in MCP UIs?** 
**A:** Listing tools/resources/prompts so users/hosts can pick capabilities without hardcoding.

**Q19. Does MCP replace Claude’s tool_result loop?** 
**A:** No. Hosts still feed tool results back into the model conversation.

**Q20. Name a client-exposed advanced primitive.** 
**A:** Sampling, elicitation, or logging (intro: awareness only).

**Q21. What’s a good tool granularity rule?** 
**A:** Narrow, composable tools with clear schemas beat one mega-tool.

**Q22. Why are Field descriptions important?** 
**A:** They become model-facing guidance for arguments—prompt engineering for tools.

**Q23. What’s the security mistake in “local = safe”?** 
**A:** STDIO servers run with the user’s privileges and can still exfiltrate or delete data.

**Q24. How do prompts help teams?** 
**A:** Share vetted instruction templates across hosts instead of each user reinventing wording.

**Q25. What MIME-related bug is classic?** 
**A:** Parsing a text/plain resource as JSON (or ignoring content type entirely).

**Q26. Draw-from-memory: how many clients for 4 servers?** 
**A:** Four clients (1:1), managed by one host.

**Q27. What does transport-agnostic mean here?** 
**A:** Same data-layer methods can ride different transports.

**Q28. What’s a notification missing that requests have?** 
**A:** An `id` / expected response.

**Q29. When is plain inline tooling better than MCP?** 
**A:** Single-app prototypes with a few stable functions and no reuse need.

**Q30. What’s the first tool you use when a new server misbehaves?** 
**A:** MCP Inspector (list/call/inspect payloads) before full host integration.

---

## Quick review checklist

- [ ] I can draw Host/Client/Server with 1:1 links.
- [ ] I can explain M×N → M+N motivation.
- [ ] I can define tools vs resources vs prompts by *who controls* them.
- [ ] I can outline initialize → list → call/read → shutdown.
- [ ] I know STDIO vs Streamable HTTP selection criteria.
- [ ] I can describe Inspector’s role.
- [ ] I handle MIME types on resource reads.
- [ ] I can sort example tasks into the triad correctly.
- [ ] I know MCP sits beside—not instead of—the model tool loop.
- [ ] I can list at least five security considerations.
- [ ] I know Advanced Topics exists for sampling/roots/progress/scaling.
- [ ] I can sketch a doc-management server using all three primitives.

---

## Glossary

| Term | Plain meaning |
|------|----------------|
| MCP | Open protocol connecting AI hosts to external tools/data |
| Host | User-facing AI application coordinating clients + model |
| Client | In-host 1:1 session object talking to one server |
| Server | Process/service exposing MCP primitives |
| Tool | Executable action, usually model-requested |
| Resource | URI-addressable data for app-controlled context |
| Prompt (MCP) | User-selectable instruction template |
| STDIO transport | Subprocess stdin/stdout messaging |
| Streamable HTTP | Remote HTTP transport with optional SSE |
| JSON-RPC 2.0 | Request/response/notification message format |
| Initialize | Lifecycle handshake + capability negotiation |
| Capability | Declared feature support (tools, resources, …) |
| Inspector | Dev UI to exercise a server |
| MIME type | Content type guiding how to parse resource bodies |
| Notification | One-way RPC message without response |
| Sampling | Server-requested LLM call via client (advanced) |
| Elicitation | Server-requested user input via host (advanced) |
| Roots | Boundary paths/scopes for server access (advanced) |
| listChanged | Capability allowing list update notifications |
| Context injection | Host inserts resource content into model context |

---

## Study drills (original practice ideas)

1. **Sketch from memory:** Host/Client/Server with two servers attached; label which hop is 1:1.
2. **Primitive sorting:** Given a list like “load README”, “create issue”, “Apply ADR template”, tag each as resource, tool, or prompt.
3. **Fake doc server:** Two tools (`list_docs`, `get_doc`) + one templated resource; exercise them in an Inspector-like mindset (even if you only print JSON).
4. **MIME handling:** Return JSON and plain text resources; write client code that branches on MIME type.
5. **Failure path:** Tool throws; ensure the host surfaces an error `tool_result` the model can recover from instead of crashing the session.
6. **Transport choice memo:** One paragraph arguing STDIO for a desktop filesystem server vs. HTTP for a shared company knowledge server.
7. **Lifecycle flashcards:** Order connect / initialize / initialized / list / call / shutdown.
8. **Security review:** For a proposed `shell_exec` tool, write three policy gates you’d require in the host.
9. **Description rewrite:** Take a vague tool blurb (“manages docs”) and rewrite it with when-to-use / when-not / side effects.
10. **Bridge drill:** Explain in four sentences how an MCP `tools/call` becomes a Claude `tool_use` / `tool_result` round-trip inside a host.

Keep drills short; depth on sampling, roots, and Streamable HTTP belongs in the Advanced Topics course linked above.

---

## Source URLs

- Intro course: https://academy.claude.com/courses/introduction-to-model-context-protocol
- Advanced course (pointer): https://academy.claude.com/courses/model-context-protocol-advanced-topics
- MCP docs / architecture: https://modelcontextprotocol.io
- Spec & SDK indexes: linked from modelcontextprotocol.io (versions change—open live)

*End of course notes. Prefer the official MCP specification and Anthropic Academy labs for hands-on syntax that may change between SDK versions.*

---

# CCAR-F Domain 2 mechanics supplement (added 2026-08-23)

> **Scope:** the concrete mechanics the **CCAR-F exam guide Domain 2 (Tool Design & MCP Integration, 18%)** tests beyond this course's conceptual intro — task statements 2.2 (structured errors), 2.4 (MCP in Claude Code / agent workflows), 2.5 (built-in tools). Tool-description craft (2.1) is covered above §6 and in 08 §Q&A; `tool_choice` and tool distribution (2.3) are covered in `03-building-with-claude-api.md` — cross-check both. Verified against current Claude Code docs; original synthesis.

## S1. Structured error responses for MCP tools (task 2.2)

**The `isError` flag** is MCP's mechanism for communicating tool failure back to the agent *inside a successful protocol response*: the `tools/call` result carries `isError: true` and the error details in `content`, so the **model** can read the failure and reason about recovery. (A protocol-level JSON-RPC error is for malformed calls; a *tool-domain* failure travels as a normal result with `isError: true`.)

**Why generic errors fail:** a uniform `"Operation failed"` gives the agent no basis for choosing between retry, alternative approach, escalation, or asking the user. Wasted retries on permanent errors and abandoned recoverable ones both trace to unstructured errors.

**Error taxonomy the exam uses (memorize the four):**

| Category | Example | Retryable? | Agent's correct move |
| --- | --- | --- | --- |
| **Transient** | Timeout, service unavailable | Yes | Retry (with backoff) |
| **Validation** | Malformed input, bad ID format | No (as-is) | Fix the input, then retry |
| **Business** | Policy violation (refund cap) | **No** | Explain to the user / alternative workflow |
| **Permission** | Caller lacks access | No | Escalate / different credential path |

**Structured error metadata pattern:** return `errorCategory` (`transient` / `validation` / `permission` / business), an **`isRetryable` boolean**, and a **human-readable description**. For business-rule violations, set retryable **false** and include a **customer-friendly explanation** the agent can relay ("refunds above $500 require a supervisor") — the agent then communicates instead of hammering retries.

**Multi-agent link (Domain 5.3):** subagents recover **transient** failures locally; only unresolvable errors propagate to the coordinator, carrying what was attempted and partial results. And keep **access failure ≠ valid empty result**: an empty result set is a *successful* query with zero matches, never `isError: true`.

## S2. MCP servers in Claude Code: scoping and configuration (task 2.4)

**Two scopes, one exam distinction:**

| File | Scope | Shared? | Use for |
| --- | --- | --- | --- |
| **`.mcp.json`** (project root) | Project | Yes — committed to version control; every teammate gets it on clone/pull | Shared team tooling (the team's Jira/GitHub/db servers) |
| **`~/.claude.json`** | User | No — personal machine only | Personal/experimental servers you're trying out |

**Environment-variable expansion** in `.mcp.json` — e.g. `${GITHUB_TOKEN}` in an env/header value — lets teams commit server *configuration* while each member supplies their own **credentials from their environment**. The exam framing: how do you share the config **without committing secrets**? → env-var expansion in project-scoped `.mcp.json`.

**Discovery model:** tools from **all configured MCP servers are discovered at connection time and are available simultaneously** to the agent — a project server and a personal server coexist in one session (official prep Exercise 2 has you verify exactly this).

**MCP resources as content catalogs:** expose issue summaries, documentation hierarchies, or database schemas as **resources** so agents get *visibility into what data exists* without burning turns on exploratory tool calls. Catalog = resource; action = tool (consistent with the triad in §7–9 above).

**Two more 2.4 skills:**

- **Description competition with built-ins:** if your MCP tool's description is thin, the agent may prefer a built-in (like Grep) over your more capable server tool. Fix by **enriching the MCP tool description** with its capabilities and outputs — description quality drives adoption, not just selection between similar tools.
- **Community vs custom servers:** prefer existing community MCP servers for standard integrations (e.g. Jira); reserve custom server builds for team-specific workflows. (Deploying/hosting MCP server infrastructure is explicitly **out of scope** for the exam.)

## S3. Built-in tools: selection criteria (task 2.5)

The agent-side toolkit the exam names — know each tool's *selection criterion*:

| Tool | Selects when… | Canonical example |
| --- | --- | --- |
| **Grep** | Searching file **contents** for patterns | Find all callers of a function; locate an error message; find import statements |
| **Glob** | Matching file **paths/names** by pattern | `**/*.test.tsx` — all test files by naming convention |
| **Read** | Load a file's full contents | Follow an import found by Grep |
| **Write** | Write a full file | Pair with Read when Edit can't anchor |
| **Edit** | **Targeted modification via unique text matching** | Replace one function body |
| **Bash** | Run commands (tests, builds, git) | Execute the test suite |

**The Edit-failure pattern (tested):** Edit requires its anchor text to match **uniquely**. When it fails on non-unique matches, the fallback is **Read the full file, then Write the modified version** — not repeated Edit attempts with longer anchors.

**Incremental exploration pattern (tested):** build codebase understanding **incrementally** — Grep for entry points first, then Read to follow imports and trace flows — rather than reading all files upfront (context bloat; see Domain 5.4). Tracing usage across wrapper modules: first identify all exported names, then search for each name across the codebase.

## S4. Supplement Q&A

**SQ1.** How does an MCP tool tell the agent it failed?
**A.** `isError: true` on the tool result, with details in content — a domain failure travels as a readable result, not a protocol error, so the model can reason about recovery.

**SQ2.** Name the four error categories and which is retryable.
**A.** Transient (retryable), validation (fix input first), business (not retryable — explain to user), permission (not retryable — escalate/credentials).

**SQ3.** What three fields make an error response "structured"?
**A.** `errorCategory`, `isRetryable` boolean, human-readable description (plus a customer-friendly explanation for business violations).

**SQ4.** Why do uniform "Operation failed" responses hurt agents?
**A.** They remove the basis for recovery decisions — causing wasted retries on permanent errors and abandonment of recoverable ones.

**SQ5.** Where does a shared team MCP server go? A personal experiment?
**A.** Project-scoped `.mcp.json` (version-controlled); user-scoped `~/.claude.json`.

**SQ6.** How do you commit MCP config without committing tokens?
**A.** Environment-variable expansion (e.g. `${GITHUB_TOKEN}`) in `.mcp.json` — each member supplies credentials via their environment.

**SQ7.** When are MCP tools from multiple servers available?
**A.** Discovered at connection time; all configured servers' tools are available simultaneously.

**SQ8.** Agents keep making exploratory calls to learn what data exists. MCP fix?
**A.** Expose content catalogs (issue summaries, doc hierarchies, schemas) as **resources**.

**SQ9.** The agent prefers built-in Grep over your capable MCP search tool. Fix?
**A.** Enrich the MCP tool's description with its capabilities and outputs — descriptions drive adoption.

**SQ10.** Grep vs Glob in one line?
**A.** Grep searches file *contents* for patterns; Glob matches file *paths/names*.

**SQ11.** Edit keeps failing on a non-unique match. Fallback?
**A.** Read the full file, then Write the modified version.

**SQ12.** Right way to build understanding of a large codebase?
**A.** Incrementally: Grep entry points → Read to follow imports/trace flows — not reading everything upfront.

**SQ13.** A subagent's search times out once, then succeeds on retry. What reaches the coordinator?
**A.** Nothing — transient failures are recovered locally; only unresolvable errors propagate (with attempts + partials).

**SQ14.** Empty search results: `isError: true` or not?
**A.** Not — a valid empty result is a successful query with zero matches; flagging it as error corrupts retry logic.

## S5. Supplement checklist

- [ ] I can state what `isError` does and where error detail lives.
- [ ] I can classify an error into the four categories and derive the agent's move.
- [ ] I know `.mcp.json` vs `~/.claude.json` and the env-var expansion trick.
- [ ] I know tools from all servers are discovered at connection time, simultaneously.
- [ ] I can pick resource-as-catalog vs tool-as-action.
- [ ] I can select among Read/Write/Edit/Bash/Grep/Glob and recite the Edit→Read+Write fallback.
