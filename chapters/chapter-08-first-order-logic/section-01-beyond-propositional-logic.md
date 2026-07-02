# Section 8.1: Beyond Propositional Logic

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 8.1-8.2  
> **Previous:** [Chapter 07 - Logical Agents](../chapter-07-logical-agents/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Why Propositional Logic Is Not Enough

[Chapter 07](../chapter-07-logical-agents/README.md) showed how knowledge-based agents represent facts with **propositional variables** and infer new facts with resolution, forward chaining, and backward chaining. That machinery works - until the world contains **objects**, **relations**, and **generalizations**.

Consider a family database. In propositional logic you might write:

```
Parent(John, Mary)
Parent(Mary, Sue)
```

But each name is a separate atomic proposition - there is no shared structure. To say "every parent of a parent is a grandparent," you cannot write one compact rule. You must **ground** (instantiate) the rule for every person:

```
(Parent(John,Mary) ∧ Parent(Mary,Sue)) → Grandparent(John,Sue)
(Parent(Alice,Bob) ∧ Parent(Bob,Carol)) → Grandparent(Alice,Carol)
...  for every triple of individuals ...
```

If the domain has $n$ people, grandparent rules alone need $O(n^3)$ sentences. Add cousins, colleagues, and course enrollments, and the knowledge base explodes.

> **Readable form:** Propositional logic treats each fact as an unrelated on/off switch. Real knowledge is about *things* and *patterns over things* - that requires a richer language.

**First-order logic (FOL)** adds **terms** (objects and functions), **predicates** (relations), **quantifiers** ($\forall$, $\exists$), and **equality** ($=$). One FOL sentence can replace thousands of propositional atoms.

---

## Objects, Relations, and Functions

Russell and Norvig organize FOL around three ideas from the real world:

| Concept | Examples | FOL representation |
|---------|----------|-------------------|
| **Objects** | people, buildings, numbers | constants: `John`, `MainHall` |
| **Relations** | parent of, enrolled in, taller than | predicates: `Parent(x,y)`, `Enrolled(s,c)` |
| **Functions** | mother of, head of department | function terms: `Mother(x)`, `Head(d)` |

A **term** denotes an object. A **sentence** (formula) is true or false.

```
Parent(John, Mary)           # atomic sentence: relation holds
Mother(Mary) = Sue           # equality between terms
∀x Parent(x, Mother(x))      # quantified sentence: everyone has a parent who is their mother
```

> **Readable form:** "For every person x, x is a parent of x's mother" - one rule covers the entire domain.

Humorous analogy: propositional logic is like storing every road in Romania as a separate boolean ("Am I in Arad? Am I in Sibiu?"). FOL is like having a **map** with cities as objects and roads as relations - you write rules about *any* city.

---

## The Expressiveness Gap

| Need | Propositional logic | First-order logic |
|------|---------------------|-------------------|
| "John is a student" | Atom `StudentJohn` | `Student(John)` |
| "All students take exams" | Infinite ground instances | `∀s \, Student(s) \Rightarrow TakesExam(s)` |
| "Someone passed" | Disjunction over individuals | `∃s \, Passed(s)` |
| "The tallest student" | No direct support | Functions + equality (later sections) |

**Compactness:** FOL can express **infinite** sets of facts with **finite** sentences. The sentence `∀x \, Human(x) \Rightarrow Mortal(x)` applies to every object in the domain - past, present, and future humans - without listing names.

**Structure:** Inference can **reuse** rules via unification ([Chapter 09](../chapter-09-inference-first-order-logic/README.md)). A single clause `Parent(x,y) ∧ Parent(y,z) ⇒ Grandparent(x,z)` derives any grandparent fact on demand.

---

## Limitations We Still Face

FOL is powerful but not magic:

1. **Still classical logic** - no built-in probability ([Chapter 12](../chapter-12-quantifying-uncertainty/README.md)).
2. **Undecidable entailment** - no algorithm guarantees halt on every query ([Section 9.6](../chapter-09-inference-first-order-logic/section-06-efficiency-and-termination.md)).
3. **Cannot quantify over predicates** - "every property of a prime > 1..." requires **second-order** logic.
4. **No default rules** - "birds typically fly" needs nonmonotonic extensions ([Chapter 10](../chapter-10-knowledge-representation/README.md)).

[Section 8.7](./section-07-comparison-of-logics.md) compares FOL with description logics and higher-order logics.

---

## Kinship: A Motivating Example

Domain: people related by `Parent`, `Spouse`, `Sibling`.

**Facts:**

$$
Parent(John, Mary), \quad Parent(Mary, Sue), \quad Spouse(John, Jane)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

**Rules (readable → FOL):**

| English | FOL |
|---------|-----|
| A parent of a parent is a grandparent | $\forall x,y,z \; Parent(x,y) \land Parent(y,z) \Rightarrow Grandparent(x,z)$ |
| Spouses share children | $\forall x,y,c \; Spouse(x,y) \land Parent(x,c) \Rightarrow Parent(y,c)$ |

From three facts and two rules, an inference engine derives `Grandparent(John, Sue)` without encoding it explicitly.

```python
# Informal forward chaining sketch
facts = {("Parent", "John", "Mary"), ("Parent", "Mary", "Sue")}
rules = [
    # Parent(x,y) & Parent(y,z) -> Grandparent(x,z)
    lambda f: {( "Grandparent", x, z) for (p1,x,y) in f if p1=="Parent"
              for (p2,y2,z) in f if p2=="Parent" and y2==y}
]
for derive in rules:
    facts |= derive(facts)
# facts now contains ("Grandparent", "John", "Sue")
```

> **Readable form:** Apply the grandparent rule to all matching parent pairs in the database; add any new grandparent facts found.

---

## Wumpus World in FOL (Preview)

[Chapter 07](../chapter-07-logical-agents/README.md) used propositional atoms like `B_{1,1}` (breeze at square (1,1)). In FOL:

$$
\forall x,y \; Breeze(x,y) \Leftrightarrow \exists u,v \; Adjacent((x,y),(u,v)) \land Pit(u,v)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

One sentence states the **general breeze-pit relationship** for all squares. Percepts become observations:

$$
\neg Breeze(1,1), \quad Stench(1,1)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

The agent's KB grows with **general laws** plus **specific percepts** - the hallmark of knowledge engineering ([Section 8.5](./section-05-knowledge-engineering.md)).

---

## From Propositions to Terms: Syntax Preview

FOL **vocabulary** contains:

- **Constant symbols:** $John$, $2$, $MainHall$
- **Predicate symbols:** $Parent$, $Student$, $>$ (arity fixed)
- **Function symbols:** $Mother$, $Plus$ (arity fixed)
- **Connectives:** $\land, \lor, \lnot, \Rightarrow, \Leftrightarrow$
- **Quantifiers:** $\forall$, $\exists$
- **Equality:** $=$

[Section 8.2](./section-02-fol-syntax.md) defines these formally. [Section 8.3](./section-03-fol-semantics.md) explains **models**: domains, interpretations, and satisfaction.

---

## Translation Practice

Translate to FOL (constants and predicates of your choice):

| # | English | Sample answer |
|---|---------|---------------|
| 1 | Every student has a mentor | $\forall s \, Student(s) \Rightarrow \exists m \, Mentor(m,s)$ |
| 2 | Some course has no students | $\exists c \, Course(c) \land \neg \exists s \, Enrolled(s,c)$ |
| 3 | No student is both a minor and a TA | $\forall s \, Student(s) \Rightarrow \neg(Minor(s) \land TA(s))$ |
| 4 | Exactly one department head | $\exists d,h \, Head(h,d) \land \forall h',d' \, Head(h',d') \Rightarrow h'=h$ |

> **Readable form (sentence 1):** For each student s, there exists some mentor m such that m mentors s.

Watch **scope**: in $\exists m \, Mentor(m,s)$, the mentor may depend on $s$ - different students may have different mentors. That is usually what "every student has a mentor" means.

---

## Connection to Course 1

Tabular ML features are often **ground facts** about objects (row = user, columns = attributes). FOL generalizes those attributes into **relations** and **rules** - the symbolic counterpart to featurized data. [Chapter 10](../chapter-10-knowledge-representation/README.md) bridges this to **knowledge graphs** used in modern systems.

---

## Common Misconceptions

**"FOL is just propositional logic with extra syntax."** The quantifiers change **semantic** structure - entailment is no longer decidable by finite truth tables.

**"∀x P(x) means P holds for one particular x."** Universal quantification ranges over **all** objects in the domain.

**"We can always finitely ground FOL for inference."** Infinite domains (natural numbers) make grounding impossible; inference must work with variables ([Chapter 09](../chapter-09-inference-first-order-logic/README.md)).

---

## Reflection Questions

1. Write a propositional encoding of three parent facts and one grandparent rule. How many ground sentences do you need for 10 people?
2. Express "not all birds fly" in FOL without using default logic.
3. Why does the Wumpus breeze rule benefit from FOL over propositional atoms per square?
4. Give an English sentence that is ambiguous until you fix quantifier scope.
5. What real-world KB would be awkward in propositional logic but natural in FOL?

---

**Next:** [Section 8.2 - FOL Syntax](./section-02-fol-syntax.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Sections 8.1-8.2. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - `logic.py`, FOL examples. [GitHub](https://github.com/aimacode/aima-python)
3. Stanford CS221 - first-order logic lectures. [stanford-cs221.github.io](https://stanford-cs221.github.io/)
4. Enderton, H. B. - *A Mathematical Introduction to Logic* (syntax vs. semantics).
5. MIT 6.034 - knowledge representation overview. [OCW](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)

---

## Section Glossary

| Term | Definition |
|------|------------|
| **First-order logic (FOL)** | Logic with objects, relations, functions, quantifiers, and equality; predicates apply to terms, not other predicates. |
| **Grounding** | Replacing all variables with constants to obtain propositional instances. |
| **Term** | Expression denoting an object: constant, variable, or function application. |
| **Predicate** | Relation symbol applied to terms; forms an atomic sentence. |
| **Quantifier** | $\forall$ (for all) or $\exists$ (there exists); binds variables in formulas. |
| **Expressiveness** | What patterns a logic can state compactly; FOL exceeds propositional logic for structured domains. |



