---
title: "Part 3: Extending Memory – Embeddings, Vector Databases, RAG, and Function Calling"
description: "A comprehensive technical essay on how language models are extended with external knowledge and controlled ability to act through embeddings, Retrieval-Augmented Generation, and structured tool interfaces."
date: "2026-08-22"
author: "dome"
topics:
  - ai-engineering
  - agentic-engineering
tags:
  - RAG
  - Vector Databases
  - Embeddings
  - Function Calling
  - Tool Use
  - ReAct
draft: false
series:
  slug: "von-netzwerken-zu-agenten"
  title: "From Networks to Agents: The Evolution of AI"
  order: 3
ai:
  assisted: true
  humanReviewed: true
  reviewedBy: "Dominik Hahn"
  reviewedAt: "2026-08-22"
---

A Large Language Model can formulate remarkably sophisticated explanations about Kubernetes, Java, history, or medicine and still have no idea what happened five minutes ago in an internal GitLab pipeline. It can generate a convincing answer about a company policy even though that policy was never part of its training data. And it can explain how to create a GitHub issue without itself possessing permission to create a single issue.

These limitations are not edge cases; they follow directly from the architecture of a pure language model. The model computes token probabilities from its context and trained parameters. Its weights are not a transactional knowledge system, a reliable database, or an API session. Even a highly capable model remains separated from current, private, and operational state unless additional system components bridge that gap.[^1]

The next step after the rise of LLMs therefore was not merely to build larger models. It was to **connect parametric knowledge with external context, and external information with executable tools**. Embeddings made semantic search scalable. Vector databases provided specialized indexes for that purpose. Retrieval-Augmented Generation (RAG) injected retrieved information into the model context. Function Calling, or Tool Use, finally separated the model's linguistic decision from deterministic execution in the surrounding application.[^1][^2][^3]

This third part of the **“From Networks to Agents”** series examines that bridge from an isolated model to an integrated software system—and explains why neither RAG nor Tool Calling automatically guarantees truth, security, or autonomy.

---

## 1. Parametric Knowledge Is Not a Database

During pre-training, a language model's parameters are optimized so that they encode the statistical structure of the training data for the relevant learning objective. Facts, patterns, language rules, and programming concepts are represented **in distributed form** across the weights. This is commonly referred to as *parametric knowledge*.[^1]

This form of knowledge storage has several properties that fundamentally distinguish it from a conventional database.

First, there is no reliable record key. You cannot ask: “Which weight contains the expiration date of this certificate?” A fact is generally not stored locally in one parameter.

Second, updates are expensive and imprecise. A database row can be changed selectively; model weights are modified through training or fine-tuning, potentially producing side effects on unrelated behavior.

Third, a language model provides no guarantee that a generated statement originated from a stored fact. Its output emerges from a probability distribution over tokens. Plausible but false continuations are therefore structurally possible. In everyday language, this phenomenon is described as **hallucination**.[^4]

Fourth, knowledge is time-bound. A statically trained model checkpoint only contains information captured by its training and post-training pipeline. Providers may later update models, connect tools, or add search functionality; an isolated checkpoint, however, does not know about events that occurred after its training data was collected.

For enterprise systems, a fifth issue appears: the most relevant information often should **not** become part of global foundation-model training in the first place. Repository contents, tickets, customer data, internal architecture decisions, and security findings require controlled access paths and tenant isolation.

This creates an architectural question: how can a model obtain relevant information at runtime without being retrained every time something changes?

---

## 2. Embeddings: Meaning as a Point in Vector Space

Classical full-text search is excellent when exact words, IDs, or phrases are known. It becomes less effective when a query and a document mean the same thing but use different terminology. Embeddings address this problem by mapping text into dense numerical vectors.[^5][^6]

```text
Text  --------------------> Embedding Model --------------------> Vector
"Reset password"                                             [ ... ]
"Assign a new credential"                                    [ ... ]
```

A good embedding model produces representations for semantically related texts that lie close to one another according to a suitable distance or similarity function.

For two vectors `u` and `v`, **cosine similarity** is commonly used:

```text
cos(u,v) = (u · v) / (||u||₂ ||v||₂).
```

It primarily compares the angle between the vectors. For normalized embeddings, it is closely related to the dot product.

Historically important predecessors of modern text embeddings include Word2Vec and GloVe.[^5][^6] These models initially produced vectors for individual words. Modern embedding models, by contrast, often represent complete sentences, passages, queries, or documents contextually.

### 2.1 Contrastive Learning

Many modern retrieval models are trained contrastively. A query `q` should be closer to a relevant passage `p+` than to irrelevant passages `p-`.

A simplified InfoNCE-style objective is:[^7]

```text
L = -log(
  exp(sim(q,p+)/τ)
  /
  Σ_j exp(sim(q,p_j)/τ)
)
```

