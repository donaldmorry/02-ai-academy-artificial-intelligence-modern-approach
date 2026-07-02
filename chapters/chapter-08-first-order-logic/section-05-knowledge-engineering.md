# Section 8.5: Knowledge Engineering

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 8.4  
> **Previous:** [Section 8.4 - Using FOL](./section-04-using-fol.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## What Is Knowledge Engineering?

**Knowledge engineering** is the process of entering **declarative knowledge** into a system so it can solve problems without those solutions being explicitly programmed. The engineer:

1. Identifies **objects and relations** in the domain
2. Designs an **ontology** (vocabulary + constraints)
3. Writes **general rules** and **specific facts**
4. Validates with **queries** and **test cases**

Russell and Norvig compare it to programming - but you declare **what is true**, not **how to compute** answers. Inference ([Chapter 09](../chapter-09-inference-first-order-logic/README.md)) derives answers.

> **Readable form:** Knowledge engineering is building a textbook about a domain in logic - the theorem prover reads the book and answers questions.

---

## The Knowledge Engineering Process

### Step 1: Identify the questions

What will users **ask**? For a campus KB:

- Who teaches AI101?
- Which rooms are in MainHall?
- Is every CS student advised by a faculty member?

Questions drive vocabulary - not the reverse.

### Step 2: Catalog objects and categories

| Category | Instances | Notes |
|----------|-----------|-------|
| People | students, faculty | may overlap roles |
| Buildings | MainHall, LabB | physical objects |
| Courses | AI101, Logic201 | abstract objects |
| Events | Enroll, Teach | reified in [Chapter 10](../chapter-10-knowledge-representation/README.md) |

Use unary predicates or **types**: $Student(x)$, $Faculty(x)$, $Course(c)$.

### Step 3: Generalize from instances

Instead of 500 facts $Teaches(Prof_i, Course_j)$, add:

$$
\forall p,c \, Teaches(p,c) \Rightarrow Faculty(p) \land Course(c)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

$$
\forall s,c \, Enrolled(s,c) \Rightarrow Student(s) \land Course(c)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

**Domain constraints** shrink invalid models.

### Step 4: Formalize and test

Add sentences incrementally; run entailment checks after each batch. Failed queries reveal missing axioms or wrong quantifier scope.

---

## Ontology Design Principles

### Reuse standard vocabularies

When possible, adopt **SUMO**, **Schema.org**, or **OWL** upper ontologies ([Section 10.8](../chapter-10-knowledge-representation/section-08-knowledge-graphs.md)).

### Prefer stable predicates

"Room temperature" changes; "Room is part of Building" is stable. Separate **fluents** (time-varying) from **structural** relations.

### Normal form for relations

| Style | Example |
|-------|---------|
| Binary relation | $Enrolled(student, course)$ |
| Role reification | $Enrollment(e) \land StudentOf(e,s) \land CourseOf(e,c)$ |

Reification helps when relations have attributes (grade, semester).

---

## A Campus Map Case Study

### Vocabulary

$Building$, $Room$, $Department$, $Person$, $In$, $WorksIn$, $Teaches$, $Enrolled$, $Chair$

### Structural facts

$$
Building(MainHall), \quad Room(R101), \quad In(R101, MainHall)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

$$
Department(CS), \quad WorksIn(Alice, CS), \quad Chair(Alice, CS)
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

### Rules

$$
\forall r,b \, Room(r) \land In(r,b) \Rightarrow Building(b)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

$$
\forall p,d \, Chair(p,d) \Rightarrow Faculty(p) \land Department(d)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

### Query examples

| Query | Expected |
|-------|----------|
| $Building(MainHall)$ | Entailed by fact |
| $In(R101, MainHall)$ | Entailed |
| $Faculty(Alice)$ | Entailed via Chair rule |

---

## Pitfalls and Ambiguities

| Pitfall | Example | Fix |
|---------|---------|-----|
| **Wrong quantifier order** | "Every student likes some professor" vs. one professor all like | Swap $\forall/\exists$ |
| **Missing domain closure** | Unintended models with extra objects | **Closed-world** assumptions ([Chapter 10](../chapter-10-knowledge-representation/README.md)) |
| **Function totality** | $Mother(x)$ undefined for non-persons | Type constraints |
| **Inconsistent axioms** | $Chair(p,d) \land \lnot Faculty(p)$ | Test for $KB \models \bot$ |

---

## Modular KB Architecture

```
Upper ontology (Thing, Part, Agent)
        ↓
Domain ontology (University, Course)
        ↓
Application facts (Alice, AI101, 2026)
```

```python
def build_campus_kb():
    kb = FOLKnowledgeBase()
    kb.tell(expr('forall r,b Room(r) & In(r,b) ==> Building(b)'))
    kb.tell(expr('In(R101, MainHall)'))
    return kb
```

---

## Validation Checklist

1. **Coverage** - do answers match expert expectations on 20+ queries?
2. **Soundness** - are rules too strong (deriving false positives)?
3. **Completeness** - are rules too weak (missing entailments)?
4. **Maintainability** - can new courses be added without rewriting axioms?
5. **Efficiency** - does inference terminate in reasonable time? ([Chapter 09](../chapter-09-inference-first-order-logic/README.md))

---

## Connection to Planning and KR

Knowledge engineering for **actions** requires **effect axioms** ([Chapter 11](../chapter-11-automated-planning/README.md)) and **frame axioms** ([Section 10.7](../chapter-10-knowledge-representation/section-07-the-frame-problem.md)).

---

## Documentation and Team Workflow

Professional ontology projects maintain:

| Artifact | Purpose |
|----------|---------|
| **Glossary** | Natural-language definitions per symbol |
| **Competency questions** | Queries the KB must answer |
| **Axiom changelog** | Track why each sentence was added |
| **Test suite** | Expected entailments and non-entailments |

**Iterative refinement:** start with 5-10 competency questions; add axioms until they pass; repeat as scope grows.

---

## Worked Exercise: Library Domain

**Competency question:** "Which patrons have overdue books?"

Vocabulary: $Patron$, $Book$, $Loan$, $DueDate$, $Returned$, $Copy$

$$
\forall l \, Loan(l) \Rightarrow \exists p,b \, PatronOf(l,p) \land BookOf(l,b)
$$
> **Readable form:** The universal quantifier makes this rule apply to every object assignment that satisfies the variables and predicates.

$$
Overdue(l) \Leftrightarrow Loan(l) \land \lnot Returned(l) \land DueDate(l) < Today
$$
> **Readable form:** This first-order sentence states a model constraint using objects, predicates, functions, and logical connectives.

Query: $\exists l,p \, Overdue(l) \land PatronOf(l,p) \land Name(p, alice)$ - tests whether Alice has overdue loans.

---

## Scaling to Enterprise Ontologies

Large organizations partition by **bounded context** (DDD terminology):

- **HR ontology** - employees, roles, reporting
- **Facilities ontology** - buildings, rooms
- **Bridge axioms** - $WorksIn(p, d) \land LocatedIn(d, b) \Rightarrow WorksNear(p, b)$

Inference across chapters requires **consistency checking** at integration points - a task for DL reasoners when fragments map to OWL.


---

## Campus KB Competency Questions (Expanded)

| # | Question | FOL query |
|---|----------|-----------|
| 1 | Is R101 in MainHall? | $In(R101, MainHall)$ |
| 2 | Who chairs CS? | $\exists p \, Chair(p, CS)$ |
| 3 | Are all chairs faculty? | $\forall p,d \, Chair(p,d) \Rightarrow Faculty(p)$ |
| 4 | Which courses does Alice teach? | $\exists c \, Teaches(Alice,c)$ |
| 5 | Does every student have an advisor? | $\forall s \, Student(s) \Rightarrow \exists f \, Advisor(f,s)$ |

Validate each after incremental TELL - catches ontology gaps early.

---

## Modular Import Pattern

```python
def load_kb():
    kb = FOLKB()
    kb.import_module('upper_ontology.owl')
    kb.import_module('university.owl')
    kb.import_module('campus_facts_2026.owl')
    return kb
```

**Versioning** - facts chapter updates each semester without touching axioms.

---

## Reflection Questions

1. Design vocabulary for a library domain (books, patrons, loans). List 10 predicates.
2. Write a rule: "only faculty can chair a department" - watch for exceptions (acting chairs).
3. When would you reify an enrollment as an object instead of a binary predicate?
4. What queries would expose a wrong $\forall/\exists$ swap in a student-advisor KB?
5. How does modular ontology design parallel software package structure?

---

**Next:** [Section 8.6 - Equality and Functions](./section-06-equality-and-functions.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/), Section 8.4.
2. Noy, N. F., & McGuinness, D. L. - ontology development 101. [Protege](https://protege.stanford.edu/)
3. Gruber, T. R. - "A translation approach to portable ontology specifications."
4. SUMO ontology - upper-level vocabulary. [ontologyportal.org](http://www.ontologyportal.org/)
5. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python) - `fol_bc_ask` on engineered KBs.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Knowledge engineering** | Art and practice of building logical knowledge bases for a domain. |
| **Ontology** | Specification of conceptual vocabulary and constraints for a domain. |
| **Domain constraint** | Sentence ruling out unintended interpretations (types, roles). |
| **Reification** | Representing a relation or event as an object with its own properties. |
| **Upper ontology** | General categories (space, time, agent) reused across domains. |
| **Validation** | Testing whether a KB entails expected and only expected conclusions. |
| **Modular KB** | Layered knowledge base with reusable upper and domain layers. |
