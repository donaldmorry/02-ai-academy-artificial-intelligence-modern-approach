# Section 5.2: Minimax

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 5.2  
> **Previous:** [Section 5.1 - Games and Game Trees](./section-01-games-and-game-trees.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Optimal Play Against a Rational Opponent

[Section 5.1](./section-01-games-and-game-trees.md) defined game trees with MAX and MIN nodes. [Minimax](../../GLOSSARY.md#minimax) computes the **best achievable utility** for MAX assuming MIN plays optimally - the foundation of game-playing AI.

> **Readable form:** MAX maximizes the score MIN will force upon him - MIN always picks the child MIN values.

---

## Minimax Value Recurrence

For state $s$:

$$
\text{MINIMAX}(s) = \begin{cases}
\text{UTILITY}(s) & \text{if terminal} \\
\max_{a \in \text{ACTIONS}(s)} \text{MINIMAX}(\text{RESULT}(s,a)) & \text{if MAX to move} \\
\min_{a \in \text{ACTIONS}(s)} \text{MINIMAX}(\text{RESULT}(s,a)) & \text{if MIN to move}
\end{cases}
$$
> **Readable form:** Terminal = utility; MAX's turn = best child value; MIN's turn = worst child value for MAX.

The **minimax decision** at the root is the action whose successor has highest minimax value.

---

## Backup on a Small Tree

AIMA Figure 5.2: terminal utilities $\{3,12,8,2,4,6,14,5,2\}$.

| Node | Type | Children values | MINIMAX |
|------|------|-----------------|---------|
| B | MIN | 3, 12, 8 | 3 |
| C | MIN | 2, 4, 6 | 2 |
| D | MIN | 14, 5, 2 | 2 |
| Root | MAX | 3, 2, 2 | 3 → action $a_1$ |

MIN at B picks 3 (not 12). MAX at root picks $a_1$.

---

## Minimax Search Algorithm (Figure 5.3)

```python
def minimax_search(game, state):
    """Return (utility, best_move) for MAX at state."""
    player = game.to_move(state)
    if game.is_terminal(state):
        return game.utility(state, player), None
    if player == 'MAX':
        best_val, best_move = float('-inf'), None
        for action in game.actions(state):
            val, _ = minimax_search(game, game.result(state, action))
            if val > best_val:
                best_val, best_move = val, action
        return best_val, best_move
    else:
        best_val, best_move = float('inf'), None
        for action in game.actions(state):
            val, _ = minimax_search(game, game.result(state, action))
            if val < best_val:
                best_val, best_move = val, action
        return best_val, best_move
```

Recursive depth-first exploration to leaves, backing up values.

---

## Separating MAX-VALUE and MIN-VALUE

AIMA uses mutual recursion:

```python
def max_value(game, state):
    if game.is_terminal(state):
        return game.utility(state, 'MAX'), None
    v, best = float('-inf'), None
    for a in game.actions(state):
        v2, _ = min_value(game, game.result(state, a))
        if v2 > v:
            v, best = v2, a
    return v, best

def min_value(game, state):
    if game.is_terminal(state):
        return game.utility(state, 'MAX'), None
    v, best = float('inf'), None
    for a in game.actions(state):
        v2, _ = max_value(game, game.result(state, a))
        if v2 < v:
            v, best = v2, a
    return v, best
```

---

## Complexity

For branching factor $b$ and depth $m$ plies:

$$
\text{Time} = O(b^m), \quad \text{Space} = O(bm) \text{ or } O(m)
$$
> **Readable form:** Same exponential blow-up as DFS - minimax alone cannot solve chess.

Chess: $35^{80} \approx 10^{123}$ - minimax is a **mathematical ideal**, not a practical algorithm without enhancements.

---

## Playing Against Suboptimal Opponents

Minimax assumes **optimal MIN**. Against weaker opponents, minimax is **safe** (MAX never does worse than minimax value) but may miss **exploitative** lines that punish mistakes.

AIMA example: risky move with 9/10 traps for MIN - exploitable if MIN is weak; minimax prefers guaranteed draw.

| Strategy | Assumption | Risk |
|----------|------------|------|
| Minimax | Optimal opponent | May miss traps |
| Exploitative | Model opponent errors | Punished if opponent is strong |

---

## Tic-Tac-Toe: Solved by Minimax

Full minimax on tic-tac-toe proves **optimal play always draws**. First player cannot force win against perfect defense.

```python
def tic_tac_toe_minimax(board, player):
    u = utility(board)
    if u is not None:
        return u, None
    if player == 'X':
        best = (-2, None)
        for a in tic_tac_toe_actions(board):
            val, _ = tic_tac_toe_minimax(result(board, a, 'X'), 'O')
            if val > best[0]:
                best = (val, a)
        return best
    else:
        best = (2, None)
        for a in tic_tac_toe_actions(board):
            val, _ = tic_tac_toe_minimax(result(board, a, 'O'), 'X')
            if val < best[0]:
                best = (val, a)
        return best
```

---

## Game Graphs and Transpositions

In chess, different move orders reach the same position - **transpositions**. Game **graph** search with transposition table reuses minimax values:

$$
\text{MINIMAX}(s) \text{ computed once per distinct } s
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

Reduces redundant subtrees - critical for practical engines.

---

## Connection to AND-OR Search

Winning strategy in win/lose games = AND-OR solution: MAX must win against **all** MIN responses. Minimax generalizes to **numeric utilities** - graded outcomes (material advantage, margin of victory).

---

## PEAS: Optimal Tic-Tac-Toe Agent

| PEAS | Perfect play agent |
|------|-------------------|
| **P** | Force win or draw |
| **E** | 3×3 board |
| **A** | Minimax move |
| **S** | Full board |

---

## Limitations Motivating Next Sections

| Issue | Remedy |
|-------|--------|
| Exponential time | [Alpha-beta pruning](./section-03-alpha-beta-pruning.md) |
| Cannot reach leaves | [Evaluation + cutoff](./section-04-imperfect-decisions.md) |
| Chance moves | [Expectiminimax](./section-05-stochastic-games.md) |
| Huge branching | [MCTS](./section-06-monte-carlo-tree-search.md) |

---

## Worked Trace: 2-Ply Game

MAX at root, two actions. Left → MIN chooses min of $\{10, 5\}$ = 5. Right → MIN chooses min of $\{3, 8\}$ = 3. MAX picks left (5 > 3).

---

## Reflection Questions

1. Why does MIN choose the minimum child utility at MIN nodes?
2. Is minimax optimal against a random opponent? Discuss.
3. What is the time complexity in terms of $b$ and $m$?
4. How do transposition tables change the search graph?
5. Why is tic-tac-toe tractable but chess is not?

---

**Next:** [Section 5.3 - Alpha-Beta Pruning](./section-03-alpha-beta-pruning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 5.2. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Shannon, C. E. (1950). Programming a computer for playing chess.
3. Russell, S., & Norvig, P. - `games.py` minimax. [aima-python](https://github.com/aimacode/aima-python)
4. Knuth, D. E., & Moore, R. W. (1975). An analysis of alpha-beta pruning.
5. MIT 6.034 - Minimax lecture. [OCW](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)



