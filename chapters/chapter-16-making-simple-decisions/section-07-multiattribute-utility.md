# Section 16.7: Multiattribute Utility

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 16.9-16.10  
> **Previous:** [Section 16.6 - Human Biases](./section-06-human-biases.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Decisions with Multiple Objectives

Real decisions rarely hinge on a single attribute. Choosing a job involves salary, location, growth, and work-life balance. Medical treatment balances survival, side effects, and cost. Autonomous vehicle policies trade safety, speed, and passenger comfort.

**Multiattribute utility theory (MAUT)** extends [utility theory](./section-02-utility-theory.md) to vectors of outcomes $\mathbf{x} = (x_1, \ldots, x_n)$.

> **Readable form:** When you care about many things at once, you need a rule for trading off - more money vs. worse commute, etc.

---

## Attributes and Outcomes

An **attribute** is a dimension of value (cost, risk, quality). An **outcome** is a bundle $\mathbf{x} = (x_1, \ldots, x_n)$ specifying each attribute's level.

Single-attribute utility $u_i(x_i)$ scores one dimension. **Overall utility** $U(\mathbf{x})$ combines them.

Naive approaches that fail:

| Approach | Problem |
|----------|---------|
| Optimize one attribute | Ignores trade-offs |
| Fixed thresholds | "Salary > 100k AND commute < 30min" may be infeasible |
| Unweighted sum of raw units | Adding dollars + minutes is meaningless |

MAUT seeks coherent $U(\mathbf{x})$ consistent with preferences over multiattribute outcomes.

---

## Preferential Independence

Attribute $X_1$ is **preferentially independent** of $X_2$ if preferences over $X_1$ don't depend on the level of $X_2$:

$$
(x_1, x_2) \succ (x_1', x_2) \iff (x_1, x_2') \succ (x_1', x_2')
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

for all $x_2, x_2'$.

**Utility independence** (stronger): preferences over lotteries on $X_1$ independent of $X_2$ level.

When attributes are **mutually utility independent**, additive form is justified:

U(\mathbf{x}) = \sum_{i=1}^{n} k_i \, u_i(x_i)
$$

with $k_i > 0$ scaling constants (after normalizing each $u_i$ to $[0,1]$).

$$
> **Readable form:** If your trade-off between salary and commute doesn't change when health benefits improve, you might use a weighted sum of separate scores.

---

## Additive Multiattribute Utility

Normalized attributes $u_i(x_i) \in [0,1]$:

$$
U(\mathbf{x}) = \sum_{i=1}^{n} w_i \, u_i(x_i), \quad \sum_i w_i = 1, \; w_i \geq 0
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Assessing weights:**

1. Elicit single-attribute utilities $u_i$ via standard gambles ([Section 16.2](./section-02-utility-theory.md))
2. Determine **swing weights** - compare importance of improving each attribute from worst to best
3. Normalize weights to sum to 1

**Swing weighting example:**

| Attribute | Worst → Best swing | Weight |
|-----------|-------------------|--------|
| Safety | Most important | 0.5 |
| Cost | Half as important | 0.25 |
| Time | Same as cost | 0.25 |

---

## Multilinear Utility

When preferential independence fails, use **multilinear** form (pairwise interactions):

$$
U(\mathbf{x}) = \sum_i k_i u_i(x_i) + \sum_{i<j} k_{ij} u_i(x_i) u_j(x_j)
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

Interaction term $k_{ij} u_i u_j$ captures **complementarity** or **substitution**.

**Example:** High safety *and* low cost together worth more than sum of parts - positive $k_{ij}$.

Assessing all interaction terms is burdensome - used when additive model fits poorly.

---

## Worked Example: Car Purchase

Attributes: {price (k\$), fuel (mpg), safety rating}.

Normalized utilities (higher better):

| Car | $u_{\text{price}}$ | $u_{\text{fuel}}$ | $u_{\text{safety}}$ |
|-----|-------------------|-------------------|---------------------|
| A | 0.8 | 0.5 | 0.9 |
| B | 0.5 | 0.9 | 0.7 |
| C | 0.6 | 0.6 | 0.95 |

Weights: $w_{\text{price}} = 0.4$, $w_{\text{fuel}} = 0.3$, $w_{\text{safety}} = 0.3$.

$$
U(A) = 0.4(0.8) + 0.3(0.5) + 0.3(0.9) = 0.32 + 0.15 + 0.27 = 0.74
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

$$
U(B) = 0.4(0.5) + 0.3(0.9) + 0.3(0.7) = 0.20 + 0.27 + 0.21 = 0.68
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

$$
U(C) = 0.4(0.6) + 0.3(0.6) + 0.3(0.95) = 0.24 + 0.18 + 0.285 = 0.705
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

MEU ranking: A > C > B.

---

## Uncertainty Over Multiattribute Outcomes

When attributes are uncertain, maximize **expected multiattribute utility**:

$$
\text{EU}(a) = \sum_{\mathbf{x}} P(\mathbf{x} \mid a) \cdot U(\mathbf{x})
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Decision networks can place a utility node depending on multiple chance nodes - each configuration maps to $\mathbf{x}$.

**VPI** extends naturally: is observing an attribute worth the cost if it changes the multiattribute ranking?

---

## Pareto Dominance

Outcome $\mathbf{x}$ **Pareto-dominates** $\mathbf{x}'$ if $\mathbf{x}$ is at least as good on every attribute and strictly better on at least one:

$$
\forall i: x_i \geq x_i' \text{ and } \exists j: x_j > x_j'
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

Dominated alternatives need not enter MAUT - eliminate them first.

**Pareto frontier:** non-dominated options for negotiation among stakeholders with different weights.

---

## Connection to MDP Rewards

[Chapter 17](../chapter-17-making-complex-decisions/README.md) reward $R(s,a,s')$ is often **scalar**. Multiobjective MDPs use vector rewards $\mathbf{R}$ and scalarization $w \cdot \mathbf{R}$ or Pareto-optimal policy sets.

[Chapter 22](../chapter-22-reinforcement-learning/README.md) **reward shaping** with multiple terms (progress, safety penalty, energy) is informal MAUT - risks misalignment if weights wrong.

---

## AI Applications

| Domain | Multiattribute formulation |
|--------|---------------------------|
| Recommendation | Relevance, diversity, novelty, fairness |
| Autonomous driving | Safety, legality, comfort, efficiency |
| Medical AI | Efficacy, toxicity, cost, patient preference |
| LLM alignment | Helpfulness, harmlessness, honesty - RLHF weights |

Explicit weight elicitation from stakeholders beats implicit aggregation in loss functions.

---

## Python: Additive MAUT

```python
def additive_maut(attributes, weights, utilities):
    """
    attributes: list of attribute names
    weights: dict name -> w_i (sum to 1)
    utilities: dict name -> u_i(x) in [0,1]
    """
    assert abs(sum(weights.values()) - 1.0) < 1e-6
    return sum(weights[a] * utilities[a] for a in attributes)

cars = {
    "A": {"price": 0.8, "fuel": 0.5, "safety": 0.9},
    "B": {"price": 0.5, "fuel": 0.9, "safety": 0.7},
    "C": {"price": 0.6, "fuel": 0.6, "safety": 0.95},
}
weights = {"price": 0.4, "fuel": 0.3, "safety": 0.3}
for name, utils in cars.items():
    print(name, additive_maut(weights.keys(), weights, utils))
```

---

## Common Mistakes

**Assuming additive without checking independence.** Preferences may change with context - test with pairwise comparisons.

**Using equal weights by default.** "0.33 each" hides political choices.

**Ignoring uncertainty.** Point estimates on each attribute miss correlated risks.

**Interpersonal utility comparison.** Group decisions need explicit aggregation rules ([Section 16.5](./section-05-decision-analysis-process.md)).

---

## Reflection Questions

1. Give an example where preferential independence fails for two attributes.
2. How does swing weighting differ from ranking attributes?
3. When would you use multilinear instead of additive utility?
4. How does Pareto dominance simplify a decision before MAUT?
5. Propose multiattribute utilities for an AI content moderation policy.

---

**Previous:** [Section 16.6 - Human Biases](./section-06-human-biases.md)  
**Next:** [Chapter 17 - Making Complex Decisions](../chapter-17-making-complex-decisions/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Sections 16.9-16.10. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Keeney, R. L., & Raiffa, H. (1976). *Decisions with Multiple Objectives*.
3. Keeney, R. L. - utility independence concepts. *Operations Research*.
4. Fishburn, P. C. - multiattribute expected utility theory.
5. AIMA Python: multiattribute examples in decision theory chapters.


