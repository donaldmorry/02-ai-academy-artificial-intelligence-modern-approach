# Section 4.7: Summary and Tradeoffs

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 4 (synthesis)  
> **Previous:** [Section 4.6 - LRTA* and Exploration](./section-06-lrta-and-exploration.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## When Classical Search Assumptions Break Down

[Chapter 03](../chapter-03-solving-problems-by-searching/README.md) built a powerful toolkit - BFS, A*, heuristics - on a **clean model**:

| Classical assumption | Meaning |
|---------------------|---------|
| **Fully observable** | Agent knows current state $s$ |
| **Deterministic** | $\text{RESULT}(s,a)$ is unique |
| **Static** | World changes only when agent acts |
| **Discrete** | Finite (or tractably enumerable) states |
| **Known model** | Transition and cost functions given |
| **Offline** | Plan completely, then execute |

**Chapter 04** relaxed each assumption. This section synthesizes **when** classical search applies and **what** to use when it does not.

> **Readable form:** Classical search is a perfect subway map with live GPS; real agents often have fog, ice, construction, and incomplete maps - different tools for each kind of mess.

Humorous analogy: classical search is **chess with full board view and fixed rules**; complex environments are **chess on a wet napkin, in the dark, while the pieces slide on their own** - you pick algorithms matching which part of the nightmare you face.

---

## Chapter 04 Roadmap

| Section | Relaxed assumption | Core tools |
|--------|-------------------|------------|
| [4.1 Local search](./section-01-local-search.md) | Path may not matter | Hill climbing, SA, GA |
| [4.2 Continuous](./section-02-continuous-spaces.md) | Discrete states | Discretization, gradient descent, LP |
| [4.3 Nondeterministic](./section-03-nondeterministic-search.md) | Deterministic outcomes | AND-OR, contingency, sensorless |
| [4.4 Partial observability](./section-04-partial-observability.md) | Full observability | Belief states, $2^{|S|}$ complexity |
| [4.5 Online](./section-05-online-search.md) | Known model, offline | Interleaving, competitive ratio |
| [4.6 LRTA*](./section-06-lrta-and-exploration.md) | Known map | Learning heuristics, mapping |

---

## Decision Guide: Which Formulation?

```
Start: problem for a goal-based agent
  |
  v
Is the FINAL STATE enough (no path cost structure)?
  |-- YES --> Local search ([Section 4.1](./section-01-local-search.md))
  |-- NO  --> continue
  |
  v
Is state space continuous?
  |-- YES --> Discretize / gradient / LP ([Section 4.2](./section-02-continuous-spaces.md))
  |-- NO  --> continue
  |
  v
Are actions nondeterministic?
  |-- YES --> AND-OR / MDP ([Section 4.3](./section-03-nondeterministic-search.md), [Chapter 17](../chapter-17-making-complex-decisions/README.md))
  |-- NO  --> continue
  |
  v
Is state fully observable?
  |-- NO  --> Belief-state / POMDP ([Section 4.4](./section-04-partial-observability.md))
  |-- YES --> continue
  |
  v
Is the model known before acting?
  |-- NO  --> Online / LRTA* ([Sections 4.5-4.6](./section-05-online-search.md))
  |-- YES --> Chapter 03 graph search (A*, etc.)
```

---

## Breaking Down Each Assumption

### Fully observable → Partial observability

**Breaks:** Agent no longer searches in $S$ - searches in **belief space** ([Section 4.4](./section-04-partial-observability.md)).

**Symptom:** Plan assumes "at cell $(2,3)$" but true state uncertain - execution mismatch.

**Fix:** Belief-state search, POMDP, active sensing, localization ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)).

**Complexity:** Up to $2^{|S|}$ belief nodes.

---

### Deterministic → Stochastic

**Breaks:** Unique $\text{RESULT}$ - need **contingency** or **expected cost** ([Section 4.3](./section-03-nondeterministic-search.md)).

**Symptom:** Plan succeeds in simulator, fails on slippery floor.

**Fix:** AND-OR trees, MDP value iteration ([Chapter 17](../chapter-17-making-complex-decisions/README.md)), policy not sequence.

---

### Static → Dynamic

**Breaks:** Precomputed path invalid mid-execution.

**Symptom:** Traffic jam, moving obstacles, other agents.

**Fix:** **Replanning**, D* Lite ([Section 4.6](./section-06-lrta-and-exploration.md)), online search ([Section 4.5](./section-05-online-search.md)), multi-agent ([Chapter 05](../chapter-05-adversarial-search-games/README.md) if adversarial).

---

### Discrete → Continuous

**Breaks:** BFS/A* neighbor enumeration infinite ([Section 4.2](./section-02-continuous-spaces.md)).

**Symptom:** Joint angles, velocities - no finite branching.

**Fix:** Discretization, gradient methods, motion planning ([Chapter 12](../chapter-11-automated-planning/README.md)).

---

### Known model → Unknown

**Breaks:** Cannot run offline A* on full graph ([Section 4.5](./section-05-online-search.md)).

**Symptom:** Robot enters unmapped building.

**Fix:** Explore while acting; LRTA*; SLAM ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)).

**Cost:** Competitive ratio $> 1$ - exploration tax.

---

### Offline → Online

**Breaks:** Deliberation must fit between actions.

**Symptom:** Game clock, real-time robot safety.

**Fix:** Anytime algorithms, bounded lookahead, LRTA*.

---

## Master Comparison Table

