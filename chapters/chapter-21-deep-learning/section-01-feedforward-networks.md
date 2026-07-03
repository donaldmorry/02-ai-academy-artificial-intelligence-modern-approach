# Section 21.1: Feedforward Networks

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 21.1  
> **Previous:** [Chapter 20 - Learning Probabilistic Models](../chapter-20-learning-probabilistic-models/README.md)  
> **Prerequisites:** [Chapter 19 - Learning from Examples](../chapter-19-learning-from-examples/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Perceptrons to Deep Representations

[Chapter 19](../chapter-19-learning-from-examples/README.md) introduced supervised learning with linear models, trees, and kernels. Chapter 21 asks a bolder question: **can we learn hierarchical representations** by stacking simple nonlinear units? A **feedforward neural network** (multilayer perceptron, MLP) composes affine transformations with element-wise nonlinearities to approximate complex input-output mappings.

Russell and Norvig frame neural networks as **function approximators inside learning agents**. The agent observes training pairs $(\mathbf{x}^{(i)}, y^{(i)})$, adjusts parameters $\theta$ to minimize empirical risk, and deploys the learned function for perception, planning features, or control signals.

> **Readable form:** A feedforward network is a pipeline of weighted sums followed by squashing functions - each layer builds features from the previous layer until the output answers your task.

Humorous analogy: a shallow perceptron is a single chef tasting one ingredient at a time; a deep MLP is a kitchen brigade where prep cooks, sauciers, and the head chef each transform the dish before plating.

---

## The Single Neuron

A neuron with input vector $\mathbf{x} \in \mathbb{R}^d$, weights $\mathbf{w}$, bias $b$, and activation $g$ computes:

$$
a = g(\mathbf{w}^\top \mathbf{x} + b)
$$
> **Readable form:** weighted sum of inputs, plus bias, passed through an activation function.

Common activations:

| Activation | Formula | Typical use |
|------------|---------|-------------|
| Sigmoid | $\sigma(z) = 1/(1+e^{-z})$ | Binary outputs (historical hidden layers) |
| Tanh | $\tanh(z)$ | Zero-centered hidden units |
| ReLU | $\max(0, z)$ | Modern hidden layers |
| Softmax | $e^{z_j}/\sum_k e^{z_k}$ | Multi-class output layer |

The **McCulloch-Pitts** and **Rosenblatt perceptron** (1950s-60s) used step activations for binary classification. Minsky and Papert (1969) showed a **single layer cannot represent XOR** - a pivotal moment in [AI history](../chapter-01-introduction/section-03-history-of-ai.md) that slowed connectionist research until multilayer networks and backpropagation revived it.

---

## Multilayer Architecture

An $L$-layer feedforward network maps input $\mathbf{x}$ through hidden layers $\mathbf{h}^{(1)}, \ldots, \mathbf{h}^{(L-1)}$ to output $\hat{\mathbf{y}}$:

$$
\mathbf{h}^{(\ell)} = g^{(\ell)}\!\left(\mathbf{W}^{(\ell)} \mathbf{h}^{(\ell-1)} + \mathbf{b}^{(\ell)}\right), \quad \mathbf{h}^{(0)} = \mathbf{x}
$$

$$
\hat{\mathbf{y}} = g^{(L)}\!\left(\mathbf{W}^{(L)} \mathbf{h}^{(L-1)} + \mathbf{b}^{(L)}\right)
$$
> **Readable form:** each layer applies a weight matrix and bias, then a nonlinearity; the last layer produces predictions.

**Why nonlinearity in hidden layers?** Without it, composing affine layers collapses to a single affine map - depth adds no expressive power. Nonlinear $g$ lets the network **fold and separate** input space in ways linear models cannot.

```python
import numpy as np

def relu(z):
    return np.maximum(0, z)

def forward_mlp(x, weights, biases, activations):
    h = x
    for W, b, g in zip(weights, biases, activations):
        h = g(W @ h + b)
    return h
```

---

## Universal Approximation

**Cybenko (1989)** and **Hornik et al.** established that a feedforward network with one sufficiently wide hidden layer and a nonpolynomial activation can approximate any continuous function on a compact domain to arbitrary accuracy - the **universal approximation theorem**.

$$
\forall f \in C(K), \forall \epsilon > 0,\ \exists \text{ network } \hat{f} \text{ s.t. } \sup_{\mathbf{x} \in K} |f(\mathbf{x}) - \hat{f}(\mathbf{x})| < \epsilon
$$
> **Readable form:** one wide hidden layer can approximate any continuous function as closely as you want - in theory.

**Caveats AIMA emphasizes:**

1. **Width may need to grow exponentially** with problem complexity - depth often more parameter-efficient.
2. The theorem guarantees existence, not **learnability** from finite data.
3. Generalization depends on regularization ([Section 21.3](./section-03-training-deep-networks.md)), not approximation alone.

---

## XOR: Why Depth Matters

XOR truth table: $(0,0)\!\to\!0$, $(0,1)\!\to\!1$, $(1,0)\!\to\!1$, $(1,1)\!\to\!0$. No single linear boundary separates label 1 from label 0. A **two-layer network** with two hidden ReLU units can carve two half-planes and combine them:

```python
# XOR with a tiny MLP (conceptual weights)
# h1 = relu(x1 + x2 - 0.5), h2 = relu(-x1 - x2 + 1.5)
# y  = sigmoid(h1 + h2 - 1.5)
```

This mirrors [Course 1's first Keras model](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/section-02-your-first-keras-model.md): stack `Dense` layers until the task becomes linearly separable in the final hidden space.

---

## Output Layers and Loss Functions

| Task | Output activation | Loss |
|------|-------------------|------|
| Regression | Linear (identity) | MSE, MAE |
| Binary classification | Sigmoid | Binary cross-entropy |
| Multi-class | Softmax | Categorical cross-entropy |

For softmax output with one-hot labels $\mathbf{y}$:

$$
\mathcal{L} = -\sum_j y_j \log \hat{y}_j
$$
> **Readable form:** cross-entropy penalizes confident wrong predictions heavily.

[Chapter 19](../chapter-19-learning-from-examples/README.md) logistic regression is a **single-layer network** with sigmoid output - the MLP generalizes this by learning hidden features instead of hand-specified ones.

---

## Parameter Count and Capacity

For layer sizes $n_0, n_1, \ldots, n_L$:

$$
|\theta| = \sum_{\ell=1}^{L} \left(n_{\ell} n_{\ell-1} + n_{\ell}\right)
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

A network with two hidden layers of 1000 units on 784-dimensional MNIST inputs has over **1.7 million** weights - enormous capacity relative to 60k training images. Without regularization, the network **memorizes** noise. This connects directly to bias-variance from Chapter 19 and dropout in [Section 21.3](./section-03-training-deep-networks.md).

---

## Feedforward Networks as Feature Learners

Hand-crafted features (SIFT, HOG) dominated vision before 2012. Deep feedforward layers **learn features end-to-end** from pixels or raw tokens. Early layers often detect edges and textures; deeper layers compose parts and objects - a pattern we revisit with convolutions in [Section 21.4](./section-04-convolutional-networks.md).

Russell and Norvig position feedforward networks as one tool in the agent's learning toolkit alongside probabilistic models ([Chapter 20](../chapter-20-learning-probabilistic-models/README.md)) and reinforcement learning ([Chapter 22](../chapter-22-reinforcement-learning/README.md)).

---

## Design Choices

| Choice | Consideration |
|--------|---------------|
| Depth vs width | Depth often improves compositional efficiency |
| Activation | ReLU default; sigmoid/tanh for gates in RNNs |
| Output size | Match task (1 for regression, $K$ for $K$ classes) |
| Weight init | Critical for deep nets ([Section 21.3](./section-03-training-deep-networks.md)) |

```python
# Keras MLP sketch for MNIST (Course 1 style)
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, Flatten

model = Sequential([
    Flatten(input_shape=(28, 28)),
    Dense(128, activation='relu'),
    Dense(64, activation='relu'),
    Dense(10, activation='softmax'),
])
```

---

## Common Pitfalls

1. **Linear activations in hidden layers** - collapses depth.
2. **Mismatched output activation and loss** - e.g., softmax with MSE.
3. **Ignoring input scaling** - gradient descent struggles on raw pixel values 0-255.
4. **Expecting universal approximation to imply easy training** - optimization is hard ([Section 21.2](./section-02-backpropagation.md)).

---

## Connection to Course 1

| Course 1 | AIMA link |
|----------|-----------|
| Chapter 08 - Why deep learning | [01-why-deep-learning.md](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-08-deep-learning/section-01-why-deep-learning.md): motivation before formalism |
| Chapter 09 - Keras MLP | [README.md](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/README.md): compile/fit workflow you now explain mathematically |
| Chapter 03 - MNIST | [08-digit-recognition-mnist.md](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/section-08-case-study-mnist-digit-recognition.md): same task, shallow vs deep |
| ML workflow | [06-ml-workflow.md](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-06-the-ml-workflow-and-data-hygiene.md): train/val/test discipline carries over |

---

## Connection to Course 3

| AIMA topic | Goodfellow et al. |
|------------|-------------------|
| XOR and depth | [Chapter 06](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) §6.1 |
| Hidden units | [Chapter 06](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) §6.3 - ReLU analysis |
| Universal approximation | [Chapter 06](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) §6.6 - depth efficiency |
| Architecture design | [Chapter 06](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) §6.4 |

---

## Reflection Questions

1. Why does composing two affine layers without nonlinearity reduce to one affine layer?
2. What problem did the XOR result create for early perceptron research?
3. How is logistic regression from Chapter 19 a special case of a neural network?
4. What does universal approximation guarantee, and what does it not guarantee?
5. When would you prefer a wide shallow network over a narrow deep one?

---

**Next:** [Section 21.2 - Backpropagation](./section-02-backpropagation.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 21. [AIMA](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. MIT Press. [deeplearningbook.org](https://www.deeplearningbook.org/)
4. Cybenko, G. (1989). "Approximation by superpositions of a sigmoidal function." *Mathematics of Control, Signals and Systems*.
5. Minsky, M., & Papert, S. (1969). *Perceptrons*. MIT Press.
