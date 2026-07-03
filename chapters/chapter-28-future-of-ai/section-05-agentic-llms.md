# Section 28.5: Agentic LLMs

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 28.5
> **Previous:** [Section 28.4 - World Models](./section-04-world-models.md)
> **Next:** [Section 28.6 - Open Problems](./section-06-open-problems.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Introduction

An **agentic LLM** is a language-model-centered agent that can maintain task state, choose actions, call tools, read observations, and decide when to stop. In AIMA terms, the LLM is not the whole agent; it is the policy or controller inside an agent architecture.

> **In plain English:** A chat model answers once. An agentic LLM loops: decide, use a tool, observe what happened, update state, and decide again.

The idea connects modern LLM systems back to early course ideas: PEAS descriptions from [Chapter 02](../chapter-02-intelligent-agents/README.md), search from Chapters 03-04, planning from Chapters 10-11, uncertainty from Chapters 12-15, reinforcement learning from Chapters 16-17, and language models from [Chapter 24](../chapter-24-deep-learning-nlp/README.md).

For production engineering details, compare this AIMA framing with Course 4's [modern LLM engineering section](https://github.com/donaldmorry/04-ai-academy-generative-deep-learning-systems/blob/main/chapters/chapter-14-conclusion/section-09-modern-llm-engineering.md).

---

## Source-Aligned Summary

AIMA treats agents as systems that perceive, choose, and act. Agentic LLMs fit that frame when you separate five layers:

| Layer | Engineering question | Student deliverable |
|-------|----------------------|---------------------|
| Goal | What outcome should the agent optimize or satisfy? | PEAS description and refusal boundaries |
| State | What does the agent remember between steps? | State schema and update rule |
| Policy | How does the LLM choose the next action? | Prompt contract or model-call wrapper |
| Tools | What can the agent do to the world? | Tool schemas, permissions, and side-effect policy |
| Evaluation | What proves the loop works safely? | Task suite, traces, and failure analysis |

The practical habit is to treat the LLM as one component in a controlled loop, not as an unconstrained actor.

> **Readable form:** agentic LLM design turns a language model into a testable perceive-decide-act loop with explicit tools and stop conditions.

---

## Components of Intelligence

Perception, reasoning, learning, language, and action interact in complete agents. In an LLM agent, those components usually appear as software boundaries:

| Agent component | LLM-agent implementation |
|-----------------|--------------------------|
| Percepts | User messages, retrieved documents, file contents, API responses |
| Belief state | Scratch state, memory store, task checklist, retrieved evidence |
| Reasoning | Prompted model call, verifier model, symbolic checker, planner |
| Action | Tool call, code execution, database query, message draft |
| Learning | Offline fine-tuning, preference data, retrieval updates, evaluation logs |

---

## The Agent Loop

The minimal loop is:

1. Read the goal and current state.
2. Choose either a tool call or a final answer.
3. Validate the chosen action against the tool contract.
4. Execute the tool if allowed.
5. Record the observation.
6. Stop when the answer is sufficient or the budget is exhausted.

This is the LLM-era version of the agent program from Chapter 02.

---

## ReAct-Style Trace

A common pattern is **ReAct**: interleave short reasoning notes with actions and observations. The reasoning note should be a compact audit trail, not an unbounded private monologue.

Example: an internal support assistant is asked, "Can this customer get a refund?"

```text
Goal: decide refund eligibility for ticket #812.

Step 1
Reasoning note: Need order date, plan, and refund policy.
Action: get_ticket(ticket_id=812)
Observation: Customer bought Pro plan 9 days ago; asks for refund.

Step 2
Reasoning note: Need the active refund rule for Pro subscriptions.
Action: search_policy(query="Pro subscription refund window")
Observation: Pro subscriptions refundable within 14 days unless account violated terms.

Step 3
Reasoning note: Need account status before answering.
Action: get_account_status(customer_id=...)
Observation: Account in good standing.

Step 4
Final: Eligible for refund under the 14-day Pro policy; include policy link.
```

This is not magic. It is a partially observable sequential decision problem with a powerful language model filling in the policy.

---

## Tool Contracts

Tool use only becomes testable when tool calls are structured. Prefer a small JSON schema over free-form text:

```json
{
  "type": "tool_call",
  "tool": "search_policy",
  "arguments": {
    "query": "Pro subscription refund window"
  }
}
```

A controller should validate:

| Contract check | Example rejection |
|----------------|-------------------|
| Tool exists | `wire_money` is not in the allowed tool list |
| Arguments match schema | `ticket_id` must be an integer |
| Permission is allowed | model cannot delete records without approval |
| Output is typed | policy search returns documents, not a final decision |
| Side effects are gated | send-email and charge-card need human approval |

---

$$
\pi_\theta(a_t \mid g, s_t, o_t, T)
$$
> **Readable form:** the model policy chooses the next action from the goal, state, latest observation, and available tools

The state update is:

$$
s_{t+1} = \operatorname{Update}(s_t, a_t, o_{t+1})
$$
> **Readable form:** after each action, the agent records the observation and updates task state before choosing again

---

```python
from dataclasses import dataclass, field


@dataclass
class AgentState:
    goal: str
    observations: list[str] = field(default_factory=list)
    steps: int = 0


TOOLS = {
    "search_policy": lambda query: f"policy snippets for: {query}",
    "calculator": lambda expression: f"calculator result for: {expression}",
}


def validate_action(action):
    if action["type"] == "final":
        return action
    if action["type"] != "tool_call":
        raise ValueError("Action must be final or tool_call")
    if action["tool"] not in TOOLS:
        raise ValueError(f"Unknown tool: {action['tool']}")
    if not isinstance(action.get("arguments"), dict):
        raise ValueError("Tool arguments must be a dictionary")
    return action


def run_agent(goal, llm_decide, max_steps=5):
    state = AgentState(goal=goal)
    for _ in range(max_steps):
        action = validate_action(llm_decide(state))
        if action["type"] == "final":
            return action["answer"], state

        tool = TOOLS[action["tool"]]
        observation = tool(**action["arguments"])
        state.observations.append(observation)
        state.steps += 1

    return "Stopped: step budget exhausted", state
```
The code is intentionally small. A real controller would add authentication, call a real sandboxed calculator, log prompts and tool outputs, sandbox side effects, and score whether the final answer cites the observations it used.

---

## Summary

| Design choice | Practical default |
|---------------|-------------------|
| Tool format | Typed JSON objects validated by a controller |
| Stop rule | Maximum steps plus confidence or task-complete criterion |
| Memory | Explicit short-term state; audited long-term writes |
| Safety | Human approval for side effects and high-stakes actions |
| Evaluation | Final answer plus tool-call trace quality |
---

## Pitfalls

- Treating "agent" as a product label instead of a loop with state and action
- Giving the model broad tools before defining schemas, permissions, and rollback
- Letting retrieved text override system policy or safety constraints
- Measuring only final answer quality while ignoring invalid tool calls
- Allowing unbounded loops without cost, latency, or stop limits
- Confusing a plausible plan with verified execution

---

## Connections

- [Chapter 02](../chapter-02-intelligent-agents/README.md) - PEAS and task environments
- [Chapter 10](../chapter-10-knowledge-representation/README.md) - representations that constrain reasoning
- [Chapter 11](../chapter-11-automated-planning/README.md) - planning and execution
- [Chapter 24](../chapter-24-deep-learning-nlp/README.md) - transformers and language models
- [Chapter 27](../chapter-27-philosophy-ethics-safety/README.md) - safety, alignment, and accountability
- [Course 4 Section 14.9](https://github.com/donaldmorry/04-ai-academy-generative-deep-learning-systems/blob/main/chapters/chapter-14-conclusion/section-09-modern-llm-engineering.md) - RAG, LoRA, deployment, and modern LLM system design

---

## Worked Example

Design a study-planner agent for this course.

| PEAS element | Concrete choice |
|--------------|-----------------|
| Performance | Learner completes weekly sections, labs, and quizzes |
| Environment | Course markdown files, calendar, learner progress log |
| Actuators | Recommend next section, create task list, quiz learner |
| Sensors | Read progress log, read chapter README files, score quiz answers |

Tool set:

| Tool | Input | Output |
|------|-------|--------|
| `read_progress` | learner id | completed sections and weak topics |
| `read_module` | course/chapter path | section titles and prerequisites |
| `schedule_task` | title, duration, date | calendar entry id |
| `ask_quiz` | topic, difficulty | question and answer key |

Failure analysis:

- If `read_progress` is stale, the agent may recommend already completed work.
- If `schedule_task` has side effects, it should ask before changing the calendar.
- If the quiz generator cannot ground questions in a section file, it should refuse that topic or retrieve the file first.

---

## Implementation Notes

Separate the LLM policy, controller, tools, state store, and evaluator. Log prompts, validated tool calls, observations, model version, random seeds, and stop reason. Save traces for review, but redact private data before using traces as training examples.

---

## Historical Context

Early AI treated subfields independently; modern agents integrate perception, reasoning, learning, and language under the PEAS framework ([Chapter 02](../chapter-02-intelligent-agents/README.md)). LLM agents make that integration easier to prototype, but the old questions remain: what is the state, what actions are allowed, what counts as success, and what uncertainty is hidden?

---

## Deployment Checklist

Before production:

- Define allowed tools and forbidden side effects.
- Add human approval for irreversible or high-stakes actions.
- Test unavailable tools, malformed tool output, contradictory documents, and prompt injection inside retrieved text.
- Monitor tool-call error rate, loop length, refusal rate, latency, cost, and user correction rate.
- Define rollback for model, prompt, tool, and memory changes.
- Complete an ethics review for affected stakeholders.

---

## Lab Extension

Extend the Chapter lab by building a three-tool agent for a toy domain:

1. `retrieve_note(topic)` returns a short course note.
2. `calculate(expression)` solves numeric checks.
3. `finish(answer, citations)` returns the final response.

Run four ablations: no retrieval, no calculator, no step limit, and no citation requirement. Measure task success, invalid tool calls, average steps, and grounded citations.

---

## Applied Deep Dive

Use **Agentic LLMs** as a complete AIMA-style design exercise rather than a vocabulary item.

### Formulation Pass

1. Name the agent, environment, sensors, actuators, and performance measure.
2. State what is fully observable and what must be inferred.
3. Identify whether the task is episodic, sequential, deterministic, stochastic, single-agent, or multi-agent.
4. List the abstraction boundaries where the model deliberately ignores detail.

### Algorithm Pass

| Step | What to verify |
|------|----------------|
| Inputs | Units, coordinate frames, symbol vocabulary, or probability semantics |
| Update | Whether the computation preserves invariants such as normalization or consistency |
| Output | Whether the result is an action, belief state, plan, explanation, or metric |
| Complexity | What grows with states, actions, objects, observations, or time horizon |



### Failure Analysis

| Failure mode | Why it matters | Mitigation |
|--------------|----------------|------------|
| Wrong abstraction | The algorithm solves a clean problem, not the deployed one | Revisit the PEAS description and task environment |
| Hidden uncertainty | Deterministic code overcommits to noisy inputs | Track beliefs, confidence intervals, or fallback policies |
| Poor evaluation | A demo succeeds while edge cases fail | Use held-out scenarios and adversarial tests |
| Integration drift | Perception, planning, and control assumptions disagree | Log interfaces and validate each boundary |

### Mastery Task

Write a one-page technical memo for **Agentic LLMs**. Include the formal model, one algorithm choice, one rejected alternative, two measurable risks, and a test plan. This turns the source chapter from reading material into engineering judgment.

## Key Terms (Glossary)

- **tool use** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **ReAct** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **planning with LLMs** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. In an agentic LLM, what part is the policy and what part is the environment?
2. Why is a tool schema safer than free-form tool text?
3. What observation would prove the agent should revise its plan?
4. What side effects require human approval in your capstone idea?
5. How would you test for prompt injection inside retrieved documents?
6. Why does a step budget matter for both cost and safety?

---

**Previous:** [Section 28.4 - World Models](./section-04-world-models.md) · **Next:** [Section 28.6 - Open Problems](./section-06-open-problems.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/) Chapter 28.5
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Marcus, G. (2020). The next decade in AI: four steps towards robust AI. *arXiv*.
4. Yao et al. (2022). ReAct: Synergizing Reasoning and Acting in Language Models. *arXiv*.




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
