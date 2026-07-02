# Section 4.6: LRTA* and Exploration

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 4.5  
> **Previous:** [Section 4.5 - Online Search](./section-05-online-search.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Real-Time Heuristic Search

[Section 3.4](../chapter-03-solving-problems-by-searching/section-04-informed-search.md) introduced **A***: expand until goal found, then execute full path. **Learning Real-Time A* (LRTA*)** - Korf 1990 - selects **one step**, updates heuristic estimates from local experience, moves, repeats.

Suited to **online** agents ([Section 4.5](./section-05-online-search.md)) mapping **unknown terrain** where the graph is revealed incrementally.

> **Readable form:** LRTA* is A* that takes a single step, learns from what it just saw, and never needs the full path in memory upfront.

Humorous analogy: LRTA* is **hiking with a pencil on your map** - after each bend you erase wrong distance guesses and write better ones, instead of plotting the entire trail at the trailhead.

---

## LRTA* Algorithm

Each state $s$ stores **estimated cost-to-go** $H(s)$ (heuristic, initially often zero or $h(s)$).

At current state $s$:

1. **Lookahead:** For each neighbor $s'$ via action $a$, compute one-step lookahead value:

$$
f(s, a) = c(s, a, s') + H(s')
$$
> **Readable form:** Step cost plus estimated remaining cost from the neighbor.

2. **Choose** action minimizing $f(s,a)$ (break ties toward $s'$ with smallest $H(s')$).

3. **Update** current state estimate:

$$
H(s) \leftarrow \min_{a} f(s, a)
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

4. **Move** to chosen $s'$; repeat until goal.

```python
def lrta_star(problem, s, H, lookahead=1):
  while not problem.is_goal(s):
    neighbors = [(a, problem.result(s, a), problem.cost(s, a))
                 for a in problem.actions(s)]
    f_vals = [(c + H[s2], a, s2) for a, s2, c in neighbors]
    f_min, best_a, best_s2 = min(f_vals)
    H[s] = f_min                    # learning step
    s = problem.execute(best_a)     # real move
  return s
```

**Learning:** $H(s)$ increases toward true shortest-path cost as agent explores - **converges** on repeated trials or single trial in static graph under conditions.

---

## Connection to A*

| Feature | A* | LRTA* |
|---------|-----|-------|
| **Expansion** | Many nodes per decision | Local neighborhood |
| **Memory** | Open/closed lists | $H(s)$ per visited state |
| **Execution** | After complete search | Immediate one step |
| **Optimality** | Yes (admissible $h$) | Asymptotic / trial-dependent |
| **Unknown graph** | Needs full model | Learns while moving |

Standard A* **RESULT** must be known; LRTA* discovers edges by **acting**.

---

## Unknown Terrain Mapping with LRTA*

Combine LRTA* with **grid exploration** ([Section 4.5](./section-05-online-search.md)):

| Phase | Behavior |
|-------|----------|
| **Initialize** | $H(s)=0$ or goal-directed $h$; unknown cells not in graph |
| **Move** | LRTA* on known free cells |
| **Discover** | After move, add new cells and edges |
| **Blocked** | Remove edge; $H$ updates propagate locally |
| **Goal found** | Terminate when $s \in G$ |

**Frontier exploration:** when no path to goal on known map, heuristic directs toward **unknown** cells (often via optimistic $H=0$ for unknown) - classic **explore vs exploit** tension ([Chapter 17](../chapter-17-making-complex-decisions/README.md) bandits formalize related ideas).

```
  S . ? ? G     Agent at S, learns right wall by bumping,
  . . # ? .     updates H, LRTA* redirects around #
```

---

## Heuristic Consistency and Convergence

If $h$ is **consistent** (triangle inequality):

$$
H(s) \leq c(s, a, s') + H(s') \quad \text{for all steps}
$$
> **Readable form:** Estimated distance to goal never overestimates the one-step progress you can actually make.

LRTA* with **initial admissible** $H$ maintains admissibility in many variants; learning can **raise** $H(s)$ to correct underestimates.

**Repeated LRTA* trials** from same start: second trial often faster - $H$ warmed up from first exploration (like **experience replay** of distances).

---

## PEAS: Mars Rover Path Planning

| PEAS | LRTA* rover |
|------|-------------|
| **P** - Performance | Reach science target; minimize energy |
| **E** - Environment | Martian terrain, partially mapped |
| **A** - Actuators | Wheels, steering |
| **S** - Sensors | Stereo vision, hazard map local |

| Constraint | LRTA* fit |
|------------|-----------|
| Communication delay | Local decisions without Earth-side full A* |
| Unknown rocks | Update graph on each move |
| Limited memory | Store $H$ only for visited states |

---

## Comparison with Other Online Methods

