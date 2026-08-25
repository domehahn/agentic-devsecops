---
title: "Part 1: From Biology to Mathematics – Perceptrons, Backpropagation, and Deep Learning"
description: "A comprehensive technical essay on the historical and mathematical foundations of neural networks: from McCulloch-Pitts and the perceptron through linear separability and backpropagation to CNNs, RNNs, and LSTMs."
date: "2026-08-22"
author: "dome"
topics:
  - ai-engineering
  - agentic-engineering
tags:
  - Neural Networks
  - Deep Learning
  - Backpropagation
  - History of AI
  - Mathematics
draft: false
series:
  slug: "von-netzwerken-zu-agenten"
  title: "From Networks to Agents: The Evolution of AI"
  order: 1
ai:
  assisted: true
  humanReviewed: true
  reviewedBy: "Dominik Hahn"
  reviewedAt: "2026-08-22"
---

The history of modern artificial intelligence is often told as a sequence of spectacular breakthroughs: the artificial neuron in 1943, the perceptron in 1958, backpropagation in 1986, AlexNet in 2012, the Transformer in 2017. That timeline is not wrong, but it is too smooth. What we now call deep learning actually emerged from several research traditions that were only loosely connected for long periods of time: mathematical logic, neurophysiology, cybernetics, statistics, optimization, signal processing, and eventually massively parallel computer architectures.[^1][^2]

The decisive change was not so much that computers eventually “learned to think.” It was that researchers gradually found a mathematical answer to a much more precise question: **How can a system change its internal parameters so that useful behavior emerges from examples?** This led to the problem of *credit assignment*: which internal connection, which weight, or which intermediate step was responsible for a prediction being correct or incorrect? That exact problem connects the perceptron of the 1950s with backpropagation, convolutional neural networks, LSTMs, and ultimately the billions of parameters in modern language models.[^2]

This first part of **“From Networks to Agents”** therefore does more than trace a historical chronology. It reconstructs the mathematical ideas from which learning neural networks emerged, explains the limits of early models, and shows why the combination of differentiable functions, gradient methods, specialized architectures, and computing power eventually opened the path to deep learning.

---

## 1. Before Artificial Intelligence: Logic, Neurophysiology, and Cybernetics

When Warren McCulloch and Walter Pitts published *A Logical Calculus of the Ideas Immanent in Nervous Activity* in 1943, “Artificial Intelligence” did not yet exist as an established research term. Their starting point was instead one of the major questions of the time: could neural activity be described formally? McCulloch and Pitts abstracted biological nerve cells into binary switching elements and showed that networks of such elements could realize logical expressions.[^3]

The model was radically simplified. An artificial neuron received binary inputs, summed their activity, and compared that sum with a threshold:

```text
Inputs                  Threshold
x₁ ------------------\
x₂ -------------------> [ Σ xᵢ >= θ ] ----> y ∈ {0,1}
x₃ ------------------/
```

In simplified form:

```text
y = 1, if Σ xᵢ >= θ
    0, otherwise
```

The scientific value of this model was not that it realistically reproduced the brain. The McCulloch-Pitts neuron ignored almost everything that makes real neurons biologically complex: temporal dynamics, biochemical processes, plastic synapses, different neurotransmitters, and stochastic effects. Its value lay in the **abstraction**. Neural activity could be studied as formal computation; networks of switching elements could be analyzed with the tools of logic.[^3]

This way of thinking fit naturally into the emerging field of cybernetics. Norbert Wiener's 1948 book *Cybernetics* investigated communication and control in machines and organisms and placed feedback loops, regulation, and information processing at the center.[^4] Later AI research would partly distance itself from cybernetics, but many fundamental ideas behind modern learning systems—feedback, state, adaptation, and control—stand in exactly this tradition.

In 1955, John McCarthy, Marvin Minsky, Nathaniel Rochester, and Claude Shannon drafted the proposal for the Dartmouth Summer Research Project on Artificial Intelligence. In it, the term **Artificial Intelligence** appeared in the programmatic sense that still shapes the field: aspects of learning and intelligence should be described precisely enough that machines could simulate them.[^5] The 1956 Dartmouth workshop was therefore less the sudden “birth” of intelligent computation than an institutional crystallization point for ongoing work in logic, cybernetics, information theory, and automata theory.

