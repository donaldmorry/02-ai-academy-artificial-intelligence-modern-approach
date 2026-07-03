# Section 20.5: EM Applications

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 20.4-20.5  
> **Previous:** [Section 20.4 - EM Algorithm](./section-04-em-algorithm.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## EM as a Universal Template

[Section 20.4](./section-04-em-algorithm.md) derived EM for **Gaussian mixtures**. The same E-step / M-step pattern applies wherever:

1. Complete-data log-likelihood $\ell_c(\theta)$ has tractable M-step.
2. Posterior over latent variables $P(\mathbf{z} \mid \mathbf{x}, \theta)$ is computable.

This section covers three major applications AIMA emphasizes: **HMM parameter learning (Baum-Welch)**, **probabilistic clustering**, and **missing data imputation**.

> **Readable form:** EM = "guess hidden stuff (E), refit parameters as if guesses were true (M)" - template reused across models.

Humorous analogy: Baum-Welch is EM wearing a **trench coat labeled "speech recognition"** - same algorithm, different latent variables.

---

## Application 1: Hidden Markov Models (Baum-Welch)

[Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md) introduced HMMs: hidden states $X_t$, observations $E_t$.

Parameters $\theta = (A, B, \pi)$:

| Symbol | Meaning |
|--------|---------|
| $A_{ij}$ | $P(X_t=j \mid X_{t-1}=i)$ - transition |
| $B_{jk}$ | $P(E_t=k \mid X_t=j)$ - emission |
| $\pi_j$ | $P(X_0=j)$ - initial distribution |

**Latent variables:** entire state sequence $\mathbf{X} = (X_0, \ldots, X_T)$.

**Observed:** emission sequence $\mathbf{e} = (e_1, \ldots, e_T)$.

---

## HMM E-Step: Expected State Occupancies

Forward-backward algorithm computes:

$$
\gamma_t(j) = P(X_t = j \mid \mathbf{e}, \theta^{(t)})
$$

$$
\xi_t(i, j) = P(X_t = i, X_{t+1} = j \mid \mathbf{e}, \theta^{(t)})
$$
> **Readable form:** $\gamma_t(j)$ = soft belief state $j$ at time $t$; $\xi_t(i,j)$ = soft belief of transitioning $i \to j$ between $t$ and $t+1$.

These are **expected sufficient statistics** for the M-step - analogous to responsibilities $\gamma_{ik}$ in mixtures.

---

## HMM M-Step: Parameter Updates

$$
\pi_j^{(t+1)} = \gamma_0(j)
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

$$
A_{ij}^{(t+1)} = \frac{\sum_{t=0}^{T-1} \xi_t(i, j)}{\sum_{t=0}^{T-1} \gamma_t(i)}
$$

$$
B_{jk}^{(t+1)} = \frac{\sum_{t: e_t = k} \gamma_t(j)}{\sum_{t=0}^{T} \gamma_t(j)}
$$
> **Readable form:** Transition $i \to j$ = fraction of expected transitions from $i$ that go to $j$; emission $k$ from state $j$ = fraction of time in $j$ when symbol $k$ was seen.

**Baum-Welch** = EM for HMMs. Viterbi finds **most likely** state sequence; Baum-Welch learns **parameters** by marginalizing over all sequences.

---

## Baum-Welch Code Sketch

```python
import numpy as np

def baum_welch_discrete(obs, n_states, n_symbols, n_iter=50):
    """
  obs: integer sequence of emissions (0 .. n_symbols-1)
  Returns learned A, B, pi via EM.
    """
    T = len(obs)
    rng = np.random.default_rng(0)
    A = rng.dirichlet(np.ones(n_states), size=n_states)
    B = rng.dirichlet(np.ones(n_symbols), size=n_states)
    pi = rng.dirichlet(np.ones(n_states))

    for _ in range(n_iter):
        # Forward (alpha) and backward (beta) - log-space recommended in practice
        alpha = np.zeros((T, n_states))
        alpha[0] = pi * B[:, obs[0]]
        alpha[0] /= alpha[0].sum() + 1e-12
        for t in range(1, T):
            alpha[t] = (alpha[t-1] @ A) * B[:, obs[t]]
            alpha[t] /= alpha[t].sum() + 1e-12

        beta = np.zeros((T, n_states))
        beta[-1] = 1.0
        for t in range(T-2, -1, -1):
            beta[t] = (A * B[:, obs[t+1]]).dot(beta[t+1])
            beta[t] /= beta[t].sum() + 1e-12

        gamma = alpha * beta
        gamma /= gamma.sum(axis=1, keepdims=True) + 1e-12

        xi = np.zeros((T-1, n_states, n_states))
        for t in range(T-1):
            denom = (alpha[t][:, None] * A * (B[:, obs[t+1]] * beta[t+1])).sum() + 1e-12
            xi[t] = (alpha[t][:, None] * A * (B[:, obs[t+1]] * beta[t+1])) / denom

        # M-step
        pi = gamma[0]
        A = xi.sum(axis=0) / (gamma[:-1].sum(axis=0)[:, None] + 1e-12)
        for j in range(n_states):
            mask = np.zeros(T)
            for t, sym in enumerate(obs):
                if sym == 0:  # example: accumulate per symbol
                    pass
            B[j] = np.bincount(obs, weights=gamma[:, j], minlength=n_symbols)
            B[j] /= B[j].sum() + 1e-12

    return pi, A, B
```

Production systems use **log-space** forward-backward for numerical stability ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)).

