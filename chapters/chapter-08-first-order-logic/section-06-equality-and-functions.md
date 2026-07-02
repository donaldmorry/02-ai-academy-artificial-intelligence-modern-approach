# Section 8.6: Equality and Functions

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 8.2-8.3  
> **Previous:** [Section 8.5 - Knowledge Engineering](./section-05-knowledge-engineering.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Equality in FOL

**Equality** ($=$) is a logical symbol with special **axioms**:

### Reflexivity, symmetry, transitivity

$$
\forall x \, x = x, \quad \forall x,y \, x = y \Rightarrow y = x
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

$$
\forall x,y,z \, (x = y \land y = z) \Rightarrow x = z
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

### Substitutivity (Leibniz's law)

$$
\forall x,y \, (x = y) \Rightarrow \bigl(\forall P \, P(x) \Leftrightarrow P(y)\bigr)
$$
> **Readable form:** Equal terms denote the same object - you can replace one with the other anywhere in a formula.

**Example:** From $Mother(John) = Mary$ and $Parent(Mary, Sue)$, derive $Parent(Mother(John), Sue)$.

---

## Function Symbols as Shorthand

A **function** $f$ maps terms to terms. **Axioms** relate functions to predicates:

$$
\forall x \, Mother(x) = y \Leftrightarrow Female(y) \land Parent(y, x)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

Functions reduce sentence count but are **total** in classical FOL - every application denotes some object. Use type guards:

$$
Person(x) \Rightarrow \exists y \, Mother(x) = y
$$
> **Readable form:** The existential quantifier says at least one object must make the stated relationship true.

---

## Unique Names Assumption (UNA)

Logic alone does **not** entail $John \neq Mary$. **UNA axiom schema** for constants $a, b$: $a \neq b$.

**When UNA fails:** `MorningStar` and `EveningStar` both denote Venus - co-reference, not inequality.

---

## Domain Closure

**Domain closure axiom** for finite enumerated objects $o_1,\ldots,o_n$:

$$
\forall x \, P(x) \Rightarrow (x = o_1 \lor x = o_2 \lor \cdots \lor x = o_n)
$$
> **Readable form:** The only students are Alice, Bob, Carol - no unnamed extra students in any model.

**Closed-world assumption (CWA):** if a ground atom is not in the KB, assume false - nonmonotonic ([Section 10.5](../chapter-10-knowledge-representation/section-05-default-reasoning.md)).

---

## Exactly-One Constraints

"Each person has exactly one biological mother":

**At least one:** $\forall x \, \exists y \, Mother(x) = y$

**At most one:** $\forall x,y,z \, Mother(x)=y \land Mother(x)=z \Rightarrow y=z$

"Exactly one department chair for $d$":

$$
\exists h \, Chair(h,d) \land \forall h' \, Chair(h',d) \Rightarrow h' = h
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

---

## Arithmetic as Functions

Peano-style: constant $0$, $Successor(x)$, $Plus(x,y)$, $Times(x,y)$.

$$
\forall x \, Plus(0, x) = x
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

$$
\forall x,y \, Plus(Successor(x), y) = Successor(Plus(x,y))
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

Modern provers include **theories** (linear arithmetic) for efficiency - beyond pure FOL.

---

## Worked Example: Co-Reference

Facts: $CEO(Acme) = Alice$, $CEO(Acme) = CEO(AcmeCorp)$, $Acme = AcmeCorp$.

Query: $CEO(AcmeCorp) = Alice$? Chain via substitution.

Unification and equality are tightly coupled in theorem provers ([Section 9.2](../chapter-09-inference-first-order-logic/section-02-unification.md)).

---

## Functions vs. Skolem Constants

| Construct | Use |
|-----------|-----|
| **Function symbol** | Denotes deterministic value ($Mother$, $Head$) |
| **Skolem constant** | Witness for $\exists$ during CNF conversion - not user vocabulary |

$\exists x \, Owns(John,x)$ → $Owns(John,Sk1)$ in Skolem form.

---

## Implementation: Equality Demodulation

Theorem provers use **demodulation**: rewrite using equations $lhs = rhs$ when $lhs$ unifies with a subterm.

```python
def substitute(term, x, replacement):
    if term == x:
        return replacement
    if is_function(term):
        return apply(term.op, [substitute(a, x, replacement) for a in term.args])
    return term
```

