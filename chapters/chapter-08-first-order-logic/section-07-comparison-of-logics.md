# Section 8.7: Comparison of Logics

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 8 (summary)  
> **Previous:** [Section 8.6 - Equality and Functions](./section-06-equality-and-functions.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## A Landscape of Logical Languages

[Chapter 07](../chapter-07-logical-agents/README.md) introduced **propositional logic**. This chapter extended to **first-order logic**. AI systems choose representations along axes:

| Axis | Options |
|------|---------|
| **Objects** | None (props) → individuals (FOL) → predicates-as-objects (higher-order) |
| **Uncertainty** | Classical true/false → probability → fuzzy |
| **Time** | Static → temporal logics → event calculus |
| **Defaults** | Monotonic → circumscription, default rules |
| **Computability** | Decidable fragments vs. full expressiveness |

No single logic wins everywhere - **engineering trade-offs**.

---

## Propositional vs. First-Order Logic

| | Propositional | First-order |
|---|---------------|-------------|
| **Atoms** | $P$, $Q$ | $Parent(x,y)$ |
| **Variables** | No | Yes |
| **Quantifiers** | No | $\forall$, $\exists$ |
| **Models** | Truth assignments | Domains + interpretations |
| **Entailment** | Decidable | Semi-decidable |

**When propositional still wins:** finite small domains fully grounded; hardware SAT; planning as propositional SAT ([Section 11.4](../chapter-11-automated-planning/section-04-sat-planning.md)).

---

## Second-Order and Higher-Order Logic

**Second-order logic (SOL)** quantifies over **predicates** and **functions**:

$$
\forall P \, \bigl(P(0) \land \forall n \, P(n) \Rightarrow P(S(n))\bigr) \Rightarrow \forall n \, P(n)
$$
> **Readable form:** Second-order logic lets you say "for every property P..." - powerful for math, expensive for automation.

**Cost:** SOL entailment is **highly undecidable**.

---

## Description Logics (DL)

**Description logics** are **decidable fragments** of FOL designed for **ontology** reasoning ([Chapter 10](../chapter-10-knowledge-representation/README.md)).

| DL syntax | Meaning |
|-----------|---------|
| $Person$ | concept (unary predicate) |
| $Parent \sqsubseteq Person$ | every parent is a person |
| $\exists hasChild.Person$ | has at least one child who is a person |

**OWL** maps to DL - reasoners (HermiT, Pellet) check subsumption.

---

## Decidable Fragments of FOL

| Fragment | Restriction | Use |
|----------|-------------|-----|
| **Horn clauses** | At most one positive literal per clause | Forward/backward chaining |
| **Datalog** | Horn + no functions | Database rules, polynomial |
| **Guarded fragment** | Quantifiers guard atoms | Decidable entailment |

[Chapter 09](../chapter-09-inference-first-order-logic/README.md) focuses on **Horn** + general resolution.

---

## Logic vs. Probability

| Need | Logic | Probability |
|------|-------|-------------|
| "All birds fly" | $\forall x \, Bird(x) \Rightarrow Flies(x)$ | $P(Flies \mid Bird) \approx 0.95$ |
| Contradiction | Explosion | Redistribute belief |

[Chapter 12](../chapter-12-quantifying-uncertainty/README.md) starts the probability thread.

---

## Temporal and Modal Extensions

**Temporal logic:** operators $Always$, $Eventually$, $Until$ over time-indexed sentences.

**Modal logic:** $K_a \phi$ - agent $a$ knows $\phi$ ([Section 10.3](../chapter-10-knowledge-representation/section-03-mental-objects.md)).

---

## Choosing a Logic: Decision Guide

```
Need guaranteed fast inference on taxonomies?  → Description logic / OWL
Need general rules + data (Horn)?            → Datalog / Prolog
Need full FOL theorems?                      → Resolution (semi-decidable)
Need defaults ("typically")?                 → Nonmonotonic logics
Need uncertainty?                            → Bayesian networks
Finite combinatorial search?                 → Propositional SAT
```

---

## Chapter 08 Synthesis

You can now write FOL, interpret models, engineer domain KBs, use equality and functions, and place FOL among alternative logics.

**Next step:** automated **inference** - unification, chaining, resolution ([Chapter 09](../chapter-09-inference-first-order-logic/README.md)).

---

## Lab Preview: Campus Map KB

Encode 20+ sentences for buildings, departments, people. Validate with backward chaining lab ([Chapter 09](../chapter-09-inference-first-order-logic/README.md)).

---

## Intensional vs. Extensional

| | Intensional | Extensional |
|---|-------------|-------------|
| Meaning | Definition / concept | Set of instances |
| Example | "Student" by criteria | Enumeration {Alice, Bob} |
| FOL | Unary predicate + rules | Ground atoms + domain closure |

Most KR systems are **intensional** - new instances automatically inherit via rules.

---

## Expressiveness and Complexity Trade-off

```
More expressive ──────────────────────────────►
   Propositional    Datalog    Horn FOL    Full FOL    SOL
Less tractable ◄──────────────────────────────
```

**Engineering rule:** use the **weakest** logic that answers your competency questions - faster inference, fewer surprises.

---

## Historical Note: Logicist AI

McCarthy, Hayes, and others envisioned **general-purpose** logical agents. Description logics, Prolog, and OWL emerged as **practical fragments** when full FOL proved too heavy for interactive systems. Modern LLM agents revisit natural language ↔ logic translation ([Chapter 28](../chapter-28-future-of-ai/README.md)) - the representational questions of Chapter 08 remain central.

---

## Review Checklist

Before leaving Chapter 08, verify you can:

1. Translate five English sentences with correct quantifier scope
2. Sketch a model $\langle \mathcal{D}, I \rangle$ for a tiny domain
3. Write domain constraints for a campus KB
4. State when UNA and CWA apply
5. Place FOL among propositional, DL, and SOL alternatives

---

## Study Guide: Logic Selection Scenarios

| Scenario | Recommended logic | Rationale |
|----------|-------------------|-----------|
| Medical taxonomy | OWL / DL | Subsumption, decidability |
| Business rules engine | Horn + forward chain | Fast, explainable |
| Math textbook proofs | HOL / Isabelle | Higher-order needed |
| Wumpus with noise | Probability + logic | Uncertain percepts |
| Blocks-world planner | Propositional / STRIPS | Finite, combinatorial |

Work through AIMA Exercise 8.x style translations for each row - practice mapping problems to logics.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed. This four-step checklist catches most knowledge-engineering errors early.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 Part III is cumulative - inference (Chapter 09), representation (Chapter 10), and planning (Chapter 11) share the same logical foundations from Chapters 07-08.


---

## Description Logic Expressivity Ladder

| DL | New feature |
|----|-------------|
| **EL** | $\exists$, $\sqcap$, subsumption |
| **ALC** | full negation, $\forall$ |
| **SHOIN** | transitive roles, nominals - OWL DL base |
| **SROIQ** | richer role chains - OWL 2 DL |

**Trade-off:** each step increases reasoning complexity.

---

## Logic Selection Decision Matrix

| Requirement | Choose |
|-------------|--------|
| Taxonomy only | EL / OWL EL profile |
| Rules + constraints | OWL DL + SWRL (careful) |
| General FOL theorems | Vampire |
| Streaming Horn rules | Datalog |
| Defaults | ASP / circumscription |
| Probability | MLNs / Bayesian nets |

---

## Reflection Questions

1. Why is full FOL avoided in real-time ontology reasoners?
2. Give a property expressible in SOL but not single-sentence FOL.
3. Is Datalog more or less expressive than propositional logic?
4. When would you choose SAT over FOL resolution for a finite domain?
5. How does OWL's open-world semantics differ from SQL NULL semantics?

---

**Next:** [Chapter 09 - Inference in First-Order Logic](../chapter-09-inference-first-order-logic/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Chapter 8 summary.
2. Baader, F., et al. - *Description Logic Handbook*.
3. W3C OWL 2 - Web Ontology Language. [w3.org/TR/owl2-overview](https://www.w3.org/TR/owl2-overview/)
4. Reiter, R. - nonmonotonic vs. classical logic.
5. Stanford CS221 - logic selection in practice.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Second-order logic** | Logic allowing quantification over predicates and functions. |
| **Description logic** | Decidable FOL fragment for concepts, roles, and ontologies. |
| **Datalog** | Horn rules without function symbols; polynomial inference. |
| **Horn clause** | Disjunction with at most one positive literal; enables chaining. |
| **Decidable fragment** | Subset of logic with terminating entailment algorithm. |
| **OWL** | Standard web ontology language based on description logics. |
| **Semi-decidable** | If $\alpha \models \beta$, a procedure exists that eventually proves it; if not, may not halt. |



