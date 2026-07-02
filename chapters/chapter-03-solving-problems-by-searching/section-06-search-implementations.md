# Section 3.6: Search Implementations

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 3 (algorithms)  
> **Previous:** [Section 3.5 - Heuristic Functions](./section-05-heuristic-functions.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Pseudocode to Python

[Sections 3.3-3.4](./section-03-uninformed-search.md) gave algorithm sketches. This section provides **production-style Python** aligned with AIMA's `aima-python` - reusable for [Romania](./section-02-example-problems.md), [8-puzzle](./section-02-example-problems.md), and custom problems.

Design goals:

1. **Problem abstraction** - same search code for all domains
2. **Graph search** - avoid re-expanding dominated paths
3. **Priority queues** - UCS and [A*](../../GLOSSARY.md#a-star-search) share infrastructure

> **Readable form:** Write search once; plug in different problems by swapping how states, actions, and costs are defined.

---

## Problem Interface

```python
class Problem:
    """Abstract search problem (AIMA-style)."""
    def initial_state(self):
        raise NotImplementedError

    def actions(self, state):
        """Return iterable of applicable actions."""
        raise NotImplementedError

    def result(self, state, action):
        """Transition model: RESULT(s, a)."""
        raise NotImplementedError

    def goal_test(self, state):
        raise NotImplementedError

    def action_cost(self, s, action, s_prime):
        """Step cost c(s, a, s'). Default unit cost."""
        return 1


class Node:
    """Search tree node."""
    __slots__ = ("state", "parent", "action", "path_cost", "depth")

    def __init__(self, state, parent=None, action=None, path_cost=0):
        self.state = state
        self.parent = parent
        self.action = action
        self.path_cost = path_cost
        self.depth = 0 if parent is None else parent.depth + 1

    def expand(self, problem):
        """Generate child nodes."""
        for action in problem.actions(self.state):
            s2 = problem.result(self.state, action)
            cost = problem.action_cost(self.state, action, s2)
            yield Node(s2, parent=self, action=action,
                       path_cost=self.path_cost + cost)

    def solution(self):
        return [node.action for node in self.path()[1:]]

    def path(self):
        node, path_rev = self, []
        while node:
            path_rev.append(node)
            node = node.parent
        return list(reversed(path_rev))
```

---

## Graph Search Core

BFS, UCS, and A* share the same skeleton: maintain a **frontier**, an **explored** map of best $g$ per state, expand until goal. See [Section 3.3](./section-03-uninformed-search.md) for DFS and iterative deepening - those use a LIFO stack or repeated depth limits instead of a priority queue.

```python
def breadth_first_search(problem):
    start = Node(problem.initial_state())
    frontier = FIFOQueue()
    frontier.add(start)
    explored = set()

    while frontier:
        node = frontier.pop()
        if problem.goal_test(node.state):
            return node
        if node.state in explored:
            continue
        explored.add(node.state)
        for child in node.expand(problem):
            if child.state not in explored and not frontier.contains(child.state):
                frontier.add(child)
    return None
```

*(Add `contains` to `FIFOQueue` or track `enqueued` states - see AIMA `aima-python`.)*

---

## Frontier with Priority Queue: UCS and A*

```python
import heapq
from collections import deque

class FIFOQueue:
    def __init__(self):
        self.q = deque()
        self.enqueued = set()

    def add(self, node):
        if node.state not in self.enqueued:
            self.q.append(node)
            self.enqueued.add(node.state)

    def pop(self):
        node = self.q.popleft()
        self.enqueued.discard(node.state)
        return node

    def contains(self, state):
        return state in self.enqueued

    def __bool__(self):
        return bool(self.q)


class PriorityQueue:
    """Min-heap frontier; tie-break by FIFO counter."""
    def __init__(self):
        self.heap = []
        self.counter = 0
        self.entry_finder = {}  # state -> (priority, node)

    def add(self, node, priority):
        state = node.state
        if state in self.entry_finder:
            old_pri, _ = self.entry_finder[state]
            if priority >= old_pri:
                return  # keep better path already queued
        self.counter += 1
        entry = (priority, self.counter, node)
        heapq.heappush(self.heap, entry)
        self.entry_finder[state] = (priority, node)

    def pop(self):
        while self.heap:
            priority, _, node = heapq.heappop(self.heap)
            state = node.state
            if self.entry_finder.get(state, (None, None))[1] is node:
                del self.entry_finder[state]
                return node
        raise IndexError("empty queue")

    def __bool__(self):
        return bool(self.heap)


def uniform_cost_search(problem):
    pq = PriorityQueue()
    start = Node(problem.initial_state())
    pq.add(start, priority=0)
    explored = {}

    while pq:
        node = pq.pop()
        if problem.goal_test(node.state):
            return node
        if node.state in explored and explored[node.state] <= node.path_cost:
            continue
        explored[node.state] = node.path_cost
        for child in node.expand(problem):
            pq.add(child, priority=child.path_cost)
    return None


def astar_search(problem, h):
    """A* with heuristic function h(state) -> nonnegative float."""
    pq = PriorityQueue()
    start = Node(problem.initial_state())
    pq.add(start, priority=h(start.state))
    explored = {}

    while pq:
        node = pq.pop()
        if problem.goal_test(node.state):
            return node
        if node.state in explored and explored[node.state] <= node.path_cost:
            continue
        explored[node.state] = node.path_cost
        for child in node.expand(problem):
            f = child.path_cost + h(child.state)
            pq.add(child, priority=f)
    return None
```

> **Readable form:** UCS uses path cost g as priority; A* uses f = g + h(state).

---

## Romania Problem Class

```python
ROMANIA = {
    "Arad": {"Zerind": 75, "Sibiu": 140, "Timisoara": 118},
    "Zerind": {"Arad": 75, "Oradea": 71},
    "Sibiu": {"Arad": 140, "Fagaras": 99, "Rimnicu Vilcea": 80},
    "Timisoara": {"Arad": 118, "Lugoj": 111},
    "Lugoj": {"Timisoara": 111, "Mehadia": 70},
    "Mehadia": {"Lugoj": 70, "Drobeta": 75},
    "Drobeta": {"Mehadia": 75, "Craiova": 120},
    "Craiova": {"Drobeta": 120, "Pitesti": 138, "Rimnicu Vilcea": 146},
    "Rimnicu Vilcea": {"Sibiu": 80, "Pitesti": 97, "Craiova": 146},
    "Fagaras": {"Sibiu": 99, "Bucharest": 211},
    "Pitesti": {"Rimnicu Vilcea": 97, "Bucharest": 101, "Craiova": 138},
    "Bucharest": {"Fagaras": 211, "Pitesti": 101},
    "Oradea": {"Zerind": 71, "Sibiu": 151},
}

COORDS = {
    "Arad": (91, 492), "Bucharest": (400, 327), "Craiova": (253, 288),
    "Fagaras": (305, 449), "Pitesti": (379, 410), "Rimnicu Vilcea": (220, 445),
    "Sibiu": (206, 372), "Timisoara": (94, 410), "Zerind": (108, 529),
    "Oradea": (196, 526), "Lugoj": (165, 379), "Mehadia": (168, 339),
    "Drobeta": (220, 290),
}


class RomaniaProblem(Problem):
    def __init__(self, initial="Arad", goal="Bucharest"):
        self.initial = initial
        self.goal = goal

    def initial_state(self):
        return self.initial

    def actions(self, state):
        return list(ROMANIA[state].keys())

    def result(self, state, action):
        return action

    def goal_test(self, state):
        return state == self.goal

    def action_cost(self, s, action, s_prime):
        return ROMANIA[s][action]


def h_SL(state, goal="Bucharest"):
    x1, y1 = COORDS[state]
    x2, y2 = COORDS[goal]
    return ((x1 - x2) ** 2 + (y1 - y2) ** 2) ** 0.5
```

---

## 8-Puzzle Problem Class

Use the `EightPuzzle` class and `h_manhattan` from [Section 3.5](./section-05-heuristic-functions.md). States are 9-tuples with `0` as blank; actions move the blank Up/Down/Left/Right. Pass to `astar_search(problem, h_manhattan)` and compare expansions against `h_zero` (UCS).

---

## Running Search

```python
prob = RomaniaProblem()
node = astar_search(prob, h=lambda s: h_SL(s))
if node:
    cities = [n.state for n in node.path()]
    cost = node.path_cost
    print("Route:", " → ".join(cities), " Cost:", cost)
```

For expansion counts, increment a counter inside the expand loop or wrap `Node.expand` temporarily ([Section 3.8 lab](./section-08-romania-a-lab.md)).

---

## Implementation Checklist

Hashable states for explored sets; tie-break priority heap with a counter; track best $g$ per state in UCS/A*; parent pointers for path reconstruction. Goal test on **pop** ensures optimal $g$ when $h$ is consistent.

---

## Testing Strategy

1. **Romania** - A* path cost must be **418** from Arad ([Section 3.8](./section-08-romania-a-lab.md))
2. **8-puzzle** - Manhattan expands far fewer nodes than $h = 0$
3. **Vacuum world** - BFS finds minimum-move solution by hand

---

## Reflection Questions

1. Why must `Node.state` be hashable for graph search?
2. Where does A* differ from UCS in the priority queue key only?
3. How would you modify `graph_search` for tree search?
4. Why tie-break heap with a counter?
5. Instrument BFS vs A* on same 8-puzzle - ratio of expansions?

---

**Next:** [Section 3.7 - Search Analysis](./section-07-search-analysis.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), online code. [aima-python](https://github.com/aimacode/aima-python)
2. Python `heapq` documentation - priority queue patterns. [docs.python.org](https://docs.python.org/3/library/heapq.html)
3. CLRS - priority queues and Dijkstra implementation notes.
4. Korf, R. E. - implementation of IDA* and memory-bounded search.
5. Berkeley CS188 project specs - classic autograder search problems. [CS188](https://inst.eecs.berkeley.edu/~cs188/)