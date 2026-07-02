# Section 3.1: Problem-Solving Agents

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 3.1-3.2  
> **Previous:** [Chapter 02 - Intelligent Agents](../chapter-02-intelligent-agents/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Goal-Based Agents to Search

[Chapter 02](../chapter-02-intelligent-agents/README.md) introduced **goal-based agents** - agents that maintain an internal goal and choose actions that move the world (or their beliefs about it) toward that goal. Chapter 3 makes that idea **algorithmic**: represent the task as a well-defined **search problem**, then apply a [search algorithm](../../GLOSSARY.md#search-algorithm) to find a sequence of actions from the initial state to a goal state.

Think of planning a road trip. You do not wander randomly until you stumble on Paris. You **formulate** the problem (start city, destination, allowed roads, cost metric), then **search** for a route. A problem-solving agent does the same, but with explicit data structures and systematic exploration instead of human intuition.

> **Readable form:** A problem-solving agent turns "I want to get there" into a precise puzzle (states, moves, costs), then uses search to find a path of moves that reaches the goal.

Humorous analogy: a problem-solving agent is like a **GPS with amnesia about geography but perfect obedience to turn-by-turn rules**. It does not "know" Romania; it knows how to expand states and compare path costs until Bucharest appears.

---

## Problem-Solving Agent Architecture

AIMA's problem-solving agent separates **deliberation** from **execution**:

```
Goal formulation → Problem formulation → Search → Execute solution
```

| Stage | Question answered | Output |
|-------|-------------------|--------|
| **Goal formulation** | What world states are desirable? | Goal description |
| **Problem formulation** | What is the abstract search space? | Five-tuple (below) |
| **Search** | Which action sequence reaches a goal? | Solution path |
| **Execute** | How do we act in the real environment? | Percepts → actions loop |

The agent program from [Section 2.4](../chapter-02-intelligent-agents/section-04-structure-of-agents.md) becomes:

```python
def problem_solving_agent(percepts, problem):
    """
    Simplified AIMA-style problem-solving agent.
    """
    if problem is None:
        problem = formulate_problem(percepts)   # from sensors + goals
    if not hasattr(problem_solving_agent, "plan"):
        problem_solving_agent.plan = search(problem)  # offline deliberation
    if problem_solving_agent.plan:
        return problem_solving_agent.plan.pop(0)      # next action
    return "NO_OP"  # replan if environment changed
```

> **Readable form:** Once per task, build the problem and search for a plan; then repeatedly pop the next action from the plan until done or replanning is needed.

**Replanning** matters when the environment is dynamic or partially observable ([Section 2.3](../chapter-02-intelligent-agents/section-03-the-nature-of-environments.md)). Chapter 3 assumes **fully observable, deterministic, static** worlds unless noted - the cleanest setting for classical search.

---

## Goal Formulation

Before search, the agent must decide **what counts as success**. Goals can be:

| Goal type | Example | Representation |
|-----------|---------|----------------|
| **State goal** | "Be in Bucharest" | Explicit goal state or set |
| **Condition goal** | "All tiles in correct positions" | Predicate on states |
| **Optimization goal** | "Minimize travel cost" | Implicit via path cost $g(n)$ |

Goals come from the [performance measure](../../GLOSSARY.md#performance-measure) in the [PEAS](../../GLOSSARY.md#peas) specification. A delivery robot's performance measure might reward on-time arrival and penalize distance; the **operational goal** becomes "reach customer location with minimum expected cost."

### PEAS for a Route-Finding Agent

| PEAS | Route finding (Romania) |
|------|-------------------------|
| **P** - Performance | Minimize driving distance/time; reach destination |
| **E** - Environment | Map, traffic (ignored in basic model), fuel |
| **A** - Actuators | Steer, accelerate, brake (abstracted as "move to neighbor city") |
| **S** - Sensors | GPS, odometer (abstracted as "know current city") |

From [Section 2.3](../chapter-02-intelligent-agents/section-03-the-nature-of-environments.md): this environment is **fully observable, single-agent, deterministic, sequential, static, discrete** - ideal for graph search.

---

## Problem Formulation: The Five Components

Russell and Norvig define a search problem with **five components**:

1. **Initial state** $s_0$ - where the agent starts.
2. **Actions** $\text{ACTIONS}(s)$ - applicable actions in state $s$.
3. **Transition model** $\text{RESULT}(s, a)$ - state reached after action $a$ in $s$.
4. **Goal test** - whether state $s$ satisfies the goal.
5. **Path cost function** $c(s, a, s')$ - step cost; total path cost is sum of step costs.

A **solution** is a sequence of actions $\langle a_1, a_2, \ldots, a_n \rangle$ such that applying them from $s_0$ reaches a goal state. An **optimal solution** minimizes total path cost among all solutions.

$$
\text{PathCost}(\pi) = \sum_{i=1}^{n} c(s_{i-1}, a_i, s_i)
$$
> **Readable form:** Total path cost = sum of each step's cost along the path from start to goal.

### States vs. World States vs. Search Nodes

| Term | Meaning |
|------|---------|
| **World state** | Complete configuration of the environment |
| **State (search)** | Abstract representation sufficient for problem solving |
| **Node (search tree)** | Data structure: state + parent + action + path cost $g$ |

Abstraction is a design choice. For route finding, a state might be **current city** (ignoring driver fatigue). For the 8-puzzle, a state is usually the **tile configuration** on a 3×3 grid.

---

## State Space and Search Spaces

The **state space** is the set of all states reachable from $s_0$ via $\text{ACTIONS}$ and $\text{RESULT}$. Its size often grows **exponentially** with problem depth - the motivation for smart search strategies in later sections.

If branching factor is $b$ (average actions per state) and solution depth is $d$, a naive brute-force search might examine $O(b^d)$ paths. That is why **informed search** ([Section 3.4](./section-04-informed-search.md)) and **good heuristics** ([Section 3.5](./section-05-heuristic-functions.md)) matter.

```
                    s0
           /        |        \
         s1         s2         s3
        /|\        /|\        /|\
      ... ...    ... ...    ... ...
```

Each edge is an action; each node is a state. **Search** is graph or tree exploration of this structure.

---

## Search vs. Exploration vs. Planning

| Term | Emphasis |
|------|----------|
| **Search** | Find a path in a known state space (Chapter 3) |
| **Exploration** | Map an unknown environment while acting (Chapter 4) |
| **Planning** | Often richer action models, time, resources (Chapter 11) |

Chapter 3 search assumes the **transition model is known** - the agent has a map, puzzle rules, or simulator. Learning the model from experience is [Chapter 19](../chapter-19-learning-from-examples/README.md) territory; acting without a full map is [Chapter 04](../chapter-04-search-complex-environments/README.md).

---

## Formulation Examples (Preview)

[Section 3.2](./section-02-example-problems.md) develops two canonical problems in full:

**Romania route finding:** states = cities; actions = drive to adjacent city; step cost = road distance; goal = Bucharest.

**8-puzzle:** states = tile permutations; actions = slide blank Up/Down/Left/Right; step cost = 1 per move; goal = target configuration.

Both share the same five-tuple pattern - only the state encoding and actions change.

---

## Measuring Solution Quality

The [rational agent](../../GLOSSARY.md#rational-agent) framework from [Section 2.2](../chapter-02-intelligent-agents/section-02-good-behavior-and-rationality.md) asks: does the agent maximize expected performance?

For problem-solving with known dynamics and costs, rationality often reduces to finding **lowest-cost goal paths** (or any goal path if all costs equal). When multiple goals exist, the agent may need **preferences** over goals - utility-based agents handle that in [Section 2.4](../chapter-02-intelligent-agents/section-04-structure-of-agents.md).

| Criterion | Question |
|-----------|----------|
| **Completeness** | Does the algorithm find a solution if one exists? |
| **Optimality** | Does it find minimum-cost solution? |
| **Time complexity** | How many nodes expanded? |
| **Space complexity** | How much memory for the frontier/explored set? |

[Section 3.7](./section-07-search-analysis.md) compares algorithms on these dimensions.

---

## Connection to Course 1

| Course 1 idea | Search framing |
|---------------|----------------|
| Feature vector | State encoding (e.g., puzzle board as tuple) |
| Loss function | Path cost or heuristic error |
| Train/validation split | Not direct - search is **planning**, not learning from labels |
| Graph algorithms (implicit) | BFS/DFS are foundational graph traversals |

Search is **symbolic AI** in its pure form: explicit states and rules, not learned weights. Later chapters combine search with learning (e.g., learned heuristics, MCTS in [Chapter 05](../chapter-05-adversarial-search-games/README.md)).

---

## Common Formulation Mistakes

**Wrong state granularity.** Including irrelevant details (exact GPS coordinates to the millimeter) explodes the state space. Including too little (only "in Romania") loses ability to distinguish progress.

**Non-markovian state.** If actions depend on history not captured in $s$, the formulation is incomplete - embed history into state or use a richer model.

**Inconsistent costs.** Negative cycles make "shortest path" ill-defined. Zero-cost loops cause infinite redundant paths unless cycle checking is used.

**Goal not testable.** "Drive safely" is not a goal test; "arrive at Bucharest" is.

---

## Worked Mini-Example: Vacuum World as Search

Two rooms {A, B}, dirt in known locations, agent starts in A facing right.

| Component | Value |
|-----------|-------|
| $s_0$ | (location=A, dirt=(A:yes, B:yes)) |
| $\text{ACTIONS}(s)$ | {Left, Right, Suck} |
| $\text{RESULT}$ | Updates location or cleans current room |
| Goal test | No dirt in any room |
| $c$ | 1 per action |

Solution: $\langle \text{Suck}, \text{Right}, \text{Suck} \rangle$ - three steps. This tiny world previews BFS in [Section 3.3](./section-03-uninformed-search.md).

---

## Reflection Questions

1. Write PEAS for an 8-puzzle-solving agent. Which environment properties from [Section 2.3](../chapter-02-intelligent-agents/section-03-the-nature-of-environments.md) apply?
2. Why is "current city" a sufficient state for basic Romania route finding but insufficient for real GPS navigation with traffic?
3. What is the difference between the agent's **plan** and the search algorithm's **frontier**?
4. If all step costs are 1, what performance measure aligns with minimizing path cost?
5. When would a problem-solving agent need to discard its plan and reformulate?

---

**Next:** [Section 3.2 - Example Problems](./section-02-example-problems.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Sections 3.1-3.2. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - AIMA Python implementations. [aima-python](https://github.com/aimacode/aima-python)
3. Pearl, J. (1984). *Heuristics* - problem formulation and search spaces. [Author publications](https://ftp.cs.ucla.edu/pub/stat_ser/r119.pdf)
4. Korf, R. E. - state-space search complexity. [Research overview](https://www.aaai.org/)
5. MIT 6.034 - Classical search lectures. [OpenCourseWare](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)



