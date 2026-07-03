# Section 20.7: Structure Learning

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 20.6  
> **Previous:** [Section 20.6 - Bayesian Network Parameter Learning](./section-06-bayesian-network-parameter-learning.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Learning the DAG, Not Just the Numbers

[Section 20.6](./section-06-bayesian-network-parameter-learning.md) assumed a known BN **structure** (which variables are parents of which). **Structure learning** searches over directed acyclic graphs (DAGs) to find a graph $G$ that best explains data - subject to acyclicity and statistical identifiability constraints.

> **Readable form:** Structure learning asks "which arrows should the Bayesian network have?" - not only "what numbers go in the CPTs?"

Humorous analogy: parameter learning tunes the **volume knobs**; structure learning rewires the **studio** - wrong topology and no amount of knob-twisting captures the music.

---

## Why Structure Matters

| Correct structure | Wrong structure |
|-------------------|-----------------|
| Efficient inference | May need dense model |
| Meaningful causality hints | Spurious dependencies |
| Generalizes with fewer parameters | Overfits correlations |

[Chapter 13](../chapter-13-probabilistic-reasoning/README.md) used expert-designed networks (alarm, medical). Structure learning **discovers** graphs from observational data - harder and more controversial for **causal** claims.

---

## Two Families of Methods

| Approach | Idea | Examples |
|----------|------|----------|
| **Score-based** | Search DAGs maximizing a scoring function | Hill-climbing + BIC |
| **Constraint-based** | Statistical tests for conditional independence | PC algorithm |

Hybrid pipelines are common: constraint-based skeleton + score-based orientation.

---

## Score-Based Structure Learning

Define score $S(G : \mathcal{D})$ - higher is better.

**Search problem:**

$$
G^* = \arg\max_{G \text{ acyclic}} S(G : \mathcal{D})
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

Search space is **superexponential** in number of variables - heuristic search required.

### Common search operators

| Move | Action |
|------|--------|
| **Add edge** | $X \to Y$ if acyclic |
| **Delete edge** | Remove dependency |
| **Reverse edge** | Flip direction if acyclic |

**Hill-climbing:** start from empty or random DAG, apply best improving move until local optimum. **Random restarts** mitigate local optima.

```python
# Pseudocode: score-based structure learning
G = empty_dag()
best_score = score(G, data)
while True:
    neighbors = {add_edge, delete_edge, reverse_edge}(G)
    G_best, s_best = argmax_{G' in neighbors} score(G', data)
    if s_best <= best_score:
        break
    G, best_score = G_best, s_best
return G
```

---

## Bayesian Information Criterion (BIC)

Popular score balancing fit and complexity:

$$
\text{BIC}(G : \mathcal{D}) = \log P(\mathcal{D} \mid \hat{\theta}_G, G) - \frac{d_G}{2} \log N
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

| Term | Meaning |
|------|---------|
| $\hat{\theta}_G$ | MLE parameters for graph $G$ |
| $d_G$ | Number of free parameters in CPTs |
| $N$ | Sample size |

> **Readable form:** BIC = log-likelihood of best-fit parameters minus penalty (half parameter count × log sample size) - favors simpler graphs unless richer ones fit much better.

From [Section 20.1](./section-01-statistical-learning.md): BIC approximates Bayesian marginal likelihood for large $N$.

**K2 score** (alternative): Bayesian score with Dirichlet priors, often used with variable orderings.

---

## Bayesian Model Selection View

$$
P(G \mid \mathcal{D}) \propto P(\mathcal{D} \mid G)\, P(G)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**Structure prior** $P(G)$ can encode sparsity (prefer fewer edges). Exact posterior over graphs is intractable for $n > \sim 20$ variables - MCMC over graph space (structure MCMC) samples high-probability DAGs.

---

## Constraint-Based Methods: PC Algorithm (Overview)

**Idea:** use conditional independence tests to remove edges.

1. Start with **complete undirected graph**.
2. For increasing order $k$, test $X \perp Y \mid \mathbf{Z}$ for $|\mathbf{Z}| = k$.
3. Remove edge $X$-$Y$ if independent given some $\mathbf{Z}$.
4. Orient edges using **v-structures** ($X \to Z \leftarrow Y$ with $X$, $Y$ non-adjacent) and propagation rules.

| Test | Data type |
|------|-----------|
| $\chi^2$ | Discrete |
| Partial correlation | Gaussian |

**Faithfulness assumption:** independences in data match graphical separations in true DAG. Violations (e.g., deterministic relationships) break PC.

---

## Conditional Independence and d-Separation

[Chapter 13](../chapter-13-probabilistic-reasoning/README.md) introduced **d-separation**: $X$ and $Y$ are d-separated by $\mathbf{Z}$ in $G$ iff all paths are blocked.

Structure learning **infers** $G$ such that empirical conditional independences align with d-separation statements.

| If data shows | Graph should |
|---------------|--------------|
| $X \perp Y \mid Z$ | Block path via $Z$ |
| $X \not\perp Y$ | Connect directly or via collider |

---

## Identifiability and Markov Equivalence

Multiple DAGs can encode the **same** set of conditional independences - **Markov equivalence class**.

| Same v-structures | May differ edge directions among non-colliders |
|-------------------|-----------------------------------------------|

**Causal direction** is generally **not** identifiable from observational data alone - need interventions, temporal order, or background knowledge.

| Evidence type | Orientation power |
|---------------|-------------------|
| Observational | Equivalence class |
| Interventional | Stronger causal ID |

---

## Combining Expert Knowledge

Practical BN construction ([Chapter 13](../chapter-13-probabilistic-reasoning/README.md)) often **hybridizes**:

| Source | Role |
|--------|------|
| Domain experts | Forbid impossible edges, require causal arcs |
| Data | Score / test among remaining candidates |
| Literature | Inform priors on structure |

**Tiered ordering:** variables ordered by time - search only forward in time (efficient, causal).

---

## Structure + Parameters: Chicken and Egg

Typical loop:

```
1. Propose structure G
2. Learn parameters θ̂_G (Section 20.6)
3. Score S(G : D)
4. Modify G; repeat
```

With latent variables, inner loop needs [EM](./section-04-em-algorithm.md) - **structural EM** alternates structure search and parameter fitting.

---

## Comparison Table

| Criterion | Score-based | Constraint-based |
|-----------|-------------|------------------|
| **Output** | Single DAG (local optimum) | Equivalence class skeleton |
| **Sample size** | Needs moderate $N$ | Tests unstable if $N$ small |
| **Hidden variables** | Extensions needed | Hard |
| **Computation** | Search-intensive | Test-intensive |

---

## Example: Three Variables

Variables: $A, B, C$. Observational data suggests $A \perp C \mid B$.

| Graph candidates | Compatible? |
|------------------|-------------|
| $A \to B \to C$ | Yes |
| $A \leftarrow B \leftarrow C$ | Yes |
| $A \to B \leftarrow C$ | Yes (v-structure at $B$) |
| $A \to B \to C$ and $A \to C$ | No - would imply dependence |

Score-based search picks highest BIC among acyclic options.

---

## Overfitting Structure

Dense graphs fit training data tightly. **BIC penalty** reduces but does not eliminate overfitting when $N$ is small.

| Symptom | Remedy |
|---------|--------|
| Too many edges | Stronger sparsity prior |
| Unstable across bootstrap samples | Aggregate consensus graphs |
| Implausible arrows | Expert constraints |

[Cross-validation](../../GLOSSARY.md#cross-validation) on held-out log-likelihood evaluates predictive structure.

---

## Connection to Course 1

| Course 1 | Structure learning |
|----------|-------------------|
| Feature selection | Choosing parents ≈ selecting relevant predictors |
| [Overfitting](../../GLOSSARY.md#overfitting) | Dense DAGs |
| Correlation vs. causation | Orientation limits |

---

## Connection to Course 3

[Structured probabilistic models](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-16-structured-probabilistic-models/README.md) and causal discovery research extend score-based ideas to nonlinear and high-dimensional settings - neural causal models, continuous optimization over adjacency.

---

## Software and Practice

Libraries (bnlearn, pgmpy, Tetrad) implement PC, hill-climbing, tabu search. Always:

1. Check acyclicity.
2. Compare multiple random starts.
3. Validate on held-out data.
4. Review edges with domain experts.

---

## Limitations (AIMA Emphasis)

| Limitation | Detail |
|------------|--------|
| **NP-hard** | Optimal structure search is combinatorial |
| **Faithfulness** | Real data may violate |
| **Latent confounders** | Hidden common causes mislead |
| **Finite samples** | Tests / scores noisy |

[Section 20.8](./section-08-nonparametric-bayes-preview.md) relaxes fixed structure assumptions in mixture context.

---

## Reflection Questions

1. Why does BIC penalize the number of parameters in a BN?
2. What is a v-structure and why does it constrain edge orientation?
3. When do score-based and constraint-based methods disagree?
4. Can observational data alone prove $X \to Y$ vs. $Y \to X$?
5. How does hill-climbing structure search relate to model selection in [Chapter 19](../chapter-19-learning-from-examples/README.md)?

---

**Next:** [Section 20.8 - Nonparametric Bayes Preview](./section-08-nonparametric-bayes-preview.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Section 20.6. [AIMA book site](https://aima.cs.berkeley.edu/)
2. Koller, D., & Friedman, N. (2009). *Probabilistic Graphical Models*, Chapter 18 - structure learning. [MIT Press](https://mitpress.mit.edu/)
3. Spirtes, P., Glymour, C., & Scheines, R. (2000). *Causation, Prediction, and Search* (2nd ed.). [MIT Press](https://mitpress.mit.edu/)
4. Heckerman, D., Geiger, D., & Chickering, D. M. (1995). Learning Bayesian networks: The combination of knowledge and statistical data. *Machine Learning*. [Springer](https://link.springer.com/)
5. Murphy, K. P. (2022). *Probabilistic Machine Learning*, Chapter 26 - structure learning. [probml.ai](https://probml.ai/)



