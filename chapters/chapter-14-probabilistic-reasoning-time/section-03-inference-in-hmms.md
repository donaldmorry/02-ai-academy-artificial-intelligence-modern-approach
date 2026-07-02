# Section 14.3: Inference in HMMs

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 14.3
> **Previous:** [Section 14.2 - Hidden Markov Models](./section-02-hidden-markov-models.md)
> **Next:** [Section 14.4 - Dynamic Bayesian Networks](./section-04-dynamic-bayesian-networks.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Filtering (Forward Algorithm)

$$
P(X_t|e_{1:t}) = \alpha P(e_t|X_t)\sum_{x_{t-1}} P(X_t|x_{t-1})P(x_{t-1}|e_{1:t-1})
$$
> **Readable form:** Predict one step, incorporate new evidence, normalize.

---

## Prediction

Sum out future evidence: $P(X_{t+k} \mid e_{1:t})$ by repeated application of $\mathbf{T}$.

---

## Smoothing (Backward)

**Forward-backward**: combine forward messages with backward likelihoods for $P(X_k \mid e_{1:t})$.

---

## Viterbi Algorithm

Dynamic programming for most likely **path** $rg\max_{x_{0:t}} P(x_{0:t}, e_{1:t})$.
Uses max instead of sum in recursion - tracks backpointers.

---

## Python: Forward Step

```python
import numpy as np
def forward_step(belief, T, O, e):
    pred = T.T @ belief
    return O[:, e] * pred / (O[:, e] * pred).sum()
```

---

## Fixed Lag Smoothing

Online approximation: smooth with delay $d$ using limited backward window.

---

## Key Takeaways

1. Filtering updates belief online with each observation.
2. Smoothing uses future evidence - offline.
3. Viterbi finds best state path, not per-step marginals.
4. All are polynomial in state space size.

---

## Study Guide

This section follows Russell & Norvig, *AIMA* 4th ed., Chapter 14.
**Focus:** Inference in HMMs. Read the section before the linked sections below.
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

- Prior: [Section 14.2](./section-02-hidden-markov-models.md)
- Next: [Section 14.4](./section-04-dynamic-bayesian-networks.md)
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

- **Filtering** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **prediction** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **smoothing** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **Viterbi** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Inference in HMMs in AIMA?
2. How does Inference in HMMs relate to Filtering, prediction, smoothing, Viterbi?
3. Give a concrete application of Inference in HMMs.
4. What assumptions does the textbook emphasize for Inference in HMMs?
5. When would an alternative to Inference in HMMs be preferable?
6. How would you verify understanding of Inference in HMMs in code?

---

**Previous:** [Section 14.2 - Hidden Markov Models](./section-02-hidden-markov-models.md) · **Next:** [Section 14.4 - Dynamic Bayesian Networks](./section-04-dynamic-bayesian-networks.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Rabiner, L. (1989). A tutorial on hidden Markov models. *Proc. IEEE*.']
