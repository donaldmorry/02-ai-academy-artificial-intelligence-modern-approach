# Section 5.8: Game Theory Preview

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 5.8  
> **Previous:** [Section 5.7 - Partial Observability in Games](./section-07-partial-observability-in-games.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Beyond Zero-Sum Tree Search

[Minimax](./section-02-minimax.md) solves **two-player zero-sum** perfect-information games. **Game theory** studies strategic interaction broadly: multiple players, non-zero-sum payoffs, simultaneous moves, mixed strategies, equilibrium concepts.

> **Readable form:** Sometimes the best move is random; sometimes cooperating beats fighting - game theory names those situations.

Humorous analogy: minimax is **fencing** (zero-sum, alternating); game theory is **dinner party politics** - coalitions, bluffing, and deals that help everyone or trap everyone.

---

## Normal-Form (Strategic-Form) Games

Represented by **payoff matrix** - players choose actions simultaneously (or without observing other's current move).

**Prisoner's dilemma** (payoffs for row player, col player):

| | Cooperate | Defect |
|---|-----------|--------|
| **Cooperate** | 3, 3 | 0, 5 |
| **Defect** | 5, 0 | 1, 1 |

Each player independently picks row/column action. **Defect** is dominant for both - mutual defect (1,1) though (3,3) Pareto better.

---

## Dominant and Dominated Strategies

**Strictly dominant strategy:** best response **regardless** of opponent action.

**Dominated strategy:** always worse than some other - rational players never use it (under common assumptions).

Iterated elimination of dominated strategies simplifies matrix games.

---

## Nash Equilibrium

Profile of strategies $(s_1^*, \ldots, s_n^*)$ where **no player** benefits by **unilateral deviation**:

$$
\forall i, \forall s_i: \quad u_i(s_i^*, s_{-i}^*) \geq u_i(s_i, s_{-i}^*)
$$
> **Readable form:** At Nash equilibrium, everyone is playing a best response to everyone else - no one wants to switch alone.

**Pure Nash:** deterministic strategies. **Mixed Nash:** randomize (essential in poker - [Section 5.7](./section-07-partial-observability-in-games.md)).

---

## Mixed Strategies

Player $i$ chooses actions with probabilities $\mathbf{p}_i = (p_{i1}, \ldots, p_{ik})$, $\sum_j p_{ij} = 1$.

**Expected utility:**

$$
u_i(\mathbf{p}_i, \mathbf{p}_{-i}) = \sum_{a_i, a_{-i}} P(a_i) P(a_{-i}) \cdot \text{payoff}_i(a_i, a_{-i})
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Matching pennies: no pure Nash; mixed (50-50) is equilibrium.

---

## Zero-Sum vs. General-Sum

| | Zero-sum | General-sum |
|--|----------|-------------|
| **Payoffs** | $u_1 + u_2 = 0$ (constant sum) | Independent interests |
| **Example** | Chess, poker chips | Negotiation, climate policy |
| **Solution concept** | Minimax value | Nash, correlated equilibrium |
| **Cooperation** | Never beneficial | Often beneficial |

---

## Extensive Form vs. Normal Form

| Form | Representation |
|------|----------------|
| **Extensive** | Game tree ([Section 5.1](./section-01-games-and-game-trees.md)) |
| **Normal** | Payoff matrix |

Any finite extensive game converts to normal form (information sets collapse histories).

---

## Mechanism Design (Preview)

Reverse game theory: design **rules** so rational self-interested players produce desired outcomes (auctions, voting). [Chapter 18](../chapter-18-multiagent-decision-making/README.md) expands multiagent decision making.

---

## Connection to AI Systems

| AI system | Game-theoretic view |
|-----------|---------------------|
| Ad auctions | Bayes-Nash equilibrium |
| GAN training | Two-player non-zero-sum ([Course 4](https://github.com/donaldmorry/04-ai-academy-generative-deep-learning-systems/blob/main/README.md)) |
| Security games | Stackelberg equilibrium |
| LLM alignment | Principal-agent games |

---

## Minimax as Special Case

Two-player zero-sum perfect-information extensive game: minimax value = **game value**; optimal strategies exist (pure at decision nodes in perfect info; mixed often needed in imperfect info).

---

## PEAS: Auction Bidding Agent

| PEAS | Online ad auction |
|------|-------------------|
| **P** | Maximize click value per cost |
| **E** | Other bidders (hidden valuations) |
| **A** | Bid amount |
| **S** | Auction context, history |

**Simultaneous, partial info, general-sum.**

---

## Worked Example: Battle of Sexes

| | Ballet | Football |
|---|--------|----------|
| **Ballet** | 2, 1 | 0, 0 |
| **Football** | 0, 0 | 1, 2 |

Two pure Nash equilibria: (Ballet, Ballet) and (Football, Football). Mixed equilibrium coordinates with probability $\frac{2}{3}$ on partner's preferred option - coordination game.

---

## Limitations of Equilibrium Analysis

- Assumes **rationality** and common knowledge
- Multiple equilibria - which to play?
- Computation hard in large games
- Doesn't model learning or bounded rationality

**Bounded rationality** links to [Chapter 02](../chapter-02-intelligent-agents/section-02-good-behavior-and-rationality.md).

---

## Correlated Equilibrium (Preview)

Players coordinate via **signals** - broader than Nash. Used in distributed systems where independent best responses cause inefficiency.

---

## Repeated Games (Preview)

When prisoner's dilemma repeats infinitely, **tit-for-tat** and **folk theorems** enable cooperation - reputation matters. Connects to multiagent RL ([Chapter 22](../chapter-22-reinforcement-learning/README.md)).

---

## Stackelberg Games (Preview)

**Leader** commits to strategy first; **follower** responds optimally. Models security: defender places resources, attacker adapts. Used in airport screening and wildlife patrol routing.

---

## AIMA Chapter 5 Synthesis

| Section | Takeaway |
|--------|----------|
| [5.1](./section-01-games-and-game-trees.md) | Formulate games with utilities |
| [5.2](./section-02-minimax.md) | Optimal play vs. rational opponent |
| [5.3](./section-03-alpha-beta-pruning.md) | Prune without changing value |
| [5.4](./section-04-imperfect-decisions.md) | Eval + cutoff for depth |
| [5.5](./section-05-stochastic-games.md) | Expectation at chance nodes |
| [5.6](./section-06-monte-carlo-tree-search.md) | Rollouts when eval weak |
| [5.7](./section-07-partial-observability-in-games.md) | Information sets |
| [5.8](./section-08-game-theory-preview.md) | Nash beyond trees |

---

## Common Mistakes

- Confusing **zero-sum** with "one player always wins" - draws exist.
- Assuming Nash is unique - coordination games have multiple equilibria.
- Ignoring **mixed** strategies in bluffing games.

---

## Worked Exercise: Finding Pure Nash

Payoff matrix for two firms choosing High or Low price:

| | High | Low |
|---|------|-----|
| **High** | 3, 3 | 0, 5 |
| **Low** | 5, 0 | 1, 1 |

Row player: if col plays High, row prefers Low (5 > 3); if col plays Low, row prefers Low (1 > 0). **Low is dominant** for row. Symmetrically Low dominates for column. Unique Nash: (Low, Low) = (1,1) - same as prisoner's dilemma structure.

---

## Reflection Questions

1. Define Nash equilibrium in words.
2. Why is (Defect, Defect) Nash in prisoner's dilemma despite (Cooperate, Cooperate) being better for both?
3. When are mixed strategies necessary?
4. How does zero-sum differ from general-sum?
5. What is the relationship between minimax and game value?

---

**Next:** [Chapter 06 - Constraint Satisfaction Problems](../chapter-06-constraint-satisfaction/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 5.8. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Osborne, M. J., & Rubinstein, A. - *A Course in Game Theory*.
3. von Neumann, J., & Morgenstern, O. (1944). *Theory of Games and Economic Behavior*.
4. Nash, J. - equilibrium existence theorem.
5. MIT 6.034 - Game theory primer. [OCW](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)
