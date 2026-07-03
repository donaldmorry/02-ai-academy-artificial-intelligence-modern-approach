# Section 2.1: Agents and Environments

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 2.1  
> **Prerequisites:** [Section 1.1 - What Is AI?](../chapter-01-introduction/section-01-what-is-ai.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Central Abstraction of AI

In [Section 1.1](../chapter-01-introduction/section-01-what-is-ai.md), Russell and Norvig defined [artificial intelligence](../../GLOSSARY.md#artificial-intelligence) as the study and construction of **agents** - entities that receive [percepts](../../GLOSSARY.md#percept) from the environment and perform actions. Chapter 2 makes that definition precise. Everything else in AIMA - search, logic, probability, learning, robotics - is machinery for building better agents.

Think of an agent like a **new hire on their first day**. They arrive with senses (eyes, ears), hands to act (keyboard, tools), and a job description that maps what they observe to what they should do. They do not need to be human, conscious, or chatty. They need a clear interface between **world** and **decision-maker**.

> **Readable form:** An agent is anything that takes in information from the world and does something back. A thermostat, a chess program, and a warehouse robot are all agents - different complexity, same pattern.

Humorous analogy: your phone's spam filter is an agent that **perceives** email text and **acts** by moving messages to the junk folder. It will never ask for a coffee break, but it fits the definition perfectly. Your cat is also an agent - though its [performance measure](../../GLOSSARY.md#performance-measure) appears optimized for knocking things off tables.

---

## Agents and Environments

An **agent** is anything that can be viewed as perceiving its environment through **sensors** and acting upon that environment through **actuators**.

| Component | Role | Chess program example | Self-driving car example |
|-----------|------|----------------------|--------------------------|
| **Environment** | Everything the agent interacts with | Board, clock, opponent | Roads, traffic, weather |
| **Sensors** | Produce percepts | Screen display of board state | Cameras, lidar, GPS, IMU |
| **Actuators** | Execute actions | Move selection sent to game engine | Steering, throttle, brakes |
| **Agent program** | Runs on the agent's architecture | Search + evaluation code | Perception + planning stack |

The **agent function** $f$ maps any given percept sequence to an action:

$$
f : \mathcal{P}^* \rightarrow \mathcal{A}
$$
> **Readable form:** The agent function takes the entire history of everything the agent has ever sensed (percept sequence) and returns the next action to take.

where $\mathcal{P}^*$ is the set of all possible percept sequences and $\mathcal{A}$ is the set of possible actions. In plain English: **given everything you have ever observed, what should you do next?**

### Percepts vs. States

A [percept](../../GLOSSARY.md#percept) is the agent's **subjective** sensory input at one instant - not necessarily the full truth about the world. A smoke detector's percept might be "alarm triggered," while the true state might be "toast burning" or "house fire." This distinction becomes critical in [Section 2.3](./section-03-the-nature-of-environments.md) when we classify **partially observable** environments.

From [Course 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-01-what-is-machine-learning.md), you trained models on **feature vectors** - fixed snapshots of data. Agent percepts are similar but **sequential**: the agent receives a stream over time, and past percepts may matter for the current decision.

---

## The Agent-Environment Boundary

Where do you draw the line between agent and environment? AIMA's answer: **wherever is most useful for design**.

For a chess program, the environment is the board and rules; the agent is the software selecting moves. For a robot vacuum, you might treat the whole robot as the agent (sensors and motors included) or split perception chapters from planning chapters - an engineering choice, not a philosophical one.

```
┌─────────────────────────────────────────┐
│              ENVIRONMENT                │
│   (everything outside the agent)        │
│                                         │
│    ┌─────────────────────────────┐    │
│    │           AGENT             │    │
│    │  sensors → program → actuators│  │
│    └─────────────────────────────┘    │
└─────────────────────────────────────────┘
         percepts ↑    ↓ actions
```

Real-life analogy: when you drive, **you** are the agent. The car's steering wheel and pedals are actuators; your eyes and ears are sensors. The road, weather, and other drivers are the environment. If you redesign a self-driving system, you might move the boundary - putting the camera processing inside the "agent" box instead of treating raw pixels as the environment.

---

## Agent Programs vs. Agent Functions

Russell and Norvig distinguish two levels:

1. **Agent function** - the abstract, ideal mapping from percept histories to actions (what perfect behavior *would* be).
2. **Agent program** - the concrete implementation running on a physical or virtual machine.

The agent function is like the **answer key** to an exam. The agent program is the **student's actual work** - hopefully close, often approximate, always bounded by memory and CPU.

```python
# Generic agent program (AIMA Figure 2.1, simplified)
def agent_program(percepts):
    """
    percepts: list of everything sensed so far
    returns: action to execute now
    """
    return agent_function(percepts)

# Main loop: perceive → decide → act → repeat
def run_agent(environment, agent_program, max_steps=1000):
    percepts = []
    for step in range(max_steps):
        percept = environment.get_percept()      # sensors
        percepts.append(percept)
        action = agent_program(percepts)         # decide
        environment.execute(action)              # actuators
        if environment.is_terminal():
            break
```

> **Readable form:** The agent loop is: sense the world, remember what you sensed, pick an action, do it, repeat until done or out of time.

This loop is the **universal skeleton** of intelligent systems. Your [k-nearest neighbors classifier](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-05-supervised-learning-k-nearest-neighbors.md) is not really an agent - it maps one input to one output with no sequential interaction. A fraud detection system that monitors a stream of transactions and can block cards **is** an agent.

---

## Examples Across Domains

### Table Lookup Agent

The simplest (and most useless at scale) agent stores every possible percept sequence and the correct action:

$$
f(\text{percept-sequence}) = \text{table}[\text{percept-sequence}]
$$
> **Readable form:** Look up the exact history of everything you've seen in a giant table and return the stored action.

For chess, the table would need entries for every possible game history - astronomically large. Table lookup is a **theoretical baseline**, not a practical design. It shows why we need **compact representations** (models, goals, utilities) covered in [Sections 2.4-2.5](./section-04-structure-of-agents.md).

### Thermostat Agent

A thermostat perceives **temperature** and acts by **turning heat on or off**:

```python
def thermostat_agent(percepts):
  current_temp = percepts[-1]["temperature"]
  if current_temp < TARGET - TOLERANCE:
      return "HEAT_ON"
  elif current_temp > TARGET + TOLERANCE:
      return "HEAT_OFF"
  return "NO_OP"
```

Two lines of logic, genuine agent. Not intelligent in the AI-hard sense - but a valid member of the taxonomy.

### Interactive Tutor Agent

An online tutoring system perceives **student answers**, **response times**, and **hint requests**; acts by **posing questions**, **giving hints**, or **advancing difficulty**. Its sensors are UI events; its actuators are rendered content. The environment includes the student - possibly a **multiagent** setting ([Section 2.3](./section-03-the-nature-of-environments.md)).

---

## Sensors and Actuators in the Real World

**Sensors** convert physical phenomena into percepts:

| Domain | Sensors | Typical percepts |
|--------|---------|------------------|
| Robotics | Cameras, lidar, touch | Depth maps, contact forces |
| Web services | HTTP requests, logs | User clicks, API payloads |
| Finance | Market feeds | Prices, volumes, news |
| Medicine | MRI, lab results | Images, biomarker values |

**Actuators** convert agent decisions into environmental effects:

| Domain | Actuators | Typical actions |
|--------|-----------|-----------------|
| Robotics | Motors, grippers | Move joint, grasp object |
| Software | API calls, DB writes | Block transaction, send alert |
| Games | Move submission | Play e4, resign |

Noisy or delayed sensors mean the agent's percept sequence **does not equal** true world history. [Section 2.2](./section-02-good-behavior-and-rationality.md) explains how [rational agents](../../GLOSSARY.md#rational-agent) must act on **beliefs**, not omniscient truth.

---

## From Course 1 Models to Agents

| Course 1 concept | Agent framing |
|------------------|---------------|
| Training data | Experience generating percepts (possibly labeled) |
| Features | Components of percepts or derived state |
| Prediction $\hat{y}$ | One type of action (classification/regression output) |
| Model | Part of agent program (often the performance element) |
| Batch inference | Agent loop with trivial environment (one-shot) |

A batch ML pipeline that scores loan applications overnight is a **degenerate agent**: one percept per customer, one action (approve/deny), no feedback loop. Add **online learning** from repayment outcomes and you approach the [learning agent architecture](./section-05-learning-agents.md) from Chapter 2.

---

## Design Questions to Ask Early

Before writing code, specify:

1. **What are the percepts?** (Format, frequency, noise)
2. **What are the actions?** (Discrete moves? Continuous control?)
3. **What is the percept sequence history?** (Markov: does only the latest percept matter?)
4. **Where is the boundary** between agent and environment?
5. **What does "success" mean?** ([Performance measure](./section-02-good-behavior-and-rationality.md) - next section)

Skipping these produces clever algorithms solving the wrong problem - like a chess engine optimized for speed that loses every game.

---

## Common Misconceptions

**"Agents must be physical robots."** False. Software agents in simulation, on servers, or in browsers count fully.

**"More sensors always help."** Not necessarily. Extra sensors add fusion complexity, calibration burden, and failure modes. A rational agent uses the **minimum adequate** perceptual apparatus.

**"The agent function must be computable."** In theory we discuss ideal $f$; in practice we accept approximations. [Section 1.2](../chapter-01-introduction/section-02-foundations-of-ai.md) noted that real agents face **bounded rationality** - limited time and memory.

---

## Reflection Questions

1. Is a calculator app an agent? Argue both sides using the sensor/actuator definition.
2. For a warehouse picking robot, list three sensors and three actuators. What percepts do they produce?
3. How does the agent program loop differ from the train/predict loop you used in [Course 1 Lab 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-lab-01-customer-segmentation-and-flower-classification.md)?
4. Why is a lookup-table agent function impossible to implement for chess, even in principle, with finite memory?

---

**Next:** [Section 2.2 - Good Behavior and Rationality](./section-02-good-behavior-and-rationality.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 2.1. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Stanford CS221 - *Introduction to Artificial Intelligence*. [Course overview](https://stanford-cs221.github.io/)
3. Russell, S., & Norvig, P. - AIMA pseudocode and figures. [AIMA code repository](https://github.com/aimacode/aima-python)
4. Mitchell, T. M. (1997). *Machine Learning* - task/experience/performance framing. [Author page](https://www.cs.cmu.edu/~tom/mlbook.html)
5. Wooldridge, M. - *An Introduction to MultiAgent Systems* (agent definitions in broader context). [Author page](https://www.cs.ox.ac.uk/people/michael.wooldridge/)
