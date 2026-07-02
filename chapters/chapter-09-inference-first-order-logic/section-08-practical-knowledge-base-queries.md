# Section 9.8: Practical Knowledge Base Queries

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 9  
> **Previous:** [Section 9.7 - Theorem Provers](./section-07-theorem-provers.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Putting It Together: KB + Inference

A practical knowledge-based system combines:

1. **Ontology** - vocabulary and constraints ([Chapter 08](../chapter-08-first-order-logic/README.md))
2. **Facts** - ground assertions
3. **Rules** - definite clauses or general axioms
4. **Inference engine** - chaining or resolution
5. **Query interface** - ASK with explanations

> **Readable form:** The KB is the library; inference is the librarian who finds answers and shows which books (rules) were used.

---

## Query Patterns

| Query type | Example | Engine |
|------------|---------|--------|
| **Ground** | $Grandparent(John,Sue)$? | BC or FC |
| **Existential** | $\exists x \, Teaches(x, AI101)$ | BC with bindings |
| **Universal** | $\forall s \, Student(s) \Rightarrow \ldots$ | Usually not queried directly |
| **Negation** | $\lnot Criminal(John)$? | CWA or refutation |

**Open-world:** failure to prove $\not\models$ query does not mean negation holds.

---

## Explanation and Justification

Users need **proof traces**:

```
Grandparent(John,Sue) ← Parent(John,Mary) ∧ Parent(Mary,Sue)  [rule gp]
  Parent(John,Mary)   ← fact
  Parent(Mary,Sue)    ← fact
```

Truth maintenance systems ([Chapter 10](../chapter-10-knowledge-representation/README.md)) extend this with dependency graphs.

---

## Case Study: Family KB

```python
from logic import *

kb = FolKB()
kb.tell(expr('forall x,y Parent(x,y) ==> Ancestor(x,y)'))
kb.tell(expr('forall x,y,z Ancestor(x,y) & Ancestor(y,z) ==> Ancestor(x,z)'))
kb.tell(expr('Parent(John,Mary)'))
kb.tell(expr('Parent(Mary,Sue)'))

list(fol_bc_ask(kb, expr('Ancestor(John,Sue)')))  # [{...}]
```

---

## Performance Engineering

1. **Profile** which predicates appear in most rules - index there.
2. **Batch TELL** - avoid re-indexing per fact.
3. **Materialized views** - cache frequent derivations.
4. **Horn-ify** when possible - avoid resolution overhead.

---

## Chapter 09 Synthesis

| Section | Topic |
|--------|-------|
| 9.1 | Grounding vs. lifting |
| 9.2 | Unification |
| 9.3 | Forward chaining / GMP |
| 9.4 | Backward chaining |
| 9.5 | Resolution |
| 9.6 | Efficiency & termination |
| 9.7 | Theorem provers |
| 9.8 | Practical KB queries |

**Next:** representational ontologies beyond raw FOL ([Chapter 10](../chapter-10-knowledge-representation/README.md)).

---

## Query API Design

```python
class LogicalAgent:
    def tell(self, sentence): ...
    def ask(self, query) -> bool: ...
    def ask_with_proof(self, query) -> tuple[bool, ProofTree]: ...
    def ask_bindings(self, query) -> list[dict]: ...
```

**ask_with_proof** essential for debugging engineered KBs - users trust answers they can audit.

---

## Incremental Inference

When new fact arrives:

- **Forward:** add to agenda, re-fire rules
- **Backward:** invalidate cached proofs touching affected predicates
- **TMS:** update justification graph ([Section 10.6](../chapter-10-knowledge-representation/section-06-truth-maintenance.md))

Batch updates more efficient than per-fact recompilation in large systems.

---

## Integration with Planning

Planning reduces to proving **reachable goal** in situation calculus - queries like:

$$
\exists s \, Situation(s) \land Holds(Goal, s) \land Reachable(s_0, s)
$$
> **Readable form:** The existential quantifier says at least one object must make the stated relationship true.

Practical planners bypass general FOL provers ([Chapter 11](../chapter-11-automated-planning/README.md)) - but logical framing unifies KR and planning.

---

## Chapter 09 Lab Preview

**Family relations KB:**

1. Implement `unify` with occurs check
2. Add Horn rules for parent, grandparent, ancestor
3. `bc_ask` for queries with proof trace
4. Test edge cases: cycles, missing facts, existential queries

Validates sections 9.2-9.4 in working code.

---

## Study Exercise: Proof Trace UI

Design ASCII or HTML format for proof trees from backward chaining. Include:

- Rule name / id
- Unifier applied
- Depth in search tree

User testing: can classmates verify $Ancestor(John,Sue)$ from your trace alone?


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed. This four-step checklist catches most knowledge-engineering errors early.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 Part III is cumulative - inference (Chapter 09), representation (Chapter 10), and planning (Chapter 11) share the same logical foundations from Chapters 07-08.


---

## Reflection Questions

1. Design ASK interface returning proof traces.
2. How handle existential queries in backward chaining?
3. When use closed-world vs. open-world for negation?
4. Profile a family KB: which predicates to index?
5. Map chapter lab (ancestor queries) to inference choice.

---

**Next:** [Chapter 10 - Knowledge Representation](../chapter-10-knowledge-representation/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Chapter 9 summary.
2. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python) - complete logic chapter.
3. Protégé - ontology + reasoner integration.
4. Doyle, J. - truth maintenance preview.
5. MIT 6.034 - FOL inference labs.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **ASK** | Agent query interface testing KB entailment. |
| **Proof trace** | Justification showing which rules and facts derived answer. |
| **Materialized view** | Cached inference results for fast repeat queries. |
| **Open-world query** | Failure to prove does not imply negation. |
| **Existential query** | Question ∃x P(x) seeking witness binding. |
| **KB architecture** | Layered ontology, facts, rules, and engine. |




---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
