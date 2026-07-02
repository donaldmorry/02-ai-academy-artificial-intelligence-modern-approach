# Section 4.1: Local Search

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 4.1  
> **Previous:** [Chapter 03 - Solving Problems by Searching](../chapter-03-solving-problems-by-searching/README.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## When the Path Does Not Matter

[Chapter 03](../chapter-03-solving-problems-by-searching/README.md) found **action sequences** from start to goal. Many tasks care only about the **final configuration**: n-queens placement, factory floor layout, VLSI chip wiring, course scheduling. These are **optimization problems** - find a state $s^*$ that maximizes $f(s)$ or minimizes cost $c(s)$.

**Local search** keeps one or a few current states and repeatedly moves to neighboring states. Memory is $O(1)$ per current state instead of $O(b^d)$ for a search frontier.

> **Readable form:** Local search hill-walks through the state space looking for a peak (or valley), not reconstructing a path from root to goal.

| Paradigm | Remembers | Returns | Typical use |
|----------|-----------|---------|-------------|
| Tree/graph search | Frontier + explored set | Action sequence | Shortest path, planning |
| Local search | Current state(s) | Goal state / best found | CSPs, scheduling, layout |

Humorous analogy: systematic search is **exploring every hallway with breadcrumbs**; local search is **hill-walking in thick fog with amnesia** - you only know if the ground slopes up under your feet.

---

## Optimization Formulation

1. **State space** - all complete candidate configurations
2. **Objective** $f(s)$ - higher is better, or **cost** $c(s)$ for minimization
3. **Neighbors** - states reachable by one local move

For **8-queens** in complete-state form (one queen per column):

$$
h(s) = \text{number of attacking queen pairs}
$$
> **Readable form:** Count pairs of queens that threaten each other; zero means a solution.

Russell and Norvig report that **steepest-ascent hill climbing** from random 8-queens starts gets stuck **86%** of the time, yet successful runs average only **4 steps**.

---

## State-Space Landscape

| Feature | Effect |
|---------|--------|
| **Local maximum** | Peak below global; greedy ascent stops |
| **Ridge** | Ascending local maxima; sideways moves needed |
| **Plateau** | Flat region; no uphill gradient |
| **Global optimum** | Best achievable value |

---

## Hill-Climbing Variants

```python
def hill_climbing(problem):
  current = problem.initial()
  while True:
    neighbor = max(problem.neighbors(current), key=problem.value)
    if problem.value(neighbor) <= problem.value(current):
      return current
    current = neighbor
```

| Variant | Behavior |
|---------|----------|
| **Steepest ascent** | Pick best neighbor |
| **Stochastic** | Random uphill neighbor |
| **First-choice** | Generate random neighbors until improvement |
| **Random-restart** | Repeat from random starts |

**Random-restart** is complete in the limit: expected restarts until success is $1/p$ if each run succeeds with probability $p$. For 8-queens, success rises from 14% to **94%** in AIMA experiments.

---

## Simulated Annealing

Hill climbing never moves **downhill**. **Simulated annealing** occasionally accepts worse states to escape local optima.

$$
P(\text{accept}) = \begin{cases} 1 & \text{if } E(s') \leq E(s) \\ e^{-(E(s') - E(s))/T} & \text{otherwise} \end{cases}
$$
> **Readable form:** Always accept downhill moves; uphill moves accepted with probability that shrinks as cost gap grows or temperature falls.

**Schedule:** start with high temperature $T$, cool toward zero. Poor schedules freeze in bad states; good schedules balance exploration and exploitation.

---

## Local Beam Search and Genetic Algorithms

**Local beam search** keeps $k$ states; each step pools all successors and retains the best $k$. Risk: **diversity collapse** - all states converge to same local max. **Stochastic beam** mitigates.

**Genetic algorithms** maintain a **population** with **selection**, **crossover**, and **mutation**. Fitness = $f(s)$. Excel when good **building blocks** combine (partial solutions).

---

## Complete-State vs. Incremental

| Formulation | State | Example |
|-------------|-------|---------|
| **Incremental** | Partial assignment | 3 queens placed of 8 |
| **Complete-state** | Full candidate | All columns filled |

Local search almost always uses **complete-state** - every neighbor is evaluable. [Chapter 06](../chapter-06-constraint-satisfaction/README.md) backtracking often uses incremental assignments.

