# Section 17.8: From MDPs to Reinforcement Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17; Chapter 22 preview  
> **Previous:** [Section 17.7 - MDP Applications](./section-07-mdp-applications.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Model Assumption

[Sections 17.1-17.7](./section-01-sequential-decisions.md) assumed the agent knows:

- Transition model $P(s' \mid s, a)$
- Reward function $R(s, a, s')$
- State space $S$ and actions $A$

**Value iteration** and **policy iteration** compute $\pi^*$ **offline** from this model.

**Reinforcement learning (RL)** addresses the case when the model is **unknown** - the agent must learn from **experience** (sequences of $(s, a, r, s')$ tuples).

> **Readable form:** MDP solving is like having a complete map of the casino. RL is learning which slot machines pay off by pulling them - without a map.

---

## Passive vs. Active Learning

| Setting | Agent behavior | Goal |
|---------|----------------|------|
| **Passive** | Follow fixed policy $\pi$ | Estimate $V^\pi$ or model |
| **Active** | Choose actions | Find $\pi^*$ while exploring |

**Adaptive control** blends estimation and optimization - exploration can hurt short-term reward ([Section 17.6](./section-06-multi-armed-bandits.md)).

---

## Model-Based RL

1. **Learn model** $\hat{P}$, $\hat{R}$ from transitions
2. **Plan** with VI/PI on $\hat{M}$
3. **Execute** $\pi$; update model

**Dynalinear / Dyna:** interleave real experience with simulated VI updates from learned model.

**Challenge:** model error compounds; **optimistic** or **uncertainty-aware** planning helps.

---

## Model-Free RL

Directly estimate **value function** or **policy** without explicit $\hat{P}$.

**Temporal-difference learning** for $V^\pi$:

$$
V(s) \leftarrow V(s) + \alpha \left[ r + \gamma V(s') - V(s) \right]
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

**Q-learning** for optimal $Q^*$ (off-policy):

$$
Q(s,a) \leftarrow Q(s,a) + \alpha \left[ r + \gamma \max_{a'} Q(s', a') - Q(s,a) \right]
$$
> **Readable form:** After each step, nudge your estimate toward "reward I got plus discounted value of where I landed." Q-learning picks the best next action in the update.

Converges to $Q^*$ under tabular settings with sufficient exploration (Robbins-Monro step sizes).

---

## The Curse of Dimensionality

Tabular VI/RL stores $|S|$ or $|S| \times |A|$ numbers.

| Problem size | Tabular? |
|--------------|----------|
| Grid $4 \times 3$ | Yes |
| Chess | No ($\sim 10^{43}$ states) |
| Atari (raw pixels) | No |
| Robotics (continuous) | No |

**Function approximation** required:

$$
V(s) \approx \hat{V}(s; \theta) = \theta^\top \phi(s) \quad \text{or neural network}
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

Deep Q-Networks (DQN), policy gradients, actor-critic - [Chapter 22](../chapter-22-reinforcement-learning/README.md).

---

## On-Policy vs. Off-Policy

| Type | Learns | Behavior policy |
|------|--------|-----------------|
| **SARSA** | $Q^\pi$ | Same as learning policy |
| **Q-learning** | $Q^*$ | Off-policy (greedy target) |

Off-policy allows learning optimal behavior from exploratory data; on-policy more stable with function approximation.

---

## Exploration in RL

Beyond $\varepsilon$-greedy and UCB:

- **Softmax** on Q-values
- **Intrinsic motivation** - bonus for novel states
- **Curiosity-driven** - prediction error reward

**Deadly triad (Sutton & Barto):** off-policy + function approximation + bootstrapping can diverge - practical algorithms use targets (DQN), clipped objectives (PPO), etc.

---

## Partial Observability Recap

When states are hidden, tabular RL on observations fails. Solutions:

- **Belief-state MDP** (POMDP planning) - [Section 17.5](./section-05-partially-observable-mdps.md)
- **Recurrent policies** - memory in network
- **State estimation** separate from control

---

## Worked Example: Model-Free on Grid

True grid world unknown to agent. Initialize $Q(s,a) = 0$. For 10,000 episodes:

1. Start at $(1,1)$
2. $\varepsilon$-greedy on $Q$
3. Observe $(s, a, r, s')$
4. Q-learning update
5. Terminal → reset

After training, greedy policy approximates VI optimum without ever storing $P$.

Compare sample efficiency: model-based Dyna often needs fewer real steps on same grid.

---

## Python: Q-Learning Loop

```python
import random

def q_learning(env, alpha=0.1, gamma=0.99, epsilon=0.1, episodes=5000):
    Q = {}
    def get_Q(s, a):
        return Q.get((s, a), 0.0)

    for _ in range(episodes):
        s = env.reset()
        while not env.is_terminal(s):
            actions = env.actions(s)
            if random.random() < epsilon:
                a = random.choice(actions)
            else:
                a = max(actions, key=lambda ac: get_Q(s, ac))
            s_next, r = env.step(s, a)
            best_next = max(get_Q(s_next, ac) for ac in env.actions(s_next)) if not env.is_terminal(s_next) else 0
            Q[(s, a)] = get_Q(s, a) + alpha * (r + gamma * best_next - get_Q(s, a))
            s = s_next
    policy = {s: max(env.actions(s), key=lambda a: get_Q(s, a)) for s in env.all_states()}
    return Q, policy
```

> **Readable form:** Explore randomly sometimes; otherwise improve Q toward observed reward plus best Q at next state.

---

## MDP → RL Concept Map

```
MDP (known model)
  ├── Value iteration / Policy iteration
  └── Model-based RL (learn model → plan)

MDP (unknown model)
  ├── Model-free TD / Q-learning
  ├── Function approximation → Deep RL
  └── POMDP → belief RL / recurrent policies

Bandits (single-state MDP)
  └── Regret-minimizing exploration
```

---

## When to Use Which

| Situation | Approach |
|-----------|----------|
| Accurate simulator available | VI / MPC |
| Model learnable, moderate $|S|$ | Dyna, model-based RL |
| Large/continuous state | Deep RL, policy gradients |
| Safety-critical | Model-based + constraints; cautious exploration |
| Cheap experimentation | Bandits, BO for hyperparameters |

---

## Connection to Course Arc

| Chapter | Role |
|--------|------|
| [16](../chapter-16-making-simple-decisions/README.md) | Utilities → rewards |
| **17** | MDP theory, offline optimal policies |
| [22](../chapter-22-reinforcement-learning/README.md) | Learning $\pi^*$ from interaction |
| [19](../chapter-19-learning-from-examples/README.md) | Supervised learning complements RL (imitation, rewards) |

---

## Common Mistakes

**Assuming RL finds global optimum** - local optima, divergence with bad features.

**No exploration** - Q-learning stuck at suboptimal policy.

**Using test environment for training** without holdout - overfitting transitions.

**Confusing episodic and continuing tasks** in bootstrap targets.

---

## Reflection Questions

1. What does Q-learning store that value iteration does not?
2. When is model-based RL more sample-efficient than Q-learning?
3. Why does function approximation break convergence guarantees?
4. How is the multi-armed bandit a degenerate MDP?
5. What role does the MDP formalism play if you never run VI?

---

**Previous:** [Section 17.7 - MDP Applications](./section-07-mdp-applications.md)  
**Next:** [Chapter 18 - Multiagent Decision Making](../chapter-18-multiagent-decision-making/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Ch. 17 & 22.
2. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning* (2nd ed.).
3. Watkins, C. J. C. H., & Dayan, P. (1992). Q-learning.
4. Silver, D., et al. - Dyna, DQN, AlphaGo planning+learning.
5. aima-python: `rl.py`, `mdp.py`.



