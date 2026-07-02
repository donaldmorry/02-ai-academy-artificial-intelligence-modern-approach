# Section B.6: Python Mapping

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Appendix B.6  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From AIMA to Python

Course labs and capstone projects implement AIMA algorithms in **Python**. This section maps pseudocode constructs to idiomatic Python 3, highlighting common porting pitfalls.

> **Readable form:** Here's how each AIMA pseudocode line becomes real Python - copy patterns, not just syntax.

---

## Assignment and Types

| AIMA | Python |
|------|--------|
| `x ← 5` | `x = 5` |
| `s ← {a, b, c}` | `s = {'a', 'b', 'c'}` or `set(['a','b','c'])` |
| `A[1..n]` | `A[0:n]` (0-based!) |
| `∞` | `float('inf')` |
| `NIL` / `None` | `None` |

---

## Control Flow

```python
# if/else
if x == goal:
    return x
elif x > 0:
    return positive(x)
else:
    return negative(x)

# for each
for x in collection:
    process(x)

for i in range(1, n + 1):  # AIMA: for i ← 1 to n
    ...

# while
while not goal(state):
    state = next_state(state)
```

---

## Functions and Classes

```python
def a_star_search(problem):
    """Returns solution node or None."""
    node = Node(problem.initial)
    frontier = [(f(node), node)]
    heapq.heapify(frontier)
    reached = {problem.initial}
    while frontier:
        _, node = heapq.heappop(frontier)
        if problem.is_goal(node.state):
            return node
        for child in expand(problem, node):
            if child.state not in reached:
                heapq.heappush(frontier, (f(child), child))
                reached.add(child.state)
    return None
```

Use `@dataclass` for Node, State objects (Python 3.7+).

---

## Priority Queue Pattern

AIMA `INSERT-INTO-QUEUE` / `POP`:

```python
import heapq

class PriorityQueue:
    def __init__(self):
        self._heap = []
        self._counter = 0  # tie-breaker for stable ordering

    def push(self, priority, item):
        heapq.heappush(self._heap, (priority, self._counter, item))
        self._counter += 1

    def pop(self):
        return heapq.heappop(self._heap)[2]

    def empty(self):
        return len(self._heap) == 0
```

---

## Graph Search Template

```python
from collections import deque

def bfs(start, goal_test, successors):
    frontier = deque([start])
    reached = {start}
    parent = {start: None}
    while frontier:
        state = frontier.popleft()
        if goal_test(state):
            return reconstruct_path(parent, state)
        for child in successors(state):
            if child not in reached:
                reached.add(child)
                parent[child] = state
                frontier.append(child)
    return None

def reconstruct_path(parent, goal):
    path = []
    while goal is not None:
        path.append(goal)
        goal = parent[goal]
    return list(reversed(path))
```

---

## NumPy for Linear Algebra

```python
import numpy as np

A = np.array([[1, 2], [3, 4]])
x = np.array([1, 0])
Ax = A @ x                    # matrix-vector
eigenvalues, V = np.linalg.eig(A)
U, s, Vt = np.linalg.svd(A)
```

Use NumPy for performance in ML and robotics labs.

---

## Testing Conventions

```python
import unittest

class TestBFS(unittest.TestCase):
    def test_simple_path(self):
        result = bfs((0,0), lambda s: s==(2,0), grid_successors)
        self.assertEqual(result[0], (0,0))
        self.assertEqual(result[-1], (2,0))

if __name__ == '__main__':
    unittest.main()
```

Course labs should include unit tests for core algorithm properties.

---

## Style Guidelines

| Guideline | Reason |
|-----------|--------|
| PEP 8 naming (`snake_case`) | Python convention |
| Type hints optional but helpful | `def search(problem: Problem) -> Optional[Node]` |
| Docstrings on public functions | Document inputs/outputs |
| Avoid global state | Pass parameters explicitly |
| Log debug info with `logging` | Not bare `print` in production code |

---

## Common Porting Bugs

| Bug | Fix |
|-----|-----|
| Off-by-one indexing | Translate 1-based AIMA to 0-based |
| Mutable default args | `def f(x, seen=None): seen = seen or set()` |
| Heap comparison of nodes | Use tie-breaker counter |
| Unhashable states in `set` | Convert to tuple or use `dict` |
| Infinite recursion | Iterative stack or increase limit |

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

## Key Takeaways

1. Map AIMA `←` to `=`, 1-based indexing to 0-based Python slices.
2. `heapq` implements AIMA priority queues with tie-breaker pattern.
3. `deque` and `dict`/`set` provide efficient BFS and reached-set operations.
4. NumPy handles linear algebra from Appendix A in implementations.
5. Unit tests and PEP 8 style are expected in course lab submissions.

## Exercises

1. Port AIMA recursive depth-limited search to iterative Python with explicit stack.
2. Implement `PriorityQueue` class and test with A* on 8-puzzle.
3. Write unit tests verifying BFS finds shortest path on small grid.
4. Fix off-by-one bug in code using `for i in range(1, len(A))` accessing `A[i+1]`.
5. Add type hints to graph search functions.

---

**Previous:** [Section B.5 - Recursion Patterns](./section-05-recursion-patterns.md)

**Next:** [Section B.7 - Online Resources](./section-07-online-resources.md)

---

## Applied Deep Dive

Use **Python Mapping** as a complete AIMA-style design exercise rather than a vocabulary item.

### Formulation Pass

1. Name the agent, environment, sensors, actuators, and performance measure.
2. State what is fully observable and what must be inferred.
3. Identify whether the task is episodic, sequential, deterministic, stochastic, single-agent, or multi-agent.
4. List the abstraction boundaries where the model deliberately ignores detail.

### Algorithm Pass

| Step | What to verify |
|------|----------------|
| Inputs | Units, coordinate frames, symbol vocabulary, or probability semantics |
| Update | Whether the computation preserves invariants such as normalization or consistency |
| Output | Whether the result is an action, belief state, plan, explanation, or metric |
| Complexity | What grows with states, actions, objects, observations, or time horizon |



### Failure Analysis

| Failure mode | Why it matters | Mitigation |
|--------------|----------------|------------|
| Wrong abstraction | The algorithm solves a clean problem, not the deployed one | Revisit the PEAS description and task environment |
| Hidden uncertainty | Deterministic code overcommits to noisy inputs | Track beliefs, confidence intervals, or fallback policies |
| Poor evaluation | A demo succeeds while edge cases fail | Use held-out scenarios and adversarial tests |
| Integration drift | Perception, planning, and control assumptions disagree | Log interfaces and validate each boundary |

### Mastery Task

Write a one-page technical memo for **Python Mapping**. Include the formal model, one algorithm choice, one rejected alternative, two measurable risks, and a test plan. This turns the source chapter from reading material into engineering judgment.