<Architecture>
**The central difference between McCulloch-Pitts and later neural networks:** The original model explains *how an artificial neuron computes*. It does not yet explain *how a network learns its parameters from data*. That second question is what made the perceptron the next decisive step.
</Architecture>

---

## 2. Rosenblatt's Perceptron: Learning Becomes a Parameter Update

Frank Rosenblatt published his perceptron model in *Psychological Review* in 1958 as a probabilistic theory of information storage and pattern recognition.[^6] Unlike the statically wired McCulloch-Pitts model, connections now carried adaptable weights. That fundamentally shifted the perspective: a system no longer had to be programmed completely by its developers; it could derive a decision boundary from training examples.

For an input vector

```text
x = (x₁, x₂, ..., xₙ)
```

and weights

```text
w = (w₁, w₂, ..., wₙ)
```

a linear perceptron first computes

```text
z = wᵀx + b
```

and then applies a threshold decision:

```text
y = 1, if z >= 0
    0, otherwise.
```

Geometrically, the condition

```text
wᵀx + b = 0
```

is a line in two input dimensions, a plane in three dimensions, and generally a **hyperplane**. The perceptron therefore does not learn arbitrary structure; it learns a linear separating surface between classes.

### Learning as Error Correction

The classic perceptron rule changes the weights only when a training example is misclassified. A common notation is:

```text
w(t+1) = w(t) + η · (y_target - y_pred) · x
b(t+1) = b(t) + η · (y_target - y_pred)
```

with learning rate `η > 0`.

This formula already contains a pattern that remains visible in modern deep-learning optimizers: an error is measured, a direction for parameter change is derived from it, and the parameters are updated iteratively. The perceptron is not yet a gradient method in the modern sense, but it establishes the core idea of **data-driven parameter learning**.[^6]

One historical detail matters here. Popular accounts often attribute the so-called perceptron convergence theorem wholesale to Rosenblatt. Rosenblatt developed the learning rule and analyzed its properties; a particularly well-known formal convergence analysis for linearly separable patterns was published by Albert Novikoff in 1962.[^7] The essence of the result is this: if a linear separating surface with a positive margin from the training points actually exists, the perceptron rule will find a separating solution after finitely many classification mistakes. For non-linearly separable data, that guarantee does not exist.

That limitation is crucial because it leads directly to the best-known symbol of the limits of early neural networks: XOR.

---

## 3. XOR and the Limit of a Single Linear Classifier

The XOR function (*exclusive OR*) outputs `1` exactly when two binary inputs differ:

| x₁ | x₂ | XOR |
|---:|---:|---:|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

In two-dimensional space, the two positive examples lie diagonally opposite one another. No single straight line can separate the positive points from the negative points.

```text
x₂
^      1 •        ○
|
|      0 ○        •
+--------------------> x₁
       0        1

• = Class 1
○ = Class 0
```

The same fact can be shown algebraically. For a perceptron with `w₁`, `w₂`, and `b`, the following conditions would all have to hold:

```text
(0,0) -> 0 : b < 0
(1,0) -> 1 : w₁ + b >= 0
(0,1) -> 1 : w₂ + b >= 0
(1,1) -> 0 : w₁ + w₂ + b < 0
```

The middle inequalities imply `w₁ + w₂ + 2b >= 0`. Together with `b < 0`, this contradicts the final condition. XOR therefore cannot be represented by a **single-layer linear perceptron**.

Marvin Minsky and Seymour Papert systematically analyzed the limits of certain perceptron classes in their 1969 book *Perceptrons*.[^8] In retrospect, this was often turned into a dramatic story: Minsky and Papert supposedly “discovered” XOR, disproved neural networks, and directly triggered the first AI winter. That story is too simple.