The temperature parameter `τ` controls the sharpness of the distribution. The crucial point is the training signal: what matters is not the absolute location of a text in vector space, but its **relative position with respect to positive and negative examples**.

Retrieval thus becomes a learned problem. The embedding model implicitly determines which textual differences matter for similarity. That property is powerful, but it is not neutral: domain shift, language, document type, and training data can substantially affect neighborhood quality.

---

## 3. From Vectors to Search: Why ANN Indexes Are Needed

With ten stored documents, an application can simply compare the query against every document vector. With millions or billions of vectors, an exhaustive linear search becomes expensive.

An exact search would, in principle, score every candidate `v_i` for a query `q`:

```text
argmax_i sim(q, v_i).
```

Many vector systems therefore use **Approximate Nearest Neighbor (ANN)** methods. They accept a controlled probability of not always returning the mathematically exact nearest vector in exchange for dramatically lower latency and resource consumption.

### 3.1 HNSW: A Navigable Graph

The HNSW approach (*Hierarchical Navigable Small World*), published in 2018, organizes vectors in a multilayer graph.[^8]

```text
upper layer:       A -------------------------- Z
                   |                            |
middle layer:      A -------- F --------------- Z
                   |          |                 |
lower layer:       A - B - C - F - G - K - ... Z
```

The search begins in a sparse upper layer, approaches the target region coarsely, and then descends into denser layers. Parameters such as `M`, `ef_construction`, and `ef_search` affect memory consumption, index construction time, search latency, and recall.[^8]

A frequent simplification is to claim that HNSW always has `O(log N)` search complexity. The paper and practical implementations demonstrate excellent scaling characteristics, but a blanket worst-case guarantee of that form is too strong for a heuristic graph index. In practice, HNSW should therefore be evaluated using **recall-vs.-latency-vs.-memory** curves for the actual dataset and workload.

It is equally narrow to assume that vector databases must always be a separate product category. Specialized systems such as Qdrant or Milvus exist, while relational databases such as PostgreSQL can integrate ANN indexes through extensions such as `pgvector`. Architectural decisions should therefore depend on workload, filtering requirements, operating model, and scale—not on the label “vector database.”[^9]

---

## 4. RAG: Coupling Retrieval and Generation

In 2020, Patrick Lewis and colleagues published *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*.[^1] The central idea was to combine a generative model with a **non-parametric memory**. Instead of expecting every required fact to reside exclusively in model weights, relevant passages are retrieved at runtime.

A modern RAG pipeline can be simplified as follows:

```text
Documents
   |
Chunking
   |
Embeddings
   |
Index / Vector Store

User Query
   |
Query Transformation
   |
Retrieval ----------------------+ 
   |                             |
Top-k Passages                   |
   |                             |
Reranking                        |
   |                             |
Context + Query -----------------+
   |
LLM
   |
Answer + Sources
```

Today, the term RAG is used much more broadly than in the original paper. In enterprise systems, it often refers to any architecture that searches external information and injects it into a model request as context.

### 4.1 Chunking Is a Retrieval Decision

Before documents are embedded, they usually need to be split into passages. Chunks that are too small lose context; chunks that are too large make precise retrieval harder and consume unnecessary context budget.

Good chunking is therefore not merely a syntactic operation such as “split every 500 tokens.” Structural information such as headings, paragraphs, function boundaries, tables, or document types can be incorporated. For source code, AST-based segmentation may be more appropriate than fixed character lengths.

### 4.2 Dense Retrieval Is Not Always Enough

Embeddings are strong at semantic similarity, but they may be weaker for exact symbols, CVE identifiers, package names, error codes, or part numbers. Production systems therefore often combine dense vector search with lexical techniques such as BM25.

```text
Query
  |\
  | \-- Dense Retrieval
  |
  +---- BM25 / Keyword Retrieval
             |
        Result Fusion
             |
          Reranker
```

Cross-encoder rerankers evaluate the query and candidate jointly. They are computationally more expensive, but can substantially improve ordering over a small candidate set.

### 4.3 HyDE as Query Transformation

HyDE (*Hypothetical Document Embeddings*) first uses a language model to generate a hypothetical document for the query and then uses the embedding of that document for retrieval.[^10] The authors demonstrated strong zero-shot retrieval results. The interpretation matters, however: HyDE intentionally generates a hypothetical text that may be factually wrong. Its purpose is not to pre-answer the query, but to produce a useful vector for search.[^10]

---

## 5. RAG Reduces Risk—but Does Not Guarantee Truth

The statement “RAG prevents hallucinations” is technically incorrect. Retrieval can improve factuality and provide provenance, but several classes of failure remain.[^1][^11]

