# Section 22.2: Passive Reinforcement Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22.2  
> **Previous:** [Section 22.1 - RL Problem Setup](./section-01-rl-problem-setup.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Evaluating a Fixed Policy

[Section 22.1](./section-01-rl-problem-setup.md) posed the full RL problem: find an optimal policy. **Passive RL** is the simpler subproblem: **given a fixed policy $\pi$, estimate how good it is** - compute $V^\pi(s)$ or $Q^\pi(s,a)$ from experience without changing actions.

This mirrors **policy evaluation** in [Chapter 17](../chapter-17-making-complex-decisions/README.md). There, you iterate the Bellman expectation equation with known $P$. Here, transitions arrive as **samples** from the environment.

> **Readable form:** Passive RL answers: "If I always follow this strategy, what score do I expect from each state?" You are not trying to improve - only to measure.

Why study passive RL first? It introduces **temporal-difference (TD) learning** - the backbone of Q-learning, SARSA, and modern deep RL - without the complexity of exploration and policy improvement.

---

## The Bellman Expectation Equation (Review)

For a fixed policy $\pi$, the true value function satisfies:

$$
V^\pi(s) = \sum_a \pi(a \mid s) \sum_{s'} P(s' \mid s, a) \left[ R(s, a, s') + \gamma V^\pi(s') \right]
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Chapter 17 solves this by **dynamic programming (DP)**: sweep all states, use exact $P$. Passive RL uses **Monte Carlo (MC)** or **TD** updates from observed transitions $(s, a, r, s')$.

| Method | Uses bootstrapping? | Needs episodic termination? |
|--------|---------------------|----------------------------|
| Monte Carlo | No - full return $G_t$ | Yes (or truncated) |
| TD(0) | Yes - uses $V(s')$ | No |
| DP (Chapter 17) | Yes - uses model | No |

> **Readable form:** Monte Carlo waits until the episode ends and backs up the actual total reward. TD updates immediately using your current guess of the next state's value.

---

## Direct Utility Estimation (Monte Carlo)

**Direct utility estimation** treats each state's value as a separate supervised learning problem: accumulate returns $G_t$ every time state $s$ is visited, set $V(s)$ to the average.

After episode $k$, for each visit to $s$ at time $t$:

$$
V(s) \leftarrow \text{average of all sampled } G_t \text{ starting from } s
$$
> **Readable form:** The value expression summarizes expected discounted future reward from the current state or policy.

**Properties:**

- **Unbiased** estimate of $V^\pi(s)$ as visits $\to \infty$
- **High variance** - one long episode's noise affects the average
- **Slow** - must wait for episode end before updating
- Ignores **temporal structure** between consecutive states

```python
def mc_policy_evaluation(episodes, gamma=0.99):
    """episodes: list of [(s, r), (s, r), ...] ending states."""
    returns = {}  # s -> list of returns
    for ep in episodes:
        G, visited = 0.0, set()
        for t in reversed(range(len(ep))):
            s, r = ep[t]
            G = r + gamma * G
            if s not in visited:
                returns.setdefault(s, []).append(G)
                visited.add(s)
    return {s: sum(gs) / len(gs) for s, gs in returns.items()}
```

> **Readable form:** Walk backward through the episode, compute return from each state, average across episodes. First-visit MC counts each state once per episode.

---

## Adaptive Dynamic Programming (Passive)

If the agent **does** learn an empirical model $\hat{P}$ from transitions while following $\pi$, it can run **policy evaluation** DP on $\hat{P}$ - **adaptive dynamic programming**.

1. Collect transitions $(s, a, r, s')$ under $\pi$
2. Update counts $\hat{P}(s' \mid s, a)$
3. Solve $V^\pi$ via Bellman sweeps on $\hat{P}$

This bridges passive RL to [Section 22.7](./section-07-model-based-reinforcement-learning.md). With limited data, $\hat{P}$ is inaccurate and $V$ estimates suffer - but sample efficiency can beat pure MC when the model generalizes.

---

## Temporal-Difference Learning: TD(0)

**TD learning** combines DP bootstrapping with MC sampling. The **TD(0) update** for policy evaluation:

$$
V(s) \leftarrow V(s) + \alpha \left[ R_{t+1} + \gamma V(s') - V(s) \right]
$$
> **Readable form:** The value expression summarizes expected discounted future reward from the current state or policy.

Define the **TD error**:

$$
\delta_t = R_{t+1} + \gamma V(s') - V(s)
$$
> **Readable form:** New estimate = old estimate + learning rate × (one-step lookahead target − old estimate). The target uses the **actual** reward plus your **current** guess of the next state's value.

| Term | Name | Role |
|------|------|------|
| $R_{t+1} + \gamma V(s')$ | TD target | One-step bootstrapped return |
| $\delta_t$ | TD error | Prediction mistake |
| $\alpha$ | Learning rate | Step size |

**Why TD matters:** Updates happen **online** after every step. Credit propagates backward over successive steps without waiting for episode termination - essential for continuing tasks and long horizons.

---

## TD vs. Monte Carlo vs. DP

```
                    ┌─────────────────────────────────────┐
                    │         Dynamic Programming          │
                    │  (full model P, all states)          │
                    └─────────────────┬───────────────────┘
                                      │ bootstraps
                    ┌─────────────────▼───────────────────┐
                    │      Temporal-Difference Learning      │
                    │  (samples, bootstraps V(s'))            │
                    └─────────────────┬───────────────────┘
                                      │ samples
                    ┌─────────────────▼───────────────────┐
                    │        Monte Carlo                   │
                    │  (samples, full returns G_t)           │
                    └─────────────────────────────────────┘
```

| Criterion | MC | TD(0) | DP |
|-----------|-----|-------|-----|
| Model needed | No | No | Yes |
| Bootstrapping | No | Yes | Yes |
| Bias | Unbiased | Biased (until convergence) | Exact |
| Variance | High | Lower | N/A |
| Online update | No | Yes | Batch |

Sutton's **random walk** example (AIMA Figure 22.1) shows TD(0) converging faster than MC with $\alpha$ schedules.

---

## Convergence of TD(0)

For tabular $V$ with fixed policy $\pi$, TD(0) converges to $V^\pi$ under standard conditions:

1. Every state is visited infinitely often
2. Learning rates satisfy Robbins-Monro:

$$
\sum_t \alpha_t(s) = \infty, \quad \sum_t \alpha_t(s)^2 < \infty
$$
> **Readable form:** Learning rates must add up forever so learning can continue, but their squared sum must stay finite so updates eventually settle.

Example: $\alpha_t = 1/t$ works. Constant $\alpha$ gives **tracking** of non-stationary targets but not strict convergence.

> **Readable form:** Visit every state often enough, shrink the learning rate appropriately, and TD(0) homes in on the true values for your fixed policy.

This is **passive** - no exploration strategy beyond whatever $\pi$ already does.

---

## TD($\lambda$): Eligibility Traces

**TD($\lambda$)** unifies MC and TD by backing up along **eligibility traces** - credit assignment over multiple steps.

$$
V(s) \leftarrow V(s) + \alpha \, \delta_t \, e_t(s)
$$
> **Readable form:** The value expression summarizes expected discounted future reward from the current state or policy.

Traces $e_t(s)$ decay geometrically when states are not visited. $\lambda = 0$ recovers TD(0); $\lambda \to 1$ approaches MC.

$$
e_t(s) = \gamma \lambda \, e_{t-1}(s) + \mathbf{1}(S_t = s)
$$
> **Readable form:** Eligibility traces remember which states were recently responsible for predictions. When a reward arrives, all recently active states get partial credit.

Traces matter in continuous control and actor-critic methods ([Section 22.6](./section-06-policy-search-and-policy-gradients.md)).

---

## Worked Example: 3-State Chain

States $\{A, B, C\}$, $\gamma = 0.9$, fixed $\pi$: always move right. Rewards: $A \to B$: 0, $B \to C$: 0, $C$ terminal: +1.

True values (DP): $V^\pi(A) \approx 0.81$, $V^\pi(B) \approx 0.9$, $V^\pi(C) = 1$.

One transition $A \to B$ with reward 0, suppose $V(A)=0.5$, $V(B)=0.6$, $\alpha=0.1$:

$$
\delta = 0 + 0.9 \times 0.6 - 0.5 = 0.04
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

$$
V(A) \leftarrow 0.5 + 0.1 \times 0.04 = 0.504
$$
> **Readable form:** The value expression summarizes expected discounted future reward from the current state or policy.

Repeated experience propagates value leftward until estimates match $V^\pi$.

---

## Estimating $Q^\pi$ Passively

For each $(s, a)$ under $\pi$, store $Q(s,a)$ and apply TD:

$$
Q(s, a) \leftarrow Q(s, a) + \alpha \left[ R_{t+1} + \gamma Q(s', a') - Q(s, a) \right]
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

where $a' \sim \pi(\cdot \mid s')$. This is **SARSA** without policy change - the on-policy TD update for $Q$ ([Section 22.3](./section-03-active-reinforcement-learning.md)).

Passive $Q$-evaluation supports **policy comparison**: which of two fixed policies is better before committing to active improvement?

---

## Connection to Chapter 17

| Chapter 17 (MDPs) | Passive RL |
|------------------|------------|
| Policy evaluation: solve Bellman | Estimate $V^\pi$ from samples |
| Value iteration needs $\max_a$ | No $\max$ - policy fixed |
| Known $P(s' \mid s,a)$ | Unknown $P$; sample transitions |

[Chapter 17](../chapter-17-making-complex-decisions/README.md) Section 3 (value iteration) computes $V^*$ actively. Passive RL is the **sample-based analog of policy evaluation** - the inner loop of policy iteration.

---

## Connection to Course 1

Monte Carlo averaging resembles estimating mean accuracy on a [hold-out set](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-06-the-ml-workflow-and-data-hygiene.md) - but RL returns are **correlated** within trajectories, not i.i.d. Bootstrapping in TD is unlike standard supervised epochs; it propagates value estimates across time ([Course 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-02-machine-learning-vs-ai-vs-deep-learning.md) contrasts RL with supervised ML).

---

## Python: TD(0) Policy Evaluation

```python
def td0_eval(env, policy, episodes=500, alpha=0.1, gamma=0.99):
    V = {}
    for _ in range(episodes):
        s = env.reset()
        done = False
        while not done:
            a = policy(s)
            s_next, r, done, _ = env.step(a)
            v = V.get(s, 0.0)
            v_next = V.get(s_next, 0.0) if not done else 0.0
            V[s] = v + alpha * (r + gamma * v_next - v)
            s = s_next
    return V
```

---

## Common Mistakes

**Using MC on continuing tasks without truncation.** Returns may not be well-defined.

**Constant $\alpha$ expecting convergence.** Works in practice for tracking; theory needs decaying $\alpha$.

**Forgetting policy is fixed.** TD error uses $V(s')$ under the **same** $\pi$; no $\max$ over actions.

**Confusing passive evaluation with Q-learning.** Q-learning is **active** and **off-policy** ([Section 22.3](./section-03-active-reinforcement-learning.md)).

---

## Reflection Questions

1. Why does TD(0) usually learn faster than Monte Carlo on long episodic tasks?
2. What assumption does adaptive DP make that pure TD(0) does not?
3. How is TD(0) policy evaluation related to the inner loop of policy iteration in Chapter 17?
4. When would high-variance MC returns still be preferable to TD?
5. What changes when you switch from estimating $V^\pi$ to $Q^\pi$ passively?

---

**Previous:** [Section 22.1 - RL Problem Setup](./section-01-rl-problem-setup.md) · **Next:** [Section 22.3 - Active RL](./section-03-active-reinforcement-learning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), §22.2. [https://aima.cs.berkeley.edu/](https://aima.cs.berkeley.edu/)
2. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.), Ch. 6. [http://incompleteideas.net/book/](http://incompleteideas.net/book/)
3. Watkins, C. J. C. H. (1989). *Models of Delayed Reinforcement Learning*. PhD thesis - Q-learning foundations.
4. Tsitsiklis, J. N., & Van Roy, B. (1997). "An analysis of temporal-difference learning with function approximation." *IEEE TAC*.
5. AIMA Python: `passive_adp.py`, `passive_td.py` in [aima-python](https://github.com/aimacode/aima-python)
