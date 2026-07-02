# Section 22.6: Policy Search and Policy Gradients

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22.6  
> **Previous:** [Section 22.5 - Deep RL](./section-05-deep-reinforcement-learning.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Why Search Policies Directly?

**Value-based** methods ([Q-learning](./section-03-active-reinforcement-learning.md), [DQN](./section-05-deep-reinforcement-learning.md)) learn $Q(s,a)$ then derive $\pi(s) = \arg\max_a Q(s,a)$. This works for **discrete** actions but struggles when:

- Action space is **continuous** (robot joint torques)
- Stochastic policies are optimal (rock-paper-scissors, poker)
- $\arg\max$ over neural Q outputs is non-differentiable or expensive

**Policy search** optimizes a parameterized policy $\pi_\mathbf{w}(a \mid s)$ **directly** - often with **policy gradient** methods that ascend the expected return gradient $\nabla_\mathbf{w} J(\mathbf{w})$.

> **Readable form:** Instead of learning "how good is each action" and picking the best, learn the action probabilities themselves and nudge them toward actions that earned high reward.

[Chapter 17](../chapter-17-making-complex-decisions/README.md) policy iteration improves $\pi$ from $Q$ - policy gradients are the **model-free, differentiable** analog.

---

## Policy Parameterizations

**Softmax policy (discrete actions):**

$$
\pi_\mathbf{w}(a \mid s) = \frac{e^{f_\mathbf{w}(s,a)}}{\sum_{a'} e^{f_\mathbf{w}(s,a')}}
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Gaussian policy (continuous actions):**

$$
\pi_\mathbf{w}(a \mid s) = \mathcal{N}\big(a; \mu_\mathbf{w}(s), \sigma_\mathbf{w}^2(s)\big)
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

Neural networks output logits or $(\mu, \sigma)$ per action dimension - [Course 3](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) supplies the architecture.

| Representation | Action type | Exploration |
|----------------|-------------|-------------|
| Softmax | Discrete | Stochastic sampling |
| Gaussian | Continuous | Sample from $\mathcal{N}$ |
| $\varepsilon$-greedy on $Q$ | Discrete | Heuristic |

---

## The Policy Objective

Maximize expected return from start distribution $\rho_0$:

$$
J(\mathbf{w}) = \mathbb{E}_{\tau \sim \pi_\mathbf{w}} \left[ \sum_{t=0}^{T} \gamma^t R_{t+1} \right]
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

where trajectory $\tau = (s_0, a_0, r_1, s_1, \ldots)$.

**Policy search** = any method optimizing $J(\mathbf{w})$:

| Method | Uses gradients? | Sample efficiency |
|--------|-----------------|-------------------|
| REINFORCE | Yes | Low |
| Actor-critic | Yes | Medium |
| PPO / TRPO | Yes | Higher |
| Evolution strategies | No (black-box) | Parallelizable |
| CMA-ES | No | Continuous control |

---

## The Policy Gradient Theorem

Sutton et al. (1999) show:

$$
\nabla_\mathbf{w} J(\mathbf{w}) = \mathbb{E}_{\pi_\mathbf{w}} \left[ \nabla_\mathbf{w} \log \pi_\mathbf{w}(a \mid s) \cdot Q^{\pi_\mathbf{w}}(s, a) \right]
$$
> **Readable form:** Increase log-probability of actions that had high Q-value under your current policy. Good actions get reinforced; bad actions get suppressed.

**Intuition:** $\nabla_\mathbf{w} \log \pi$ points in parameter space to make $(s,a)$ more likely. Weight by $Q(s,a)$ - how much better than average that action was.

**Log-derivative trick** enables this without differentiating through the environment:

$$
\nabla_\mathbf{w} \mathbb{E}_{a \sim \pi}[f(a)] = \mathbb{E}_{a \sim \pi}[f(a) \nabla_\mathbf{w} \log \pi_\mathbf{w}(a)]
$$
> **Readable form:** This derivative measures local sensitivity: a larger magnitude means the output changes more for a small input change.

---

## REINFORCE: Monte Carlo Policy Gradient

Replace $Q^{\pi}(s,a)$ with **full return** $G_t$ from time $t$:

$$
\mathbf{w} \leftarrow \mathbf{w} + \alpha \, G_t \, \nabla_\mathbf{w} \log \pi_\mathbf{w}(a_t \mid s_t)
$$
> **Readable form:** This derivative measures local sensitivity: a larger magnitude means the output changes more for a small input change.

**Algorithm:**

