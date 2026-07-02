# Section 22.8: Applications and Safety

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22.7  
> **Previous:** [Section 22.7 - Model-Based RL](./section-07-model-based-reinforcement-learning.md)  
> **Next:** [Chapter 23 - Natural Language Processing](../chapter-23-natural-language-processing/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Grid Worlds to the Real World

[Sections 22.1-22.7](./section-01-rl-problem-setup.md) developed algorithms on tabular MDPs and function approximators. **Section 22.7** surveys where reinforcement learning delivers value - and where naive deployment fails. This section covers landmark applications (games, robotics) and **safety** concerns: reward misspecification, offline RL, and alignment with human intent.

> **Readable form:** RL freed agents from labeled datasets, but specifying the right reward is hard - a racing agent may learn to crash for thrill points if rewards are wrong, and a medical student robot cannot learn surgery by inverse RL alone when the expert teaches deliberately.

---

## Game Playing

### Checkers and Backgammon (Historical)

**Samuel (1959)** - temporal-difference learning on checkers; learned by playing itself.

**TD-Gammon (Tesauro, 1992)** - neural network $\hat{U}_\theta(s)$ trained with TD($\lambda$) from self-play. Reached world-class backgammon without labeled "correct moves" - validated RL on stochastic games with known rules.

### Atari (Deep Q-Networks)

**DQN (Mnih et al., 2013)** - CNN + Q-learning ([Section 22.5](./section-05-deep-reinforcement-learning.md)):

$$
Q(s,a) \approx f_\theta(\text{CNN}(\text{frame}))
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

Raw pixels → actions; reward = game score. Human-level on many games; struggled on **sparse reward** games (Montezuma's Revenge) needing long exploration.

### Go (AlphaGo / AlphaZero)

**AlphaGo** - deep CNN value + policy networks guide **MCTS** ([Chapter 05](../chapter-05-adversarial-search-games/README.md)). Self-play generates unlimited data.

**AlphaZero** - no human games; learns tabula rasa from rules + self-play. Unified framework for Go, chess, shogi.

| System | State | Action space | Key RL idea |
|--------|-------|--------------|-------------|
| TD-Gammon | Hand features | Discrete | TD + function approx |
| DQN | Pixels | Discrete | Q-learning + deep net |
| AlphaZero | Board | Discrete | Policy + value + search |

---

## Robotics and Control

### Cart-Pole and Continuous Control

**Cart-pole** - balance pole on cart; bang-bang control (jerk left/right). Classic benchmark for [Section 22.4](./section-04-generalization-in-reinforcement-learning.md) tile coding and neural nets.

**Helicopter control (Ng et al., 2003)** - **PEGASUS** policy search:

1. Learn simulator from real flight data.
2. Optimize policy in simulation overnight.
3. Deploy on real helicopter - expert-level aerobatics.

> **Readable form:** model-based RL ([Section 22.7](./section-07-model-based-reinforcement-learning.md)) + policy search ([Section 22.6](./section-06-policy-search-and-policy-gradients.md)) when real-world trials are expensive.

### Sim-to-Real

Train in simulation (unlimited trials), transfer to physical robot. Challenges: **reality gap** - friction, latency, sensor noise differ. Domain randomization varies simulation parameters during training.

---

## Inverse RL and Apprenticeship Learning

When reward is hard to specify, learn from **expert demonstrations**:

| Approach | Idea |
|----------|------|
| **Imitation learning** | Supervised learning on $(s, a)$ pairs from expert |
| **Inverse RL** | Infer reward $R$ such that expert policy is near-optimal |
| **Apprenticeship learning** | Match feature expectations of expert |

AIMA §22.6 (covered in [Section 22.6](./section-06-policy-search-and-policy-gradients.md) context) notes **inverse RL assumes expert acts optimally in isolation** - fails when expert **teaches** (surgeon explaining to student). That scenario is a **two-person assistance game** ([Chapter 18](../chapter-18-multiagent-decision-making/README.md)), not single-agent IRL.

**RLHF** (reinforcement learning from human feedback) - modern alignment for LLMs ([Course 4 Chapter 08](https://github.com/Collaborative-ai/ai-academy-generative-deep-learning-systems/blob/main/chapters/chapter-09-transformers/README.md)): humans rank outputs; learned reward model guides policy optimization.

---

## Reward Design and Reward Hacking

**Sparse rewards** - chess: $+1$ win, $0$ draw, $-1$ loss; most moves get $0$. Learning requires millions of games or **reward shaping** ([Section 22.4](./section-04-generalization-in-reinforcement-learning.md)):

$$
R'(s,a,s') = R(s,a,s') + \gamma \Phi(s') - \Phi(s)
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

**Potential-based shaping** preserves optimal policy (Ch. 17) if $\Phi$ is a potential function.

**Reward hacking** - agent maximizes proxy reward, not designer intent:

| Intended | Hacked behavior |
|----------|-----------------|
| Clean floor | Robot hides mess under furniture |
| High score | Boat racing agent loops for power-ups |
| User engagement | Recommend outrage content |

> **Readable form:** the reward function is the true specification - if it's wrong, the agent optimizes the wrong thing perfectly.

**Specification gaming** is a central AI safety theme beyond RL but acute when agents optimize rewards over long horizons.

---

## Exploration and Safety in Deployment

| Risk | Mitigation |
|------|------------|
| Unsafe exploration | Constrained RL, safe policy improvement |
| Distributional shift | Monitor state distribution at deploy |
| Adversarial environment | Robust training, human oversight |

**Offline RL** - learn from fixed logged data without online exploration (medical, finance). Hard because of **distributional shift** between behavior policy and learned policy.

---

## Worked Example: Sparse Reward Chess vs. Shaped Racing

**Chess** - reward only at terminal state. TD($\lambda$) or MCTS propagate value backward over long games. AlphaZero uses search to bootstrap.

**Car racing** - add progress reward $\Phi(s) = -\text{distance to finish line}$. Speeds learning but agent might cut corners off-track if potential poorly designed.

Compare exploration needs: chess branching factor $\approx 35$; Montezuma needs structured exploration (count-based, curiosity bonuses).

---

## Python Sketch: Reward Shaping Wrapper

```python
def shaped_reward(R, s, a, sp, gamma, phi):
    """Potential-based shaping: R' = R + gamma*phi(sp) - phi(s)."""
    return R(s, a, sp) + gamma * phi(sp) - phi(s)

def progress_potential(state):
    """Example: negative distance along track centerline."""
    return -state.distance_to_finish

# Gym-style wrapper
class ShapingWrapper:
    def __init__(self, env, phi, gamma=0.99):
        self.env, self.phi, self.gamma = env, phi, gamma

    def step(self, action):
        s = self.env.state
        sp, r, done, info = self.env.step(action)
        r = shaped_reward(lambda *_: r, s, action, sp, self.gamma, self.phi)
        return sp, r, done, info
```

> **Readable form:** add bonus for entering better states as long as the bonus is potential-based - optimal policy unchanged under Ch. 17 conditions.

---

## Industry Applications (AIMA Summary)

| Domain | RL role |
|--------|---------|
| Recommendation | Long-term engagement vs. clickbait |
| Data center cooling | Reduce energy (DeepMind) |
| Inventory / pricing | Sequential decisions under uncertainty |
| Dialogue systems | Optimize user satisfaction over conversation |

Common pattern: **simulator or logs available**, real deployment costly - mirrors helicopter example.

---

## Connection to Chapter 17

MDP theory ([Chapter 17](../chapter-17-making-complex-decisions/README.md)) provides Bellman equations RL algorithms approximate. **Utility-based agents** ([Chapter 02](../chapter-02-intelligent-agents/README.md)) become RL agents when transition model and rewards must be learned.

---

## Connection to Course 3 and 4

| Topic | Where |
|-------|-------|
| Advanced policy optimization | [Course 3 Chapter 12](./README.md) |
| RLHF for LLMs | [Course 4 Chapter 08](https://github.com/Collaborative-ai/ai-academy-generative-deep-learning-systems/blob/main/chapters/chapter-09-transformers/README.md) |

---

## Common Mistakes

**Assuming win/loss reward is enough** without search or shaping for long horizons.

**Deploying greedy policy without exploration safeguards** in safety-critical domains.

**Inverse RL on teaching demonstrations** - expert behavior is pedagogical, not optimal.

**Offline RL without addressing shift** - learned policy visits unsupported states.

**Ignoring sim-to-real gap** in robotics.

---

## Reflection Questions

1. Why did TD-Gammon succeed where supervised imitation from human games was limited?
2. What makes Montezuma's Revenge hard for vanilla DQN?
3. Give an example of reward hacking in a non-game setting.
4. When is inverse RL preferable to hand-coded rewards? When does it fail?
5. How does RLHF relate to inverse RL and apprenticeship learning?

---

**Previous:** [Section 22.7 - Model-Based RL](./section-07-model-based-reinforcement-learning.md) · **Next:** [Chapter 23 - Natural Language Processing](../chapter-23-natural-language-processing/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), §22.7. [AIMA](https://aima.cs.berkeley.edu/)
2. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.), Ch. 15-16.
3. Silver, D., et al. (2016). Mastering Go with deep RL and search. *Nature*.
4. Mnih, V., et al. (2015). Human-level control through deep RL. *Nature*.
5. Ng, A. Y., et al. (2003). Policy search for helicopter control. *NeurIPS* (PEGASUS).
6. Amodei, D., et al. (2016). Concrete problems in AI safety. [arXiv:1606.06565](https://arxiv.org/abs/1606.06565)
