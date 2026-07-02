# Section B.5: Recursion Patterns

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Appendix B.5  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Recursive Thinking in AI

Many AIMA algorithms are naturally **recursive** - depth-first search, game trees, divide-and-conquer, grammar derivations, and backtracking. Understanding recursion patterns enables clean implementations and correct base cases.

> **Readable form:** Recursion is a function calling itself on smaller pieces - define the trivial case, then reduce everything to that.

---

## Anatomy of Recursion

Every recursive function needs:

1. **Base case** - stops recursion (prevents infinite loop)
2. **Recursive case** - calls self on strictly smaller/simpler input
3. **Progress** - each call moves toward base case

```
function FACTORIAL(n)
    if n ≤ 1 then return 1
    else return n * FACTORIAL(n - 1)
```

---

## Common Patterns

### Linear Recursion

Single recursive call - factorial, list sum:

```python
def sum_list(lst):
    if not lst:
        return 0
    return lst[0] + sum_list(lst[1:])
```

### Tree Recursion

Multiple recursive calls - Fibonacci (inefficient), minimax:

```python
def minimax(state, depth):
    if depth == 0 or is_terminal(state):
        return evaluate(state)
    if is_max_player(state):
        return max(minimax(child, depth-1) for child in children(state))
    return min(minimax(child, depth-1) for child in children(state))
```

### Divide and Conquer

Split problem in half - merge sort, quicksort:

```
function MERGE-SORT(A, lo, hi)
    if lo < hi then
        mid ← (lo + hi) / 2
        MERGE-SORT(A, lo, mid)
        MERGE-SORT(A, mid+1, hi)
        MERGE(A, lo, mid, hi)
```

---

## Backtracking

Try assignment, recurse, undo if failure:

```python
def backtrack(assignment, variables, domains, constraints):
    if complete(assignment):
        return assignment
    var = select_unassigned(variables, assignment)
    for value in order_domain_values(var, domains):
        if consistent(var, value, assignment, constraints):
            assignment[var] = value
            result = backtrack(assignment, variables, domains, constraints)
            if result is not None:
                return result
            del assignment[var]  # undo
    return None
```

Used in CSP ([Chapter 06](../chapter-06-constraint-satisfaction/README.md)), propositional satisfiability.

> **Readable form:** Backtracking is "try it, go deeper, if stuck erase and try something else."

---

## Tail Recursion

Recursive call is **last operation** - can be optimized to iteration by compiler:

```python
def factorial_tail(n, acc=1):
    if n <= 1:
        return acc
    return factorial_tail(n - 1, n * acc)
```

Python does not optimize tail calls - use iteration or `trampoline` for deep recursion.

---

## Memoization

Cache results to avoid redundant subproblems - **dynamic programming**:

```python
def fib_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo)
    return memo[n]
```

Transforms exponential tree recursion to $O(n)$ time.

**Top-down (memoization)** vs. **bottom-up (tabulation)**.

---

## Recursion Depth Limits

Python default recursion limit ~1000 - deep DFS may hit `RecursionError`.

**Solutions:**
- Convert to iterative with explicit stack
- `sys.setrecursionlimit()` (use cautiously)
- IDA* instead of depth-limited recursive DFS

---

## Mutual Recursion

Functions call each other - parser/lexer, alternating game players:

```
function EVEN(n)
    if n = 0 then return true
    else return ODD(n - 1)

function ODD(n)
    if n = 0 then return false
    else return EVEN(n - 1)
```

---

## AIMA Examples

| Algorithm | Recursion type |
|-----------|----------------|
| DFS | Linear on depth |
| Minimax / α-β | Tree on game tree |
| MERGE-SORT | Divide and conquer |
| Backtracking CSP | Backtracking |
| Forward chaining | Can be iterative or recursive |
| Parse tree construction | Tree recursion |

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

## Key Takeaways

1. Every recursive function needs base cases and guaranteed progress toward them.
2. Tree recursion (minimax) and backtracking are central AI patterns.
3. Memoization converts overlapping subproblems to efficient dynamic programming.
4. Python's recursion limit may require iterative reformulation for deep search.
5. Tail recursion can be iteration-equivalent but Python does not optimize it automatically.

## Exercises

1. Implement recursive and memoized Fibonacci; compare runtime for n=35.
2. Write recursive DFS for graph given adjacency list.
3. Implement backtracking for N-queens problem.
4. Convert recursive factorial to iterative version.
5. Trace minimax call tree for tic-tac-toe on partially filled board.

---

**Previous:** [Section B.4 - Data Structures](./section-04-data-structures.md)

**Next:** [Section B.6 - Python Mapping](./section-06-python-mapping.md)