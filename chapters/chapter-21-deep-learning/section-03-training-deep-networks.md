# Section 21.3: Training Deep Networks

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 21.4-21.5  
> **Previous:** [Section 21.2 - Backpropagation](./section-02-backpropagation.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Beyond Backpropagation

[Section 21.2](./section-02-backpropagation.md) showed how to compute gradients. **Training deep networks** is the art of turning those gradients into stable, generalizable learning on high-dimensional data. AIMA Section 21.4 covers optimization (SGD, momentum, Adam), regularization (weight decay, dropout, data augmentation), and normalization (batch norm). Section 21.5 addresses **generalization** - why test error matters more than training loss.

> **Readable form:** backprop tells you which direction to move each weight; training deep networks is everything else that keeps the optimization from exploding, stalling, or memorizing the training set.

Humorous analogy: backprop is the steering wheel; initialization, learning rate, and regularization are the suspension, brakes, and traction control that keep the car on the road.

---

## The Training Loop

```
for epoch in 1..E:
    for minibatch B in training_data:
        y_hat = forward(x_B; theta)
        loss = L(y_hat, y_B) + lambda * Omega(theta)
        theta <- theta - eta * grad_theta(loss)
    evaluate on validation set
```

| Symbol | Meaning |
|--------|---------|
| $\eta$ | Learning rate (step size) |
| $\lambda$ | Regularization strength |
| $\Omega(\theta)$ | Penalty on weights (e.g., $\|\theta\|_2^2$) |

**Empirical risk minimization** from [Chapter 19](../chapter-19-learning-from-examples/README.md) - now with millions of parameters and non-convex loss surfaces.

---

## Stochastic Gradient Descent and Variants

Full-batch gradient on all $N$ examples:

$$
\theta \leftarrow \theta - \eta \nabla_\theta \frac{1}{N}\sum_{i=1}^{N} \mathcal{L}^{(i)}
$$
> **Readable form:** This derivative tells how the loss changes when the parameter changes; backprop uses it to decide the update direction.

**SGD** uses a mini-batch $\mathcal{B}$ of size $m \ll N$:

$$
\theta \leftarrow \theta - \eta \frac{1}{m}\sum_{i \in \mathcal{B}} \nabla_\theta \mathcal{L}^{(i)}
$$
> **Readable form:** estimate the true gradient from a random subset - noisier but far cheaper per step, and noise can help escape shallow local minima.

**Momentum** accumulates a velocity vector:

$$
\mathbf{v} \leftarrow \beta \mathbf{v} + \nabla_\theta J, \quad \theta \leftarrow \theta - \eta \mathbf{v}
$$
> **Readable form:** This derivative measures local sensitivity: a larger magnitude means the output changes more for a small input change.

**Adam** adapts per-parameter learning rates using running estimates of first and second moments of gradients - default choice in many frameworks ([Course 1 Keras](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/section-02-your-first-keras-model.md)).

| Optimizer | When useful |
|-----------|-------------|
| SGD + momentum | Classic baseline, good with tuned schedule |
| Adam | Fast convergence, forgiving defaults |
| Learning-rate decay | Required for fine convergence |

---

## Weight Initialization

Poor initialization can **vanish** or **explode** activations before training begins. For layer $\ell$ with $n_{\text{in}}$ inputs:

**Xavier (Glorot)** for tanh/sigmoid:

$$
W_{ij} \sim \mathcal{U}\left(-\sqrt{\frac{6}{n_{\text{in}} + n_{\text{out}}}}, \sqrt{\frac{6}{n_{\text{in}} + n_{\text{out}}}}\right)
$$
> **Readable form:** Xavier initialization samples each weight uniformly within bounds determined by the layer's input and output widths.

**He initialization** for ReLU:

$$
W_{ij} \sim \mathcal{N}\left(0, \frac{2}{n_{\text{in}}}\right)
$$
> **Readable form:** scale random weights inversely with layer width so signal variance stays roughly constant across layers.

Zero initialization fails - all hidden units stay symmetric. Always break symmetry with small random weights.

---

## Batch Normalization

**Batch normalization** standardizes activations within each mini-batch:

\hat{z}_j = \frac{z_j - \mu_{\mathcal{B}}}{\sqrt{\sigma_{\mathcal{B}}^2 + \epsilon}}, \quad
y_j = \gamma_j \hat{z}_j + \beta_j

Learnable scale $\gamma_j$ and shift $\beta_j$ restore expressiveness. Benefits:

| Benefit | Mechanism |
|---------|-----------|
| Faster training | Reduces internal covariate shift |
| Higher learning rates | Stabilizes gradient magnitudes |
| Mild regularization | Noise from batch statistics |

Apply before or after activation depending on architecture - follow convention of the reference model you reproduce.

---

## Regularization

### Weight decay (L2)

$$
\Omega(\theta) = \frac{1}{2}\|\theta\|_2^2 \quad \Rightarrow \quad \theta \leftarrow (1 - \eta\lambda)\theta - \eta \nabla_\theta \mathcal{L}
$$
> **Readable form:** This derivative tells how the loss changes when the parameter changes; backprop uses it to decide the update direction.

Equivalent to **Gaussian prior** on weights - connects to [Chapter 20](../chapter-20-learning-probabilistic-models/README.md) MAP learning.

### Dropout

During training, randomly zero each unit with probability $p$:

$$
h_j' = \frac{1}{1-p} \cdot m_j h_j, \quad m_j \sim \text{Bernoulli}(1-p)
$$
> **Readable form:** randomly silence neurons so the network cannot rely on any single path - forces redundant representations (ensemble effect).

At test time, use all units without dropout (or weight scaling as above).

### Data augmentation

For images: random crops, flips, color jitter. Artificially expands training set without new labels - critical for vision ([Section 21.4](./section-04-convolutional-networks.md)).

---

## Learning Rate Schedules

| Schedule | Formula sketch |
|----------|----------------|
| Step decay | $\eta \leftarrow \eta / k$ every $T$ epochs |
| Cosine | $\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max}-\eta_{\min})(1 + \cos(\pi t / T))$ |
| Warmup | Linear increase for first $W$ steps, then decay |

