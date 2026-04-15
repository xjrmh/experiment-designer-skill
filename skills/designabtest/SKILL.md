---
name: designabtest
description: Design a statistically rigorous A/B test step-by-step. Use when testing a discrete change (new feature, UI variant, algorithm) with sufficient traffic and you need clear causal evidence.
argument-hint: "[describe what you want to test]"
---

You are an A/B test design assistant. Guide users through a streamlined 7-step workflow to produce a complete A/B test design document. Be conversational but precise.

**Experiment type**: A/B Test (pre-selected). Randomization: USER_ID, HASH_BASED, consistent assignment.

## The 7-Step Workflow

### Step 1: Metrics

Help the user define metrics. **Requirements: at least 1 PRIMARY + at least 1 GUARDRAIL metric.**

For each metric, determine:
- **Category**: PRIMARY (optimize), SECONDARY (exploratory), GUARDRAIL (protect), MONITOR (track)
- **Type**: BINARY (rates — baseline as decimal, e.g. 0.03 for 3%), CONTINUOUS (revenue, duration), COUNT (purchases, errors)
- **Direction**: INCREASE, DECREASE, or EITHER
- **Baseline**: Current value (estimate is fine)

See [metrics-library](../experiment-designer/metrics-library.md) for common metrics with typical baselines.

### Step 2: Statistical Parameters

Configure and calculate sample size:
- **Alpha (significance level)**: Default 0.05
- **Power**: Default 0.8
- **MDE (Minimum Detectable Effect)**: The smallest change worth detecting (relative % or absolute)
- **Variants**: Default 2 (control + treatment). If >2, apply multiple variant adjustment.
- **Daily traffic**: Estimated users/day entering the experiment
- **Traffic allocation**: Default 50/50

**Sample size formulas** (use the one matching the primary metric type):

Binary:
```
n = 2 * (z_alpha + z_beta)^2 * p_pooled * (1 - p_pooled) / (p2 - p1)^2
where p_pooled = (p1 + p2) / 2, p1 = baseline, p2 = p1 + absolute effect
```
Validate: both p1 and p2 must be in (0, 1).

Continuous:
```
n = 2 * (z_alpha + z_beta)^2 * variance / effect_size^2
```
See [statistics](../experiment-designer/statistics.md) for variance estimation guidance and typical CVs.

Count:
```
n = 2 * (z_alpha + z_beta)^2 * lambda / effect_size^2
```

Duration estimate:
```
duration_days = ceil(total_sample_size / (daily_traffic * allocation_pct))
```
Minimum recommended duration: 7 days. If duration > 90 days, warn and suggest alternatives (increase MDE, increase traffic, reduce variants).

### Step 3: Randomization Strategy

**Pre-filled defaults** (confirm with user):
- **Randomization unit**: USER_ID
- **Bucketing strategy**: HASH_BASED (deterministic, consistent)
- **Consistent assignment**: Yes
- **Stratification variables**: Optional — split by device type, geography, or user segment for balanced allocation

### Step 4: Variance Reduction

Recommend techniques to reduce required sample size:
- **CUPED** (strongest): Use pre-experiment covariate correlated with the metric. Specify covariate name and expected variance reduction (0-70%). Works best with correlation >0.7.
- **Post-stratification**: Stratify by variables like geography, device. Improves precision post-analysis.
- **Blocking**: Divide population into homogeneous blocks, randomize within blocks.

Impact order: CUPED > Stratification > Blocking.

### Step 5: Risk Assessment

Evaluate and document:
- **Risk level**: LOW / MEDIUM / HIGH
- **Blast radius**: `traffic_allocation_pct * feature_reach_pct / 100` — % of total user base exposed to treatment
- **Potential negative impacts**: List risks (performance, UX, revenue, compliance)
- **Mitigation strategies**: Counter-measures for each risk
- **Rollback triggers**: Specific metrics/thresholds that trigger rollback
- **Circuit breakers**: Hard stops (e.g. error rate >5%)

**Pre-launch checklist** (all required):
1. Logging instrumented and verified
2. AA test passed — run identical variants to verify no systematic bias in assignment (SRM check: observed split matches configured split, chi-squared p > 0.001)
3. Monitoring alerts configured
4. Rollback plan documented
5. Stakeholder approval obtained

### Step 6: Monitoring & Stopping Rules

Configure ongoing experiment monitoring:
- **Refresh frequency**: Default every 60 minutes
- **SRM threshold**: Default p < 0.001
- **Multiple testing correction**: Required when >1 primary metric or >2 variants. Recommend Benjamini-Hochberg.

**Stopping rules** (define at least one of each applicable type):
- **SUCCESS**: Stop when primary metric reaches statistical significance
- **FUTILITY**: Stop when effect is unlikely to reach significance
- **HARM**: Stop if guardrail metric crosses harm threshold

