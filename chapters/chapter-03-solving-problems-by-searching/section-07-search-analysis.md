# Section 3.7: Search Analysis

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 3.4-3.5  
> **Previous:** [Section 3.6 - Search Implementations](./section-06-search-implementations.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Why Analysis Matters

Implementing [BFS, UCS, and A*](./section-06-search-implementations.md) is not enough - practitioners must predict **whether search will finish**, **whether the answer is optimal**, and **what resources** are required. This section systematizes **completeness**, **optimality**, **time complexity**, and **space complexity** for all strategies in Chapter 03.

> **Readable form:** Analysis answers four questions - Will it find a solution? Will it be the best? How long? How much memory?

These properties guide algorithm choice before writing code - especially when state spaces reach billions ([8-puzzle](./section-02-example-problems.md)) or beyond.

---

## Terminology for Complexity

| Symbol | Meaning |
|--------|---------|
| $b$ | **Branching factor** - max (or avg) successors per state |
| $d$ | **Depth** of shallowest goal |
| $m$ | **Maximum depth** of search tree (may be $\infty$) |
| $C^*$ | Cost of optimal solution |
| $\epsilon$ | Lower bound on step cost ($> 0$ for UCS termination) |
| $N$ | Number of nodes in state space |

**Generated vs. expanded:** Analysis usually counts **expanded** nodes (removed from frontier and processed). Generated includes duplicates discarded.

---

## Completeness

**Definition:** An algorithm is **complete** if it finds a solution whenever one exists (in finite time, for reasonable cost models).

| Algorithm | Complete? | Condition / caveat |
|-----------|-----------|-------------------|
| BFS (graph) | Yes | Finite state space |
| DFS (graph) | Yes | Finite state space |
| DFS (tree, infinite) | **No** | Infinite path avoids goal |
| Depth-limited | If $l \geq d$ | Otherwise may miss deeper goals |
| IDS | Yes | Finite space, uniform costs |
| UCS | Yes | Step cost $\geq \epsilon > 0$ |
| Greedy | **No** | Can cycle on infinite graphs |
| A* | Yes | Finite space, admissible $h$, graph search |

**Zero-cost cycles** break UCS completeness (infinite loops with no progress). **Graph search** with explored set is essential for completeness in finite spaces with cycles ([Romania](./section-02-example-problems.md) has cycles).

---

## Optimality

**Definition:** An algorithm is **optimal** if it returns a **minimum-cost** solution when one exists.

| Algorithm | Optimal? | Requirement |
|-----------|----------|-------------|
| BFS | Yes | Uniform step cost (= 1) |
| DFS | No | May return deep suboptimal path |
| IDS | Yes | Uniform step cost |
| UCS | Yes | Nonnegative costs, $\epsilon > 0$ |
| Greedy | No | Ignores $g(n)$ |
| A* | Yes | Admissible $h$ (tree); consistent $h$ (graph) |

When step costs vary, **BFS by depth is not optimal** - use UCS or A*.

**Rational agent link:** For a [goal-based agent](../chapter-02-intelligent-agents/section-04-structure-of-agents.md) whose [performance measure](../../GLOSSARY.md#performance-measure) is negative path cost, optimality of search aligns with **local rationality** given known model.

---

## Time Complexity

Worst-case **time** is often expressed as nodes expanded:

### Uninformed

| Algorithm | Time |
|-----------|------|
| BFS | $O(b^d)$ |
| DFS | $O(b^m)$ |
| Depth-limited | $O(b^l)$ |
| IDS | $O(b^d)$ - dominated by deepest iteration |
| UCS | $O(b^{1 + \lfloor C^* / \epsilon \rfloor})$ |

For IDS, shallow iterations contribute geometric series:

$$
\sum_{i=0}^{d} b^i = \frac{b^{d+1} - 1}{b - 1} = O(b^d)
$$
> **Readable form:** IDS repeats shallow work, but the deepest level dominates total time - same order as BFS.

### Informed

| Algorithm | Time (worst case) |
|-----------|-------------------|
| Greedy | $O(b^m)$ - can degrade to exhaustive |
| A* | Exponential in problem size - but **practical** with good $h$ |

**Effective branching factor** $b^*$: if A* expands $N$ nodes to depth $d$, solve $N \approx (b^*)^d$. Good heuristics yield $b^* \approx 1.2$ on 8-puzzle vs. $b \approx 3$.

---

## Space Complexity

Often the **binding constraint** in search:

| Algorithm | Space |
|-----------|-------|
| BFS | $O(b^d)$ - stores entire frontier level |
| DFS | $O(bm)$ or $O(m)$ - one path + siblings |
| IDS | $O(bd)$ |
| UCS | $O(b^{C^*/\epsilon})$ - similar to time |
| A* | $O(b^d)$ worst case - often less in practice |

**Memory map:** BFS and A* frontiers grow exponentially; DFS and IDS suit memory-limited agents.

Example: $b=10$, $d=12$ → BFS frontier $\approx 10^{12}$ nodes - impossible. IDS at $O(10 \times 12)$ path memory - feasible.

---

## A* Optimality Efficiency

**Theorem (de Champeaux, AIMA):** No algorithm using the same heuristic information can expand **fewer** nodes than A* in the worst case while guaranteeing optimality.

A* with admissible $h$ expands:

- All nodes with $f(n) < C^*$
- Some nodes with $f(n) = C^*$

Nodes with $f(n) > C^*$ are never expanded - **pruned** by optimality.

**Corollary:** Tighter admissible heuristics (dominating) reduce the set $\{n : f(n) < C^*\}$.

---

## Admissibility vs. Consistency - Analysis Impact

| Property | Guarantees |
|----------|------------|
| Admissible $h$ | A* tree search optimal; $h$ never overestimates |
| Consistent $h$ | $f$ nondecreasing along paths; graph A* need not re-open nodes |

Admissible but **inconsistent** heuristics may require **re-opening** closed nodes in graph search if a cheaper path arrives later - implementation complexity.

Most analyses assume **consistent** $h$ for cleaner graph-search proofs ([Section 3.4](./section-04-informed-search.md)).

---

## Branching Factor in Practice

Measuring $b$ on actual problems:

$$
b_{\text{avg}} = \left(\frac{\text{total successors generated}}{\text{non-leaf nodes expanded}}\right)^{1/d}
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

| Problem | Typical $b$ |
|---------|-------------|
| Romania | ~2-3 |
| 8-puzzle | ~3 |
| Vacuum | small |

The same analysis framework applies to both Romania and the 8-puzzle - only $b$, $d$, and $| \mathcal{S} |$ change.

---

## Completeness vs. Cost

**Infinite state spaces:** BFS/DFS may not terminate with infinite depth. IDS requires finite minimum depth for uniform-cost goals.

**Partial observability** and **nondeterminism** change analysis - [Chapter 04](../chapter-04-search-complex-environments/README.md).

---

## Comparing All Chapter 03 Algorithms

| Algorithm | Complete | Optimal | Time | Space | Notes |
|-----------|----------|---------|------|-------|-------|
| BFS | Yes | Yes† | $O(b^d)$ | $O(b^d)$ | †uniform cost |
| DFS | Finite* | No | $O(b^m)$ | $O(bm)$ | |
| IDS | Yes | Yes† | $O(b^d)$ | $O(bd)$ | Best uninformed uniform cost |
| UCS | Yes | Yes | high | high | Weighted graphs |
| Greedy | No | No | varies | $O(b^m)$ | Fast but risky |
| A* | Yes | Yes | lower with good $h$ | $O(b^d)$ worst case | **Preferred informed** |

---

## Scaling and Hard Problems

| Problem | $| \mathcal{S} |$ | Practical approach |
|---------|-------------|---------------------|
| Romania | ~20 | Any algorithm instant |
| 8-puzzle | 181,440 | A* + Manhattan |
| 15-puzzle | $\sim 10^{13}$ | Pattern databases, IDA* |
| Chess | astronomical | Heuristic + game search ([Chapter 05](../chapter-05-adversarial-search-games/README.md)) |

When $b^d$ exceeds memory, **IDA*** (iterative deepening A*) - depth-first with A* f-cost cutoff - trades space for time re-computation.

---

## PEAS and Resource Limits

Real agents have **bounded memory** - maps to space complexity. A drone with 512 MB cannot run BFS on huge grids; IDS or A* with good $h$ is appropriate.

[Performance measure](../../GLOSSARY.md#performance-measure): if the measure is "minimize time to first solution" not "minimize cost", greedy or weighted A* may be rational despite suboptimality ([Section 2.2](../chapter-02-intelligent-agents/section-02-good-behavior-and-rationality.md)).

---

## Worked Example: Complexity Estimate for 8-Puzzle

Assume $b=3$, optimal solution depth $d=18$:

**BFS nodes (rough):** $3^{18} \approx 3.9 \times 10^8$ - too many for naive BFS memory.

**A* with Manhattan:** effective $b^* \approx 1.5$-$2$ → expansions often $10^4$-$10^5$ for random instances - tractable.

This explains why [heuristic design](./section-05-heuristic-functions.md) matters more than raw CPU speed.

---

## Common Exam Questions

1. **Why is IDS preferred over BFS for large depth?** Same time order, linear space.

2. **When is UCS not optimal?** Negative cycles or zero-cost loops without proper handling.

3. **Does consistent h imply fewer expansions than admissible h?** Consistent is subset of admissible with extra property - not necessarily more informed; dominance among heuristics matters.

4. **Space of A* vs BFS at same depth?** Same worst-case order; A* often expands fewer nodes so frontier may be smaller in practice.

---

## Reflection Questions

1. Construct a tiny graph where greedy finds a goal but not minimum cost.
2. For Romania, estimate $d$ and $b$ - is BFS reasonable?
3. Why does UCS require $\epsilon > 0$ step cost lower bound?
4. Compare IDS and A* space when $d=20$, $b=3$.
5. How does weighted A* trade optimality for time? ([Section 3.4](./section-04-informed-search.md))

---

**Next:** [Section 3.8 - Romania A* Lab](./section-08-romania-a-lab.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Sections 3.4-3.5. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Korf, R. E. - depth-first iterative-deepening analysis. *Artificial Intelligence*, 1985.
3. Dealing, J. - complexity of heuristic search.
4. Hart, Nilsson, Raphael (1968) - A* optimality.
5. Appendix A in AIMA - $O(\cdot)$ notation and complexity review. [Chapter A](../chapter-a-mathematical-background/README.md)
