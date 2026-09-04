---
title: Claude with Amazon Bedrock
source: https://academy.claude.com/courses/claude-with-amazon-bedrock
disclaimer: Original study notes for exam prep — not official Anthropic or AWS material. Not a lesson transcript.
approx_length: ~5500–7000 words
deepened: 2026-08-23
cross_check: AWS Bedrock public docs (Converse, InvokeModel, inference profiles, IAM)
---

# Claude with Amazon Bedrock — Primary Study Notes

> **Disclaimer:** These are **original** cert-prep notes aligned to the **public** Claude Academy course outline at [Claude with Amazon Bedrock](https://academy.claude.com/courses/claude-with-amazon-bedrock). They are **not** Academy lesson dumps or verbatim course prose. Pair with current AWS Bedrock and Anthropic docs for live model IDs, Regions, and API fields. Completing the official course remains the source of truth for quizzes.

**Who it is for:** Developers adding Claude features to apps via Amazon Bedrock.

**Prerequisites (course):** Python, JSON basics, AWS account with Bedrock model access enabled.

**How to use:** Platform-specific sections (enablement, IDs, IAM, Converse vs Invoke, residency) are the highest-yield differentiators vs the Anthropic API and Vertex tracks. Shared Claude skills (prompting, tools, RAG, MCP, agents) still appear — know them in Bedrock's request shape.

---

## 1. Course map (public modules)

Public page (~65 lessons, multiple quizzes) groups topics roughly as:

1. **Working with the API** — auth, basic requests, conversation management, system prompts, structured output
2. **Prompt engineering** — strategies, evaluation frameworks, systematic testing
3. **Tool use** — JSON Schema tools, multi-turn tool loops, batch tool calling, built-ins
4. **RAG** — chunking, embeddings, BM25 hybrid search, multi-index, reranking, contextual retrieval
5. **Model Context Protocol (MCP)** — tools, resources, prompts; servers and clients
6. **Agents** — Claude Code / computer-use style automation patterns on top of Bedrock access
7. **Advanced Claude features** — extended thinking, vision, prompt caching, streaming, temperature, structured extraction

For certification-style prep, prioritize **access patterns, model IDs, IAM, Regions, Converse vs InvokeModel**, then know how shared Claude skills look when the transport is Bedrock.

**Spine for scenario questions:** Enable model → pick Region/profile → IAM least privilege → Converse (default) or Invoke (native) → manage history yourself → tools/RAG/eval as needed → residency-aware profile choice.

---

## 2. Access patterns on Bedrock

### 2.1 Enablement and credentials

1. In the Bedrock console (per account/Region), **request/enable** Anthropic Claude models you need.
2. Call **Bedrock Runtime** with the standard AWS credential chain (env vars, shared config, instance role, SSO, etc.).
3. Create a runtime client in your Region, for example `boto3.client("bedrock-runtime", region_name="us-east-1")`.

You do **not** send an Anthropic API key for classic Bedrock Runtime calls. Billing and tenancy ride on AWS. (Separate offerings like Claude Platform on AWS may differ — know which product the question names.)

**Common enablement failures:**

- Model enabled in Region A, client pointed at Region B
- Model enabled, but calling raw foundation ID when an inference profile is required
- Credentials valid for AWS generally, but role lacks Bedrock invoke on the right ARNs

### 2.2 Model IDs and inference profiles

Bedrock identifies Claude with IDs such as:

- Foundation-style: `anthropic.claude-…`
- **Cross-Region inference profiles**: often prefixed by geography, e.g. `us.anthropic.claude-…`, `eu.anthropic.…`, `global.anthropic.…`
- **Application inference profiles**: customer-created profiles (often for cost attribution or single-Region pinning)

**Exam-critical facts:**

- Not every model exists in every Region. Calling a missing model yields confusing "does not exist" / on-demand not supported style errors.
- Newer Claude models frequently require an **inference profile ID**, not only a raw foundation model ID.
- Inference profiles route to Regions where the model is hosted. Check the Bedrock console under cross-Region inference / the model detail page for the exact ID.
- IAM must allow the resources you actually invoke (foundation model ARNs **and** inference profile ARNs when both are in play). Wildcards on Region in ARNs are common in sample policies — still prefer least privilege.

Always confirm current IDs on AWS model cards — strings change as new Claude versions launch.

**Decision tree — which ID to pass:**

```text
Does the model require an inference profile for on-demand?
 YES → Use system geo profile (us./eu./global.) OR your application profile ARN/ID
 NO → Foundation model ID may work in that Region (still verify)

Need single-Region residency?
 → Do NOT use multi-Region geo profiles; use application profile pointing at one Region FM ARN
 (or other in-Region options documented by AWS for that Region)

Need cost attribution per team/app?
 → Application inference profile with tags
```
---

## 3. Converse API vs InvokeModel (high-yield)

| | **Converse / ConverseStream** | **InvokeModel / InvokeModelWithResponseStream** |
| --- | --- | --- |
| Shape | Unified Bedrock message schema | Model-native JSON body |
| Portability | Same code shape across many Bedrock chat models | Provider-specific payloads |
| Multi-turn / tools | First-class `messages`, `system`, `toolConfig` | You build Anthropic Messages-style body yourself |
| Claude extras | Use `additionalModelRequestFields` for knobs Converse does not surface | Full native Anthropic request fields |
| When to use | Default for chat, agents, tools | Embeddings/image gens, legacy, or native-only features |

**Default for exams:** prefer **Converse** for conversational Claude on Bedrock. Drop to **InvokeModel** when you need the native Anthropic Messages payload (certain tool types / fields) or non-chat modalities.

AWS public docs emphasize: Converse provides a consistent API for Bedrock models that support messages; unique parameters still pass through a model-specific structure.

### 3.1 Converse mental model

```text
client.converse(
 modelId=...,
 messages=[{role, content:[{text|...}]}],
 system=[...], # optional
 inferenceConfig={maxTokens, temperature, topP,...},
 toolConfig={...}, # optional
 additionalModelRequestFields={...} # Claude-specific extras
)
```

Read text from a path like `response["output"]["message"]["content"][i]["text"]` (exact indexing depends on content blocks).

**Statelessness:** Bedrock does not remember prior turns for you. You must resend the full alternating user/assistant history each call.

### 3.2 InvokeModel mental model (Claude)

Body typically includes:

- `anthropic_version` (Bedrock Claude Messages value such as `bedrock-2023-05-31` — confirm current docs)
- `max_tokens` (required in native Messages style)
- `messages` array
- optional `system`, `temperature`, tools, etc.

You parse the Anthropic-shaped response yourself. Content-Type is usually `application/json`; body is bytes in boto3 (`json.dumps(...).encode()`).

### 3.3 Streaming

| Non-stream | Stream |
| --- | --- |
| `converse` | `converse_stream` |
| `invoke_model` | `invoke_model_with_response_stream` |

Same IAM family as non-stream, with the **stream-specific** action. Granting only `bedrock:InvokeModel` is a classic reason streaming fails with AccessDenied while non-stream works.

Stream responses arrive as event sequences your code reassembles (content deltas, message stop, etc.). Streaming changes delivery, not the need for tools/history correctness.

### 3.4 When Converse is not enough

Choose InvokeModel when:

- You need embeddings or image generation models (not chat Converse)
- A Claude field is easier or only documented in native Messages form
- Debugging requires seeing the exact Anthropic-shaped body
- Migrating existing Anthropic API code with minimal mapping (still change auth, version field, endpoint)

Choose Converse when:

- Building portable Bedrock chat/agents across model families
- You want first-class `toolConfig` and Guardrails integration patterns
- Team standardizes on one Bedrock conversation API

---

## 4. IAM essentials (least privilege)

Both Converse and Invoke ultimately authorize with Bedrock inference actions:

- `bedrock:InvokeModel` — required for `InvokeModel` **and** `Converse`
- `bedrock:InvokeModelWithResponseStream` — required for stream variants

**Surprise exam fact:** An `AccessDeniedException` on `Converse` is often fixed by granting `bedrock:InvokeModel`, not a mythical separate `bedrock:Converse` permission in classic policies. AWS docs state Converse requires `bedrock:InvokeModel`; ConverseStream requires `bedrock:InvokeModelWithResponseStream`.

Also grant when using profiles / console selection:

- `bedrock:GetInferenceProfile` — run inference with an inference profile
- `bedrock:ListInferenceProfiles` — choose profiles in console
- Access to the correct **model / inference profile ARNs**
- Any Guardrails / knowledge-base actions if the question includes those features

**Inference profile IAM trap:** When using a cross-Region profile, policy must allow invoke on:

1. The inference profile ARN (entry point)
2. The underlying foundation model ARNs in **each destination Region** the profile can route to

Allowing only the profile ARN is a frequent Lambda role failure mode.

Least privilege checklist:

- Scope by account and Region where possible
- List specific model family ARNs rather than `*`
- Separate roles for prod vs experiment
- Deny dangerous adjacent actions your app does not need (model customization, marketplace purchase, etc.) unless required
- For residency: combine profile choice with IAM Region conditions where policy requires it
---

## 5. Regions and data residency

- Pick a **source Region** for the runtime client that supports your access pattern.
- Use **geo inference profiles** (`us.`, `eu.`, …) when you need availability across a geography.
- Use **global** profiles when allowed and residency is flexible.
- For **single-Region residency**, follow current AWS guidance: application inference profiles pointing at one Region foundation model ARN, or other in-Region options where applicable. Do **not** assume cross-Region profiles keep data in one AZ/Region.

Compliance questions: matching profile geography to policy beats "whatever model ID worked in the tutorial."

**Residency decision tree:**

```text
Flexible residency, maximize availability?
 → global inference profile (if model supports)

Must stay in US or EU broadly?
 → us. or eu. geo profile

Must stay in ONE Region?
 → application inference profile pinned to that Region FM ARN
 (+ IAM Region conditions as required by your org)

Unsure model available?
 → Check Bedrock console model card / ListFoundationModels / profile docs
```

**Claude Code on Bedrock note (conceptual):** if an agentic coding tool defaults to cross-Region profiles, single-Region residency may require creating an application profile and pointing the tool at that profile ID — a common enterprise gotcha called out in public AWS blogs.

---

## 6. Claude features through Bedrock (course themes)

### 6.1 Conversations and system prompts

Same ideas as Anthropic Messages API: roles alternate; system sets durable behavior; keep history tidy. Bedrock Converse uses field names like `system` list and `inferenceConfig`.

**History hygiene:** trim old turns, summarize when needed, never drop unpaired tool results mid-loop.

### 6.2 Prompt engineering and evaluation

Course emphasis: structured prompting, systematic test sets, **model-based grading** and **code-based grading**. For exams: evaluation is a workflow (cases → run → score → iterate), not a one-off vibe check.

| Grading style | What it checks | Strength |
| --- | --- | --- |
| Code-based | Exact match, regex, JSON schema, unit asserts | Deterministic, cheap |
| Model-based | Rubric scored by another model call | Flexible for open answers |
| Human spot | Sample review | Calibration |

### 6.3 Tool use

Define tools with JSON Schema; Claude returns tool calls; your app executes and returns tool results in the next turn. Support multi-turn and batch tool calling patterns. On Converse, tools live under `toolConfig`; native-only Anthropic tool flavors may need InvokeModel / additional fields.

**Tool loop correctness:**

1. Model returns assistant message with tool use block(s)
2. App runs tools
3. App appends tool result(s) in the required role/shape
4. Call model again with full history
5. Repeat until final text answer or stop condition

Never invent tool results. Never reorder tool_use / tool_result pairing carelessly.

### 6.4 RAG

Production RAG topics in the outline:

- Chunking strategies
- Embeddings (often a separate Bedrock embedding model via **Invoke**, not Converse)
- Lexical search (**BM25**) + vector hybrid
- Multi-index architectures, reranking
- **Contextual retrieval** (enrich chunks with context before embed/index)

Know *why* hybrid search exists (exact keywords vs semantic match) more than any single library.

Bedrock Knowledge Bases may appear in AWS-centric questions — still map to the same RAG concepts: ingest, chunk, embed, retrieve, generate, cite.

### 6.5 Advanced features

- **Extended thinking / reasoning** — extra tokens and cost. Current models use **adaptive thinking** (the model decides when/how deeply to think; depth tuned via **effort levels**) — fixed `budget_tokens` thinking budgets are **deprecated on 4.6 and removed (400 error) on newer frontier models**; verify the specific model card on Bedrock before using either shape
- **Vision** — image content blocks
- **Prompt caching** — reduce cost/latency for stable prefixes when available on the model/platform
- **Streaming** — better UX for long answers
- **Structured extraction** — constrain outputs (schemas / careful prompting); temperature applies **only on older models** — sampling params are removed on current frontier models, so schemas are the determinism lever

### 6.6 MCP and agents

MCP standardizes tools, resources, and prompts across clients/servers. Agents section connects Bedrock-backed Claude to automation patterns (Claude Code, computer use). For cert prep: MCP is a **protocol for modular tool/resource exposure**, not a Bedrock-only API.

Agent patterns still apply: parallelize, chain, route, debug with traces.
---

## 7. Comparison snapshot: Bedrock vs Anthropic API vs Vertex

| Topic | Anthropic API | Amazon Bedrock | Google Vertex (Claude) |
| --- | --- | --- | --- |
| Auth | Anthropic API key | AWS IAM / credentials | GCP ADC / service account |
| Client | `Anthropic` | `bedrock-runtime` (Converse/Invoke) | `AnthropicVertex` |
| Model ID style | Anthropic IDs | `anthropic.…` / `us.anthropic.…` | Vertex publisher IDs (e.g. `claude-sonnet-4-6`) |
| Version field | Headers / SDK | `anthropic_version` in Invoke body; Converse abstracts much | `anthropic_version: vertex-2023-10-16` on raw HTTP |
| Unified multi-model chat | Messages | **Converse** (Bedrock-wide) | Messages via Vertex backend |
| Billing / contracts | Anthropic | AWS | Google Cloud |
| Residency knobs | Anthropic policies | Regions + inference profiles | global / us / eu / regional endpoints |
| Enablement | API key access | Bedrock model access per account/Region | Model Garden enablement |

**Pick Bedrock when:** AWS-centric IAM, Converse portability across Bedrock models, AWS compliance boundary, existing AWS networking/VPC patterns.

**Pick Vertex when:** GCP is the control plane, Google billing/residency, Model Garden partner path.

**Pick Anthropic API when:** fastest path to newest Anthropic-only features and simplest key-based auth.

**Do not mix ID formats across clouds** — that alone answers several exam distractors.

---

## 8. Exam traps (Bedrock-specific)

| Trap | Reality |
| --- | --- |
| Looking for `bedrock:Converse` permission | Converse uses `bedrock:InvokeModel` |
| Using foundation ID when profile required | Switch to inference profile ID/ARN |
| Granting profile ARN only | Also grant destination FM ARNs for geo profiles |
| Assuming history is stored | Stateless; resend messages |
| Using Converse for embeddings | Invoke embedding models separately |
| Cross-Region profile for single-Region policy | Wrong residency tool; pin with application profile |
| Copying Vertex model IDs into Bedrock | Different catalog strings |
| Streaming AccessDenied with Invoke granted | Missing `InvokeModelWithResponseStream` |
| "Enable once globally" | Enablement is per account/Region patterns |
| Treating Guardrails as optional when question requires them | Wire Guardrails config on Converse when asked |

---

## 9. Exam tips

- Enable model access **and** use a Region/profile where the model exists.
- Prefer **Converse** unless the question forces native Invoke.
- Remember **stateless** history management.
- IAM for Converse ≈ `bedrock:InvokeModel` (+ stream variant).
- Newer models → check **inference profile** IDs.
- RAG answers should mention chunking + retrieval quality, not "just stuff the PDF in the prompt" for production.
- Eval = systematic grading, not single anecdotal prompts.
- Distinguish advisory prompting from tool/MCP plumbing.
- For residency, name the profile strategy explicitly.
- For tool loops, insist on correct message ordering.

---

## 10. Minimal practice sketch (conceptual)

1. Enable Claude in Bedrock console for your account/Region.
2. Create runtime client with AWS creds.
3. Call `converse` with an inference profile model ID and one user message.
4. Add a second turn by appending prior assistant output.
5. Add a tiny tool (e.g. calculator) via `toolConfig` and complete one tool loop.
6. Stream the same prompt with `converse_stream`.
7. Intentionally use a wrong Region/model ID once to recognize the failure mode.
8. Sketch an IAM policy that includes profile + destination FM ARNs.
9. Write five eval cases and grade with code asserts + one model rubric.
---

## 11. Self-check Q&A (with answers)

**Q1.** Converse call fails with AccessDenied though Invoke worked yesterday with same role. Likely cause? 
**A.** Missing `bedrock:InvokeModel` (or resource ARN for the new model/profile) — Converse uses the same invoke permission family. If only streaming fails, check stream action.

**Q2.** Model ID from the catalog works in docs but fails in your Region. What next? 
**A.** Confirm Regional availability; switch to the correct **inference profile** ID for cross-Region routing.

**Q3.** Why doesn't Claude remember the previous turn on Bedrock? 
**A.** Runtime is stateless; you must resend full message history.

**Q4.** When choose InvokeModel over Converse for Claude? 
**A.** Need native Anthropic request/response fields or features Converse does not expose cleanly; or non-chat models (embeddings, etc.).

**Q5.** What belongs in `inferenceConfig` on Converse? 
**A.** Shared knobs like `maxTokens`, `temperature`, `topP` — Claude-specific extras often go in `additionalModelRequestFields`.

**Q6.** Name two RAG improvements beyond naive top-k embedding search. 
**A.** BM25 hybrid search; reranking; contextual retrieval; better chunking / multi-index.

**Q7.** What is MCP in one sentence? 
**A.** A protocol for exposing tools, resources, and prompts to AI clients in a modular, reusable way.

**Q8.** How do you keep multi-turn tool use correct? 
**A.** Append assistant tool-call messages and user/tool-result messages in order; never drop results mid-loop.

**Q9.** Cross-Region profile invoke: IAM allows the profile ARN only. Still AccessDenied. Why? 
**A.** Also need invoke permission on foundation model ARNs in each destination Region the profile routes to.

**Q10.** Which IAM action does ConverseStream require? 
**A.** `bedrock:InvokeModelWithResponseStream` (not only `InvokeModel`).

**Q11.** How do you pursue single-Region residency with newer Claude models that require profiles? 
**A.** Create an application inference profile pointing at one Region foundation model ARN; avoid multi-Region geo prefixes.

**Q12.** Why might embeddings use Invoke while chat uses Converse in the same app? 
**A.** Embeddings are not conversational Converse workloads; Invoke is the right modality API.

**Q13.** What is the Bedrock Claude Invoke body version field commonly set to? 
**A.** `anthropic_version` such as `bedrock-2023-05-31` (confirm current docs).

**Q14.** Code-based vs model-based eval — when prefer code-based? 
**A.** When success is objectively checkable (JSON schema, exact fields, unit asserts).

**Q15.** Name three differences vs Vertex Claude. 
**A.** AWS IAM vs GCP ADC; `anthropic.` / geo profile IDs vs publisher IDs; Converse vs Messages-on-Vertex.

**Q16.** Guardrails on Converse — conceptual role? 
**A.** Platform safety/filters applied around model calls when configured — not a substitute for app authz.

**Q17.** Batch tool calling means what at a high level? 
**A.** Model can request multiple tools in one turn; app returns multiple results before the next model call.

**Q18.** Prompt caching helps most when? 
**A.** Stable long prefixes (system, tools, large docs) reused across calls — when supported for that model/platform.

**Q19.** You enabled Claude in us-east-1 but call eu-west-1. What happens? 
**A.** Failure or unavailable model — enablement and availability are Region-sensitive; fix Region or use appropriate profile.

**Q20.** What does contextual retrieval add? 
**A.** Enrich chunks with surrounding document context before embedding/indexing to improve retrieval quality.

**Q21.** Agent pattern that classifies then sends to specialized prompts/tools? 
**A.** Routing.

**Q22.** Why is temperature usually lower for tool argument generation? 
**A.** Need more deterministic structured arguments for reliable execution.

**Q23.** Application inference profile primary benefits? 
**A.** Cost attribution/tags and/or pinning routing behavior (including single-Region sources) vs system geo profiles.

**Q24.** Is billing via Anthropic API key on classic Bedrock Runtime? 
**A.** No — AWS credentials and AWS billing for classic Bedrock Runtime Claude calls.
---

## 12. Review checklist (before exam)

- [ ] Enablement vs credentials vs correct Region all distinguished
- [ ] Foundation model ID vs geo profile vs application profile
- [ ] Converse vs Invoke decision criteria memorized
- [ ] IAM: InvokeModel for Converse; stream action for streams
- [ ] Profile IAM includes destination FM ARNs
- [ ] Stateless multi-turn history
- [ ] toolConfig loop ordering
- [ ] RAG hybrid / rerank / contextual retrieval vocabulary
- [ ] Eval = systematic cases + code/model grading
- [ ] Residency decision tree (global / geo / single-Region pin)
- [ ] Bedrock vs Vertex vs Anthropic API comparison table
- [ ] Streaming event reassembly conceptual understanding

---

## 13. Glossary

- **Bedrock Runtime** — API surface for invoking foundation models (`bedrock-runtime` client).
- **Converse** — Unified conversational inference API across Bedrock chat models.
- **ConverseStream** — Streaming variant of Converse.
- **InvokeModel** — Model-native body invocation API.
- **Inference profile** — Resource that routes inference (system geo or application).
- **Foundation model (FM) ARN** — Underlying model resource in a Region.
- **Application inference profile** — Customer-created profile for attribution and/or pinned routing.
- **Geo profile prefix** — e.g. `us.`, `eu.`, `global.` on profile IDs.
- **additionalModelRequestFields** — Escape hatch for model-specific Converse params.
- **inferenceConfig** — Shared sampling/max token knobs on Converse.
- **toolConfig** — Converse tool definitions and tool choice settings.
- **Stateless inference** — No server-side chat memory between calls.
- **BM25** — Lexical ranking function used in hybrid RAG.
- **Contextual retrieval** — Prepend context to chunks before embed/index.
- **MCP** — Model Context Protocol for tools/resources/prompts.
- **Guardrails** — Bedrock safety filters configurable around calls.
- **Knowledge Base** — Managed RAG retrieval component in AWS Bedrock ecosystem.
- **Model-based grading** — Using a model to score outputs against a rubric.
- **Code-based grading** — Programmatic asserts on outputs.
- **AccessDeniedException** — IAM authorization failure from Bedrock APIs.

---

## 14. Deeper decision trees

### 14.1 API shape picker

```text
Is the workload conversational messages with optional tools?
 YES → Converse / ConverseStream
 NO → Is it embeddings / image / native-only payload?
 YES → InvokeModel / stream variant
 NO → Re-check modality docs
```

### 14.2 Failure triage

```text
AccessDenied?
 → Check action (Invoke vs InvokeWithResponseStream)
 → Check resource ARNs (profile + destination FMs)
 → Check account model enablement

Validation / model does not exist / on-demand not supported?
 → Wrong Region or need inference profile ID

Weird empty/partial answer?
 → maxTokens too low; stop reasons; tool loop incomplete

Tool loop spinning?
 → Missing/malformed tool results; unbounded max turns
```

### 14.3 Production readiness mini-rubric

1. Correct profile for residency and availability
2. Least-privilege IAM tested with deny cases
3. Timeouts/retries for stream and non-stream
4. Eval suite with fixed cases before prompt tweaks
5. Logging without secret leakage
6. Cost monitors on tokens and profile tags
7. Fallback model or Region strategy documented

---

## 15. Shared skill themes mapped to Bedrock fields

| Claude skill theme | Bedrock expression |
| --- | --- |
| System prompt | `system` on Converse |
| Temperature | `inferenceConfig.temperature` |
| Tools | `toolConfig` (or native tools on Invoke) |
| Multi-turn | App-managed `messages` array |
| Streaming | `converse_stream` / invoke stream |
| Structured output | Careful prompting + schema validation; platform features where offered |
| Vision | Image content blocks in messages |
| Caching | Model/platform-specific cache fields when available |
| Agents | Your orchestration + optional Claude Code with Bedrock backend |

---

## 16. Study rhythm

Day 1: Sections 1–5 (access, Converse/Invoke, IAM, residency). Draw the comparison table from memory. 
Day 2: Sections 6–10 (features, traps, practice sketch). Run the failure triage aloud. 
Day 3: All Q&A closed-book; checklist; skim glossary gaps only.

**Mantra:** Enable, profile, IAM, Converse-by-default, history-is-yours, residency-on-purpose.

---

*Aligned to https://academy.claude.com/courses/claude-with-amazon-bedrock. Verify live model IDs, Regions, and IAM examples in AWS Bedrock documentation before production use.*
---

## 17. Worked mini-scenarios (exam style)

**Scenario 1 — New Sonnet in prod Lambda** 
Team copies an old foundation model ID into `converse`. Error says on-demand throughput is not supported / use an inference profile. 
**Answer path:** Look up the system or application inference profile ID for that Claude version; update `modelId`; extend IAM for profile + destination FM ARNs; retest in the Lambda Region.

**Scenario 2 — EU data policy** 
Legal requires processing in the EU. Engineer uses `global.…` profile because it "just worked" in a tutorial. 
**Answer path:** Wrong. Prefer `eu.` geo profile for EU multi-Region residency, or a single-Region EU application profile if policy demands one Region. Document which guarantee you actually have.

**Scenario 3 — Streaming chat widget** 
Non-stream chat works; stream fails AccessDenied. 
**Answer path:** Add `bedrock:InvokeModelWithResponseStream` on the same resources; keep reassembling stream events in the client.

**Scenario 4 — Tool-using support agent** 
Model asks for two tools; app returns only one result; next call confuses roles. 
**Answer path:** Return all tool results paired correctly; preserve assistant tool-use message; then continue.

**Scenario 5 — RAG "hallucinated policy"** 
Naive top-k embedding retrieval misses exact clause IDs. 
**Answer path:** Add BM25/hybrid, consider rerank, improve chunking, try contextual retrieval; evaluate with fixed cases containing those clause IDs.

**Scenario 6 — Multi-cloud migration** 
Port from Anthropic API to Bedrock. Engineer pastes `x-api-key` and Anthropic base URL into AWS code. 
**Answer path:** Use AWS credentials + Bedrock Runtime; map Messages fields to Converse or keep Invoke with `anthropic_version` for Bedrock; change model IDs to Bedrock catalog forms.

**Scenario 7 — Cost attribution** 
Finance cannot see which product line spent Bedrock Claude tokens. 
**Answer path:** Application inference profiles with tags per product; invoke via those profile ARNs; dashboards by tag.

**Scenario 8 — Eval disagreement** 
Prompt looks great on three cherry-picked examples, fails silently in prod. 
**Answer path:** Build a held-out eval set; combine code-based checks for schema with model-based rubrics for quality; gate deploys on eval thresholds.

---

## 18. Quick reference card

| Need | Bedrock move |
| --- | --- |
| Default chat/agent | Converse |
| Stream tokens | ConverseStream + stream IAM |
| Native Anthropic body / embeddings | InvokeModel |
| Cross-Region availability | `us.` / `eu.` / `global.` profile |
| Single-Region pin | Application profile → one FM ARN |
| Auth | AWS credential chain, not Anthropic key |
| Tools | `toolConfig` (+ loop) |
| Claude-only knobs | `additionalModelRequestFields` |
| Portable vs Vertex | Different IDs and auth entirely |

**One-sentence mantra:** On Bedrock, enable the model, call the right profile with Invoke-family IAM, prefer Converse, own your history, and pick residency deliberately.

---

## 19. Extra Q&A (stretch)

**Q25.** Can you use the same boto3 Converse code for Claude and another Bedrock chat model? 
**A.** Often yes for the message shape — that portability is a Converse selling point — but model-specific fields and capabilities still differ.

**Q26.** What breaks if you omit `max_tokens` on native Invoke Messages-style Claude? 
**A.** Request validation errors — `max_tokens` is required in native Messages-style bodies.

**Q27.** Where do system prompts go on Converse vs a mistaken `role: system` inside messages? 
**A.** Use the Converse `system` parameter (list of system blocks); do not invent a system role inside messages the way some other APIs do.

**Q28.** Why list `GetInferenceProfile` in some IAM policies? 
**A.** Required to run inference with an inference profile per AWS inference prerequisites docs.

**Q29.** Hybrid search in one line? 
**A.** Combine lexical (BM25) and vector retrieval so exact tokens and semantic matches both surface.

**Q30.** Computer use / Claude Code themes in this course — what to remember? 
**A.** They are agentic automation patterns that can ride on Bedrock-backed Claude; still need AWS auth, model access, and verification discipline.

---

## 20. Study close

If you can redraw (1) Converse vs Invoke, (2) IAM actions for stream/non-stream, (3) profile types for residency, and (4) the three-cloud comparison table without notes, you are ready for the Bedrock-differentiating exam items. Then refresh shared Claude skills (tools, RAG, MCP, eval) using the same vocabulary as the Anthropic API and Vertex tracks — only the transport and ID formats change.
