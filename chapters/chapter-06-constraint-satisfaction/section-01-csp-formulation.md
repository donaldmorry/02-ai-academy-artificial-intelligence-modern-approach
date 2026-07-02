# Section 06.1: CSP Formulation

> **Source:** Russell & Norvig, *AIMA* (4th ed.), Chapter 6
> **Previous:** [Chapter 05 Overview](../chapter-05-adversarial-search-games/README.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

Search treats states as black boxes. CSPs open the box: states are **assignments** to variables, and constraints prune huge regions of the search space at once.

---

## CSP Definition

A **constraint satisfaction problem** has three components:

| Component | Symbol | Meaning |
|-----------|--------|--------|
| Variables | $X = \{X_1, \ldots, X_n\}$ | Things to assign values to |
| Domains | $D = \{D_1, \ldots, D_n\}$ | Allowed values per variable |
| Constraints | $C$ | Allowed combinations |

A **solution** is a complete, **consistent** assignment - every variable has a value and no constraint is violated.

> **Readable form:** A CSP is a fill-in-the-blanks puzzle where each blank has legal options and rules say which combinations are allowed.

---

## Map Coloring Example

Australia map coloring from AIMA §6.1:

$$X = \{\text{WA, NT, Q, NSW, V, SA, T}\}, \quad D_i = \{\text{red, green, blue}\}$$

Constraints: neighboring regions must differ. The **constraint graph** has nodes = variables, edges = shared constraints.

Once $\text{SA} = \text{blue}$, five neighbors cannot be blue - pruning $3^5 = 243$ naive combinations down to $2^5 = 32$.

---

## CSP vs. State-Space Search

| Aspect | State-space search | CSP |
|--------|-------------------|-----|
| State | Atomic | Factored (variable assignments) |
| Goal test | Per-state | All constraints satisfied |
| Pruning | Heuristic estimates | Constraint propagation |
| Transition model | Domain-specific | Deduced from constraints |

---


## Key Equations


$$
CSP = \langle X, D, C \rangle
$$
> **Readable form:** A CSP is a tuple of variables, domains, and constraints.

$$
\text{consistent}(A) \Leftrightarrow \forall C_j \in C: C_j \text{ satisfied by } A
$$
> **Readable form:** An assignment is consistent if every constraint is satisfied.


---

## Implementation

```python
class CSP:
    def __init__(self, variables, domains, constraints):
        self.variables = variables
        self.domains = {v: list(domains[v]) for v in variables}
        self.constraints = constraints  # list of (scope, relation)

    def is_consistent(self, assignment):
        for scope, rel in self.constraints:
            if all(v in assignment for v in scope):
                if not rel(*[assignment[v] for v in scope]):
                    return False
        return True

def australia_csp():
    colors = ["red", "green", "blue"]
    vars = ["WA", "NT", "Q", "NSW", "V", "SA", "T"]
    domains = {v: colors for v in vars}
    neighbors = [("SA","WA"),("SA","NT"),("SA","Q"),("SA","NSW"),("SA","V"),
                 ("WA","NT"),("NT","Q"),("Q","NSW"),("NSW","V")]
    constraints = [((a,b), lambda x,y, a=a,b=b: x != y) for a,b in neighbors]
    return CSP(vars, domains, constraints)
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
| **constraint satisfaction problem** | Concept from Chapter 6 |
| **domain** | Concept from Chapter 6 |
| **constraint** | Concept from Chapter 6 |
| **consistent assignment** | Concept from Chapter 6 |
| **constraint graph** | Concept from Chapter 6 |

---

## Key Takeaways

1. **CSP Formulation** covers Variables.
2. Formalism connects to implementable algorithms.
3. Builds toward chapter lab and later chapters.

---

## Reflection Questions

1. Explain **csp formulation** in your own words.
2. What assumptions does Chapter 6 make?
3. How verify on a toy example?
4. Real-world AI applications?
5. Which later chapter depends on this?

---

**Previous:** [Chapter 05 Overview](../chapter-05-adversarial-search-games/README.md) · **Next:** [Section 06.2](./section-02-backtracking-search.md)

---

## References

1. Russell & Norvig (2020). *AIMA* (4th ed.). https://aima.cs.berkeley.edu/
2. aima-python: https://github.com/aimacode/aima-python
3. Berkeley CS188: https://inst.eecs.berkeley.edu/~cs188/
4. MIT 6.034 OCW.
5. Stanford AI Index: https://aiindex.stanford.edu/

---

## Worked Example

Extended notes on **CSP Formulation** from Chapter 6.

---

## Common Pitfalls

Extended notes on **CSP Formulation** from Chapter 6.

---

## Historical Note

Extended notes on **CSP Formulation** from Chapter 6.

---

## Further Reading

Extended notes on **CSP Formulation** from Chapter 6.

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
