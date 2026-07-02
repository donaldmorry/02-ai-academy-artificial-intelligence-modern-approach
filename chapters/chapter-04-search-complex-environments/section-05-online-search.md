# Section 4.5: Online Search

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 4.5  
> **Previous:** [Section 4.4 - Partial Observability](./section-04-partial-observability.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Planning While Acting

[Chapter 03](../chapter-03-solving-problems-by-searching/README.md) **offline search** computes a full solution before the first action. The agent then executes blindly until replanning. **Online search** **interleaves computation and action**: observe, think a little, act, repeat.

```
Offline:  [======== SEARCH ========] → execute → execute → execute
Online:   think → act → think → act → think → act → ...
```

> **Readable form:** The agent does not wait for a complete map of the future - it takes one step, learns what it can, and searches again from where it actually is.

Humorous analogy: offline search is **memorizing turn-by-turn directions before leaving home**; online search is **checking your phone at every intersection** - slower per decision, but you won't drive into a newly closed road.

This matters when the environment is **unknown** or **changing** - assumptions from [Section 3.1](../chapter-03-solving-problems-by-searching/section-01-problem-solving-agents.md) fail.

---

## Online Search Agents

An **online search agent** maintains:

| Component | Role |
|-----------|------|
| **Current state** $s$ (or belief $b$) | Where am I now? |
| **Visited / explored map** | What have I learned? |
| **Goal test** | Am I done? |
| **Search procedure** | Choose next action from limited lookahead |

```python
def online_search_agent(problem):
  s = problem.initial
  while not problem.is_goal(s):
    action = select_action(problem, s)   # local search / LRTA*
    s = problem.execute(action)          # real world + percept
    problem.update_map(s, action)        # learn transitions
  return s
```

**Interleaving** bounds deliberation time per step - critical for robotics and games with clocks ([Chapter 05](../chapter-05-adversarial-search-games/README.md)).

---

## Offline vs Online

| Dimension | Offline | Online |
|-----------|---------|--------|
| **Map** | Known upfront | Discovered or partial |
| **Commitment** | Full plan | Next action(s) only |
| **Cost model** | Path cost in graph | Action cost + **search cost** |
| **Optimality** | Optimal in model | Competitive ratio vs clairvoyant |
| **Examples** | A* on Romania map | Robot exploring building |

**Replanning** bridges the two: offline plan, execute until surprise, search again ([Section 4.4](./section-04-partial-observability.md) belief updates trigger replans).

---

## Competitive Analysis

How good is an online algorithm compared to an **offline optimal** algorithm that knows the full graph?

**Competitive ratio** for cost:

$$
c_{\text{online}}(\text{problem}) \leq k \cdot c_{\text{offline}}(\text{optimal})
$$
> **Readable form:** Online cost is at most $k$ times the best possible cost if you had known the entire world in advance.

| Algorithm | Competitive ratio (typical settings) |
|-----------|--------------------------------------|
| **Depth-first online exploration** | Can be exponential in $|V|$ vs shortest path |
| **Greedy nearest unvisited** | Poor on adversarial graphs |
| **LRTA*** ([Section 4.6](./section-06-lrta-and-exploration.md)) | Depends on heuristic; bounded under conditions |

**Competitive analysis** is worst-case over **problem instances** (graph layouts), not average-case - analogous to adversarial thinking in [Chapter 05](../chapter-05-adversarial-search-games/README.md).

For **exploration** of unknown graphs, an agent must sometimes visit dead ends to discover structure - any online algorithm pays **exploration cost** no offline oracle pays on a known map.

---

## Mapping Unknown Terrain

**Unknown terrain mapping:** grid cells start **unknown**; moving reveals neighbors (free or blocked). Goal: reach target $G$.

| Cell label | Meaning |
|------------|---------|
| **Free** | Traversable (observed) |
| **Blocked** | Obstacle (observed) |
| **Unknown** | Not yet sensed |

Online agent plans through **known free** cells into **unknown** frontier - must eventually **explore** to find a path or prove none exists.

```
  S * ? ?        S = start, G = goal, ? = unknown
  . . ? G        . = known free, # = known blocked
  . # ? .
```

**Interleaved loop:**

1. Run A* (or BFS) on currently known subgraph toward $G$ (or toward nearest unknown).
2. Execute first step of plan.
3. Observe new cells; update map.
4. Repeat.

Preview: [Section 4.6](./section-06-lrta-and-exploration.md) uses **LRTA*** for efficient local updates.

---

## PEAS: Cave-Exploring Robot

| PEAS | Online mapping |
|------|----------------|
| **P** - Performance | Reach goal; minimize total travel + time thinking |
| **E** - Environment | Cave, unknown layout, static |
| **A** - Actuators | Move forward, turn |
| **S** - Sensors | Lidar (local free space) |

| Property | Classification |
|----------|----------------|
| Observability | Partial → improves with motion |
| Determinism | Deterministic motion |
| Search | Online interleaved |

---

## Time and Action Costs

Real agents pay for **thinking**:

$$
\text{Total cost} = \sum_i c_{\text{action}}(a_i) + \sum_j c_{\text{compute}}(\text{search}_j)
$$
> **Readable form:** Bill both footsteps and CPU cycles - rushing into bad moves wastes action cost; overthinking wastes time.

| Strategy | Trade-off |
|----------|-----------|
| **Anytime search** | Better plan if more time |
| **Bounded lookahead** | Fixed compute per step |
| **Incremental A*** | Reuse previous search tree |

[Section 5.4](../chapter-05-adversarial-search-games/section-04-imperfect-decisions.md) **iterative deepening** shares the anytime spirit for games.

---

## Online DFS and Exploration

**Online depth-first exploration** systematically visits nodes in unknown graphs:

- Mark current node visited.
- Choose unvisited neighbor; move there.
- Backtrack when dead end.

Guarantees **complete exploration** of finite connected unknown graph but may follow **very long** paths before reaching goal.

| Property | Online DFS |
|----------|------------|
| Completeness (explore all) | Yes (finite graph) |
| Path to goal | Not shortest |
| Memory | $O(|V|)$ visited |

---

## Worked Example: Small Unknown Grid

3×3 grid, start $S$ at (0,0), goal $G$ at (2,2). Unknown walls revealed on entry.

**Known after step 0:** only (0,0) free.

**Iteration 1:** Plan on known cells - only (0,0). Must move to reveal neighbor → say **Right** to (1,0).

**Iteration 2:** (1,0) free; see (2,0) unknown. A* on known: move Right toward frontier.

**Iteration 3:** Discover (2,0) blocked. Replan: go Up then Right around wall.

| Step | Known cells | Next action | Note |
|------|-------------|-------------|------|
| 0 | $\{(0,0)\}$ | Right | explore |
| 1 | $\{(0,0),(1,0)\}$ | Right | hit wall at (2,0) |
| 2 | + blocked (2,0) | Up | replan |
| 3 | ... | ... | toward $G$ |

**Offline oracle** with full map might take 4 moves; online pays extra for discovering (2,0) is blocked - competitive ratio $> 1$.

---

## When Online Search Wins

| Scenario | Why online |
|----------|------------|
| **Map too large to store** | Learn local subgraph |
| **Environment changes** | Stale offline plan invalid |
| **Partial observability** | Must act to perceive ([Section 4.4](./section-04-partial-observability.md)) |
| **Real-time deadline** | Cannot finish full A* before moving |

| Scenario | Prefer offline |
|----------|----------------|
| **Static known map** | A* optimal ([Section 3.4](../chapter-03-solving-problems-by-searching/section-04-informed-search.md)) |
| **Contingency tree small** | AND-OR once ([Section 4.3](./section-03-nondeterministic-search.md)) |

---

## Connection to Other Chapters

| Topic | Chapter |
|-------|--------|
| LRTA* algorithm | [Section 4.6](./section-06-lrta-and-exploration.md) |
| MDP acting while learning | [Chapter 17](../chapter-17-making-complex-decisions/README.md) |
| Reinforcement learning | [Chapter 19](../chapter-19-learning-from-examples/README.md) |
| Real-time games | [Chapter 05](../chapter-05-adversarial-search-games/README.md) |

---

## Common Mistakes

**Assuming offline optimal cost is achievable online** without exploration penalty.

**Never replanning** after new percepts - plan becomes invalid.

**Unbounded deliberation** per step - robot freezes while computing full A* on growing map.

**Ignoring competitive ratio** - algorithm that works on open maps may fail on maze-like adversarial layouts.

---

## Reflection Questions

1. What does it mean to interleave computation and action?
2. Define competitive ratio in your own words. Why is it greater than 1 for unknown mazes?
3. When does online search require visiting cells that are not on the final shortest path?
4. How does unknown terrain mapping change the state of the search problem over time?
5. Compare online search to contingency planning from [Section 4.3](./section-03-nondeterministic-search.md).

---

**Next:** [Section 4.6 - LRTA* and Exploration](./section-06-lrta-and-exploration.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 4.5. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Korf, R. E. - real-time search. *Artificial Intelligence*.
3. Russell, S., & Norvig, P. - `search.py` online agents. [aima-python](https://github.com/aimacode/aima-python)
4. Sloan, R. H. - competitive analysis of online algorithms.
5. MIT 6.034 - Online search and exploration. [OCW](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)
