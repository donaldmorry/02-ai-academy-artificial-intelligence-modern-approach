# Section 20.8: Nonparametric Bayes Preview

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 20.7 (overview)  
> **Previous:** [Section 20.7 - Structure Learning](./section-07-structure-learning.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## When Fixed $K$ Is Not Enough

[Section 20.4](./section-04-em-algorithm.md) fit Gaussian mixtures with **fixed** component count $K$. Choosing $K$ requires BIC, cross-validation, or domain guesswork. **Nonparametric Bayesian** models let the data **determine complexity** - effectively unbounded components with most unused.

> **Readable form:** Nonparametric Bayes = "start with infinitely many hypotheses, let data decide how many matter" - the model grows with evidence.

Humorous analogy: fixed-$K$ EM is a **talent show with exactly four slots**; nonparametric Bayes is **open mic night** where strong performers keep the mic and weak acts fade out.

---

## Parametric vs. Nonparametric (Revisited)

[Section 20.1](./section-01-statistical-learning.md) contrasted:

| Type | Parameters | Example |
|------|------------|---------|
| **Parametric** | Fixed dim. of $\theta$ | 3-component GMM |
| **Nonparametric** | Grows with $N$ | Dirichlet process mixture |

"Nonparametric" does **not** mean "no parameters" - it means **infinitely many** parameters in the limiting representation, with only finitely many active for finite data.

---

## The Dirichlet Process (DP)

A **random probability measure** $G \sim \text{DP}(\alpha, G_0)$:

- **Concentration** $\alpha$ controls how closely $G$ follows base measure $G_0$.
- **Base measure** $G_0$ - e.g., $\mathcal{N}(\mu_0, \sigma_0^2)$ for 1D data.

**Stick-breaking construction** (Griffiths, Engen, McCloskey; Sethuraman):

$$
G = \sum_{k=1}^{\infty} \pi_k \, \delta_{\theta_k^*}, \quad
\pi_k = v_k \prod_{\ell=1}^{k-1}(1 - v_\ell), \quad v_k \sim \text{Beta}(1, \alpha)
$$
> **Readable form:** Build a discrete distribution from infinitely many atoms - each new stick piece $v_k$ takes a slice of remaining probability mass.

Only countably many $\pi_k$ are positive; rest are negligible.

---

## Chinese Restaurant Process (CRP) Intuition

Equivalent sequential story for clustering:

1. Customer 1 sits at table 1.
2. Customer $n$ joins existing table $k$ with probability $\propto$ occupants at $k$, or opens new table with probability $\propto \alpha$.

| Parameter | Effect |
|-----------|--------|
| Large $\alpha$ | More new tables → more clusters |
| Small $\alpha$ | Fewer, larger clusters |

**Exchangeability:** order of customers does not matter - consistent with Bayesian i.i.d. models.

---

## Dirichlet Process Mixture Model (DPMM)

Each data point $\mathbf{x}^{(i)}$:

$$
z^{(i)} \sim G, \quad \mathbf{x}^{(i)} \sim F(\cdot \mid z^{(i)})
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Example: $z^{(i)} = \boldsymbol{\mu}_{k}$ cluster mean, $\mathbf{x}^{(i)} \sim \mathcal{N}(\boldsymbol{\mu}_k, \sigma^2 I)$.

Cluster assignment follows CRP - **no fixed $K$**. With more data, new clusters can appear.

Compare [k-means](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md) (fixed $k$) and [EM GMM](./section-04-em-algorithm.md) (fixed $K$).

---

## Inference: Beyond Standard EM

Exact inference over infinite mixtures is impossible. Standard approaches:

| Method | Idea |
|--------|------|
| **Collapsed Gibbs sampling** | Integrate out parameters, sample cluster assignments |
| **Truncated variational** | Approximate with max $K_{\max}$ components |
| **Split-merge MCMC** | Efficiently explore cluster partitions |

```python
# Simplified CRP clustering sketch (1D, known variance)
import numpy as np

def crp_cluster_gibbs(x, alpha=1.0, sigma2=1.0, n_iter=500):
    """Collapsed-ish Gibbs: assign points to clusters; new clusters from prior."""
    N = len(x)
    z = np.zeros(N, dtype=int)  # cluster labels
    K = 1
    for _ in range(n_iter):
        for i in range(N):
            counts = np.bincount(z, minlength=K)
            # Likelihood of x[i] under each existing cluster mean
            means = [x[z == k].mean() for k in range(K)]
            log_probs = []
            for k in range(K):
                nk = counts[k] - (1 if z[i] == k else 0)
                log_probs.append(np.log(nk + 1e-12) - 0.5 * (x[i] - means[k])**2 / sigma2)
            log_probs.append(np.log(alpha) - 0.5 * (x[i] - x.mean())**2 / sigma2)  # new
            probs = np.exp(np.array(log_probs) - np.max(log_probs))
            probs /= probs.sum()
            choice = np.random.choice(len(probs), p=probs)
            if choice == K:
                z[i] = K
                K += 1
            else:
                z[i] = choice
    return z
```

Production DPMM uses conjugate base measures for analytic marginalization.

---

## Hierarchical Dirichlet Process (HDP) - Preview

**Topic models** (e.g., LDA) use hierarchical DPs: **infinite topics** per document collection. Each document mixes a subset of topics; corpus shares global topic measure.

| Level | Role |
|-------|------|
| Corpus | Global topic distribution |
| Document | Subset mixture of topics |
| Word | Drawn from topic |

Connects to NLP and [Course 3 representation learning](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-15-representation-learning/README.md) - neural topic models as modern successors.

---

## Advantages and Trade-offs

| Advantage | Trade-off |
|-----------|-----------|
| Automatic complexity control | Heavier computation |
| Coherent uncertainty on $K$ | Harder to implement than EM |
| Principled Bayesian Occam's razor | Sensitive to $\alpha$, $G_0$ |
| Flexible clustering | Interpretability of many small clusters |

---

## Relation to Chapter 20 Themes

| Earlier section | Nonparametric extension |
|----------------|-------------------------|
| [MLE](./section-02-maximum-likelihood.md) | Marginal likelihood integrates $\theta$ |
| [Bayesian learning](./section-03-bayesian-learning.md) | Priors on infinite mixtures |
| [EM](./section-04-em-algorithm.md) | Gibbs / variational replace E-step |
| [Structure learning](./section-07-structure-learning.md) | Nonparametric BN structure priors (advanced) |

---

## Infinite Hidden Markov Models (iHMM) - Brief

HDP-HMM: unbounded number of **states** in temporal models ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)). Data determine how many distinct regimes appear - useful when state count is unknown (regime switching in finance, behavior modes in robotics).

