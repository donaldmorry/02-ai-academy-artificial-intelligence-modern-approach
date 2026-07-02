# Section 5.8: Connect Four Adversarial Search Lab

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 5  
> **Previous:** [Section 5.7 - Partial Observability in Games](./section-07-partial-observability-in-games.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Lab Goal

Build a playable Connect Four agent using minimax, alpha-beta pruning, and a heuristic evaluation function.

This lab turns the ideas from the chapter into an engineered game-playing system.

You will not search the full game tree.

Instead, you will combine:

- State representation
- Legal move generation
- Terminal testing
- Depth-limited minimax
- Alpha-beta pruning
- Heuristic evaluation
- Move ordering
- Experiment logging

---

## Deliverables

Submit:

1. A runnable game engine.
2. At least two agents.
3. A match table.
4. A short report.
5. A note describing one search bug you found and fixed.

Your agents should be able to play:

- Human vs agent
- Agent vs random
- Agent vs agent

---

## Rules Summary

Connect Four is played on a 6-row by 7-column grid.

Players alternate dropping pieces into columns.

A piece falls to the lowest available row in that column.

The first player to connect four pieces wins.

Connections may be:

- Horizontal
- Vertical
- Diagonal down-right
- Diagonal up-right

If the board fills with no winner, the game is a draw.

---

## Representation

Use:

```python
ROWS = 6
COLS = 7
EMPTY = 0
MAX_PLAYER = 1
MIN_PLAYER = -1

Board = tuple[tuple[int, ...], ...]
```

The top row should be index `0`.

The bottom row should be index `ROWS - 1`.

This makes printing natural.

It also makes dropping a piece a simple scan upward from the bottom.

---

## Board Helpers

Implement:

```python
def new_board() -> Board:
    return tuple(tuple(EMPTY for _ in range(COLS)) for _ in range(ROWS))


def legal_moves(board: Board) -> list[int]:
    return [c for c in range(COLS) if board[0][c] == EMPTY]


def drop_piece(board: Board, col: int, player: int) -> Board:
    rows = [list(row) for row in board]
    for r in range(ROWS - 1, -1, -1):
        if rows[r][col] == EMPTY:
            rows[r][col] = player
            return tuple(tuple(row) for row in rows)
    raise ValueError(f"Column {col} is full")
```

Test each helper before writing minimax.

---

## Win Detection

Implement:

```python
def winner(board: Board) -> int | None:
    directions = [(0, 1), (1, 0), (1, 1), (-1, 1)]
    for r in range(ROWS):
        for c in range(COLS):
            player = board[r][c]
            if player == EMPTY:
                continue
            for dr, dc in directions:
                cells = []
                for k in range(4):
                    rr = r + dr * k
                    cc = c + dc * k
                    if 0 <= rr < ROWS and 0 <= cc < COLS:
                        cells.append(board[rr][cc])
                if len(cells) == 4 and all(x == player for x in cells):
                    return player
    return None
```

Also implement:

```python
def terminal(board: Board) -> bool:
    return winner(board) is not None or not legal_moves(board)
```

---

## Evaluation Function

The evaluation function estimates board value for `MAX_PLAYER`.

Use a window-based heuristic.

A window is any 4-cell line segment.

Suggested scoring:

- Four MAX pieces: `+100000`
- Three MAX and one empty: `+100`
- Two MAX and two empty: `+10`
- Four MIN pieces: `-100000`
- Three MIN and one empty: `-120`
- Two MIN and two empty: `-10`

The larger penalty for MIN threats makes the agent block.

---

## Window Collection

Implement:

```python
def windows(board: Board):
    for r in range(ROWS):
        for c in range(COLS - 3):
            yield [board[r][c + i] for i in range(4)]

    for r in range(ROWS - 3):
        for c in range(COLS):
            yield [board[r + i][c] for i in range(4)]

    for r in range(ROWS - 3):
        for c in range(COLS - 3):
            yield [board[r + i][c + i] for i in range(4)]

    for r in range(3, ROWS):
        for c in range(COLS - 3):
            yield [board[r - i][c + i] for i in range(4)]
```

Then score each window.

---

## Center Preference

Good Connect Four play values the center column.

Add a small score for occupying the center:

```python
def center_score(board: Board) -> int:
    center = COLS // 2
    return sum(row[center] for row in board) * 3
```

This is not enough to win alone.

It improves move ordering and early play.

---

## Minimax

Implement depth-limited minimax.

Recommended signature:

```python
def minimax(board: Board, depth: int, maximizing: bool) -> tuple[int, int | None]:
    if depth == 0 or terminal(board):
        return evaluate(board), None

    moves = legal_moves(board)
    if maximizing:
        best_value = -10**18
        best_move = None
        for move in moves:
            child = drop_piece(board, move, MAX_PLAYER)
            value, _ = minimax(child, depth - 1, False)
            if value > best_value:
                best_value = value
                best_move = move
        return best_value, best_move

    best_value = 10**18
    best_move = None
    for move in moves:
        child = drop_piece(board, move, MIN_PLAYER)
        value, _ = minimax(child, depth - 1, True)
        if value < best_value:
            best_value = value
            best_move = move
    return best_value, best_move
```

First get this correct.

Then add alpha-beta.

---

## Alpha-Beta Pruning

Add parameters:

- `alpha`
- `beta`

Prune when `alpha >= beta`.

Track how many nodes were expanded.

You may use a mutable counter:

```python
stats = {"nodes": 0, "cutoffs": 0}
```

Report node counts for plain minimax and alpha-beta at the same depth.

This is the clearest evidence that pruning works.

---

## Move Ordering

Search center moves first:

```python
def ordered_moves(board: Board) -> list[int]:
    moves = legal_moves(board)
    center = COLS // 2
    return sorted(moves, key=lambda c: abs(c - center))
```

Run alpha-beta:

1. Without move ordering.
2. With move ordering.

Compare node counts.

Move ordering should improve pruning.

---

## Agents

Implement at least:

1. `RandomAgent`
2. `MinimaxAgent(depth=3)`
3. `AlphaBetaAgent(depth=5)`

Optional:

4. `AlphaBetaAgent(depth=6, ordered=True)`
5. `ThreatAgent` that always blocks immediate wins.

Each agent should expose:

```python
def choose_move(board: Board, player: int) -> int:
    ...
```

---

## Match Protocol

Run:

- AlphaBeta depth 4 vs Random
- AlphaBeta depth 5 vs Random
- AlphaBeta depth 4 vs Minimax depth 3
- AlphaBeta ordered vs AlphaBeta unordered

For each pairing:

1. Play 20 games.
2. Alternate first player.
3. Record wins, losses, draws.
4. Record average nodes per move.
5. Record average seconds per move.

---

## Required Table

| Agent A | Agent B | A wins | B wins | Draws | Avg nodes | Avg seconds |
|---------|---------|--------|--------|-------|-----------|-------------|

Fill at least four rows.

---

## Debugging Checklist

If your agent plays badly:

1. Verify `winner` on hand-made boards.
2. Verify `drop_piece` uses gravity.
3. Verify MIN and MAX signs are not swapped.
4. Verify terminal wins return large values.
5. Verify the agent blocks a one-move opponent win.
6. Verify legal moves exclude full columns.
7. Print the board after every move for one game.
8. Compare depth 1 decisions to direct evaluation.

---

## Analysis Questions

Answer:

1. How much did alpha-beta reduce node count?
2. Did move ordering matter?
3. Which heuristic term mattered most?
4. What horizon effect did you observe?
5. Why is exact minimax infeasible for full Connect Four?
6. How would iterative deepening help under a time limit?
7. What changes if the opponent is not optimal?
8. How does this lab connect to stochastic games?

---

## Extension: Immediate Threats

Add a tactical layer:

1. If you can win immediately, play that move.
2. Else if the opponent can win immediately, block.
3. Else use alpha-beta.

Compare against plain alpha-beta.

This often improves shallow agents.

It is also a practical reminder that search and domain knowledge work well together.

---

## Extension: Transposition Table

Many board positions can be reached through different move orders.

Cache evaluated states:

```python
cache: dict[tuple[Board, int, bool], tuple[int, int | None]] = {}
```

Use the remaining depth and side-to-move as part of the key.

Report cache hit rate.

---

## Reflection

Adversarial search is a negotiation between exact reasoning and time.

Minimax defines optimal play.

Alpha-beta preserves minimax's answer while avoiding irrelevant branches.

Heuristics make depth limits useful.

Move ordering turns the same theory into a much faster program.

That combination is why game search remains one of AI's cleanest engineering laboratories.

---

**Previous:** [Section 5.7 - Partial Observability in Games](./section-07-partial-observability-in-games.md)  
**Next:** [Chapter 06 - Constraint Satisfaction](../chapter-06-constraint-satisfaction/README.md)
