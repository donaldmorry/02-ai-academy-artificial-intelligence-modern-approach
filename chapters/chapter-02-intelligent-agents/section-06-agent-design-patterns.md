# Section 2.6: Agent Design Patterns

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 2 synthesis  
> **Previous:** [Section 2.5 - Learning Agents](./section-05-learning-agents.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Taxonomy to Engineering

[Sections 2.1-2.5](./section-01-agents-and-environments.md) built vocabulary: [percepts](../../GLOSSARY.md#percept), [performance measures](../../GLOSSARY.md#performance-measure), [PEAS](../../GLOSSARY.md#peas), environment properties, agent architectures, learning components. This section turns theory into **design patterns** - repeatable recipes mapping environment classes to agent structures and AI techniques.

Think of it as a **restaurant menu for agent architects**: the environment tells you whether you need soup (reflex), salad (model), entrée (planning), or dessert with wine pairing (utility + learning).

> **Readable form:** Match how hard the world is to how fancy your agent's brain needs to be - don't bring a SAT solver to a thermostat job.

Humorous analogy: sending a **utility-based planner** to solve a fully observable episodic spam label is like hiring a Michelin chef to butter toast. Technically capable, financially ruinous, late to breakfast.

---

## The Design Pipeline

Follow this order on every new agent project:

```
1. PEAS specification          ([Section 2.3](./section-03-the-nature-of-environments.md))
2. Classify six properties     (observable, agents, deterministic, ...)
3. Choose base architecture    ([Section 2.4](./section-04-structure-of-agents.md))
4. Select algorithms           (Chapters 03-22)
5. Add learning if needed      ([Section 2.5](./section-05-learning-agents.md))
6. Validate on performance measure ([Section 2.2](./section-02-good-behavior-and-rationality.md))
```

```python
def design_agent(peas, env_properties):
    arch = select_architecture(env_properties)
    algorithms = map_algorithms(env_properties, arch)
    if env_properties.unknown_or_changing_dynamics:
        arch = wrap_with_learning(arch)
    return AgentProgram(architecture=arch, algorithms=algorithms, peas=peas)
```

> **Readable form:** Write the spec, classify the world, pick a brain structure, plug in algorithms, add learning if the world shifts, then test against your scorecard.

---

## Pattern 1: Reactive Control

**Environment profile:**

- Fully or nearly fully observable
- Episodic or short horizon
- Dynamic (must act quickly)
- Low cost of suboptimal action

**Architecture:** Simple reflex or **model-based reflex** with lightweight state

**Examples:** Emergency braking, basic thermostat, collision avoidance reflex

```python
def reactive_pattern(percept, rules):
    return rules.match(percept)  # O(1) lookup, microseconds
```

**Techniques:** Hand-crafted rules, lookup tables, shallow [neural networks](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-02-machine-learning-vs-ai-vs-deep-learning.md) for perception → reflex

**Avoid:** Deep multi-step planning - too slow for dynamic layer

---

## Pattern 2: Deliberative Planning

**Environment profile:**

- Sequential
- Mostly deterministic or well-modeled uncertainty
- Static or semidynamic (time to plan)
- Clear goal states

**Architecture:** Goal-based (model-based + planner)

**Examples:** Logistics routing on known maps, puzzle solving, classical robotics pick-and-place in structured warehouses

**Techniques:** BFS, A* (Chapter 03), STRIPS planners (Chapter 11)

$$
\text{plan} = \text{SEARCH}(s_0, g, \text{successors}, \text{cost})
$$
> **Readable form:** Search from start state to goal state using known transition rules, minimizing cost.

**Course 1 link:** Unlike batch prediction, planning **generates action sequences** - the agent chooses **what to do**, not just **what label fits**.

---

## Pattern 3: Belief-State Reasoning

**Environment profile:**

- **Partially observable**
- Sequential
- Sensors noisy or incomplete

**Architecture:** Model-based with **belief tracking** → goal or utility layer on beliefs

**Examples:** Robot localization, medical diagnosis, poker, Wumpus World (Chapter 07)

**Techniques:** Hidden Markov models, Kalman filters (Chapter 14), particle filters, POMDPs (Chapter 17)

```python
def belief_update(belief, percept, action, models):
    # Predict: integrate transition model
    belief = models.transition.predict(belief, action)
    # Correct: integrate sensor model
    belief = models.sensor.update(belief, percept)
    return belief
```

> **Readable form:** Keep a probability distribution over possible true states, update it when you act and when you sense.

**Key insight:** planning happens in **belief space**, not true state space - because the agent never knows true state for certain.

---

## Pattern 4: Expected Utility Maximization

**Environment profile:**

- Tradeoffs among conflicting objectives
- Stochastic outcomes
- Preferences need numeric weighting

**Architecture:** Utility-based agent

**Examples:** Autonomous taxi, treatment selection, financial portfolio agents

**Techniques:** Decision networks (Chapter 16), MDP value iteration (Chapter 17)

[Section 2.2](./section-02-good-behavior-and-rationality.md) rationality formalizes **why**; this pattern formalizes **how** when multiple goals collide.

---

## Pattern 5: Competitive Multiagent

**Environment profile:**

- **Multiagent**, competitive or mixed
- Opponent actions affect payoff
- Often partially observable

**Architecture:** Utility-based + **opponent model** + game-tree search

**Examples:** Chess, Go, negotiation bots, ad auctions

**Techniques:** Minimax, alpha-beta (Chapter 05), MCTS, game-theoretic equilibria (Chapter 18)

From [Section 1.4](../chapter-01-introduction/section-04-state-of-the-art.md), AlphaGo combined **pattern learning** with **MCTS** - hybrid of Patterns 5 and 7.

---

## Pattern 6: Learning-Centric

**Environment profile:**

- Unknown or changing dynamics
- Large percept spaces (vision, language)
- Designer cannot specify model/rules

**Architecture:** Learning agent wrapping any base pattern

**Examples:** Spam filters, recommendation systems, modern game agents, LLM assistants

**Techniques:** [Supervised learning](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-03-supervised-vs-unsupervised-learning.md) (Chapter 19), deep learning (Chapter 21), reinforcement learning (Chapter 22)

| Learning type | When | Course 1 anchor |
|-------------|------|-----------------|
| Supervised | Labels available | Classification/regression chapters |
| Unsupervised | Structure discovery | [k-means](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md) |
| Reinforcement | Sequential rewards | Preview Chapter 22 |

---

## Pattern 7: Layered Hybrid (Production Default)

Real deployed agents **combine patterns**:

| Layer | Speed | Pattern | Example (self-driving) |
|-------|-------|---------|------------------------|
| Reactive | ms | Reflex | AEB brake on lidar obstacle |
| Tactical | 100 ms | Model-based | Lane tracking, object tracking |
| Strategic | seconds | Goal/utility planning | Route selection, merge negotiation |
| Learning | offline/online | Learning element | Improve perception from fleet data |

```
┌─────────────────────────────────────┐
│  Strategic planner (utility/goals)  │
├─────────────────────────────────────┤
│  Tactical model-based control       │
├─────────────────────────────────────┤
│  Reactive safety reflexes           │
└─────────────────────────────────────┘
         ▲ percepts    actions ▼
```

> **Readable form:** Fast layers handle emergencies; slow layers handle strategy; learning improves all layers over time.

Brooks argued bottom layers alone suffice for some tasks; modern practice uses **both** reactivity and deliberation.

---

## Master Mapping Table

| If dominant property is... | Start with pattern | Primary chapters |
|---------------------------|-------------------|-----------------|
| Fully obs. + episodic | Reactive / batch ML | 19, Course 1 |
| Sequential + deterministic goals | Deliberative planning | 03, 11 |
| Partial observability | Belief-state | 12, 14, 17 |
| Conflicting objectives | Expected utility | 16, 17 |
| Competitive opponents | Multiagent games | 05, 18 |
| Unknown dynamics | Learning-centric | 19-22 |
| Real-world deployment | Layered hybrid | All of the above |

---

## Environment Hardness vs. Architecture Power

```
Complexity of environment
         ▲
         │     ┌── utility + learning + multiagent
         │    ╱
         │   ╱  goal-based + belief
         │  ╱
         │ ╱   model-based reflex
         │╱
         └──────────────────────► Power of architecture
```

Under-powering causes failure (reflex agent in poker). Over-powering wastes latency and engineering (full POMDP for thermostat).

---

## Design Anti-Patterns

| Anti-pattern | Symptom | Fix |
|--------------|---------|-----|
| **Oracle assumption** | Train on full state, deploy on partial percepts | Belief-state pattern |
| **Metric myopia** | Optimizes proxy, fails true $P$ | Revisit [performance measure](../../GLOSSARY.md#performance-measure) |
| **Planner-only** | Slow reactions in dynamic world | Add reactive layer |
| **Data without critic** | Offline model never closes loop | Add learning agent critic |
| **Exploration in production** | Problem generator breaks safety | Sandbox exploration |

[Section 1.5](../chapter-01-introduction/section-05-risks-and-benefits-of-ai.md) catalogs societal versions of metric myopia.

---

## Worked Mapping: Warehouse Robot

[PEAS](../../GLOSSARY.md#peas) from Chapter 02 lab:

| Property | Value | Pattern choice |
|----------|-------|----------------|
| Observable | Partially (occlusion, other robots) | Belief-state |
| Agents | Multiagent cooperative | Coordination + planning |
| Deterministic | Mostly, with slip | Stochastic planning optional |
| Episodic | Sequential pick routes | Deliberative |
| Static | Semidynamic | Anytime planning |
| Discrete | Grid + continuous motion | Hybrid |

**Recommended stack:** A* on warehouse graph (tactical) + localization filter (belief) + fleet coordination (multiagent) + learning from pick times (learning element).

---

## Decision Checklist (Printable)

Before implementation, answer:

1. ☐ PEAS written and reviewed with stakeholders?
2. ☐ Six properties classified with evidence?
3. ☐ [Rationality](./section-02-good-behavior-and-rationality.md) - is $P$ aligned with real goals?
4. ☐ Minimum architecture identified (not maximum)?
5. ☐ Latency budget matches environment dynamics?
6. ☐ Learning needed, or hand-model sufficient?
7. ☐ Safety reflexes for irreversible actions?
8. ☐ Evaluation simulates **deployment** percepts, not oracle state?

---

## Looking Ahead

[Section 2.7](./section-07-case-studies.md) applies these patterns to vacuum world, taxi, and medical diagnosis - concrete walkthroughs before Chapter 03's search algorithms supply planning tools.

The [course roadmap](../chapter-01-introduction/section-06-course-roadmap.md) shows how Chapter 02 anchors Parts II-V: every technique is **in service of a [rational agent](../../GLOSSARY.md#rational-agent)** in a specified environment.

---

## Reflection Questions

1. Which design pattern best describes GitHub Copilot? Can you identify layers?
2. A fully observable, deterministic, episodic environment - is learning ever justified?
3. Why do production self-driving stacks rarely use a single monolithic architecture?
4. Map each Course 1 chapter (01-03) to the closest design pattern.

---

**Next:** [Section 2.7 - Case Studies](./section-07-case-studies.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 2. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Stanford CS221 - designing rational agents. [Course site](https://stanford-cs221.github.io/)
3. Gamma, E., et al. (1994). *Design Patterns* - analogical inspiration for agent patterns. [Addison-Wesley](https://www.pearson.com/en-us/subject-catalog/p/design-patterns-elements-of-reusable-object-oriented-software/P200000003332)
4. Thrun, S., et al. - Stanley DARPA Grand Challenge architecture. [Stanford Racing Team](https://stanford.edu/~cpiech/cs221/apps/drivingSimulator.html)
5. Browne, C., et al. (2012). "A Survey of Monte Carlo Tree Search Methods." [IEEE TCIAIG](https://ieeexplore.ieee.org/document/6145622) - hybrid game agent pattern



