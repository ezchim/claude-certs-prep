---
title: Claude with Amazon Bedrock — Exam-Prep Study Notes (Primary Source) — Simplified Technical English
source: https://academy.claude.com/courses/claude-with-amazon-bedrock
disclaimer: Original study notes for exam prep — not official Anthropic or AWS material. Not a lesson transcript.
approx_length: ~5500–7000 words
deepened: 2026-08-23
cross_check: AWS Bedrock public docs (Converse, InvokeModel, inference profiles, IAM)
---

# Claude with Amazon Bedrock — Primary Study Notes

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Bedrock, Amazon Bedrock, Converse, InvokeModel, IAM, RAG, MCP, Vertex. Anthropic API, inference profile, boto3, Region, ARN, JSON, schema, streaming, prompt caching, temperature) are exceptions. They stay as written. Model IDs, Regions, and API fields change. Check the current AWS Bedrock and Anthropic docs before the exam.

> **Disclaimer:** These notes are **original** cert-prep notes. They align to the **public** Claude Academy course outline at [Claude with Amazon Bedrock](https://academy.claude.com/courses/claude-with-amazon-bedrock). They are **not** Academy lesson dumps. They are **not** verbatim course prose. Use current AWS Bedrock and Anthropic docs with these notes for live model IDs, Regions, and API fields. The official course is the source of truth for quizzes.

**Who it is for:** Developers who add Claude features to apps with Amazon Bedrock.

**Prerequisites (course):** Python, JSON basics, and an AWS account with Bedrock model access enabled.

**How to use:** Platform-specific sections are the most valuable differences from the Anthropic API and Vertex tracks. These sections cover enablement, IDs, IAM, Converse vs Invoke, and residency. Shared Claude skills still appear. These skills include prompting, tools, RAG, MCP, and agents. Know those skills in the Bedrock request shape.

---

## 1. Course map (public modules)

The public page has about 65 lessons and multiple quizzes. It groups topics roughly as:

1. **Working with the API** — auth, basic requests, conversation management, system prompts, structured output
2. **Prompt engineering** — strategies, evaluation frameworks, systematic testing
3. **Tool use** — JSON Schema tools, multi-turn tool loops, batch tool calling, built-ins
4. **RAG** — chunking, embeddings, BM25 hybrid search, multi-index, reranking, contextual retrieval
5. **Model Context Protocol (MCP)** — tools, resources, prompts. Servers and clients
6. **Agents** — Claude Code / computer-use style automation patterns on top of Bedrock access
7. **Advanced Claude features** — extended thinking, vision, prompt caching, streaming, temperature, structured extraction

For certification-style prep, give first priority to **access patterns, model IDs, IAM, Regions, and Converse vs InvokeModel**. Then know how shared Claude skills look when the transport is Bedrock.

**Main sequence for scenario questions:** Enable the model, then select the Region or profile → use IAM least privilege → use Converse (default). Alternatively, Invoke (native) → manage history yourself → add tools/RAG/eval as needed → select a residency-aware profile.

---

## 2. Access patterns on Bedrock

### 2.1 Enablement and credentials

1. In the Bedrock console (per account/Region), **request/enable** the Anthropic Claude models you need.
2. Call **Bedrock Runtime** with the standard AWS credential chain (env vars, shared config, instance role, SSO, and similar).
3. Create a runtime client in your Region. Example: `boto3.client("bedrock-runtime", region_name="us-east-1")`.

You do **not** send an Anthropic API key for common Bedrock Runtime calls. Billing and tenancy use AWS. Separate offerings like Claude Platform on AWS can differ. Know which product the question names.

**Common enablement failures:**

- You enable the model in Region A, but the client points at Region B
- You must have You enable the model, but you call a raw foundation ID when an inference profile.
- Credentials are valid for AWS in general, but the role does not have Bedrock invoke on the correct ARNs

### 2.2 Model IDs and inference profiles

Bedrock identifies Claude with IDs such as:

- Foundation-style: `anthropic.claude-…`
- **Cross-Region inference profiles**: often prefixed by geography, e.g. `us.anthropic.claude-…`, `eu.anthropic.…`, `global.anthropic.…`
- **Application inference profiles**: customer-created profiles (often for cost attribution or single-Region pinning)

**Exam-critical facts:**

- Not every model exists in every Region. If you call a missing model, you get confusing errors. The errors often say "does not exist" or "on-demand not supported."
- Newer Claude models often need an **inference profile ID**. A raw foundation model ID is not enough.
- Inference profiles route to Regions where the model is hosted. Check the Bedrock console under cross-Region inference or the model detail page for the exact ID.
- IAM must allow the resources you actually invoke. This includes foundation model ARNs **and** inference profile ARNs when you use both. Sample policies often use wildcards on Region in ARNs. Still prefer least privilege.

Always confirm current IDs on AWS model cards. Strings change when new Claude versions launch.

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
| Multi-turn / tools | Direct support for `messages`, `system`, `toolConfig` | You build the Anthropic Messages-style body yourself |
| Claude extras | Use `additionalModelRequestFields` for the parameters that Converse does not show | Full native Anthropic request fields |
| When to use | Default for chat, agents, tools | Embeddings/image gens, legacy, or native-only features |

**Default for exams:** Prefer **Converse** for conversational Claude on Bedrock. Use **InvokeModel** when you need the native Anthropic Messages payload. Use it also for certain tool types/fields or non-chat modalities.

AWS public docs emphasize this: Converse gives a consistent API for Bedrock models that support messages. Unique parameters still pass through a model-specific structure.

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

Read text from a path like `response["output"]["message"]["content"][i]["text"]`. Exact indexing depends on content blocks.

**Statelessness:** Bedrock does not remember prior turns for you. You must send the full alternating user/assistant history again on each call.

### 3.2 InvokeModel mental model (Claude)

The body typically includes:

- `anthropic_version` (Bedrock Claude Messages value such as `bedrock-2023-05-31` — confirm current docs)
- `max_tokens` (required in native Messages style)
- `messages` array
- optional `system`, `temperature`, tools, etc.

You parse the Anthropic-shaped response yourself. Content-Type is usually `application/json`. In boto3 the body is bytes (`json.dumps(...).encode()`).

### 3.3 Streaming

| Non-stream | Stream |
| --- | --- |
| `converse` | `converse_stream` |
| `invoke_model` | `invoke_model_with_response_stream` |

Stream calls use the same IAM family as non-stream, plus the **stream-specific** action. If you grant only `bedrock:InvokeModel`, streaming often fails with AccessDenied. Non-stream still works. This is a common exam error.

Stream responses arrive as event sequences. Your code puts the events back together (content deltas, message stop, and similar). Streaming changes delivery. It does not change the need for correct tools and history.

### 3.4 When Converse is not enough

Select InvokeModel when:

- You need embeddings or image generation models (not chat Converse)
- A Claude field is easier, or docs only show it in native Messages form
- You need to see the exact Anthropic-shaped body for debug
- You migrate existing Anthropic API code with minimal mapping (still change auth, version field, and endpoint)

Select Converse when:

- You build portable Bedrock chat/agents across model families
- You want native `toolConfig` and Guardrails integration patterns
- The team uses one Bedrock conversation API as the standard

---

## 4. IAM essentials (least privilege)

Both Converse and Invoke authorize with Bedrock inference actions:

- `bedrock:InvokeModel` — required for `InvokeModel` **and** `Converse`
- `bedrock:InvokeModelWithResponseStream` — required for stream variants

**Surprise exam fact:** An `AccessDeniedException` on `Converse` is often fixed when you grant `bedrock:InvokeModel`. Common policies do not use a separate `bedrock:Converse` permission. AWS docs state that Converse needs `bedrock:InvokeModel`. ConverseStream needs `bedrock:InvokeModelWithResponseStream`.

Also grant these when you use profiles or console selection:

- `bedrock:GetInferenceProfile` — run inference with an inference profile
- `bedrock:ListInferenceProfiles` — select profiles in the console.
- Access to the correct **model / inference profile ARNs**
- Any Guardrails / knowledge-base actions if the question includes those features

**Inference profile IAM trap:** When you use a cross-Region profile, the policy must allow invoke on:

1. The inference profile ARN (entry point)
2. The underlying foundation model ARNs in **each destination Region** the profile can route to

If you allow only the profile ARN, Lambda roles often fail. This is a frequent failure mode.

Least privilege checklist:

- Limit scope by account and Region where possible
- List specific model family ARNs rather than `*`
- Use separate roles for production and for experiments.
- You must have Deny dangerous adjacent actions your app does not need (model customization, marketplace purchase, and similar) unless they.
- For residency: combine profile choice with IAM Region conditions where policy requires it
---

## 5. Regions and data residency

- Select a **source Region** for the runtime client that supports your access pattern.
- Use **geo inference profiles** (`us.`, `eu.`, …) when you need availability across a geography.
- Use **global** profiles when they are allowed and residency is flexible.
- For **single-Region residency**, follow current AWS guidance. Use application inference profiles that point at one Region foundation model ARN. Use other in-Region options where they apply. Do **not** assume that cross-Region profiles keep data in one AZ/Region.

For compliance questions, match the profile geography to the policy. A model ID that worked in a tutorial is not enough.

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

**Claude Code on Bedrock note (conceptual):** If an agentic coding tool uses cross-Region profiles by default, single-Region residency can need an application profile. Point the tool at that profile ID. Public AWS blogs name this as a common enterprise error.

---

## 6. Claude features through Bedrock (course themes)

### 6.1 Conversations and system prompts

The ideas are the same as the Anthropic Messages API. Roles alternate. The system prompt sets durable behavior. Keep history tidy. Bedrock Converse uses field names like the `system` list and `inferenceConfig`.

**History care:** Trim old turns. Summarize when you need to. Never drop unpaired tool results in the middle of a loop.

### 6.2 Prompt engineering and evaluation

The course emphasis is structured prompting, systematic test sets, **model-based grading**, and **code-based grading**. For exams: evaluation is a workflow (cases → run → score → iterate). It is not a one-time informal check.

| Grading style | What it checks | Strength |
| --- | --- | --- |
| Code-based | Exact match, regex, JSON schema, unit asserts | Deterministic and cheap |
| Model-based | Rubric scored by another model call | Flexible for open answers |
| Human spot | Sample review | Calibration |

### 6.3 Tool use

Define tools with JSON Schema. Claude returns tool calls. Your app runs the tools and returns tool results in the next turn. Support multi-turn and batch tool calling patterns. On Converse, tools are set under `toolConfig`. Native-only Anthropic tool types can need InvokeModel or additional fields.

**Tool loop correctness:**

1. The model returns an assistant message with tool use block(s)
2. The app runs the tools
3. The app appends tool result(s) in the required role/shape
4. Call the model again with the full history
5. Repeat until a final text answer or a stop condition

Never invent tool results. Never change the order of tool_use / tool_result pairs without care.

### 6.4 RAG

Production RAG topics in the outline:

- Chunking strategies
- Embeddings (often a separate Bedrock embedding model via **Invoke**, not Converse)
- Lexical search (**BM25**) plus vector hybrid
- Multi-index architectures, reranking
- **Contextual retrieval** (add context to chunks before you embed or index)

Know *why* hybrid search exists. It covers exact keywords vs semantic match. This matters more than any single library.

Bedrock Knowledge Bases can appear in AWS-centric questions. Still map them to the same RAG concepts: ingest, chunk, embed, retrieve, generate, cite.

### 6.5 Advanced features

- **Extended thinking / reasoning** — extra tokens and cost. Current models use **adaptive thinking**. The model decides when and how deeply to think. You tune depth with **effort levels**. Claude 4.6 deprecates fixed `budget_tokens` thinking budgets. Newer frontier models remove them and return a 400 error. Check the specific model card on Bedrock before you use either shape.
- **Vision** — image content blocks
- **Prompt caching** — lower cost and latency for stable prefixes when the model/platform offers it
- **Streaming** — better UX for long answers
- **Structured extraction** — limit outputs (schemas / careful prompting). Temperature applies **only on older models**. Current frontier models remove the sampling parameters. Schemas give you determinism.

### 6.6 MCP and agents

MCP standardizes tools, resources, and prompts across clients and servers. The Agents section connects Bedrock-backed Claude to automation patterns (Claude Code, computer use). For cert prep: MCP is a **protocol for modular tool/resource exposure**. It is not a Bedrock-only API.

Agent patterns still apply: parallelize, chain, route, and debug with traces.
---

## 7. Comparison snapshot: Bedrock vs Anthropic API vs Vertex

| Topic | Anthropic API | Amazon Bedrock | Google Vertex (Claude) |
| --- | --- | --- | --- |
| Auth | Anthropic API key | AWS IAM / credentials | GCP ADC / service account |
| Client | `Anthropic` | `bedrock-runtime` (Converse/Invoke) | `AnthropicVertex` |
| Model ID style | Anthropic IDs | `anthropic.…` / `us.anthropic.…` | Vertex publisher IDs (e.g. `claude-sonnet-4-6`) |
| Version field | Headers / SDK | `anthropic_version` in the Invoke body. Converse covers much of this | `anthropic_version: vertex-2023-10-16` on raw HTTP |
| Unified multi-model chat | Messages | **Converse** (Bedrock-wide) | Messages via Vertex backend |
| Billing / contracts | Anthropic | AWS | Google Cloud |
| Residency knobs | Anthropic policies | Regions + inference profiles | global / us / eu / regional endpoints |
| Enablement | API key access | Bedrock model access per account/Region | Model Garden enablement |

**Select Bedrock when:** You need AWS-centric IAM, Converse portability across Bedrock models, an AWS compliance boundary. Alternatively, existing AWS networking/VPC patterns.

**Pick Vertex when:** GCP is the control plane. You need Google billing/residency or the Model Garden partner path.

**Pick Anthropic API when:** You need the fastest path to newest Anthropic-only features and the simplest key-based auth.

**Do not mix ID formats across clouds.** That fact alone answers several exam distractors.

---

## 8. Exam traps (Bedrock-specific)

| Trap | Reality |
| --- | --- |
| You look for a `bedrock:Converse` permission | Converse uses `bedrock:InvokeModel` |
|You must have You use a foundation ID when a profile.|Switch to an inference profile ID/ARN. |
| You grant the profile ARN only | Also grant destination FM ARNs for geo profiles |
|Store You assume history.|Runtime is stateless. Send messages again. |
| You use Converse for embeddings | Invoke embedding models separately |
| You use a cross-Region profile for a single-Region policy | Wrong residency tool. Pin with an application profile |
| You copy Vertex model IDs into Bedrock | Catalog strings differ |
| Streaming AccessDenied with Invoke granted | The policy misses `InvokeModelWithResponseStream` |
| "Enable once globally" | Enablement follows per account/Region patterns |
| You treat Guardrails as optional when the question requires them | Put Guardrails config on Converse when the question asks |

---

## 9. Exam tips

- Enable model access **and** use a Region/profile where the model exists.
- Prefer **Converse** unless the question forces native Invoke.
- Remember **stateless** history management.
- IAM for Converse ≈ `bedrock:InvokeModel` (plus the stream variant).
- For newer models, check **inference profile** IDs.
- For RAG answers, mention chunking and retrieval quality. Do not put the full PDF in the prompt for production.
- Eval = systematic grading. It is not a single anecdotal prompt.
- Distinguish advisory prompting from tool/MCP connections.
- For residency, name the profile strategy explicitly.
- For tool loops, insist on correct message ordering.

---

## 10. Minimal practice sketch (conceptual)

1. Enable Claude in the Bedrock console for your account/Region.
2. Create a runtime client with AWS creds.
3. Call `converse` with an inference profile model ID and one user message.
4. Add a second turn. Append the prior assistant output.
5. Add a tiny tool (e.g. calculator) via `toolConfig` and complete one tool loop.
6. Stream the same prompt with `converse_stream`.
7. Use a wrong Region/model ID once so you recognize the failure mode.
8. Sketch an IAM policy that includes profile + destination FM ARNs.
9. Write five eval cases. Grade with code asserts plus one model rubric.
---

## 11. Self-check Q&A (with answers)

**Q1.** A Converse call fails with AccessDenied. Invoke works with the same role. What is the likely cause? 
**A.** The role misses `bedrock:InvokeModel` (or the resource ARN for the new model/profile). Converse uses the same invoke permission family. If only streaming fails, check the stream action.

**Q2.** A model ID from the catalog works in docs but fails in your Region. What next? 
**A.** Confirm Regional availability. Switch to the correct **inference profile** ID for cross-Region routing.

**Q3.** Why does Claude not remember the previous turn on Bedrock? 
**A.** Runtime is stateless. You must send the full message history again.

**Q4.** When do you select InvokeModel over Converse for Claude? 
**A.** You need native Anthropic request/response fields or features that Converse does not show cleanly. Or you use non-chat models (embeddings, and similar).

**Q5.** What belongs in `inferenceConfig` on Converse? 
**A.** Shared knobs like `maxTokens`, `temperature`, and `topP`. Claude-specific extras often go in `additionalModelRequestFields`.

**Q6.** Name two RAG improvements beyond naive top-k embedding search. 
**A.** BM25 hybrid search. Reranking. Contextual retrieval. Better chunking / multi-index.

**Q7.** What is MCP in one sentence? 
**A.** A protocol that exposes tools, resources, and prompts to AI clients in a modular, reusable way.

**Q8.** How do you keep multi-turn tool use correct? 
**A.** Append assistant tool-call messages and user/tool-result messages in order. Never drop results in the middle of a loop.

**Q9.** Cross-Region profile invoke: IAM allows the profile ARN only. You still get AccessDenied. Why? 
**A.** You also need invoke permission on foundation model ARNs in each destination Region the profile routes to.

**Q10.** Which IAM action does ConverseStream require? 
**A.** `bedrock:InvokeModelWithResponseStream` (not only `InvokeModel`).

**Q11.** How do you pursue single-Region residency with newer Claude models that require profiles? 
**A.** Create an application inference profile that points at one Region foundation model ARN. Avoid multi-Region geo prefixes.

**Q12.** Why might embeddings use Invoke while chat uses Converse in the same app? 
**A.** Embeddings are not conversational Converse workloads. Invoke is the correct modality API.

**Q13.** What is the Bedrock Claude Invoke body version field commonly set to? 
**A.** `anthropic_version` such as `bedrock-2023-05-31` (confirm current docs).

**Q14.** Code-based vs model-based eval — when do you prefer code-based? 
**A.** When success is objectively checkable (JSON schema, exact fields, unit asserts).

**Q15.** Name three differences vs Vertex Claude. 
**A.** AWS IAM vs GCP ADC. `anthropic.` / geo profile IDs vs publisher IDs. Converse vs Messages-on-Vertex.

**Q16.** Guardrails on Converse — conceptual role? 
**A.** Platform safety/filters that you apply around model calls when you configure them. They are not a substitute for app authz.

**Q17.** What does batch tool calling mean at a high level? 
**A.** The model can request multiple tools in one turn. The app returns multiple results before the next model call.

**Q18.** Prompt caching helps most when? 
**A.** Stable long prefixes (system, tools, large docs) are reused across calls — when the model/platform supports it.

**Q19.** You enabled Claude in us-east-1 but call eu-west-1. What happens? 
**A.** Failure or an unavailable model. Enablement and availability are Region-sensitive. Fix the Region or use the appropriate profile.

**Q20.** What does contextual retrieval add? 
**A.** It adds surrounding document context to chunks before you embed or index. This improves retrieval quality.

**Q21.** Agent pattern that classifies then sends to specialized prompts/tools? 
**A.** Routing.

**Q22.** Why is temperature usually lower for tool argument generation? 
**A.** You need more deterministic structured arguments for reliable execution.

**Q23.** Application inference profile primary benefits? 
**A.** Cost attribution/tags and/or pinning of routing behavior (including single-Region sources) vs system geo profiles.

**Q24.** Is billing via Anthropic API key on common Bedrock Runtime?
**A.** No. Common Bedrock Runtime Claude calls use AWS credentials and AWS billing.
---

## 12. Review checklist (before exam)

- [ ] You distinguish enablement vs credentials vs the correct Region
- [ ] Foundation model ID vs geo profile vs application profile
- [ ] You memorize Converse vs Invoke decision criteria
- [ ] IAM: InvokeModel for Converse. Stream action for streams
- [ ] Profile IAM includes destination FM ARNs
- [ ] Stateless multi-turn history
- [ ] toolConfig loop ordering
- [ ] RAG hybrid / rerank / contextual retrieval vocabulary
- [ ] Eval = systematic cases + code/model grading
- [ ] Residency decision tree (global / geo / single-Region pin)
- [ ] Bedrock vs Vertex vs Anthropic API comparison table
- [ ] How you put stream events back together (conceptual)

---

## 13. Glossary

- **Bedrock Runtime** — API surface that invokes foundation models (`bedrock-runtime` client).
- **Converse** — Unified conversational inference API across Bedrock chat models.
- **ConverseStream** — Streaming variant of Converse.
- **InvokeModel** — Model-native body invocation API.
- **Inference profile** — Resource that routes inference (system geo or application).
- **Foundation model (FM) ARN** — Underlying model resource in a Region.
- **Application inference profile** — Customer-created profile for attribution and/or pinned routing.
- **Geo profile prefix** — e.g. `us.`, `eu.`, `global.` on profile IDs.
- **additionalModelRequestFields** — Field for model-specific Converse params that Converse does not show.
- **inferenceConfig** — Shared sampling/max token knobs on Converse.
- **toolConfig** — Converse tool definitions and tool choice settings.
- **Stateless inference** — No server-side chat memory between calls.
- **BM25** — Lexical ranking function used in hybrid RAG.
- **Contextual retrieval** — Prepend context to chunks before you embed or index.
- **MCP** — Model Context Protocol for tools/resources/prompts.
- **Guardrails** — Bedrock safety filters that you can configure around calls.
- **Knowledge Base** — Managed RAG retrieval component in the AWS Bedrock ecosystem.
- **Model-based grading** — You use a model to score outputs against a rubric.
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
2. Least-privilege IAM that you test with deny cases
3. Timeouts/retries for stream and non-stream
4. Eval suite with fixed cases before prompt tweaks
5. Logging that does not leak secrets
6. Cost monitors on tokens and profile tags
7. Fallback model or Region strategy that you document

---

## 15. Shared skill themes mapped to Bedrock fields

| Claude skill theme | Bedrock expression |
| --- | --- |
| System prompt | `system` on Converse |
| Temperature | `inferenceConfig.temperature` |
| Tools | `toolConfig` (or native tools on Invoke) |
| Multi-turn | App-managed `messages` array |
| Streaming | `converse_stream` / invoke stream |
| Structured output | Careful prompting + schema validation. Platform features where offered |
| Vision | Image content blocks in messages |
| Caching | Model/platform-specific cache fields when available |
| Agents | Your orchestration + optional Claude Code with Bedrock backend |

---

## 16. Study rhythm

Day 1: Sections 1–5 (access, Converse/Invoke, IAM, residency). Draw the comparison table from memory. 
Day 2: Sections 6–10 (features, traps, practice sketch). Run the failure triage aloud. 
Day 3: All Q&A closed-book. Checklist. Check glossary gaps only.

Rule to memorize: enable, profile, IAM, Converse by default, you manage the history, residency with purpose.

---

*Aligned to https://academy.claude.com/courses/claude-with-amazon-bedrock. Check live model IDs, Regions, and IAM examples in AWS Bedrock documentation before production use.*
---

## 17. Worked mini-scenarios (exam style)

**Scenario 1 — New Sonnet in prod Lambda** 
The team copies an old foundation model ID into `converse`. The platform does not support The error says on-demand throughput / use an inference profile.
**Answer path:** Look up the system or application inference profile ID for that Claude version. Update `modelId`. Extend IAM for profile + destination FM ARNs. Retest in the Lambda Region.

**Scenario 2 — EU data policy** 
Legal requires processing in the EU. An engineer uses a `global.…` profile because it worked in a tutorial. 
**Answer path:** Wrong. Prefer an `eu.` geo profile for EU multi-Region residency. Or use a single-Region EU application profile if policy demands one Region. Document which guarantee you actually have.

**Scenario 3 — Streaming chat widget** 
Non-stream chat works. Stream fails with AccessDenied. 
**Answer path:** Add `bedrock:InvokeModelWithResponseStream` on the same resources. Keep putting stream events back together in the client.

**Scenario 4 — Tool-using support agent** 
The model asks for two tools. The app returns only one result. The next call confuses roles. 
**Answer path:** Return all tool results paired correctly. Keep the assistant tool-use message. Then continue.

**Scenario 5 — RAG "hallucinated policy"** 
Naive top-k embedding retrieval misses exact clause IDs. 
**Answer path:** Add BM25/hybrid. Consider rerank. Improve chunking. Try contextual retrieval. Evaluate with fixed cases that contain those clause IDs.

**Scenario 6 — Multi-cloud migration** 
Port from Anthropic API to Bedrock. An engineer pastes `x-api-key` and the Anthropic base URL into AWS code. 
**Answer path:** Use AWS credentials + Bedrock Runtime. Map Messages fields to Converse, or keep Invoke with `anthropic_version` for Bedrock. Change model IDs to Bedrock catalog forms.

**Scenario 7 — Cost attribution** 
Finance cannot see which product line spent Bedrock Claude tokens. 
**Answer path:** Use application inference profiles with tags per product. Invoke via those profile ARNs. Build dashboards by tag.

**Scenario 8 — Eval disagreement** 
A prompt performs well on three selected examples. It fails silently in production. 
**Answer path:** Build a held-out eval set. Combine code-based checks for schema with model-based rubrics for quality. Gate deploys on eval thresholds.

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

**One-sentence rule:** On Bedrock, enable the model, call the correct profile with Invoke-family IAM, prefer Converse, manage your history yourself, and select residency with purpose.

---

## 19. Extra Q&A (stretch)

**Q25.** Can you use the same boto3 Converse code for Claude and another Bedrock chat model? 
**A.** Often yes for the message shape. That portability is a main benefit of Converse. Model-specific fields and capabilities still differ.

**Q26.** What breaks if you omit `max_tokens` on native Invoke Messages-style Claude? 
**A.** Request validation errors. You must have `max_tokens` in native Messages-style bodies.

**Q27.** Where do system prompts go on Converse vs a mistaken `role: system` inside messages? 
**A.** Use the Converse `system` parameter (list of system blocks). Do not invent a system role inside messages the way some other APIs do.

**Q28.** Why list `GetInferenceProfile` in some IAM policies? 
**A.** The AWS inference prerequisites docs require this action to run inference with an inference profile.

**Q29.** Hybrid search in one line? 
**A.** Combine lexical (BM25) and vector retrieval so exact tokens and semantic matches both surface.

**Q30.** Computer use / Claude Code themes in this course — what to remember? 
**A.** They are agentic automation patterns that can run with Bedrock-backed Claude. You still need AWS auth, model access, and verification discipline.

---

## 20. Study close

Redraw these items without notes: (1) Converse vs Invoke, (2) IAM actions for stream/non-stream, (3) profile types for residency, (4) the three-cloud comparison table. If you can do this, you are ready for Bedrock-differentiating exam items. Then refresh shared Claude skills (tools, RAG, MCP, eval). Use the same vocabulary as the Anthropic API and Vertex tracks. Only the transport and ID formats change.