| Method | Learning | Typical use |
|--------|----------|-------------|
| **LRTA*** | Updates $H(s)$ | Path planning, unknown maps |
| **D* / D* Lite** | Reprioritize when map changes | Robotics with frequent updates |
| **Real-time DFS** | No heuristic learning | Maze exploration |
| **Q-learning** | Q-values per $(s,a)$ | [Chapter 19](../chapter-19-learning-from-examples/README.md) |

D* Lite ([Chapter 12](../chapter-11-automated-planning/README.md) robotics literature) scales LRTA*-style ideas to **dynamic** obstacles.

---

## Exploration vs Exploitation in Search

| Term | In mapping context |
|------|-------------------|
| **Exploitation** | Follow current best path toward $G$ |
| **Exploration** | Visit unknown to refine map / $H$ |

LRTA* with zero initial $H$ explores outward; as $H$ improves, behavior shifts toward exploitation.

**Infinite grid:** without exploration strategy, agent may loop - mark visited or use **upper confidence** style bonuses (link to RL curiosity, [Course 3](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/README.md) optional topics).

---

## Worked Example: LRTA* on 4-Node Line

Nodes $A$-$B$-$C$-$G$, costs 1 per edge. Start $A$, goal $G$. Initial $H(x)=0$ everywhere.

**At A:**

Neighbors: $B$ with $f = 1 + H(B) = 1$. Choose move to $B$. Update $H(A) \leftarrow 1$.

**At B:**

$f$ to $A$: $1 + H(A) = 1 + 1 = 2$; to $C$: $1 + 0 = 1$. Move to $C$. $H(B) \leftarrow 1$.

**At C:**

To $B$: $1+1=2$; to $G$: $1+0=1$. Move to $G$. $H(C) \leftarrow 1$.

**Goal reached** in 3 steps - optimal path.

**Second trial from A:** Now $H(B)=1$, chain propagates - same optimal choices with informed estimates.

| State | $H$ after first visit |
|-------|----------------------|
| A | 1 |
| B | 1 |
| C | 1 |
| G | 0 |

If an obstacle later blocks $B$-$C$, LRTA* at $B$ recomputes $\min f$ - may route back through $A$ and alternate path if exists.

---

## Unknown Terrain: Bump and Learn

Robot in room, **unknown** obstacles. On **bump**, learn cell blocked:

1. Remove edge from graph.
2. Set $H(\text{blocked}) = \infty$ or prune.
3. LRTA* at current cell picks next-best neighbor.

**Mapping** accumulates:

| Data structure | Content |
|----------------|---------|
| **Occupancy grid** | free / blocked / unknown |
| **$H$ table** | cost-to-go estimates |
| **Visited set** | prevent trivial loops |

After $T$ steps, map coverage grows; $H$ approximates true distances on revealed subgraph.

---

## Limitations

| Limitation | Detail |
|------------|--------|
| **Suboptimality** | First trial may wander |
| **Local minima in $H$** | Wrong estimates → wrong moves |
| **Non-stationary** | Moving obstacles need D* or replan |
| **Large state space** | Tabular $H$ impractical - function approximation ([Course 3](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/README.md)) |

---

## Connection to Other Chapters

| Topic | Chapter |
|-------|--------|
| Online search framework | [Section 4.5](./section-05-online-search.md) |
| A* and heuristics | [Section 3.4-3.5](../chapter-03-solving-problems-by-searching/section-04-informed-search.md) |
| MDP policy iteration | [Chapter 17](../chapter-17-making-complex-decisions/README.md) |
| SLAM | [Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md) |

---

## Common Mistakes

**Expecting first-trial optimality** of offline A*.

**Not updating $H(s)$ before leaving** state $s$ - learning step skipped.

**Zero heuristic forever** on large unknown map - aimless drift without goal bias.

**Confusing LRTA* with full RTA*** (deeper lookahead each step) - parameter variants differ.

---

## Reflection Questions

1. How does LRTA* differ from standard A* in when it commits to actions?
2. What does the update $H(s) \leftarrow \min_a f(s,a)$ learn about the environment?
3. Why is LRTA* well suited to unknown terrain mapping?
4. How does a second trial from the same start differ from the first?
5. When would D* Lite be preferable to basic LRTA*?

---

**Next:** [Section 4.7 - Summary and Tradeoffs](./section-07-summary-and-tradeoffs.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 4.5. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Korf, R. E. (1990). Real-time heuristic search. *Artificial Intelligence*.
3. Koenig, S., & Likhachev, M. - D* Lite. *AAAI*.
4. Russell, S., & Norvig, P. - `LRTAStar` in aima-python. [github.com/aimacode/aima-python](https://github.com/aimacode/aima-python)
5. Stentz, A. - The focussed D* algorithm for real-time replanning.



