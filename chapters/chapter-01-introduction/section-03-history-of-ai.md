# Section 1.3: History of AI

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 1.3  
> **Previous:** [Section 1.2 - Foundations of AI](./section-02-foundations-of-ai.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)    
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## A Story of Ambition, Hype, and Comebacks

The history of [artificial intelligence](../../GLOSSARY.md#artificial-intelligence) is not a smooth upward curve. It is a **roller coaster**: bursts of optimism, funding booms, embarrassing failures, "AI winters" when grants dried up, and sudden revivals when someone cracked a problem everyone thought was decades away.

Understanding this history matters for two reasons. First, it explains **why the field looks the way it does today** - why deep learning dominates headlines while logic-based expert systems feel like ancient history. Second, it calibrates expectations. Every generation believed *their* breakthrough was "real AI" around the corner. Humility is a professional skill.

> **Readable form:** AI history is like fashion trends. Bell-bottoms (symbolic logic) were huge, then embarrassing, then vintage cool in niche circles. Skinny jeans (deep learning) rule today - but something else will follow.


---

## 1943-1956: The Birth of AI

### McCulloch-Pitts Neurons (1943)

Warren McCulloch and Walter Pitts proposed a **mathematical model of neurons** - binary on/off units whose outputs depend on weighted inputs. This linked biology to computation and foreshadowed neural networks decades later.

### Turing's Vision (1950)

Alan Turing's paper "Computing Machinery and Intelligence" asked whether machines could think and proposed the [Turing test](../../GLOSSARY.md#turing-test) ([Section 1.1](./section-01-what-is-ai.md)). He also argued that **child-machine** learning - educate a blank-slate system - might be easier than hand-programming adult intelligence.

### The Dartmouth Workshop (1956)

John McCarthy, Marvin Minsky, Nathaniel Rochester, and Claude Shannon organized a summer workshop at Dartmouth College. McCarthy **coined the term "artificial intelligence."** The proposal was breathtakingly optimistic:

> "An attempt will be made to find how to make machines use language, form abstractions and concepts, solve kinds of problems now reserved for humans, and improve themselves."

Attendees included future Nobel-level figures. They believed **a few summers** of work might suffice. Seventy years later, we are still working - but on a vastly larger scale.

**Key programs from this era:**
- **Logic Theorist** (Newell & Simon, 1956) - proved theorems from *Principia Mathematica*
- **General Problem Solver** (1957) - means-ends analysis for puzzles
- **Checkers program** (Samuel, 1959) - learned by playing itself; early [machine learning](../../GLOSSARY.md#machine-learning)

Samuel's checkers work connects directly to [Course 1](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-01-what-is-machine-learning.md): learning from experience to improve performance - decades before sklearn existed.

---

## 1957-1973: Early Enthusiasm and Discovery

Funding flowed from DARPA and other agencies. Researchers believed symbolic reasoning would scale to full intelligence.

### Milestones

| Year | Achievement | Significance
|------|-------------|--------------|
| 1958 | McCarthy invents **LISP** | AI's first language; symbolic lists everywhere |
| 1959 | **Perceptron** (Rosenblatt) | Trainable linear classifier - ancestor of neural nets |
| 1965 | **ELIZA** (Weizenbaum) | Rule-based "therapist" chatbot - fooled some users |
| 1966 | **Shakey the Robot** | Mobile robot combining perception, planning, action |
| 1969 | **A* search** (Hart, Nilsson, Raphael) | Optimal pathfinding - still used in GPS today |

ELIZA is the hipster ancestor of ChatGPT: no learning, just pattern matching - yet users attributed deep empathy to it. Weizenbaum was horrified.

> **Readable form:** ELIZA was a parlor trick that swapped "Tell me more about your mother" into any sentence. People cried to it anyway. Section: humans *want* to believe machines understand them.

---

## 1966-1974: First Reality Check

Optimism met limits.

### The Lighthill Report (1973, UK)

James Lighthill criticized AI for failing on **combinatorial explosion** - search spaces grow faster than any computer can explore. British funding collapsed.

### Minsky & Papert's *Perceptrons* (1969)

Showed single-layer perceptrons **cannot learn XOR** - a trivial non-linear function. Funding for neural networks evaporated for fifteen years.

```python
# XOR cannot be separated by a single linear boundary
# Inputs (0,0)->0, (0,1)->1, (1,0)->1, (1,1)->0
# No single line divides the 1s from the 0s in 2D input space
```

This was not a death blow to AI overall - symbolic systems continued - but it killed the first neural network wave.

---

## 1980-1987: Expert Systems and the AI Boom

If general intelligence was hard, maybe **narrow domain expertise** would pay. **Expert systems** encoded human knowledge as if-then rules:

```
IF patient has fever AND cough AND loss of smell
THEN hypothesis = respiratory_infection (confidence 0.7)
```

### Commercial Success

- **MYCIN** (1970s) - diagnosed blood infections; outperformed some doctors on test cases
- **XCON/R1** (DEC, 1980s) - configured VAX computers; saved millions annually
- Japan's **Fifth Generation Project** (1982) - massive government investment in logic-based AI

Corporations bought AI startups. "AI" appeared in annual reports. Rule-based systems worked when domains were **closed**, experts were **available**, and maintenance costs were **ignored**.

Real-life analogy: expert systems were like **printed cookbooks** - excellent for known recipes, useless when ingredients change unless someone rewrites the book nightly.

---

## 1987-1993: Second AI Winter

Expert systems **failed to scale**. Knowledge acquisition became a bottleneck - extracting rules from experts was slow, expensive, and incomplete. Maintenance costs exploded as domains shifted.

Simultaneously, **cheap desktop PCs** outperformed specialized Lisp machines. The hardware market for AI collapsed. DARPA shifted focus. Again, "AI" became a dirty word in grant proposals - researchers rebranded work as "machine learning," "informatics," or "cognitive systems."

> **Readable form:** Expert systems were like hiring a genius consultant who charges by the hour to update a spreadsheet every time the tax law changes. Eventually CFOs said "no more."

---

## 1993-2011: The Statistical Revolution

Three forces converged - the same trio that powered your [Course 1](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-01-what-is-machine-learning.md) curriculum:

1. **Data** - the web generated text, clicks, and logs at scale
2. **Compute** - PCs and clusters got fast enough
3. **Algorithms** - probabilistic methods matured

### Key Developments

| Era | Technique | Impact
|-----|-----------|--------|
| 1990s | Hidden Markov Models | Speech recognition in products |
| 1995 | Support Vector Machines | Strong classification performance |
| 1997 | **Deep Blue** beats Kasparov | Search + hardware; not ML, but AI headline |
| 2000s | **Statistical NLP** | Google Translate replaces rule pipelines |
| 2006 | Deep belief networks (Hinton) | Reignited neural network research |
| 2011 | **IBM Watson** wins Jeopardy! | NLP + search + ML ensemble |

The paradigm shift: instead of hand-writing rules, **learn patterns from data**. This is [supervised learning](../../GLOSSARY.md#supervised-learning) at industrial scale - the philosophy of Arthur Samuel's checkers program, finally winning the culture war.

---

## 2012-Present: Deep Learning Era

### ImageNet Moment (2012)

Alex Krizhevsky's **AlexNet** crushed the ImageNet competition using deep convolutional neural networks on GPUs. Error rates dropped from ~26% to ~16% overnight. Computer vision was never the same.

### Reinforcement Learning Breakthroughs

- **DQN** (2015) - deep Q-learning plays Atari from pixels
- **AlphaGo** (2016) - defeats Lee Sedol; combines deep nets with Monte Carlo tree search
- **AlphaZero** (2017) - self-play masters chess, shogi, Go without human games

### Language Models and Generative AI

- **Transformers** (2017) - "Attention Is All You Need"
- **BERT, GPT series** - pretraining on web text
- **2022-present** - ChatGPT, multimodal models, coding assistants, agents

The [state of the art](./section-04-state-of-the-art.md) today would sound like science fiction to the Dartmouth attendees - yet core ideas (search, learning, reasoning) appear in every breakthrough.

---

## Timeline at a Glance

```
1943  McCulloch-Pitts neuron
1950  Turing test
1956  Dartmouth - "AI" coined
1969  Perceptrons book → NN winter
1973  Lighthill report → funding cuts
1980s Expert systems boom
1987  Lisp machine crash → second winter
1997  Deep Blue
2006  Deep learning revival
2012  AlexNet
2016  AlphaGo
2022  ChatGPT mainstream
```

---

## Sections from History

1. **Breakthroughs are unpredictable.** Neural nets were "dead" twice before dominating.
2. **Narrow success ≠ general intelligence.** Deep Blue did not "understand" chess; Watson did not "know" trivia the way humans do.
3. **Hardware and data matter as much as algorithms.** AlexNet needed GPUs; modern LLMs need datacenters.
4. **Hype cycles hurt.** Overpromising triggers winters that delay legitimate research.
5. **The [rational agent](../../GLOSSARY.md#rational-agent) framework outlasts trends.** Search, logic, probability, and learning keep returning under new names.

---

## Reflection Questions

1. What triggered each AI winter? Do you see similar warning signs today?
2. Why did expert systems succeed commercially before neural networks did?
3. Which historical milestone most changed *your daily life*? (GPS A*, Google Translate, Siri, ChatGPT?)

---

**Previous:** [Section 1.2 - Foundations of AI](./section-02-foundations-of-ai.md) · **Next:** [Section 1.4 - State of the Art](./section-04-state-of-the-art.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), §1.3. [https://aima.cs.berkeley.edu/](https://aima.cs.berkeley.edu/)
2. Wikipedia - "History of artificial intelligence." [https://en.wikipedia.org/wiki/History_of_artificial_intelligence](https://en.wikipedia.org/wiki/History_of_artificial_intelligence)
3. McCarthy, J., et al. (1955). "A Proposal for the Dartmouth Summer Research Project on Artificial Intelligence." [PDF at Stanford](http://jmc.stanford.edu/articles/dartmouth/dartmouth.pdf)
4. Krizhevsky, A., Sutskever, I., & Hinton, G. E. (2012). "ImageNet Classification with Deep Convolutional Neural Networks." [NeurIPS paper](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks)
5. Silver, D., et al. (2016). "Mastering the game of Go with deep neural networks and tree search." *Nature*. [https://www.nature.com/articles/nature16961](https://www.nature.com/articles/nature16961)

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
