# Section 28.8: Capstone Preparation

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 28.8
> **Previous:** [Section 28.7 - Future Scenarios](./section-07-future-scenarios.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Introduction

**Capstone Preparation** synthesizes AI techniques toward integrated intelligent systems and research frontiers.

---

## Source-Aligned Summary

This section reworks the source chapter into original course notes for **Capstone Preparation**. The source book supplies the sequence of ideas; this course version turns that sequence into a student-facing workflow: define the task, choose the representation, identify the algorithmic assumptions, and decide how to evaluate the result.

For **Future Of Ai**, the important habit is to separate three layers:

| Layer | Question to ask | Student deliverable |
|-------|-----------------|---------------------|
| Problem model | What are the states, actions, observations, or symbols? | A precise formulation |
| Inference or control method | What computation transforms inputs into decisions? | Pseudocode or runnable code |
| Evaluation | What evidence would show the method works? | Metrics, traces, and failure analysis |

The section should not be read as a historical artifact. Treat it as an engineering design pattern: identify the simplifying assumptions first, then decide whether the real environment violates them.

> **Readable form:** The source chapter gives the conceptual lineage; this course section translates it into implementable decisions and testable claims.

---

## Components of Intelligence

Perception, reasoning, learning, language, and action interact in complete agents - no single chapter suffices.

---

## Integration Patterns

Pipelines, blackboard architectures, and end-to-end learning represent design choices with different debuggability.

---

## Emerging Directions

Neuro-symbolic methods, world models, and LLM-based agents combine strengths of classical AI and deep learning.

---

## Capstone Relevance

Select techniques from at least three course parts; justify choices with PEAS analysis and evaluation plan.

---

$$
\text{Agent} = \langle \text{Percepts}, \text{KB}, \text{Planner}, \text{Learner}, \text{Actuators} \rangle
$$
> **Readable form:** A complete architecture couples chapters with shared state and goals.

---

```python
class CapstoneAgent:
    def __init__(self, searcher, learner, reasoner):
        self.searcher, self.learner, self.reasoner = searcher, learner, reasoner
    def act(self, percept):
        belief = self.reasoner.update(percept)
        plan = self.searcher.plan(belief)
        return plan[0]
```
---

## Summary

| Open problem | Why hard? |
|--------------|----------|
| Common sense | Coverage, compositionality |
| Causality | Intervention vs correlation |
| Continual learning | Catastrophic forgetting |
---

## Pitfalls

- Overclaiming AGI timelines
- Monolithic models without verification
- Neglecting ethics section in capstone

---

## Connections

- [Chapter 02](../chapter-02-intelligent-agents/README.md)
- [Chapter 24](../chapter-24-deep-learning-nlp/README.md)
- [Chapter 27](../chapter-27-philosophy-ethics-safety/README.md)

---

## Worked Example

Walk through a toy instance of **Capstone Preparation** by hand: specify inputs, apply the main formula or algorithm, and interpret outputs. Compare with a naive baseline.

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

Extend the Chapter lab with an ablation removing one component of **capstone preparation** and measure impact on your chosen metric.

---

---

## Applied Deep Dive

Use **Capstone Preparation** as a complete AIMA-style design exercise rather than a vocabulary item.

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



### Failure Analysis

| Failure mode | Why it matters | Mitigation |
|--------------|----------------|------------|
| Wrong abstraction | The algorithm solves a clean problem, not the deployed one | Revisit the PEAS description and task environment |
| Hidden uncertainty | Deterministic code overcommits to noisy inputs | Track beliefs, confidence intervals, or fallback policies |
| Poor evaluation | A demo succeeds while edge cases fail | Use held-out scenarios and adversarial tests |
| Integration drift | Perception, planning, and control assumptions disagree | Log interfaces and validate each boundary |

### Mastery Task

Write a one-page technical memo for **Capstone Preparation**. Include the formal model, one algorithm choice, one rejected alternative, two measurable risks, and a test plan. This turns the source chapter from reading material into engineering judgment.

## Key Terms (Glossary)

- **PEAS** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **technique selection** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **evaluation plan** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. List five agent components.
2. What is neuro-symbolic integration?
3. Define a world model in RL.
4. How can LLMs use tools?
5. What belongs in a capstone ethics section?
6. How does capstone preparation support rational agency?

---

**Previous:** [Section 28.7 - Future Scenarios](./section-07-future-scenarios.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/) Chapter 28.8
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Marcus, G. (2020). The next decade in AI: four steps towards robust AI. *arXiv*.




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
