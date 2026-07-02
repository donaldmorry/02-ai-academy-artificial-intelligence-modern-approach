# Section 1.6: Course Roadmap

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 1 + course structure  
> **Previous:** [Section 1.5 - Risks and Benefits](./section-05-risks-and-benefits-of-ai.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)    
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From "What Is AI?" to Building Intelligent Agents

You have completed the intellectual prologue to AIMA:

- [Section 1.1](./section-01-what-is-ai.md) - four definitions, [Turing test](../../GLOSSARY.md#turing-test), [rational agents](../../GLOSSARY.md#rational-agent)
- [Section 1.2](./section-02-foundations-of-ai.md) - philosophy, math, economics, neuroscience, psychology, engineering
- [Section 1.3](./section-03-history-of-ai.md) - Dartmouth to deep learning
- [Section 1.4](./section-04-state-of-the-art.md) - games, vision, language, robotics, science
- [Section 1.5](./section-05-risks-and-benefits-of-ai.md) - benefits, bias, safety, governance

This final section answers: **where does the rest of Course 2 go, and how do you navigate 28 chapters without drowning?**

> **Readable form:** Chapter 01 is the airport map. Chapter 02 onward is the actual flight - search, logic, probability, learning, language, vision, robots, ethics. This section is your boarding pass.

---

## The Unifying Framework: Rational Agents

Every technique in AIMA serves one master idea from [Section 1.1](./section-01-what-is-ai.md):

**An intelligent agent perceives, reasons, and acts to maximize expected performance.**

Chapter 02 formalizes this with the **PEAS** description (Performance, Environment, Actuators, Sensors) and environment properties (observable vs. partially observable, deterministic vs. stochastic, episodic vs. sequential).

```python
# Agent loop - the heartbeat of AIMA
def agent_loop(percepts):
    while True:
        action = select_action(percepts, knowledge, goals)
        execute(action)
        percepts = get_new_percepts()
```

Different chapters equip agents with different **competences**:

| Competence | Question answered | Chapters
|------------|-------------------|---------|
| Search | "What sequence of actions reaches the goal?" | 03-06 |
| Logic | "What can I prove true given my knowledge?" | 07-11 |
| Probability | "What should I believe under uncertainty?" | 12-18 |
| Learning | "How do I improve from experience?" | 19-22 |
| Language | "How do I understand and generate text?" | 23-24 |
| Vision | "How do I interpret images?" | 25 |
| Robotics | "How do I act in the physical world?" | 26 |
| Ethics | "What should I optimize - and for whom?" | 27-28 |

Your [Course 1](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-01-what-is-machine-learning.md) experience maps directly to the **Learning** row - but now you will see how learning interacts with search (AlphaGo), logic (knowledge graphs + LLMs), and probability (Bayesian deep learning).

---

## Course 2 Structure: Seven Parts

### Part I: Artificial Intelligence (Chapters 01-02) ← You are here

**Goal:** Define AI, survey history and state of the art, introduce the agent framework.

**Next step:** [Chapter 02 - Intelligent Agents](../chapter-02-intelligent-agents/README.md)

### Part II: Problem-Solving (Chapters 03-06)

**Goal:** Find action sequences via **search** and **constraint satisfaction**.

- Chapter 03 - BFS, DFS, A*, heuristics (uninformed → informed search)
- Chapter 04 - Local search, nondeterministic environments
- Chapter 05 - Adversarial search, minimax, alpha-beta, MCTS
- Chapter 06 - CSPs, backtracking, arc consistency

Real-life analogy: Part II is **GPS for abstract spaces** - navigating mazes, puzzles, and games rather than highways.

> **Readable form:** Before an agent can learn or reason probabilistically, it often needs to **find a path**. Search is the boring superpower behind chess engines, route planners, and puzzle solvers.

### Part III: Knowledge, Reasoning & Planning (Chapters 07-11)

**Goal:** Represent knowledge formally and **plan** multi-step actions.

- Propositional and first-order logic
- Inference, resolution, knowledge engineering
- Automated planning (STRIPS, hierarchical planning)

Connects to [foundations](./section-02-foundations-of-ai.md): the "laws of thought" approach made computational.

### Part IV: Uncertain Knowledge & Reasoning (Chapters 12-18)

**Goal:** Reason and decide under **uncertainty**.

Key equation (appears throughout):

$$
P(h \mid e) = \frac{P(e \mid h)\, P(h)}{P(e)}
$$
> **Readable form:** Posterior belief in hypothesis h equals likelihood of evidence given h, times prior on h, divided by total probability of evidence.

Topics: Bayesian networks, HMMs, MDPs, POMDPs, game theory. If Course 1's [supervised learning](../../GLOSSARY.md#supervised-learning) felt intuitive but ad hoc, Part IV gives **mathematical foundations** for why probabilistic models work.

### Part V: Machine Learning (Chapters 19-22)

**Goal:** Formal treatment of learning - your Course 1 skills, upgraded.

| Chapter | Topics | Course 1 bridge
|--------|--------|-----------------|
| 19 | Decision trees, linear models, ensembles | [Chapter 02-03](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-02-regression-models/README.md) |
| 20 | EM, probabilistic models | Beyond Course 1 |
| 21 | Neural networks, CNNs, RNNs | Preview of Course 3 |
| 22 | Reinforcement learning | AlphaGo, robotics |

[Machine learning](../../GLOSSARY.md#machine-learning) is one competence among many - but an increasingly dominant one in the [state of the art](./section-04-state-of-the-art.md).

### Part VI: Communicating, Perceiving & Acting (Chapters 23-26)

**Goal:** Language, vision, robotics - AI in the wild.

These chapters connect to applications from [Section 1.4](./section-04-state-of-the-art.md): translation, image classification, warehouse robots.

### Part VII: Conclusions (Chapters 27-28) + Appendices (A-B)

**Goal:** Limits, ethics, safety, future directions; mathematical background.

Chapter 27 expands [risks and benefits](./section-05-risks-and-benefits-of-ai.md) into full ethical analysis. Chapter 28 asks what **integrated AI architectures** might look like decades ahead.

Appendix A covers complexity, linear algebra, probability - reference when math gets dense. Appendix B covers BNF and pseudocode conventions.

---

## Dependency Map: How Chapters Connect

```
Chapter 01 (Introduction)
    ↓
Chapter 02 (Agents) ─────────────────────────────┐
    ↓                                              │
Chapters 03-06 (Search)                             │
    ↓                                              │
Chapters 07-11 (Logic & Planning)                   │
    ↓                                              │
Chapters 12-18 (Probability & Decisions) ←──────────┤
    ↓                                              │
Chapters 19-22 (Learning) ← Course 1 ML ────────────┤
    ↓                                              │
Chapters 23-26 (NLP, Vision, Robotics)              │
    ↓                                              │
Chapters 27-28 (Ethics, Future) ←───────────────────┘
```

**Suggested paths:**

- **ML-first learners** (strong Course 1 background): 01 → 02 → 19-22 → 12-15 → 03-06 → remaining chapters
- **Classical AI path** (textbook order): 01 → 02 → 03 → ... → 28
- **Practitioner fast track**: 01 → 02 → 03 → 12 → 19 → 21 → 24 → 27

There is no single correct order - but **Chapter 02 is non-negotiable** before diving deep elsewhere. Agents are the spine.

---

## Connections to Other Courses in the Ecosystem

| Course 2 chapter | Deepens in
|-----------------|------------|
| 21 Deep Learning | **Course 3** - mathematical treatment of neural nets |
| 24 Deep Learning NLP | **Course 3** + **Course 4** - transformers, generative models |
| 19 Learning from Examples | **Course 1** - same algorithms, formal proofs |
| 27 Ethics & Safety | **Course 4** - societal impact of generative AI |

Course 2 is the **hub**. Course 1 gave you hands-on ML. Course 3 (Goodfellow) goes deeper on neural networks. Course 4 (Foster) covers VAEs, GANs, diffusion - generative [state of the art](./section-04-state-of-the-art.md).

---

## Chapter 01 Lab: AI Landscape Report

The [chapter README](./README.md) assigns a short project:

1. Pick **three state-of-the-art systems** (e.g., AlphaFold, GPT-4, Waymo)
2. Classify each by **AI subfield** (search, learning, NLP, vision, robotics)
3. Identify **inputs and outputs**
4. Discuss **one risk per system**

This exercises exactly the synthesis Chapter 01 demands - connecting [history](./section-03-history-of-ai.md), [capabilities](./section-04-state-of-the-art.md), and [risks](./section-05-risks-and-benefits-of-ai.md).

---

## Capstone Preview: Design an Intelligent Agent

Course 2 ends with a **capstone** (see [course README](../../README.md)):

1. Choose a domain (healthcare triage, warehouse robot, game, smart home)
2. Write a **PEAS** description
3. Combine techniques - search + learning + reasoning
4. Handle **uncertainty** and multi-step planning
5. Evaluate performance and **ethical considerations**

The capstone is where [artificial intelligence](../../GLOSSARY.md#artificial-intelligence) stops being chapter exercises and becomes **system design**.

Example sketch - warehouse robot:

| PEAS | Specification
|------|---------------|
| **Performance** | Packages sorted per hour, damage rate |
| **Environment** | Warehouse aisles, humans, conveyor belts |
| **Actuators** | Wheels, gripper, lift |
| **Sensors** | Lidar, cameras, bump sensors |

Techniques: A* navigation (Chapter 03), object detection (Chapter 25), MDP for uncertain human movement (Chapter 17), RL for grasping (Chapter 22).

---

## Study Strategies for a 1,100-Page Textbook

1. **Read sections before algorithms** - intuition first, pseudocode second
2. **Implement one algorithm per chapter** - BFS in Chapter 03, minimax in Chapter 05, etc.
3. **Connect to Course 1 code** - reimplement k-NN after Chapter 19's theory
4. **Use Appendix A as reference** - do not memorize every proof upfront
5. **Revisit [risks](./section-05-risks-and-benefits-of-ai.md)** when tempted to overclaim project results

Estimated pace: **16-24 weeks** at 8-12 hours per week. Chapter 01 alone is 8-10 hours including this section set.

---

## Self-Assessment Checklist

Before leaving Chapter 01, confirm you can:

- [ ] Explain the four AI definition quadrants ([Section 1.1](./section-01-what-is-ai.md))
- [ ] Name three foundations of AI and their contributions ([Section 1.2](./section-02-foundations-of-ai.md))
- [ ] Identify two AI winters and their causes ([Section 1.3](./section-03-history-of-ai.md))
- [ ] Give an example of superhuman narrow AI vs. general fragility ([Section 1.4](./section-04-state-of-the-art.md))
- [ ] List three categories of AI risk ([Section 1.5](./section-05-risks-and-benefits-of-ai.md))
- [ ] Sketch how Chapters 03-28 relate to the agent framework (this section)

---

## What Comes Next

**Chapter 02 - Intelligent Agents** defines rationality precisely, classifies environments, and introduces agent architectures (simple reflex → goal-based → utility-based → learning). Everything afterward assumes you speak that language.

You entered Course 2 as an ML practitioner. You leave Chapter 01 as someone who knows **where ML sits inside the larger quest for intelligent systems** - and why the quest is far from finished.

---

**Previous:** [Section 1.5 - Risks and Benefits](./section-05-risks-and-benefits-of-ai.md) · **Next Chapter:** [Chapter 02 - Intelligent Agents](../chapter-02-intelligent-agents/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Chapters 1-2. [https://aima.cs.berkeley.edu/](https://aima.cs.berkeley.edu/)
2. Berkeley AI Materials - AIMA code and figures. [https://github.com/aimacode/aima-python](https://github.com/aimacode/aima-python)
3. Stanford CS221 - Course schedule mirroring agent-first pedagogy. [https://stanford-cs221.github.io/](https://stanford-cs221.github.io/)
4. MIT Open Learning Library - *Introduction to Machine Learning* (bridging ML and AI). [https://openlearninglibrary.mit.edu/](https://openlearninglibrary.mit.edu/)
5. Russell, S. (2024). "Provably Beneficial Artificial Intelligence." Communications of the ACM. [https://cacm.acm.org/](https://cacm.acm.org/)



