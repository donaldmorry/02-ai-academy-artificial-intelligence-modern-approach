# Section 3.3: Uninformed Search

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 3.3-3.4  
> **Previous:** [Section 3.2 - Example Problems](./section-02-example-problems.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Search Strategies Overview

A [search algorithm](../../GLOSSARY.md#search-algorithm) takes a formulated problem and returns a **solution path** (or failure). Strategies differ in **which frontier node they expand next**. **Uninformed** (blind) search uses only the problem definition - no estimate of distance to goal.

| Strategy | Expand order | Uses path cost? |
|----------|--------------|-----------------|
| **BFS** | Shallowest first | No (implicit: all steps cost 1) |
| **DFS** | Deepest first | No |
| **UCS** | Lowest $g(n)$ first | Yes |
| **Depth-limited DFS** | DFS with cap $l$ | No |
| **Iterative deepening** | DFS with increasing limits | No |

> **Readable form:** Uninformed search explores the state space without guessing how close you are to the goal - only structure and costs matter.

This section covers each strategy, when to use it, and how **depth limits** and **iterative deepening** fix DFS weaknesses.

---

## Measuring Search Performance

Before algorithms, fix evaluation criteria ([Section 3.7](./section-07-search-analysis.md) expands):

| Property | Meaning |
|----------|---------|
| **Completeness** | Finds solution if exists (finite space, step cost $> \epsilon$ for UCS) |
| **Optimality** | Finds lowest-cost solution |
| **Time** | Nodes generated or expanded |
| **Space** | Maximum nodes stored in frontier + explored |

Let $b$ = branching factor, $d$ = depth of shallowest solution, $m$ = maximum depth, $C^*$ = optimal solution cost.

---

## Breadth-First Search (BFS)

BFS expands the **shallowest** unexpanded node first - implemented with a **FIFO queue** (frontier).

```
Expand order (tree, b=2):     Level 0: s0
                              Level 1: A, B
                              Level 2: C, D, E, F
                              ...
```

### Algorithm Sketch

```
function BFS(problem):
    node ← NODE(problem.initial)
    if problem.goal_test(node.state): return solution(node)
    frontier ← QUEUE(node)
    explored ← empty set
    while frontier not empty:
        node ← POP(frontier)           # FIFO
        explored.add(node.state)
        for each child of node:
            if child.state not in explored and not in frontier:
                if problem.goal_test(child.state): return solution(child)
                ADD(frontier, child)   # enqueue
    return FAILURE
```

> **Readable form:** Explore all nodes at depth 1, then depth 2, and so on - like ripples spreading outward.

### Properties

| Property | BFS |
|----------|-----|
| Complete? | Yes (finite state space, graph search) |
| Optimal? | Yes if step cost is uniform |
| Time | $O(b^d)$ |
| Space | $O(b^d)$ - **memory bottleneck** |

BFS on the [8-puzzle](./section-02-example-problems.md) at depth 20 may need to store millions of nodes. Space, not time, often kills BFS.

### When to Use BFS

- Step costs are **equal** (or nearly equal)
- Solution depth $d$ is **moderate**
- You need the **shortest number of steps**
- State space is small (vacuum world, tiny grids)

---

## Depth-First Search (DFS)

DFS expands the **deepest** unexpanded node first - **LIFO stack** (or recursive).

```
Go deep: s0 → A → C → G (goal?) 
Backtrack if dead end, try sibling D...
```

### Algorithm Sketch

```
function DFS(problem):
    frontier ← STACK(NODE(problem.initial))
    explored ← empty set
    while frontier not empty:
        node ← POP(frontier)           # LIFO
        if problem.goal_test(node.state): return solution(node)
        explored.add(node.state)
        for each child of node (reverse order optional):
            if child.state not in explored and not in frontier:
                PUSH(frontier, child)
    return FAILURE
```

### Properties

| Property | DFS |
|----------|-----|
| Complete? | No in infinite spaces; yes in finite with graph search |
| Optimal? | No |
| Time | $O(b^m)$ - may explore deep dead ends |
| Space | $O(bm)$ - **linear in depth** |

DFS uses far less memory than BFS but may return **long, suboptimal** paths and get lost in deep branches.

### When to Use DFS

- Memory is severely limited
- Many solutions exist near deep leaves
- Path quality matters less than finding **any** solution quickly
- Combined with **backtracking** in CSPs ([Chapter 06](../chapter-06-constraint-satisfaction/README.md))

---

## Depth-Limited Search

Plain DFS with maximum depth $l$. Nodes at depth $l$ are treated as leaves - no expansion.

$$
\text{If solution depth } d > l \Rightarrow \text{returns FAILURE (incomplete)}
$$
> **Readable form:** DFS stops after l steps down any branch - may miss deeper goals.

| Property | Depth-limited DFS |
|----------|-------------------|
| Complete? | Only if $l \geq d$ |
| Optimal? | No |
| Time | $O(b^l)$ |
| Space | $O(bl)$ |

Choosing $l$ requires domain knowledge. Too small → miss solution. Too large → DFS drawbacks return.

---

## Iterative Deepening Search (IDS)

IDS runs depth-limited DFS with $l = 0, 1, 2, \ldots$ until a goal is found.

```
l=0: expand s0 only
l=1: expand to depth 1
l=2: expand to depth 2  → goal found
```

### Why Not Just Use BFS?

IDS **repeats** work at shallow depths - seems wasteful. But for large $b$ and $d$:

- Time remains $O(b^d)$ - shallow levels contribute exponentially less
- Space is $O(bd)$ - **like DFS**, unlike BFS

For $b = 10$, $d = 5$, repeated shallow expansions add only ~10% overhead (AIMA analysis).

| Property | IDS |
|----------|-----|
| Complete? | Yes (finite space, uniform costs) |
| Optimal? | Yes (uniform step costs) |
| Time | $O(b^d)$ |
| Space | $O(bd)$ |

**IDS is the preferred uninformed method** when step costs are uniform and memory is tight - chess endgames, large puzzles, game tree setups.

---

## Uniform-Cost Search (UCS)

UCS expands the node with **lowest path cost** $g(n)$ from the start - **priority queue** ordered by $g$.

$$
g(n) = \text{cost of path from initial state to } n
$$
> **Readable form:** g(n) = total cost paid so far to reach node n.

When all step costs are equal, UCS behaves like BFS. When costs vary ([Romania map](./section-02-example-problems.md)), UCS finds **cheapest** routes.

### Algorithm Sketch

```
function UCS(problem):
    node ← NODE(problem.initial, g=0)
    frontier ← PRIORITY_QUEUE(node, key=g)
    explored ← map state → best_g
    while frontier not empty:
        node ← POP_MIN(frontier)
        if problem.goal_test(node.state): return solution(node)
        if node.state in explored and explored[node.state] ≤ node.g:
            continue
        explored[node.state] ← node.g
        for each child with step_cost c:
            child.g ← node.g + c
            if child.state not explored or child.g < explored[child.state]:
                ADD(frontier, child)
    return FAILURE
```

### Properties

| Property | UCS |
|----------|-----|
| Complete? | Yes if every step cost $\geq \epsilon > 0$ |
| Optimal? | Yes |
| Time | $O(b^{1 + \lfloor C^* / \epsilon \rfloor})$ |
| Space | Same order as time |

UCS is **Dijkstra's algorithm** on the implicit state graph. Same optimality proof: first goal pop has minimum $g$.

### Romania Example

From Arad, UCS expands in order of increasing $g$: Zerind (75), Sibiu (140), Timisoara (118) - priority queue order, not alphabetical. Eventually reaches Bucharest at $g = 418$.

---

## Comparing Uninformed Strategies

| Algorithm | Complete | Optimal | Time | Space |
|-----------|----------|---------|------|-------|
| BFS | Yes* | Yes† | $O(b^d)$ | $O(b^d)$ |
| DFS | Finite* | No | $O(b^m)$ | $O(bm)$ |
| Depth-limited | If $l \geq d$ | No | $O(b^l)$ | $O(bl)$ |
| IDS | Yes* | Yes† | $O(b^d)$ | $O(bd)$ |
| UCS | Yes‡ | Yes | $O(b^{C^*/\epsilon})$ | $O(b^{C^*/\epsilon})$ |

\*Finite graph with cycle checking. †Uniform step costs. ‡Step cost lower bound $\epsilon > 0$.

---

## Tree Search vs. Graph Search

**Tree search:** No memory of visited states - can revisit states on different paths.

**Graph search:** Maintain **explored set** (or best $g$ per state) - never re-expand worse paths to same state.

For Romania, reaching Sibiu via Arad (140) vs. longer detour - graph search keeps cheapest $g$.

[Section 3.6](./section-06-search-implementations.md) implements graph search with hash sets.

---

## Choosing an Uninformed Algorithm

```
Uniform step costs?
  ├─ Yes → Need optimality + low memory? → IDS
  │         Memory OK? → BFS
  └─ No  → UCS (weighted graph)

Need any solution fast, memory tight, optimality optional? → DFS (+ depth limit)
```

For [Romania](./section-02-example-problems.md): **UCS** (or A* in [Section 3.4](./section-04-informed-search.md)). For 8-puzzle with unit costs: **IDS** or BFS if memory allows.

---

## Connection to Course 1

Graph traversals in ML pipelines (dependency graphs, feature DAGs) use the same FIFO/LIFO discipline. UCS parallels **shortest-path** routing in network tools - Dijkstra with explicit priority queues.

---

## Common Misconceptions

**"DFS always uses less time."** DFS time can be $O(b^m)$ with $m \gg d$ - much worse than BFS when shallow solutions exist.

**"BFS is always optimal."** Only with **uniform** step costs. On Romania with different road lengths, BFS by **depth** is not optimal - use UCS.

**"IDS is slow because it repeats."** Overhead is modest; memory savings often dominate for large $d$.

---

## Reflection Questions

1. Why does UCS fail to guarantee termination if zero-cost cycles exist?
2. For 8-puzzle, compare BFS vs. IDS on time and space when $d = 18$, $b = 3$.
3. When is depth-limited DFS complete without iterative deepening?
4. Implement trace: UCS from Arad - which city is expanded first, second, third?
5. How does graph search change DFS completeness in infinite trees?

---

**Next:** [Section 3.4 - Informed Search](./section-04-informed-search.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Sections 3.3-3.4. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Dijkstra, E. W. (1959). A note on two problems in connexion with graphs. *Numerische Mathematik*.
3. Korf, R. E. - iterative deepening analysis for IDA*. [CMU](https://www.cs.cmu.edu/)
4. aimacode/aima-python - `breadth_first_graph_search`, `uniform_cost_search`. [GitHub](https://github.com/aimacode/aima-python)
5. CLRS - priority-queue analysis for Dijkstra/UCS. *Introduction to Algorithms*.
