# Section 7.3: Inference and Entailment

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 7.5  
> **Previous:** [Section 7.2 - Propositional Logic](./section-02-propositional-logic.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Entailment

**[Entailment](../../GLOSSARY.md#entailment)** is the central semantic relation: knowledge base $KB$ entails sentence $\alpha$ if $\alpha$ is true in every world where $KB$ is true.

$$
KB \models \alpha
$$
> **Readable form:** This formal sentence is the symbolic version of the relationship stated in the surrounding explanation.

Formally, let $M(KB)$ be the set of models satisfying $KB$:

$$
KB \models \alpha \iff M(KB) \subseteq M(\alpha)
$$
> **Readable form:** Entailment = every truth assignment making KB true also makes the query true.

**Logical equivalence:** $\alpha \equiv \beta$ iff $M(\alpha) = M(\beta)$. Entailment is one-directional: $KB \models \alpha$ does not imply $\alpha \models KB$.

---

## Inference

**Inference** is syntactic - deriving sentences using proof rules. Write $KB \vdash_i \alpha$ when algorithm $i$ derives $\alpha$ from $KB$.

| Property | Definition | Importance |
|----------|------------|------------|
| **Soundness** | $KB \vdash_i \alpha \Rightarrow KB \models \alpha$ | No false conclusions |
| **Completeness** | $KB \models \alpha \Rightarrow KB \vdash_i \alpha$ | All truths found |

> **Readable form:** Sound = trustworthy; complete = finds everything (for decidable fragments).

A **proof** is a sequence of inference steps from $KB$ to $\alpha$.

---

## Model Checking Algorithm

Brute-force entailment for finite symbol set $\mathcal{P}$:

```python
from itertools import product

def model_satisfies(model, kb_sentences, eval_fn):
  return all(eval_fn(s, model) for s in kb_sentences)

def entails_model_check(kb, alpha, symbols, eval_fn):
  """KB |= alpha?  Exhaustive for small |symbols|."""
  for values in product([False, True], repeat=len(symbols)):
    model = dict(zip(symbols, values))
    if model_satisfies(model, kb, eval_fn):
      if not eval_fn(alpha, model):
        return False  # counterexample found
  return True
```

**Complexity:** $O(2^{|\mathcal{P}|})$ - fine for toy Wumpus; motivates smarter methods in [Sections 7.4-7.6](./section-04-theorem-proving.md).

---

## Validity, Satisfiability, Unsatisfiability

| Term | Condition |
|------|-----------|
| **Valid** (tautology) | $\models \alpha$ - true in all models |
| **Satisfiable** | true in some model |
| **Unsatisfiable** | true in no models |

Key link for **refutation**:

$$
KB \models \alpha \iff (KB \land \neg\alpha) \text{ is unsatisfiable}
$$
> **Readable form:** To prove a query, show KB plus negated query cannot be satisfied together.

---

## Inference Methods in AIMA

| Method | Mechanism | Section |
|--------|-----------|--------|
| Truth tables / model check | Enumerate models | This section |
| **Resolution** | Refutation on CNF clauses | [7.4](./section-04-theorem-proving.md) |
| **Forward chaining** | Data-driven on Horn clauses | [7.5](./section-05-forward-chaining.md) |
| **Backward chaining** | Goal-driven on Horn clauses | [7.6](./section-06-backward-chaining.md) |

---

## Wumpus Entailment Sketch

$KB$ includes: no pit at [1,1]; breeze at [2,1] (percept); breeze axiom linking $B_{2,1}$ to neighboring pits.

Query: is $P_{2,2}$ entailed?

Enumerate pit assignments for unknown squares consistent with $KB$. If $P_{2,2}$ is true in **all** models → entailed (pit definitely there). If false in all → $\neg P_{2,2}$ entailed. Mixed → unknown (not entailed either way).

---

## Soundness vs. Completeness Trade-offs

| System | Sound | Complete | Decidable |
|--------|-------|----------|-----------|
| Propositional resolution | Yes | Yes | Yes (exponential worst case) |
| Forward chaining (Horn) | Yes | Yes (Horn) | Linear time |
| Heuristic Wumpus agent | May skip | No | Fast |

Production agents often use **sound but incomplete** inference for speed - never declare safe what isn't provable safe.

---

## Connection to Chapter 12

Logical entailment is **certain** truth in all models. When $KB$ is incomplete, [Chapter 12](../chapter-12-quantifying-uncertainty/README.md) uses $P(\alpha \mid KB)$ - degrees of belief.

---

## Worked Example

$KB = \{P \Rightarrow Q,\; P\}$. Query: $Q$?

Models satisfying $KB$: only $\{P{=}T, Q{=}T\}$ (if $P$ true, $P \Rightarrow Q$ forces $Q$). Query $Q$ true in that model → $KB \models Q$.

Counterexample search for $KB \land \neg Q$: $P{=}T, Q{=}F$ violates $P \Rightarrow Q$ → unsatisfiable → entailment confirmed.

---

## Common Mistakes

**Confusing entailment with proof.** Entailment is semantic; proof is syntactic - linked by soundness/completeness.

**Thinking "not entailed" means "false".** $\alpha$ not entailed and $\neg\alpha$ not entailed → unknown.

**Model checking at scale.** 50 symbols → $2^{50}$ models - use resolution or specialized inference.

---


---

## Extended Worked Example

Trace the algorithm on a minimal instance by hand before coding. State the invariant maintained at each step (e.g., consistency of partial assignment, backup values at MIN nodes, or domain non-emptiness after propagation).

| Step | State | Action | Notes |
|------|-------|--------|-------|
| 1 | Initial | - | Baseline |
| 2 | After first decision | - | Check constraints |
| 3 | Terminal / goal | - | Verify solution |

> **Readable form:** Hand simulation catches off-by-one errors that unit tests miss.

---

## Implementation Notes (aima-python)

Compare your code to the reference implementation in `aima-python`. Instrument node expansions, backtracks, or inference steps. Log domains after each `revise` or each alpha-beta cutoff.

| Check | Why |
|-------|-----|
| Correct stopping condition | Prevents infinite loops |
| Neighbor / action generation | Common source of bugs |
| Tie-breaking | Affects reproducibility |

---

## Comparison Table

| Aspect | Naive approach | This section's technique |
|--------|----------------|-------------------------|
| Time (worst) | Exponential | Problem-dependent |
| Memory | Varies | See section |
| Completeness | See section | See section |
| Optimality | See section | See section |

---

## AIMA Connection

Re-read the matching section in Russell & Norvig (4th ed.) and complete one end-of-chapter exercise. Connect definitions to the [GLOSSARY.md](../../GLOSSARY.md) entries cited in the section header.

---

## Common Mistakes

- Skipping worked examples and going straight to code.
- Ignoring environment assumptions from PEAS (static vs dynamic, fully vs partially observable).
- Confusing worst-case complexity with typical performance on structured instances.
- Forgetting to restore state on backtrack (CSP domains, game tree depth, belief sets).


## Reflection Questions

1. State the formal definition of $KB \models \alpha$.
2. What is a counterexample to entailment?
3. How does refutation connect entailment to unsatisfiability?
4. Why is soundness critical for a Wumpus agent?
5. When is forward chaining complete?
6. Give an example where $KB \not\models \alpha$ and $KB \not\models \neg\alpha$.

---

**Next:** [Section 7.4 - Theorem Proving with Resolution](./section-04-theorem-proving.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 7.5.
2. aima-python - `tt_entails`, `pl_resolution`. [GitHub](https://github.com/aimacode/aima-python)
3. Mendelson, E. - *Introduction to Mathematical Logic*.
4. Tarski - semantic definition of logical consequence.
5. MIT 6.034 - entailment and inference lecture.



