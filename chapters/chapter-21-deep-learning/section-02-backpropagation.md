# Section 21.2: Backpropagation

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 21.2  
> **Previous:** [Section 21.1 - Feedforward Networks](./section-01-feedforward-networks.md)  
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Learning Problem

[Section 21.1](./section-01-feedforward-networks.md) defined feedforward networks as parameterized functions $f(\mathbf{x}; \theta)$. **Learning** means finding $\theta$ that minimizes a loss $\mathcal{L}$ on training data. For a single example:

$$
\mathcal{L}^{(i)} = \ell\big(f(\mathbf{x}^{(i)}; \theta), y^{(i)}\big)
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

Empirical risk:

$$
J(\theta) = \frac{1}{N} \sum_{i=1}^{N} \mathcal{L}^{(i)} + \lambda \Omega(\theta)
$$
> **Readable form:** average prediction error on training data, plus optional penalty on large weights.

[Chapter 19](../chapter-19-learning-from-examples/README.md) used gradient descent on linear models. Neural networks reuse the same idea but need an efficient way to compute $\nabla_\theta J$ when $\theta$ contains millions of entries spread across layers. **Backpropagation** (Rumelhart, Hinton, Williams, 1986) is that algorithm - not a separate learning rule, but **efficient gradient computation** via the chain rule.

Humorous analogy: backprop is like a factory quality inspector working backward from the final product defect to identify which station on the assembly line caused it.

---

## Computational Graphs

View the network as a **directed acyclic graph** (DAG). Each node applies an operation; edges carry values forward and gradients backward.

```
x → [W1,b1] → z1 → [ReLU] → h1 → [W2,b2] → z2 → [softmax] → y_hat → [loss]
```

Forward pass: compute activations topologically. Backward pass: apply chain rule in reverse order.

> **Readable form:** forward pass computes predictions; backward pass distributes blame for the error back to each parameter.

Modern frameworks ([Course 1 Keras](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/section-01-keras-and-tensorflow-setup.md), PyTorch) build this graph automatically - **automatic differentiation**. AIMA still expects you to derive gradients by hand for at least one layer to understand what the framework computes.

---

## Two-Layer Network Notation

Consider one hidden layer:

$$
\mathbf{z}^{(1)} = \mathbf{W}^{(1)} \mathbf{x} + \mathbf{b}^{(1)}, \quad \mathbf{h} = g(\mathbf{z}^{(1)})
$$
> **Readable form:** The first layer forms an affine preactivation from input features, then applies nonlinearity to produce hidden activations.

$$
\mathbf{z}^{(2)} = \mathbf{W}^{(2)} \mathbf{h} + \mathbf{b}^{(2)}, \quad \hat{\mathbf{y}} = \mathrm{softmax}(\mathbf{z}^{(2)})
$$
> **Readable form:** Softmax turns raw scores into a normalized distribution, so larger logits receive more probability mass.

Cross-entropy loss for one-hot label $\mathbf{y}$:

$$
\mathcal{L} = -\sum_j y_j \log \hat{y}_j
$$
> **Readable form:** softmax turns logits into probabilities; cross-entropy measures how far predicted probabilities are from the true label.

---

## Output Layer Gradients

Define **error signal** at output logits $\delta^{(2)} = \partial \mathcal{L} / \partial \mathbf{z}^{(2)}$. For softmax + cross-entropy, the combined gradient has an elegant form:

$$
\delta^{(2)} = \hat{\mathbf{y}} - \mathbf{y}
$$
> **Readable form:** output error equals predicted probability minus true label - the same as logistic regression.

Weight and bias gradients:

$$
\frac{\partial \mathcal{L}}{\partial \mathbf{W}^{(2)}} = \delta^{(2)} \mathbf{h}^\top, \quad
\frac{\partial \mathcal{L}}{\partial \mathbf{b}^{(2)}} = \delta^{(2)}
$$
> **Readable form:** This derivative tells how the loss changes when the parameter changes; backprop uses it to decide the update direction.

