# Section 16.4: Value of Information

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 16.6  
> **Previous:** [Section 16.3 - Decision Networks](./section-03-decision-networks.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Should You Gather More Evidence?

A rational agent acts on current beliefs - but should it **pay** for more information before acting? A medical agent orders tests; a business agent commissions market research; a robot agent explores unknown terrain. Information has **cost** (time, money, risk) and **benefit** (better decisions).

**Value of information** quantifies the expected utility gain from observing a variable before making a decision, compared to acting immediately.

> **Readable form:** How much is it worth to look before you leap? If the peek doesn't change your decision, it's worth zero. If it might flip you to a much better action, it can be worth a lot.

---

## Value of Perfect Information (VPI)

Let $D$ be a decision, $E$ evidence already observed, and $Q$ a new query variable we *could* observe perfectly.

**Current MEU** (act now):

$$
\text{MEU}(E) = \max_d \sum_{\mathbf{x}} P(\mathbf{x} \mid E) \cdot U(d, \mathbf{x})
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**MEU with perfect knowledge of $Q$** (observe $Q$, then act optimally):

$$
\text{MEU}(E, Q) = \sum_q P(q \mid E) \max_d \sum_{\mathbf{x}} P(\mathbf{x} \mid q, E) \cdot U(d, \mathbf{x})
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**Value of perfect information:**

$$
\text{VPI}(Q \mid E) = \text{MEU}(E, Q) - \text{MEU}(E)
$$
> **Readable form:** VPI = expected utility if you knew Q for sure (and then chose the best action for each q) minus expected utility if you act now without knowing Q.

**Key properties:**

- $\text{VPI}(Q \mid E) \geq 0$ always (observation cannot hurt if you can ignore it)
- $\text{VPI}(Q \mid E) = 0$ if the optimal action is the same for every value of $Q$
- VPI is **not** the utility of knowing $Q$ - it is the **improvement in decision quality**

---

## When Is VPI Zero?

VPI is zero when $Q$ is **irrelevant** to the optimal decision - formally, when the optimal action $d^*$ is the same for all $q$:

$$
\forall q, \quad d^*(q) = d^*
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

**Example:** If treatment is optimal regardless of a noisy cosmetic test result, VPI of that test is zero - don't order it.

Conversely, VPI is high when different values of $Q$ lead to **different** optimal actions with substantially different utilities.

---

## Oil-Drilling Example (Revisited)

From [Section 16.3](./section-03-decision-networks.md):

- Prior $P(\text{Oil}) = 0.5$
- Drilling cost vs. payoff makes "drill" attractive if $P(\text{oil}) > 0.3$ (numbers illustrative)
- Seismic test shifts posterior: positive → $P(\text{oil}) = 0.85$, negative → $0.15$

| Strategy | Description |
|----------|-------------|
| Drill without test | EU based on prior 0.5 |
| Test then drill if positive | Pay test cost; drill only when posterior favors |

$$
\text{VPI}(\text{Seismic}) = \text{EU}(\text{test-then-decide}) - \text{EU}(\text{drill-now decision without test})
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

If test costs \$10,000 and VPI = \$25,000, the test is worth ordering. If VPI = \$5,000, skip it.

---

## Value of Sample Information

Real observations are **imperfect** - test sensitivity/specificity, noisy sensors, limited surveys.

**Value of sample information (VSI)** for observation $Z$:

$$
\text{VSI}(Z \mid E) = \text{MEU}(\text{observe } Z \text{, then act}) - \text{MEU}(E)
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

where the observation model is $P(Z \mid \text{true state}, E)$.

$$
0 \leq \text{VSI}(Z) \leq \text{VPI}(Q)
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

Perfect information is an upper bound; imperfect tests have lower value. A useless test has VSI = 0.

| Information type | Model | Value |
|------------------|-------|-------|
| Perfect | See true state of $Q$ | VPI |
| Noisy test | $P(Z \mid Q)$ | VSI |
| Free signal | Cost = 0 | Still bounded by VPI |

---

## Medical Testing: VPI in Practice

**Scenario:** Disease prevalence 1%. Test 99% sensitive, 99% specific. Treatment helps if sick, harms if healthy.

Without test, treat-if-prevalence-high-enough analysis may say "don't treat." With test:

- Positive result → posterior may justify treatment
- Negative result → confirms no treatment

VPI captures whether this **decision switch** justifies test cost and patient burden.

**Base-rate reminder:** Rare diseases mean even good tests produce many false positives - VSI accounts for this through $P(Q \mid Z)$, not just test accuracy.

---

## Information Gathering as a Decision

Observing is itself an **action** with cost $C_{\text{obs}}$:

$$
\text{Gather } Q \text{ if } \text{VSI}(Q) > C_{\text{obs}}
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

Multi-step information gathering: observe $Q_1$, update, optionally observe $Q_2$, etc. Optimal **information policies** form a decision tree - [Chapter 17](../chapter-17-making-complex-decisions/README.md) handles infinite horizons; decision trees handle finite observation sequences.

```
        [Root: evidence E]
         /            \
   observe Q         act now
      /    \
  q1,q2   ...   → MEU after each branch
```

---

## VPI and Sensitivity Analysis

VPI depends on **probabilities** and **utilities**. Sensitivity plots show:

- VPI vs. prior $P(\text{disease})$
- VPI vs. test cost
- Threshold where VPI drops below cost

A test worth ordering at 5% prevalence may be worthless at 0.1% - [Section 16.5](./section-05-decision-analysis-process.md) documents these thresholds for stakeholders.

---

## Worked Calculation: Binary Decision

Two actions: {A, B}. One unknown $Q \in \{0, 1\}$. Prior $P(Q=1) = 0.4$.

Utilities $U(a, q)$:

| | $Q=0$ | $Q=1$ |
|---|-------|-------|
| A | 10 | 5 |
| B | 0 | 20 |

**Without knowing Q:**

$$
\text{EU}(A) = 0.6 \times 10 + 0.4 \times 5 = 8, \quad \text{EU}(B) = 0.6 \times 0 + 0.4 \times 20 = 8
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

Tie at MEU = 8.

**With perfect Q:** If $Q=0$, pick A (10); if $Q=1$, pick B (20).

$$
\text{MEU}(Q) = 0.6 \times 10 + 0.4 \times 20 = 14
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

$$
\text{VPI}(Q) = 14 - 8 = 6
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

Knowing $Q$ is worth 6 utility units - pay up to 6 for perfect information.

---

## Exploration Connection

[Chapter 17](../chapter-17-making-complex-decisions/README.md) **multi-armed bandits** balance exploration (gathering information about arm rewards) and exploitation. VPI is the **single-step** decision-theoretic answer: explore when expected information value exceeds opportunity cost.

[Chapter 22](../chapter-22-reinforcement-learning/README.md) extends this to sequential exploration without known models.

---

## Python: VPI Calculator

```python
def meu(actions, outcomes, prob, utility):
    return max(
        sum(prob(o) * utility(a, o) for o in outcomes)
        for a in actions
    )

def meu_with_observation(actions, outcomes, obs_values, p_obs, p_outcome_given_obs, utility):
  # p_outcome_given_obs(o, q) = P(o | q)
    total = 0.0
    for q in obs_values:
        pq = p_obs(q)
        post = lambda o: p_outcome_given_obs(o, q)  # normalize if needed
        best = max(
            sum(post(o) * utility(a, o) for o in outcomes)
            for a in actions
        )
        total += pq * best
    return total

def vpi(actions, outcomes, prior, obs_values, likelihood, utility):
    meu_now = meu(actions, outcomes, prior, utility)
    p_obs = lambda q: sum(prior(o) * likelihood(q, o) for o in outcomes)
    meu_obs = meu_with_observation(actions, outcomes, obs_values, p_obs,
                                   lambda o, q: likelihood(q, o) * prior(o), utility)
    return meu_obs - meu_now
```

---

## Common Mistakes

**Confusing VPI with utility of outcomes.** VPI measures *decision improvement*, not raw payoff of learning.

**Using VPI when actions don't change.** Diagnostic curiosity alone doesn't justify VPI > 0 unless decisions differ.

**Ignoring test costs and risks.** VSI > 0 is necessary but not sufficient; need VSI > cost.

**Perfect information assumption in practice.** Real tests give VSI < VPI - budget using VSI.

---

## Reflection Questions

1. Prove that VPI is always non-negative when the agent can always ignore new information.
2. Construct an example where a perfect test has VPI = 0 but an imperfect test has VSI > 0. Is this possible?
3. How does base-rate neglect affect whether humans order tests vs. VPI analysis?
4. Relate VPI to the "explore vs. exploit" trade-off in bandits.
5. In the binary example, what prior makes action B optimal without observation?

---

**Previous:** [Section 16.3 - Decision Networks](./section-03-decision-networks.md)  
**Next:** [Section 16.5 - Decision Analysis Process](./section-05-decision-analysis-process.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 16.6. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Howard, R. A. - value of information in decision analysis.
3. Raiffa, H. - *Decision Analysis* - introductory treatment.
4. Kahneman, D., & Tversky - why humans mis-value information.
5. AIMA Python: decision network examples. [aima-python](https://github.com/aimacode/aima-python)



