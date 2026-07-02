# Section 12.4: Independence

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 12.4  
> **Previous:** [Section 12.3 - Conditional Probability](./section-03-conditional-probability.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Unconditional Independence

Variables $X$ and $Y$ are **independent** iff:

$$
P(X \mid Y) = P(X) \quad \text{equivalently} \quad P(X, Y) = P(X) \, P(Y)
$$
> **Readable form:** Learning $Y$ tells you nothing about $X$ - the joint is the product of marginals.

Example: fair coin flips $C_1, C_2$ - $P(C_2 \mid C_1) = P(C_2) = 0.5$. Dental **Toothache** and **Meteorite** striking the office are (we hope) independent.

Independence **dramatically** reduces parameters: joint over $n$ Boolean independent variables needs $n$ numbers instead of $2^n - 1$.

---

## Conditional Independence

More useful in AI: $X$ and $Y$ are **conditionally independent** given $Z$ iff:

$$
P(X \mid Y, Z) = P(X \mid Z)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

equivalently:

$$
P(X, Y \mid Z) = P(X \mid Z) \, P(Y \mid Z)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Notation: $(X \perp Y) \mid Z$.

| Relationship | Meaning |
|--------------|---------|
| Independent | No direct information flow |
| Conditionally independent given Z | Any shared influence is mediated through Z |

Example (AIMA alarm network preview): $(\text{JohnCalls} \perp \text{MaryCalls}) \mid \text{Alarm}$. Given whether the alarm sounded, the roommates' phone calls do not inform each other.

---

## Naive Bayes Assumption

For classification with features $F_1, \ldots, F_n$ and class $C$:

P(C \mid f_1, \ldots, f_n) \propto P(C) \prod_i P(f_i \mid C)
$$

This assumes **features are conditionally independent given the class** - almost always false, yet often works well. Full treatment in [Section 12.6](./section-06-naive-bayes-classifier.md).

$$
> **Readable form:** Naive Bayes pretends that once you know the class, features don't influence each other.

---

## Why Independence Matters for Representation

| Model | Parameters for $n$ Booleans |
|-------|----------------------------|
| Full joint | $2^n - 1$ |
| Fully independent | $n$ |
| Bayesian network with sparse parents | Often $O(n)$ to $O(n \cdot k)$ |

[Chapter 13](../chapter-13-probabilistic-reasoning/README.md) builds graphs whose edges encode **conditional dependence** and whose missing edges encode **conditional independence**.

---

## Testing Independence from Data

Empirical checks use contingency tables and chi-squared tests (outside AIMA scope). Caution: **finite samples** can suggest independence that is not true, or hide weak dependence. Structural assumptions (naive Bayes) trade bias for variance - a theme in [Chapter 20](../chapter-20-learning-probabilistic-models/README.md).

```python
import numpy as np

def naive_bayes_posterior(prior_c, likelihoods_f_given_c, p_evidence):
    '''Unnormalized P(C|f) up to scale; likelihoods_f_given_c is list per feature.'''
    num = prior_c
    for lf in likelihoods_f_given_c:
        num *= lf
    return num / p_evidence

# Two binary features, one class
prior = 0.6
likes = [0.8, 0.3]  # P(f_i | C)
p_e = 0.25  # precomputed
print(f"P(C|f) ≈ {naive_bayes_posterior(prior, likes, p_e):.3f}")
```

---

## Explaining Away (Preview)

Three variables: Sprinkler, Rain, WetGrass. Sprinkler and Rain are independent, but both cause WetGrass. Observing wet grass makes rain **more** likely; observing **also** that the sprinkler ran **lowers** belief in rain - the grass is "explained away" by the sprinkler.

$$
P(\text{Rain} \mid \text{wet}, \text{sprinkler}) < P(\text{Rain} \mid \text{wet})
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Full analysis in [Chapter 13](../chapter-13-probabilistic-reasoning/README.md) with Bayesian networks.

---

## Gaussian Naive Bayes for Continuous Features

When features are continuous, assume $P(f_i \mid c) = \mathcal{N}(\mu_{i,c}, \sigma_{i,c}^2)$. Training estimates per-class means and variances. Used for simple document length features or sensor readings alongside discrete naive Bayes.

---

## Information-Theoretic View

Conditional independence reduces **mutual information** between variables given $Z$. If $(X \perp Y) \mid Z$, then $I(X;Y \mid Z) = 0$. This connects to feature selection and model compression in machine learning.

---

## Worked Example: Three Coin Tosses

Fair coins $C_1, C_2, C_3$ - all independent. $P(C_1{=}H, C_2{=}H, C_3{=}T) = 0.5^3 = 0.125$. If instead $C_2$ always copies $C_1$, independence fails and joint needs only 4 parameters for the dependency structure.

---

## Link to Chapter 20

[Chapter 20](../chapter-20-learning-probabilistic-models/README.md) learns parameters and structure from data. Independence assumptions determine which parameters are free during **maximum likelihood** estimation.

---

## Common Mistakes

**Assuming independence because variables seem unrelated.** Hidden common causes create dependence - see **explaining away** in [Chapter 13](../chapter-13-probabilistic-reasoning/README.md).

**Confusing marginal and conditional independence.** $X, Y$ independent does not imply independent given $Z$ (and vice versa).

**Using naive Bayes when features are strongly correlated.** Performance may degrade; consider feature engineering or richer models.

---

## Key Takeaways

1. Independence factorizes joints into products of marginals.
2. Conditional independence given $Z$ is the key structural assumption in BNs.
3. Naive Bayes applies conditional independence of features given class.
4. Independence assumptions reduce parameters from exponential to linear.
5. Graph topology in Chapter 13 encodes independence claims.



## Study Guide

Work through each equation with the readable-form blockquote, then reproduce the main formulas from memory before opening the next section.

---

## Notation Review

| Symbol | Meaning |
|--------|---------|
| $P(X)$ | Prior or marginal |
| $P(X \mid e)$ | Posterior given evidence |
| $P(X,Y)$ | Joint probability |

---

## Worked Drill

1. State the main definition.
2. Apply to a numeric toy.
3. Run the Python snippet.
4. List one common mistake.
5. Link to the **Next** section.

---

## Practice Problems

1. Define glossary terms from memory.
2. Compute a small numerical exercise.
3. Compare to [Chapter 07](../chapter-07-logical-agents/README.md) logical reasoning.
4. Preview how [Chapter 13](../chapter-13-probabilistic-reasoning/README.md) builds on this idea.

---

## Engineering Note

Calibrate probabilistic models on held-out data; base rates and independence assumptions often break silently in deployment.

---

## Deep Dive

Trace one AIMA figure or example related to this section. Annotate each arrow or table entry with the corresponding probability factor. Teaching others is the fastest way to expose gaps in your own understanding.

---


---

## Reflection Questions

1. Give an example where X and Y are independent but not independent given Z.
2. Why is naive Bayes 'naive'? When is the assumption harmless?
3. How many parameters in a joint over 10 Booleans if all are independent?
4. Express conditional independence in terms of the chain rule.
5. How does conditional independence relate to d-separation (preview)?

---

**Next:** [Section 12.5 - Bayes' Rule Applications](./section-05-bayes-rule-applications.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Pearl, J. (1988). *Probabilistic Reasoning in Intelligent Systems*. Morgan Kaufmann.
3. Murphy, K. (2022). *Probabilistic Machine Learning*. MIT Press.
4. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python)

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Independence** | P(X,Y)=P(X)P(Y); observing Y does not change belief about X. |
| **Conditional independence** | (X ⊥ Y)|Z: given Z, X and Y provide no extra information about each other. |
| **Naive Bayes assumption** | Features are conditionally independent given the class label. |
| **Explaining away** | When two causes become dependent after observing their common effect. |


