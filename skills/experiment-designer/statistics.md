# Statistical Methods Reference

## Sample Size Formulas

All formulas use two-tailed tests with z-scores: z_alpha = Z(1 - alpha/2), z_beta = Z(power).

### Binary Metrics (Proportions Test)
```
n = 2 * (z_alpha + z_beta)^2 * p_pooled * (1 - p_pooled) / (p2 - p1)^2

where p_pooled = (p1 + p2) / 2
      p1 = baseline rate (decimal, e.g. 0.05 for 5%)
      p2 = p1 + absolute effect size
```
**Validation**: Both p1 and p2 must be in (0, 1). If a relative MDE pushes p2 outside this range, reduce the MDE or note that the effect size is unrealistic. For example, a 20% relative increase on a baseline of 0.90 gives p2 = 1.08, which is invalid.

### Continuous Metrics (Two-Sample T-Test)
```
n = 2 * (z_alpha + z_beta)^2 * variance / effect_size^2

where variance = sigma^2 (see typical CV table below if unknown)
      effect_size = baseline * (mde_pct / 100) for relative MDE
```
**Variance estimation**: If variance is unknown, estimate using `variance = (baseline * CV)^2` where CV is the coefficient of variation. The old default of CV=10% is too low for most metrics. Use these typical values:

| Metric type | Typical CV | Example |
|-------------|-----------|---------|
| Page load time, latency | 30-50% | 1500ms baseline → sd ~500-750ms |
| Revenue per user | 80-150% | $50 baseline → sd ~$40-75 |
| Session duration | 80-150% | 180s baseline → sd ~150-270s |
| Engagement scores | 25-40% | 7.5 baseline → sd ~2-3 |
| Order value | 40-60% | $75 baseline → sd ~$30-45 |

**Warning**: Using a CV that is too low will dramatically underestimate the required sample size. When in doubt, use a higher CV or request actual variance from the user's data.

### Count Metrics (Poisson Approximation)
```
n = 2 * (z_alpha + z_beta)^2 * lambda / effect_size^2

where lambda = baseline count rate
```

## Adjustments

### Unequal Traffic Allocation
When control/treatment split is not 50/50:
```
total_n = 2 * n * (1 + r)^2 / (4 * r)
n_control = total_n / (1 + r)
n_treatment = r * total_n / (1 + r)

where n = per-variant sample size from the balanced (50/50) formula
      r = treatment_allocation / control_allocation
```
For r=1 (balanced): total_n = 2n. For r=2 (67/33 split): total_n = 2.25n.

### Multiple Variants
When more than 2 variants (including control), apply a multiple testing correction
to alpha and recalculate sample size:
```
adjusted_alpha = alpha / (k - 1)              # Bonferroni correction
adjusted_z_alpha = Z(1 - adjusted_alpha / 2)  # larger z-score
n_adjusted = n * (adjusted_z_alpha + z_beta)^2 / (z_alpha + z_beta)^2

where k = number of variants (including control)
```
Example: k=3, alpha=0.05 → adjusted_alpha=0.025, z_alpha goes from 1.96 to 2.24,
n inflates by ~1.21x (not 2x). Use Holm or Benjamini-Hochberg for less conservative correction.

### Cluster Design Effect
For cluster randomized experiments:
```
DEFF = 1 + (m - 1) * ICC
adjusted_n = n * DEFF
clusters_per_arm = ceil(adjusted_n / m)

where m = average cluster size
      ICC = intra-cluster correlation (typically 0.01 - 0.1)
```

### Switchback Autocorrelation
For switchback experiments:
```
inflation_factor = (1 + rho) / (1 - rho)
adjusted_n = ceil(n * inflation_factor)
effective_periods = floor(num_periods / inflation_factor)

where rho = temporal autocorrelation (0 to 1)
```

