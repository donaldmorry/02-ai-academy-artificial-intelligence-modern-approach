# Section 5.6: Monte Carlo Tree Search

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 5.4.1  
> **Previous:** [Section 5.5 - Stochastic Games](./section-05-stochastic-games.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Search Meets Sampling

[Minimax](./section-02-minimax.md) needs full tree expansion. [MCTS](../../GLOSSARY.md#mcts) (**Monte Carlo Tree Search**) builds an **asymmetric** search tree guided by **random rollouts** - revolutionized Go, then chess (AlphaZero).

> **Readable form:** Repeatedly simulate random playouts from promising lines; invest more simulations where win rate looks good.

Four phases per iteration: **Selection, Expansion, Simulation, Backpropagation**.

---

## MCTS Loop

```
repeat for budget N:
  1. SELECT   - descend tree using UCT until leaf
  2. EXPAND   - add one child for unexplored action
  3. SIMULATE - rollout to terminal with default policy
  4. BACKUP   - update visit counts and win totals on path
return action with most visits from root
```

---

## Tree Node Statistics

Each node stores:

| Statistic | Meaning |
|-----------|---------|
| $N(s)$ | Visit count |
| $W(s)$ | Total win value (from MAX perspective) |
| $Q(s) = W(s)/N(s)$ | Estimated value |

---

## UCT Selection Rule

**Upper Confidence bounds applied to Trees** (Kocsis & Szepesvári, 2006):

$$
U(s,a) = \frac{Q(s,a)}{N(s,a)} + c \sqrt{\frac{\ln N(s)}{N(s,a)}}
$$
> **Readable form:** Pick action balancing average win rate plus exploration bonus for rarely tried moves.

Select action maximizing $U(s,a)$ during descent. Exploration term grows for untried or under-sampled actions.

```python
import math

def uct_score(parent_visits, child_wins, child_visits, c=1.414):
    if child_visits == 0:
        return float('inf')
    exploitation = child_wins / child_visits
    exploration = c * math.sqrt(math.log(parent_visits) / child_visits)
    return exploitation + exploration
```

---

## Implementation Skeleton

```python
class MCTSNode:
    def __init__(self, state, parent=None, action=None):
        self.state = state
        self.parent = parent
        self.action = action
        self.children = {}
        self.wins = 0.0
        self.visits = 0

def mcts(root_state, game, iterations=1000):
    root = MCTSNode(root_state)
    for _ in range(iterations):
        node = select(root, game)
        node = expand(node, game)
        reward = simulate(node.state, game)
        backpropagate(node, reward)
    return max(root.children.items(), key=lambda x: x[1].visits)[0]

def select(node, game):
    while not game.is_terminal(node.state) and node.children:
        if len(node.children) < len(game.actions(node.state)):
            return node
        node = max(node.children.values(),
                   key=lambda c: uct_score(node.visits, c.wins, c.visits))
    return node
```

---

## Rollout Policy

Default policy: random legal moves until terminal. Better policies (lightweight eval, policy network) improve MCTS strength.

| Rollout quality | Effect |
|-----------------|--------|
| Random | Noisy but unbiased with enough samples |
| Heuristic | Faster convergence |
| Learned (AlphaGo) | Superhuman play |

---

## AlphaGo Architecture (Conceptual)

1. **Policy network** $p(a|s)$ - guides expansion and rollouts
2. **Value network** $v(s)$ - replaces rollout terminal eval
3. MCTS with neural priors - self-play training

Combines [deep learning](../chapter-21-deep-learning/README.md) with game-tree search.

---

## MCTS vs Minimax

| Aspect | Minimax + $\alpha\beta$ | MCTS |
|--------|-------------------------|------|
| Tree shape | Uniform depth | Asymmetric |
| Leaf eval | Static function | Rollout or value net |
| Branching | All moves to depth $d$ | Focuses on promising |
| Best for | Chess (historically) | Go, large branching |

Go branching ~250 - minimax impractical; MCTS shines.

---

## Anytime Property

More iterations → better move estimate. Stop when time budget exhausted - natural for real-time games.

---

## PEAS: Go MCTS Agent

| PEAS | Classic Go MCTS |
|------|-----------------|
| **P** | Maximize win rate |
| **E** | 19×19 board, rules |
| **A** | Place stone |
| **S** | Full board |

---

## Deterministic Games with MCTS

Chess engines now use neural MCTS (AlphaZero). Pure MCTS without policy prior is weaker than alpha-beta in chess - domain-dependent tool choice.

---

## Multiplayer and Incomplete Information

MCTS extends to multiplayer (max visits per root child) and imperfect information (determinization, information sets - [Section 5.7](./section-07-partial-observability-in-games.md)).

---

## Worked Example: 3 Iterations

Root has actions A, B. Iter 1: rollout from A wins → $N(A)=1, W(A)=1$. Iter 2: UCT explores B → loss. Iter 3: A has higher exploitation → more A visits. After 1000 iters, pick most visited.

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

1. List the four MCTS phases in order.
2. What does the exploration term in UCT encourage?
3. Why is MCTS well-suited to Go but historically less so to chess?
4. How did AlphaGo improve on pure MCTS?
5. What is the anytime property?

---

**Next:** [Section 5.7 - Partial Observability in Games](./section-07-partial-observability-in-games.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 5.4.1. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Browne, C., et al. (2012). Survey of MCTS methods. *IEEE TCIAIG*.
3. Silver, D., et al. (2016). Mastering Go with deep neural networks and tree search. *Nature*.
4. Kocsis, L., & Szepesvári, C. (2006). Bandit based Monte-Carlo planning.
5. Russell, S., & Norvig, P. - MCTS examples. [aima-python](https://github.com/aimacode/aima-python)



