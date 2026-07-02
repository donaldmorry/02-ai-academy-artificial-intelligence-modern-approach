# Section 4.8: N-Queens Local Search Lab

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 4  
> **Previous:** [Section 4.7 - Summary and Tradeoffs](./section-07-summary-and-tradeoffs.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Lab Goal

Build and compare local-search solvers for the n-queens problem.

The point is not only to solve n-queens.

The point is to learn how search behavior changes when a problem is represented as a complete state with an objective function instead of as a path from an initial state.

In this lab, you will:

1. Represent a board compactly.
2. Count queen conflicts efficiently.
3. Implement steepest-ascent hill climbing.
4. Add random restarts.
5. Implement simulated annealing.
6. Compare success rate, runtime, and search trace quality.
7. Explain which algorithm you would deploy for large boards.

---

## Deliverables

Submit:

1. A runnable Python file or notebook.
2. A short experiment report.
3. At least two plots.
4. A table comparing algorithms.
5. A paragraph explaining one failure mode.
6. A paragraph connecting this lab to online or nondeterministic search.

Keep the report concise.

Evidence matters more than polish.

---

## Problem Setup

The n-queens problem asks you to place n queens on an n by n chessboard so that no two queens attack each other.

Queens attack along:

- Rows
- Columns
- Diagonals from upper left to lower right
- Diagonals from upper right to lower left

Use a complete-state representation:

- One queen per column.
- The value at index `c` is the row of the queen in column `c`.
- Example: `[1, 3, 0, 2]` is a 4-queen solution.

With one queen per column, column conflicts are impossible.

Row and diagonal conflicts remain.

---

## Objective Function

Use the number of attacking queen pairs as the cost.

A solution has cost `0`.

Hill climbing should minimize cost.

Simulated annealing can sometimes accept a worse move to escape local minima.

Implementation hint:

- Two queens conflict by row if their row values match.
- Two queens conflict by diagonal if `abs(row_i - row_j) == abs(col_i - col_j)`.

---

## Baseline Data Structure

Start with this representation:

```python
from dataclasses import dataclass
from random import randrange, choice, random
from time import perf_counter


Board = tuple[int, ...]


@dataclass
class RunResult:
    algorithm: str
    n: int
    solved: bool
    steps: int
    restarts: int
    final_conflicts: int
    seconds: float
    trace: list[int]
```

You may adapt it, but keep the same fields in your final table.

---

## Conflict Counter

Implement:

```python
def conflicts(board: Board) -> int:
    total = 0
    n = len(board)
    for i in range(n):
        for j in range(i + 1, n):
            same_row = board[i] == board[j]
            same_diag = abs(board[i] - board[j]) == abs(i - j)
            if same_row or same_diag:
                total += 1
    return total
```

Then test:

```python
assert conflicts((1, 3, 0, 2)) == 0
assert conflicts((0, 0, 0, 0)) > 0
```

Do not continue until these pass.

---

## Neighbor Generator

A neighbor changes one queen to a different row in the same column.

Implement:

```python
def neighbors(board: Board):
    n = len(board)
    for col in range(n):
        current = board[col]
        for row in range(n):
            if row == current:
                continue
            candidate = list(board)
            candidate[col] = row
            yield tuple(candidate)
```

For n queens, this gives `n * (n - 1)` neighbors.

That is fine for `n <= 100`.

For larger n, you may sample neighbors.

---

## Random Board

Implement:

```python
def random_board(n: int) -> Board:
    return tuple(randrange(n) for _ in range(n))
```

Run:

```python
for n in [4, 8, 20]:
    b = random_board(n)
    assert len(b) == n
    assert all(0 <= r < n for r in b)
```

---

## Hill Climbing

Implement steepest-ascent hill climbing as cost minimization.

At each step:

1. Compute the current cost.
2. Generate all neighbors.
3. Find the neighbor with the lowest cost.
4. Move only if the neighbor improves cost.
5. Stop when cost is `0` or no neighbor improves.

Recommended signature:

```python
def hill_climb_once(n: int, max_steps: int = 10_000) -> RunResult:
    start = perf_counter()
    board = random_board(n)
    trace = [conflicts(board)]

    for step in range(1, max_steps + 1):
        current_cost = trace[-1]
        if current_cost == 0:
            break
        best = min(neighbors(board), key=conflicts)
        best_cost = conflicts(best)
        if best_cost >= current_cost:
            break
        board = best
        trace.append(best_cost)

    final = conflicts(board)
    return RunResult(
        algorithm="hill_climb_once",
        n=n,
        solved=final == 0,
        steps=len(trace) - 1,
        restarts=0,
        final_conflicts=final,
        seconds=perf_counter() - start,
        trace=trace,
    )
```

After implementing it, run it 20 times for `n=8`.

You should see some successes and some local-minimum failures.

---

## Random Restarts

Hill climbing is incomplete because it can stop at a local minimum.

Random restart tries again from a new random board.

Implement:

```python
def hill_climb_restarts(n: int, max_restarts: int = 100) -> RunResult:
    start = perf_counter()
    total_steps = 0
    full_trace = []

    for restart in range(max_restarts + 1):
        result = hill_climb_once(n)
        total_steps += result.steps
        full_trace.extend(result.trace)
        if result.solved:
            return RunResult(
                algorithm="hill_climb_restarts",
                n=n,
                solved=True,
                steps=total_steps,
                restarts=restart,
                final_conflicts=0,
                seconds=perf_counter() - start,
                trace=full_trace,
            )

    return RunResult(
        algorithm="hill_climb_restarts",
        n=n,
        solved=False,
        steps=total_steps,
        restarts=max_restarts,
        final_conflicts=full_trace[-1] if full_trace else -1,
        seconds=perf_counter() - start,
        trace=full_trace,
    )
```

You may improve this implementation by passing seeds and avoiding duplicate conflict calls.

---

## Simulated Annealing

Simulated annealing uses randomness to escape local minima.

At step `t`:

- Pick a random neighbor.
- If it improves cost, accept it.
- If it is worse, accept it with a probability controlled by temperature.
- Lower the temperature over time.

Recommended temperature schedule:

```python
def temperature(t: int, start: float = 10.0, decay: float = 0.995) -> float:
    return start * (decay ** t)
```

Acceptance logic:

```python
from math import exp


def accept(delta: int, temp: float) -> bool:
    if delta < 0:
        return True
    if temp <= 1e-9:
        return False
    return random() < exp(-delta / temp)
```

Here `delta = next_cost - current_cost`.

Negative delta means improvement.

---

## Annealing Solver

Implement:

```python
def simulated_annealing(n: int, max_steps: int = 50_000) -> RunResult:
    start = perf_counter()
    board = random_board(n)
    trace = [conflicts(board)]

    for step in range(1, max_steps + 1):
        current = trace[-1]
        if current == 0:
            break
        temp = temperature(step)
        candidate = choice(list(neighbors(board)))
        candidate_cost = conflicts(candidate)
        if accept(candidate_cost - current, temp):
            board = candidate
            trace.append(candidate_cost)
        else:
            trace.append(current)

    final = conflicts(board)
    return RunResult(
        algorithm="simulated_annealing",
        n=n,
        solved=final == 0,
        steps=len(trace) - 1,
        restarts=0,
        final_conflicts=final,
        seconds=perf_counter() - start,
        trace=trace,
    )
```

For large n, replace `choice(list(neighbors(board)))` with a direct random one-column move.

---

## Experiment Protocol

Run all algorithms on:

- `n = 8`
- `n = 20`
- `n = 50`

For each n and algorithm:

1. Run 30 trials.
2. Record success rate.
3. Record median steps.
4. Record median runtime.
5. Record median final conflicts for failed runs.

Use a fixed random seed for repeatability.

Report hardware only briefly.

---

## Required Table

Create a table with columns:

- Algorithm
- n
- Trials
- Success rate
- Median steps
- Median seconds
- Median final conflicts
- Notes

Fill one row per algorithm and board size.

---

## Required Plots

Plot 1:

- Conflict count over steps for one successful run.

Plot 2:

- Success rate by n for all algorithms.

Optional plot:

- Runtime by n.

---

## Analysis Prompts

Answer these in the report:

1. Which algorithm solves `n=8` most reliably?
2. Which algorithm scales best to `n=50`?
3. Does hill climbing fail because the board is "almost solved" or because no single move improves it?
4. How does the annealing temperature schedule affect exploration?
5. What did random restarts buy you?
6. How would your design change if evaluating conflicts were expensive?
7. Why is this not a shortest-path problem?
8. What does this lab teach about optimization landscapes?

---

## Efficiency Extension

The pairwise conflict counter is simple but costs quadratic time.

Improve it with counters:

- Row counter: `rows[row]`
- Main diagonal counter: `diag1[row - col]`
- Anti diagonal counter: `diag2[row + col]`

The number of attacking pairs in a bucket of size `k` is `k * (k - 1) // 2`.

Use this to compute conflicts in linear time.

Then rerun the experiment for:

- `n = 100`
- `n = 500`

Explain what changed.

---

## Grading Rubric

| Area | Full credit |
|------|-------------|
| Representation | One queen per column; valid boards always maintained |
| Objective | Conflict count is correct and tested |
| Hill climbing | Stops correctly at local minima and solutions |
| Restarts | Aggregates trials and reports restarts clearly |
| Annealing | Implements temperature and probabilistic acceptance |
| Experiments | Uses repeated trials and reports uncertainty honestly |
| Analysis | Explains failure modes with evidence |
| Code quality | Readable functions, no hidden global state |

---

## Reflection

Local search is attractive because it uses little memory.

It can handle large state spaces that would make BFS or A* useless.

But it gives up path guarantees.

You get a good final state, not a proof that the path is optimal.

That tradeoff appears again in neural-network training, planning under uncertainty, and stochastic optimization.

---

**Previous:** [Section 4.7 - Summary and Tradeoffs](./section-07-summary-and-tradeoffs.md)  
**Next:** [Chapter 05 - Adversarial Search and Games](../chapter-05-adversarial-search-games/README.md)