---

## Application 2: Probabilistic Clustering

**Gaussian mixture EM** ([Section 20.4](./section-04-em-algorithm.md)) is clustering with:

| Feature | Benefit |
|---------|---------|
| Soft assignments $\gamma_{ik}$ | Uncertainty per point |
| Elliptical clusters | $\boldsymbol{\Sigma}_k$ captures correlation |
| Generative density | Score new points, detect anomalies |

vs. [k-means](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md):

| k-means | GMM + EM |
|---------|----------|
| Minimize within-cluster variance | Maximize likelihood |
| Hard 0/1 membership | Probabilistic membership |
| Single scale (spherical) | Flexible covariances |

**Customer segmentation example:** responsibilities show borderline customers split between segments - actionable for marketing uncertainty.

```python
# After EM: assign cluster by max responsibility
labels = resp.argmax(axis=1)
probs = resp.max(axis=1)  # confidence of assignment
```

---

## Application 3: Missing Data

Dataset with **incomplete observations** - some features missing per row. Model: joint $P(\mathbf{x}_{\text{obs}}, \mathbf{x}_{\text{miss}} \mid \theta)$.

**Likelihood:**

$$
P(\mathbf{x}_{\text{obs}} \mid \theta) = \sum_{\mathbf{x}_{\text{miss}}} P(\mathbf{x}_{\text{obs}}, \mathbf{x}_{\text{miss}} \mid \theta)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

(or integral for continuous missing values)

EM treats $\mathbf{x}_{\text{miss}}$ as latent:

| Step | Action |
|------|--------|
| **E** | Impute distribution $P(\mathbf{x}_{\text{miss}} \mid \mathbf{x}_{\text{obs}}, \theta^{(t)})$ |
| **M** | Re-estimate $\theta$ as if completed data were observed (weighted) |

---

## Missing Data: Multivariate Gaussian

If $\mathbf{x} = (\mathbf{x}_a, \mathbf{x}_b)$ and $\mathbf{x}_b$ missing, under $\mathcal{N}(\boldsymbol{\mu}, \boldsymbol{\Sigma})$ the conditional $\mathbf{x}_b \mid \mathbf{x}_a$ is Gaussian - E-step uses conditional expectations.

**M-step** updates $\boldsymbol{\mu}, \boldsymbol{\Sigma}$ with expected complete sufficient statistics across all patterns of missingness.

| Approach | Method |
|----------|--------|
| **Listwise deletion** | Drop incomplete rows - wasteful |
| **Mean imputation** | Single fill-in - underestimates variance |
| **EM** | Principled, preserves uncertainty |

---

## Missing Data in Bayesian Networks

For BN parameter learning with incomplete cases ([Section 20.6](./section-06-bayesian-network-parameter-learning.md)):

