# Section 11.6: Scheduling and Resources

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11.6
> **Previous:** [Section 11.5 - Hierarchical Planning](./section-05-hierarchical-planning.md)
> **Next:** [Section 11.7 - Multiagent and Contingent Planning](./section-07-multiagent-and-contingent-planning.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Scheduling and Resources

**Planning** orders actions; **scheduling** assigns **time** and **resources** respecting constraints.

**Classical planning** often assumes **unit duration** and unlimited resources - unrealistic for factories, spacecraft, project management.

> **Readable form:** Who does what when, without double-booking machines or people.

---

## Resources

**Resource** $r$ with capacity $C_r$. Action $a$ may **consume** or **require** resources:

$$
Requires(a, r, k)
$$
> **Readable form:** This formal sentence is the symbolic version of the relationship stated in the surrounding explanation.

**Conflict:** two actions needing full machine capacity cannot overlap.

---

## Temporal Constraints

**Start/end times** $Start(a)$, $End(a)$.

**Precedence:** $End(a) \leq Start(b)$ if $a$ before $b$.

**Duration:** $End(a) = Start(a) + Duration(a)$.

---

## Constraint-Based Scheduling

Encode as **constraint satisfaction** ([Chapter 06](../chapter-06-constraint-satisfaction/README.md)) or **linear programming**.

**Disjunctive constraints:** job on machine A OR machine B.

---

## Integration with Planning

1. **Plan** action sequence (STRIPS/HTN)
2. **Schedule** times and resources
3. **Execute** with monitoring

**Temporal planning** merges both - actions have duration, concurrent execution.

---

## Real-World Examples

- NASA mission planning (EUROPA scheduler)
- Factory floor job-shop scheduling
- Cloud workload orchestration

---

## Metrics

| Metric | Meaning |
|--------|---------|
| **Makespan** | Total time to complete |
| **Tardiness** | Lateness vs. deadline |
| **Resource utilization** | Fraction of capacity used |

---

## Job-Shop Scheduling

$n$ jobs, $m$ machines - each job sequence of operations on specific machines.

**Disjunctive graph:** nodes = operations; edges = precedence + machine exclusivity.

**Makespan minimization** NP-hard - constraint solvers use branch-and-bound.

---

## Resource Types in PDDL

PDDL 2.1 **durative actions:**

```
:duration (= ?duration 5)
:condition (at start (available ?r))
:effect (at end (not (in-use ?r)))
```

Continuous time - bridge planning and scheduling.

---

## Integration Pipeline

```
HTN decomposition → STRIPS primitive sequence → scheduler assigns times
```

NASA workflows combine planning levels for spacecraft operations.

---

## Study Exercise: Makespan

Three jobs on two machines - draw disjunctive graph. Estimate minimum makespan lower bound.

## Worked Example: Scheduling and Resources

Apply the ideas from **Time, resources, constraint-based scheduling** to a small concrete instance. State assumptions explicitly, write the governing equations, and interpret each term. Compare your result with the AIMA chapter exercise that matches Chapter 11.6.

> **Readable form:** Translate the abstract definition into numbers or code for one toy case.


## Design Checklist

| Step | Action |
|------|--------|
| 1 | Identify variables and constraints for scheduling and resources |
| 2 | Choose representation aligned with Time, resources, constraint-based scheduling |
| 3 | Verify against AIMA Chapter 11.6 definitions |
| 4 | Implement minimal prototype in Python |
| 5 | Test edge cases and document failure modes |


## Connections Across Course 2

This section on **Scheduling and Resources** links forward to later chapters that reuse the same abstractions. Revisit [../../GLOSSARY.md](../../GLOSSARY.md) entries while reading, and cross-check notation with [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md). Prior search and logic chapters provide algorithmic scaffolding; probability chapters supply uncertainty semantics when Time, resources, constraint-based scheduling involves partial information.


## Key Takeaways

1. **Scheduling and Resources** - core idea 1 from AIMA Chapter 11.6 (Time, resources, constraint-based scheduling).
2. **Scheduling and Resources** - core idea 2 from AIMA Chapter 11.6 (Time, resources, constraint-based scheduling).
3. **Scheduling and Resources** - core idea 3 from AIMA Chapter 11.6 (Time, resources, constraint-based scheduling).
4. **Scheduling and Resources** - core idea 4 from AIMA Chapter 11.6 (Time, resources, constraint-based scheduling).
5. **Scheduling and Resources** - core idea 5 from AIMA Chapter 11.6 (Time, resources, constraint-based scheduling).


## Common Mistakes

- **Confusing syntax with semantics** in Scheduling and Resources - symbols must map to models or distributions.
- **Skipping units and domains** when Time, resources, constraint-based scheduling involves continuous or structured variables.
- **Treating heuristics as guarantees** - always state when optimality or soundness holds.
- **Ignoring complexity** - tractability assumptions in Chapter 11.6 may fail at scale.
- **Omitting sensor/actuator noise** when deploying ideas from textbook toy domains.


## Deeper Reading

Re-read Russell & Norvig Chapter 11.6 with pen and paper. For **Scheduling and Resources**, derive each formula from first principles before using libraries. The aima-python repository implements many algorithms from this chapter - compare your pseudocode to the reference implementation and note API differences.


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

Review the AIMA Chapter 11.6 exercises and trace one algorithm by hand before the lab.

---

## Key Terms (Glossary)

- **Time** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **resources** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **constraint-based** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **scheduling** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Scheduling and Resources in AIMA?
2. How does Scheduling and Resources relate to Time, resources, constraint-based scheduling?
3. Give a concrete application of Scheduling and Resources.
4. What assumptions does the textbook emphasize for Scheduling and Resources?
5. When would an alternative to Scheduling and Resources be preferable?
6. How would you verify understanding of Scheduling and Resources in code?

## Scheduling Review Check

Before moving on, write one tiny scheduling instance with two actions, one shared resource, and one deadline. Mark whether the conflict is temporal, resource-based, or both, then state which constraint would make the schedule infeasible.

---

**Previous:** [Section 11.5 - Hierarchical Planning](./section-05-hierarchical-planning.md) · **Next:** [Section 11.7 - Multiagent and Contingent Planning](./section-07-multiagent-and-contingent-planning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Ghallab, M., Nau, D., & Traverso, P. (2004). *Automated Planning*. Morgan Kaufmann.']


