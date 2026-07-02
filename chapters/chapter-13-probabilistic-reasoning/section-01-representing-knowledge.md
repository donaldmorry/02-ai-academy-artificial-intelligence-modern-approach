# Section 13.1: Representing Knowledge

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 13.1
> **Previous:** [Chapter 12 - Quantifying Uncertainty](../chapter-12-quantifying-uncertainty/README.md)
> **Next:** [Section 13.2 - Semantics of BNs](./section-02-semantics-of-bns.md)
> **Prerequisites:** [Chapter 12 - Quantifying Uncertainty](../chapter-12-quantifying-uncertainty/README.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## From Joints to Networks

[Chapter 12](../chapter-12-quantifying-uncertainty/README.md) showed that the **full joint distribution** answers any probabilistic query but requires $O(2^n)$ parameters for $n$ Boolean variables.
A **Bayesian network** (BN) is a directed acyclic graph (DAG) plus **conditional probability tables** (CPTs) that factor the joint into local pieces.
$$
P(x_1,\ldots,x_n) = \prod_{i=1}^{n} P(x_i \mid \text{Parents}(X_i))
$$
> **Readable form:** Joint = product of each variable's probability given its parents in the graph.

---

## Nodes and CPTs

Each **node** is a random variable; each **edge** $Y \to X$ means $Y$ is a parent of $X$ in the factorization.
A **CPT** stores $P(X \mid \text{pa}(X))$ for all combinations of parent values.
| Element | Represents |
|---------|------------|
| Node | Random variable |
| Edge | Direct dependence in factorization |
| CPT | Conditional distribution given parents |
| Missing edge | Conditional independence claim |

---

## The Alarm Network (Preview)

AIMA's canonical example: `Burglary` and `Earthquake` cause `Alarm`; `JohnCalls` and `MaryCalls` depend on `Alarm`.
Five Boolean nodes - full joint needs 31 parameters; BN needs 10 (Figure 13.2).
Full semantics in [Section 13.2](./section-02-semantics-of-bns.md); construction in [Section 13.6](./section-06-bn-construction.md).

---

## Factoring the Joint

Any distribution can be written via the chain rule; a BN **commits** to a sparse parent set per variable.
$$
P(B,J,M,A,E) = P(B)\,P(E)\,P(A|B,E)\,P(J|A)\,P(M|A)
$$
> **Readable form:** Burglary and Earthquake are independent; calls depend only on Alarm.

---

## Query Types

BNs support **marginal**, **conditional**, and **MAP** queries.
- $P(\text{Burglary} \mid \text{JohnCalls})$ - diagnostic
- $P(\text{Alarm} \mid \text{Earthquake})$ - predictive
- Most likely explanation - MAP assignment

---

## Python: Tiny BN CPT

```python
# P(A|B,E) as dict keyed by (b,e)
cpt_A = {(False,False):0.001,(False,True):0.29,(True,False):0.94,(True,True):0.95}
def p_alarm(b, e): return cpt_A[(b,e)]
```

---

## Connection to Logic

BNs generalize logical implications: rules become conditional probabilities not certainties.
[Chapter 08](../chapter-08-first-order-logic/README.md) FOL cannot express uncertainty; BNs cannot express quantifiers - [Chapter 15](../chapter-15-probabilistic-programming/README.md) bridges relational structure.

---

## Key Takeaways

1. BNs factor joints using conditional independence.
2. CPT size is $O(2^k)$ for $k$ parents - keep parent sets small.
3. Graph structure encodes domain causal and statistical assumptions.
4. Exact inference algorithms in [Section 13.3](./section-03-exact-inference.md).

---

## Study Guide

This section follows Russell & Norvig, *AIMA* 4th ed., Chapter 13.
**Focus:** Representing Knowledge with Bayesian Networks. Read the section before the linked sections below.
Work through each equation with the readable-form blockquote, then close the book and reproduce the main formulas from memory.

---

## Notation Review

| Symbol | Meaning |
|--------|---------|
| $P(X)$ | Prior or marginal over $X$ |
| $P(X \mid e)$ | Posterior after evidence $e$ |
| $\mathbf{P}$ | Vector or table of probabilities |
| $\perp$ | Independence (conditional if subscripted) |
> **Readable form:** Keep symbols consistent with [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md) and the course [GLOSSARY.md](../../GLOSSARY.md).

---

## Connection Map

- Prior: [Chapter 12](../chapter-12-quantifying-uncertainty/README.md)
- Next: [Section 13.2](./section-02-semantics-of-bns.md)
- Related: [Chapter 15](../chapter-15-probabilistic-programming/README.md)
Cross-links are intentional: probabilistic reasoning in AIMA is cumulative - later chapters assume fluency with earlier formulas.

---

## Worked Drill

1. State the main theorem or algorithm in one sentence.
2. Identify inputs and outputs on a toy instance.
3. Compute or trace one step by hand.
4. Implement a 10-line Python sketch.
5. List one failure mode when assumptions break.

---

## Historical Note

Probability in AI moved from ad hoc certainty factors (MYCIN) to principled Bayesian networks (Pearl, 1980s) and modern probabilistic programming.
Understanding this history explains why AIMA emphasizes coherent probability axioms over rule confidence scores.

---

## Engineering Checklist

| Step | Action |
|------|--------|
| Model | Specify variables, structure, parameters |
| Validate | Check independencies with domain expert |
| Infer | Pick exact or approximate algorithm |
| Test | Posterior predictions on held-out cases |
| Deploy | Monitor calibration and drift |

---

## Common Exam Questions

Examiners often ask you to: (1) state definitions precisely, (2) apply one formula on a numeric toy, (3) compare two methods, (4) identify which independence assumption fails in a scenario.

---

## Python and aima-python

aimacode/aima-python includes educational implementations of probability, BN inference, and HMM routines.
Run small scripts while reading - debugging code catches gaps in understanding faster than rereading prose.

---

## Further Reading

Consult Murphy's *Probabilistic Machine Learning* for ML-oriented depth; Pearl for causal and BN foundations; course Chapter README for lab instructions.

---

## Detailed Example

Pick a three-variable instance from the section. Write the joint factorization, identify one conditional independence, and compute one query by hand or with a short script.
$$
P(A,B,C) = P(C|A,B)\,P(B|A)\,P(A)
$$
> **Readable form:** Any Bayesian network joint decomposes into this product form.

---

## Practice Problems

1. **Concept:** Define the key term introduced in this section without looking.
2. **Compute:** Apply the main formula to a numeric toy with 2-3 variables.
3. **Compare:** When does the method in this section fail vs. the next section's method?
4. **Code:** Modify the Python snippet to print intermediate factors or beliefs.
5. **Connect:** Cite one link to a prior chapter section by filename.

---

## Chapter Synthesis

Place this section in the Part IV arc: uncertainty representation → inference → time → programs → decisions.
After finishing, summarize the section in five sentences and add them to your course notes before moving to the **Next** link.

---

## Deep Dive: Assumptions and Failure Modes

Every algorithm in this section rests on explicit assumptions - Markov properties, independence, linearity, or sampling ergodicity.
List which assumption is most likely violated in your application domain. What happens to the output when it breaks?
Robust pipelines combine multiple inference methods: exact on small subgraphs, Gibbs on the rest, particle filters when beliefs are multimodal.

---

## Self-Test Checklist

| Check | Pass criterion |
|-------|----------------|
| Definitions | Can state without notes |
| Formula | Can write main equation from memory |
| Toy example | Hand-compute one step |
| Code | Script runs without error |
| Links | Can name prior and next section |
> **Readable form:** If any row fails, reread that section before continuing the chapter.

---

## Quick Reference Card

Copy into your course notes:
- **Section focus:** one-sentence summary
- **Core formula:** write the main equation
- **Algorithm steps:** numbered list from the section
- **Prior link:** filename of previous section
- **Next link:** filename of next section
- **Glossary pick:** two terms you did not know before
- **Exam trap:** one misconception from the section
- **AIMA figure:** cite one figure or table number if applicable
Review this card before exams and before starting the next chapter.

---


---
## Key Terms (Glossary)

- **Joint** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **distributions** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **factoring** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **BN** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **syntax** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Representing Knowledge in AIMA?
2. How does Representing Knowledge relate to Joint distributions, factoring, BN syntax?
3. Give a concrete application of Representing Knowledge.
4. What assumptions does the textbook emphasize for Representing Knowledge?
5. When would an alternative to Representing Knowledge be preferable?
6. How would you verify understanding of Representing Knowledge in code?

---

**Previous:** [Chapter 12 - Quantifying Uncertainty](../chapter-12-quantifying-uncertainty/README.md) · **Next:** [Section 13.2 - Semantics of BNs](./section-02-semantics-of-bns.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Pearl, J. (1988). *Probabilistic Reasoning in Intelligent Systems*. Morgan Kaufmann.']



