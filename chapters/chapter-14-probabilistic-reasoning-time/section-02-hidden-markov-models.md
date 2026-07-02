# Section 14.2: Hidden Markov Models

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 14.2
> **Previous:** [Section 14.1 - Time and Uncertainty](./section-01-time-and-uncertainty.md)
> **Next:** [Section 14.3 - Inference in HMMs](./section-03-inference-in-hmms.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## HMM Definition

A **hidden Markov model** (HMM) is a temporal model with finite discrete state space.
Components: initial $P(X_0)$, transition $P(X_t \mid X_{t-1})$, emission $P(E_t \mid X_t)$.

---

## Matrix Form

Transition matrix $\mathbf{T}_{ij} = P(X_t{=}j \mid X_{t-1}{=}i)$; emission matrix $\mathbf{O}_{jk} = P(E_t{=}k \mid X_t{=}j)$.

---

## Canonical Examples

| Domain | Hidden state | Observation |
|--------|--------------|-------------|
| Weather | Rain/Sun | Umbrella |
| POS tagging | Part of speech | Word |
| Robot | Location | Color |

---

## Three Classic Problems (Rabiner)

1. Evaluation: $P(e_{1:t})$ - forward algorithm.
2. Decoding: most likely state sequence - Viterbi.
3. Learning: estimate $\mathbf{T}, \mathbf{O}$ from data.

---

## State Diagram

States as nodes; labeled edges for transition probabilities; emissions as labels on states.

---

## Umbrella HMM Walkthrough

Hidden weather $\in \{\text{Rain}, \text{Sun}\}$; observation umbrella $\in \{\text{carry}, \text{none}\}$.
Transition matrix encodes weather persistence; emission matrix encodes umbrella likelihood given weather.
Filtering increases belief in Rain after observing umbrella - preview [Section 14.3](./section-03-inference-in-hmms.md).

---

## Parameter Counting

For $|S|$ states and $|O|$ observations: transition needs $|S|^2$ parameters (rows sum to 1), emission $|S| \cdot |O|$.
Compared to tabulating joint over $T$ steps - HMM is $O(|S|^2 + |S||O|)$ per step vs. exponential in $T$.

---

## Key Takeaways

1. HMMs are BNs with repeated 2-slice structure.
2. Matrices compactly encode stationary dynamics.
3. Inference algorithms in [Section 14.3](./section-03-inference-in-hmms.md).

---

## Study Guide

This section follows Russell & Norvig, *AIMA* 4th ed., Chapter 14.
**Focus:** Hidden Markov Models. Read the section before the linked sections below.
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

- Prior: [Section 14.1](./section-01-time-and-uncertainty.md)
- Next: [Section 14.3](./section-03-inference-in-hmms.md)
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

- **Transition** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **and** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **sensor** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **models** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **joint** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **distribution** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Hidden Markov Models in AIMA?
2. How does Hidden Markov Models relate to Transition and sensor models, joint distribution?
3. Give a concrete application of Hidden Markov Models.
4. What assumptions does the textbook emphasize for Hidden Markov Models?
5. When would an alternative to Hidden Markov Models be preferable?
6. How would you verify understanding of Hidden Markov Models in code?

---

**Previous:** [Section 14.1 - Time and Uncertainty](./section-01-time-and-uncertainty.md) · **Next:** [Section 14.3 - Inference in HMMs](./section-03-inference-in-hmms.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Rabiner, L. (1989). A tutorial on hidden Markov models. *Proc. IEEE*.']



