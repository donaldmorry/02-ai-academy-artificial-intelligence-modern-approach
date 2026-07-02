# Chapter 17: Making Complex Decisions

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17  
> **Part:** Part IV: Uncertain Knowledge & Reasoning  
> **Estimated time:** 13–15 hours  
> **Prerequisites:** [Chapter 16 — Making Simple Decisions](../chapter-16-making-simple-decisions/README.md); [Chapter 14 — Probabilistic Reasoning over Time](../chapter-14-probabilistic-reasoning-time/README.md) (recommended)

---

## Chapter Overview

Model sequential decisions with Markov decision processes and partially observable MDPs. Cover value iteration, policy iteration, POMDP belief-state planning, and multi-armed bandits as a foundation for reinforcement learning.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Formulate MDPs with states, actions, transitions, and rewards
2. Compute optimal policies via value iteration and policy iteration
3. Explain the Bellman equations for state and action values
4. Formulate POMDPs and solve small instances over belief states
5. Analyze exploration-exploitation in multi-armed bandits
6. Implement ε-greedy and UCB bandit strategies
7. Connect MDP solutions to RL algorithms in later chapters

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Sequential Decisions](./section-01-sequential-decisions.md) | Policies, rewards, horizons, discounting |
| 2 | [MDPs](./section-02-markov-decision-processes.md) | Transition model, reward function, Markov property |
| 3 | [Value Iteration](./section-03-value-iteration.md) | Bellman optimality, convergence |
| 4 | [Policy Iteration](./section-04-policy-iteration.md) | Policy evaluation, improvement |
| 5 | [POMDPs](./section-05-partially-observable-mdps.md) | Belief states, value function over beliefs |
| 6 | [Bandits](./section-06-multi-armed-bandits.md) | Regret, exploration strategies, UCB1 |
| 7 | [MDP Applications](./section-07-mdp-applications.md) | Inventory, navigation, maintenance |
| 8 | [From MDPs to RL](./section-08-from-mdps-to-reinforcement-learning.md) | Model-free preview, function approximation needs |

---

## Lab / Project

Solve a grid-world MDP with value iteration. Add partial observability in a tiny POMDP and compare myopic vs. belief-based policies.

Hands-on lab: [Lab 17 - Value Iteration on Grid World](./section-lab-17-value-iteration-on-grid-world.md).

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Hyperparameter tuning | Course 1 Chapter 07: bandits for experiment allocation |
| Deep RL | Course 3 Chapter 12: policy gradients and Q-learning at scale |
| Sequential generation | Course 4 Chapter 06: autoregressive decision processes |

---

## Prerequisites

- [Chapter 16 — Making Simple Decisions](../chapter-16-making-simple-decisions/README.md)
- [Chapter 14 — Probabilistic Reasoning over Time](../chapter-14-probabilistic-reasoning-time/README.md) (recommended)

---

## Self-Assessment

1. Write the Bellman equation for optimal state values.
2. What is the difference between a policy and a value function?
3. Why are POMDPs computationally harder than MDPs?
4. Define regret in the multi-armed bandit setting.
5. How does discount factor γ affect optimal policies?
6. When does value iteration converge?

---

**Previous:** [Chapter 16 — Making Simple Decisions](../chapter-16-making-simple-decisions/README.md)  
**Next:** [Chapter 18 — Multiagent Decision Making](../chapter-18-multiagent-decision-making/README.md)
