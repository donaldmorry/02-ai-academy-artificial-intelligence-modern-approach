# Section 22.5: Deep Reinforcement Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22.5  
> **Previous:** [Section 22.4 - Generalization](./section-04-generalization-in-reinforcement-learning.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Tabular Q to Deep Q-Networks

[Function approximation](./section-04-generalization-in-reinforcement-learning.md) lets RL scale beyond tiny grids. **Deep reinforcement learning** uses neural networks as function approximators - enabling agents to learn from **high-dimensional sensory input** (pixels, lidar, joint angles) without hand-crafted features.

The landmark **Deep Q-Network (DQN)** (Mnih et al., 2015) played Atari games from raw pixels at human-level performance, combining:

1. [Q-learning](./section-03-active-reinforcement-learning.md) (off-policy TD control)
2. Convolutional neural networks ([Chapter 21](../chapter-21-deep-learning/README.md), [Course 3 Chapter 09](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-09-convolutional-networks/README.md))
3. **Experience replay** and **target networks** for stability

> **Readable form:** DQN is Q-learning with a neural net as the Q-table, plus a replay memory so learning is not destroyed by correlated consecutive frames, plus a frozen copy of the network to compute stable targets.

---

## The DQN Architecture

Input: stack of $m$ recent frames (e.g., $84 \times 84 \times 4$ grayscale) - partial observability handled by history.

$$
\hat{Q}(s, a; \mathbf{w}) = \text{CNN}_\mathbf{w}(\text{frames})
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

Output: one Q-value per discrete action (no $\arg\max$ inside network - output vector of size $|\mathcal{A}|$).

```
Frames → Conv → Conv → Conv → FC → FC → [Q(a₁), Q(a₂), ..., Q(aₙ)]
```

Action selection: $\varepsilon$-greedy on network outputs.

[Course 3 Chapter 06](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) explains forward pass and backprop; DQN applies them with TD targets instead of fixed labels.

---

## The DQN Loss

Sample minibatch $\mathcal{B}$ of transitions $(s, a, r, s', \text{done})$ from replay buffer:

$$
y_i = r_i + \gamma (1 - \text{done}_i) \max_{a'} \hat{Q}(s'_i, a'; \mathbf{w}^-)
$$
> **Readable form:** DQN target equals reward plus discounted best next-state Q-value from the frozen target network, with no future term after terminal states.

$$
\mathcal{L}(\mathbf{w}) = \frac{1}{|\mathcal{B}|} \sum_{i \in \mathcal{B}} \left( y_i - \hat{Q}(s_i, a_i; \mathbf{w}) \right)^2
$$
> **Readable form:** Train the Q-network by minimizing average squared error between target values and current Q-predictions for sampled transitions.

Update $\mathbf{w}$ by gradient descent (Adam, RMSprop - [Course 3 Chapter 08](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-08-optimization/README.md)).

> **Readable form:** Minimize squared error between the network's Q-prediction for the action taken and the one-step Q-learning target computed from reward plus best next-state Q from a **separate frozen** target network.

$\mathbf{w}^-$ denotes **target network** weights - copied from $\mathbf{w}$ every $C$ steps.

---

## Experience Replay

Store transitions in buffer $\mathcal{D}$; sample **uniform random** minibatches for updates.

**Why replay matters:**

| Problem without replay | How replay helps |
|------------------------|------------------|
| Consecutive frames highly correlated | IID-ish samples from buffer |
| Catastrophic forgetting of rare events | Old transitions revisited |
| Non-stationary data distribution | Mixes past policies |

Buffer size: typically $10^5$-$10^6$ transitions. **Prioritized replay** samples transitions with high TD error more often - faster learning on informative events.

```python
import random
from collections import deque

class ReplayBuffer:
    def __init__(self, capacity=100_000):
        self.buffer = deque(maxlen=capacity)

    def push(self, s, a, r, s_next, done):
        self.buffer.append((s, a, r, s_next, done))

    def sample(self, batch_size):
        return random.sample(self.buffer, batch_size)
```

---

## Target Networks

The Q-learning target uses $\max_{a'} \hat{Q}(s', a'; \mathbf{w})$. If the same $\mathbf{w}$ appears in target and prediction, learning **chases a moving target** - oscillation and divergence ([deadly triad](./section-04-generalization-in-reinforcement-learning.md)).

**Fix:** maintain $\mathbf{w}^-$ fixed for $C$ environment steps:

$$
\mathbf{w}^- \leftarrow \mathbf{w} \quad \text{every } C \text{ steps}
$$
> **Readable form:** The value expression summarizes expected discounted future reward from the current state or policy.

Or **soft update** (Polyak averaging):

$$
\mathbf{w}^- \leftarrow \tau \mathbf{w} + (1 - \tau) \mathbf{w}^-
$$
> **Readable form:** Keep a slow-moving snapshot of the network for computing "what should Q be?" while the main network catches up - like grading an exam with last year's answer key, updated occasionally.

---

## DQN Training Loop

```python
def train_dqn(env, q_net, target_net, buffer, optimizer, gamma=0.99):
    s = preprocess(env.reset())
    for step in range(total_steps):
        a = epsilon_greedy(q_net(s), eps)
        s_next, r, done, _ = env.step(a)
        s_next = preprocess(s_next)
        buffer.push(s, a, r, s_next, done)

        if len(buffer) > min_replay:
            batch = buffer.sample(BATCH_SIZE)
            loss = compute_dqn_loss(batch, q_net, target_net, gamma)
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()

        if step % TARGET_SYNC == 0:
            target_net.load_state_dict(q_net.state_dict())

        s = s_next if not done else preprocess(env.reset())
```

**Preprocessing:** grayscale, resize, crop, frame stacking, reward clipping ($r \in \{-1, 0, +1\}$) - stabilizes training across games.

---

## Beyond Vanilla DQN

| Extension | Idea | Benefit |
|-----------|------|---------|
| **Double DQN** | Use $Q$ to select $a'$, target net to evaluate | Less overestimation |
| **Dueling DQN** | Separate $V(s)$ and advantage $A(s,a)$ streams | Better value decomposition |
| **Rainbow** | Combines 6+ improvements | State-of-art Atari |
| **Noisy Nets** | Learned exploration noise in weights | Replace $\varepsilon$-greedy |

Double DQN target:

$$
y = r + \gamma \hat{Q}\left(s', \arg\max_{a'} \hat{Q}(s', a'; \mathbf{w}); \mathbf{w}^-\right)
$$
> **Readable form:** Pick the best action using the main network, but score that action with the target network - decouples selection from evaluation.

---

## DQN Limitations

**Discrete actions only.** $\arg\max$ over finite action set - unsuitable for continuous torque control.

**Sample inefficiency.** Millions of frames for Atari; humans learn faster.

**Stability still fragile.** Hyperparameters matter; reward clipping can distort task semantics.

**Offline RL gap.** Standard DQN assumes online data collection - dangerous for real robots ([Section 22.8](./section-08-applications-and-safety.md)).

For continuous control, see [Section 22.6](./section-06-policy-search-and-policy-gradients.md) (DDPG, PPO) and [Course 3](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-12-applications/README.md) for scaled training infrastructure.

---

## Connection to Q-Learning Theory

Tabular Q-learning converges under GLIE ([Section 22.3](./section-03-active-reinforcement-learning.md)). DQN **has no such guarantee** - function approximation + off-policy + bootstrapping without careful engineering.

Practical stabilizers map to theory:

| DQN trick | Addresses |
|-----------|-----------|
| Replay | Correlated sample bias |
| Target network | Moving target / deadly triad |
| Gradient clipping | Exploding updates |
| Reward clipping | Scale-sensitive Q-values |

---

## Connection to Chapter 17 and Chapter 21

[Chapter 17](../chapter-17-making-complex-decisions/README.md) value iteration assumes known $P$ and enumerable states. DQN is **model-free** value iteration at scale with a neural representer.

[Chapter 21](../chapter-21-deep-learning/README.md) provides CNN building blocks; DQN is a **deployment** of deep learning where labels are bootstrapped TD targets, not human annotations - a fundamentally different training loop than [Course 1 Keras models](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/section-02-your-first-keras-model.md).

---

## AlphaGo and Hybrid Systems

Pure DQN is model-free. **AlphaGo** combines:

- Deep networks for value $\hat{V}(s)$ and policy $\hat{\pi}(s)$
- **Monte Carlo tree search** (MCTS) for lookahead
- Self-play RL

This is **deep RL + planning** - bridges to [Section 22.7](./section-07-model-based-reinforcement-learning.md) and adversarial search ([Chapter 05](../chapter-05-adversarial-search-games/README.md)).

$$
a^* \approx \arg\max_a \text{MCTS}\big(s, \hat{\pi}, \hat{V}\big)
$$
> **Readable form:** Neural nets suggest promising moves; tree search simulates futures using those hints - best of learning and planning.

---

## Hardware and Scale

Training DQN on 49 Atari games used a single GPU for days. Modern labs distribute RL across thousands of actors ([Course 3 Chapter 12](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-12-applications/section-01-large-scale-deep-learning.md)):

- Parallel actors collect experience
- Learner updates GPU network
- IMPALA, SEED RL architectures

---

## Evaluation Protocol

Report **median human-normalized score** across games - not cherry-picked peaks.

| Metric | Meaning |
|--------|---------|
| Episodic return | Total reward per game |
| Human normalized | Agent score / human score |
| Sample efficiency | Steps to threshold performance |

Always evaluate with $\varepsilon = 0$ (greedy) over multiple random seeds.

---

## Common Mistakes

**Training without target network.** Classic divergence.

**Tiny replay buffer.** Overfits recent policy.

**Not preprocessing frames.** Raw 210×160 RGB is noisy and slow.

**Using DQN for continuous actions.** Discretization loses precision.

**Ignoring frame stacking.** Violates Markov property for velocity.

---

## Reflection Questions

1. What roles do experience replay and target networks play in DQN?
2. Why is DQN off-policy, and how does replay reinforce that?
3. How does Double DQN address Q-learning overestimation?
4. What prevents DQN from inheriting tabular Q-learning convergence proofs?
5. How does AlphaGo combine model-free learning with search?

---

**Previous:** [Section 22.4 - Generalization](./section-04-generalization-in-reinforcement-learning.md) · **Next:** [Section 22.6 - Policy Search](./section-06-policy-search-and-policy-gradients.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), §22.5. [https://aima.cs.berkeley.edu/](https://aima.cs.berkeley.edu/)
2. Mnih, V., et al. (2015). "Human-level control through deep reinforcement learning." *Nature*. [https://www.nature.com/articles/nature14236](https://www.nature.com/articles/nature14236)
3. Van Hasselt, H., Guez, A., & Silver, D. (2016). "Deep Reinforcement Learning with Double Q-learning." *AAAI*. [https://arxiv.org/abs/1509.06461](https://arxiv.org/abs/1509.06461)
4. Wang, Z., et al. (2016). "Dueling Network Architectures for Deep Reinforcement Learning." *ICML*. [https://arxiv.org/abs/1511.06581](https://arxiv.org/abs/1511.06581)
5. Silver, D., et al. (2016). "Mastering the game of Go with deep neural networks and tree search." *Nature*. [https://www.nature.com/articles/nature16961](https://www.nature.com/articles/nature16961)

