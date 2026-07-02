# Section 11.4: SAT Planning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11.4
> **Previous:** [Section 11.3 - Planning Graphs](./section-03-planning-graphs.md)
> **Next:** [Section 11.5 - Hierarchical Planning](./section-05-hierarchical-planning.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Planning as SAT

Encode planning as **Boolean satisfiability**: find assignment making CNF true.

**Bounded planning:** plan length $\leq T$; try $T=1,2,3,\ldots$ until SAT.

> **Readable form:** Assign true/false to "action $a$ happens at time $t$" and "fluent $f$ holds at $t$" so physics and goals are satisfied.

---

## Encoding Sketch

Variables:

- $F_{f,t}$ - fluent $f$ true at time $t$
- $A_{a,t}$ - action $a$ executes at $t$

**Initial state:** $F_{f,0}$ for $f \in s_0$; $\lnot F_{f,0}$ otherwise.

**Preconditions:** $A_{a,t} \Rightarrow F_{p,t}$ for each precondition $p$ of $a$.

**Frame axioms:** $F_{f,t+1} \Leftrightarrow (F_{f,t} \land \lnot Deletes(a,f)) \lor Adds(a,f)$ - simplified.

**Goal:** $F_{g,T}$ for goal fluents $g$.

**Mutex:** at most one action per time step (optional simplification).

---

## Advantages

- Leverage **fast SAT solvers** (MiniSat, Glucose)
- Natural **parallel action** constraints
- Extensions to **soft goals**, **preferences**

---

## Disadvantages

- CNF size grows with $T$ and fluents
- Frame axioms expensive - **SAS+** encodings more compact
- May need many $T$ iterations

---

## SATPlan Algorithm

```
for T = 0 to MAX:
    encode CNF for horizon T
    if SAT_solver(CNF) returns model:
        extract action sequence from model
        return plan
return failure
```

Used in early AI planners; modern variants in competition tracks.

---

## CNF Variable Count

$T$ steps, $F$ fluents, $A$ actions:

Roughly $O(T \cdot (F + A))$ variables - frame axioms add $O(T \cdot F \cdot A)$ clauses.

**SAS+** avoids explicit frame axioms - smaller CNF.

---

## Stepwise SATPlan

```
T=0: impossible unless goal in s0
T=1: try one action
T=2: try two actions
...
```

First satisfiable $T$ gives shortest plan (if costs uniform).

---

## Parallel Plans in SAT

Mutex constraints at each time step - at most one action affecting same fluent.

Or allow parallel with exclusion clauses between conflicting actions.

---

## Study Exercise: SAT Encoding Size

Count CNF variables for 2-step plan, 3 fluents, 2 actions - frame axioms dominate. Motivates SAS+ in Fast-Downward.

## Worked Example: SAT Planning

Apply the ideas from **CNF encoding, bounded planning horizon** to a small concrete instance. State assumptions explicitly, write the governing equations, and interpret each term. Compare your result with the AIMA chapter exercise that matches Chapter 11.4.

> **Readable form:** Translate the abstract definition into numbers or code for one toy case.


## Design Checklist

| Step | Action |
|------|--------|
| 1 | Identify variables and constraints for sat planning |
| 2 | Choose representation aligned with CNF encoding, bounded planning horizon |
| 3 | Verify against AIMA Chapter 11.4 definitions |
| 4 | Implement minimal prototype in Python |
| 5 | Test edge cases and document failure modes |


## Connections Across Course 2

This section on **SAT Planning** links forward to later chapters that reuse the same abstractions. Revisit [../../GLOSSARY.md](../../GLOSSARY.md) entries while reading, and cross-check notation with [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md). Prior search and logic chapters provide algorithmic scaffolding; probability chapters supply uncertainty semantics when CNF encoding, bounded planning horizon involves partial information.


## Key Takeaways

1. **SAT Planning** - core idea 1 from AIMA Chapter 11.4 (CNF encoding, bounded planning horizon).
2. **SAT Planning** - core idea 2 from AIMA Chapter 11.4 (CNF encoding, bounded planning horizon).
3. **SAT Planning** - core idea 3 from AIMA Chapter 11.4 (CNF encoding, bounded planning horizon).
4. **SAT Planning** - core idea 4 from AIMA Chapter 11.4 (CNF encoding, bounded planning horizon).
5. **SAT Planning** - core idea 5 from AIMA Chapter 11.4 (CNF encoding, bounded planning horizon).


## Common Mistakes

- **Confusing syntax with semantics** in SAT Planning - symbols must map to models or distributions.
- **Skipping units and domains** when CNF encoding, bounded planning horizon involves continuous or structured variables.
- **Treating heuristics as guarantees** - always state when optimality or soundness holds.
- **Ignoring complexity** - tractability assumptions in Chapter 11.4 may fail at scale.
- **Omitting sensor/actuator noise** when deploying ideas from textbook toy domains.


## Deeper Reading

Re-read Russell & Norvig Chapter 11.4 with pen and paper. For **SAT Planning**, derive each formula from first principles before using libraries. The aima-python repository implements many algorithms from this chapter - compare your pseudocode to the reference implementation and note API differences.


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

Review the AIMA Chapter 11.4 exercises and trace one algorithm by hand before the lab.

---

## Key Terms (Glossary)

- **CNF** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **encoding** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **bounded** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **planning** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **horizon** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of SAT Planning in AIMA?
2. How does SAT Planning relate to CNF encoding, bounded planning horizon?
3. Give a concrete application of SAT Planning.
4. What assumptions does the textbook emphasize for SAT Planning?
5. When would an alternative to SAT Planning be preferable?
6. How would you verify understanding of SAT Planning in code?

---

**Previous:** [Section 11.3 - Planning Graphs](./section-03-planning-graphs.md) · **Next:** [Section 11.5 - Hierarchical Planning](./section-05-hierarchical-planning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Ghallab, M., Nau, D., & Traverso, P. (2004). *Automated Planning*. Morgan Kaufmann.']

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
