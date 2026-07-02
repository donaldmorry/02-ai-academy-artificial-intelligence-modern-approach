# Chapter 20: Learning Probabilistic Models

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 20
> **Part:** Part V: Machine Learning
> **Estimated time:** 12–14 hours
> **Prerequisites:** Chapter 13: Probabilistic Reasoning; Chapter 19: Learning from Examples

---

## Chapter Overview

Learn parameters and structure of probabilistic models. Cover maximum likelihood, Bayesian parameter estimation, the EM algorithm for latent variables, and structure learning for Bayesian networks. Connect to mixture models and hidden variable methods.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Apply maximum likelihood estimation to parametric distributions
2. Perform Bayesian parameter learning with conjugate priors
3. Derive and implement the EM algorithm for mixture models
4. Explain E-step and M-step interpretations and convergence properties
5. Learn BN parameters with complete and incomplete data
6. Survey structure learning with score-based and constraint-based methods
7. Identify when latent variables require EM or variational methods

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Statistical Learning](./section-01-statistical-learning.md) | Data, parameters, likelihood, priors |
| 2 | [Maximum Likelihood](./section-02-maximum-likelihood.md) | MLE examples, overfitting, regularization link |
| 3 | [Bayesian Learning](./section-03-bayesian-learning.md) | Posterior, MAP, conjugate priors |
| 4 | [EM Algorithm](./section-04-em-algorithm.md) | Latent variables, mixture of Gaussians |
| 5 | [EM Applications](./section-05-em-applications.md) | HMM training, clustering, missing data |
| 6 | [BN Parameter Learning](./section-06-bayesian-network-parameter-learning.md) | Complete data, Dirichlet priors |
| 7 | [Structure Learning](./section-07-structure-learning.md) | Search, scoring (BIC), constraint-based methods |
| 8 | [Nonparametric Bayes Preview](./section-08-nonparametric-bayes-preview.md) | Dirichlet process overview |

---

## Lab / Project

Implement EM for a two-component Gaussian mixture on 2D data. Plot convergence of log-likelihood and visualize learned components.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Clustering | [Course 1 Chapter 01](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md): k-means vs. probabilistic mixtures |
| Naive Bayes generative models | [Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-04-naive-bayes-for-text.md): count-based parameter learning |
| Linear factor models | [Course 3 Chapter 13](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-13-linear-factor-models/README.md): PCA and probabilistic PCA |
| Structured PGMs | [Course 3 Chapter 16](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-16-structured-probabilistic-models/README.md): graphical model foundations |
| HMM training | Course 2 Chapter 14: Baum-Welch as EM |

---

## Prerequisites

- Chapter 13: Probabilistic Reasoning
- Chapter 19: Learning from Examples

---

## Self-Assessment

1. What problem does EM solve that MLE cannot directly?
2. Write the two steps of EM in general terms.
3. When does EM guarantee convergence to global optimum?
4. How does a Dirichlet prior act as pseudo-counts?
5. What is structure learning for Bayesian networks?
6. Why is Baum-Welch considered an EM algorithm?

---

**Previous:** [Chapter 19 — Learning from Examples](../chapter-19-learning-from-examples/README.md)
**Next:** [Chapter 21 — Deep Learning](../chapter-21-deep-learning/README.md)
