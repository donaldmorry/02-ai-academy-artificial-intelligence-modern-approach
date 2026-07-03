# Section 2.2: Good Behavior and Rationality

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 2.2  
> **Previous:** [Section 2.1 - Agents and Environments](./section-01-agents-and-environments.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## What Does "Good" Mean for an Agent?

[Section 2.1](./section-01-agents-and-environments.md) defined agents as mappings from percept sequences to actions. But which mapping is **good**? A chess program that loses every game executes *some* agent function - just not one you would deploy.

Russell and Norvig answer: **good behavior is rational behavior**, and rationality is defined relative to a [performance measure](../../GLOSSARY.md#performance-measure), the agent's prior knowledge, the actions it can perform, and the percept sequence it has received. This is the normative heart of AIMA - not "think like a human," but **do well at the job you were given**.

> **Readable form:** A rational agent does the best it can with what it knows, what it can sense, and what it cares about - not what an all-knowing god would do.

Humorous analogy: rationality is like being the **best contestant on a game show with incomplete clues**. You maximize your expected prize money given the hints so far. Omniscience would be having the producer whisper all answers - useful fantasy, irrelevant to contestant design.

---

## Performance Measures

A [performance measure](../../GLOSSARY.md#performance-measure) evaluates how well an agent is doing. Russell and Norvig insist it must be **specified from the outside** - based on the actual state of the environment, not merely on what the agent perceives.

### Designing Performance Measures

| Principle | Explanation | Pitfall if ignored |
|-----------|-------------|-------------------|
| **Be specific** | Quantify success | "Do well" is unimplementable |
| **Measure states** | Reward outcomes, not just actions | Paying per mile driven encourages circles |
| **Time matters** | Average over runs, discount future | One lucky win ≠ strong policy |
| **Align with goals** | Measure what stakeholders want | Click optimization → outrage bait |

**Vacuum cleaner example:** A reasonable measure is **amount of dirt cleaned per unit time**, averaged over many room layouts. A bad measure is **least energy used** (agent never turns on) or **fewest motor commands** (agent does nothing).

From [Section 1.5](../chapter-01-introduction/section-05-risks-and-benefits-of-ai.md), misaligned measures cause real harm: recommendation systems maximizing engagement, hiring tools optimizing historical promotion patterns. The [rational agent](../../GLOSSARY.md#rational-agent) framework makes the issue explicit: **you get what you measure**.

### Formal Sketch

Let $r_t$ be the reward or performance contribution at time $t$. A common objective is **expected cumulative performance**:

$$
\mathbb{E}\left[\sum_{t=0}^{T} \gamma^t \, r_t \right]
$$
> **Readable form:** Expected total reward over time, optionally discounting future rewards by factor gamma (values near 1 care about the long term; near 0 care about now).

where $\gamma \in [0, 1]$ is a discount factor (Chapter 17 develops this for MDPs). Chapter 2 introduces the idea; utility-based agents in [Section 2.4](./section-04-structure-of-agents.md) make it concrete.

---

## Rationality Defined

**Definition (Russell & Norvig):** For each possible percept sequence, a rational agent should select an action that is expected to maximize its [performance measure](../../GLOSSARY.md#performance-measure), given:

1. The evidence provided by the percept sequence
2. Whatever built-in knowledge the agent has

Rationality is **not** perfection. It is **optimal with respect to available information**.

$$
a^* = \arg\max_{a \in \mathcal{A}} \mathbb{E}[\text{Performance} \mid a, \text{percepts}, \text{knowledge}]
$$
> **Readable form:** Pick the action with the highest expected score given everything you've sensed and everything you were born knowing.

### What Rationality Is Not

| Term | Meaning | Relation to rationality |
|------|---------|-------------------------|
| **Omniscience** | Knows the true state of the world | Stronger than rational; impossible in practice |
| **Clairvoyance** | Knows future outcomes | Not required; expectations suffice |
| **Successful** | Actually achieved best outcome | Rational choice can fail if environment is stochastic |
| **Human-like** | Mimics human errors or intuition | Independent dimension |

A rational agent that takes out travel insurance before a flight is not clairvoyant - it accepts a small certain cost against a large uncertain loss. If the flight is fine, the purchase still may have been rational.

---

## Omniscience vs. Autonomy

**Omniscient agent:** knows the actual outcome of every action in every state. **Rational agent:** chooses actions that maximize *expected* performance given percepts and knowledge.

No real agent is omniscient. Partial observability ([Section 2.3](./section-03-the-nature-of-environments.md)) guarantees the agent's beliefs can diverge from reality.

**Autonomy** means the agent learns or discovers knowledge itself rather than relying entirely on designer-provided facts.

| Knowledge source | Autonomy level | Example |
|------------------|----------------|---------|
| Designer hard-codes all rules | Low | Hand-crafted chess evaluation |
| Designer provides learning algorithm + data | High | AlphaZero self-play |
| Hybrid | Medium | GPS map preloaded + online traffic learning |

From [Course 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-01-what-is-machine-learning.md), Mitchell's definition ties learning to performance improvement. A learning agent ([Section 2.5](./section-05-learning-agents.md)) increases autonomy over time - shifting burden from programmer to experience.

```python
# Rational action selection (conceptual)
def rational_action(percepts, knowledge, performance_measure):
    best_action = None
    best_expected_score = float("-inf")
    for action in legal_actions(percepts, knowledge):
        score = expected_performance(
            action, percepts, knowledge, performance_measure
        )
        if score > best_expected_score:
            best_expected_score = score
            best_action = action
    return best_action
```

> **Readable form:** Try every legal action, estimate how well each would score on average, pick the winner.

In large state spaces, enumeration is impossible - hence search (Chapter 03), probability (Chapter 12), and learning (Chapter 19).

---

## Rationality Depends on the Four Factors

The same agent program can be rational or irrational depending on context:

### 1. The Performance Measure

An agent maximizing **passenger safety** brakes aggressively. One maximizing **arrival time** tailgates. Same sensors, different rationality.

### 2. Prior Knowledge

A chess agent knowing opening theory plays better early moves. A medical agent with epidemiology priors diagnoses faster. Rationality is always **conditional on what you start with**.

### 3. Available Actions

A robot arm with only three joints cannot reach some objects - rationality is over **feasible** actions, not wishful ones.

### 4. Percept Sequence

Decisions must be based on **actual percepts received**, not hidden truths. A rational driver slows in fog because **visibility** (percept) degraded - even if the road ahead is empty (true state).

---

## Exploration vs. Exploitation Preview

Sometimes the rational action is to **gather information** rather than immediately maximize short-term performance. Trying a new restaurant (exploration) may reduce tonight's dinner quality but improve long-term dining choices (exploitation).

This tension appears formally in:

- [Learning agents](./section-05-learning-agents.md) - problem generator proposes exploratory actions
- Multi-armed bandits (Chapter 17)
- Reinforcement learning (Chapter 22)

A [supervised learning](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-03-supervised-vs-unsupervised-learning.md) model trained once on fixed data never explores; an agent in a changing environment must.

---

## Bounded Rationality

Real agents have **limited computation**. Russell and Norvig acknowledge that maximizing expected performance exactly may be intractable. **Bounded rationality** (Herbert Simon) means satisficing - finding **good enough** actions within time and memory limits.

| Ideal rationality | Bounded rationality |
|-------------------|---------------------|
| Optimal chess move via full search | Depth-limited search + heuristics |
| Exact Bayesian inference | Approximate sampling |
| Global optimum | Local optimum in 50 ms |

This connects to [Section 1.2](../chapter-01-introduction/section-02-foundations-of-ai.md): AI is engineering under constraints, not pure mathematics.

> **Readable form:** Real agents are rational like a chess player using a clock - not searching every line to mate, but picking the best move they can find before time runs out.

---

## Rationality and Ethics

Maximizing a performance measure is **amoral** - the measure encodes values. A rational surveillance agent maximizes detection rate; whether that is **good** is a human policy question, revisited in [Chapter 27](../chapter-27-philosophy-ethics-safety/README.md).

[Section 1.5](../chapter-01-introduction/section-05-risks-and-benefits-of-ai.md) warned that optimizing the wrong metric at scale causes systemic harm. Rationality formalizes the warning: **specify $P$ carefully before debating algorithms**.

---

## Worked Example: Taxi Agent

Consider an automated taxi (developed fully in [Section 2.7](./section-07-case-studies.md)):

- **Performance measure:** safe, legal, comfortable trip in minimum time; penalize collisions and traffic violations heavily
- **Knowledge:** road rules, map, vehicle dynamics
- **Actions:** steer, accelerate, brake, signal, communicate
- **Percepts:** cameras, GPS, speed, passenger requests

A rational taxi **yields to pedestrians** even when no police are visible - not because percepts prove a ticket, but because collision cost in the performance measure dominates shaving five seconds.

---

## Connection to Expected Utility

[Section 1.1](../chapter-01-introduction/section-01-what-is-ai.md) previewed utility maximization:

$$
\text{action}^* = \arg\max_a \sum_s P(s \mid e) \, U(s, a)
$$
> **Readable form:** Choose the action maximizing expected utility over possible world states, weighted by belief in each state given evidence.

Rationality in Chapter 2 is the **chapter-level narrative**; utility-based agents in [Section 2.4](./section-04-structure-of-agents.md) and decision theory in Chapter 16 provide the **mathematical machinery**.

---

## Reflection Questions

1. Design two different performance measures for the same email spam filter. How would rational behavior differ?
2. Can an agent be rational yet harmful to society? Give an example.
3. Why is omniscience neither required nor achievable for rational agents?
4. How does bounded rationality explain why Deep Blue used depth-limited search instead of exhaustive game-tree search?

---

**Next:** [Section 2.3 - The Nature of Environments](./section-03-the-nature-of-environments.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 2.2. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Stanford CS221 - Lecture notes on rational agents. [Course site](https://stanford-cs221.github.io/)
3. Simon, H. A. (1957). *Models of Man* - bounded rationality. [Stanford Encyclopedia: Bounded Rationality](https://plato.stanford.edu/entries/bounded-rationality/)
4. Amodei, D., et al. (2016). "Concrete Problems in AI Safety." [arXiv:1606.06565](https://arxiv.org/abs/1606.06565) - misaligned performance measures
5. Russell, S. (2019). *Human Compatible* - aligning objectives with human values. [Book information](https://humancompatible.ai/)



