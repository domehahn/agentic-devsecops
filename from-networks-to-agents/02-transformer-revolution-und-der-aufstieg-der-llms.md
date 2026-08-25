---
title: "Part 2: Attention Is All You Need – Transformers, OpenAI, Google, and the Era of LLMs"
description: "An in-depth technical essay on the Transformer revolution: from Seq2Seq and self-attention through BERT and GPT to scaling laws, in-context learning, and modern alignment methods."
date: "2026-08-22"
author: "dome"
topics:
  - ai-engineering
  - agentic-engineering
tags:
  - Transformers
  - LLM
  - Self-Attention
  - OpenAI
  - Google
  - Scaling Laws
draft: false
series:
  slug: "von-netzwerken-zu-agenten"
  title: "From Networks to Agents: The Evolution of AI"
  order: 2
ai:
  assisted: true
  humanReviewed: true
  reviewedBy: "Dominik Hahn"
  reviewedAt: "2026-08-22"
---

The Transformer is often described as the architecture that “eliminated recurrence” in 2017 and thereby directly paved the way to ChatGPT. That shorthand captures an important point, but it hides the deeper scientific shift. The Transformer was not merely a faster RNN. It changed the elementary computation through which sequence models represent relationships between pieces of information.[^1]

A recurrent network builds its state step by step along a sequence. The `t`-th word computationally depends on the state of the previous step. A Transformer, by contrast, can compute relationships among many positions in a sequence directly within a layer. This principle—**self-attention**—shortened the path length between distant tokens, made training easier to parallelize, and proved remarkably scalable.[^1]

That alone does not yet produce large language models in the modern sense. Only the combination of Transformer architectures, self-supervised pre-training, large datasets, specialized hardware, distributed training, scaling laws, and later post-training transformed language models from specialized NLP components into general-purpose systems capable of performing tasks from natural-language instructions and contextual examples.[^2][^3][^4]

This second part of **“From Networks to Agents”** reconstructs that development. It explains the mathematics of attention, places BERT and GPT in their architectural context, separates documented facts from speculation about proprietary models, and examines why scaling produced extraordinary capabilities without automatically guaranteeing reliability or alignment.

---

## 1. Before the Transformer: Seq2Seq and the Bottleneck of Recurrent Encoders

In 2014, Ilya Sutskever, Oriol Vinyals, and Quoc V. Le showed that deep LSTMs could translate sequences directly into other sequences.[^5] An encoder reads the input sequence and creates an internal state; a decoder generates the output sequence from it.

```text
x₁ -> x₂ -> ... -> x_T
        Encoder RNN
             |
             v
       Context / State
             |
             v
        Decoder RNN
             |
       y₁ -> y₂ -> ...
```

In early Seq2Seq systems, the encoder had to compress the relevant content of an entire input sequence into a limited representation. This information bottleneck became increasingly problematic for long sequences. Dzmitry Bahdanau, Kyunghyun Cho, and Yoshua Bengio therefore proposed an attention mechanism in which the decoder could weight different encoder states at every output step.[^6]

Instead of relying only on a final vector, the system conceptually computes:

```text
context_t = Σ_i α_(t,i) · h_i
```

where `h_i` are encoder states and `α_(t,i)` are the attention weights calculated for the current step. The translation of a target word can thereby access specifically those parts of the input sentence that are most relevant at that moment.

The attention idea therefore predated the Transformer. The remaining bottleneck lay in the encoder and decoder themselves: RNNs and LSTMs process temporal states in a dependency chain. Matrix operations inside one step and multiple sequences can be parallelized, but state `h_t` fundamentally depends on information from `h_(t-1)`. The longer the sequence, the longer the sequential dependency path.

The Transformer therefore reframed the question: **What if relationships within the sequence themselves became the primary computation and recurrence was no longer needed at all?**

---

## 2. 2017: What “Attention Is All You Need” Actually Changed

Ashish Vaswani and seven other Google researchers published *Attention Is All You Need* in 2017.[^1] The original Transformer was an **encoder-decoder architecture** for machine translation. It did not consist solely of one attention formula. In addition to multi-head attention, it contained position-wise feed-forward networks, residual connections, layer normalization, embeddings, and positional information.[^1]

The central innovation was that relationships within a sequence could be modeled without recurrent or convolutional layers.

