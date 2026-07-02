# Section 17.5: Partially Observable MDPs

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17.4-17.5  
> **Previous:** [Section 17.4 - Policy Iteration](./section-04-policy-iteration.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## When States Are Hidden

In an MDP, the agent knows its current state $s_t$ exactly. In many real problems, perception is **noisy or partial**:

- Robot localization: GPS error, occluded landmarks
- Medical treatment: patient state not fully observable
- Poker: opponent cards hidden
- Dialogue: user intent uncertain

A **partially observable MDP (POMDP)** extends the MDP tuple with an **observation model**:

$$
\langle S, A, T, R, \Omega, O, \gamma \rangle
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

| Symbol | Meaning |
|--------|---------|
| $\Omega$ | Set of observations |
| $O(o \mid s', a)$ | Probability of observation $o$ after action $a$ leads to $s'$ |

> **Readable form:** You still choose actions and get rewards, but you do not know for sure where you are - you only get clues (observations) that depend on the true state.

---

## Belief States

Because the true state is unknown, the agent maintains a **belief state** $b(s)$:

$$
b(s) = P(s \mid o_{1:t}, a_{1:t-1})
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Belief is a **probability distribution** over $S$. For $|S| = n$, belief lives in an $(n-1)$-simplex (continuous space).

**Belief update** after action $a$ and observation $o$:

$$
b'(s') = \eta \, O(o \mid s', a) \sum_s P(s' \mid s, a) \, b(s)
$$
> **Readable form:** New belief in state $s'$ equals normalized observation likelihood times the predicted probability of reaching $s'$ from prior beliefs.

where $\eta$ normalizes so $\sum_{s'} b'(s') = 1$.

> **Readable form:** Predict where you might be after your action, then reweight by how likely your sensor reading is in each possible new state. Renormalize to get a fresh probability map.

---

## Value Function Over Beliefs

The fundamental POMDP insight (AIMA Section 17.4): the **optimal action depends only on the current belief**, not the full history.

Define **value of information** implicitly: acting to reduce uncertainty can be optimal when observations are informative.

**Value function** $V(b)$ assigns expected discounted return starting from belief $b$.

Bellman equation in belief space:

$$
V(b) = \max_a \left[ \sum_{s} b(s) R(s,a) + \gamma \sum_{o} P(o \mid b, a) \, V(b^{a,o}) \right]
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

where $b^{a,o}$ is the updated belief after $(a, o)$.

---

## POMDP Decision Cycle

```
1. Start with belief b
2. Choose action a = π*(b)
3. Execute a, receive observation o and reward r
4. Update b ← UPDATE(b, a, o)
5. Repeat
```

This is **planning in belief-state space** - analogous to contingency planning ([Chapter 04](../chapter-04-search-complex-environments/README.md)) but with continuous beliefs.

---

## Why POMDPs Are Hard

| MDP | POMDP |
|-----|-------|
| $|S|$ states | Continuous belief space |
| Tabular $V(s)$ | $V(b)$ piecewise-linear (finite horizons) or complex |
| Polynomial algorithms (VI, PI) | PSPACE-complete in general |

Exact POMDP solvers (e.g., point-based methods, SARSOP) work for small $|S|, |A|, |O|$.

**Practical approximations:**

- **Myopic:** act as if current MAP state $\hat{s} = \arg\max_s b(s)$ is true
- **QMDP:** ignore future observability in value backup
- **Monte Carlo tree search** over beliefs
- **Deep RL** with recurrent networks ([Chapter 22](../chapter-22-reinforcement-learning/README.md))

---

## Worked Example: Tiger Problem

Classic POMDP: two doors - one hides a tiger (−100), one treasure (+10). Agent can listen (noisy clue) or open a door.

States: tiger left (TL), tiger right (TR).  
Actions: listen, open-left, open-right.  
Observations after listen: hear-left, hear-right (85% accurate).

Start $b = (0.5, 0.5)$. **Listen** costs −1 but sharpens belief. Optimal policy listens until confident, then opens safe door.

If agent opens immediately: expected reward $= 0.5 \times 10 + 0.5 \times (-100) = -45$.

After hearing-left with 85% accuracy: $b(\text{TL}) \approx 0.85$ - opening right yields $\approx 0.15 \times (-100) + 0.85 \times 10 = 6.5$ minus open cost.

> **Readable form:** Pay a small cost to listen because information prevents catastrophic mistakes.

---

## Connection to Filtering

Belief update is **Bayesian filtering** ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)):

- HMM: fixed actions (passive)
- POMDP: choose actions that shape future beliefs and rewards

Kalman filters for linear Gaussian POMDPs admit closed-form belief updates.

---

## Alpha Vectors (Sketch)

For finite-horizon POMDPs, $V(b)$ is the **maximum of finitely many linear functions** of $b$:

$$
V(b) = \max_{\alpha \in \Gamma} \sum_s \alpha(s) \, b(s)
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

Each $\alpha$ vector corresponds to a conditional plan. **Witness algorithm** and point-based backups extend VI to belief space - beyond scope here but key for exact small POMDPs.

---

## Python: Belief Update

```python
import numpy as np

def belief_update(b, action, obs, T, O, states):
    """b: dict or array over states; T[s][a][sp]; O[sp][a][o]."""
    b_arr = np.array([b[s] for s in states])
    new = np.zeros(len(states))
    for i, sp in enumerate(states):
        pred = sum(T[s][action][sp] * b_arr[j] for j, s in enumerate(states))
        new[i] = O[sp][action][obs] * pred
    new /= new.sum()
    return {s: new[i] for i, s in enumerate(states)}

def qmdp_action(b, actions, R, T, O, gamma, V_mdp):
    """Greedy: use MDP value at belief-weighted states (approximation)."""
    best_a, best_q = None, float("-inf")
    for a in actions:
        q = sum(b[s] * R[s][a] for s in b)
        for sp in b:
            q += gamma * sum(
                T[s][a][sp] * b[s] * V_mdp[sp] for s in b
            )
        if q > best_q:
            best_a, best_q = a, q
    return best_a
```

> **Readable form:** Multiply prior by transition, weight by observation likelihood, normalize. QMDP shortcut uses full-observability values - fast but suboptimal when gathering information matters.

---

## POMDP vs. MDP Design Choice

| Model full observability? | When |
|---------------------------|------|
| **MDP** | State features fully capture decision-relevant info |
| **POMDP** | Sensors incomplete; information actions have cost/value |
| **Belief MDP** | Theoretically equivalent: state = belief |

Many robotics pipelines use **separate localization + MDP planning** - pragmatic factorization.

---

## Common Mistakes

**Treating MAP state as truth** - ignores risk when belief is diffuse.

**Forgetting observation model** in belief update - inconsistent filtering.

**Assuming POMDP value iteration is like MDP VI** - belief space is continuous; grid discretization explodes.

**Ignoring **information-gathering** actions** - myopic policies open wrong doors.

---

## Reflection Questions

1. Why is the optimal POMDP policy a function of belief rather than last observation alone?
2. How does the tiger problem illustrate value of information from [Chapter 16](../chapter-16-making-simple-decisions/section-04-value-of-information.md)?
3. When might QMDP be nearly optimal?
4. What is the dimensionality of belief space for $|S| = 100$?
5. How do POMDPs relate to reinforcement learning with partial observability?

---

**Previous:** [Section 17.4 - Policy Iteration](./section-04-policy-iteration.md)  
**Next:** [Section 17.6 - Bandits](./section-06-multi-armed-bandits.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Sections 17.4-17.5.
2. Kaelbling, L. P., Littman, M. L., & Cassandra, A. R. (1998). Planning and acting in partially observable stochastic domains.
3. Pineau, J., et al. - point-based POMDP solvers.
4. Thrun, S., Burgard, W., & Fox, D. (2005). *Probabilistic Robotics*.
5. aima-python: `pomdp.py`, tiger problem implementations.

