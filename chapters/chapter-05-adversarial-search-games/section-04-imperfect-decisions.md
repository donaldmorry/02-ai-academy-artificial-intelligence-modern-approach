# Section 5.4: Imperfect Decisions

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 5.4  
> **Previous:** [Section 5.3 - Alpha-Beta Pruning](./section-03-alpha-beta-pruning.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## When the Tree Is Too Deep

Chess has branching $\approx 35$ and games $\gg 50$ plies. [Minimax](./section-02-minimax.md) to terminal states is impossible. **Cutoff** search replaces terminal utilities with an **evaluation function** $\text{EVAL}(s)$ estimating win probability from non-terminal $s$.

> **Readable form:** Stop searching before the game ends and score the position with a smart guess - like judging a chess board without playing to checkmate.

Humorous analogy: evaluation functions are **movie trailers** - you never see the ending, but you guess if the film will be good.

---

## Depth-Limited Minimax

**H-MINIMAX** at $(s, d)$ - heuristic minimax value at state $s$ with depth limit $d$:

$$
\text{H-MINIMAX}(s, d) = \begin{cases}
\text{EVAL}(s) & \text{if } d = 0 \text{ or terminal}(s) \\
\max_a \text{H-MINIMAX}(\text{RESULT}(s,a), d-1) & \text{MAX} \\
\min_a \text{H-MINIMAX}(\text{RESULT}(s,a), d-1) & \text{MIN}
\end{cases}
$$
> **Readable form:** Same as minimax, but at depth 0 use EVAL instead of true utility.

Depth $d=6$ with [alpha-beta](./section-03-alpha-beta-pruning.md) is practical for chess engines under clock constraints.

---

## Designing Evaluation Functions

Good $\text{EVAL}(s)$ should:

| Criterion | Meaning |
|-----------|---------|
| **Correlate with winning** | Higher eval ≈ better for MAX |
| **Be fast** | Called at millions of nodes |
| **Be stable** | Small board changes → small eval changes |
| **Encode expertise** | Material, mobility, king safety |

**Chess example (simplified):**

$$
\text{EVAL}(s) = w_p P + w_n N + w_b B + w_r R + w_q Q + \text{position bonuses}
$$
> **Readable form:** Weighted sum of piece counts plus positional terms.

Weights tuned from **self-play** or **grandmaster games** ([Chapter 22](../chapter-22-reinforcement-learning/README.md)).

---

## Features and Linear Evaluators

Like [Course 1](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/README.md) linear models:

$$
\text{EVAL}(s) = \sum_i w_i f_i(s)
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

Features $f_i(s)$: piece counts, pawn structure, attack on king. Weights $w_i$ learned or hand-tuned.

**Nonlinear** evaluators (neural nets) power AlphaZero - still used at **leaf** nodes with MCTS ([Section 5.6](./section-06-monte-carlo-tree-search.md)).

---

## Quiescence Search

**Problem:** cutoff mid-capture leaves eval unstable - queen "hanging" looks fine until capture resolved.

**Quiescence search:** at depth 0, if position **tactical** (captures, checks), continue searching **capture moves only** until **quiet** position.

| Position type | Action at cutoff |
|---------------|------------------|
| **Quiet** | Apply EVAL |
| **Tactical** | Extend search (limited) |

Reduces **horizon effect** partially - not eliminated entirely.

```python
def quiescence(state, alpha, beta):
  stand_pat = EVAL(state)
  if stand_pat >= beta:
    return beta
  if stand_pat > alpha:
    alpha = stand_pat
  for move in capture_moves(state):
    score = -quiescence(result(state, move), -beta, -alpha)
    if score >= beta:
      return beta
    if score > alpha:
      alpha = score
  return alpha
```

---

## Horizon Effect

**Horizon effect:** serious blunder lies **just beyond** search horizon - delayed tactic invisible to fixed-depth search.

**Mitigations:**

- **Deeper search** (alpha-beta, MCTS)
- **Quiescence** extensions
- **Singular extensions** - force deep line when one move dominates
- **Endgame tablebases** - exact utilities for ≤ 6 pieces

---

## Nonterminal Errors

AIMA Figure 5.16: heuristic minimax may prefer wrong root move because EVAL misorders **successor** values - greedy eval at cutoff ≠ backed-up minimax truth.

**Principle:** evaluation must predict **outcome after optimal play from $s$**, not just static appearance.

---

## Cutting Off vs. Terminal

| | Terminal | Cutoff |
|--|----------|--------|
| **Utility source** | True game outcome | EVAL |
| **Correctness** | Exact | Approximate |
| **Depth** | Full game | Fixed $d$ |

**Iterative deepening:** increase $d$ until time expires - any-time improvement.

---

## Connect Four Evaluation Sketch

Features for MAX (red):

- Count threats of 3-in-a-row with open ends
- Center column occupancy
- Blocks opponent threats

$$
\text{EVAL}(s) = 3 \cdot (\text{my\_threes}) - 3 \cdot (\text{opp\_threes}) + 1 \cdot (\text{center\_control})
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

Chapter lab uses this style eval with depth-limited alpha-beta.

---

## PEAS: Depth-Limited Chess Engine

| PEAS | Component |
|------|-----------|
| **P** | Strong moves under clock |
| **E** | Board, rules, opponent |
| **A** | Move from H-MINIMAX |
| **S** | Board |

**Performance** trades depth vs. eval quality - [rational agent](../../GLOSSARY.md#rational-agent) under time pressure.

---

## Worked Example: Material-Only Eval

MAX (white) up a pawn: $\text{EVAL} = +1$. MIN up a rook: $\text{EVAL} = -5$. Depth-2 search might miss mate in 3 that sacrifices rook - horizon effect. Quiescence continues capture lines - may see mate threat.

---

## Connection to Heuristics in Chapter 03

[Admissible heuristics](../../GLOSSARY.md#admissible-heuristic) guarantee A* optimality. Game evaluators need **accuracy in sign and magnitude** for backed-up decisions - correlation with win probability, not admissibility.

---

## Singular Extensions and Tablebases

**Singular extension:** when one move is clearly best (score far above siblings), extend search one ply on that line only - catches tactics without uniform depth increase.

**Endgame tablebases:** precomputed exact utilities for all positions with ≤ $k$ pieces. Chess engines consult tablebases at leaves - perfect play in endgame, heuristic eval in midgame.

---

## Iterative Deepening for Games

Same pattern as [Chapter 03](../chapter-03-solving-problems-by-searching/section-03-uninformed-search.md): search depth 1, then 2, … until clock expires. Return best move from deepest completed iteration - **anytime** algorithm under tournament time controls.

---

## Common Mistakes

- Using same eval for opening and endgame without phase-specific terms.
- Ignoring mobility - cramped positions look equal on material alone.
- Fixed depth without iterative deepening under time pressure.

---

## Reflection Questions

1. Why can't chess programs search to terminal states?
2. What is the horizon effect?
3. How does quiescence search differ from fixed-depth cutoff?
4. List three features you would include in a chess evaluation function.
5. When might a high EVAL at cutoff still lead to a losing line?

---

**Next:** [Section 5.5 - Stochastic Games](./section-05-stochastic-games.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 5.4. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - evaluation functions in `games.py`. [aima-python](https://github.com/aimacode/aima-python)
3. Berliner, H. - evaluation in chess machines.
4. Silver, D., et al. - neural evaluation in AlphaGo.
5. MIT 6.034 - Game evaluation. [OCW](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)
