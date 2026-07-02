# Lab 07: Wumpus World Inference Agent

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 7  
> **Prerequisites:** Sections 7.1-7.7  
> **Estimated time:** 3-5 hours  
> **Tools:** Python 3.10+, no external libraries required  
> **Assessment:** [Shared Assessment Appendix](../../ASSESSMENT_APPENDIX.md)

---

## Lab Objective

Build a small knowledge-based agent for a 4x4 Wumpus World. The goal is not to build a perfect game player; the goal is to connect percepts, propositional sentences, entailment, and safe action selection.

By the end, you should be able to:

1. Encode pits, breezes, the Wumpus, stenches, and safe squares as propositions.
2. Add percepts to a knowledge base after each move.
3. Infer whether neighboring squares are safe, risky, or unknown.
4. Choose a conservative next action based on entailed safety.
5. Explain one failure mode caused by incomplete knowledge.

---

## Part A: Representation

Use coordinates `(x, y)` with `(1, 1)` as the start. Define proposition names as strings:

```python
def pit(x, y): return f"P_{x}_{y}"
def breeze(x, y): return f"B_{x}_{y}"
def wumpus(x, y): return f"W_{x}_{y}"
def stench(x, y): return f"S_{x}_{y}"
def safe(x, y): return f"Safe_{x}_{y}"


def neighbors(x, y, n=4):
    for nx, ny in [(x-1, y), (x+1, y), (x, y-1), (x, y+1)]:
        if 1 <= nx <= n and 1 <= ny <= n:
            yield nx, ny
```

Your KB may be a full propositional logic engine, a set of Horn rules, or a carefully documented custom inference procedure.

---

## Part B: Local Rules

Encode at least these rules:

| Percept fact | Inference |
|--------------|-----------|
| No breeze at `(x,y)` | No adjacent square contains a pit |
| Breeze at `(x,y)` | At least one adjacent square may contain a pit |
| No stench at `(x,y)` | No adjacent square contains the Wumpus |
| Stench at `(x,y)` | At least one adjacent square may contain the Wumpus |
| No pit and no Wumpus at a square | The square is safe |

You can start with conservative negative inference:

```python
class WumpusKB:
    def __init__(self, n=4):
        self.n = n
        self.no_pit = set()
        self.no_wumpus = set()
        self.visited = {(1, 1)}

    def tell_percept(self, x, y, has_breeze, has_stench):
        self.visited.add((x, y))
        if not has_breeze:
            self.no_pit.update(neighbors(x, y, self.n))
        if not has_stench:
            self.no_wumpus.update(neighbors(x, y, self.n))

    def is_entailed_safe(self, square):
        return square in self.no_pit and square in self.no_wumpus
```

---

## Part C: Safe Action Policy

Implement `choose_action(current, kb)`:

1. Prefer unvisited neighboring squares entailed safe.
2. If none exist, return to a previously visited safe square.
3. If neither is available, stop and explain what is unknown.

```python
def choose_action(current, kb):
    x, y = current
    candidates = list(neighbors(x, y, kb.n))
    for square in candidates:
        if square not in kb.visited and kb.is_entailed_safe(square):
            return ("move", square)
    for square in candidates:
        if square in kb.visited:
            return ("backtrack", square)
    return ("stop", "no entailed safe move")
```

---

## Part D: Test Trace

Run this trace and record the KB state after each percept:

| Step | Location | Breeze? | Stench? | Expected inference |
|------|----------|---------|---------|--------------------|
| 1 | `(1,1)` | No | No | `(1,2)` and `(2,1)` are safe |
| 2 | `(2,1)` | Yes | No | Wumpus is not adjacent to `(2,1)` |
| 3 | `(1,2)` | No | Yes | Pit is not adjacent to `(1,2)` |

Deliver a table with:

- Known safe squares.
- Known no-pit squares.
- Known no-Wumpus squares.
- Candidate next action.
- One sentence explaining why that action is rational.

---

## Deliverables

- `wumpus_kb.py` or notebook cells implementing the KB and action policy.
- One trace table over at least three percepts.
- One counterexample where the agent cannot prove a square safe even if it is actually safe.
- One paragraph comparing this lab with search in Chapter 03 and CSPs in Chapter 06.

---

## Reflection

1. Why is "not known unsafe" different from "entailed safe"?
2. What rule would require disjunction or full model checking?
3. How would first-order logic reduce repeated proposition templates?
4. What makes Wumpus World a useful bridge from agents to logic?

---

## References

- Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 7.
- [Chapter 07 README](./README.md)
