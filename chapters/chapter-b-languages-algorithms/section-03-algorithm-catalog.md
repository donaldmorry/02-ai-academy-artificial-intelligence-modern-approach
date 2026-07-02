# Section B.3: Algorithm Catalog

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Appendix B.3  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Algorithms Referenced in AIMA

Appendix B catalogs classic algorithms cited throughout the textbook. This section summarizes **search, sort, and graph algorithms** with complexity and AI chapter references.

> **Readable form:** A map of the algorithm zoo - which animal solves which problem and how fast it runs.

---

## Search Algorithms

| Algorithm | Chapter | Strategy | Complete | Optimal | Time | Space |
|-----------|--------|----------|----------|---------|------|-------|
| BFS | 03 | FIFO frontier | Yes* | Yes* | $O(b^d)$ | $O(b^d)$ |
| DFS | 03 | LIFO stack | No | No | $O(b^m)$ | $O(bm)$ |
| UCS | 03 | Cost-based PQ | Yes | Yes | $O(b^{1+\lfloor C^*/\epsilon\rfloor})$ | $O(b^d)$ |
| A* | 03 | $f=g+h$ PQ | Yes* | Yes† | $O(b^d)$ | $O(b^d)$ |
| IDA* | 03 | Iterative deepening + f | Yes* | Yes† | $O(b^d)$ | $O(bd)$ |
| Minimax | 05 | Adversarial | - | - | $O(b^m)$ | $O(bm)$ |
| α-β | 05 | Pruned minimax | - | - | $O(b^{m/2})$ | $O(bm)$ |

*Finite state space  
†With admissible heuristic

---

## Sorting Algorithms

| Algorithm | Average | Worst | Stable | In-place |
|-----------|---------|-------|--------|----------|
| Merge sort | $O(n \log n)$ | $O(n \log n)$ | Yes | No |
| Quick sort | $O(n \log n)$ | $O(n^2)$ | No | Yes |
| Heap sort | $O(n \log n)$ | $O(n \log n)$ | No | Yes |
| Insertion sort | $O(n^2)$ | $O(n^2)$ | Yes | Yes |

Used in preprocessing, priority queue construction, ordering heuristic values.

---

## Graph Algorithms

| Algorithm | Problem | Complexity |
|-----------|---------|------------|
| BFS | Shortest path (unweighted) | $O(V+E)$ |
| Dijkstra | Shortest path (nonneg weights) | $O((V+E)\log V)$ |
| Bellman-Ford | Shortest path (general weights) | $O(VE)$ |
| Floyd-Warshall | All-pairs shortest paths | $O(V^3)$ |
| Kruskal/Prim | Minimum spanning tree | $O(E \log V)$ |
| Topological sort | DAG ordering | $O(V+E)$ |

Relevant for planning graphs, Bayesian network structure, knowledge graphs.

---

## Constraint Satisfaction

| Algorithm | Chapter | Approach |
|-----------|--------|----------|
| Backtracking search | 06 | Depth-first assignment |
| Forward checking | 06 | Prune inconsistent domains |
| AC-3 | 06 | Arc consistency |
| Min-conflicts | 06 | Local search repair |

---

## Logic and Inference

| Algorithm | Chapter | Input |
|-----------|--------|-------|
| DPLL | 07 | Propositional CNF |
| WalkSAT | 07 | Randomized SAT |
| Forward chaining | 09 | Definite clauses |
| Backward chaining | 09 | Goal-directed |
| Resolution | 09 | General FOL |

---

## Machine Learning

| Algorithm | Chapter | Type |
|-----------|--------|------|
| k-NN | 19 | Instance-based |
| Decision tree (ID3) | 19 | Supervised |
| Backpropagation | 21 | Neural network training |
| Q-learning | 22 | Model-free RL |
| EM | 20 | Latent variable ML |

---

## When to Use What

