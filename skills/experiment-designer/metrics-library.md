# Common Metrics Library

Use these as starting points when helping users select metrics. Adjust baselines to match the user's actual data when available.

## Binary Metrics

Baselines expressed as decimals (0.05 = 5%). Use proportions test for sample size calculation.

| Metric | Typical Category | Direction | Baseline | Notes |
|--------|-----------------|-----------|----------|-------|
| Conversion Rate | PRIMARY | INCREASE | 0.05 (5%) | Users completing a desired action |
| Click-Through Rate (CTR) | PRIMARY | INCREASE | 0.03 (3%) | Users clicking a specific element |
| Bounce Rate | GUARDRAIL | DECREASE | 0.40 (40%) | Users leaving without interaction |
| Signup Rate | PRIMARY | INCREASE | 0.02 (2%) | Visitors creating an account |
| Retention Rate (7-day) | PRIMARY | INCREASE | 0.25 (25%) | Users returning within 7 days |

## Continuous Metrics

Baselines as raw values. Provide variance (sigma^2) or standard deviation for sample size calculation. If unknown, default to CV=10% (variance = (baseline * 0.1)^2).

| Metric | Typical Category | Direction | Baseline | Variance | Notes |
|--------|-----------------|-----------|----------|----------|-------|
| Revenue per User | PRIMARY | INCREASE | $50 | 2,500 (sd=50) | Average revenue generated per user |
| Average Order Value | PRIMARY | INCREASE | $75 | 1,875 (sd=43) | Average transaction value |
| Session Duration | SECONDARY | INCREASE | 180s | 14,400 (sd=120) | Average session length in seconds |
| Page Load Time | GUARDRAIL | DECREASE | 1,500ms | 250,000 (sd=500) | Average page load in milliseconds |
| Engagement Score | SECONDARY | INCREASE | 7.5 | 6.25 (sd=2.5) | Composite engagement metric |

## Count Metrics

Baselines as average counts. Use Poisson approximation (variance = lambda = baseline).

| Metric | Typical Category | Direction | Baseline | Notes |
|--------|-----------------|-----------|----------|-------|
| Number of Purchases | PRIMARY | INCREASE | 1.5 | Purchases per user |
| Pages per Session | SECONDARY | INCREASE | 4.0 | Pages viewed per session |
| Error Count | GUARDRAIL | DECREASE | 0.1 | Errors encountered per user |
| Feature Usage Count | SECONDARY | INCREASE | 2.5 | Times a feature is used per user |

## Input Validation Rules

Before calculating sample size, validate metric inputs:

| Metric type | Baseline constraint | Additional checks |
|-------------|-------------------|-------------------|
| BINARY | Must be in (0, 1) exclusive — reject 0, 1, negatives, or values >1 | p2 = baseline + absolute_effect must also be in (0, 1) |
| CONTINUOUS | Must be positive (or explicitly allow negative for delta metrics) | Variance/SD must be positive. If user provides CV, verify CV > 0 |
| COUNT | Must be non-negative (>= 0) | Lambda = 0 is degenerate — warn if baseline count is 0 |

If a user provides a percentage (e.g. "5%") for a binary metric, convert to decimal (0.05) before calculation.

## Choosing the Right Metric Type

| If the metric is... | Use type | Baseline format |
|---------------------|----------|-----------------|
| A rate or percentage (conversion, CTR, bounce) | BINARY | Decimal (0.05 for 5%) |
| A dollar amount, duration, or score | CONTINUOUS | Raw value ($50, 180s) |
| A count of events per user | COUNT | Average count (1.5) |

## Metric Category Guidelines

- **PRIMARY**: The metric you're trying to move. Every experiment needs at least one.
- **GUARDRAIL**: Metrics that must not degrade. Every experiment needs at least one. Common guardrails: error rate, page load time, bounce rate, revenue (if not primary).
- **SECONDARY**: Interesting to observe but not the primary goal. Helps explain results.
- **MONITOR**: Operational metrics to track (e.g. traffic volume, latency). Not used in statistical analysis.

## Quick Metric Selection by Experiment Goal

| Goal | Suggested PRIMARY | Suggested GUARDRAIL |
|------|-------------------|---------------------|
| Improve conversion funnel | Conversion Rate | Bounce Rate, Page Load Time |
| Increase revenue | Revenue per User | Conversion Rate, Error Count |
| Boost engagement | Session Duration or Engagement Score | Bounce Rate, Error Count |
| Optimize content | CTR | Bounce Rate, Revenue per User |
| Reduce friction | Signup Rate or Conversion Rate | Page Load Time, Error Count |
| Improve retention | Retention Rate (7-day) | Revenue per User, Error Count |