---

## PEAS: 8-Queens Local Search

| PEAS | 8-queens |
|------|----------|
| **P** | Minimize attacking pairs ($h = 0$) |
| **E** | Chess board; static |
| **A** | Move one queen within its column |
| **S** | Full board (fully observable) |

---

## Worked Example: Tiny Landscape

States $\{A,B,C,D\}$, values $f(A)=1, f(B)=3, f(C)=2, f(D)=5$. Chain $A$-$B$-$C$-$D$.

Steepest ascent from $A$: stops at local max $B(3)$, missing global max $D(5)$. Random restart from $D$ succeeds immediately.

---

## Implementation Notes

- **Neighbor generation** dominates runtime - cache conflict counts for n-queens (update incrementally).
- **Sideways moves** on plateaus: allow equal-value steps with a cap.
- **Temperature schedule** for SA: $T_t = T_0 / \log(1+t)$ or geometric cooling $T_t = \alpha T_{t-1}$.

---

## Connection to Later Chapters

| Link | Idea |
|------|------|
| [Section 4.2](./section-02-continuous-spaces.md) | Gradient descent on continuous objectives |
| [Chapter 06](../chapter-06-constraint-satisfaction/section-06-local-search-for-csps.md) | Min-conflicts for CSPs |
| Course 3 | Neural training = high-dimensional local search |

---

## Genetic Algorithm Sketch (8-Queens)

```python
def genetic_n_queens(n, pop_size=100, generations=500):
  population = [random_assignment(n) for _ in range(pop_size)]
  for _ in range(generations):
    population.sort(key=lambda s: -fitness(s))  # higher = fewer attacks
    parents = population[:pop_size // 2]
    offspring = []
    for i in range(0, len(parents), 2):
      c1, c2 = crossover(parents[i], parents[i+1])
      offspring.extend([mutate(c1), mutate(c2)])
    population = parents + offspring
  return min(population, key=conflicts)
```

**Crossover:** splice column segments between two parent vectors. **Mutation:** change one queen's row. Population diversity prevents single-state stagnation.

---

## AIMA Statistics (8-Queens)

| Method | Success rate (random start) | Avg steps if success |
|--------|----------------------------|----------------------|
| Steepest-ascent hill climbing | 14% | 4 |
| Random-restart hill climbing | 94% | ~4 per successful run |
| Simulated annealing | High with good schedule | Varies |

These numbers explain why **restarts** and **stochastic** methods are standard on combinatorial landscapes.

---

## When Local Search Wins (and Loses)

**Wins:** huge state spaces where any solution suffices; memory constraints; landscapes with helpful structure; combining with restarts or SA.

**Loses:** need **proven optimal** path or cost; tightly constrained problems without propagation; NP-hard landscapes with exponentially many local optima.

---

## Common Mistakes

- Using local search when the **path** must be optimal (route planning).
- No restarts on NP-hard landscapes with exponential local optima.
- Confusing **objective** $h$ with A* **heuristic** - local search maximizes/minimizes state value, not path estimate.

---

## Reflection Questions

1. Why does local search use $O(1)$ memory per state while BFS uses $O(b^d)$?
2. How does simulated annealing's acceptance rule change as $T \to 0$?
3. What problem does random-restart solve that plain hill climbing cannot?
4. Contrast local beam search with $k$ independent hill climbs.
5. When is a genetic algorithm preferable to simulated annealing?

6. How do AIMA's 8-queens success rates motivate random-restart hill climbing?

---

## Study Checklist

Before moving on, you should be able to explain: complete-state vs. incremental formulation; why plateaus and ridges defeat greedy ascent; the role of temperature in simulated annealing; and when local search beats systematic graph search.

---

**Next:** [Section 4.2 - Continuous Spaces](./section-02-continuous-spaces.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 4.1. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - `search.py` local search. [aima-python](https://github.com/aimacode/aima-python)
3. Kirkpatrick, S., Gelatt, C. D., & Vecchi, M. P. (1983). Optimization by simulated annealing. *Science*.
4. Hoos, H. H., & Stützle, T. - stochastic local search. [SLS book](https://www.stochastic-local-search.eu/)
5. MIT 6.034 - Local search lectures. [OCW](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)
