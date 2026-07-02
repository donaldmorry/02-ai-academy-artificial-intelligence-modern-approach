# Section B.2: Pseudocode Conventions

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Appendix B.2  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Reading AIMA Algorithms

AIMA presents algorithms in **pseudocode** - language-independent notation balancing precision and readability. Understanding conventions is prerequisite for implementing search, logic, and learning algorithms from the text.

> **Readable form:** AIMA pseudocode is almost Python - but watch for 1-based indexing and array conventions.

---

## Variable and Assignment

```
x ← 5                    // assignment
result ← A*SEARCH(problem)  // function call returns value
```

- `←` denotes assignment (not equality)
- Variables are local unless stated global
- Arrays use subscript notation

---

## Arrays and Indexing

**AIMA uses 1-based indexing** by default:

```
A[1]           // first element
A[1..n]        // elements 1 through n
A[i+1]         // expression indexing
```

**Matrices:** $A_{i,j}$ or `A[i,j]` - row $i$, column $j$.

When implementing in Python (0-based), adjust carefully:

```python
# AIMA: for i ← 1 to LENGTH(A)
# Python:
for i in range(len(A)):  # 0 to n-1
    ...
```

> **Readable form:** Off-by-one errors are the #1 bug when porting AIMA to Python - always check indexing.

---

## Control Structures

```
if x = Goal then return x
if x > 0 then
    return POSITIVE(x)
else
    return NEGATIVE(x)

for each x in S do
    PROCESS(x)

while not GOAL(state) do
    state ← NEXT(state)

repeat
    action ← POLICY(state)
until TERMINAL(state)
```

`return` exits function immediately.

---

## Functions and Procedures

```
function A*SEARCH(problem) returns solution or failure
    ...
    return solution

procedure INSERT(node, queue)
    ...
    // no return value
```

**Input/output** specified in header:
- `function NAME(inputs) returns output`
- `procedure NAME(inputs)` - side effects only

---

## Data Structures in Pseudocode

| AIMA | Meaning |
|------|---------|
| `Queue` | FIFO - append back, pop front |
| `Stack` | LIFO - push/pop top |
| `Priority-Queue` | Pop minimum (or maximum) by key |
| `Set` | Unordered unique elements |
| `Dictionary/Map` | Key-value pairs |
| `List` | Ordered sequence |

```
INSERT(node, frontier)       // add to queue
node ← POP(frontier)         // remove per queue type
EMPTY?(queue)                // boolean test
```

---

## Common Patterns

**Best-first search skeleton:**

```
function BEST-FIRST-SEARCH(problem, f) returns solution or failure
    node ← NODE(STATE=problem.INITIAL)
    frontier ← PRIORITY-QUEUE(node, priority=f(node))
    reached ← {problem.INITIAL}
    while not EMPTY?(frontier) do
        node ← POP(frontier)
        if problem.IS-GOAL(node.STATE) then return SOLUTION(node)
        for each child in EXPAND(problem, node) do
            if child.STATE not in reached then
                INSERT(child, frontier)
                add child.STATE to reached
    return failure
```

---

## Mathematical Notation in Code

Mixed freely:

```
utility ← ∑ᵢ P(sᵢ) U(sᵢ)
Q(s,a) ← R(s,a) + γ maxₐ' Q(s',a')
```

Greek letters, summation, argmax appear inline.

---

## Comments and Indentation

Indentation shows block structure (Python-style in 4th edition).

`//` or inline comments explain non-obvious steps.

---

---

## Study Notes

Russell and Norvig present this material as foundational for multiple AI subfields. When working through exercises:

- Re-derive key formulas before looking up answers
- Implement at least one algorithm or calculation in Python
- Connect concepts to the relevant course chapter
- Test edge cases - algorithms often fail on boundaries first

> **Readable form:** Mastery means you can explain it, compute it, and code it - not just recognize the definition on a flashcard.

### Cross-Chapter Connections

| Related chapter | Connection |
|----------------|------------|
| Earlier foundations | Provides prerequisites for formal derivations |
| Applied chapters | Techniques appear in agents, learning, and perception |
| Ethics and safety | Deployment context shapes how methods are used |

### Practice Checklist

| Step | Action |
|------|--------|
| 1 | Read the AIMA section alongside this section |
| 2 | Complete all five exercises |
| 3 | Implement one code example from scratch |
| 4 | Explain one key formula in plain English |
| 5 | Identify one real-world application |

---

## Additional Examples

### Worked Example

Consider applying the concepts from this section to a concrete AI problem. Identify inputs, outputs, constraints, and evaluation criteria before implementing. Start with the smallest instance that exercises the core idea.

### Implementation Reminder

```python
# Template: verify understanding with executable code
def verify_understanding():
    """Replace with section-specific test."""
    assert True  # placeholder - implement real check
    return "ok"
```

### Summary Table

| Concept | Definition | AI application |
|---------|------------|----------------|
| Core idea | See section sections above | Chapter-specific |
| Key formula | See math blocks above | Computation |
| Pitfall | Common student error | Debugging |
| Extension | Advanced topic | Further reading |

> **Readable form:** If you can fill in this table from memory, you understand the section well enough to move on.

## Key Takeaways

1. `←` is assignment; functions return values, procedures do not.
2. AIMA defaults to 1-based array indexing - adjust for Python.
3. Queue, Stack, and Priority-Queue operations follow standard ADT semantics.
4. Algorithm headers specify inputs, outputs, and failure conditions.
5. Pseudocode mixes imperative structure with mathematical notation.

## Exercises

1. Translate AIMA loop `for i ← 1 to n do` to Python.
2. Implement `POP` and `INSERT` for a priority queue using `heapq`.
3. Rewrite the BEST-FIRST-SEARCH skeleton with explicit reached-set handling.
4. Identify off-by-one risk when accessing `A[n]` in AIMA vs. Python.
5. Write pseudocode for binary search following AIMA conventions.

---

**Previous:** [Section B.1 - BNF Notation](./section-01-bnf-notation.md)

**Next:** [Section B.3 - Algorithm Catalog](./section-03-algorithm-catalog.md)