1. Run episode with $\pi_\mathbf{w}$
2. For each step $t$, compute $G_t$
3. Update $\mathbf{w}$ proportional to $G_t \nabla \log \pi(a_t \mid s_t)$

```python
def reinforce_update(policy, trajectory, gamma=0.99, alpha=1e-3):
    # trajectory: list of (s, a, r)
    G = 0.0
    for t in reversed(range(len(trajectory))):
        s, a, r = trajectory[t]
        G = r + gamma * G
        log_prob = policy.log_prob(s, a)
        loss = -G * log_prob  # gradient ascent on J
        loss.backward()
        optimizer.step()
        optimizer.zero_grad()
```

**Properties:**

- **Unbiased** gradient estimate
- **High variance** - $G_t$ noisy
- **On-policy** - must sample from current $\pi_\mathbf{w}$
- Episodic or truncated horizons

Subtract **baseline** $b(s_t)$ from $G_t$ to reduce variance without changing expected gradient:

$$
G_t - b(s_t) \quad \text{common choice: } b(s) = V(s)
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

---

## Actor-Critic Methods

Combine **policy** (actor) with **value function** (critic):

| Component | Role | Update |
|-----------|------|--------|
| Actor | $\pi_\mathbf{w}(a \mid s)$ | Policy gradient |
| Critic | $\hat{V}(s; \mathbf{v})$ or $\hat{Q}(s,a)$ | TD learning |

**Advantage actor-critic (A2C/A3C):**

$$
\nabla_\mathbf{w} J \approx \mathbb{E}\left[ \nabla_\mathbf{w} \log \pi_\mathbf{w}(a \mid s) \cdot A(s,a) \right]
$$
> **Readable form:** This derivative measures local sensitivity: a larger magnitude means the output changes more for a small input change.

**Advantage:**

