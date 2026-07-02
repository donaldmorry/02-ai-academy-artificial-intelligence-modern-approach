# Chapter 11: Automated Planning

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11
> **Part:** Part III: Knowledge, Reasoning & Planning
> **Estimated time:** 12–14 hours
> **Prerequisites:** Chapter 03: Solving Problems by Searching; Chapter 09: Inference in First-Order Logic

---

## Chapter Overview

Formalize classical planning with STRIPS, PDDL, forward and backward state-space search, planning graphs, and SAT-based planning. Extend to hierarchical task networks, scheduling, multiagent planning, and integration with acting in uncertain environments.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Represent planning domains with states, actions, preconditions, and effects
2. Implement forward state-space and goal regression planners
3. Apply GRAPHPLAN and analyze its planning-graph structure
4. Encode planning as SAT and solve with modern SAT solvers
5. Describe hierarchical planning with HTNs and decomposition methods
6. Model resource constraints and scheduling in planning problems
7. Connect classical planning assumptions to real-world robotics needs

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Planning Problem Definition](./section-01-planning-problem-definition.md) | Initial state, goal, actions, STRIPS representation |
| 2 | [State-Space Planning](./section-02-state-space-planning.md) | Forward search, heuristics for planning |
| 3 | [Planning Graphs](./section-03-planning-graphs.md) | GRAPHPLAN, mutex relations, solution extraction |
| 4 | [SAT Planning](./section-04-sat-planning.md) | CNF encoding, bounded planning horizon |
| 5 | [Hierarchical Planning](./section-05-hierarchical-planning.md) | HTNs, methods, decomposition |
| 6 | [Scheduling and Resources](./section-06-scheduling-and-resources.md) | Time, resources, constraint-based scheduling |
| 7 | [Multiagent and Contingent Planning](./section-07-multiagent-and-contingent-planning.md) | Joint plans, sensing actions preview |
| 8 | [PDDL and Practical Systems](./section-08-pddl-and-practical-systems.md) | Fast-Downward, domain engineering |

---

## Lab / Project

Specify a blocks-world domain in PDDL and solve with an off-the-shelf planner. Compare plan length and search time for two heuristics if available.

Hands-on lab: [Lab 11 - STRIPS Planning in Blocks World](./section-lab-11-strips-planning-in-blocks-world.md).

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Search algorithms | Course 2 Chapter 03: planning as heuristic search |
| Robot task planning | Course 2 Chapter 26: motion and task planning integration |
| LLM agents | Course 2 Chapter 28: language models as planners |

---

## Prerequisites

- Chapter 03: Solving Problems by Searching
- Chapter 09: Inference in First-Order Logic

---

## Self-Assessment

1. What are the components of a STRIPS action?
2. How does GRAPHPLAN detect when no plan exists at a given length?
3. Why is planning often easier in backward (regression) form for some domains?
4. What is an HTN method and how does it differ from STRIPS actions?
5. How can planning be reduced to SAT solving?
6. What classical planning assumptions break in robotics?

---

**Previous:** [Chapter 10 — Knowledge Representation](../chapter-10-knowledge-representation/README.md)
**Next:** [Chapter 12 — Quantifying Uncertainty](../chapter-12-quantifying-uncertainty/README.md)
