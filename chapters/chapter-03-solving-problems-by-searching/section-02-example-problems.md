# Section 3.2: Example Problems

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 3.2  
> **Previous:** [Section 3.1 - Problem-Solving Agents](./section-01-problem-solving-agents.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Why Canonical Examples Matter

Before comparing BFS, A*, and other [search algorithms](../../GLOSSARY.md#search-algorithm), we need **shared benchmarks**. Russell and Norvig use a small set of problems repeatedly because each stresses different aspects of formulation and search:

| Problem | Stresses |
|---------|----------|
| **Romania route finding** | Weighted graphs, geographic heuristics |
| **8-puzzle** | Large state space, combinatorics, admissible heuristics |
| **Vacuum world** | Tiny space, action costs, agent demos |
| **N-queens (as CSP later)** | Constraint structure ([Chapter 06](../chapter-06-constraint-satisfaction/README.md)) |

This section formalizes **Romania** and the **8-puzzle** - the two workhorses for the rest of Chapter 03.

> **Readable form:** Romania teaches route search with real distances; the 8-puzzle teaches sliding-block search with huge numbers of board layouts.

---

## Romania Route Finding

The **Romania map** from AIMA connects cities with road distances (kilometers). A tourist agent starts in **Arad** and must reach **Bucharest** (București).

### PEAS Specification

| PEAS | Detail |
|------|--------|
| **P** | Minimize total driving distance; reach Bucharest |
| **E** | Static map of cities and roads; no traffic, no breakdowns |
| **A** | Select adjacent city to drive toward |
| **S** | Perfect knowledge of current city (GPS abstraction) |

Environment class: fully observable, single-agent, deterministic, sequential, static, **discrete** (city-level abstraction).

### Problem Formulation

| Component | Romania formulation |
|-----------|---------------------|
| **Initial state** | Arad |
| **Actions** | Go to any city connected by a road from current city |
| **Transition model** | $\text{RESULT}(\text{Arad}, \text{to Zerind}) = \text{Zerind}$ |
| **Goal test** | Current city == Bucharest |
| **Path cost** | Sum of edge distances along route |

### Map Structure (AIMA Figure 3.9)

```
                    Oradea(71)----Zerind(75)----Arad(118)
                       |                            | \
                    (151)                        (140) (96)
                       |                            |   \
                   Cluj-Napoca                  Sibiu  Timisoara(118)
                       |                         / \      |
                    (99)                    (80) (97)   (111)
                       |                    /       \       |
                  Eforie(86)            Fagaras   Rimnicu   Lugoj(70)
                       |                  (99)    Vilcea      |
                    (87)                    |      (146)    (75)
                       |                    |        \        |
                  Hirsova                 Bucharest  Pitesti  Mehadia
                       |                    ^       /  \       |
                    (151)                   |  (138) (101)  (85)
                       |                    |        \        |
                   Urliceni-----------------+      Craiova--Drobeta
```

*(Distances on edges; simplified ASCII - see AIMA for exact graph.)*

Key adjacency from **Arad**: Zerind (75), Sibiu (140), Timisoara (118).

### State Space Size

If there are $|V| = 20$ cities, a state is **which city you are in** - at most 20 states, far smaller than paths. Search explores **paths**, not just cities; many routes can revisit logic via different orderings, but with **graph search** ([Section 3.6](./section-06-search-implementations.md)) each city is expanded once per cheapest path.

### Sample Solution Path

One path (not necessarily optimal):

$$
\text{Arad} \rightarrow \text{Sibiu} \rightarrow \text{Fagaras} \rightarrow \text{Bucharest}
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

Path cost: $140 + 99 + 211 = 450$ km (AIMA edge labels).

The **optimal** path (UCS or A* finds it):

$$
\text{Arad} \rightarrow \text{Sibiu} \rightarrow \text{Rimnicu Vilcea} \rightarrow \text{Pitesti} \rightarrow \text{Bucharest}
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

Cost: $140 + 80 + 97 + 101 = 418$ km.

[Section 3.8](./section-08-romania-a-lab.md) walks through A* step by step on this map.

---

## The 8-Puzzle

A 3×3 grid holds tiles 1-8 and one **blank**. Legal moves slide a tile adjacent to the blank into the blank position.

```
Start:                 Goal:
┌───┬───┬───┐         ┌───┬───┬───┐
│ 1 │ 2 │ 3 │         │ 1 │ 2 │ 3 │
├───┼───┼───┤         ├───┼───┼───┤
│ 4 │   │ 5 │         │ 4 │ 5 │ 6 │
├───┼───┼───┤         ├───┼───┼───┤
│ 7 │ 8 │ 6 │         │ 7 │ 8 │   │
└───┴───┴───┘         └───┴───┴───┘
```

### PEAS Specification

| PEAS | Detail |
|------|--------|
| **P** | Reach goal configuration in minimum moves |
| **E** | Puzzle board, fixed rules, no external interference |
| **A** | Slide tile into blank (Up/Down/Left/Right relative to blank) |
| **S** | Full board configuration (fully observable) |

### Problem Formulation

| Component | 8-puzzle formulation |
|-----------|---------------------|
| **Initial state** | Tuple or 3×3 array of tile positions |
| **Actions** | Move blank Up, Down, Left, Right (if legal) |
| **Transition model** | Swap blank with neighbor tile |
| **Goal test** | Board matches target permutation |
| **Path cost** | 1 per move (uniform step cost) |

Example state encoding:

```python
# Flat tuple: 0 = blank
start = (1, 2, 3, 4, 0, 5, 7, 8, 6)
goal  = (1, 2, 3, 4, 5, 6, 7, 8, 0)
```

### Branching Factor

The blank has 2-4 neighbors depending on position:

| Blank position | Legal moves |
|--------------|-------------|
| Corner | 2 |
| Edge (non-corner) | 3 |
| Center | 4 |

Average branching factor $b \approx 3$ for random states.

### State Space Size

9! / 2 = **181,440** reachable configurations (half of 9! are unreachable due to parity).

$$
|\mathcal{S}| = \frac{9!}{2} = 181{,}440
$$
> **Readable form:** About 181 thousand distinct solvable board layouts.

Compare to Romania's ~20 cities - the 8-puzzle state space is **vast** for uninformed search at depth ~20-30 moves. That motivates **heuristics** in [Sections 3.4-3.5](./section-04-informed-search.md).

### Parity and Reachability

Not every permutation is reachable from every other. Swapping two tiles (without blank) **flips parity**. Start and goal must have the **same parity** or no solution exists - a cheap goal test filter.

---

## Side-by-Side Comparison

| Aspect | Romania | 8-puzzle |
|--------|---------|----------|
| State | Current city (string) | Board permutation |
| $| \mathcal{S} |$ | ~20 | 181,440 |
| Typical $b$ | 2-4 | ~3 |
| Step costs | Variable (road km) | Uniform (1) |
| Natural heuristic | Straight-line distance to goal city | Manhattan distance of tiles |
| Graph structure | Explicit road network | Implicit via move rules |

---

## Vacuum World (Compact Reference)

Two-room vacuum from [Section 3.1](./section-01-problem-solving-agents.md):

| State | (location, dirt_A, dirt_B) |
| Actions | {Left, Right, Suck} |
| Goal | Both rooms clean |

Only **8 distinct world states** - useful for tracing BFS by hand. Too small for heuristic insight; good for debugging search code.

---

## Route Finding Variants

Real applications extend the Romania template:

| Variant | Change to formulation |
|---------|----------------------|
| **Multiple goals** | Goal test: any of $\{g_1, g_2, \ldots\}$ |
| **Time-dependent costs** | State includes time; costs vary |
| **Partial observability** | Belief-state search ([Chapter 04](../chapter-04-search-complex-environments/README.md)) |
| **Continuous motion** | Discretize or use hybrid planners ([Chapter 26](../chapter-26-robotics/README.md)) |

The five-tuple pattern still applies - only state and cost grow richer.

---

## Puzzle Family Generalizations

| Puzzle | State space (rough) |
|--------|---------------------|
| **15-puzzle** (4×4) | $\approx 1.3 \times 10^{13}$ |
| **Rubik's cube** | $\approx 4.3 \times 10^{19}$ |
| **15-puzzle with pattern DB** | Still hard - needs advanced heuristics |

The 8-puzzle is the **smallest nontrivial** sliding puzzle for teaching; 15-puzzle appears in research on **pattern databases** ([Section 3.5](./section-05-heuristic-functions.md)).

---

## Formulating Your Own Problem

Checklist when given an informal task:

1. **What is one state?** (Minimal sufficient summary)
2. **What actions change state?** (Inverse: what cannot change?)
3. **Are actions reversible?** (Affects bidirectional search in Chapter 04)
4. **What is step cost?** (Uniform vs. weighted)
5. **How do we detect goal?** (Exact test, not vague wish)
6. **Estimate $b$ and $d$** - predicts search difficulty ([Section 3.7](./section-07-search-analysis.md))

Example: **warehouse robot** picking from shelf grid.

| Component | Formulation |
|-----------|-------------|
| $s_0$ | (robot position, item locations, carrying?) |
| Actions | Move N/S/E/W, pick, drop |
| Cost | Time per move + pick penalty |
| Goal | All items at packing station |

[PEAS](../../GLOSSARY.md#peas) from [Section 2.3](../chapter-02-intelligent-agents/section-03-the-nature-of-environments.md) guides performance and sensor/actuator choices before this table.

---

## Python: Romania Graph Fragment

```python
ROMANIA = {
    "Arad": {"Zerind": 75, "Sibiu": 140, "Timisoara": 118},
    "Zerind": {"Arad": 75, "Oradea": 71},
    "Oradea": {"Zerind": 71, "Cluj-Napoca": 151},
    "Sibiu": {"Arad": 140, "Fagaras": 99, "Rimnicu Vilcea": 80, "Oradea": 151},
    "Fagaras": {"Sibiu": 99, "Bucharest": 211},
    "Rimnicu Vilcea": {"Sibiu": 80, "Pitesti": 97, "Craiova": 146},
    "Pitesti": {"Rimnicu Vilcea": 97, "Bucharest": 101, "Craiova": 138},
    "Bucharest": {"Fagaras": 211, "Pitesti": 101, "Giurgiu": 90, "Urziceni": 85},
    # ... remaining cities from AIMA map
}

def actions(city):
    return list(ROMANIA[city].keys())

def step_cost(city, neighbor):
    return ROMANIA[city][neighbor]
```

This adjacency dict is the **environment model** for all search code in [Section 3.6](./section-06-search-implementations.md).

---

## Common Misconceptions

**"Romania is just Dijkstra."** Uniform-cost search on Romania **is** Dijkstra's algorithm on an undirected weighted graph - same idea, agent framing adds goal-directed interpretation.

**"8-puzzle states are strings."** Any hashable encoding works; tuples of ints are standard for O(1) hashing in explored sets.

**"Bigger state space always means harder."** Structure matters: Romania has low branching; Rubik's cube has enormous space but clever group theory heuristics help.

---

## Reflection Questions

1. Formulate the 8-puzzle with **tile moves** (move blank implicitly) vs. **blank moves** - same state space?
2. What is the branching factor at the corner of the 8-puzzle board?
3. Write the five-tuple for Romania with goal **any coastal city** instead of Bucharest.
4. Why is parity important before running BFS on a random 8-puzzle instance?
5. Map Romania's PEAS properties to the six environment dimensions from [Section 2.3](../chapter-02-intelligent-agents/section-03-the-nature-of-environments.md).

---

**Next:** [Section 3.3 - Uninformed Search](./section-03-uninformed-search.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 3.2, Figure 3.9 (Romania). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Korf, R. E. (1997). Finding optimal solutions to Rubik's Cube using pattern databases. *AAAI*. [AAAI](https://www.aaai.org/)
3. aimacode/aima-python - `search.py`, Romania and 8-puzzle problems. [GitHub](https://github.com/aimacode/aima-python)
4. Hart, P. E., Nilsson, N. J., & Raphael, B. (1968). A formal basis for the heuristic determination of minimum cost paths. *IEEE Trans. SSC* - 8-puzzle as motivating example.
5. Wikipedia - **n-puzzle** parity and state-space size. [N-puzzle](https://en.wikipedia.org/wiki/N_puzzle)



