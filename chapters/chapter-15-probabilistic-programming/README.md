# Chapter 15: Probabilistic Programming

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 15
> **Part:** Part IV: Uncertain Knowledge & Reasoning
> **Estimated time:** 10–12 hours
> **Prerequisites:** Chapter 12: Quantifying Uncertainty; Chapter 13: Probabilistic Reasoning

---

## Chapter Overview

Express probabilistic models as programs with random choices, enabling open-universe and relational models. Survey languages (Stan, Pyro, Church concepts), inference for program-defined models, and applications to cognitive science and data analysis with complex latent structure.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Explain probabilistic programs as generative model specifications
2. Represent open-universe models with stochastic object creation
3. Contrast relational probability models with flat Bayesian networks
4. Run inference on program-defined models using MCMC or VI
5. Read and write simple models in a probabilistic programming language
6. Identify when probabilistic programming beats hand-built BNs
7. Connect programmatic models to modern Bayesian data science workflows

---

## Sections

| # | Section | Topics | Status |
|---|--------|--------|--------|
| 1 | [Relational Probability](./section-01-relational-probability.md) | Objects, relations, identity uncertainty |
| 2 | [Open-Universe Models](./section-02-open-universe-models.md) | Number uncertainty, existence uncertainty |
| 3 | [Probabilistic Programs](./section-03-probabilistic-programs.md) | Random choices, conditioning, observe |
| 4 | [Inference in Programs](./section-04-inference-in-programs.md) | MCMC, HMC, variational inference in PPLs |
| 5 | [Stan and Pyro Overview](./section-05-stan-and-pyro-overview.md) | Syntax patterns, workflow, diagnostics |
| 6 | [Cognitive Models](./section-06-cognitive-models.md) | Learning as inference, human reasoning approximations |
| 7 | [Practical Workflow](./section-07-practical-workflow.md) | Prior selection, convergence checks, posterior predictive |

**Lab:** [Lab 15 — Stan/Pyro Models](./section-lab-15-bayesian-regression-and-mixture-models.md)

---

## Lab / Project

**[Lab 15: Bayesian Regression and Mixture Models](./section-lab-15-bayesian-regression-and-mixture-models.md)** — Implement Bayesian linear regression and a Gaussian mixture model in Stan and Pyro. Compare posterior estimates with OLS and analytical solutions where available.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Bayesian statistics | Course 3 Chapter 07: Bayesian neural network priors |
| Generative modeling | Course 4 Chapter 01: VAE as amortized inference |
| FOL representations | Course 2 Chapter 08: relational structure in models |

---

## Prerequisites

- Chapter 12: Quantifying Uncertainty
- Chapter 13: Probabilistic Reasoning

---

## Self-Assessment

1. What is an open-universe model and why does standard BN notation struggle?
2. How does conditioning work in a probabilistic program?
3. When would you choose HMC over Gibbs sampling?
4. What is relational uncertainty?
5. How do PPLs separate model specification from inference?
6. Give an example where object identity is uncertain.

---

**Previous:** [Chapter 14 — Probabilistic Reasoning over Time](../chapter-14-probabilistic-reasoning-time/README.md)
**Next:** [Chapter 16 — Making Simple Decisions](../chapter-16-making-simple-decisions/README.md)