```text
Input
  |
Embedding + Position
  |
+---------------------------+
| Multi-Head Self-Attention |
+---------------------------+
  |
Residual + Norm
  |
Feed-Forward Network
  |
Residual + Norm
  |
  ...
```

This had major consequences for training. All tokens in a sequence could be processed largely in parallel within a layer. In addition, information between two distant positions no longer had to traverse dozens or hundreds of recurrent states. Self-attention creates a direct interaction path between positions inside a layer.[^1]

That does not mean Transformers scale to arbitrarily long sequences for free. Standard self-attention creates an `N × N` matrix for `N` tokens and therefore has a quadratic memory and compute component with respect to sequence length. The architecture traded one bottleneck for another: **global interaction became parallelizable, but expensive at long context lengths**.

---

## 3. Self-Attention Mathematically: Query, Key, and Value

Consider a sequence of token representations as a matrix

```text
X ∈ R^(N × d_model).
```

Three linear projections are computed for every token:

```text
Q = XW_Q
K = XW_K
V = XW_V.
```

The terms *Query*, *Key*, and *Value* deliberately evoke information retrieval, but they should not be interpreted too literally. They are learned vector representations whose meaning emerges during training.

The **Scaled Dot-Product Attention** defined in the original paper is:[^1]

```text
Attention(Q,K,V) = softmax(QKᵀ / √d_k) V.
```

The operation can be decomposed into four steps.

### 3.1 Similarity Through Dot Products

`QKᵀ` produces a matrix of pairwise scores:

```text
S_ij = q_i · k_j.
```

For every query token `i`, the model evaluates how strongly it should respond to the key representation of token `j`. At this stage, the score is not a semantic probability. It is a learned dot product.

### 3.2 Scaling

Dividing by `√d_k` prevents the variance of the dot products from growing strongly with dimensionality. Without this scaling, very large values could push the softmax function into heavily saturated regions and make optimization harder.[^1]

### 3.3 Softmax

```text
A = softmax(S / √d_k)
```

normalizes scores row-wise. The resulting weights are positive and sum to `1` for each query.

### 3.4 Weighted Aggregation

```text
O = AV
```

combines the value vectors according to the calculated weights. The representation of a token therefore becomes context-dependent on other positions in the sequence.

<Architecture>
**Self-attention does not store fixed grammar rules such as “subject looks for verb.”** Functional patterns of that kind can emerge in individual heads or combinations of layers, but their internal roles are learned during training rather than programmed in advance.
</Architecture>

---

## 4. Multi-Head Attention: Multiple Projection Spaces Instead of One Relationship

The Transformer does not compute attention only once. **Multi-Head Attention** uses several separate projections:

```text
head_i = Attention(QW_i^Q, KW_i^K, VW_i^V)

MultiHead(Q,K,V) = Concat(head_1, ..., head_h) W^O.
```

Each head operates in its own learned subspace. This allows the model to represent different types of dependency at the same time. Interpretability studies have shown that some attention heads correlate with syntactic or positional patterns; this does not mean every head can reliably be assigned one stable, clearly named linguistic function.[^7]

That distinction matters. A teaching example such as “head 1 detects subject-verb relations, head 2 resolves pronouns” can illustrate the idea, but would be too strong as a factual claim about real models. Transformer representations are distributed: relevant information can be encoded across many heads and layers.

---

## 5. Order Without Recurrence: Positional Information and RoPE

Self-attention has no natural notion of token order without additional information. If the input rows were permuted and no positional information were added, the pure attention operation would remain correspondingly symmetric to such permutations.

The original paper therefore added sinusoidal positional vectors to token embeddings:[^1]

```text
PE(pos, 2i)   = sin(pos / 10000^(2i/d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model)).
```

Later model families developed other approaches. **Rotary Position Embeddings (RoPE)** rotate query and key components as a function of position and thereby encode relative position relationships into attention scores.[^8] RoPE was used, among other models, in LLaMA.[^9]

Precision matters here as well: RoPE does not automatically guarantee arbitrarily good extrapolation to context lengths far beyond training. Long-context behavior depends on training, frequency scaling, architecture, data, and additional methods. The common claim that RoPE “solves” the context-length problem is therefore too broad.

---

