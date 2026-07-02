# Section 07.5: Forward Chaining

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Logical Agents
> **Enhanced with:** original explanations, worked checks, implementation guidance, diagnostics, and mastery prompts
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Core Idea

**Forward Chaining** is part of AIMA's broader program of building rational agents: represent the task clearly, compute with the representation, and evaluate whether the resulting behavior improves expected performance.

For this chapter, the working frame is **logic-based agents**. The practical objective is to **represent facts as sentences and derive consequences soundly**. That phrasing matters: AIMA is not just a catalog of algorithms; it is a discipline for matching problem structure to a representation and then choosing an inference, learning, or decision procedure.

This section is an original course rewrite aligned to the source chapter. Use the textbook for the full exposition, and use this file as the implementation-oriented study guide.

---

## Learning Targets

By the end of this section, you should be able to:

1. Define **Forward Chaining** using agent-centered language.
2. Identify the representation the method requires.
3. Write the core equation or algorithmic test.
4. Work through a small example by hand.
5. Implement a minimal sanity check in Python.
6. Name a failure mode that would matter in a deployed agent.

---

## Representation Checklist

| Layer | What to specify | Why it matters |
|-------|-----------------|----------------|
| Task | Agent, environment, performance measure | Prevents solving the wrong problem |
| Data | propositional symbols, logical sentences, truth assignments, and knowledge-base updates | Makes assumptions explicit |
| Computation | Inference, search, optimization, or learning | Determines complexity and guarantees |
| Output | Action, belief, plan, prediction, proof, or explanation | Connects algorithm to agent behavior |
| Evaluation | truth-table checks, entailment traces, countermodels, and proof steps | Turns correctness into observable evidence |

Before using any algorithm in this section, fill out the table for a toy version of the problem. The toy version is where hidden assumptions become visible.

---

## Mathematical Anchors

$$
KB \models \alpha
$$
> **Readable form:** Every model that makes the knowledge base true also makes the query true.

$$
KB \vdash_i \alpha
$$
> **Readable form:** Inference procedure i can derive the query from the knowledge base.

$$
\operatorname{ASK}(KB,\alpha) \rightarrow \{true,false\}
$$
> **Readable form:** The agent queries whether a sentence follows from what it knows.

For each equation, identify the inputs, output, and one condition under which the equation would be inappropriate.

---

## Worked Micro-Example

Use a deliberately small problem:

1. Choose two or three variables, actions, agents, symbols, or examples.
2. Write the representation explicitly.
3. Apply the equation or algorithm one step at a time.
4. State the result in agent language: what belief changes, what action is selected, or what prediction is made?
5. Change one assumption and repeat the final step.

This small example is the antidote to passive reading. If the method cannot be explained on a small case, scaling it will only hide the confusion.

---

## Implementation Sketch

```python
from itertools import product

def model_check(kb, query, symbols):
    for values in product([False, True], repeat=len(symbols)):
        model = dict(zip(symbols, values))
        if kb(model) and not query(model):
            return False, model
    return True, None
```

This code is intentionally minimal. Your lab version should add input validation, tests for edge cases, logging, and a comparison against a simpler baseline.

---

## Diagnostics

| Diagnostic | Healthy signal | Warning signal |
|------------|----------------|----------------|
| Toy example | Matches the hand calculation | Disagrees before scaling |
| Boundary case | Handles empty, impossible, or degenerate input | Crashes or returns a confident nonsense answer |
| Complexity check | Growth matches the theory | Runtime explodes unexpectedly |
| Baseline comparison | Improves on a simpler method where expected | Adds complexity without measurable gain |
| Explanation trace | You can inspect why the output occurred | The system only emits an answer |

---

## Failure Modes

| Failure mode | Consequence | Mitigation |
|--------------|-------------|------------|
| Representation mismatch | The agent optimizes the wrong abstraction | Revisit the PEAS description and variables |
| Hidden uncertainty | The method acts overconfidently | Track probabilities, alternatives, or confidence |
| Evaluation leakage | Reported success is inflated | Separate development, validation, and final test cases |
| Scaling surprise | Correct toy method becomes infeasible | Analyze complexity before adding data |
| Objective mismatch | Local metric conflicts with real utility | Review incentives and downstream effects |

---

## Practice Lab

Complete this mini-lab before moving on:

1. Build the smallest possible instance of **Forward Chaining**.
2. Solve it by hand.
3. Implement the same case in Python.
4. Add one edge case that should fail gracefully.
5. Record the complexity driver: states, actions, symbols, examples, agents, or time steps.
6. Write a short note explaining when you would not use this method.

---

## Cross-Course Connections

- **Course 1:** turns this concept into data preparation, metrics, and working code.
- **Course 2:** uses it inside rational-agent design.
- **Course 3:** deepens the math, optimization, and representation-learning pieces.
- **Course 4:** shows how the same abstractions reappear in generative and agentic systems.

---

## Reflection Questions

1. What is the representation used by **Forward Chaining**?
2. What computation turns that representation into an answer?
3. What assumption is easiest to violate in a real system?
4. Which diagnostic would you run first?
5. How does this idea help an agent act rationally?
6. What previous chapter supplies the strongest prerequisite?

---

## References

- Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [https://aima.cs.berkeley.edu/](https://aima.cs.berkeley.edu/)
- AIMA Python code: [https://github.com/aimacode/aima-python](https://github.com/aimacode/aima-python)

---

**Previous:** [Section 07.4: Theorem Proving](./section-04-theorem-proving.md)  
**Next:** [Section 07.6: Backward Chaining](./section-06-backward-chaining.md)




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
