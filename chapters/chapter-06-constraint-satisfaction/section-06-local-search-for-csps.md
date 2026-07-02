# Section 06.6: Local Search for CSPs

> **Source:** Russell & Norvig, *AIMA* (4th ed.), Chapter 6
> **Previous:** [Section 06.5](./section-05-csp-structure.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

When $n$ is huge, complete backtracking may fail. Min-conflicts local search solves million-variable CSPs in seconds.

---

## Min-Conflicts

Start with random complete assignment. Repeatedly pick a conflicted variable and reassign to the value minimizing constraint violations. Terminate when consistent or step limit reached.

---

## Performance

On $n$-queens with $n = 10{,}000$, min-conflicts often finds solutions in ~50 steps. Works because CSPs from real problems often have **deep solution clusters**.

---

## Hybrid Methods

Run propagation first, then local search. Or: random restarts + min-conflicts for hard instances.

---


## Key Equations


$$
\text{conflicts}(X_i, v) = |\{C : C \text{ violated by } X_i=v\}|
$$
> **Readable form:** Min-conflicts picks the value minimizing constraint violations for the chosen variable.


---

## Implementation

```python
import random

def min_conflicts(csp, max_steps=10000):
    assignment = {v: random.choice(csp.domains[v]) for v in csp.variables}
    for _ in range(max_steps):
        if csp.is_consistent(assignment): return assignment
        var = random.choice([v for v in csp.variables
                             if not csp.locally_consistent(var, assignment)])
        assignment[var] = min(csp.domains[var],
                              key=lambda val: csp.conflict_count(var, val, assignment))
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
| **min-conflicts** | Concept from Chapter 6 |
| **local search** | Concept from Chapter 6 |
| **random restart** | Concept from Chapter 6 |
| **plateau** | Concept from Chapter 6 |

---

## Key Takeaways

1. **Local Search for CSPs** covers Min-conflicts heuristic.
2. Formalism connects to implementable algorithms.
3. Builds toward chapter lab and later chapters.

---

## Reflection Questions

1. Explain **local search for csps** in your own words.
2. What assumptions does Chapter 6 make?
3. How verify on a toy example?
4. Real-world AI applications?
5. Which later chapter depends on this?

---

**Previous:** [Section 06.5](./section-05-csp-structure.md) · **Next:** [Section 06.7](./section-07-csp-applications.md)

---

## References

1. Russell & Norvig (2020). *AIMA* (4th ed.). https://aima.cs.berkeley.edu/
2. aima-python: https://github.com/aimacode/aima-python
3. Berkeley CS188: https://inst.eecs.berkeley.edu/~cs188/
4. MIT 6.034 OCW.
5. Stanford AI Index: https://aiindex.stanford.edu/

---

## Worked Example

Extended notes on **Local Search for CSPs** from Chapter 6.

---

## Common Pitfalls

Extended notes on **Local Search for CSPs** from Chapter 6.

---

## Historical Note

Extended notes on **Local Search for CSPs** from Chapter 6.

---

## Further Reading

Extended notes on **Local Search for CSPs** from Chapter 6.




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
