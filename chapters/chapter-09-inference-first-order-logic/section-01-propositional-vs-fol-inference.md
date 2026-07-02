# Section 9.1: Propositional vs. FOL Inference

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 9.1  
> **Previous:** [Chapter 08 - First-Order Logic](../chapter-08-first-order-logic/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## From Propositional to First-Order Inference

[Chapter 07](../chapter-07-logical-agents/README.md) gave us **resolution**, **forward chaining**, and **backward chaining** for propositional logic. [Chapter 08](../chapter-08-first-order-logic/README.md) added variables and quantifiers. The central question of this chapter: how do we **automate** inference when sentences contain $\forall$ and $\exists$?

**Propositional inference** operates on a finite set of atoms. **FOL inference** must handle **schemas** - one rule applies to infinitely many ground instances.

> **Readable form:** Instead of listing every grandparent fact, one FOL rule plus unification derives each case on demand.

---

## Grounding and the Herbrand Universe

A **ground sentence** has no variables. **Grounding** substitutes constants for variables in all possible ways.

The **Herbrand universe** $\mathcal{U}_H$ is the set of all ground terms:

$$
\mathcal{U}_H = \{c_1, c_2, \ldots, f_1(c_1), f_1(f_1(c_1)), \ldots\}
$$
> **Readable form:** The equality states that two terms denote the same object or value inside the model.

For KB with constants $\{John, Mary\}$ and function $Mother$:

$$
John, Mary, Mother(John), Mother(Mary), Mother(Mother(John)), \ldots
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

**Herbrand base:** all ground atoms $P(t_1,\ldots,t_n)$ with terms from $\mathcal{U}_H$.

| Approach | Idea | Problem |
|----------|------|---------|
| **Full grounding** | Convert to propositional logic | Infinite if functions present |
| **Lifted inference** | Operate on variables directly | Requires unification |

Russell and Norvig advocate **lifted** methods - the standard in Prolog, Datalog, and theorem provers.

---

## Entailment in FOL

$KB \models \alpha$ iff $\alpha$ is true in **every model** where all sentences in $KB$ are true.

Unlike propositional logic, FOL entailment is **semi-decidable**:

- If $KB \models \alpha$, a complete procedure (resolution refutation) **eventually** finds a proof.
- If $KB \not\models \alpha$, the procedure may **never halt**.

> **Readable form:** You can always prove what follows - but you cannot always prove what doesn't follow.

**Church's theorem:** FOL validity is undecidable. **Semi-decidability** is the best we get for complete proof systems.

---

## Inference Rule Preview

| Rule | Propositional | First-order extension |
|------|---------------|----------------------|
| Modus Ponens | $P$, $P \Rightarrow Q \vdash Q$ | **Generalized Modus Ponens** with unification |
| Resolution | On literals | After CNF + skolemization |
| Forward chaining | On definite clauses | Matching + MGU |
| Backward chaining | Goal-directed | Unification + backtracking |

Each subsequent section develops one of these ([Section 9.2](./section-02-unification.md) onward).

---

## Definite Clauses and Horn KBs

A **definite clause** has exactly one positive literal:

$$
\lnot P_1 \lor \cdots \lor \lnot P_n \lor Q \quad \equiv \quad P_1 \land \cdots \land P_n \Rightarrow Q
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**Horn KB:** conjunction of definite clauses. Forward and backward chaining are **sound and complete** for Horn FOL.

Non-Horn sentences (disjunctions of multiple positives) require resolution.

---

## Example: Kinship Grounding vs. Lifting

Rule: $\forall x,y,z \, Parent(x,y) \land Parent(y,z) \Rightarrow Grandparent(x,z)$

Facts: $Parent(John,Mary)$, $Parent(Mary,Sue)$

**Grounding approach:** generate ground rule for each triple - infinite if domain unbounded.

**Lifted approach:** unify $x{=}John$, $y{=}Mary$, $z{=}Sue$ → derive $Grandparent(John,Sue)$ in one step.

```python
# Conceptual: one rule, many instances via unification
rule = "Parent(x,y) & Parent(y,z) ==> Grandparent(x,z)"
facts = ["Parent(John,Mary)", "Parent(Mary,Sue)"]
# unify rule with facts → Grandparent(John,Sue)
```

---

## Complexity Landscape

| Fragment | Decidability | Typical algorithm |
|----------|--------------|-------------------|
| Propositional | Decidable | DPLL, resolution |
| Horn FOL | Decidable | Forward/backward chaining |
| Full FOL | Semi-decidable | Resolution |
| FOL + arithmetic | Undecidable | Theories + heuristics |

---

## Connection to Agents

The logical agent loop from [Chapter 07](../chapter-07-logical-agents/README.md) extends naturally:

1. **TELL** FOL sentences (rules + percepts)
2. **ASK** FOL queries
3. **Inference engine** returns answers or **unknown** (non-termination)

Wumpus agents benefit from lifted rules: one breeze axiom covers all squares.

---

## Worked Example: Herbrand Base Size

Domain: constants $\{a,b\}$, no functions, predicates $P/1$, $Q/2$.

**Herbrand universe:** $\{a, b\}$ (finite).

**Herbrand base atoms:** $P(a)$, $P(b)$, $Q(a,a)$, $Q(a,b)$, $Q(b,a)$, $Q(b,b)$ - six ground atoms.

With function $f/1$: universe becomes $\{a, b, f(a), f(b), f(f(a)), \ldots\}$ - **infinite**.

> **Readable form:** Functions generate infinitely many term names; grounding never finishes.

---

## Propositional Reduction (Theoretical)

For finite domain without functions, FOL can be **grounded** to propositional logic:

1. Instantiate each universally quantified sentence over all tuples of constants
2. Remove existentials via Skolem constants
3. Run DPLL or resolution on ground CNF

**Practical limit:** $|constants|^k$ groundings for $k$-ary clauses - blows up quickly.

---

## Agent Design Implications

| Design choice | Consequence |
|---------------|-------------|
| Horn rules only | Forward/backward chaining, fast |
| Arbitrary disjunctions | Need resolution |
| Function symbols | Risk non-termination |
| Finite enumeration | Grounding may work |

Wumpus FOL agents in AIMA use **Horn** rules for tractability.

---

## Study Exercise: Ground Instance Count

KB: $\forall x,y \, Parent(x,y) \Rightarrow Ancestor(x,y)$; constants $\{a,b,c\}$.

How many ground instances of the rule? $3 \times 3 = 9$ for pairs $(x,y)$.

Add function $Mother$ - Herbrand universe infinite → grounding never completes.

**Takeaway:** lifted inference essential with recursive rules and functions.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed. This four-step checklist catches most knowledge-engineering errors early.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 Part III is cumulative - inference (Chapter 09), representation (Chapter 10), and planning (Chapter 11) share the same logical foundations from Chapters 07-08.


---

## Reflection Questions

1. What is the Herbrand universe for constants {a,b} and function f?
2. Why is full grounding impractical with function symbols?
3. Explain semi-decidability in terms an agent designer would care about.
4. When does forward chaining terminate on FOL Horn KBs?
5. How does lifted inference avoid the $O(n^3)$ grounding of kinship rules?

---

**Next:** [Section 9.2 - Unification](./section-02-unification.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Section 9.1.
2. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python) - `fol_fc_ask`, `fol_bc_ask`.
3. Herbrand, J. - *Logical Investigations*.
4. Robinson, J. A. - resolution and unification.
5. Church, A. - undecidability of FOL.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Herbrand universe** | Set of all ground terms from constants and functions. |
| **Grounding** | Replacing variables with constants to form ground instances. |
| **Lifted inference** | Reasoning on variables without full propositional grounding. |
| **Semi-decidable** | Entailments provable in finite time; non-entailments may loop. |
| **Definite clause** | Disjunction with at most one positive literal. |
| **Horn KB** | Knowledge base of definite clauses. |



