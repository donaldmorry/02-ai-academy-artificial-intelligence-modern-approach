# Section 11.2: State-Space Planning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11.2
> **Previous:** [Section 11.1 - Planning Problem Definition](./section-01-planning-problem-definition.md)
> **Next:** [Section 11.3 - Planning Graphs](./section-03-planning-graphs.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## State-Space Planning

**Forward state-space search:** start at $s_0$; apply applicable actions; stop at goal.

Same framework as [Chapter 03](../chapter-03-solving-problems-by-searching/README.md) - BFS, A*, greedy - but states are fluent sets.

> **Readable form:** Explore the tree of world states reachable by actions until you hit the goal.

---

## Applicable Actions

Action $a$ **applicable** in $s$ iff $Pre(a) \subseteq s$.

Branching factor depends on domain - blocks world can be large.

---

## Heuristics for Planning

**Relaxed problem:** ignore delete lists → monotonic search toward goal.

**Goal count heuristic:** $h(s)$ = number of goal fluents not in $s$.

**FF heuristic:** plan on relaxed problem, use relaxed plan length.

**Pattern databases:** precompute costs for subgoals.

---

## Forward vs. Backward (Regression)

**Forward:** from initial state toward goal.

**Backward (regression):** from goal toward initial - find actions that could have produced goal fluents.

Regression can reduce branching when goals are small ([Section 11.3](./section-03-planning-graphs.md) uses forward structure differently).

---

## Example Search Trace

Goal: $On(A,B)$. Forward BFS may explore many stack configurations before solution.

**A\*** with admissible heuristic often finds optimal plan faster.

---

## Implementation

```python
def forward_plan(problem, frontier, explored):
    while frontier:
        state, plan = frontier.pop()
        if goal_satisfied(state, problem.goal):
            return plan
        for action in applicable_actions(state, problem.actions):
            new_state = apply_action(state, action)
            if new_state not in explored:
                explored.add(new_state)
                frontier.push((new_state, plan + [action]))
    return None
```

---

## Complexity

Planning is **PSPACE-complete** for classical STRIPS - harder than NP SAT in general.

Practical planners use heuristics and structure - not brute force.

---

## Blocks World Heuristic Example

Goal: $On(A,B), On(B,C)$

**Ignore delete lists:** assume blocks can be stacked freely → relaxed plan length 2.

**Admissible:** relaxed plan never shorter than real plan - safe for A*.

---

## Regression Example

Goal: $On(A,B)$

Regress $PutDown(A,B)$: preconditions $Holding(A)$, $Clear(B)$

Continue regressing until initial state subsumes preconditions.

Effective when goals have few fluents.

---

## State Space Size

$n$ blocks, table + $n$ stacks - state count grows quickly.

**Symmetry breaking:** treat block labels as distinct (A≠B) but prune isomorphic configurations in advanced planners.

---

## Study Exercise: Heuristic Admissibility

Prove goal-count heuristic admissible for blocks world (never overestimates).

Construct example where ignore-delete-lists heuristic is **not** admissible - when?

## Worked Example: State-Space Planning

Apply the ideas from **Forward search, heuristics for planning** to a small concrete instance. State assumptions explicitly, write the governing equations, and interpret each term. Compare your result with the AIMA chapter exercise that matches Chapter 11.2.

> **Readable form:** Translate the abstract definition into numbers or code for one toy case.


## Design Checklist

| Step | Action |
|------|--------|
| 1 | Identify variables and constraints for state-space planning |
| 2 | Choose representation aligned with Forward search, heuristics for planning |
| 3 | Verify against AIMA Chapter 11.2 definitions |
| 4 | Implement minimal prototype in Python |
| 5 | Test edge cases and document failure modes |


## Connections Across Course 2

This section on **State-Space Planning** links forward to later chapters that reuse the same abstractions. Revisit [../../GLOSSARY.md](../../GLOSSARY.md) entries while reading, and cross-check notation with [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md). Prior search and logic chapters provide algorithmic scaffolding; probability chapters supply uncertainty semantics when Forward search, heuristics for planning involves partial information.


## Key Takeaways

1. **State-Space Planning** - core idea 1 from AIMA Chapter 11.2 (Forward search, heuristics for planning).
2. **State-Space Planning** - core idea 2 from AIMA Chapter 11.2 (Forward search, heuristics for planning).
3. **State-Space Planning** - core idea 3 from AIMA Chapter 11.2 (Forward search, heuristics for planning).
4. **State-Space Planning** - core idea 4 from AIMA Chapter 11.2 (Forward search, heuristics for planning).
5. **State-Space Planning** - core idea 5 from AIMA Chapter 11.2 (Forward search, heuristics for planning).


## Common Mistakes

- **Confusing syntax with semantics** in State-Space Planning - symbols must map to models or distributions.
- **Skipping units and domains** when Forward search, heuristics for planning involves continuous or structured variables.
- **Treating heuristics as guarantees** - always state when optimality or soundness holds.
- **Ignoring complexity** - tractability assumptions in Chapter 11.2 may fail at scale.
- **Omitting sensor/actuator noise** when deploying ideas from textbook toy domains.


## Deeper Reading

Re-read Russell & Norvig Chapter 11.2 with pen and paper. For **State-Space Planning**, derive each formula from first principles before using libraries. The aima-python repository implements many algorithms from this chapter - compare your pseudocode to the reference implementation and note API differences.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 is cumulative - search, logic, probability, and learning share representational foundations.

---

## Practice Trace

Hand-execute the main algorithm from this section on a toy instance small enough to fit on one page. Record each intermediate structure (clause set, belief state, plan layer, or gradient step). Compare your trace to aima-python reference code if available.

> **Readable form:** If you cannot trace it by hand on paper, you do not yet understand it well enough to deploy it.

## Study Note

Review the AIMA Chapter 11.2 exercises and trace one algorithm by hand before the lab.

---

## Key Terms (Glossary)

- **Forward** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **search** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **heuristics** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **for** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **planning** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of State-Space Planning in AIMA?
2. How does State-Space Planning relate to Forward search, heuristics for planning?
3. Give a concrete application of State-Space Planning.
4. What assumptions does the textbook emphasize for State-Space Planning?
5. When would an alternative to State-Space Planning be preferable?
6. How would you verify understanding of State-Space Planning in code?

---

**Previous:** [Section 11.1 - Planning Problem Definition](./section-01-planning-problem-definition.md) · **Next:** [Section 11.3 - Planning Graphs](./section-03-planning-graphs.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Ghallab, M., Nau, D., & Traverso, P. (2004). *Automated Planning*. Morgan Kaufmann.']