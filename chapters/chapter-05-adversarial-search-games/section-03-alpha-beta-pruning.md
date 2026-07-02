# Section 5.3: Alpha-Beta Pruning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 5.3  
> **Previous:** [Section 5.2 - Minimax](./section-02-minimax.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Pruning Without Changing Minimax Values

[Minimax](./section-02-minimax.md) explores the entire game tree. [Alpha-beta pruning](../../GLOSSARY.md#alpha-beta-pruning) eliminates subtrees that **cannot affect** the root decision - same minimax move, far fewer nodes.

> **Readable form:** Track best-so-far for MAX ($\alpha$) and MIN ($\beta$); stop when a branch is provably worse than an alternative already examined.

Named by Knuth and Moore (1975); essential for practical game programs since the 1960s.

---

## Alpha and Beta Bounds

At a MAX node:

- $\alpha$ = best (highest) value MAX can guarantee so far along path
- $\beta$ = best (lowest) value MIN can force so far

**Prune** when $\alpha \geq \beta$: MIN already has a refutation elsewhere; this branch won't be chosen.

| Parameter | Meaning | Updated at |
|-----------|---------|------------|
| $\alpha$ | MAX's lower bound on true value | MAX nodes |
| $\beta$ | MIN's upper bound on true value | MIN nodes |

---

## Pruning Condition

During MIN node exploration at child with value $v$:

If $v \leq \alpha$, prune remaining MIN children - MAX won't choose parent.

During MAX node exploration:

If $v \geq \beta$, prune remaining MAX children - MIN won't allow parent.

```python
def alphabeta(game, state, alpha, beta):
    if game.is_terminal(state):
        return game.utility(state, 'MAX'), None
    player = game.to_move(state)
    if player == 'MAX':
        v, move = float('-inf'), None
        for a in game.actions(state):
            v2, _ = alphabeta(game, game.result(state, a), alpha, beta)
            if v2 > v:
                v, move = v2, a
            alpha = max(alpha, v)
            if alpha >= beta:
                break  # prune
        return v, move
    else:
        v, move = float('inf'), None
        for a in game.actions(state):
            v2, _ = alphabeta(game, game.result(state, a), alpha, beta)
            if v2 < v:
                v, move = v2, a
            beta = min(beta, v)
            if alpha >= beta:
                break
        return v, move
```

---

## Correctness Intuition

MIN considers move $b_1$ giving MAX at most 3. Later MIN branch could offer MAX 12 - but MAX already has line guaranteeing 5 elsewhere. MIN never reaches 12 backup - **pruned**.

**Key theorem:** Alpha-beta returns **identical minimax values** at the root (with optimal move ordering, explores $O(b^{m/2})$ nodes vs. $O(b^m)$).

---

## Move Ordering

Pruning efficiency depends on trying **good moves first**:

| Heuristic ordering | Effect |
|--------------------|--------|
| Capture moves first | Chess engines |
| Killer moves | Historically good at depth |
| MVV-LVA | Most valuable victim, least valuable attacker |

**Perfect ordering:** $O(b^{m/2})$ - effective branching factor $\sqrt{b}$.

For chess $b \approx 35$, $\sqrt{35} \approx 6$ - difference between impossible and deep search.

---

## Effective Branching Factor

Measure nodes expanded $N$, depth $d$, solve $b_{\text{eff}}^d \approx N$.

| Algorithm | Typical nodes (good order) |
|-----------|---------------------------|
| Minimax | $O(b^d)$ |
| Alpha-beta | $O(b^{d/2})$ |

Lab ([Section 5.8](./section-08-connect-four-adversarial-search-lab.md)): compare node counts on Connect Four.

---

## Depth-Limited Alpha-Beta

Combine with cutoff depth $d$ and evaluation function $eval(s)$ at leaves ([Section 5.4](./section-04-imperfect-decisions.md)):

```python
def alphabeta_cutoff(game, state, alpha, beta, depth):
    if depth == 0 or game.is_terminal(state):
        return game.eval(state), None
    # ... same as alphabeta with depth-1 recursion
```

Evaluation replaces terminal utility at horizon - introduces **imperfection**.

---

## Transposition Tables

Cache $(state, depth, \alpha, \beta) \mapsto value$ for graph game states. Must store sufficient bound type (exact, lower, upper) for correctness with pruning.

---

## Fail-Soft vs Fail-Hard

| Style | On cache hit |
|-------|--------------|
| Fail-soft | Return cached value if within window |
| Fail-hard | Re-search if bounds incompatible |

Modern engines use sophisticated TT schemes.

---

## PEAS: Chess Engine Search Core

| PEAS | Alpha-beta chess |
|------|------------------|
| **P** | Maximize eval at horizon |
| **E** | Board, clock |
| **A** | Alpha-beta move |
| **S** | Full board |

Iterative deepening + alpha-beta + ordering = standard architecture.

---

## Comparison Table

| Feature | Minimax | Alpha-beta |
|---------|---------|------------|
| Root value | Same | Same |
| Optimal move | Same | Same |
| Nodes expanded | All | Subset |
| Time (worst case) | $O(b^m)$ | Still $O(b^m)$ |
| Time (good order) | $O(b^m)$ | $O(b^{m/2})$ |

Worst-case ordering (weakest first) - no pruning benefit.

---

## Worked Example

MAX root, children A and B. Explore A first: MIN replies yield backed-up 3. $\alpha = 3$. Explore B: first MIN child gives 2. At MIN node, $2 \leq \alpha$? Not yet - continue. If MIN's first reply to B is 1 and $\alpha=3$, remaining B children pruned if MIN already found $\leq 1$ and MAX won't prefer B over A... Trace carefully per AIMA Figure 5.5.

---

## Connection to Branch and Bound

Alpha-beta is **branch-and-bound** on game trees - same logic as OR-tree search with bounds in optimization.

---

## Extended Worked Example: Step-by-Step Pruning

Tree from AIMA Figure 5.5 (simplified):

```
           MAX
          /   \
       MIN     MIN
      / \     / \
     3  12   2   8
```

**Left MIN explored:** values 3, 12 → MIN backs up **3**. At MAX, $\alpha = 3$.

**Right MIN:** first child returns **2**. MIN will choose $\leq 2$. Parent MAX already has 3 from left - right branch cannot beat 3 even if second child is 8.

**Prune** second child of right MIN - never evaluated.

**Nodes expanded:** 5 leaves instead of 6. On larger trees, savings compound exponentially with good ordering.

---

## Implementation Notes

| Technique | Detail |
|-----------|--------|
| **Iterative deepening** | Run alpha-beta at $d=1,2,3,\ldots$ until time limit - reuse move ordering from shallow passes |
| **Aspiration windows** | Narrow $(\alpha, \beta)$ around expected score - re-search if fail |
| **Null-move pruning** | Skip move to test if position still strong - heuristic, not sound alone |
| **Late move reduction** | Search unpromising moves at reduced depth first |

```python
def iterative_deepening_ab(game, state, max_time):
    best_move = None
    for depth in range(1, 100):
        if time_exceeded(max_time):
            break
        val, best_move = alphabeta(game, state, -inf, inf, depth)
    return best_move
```

**AIMA connection:** `alphabeta_search` in aima-python returns `(value, best_action)` - compare node counts with and without pruning on Tic-Tac-Toe.

---

## Pruning Efficiency Table

| Move ordering quality | Nodes (approx) | $b_{\text{eff}}$ at depth 8 |
|-----------------------|----------------|------------------------------|
| Random | $O(b^m)$ | $\approx b$ |
| Good captures-first | $O(b^{m/2})$ | $\approx \sqrt{b}$ |
| Perfect (theoretical) | Minimal | $\sqrt{b}$ |

For $b=35$, $m=8$: minimax $\approx 35^8 \approx 2 \times 10^{12}$; ideal alpha-beta $\approx 35^4 \approx 1.5 \times 10^6$.

---

## Common Mistakes

**Wrong inequality at MIN node.** Prune when $v \leq \alpha$ (not $\geq \beta$) at MIN - easy to invert when coding.

**Not passing updated $\alpha$, $\beta$ down.** Bounds must propagate along current path only, not globally.

**Assuming pruning changes best move.** It should not - if it does, bug in bound updates or move ordering side effects.

**Ignoring transposition table bounds.** Storing only exact values without bound type causes incorrect pruning.

---

## Reflection Questions

1. Why does alpha-beta not change the minimax value at the root?
2. What move ordering maximizes pruning?
3. Why is worst-case alpha-beta still exponential in $m$?
4. How does iterative deepening interact with alpha-beta?
5. What is stored in a transposition table entry?

---

**Next:** [Section 5.4 - Evaluation Functions](./section-04-imperfect-decisions.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 5.3. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Knuth, D. E., & Moore, R. W. (1975). An analysis of alpha-beta pruning. *Artificial Intelligence*.
3. Pearl, J. (1984). *Heuristics*, Chapter 5.
4. Russell, S., & Norvig, P. - `games.py` alphabeta. [aima-python](https://github.com/aimacode/aima-python)
5. Marsland, T. A. - computer chess search surveys.