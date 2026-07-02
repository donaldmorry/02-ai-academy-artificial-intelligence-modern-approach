# Chapter 07: Logical Agents

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 7
> **Part:** Part III: Knowledge, Reasoning & Planning
> **Estimated time:** 12–14 hours
> **Prerequisites:** Chapter 02: Intelligent Agents; Chapter 06: Constraint Satisfaction Problems

---

## Chapter Overview

Build knowledge-based agents that represent world states in propositional logic, reason via inference, and act safely in partially known environments. Use the Wumpus World as a running example for knowledge bases, entailment, model checking, and theorem-proving approaches including resolution and forward/backward chaining.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Distinguish knowledge bases, entailment, and inference in agent design
2. Write propositional logic sentences using syntax and semantics
3. Apply truth tables and model checking for entailment
4. Convert sentences to conjunctive normal form for resolution
5. Implement resolution refutation and understand completeness
6. Apply forward and backward chaining for Horn clauses
7. Design a Wumpus World agent policy using logical inference

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Knowledge-Based Agents](./section-01-knowledge-based-agents.md) | KB, TELL, ASK, declarative vs. procedural |
| 2 | [Propositional Logic](./section-02-propositional-logic.md) | Syntax, semantics, models, satisfiability |
| 3 | [Inference and Entailment](./section-03-inference-and-entailment.md) | Soundness, completeness, validity |
| 4 | [Theorem Proving](./section-04-theorem-proving.md) | Resolution, CNF conversion, refutation proofs |
| 5 | [Forward Chaining](./section-05-forward-chaining.md) | Horn clauses, data-driven inference |
| 6 | [Backward Chaining](./section-06-backward-chaining.md) | Goal-driven inference, proof trees |
| 7 | [Wumpus World](./section-07-wumpus-world.md) | Percepts, actions, safe vs. risky squares |
| 8 | [Logic Limitations Preview](./section-08-logic-limitations-preview.md) | Expressiveness gap motivating first-order logic |

---

## Lab / Project

Implement a propositional inference engine (resolution or forward chaining) and use it to decide safe moves in a small Wumpus World grid.

Hands-on lab: [Lab 07 - Wumpus World Inference Agent](./section-lab-07-wumpus-world-inference-agent.md).

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Rule-based systems | Course 1 Chapter 04: text classification feature logic |
| First-order extension | Course 2 Chapter 08: FOL generalizes propositional logic |
| Knowledge graphs | Course 2 Chapter 10: ontologies and structured knowledge |

---

## Prerequisites

- Chapter 02: Intelligent Agents
- Chapter 06: Constraint Satisfaction Problems

---

## Self-Assessment

1. What is the difference between entailment and logical equivalence?
2. When is resolution refutation complete for propositional logic?
3. Why are Horn clauses important for efficient inference?
4. Describe what a Wumpus World agent knows after perceiving Stench at (1,1).
5. What is the difference between forward and backward chaining?
6. Give an example sentence that is satisfiable but not valid.

---

**Previous:** [Chapter 06 — Constraint Satisfaction Problems](../chapter-06-constraint-satisfaction/README.md)
**Next:** [Chapter 08 — First-Order Logic](../chapter-08-first-order-logic/README.md)
