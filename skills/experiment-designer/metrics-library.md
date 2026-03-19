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

Baselines as raw values. Provide variance (sigma^2) or standard deviation for sample size calculation. If unknown, estimate CV from the typical ranges in [statistics.md](statistics.md) (e.g. revenue 80-150%, latency 30-50%, engagement 25-40%). Do NOT default to CV=10% — this dramatically underestimates variance and sample size for most metrics.

| Metric | Typical Category | Direction | Baseline | Variance | Notes |
|--------|-----------------|-----------|----------|----------|-------|
| Revenue per User | PRIMARY | INCREASE | $50 | 2,500 (sd=50) | Average revenue generated per user |
| Average Order Value | PRIMARY | INCREASE | $75 | 1,875 (sd=43) | Average transaction value |
| Session Duration | SECONDARY | INCREASE | 180s | 23,104 (sd=152) | Average session length in seconds |
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
| Compare ranking systems | Win Rate (interleaving) | Latency, Coverage, Error Count |

## Ratio Metrics

Some key metrics are ratios (numerator/denominator). These require special variance estimation — see [statistics.md](statistics.md) for delta method and linearization formulas.

| Metric | Numerator | Denominator | Typical Category | Direction | Typical Baseline |
|--------|-----------|-------------|-----------------|-----------|-----------------|
| Revenue per Session | Total revenue | Sessions | PRIMARY | INCREASE | $2-$15 |
| Clicks per Impression | Clicks | Impressions | PRIMARY | INCREASE | 0.01-0.05 |
| Add-to-Cart per Visit | Add-to-cart events | Visits | PRIMARY | INCREASE | 0.05-0.15 |
| Orders per Visit | Orders | Visits | PRIMARY | INCREASE | 0.01-0.05 |

## Industry-Specific Baselines

Metric baselines vary significantly by industry. Use these ranges as starting points and adjust to your actual data.

### E-Commerce
| Metric | Typical Baseline | Notes |
|--------|-----------------|-------|
| Conversion Rate | 0.02-0.04 (2-4%) | Desktop higher than mobile |
| Average Order Value | $50-$150 | Varies by product category |
| Cart Abandonment | 0.65-0.75 (65-75%) | Industry standard is high |
| Revenue per User | $1-$10 | Heavily skewed distribution |

### SaaS / B2B
| Metric | Typical Baseline | Notes |
|--------|-----------------|-------|
| Free Trial Conversion | 0.05-0.15 (5-15%) | Depends on trial length |
| Activation Rate | 0.20-0.40 (20-40%) | Users completing key action |
| Feature Adoption | 0.10-0.30 (10-30%) | Users engaging with a feature |
| Churn Rate (monthly) | 0.03-0.08 (3-8%) | Lower is better |

### Marketplace / Two-Sided
| Metric | Typical Baseline | Notes |
|--------|-----------------|-------|
| Booking/Order Rate | 0.05-0.15 (5-15%) | Search to booking |
| Supply Utilization | 0.40-0.70 (40-70%) | % of supply being used |
| Match Rate | 0.30-0.60 (30-60%) | Successful matches |
| Take Rate | 0.10-0.25 (10-25%) | Platform commission |

### Content / Media
| Metric | Typical Baseline | Notes |
|--------|-----------------|-------|
| CTR | 0.01-0.05 (1-5%) | Varies by placement |
| Time on Content | 30-120s | Article/video engagement |
| Scroll Depth | 0.40-0.60 (40-60%) | % of page scrolled |
| Share Rate | 0.001-0.01 (0.1-1%) | Very low for most content |

### Fintech
| Metric | Typical Baseline | Notes |
|--------|-----------------|-------|
| Application Completion | 0.30-0.60 (30-60%) | Form completion rate |
| Approval Rate | 0.40-0.70 (40-70%) | Depends on risk tolerance |
| Funding Rate | 0.60-0.85 (60-85%) | Approved users completing funding |
| DAU/MAU | 0.10-0.30 (10-30%) | User stickiness |

