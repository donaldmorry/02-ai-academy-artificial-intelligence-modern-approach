# Section A.2: Linear Algebra Basics

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Appendix A.2
> **Previous:** [Section A.1 - Complexity Analysis](./section-01-complexity-analysis.md)
> **Next:** [Section A.3 - Advanced Linear Algebra](./section-03-advanced-linear-algebra.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Introduction

**Linear Algebra Basics** provides mathematical tools referenced throughout AIMA and Courses 2-4.

---

## Source-Aligned Summary

This section reworks the source chapter into original course notes for **Linear Algebra Basics**. The source book supplies the sequence of ideas; this course version turns that sequence into a student-facing workflow: define the task, choose the representation, identify the algorithmic assumptions, and decide how to evaluate the result.

For **A Mathematical Background**, the important habit is to separate three layers:

| Layer | Question to ask | Student deliverable |
|-------|-----------------|---------------------|
| Problem model | What are the states, actions, observations, or symbols? | A precise formulation |
| Inference or control method | What computation transforms inputs into decisions? | Pseudocode or runnable code |
| Evaluation | What evidence would show the method works? | Metrics, traces, and failure analysis |

The section should not be read as a historical artifact. Treat it as an engineering design pattern: identify the simplifying assumptions first, then decide whether the real environment violates them.

> **Readable form:** The source chapter gives the conceptual lineage; this course section translates it into implementable decisions and testable claims.

---

## Definitions

State formal definitions with notation consistent with [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md).

---

## Worked Examples

Apply definitions to small numeric instances - 2×2 matrices, Bernoulli trials, gradient of quadratic loss.

---

## AI Connections

Link concepts to search complexity, neural network linear algebra, probabilistic inference, and optimization.

---

## Study Tips

Identify personal gaps; cross-reference [Course 3 Chapter 02](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-02-linear-algebra/README.md) for deeper treatment.

---

$$
O(g(n)) = \{f: \exists c,n_0.\; \forall n\ge n_0.\; 0 \le f(n) \le c\,g(n)\}
$$
> **Readable form:** Big-O bounds asymptotic growth from below up to constant factors.

---

```python
import numpy as np

def gradient_quadratic(w):
  # f(w) = w^T w
    return 2 * w
```
---

## Summary

| Symbol | Meaning |
|--------|----------|
| $\mathbb{E}[X]$ | Expected value |
| $
abla f$ | Gradient |
---

## Pitfalls

- Confusing O, Θ, Ω
- Dimension mismatches in matrix multiply
- Treating correlated variables as independent

---

## Connections

- [Chapter 03](../chapter-03-solving-problems-by-searching/README.md)
- [Chapter 12](../chapter-12-quantifying-uncertainty/README.md)
- [Chapter 21](../chapter-21-deep-learning/README.md)

---

## Worked Example

Walk through a toy instance of **Linear Algebra Basics** by hand: specify inputs, apply the main formula or algorithm, and interpret outputs. Compare with a naive baseline.

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

Extend the Chapter lab with an ablation removing one component of **linear algebra basics** and measure impact on your chosen metric.

---

---

## Applied Deep Dive

Use **Linear Algebra Basics** as a complete AIMA-style design exercise rather than a vocabulary item.

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

Write a one-page technical memo for **Linear Algebra Basics**. Include the formal model, one algorithm choice, one rejected alternative, two measurable risks, and a test plan. This turns the source chapter from reading material into engineering judgment.

## Key Terms (Glossary)

- **vector** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **matrix** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **dot product** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. State the axioms of probability.
2. Compute derivative of $w^Tw$.
3. What is NP-completeness?
4. Define eigenvalue.
5. When is SVD used in ML?
6. How does linear algebra basics support rational agency?

---

**Previous:** [Section A.1 - Complexity Analysis](./section-01-complexity-analysis.md) · **Next:** [Section A.3 - Advanced Linear Algebra](./section-03-advanced-linear-algebra.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/) Appendix A.2
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Strang, G. (2016). *Introduction to Linear Algebra*.




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
