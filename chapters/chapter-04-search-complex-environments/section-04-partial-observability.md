# Section 4.4: Partial Observability

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 4.4  
> **Previous:** [Section 4.3 - Nondeterministic Search](./section-03-nondeterministic-search.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Acting Without Knowing the True State

[Section 2.3](../chapter-02-intelligent-agents/section-03-the-nature-of-environments.md) classified environments by **observability**. Classical search ([Chapter 03](../chapter-03-solving-problems-by-searching/README.md)) assumes the agent always knows $s$ - full GPS, complete board view. **Partially observable** settings: noisy sensors, limited range, hidden variables.

The agent maintains a **[belief state](../../GLOSSARY.md#belief-state)** $b$ - a set or distribution over physical states it considers possible given history.

> **Readable form:** Instead of "I am in Bucharest," the agent thinks "I am in Bucharest with 70% probability, Giurgiu with 30%."

Humorous analogy: partial observability is **playing hide-and-seek where you wear fogged goggles** - you carry a mental list of every corner you might still be standing in.

[Section 4.3](./section-03-nondeterministic-search.md) introduced sensorless planning (belief with no percepts). This section adds **percepts** that refine beliefs without fully revealing state.

---

## Belief States

Let $S$ be physical states, $E$ percepts. A **belief state** $b \subseteq S$ (or probability mass function $b(s)$) summarizes all information from prior belief and percept history.

**Update after action $a$ and percept $e$:**

1. **Prediction (action):** $b' = \{ s' \mid \exists s \in b,\, s' = \text{RESULT}(s,a) \}$ (deterministic) or weighted by transition probabilities.
2. **Update (percept):** $b'' = \{ s \in b' \mid P(e \mid s) > 0 \}$ (or Bayes' rule).

$$
b''(s) = \eta \, P(e \mid s) \sum_{s' \in b} P(s \mid s', a) \, b(s')
$$
> **Readable form:** New belief = reweight each state by how likely the percept is there, after accounting for where you could have moved from.

For **sensorless** case, skip percept update - only prediction steps ([Section 4.3](./section-03-nondeterministic-search.md)).

| Representation | Size | Use |
|----------------|------|-----|
| **Explicit set** | Up to $|S|$ | Small grids |
| **Probability vector** | $|S|$ floats | POMDPs ([Chapter 17](../chapter-17-making-complex-decisions/README.md)) |
| **Logical formula** | Compact sometimes | Planning ([Chapter 12](../chapter-11-automated-planning/README.md)) |

---

## Belief-State Search

Formulate a new search problem:

| Component | Belief-space formulation |
|-----------|--------------------------|
| **Initial state** | $b_0$ from percepts |
| **Actions** | Same physical actions |
| **Transition** | $b' = \text{UPDATE}(b, a, e)$ - nature picks $e$ or enumerate percepts |
| **Goal test** | $b \subseteq G$ (all states in belief satisfy goal) |
| **Cost** | Per action or per planning step |

Search runs in **belief space**, not physical space. A single belief node may represent exponentially many physical states.

```
Physical states:     Belief states:
  A - B - C            {A,B,C}
                       {B,C}
                       {C}  ← goal belief
```

---

## Belief-State Search Complexity

Let $|S| = n$, branching factor $b$ actions, $|E|$ distinct percepts per step.

| Quantity | Worst case |
|----------|------------|
| **Belief states** | $2^n$ (each subset of $S$) |
| **Search depth $d$** | Physical depth × percept branching |
| **Time** | $O((b \cdot |E|)^d \cdot T_{\text{update}})$ |

> **Readable form:** Belief search can blow up to exponential in the number of physical states - you are searching over sets of worlds, not one world.

**Example:** $n = 10$ grid cells → up to $2^{10} = 1024$ belief nodes. $n = 30$ → over one billion - intractable without structure.

| Mitigation | Idea |
|------------|------|
| **Merge equivalent beliefs** | Canonicalize sets |
| **Particle filter** | Approximate with $k$ samples |
| **Domain-specific invariants** | Only reachable beliefs |
| **POMDP solvers** | Avoid explicit $2^n$ ([Chapter 17](../chapter-17-making-complex-decisions/README.md)) |

This is why [Chapter 03](../chapter-03-solving-problems-by-searching/section-07-search-analysis.md) completeness guarantees do not transfer blindly - the **formulated** graph may be astronomical.

---

## Localization Example

Robot on a **map** with distinctive landmarks. Percepts: "see landmark L" or "no landmark."

| Physical state | Percept at state |
|----------------|------------------|
| Near L | see L |
| Far from L | none |

After **MoveForward**, belief spreads along corridor; percept **see L** collapses belief to states with line-of-sight to L.

**Kidnapped robot problem:** belief reset to uniform over $S$ - any localization plan must recover from arbitrary $s$.

---

## PEAS: Hospital Delivery Robot (Partial Observability)

| PEAS | Belief-state navigation |
|------|-------------------------|
| **P** - Performance | Deliver to ward; minimize time and wrong-room errors |
| **E** - Environment | Hospital corridors, doors, people |
| **A** - Actuators | Drive, announce arrival |
| **S** - Sensors | WiFi fingerprint, door labels (sometimes wrong) |

| Property | Value |
|----------|-------|
| Observability | **Partial** - ambiguous junctions |
| Determinism | Mostly deterministic motion |
| Planning | Belief-state search or POMDP |

---

## Comparison: Physical vs Belief Search

| Aspect | Physical search | Belief-state search |
|--------|-----------------|---------------------|
| State | One $s \in S$ | Subset or dist. on $S$ |
| Goal | $s \in G$ | $b \subseteq G$ |
| Size $|S|$ | Problem given | Up to $2^{|S|}$ |
| Percepts | Ignored | Drive updates |
| Solution | Action sequence | Contingent or sensing policy |

**Active sensing:** actions chosen to **reduce** belief entropy - explore to disambiguate ([Section 4.6](./section-06-lrta-and-exploration.md)).

---

## Worked Example: 2×2 Grid with Noisy Sensor

Cells numbered 1-4. Robot at 1 or 2 (cannot distinguish initially): $b_0 = \{1, 2\}$. Goal: cell 4. Actions: MoveRight (deterministic).

**Step 1 - MoveRight from $b_0$:**

From 1 → 2; from 2 → 3 (assume layout 1-2 / 3-4). Belief $b_1 = \{2, 3\}$.

**Percept:** noisy "wall on right" - true at cells 2 and 4.

| Cell | $P(\text{wall-right} \mid \text{cell})$ |
|------|----------------------------------------|
| 2 | 0.9 |
| 3 | 0.1 |

Bayes update on $b_1$:

- Prior mass on 2 and 3 (equal 0.5 each in $b_1$).
- Likelihood: 0.9 for 2, 0.1 for 3.
- Posterior favors 2 but 3 not eliminated.

If percept were **see opening on right** (likely at 3): belief shifts toward 3.

**Plan:** continue moving right until belief $\subseteq \{4\}$ or percept confirms goal landmark.

| Step | Belief (set approx.) | Action |
|------|----------------------|--------|
| 0 | $\{1,2\}$ | MoveRight |
| 1 | $\{2,3\}$ | MoveRight (if belief allows) |
| 2 | $\{3,4\}$ or $\{4\}$ | Stop at goal test |

Exact sets depend on map topology - exercise: draw AND-OR over beliefs.

---

## Link to POMDPs

**Partially Observable MDP (POMDP):** belief $b$ is sufficient statistic; optimal policy $\pi(b)$ maximizes expected utility.

Belief-state search is the **qualitative** special case: find **any** plan reaching $b \subseteq G$ with probability 1 or above threshold.

| Tool | Optimality | Scalability |
|------|------------|-------------|
| Belief-state BFS | Shortest plan in belief graph | Poor for large $n$ |
| POMDP value iteration | Expected utility optimal | Still hard (#P-hard) |
| Online + LRTA* | Suboptimal, reactive | Better ([Section 4.6](./section-06-lrta-and-exploration.md)) |

---

## Connection to Other Chapters

| Topic | Chapter |
|-------|--------|
| Sensorless (no percepts) | [Section 4.3](./section-03-nondeterministic-search.md) |
| Rationality under uncertainty | [Section 2.2](../chapter-02-intelligent-agents/section-02-good-behavior-and-rationality.md) |
| Bayesian networks | [Chapter 13](../chapter-13-probabilistic-reasoning/README.md) |
| Kalman / particle filters | [Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md) |

---

## Common Mistakes

**Assuming unique physical state.** Planning in $S$ when sensors are ambiguous - plans fail at execution.

**Ignoring percept after action order.** Update must predict then observe (or model joint model correctly).

**Exponential blowup surprise.** $2^n$ beliefs is inherent to worst case - need approximation in practice.

**Goal test too weak.** "Possibly at goal" ($b \cap G \neq \emptyset$) vs "certainly at goal" ($b \subseteq G$).

---

## Reflection Questions

1. What is a belief state, and how does it differ from a physical state?
2. Why can the number of belief states be $2^{|S|}$ in the worst case?
3. How does a percept refine a belief set? Write the update in words for "see landmark L."
4. When is belief-state search preferable to solving a full POMDP?
5. Relate belief-state goals to the sensorless plans in [Section 4.3](./section-03-nondeterministic-search.md).

---

**Next:** [Section 4.5 - Online Search](./section-05-online-search.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 4.4. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Kaelbling, L. P., Littman, M. L., & Cassandra, A. R. - POMDPs. *Artificial Intelligence*.
3. Russell, S., & Norvig, P. - belief-state search in aima-python. [github.com/aimacode/aima-python](https://github.com/aimacode/aima-python)
4. LaValle, S. M. - planning under sensing. [Planning Algorithms](http://planning.cs.uiuc.edu/)
5. Thrun, S., Burgard, W., & Fox, D. - *Probabilistic Robotics*.



