# Section 2.7: Case Studies

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 2 examples  
> **Previous:** [Section 2.6 - Agent Design Patterns](./section-06-agent-design-patterns.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Why Case Studies?

Theory without examples is a **manual for a machine you've never seen**. Russell and Norvig revisit three canonical domains throughout AIMA - vacuum world, autonomous taxi, medical diagnosis - because they span the complexity spectrum from toy grid worlds to safety-critical reasoning.

This section applies everything from Chapter 02: [PEAS](../../GLOSSARY.md#peas), environment properties, agent architectures ([Section 2.4](./section-04-structure-of-agents.md)), learning ([Section 2.5](./section-05-learning-agents.md)), and design patterns ([Section 2.6](./section-06-agent-design-patterns.md)). Each case ends with **what we would build next** in later chapters.

> **Readable form:** Three stories - a robot vacuum, a taxi, a doctor's assistant - showing how the same agent framework scales from homework problem to real life.

Humorous analogy: vacuum world is **training wheels**; the taxi is **city traffic**; medical diagnosis is **open-heart surgery on a unicycle** - same balance principle, very different stakes.

---

## Case Study 1: Vacuum World

The vacuum world is AIMA's **hello world** for agents - a simple grid where a vacuum moves and cleans dirt.

### PEAS Specification

| PEAS | Vacuum world (standard) |
|------|-------------------------|
| **P** | Maximize clean squares per unit time; minimize energy; penalize infinite loops |
| **E** | Grid with dirt, walls; optional charging station |
| **A** | Move (up/down/left/right), Suck, optional NoOp |
| **S** | Current location, contents of current cell (Clean/Dirty) |

### Environment Properties

| Property | Classification | Notes |
|----------|----------------|-------|
| Observable | Fully (in basic variant) | Agent knows location + cell status |
| Agents | Single | Empty house assumption |
| Deterministic | Yes | Actions have predictable effects |
| Episodic | Sequential | Past moves affect which cells remain dirty |
| Static | Yes | Dirt does not reappear while deciding |
| Discrete | Yes | Finite grid |

### Agent Progression

**Simple reflex** (from [Section 2.4](./section-04-structure-of-agents.md)):

```python
def vacuum_reflex(percept):
    loc, contents = percept["location"], percept["contents"]
    if contents == "Dirty":
        return "Suck"
    return "Move" if loc == "A" else "Move"  # arbitrary wander
```

Works on **one-cell** worlds. On multi-cell grids without memory, the agent **revisits** clean cells forever - poor [performance measure](../../GLOSSARY.md#performance-measure) score.

**Model-based improvement:** track `internal_state = {"A": "Clean", "B": "Dirty", ...}` updated after each move/suck.

**Goal-based:** goal = all cells Clean; plan shortest tour (Chapter 03 search).

**Performance:** for $n$ cells, optimal cleaning is $O(n)$ with a map; reflex without state can be $O(\infty)$.

$$
\text{Score} = \frac{\text{cells cleaned}}{\text{time steps}} - \lambda \cdot \text{energy used}
$$
> **Readable form:** Score equals dirt cleaned per step minus a penalty for energy wasted wandering.

### Pedagogical Sections

1. **Fully observable ≠ intelligent** - you can see everything and still act foolishly without state.
2. [Rationality](./section-02-good-behavior-and-rationality.md) depends on architecture matching sequential structure.
3. Vacuum world previews **search** (Chapter 03) on state graphs whose nodes are world configurations.

### Real-World Vacuum Robots

Roomba-class devices add:

- **Partial observability** (furniture, unknown maps) → SLAM, belief-state pattern
- **Stochastic** slip and battery → utility tradeoffs
- **Learning** from home layout → [learning agent](./section-05-learning-agents.md)

Your Course 1 instincts might say "classify each cell Dirty/Clean" - but the **sequential** cleaning tour is a planning problem, not episodic classification.

---

## Case Study 2: Autonomous Taxi Driver

Russell and Norvig's taxi driver is the **running example** for a rich continuous environment - the stress test for Chapter 02 concepts.

### PEAS Specification

| PEAS | Autonomous taxi |
|------|-----------------|
| **P** | Safe, legal, comfortable trip; minimize time and fuel; maximize passenger satisfaction and revenue |
| **E** | Roads, traffic, pedestrians, weather, passengers, other vehicles, regulations |
| **A** | Steering, acceleration, braking, signaling, horn, display, voice communication, door locks |
| **S** | Cameras, lidar, radar, GPS, IMU, speed, engine diagnostics, passenger inputs, network traffic data |

### Environment Properties

| Property | Classification |
|----------|----------------|
| Observable | **Partially** - occlusions, unpredictable pedestrians |
| Agents | **Multiagent** - drivers, pedestrians, cyclists negotiate space |
| Deterministic | **Stochastic** - others' behavior, weather, sensor noise |
| Episodic | **Sequential** - merge now affects lane options later |
| Static | **Dynamic** - world changes during planning |
| Discrete | **Continuous** - position, velocity, steering |

This environment hits the **hard pole on every axis** ([Section 2.3](./section-03-the-nature-of-environments.md)). No single architecture suffices.

### Recommended Architecture ([Section 2.6](./section-06-agent-design-patterns.md))

**Layered hybrid:**

```python
def taxi_agent(percepts, state):
    # Layer 0: hard safety reflex (milliseconds)
    if emergency_brake_trigger(percepts):
        return EMERGENCY_BRAKE

    # Layer 1: update belief over world (model-based)
    state = localize_and_track(percepts, state)

    # Layer 2: tactical behavior (rules + short horizon)
    if state.intersection_ahead:
        tactical = negotiate_intersection(state)

    # Layer 3: strategic route (goal/utility planning)
    route = planner(state.position, state.destination, state.traffic_belief)

    # Layer 4: utility arbitration
    action = maximize_expected_utility(
        candidates=[tactical, route, lane_keep(state)],
        utility=PASSENGER_SAFETY_COMFORT_TIME,
        belief=state.belief,
    )
    return action
```

> **Readable form:** Brake instantly if collision imminent; update where you think you are; handle the intersection ahead; replan the route; pick the best combined action for safety, comfort, and speed.

### Utility Tradeoffs

Taxi [performance measure](../../GLOSSARY.md#performance-measure) encodes painful tradeoffs:

| Action | Utility gain | Utility loss |
|--------|--------------|--------------|
| Aggressive merge | Saves 30 seconds | Risk + passenger discomfort |
| Yield to pedestrian | Safety, legal | Delay |
| Detour around construction | Smooth ride | Extra miles |

Formalized in Chapter 16 as **multi-attribute utility**. [Section 1.5](../chapter-01-introduction/section-05-risks-and-benefits-of-ai.md) notes safety-critical stakes when utilities mis-weight lives vs. seconds.

### Techniques Roadmap

| Subproblem | Chapter | Technique |
|------------|--------|-----------|
| Localization | 14 | Kalman / particle filters |
| Prediction of others | 18, 21 | Multiagent + learned behavior models |
| Route planning | 03, 04 | A*, dynamic replanning |
| Low-level control | 26 | PID, model predictive control |
| Fleet learning | 22 | RL from disengagement logs |

### Connection to Course 1

A taxi fare [regression model](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-02-regression-models/section-02-linear-regression.md) predicts price from distance - one **episodic** slice. The driving agent chooses **actions** that *create* the trip distribution. Prediction supports planning; it does not replace the agent loop ([Section 2.1](./section-01-agents-and-environments.md)).

---

## Case Study 3: Medical Diagnosis Agent

Medical diagnosis illustrates **knowledge-intensive**, **partially observable**, **high-stakes** reasoning - closer to logical and probabilistic AI than to reflex control.

### PEAS Specification

| PEAS | Medical diagnosis assistant |
|------|----------------------------|
| **P** | Correct diagnosis, effective treatment, minimize harm, cost, and time to diagnosis |
| **E** | Patient physiology, disease processes, hospital resources, guidelines, legal constraints |
| **A** | Ask questions, order tests, recommend treatments/referrals, document reasoning |
| **S** | Patient reports, vitals, lab results, imaging, EHR history, literature access |

### Environment Properties

| Property | Classification |
|----------|----------------|
| Observable | **Partially** - internal state hidden |
| Agents | **Multiagent** - patient, specialists, insurers interact |
| Deterministic | **Stochastic** - disease progression, test noise |
| Episodic | **Sequential** - test → result → revised hypothesis |
| Static | **Semidynamic** - patient worsens while waiting |
| Discrete | **Mixed** - discrete diagnoses, continuous vitals |

### Architecture: Belief + Utility

Unlike vacuum world, **simple reflex is dangerous**. Standard pattern:

1. **Maintain belief** over diseases $P(\text{disease} \mid \text{evidence})$
2. **Choose next action** (test or treat) maximizing expected utility of patient outcome minus cost

$$
a^* = \arg\max_a \mathbb{E}\left[ U(\text{patient outcome}) - C(\text{cost}, \text{risk}) \mid a, \text{evidence} \right]
$$
> **Readable form:** Pick the test or treatment with the best expected patient outcome minus cost and risk, given everything you know so far.

```python
def diagnosis_step(belief, percepts, possible_actions):
    belief = bayesian_update(belief, percepts)  # Chapter 12
    best_a, best_value = None, float("-inf")
    for a in possible_actions:  # order MRI, prescribe, wait, ...
        value = expected_utility(belief, a)
        if value > best_value:
            best_value, best_a = value, a
    return best_a, belief
```

### Relation to Course 1 Classification

A [classifier](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/section-01-classification-fundamentals.md) mapping symptoms → disease label is **one step** of the agent. Full diagnosis is **sequential active learning**:

| ML classifier | Diagnosis agent |
|---------------|-----------------|
| Fixed feature vector | Agent **requests** new percepts (tests) |
| One-shot prediction | Iterative belief revision |
| Accuracy on test set | Utility of patient outcome |

[Naive Bayes](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/section-01-classification-fundamentals.md) appears in Chapter 12 as a diagnostic engine - now you see it inside the **performance element** of a larger agent.

### Knowledge Sources

- **Prior knowledge:** epidemiology, anatomy (designer or medical ontologies - Chapter 10)
- **Learned models:** imaging classifiers (Chapter 21, 25)
- **Guidelines:** encoded rules (Chapter 07-09 logic)

### Ethical Constraints

[Section 1.5](../chapter-01-introduction/section-05-risks-and-benefits-of-ai.md): bias in training data, accountability, **human-in-the-loop** for irreversible treatments. [Performance measure](../../GLOSSARY.md#performance-measure) must weight **false negative** (missed cancer) vs. **false positive** (unnecessary biopsy) asymmetrically - not raw [accuracy](../../GLOSSARY.md#accuracy).

---

## Comparative Summary

| Dimension | Vacuum | Taxi | Medical diagnosis |
|-----------|--------|------|-------------------|
| **Complexity** | Toy | Real-world hard | Knowledge-intensive |
| **Observability** | Full | Partial | Partial |
| **Risk** | Low | High | Very high |
| **Architecture** | Reflex → search | Layered hybrid | Belief + utility |
| **Primary chapters** | 03 | 03-04, 14, 16-18, 22, 26 | 12-13, 16, 07, 21 |
| **Course 1 analog** | Sequential control | Regression on fares | Classification |

```
        Risk / stakes
            ▲
            │                    ● Medical
            │              
            │         ● Taxi
            │
            │  ● Vacuum
            └────────────────────────► Environment complexity
```

---

## Chapter 02 Lab Preview

The chapter lab asks you to produce **PEAS + architecture diagram** for a warehouse picking robot. Using this section's template:

1. **PEAS** - performance (pick rate, errors), environment (shelves, humans, AMRs), actuators (gripper, base), sensors (cameras, encoders)
2. **Properties** - partially observable, multiagent cooperative, sequential, semidynamic
3. **Architecture** - layered: safety reflex + localization belief + path planner + optional learning from pick failures
4. **Justify** each choice by mapping to [design patterns](./section-06-agent-design-patterns.md)

---

## From Case Studies to Search

Vacuum world's goal-based variant **is** a search problem: states = dirt configurations, actions = move/suck, goal = all clean. Chapter 03 begins there.

The taxi and medical cases need **search plus** probability, utility, and learning - the rest of Course 2. Chapter 02 gave you the **map**; upcoming chapters supply the **engines**.

---

## Reflection Questions

1. Modify vacuum world so dirt reappears randomly. Which environment properties change? Which architecture do you need?
2. For the taxi, identify one reflex, one model-based, and one utility-based decision in a single intersection approach.
3. Why is asymmetric misclassification cost more important than accuracy for medical agents?
4. Pick a system from [Section 1.4](../chapter-01-introduction/section-04-state-of-the-art.md) and write its PEAS + property classification.

---

**Next:** [Chapter 03 - Solving Problems by Searching](../chapter-03-solving-problems-by-searching/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 2 examples. [AIMA book site](https://aima.cs.berkeley.edu/)
2. AIMA Python - `vacuum` and `agents` environments. [GitHub](https://github.com/aimacode/aima-python)
3. Stanford CS221 - PEAS and taxi driver example. [Course site](https://stanford-cs221.github.io/)
4. Shortliffe, E. H. (1976). *Computer-Based Medical Consultations: MYCIN*. - early diagnosis agent. [ACM Digital Library](https://dl.acm.org/doi/10.5555/1095294)
5. Waymo - autonomous driving system overview. [waymo.com/tech](https://waymo.com/tech/)
6. Russell, S., & Norvig, P. - online code figures for vacuum world. [AIMA figures](https://aima.cs.berkeley.edu/figures.pdf)