```
Small discrete state space?     → BFS, UCS, A*
Large state space, good heuristic? → A*, IDA*
Adversarial game?               → α-β pruning
Constraints, no obvious search? → CSP backtracking + AC-3
Logical knowledge base?         → Forward/backward chaining
Uncertain, structured?          → Variable elimination, MCMC
Data available, no model?       → Supervised learning, RL
```

---

---

## Study Notes

Russell and Norvig present this material as foundational for multiple AI subfields. When working through exercises:

- Re-derive key formulas before looking up answers
- Implement at least one algorithm or calculation in Python
- Connect concepts to the relevant course chapter
- Test edge cases - algorithms often fail on boundaries first

> **Readable form:** Mastery means you can explain it, compute it, and code it - not just recognize the definition on a flashcard.

### Cross-Chapter Connections

| Related chapter | Connection |
|----------------|------------|
| Earlier foundations | Provides prerequisites for formal derivations |
| Applied chapters | Techniques appear in agents, learning, and perception |
| Ethics and safety | Deployment context shapes how methods are used |

### Practice Checklist

| Step | Action |
|------|--------|
| 1 | Read the AIMA section alongside this section |
| 2 | Complete all five exercises |
| 3 | Implement one code example from scratch |
| 4 | Explain one key formula in plain English |
| 5 | Identify one real-world application |

---

## Additional Examples

### Worked Example

Consider applying the concepts from this section to a concrete AI problem. Identify inputs, outputs, constraints, and evaluation criteria before implementing. Start with the smallest instance that exercises the core idea.

### Implementation Reminder

```python
# Template: verify understanding with executable code
def verify_understanding():
    """Replace with section-specific test."""
    assert True  # placeholder - implement real check
    return "ok"
```

### Summary Table

| Concept | Definition | AI application |
|---------|------------|----------------|
| Core idea | See section sections above | Chapter-specific |
| Key formula | See math blocks above | Computation |
| Pitfall | Common student error | Debugging |
| Extension | Advanced topic | Further reading |

> **Readable form:** If you can fill in this table from memory, you understand the section well enough to move on.

---

## Algorithm Selection Decision Tree

```
Is the problem adversarial?
  Yes → Minimax with alpha-beta pruning
  No → Is the state space continuous?
    Yes → RRT, PRM, or potential fields
    No → Is optimality required?
      Yes → A* with admissible heuristic, or UCS
      No → Greedy best-first or weighted A*
```

### Complexity Quick Reference

| Input size $n$ | $O(n)$ feasible? | $O(n^2)$ feasible? | $O(2^n)$ feasible? |
|----------------|------------------|--------------------|--------------------|
| 10 | Always | Always | Always |
| 100 | Always | Always | Marginal |
| 1,000 | Always | Usually | No |
| 10,000 | Usually | Sometimes | No |

### AIMA Chapter Map

| Algorithm family | Primary chapters |
|------------------|-----------------|
| Uninformed search | 03 |
| Informed search | 03, 04 |
| Adversarial search | 05 |
| CSP | 06 |
| Propositional logic | 07, 09 |
| Probability | 12-15 |
| ML | 19-22 |

> **Readable form:** Pick the algorithm family first, then the specific method based on optimality, completeness, and space budget.

## Key Takeaways

1. BFS/UCS/A* family solves state-space search with varying optimality guarantees.
2. α-β pruning dramatically reduces game tree search while preserving minimax value.
3. Graph algorithms (Dijkstra, etc.) apply to planning and knowledge structure problems.
4. CSP algorithms combine backtracking with constraint propagation (AC-3).
5. Algorithm selection depends on problem structure, size, and optimality requirements.

## Exercises

1. For 8-puzzle, estimate BFS nodes expanded vs. A* with Manhattan heuristic.
2. Implement Dijkstra on a 10-node graph and trace execution.
3. When does BFS use less memory than IDA*?
4. Apply AC-3 to a simple map-coloring CSP with 3 variables.
5. Match each AIMA chapter problem to its primary algorithm from this catalog.

---

**Previous:** [Section B.2 - Pseudocode Conventions](./section-02-pseudocode-conventions.md)

**Next:** [Section B.4 - Data Structures](./section-04-data-structures.md)