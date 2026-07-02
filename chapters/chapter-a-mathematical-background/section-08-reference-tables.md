# Section A.8: Reference Tables

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Appendix A.8  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Quick Reference for AIMA

This section consolidates **identities, formulas, and tables** used across AIMA chapters. Bookmark for problem sets, exams, and implementation work.

> **Readable form:** A cheat sheet - the formulas you reach for repeatedly so you don't have to re-derive them every time.

---

## Logarithm and Exponent Identities

| Identity | Formula |
|----------|---------|
| Product | $\log(ab) = \log a + \log b$ |
| Quotient | $\log(a/b) = \log a - \log b$ |
| Power | $\log(a^b) = b \log a$ |
| Change of base | $\log_b a = \frac{\log_c a}{\log_c b}$ |
| Exponential product | $e^{a+b} = e^a e^b$ |
| Softmax stable | $\text{softmax}(\mathbf{z})_i = \frac{e^{z_i - \max z}}{\sum_j e^{z_j - \max z}}$ |

---

## Trigonometry

| Identity | Formula |
|----------|---------|
| Pythagorean | $\sin^2\theta + \cos^2\theta = 1$ |
| Sum of angles | $\sin(\alpha+\beta) = \sin\alpha\cos\beta + \cos\alpha\sin\beta$ |
| Double angle | $\cos(2\theta) = \cos^2\theta - \sin^2\theta$ |
| Euler | $e^{i\theta} = \cos\theta + i\sin\theta$ |

2D rotation: $\mathbf{x}' = \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix} \mathbf{x}$

---

## Probability Quick Reference

| Distribution | PMF/PDF | Mean | Variance |
|--------------|---------|------|----------|
| $\text{Bernoulli}(p)$ | $p^x(1-p)^{1-x}$ | $p$ | $p(1-p)$ |
| $\text{Binomial}(n,p)$ | $\binom{n}{k}p^k(1-p)^{n-k}$ | $np$ | $np(1-p)$ |
| $\text{Poisson}(\lambda)$ | $\frac{\lambda^k e^{-\lambda}}{k!}$ | $\lambda$ | $\lambda$ |
| $\text{Uniform}(a,b)$ | $\frac{1}{b-a}$ | $\frac{a+b}{2}$ | $\frac{(b-a)^2}{12}$ |
| $\text{Gaussian}(\mu,\sigma^2)$ | see Section A.5 | $\mu$ | $\sigma^2$ |
| $\text{Exponential}(\lambda)$ | $\lambda e^{-\lambda x}$ | $1/\lambda$ | $1/\lambda^2$ |

**Bayes:** $P(H|e) = \frac{P(e|H)P(H)}{P(e)}$

---

## Information Theory

**Entropy:**

$$
H(X) = -\sum_x P(x) \log P(x)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**Cross-entropy:**

$$
H(P, Q) = -\sum_x P(x) \log Q(x)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**KL divergence:**

$$
D_{KL}(P \| Q) = \sum_x P(x) \log \frac{P(x)}{Q(x)}
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**Mutual information:**

$$
I(X; Y) = H(X) - H(X|Y) = D_{KL}(P(X,Y) \| P(X)P(Y))
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Used in decision trees, variational inference, representation learning.

---

## Matrix Identities

| Identity | Formula |
|----------|---------|
| Inverse product | $(\mathbf{AB})^{-1} = \mathbf{B}^{-1}\mathbf{A}^{-1}$ |
| Transpose product | $(\mathbf{AB})^\top = \mathbf{B}^\top\mathbf{A}^\top$ |
| Determinant product | $\det(\mathbf{AB}) = \det(\mathbf{A})\det(\mathbf{B})$ |
| Woodbury | $(\mathbf{A} + \mathbf{UCV})^{-1} = \mathbf{A}^{-1} - \mathbf{A}^{-1}\mathbf{U}(\mathbf{C}^{-1}+\mathbf{VA}^{-1}\mathbf{U})^{-1}\mathbf{VA}^{-1}$ |
| Trace cyclic | $\text{tr}(\mathbf{ABC}) = \text{tr}(\mathbf{CAB})$ |

---

## ML Loss Functions

| Loss | Formula | Use |
|------|---------|-----|
| MSE | $\frac{1}{n}\sum(y_i - \hat{y}_i)^2$ | Regression |
| MAE | $\frac{1}{n}\sum|y_i - \hat{y}_i|$ | Robust regression |
| Cross-entropy | $-\sum y_i \log \hat{y}_i$ | Classification |
| Hinge | $\max(0, 1 - y\hat{y})$ | SVM |
| Huber | Smooth L1/L2 hybrid | Robust regression |

---

## Optimization

| Algorithm | Update |
|-----------|--------|
| SGD | $\theta \leftarrow \theta - \eta \nabla L$ |
| Momentum | $v \leftarrow \beta v + \nabla L$; $\theta \leftarrow \theta - \eta v$ |
| Adam | Adaptive moments $\mathbf{m}, \mathbf{v}$ |

Learning rate schedules: step decay, cosine annealing, warmup.

---

## Complexity Classes

| Class | Examples |
|-------|----------|
| $O(1)$ | Hash lookup (average) |
| $O(\log n)$ | Binary search |
| $O(n)$ | Linear scan |
| $O(n \log n)$ | Merge sort, FFT |
| $O(n^2)$ | Naive matrix multiply |
| $O(2^n)$ | Subset enumeration |
| NP-complete | SAT, TSP, graph coloring |

---

## Greek Symbols in AIMA

| Symbol | Common meaning |
|--------|----------------|
| $\alpha, \beta$ | Search depths, learning rates |
| $\gamma$ | Discount factor (RL) |
| $\lambda$ | Regularization, eigenvalues |
| $\mu, \sigma$ | Mean, std dev |
| $\theta$ | Parameters |
| $\pi$ | Policy |
| $\Sigma$ | Covariance, summation |

See [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md) for full course notation.

---

## Python Reference

```python
import numpy as np
from scipy import stats, linalg, optimize

# Linear algebra
np.linalg.solve(A, b)
np.linalg.eig(A)
np.linalg.svd(A)

# Probability
stats.norm.pdf(x, loc=mu, scale=sigma)
np.random.multivariate_normal(mu, Sigma, size=n)

# Optimization
optimize.minimize(f, x0, jac=grad_f)
```

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

## Key Takeaways

1. Logarithm, trig, and matrix identities appear repeatedly in AI derivations.
2. Distribution tables summarize PMFs/PDFs with means and variances for quick lookup.
3. Entropy, cross-entropy, and KL divergence underpin information-theoretic learning.
4. ML loss functions and optimization updates are standard across chapters.
5. This reference complements MATH_CONVENTIONS.md - use both during coursework.

## Exercises

1. Simplify $\log\frac{p}{1-p}$ (log-odds) in terms of sigmoid.
2. Compute KL divergence between $\text{Bernoulli}(0.5)$ and $\text{Bernoulli}(0.8)$.
3. Verify Woodbury identity for 2×2 matrices numerically.
4. Write cross-entropy in terms of one-hot labels and softmax outputs.
5. Identify which complexity class applies to Dijkstra's algorithm on sparse graphs.

---

**Previous:** [Section A.7 - Calculus for ML](./section-07-calculus-for-ml.md)

**Next:** [Chapter B - Languages and Algorithms](../chapter-b-languages-algorithms/README.md)
