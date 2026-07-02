# Section 9.5: Resolution in FOL

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 9.4  
> **Previous:** [Section 9.4 - Backward Chaining](./section-04-backward-chaining.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Propositional to FOL Resolution

[Chapter 07](../chapter-07-logical-agents/README.md) defined **resolution** on CNF clauses:

$$
\frac{\ell_1 \lor \cdots \lor \ell_k, \quad \lnot m \lor m_2 \lor \cdots}{\ell_1 \lor \cdots \lor \ell_k \lor m_2 \lor \cdots}
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

**FOL extension:** unify complementary literals $P(\ldots)$ and $\lnot P(\ldots)$ with MGU $\theta$; apply $\theta$ to resolvent.

**Lifting theorem:** every FOL resolution step corresponds to a set of propositional resolutions on ground instances.

> **Readable form:** Find a substitution making the positive and negative atom the same, then remove both and keep the rest.

---

## Converting FOL to CNF

Pipeline (AIMA):

1. **Eliminate** $\Leftrightarrow$, $\Rightarrow$ (implications to $\lor/\land$)
2. **Move** $\lnot$ inward (De Morgan, quantifier rules)
3. **Standardize** variables apart
4. **Skolemize** existentials
5. **Drop** universal quantifiers
6. **Distribute** $\lor$ over $\land$ → CNF

### Skolemization

$$
\forall x \, \exists y \, Parent(x,y) \Rightarrow Child(y,x)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

Eliminate $\Rightarrow$:

$$
\forall x \, \lnot \exists y \, Parent(x,y) \lor Child(y,x)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

$$
\forall x \, \forall y \, \lnot Parent(x,y) \lor Child(y,x)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

No $\exists$ left - if we had:

$$
\exists y \, Owns(John,y)
$$
> **Readable form:** The existential quantifier says at least one object must make the stated relationship true.

becomes $Owns(John, Sk_{John})$ - **Skolem constant** (depends on free variables in scope).

\forall x \, \exists y \, Owns(x,y) \Rightarrow Owns(x, Owner(x))
$$

**Skolem function** $Owner(x)$.

$$
> **Readable form:** Replace "there exists some y" with a fresh name (or function of outer variables) - valid for refutation tests only.

**Skolemization is sound for refutation:** $KB \models \alpha$ iff $KB \cup \{\lnot \alpha\}$ unsatisfiable after Skolemizing both (with standard restrictions).

---

## Refutation Proof

To prove $KB \models \alpha$:

1. Convert $KB \land \lnot \alpha$ to CNF clause set $S$
2. Repeatedly resolve pairs in $S$, add resolvents
3. If **empty clause** $\square$ derived → **unsatisfiable** → $KB \models \alpha$

**Example (ground):**

$KB$: $\lnot King(John) \lor Greedy(John)$, $King(John)$, $\lnot Greedy(John) \lor Evil(John)$

Query: $Evil(John)$ → add $\lnot Evil(John)$

Resolve → $\lnot Greedy(John)$ → with Greedy rule → ... → $\square$

---

## FOL Resolution Example

**Clauses:**

$C_1: \lnot Parent(x,z) \lor Ancestor(x,z)$

$C_2: Parent(John,Mary)$

Resolve: $\{x/John, z/Mary\}$ → $Ancestor(John,Mary)$

**Binary resolution** produces new clause (unit clause here).

---

## Completeness

**Resolution refutation** is **refutation-complete** for FOL: if $S$ is unsatisfiable, resolution eventually derives $\square$ (given fair search strategy).

**Not decidable** - may loop if satisfiable.

**Strategies improving practice:**

- **Set of support** - resolve one clause from goal-derived set
- **Unit preference** - prioritize unit clauses
- **Subsumption** - drop clauses subsumed by others

---

## Full CNF Walkthrough

Sentence:

$$
\forall x \, Smart(x) \Rightarrow \exists y \, Teaches(x,y)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

1. $\forall x \, \lnot Smart(x) \lor \exists y \, Teaches(x,y)$
2. $\forall x \, \lnot Smart(x) \lor Teaches(x, Skill(x))$ (Skolem)
3. $\lnot Smart(x) \lor Teaches(x, Skill(x))$ (drop $\forall$)
4. Already clause form

---

## Equality Resolution

Treat equality with **paramodulation** or demodulation ([Section 8.6](../chapter-08-first-order-logic/section-06-equality-and-functions.md)):

From $a=b$ and $P(a)$, derive $P(b)$.

Pure binary resolution on equality literals needs extra axioms - modern provers integrate equality handling.

---

## vs. Chaining

| | Resolution | Horn chaining |
|---|------------|---------------|
| **Clauses** | Any CNF | Definite only |
| **Control** | Refutation search | Direct construction |
| **Disjunction** | Yes | No |
| **Typical use** | Theorem provers | Rules, databases |

---

## Resolution Graph (Conceptual)

```
KB clauses + ¬query
        |
    resolve pairs
        |
    new clauses
        |
    ... until □ or exhaustion
```

Search space is **huge** - heuristics essential ([Section 9.6](./section-06-efficiency-and-termination.md)).

---

## Complete Refutation Example

$KB$: $\forall x \, King(x) \Rightarrow Greedy(x)$, $King(John)$

Query: $Evil(John)$ - add axiom $\forall x \, Greedy(x) \Rightarrow Evil(x)$

**Negated:** $\lnot Evil(John)$

**CNF clauses:**

1. $\lnot King(x) \lor Greedy(x)$
2. $King(John)$
3. $\lnot Greedy(x) \lor Evil(x)$
4. $\lnot Evil(John)$

**Resolution steps:**

- 2+1: $Greedy(John)$
- 3+4: $\lnot Greedy(John)$
- $Greedy(John)$ + $\lnot Greedy(John)$ → $\square$

---

## Skolemization Scope Rules

$\exists y$ inside $\forall x$ → Skolem **function** $Sk(x)$.

$\exists y$ with no enclosing $\forall$ → Skolem **constant**.

**Wrong:** Skolemize before removing implications - can produce unsound clauses.

---

## Factoring

Clause $P(x) \lor P(y) \lor Q$ - factor to $P(x) \lor Q$ with $\{y/x\}$ when unifiable - removes redundant literals within clause.

---

## Resolution Strategies

| Strategy | Description |
|----------|-------------|
| **Linear** | Each resolvent with previous clause |
| **Input** | One clause always from original set |
| **Set of support** | Interact with goal-derived clauses |
| **Unit preference** | Prioritize unit clauses |

---

## Reflection Questions

1. Skolemize $\forall x \, \exists y \, \exists z \, R(x,y,z)$.
2. Why must $\lnot \alpha$ be CNF-converted with same pipeline as KB?
3. Derive empty clause from $\{P \lor Q, \lnot P \lor R, \lnot Q, \lnot R\}$.
4. Is resolution sound for proving satisfiability directly?
5. When does Skolemization depend on outer universal variables?

---

**Next:** [Section 9.6 - Efficiency and Termination](./section-06-efficiency-and-termination.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 9.4. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Robinson, J. A. - resolution principle.
3. aimacode/aima-python - `to_cnf`, `pl_resolution` extensions.
4. Vampire prover - practical FOL resolution. [vprover.github.io](https://vprover.github.io/)
5. Nieuwenhuis, R., & Rubio, V. - superposition for equality.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Resolution** | Inference rule combining clauses with complementary literals. |
| **Skolemization** | Eliminating $\exists$ by fresh Skolem constants/functions. |
| **Refutation** | Proving entailment by showing $KB \land \lnot \alpha$ unsatisfiable. |
| **Empty clause** | $\square$ - false; indicates unsatisfiable clause set. |
| **Lifting** | FOL resolution step represents ground resolution instances. |
| **Paramodulation** | Equality-aware inference replacing subterms using equations. |
| **CNF** | Conjunctive normal form: conjunction of disjunctions of literals. |


