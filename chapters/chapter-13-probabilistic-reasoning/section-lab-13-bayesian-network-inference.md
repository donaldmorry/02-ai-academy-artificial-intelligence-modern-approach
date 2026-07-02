# Lab 13: Bayesian Network Inference

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 13  
> **Prerequisites:** Sections 13.1-13.6  
> **Estimated time:** 4-6 hours  
> **Tools:** Python 3.10+, optional NumPy  
> **Assessment:** [Shared Assessment Appendix](../../ASSESSMENT_APPENDIX.md)

---

## Lab Objective

Build a small Bayesian network and answer diagnostic queries with enumeration or variable elimination. The lab emphasizes factorization, evidence conditioning, normalization, and explaining away.

By the end, you should be able to:

1. Encode a Bayesian network with Boolean variables.
2. Compute joint probabilities from parent assignments.
3. Condition on evidence and normalize a posterior.
4. Compare prior and posterior beliefs.
5. Explain why exact inference scales poorly with network width.

---

## Part A: Burglary Network

Use the classic AIMA-style network:

| Variable | Parents | Meaning |
|----------|---------|---------|
| `B` | none | Burglary |
| `E` | none | Earthquake |
| `A` | `B,E` | Alarm |
| `J` | `A` | John calls |
| `M` | `A` | Mary calls |

Starter CPTs:

```python
P_B = {True: 0.001, False: 0.999}
P_E = {True: 0.002, False: 0.998}
P_A = {
    (True, True): 0.95,
    (True, False): 0.94,
    (False, True): 0.29,
    (False, False): 0.001,
}
P_J = {True: 0.90, False: 0.05}  # keyed by A=True/False
P_M = {True: 0.70, False: 0.01}
```

---

## Part B: Joint Probability

```python
def p_alarm(a, b, e):
    p = P_A[(b, e)]
    return p if a else 1 - p


def p_call(call, alarm, table):
    p = table[alarm]
    return p if call else 1 - p


def joint(assign):
    b, e, a, j, m = [assign[k] for k in "BEAJM"]
    return (
        P_B[b]
        * P_E[e]
        * p_alarm(a, b, e)
        * p_call(j, a, P_J)
        * p_call(m, a, P_M)
    )
```

Check that the full joint sums to approximately 1.

---

## Part C: Enumeration Query

Compute `P(B | J=True, M=True)`.

```python
import itertools


def all_assignments():
    for values in itertools.product([False, True], repeat=5):
        yield dict(zip("BEAJM", values))


def query_burglary(j=True, m=True):
    totals = {True: 0.0, False: 0.0}
    for assignment in all_assignments():
        if assignment["J"] == j and assignment["M"] == m:
            totals[assignment["B"]] += joint(assignment)
    z = totals[True] + totals[False]
    return {k: v / z for k, v in totals.items()}
```

Record the posterior and compare it with the prior `P(B=True)`.

---

## Part D: Explaining Away

Run two more queries:

1. `P(B | J=True, M=True)`
2. `P(B | J=True, M=True, E=True)`

The second should reduce burglary probability because the earthquake explains part of the alarm evidence.

Deliver a table:

| Query | Posterior `P(B=True)` | Explanation |
|-------|-----------------------|-------------|
| Prior | | |
| `J,M` | | |
| `J,M,E` | | |

---

## Optional Extension: Variable Elimination

Implement factors as dictionaries keyed by tuples. Eliminate hidden variables in the order `M`, `J`, `A`, `E` or a different order and compare intermediate factor sizes.

```python
def normalize(dist):
    total = sum(dist.values())
    return {k: v / total for k, v in dist.items()}
```

The point of the extension is not speed on five variables; it is seeing why factor size, not the total joint table alone, controls exact inference cost.

---

## Deliverables

- Code for joint probability and enumeration query.
- Posterior table for the three queries above.
- One paragraph explaining explaining away.
- One note about which hidden variable would be expensive to eliminate in a wider network.

---

## Reflection

1. Why must posterior probabilities be normalized?
2. What independence assumptions make the network compact?
3. Why is `P(B | J,M,E)` lower than `P(B | J,M)`?
4. When would approximate inference be preferable?

---

## References

- Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 13.
- [Chapter 13 README](./README.md)
