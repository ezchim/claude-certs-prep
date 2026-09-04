# Introduction to Model Context Protocol — Exam Prep Study Notes — Simplified Technical English

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names MCP, Claude, JSON-RPC, STDIO, SSE stay as written. These notes are **original exam-prep notes**. They are **not** official Anthropic course material. They are **not** verbatim lesson text. The official course is the source of truth. Spec versions evolve. Topics follow the public outline of [Introduction to Model Context Protocol](https://academy.claude.com/courses/introduction-to-model-context-protocol).

> **Advanced follow-on (pointer only):** Take [Model Context Protocol: Advanced Topics](https://academy.claude.com/courses/model-context-protocol-advanced-topics) after this intro course. It covers sampling, progress/logging, and roots. It covers JSON-RPC message details and STDIO handshake depth. It covers Streamable HTTP / SSE and stateful vs. stateless scaling.

**Check live docs:** [modelcontextprotocol.io](https://modelcontextprotocol.io) (architecture, primitives, transports, lifecycle). Spec versions evolve. Prefer concepts. Do not memorize one date stamp.

---

## Overview

MCP (Model Context Protocol) is an open standard. It connects AI hosts (Claude Desktop, IDEs, custom agents) to external tools and data. You do **not** write a unique JSON tool schema by hand for every app and service pair.

The intro course public learning goals emphasize these topics. Architecture: you shift tool definition and execution to specialized servers. Messaging does not depend on one transport. You learn the end-to-end request flow. You build servers with the Python SDK (decorators vs. raw JSON schemas). You use document-style tools. You use the MCP Inspector. You use **resources** (direct + templated). You read resources on the client side and handle MIME types. You use **prompts** as reusable workflows. You select among tools / resources / prompts. You use integration patterns such as autocomplete and context injection.

**Public module shape:** Introduction → Hands-on MCP servers → Connecting with MCP clients.

**Prereqs the public course lists:** basic Python, async/await, and API literacy.

**How these notes relate to the API course:** Building with the Claude API teaches the Messages/tool loop inside *one* app. MCP teaches how many apps share the same external capabilities through one common protocol. On exams, if you see “reuse across hosts” or “Inspector,” think MCP.

---

## Key concepts (cheat sheet)

| Concept | One-liner |
|---------|-----------|
| **Host** | The AI application the user sees (it orchestrates clients, UI, and model calls) |
| **Client** | Connector inside the host. **1:1** with a single MCP server |
| **Server** | Process or service that exposes tools, resources, and/or prompts |
| **Tools** | Model-controlled actions (side effects are allowed. Like function calling) |
| **Resources** | App-controlled, mostly read-only data through URIs |
| **Prompts** | User-controlled, pre-crafted instruction templates |
| **Transport** | How JSON-RPC messages move (commonly **STDIO** locally. Streamable HTTP / SSE remotely) |
| **Data layer** | JSON-RPC methods, lifecycle, primitives, notifications |
| **Inspector** | Browser-based helper that lists and calls tools and helps you debug a server |
| **Capability negotiation** | Initialize handshake declares what each side supports |

**Working model:** The host asks the model what to do. The model may select a tool. The client calls the MCP server. The server talks to real systems. Results flow back into the model context.

---

## Deep notes by topic

### 1. Why MCP exists

Without MCP, every AI app copies integration code for GitHub, Slack, databases, filesystems, and similar systems. With MCP:

- **N servers** (one per system or capability domain) + **M hosts/clients** is better than **M×N** custom adapters.
- Servers own auth to the backend, input validation, and domain logic.
- Hosts keep their focus on UX, policy, and model orchestration.

Exam phrase to remember: MCP **shifts** tool definition and execution from your one large app server toward **specialized MCP servers**.

**Analogy:** USB standard vs. proprietary cables. Hosts speak MCP. Servers speak MCP. Backends stay behind the server adapter.

**What MCP is not**

- Not a replacement for the Claude Messages API (the host still calls the model).
- Not a database or an embedding store by itself.
- Not a guarantee of safety. Servers with a wide scope are still dangerous.
- Not “only for Anthropic”. It is an open protocol. Many hosts can implement it.

### 2. Architecture: hosts, clients, servers

```
User ↔ Host (LLM app) ↔ Client₁ ↔ Server₁ (e.g. docs)
 ↔ Client₂ ↔ Server₂ (e.g. calendar)
```

- **Host responsibilities:** Start and manage clients. Present prompts/resources to users. Decide what context reaches the model. Enforce user consent and security policy. Run or call the LLM.
- **Client responsibilities:** Run capability negotiation and session lifecycle. Translate host needs into MCP requests. Isolate one server from another.
- **Server responsibilities:** Advertise and implement primitives. Keep a narrow focus. A single domain is easier to secure and test.

**1:1 rule:** each client talks to exactly one server. A host that uses three servers runs three clients. This isolation is a frequent exam topic.

**Local vs. remote servers**

| | Local (often STDIO) | Remote (often Streamable HTTP) |
|--|---------------------|--------------------------------|
| Process | Host starts a subprocess | Independent service |
| Typical fan-out | One client per process | Many clients to one service |
| Auth story | OS user privileges | HTTP auth / OAuth / tokens |
| Failure mode | A crash stops that server | Network partitions, version skew |

Security boundary reminder: a compromised or overly powerful server is dangerous. Use least privilege. Set clear roots/paths. Get user approval for sensitive tools. These rules still apply in “local” STDIO setups.

### 3. Layers: data vs. transport

Official architecture splits MCP into:

1. **Data layer** — JSON-RPC 2.0 message semantics: lifecycle, tools/resources/prompts, notifications, and client features like sampling/elicitation (advanced).
2. **Transport layer** — how bytes move: STDIO vs. Streamable HTTP (with optional SSE), framing, and transport auth.

Intro exam level: you can explain that the **same** JSON-RPC methods travel on different transports. The Advanced course goes deeper on production HTTP scaling.

### 4. Message flow (request → Claude → tools → back)

A typical turn:

1. The user asks a question in the host.
2. The host assembles context (chat history, selected resources, chosen prompt template).
3. The host calls Claude with available tool definitions that it discovered from MCP servers.
4. Claude returns a tool call (name + arguments).
5. The host/client sends an MCP tools/call to the correct server.
6. The server executes (read file, edit doc, query API) and returns structured results.
7. The host adds tool results into the conversation and calls Claude again.
8. Claude produces a user-visible answer (or more tool calls).

Know the difference between **request/result** pairs (RPC calls) and **notifications** (one-way events). A deeper message taxonomy appears in the Advanced Topics course.

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

MCP sessions must not call methods in a random order. Conceptual lifecycle:

1. **Connect** transport (start a STDIO process or open an HTTP session).
2. **Initialize** — exchange protocol version, `clientInfo` / `serverInfo`, and capabilities.
3. **Initialized notification** — the client signals it is ready.
4. **Operate** — `tools/list`, `tools/call`, `resources/list`, `resources/read`, `prompts/list`, `prompts/get`, and similar methods.
5. **Stop** — close streams / stop the subprocess. Do not leave child processes running.

**Capability negotiation intuition**

- The server may declare support for tools, resources, and prompts. It may also declare whether lists can change (`listChanged`).
- The client may declare support for features that servers might request later (sampling, elicitation—advanced).
- A call to an unsupported method is incorrect protocol use. Discover first.

**Exam rule:** if the question asks “what happens before tools/list?”, answer **initialize / negotiate**. Do not answer “call the tool immediately.”

### 6. Building MCP servers (Python SDK mindset)

Intro course emphasis: use the SDK so you **do not** write large JSON schemas by hand.

- Decorators / type hints / `Field` descriptions become tool metadata that the model sees.
- Good descriptions improve tool selection greatly. Treat them as prompt engineering.
- Example domain from the public outline: **document management** — tools to read and edit documents.
- Keep tools narrow: `read_document(id)` and `update_section(id, section, text)` are better than a vague `do_anything(payload)`.

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

- What the tool does, in one sentence.
- When to use it / when not to use it.
- Units, id formats, side effects.
- Failure modes the model should expect.

**Inspector:** run the server and open the MCP Inspector in a browser. List tools, run calls, and inspect payloads before you connect a full host. This is a strong exam answer for “how do you debug a server?”

### 7. Resources: direct vs. templated

**Resources** expose read-oriented data. The *application* (not the model alone) decides to fetch the data and inject it.

- **Direct resources:** static URI, e.g. `docs://index` or `config://app-settings`.
- **Templated resources:** URI templates with parameters, e.g. `docs://{doc_id}` — the client fills params.

Client reading checklist:

- Request the resource by URI.
- Handle **MIME types** (e.g. `application/json` vs. `text/plain`) so you parse correctly.
- Inject content into the model context with a clear plan (whole doc vs. excerpt).

**When resource vs. tool?** If the work is mainly “fetch data for context” with no meaningful side effect → resource. If it changes state or performs an action → tool.

**URI design tips**

- Use a clear scheme (`docs://`, `db://`, `config://`) per server domain.
- Keep ids URL-safe. Document template parameters.
- Prefer immutable snapshots for audit when you inject into prompts.

**MIME handling examples**

| MIME | Client behavior |
|------|-----------------|
| `text/plain` | Inject as text. Maybe truncate |
| `text/markdown` | Same, keep headings if possible |
| `application/json` | Parse. Maybe pretty-print selected fields |
| `image/png` | The host may route to vision-capable message blocks |

**Common exam error:** you ignore MIME and always run `json.loads`. Binary or plain text then fails.

### 8. Prompts as MCP primitives

**Prompts** are reusable, high-quality instruction templates for common workflows. The course publicly mentions examples like document formatting.

- **User-controlled:** you typically select them from a menu (“Format selection as RFC-style memo”). The model does not invent them without user action.
- You can parameterize slots (title, tone, audience).
- They help teams share best-practice instructions across hosts.

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

Host UX: the user selects a prompt. The user fills args. The host expands the prompt. The host sends it to the model. The host may attach related resources.

### 9. Choosing tools vs. resources vs. prompts

| Primitive | Who steers? | Typical use | Side effects? |
|-----------|-------------|-------------|---------------|
| Tool | Model | Search, edit, send, compute | Often yes |
| Resource | Application / host logic | Load file, config, dataset into context | Generally no |
| Prompt | User | Start a known workflow with vetted instructions | No (instructions only) |

**Common exam error:** you call everything a “tool.” Graders look for the control-plane distinction above.

**Sorting drill (memorize)**

- “Load README into context” → **resource**
- “Create GitHub issue” → **tool**
- “Apply ADR template” → **prompt**
- “Search docs then edit” → **tools** (+ maybe resources for known URIs)
- “Show company style guide” → **resource** or **prompt**, depending on raw data vs. a workflow template

### 10. Transports (intro-level)

MCP is **transport-agnostic** at the message layer (JSON-RPC style messages). Common options:

- **STDIO:** the client starts the server as a subprocess. They communicate over stdin/stdout. This is ideal for local desktop/IDE integrations. You need a correct **initialization handshake** before ordinary calls (the Advanced Topics course stresses the details more).
- **HTTP + SSE / Streamable HTTP:** for remote/networked servers. Servers can push events. Think about sessions, auth, and scaling tradeoffs. Modern docs emphasize **Streamable HTTP** (HTTP POST client→server, optional SSE for streaming). They often recommend OAuth for tokens.

Intro exam level: name the transports. Know when you select local STDIO vs. remote HTTP. Leave dual-connection SSE details and stateless horizontal scaling to the advanced course.

**Transport choice memo (original)**

Use STDIO when the server must touch the local filesystem or a developer laptop secret store. Use STDIO when the host can manage a child process. Use Streamable HTTP when multiple hosts across a company share one managed integration. That path needs real auth, audit logs, and central updates.

### 11. Practical integration patterns

- **Context injection:** the host reads resources → attaches relevant text to the model request.
- **Autocomplete / discovery:** list tools, resources, and prompts so UIs can suggest actions.
- **Lifecycle:** connect → initialize/negotiate → operate → stop cleanly (do not leave subprocesses running).
- **Composition:** one host, many servers (docs + issue tracker + DB), each behind its own client.
- **Notifications:** if a server declares list-changed capability, it may notify clients to refresh `tools/list` (and similar). Hosts should update the model’s available tools.
- **Error surfacing:** tool failures should become structured errors the model can see. They should not become silent host crashes.

### 12. Client features (pointer — advanced)

Intro awareness only (depth is in Advanced Topics):

- **Sampling:** the server asks the host/client to run an LLM completion (model-agnostic servers).
- **Elicitation:** the server asks the host to collect user input / confirmation.
- **Logging / progress:** the server emits logs or progress for long work.

Do not claim extra depth on these on an intro quiz. Know they exist. Know which side exposes them.

### 13. Security considerations (intro must-pass)

1. **User consent:** hosts should approve sensitive tool calls, especially on first use.
2. **Least privilege:** filesystem servers get a directory root. They do not get the entire `$HOME` disk when that is possible.
3. **Secret handling:** servers should use OS keychains / env / secret managers. Do not hardcode tokens in tool responses.
4. **Confused deputy:** a malicious document could try to coerce the model into calling `delete_repo`. You still need policy gates.
5. **Supply chain:** treat third-party MCP servers like software you install. Review permissions.
6. **Transport auth:** remote servers need real authentication. STDIO inherits the user’s OS privileges. It is still not “safe by default.”
7. **Audit:** log tool name, args (redacted), actor, and timestamp.

### 14. End-to-end document-management mental lab

Match the course public project theme. Do not copy lab prose:

1. The server exposes `list_docs`, `read_document`, and `update_section` tools.
2. The server exposes a `docs://{doc_id}` templated resource and a `docs://index` direct resource.
3. The server exposes a `format_document` prompt.
4. Use Inspector to call `read_document` before you wire Claude.
5. The host lists prompts for a “Format” menu. On select, it expands the prompt and optionally injects a resource.
6. The model may still call `update_section` as a tool when the user asks to edit.

This single domain exercises **all three primitives**.

### 15. MCP vs. plain Messages tool defs

| Concern | Plain tool JSON in Messages | MCP |
|---------|------------------------------|-----|
| Discovery | Hardcoded in the app | `*/list` from servers |
| Reuse | Copy schemas | Shared servers across hosts |
| Resources / prompts | You build your own | First-class primitives |
| Ops | In-process functions | Local processes or remote services |
| Best when | One app, few tools | Ecosystem / many hosts |

Bridge insight: hosts often **translate** MCP tool schemas into the model provider’s tool format internally. MCP does not remove the Claude tool loop. It standardizes the outer integration.

### 16. JSON-RPC mental model (lightweight)

Enough for intro exams:

- **Request:** has `id`, `method`, `params` → expects **result** or **error** with the same `id`.
- **Notification:** has `method` (and maybe params) but **no `id`** → no response.
- Methods look like `initialize`, `tools/list`, `tools/call`, `resources/read`, `prompts/get`.
- Unknown methods / bad params → structured JSON-RPC errors. They are not HTTP HTML pages. Even on HTTP transports, the payload is still RPC-shaped.

### 17. Common failure modes & debugging

| Symptom | Likely cause | Fix direction |
|---------|--------------|---------------|
| Host shows no tools | Init failed / capabilities missing / list not called | Check initialize + tools/list in Inspector |
| Tool call looks like 404 / unknown tool | Stale list after a server change | Handle list_changed. Refresh |
| MIME parse error | Assumed JSON for a text resource | Branch on MIME |
| Hung STDIO server | Handshake incomplete / logs written to stdout | Keep protocol on stdout. Log to stderr |
| Remote 401 | Missing bearer/OAuth | Fix transport auth |
| Model never calls your tool | Weak description / too many overlapping tools | Rewrite descriptions. Reduce the set |
| Child processes left running | Host did not close the lifecycle | Stop clients on exit |

---

### 18. Host implementation checklist (what “good” looks like)

When you build or review a host that speaks MCP, graders look for more than one successful connect. Production reviewers look for the same.

1. **Config surface:** the user can add/remove servers (command + args for STDIO. URL + auth for HTTP).
2. **Consent UX:** first-time tool use prompts. Distinguish read-only vs. mutating tools.
3. **Unified registry:** merge tools from all clients with namespaced collision handling (`docs.read` vs. `db.read`).
4. **Model translation:** map MCP tool schemas into the provider tool format your LLM API expects.
5. **Result hygiene:** truncate huge tool outputs. Keep enough detail for the model to continue.
6. **Cancellation:** the user can abort long tool calls. The client cancels in-flight RPC when supported.
7. **Observability:** per-server latency, error rates, and which prompts/resources were injected.
8. **Graceful degradation:** if one server dies, other servers keep working.

### 19. Server design patterns (document domain + beyond)

**Pattern A — Thin adapter**
MCP server wraps an existing REST API. Tools ≈ endpoints. Resources ≈ GETs that are safe to inject.

**Pattern B — Domain facade**
Fewer, smarter tools (`search_and_summarize_locally`) that hide backend complexity. Easier for models. Harder to keep descriptions honest.

**Pattern C — Read/write split**
Expose reads as resources. Expose writes as tools that need confirmation. This matches the control-plane triad cleanly.

**Pattern D — Prompt pack**
The server mainly ships prompts + a few resources (style guides). This is strong for teams that standardize workflows without write power for the model.

**Anti-pattern — do-everything server**
One server that runs arbitrary shell commands, reads all files, and sends email. This fails least-privilege. Host permission UIs then show high risk.

### 20. Mapping Claude Academy modules → study actions

| Public module | What to practice in this session |
|---------------|--------------------------|
| Introduction | Draw architecture. Recite the triad |
| Hands-on servers | Build 2–3 tools. Open Inspector |
| Connecting with clients | Read a resource + MIME branch. List prompts. Inject context |

Link back to Building with the Claude API. After MCP returns tool output, you still perform the Messages `tool_result` continuation. MCP did not erase that loop.

### 21. Worked story: autocomplete + context injection

**Autocomplete:** The host calls `prompts/list` and `tools/list` on idle. The UI shows “Format as ADR”, “Search docs”, “Create issue”. The user selects “Format as ADR” → the host calls `prompts/get` with args → expands text into the composer.

**Context injection:** The user @-mentions a doc. The host resolves the `docs://{id}` resource and checks MIME. The host injects a truncated body into the next user message inside tagged `<doc>` delimiters. It *separately* keeps write tools available if the user asks to edit.

**Exam insight:** autocomplete is **discovery UX**. Injection is **context assembly**. Both are host responsibilities, even though data comes from servers.

### 22. Versioning & compatibility mindset

- Protocol versions appear during initialize. Incompatible versions should fail immediately.
- Servers and hosts upgrade independently. Capability flags exist so optional features do not break required behavior.
- For exams: prefer the answer “negotiate capabilities and degrade gracefully.” Do not invent a single forever-version number.

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
| Best for | Desktop, IDE, local files | Team/shared integrations |
| Auth | OS user | Tokens / OAuth / headers |
| Scaling | Per-user processes | Central service |
| Ops complexity | Process mgmt | Network + auth + versioning |

### Host vs Client vs Server (blame chart)

| Bug | First place to look |
|-----|---------------------|
| Wrong tool chosen | Tool descriptions + model prompt (host) |
| Tool not listed | Client discovery / server capabilities |
| Wrong file edited | Server implementation / authz roots |
| UI missing prompt menu | Host did not list/present prompts |
| MIME crash | Client resource reader |

---

## Exam tips

1. Draw the **Host → Client → Server** triangle without notes. Mark 1:1 client–server.
2. Memorize the triad: **tools = model-controlled**, **resources = app-controlled**, **prompts = user-controlled**.
3. SDK decorators exist so you avoid brittle hand-written schemas. Remember this fact.
4. Inspector = first debugging stop for servers.
5. MIME type handling on resource reads is an easy trap question.
6. Know that an **Advanced Topics** course exists for sampling, roots, progress, and production transports. Do not claim intro-course depth on those topics.
7. Security: local STDIO is not “no security”. It still runs with user privileges.
8. Initialize before list/call.
9. Notifications ≠ requests (no response id).
10. MCP complements Messages tool use. It does not delete the tool_result loop.

---

## Exam traps

| Trap | Fix |
|------|-----|
| “Client can talk to many servers” | No—the **host** has many clients. Each client is 1:1 |
| Everything is a tool | Use resource/prompt when the control plane differs |
| Skip Inspector, debug only in the prod host | Inspector first |
| Ignore MIME | Always branch |
| Assume STDIO is sandboxed | It inherits user privileges |
| Memorize one HTTP dialect forever | Concepts: Streamable HTTP / SSE exist. Check the current spec |
| Put secrets in resource bodies without care | Redact. Use secure stores |
| Forget shutdown | Stop child processes |

---

## Self-check Q&A

**Q1. What are the three MCP roles?**
**A:** Host (LLM app), Client (1:1 connector), Server (exposes primitives).

**Q2. Who typically decides to invoke a tool?**
**A:** The model (with host policy gates). Tools are model-controlled actions.

**Q3. Who typically selects a prompt template?**
**A:** The user (user-controlled primitive), often through UI.

**Q4. Direct vs. templated resource?**
**A:** Direct = fixed URI. Templated = URI pattern with parameters that the client fills.

**Q5. Why use the Python SDK decorators?**
**A:** Derive tool schemas from types/Field descriptions. Do not maintain raw JSON schemas by hand.

**Q6. What is the MCP Inspector for?**
**A:** Browser-based testing/debugging of server tools/resources before full host integration.

**Q7. Name two transports and a use case for each.**
**A:** STDIO for local subprocess servers. Streamable HTTP (or HTTP/SSE family) for remote servers.

**Q8. Where do you go next after this intro?**
**A:** [MCP Advanced Topics](https://academy.claude.com/courses/model-context-protocol-advanced-topics) for sampling, progress/roots, and production transport/scaling details.

**Q9. What problem does MCP reduce mathematically?**
**A:** M×N custom integrations toward M hosts + N specialized servers.

**Q10. What are the two MCP layers?**
**A:** Data layer (JSON-RPC semantics/primitives) and transport layer (STDIO / HTTP).

**Q11. What must happen before ordinary tools/list calls?**
**A:** Transport connect + initialize capability negotiation (+ initialized signal).

**Q12. Who owns backend API keys for a GitHub MCP server?**
**A:** Typically the server (or its secret store). Not every host hardcoding tokens.

**Q13. How should a client react to tools/list_changed?**
**A:** Refresh tool lists and update what the model can call.

**Q14. Resource or tool: “fetch schema for context”?**
**A:** Resource (read-oriented, app-controlled injection).

**Q15. Resource or tool: “insert a row”?**
**A:** Tool (side effect / action).

**Q16. Why keep stdout clean on STDIO servers?**
**A:** Protocol messages travel on stdio. Logging noise can break framing. Use stderr for logs.

**Q17. What is context injection?**
**A:** The host reads resources (or other data) and attaches them into the model request with a clear plan.

**Q18. What is autocomplete/discovery in MCP UIs?**
**A:** Listing tools/resources/prompts so users/hosts can pick capabilities without hardcoding.

**Q19. Does MCP replace Claude’s tool_result loop?**
**A:** No. Hosts still feed tool results back into the model conversation.

**Q20. Name a client-exposed advanced primitive.**
**A:** Sampling, elicitation, or logging (intro: awareness only).

**Q21. What is a good tool granularity rule?**
**A:** Narrow, composable tools with clear schemas are better than one oversized tool.

**Q22. Why are Field descriptions important?**
**A:** They become model-facing guidance for arguments. This is prompt engineering for tools.

**Q23. What is the security mistake in “local = safe”?**
**A:** STDIO servers run with the user’s privileges. They can still exfiltrate or delete data.

**Q24. How do prompts help teams?**
**A:** Share vetted instruction templates across hosts. Each user does not reinvent the wording.

**Q25. What MIME-related bug is common?**
**A:** Parsing a text/plain resource as JSON (or ignoring content type entirely).

**Q26. Without notes: how many clients for 4 servers?**
**A:** Four clients (1:1), managed by one host.

**Q27. What does transport-agnostic mean here?**
**A:** The same data-layer methods can travel on different transports.

**Q28. What is a notification missing that requests have?**
**A:** An `id` / expected response.

**Q29. When is plain inline tooling better than MCP?**
**A:** Single-app prototypes with a few stable functions and no reuse need.

**Q30. What is the first tool you use when a new server does not work?**
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
| Host | User-facing AI application that coordinates clients + model |
| Client | In-host 1:1 session object talking to one server |
| Server | Process/service that exposes MCP primitives |
| Tool | Executable action, usually model-requested |
| Resource | URI-addressable data for app-controlled context |
| Prompt (MCP) | User-selectable instruction template |
| STDIO transport | Subprocess stdin/stdout messaging |
| Streamable HTTP | Remote HTTP transport with optional SSE |
| JSON-RPC 2.0 | Request/response/notification message format |
| Initialize | Lifecycle handshake + capability negotiation |
| Capability | Declared feature support (tools, resources, …) |
| Inspector | Dev UI to exercise a server |
| MIME type | Content type that guides how you parse resource bodies |
| Notification | One-way RPC message without response |
| Sampling | Server-requested LLM call via client (advanced) |
| Elicitation | Server-requested user input via host (advanced) |
| Roots | Boundary paths/scopes for server access (advanced) |
| listChanged | Capability that allows list update notifications |
| Context injection | Host inserts resource content into model context |

---

## Study drills (original practice ideas)

1. **Sketch without notes:** Host/Client/Server with two servers attached. Label which hop is 1:1.
2. **Primitive sorting:** Given a list like “load README”, “create issue”, “Apply ADR template”, tag each as resource, tool, or prompt.
3. **Fake doc server:** Two tools (`list_docs`, `get_doc`) + one templated resource. Exercise them in an Inspector-like mindset (even if you only print JSON).
4. **MIME handling:** Return JSON and plain text resources. Write client code that branches on MIME type.
5. **Failure path:** A tool throws. Ensure the host surfaces an error `tool_result` the model can recover from. Do not crash the session.
6. **Transport choice memo:** One paragraph that argues STDIO for a desktop filesystem server vs. HTTP for a shared company knowledge server.
7. **Lifecycle flashcards:** Order connect / initialize / initialized / list / call / shutdown.
8. **Security review:** For a proposed `shell_exec` tool, write three policy gates you would require in the host.
9. **Description rewrite:** Take a vague tool blurb (“manages docs”). Rewrite it with when-to-use / when-not / side effects.
10. **Bridge drill:** Explain in four sentences how an MCP `tools/call` becomes a Claude `tool_use` / `tool_result` round-trip inside a host.

Keep drills short. Depth on sampling, roots, and Streamable HTTP belongs in the Advanced Topics course linked above.

---

## Source URLs

- Intro course: https://academy.claude.com/courses/introduction-to-model-context-protocol
- Advanced course (pointer): https://academy.claude.com/courses/model-context-protocol-advanced-topics
- MCP docs / architecture: https://modelcontextprotocol.io
- Spec & SDK indexes: linked from modelcontextprotocol.io (versions change—open live)

*End of course notes. Prefer the official MCP specification and Anthropic Academy labs. Hands-on syntax may change between SDK versions.*

---

# CCAR-F Domain 2 mechanics supplement (added 2026-08-23)

> **Scope:** These are the concrete mechanics that **CCAR-F exam guide Domain 2 (Tool Design & MCP Integration, 18%)** tests beyond this course conceptual intro. The tasks are 2.2 (structured errors), 2.4 (MCP in Claude Code / agent workflows), and 2.5 (built-in tools). §6 and 08 §Q&A cover tool-description craft (2.1). `03-building-with-claude-api.md` covers `tool_choice` and tool distribution (2.3). Cross-check both. The author verified these mechanics against current Claude Code docs. Original synthesis.

## S1. Structured error responses for MCP tools (task 2.2)

**The `isError` flag** is MCP's mechanism for tool failure. It sends the failure back to the agent *inside a successful protocol response*. The `tools/call` result carries `isError: true`. Error details are in `content`. The **model** can then read the failure and reason about recovery. A protocol-level JSON-RPC error is for malformed calls. A *tool-domain* failure travels as a normal result with `isError: true`.

**Why generic errors fail:** a uniform `"Operation failed"` gives the agent no basis to choose. The agent cannot choose among retry, alternative approach, escalation, or asking the user. Wasted retries on permanent errors come from unstructured errors. Abandoned recoverable errors come from the same cause.

**Error taxonomy the exam uses (memorize the four):**

| Category | Example | Retryable? | Agent's correct move |
| --- | --- | --- | --- |
| **Transient** | Timeout, service unavailable | Yes | Retry (with backoff) |
| **Validation** | Malformed input, bad ID format | No (as-is) | Fix the input, then retry |
| **Business** | Policy violation (refund cap) | **No** | Explain to the user / alternative workflow |
| **Permission** | Caller lacks access | No | Escalate / different credential path |

**Structured error metadata pattern:** return `errorCategory` (`transient` / `validation` / `permission` / business). Return an **`isRetryable` boolean**. Return a **human-readable description**. For business-rule violations, set retryable **false**. Include a **customer-friendly explanation** the agent can relay ("refunds above $500 require a supervisor"). The agent then communicates. It does not send many retries.

**Multi-agent link (Domain 5.3):** subagents recover **transient** failures locally. Only unresolvable errors propagate to the coordinator. They carry what was attempted and partial results. Also keep **access failure ≠ valid empty result**: an empty result set is a *successful* query with zero matches. It is never `isError: true`.

## S2. MCP servers in Claude Code: scoping and configuration (task 2.4)

**Two scopes, one exam distinction:**

| File | Scope | Shared? | Use for |
| --- | --- | --- | --- |
| **`.mcp.json`** (project root) | Project | Yes — committed to version control. Every teammate gets it on clone/pull | Shared team tooling (the team's Jira/GitHub/db servers) |
| **`~/.claude.json`** | User | No — personal machine only | Personal/experimental servers you try out |

**Environment-variable expansion** in `.mcp.json` — e.g. `${GITHUB_TOKEN}` in an env/header value. Teams can commit server *configuration*. Each member supplies their own **credentials from their environment**. The exam framing: how do you share the config **without committing secrets**? Use env-var expansion in project-scoped `.mcp.json`.

**Discovery model:** tools from **all configured MCP servers Claude Code discovers tools from all configured MCP servers at connection time. The agent can use them simultaneously.** to the agent. A project server and a personal server coexist in one session (official prep Exercise 2 has you verify exactly this).

**MCP resources as content catalogs:** expose issue summaries, documentation hierarchies, or database schemas as **resources**. Agents then get *visibility into what data exists* without extra turns on exploratory tool calls. Catalog = resource. Action = tool (consistent with the triad in §7–9 above).

**Two more 2.4 skills:**

- **Description competition with built-ins:** if your MCP tool description is weak, the agent may prefer a built-in (like Grep). It may ignore your more capable server tool. Fix this: **enrich the MCP tool description** with its capabilities and outputs. Description quality drives adoption. It is not only selection between similar tools.
- **Community vs custom servers:** prefer existing community MCP servers for standard integrations (e.g. Jira). Reserve custom server builds for team-specific workflows. (The exam does not cover how you deploy or host MCP server infrastructure.)

## S3. Built-in tools: selection criteria (task 2.5)

The agent-side toolkit the exam names — know each tool's *selection criterion*:

| Tool | Selects when… | Canonical example |
| --- | --- | --- |
| **Grep** | Search file **contents** for patterns | Find all callers of a function. Locate an error message. Find import statements |
| **Glob** | Match file **paths/names** by pattern | `**/*.test.tsx` — all test files by naming convention |
| **Read** | Load a file's full contents | Follow an import found by Grep |
| **Write** | Write a full file | Pair with Read when Edit cannot find a unique anchor |
| **Edit** | **Targeted modification via unique text matching** | Replace one function body |
| **Bash** | Run commands (tests, builds, git) | Execute the test suite |

**The Edit-failure pattern (tested):** Edit requires its anchor text to match **uniquely**. When it fails on non-unique matches, use this fallback:  **Read the full file, then Write the modified version**. Do not repeat Edit attempts with longer anchors.

**Incremental exploration pattern (tested):** build codebase understanding **incrementally**. Grep for entry points first. Then Read to follow imports and trace flows. Do not read all files first (too much context. See Domain 5.4). Tracing usage across wrapper modules: first identify all exported names, then search for each name across the codebase.

## S4. Supplement Q&A

**SQ1.** How does an MCP tool tell the agent it failed?
**A.** `isError: true` on the tool result, with details in content. A domain failure travels as a readable result, not a protocol error, so the model can reason about recovery.

**SQ2.** Name the four error categories and which is retryable.
**A.** Transient (retryable), validation (fix input first), business (not retryable — explain to user), permission (not retryable — escalate/credentials).

**SQ3.** What three fields make an error response "structured"?
**A.** `errorCategory`, `isRetryable` boolean, human-readable description (plus a customer-friendly explanation for business violations).

**SQ4.** Why do uniform "Operation failed" responses hurt agents?
**A.** They remove the basis for recovery decisions. This causes wasted retries on permanent errors and abandonment of recoverable ones.

**SQ5.** Where does a shared team MCP server go? A personal experiment?
**A.** Project-scoped `.mcp.json` (version-controlled). User-scoped `~/.claude.json`.

**SQ6.** How do you commit MCP config without committing tokens?
**A.** Environment-variable expansion (e.g. `${GITHUB_TOKEN}`) in `.mcp.json`. Each member supplies credentials through their environment.

**SQ7.** When are MCP tools from multiple servers available?
**A.** Discovered at connection time. Tools from all configured servers are available simultaneously.

**SQ8.** Agents keep making exploratory calls to learn what data exists. MCP fix?
**A.** Expose content catalogs (issue summaries, doc hierarchies, schemas) as **resources**.

**SQ9.** The agent prefers built-in Grep over your capable MCP search tool. Fix?
**A.** Enrich the MCP tool's description with its capabilities and outputs. Descriptions drive adoption.

**SQ10.** Grep vs Glob in one line?
**A.** Grep searches file *contents* for patterns. Glob matches file *paths/names*.

**SQ11.** Edit keeps failing on a non-unique match. Fallback?
**A.** Read the full file, then Write the modified version.

**SQ12.** Right way to build understanding of a large codebase?
**A.** Incrementally: Grep entry points → Read to follow imports/trace flows. Do not read everything first.

**SQ13.** A subagent's search times out once, then succeeds on retry. What reaches the coordinator?
**A.** Nothing. Subagents recover transient failures locally. Only unresolvable errors propagate (with attempts + partials).

**SQ14.** Empty search results: `isError: true` or not?
**A.** Not. A valid empty result is a successful query with zero matches. If you flag it as error, you corrupt retry logic.

## S5. Supplement checklist

- [ ] I can state what `isError` does and where error detail lives.
- [ ] I can classify an error into the four categories and derive the agent's move.
- [ ] I know `.mcp.json` vs `~/.claude.json` and the env-var expansion method.
- [ ] I know tools from all servers are discovered at connection time, simultaneously.
- [ ] I can pick resource-as-catalog vs tool-as-action.
- [ ] I can select among Read/Write/Edit/Bash/Grep/Glob and recite the Edit→Read+Write fallback.
