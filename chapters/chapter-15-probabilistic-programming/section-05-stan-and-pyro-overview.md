# Section 15.5: Stan and Pyro Overview

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 15.5
> **Previous:** [Section 15.4 - Inference in Programs](./section-04-inference-in-programs.md)
> **Next:** [Section 15.6 - Cognitive Models](./section-06-cognitive-models.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Stan: Declarative Probabilistic Models

**Stan** separates **model** (`.stan` file) from **inference** (HMC/NUTS sampler).

```stan
data {
  int<lower=0> N;
  vector[N] y;
  vector[N] x;
}
parameters {
  real alpha;
  real beta;
  real<lower=0> sigma;
}
model {
  alpha ~ normal(0, 10);
  beta ~ normal(0, 10);
  sigma ~ cauchy(0, 5);
  y ~ normal(alpha + beta * x, sigma);
}
```

> **Readable form:** Declare parameters with priors; declare how data is generated - Stan samples the posterior.

---

## Pyro: Probabilistic Programming in Python

**Pyro** embeds random choices in Python control flow:

```python
import pyro
import pyro.distributions as dist

def model(x):
    w = pyro.sample("w", dist.Normal(0, 1))
    b = pyro.sample("b", dist.Normal(0, 1))
    mu = w * x + b
    return pyro.sample("y", dist.Normal(mu, 0.1))

# inference: pyro.infer.MCMC(pyro.infer.NUTS(model)).run(x, y_obs)
```

---

## Workflow Comparison

| Step | Stan | Pyro |
|------|------|------|
| Specify model | Stan language | Python functions |
| Inference | `sample()` | `MCMC`, `SVI` |
| Diagnostics | `summary()`, R-hat | Tensor shapes, loss curves |
| Deploy | CmdStan, PyStan | PyTorch ecosystem |

---

## Hamiltonian Monte Carlo (HMC)

Uses gradients of log-posterior to simulate physical dynamics - efficient in continuous parameter spaces.

**NUTS** (No-U-Turn Sampler) adapts trajectory length automatically - Stan default.

---

## Variational Inference in Pyro

**SVI** optimizes a simpler distribution to approximate the posterior - scales to large data and deep models.

Trade-off: faster, biased vs. MCMC asymptotic exactness.

---

## Diagnostics

| Diagnostic | Meaning |
|------------|---------|
| **R-hat** | Across chains - should be $\approx 1$ |
| **ESS** | Effective sample size |
| **Divergences** | HMC integration failures - reparameterize |

---

## When to Choose Which

| Situation | Tool |
|-----------|------|
| Tabular Bayesian regression | Stan |
| Deep generative model | Pyro |
| Custom MCMC in research | Pyro flexibility |
| Production batch jobs | CmdStan |

---

## Key Takeaways

1. **Stan** excels at declarative statistical models with HMC.
2. **Pyro** integrates with PyTorch for deep probabilistic models.
3. Both separate **model** from **inference engine**.
4. Always check **convergence diagnostics** before trusting posteriors.
5. PPLs implement [Section 15.4](./section-04-inference-in-programs.md) algorithms at scale.

## Worked Example: Stan and Pyro Overview

Apply the ideas from **Syntax patterns, workflow, diagnostics** to a small concrete instance. State assumptions explicitly, write the governing equations, and interpret each term. Compare your result with the AIMA chapter exercise that matches Chapter 15.5.

> **Readable form:** Translate the abstract definition into numbers or code for one toy case.


## Design Checklist

| Step | Action |
|------|--------|
| 1 | Identify variables and constraints for stan and pyro overview |
| 2 | Choose representation aligned with Syntax patterns, workflow, diagnostics |
| 3 | Verify against AIMA Chapter 15.5 definitions |
| 4 | Implement minimal prototype in Python |
| 5 | Test edge cases and document failure modes |


## Connections Across Course 2

This section on **Stan and Pyro Overview** links forward to later chapters that reuse the same abstractions. Revisit [../../GLOSSARY.md](../../GLOSSARY.md) entries while reading, and cross-check notation with [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md). Prior search and logic chapters provide algorithmic scaffolding; probability chapters supply uncertainty semantics when Syntax patterns, workflow, diagnostics involves partial information.


## Common Mistakes

- **Confusing syntax with semantics** in Stan and Pyro Overview - symbols must map to models or distributions.
- **Skipping units and domains** when Syntax patterns, workflow, diagnostics involves continuous or structured variables.
- **Treating heuristics as guarantees** - always state when optimality or soundness holds.
- **Ignoring complexity** - tractability assumptions in Chapter 15.5 may fail at scale.
- **Omitting sensor/actuator noise** when deploying ideas from textbook toy domains.


## Deeper Reading

Re-read Russell & Norvig Chapter 15.5 with pen and paper. For **Stan and Pyro Overview**, derive each formula from first principles before using libraries. The aima-python repository implements many algorithms from this chapter - compare your pseudocode to the reference implementation and note API differences.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 is cumulative - search, logic, probability, and learning share representational foundations.

---

## Practice Trace

Hand-execute the main algorithm from this section on a toy instance small enough to fit on one page. Record each intermediate structure (clause set, belief state, plan layer, or gradient step). Compare your trace to aima-python reference code if available.

> **Readable form:** If you cannot trace it by hand on paper, you do not yet understand it well enough to deploy it.


---

## Practice Trace

Hand-execute the main algorithm from this section on a toy instance small enough to fit on one page. Record each intermediate structure (clause set, belief state, plan layer, or gradient step). Compare your trace to aima-python reference code if available.

> **Readable form:** If you cannot trace it by hand on paper, you do not yet understand it well enough to deploy it.

## Study Note

Review the AIMA Chapter 15.5 exercises and trace one algorithm by hand before the lab.

---

## Key Terms (Glossary)

- **Syntax** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **patterns** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **workflow** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **diagnostics** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Stan and Pyro Overview in AIMA?
2. How does Stan and Pyro Overview relate to Syntax patterns, workflow, diagnostics?
3. Give a concrete application of Stan and Pyro Overview.
4. What assumptions does the textbook emphasize for Stan and Pyro Overview?
5. When would an alternative to Stan and Pyro Overview be preferable?
6. How would you verify understanding of Stan and Pyro Overview in code?

---

**Previous:** [Section 15.4 - Inference in Programs](./section-04-inference-in-programs.md) · **Next:** [Section 15.6 - Cognitive Models](./section-06-cognitive-models.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Stan Development Team. [mc-stan.org](https://mc-stan.org/)']

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
