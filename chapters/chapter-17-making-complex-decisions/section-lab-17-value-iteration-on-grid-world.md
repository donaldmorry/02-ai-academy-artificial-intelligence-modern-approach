# Lab 17: Value Iteration on Grid World

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17  
> **Prerequisites:** Sections 17.1-17.4  
> **Estimated time:** 4-6 hours  
> **Tools:** Python 3.10+, optional NumPy  
> **Assessment:** [Shared Assessment Appendix](../../ASSESSMENT_APPENDIX.md)

---

## Lab Objective

Solve a small Markov decision process with value iteration. You will turn the Bellman optimality equation into code, watch values converge, and extract a policy.

By the end, you should be able to:

1. Represent an MDP with states, actions, transitions, rewards, and discount factor.
2. Implement Bellman backups.
3. Track convergence with a maximum value change.
4. Extract a greedy policy from the final value function.
5. Explain how living rewards and discounting change behavior.

---

## Bellman Backup

$$
V_{k+1}(s) = \max_a \sum_{s'} P(s' \mid s,a)\left[R(s,a,s') + \gamma V_k(s')\right]
$$
> **Readable form:** the next value of a state is the best expected immediate reward plus discounted future value

---

## Part A: Grid Definition

Use a 3x4 grid with two terminal states:

```python
WIDTH, HEIGHT = 4, 3
WALLS = {(1, 1)}
TERMINALS = {(3, 2): 1.0, (3, 1): -1.0}
ACTIONS = ["U", "D", "L", "R"]


def states():
    for y in range(HEIGHT):
        for x in range(WIDTH):
            if (x, y) not in WALLS:
                yield (x, y)
```

Coordinates use `(x, y)` with `(0, 0)` in the lower-left if you prefer plotting, or just treat them as labels.

---

## Part B: Transition Model

Start deterministic:

```python
def move(s, a):
    x, y = s
    dx, dy = {
        "U": (0, 1), "D": (0, -1),
        "L": (-1, 0), "R": (1, 0),
    }[a]
    nxt = (x + dx, y + dy)
    if not (0 <= nxt[0] < WIDTH and 0 <= nxt[1] < HEIGHT) or nxt in WALLS:
        return s
    return nxt


def transitions(s, a):
    return [(move(s, a), 1.0)]
```

Optional stochastic extension: intended direction has probability 0.8, left/right slips share probability 0.2.

---

## Part C: Value Iteration

```python
def reward(sp, living_reward=-0.04):
    return TERMINALS.get(sp, living_reward)


def value_iteration(gamma=0.95, living_reward=-0.04, theta=1e-6):
    S = list(states())
    V = {s: 0.0 for s in S}

    while True:
        delta = 0.0
        new_V = V.copy()
        for s in S:
            if s in TERMINALS:
                new_V[s] = TERMINALS[s]
                continue
            q_values = []
            for a in ACTIONS:
                q = 0.0
                for sp, p in transitions(s, a):
                    r = reward(sp, living_reward)
                    q += p * (r + gamma * V[sp])
                q_values.append(q)
            new_V[s] = max(q_values)
            delta = max(delta, abs(new_V[s] - V[s]))
        V = new_V
        if delta < theta:
            return V
```

---

## Part D: Policy Extraction

```python
def best_action(s, V, gamma=0.95, living_reward=-0.04):
    if s in TERMINALS:
        return "T"
    scores = {}
    for a in ACTIONS:
        scores[a] = sum(
            p * (reward(sp, living_reward) + gamma * V[sp])
            for sp, p in transitions(s, a)
        )
    return max(scores, key=scores.get)
```

Print a grid of arrows and terminal rewards.

---

## Experiments

Run three settings:

| Setting | Gamma | Living reward | Prediction |
|---------|-------|---------------|------------|
| Baseline | 0.95 | -0.04 | Seeks +1 while avoiding -1 |
| Impatient | 0.50 | -0.04 | Shorter horizon |
| Painful living | 0.95 | -0.20 | Faster termination |

Explain how the policy changes.

---

## Deliverables

- Value iteration implementation.
- Final value table and policy table.
- Convergence log: iterations and final `delta`.
- One paragraph explaining the effect of discount factor.
- Optional stochastic transition comparison.

---

## Reflection

1. Why does value iteration converge for discounted MDPs?
2. What is the difference between value iteration and policy iteration?
3. How does living reward shape risk-taking?
4. Why is an MDP model required here but not in Q-learning?

---

## References

- Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17.
- [Chapter 17 README](./README.md)
