# Section 16.1: Combining Beliefs and Desires

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 16.1-16.2  
> **Previous:** [Chapter 15 - Probabilistic Programming](../chapter-15-probabilistic-programming/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Beliefs to Actions

[Chapter 13](../chapter-13-probabilistic-reasoning/README.md) taught agents how to **update beliefs** under uncertainty using probability and Bayesian networks. But knowing $P(\text{disease} \mid \text{symptoms})$ is not enough - a medical agent must also decide whether to order a test, prescribe treatment, or wait.

**Decision theory** combines two ingredients:

| Ingredient | Represents | Formal tool |
|------------|------------|-------------|
| **Beliefs** | What the world is probably like | Probabilities $P(s \mid e)$ |
| **Desires** | What outcomes the agent prefers | Utilities $U(s)$ |

> **Readable form:** Probability tells you how likely each outcome is; utility tells you how much you care about each outcome. Rational action maximizes **expected** utility, weighting outcomes by their probability.

This is the bridge from **reasoning** (Part IV of AIMA) to **acting** - the same [rational agent](../../GLOSSARY.md#rational-agent) framework from [Chapter 02](../chapter-02-intelligent-agents/README.md), now with explicit preferences.

---

## Decisions Under Uncertainty

A **decision problem** under uncertainty has:

1. **Evidence** $e$ - what the agent has observed
2. **Actions** $A = \{a_1, a_2, \ldots\}$ - choices available now
3. **Outcomes** - consequences of each action, possibly uncertain
4. **Utilities** $U(o)$ - preference over outcomes $o$

The agent chooses the action that maximizes **expected utility**:

$$
\text{EU}(a \mid e) = \sum_o P(o \mid a, e) \cdot U(o)
$$

$$
a^* = \arg\max_a \text{EU}(a \mid e)
$$
> **Readable form:** For each possible action, multiply each outcome's utility by how likely that outcome is (given you take that action and your evidence), then add them up. Pick the action with the highest total.

This is the **maximum expected utility (MEU)** principle - the normative standard for decisions under uncertainty when preferences satisfy certain axioms ([Section 16.2](./section-02-utility-theory.md)).

---

## A Simple Medical Example

A doctor must decide: **treat now** or **order a blood test first**.

| Outcome | Utility $U$ |
|---------|-------------|
| Patient healthy, no treatment | 0 |
| Patient sick, treated successfully | +100 |
| Patient sick, untreated (worsens) | −200 |
| False alarm (healthy, unnecessary treatment) | −20 |
| Test reveals correct diagnosis | +5 (information value) |

Suppose prior $P(\text{sick}) = 0.3$. Treating immediately:

$$
\text{EU}(\text{treat}) = 0.3 \times 100 + 0.7 \times (-20) = 30 - 14 = 16
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

Waiting without test might yield lower expected utility if deterioration is likely. The MEU framework forces explicit probability and preference numbers - uncomfortable but clarifying.

---

## Why Not Maximize Expected Monetary Value?

Money is convenient but not always linear in happiness. Consider a **fair coin flip**:

- Heads: win \$1000
- Tails: lose \$1000

Expected monetary value = \$0. Many people refuse this bet despite positive EMV for asymmetric bets. **Utility functions** capture diminishing returns: the pain of losing \$1000 exceeds the pleasure of gaining \$1000 for risk-averse agents.

$$
U(\text{wealth}) \neq \text{wealth} \quad \text{in general}
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

We develop this formally in [Section 16.2](./section-02-utility-theory.md).

---

## MEU vs. Other Decision Rules

| Rule | Criterion | When used |
|------|-----------|-----------|
| **MEU** | Maximize $\sum P \cdot U$ | Normative default; axioms satisfied |
| **Maximax** | Maximize best-case outcome | Optimistic; ignores risk |
| **Maximin** | Maximize worst-case outcome | Extremely cautious |
| **Minimize regret** | Minimize max opportunity loss | Sensitivity to alternatives |

MEU reduces to **maximin** when utility is 0/1 (success/failure only) and probabilities are unknown. MEU reduces to **deterministic optimization** when outcomes are certain.

> **Readable form:** MEU is the "gold standard" when you have probabilities and a coherent utility scale. Other rules are shortcuts for special cases or when probabilities are missing.

---

## Connection to Bayesian Networks

In [Chapter 13](../chapter-13-probabilistic-reasoning/README.md), Bayesian networks compute $P(X \mid \mathbf{e})$ for query variables $X$. Decision theory adds:

- **Decision nodes** - actions the agent controls
- **Utility nodes** - value of outcome configurations

[Section 16.3](./section-03-decision-networks.md) unifies these as **decision networks** (influence diagrams).

```
Evidence → Chance variables → Utility
              ↑
         Decision nodes
```

---

## The Decision-Making Pipeline

```
1. Model the problem (variables, dependencies)
2. Assess probabilities (data, experts, BN inference)
3. Assess utilities (preferences, stakeholders)
4. Compute EU for each action
5. Choose MEU action
6. (Optional) Compute value of information - is more evidence worth it?
```

Step 6 is [Section 16.4](./section-04-value-of-information.md). Steps 1-5 are structured in [Section 16.5](./section-05-decision-analysis-process.md).

---

## Worked Example: Umbrella Decision

You leave for work. Evidence: weather forecast says 40% rain. Actions: {take umbrella, leave umbrella}.

| Outcome | $P(\cdot \mid \text{action})$ | $U$ |
|---------|-------------------------------|-----|
| Rain, have umbrella | - | 0 (dry, slight inconvenience) |
| Rain, no umbrella | - | −10 (wet) |
| No rain, have umbrella | - | −2 (carrying unnecessarily) |
| No rain, no umbrella | - | 0 |

**Take umbrella:**

$$
\text{EU} = 0.4 \times 0 + 0.6 \times (-2) = -1.2
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

**Leave umbrella:**

$$
\text{EU} = 0.4 \times (-10) + 0.6 \times 0 = -4.0
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

MEU: take umbrella ($-1.2 > -4.0$). Small utilities, clear conclusion.

---

## Python Sketch: MEU Calculator

```python
def maximum_expected_utility(actions, outcomes, prob_fn, utility):
    """
    actions: list of action names
    outcomes: list of outcome names
    prob_fn(a, o) -> P(o | a)  (sums to 1 over o for each a)
    utility(o) -> U(o)
    """
    best_a, best_eu = None, float("-inf")
    for a in actions:
        eu = sum(prob_fn(a, o) * utility(o) for o in outcomes)
        if eu > best_eu:
            best_a, best_eu = a, eu
    return best_a, best_eu

# Umbrella example
actions = ["umbrella", "no_umbrella"]
outcomes = ["rain", "no_rain"]
P_rain = 0.4

def prob(a, o):
    if a == "umbrella":
        return P_rain if o == "rain" else 1 - P_rain
    return P_rain if o == "rain" else 1 - P_rain

def U(o):
  # simplified: combine rain/no-rain with implicit action in full model
    return {"rain": -10, "no_rain": 0}[o]

# Full model would condition utility on (action, outcome) pairs
```

> **Readable form:** Loop over actions, compute weighted average utility for each, return the winner.

---

## Normative vs. Descriptive

| Perspective | Question | Chapter coverage |
|-------------|----------|-----------------|
| **Normative** | What *should* a rational agent do? | MEU, axioms, decision networks |
| **Descriptive** | What do humans *actually* do? | [Section 16.6](./section-06-human-biases.md) |

AI systems often implement normative rules; human-in-the-loop systems must account for descriptive deviations (framing, loss aversion).

---

## Common Mistakes

**Confusing probability with utility.** High probability of a bad outcome does not automatically mean avoid an action - if all actions are risky, pick the least bad in EU terms.

**Using monetary values directly.** \$1M to a billionaire vs. a student has different utility impact.

**Ignoring opportunity cost.** "Do nothing" is an action with its own EU.

**Double-counting evidence.** Probabilities in $\text{EU}(a \mid e)$ must be conditioned on the same evidence $e$ used for all actions.

---

## Link to Sequential Decisions

This chapter covers **single-stage** decisions: observe, act once, receive outcome. [Chapter 17](../chapter-17-making-complex-decisions/README.md) extends to **Markov decision processes** where actions unfold over time with rewards and discounting.

| Single-stage (Ch. 16) | Sequential (Ch. 17) |
|-----------------------|---------------------|
| One decision node | Policy $\pi(s)$ for all states |
| Utility at outcome | Reward at each step |
| $P(o \mid a, e)$ | $P(s' \mid s, a)$ transition model |

---

## Reflection Questions

1. Why does MEU require both probabilities and utilities? What happens if either is missing?
2. Reformulate the umbrella example with utilities where leaving the umbrella is MEU-optimal. What changed?
3. How does MEU relate to the performance measure of a rational agent in [Chapter 02](../chapter-02-intelligent-agents/README.md)?
4. When might a maximin rule be defensible despite violating MEU?
5. What additional variables would you add to the medical example to make it a realistic decision network?

---

**Next:** [Section 16.2 - Utility Theory](./section-02-utility-theory.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Sections 16.1-16.2. [AIMA book site](https://aima.cs.berkeley.edu/)
2. von Neumann, J., & Morgenstern, O. (1944). *Theory of Games and Economic Behavior* - foundation of expected utility.
3. Howard, R. A. - decision analysis and influence diagrams. [INFORMS history](https://www.informs.org/)
4. Kahneman, D., & Tversky, A. - prospect theory (descriptive contrast). [Nobel lecture materials](https://www.nobelprize.org/)
5. AIMA Python: `decision_theory.py` in [aima-python](https://github.com/aimacode/aima-python)
