# Section 2.3: The Nature of Environments

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 2.3  
> **Previous:** [Section 2.2 - Good Behavior and Rationality](./section-02-good-behavior-and-rationality.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Why Classify Environments?

Before choosing algorithms, AI engineers characterize the **task environment** - the problem specification independent of any particular agent design. Russell and Norvig compare this to a **product requirements document** before writing code. Misclassify the environment and you deploy a reflex thermostat to play poker.

[Section 2.2](./section-02-good-behavior-and-rationality.md) defined rationality relative to a [performance measure](../../GLOSSARY.md#performance-measure). This section asks: **what world is the agent rational *in*?** The answer shapes architecture ([Section 2.4](./section-04-structure-of-agents.md)), algorithm choice (Chapters 03-22), and engineering budget.

> **Readable form:** Describe the job (PEAS) and the world's difficulty (six properties) before picking how smart your agent needs to be.

Humorous analogy: environment classification is like reading the **weather forecast before packing**. Sunny + static beach → sunscreen and a book. Blizzard + dynamic mountain → layers, GPS, and possibly a rescue beacon. Same human, different environment, different kit.

---

## PEAS: Task Environment Specification

[PEAS](../../GLOSSARY.md#peas) is a checklist for describing a task environment:

| Letter | Component | Question to answer |
|--------|-----------|-------------------|
| **P** | Performance measure | How is success scored? |
| **E** | Environment | What is external to the agent? |
| **A** | Actuators | How can the agent act? |
| **S** | Sensors | What can the agent perceive? |

### Example: Autonomous Taxi

| PEAS | Specification |
|------|---------------|
| **P** | Safe, legal, comfortable trip; minimize time; maximize passenger satisfaction and profit |
| **E** | Roads, traffic, pedestrians, weather, passengers, other vehicles |
| **A** | Steering, acceleration, braking, signaling, horn, display, verbal communication |
| **S** | Cameras, lidar, GPS, speedometer, engine sensors, microphones, touchscreen |

### Example: Medical Diagnosis System

| PEAS | Specification |
|------|---------------|
| **P** | Correct diagnosis, effective treatment, minimize harm and cost |
| **E** | Patient body, disease processes, hospital workflow, regulations |
| **A** | Questions to patient/staff, test orders, treatment recommendations, referrals |
| **S** | Symptoms reported, lab results, imaging, medical records, vital signs |

### Example: Spam Filter

| PEAS | Specification |
|------|---------------|
| **P** | Maximize correct classification; minimize false positives (lost real mail) |
| **E** | Email stream, user behavior, evolving spam tactics |
| **A** | Label as spam/ham, quarantine, update filters |
| **S** | Message text, headers, attachments, metadata |

Compare to [Course 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/section-01-classification-fundamentals.md): PEAS adds **actuators** and **environment dynamics** around the classifier. A spam filter that only outputs labels is half an agent; one that **acts** on mailboxes is complete.

```python
# PEAS as a design document (illustrative)
TASK = {
    "performance": "maximize dirt_removed / time, penalize collisions",
    "environment": "grid rooms with dirt, walls, optional charging dock",
    "actuators": ["move_forward", "turn_left", "turn_right", "suck"],
    "sensors": ["location", "contents_of_current_cell"],
}
```

> **Readable form:** PEAS is a four-line spec sheet: what success means, what's out there, what you can do, what you can see.

Full case studies in [Section 2.7](./section-07-case-studies.md).

---

## Six Properties of Task Environments

Russell and Norvig classify environments along **six independent dimensions**. Real tasks are often mixed; the art is identifying which properties **dominate** design.

### 1. Fully Observable vs. Partially Observable

**Fully observable:** sensors detect all relevant aspects of the current state.  
**Partially observable:** sensors miss critical information.

| Fully observable | Partially observable |
|------------------|----------------------|
| Chess (if board fully visible) | Poker (opponents' cards hidden) |
| Checkers with perfect vision | Autonomous driving (occluded pedestrians) |
| SQL query optimizer (full schema visible) | Medical diagnosis (hidden internal disease) |

If partially observable, the agent must **track internal state** - a **model-based** architecture ([Section 2.4](./section-04-structure-of-agents.md)). [Percepts](../../GLOSSARY.md#percept) alone are insufficient.

### 2. Single-Agent vs. Multiagent

**Single-agent:** performance depends only on the agent's actions (other entities can be treated as part of environment).  
**Multiagent:** other agents' actions matter strategically.

| Single-agent | Multiagent |
|--------------|------------|
| Crossword puzzle | Chess, negotiation |
| Vacuum in empty house | Taxi in traffic |
| Solo maze solving | Auction bidding |

Subcategories:

- **Competitive** - zero-sum; one's gain is another's loss (chess)
- **Cooperative** - shared goals (warehouse robots coordinating)
- **Mixed** - partial cooperation (traffic: avoid collision yet compete for lane)

Chapter 18 formalizes multiagent decision-making.

### 3. Deterministic vs. Stochastic (Nondeterministic)

**Deterministic:** next state fully determined by current state and action.  
**Stochastic:** actions have probabilistic outcomes.

| Deterministic | Stochastic |
|---------------|------------|
| Chess (given move) | Dice games |
| Eight-puzzle | Robot slip on wet floor |
| File system commands | Network packet loss |

Stochastic environments demand **probability** (Chapters 12-18) and often **contingency planning** (Chapter 11).

### 4. Episodic vs. Sequential

**Episodic:** each action/decision is independent; current episode does not affect next.  
**Sequential:** current decisions affect future states and rewards.

| Episodic | Sequential |
|----------|------------|
| Image classification (one image) | Chess, robotics navigation |
| Spam label per email (often) | Portfolio management |
| [k-NN](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-05-supervised-learning-k-nearest-neighbors.md) on fixed rows | Customer support over conversation |

Sequential environments require **planning** and **memory** - mistakes compound.

### 5. Static vs. Dynamic (Semidynamic)

**Static:** environment does not change while agent deliberates.  
**Dynamic:** environment can change during deliberation.  
**Semidynamic:** environment frozen but [performance measure](../../GLOSSARY.md#performance-measure) still ticks (e.g., chess clock).

| Static | Dynamic |
|--------|---------|
| Crossword | Taxi driving |
| Offline planning on map snapshot | Real-time strategy games |
| Batch ML training | Live fraud detection |

Dynamic environments need **fast decisions**, interrupt handling, and often **anytime algorithms** (return best answer so far).

### 6. Discrete vs. Continuous

**Discrete:** finite or countably infinite states and actions.  
**Continuous:** states, time, or actions vary continuously.

| Discrete | Continuous |
|----------|------------|
| Chess board positions | Robot joint angles |
| Grid worlds | Vehicle steering angle |
| Token sequences (finite vocab steps) | Physics simulation time |

Continuous control appears in robotics (Chapter 26) and deep RL (Chapter 22).

---

## Environment Property Summary Table

| Property | Question | Harder pole |
|----------|----------|-------------|
| Observability | Can you see everything that matters? | Partially observable |
| Agents | Do other agents strategize against you? | Multiagent competitive |
| Determinism | Are outcomes predictable? | Stochastic |
| Episodicity | Do decisions affect the future? | Sequential |
| Dynamics | Does world change while you think? | Dynamic |
| Continuity | Infinite states/actions? | Continuous |

**AIMA's meta-section:** the **hardest real-world problems** are partially observable, multiagent, stochastic, sequential, dynamic, and continuous. Autonomous driving checks every box. No wonder it remains unsolved in the general case.

---

## Worked Classification: Chess vs. Poker

| Property | Chess | Poker |
|----------|-------|-------|
| Observable | Fully (perfect info variant) | Partially |
| Agents | Multiagent competitive | Multiagent competitive |
| Deterministic | Deterministic | Stochastic (shuffle, hidden cards) |
| Episodic | Sequential | Sequential |
| Static | Semidynamic (clock) | Dynamic (betting rounds) |
| Discrete | Discrete | Discrete |

Chess engines lean on **search** (Chapter 05). Poker bots need **probability**, **opponent modeling**, and **bluffing** - far harder in practice despite similar board-game aesthetics.

---

## Worked Classification: Course 1 ML Tasks

| Task | Typical environment class |
|------|---------------------------|
| House price [regression](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-02-regression-models/section-02-linear-regression.md) | Single-agent, episodic, static, often batch |
| Image classification | Episodic per image; environment static at inference |
| Online ad click prediction | Sequential, dynamic, stochastic |
| [Clustering](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-04-unsupervised-learning-k-means-clustering.md) | Offline; not really an agent environment |

Course 1 focused on **episodic prediction**. Course 2 expands to **embedded agents** in rich worlds.

---

## Mapping Properties to Techniques (Preview)

| If environment is... | Often need... | Chapter |
|---------------------|---------------|--------|
| Partially observable | Belief states, sensors fusion | 04, 12, 17 |
| Multiagent competitive | Game theory, minimax | 05, 18 |
| Stochastic | Probability, MDPs | 12, 17 |
| Sequential | Planning, RL | 11, 22 |
| Dynamic | Real-time search, reactive layers | 04, 26 |
| Continuous | Control theory, function approximation | 22, 26 |

[Section 2.6](./section-06-agent-design-patterns.md) systematizes these mappings into design patterns.

---

## PEAS Exercise Template

When starting a project, fill this template before coding:

```markdown
## PEAS: [Project Name]

**Performance:** [measurable criteria + penalties]

**Environment:** [entities, dynamics, what is NOT the agent]

**Actuators:** [list actions with granularity]

**Sensors:** [list percepts, rate, noise]

## Environment properties:
- Observable: fully / partially - [why]
- Agents: single / multi - [who else]
- Deterministic: yes / no - [sources of randomness]
- Episodic: yes / no - [dependencies across time]
- Static: yes / dynamic - [what changes while deciding]
- Discrete: yes / continuous - [state/action space]
```

---

## Common Mistakes

**Treating simulation as fully observable when sensors are partial in deployment.** Training on perfect state estimates fails on noisy real robots.

**Ignoring multiagent dynamics in markets or traffic.** Other participants adapt - your agent is not alone.

**Assuming episodic when feedback is delayed.** Credit assignment over long horizons needs RL, not one-shot classification.

---

## Reflection Questions

1. Write a PEAS description for a smart-home thermostat that learns occupant preferences.
2. Classify the environment of a [random forest](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-02-regression-models/section-04-ensemble-methods-random-forests-and-gradient-boosting.md) predicting churn from monthly snapshots. Is it an agent problem?
3. Which single property makes autonomous driving hardest? Defend your choice.
4. Is a two-player video game always multiagent? When could it be treated as single-agent?

---

**Next:** [Section 2.4 - Structure of Agents](./section-04-structure-of-agents.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 2.3. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Stanford CS221 - Task environments and PEAS. [Course site](https://stanford-cs221.github.io/)
3. AIMA Python - `agents.py` environment examples. [GitHub repository](https://github.com/aimacode/aima-python/blob/master/agents.py)
4. Brooks, R. A. (1991). "Intelligence without representation." *Artificial Intelligence* - reactive agents in dynamic worlds. [DOI link](https://doi.org/10.1016/0004-3702(91)90053-M)
5. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* - agent-environment loop. [Free online edition](http://incompleteideas.net/book/the-book.html)