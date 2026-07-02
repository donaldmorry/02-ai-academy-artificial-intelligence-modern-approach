# Chapter 09: Inference in First-Order Logic

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 9
> **Part:** Part III: Knowledge, Reasoning & Planning
> **Estimated time:** 13–15 hours
> **Prerequisites:** Chapter 07: Logical Agents; Chapter 08: First-Order Logic

---

## Chapter Overview

Develop automated inference for FOL through unification, generalized Modus Ponens, forward and backward chaining with definite clauses, and resolution with refutation completeness. Address undecidability, semi-decidability, and practical theorem-proving strategies.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Apply unification to find most general unifiers for FOL literals
2. Use Generalized Modus Ponens for first-order forward inference
3. Implement forward chaining over definite clauses in FOL
4. Implement backward chaining with backtracking and unification
5. Convert FOL sentences to clause form for resolution
6. Explain semi-decidability of FOL entailment and practical implications
7. Compare inference strategies for efficiency and completeness

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Propositional vs. FOL Inference](./section-01-propositional-vs-fol-inference.md) | Grounding, Herbrand universe preview |
| 2 | [Unification](./section-02-unification.md) | MGU, occurs check, unification algorithm |
| 3 | [Forward Chaining in FOL](./section-03-forward-chaining-in-fol.md) | Generalized Modus Ponens, matching |
| 4 | [Backward Chaining](./section-04-backward-chaining.md) | Goal decomposition, proof trees, loops |
| 5 | [Resolution in FOL](./section-05-resolution-in-fol.md) | CNF conversion, skolemization, refutation |
| 6 | [Efficiency and Termination](./section-06-efficiency-and-termination.md) | Indexing, subsumption, undecidability |
| 7 | [Theorem Provers](./section-07-theorem-provers.md) | Vampire, Prolog-style execution models |
| 8 | [Practical Knowledge Base Queries](./section-08-practical-knowledge-base-queries.md) | Combining inference with KB design |

---

## Lab / Project

Implement unification and a small backward-chaining prover for Horn clauses in a family-relations knowledge base. Test on grandparent and ancestor queries.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Logic programming | Course 2 Chapter 10: rule-based ontological reasoning |
| Automated planning | Course 2 Chapter 11: planning as logical inference |
| Neuro-symbolic AI | Course 3 Chapter 12: combining logic and neural models |

---

## Prerequisites

- Chapter 07: Logical Agents
- Chapter 08: First-Order Logic

---

## Self-Assessment

1. What is the most general unifier of P(x,f(x)) and P(y,y)?
2. Why is FOL entailment only semi-decidable?
3. How does Generalized Modus Ponens extend Modus Ponens?
4. What role does skolemization play in resolution?
5. When does backward chaining risk infinite loops?
6. Contrast forward vs. backward chaining for large KBs.

---

**Previous:** [Chapter 08 — First-Order Logic](../chapter-08-first-order-logic/README.md)
**Next:** [Chapter 10 — Knowledge Representation](../chapter-10-knowledge-representation/README.md)
