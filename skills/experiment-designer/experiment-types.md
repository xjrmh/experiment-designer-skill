# Experiment Types Reference

## A/B Test

**When to use**: Discrete changes, sufficient traffic (>1,000 users/day), need clear causal evidence, treatment effect is expected to be immediate.

**Use cases**: Testing new features or UI changes, comparing algorithms or ranking systems, optimizing conversion funnels, testing marketing copy.

**Pros**: Simple to understand and implement, clean causal inference, well-established statistical methods, easy to analyze.

**Cons**: Can only test one change at a time, requires sufficient traffic, may take time to reach significance.

**Defaults**: alpha=0.05, power=0.8, 2 variants, 50/50 split.

**Randomization**: USER_ID, HASH_BASED, consistent assignment.

**Examples**: New checkout flow vs current, two recommendation algorithms, green vs blue CTA button.

---

## Cluster Randomized Experiment

**When to use**: Treatment affects groups (cities, stores, schools), network effects or interference between users, marketplace experiments, geographic-based treatments. Need 20+ clusters.

**Use cases**: Driver incentive programs across cities, marketplace pricing by region, store-level promotions.

**Pros**: Handles network effects and spillover, suitable for group-level interventions, prevents contamination.

**Cons**: Requires larger sample sizes, fewer degrees of freedom, more complex analysis, need many clusters for power.

**Defaults**: alpha=0.05, power=0.8, 2 variants, 50/50 split.

**Randomization**: CLUSTER, RANDOM, consistent assignment.

**Key parameters**:
- **ICC (Intra-Cluster Correlation)**: Proportion of total variance that is between clusters (typically 0.01-0.1)
- **Cluster size**: Average number of units per cluster
- **Design Effect**: DEFF = 1 + (cluster_size - 1) x ICC. Inflates required sample size.

**Estimating ICC**: If historical data is available, compute ICC = variance_between_clusters / (variance_between_clusters + variance_within_clusters) using a one-way ANOVA or mixed-effects model on the primary metric grouped by cluster. If no data is available, use these rough guides:
- ICC ~0.01: Weak clustering (e.g. users in large cities)
- ICC ~0.05: Moderate clustering (e.g. students in schools, customers in stores)
- ICC ~0.10+: Strong clustering (e.g. patients in clinics, employees in small teams)

**Warnings**:
- Need at least 10 clusters per arm for reliable inference
- Balance clusters in size for stable estimates
- Larger ICC or cluster size = much larger sample sizes
- With cluster_size=100 and ICC=0.1, DEFF=10.9 — sample size inflates ~11x

---

## Switchback Experiment

**When to use**: Strong network effects, supply-demand coupling (marketplaces), user-level randomization causes issues, treatments can be switched quickly.

**Use cases**: Rideshare pricing algorithms, delivery dispatch systems, restaurant recommendations in food delivery.

**Pros**: Handles network effects well, all users experience both conditions, reduces time-of-day variance.

**Cons**: Carryover effects, longer duration, complex analysis, sensitive to non-stationarity.

**Defaults**: alpha=0.05, power=0.8, 2 variants, 50/50 split.

**Randomization**: SESSION, RANDOM, not consistent (users re-randomized each period).

**Key parameters**:
- **Number of periods**: Minimum 4 recommended
- **Period length**: Hours per switch (balance between more data and carryover risk)
- **Autocorrelation (rho)**: 0 to 1. Higher rho inflates required sample size.
- **Inflation factor**: (1 + rho) / (1 - rho). Multiplied into sample size.

**Estimating rho**: Compute the lag-1 autocorrelation of the primary metric across consecutive time periods using historical data. If unavailable, use rho ~0.3-0.5 for most marketplace metrics (conservative). Lower values (~0.1) for metrics with high natural variability across periods.

**Warnings**:
- Assumes no carryover effects between periods
- Minimum 7 days to capture weekly patterns
- Non-stationarity (trends, seasonality) can bias results

---

## Causal Inference (Observational)

**When to use**: Cannot run a randomized experiment, analyzing historical data or policy changes, natural experiments, studying long-term effects.

**Use cases**: Policy change impact across regions (DiD), eligibility threshold effects (RDD), effect of education on earnings (IV).

**Pros**: Can analyze past data without running an experiment, useful for post-hoc analysis, study effects over longer periods, lower cost.

**Cons**: Stronger assumptions required, risk of confounding bias, more complex statistical methods, results may be less definitive.

**Defaults**: alpha=0.05, power=0.8, 2 groups.

**Methods**:

