# Section 5.7: Partial Observability in Games

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 5.5-5.6  
> **Previous:** [Section 5.6 - Monte Carlo Tree Search](./section-06-monte-carlo-tree-search.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Imperfect Information Games

[Section 5.1](./section-01-games-and-game-trees.md) assumed **perfect information** - both players see the full state. **Poker, bridge, Kriegspiel** hide information: cards in hand, face-down pieces.

> **Readable form:** Players maintain beliefs over possible hidden states; optimal play mixes information sets with game-tree search.

AIMA distinguishes **imperfect information** (private cards) from **partial observability** (fog of war) - both require belief tracking beyond standard minimax.

---

## Information Sets

In standard game theory, an **information set** groups indistinguishable states from one player's view.

| Perfect info | Imperfect info |
|--------------|----------------|
| One node per state | Multiple states per information set |
| Minimax on states | Strategy on information sets |

Player chooses same action for all states in the set - **mixed strategies** may be optimal.

---

## Belief States in Card Games

After each action and percept, update belief over opponent hands consistent with public history:

$$
b'(s') \propto P(\text{percepts} \mid s') \sum_{s \in b} P(s' \mid s, a)
$$
> **Readable form:** Bayesian update over hidden deals - same framework as [Section 4.7](../chapter-04-search-complex-environments/section-04-partial-observability.md).

---

## Kriegspiel

Chess where players **cannot see** opponent pieces - only announcements (check, capture). Belief over board configurations explodes. Humans use deduction; AI uses belief-state search or sampling.

---

## Poker: Abstraction and Equilibrium

Heads-up poker solved at scale (Bowling et al., 2015) using:

| Technique | Role |
|-----------|------|
| **Card abstraction** | Bucket similar hands |
| **Counterfactual regret minimization** | Approximate Nash equilibrium |
| **Belief tracking** | Opponent range narrowing |

Not pure minimax - **game-theoretic equilibrium** in imperfect information.

---

## Determinization for MCTS

**Imperfect information MCTS** heuristic:

1. Sample a **determinization** - complete hidden state consistent with observations
2. Run [MCTS](./section-06-monte-carlo-tree-search.md) on perfect-information sample
3. Aggregate statistics across samples

```python
def determinized_mcts(partial_state, game, samples=100, iters=500):
    action_votes = {}
    for _ in range(samples):
        full_state = game.sample_determinization(partial_state)
        action = mcts(full_state, game, iterations=iters)
        action_votes[action] = action_votes.get(action, 0) + 1
    return max(action_votes, key=action_votes.get)
```

Biased if determinizations unrepresentative - known limitation.

---

## Bridge and Bidding

Bidding conveys information - partnership game with partial observability. AI bridge lags poker/chess but uses constraint satisfaction + belief ([Chapter 06](../chapter-06-constraint-satisfaction/README.md)).

---

## PEAS: Texas Hold'em Bot

| PEAS | Poker agent |
|------|-------------|
| **P** | Maximize chip EV |
| **E** | Cards, bets, opponents |
| **A** | Fold/call/raise |
| **S** | Own cards, public board, betting history |

**Hidden:** opponent hole cards, undealt deck order.

---

## Comparison to Wumpus World

[Chapter 07](../chapter-07-logical-agents/section-07-wumpus-world.md): single agent, logical belief. Imperfect games: **adversarial** hidden state - opponent may exploit your beliefs.

| Wumpus | Poker |
|--------|-------|
| Nature static | Opponent adapts |
| Logic KB | Probability + game theory |
| Safe squares | Pot odds, bluffing |

---

## Nash Equilibrium Preview

Two-player zero-sum imperfect games: optimal play is **Nash equilibrium** - neither player benefits from unilateral deviation.

$$
u_1(\sigma_1^*, \sigma_2) \geq u_1(\sigma_1, \sigma_2) \quad \forall \sigma_1
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

Mixed strategies $\sigma$ randomize over actions. [Chapter 18](../chapter-18-multiagent-decision-making/README.md) expands game theory.

---

## StarCraft and Real-Time Partial Observability

RTS games: **fog of war** + simultaneous moves + real time. Belief maps, scouting, abstraction - research frontier beyond classical AIMA minimax.

---

## Worked Example: Simplified Kuhn Poker

Three-card deck {J, Q, K}. One card each, one bet round. Perfect recall information sets - solve by backward induction on trees with chance (deal) and imperfect info nodes.

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

---

## Kuhn Poker and Information Sets (Preview)

Three-card Kuhn poker illustrates **imperfect information**: players cannot distinguish states that differ only in opponent's card. Optimal play requires **mixed strategies** - not pure minimax on a single tree. See [Section 5.7](./section-07-partial-observability-in-games.md) and [Chapter 18](../chapter-18-multiagent-decision-making/README.md).

---

## Reflection Questions

1. What is an information set?
2. Why doesn't standard minimax apply directly to poker?
3. Explain determinization in imperfect-information MCTS.
4. How does belief-state search from Chapter 04 relate to card games?
5. What is a mixed strategy and when is it necessary?

---

**Next:** [Section 5.8 - Connect Four Lab](./section-08-connect-four-adversarial-search-lab.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Sections 5.5-5.6. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Bowling, M., et al. (2015). Heads-up limit hold'em poker is solved. *Science*.
3. Billings, D., et al. - poker AI surveys.
4. Silver, D., & Johanson, M. - imperfect information games.
5. Osborne, M. J., & Rubinstein, A. (1994). *A Course in Game Theory*.