## 6. Three Transformer Paths: Encoder, Decoder, and Encoder-Decoder

After 2017, the field did not split into a simple two-camp story of “Google = encoder” and “OpenAI = decoder.” Three major architecture paths emerged.

### 6.1 Encoder-Only: BERT

Google released **BERT** (*Bidirectional Encoder Representations from Transformers*) in 2018.[^10] BERT uses Transformer encoders and can incorporate context on both sides of a token. Its original pre-training combined Masked Language Modeling with Next Sentence Prediction.

In masked language modeling, selected tokens are hidden and reconstructed from surrounding context. This makes BERT highly effective for contextual representations, classification, extraction, and other tasks where the complete input is available at once.

The statement that BERT “cannot generate text by design” would nevertheless be too absolute. A bidirectional encoder is not constructed as an autoregressive generator, but generative methods based on masked models are possible. The decisive point is that BERT's original architecture and pre-training objective were not optimized for left-to-right free text generation.

### 6.2 Decoder-Only: GPT

OpenAI's GPT line used autoregressive Transformers that predict the next token from previous tokens.[^11][^12][^2]

The probability decomposition is:

```text
P(x₁,...,x_N) = Π_i P(x_i | x_<i).
```

With **causal masking**, position `i` cannot attend to future positions. The training objective looks surprisingly unspecific: predict the next token. But with sufficiently large and diverse data, that task forces the model to represent statistical regularities in syntax, semantics, facts, code, and many forms of textual structure.

GPT-1 showed in 2018 that generative pre-training could be effectively combined with subsequent fine-tuning on NLP tasks.[^11] GPT-2 demonstrated stronger zero-shot properties in 2019.[^12] GPT-3 scaled to 175 billion parameters in 2020 and showed dramatically that tasks could be specified by examples directly in the prompt—**in-context learning**.[^2]

### 6.3 Encoder-Decoder Remains Relevant

The original encoder-decoder family also remained important. Models such as T5 framed different NLP tasks as text-to-text problems while continuing to use both sides of the Transformer architecture.[^13] The later dominance of decoder-only chat and foundation models therefore does not mean encoder-decoder models made a technological “wrong turn.”

---

## 7. GPT-4 and the Boundary of Public Information

Especially with proprietary models, documented facts must be separated cleanly from industry rumor. In the 2023 GPT-4 Technical Report, OpenAI stated that GPT-4 is a **large multimodal model** that can process text and image inputs and produce text outputs.[^14] OpenAI explicitly did not disclose detailed information about model size, hardware, training compute, dataset construction, or architecture.

It follows that claims such as GPT-4 having “more than one trillion parameters” or being definitively a particular **Mixture-of-Experts (MoE) model** are not supported by the official publication. Such numbers have circulated in media and community discussions, but they are not a verified technical specification. A source-based technical article should therefore not present them as fact.

This example illustrates a fundamental problem in modern AI reporting: the more economically important a model becomes, the fewer complete technical details some providers publish. The correct statement is then not “we know the architecture is X,” but: **the public primary source provides no reliable statement on this point.**

---

## 8. KV Cache and FlashAttention: Why Inference Is a Systems Problem

The Transformer removed one training bottleneck but created new systems challenges. During autoregressive generation, output appears token by token. Every new token requires the keys and values of the preceding sequence.

A **KV cache** stores these already computed representations so they do not need to be recomputed from scratch at every generation step. This reduces redundant computation while memory consumption grows with context length, number of layers, batch size, and KV dimensions.

**FlashAttention** addresses a different problem. Dao et al. showed in 2022 that practical attention performance is heavily constrained by data movement between different levels of GPU memory.[^15] FlashAttention computes exact attention in blocks and in an IO-aware manner, avoiding unnecessary materialization of the full large attention matrix in slower memory.

These optimizations demonstrate why the history of LLMs cannot be understood from model equations alone. The practically usable model size also depends on memory hierarchies, parallelization strategies, network bandwidth, kernels, and serving architectures.

---

## 9. Scaling Laws: “Bigger” Became a Scientific Strategy

One of the most consequential observations of the LLM era was that model quality could improve relatively regularly over wide ranges as parameters, data, and compute increased. Kaplan et al. described empirical **scaling laws** for autoregressive language models in 2020.[^3]

