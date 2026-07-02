# Section 4.3: Nondeterministic Search

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 4.3  
> **Previous:** [Section 4.2 - Continuous Spaces](./section-02-continuous-spaces.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## When Actions Have Unpredictable Outcomes

[Chapter 03](../chapter-03-solving-problems-by-searching/README.md) assumed **deterministic** transition models: $\text{RESULT}(s, a)$ is a single state. Real environments violate this - wet floors, unreliable actuators, adversarial wind, packet loss. A **nondeterministic** model returns a **set** or **distribution** of possible outcomes.

$$
\text{RESULT}(s, a) \in \{s_1', s_2', \ldots\} \quad \text{or} \quad P(s' \mid s, a)
$$
> **Readable form:** After you act, the world might land in several different states - search must branch on uncertainty, not follow a single thread.

Humorous analogy: deterministic search is **following a recipe**; nondeterministic search is **cooking while someone randomly swaps your ingredients** - you need backup plans, not one fixed sequence.

From [Section 2.3](../chapter-02-intelligent-agents/section-03-the-nature-of-environments.md): **stochastic** environments require contingency. [Chapter 17](../chapter-17-making-complex-decisions/README.md) formalizes this with MDPs; this section builds the **search-tree** view.

---

## AND-OR Search Trees

Standard OR trees ([Section 3.1](../chapter-03-solving-problems-by-searching/section-01-problem-solving-agents.md)): agent picks one action per node - **OR** node.

Nondeterministic problems add **AND** nodes: after the agent acts, **nature** chooses an outcome. The agent needs a strategy that works for **all** (or most) nature's choices.

```
           OR (agent at s)
          /    |    \
        a1    a2    a3
        |     |     |
       AND   AND   AND
      / | \  ...
   s1' s2' s3'
```

| Node type | Who decides | Children meaning |
|-----------|-------------|------------------|
| **OR** | Agent | Alternative actions |
| **AND** | Nature | Possible outcomes of one action |

A **solution** is a **conditional plan** (contingency plan): at each OR node, pick action $a$; at each AND node, specify what to do for **every** outcome $s'$.

> **Readable form:** A solution is not one path - it is a tree of "if this happens, do that" branches.

---

## Contingency Planning

**Contingency planning** computes a plan **before** execution that handles every observable outcome. Contrast with **online** replanning ([Section 4.5](./section-05-online-search.md)).

**Algorithm sketch (AND-OR search):**

1. Start at OR node with current state.
2. For each action, create AND node with all $\text{RESULT}(s, a)$ children.
3. **OR node solved** if some child AND node is solved.
4. **AND node solved** if **every** child OR subtree is solved.

```python
def and_or_search(problem, state):
  if problem.is_goal(state):
    return PlanNode(goal=state)
  for action in problem.actions(state):
    outcomes = problem.results(state, action)  # list of possible states
    subplans = [and_or_search(problem, s2) for s2 in outcomes]
    if all(subplans):
      return PlanNode(action=action, branches=dict(zip(outcomes, subplans)))
  return None  # failure
```

| Planning style | When outcomes known | Plan form |
|----------------|---------------------|-----------|
| **Classical (OR only)** | Deterministic | Linear sequence |
| **Contingency (AND-OR)** | Nondeterministic, fully observable | Tree / policy |
| **Online** | Discovered during execution | Replan each step |

---

## Slippery Grid World

AIMA's **slippery vacuum** or grid: agent moves Up/Down/Left/Right but with probability $p$ slips perpendicular.

Example: intended **Up** yields:

| Outcome | Probability |
|---------|-------------|
| Up | 0.8 |
| Left | 0.1 |
| Right | 0.1 |

**Deterministic plan** $\langle \text{Up}, \text{Up} \rangle$ may fail if nature slips. **Contingency plan:** "Move Up; **if** landed left of target, move Right; **if** right, move Left; **if** correct, continue."

For **optimality** under probabilities, switch to **expectimax** or **MDP value iteration** ([Chapter 17](../chapter-17-making-complex-decisions/README.md)). AND-OR here targets **goal reachability** regardless of which branch nature takes (qualitative) or with threshold probability.

---

## Sensorless Planning (Blind Robot)

**Sensorless planning** (conformant planning): the agent has **no percepts** - it cannot observe the outcome of its actions. The state of knowledge is **belief** over physical states; after each action, the agent knows only that it applied $a$, not where it ended up.

Initially: belief $b_0 = \{s_1, s_2, \ldots\}$ (all states consistent with problem).

After action $a$:

$$
b' = \{ s' \mid \exists s \in b,\, s' \in \text{RESULT}(s, a) \}
$$
> **Readable form:** New belief = all states reachable from **some** state you might still be in, after action $a$.

**Goal:** belief subset of goal states (e.g., "definitely at goal" or "possibly at goal").

Sensorless planning is search in **belief space** - preview of [Section 4.4](./section-04-partial-observability.md). AND-OR structure appears: nature picks the true physical state within the belief.

Humorous analogy: sensorless planning is **navigating your apartment in the dark with amnesia** - you memorize a sequence of turns knowing you could have started in any room.

---

## PEAS: Slippery Delivery Robot

| PEAS | Nondeterministic grid |
|------|------------------------|
| **P** - Performance | Reach goal reliably; minimize expected steps |
| **E** - Environment | Grid, slippery tiles, no adversary |
| **A** - Actuators | Move in four directions |
| **S** - Sensors | Perfect position sensing (contingency) OR none (sensorless) |

| Variant | Observability | Search type |
|---------|---------------|-------------|
| Slippery + GPS | Full | AND-OR contingency |
| Slippery + no GPS | None | Belief-state ([Section 4.4](./section-04-partial-observability.md)) |
| Known slip model | - | MDP optimal policy ([Chapter 17](../chapter-17-making-complex-decisions/README.md)) |

---

## AND-OR vs Game Trees

| Structure | Agent nodes | Opponent nodes |
|-----------|-------------|----------------|
| **AND-OR** | OR | AND (nature) |
| **Minimax** ([Chapter 05](../chapter-05-adversarial-search-games/README.md)) | MAX | MIN (adversary) |

Nature is not **malicious** - it may satisfy all outcomes in AND node. Adversary **minimizes** agent utility. Contingency planning assumes nature can pick the **worst** outcome for the agent among successors (pessimistic) or all must be handled (safety).

---

## Complexity of Contingency Plans

If branching factor is $b$ actions and $k$ outcomes per action, plan depth $d$:

$$
\text{OR nodes} \sim O(b^d), \quad \text{AND fan-out} \sim k^d
$$
> **Readable form:** Contingency trees grow as actions AND outcomes multiply - often far larger than deterministic plans.

**Pruning:** If two outcomes lead to equivalent subtrees, merge. **Dead-end beliefs** in sensorless case prune entire branches.

---

## Worked Example: Tiny Sensorless 1×3 Corridor

Cells $\{A, B, C\}$, robot starts **unknown** among $\{A, B, C\}$, goal = $C$. Actions: **MoveRight** (stops at wall).

**Belief evolution:**

| Step | Action | Belief after |
|------|--------|--------------|
| 0 | - | $\{A, B, C\}$ |
| 1 | MoveRight | $\{B, C, C\} = \{B, C\}$ |
| 2 | MoveRight | $\{C, C\} = \{C\}$ |

Plan: $\langle \text{MoveRight}, \text{MoveRight} \rangle$ - **guaranteed** goal without sensing.

If goal were $B$: from $\{A,B,C\}$, one MoveRight gives $\{B,C\}$ - cannot distinguish $B$ from $C$ without sensors. **No sensorless plan** reaches $B$ with certainty.

| Goal | Sensorless plan exists? |
|------|-------------------------|
| $C$ | Yes (2 moves) |
| $B$ | No (symmetry) |
| $A$ | Yes (0 moves if already there - vacuous) |

This illustrates **information gain** without explicit sensing - actions **reduce** belief.

---

## Relationship to MDPs

| AND-OR / contingency | MDP |
|------------------------|-----|
| Qualitative "must succeed" | Quantitative expected utility |
| Tree of if-then rules | Policy $\pi(s)$ |
| Exponential plan size | Polynomial in $|S|$ for VI (tabular) |

[Chapter 17](../chapter-17-making-complex-decisions/README.md) replaces exhaustive AND-OR with **Bellman backups**. Search view remains pedagogically valuable for **understanding** branching on uncertainty.

---

## Connection to Other Chapters

| Topic | Where |
|-------|-------|
| Adversarial AND structure | [Chapter 05 - Minimax](../chapter-05-adversarial-search-games/section-02-minimax.md) |
| Belief-state search | [Section 4.4](./section-04-partial-observability.md) |
| Conformant planning | [Chapter 12](../chapter-11-automated-planning/README.md) |
| POMDPs | [Chapter 17](../chapter-17-making-complex-decisions/README.md) |

---

## Common Mistakes

**Treating stochastic RESULT as deterministic.** Picking "most likely" outcome only - plan fails on slip.

**Confusing sensorless with partial observability.** Sensorless = **no** percepts ever; partial = noisy or limited percepts ([Section 4.4](./section-04-partial-observability.md)).

**Linear plan for AND node.** Single sequence cannot branch on outcomes - need conditional structure.

**Ignoring AND-node "all children" rule.** Solving one outcome branch is insufficient.

---

## Reflection Questions

1. What is the difference between an OR node and an AND node in an AND-OR tree?
2. Why does a slippery grid break the classical search assumption of a unique $\text{RESULT}(s,a)$?
3. In sensorless planning, how can an action reduce the belief set without any sensors?
4. Give an example where no sensorless plan exists to reach a goal with certainty.
5. When is contingency planning preferable to online replanning ([Section 4.5](./section-05-online-search.md))?

---

**Next:** [Section 4.4 - Partial Observability](./section-04-partial-observability.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 4.3. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - `search.py` AND-OR search. [aima-python](https://github.com/aimacode/aima-python)
3. Goldman, R. P., & Boddy, M. S. - contingency planning surveys.
4. Cimatti, A., & Roveri, M. - conformant planning via search.
5. MIT 6.034 - Nondeterministic and sensorless planning. [OCW](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)



