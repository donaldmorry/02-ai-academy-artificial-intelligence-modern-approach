# Section 26.5: Mapping and SLAM

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 26.5
> **Previous:** [Section 26.4 - Localization](./section-04-localization.md)
> **Next:** [Section 26.6 - Motion Planning](./section-06-motion-planning.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Introduction

Russell and Norvig integrate **mapping and slam** into embodied agents that perceive, plan, and act in physical environments.

---

## Source-Aligned Summary

This section reworks the source chapter into original course notes for **Mapping and SLAM**. The source book supplies the sequence of ideas; this course version turns that sequence into a student-facing workflow: define the task, choose the representation, identify the algorithmic assumptions, and decide how to evaluate the result.

For **Robotics**, the important habit is to separate three layers:

| Layer | Question to ask | Student deliverable |
|-------|-----------------|---------------------|
| Problem model | What are the states, actions, observations, or symbols? | A precise formulation |
| Inference or control method | What computation transforms inputs into decisions? | Pseudocode or runnable code |
| Evaluation | What evidence would show the method works? | Metrics, traces, and failure analysis |

The section should not be read as a historical artifact. Treat it as an engineering design pattern: identify the simplifying assumptions first, then decide whether the real environment violates them.

> **Readable form:** The source chapter gives the conceptual lineage; this course section translates it into implementable decisions and testable claims.

---

## Motivation

Robots operating in the real world require reliable simultaneous localization and mapping. This section formalizes models and algorithms from Chapter 26.5.

---

## Formal Model

Define state, actions, observations, and transition/sensor models. Connect to MDPs and POMDPs from [Chapter 17](../chapter-17-making-complex-decisions/README.md).

---

## Algorithms

Survey classical and modern implementations; compare sample efficiency, completeness, and assumptions.

---

## Robot Examples

Warehouse AMRs, surgical arms, and domestic assistants illustrate design trade-offs in sensors, planning horizon, and safety.

---

$$
P(\mathbf{x}_{1:t}, \mathbf{m} \mid \mathbf{z}_{1:t}, \mathbf{u}_{1:t})
$$
> **Readable form:** Joint posterior over trajectory and map.

---

```python
def log_odds_update(cell, inverse_sensor_log_odds):
    return cell + inverse_sensor_log_odds
```
---

## Summary

| Component | Role in Mapping and SLAM |
|-----------|----------------|
| State | Configuration |
| Action | Motor commands |
| Observation | Sensor readings |
---

## Pitfalls

- Ignoring dynamics and delays
- Sim-to-real gap
- Insufficient safety margins

---

## Connections

- [Chapter 03](../chapter-03-solving-problems-by-searching/README.md)
- [Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)
- [Chapter 25](../chapter-25-computer-vision/README.md)

---

## Worked Example

Walk through a toy instance of **Mapping and SLAM** by hand: specify inputs, apply the main formula or algorithm, and interpret outputs. Compare with a naive baseline.

---

## Implementation Notes

Separate data loading, model/planning core, and evaluation. Log random seeds, versions, and hardware. Save artifacts for reproducibility.

---

## Historical Context

Early AI treated subfields independently; modern agents integrate perception, reasoning, learning, and language under the PEAS framework ([Chapter 02](../chapter-02-intelligent-agents/README.md)).

---

## Deployment Checklist

Before production: document failure modes, monitor drift, define rollback, and complete an ethics review for affected stakeholders.

---

## Lab Extension

Extend the Chapter lab with an ablation removing one component of **mapping and slam** and measure impact on your chosen metric.

---

---

## Applied Deep Dive

Use **Mapping and SLAM** as a complete AIMA-style design exercise rather than a vocabulary item.

### Formulation Pass

1. Name the agent, environment, sensors, actuators, and performance measure.
2. State what is fully observable and what must be inferred.
3. Identify whether the task is episodic, sequential, deterministic, stochastic, single-agent, or multi-agent.
4. List the abstraction boundaries where the model deliberately ignores detail.

### Algorithm Pass

| Step | What to verify |
|------|----------------|
| Inputs | Units, coordinate frames, symbol vocabulary, or probability semantics |
| Update | Whether the computation preserves invariants such as normalization or consistency |
| Output | Whether the result is an action, belief state, plan, explanation, or metric |
| Complexity | What grows with states, actions, objects, observations, or time horizon |


### Minimal Implementation Sketch

The goal of this snippet is not a full production system. It is a shape check: does the computation expose the variables the section claims matter?

```python
def log_odds_update(cell, inverse_sensor_log_odds):
    return cell + inverse_sensor_log_odds
```

Run the snippet with at least three hand-chosen inputs, including one edge case, and write down whether the output matches your physical or logical intuition.

### Failure Analysis

| Failure mode | Why it matters | Mitigation |
|--------------|----------------|------------|
| Wrong abstraction | The algorithm solves a clean problem, not the deployed one | Revisit the PEAS description and task environment |
| Hidden uncertainty | Deterministic code overcommits to noisy inputs | Track beliefs, confidence intervals, or fallback policies |
| Poor evaluation | A demo succeeds while edge cases fail | Use held-out scenarios and adversarial tests |
| Integration drift | Perception, planning, and control assumptions disagree | Log interfaces and validate each boundary |

### Mastery Task

Write a one-page technical memo for **Mapping and SLAM**. Include the formal model, one algorithm choice, one rejected alternative, two measurable risks, and a test plan. This turns the source chapter from reading material into engineering judgment.

## Key Terms (Glossary)

- **occupancy grid** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **SLAM** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **FastSLAM** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the role of simultaneous localization and mapping in a mobile manipulator?
2. How does uncertainty enter the model?
3. Name two failure modes.
4. Compare classical vs learning-based approaches.
5. What safety standard would you cite?
6. How does mapping and slam support rational agency?

---

**Previous:** [Section 26.4 - Localization](./section-04-localization.md) · **Next:** [Section 26.6 - Motion Planning](./section-06-motion-planning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/) Chapter 26.5
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Thrun, S., Burgard, W., & Fox, D. (2005). *Probabilistic Robotics*.




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