In simplified form, a loss term can be represented as a power law in a resource such as parameter count:

```text
L(N) ≈ A · N^(-α) + L_∞
```

and analogously for dataset size or compute. The exact exponents depend on the experimental setup; the important observation was relatively smooth scaling.

In 2022, Hoffmann et al. showed with **Chinchilla** that many large models had been oversized and trained on too few tokens for a fixed compute budget.[^4] Their result shifted attention toward **compute-optimal** relationships between model size and training data. The frequently cited heuristic of roughly 20 training tokens per parameter comes from this experimental regime and should not be treated as a universal law of nature for every architecture or objective.[^4]

The strategic consequence was enormous nonetheless: training became more predictable. If loss can be extrapolated relatively reliably across orders of magnitude, scaling becomes not just an engineering project but a bet on measurable returns from additional compute.

---

## 10. Emergence: Sudden Capabilities or a Measurement Effect?

As model size increased, researchers reported so-called **emergent abilities**: tasks on which smaller models appeared to perform near zero and larger models suddenly performed much better.[^16]

This observation encouraged a strong interpretation: certain capabilities would qualitatively “appear” once a critical scaling threshold was crossed. The research picture is more nuanced. Schaeffer, Miranda, and Koyejo showed in 2023 that some abrupt jumps can result from discontinuous evaluation metrics. With more continuous metrics, some improvements look substantially smoother.[^17]

This does not disprove emergence. Rather, the strong claim that scaling necessarily produces unpredictable qualitative phase transitions is not generally established. For engineering decisions, the distinction is critical: **a benchmark jump is not automatically evidence of a new internal cognitive mechanism.**

---

## 11. From Base Model to Assistant: Post-Training and Alignment

An autoregressive base model initially optimizes next-token prediction. It therefore has no inherent objective to be helpful, truthful, or safe in response to user requests. The modern assistant form emerges through **post-training**.

### 11.1 Supervised Fine-Tuning

Supervised Fine-Tuning (SFT) uses examples of desired interactions:

```text
Instruction -> desired answer
```

The model learns the form of instruction following. SFT alone, however, can only imitate behavior represented in the demonstration data.

### 11.2 RLHF

InstructGPT demonstrated in 2022 a pipeline combining SFT, human preference comparisons, a reward model, and reinforcement learning.[^18]

Simplified:

```text
Prompts
  |
multiple model responses
  |
Human Ranking
  |
Reward Model R(x,y)
  |
Policy Optimization
```

Human preferences do not define an objective truth value. They create a training signal about which responses are preferred under the chosen guidelines and evaluator judgments.

### 11.3 Direct Preference Optimization

**Direct Preference Optimization (DPO)** introduced in 2023 a simpler way to use preference data directly to optimize a language model without separately training a reward model and then running a classic RL loop.[^19]

DPO uses preferred and rejected responses and optimizes their relative probabilities against a reference model. The method is conceptually elegant and widely used in practice, but it does not automatically replace every other post-training technique. Modern systems combine, depending on the goal, SFT, preference optimization, reinforcement learning, synthetic data, verifiers, and other methods.

<Architecture>
**Alignment is not a single algorithm.** It is a layer of system design in which training data, preferences, rules, verifiers, safety evaluations, and runtime controls jointly shape observable behavior.
</Architecture>

---

## 12. The Real Break: Language Models Became Programmable Generalists

The Transformer did not suddenly make machines “understand” language. Its historical impact is more precisely described like this: it provided a computational architecture that scaled extraordinarily well with data and compute. Autoregressive pre-training transformed that scalability into general-purpose text models. In-context learning turned natural language and examples into a form of temporary task specification. Post-training then transformed base models into interactive assistants.[^1][^2][^18]

A fundamental limit still remained. A language model can talk about a database without being able to query it. It can plausibly state a current stock price without actually knowing it. It can produce a GitHub issue description without creating the issue. Its parameters contain statistically compressed knowledge, but they are neither a reliable live data source nor an authorized interface to external systems.

The next development step was therefore not primarily to squeeze even more knowledge into weights. Instead, models received **external knowledge and tools**: embeddings, retrieval, vector databases, RAG, and structured tool calls.

