# Section 9.7: Theorem Provers

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 9.5  
> **Previous:** [Section 9.6 - Efficiency and Termination](./section-06-efficiency-and-termination.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Automated Theorem Provers

**Resolution** is the theoretical backbone; **production provers** add heuristics, equality, and theories.

| Prover | Style | Notes |
|--------|-------|-------|
| **Vampire** | Resolution + superposition | CASC winner |
| **E** | Equational | Fast on equality-heavy problems |
| **SPASS** | Resolution + splitting | First-order with equality |
| **Prover9/Mace4** | Resolution + model finder | Educational |

---

## Architecture of Modern Provers

1. **CNF conversion** (including skolemization)
2. **Preprocessing** (pure literal deletion, subsumption)
3. **Given-clause loop:** pick clause, generate resolvents, retain best
4. **Termination:** empty clause or resource limit

**Superposition:** inference rule for equality - replaces paramodulation in modern systems.

---

## Prolog as Theorem Prover

Prolog executes **Horn clauses** via backward chaining - incomplete for full FOL but efficient for rules + facts.

| Feature | Prolog | Full prover |
|---------|--------|-------------|
| Fragment | Horn | Full CNF |
| Search | Depth-first | Clause selection heuristics |
| Completeness | No (rule order) | Refutation complete |
| Speed | Interactive | Batch |

**Constraint logic programming** extends Prolog with arithmetic constraints.

---

## TPTP and Competitions

**TPTP** (Thousands of Problems for Theorem Provers) - standard benchmark library.

**CASC** (CADE ATP System Competition) - annual prover evaluation.

Useful for comparing algorithms on scheduling, algebra, and puzzle problems.

---

## Integration with Agents

Logical agents rarely embed Vampire directly. Typical pipeline:

1. Engineer Horn KB ([Chapter 08](../chapter-08-first-order-logic/README.md))
2. Use **backward chaining** for interactive queries
3. Escalate hard goals to external prover via TPTP export

```python
# Simplified: export to TPTP format
def to_tptp(kb):
    lines = ["fof(axiom1, axiom, ...).", "fof(conjecture, conjecture, ...)."]
    return "\n".join(lines)
```

---

## When to Use Which Engine

```
Horn rules + interactive queries     → Prolog / backward chaining
Large CNF, equality, open theory     → Vampire / E
Finite domain combinatorics          → SAT / ASP
Ontology subsumption                 → DL reasoner (HermiT)
```

---

## Vampire Pipeline (Simplified)

1. Parse TPTP CNF
2. **Preprocess:** flatten, propagate units, remove tautologies
3. **Clause selection:** pick given clause by weight heuristic
4. **Infer:** resolution, superposition, equality reasoning
5. **Simplify:** subsumption, demodulation
6. Repeat until $\square$ or limit

Competition configs tune hundreds of parameters.

---

## E Prover and Equality

**E** specializes in **equational** theories - algebraic structures where $=$ is central.

Superposition calculus handles equality without exploding clause count as naive paramodulation might.

---

## Prolog vs. Vampire: Same Problem

Horn problem: "Is $Grandparent(john,sue)$ entailed?"

- **Prolog:** milliseconds via backward chaining
- **Vampire:** converts to CNF, runs general resolution - slower but handles non-Horn if needed

Choose tool matching **logical fragment**.

---

## Educational Tools

| Tool | Role |
|------|------|
| aima-python | Pedagogical chaining, unification |
| Prover9 | Classroom resolution |
| TPTP problem library | Benchmark problems |
| Metamath | Formal proof verification (different tradition) |

---

## Study Exercise: TPTP Export

Write three axioms in TPTP `fof` syntax for kinship KB. Run Vampire (if installed) or read CASC results for similar problems.

Compare runtime to aima-python `fol_bc_ask` on same Horn fragment.


---

## AIMA Study Supplement

Russell and Norvig develop this topic as part of the knowledge-based agent pipeline: represent facts declaratively, infer answers with sound rules, and validate against competency questions. Re-read the corresponding AIMA section and work the end-of-chapter exercises - translation practice, hand traces of algorithms, and short proofs build the fluency these chapters assume.

**Worked habit:** for each new predicate or action schema, write (1) one ground instance, (2) one general rule, (3) one query, and (4) one negative control case that should **not** be entailed. This four-step checklist catches most knowledge-engineering errors early.

**Connection:** link this section's concepts to the prior chapter's vocabulary and to the next section's algorithms. Course 2 Part III is cumulative - inference (Chapter 09), representation (Chapter 10), and planning (Chapter 11) share the same logical foundations from Chapters 07-08.


---

## Reflection Questions

1. Compare Vampire and Prolog on the same Horn problem.
2. What is TPTP used for?
3. When export to external prover vs. embedded chaining?
4. What is superposition for equality?
5. Name two preprocessing steps in modern provers.

---

**Next:** [Section 9.8 - Practical Knowledge Base Queries](./section-08-practical-knowledge-base-queries.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Section 9.5.
2. Vampire prover - [vprover.github.io](https://vprover.github.io/)
3. Sutcliffe, G. - TPTP and CASC.
4. Clocksin & Mellish - Prolog.
5. Nieuwenhuis, R. - superposition calculus.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Theorem prover** | Automated system for FOL refutation. |
| **TPTP** | Standard benchmark language for ATP. |
| **CASC** | Annual automated theorem proving competition. |
| **Superposition** | Modern equality inference rule. |
| **Given-clause algorithm** | Iterative clause selection in provers. |
| **Horn fragment** | Definite clauses efficiently handled by Prolog. |

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
