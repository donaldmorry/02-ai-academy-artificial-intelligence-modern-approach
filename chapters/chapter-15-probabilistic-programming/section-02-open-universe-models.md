# Section 15.2: Open-Universe Models

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 15.2
> **Previous:** [Section 15.1 - Relational Probability](./section-01-relational-probability.md)
> **Next:** [Section 15.3 - Probabilistic Programs](./section-03-probabilistic-programs.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Closed vs. Open Universe

**Closed-world** assumption: fixed objects. **Open-universe**: uncertain **existence** and **number**.

---

## Number Uncertainty

How many aircraft in airspace? Model $N$ then generate $N$ object traces.

---

## Existence Variables

Boolean `Exists(obj)` - object may or may not be real.

---

## BN Limitations

Standard BN notation cannot express 'possibly unbounded objects' - needs programmatic or higher-order representation.

---

## Surveillance Example

Radar returns may be clutter or aircraft; number of aircraft unknown; each track associates observations to identity.
Open-universe reasoning maintains hypotheses over existence, count, and data association jointly.
Military and air-traffic applications drove early open-universe probabilistic databases and multi-target tracking research.

---

## Key Takeaways

1. Open-universe models essential for surveillance and NLP entity linking.
2. Number uncertainty precedes identity uncertainty.
3. Probabilistic programs express open universes naturally.

---

## Study Guide

This section follows Russell & Norvig, *AIMA* 4th ed., Chapter 15.
**Focus:** Open-Universe Models. Read the section before the linked sections below.
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

- Prior: [Section 15.1](./section-01-relational-probability.md)
- Next: [Section 15.3](./section-03-probabilistic-programs.md)
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

- **Number** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **uncertainty** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **existence** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **uncertainty** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Open-Universe Models in AIMA?
2. How does Open-Universe Models relate to Number uncertainty, existence uncertainty?
3. Give a concrete application of Open-Universe Models.
4. What assumptions does the textbook emphasize for Open-Universe Models?
5. When would an alternative to Open-Universe Models be preferable?
6. How would you verify understanding of Open-Universe Models in code?

---

**Previous:** [Section 15.1 - Relational Probability](./section-01-relational-probability.md) · **Next:** [Section 15.3 - Probabilistic Programs](./section-03-probabilistic-programs.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Stan Development Team. [mc-stan.org](https://mc-stan.org/)']



