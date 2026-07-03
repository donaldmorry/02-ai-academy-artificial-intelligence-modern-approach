# Section 16.6: Human Biases

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 16.8  
> **Previous:** [Section 16.5 - Decision Analysis Process](./section-05-decision-analysis-process.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Normative vs. Descriptive Decision Theory

[Sections 16.1-16.5](./section-01-combining-beliefs-and-desires.md) describe **normative** theory: how a rational agent *should* decide if it satisfies preference axioms and maximizes expected utility.

**Descriptive** theory asks: how do humans *actually* decide? Kahneman and Tversky's research (1970s-2000s) documented systematic **biases** - predictable deviations from MEU. AI systems interacting with humans must account for both.

> **Readable form:** Normative theory is the math textbook answer. Descriptive theory is what people really do - often wrong in predictable ways.

---

## Prospect Theory Overview

**Expected utility theory** assumes:

- Preferences over final wealth states
- Linear weighting of probabilities
- Risk aversion explained by concave $U(W)$

**Prospect theory** (Kahneman & Tversky, 1979) proposes:

1. **Reference dependence** - outcomes evaluated as gains/losses relative to a reference point, not absolute wealth
2. **Loss aversion** - losses loom larger than equivalent gains: $U(-x) < -U(+x)$
3. **Diminishing sensitivity** - concave for gains, convex for losses (S-shaped value function)
4. **Probability weighting** - overweight small probabilities, underweight moderate/large ones

$$
V = \sum_i \pi(p_i) \cdot v(\Delta x_i)
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

where $v$ is the value function (not standard utility) and $\pi$ distorts probabilities.

---

## The Value Function Shape

Typical prospect theory value function:

v(x) = \begin{cases}
x^\alpha & x \geq 0 \text{ (gains)} \\
-\lambda (-x)^\beta & x < 0 \text{ (losses)}
\end{cases}

with $\alpha, \beta \approx 0.88$ and **loss aversion** $\lambda \approx 2.25$.

| Phenomenon | EU prediction | Prospect theory |
|------------|---------------|-----------------|
| Reject fair ±\$100 bet | Risk aversion (concave U) | Loss aversion + reference point |
| Buy lottery + insurance | Inconsistent risk attitude | Different domains, weighting |
| Endowment effect | Shouldn't matter | Losing owned item hurts more |

> **Readable form:** Losing \$100 feels about twice as bad as gaining \$100 feels good - for many people, at typical stakes.

---

## Framing Effects

**Framing:** logically equivalent descriptions yield different choices.

**Asian disease problem** (Tversky & Kahnedy):

- **Gain frame:** "200 saved" vs. "1/3 chance 600 saved" - risk-averse choice
- **Loss frame:** "400 die" vs. "2/3 chance 600 die" - risk-seeking choice

Same outcomes; different frames → different preferences. MEU on the same outcome space should be **invariant** to framing - humans violate this.

**AI implication:** UI wording changes user decisions (opt-in vs. opt-out defaults, "95% success" vs. "5% failure").

---

## Base-Rate Neglect

Given test result, humans often ignore **prior probability** (base rate):

- Disease prevalence 1%, test 95% accurate
- Positive test → many intuit $P(\text{disease}) \approx 95\%$
- Correct (Bayesian): $P \approx 16\%$ with these numbers

[Chapter 12](../chapter-12-quantifying-uncertainty/README.md) Bayes' rule fixes this **normatively**; humans need training or automated support.

**Representativeness heuristic:** "Looks like disease" → ignore base rate.

---

## Anchoring and Adjustment

Initial numbers **anchor** estimates even when irrelevant:

- Spin a wheel (1-100), then estimate African UN members
- Higher anchor → higher estimates

In negotiations and forecasting, first offers anchor outcomes. Decision analysis mitigates by **structured elicitation** independent of anchors ([Section 16.5](./section-05-decision-analysis-process.md)).

---

## Overconfidence

Experts often assign **too-narrow confidence intervals**:

- "90% sure true value in this range" - true value falls outside ~45% of the time (calibration studies)

**Planning fallacy:** projects finish late despite experience - underestimate tails.

AI systems reporting calibrated probabilities ([Chapter 12](../chapter-12-quantifying-uncertainty/README.md)) should be **evaluated for calibration**, not just accuracy.

---

## Conjunction and Availability

**Conjunction fallacy:** $P(A \land B)$ judged higher than $P(A)$ - violates probability axioms.

**Linda problem:** "Bank teller AND feminist" ranked more probable than "bank teller" because the story "fits."

**Availability heuristic:** judge frequency by ease of recall - plane crashes feel common after news coverage.

---

## Table: Bias → MEU Violation → Mitigation

| Bias | MEU violation | Mitigation |
|------|---------------|------------|
| Loss aversion | Distorted utilities | Separate gains/losses; reference reset |
| Framing | Same EU, different choice | Standardize outcome descriptions |
| Base-rate neglect | Wrong $P$ | Force Bayes display; show tree |
| Anchoring | Biased inputs | Blind elicitation; ranges |
| Overconfidence | Too-sharp distributions | Calibration training; wider priors |
| Sunk cost | Future EU should ignore past | Explicit "from here" framing |

---

## Should AI Be Descriptive or Normative?

| Design choice | Rationale |
|---------------|-----------|
| **Normative (MEU)** | Medical, safety-critical, legal - correct by theory |
| **Descriptive** | Marketing, UX nudges - predict human behavior |
| **Hybrid** | Explain MEU recommendation, present in human-friendly frame |
| **Debiasing** | Education, checklists, automated Bayes calculators |

**Nudge ethics:** exploiting biases for good (organ donation defaults) vs. manipulation - [Chapter 27](../chapter-27-philosophy-ethics-safety/README.md).

---

## Prospect Theory vs. Utility Theory in AI

Machine learning often uses **loss functions** (cross-entropy, MSE) that don't match human perception of errors:

- False negative vs. false positive costs differ in medicine - encode in utility matrix, not 0/1 loss
- Asymmetric costs → weighted loss or cost-sensitive learning ([Course 1 Chapter 03](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/README.md))

RL reward shaping must align with true objectives - humans' loss aversion suggests **asymmetric rewards** may be needed when training agents for human-facing tasks.

---

## Python: Framing Demonstration

```python
def eu_gamble(probs, outcomes):
    return sum(p * o for p, o in zip(probs, outcomes))

# Asian disease - gain frame (lives saved, positive utilities)
program_a = eu_gamble([1.0], [200])          # 200 saved for sure
program_b = eu_gamble([1/3, 2/3], [600, 0])  # EU = 200
# Many prefer A (risk-averse in gains)

# Loss frame (lives lost, negative utilities)
program_c = eu_gamble([1.0], [-400])         # 400 die
program_d = eu_gamble([2/3, 1/3], [-600, 0]) # EU = -400
# Many prefer D (risk-seeking in losses) - same EU, different preference
```

---

## Common Mistakes

**Assuming users are rational.** Interfaces designed for MEU agents confuse humans.

**Exploiting biases without disclosure.** Short-term gains, long-term trust loss.

**Ignoring reference points.** "You saved \$50" vs. "You missed \$50 sale" - same economics, different response.

**Treating debiasing as trivial.** Checklists help but don't eliminate biases.

---

## Nudging Without Violating Autonomy

**Libertarian paternalism** ([Thaler & Sunstein](https://www.nobelprize.org/)): structure choices (defaults, framing) to steer toward welfare while preserving freedom to opt out.

AI assistants can **present MEU-optimal defaults** with transparent probabilities - normative recommendation + descriptive presentation.

Ethical line: nudge vs. manipulate - especially when exploiting loss aversion for commercial gain.

---

## Reflection Questions

1. How does loss aversion differ from risk aversion in standard utility theory?
2. Reformulate the Asian disease problem in a single frame where MEU is obvious. Do choices still diverge?
3. Why might a rational agent (MEU) use prospect-theory-shaped presentations for humans?
4. Give an AI product example where framing changes user behavior without changing outcomes.
5. Can an agent satisfy both MEU axioms and prospect theory's value function? Explain.

---

**Previous:** [Section 16.5 - Decision Analysis Process](./section-05-decision-analysis-process.md)  
**Next:** [Section 16.7 - Multiattribute Utility](./section-07-multiattribute-utility.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 16.8. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Kahneman, D., & Tversky, A. (1979). Prospect theory. *Econometrica*.
3. Kahneman, D. (2011). *Thinking, Fast and Slow*.
4. Thaler, R. H., & Sunstein, C. R. - *Nudge*.
5. Tversky, A., & Kahneman, D. - framing and conjunction fallacy papers.


