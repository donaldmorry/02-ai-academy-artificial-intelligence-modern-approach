# Chapter 19: Learning from Examples

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 19  
> **Part:** Part V: Machine Learning  
> **Estimated time:** 13–15 hours  
> **Prerequisites:** [Chapter 12 — Quantifying Uncertainty](../chapter-12-quantifying-uncertainty/README.md); Course 1: Chapters 01–03 (Applied ML foundations)

---

## Chapter Overview

Study supervised learning from a formal AI perspective: decision trees, linear and nonlinear models, ensemble methods, and learning theory basics. Cover bias-variance trade-off, regularization, cross-validation, and comparison with Course 1 implementations.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Formulate supervised learning as empirical risk minimization
2. Build and prune decision trees using information gain and Gini index
3. Train linear and logistic regression models with regularization
4. Apply kernel methods and support vector machines conceptually
5. Implement bagging, random forests, and boosting (AdaBoost overview)
6. Explain PAC learning, VC dimension, and generalization bounds at introductory level
7. Evaluate models with proper train/validation/test methodology

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Learning Agents](./section-01-learning-agents.md) | Inductive learning, training vs. test, overfitting |
| 2 | [Decision Trees](./section-02-decision-trees.md) | Entropy, information gain, pruning |
| 3 | [Linear Models](./section-03-linear-models.md) | Regression, classification, gradient descent |
| 4 | [Nonparametric Models](./section-04-nonparametric-models.md) | k-NN, kernel density, locally weighted regression |
| 5 | [Support Vector Machines](./section-05-support-vector-machines.md) | Margins, kernels, soft margin |
| 6 | [Ensemble Learning](./section-06-ensemble-learning.md) | Bagging, random forests, boosting |
| 7 | [Learning Theory](./section-07-learning-theory.md) | PAC, VC dimension, bias-variance decomposition |
| 8 | [Model Selection](./section-08-model-selection.md) | Cross-validation, hyperparameters, statistical tests |

---

## Lab / Project

On a tabular dataset, compare decision tree, logistic regression, and random forest with cross-validation. Analyze bias-variance via learning curves.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Scikit-Learn workflows | Course 1 Chapters 02–05: hands-on classical ML |
| Statistical learning theory | Course 3 Chapter 05: ML basics formalized |
| Feature learning | Course 4 Chapter 03: representations beyond hand features |

---

## Prerequisites

- [Chapter 12 — Quantifying Uncertainty](../chapter-12-quantifying-uncertainty/README.md)
- Course 1: Chapters 01–03 (Applied ML foundations)

---

## Self-Assessment

1. What is the difference between empirical and true error?
2. How does information gain guide decision tree splits?
3. Explain the bias-variance trade-off with an example.
4. Why does boosting reduce bias but can increase variance?
5. What role does the margin play in SVMs?
6. When is a high VC dimension concerning?

---

**Previous:** [Chapter 18 — Multiagent Decision Making](../chapter-18-multiagent-decision-making/README.md)  
**Next:** [Chapter 20 — Learning Probabilistic Models](../chapter-20-learning-probabilistic-models/README.md)
