# Section 12.2: Basic Probability

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 12.2  
> **Previous:** [Section 12.1 - Acting Under Uncertainty](./section-01-acting-under-uncertainty.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Sample Spaces and Events

[Section 12.1](./section-01-acting-under-uncertainty.md) argued that agents need **degrees of belief**. Chapter 12.2 supplies the formal language. Probabilistic statements are about **possible worlds** - instead of ruling worlds in or out, probability assigns each world a number between 0 and 1.

The set of all possible worlds is the **sample space** $\Omega$; an individual world is $\omega \in \Omega$. Worlds are **mutually exclusive** (at most one is true) and **exhaustive** (exactly one is true).

Rolling two fair dice gives 36 worlds: $(1,1), (1,2), \ldots, (6,6)$. If the dice are fair and independent:

$$
P(\omega) = \frac{1}{36} \quad \text{for each } \omega \in \Omega
$$
> **Readable form:** Sample space = all complete scenarios; each world gets a probability; all probabilities sum to 1.

An **event** is a set of worlds - the same idea as a proposition in logic. The probability of event $A$:

$$
P(A) = \sum_{\omega \in A} P(\omega)
$$
> **Readable form:** Event probability = sum of probabilities of all worlds where the event is true.

---

## Kolmogorov Axioms

AIMA builds probability on two foundational axioms (Equations 12.1-12.2):

$$
0 \leq P(\omega) \leq 1 \quad \text{for all } \omega, \qquad \sum_{\omega \in \Omega} P(\omega) = 1
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

From these, useful identities follow immediately:

$$
P(\neg a) = 1 - P(a)
$$

$$
P(a \lor b) = P(a) + P(b) - P(a \land b)
$$
> **Readable form:** Negation flips probability; for OR, add individual probabilities but subtract the overlap counted twice.

These are **Kolmogorov's axioms**. Violating them is not merely sloppy - de Finetti showed that an agent with inconsistent beliefs can be forced into a **Dutch book**: a set of bets guaranteeing loss (AIMA Figure 12.2).

| Agent belief | Problem |
|--------------|---------|
| $P(a)=0.4, P(b)=0.3, P(a \land b)=0, P(a \lor b)=0.8$ | Violates inclusion-exclusion |
| Rational agent | Beliefs satisfy axioms; no guaranteed-loss betting scheme exists |

---

## Priors, Evidence, and Conditionals

**Unconditional** (or **prior**) probabilities $P(X)$ express belief before specific evidence:

$$
P(\text{cavity}) = 0.2
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**Conditional** (or **posterior**) probabilities $P(X \mid e)$ condition on evidence $e$:

$$
P(\text{cavity} \mid \text{toothache}) = 0.6
$$
> **Readable form:** Prior = belief with no extra info; posterior = belief after seeing evidence.

Crucially, $P(\text{cavity}) = 0.2$ remains valid after observing toothache - it is simply not the number you need for diagnosis. Conditioning is not logical implication: toothache does not *cause* us to conclude cavity with certainty; it updates our **degree of belief** given partial information.

The definition of conditional probability (when $P(b) > 0$):

$$
P(a \mid b) = \frac{P(a \land b)}{P(b)}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Full treatment in [Section 12.3](./section-03-conditional-probability.md).

---

## Random Variables and Distributions

Variables in probability are **random variables** (capitalized: `Cavity`, `Weather`, `Total`). Each maps worlds to values in a **range**. Boolean variables use $\{\text{true}, \text{false}\}$ or $\{0, 1\}$ (Bernoulli).

A **probability distribution** assigns probabilities to each value:

$$
\mathbf{P}(\text{Weather}) = \langle 0.6, 0.1, 0.29, 0.01 \rangle
$$
> **Readable form:** The probability expression encodes how evidence, hidden variables, or outcomes combine under the model.

for $\langle \text{sun}, \text{rain}, \text{cloud}, \text{snow} \rangle$. Bold $\mathbf{P}$ indicates a vector over values.

| Variable type | Range example | Distribution name |
|---------------|---------------|-------------------|
| Boolean | $\{\text{true}, \text{false}\}$ | Bernoulli |
| Finite discrete | $\{1,\ldots,6\}$ | Categorical |
| Continuous | $\mathbb{R}$ | Density function |

For continuous variables, $P(X = x)$ is a **probability density** - the probability of an exact point is 0; we integrate over intervals:

$$
P(18°\text{C} \leq \text{NoonTemp} \leq 26°\text{C}) = \int_{18}^{26} p(x)\, dx = 1
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Uniform density on $[18, 26]$: $p(x) = 1/8$ per degree (width 8°C).

---

## Joint Distributions

For multiple variables, the **joint distribution** $\mathbf{P}(X, Y)$ lists probabilities of all value combinations. For `Weather` (4 values) and `Cavity` (2 values), the joint table has $4 \times 2 = 8$ entries.

A **possible world** in the factored representation assigns a value to every random variable. With `Cavity`, `Toothache`, `Weather`: $2 \times 2 \times 4 = 16$ worlds.

The **full joint distribution** over all variables completely specifies the probability model. In principle, any query can be answered by summing entries from the full joint - but the table has $O(2^n)$ entries for $n$ Boolean variables ([Chapter 13](../chapter-13-probabilistic-reasoning/README.md) addresses this).

```python
import numpy as np

# Simplified dental joint (AIMA Fig 12.3 style)
joint = np.array([
    # toothache=T, catch=T/F; cavity=T/F rows
    [[0.108, 0.012], [0.072, 0.008]],
    [[0.016, 0.064], [0.144, 0.576]],
])

p_cavity = joint[:, :, :].sum()  # marginal - use proper indexing in practice
# P(cavity) = 0.108+0.012+0.072+0.008 = 0.200
print(f"P(cavity) = {0.108+0.012+0.072+0.008:.3f}")
```

---

## Frequentist vs. Subjectivist Interpretations

AIMA notes two philosophical camps:

| View | Meaning of $P(\text{cavity}) = 0.2$ |
|------|-------------------------------------|
| **Frequentist** | In the long run, 20% of similar patients have cavities |
| **Subjectivist** | The agent's justified degree of belief is 0.2 |

For AI engineering, the **subjectivist** view dominates: probabilities encode an agent's knowledge state and update with evidence via [Bayes' rule](../../GLOSSARY.md#bayes-rule). The axioms constrain how beliefs must cohere, not which priors are "correct."

---

## Connection to Logic

Probability and logic share the same **ontology** (possible worlds) but differ **epistemologically**:

| Logic | Probability |
|-------|-------------|
| Sentence is true, false, or unknown | Degree of belief in $[0, 1]$ |
| Conjunction via AND | Product rule (with conditioning) |
| Entailment is all-or-nothing | High probability, rarely certainty |

[Chapter 07](../chapter-07-logical-agents/README.md) agents use KB entailment; probabilistic agents maintain distributions. [Chapter 13](../chapter-13-probabilistic-reasoning/README.md) combines both via Bayesian networks.

---

## Python: Verifying Axioms

```python
import numpy as np

def check_axioms(probs):
    '''Verify Kolmogorov axioms for a discrete distribution.'''
    probs = np.asarray(probs, dtype=float)
    assert np.all(probs >= 0) and np.all(probs <= 1), "Out of [0,1] range"
    assert np.isclose(probs.sum(), 1.0), f"Sum is {probs.sum()}, not 1"
    return True

check_axioms(np.ones(6) / 6)  # fair die

p_a, p_b, p_ab = 0.4, 0.3, 0.1
p_or = p_a + p_b - p_ab
assert np.isclose(p_or, 0.6)
print(f"P(a or b) = {p_or}")
```

---

## Key Takeaways

1. Sample space $\Omega$ lists mutually exclusive, exhaustive possible worlds.
2. Kolmogorov axioms constrain coherent degrees of belief.
3. Random variables map worlds to values; distributions assign probabilities.
4. The full joint distribution answers any query but grows exponentially.
5. Priors and posteriors are both valid - they answer different questions.



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

1. Why can't $P(\text{cavity} \mid \text{toothache})$ replace $P(\text{cavity})$ after observing toothache?
2. What is the difference between a probability and a probability density?
3. How many independent parameters does a joint over 5 Boolean variables require?
4. Why do Kolmogorov axioms matter for AI agents?
5. When is a frequentist interpretation natural vs. a subjectivist one?

---

**Next:** [Section 12.3 - Conditional Probability](./section-03-conditional-probability.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. de Finetti, B. (1937). La prévision. *Annales de l'Institut Henri Poincaré*.
3. Grinstead, C., & Snell, J. (1997). *Introduction to Probability*. AMS.
4. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python)

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Sample space** | The set Ω of all mutually exclusive, exhaustive possible worlds. |
| **Event** | A set of worlds; probability is the sum of world probabilities in the set. |
| **Random variable** | A function mapping each world to a value in the variable's range. |
| **Joint distribution** | A probability assignment to every combination of variable values. |
| **Kolmogorov axioms** | Non-negativity, normalization to 1, and additivity for disjoint events. |
