# Section 13.5: Gibbs Sampling

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 13.4.2
> **Previous:** [Section 13.4 - Approximate Inference](./section-04-approximate-inference.md)
> **Next:** [Section 13.6 - BN Construction](./section-06-bn-construction.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## MCMC for BNs

**Markov Chain Monte Carlo** constructs samples whose stationary distribution is $P(\mathbf{X} \mid e)$.
**Gibbs sampling** is a simple MCMC that resamples each non-evidence variable from $P(X_i \mid \text{MB}(X_i), e)$.

---

## Markov Blanket

Variables in the **Markov blanket** of $X_i$: parents, children, and children's other parents.
$$
P(x_i \mid \text{MB}(X_i)) \propto P(x_i \mid \text{pa}(X_i)) \prod_{Y \in \text{children}(X_i)} P(y \mid \text{pa}(Y))
$$
> **Readable form:** Local conditional depends only on blanket - enables efficient Gibbs updates.

---

## Gibbs Algorithm

1. Initialize non-evidence variables randomly.
2. For each $X_i$ not in evidence, sample from $P(X_i \mid \text{MB}(X_i), e)$.
3. Repeat for many iterations.
4. Average samples after **burn-in**.

---

## Burn-in and Convergence

Early samples reflect initialization, not target distribution - discard first $B$ iterations.
Diagnostics: multiple chains, Gelman-Rubin statistic (see [Chapter 15](../chapter-15-probabilistic-programming/README.md)).

---

## Blocked Gibbs and Hybrid Methods

When variables are strongly correlated, **blocked Gibbs** updates groups jointly.
Hybrid: cutset conditioning + Gibbs on remaining variables.

---

## When Gibbs Works Well

Low tree-width local structure; discrete small domains.
Struggles with deterministic or near-deterministic CPTs (mixing slow).

---

## Key Takeaways

1. Gibbs cycles through variable updates using Markov blanket conditionals.
2. Burn-in discards non-representative early samples.
3. MCMC enables approximate inference in large BNs.
4. Convergence must be checked, not assumed.

---

## Study Guide

This section follows Russell & Norvig, *AIMA* 4th ed., Chapter 13.
**Focus:** Gibbs Sampling. Read the section before the linked sections below.
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

- Prior: [Section 13.4](./section-04-approximate-inference.md)
- Next: [Section 13.6](./section-06-bn-construction.md)
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

- **MCMC** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **for** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **BNs** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **burn-in** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **convergence** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Gibbs Sampling in AIMA?
2. How does Gibbs Sampling relate to MCMC for BNs, burn-in, convergence?
3. Give a concrete application of Gibbs Sampling.
4. What assumptions does the textbook emphasize for Gibbs Sampling?
5. When would an alternative to Gibbs Sampling be preferable?
6. How would you verify understanding of Gibbs Sampling in code?

---

**Previous:** [Section 13.4 - Approximate Inference](./section-04-approximate-inference.md) · **Next:** [Section 13.6 - BN Construction](./section-06-bn-construction.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Pearl, J. (1988). *Probabilistic Reasoning in Intelligent Systems*. Morgan Kaufmann.']
