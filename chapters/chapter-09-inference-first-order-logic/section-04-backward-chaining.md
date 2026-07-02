# Section 9.4: Backward Chaining

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 9.3  
> **Previous:** [Section 9.3 - Forward Chaining in FOL](./section-03-forward-chaining-in-fol.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Goal-Directed Inference

**Forward chaining** is **data-driven** - derives all consequences. **Backward chaining** is **goal-driven** - starts from query and works backward to known facts.

Ideal when user asks specific questions: "Is John a grandparent of Sue?" not "derive everything."

> **Readable form:** Work backward from what you want to prove until you hit facts you already know.

---

## Backward Chaining Algorithm

```
bc(KB, goal):
  if goal is known fact: return success
  for each rule where head unifies with goal:
    if bc(KB, each premise): return success
  return failure
```

**Proof tree:** goal at root; children are premises; leaves are facts.

---

## Example: Family Relations

Query: $Grandparent(John, Sue)$

Rule: $Parent(x,y) \land Parent(y,z) \Rightarrow Grandparent(x,z)$

Subgoals: $Parent(John,y)$, $Parent(y,Sue)$

Unify $y=Mary$ from facts → success.

---

## Backtracking and Search

Backward chaining is **depth-first search** over proof trees.

```python
def bc_ask(kb, goal, theta=None):
    if theta is None:
        theta = {}
    for fact in kb.facts:
        phi = unify(goal, fact, theta)
        if phi is not None:
            yield phi
    for rule in kb.rules:
        phi = unify(goal, rule.head, theta)
        if phi is None:
            continue
        if all(bc_ask(kb, prem, phi) for prem in rule.premises):
            yield phi
```

---

## Loops and Infinite Paths

**Danger:** recursive rules without base cases cause infinite loops.

$Ancestor(x,y) \leftarrow Parent(x,y)$
$Ancestor(x,y) \leftarrow Parent(x,z) \land Ancestor(z,y)$

Query $Ancestor(John,Sue)$ with cyclic unification paths - need **cycle detection** or **tabling** (memoization).

| Fix | Mechanism |
|-----|-----------|
| **Cycle check** | Track current goal stack |
| **Tabling** | Cache proved goals (XSB Prolog) |
| **Depth limit** | Practical but incomplete |

---

## Forward vs. Backward Chaining

| Criterion | Forward | Backward |
|-----------|---------|----------|
| Trigger | New facts | Queries |
| Work done | All consequences | Relevant to goal |
| Best for | Reactive agents, monitoring | Diagnostic queries |
| Risk | Irrelevant inference | Infinite loops |

**Hybrid:** forward cache common derivations; backward for interactive queries.

---

## Prolog Connection

Prolog is **backward chaining** over Horn clauses with **depth-first** search and user-defined rule order.

```prolog
grandparent(X,Z) :- parent(X,Y), parent(Y,Z).
?- grandparent(john, sue).
```

Rule order affects completeness and efficiency - first matching clause wins.

---

## Worked Trace: Backward Chaining Proof

Query: $Ancestor(John,Sue)$

Rules:
- $Parent(x,y) \Rightarrow Ancestor(x,y)$
- $Parent(x,z) \land Ancestor(z,y) \Rightarrow Ancestor(x,y)$

**Attempt rule 2:** subgoals $Parent(John,z)$, $Ancestor(z,Sue)$

$Parent(John,Mary)$ unifies $z{=}Mary$ → subgoal $Ancestor(Mary,Sue)$

**Rule 1 on $Ancestor(Mary,Sue)$:** subgoal $Parent(Mary,Sue)$ - fact ✓

**Success** with $\theta = \{z/Mary\}$.

---

## Answer Extraction

For query $\exists x \, Parent(x,Sue)$, backward chaining returns **bindings**:

$$
\{x/Mary\}, \{x/John\}, \ldots
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

Each binding is a **answer substitution**. Prolog displays all via backtracking.

---

## Negation in Backward Chaining

**Negation as failure:** to prove $\lnot G$, attempt $G$; if all branches fail, $\lnot G$ succeeds.

**Unsafe** with open-world semantics - document CWA assumptions clearly.

---

## Comparison with Resolution

| | Backward chaining | Resolution |
|---|-------------------|------------|
| Direction | Goal-driven | Refutation |
| Clauses | Horn definite | General CNF |
| Completeness | Horn fragment | FOL (semi-decidable) |
| Typical use | Expert systems, Prolog | Batch theorem proving |

Many Prolog programs are **incomplete** for FOL but **complete** for Horn subset.

---

## Study Exercise: Prolog Ordering

```prolog
ancestor(X,Y) :- parent(X,Y).
ancestor(X,Y) :- parent(X,Z), ancestor(Z,Y).
```

Swap clause order - does termination change for cyclic family trees?

Implement cycle detection via `\+ member(Y, [X|_])` in path - incomplete but practical.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed. This four-step checklist catches most knowledge-engineering errors early.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 Part III is cumulative - inference (Chapter 09), representation (Chapter 10), and planning (Chapter 11) share the same logical foundations from Chapters 07-08.


---

## Reflection Questions

1. Draw the proof tree for Grandparent(John,Sue).
2. How can recursive ancestor rules cause loops?
3. What is tabling and how does it help?
4. When prefer backward over forward chaining?
5. How does Prolog rule order affect search?

---

**Next:** [Section 9.5 - Resolution in FOL](./section-05-resolution-in-fol.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Section 9.3.
2. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python) - `fol_bc_ask`.
3. Clocksin, W., & Mellish, C. - *Programming in Prolog*.
4. Warren, D. H. D. - efficient Prolog implementation.
5. XSB Prolog - tabling.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Backward chaining** | Goal-driven inference from query to facts. |
| **Proof tree** | Tree of subgoals showing derivation. |
| **Backtracking** | Undo failed unifications and try alternatives. |
| **Tabling** | Memoization of proved goals to avoid recomputation. |
| **Depth-first search** | Default Prolog search strategy. |
| **Cycle detection** | Preventing infinite proof loops. |



