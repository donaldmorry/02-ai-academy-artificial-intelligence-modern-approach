# Artificial Intelligence: A Modern Approach - Academy Course

<p align="center">
  <img src="./assets/course-symbol.png" alt="Symbolic banner for rational agents, search, logic, planning, uncertainty, and AI safety" width="100%">
</p>

A rigorous academy-style course in classical and modern artificial intelligence: rational agents, search, games, logic, planning, probability, decision theory, learning, language, perception, robotics, ethics, safety, and future AI systems.

This repository follows the intellectual structure of Russell and Norvig's *Artificial Intelligence: A Modern Approach* and turns it into a navigable course with chapters, sections, labs, capstone work, glossary support, and assessment prompts. The goal is not only to know AI techniques, but to learn how to choose representations, reason about assumptions, and evaluate intelligent behavior.

## Who This Course Is For

- Learners who already have basic ML experience and want the full AI map.
- Engineers who need search, planning, uncertainty, logic, and decision theory.
- Graduate-level self-study groups using AIMA as the central reference.
- Researchers and builders who want a foundation for agentic systems and AI safety.

You should know Python and basic probability. Linear algebra and deeper probability are reviewed in the appendices and connected to Course 3.

## What You Will Be Able To Do

By the end of the course, you will be able to:

1. Specify agents with PEAS descriptions and task-environment properties.
2. Implement uninformed, informed, local, adversarial, and constraint-based search.
3. Use propositional logic, first-order logic, and knowledge representation patterns.
4. Build classical planners and reason about action, time, and resources.
5. Model uncertainty with Bayesian networks, temporal models, and probabilistic programs.
6. Solve decision problems with utility theory, MDPs, POMDPs, and bandits.
7. Connect learning theory, neural networks, NLP, vision, robotics, and reinforcement learning.
8. Analyze AI ethics, governance, safety cases, specification gaming, and alignment risks.
9. Design a capstone intelligent agent with implementation, evaluation, and safety arguments.

## Curriculum Map

| Chapter | Focus | Sections |
|---------|-------|----------|
| [01](./chapters/chapter-01-introduction/README.md) | Introduction to Artificial Intelligence | 6 |
| [02](./chapters/chapter-02-intelligent-agents/README.md) | Intelligent Agents | 7 |
| [03](./chapters/chapter-03-solving-problems-by-searching/README.md) | Solving Problems by Searching | 8 |
| [04](./chapters/chapter-04-search-complex-environments/README.md) | Search in Complex Environments | 8 |
| [05](./chapters/chapter-05-adversarial-search-games/README.md) | Adversarial Search and Games | 9 |
| [06](./chapters/chapter-06-constraint-satisfaction/README.md) | Constraint Satisfaction Problems | 8 |
| [07](./chapters/chapter-07-logical-agents/README.md) | Logical Agents | 9 |
| [08](./chapters/chapter-08-first-order-logic/README.md) | First-Order Logic | 7 |
| [09](./chapters/chapter-09-inference-first-order-logic/README.md) | Inference in First-Order Logic | 8 |
| [10](./chapters/chapter-10-knowledge-representation/README.md) | Knowledge Representation | 8 |
| [11](./chapters/chapter-11-automated-planning/README.md) | Automated Planning | 9 |
| [12](./chapters/chapter-12-quantifying-uncertainty/README.md) | Quantifying Uncertainty | 7 |
| [13](./chapters/chapter-13-probabilistic-reasoning/README.md) | Probabilistic Reasoning | 9 |
| [14](./chapters/chapter-14-probabilistic-reasoning-time/README.md) | Probabilistic Reasoning over Time | 9 |
| [15](./chapters/chapter-15-probabilistic-programming/README.md) | Probabilistic Programming | 8 |
| [16](./chapters/chapter-16-making-simple-decisions/README.md) | Making Simple Decisions | 7 |
| [17](./chapters/chapter-17-making-complex-decisions/README.md) | Making Complex Decisions | 9 |
| [18](./chapters/chapter-18-multiagent-decision-making/README.md) | Multiagent Decision Making | 7 |
| [19](./chapters/chapter-19-learning-from-examples/README.md) | Learning from Examples | 8 |
| [20](./chapters/chapter-20-learning-probabilistic-models/README.md) | Learning Probabilistic Models | 8 |
| [21](./chapters/chapter-21-deep-learning/README.md) | Deep Learning | 8 |
| [22](./chapters/chapter-22-reinforcement-learning/README.md) | Reinforcement Learning | 9 |
| [23](./chapters/chapter-23-natural-language-processing/README.md) | Natural Language Processing | 8 |
| [24](./chapters/chapter-24-deep-learning-nlp/README.md) | Deep Learning for Natural Language Processing | 8 |
| [25](./chapters/chapter-25-computer-vision/README.md) | Computer Vision | 8 |
| [26](./chapters/chapter-26-robotics/README.md) | Robotics | 8 |
| [27](./chapters/chapter-27-philosophy-ethics-safety/README.md) | Philosophy, Ethics, and Safety | 8 |
| [28](./chapters/chapter-28-future-of-ai/README.md) | The Future of Artificial Intelligence | 8 |
| [A](./chapters/chapter-a-mathematical-background/README.md) | Mathematical Background | 8 |
| [B](./chapters/chapter-b-languages-algorithms/README.md) | Languages and Algorithms | 7 |

## Hands-On Labs

This course includes labs across the AI stack: Romania A*, N-Queens local search, Connect Four, Sudoku CSP, Wumpus World inference, STRIPS planning, Bayesian network inference, temporal probabilistic reasoning, probabilistic programming, value iteration, and tabular Q-learning. The labs are deliberately small enough to finish, but formal enough to expose the assumptions behind each method.

## How To Study

Treat each chapter as a system design exercise. For every algorithm, ask four questions:

1. What representation does it require?
2. What assumptions make the computation valid?
3. What evidence shows it worked?
4. What breaks when the real environment violates the assumptions?

A strong study session includes a worked example, a small implementation, and one failure case.

## Capstone

The capstone is an intelligent-agent design project. You will choose a domain, define the task environment, combine at least three AI techniques, evaluate behavior, and write an ethics and safety review. See [the capstone specification](./projects/capstone/README.md).

## Repository Guide

| Path | Purpose |
|------|---------|
| `chapters/` | Course chapters and section files |
| `projects/` | Capstone project specification and rubrics |
| `resources/` | Supplementary references and study aids |
| `assets/` | Repository banner and visual assets |
| `GLOSSARY.md` | Shared technical vocabulary |
| `MATH_CONVENTIONS.md` | Math formatting conventions |
| `ASSESSMENT_APPENDIX.md` | Reusable mastery prompts and review templates |

## Source And Attribution

This is an independent educational curriculum inspired by Stuart Russell and Peter Norvig's *Artificial Intelligence: A Modern Approach*. It is not a replacement for the book. Use the original text for authoritative exposition and this repository for structured study, labs, review, and project work.

## Start Here

Begin with [Chapter 01: Introduction to Artificial Intelligence](./chapters/chapter-01-introduction/README.md).
