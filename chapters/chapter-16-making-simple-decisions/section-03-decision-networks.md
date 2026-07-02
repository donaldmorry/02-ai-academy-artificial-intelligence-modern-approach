# Section 16.3: Decision Networks

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 16.5-16.6  
> **Previous:** [Section 16.2 - Utility Theory](./section-02-utility-theory.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Bayesian Networks to Decision Networks

[Chapter 13](../chapter-13-probabilistic-reasoning/README.md) introduced **Bayesian networks (BNs)** - directed acyclic graphs whose nodes are random variables and whose edges encode conditional dependencies. BNs answer: *What do I believe given evidence?*

**Decision networks** (also called **influence diagrams**) extend BNs with:

| Node type | Shape (convention) | Meaning |
|-----------|-------------------|---------|
| **Chance** | Oval / circle | Random variable, as in BNs |
| **Decision** | Rectangle | Action the agent chooses |
| **Utility** | Diamond / hexagon | Value function over variable configurations |

> **Readable form:** A decision network is a flowchart of uncertainty and choices. Chance nodes are things you don't control; decision nodes are levers you pull; the utility node scores how happy you are with the final situation.

This unifies [Section 16.1](./section-01-combining-beliefs-and-desires.md) MEU computation with the structured models from probabilistic reasoning.

---

## Anatomy of a Decision Network

A decision network specifies:

1. **Chance nodes** $X_i$ with conditional probability tables $P(X_i \mid \text{Parents}(X_i))$
2. **Decision nodes** $D_j$ - no CPT; the agent selects values
3. **Utility node** $U$ with function $U(\text{Parents}(U))$ mapping outcome configurations to real numbers

**Information arcs** point *into* decision nodes: a decision can depend only on variables known when the decision is made. No arc from a future decision into an earlier chance node without proper temporal ordering.

**Evaluating** a decision network:

1. For each assignment to decision nodes (a **policy** for the diagram's decisions)
2. Compute expected utility at the utility node using BN inference
3. Choose the decision assignment maximizing expected utility

---

## MEU in a Decision Network

For a single decision node $D$ with parents $\text{Pa}(D)$ (information available at decision time):

$$
\text{EU}(d \mid \mathbf{e}, \text{pa}(D)) = \sum_{\mathbf{y}} P(\mathbf{y} \mid d, \mathbf{e}, \text{pa}(D)) \cdot U(\mathbf{y}, d)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

where $\mathbf{y}$ ranges over values of chance nodes not yet observed, and $U$ depends on the relevant outcome variables.

$$
d^* = \arg\max_d \text{EU}(d \mid \mathbf{e}, \text{pa}(D))
$$
> **Readable form:** For each possible action, average the utility over uncertain outcomes (given what you know), then pick the action with the highest average.

For **multiple decision nodes**, we need a **policy** $\pi$ specifying an action for each decision node given its information. Optimal policy maximizes global expected utility - solved by **backward induction** or specialized influence-diagram algorithms.

---

## Classic Example: Oil Drilling

Variables:

- **Oil** - {present, absent} (unknown)
- **Seismic** test result - {positive, negative} (optional observation)
- **Drill** - decision {yes, no}
- **Utility** - profits minus costs

Structure:

```
Oil ──→ Seismic
Oil ──→ Utility
Drill ──→ Utility
Seismic ──→ Drill   (test result informs drill decision)
```

| Scenario | Expected profit logic |
|----------|----------------------|
| Drill without test | High variance; MEU balances $P(\text{oil})$ vs. drilling cost |
| Test then drill | Pay test cost; update $P(\text{oil} \mid \text{seismic})$; drill if EU positive |

The optimal policy might be: *if seismic positive, drill; if negative, don't drill* - a **contingent plan** rather than a fixed action.

---

## Comparison: BN vs. Decision Network

| Feature | Bayesian network | Decision network |
|---------|------------------|------------------|
| Purpose | Belief updating | Action selection |
| Agent control | None | Decision nodes |
| Output | $P(X \mid \mathbf{e})$ | Optimal decisions + EU |
| Evaluation | Variable elimination, sampling | MEU + inference |

Every decision network has an underlying BN if decision nodes are replaced by chance nodes with known policies - but the decision network exposes **what to optimize**.

---

## Multi-Stage Decisions

When decisions are sequential (test → treat), the diagram has multiple decision nodes $D_1, D_2, \ldots$ ordered in time.

**Policy for $D_2$** may depend on outcomes observed between $D_1$ and $D_2$. The optimal policy at $D_1$ must anticipate future decisions - **dynamic programming** on the diagram.

[Chapter 17](../chapter-17-making-complex-decisions/README.md) generalizes this to MDPs with many states; decision networks handle **sparse**, **structured** domains with explicit variables.

---

## Solving Single-Decision Networks

Algorithm sketch:

1. Instantiate evidence $\mathbf{e}$
2. For each value $d$ of decision node $D$:
   - Set $D = d$
   - Use BN inference to compute $P(\cdot \mid d, \mathbf{e})$ for variables affecting $U$
   - Compute $\text{EU}(d) = \mathbb{E}[U \mid d, \mathbf{e}]$
3. Return $d^* = \arg\max_d \text{EU}(d)$

```python
def meu_single_decision(decision_var, values, evidence, bn, utility_fn):
    best_d, best_eu = None, float("-inf")
    for d in values:
        eu = 0.0
        # enumerate or sample outcomes affecting utility
        for assignment in bn.relevant_assignments(decision_var, d, evidence):
            p = bn.probability(assignment, evidence={**evidence, decision_var: d})
            u = utility_fn(assignment)
            eu += p * u
        if eu > best_eu:
            best_d, best_eu = d, eu
    return best_d, best_eu
```

For large networks, **variable elimination** avoids full enumeration - same machinery as BN inference with max operations at decision nodes.

---

## Worked Example: Treatment Decision

Chance nodes: **Disease** $\in \{\text{yes}, \text{no}\}$, $P(\text{yes}) = 0.02$.

Decision: **Treat** $\in \{\text{yes}, \text{no}\}$.

Utilities (simplified):

| Disease | Treat | $U$ |
|---------|-------|-----|
| yes | yes | 90 |
| yes | no | −100 |
| no | yes | −10 |
| no | no | 0 |

**EU(treat yes):**

$$
0.02 \times 90 + 0.98 \times (-10) = 1.8 - 9.8 = -8.0
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

**EU(treat no):**

$$
0.02 \times (-100) + 0.98 \times 0 = -2.0
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

MEU: **do not treat** ($-2.0 > -8.0$) - rare disease makes treatment's side effects dominate. Adding a **Test** chance node can flip the optimal policy ([Section 16.4](./section-04-value-of-information.md)).

---

## Building Decision Networks: Best Practices

| Step | Action |
|------|--------|
| 1 | Identify decisions, uncertainties, and objectives |
| 2 | Order decisions temporally; add information arcs |
| 3 | Specify CPTs from data or experts |
| 4 | Elicit utilities ([Section 16.2](./section-02-utility-theory.md)) |
| 5 | Validate with sensitivity analysis ([Section 16.5](./section-05-decision-analysis-process.md)) |
| 6 | Solve and document assumptions |

Avoid **illegal information arcs** - decisions cannot depend on variables not yet observed. Avoid **redundant chance nodes** that duplicate the same uncertainty.

---

## Connection to MDPs

| Decision network | MDP |
|------------------|-----|
| Explicit variables | State abstraction |
| Few decisions | Many time steps |
| Structured CPTs | Transition matrix $P(s' \mid s,a)$ |
| Influence diagram algorithms | Value iteration |

Decision networks excel when the domain is **factored** (medical, business). MDPs excel when the state is **uniform** (grid worlds, robotics discretization).

---

## Python: Influence Diagram Libraries

Production tools: **GeNIe/Smile**, **OpenMarkov**, **pgmpy** (Python). Conceptual workflow in pgmpy:

```python
# Conceptual - API varies by version
# 1. Build BN structure with decision/utility extensions
# 2. Set CPTs and utility table
# 3. Call inference routine for MEU
```

AIMA Python (`decision_theory.py`, `decision_networks.py`) includes textbook examples for oil drilling and medical decisions.

---

## Common Mistakes

**Forgetting information arcs.** Letting a decision depend on future test results that haven't been observed yet inflates EU incorrectly.

**Confusing posterior with prior.** CPTs for chance nodes are assessed before decisions; after evidence, update with inference.

**Single utility for multiple stakeholders.** Different parties may need separate utility nodes or negotiated weights ([Section 16.7](./section-07-multiattribute-utility.md)).

**Omitting "do nothing".** Every decision node should include status quo if realistic.

---

## Reflection Questions

1. Draw a decision network for "umbrella" with forecast as chance node and "take umbrella" as decision. What is the information arc?
2. Why does adding a decision node change what we compute, if the BN structure is otherwise the same?
3. When is a decision network preferable to a full MDP formulation?
4. How does backward induction on a multi-decision diagram relate to dynamic programming?
5. In the treatment example, what prior disease probability would make treatment MEU-optimal?

---

**Previous:** [Section 16.2 - Utility Theory](./section-02-utility-theory.md)  
**Next:** [Section 16.4 - Value of Information](./section-04-value-of-information.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Sections 16.5-16.6. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Howard, R. A., & Matheson, J. E. - influence diagrams. *Decision Analysis*.
3. Shachter, R. D. - evaluating influence diagrams. *Operations Research*.
4. Pearl, J. - probabilistic and decision-theoretic graphical models.
5. AIMA Python: [aima-python/decision_theory.py](https://github.com/aimacode/aima-python)