First, it was known that multi-layer structures could compute functions that a single perceptron could not represent. The practical problem was how such networks could be **trained efficiently**. Second, declining expectations and funding in AI research resulted from several developments: disappointing results in machine translation, limited computing power, exaggerated promises, and critical evaluations of large research programs also played important roles.[^9] *Perceptrons* contributed significantly to skepticism about neural approaches of the time, but it was not the single cause of an entire “AI winter.”

This historical precision matters. The crisis was not that neural networks had been mathematically “disproved.” It was that **representational power, learning algorithms, and available computational resources did not yet fit together**.

---

## 4. Multiple Layers: Why Nonlinearity Changes Expressive Power

A single perceptron defines a linear decision boundary. Add a hidden layer and nonlinear activation functions, however, and the network can transform the input space step by step.

```text
x₁ ---\
       > [Hidden Layer] ---> [Output]
x₂ ---/
```

Formally, a layer can be written as

```text
a^(l) = φ(W^(l) a^(l-1) + b^(l))
```

Without a nonlinear function `φ`, a cascade of multiple linear layers would still reduce to a single linear mapping, because a composition of linear mappings is linear. Nonlinearity is therefore not a cosmetic addition; it is the mathematical reason deep networks can model complex decision surfaces.

For XOR, for example, two hidden units can detect different subregions of the input space, after which the output unit combines those intermediate representations. The problem is not solved because one straight line suddenly becomes curved. Instead, the hidden layer **creates a new representation** in which the desired separation becomes possible.

Later work on universal approximation showed that feed-forward networks with sufficiently many hidden units can, under certain assumptions, approximate arbitrary continuous functions on compact sets as closely as desired.[^10] That result is frequently overinterpreted. It does not say that a particular network can learn a function efficiently, that a small amount of training data is sufficient, or that optimization is easy. **Representability is not the same as learnability.**

The actual credit-assignment problem therefore remains: when a deep network makes an error at the output, how can we determine which weight in an early layer contributed how much to that error?

---

## 5. Backpropagation: The Chain Rule, Not Magic

The common shorthand “backpropagation was invented in 1986” is historically inaccurate. Its mathematical foundations go back much further. Reverse-mode automatic differentiation had been formally described around 1970; Paul Werbos discussed gradient-based methods for multistage systems and neural networks in the 1970s. The 1986 work by David Rumelhart, Geoffrey Hinton, and Ronald Williams was nevertheless decisive in making backpropagation widely known as a practical training method for multi-layer neural networks.[^11][^12][^13]

Mathematically, backpropagation is an efficient application of the **chain rule**.

Consider a network with two layers:

```text
z¹ = W¹x + b¹
 a¹ = φ(z¹)

z² = W²a¹ + b²
ŷ  = ψ(z²)
```

and a loss function

```text
L(ŷ, y).
```

For a weight `w` in the first layer, we need

```text
∂L / ∂w.
```

Because `L` does not depend on `w` directly, but through several intermediate quantities, we obtain, for example:

```text
∂L/∂w
 = ∂L/∂ŷ
 · ∂ŷ/∂z²
 · ∂z²/∂a¹
 · ∂a¹/∂z¹
 · ∂z¹/∂w.
```

Backpropagation does not recompute these local derivatives independently from scratch for every weight. Instead, the forward pass is recorded and the error is then propagated backward layer by layer. Intermediate terms that have already been computed are reused. This form of dynamic programming is what makes the method efficient.

### Error Terms and Gradients

For a layer `l`, define an error term

```text
δ^(l) = ∂L / ∂z^(l)
```

For the final layer, it follows directly from the loss and activation functions. For earlier layers, schematically:

```text
δ^(l) = (W^(l+1))ᵀ δ^(l+1) ⊙ φ'(z^(l))
```

where `⊙` denotes element-wise multiplication.

The weight gradients then follow from

```text
∂L/∂W^(l) = δ^(l) (a^(l-1))ᵀ.
```

With gradient descent, parameters are updated as:

```text
W <- W - η ∇_W L
b <- b - η ∇_b L.
```

