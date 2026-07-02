# Lab 22: Tabular Q-Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22  
> **Prerequisites:** Sections 22.1-22.3 and Chapter 17  
> **Estimated time:** 4-6 hours  
> **Tools:** Python 3.10+, optional NumPy  
> **Assessment:** [Shared Assessment Appendix](../../ASSESSMENT_APPENDIX.md)

---

## Lab Objective

Train a tabular Q-learning agent on a small grid world. This lab contrasts model-free learning with the value-iteration lab: the agent does not know transition probabilities in advance and must learn from experience.

By the end, you should be able to:

1. Implement an epsilon-greedy exploration policy.
2. Apply the Q-learning update.
3. Track return per episode.
4. Extract a greedy policy from learned Q-values.
5. Diagnose reward hacking and exploration failures.

---

## Q-Learning Update

$$
Q(s,a) \leftarrow Q(s,a) + \alpha\left[r + \gamma \max_{a'} Q(s',a') - Q(s,a)\right]
$$
> **Readable form:** move the current action value toward observed reward plus the best estimated future value

---

## Part A: Environment

Use a deterministic grid first:

```python
ACTIONS = ["U", "D", "L", "R"]
START = (0, 0)
GOAL = (3, 2)
TRAP = (3, 1)
WALLS = {(1, 1)}
WIDTH, HEIGHT = 4, 3


def step(state, action):
    if state in {GOAL, TRAP}:
        return START, 0.0, True
    x, y = state
    dx, dy = {
        "U": (0, 1), "D": (0, -1),
        "L": (-1, 0), "R": (1, 0),
    }[action]
    nxt = (x + dx, y + dy)
    if not (0 <= nxt[0] < WIDTH and 0 <= nxt[1] < HEIGHT) or nxt in WALLS:
        nxt = state
    if nxt == GOAL:
        return nxt, 1.0, True
    if nxt == TRAP:
        return nxt, -1.0, True
    return nxt, -0.04, False
```

---

## Part B: Agent

```python
import random
from collections import defaultdict


Q = defaultdict(float)


def epsilon_greedy(state, epsilon):
    if random.random() < epsilon:
        return random.choice(ACTIONS)
    return max(ACTIONS, key=lambda a: Q[(state, a)])


def update_q(s, a, r, sp, alpha=0.2, gamma=0.95):
    best_next = max(Q[(sp, ap)] for ap in ACTIONS)
    target = r + gamma * best_next
    Q[(s, a)] += alpha * (target - Q[(s, a)])
```

---

## Part C: Training Loop

```python
def run_episode(epsilon=0.2, max_steps=100):
    state = START
    total = 0.0
    for _ in range(max_steps):
        action = epsilon_greedy(state, epsilon)
        nxt, reward, done = step(state, action)
        update_q(state, action, reward, nxt)
        total += reward
        state = nxt
        if done:
            break
    return total


returns = [run_episode(epsilon=0.2) for _ in range(500)]
print(sum(returns[-50:]) / 50)
```

---

## Part D: Experiments

Compare:

| Run | Epsilon | Alpha | Expected behavior |
|-----|---------|-------|-------------------|
| Low exploration | 0.01 | 0.2 | May get stuck with poor policy |
| Baseline | 0.20 | 0.2 | Learns reliable path |
| High exploration | 0.60 | 0.2 | Learns but returns stay noisy |
| High learning rate | 0.20 | 0.8 | Unstable estimates |

Plot or print a moving average of returns over episodes.

---

## Part E: Safety Check

Change the trap penalty from `-1.0` to `-0.05`. Does the learned policy become more willing to pass near or through the trap? This is a tiny reward-design example: the agent does what the reward says, not what you meant.

---

## Deliverables

- Q-learning implementation.
- Return curve or moving-average table.
- Final greedy policy grid.
- Short comparison with Lab 17 value iteration.
- One reward-design failure case and mitigation.

---

## Reflection

1. Why is Q-learning called off-policy?
2. What does epsilon control?
3. Why can sparse rewards slow learning?
4. How does reward hacking appear in this tiny environment?

---

## References

- Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22.
- [Chapter 22 README](./README.md)
