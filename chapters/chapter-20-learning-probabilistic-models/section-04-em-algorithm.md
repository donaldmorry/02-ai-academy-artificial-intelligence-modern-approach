# Section 20.4: EM Algorithm

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 20.4  
> **Previous:** [Section 20.3 - Bayesian Learning](./section-03-bayesian-learning.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Latent Variable Problem

Many models posit **unobserved** [latent variables](../../GLOSSARY.md#latent-variable) $\mathbf{z}$ that generate observed data $\mathbf{x}$. Examples:

| Model | Observed | Latent |
|-------|----------|--------|
| **Gaussian mixture** | Points $\mathbf{x}^{(i)}$ | Component $z^{(i)} \in \{1,\ldots,K\}$ |
| **HMM** | Emissions | Hidden states |
| **Factor analysis** | Features | Low-dim factors |

The marginal log-likelihood is:

$$
\ell(\theta) = \sum_{i=1}^{N} \log \sum_{\mathbf{z}^{(i)}} P(\mathbf{x}^{(i)}, \mathbf{z}^{(i)} \mid \theta)
$$
> **Readable form:** For each data point, sum over all possible hidden explanations, take log, then sum over points - differentiation is messy because $\log$ wraps a $\sum$.

Direct [MLE](./section-02-maximum-likelihood.md) via gradient methods is possible but the **Expectation-Maximization (EM)** algorithm provides a principled iterative scheme with closed-form steps for many models.

Humorous analogy: EM is **detective work with amnesia** - guess who did it (E-step), update your theory of criminals (M-step), repeat until the story stabilizes.

---

## EM: High-Level Idea

Given current parameters $\theta^{(t)}$, EM alternates:

1. **E-step (Expectation):** Compute expected sufficient statistics of latent variables under $P(\mathbf{z} \mid \mathbf{x}, \theta^{(t)})$.
2. **M-step (Maximization):** Set $\theta^{(t+1)}$ to maximize expected complete-data log-likelihood as if $\mathbf{z}$ were observed.

Each iteration **increases** (or leaves unchanged) the observed-data log-likelihood $\ell(\theta)$. EM climbs a hill but may find a **local** maximum.

---

## Complete-Data vs. Incomplete-Data Log-Likelihood

**Complete-data** log-likelihood (if we knew all $\mathbf{z}^{(i)}$):

$$
\ell_c(\theta) = \sum_{i=1}^{N} \log P(\mathbf{x}^{(i)}, \mathbf{z}^{(i)} \mid \theta)
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

**Q-function** - expected complete log-likelihood w.r.t. latent posterior:

$$
Q(\theta \mid \theta^{(t)}) = \mathbb{E}_{\mathbf{z} \mid \mathbf{x}, \theta^{(t)}} \left[ \ell_c(\theta) \right]
$$
> **Readable form:** Q = average complete-data log-likelihood over all plausible hidden assignments, weighted by how likely each is given current parameters.

**M-step:**

$$
\theta^{(t+1)} = \arg\max_{\theta} \, Q(\theta \mid \theta^{(t)})
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

---

## General EM Algorithm (AIMA)

```
Initialize parameters θ⁽⁰⁾
Repeat until convergence:
    E-step: Compute Q(θ | θ⁽ᵗ⁾) or equivalently expected sufficient statistics
    M-step: θ⁽ᵗ⁺¹⁾ = argmax_θ Q(θ | θ⁽ᵗ⁾)
Return θ
```

| Step | Computes | Analogy |
|------|----------|---------|
| **E** | Soft assignments to latent values | "Who belongs to which cluster?" |
| **M** | Parameters that best fit those assignments | "Where are cluster centers?" |

This is **soft** k-means when $z$ is cluster ID - compare [Course 1 k-means](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md) (hard assignment).

---

## Mixture of Gaussians: Model

$K$ components. Latent $z^{(i)} \in \{1,\ldots,K\}$ with $P(z=k) = \pi_k$, $\sum_k \pi_k = 1$.

$$
P(\mathbf{x}^{(i)} \mid z^{(i)}=k, \theta) = \mathcal{N}(\mathbf{x}^{(i)} \mid \boldsymbol{\mu}_k, \boldsymbol{\Sigma}_k)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Joint:

$$
P(\mathbf{x}^{(i)}, z^{(i)}=k \mid \theta) = \pi_k \, \mathcal{N}(\mathbf{x}^{(i)} \mid \boldsymbol{\mu}_k, \boldsymbol{\Sigma}_k)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Marginal (mixture density):

$$
P(\mathbf{x}^{(i)} \mid \theta) = \sum_{k=1}^{K} \pi_k \, \mathcal{N}(\mathbf{x}^{(i)} \mid \boldsymbol{\mu}_k, \boldsymbol{\Sigma}_k)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Parameters: $\theta = \{\pi_k, \boldsymbol{\mu}_k, \boldsymbol{\Sigma}_k\}_{k=1}^K$.

---

## Mixture of Gaussians: E-Step

**Responsibility** - posterior probability point $i$ belongs to component $k$:

\gamma_{ik}^{(t)} = P(z^{(i)}=k \mid \mathbf{x}^{(i)}, \theta^{(t)})
= \frac{\pi_k^{(t)} \, \mathcal{N}(\mathbf{x}^{(i)} \mid \boldsymbol{\mu}_k^{(t)}, \boldsymbol{\Sigma}_k^{(t)})}
{\sum_{j=1}^{K} \pi_j^{(t)} \, \mathcal{N}(\mathbf{x}^{(i)} \mid \boldsymbol{\mu}_j^{(t)}, \boldsymbol{\Sigma}_j^{(t)})}
> **Readable form:** Responsibility of cluster $k$ for point $i$ = (weight of $k$ × Gaussian density at $i$) / (sum over all clusters of same quantity).

$\gamma_{ik}$ replaces hard 0/1 assignments from k-means.

---

## Mixture of Gaussians: M-Step

Effective counts and weighted statistics:

$$
N_k^{(t)} = \sum_{i=1}^{N} \gamma_{ik}^{(t)}
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Mixing weights:**

$$
\pi_k^{(t+1)} = \frac{N_k^{(t)}}{N}
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

**Means:**

$$
\boldsymbol{\mu}_k^{(t+1)} = \frac{1}{N_k^{(t)}} \sum_{i=1}^{N} \gamma_{ik}^{(t)} \, \mathbf{x}^{(i)}
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Covariances** (full):

$$
\boldsymbol{\Sigma}_k^{(t+1)} = \frac{1}{N_k^{(t)}} \sum_{i=1}^{N} \gamma_{ik}^{(t)} (\mathbf{x}^{(i)} - \boldsymbol{\mu}_k^{(t+1)})(\mathbf{x}^{(i)} - \boldsymbol{\mu}_k^{(t+1)})^\top
$$
> **Readable form:** M-step = weighted k-means update where weights are responsibilities - cluster mean is weighted average of all points.

**Spherical** variant: use $\sigma_k^2 \mathbf{I}$ - single variance per cluster. **Diagonal** $\boldsymbol{\Sigma}_k$: only diagonal entries updated.

---

## Full Implementation Sketch

```python
import numpy as np
from scipy.stats import multivariate_normal

def em_gaussian_mixture(X, K, max_iter=100, tol=1e-4, reg_cov=1e-6):
    """
    EM for K-component Gaussian mixture (full covariance).
    X: (N, d) data matrix
    """
    N, d = X.shape
    rng = np.random.default_rng(0)

    # Random initialization from data
    idx = rng.choice(N, K, replace=False)
    mu = X[idx].copy()
    Sigma = np.array([np.cov(X, rowvar=False) + reg_cov * np.eye(d) for _ in range(K)])
    pi = np.ones(K) / K

    log_liks = []
    for _ in range(max_iter):
        # E-step: responsibilities (N, K)
        resp = np.zeros((N, K))
        for k in range(K):
            resp[:, k] = pi[k] * multivariate_normal.pdf(X, mean=mu[k], cov=Sigma[k])
        resp /= resp.sum(axis=1, keepdims=True) + 1e-12

        # M-step
        Nk = resp.sum(axis=0)
        pi = Nk / N
        mu_new = (resp.T @ X) / Nk[:, None]
        Sigma_new = []
        for k in range(K):
            diff = X - mu_new[k]
            Sk = (resp[:, k][:, None] * diff).T @ diff / Nk[k]
            Sigma_new.append(Sk + reg_cov * np.eye(d))
        Sigma = np.array(Sigma_new)

        # Log-likelihood monitoring
        ll = 0.0
        for i in range(N):
            ll += np.log(sum(pi[k] * multivariate_normal.pdf(X[i], mu[k], Sigma[k])
                               for k in range(K)) + 1e-12)
        log_liks.append(ll)

        if len(log_liks) > 1 and abs(log_liks[-1] - log_liks[-2]) < tol:
            break
        mu = mu_new

    return pi, mu, Sigma, resp, log_liks
```

---

## Convergence Properties

| Property | Statement |
|----------|-----------|
| **Monotonicity** | $\ell(\theta^{(t+1)}) \geq \ell(\theta^{(t)})$ |
| **Stationary points** | Converges to local max / saddle of $\ell(\theta)$ |
| **Global optimum** | **Not** guaranteed for mixtures |
| **Rate** | Often linear near optimum |

**Initialization matters:** random restarts, k-means++ seeds, or hierarchical methods. The chapter lab asks you to plot log-likelihood vs. iteration - it should climb smoothly.

---

## Identifiability and Symmetries

Permuting component labels leaves likelihood unchanged - $\pi_1, \mu_1$ swapped with $\pi_2, \mu_2$ is the same model. EM breaks symmetry only through initialization.

**Collapsed components:** if $\pi_k \to 0$, component $k$ disappears (singularities if $\boldsymbol{\Sigma}_k \to 0$ at a single point). Use **regularization** $\boldsymbol{\Sigma}_k + \epsilon \mathbf{I}$.

---

## EM as Lower-Bound Optimization

EM maximizes a **lower bound** on $\ell(\theta)$ via Jensen's inequality - the Q-function is a tight bound at $\theta^{(t)}$. This connects to variational inference in [Course 3 Chapter 19](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-19-approximate-inference/README.md).

---

## Comparison: k-Means vs. EM-GMM

| Aspect | k-means | EM (GMM) |
|--------|---------|----------|
| Assignment | Hard (nearest center) | Soft (responsibilities) |
| Shape | Spherical clusters (implicit) | Ellipsoids via $\boldsymbol{\Sigma}_k$ |
| Probabilistic | No density model | Full $P(\mathbf{x})$ |
| Objective | Minimize inertia | Maximize log-likelihood |

[Course 1 clustering](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md) is faster; GMMs support **density estimation** and **principled model selection** via BIC.

---

## Choosing $K$

EM does not select $K$ automatically. Use:

| Method | Description |
|--------|-------------|
| **BIC** | Penalize parameter count |
| **Cross-validation** | Held-out log-likelihood |
| **Elbow on log-likelihood** | Diminishing returns |

[Section 20.7](./section-07-structure-learning.md) discusses similar scoring for BN structure.

---

## When EM Struggles

| Issue | Mitigation |
|-------|------------|
| Local optima | Multiple restarts |
| Slow convergence | Accelerated EM, smart init |
| High-dimensional $\boldsymbol{\Sigma}_k$ | Diagonal / tied covariances |
| Many components | Nonparametric Bayes ([Section 20.8](./section-08-nonparametric-bayes-preview.md)) |

---

## Connection to Chapter 14

Baum-Welch for HMMs **is** EM - [Section 20.5](./section-05-em-applications.md) and [Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md).

---

## Connection to Course 3

[Probabilistic PCA and factor analysis](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-13-linear-factor-models/README.md) derive EM for latent linear models - same E/M pattern with different sufficient statistics.

---

## Reflection Questions

1. Why does the E-step compute $P(z \mid \mathbf{x}, \theta)$ rather than only the most likely $z$?
2. Show that k-means is a limiting case of EM for GMM with $\boldsymbol{\Sigma}_k = \epsilon \mathbf{I}$ as $\epsilon \to 0$.
3. What happens in the M-step if all responsibilities $\gamma_{ik}$ are zero for component $k$?
4. Why does EM guarantee monotonic increase of log-likelihood but not global optimality?
5. How many parameters in a $K$-component $d$-dimensional full-covariance GMM?

---

**Next:** [Section 20.5 - EM Applications](./section-05-em-applications.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 20.4. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Dempster, A. P., Laird, N. M., & Rubin, D. B. (1977). Maximum likelihood from incomplete data via the EM algorithm. *JRSS B*. [JSTOR](https://www.jstor.org/)
3. Bishop, C. M. (2006). *Pattern Recognition and Machine Learning*, Section 9.2 - mixture models and EM. [Microsoft Research](https://www.microsoft.com/en-us/research/people/cmbishop/)
4. Murphy, K. P. (2022). *Probabilistic Machine Learning*, Chapter 8 - EM algorithm. [probml.ai](https://probml.ai/)
5. McLachlan, G., & Krishnan, T. (2008). *The EM Algorithm and Extensions*. [Wiley](https://www.wiley.com/)


