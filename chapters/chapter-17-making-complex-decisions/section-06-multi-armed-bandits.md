# Section 17.6: Multi-Armed Bandits

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 17.3  
> **Previous:** [Section 17.5 - POMDPs](./section-05-partially-observable-mdps.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Exploration vs. Exploitation

A **multi-armed bandit** is the simplest sequential decision problem:

- $K$ slot machines ("arms"), each with unknown expected reward $\mu_i$
- Each step: pull one arm, receive stochastic reward $R \sim P_i$
- Goal: maximize cumulative reward over $T$ pulls

No state transitions - only **which arm to try**. This isolates the **exploration-exploitation trade-off** central to reinforcement learning.

> **Readable form:** You face several mysterious buttons. Each gives random payouts with different averages. You have limited tries - when do you keep pressing the best-looking button vs. testing others?

---

## Regret

**Regret** after $T$ steps measures opportunity cost vs. always pulling the best arm:

$$
\text{Regret}(T) = T \cdot \mu^* - \sum_{t=1}^{T} R_t, \quad \mu^* = \max_i \mu_i
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Expected regret** compares your policy to the oracle that knew $\mu_i$ from the start.

A good algorithm achieves **sublinear regret**: $\text{Regret}(T) = o(T)$, so average regret per step $\to 0$.

| Algorithm | Expected regret (worst case) |
|-----------|------------------------------|
| Random | Linear $O(T)$ |
| $\varepsilon$-greedy | Linear (constant $\varepsilon$) |
| UCB1 | $O(\sqrt{T \log T})$ |
| Thompson sampling | Near-optimal Bayesian |

---

## Sample-Average Estimates

After $n_i$ pulls of arm $i$, estimate:

$$
\hat{\mu}_i = \frac{1}{n_i} \sum_{t: \text{arm } i} R_t
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Greedy:** pull $\arg\max_i \hat{\mu}_i$ - converges to best arm if all arms tried infinitely often, but may **prematurely commit** to a lucky early pull.

---

## $\varepsilon$-Greedy

With probability $\varepsilon$, pull uniformly random arm; else greedy on $\hat{\mu}_i$.

$$
a_t = \begin{cases}
\text{random arm} & \text{w.p. } \varepsilon \\
\arg\max_i \hat{\mu}_i & \text{w.p. } 1 - \varepsilon
\end{cases}
$$
> **Readable form:** At time $t$, explore by choosing a random arm with probability epsilon; otherwise exploit the arm with the highest estimated mean reward.

Decay $\varepsilon_t \to 0$ (e.g., $\varepsilon_t = 1/t$) to achieve convergence. Simple baseline in RL ([Chapter 22](../chapter-22-reinforcement-learning/README.md)).

> **Readable form:** Usually pick the best-looking arm, but occasionally gamble on a random one to avoid getting stuck.

---

## Upper Confidence Bound (UCB1)

**Optimism in the face of uncertainty:** prefer arms that *could* be best.

$$
a_t = \arg\max_i \left[ \hat{\mu}_i + c \sqrt{\frac{\ln t}{n_i}} \right]
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

- $\hat{\mu}_i$ - exploitation
- $\sqrt{\ln t / n_i}$ - bonus for under-sampled arms
- $c$ - exploration constant (often $\sqrt{2}$)

**Theorem (Auer et al.):** UCB1 achieves $O(\sqrt{T \log T})$ regret for bounded rewards.

> **Readable form:** Score each arm as "what I've seen so far" plus a bonus for arms I haven't tried much. Uncertainty shrinks as $n_i$ grows.

---

## Worked Example: Three Arms

True means: $\mu_1 = 0.5$, $\mu_2 = 0.7$, $\mu_3 = 0.6$. After 10 rounds, arm 2 pulled once with reward 1.0 (lucky), arm 3 pulled 5 times with $\hat{\mu}_3 = 0.58$, arm 1 pulled 4 times with $\hat{\mu}_1 = 0.50$.

At $t = 10$, UCB1 bonuses ( $c = \sqrt{2}$ ):

$$
\text{UCB}_2 = 1.0 + \sqrt{2} \sqrt{\ln 10 / 1} \approx 1.0 + 2.15 = 3.15
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

$$
\text{UCB}_3 = 0.58 + \sqrt{2} \sqrt{\ln 10 / 5} \approx 0.58 + 0.96 = 1.54
$$
> **Readable form:** The value summarizes preference or expected payoff so an agent can compare available choices.

Greedy picks arm 2 (misleading); UCB1 may still explore arms 1 and 3 until estimates stabilize.

---

## Contextual Bandits

**Contextual bandits:** each round includes context $x_t$; arm $i$ has reward distribution $P(R \mid x, i)$.

Applications: ad placement, news recommendation, clinical trials with patient features.

Linear models: $\mathbb{E}[R \mid x, a] = w_a^\top x$ - bridge to supervised learning ([Chapter 19](../chapter-19-learning-from-examples/README.md)).

---

## Gittins Index (Brief)

For **Bayesian bandits** with prior on each arm (e.g., Beta for Bernoulli), the **Gittins index** gives the optimal policy for **infinite discounted horizon** - computable per arm independently.

UCB is a frequentist approximation; Thompson sampling draws $\tilde{\mu}_i$ from posterior and pulls $\arg\max_i \tilde{\mu}_i$ - often excellent empirically.

---

## Bandits vs. Full MDPs

| Bandit | MDP |
|--------|-----|
| Single state | Many states |
| Action affects only reward | Action changes state |
| Regret analysis clean | Bellman equations |
| Hyperparameter tuning, A/B tests | Navigation, robotics |

**Restless bandits:** each arm is a small MDP - NP-hard in general; Gittins index optimal for special cases.

---

## Python: UCB1 and ε-Greedy

```python
import math
import random

class UCB1Bandit:
    def __init__(self, k, c=math.sqrt(2)):
        self.k = k
        self.c = c
        self.counts = [0] * k
        self.values = [0.0] * k
        self.t = 0

    def select(self):
        self.t += 1
        for i in range(self.k):
            if self.counts[i] == 0:
                return i
        ucb = [
            self.values[i] + self.c * math.sqrt(math.log(self.t) / self.counts[i])
            for i in range(self.k)
        ]
        return max(range(self.k), key=lambda i: ucb[i])

    def update(self, arm, reward):
        self.counts[arm] += 1
        n = self.counts[arm]
        self.values[arm] += (reward - self.values[arm]) / n

def run_bandit(env, agent, T=1000):
    total = 0
    for _ in range(T):
        arm = agent.select()
        reward = env.pull(arm)
        agent.update(arm, reward)
        total += reward
    return total
```

> **Readable form:** Track pull counts and running averages; UCB adds exploration bonus inversely proportional to how often each arm was tried.

---

## Applications in AI Systems

- **Hyperparameter search:** each arm = configuration ([Course 1 Chapter 07](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/README.md))
- **Clinical trials:** ethical exploration
- **Recommendation systems:** cold-start users
- **Neural architecture search:** expensive evaluations favor UCB/BO

---

## Common Mistakes

**Never exploring** - greedy locks onto suboptimal arm.

**Constant $\varepsilon$ forever** - linear regret persists.

**Comparing arms with unequal sample sizes** without confidence bounds.

**Ignoring non-stationarity** - reward distributions drift; sliding windows needed.

---

## Reflection Questions

1. Why does UCB1 bonus grow for under-pulled arms even if $\hat{\mu}_i$ is low?
2. Prove that random exploration yields linear regret.
3. How is a bandit a special case of an MDP?
4. When is Thompson sampling preferable to UCB1?
5. How would you adapt bandit algorithms for delayed rewards?

---

**Previous:** [Section 17.5 - POMDPs](./section-05-partially-observable-mdps.md)  
**Next:** [Section 17.7 - MDP Applications](./section-07-mdp-applications.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *AIMA* (4th ed.), Section 17.3.
2. Auer, P., Cesa-Bianchi, N., & Fischer, P. (2002). Finite-time analysis of the multiarmed bandit problem.
3. Gittins, J. C. - dynamic allocation indices.
4. Lattimore, T., & Szepesvári, C. (2020). *Bandit Algorithms*.
5. Sutton & Barto (2018). *RL*, Section 2.6-2.7.
