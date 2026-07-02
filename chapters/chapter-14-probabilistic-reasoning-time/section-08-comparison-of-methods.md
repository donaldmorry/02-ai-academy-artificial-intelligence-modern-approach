# Section 14.8: Comparison of Methods

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 14.7
> **Previous:** [Section 14.7 - Applications](./section-07-applications.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Method Selection Guide

| Setting | Method |
|---------|--------|
| Discrete small state | HMM exact |
| Linear Gaussian continuous | Kalman |
| Nonlinear / multimodal | Particle filter |
| High-dimensional slice | Approximate DBN inference |

---

## When Linearization Fails

EKF single Gaussian cannot represent bimodal pose hypotheses after ambiguous corridor.

---

## Computational Trade-offs

Kalman: $O(n^3)$ in state dimension; particles: $O(N \cdot d)$ for $N$ particles, state dim $d$.

---

## Hybrid Pipelines

Coarse particle filter for global localization + fine Kalman for local tracking.

---

## Chapter 14 Synthesis

Temporal probability unifies filtering, smoothing, and decoding - foundation for robotics ([Chapter 26](../chapter-26-robotics/README.md)) and PPLs ([Chapter 15](../chapter-15-probabilistic-programming/README.md)).

---

## Decision Checklist

1. Discrete vs. continuous state?
2. Linear Gaussian?
3. Multimodal beliefs?
4. Online or batch?
5. Compute budget for particles vs. exact HMM?

---

## Key Takeaways

1. Match algorithm to Markov structure and noise model.
2. No single filter is universally optimal.
3. Particle and Kalman filters are complementary tools.
4. DBNs provide modeling language; algorithms provide inference.

---

## Study Guide

This section follows Russell & Norvig, *AIMA* 4th ed., Chapter 14.
**Focus:** Comparing Temporal Inference Methods. Read the section before the linked sections below.
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

- Prior: [Section 14.7](./section-07-applications.md)
- Next: [Chapter 15](../chapter-15-probabilistic-programming/README.md)
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

- **When** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **linearization** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **fails** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **particle** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **filter** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **trade-offs** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Comparison of Methods in AIMA?
2. How does Comparison of Methods relate to When linearization fails, particle filter trade-offs?
3. Give a concrete application of Comparison of Methods.
4. What assumptions does the textbook emphasize for Comparison of Methods?
5. When would an alternative to Comparison of Methods be preferable?
6. How would you verify understanding of Comparison of Methods in code?

---

**Previous:** [Section 14.7 - Applications](./section-07-applications.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Rabiner, L. (1989). A tutorial on hidden Markov models. *Proc. IEEE*.']