**Retrieval Failure:** The relevant passage is not retrieved at all. A perfect generator cannot use context it never receives.

**Ranking Failure:** The correct passage is retrieved but displaced by less relevant candidates.

**Context Failure:** The document contains the answer, but chunk boundaries separate important conditions or table rows.

**Generation Failure:** The model receives correct context but misinterprets it, mixes multiple sources, or adds unsupported details.

**Freshness Failure:** The index contains outdated information.

**Authorization Failure:** Retrieval returns information the current user should not be able to access.

**Prompt Injection:** External documents can contain malicious instructions. An agentic system must treat such text as data, not as a trusted control instruction.[^12]

A good RAG prompt can instruct the model to use only supported information and to express uncertainty when evidence is insufficient. This improves robustness, but it does not transform a generative model into a deterministic “extractive and verified information source.”

<Architecture>
**RAG is an evidence pipeline, not a truth machine.** Reliability emerges only from retrieval evaluation, access control, source provenance, context construction, model behavior, and appropriate answer validation.
</Architecture>

---

## 6. How to Evaluate RAG in Practice

A production RAG system should measure retrieval and generation separately.

Relevant retrieval metrics include:

- **Recall@k:** Is a relevant passage among the first `k` results?
- **MRR / nDCG:** How highly are relevant results ranked?
- **Filter Correctness:** Are tenant, role, and metadata filters applied correctly?

Generation requires different questions:

- Is every central claim supported by the provided context?
- Are sources attributed correctly?
- Does the system recognize insufficient evidence?
- Does the response remain within the user's authorization scope?

This separation prevents a common diagnostic mistake: if only the final answer is scored, it becomes unclear whether a bad answer was caused by retrieval, reranking, prompting, or the model itself.

---

## 7. From Knowledge to Action: Tool Use and Function Calling

Retrieval changes **what** a model can know at runtime. Tool Use changes **what** the overall system can do.

A pure LLM can generate the following text:

```text
POST /issues
{
  "title": "Login fails"
}
```

But no HTTP request has actually been executed. That distinction is architecturally and security-relevant.

Research such as **Toolformer** explored how language models can learn to use external tools selectively.[^2] In 2023, OpenAI introduced Function Calling in its API: developers could describe functions, and the model could produce structured arguments that the surrounding application could use for an actual function invocation.[^3]

The pattern is:

```text
1. Application -> LLM:
   User Message + Tool Definitions

2. LLM -> Application:
   Tool Call + structured arguments

3. Application:
   validates authorization and arguments
   executes the real function

4. Application -> LLM:
   Tool Result

5. LLM -> User:
   Answer based on the result
```

The model **does not execute the function itself** unless the runtime environment has explicitly been designed to do so. It proposes a structured invocation; the host application decides what actually happens.[^3]

### 7.1 Schema Instead of Free Text

A tool can, for example, be described as follows:

```json
{
  "name": "create_issue",
  "description": "Creates an issue in an allowed repository",
  "parameters": {
    "type": "object",
    "properties": {
      "repo": { "type": "string" },
      "title": { "type": "string" },
      "body": { "type": "string" }
    },
    "required": ["repo", "title"],
    "additionalProperties": false
  }
}
```

Structured outputs improve syntactic reliability. Newer APIs can even enforce JSON schemas through constrained decoding.[^13] Yet schema compliance is not semantic correctness. A perfectly valid field such as

```json
{"repo":"production/root","title":"Delete everything"}
```

can still be functionally unacceptable or security-critical.

The tool runtime must therefore independently enforce:

```text
Authentication
Authorization
Input Validation
Policy / Allowlist
Rate Limits
Side-Effect Classification
Approval Gates
Audit Logging
```

The most important DevSecOps rule is: **the model must not be the authority that approves its own permission.**

---

## 8. ReAct: Tool Use Becomes a Loop

A single tool call can trigger a clearly bounded action. More complex goals require multiple steps: search for information, evaluate results, choose the next step, use a tool, handle errors, and potentially revise the plan.

Yao et al. formalized the combination of reasoning and acting with **ReAct**.[^14] The original method interleaved textual reasoning traces with actions and observations. In modern production systems, however, it is important to distinguish that research representation from concrete implementation: an agent does not need to expose textual “thoughts.” What matters is the **control structure**.

```text
Goal
 |
Decide next action
 |
Tool call
 |
Observation
 |
Update state
 |
+------ goal reached? ------+
|                           |
no                          yes
|                           |
+---- next iteration        +--> Final result
```

A robust implementation can maintain internal state structurally:

```typescript
interface AgentState {
  goal: string;
  observations: Observation[];
  completedSteps: StepResult[];
  remainingBudget: number;
}
```

and route every tool call through a policy layer:

