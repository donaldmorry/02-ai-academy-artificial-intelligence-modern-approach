# Section 17.7: MDP Applications

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17.1-17.2  
> **Previous:** [Section 17.6 - Bandits](./section-06-multi-armed-bandits.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## From Theory to Practice

Markov decision processes model sequential decisions in operations research, robotics, finance, and healthcare. The same Bellman machinery ([Sections 17.3-17.4](./section-03-value-iteration.md)) applies once you define states, actions, transitions, and rewards.

This section walks through three canonical formulations: **inventory control**, **robot navigation**, and **preventive maintenance**.

> **Readable form:** Real problems become MDPs when you can list "situations," "choices," "what happens next (maybe randomly)," and "payoffs per step."

---

## Inventory Management

A store holds $s \in \{0, 1, \ldots, S_{\max}\}$ units of stock. Each day:

1. **Demand** $D$ is random (e.g., Poisson with mean $\lambda$)
2. Agent orders quantity $a \in \{0, \ldots, A_{\max}\}$ (arrives immediately or next period per model)
3. **Sales** $= \min(s + a, D)$; **holding cost** $h$ per unit; **stockout penalty** $p$ per unmet demand

**State:** inventory level $s$  
**Action:** order quantity $a$  
**Transition:** $s' = \max(0, s + a - D)$ with distribution over $D$  
**Reward:** sales revenue minus holding and stockout costs

$$
R(s, a, D) = \text{price} \cdot \min(s+a, D) - h \cdot s' - p \cdot \max(0, D - s - a)
$$
> **Readable form:** Inventory reward equals sales revenue minus holding cost for leftover stock and penalty cost for unmet demand.

**Discount $\gamma$** reflects time value of money. Optimal policy often has ** at $(s, S)$ structure**: reorder when $s \leq s_{\text{low}}$, order up to $S$.

> **Readable form:** Balance ordering costs against running out. VI finds reorder thresholds automatically.

---

## Worked Example: Tiny Inventory MDP

States $\{0, 1, 2\}$, actions order $\{0, 1\}$, demand $\in \{0, 1\}$ equally likely, $h = 1$, $p = 5$, revenue $= 4$ per unit sold, $\gamma = 0.9$.

From $s = 0$, ordering 1 yields expected next state 0 or 1 with sales revenue offset by costs. Value iteration identifies when ordering is worth stockout risk vs. holding cost.

Policy sketch: at $s = 0$, order (avoid stockout); at $s = 2$, do not order (avoid holding).

---

## Robot Navigation (Grid World)

AIMA's $4 \times 3$ grid ([Figure 17.1](https://aima.cs.berkeley.edu/)): robot moves {N, S, E, W} with stochastic slip (80% intended, 10% each perpendicular). Terminal cells: $+1$ goal, $-1$ pit. Step reward $r$ for nonterminal moves (often small negative, e.g., $-0.04$).

**State:** grid coordinates (excluding walls)  
**Action:** move directions  
**$P(s' \mid s, a)$:** slip model  
**$R$:** $r$ per step until terminal absorption

Optimal policy balances **shortest path** vs. **pit avoidance** - risk-sensitive routing. Changing $r$ shifts policies ([Section 17.1](./section-01-sequential-decisions.md)).

Connection to [Chapter 03](../chapter-03-solving-problems-by-searching/README.md): deterministic grid = search; stochastic slip = MDP.

---

## Maintenance and Reliability

Machine **degradation state** $s \in \{0, 1, \ldots, K\}$ ($0$ = new, $K$ = failure). Each period:

- **Operate:** earn production reward $R_{\text{prod}}$, degrade stochastically
- **Maintain:** cost $C_m$, reset or improve state
- **Replace:** cost $C_r$, reset to new

**Failure** in state $K$ incurs downtime cost $C_f$.

$$
\text{Goal: } \max_\pi \mathbb{E}\left[\sum_t \gamma^t R(s_t, a_t)\right]
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

Optimal policies are **threshold rules**: maintain when $s \geq s^*$, else operate - classic in reliability engineering.

---

## Other Application Domains

| Domain | States | Actions | Notes |
|--------|--------|---------|-------|
| **Portfolio** | wealth, market regime | allocation | Continuous approximations |
| **Queueing** | queue length | serve/admit | Admission control |
| **Clinical** | disease stage | treat/wait | POMDP if partial tests |
| **Resource allocation** | budgets per project | fund levels | Restless bandits variant |
| **Dialogue** | user intent belief | utterance | POMDP formulation |

---

## Structuring an MDP Model

```
1. Identify decision epochs (discrete time steps)
2. Define state variables capturing Markov sufficiency
3. List feasible actions per state
4. Estimate P(s'|s,a) from data or physics
5. Assign rewards aligned with business objective
6. Choose γ (or finite horizon T)
7. Solve with VI/PI; validate sensitivity
```

**Reward engineering** is critical - misaligned rewards produce optimal-but-useless policies ([Chapter 22](../chapter-22-reinforcement-learning/README.md) discusses reward shaping).

---

## Scaling Beyond Tabular Methods

Real applications often have $|S| \gg 10^6$:

| Technique | Idea |
|-----------|------|
| **State aggregation** | Cluster similar states |
| **Linear function approximation** | $V(s) \approx \theta^\top \phi(s)$ |
| **Simulation + MCTS** | Sample trajectories |
| **Model predictive control** | Short horizon replanning |

[Section 17.8](./section-08-from-mdps-to-reinforcement-learning.md) covers the model-free transition.

---

## Python: Inventory MDP Sketch

```python
import numpy as np

def inventory_mdp(max_stock, max_order, lam, price, hold, stockout, gamma):
  states = range(max_stock + 1)
  actions = range(max_order + 1)

  def transitions(s, a):
    probs = {}
    for d in range(max_stock + max_order + 1):
      p_d = poisson_pmf(d, lam)
      sold = min(s + a, d)
      sp = max(0, s + a - d)
      r = price * sold - hold * sp - stockout * max(0, d - s - a)
      probs[(sp, r)] = probs.get((sp, r), 0) + p_d
    return probs

  V = {s: 0.0 for s in states}
  for _ in range(200):
    new_V = {}
    for s in states:
      best = float("-inf")
      for a in actions:
        if a + s > max_stock + max_order:
          continue
        q = sum(p * (r + gamma * V[sp]) for (sp, r), p in transitions(s, a).items())
        best = max(best, q)
      new_V[s] = best
    V = new_V
  return V

def poisson_pmf(k, lam):
  from math import exp, factorial
  return exp(-lam) * lam**k / factorial(k)
```

> **Readable form:** For each stock level and order size, average over demand scenarios; Bellman max finds best order policy.

---

## Validation and Sensitivity

After solving:

1. **Simulation** under optimal $\pi$ with confidence intervals
2. **Sensitivity** to $\gamma$, reward scale, transition estimates
3. **Compare** to heuristics ($(s,S)$, greedy routing)
4. **Expert review** of policy plausibility

AIMA notes: **reward scale invariance** for optimal policy under multiplication by positive constant (with fixed $\gamma$) - but additive shifts can change decisions.

---

## Common Mistakes

**Non-Markov states** - omitting variables needed for optimal decisions (e.g., pending orders).

**Wrong time step** - mixing daily demand with hourly decisions.

**Ignoring constraints** - action sets must reflect real feasibility.

**Overfitting transition model** from sparse data.

---

## Factored and Continuous MDPs

Real problems often use **factored MDPs** - state as vector of variables with sparse dependencies in $P$ and $R$. **Continuous state** MDPs (e.g., robot pose $(x, y, \theta)$) require discretization or function approximation ([Section 17.8](./section-08-from-mdps-to-reinforcement-learning.md)).

Operations research literature (Puterman, Bertsekas) catalogs structural results: monotone policies for convex inventory costs, threshold policies for reliability.

> **Readable form:** When state has many components, exploit structure instead of flat tables - the math is the same, the representation differs.

---

## Reflection Questions

1. Formulate a coffee-shop inventory MDP with perishable goods. What states are needed?
2. How does the grid-world pit change optimal policy as step penalty $r$ varies?
3. When is maintenance MDP better as a POMDP?
4. Why might $(s, S)$ policies emerge from VI on inventory problems?
5. What data would you collect to estimate $P(s' \mid s, a)$ in navigation?

---

**Previous:** [Section 17.6 - Bandits](./section-06-multi-armed-bandits.md)  
**Next:** [Section 17.8 - From MDPs to RL](./section-08-from-mdps-to-reinforcement-learning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Sections 17.1-17.2.
2. Puterman, M. L. (1994). *Markov Decision Processes* - inventory, reliability.
3. Bertsekas, D. P. - dynamic programming applications.
4. Powell, W. B. - approximate dynamic programming for operations.
5. aima-python: `mdp.py` - $4 \times 3$ grid world.
