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
adjusted_n = n * (1 + r)^2 / (4 * r)

where r = treatment_allocation / control_allocation
```

### Multiple Variants
When more than 2 variants (including control):
```
adjusted_n = n * (k - 1)

where k = number of variants
```
Consider multiple testing correction (Bonferroni, Holm, or Benjamini-Hochberg).

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
total_n = n_per_cell * total_cells

If detecting interactions: interaction_n = n_per_cell * 4 * total_cells
```

### Causal Inference: DiD Serial Correlation
```
VIF = (1 + rho) / (1 - rho)
adjusted_n = ceil(n * VIF)
```

### MAB Exploration Budget
For epsilon-greedy multi-armed bandit:
```
explore_budget = ceil(horizon * epsilon)
per_arm_explore = ceil(explore_budget / num_arms)
estimated_regret = epsilon * horizon * (arms - 1) / arms
```

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
