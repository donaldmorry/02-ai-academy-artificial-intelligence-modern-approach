# Section 20.3: Bayesian Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 20.3  
> **Previous:** [Section 20.2 - Maximum Likelihood](./section-02-maximum-likelihood.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Beyond Point Estimates

[Maximum likelihood](./section-02-maximum-likelihood.md) returns a single $\hat{\theta}$. **Bayesian learning** treats $\theta$ itself as a random variable with a **prior** $P(\theta)$ updated by data to a **posterior** $P(\theta \mid \mathcal{D})$.

$$
P(\theta \mid \mathcal{D}) = \frac{P(\mathcal{D} \mid \theta)\, P(\theta)}{P(\mathcal{D})}
$$
> **Readable form:** Posterior ∝ likelihood × prior - beliefs after data combine what we thought before with how well each parameter explains the evidence.

Humorous analogy: MLE picks one "best guess" hero; Bayesian learning keeps the **whole cast** with updated billing - some parameters remain plausible, others fade, and uncertainty is explicit.

---

## The Bayesian Learning Pipeline

```
Prior P(θ)  +  Data D  →  Posterior P(θ|D)  →  Predictions P(x_new|D)
```

| Stage | Question answered |
|-------|-------------------|
| **Prior** | What do we believe before data? |
| **Likelihood** | How probable is data under each $\theta$? |
| **Posterior** | What do we believe after data? |
| **Posterior predictive** | What future data do we expect? |

**Posterior predictive distribution:**

$$
P(\mathbf{x}_{\text{new}} \mid \mathcal{D}) = \int P(\mathbf{x}_{\text{new}} \mid \theta)\, P(\theta \mid \mathcal{D})\, d\theta
$$
> **Readable form:** Predict new data by averaging over all parameter values, weighted by posterior belief.

This integrates **model uncertainty** - critical when $N$ is small or the model is fragile.

---

## Maximum A Posteriori (MAP)

Full Bayesian integration can be expensive. **MAP** finds the posterior mode:

$$
\hat{\theta}_{\text{MAP}} = \arg\max_{\theta} P(\theta \mid \mathcal{D})
= \arg\max_{\theta} \left[ \log P(\mathcal{D} \mid \theta) + \log P(\theta) \right]
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

| Method | Output | Uncertainty |
|--------|--------|-------------|
| **MLE** | $\arg\max \log P(\mathcal{D} \mid \theta)$ | None |
| **MAP** | Posterior mode | Point only |
| **Full Bayesian** | Full $P(\theta \mid \mathcal{D})$ | Captured in distribution |

MAP is the bridge to [regularization](../../GLOSSARY.md#regularization) in [Course 3](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-07-regularization/README.md): log-prior acts as penalty.

---

## Conjugate Priors

A prior is **conjugate** to a likelihood if the posterior has the **same functional form** as the prior - closed-form updates without integration.

| Likelihood | Conjugate prior | Posterior |
|------------|-----------------|-----------|
| Bernoulli / Binomial | $\text{Beta}(\alpha, \beta)$ | $\text{Beta}(\alpha + n_H, \beta + n_T)$ |
| Multinomial | $\text{Dirichlet}(\boldsymbol{\alpha})$ | $\text{Dirichlet}(\boldsymbol{\alpha} + \mathbf{n})$ |
| Gaussian (known $\sigma^2$) | Gaussian on $\mu$ | Gaussian |
| Gaussian (unknown $\mu, \sigma^2$) | Normal-Inverse-Gamma | Same family |

Conjugacy is mathematically convenient; with enough data, likelihood dominates prior.

---

## Worked Example: Beta-Bernoulli

Prior: $\theta \sim \text{Beta}(\alpha, \beta)$ - probability of heads.

$$
P(\theta) \propto \theta^{\alpha-1}(1-\theta)^{\beta-1}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Observe $n_H$ heads, $n_T$ tails. Posterior:

$$
P(\theta \mid \mathcal{D}) = \text{Beta}(\alpha + n_H,\; \beta + n_T)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Posterior mean:

$$
\mathbb{E}[\theta \mid \mathcal{D}] = \frac{\alpha + n_H}{\alpha + \beta + n_H + n_T}
$$
> **Readable form:** Posterior mean = (prior pseudo-heads + observed heads) / (total pseudo + observed trials).

| Prior | $n_H=7, n_T=3$ | Posterior mean |
|-------|----------------|----------------|
| $\text{Beta}(1,1)$ uniform | - | 0.70 (≈ MLE) |
| $\text{Beta}(2,2)$ | - | 0.64 |
| $\text{Beta}(10,10)$ strong | - | 0.56 |

```python
import numpy as np
from scipy import stats

def beta_bernoulli_posterior(alpha, beta, n_heads, n_tails):
    post = stats.beta(alpha + n_heads, beta + n_tails)
    return {
        "mean": post.mean(),
        "mode": (post.mean() * (post.mean() - 1))  # use .mode() in scipy 1.11+
        if False else (alpha + n_heads - 1) / (alpha + beta + n_heads + n_tails - 2)
        if alpha + n_heads > 1 and beta + n_tails > 1 else post.mean(),
        "credible_95": post.interval(0.95),
    }

print(beta_bernoulli_posterior(2, 2, 7, 3))
```

---

## Dirichlet-Multinomial (Preview for BNs)

For $k$ categories with counts $n_1, \ldots, n_k$:

$$
P(\mathbf{p} \mid \mathcal{D}) = \text{Dirichlet}(\alpha_1 + n_1, \ldots, \alpha_k + n_k)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**Pseudo-counts:** $\alpha_j$ act as imaginary observations of category $j$ before real data arrive. [Section 20.6](./section-06-bayesian-network-parameter-learning.md) uses this for CPT learning in [Bayesian networks](../chapter-13-probabilistic-reasoning/README.md).

$$
\mathbb{E}[p_j \mid \mathcal{D}] = \frac{\alpha_j + n_j}{\sum_\ell (\alpha_\ell + n_\ell)}
$$
> **Readable form:** Expected probability of category $j$ = (pseudo-count + real count) / (total pseudo + real counts).

---

## Bayesian Gaussian with Known Variance

Data $x^{(1)}, \ldots, x^{(N)}$, known $\sigma^2$, prior $\mu \sim \mathcal{N}(\mu_0, \sigma_0^2)$.

Posterior for $\mu$ is Gaussian $\mathcal{N}(\mu_N, \sigma_N^2)$ with:

$$
\mu_N = \frac{\sigma^2 \mu_0 + N \bar{x}\, \sigma_0^2}{\sigma^2 + N \sigma_0^2}, \quad
\frac{1}{\sigma_N^2} = \frac{1}{\sigma_0^2} + \frac{N}{\sigma^2}
$$
> **Readable form:** Posterior mean is a precision-weighted average of prior mean and sample mean; posterior precision = prior precision + data precision.

As $N \to \infty$, $\mu_N \to \bar{x}$ - data overwhelm prior.

---

## Model Comparison: Bayesian Evidence

To compare models $M_1, M_2$:

$$
P(M_i \mid \mathcal{D}) \propto P(\mathcal{D} \mid M_i)\, P(M_i)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**Marginal likelihood** (evidence):

$$
P(\mathcal{D} \mid M) = \int P(\mathcal{D} \mid \theta, M)\, P(\theta \mid M)\, d\theta
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

| Property | Implication |
|----------|-------------|
| Automatic Occam's razor | Simpler models favored if fit is similar |
| Sensitive to priors | Requires careful prior design |
| Often intractable | Approximations: BIC, variational, MCMC |

BIC from [Section 20.1](./section-01-statistical-learning.md) approximates $-2\log P(\mathcal{D} \mid M)$ for large $N$.

---

## Predictive vs. Fitted

| Distribution | Use |
|--------------|-----|
| $P(\theta \mid \mathcal{D})$ | Parameter uncertainty |
| $P(\mathbf{x}_{\text{new}} \mid \mathcal{D})$ | Forecasting with uncertainty |
| $P(\mathbf{x}^{(i)} \mid \hat{\theta}_{\text{MLE}})$ | Plug-in (underestimates uncertainty) |

**Plug-in prediction** uses $\hat{\theta}_{\text{MLE}}$ only - fast but **overconfident** on small data. Bayesian predictive averages over $\theta$.

---

## Connection to Chapter 13: Belief Updating

[Chapter 13](../chapter-13-probabilistic-reasoning/README.md) performs **inference** in fixed BNs. Bayesian parameter learning is **belief updating** at the meta-level: CPT entries are uncertain quantities with Dirichlet posteriors.

Observed BN data + Dirichlet prior → updated CPT distributions → propagate in queries.

---

## Connection to Course 1

| Course 1 idea | Bayesian view |
|---------------|---------------|
| Laplace smoothing in Naive Bayes | MAP with Dirichlet/Beta prior |
| [Train/validation split](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning) | Approximate model comparison |
| [Overfitting](../../GLOSSARY.md#overfitting) | Prior limits extreme parameters |

---

## Connection to Course 3

| Course 3 topic | Bayesian link |
|----------------|---------------|
| [Dropout](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-07-regularization/README.md) | Approximate Bayesian ensemble (informal) |
| [Variational inference](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-19-approximate-inference/README.md) | Approximate $P(\theta \mid \mathcal{D})$ when exact is hard |
| [Deep generative models](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-20-deep-generative-models/README.md) | Amortized posterior $q(\mathbf{z} \mid \mathbf{x})$ |

Chapter 20's conjugate updates are the **exact** special case that variational methods generalize.

---

## When Full Bayesian Learning Is Hard

| Challenge | Remedy |
|-----------|--------|
| Non-conjugate models | MCMC, variational inference |
| High-dimensional $\theta$ | Structured approximations |
| Latent variables | EM (point estimate) or variational Bayes |
| Structure learning | Marginal likelihood + search ([Section 20.7](./section-07-structure-learning.md)) |

[Section 20.4](./section-04-em-algorithm.md) can be viewed as MAP/MLE with latent variables; **variational Bayes** extends EM to approximate posteriors.

---

## Credible Intervals vs. Confidence Intervals

| Interval type | Interpretation |
|---------------|----------------|
| **Bayesian credible** | 95% probability $\theta$ lies in interval (given data + prior) |
| **Frequentist confidence** | Procedure covers true $\theta$ 95% of repeated experiments |

For AI agents acting under uncertainty ([Chapter 16](../chapter-16-making-simple-decisions/README.md)), credible intervals align naturally with **subjective** belief states.

---

## Worked Mini-Example: Predicting the Next Flip

$\text{Beta}(2,2)$ prior, observe 7H, 3T. Posterior $\text{Beta}(9,5)$.

Probability next flip is heads (posterior predictive for Bernoulli):

$$
P(x_{N+1} = H \mid \mathcal{D}) = \mathbb{E}[\theta \mid \mathcal{D}] = \frac{9}{14} \approx 0.64
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Compare MLE plug-in: $0.70$. Bayesian prediction is **more conservative** toward the prior mean $0.5$.

---

## Common Pitfalls

**Confusing prior with regularization hyperparameter.** Priors encode belief; $\lambda$ in L2 is often tuned by validation - philosophically different though mathematically similar to Gaussian prior.

**Improper priors** (e.g., uniform on $\mathbb{R}$) - may yield non-normalizable posteriors; use with care.

**Double-counting data** - using test set to set prior strength then evaluating on same test set.

**Ignoring computational cost** - full posteriors over millions of neural weights need approximations.

---

## Reflection Questions

1. How does the Beta posterior change with prior $\text{Beta}(1,1)$ vs. $\text{Beta}(50,50)$ after 10 flips?
2. When does MAP differ substantially from MLE?
3. Why does Bayesian model evidence penalize overly complex models?
4. What is the difference between posterior over $\theta$ and posterior predictive over new data?
5. How do Dirichlet pseudo-counts prevent zero probabilities in BN CPTs?

---

**Next:** [Section 20.4 - EM Algorithm](./section-04-em-algorithm.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 20.3. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Gelman, A., et al. (2013). *Bayesian Data Analysis* (3rd ed.), Chapters 1-2. [CRC Press](https://sites.stat.columbia.edu/gelman/book/)
3. Bishop, C. M. (2006). *Pattern Recognition and Machine Learning*, Section 2.3 - Bayesian methods. [Microsoft Research](https://www.microsoft.com/en-us/research/people/cmbishop/)
4. Murphy, K. P. (2022). *Probabilistic Machine Learning*, Chapter 4 - conjugate models. [probml.ai](https://probml.ai/)
5. Jaynes, E. T. (2003). *Probability Theory: The Logic of Science*. [Cambridge University Press](https://www.cambridge.org/)
