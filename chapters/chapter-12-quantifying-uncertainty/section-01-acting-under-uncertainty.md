# Section 12.1: Acting Under Uncertainty

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 12.1  
> **Previous:** [Chapter 11 - Automated Planning](../chapter-11-automated-planning/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Why Logic Alone Fails

[Chapter 07](../chapter-07-logical-agents/README.md) gave agents the power of logical inference - but the real world rarely offers clean true/false facts. Consider dental diagnosis. A naive rule:

```
Toothache ⇒ Cavity
```

fails immediately: toothaches have many causes. Flipping the arrow does not help either - not every cavity hurts. Listing every exception leads to an endless qualification problem.

Russell and Norvig identify three reasons logic breaks down in judgmental domains (medicine, law, business, dating):

| Problem | Meaning | Example |
|---------|---------|---------|
| **Laziness** | Too hard to enumerate all conditions | Every possible cause of toothache |
| **Theoretical ignorance** | Science is incomplete | Unknown rare syndromes |
| **Practical ignorance** | Tests unavailable or inconclusive | Patient cannot afford MRI |

> **Readable form:** We use probability when we cannot write perfect rules - we assign degrees of belief instead of absolute truth.

Humorous analogy: logic is like a **bouncer with a strict dress code** ("no cavities without toothache"). Probability is the **host who says "80% chance they're on the list"** and updates as people arrive.

---

## Degrees of Belief

A probabilistic agent assigns a **degree of belief** between 0 (certainly false) and 1 (certainly true). Crucially, probability statements are relative to a **knowledge state**, not the physical world:

- Before exam: $P(\text{cavity} \mid \text{toothache}) = 0.6$
- After X-ray: $P(\text{cavity} \mid \text{toothache, dark spot}) = 0.95$

Both can be correct - they answer different questions. The patient either has a cavity or not; our **belief** changes as evidence arrives.

---

## Decision Theory: Probability + Utility

Acting under uncertainty requires more than beliefs - agents need **preferences** over outcomes. [Utility theory](../../GLOSSARY.md#utility) assigns a number to each outcome; rational agents maximize **expected utility**:

$$
EU(a) = \sum_s P(s \mid e) \, U(s, a)
$$
> **Readable form:** Expected utility = sum over possible states of (probability of state given evidence × utility of taking action a in that state).

Consider airport plans:

| Plan | P(catch flight) | Wait time | Utility trade-off |
|------|-----------------|-----------|-------------------|
| Leave 90 min early | 0.97 | Short | Often best |
| Leave 24 hours early | 0.999 | Absurd | High reliability, terrible wait |

A **decision-theoretic agent** (Figure 12.1 in AIMA) maintains a probabilistic belief state, predicts outcomes of actions, and picks the action with highest expected utility.

```python
def expected_utility(belief, action, utility_fn):
    '''EU(a) = sum_s P(s|e) * U(s, a) for discrete states.'''
    return sum(prob * utility_fn(state, action) for state, prob in belief.items())

def choose_action(belief, actions, utility_fn):
    return max(actions, key=lambda a: expected_utility(belief, a, utility_fn))

belief = {"on_time": 0.7, "traffic": 0.3}
utilities = {
    "leave_early": {"on_time": 90, "traffic": 70},
    "leave_late": {"on_time": 100, "traffic": 10},
}
best = choose_action(belief, utilities.keys(), lambda s, a: utilities[a][s])
print(f"Best plan: {best}")
```

---

## The DT-Agent Architecture

```
Percepts → Update belief state → Predict action outcomes → Maximize EU → Act
```

Compared to logical agents ([Chapter 07](../chapter-07-logical-agents/README.md)), the belief state holds **probabilities**, not just possible worlds. This same architecture extends through [Chapter 16](../chapter-16-making-simple-decisions/README.md) (utilities) and [Chapter 17](../chapter-17-making-complex-decisions/README.md) (MDPs).

```python
class DTAgent:
    '''Simplified decision-theoretic agent (AIMA Fig. 12.1).'''
    def __init__(self, transition_model, sensor_model, utility_fn):
        self.belief = {}
        self.transition = transition_model
        self.sensor = sensor_model
        self.utility = utility_fn

    def update(self, action, percept):
        self.belief = self.sensor.update(
            self.transition.predict(self.belief, action), percept)

    def act(self, actions):
        return choose_action(self.belief, actions, self.utility)
```

---

## Probability vs. Other Uncertainty Formalisms

AIMA briefly contrasts probability with alternatives from the 1970s-80s:

| Approach | Idea | AIMA stance |
|----------|------|-------------|
| **Certainty factors** | Fudge numbers on rules (MYCIN) | Can overcount evidence |
| **Fuzzy logic** | Degrees of membership in vague sets | Addresses vagueness, not belief |
| **Dempster-Shafer** | Interval beliefs | Related but less standard in modern AI |

Modern AI overwhelmingly uses **probability theory** because it has clear semantics, principled updating ([Bayes' rule](../../GLOSSARY.md#bayes-rule)), and composes with decision theory.

---

## Connection to the Wumpus World

Recall the Wumpus world from [Chapter 04](../chapter-04-search-complex-environments/README.md): the agent does not know pit locations with certainty. [Section 12.7](./section-07-wumpus-revisited.md) revisits this grid - instead of "pit or no pit," the agent maintains $P(\text{pit} \mid \text{percepts})$ for each square. Search gave safe paths; probability quantifies **how safe**.

---

## From Qualification Problem to Probability

The **qualification problem** (from [Chapter 10](../chapter-10-knowledge-representation/README.md)) asked: how do we list every condition that affects an action or diagnosis? Probability sidesteps exhaustive enumeration:

$$
P(\text{cavity} \mid \text{toothache}) = 0.6
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

summarizes countless unmodeled factors into one number learned from data or expert judgment. This is both **lazy** (we do not model every mechanism) and **principled** (the number has a clear interpretation).

---

## Observations vs. States

Agents distinguish **hidden states** (what is true in the world) from **observations** (what sensors report). Logical agents often conflate these; probabilistic agents maintain $P(\text{state} \mid \text{observations})$ explicitly. This distinction becomes central in [Chapter 13](../chapter-13-probabilistic-reasoning/README.md) (Bayesian networks) and [Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md) (HMMs).

| Concept | Example | Role |
|---------|---------|------|
| State | Cavity present | Hidden truth |
| Observation | Toothache, X-ray | Evidence |
| Belief | $P(\text{cavity} \mid e)$ | Agent knowledge |

---

## Rationality Under Uncertainty

A rational agent under uncertainty does not seek certainty - it seeks **optimal expected outcomes**. This connects three AIMA themes:

1. **Beliefs** - probability ([this chapter](./README.md))
2. **Preferences** - utility ([Chapter 16](../chapter-16-making-simple-decisions/README.md))
3. **Sequential decisions** - MDPs ([Chapter 17](../chapter-17-making-complex-decisions/README.md))

> **Readable form:** Decision theory = probability theory + utility theory. Choose the action with the best average outcome, weighted by how likely each outcome is.

---

## Key Takeaways

1. **Uncertainty is unavoidable** in real domains - logic's all-or-nothing model is too brittle.
2. **Probabilities express degrees of belief** relative to evidence, not facts about the world.
3. **Decision theory** combines probability with utility for rational action selection.
4. The **DT-agent** architecture generalizes belief-state agents from earlier chapters.
5. **Observations update beliefs** about hidden states; the next sections formalize the math.

---

## Common Mistakes

**Treating probability as vagueness.** "Probably rainy" is not fuzzy membership in {rainy, dry}; it is a degree of belief about a crisp meteorological state.

**Confusing $P(A)$ with $P(A \mid B)$.** After seeing evidence, the relevant quantity is usually conditional - developed in [Section 12.3](./section-03-conditional-probability.md).

**Ignoring base rates.** High test accuracy does not imply high posterior disease probability if the disease is rare - see [Section 12.5](./section-05-bayes-rule-applications.md).

---

## Part IV Roadmap

This chapter opens **Part IV: Uncertain Knowledge and Reasoning** in AIMA. The arc continues through Bayesian networks ([Chapter 13](../chapter-13-probabilistic-reasoning/README.md)), temporal models ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)), probabilistic programming ([Chapter 15](../chapter-15-probabilistic-programming/README.md)), and decision theory ([Chapter 16](../chapter-16-making-simple-decisions/README.md)).

| Chapter | Focus |
|--------|-------|
| 12 | Probability foundations, naive Bayes |
| 13 | Bayesian networks, inference |
| 14 | HMMs, Kalman and particle filters |
| 15 | Probabilistic programs |

---

## Historical Context

Probability entered AI prominently in the 1980s-90s as expert systems hit limits of brittle rules. Bayesian networks (Pearl) and decision analysis (Howard) provided engineering discipline. Modern ML still rests on the same axioms taught here.



## Study Guide

This section follows Russell & Norvig, *AIMA* 4th ed., Chapter 12: **Acting Under Uncertainty**. Work through each equation with the readable-form blockquote, then reproduce the main formulas from memory.

- **Previous:** [Chapter 11 - Automated Planning](../chapter-11-automated-planning/README.md)
- **Next:** [Section 12.2 - Basic Probability](./section-02-basic-probability.md)

---

## Notation Review

| Symbol | Meaning |
|--------|---------|
| $P(X)$ | Prior or marginal over $X$ |
| $P(X \mid e)$ | Posterior after evidence $e$ |
| $P(X \mid Y)$ | Conditional probability |
| $\Omega$ | Sample space of worlds |

> **Readable form:** Symbols follow [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md).

---

## Worked Drill

1. State the main rule or definition in one sentence.
2. Apply it to a numeric toy example.
3. Identify one common mistake from the section.
4. Run the Python snippet and change one parameter.
5. Explain how this section links to the **Next** section.

---

## Practice Problems

1. **Concept:** Define the key terms from the Section Glossary without looking.
2. **Compute:** Solve a small numerical exercise using the section's main formula.
3. **Compare:** How does this topic differ from logical reasoning in [Chapter 07](../chapter-07-logical-agents/README.md)?
4. **Connect:** Which [Chapter 13](../chapter-13-probabilistic-reasoning/README.md) idea builds directly on this section?

---

## Engineering Perspective

Real agents need calibrated beliefs, not just correct algorithms. When deploying naive Bayes or diagnostic Bayes, track precision/recall and calibration on held-out data - especially when priors shift in production.

---

## Historical Note

Probability replaced ad hoc certainty factors in expert systems because axiomatic beliefs compose correctly under evidence. AIMA's presentation follows the subjectivist tradition suitable for knowledge-based agents.

---

## Chapter 12 Synthesis

| Section | Topic |
|--------|-------|
| 12.1 | Acting under uncertainty |
| 12.2 | Basic probability |
| 12.3 | Conditional probability |
| 12.4 | Independence |
| 12.5 | Bayes applications |
| 12.6 | Naive Bayes |
| 12.7 | Wumpus revisited |

---


---

## Reflection Questions

1. Why can't we fix medical diagnosis rules by listing all exceptions?
2. Give an example where high probability of success does not imply a rational plan.
3. How does a probabilistic belief state differ from the belief state in Chapter 07?
4. When might fuzzy logic be more appropriate than probability?
5. What three problems does probability theory address that logic cannot?
6. How does the DT-agent architecture prepare you for Chapter 16?

---

**Next:** [Section 12.2 - Basic Probability](./section-02-basic-probability.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Pearl, J. (1988). *Probabilistic Reasoning in Intelligent Systems*. Morgan Kaufmann.
3. Shortliffe, E. (1976). MYCIN certainty factors - historical contrast.
4. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python)

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Degree of belief** | A number in [0,1] expressing how strongly an agent holds a proposition given its evidence. |
| **Decision-theoretic agent** | An agent that maintains probabilistic beliefs and selects actions maximizing expected utility. |
| **Expected utility** | Weighted average of outcome utilities, weights being outcome probabilities given evidence. |
| **Qualification problem** | The difficulty of listing all conditions under which a rule or action applies. |
