# Section 8.2: FOL Syntax

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 8.2  
> **Previous:** [Section 8.1 - Beyond Propositional Logic](./section-01-beyond-propositional-logic.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Vocabulary and Signature

A **first-order language** is defined by a **signature** (vocabulary):

| Symbol type | Examples | Arity |
|-------------|----------|-------|
| **Constant symbols** | $John$, $0$, $Paris$ | 0 (nullary) |
| **Predicate symbols** | $Parent$, $Student$, $>$ | $n \geq 0$ |
| **Function symbols** | $Mother$, $Successor$, $Head$ | $n \geq 0$ |

Plus logical symbols: $\land, \lor, \lnot, \Rightarrow, \Leftrightarrow, \forall, \exists, (, ), ,$ and $=$.

**Arity** fixes how many arguments a predicate or function takes. $Parent$ has arity 2; $Student$ has arity 1; $>$ typically has arity 2.

> **Readable form:** The signature is the "dictionary" of names for objects, relations, and functions - syntax alone does not say what they mean ([Section 8.3](./section-03-fol-semantics.md) assigns meaning).

---

## Terms

**Terms** denote objects. Defined recursively:

1. Every **variable** $x, y, z, \ldots$ is a term.
2. Every **constant symbol** is a term.
3. If $f$ is an $n$-ary function symbol and $t_1,\ldots,t_n$ are terms, then $f(t_1,\ldots,t_n)$ is a term.

**Examples:**

$$
John, \quad x, \quad Mother(Mary), \quad Plus(Successor(0), 1)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

Nothing in pure syntax says $Mother(Mary)$ "means" Mary's mother - that is **semantic interpretation**.

---

## Atomic Sentences

If $P$ is an $n$-ary predicate and $t_1,\ldots,t_n$ are terms, then

$$
P(t_1, \ldots, t_n)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

is an **atomic sentence** (atom). Equality is special:

$$
t_1 = t_2
$$
> **Readable form:** The equality states that two terms denote the same object or value inside the model.

is an atomic sentence when $t_1, t_2$ are terms.

**Examples:**

$$
Parent(John, Mary), \quad Student(x), \quad Mother(Mary) = Sue, \quad Greater(Plus(x,1), x)
$$
> **Readable form:** The equality states that two terms denote the same object or value inside the model.

---

## Complex Sentences

Given sentences $\alpha, \beta$:

| Construct | Syntax |
|-----------|--------|
| Negation | $\lnot \alpha$ |
| Conjunction | $\alpha \land \beta$ |
| Disjunction | $\alpha \lor \beta$ |
| Implication | $\alpha \Rightarrow \beta$ |
| Biconditional | $\alpha \Leftrightarrow \beta$ |

**Parentheses** resolve ambiguity. $\lnot P(x) \land Q(x)$ parses as $(\lnot P(x)) \land Q(x)$ by convention in AIMA, but always clarify in exams.

---

## Quantified Sentences

If $\alpha$ is a sentence and $x$ is a variable:

$$
\forall x \, \alpha \qquad \exists x \, \alpha
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

are sentences. The variable $x$ in the quantifier is **bound**; other occurrences of $x$ in $\alpha$ are bound by that quantifier. Variables not bound by any quantifier are **free**.

**Example:**

$$
\forall x \, \exists y \, Parent(x, y)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

- $x$ is bound by $\forall$
- $y$ is bound by $\exists$
- No free variables → **closed sentence** (sentence without free variables)

$$
\exists y \, Parent(x, y)
$$
> **Readable form:** There exists some y such that x is a parent of y, but x itself is still unspecified.

has free variable $x$ → **open formula** (not a sentence in strict sense until $x$ is bound or replaced).

> **Readable form:** "For all x, there exists y such that x is a parent of y" - every person has at least one child (in a kinship reading).

---

## Operator Precedence (AIMA Convention)

From highest to lowest binding strength:

1. $\lnot$
2. $\land$
3. $\lor$
4. $\Rightarrow$, $\Leftrightarrow$
5. $\forall$, $\exists$ (bind everything to the right unless parenthesized)

So:

$$
\forall x \, P(x) \Rightarrow Q(x) \quad \text{means} \quad \forall x \, (P(x) \Rightarrow Q(x))
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

not $(\forall x \, P(x)) \Rightarrow Q(x)$.

---

## BNF Grammar (Simplified)

Russell and Norvig give a BNF-style grammar ([Appendix B](../chapter-b-languages-algorithms/README.md)):

```
Sentence  → AtomicSentence | ComplexSentence | QuantifiedSentence
AtomicSentence → Predicate(TermList) | Term = Term
ComplexSentence → Sentence Connective Sentence | ¬ Sentence | ( Sentence )
QuantifiedSentence → Quantifier Variable Sentence
Term → FunctionSymbol(TermList) | Constant | Variable
```

This grammar is **context-free** - parsers for FOL resemble programming-language parsers.

---

## Worked Examples: Building Formulas

### Example 1: Siblings

"Two people are siblings if they share a parent":

$$
\forall x,y \, Sibling(x,y) \Leftrightarrow \exists p \, Parent(p,x) \land Parent(p,y) \land x \neq y
$$
> **Readable form:** x and y are siblings iff some parent p is parent of both x and y, and x is not y.

### Example 2: At Least Two Students

"There are at least two distinct students":

$$
\exists x,y \, Student(x) \land Student(y) \land x \neq y
$$
> **Readable form:** The existential quantifier says at least one object must make the stated relationship true.

### Example 3: Unique Department Head

"Department $d$ has exactly one head" (sketch):

$$
\exists h \, Head(h,d) \land \forall h' \, Head(h',d) \Rightarrow h' = h
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

