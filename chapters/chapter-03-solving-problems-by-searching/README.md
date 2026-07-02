# Chapter 03: Solving Problems by Searching

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 3  
> **Part:** Part II: Problem-Solving  
> **Estimated time:** 12–14 hours  
> **Prerequisites:** [Chapter 02 — Intelligent Agents](../chapter-02-intelligent-agents/README.md)

## Chapter Overview

Equip problem-solving agents with systematic search. Formalize problems as initial states, actions, transition models, goal tests, and path costs. Implement uninformed strategies (BFS, DFS, UCS, iterative deepening) and informed strategies (greedy best-first, A*) with [admissible](../../GLOSSARY.md#admissible-heuristic) and consistent [heuristics](../../GLOSSARY.md#heuristic). Analyze completeness, optimality, and complexity. Complete a hands-on Romania [A*](../../GLOSSARY.md#a-star-search) lab.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Formulate search problems from informal task descriptions using the five-tuple and [PEAS](../../GLOSSARY.md#peas)
2. Implement breadth-first, depth-first, uniform-cost, and A* search in Python
3. Explain iterative deepening and when it is preferable to BFS or DFS
4. Design admissible heuristics using relaxed problems, Manhattan distance, and straight-line distance
5. Prove A* optimality given admissible and consistent heuristics
6. Compare search algorithms on completeness, optimality, time, and space
7. Apply search to the Romania map, 8-puzzle, and route-planning agents

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Problem-Solving Agents](./section-01-problem-solving-agents.md) | Goal formulation, five-tuple, goal-based agents, PEAS |
| 2 | [Example Problems](./section-02-example-problems.md) | Romania map, 8-puzzle, vacuum world, state-space size |
| 3 | [Uninformed Search](./section-03-uninformed-search.md) | BFS, DFS, UCS, depth limits, iterative deepening |
| 4 | [Informed Search](./section-04-informed-search.md) | Greedy best-first, A*, admissibility, consistency |
| 5 | [Heuristic Functions](./section-05-heuristic-functions.md) | Designing $h(n)$, dominance, relaxed problems, pattern DBs |
| 6 | [Search Implementations](./section-06-search-implementations.md) | Python Problem/Node classes, BFS, UCS, A* |
| 7 | [Search Analysis](./section-07-search-analysis.md) | Complexity, completeness, optimality, effective branching factor |
| 8 | [Romania A* Lab](./section-08-romania-a-lab.md) | Full A* walkthrough, UCS/greedy comparison, PEAS report |

---

## Lab / Project

**[Section 3.8 — Romania A* Lab](./section-08-romania-a-lab.md):** Implement and trace A* on the AIMA Romania map from Arad to Bucharest. Verify optimal cost **418 km**, compare nodes expanded against UCS and greedy best-first, and write a PEAS specification for the route-finding agent. Optional extension: 8-puzzle with Manhattan distance.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Goal-based agents & PEAS | [Chapter 02](../chapter-02-intelligent-agents/README.md) |
| Graph algorithms intuition | Course 1 Chapter 01: algorithmic thinking for ML pipelines |
| Local & nondeterministic search | [Chapter 04](../chapter-04-search-complex-environments/README.md) |
| Heuristic search at scale | [Chapter 05](../chapter-05-adversarial-search-games/README.md): game-tree search |
| Planning as search | [Chapter 11](../chapter-11-automated-planning/README.md): automated planning |

---

## Prerequisites

- [Chapter 02 — Intelligent Agents](../chapter-02-intelligent-agents/README.md) (especially Sections 2.3–2.4 on PEAS and goal-based agents)

---

## Self-Assessment

1. What five components define a search problem?
2. When is uniform-cost search identical to Dijkstra's algorithm?
3. State the conditions for A* optimality and define consistency.
4. Why is iterative deepening search useful despite repeated work?
5. Give an admissible heuristic for the 8-puzzle and justify admissibility.
6. Compare space complexity of BFS vs. DFS on a tree with branching factor $b$ and depth $d$.
7. Trace $f = g + h$ for the first three A* expansions from Arad.

---

**Previous:** [Chapter 02 — Intelligent Agents](../chapter-02-intelligent-agents/README.md)  
**Next:** [Chapter 04 — Search in Complex Environments](../chapter-04-search-complex-environments/README.md)
