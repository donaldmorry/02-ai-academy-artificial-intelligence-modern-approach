# Chapter 05: Adversarial Search and Games

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 5
> **Part:** Part II: Problem-Solving
> **Estimated time:** 12–14 hours
> **Prerequisites:** Chapter 03: Solving Problems by Searching; Chapter 04: Search in Complex Environments (recommended)

---

## Chapter Overview

Model competitive multiagent settings as game trees. Develop minimax decision-making, alpha-beta pruning, and evaluation functions for intractable games. Study stochastic games, expectiminimax, and Monte Carlo Tree Search (MCTS) as used in modern game engines and Go programs.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Formulate two-player zero-sum games with game trees and utility values
2. Implement minimax and verify correctness on small game trees
3. Apply alpha-beta pruning while preserving minimax outcomes
4. Design evaluation functions and cutoff strategies for large games
5. Extend to stochastic games with expectiminimax
6. Explain MCTS phases: selection, expansion, simulation, backpropagation
7. Connect game-playing advances to real-time decision systems

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Games and Game Trees](./section-01-games-and-game-trees.md) | Zero-sum, perfect information, utility at terminal states |
| 2 | [Minimax](./section-02-minimax.md) | Optimal decisions assuming rational opponent |
| 3 | [Alpha-Beta Pruning](./section-03-alpha-beta-pruning.md) | Pruning conditions, move ordering, effective branching factor |
| 4 | [Imperfect Decisions](./section-04-imperfect-decisions.md) | Evaluation functions, quiescence search, horizon effect |
| 5 | [Stochastic Games](./section-05-stochastic-games.md) | Expectiminimax, dice, backgammon |
| 6 | [Monte Carlo Tree Search](./section-06-monte-carlo-tree-search.md) | UCT, rollouts, AlphaGo-style architectures |
| 7 | [Partial Observability in Games](./section-07-partial-observability-in-games.md) | Card games, belief tracking overview |
| 8 | [Game Theory Preview](./section-08-game-theory-preview.md) | Normal-form games, Nash equilibrium introduction |

---

## Lab / Project

Build a minimax with alpha-beta connect-four agent. Add depth-limited search with a simple evaluation function and measure nodes pruned vs. plain minimax.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Tree search fundamentals | Course 2 Chapter 03: informed search heuristics analog |
| MCTS in RL | Course 2 Chapter 22: RL combines search and learning |
| Self-play training | Course 4 Chapter 08: generative adversarial training parallels |

---

## Prerequisites

- Chapter 03: Solving Problems by Searching
- Chapter 04: Search in Complex Environments (recommended)

---

## Self-Assessment

1. What assumption does minimax make about the opponent?
2. How does alpha-beta pruning affect the final minimax value?
3. Why are evaluation functions necessary in chess-like games?
4. Contrast expectiminimax with standard minimax.
5. List the four steps of each MCTS iteration.
6. What is the horizon effect and how might search mitigate it?

---

**Previous:** [Chapter 04 — Search in Complex Environments](../chapter-04-search-complex-environments/README.md)
**Next:** [Chapter 06 — Constraint Satisfaction Problems](../chapter-06-constraint-satisfaction/README.md)