---

## Scope and Variable Renaming

**Scope** of $\forall x$ is the subformula $\alpha$ in $\forall x \, \alpha$. Inner quantifiers can **shadow** outer variables:

$$
\forall x \, \exists x \, P(x)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

The inner $\exists x$ binds its own $x$ - confusing; rename for clarity:

$$
\forall x \, \exists y \, P(y)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**Alpha conversion:** renaming bound variables preserves meaning. $\forall x \, Student(x)$ ≡ $\forall y \, Student(y)$.

---

## Terms vs. Sentences vs. Formulas

| Category | Truth value? | Example |
|----------|--------------|---------|
| **Term** | No - denotes object | $Mother(John)$ |
| **Atomic sentence** | Yes | $Parent(John, Mary)$ |
| **Formula** | May have free vars | $Parent(x, y)$ |
| **Sentence (closed)** | Yes in a model | $\forall x \, Student(x) \Rightarrow Enrolled(x)$ |

Inference engines often work with **clauses** - disjunctions of literals - after conversion ([Chapter 09](../chapter-09-inference-first-order-logic/README.md)).

---

## Python: Parsing FOL (Sketch)

AIMA Python represents FOL with strings and `Expr` objects:

```python
from aima_python.logic import expr

# AIMA-style (library-dependent)
s1 = expr('Parent(John, Mary)')
s2 = expr('forall x, y Parent(x,y) ==> Ancestor(x,y)')
# Syntax check: well-formed terms and predicates
```

> **Readable form:** Build a parse tree from the string; reject malformed expressions like Parent(John) if Parent requires two arguments.

---

## Common Syntax Errors

| Error | Problem |
|-------|---------|
| $Parent(John)$ | Wrong arity - Parent needs two terms |
| $\forall x \, Parent(x)$ | Missing second argument |
| $\exists x \, Student(x) \Rightarrow Enrolled(x,c)$ | Often meant $\forall x$ - classic translation bug |
| Nested same variable name | Ambiguous scope |

---

## Connection to Propositional Logic

If you **ground** all variables and **replace** predicates with propositional atoms, FOL collapses to propositional logic. FOL syntax is a **conservative extension**: every propositional sentence has an FOL reading with zero-ary predicates.

---

## Reflection Questions

1. Write BNF derivations for $\exists x \, P(x) \land Q(x)$.
2. Which variables are free in $\forall x \, (P(x,y) \Rightarrow \exists y \, R(x,y))$?
3. Express "every course has at least two enrolled students" using only $\forall, \exists, \land, \Rightarrow, =$.
4. Why must function symbols always appear inside predicate arguments, never alone as sentences?
5. Convert $\forall x \, \lnot P(x)$ to an equivalent form with $\exists$ (De Morgan for quantifiers - preview semantics).

---

**Next:** [Section 8.3 - FOL Semantics](./section-03-fol-semantics.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 8.2. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - `logic.py` expression parser. [GitHub](https://github.com/aimacode/aima-python)
3. Enderton, H. B. - formal FOL syntax and inductive definitions.
4. Wikipedia - First-order logic syntax. [en.wikipedia.org/wiki/First-order_logic](https://en.wikipedia.org/wiki/First-order_logic)
5. Stanford Encyclopedia of Philosophy - "Classical Logic." [plato.stanford.edu](https://plato.stanford.edu/entries/logic-classical/)

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Signature** | Set of non-logical symbols (constants, predicates, functions) with arities. |
| **Term** | Expression denoting an object; built from constants, variables, and functions. |
| **Atomic sentence** | Predicate applied to terms, or equality between terms. |
| **Bound variable** | Variable within the scope of a quantifier. |
| **Free variable** | Variable not bound by any quantifier in a formula. |
| **Closed sentence** | Formula with no free variables; has definite truth in a model. |
| **Arity** | Number of arguments a predicate or function symbol takes. |

