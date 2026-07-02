# Lab 11: STRIPS Planning in Blocks World

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11  
> **Prerequisites:** Sections 11.1-11.4  
> **Estimated time:** 4-6 hours  
> **Tools:** Python 3.10+, optional PDDL planner  
> **Assessment:** [Shared Assessment Appendix](../../ASSESSMENT_APPENDIX.md)

---

## Lab Objective

Implement a tiny classical planner for Blocks World using STRIPS-style actions. This lab makes planning concrete: states are sets of facts, actions have preconditions and effects, and search finds an action sequence from the initial state to the goal.

By the end, you should be able to:

1. Represent states as sets of symbolic facts.
2. Define STRIPS actions with preconditions, add effects, and delete effects.
3. Run forward breadth-first search over plans.
4. Explain why action schemas need grounding.
5. Compare your planner with PDDL-style domain descriptions.

---

## Part A: Action Representation

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Action:
    name: str
    pre: frozenset[str]
    add: frozenset[str]
    delete: frozenset[str]


def applicable(action, state):
    return action.pre <= state


def apply(action, state):
    return (state - action.delete) | action.add
```

Use facts such as `on(A,B)`, `clear(A)`, `ontable(A)`, and `holding(A)`. Keep facts as strings so the lab focuses on planning rather than parsing.

---

## Part B: Ground Actions

Create grounded actions for three blocks: `A`, `B`, and `C`.

| Schema | Preconditions | Add effects | Delete effects |
|--------|---------------|-------------|----------------|
| `pickup(x)` | `ontable(x)`, `clear(x)`, `handempty` | `holding(x)` | `ontable(x)`, `clear(x)`, `handempty` |
| `putdown(x)` | `holding(x)` | `ontable(x)`, `clear(x)`, `handempty` | `holding(x)` |
| `unstack(x,y)` | `on(x,y)`, `clear(x)`, `handempty` | `holding(x)`, `clear(y)` | `on(x,y)`, `clear(x)`, `handempty` |
| `stack(x,y)` | `holding(x)`, `clear(y)` | `on(x,y)`, `clear(x)`, `handempty` | `holding(x)`, `clear(y)` |

---

## Part C: Breadth-First Planner

```python
from collections import deque


def plan(initial, goal, actions, max_depth=12):
    initial = frozenset(initial)
    goal = frozenset(goal)
    frontier = deque([(initial, [])])
    seen = {initial}

    while frontier:
        state, path = frontier.popleft()
        if goal <= state:
            return path
        if len(path) >= max_depth:
            continue
        for action in actions:
            if applicable(action, state):
                nxt = frozenset(apply(action, state))
                if nxt not in seen:
                    seen.add(nxt)
                    frontier.append((nxt, path + [action.name]))
    return None
```

Test goal:

```python
initial = {
    "ontable(A)", "ontable(B)", "ontable(C)",
    "clear(A)", "clear(B)", "clear(C)", "handempty",
}
goal = {"on(A,B)", "on(B,C)"}
```

---

## Part D: Analysis

Record:

| Question | Your answer |
|----------|-------------|
| Plan found | |
| Plan length | |
| Number of grounded actions | |
| First impossible action you tested | |
| One repeated state avoided by `seen` | |

Then change the goal to `on(C,B)` and `on(B,A)`. Compare plan length and explain what changed.

---

## Deliverables

- Grounded action generator or manually listed grounded actions.
- Planner output for two goals.
- One trace showing state facts before and after a `stack` or `unstack`.
- One paragraph explaining how PDDL would separate domain schema from problem instance.

---

## Reflection

1. Why does STRIPS assume deterministic actions?
2. What happens if delete effects are omitted?
3. Why does planning become difficult as objects increase?
4. Which heuristic from search could help this planner?

---

## References

- Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11.
- [Chapter 11 README](./README.md)
