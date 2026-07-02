# Section 12.3: Conditional Probability

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 12.3  
> **Previous:** [Section 12.2 - Basic Probability](./section-02-basic-probability.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## The Definition of Conditioning

[Section 12.2](./section-02-basic-probability.md) introduced priors and posteriors. Chapter 12.3 makes conditioning precise. When $P(b) > 0$:

$$
P(a \mid b) = \frac{P(a \land b)}{P(b)}
$$
> **Readable form:** Probability of $a$ given $b$ = share of $b$-worlds that are also $a$-worlds.

Rearranging gives the **product rule**:

$$
P(a \land b) = P(a \mid b) \, P(b) = P(b \mid a) \, P(a)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

This is the workhorse identity for probabilistic reasoning - every inference algorithm in [Chapter 13](../chapter-13-probabilistic-reasoning/README.md) exploits variants of it.

---

## The Chain Rule

For variables $X_1, \ldots, X_n$, the joint factors recursively:

$$
P(x_1, \ldots, x_n) = P(x_1) \, P(x_2 \mid x_1) \, P(x_3 \mid x_1, x_2) \cdots P(x_n \mid x_1, \ldots, x_{n-1})
$$
> **Readable form:** Build the full joint by multiplying conditional probabilities one variable at a time.

For three dental variables:

$$
P(c, t, e) = P(c) \, P(t \mid c) \, P(e \mid t, c)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Each factor may be easier to elicit from experts than a flat $2^3$ table. [Section 12.4](./section-04-independence.md) shows when factors simplify further.

---

## Bayes' Rule

From the product rule, when $P(b) > 0$:

$$
P(a \mid b) = \frac{P(b \mid a) \, P(a)}{P(b)}
$$
> **Readable form:** Posterior = (likelihood × prior) / evidence probability.

In diagnostic notation with hypothesis $h$ and evidence $e$:

$$
P(h \mid e) = \frac{P(e \mid h) \, P(h)}{P(e)}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

| Term | Name | Typical source |
|------|------|----------------|
| $P(h)$ | Prior | Population base rate, expert judgment |
| $P(e \mid h)$ | Likelihood | Sensor model, test characteristics |
| $P(e)$ | Evidence | Normalizing constant; often computed by summing |
| $P(h \mid e)$ | Posterior | What we want after observing $e$ |

Applications in [Section 12.5](./section-05-bayes-rule-applications.md).

---

## Computing $P(e)$: The Law of Total Probability

When hypotheses $H_1, \ldots, H_k$ partition the space:

$$
P(e) = \sum_{i=1}^{k} P(e \mid H_i) \, P(H_i)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Example: disease test with $H \in \{\text{disease}, \text{healthy}\}$:

$$
P(\text{positive}) = P(+ \mid D) P(D) + P(+ \mid \neg D) P(\neg D)
$$
> **Readable form:** Overall rate of positive tests = positives among sick × rate of sickness + positives among healthy × rate of health.

---

## Marginalization

To obtain $P(X)$ from a joint, **sum out** other variables:

$$
P(X) = \sum_y P(X, y)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

For Boolean $Cavity$ and $Toothache$:

$$
P(c) = P(c, t) + P(c, \neg t)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Marginalization is the brute-force inference method on joint tables - exact but expensive. Bayesian networks ([Chapter 13](../chapter-13-probabilistic-reasoning/README.md)) exploit structure to avoid summing everything.

```python
import numpy as np

# joint[cavity][toothache] simplified
joint = np.array([[0.576, 0.144], [0.080, 0.200]])  # rows: notooth, tooth; cols: nocav, cav
p_cavity = joint[:, 1].sum()
p_toothache = joint[1, :].sum()
print(f"P(cavity)={p_cavity:.3f}, P(toothache)={p_toothache:.3f}")
```

---

## Conditioning and Causation

$P(\text{cavity} \mid \text{toothache}) \neq P(\text{toothache} \mid \text{cavity})$ in general. **Causation** runs from cavity → toothache, but **diagnostic inference** runs backward using Bayes' rule. Pearl's work (see [PEARL references]) formalizes causal vs. evidential direction; for now, remember: conditioning answers "how should I update belief?" not "what caused what?"

---

## Python: Bayes' Rule Helper

```python
def bayes(prior_h, likelihood_e_given_h, p_e):
    '''P(h|e) = P(e|h)*P(h) / P(e).'''
    return likelihood_e_given_h * prior_h / p_e

def total_probability(likelihoods, priors):
    '''P(e) = sum_h P(e|h) P(h).'''
    return sum(le * ph for le, ph in zip(likelihoods, priors))

# Medical toy example
p_disease = 0.01
p_pos_given_d = 0.99
p_pos_given_healthy = 0.05
p_pos = total_probability([p_pos_given_d, p_pos_given_healthy], [p_disease, 1 - p_disease])
p_d_given_pos = bayes(p_disease, p_pos_given_d, p_pos)
print(f"P(disease|+) = {p_d_given_pos:.4f}")
```

---

## Worked Example: Dental Joint

AIMA's dental example ties together conditioning and marginalization. Given $P(c,t,e)$ factors or a joint table, compute $P(c \mid t)$:

$$
P(c \mid t) = \frac{P(c, t)}{P(t)} = \frac{\sum_e P(c, t, e)}{\sum_{c',e} P(c', t, e)}
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Step-by-step enumeration prevents sign errors. Always verify marginals sum to 1.

| Query | Formula type |
|-------|--------------|
| $P(c \mid t)$ | Conditional from joint |
| $P(t)$ | Marginal over $c, e$ |
| $P(c \mid t, e)$ | Conditioning on multiple evidence vars |

---

## Full Joint vs. Factored Representation

Storing $P(X_1,\ldots,X_n)$ explicitly requires $O(\prod_i |X_i|)$ space. The chain rule motivates **Bayesian networks** ([Chapter 13](../chapter-13-probabilistic-reasoning/README.md)): sparse parent sets yield compact factors $P(X_i \mid \text{Pa}(X_i))$.

| Representation | Space | Query cost (naive) |
|----------------|-------|-------------------|
| Full joint table | Exponential | Linear in table size |
| Chain rule factors | Depends on parent count | Still hard without structure |
| Bayesian network | Often polynomial | Variable elimination, sampling |

---

## Diagnostic Reasoning Pipeline

1. Specify hypotheses $\{H_i\}$ and prior $P(H_i)$.
2. Specify likelihood model $P(e \mid H_i)$ for each observation.
3. Compute $P(e)$ via total probability.
4. Apply Bayes' rule for each hypothesis.
5. Normalize if working with unnormalized posteriors.

This pipeline appears in medical AI, spam filters ([Section 12.5](./section-05-bayes-rule-applications.md)), and sensor fusion.

---

## Connection to Entailment

Logical entailment $KB \models \alpha$ is true in **all** worlds satisfying $KB$. Probabilistic entailment asks whether $P(\alpha \mid KB) \geq 1 - \epsilon$. As $\epsilon \to 0$, we approach logical certainty - but probabilistic agents rarely need $\epsilon = 0$ for action.

---

## Common Mistakes

**Dividing by zero.** $P(a \mid b)$ undefined when $P(b) = 0$ - evidence must be possible under the model.

**Forgetting to normalize.** Unnormalized products are fine for argmax classification but not for calibrated probabilities.

**Confusing $P(a \mid b)$ with $P(b \mid a)$.** Bayes' rule swaps them with priors and evidence terms.

---

## Key Takeaways

1. Conditional probability is defined via $P(a \land b) / P(b)$.
2. The product rule and chain rule factor joints into conditionals.
3. Bayes' rule reverses conditioning direction using priors and likelihoods.
4. Marginalization sums hidden variables out of joints.
5. Conditioning updates belief; it does not by itself establish causation.



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

1. Derive Bayes' rule from the product rule in one line of algebra.
2. Why must $P(e)$ appear in the denominator of Bayes' rule?
3. How does the chain rule relate to Bayesian network factorization?
4. When are $P(a|b)$ and $P(b|a)$ equal?
5. What is the computational cost of marginalizing $k$ variables from a joint?

---

**Next:** [Section 12.4 - Independence](./section-04-independence.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Pearl, J. (1988). *Probabilistic Reasoning in Intelligent Systems*. Morgan Kaufmann.
3. Grinstead, C., & Snell, J. (1997). *Introduction to Probability*. AMS.
4. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python)

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Conditional probability** | P(a|b): probability of a among worlds where b holds. |
| **Product rule** | P(a∧b) = P(a|b)P(b); foundation for chain rule and Bayes. |
| **Chain rule** | Factorization of a joint into a product of conditionals. |
| **Bayes' rule** | Formula for updating P(h) to P(h|e) using likelihood and evidence. |
| **Marginalization** | Summing a joint over values of variables to eliminate them. |



