# Section B.4: Data Structures

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Appendix B.4  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Structures Behind AI Algorithms

Efficient AI implementations depend on choosing the right **abstract data types**. Appendix B reviews structures referenced in AIMA pseudocode and their Python equivalents.

> **Readable form:** Data structures are containers - pick the right box and your search runs fast; pick wrong and it crawls.

---

## Queues and Stacks

**Queue (FIFO):**
- `ENQUEUE(q, item)` - $O(1)$ amortized
- `DEQUEUE(q)` - $O(1)$
- BFS frontier

**Stack (LIFO):**
- `PUSH(s, item)` - $O(1)$
- `POP(s)` - $O(1)$
- DFS frontier

```python
from collections import deque
queue = deque()
queue.append(item)       # enqueue
item = queue.popleft()   # dequeue

stack = []
stack.append(item)       # push
item = stack.pop()       # pop
```

---

## Priority Queues

**Operations:**
- `INSERT(pq, key, value)` - $O(\log n)$ with binary heap
- `EXTRACT-MIN(pq)` - $O(\log n)$
- `DECREASE-KEY` - $O(\log n)$ (required for efficient Dijkstra)

Used in: UCS, A*, Dijkstra, best-first search.

```python
import heapq
heap = []
heapq.heappush(heap, (priority, item))
priority, item = heapq.heappop(heap)
```

**Note:** Python `heapq` is min-heap; negate priorities for max-heap.

---

## Hash Tables (Dictionaries)

**Operations:** average $O(1)$ insert, lookup, delete.

Uses:
- `reached` set in graph search
- Memoization in dynamic programming
- Symbol tables in logic
- Word-to-index mappings in NLP

```python
reached = set()
if state not in reached:
    reached.add(state)

memo = {}
def fib(n):
    if n not in memo:
        memo[n] = fib(n-1) + fib(n-2) if n > 1 else n
    return memo[n]
```

**Worst case:** $O(n)$ with collisions - rare with good hash function.

---

## Heaps

**Binary heap** - complete binary tree with heap property:
- Min-heap: parent $\leq$ children
- Stored as array: parent at $i$, children at $2i$, $2i+1$

| Operation | Time |
|-----------|------|
| Insert | $O(\log n)$ |
| Extract-min | $O(\log n)$ |
| Peek-min | $O(1)$ |
| Build heap | $O(n)$ |

---

## Trees and Graphs

**Search tree:** nodes with parent pointers, state, path cost $g$, heuristic $h$.

**Graph:** vertices + edges - adjacency list (sparse) or matrix (dense).

```python
# Adjacency list
graph = {
    'A': ['B', 'C'],
    'B': ['D'],
    'C': ['D'],
    'D': []
}
```

**Union-Find** - Kruskal's MST, connected components - near $O(1)$ amortized with path compression.

---

## Disjoint Sets and Graphs in AI

| Structure | AI use |
|-----------|--------|
| Adjacency list | State-space graphs, Bayes nets |
| Priority queue | A*, Dijkstra |
| Hash set | Duplicate detection |
| Trie | Word dictionaries, prefix search |
| KD-tree | Nearest neighbor (k-NN) |

---

## Python Standard Library Map

| AIMA ADT | Python |
|----------|--------|
| Queue | `collections.deque` |
| Stack | `list` |
| Priority queue | `heapq` |
| Set | `set` |
| Dictionary | `dict` |
| Default dict | `collections.defaultdict` |
| Counter | `collections.Counter` |

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

## Key Takeaways

1. Queues support BFS; stacks support DFS - both $O(1)$ per operation.
2. Priority queues (binary heaps) enable UCS, A*, and Dijkstra in $O(\log n)$ per operation.
3. Hash tables provide average $O(1)$ lookup for reached sets and memoization.
4. Adjacency lists efficiently represent sparse graphs in search and reasoning.
5. Python's `deque`, `heapq`, `dict`, and `set` map directly to AIMA ADTs.

## Exercises

1. Implement a priority queue wrapper class with `insert` and `extract_min`.
2. Compare list-as-queue (pop(0)) vs. `deque` for BFS on large graphs.
3. Build adjacency list and matrix for same graph - when is each better?
4. Implement memoized recursive Fibonacci and measure speedup.
5. Trace heap array after inserting 5, 3, 7, 1 into min-heap.

---

**Previous:** [Section B.3 - Algorithm Catalog](./section-03-algorithm-catalog.md)

**Next:** [Section B.5 - Recursion Patterns](./section-05-recursion-patterns.md)