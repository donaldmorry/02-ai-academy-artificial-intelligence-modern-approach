# Section 9.2: Unification

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 9.2  
> **Previous:** [Section 9.1 - Propositional vs. FOL Inference](./section-01-propositional-vs-fol-inference.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## The Unification Problem

**Unification** finds a **substitution** $\theta$ making two expressions **identical**.

Given expressions $E_1$ and $E_2$, a **unifier** $\theta$ satisfies $E_1\theta = E_2\theta$.

**Example:**

$$
Parent(x, Mother(y)) \quad \text{and} \quad Parent(John, z)
$$
> **Readable form:** Match two parent expressions by finding substitutions that make their argument structures identical.

Unifier: $\theta = \{x/John, z/Mother(y)\}$ (also written $\{x \mapsto John, z \mapsto Mother(y)\}$).

> **Readable form:** Find values for variables so both sides become the same expression.

Unification is the engine behind Generalized Modus Ponens, Prolog execution, and resolution.

---

## Most General Unifier (MGU)

A unifier $\theta$ is **most general** if for every other unifier $\sigma$ there exists $\lambda$ such that $\sigma = \theta\lambda$.

**MGU** leaves variables as free as possible - critical for completeness.

| Expressions | MGU |
|-------------|-----|
| $P(x,f(x))$, $P(y,y)$ | $\{y/x, y/f(x)\}$ - fails occurs check |
| $P(x,f(x))$, $P(y,f(y))$ | $\{y/x\}$ |
| $Knows(John,x)$, $Knows(y,z)$ | $\{y/John, x/z\}$ |

---

## The Occurs Check

Substitution $\{x/t\}$ **fails** if variable $x$ **occurs in** term $t$ (and $x \neq t$).

**Why:** otherwise infinite structure: $x = f(x) = f(f(x)) = \cdots$

$$
P(x, f(x)) \text{ vs } P(y, y) \Rightarrow \text{FAIL (occurs check)}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Standard unification algorithms include the occurs check for **soundness**.

---

## Unification Algorithm (Sketch)

```
unify(x, y, θ):
  if θ = failure: return failure
  if x == y: return θ
  if x is variable: return unify-var(x, y, θ)
  if y is variable: return unify-var(y, x, θ)
  if x and y are functions with same symbol:
       unify corresponding arguments left-to-right
  else: return failure
```

```python
def unify(x, y, theta=None):
    if theta is None:
        theta = {}
    if x == y:
        return theta
    if isinstance(x, Var):
        return unify_var(x, y, theta)
    if isinstance(y, Var):
        return unify_var(y, x, theta)
    if isinstance(x, Expr) and isinstance(y, Expr) and x.op == y.op:
        return unify(x.args, y.args, theta)
    return None  # failure
```

---

## Unification vs. Matching

| | Unification | Matching |
|---|-------------|----------|
| Both sides | May have variables | One side ground |
| Direction | Bidirectional | Rule → fact |
| Use | Resolution, Prolog | Forward chaining |

**Matching** is one-way: find $\theta$ such that $Pattern\theta = GroundFact$.

---

## Multiple Unifiers

$P(x,y)$ and $P(a,a)$ → MGU $\{x/a, y/a\}$.

$P(x,y)$ and $P(a,b)$ → MGU $\{x/a, y/b\}$.

Non-unifiable: $P(x,x)$ and $P(a,b)$ when $a \neq b$.

---

## Implementation Notes

- Represent terms as trees; substitutions as dictionaries.
- Apply substitution **composition**: $(E\theta)\sigma = E(\theta\sigma)$.
- **Standardizing apart:** rename variables in clauses before resolution to avoid accidental capture.

aima-python `unify.py` implements Robinson unification with occurs check.

---

## Worked Example: Step-by-Step Unification

Unify $Knows(John, x)$ and $Knows(y, Mother(y))$:

1. Predicate symbols match: $Knows$
2. Unify $John$ with $y$ → $\theta = \{y/John\}$
3. Unify $x$ with $Mother(y)\theta = Mother(John)$ → $\theta = \{y/John, x/Mother(John)\}$

**Result:** $Knows(John, Mother(John))$ on both sides.

---

## Composition of Substitutions

If $\theta = \{x/a\}$ and $\sigma = \{a/b\}$, then $\theta\sigma = \{x/b\}$ (apply $\theta$ first, then $\sigma$ to result).

Order matters when variables chain - provers apply substitutions carefully during resolution.

---

## Unification in Type Systems

**Typed unification** requires argument sorts to match. $Mother(x)$ unifies with $Mother(John)$ only if $x$ and $John$ share sort $Person$.

Ill-typed unifications rejected early - reduces search in large KBs.

---

## Debugging Unification Failures

| Failure | Cause |
|---------|-------|
| Functor clash | $P(a)$ vs $Q(a)$ - different predicates |
| Arity mismatch | $P(a)$ vs $P(a,b)$ |
| Occurs check | $\{x/f(x)\}$ infinite |
| Constant clash (with UNA) | $a$ vs $b$ when distinct |

```python
def unify_var(var, term, theta):
    if var in theta:
        return unify(theta[var], term, theta)
    if occurs_check(var, term):
        return None
    return {**theta, var: term}
```

---

## Study Exercise: Implement unify

Complete the aima-python style unifier:

1. Handle variable-variable case
2. Handle constant-constant clash
3. Recurse on argument lists
4. Test on MGU exercises from AIMA

**Challenge:** $P(f(x),x)$ and $P(y,f(y))$ - occurs check fails.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed. This four-step checklist catches most knowledge-engineering errors early.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 Part III is cumulative - inference (Chapter 09), representation (Chapter 10), and planning (Chapter 11) share the same logical foundations from Chapters 07-08.


---

## Reflection Questions

1. What is the MGU of P(x,f(x)) and P(y,f(y))?
2. Why does the occurs check reject {x/f(x)}?
3. Distinguish unification from matching with an example.
4. What does standardizing apart prevent in resolution?
5. Implement unify-var for variables and constants.

---

**Next:** [Section 9.3 - Forward Chaining in FOL](./section-03-forward-chaining-in-fol.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Section 9.2.
2. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python) - `unify` chapter.
3. Robinson, J. A. - unification algorithm.
4. Martelli, A., & Montanari, U. - efficient unification.
5. Pereira, S. - Prolog unification history.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Unification** | Finding substitution making two expressions identical. |
| **MGU** | Most general unifier; subsumes all other unifiers. |
| **Occurs check** | Fails substitution when variable occurs inside assigned term. |
| **Substitution** | Mapping from variables to terms. |
| **Standardizing apart** | Renaming variables to avoid capture in resolution. |
| **Matching** | One-way unification against ground terms. |