The significance of these equations extends far beyond classic multi-layer perceptrons. In modern frameworks such as PyTorch or JAX, automatic-differentiation systems compute exactly these kinds of derivative graphs for highly complex models with billions of parameters. Backpropagation is therefore not a historical niche algorithm, but a mechanism that remains central to supervised and self-supervised deep learning.

<Architecture>
**Backpropagation answers the credit-assignment question:** The error at the output is decomposed into local contributions through the chain rule. This makes it possible to determine, for every differentiable parameter, which direction of change should reduce the loss.
</Architecture>

---

## 6. Activation Functions: From Sigmoid to ReLU and GELU

Early multi-layer networks often used sigmoid or `tanh` functions:

```text
σ(z) = 1 / (1 + e^(-z))

tanh(z) = (e^z - e^(-z)) / (e^z + e^(-z)).
```

They are smooth and therefore easy to differentiate. But both functions saturate for large positive or negative inputs. Their derivatives then become very small. In deep networks, many such derivatives are multiplied together during backpropagation, causing gradients to shrink exponentially. This **vanishing-gradient problem** makes it difficult to train early layers and was especially important in recurrent networks.[^14]

With the rediscovery and broad adoption of the Rectified Linear Unit (ReLU),

```text
ReLU(z) = max(0, z)
```

training many deep feed-forward and convolutional networks became substantially more practical. Nair and Hinton demonstrated the effectiveness of rectified linear units in deep models in 2010.[^15] ReLU is linear for positive values and avoids the strong saturation of sigmoid functions in that region. It does not solve every optimization problem; permanently negative pre-activations, for example, can create “dead” units.

Later Transformer models often use smoother variants such as GELU (*Gaussian Error Linear Unit*):

```text
GELU(x) = x · Φ(x)
```

where `Φ` is the cumulative distribution function of the standard normal distribution.[^16] The historical evolution of activation functions illustrates a recurring pattern: architecture and optimization cannot be separated. A mathematically expressive network structure is of little use if gradients cannot be transported reliably across many layers.

---

## 7. Convolutional Neural Networks: Encoding Structural Knowledge in the Architecture

A fully connected network initially treats every input feature independently of its spatial position. For images, that is inefficient. A 224×224 RGB image already contains more than 150,000 input values; a fully connected layer with only 1,000 units would require more than 150 million weights.

Convolutional neural networks exploit two strong assumptions about image data: **local neighborhoods matter** and **the same visual features can occur at different positions**. Instead of learning separate weights for every image location, a small filter is slid across the image:

```text
Y[i,j] = Σ_m Σ_n K[m,n] · X[i+m, j+n].
```

This creates local receptive fields and **weight sharing**. The number of parameters drops dramatically while edges, textures, and increasingly abstract visual structures can be learned hierarchically.

Yann LeCun and colleagues demonstrated a practically successful, gradient-trained convolutional network for handwriting recognition with LeNet-5 in 1998.[^17] The highly visible breakthrough of modern deep learning followed in 2012 with AlexNet: Krizhevsky, Sutskever, and Hinton combined a deep CNN with GPUs, ReLUs, data augmentation, and additional training techniques and won the ImageNet Large Scale Visual Recognition Challenge by a substantial margin.[^18]

That success did not show that a fundamentally new learning algorithm had suddenly been invented in 2012. Many of the mathematical components had existed for decades. What mattered was that **data, architecture, optimization, and computing power reached a new performance level at the same time**.

---

## 8. Recurrent Networks and LSTM: Learning Across Time

Images have spatial structure; language, audio, and time series also have temporal order. Recurrent Neural Networks (RNNs) model this dependence by carrying a hidden state from one step to the next:

```text
h_t = φ(W_x x_t + W_h h_(t-1) + b)
```

The state `h_t` therefore depends not only on the current input `x_t`, but also on the previous internal representation. Conceptually, an RNN can be “unrolled” through time:

```text
x₁ -> [RNN] -> h₁ -> [RNN] -> h₂ -> [RNN] -> h₃
```

Training typically uses **Backpropagation Through Time (BPTT)**—the same chain rule applied across the unrolled temporal computation graph.[^19] This is exactly where the vanishing- and exploding-gradient problems become more severe: information and gradients must travel across many time steps.

