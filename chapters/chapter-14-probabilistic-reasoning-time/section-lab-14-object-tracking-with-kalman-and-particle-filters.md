# Lab 14: Object Tracking with Kalman and Particle Filters

> **Chapter:** [14 - Probabilistic Reasoning over Time](./README.md)  
> **Glossary:** [GLOSSARY.md](../../GLOSSARY.md) | **Math:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Objective

Track a moving object in 2D with a **Kalman filter** (linear Gaussian) and compare to a **particle filter** when observations become nonlinear.

---

## Setup

```python
import numpy as np
import matplotlib.pyplot as plt
```

Simulate constant-velocity motion with noisy position measurements.

---

## Part 1: Kalman Filter

1. Define $F$, $H$, $Q$, $R$ for 2D position/velocity state.
2. Implement predict-update loop ([Section 14.5](./section-05-kalman-filters.md)).
3. Plot true vs estimated trajectory.

---

## Part 2: Nonlinear Observations

1. Replace linear $H$ with range-bearing sensor $h(x)$.
2. Implement particle filter ([Section 14.6](./section-06-particle-filtering.md)).
3. Compare RMSE and runtime.

---

## Deliverables

- Plots: Kalman vs ground truth; particle filter with nonlinear sensor
- Short report: when Kalman fails and why particles help
- Link to [Chapter 26](../chapter-26-robotics/section-04-localization.md) localization

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Thrun et al., *Probabilistic Robotics*

## Extension

Try different process noise $\Sigma_x$ and observe filter lag vs jitter.

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
