# Section 20.1: Statistical Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 20.1  
> **Previous:** [Chapter 19 - Learning from Examples](../chapter-19-learning-from-examples/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Supervised Learning to Probabilistic Models

[Chapter 19](../chapter-19-learning-from-examples/README.md) treated learning largely as **function fitting**: choose a hypothesis class, minimize empirical loss on labeled examples, and evaluate generalization. Chapter 20 shifts the lens to **probabilistic models** - explicit distributions over data and hidden structure, with parameters learned from observations.

Why bother? Many AI tasks are not "predict label $y$ from features $\mathbf{x}$" alone. We need:

| Task | Probabilistic framing |
|------|----------------------|
| **Density estimation** | What is $P(\mathbf{x})$? |
| **Clustering** | What latent groups generated the data? |
| **Missing data** | What values are likely for unobserved variables? |
| **Causal structure** | Which variables depend on which? |

> **Readable form:** Statistical learning asks "what distribution (or graphical model) best explains the data we saw?" - not only "what function maps inputs to outputs?"

Humorous analogy: supervised learning is like **memorizing exam answers**; statistical learning is like **building a weather model** - you explain patterns, quantify uncertainty, and predict what you have not yet measured.

---

## The Statistical Learning Setup

AIMA's Chapter 20 assumes:

1. **Data** $\mathcal{D} = \{\mathbf{x}^{(1)}, \ldots, \mathbf{x}^{(N)}\}$ - i.i.d. samples from an unknown distribution.
2. **Model family** $\mathcal{M}_\theta$ - parameterized by $\theta$ (e.g., Gaussian mean/variance, BN CPT entries).
3. **Learning** - infer $\theta$ (or a distribution over $\theta$) from $\mathcal{D}$.
4. **Goal** - high **likelihood** on held-out data, sensible **posterior** beliefs, or useful **latent** structure.

$$
P(\mathcal{D} \mid \theta) = \prod_{i=1}^{N} P(\mathbf{x}^{(i)} \mid \theta)
$$
> **Readable form:** Likelihood of the full dataset = product of each example's probability under parameters $\theta$ (assuming independent, identically distributed samples).

This connects directly to [maximum likelihood estimation](../../GLOSSARY.md#maximum-likelihood-estimation) in [Section 20.2](./section-02-maximum-likelihood.md) and [Bayesian learning](./section-03-bayesian-learning.md) when we place a prior on $\theta$.

---

## Parameters vs. Hyperparameters vs. Structure

| Concept | Example | Learned from data? |
|---------|---------|------------------|
| **Parameters** $\theta$ | Gaussian $\mu, \sigma^2$; BN CPT entries | Yes (MLE/MAP/Bayesian) |
| **Hyperparameters** | Number of mixture components $K$; Dirichlet concentration | Often cross-validation or model selection |
| **Structure** | BN directed edges | [Section 20.7](./section-07-structure-learning.md) |

[Chapter 13](../chapter-13-probabilistic-reasoning/README.md) gave us **inference** given a fixed model. Chapter 20 adds **learning** - the model itself is uncertain until we see data.

---

## Parametric vs. Nonparametric Models

**Parametric:** fixed finite $\theta$ regardless of $N$.

- Univariate Gaussian: $\theta = (\mu, \sigma^2)$
- Multinomial over $k$ categories: $\theta = (p_1, \ldots, p_k)$

**Nonparametric:** capacity grows with data (preview in [Section 20.8](./section-08-nonparametric-bayes-preview.md)).

- Dirichlet process mixtures: effectively unbounded $K$
- k-NN "model" stores all training points ([Course 1 k-NN](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-05-supervised-learning-k-nearest-neighbors.md))

| Approach | Strength | Weakness |
|----------|----------|----------|
| **Parametric** | Data-efficient, interpretable | Wrong family → bias |
| **Nonparametric** | Flexible | More data, harder inference |

---

## Complete vs. Incomplete Data

**Complete data:** every relevant variable is observed for every example.

- Estimate Gaussian $\mu$ as sample mean - closed form.
- Estimate BN CPTs by counting ([Section 20.6](./section-06-bayesian-network-parameter-learning.md)).

**Incomplete data:** some variables are hidden or missing.

- Mixture models: cluster identity $z^{(i)}$ unobserved → [EM algorithm](./section-04-em-algorithm.md).
- HMMs: hidden states → [Section 20.5](./section-05-em-applications.md), [Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md).
- Survey responses with skipped questions → same machinery.

$$
\log P(\mathcal{D}_{\text{obs}} \mid \theta) = \log \sum_{\mathbf{h}} P(\mathcal{D}_{\text{obs}}, \mathbf{h} \mid \theta)
$$
> **Readable form:** Log-likelihood with hidden variables requires summing (or integrating) over all possible completions $\mathbf{h}$ - often intractable directly.

---

## Generative vs. Discriminative (Revisited)

[Chapter 19](../chapter-19-learning-from-examples/README.md) contrasted models that learn $P(y \mid \mathbf{x})$ (discriminative) vs. $P(\mathbf{x}, y)$ (generative). Chapter 20 focuses on **generative** probabilistic models:

$$
P(\mathbf{x}) = \sum_{z} P(\mathbf{x} \mid z)\, P(z) \quad \text{(mixture)}
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Discriminative models can win on pure prediction accuracy; generative models support **imputation**, **simulation**, **anomaly detection**, and **structure discovery**.

| Model type | Typical use |
|------------|-------------|
| **Generative** | Clustering, density estimation, BN reasoning |
| **Discriminative** | Classification with sharp decision boundaries |

[Naive Bayes](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning) in Course 1 is a simple generative classifier - Chapter 20 generalizes the parameter-learning story.

---

## The Learning Pipeline

```
Data → Choose model family → Fit parameters (MLE / MAP / Bayesian)
     → Evaluate (held-out log-likelihood, BIC, downstream task)
     → Optionally learn structure (BN edges)
```

```python
import numpy as np
from scipy import stats

def fit_univariate_gaussian_mle(x):
    """MLE for N(mu, sigma^2) - preview of Section 20.2."""
    mu_hat = np.mean(x)
    sigma2_hat = np.var(x, ddof=0)  # MLE uses N denominator
    return mu_hat, sigma2_hat

# Example: 1000 draws from N(3, 4)
rng = np.random.default_rng(42)
x = rng.normal(loc=3.0, scale=2.0, size=1000)
mu_hat, sigma2_hat = fit_univariate_gaussian_mle(x)
print(f"mu_hat={mu_hat:.3f}, sigma2_hat={sigma2_hat:.3f}")
```

> **Readable form:** Learning here is "find the Gaussian that makes the observed numbers most probable" - the sample mean and (biased) sample variance.

---

## Priors and the Bayesian Perspective (Preview)

Even before full Bayesian treatment ([Section 20.3](./section-03-bayesian-learning.md)), AIMA emphasizes that **priors** encode background knowledge:

| Prior strength | Effect |
|----------------|--------|
| **Weak / vague** | Data dominate; close to MLE with enough $N$ |
| **Informative** | Stabilize estimates when data are scarce |
| **Conjugate** | Closed-form posterior updates |

A [Dirichlet prior](../../GLOSSARY.md) on multinomial parameters acts like **pseudo-counts** - developed fully in [Section 20.6](./section-06-bayesian-network-parameter-learning.md).

---

## Connection to Chapter 13: Bayesian Networks

A Bayesian network specifies:

$$
P(X_1, \ldots, X_n) = \prod_{i=1}^{n} P(X_i \mid \text{Parents}(X_i))
$$
> **Readable form:** The joint probability is built by multiplying the conditional factors specified by the model.

**Learning** splits into:

1. **Parameter learning** - CPT entries from data ([Section 20.6](./section-06-bayesian-network-parameter-learning.md)).
2. **Structure learning** - which edges exist ([Section 20.7](./section-07-structure-learning.md)).

**Inference** ([Chapter 13](../chapter-13-probabilistic-reasoning/README.md)) and **learning** compose the full BN workflow: learn structure and parameters, then answer queries.

---

## Connection to Course 1

| Course 1 topic | Chapter 20 view |
|----------------|-----------------|
| [k-means clustering](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md) | Hard assignment; EM gives soft probabilistic version |
| [Supervised vs. unsupervised](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-03-supervised-vs-unsupervised-learning.md) | Latent $z$ makes clustering a statistical estimation problem |
| [Overfitting](../../GLOSSARY.md#overfitting) | High-capacity mixtures / many BN parameters |
| [Cross-validation](../../GLOSSARY.md#cross-validation) | Choose $K$, hyperparameters, structure score |

---

## Connection to Course 3

| Course 3 topic | Chapter 20 link |
|----------------|-----------------|
| [ML basics](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-05-machine-learning-basics/README.md) | Log-likelihood, regularization as priors |
| [Linear factor models](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-13-linear-factor-models/README.md) | Latent variables + EM for PPCA/factor analysis |
| [Regularization](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-07-regularization/README.md) | MAP estimation = MLE + penalty |

Deep generative models ([Course 3 Chapter 20](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-20-deep-generative-models/README.md)) replace hand-crafted densities with neural networks, but the **latent-variable** logic of Chapter 20 remains.

---

## Model Selection and Occam's Razor

More complex models fit training data better but may generalize worse. Chapter 20 uses:

| Criterion | Idea |
|-----------|------|
| **Held-out log-likelihood** | Direct predictive score |
| **BIC** | Penalize parameter count: $\log P(\mathcal{D} \mid \hat{\theta}) - \frac{d}{2}\log N$ |
| **Bayesian model evidence** | Integrate over $\theta$ |

Structure learning ([Section 20.7](./section-07-structure-learning.md)) relies heavily on scoring trade-offs between fit and complexity.

---

## Worked Mini-Example: Coin Flips as Multinomial

Observe 7 heads, 3 tails. Model: $P(H) = \theta$, $P(T) = 1-\theta$.

$$
\hat{\theta}_{\text{MLE}} = \frac{7}{10} = 0.7
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

With $\text{Beta}(2,2)$ prior (pseudo 2 heads, 2 tails), MAP shifts toward 0.5 - [Section 20.3](./section-03-bayesian-learning.md) derives this.

| Method | $\hat{\theta}$ | Interpretation |
|--------|----------------|----------------|
| MLE | 0.70 | Pure frequency |
| MAP (Beta 2,2) | 0.64 | Shrunk toward prior mean 0.5 |

---

## Common Pitfalls

**Confusing likelihood and probability.** $P(\mathcal{D} \mid \theta)$ is a function of $\theta$ for fixed data - not a distribution over datasets.

**Ignoring i.i.d. assumption.** Sequential data need temporal models ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)).

**Treating hyperparameters as afterthought.** $K$ in a mixture is not learned by standard EM without model selection.

**Using generative models for discrimination without calibration.** High $P(\mathbf{x})$ does not always yield best classifiers.

---

## Reflection Questions

1. How does "learning a probabilistic model" differ from the empirical risk minimization view in [Chapter 19](../chapter-19-learning-from-examples/README.md)?
2. Why does incomplete data force summation over hidden variables in the likelihood?
3. When would you prefer a generative mixture model over k-means for clustering?
4. What role do priors play when $N$ is small?
5. How do parameter learning and structure learning decompose the BN learning problem?

---

**Next:** [Section 20.2 - Maximum Likelihood](./section-02-maximum-likelihood.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 20.1. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Bishop, C. M. (2006). *Pattern Recognition and Machine Learning*, Chapter 2 - probability distributions and MLE. [Springer](https://www.microsoft.com/en-us/research/people/cmbishop/)
3. Murphy, K. P. (2022). *Probabilistic Machine Learning: An Introduction*, Chapters 3-4. [probml.ai](https://probml.ai/)
4. AIMA Python - learning probabilistic models. [aima-python](https://github.com/aimacode/aima-python)
5. Stanford CS228 - probabilistic graphical models. [Course notes](https://cs228.stanford.edu/)



