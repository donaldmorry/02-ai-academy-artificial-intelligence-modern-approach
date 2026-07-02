# Lab 15: Bayesian Regression and Mixture Models

> **Prerequisites:** Sections [15.1](./section-01-relational-probability.md)-[15.7](./section-07-practical-workflow.md) | [Chapter 12](../chapter-12-quantifying-uncertainty/README.md) | [Chapter 13](../chapter-13-probabilistic-reasoning/README.md)  
> **Estimated time:** 10-12 hours  
> **Glossary:** [bayesian-inference](../../GLOSSARY.md#bayesian-inference) | [posterior](../../GLOSSARY.md#posterior) | [prior](../../GLOSSARY.md#prior)  
> **Math:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Objectives

By the end of this lab you will:

1. Fit **Bayesian linear regression** in **Stan** and **Pyro**
2. Compare posterior estimates to **OLS** analytical solution
3. Fit a **Gaussian mixture model** (GMM) with unknown component count
4. Compare mixture posteriors to **analytical** solutions where available
5. Run **convergence diagnostics** ($\hat{R}$, ESS, trace plots)
6. Perform **posterior predictive checks**

> **Humorous briefing:** You will make Stan and Pyro argue about the same dataset. If they agree, science works. If they disagree, you get to debug priors until they reconcile.

---

## Setup

```bash
python -m venv venv && source venv/bin/activate
pip install numpy pandas matplotlib seaborn scipy scikit-learn
pip install cmdstanpy pyro-ppl torch
```

Install CmdStan (required by cmdstanpy):

```python
import cmdstanpy
cmdstanpy.install_cmdstan()
```

```python
import warnings
warnings.filterwarnings('ignore')

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from pathlib import Path
from scipy import stats
from sklearn.linear_model import LinearRegression

RANDOM_STATE = 42
np.random.seed(RANDOM_STATE)
RESULTS_DIR = Path('lab15_results')
RESULTS_DIR.mkdir(exist_ok=True)
```

---

## Part A: Generate Synthetic Regression Data (20 min)

True data-generating process:

$$
y_i = \alpha + \beta x_i + \epsilon_i, \quad \epsilon_i \sim \mathcal{N}(0, \sigma^2)
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

```python
N = 50
TRUE_ALPHA, TRUE_BETA, TRUE_SIGMA = 2.0, 1.5, 0.8

x = np.linspace(0, 10, N)
y = TRUE_ALPHA + TRUE_BETA * x + np.random.normal(0, TRUE_SIGMA, N)

plt.figure(figsize=(8, 5))
plt.scatter(x, y, alpha=0.7)
plt.xlabel('x'); plt.ylabel('y'); plt.title('Synthetic regression data')
plt.savefig(RESULTS_DIR / 'regression_data.png', dpi=120)
plt.close()

pd.DataFrame({'x': x, 'y': y}).to_csv(RESULTS_DIR / 'regression.csv', index=False)
```

---

## Part B: OLS Baseline (30 min)

```python
X = x.reshape(-1, 1)
ols = LinearRegression().fit(X, y)
alpha_ols = ols.intercept_
beta_ols = ols.coef_[0]
sigma_ols = np.sqrt(np.mean((y - ols.predict(X))**2))

print(f'OLS: alpha={alpha_ols:.3f}, beta={beta_ols:.3f}, sigma={sigma_ols:.3f}')
print(f'True: alpha={TRUE_ALPHA}, beta={TRUE_BETA}, sigma={TRUE_SIGMA}')
```

**Uninformative-prior Bayesian limit:** with flat priors and large $N$, posterior means ≈ OLS.

---

## Part C: Stan Linear Regression (90 min)

Create `linear_regression.stan`:

```stan
data {
  int<lower=0> N;
  vector[N] x;
  vector[N] y;
}
parameters {
  real alpha;
  real beta;
  real<lower=0> sigma;
}
model {
  alpha ~ normal(0, 10);
  beta ~ normal(0, 10);
  sigma ~ exponential(1);
  y ~ normal(alpha + beta * x, sigma);
}
generated quantities {
  vector[N] y_pred;
  for (n in 1:N)
    y_pred[n] = normal_rng(alpha + beta * x[n], sigma);
}
```

Fit:

```python
import cmdstanpy

stan_data = {'N': N, 'x': x.tolist(), 'y': y.tolist()}
model = cmdstanpy.CmdStanModel(stan_file='linear_regression.stan')
fit = model.sample(
    data=stan_data,
    chains=4,
    parallel_chains=4,
    iter_warmup=500,
    iter_sampling=1000,
    seed=RANDOM_STATE
)

summary = fit.summary()
print(summary.loc[['alpha', 'beta', 'sigma'], ['Mean', 'StdDev', '5%', '95%']])

# Diagnostics
print('R-hat:', summary.loc[['alpha','beta','sigma'], 'R_hat'].max())
```

**Deliverable table:**

| Parameter | True | OLS | Stan mean | Stan 90% CI |
|-----------|------|-----|-----------|-------------|
| $\alpha$ | 2.0 | ... | ... | ... |
| $\beta$ | 1.5 | ... | ... | ... |
| $\sigma$ | 0.8 | ... | ... | ... |

Trace plot:

```python
fit.draws_pd(vars=['alpha', 'beta', 'sigma']).plot(subplots=True, figsize=(10, 6))
plt.tight_layout()
plt.savefig(RESULTS_DIR / 'stan_traces.png')
```

---

## Part D: Pyro Linear Regression (90 min)

```python
import torch
import pyro
import pyro.distributions as dist
from pyro.infer import MCMC, NUTS
from pyro.infer.mcmc.util import summary

pyro.set_rng_seed(RANDOM_STATE)

x_t = torch.tensor(x, dtype=torch.float32)
y_t = torch.tensor(y, dtype=torch.float32)

def regression_model(x, y=None):
    alpha = pyro.sample("alpha", dist.Normal(0., 10.))
    beta = pyro.sample("beta", dist.Normal(0., 10.))
    sigma = pyro.sample("sigma", dist.Exponential(1.))

    mean = alpha + beta * x
    with pyro.plate("data", len(x)):
        pyro.sample("obs", dist.Normal(mean, sigma), obs=y)

nuts_kernel = NUTS(regression_model)
mcmc = MCMC(nuts_kernel, num_samples=1000, warmup_steps=500, num_chains=4)
mcmc.run(x_t, y_t)
mcmc.summary()
```

Compare Pyro posterior means to Stan - should agree within Monte Carlo error.

**Optional SVI path:**

```python
from pyro.infer import SVI, Trace_ELBO
from pyro.optim import Adam
from pyro.infer.autoguide import AutoDiagonalNormal

guide = AutoDiagonalNormal(regression_model)
svi = SVI(regression_model, guide, Adam({"lr": 0.02}), loss=Trace_ELBO())
for _ in range(3000):
    svi.step(x_t, y_t)
```

Compare SVI vs MCMC - note possible variance underestimation.

---

## Part E: Posterior Predictive Check - Regression (45 min)

```python
# Stan: use generated quantities y_pred from draws
y_pred = fit.stan_variable('y_pred')  # shape: (draws, N)
y_ppc_mean = y_pred.mean(axis=0)
y_ppc_band = np.percentile(y_pred, [5, 95], axis=0)

plt.figure(figsize=(8, 5))
plt.scatter(x, y, label='observed', alpha=0.6)
plt.plot(x, y_ppc_mean, 'r-', label='posterior mean')
plt.fill_between(x, y_ppc_band[0], y_ppc_band[1], alpha=0.2, color='red')
plt.legend()
plt.savefig(RESULTS_DIR / 'regression_ppc.png')
```

**Question:** Do observed points lie within predictive bands? Systematic deviations → model misspecification.

---

## Part F: Mixture Data Generation (30 min)

Two-component Gaussian mixture:

$$
P(y) = 0.6 \cdot \mathcal{N}(-2, 0.5^2) + 0.4 \cdot \mathcal{N}(3, 1.0^2)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

```python
N_MIX = 300
z_true = np.random.choice([0, 1], size=N_MIX, p=[0.6, 0.4])
mu_true = np.array([-2.0, 3.0])
sigma_true = np.array([0.5, 1.0])

y_mix = np.array([
    np.random.normal(mu_true[z], sigma_true[z]) for z in z_true
])

plt.hist(y_mix, bins=30, density=True, alpha=0.7)
plt.title('Mixture data')
plt.savefig(RESULTS_DIR / 'mixture_data.png')
```

---

## Part G: Analytical Benchmark - Known Labels (45 min)

When component labels $z_i$ are **known**, each component mean is independent Gaussian posterior (conjugate with known $\sigma$) or use MLE:

```python
for k in [0, 1]:
    subset = y_mix[z_true == k]
    print(f'Component {k}: mean={subset.mean():.3f}, std={subset.std():.3f}')
    print(f'  True: mu={mu_true[k]}, sigma={sigma_true[k]}')
```

This is the **ceiling** for unsupervised recovery - unsupervised GMM should approximate these marginals.

---

## Part H: Stan Mixture Model (120 min)

`mixture.stan`:

```stan
data {
  int<lower=1> N;
  int<lower=1> K;
  vector[N] y;
}
parameters {
  simplex[K] pi;
  ordered vector[K] mu;
  vector<lower=0>[K] sigma;
}
model {
  pi ~ dirichlet(rep_vector(1.0, K));
  mu ~ normal(0, 5);
  sigma ~ exponential(1);
  for (n in 1:N)
    y[n] ~ normal_mixture(pi, mu, sigma);
}
```

```python
mix_data = {'N': N_MIX, 'K': 2, 'y': y_mix.tolist()}
mix_model = cmdstanpy.CmdStanModel(stan_file='mixture.stan')
mix_fit = mix_model.sample(data=mix_data, chains=4, iter_sampling=1000, iter_warmup=500, seed=RANDOM_STATE)

mix_summary = mix_fit.summary()
print(mix_summary.loc[['mu[1]','mu[2]','sigma[1]','sigma[2]','pi[1]'], ['Mean','5%','95%','R_hat']])
```

**Label switching:** `ordered vector[K] mu` enforces $\mu_1 < \mu_2$ - maps to true components.

Compare:

| Param | True | Known-label MLE | Stan posterior mean |
|-------|------|-----------------|---------------------|
| $\mu_1$ | -2.0 | ... | ... |
| $\mu_2$ | 3.0 | ... | ... |
| $\pi_1$ | 0.6 | ... | ... |

---

## Part I: Pyro Mixture Model (120 min)

```python
def gmm_model(y, K=2):
    pi = pyro.sample("pi", dist.Dirichlet(torch.ones(K)))
    mu = pyro.sample("mu", dist.Normal(0., 5.).expand([K]).to_event(1))
    sigma = pyro.sample("sigma", dist.Exponential(1.).expand([K]).to_event(1))

    with pyro.plate("data", len(y)):
        k = pyro.sample("z", dist.Categorical(pi))
        pyro.sample("obs", dist.Normal(mu[k], sigma[k]), obs=y)

y_mix_t = torch.tensor(y_mix, dtype=torch.float32)

# Use enumeration for discrete z
from pyro.infer import config
pyro_model = config.enumerate(gmm_model)
nuts_gmm = NUTS(pyro_model)
mcmc_gmm = MCMC(nuts_gmm, num_samples=1000, warmup_steps=500, num_chains=2)
mcmc_gmm.run(y_mix_t, K=2)
mcmc_gmm.summary()
```

Sort $\mu$ post-hoc for comparison to Stan ordered parameterization.

---

## Part J: Model Selection - K=2 vs K=3 (60 min)

Fit Stan mixture with `K=3`. Compare log density or use LOO:

```python
# pip install arviz
import arviz as az
idata_k2 = az.from_cmdstanpy(posterior=mix_fit)
# Compare with K=3 fit - lower LOO score preferred (with caution for multimodality)
```

**Written answer:** Does $K=3$ improve posterior predictive fit materially, or overfit?

---

## Part K: Posterior Predictive - Mixture (45 min)

```python
K = 2
pi_s = mix_fit.stan_variable('pi')
mu_s = mix_fit.stan_variable('mu')
sigma_s = mix_fit.stan_variable('sigma')

# One predictive draw
idx = 0
y_sim = []
for _ in range(N_MIX):
    k = np.random.choice(K, p=pi_s[idx])
    y_sim.append(np.random.normal(mu_s[idx, k], sigma_s[idx, k]))

plt.hist(y_mix, bins=30, density=True, alpha=0.5, label='observed')
plt.hist(y_sim, bins=30, density=True, alpha=0.5, label='posterior predictive')
plt.legend()
plt.savefig(RESULTS_DIR / 'mixture_ppc.png')
```

---

## Deliverables Checklist

- [ ] `lab15_results/regression.csv` + plots
- [ ] Comparison table: OLS vs Stan vs Pyro for regression
- [ ] Stan $\hat{R}$ < 1.01 for all regression parameters
- [ ] Mixture recovery table vs known labels
- [ ] Stan + Pyro mixture comparison
- [ ] 1-page reflection: when would RPM/OUPM (Sections 15.1-15.2) beat flat GMM for your data?

---

## Extension Challenges

1. **Hierarchical regression:** groups with partial pooling in Stan
2. **Robust regression:** Student-$t$ likelihood - outlier resistance
3. **Cognitive OCR toy:** implement Figure 15.11 generative model in Pyro (Poisson letters + noise)
4. **Citation mini-model:** two noisy strings, one latent paper - identity uncertainty

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| CmdStan not found | `cmdstanpy.install_cmdstan()` |
| Pyro shape errors | Check `plate` and `.to_event(1)` |
| Mixture label switching | Use `ordered mu` in Stan; sort in Pyro |
| Low ESS | Increase iterations; reparameterize |
| Stan divergences | `adapt_delta=0.95`; non-centered |

---

## References

- Russell & Norvig, AIMA 4e, Ch. 15
- [Sections 15.1-15.7](./section-01-relational-probability.md)
- [Stan User's Guide - regression](https://mc-stan.org/docs/stan_users_guide/regression.html)
- [Pyro - Bayesian Regression tutorial](https://pyro.ai/examples/bayesian_regression.html)
- [GLOSSARY.md](../../GLOSSARY.md)

---

**Previous:** [Section 15.7](./section-07-practical-workflow.md) | **Next:** [Chapter 16 - Making Simple Decisions](../chapter-16-making-simple-decisions/README.md)
