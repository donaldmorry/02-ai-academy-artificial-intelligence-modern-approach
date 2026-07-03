# Section 4.2: Continuous Spaces

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 4.2  
> **Previous:** [Section 4.1 - Local Search](./section-01-local-search.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Beyond Discrete Grids

[Section 4.1](./section-01-local-search.md) and [Chapter 03](../chapter-03-solving-problems-by-searching/README.md) often assume **finite, discrete** state spaces - cities on a map, tiles on a board. Real robots, drones, and factory arms live in **continuous** spaces: positions in $\mathbb{R}^2$ or $\mathbb{R}^3$, joint angles on $[0, 2\pi)$, velocities in $\mathbb{R}^n$.

A continuous state is a vector $\mathbf{x} \in \mathbb{R}^n$. Actions may be continuous too (throttle, steering angle). The transition model is often a differential equation or smooth simulator rather than a lookup table.

> **Readable form:** Instead of finitely many labeled states, the agent moves through a smooth space of real-valued coordinates - infinitely many points between any two configurations.

Humorous analogy: discrete search is **choosing exits at numbered highway interchanges**; continuous search is **steering a canoe on a lake** - you can point anywhere, but you cannot try "every possible angle" one by one.

---

## Why Continuous Spaces Are Hard

| Challenge | Consequence |
|-----------|-------------|
| **Infinite states** | Cannot enumerate neighbors |
| **No guaranteed global optimum** | Gradient methods find local optima |
| **Numerical error** | Floating-point drift in simulation |
| **Constraints** | Feasible region may be curved or disconnected |

Classical BFS and A* from [Section 3.3](../chapter-03-solving-problems-by-searching/section-03-uninformed-search.md) require discrete successors. Continuous problems need **discretization**, **analytic optimization**, or **sampling**.

---

## Strategy 1: Discretization

**Discretization** maps continuous coordinates to a finite grid or mesh so standard search applies.

For a 2D robot workspace with $x \in [0, 10]$, $y \in [0, 10]$:

$$
x_{\text{grid}} = \lfloor x / \Delta \rfloor, \quad y_{\text{grid}} = \lfloor y / \Delta \rfloor
$$
> **Readable form:** Divide space into cells of width $\Delta$; each cell is one search state.

| $\Delta$ | States (approx.) | Trade-off |
|----------|------------------|-----------|
| 1.0 m | $10 \times 10 = 100$ | Fast search, coarse paths |
| 0.1 m | $10{,}000$ | Finer paths, more memory |
| 0.01 m | $1{,}000{,}000$ | Near-continuous, expensive |

**Multi-resolution grids** refine only near obstacles or the current frontier - a bridge to [Section 3.4](../chapter-03-solving-problems-by-searching/section-04-informed-search.md) heuristics.

Discretization also appears in **motion planning** ([Chapter 12](../chapter-11-automated-planning/README.md)): configuration space is sliced into cells for PRM or lattice planners.

---

## Strategy 2: Gradient Descent

When the objective $f(\mathbf{x})$ is differentiable, **gradient descent** moves opposite the gradient:

$$
\mathbf{x}_{t+1} = \mathbf{x}_t - \alpha \nabla f(\mathbf{x}_t)
$$
> **Readable form:** Take small steps downhill - $\alpha$ is the step size (learning rate); $\nabla f$ points uphill, so subtract it.

For **maximization** (as in local search), use **gradient ascent**: add $\alpha \nabla f$.

```python
def gradient_descent(f, grad_f, x0, alpha=0.01, steps=1000):
  """Minimize f starting at x0."""
  x = x0
  for _ in range(steps):
    g = grad_f(x)
    if norm(g) < 1e-6:
      break
    x = x - alpha * g
  return x
```

| Variant | Idea | When useful |
|---------|------|-------------|
| **Batch GD** | Full gradient each step | Smooth, moderate $n$ |
| **Stochastic GD** | Mini-batch gradient | Large datasets ([Course 3](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/README.md)) |
| **Momentum** | Velocity term dampens zigzag | Ravines in loss landscape |
| **Adam** | Adaptive per-coordinate rates | Deep learning default |

Gradient descent is **local**: it converges to critical points (minima, maxima, saddles). Combine with **random restarts** from [Section 4.1](./section-01-local-search.md) or **simulated annealing** for global search.

**Connection:** The same math powers neural network training in [Course 3 Chapter 04](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-04-numerical-computation/README.md) - loss $L(\mathbf{w})$ minimized by $\mathbf{w} \leftarrow \mathbf{w} - \alpha \nabla L$.

---

## Strategy 3: Linear Programming (Introduction)

Many continuous problems with **linear objectives and linear constraints** reduce to **linear programming (LP)**:

$$
\begin{aligned}
\min_{\mathbf{x}} \quad & \mathbf{c}^\top \mathbf{x} \\
\text{s.t.} \quad & A\mathbf{x} \leq \mathbf{b} \\
& \mathbf{x} \geq \mathbf{0}
\end{aligned}
$$
> **Readable form:** Minimize a weighted sum of variables, subject to linear inequalities - the feasible region is a convex polytope.

**Simplex** and **interior-point** methods solve LPs in polynomial time in practice (though worst-case exponential for simplex). No gradient step-size tuning - the optimum is at a **vertex** (or face) of the polytope.

| Problem type | LP formulation sketch |
|--------------|----------------------|
| **Resource allocation** | Variables = hours per machine; constraints = capacity |
| **Shortest path (flow)** | Variables = flow on edges; minimize cost |
| **Robot reachability (relaxed)** | Linearize dynamics → approximate |

**Mixed-integer linear programming (MILP)** adds discrete variables - scheduling with "on/off" machines bridges to [Chapter 06](../chapter-06-constraint-satisfaction/README.md).

LP is **not** universal: nonlinear dynamics (quadrotor aerodynamics) need nonlinear programming or search.

---

## PEAS: Warehouse Robot in Continuous Space

| PEAS | Continuous navigation |
|------|------------------------|
| **P** - Performance | Minimize travel time; avoid collisions |
| **E** - Environment | Warehouse floor, shelves, humans |
| **A** - Actuators | Wheel velocities $(v, \omega)$ |
| **S** - Sensors | Lidar, encoders, cameras |

From [Section 2.3](../chapter-02-intelligent-agents/section-03-the-nature-of-environments.md): **continuous, partially observable, dynamic** - stricter than Romania graph search. Discretization or sampling-based planners approximate the PEAS task.

---

## Combining Discretization and Gradient Methods

A practical pipeline:

```
1. Discretize coarse roadmap (PRM nodes, visibility graph)
2. Graph search for topological route ([Chapter 03](../chapter-03-solving-problems-by-searching/README.md))
3. Smooth path with gradient-based optimization in continuous space
4. Track with local corrections (online - [Section 4.5](./section-05-online-search.md))
```

**Trajectory optimization** treats the whole path $\mathbf{x}(t)$ as variables and minimizes integral cost - continuous calculus of variations, often solved by discretizing time into $T$ steps and running gradient descent on $T \times n$ parameters.

---

## Objective Landscapes in $\mathbb{R}^n$

| Feature | Gradient descent behavior |
|---------|---------------------------|
| **Convex bowl** | Converges to global minimum |
| **Local minimum** | Stuck unless restarted |
| **Saddle point** | Slow escape; momentum helps |
| **Flat region** | $\|\nabla f\| \approx 0$ - step size matters |

For **constraint** $g(\mathbf{x}) \leq 0$, **projected gradient descent** steps then projects back to feasible set:

$$
\mathbf{x}_{t+1} = \Pi_{\mathcal{C}}\bigl(\mathbf{x}_t - \alpha \nabla f(\mathbf{x}_t)\bigr)
$$
> **Readable form:** After each gradient step, snap the point back to the nearest legal configuration.

---

## Worked Example: 2D Point Robot

Minimize distance to goal $G = (5, 5)$ from start $S = (0, 0)$ with cost $f(x, y) = (x-5)^2 + (y-5)^2$.

**Gradient:** $\nabla f = (2(x-5),\, 2(y-5))$.

At $(0, 0)$: $\nabla f = (-10, -10)$. With $\alpha = 0.1$:

$$
(x_1, y_1) = (0, 0) - 0.1 \cdot (-10, -10) = (1, 1)
$$
> **Readable form:** The score ranks search nodes by combining cost so far with an estimate of remaining cost.

Iterate: the path spirals inward toward $(5, 5)$ - here **convex**, so any reasonable $\alpha$ reaches the global minimum.

**Discretized alternative:** grid with $\Delta = 1$, 8-connected moves, Dijkstra from $(0,0)$ to $(5,5)$ - path length 10 (Manhattan) or $\approx 7.07$ diagonal if allowed. Finer $\Delta$ approaches continuous optimum.

| Method | Path quality | Compute |
|--------|--------------|---------|
| Gradient ($\alpha=0.1$) | Smooth, optimal | $O(\text{iterations})$ |
| Grid $\Delta=1$ | Zigzag on axes | $O(\text{cells})$ |
| Grid $\Delta=0.1$ | Near-optimal | 100× more cells |

---

## When to Use Which Tool

| Situation | Prefer |
|-----------|--------|
| Linear constraints + linear cost | LP solver |
| Smooth objective, local optimum OK | Gradient descent |
| Obstacles, nonconvex, need connectivity | Discretization + graph search |
| Unknown global structure | Random restarts, SA, evolutionary ([Section 4.1](./section-01-local-search.md)) |
| Real-time replanning | Hybrid + online search ([Section 4.5](./section-05-online-search.md)) |

---

## Connection to Classical Search

| Chapter 03 assumption | Continuous reality |
|---------------------|-------------------|
| Finite $\text{ACTIONS}(s)$ | Infinite control inputs |
| Exact $\text{RESULT}$ | Numerical integration error |
| Discrete goal test | Tolerance $\|\mathbf{x} - \mathbf{x}_g\| < \epsilon$ |

Discretization **restores** the five-tuple from [Section 3.1](../chapter-03-solving-problems-by-searching/section-01-problem-solving-agents.md) at the cost of state-space size. Gradient methods **bypass** explicit search when calculus applies.

---

## Connection to Other Chapters

| Topic | Chapter |
|-------|--------|
| MDP continuous states | [Chapter 17](../chapter-17-making-complex-decisions/README.md) |
| Motion planning (RRT, PRM) | [Chapter 12](../chapter-11-automated-planning/README.md) |
| Deep learning optimization | [Course 3](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/README.md) |
| CSP with continuous variables | [Chapter 06](../chapter-06-constraint-satisfaction/README.md) |

---

## Common Mistakes

**Grid too coarse.** Robot "fits" through walls in abstract grid but not in reality - inflate obstacles by robot radius.

**Learning rate too large.** Gradient descent oscillates or diverges; too small - glacial progress.

**Confusing discretization resolution with optimality.** Finer grid improves path but does not guarantee global optimum in nonconvex spaces.

**Applying LP to nonlinear physics.** Linearized models are approximations; verify feasibility in simulation.

---

## Reflection Questions

1. How does discretization trade memory for solution quality? Give a numeric example with $\Delta$.
2. Why does gradient descent on a convex function find the global minimum, but fail on arbitrary landscapes?
3. What makes a warehouse routing problem a good LP candidate? What would break the LP model?
4. When would you combine graph search on a coarse grid with continuous smoothing?
5. How do continuous state spaces violate the assumptions of BFS optimality proofs from [Section 3.3](../chapter-03-solving-problems-by-searching/section-03-uninformed-search.md)?

---

**Next:** [Section 4.3 - Nondeterministic Search](./section-03-nondeterministic-search.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 4.2. [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Boyd, S., & Vandenberghe, L. (2004). *Convex Optimization*. [Stanford EE364](https://web.stanford.edu/~boyd/cvxbook/)
3. Russell, S., & Norvig, P. - `search.py` continuous examples. [aima-python](https://github.com/aimacode/aima-python)
4. LaValle, S. M. - motion planning and configuration spaces. [Planning Algorithms](http://planning.cs.uiuc.edu/)
5. Nocedal, J., & Wright, S. (2006). *Numerical Optimization* - gradient methods.



