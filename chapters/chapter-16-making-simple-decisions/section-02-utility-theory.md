# Section 16.2: Utility Theory

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 16.3-16.4  
> **Previous:** [Section 16.1 - Combining Beliefs and Desires](./section-01-combining-beliefs-and-desires.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Why Utility Functions?

[Section 16.1](./section-01-combining-beliefs-and-desires.md) introduced **maximum expected utility (MEU)**. But where do utilities come from? We cannot simply use dollars, raw accuracy, or arbitrary scores - preferences must be **coherent** to justify EU maximization.

**Utility theory** answers: under what conditions can we represent preferences with a real-valued function $U$ such that the agent prefers lottery $L_1$ to $L_2$ if and only if $\text{EU}(L_1) > \text{EU}(L_2)$?

> **Readable form:** A utility function is a consistent numerical score for outcomes. If your preferences follow certain "sanity rules" (axioms), there exists a utility scale where maximizing expected utility matches your preferences.

---

## Lotteries and Preferences

A **lottery** $L = [p_1, o_1; p_2, o_2; \ldots; p_n, o_n]$ is a probability distribution over outcomes $o_i$ with $\sum_i p_i = 1$.

Write $L_1 \succ L_2$ if the agent **strictly prefers** $L_1$ to $L_2$, and $L_1 \sim L_2$ if **indifferent**.

| Notation | Meaning |
|----------|---------|
| $A \succ B$ | Prefer $A$ to $B$ |
| $A \sim B$ | Indifferent |
| $A \succeq B$ | Prefer or indifferent |

---

## Axioms of Rational Preference

von Neumann and Morgenstern (1944) showed that if preferences satisfy:

| Axiom | Informal statement |
|-------|-------------------|
| **Orderability** | For any $A, B$: either $A \succeq B$ or $B \succeq A$ (or both) |
| **Continuity** | If $A \succ B \succ C$, there exists $p$ such that $[p,A; 1-p,C] \sim B$ |
| **Substitutability** | If $A \sim B$, then $[p,A; q,C] \sim [p,B; q,C]$ |
| **Monotonicity** | If $A \succ B$ and $p > q$, then $[p,A; 1-p,B] \succ [q,A; 1-q,B]$ |
| **Decomposability** | Compound lotteries equivalent to flattened lotteries (no "fun of gambling" for its own sake) |

then there exists a utility function $U$ such that:

$$
L_1 \succeq L_2 \iff \sum_i p_i U(o_i) \geq \sum_i q_i U(o_i)
$$
> **Readable form:** If you can rank options consistently, can trade off probabilities smoothly, and don't care about multi-stage randomness beyond final outcomes, your preferences can be encoded as expected utility.

---

## Constructing Utilities: Standard Gamble

To assess $U(o)$ for outcome $o$:

1. Fix anchor utilities: $U(\text{best}) = u_\top$, $U(\text{worst}) = u_\bot$ (often 100 and 0)
2. For outcome $o$, find probability $p$ where the agent is indifferent between:
   - **Certain** $o$
   - Lottery $[p, \text{best}; 1-p, \text{worst}]$
3. Then $U(o) = p \cdot u_\top + (1-p) \cdot u_\bot$

**Example:** Best = "cure patient" ($U=100$), worst = "patient dies" ($U=0$). If "partial recovery" is indifferent to $[0.6, \text{cure}; 0.4, \text{death}]$, then $U(\text{partial}) = 60$.

---

## Utility Is Unique Up to Positive Affine Transform

If $U$ represents preferences, so does $U' = aU + b$ for any $a > 0$ and any $b$.

\sum p_i U'(o_i) = a \sum p_i U(o_i) + b
$$

Since $b$ is constant across lotteries and $a > 0$ preserves ordering, **only ratios of utility differences matter** for decisions - not absolute zero or unit.

$$
> **Readable form:** You can rescale utilities (multiply by a positive number, add a constant) without changing which action MEU recommends.

---

## Risk Attitudes

Consider initial wealth $W$ and a gamble: 50% win \$100, 50% lose \$100.

| Attitude | Shape of $U(W)$ | Behavior |
|----------|-----------------|----------|
| **Risk-neutral** | Linear: $U(W) = W$ | Accept fair gambles; MEU = EMV |
| **Risk-averse** | Concave: $\frac{d^2U}{dW^2} < 0$ | Reject many fair gambles; buy insurance |
| **Risk-seeking** | Convex | Accept unfair gambles; buy lottery tickets |

**Insurance example:** Risk-averse agent pays premium $> 0$ to avoid rare large loss - even though insurer profits on average. Expected monetary value of insurance is negative for buyer; expected **utility** is positive.

$$
U(W - \text{premium}) > \mathbb{E}[U(W - \text{loss})]
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

---

## The St. Petersburg Paradox

A coin is flipped until tails. Payoff = $2^k$ dollars if tails on flip $k$. Expected monetary value is **infinite**:

$$
\mathbb{E}[\text{payoff}] = \sum_{k=1}^{\infty} \frac{1}{2^k} \cdot 2^k = \infty
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

Yet people pay only a few dollars to play. Bernoulli resolved this with **logarithmic utility** $U(W) = \log W$:

$$
\mathbb{E}[U] = \sum_{k=1}^{\infty} \frac{1}{2^k} \log(2^k) = 2 \log 2 \quad \text{(finite)}
$$
> **Readable form:** Infinite expected dollars doesn't mean infinite happiness. Diminishing marginal utility explains why we wouldn't pay a fortune for an unbounded gamble.

---

## Utility of Money: Typical Curve

For wealth $W$:

U(W) = \begin{cases}
W^{0.88} & \text{(empirical fits, simplified)} \\
\log W & \text{(classic Bernoulli)}
\end{cases}

| Wealth change | Risk-neutral value | Risk-averse utility gain |
|---------------|-------------------|--------------------------|
| +\$100 | +\$100 | Less than linear |
| −\$100 | −\$100 | More than linear (in magnitude) |

This asymmetry drives **loss aversion** in descriptive models ([Section 16.6](./section-06-human-biases.md)).

---

## Multi-Outcome Lotteries and EU

For action $a$ with uncertain outcomes:

$$
\text{EU}(a) = \sum_{s} P(s \mid e) \cdot U(\text{result}(a, s))
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

When the agent's action affects which state occurs, use $P(s \mid a, e)$ - the **causal** or **interventional** probability appropriate to the decision.

---

## Worked Example: Job Offer

Two job offers as lotteries over "career satisfaction" (assessed utilities):

| Outcome | $U$ |
|---------|-----|
| Dream career | 100 |
| Good steady job | 70 |
| Mediocre | 40 |
| Bad fit | 10 |

**Startup offer:** 20% dream, 50% good, 20% mediocre, 10% bad.

$$
\text{EU}_{\text{startup}} = 0.2(100) + 0.5(70) + 0.2(40) + 0.1(10) = 20 + 35 + 8 + 1 = 64
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

**Corporate offer:** 5% dream, 60% good, 30% mediocre, 5% bad.

$$
\text{EU}_{\text{corp}} = 5 + 42 + 12 + 0.5 = 59.5
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

MEU: startup (64 > 59.5), even though corporate has higher $P(\text{good})$.

---

## Assessing Utilities in Practice

| Method | Description | Use case |
|--------|-------------|----------|
| **Standard gamble** | Probability equivalence | Single-attribute outcomes |
| **Probability equivalent** | Certainty equivalent | Insurance, pricing |
| **Bidirectional comparison** | Pairwise ranking + consistency check | Group decisions |
| **Midpoint chaining** | Bisect utility intervals | Large outcome sets |

Challenges: **cognitive biases**, **inconsistent assessments**, **stakeholder disagreement**. [Section 16.5](./section-05-decision-analysis-process.md) addresses facilitation and sensitivity analysis.

---

## Utility vs. Reward in RL

[Chapter 17](../chapter-17-making-complex-decisions/README.md) uses **reward functions** $R(s, a, s')$ in MDPs. Rewards are utilities over state transitions, often discounted:

$$
U_{\text{trajectory}} = \sum_{t=0}^{\infty} \gamma^t R(s_t, a_t, s_{t+1})
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

Poor reward design (wrong utility scale or missing terms) causes **reward hacking** - optimal policy for wrong objective.

---

## Python: Certainty Equivalent

```python
import math

def eu_lottery(probs, utilities):
    return sum(p * u for p, u in zip(probs, utilities))

def certainty_equivalent(probs, utilities, U_inv):
    """U_inv maps utility back to monetary (or raw) value."""
    return U_inv(eu_lottery(probs, utilities))

# Log utility, 50-50 win/lose $100 from W=1000
W = 1000
U = lambda w: math.log(w)
U_inv = lambda u: math.exp(u)

probs = [0.5, 0.5]
wealth_outcomes = [W + 100, W - 100]
utilities = [U(w) for w in wealth_outcomes]
ce = certainty_equivalent(probs, utilities, U_inv)
# ce < W => risk-averse agent values gamble below status quo
```

---

## Common Mistakes

**Treating utility as probability.** Utilities can be negative and need not sum to 1.

**Comparing utilities across agents.** Affine uniqueness is per-agent; interpersonal comparison is contested.

**Using EMV for risk-averse stakeholders.** Financial ROI may mis-rank medical or safety decisions.

**Ignoring state-dependent utility.** $U(\text{wealth})$ may depend on health, context ([Section 16.7](./section-07-multiattribute-utility.md)).

---

## Reflection Questions

1. Which axiom would a "fan of suspense" violate by preferring two-stage gambles over equivalent one-stage gambles?
2. Show that risk-neutral agents maximize expected monetary value.
3. Construct a standard gamble to assess utility of "minor side effects" in a clinical trial.
4. Why is $U' = 2U + 5$ equivalent for MEU but $U' = U^2$ generally not?
5. How does concavity of $U$ relate to buying insurance?

---

**Previous:** [Section 16.1 - Combining Beliefs and Desires](./section-01-combining-beliefs-and-desires.md)  
**Next:** [Section 16.3 - Decision Networks](./section-03-decision-networks.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Sections 16.3-16.4. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. von Neumann, J., & Morgenstern, O. (1944). *Theory of Games and Economic Behavior*.
3. Keeney, R. L., & Raiffa, H. (1976). *Decisions with Multiple Objectives* - multi-attribute preview.
4. Pratt, J. W. (1964). Risk aversion in the small and in the large. *Econometrica*.
5. Bernoulli, D. (1738). Exposition of a new theory on the measurement of risk.

