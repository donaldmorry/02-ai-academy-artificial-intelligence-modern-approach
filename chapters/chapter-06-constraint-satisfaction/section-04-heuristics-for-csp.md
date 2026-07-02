# Section 06.4: Heuristics for CSP

> **Source:** Russell & Norvig, *AIMA* (4th ed.), Chapter 6
> **Previous:** [Section 06.3](./section-03-constraint-propagation.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

Variable and value ordering heuristics often matter more than raw propagation - MRV, degree, and LCV are the CSP solver's secret weapons.

---

## Minimum Remaining Values (MRV)

Choose the variable with the **fewest legal values** remaining. Fail-first: if a variable will cause failure, discover it early.

---

## Degree Heuristic

Among MRV ties, pick the variable involved in the **most constraints** with unassigned variables - it constrains the most future choices.

---

## Least Constraining Value (LCV)

When trying values, order by how many options remain for neighbors. Pick the value that **rules out the fewest** choices for other variables.

---


## Key Equations


$$
\text{MRV}(X) = \arg\min_{X_i \in \text{unassigned}} |D_i|
$$
> **Readable form:** MRV selects the variable with the smallest remaining domain.


---

## Implementation

```python
def select_unassigned(csp, assignment):
    unassigned = [v for v in csp.variables if v not in assignment]
    return min(unassigned, key=lambda v: len(csp.domains[v]))

def order_values(var, csp, assignment):
    def conflicts(value):
        assignment[var] = value
        count = sum(1 for nb in csp.neighbors(var)
                    if nb not in assignment
                    and all(d == value for d in [assignment.get(nb)]))
        del assignment[var]
        return count
    return sorted(csp.domains[var], key=conflicts)
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
| **MRV** | Concept from Chapter 6 |
| **degree heuristic** | Concept from Chapter 6 |
| **LCV** | Concept from Chapter 6 |
| **fail-first** | Concept from Chapter 6 |

---

## Key Takeaways

1. **Heuristics for CSP** covers MRV.
2. Formalism connects to implementable algorithms.
3. Builds toward chapter lab and later chapters.

---

## Reflection Questions

1. Explain **heuristics for csp** in your own words.
2. What assumptions does Chapter 6 make?
3. How verify on a toy example?
4. Real-world AI applications?
5. Which later chapter depends on this?

---

**Previous:** [Section 06.3](./section-03-constraint-propagation.md) · **Next:** [Section 06.5](./section-05-csp-structure.md)

---

## References

1. Russell & Norvig (2020). *AIMA* (4th ed.). https://aima.cs.berkeley.edu/
2. aima-python: https://github.com/aimacode/aima-python
3. Berkeley CS188: https://inst.eecs.berkeley.edu/~cs188/
4. MIT 6.034 OCW.
5. Stanford AI Index: https://aiindex.stanford.edu/

---

## Worked Example

Extended notes on **Heuristics for CSP** from Chapter 6.

---

## Common Pitfalls

Extended notes on **Heuristics for CSP** from Chapter 6.

---

## Historical Note

Extended notes on **Heuristics for CSP** from Chapter 6.

---

## Further Reading

Extended notes on **Heuristics for CSP** from Chapter 6.




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
