# Section 6.8: Sudoku CSP Lab

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 6  
> **Previous:** [Section 6.7 - CSP Applications](./section-07-csp-applications.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Lab Goal

Build a Sudoku solver as a constraint satisfaction problem.

This lab is a compact version of a real CSP workflow:

1. Define variables.
2. Define domains.
3. Define constraints.
4. Apply propagation.
5. Search with backtracking.
6. Add variable and value heuristics.
7. Compare solver variants.

You will finish with a solver that can handle standard 9 by 9 puzzles.

---

## Deliverables

Submit:

1. A runnable solver.
2. At least five solved puzzles.
3. A table comparing solver variants.
4. A short explanation of the hardest puzzle.
5. A note about one propagation or search bug.

Your code should print the solved grid clearly.

---

## Sudoku as a CSP

Standard Sudoku has:

- 81 variables.
- Each variable is a cell.
- Each domain is `{1, 2, 3, 4, 5, 6, 7, 8, 9}` unless the puzzle gives a fixed digit.
- Row constraints require all digits in each row to differ.
- Column constraints require all digits in each column to differ.
- Box constraints require all digits in each 3 by 3 box to differ.

A solution is a complete assignment satisfying all constraints.

---

## Coordinates

Use row and column coordinates:

```python
Cell = tuple[int, int]
ROWS = range(9)
COLS = range(9)
DIGITS = set(range(1, 10))
CELLS = [(r, c) for r in ROWS for c in COLS]
```

Represent domains:

```python
Domains = dict[Cell, set[int]]
```

---

## Parsing

Represent a puzzle as an 81-character string.

Use digits for givens.

Use `.` or `0` for blanks.

Example:

```python
PUZZLE = (
    "53..7...."
    "6..195..."
    ".98....6."
    "8...6...3"
    "4..8.3..1"
    "7...2...6"
    ".6....28."
    "...419..5"
    "....8..79"
)
```

Implement:

```python
def parse(puzzle: str) -> Domains:
    assert len(puzzle) == 81
    domains = {}
    for cell, ch in zip(CELLS, puzzle):
        if ch in ".0":
            domains[cell] = set(DIGITS)
        else:
            domains[cell] = {int(ch)}
    return domains
```

---

## Units and Peers

A unit is a row, column, or box.

A peer of a cell is any other cell in the same row, column, or box.

Build:

```python
def box_index(r: int, c: int) -> tuple[int, int]:
    return r // 3, c // 3


def peers(cell: Cell) -> set[Cell]:
    r, c = cell
    result = set()
    result.update((r, cc) for cc in COLS if cc != c)
    result.update((rr, c) for rr in ROWS if rr != r)
    br, bc = box_index(r, c)
    for rr in range(br * 3, br * 3 + 3):
        for cc in range(bc * 3, bc * 3 + 3):
            if (rr, cc) != cell:
                result.add((rr, cc))
    return result
```

Cache peers for speed.

---

## Consistency Check

Implement:

```python
def is_consistent(domains: Domains) -> bool:
    for cell in CELLS:
        if not domains[cell]:
            return False
        if len(domains[cell]) == 1:
            value = next(iter(domains[cell]))
            for peer in PEERS[cell]:
                if domains[peer] == {value}:
                    return False
    return True
```

This catches contradictions early.

---

## Assignment Test

Implement:

```python
def solved(domains: Domains) -> bool:
    return all(len(domains[cell]) == 1 for cell in CELLS) and is_consistent(domains)
```

Then test it on a known solved grid.

---

## Forward Checking

When a cell is assigned a value, remove that value from its peers.

Implement:

```python
def assign(domains: Domains, cell: Cell, value: int) -> Domains | None:
    new_domains = {c: set(vs) for c, vs in domains.items()}
    new_domains[cell] = {value}
    for peer in PEERS[cell]:
        if value in new_domains[peer]:
            new_domains[peer].remove(value)
            if not new_domains[peer]:
                return None
    return new_domains
```

This is enough to solve easy puzzles with backtracking.

---

## AC-3 Propagation

AC-3 enforces arc consistency.

For Sudoku, if a cell has singleton domain `{v}`, then every peer must remove `v`.

Implement:

```python
from collections import deque


def ac3(domains: Domains) -> Domains | None:
    domains = {c: set(vs) for c, vs in domains.items()}
    queue = deque((a, b) for a in CELLS for b in PEERS[a])
    while queue:
        xi, xj = queue.popleft()
        if revise(domains, xi, xj):
            if not domains[xi]:
                return None
            for xk in PEERS[xi] - {xj}:
                queue.append((xk, xi))
    return domains
```

You write `revise`.

For inequality constraints, `revise(xi, xj)` removes a value from `xi` only when `xj` is forced to the same value.

---

## MRV Heuristic

Minimum remaining values chooses the unassigned variable with the smallest domain.

Implement:

```python
def select_unassigned_variable(domains: Domains) -> Cell:
    choices = [cell for cell in CELLS if len(domains[cell]) > 1]
    return min(choices, key=lambda cell: len(domains[cell]))
```

MRV fails fast.

It often turns Sudoku from brute force into a short search.

---

## LCV Heuristic

Least constraining value tries values that eliminate the fewest peer options first.

Implement:

```python
def order_values(domains: Domains, cell: Cell) -> list[int]:
    def eliminated(value: int) -> int:
        return sum(value in domains[p] for p in PEERS[cell])
    return sorted(domains[cell], key=eliminated)
```

LCV usually helps less than MRV, but it is worth measuring.

---

## Backtracking Solver

Implement:

```python
def backtrack(domains: Domains, stats: dict[str, int]) -> Domains | None:
    stats["nodes"] += 1
    domains = ac3(domains)
    if domains is None:
        stats["failures"] += 1
        return None
    if solved(domains):
        return domains

    cell = select_unassigned_variable(domains)
    for value in order_values(domains, cell):
        child = assign(domains, cell, value)
        if child is None:
            continue
        result = backtrack(child, stats)
        if result is not None:
            return result
    stats["failures"] += 1
    return None
```

Keep the stats.

You will need them for the experiment table.

---

## Printing

Implement a readable board printer:

```python
def render(domains: Domains) -> str:
    rows = []
    for r in ROWS:
        parts = []
        for c in COLS:
            values = domains[(r, c)]
            parts.append(str(next(iter(values))) if len(values) == 1 else ".")
        rows.append("".join(parts))
    return "\n".join(rows)
```

Add separators if you like.

Do not make debugging dependent on a GUI.

---

## Experiment Variants

Compare:

1. Plain backtracking.
2. Backtracking plus forward checking.
3. Backtracking plus AC-3.
4. Backtracking plus AC-3 plus MRV.
5. Backtracking plus AC-3 plus MRV plus LCV.

If your implementation combines some variants, explain that.

The point is to measure the value of propagation and heuristics.

---

## Required Table

| Puzzle | Variant | Solved | Nodes | Failures | Seconds |
|--------|---------|--------|-------|----------|---------|

Use at least five puzzles.

Include one easy, one medium, and one hard puzzle.

---

## Analysis Questions

Answer:

1. Which component reduced search the most?
2. Did AC-3 alone solve any puzzle?
3. Did LCV help consistently?
4. What kind of puzzle caused the most backtracking?
5. How did MRV change the search tree?
6. Why is Sudoku naturally a CSP?
7. What would change for KenKen or map coloring?
8. Where would local search be weaker than CSP search here?

---

## Debugging Checklist

If the solver gives an invalid result:

1. Verify peer counts. Each cell should have 20 peers.
2. Verify boxes use integer division by 3.
3. Verify givens are never removed.
4. Verify copied domains do not alias old sets.
5. Verify `revise` handles only singleton neighbor domains.
6. Verify `solved` also checks consistency.
7. Verify failed branches return `None`.
8. Verify stats do not change solver behavior.

---

## Extension: Naked Singles

AC-3 handles many singleton implications.

You can also scan each unit:

- If a digit can go in only one cell in a row, assign it.
- If a digit can go in only one cell in a column, assign it.
- If a digit can go in only one cell in a box, assign it.

This is a common human solving technique.

Add it as a propagation pass and measure node reduction.

---

## Extension: Puzzle Generator

Generate a completed grid by solving from an empty board.

Then remove clues one at a time.

Keep removals only when the puzzle still has a unique solution.

This extension requires counting solutions, not just finding one.

---

## Reflection

CSPs are powerful because they separate problem structure from search control.

Sudoku has no path cost and no adversary.

The challenge is consistency.

Propagation shrinks domains before search spends effort.

MRV turns the tightest constraint into the next decision.

That pattern reappears in scheduling, configuration, planning, and resource allocation.

---

**Previous:** [Section 6.7 - CSP Applications](./section-07-csp-applications.md)  
**Next:** [Chapter 07 - Logical Agents](../chapter-07-logical-agents/README.md)
