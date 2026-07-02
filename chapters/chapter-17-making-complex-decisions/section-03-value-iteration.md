# Section 17.3: Value Iteration

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17.3  
> **Previous:** [Section 17.2 - MDPs](./section-02-markov-decision-processes.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Computing Optimal Policies

Given MDP $\langle S, A, T, R, \gamma \rangle$, we want $V^*(s)$ and $\pi^*(s)$.

**Value iteration** is a dynamic programming algorithm:

1. Initialize $V_0(s)$ arbitrarily (often 0)
2. **Bellman update** for each state:
   $$
   V_{k+1}(s) = \max_a \left[ R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) V_k(s') \right]
   $$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

3. Repeat until convergence: $\max_s |V_{k+1}(s) - V_k(s)| < \epsilon$

Extract policy:

$$
\pi^*(s) = \arg\max_a \left[ R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) V^*(s') \right]
$$
> **Readable form:** Repeatedly improve each state's score by asking "what's the best one-step action plus discounted value of where I'd end up?" When numbers stop changing much, you've found optimal values.

---

## Why It Converges

Define **Bellman operator** $\mathcal{B}$:

$$
(\mathcal{B}V)(s) = \max_a \left[ R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) V(s') \right]
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**Contraction mapping** with modulus $\gamma < 1$:

$$
\|\mathcal{B}V - \mathcal{B}V'\|_\infty \leq \gamma \|V - V'\|_\infty
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

By Banach fixed-point theorem, repeated application converges to unique $V^*$.

**Convergence rate:** error bounded by $\gamma^k$ after $k$ iterations.

---

## Worked Trace: Tiny MDP

States $\{A, B\}$, actions $\{stay, go\}$, γ = 0.9.

From $A$: go → $B$ w.p. 1, $R=0$. stay → $A$, $R=1$.

From $B$: stay → $B$, $R=5$. go → $A$, $R=0$.

Terminal? No - continuing task.

Iteration ( $V_0 = 0$ ):

**$A$:** stay: $1 + 0.9(0)=1$; go: $0 + 0.9 V(B)$ → if $V_0(B)=0$, pick stay, $V_1(A)=1$.

**$B$:** stay: $5 + 0.9(0)=5$ → $V_1(B)=5$.

**Round 2 - $A$:** stay: $1 + 0.9(5)=5.5$; go: $0 + 0.9(5)=4.5$ → $V_2(A)=5.5$.

**$B$:** stay: $5 + 0.9(5.5)=9.95$ → $V_2(B)=9.95$.

Continues toward fixed point - optimal: stay in $B$ forever once there.

---

## Grid World Value Iteration

AIMA 4×3 grid ([Section 17.2](./section-02-markov-decision-processes.md)):

- Initialize $V=0$ everywhere except terminal cells fixed at ±1
- Sweep all non-terminal cells with Bellman max
- After ~10-20 iterations with γ=0.9, values stabilize
- Policy arrows point toward +1, away from −1

**Living reward** −0.04 makes "long detour" vs. "risky shortcut" tradeoff visible in $\pi^*$.

---

## Asynchronous Value Iteration

**Synchronous:** update all states each iteration using $V_k$.

**Asynchronous:** update one state at a time using latest values - often faster in practice (prioritized sweeping).

**Real-time dynamic programming (RTDP):** sample relevant states only - useful for large spaces.

---

## Q-Value Iteration

Alternatively iterate $Q$:

$$
Q_{k+1}(s,a) = R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) \max_{a'} Q_k(s', a')
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Then $V^*(s) = \max_a Q^*(s,a)$.

Foundation for **Q-learning** when $T$ unknown ([Section 17.8](./section-08-from-mdps-to-reinforcement-learning.md)).

---

## Python Implementation

```python
def value_iteration(mdp, epsilon=1e-6, max_iter=10000):
    V = {s: 0.0 for s in mdp.states}
    while max_iter > 0:
        delta = 0
        for s in mdp.states:
            if mdp.is_terminal(s):
                continue
            v_old = V[s]
            best = float("-inf")
            for a in mdp.actions(s):
                q = mdp.R[s][a] + mdp.gamma * sum(
                    mdp.T[s][a][sp] * V[sp] for sp in mdp.states
                )
                best = max(best, q)
            V[s] = best
            delta = max(delta, abs(v_old - V[s]))
        if delta < epsilon:
            break
        max_iter -= 1
    policy = extract_policy(mdp, V)
    return V, policy

def extract_policy(mdp, V):
    policy = {}
    for s in mdp.states:
        if mdp.is_terminal(s):
            continue
        policy[s] = max(
            mdp.actions(s),
            key=lambda a: mdp.R[s][a] + mdp.gamma * sum(
                mdp.T[s][a][sp] * V[sp] for sp in mdp.states
            ),
        )
    return policy
```

---

## Complexity and Practical Limits

Per iteration: $O(|S| \cdot |A| \cdot |S|)$ for dense $T$.

For $|S| = 10^6$, tabular VI infeasible - use:

- **Sparse** transitions (only neighboring grid cells)
- **Linear function approximation** $V(s) \approx \mathbf{w} \cdot \phi(s)$
- **Deep RL** ([Chapter 22](../chapter-22-reinforcement-learning/README.md))

---

## Optimal Policy May Be Stochastic?

For **fully observed MDPs**, there exists an optimal **deterministic** policy. Stochastic policies matter in POMDPs and adversarial settings.

---

## Comparison to Search

| Search ([Chapter 03](../chapter-03-solving-problems-by-searching/README.md)) | Value iteration |
|---------------------------------------------------------------------|-----------------|
| Single trajectory to goal | All states simultaneously |
| Deterministic actions common | Stochastic transitions native |
| Path cost $g(n)$ | Discounted cumulative reward |

VI is **global** optimization over infinite horizon.

---

## Common Mistakes

**Not fixing terminal state values** during updates.

**Using γ = 1** without proper terminal structure.

**Extracting policy before convergence** - suboptimal actions.

**Off-by-one in reward timing** - $R(s,a)$ vs. $R(s')$.

---

## Asynchronous and Prioritized Updates

**Asynchronous value iteration** updates one state at a time in any order - useful when updates propagate from goal states outward. **Prioritized sweeping** maintains a priority queue of states whose value changed significantly, revisiting predecessors first.

For the AIMA $4 \times 3$ grid, updates from the $+1$ terminal propagate backward through the state space; asynchronous variants can converge faster when only a subset of states matter for the optimal policy.

> **Readable form:** Instead of updating every state each sweep, focus effort on states whose values just changed - ripples spread from goals and penalties.

---

## Online Planning Preview

Value iteration is **offline** - full model required before acting. **Real-time dynamic programming (RTDP)** samples trajectories and updates $V$ along visited states only, bridging to online MDP solvers used in robotics when the full grid is too large to sweep each step.

---

## Reflection Questions

1. Prove that $V^*$ satisfies the Bellman optimality equation (fixed point).
2. How many iterations until error < ε if $\|V_1 - V^*\|_\infty \leq 1$ and γ=0.9?
3. Run mental VI for one non-terminal cell in grid world after neighbors known.
4. When is asynchronous VI guaranteed to converge?
5. Relate VI update to one-step lookahead in game tree search.

---

**Previous:** [Section 17.2 - MDPs](./section-02-markov-decision-processes.md)  
**Next:** [Section 17.4 - Policy Iteration](./section-04-policy-iteration.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 17.3.
2. Bellman, R. (1957). *Dynamic Programming*.
3. Bertsekas, D. P. - value iteration and async methods.
4. Sutton & Barto (2018). *RL*, Section 4.4.
5. aima-python: `value_iteration` in `mdp.py`.


