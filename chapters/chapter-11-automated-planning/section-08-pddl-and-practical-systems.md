# Section 11.8: PDDL and Practical Systems

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11.8
> **Previous:** [Section 11.7 - Multiagent and Contingent Planning](./section-07-multiagent-and-contingent-planning.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## PDDL: Planning Domain Definition Language

**PDDL** standardizes STRIPS-like domains for planner interchange.

**Domain file:** types, predicates, actions.

**Problem file:** objects, initial state, goal.

> **Readable form:** One file format so any planner can read your blocks world and try to solve it.

---

## PDDL Example (Blocks World Fragment)

```lisp
(:action pick-up
  :parameters (?b - block)
  :precondition (and (clear ?b) (on-table ?b) (hand-empty))
  :effect (and (holding ?b) (not (clear ?b)) (not (on-table ?b))
               (not (hand-empty))))

(:action put-down
  :parameters (?b - block)
  :precondition (holding ?b)
  :effect (and (clear ?b) (on-table ?b) (hand-empty)
               (not (holding ?b))))
```

---

## Fast-Downward

**Fast-Downward** - dominant classical planner; **SAS+** translation, **landmark** heuristics, **lazy search**.

Pipeline: PDDL → finite-domain representation → heuristic search.

Competition winner on many IPC benchmarks.

---

## Domain Engineering

Good PDDL requires:

1. **Minimal fluents** - avoid redundant state variables
2. **Symmetric breaking** - reduce isomorphic states
3. **Action costs** - metric planning for optimal plans
4. **Typing** - prune invalid parameters

---

## Practical Systems

| System | Notes |
|--------|-------|
| Fast-Downward | Classical STRIPS/ADL |
| OPTIC | Temporal/numeric |
| Pyhop | HTN educational |
| ROS Plan | Robotics integration |

---

## Chapter 11 Synthesis

| Section | Topic |
|--------|-------|
| 11.1 | STRIPS problem definition |
| 11.2 | State-space search |
| 11.3 | GRAPHPLAN |
| 11.4 | SAT planning |
| 11.5 | HTN |
| 11.6 | Scheduling |
| 11.7 | Multiagent/contingent |
| 11.8 | PDDL & Fast-Downward |

**Next:** uncertainty and probability ([Chapter 12](../chapter-12-quantifying-uncertainty/README.md)).

---

## Lab Connection

Specify blocks world in PDDL; run Fast-Downward; compare plan length and search time across heuristics.

---

## PDDL Types Example

```
(:types block - object
        table - location)
```

Typing reduces invalid $On(A, B)$ where $B$ must be block - prunes search.

---

## Fast-Downward Heuristic Stack

1. Translate PDDL → SAS+
2. Compute **landmarks** (fluents that must be achieved)
3. **FF heuristic** from relaxed plan
4. **Lazy best-first** search with preferred operators

Dominates many IPC tracks.

---

## IPC Benchmark Families

| Domain | Feature |
|--------|---------|
| Blocks-world | Manipulation |
| Logistics | Transport |
| Rovers | Science data |
| Visit-all | Coverage |

Chapter lab uses blocks-world - compare heuristics on same domain file.

---

## Beyond Classical Planning

| Extension | Violates |
|-----------|----------|
| Sensing | Full observability |
| Nondeterministic effects | Determinism |
| Durative actions | Instantaneous |
| Numeric fluents | Discrete-only |

PDDL 3+ and hybrid planners address robotics gaps ([Chapter 26](../chapter-26-robotics/README.md)).

---

## Study Exercise: PDDL Lab

Run chapter lab: blocks-world PDDL with Fast-Downward. Record plan length, nodes expanded, time for `--search astar(lmcut())` vs. `lazy_greedy(ff())`.

Write one paragraph interpreting results.

## Worked Example: PDDL and Practical Systems

Apply the ideas from **Fast-Downward, domain engineering** to a small concrete instance. State assumptions explicitly, write the governing equations, and interpret each term. Compare your result with the AIMA chapter exercise that matches Chapter 11.8.

> **Readable form:** Translate the abstract definition into numbers or code for one toy case.


## Design Checklist

| Step | Action |
|------|--------|
| 1 | Identify variables and constraints for pddl and practical systems |
| 2 | Choose representation aligned with Fast-Downward, domain engineering |
| 3 | Verify against AIMA Chapter 11.8 definitions |
| 4 | Implement minimal prototype in Python |
| 5 | Test edge cases and document failure modes |


## Connections Across Course 2

This section on **PDDL and Practical Systems** links forward to later chapters that reuse the same abstractions. Revisit [../../GLOSSARY.md](../../GLOSSARY.md) entries while reading, and cross-check notation with [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md). Prior search and logic chapters provide algorithmic scaffolding; probability chapters supply uncertainty semantics when Fast-Downward, domain engineering involves partial information.


## Key Takeaways

1. **PDDL and Practical Systems** - core idea 1 from AIMA Chapter 11.8 (Fast-Downward, domain engineering).
2. **PDDL and Practical Systems** - core idea 2 from AIMA Chapter 11.8 (Fast-Downward, domain engineering).
3. **PDDL and Practical Systems** - core idea 3 from AIMA Chapter 11.8 (Fast-Downward, domain engineering).
4. **PDDL and Practical Systems** - core idea 4 from AIMA Chapter 11.8 (Fast-Downward, domain engineering).
5. **PDDL and Practical Systems** - core idea 5 from AIMA Chapter 11.8 (Fast-Downward, domain engineering).


## Common Mistakes

- **Confusing syntax with semantics** in PDDL and Practical Systems - symbols must map to models or distributions.
- **Skipping units and domains** when Fast-Downward, domain engineering involves continuous or structured variables.
- **Treating heuristics as guarantees** - always state when optimality or soundness holds.
- **Ignoring complexity** - tractability assumptions in Chapter 11.8 may fail at scale.
- **Omitting sensor/actuator noise** when deploying ideas from textbook toy domains.


## Deeper Reading

Re-read Russell & Norvig Chapter 11.8 with pen and paper. For **PDDL and Practical Systems**, derive each formula from first principles before using libraries. The aima-python repository implements many algorithms from this chapter - compare your pseudocode to the reference implementation and note API differences.

## Key Terms (Glossary)

- **Fast-Downward** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **domain** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **engineering** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **inference** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **representation** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **agent** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of PDDL and Practical Systems in AIMA?
2. How does PDDL and Practical Systems relate to Fast-Downward, domain engineering?
3. Give a concrete application of PDDL and Practical Systems.
4. What assumptions does the textbook emphasize for PDDL and Practical Systems?
5. When would an alternative to PDDL and Practical Systems be preferable?
6. How would you verify understanding of PDDL and Practical Systems in code?

---

**Previous:** [Section 11.7 - Multiagent and Contingent Planning](./section-07-multiagent-and-contingent-planning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Ghallab, M., Nau, D., & Traverso, P. (2004). *Automated Planning*. Morgan Kaufmann.']