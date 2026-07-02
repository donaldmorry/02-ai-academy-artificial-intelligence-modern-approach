# Chapter 12: Quantifying Uncertainty

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 12
> **Part:** Part IV: Uncertain Knowledge & Reasoning
> **Estimated time:** 11–13 hours
> **Prerequisites:** Chapter 07: Logical Agents; Appendix A: Mathematical Background (probability sections)

---

## Chapter Overview

Introduce probability as the language of uncertainty for AI agents. Cover axioms, conditional probability, Bayes' rule, joint distributions, and naive Bayes classifiers. Contrast Bayesian and frequentist interpretations and connect to real diagnostic and classification problems.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Apply Kolmogorov axioms and basic probability identities
2. Use conditional probability and independence assumptions appropriately
3. Apply Bayes' rule to diagnostic and hypothesis-update problems
4. Construct and query joint probability tables for small domains
5. Implement a naive Bayes text or tabular classifier
6. Explain base-rate neglect and how Bayes corrects it
7. Identify when independence assumptions are justified or harmful

---

## Sections

| # | Section | Topics | Status |
|---|--------|--------|--------|
| 1 | [Acting Under Uncertainty](./section-01-acting-under-uncertainty.md) | Why logic alone is insufficient, probability as belief | ✅ |
| 2 | [Basic Probability](./section-02-basic-probability.md) | Axioms, sample space, priors, joint distributions | ✅ |
| 3 | [Conditional Probability](./section-03-conditional-probability.md) | Product rule, chain rule, Bayes' rule | ✅ |
| 4 | [Independence](./section-04-independence.md) | Conditional independence, naive Bayes assumption | ✅ |
| 5 | [Bayes' Rule Applications](./section-05-bayes-rule-applications.md) | Medical diagnosis, base rates, sensor fusion | ✅ |
| 6 | [Naive Bayes Classifier](./section-06-naive-bayes-classifier.md) | Training, smoothing, log probabilities, spam | ✅ |
| 7 | [Wumpus Revisited](./section-07-wumpus-revisited.md) | Probabilistic reasoning in partially known grids | ✅ |

---

## Lab / Project

Train a naive Bayes spam classifier on a public email corpus. Report accuracy, precision/recall, and effect of Laplace smoothing.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Classification models | Course 1 Chapter 03: logistic regression and Naive Bayes practice |
| Bayesian deep learning | Course 3 Chapter 07: uncertainty in neural networks |
| Probabilistic generative models | Course 4 Chapter 01: VAE probabilistic foundations |

---

## Prerequisites

- Chapter 07: Logical Agents
- Appendix A: Mathematical Background (probability sections)

---

## Self-Assessment

1. State Bayes' rule and define each term in a diagnostic context.
2. When are two variables conditionally independent given a third?
3. Why does naive Bayes use conditional independence of features?
4. What problem does Laplace smoothing solve?
5. Compute P(A|B) given P(B|A), P(A), and P(B) for a small example.
6. Why is base-rate information essential for correct medical testing conclusions?

---

**Previous:** [Chapter 11 — Automated Planning](../chapter-11-automated-planning/README.md)
**Next:** [Chapter 13 — Probabilistic Reasoning](../chapter-13-probabilistic-reasoning/README.md)
