# Section 8.3: FOL Semantics

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 8.2  
> **Previous:** [Section 8.2 - FOL Syntax](./section-02-fol-syntax.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Models and Interpretations

Syntax tells us which strings are **well-formed**. **Semantics** tells us which sentences are **true** in which **worlds**.

A **model** $M$ for a first-order language consists of:

1. **Domain** $\mathcal{D}$ - nonempty set of objects.
2. **Interpretation** $I$ mapping:
   - each constant symbol to an object in $\mathcal{D}$
   - each $n$-ary predicate symbol to a relation $R \subseteq \mathcal{D}^n$
   - each $n$-ary function symbol to a function $f: \mathcal{D}^n \to \mathcal{D}$

Write $M = \langle \mathcal{D}, I \rangle$.

> **Readable form:** A model is a concrete world: a set of things plus meanings for every name, relation, and function in the vocabulary.

**Example model** for kinship:

| Symbol | Interpretation |
|--------|----------------|
| $\mathcal{D}$ | $\{john, mary, sue, jane\}$ |
| $I(John)$ | $john$ |
| $I(Parent)$ | $\{(john,mary), (mary,sue)\}$ |
| $I(Mother)$ | function mapping $mary \mapsto sue$, others default |

---

## Variable Assignments and Satisfaction

To evaluate formulas with **free variables**, we need a **variable assignment** $\sigma$ mapping each variable to an object in $\mathcal{D}$.

Write $(M, \sigma) \models \alpha$ to mean **$\alpha$ is satisfied** in model $M$ under assignment $\sigma$.

### Atomic sentences

$$
(M,\sigma) \models P(t_1,\ldots,t_n) \iff (I(t_1^\sigma), \ldots, I(t_n^\sigma)) \in I(P)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

where $t^\sigma$ is the object denoted by term $t$ under $\sigma$.

$$
(M,\sigma) \models t_1 = t_2 \iff I(t_1^\sigma) = I(t_2^\sigma)
$$
> **Readable form:** The equality states that two terms denote the same object or value inside the model.

### Connectives

Standard truth conditions (as in propositional logic):

$$
(M,\sigma) \models \lnot \alpha \iff (M,\sigma) \not\models \alpha
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

$$
(M,\sigma) \models \alpha \land \beta \iff (M,\sigma) \models \alpha \text{ and } (M,\sigma) \models \beta
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

(similarly for $\lor$, $\Rightarrow$, $\Leftrightarrow$)

### Quantifiers

$$
(M,\sigma) \models \forall x \, \alpha \iff \text{for every } d \in \mathcal{D}, \; (M, \sigma[x \mapsto d]) \models \alpha
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

(M,\sigma) \models \exists x \, \alpha \iff \text{there exists } d \in \mathcal{D}, \; (M, \sigma[x \mapsto d]) \models \alpha
$$

where $\sigma[x \mapsto d]$ overrides $x$'s assignment to $d$.

$$
> **Readable form:** $\forall x \, \alpha$ is true if $\alpha$ holds no matter which domain object we assign to $x$; $\exists x \, \alpha$ if some object makes $\alpha$ true.

For **closed sentences**, truth does not depend on $\sigma$ - only on $M$.

---

## Entailment, Validity, Satisfiability

| Concept | Definition |
|---------|------------|
| **Model of sentence $\alpha$** | $M$ such that $M \models \alpha$ |
| **Satisfiable** | Has at least one model |
| **Valid** | True in **every** model ($\models \alpha$) |
| **Entailment** | $\alpha \models \beta$ iff every model of $\alpha$ is a model of $\beta$ |

**Logical equivalence:** $\alpha \equiv \beta$ iff $\alpha \models \beta$ and $\beta \models \alpha$.

Quantifier equivalences (De Morgan):

$$
\lnot \forall x \, P(x) \equiv \exists x \, \lnot P(x)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

$$
\lnot \exists x \, P(x) \equiv \forall x \, \lnot P(x)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

---

## Worked Example: Evaluating a Sentence

Sentence: $\forall x \, \exists y \, Parent(x,y)$

Model: domain = $\{a,b\}$, $Parent = \{(a,b), (b,a)\}$.

- For $x=a$: choose $y=b$ → $(a,b) \in Parent$ ✓
- For $x=b$: choose $y=a$ → $(b,a) \in Parent$ ✓

Sentence is **true** in this model ("everyone has at least one child").

Change $Parent = \{(a,b)\}$ only. For $x=b$, no $y$ works → sentence **false**.

---

## Domain Assumptions

| Assumption | Effect |
|------------|--------|
| **Nonempty domain** | $\exists x \, x=x$ always valid |
| **Infinite domain** | Needed for arithmetic axioms |
| **Unique names** | Distinct constants denote distinct objects (not logic-valid - extra axiom) |

**Unique names assumption (UNA):** $John \neq Mary$ unless stated. Real ontologies often omit UNA; reasoning must allow co-reference.

---

## Standard Models: Numbers

**Peano axioms** (sketch) define $\mathcal{D} = \mathbb{N}$, $0$, $Successor$, $Plus$, with sentences:

$$
\forall x \, \lnot(Successor(x) = 0)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

$$
\forall x,y \, Successor(x) = Successor(y) \Rightarrow x = y
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

FOL cannot **categorically** pin down $\mathbb{N}$ (Löwenheim-Skolem) - but in practice we use intended models.

---

## Herbrand Interpretations (Preview)

For inference ([Chapter 09](../chapter-09-inference-first-order-logic/README.md)), the **Herbrand universe** is the set of all ground terms constructible from constants and functions. A **Herbrand model** grounds predicates on tuples of ground terms.

> **Readable form:** Build all possible "name expressions" from constants and functions; treat each as an object; check which ground atoms hold.

This connects FOL reasoning to (possibly infinite) propositional groundings.

---

## Truth Tables vs. Models

| | Propositional logic | First-order logic |
|---|---------------------|-------------------|
| Models | Assignments to finitely many atoms | Domains + interpretations |
| Count | Finite if $n$ atoms | Often infinite; may be uncountable |
| Entailment check | Decidable (finite models) | **Semi-decidable** |

You cannot enumerate all FOL models algorithmically in general.

---

## Semantic Pitfalls in Translation

| English | Wrong FOL | Better FOL |
|---------|-----------|------------|
| "All students like some course" | $\exists c \, \forall s \, Likes(s,c)$ | $\forall s \, \exists c \, Likes(s,c)$ |
| "There is a course all students like" | $\forall s \, \exists c \, Likes(s,c)$ | $\exists c \, \forall s \, Likes(s,c)$ |

**Order of quantifiers matters** when alternating $\forall$ and $\exists$.

---

## Connection to Agents

A knowledge-based agent's **KB** is a set of sentences intended to describe the **real world** - the intended model. **ASK** queries test entailment: does the KB logically force the query in all models of the KB?

If $KB \models Safe(2,1)$, every world consistent with the KB has safe square (2,1) - strong guarantee for Wumpus agents.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed. This four-step checklist catches most knowledge-engineering errors early.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 Part III is cumulative - inference (Chapter 09), representation (Chapter 10), and planning (Chapter 11) share the same logical foundations from Chapters 07-08.


---

## Reflection Questions

1. Construct a model where $\forall x \, \exists y \, Parent(x,y)$ is true but $\exists y \, \forall x \, Parent(x,y)$ is false.
2. Is $\exists x \, \forall y \, x = y$ satisfiable? Valid?
3. Explain why closed sentences have truth independent of variable assignment.
4. Give two models of $\exists x \, Student(x)$ with different domain sizes.
5. How does entailment differ from "true in the intended real-world model"?

---

**Next:** [Section 8.4 - Using FOL](./section-04-using-fol.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Section 8.2.
2. Enderton, H. B. - Tarski semantics for FOL.
3. Stanford Encyclopedia - "First-order Model Theory." [plato.stanford.edu](https://plato.stanford.edu/entries/model-theory-first-order/)
4. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python) - `pl_true`, model checking for finite domains.
5. Hodges, W. - *A Shorter Model Theory*.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Model** | Pair $\langle \mathcal{D}, I \rangle$: domain plus interpretation of symbols. |
| **Domain** | Nonempty set of objects over which quantifiers range. |
| **Interpretation** | Maps constants, predicates, and functions to objects, relations, and functions on $\mathcal{D}$. |
| **Satisfaction** | $(M,\sigma) \models \alpha$: sentence $\alpha$ is true in $M$ under assignment $\sigma$. |
| **Valid** | True in every model. |
| **Entailment** | $\alpha \models \beta$: every model of $\alpha$ satisfies $\beta$. |
| **Variable assignment** | Maps variables to domain objects for evaluating open formulas. |

