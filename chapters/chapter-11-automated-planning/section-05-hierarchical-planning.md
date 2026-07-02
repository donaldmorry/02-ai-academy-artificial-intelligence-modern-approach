# Section 11.5: Hierarchical Planning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11.5
> **Previous:** [Section 11.4 - SAT Planning](./section-04-sat-planning.md)
> **Next:** [Section 11.6 - Scheduling and Resources](./section-06-scheduling-and-resources.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Hierarchical Task Network (HTN) Planning

**Classical planning:** find action sequence from STRIPS.

**HTN planning:** decompose **tasks** into subtasks using **methods** until primitive actions remain.

> **Readable form:** Instead of searching raw actions, follow a recipe book that breaks "make dinner" into "shop, cook, serve."

---

## HTN Components

| Component | Role |
|-----------|------|
| **Task** | Abstract goal (compound or primitive) |
| **Method** | Recipe: preconditions + subtask list |
| **Primitive task** | Maps to STRIPS action |
| **Initial task network** | Partially ordered tasks to achieve |

---

## Example: Travel Planning

Task: `GetToConference`

Method 1: `BookFlight` → `GoToAirport` → `Fly` → `TaxiToHotel`

Method 2: `Drive` → `Park` → `CheckIn`

Planner chooses method satisfying preconditions.

---

## vs. STRIPS

| STRIPS | HTN |
|--------|-----|
| Flat action search | Hierarchical decomposition |
| Domain-independent heuristics | Domain-specific methods |
| Good for toy domains | Good for real workflows |

Industrial robotics and logistics often use HTN (SHOP2, Pyhop).

---

## Partial Order

Subtasks may be **unordered** within method - planner inserts ordering constraints.

Reduces commitment early - more flexible than total-order plans.

---

## Python: Pyhop Sketch

```python
def travel_by_flight(state, task):
    return [('book_flight',), ('go_airport',), ('fly',), ('taxi',)]

pyhop.declare_methods('get_to_conference', travel_by_flight, travel_by_car)
```

---

## SHOP2 and Task Decomposition

**SHOP2** (Simple Hierarchical Ordered Planner) - total-order HTN.

Methods have **preconditions** on world state; subtasks ordered.

Used in logistics, military planning domains.

---

## Method Preconditions

```
Method: BuildHouse
  Pre: HasPermit
  Subtasks: Foundation → Frame → Roof → Interior
```

If $HasPermit$ false, method unavailable - try alternate method or fail.

---

## Primitive Task Mapping

Primitive `LayFoundation` maps to STRIPS action with same name - HTN leaf.

Decomposition **must** end in executable primitives - otherwise abstract plan only.

---

## Study Exercise: HTN Decomposition

Decompose "PrepareCourse" into subtasks: syllabus, assignments, lectures. Write two alternative methods (in-person vs. online).

## Worked Example: Hierarchical Planning

Apply the ideas from **HTNs, methods, decomposition** to a small concrete instance. State assumptions explicitly, write the governing equations, and interpret each term. Compare your result with the AIMA chapter exercise that matches Chapter 11.5.

> **Readable form:** Translate the abstract definition into numbers or code for one toy case.


## Design Checklist

| Step | Action |
|------|--------|
| 1 | Identify variables and constraints for hierarchical planning |
| 2 | Choose representation aligned with HTNs, methods, decomposition |
| 3 | Verify against AIMA Chapter 11.5 definitions |
| 4 | Implement minimal prototype in Python |
| 5 | Test edge cases and document failure modes |


## Connections Across Course 2

This section on **Hierarchical Planning** links forward to later chapters that reuse the same abstractions. Revisit [../../GLOSSARY.md](../../GLOSSARY.md) entries while reading, and cross-check notation with [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md). Prior search and logic chapters provide algorithmic scaffolding; probability chapters supply uncertainty semantics when HTNs, methods, decomposition involves partial information.


## Key Takeaways

1. **Hierarchical Planning** - core idea 1 from AIMA Chapter 11.5 (HTNs, methods, decomposition).
2. **Hierarchical Planning** - core idea 2 from AIMA Chapter 11.5 (HTNs, methods, decomposition).
3. **Hierarchical Planning** - core idea 3 from AIMA Chapter 11.5 (HTNs, methods, decomposition).
4. **Hierarchical Planning** - core idea 4 from AIMA Chapter 11.5 (HTNs, methods, decomposition).
5. **Hierarchical Planning** - core idea 5 from AIMA Chapter 11.5 (HTNs, methods, decomposition).


## Common Mistakes

- **Confusing syntax with semantics** in Hierarchical Planning - symbols must map to models or distributions.
- **Skipping units and domains** when HTNs, methods, decomposition involves continuous or structured variables.
- **Treating heuristics as guarantees** - always state when optimality or soundness holds.
- **Ignoring complexity** - tractability assumptions in Chapter 11.5 may fail at scale.
- **Omitting sensor/actuator noise** when deploying ideas from textbook toy domains.


## Deeper Reading

Re-read Russell & Norvig Chapter 11.5 with pen and paper. For **Hierarchical Planning**, derive each formula from first principles before using libraries. The aima-python repository implements many algorithms from this chapter - compare your pseudocode to the reference implementation and note API differences.


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

## Key Terms (Glossary)

- **HTNs** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **methods** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **decomposition** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **inference** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **representation** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **agent** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Hierarchical Planning in AIMA?
2. How does Hierarchical Planning relate to HTNs, methods, decomposition?
3. Give a concrete application of Hierarchical Planning.
4. What assumptions does the textbook emphasize for Hierarchical Planning?
5. When would an alternative to Hierarchical Planning be preferable?
6. How would you verify understanding of Hierarchical Planning in code?

---

**Previous:** [Section 11.4 - SAT Planning](./section-04-sat-planning.md) · **Next:** [Section 11.6 - Scheduling and Resources](./section-06-scheduling-and-resources.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Ghallab, M., Nau, D., & Traverso, P. (2004). *Automated Planning*. Morgan Kaufmann.']

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