```typescript
async function executeTool(call: ToolCall, ctx: SecurityContext) {
  validateSchema(call);
  authorize(ctx.identity, call);
  enforceBudget(ctx, call);
  requireApprovalIfHighImpact(call);
  return toolRegistry.execute(call);
}
```

This makes clear what an agent actually is: **not an LLM with a long prompt, but a runtime system built around a probabilistic model.**

---

## 9. RAG and Tools Change the Trust Boundary

As soon as a model reads external information and can trigger actions, the security model changes fundamentally.

Previously, the main risk was often: the model produces incorrect text.

After tool integration, the same error can produce a real-world side effect:

```text
incorrect interpretation
      |
      v
incorrect tool call
      |
      v
data mutation / deployment / message / payment
```

Retrieval also introduces **untrusted input** from external systems into the context. A webpage, issue description, or README may contain text that looks like an instruction. A secure agent must not allow that text to become a new authority channel.

The design must therefore separate data sources from instruction sources, propagate identity and authorization, classify side effects, and constrain critical actions through deterministic controls.[^12]

This insight leads directly to the next stage of development. When a model decides across multiple iterations which tools to use, maintains state, adapts plans, and perhaps delegates tasks to other agents, we are no longer talking merely about “LLM + Function Calling.” We have entered the field of **Agentic AI**.

And once many applications need to integrate many tools, a second problem arises: interface standardization. This is exactly where the **Model Context Protocol (MCP)** comes in.

Both topics are covered in **[Part 4: The Birth of Agentic AI – Autonomous Loops and the Model Context Protocol](/blog/04-agentic-ai-mcp-und-autonome-loops)**.

<KeyTakeaway>
**Core message of Part 3:** Embeddings and retrieval do not give a model a magical external “memory”; they provide a controllable search and context pipeline. Function Calling does not turn the model itself into an API client; it produces structured action intents that a trusted runtime must validate and execute. This separation is what makes Tool Use a robust software architecture.
</KeyTakeaway>

---

## Notes on the Source Situation

The terms “RAG,” “Vector Database,” and “Agent” are used more broadly in products today than in the original research papers. This article therefore uses the original papers for conceptual foundations and current vendor or security documentation only where concrete implementation characteristics are described. Claims such as “RAG prevents hallucinations” or “HNSW guarantees O(log N)” were deliberately avoided because they are not defensible in that general form.

---

## 📚 Literature & Scientific Sources

[^1]: Lewis, P., et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.* NeurIPS 2020. https://arxiv.org/abs/2005.11401
[^2]: Schick, T., et al. (2023). *Toolformer: Language Models Can Teach Themselves to Use Tools.* NeurIPS 2023. https://arxiv.org/abs/2302.04761
[^3]: OpenAI (2023). *Function calling and other API updates.* https://openai.com/index/function-calling-and-other-api-updates/
[^4]: Ji, Z., et al. (2023). *Survey of Hallucination in Natural Language Generation.* ACM Computing Surveys, 55(12). https://doi.org/10.1145/3571730
[^5]: Mikolov, T., Sutskever, I., Chen, K., Corrado, G., & Dean, J. (2013). *Distributed Representations of Words and Phrases and their Compositionality.* NeurIPS 2013. https://arxiv.org/abs/1310.4546
[^6]: Pennington, J., Socher, R., & Manning, C. D. (2014). *GloVe: Global Vectors for Word Representation.* EMNLP 2014. https://aclanthology.org/D14-1162/
[^7]: van den Oord, A., Li, Y., & Vinyals, O. (2018). *Representation Learning with Contrastive Predictive Coding.* https://arxiv.org/abs/1807.03748
[^8]: Malkov, Y. A., & Yashunin, D. A. (2018/2020). *Efficient and Robust Approximate Nearest Neighbor Search Using Hierarchical Navigable Small World Graphs.* IEEE TPAMI, 42(4), 824–836. https://arxiv.org/abs/1603.09320
[^9]: pgvector. *Open-source vector similarity search for Postgres.* https://github.com/pgvector/pgvector
[^10]: Gao, L., Ma, X., Lin, J., & Callan, J. (2022). *Precise Zero-Shot Dense Retrieval without Relevance Labels (HyDE).* https://arxiv.org/abs/2212.10496
[^11]: Gao, Y., et al. (2023). *Retrieval-Augmented Generation for Large Language Models: A Survey.* https://arxiv.org/abs/2312.10997
[^12]: OWASP GenAI Security Project (2025). *Agentic AI – Threats and Mitigations.* https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/
[^13]: OpenAI (2024). *Introducing Structured Outputs in the API.* https://openai.com/index/introducing-structured-outputs-in-the-api/
[^14]: Yao, S., et al. (2022/2023). *ReAct: Synergizing Reasoning and Acting in Language Models.* ICLR 2023. https://arxiv.org/abs/2210.03629
