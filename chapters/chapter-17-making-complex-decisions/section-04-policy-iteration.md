# Section 17.4: Policy Iteration

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17.3  
> **Previous:** [Section 17.3 - Value Iteration](./section-03-value-iteration.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Two-Phase Optimization

**Policy iteration** alternates:

1. **Policy evaluation** - compute $V^\pi$ for fixed $\pi$
2. **Policy improvement** - update $\pi$ greedily from $V^\pi$

Often converges in **fewer outer iterations** than value iteration needs sweeps, though each evaluation step can be costly.

> **Readable form:** Lock in a plan, score how good it is everywhere, then upgrade every state's action to be greedy. Repeat until the plan stops changing.

---

## Policy Evaluation

Given deterministic policy $\pi(s) = a$, solve linear system:

$$
V^\pi(s) = R(s, \pi(s)) + \gamma \sum_{s'} P(s' \mid s, \pi(s)) V^\pi(s')
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

For all $s$ - $|S|$ equations, $|S|$ unknowns.

**Iterative evaluation** (like VI but without max):

$$
V_{k+1}(s) = R(s, \pi(s)) + \gamma \sum_{s'} P(s' \mid s, \pi(s)) V_k(s')
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Repeat until $\Delta < \theta$ (small threshold).

---

## Policy Improvement Theorem

Define new greedy policy:

$$
\pi'(s) = \arg\max_a \left[ R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) V^\pi(s') \right]
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**Theorem:** $V^{\pi'}(s) \geq V^\pi(s)$ for all $s$, with strict inequality unless $\pi$ is optimal.

**Corollary:** Finite MDP → policy iteration reaches $\pi^*$ in finite steps (at most $|A|^{|S|}$ policies, often far fewer).

---

## Algorithm Sketch

```
π ← arbitrary policy (e.g., random)
loop:
    V ← POLICY-EVALUATION(π, MDP)
    π' ← greedy policy from V
    if π' == π: return π
    π ← π'
```

---

## Worked Example

Reuse two-state MDP from [Section 17.3](./section-03-value-iteration.md).

**Initial π:** always `go` from A, `stay` from B.

Evaluate: solve coupled equations or iterate.

Improve at A: compare stay vs go using $V^\pi$ - likely switch to stay if B not yet reached.

Few cycles to $\pi^*$: stay at B, navigate A→B optimally.

---

## Policy Iteration vs. Value Iteration

| Aspect | Value iteration | Policy iteration |
|--------|-----------------|------------------|
| Update | Bellman optimality on $V$ | Full policy eval + improve |
| Iterations | Many cheap sweeps | Few expensive rounds |
| Intermediate policy | May be suboptimal until end | Always valid policy |
| Linear algebra | Optional | Can solve exactly |

**Modified policy iteration:** partial evaluation (few sweeps) + improvement - hybrid used in practice.

---

## Solving Policy Evaluation Exactly

Rearrange Bellman equations for $\pi$:

$$
V = R^\pi + \gamma P^\pi V \implies V = (I - \gamma P^\pi)^{-1} R^\pi
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Matrix inversion $O(|S|^3)$ - feasible for small $|S|$ only.

---

## Python: Policy Iteration

```python
def policy_evaluation(mdp, policy, theta=1e-8):
    V = {s: 0.0 for s in mdp.states}
    while True:
        delta = 0
        for s in mdp.states:
            if mdp.is_terminal(s):
                continue
            a = policy[s]
            v = mdp.R[s][a] + mdp.gamma * sum(
                mdp.T[s][a][sp] * V[sp] for sp in mdp.states
            )
            delta = max(delta, abs(v - V[s]))
            V[s] = v
        if delta < theta:
            return V

def policy_improvement(mdp, V):
    policy = {}
    stable = True
    for s in mdp.states:
        if mdp.is_terminal(s):
            continue
        old_a = policy.get(s)
        best_a = max(
            mdp.actions(s),
            key=lambda a: mdp.R[s][a] + mdp.gamma * sum(
                mdp.T[s][a][sp] * V[sp] for sp in mdp.states
            ),
        )
        policy[s] = best_a
    return policy

def policy_iteration(mdp):
    policy = {s: mdp.actions(s)[0] for s in mdp.states if not mdp.is_terminal(s)}
    while True:
        V = policy_evaluation(mdp, policy)
        new_policy = policy_improvement(mdp, V)
        if new_policy == policy:
            return policy, V
        policy = new_policy
```

---

## When Policy Iteration Wins

- **Few actions**, many states - evaluation exploits structure
- Need **anytime valid policy** during computation (robotics safety)
- **Sparse** policies - improvement changes few states

Value iteration simpler to implement; often comparable for medium grids.

---

## Generalized Policy Iteration (GPI)

Sutton & Barto view: **value function** and **policy** chase each other until consistent with optimality.

```
     ┌──────────────┐
     │  Evaluation  │  drives V toward V^π
     └──────┬───────┘
            │
     ┌──────▼───────┐
     │ Improvement  │  drives π toward π*
     └──────────────┘
```

VI performs both in one step (max inside update).

---

## Common Mistakes

**Comparing π before evaluation converged** - unstable improvement.

**Not updating all states** in improvement pass.

**Random tie-breaking** hiding policy oscillation (rare with strict improvement).

---

## Policy Loss Bound

After $i$ iterations of value iteration with max error $\epsilon$, the **greedy policy** $\pi_i$ satisfies:

$$
|V^*(s) - V^{\pi_i}(s)| \leq \frac{2\gamma \epsilon}{1 - \gamma}
$$
> **Readable form:** If the value estimate is within epsilon, the greedy policy's value is bounded near optimal, with the discount factor amplifying the error.

Policy iteration often reaches exact $\pi^*$ in fewer conceptual rounds because each improvement step is exact relative to current $V^\pi$.

> **Readable form:** Even approximate values can yield near-optimal policies if $\epsilon$ is small and $\gamma$ is not too close to 1.

---

## Reflection Questions

1. Why must policy evaluation reach convergence before improvement (or use partial eval with care)?
2. Upper bound on number of policy iteration steps?
3. When is matrix inversion for policy evaluation preferable to iterative?
4. Does policy iteration always change π each round?
5. How does modified policy iteration interpolate VI and PI?

---

**Previous:** [Section 17.3 - Value Iteration](./section-03-value-iteration.md)  
**Next:** [Section 17.5 - POMDPs](./section-05-partially-observable-mdps.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 17.3.
2. Howard, R. A. (1960). Dynamic programming and Markov processes.
3. Puterman, M. L. (1994). *MDPs*, policy iteration chapter.
4. Sutton & Barto (2018). *RL*, Section 4.3 - GPI.
5. aima-python: `policy_iteration`.

