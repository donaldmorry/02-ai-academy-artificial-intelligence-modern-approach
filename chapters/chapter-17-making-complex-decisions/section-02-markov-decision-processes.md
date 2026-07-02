# Section 17.2: Markov Decision Processes

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17.2  
> **Previous:** [Section 17.1 - Sequential Decisions](./section-01-sequential-decisions.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Definition of an MDP

A **Markov decision process** is a tuple $\langle S, A, T, R, \gamma \rangle$:

| Symbol | Meaning |
|--------|---------|
| $S$ | Set of states |
| $A$ | Set of actions (may be $A(s)$ state-dependent) |
| $T$ | Transition model $P(s' \mid s, a)$ |
| $R$ | Reward function $R(s, a, s')$ or $R(s)$ |
| $\gamma$ | Discount factor $[0, 1]$ |

> **Readable form:** An MDP is a mathematical model of "situations you can be in, moves you can make, how the world randomly responds, and what payoff you get." The Markov property says the future depends only on the current state, not the full past.

---

## The Markov Property

$$
P(s_{t+1} \mid s_t, a_t, s_{t-1}, \ldots) = P(s_{t+1} \mid s_t, a_t)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**State** must summarize all decision-relevant history. If not, **augment state** (e.g., include last observation in POMDP - [Section 17.5](./section-05-partially-observable-mdps.md)).

---

## Transition Model

For each $(s, a)$, $P(\cdot \mid s, a)$ is a probability distribution over $S$:

$$
\sum_{s' \in S} P(s' \mid s, a) = 1
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**Deterministic** special case: $s' = f(s,a)$ with probability 1.

**Example (4×3 grid world):**

- States: cells + terminal
- Actions: {N, S, E, W}
- Slippage: intended direction 0.8, perpendicular 0.1 each
- Walls: bump → stay in place

---

## Reward Function

Common forms:

| Form | Meaning |
|------|---------|
| $R(s)$ | Reward depends on arrival state |
| $R(s, a)$ | Includes action cost |
| $R(s, a, s')$ | Most general |

**Expected immediate reward:**

$$
R(s, a) = \sum_{s'} P(s' \mid s, a) \, R(s, a, s')
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Small **living penalty** (e.g., $-0.04$ per step) encourages shorter paths in grid worlds.

---

## Grid World Example (AIMA)

4×3 grid, start (1,1), goal +1 at (4,3), pit −1 at (4,2), γ = 0.9.

```
.  .  .  +1
.  X  .  -1
S  .  .  .
```

Actions move with stochastic slip. Optimal policy (from value iteration) often goes **around** the pit even though shortest path is risky - risk encoded via expected reward and discount.

---

## Solution Objects

| Object | Definition |
|--------|------------|
| **Policy** $\pi$ | $\pi(a \mid s)$ or $\pi(s)$ |
| **State-value** $V^\pi(s)$ | Expected return from $s$ following $\pi$ |
| **Action-value** $Q^\pi(s,a)$ | Expected return from $s$, take $a$, then $\pi$ |
| **Optimal** $\pi^*, V^*, Q^*$ | Maximize expected return |

Relationship:

$$
V^\pi(s) = \sum_a \pi(a \mid s) Q^\pi(s, a)
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

$$
Q^\pi(s, a) = R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) V^\pi(s')
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

---

## Bellman Equation for $V^\pi$

V^\pi(s) = \sum_a \pi(a \mid s) \left[ R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) V^\pi(s') \right]
$$

**Recursive structure:** value today = expected reward + discounted value tomorrow.

$$
> **Readable form:** The value of being in state s equals what you expect to earn right away plus γ times the average value of where you land next.

---

## Optimal Bellman Equation

$$
V^*(s) = \max_a \left[ R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) V^*(s') \right]
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

$$
\pi^*(s) = \arg\max_a \left[ R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) V^*(s') \right]
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Same for $Q^*$:

$$
Q^*(s,a) = R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) \max_{a'} Q^*(s', a')
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

These are foundation for [Section 17.3](./section-03-value-iteration.md) and [17.4](./section-04-policy-iteration.md).

---

## MDP vs. Markov Chain

**Markov chain:** no actions, fixed transition matrix.

**MDP:** agent chooses actions affecting transitions - **controlled** stochastic process.

---

## Formulating Real Problems as MDPs

| Application | States | Actions | Reward |
|-------------|--------|---------|--------|
| Inventory | Stock level | Order quantity | Sales − holding cost |
| Maintenance | Machine condition | Repair/wait | Production − failure cost |
| Routing | Location | Next hop | −travel time |
| HVAC control | Temp, occupancy | Heat/cool | Comfort − energy |

**State design** is critical - too coarse loses Markov property; too fine explodes $|S|$.

---

## Python: MDP Data Structure

```python
from collections import defaultdict

class MDP:
    def __init__(self, states, actions, transitions, rewards, gamma):
        self.states = states
        self.actions = actions  # or dict state -> list
        self.T = transitions    # T[s][a][s'] = P(s'|s,a)
        self.R = rewards        # R[s][a] expected immediate
        self.gamma = gamma

    def expected_reward(self, s, a):
        return self.R[s][a]

    def next_dist(self, s, a):
        return self.T[s][a]
```

---

## Complexity Overview

|Solving method | Time (tabular) |
|--------------|----------------|
| Value iteration | $O(|S|^2 |A|)$ per iteration |
| Policy iteration | Fewer iterations, expensive evaluation |

$|S|$ and $|A|$ grow exponentially in problem features - **function approximation** needed at scale ([Section 17.8](./section-08-from-mdps-to-reinforcement-learning.md)).

---

## Connection to HMMs

[Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md) **HMMs** model state evolution without control. MDP adds **actions** influencing transitions - **controlled HMM** or **POMDP** when state hidden.

---

## Common Mistakes

**Transition probabilities don't sum to 1** for some $(s,a)$.

**Terminal states** still have outgoing transitions - should be absorbing with 0 reward.

**Reward sign confusion** - costs as negative rewards vs. separate cost function.

**Non-Markov state** - using partial observations as state without belief update.

---

## Reflection Questions

1. Write the Bellman equation for $Q^\pi(s,a)$.
2. For deterministic transitions, what does $P(s' \mid s,a)$ look like?
3. How does a living penalty change optimal paths in grid world?
4. Formulate a two-state MDP (working/broken machine) with repair action.
5. Why is $|S|$ often the limiting factor in MDP solvers?

---

**Previous:** [Section 17.1 - Sequential Decisions](./section-01-sequential-decisions.md)  
**Next:** [Section 17.3 - Value Iteration](./section-03-value-iteration.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 17.2.
2. Puterman, M. L. (1994). *Markov Decision Processes*.
3. Sutton & Barto (2018). *RL*, Chapter 3 - MDP formalism.
4. Bellman, R. (1957). Dynamic Programming - origin of Bellman equations.
5. aima-python: `GridMDP`, `sequential_decision_environment.py`.