---

## Backpropagation to Hidden Layer

Chain rule through the hidden activation:

$$
\delta^{(1)} = \big(\mathbf{W}^{(2)\top} \delta^{(2)}\big) \odot g'(\mathbf{z}^{(1)})
$$
> **Readable form:** push output error back through output weights, then multiply by the activation derivative element-wise.

For ReLU, $g'(z) = 1$ if $z > 0$ else $0$. Hidden weight gradients:

$$
\frac{\partial \mathcal{L}}{\partial \mathbf{W}^{(1)}} = \delta^{(1)} \mathbf{x}^\top, \quad
\frac{\partial \mathcal{L}}{\partial \mathbf{b}^{(1)}} = \delta^{(1)}
$$
> **Readable form:** This derivative tells how the loss changes when the parameter changes; backprop uses it to decide the update direction.

This pattern **generalizes to arbitrary depth**: for layer $\ell$,

$$
\delta^{(\ell)} = \big(\mathbf{W}^{(\ell+1)\top} \delta^{(\ell+1)}\big) \odot g'\big(\mathbf{z}^{(\ell)}\big)
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

$$
\frac{\partial \mathcal{L}}{\partial \mathbf{W}^{(\ell)}} = \delta^{(\ell)} \big(\mathbf{h}^{(\ell-1)}\big)^\top
$$
> **Readable form:** This derivative tells how the loss changes when the parameter changes; backprop uses it to decide the update direction.

---

## Full Derivation Sketch (Scalar Chain Rule)

For scalar weight $w_{jk}^{(\ell)}$ connecting unit $k$ at layer $\ell-1$ to unit $j$ at layer $\ell$:

\frac{\partial \mathcal{L}}{\partial w_{jk}^{(\ell)}} =
\frac{\partial \mathcal{L}}{\partial z_j^{(\ell)}} \cdot
\frac{\partial z_j^{(\ell)}}{\partial w_{jk}^{(\ell)}} =
\delta_j^{(\ell)} \cdot h_k^{(\ell-1)}

Summing over all paths in the graph equals the multivariate chain rule - backprop avoids redundant computation by **reusing** $\delta$ vectors layer by layer. Complexity is **O(number of parameters)** - same order as one forward pass.

---

## Mini-Batch Stochastic Gradient Descent

Full-batch gradient descent on all $N$ examples is expensive. **SGD** updates on mini-batches $\mathcal{B}$:

$$
\theta \leftarrow \theta - \eta \frac{1}{|\mathcal{B}|} \sum_{i \in \mathcal{B}} \nabla_\theta \mathcal{L}^{(i)}
$$
> **Readable form:** take a small random sample of examples, compute average gradient, nudge weights opposite to that gradient.

| Batch size | Trade-off |
|------------|-----------|
| 1 | Noisy, fast per step |
| 32-256 | Standard GPU sweet spot |
| $N$ | Smooth but slow |

[Course 1's `model.fit`](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/section-02-your-first-keras-model.md) hides this loop; understanding backprop explains what `loss.backward()` triggers in PyTorch ([Course 3 Chapter 06](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md)).

---

## Implementation Pattern

```python
def train_step(model, x_batch, y_batch, lr=0.01):
    # Forward
    y_hat = model.forward(x_batch)
    loss = cross_entropy(y_hat, y_batch)
    # Backward (framework computes deltas internally)
    grads = model.backward(loss)
    for param, grad in zip(model.params, grads):
        param -= lr * grad
```

Manual implementation steps for layer $\ell$:

1. Cache $\mathbf{h}^{(\ell-1)}$ and $\mathbf{z}^{(\ell)}$ during forward pass.
2. Receive $\delta^{(\ell)}$ from layer above.
3. Compute $\partial \mathcal{L}/\partial \mathbf{W}^{(\ell)}$ and $\partial \mathcal{L}/\partial \mathbf{b}^{(\ell)}$.
4. Pass $\delta^{(\ell-1)}$ downward.

---

## Vanishing and Exploding Gradients (Preview)

Repeated multiplication by $\mathbf{W}^{(\ell)\top}$ and $g'(\cdot)$ across many layers can **shrink** or **grow** $\delta$ exponentially. Sigmoid/tanh saturate ($g' \approx 0$), causing **vanishing gradients** in early layers. ReLU helps but does not eliminate the problem in very deep nets - addressed in [Section 21.3](./section-03-training-deep-networks.md) (initialization, normalization, ResNet) and [Section 21.5](./section-05-recurrent-networks.md) (LSTM gates).