Sepp Hochreiter and Jürgen Schmidhuber introduced the **Long Short-Term Memory** architecture in 1997 specifically to address this problem.[^20] LSTMs use a cell state and learnable gates controlling which information is stored, forgotten, or emitted.

A common formulation is:

```text
f_t = σ(W_f [h_(t-1), x_t] + b_f)       # Forget Gate
i_t = σ(W_i [h_(t-1), x_t] + b_i)       # Input Gate
o_t = σ(W_o [h_(t-1), x_t] + b_o)       # Output Gate

ĉ_t = tanh(W_c [h_(t-1), x_t] + b_c)
c_t = f_t ⊙ c_(t-1) + i_t ⊙ ĉ_t
h_t = o_t ⊙ tanh(c_t)
```

The additive path through the cell state makes it easier to preserve information and gradients over longer sequences. LSTMs therefore became a central architecture for speech recognition, machine translation, and other sequence tasks before Transformers took over many of those domains.

A common statement needs refinement, however: recurrence makes parallel computation **across the sequence dimension** difficult because `h_t` depends on `h_(t-1)`. It does not make parallelization “mathematically impossible.” Operations within a time step, different sequences in a batch, and parts of the linear algebra can certainly be parallelized. The critical bottleneck is the serial dependency between successive states.

---

## 9. What Remains from This Era: Representation, Differentiability, and Scalability

Between 1943 and the early 2000s, three principles were established without which today's generative AI is difficult to understand.

First, **learning as parameter optimization** was formalized. The perceptron showed that weights could be adapted from examples. Backpropagation made the same idea efficient for deeply nested, differentiable computation graphs.

Second, it became clear that **architecture embodies inductive assumptions**. CNNs exploit spatial locality and weight sharing; RNNs and LSTMs model temporal dependencies. A neural network is not merely “a set of weights,” but a structural hypothesis about how information should be processed.

Third, theoretical expressive power alone proved insufficient. A model must be trainable, must receive enough data, and must run on hardware that supports its computational structure efficiently. AlexNet was therefore as much a hardware and systems event as a model event.[^18]

The next major boundary lay in sequences. LSTMs could model long dependencies substantially better than simple RNNs, but they remained constrained by recurrent computation along the time axis. In 2017, an architecture appeared that did not merely improve recurrence, but largely replaced it for central sequence-modeling tasks: the **Transformer**.[^21]

How self-attention removed this serial dependency, why BERT and GPT emerged from it, and why scaling became a scientific strategy in its own right is the subject of **[Part 2: Attention Is All You Need – Transformers, OpenAI, Google, and the Era of LLMs](/blog/02-transformer-revolution-und-der-aufstieg-der-llms)**.

<KeyTakeaway>
**Core message of Part 1:** Deep learning did not emerge from a single invention. It became possible when four things came together: learnable parameters, nonlinear representations, efficient credit assignment through gradients, and architectures whose structure matched both the data and the available hardware.
</KeyTakeaway>

---

## Notes on the Sources

The history of backpropagation and the causes of early “AI winters” are weighted differently across the secondary literature. This article therefore avoids the simplified attribution that backpropagation was only “invented” in 1986 or that the book *Perceptrons* alone caused the first AI winter. The work of Rumelhart, Hinton, and Williams was extraordinarily important in popularizing the method; mathematical precursors and earlier applications already existed.[^11][^12][^13]

---

## 📚 Literature & Scientific Sources

