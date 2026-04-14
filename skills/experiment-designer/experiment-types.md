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
| **Interrupted Time Series (ITS)** | No concurrent shock at intervention time; pre-trend extrapolates to counterfactual | Single series with a sharp intervention (policy change, launch), long pre-period (20+ points), no control group available |

**Method-specific notes**:
- DiD: Serial correlation inflates variance. AR(1) VIF = (1 + rho) / (1 - rho)
- RDD: Sample size depends on bandwidth near threshold. Consult a statistician for bandwidth selection
- PSM: Effective sample size depends on match quality and covariate overlap
- IV: Weak instruments lead to biased estimates. Check first-stage F-statistic > 10
- Synthetic Control: Requires a donor pool of untreated units and a long pre-treatment period (typically 20+ time periods) for a good fit. Effect is estimated as the gap between the treated unit and its synthetic counterfactual post-treatment. Inference via placebo tests (apply the method to each donor unit as if it were treated — the treated unit's effect should be an outlier).
- ITS: Segmented regression `y_t = β0 + β1*t + β2*D_t + β3*(t - t*)*D_t + ε`, where D_t is post-intervention indicator, t* is intervention time. β2 = immediate level change, β3 = change in slope. Strongest threat is a concurrent shock; a DiD with a control series is strictly better when available. Model autocorrelation with Newey-West or Prais-Winsten SE. Minimum 20 pre-period and 10 post-period observations; more is better.

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

---

## Holdout / Holdback

**When to use**: Measure the long-term effect of a feature that has already launched (or is about to launch at 100%). A small % of users is held back on control permanently (or for a fixed long window) so that retention, LTV, and novelty-decay effects can be measured over 30-90+ days.

**Use cases**: Post-launch validation of a major feature (new feed algorithm, pricing change), long-term retention / LTV measurement, detecting novelty effects that fade after 2-4 weeks, annual feature-value quantification for planning.

**Pros**: Only way to measure true long-term causal effect post-launch, decouples novelty from sustained lift, naturally accommodates seasonal effects if held for a full cycle, minimal incremental infra once launched.

**Cons**: Holdout users miss out on improvements (opportunity cost), requires durable suppression logic, ongoing cost of running both code paths, political friction ("we already know it works"), sample size may be small if holdback % is tiny.

**Defaults**: alpha=0.05, power=0.8, 5% holdback, 90-day duration.

**Randomization**: USER_ID, HASH_BASED, consistent assignment (the same users stay in holdback for the full duration).

**Key parameters**:
- **Holdback percentage**: Typically 1-10%. Smaller % = less opportunity cost but larger MDE floor. Use the MDE feasibility formula to right-size.
- **Duration**: 30 days minimum for short-term metrics, 60-90 days for retention, 90-180+ days for LTV.
- **Primary metric**: Long-term metrics (30/60/90-day retention, LTV, cumulative engagement) — NOT the short-term metric the feature was designed to move.

**Warnings**:
- Hold the same users for the full duration — do NOT rotate.
- Suppression must survive logout, device change, reinstall (use stable USER_ID hash).
- Run as a single arm with one permanent treatment (launched feature) vs control (original). Multi-arm holdouts are rarely worth the complexity.
- Document the holdout in a canonical registry so downstream experiments can account for it.

---

## Stepped-Wedge Design

**When to use**: A causal experiment is desired, but the treatment must be rolled out to all clusters eventually (e.g. infrastructure migration, regulatory requirement, staffing). Clusters cross over from control to treatment in a staggered sequence, so every cluster contributes both control and treatment observations.

**Use cases**: Infrastructure rollouts (new logging stack cluster by cluster), healthcare interventions (hospital-by-hospital policy change), regional feature rollouts that will reach everyone, phased product migrations where A/B is politically impossible.

**Pros**: Accommodates "everyone gets it eventually" constraint while preserving causal identification, every cluster serves as its own pre-period control (removes cluster fixed effects), well-established in health economics and implementation science.

**Cons**: Longer duration than standard cluster RCT (must span all crossover periods), confounded with secular time trends — requires time fixed effects and mixed-effects modeling, more complex analysis, assumes the treatment effect is constant across crossover times (strong assumption).

**Defaults**: alpha=0.05, power=0.8, 4-6 time periods, 8+ clusters crossed over in randomly-assigned sequence.

**Randomization**: CLUSTER + period schedule (each cluster is randomly assigned a crossover time). The random element is WHEN each cluster crosses over, not WHETHER.

**Key parameters**:
- **Number of clusters (I)**: Minimum 8. More is substantially better; stepped-wedge is often less efficient than parallel cluster RCT when I is small.
- **Number of periods (T)**: Minimum 3 (baseline + at least 2 crossover steps). 4-6 is typical.
- **Sequences**: In the simplest form, one cluster crosses over per period (I = T-1). Group-based sequences allow multiple clusters per step.
- **ICC** and **cluster-period correlation (CAC)**: Both affect design effect; see Hussey & Hughes (2007).
- **Analytical model**: Mixed-effects model with cluster random intercept, period fixed effects, treatment indicator. Pre-register this before launch.

**Warnings**:
- Strongly confounded with time trends — fit time fixed effects or splines; a sudden secular change looks identical to the treatment effect.
- Assumes constant treatment effect across crossover times; if novelty or fatigue is expected, interact treatment with time-since-crossover.
- Loss of any cluster mid-way severely biases estimates — commit only clusters that can definitely complete the full sequence.

---

## Geo Experiment / Matched Markets

**When to use**: User-level randomization is infeasible (brand / awareness / above-the-line marketing, supply-side experiments in marketplaces, platforms where users aren't uniquely identifiable across channels), but geographic units (DMAs, cities, states, countries) can be randomized or matched. Standard in marketing measurement.

**Use cases**: TV / OOH / brand campaign lift, marketplace supply-side incentives by city, policy rollouts to specific DMAs, offline-online attribution tests, creative / channel-level lift tests.

**Pros**: Works when user-level randomization is impossible, supports brand and offline-heavy channels that upper-funnel tests cannot, widely accepted for marketing measurement, MMM-friendly.

**Cons**: Very few units (10-200 geos typically) → much larger MDE than user-level (often 10-30% required), spillover between nearby geos can bias estimates, long duration (2-8 weeks), seasonal / local shocks contaminate small geo sets, heavy reliance on pre-period matching quality.

**Defaults**: alpha=0.10 (more lenient due to low power), power=0.8, 2-8 week duration with 4-12 week pre-period for matching.

**Randomization**: GEO unit (DMA / city / country). Either (1) pure random assignment across matched pairs, or (2) optimization-based matching (TBR, GeoLift, SparseSC) that chooses a test set and synthesizes a control from the remaining donor pool.

**Key parameters**:
- **Matching method**: Time-Based Regression (TBR, Google), GeoLift (Meta), Synthetic Control (SparseSC), or simple matched pairs on pre-period metric correlation.
- **Pre-period length**: 4-12 weeks minimum; longer is better for matching quality.
- **Test / control geo split**: Often asymmetric — treat a small number of geos (5-20) and synthesize control from the rest.
- **Spillover radius**: Geos adjacent to treatment geos may be contaminated (commuters, media overlap). Exclude or bucket separately.
- **Primary metric**: Geo-level aggregate (total revenue, total conversions, total users) — NOT per-user metrics.

**Warnings**:
- Pre-period matching quality (RMSPE, pre-period correlation) must be checked before launch — poor matches mean the "synthetic control" is noise.
- Placebo / in-time placebo tests (pretend treatment started earlier) should be run to confirm the method produces null results when nothing happened.
- Spillover between geos is the dominant threat — exclude bordering geos or use a buffer zone.
- Seasonality can dominate a short test — include seasonal controls in the model or run long enough to span the cycle.
- Power is often much lower than users assume; use simulation-based power on historical data before committing.

---

## Incrementality / PSA / Ghost-Ad Test

**When to use**: Measure the true causal lift of an ad campaign (or of being exposed to an ad) vs observational bias from targeting / selection. The treatment group is served the real ad; the control group is ad-eligible but served a PSA (public-service ad) or a "ghost" impression that was logged but never delivered.

**Use cases**: Paid media lift measurement, channel attribution, retargeting vs prospecting lift, creative A/B on conversion lift (not just CTR), TV incrementality with digital match.

**Pros**: The only clean way to measure ad-exposure lift — observational "exposed vs not exposed" comparisons are severely confounded by targeting. Industry standard (Meta Conversion Lift, Google Brand Lift, Ghost Ads, LiftIQ). Naturally accounts for intent-to-treat.

**Cons**: Requires ad-serving control (DSP / platform cooperation), PSA is true cost / opportunity cost paid to run the experiment, typically small effect sizes (1-5% relative lift) that need large samples, exposure itself is stochastic which adds variance, intent-to-treat vs actually-exposed analyses give different answers.

**Defaults**: alpha=0.05 (or 0.10 for brand metrics), power=0.8, 50/50 split on ad-eligibility, 2-4 week duration.

**Randomization**: USER_ID, HASH_BASED, consistent assignment, randomized at ad-auction eligibility (NOT at impression — randomizing at impression biases toward exposed users).

**Key parameters**:
- **Exposure definition**: "Won an impression in the auction" (ghost-ad) vs "Saw a PSA in place of the real ad" (PSA). Ghost-ad is cleaner but requires platform support.
- **Exposure rate**: Fraction of ad-eligible users who actually see the ad. Lower = larger sample needed to detect lift.
- **Primary metric**: Conversion / revenue / brand-survey metric, as observed across ALL assigned users (intent-to-treat), not just exposed. Separately report treatment-on-treated (2SLS with exposure as instrument).
- **Control type**: PSA (public-service ad), ghost ad (logged impression, not served), or no-ad hold-out.

**Warnings**:
- Randomize at eligibility, NOT at impression. Randomizing at impression selects on the exposure event and invalidates causal inference.
- Report intent-to-treat as primary; treatment-on-treated is secondary and requires a first-stage exposure analysis.
- Cross-device exposure often unmeasured — account for this in scope claims.
- Ad platforms may not support true PSA/ghost-ad — verify vendor-side infrastructure before committing.
- For small conversion rates, sample sizes can be enormous (millions of users).

---

## Crossover / Within-Subject Design

**When to use**: Each user can see both arms at different times (with a washout period), and carryover effects are either absent or removable. Every user becomes their own control, removing between-user variance — often ~50% fewer users than a parallel A/B.

**Use cases**: Personal-productivity features where cross-arm exposure doesn't confuse the user (keyboard shortcuts, notification frequency), dosage-like studies (different email frequencies across weeks), sensory / UI preference tests, any setting where novelty is short and carryover is minimal.

**Pros**: Dramatically reduces required sample size (no between-user variance), paired analysis is statistically efficient, well-suited to small populations (B2B, internal tools) where user-level A/B is underpowered.

**Cons**: Requires a washout period (lost time), assumes no carryover between periods (strong assumption — test it), order effects (seeing A then B vs B then A can differ), not suitable when the treatment is "sticky" or when exposure to one arm permanently changes behavior.

**Defaults**: alpha=0.05, power=0.8, 2 periods (AB / BA), equal length with a washout between.

**Randomization**: USER_ID, HASH_BASED, consistent assignment, plus randomized ORDER (half AB, half BA).

**Key parameters**:
- **Period length**: Long enough for the outcome to stabilize, short enough to run the full design within a reasonable window.
- **Washout duration**: Period between arms during which no treatment is applied; must be long enough for carryover to decay. Often 1-7 days.
- **Order effect test**: Run a pre-period comparison or post-hoc order × period interaction test; if significant, a crossover analysis is biased — fall back to first-period-only (parallel) analysis.
- **Primary model**: Paired t-test on within-user difference, or a mixed model with period + treatment fixed effects and user random intercept.

**Warnings**:
- Carryover is the dominant threat. Pre-specify a carryover / order-effect test and an escape plan (use first-period data only) if it fails.
- Not suitable when treatment permanently changes user behavior (e.g. learning a new UI).
- Dropout between periods severely biases the analysis — confirm follow-through incentives.
- Seasonality across the two periods can contaminate results (e.g. Week 1 weekend, Week 2 weekdays). Match period composition.

---

## Bayesian A/B Test

**When to use**: Prefer posterior probabilities and expected-loss decision making over p-values. Common when stakeholders think in "probability X beats Y" terms, when continuous monitoring is desired (under proper priors, no alpha inflation), when prior information from past experiments is valuable, or when results must be combined in a decision-theoretic framework.

**Use cases**: Continuous product optimization with stakeholder communication, experiments with strong prior beliefs (incremental feature tweaks), multi-arm comparisons with loss-based decision making, experiments whose outcome feeds a business decision with known cost-of-error.

**Pros**: Direct probabilistic interpretation ("85% chance B beats A"), no pre-registered stopping rules needed if priors and loss function are pre-specified, naturally accommodates informative priors from past experiments, intuitive decision via expected loss rather than p-value thresholds.

**Cons**: Prior choice is consequential and must be defensible (weak priors mimic frequentist; informative priors introduce analyst degrees of freedom), stakeholders may not be familiar, computation can be heavier (MCMC for non-conjugate models), misuse is easy — "continuous monitoring is free" only holds under correct Bayesian updating with proper priors.

**Defaults**: weak priors (Beta(1,1) for rates, Normal(0, large σ²) for means), decision threshold expected loss < 0.5% of baseline OR P(beat) > 0.95.

**Randomization**: USER_ID, HASH_BASED, consistent assignment.

**Key parameters**:
- **Prior**: Beta(α, β) for rates, Normal(μ, σ²) for means, or empirical Bayes / hierarchical when drawing from a history of similar experiments.
- **Decision rule**: Expected loss threshold (preferred) OR P(treatment beats control) threshold.
- **Sample size**: Sized to achieve a target posterior width, or target probability that expected loss decision is correct. Simulation-based is recommended.
- **Reporting**: Posterior mean and 95% credible interval, P(beat control), P(best among arms), expected loss, plus frequentist p-value for skeptical stakeholders.

**Warnings**:
- Informative priors introduce analyst degrees of freedom — pre-register priors before seeing data.
- "Continuous monitoring is free" requires proper priors AND a proper loss-based stopping rule; stopping the moment P(beat) > 0.95 without a loss function is not the same thing and can still inflate error rates.
- Report sensitivity of conclusions to prior choice (run under weak, moderate, and strong priors).
- For rare outcomes, weakly informative priors (e.g. Beta(2, 20) instead of Beta(1, 1)) stabilize estimates.

---

## Equivalence / Non-Inferiority Test (TOST)

**When to use**: The goal is NOT to prove the treatment is better but to prove it is (a) equivalent within ±δ (equivalence) or (b) not worse by more than δ (non-inferiority). The null hypothesis is reversed: H0 is "meaningfully different / worse"; H1 is "within margin". Two one-sided tests (TOST) at α each.

**Use cases**: Switching payment vendors with no expected improvement (prove parity), refactoring a production system (prove no regression), cost-reduction changes (prove performance maintained), regulatory non-inferiority (medical devices, drug reformulation).

**Pros**: Directly answers "is the new version not worse" — a question that superiority tests cannot answer (non-significant in a superiority test ≠ proof of equivalence). Sample-size choice forces an explicit statement of "how much worse is acceptable" (δ), which sharpens decision criteria.

**Cons**: Requires a defensible equivalence margin δ, which stakeholders rarely want to commit to upfront. Sample size can be 2-4x that of an equivalent superiority test (especially when true difference is near zero). Two-sided equivalence requires both bounds; one-sided non-inferiority is easier but weaker evidence.

**Defaults**: alpha=0.05 per one-sided test (so 90% CI must be fully inside ±δ for two-sided equivalence), power=0.8, δ set by stakeholder negotiation.

**Randomization**: USER_ID, HASH_BASED, consistent assignment.

**Key parameters**:
- **Equivalence margin (δ)**: The maximum difference considered practically unimportant. Must be pre-registered and defended (business, regulatory, or domain justification).
- **Direction**: Equivalence (two-sided: |Δ| < δ) vs non-inferiority (one-sided: Δ > −δ, treatment must not be worse by more than δ).
- **Expected true difference (Δ)**: Often assumed 0 for equivalence; non-inferiority may assume a small negative.
- **Analysis**: Two one-sided tests, OR check whether a (1 − 2α) two-sided CI is fully inside ±δ — equivalent procedures.

**Warnings**:
- Equivalence margin δ must be pre-registered — choosing δ after seeing data invalidates the test.
- Sample size can balloon when (δ − |Δ|) is small; run the feasibility check early.
- A non-significant superiority test is NOT proof of equivalence — this is the classic reason to use TOST.
- Report the confidence interval, not just "significant equivalence", so reviewers can judge margin adequacy.

---

## Always-Valid / mSPRT (Confidence Sequences)

**When to use**: Peeking is desired throughout the experiment — every day, every hour, or continuously — without any pre-registered look schedule, and without inflating false-positive rates. Contrast with group sequential, which requires pre-registered looks and alpha-spending.

**Use cases**: Dashboards that show "live" experiment results, high-cadence decision making where waiting for a fixed-sample test is costly, experiments where the number of peeks is unknown in advance, academic / observational settings that need anytime-valid inference.

**Pros**: Truly anytime-valid — the false-positive guarantee holds regardless of when and how often you look, no pre-registered look schedule required (unlike group sequential), naturally communicated as "the 95% confidence sequence currently covers the true effect".

**Cons**: Larger maximum sample size vs fixed-sample (~10-30% inflation), less familiar to most practitioners than p-values, requires choosing a mixture prior (which is a design-time decision analogous to the Bayesian prior), decision rule still needs stakeholder buy-in.

**Defaults**: alpha=0.05, power=0.8, mSPRT with Normal(0, σ_tune²) mixture prior, σ_tune chosen based on expected effect size.

**Randomization**: USER_ID, HASH_BASED, consistent assignment.

**Key parameters**:
- **Mixture prior**: Normal(0, σ_tune²) is standard; σ_tune should be roughly the expected effect size. Wider priors = more robust to small effects but slower convergence; tighter priors = more sensitive to effects near σ_tune.
- **Stopping rule**: Stop the first time the confidence sequence excludes 0 (or excludes the practical-equivalence region).
- **Max sample size**: Set a hard cap to bound the experiment — mSPRT can run forever on a null.

**Warnings**:
- mSPRT is not a Bayesian test — it has frequentist anytime-valid guarantees but uses a mixture prior as a computational device. Don't confuse the two.
- Always set a maximum sample size — without one, you'll run indefinitely on a null effect.
- The mixture prior choice is a design-time decision and must be pre-registered.
- Reference: Johari, Koomen, Pekelis, Walsh (2022), "Always Valid Inference: Continuous Monitoring of A/B Tests".
