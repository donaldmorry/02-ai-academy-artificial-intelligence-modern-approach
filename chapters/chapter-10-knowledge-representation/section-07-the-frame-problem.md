# Section 10.7: The Frame Problem

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 10.7  
> **Previous:** [Section 10.6 - Truth Maintenance](./section-06-truth-maintenance.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## The Frame Problem

When an action occurs, **most things don't change**. Representing persistence explicitly is overwhelming.

**Frame problem (McCarthy & Hayes):** difficulty of efficiently representing **what stays the same** after actions without enumerating all non-effects.

> **Readable form:** After moving the robot, the wall color doesn't change - we shouldn't need to say that thousands of times.

---

## Qualification and Ramification

Related challenges:

- **Qualification:** unexpected preconditions (can't drive through wall)
- **Ramification:** indirect effects (wet floor → slippery)

---

## STRIPS Approach

**Add lists** and **delete lists** on actions - only listed fluents change.

```
Action: Move(robot, from, to)
  Precond: At(robot, from)
  Add: At(robot, to)
  Delete: At(robot, from)
```

**Frame assumption:** unlisted fluents persist (implicit).

Used in [Chapter 11](../chapter-11-automated-planning/README.md) classical planning.

---

## Successor-State Axioms

For each fluent $f$:

$$
Holds(f, do(a,s)) \Leftrightarrow EffectAdds(a,f) \lor (Holds(f,s) \land \lnot EffectDeletes(a,f))
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

One axiom per fluent - compact frame solution in situation calculus.

---

## Event Calculus Frame Axioms

Persistence via $Initiates$/$Terminates$ - fluents hold until ended.

Combines with common-sense law of inertia.

---

## Comparison

| Approach | Frame handling |
|----------|----------------|
| Enumerate non-effects | Exponential |
| STRIPS add/delete | Implicit persistence |
| Successor-state | One axiom per fluent |
| Probabilistic models | Learn what persists |

---

## Modern Perspectives

**Learning dynamics** from data (model-based RL) sidesteps hand-coded frame axioms.

Symbolic planning still relies on STRIPS/PDDL conventions for efficiency.

---

## Frame Axiom Proliferation (Historical)

Early STRIPS planners listed **frame axioms** explicitly - one per fluent per action:

"Moving doesn't change color of blocks."

$n$ fluents × $m$ actions → $O(nm)$ axioms - unmaintainable.

**Successor-state** and **STRIPS delete lists** solve compactly.

---

## Fluent Persistence Formalized

$$
Holds(f, do(a,s)) \Leftrightarrow \gamma_f^+(a,s) \lor (Holds(f,s) \land \lnot \gamma_f^-(a,s))
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

$\gamma^+$ = action adds $f$; $\gamma^-$ = action deletes $f$.

---

## Probabilistic Frame Handling

**DBN** (dynamic Bayesian networks): $P(f_{t+1} \mid f_t, a_t)$ - learn persistence probabilities from data instead of axioms.

---

## Study Exercise: Frame Counting

Domain: 10 fluents, 5 actions. How many explicit frame axioms if listing all non-effects?

Compare to one successor-state axiom per fluent (10 axioms) - orders of magnitude savings.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed. This four-step checklist catches most knowledge-engineering errors early.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 Part III is cumulative - inference (Chapter 09), representation (Chapter 10), and planning (Chapter 11) share the same logical foundations from Chapters 07-08.


---

## Blocks World Frame Count

**Fluents:** $On$, $OnTable$, $Clear$, $Holding$, $HandEmpty$ - ~50 ground atoms for 5 blocks.

**Actions:** ~20 ground moves.

**Naive frame axioms:** $50 \times 20 = 1000$ persistence sentences.

**STRIPS:** 0 explicit frame axioms - implicit in add/delete lists.

**Successor-state:** 5 unified axioms (one per fluent schema).

---

## Event Calculus Persistence

**HoldsAt** inference:

$$
HoldsAt(f, t_2) \leftarrow HoldsAt(f, t_1) \land t_1 < t_2 \land \lnot Clipped(f, t_1, t_2)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

$Clipped$ - event terminated $f$ between $t_1$ and $t_2$.

**Planning connection:** STRIPS is **compiled** event calculus for restricted action classes.

---

## Reflection Questions

1. State frame problem in one sentence.
2. How STRIPS handles inertia.
3. Write successor-state axiom sketch.
4. Distinguish qualification and ramification.
5. How do learned world models relate to frame axioms?

---

**Next:** [Section 10.8 - Knowledge Graphs](./section-08-knowledge-graphs.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Section 10.7.
2. McCarthy & Hayes - original frame problem.
3. Fikes & Nilsson - STRIPS.
4. Shanahan - frame problem surveys.
5. Planning literature on add/delete lists.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Frame problem** | Challenge of representing unchanged fluents efficiently. |
| **Qualification problem** | Unstated preconditions for actions. |
| **Ramification problem** | Indirect side effects of actions. |
| **Add list** | STRIPS fluents made true by action. |
| **Delete list** | STRIPS fluents made false by action. |
| **Successor-state axiom** | Compact post-action fluent definition. |




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