**E-step:** infer posterior over missing variables given observed entries and current CPTs - often via [Gibbs sampling](../chapter-13-probabilistic-reasoning/README.md) or message passing.

**M-step:** update CPT counts with expected counts.

General **structured EM** alternates inference and parameter updates - backbone of learning in graphical models.

---

## Comparison Table: EM Applications

| Domain | Latent $\mathbf{z}$ | E-step tool | M-step |
|--------|---------------------|-------------|--------|
| **GMM** | Component labels | Responsibilities | Weighted means/covariances |
| **HMM** | State sequence | Forward-backward | Update $A, B, \pi$ |
| **Missing data** | Missing features | Conditional expectations | Standard parametric MLE |
| **PPCA** | Latent factors | Posterior $p(z \mid x)$ | Factor loadings ([Course 3](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-13-linear-factor-models/README.md)) |

---

## Speech Recognition Connection

Classic pipeline: discrete HMM per phoneme, Baum-Welch on transcribed speech. Modern end-to-end neural ASR ([Course 3 sequence modeling](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md)) replaces explicit HMMs but **latent alignment** ideas persist (CTC, attention).

---

## Clustering Evaluation with EM

| Metric | Purpose |
|--------|---------|
| **Log-likelihood** on test set | Generative fit |
| **BIC** | Penalize $K$ |
| **Adjusted Rand index** | Compare to ground-truth labels (if available) |

Soft assignments enable **entropy** per point - high entropy = ambiguous cluster membership.

---

## Limitations Across Applications

| Limitation | Consequence |
|------------|-------------|
| Local optima | Different inits → different solutions |
| Model misspecification | EM fits wrong model confidently |
| Large $T$ HMMs | Forward-backward $O(T \cdot |S|^2)$ |
| Not online | Batch algorithm; streaming variants exist |

**Variational EM** and **stochastic EM** scale to large data and deep models.

---

## Connection to Course 1

| Course 1 | EM upgrade |
|----------|------------|
| k-means | GMM with soft assignments |
| Mean imputation | EM for Gaussian missing data |
| Hierarchical clustering | Overlapping clusters via mixtures |

---

## Connection to Course 3

| Course 3 | EM connection |
|----------|---------------|
| [Sequence modeling](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) | Neural HMM / RNN training parallels |
| [Linear factor models](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-13-linear-factor-models/README.md) | EM for PPCA |
| [VAE](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-20-deep-generative-models/README.md) | Amortized E-step via encoder |

---

## Worked Mini-Example: HMM Intuition

Two hidden states (Rain/Sun), observe umbrella/no-umbrella:

| $t$ | 1 | 2 | 3 | 4 |
|-----|---|---|---|---|
| Obs | U | U | N | U |

After Baum-Welch on many sequences, $A$ captures weather persistence, $B$ links Rain → umbrella. $\gamma_t(\text{Rain})$ high when umbrella observed - soft inference.

---

## Reflection Questions

1. Why is Baum-Welch considered an EM algorithm? Identify $\mathbf{z}$, E-step, and M-step.
2. How does probabilistic clustering differ from k-means on overlapping groups?
3. Why is listwise deletion biased when data are not missing completely at random?
4. What is the computational complexity of one Baum-Welch iteration?
5. When would variational methods replace exact E-steps?

---

**Next:** [Section 20.6 - Bayesian Network Parameter Learning](./section-06-bayesian-network-parameter-learning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Sections 20.4-20.5, 14.4. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Rabiner, L. R. (1989). A tutorial on hidden Markov models. *Proceedings of the IEEE*. [IEEE Xplore](https://ieeexplore.ieee.org/)
3. Little, R. J. A., & Rubin, D. B. (2019). *Statistical Analysis with Missing Data* (3rd ed.). [Wiley](https://www.wiley.com/)
4. Bishop, C. M. (2006). *Pattern Recognition and Machine Learning*, Sections 9.3-9.4. [Microsoft Research](https://www.microsoft.com/en-us/research/people/cmbishop/)
5. Murphy, K. P. (2022). *Probabilistic Machine Learning*, Chapters 8, 19. [probml.ai](https://probml.ai/)
