# Capstone Project: Intelligent Agent for a Complex Domain

> **Course:** 2 — Artificial Intelligence: A Modern Approach  
> **When:** After completing Chapters 01–28 and appendices as needed  
> **Estimated time:** 20–35 hours

---

## Project Overview

Demonstrate mastery of Course 2 by designing, implementing, and evaluating an **intelligent agent** that combines at least three AIMA techniques. The project should show formal problem formulation, working algorithms, uncertainty or planning, and ethical analysis.

---

## Requirements

### 1. Domain Selection

Choose one domain:

- **Healthcare triage:** prioritize cases under uncertainty
- **Warehouse robotics:** plan routes and resolve constraints
- **Game-playing:** search and adversarial decision-making
- **Smart home:** reason about events, defaults, and actions
- **Custom:** a domain with explicit agent/environment boundaries

### 2. Technical Requirements

| Component | Requirement |
|-----------|-------------|
| PEAS formulation | Performance, environment, actuators, sensors |
| Search or planning | A*, CSP, alpha-beta, MCTS, STRIPS, or HTN |
| Knowledge representation | Logic, ontology, rules, or probabilistic graph |
| Uncertainty or decisions | Bayesian inference, MDP/POMDP, utility, or bandits |
| Learning component | Supervised, unsupervised, or reinforcement-learning baseline |
| Evaluation | Metrics, ablations, failure cases, and complexity notes |
| Ethics | Safety, misuse, bias, privacy, or human oversight analysis |

### 3. Deliverables

```text
capstone/
├── README.md              # Domain, PEAS, architecture, results
├── notebooks/             # Experiments and traces
├── src/                   # Agent implementation
├── tests/                 # Unit tests for core algorithms
├── data/                  # Small reproducible toy data or configs
└── report.md              # Evaluation, limitations, ethics
```

---

## Evaluation Rubric

| Criterion | Weight | Excellent |
|-----------|--------|-----------|
| Problem formulation | 15% | Clear PEAS, state space, assumptions, and performance measure |
| Algorithmic integration | 25% | Multiple AIMA methods work together coherently |
| Correctness evidence | 20% | Tests, traces, hand checks, and baseline comparisons |
| Uncertainty/decision quality | 15% | Probabilistic or utility reasoning is explicit and justified |
| Ethics and safety | 10% | Concrete risks with mitigations and monitoring |
| Documentation | 10% | Reproducible, readable, honest about limits |
| Presentation | 5% | Can explain the agent to both technical and nontechnical audiences |

---

## Milestone: AI Generalist

Completing this capstone plus Course 2 chapters earns the **AI Generalist** milestone and qualifies you to begin the deep-learning theory course with a full agent-centered mental model.