---

## Gaussian Process Preview

Another nonparametric family: **Gaussian processes** prior over functions $f(x)$ - any finite set of inputs has joint Gaussian. Dominant in Bayesian optimization and spatial statistics.

| Object | Nonparametric prior |
|--------|---------------------|
| Clustering | Dirichlet process |
| Functions | Gaussian process |
| Topic models | HDP |

Full treatment exceeds AIMA Chapter 20 scope - flagged for advanced reading.

---

## Connection to Course 1

| Course 1 | Nonparametric Bayes |
|----------|---------------------|
| Choosing $k$ in k-means | $\alpha$ governs cluster birth rate |
| [Model selection](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning) | Integrated complexity control |
| Bootstrap stability | Posterior over partitions |

---

## Connection to Course 3

| Course 3 | Link |
|----------|------|
| [Deep generative models](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-20-deep-generative-models/README.md) | Neural samplers vs. DP mixtures |
| [VAE](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-20-deep-generative-models/README.md) | Continuous latent - different but related uncertainty |
| [Approximate inference](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-19-approximate-inference/README.md) | Variational truncations of DPMM |

Modern **Bayesian deep learning** applies nonparametric ideas to widths, depths, and functional priors - active research frontier.

---

## Choosing Hyperparameters

| Hyperparameter | Role | Tuning |
|----------------|------|--------|
| $\alpha$ | Cluster/table creation rate | Domain knowledge, cross-validation |
| $G_0$ | Component param prior | Conjugate choice for tractability |
| $\sigma^2$ in GMM-DP | Within-cluster spread | Empirical Bayes |

Too large $\alpha$ → many tiny clusters (overfitting); too small → under-partitioning.

---

## Worked Intuition: Growing $N$

| $N$ | Typical active clusters (moderate $\alpha$) |
|-----|---------------------------------------------|
| 20 | 2-4 |
| 200 | 5-12 |
| 2000 | Grows sublinearly - not 2000 clusters |

DP **preferential attachment** reuses existing clusters - natural Occam pressure.

---

## When to Use Nonparametric Bayes

| Use when | Prefer fixed parametric when |
|----------|------------------------------|
| $K$ genuinely unknown | $K$ known (e.g., 3 credit ratings) |
| Rich heterogeneity expected | Real-time, low-latency deployment |
| Full Bayesian analysis desired | Simple baseline needed fast |
| Exploratory clustering | Production EM GMM suffices |

---

## Chapter 20 Synthesis

| Section | Core idea |
|--------|-----------|
| 20.1 | Statistical learning framework |
| 20.2 | MLE |
| 20.3 | Bayesian / MAP / conjugacy |
| 20.4 | EM for latent variables |
| 20.5 | HMM, clustering, missing data |
| 20.6 | BN parameter learning |
| 20.7 | BN structure learning |
| 20.8 | Nonparametric Bayes preview |

**Next chapter:** [Chapter 21 - Deep Learning](../chapter-21-deep-learning/README.md) bridges probabilistic ideas to neural representation learning.

---

## Reflection Questions

1. How does the Chinese restaurant process allocate new data points to clusters?
2. What role does concentration parameter $\alpha$ play?
3. Why is standard EM insufficient for Dirichlet process mixtures?
4. Compare DPMM clustering to fixing $K=3$ in a GMM.
5. What is the relationship between stick-breaking and CRP representations?

---

**Next:** [Chapter 21 - Deep Learning](../chapter-21-deep-learning/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 20.7. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Teh, Y. W. (2010). Dirichlet processes. *Encyclopedia of Machine Learning*. [Springer](https://link.springer.com/)
3. Neal, R. M. (2000). Markov chain sampling methods for Dirichlet process mixture models. *J. Computational and Graphical Statistics*. [JSTOR](https://www.jstor.org/)
4. Hjort, N. L., et al. (2010). *Bayesian Nonparametrics*. [Cambridge University Press](https://www.cambridge.org/)
5. Murphy, K. P. (2022). *Probabilistic Machine Learning: Advanced Topics*, Chapter 32 - Dirichlet processes. [probml.ai](https://probml.ai/)