$$
\|\delta^{(1)}\| \approx \|\delta^{(L)}\| \prod_{\ell=2}^{L} \|\mathbf{W}^{(\ell)\top}\| \, |g'|
$$
> **Readable form:** gradient magnitude at the bottom layer roughly equals output gradient times products of weight and activation derivatives - deep products go tiny or huge fast.

---

## Verifying Gradients

**Numerical gradient check** compares analytical backprop to finite differences:

$$
\frac{\partial \mathcal{L}}{\partial \theta_j} \approx \frac{\mathcal{L}(\theta + \epsilon \mathbf{e}_j) - \mathcal{L}(\theta - \epsilon \mathbf{e}_j)}{2\epsilon}
$$
> **Readable form:** This derivative tells how the loss changes when the parameter changes; backprop uses it to decide the update direction.

Use $\epsilon \approx 10^{-5}$ for float64. Discrepancies flag implementation bugs - essential when coding from scratch in [Course 3 Chapter 06](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md).

---

## Backprop Through Time (Preview)

Recurrent networks ([Section 21.5](./section-05-recurrent-networks.md)) unroll the same weights across time steps. Backpropagation on the unrolled graph is **BPTT** - same chain rule, shared parameters summed across time. Truncated BPTT limits unroll length for memory efficiency.

---

## Connection to Course 1

| Course 1 | AIMA link |
|----------|-----------|
| Keras compile + fit | [02-your-first-keras-model.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/section-02-your-first-keras-model.md): backprop runs inside `fit` |
| TensorFlow setup | [01-keras-tensorflow-setup.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/section-01-keras-and-tensorflow-setup.md): autodiff engine |
| Network sizing | [03-network-sizing-architecture.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/section-03-network-sizing-and-architecture.md): depth affects gradient paths |
| Chapter 19 gradient descent | [Chapter 19](../chapter-19-learning-from-examples/README.md): same update rule, harder gradients |

---

## Connection to Course 3

| AIMA topic | Goodfellow et al. |
|------------|-------------------|
| Computational graphs | [Chapter 06](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) §6.2 |
| Full backprop derivation | [Chapter 06](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) §6.5 |
| Autograd comparison | [Chapter 06](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) §6.9 |
| Vanishing gradients | [Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) §10.2 |

---

## Reflection Questions

1. Why is backpropagation considered efficient compared to naive per-weight finite differences?
2. Derive $\delta^{(2)} = \hat{\mathbf{y}} - \mathbf{y}$ for softmax + cross-entropy.
3. What role does $g'(\mathbf{z})$ play when propagating errors to earlier layers?
4. Why do saturated sigmoid hidden units hinder learning in deep networks?
5. How would you verify your backprop implementation before training on real data?

---

**Previous:** [Section 21.1 - Feedforward Networks](./section-01-feedforward-networks.md) · **Next:** [Section 21.3 - Training Deep Networks](./section-03-training-deep-networks.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 21. [AIMA](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. MIT Press. [deeplearningbook.org](https://www.deeplearningbook.org/) Ch. 6 - backpropagation and computational graphs
4. Rumelhart, D., Hinton, G., & Williams, R. (1986). "Learning representations by back-propagating errors." *Nature*.
5. Baydin, A. et al. (2018). "Automatic differentiation in machine learning: a survey." *JMLR*.
