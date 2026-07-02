# Section 3.5: Heuristic Functions

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 3.5-3.6  
> **Previous:** [Section 3.4 - Informed Search](./section-04-informed-search.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## What Makes a Heuristic Useful?

[A* search](./section-04-informed-search.md) is only as good as its [heuristic](../../GLOSSARY.md#heuristic) $h(n)$. Two heuristics can both be [admissible](../../GLOSSARY.md#admissible-heuristic) yet expand vastly different numbers of nodes. The **quality** of $h$ determines practical performance.

**Effective heuristic:** closer to $h^*(n)$ without crossing the admissibility line.

$$
\text{Informative heuristic: } h_2(n) > h_1(n) \text{ for most } n, \text{ both admissible} \Rightarrow h_2 \text{ expands fewer nodes}
$$
> **Readable form:** Among optimistic guesses, bigger (tighter) estimates that still never overshoot tend to search less.

Designing $h$ is where **domain knowledge** enters classical AI - maps, physics, problem structure.

---

## Domain Knowledge in Heuristics

| Domain | Knowledge exploited | Heuristic |
|--------|---------------------|-----------|
| Route finding | Euclidean geometry | Straight-line (SL) distance |
| 8-puzzle | Tiles move on grid | Manhattan distance |
| 8-puzzle | Exact subproblems | Pattern database |
| Robot arm | Kinematics | Minimum joint rotations |
| Scheduling | Precedence constraints | Critical path length |

From [PEAS](../../GLOSSARY.md#peas) ([Section 2.3](../chapter-02-intelligent-agents/section-03-the-nature-of-environments.md)): sensors and environment model determine **what information** is available to compute $h$. Romania GPS coordinates enable SL; abstract puzzle grid enables Manhattan.

---

## Romania: Straight-Line Distance

For goal Bucharest, define:

$$
h_{\text{SL}}(n) = \text{straight-line distance from city } n \text{ to Bucharest}
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

AIMA provides coordinates; precompute pairwise SL to goal.

**Admissibility argument:** Roads follow terrain - no straight road shorter than the crow flies. Therefore $h_{\text{SL}}(n) \leq h^*(n)$.

**Consistency argument:** Triangle inequality on Euclidean plane: for any edge $(n, n')$ with road length $c$, $c \geq \text{SL}(n, n') \geq |h_{\text{SL}}(n) - h_{\text{SL}}(n')|$ - satisfies $h(n) \leq c + h(n')$.

```python
# Simplified SL heuristic (coordinates in km, x/y arbitrary)
COORDS = {"Arad": (91, 492), "Bucharest": (400, 327), ...}

def h_SL(city, goal="Bucharest"):
    x1, y1 = COORDS[city]
    x2, y2 = COORDS[goal]
    return ((x1 - x2)**2 + (y1 - y2)**2) ** 0.5
```

[Section 3.8](./section-08-romania-a-lab.md) uses these values in a full trace.

---

## 8-Puzzle: Manhattan Distance

For each tile $t$ (except blank), count **grid steps** tile must move to goal position, summing over tiles:

$$
h_{\text{Man}}(n) = \sum_{\text{tiles } t} \left| \Delta x_t \right| + \left| \Delta y_t \right|
$$
> **Readable form:** Manhattan = sum of horizontal + vertical distances each tile is away from its goal cell.

Example: tile 5 one row up, two columns left of goal → contributes $1 + 2 = 3$.

### Why Manhattan Is Admissible

Each move slides **one** tile **one** square - reduces Manhattan sum by **at most 1**. Need at least $h_{\text{Man}}$ moves → true cost $h^* \geq h_{\text{Man}}$.

### Why Manhattan Is Consistent

Moving blank swaps one tile one step: Manhattan for that tile decreases by 1, others unchanged → $\Delta h \leq 1 = \text{step cost}$.

```python
GOAL_POS = {1:(0,0), 2:(1,0), 3:(2,0), 4:(0,1), 5:(1,1),
            6:(2,1), 7:(0,2), 8:(1,2)}  # 0 = blank

def h_manhattan(state):
    total = 0
    for i, tile in enumerate(state):
        if tile == 0:
            continue
        x, y = i % 3, i // 3
        gx, gy = GOAL_POS[tile]
        total += abs(x - gx) + abs(y - gy)
    return total
```

---

## Misplaced Tiles Heuristic

Count tiles not in goal position (exclude blank):

$$
h_{\text{mis}}(n) = \#\{\text{tiles in wrong position}\}
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

**Admissible:** each move fixes at most one tile → $h^* \geq h_{\text{mis}}$.

**Weaker than Manhattan:** $h_{\text{mis}}(n) \leq h_{\text{Man}}(n)$ typically - Manhattan dominates misplaced tiles.

| Heuristic | Typical A* expansions (8-puzzle) |
|-----------|-------------------------------|
| $h = 0$ (UCS) | Very large |
| Misplaced tiles | Moderate |
| Manhattan | Small |

---

## Dominance and Combining Heuristics

**Definition:** $h_2$ **dominates** $h_1$ if for all $n$:

$$
h_2(n) \geq h_1(n) \quad \text{and both admissible}
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

Dominating heuristic **expands no more nodes** than dominated one (A* with $h_2$ ⊆ expansions of $h_1$).

**Max heuristic:**

h(n) = \max(h_1(n), h_2(n), \ldots, h_k(n))
$$

If each $h_i$ admissible, $h$ is admissible and **at least as informed** as any component.

$$
> **Readable form:** Take the largest optimistic estimate among several - still safe, never worse than the best single heuristic.

---

## Relaxed Problems

Powerful **automatic** heuristic design:

1. Start with original problem constraints
2. **Relax** (remove) some constraints → easier problem
3. Cost of optimal solution to relaxed problem = admissible $h$ for original

| Original | Relaxation | Heuristic |
|----------|------------|-----------|
| 8-puzzle (one tile move) | Tile moves anywhere | Misplaced tiles variant |
| 8-puzzle | Tile slides without blocking | Manhattan (tiles move independently) |
| Route finding | No road graph - fly direct | Straight-line distance |
| N-queens | No diagonal attacks | Sum of row conflicts only |

**Key theorem:** Optimal cost in **relaxed** problem $\leq$ optimal cost in **original** → admissibility.

---

## Pattern Databases (Advanced)

Precompute exact solution costs for **all** configurations of a tile subset (e.g., 7 tiles in 15-puzzle), store in lookup table.

$$
h_{\text{PDB}}(n) = \text{optimal cost to solve abstracted subpattern}
$$
> **Readable form:** The score compares vector representations; closer or more aligned vectors receive a better match score.

Sum of disjoint pattern databases can be **inadmissible**; **max** of pattern costs or **consistent merging** (e.g., Fringe, Korf) restores admissibility.

Used in record-breaking 15-puzzle and Rubik's cube solvers - beyond 8-puzzle scale but same principle.

---

## Learning Heuristics (Preview)

Modern systems **learn** $h$ from data ( neural networks on states). Learned heuristics may be inadmissible unless trained with constraints. Connects to:

- [Chapter 19](../chapter-19-learning-from-examples/README.md) - supervised learning
- [Chapter 05](../chapter-05-adversarial-search-games/README.md) - MCTS rollouts as heuristic
- [Chapter 22](../chapter-22-reinforcement-learning/README.md) - value functions as $h$

Classical AI emphasizes **provable** admissibility; learned heuristics trade proofs for raw speed.

---

## Heuristic Design Workflow

```
1. Identify what constraints make the problem hard
2. Propose relaxation - what if we ignore constraint C?
3. Replace optimal relaxed cost with closed form (Manhattan, SL)
4. Verify h(n) ≤ h*(n) - proof or argument
5. Check consistency for graph search
6. Compare dominance vs. simpler h - implement max()
7. Profile nodes expanded on benchmark instances
```

---

## PEAS: Warehouse Robot Heuristic Sketch

| PEAS | Detail |
|------|--------|
| **P** | Minimize time to deliver all packages |
| **E** | Grid warehouse, obstacles |
| **A** | Move, pick, drop |
| **S** | Position, inventory |

Possible $h$:

- **SL distance** to next delivery point (ignores shelves) - admissible if robot moves in straight lines in open floor
- **Manhattan** on grid without obstacles - admissible lower bound
- Add **+1 per undelivered package** - may need careful proof

Bad $h$: Euclidean distance **through walls** - inadmissible if walls block.

---

## Testing Heuristics

| Test | Method |
|------|--------|
| Admissibility | Compare $h(n)$ to known $h^*(n)$ on small solved instances |
| Consistency | Check $h(n) \leq c + h(n')$ for all edges |
| Informativeness | Count A* expansions vs. $h=0$ |
| Dominance | Verify $h_2(n) \geq h_1(n)$ pointwise |

On 8-puzzle, random 1000 solvable instances: plot expansions histogram for Manhattan vs. misplaced.

---

## Common Mistakes

**Using goal-dependent heuristics incorrectly.** $h$ must be defined relative to **fixed** goal in problem formulation.

**Double-counting distance.** Summing two full-path estimates may overestimate - use max or proven merge.

**Confusing heuristic with cost function.** Step cost $c$ is exact; $h$ is estimate only.

**Inadmissible pattern DB sums.** Adding abstract costs can exceed true distance.

---

## Worked Comparison: Two Admissible Heuristics

State: 8-puzzle start from [Section 3.2](./section-02-example-problems.md).

| Heuristic | Value at start (typical) |
|-----------|-------------------------|
| Misplaced | 5-7 |
| Manhattan | 12-14 |

Manhattan **dominates** misplaced → A* with Manhattan expands ≤ nodes. Both find optimal  moves when costs uniform.

---

## Reflection Questions

1. Prove Manhattan admissibility formally using "each move reduces $h$ by at most 1."
2. Design a relaxed problem for Romania whose optimal cost equals SL distance.
3. Why is $\max(h_1, h_2)$ admissible if $h_1, h_2$ are admissible?
4. Can a heuristic be consistent but not informative (always 0)?
5. How would partial observability change heuristic design? ([Chapter 04](../chapter-04-search-complex-environments/README.md))

---

**Next:** [Section 3.6 - Search Implementations](./section-06-search-implementations.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Sections 3.5-3.6. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Pearl, J. (1984). *Heuristics* - relaxed models and admissibility. [UCLA](https://ftp.cs.ucla.edu/pub/stat_ser/r119.pdf)
3. Korf, R. E. (1997). Finding optimal solutions to Rubik's Cube using pattern databases. *AAAI*.
4. Culberson, J. C., & Schaeffer, J. (1998). Pattern databases. *Computational Intelligence*.
5. aimacode/aima-python - `EightPuzzle`, `manhattan`, `romania_map`. [GitHub](https://github.com/aimacode/aima-python)


