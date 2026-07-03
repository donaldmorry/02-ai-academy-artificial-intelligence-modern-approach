# Section 22.4: Generalization in Reinforcement Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 22.4  
> **Previous:** [Section 22.3 - Active RL](./section-03-active-reinforcement-learning.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## When Tabular RL Breaks

[Q-learning](./section-03-active-reinforcement-learning.md) stores one number per $(s,a)$ pair. For a grid world with 100 states and 4 actions, that is 400 entries - manageable. For:

- Chess: $\sim 10^{47}$ states
- Go: $\sim 10^{170}$ states  
- Atari: $210 \times 160 \times 3$ pixels per frame
- Continuous control: infinite states in $\mathbb{R}^n$

tabular methods are **impossible**. The agent must **generalize** - share learned value across similar states.

> **Readable form:** Tabular RL is like memorizing a separate phone number for every person on Earth. Generalization is learning patterns - area codes, country codes - so new numbers need less memorization.

[Chapter 17](../chapter-17-making-complex-decisions/README.md) value iteration over explicit states faces the same curse of dimensionality. Function approximation replaces the table with a parameterized estimator - connecting RL to [supervised learning](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-03-supervised-vs-unsupervised-learning.md) and [neural networks](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/README.md).

---

## Function Approximation for Value Functions

Approximate $Q(s,a)$ or $V(s)$ with a function $\hat{Q}(s,a; \mathbf{w})$ or $\hat{V}(s; \mathbf{w})$ with parameters $\mathbf{w}$.

**Linear approximation:**

$$
\hat{Q}(s, a; \mathbf{w}) = \mathbf{w}^\top \mathbf{x}(s, a)
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

where $\mathbf{x}(s,a)$ is a **feature vector** - hand-crafted or learned.

**Nonlinear approximation:**

$$
\hat{Q}(s, a; \mathbf{w}) = f_\mathbf{w}(s, a) \quad \text{(neural network)}
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

TD update with function approximation:

$$
\mathbf{w} \leftarrow \mathbf{w} + \alpha \left[ r + \gamma \hat{Q}(s', a'; \mathbf{w}) - \hat{Q}(s, a; \mathbf{w}) \right] \nabla_\mathbf{w} \hat{Q}(s, a; \mathbf{w})
$$
> **Readable form:** Same TD error as tabular methods, but the update changes weights in the direction that would increase $\hat{Q}(s,a)$ - gradient descent on the squared TD error.

This is **semi-gradient** TD - not the true gradient of a fixed objective, because the target depends on $\mathbf{w}$.

---

## Tile Coding and Coarse Coding

**Tile coding** (Sutton) is a classic linear function approximator for continuous or large discrete spaces.

- Overlay multiple **tilings** (grids offset from each other) on state variables
- Each tile is one binary feature (1 if state falls in tile, else 0)
- Sparse features → efficient updates, local generalization

```
Tiling 1:  |██|   |   |
Tiling 2:    |██|   |   |
Tiling 3:      |██|   |   |
State ● activates one tile per tiling → 3 active features
```

> **Readable form:** Tile coding groups nearby states into overlapping buckets. Learning in one bucket nudges values for nearby states covered by neighboring tiles.

**Coarse coding** uses overlapping radial basis functions - Gaussian bumps centered on prototypical states.

| Method | Generalization | Parameters |
|--------|----------------|------------|
| Tabular | None | $|\mathcal{S}| \times |\mathcal{A}|$ |
| Tile coding | Local, smooth | # tilings × tiles per tiling |
| Neural net | Global, hierarchical | Millions |

---

## The Deadly Triad

Sutton warns that **off-policy learning + function approximation + bootstrapping** can diverge - the **deadly triad**.

| Ingredient | Example |
|------------|---------|
| Function approximation | Neural $\hat{Q}$ |
| Bootstrapping | TD targets with $Q(s')$ |
| Off-policy | Q-learning, replay buffer |

Q-learning with linear features can oscillate or diverge. Mitigations:

- **On-policy** methods (SARSA, A2C)
- **Experience replay** with stable targets (DQN)
- **Gradient TD** family (GTD, TDC) - true gradient methods
- Conservative updates, target networks

> **Readable form:** Combining "guess the future" (bootstrapping), "learn from old random behavior" (off-policy), and "share weights across states" (function approximation) can make learning unstable. Deep RL engineers many workarounds.

---

## Feature Engineering vs. Representation Learning

**Hand-crafted features** (tile coding, polynomial bases):

- Pros: interpretable, stable, low data
- Cons: require domain expertise; poor on raw pixels

**Representation learning** ([Chapter 21](../chapter-21-deep-learning/README.md)):

- CNN layers extract visual features for Atari
- End-to-end $\hat{Q}(s,a)$ with $s$ as image stack

[Course 3 Chapter 06](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md) covers backprop; [Course 3 Chapter 09](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-09-convolutional-networks/README.md) covers convolutions for spatial input - both feed into [Section 22.5](./section-05-deep-reinforcement-learning.md).

---

## Linear vs. Nonlinear: When to Use What

| Setting | Recommendation |
|---------|----------------|
| Low-dimensional continuous control | Tile coding, linear FA |
| Raw pixels / high-D observations | Deep Q-network |
| Small discrete MDP (verified) | Tabular Q-learning |
| Safety-critical, need guarantees | Tabular or verified linear |

[Course 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-02-regression-models/section-03-decision-trees-for-regression.md) decision trees generalize by partitioning feature space - analogous intuition, but RL targets are non-stationary TD errors, not fixed labels.

---

## Kernel Methods and Nearest Neighbors

**Kernel-based RL** generalizes from stored exemplar states weighted by similarity $k(s, s_i)$.

**$k$NN Q-learning:** store transitions; estimate $Q(s,a)$ from $k$ nearest stored states where $a$ was taken.

Simple but scales poorly; useful for small continuous domains and teaching.

---

## Generalization in Policy Space

Function approximation applies to **policies** too:

$$
\pi_\mathbf{w}(a \mid s) = \text{softmax}(f_\mathbf{w}(s, a))
$$
> **Readable form:** Softmax turns raw scores into a normalized distribution, so larger logits receive more probability mass.

[Section 22.6](./section-06-policy-search-and-policy-gradients.md) develops **policy gradients** - often more stable with neural nets than Q-learning for continuous actions.

| Approach | Approximates | Action space |
|----------|--------------|--------------|
| Value-based FA | $Q(s,a)$ | Discrete (via $\arg\max$) |
| Policy-based FA | $\pi(a \mid s)$ | Discrete or continuous |

---

## Overfitting in RL

Unlike [supervised learning](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-06-the-ml-workflow-and-data-hygiene.md), RL lacks a fixed dataset - the agent **creates** its own distribution. Overfitting manifests as:

- **Memorizing** training environments without transfer
- **Catastrophic forgetting** when distribution shifts
- **Overfitting to replay buffer** - stale experiences dominate

Mitigations: diverse exploration, regularization ([Course 3 Chapter 07](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-07-regularization/README.md)), domain randomization, evaluation on held-out environment seeds.

---

## Worked Example: Mountain Car with Tile Coding

Mountain Car: car in valley must rock back and forth to reach hilltop. State: $(\text{position}, \text{velocity})$ - continuous.

Features: 8 tilings × 8 tiles × 2 dimensions = sparse 512-bit vector (with bias).

$$
\hat{Q}(s, a) = \sum_i w_{a,i} \, x_i(s)
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

Update only weights for active tiles - $O(\#\text{tilings})$ per step, not $O(512)$.

```python
class TileCoder:
    def __init__(self, num_tilings, tiles_per_dim, dims):
        self.num_tilings = num_tilings
        self.tiles = tiles_per_dim
        self.dims = dims  # [(low, high), ...]

    def features(self, state):
        # returns list of active tile indices per tiling
        active = []
        for t in range(self.num_tilings):
            idx = self._hash(state, t)
            active.append(idx)
        return active

    def _hash(self, state, tiling_offset):
        # map continuous state + offset to tile index
        ...
```

---

## Connection to Chapter 17

Chapter 17 tabular value iteration assumes enumerable $\mathcal{S}$. Function approximation is the standard escape for **continuous state MDPs** - approximate:

$$
V_{k+1}(s) \approx \max_a \mathbb{E}[R + \gamma V_k(S') \mid s, a]
$$
> **Readable form:** The value expression summarizes expected discounted future reward from the current state or policy.

with $V_k$ represented by $\hat{V}(s; \mathbf{w})$.

POMDP belief states are continuous even when underlying states are finite - function approximation over belief simplices appears in point-based methods ([Chapter 17](../chapter-17-making-complex-decisions/README.md)).

---

## Stability Diagnostics

Monitor during training:

| Signal | Healthy | Unstable |
|--------|---------|----------|
| TD error magnitude | Decreasing trend | Exploding |
| Q-values | Bounded | Unbounded growth |
| Policy performance | Improving | Oscillating / collapsing |
| Weight norms | Stable | Diverging |

Early stopping on environment return - analogous to validation curves in [Course 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/section-03-classification-accuracy-measures.md).

---

## Common Mistakes

**Applying tabular Q-learning to images.** Never visits same pixel configuration twice.

**Ignoring deadly triad.** Naive neural Q-learning diverges without DQN tricks.

**Poor feature scaling.** Tile coding and neural nets need normalized state variables.

**$\arg\max$ over continuous actions.** Need discretization or policy gradients.

**Evaluating on training seeds only.** Apparent generalization fails on new random seeds.

---

## Reflection Questions

1. Why does tabular Q-learning fail on Atari from raw pixels?
2. What are the three elements of the deadly triad? Which does SARSA avoid?
3. How does tile coding achieve local generalization with sparse updates?
4. When is linear function approximation preferable to a deep network?
5. How does RL overfitting differ from supervised overfitting?

---

**Previous:** [Section 22.3 - Active RL](./section-03-active-reinforcement-learning.md) · **Next:** [Section 22.5 - Deep RL](./section-05-deep-reinforcement-learning.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), §22.4. [https://aima.cs.berkeley.edu/](https://aima.cs.berkeley.edu/)
2. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.), Ch. 9-11. [http://incompleteideas.net/book/](http://incompleteideas.net/book/)
3. Tsitsiklis, J. N., & Van Roy, B. (1997). "An analysis of temporal-difference learning with function approximation." *IEEE TAC*.
4. Sutton, R. S. (1996). "Generalization in reinforcement learning: Successful examples using sparse coarse coding." *NeurIPS*.
5. Mnih, V., et al. (2015). "Human-level control through deep reinforcement learning." *Nature* - DQN precursor. [https://www.nature.com/articles/nature14236](https://www.nature.com/articles/nature14236)
