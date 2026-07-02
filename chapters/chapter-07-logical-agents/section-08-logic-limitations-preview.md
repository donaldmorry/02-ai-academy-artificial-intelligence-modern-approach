# Section 7.8: Logic Limitations Preview

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 7 (outlook)  
> **Previous:** [Section 7.7 - Wumpus World](./section-07-wumpus-world.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Where Propositional Logic Stops

[Sections 7.1-7.7](./section-01-knowledge-based-agents.md) built agents with **propositional logic** - powerful for finite, structured worlds like Wumpus. Real knowledge needs **objects**, **relations**, **quantifiers**, **time**, and **uncertainty**. This section maps the gaps and previews fixes.

> **Readable form:** Boolean facts about fixed cells work in a 4×4 cave; they don't scale to "all patients with symptom X" or "maybe there's a pit."

Humorous analogy: propositional logic is **flashcards** - great for vocabulary, awkward for writing novels.

---

## Expressiveness Gaps

| Need | Propositional approach | Problem |
|------|------------------------|---------|
| "All neighbors have pits → breeze" | Copy rule per cell | $O(n^2)$ rules |
| "There exists a pit" | $P_{1,1} \lor P_{1,2} \lor \cdots$ | Long, inflexible |
| "Same wumpus everywhere" | Manual equality | No identity |
| "Eventually safe" | No temporal logic | Can't express |
| "Probably a pit" | No probability | Can't rank beliefs |

---

## First-Order Logic (FOL)

[Chapter 08](../chapter-08-first-order-logic/README.md) adds:

- **Predicates:** $Pit(x, y)$, $Adjacent(a, b)$
- **Quantifiers:** $\forall$, $\exists$
- **Functions:** $Location(agent)$

**One rule** for all cells:

$$
\forall x, y: Breeze(x,y) \Leftrightarrow \exists u,v: Adjacent((x,y), (u,v)) \land Pit(u,v)
$$
> **Readable form:** A square is breezy iff some adjacent square has a pit - one sentence for the whole grid.

**Unification** and **resolution** extend to FOL; **forward/backward chaining** with definite clauses ([Chapter 09](../chapter-09-inference-first-order-logic/README.md)).

---

## Temporal and Modal Extensions

**Situation calculus**, **event calculus** - actions change state over time (planning, [Chapter 11](../chapter-11-automated-planning/README.md)).

**Modal logic** - knowledge of other agents ($K_i \alpha$), belief, obligation - multiagent settings ([Chapter 18](../chapter-18-multiagent-decision-making/README.md)).

---

## Uncertainty

When KB **does not entail** $\alpha$ or $\neg\alpha$, propositional logic says **unknown** only - no **degrees**.

**Probability** ([Chapters 12-15](../chapter-12-quantifying-uncertainty/README.md)): $P(Pit_{2,2} \mid Breeze_{1,2}) = 0.7$.

**Bayes nets** structure joint distributions; **POMDPs** combine belief and action ([Chapter 17](../chapter-17-making-complex-decisions/README.md)).

Wumpus **hybrid agent** in AIMA: logic for certain safe moves, probability for exploration.

---

## Computational Limits

| Task | Complexity |
|------|------------|
| Propositional SAT | NP-complete |
| Propositional entailment | co-NP-complete |
| FOL entailment | **Undecidable** |
| Horn FOL | Decidable, PSPACE |

**Tractable fragments** matter in practice: Horn, description logic, Datalog.

---

## Knowledge Engineering Burden

Declarative KBs need **ontology** design:

- What symbols?
- What granularity?
- What invariants?

[Chapter 10](../chapter-10-knowledge-representation/README.md) ontologies, semantic web; **knowledge graphs** in industry.

---

## Comparison Table: Representations

| Representation | Strength | Weakness |
|----------------|----------|----------|
| **Propositional** | Decidable inference | Verbose, no structure |
| **First-order** | Compact rules | Harder inference |
| **Probabilistic** | Handles noise | Computation, elicitation |
| **Neural** | Learning from data | Opaque, data hunger |

Modern AI **combines** all - neuro-symbolic systems.

---

## From Wumpus to Real Robotics

| Wumpus | Real world |
|--------|------------|
| Discrete grid | Continuous pose |
| Fixed percepts | Noisy sensors |
| Known dynamics | Unknown physics |
| Logic KB | Logic + filters + learning |

[Chapter 25-26](../chapter-26-robotics/README.md) robotics stacks.

---

## PEAS: Choosing Representation

Ask before coding:

1. Finite vs. infinite objects?
2. Certain vs. uncertain beliefs?
3. Static vs. changing world?
4. Proof required vs. good-enough policy?

Answers pick logic / probability / learning mix.

---

## Chapter 07 Takeaways

| Section | Core idea |
|--------|-----------|
| [7.1](./section-01-knowledge-based-agents.md) | TELL / ASK architecture |
| [7.2](./section-02-propositional-logic.md) | Syntax, semantics, models |
| [7.3](./section-03-inference-and-entailment.md) | $\models$ vs. $\vdash$ |
| [7.4](./section-04-theorem-proving.md) | Resolution, CNF |
| [7.5-7.6](./section-05-forward-chaining.md) | Horn chaining |
| [7.7](./section-07-wumpus-world.md) | Integrated agent |

---

## Description Logic and Ontologies (Preview)

**Description logics** (e.g., ALC) balance expressiveness and decidability for **ontology** reasoning - "all mammals are animals," "some patients have condition X." [Chapter 10](../chapter-10-knowledge-representation/README.md) develops this for knowledge graphs and semantic web.

---

## Nonmonotonic Reasoning (Preview)

**Default rules:** "Birds typically fly" - conclusions retract when exceptions appear (penguins). Propositional logic is **monotonic**: adding KB facts never invalidates old conclusions. Real common-sense reasoning often isn't.

---

## Hybrid Wumpus Agent (AIMA)

AIMA's **hybrid agent** combines:

| Component | Handles |
|-----------|---------|
| **Logic** | Provably safe squares |
| **Probability** | Risk-ranked exploration when no safe move |
| **Planning** | Return path with gold |

This foreshadows modern agents that mix symbolic rules with learned models.

---

## Lab Extension Ideas

- Add temporal indices $Breeze_{i,j}^t$ and axiom schemas.
- Compare resolution vs. forward chaining node counts on same KB.
- Implement **possible model** enumeration for "unknown" squares.

---

## Historical Note

Propositional logic dominated early AI (1960s-80s) before probability and learning resurged. Modern systems often use **logic for constraints** and **learning for perception** - the Wumpus agent remains the pedagogical bridge between search, CSP, and knowledge-based reasoning.

---

## Summary: What You Can and Cannot Say

| Statement type | Propositional logic | Need instead |
|----------------|---------------------|--------------|
| "Cell (2,2) has a pit" | $P_{2,2}$ | - |
| "Every cell has at most one wumpus" | Exponential sentences | FOL $\forall$ |
| "Probably safe" | Not expressible | Probability |
| "After move, breeze" | Time-indexed atoms | Temporal logic |

---

## Reflection Questions

1. Why is "for all cells" awkward in propositional logic?
2. What does FOL add that propositional logic lacks?
3. When is unknown not enough - need probability?
4. Why is FOL entailment undecidable?
5. How would you extend the Wumpus agent with probabilities?

---

**Next:** [Chapter 08 - First-Order Logic](../chapter-08-first-order-logic/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Chapters 7-9 outlook. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Brachman, R. J., & Levesque, H. J. - *Knowledge Representation and Reasoning*.
3. McCarthy, J. - programs with common sense (motivation for FOL).
4. Pearl, J. - probability vs. logic.
5. MIT 6.034 - Beyond propositional logic. [OCW](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)



