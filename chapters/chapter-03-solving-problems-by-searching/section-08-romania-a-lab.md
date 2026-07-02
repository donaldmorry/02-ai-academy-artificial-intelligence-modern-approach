# Section 3.8: Romania A* Lab

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 3 (Romania example)  
> **Previous:** [Section 3.7 - Search Analysis](./section-07-search-analysis.md)  
> **Prerequisites:** [Sections 3.1-3.7](./section-01-problem-solving-agents.md)  
> **Estimated time:** 2-3 hours  
> **Tools:** Python 3.10+, optional Jupyter  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Lab Objectives

1. Run [A* search](../../GLOSSARY.md#a-star-search) on the [Romania map](./section-02-example-problems.md) from Arad to Bucharest
2. Trace expansion order with $f(n) = g(n) + h(n)$ and [straight-line heuristic](../../GLOSSARY.md#admissible-heuristic)
3. Verify **optimal path cost 418 km** and compare with UCS and greedy best-first
4. Instrument **nodes expanded** and relate to [complexity analysis](./section-07-search-analysis.md)
5. Connect implementation to [PEAS](../../GLOSSARY.md#peas) route-finding agent specification

### Key Formulas

$$
f(n) = g(n) + h(n), \quad h_{\text{SL}}(n) \leq h^*(n)
$$
> **Readable form:** Total estimated cost = cost so far + straight-line distance to Bucharest; SL never overestimates true road distance.

---

## Part A: Setup

Copy or import the Romania graph from [Section 3.6](./section-06-search-implementations.md):

```python
ROMANIA = {
    "Arad": {"Zerind": 75, "Sibiu": 140, "Timisoara": 118},
    "Zerind": {"Arad": 75, "Oradea": 71},
    "Oradea": {"Zerind": 71, "Sibiu": 151},
    "Sibiu": {"Arad": 140, "Fagaras": 99, "Rimnicu Vilcea": 80, "Oradea": 151},
    "Timisoara": {"Arad": 118, "Lugoj": 111},
    "Lugoj": {"Timisoara": 111, "Mehadia": 70},
    "Mehadia": {"Lugoj": 70, "Drobeta": 75},
    "Drobeta": {"Mehadia": 75, "Craiova": 120},
    "Craiova": {"Drobeta": 120, "Pitesti": 138, "Rimnicu Vilcea": 146},
    "Rimnicu Vilcea": {"Sibiu": 80, "Pitesti": 97, "Craiova": 146},
    "Fagaras": {"Sibiu": 99, "Bucharest": 211},
    "Pitesti": {"Rimnicu Vilcea": 97, "Bucharest": 101, "Craiova": 138},
    "Bucharest": {"Fagaras": 211, "Pitesti": 101},
}

COORDS = {
    "Arad": (91, 492), "Bucharest": (400, 327), "Craiova": (253, 288),
    "Fagaras": (305, 449), "Pitesti": (379, 410), "Rimnicu Vilcea": (220, 445),
    "Sibiu": (206, 372), "Timisoara": (94, 410), "Zerind": (108, 529),
    "Oradea": (196, 526), "Lugoj": (165, 379), "Mehadia": (168, 339),
    "Drobeta": (220, 290),
}

def h_SL(city, goal="Bucharest"):
    x1, y1 = COORDS[city]
    x2, y2 = COORDS[goal]
    return ((x1 - x2)**2 + (y1 - y2)**2) ** 0.5

# Precomputed SL distances to Bucharest (approximate, for reference)
SL_TO_BUCHAREST = {city: round(h_SL(city), 1) for city in COORDS}
```

**Question:** Why is SL distance [admissible](../../GLOSSARY.md#admissible-heuristic)?

---

## Part B: Manual A* Trace (First Expansions)

**Initial:** Arad, $g=0$, $h \approx 366$, $f = 366$.

| Step | Expand | $g$ | $h$ (SL) | $f$ | New frontier highlights |
|------|--------|-----|----------|-----|-------------------------|
| 0 | Arad | 0 | 366 | 366 | Zerind(75+374), Timisoara(118+329), Sibiu(140+253) |
| 1 | Sibiu | 140 | 253 | 393 | Rimnicu(220+193), Fagaras(239+176), Timisoara, Zerind |
| 2 | Rimnicu Vilcea | 220 | 193 | 413 | Pitesti(317+98), Fagaras, ... |
| 3 | Pitesti | 317 | 98 | 415 | Bucharest(418+0), ... |
| 4 | Bucharest | 418 | 0 | 418 | **Goal** |

**Optimal path:**

$$
\text{Arad} \xrightarrow{140} \text{Sibiu} \xrightarrow{80} \text{Rimnicu Vilcea} \xrightarrow{97} \text{Pitesti} \xrightarrow{101} \text{Bucharest}
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

Total: $140 + 80 + 97 + 101 = 418$ km.

**Suboptimal greedy path** (may expand Fagaras early):

$$
\text{Arad} \rightarrow \text{Sibiu} \rightarrow \text{Fagaras} \rightarrow \text{Bucharest} = 450 \text{ km}
$$
> **Readable form:** A* balances cost paid with estimated remaining; greedy follows only the map crow-fly direction and can pick 450 km route.

---

## Part C: Instrumented A* Implementation

```python
import heapq
from dataclasses import dataclass, field
from typing import Any, List, Optional, Tuple

@dataclass(order=True)
class PrioritizedNode:
    priority: float
    counter: int
    node: Any = field(compare=False)

class RomaniaLab:
    def __init__(self, initial="Arad", goal="Bucharest"):
        self.initial = initial
        self.goal = goal
        self.expanded_log = []

    def successors(self, city):
        for nxt, cost in ROMANIA[city].items():
            yield nxt, cost

    def astar(self, h_fn):
        start = self.initial
        frontier = []
        counter = 0
        heapq.heappush(frontier, PrioritizedNode(h_fn(start), counter, (start, None, 0, [])))
        best_g = {start: 0}
        expanded = 0

        while frontier:
            _, _, (state, parent, g, path) = heapq.heappop(frontier)
            if state in self.expanded_log or (state in best_g and g > best_g[state]):
                continue
            expanded += 1
            self.expanded_log.append((state, g, h_fn(state), g + h_fn(state)))
            path = path + [state]
            if state == self.goal:
                return path, g, expanded
            for nxt, cost in self.successors(state):
                g2 = g + cost
                if nxt not in best_g or g2 < best_g[nxt]:
                    best_g[nxt] = g2
                    counter += 1
                    f = g2 + h_fn(nxt)
                    heapq.heappush(frontier, PrioritizedNode(f, counter, (nxt, state, g2, path)))
        return None, float("inf"), expanded
```

Run:

```python
lab = RomaniaLab()
path, cost, n_exp = lab.astar(h_SL)
print("Path:", " → ".join(path))
print("Cost:", cost)
print("Expansions:", n_exp)
for city, g, h, f in lab.expanded_log:
    print(f"  {city:18} g={g:3.0f} h={h:6.1f} f={g+h:6.1f}")
```

**Expected:** Cost 418, path through Sibiu → Rimnicu Vilcea → Pitesti.

---

## Part D: Compare UCS, Greedy, A*

```python
def h_zero(city):
    return 0

def greedy_search(lab, h_fn):
    """Best-first with f = h only."""
    frontier = [(h_fn(lab.initial), lab.initial, 0, [lab.initial])]
    expanded = 0
    visited = set()
    while frontier:
        _, state, g, path = heapq.heappop(frontier)
        if state in visited:
            continue
        visited.add(state)
        expanded += 1
        if state == lab.goal:
            return path, g, expanded
        for nxt, cost in lab.successors(state):
            if nxt not in visited:
                heapq.heappush(frontier, (h_fn(nxt), nxt, g + cost, path + [nxt]))
    return None, 0, expanded

def ucs(lab):
    return lab.astar(h_zero)

for name, fn in [("UCS", lambda l: l.astar(h_zero)),
                 ("Greedy", lambda l: greedy_search(l, h_SL)),
                 ("A*", lambda l: l.astar(h_SL))]:
    lab = RomaniaLab()
    if name == "Greedy":
        path, cost, n = fn(lab)
    else:
        path, cost, n = fn(lab)
    print(f"{name:6} cost={cost} expansions={n} path={' → '.join(path)}")
```

| Algorithm | Expected cost | Typical expansions (approx) |
|-----------|---------------|----------------------------|
| UCS | 418 | More than A* |
| Greedy | 450 (often) | Fewest expansions, not optimal |
| A* | 418 | Fewest among optimal |

Fill in your measured expansion counts in lab report.

---

## Part E: PEAS Lab Report Section

Write a short PEAS block for this lab agent:

| PEAS | Your Romania A* agent |
|------|----------------------|
| **P** | Minimize driving distance (418 km optimal) |
| **E** | Static Romania map, full observability |
| **A** | "Drive to adjacent city" |
| **S** | Current city name |

Relate to [Section 2.3](../chapter-02-intelligent-agents/section-03-the-nature-of-environments.md) environment properties.

---

## Part F: Extension - 8-Puzzle (Optional)

Apply [Manhattan heuristic](./section-05-heuristic-functions.md) from [Section 3.6](./section-06-search-implementations.md):

```python
start = (1, 2, 3, 4, 0, 5, 7, 8, 6)
goal  = (1, 2, 3, 4, 5, 6, 7, 8, 0)
# Run astar_search(EightPuzzle(start), h_manhattan)
# Report expansions vs h_zero
```

Compare node counts - expect orders-of-magnitude reduction with Manhattan.

---

## Part G: Verification Checklist

- [ ] Optimal path cost = **418**
- [ ] Every expanded node satisfies $f = g + h$
- [ ] For all expanded edges: $h(n) \leq c + h(n')$ (consistency check sample)
- [ ] Greedy may return cost **450** - document suboptimality
- [ ] A* expansions ≤ UCS expansions

---

## Analysis Questions

1. List cities expanded by A* in your trace - how many vs. total cities in map?
2. Why does A* expand Bucharest only when $g=418$?
3. If we doubled all heuristic values (inadmissible), what could happen?
4. What is effective branching factor for A* on this small graph?
5. How would [performance measure](../../GLOSSARY.md#performance-measure) change if the goal were fastest time rather than shortest distance?

---

## Deliverables

1. **Trace table** - first 8 expansions with $g, h, f$
2. **Comparison table** - UCS vs Greedy vs A* (cost, expansions, path)
3. **PEAS paragraph** - 4-6 sentences
4. **Optional:** 8-puzzle expansion counts

---

## Common Pitfalls

**Wrong coordinate system.** SL values need not match road km exactly - only lower bound.

**Goal test before optimal $g$.** First time Bucharest is generated may not be optimal path without consistent $h$ and proper queue handling.

**Forgetting undirected edges.** Romania graph is bidirectional - include both directions in ROMANIA dict.

---

## Reflection Questions

1. Which single expansion most reduced the frontier toward Bucharest?
2. Could bidirectional search help on Romania? ([Chapter 04](../chapter-04-search-complex-environments/README.md))
3. How does this lab connect to [rational agent](../../GLOSSARY.md#rational-agent) framework?
4. Design a second admissible heuristic for Romania weaker than SL.
5. What changes if goal is Craiova instead of Bucharest?

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Figure 3.9, Romania A* example. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - `romania_map`, `astar_search`. [GitHub](https://github.com/aimacode/aima-python)
3. [Section 3.6](./section-06-search-implementations.md) - full Problem/Node classes
4. [Section 3.7](./section-07-search-analysis.md) - interpreting expansion counts
5. Berkeley CS188 Project 1 - classic search autograder. [CS188](https://inst.eecs.berkeley.edu/~cs188/)

---

**Chapter complete.** **Next chapter:** [Chapter 04 - Search in Complex Environments](../chapter-04-search-complex-environments/README.md)