That is the subject of **[Part 3: Extending Memory – Embeddings, Vector Databases, RAG, and Function Calling](/blog/03-von-statischem-wissen-zu-rag-und-tool-use)**.

<KeyTakeaway>
**Core message of Part 2:** The Transformer combined global, parallelizable sequence interaction with an architecture that scaled remarkably far. LLMs, however, emerged only from the interaction of Transformers, data, compute, self-supervised pre-training, and post-training. Proprietary details must be strictly separated from publicly documented facts.
</KeyTakeaway>

---

## Notes on the Sources

OpenAI did not publish GPT-4's parameter count or complete architecture. This article therefore deliberately makes no claim that GPT-4 has a specific trillion-parameter size or a confirmed MoE structure.[^14] Likewise, “emergence” is not presented as a conclusively settled scaling phenomenon because conflicting empirical interpretations exist.[^16][^17]

---

## 📚 Literature & Scientific Sources

[^1]: Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, Ł., & Polosukhin, I. (2017). *Attention Is All You Need.* NeurIPS 2017. https://arxiv.org/abs/1706.03762
[^2]: Brown, T. B., et al. (2020). *Language Models are Few-Shot Learners.* NeurIPS 2020. https://arxiv.org/abs/2005.14165
[^3]: Kaplan, J., et al. (2020). *Scaling Laws for Neural Language Models.* https://arxiv.org/abs/2001.08361
[^4]: Hoffmann, J., et al. (2022). *Training Compute-Optimal Large Language Models.* https://arxiv.org/abs/2203.15556
[^5]: Sutskever, I., Vinyals, O., & Le, Q. V. (2014). *Sequence to Sequence Learning with Neural Networks.* NeurIPS 2014. https://arxiv.org/abs/1409.3215
[^6]: Bahdanau, D., Cho, K., & Bengio, Y. (2014). *Neural Machine Translation by Jointly Learning to Align and Translate.* https://arxiv.org/abs/1409.0473
[^7]: Clark, K., Khandelwal, U., Levy, O., & Manning, C. D. (2019). *What Does BERT Look at? An Analysis of BERT's Attention.* https://arxiv.org/abs/1906.04341
[^8]: Su, J., Lu, Y., Pan, S., Murtadha, A., Wen, B., & Liu, Y. (2021). *RoFormer: Enhanced Transformer with Rotary Position Embedding.* https://arxiv.org/abs/2104.09864
[^9]: Touvron, H., et al. (2023). *LLaMA: Open and Efficient Foundation Language Models.* https://arxiv.org/abs/2302.13971
[^10]: Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2018). *BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding.* https://arxiv.org/abs/1810.04805
[^11]: Radford, A., Narasimhan, K., Salimans, T., & Sutskever, I. (2018). *Improving Language Understanding by Generative Pre-Training.* OpenAI. https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf
[^12]: Radford, A., et al. (2019). *Language Models are Unsupervised Multitask Learners.* OpenAI. https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf
[^13]: Raffel, C., et al. (2020). *Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer.* JMLR 21(140). https://jmlr.org/papers/v21/20-074.html
[^14]: OpenAI (2023). *GPT-4 Technical Report.* https://arxiv.org/abs/2303.08774 ; OpenAI, *GPT-4 research*. https://openai.com/index/gpt-4-research/
[^15]: Dao, T., Fu, D. Y., Ermon, S., Rudra, A., & Ré, C. (2022). *FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness.* NeurIPS 2022. https://arxiv.org/abs/2205.14135
[^16]: Wei, J., et al. (2022). *Emergent Abilities of Large Language Models.* Transactions on Machine Learning Research. https://arxiv.org/abs/2206.07682
[^17]: Schaeffer, R., Miranda, B., & Koyejo, S. (2023). *Are Emergent Abilities of Large Language Models a Mirage?* NeurIPS 2023. https://arxiv.org/abs/2304.15004
[^18]: Ouyang, L., et al. (2022). *Training language models to follow instructions with human feedback.* NeurIPS 2022. https://arxiv.org/abs/2203.02155
[^19]: Rafailov, R., Sharma, A., Mitchell, E., Ermon, S., Manning, C. D., & Finn, C. (2023). *Direct Preference Optimization: Your Language Model is Secretly a Reward Model.* NeurIPS 2023. https://arxiv.org/abs/2305.18290
