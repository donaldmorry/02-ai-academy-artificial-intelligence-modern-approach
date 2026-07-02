# Chapter 02: Intelligent Agents

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 2
> **Part:** Part I: Artificial Intelligence
> **Estimated time:** 10–12 hours
> **Prerequisites:** Chapter 01: Introduction to AI

---

## Chapter Overview

Introduce agents as the central abstraction of AI: entities that perceive through sensors and act through actuators. Formalize rationality via performance measures, task environments (PEAS), and environment properties. Survey agent architectures from simple reflex agents through goal-based, utility-based, and learning agents.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Specify task environments using PEAS descriptions
2. Classify environments along six dimensions: observable, agents, deterministic, episodic, static, discrete
3. Define rational agent behavior relative to performance measures and prior knowledge
4. Compare reflex, model-based, goal-based, utility-based, and learning agent designs
5. Map environment properties to appropriate agent architectures
6. Design a simple agent program structure for a chosen domain
7. Connect agent taxonomy to techniques covered in later chapters

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Agents and Environments](./section-01-agents-and-environments.md) | Percepts, actions, sensors, actuators, agent function vs. program |
| 2 | [Good Behavior and Rationality](./section-02-good-behavior-and-rationality.md) | Performance measures, rationality, omniscience vs. autonomy |
| 3 | [The Nature of Environments](./section-03-the-nature-of-environments.md) | PEAS, fully/partially observable, multiagent, stochastic |
| 4 | [Structure of Agents](./section-04-structure-of-agents.md) | Reflex, model-based, goal-based, utility-based agents |
| 5 | [Learning Agents](./section-05-learning-agents.md) | Performance element, critic, learning element, problem generator |
| 6 | [Agent Design Patterns](./section-06-agent-design-patterns.md) | Mapping environment classes to architectural choices |
| 7 | [Case Studies](./section-07-case-studies.md) | Vacuum world, taxi driver, medical diagnosis agent |

---

## Lab / Project

Produce a full PEAS specification and agent architecture diagram for a warehouse picking robot. Justify each architectural choice given the environment properties.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Environment classification | Course 1 Chapter 07: operationalizing agents in production |
| Utility-based decisions | Course 2 Chapter 16: formal utility theory |
| Learning agents | Course 3 Chapter 01: agents that learn representations |

---

## Prerequisites

- Chapter 01: Introduction to AI

---

## Self-Assessment

1. Write a PEAS description for an autonomous drone delivery system.
2. Is chess fully or partially observable? Static or dynamic? Justify.
3. What is the difference between a model-based reflex agent and a goal-based agent?
4. Why can a rational agent be ignorant of the true environment dynamics?
5. Which agent type requires explicit utility functions?
6. How does a learning agent's problem generator support exploration?

---

**Previous:** [Chapter 01 — Introduction to Artificial Intelligence](../chapter-01-introduction/README.md)
**Next:** [Chapter 03 — Solving Problems by Searching](../chapter-03-solving-problems-by-searching/README.md)