$$
A(s, a) = Q(s, a) - V(s) \approx r + \gamma V(s') - V(s) = \delta_t
$$
> **Readable form:** Use the TD error as "how surprising was this reward?" - lower variance than full Monte Carlo return, online updates every step.

**A3C:** asynchronous parallel actors (no replay - on-policy). **A2C:** synchronized batch for stability.

---

## Trust Region and Proximal Methods

Large policy gradient steps can **collapse** performance - policy moves to region where old data is irrelevant.

**TRPO** constrains KL divergence between old and new policy:

$$
\max_\mathbf{w} \mathbb{E}\left[\frac{\pi_\mathbf{w}(a|s)}{\pi_{\mathbf{w}_{\text{old}}}(a|s)} A(s,a)\right] \quad \text{s.t. } \mathbb{E}[\text{KL}(\pi_{\mathbf{w}_{\text{old}}} \| \pi_\mathbf{w})] \leq \delta
$$
> **Readable form:** Improve expected advantage under the new policy while constraining the KL distance from the old policy.

**PPO (Proximal Policy Optimization)** clips the probability ratio:

$$
L^{\text{CLIP}} = \mathbb{E}\left[ \min\left( r_t(\mathbf{w}) A_t, \text{clip}(r_t(\mathbf{w}), 1-\varepsilon, 1+\varepsilon) A_t \right) \right]
$$
> **Readable form:** PPO uses the smaller of unclipped and clipped policy-ratio objectives so a single update cannot change action probabilities too aggressively.

where $r_t(\mathbf{w}) = \pi_\mathbf{w}(a_t|s_t) / \pi_{\mathbf{w}_{\text{old}}}(a_t|s_t)$.

> **Readable form:** PPO says "improve the policy, but don't change action probabilities too much in one update" - the workhorse of modern robotics and game AI.

---

## Value-Based vs. Policy-Based vs. Actor-Critic

```
Value-based (DQN)     Policy-based (REINFORCE)     Actor-Critic (PPO)
     Q(s,a)                  π(a|s)                  π + V
        ↓                       ↓                       ↓
   argmax_a                  sample a               sample a
```

| Criterion | Value-based | Policy-based | Actor-critic |
|-----------|-------------|--------------|--------------|
| Continuous actions | Hard | Natural | Natural |
| Convergence (tabular) | Q-learning proofs | Noisy but general | Mixed |
| Sample efficiency | Replay helps | Poor (MC) | Moderate |
| Stochastic policies | Awkward | Natural | Natural |

---

## Deep Deterministic Policy Gradient (DDPG)

For **continuous** control with Q-learning flavor:

- **Actor** $\mu_\mathbf{w}(s)$ outputs deterministic action
- **Critic** $Q_\mathbf{v}(s,a)$ evaluates actor's actions
- Off-policy with replay (like DQN)

$$
\nabla_\mathbf{w} J \approx \mathbb{E}\left[ \nabla_a Q(s,a)|_{a=\mu(s)} \nabla_\mathbf{w} \mu_\mathbf{w}(s) \right]
$$
> **Readable form:** This derivative measures local sensitivity: a larger magnitude means the output changes more for a small input change.

**TD3** adds twin critics and delayed policy updates for stability.

---

## Connection to Chapter 17

Policy iteration in [Chapter 17](../chapter-17-making-complex-decisions/README.md):

1. **Evaluate** $V^\pi$ (policy evaluation)
2. **Improve** $\pi(s) = \arg\max_a Q^\pi(s,a)$

REINFORCE skips explicit $Q$ table - gradient steps on $\pi$ toward high-return trajectories. Actor-critic reintroduces $V$ or $Q$ as critic for the evaluation step - **generalized policy iteration** with function approximation.

---

## Connection to Course 1 and Course 3

Policy gradients use **stochastic gradient ascent** - same optimizer machinery as [Course 3 Chapter 08](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-08-optimization/README.md) (Adam, learning rates, clipping).

Unlike [supervised classification](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/section-02-logistic-regression.md), labels are **returns** or **advantages** - non-stationary, high-variance targets. [Regularization](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-07-regularization/section-01-regularization-strategies.md) and entropy bonuses prevent premature policy collapse.

**RLHF** for LLMs ([Section 22.8](./section-08-applications-and-safety.md)) uses PPO to fine-tune language model policies on human preference rewards - policy gradients at billion-parameter scale.

---

## Entropy Regularization

Encourage exploration by penalizing overly deterministic policies:

$$
J'(\mathbf{w}) = J(\mathbf{w}) + \beta \, \mathbb{E}_{s}\left[ H(\pi_\mathbf{w}(\cdot \mid s)) \right]
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

Entropy $H$ bonus maintains action diversity - critical in multi-modal reward landscapes.

---

## Practical Training Tips

| Tip | Rationale |
|-----|-----------|
| Normalize advantages | Stable gradient scale |
| Clip gradients | Prevent explosion |
| Parallel env rollouts | More samples per wall-clock |
| Observation normalization | Neural net conditioning |
| Reward scaling | Match value network range |

Monitor **KL divergence** between policy updates - early warning of destructive steps.

---

## Common Mistakes

**REINFORCE without baseline.** Variance explodes; training stalls.

**Off-policy data with vanilla policy gradient.** Wrong gradient - use importance sampling or on-policy collection.

**Deterministic policy at exploration time.** Gaussian policies need sampling during training.

**Ignoring action bounds.** Tanh squashing for bounded torque; unbounded Gaussian samples invalid actions.

**Confusing actor-critic with Q-learning.** Critic bootstraps; actor does not $\arg\max$ over discrete set.

---

## Reflection Questions

1. State the policy gradient theorem in words. What does $\nabla \log \pi$ represent?
2. Why does REINFORCE have high variance, and how does a baseline help?
3. When is a policy-based approach preferable to DQN?
4. What problem does PPO's clipped objective solve?
5. How does actor-critic relate to policy iteration in Chapter 17?

---

**Previous:** [Section 22.5 - Deep RL](./section-05-deep-reinforcement-learning.md) · **Next:** [Section 22.7 - Model-Based RL](./section-07-model-based-reinforcement-learning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), §22.6. [https://aima.cs.berkeley.edu/](https://aima.cs.berkeley.edu/)
2. Sutton, R. S., et al. (1999). "Policy Gradient Methods for Reinforcement Learning with Function Approximation." *NeurIPS*. [http://papers.nips.cc/paper/1713-policy-gradient-methods-for-reinforcement-learning-with-function-approximation.pdf](http://papers.nips.cc/paper/1713-policy-gradient-methods-for-reinforcement-learning-with-function-approximation.pdf)
3. Williams, R. J. (1992). "Simple statistical gradient-following algorithms for connectionist reinforcement learning." *Machine Learning*.
4. Schulman, J., et al. (2017). "Proximal Policy Optimization Algorithms." [https://arxiv.org/abs/1707.06347](https://arxiv.org/abs/1707.06347)
5. Lillicrap, T. P., et al. (2016). "Continuous control with deep reinforcement learning." *ICLR* (DDPG). [https://arxiv.org/abs/1509.02971](https://arxiv.org/abs/1509.02971)

