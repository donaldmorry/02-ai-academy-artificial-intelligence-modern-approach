# Section 21.8: Unsupervised Deep Learning Preview

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 21.8  
> **Previous:** [Section 21.7 - Applications](./section-07-applications.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Learning Without Labels

Sections 21.1-21.7 focused on **supervised** deep learning - labeled $(\mathbf{x}, y)$ pairs drive backpropagation. Chapter 21.8 previews **unsupervised** and **self-supervised** approaches: learning structure from unlabeled data $\{\mathbf{x}^{(i)}\}$ alone.

Why does AIMA include this in a deep learning chapter? Three reasons:

1. **Labels are expensive** - images, text, and video exist in abundance without annotations.
2. **Pretraining** historically bootstrapped supervised tasks (pre-2012 deep learning revival).
3. **Generative models** - agents that simulate environments or impute missing data - require unsupervised objectives.

Russell and Norvig connect to [Chapter 20](../chapter-20-learning-probabilistic-models/README.md) (density estimation) and forward to generative deep learning in later courses.

> **Readable form:** unsupervised deep learning finds patterns in raw data without being told the right answer for each example.

Humorous analogy: supervised learning is flashcards with answers on the back; unsupervised learning is a toddler sorting toys by shape without anyone naming the shapes - sometimes they invent "categories" you did not expect.

---

## Unsupervised vs Self-Supervised

| Paradigm | Labels | Example |
|----------|--------|---------|
| **Supervised** | Human-provided | ImageNet class IDs |
| **Unsupervised** | None | Clustering, density estimation |
| **Self-supervised** | Automatically generated from data | Predict masked words (BERT) |

Modern "unsupervised" deep learning often means **self-supervised pretraining** + supervised fine-tuning - the boundary blurs. AIMA uses "unsupervised" broadly for **objectives that do not require external labels**.

---

## Autoencoders

An **autoencoder** compresses input through an **encoder** $f_\text{enc}$ into a **code** (bottleneck) $\mathbf{z}$, then reconstructs via **decoder** $f_\text{dec}$:

$$
\mathbf{z} = f_\text{enc}(\mathbf{x}; \theta_\text{enc}), \quad
\hat{\mathbf{x}} = f_\text{dec}(\mathbf{z}; \theta_\text{dec})
$$
> **Readable form:** The encoder compresses input into latent code $z$, and the decoder reconstructs the input from that code.

**Reconstruction loss:**

$$
\mathcal{L}(\mathbf{x}) = \|\mathbf{x} - \hat{\mathbf{x}}\|^2 \quad \text{(MSE)} \quad \text{or} \quad \text{BCE for binary pixels}
$$
> **Readable form:** squeeze the input through a narrow layer, then expand back - penalize failing to reconstruct the original.

### Undercomplete Autoencoder

Bottleneck dimension $d_{\text{code}} < d_{\text{input}}$ forces **compression** - learns a nonlinear generalization of PCA ([Course 1 Chapter 06](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-06-principal-component-analysis/README.md)).

```python
# Keras autoencoder sketch
encoder = Sequential([Dense(128, activation='relu'), Dense(32, activation='relu')])
decoder = Sequential([Dense(128, activation='relu'), Dense(784, activation='sigmoid')])
# Train on x with target x - unsupervised
```

### Overcomplete Autoencoders

Code dimension larger than input - needs **regularization** (sparsity penalty, dropout) to prevent identity mapping.

---

## Denoising Autoencoders

**Denoising autoencoder (DAE)** corrupts input $\tilde{\mathbf{x}}$ (additive noise, masking) and trains to recover clean $\mathbf{x}$:

$$
\mathcal{L} = \|\mathbf{x} - f_\text{dec}(f_\text{enc}(\tilde{\mathbf{x}}))\|^2
$$
> **Readable form:** show the network a corrupted version and teach it to restore the original - learns robust features, not just copy.

DAEs learn **manifold structure** - data concentrate on lower-dimensional surfaces in high-dimensional space. Useful for **denoising** and **anomaly detection** (high reconstruction error on OOD inputs).

---

## Sparse and Contractive Autoencoders

**Sparsity:** penalize average activation of hidden units toward a small target $\rho$ (KL penalty) - encourages **specialized detectors** (edge-like filters resembling CNN early layers).

**Contractive:** penalize $\|\partial f_\text{enc}/\partial \mathbf{x}\|_F^2$ - insensitivity to small input perturbations, smoother representations.

[Course 3 Chapter 14 - Autoencoders](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-14-autoencoders/README.md) develops these variants rigorously.

---

## Historical Pretraining (2006-2012)

Before ImageNet-scale supervised CNNs, **layer-wise pretraining** with Restricted Boltzmann Machines (RBMs) and autoencoders initialized deep belief networks:

1. Train RBM/autoencoder on layer 1
2. Freeze, train layer 2 on layer-1 representations
3. Stack and **fine-tune** with supervised labels

Hinton et al. (2006) showed deep nets could be trained this way - a milestone toward modern deep learning. **Supervised end-to-end training** with ReLU, better init, and GPUs largely superseded layer-wise pretraining for classification, but **unsupervised pretraining** resurged as **self-supervised learning** (SimCLR, MAE, BERT).

| Era | Dominant strategy |
|-----|-------------------|
| 2006-2011 | Layer-wise unsupervised pretraining |
| 2012-2016 | Supervised CNN end-to-end |
| 2018-present | Self-supervised pretrain + fine-tune |

---

## Generative Models Preview

Autoencoders **reconstruct** but do not define a tractable density $P(\mathbf{x})$. **Generative adversarial networks (GANs)** and **variational autoencoders (VAEs)** extend toward sampling:

| Model | Idea | AIMA / course placement |
|-------|------|-------------------------|
| VAE | Probabilistic encoder; maximize ELBO | Course 4 generative chapters |
| GAN | Generator vs discriminator game | Course 4 |
| Diffusion | Iterative denoising | Course 4 |

Chapter 21.8 **previews** these - full treatment exceeds AIMA Ch. 21 scope. Connection: unsupervised objectives learn **world models** useful for [Chapter 22 RL](../chapter-22-reinforcement-learning/README.md) and simulation.

---

## Self-Supervised Learning on Images

Without class labels, create **pretext tasks**:

| Method | Pretext task |
|--------|--------------|
| Rotation prediction | Predict 0°/90°/180°/270° rotation |
| Jigsaw | Reassemble shuffled patches |
| Contrastive (SimCLR) | Match augmented views of same image |

Learned representations transfer to classification with **linear probe** or fine-tuning - often competitive with supervised pretraining on small labeled sets.

> **Readable form:** teach the network "these two cropped views are the same photo" - it learns useful vision features without human class names.

Links to [Section 21.4](./section-04-convolutional-networks.md) CNN backbones and [Chapter 25](../chapter-25-computer-vision/README.md).

---

## Self-Supervised Learning on Text

**Language modeling:** predict next token $P(x_t \mid x_{<t})$ - no human labels needed; web text provides supervision.

$$
\mathcal{L} = -\sum_t \log P(x_t \mid x_1, \ldots, x_{t-1}; \theta)
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

**Masked language modeling (BERT):** predict masked tokens from context - bidirectional representations.

These pretrained models fine-tune on labeled tasks ([Chapter 24](../chapter-24-deep-learning-nlp/README.md)) - the dominant NLP paradigm since 2018.

---

## Clustering with Deep Features

Classical k-means ([Course 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md)) in pixel space works poorly on complex images. Pipeline:

1. Train autoencoder or use pretrained CNN
2. Extract code or penultimate layer features
3. Cluster in embedding space

**Deep embedded clustering (DEC)** jointly optimizes representation and cluster assignments - bridges [Chapter 19](../chapter-19-learning-from-examples/README.md) unsupervised themes with deep nets.

---

## Evaluation Challenges

Unsupervised objectives lack a single "accuracy." Evaluation strategies:

| Method | Metric |
|--------|--------|
| Reconstruction | MSE, PSNR on holdout data |
| Downstream transfer | Linear probe accuracy on labeled task |
| Clustering | NMI, ARI vs known labels (if available) |
| Generative | FID, IS for images; human evaluation |

Never tune on test labels leaked through repeated probing - hold out a **downstream validation set**.

---

## When to Use Unsupervised Deep Learning

| Scenario | Approach |
|----------|----------|
| Few labels, many unlabeled images | Self-supervised pretrain + fine-tune |
| Anomaly detection | Autoencoder reconstruction threshold |
| Dimensionality reduction | Undercomplete autoencoder |
| Exploration in RL | Learn latent dynamics model |
| Full generative sampling | VAE/GAN/diffusion (beyond this preview) |

If labels are abundant and task is fixed, **supervised end-to-end** ([Sections 21.1-21.7](./section-01-feedforward-networks.md)) remains simpler.

---

## Connection to Course 1

| Course 1 | AIMA link |
|----------|-----------|
| k-means clustering | [04-k-means-clustering.md](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md): classical unsupervised baseline |
| PCA | [Chapter 06](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-06-principal-component-analysis/README.md): linear compression vs autoencoder |
| Supervised vs unsupervised | [03-supervised-vs-unsupervised.md](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-03-supervised-vs-unsupervised-learning.md): paradigm framing |
| Chapter 08 | [01-why-deep-learning.md](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-08-deep-learning/section-01-why-deep-learning.md): representation learning narrative |

---

## Connection to Course 3

| AIMA topic | Goodfellow et al. |
|------------|-------------------|
| Autoencoders | [Chapter 14](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-14-autoencoders/README.md) - full chapter |
| Representation learning | [Chapter 15](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-15-representation-learning/README.md) |
| Linear factor models / PCA link | [Chapter 13](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-13-linear-factor-models/README.md) |
| Deep generative models | [Chapter 20](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-20-deep-generative-models/README.md) |

---

## Chapter Summary

Chapter 21 progressed from **feedforward approximators** through **backprop**, **training craft**, **CNNs**, **RNNs/LSTM**, **methodology**, and **applications**, ending with **unsupervised preview**. You can now:

- Read AIMA Ch. 21 as a coherent map of deep learning
- Connect [Course 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/README.md) practice to formal derivations
- Pursue [Course 3 Chapters 06-09](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) for mathematical depth

**Next chapter:** [Chapter 22 - Reinforcement Learning](../chapter-22-reinforcement-learning/README.md) - learning from rewards rather than labels.

---

## Reflection Questions

1. How does an undercomplete autoencoder relate to PCA?
2. Why does a denoising autoencoder learn more useful features than a plain autoencoder?
3. What problem ended layer-wise RBM pretraining's dominance, and what revived similar ideas?
4. How is language modeling "unsupervised" despite producing accurate next-word predictions?
5. How would you evaluate whether unsupervised pretraining helped a downstream classifier?

---

**Previous:** [Section 21.7 - Applications](./section-07-applications.md) · **Next:** [Chapter 22 - Reinforcement Learning](../chapter-22-reinforcement-learning/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 21. [AIMA](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. Ch. 14-15. [deeplearningbook.org](https://www.deeplearningbook.org/)
4. Hinton, G., & Salakhutdinov, R. (2006). "Reducing the dimensionality of data with neural networks." *Science*.
5. Vincent, P. et al. (2008). "Extracting and composing robust features with denoising autoencoders." *ICML*.
