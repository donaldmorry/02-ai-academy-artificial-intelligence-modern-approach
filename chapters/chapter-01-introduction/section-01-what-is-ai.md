# Section 1.1: What Is AI?

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 1  
> **Prerequisites:** [Course 1 - What Is Machine Learning?](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-01-what-is-machine-learning.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)    
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Question Everyone Asks

"What is artificial intelligence?" sounds simple until you try to answer it at a dinner party. One person points to ChatGPT. Another mentions self-driving cars. A third insists AI is just "fancy statistics." All three are partially right - and all three miss the deeper question AI researchers have wrestled with for seventy years: **what would it mean for a machine to be intelligent at all?**

Russell and Norvig open their textbook with a deceptively modest claim: [Artificial Intelligence](../../GLOSSARY.md#artificial-intelligence) is **the study and construction of agents that receive percepts from the environment and perform actions**. That definition is deliberately broad. It covers chess programs, spam filters, warehouse robots, and hypothetical future systems we have not yet built. It also sidesteps the philosophical quicksand of "Can machines think?" - a question we will return to, but not let block practical progress.

If you completed [Course 1](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-01-what-is-machine-learning.md), you already know one powerful slice of AI: [machine learning](../../GLOSSARY.md#machine-learning), where systems improve from data. AIMA widens the lens. **Search, logic, probability, planning, and learning** are all tools in the same toolbox - unified by the idea of a [rational agent](../../GLOSSARY.md#rational-agent) acting in an environment.

---

## Four Ways to Define AI

Russell and Norvig organize definitions along two axes:

| | **Humanly** (match human behavior) | **Rationally** (maximize performance) |
|---|---|---|
| **Thinking** | Cognitive science: model human thought | Laws of thought: correct reasoning |
| **Acting** | [Turing test](../../GLOSSARY.md#turing-test): fool a human judge | Rational agent: do the right thing |

Think of it like evaluating a new employee. You could ask: *Does she think like our best analyst?* (thinking humanly). *Does her reasoning follow logical rules?* (thinking rationally). *Can she pass as a human on a phone call?* (acting humanly). Or simply: *Does she get the job done?* (acting rationally). AI researchers have proposed definitions in all four quadrants.

### Acting Humanly: The Turing Test

Alan Turing proposed the **Imitation Game** in 1950: a human interrogator chats via text with two hidden participants - one human, one machine. If the interrogator cannot reliably tell which is which, the machine passes the [Turing test](../../GLOSSARY.md#turing-test).

Real-life analogy: it is like a **blind taste test** for conversation. If you cannot tell Coke from Pepsi, branding failed. If you cannot tell human from machine, the machine "passes."

A full Turing-test-capable system would need:

- **Natural language processing** - understand and generate text
- **Knowledge representation** - store what it knows about the world
- **Automated reasoning** - answer questions and draw conclusions
- **Machine learning** - adapt to new situations
- **Computer vision and robotics** - if the test includes physical interaction

> **Readable form:** The Turing test says "if it talks like a human and you can't tell the difference, call it intelligent." It is a clever social test, not a scientific one - and modern chatbots can fool people without truly understanding anything.

**Limitations:** The Turing test rewards deception over honesty. A system that refuses to answer hard questions ("I'm tired, let's talk about movies") might pass more easily than one that tries to reason correctly. It also conflates **appearance** of intelligence with **substance** - a theme we revisit in [Section 1.4](./section-04-state-of-the-art.md).

### Thinking Humanly: Cognitive Modeling

This approach asks: can we build computer models that reproduce human problem-solving? Early AI collaborated with psychologists, using reaction times and error patterns as evidence that a program's internal steps match human cognition.

Example: when people solve algebra word problems, they often make specific arithmetic errors. A cognitive model that reproduces those *same* errors - not just the right answers - is considered more faithful to human thinking.

### Thinking Rationally: Laws of Thought

Aristotle sought **syllogisms** - patterns of valid inference. The dream: codify all correct reasoning as logical rules; feed facts; derive guaranteed-true conclusions.

```python
# Propositional logic sketch (simplified)
facts = {"raining": True, "raining -> carry_umbrella": True}
# Modus ponens: if raining and (raining -> carry_umbrella), then carry_umbrella
if facts["raining"] and facts["raining -> carry_umbrella"]:
    conclude("carry_umbrella")  # logically valid
```

> **Readable form:** "Thinking rationally" means following the rules of logic so your conclusions *must* be true - if your premises are true. Like a tax form: if you fill every box correctly, the math guarantees the right result.

Problems arise in the real world: we rarely have perfect information, and pure logic struggles with **uncertainty**, **preferences**, and **computational limits**. Course 2 addresses this by adding probability (Chapters 12-18) and bounded rationality.

### Acting Rationally: The Rational-Agent Approach

This is AIMA's **central framework**. A [rational agent](../../GLOSSARY.md#rational-agent) selects actions that maximize **expected performance**, given:

- Its **percepts** (what it senses)
- Its **knowledge** (what it believes about the world)
- Its **goals** (what it is trying to achieve)

Formally, rationality is tied to **expected utility**:

$$
\text{action}^* = \arg\max_a \sum_s P(s \mid e) \, U(s, a)
$$
> **Readable form:** Choose the action that maximizes expected utility over possible world states, weighted by how likely each state is given your evidence - like choosing an umbrella when the forecast says 70% rain.

where $e$ is evidence from percepts, $s$ are possible world states, and $U(s, a)$ is the utility of taking action $a$ in state $s$.

**Why rational agents win as a definition:** They subsume the other three. Passing the Turing test is one possible goal. Cognitive modeling is one way to design agents. Logical reasoning is one tool agents use. But the agent framework also covers **self-driving cars** (maximize safe arrival), **recommendation systems** (maximize engagement or revenue), and **game-playing engines** (maximize win probability) - without requiring human-like conversation.

---

## Weak AI vs. Strong AI

| Claim | Weak AI | Strong AI
|-------|---------|-----------|
| **Systems can behave intelligently** | Yes | Yes |
| **Systems truly "have minds"** | No - simulation only | Yes - genuine consciousness |
| **Example stance** | "This chess program calculates well" | "This program understands chess the way humans do" |

Most working AI engineers operate in **weak AI** territory: build systems that solve tasks, without claiming silicon has inner experience. **Strong AI** (or **artificial general intelligence**, AGI) remains a research aspiration and philosophical debate - see [Section 1.5](./section-05-risks-and-benefits-of-ai.md).

Humorous analogy: weak AI is a **flight simulator** that never leaves the ground but trains pilots perfectly well. Strong AI would be an actual airplane that flies itself *and* feels the wind. We have excellent simulators; the self-aware airplane is still science fiction.

---

## AI vs. Machine Learning: The Relationship

From [Course 1, Section 1.2](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-02-machine-learning-vs-ai-vs-deep-learning.md):

```
AI  ⊃  Machine Learning  ⊃  Deep Learning
```

**Not all AI learns from data.** Deep Blue beat Kasparov using search and hand-crafted evaluation - no [supervised learning](../../GLOSSARY.md#supervised-learning) pipeline. **Not all ML is "intelligent" in the agent sense.** A linear regression predicting house prices is ML but barely qualifies as an agent (no sequential decisions, no environment interaction).

AIMA's message: treat ML as one **competence** an agent may need, alongside search, planning, and reasoning. Your Course 1 skills become essential in Chapters 19-22 - but they sit inside a larger architecture.

---

## What Makes a Problem "AI"?

Russell and Norvig suggest AI problems share traits like:

1. **Large search spaces** - too many possibilities to enumerate (chess: ~$10^{40}$ legal positions)
2. **Uncertainty** - incomplete or noisy information
3. **Complexity** - real environments change dynamically
4. **Knowledge-intensive** - success requires vast background facts (medical diagnosis, legal analysis)

If you can solve it with a ten-line `if/else` script, it is probably not AI - it is just programming. If you need representation, inference, learning, or search, welcome to the field.

---

## Mitchell's Learning Definition - The Bridge to Course 1

Russell and Norvig connect AI to Tom Mitchell's classic learning definition (see [Course 1](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-01-what-is-machine-learning.md)):

> A computer program **learns** from experience $E$ with respect to task $T$ and performance measure $P$, if its performance at $T$, as measured by $P$, improves with experience $E$.

| Component | Chess agent example | Spam filter example |
|-----------|---------------------|---------------------|
| **Task ($T$)** | Win chess games | Classify email as spam/ham |
| **Experience ($E$)** | Self-play games, human games | Labeled inbox history |
| **Performance ($P$)** | Win rate vs. strong opponents | Precision/recall on held-out mail |

AIMA's twist: learning is one **competence** inside a broader agent that also searches, plans, and reasons under uncertainty. Your Course 1 models were learning agents with a narrow task specification - Chapter 02 generalizes that to arbitrary environments.

---

## Performance Measures and Objective Design

Rational agents require an explicit **performance measure**. Russell and Norvig warn this is subtle:

- **What you measure is what you get.** An agent rewarded for "clean floors" might hide mess under furniture. A recommendation system optimized for clicks might amplify outrage.
- **Partial observability** means the agent never sees the full state - only percepts. A thermostat's percept is temperature, not "tenant comfort."
- **Multi-agent environments** mean your performance depends on others' actions - traffic, markets, games.

Designing $U(s, a)$ is often harder than implementing the algorithm. [Section 1.5](./section-05-risks-and-benefits-of-ai.md) returns to misaligned objectives at societal scale; Chapter 02 formalizes environment types that constrain rational design.

---

## Reflection Questions

1. Which of the four definitions (thinking/acting × humanly/rationally) best matches how *you* would explain AI to a non-technical friend? What does that definition miss?
2. Could a system pass the Turing test while being completely irrational in its actions outside the chat window?
3. How does the rational-agent view change how you think about the ML models you built in Course 1?

---

**Next:** [Section 1.2 - Foundations of AI](./section-02-foundations-of-ai.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 1. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Stanford University - *Introduction to Artificial Intelligence* (CS221). [Course overview](https://stanford-cs221.github.io/)
3. Turing, A. M. (1950). "Computing Machinery and Intelligence." *Mind*, 59(236), 433-460. [Wikipedia summary](https://en.wikipedia.org/wiki/Computing_Machinery_and_Intelligence)
4. Stanford Encyclopedia of Philosophy - "The Turing Test." [plato.stanford.edu/entries/turing-test](https://plato.stanford.edu/entries/turing-test/)
5. Mitchell, T. M. (1997). *Machine Learning*. McGraw-Hill. [Author page](https://www.cs.cmu.edu/~tom/mlbook.html)




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