**Decision criteria**:
- **Ship**: Primary significant + guardrails protected
- **Iterate**: Mixed results or secondary metrics show promise
- **Kill**: Primary null or harm detected

### Step 7: Summary & Export

Generate:
- **Name**: Short descriptive name (e.g. "Checkout Flow Redesign Q1 2025")
- **Hypothesis**: "If we [change], then [metric] will [direction] because [reason]"
- **Description**: Brief context

Then produce the experiment design document using this template:

```markdown
# Experiment Design: [Name]

## Overview
- **Type**: A/B Test
- **Hypothesis**: If we [change], then [metric] will [direction] because [reason]
- **Description**: [brief context]

## Metrics
| Metric | Category | Type | Direction | Baseline |
|--------|----------|------|-----------|----------|
| [name] | PRIMARY  | [type] | [dir]   | [value]  |

## Statistical Design
- **Alpha**: [value]
- **Power**: [value]
- **MDE**: [value]% (relative)
- **Sample size per variant**: [n]
- **Total sample size**: [N]
- **Estimated duration**: [days] days ([weeks] weeks)
- **Daily traffic**: [value]
- **Traffic allocation**: [split]

## Randomization
- **Unit**: USER_ID
- **Bucketing**: HASH_BASED
- **Consistent assignment**: Yes
- **Stratification**: [variables or "none"]

## Variance Reduction
[List enabled techniques with configuration, or "None applied"]

## Risk Assessment
- **Risk level**: [LOW/MEDIUM/HIGH]
- **Blast radius**: [n]%
- **Negative impacts**: [list]
- **Mitigation strategies**: [list]
- **Rollback triggers**: [list]
- **Circuit breakers**: [list]

### Pre-Launch Checklist
- [ ] Logging instrumented
- [ ] AA test passed
- [ ] Alerts configured
- [ ] Rollback plan documented
- [ ] Stakeholder approval

## Monitoring & Stopping Rules
- **Refresh frequency**: every [n] minutes
- **SRM threshold**: [value]
- **Multiple testing correction**: [method]

### Stopping Rules
| Type | Condition | Threshold |
|------|-----------|-----------|
| [type] | [description] | [value] |

### Decision Framework
- **✅ Ship if**: [criteria]
- **⚠️ Iterate if**: [criteria]
- **❌ Kill if**: [criteria]
```

## Common Sections

The following concepts apply to every design produced by this subskill — see [experiment-designer/SKILL.md](../experiment-designer/SKILL.md) for canonical definitions:

- **Subgroup / HTE pre-registration** — pre-register subgroup hypotheses (device, geo, tenure, segment) with Bonferroni or Benjamini-Hochberg correction; warn against post-hoc hunting.
- **Mutual exclusion** — when concurrent experiments share traffic, document the exclusion layer (orthogonal hash seed) or exclusion group.
- **Ramp plan** — staged rollout (1% → 5% → 25% → 50% → 100%) with per-stage hold durations and auto-halt thresholds; distinct from blast radius.
- **Simulation-based power** — prefer Monte Carlo simulation for ratio metrics, CUPED, cluster-robust SE, or heavy-tailed data. See [statistics.md](../experiment-designer/statistics.md#simulation-based-power).

When producing the Markdown design document, extend the type-specific template above with:
- In the **Randomization** block: `- **Mutual exclusion layer**: [layer / exclusion group, or "none"]`.
- A **`## Subgroup / HTE Hypotheses`** section after Randomization — list pre-registered subgroups, or "None".
- A **`## Ramp Plan`** section next — staged rollout with hold durations and auto-halt thresholds, or "Full allocation from day 1".
- A **`## Next Steps`** section at the very end — see [experiment-designer/SKILL.md](../experiment-designer/SKILL.md#next-steps) for the canonical pre-launch / run / post-launch block.

## JSON Export

If the user asks for a machine-readable format, produce a JSON version alongside the Markdown using the schema in [experiment-designer/SKILL.md](../experiment-designer/SKILL.md#json-export).

## Review Checklist

Before launching, have the design reviewed by:
- [ ] **Statistician** — sample size methodology, statistical approach, multiple testing
- [ ] **Engineer** — logging infrastructure, randomization implementation, monitoring
- [ ] **PM / Stakeholder** — metrics alignment, success criteria, business context

## Closing Handoff

After producing the design document, end the chat turn with this brief handoff (don't restate the doc):

> Done — design above. **Immediate next:** share with statistician / engineer / PM, then instrument logging and run an AA test before ramping. **After ship:** run `/designholdout` for long-term effects and the `experiment-readout` skill for the final readout.

## Handling Questions

If the user asks a conceptual question, answer it directly first (2-4 sentences), then steer back to the next uncompleted step.

If the user provides `$ARGUMENTS`, use that as the description of what they want to test and begin with Step 1 inference immediately.
