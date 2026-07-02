# Section 5.1: Games and Game Trees

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Sections 5.1-5.2  
> **Previous:** [Chapter 04 - Search in Complex Environments](../chapter-04-search-complex-environments/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Single-Agent Search to Games

[Chapter 03](../chapter-03-solving-problems-by-searching/README.md) had one agent choosing actions. **Games** add an **opponent** whose goals conflict with yours. The environment is **multiagent**, **adversarial**, and typically **turn-based**.

> **Readable form:** You don't just search for a path - you search while assuming the other player will try to stop you.

Humorous analogy: classical search is **solving a maze alone**; game search is **chess in a haunted house** - every corridor you pick, a ghost picks the worst reply.

---

## Game Formulation (AIMA)

A **two-player game** is defined by:

1. **$s_0$** - initial state (board configuration, whose turn)
2. **$\text{PLAYER}(s)$** - MAX or MIN (who moves in $s$)
3. **$\text{ACTIONS}(s)$** - legal moves
4. **$\text{RESULT}(s, a)$** - successor state
5. **$\text{TERMINAL-TEST}(s)$** - is the game over?
6. **$\text{UTILITY}(s, p)$** - numeric payoff for player $p$ at terminal $s$

---

## Two-Player Zero-Sum Games

**Zero-sum:** one player's gain equals the other's loss. Utility for MAX = $U$; for MIN = $-U$.

| Property | Typical chess/checkers |
|----------|------------------------|
| **Players** | 2 |
| **Information** | Perfect (full board visible) |
| **Chance** | None (deterministic) |
| **Payoff** | Win / loss / draw → $+1$, $-1$, $0$ |

---

## Game Trees vs. Search Trees

**Game trees** alternate **MAX nodes** (our move) and **MIN nodes** (opponent's move):

| Node | Who chooses | Objective |
|------|-------------|-----------|
| **MAX** | Us | Maximize utility |
| **MIN** | Opponent | Minimize our utility |

Depth $m$ = number of **plies**. Branching factor $b$ ≈ legal moves per position.

---

## Tic-Tac-Toe vs Chess Scale

| Game | $b$ (approx) | Searchable depth? |
|------|--------------|-------------------|
| Tic-tac-toe | ≤ 9 | Full tree |
| Chess | 35 | Needs heuristics + pruning |
| Go | 250 | MCTS ([Section 5.6](./section-06-monte-carlo-tree-search.md)) |

---

## Utilities and Terminal States

$$
\text{UTILITY}(s) = \begin{cases} +1 & \text{MAX wins} \\ 0 & \text{draw} \\ -1 & \text{MAX loses} \end{cases}
$$
> **Readable form:** MAX wants high utility; MIN wants low (zero-sum).

---

## Game Tree Example (Tiny)

```
        MAX
       /   \
    MIN     MIN
   / \     / \
  3   12   8   2
```

If MIN plays optimally: left MIN picks 3; right MIN picks 2. MAX picks left achieving **3** vs. **2**.

---

## Extended Worked Example: Tree Enumeration

Depth 2, branching 2:

```
                    [MAX s0]
                   /        \
              [MIN]          [MIN]
              /  \           /  \
            5    8          3    9
```

Backup: left MIN → 5, right MIN → 3, MAX → **5**.

---

## PEAS: Chess-Playing Agent

| PEAS | Chess program |
|------|---------------|
| **P** | Win/draw; fast strong moves |
| **E** | Board, rules, clock; opponent |
| **A** | Legal move selection |
| **S** | Board state (perfect info) |

---

## Implementation Notes (aima-python)

| Function | Purpose |
|----------|---------|
| `Game.actions(state)` | Legal moves |
| `Game.to_move(state)` | `'MAX'` or `'MIN'` |
| `Game.utility(state, player)` | Terminal payoff |

**Common mistake:** Forgetting to flip `to_move` in `result`.

---

## Connection to Search Complexity

Game tree size $\approx O(b^m)$. [Chapter 03](../chapter-03-solving-problems-by-searching/section-07-search-analysis.md) complexity applies with alternating min/max layers.

---

## Rational Play Assumption

Game algorithms assume **opponent plays optimally** ([minimax](./section-02-minimax.md)). Real humans blunder - exploitative play differs ([Section 5.8](./section-08-connect-four-adversarial-search-lab.md)).

---

## Comparison: Game Types in Chapter 05

| Type | Chance? | Hidden info? | Algorithm |
|------|---------|--------------|-----------|
| Deterministic perfect info | No | No | Minimax, α-β |
| Stochastic | Yes | No | Expectiminimax |
| Imperfect info | Maybe | Yes | Belief, ISMCTS |

---

## Common Mistakes

**Confusing ply and move.** Check whether depth counts half-moves or full rounds.

**Building full chess tree.** $35^{50}$ is impossible - plan for cutoff.

**Assuming symmetric utilities** outside zero-sum games.

---

## AIMA Connection

Russell & Norvig define games as search where the transition model includes an adversary - utility replaces path cost.

---


---

## Extended Worked Example

Trace the algorithm on a minimal instance by hand before coding. State the invariant maintained at each step (e.g., consistency of partial assignment, backup values at MIN nodes, or domain non-emptiness after propagation).

| Step | State | Action | Notes |
|------|-------|--------|-------|
| 1 | Initial | - | Baseline |
| 2 | After first decision | - | Check constraints |
| 3 | Terminal / goal | - | Verify solution |

> **Readable form:** Hand simulation catches off-by-one errors that unit tests miss.

---

## Implementation Notes (aima-python)

Compare your code to the reference implementation in `aima-python`. Instrument node expansions, backtracks, or inference steps. Log domains after each `revise` or each alpha-beta cutoff.

| Check | Why |
|-------|-----|
| Correct stopping condition | Prevents infinite loops |
| Neighbor / action generation | Common source of bugs |
| Tie-breaking | Affects reproducibility |

---

## Comparison Table

| Aspect | Naive approach | This section's technique |
|--------|----------------|-------------------------|
| Time (worst) | Exponential | Problem-dependent |
| Memory | Varies | See section |
| Completeness | See section | See section |
| Optimality | See section | See section |

---

## AIMA Connection

Re-read the matching section in Russell & Norvig (4th ed.) and complete one end-of-chapter exercise. Connect definitions to the [GLOSSARY.md](../../GLOSSARY.md) entries cited in the section header.

---

## Common Mistakes

- Skipping worked examples and going straight to code.
- Ignoring environment assumptions from PEAS (static vs dynamic, fully vs partially observable).
- Confusing worst-case complexity with typical performance on structured instances.
- Forgetting to restore state on backtrack (CSP domains, game tree depth, belief sets).


## Reflection Questions

1. Define zero-sum game.
2. Role of eval function?
3. Alpha-beta pruning?
4. MCTS vs minimax?
5. PEAS for chess?

---

**Next:** [Section 5.2 - Minimax](./section-02-minimax.md)

---

## References

1. Russell & Norvig AIMA Ch 5
2. aima-python games.py
3. Knuth & Moore alpha-beta