| Method | Key Assumption | Best For |
|--------|---------------|----------|
| **Difference-in-Differences (DiD)** | Parallel trends between groups pre-treatment | Policy changes with pre/post data |
| **Regression Discontinuity (RDD)** | Continuity at the threshold | Eligibility cutoffs, score-based assignment |
| **Propensity Score Matching (PSM)** | Overlap in covariates, no unmeasured confounders | Observational data with rich covariates |
| **Instrumental Variables (IV)** | Valid, strong instrument (relevant + exogenous) | When direct randomization impossible |
| **Synthetic Control** | Untreated units can reconstruct the treated unit's counterfactual | Few treated units (1-5), many pre-treatment periods, aggregate data |

**Method-specific notes**:
- DiD: Serial correlation inflates variance. AR(1) VIF = (1 + rho) / (1 - rho)
- RDD: Sample size depends on bandwidth near threshold. Consult a statistician for bandwidth selection
- PSM: Effective sample size depends on match quality and covariate overlap
- IV: Weak instruments lead to biased estimates. Check first-stage F-statistic > 10
- Synthetic Control: Requires a donor pool of untreated units and a long pre-treatment period (typically 20+ time periods) for a good fit. Effect is estimated as the gap between the treated unit and its synthetic counterfactual post-treatment. Inference via placebo tests (apply the method to each donor unit as if it were treated — the treated unit's effect should be an outlier).

**Verifying assumptions**:
- **DiD — Parallel trends**: Plot the outcome for treatment and control groups over multiple pre-treatment periods. Trends should be visually parallel. Formally test with a placebo/pre-trend test (regress outcome on group x time interactions for pre-periods; coefficients should be non-significant).
- **RDD — Continuity at threshold**: Check that covariates do not jump at the cutoff (McCrary density test). Plot the outcome against the running variable; there should be no manipulation of the running variable near the threshold.
- **PSM — Overlap and no unmeasured confounders**: Check covariate balance after matching (standardized mean differences < 0.1). Overlap means both groups have similar propensity score distributions. The "no unmeasured confounders" assumption cannot be tested — argue it based on domain knowledge and conduct sensitivity analysis (e.g. Rosenbaum bounds).
- **IV — Instrument validity**: Relevance: first-stage F-statistic > 10. Exclusion: argue that the instrument affects the outcome only through the treatment (untestable, domain-knowledge based). If multiple instruments, use the Sargan/Hansen overidentification test.
- **Synthetic Control — Pre-treatment fit**: The synthetic control should closely replicate the treated unit's outcome in the pre-treatment period (low RMSPE). Check that weights are not concentrated on a single donor unit. Run placebo tests: apply the method to each untreated unit and check that the treated unit's effect is unusually large (p-value = rank of treated unit's effect / number of units). Tools: R `Synth` package, Python `SparseSC`, Google `CausalImpact` (Bayesian structural time-series variant).

---

## Factorial Design

**When to use**: Testing multiple independent changes simultaneously, interested in interaction effects, have sufficient traffic, changes are unlikely to conflict.

**Use cases**: 2x2 button color x copy length, email subject x send time x personalization, landing page hero x headline x CTA.

**Pros**: Test multiple factors efficiently, detect interaction effects, more information per user, faster than sequential A/B tests.

**Cons**: More complex analysis, requires larger sample sizes, harder to interpret with many factors, false positive risk increases.

**Defaults**: alpha=0.05, power=0.8, 4 variants (2x2), 25/25/25/25 split.

**Randomization**: USER_ID, HASH_BASED, consistent assignment.

**Key parameters**:
- **Factors**: Each with a name and number of levels (e.g. Button Color with 2 levels)
- **Total cells**: Product of all factor levels (e.g. 2 x 3 = 6 cells)
- **Detect interaction**: If yes, sample size inflated ~4x per cell

**Warnings**:
- With many factors, total cells grow fast (3 factors with 3 levels each = 27 cells)
- Verify factor independence before running
- Consider Bonferroni correction for multiple comparisons

---

## Multi-Armed Bandit (MAB)

**When to use**: Continuous optimization is priority, opportunity cost of poor variants is high, content recommendations, personalization, can tolerate adaptive allocation.

**Use cases**: Content headline optimization, recommendation algorithm selection, promotional banner testing, push notification copy.

**Pros**: Minimizes opportunity cost, adapts to changing conditions, automatic traffic allocation, good for ongoing optimization.

**Cons**: Less interpretable than A/B tests, harder to establish causality, requires more sophisticated infrastructure, may converge to local optimum.

**Defaults**: alpha=0.05, power=0.8, 3 arms.

**Randomization**: REQUEST, RANDOM, not consistent (re-randomized each request).

**Key parameters**:
- **Number of arms**: 2 or more variants
- **Horizon**: Total observations expected (minimum 1,000)
- **Exploration rate (epsilon)**: 0 to 0.5. Higher = more exploration, slower convergence
- **Exploration budget per arm**: (horizon x epsilon) / num_arms
- **Estimated regret**: epsilon x horizon x (arms - 1) / arms x delta (delta = avg reward gap; use 1 for binary rewards)

**Choosing epsilon**:
| Context | Recommended epsilon | Rationale |
|---------|-------------------|-----------| 
| Revenue-critical (checkout, pricing) | 0.01-0.03 | Minimize regret on high-value actions |
| Conversion funnels | 0.05 | Balance learning with conversion loss |
| Content/recommendations | 0.10-0.15 | Content variety has value; lower cost of suboptimal choice |
| Early exploration / cold start | 0.20-0.30 | Need data fast; accept short-term regret |

**Notes**:
- Traditional fixed-sample statistical testing does not apply
- Sample size shown is the exploration budget per arm, not a significance calculation
- Skip variance reduction (adaptive nature handles this)
- Reward function must be well-defined before starting

---

## Group Sequential Test

**When to use**: Standard A/B test scenario, but you want the option to stop early for efficacy, futility, or harm without inflating false positive rates. If your experiment could run for weeks, sequential testing lets you check in periodically with statistical rigor.

**Use cases**: Any A/B test where early stopping is desirable — high-traffic feature launches, pricing experiments, experiments with potential negative impact, time-sensitive decisions.

**Pros**: Can stop early if effect is large (saves time and opportunity cost), formal statistical guarantees at interim analyses, expected sample size is typically 50-80% of maximum, same rigor as fixed-sample tests.

**Cons**: Maximum sample size is 5-20% larger than fixed-sample equivalent, requires pre-registering the analysis schedule and boundaries, more complex to implement and explain, cannot change boundaries mid-experiment.

**Defaults**: alpha=0.05, power=0.8, 2 variants, O'Brien-Fleming spending function, 4 equally spaced looks.

**Randomization**: USER_ID, HASH_BASED, consistent assignment.

**Key parameters**:
- **Number of looks (K)**: 2-5 planned analyses including the final look
- **Information fractions**: When each look occurs (default: equally spaced at 1/K, 2/K, ..., 1)
- **Alpha-spending function**: O'Brien-Fleming (conservative early, liberal late — default), Pocock (equal boundaries), or Lan-DeMets (flexible)
- **Futility boundaries**: Optional. Stop early if effect is unlikely to reach significance. Non-binding (advisory) recommended.

**Maximum sample size inflation**:
| K (looks) | OBF inflation | Pocock inflation |
|-----------|--------------|-----------------|
| 2 | 1.014 (~1%) | 1.101 (~10%) |
| 3 | 1.022 (~2%) | 1.166 (~17%) |
| 4 | 1.027 (~3%) | 1.211 (~21%) |
| 5 | 1.031 (~3%) | 1.244 (~24%) |

**Warnings**:
- Boundaries must be pre-registered — do not change the number of looks or boundaries mid-experiment
- Do not peek at results between planned looks (or use continuous monitoring with always-valid p-values)
- OBF is strongly recommended as default — Pocock allows easier early stopping but costs significantly more maximum sample size

---

## Interleaving Experiment

**When to use**: Comparing two ranking or recommendation systems. Results from both systems are merged into a single interleaved list; user interactions reveal which system is preferred. 10-100x more sensitive than A/B testing for ranking quality.

**Use cases**: Search ranking algorithm comparison, recommendation feed evaluation, content ordering optimization, ad ranking systems.

**Pros**: Dramatically more sensitive than A/B tests (each user is their own control), eliminates between-user variance, requires far fewer users, low risk since users see items from both systems.

**Cons**: Only works for ranking/list-based interfaces, does not directly measure user-facing metrics (engagement, revenue) — those need a follow-up A/B test, credit assignment can be complex, requires running both systems simultaneously.

**Defaults**: alpha=0.05, power=0.8, Team Draft interleaving, win rate as primary metric.

**Randomization**: USER_ID, HASH_BASED, consistent assignment (interleaving randomization is per-session within each user).

**Key parameters**:
- **Interleaving method**: Team Draft (default — simple, fair), Balanced (strict 50/50 item contribution), Probabilistic (item-level credit), Optimized (maximum sensitivity)
- **Primary metric**: Win rate — fraction of sessions where System B is preferred (baseline 0.50, MDE typically 0.51-0.55)
- **Credit function**: How user interactions (clicks, purchases) are attributed to each system

**Sample size** (much smaller than A/B tests):
```
n = (z_alpha + z_beta)^2 / (2 * (mde - 0.50)^2)
```
With alpha=0.05, power=0.8, mde=0.51: n ≈ 39,200 sessions.

**Important**: Interleaving validates ranking quality but not user-facing business metrics. A positive interleaving result should typically be followed by a standard A/B test to measure impact on engagement, revenue, etc.

**Warnings**:
- Only applicable to ranked list / recommendation interfaces
- Requires running both systems in parallel (latency and infrastructure cost)
- Position bias must be handled by the interleaving method (Team Draft handles this naturally)
- Not suitable when systems produce fundamentally different types of results
