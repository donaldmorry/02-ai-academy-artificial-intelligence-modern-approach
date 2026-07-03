# Section 22.7: Model-Based Reinforcement Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22.7  
> **Previous:** [Section 22.6 - Policy Search](./section-06-policy-search-and-policy-gradients.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Learning the World, Then Planning in It

[Model-free RL](./section-03-active-reinforcement-learning.md) learns $Q$ or $\pi$ directly from experience. **Model-based RL** learns an explicit model of the environment:

$$
\hat{P}(s' \mid s, a), \quad \hat{R}(s, a, s')
$$
> **Readable form:** Learn a transition model for next states and a reward model for outcomes, then use those learned rules for planning.

then **plans** with that model - value iteration, tree search, or MPC - to choose actions.

> **Readable form:** Model-based RL is like learning the rules of chess by playing, writing them down, then thinking several moves ahead before touching a piece again.

[Chapter 17](../chapter-17-making-complex-decisions/README.md) assumes $P$ and $R$ are **given**. Model-based RL **acquires** them from data - bridging planning (Part III-IV) and learning (Part V).

---

## Why Learn a Model?

| Advantage | Explanation |
|-----------|-------------|
| **Sample efficiency** | Simulated rollouts replace real interactions |
| **Interpretability** | Inspect $\hat{P}$ for errors |
| **Transfer** | Replan under new rewards without relearning $Q$ |
| **Safety** | Test policies in simulation first |

| Disadvantage | Explanation |
|--------------|-------------|
| **Compounding error** | Wrong $\hat{P}$ → wrong plans → wrong data |
| **Model class bias** | Model too simple to capture reality |
| **Compute** | Planning at each step is expensive |

**Model-free** methods avoid model error but need vastly more samples - the classic tradeoff.

---

## Active Adaptive Dynamic Programming (Active ADP)

From [Section 22.3](./section-03-active-reinforcement-learning.md): the earliest model-based RL loop.

1. **Act** with current policy $\pi$ in real environment
2. **Observe** $(s, a, r, s')$
3. **Update** $\hat{P}$ from counts or function approximator
4. **Plan:** run value iteration on $\hat{P}$, $\hat{R}$ to improve $\pi$
5. Repeat

This is **certainty equivalence** - plan as if $\hat{P}$ were true. Works when model is accurate and state space is small.

```python
def update_model(counts, s, a, r, s_next):
    counts[(s, a, s_next)] = counts.get((s, a, s_next), 0) + 1
    # P_hat(s'|s,a) = counts(s,a,s') / sum_s' counts(s,a,s')

def plan_value_iteration(P_hat, R_hat, gamma, states, actions):
    V = {s: 0.0 for s in states}
    for _ in range(100):  # sweeps
        for s in states:
            V[s] = max(a for a in actions
                for sum_s_p in [sum(P_hat.get((s,a,sp),0) *
                    (R_hat.get((s,a,sp),0) + gamma*V[sp])
                    for sp in states)])
    return V
```

---

## Dyna: Integrating Planning and Acting

Sutton's **Dyna** architecture unifies model-free and model-based updates:

1. **Direct RL** - Q-learning (or SARSA) on real experience
2. **Model learning** - update $\hat{P}$, $\hat{R}$ from real transition
3. **Planning** - simulate $k$ **imagined** transitions from model; apply Q-updates

```
Real env ──► Q-learning update
     │
     └──► Model update ──► k simulated transitions ──► Q-learning updates
```

> **Readable form:** Every real step also triggers several "practice" steps in your mental model of the world. You learn from both reality and daydreams.

Dyna-Q with $k=100$ planning steps per real step can learn optimal grid-world policies orders of magnitude faster than pure Q-learning.

**When Dyna fails:** if model is wrong in unreachable (from current policy) regions, planning reinforces hallucinations - **exploration** must populate model coverage.

---

## Monte Carlo Tree Search (MCTS)

For large or continuous state spaces, full value iteration on $\hat{P}$ is infeasible. **MCTS** builds a **sparse search tree** from current state:

Four phases per simulation:

1. **Selection** - traverse tree by UCB1 until leaf
2. **Expansion** - add child node
3. **Simulation** - rollout with default policy to terminal
4. **Backpropagation** - update visit counts and values along path

UCB1 at selection:

$$
a^* = \arg\max_a \left[ Q(s,a) + c \sqrt{\frac{\ln N(s)}{N(s,a)}} \right]
$$
> **Readable form:** MCTS balances trying promising moves (high Q) with exploring rarely tried moves (low visit count). Run thousands of simulations per real move.

**AlphaGo / AlphaZero** combine MCTS with learned $\hat{V}$ and $\hat{\pi}$ ([Section 22.5](./section-05-deep-reinforcement-learning.md)) - no rollout policy needed when neural nets guide search.

| Component | Role |
|-----------|------|
| Learned policy $\hat{\pi}$ | Prior over actions in expansion |
| Learned value $\hat{V}$ | Leaf evaluation instead of rollout |
| MCTS | Lookahead + backup |

---

## Model Learning: Tabular vs. Function Approximation

**Tabular counts:**

$$
\hat{P}(s' \mid s, a) = \frac{N(s, a, s')}{\sum_{s''} N(s, a, s'')}
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Neural dynamics model:**

$$
\hat{s}_{t+1} = f_\mathbf{w}(s_t, a_t), \quad \hat{r}_t = g_\mathbf{w}(s_t, a_t)
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

Train $f_\mathbf{w}$ by supervised regression on $(s,a) \to s'$ transitions - [Course 1 regression](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-02-regression-models/README.md) machinery applies.

**Ensemble models** (PETS, MBPO) capture epistemic uncertainty - plan cautiously where model is uncertain.

---

## Model-Predictive Control (MPC)

For continuous control, optimize action sequence over horizon $H$:

$$
a_{t:t+H}^* = \arg\max_{a_{t:t+H}} \mathbb{E}\left[ \sum_{k=0}^{H} \gamma^k \hat{R}(s_{t+k}, a_{t+k}) \right]
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

subject to $\hat{s}_{t+k+1} = f_\mathbf{w}(\hat{s}_{t+k}, a_{t+k})$.

Execute **only first action** $a_t^*$, observe real $s_{t+1}$, **replan** - receding horizon control.

> **Readable form:** Plan the next few seconds of motor commands, execute the first one, measure where you actually ended up, replan. Classic robotics when dynamics are learnable.

MPC handles model error better than open-loop execution of full $H$-step plan.

---

## World Models and Latent Dynamics

**World models** (Ha & Schmidhuber) learn compressed latent $z_t$:

$$
z_{t+1} = f(z_t, a_t), \quad \hat{o}_{t+1} = g(z_{t+1})
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

Agent trains in **dream** latent space - extreme sample efficiency for visual domains. Connects to [Course 3 autoencoders](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-14-autoencoders/README.md) and representation learning ([Chapter 15](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-15-representation-learning/README.md)).

---

## Comparison: Model-Based vs. Model-Free

| Aspect | Model-based | Model-free |
|--------|-------------|------------|
| Data efficiency | High | Low |
| Computation per step | High (planning) | Low (forward pass) |
| Asymptotic performance | Limited by model | Can exceed with enough data |
| Examples | Dyna, AlphaZero, MBPO | DQN, PPO |
| Error mode | Compounding model bias | Slow convergence |

**Hybrid** systems dominate state-of-the-art: learn model + model-free critic, or short-horizon MPC + learned value.

---

## Connection to Chapter 17

[Chapter 17](../chapter-17-making-complex-decisions/README.md) provides the **planning algorithms** once $P$, $R$ are known:

| Chapter 17 tool | Model-based RL use |
|----------------|-------------------|
| Value iteration | Plan on $\hat{P}$ after each batch of data |
| Policy iteration | Active ADP inner loop |
| POMDP belief update | Model-based under partial observability |
| Bandits | Explore to reduce model uncertainty |

Section 8 of Chapter 17 ("From MDPs to RL") previews this chapter - model-free when $P$ is unknown or too large to estimate reliably.

---

## Exploration for Model Learning

Model quality depends on **coverage** of $(s,a)$ space. Naive exploitation visits narrow tube of states - model blind elsewhere.

**Strategies:**

- $\varepsilon$-greedy during data collection
- **Optimism in face of uncertainty** - reward bonuses for high model variance states
- **Random network distillation** - explore novel states
- **Goal-conditioned** exploration

> **Readable form:** Your mental model is only good where you've actually been. Exploration fills in the map before planning cross-country.

---

## Sim-to-Real Transfer

Robotics pipeline:

1. Learn $\hat{P}$ or policy in **simulator**
2. Domain randomization - vary physics parameters
3. Fine-tune with limited real data (model-free or model-based)

Model-based RL shines when simulation is cheap and reality is expensive ([Chapter 26](../chapter-26-robotics/README.md)).

---

## Python Sketch: Dyna-Q

```python
def dyna_q(env, episodes=50, planning_steps=50, alpha=0.1, gamma=0.99):
    Q, model = {}, {}
    for _ in range(episodes):
        s = env.reset()
        done = False
        while not done:
            a = epsilon_greedy(Q, s, 0.1)
            s_next, r, done, _ = env.step(a)
            # Direct RL
            q_learning_step(Q, s, a, r, s_next, alpha, gamma, done)
            # Model learning
            model[(s, a)] = (r, s_next)
            # Planning
            for _ in range(planning_steps):
                sa = random.choice(list(model.keys()))
                r_sim, sp = model[sa]
                q_learning_step(Q, sa[0], sa[1], r_sim, sp, alpha, gamma, False)
            s = s_next
    return Q
```

---

## Common Mistakes

**Planning with unvalidated model.** Simulated policies fail on hardware.

**No exploration for model coverage.** Dyna reinforces wrong Q in unvisited regions.

**Confusing model-based with offline RL.** Offline uses fixed dataset; model-based often interacts online to improve $\hat{P}$.

**Ignoring stochasticity.** Deterministic models of stochastic envs bias planning.

**Infinite planning horizon without discount.** MPC needs finite $H$ or terminal value estimate.

---

## Reflection Questions

1. What is certainty equivalence, and when does it fail?
2. How does Dyna combine model-free and model-based updates?
3. What roles do learned $\hat{V}$ and $\hat{\pi}$ play in AlphaZero's MCTS?
4. Why does model error compound over long imagined rollouts?
5. How does model-based RL connect to value iteration in Chapter 17?

---

**Previous:** [Section 22.6 - Policy Search](./section-06-policy-search-and-policy-gradients.md) · **Next:** [Section 22.8 - Applications and Safety](./section-08-applications-and-safety.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), §22.7. [https://aima.cs.berkeley.edu/](https://aima.cs.berkeley.edu/)
2. Sutton, R. S. (1990). "Integrated architectures for learning, planning, and reacting based on approximating dynamic programming." *ICML* - Dyna.
3. Browne, C., et al. (2012). "A Survey of Monte Carlo Tree Search Methods." *IEEE TCIAIG*. [https://doi.org/10.1109/TCIAIG.2012.2186810](https://doi.org/10.1109/TCIAIG.2012.2186810)
4. Silver, D., et al. (2017). "Mastering Chess and Shogi by Self-Play with a General Reinforcement Learning Algorithm." *arXiv:1712.01815*. [https://arxiv.org/abs/1712.01815](https://arxiv.org/abs/1712.01815)
5. Chua, K., et al. (2018). "Deep Reinforcement Learning in a Handful of Trials using Probabilistic Dynamics Models." *NeurIPS* (PETS). [https://arxiv.org/abs/1805.12114](https://arxiv.org/abs/1805.12114)
