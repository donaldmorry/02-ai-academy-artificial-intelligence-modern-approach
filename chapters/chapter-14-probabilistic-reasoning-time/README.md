# Chapter 14: Probabilistic Reasoning over Time

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 14
> **Part:** Part IV: Uncertain Knowledge & Reasoning
> **Estimated time:** 12–14 hours
> **Prerequisites:** Chapter 13: Probabilistic Reasoning

---

## Chapter Overview

Model temporal processes with hidden Markov models, dynamic Bayesian networks, Kalman filters, and particle filters. Address state estimation, smoothing, prediction, and tracking in continuous and discrete state spaces for robotics, finance, and signal processing applications.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Formulate temporal models with Markov assumptions and sensor models
2. Implement HMM filtering, prediction, and smoothing algorithms
3. Apply the Viterbi algorithm for most likely state sequences
4. Derive and implement the Kalman filter for linear Gaussian systems
5. Use particle filters for nonlinear non-Gaussian tracking
6. Represent temporal models as dynamic Bayesian networks
7. Select appropriate temporal inference methods for a given domain

---

## Sections

| # | Section | Topics | Status |
|---|--------|--------|--------|
| 1 | [Time and Uncertainty](./section-01-time-and-uncertainty.md) | States, observations, stationarity, Markov property |
| 2 | [Hidden Markov Models](./section-02-hidden-markov-models.md) | Transition and sensor models, joint distribution |
| 3 | [Inference in HMMs](./section-03-inference-in-hmms.md) | Filtering, prediction, smoothing, Viterbi |
| 4 | [Dynamic Bayesian Networks](./section-04-dynamic-bayesian-networks.md) | Unrolling, 2-slice DBNs, general structure |
| 5 | [Kalman Filters](./section-05-kalman-filters.md) | Linear Gaussian assumptions, update equations |
| 6 | [Particle Filtering](./section-06-particle-filtering.md) | Importance sampling, resampling, degeneracy |
| 7 | [Applications](./section-07-applications.md) | Tracking, speech recognition, econometrics |
| 8 | [Comparison of Methods](./section-08-comparison-of-methods.md) | When linearization fails, particle filter trade-offs |

**Lab:** [Lab 14 — Object Tracking](./section-lab-14-object-tracking-with-kalman-and-particle-filters.md)

---

## Lab / Project

Track a moving object in 2D with a Kalman filter and compare against a particle filter when adding nonlinear observations.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Time series features | Course 1 Chapter 02: regression on sequential data |
| RNN sequence models | Course 3 Chapter 10: neural sequence modeling |
| Video generation | Course 4 Chapter 09: temporal generative models |

---

## Prerequisites

- Chapter 13: Probabilistic Reasoning

---

## Self-Assessment

1. What Markov assumption do HMMs make?
2. Contrast filtering vs. smoothing in temporal inference.
3. When is the Kalman filter optimal?
4. Why are particle filters needed for nonlinear systems?
5. What does the Viterbi algorithm compute?
6. How does a DBN relate to an unrolled BN over time?

---

**Previous:** [Chapter 13 — Probabilistic Reasoning](../chapter-13-probabilistic-reasoning/README.md)
**Next:** [Chapter 15 — Probabilistic Programming](../chapter-15-probabilistic-programming/README.md)
