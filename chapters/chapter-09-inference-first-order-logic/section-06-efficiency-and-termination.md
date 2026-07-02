# Section 9.6: Efficiency and Termination

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 9.4  
> **Previous:** [Section 9.5 - Resolution in FOL](./section-05-resolution-in-fol.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Why FOL Inference Is Hard

**Undecidability:** no algorithm decides all FOL entailment questions.

**Semi-decidability:** complete methods exist but may not terminate on non-entailments.

**Sources of complexity:**

- Infinite Herbrand universe (functions)
- Exponential search space (resolution)
- Equality reasoning (symmetry of unification)

> **Readable form:** Proving what follows is possible; proving what doesn't follow may take forever.

---

## Termination Strategies

| Strategy | Effect |
|----------|--------|
| **Horn restriction** | Forward/backward chaining terminates on finite groundings |
| **Datalog** | No functions → finite Herbrand base |
| **Depth bounds** | Incomplete but practical |
| **Subsumption** | Drop clauses subsumed by others |
| **Indexing** | Reduce unification attempts |

**Subsumption:** clause $C_1$ subsumes $C_2$ if $C_1\theta \subseteq C_2$ for some $\theta$.

---

## Indexing Techniques

**Predicate indexing:** hash rules by first literal predicate.

**Discrimination trees:** trie of term paths for fast lookup.

**Watch literals:** DPLL-style two-watched literals for resolution.

---

## Herbrand's Theorem

$KB$ unsatisfiable iff some **finite** subset of ground instances is propositional unsatisfiable.

Connects FOL refutation to finite ground search - but finite subset may be huge.

---

## When Inference Terminates

| KB type | Termination |
|---------|-------------|
| Propositional | Always |
| Horn + finite domain | Always |
| Horn + functions | May not |
| Full FOL resolution | Semi-decidable |

---

## Resource Limits in Practice

Theorem provers use:

- **Time limits** (competition: 300s)
- **Clause limits**
- **Given-clause algorithm** with prioritization

Agents may return **unknown** rather than hang - trade completeness for responsiveness.

```python
def ask_with_timeout(kb, query, seconds=5):
    with timeout(seconds):
        return resolution_refute(kb, query)
    return "unknown"
```

---

## Worked Example: Subsumption

$C_1: \lnot P(x) \lor Q(x)$ subsumes $C_2: \lnot P(a) \lor Q(a) \lor R(b)$?

Yes - $\theta = \{x/a\}$ makes $C_1\theta$ a subset of literals in $C_2$. Drop $C_2$ as redundant.

---

## Set-of-Support Strategy

One clause in every resolution derives from **set of support** (SOS) - typically the negated query.

Focuses search on goal-relevant clauses - incomplete alone but effective heuristic.

---

## Looping vs. Non-Termination

| Phenomenon | Cause | Example |
|------------|-------|---------|
| **Loop** | Repeat same clause set | Bad Prolog rule order |
| **Infinite generation** | Function terms in Herbrand universe | $f(0), f(f(0)), \ldots$ |
| **Undecidable** | No proof exists | Entailment of non-theorem |

Distinguish **implementation bug** from **theoretical limit**.

---

## Practical Inference Budgets

Production systems set:

- Max depth / max clauses / max time
- Return **unknown** with partial proof trace
- **Anytime algorithms** - best answer so far if interrupted

Critical for embedded agents with real-time constraints.

---

## Study Exercise: Termination Analysis

Given KB with $f(0)$, $f(Succ(x))$ rules - will forward chaining terminate?

Sketch Herbrand universe growth. Contrast with Datalog program over same predicates without functions.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed. This four-step checklist catches most knowledge-engineering errors early.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 Part III is cumulative - inference (Chapter 09), representation (Chapter 10), and planning (Chapter 11) share the same logical foundations from Chapters 07-08.


---

## Indexing and Subsumption Examples

**Subsumption:** clause $P(x) \lor Q(y)$ subsumes $P(John) \lor Q(Mary)$ - delete ground instance as redundant.

**Predicate indexing:** rules for $Parent$ stored separately from $Student$ - new $Parent$ fact triggers only parent-rules.

**Discrimination tree:** trie on term structure - unification candidates found by path traversal, not full scan.

---

## Decidable vs. Semi-Decidable Regions

| Fragment | Decidable? |
|----------|------------|
| Propositional | Yes |
| Datalog | Yes |
| Horn FOL (no functions) | Yes |
| Full FOL | Semi-decidable |
| FOL + arithmetic | Undecidable |
| ALC DL | Yes (EXPTIME) |

**Practice:** stay in Horn/Datalog for streaming rules; escalate to ATP for occasional hard theorems.

---

## Resource Limits in Production

```python
def ask_bounded(kb, query, timeout_sec=5, max_inferences=100000):
    start = time.time()
    for step in inference_engine(kb, query):
        if time.time() - start > timeout_sec:
            return UNKNOWN
        if step.count > max_inferences:
            return UNKNOWN
    return step.result
```

**UNKNOWN** ≠ false - critical for safety systems.

---

## Reflection Questions

1. Why is FOL entailment undecidable?
2. Explain subsumption with an example.
3. When does a Horn KB with functions fail to terminate?
4. What is the given-clause algorithm?
5. How should agents handle inference timeouts?

---

**Next:** [Section 9.7 - Theorem Provers](./section-07-theorem-provers.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Section 9.5.
2. Church, A. - undecidability.
3. Herbrand, J. - Herbrand's theorem.
4. Sagonas, K. - tabling and termination.
5. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python) - indexing in inference.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Undecidability** | No algorithm decides all entailment questions. |
| **Subsumption** | Dropping more specific clauses covered by general ones. |
| **Given-clause** | Prover loop selecting clauses to resolve. |
| **Discrimination tree** | Index for fast term lookup. |
| **Watch literal** | Heuristic tracking active literals in clauses. |
| **Resource bound** | Time/clause limit for practical inference. |
