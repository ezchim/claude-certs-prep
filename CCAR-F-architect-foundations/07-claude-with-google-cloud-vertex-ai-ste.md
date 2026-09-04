---
title: Claude with Google Cloud's Vertex AI — Exam-Prep Study Notes (Primary Source) — Simplified Technical English
source: https://academy.claude.com/courses/claude-with-google-cloud-s-vertex-ai
disclaimer: Original study notes for exam prep — not official Anthropic or Google Cloud material. Not a lesson transcript.
approx_length: ~5500–7000 words
deepened: 2026-08-23
cross_check: Google Cloud Vertex AI / Model Garden Claude public docs (ADC, endpoints, model IDs)
---

# Claude with Google Cloud's Vertex AI — Primary Study Notes

> **Note:** This edition follows the ASD-STE100 writing rules: simple present tense, active voice, one word = one meaning, sentences of 20–25 words or less. Technical names (Claude, Vertex, Model Garden, ADC, MCP, prompting, caching, RAG) are exceptions and stay as written. Model IDs, endpoints, and feature matrices change. Check the current Google Cloud and Anthropic docs before the exam.

> **Disclaimer:** These notes are **original** cert-prep notes. They align to the **public** Claude Academy course at [Claude with Google Cloud's Vertex AI](https://academy.claude.com/courses/claude-with-google-cloud-s-vertex-ai). They are **not** Academy lesson dumps. They are **not** verbatim course prose. Confirm live model IDs, endpoints, and feature matrices in Google Cloud and Anthropic docs. The official course is the source of truth for quizzes.

**Who it is for:** Developers who add Claude features with Google Cloud Vertex AI / Agent Platform.

**Prerequisites (course):** Python, JSON basics, and a Google Cloud project with Vertex AI access.

**How to use:** Platform-specific material is the most valuable difference. This includes Model Garden enablement, ADC, global/us/eu/regional endpoints, publisher model IDs, and feature parity caveats. Shared Claude skills overlap the Bedrock track. Know those skills in the Vertex Messages-shaped client.

---

## 1. Course map (public modules)

The public page has about 66 lessons and multiple quizzes. It organizes roughly as:

1. **Accessing Claude with the API** — auth, requests, multi-turn, system prompts, temperature, streaming, structured output
2. **Prompt engineering techniques** — strategies, evaluation, automated grading
3. **Tool use with Claude** — function calling, multi-turn tools, batch tools, built-ins
4. **RAG** — chunking, embeddings, BM25 hybrid, multi-index, reranking, contextual retrieval
5. **Model Context Protocol** — tools, resources, prompts. Server/client lifecycle
6. **Agents and workflows** — parallelization, chaining, routing, debugging
7. Related themes: vision, PDF, citations, prompt caching, Claude Code / computer-use style apps

Shared Claude skills overlap heavily with the Bedrock course. What is **distinct** for exams is **GCP setup, auth, regions/endpoints, model ID format, and API differences vs Anthropic and Bedrock**.

**Main sequence:** Enable in Model Garden, then set ADC. Alternatively, gcloud, then select the endpoint class (global/us/eu/regional) → `AnthropicVertex` Messages call → own history → tools/RAG/eval → know parity gaps vs Anthropic API.

---

## 2. Setup and authentication

### 2.1 Enable Claude in Model Garden

1. Open Vertex AI / Model Garden in Google Cloud Console. Select the correct project.
2. Search **Anthropic** / Claude.
3. **Enable** the model(s) you need for the project (if they are not already enabled).
4. Confirm the model is available for the location/endpoint style you plan to use.

If you skip enablement, the docs work, but your project fails. This is a common failure.

### 2.2 gcloud and Application Default Credentials (ADC)

Typical local setup:

```bash
gcloud init
gcloud auth login
gcloud config set project YOUR_PROJECT_ID
gcloud auth application-default login
```

The Anthropic **Vertex** SDK uses Google auth (ADC / service account / workload identity). You do **not** pass an Anthropic API key for Vertex-backed calls.

Production: prefer **service accounts** with least-privilege Vertex AI permissions. Do not use long-lived user ADC on servers. Use workload identity on GKE/Cloud Run where possible.

### 2.3 Install SDK

Python example: `pip install -U "anthropic[vertex]"`. 
TypeScript: `@anthropic-ai/vertex-sdk`.

Client pattern:

```python
from anthropic import AnthropicVertex

client = AnthropicVertex(project_id="YOUR_PROJECT_ID", region="global")
message = client.messages.create(
 model="claude-sonnet-4-6", # example — check current IDs
 max_tokens=1024,
 messages=[{"role": "user", "content": "Hello"}],
)
```

Env vars commonly involved: project id and region (`ANTHROPIC_VERTEX_PROJECT_ID`, `CLOUD_ML_REGION`, etc., depending on SDK docs).

**Auth decision tree:**

```text
Local laptop prototyping?
 → gcloud auth application-default login (user ADC)
Cloud Run / GKE / GCE prod?
 → Service account + workload identity / attached SA
CI from GitHub without long-lived keys?
 → Workload identity federation patterns
Still tempted to set ANTHROPIC_API_KEY for Vertex?
 → Wrong product path — that is Anthropic API, not Vertex
```
---

## 3. How the Vertex Claude API differs from Anthropic API

The **Messages** shape is nearly the same. The platform differences are important:

| Topic | Anthropic API | Claude on Vertex / Agent Platform |
| --- | --- | --- |
| Auth | `x-api-key` / Anthropic key | Google Cloud credentials |
| Model location | In request body `model` | Often in **URL path** for raw HTTP. SDK still takes `model=` |
| Version marker | API version headers | Body field `anthropic_version` = `vertex-2023-10-16` on raw HTTP |
| Endpoint | `api.anthropic.com` | `…-aiplatform.googleapis.com` or multi-region/global hosts |
| Billing / ToS | Anthropic | Google Cloud partner model terms |
| Some platform features | Batches, Files, certain server tools, etc. | Check "supported / not supported" lists — they differ |

**Raw HTTP reminder:** the model goes in the publisher path 
`/v1/projects/{PROJECT}/locations/{LOCATION}/publishers/anthropic/models/{MODEL_ID}:rawPredict` 
(and streamPredict for streaming where applicable) 
and the JSON body includes `anthropic_version`, `messages`, `max_tokens`, …

**SDK reminder:** `AnthropicVertex` lets you keep writing `messages.create(...)` with familiar fields. The SDK handles Vertex routing.

**Parity caveat rule:** Same Claude family. The feature surface is not always the same. If a question says "on Vertex," do not assume every Anthropic-only admin/files/batches/server-tool feature exists.

---

## 4. Regions and endpoints (high-value exam topic)

Agent Platform / Vertex offers three endpoint styles (public Google/Anthropic docs):

### 4.1 Global

- `region="global"`
- Dynamic routing for availability across supported regions with capacity
- Usually **standard pricing** (no residency premium)
- Best default when data residency is flexible
- Pay-as-you-go oriented. Provisioned throughput needs regional capacity

Hosts conceptually use global AI Platform endpoints (confirm current host strings in docs).

### 4.2 Multi-region (`us` or `eu`)

- Routes within that geography
- Google docs often cite these example hosts: `aiplatform.us.rep.googleapis.com` and `aiplatform.eu.rep.googleapis.com`.
- Data residency at **geo** granularity with better availability than a single region
- Typically a **pricing premium** vs global (Google commonly cites a premium of about 10 percent for newer models from 4.5 onward. Confirm the pricing pages).

### 4.3 Specific regional (e.g. `us-east5`, `europe-west1`)

- Strict single-region routing
- Needed for hard residency, some enterprise controls, **provisioned throughput**
- Also typically premium vs global
- Model availability can lag. Newest models may appear first on global / key regions

**Decision flow for exams:**

```text
1. Flexible residency → global
2. Must stay in US or EU broadly → us / eu
3. Must pin one region or use provisioned throughput → specific region
4. Always verify the chosen Claude version is enabled and available in that location
```

Common exam error: You use `global` when the compliance question requires EU residency. Availability is not the same as residency.

**Common exam error:** You expect provisioned throughput on global. Public materials emphasize provisioned capacity on regional endpoints.

---

## 5. Model IDs on Vertex

Vertex publisher IDs look like Anthropic names. They follow **Google's catalog strings**. Examples (illustrative — always check live docs):

- `claude-sonnet-4-6`
- `claude-haiku-4-5@20251001` (some IDs include date suffixes)
- `claude-opus-4-6`

They are **not** Bedrock's `anthropic.claude-…` or `us.anthropic.…` inference profile strings. They can also differ slightly from Anthropic API aliases.

Lifecycle (deprecated/retired) dates on partner platforms can differ from Anthropic's own schedule. Exam answers must say "check Google's Claude model page." Do not invent dates.

**ID comparison cheat:**

| Platform | Example flavor |
| --- | --- |
| Anthropic API | Anthropic model aliases / versioned IDs |
| Bedrock | `anthropic.claude-…` or `us.anthropic.claude-…` |
| Vertex | `claude-sonnet-4-6` publisher ID |

Copy-paste across clouds is an intentional distractor.
---

## 6. Feature support (what to expect)

Typically available on Vertex Claude (confirm per model):

- Messages API (chat)
- Tool use (including several Anthropic tool types, with caveats)
- Prompt caching, thinking/extended reasoning where offered
- Vision and documents (check the payload size limits. Vertex may cap the request size, for example tens of MB).
- Citations, structured outputs where documented
- Streaming via SDK/stream APIs

The surface is often **not** the same as the full Anthropic API. Public docs identify certain Files and URL helpers, server-side tools, Message Batches, and Admin APIs, and some managed-agent infrastructure. If a question says "on Vertex," do not assume every Anthropic-only feature exists.

Context windows: newer Claude generations may offer large windows (e.g. 1M-class on listed models). Older ones may be smaller. Prefer "check model card" over memorizing every number.

**Parity decision rule:**

```text
Feature required by product?
 → Check Vertex/Agent Platform support matrix for THAT model
 → If missing, either redesign, use Anthropic API (if policy allows), or wait for parity
Never answer "Claude can do X on Anthropic API, therefore Vertex too" without checking
```

---

## 7. Course skill themes (shared with Bedrock track)

### 7.1 Multi-turn and system prompts

Alternate user/assistant. Keep history coherent. Put durable behavior in `system`. HTTP calls are stateless. Your app stores history.

### 7.2 Prompt engineering and eval

Use XML-ish structure, examples, and output control. Build **test sets**. Grade with model-based and code-based scorers. Production quality is an iteration loop. It is not a single clever prompt.

### 7.3 Tool use

JSON Schema tools → model tool calls → your execution → tool results back into messages. Multi-turn and batch patterns matter for agents.

### 7.4 RAG

Chunk → embed → retrieve (vector + **BM25** hybrid) → optionally rerank → contextual retrieval techniques → answer with citations when available. The conceptual pipeline is the same as other Claude cloud courses. Storage/embeddings may use GCP services (Vertex AI embeddings, Vector Search, etc.).

### 7.5 MCP

Define tools, resources, prompt templates. Run MCP servers/clients. Integrate into apps. Protocol knowledge transfers across Anthropic API, Bedrock, and Vertex transports.

### 7.6 Agents and workflows

Patterns to memorize:

| Pattern | Idea | Use when |
| --- | --- | --- |
| Parallelization | Fan-out independent subtasks | Embarrassingly parallel work |
| Chaining | Output of step N feeds step N+1 | Pipelines with dependencies |
| Routing | Classify then send to specialized prompts/tools | Mixed intent traffic |
| Debugging | Log traces, constrain tools, evaluate intermediate steps | Agents that behave without consistency |

---

## 8. Vertex vs Bedrock vs Anthropic API (comparison sheet)

| Dimension | Anthropic API | Bedrock | Vertex AI Claude |
| --- | --- | --- | --- |
| Cloud identity | Anthropic account | AWS IAM | GCP IAM / ADC |
| Primary SDK | `Anthropic` | AWS SDK + Converse | `AnthropicVertex` |
| Model ID flavor | Anthropic IDs | `anthropic.` / geo profiles | Publisher IDs (`claude-…`) |
| Unified multi-model chat API | Messages | **Converse** (Bedrock-wide) | Messages via Vertex backend |
| Version marker | Anthropic headers | `bedrock-2023-05-31` style on Invoke | `vertex-2023-10-16` on raw HTTP |
| Residency control | Anthropic options | Regions + inference profiles | global / us / eu / regional |
| Enterprise packaging | Anthropic contracts | AWS Marketplace / Bedrock | Google Cloud / Model Garden |
| Enablement | API access | Bedrock model access | Model Garden enable |

Select Vertex when: GCP is already the control plane, you need Google billing/residency, or org policy mandates Vertex. 
**Pick Bedrock when:** AWS-centric IAM, Converse portability across Bedrock models, AWS compliance boundary. 
**Pick Anthropic API when:** fastest path to newest Anthropic-only features and simplest key-based auth.

---

## 9. Exam traps (Vertex-specific)

| Trap | Reality |
| --- | --- |
| Using Anthropic API key with AnthropicVertex | Wrong auth — need GCP ADC/SA |
| Pasting Bedrock model IDs | Wrong catalog |
| Choosing global under EU residency mandate | Wrong endpoint class |
| Assuming provisioned throughput on global | Typically regional |
| Assuming full Anthropic feature parity | Check support matrix |
| Forgetting Model Garden enable | Project-level enablement required |
| Putting model only in body for raw HTTP | Model often in URL path. Body needs `anthropic_version` |
| Believing Vertex stores chat history | Stateless. App resends messages |
| Ignoring request size limits for PDFs/images | Vertex may enforce payload caps |
| Using `europe-west1` for a brand-new model only on global | Regional lag |
---

## 10. Exam tips

- Setup order: enable model → gcloud/ADC → `AnthropicVertex(project, region)`.
- Memorize **`anthropic_version: vertex-2023-10-16`** for raw HTTP.
- Region choice is a **compliance and availability** question. It is not a minor style question.
- Do not paste Bedrock model IDs into Vertex or vice versa.
- Stateless multi-turn history is still your job.
- For "feature X on Vertex?" — prefer "supported if listed for Agent Platform / that model." Do not use Anthropic-API assumptions.
- Agent workflow vocabulary: parallelize, chain, route.
- RAG answers must cite hybrid retrieval and evaluation. Do not cite embeddings only.
- Pricing premium for us/eu/regional vs global is a known exam-adjacent fact. Confirm current numbers on pricing pages.
- Service accounts in production are better than user ADC on servers.

---

## 11. Minimal practice sketch (conceptual)

1. Enable a Claude model in Model Garden.
2. Configure gcloud project + ADC.
3. Call `AnthropicVertex` with `region="global"` and a short Messages request.
4. Repeat with `region="us"` and note endpoint/pricing implications.
5. Build a two-turn chat. Append the assistant output.
6. Add one custom tool and complete a tool loop.
7. Sketch a tiny RAG path (chunk → retrieve → answer) and an eval rubric with 5 fixed cases.
8. Attempt a raw HTTP `rawPredict` once to see `anthropic_version` and path-based model ID.
9. Intentionally use a Bedrock-style model ID once so you recognize the failure mode.
10. Document which features you rely on. Verify each against the Vertex support list.

---

## 12. Self-check Q&A (with answers)

**Q1.** How does local auth usually work for AnthropicVertex? 
**A.** Google Application Default Credentials after `gcloud auth application-default login` (or a service account in prod). It is not an Anthropic API key.

**Q2.** What goes in the raw request body instead of/in addition to Anthropic headers? 
**A.** `anthropic_version` set to `vertex-2023-10-16`. The model often lives in the URL path.

**Q3.** When do you select `region="eu"` instead of `global`? 
**A.** When you need EU geographic data residency with multi-region availability inside the EU. You accept possible premium pricing.

**Q4.** Why might a brand-new Claude model fail in `europe-west1` but work with `global`? 
**A.** Regional availability lags. Newest models often appear on global or select regions first.

**Q5.** Name one difference vs Bedrock model IDs. 
**A.** Vertex uses publisher IDs like `claude-sonnet-4-6`. Bedrock uses `anthropic.claude-…` or `us.anthropic.…` profiles.

**Q6.** Is conversation history stored by Vertex between calls? 
**A.** No. Your application must resend messages each turn (unless you build your own session store).

**Q7.** What agent pattern sends different query types to different tools/prompts? 
**A.** Routing.

**Q8.** Give two reasons Converse knowledge still helps even if you deploy on Vertex. 
**A.** Shared Claude concepts (tools, system prompts, eval). Comparison questions on cloud exams often contrast Bedrock's Converse with Vertex's Messages-on-GCP approach.

**Q9.** When is a specific regional endpoint required? 
**A.** Hard single-region residency and/or provisioned throughput (per public guidance).

**Q10.** What is Model Garden's role? 
**A.** Discovery and enablement of partner models (including Claude) for your GCP project.

**Q11.** Why prefer service accounts in production? 
**A.** Least privilege, rotatable, no interactive user ADC on servers, fits workload identity.

**Q12.** Streaming on Vertex — does it change tool-loop correctness rules? 
**A.** No. You still assemble the final message and keep tool_use/tool_result pairing correct.

**Q13.** Name two features that may differ from Anthropic API on Vertex. 
**A.** Examples: Message Batches, some Files/URL helpers, some server tools, Admin APIs. Always verify current lists.

**Q14.** Hybrid RAG means what? 
**A.** Combining lexical (BM25) and vector retrieval before generation.

**Q15.** Parallelization vs chaining? 
**A.** Parallelization fans out independent work. Chaining sequences dependent steps.

**Q16.** What host flavor is associated with US multi-region? 
**A.** Often `aiplatform.us.rep.googleapis.com` style hosts (confirm docs).

**Q17.** Can you use Claude on Vertex without enabling it in the project? 
**A.** No. You must have Enablement in Model Garden / project access.

**Q18.** Where does `max_tokens` go? 
**A.** In the Messages request body (SDK parameter). It is the required upper bound on generation.

**Q19.** Temperature for structured extraction? 
**A.** On older models (4.6 and earlier): lower for more deterministic outputs. Current frontier models have removed sampling params entirely. Rely on schemas/structured outputs for determinism.

**Q20.** What should you check before assuming 1M context? 
**A.** The specific model card on Vertex. Not all models share the same window.

**Q21.** Workload identity purpose in one line? 
**A.** Let cloud workloads obtain GCP credentials without long-lived downloaded keys.

**Q22.** Citations feature — exam caution? 
**A.** Available where documented for that model/platform. Do not invent citations support.

**Q23.** Why might PDF/image requests fail despite valid auth? 
**A.** Payload size limits or unsupported media handling on Vertex path.

**Q24.** Bedrock `additionalModelRequestFields` analog on Vertex? 
**A.** Not the same API. On Vertex you generally pass Anthropic Messages fields the SDK/API supports. Extras follow Anthropic/Vertex docs, not Bedrock Converse naming.
---

## 13. Review checklist (before exam)

- [ ] Model Garden enablement steps
- [ ] ADC vs service account vs Anthropic API key (which belongs where)
- [ ] global vs us/eu vs regional decision tree
- [ ] Raw HTTP: path model ID + `anthropic_version: vertex-2023-10-16`
- [ ] Publisher ID format vs Bedrock IDs
- [ ] Feature parity caveats vs Anthropic API
- [ ] Stateless history + tool loop ordering
- [ ] Agent patterns: parallelize, chain, route
- [ ] RAG hybrid / contextual retrieval vocabulary
- [ ] Eval = systematic code + model grading
- [ ] Three-cloud comparison table from memory
- [ ] Provisioned throughput tied to regional endpoints

---

## 14. Glossary

- **Model Garden** — GCP catalog to discover/enable partner and Google models.
- **Agent Platform** — Google Cloud surface for agentic / partner model serving (naming in docs evolves).
- **ADC** — Application Default Credentials for Google auth.
- **AnthropicVertex** — Official Anthropic SDK client for Vertex-backed Claude.
- **Publisher model ID** — e.g. `claude-sonnet-4-6` under publisher `anthropic`.
- **rawPredict** — Vertex raw prediction endpoint for Anthropic-shaped JSON bodies.
- **anthropic_version** — Body field. Vertex uses `vertex-2023-10-16` on raw HTTP.
- **Global endpoint** — Dynamic multi-region routing for availability. Flexible residency.
- **Multi-region endpoint** — `us` or `eu` geo routing with geo residency.
- **Regional endpoint** — Single-region routing. Residency pin. Provisioned throughput.
- **Provisioned throughput** — Reserved capacity (typically regional).
- **Workload identity** — Cloud workload auth without static keys.
- **Feature parity** — Whether Vertex supports the same capabilities as Anthropic API for a feature.
- **BM25** — Lexical retrieval component in hybrid RAG.
- **Contextual retrieval** — Context-enriched chunks before embedding.
- **Routing / chaining / parallelization** — Core agent workflow patterns.
- **Code-based grading** — Programmatic eval asserts.
- **Model-based grading** — Rubric scoring via another model call.

---

## 15. Worked mini-scenarios

**Scenario 1 — Wrong key** 
An engineer sets `ANTHROPIC_API_KEY` and wonders why AnthropicVertex fails in a VPC-SC GCP project. 
**Answer:** Use GCP credentials (ADC/SA). Anthropic API keys are a different product path.

**Scenario 2 — Compliance** 
Policy: process in EU. Code uses `region="global"` for cheaper tokens. 
**Answer:** Switch to `eu` multi-region or a specific EU regional endpoint. Accept premium if applicable. Verify model availability.

**Scenario 3 — New model lag** 
Opus is brand-new on global. The regional endpoint 404s/unavailable. 
**Answer:** Use global (if residency allows) or wait/enable in the needed region. Do not invent Bedrock-style profile prefixes.

**Scenario 4 — Raw HTTP debugging** 
SDK works. Hand-rolled curl fails. 
**Answer:** Check bearer token from gcloud, URL location segment, publisher path model ID, and `anthropic_version` in JSON.

**Scenario 5 — Parity miss** 
App depends on an Anthropic-only batch API. Port to Vertex breaks. 
**Answer:** Check the support matrix. Redesign with your own queue, or stay on Anthropic API if policy allows.

**Scenario 6 — Agent design** 
Mixed intents: reset password vs order status vs FAQ. 
**Answer:** Routing pattern to specialized prompts/tools. Evaluate each branch.

**Scenario 7 — RAG on GCP** 
Exact SKU strings missed by vector-only search. 
**Answer:** Hybrid BM25 + vectors. Rerank. Use contextual retrieval. Use fixed eval cases with those SKUs.

**Scenario 8 — Prod auth hardening** 
A developer commits user ADC JSON to the repo for Cloud Run. 
**Answer:** Stop. Use the attached service account or workload identity. Rotate the exposed credentials.

---

## 16. Deeper endpoint comparison table

| | Global | Multi-region us/eu | Specific regional |
| --- | --- | --- | --- |
| Residency | Flexible / none guaranteed | Geo-level | Single region |
| Availability | Highest (dynamic) | High within geo | Depends on one region |
| Typical price | Baseline | Premium (often ~10% on newer models) | Premium |
| Provisioned throughput | Generally no | Check docs | Yes (typical home) |
| Best for | Most apps without residency rules | US/EU compliance + HA | Hard pin / reserved capacity |

---

## 17. Study rhythm

Day 1: Setup, auth, endpoints, model IDs. Draw residency decision tree. 
Day 2: Feature parity, shared skills, comparison vs Bedrock/Anthropic. 
Day 3: Q&A closed-book + scenarios + checklist.

**Rule to memorize:** Enable in Garden, authenticate with Google, select the endpoint for residency, use publisher IDs, assume Messages-not-Converse, verify parity before you promise features.
---

## 18. Mapping shared Claude skills to Vertex request fields

| Skill theme | Vertex expression |
| --- | --- |
| System prompt | top-level `system` on Messages |
| Temperature | `temperature` request field |
| Tools | `tools` / tool choice fields (Anthropic shape) |
| Multi-turn | App-managed `messages` |
| Streaming | SDK stream helpers / streamPredict |
| Structured output | Schema prompting + validation. Platform features if listed |
| Vision / PDF | Content blocks. Watch size limits |
| Caching | Prompt cache fields where supported on Vertex model |
| Thinking | Extended thinking params where supported |
| Agents | Your orchestration (parallel/chain/route) + optional Claude Code |

Contrast with Bedrock naming: Bedrock Converse uses `inferenceConfig`, `toolConfig`, `additionalModelRequestFields`. Vertex stays closer to Anthropic Messages names via `AnthropicVertex`.

---

## 19. Failure triage tree

```text
Auth error / 401 / 403?
 → ADC present? Correct project? SA roles include Vertex AI user permissions?
 → Model enabled in Model Garden for this project?

404 / model not found?
 → Wrong publisher ID string?
 → Model not available in chosen location?
 → Accidentally used Bedrock-style ID?

Empty/truncated output?
 → max_tokens too low; stop reasons; tool loop incomplete

Tool loop errors?
 → Missing tool_result; role ordering; schema mismatch

Residency audit fail?
 → global used under geo mandate → move to us/eu/regional
```

---

## 20. Extra self-check Q&A

**Q25.** What does `locations/global` mean in a Vertex path? 
**A.** Use of the global endpoint location rather than a single region like `us-east5`.

**Q26.** Why might finance care about us/eu endpoints? 
**A.** Premium pricing vs global for newer models. Cost and compliance both matter.

**Q27.** Chaining example in one line? 
**A.** Extract entities → fetch records → draft answer. Each step feeds the next.

**Q28.** Is MCP Vertex-specific? 
**A.** No. MCP is transport-agnostic. Vertex is one place you might host or consume tools alongside Claude.

**Q29.** Computer use / Claude Code in this course — takeaway? 
**A.** Agentic apps can run with Vertex-backed Claude. You still need GCP auth, enablement, and verification.

**Q30.** Best first endpoint for a startup with no residency rules? 
**A.** Global — availability and baseline pricing. Then revisit if compliance appears.

**Q31.** How do you pass project id to the SDK? 
**A.** `AnthropicVertex(project_id=..., region=...)` and/or documented env vars.

**Q32.** What is wrong with storing only the last user message each turn? 
**A.** You lose assistant/tool history. This breaks multi-turn coherence and tool loops.

**Q33.** Code-based grading example for a Vertex app? 
**A.** Assert JSON schema, required fields, and forbidden PII patterns on outputs in CI.

**Q34.** Model-based grading example? 
**A.** Rubric prompt scoring helpfulness/groundedness on a 1–5 scale with a fixed judge prompt.

**Q35.** Why compare Bedrock Converse on an exam for a Vertex course? 
**A.** Cloud choice questions test whether you know auth, IDs, and API shapes differ even when Claude skills transfer.
---

## 21. Production readiness mini-rubric (Vertex)

1. Model enabled in the correct project. ID verified against Model Garden card
2. Endpoint class matches residency policy (global / us / eu / regional)
3. Prod uses service account or workload identity — not laptop user ADC
4. IAM roles least-privilege for prediction only where possible
5. Timeouts/retries for stream and non-stream paths
6. Eval suite gates prompt/model changes
7. Logging redacts secrets and sensitive prompts as required
8. Cost alerts on token usage. Understand premium endpoint pricing
9. Documented fallback if regional capacity fails (only if policy allows)
10. Feature dependencies checked against Vertex support matrix

---

## 22. Side-by-side request anatomy (conceptual)

**Anthropic API:** host `api.anthropic.com` + API key header + body `{model, messages, max_tokens,...}`

**Bedrock Converse:** AWS SigV4 to `bedrock-runtime` + `{modelId, messages, inferenceConfig, toolConfig,...}`

**Vertex rawPredict:** Google bearer token to `…aiplatform…/publishers/anthropic/models/{MODEL}:rawPredict` + body `{anthropic_version, messages, max_tokens,...}` (model in path)

If you can rewrite one hello-world request across all three shapes, you can answer the cloud comparison questions.

---

## 23. Team onboarding one-pager (what to paste in a wiki)

1. Enable Claude models X/Y in Model Garden for project P
2. Install `anthropic[vertex]`. Set project. Prefer `region=global` unless compliance says otherwise
3. Local: `gcloud auth application-default login`
4. Prod: service account with documented roles
5. Never commit credentials
6. Use publisher model IDs from the model card — do not copy Bedrock IDs
7. Own conversation history in our session store service
8. Tools must return results for every tool_use
9. Before you use an advanced Anthropic-only feature, check the Vertex support list
10. Run the shared eval pack before you promote prompt changes.

---

## 24. Quick reference card

| Need | Vertex move |
| --- | --- |
| Auth | ADC / service account |
| Enable | Model Garden |
| Default location | `global` if residency flexible |
| US/EU residency + HA | `us` / `eu` |
| Hard pin / provisioned | Specific region |
| SDK | `AnthropicVertex` |
| Raw version field | `vertex-2023-10-16` |
| Model ID | Publisher `claude-…` |
| History | App-managed |
| vs Bedrock | Messages not Converse. Different IDs/auth |

**One-sentence rule:** On Vertex, enable in Garden and authenticate with Google. Select the endpoint for residency, call Messages with publisher IDs, and verify feature parity before you promise capabilities.

---

## 25. Study close

Redraw these four items without notes. The list also includes (1) the three endpoint classes, (2) ADC vs API key, (3) publisher vs Bedrock ID examples, (4) the three-cloud comparison table. If you can, you are ready for Vertex-differentiating items. Then refresh shared Claude skills (tools, RAG, MCP, agents, eval). The ideas are the same as Bedrock/Anthropic tracks. Vertex transport and constraints differ.

---

*Aligned to https://academy.claude.com/courses/claude-with-google-cloud-s-vertex-ai. Confirm model IDs, regions, pricing premiums, and feature matrices on Google Cloud and Anthropic docs before shipping.*

---

## 26. Final flash drills

Say out loud without notes:

1. Enable → ADC → endpoint class → publisher model ID → Messages call.
2. Global vs us/eu vs regional in one sentence each.
3. Three reasons Vertex differs from Anthropic API (auth, version field/path, feature parity).
4. Three reasons Vertex differs from Bedrock (IAM cloud, Converse vs Messages, ID format).
5. Name parallelization, chaining, and routing with a one-line example each.

If any drill fails, re-read only that section's tables, then retry. Prefer short closed-book loops over re-reading the entire file end-to-end.