---

## Common Errors

| Error | Issue |
|-------|-------|
| Using $=$ for similarity | "John is like Mary" ≠ equality |
| Non-functional relations as functions | Two mothers → function axiom inconsistent |
| Forgetting UNA | Derives contradictions from $John \neq John$ variants |
| Open vs. closed world | "Not in KB" treated as false without CWA |

---

## Equality and Unification Together

Theorem provers treat equality as **equational theory**:

1. **Unify** literals in resolution
2. **Demodulate** using $lhs = rhs$ rewrite rules
3. **Paramodulate** - generate equality substitutions during resolution

**Example chain:** from $Mother(John)=Mary$, $Parent(Mary,Sue)$, derive $Parent(Mother(John),Sue)$ via substitution into $Parent(y,Sue)$ with $y=Mary$, then replace $Mary$ with $Mother(John)$.

---

## Sorts and Many-Sorted Logic

**Many-sorted FOL** attaches **sorts** (types) to variables:

$$
\forall p:Person \, \exists m:Person \, Mother(p) = m
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

Prevents ill-formed $Mother(Desk)$ without extra axioms. OWL and description logics build sorts into the syntax ([Section 10.8](../chapter-10-knowledge-representation/section-08-knowledge-graphs.md)).

---

## Worked Exercise: Course Instructor

**Exactly one instructor per course:**

$$
\forall c \, Course(c) \Rightarrow \exists! p \, Teaches(p,c)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

Expanded:

$$
\forall c \, Course(c) \Rightarrow \bigl(\exists p \, Teaches(p,c) \land \forall p' \, Teaches(p',c) \Rightarrow p'=p'\bigr)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

**At most one** without existence: graduate seminars with team-teaching violate existence - ontology must allow optional instructors or team predicates $CoTeaches(p_1,p_2,c)$.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed. This four-step checklist catches most knowledge-engineering errors early.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 Part III is cumulative - inference (Chapter 09), representation (Chapter 10), and planning (Chapter 11) share the same logical foundations from Chapters 07-08.


---

## Peano Arithmetic in FOL

**Successor injectivity:**

$$
\forall x,y \, S(x) = S(y) \Rightarrow x = y
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

**Induction schema** (second-order in full form):

$$
(P(0) \land \forall n \, (P(n) \Rightarrow P(S(n)))) \Rightarrow \forall n \, P(n)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Single FOL sentence cannot capture induction over all predicates - SOL needed for categorical $\mathbb{N}$.

---

## Co-Reference and Equality Chains

Given:

$$
CEO(Acme) = Alice, \quad CEO(Acme) = CEO(AcmeCorp), \quad Acme = AcmeCorp
$$
> **Readable form:** The equality states that two terms denote the same object or value inside the model.

Prove $CEO(AcmeCorp) = Alice$ by transitivity of equality and substitutivity.

---

## Reflection Questions

1. From $a=b$ and $P(a)$, prove $P(b)$ using substitutivity.
2. Write exactly-one axioms for "each course has one instructor."
3. When should co-reference ($MorningStar = EveningStar$) be asserted vs. inferred?
4. Why are Skolem functions introduced during inference but not in user ontologies?
5. How does domain closure interact with $\exists$ queries ("is there another student?")?

---

**Next:** [Section 8.7 - Comparison of Logics](./section-07-comparison-of-logics.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Sections 8.2-8.3.
2. Leibniz's law - Stanford Encyclopedia. [plato.stanford.edu](https://plato.stanford.edu/entries/identity-indiscernible/)
3. Reiter, R. - closed-world assumption foundations.
4. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python) - equality in `unify`.
5. Description Logic Handbook - functional roles vs. equality.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Substitutivity** | Equal terms can replace each other in any context (Leibniz). |
| **Unique names assumption** | Distinct constants denote distinct objects (extra axiom). |
| **Domain closure** | Every object satisfying $P$ is one of a finite listed set. |
| **Closed-world assumption** | Unstated ground atoms assumed false (nonmonotonic). |
| **Open world** | Classical FOL: lack of proof does not imply falsehood. |
| **Skolem function** | Function introduced to eliminate $\exists$ in clausal form. |
| **Demodulation** | Oriented equality rewriting used in theorem proving. |



