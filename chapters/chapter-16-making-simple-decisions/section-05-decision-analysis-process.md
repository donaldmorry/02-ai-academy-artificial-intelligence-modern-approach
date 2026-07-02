# Section 16.5: Decision Analysis Process

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 16.7  
> **Previous:** [Section 16.4 - Value of Information](./section-04-value-of-information.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Theory to Practice

Decision theory provides normative principles - MEU, utility functions, decision networks, VPI. **Decision analysis** is the disciplined process of applying these tools to real problems with stakeholders, uncertainty, and organizational constraints.

Howard (1960s) pioneered decision analysis in oil drilling, aerospace, and public policy. The goal is not a single "correct number" but a **transparent model** that clarifies trade-offs and supports defensible choices.

> **Readable form:** Decision analysis is structured common sense with math - draw the problem, put numbers where you can, stress-test where you can't, and document what you assumed.

---

## The Eight-Step Process

| Step | Activity | Output |
|------|----------|--------|
| 1 | **Frame the problem** | Scope, stakeholders, objectives |
| 2 | **Identify alternatives** | Decision options (including status quo) |
| 3 | **Structure uncertainties** | Variables, dependencies, diagram |
| 4 | **Assess probabilities** | Data, models, expert elicitation |
| 5 | **Assess utilities** | Preferences, risk attitude ([Section 16.2](./section-02-utility-theory.md)) |
| 6 | **Compute MEU** | Optimal action, expected value |
| 7 | **Sensitivity analysis** | Which inputs matter most |
| 8 | **Communicate & decide** | Recommendation with caveats |

Steps 3-6 map directly to [decision networks](./section-03-decision-networks.md). Steps 7-8 prevent false precision.

---

## Problem Framing

**Framing** determines everything downstream. Questions to ask:

- What decision is actually being made? By whom? By when?
- What is *out of scope*? (Avoid scope creep in the model)
- Are we choosing an action or designing a policy over time?
- Single objective or multiple attributes? ([Section 16.7](./section-07-multiattribute-utility.md))

**Common framing errors:**

| Error | Example | Fix |
|-------|---------|-----|
| Wrong question | "Which drug?" when "Treat or not?" is primary | Separate decisions |
| Hidden option | Ignoring "wait and monitor" | Include status quo |
| Binary trap | "Launch or cancel" vs. phased rollout | Expand action set |

---

## Probability Assessment

Sources ranked by reliability:

1. **Large-scale data** - frequencies, clinical trials, A/B tests
2. **Calibrated models** - BNs, regression, simulation
3. **Expert judgment** - structured elicitation with calibration training
4. **Default priors** - weak; document uncertainty

**Elicitation techniques:**

- **Reference class forecasting** - base rates from similar cases
- **Decomposition** - $P(A \land B) = P(A) P(B \mid A)$ for rare events
- **Calibration** - experts assign probabilities to events they can later verify

From [Chapter 12](../chapter-12-quantifying-uncertainty/README.md): always condition on the same evidence across alternatives.

---

## Utility Elicitation in Groups

Multiple stakeholders often disagree on utilities. Approaches:

| Approach | Description |
|----------|-------------|
| **Single decision-maker** | One utility function; others advise |
| **Negotiated weights** | Weighted sum of stakeholder utilities |
| **Pareto analysis** | Identify dominated alternatives |
| **Scenario comparison** | Show rankings under different weight sets |

Transparency matters: publishing utility ranges often resolves disputes more than hiding assumptions.

---

## Sensitivity Analysis

**One-way sensitivity:** vary one parameter while holding others fixed.

$$
\text{EU}(a) \text{ vs. } P(\text{disease}) \text{ plotted for each action } a
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**Tornado diagrams:** rank parameters by swing in optimal decision or EU.

**Threshold analysis:** find critical values where the optimal action switches.

**Example:** Treatment is optimal if $P(\text{disease}) > 0.15$. Current estimate 0.12 ± 0.05 - recommendation is **sensitive**; gather more data ([Section 16.4](./section-04-value-of-information.md)).

**Two-way sensitivity:** heatmaps over $(p, u)$ pairs - useful when both probability and utility are uncertain.

---

## Value of Modeling

Building the model often changes decisions **before** MEU is computed - stakeholders discover missing options or dependencies. This **clarity value** exceeds the arithmetic result.

Document:

- Assumptions explicitly listed
- Data sources and dates
- Known model limitations
- Recommended action **and** conditions that would reverse it

---

## Worked Walkthrough: Equipment Replacement

**Decision:** Replace aging server now or wait one year?

| Variable | Values | Notes |
|----------|--------|-------|
| Failure in year | {yes, no} | Affects downtime cost |
| New tech gain | {low, high} | Performance if wait |
| Replace now | decision | Capital expense |

Utilities combine: downtime cost, capital cost, performance revenue.

1. Structure BN: failure rate depends on age; tech gain uncertain
2. Assess $P(\text{failure})$, utilities for downtime (\$-100k), replacement cost (\$-50k)
3. MEU: compare EU(replace now) vs. EU(wait)
4. Sensitivity: if failure probability < 0.1, waiting wins - current estimate 0.25

**Output:** Replace now; revisit if failure rate drops below 0.15 (monitoring trigger).

---

## Decision Analysis vs. Cost-Benefit Analysis

| Aspect | Decision analysis | Cost-benefit |
|--------|-------------------|--------------|
| Preferences | Explicit utilities | Often monetary conversion |
| Uncertainty | Probabilities central | Sometimes point estimates |
| Risk attitude | Utility curvature | Often risk-neutral dollars |
| Information value | VPI/VSI formal | Less standard |

Many public-sector analyses convert lives to dollars (value of statistical life) - controversial but enables comparison ([Section 16.2](./section-02-utility-theory.md)).

---

## Software and Reproducibility

Modern workflow:

```python
# Pseudocode decision analysis pipeline
model = DecisionNetwork.from_spec(yaml_spec)
model.set_probabilities(data_or_experts)
model.set_utilities(elicitation_session)
result = model.solve_meu(evidence=current_evidence)
sensitivity = model.tornado_diagram(parameters=["p_failure", "u_downtime"])
report = DecisionReport(result, sensitivity, assumptions=spec.assumptions)
```

Reproducibility: version-control the model spec, data, and elicitation records.

---

## Integration with AI Systems

Autonomous agents can embed decision analysis:

- **Medical CDS:** BN + utility → treatment recommendation with explanation
- **Autonomous vehicles:** risk-weighted planning under uncertainty
- **Business agents:** bid/pricing with EU maximization

Human oversight requires **explainability** - show which probabilities and utilities drove the recommendation.

---

## Common Mistakes

**Analysis paralysis.** Perfect model unavailable; deadline forces decision. Use sensitivity to identify critical unknowns.

**Garbage in, gospel out.** Precise MEU from bad inputs - always show ranges.

**Omitting opponents' utilities.** Competitive settings need [Chapter 18](../chapter-18-multiagent-decision-making/README.md).

**Static model for dynamic world.** Revisit when evidence shifts materially.

---

## Reflection Questions

1. Why is problem framing considered the most important step?
2. Design a tornado diagram for the medical treatment network - which parameters would you expect to rank highest?
3. How would you handle two stakeholders whose utilities rank alternatives differently?
4. When does sensitivity analysis recommend gathering more information vs. acting now?
5. What documentation would you require before deploying an automated decision-analysis system in healthcare?

---

## Facilitated Elicitation Workshop

Professional decision analysts run **structured workshops**:

| Phase | Activity |
|-------|----------|
| Opening | Align on frame, stakeholders, boundaries |
| Structure | Whiteboard influence diagram collaboratively |
| Probabilities | Delphi method, calibration exercises |
| Utilities | Standard gambles in breakout sessions |
| Synthesis | Roll back tree, present MEU recommendation |
| Close | Sensitivity report, monitoring plan |

**Calibration training:** forecasters practice estimating 90% confidence intervals - feedback reduces overconfidence before numbers enter the model.

---

## Expected Value of Perfect Information in Workshops

After base-case MEU, compute **VPI** for key uncertainties:

- If VPI of seismic survey > cost → recommend survey
- If VPI = 0 for additional lab test → skip test in protocol

Bridges [Section 16.4](./section-04-value-of-information.md) to executive communication - "this test is worth at most \$X to the decision."

---

**Previous:** [Section 16.4 - Value of Information](./section-04-value-of-information.md)  
**Next:** [Section 16.6 - Human Biases](./section-06-human-biases.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 16.7. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Howard, R. A. - decision analysis methodology. *IEEE Transactions on Systems Science and Cybernetics*.
3. Raiffa, H. (1968). *Decision Analysis* - classic textbook.
4. Clemen, R. T., & Reilly, T. - *Making Hard Decisions*.
5. von Winterfeldt, D., & Edwards, W. - *Decision Analysis and Behavioral Research*.