### Factorial Cells
For factorial designs:
```
total_cells = product of all factor levels

Main effects only:
  n_per_cell: use base formula with adjusted_alpha = alpha / num_tests
  total_n = n_per_cell * total_cells

Detecting interactions:
  n_per_cell: use base formula with adjusted_alpha = alpha / num_tests
  total_n = n_per_cell * 4 * total_cells
  (4x factor: interaction contrast variance is 4x that of main effect contrast)

where num_tests = number of main effects + number of interactions
```

### Causal Inference: DiD Serial Correlation
```
VIF = (1 + rho) / (1 - rho)
adjusted_n = ceil(n * VIF)

where rho = first-order temporal autocorrelation of the outcome series within unit
            (estimate from residuals of a baseline-period regression of the
             outcome on unit fixed effects; AR(1) coefficient).
            Typical range 0.3-0.9 for daily count or revenue series.
            If rho is unknown, run sensitivity at rho ∈ {0.3, 0.6, 0.8}.
```

### MAB Exploration Budget
For epsilon-greedy multi-armed bandit:
```
explore_budget = ceil(horizon * epsilon)
per_arm_explore = ceil(explore_budget / num_arms)
estimated_regret = epsilon * horizon * (arms - 1) / arms * delta

where delta = mean reward gap between the best arm and a uniformly-chosen
              other arm, expressed in the same units as the reward
              (e.g. delta = 0.02 for a 2-pp CTR gap on binary rewards).
              For a worst-case bound, use the maximum possible reward
              (delta = 1 for 0/1 rewards) — but this overstates true regret
              when arms are close in performance.
```

