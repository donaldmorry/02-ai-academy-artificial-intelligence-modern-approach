# Section 15.7: Practical Workflow

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 15.7
> **Previous:** [Section 15.6 - Cognitive Models](./section-06-cognitive-models.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Overview

A practical probabilistic-programming workflow starts from the source idea in AIMA Chapter 15: a probabilistic program defines a probability distribution over execution traces. Every random choice in the program is a random variable; conditioning on evidence turns prior traces into a posterior distribution over explanations, predictions, and hidden causes.

The workflow is therefore not just “write Stan or Pyro code.” It is a disciplined loop:

1. Write the generative story.
2. Choose priors that encode real assumptions.
3. Condition on observed evidence.
4. Run approximate inference.
5. Check whether inference has mixed or converged well enough.
6. Run posterior predictive checks against data the model should reproduce.
7. Revise the model or inference procedure when the checks fail.

> **Readable form:** A probabilistic program is a model of possible executions; inference asks which executions are plausible after observing evidence.

---

## Source-Aligned Summary

AIMA describes two routes to more expressive probability models. One route extends logic into relational and open-universe probability models. The other route extends ordinary programs with random choices, producing **generative programs**. This section focuses on the practical workflow implied by the second route.

In a generative program, an execution trace is the ordered record of random choices made by one run of the program. The probability of a trace is the product of the probabilities of those choices, conditioned on earlier choices.

$$
P(\omega)=\prod_i P(x_i \mid x_1,\ldots,x_{i-1})
$$
> **Readable form:** The probability of one program run is the product of the probabilities of the random choices that produced that run.

This source framing matters because it changes how you debug models. You do not only inspect parameters; you inspect traces, evidence constraints, and proposal moves through the trace space.

---

## Step 1: Write the Generative Story

AIMA's degraded-text example is the right mental model. The program first samples latent text, then samples noise and rendering details, and finally produces an observed image. Inference reverses that story: given the image, infer the likely latent text and noise settings.

| Workflow question | Practical answer |
|-------------------|------------------|
| What is hidden? | Latent variables such as true letters, object identities, skills, causes, or parameters |
| What is observed? | Evidence such as pixels, ratings, outcomes, sensor readings, or measurements |
| What is random? | Every stochastic choice in the generative program |
| What is deterministic? | Computations that transform sampled values into derived quantities |
| What is queried? | Posterior beliefs, predictions, rankings, or decisions |

A good generative story can be simulated before any data are observed. If prior samples look impossible, inference will not rescue the model.

---

## Step 2: Choose Priors Deliberately

Prior selection is where domain knowledge enters before evidence. AIMA shows this through examples such as book recommendations, player skill ratings, and degraded text. Priors do not need to be perfect, but they must be explicit enough to audit.

| Prior choice | Check |
|--------------|-------|
| Weakly informative prior | Does it keep inference away from absurd values without dominating the data? |
| Strong domain prior | Can you defend the assumption with source knowledge or measurements? |
| Hierarchical prior | Does it share statistical strength across related objects or groups? |
| Structural prior | Does it encode likely existence, identity, sequence, or dependency structure? |

Run **prior predictive checks** before fitting: sample from the model without conditioning on observations and inspect whether generated data are plausible. This catches many modeling errors earlier than posterior diagnostics.

---

## Step 3: Run Inference and Check Mixing

AIMA is blunt about Monte Carlo inference: there is no foolproof convergence test. In probabilistic programs, MCMC modifies execution traces, and changing one random choice can invalidate later choices if the program's control flow changes.

Use several diagnostics together:

| Diagnostic | What it reveals |
|------------|-----------------|
| Multiple chains | Whether independent runs reach similar regions of trace space |
| Trace plots | Whether sampled quantities wander, stick, or jump abruptly |
| Effective sample size | Whether nominal samples contain enough independent information |
| Divergences or rejected moves | Whether the sampler struggles with geometry or proposals |
| Sensitivity to initialization | Whether conclusions depend on starting traces |

If inference fails, AIMA suggests two broad repair paths: improve the inference procedure or improve the model. In the degraded-text example, adding a Markov model for letter sequences improves the prior structure; designing better MCMC proposals improves exploration.

---

## Step 4: Posterior Predictive Checks

A posterior predictive check asks: if the fitted model is true enough, can it generate data that look like the observed data in the ways that matter?

$$
y^{\mathrm{rep}} \sim P(y^{\mathrm{rep}} \mid y)
$$
> **Readable form:** Draw new simulated observations from the posterior and compare them with the real observations.

For a text-reading model, generate degraded images from posterior traces and compare letter ambiguity, noise level, and visual structure. For a rating model, simulate future match outcomes and compare win rates. For a customer-recommendation model, simulate recommendation distributions and check whether kind, dishonest, and expert customers behave differently.

| Posterior predictive failure | Likely interpretation |
|------------------------------|-----------------------|
| Simulated data are too clean | Observation-noise model is too optimistic |
| Simulated data are too variable | Priors or likelihood are too diffuse |
| Group patterns disappear | Hierarchical or relational structure is missing |
| Rare events never appear | Tail behavior is under-modeled |
| Predictions match averages but not extremes | Model captures central tendency but not uncertainty |

---

## Practical Workflow Template

Use this checklist for every PPL project in the lab:

| Stage | Required artifact |
|-------|-------------------|
| Generative story | One paragraph explaining how latent variables produce observations |
| Variables | Table of latent variables, observed variables, parameters, and deterministic quantities |
| Priors | Rationale and prior predictive samples |
| Inference | Sampler or algorithm, settings, chains, seeds, and runtime |
| Convergence | Trace plots and at least two diagnostics |
| Posterior | Credible intervals, posterior means, or posterior samples for target queries |
| Posterior predictive | Simulated data compared with observed data |
| Revision | One model or inference change motivated by a failed check |

---

## Failure Analysis

| Failure mode | Why it matters | Mitigation |
|--------------|----------------|------------|
| Prior too strong | Evidence cannot move the posterior enough | Compare prior and posterior; weaken or justify the prior |
| Prior too weak | Sampler explores unrealistic regions | Add weakly informative constraints or hierarchical structure |
| Poor proposal distribution | MCMC mixes slowly or sticks | Use specialized proposals, reparameterization, or adaptive inference |
| Bad likelihood | Posterior explains the wrong data features | Run posterior predictive checks by feature slice |
| Hidden multimodality | Chains disagree while summaries look stable | Inspect chains separately and try alternate initializations |
| Overfit posterior predictive | Model reproduces seen data but fails new cases | Hold out observations and test predictive calibration |

---

## Worked Mini-Project

Model noisy customer ratings for books:

1. Sample each book quality from a categorical or ordered prior.
2. Sample each customer kindness and honesty.
3. Generate ratings conditioned on quality, kindness, and honesty.
4. Condition on observed ratings.
5. Infer posterior book quality and customer traits.
6. Generate posterior predictive ratings for held-out customer-book pairs.

This mirrors AIMA's relational book-recommendation example while forcing the same workflow used in Stan or Pyro.

---

## Reflection Questions

1. What random choices define the execution trace in your model?
2. Which prior encodes the strongest domain assumption?
3. What evidence conditions the posterior?
4. What diagnostic would convince you that inference is not mixing?
5. What posterior predictive statistic should match the observed data?
6. Would you improve the model, the sampler, or the data collection first?

---

**Previous:** [Section 15.6 - Cognitive Models](./section-06-cognitive-models.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 15.
2. Stan Development Team. [https://mc-stan.org/](https://mc-stan.org/)
3. Pyro documentation. [https://pyro.ai/](https://pyro.ai/)

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
