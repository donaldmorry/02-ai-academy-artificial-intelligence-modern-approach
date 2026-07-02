# Chapter 22: Reinforcement Learning

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22
> **Part:** Part V: Machine Learning
> **Estimated time:** 13–15 hours
> **Prerequisites:** Chapter 17: Making Complex Decisions; Chapter 21: Deep Learning

---

## Chapter Overview

Treat RL as learning optimal behavior from reward signals without a labeled dataset. Cover passive and active RL, temporal-difference learning, Q-learning, policy search, function approximation, and deep RL (DQN, policy gradients). Connect to MDP theory.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Distinguish RL from supervised and unsupervised learning
2. Implement model-based and model-free RL on grid worlds
3. Derive temporal-difference updates for V and Q functions
4. Implement Q-learning and SARSA with exploration strategies
5. Apply function approximation and deep Q-networks
6. Explain policy gradient and actor-critic methods at high level
7. Analyze exploration, stability, and reward design challenges

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [RL Problem Setup](./section-01-rl-problem-setup.md) | Agent, environment, reward hypothesis |
| 2 | [Passive RL](./section-02-passive-reinforcement-learning.md) | Direct utility estimation, adaptive DP, TD(0) |
| 3 | [Active RL](./section-03-active-reinforcement-learning.md) | Exploration, Q-learning, SARSA |
| 4 | [Generalization](./section-04-generalization-in-reinforcement-learning.md) | Function approximation, tile coding, neural nets |
| 5 | [Deep RL](./section-05-deep-reinforcement-learning.md) | DQN, experience replay, target networks |
| 6 | [Policy Search](./section-06-policy-search-and-policy-gradients.md) | REINFORCE, actor-critic, PPO overview |
| 7 | [Model-Based RL](./section-07-model-based-reinforcement-learning.md) | Learning models, planning with learned dynamics |
| 8 | [Applications and Safety](./section-08-applications-and-safety.md) | Games, robotics, offline RL, reward hacking |

---

## Lab / Project

Train a Q-learning agent on FrozenLake or a custom grid world. Extend with a neural network function approximator for larger state spaces.

Hands-on lab: [Lab 22 - Tabular Q-Learning](./section-lab-22-tabular-q-learning.md).

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| MDP theory | Course 2 Chapter 17: Bellman equations foundation |
| Neural function approximation | [Course 1 Chapter 09](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/README.md): Keras training basics |
| Deep RL at scale | [Course 3 Chapter 12](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-12-applications/README.md): advanced applications |
| Optimization for policy learning | [Course 3 Chapter 08](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-08-optimization/README.md): SGD, Adam |

---

## Prerequisites

- Chapter 17: Making Complex Decisions
- Chapter 21: Deep Learning

---

## Self-Assessment

1. What is the difference between Q-learning and SARSA?
2. Why is exploration necessary in active RL?
3. How does temporal-difference learning combine MC and DP ideas?
4. What roles do experience replay and target networks play in DQN?
5. When does the reward hypothesis fail in practice?
6. Contrast value-based and policy-based deep RL.

---

**Previous:** [Chapter 21 — Deep Learning](../chapter-21-deep-learning/README.md)
**Next:** [Chapter 23 — Natural Language Processing](../chapter-23-natural-language-processing/README.md)
