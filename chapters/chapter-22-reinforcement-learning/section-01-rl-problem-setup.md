# Section 22.1: RL Problem Setup

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22.1  
> **Previous:** [Chapter 21 - Deep Learning](../chapter-21-deep-learning/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Learning from Interaction, Not Labels

[Supervised learning](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-03-supervised-vs-unsupervised-learning.md) gives the agent a dataset of input-output pairs: "this email is spam," "this image is a cat." The agent never chooses what data to see next - the training set is fixed before learning begins.

**Reinforcement learning (RL)** inverts part of that contract. The agent **acts** in an environment, receives **observations** and **scalar rewards**, and must discover a **policy** - a mapping from states to actions - that maximizes cumulative reward over time. There is no supervisor labeling the correct action at each step; only occasional feedback signals whether things are going well or poorly.

> **Readable form:** RL is like learning to ride a bike. Nobody tells you the exact handlebar angle at each millisecond. You try, wobble, fall (negative reward), eventually balance (positive reward), and improve from experience.

This section formalizes the RL problem and connects it to **Markov decision processes (MDPs)** from [Chapter 17](../chapter-17-making-complex-decisions/README.md), where the transition model $P(s' \mid s, a)$ and reward function $R(s, a)$ are **known**. In RL, those ingredients are typically **unknown** - the agent must learn from samples.

---

## The Agent-Environment Interface

At each discrete time step $t$:

1. The agent observes state $S_t$ (or observation $O_t$ if not fully observable)
2. The agent selects action $A_t$ according to policy $\pi$
3. The environment transitions to $S_{t+1}$ and emits reward $R_{t+1}$

```
     ┌─────────┐  action A_t   ┌─────────────┐
     │  Agent  │ ────────────► │ Environment │
     │  (π)    │ ◄──────────── │             │
     └─────────┘  reward R_{t+1}└─────────────┘
                  state S_{t+1}
```

The **history** is the sequence of observations, actions, and rewards:

$$
H_t = O_1, A_1, R_2, O_2, A_2, R_3, \ldots, O_t
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

A **Markov** policy depends only on the current state (or belief state in POMDPs):

$$
\pi(a \mid s) = P(A_t = a \mid S_t = s)
$$
> **Readable form:** The policy is the agent's strategy - for each situation, what action to take. In MDP theory we seek the optimal policy $\pi^*$; in RL we **learn** $\pi$ from experience.

---

## MDP Recap: The Target Problem

[Chapter 17](../chapter-17-making-complex-decisions/README.md) defines an MDP as $\langle \mathcal{S}, \mathcal{A}, P, R, \gamma \rangle$:

| Component | Symbol | Meaning |
|-----------|--------|---------|
| States | $\mathcal{S}$ | All possible situations |
| Actions | $\mathcal{A}$ | Choices available to the agent |
| Transitions | $P(s' \mid s, a)$ | Dynamics of the world |
| Rewards | $R(s, a)$ or $R(s, a, s')$ | Immediate feedback |
| Discount | $\gamma \in [0, 1]$ | Weight of future vs. present reward |

The **return** from time $t$ is discounted cumulative reward:

$$
G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}
$$
> **Readable form:** Return is the total score from now onward, with future rewards counted less heavily when $\gamma < 1$. This keeps infinite horizons finite and reflects that immediate outcomes often matter more.

The **optimal state-value function** satisfies the Bellman optimality equation from Chapter 17:

$$
V^*(s) = \max_a \sum_{s'} P(s' \mid s, a) \left[ R(s, a, s') + \gamma V^*(s') \right]
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

RL algorithms approximate $V^*$, $Q^*$, or $\pi^*$ **without** explicit access to $P$.

---

## The Reward Hypothesis

Sutton and Barto (and AIMA) articulate the **reward hypothesis**: goals and purposes can be formalized as the maximization of expected cumulative reward. Design the reward function $R$, and rational behavior follows.

This is powerful but dangerous (see [Section 22.8](./section-08-applications-and-safety.md)):

| Well-designed reward | Pathological reward |
|----------------------|---------------------|
| +1 for reaching goal | Agent loops forever collecting tiny bonuses |
| −1 per step (encourage speed) | Agent refuses to act to avoid penalties |
| Shaped reward for progress | Agent exploits loopholes in the shaping |

> **Readable form:** The reward function is the specification language for RL. Get it wrong and the agent optimizes the wrong thing brilliantly.

---

## RL vs. Other Learning Paradigms

| Paradigm | Training signal | Who generates data? | Example |
|----------|-----------------|---------------------|---------|
| Supervised | Labels $y$ for inputs $x$ | External dataset | Image classification ([Course 1 Chapter 03](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/README.md)) |
| Unsupervised | Structure in $x$ alone | External dataset | Clustering ([Course 1 Chapter 01](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md)) |
| Reinforcement | Scalar reward $R_t$ | **Agent's own actions** | Game playing, robotics |

**Key distinction:** In RL, the agent's current policy **changes the distribution** of future training data. A bad policy visits bad states and learns slowly - the **exploration problem** ([Section 22.3](./section-03-active-reinforcement-learning.md)).

Deep RL combines RL with neural networks from [Chapter 21](../chapter-21-deep-learning/README.md) and [Course 3](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) - scaling to Atari pixels and continuous control.

---

## Episodic vs. Continuing Tasks

**Episodic tasks** have terminal states: chess ends in win/loss/draw; a grid-world episode ends at the goal or cliff. The return $G_t$ is summed over a finite horizon.

**Continuing tasks** have no terminal state: inventory management, server load balancing, lifelong learning. Discounting $\gamma < 1$ is essential for bounded returns.

**Finite vs. infinite horizon:**

$$
\text{Finite horizon } H: \quad G_t = \sum_{k=0}^{H-1} R_{t+k+1}
$$

$$
\text{Infinite discounted:} \quad G_t = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}
$$
> **Readable form:** Episodic = one game per training episode. Continuing = the game never ends; discounting prevents infinite totals.

---

## Fully Observable vs. Partially Observable

When the agent sees the full state $S_t$, the problem is an MDP. When observations $O_t$ are noisy or incomplete, it is a **POMDP** - belief-state planning from Chapter 17 applies, and RL must learn over histories or recurrent representations.

| Setting | Agent sees | RL approach |
|---------|------------|-------------|
| MDP | True state $s$ | Tabular or function approximation on $s$ |
| POMDP | Observation $o$ | Memory, RNN, belief vectors |

Atari games are POMDPs from pixels alone - one frame does not reveal velocity. DQN and successors stack frames or use recurrent cores ([Section 22.5](./section-05-deep-reinforcement-learning.md)).

---

## Value Functions and Action-Value Functions

The **state-value function** for policy $\pi$:

$$
V^\pi(s) = \mathbb{E}_\pi[G_t \mid S_t = s]
$$
> **Readable form:** The value expression summarizes expected discounted future reward from the current state or policy.

The **action-value function** (Q-function):

$$
Q^\pi(s, a) = \mathbb{E}_\pi[G_t \mid S_t = s, A_t = a]
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

Relationship:

$$
V^\pi(s) = \sum_a \pi(a \mid s) \, Q^\pi(s, a)
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

Optimal Q-values satisfy:

$$
Q^*(s, a) = \sum_{s'} P(s' \mid s, a) \left[ R(s, a, s') + \gamma \max_{a'} Q^*(s', a') \right]
$$
> **Readable form:** $V$ tells you how good a state is under your policy. $Q$ tells you how good a specific action is in that state. Optimal control picks $\arg\max_a Q^*(s, a)$.

Chapter 17 computes these via value iteration when $P$ is known. RL estimates them from trajectories ([Section 22.2](./section-02-passive-reinforcement-learning.md)).

---

## Model-Based vs. Model-Free RL

| Approach | Learns | Planning | Example |
|----------|--------|----------|---------|
| **Model-based** | $\hat{P}$, $\hat{R}$ then plans | Dyna, MCTS, world models | [Section 22.7](./section-07-model-based-reinforcement-learning.md) |
| **Model-free** | $V$, $Q$, or $\pi$ directly | None (trial-and-error) | Q-learning, policy gradients |

Model-based methods sample-efficient but suffer if the learned model is wrong. Model-free methods are robust but data-hungry - deep RL on GPUs addresses scale ([Course 3 Chapter 12](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-12-applications/section-01-large-scale-deep-learning.md)).

---

## On-Policy vs. Off-Policy

| Type | Behavior policy | Target policy | Example |
|------|-----------------|---------------|---------|
| **On-policy** | Same as learning target | Same | SARSA |
| **Off-policy** | Can differ (e.g., explore) | Greedy w.r.t. $Q$ | Q-learning |

Off-policy learning reuses old data and separates exploration from exploitation - critical for replay buffers in DQN.

---

## Python Sketch: Grid-World Environment API

```python
import numpy as np

class GridWorld:
    """Tiny MDP: agent moves on grid, +1 at goal, -1 per step."""
    def __init__(self, size=4, goal=(3, 3)):
        self.size = size
        self.goal = goal
        self.reset()

    def reset(self):
        self.pos = (0, 0)
        return self._state()

    def _state(self):
        return self.pos[0] * self.size + self.pos[1]

    def step(self, action):
        # actions: 0=N, 1=E, 2=S, 3=W
        dr, dc = [(-1, 0), (0, 1), (1, 0), (0, -1)][action]
        r, c = self.pos
        r = np.clip(r + dr, 0, self.size - 1)
        c = np.clip(c + dc, 0, self.size - 1)
        self.pos = (r, c)
        done = self.pos == self.goal
        reward = 1.0 if done else -0.01
        return self._state(), reward, done, {}
```

> **Readable form:** Standard RL APIs return `(observation, reward, done, info)` after each `step(action)` - the same contract used by OpenAI Gym and Gymnasium.

---

## Connection to the Learning Agent Architecture

[Chapter 02](../chapter-02-intelligent-agents/section-05-learning-agents.md) describes a learning agent with performance, critic, learning, and problem generator elements. RL maps cleanly:

| Learning agent component | RL counterpart |
|--------------------------|----------------|
| Performance element | Current policy $\pi$ |
| Critic | Value function $V$ or $Q$ |
| Learning element | TD / Q / policy gradient updates |
| Problem generator | Exploration strategy |

The **critic** provides the learning signal; without it, the agent cannot assign credit across time.

---

## Common Mistakes

**Treating RL as supervised learning with delayed labels.** Actions affect future states; i.i.d. assumption fails.

**Ignoring discounting.** Without $\gamma < 1$, continuing tasks have undefined objectives.

**Assuming full observability.** Raw pixels require representation learning ([Chapter 21](../chapter-21-deep-learning/README.md)).

**Confusing reward with utility.** Chapter 16 utilities are normative preferences; RL rewards are design artifacts that may misalign ([Section 22.8](./section-08-applications-and-safety.md)).

---

## Chapter Roadmap

| Section | Focus |
|--------|-------|
| [22.2](./section-02-passive-reinforcement-learning.md) | Learn $V$ for a **fixed** policy (TD) |
| [22.3](./section-03-active-reinforcement-learning.md) | Learn $Q^*$ and improve policy (Q-learning, SARSA) |
| [22.4](./section-04-generalization-in-reinforcement-learning.md) | Function approximation for large $\mathcal{S}$ |
| [22.5](./section-05-deep-reinforcement-learning.md) | DQN, experience replay |
| [22.6](./section-06-policy-search-and-policy-gradients.md) | Policy gradients |
| [22.7](./section-07-model-based-reinforcement-learning.md) | Learn model, plan |
| [22.8](./section-08-applications-and-safety.md) | Deployment and alignment |

---

## Reflection Questions

1. What elements of an MDP are unknown in a typical RL problem? Which does Chapter 17 assume known?
2. Why must an RL agent explore when a supervised learner does not?
3. Give an example where the reward hypothesis leads to unintended behavior.
4. When is $V^\pi$ sufficient, and when do you need $Q^\pi$?
5. How does the agent-environment loop relate to the learning agent diagram in Chapter 02?

---

**Previous:** [Chapter 21 - Deep Learning](../chapter-21-deep-learning/README.md) · **Next:** [Section 22.2 - Passive RL](./section-02-passive-reinforcement-learning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), §22.1. [https://aima.cs.berkeley.edu/](https://aima.cs.berkeley.edu/)
2. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.), Ch. 3. [http://incompleteideas.net/book/](http://incompleteideas.net/book/)
3. Silver, D., et al. (2015). "Introduction to Reinforcement Learning." DeepMind lectures. [https://www.davidsilver.uk/teaching/](https://www.davidsilver.uk/teaching/)
4. OpenAI Gymnasium documentation. [https://gymnasium.farama.org/](https://gymnasium.farama.org/)
5. AIMA Python: `rl.py` in [aima-python](https://github.com/aimacode/aima-python)
