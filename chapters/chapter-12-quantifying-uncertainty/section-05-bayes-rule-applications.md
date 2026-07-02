# Section 12.5: Bayes' Rule Applications

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 12.5  
> **Previous:** [Section 12.4 - Independence](./section-04-independence.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Medical Diagnosis

The canonical Bayes' rule application: disease $D$, test result $+$. Given:

$$
P(D) = 0.001, \quad P(+ \mid D) = 0.99, \quad P(+ \mid \neg D) = 0.05
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Compute $P(D \mid +)$:

$$
P(+) = P(+ \mid D)P(D) + P(+ \mid \neg D)P(\neg D) = 0.99 \cdot 0.001 + 0.05 \cdot 0.999 \approx 0.051
$$

$$
P(D \mid +) = \frac{0.99 \cdot 0.001}{0.051} \approx 0.019
$$
> **Readable form:** Even with a 99% accurate test, a positive result means only ~2% chance of disease when the disease is rare.

This is **base-rate neglect** - humans overweight the likelihood and ignore the prior. Bayes forces both terms.

---

## Spam Filtering

Let $S$ = spam, $w$ = word "lottery" appears. Estimate from training corpus:

$$
P(S \mid w) = \frac{P(w \mid S) \, P(S)}{P(w)}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

With many words $w_1, \ldots, w_n$, naive independence ([Section 12.6](./section-06-naive-bayes-classifier.md)) gives:

$$
P(S \mid \mathbf{w}) \propto P(S) \prod_i P(w_i \mid S)
$$
> **Readable form:** The joint probability is built by multiplying the conditional factors specified by the model.

Spam filters score entire emails; highest posterior class wins.

---

## Sensor Fusion

Robot with two noisy sensors reading the same hidden state $X$. Independent noise given $X$:

$$
P(X \mid e_1, e_2) \propto P(e_1 \mid X) \, P(e_2 \mid X) \, P(X)
$$
> **Readable form:** Multiply prior by each sensor likelihood; renormalize.

If sensors disagree, the more reliable sensor (sharper likelihood) dominates. Dependent sensors require joint likelihood models - cannot multiply blindly.

---

## Odds Form of Bayes' Rule

**Odds** $O(h) = P(h) / P(\neg h)$. Bayes in odds form:

$$
O(h \mid e) = \frac{P(e \mid h)}{P(e \mid \neg h)} \cdot O(h)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

The ratio $P(e \mid h) / P(e \mid \neg h)$ is the **likelihood ratio** or **Bayes factor**. Useful when priors are tiny (medical screening).

---

## Sequential Updating

Observing evidence $e_1$, then $e_2$:

$$
P(h \mid e_1, e_2) \propto P(e_2 \mid h, e_1) \, P(h \mid e_1)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

If $e_2$ is independent of past evidence given $h$, this simplifies to another Bayes update with prior $P(h \mid e_1)$. This is **recursive Bayesian estimation** - foundation for [Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md) filtering.

```python
def update_belief(prior, like_if_h, like_if_not_h):
  p_e = like_if_h * prior + like_if_not_h * (1 - prior)
  return like_if_h * prior / p_e

p = 0.001
p = update_belief(p, 0.99, 0.05)   # first test +
p = update_belief(p, 0.99, 0.05)   # second test +
print(f"P(D|++, prior rare) = {p:.4f}")
```

---

## Legal and Forensic Reasoning

Bayes appears in DNA evidence, fingerprint analysis, and jury instruction debates. Courts sometimes struggle with base rates - expert testimony must separate $P(e \mid h)$ from $P(h \mid e)$. AIMA uses medical examples; the math is identical.

---

## Multiple Hypotheses

With hypotheses $H_1, \ldots, H_k$:

$$
P(H_i \mid e) = \frac{P(e \mid H_i) P(H_i)}{\sum_j P(e \mid H_j) P(H_j)}
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

The denominator ensures posteriors sum to 1. For classification, argmax posterior equals argmax numerator when comparing classes.

---

## Soft Evidence and Likelihoods

Sometimes evidence is **partially observed** (noisy sensor value rather than Boolean). Replace $P(e \mid h)$ with likelihood function $L(h) = P(e_{\text{obs}} \mid h)$ for continuous $e$. Normalization still uses total probability integral or sum.

---

## Connection to Decision Networks

Posterior $P(h \mid e)$ feeds **decision nodes** in influence diagrams ([Chapter 16](../chapter-16-making-simple-decisions/README.md)). Bayes answers "what do I believe?"; decision theory answers "what should I do?"

---

## Worked Screening Example

Disease rate 1 in 10,000; test sensitivity 99%, false positive 2%. Even after positive test, posterior remains below 0.5% - most positives are false alarms. Public health messaging must communicate this to avoid panic testing.

---

## Common Mistakes

**Prosecutor's fallacy.** Confusing $P(e \mid h)$ with $P(h \mid e)$ in court.

**Double counting evidence.** Applying Bayes twice on the same datum without accounting for dependence.

**Ignoring costs.** High posterior of spam still might not justify deletion if false positive cost is huge - needs utility ([Chapter 16](../chapter-16-making-simple-decisions/README.md)).

---

## Key Takeaways

1. Base rates are essential - high test accuracy ≠ high posterior for rare events.
2. Spam filtering and diagnosis share the same Bayes template.
3. Independent sensors multiply likelihoods before renormalizing.
4. Odds form highlights how evidence shifts belief multiplicatively.
5. Sequential observations update the posterior, which becomes the next prior.



## Study Guide

Work through each equation with the readable-form blockquote, then reproduce the main formulas from memory before opening the next section.

---

## Notation Review

| Symbol | Meaning |
|--------|---------|
| $P(X)$ | Prior or marginal |
| $P(X \mid e)$ | Posterior given evidence |
| $P(X,Y)$ | Joint probability |

---

## Worked Drill

1. State the main definition.
2. Apply to a numeric toy.
3. Run the Python snippet.
4. List one common mistake.
5. Link to the **Next** section.

---

## Practice Problems

1. Define glossary terms from memory.
2. Compute a small numerical exercise.
3. Compare to [Chapter 07](../chapter-07-logical-agents/README.md) logical reasoning.
4. Preview how [Chapter 13](../chapter-13-probabilistic-reasoning/README.md) builds on this idea.

---

## Engineering Note

Calibrate probabilistic models on held-out data; base rates and independence assumptions often break silently in deployment.

---

## Deep Dive

Trace one AIMA figure or example related to this section. Annotate each arrow or table entry with the corresponding probability factor. Teaching others is the fastest way to expose gaps in your own understanding.

---


---

## Reflection Questions

1. Construct a numerical example where a 95% accurate test still yields posterior < 50%.
2. Why can't you multiply sensor likelihoods if sensors share a calibration error?
3. How does odds form help intuition for rare disease screening?
4. What role does P(S) play in spam filtering?
5. How does sequential updating connect to HMM filtering?

---

**Next:** [Section 12.6 - Naive Bayes Classifier](./section-06-naive-bayes-classifier.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Pearl, J. (1988). *Probabilistic Reasoning in Intelligent Systems*. Morgan Kaufmann.
3. Murphy, K. (2022). *Probabilistic Machine Learning*. MIT Press.
4. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python)

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Base-rate neglect** | Ignoring prior probability when interpreting diagnostic test results. |
| **Likelihood ratio** | P(e|h)/P(e|¬h); how much evidence e favors h over ¬h. |
| **Sensor fusion** | Combining multiple observations to refine P(state|evidence). |
| **Posterior** | Updated belief P(h|e) after incorporating evidence. |