[^1]: Schmidhuber, J. (2015). *Deep learning in neural networks: An overview.* Neural Networks, 61, 85–117. https://doi.org/10.1016/j.neunet.2014.09.003
[^2]: Schmidhuber, J. (2022). *Annotated History of Modern AI and Deep Neural Networks.* arXiv:2212.11279. https://arxiv.org/abs/2212.11279
[^3]: McCulloch, W. S., & Pitts, W. (1943). *A logical calculus of the ideas immanent in nervous activity.* Bulletin of Mathematical Biophysics, 5, 115–133. https://doi.org/10.1007/BF02478259
[^4]: Wiener, N. (1948). *Cybernetics: Or Control and Communication in the Animal and the Machine.* MIT Press / Wiley.
[^5]: McCarthy, J., Minsky, M. L., Rochester, N., & Shannon, C. E. (1955). *A Proposal for the Dartmouth Summer Research Project on Artificial Intelligence.* https://www-formal.stanford.edu/jmc/history/dartmouth/dartmouth.html
[^6]: Rosenblatt, F. (1958). *The perceptron: A probabilistic model for information storage and organization in the brain.* Psychological Review, 65(6), 386–408. https://doi.org/10.1037/h0042519
[^7]: Novikoff, A. B. J. (1962). *On convergence proofs on perceptrons.* Proceedings of the Symposium on the Mathematical Theory of Automata, 12, 615–622.
[^8]: Minsky, M., & Papert, S. (1969). *Perceptrons: An Introduction to Computational Geometry.* MIT Press.
[^9]: Crevier, D. (1993). *AI: The Tumultuous History of the Search for Artificial Intelligence.* Basic Books. See also ALPAC (1966), *Languages and Machines: Computers in Translation and Linguistics.* National Academy of Sciences.
[^10]: Hornik, K., Stinchcombe, M., & White, H. (1989). *Multilayer feedforward networks are universal approximators.* Neural Networks, 2(5), 359–366. https://doi.org/10.1016/0893-6080(89)90020-8
[^11]: Linnainmaa, S. (1970). *The representation of the cumulative rounding error of an algorithm as a Taylor expansion of the local rounding errors.* Master's thesis, University of Helsinki. Historical context: Schmidhuber (2015), DOI above.
[^12]: Werbos, P. J. (1974). *Beyond Regression: New Tools for Prediction and Analysis in the Behavioral Sciences.* Harvard University doctoral thesis. https://gwern.net/doc/ai/nn/1974-werbos.pdf
[^13]: Rumelhart, D. E., Hinton, G. E., & Williams, R. J. (1986). *Learning representations by back-propagating errors.* Nature, 323, 533–536. https://doi.org/10.1038/323533a0
[^14]: Hochreiter, S. (1991). *Untersuchungen zu dynamischen neuronalen Netzen.* Diploma thesis, Technische Universität München; see also Bengio, Y., Simard, P., & Frasconi, P. (1994). *Learning long-term dependencies with gradient descent is difficult.* IEEE Transactions on Neural Networks, 5(2), 157–166. https://doi.org/10.1109/72.279181
[^15]: Nair, V., & Hinton, G. E. (2010). *Rectified Linear Units Improve Restricted Boltzmann Machines.* ICML 2010. https://www.cs.toronto.edu/~hinton/absps/reluICML.pdf
[^16]: Hendrycks, D., & Gimpel, K. (2016). *Gaussian Error Linear Units (GELUs).* arXiv:1606.08415. https://arxiv.org/abs/1606.08415
[^17]: LeCun, Y., Bottou, L., Bengio, Y., & Haffner, P. (1998). *Gradient-based learning applied to document recognition.* Proceedings of the IEEE, 86(11), 2278–2324. https://doi.org/10.1109/5.726791
[^18]: Krizhevsky, A., Sutskever, I., & Hinton, G. E. (2012). *ImageNet Classification with Deep Convolutional Neural Networks.* Advances in Neural Information Processing Systems 25. https://proceedings.neurips.cc/paper/2012/hash/c399862d3b9d6b76c8436e924a68c45b-Abstract.html
[^19]: Werbos, P. J. (1990). *Backpropagation through time: what it does and how to do it.* Proceedings of the IEEE, 78(10), 1550–1560. https://doi.org/10.1109/5.58337
[^20]: Hochreiter, S., & Schmidhuber, J. (1997). *Long Short-Term Memory.* Neural Computation, 9(8), 1735–1780. https://doi.org/10.1162/neco.1997.9.8.1735
[^21]: Vaswani, A., et al. (2017). *Attention Is All You Need.* Advances in Neural Information Processing Systems 30. https://arxiv.org/abs/1706.03762
