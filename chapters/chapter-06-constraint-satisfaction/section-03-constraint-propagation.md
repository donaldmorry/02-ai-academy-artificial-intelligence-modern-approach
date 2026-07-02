# Section 06.3: Constraint Propagation

> **Source:** Russell & Norvig, *AIMA* (4th ed.), Chapter 6
> **Previous:** [Section 06.2](./section-02-backtracking-search.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

Forward checking and arc consistency detect dead ends **before** recursion wastes time - the key to practical CSP solvers.

---

## Forward Checking

When assigning $X_i = v$, **forward checking** removes $v$ from the domains of all unassigned neighbors of $X_i$. If any domain becomes empty, backtrack immediately.

---

## Arc Consistency

An arc $(X_i, X_j)$ is **arc-consistent** if for every value in $D_i$ there exists some value in $D_j$ satisfying the constraint. **AC-3** maintains arc consistency across the entire network.

---

## AC-3 Algorithm

```
AC-3(csp):
    queue ← all arcs (Xi, Xj) from constraints
    while queue not empty:
        (Xi, Xj) ← REMOVE(queue)
        if REVISE(csp, Xi, Xj):
            if Di is empty: return failure
            for each Xk neighbor of Xi (Xk ≠ Xj):
                ADD((Xk, Xi), queue)
    return success
```

---


## Key Equations


$$
\text{REVISE}: D_j \leftarrow \{v_j \in D_j : \exists v_i \in D_i, \text{consistent}(v_i, v_j)\}
$$
> **Readable form:** Revise removes values from Xj's domain that have no supporting value in Xi's domain.


---

## Implementation

```python
def revise(csp, xi, xj):
    revised = False
    for y in list(csp.domains[xj]):
        if not any(csp.constraint_ok(xi, x, xj, y) for x in csp.domains[xi]):
            csp.domains[xj].remove(y)
            revised = True
    return revised

def ac3(csp):
    from collections import deque
    queue = deque([(a,b) for a,b in csp.arcs] + [(b,a) for a,b in csp.arcs])
    while queue:
        xi, xj = queue.popleft()
        if revise(csp, xi, xj):
            if not csp.domains[xj]: return False
            for xk in csp.neighbors(xj):
                if xk != xi: queue.append((xk, xj))
    return True
```

---

## Connections

- **Combinatorial optimization** → Course 1 Chapter 03: classification feature selection analogies
- **SAT and planning** → Course 2 Chapter 11: planning as CSP/SAT
- **Graphical model structure** → Course 2 Chapter 13: Bayesian network topology

---

## Key Vocabulary

| Term | Meaning |
|------|---------|
| **forward checking** | Concept from Chapter 6 |
| **arc consistency** | Concept from Chapter 6 |
| **AC-3** | Concept from Chapter 6 |
| **domain pruning** | Concept from Chapter 6 |

---

## Key Takeaways

1. **Constraint Propagation** covers Forward checking.
2. Formalism connects to implementable algorithms.
3. Builds toward chapter lab and later chapters.

---

## Reflection Questions

1. Explain **constraint propagation** in your own words.
2. What assumptions does Chapter 6 make?
3. How verify on a toy example?
4. Real-world AI applications?
5. Which later chapter depends on this?

---

**Previous:** [Section 06.2](./section-02-backtracking-search.md) · **Next:** [Section 06.4](./section-04-heuristics-for-csp.md)

---

## References

1. Russell & Norvig (2020). *AIMA* (4th ed.). https://aima.cs.berkeley.edu/
2. aima-python: https://github.com/aimacode/aima-python
3. Berkeley CS188: https://inst.eecs.berkeley.edu/~cs188/
4. MIT 6.034 OCW.
5. Stanford AI Index: https://aiindex.stanford.edu/

---

## Worked Example

Extended notes on **Constraint Propagation** from Chapter 6.

---

## Common Pitfalls

Extended notes on **Constraint Propagation** from Chapter 6.

---

## Historical Note

Extended notes on **Constraint Propagation** from Chapter 6.

---

## Further Reading

Extended notes on **Constraint Propagation** from Chapter 6.




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