For Thompson Sampling and UCB, regret is asymptotically `O(sqrt(K * T * log T))` —
fundamentally smaller than ε-greedy's `O(epsilon * T * K)` when ε is fixed.
Use simulation rather than closed-form for these strategies; see
[Russo et al. (2018), "A Tutorial on Thompson Sampling"](https://arxiv.org/abs/1707.02038).

## Duration Estimation

```
effective_daily_traffic = daily_traffic * (sum_of_allocation_pcts / 100)
days = ceil(total_sample_size / effective_daily_traffic)
weeks = ceil(days / 7)
```

Minimum recommended duration: **7 days** (to capture weekly patterns). Consider adding 2-3 buffer days for ramp-up/cool-down.

## MDE Feasibility Check

Given available traffic and max duration, calculate the achievable MDE:
```
available_n_per_variant = daily_traffic * max_days * allocation_pct
achievable_mde = (z_alpha + z_beta) * sqrt(2 * variance / available_n_per_variant)  # continuous
achievable_mde = (z_alpha + z_beta) * sqrt(2 * p * (1-p) / available_n_per_variant)  # binary
achievable_mde = (z_alpha + z_beta) * sqrt(2 * lambda / available_n_per_variant)     # count

where allocation_pct = fraction of traffic in the smallest variant (e.g. 0.5 for 50/50)
```

If achievable MDE > target MDE, the experiment is not feasible with current traffic. Suggest: increasing traffic allocation, extending max duration, or accepting a larger MDE.

## Common Warnings

| Condition | Warning |
|-----------|---------|
| n < 100 per variant | Very small sample. Results may be unreliable |
| MDE < 1% relative | Very small effect needs large traffic or long duration |
| Alpha > 0.05 | Risk of false positives increases |
| Power < 0.8 | Risk of false negatives increases |
| Clusters < 10 per arm | Fewer clusters = unreliable inference |
| Cluster size imbalance | Balance clusters for stable estimates |
| Duration < 7 days | May miss weekly patterns |
| Duration > 90 days | Consider if the experiment is worth running this long |

## Standard Defaults

| Parameter | Default | Notes |
|-----------|---------|-------|
| Alpha | 0.05 | 5% false positive rate (two-tailed) |
| Power | 0.80 | 80% chance of detecting a true effect |
| MDE | 5% relative | Smallest change worth detecting |
| Traffic allocation | 50/50 | Balanced split is most efficient |
| Variants | 2 | Control + treatment |

## One-Sided Tests

All formulas above assume two-tailed tests. For one-sided tests (directional hypothesis), use `z_alpha = Z(1 - alpha)` instead of `z_alpha = Z(1 - alpha/2)`.

```
Two-tailed (default): z_alpha = Z(1 - 0.05/2) = Z(0.975) = 1.96
One-sided:            z_alpha = Z(1 - 0.05)   = Z(0.95)  = 1.645
```

**When one-sided is appropriate**:
- You have a strong directional hypothesis (e.g. "new checkout flow will increase conversion, not decrease it")
- You only care about detecting an effect in one direction
- You will NOT act differently if the effect is in the opposite direction

**When to stick with two-tailed (default)**:
- You want to detect effects in either direction
- A negative result would also be actionable (e.g. you'd kill the experiment)
- You are unsure about the direction of the effect
- Regulatory or organizational standards require two-tailed tests

**Impact**: One-sided tests require ~20% smaller sample size than two-tailed for the same alpha and power.

## Ratio Metrics

Many important metrics are ratios: revenue per session, clicks per impression, orders per visit. Standard formulas assume independent observations, but ratio metrics have correlated numerator and denominator.

### Delta Method (Recommended)
For a ratio metric Y/X where Y = numerator (e.g. revenue) and X = denominator (e.g. sessions):
```
ratio = mean(Y) / mean(X)

variance_ratio = (1 / mean(X)^2) * (var(Y) + ratio^2 * var(X) - 2 * ratio * cov(Y, X))
```

Use `variance_ratio` in place of `variance` in the continuous metric sample size formula.

### Linearization (Alternative)
Transform the ratio into a per-user metric:
```
linearized_i = Y_i - ratio * X_i
```
Then compute sample size using `var(linearized)` as the variance. This is equivalent to the delta method but easier to implement in practice.

### Bootstrap (When Delta Method Assumptions Fail)
If the ratio distribution is highly skewed or has heavy tails, use bootstrap to estimate variance:
1. Resample users with replacement (1,000+ iterations)
2. Compute the ratio metric for each bootstrap sample
3. Use the bootstrap variance in the sample size formula

**Warning**: Using `var(Y_i / X_i)` (variance of per-user ratios) is incorrect when users have different numbers of observations. Always use the delta method or linearization.

**See also**: [Simulation-Based Power](#simulation-based-power) — ratio metrics are a primary case where closed-form power estimates understate true sample requirements.

## Group Sequential Testing

### Alpha-Spending Boundaries

For group sequential designs with K planned looks:

**O'Brien-Fleming boundaries** (approximate):
```
z_k = z_K / sqrt(t_k)
where t_k = k/K (information fraction at look k)
      z_K ≈ z_alpha for the final look
```
This `z_K / sqrt(t_k)` form is a closed-form approximation; exact OBF boundaries solve a recursive integral and are best obtained from `gsDesign` (R) or `statsmodels.stats.proportion` (Python). The approximation is tightest at the final look and slightly conservative at very early looks.

Example (K=4, alpha=0.05, two-sided, from gsDesign):
```
| Look | t_k  | z_k   | p_k     |
|------|------|-------|---------|
| 1    | 0.25 | 4.049 | 0.00005 |
| 2    | 0.50 | 2.863 | 0.0042  |
| 3    | 0.75 | 2.337 | 0.0194  |
| 4    | 1.00 | 2.024 | 0.0430  |
```

**Pocock boundaries** (K=4, alpha=0.05):
```
z_k = 2.361 for all k
p_k = 0.0182 for all k
```

### Maximum Sample Size Inflation

```
n_max = n_fixed * inflation_factor

| K | OBF     | Pocock  |
|---|---------|---------|
| 2 | 1.014   | 1.101   |
| 3 | 1.022   | 1.166   |
| 4 | 1.027   | 1.211   |
| 5 | 1.031   | 1.244   |
```

### Futility Boundaries

Non-binding futility using conditional power:
```
conditional_power = 1 - Phi(z_final - z_current * sqrt(t_max / t_current))
Stop for futility if conditional_power < 0.20
```

## Bayesian A/B Formulas

### Beta-Binomial (rates)

For binary outcomes with prior `Beta(α0, β0)`:
```
posterior_control    = Beta(α0 + successes_c, β0 + failures_c)
posterior_treatment  = Beta(α0 + successes_t, β0 + failures_t)
```

**Probability treatment beats control**:
```
P(θ_t > θ_c) = integral over the joint posterior
            ≈ Monte Carlo: sample N times from each posterior, count fraction where t > c
```

**Expected loss** (the probability-weighted magnitude of picking the wrong arm):
```
E[loss | ship treatment]  = E[max(0, θ_c − θ_t)]
E[loss | ship control]    = E[max(0, θ_t − θ_c)]
Ship the arm with lower expected loss, provided loss < tolerance (e.g. 0.5% of baseline).
```

### Normal-Normal (continuous means)

For continuous outcomes with known variance σ² and prior `Normal(μ0, τ0²)`:
```
posterior = Normal(μ_post, τ_post²)
  where τ_post² = 1 / (1/τ0² + n/σ²)
        μ_post  = τ_post² * (μ0/τ0² + sum(y)/σ²)
```

For unknown variance, use Normal-Inverse-Gamma conjugate or numerical MCMC.

### Prior Recommendations

| Context | Prior | Notes |
|---------|-------|-------|
| Default / no strong beliefs | Beta(1,1) or Normal(0, large σ²) | Near-frequentist behavior |
| Rare events (conversion < 2%) | Beta(2, 98) — centered at 2% | Stabilizes estimates, lightly informative |
| Drawing from past experiment distribution | Empirical Bayes: fit α,β to past experiment effect distribution | Best when 10+ past experiments exist |
| Regulated / skeptical | Weakly informative centered at 0 | Requires strong evidence to move posterior |

### Sample Size (approximate)

Closed-form Bayesian sample size is rarely clean — use simulation:
1. Fix priors, fix decision rule (e.g. ship if E[loss] < ε AND P(beat) > 0.95).
2. Simulate from the alternative hypothesis (true lift = MDE) with n per arm.
3. Compute posterior, apply decision rule.
4. Repeat 1,000+ times; fraction reaching correct decision = Bayesian power.
5. Binary search over n until power = 0.8.

For a rough frequentist-equivalent starting point, use the base binary/continuous formula — posterior-based decisions and p-value-based decisions are close under weak priors.

## Equivalence / Non-Inferiority (TOST)

### Test structure

Two one-sided tests against the equivalence margin ±δ:
```
H0_lower: Δ ≤ −δ    vs H1_lower: Δ > −δ
H0_upper: Δ ≥ +δ    vs H1_upper: Δ < +δ
Reject H0 (declare equivalence) if BOTH one-sided tests reject at level α.
```

Equivalently, the (1 − 2α) two-sided CI for Δ must be fully contained in [−δ, +δ].

**Non-inferiority**: Only the lower bound matters:
```
H0: Δ ≤ −δ   vs H1: Δ > −δ
Reject H0 (declare non-inferiority) if the one-sided test rejects at level α.
```

### Sample Size

```
n = 2 * (z_alpha + z_beta)^2 * sigma^2 / (delta - abs(Delta))^2

where delta = equivalence margin
      Delta = expected true difference (often 0 for equivalence, small for non-inferiority)
      sigma = standard deviation of the metric
      z_alpha = Z(1 - alpha)         # one-sided
      z_beta  = Z(power)
```

For binary metrics, replace `sigma^2` with `p * (1 - p)` (the leading `2 *` already accounts for the two-arm structure).

**Feasibility**: If δ is small and expected |Δ| is close to δ, the denominator `(δ − |Δ|)` becomes tiny and n explodes. Run feasibility before committing.

**Standard convention**: Report the 90% CI when testing equivalence at α=0.05 per side (equivalent to two-sided 90% coverage). Report 95% one-sided CI for non-inferiority at α=0.05.

### Choosing δ

The equivalence margin is the hardest design choice. Framework:
1. **Regulatory**: If regulated, a fixed margin is often prescribed (e.g. FDA: 1.5× hazard ratio).
2. **Business cost**: What decrement in the metric would change a downstream decision? Set δ to that value.
3. **Historical variability**: δ should be smaller than the variability between historical experiments on the same metric.
4. **Sanity check**: If δ > typical MDE, the test is trivial; if δ < 0.1 × typical MDE, n will be infeasible.

## Always-Valid Inference / mSPRT

### Confidence Sequence (mixture SPRT)

For a one-parameter test with mixture prior `f(θ) = Normal(0, σ_tune²)`:
```
M_n = integral of exp(sum_i log p(X_i | θ) − log p(X_i | 0)) * f(θ) dθ
```
`M_n` is a martingale under H0; the level-α boundary is `M_n ≥ 1/α`.

**Confidence sequence**:
```
CS_n = {θ : M_n(θ) < 1/α}
```
The coverage probability `P(θ_true ∈ CS_n for all n)` ≥ 1 − α for any sample size — i.e. anytime-valid.

### Sample-Size Inflation

Compared to a fixed-sample test at the same α and power, mSPRT requires roughly:
```
n_mSPRT ≈ n_fixed * [1 + 2 * log(1/alpha) / (z_alpha + z_beta)^2]    # loose upper bound
```
Plugging in α=0.05, power=0.8: closed form gives ~76% inflation, but this is a loose worst-case bound — empirical inflation under properly-tuned `σ_tune` is **10-30%** (Johari et al. 2022, simulation). Use the closed-form value as a hard cap for budgeting; expect actual stopping to occur much earlier. For precise sizing, simulate against historical data with the planned mixture prior.

### Mixture prior choice

- `σ_tune` should be on the order of the expected effect size.
- Wider `σ_tune` = robust to small effects but slower stopping.
- Tighter `σ_tune` = faster stopping if effect ≈ σ_tune, slower if far from it.
- Rule of thumb: set σ_tune = MDE.

**Reference**: Johari, Koomen, Pekelis, Walsh (2022), "Always Valid Inference: Continuous Monitoring of A/B Tests", *Management Science*.

## Simulation-Based Power

Closed-form power formulas assume iid observations, near-normal distributions, and simple-mean estimators. When any assumption breaks, simulate:

### Procedure

```
1. Obtain historical data representative of the experiment population.
2. Define the DGP (data-generating process):
   - null:  resample with replacement from historical data (no effect).
   - alt:   resample AND apply the hypothesized effect (e.g. multiply revenue by 1 + MDE in one arm).
3. For each simulated experiment:
   a. Draw n_per_arm users per arm according to the DGP.
   b. Apply the planned analysis exactly (CUPED, cluster-robust SE, ratio-metric delta method, etc.).
   c. Record whether the test would reject H0.
4. Repeat 1,000+ times (10,000+ for precise tail behavior).
5. Power = fraction of alt simulations that reject.
   Type-I rate = fraction of null simulations that reject (should be ≤ alpha).
6. Binary-search over n to hit target power.
```

### When to use

| Situation | Why simulate |
|-----------|-------------|
| Ratio metrics (revenue/session) | Delta-method variance is approximate; simulation verifies |
| CUPED / regression adjustment | Effective variance depends on covariate correlation; closed forms are rough |
| Cluster-robust or HC3 SE | Finite-sample behavior differs from asymptotic |
| Heavy-tailed continuous (revenue, duration) | Normal approximation may be poor at moderate n |
| Bayesian decision rules | Closed-form Bayesian power is usually unavailable |
| Novel / custom estimators | No closed form exists |

### Validation

Verify the simulation code reproduces a known closed-form result before trusting it on the novel case: run with iid normal data and a simple t-test; the simulated power should match the formula within Monte Carlo noise.

### Interrupted Time Series Sample Size

For ITS, the "sample" is time periods. Rough rule:
```
Required pre-periods ≈ 20+
Required post-periods ≈ 10+
Power depends on: effect size, autocorrelation in the series, and presence of seasonality.
```
Use simulation with historical data + injected intervention to estimate power directly.

### Interleaving Sample Size

For interleaving experiments comparing two ranking systems:
```
n = (z_alpha + z_beta)^2 / (2 * (target_win_rate - 0.50)^2)

where target_win_rate = expected win rate for the better system (typically 0.51-0.55)
      baseline = 0.50 (equal preference)
```
This uses the binary metric formula with baseline p = 0.50 and effect size = (target - 0.50).

## Choosing MDE (Minimum Detectable Effect)

MDE is the hardest parameter for most users. Use this framework:

### Business Impact Approach
1. **Calculate the revenue impact**: If the metric moves by X%, what is the annual revenue/business impact?
2. **Set MDE to the smallest X% where the impact justifies the cost** of building, maintaining, and running the experiment.
3. Example: If 1% conversion lift = $500K/year in revenue, and the feature costs $100K to build, then an MDE of 1% is justified.

### Historical Approach
1. **Look at past experiments**: What effect sizes have similar experiments detected?
2. **Use the median observed effect** as a realistic MDE. Most experiments see 1-5% relative lifts.
3. If past experiments in this area typically show 2-3% lifts, set MDE to ~2%.

### Practical Rules of Thumb

| Experiment type | Typical MDE range | Notes |
|----------------|-------------------|-------|
| UI/UX changes | 2-5% relative | Small visual changes rarely move metrics by >5% |
| New features | 5-15% relative | Significant changes can have larger effects |
| Algorithm changes | 1-3% relative | Algorithmic improvements are often small but valuable |
| Pricing changes | 5-20% relative | Pricing has large but uncertain effects |
| Content optimization | 10-30% relative | Content variations can have wide effect ranges |

### Feasibility Check
If the calculated duration is too long for your target MDE:
- Can you **accept a larger MDE**? (detect only large effects)
- Can you **increase traffic allocation**? (more users in the experiment)
- Can you **use variance reduction** (CUPED)? (reduce required n by 20-50%)
- Can you **use a more sensitive design**? (interleaving for ranking, sequential for early stopping)

## Effect Size Interpretation

### Cohen's Conventions
| Effect size (Cohen's d) | Interpretation | Example |
|------------------------|----------------|---------|
| 0.01-0.2 | Very small to small | Typical UI tweaks, minor copy changes |
| 0.2-0.5 | Small to medium | Feature redesigns, algorithmic improvements |
| 0.5-0.8 | Medium to large | Major product changes, new features |
| > 0.8 | Large | Fundamental changes, pricing overhauls |

### Computing Cohen's d
```
d = (mean_treatment - mean_control) / pooled_sd

Equal arms:
  pooled_sd = sqrt((sd_treatment^2 + sd_control^2) / 2)

Unequal arms (n_t ≠ n_c):
  pooled_sd = sqrt(((n_t - 1) * sd_t^2 + (n_c - 1) * sd_c^2) / (n_t + n_c - 2))
```
Use the unequal form whenever sample sizes differ by >10% (e.g. holdouts, geo, any unbalanced allocation).

### Practical vs Statistical Significance

A result can be:
- **Statistically significant AND practically significant**: Ship it ✓
- **Statistically significant but NOT practically significant**: Effect is real but too small to matter. Consider: is the implementation cost worth a 0.1% lift?
- **Not statistically significant but potentially practically significant**: Underpowered experiment. Consider: extend the experiment, increase traffic, or use variance reduction.
- **Neither**: No evidence of a meaningful effect. Kill it ✗

Always report confidence intervals alongside p-values. A 95% CI of [0.1%, 5.2%] is much more informative than "p < 0.05".
