# Section A.6: Expectation and Variance

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Appendix A.6  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Summarizing Random Variables

**Expectation** (mean) and **variance** compress distribution information into scalars - used in loss functions, risk assessment, and convergence analysis throughout AI.

> **Readable form:** Expectation is the long-run average; variance measures how spread out outcomes are around that average.

---

## Expectation

**Discrete:**

$$
\mathbb{E}[X] = \sum_x x \, P(X=x)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**Continuous:**

$$
\mathbb{E}[X] = \int_{-\infty}^{\infty} x \, p(x) \, dx
$$
> **Readable form:** The expectation or variance summarizes the distribution by averaging outcomes or squared deviations.

**Function of RV:**

$$
\mathbb{E}[g(X)] = \sum_x g(x) P(X=x) \quad \text{or} \quad \int g(x) p(x) dx
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**Linearity** (always):

$$
\mathbb{E}[aX + bY] = a\mathbb{E}[X] + b\mathbb{E}[Y]
$$
> **Readable form:** The expectation or variance summarizes the distribution by averaging outcomes or squared deviations.

even if $X, Y$ dependent.

---

## Variance and Standard Deviation

$$
\text{Var}(X) = \mathbb{E}[(X - \mathbb{E}[X])^2] = \mathbb{E}[X^2] - (\mathbb{E}[X])^2
$$
> **Readable form:** The expectation or variance summarizes the distribution by averaging outcomes or squared deviations.

$$
\sigma_X = \sqrt{\text{Var}(X)}
$$
> **Readable form:** The expectation or variance summarizes the distribution by averaging outcomes or squared deviations.

**Variance of sum** (if independent):

$$
\text{Var}(X + Y) = \text{Var}(X) + \text{Var}(Y)
$$
> **Readable form:** The expectation or variance summarizes the distribution by averaging outcomes or squared deviations.

```python
X = np.random.binomial(10, 0.5, size=10000)
print(np.mean(X), np.var(X))  # approx 5.0, 2.5
```

---

## Covariance and Correlation

$$
\text{Cov}(X,Y) = \mathbb{E}[(X-\mathbb{E}[X])(Y-\mathbb{E}[Y])] = \mathbb{E}[XY] - \mathbb{E}[X]\mathbb{E}[Y]
$$
> **Readable form:** The equation projects data onto directions that preserve the most variance in a lower-dimensional representation.

$$
\rho_{X,Y} = \frac{\text{Cov}(X,Y)}{\sigma_X \sigma_Y} \in [-1, 1]
$$
> **Readable form:** The equation projects data onto directions that preserve the most variance in a lower-dimensional representation.

**Covariance matrix** for vector $\mathbf{X}$:

$$
\boldsymbol{\Sigma}_{ij} = \text{Cov}(X_i, X_j)
$$
> **Readable form:** The equation projects data onto directions that preserve the most variance in a lower-dimensional representation.

Diagonal entries are variances; off-diagonal are covariances.

---

## Conditional Expectation

$$
\mathbb{E}[X \mid Y=y] = \sum_x x P(X=x \mid Y=y)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**Tower property:**

$$
\mathbb{E}[\mathbb{E}[X \mid Y]] = \mathbb{E}[X]
$$
> **Readable form:** The expectation or variance summarizes the distribution by averaging outcomes or squared deviations.

Used in EM algorithm, Kalman filtering, policy evaluation.

---

## Inequalities

**Markov:** for nonnegative $X$ and $a > 0$:

$$
P(X \geq a) \leq \frac{\mathbb{E}[X]}{a}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**Chebyshev:**

$$
P(|X - \mathbb{E}[X]| \geq k\sigma) \leq \frac{1}{k^2}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**Jensen** (convex $\phi$):

$$
\phi(\mathbb{E}[X]) \leq \mathbb{E}[\phi(X)]
$$
> **Readable form:** This formal sentence is the symbolic version of the relationship stated in the surrounding explanation.

Used in variational inference proofs.

---

## Law of Large Numbers

Sample mean $\bar{X}_n = \frac{1}{n}\sum_{i=1}^n X_i$ converges to $\mathbb{E}[X]$ as $n \to \infty$.

Justifies Monte Carlo estimation:

$$
\mathbb{E}[f(X)] \approx \frac{1}{n}\sum_{i=1}^n f(X_i)
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

---

## AI Applications

| Application | Expectation/variance use |
|-------------|-------------------------|
| MSE loss | $\mathbb{E}[(y - \hat{y})^2]$ |
| Risk-sensitive planning | Variance of returns |
| Bayesian decision theory | Expected utility |
| MCMC | Monte Carlo expectations |
| Batch normalization | Control mean/variance of activations |

---

---

## Study Notes

Russell and Norvig present this material as foundational for multiple AI subfields. When working through exercises:

- Re-derive key formulas before looking up answers
- Implement at least one algorithm or calculation in Python
- Connect concepts to the relevant course chapter
- Test edge cases - algorithms often fail on boundaries first

> **Readable form:** Mastery means you can explain it, compute it, and code it - not just recognize the definition on a flashcard.

### Cross-Chapter Connections

| Related chapter | Connection |
|----------------|------------|
| Earlier foundations | Provides prerequisites for formal derivations |
| Applied chapters | Techniques appear in agents, learning, and perception |
| Ethics and safety | Deployment context shapes how methods are used |

### Practice Checklist

| Step | Action |
|------|--------|
| 1 | Read the AIMA section alongside this section |
| 2 | Complete all five exercises |
| 3 | Implement one code example from scratch |
| 4 | Explain one key formula in plain English |
| 5 | Identify one real-world application |

---

## Additional Examples

### Worked Example

Consider applying the concepts from this section to a concrete AI problem. Identify inputs, outputs, constraints, and evaluation criteria before implementing. Start with the smallest instance that exercises the core idea.

### Implementation Reminder

```python
# Template: verify understanding with executable code
def verify_understanding():
    """Replace with section-specific test."""
    assert True  # placeholder - implement real check
    return "ok"
```

### Summary Table

| Concept | Definition | AI application |
|---------|------------|----------------|
| Core idea | See section sections above | Chapter-specific |
| Key formula | See math blocks above | Computation |
| Pitfall | Common student error | Debugging |
| Extension | Advanced topic | Further reading |

> **Readable form:** If you can fill in this table from memory, you understand the section well enough to move on.

## Key Takeaways

1. Expectation is the probability-weighted average; linearity holds regardless of independence.
2. Variance measures spread; for independent variables, variances of sums add.
3. Covariance and correlation quantify linear relationships between variables.
4. Conditional expectation underlies EM, filtering, and nested random models.
5. Markov, Chebyshev, and LLN support theoretical guarantees for learning algorithms.

## Exercises

1. Compute $\mathbb{E}[X]$ and $\text{Var}(X)$ for $\text{Bernoulli}(p)$.
2. Prove $\text{Var}(X) = \mathbb{E}[X^2] - (\mathbb{E}[X])^2$.
3. For independent $X, Y$ with variances 2 and 3, find $\text{Var}(2X - Y)$.
4. Use Chebyshev to bound $P(|X - 50| \geq 20)$ when $\text{Var}(X) = 64$.
5. Estimate $\mathbb{E}[\sin(X)]$ for $X \sim \mathcal{N}(0,1)$ via Monte Carlo with 10,000 samples.

---

**Previous:** [Section A.5 - Distributions](./section-05-distributions.md)

**Next:** [Section A.7 - Calculus for ML](./section-07-calculus-for-ml.md)
