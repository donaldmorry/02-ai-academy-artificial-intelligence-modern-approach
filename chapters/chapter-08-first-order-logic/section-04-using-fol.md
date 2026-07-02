# Section 8.4: Using FOL

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 8.3  
> **Previous:** [Section 8.3 - FOL Semantics](./section-03-fol-semantics.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Kinship Domain

Russell and Norvig use **family relationships** as the canonical FOL tutorial domain because the objects (people), relations (parent, spouse, sibling), and recursive rules (ancestor) mirror everyday reasoning.

### Vocabulary

| Symbol | Meaning |
|--------|---------|
| $Parent(x,y)$ | $x$ is a parent of $y$ |
| $Spouse(x,y)$ | $x$ and $y$ are married |
| $Male(x)$, $Female(x)$ | gender (optional) |
| $Mother(x) = y$ | $y$ is $x$'s mother (function) |
| $Father(x) = y$ | $y$ is $x$'s father |

### Basic axioms

**Parent from mother or father:**

$$
\forall c \, Parent(Mother(c), c) \land Parent(Father(c), c)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

**Spouse symmetry:**

$$
\forall x,y \, Spouse(x,y) \Leftrightarrow Spouse(y,x)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

**Grandparent:**

$$
\forall x,y,z \, Parent(x,y) \land Parent(y,z) \Rightarrow Grandparent(x,z)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

**Ancestor (recursive):**

$$
\forall x,y \, Parent(x,y) \Rightarrow Ancestor(x,y)
$$

$$
\forall x,y,z \, Ancestor(x,y) \land Ancestor(y,z) \Rightarrow Ancestor(x,z)
$$
> **Readable form:** Everyone's mother and father are parents; grandparent means parent chain of length 2; ancestor means any parent chain of any length.

Given facts $Parent(John,Mary)$, $Parent(Mary,Sue)$: $KB \models Grandparent(John,Sue)$ and $KB \models Ancestor(John,Sue)$. Inference unfolds the recursive rule ([Chapter 09](../chapter-09-inference-first-order-logic/README.md)).

---

## Sets, Lists, and Functions

FOL has no built-in set theory syntax, but we **encode** collections:

### Set membership

Predicate $Member(x,s)$ - object $x$ belongs to set $s$.

**Singleton set:**

$$
\forall x \, Member(x,\{a\}) \Leftrightarrow x = a
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

Practical ontologies use **named constants** for finite sets: $Course101Students = \{Alice, Bob\}$ as many ground $Member$ facts.

### Lists

**Cons** function $\text{Cons}(head, tail)$, **Nil** empty list:

$$
\forall x,l \, Length(\text{Cons}(x,l)) = 1 + Length(l)
$$

$$
Length(Nil) = 0
$$
> **Readable form:** Length of a list = 1 + length of tail; empty list has length 0.

---

## Circuits and Arithmetic

Digital circuits illustrate **structural** modeling: gates as functions/predicates on signals.

### One-bit full adder

Signals: inputs $A,B,C_{in}$; outputs $Sum$, $C_{out}$.

$$
\forall a,b,c \, Sum(a,b,c) = XOR(XOR(a,b), c)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

$$
\forall a,b,c \, Carry(a,b,c) = OR(AND(a,b), AND(c, XOR(a,b)))
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

Russell and Norvig show how circuit verification reduces to proving output predicates follow from input predicates and gate laws - theorem proving as **hardware verification**.

---

## Wumpus World Revisited

Propositional version ([Chapter 07](../chapter-07-logical-agents/README.md)) used $B_{x,y}$, $P_{x,y}$ per cell. FOL version:

**Breeze iff adjacent pit:**

$$
\forall x,y \, Breeze(x,y) \Leftrightarrow \exists u,v \, Adjacent((x,y),(u,v)) \land Pit(u,v)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

**Stench iff adjacent wumpus:**

$$
\forall x,y \, Stench(x,y) \Leftrightarrow \exists u,v \, Adjacent((x,y),(u,v)) \land Wumpus(u,v)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

**Safety:**

$$
\forall x,y \, \lnot Pit(x,y) \land \lnot Wumpus(x,y) \Rightarrow Safe(x,y)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

One agent program: TELL general laws once; TELL percepts each step; ASK $Safe(x,y)$ before moving.

---

## Database Analogy

| Relational DB | FOL |
|---------------|-----|
| Row in `Parent` table | Ground atom $Parent(John,Mary)$ |
| SQL view with join | Rule with variables |
| Constraint | Sentence that must hold in all models |
| Query | Entailment or model finding |

$$
\text{SELECT child FROM Parent WHERE parent='John'}
$$
> **Readable form:** The equality states that two terms denote the same object or value inside the model.

corresponds to asking: $\exists c \, Parent(John,c) \land Query(c)$ - backward chaining finds bindings for $c$.

---

## Worked Translation: Campus Map

| English | FOL |
|---------|-----|
| MainHall is a building | $Building(MainHall)$ |
| CS dept is in MainHall | $In(DeptCS, MainHall)$ |
| Every department has a chair | $\forall d \, Department(d) \Rightarrow \exists p \, Chair(p,d)$ |
| Alice is enrolled in AI101 | $Enrolled(Alice, AI101)$ |
| All AI101 students must know logic | $\forall s \, Enrolled(s,AI101) \Rightarrow KnowsLogic(s)$ |

**Entailment:** if Alice is enrolled in AI101 and the rule holds, $KB \models KnowsLogic(Alice)$.

---

## Implementation Sketch

```python
class FOLKB:
    def __init__(self):
        self.facts = set()      # ('Parent', 'John', 'Mary')
        self.rules = []         # (premises, conclusion template)

    def tell(self, atom):
        self.facts.add(atom)

    def ask(self, query):
        if query in self.facts:
            return True
        for rule in self.rules:
            if self._rule_entails(rule, query):
                return True
        return False
```

Real systems use unification ([Section 9.2](../chapter-09-inference-first-order-logic/section-02-unification.md)).

---

## Design Tips

1. **Choose predicates for properties** ($Student(x)$) vs. **functions for unique values** ($Age(x)=25$) when exactly one value exists.
2. **Avoid overloading** - separate $EnrolledIn(s,c)$ from $Teaches(p,c)$.
3. **General laws first**, ground facts second - matches agent TELL order.
4. **Test queries** early - bad vocabulary shows up as awkward rules.


---

## Reflection Questions

1. Write ancestor rules without using $\Rightarrow$ - use only $\lor$ and $\lnot$ (clausal form preview).
2. Encode "exactly one wumpus on the grid" for a 4×4 Wumpus World.
3. How would you represent a list $[a,b,c]$ without Lisp-style cons?
4. Which circuit facts are ground vs. quantified?
5. Map three SQL queries to FOL entailment questions.

---

**Next:** [Section 8.5 - Knowledge Engineering](./section-05-knowledge-engineering.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Section 8.3.
2. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python) - kinship and Wumpus FOL KBs.
3. Codd, E. F. - relational model vs. logic (historical parallel).
4. MIT 6.034 - Wumpus FOL encoding lab.
5. OWL Primer - modern successor for ontologies ([Chapter 10](../chapter-10-knowledge-representation/README.md)).

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Ground atom** | Predicate with constant arguments only; no variables. |
| **Axiom schema** | Pattern generating infinitely many axioms (e.g., induction). |
| **Domain encoding** | Representing specialized structures (lists, circuits) in pure FOL. |
| **Recursive rule** | Rule whose conclusion appears in its own premise chain (ancestor). |
| **Structural model** | FOL description mirroring physical/compositional structure (gates, lists). |
| **Entailment query** | Question whether KB logically forces a sentence in all models. |
