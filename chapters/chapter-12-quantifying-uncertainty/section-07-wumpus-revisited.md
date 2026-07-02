# Section 12.7: Wumpus Revisited

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 12.7  
> **Previous:** [Section 12.6 - Naive Bayes Classifier](./section-06-naive-bayes-classifier.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## From Boolean to Probabilistic Grids

[Chapter 04](../chapter-04-search-complex-environments/README.md) introduced the Wumpus world: pits, breeze percepts, partial observability. Logical agents mark squares as **safe**, **possible**, or **unknown**. Probabilistic agents assign:

$$
P(\text{pit}_{i,j} \mid \text{percepts})
$$
> **Readable form:** Estimate the probability that cell $(i,j)$ contains a pit after observing the percepts so far.

for each cell - a **belief map** over the grid.

> **Readable form:** Instead of yes/no pit, each square gets a danger probability updated as you explore.

---

## Pit Detection Model

If a pit exists in adjacent cell, breeze is sensed with high probability. AIMA uses a simple model: each adjacent pit independently causes breeze with probability $p_b$ (e.g., 0.5), combined appropriately. With no adjacent pits, no breeze.

Given breeze at $(2,2)$, cells $(2,3), (3,2), (1,2), (2,1)$ are **candidate pits**. Logical agent: at least one pit among candidates. Probabilistic agent: distributes belief among configurations consistent with percepts.

---

## Enumerating Configurations

For $n$ candidate cells, $2^n$ pit configurations exist. Prior: each cell has independent pit probability $p_{\text{prior}}$ (e.g., 0.2). Likelihood of percepts given a configuration: product of sensor model terms.

$$
P(\text{config} \mid e) \propto P(e \mid \text{config}) \, P(\text{config})
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Normalize over configurations. For cell $(i,j)$:

$$
P(\text{pit}_{i,j} \mid e) = \sum_{\text{config}: \text{pit at }(i,j)} P(\text{config} \mid e)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

---

## Safe vs. Optimal Action

Logical agent might require **certainty** of safety before entering a square. Probabilistic **decision-theoretic** agent ([Section 12.1](./section-01-acting-under-uncertainty.md)) enters if expected utility favors exploration:

$$
EU(\text{enter}) = P(\text{safe}) \cdot U_{\text{gold}} + P(\text{pit}) \cdot U_{\text{death}}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Low pit probability may justify risk when gold utility is high - links to [Chapter 16](../chapter-16-making-simple-decisions/README.md).

```python
def pit_prob_cell(configs, posteriors):
    '''Marginal pit probability for one cell from configuration posteriors.'''
    return sum(p for cfg, p in zip(configs, posteriors) if cfg.has_pit)

def should_enter(p_safe, u_gold, u_death):
    eu = p_safe * u_gold + (1 - p_safe) * u_death
    return eu > 0  # simplified threshold
```

---

## Comparison to Logical Wumpus

| Aspect | Logical (Chapter 04) | Probabilistic (Ch. 12.7) |
|--------|---------------------|--------------------------|
| Cell label | Safe / possible / unknown | $P(\text{pit}) \in [0,1]$ |
| New percept | Deductive inference | Bayesian update |
| Action choice | Conservative rules | Expected utility |
| Scalability | CSP/backtracking | Exponential in candidates |

[Chapter 13](../chapter-13-probabilistic-reasoning/README.md) replaces enumeration with **Bayesian networks** over grid variables.

---

## Multi-Square Updates

As the agent moves, new breeze or stench percepts **eliminate** inconsistent configurations. Each elimination step renormalizes beliefs - analogous to **constraint propagation** in CSPs ([Chapter 06](../chapter-06-constraint-satisfaction/README.md)) but soft rather than hard.

---

## Wumpus vs. Minesweeper

Minesweeper players intuitively assign pit probabilities. Expert play combines logic (forced safe moves) with probability (best expected value when guessing). The Wumpus exercise makes this explicit for AI agents.

---

## Implementation Sketch

```python
from itertools import product

def configs(candidates):
    for bits in product([0, 1], repeat=len(candidates)):
        yield dict(zip(candidates, bits))

def posterior_over_configs(percept_model, prior_p, candidates):
    unnorm = []
    for cfg in configs(candidates):
        p = prior_p ** sum(cfg.values()) * (1-prior_p) ** (len(cfg)-sum(cfg.values()))
        p *= percept_model.likelihood(cfg)
        unnorm.append(p)
    z = sum(unnorm)
    return [p/z for p in unnorm]
```

---

## When Exact Inference Fails

Large grids with many unknown cells make $2^n$ enumeration impossible. Approximate methods - loopy belief propagation, sampling ([Chapter 13](../chapter-13-probabilistic-reasoning/README.md)) - scale better.

---

## Historical Note

Early AI used **ad hoc certainty factors** in systems like MYCIN. Wumpus probabilistic analysis shows why principled probabilities replace fudge factors when combining multiple pieces of evidence.

---

## Lab Extension

Extend the enumeration code to print marginal pit probability for each candidate cell after each percept. Compare greedy EU-based movement against always waiting for certainty.

---

## Key Takeaways

1. Wumpus percepts constrain pit configurations; Bayes assigns posterior over configs.
2. Marginal pit probability guides safer exploration than pure logic.
3. Decision theory chooses actions, not just labels.
4. Exact enumeration is feasible only for small candidate sets.
5. This chapter bridges search (Chapter 04) and graphical models (Chapter 13).



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

1. Why does a breeze percept not pin a pit to a single adjacent cell?
2. How does EU-based movement differ from 'enter only if safe'?
3. What is the complexity of exact pit configuration enumeration?
4. How would a BN represent Wumpus variables (preview)?
5. Connect Wumpus belief maps to robot localization.

---

**Next:** [Chapter 13 - Probabilistic Reasoning](../chapter-13-probabilistic-reasoning/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python)
3. Russell & Norvig aima-python: wumpus_world, probability chapters.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Belief map** | Grid of P(hidden state | percepts) over environment cells. |
| **Configuration** | A complete assignment of pit presence to candidate cells. |
| **Sensor model** | P(percept | local world configuration); links hidden to observed. |
| **Marginal probability** | P(single variable) obtained by summing over other variables. |
