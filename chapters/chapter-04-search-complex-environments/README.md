# Chapter 04: Search in Complex Environments

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 4  
> **Part:** Part II: Problem-Solving  
> **Estimated time:** 11–13 hours  
> **Prerequisites:** [Chapter 03 — Solving Problems by Searching](../chapter-03-solving-problems-by-searching/README.md)

## Chapter Overview

Extend search beyond classical deterministic fully observable settings. Study [local search](../../GLOSSARY.md#local-search) and optimization landscapes, search in continuous spaces, and algorithms for nondeterministic, partially observable, and online environments including [belief-state](../../GLOSSARY.md#belief-state) search, LRTA*, and exploration strategies.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Implement [hill-climbing](../../GLOSSARY.md#hill-climbing), [simulated annealing](../../GLOSSARY.md#simulated-annealing), and local beam search
2. Analyze local search plateaus, ridges, and random restarts
3. Discretize and search continuous state spaces with gradient methods
4. Model nondeterministic actions with AND-OR search trees
5. Formulate partially observable problems as belief-state search
6. Apply online search agents and learning-real-time A* (LRTA*)
7. Compare offline vs. online search trade-offs

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Local Search](./section-01-local-search.md) | Optimization problems, landscapes, hill climbing, simulated annealing |
| 2 | [Continuous Spaces](./section-02-continuous-spaces.md) | Discretization, gradient descent, linear programming intro |
| 3 | [Nondeterministic Search](./section-03-nondeterministic-search.md) | AND-OR trees, contingency planning, slippery grid |
| 4 | [Partial Observability](./section-04-partial-observability.md) | Belief states, sensorless planning, belief updates |
| 5 | [Online Search](./section-05-online-search.md) | Interleaving action and discovery |
| 6 | [LRTA* and Exploration](./section-06-lrta-and-exploration.md) | Learning real-time A*, exploration costs |
| 7 | [Summary and Tradeoffs](./section-07-summary-and-tradeoffs.md) | Offline vs. online search, memory and optimality |
| 8 | [N-Queens Local Search Lab](./section-08-n-queens-local-search-lab.md) | Hill climbing, simulated annealing, experiments and plots |

---

## Lab / Project

**[Section 4.8 — N-Queens Local Search Lab](./section-08-n-queens-local-search-lab.md):** Solve n-queens with hill-climbing and simulated annealing (n=20+). Plot objective vs. iteration and compare success rates over 100 random restarts.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Gradient-based optimization | Course 3 Chapter 04: deep learning optimization |
| Exploration vs. exploitation | Course 2 Chapter 17: MDPs and bandits formalize this |
| Local minima in training | Course 4 Chapter 02: GAN training dynamics and mode collapse |

---

## Prerequisites

- [Chapter 03 — Solving Problems by Searching](../chapter-03-solving-problems-by-searching/README.md)

---

## Self-Assessment

1. Why does hill-climbing fail on plateaus and ridges?
2. What is the difference between a belief state and a physical state?
3. When is online search preferable to offline planning?
4. Explain how simulated annealing escapes local optima.
5. What makes belief-state search exponentially harder than standard search?
6. Describe the role of the goal test in sensorless planning.

---

**Previous:** [Chapter 03 — Solving Problems by Searching](../chapter-03-solving-problems-by-searching/README.md)  
**Next:** [Chapter 05 — Adversarial Search and Games](../chapter-05-adversarial-search-games/README.md)
