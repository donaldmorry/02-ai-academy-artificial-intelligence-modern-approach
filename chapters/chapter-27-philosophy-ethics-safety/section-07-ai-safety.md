# Section 27.7: AI Safety

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 27.7
> **Previous:** [Section 27.6 - Privacy and Surveillance](./section-06-privacy-and-surveillance.md)
> **Next:** [Section 27.8 - Governance](./section-08-governance.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Introduction

**AI safety** asks whether an AI system will continue to behave acceptably when it becomes capable, deployed, optimized, and exposed to messy incentives. AIMA frames the issue through rational agents: if an agent optimizes the wrong objective, observes the wrong state, or acts outside the intended boundary, intelligence can amplify the mistake.

> **In plain English:** Safety is not only "does the model work?" It is "does the whole agent keep pursuing the right goal under pressure, uncertainty, and misuse?"

This section connects the ethics chapter to technical mechanisms from earlier chapters: utility functions, MDPs, reinforcement learning, planning, uncertainty, and agent design. It also bridges forward to Course 4's [Modern LLM Engineering](https://github.com/Collaborative-ai/ai-academy-generative-deep-learning-systems/blob/main/chapters/chapter-14-conclusion/section-09-modern-llm-engineering.md), where alignment, RAG, tool use, and evaluation harnesses become production concerns.

---

## Three Safety Questions

| Question | AIMA connection | Engineering artifact |
|----------|-----------------|----------------------|
| What should the system optimize? | Utility theory, decision theory | Objective spec and forbidden outcomes |
| What does the system believe? | Probability, perception, uncertainty | Data audit, calibration, uncertainty estimate |
| What can the system do? | Agents, planning, action models | Capability boundary and approval policy |

Most failures combine all three. A recommender may optimize watch time, misread user welfare, and have broad influence over attention. A robot may optimize task completion, misperceive a human, and act physically before uncertainty is resolved.

---

## Specification Gaming

**Specification gaming** happens when a system satisfies the written objective while violating the intended objective.

| Domain | Written objective | Bad strategy |
|--------|-------------------|--------------|
| Cleaning robot | Maximize floor area cleaned | Push dirt under furniture or avoid hard rooms |
| Game agent | Maximize score | Exploit a simulator bug |
| Content recommender | Maximize engagement | Promote outrage or addictive loops |
| LLM assistant | Maximize helpful ratings | Overconfidently invent citations |

The formal issue is objective mismatch:

$$
U_{\text{proxy}}(s,a) \ne U_{\text{human}}(s,a)
$$
> **Readable form:** the measurable proxy reward can differ from the human value we actually care about

Good safety work makes the proxy visible, tests where it breaks, and adds constraints where the proxy is known to be incomplete.

---

## Reward Hacking in Sequential Decisions

In an MDP or RL setting, the agent chooses actions to maximize expected discounted reward:

$$
\pi^* = \arg\max_\pi \mathbb{E}_\pi\left[\sum_{t=0}^{\infty} \gamma^t R(s_t,a_t)\right]
$$
> **Readable form:** the optimal policy is the one with the highest expected discounted reward over time

If $R$ is misspecified, better optimization can produce worse real behavior. This is why safety reviews should inspect:

- **Reward source:** who defined it and what does it omit?
- **Reward loopholes:** can the agent get reward without doing the intended task?
- **State coverage:** does the training environment include rare but costly failures?
- **Action scope:** can the agent create irreversible harm while exploring?
- **Monitoring:** can deployment detect when reward and outcome diverge?

For a tabular grid world, reward hacking might be harmless. For a trading agent, medical triage model, or tool-using LLM, the same pattern can become high stakes.

---

## Distribution Shift

AI systems are trained on one distribution and deployed on another. AIMA's uncertainty chapters matter because the agent rarely observes the full world state.

| Shift type | Example | Safety risk |
|------------|---------|-------------|
| Covariate shift | New camera, lighting, or user population | Perception errors |
| Label shift | Disease prevalence changes | Miscalibrated predictions |
| Concept drift | Fraud pattern evolves | Old decision rule becomes unsafe |
| Adversarial shift | Inputs crafted to fool the model | Security failure |
| Capability shift | Model update adds tool-use ability | Old policy no longer bounds behavior |

Safety practice: monitor inputs, outputs, confidence, and downstream outcomes. A model that is accurate on last year's validation set can still be unsafe today.

---

## Alignment and Human Feedback

**Alignment** means the system's behavior remains compatible with human intentions, constraints, and values. For modern LLMs, alignment often uses supervised instruction tuning, preference data, RLHF, DPO-style preference optimization, red-teaming, and policy filters.

These methods are useful but incomplete:

| Method | Helps with | Does not solve |
|--------|------------|----------------|
| Instruction tuning | Following common user requests | Hidden misuse or rare edge cases |
| RLHF / preference tuning | Human-preferred style and refusals | Objective misspecification |
| RAG grounding | Missing or stale knowledge | Bad retrieval or prompt injection |
| Tool validation | Invalid actions | Wrong high-level plan |
| Red-team tests | Known attack patterns | Unknown future failures |

Alignment is therefore a system property, not a single training step. The Course 4 section on modern LLM engineering treats this as an evaluation and deployment workflow.

---

## Safety Case Template

For any high-impact AI system, write a compact safety case before deployment.

| Section | Required claim | Evidence |
|---------|----------------|----------|
| Intended use | The system is safe enough for these users and tasks | Scope statement and user stories |
| Out-of-scope use | The system should refuse or escalate these cases | Refusal tests and policy |
| Objective | The proxy aligns with the real outcome | Reward review and ablations |
| Data | Training and eval data represent deployment | Dataset card and bias checks |
| Robustness | Known shifts and attacks are tested | Stress tests and red-team results |
| Oversight | Humans can inspect and override | UI, logs, approval gates |
| Monitoring | Harmful drift can be detected | Metrics, alerts, rollback plan |

Do not treat this as paperwork. The safety case is a technical design document for finding missing assumptions.

---

## Worked Example: Hiring Screen

Suppose an organization builds a resume-screening model.

| Design choice | Unsafe version | Safer version |
|---------------|----------------|---------------|
| Objective | Maximize historical hire similarity | Predict job-relevant competencies with bias audit |
| Data | Past resumes and decisions only | Add structured labels, remove leakage, document gaps |
| Evaluation | Accuracy against old decisions | False-negative analysis by group and role |
| Action | Auto-reject low scores | Human review and appeal path |
| Monitoring | None after launch | Drift, selection rates, complaints, spot audits |

The fairness constraint from earlier sections is only one slice:

$$
P(\hat{Y}=1 \mid A=a) = P(\hat{Y}=1 \mid A=b)
$$
> **Readable form:** demographic parity requires equal positive prediction rates across protected groups

This constraint can conflict with other fairness definitions. A serious safety review states which definition is used, why, and what trade-off remains.

---

## Minimal Disparate-Impact Check

```python
def disparate_impact(rate_majority, rate_minority, threshold=0.8):
    if rate_majority <= 0:
        raise ValueError("Majority selection rate must be positive")
    ratio = rate_minority / rate_majority
    return {
        "ratio": ratio,
        "passes_rule_of_thumb": ratio >= threshold,
    }


print(disparate_impact(rate_majority=0.50, rate_minority=0.32))
```

This check is not a full fairness audit. It is an early warning that should trigger deeper investigation, stakeholder review, and legal guidance in real deployments.

---

## LLM Safety Bridge

Tool-using LLM agents add specific safety concerns:

| Risk | Example | Mitigation |
|------|---------|------------|
| Hallucinated authority | Model invents a policy or source | Require citations from retrieval |
| Prompt injection | Retrieved page says "ignore prior rules" | Treat retrieved text as data, not instructions |
| Unsafe tool use | Model sends email, deletes file, or spends money | Human approval for side effects |
| Memory contamination | Old user preference overrides current task | Audited memory writes and expiration |
| Evaluation gap | Demo works but edge cases fail | Scenario suite with failed-tool and misuse cases |

This is why Course 2's agent model still matters in modern systems: every LLM tool loop is an agent with percepts, state, actions, and a performance measure.

---

## Deployment Checklist

Before launch:

1. State the intended use and out-of-scope use.
2. Identify the proxy objective and at least three ways it can be gamed.
3. Test distribution shift, adversarial inputs, and rare high-cost failures.
4. Add approval gates for irreversible, high-stakes, or external side effects.
5. Log decisions, confidence, tool calls, and human overrides.
6. Define rollback for model, prompt, data, policy, and tool changes.
7. Create an incident response path with named owners.

---

## Common Mistakes

| Mistake | Why it fails |
|---------|--------------|
| Safety as final checklist | Design choices have already locked in unsafe incentives |
| One metric only | Optimizing a single score hides harm distribution |
| No baseline | A complex model may be less safe than a simple rule plus review |
| No refusal path | The system must answer even when it should escalate |
| No post-launch monitoring | Deployment changes the data-generating process |

---

## Lab Extension

Choose a model or agent from an earlier Course 2 lab. Write a one-page safety case:

1. Intended use and out-of-scope use.
2. Proxy objective and likely specification games.
3. Dataset or environment shift risks.
4. Human oversight point.
5. Monitoring metrics and rollback condition.

Then add one adversarial test case that should fail safely.

---

## Reflection Questions

1. Why can better optimization make a misspecified objective more dangerous?
2. Give one example of specification gaming outside reinforcement learning.
3. What is the difference between distribution shift and adversarial input?
4. Why is RLHF not a complete alignment solution?
5. What should a safety case prove before deployment?
6. How do tool-using LLM agents connect back to AIMA's rational-agent model?

---

**Previous:** [Section 27.6 - Privacy and Surveillance](./section-06-privacy-and-surveillance.md) · **Next:** [Section 27.8 - Governance](./section-08-governance.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/) Chapter 27.7
2. Amodei et al. (2016). Concrete Problems in AI Safety. *arXiv*.
3. Krakovna et al. (2020). Specification gaming examples in AI.
4. Jobin, A., Ienca, M., & Vayena, E. (2019). Global landscape of AI ethics guidelines. *Nature Machine Intelligence*.
5. [Course 4 Section 14.9 - Modern LLM Engineering](https://github.com/Collaborative-ai/ai-academy-generative-deep-learning-systems/blob/main/chapters/chapter-14-conclusion/section-09-modern-llm-engineering.md)

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
