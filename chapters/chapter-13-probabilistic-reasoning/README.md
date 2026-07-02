# Chapter 13: Probabilistic Reasoning

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 13
> **Part:** Part IV: Uncertain Knowledge & Reasoning
> **Estimated time:** 13–15 hours
> **Prerequisites:** Chapter 12: Quantifying Uncertainty

---

## Chapter Overview

Represent uncertain knowledge with Bayesian networks: directed acyclic graphs with conditional probability tables. Perform exact inference via variable elimination and sampling methods (likelihood weighting, Gibbs). Study network construction, conditional independence semantics, and approximate inference trade-offs.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Construct Bayesian networks reflecting domain causal structure
2. Read conditional independence statements from network topology
3. Perform exact inference with variable elimination
4. Implement likelihood weighting and Gibbs sampling for approximate inference
5. Compare complexity of exact vs. approximate methods
6. Design CPTs for small networks and assess sensitivity
7. Apply Bayesian networks to diagnosis, prediction, and decision support

---

## Sections

| # | Section | Topics | Status |
|---|--------|--------|--------|
| 1 | [Representing Knowledge](./section-01-representing-knowledge.md) | Joint distributions, factoring, BN syntax |
| 2 | [Semantics of BNs](./section-02-semantics-of-bns.md) | Conditional independence, d-separation overview |
| 3 | [Exact Inference](./section-03-exact-inference.md) | Enumeration, variable elimination, factor operations |
| 4 | [Approximate Inference](./section-04-approximate-inference.md) | Sampling, likelihood weighting, particle filtering preview |
| 5 | [Gibbs Sampling](./section-05-gibbs-sampling.md) | MCMC for BNs, burn-in, convergence |
| 6 | [BN Construction](./section-06-bn-construction.md) | Causal chains, common causes, explaining away |
| 7 | [Case Studies](./section-07-case-studies.md) | Medical diagnosis, alarm network, mud robotics |
| 8 | [Inference Complexity](./section-08-inference-complexity.md) | Tree-width, NP-hardness, junction trees overview |

---

## Lab / Project

Build a small Bayesian network (5–8 nodes) for a domain of your choice. Implement variable elimination or use a library; compare with Gibbs sampling estimates.

Hands-on lab: [Lab 13 - Bayesian Network Inference](./section-lab-13-bayesian-network-inference.md).

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Probabilistic graphical models | Course 3 Chapter 16: PGM chapter deep dive |
| Variational inference | Course 4 Chapter 01: VAE encoder approximates posterior |
| Causal inference | Course 2 Chapter 16: decision networks extend BNs |

---

## Prerequisites

- Chapter 12: Quantifying Uncertainty

---

## Self-Assessment

1. What does a node and CPT represent in a Bayesian network?
2. Explain the 'explaining away' effect with a three-node example.
3. How does variable elimination exploit conditional independence?
4. When is Gibbs sampling preferred over exact inference?
5. What is d-separation used for?
6. Why can exact inference be exponential in network width?

---

**Previous:** [Chapter 12 — Quantifying Uncertainty](../chapter-12-quantifying-uncertainty/README.md)
**Next:** [Chapter 14 — Probabilistic Reasoning over Time](../chapter-14-probabilistic-reasoning-time/README.md)
