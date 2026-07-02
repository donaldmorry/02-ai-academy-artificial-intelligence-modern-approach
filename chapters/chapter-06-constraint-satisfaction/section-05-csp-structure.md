# Section 6.5: CSP Structure

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Section 6.4  
> **Previous:** [Section 6.4 - CSP Heuristics](./section-04-heuristics-for-csp.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Exploiting Problem Structure

General CSPs are NP-complete. Many real instances have **structure** - sparse constraint graphs, tree topology, independent subproblems - enabling algorithms faster than exponential search.

> **Readable form:** If constraints connect variables in a simple pattern, solve pieces separately or use dynamic programming on a tree.

---

## Independent Subproblems

If the **constraint graph** disconnects into components $G_1, G_2$, solve each independently:

$$
\text{Solution}(CSP) = \text{Solution}(G_1) \times \text{Solution}(G_2)
$$
> **Readable form:** The constraint expression restricts which variable assignments are allowed in the solution.

**Example:** Australia map with Tasmania isolated - solve mainland and T separately.

| Check | Action |
|-------|--------|
| Connected components | Run solver per component |
| Complexity | Sum of subproblem costs, not product of domains |

---

## Tree-Structured CSPs

A **tree** has no cycles. Pick a root; orient edges parent → child. Solve **from leaves upward**:

1. Remove leaf $X_i$, make its constraint **unary** on parent (project $X_i$ out)
2. Repeat until one variable
3. Assign root; assign children top-down using stored constraints

**Time:** $O(n d^2)$ for $n$ variables, domain size $d$ - **linear in $n$** for fixed $d$.

```
    WA
     |
    NT - Q
     |
    SA - NSW - V
```

If graph were a tree (remove some edges), this algorithm applies.

---

## Cutset Conditioning

For **cyclic** graphs, choose a **cutset** of variables whose removal makes the remaining graph a **forest**. Try all assignments to cutset (brute force on cutset size $c$), solve each tree component:

$$
\text{Time} \approx O(d^c \cdot n d^2)
$$
> **Readable form:** Guess the hard variables that tie cycles together; solve the easy tree parts for each guess.

**Treewidth** $w$: size of smallest cutset in a **tree decomposition** - measures "how tree-like" the CSP is.

---

## Treewidth Intuition

| Treewidth | Example |
|-----------|---------|
| 1 | Tree |
| 2 | Almost tree (series-parallel) |
| High | Dense constraints (Sudoku) |

Algorithms exist polynomial in $n$ for fixed treewidth $w$ - impractical if $w$ is large.

---

## Extended Worked Example: Tree CSP

Variables A-B-C in a line (tree). Domains $\{1,2\}$, constraint $X_i \neq X_{i+1}$.

1. Remove leaf C: A-B with unary on B from C
2. Remove B: unary on A
3. Assign A=1, propagate B=2, C=1 - **fail** at C
4. Backtrack A=2, B=1, C=2 - **success**

---

## Comparison Table

| Structure | Algorithm | Complexity |
|-----------|-----------|------------|
| Independent components | Separate solve | $\sum$ subproblems |
| Tree | Directed arc consistency | $O(n d^2)$ |
| General graph | Backtracking + MAC | Exponential worst |
| Low treewidth | Tree decomposition | Polynomial in $n$ |

---

## Implementation Notes

```python
def tree_csp_solver(csp, root):
    ordered = topological_from_root(csp, root)
    for Xi in reversed(ordered):
        project_out_leaf(Xi)
    assignment = {}
    for Xi in ordered:
        assignment[Xi] = any_consistent_value(Xi, assignment)
    return assignment
```

**AIMA connection:** Section 6.4 - structure is why map coloring is easy but random 3-SAT at threshold is hard.

---

## PEAS: Compiler Register Allocation

| PEAS | Structured CSP |
|------|----------------|
| **P** | Color interference graph with few registers |
| **E** | Sparse interference (tree-like for expressions) |
| **A** | Assign register per variable |
| **S** | Interference edges |

Expression trees often have low treewidth.

---

## Connection to Bayesian Networks

[Chapter 13](../chapter-13-probabilistic-reasoning/README.md): same **graph structure** ideas - tree networks enable efficient inference; high treewidth is intractable.

---

## Common Mistakes

**Assuming all CSPs are hard.** Check constraint graph first.

**Cutset too large.** Exponential in cutset size - pick minimal cycle-breaking set.

**Confusing tree with low degree.** Graph can have degree 2 but contain cycles.

---

## AIMA Connection

Dechter's *Constraint Processing* extends tree algorithms - AIMA gives the core tree-CSP and cutset conditioning intuition.

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

**Next:** [Section 6.6 - Local Search for CSPs](./section-06-local-search-for-csps.md)

---

## References

1. Russell & Norvig AIMA Ch 6
2. aima-python csp.py
3. Dechter Constraint Processing