**Early stopping:** halt when validation loss stops improving - simplest regularizer against overfitting ([Chapter 19 - Model Selection](../chapter-19-learning-from-examples/section-08-model-selection.md)).

---

## Monitoring Training

Track **both** training and validation curves:

```
Loss
  |  train ----
  |  val   ----  (gap = overfitting)
  +-------------------> epoch
```

| Pattern | Diagnosis |
|---------|-----------|
| Train ↓, val ↓ | Learning |
| Train ↓, val flat then ↑ | Overfitting |
| Both flat | Learning rate too low or vanishing gradients |
| Both explode | Learning rate too high or bad initialization |

---

## Residual Connections (Preview)

**ResNet** blocks add skip connections ([Section 21.4](./section-04-convolutional-networks.md)):

$$
\mathbf{z}^{(i)} = g_r\big(\mathbf{z}^{(i-1)} + f(\mathbf{z}^{(i-1)})\big)
$$
> **Readable form:** each layer learns a small correction to the previous representation instead of replacing it entirely - gradients flow through identity shortcuts.

Mitigates vanishing gradients in very deep networks (50-200+ layers).

---

## Worked Example: MNIST MLP

Two hidden layers (128, 64), ReLU, softmax output, cross-entropy loss.

| Hyperparameter | Typical value |
|----------------|---------------|
| Batch size | 128 |
| Learning rate | $10^{-3}$ (Adam) |
| Epochs | 20-50 |
| Weight decay | $10^{-4}$ |
| Dropout | 0.2 on hidden layers |

Expected: ~98% test accuracy with modest architecture. Deeper nets need careful init + batch norm.

---

## Python Sketch: Training Loop with Validation

```python
import numpy as np

def train_epoch(model, X, y, batch_size=128, lr=1e-3):
    n = len(X)
    perm = np.random.permutation(n)
    for start in range(0, n, batch_size):
        idx = perm[start:start + batch_size]
        loss, grads = model.loss_and_grad(X[idx], y[idx])
        model.apply_grads(grads, lr)

def fit(model, X_train, y_train, X_val, y_val, epochs=30, lr=1e-3):
    history = {"train": [], "val": []}
    for ep in range(epochs):
        train_epoch(model, X_train, y_train, lr=lr)
        history["train"].append(model.evaluate(X_train, y_train))
        history["val"].append(model.evaluate(X_val, y_val))
    return history
```

> **Readable form:** shuffle data each epoch, update on mini-batches, log validation metrics to catch overfitting.

---

## Connection to Course 1

| Topic | Course 1 link |
|-------|---------------|
| Keras `fit` callbacks | [02-your-first-keras-model.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/section-02-your-first-keras-model.md) |
| CNN training | [04-cnn-image-classification.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-10-convolutional-neural-networks/README.md) |
| Overfitting / dropout | [Chapter 19](../chapter-19-learning-from-examples/README.md) bias-variance |

---

## Connection to Course 3

Rigorous treatment of optimization and generalization: [Course 3 Chapter 07](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-07-regularization/README.md), [Chapter 08](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-08-optimization/README.md).

---

## Common Mistakes

**Using training accuracy alone.** Validation set is mandatory.

**Learning rate too large.** Loss spikes or becomes NaN - reduce $\eta$ by 10×.

**Forgetting to scale inputs.** Normalize features to zero mean, unit variance (or $[0,1]$ for images).

**Dropout at test time.** Disable dropout during inference.

**No weight decay on biases.** Often biases are excluded from L2 penalty - check framework defaults.

---

## Reflection Questions

1. Why does mini-batch SGD prefer GPU hardware with high parallelism?
2. How does dropout relate to training an ensemble of sub-networks?
3. What problem does batch normalization address that weight initialization alone cannot?
4. When would you choose SGD with momentum over Adam?
5. How does early stopping implement a form of regularization?

---

**Previous:** [Section 21.2 - Backpropagation](./section-02-backpropagation.md) · **Next:** [Section 21.4 - Convolutional Networks](./section-04-convolutional-networks.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), §§21.4-21.5. [AIMA](https://aima.cs.berkeley.edu/)
2. Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*, Ch. 7-8. [deeplearningbook.org](https://www.deeplearningbook.org/)
3. Ioffe, S., & Szegedy, C. (2015). Batch normalization. *ICML*.
4. Srivastava, N., et al. (2014). Dropout. *JMLR*.
5. He, K., et al. (2015). Delving deep into rectifiers. *ICCV* (He initialization).
