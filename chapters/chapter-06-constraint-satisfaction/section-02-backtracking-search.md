# Section 06.2: Backtracking Search

> **Source:** Russell & Norvig, *AIMA* (4th ed.), Chapter 6
> **Previous:** [Section 06.1](./section-01-csp-formulation.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

Naive backtracking tries every value in order. For CSPs we can do better - but first understand the baseline search tree over partial assignments.

---

## Backtracking Algorithm

CSP backtracking is depth-first search over **partial assignments**. At each step, assign a value to an unassigned variable; if consistent, recurse; else backtrack.

```
BACKTRACKING-SEARCH(csp):
    return BACKTRACK({}, csp)

BACKTRACK(assignment, csp):
    if complete(assignment): return assignment
    var ← SELECT-UNASSIGNED-VARIABLE(csp, assignment)
    for value in ORDER-DOMAIN-VALUES(var, assignment, csp):
        if consistent(assignment ∪ {var=value}):
            result ← BACKTRACK(assignment ∪ {var=value}, csp)
            if result ≠ failure: return result
    return failure
```

---

## Constraint Graphs

The **constraint graph** reveals structure. High **degree** variables participate in many constraints - they are good candidates for early assignment (degree heuristic). Disconnected components decompose into independent subproblems.

---

## Complexity

General CSP solving is **NP-complete**. But structure-aware algorithms solve many practical instances efficiently. [Chapter 03](../chapter-03-solving-problems-by-searching/README.md) search gave us the skeleton; CSP adds domain-specific pruning.

---


## Key Equations


$$
O(b^d) \text{ naive search} \rightarrow \text{pruned by consistency}
$$
> **Readable form:** Constraint checking eliminates branches that atomic search would still explore.


---

## Implementation

```python
def backtrack(assignment, csp):
    if len(assignment) == len(csp.variables):
        return assignment
    var = next(v for v in csp.variables if v not in assignment)
    for value in csp.domains[var]:
        assignment[var] = value
        if csp.is_consistent(assignment):
            result = backtrack(assignment, csp)
            if result: return result
        del assignment[var]
    return None
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
| **backtracking** | Concept from Chapter 6 |
| **partial assignment** | Concept from Chapter 6 |
| **constraint graph** | Concept from Chapter 6 |
| **branching factor** | Concept from Chapter 6 |

---

## Key Takeaways

1. **Backtracking Search** covers Naive backtracking.
2. Formalism connects to implementable algorithms.
3. Builds toward chapter lab and later chapters.

---

## Reflection Questions

1. Explain **backtracking search** in your own words.
2. What assumptions does Chapter 6 make?
3. How verify on a toy example?
4. Real-world AI applications?
5. Which later chapter depends on this?

---

**Previous:** [Section 06.1](./section-01-csp-formulation.md) · **Next:** [Section 06.3](./section-03-constraint-propagation.md)

---

## References

1. Russell & Norvig (2020). *AIMA* (4th ed.). https://aima.cs.berkeley.edu/
2. aima-python: https://github.com/aimacode/aima-python
3. Berkeley CS188: https://inst.eecs.berkeley.edu/~cs188/
4. MIT 6.034 OCW.
5. Stanford AI Index: https://aiindex.stanford.edu/

---

## Worked Example

Extended notes on **Backtracking Search** from Chapter 6.

---

## Common Pitfalls

Extended notes on **Backtracking Search** from Chapter 6.

---

## Historical Note

Extended notes on **Backtracking Search** from Chapter 6.

---

## Further Reading

Extended notes on **Backtracking Search** from Chapter 6.




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
