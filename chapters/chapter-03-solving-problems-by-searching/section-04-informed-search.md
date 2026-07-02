# Section 3.4: Informed Search

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 3.5  
> **Previous:** [Section 3.3 - Uninformed Search](./section-03-uninformed-search.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Idea of Heuristics

[Uninformed search](./section-03-uninformed-search.md) treats all frontier nodes equally except for depth or $g(n)$. **Informed search** adds a [heuristic](../../GLOSSARY.md#heuristic) function $h(n)$ - an estimate of the **remaining cost** from state $n$ to a goal.

$$
h(n) \approx h^*(n) = \text{true cheapest cost from } n \text{ to goal}
$$
> **Readable form:** h(n) guesses how much more it will cost to finish from here; h-star(n) is the actual minimum remaining cost (usually unknown).

Good heuristics **focus** expansion toward promising states - often expanding far fewer nodes than UCS while still finding optimal paths (when conditions are met).

Humorous analogy: uninformed search is **exploring every aisle** in a supermarket for milk; informed search is **following the "Dairy" sign** - wrong signs cause detours, good signs save time.

---

## Best-First Search (General Form)

Expand the node with **best evaluation function** $f(n)$ first - priority queue ordered by $f$.

| Variant | $f(n)$ | Behavior |
|---------|--------|----------|
| **Greedy best-first** | $h(n)$ | Rush toward goal estimate |
| **A\*** | $g(n) + h(n)$ | Balance cost so far + estimate to go |
| **Weighted A\*** | $g(n) + w \cdot h(n)$ | Trade optimality for speed |

[Section 3.5](./section-05-heuristic-functions.md) designs $h$; this section proves **when A* works**.

---

## Greedy Best-First Search

Greedy search uses $f(n) = h(n)$ only - ignores sunk cost $g(n)$.

On [Romania](./section-02-example-problems.md) from Arad with $h$ = straight-line distance to Bucharest:

1. Expand Arad → neighbors Zerind, Sibiu, Timisoara
2. Pick Sibiu (smallest $h$ among neighbors) - depends on SL distances
3. Continue toward cities that **look** closest to Bucharest on map

### Properties

| Property | Greedy |
|----------|--------|
| Complete? | No - can loop in infinite spaces without checks |
| Optimal? | **No** - can take high-cost detours |
| Time/Space | $O(b^m)$ worst case; often much better in practice |

Greedy may reach goal quickly with **suboptimal** path - e.g., Arad → Sibiu → Fagaras → Bucharest (450 km) vs. optimal 418 km.

**Use when:** fast approximate solution acceptable; as building block in other algorithms.

---

## A* Search

[AIMA's centerpiece](../../GLOSSARY.md#a-star-search): expand node minimizing

$$
f(n) = g(n) + h(n)
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

where:

$$
g(n) = \text{cost from start to } n, \quad h(n) = \text{estimated cost from } n \text{ to goal}
$$
> **Readable form:** f(n) = cost paid so far + estimated cost remaining - total estimated path cost through n.

A* is **UCS** when $h(n) = 0$ everywhere; **greedy** when $g(n) = 0$ (not useful). With good $h$, A* combines UCS optimality tendency with greedy focus.

### Algorithm

Same skeleton as UCS, but priority key is $f = g + h$:

```
function A_STAR(problem, h):
    node ← NODE(problem.initial, g=0)
    node.f ← h(node.state)
    frontier ← PRIORITY_QUEUE(node, key=f)
    explored ← map state → best_g
    while frontier not empty:
        node ← POP_MIN(frontier)
        if problem.goal_test(node.state): return solution(node)
        explored[node.state] ← node.g
        for each child with step_cost c:
            child.g ← node.g + c
            child.f ← child.g + h(child.state)
            if child.state not explored or child.g < explored[child.state]:
                ADD(frontier, child)
    return FAILURE
```

---

## Admissible Heuristics

**Definition:** $h(n)$ is **admissible** if it **never overestimates** true remaining cost:

$$
h(n) \leq h^*(n) \quad \text{for all nodes } n
$$
> **Readable form:** The guess is always optimistic - it says the goal is at least as close as reality allows.

Examples on Romania:

| Heuristic | Admissible? | Why |
|-----------|-------------|-----|
| Straight-line distance to Bucharest | Yes | Shortest path on map ≥ straight line |
| $h(n) = 0$ | Yes | Vacuous (degrades to UCS) |
| $h(n) = 2 \times \text{SL}(n)$ | No | Doubles may exceed true road distance |
| Max road speed × SL distance | Maybe | If speeds/costs calibrated lower bound |

For 8-puzzle with unit-cost moves, **Manhattan distance** is admissible ([Section 3.5](./section-05-heuristic-functions.md)).

See [admissible heuristic](../../GLOSSARY.md#admissible-heuristic) in the glossary.

---

## Optimality of A*

**Theorem (AIMA):** If $h(n)$ is admissible and the state space is finite, A* tree search finds an **optimal** solution.

**Sketch:** When A* selects goal $G$, every node on frontier has $f \geq f(G) = g(G) = C^*$. For any suboptimal goal $G'$ with cost $C' > C^*$, we have $f(G') = C' > C^*$. Admissibility ensures no unexplored path can sneak below $C^*$ without being expanded first.

With **graph search**, need **consistency** (below) for first expansion of goal to be optimal.

---

## Consistent (Monotonic) Heuristics

**Definition:** $h$ is **consistent** if for every node $n$ and successor $n'$ via cost $c$:

$$
h(n) \leq c(n, a, n') + h(n')
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

Triangle inequality along any edge - heuristic drops by at most step cost.

Equivalent formulation along path:

$$
f(n') = g(n') + h(n') \geq g(n) + h(n) = f(n)
$$
> **Readable form:** f never decreases along any path - monotonic.

| Fact | Statement |
|------|-----------|
| Consistent $\Rightarrow$ Admissible | True (set $n' = goal$, $h(goal)=0$) |
| Admissible $\Rightarrow$ Consistent | **False** in general |

Consistent heuristics guarantee:

1. A* **graph search** optimality without re-opening nodes (with standard implementation)
2. $f$ values along any path are nondecreasing

Most natural physical heuristics (SL distance, Manhattan) are consistent.

---

## A* vs. UCS vs. Greedy - Romania

From Arad to Bucharest, $h$ = straight-line distance:

| Algorithm | Typical behavior |
|-----------|------------------|
| UCS | Expands many cities in increasing $g$ |
| Greedy | Few expansions, may miss 418 km path |
| A* | Fewer than UCS; finds 418 km optimal |

A* expands **all nodes with $f < C^*$** and some with $f = C^*$ - ** optimally efficient** among algorithms using the same heuristic (no algorithm using only $h$ expands fewer nodes and guarantees optimality).

---

## Weighted A* and Suboptimality Bounds

When speed matters more than perfection:

$$
f_w(n) = g(n) + w \cdot h(n), \quad w > 1
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

Expands fewer nodes; solution cost $\leq w \cdot C^*$ under admissible $h$ (weighted A* analysis).

Used in robotics and games when near-optimal in milliseconds beats optimal in seconds.

---

## Beam Search (Brief)

Keep only top-$k$ nodes by $f$ at each depth - **incomplete**, **not optimal**, but scales to huge spaces. Common in NLP decoding and neural search; not guaranteed optimal like A*.

---

## Informed Search Properties Summary

| Algorithm | Optimal | Complete | Needs |
|-----------|---------|----------|-------|
| Greedy | No | No* | Any $h$ |
| A* (tree) | Yes | Yes* | Admissible $h$ |
| A* (graph) | Yes | Yes* | Consistent $h$ (standard) |
| Weighted A* | Bounded | Yes* | Admissible $h$ |

\*With graph search and proper duplicate handling in finite spaces.

---

## PEAS Connection

For a [goal-based agent](../chapter-02-intelligent-agents/section-04-structure-of-agents.md) navigating Romania:

| PEAS element | Informed search role |
|--------------|---------------------|
| **Performance** | Minimize distance - A* with admissible $h$ |
| **Environment** | Known map enables $h$ = SL distance |
| **Actuators** | Move to neighbor |
| **Sensors** | Current city for $g$ and $h$ lookup |

Without geographic coordinates, SL heuristic unavailable - domain-specific $h$ design required ([Section 3.5](./section-05-heuristic-functions.md)).

---

## Common Pitfalls

**Inadmissible $h$ breaks optimality.** Inflated heuristic → A* may skip optimal branch.

**Confusing $g$ and $h$.** $g$ is **known** sunk cost; $h$ is **estimate** only.

**Forgetting graph search duplicates.** Same app may reach same city with different $g$ - keep best $g$ on frontier.

**Using inadmissible $h$ for "speed" without weighted A* theory.** You lose optimality guarantees unpredictably.

---

## Worked Example: A* Node Ordering

At Sibiu ($g = 140$), $h$ = SL to Bucharest ≈ 253:

| Neighbor | $g' $ | $h$ | $f = g' + h$ |
|----------|-------|-----|--------------|
| Fagaras | 239 | ~176 | ~415 |
| Rimnicu Vilcea | 220 | ~193 | ~413 |
| Arad | 280 | ... | higher |

Rimnicu Vilcea may enter frontier before Fagaras despite Fagaras **looking** closer to Bucharest - A* respects **total** estimated cost.

Full expansion trace: [Section 3.8](./section-08-romania-a-lab.md).

---

## Reflection Questions

1. Prove: consistent $h$ implies admissible $h$ when $h(goal) = 0$.
2. Why is greedy best-first not optimal on Romania even with perfect SL $h$?
3. What happens to A* if $h(n) = h^*(n)$ exactly for all $n$?
4. Is $h = 0$ admissible? What algorithm does A* become?
5. Give an admissible but inconsistent heuristic (hint: tiny graph by hand).

---

**Next:** [Section 3.5 - Heuristic Functions](./section-05-heuristic-functions.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 3.5. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Hart, P. E., Nilsson, N. J., & Raphael, B. (1968). A formal basis for the heuristic determination of minimum cost paths. *IEEE Trans. Systems Science and Cybernetics* - original A*.
3. Dechter, R., & Pearl, J. (1985). Generalized best-first search strategies. *JACM*.
4. Pearl, J. *Heuristics: Intelligent Search Strategies* - admissibility and consistency.
5. aimacode/aima-python - `astar_search`, `greedy_best_first_graph_search`. [GitHub](https://github.com/aimacode/aima-python)



