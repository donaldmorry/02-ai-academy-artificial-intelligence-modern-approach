# Section 20.6: Bayesian Network Parameter Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 20.5  
> **Previous:** [Section 20.5 - EM Applications](./section-05-em-applications.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Learning CPTs from Data

[Chapter 13](../chapter-13-probabilistic-reasoning/README.md) specified **Bayesian networks** (BNs) with conditional probability tables (CPTs). Given a **fixed structure** (DAG), parameter learning estimates each local conditional $P(X_i \mid \text{Parents}(X_i))$ from data.

[Section 20.7](./section-07-structure-learning.md) addresses **which edges** exist; this section covers **numeric CPT entries** with complete and incomplete data.

> **Readable form:** Parameter learning = fill in every cell of every CPT so the BN best matches observed cases - counting when data are complete, EM when some variables are missing.

Humorous analogy: structure learning picks the **family tree**; parameter learning fills in **how each relative behaves** given their parents.

---

## Factorization and Decomposition

BN joint distribution:

$$
P(X_1, \ldots, X_n) = \prod_{i=1}^{n} P(X_i \mid \text{Pa}(X_i))
$$
> **Readable form:** The joint probability is built by multiplying the conditional factors specified by the model.

For i.i.d. cases $\mathbf{x}^{(1)}, \ldots, \mathbf{x}^{(N)}$:

P(\mathcal{D} \mid \theta) = \prod_{m=1}^{N} \prod_{i=1}^{n} P(x_i^{(m)} \mid \text{Pa}(x_i^{(m)}), \theta_i)
$$

**Key property:** parameters $\theta_i$ for node $X_i$ are **separable** - learning each CPT is independent given structure.

$$
> **Readable form:** Log-likelihood splits into a sum of terms, one per BN node - learn each node's table separately.

---

## Complete Data: Maximum Likelihood

For discrete $X_i$ with parents $\text{Pa}(X_i)$, let $\theta_{ijk} = P(X_i = k \mid \text{Pa}(X_i) = j)$.

**Count** $N_{ijk}$ = number of cases where $X_i=k$ and parent config $=j$.

$$
\hat{\theta}_{ijk}^{\text{MLE}} = \frac{N_{ijk}}{N_{ij\cdot}} = \frac{N_{ijk}}{\sum_k N_{ijk}}
$$
> **Readable form:** MLE CPT entry = fraction of rows with parent config $j$ where child takes value $k$.

| Parent config $j$ | Child $k$ | Count | MLE |
|-------------------|-----------|-------|-----|
| Alarm=T | Burglary=T | 12 | 12/50 |
| Alarm=T | Burglary=F | 38 | 38/50 |

Same as [multinomial MLE](./section-02-maximum-likelihood.md) per parent context.

---

## Complete Data: Python Example

```python
import pandas as pd
import numpy as np

def learn_cpt_mle(data, child, parents):
    """
    data: DataFrame with discrete columns
    Returns dict: parent_assignment -> {child_val: prob}
    """
    if not parents:
        counts = data[child].value_counts()
        total = counts.sum()
        return {(): {k: counts[k] / total for k in counts.index}}

    grouped = data.groupby(parents + [child]).size().reset_index(name='count')
    cpt = {}
    for parent_vals, sub in grouped.groupby(parents):
        if not isinstance(parent_vals, tuple):
            parent_vals = (parent_vals,)
        total = sub['count'].sum()
        cpt[parent_vals] = {
            row[child]: row['count'] / total
            for _, row in sub.iterrows()
        }
    return cpt

# Toy alarm-network style data
df = pd.DataFrame({
    'Burglary': ['T','F','F','T','F'] * 20,
    'Earthquake': ['F','F','T','F','F'] * 20,
    'Alarm': ['T','F','T','T','F'] * 20,
})
print(learn_cpt_mle(df, 'Alarm', ['Burglary', 'Earthquake']))
```

---

## Zero Counts and Overfitting

If $N_{ijk} = 0$, MLE sets $\hat{\theta}_{ijk} = 0$ - **zero probability** to unseen combinations → inference disasters.

| Problem | Example failure |
|---------|-----------------|
| $\hat{\theta} = 0$ | Any query involving unseen combo gets probability 0 |
| Sparse parents | Exponential parent configurations |

**Fix:** [Bayesian learning](./section-03-bayesian-learning.md) with Dirichlet priors.

---

## Dirichlet Priors and Pseudo-Counts

For each CPT row (fixed parent config $j$), place Dirichlet prior over child outcomes:

$$
P(\boldsymbol{\theta}_{ij\cdot}) = \text{Dirichlet}(\alpha_{ij1}, \ldots, \alpha_{ij d_i})
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Posterior after counts $N_{ijk}$:

$$
P(\boldsymbol{\theta}_{ij\cdot} \mid \mathcal{D}) = \text{Dirichlet}(\alpha_{ij1} + N_{ij1}, \ldots, \alpha_{ij d_i} + N_{ij d_i})
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**MAP / posterior mean:**

$$
\hat{\theta}_{ijk} = \frac{\alpha_{ijk} + N_{ijk}}{\sum_{k'} (\alpha_{ijk'} + N_{ijk'})}
$$
> **Readable form:** Smoothed probability = (pseudo-count + real count) / (sum of pseudo + real counts for that parent row).

**Laplace smoothing:** $\alpha_{ijk} = 1$ for all $k$ - uniform pseudo-count.

---

## Equivalent Sample Size

Prior strength $s = \sum_k \alpha_{ijk}$ acts like **imaginary prior observations**.

| Prior | Effect |
|-------|--------|
| $\alpha = 1$ (Laplace) | Mild smoothing toward uniform |
| $\alpha < 1$ | Flatter prior, less smoothing |
| $\alpha \gg N_{ij\cdot}$ | Prior dominates - underfits data |

Tie to [Naive Bayes](../../GLOSSARY.md#naive-bayes) smoothing in Course 1.

---

## Continuous Variables

When $X_i$ or parents are continuous, CPTs become **parametric conditionals**:

| Form | Parameters learned |
|------|-------------------|
| $P(X_i \mid \text{Pa}) = \mathcal{N}(\mu_{ij}, \sigma_{ij}^2)$ | Mean, variance per parent config |
| Linear Gaussian | Regression coefficients |
| CPD with neural net | Weights ([Course 3](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/README.md)) |

MLE: fit regression of $X_i$ on $\text{Pa}(X_i)$ for each configuration or shared parameters.

---

## Incomplete Data: EM for BNs

When some variables are unobserved in some cases, local counting fails. **EM:**

| Step | Action |
|------|--------|
| **E** | Compute expected counts $\mathbb{E}[N_{ijk}]$ w.r.t. posterior over missing values given observed |
| **M** | Update $\theta_{ijk}$ from expected counts (with Dirichlet if Bayesian) |

E-step requires **BN inference** - variable elimination, junction tree, or Gibbs ([Chapter 13](../chapter-13-probabilistic-reasoning/README.md)).

[Section 20.5](./section-05-em-applications.md) framed missing-data EM generally; BNs add **graph structure** to factor inference.

---

## Bayesian Parameter Learning (Full)

Rather than point estimates, maintain **posterior over each CPT row** (Dirichlet) or over Gaussian parameters (Normal-Inverse-Wishart).

**Posterior predictive** for new case:

$$
P(x_i \mid \mathbf{x}_{\text{obs}}, \mathcal{D}) = \int P(x_i \mid \text{Pa}, \theta_{ij}) P(\theta_{ij} \mid \mathcal{D})\, d\theta_{ij}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

For Dirichlet, this is **Dirichlet-multinomial** - closed form.

---

## Table: Complete vs. Incomplete Data

| Setting | Method | Complexity |
|---------|--------|------------|
| Complete, discrete | Count / MLE | $O(N \cdot n)$ |
| Complete + Dirichlet | Smoothed counts | Same |
| Incomplete | EM + inference | Per-iteration inference cost |
| Fully Bayesian | MCMC / variational | Higher |

---

## Example: Alarm Network Parameter

Structure (simplified): Burglary → Alarm ← Earthquake, Alarm → JohnCalls.

Learning $P(\text{Alarm} \mid \text{Burglary}, \text{Earthquake})$:

| B | E | P(Alarm=T) MLE | + Laplace |
|---|---|----------------|-----------|
| T | T | 0.95 | (0.95·N+1)/(N+2) |
| T | F | 0.94 | smoothed |
| F | T | 0.29 | smoothed |
| F | F | 0.001 | smoothed |

Rare row B=F,E=F, Alarm=T needs smoothing most.

---

## Tying Parameters

**Parameter tying** shares $\theta$ across nodes or contexts - reduces parameters, improves generalization when assumptions hold.

Example: all time slices in a **dynamic BN** ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)) share transition matrices.

| Tied | Untied |
|------|--------|
| Fewer parameters | More flexible |
| Better with sparse data | Risk overfitting |

---

## Connection to Chapter 13

| Chapter 13 | Chapter 20 |
|-----------|-----------|
| Given CPTs, compute queries | Learn CPTs from data |
| Inference algorithms | E-step of EM |
| Alarm network example | Parameter estimation for same network |

Full pipeline: **learn parameters** → **infer** → **decide** ([Chapter 16](../chapter-16-making-simple-decisions/README.md)).

---

## Connection to Course 1

| Course 1 | BN parameter learning |
|----------|----------------------|
| Frequency tables | CPT MLE |
| Laplace smoothing | Dirichlet prior |
| [Supervised learning](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-03-supervised-vs-unsupervised-learning.md) | Local conditional models |

---

## Connection to Course 3

[Structured probabilistic models](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-16-structured-probabilistic-models/README.md) scale BNs to high dimensions - parameter learning uses gradients rather than pure counting.

---

## Common Pitfalls

**Learning from interventional data with observational CPT formulas** - needs causal adjustment ([Chapter 16](../chapter-16-making-simple-decisions/README.md)).

**Unnormalized counts** - must normalize each CPT row to sum to 1.

**Forgetting to align parent value orderings** across rows.

**Applying complete-data MLE when 30% of entries are missing** - biased without EM.

---

## Reflection Questions

1. Why does BN log-likelihood decompose into a sum over nodes?
2. What happens to MLE when a parent-child combination never appears in data?
3. How do Dirichlet pseudo-counts relate to Laplace smoothing?
4. What inference algorithm would you use in the E-step for a polytree BN?
5. Why is parameter tying natural in dynamic Bayesian networks?

---

**Next:** [Section 20.7 - Structure Learning](./section-07-structure-learning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 20.5. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Heckerman, D. (1998). A tutorial on learning with Bayesian networks. *Learning in Graphical Models*. [Microsoft Research](https://www.microsoft.com/en-us/research/people/heckerman/)
3. Koller, D., & Friedman, N. (2009). *Probabilistic Graphical Models*, Chapter 16. [MIT Press](https://mitpress.mit.edu/)
4. Murphy, K. P. (2022). *Probabilistic Machine Learning*, Chapter 26 - BN parameter learning. [probml.ai](https://probml.ai/)
5. AIMA Python - learning Bayes nets. [aima-python](https://github.com/aimacode/aima-python)


