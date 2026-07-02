# Section 11.3: Planning Graphs

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11.3
> **Previous:** [Section 11.2 - State-Space Planning](./section-02-state-space-planning.md)
> **Next:** [Section 11.4 - SAT Planning](./section-04-sat-planning.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Planning Graphs

**GRAPHPLAN** (Blum & Furst) builds a **layered bipartite graph** alternating **action layers** and **fluent layers**.

Level 0: initial fluents. Level $i$: actions applicable at $i$; level $i+1$: fluents after those actions.

> **Readable form:** Build a ladder of "what could be true" and "what actions could run" level by level until goals appear together.

---

## Graph Structure

- $P_i$ - fluents possible at step $i$
- $A_i$ - actions executable at step $i$ producing $P_{i+1}$

Expand until all goal fluents in some $P_k$ **without mutex** conflict.

---

## Mutex Relations

Two fluents **mutex** (mutually exclusive) if they cannot coexist:

1. **Inconsistent effects** - one adds, other deletes same fluent
2. **Interference** - action deleting precondition of another
3. **Competing needs** - mutex preconditions

Two actions mutex if their effects interfere.

**Goal mutex:** goals in same layer mutex → no plan at that length.

---

## Solution Extraction

Backward from goal layer: pick non-mutex actions achieving goals; recurse to predecessors.

Produces **parallel** plan slices - actions at same level can run concurrently (if mutex-free).

---

## Termination

If graph expands to level $k$ with goals mutex at all levels → **no plan** of length $\leq k$.

May need to expand further or prove unsolvable.

---

## Limitations

GRAPHPLAN handles **STRIPS** without conditional effects or complex ADL - modern planners generalize.

Still influential for **relaxed planning graph** heuristics in Fast-Downward.

---

## GRAPHPLAN Mutex Example

Goals $On(A,B)$ and $On(B,A)$ at same level - **inconsistent** if both require $Clear$ conflicts.

Mutex detection proves **no parallel plan** at that length.

---

## Relaxed Planning Graph

Ignore delete lists when building graph → **monotonic** fluent growth.

Extract relaxed plan → heuristic estimate for A* / greedy search.

**Fast-Downward** landmark heuristics extend this idea.

---

## Layered Plans

Plan slice at level $k$: $\{Move(A,B), Move(C,D)\}$ - parallel if mutex-free.

Real execution may serialize if resources conflict.

---

## Study Exercise: Mutex by Hand

Two actions at same level both require $HandEmpty$ - mutex? Draw planning graph level 0-2 for gripper domain.

## Worked Example: Planning Graphs

Apply the ideas from **GRAPHPLAN, mutex relations, solution extraction** to a small concrete instance. State assumptions explicitly, write the governing equations, and interpret each term. Compare your result with the AIMA chapter exercise that matches Chapter 11.3.

> **Readable form:** Translate the abstract definition into numbers or code for one toy case.


## Design Checklist

| Step | Action |
|------|--------|
| 1 | Identify variables and constraints for planning graphs |
| 2 | Choose representation aligned with GRAPHPLAN, mutex relations, solution extraction |
| 3 | Verify against AIMA Chapter 11.3 definitions |
| 4 | Implement minimal prototype in Python |
| 5 | Test edge cases and document failure modes |


## Connections Across Course 2

This section on **Planning Graphs** links forward to later chapters that reuse the same abstractions. Revisit [../../GLOSSARY.md](../../GLOSSARY.md) entries while reading, and cross-check notation with [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md). Prior search and logic chapters provide algorithmic scaffolding; probability chapters supply uncertainty semantics when GRAPHPLAN, mutex relations, solution extraction involves partial information.


## Key Takeaways

1. **Planning Graphs** - core idea 1 from AIMA Chapter 11.3 (GRAPHPLAN, mutex relations, solution extraction).
2. **Planning Graphs** - core idea 2 from AIMA Chapter 11.3 (GRAPHPLAN, mutex relations, solution extraction).
3. **Planning Graphs** - core idea 3 from AIMA Chapter 11.3 (GRAPHPLAN, mutex relations, solution extraction).
4. **Planning Graphs** - core idea 4 from AIMA Chapter 11.3 (GRAPHPLAN, mutex relations, solution extraction).
5. **Planning Graphs** - core idea 5 from AIMA Chapter 11.3 (GRAPHPLAN, mutex relations, solution extraction).


## Common Mistakes

- **Confusing syntax with semantics** in Planning Graphs - symbols must map to models or distributions.
- **Skipping units and domains** when GRAPHPLAN, mutex relations, solution extraction involves continuous or structured variables.
- **Treating heuristics as guarantees** - always state when optimality or soundness holds.
- **Ignoring complexity** - tractability assumptions in Chapter 11.3 may fail at scale.
- **Omitting sensor/actuator noise** when deploying ideas from textbook toy domains.


## Deeper Reading

Re-read Russell & Norvig Chapter 11.3 with pen and paper. For **Planning Graphs**, derive each formula from first principles before using libraries. The aima-python repository implements many algorithms from this chapter - compare your pseudocode to the reference implementation and note API differences.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 is cumulative - search, logic, probability, and learning share representational foundations.

---

## Practice Trace

Hand-execute the main algorithm from this section on a toy instance small enough to fit on one page. Record each intermediate structure (clause set, belief state, plan layer, or gradient step). Compare your trace to aima-python reference code if available.

> **Readable form:** If you cannot trace it by hand on paper, you do not yet understand it well enough to deploy it.


---

## Practice Trace

Hand-execute the main algorithm from this section on a toy instance small enough to fit on one page. Record each intermediate structure (clause set, belief state, plan layer, or gradient step). Compare your trace to aima-python reference code if available.

> **Readable form:** If you cannot trace it by hand on paper, you do not yet understand it well enough to deploy it.

## Study Note

Review the AIMA Chapter 11.3 exercises and trace one algorithm by hand before the lab.

---

## Key Terms (Glossary)

- **GRAPHPLAN** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **mutex** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **relations** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **solution** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **extraction** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Planning Graphs in AIMA?
2. How does Planning Graphs relate to GRAPHPLAN, mutex relations, solution extraction?
3. Give a concrete application of Planning Graphs.
4. What assumptions does the textbook emphasize for Planning Graphs?
5. When would an alternative to Planning Graphs be preferable?
6. How would you verify understanding of Planning Graphs in code?

---

**Previous:** [Section 11.2 - State-Space Planning](./section-02-state-space-planning.md) · **Next:** [Section 11.4 - SAT Planning](./section-04-sat-planning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Ghallab, M., Nau, D., & Traverso, P. (2004). *Automated Planning*. Morgan Kaufmann.']

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
