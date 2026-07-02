# Section 17.1: Sequential Decisions

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17.1  
> **Previous:** [Chapter 16 - Making Simple Decisions](../chapter-16-making-simple-decisions/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Beyond One-Shot Decisions

[Chapter 16](../chapter-16-making-simple-decisions/README.md) treated **single-stage** decisions: observe evidence, choose an action, receive utility. Many AI problems are **sequential**: the agent acts repeatedly, the world evolves, and rewards accumulate over time.

Examples:

- Robot navigation through a building
- Inventory restocking each week
- Game playing (chess moves)
- Dialogue systems (turn-by-turn)

> **Readable form:** Instead of one big choice, you make a chain of choices. Each choice changes what happens next and what payoffs you collect along the way.

---

## Policies vs. Actions

A single **action** is what you do now. A **policy** $\pi$ is a complete plan:

$$
\pi(s) = a \quad \text{or} \quad \pi(a \mid s) = P(a \mid s)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

| Type | Notation | Use |
|------|----------|-----|
| **Deterministic policy** | $\pi(s) = a$ | Grid world, tabular RL |
| **Stochastic policy** | $\pi(a \mid s)$ | Partial observability, exploration |

**Optimal policy** $\pi^*$ maximizes expected cumulative reward over the horizon.

---

## Rewards and Utilities Over Time

At each time step $t$, the agent receives **reward** $R_t$ (often written $R(s_t, a_t, s_{t+1})$).

**Return** (cumulative reward) for finite horizon $T$:

$$
G_t = \sum_{k=0}^{T-t} R_{t+k}
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Infinite horizon with discount** $\gamma \in [0, 1)$:

$$
G_t = \sum_{k=0}^{\infty} \gamma^k R_{t+k}
$$
> **Readable form:** Add up all future rewards, but count distant rewards for less using discount factor γ. γ near 1 = patient; γ near 0 = greedy for immediate payoff.

**Why discount?**

- Economic: future money worth less
- Mathematical: guarantees convergent sums for bounded rewards
- Modeling: agent may not survive or environment may change

---

## Horizons

| Horizon | Planning | Example |
|---------|----------|---------|
| **Finite** $T$ | Backward induction | 10-step game |
| **Infinite** | Stationary policies, $\gamma < 1$ | Maintenance, RL |
| **Indefinite** | First exit to terminal | Reach goal state |

**Terminal states** absorb: once entered, no further actions or rewards (except possibly terminal reward).

---

## Sequential Decision Diagram

```
s₀ →[a₀]→ s₁ →[a₁]→ s₂ → ... → s_T
      R₀      R₁
```

The agent chooses $a_t$ based on **history** $h_t = (s_0, a_0, s_1, \ldots, s_t)$.

**Markov assumption** (next section): optimal action depends only on current $s_t$, not full history - enormous simplification.

---

## Expected Utility Over Trajectories

For policy $\pi$:

$$
\mathbb{E}[G_0 \mid \pi] = \mathbb{E}\left[\sum_{t=0}^{\infty} \gamma^t R_t \;\middle|\; \pi\right]
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Value of policy** $V^\pi(s)$ = expected return starting in $s$ following $\pi$ ([Section 17.3](./section-03-value-iteration.md)).

---

## Worked Example: Three-Step Grades

Student chooses effort {low, high} each day. States summarize "knowledge level" {0,1,2}. High effort costs −1 reward; level increases prob. 0.8; low effort cost 0, increase prob. 0.3. Terminal reward at level 2 = +10.

Finite horizon 3 days, no discount.

Enumerate policies is small; MEU over trajectories picks high effort when behind - preview of **dynamic programming**.

---

## Comparison to Chapter 16

| Chapter 16 | Chapter 17 |
|-----------|-----------|
| Decision networks | MDP / POMDP |
| Utility at outcome | Reward per step |
| One or few decisions | Many time steps |
| VPI for tests | Exploration in bandits |

Utility from Ch. 16 becomes **reward function** in Ch. 17.

---

## Stochastic vs. Deterministic Transitions

If actions have deterministic effects, sequential problem reduces to **search in state space** ([Chapter 03](../chapter-03-solving-problems-by-searching/README.md)).

**Stochastic transitions** require **expectation** over next states - core of MDPs ([Section 17.2](./section-02-markov-decision-processes.md)).

---

## Offline vs. Online Planning

| Mode | Know model? | Examples |
|------|-------------|----------|
| **Offline** | Yes - compute $\pi^*$ before acting | Value iteration |
| **Online** | Partial - replan each step | RTDP, MCTS |
| **Model-free** | No - learn from experience | Q-learning ([Chapter 22](../chapter-22-reinforcement-learning/README.md)) |

This chapter focuses on **known-model** MDPs; [Section 17.8](./section-08-from-mdps-to-reinforcement-learning.md) previews learning.

---

## Python: Rollout Simulation

```python
import random

def rollout(env, policy, gamma=0.99, max_steps=1000):
    s = env.reset()
    G, t = 0.0, 0
    while t < max_steps and not env.is_terminal(s):
        a = policy(s)
        s, r = env.step(s, a)  # stochastic transition inside
        G += (gamma ** t) * r
        t += 1
    return G

def evaluate_policy(env, policy, n_episodes=1000, gamma=0.99):
    return sum(rollout(env, policy, gamma) for _ in range(n_episodes)) / n_episodes
```

> **Readable form:** Simulate many episodes following policy π; average total discounted reward to estimate how good π is.

---

## Rational Sequential Agent

From [Chapter 02](../chapter-02-intelligent-agents/README.md): a **sequential rational agent** maximizes expected discounted cumulative reward given its model of the environment.

Performance measure = $\mathbb{E}[G_0 \mid \pi^*, \text{model}]$.

---

## AIMA Grid World Preview

Russell & Norvig's $4 \times 3$ stochastic grid (Figure 17.1) is the running example for Chapter 17. The agent starts near the bottom-left, receives small negative step rewards $r$ (e.g., $-0.04$), and aims for $+1$ at $(4,3)$ while avoiding $-1$ at $(4,2)$. Movement is stochastic: intended direction succeeds with probability 0.8; the agent slips perpendicular with probability 0.1 each.

Optimal policies change qualitatively as $r$ varies - when life is painful enough ($r \ll 0$), the agent races for the nearest exit; when $r > 0$, it may avoid terminals entirely. This shows how **reward scale** interacts with **risk** before we formalize MDPs in [Section 17.2](./section-02-markov-decision-processes.md).

> **Readable form:** Tiny changes to per-step penalty can flip "play it safe" into "rush for the goal" - reward design is not cosmetic.

---

## Reward Scales and Invariance

Multiplying all rewards by a positive constant $c > 0$ does not change the optimal policy if $\gamma$ is fixed. Adding a constant to all rewards *can* change decisions when $\gamma < 1$ because future additive offsets are discounted differently.

Bounded rewards with $\gamma < 1$ guarantee finite optimal values:

$$
|V^*(s)| \leq \frac{R_{\max}}{1 - \gamma}
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

---

## Common Mistakes

**Confusing reward with state.** Reward is signal, not world description.

**γ = 1** on infinite horizon with positive rewards everywhere → infinite value (ill-defined).

**Using wrong horizon** for episodic vs. continuing tasks.

**Policy depending on irrelevant history** when Markov state is sufficient.

---

## Reflection Questions

1. Why does discounting ensure bounded returns for bounded rewards?
2. Give an example where a stochastic policy outperforms any deterministic policy.
3. How is a finite-horizon decision tree related to a sequential decision problem?
4. What is the difference between return $G_t$ and value $V^\pi(s)$?
5. When would offline planning fail in a changing environment?

---

**Next:** [Section 17.2 - MDPs](./section-02-markov-decision-processes.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 17.1.
2. Bellman, R. (1957). *Dynamic Programming*.
3. Puterman, M. L. (1994). *Markov Decision Processes*.
4. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning* (2nd ed.), Ch. 3.
5. aima-python: `mdp.py` grid worlds.



