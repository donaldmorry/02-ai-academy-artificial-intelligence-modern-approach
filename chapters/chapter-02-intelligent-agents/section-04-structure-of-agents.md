# Section 2.4: Structure of Agents

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 2.4  
> **Previous:** [Section 2.3 - The Nature of Environments](./section-03-the-nature-of-environments.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Agent Function to Agent Architecture

[Section 2.1](./section-01-agents-and-environments.md) defined the agent function $f : \mathcal{P}^* \rightarrow \mathcal{A}$. Implementing $f$ naïvely requires infinite memory. Real agents use **internal structure** - state variables, models, goals, utilities - to compress the mapping from percept history to action.

Russell and Norvig present a **ladder of increasing sophistication**:

1. Simple reflex agents  
2. Model-based reflex agents  
3. Goal-based agents  
4. Utility-based agents  

Each rung adds representational power to handle richer [PEAS](../../GLOSSARY.md#peas) environments ([Section 2.3](./section-03-the-nature-of-environments.md)). [Learning agents](./section-05-learning-agents.md) wrap any of these with components that improve from experience.

> **Readable form:** Agent architectures are like career levels - intern follows rules, analyst builds a mental model, manager pursues goals, executive weighs tradeoffs with a scorecard.

Humorous analogy: a **simple reflex agent** is the friend who always says "bring a jacket" when you mention leaving the house - no memory of whether it's July in Phoenix. A **utility-based agent** checks the forecast, your calendar, and how much you hate being sweaty.

---

## Simple Reflex Agents

**Idea:** select actions based **only on the current percept**, using condition-action rules.

$$
a_t = \text{rule}(\text{percept}_t)
$$
> **Readable form:** If you see X right now, do Y - ignore everything that happened before.

```python
# Simple reflex vacuum agent (two-cell world)
def simple_reflex_vacuum(percept):
    location, contents = percept["location"], percept["contents"]
    if contents == "Dirty":
        return "Suck"
    elif location == "A":
        return "MoveRight"
    else:
        return "MoveLeft"
```

### When Simple Reflex Works

- **Fully observable** environments where current percept summarizes everything needed
- **Episodic** tasks with no hidden state
- Fast reaction requirements (low latency)

### When It Fails

**Partial observability:** the percept does not reveal hidden state. A vacuum that cannot see the other room wanders randomly without a map.

**Sequential dependencies:** current percept omits history needed for optimal play. You cannot play tic-tac-toe well by looking at only the latest mark without the full board (unless the percept encodes the full board each time - then the board *is* the state).

---

## Model-Based Reflex Agents

**Idea:** maintain an **internal state** updated by a **transition model** (how the world evolves) and a **sensor model** (how percepts relate to state).

```
internal_state = UPDATE(internal_state, percept, models)
action = rule(internal_state)
```

> **Readable form:** Keep a best guess of how the world really is, update it when you sense something new, then apply rules to that guess.

```python
def model_based_reflex(percept, state, transition_model, sensor_model):
    # Belief update (conceptual; Chapter 12 makes this probabilistic)
    state = transition_model.predict(state, last_action)
    state = sensor_model.correct(state, percept)
    action = reflex_rules(state)
    return action, state
```

### What the Model Contains

| Model piece | Answers |
|-------------|---------|
| **Transition model** | If I am in state $s$ and do $a$, what happens? |
| **Sensor model** | If true state is $s$, what percept do I expect? |
| **State representation** | Variables summarizing unobserved history |

A robot with lidar builds a **map** (state) even when walls are not currently visible - classic model-based behavior.

### Relation to Course 1

[Supervised learning](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-03-supervised-vs-unsupervised-learning.md) models map features to labels without explicit world dynamics. A model-based agent adds **temporal structure**: "if I believe the patient has infection $s$, ordering test $a$ may change my beliefs."

---

## Goal-Based Agents

**Idea:** combine state information with **goals** (desired outcomes) and **search/planning** to choose actions that achieve them.

$$
a^* = \text{PLAN}(\text{state}, \text{goal})
$$
> **Readable form:** Given where you think you are and where you want to be, search for a sequence of actions to get there.

```python
def goal_based_agent(percept, state, goal, planner):
    state = update_state(state, percept)
    plan = planner(state, goal)          # Chapter 03 algorithms live here
    return plan[0] if plan else "Wait"
```

### Goals vs. Performance Measures

| Concept | Role |
|---------|------|
| **Goal** | Binary or explicit target state ("at airport," "board cleared") |
| **[Performance measure](../../GLOSSARY.md#performance-measure)** | Numeric scoring of any outcome |

Goals factor planning: once you specify **goal state $g$**, search finds paths to $g$. Performance measures may prefer **among multiple ways** to reach $g$ - leading to utilities.

### When Goals Help

- **Sequential** environments where you must chain actions
- **Partially observable** worlds once state is tracked
- Domains with clear achievement conditions (deliver package, prove theorem)

Chapters 03 and 11 develop planners; logical agents (Chapter 07) represent goals as formulas.

---

## Utility-Based Agents

**Idea:** when goals are insufficient - multiple conflicting objectives, tradeoffs, uncertainty - assign **utilities** $U(s)$ or $U(s, a)$ to states/actions and maximize **expected utility**.

$$
a^* = \arg\max_a \mathbb{E}[U \mid a, \text{state}]
$$
> **Readable form:** Score every possible outcome on a happiness scale, pick the action with the best average score given uncertainty.

| Situation | Why goals fail | Utility helps |
|-----------|----------------|---------------|
| Taxi routing | Many routes reach airport | Prefer faster, smoother, cheaper |
| Medical treatment | Cure vs. side effects | Trade quality-adjusted life years |
| Game with risk | Win vs. safe draw | Weight expected points |

```python
def utility_based_agent(state, actions, utility_fn, belief):
    best_a, best_u = None, float("-inf")
    for a in actions:
        expected_u = sum(
            belief(s) * utility_fn(s, a) for s in possible_states
        )
        if expected_u > best_u:
            best_u, best_a = expected_u, a
    return best_a
```

Chapter 16 formalizes utility theory; Chapter 17 adds sequential decision-making (MDPs).

### Connection to [Rational Agents](../../GLOSSARY.md#rational-agent)

[Section 2.2](./section-02-good-behavior-and-rationality.md) defined rationality via [performance measures](../../GLOSSARY.md#performance-measure). Utility functions **operationalize** those measures inside the agent program - the bridge from philosophy to arithmetic.

---

## Architecture Comparison Diagram

```
Percept
   │
   ▼
┌──────────────────────────────────────────────────┐
│  STATE (optional) ◄── models ◄── percept         │
│     │                                             │
│     ├──► Reflex rules ──────────► Action         │
│     ├──► + Goal ──► Planner ────► Action         │
│     └──► + Utility ─► EU max ───► Action         │
└──────────────────────────────────────────────────┘
```

| Architecture | Needs state? | Needs goals? | Needs utility? | Example |
|--------------|--------------|--------------|----------------|---------|
| Simple reflex | No | No | No | Thermostat |
| Model-based reflex | Yes | No | No | Robot vacuum with map |
| Goal-based | Yes | Yes | No | Route planner |
| Utility-based | Yes | Optional | Yes | Autonomous taxi |

---

## Hybrid and Layered Architectures

Real systems **stack** layers:

1. **Reactive layer** - fast reflexes (emergency brake)
2. **Deliberative layer** - slow planning (route optimization)
3. **Learning layer** - updates models from data ([Section 2.5](./section-05-learning-agents.md))

Brooks's subsumption architecture (1980s robotics) prioritized bottom layers - intelligence without explicit global models. Modern self-driving stacks blend both: neural nets for perception (model building), planners for trajectories (goal/utility).

---

## Choosing an Architecture

Use [Section 2.3](./section-03-the-nature-of-environments.md) properties:

| Environment challenge | Minimum architecture |
|-----------------------|---------------------|
| Fully obs., episodic | Simple reflex may suffice |
| Partial observability | Model-based |
| Long-horizon consequences | Goal-based planning |
| Tradeoffs under uncertainty | Utility-based |
| Unknown dynamics | Learning agent on top |

[Section 2.6](./section-06-agent-design-patterns.md) tabulates full design patterns.

---

## Worked Example: Wumpus World Preview

Chapter 07's Wumpus World is **partially observable**, **sequential**, **deterministic** (in the standard formulation). A simple reflex agent dies quickly. Survivors maintain a **belief state** (model-based), pursue **safe gold extraction** (goal), and eventually reason about **risk vs. reward** (utility). The pedagogical arc mirrors this section's ladder.

---

## From Classifiers to Utility-Based Agents

Your [logistic regression](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/section-01-classification-fundamentals.md) outputs $P(y{=}1 \mid x)$. Hook it to **asymmetric costs**:

\text{action} = \begin{cases}
\text{approve} & \text{if } P(\text{fraud}) \cdot C_{\text{fraud}} < P(\text{legit}) \cdot C_{\text{false\_block}} \\
\text{deny} & \text{otherwise}
\end{cases}
> **Readable form:** Approve when expected cost of fraud is less than expected cost of wrongly blocking a good customer.

Same model, **utility-based** decision layer - a pattern production ML teams use daily.

---

## Reflection Questions

1. Is a chess program that searches to checkmate goal-based, utility-based, or both?
2. What internal state must a lane-keeping assist system maintain on a highway?
3. Why does a goal-based medical agent still need a performance measure beyond "cure patient"?
4. Draw the simplest architecture adequate for a smoke alarm. Justify.

---

**Next:** [Section 2.5 - Learning Agents](./section-05-learning-agents.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 2.4. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Stanford CS221 - Agent types lecture materials. [Course site](https://stanford-cs221.github.io/)
3. AIMA Figure 2.9-2.13 - agent diagrams. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
4. Brooks, R. A. - subsumption architecture. [MIT publication list](https://people.csail.mit.edu/brooks/)
5. Thrun, S., Burgard, W., & Fox, D. (2005). *Probabilistic Robotics* - model-based mobile robots. [Publisher page](https://mitpress.mit.edu/9780262201629/probabilistic-robotics/)


