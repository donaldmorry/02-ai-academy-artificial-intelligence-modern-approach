# Section 5.5: Stochastic Games

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 5.4  
> **Previous:** [Section 5.4 - Evaluation Functions](./section-04-imperfect-decisions.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Games with Chance

[Sections 5.1-5.4](./section-01-games-and-game-trees.md) assumed **deterministic** moves. **Stochastic games** add **chance nodes** - dice, card shuffles, random events - between player moves.

> **Readable form:** Game tree has MAX nodes, MIN nodes, and CHANCE nodes where nature picks outcomes with known probabilities.

Examples: backgammon, Monopoly, poker (with hidden cards - [Section 5.7](./section-07-partial-observability-in-games.md)).

---

## Expectiminimax

Generalization of [minimax](./section-02-minimax.md): at chance node $s$ with outcomes $r_i$ and probabilities $P(r_i)$:

$$
\text{EXPECTIMINIMAX}(s) = \sum_i P(r_i) \cdot \text{EXPECTIMINIMAX}(\text{RESULT}(s, r_i))
$$
> **Readable form:** Chance node value = weighted average of child values.

At MAX/MIN nodes, same max/min as before.

---

## Algorithm Sketch

```python
def expectiminimax(game, state):
    if game.is_terminal(state):
        return game.utility(state, 'MAX'), None
    player = game.to_move(state)
    if player == 'CHANCE':
        expected = 0
        for outcome, prob in game.chance_outcomes(state):
            val, _ = expectiminimax(game, game.result(state, outcome))
            expected += prob * val
        return expected, None
    if player == 'MAX':
        return max_child(game, state, expectiminimax)
    else:
        return min_child(game, state, expectiminimax)
```

---

## Game Tree Structure

```
        MAX
       /   \
    CHANCE  ...
    / | \
  .2 .5 .3  (probabilities)
   |  |  |
  MIN MIN MIN
```

Branching at chance multiplies children - complexity worse than deterministic minimax.

---

## Backgammon

~20 dice outcomes per roll (doubles counted). Depth $d$ with branching $b$ actions and chance $c$:

$$
\text{Time} \approx O((b \cdot c)^{d/2}) \text{ with pruning extensions}
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

TD-Gammon (Tesauro, 1992) used neural network eval + expectiminimax-style rollout - milestone in learned game play.

---

## Utility Ranges in Backgammon

Utilities from 0 to 1 (win probability) or $-192$ to $+192$ (gammon/backgammon scoring). Eval must match utility scale for expectiminimax backup.

---

## Position Evaluation Under Uncertainty

Chance introduces **risk**:

| Position type | Character |
|---------------|-----------|
| Safe lead | High expected utility, low variance |
| Gammon chase | High mean, high risk |

Pure expectiminimax maximizes **expected** utility - rational under risk neutrality ([Chapter 16](../chapter-16-making-simple-decisions/README.md)).

---

## Alpha-Beta for Stochastic Games?

Standard alpha-beta **does not** apply directly at chance nodes - no pruning via $\alpha \geq \beta$ without extra assumptions.

**ProbCut**, **scout** variants exist for practical engines - beyond AIMA core.

---

## Monte Carlo Alternative

[MCTS](./section-06-monte-carlo-tree-search.md) samples chance outcomes in rollouts instead of full expectiminimax tree - scales to Go and complex games.

| Expectiminimax | MCTS |
|----------------|------|
| All chance branches | Sampled rollouts |
| Exact expectation | Monte Carlo estimate |
| Exponential | Polynomial per iteration |

---

## PEAS: Backgammon Agent

| PEAS | TD-Gammon style |
|------|-----------------|
| **P** | Maximize match win probability |
| **E** | Board, dice |
| **A** | Legal move |
| **S** | Full board, last dice roll |

Environment: **multiagent, stochastic, sequential, fully observable** (after dice revealed).

---

## Worked Example

MAX to move, then fair die $\{1,2,3\}$ each prob $1/3$, then MIN minimizes. Die=1 → utility 10; die=2 → 4; die=3 → 6. Chance value = $(10+4+6)/3 = 6.67$. MIN at each die outcome picks worst for MAX before averaging.

---

## Connection to MDPs

[Chapter 17](../chapter-17-making-complex-decisions/README.md): stochastic games are **Markov games** - states, actions, transition probabilities, rewards. Expectiminimax is finite-horizon special case.

---

## Extended Worked Example: Dice Game Tree

MAX chooses **Play** or **Pass**. If **Play**, fair die $\{1,2,3,4,5,6\}$ each with probability $1/6$:

| Die | Utility for MAX |
|-----|-----------------|
| 1, 2 | 0 |
| 3, 4 | 5 |
| 5, 6 | 10 |

$$
\text{EXPECT}(\text{Play}) = \frac{2}{6}(0) + \frac{2}{6}(5) + \frac{2}{6}(10) = \frac{30}{6} = 5
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

**Pass** gives fixed utility 4. MAX chooses **Play** (5 > 4). MIN nodes below would minimize before chance averages - order of backup: **depth-first to leaves**, chance averages on way up.

---

## Implementation Notes

```python
def chance_node_value(game, state):
    total = 0.0
    for outcome, prob in game.chance_outcomes(state):
        u, _ = expectiminimax(game, game.result(state, outcome))
        total += prob * u
    return total, None
```

| Issue | Solution |
|-------|----------|
| Many dice outcomes | Merge equivalent outcomes (backgammon) |
| Hidden shuffle | Chance node at deal, belief over decks ([Section 5.7](./section-07-partial-observability-in-games.md)) |
| Explosion | MCTS sampling instead of full expansion |

**AIMA connection:** `expect_minimax` in aima-python extends `minmax` with `'CHANCE'` player - study backgammon-style state transitions.

---

## Expectiminimax vs. MCTS Comparison

| Aspect | Expectiminimax | MCTS |
|--------|----------------|------|
| Chance handling | Exact $\sum p_i v_i$ | Sample in rollout |
| Branching | Multiplies at chance | Samples one path |
| Optimality | Exact (finite tree) | Converges with iterations |
| Best for | Small chance branching | Go, large games |

---

## Risk and Utility (Preview)

Maximizing **expected** utility ignores **variance**. A gamble with mean 5 and variance 100 may be worse than certain 4 for risk-averse agents - [Chapter 16](../chapter-16-making-simple-decisions/README.md) utility theory.

---

## Common Mistakes

**Applying alpha-beta at chance nodes.** Standard bounds fail - need expectiminimax-specific pruning or switch to MCTS.

**Forgetting probabilities must sum to 1.** Normalization errors bias chance values.

**Treating stochastic opponent as chance.** Opponent choice is MIN, not nature - only **true** randomness gets chance nodes.

---

## Reflection Questions

1. How does a chance node differ from MIN and MAX nodes?
2. Write expectiminimax for a fair coin flip with two child utilities.
3. Why doesn't standard alpha-beta prune chance nodes?
4. Compare expectiminimax and MCTS for backgammon-scale branching.
5. When is maximizing expected utility insufficient for rational play?

---

**Next:** [Section 5.6 - Monte Carlo Tree Search](./section-06-monte-carlo-tree-search.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 5.4. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Tesauro, G. (1995). TD-Gammon. *Communications of the ACM*.
3. von Neumann, J. - game theory and mixed strategies.
4. Russell, S., & Norvig, P. - `games.py`. [aima-python](https://github.com/aimacode/aima-python)
5. Browne, C., et al. (2012). A survey of Monte Carlo tree search methods.



