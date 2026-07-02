# Section 20.2: Maximum Likelihood

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 20.2  
> **Previous:** [Section 20.1 - Statistical Learning](./section-01-statistical-learning.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Maximum Likelihood Principle

[Section 20.1](./section-01-statistical-learning.md) posed learning as choosing parameters $\theta$ for a model family. **Maximum likelihood estimation (MLE)** picks the $\theta$ that makes the observed data most probable:

$$
\hat{\theta}_{\text{MLE}} = \arg\max_{\theta} \, P(\mathcal{D} \mid \theta)
= \arg\max_{\theta} \prod_{i=1}^{N} P(\mathbf{x}^{(i)} \mid \theta)
$$
> **Readable form:** The joint probability is built by multiplying the conditional factors specified by the model.

Equivalently, maximize the **log-likelihood** (sum instead of product, numerically stable):

$$
\hat{\theta}_{\text{MLE}} = \arg\max_{\theta} \sum_{i=1}^{N} \log P(\mathbf{x}^{(i)} \mid \theta)
$$
> **Readable form:** MLE = find parameters that assign the highest total log-probability to every training example you actually saw.

Humorous analogy: MLE is the **prosecutor's choice of theory** - "which parameter values make the evidence we collected most likely?" It is not guaranteed to be the "true" world, only the best explanation under the model class.

---

## Why Log-Likelihood?

| Issue with product | Log fix |
|--------------------|---------|
| Underflow for large $N$ | Sum of logs |
| Harder calculus | Derivatives of log often simplify |
| Monotonic transform | Same argmax as raw likelihood |

For Gaussians, logs turn products of exponentials into quadratics - connecting MLE to [ordinary least squares](../../GLOSSARY.md#ordinary-least-squares) in linear regression ([Course 1](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-02-regression-models)).

---

## Example 1: Univariate Gaussian

Model: $X \sim \mathcal{N}(\mu, \sigma^2)$.

$$
\log P(\mathcal{D} \mid \mu, \sigma^2) = -\frac{N}{2}\log(2\pi\sigma^2) - \frac{1}{2\sigma^2}\sum_{i=1}^{N}(x^{(i)} - \mu)^2
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Setting derivatives to zero:

\hat{\mu}_{\text{MLE}} = \frac{1}{N}\sum_{i=1}^{N} x^{(i)}, \quad
\hat{\sigma}^2_{\text{MLE}} = \frac{1}{N}\sum_{i=1}^{N}(x^{(i)} - \hat{\mu})^2
> **Readable form:** MLE Gaussian = sample mean and sample variance (divide by $N$, not $N-1$).

```python
import numpy as np

def gaussian_log_likelihood(x, mu, sigma2):
    n = len(x)
    return (-0.5 * n * np.log(2 * np.pi * sigma2)
            - 0.5 * np.sum((x - mu) ** 2) / sigma2)

x = np.array([2.1, 2.8, 3.0, 3.2, 15.0])  # one outlier
mu_hat = x.mean()
sigma2_hat = ((x - mu_hat) ** 2).mean()
print(f"log L = {gaussian_log_likelihood(x, mu_hat, sigma2_hat):.2f}")
```

The outlier inflates $\hat{\sigma}^2$ - MLE is sensitive to misspecified tails (motivation for robust models).

---

## Example 2: Bernoulli / Binomial

$N$ flips, $n_H$ heads. $\theta = P(H)$.

$$
\hat{\theta}_{\text{MLE}} = \frac{n_H}{N}
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

Log-likelihood curve is concave → unique global maximum.

| Observed | MLE $\hat{\theta}$ |
|----------|-------------------|
| 70 heads / 100 | 0.70 |
| 0 heads / 100 | 0.00 (boundary) |

---

## Example 3: Multinomial (Categorical)

Counts $n_1, \ldots, n_k$ for $k$ categories, $\sum_j n_j = N$.

$$
\hat{p}_j^{\text{MLE}} = \frac{n_j}{N}, \quad j = 1, \ldots, k
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

Constraint $\sum_j p_j = 1$ enforced via Lagrange multiplier. This is the frequency estimator behind **maximum likelihood CPT learning** for BN nodes with complete data ([Section 20.6](./section-06-bayesian-network-parameter-learning.md)).

---

## Example 4: Multivariate Gaussian

$\mathbf{x} \in \mathbb{R}^d$, parameters $\boldsymbol{\mu}$, $\boldsymbol{\Sigma}$.

\hat{\boldsymbol{\mu}}_{\text{MLE}} = \frac{1}{N}\sum_{i=1}^{N} \mathbf{x}^{(i)}, \quad
\hat{\boldsymbol{\Sigma}}_{\text{MLE}} = \frac{1}{N}\sum_{i=1}^{N}(\mathbf{x}^{(i)} - \hat{\boldsymbol{\mu}})(\mathbf{x}^{(i)} - \hat{\boldsymbol{\mu}})^\top

Requires $N > d$ for invertible $\boldsymbol{\Sigma}$ - otherwise singular covariance. Regularization (shrink $\boldsymbol{\Sigma}$) links to MAP / Bayesian views ([Section 20.3](./section-03-bayesian-learning.md)).

---

## MLE as Empirical Risk Minimization

Negative log-likelihood is a **loss function**:

$$
\mathcal{L}(\theta) = -\frac{1}{N}\sum_{i=1}^{N} \log P(\mathbf{x}^{(i)} \mid \theta)
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

| Model | NLL loss | Course 1 analog |
|-------|----------|-----------------|
| Gaussian regression | Squared error (up to constants) | [Linear regression](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-02-regression-models) |
| Logistic | Cross-entropy | Classification |
| Multinomial | Cross-entropy over classes | Softmax classifier |

[Chapter 19](../chapter-19-learning-from-examples/README.md) empirical risk minimization **is** MLE when the loss is negative log-likelihood of a correctly specified model.

---

## Properties of MLE

**Consistency:** as $N \to \infty$, $\hat{\theta}_{\text{MLE}} \to \theta^*$ (true parameter) under regularity conditions.

**Asymptotic normality:**

$$
\sqrt{N}(\hat{\theta}_{\text{MLE}} - \theta^*) \xrightarrow{d} \mathcal{N}(0, \mathcal{I}^{-1}(\theta^*))
$$
> **Readable form:** This formal sentence is the symbolic version of the relationship stated in the surrounding explanation.

where $\mathcal{I}$ is the Fisher information matrix.

**Invariance:** if $\hat{\theta}$ is MLE for $\theta$, then $g(\hat{\theta})$ is MLE for $g(\theta)$.

| Property | Practical meaning |
|----------|-------------------|
| Consistency | More data → better estimates |
| Asymptotic efficiency | Best among "well-behaved" estimators at large $N$ |
| Finite-sample bias | Small $N$ → overfitting risk |

---

## Overfitting and MLE

MLE with rich models can **memorize** noise:

- High-degree polynomial regression → low training error, wild extrapolation.
- Full covariance with $N \approx d$ → spurious correlations.
- Too many mixture components → each point its own tiny Gaussian.

```
Training log-likelihood:  always increases with model capacity
Test log-likelihood:      rises then falls  →  overfitting peak
```

| Symptom | MLE behavior |
|---------|--------------|
| Training NLL very low | Model may be too flexible |
| Test NLL much worse | Poor generalization |
| Boundary estimates ($\hat{p}=0$) | Zero probability to unseen events |

[Cross-validation](../../GLOSSARY.md#cross-validation) from [Chapter 19](../chapter-19-learning-from-examples/README.md) applies directly when tuning hyperparameters like mixture count $K$.

---

## Regularization as MAP (Bridge)

Pure MLE has no penalty for complexity. **Maximum a posteriori (MAP)** adds a prior ([Section 20.3](./section-03-bayesian-learning.md)):

$$
\hat{\theta}_{\text{MAP}} = \arg\max_{\theta} \left[ \log P(\mathcal{D} \mid \theta) + \log P(\theta) \right]
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

| Prior | MAP effect | Course 3 link |
|-------|------------|---------------|
| Gaussian on weights | L2 penalty | [L2 weight decay](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-07-regularization/section-02-l2-and-l1-weight-decay.md) |
| Laplace on weights | L1 sparsity | L1 regularization |
| Dirichlet on probabilities | Smoothed counts | Pseudo-counts in BN CPTs |

> **Readable form:** Regularization = "don't move parameters far from what we believed before seeing data."

---

## When MLE Fails Directly: Latent Variables

If data include hidden variables $\mathbf{z}^{(i)}$, the marginal likelihood is:

$$
P(\mathbf{x}^{(i)} \mid \theta) = \sum_{\mathbf{z}} P(\mathbf{x}^{(i)}, \mathbf{z} \mid \theta)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

No closed-form maximization for mixtures, HMMs, or factor analysis. **EM** ([Section 20.4](./section-04-em-algorithm.md)) iteratively optimizes a surrogate - each step increases log-likelihood.

| Setting | Direct MLE? |
|---------|-------------|
| Complete Gaussian | Yes - closed form |
| Gaussian mixture | No - need EM |
| BN with missing values | No - need EM or MCMC |
| HMM | No - Baum-Welch = EM |

---

## Numerical Optimization for MLE

When no closed form exists, use gradient ascent on $\ell(\theta) = \sum_i \log P(\mathbf{x}^{(i)} \mid \theta)$:

```python
import numpy as np

def mle_bernoulli_gradient_ascent(counts_heads, n_trials, lr=0.01, steps=500):
    """MLE for Bernoulli via gradient ascent on log-likelihood."""
    theta = 0.5
    for _ in range(steps):
        # d/dtheta [ n_H log theta + n_T log(1-theta) ]
        grad = counts_heads / theta - (n_trials - counts_heads) / (1 - theta)
        theta += lr * grad
        theta = np.clip(theta, 1e-6, 1 - 1e-6)
    return theta

print(mle_bernoulli_gradient_ascent(70, 100))  # ≈ 0.70
```

Modern deep learning ([Course 3](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md)) is largely MLE (or MAP) with neural network parameterizations of $P(\mathbf{x} \mid \theta)$.

---

## MLE for Bayesian Network Parameters (Preview)

With complete data and a fixed BN structure, each CPT entry is a **local** MLE problem - count and normalize:

$$
\hat{\theta}_{ijk} = \frac{N_{ijk}}{N_{ij\cdot}}
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

where $N_{ijk}$ counts how often child $X_i=k$ given parent configuration $j$. [Section 20.6](./section-06-bayesian-network-parameter-learning.md) adds Dirichlet priors.

---

## Comparison Table: Estimators

| Estimator | Formula flavor | Small $N$ | Large $N$ |
|-----------|----------------|-----------|-----------|
| **MLE** | Pure data fit | Overfit risk | Consistent |
| **MAP** | Data + prior | Stabilized | ≈ MLE |
| **Posterior mean** | Full Bayesian | Uncertainty quantified | Computationally heavier |

---

## Connection to Course 1

| Course 1 method | MLE interpretation |
|-----------------|-------------------|
| Sample mean / variance | Gaussian MLE |
| OLS regression | Gaussian noise MLE for linear model |
| Logistic regression | Iterative MLE (gradient descent) |
| Naive Bayes counts | MLE of conditional probabilities (with smoothing as MAP) |

---

## Common Mistakes

**Maximizing $P(\theta \mid \mathcal{D})$ without Bayes' rule** - that is MAP/Bayesian, not MLE.

**Using $N-1$ variance** - unbiased for population variance, but not the MLE formula.

**Assuming MLE is always unique** - mixtures are notorious for multimodal likelihood surfaces.

**Ignoring constraints** - probabilities must sum to 1; use Lagrange multipliers or reparameterization.

---

## Reflection Questions

1. Derive $\hat{\mu}_{\text{MLE}}$ for a univariate Gaussian by differentiating the log-likelihood.
2. Why does MLE variance use divisor $N$ rather than $N-1$?
3. How is minimizing squared error related to Gaussian MLE in regression?
4. What happens to MLE $\hat{p}$ for a category never seen in training?
5. Why can't we set derivatives to zero for a two-component Gaussian mixture?

---

**Next:** [Section 20.3 - Bayesian Learning](./section-03-bayesian-learning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 20.2. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Casella, G., & Berger, R. (2002). *Statistical Inference*, Chapter 7 - point estimation. [Cengage](https://www.cengage.com/)
3. Bishop, C. M. (2006). *Pattern Recognition and Machine Learning*, Section 2.1 - MLE examples. [Microsoft Research](https://www.microsoft.com/en-us/research/people/cmbishop/)
4. Murphy, K. P. (2022). *Probabilistic Machine Learning*, Section 4.2 - MLE and MAP. [probml.ai](https://probml.ai/)
5. MIT 18.650 - Statistics for Applications, MLE lectures. [OpenCourseWare](https://ocw.mit.edu/)

