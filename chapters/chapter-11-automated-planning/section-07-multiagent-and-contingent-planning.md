# Section 11.7: Multiagent and Contingent Planning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 11.7
> **Previous:** [Section 11.6 - Scheduling and Resources](./section-06-scheduling-and-resources.md)
> **Next:** [Section 11.8 - PDDL and Practical Systems](./section-08-pddl-and-practical-systems.md)
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Multiagent Planning

Multiple agents acting in shared environment - plans must coordinate or compete.

**Joint plan:** combined action sequence for all agents.

**Distributed planning:** each agent plans locally with communication.

> **Readable form:** Two robots in one room need plans that don't crash them into each other.

---

## Types

| Setting | Feature |
|---------|---------|
| **Cooperative** | Shared goal |
| **Competitive** | Adversarial ([Chapter 05](../chapter-05-adversarial-search-games/README.md)) |
| **Mixed** | Teams + opponents |

---

## Contingent and Conformant Planning

**Continggent planning:** **sensing actions** branch on observations - plan is tree, not sequence.

**Conformant planning:** no sensing; plan must work for all initial states in belief set.

Violates classical fully observable assumption - bridge to [Chapter 12](../chapter-12-quantifying-uncertainty/README.md).

---

## Communication Actions

Agents share information via **tell** or **observe** actions in multiagent epistemic settings.

---

## Decentralized POMDP

Formal framework for uncertain multiagent - beyond classical planning scope but motivates extensions.

---

## Practical Approach

1. **Centralized planner** - single STRIPS with joint actions (exponential)
2. **Distributed HTN** - allocate tasks per agent
3. **Market-based** - agents bid for tasks

Robotics often uses centralized plan + decentralized execution with reactive collision avoidance.

---

## Multi-Robot Mutex

Two robots cannot occupy same location - **mutex** on $At(r, loc)$ fluent.

Joint state space exponential in robots - **factored planning** exploits independence.

---

## Contingent Plan Tree

```
CheckWeather
  ├─ rain → [IndoorRoute]
  └─ clear → [OutdoorRoute]
```

Branch on sensing action outcomes - plan is tree, not list.

---

## Belief State Planning

When initial state uncertain, plan from **belief state** (set of possible worlds).

Conformant: one plan works for all - conservative.

Contingent: branches adapt - more efficient when sensing available.

---

## Study Exercise: Two-Robot Mutex

Encode mutual exclusion on location in STRIPS. Does joint state space size grow as $|S_1| \times |S_2|$?

## Worked Example: Multiagent and Contingent Planning

Apply the ideas from **Joint plans, sensing actions preview** to a small concrete instance. State assumptions explicitly, write the governing equations, and interpret each term. Compare your result with the AIMA chapter exercise that matches Chapter 11.7.

> **Readable form:** Translate the abstract definition into numbers or code for one toy case.


## Design Checklist

| Step | Action |
|------|--------|
| 1 | Identify variables and constraints for multiagent and contingent planning |
| 2 | Choose representation aligned with Joint plans, sensing actions preview |
| 3 | Verify against AIMA Chapter 11.7 definitions |
| 4 | Implement minimal prototype in Python |
| 5 | Test edge cases and document failure modes |


## Connections Across Course 2

This section on **Multiagent and Contingent Planning** links forward to later chapters that reuse the same abstractions. Revisit [../../GLOSSARY.md](../../GLOSSARY.md) entries while reading, and cross-check notation with [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md). Prior search and logic chapters provide algorithmic scaffolding; probability chapters supply uncertainty semantics when Joint plans, sensing actions preview involves partial information.


## Key Takeaways

1. **Multiagent and Contingent Planning** - core idea 1 from AIMA Chapter 11.7 (Joint plans, sensing actions preview).
2. **Multiagent and Contingent Planning** - core idea 2 from AIMA Chapter 11.7 (Joint plans, sensing actions preview).
3. **Multiagent and Contingent Planning** - core idea 3 from AIMA Chapter 11.7 (Joint plans, sensing actions preview).
4. **Multiagent and Contingent Planning** - core idea 4 from AIMA Chapter 11.7 (Joint plans, sensing actions preview).
5. **Multiagent and Contingent Planning** - core idea 5 from AIMA Chapter 11.7 (Joint plans, sensing actions preview).


## Common Mistakes

- **Confusing syntax with semantics** in Multiagent and Contingent Planning - symbols must map to models or distributions.
- **Skipping units and domains** when Joint plans, sensing actions preview involves continuous or structured variables.
- **Treating heuristics as guarantees** - always state when optimality or soundness holds.
- **Ignoring complexity** - tractability assumptions in Chapter 11.7 may fail at scale.
- **Omitting sensor/actuator noise** when deploying ideas from textbook toy domains.


## Deeper Reading

Re-read Russell & Norvig Chapter 11.7 with pen and paper. For **Multiagent and Contingent Planning**, derive each formula from first principles before using libraries. The aima-python repository implements many algorithms from this chapter - compare your pseudocode to the reference implementation and note API differences.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 is cumulative - search, logic, probability, and learning share representational foundations.

---

## Practice Trace

Hand-execute the main algorithm from this section on a toy instance small enough to fit on one page. Record each intermediate structure (clause set, belief state, plan layer, or gradient step). Compare your trace to aima-python reference code if available.

> **Readable form:** If you cannot trace it by hand on paper, you do not yet understand it well enough to deploy it.


---

## Practice Trace

Hand-execute the main algorithm from this section on a toy instance small enough to fit on one page. Record each intermediate structure (clause set, belief state, plan layer, or gradient step). Compare your trace to aima-python reference code if available.

> **Readable form:** If you cannot trace it by hand on paper, you do not yet understand it well enough to deploy it.

## Study Note

Review the AIMA Chapter 11.7 exercises and trace one algorithm by hand before the lab.

---

## Key Terms (Glossary)

- **Joint** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **plans** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **sensing** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **actions** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **preview** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. What is the central idea of Multiagent and Contingent Planning in AIMA?
2. How does Multiagent and Contingent Planning relate to Joint plans, sensing actions preview?
3. Give a concrete application of Multiagent and Contingent Planning.
4. What assumptions does the textbook emphasize for Multiagent and Contingent Planning?
5. When would an alternative to Multiagent and Contingent Planning be preferable?
6. How would you verify understanding of Multiagent and Contingent Planning in code?

---

**Previous:** [Section 11.6 - Scheduling and Resources](./section-06-scheduling-and-resources.md) · **Next:** [Section 11.8 - PDDL and Practical Systems](./section-08-pddl-and-practical-systems.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. aimacode/aima-python - logic, probability, planning, learning. [GitHub](https://github.com/aimacode/aima-python)
3. ['Ghallab, M., Nau, D., & Traverso, P. (2004). *Automated Planning*. Morgan Kaufmann.']

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
