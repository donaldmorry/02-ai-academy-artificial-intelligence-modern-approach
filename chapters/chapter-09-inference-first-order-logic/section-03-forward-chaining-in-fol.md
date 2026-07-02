# Section 9.3: Forward Chaining in FOL

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 9.3  
> **Previous:** [Section 9.2 - Unification](./section-02-unification.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Generalized Modus Ponens (GMP)

**Propositional Modus Ponens:**

$$
\frac{p, \quad p \Rightarrow q}{q}
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

**Generalized Modus Ponens** for definite clauses:

$$
\frac{p_1', \ldots, p_n', \quad (p_1 \land \cdots \land p_n \Rightarrow q)}{q\theta}
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

where $\theta = \text{UNIFY}(p_i', p_i)$ for each premise $p_i$ (same $\theta$ for all).

**Requirements:**

- $p_i'$ are **ground** sentences in KB (facts)
- Rule is **definite clause** (Horn): single positive literal in conclusion
- $\theta$ is the **most general unifier**

> **Readable form:** If ground facts match the rule's premises under one substitution, add the conclusion with that substitution applied.

---

## Example: Crime Detection (AIMA)

**Rule:**

$$
Criminal(x) \land Weapon(y) \land Owns(x,y) \land Sells(x,y,z) \Rightarrow American(x)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

**Facts:**

$$
Criminal(Nono), \quad Weapon(M1), \quad Owns(Nono,M1), \quad Sells(Nono,M1,West)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

**Unification:** $\theta = \{x/Nono, y/M1, z/West\}$

**Derive:** $American(Nono)$

---

## Definite Clauses and Horn KBs

A **definite clause** has form:

$$
p_1 \land p_2 \land \cdots \land p_n \Rightarrow q
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

or equivalently CNF: $\lnot p_1 \lor \cdots \lor \lnot p_n \lor q$ - at most one positive literal.

**Horn KB:** facts + definite clauses.

| Property | Horn KB |
|----------|---------|
| Forward chaining | **Complete** for positive queries |
| Termination | Guaranteed if **finite** ground facts reachable |
| Efficiency | Polynomial in facts for each pass |

Non-Horn clauses need resolution ([Section 9.5](./section-05-resolution-in-fol.md)).

---

## FOL Forward-Chaining Algorithm

```
function FOL-Forward-Chaining(KB):
    repeat
        new ← empty
        for each rule p1 ∧ ... ∧ pn ⇒ q in KB:
            for each θ such that p1θ,...,pnθ are ground facts in KB:
                if qθ is not already a fact:
                    add qθ to new
        if new is empty: return
        add all sentences in new to KB
    until no new facts
```

**Matching:** find $\theta$ such that premises ground-match facts - uses **unification** then check if result is already known.

**Incremental:** each iteration adds at least one fact or stops.

---

## Kinship Forward Chaining

**Rules:**

$$
Parent(x,y) \Rightarrow Ancestor(x,y)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

$$
Ancestor(x,y) \land Ancestor(y,z) \Rightarrow Ancestor(x,z)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

**Facts:** $Parent(John,Mary)$, $Parent(Mary,Sue)$

| Iteration | New facts |
|-----------|-----------|
| 1 | $Ancestor(John,Mary)$, $Ancestor(Mary,Sue)$ |
| 2 | $Ancestor(John,Sue)$ |
| 3 | none - halt |

Query $Ancestor(John,Sue)$: **yes** (now a fact).

---

## Efficiency: Indexing

Naive matching tries every rule against every fact set - expensive.

**Predicate indexing:** index rules by first premise predicate $p_1$ - only try $Parent$ rules when new $Parent$ facts arrive.

**Rete algorithm:** network sharing common subexpressions in rule premises - used in expert systems (CLIPS, Drools).

> **Readable form:** When a new Parent fact appears, only re-run rules that start with Parent - not the entire KB.

---

## Datalog

**Datalog** = Horn clauses **without function symbols**.

- Herbrand universe is **finite** (constants only)
- Forward chaining **always terminates**
- Used in databases (recursive SQL), BigQuery rules, knowledge graphs

**Example:**

$$
Ancestor(x,y) \leftarrow Parent(x,y)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

$$
Ancestor(x,z) \leftarrow Parent(x,y), Ancestor(y,z)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

---

## Limitations

| Issue | Detail |
|-------|--------|
| **Non-Horn** | Disjunction in conclusions - need resolution |
| **Negative goals** | $\lnot Q$ - not goal-directed; use negation as failure (Prolog) or CWA |
| **Infinite loops** | Recursive rules + infinite terms - need tabling |
| **Irrelevant work** | Derives all consequences, not just query |

Backward chaining ([Section 9.4](./section-04-backward-chaining.md)) targets queries.

---

## Comparison to Chapter 07

Propositional forward chaining: if $A$ and $A \Rightarrow B$ in KB, add $B$.

FOL: same, but $A\theta$ must **unify** with premise patterns - one rule fires for many groundings per iteration batch.

---

## Python Sketch

```python
def fol_forward_chain(kb):
    changed = True
    while changed:
        changed = False
        for rule in kb.rules:
            theta_list = match_premises(rule.premises, kb.facts)
            for theta in theta_list:
                conclusion = apply_theta(rule.conclusion, theta)
                if conclusion not in kb.facts:
                    kb.facts.add(conclusion)
                    changed = True
```

`match_premises` enumerates MGUs making all premises ground members of `facts`.

---

## Crime Example Extended

Add rule:

$$
Weapon(y) \land Sells(x,y,z) \Rightarrow Criminal(x)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

From $Weapon(M1)$, $Sells(Nono,M1,West)$ derive $Criminal(Nono)$ **before** American rule chain - forward chaining order-independent for Horn.

**Conflict resolution:** if two rules derive contradictory literals - KB **inconsistent** - forward chaining should alarm.

---

## Incremental Forward Chaining

**Rete** maintains join nodes - new fact triggers only connected rules:

```
[Parent(x,y)] --join--> [Parent(y,z)] --implies--> Grandparent(x,z)
```

New fact $Parent(Mary,Sue)$ activates join where $y=Mary$ - not entire KB scan.

---

## Conjunctive Queries via Forward Chaining

Forward chaining finds **all** ground atoms - answer $\exists x \, Criminal(x)$ by listing derived $Criminal$ instances.

**Not goal-directed** - derives irrelevant $Ancestor$ facts if rules present - backward chaining preferred for single queries.

---

## Worked GMP Trace Table

| Step | Rule applied | $\theta$ | New fact |
|------|--------------|----------|----------|
| 0 | - | - | $Parent(J,M)$, $Parent(M,S)$ |
| 1 | grandparent | $x{=}J,y{=}M,z{=}S$ | $Grandparent(J,S)$ |
| 2 | ancestor base | $x{=}J,y{=}M$ | $Ancestor(J,M)$ |
| 3 | ancestor base | $x{=}M,y{=}S$ | $Ancestor(M,S)$ |
| 4 | ancestor trans | $x{=}J,y{=}M,z{=}S$ | $Ancestor(J,S)$ |

---

## Reflection Questions

1. Apply GMP to derive $Grandparent(John,Sue)$ from grandparent rule and parent facts.
2. Why does forward chaining with function symbols risk non-termination?
3. How does predicate indexing reduce work when KB has 10,000 rules?
4. Is GMP sound? Complete for Horn KBs?
5. When is forward chaining preferable to backward chaining for a warehouse inventory KB?

---

**Next:** [Section 9.4 - Backward Chaining](./section-04-backward-chaining.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 9.3. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - `fol_fc_ask`.
3. Forgy, C. L. - Rete algorithm.
4. Abiteboul, H., et al. - Datalog and databases.
5. CLIPS documentation - forward-chaining expert systems.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Generalized Modus Ponens** | FOL inference rule matching ground facts to rule premises via unification. |
| **Definite clause** | Horn clause: at most one positive literal; enables forward chaining. |
| **Forward chaining** | Data-driven inference: derive new facts until saturation. |
| **Datalog** | Horn rules without functions; guaranteed finite termination. |
| **Rete algorithm** | Efficient pattern-matching network for forward chaining. |
| **Predicate indexing** | Organizing rules by predicate to avoid full KB scans. |