| Method | Observability | Determinism | Model | Optimality | Memory |
|--------|---------------|-------------|-------|------------|--------|
| **BFS/A*** ([Mod 03](../chapter-03-solving-problems-by-searching/README.md)) | Full | Yes | Known | Yes* | $O(b^d)$ |
| **Local search** ([4.1](./section-01-local-search.md)) | Full | Yes | Known | Local opt. | $O(1)$ |
| **Gradient/LP** ([4.2](./section-02-continuous-spaces.md)) | Full | Yes | Known | Convex/LP | $O(n)$ |
| **AND-OR** ([4.3](./section-03-nondeterministic-search.md)) | Full | No | Known | Contingent | Plan size |
| **Belief search** ([4.4](./section-04-partial-observability.md)) | Partial | Varies | Known | Belief goal | $O(2^n)$ |
| **Online/LRTA*** ([4.5-4.6](./section-05-online-search.md)) | Varies | Varies | Unknown | Competitive | Visited |

*With admissible heuristic for A*.

---

## PEAS: Choosing an Architecture

| PEAS | Classical A* | Belief + online |
|------|--------------|-----------------|
| **P** | Shortest path | Reach goal reliably under uncertainty |
| **E** | Static map, full GPS | Unknown, partial sensors |
| **A** | Move | Move + optional sense |
| **S** | Exact position | Noisy / local |

Match algorithm to **E** and **S** columns first - performance **P** is unachievable with wrong model class.

---

## Worked Example: Same Robot, Four Formulations

**Task:** Deliver package in a building.

| Scenario | Formulation | Algorithm |
|----------|-------------|-----------|
| **A** - CAD map, perfect GPS | Chapter 03 five-tuple | A* |
| **B** - Map, slippery floors | AND-OR or MDP | Contingency / VI |
| **C** - Map, GPS noise | Belief-state search | POMDP approximate |
| **D** - No map | Online + mapping | LRTA* + grid learn |

**Scenario D cost:** First trip explores dead ends; second trip with learned $H$ approaches A* cost on revealed graph ([Section 4.6](./section-06-lrta-and-exploration.md)).

| Trip | Actions | Notes |
|------|---------|-------|
| 1 | 45 | Discovery |
| 2 | 28 | Near shortest on known subgraph |
| Oracle (full map) | 25 | Lower bound |

Competitive ratio $\approx 45/25 = 1.8$ for this instance.

---

## Connections Across the Course

| Chapter 04 topic | Later formalization |
|-----------------|---------------------|
| Nondeterminism | [Chapter 17 - MDPs](../chapter-17-making-complex-decisions/README.md) |
| Partial observability | POMDPs, HMMs ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)) |
| Local search | [Chapter 06 - CSPs](../chapter-06-constraint-satisfaction/README.md), training ([Course 3](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/README.md)) |
| Online exploration | [Chapter 19 - RL](../chapter-19-learning-from-examples/README.md) |
| Adversarial dynamics | [Chapter 05 - Games](../chapter-05-adversarial-search-games/README.md) |
| Planning | [Chapter 12](../chapter-11-automated-planning/README.md) STRIPS, HTN |

---

## What We Still Assume (Usually)

Even Chapter 04 often keeps:

- **Single agent** (no adversary) unless [Chapter 05](../chapter-05-adversarial-search-games/README.md)
- **Markov state** (or belief sufficient)
- **Goal specified** explicitly
- **Computational feasibility** of tabular methods on small instances

Dropping these leads to **game theory**, **multi-agent POMDPs**, and **bounded rationality** ([Section 2.2](../chapter-02-intelligent-agents/section-02-good-behavior-and-rationality.md)).

---

## Practical Engineering Checklist

Before coding a search agent, ask:

1. **Do I need the path or only the destination?** → Local vs graph search.
2. **Is my state vector continuous?** → Discretize or optimize.
3. **Can actions fail?** → Stochastic model.
4. **Do I know where I am?** → Belief state.
5. **Do I know the map?** → Online exploration.
6. **How much time per decision?** → Offline vs real-time.
7. **Is the environment changing?** → Replanning frequency.

---

## Common Integration Patterns

| Pattern | Components |
|---------|------------|
| **Global + local** | Coarse A* + continuous smoother ([Section 4.2](./section-02-continuous-spaces.md)) |
| **Plan + execute + monitor** | Offline plan, online replan on surprise |
| **Explore then exploit** | LRTA* trial 1 map, trial 2 deliver |
| **Search + learn** | Heuristic from past trips ([Chapter 19](../chapter-19-learning-from-examples/README.md)) |

---

## Reflection Questions

1. List the six classical search assumptions and give one real-world violation for each.
2. Why does belief-state search not reduce to ordinary A* on a larger graph without exponential blowup?
3. When is local search preferable to A* even if the environment is fully observable and static?
4. Explain exploration cost in online search using the competitive ratio idea.
5. Which Chapter 04 section would you extend first for a drone with GPS, wind, and no prior map - and why?

---

**Next:** [Chapter 05 - Adversarial Search and Games](../chapter-05-adversarial-search-games/README.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Chapter 4. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - aima-python Chapter 4 notebooks. [github.com/aimacode/aima-python](https://github.com/aimacode/aima-python)
3. Korf, R. E. - complexity of search - surveys.
4. LaValle, S. M. - *Planning Algorithms* - integration of search variants.
5. MIT 6.034 - Problem solving with uncertainty and sensing. [OCW](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)