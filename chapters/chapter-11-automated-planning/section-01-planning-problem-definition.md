# Section 11.1: Planning Problem Definition

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11.1
> **Previous:** [Chapter 10 - Knowledge Representation](../chapter-10-knowledge-representation/README.md)
> **Next:** [Section 11.2 - State-Space Planning](./section-02-state-space-planning.md)
> **Prerequisites:** [Chapter 10 - Knowledge Representation](../chapter-10-knowledge-representation/README.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## What Is Planning?

**Planning** constructs a **sequence of actions** transforming an **initial state** into a **goal state**. Unlike reactive agents, planners **look ahead** - search in space of possible futures.

[Chapter 03](../chapter-03-solving-problems-by-searching/README.md) introduced search; planning specializes it with **structured action representations** and **domain-independent algorithms**.

> **Readable form:** Search finds a path; planning finds a *program* of actions using knowledge about how actions change the world.

---

## Classical Planning Assumptions

| Assumption | Meaning |
|------------|---------|
| **Fully observable** | Know complete state |
| **Deterministic** | Action effects fixed |
| **Static** | Only planner's actions change world |
| **Discrete** | Finite states, instant actions |
| **Initial-state goals** | Goal is conjunction of fluents |

Real robotics violates most of these - extensions in later sections and [Chapter 26](../chapter-26-robotics/README.md).

---

## States as Conjunctions of Fluents

**State:** set (conjunction) of ground fluents.

$$
At(Robot, A) \land Holding(Robot, Block) \land On(Block, Table)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

**Closed world** in planners: fluent false if not listed.

---

## STRIPS Representation

**STRIPS action** $a$:

- **Preconditions** $Pre(a)$ - fluents required
- **Add list** $Add(a)$ - fluents made true
- **Delete list** $Del(a)$ - fluents removed

**Result** of applying $a$ in state $s$:

$$
Result(s, a) = (s \setminus Del(a)) \cup Add(a) \quad \text{if } Pre(a) \subseteq s
$$
> **Readable form:** If preconditions hold, add new facts and remove deleted ones; everything else stays (frame assumption).

---

## Example: Blocks World

Action $Move(b, x, y)$ - move block $b$ from $x$ to $y$.

```
Pre: On(b,x), Clear(b), Clear(y)
Add: On(b,y), Clear(x)
Delete: On(b,x), Clear(y)
```

Initial: $On(A,Table), On(B,A), Clear(B), \ldots$

Goal: $On(A,B) \land On(B,C)$

---

## Planning vs. General Search

| | General search ([Chapter 03](../chapter-03-solving-problems-by-searching/README.md)) | Planning |
|---|-----------------------------------------------------------------------------|----------|
| State | Arbitrary | Factored fluents |
| Successor | Domain-specific | STRIPS apply |
| Heuristics | Ad hoc | Relaxed planning graph |

Planning exploits **structure** - same algorithm on blocks world, logistics, gripper.

---

## Formal Definition

**Planning problem** $P = (S, s_0, G, A)$:

- $S$ - state space
- $s_0 \in S$ - initial state
- $G \subseteq$ fluents - goal condition
- $A$ - actions with STRIPS specs

**Solution:** action sequence $\langle a_1,\ldots,a_n \rangle$ such that $s_n \models G$ where $s_{i+1} = Result(s_i, a_{i+1})$.

---

## Python: STRIPS Apply

```python
def apply_action(state, action):
    if not action.precond <= state:
        return None
    return (state - action.del_list) | action.add_list
```

---

## Gripper Domain

Robot with gripper; blocks on table.

Fluents: $At(block, location)$, $Holding(gripper, block)$, $Clear(block)$, $OnTable(block)$

Actions: $PickUp(b)$, $PutDown(b)$, $Move(b, from, to)$

Classic IPC benchmark - tests planner handling of arm constraints.

---

## ADL Extensions

**Action Description Language** allows:

- Conditional effects: when $Clear(b)$ then effect $E$
- Disjunctive preconditions
- Quantified preconditions

PDDL 2.x+ incorporates ADL features - richer than original STRIPS.

---

## Plan Validity

Sequence $\langle a_1,\ldots,a_n\rangle$ **valid** iff:

1. Each $a_i$ applicable in state $s_{i-1}$
2. $s_n \models G$
3. $s_{i} = Result(s_{i-1}, a_i)$

**Optimal plan** minimizes $\sum cost(a_i)$ in metric planning.

---

## Study Exercise: STRIPS Hand Simulation

Apply $PickUp(A)$, $Stack(A,B)$ manually on paper state. Track fluent set after each step.

Verify $Result$ matches your manual update - builds intuition before coding planners.

## Worked Example: Planning Problem Definition

Apply the ideas from **Initial state, goal, actions, STRIPS representation** to a small concrete instance. State assumptions explicitly, write the governing equations, and interpret each term. Compare your result with the AIMA chapter exercise that matches Chapter 11.1.

> **Readable form:** Translate the abstract definition into numbers or code for one toy case.


## Design Checklist

| Step | Action |
|------|--------|
| 1 | Identify variables and constraints for planning problem definition |
| 2 | Choose representation aligned with Initial state, goal, actions, STRIPS representation |
| 3 | Verify against AIMA Chapter 11.1 definitions |
| 4 | Implement minimal prototype in Python |
| 5 | Test edge cases and document failure modes |


## Connections Across Course 2

This section on **Planning Problem Definition** links forward to later chapters that reuse the same abstractions. Revisit [../../GLOSSARY.md](../../GLOSSARY.md) entries while reading, and cross-check notation with [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md). Prior search and logic chapters provide algorithmic scaffolding; probability chapters supply uncertainty semantics when Initial state, goal, actions, STRIPS representation involves partial information.


## Key Takeaways

1. **Planning Problem Definition** - core idea 1 from AIMA Chapter 11.1 (Initial state, goal, actions, STRIPS representation).
2. **Planning Problem Definition** - core idea 2 from AIMA Chapter 11.1 (Initial state, goal, actions, STRIPS representation).
3. **Planning Problem Definition** - core idea 3 from AIMA Chapter 11.1 (Initial state, goal, actions, STRIPS representation).
4. **Planning Problem Definition** - core idea 4 from AIMA Chapter 11.1 (Initial state, goal, actions, STRIPS representation).
5. **Planning Problem Definition** - core idea 5 from AIMA Chapter 11.1 (Initial state, goal, actions, STRIPS representation).


## Common Mistakes

- **Confusing syntax with semantics** in Planning Problem Definition - symbols must map to models or distributions.
- **Skipping units and domains** when Initial state, goal, actions, STRIPS representation involves continuous or structured variables.
- **Treating heuristics as guarantees** - always state when optimality or soundness holds.
- **Ignoring complexity** - tractability assumptions in Chapter 11.1 may fail at scale.
- **Omitting sensor/actuator noise** when deploying ideas from textbook toy domains.


## Deeper Reading

Re-read Russell & Norvig Chapter 11.1 with pen and paper. For **Planning Problem Definition**, derive each formula from first principles before using libraries. The aima-python repository implements many algorithms from this chapter - compare your pseudocode to the reference implementation and note API differences.

## Key Terms (Glossary)

- **Initial** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **state** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **goal** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **actions** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **STRIPS** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **representation** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Planning Problem Definition in AIMA?
2. How does Planning Problem Definition relate to Initial state, goal, actions, STRIPS representation?
3. Give a concrete application of Planning Problem Definition.
4. What assumptions does the textbook emphasize for Planning Problem Definition?
5. When would an alternative to Planning Problem Definition be preferable?
6. How would you verify understanding of Planning Problem Definition in code?

---

**Previous:** [Chapter 10 - Knowledge Representation](../chapter-10-knowledge-representation/README.md) · **Next:** [Section 11.2 - State-Space Planning](./section-02-state-space-planning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Ghallab, M., Nau, D., & Traverso, P. (2004). *Automated Planning*. Morgan Kaufmann.']
