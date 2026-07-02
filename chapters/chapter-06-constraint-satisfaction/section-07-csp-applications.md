# Section 6.7: CSP Applications

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 6 (applications)  
> **Previous:** [Section 6.6 - Local Search for CSPs](./section-06-local-search-for-csps.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## CSPs in Practice

[Sections 6.1-6.6](./section-01-csp-formulation.md) built algorithms. This section maps **real domains** to $\langle X, D, C \rangle$ and chooses backtracking, propagation, structure, or local search accordingly.

> **Readable form:** Scheduling, puzzles, and configuration are the same abstract problem with different variable names.

---

## Sudoku

81 variables, domain $\{1,\ldots,9\}$ (reduced by givens).

| Constraint type | Encoding |
|-----------------|----------|
| Row alldiff | 9 global constraints |
| Column alldiff | 9 global |
| Box alldiff | 9 global |

**Solver stack:** MAC (AC-3 after each assignment) + MRV + LCV. Hard puzzles need full stack; easy puzzles solved by AC-3 alone ([Section 6.3](./section-03-constraint-propagation.md)).

---

## Map and Graph Coloring

**Variables:** regions. **Domains:** colors. **Constraints:** adjacent $\neq$.

| Instance | Structure |
|----------|-----------|
| Australia | Sparse, tree-like regions |
| Planar maps | 4-color theorem - 4 colors always suffice |

Used in register allocation (interference graph coloring) and frequency assignment.

---

## Timetabling

**Variables:** exam slot (or course-section). **Domains:** time rooms. **Constraints:**

- No student double-booked
- Room capacity
- Instructor availability
- Precedence (exam A before B)

Often **soft constraints** as penalties (Max-CSP) when perfect schedule impossible.

---

## Configuration and Design

Product configurators (cars, PCs): **variables** = options; **constraints** = compatibility rules ("sunroof only with premium package").

| Approach | When |
|----------|------|
| Backtracking + propagation | Interactive configurator (< 1s response) |
| Local search | Large legacy rule bases |

---

## SAT and CSP

**Boolean CSP** = **SAT**: variables $\{0,1\}$, constraints = clauses.

| CSP technique | SAT analogue |
|---------------|--------------|
| Backtracking | DPLL |
| Conflict learning | CDCL solvers |
| WalkSAT / min-conflicts | Stochastic SAT |

Modern SAT solvers solve millions of clauses - [Chapter 07](../chapter-07-logical-agents/section-03-inference-and-entailment.md) connects logic to CSP.

---

## Extended Worked Example: Exam Scheduling

3 exams $\{E_1,E_2,E_3\}$, 2 slots $\{S_1,S_2\}$. Students: $E_1$ conflicts with $E_2$, $E_2$ with $E_3$.

Variables = exams, domains = slots, binary $\neq$ if shared students.

Solution: $E_1=S_1$, $E_2=S_2$, $E_3=S_1$ - valid 3-coloring of conflict graph.

---

## Comparison: Algorithm Choice

| Application | Typical method |
|-------------|----------------|
| Sudoku | MAC + MRV |
| Million-queens | Min-conflicts |
| VLSI routing | Backtracking + structure |
| Industrial SAT | CDCL |

---

## Implementation Notes

| Pitfall | Fix |
|---------|-----|
| Exploding alldiff binary encoding | Global constraint propagator |
| Slow consistency check | Incremental check on assignment |
| Real-time UX | Preprocess with AC-3 before user sees |

**AIMA connection:** `csp.py` map coloring and N-queens - template for custom applications.

---

## PEAS: University Exam Scheduler

| PEAS | CSP scheduler |
|------|---------------|
| **P** | Zero conflicts; minimize late exams |
| **E** | Student enrollments, room list |
| **A** | Assign slot |
| **S** | Partial schedule, conflict count |

---

## Connection to Planning

[Chapter 11](../chapter-11-automated-planning/README.md): planning can compile to SAT/CSP - same solvers, different compilation.

---

## Common Mistakes

**Wrong CSP formulation.** Extra variables can simplify constraints (auxiliary scheduling slots).

**Ignoring soft constraints.** Real timetabling rarely has perfect solutions - use weighted violations.

**Using min-conflicts on tight Sudoku.** Need propagation for hard puzzles.

---

## AIMA Chapter 6 Applications

Russell & Norvig emphasize **formulation** as half the battle - same algorithms, different domain modeling.

---


---

## Extended Worked Example

Trace the algorithm on a minimal instance by hand before coding. State the invariant maintained at each step (e.g., consistency of partial assignment, backup values at MIN nodes, or domain non-emptiness after propagation).

| Step | State | Action | Notes |
|------|-------|--------|-------|
| 1 | Initial | - | Baseline |
| 2 | After first decision | - | Check constraints |
| 3 | Terminal / goal | - | Verify solution |

> **Readable form:** Hand simulation catches off-by-one errors that unit tests miss.

---

## Implementation Notes (aima-python)

Compare your code to the reference implementation in `aima-python`. Instrument node expansions, backtracks, or inference steps. Log domains after each `revise` or each alpha-beta cutoff.

| Check | Why |
|-------|-----|
| Correct stopping condition | Prevents infinite loops |
| Neighbor / action generation | Common source of bugs |
| Tie-breaking | Affects reproducibility |

---

## Comparison Table

| Aspect | Naive approach | This section's technique |
|--------|----------------|-------------------------|
| Time (worst) | Exponential | Problem-dependent |
| Memory | Varies | See section |
| Completeness | See section | See section |
| Optimality | See section | See section |

---

## AIMA Connection

Re-read the matching section in Russell & Norvig (4th ed.) and complete one end-of-chapter exercise. Connect definitions to the [GLOSSARY.md](../../GLOSSARY.md) entries cited in the section header.

---

## Common Mistakes

- Skipping worked examples and going straight to code.
- Ignoring environment assumptions from PEAS (static vs dynamic, fully vs partially observable).
- Confusing worst-case complexity with typical performance on structured instances.
- Forgetting to restore state on backtrack (CSP domains, game tree depth, belief sets).


## Reflection Questions

1. CSP vs search?
2. MRV?
3. AC-3 vs forward checking?
4. Tree CSPs?
5. Real application?

---

**Next:** [Section 6.8 - CSP Sudoku Lab](./section-08-sudoku-csp-lab.md)

---

## References

1. Russell & Norvig AIMA Ch 6
2. aima-python csp.py
3. Dechter Constraint Processing