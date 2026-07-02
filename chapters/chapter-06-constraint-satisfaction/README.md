# Chapter 06: Constraint Satisfaction Problems

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 6
> **Part:** Part II: Problem-Solving
> **Estimated time:** 10–12 hours
> **Prerequisites:** Chapter 03: Solving Problems by Searching

---

## Chapter Overview

Formulate problems as CSPs with variables, domains, and constraints. Master backtracking search enhanced by constraint propagation (forward checking, arc consistency / AC-3), heuristic variable and value ordering, and structure exploitation via tree decomposition and cutset conditioning.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Model real-world scheduling and allocation tasks as CSPs
2. Implement backtracking search for CSPs
3. Apply forward checking and maintain arc consistency with AC-3
4. Use MRV, degree, and LCV heuristics to reduce search effort
5. Exploit problem structure: independent subproblems, tree CSPs, cutset conditioning
6. Handle soft constraints and Max-CSP formulations
7. Relate CSP techniques to SAT solving and planning

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [CSP Formulation](./section-01-csp-formulation.md) | Variables, domains, constraints, binary vs. global constraints |
| 2 | [Backtracking Search](./section-02-backtracking-search.md) | Naive backtracking, constraint graphs |
| 3 | [Constraint Propagation](./section-03-constraint-propagation.md) | Forward checking, arc consistency, AC-3 |
| 4 | [Heuristics for CSP](./section-04-heuristics-for-csp.md) | MRV, degree heuristic, least-constraining-value |
| 5 | [Structure in CSPs](./section-05-csp-structure.md) | Tree algorithms, cutset conditioning, constraint graphs |
| 6 | [Local Search for CSPs](./section-06-local-search-for-csps.md) | Min-conflicts heuristic, large-scale CSPs |
| 7 | [Applications](./section-07-csp-applications.md) | Sudoku, timetabling, map coloring, resource allocation |

---

## Lab / Project

Implement backtracking with AC-3 and MRV/LCV for map coloring and Sudoku. Compare node counts against naive backtracking on increasing puzzle sizes.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Combinatorial optimization | Course 1 Chapter 03: classification feature selection analogies |
| SAT and planning | Course 2 Chapter 11: planning as CSP/SAT |
| Graphical model structure | Course 2 Chapter 13: Bayesian network topology |

---

## Prerequisites

- Chapter 03: Solving Problems by Searching

---

## Self-Assessment

1. What is the difference between a constraint and a goal test in search?
2. How does forward checking differ from arc consistency?
3. When does the MRV heuristic help most?
4. What property makes tree-structured CSPs solvable in linear time?
5. Explain the min-conflicts local search algorithm.
6. Give a real-world problem that maps naturally to a CSP.

---

**Previous:** [Chapter 05 — Adversarial Search and Games](../chapter-05-adversarial-search-games/README.md)
**Next:** [Chapter 07 — Logical Agents](../chapter-07-logical-agents/README.md)
