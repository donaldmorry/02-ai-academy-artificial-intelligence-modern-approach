# Section 22.3: Active Reinforcement Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22.3  
> **Previous:** [Section 22.2 - Passive RL](./section-02-passive-reinforcement-learning.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Evaluation to Improvement

[Passive RL](./section-02-passive-reinforcement-learning.md) estimates $V^\pi$ for a **fixed** policy. **Active RL** seeks an **optimal** policy $\pi^*$ that maximizes expected return - the model-free counterpart to **value iteration** and **policy iteration** in [Chapter 17](../chapter-17-making-complex-decisions/README.md).

The agent must solve two coupled problems:

1. **Exploitation** - act according to current best estimates of $Q$ or $V$
2. **Exploration** - try suboptimal actions to discover better strategies

> **Readable form:** Active RL is learning to play chess by playing games, not just watching someone else's games. You must try risky moves sometimes or you never learn better than your opening habit.

Without exploration, the agent may converge to a **locally optimal** policy while missing globally better behavior - the exploration-exploitation tradeoff also appears in multi-armed bandits ([Chapter 17](../chapter-17-making-complex-decisions/README.md), Section 6).

---

## Generalized Policy Iteration

Active RL often alternates two phases:

```
┌──────────────────┐         ┌──────────────────┐
│ Policy evaluation │ ──────► │ Policy improvement │
│  (estimate Q^π)   │         │  π ← greedy(Q)     │
└────────▲─────────┘         └─────────┬──────────┘
         └─────────────────────────────┘
              Generalized Policy Iteration (GPI)
```

**Policy improvement** for Q-functions:

$$
\pi'(s) = \arg\max_a Q(s, a)
$$
> **Readable form:** The action-value rule compares actions by immediate reward plus discounted future value.

Chapter 17 performs this with exact Bellman backups. Active methods use **sampled** backups - Q-learning and SARSA.

---

## The Q-Learning Update Rule

**Q-learning** (Watkins, 1989) is an **off-policy** TD control algorithm. It learns $Q^*$ directly while following any **behavior policy** that explores (e.g., $\varepsilon$-greedy).

After observing transition $(s, a, r, s')$:

$$
Q(s, a) \leftarrow Q(s, a) + \alpha \left[ r + \gamma \max_{a'} Q(s', a') - Q(s, a) \right]
$$
> **Readable form:** The action-value rule compares actions by immediate reward plus discounted future value.

Define the **Q-learning target**:

$$
y = r + \gamma \max_{a'} Q(s', a')
$$
> **Readable form:** The target value is the immediate reward plus the discounted value of the best next action.

The update is $Q(s,a) \leftarrow Q(s,a) + \alpha (y - Q(s,a))$.

> **Readable form:** After taking action $a$ in state $s$, move your Q-estimate toward the reward you got plus the best Q-value you believe any action has in the next state - even if you would not actually take that best action while exploring.

This uses the **Bellman optimality** backup:

$$
Q^*(s, a) = \mathbb{E}\left[ R + \gamma \max_{a'} Q^*(S', a') \mid s, a \right]
$$
> **Readable form:** The action-value rule compares actions by immediate reward plus discounted future value.

---

## Why Q-Learning Is Off-Policy

| Aspect | Q-learning | SARSA (below) |
|--------|------------|---------------|
| Bootstrap target | $\max_{a'} Q(s', a')$ | $Q(s', a')$ where $a'$ is **actually taken** |
| Policy learned | Greedy w.r.t. $Q$ (optimal) | Policy that generated data |
| Behavior policy | Can differ (explore) | Same as target |

**Off-policy** means the update learns about the **greedy optimal** policy while the agent follows an exploratory policy (e.g., random 10% of the time).

**Advantages:**

- Reuses data from any behavior - old trajectories, human demos, replay buffers ([Section 22.5](./section-05-deep-reinforcement-learning.md))
- Separates exploration from the value being learned

**Risks:**

- Overestimation from $\max$ operator (addressed by Double DQN)
- Deadly triad with function approximation ([Section 22.4](./section-04-generalization-in-reinforcement-learning.md))

```python
def q_learning_step(Q, s, a, r, s_next, alpha, gamma, done):
    best_next = 0.0 if done else max(Q.get((s_next, ap), 0.0) for ap in ACTIONS)
    target = r + gamma * best_next
    old = Q.get((s, a), 0.0)
    Q[(s, a)] = old + alpha * (target - old)
```

---

## Q-Learning Convergence Conditions

Watkins and Dayan (1992) established convergence of tabular Q-learning to $Q^*$ under:

1. **Tabular representation** - each $(s,a)$ pair has independent $Q(s,a)$
2. **Bounded rewards** - $|r| \leq R_{\max}$
3. **Learning rate conditions** - for each $(s,a)$:

$$
\sum_t \alpha_t(s,a) = \infty, \quad \sum_t \alpha_t(s,a)^2 < \infty
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

4. **Greedy in the limit with infinite exploration (GLIE):** every action in every state is taken infinitely often, and exploration decays so the policy becomes greedy:

\pi(a \mid s) \to \mathbf{1}[a = \arg\max_{a'} Q(s,a')] \quad \text{as } t \to \infty
$$

Example GLIE scheme: $\varepsilon_t$-greedy with $\varepsilon_t = 1/t$.

$$
> **Readable form:** Visit every state-action pair endlessly, shrink learning rates appropriately, and eventually exploit more than explore - then tabular Q-learning finds the true optimal Q-function.

**When convergence fails:**

- **Function approximation** - no general guarantee; can diverge
- **Non-stationary targets** - deep Q-networks use tricks ([Section 22.5](./section-05-deep-reinforcement-learning.md))
- **Insufficient exploration** - never visits optimal path

---

## SARSA: On-Policy TD Control

**SARSA** (State-Action-Reward-State-Action) updates using the **action actually taken** at $s'$:

Q(s, a) \leftarrow Q(s, a) + \alpha \left[ r + \gamma Q(s', a') - Q(s, a) \right]
$$

where $a' \sim \pi(\cdot \mid s')$ is the **next** action under the current policy.

$$
> **Readable form:** SARSA asks: "Given I will actually do $a'$ next (including random exploration), how good is $(s,a)$?" Q-learning asks: "How good is $(s,a)$ if I were optimal at $s'$?"

SARSA is **on-policy** - it learns $Q^\pi$ for the policy $\pi$ being followed, not necessarily $Q^*$.

---

## Q-Learning vs. SARSA: Cliff Walking

AIMA's **cliff walking** grid illustrates the difference:

- Agent starts left, goal bottom-right
- **Cliff** along bottom row: large negative reward if entered
- Optimal path: along top edge (longer but safe)

| Algorithm | Learned behavior | Why |
|-----------|------------------|-----|
| Q-learning | Near cliff edge (risky shortest path) | Learns $Q^*$ assuming optimal future |
| SARSA | Safer path away from cliff | Accounts for $\varepsilon$-greedy slips near cliff |

Under $\varepsilon$-greedy exploration, Q-learning's target assumes no random fall off the cliff; SARSA's target includes that risk.

> **Readable form:** Q-learning is optimistic about your future behavior. SARSA is cautious about the mistakes your current sloppy policy will make.

**Rule of thumb:**

- Want **optimal** policy after learning with separate deployment policy → Q-learning
- Want **safe** learning under the **same** exploratory policy → SARSA

---

## Exploration Strategies

### $\varepsilon$-Greedy

With probability $\varepsilon$, random action; else $\arg\max_a Q(s,a)$.

$$
\pi(a \mid s) = \begin{cases} 1 - \varepsilon + \varepsilon/|\mathcal{A}| & a = \arg\max_{a'} Q(s,a') \\ \varepsilon/|\mathcal{A}| & \text{otherwise} \end{cases}
$$
> **Readable form:** The action-value rule compares actions by immediate reward plus discounted future value.

Decay $\varepsilon$ over time for GLIE.

### Softmax (Boltzmann)

$$
\pi(a \mid s) = \frac{e^{Q(s,a)/\tau}}{\sum_{a'} e^{Q(s,a')/\tau}}
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

Temperature $\tau$: high = explore; low = exploit.

### Optimistic Initialization

Initialize $Q(s,a)$ high so untried actions look attractive - encourages systematic exploration in tabular settings. **UCB** from bandits ([Chapter 17](../chapter-17-making-complex-decisions/README.md)) extends this idea with confidence bounds.

---

## Tabular Q-Learning and SARSA Loops

```python
import random

ACTIONS = [0, 1, 2, 3]  # example

def epsilon_greedy(Q, s, eps):
    if random.random() < eps:
        return random.choice(ACTIONS)
    return max(ACTIONS, key=lambda a: Q.get((s, a), 0.0))

def train_q_learning(env, episodes=2000, alpha=0.1, gamma=0.99, eps=0.1):
    Q = {}
    for _ in range(episodes):
        s, done = env.reset(), False
        while not done:
            a = epsilon_greedy(Q, s, eps)
            s_next, r, done, _ = env.step(a)
            q_learning_step(Q, s, a, r, s_next, alpha, gamma, done)
            s = s_next
    return Q

def train_sarsa(env, episodes=2000, alpha=0.1, gamma=0.99, eps=0.1):
    Q = {}
    for _ in range(episodes):
        s = env.reset()
        a = epsilon_greedy(Q, s, eps)
        done = False
        while not done:
            s_next, r, done, _ = env.step(a)
            a_next = epsilon_greedy(Q, s_next, eps) if not done else None
            target = r if done else r + gamma * Q.get((s_next, a_next), 0.0)
            old = Q.get((s, a), 0.0)
            Q[(s, a)] = old + alpha * (target - old)
            s, a = s_next, a_next
    return Q
```

---

## Connection to Chapter 17 Bellman Optimality

Chapter 17 value iteration:

$$
V_{k+1}(s) = \max_a \sum_{s'} P(s' \mid s,a) [R + \gamma V_k(s')]
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Q-learning is the **sampled, model-free** version - one transition replaces the expectation over $s'$:

$$
Q(s,a) \leftarrow Q(s,a) + \alpha [R + \gamma \max_{a'} Q(s',a') - Q(s,a)]
$$
> **Readable form:** The action-value rule compares actions by immediate reward plus discounted future value.

[Chapter 17](../chapter-17-making-complex-decisions/README.md) guarantees convergence to $V^*$ with exact backups. Q-learning converges to $Q^*$ with tabular GLIE - same optimum, different mechanism.

---

## Comparison Table

| Algorithm | On/off policy | Target | Converges to |
|-----------|---------------|--------|--------------|
| Q-learning | Off | $r + \gamma \max_{a'} Q(s',a')$ | $Q^*$ (tabular, GLIE) |
| SARSA | On | $r + \gamma Q(s',a')$ | $Q^\pi$ |
| Expected SARSA | On | $r + \gamma \mathbb{E}_\pi[Q(s',\cdot)]$ | $Q^\pi$ |
| Expected SARSA | On | $r + \gamma \mathbb{E}_\pi[Q(s',\cdot)]$ | $Q^\pi$ |
| Double Q-learning | Off | Two tables decouple $\max$ select/evaluate | $Q^*$ (reduced bias) |

**Expected SARSA** lowers variance vs. SARSA; **Double Q-learning** is the tabular precursor to Double DQN ([Section 22.5](./section-05-deep-reinforcement-learning.md)). See [Course 1 ML paradigms](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-02-machine-learning-vs-ai-vs-deep-learning.md) and [Course 3](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) for scaling Q-learning to neural nets.

---

## Common Mistakes

**No exploration.** $\arg\max Q$ stuck at initial suboptimal actions.

**$\varepsilon$ too high forever.** Never exploits; noisy policy at deployment.

**Confusing Q-learning target with SARSA.** The $\max$ vs. actual $a'$ distinction is the core difference.

**Assuming deep Q-learning inherits tabular convergence proofs.** It does not - requires engineering stabilizers.

**Using Q-learning in risky domains during training.** SARSA may be safer on-policy.

---

## Reflection Questions

1. Write the Q-learning update and identify which term makes it off-policy.
2. Under $\varepsilon$-greedy exploration near a cliff, why does SARSA learn a safer path than Q-learning?
3. State the GLIE conditions and explain why infinite exploration is necessary.
4. How does Q-learning relate to the Bellman optimality equation in Chapter 17?
5. When would you choose Expected SARSA over vanilla SARSA?

---

**Previous:** [Section 22.2 - Passive RL](./section-02-passive-reinforcement-learning.md) · **Next:** [Section 22.4 - Generalization](./section-04-generalization-in-reinforcement-learning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), §22.3. [https://aima.cs.berkeley.edu/](https://aima.cs.berkeley.edu/)
2. Watkins, C. J. C. H., & Dayan, P. (1992). "Q-learning." *Machine Learning*, 8(3-4). [https://doi.org/10.1007/BF00992698](https://doi.org/10.1007/BF00992698)
3. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.), Ch. 6-7. [http://incompleteideas.net/book/](http://incompleteideas.net/book/)
4. Van Hasselt, H. (2010). "Double Q-learning." *NeurIPS*. [https://arxiv.org/abs/1009.4224](https://arxiv.org/abs/1009.4224)
5. AIMA Python: `q_learning.py`, `sarsa.py` in [aima-python](https://github.com/aimacode/aima-python)
