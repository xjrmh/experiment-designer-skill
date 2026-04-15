# Experiment Designer Skills

A suite of skills for designing statistically rigorous experiments end-to-end. Covers **16 designs** — from standard A/B tests to always-valid sequential monitoring, stepped-wedge rollouts, geo/matched-markets, and post-launch holdouts. Every run produces a complete design document with sample-size math, randomization strategy, variance-reduction plan, risk assessment, and ship/iterate/kill criteria.

## Why use a skill?

Generic prompts drift — asking an LLM to "design an A/B test for me" twice produces two different structures with sample-size math you have to double-check. The skills enforce a fixed 8-step spine, run the correct formula for the design you picked (two-proportion, ratio metrics with the delta method, cluster-adjusted with ICC inflation, sequential with alpha-spending, TOST for equivalence), and emit a consistent reviewable artifact — not prose. Think of it as an experiment-design template with a statistician reading along.

## Two ways to use it

**Start with `/experiment-designer`** when you're not sure which design fits. It runs a short decision helper, recommends a type, and routes you into the matching workflow.

**Jump straight to a type-specific skill** (e.g. `/designabtest`, `/designsequential`) when you already know the design. Each one skips the type-selection step and pre-fills sensible defaults for that design.

## The 8-step workflow

Every skill walks through the same spine:

1. **Experiment type** — confirmed upfront (or skipped in type-specific skills)
2. **Metrics** — primary, guardrail, and secondary
3. **Statistical parameters** — sample size, power, MDE, duration
4. **Randomization** — unit, allocation, and salting strategy
5. **Variance reduction** — CUPED, stratification, re-randomization
6. **Risk assessment** — failure modes, mitigations, pre-launch checklist
7. **Monitoring & stopping rules** — SRM detection, early-stop, ship/kill
8. **Summary & export** — Markdown design doc + optional JSON

**Quick mode** — skip confirmations when you already know the parameters:
`/experiment-designer --quick A/B, conversion, 5% MDE, 10K daily`

**Backtracking** — say *"go back to Step N"* at any time to revise a prior decision; all downstream calculations recompute automatically.

## The 16 designs

| Skill | Design | When to use |
|---|---|---|
| `/designabtest` | A/B Test | Discrete change, ≥1K users/day, need clear causal evidence |
| `/designsequential` | Group Sequential | Pre-planned interim looks with OBF / Pocock / Lan-DeMets alpha-spending |
| `/designalwaysvalid` | Always-Valid (mSPRT) | Continuous peeking without alpha inflation; live dashboards |
| `/designbayesian` | Bayesian A/B | Posterior probabilities, expected-loss stopping, informative priors |
| `/designequivalence` | Equivalence / Non-Inferiority (TOST) | Prove a new variant is not worse within ±δ — vendor swaps, refactors |
| `/designclustertest` | Cluster Randomized | Treatment affects groups (cities, stores); network effects |
| `/designsteppedwedge` | Stepped-Wedge | Phased rollout where all clusters must eventually receive the treatment |
| `/designswitchback` | Switchback | Strong network effects, supply–demand coupling, marketplace pricing |
| `/designcrossover` | Crossover (Within-Subject) | Each user sees both arms with washout; ~50% sample reduction |
| `/designgeo` | Geo / Matched Markets | Brand, TV, OOH — randomize at DMA/city using TBR / synthetic control |
| `/designincrementality` | Incrementality / PSA / Ghost-Ad | True paid-media lift vs observational bias |
| `/designcausalinference` | Causal Inference | Natural experiments, post-hoc: DiD, RDD, PSM, IV, Synthetic Control, ITS |
| `/designfactorial` | Factorial | Multiple factors simultaneously; interaction effects |
| `/designbandit` | Multi-Armed Bandit | Continuous optimization; minimize regret; recommendations |
| `/designinterleaving` | Interleaving | Compare ranking/recommendation systems; 10–100× more sensitive than A/B |
| `/designholdout` | Holdout / Holdback | Post-launch: measure long-term effects (retention, LTV, novelty decay) |

## Requirements

Designed for [Claude Code](https://claude.com/product/claude-code). The SKILL.md files are plain Markdown prompts — they can be adapted to Cursor rules, the Claude Agent SDK, Custom GPTs, or any agent tool that accepts custom instructions by copying the body into that tool's instruction format. Slash-command autoloading and the router in `/experiment-designer` are Claude Code-specific; the workflow content and statistical logic are not.

## Install

**Claude Code plugin marketplace** (recommended — native, versioned, `/plugin update` works):

```
/plugin marketplace add xjrmh/experiment-designer-skill
/plugin install experiment-designer@experiment-designer-skill
```

**Shell one-liner** (works anywhere `~/.claude/skills/` is read — Claude Code, Agent SDK, etc.):

```bash
rm -rf /tmp/eds && git clone https://github.com/xjrmh/experiment-designer-skill /tmp/eds && \
  mkdir -p ~/.claude/skills && cp -r /tmp/eds/skills/* ~/.claude/skills/ && \
  rm -rf /tmp/eds
```

For a project-level install instead, replace `~/.claude/skills/` with `.claude/skills/` inside your project root.

## Demo

Quick start — three ways to invoke the router:

```
/experiment-designer
/experiment-designer I want to test a new checkout flow
/experiment-designer --quick A/B test, conversion rate, 5% MDE, 10K daily traffic
```

Or jump straight to any of the 16 design-specific skills listed in the table above (e.g. `/designsequential test checkout with an early-stopping option`, `/designgeo measure TV campaign lift across DMAs`).

**End-to-end example.** Input:

```
/designabtest "simplify the checkout flow"
```

Output — a Markdown design document (truncated):

~~~md
# Experiment Design: Simplified Checkout Flow

## Overview
- **Type**: A/B Test
- **Hypothesis**: If we collapse checkout to a single page, then completion
  rate will increase because fewer drop-off points remain.

## Metrics
| Metric              | Category  | Type       | Direction | Baseline |
|---------------------|-----------|------------|-----------|----------|
| Checkout completion | PRIMARY   | BINARY     | INCREASE  | 0.10     |
| Revenue per user    | GUARDRAIL | CONTINUOUS | EITHER    | $18.40   |
| Refund rate         | GUARDRAIL | BINARY     | DECREASE  | 0.012    |

## Statistical Design
- **Alpha**: 0.05 (two-sided)   **Power**: 0.80
- **MDE**: 5% relative (0.10 → 0.105)
- **Sample size per variant**: 56,448   **Total**: 112,896
- **Duration**: ~6 days @ 20K daily traffic, 50/50 split

## Randomization
- **Unit**: user_id, hash-based, consistent assignment
- **Stratification**: device_type

## Monitoring & Stopping Rules
- SRM check at p < 0.001, evaluated hourly
- Kill if refund rate rises >20% relative
- Ship if primary wins at p < 0.05 AND no guardrail degrades >1%

## Decision Framework
- **Ship if**: completion lifts ≥5% relative, no guardrail breach
- **Iterate if**: directional lift but not significant
- **Kill if**: guardrail breach or negative completion trend
...
~~~

A full run also includes variance-reduction plan (CUPED, stratification), risk assessment and pre-launch checklist, ramp plan with per-stage auto-halt thresholds, type-specific parameters (priors for Bayesian, margin δ for TOST, cluster sequence for stepped-wedge, etc.), post-experiment guidance on effect size and practical significance, a statistician/engineer/PM sign-off checklist, and optional JSON export for platforms like Statsig, Eppo, and GrowthBook.

## Repository layout

```
skills/
├── experiment-designer/            # General-purpose skill (all 16 designs)
│   ├── SKILL.md                    # 8-step workflow, decision helper, output template
│   ├── experiment-types.md         # Reference for every design type
│   ├── statistics.md               # Sample size, ratio metrics, sequential, mSPRT,
│   │                               # Bayesian, TOST, MDE, effect size interpretation
│   └── metrics-library.md          # Common metrics + industry-specific baselines
│
├── designabtest/SKILL.md           # A/B test
├── designsequential/SKILL.md       # Group sequential (alpha-spending)
├── designalwaysvalid/SKILL.md      # Always-valid (mSPRT)
├── designbayesian/SKILL.md         # Bayesian A/B
├── designequivalence/SKILL.md      # Equivalence / non-inferiority (TOST)
├── designclustertest/SKILL.md      # Cluster randomized
├── designsteppedwedge/SKILL.md     # Stepped-wedge cluster crossover
├── designswitchback/SKILL.md       # Switchback
├── designcrossover/SKILL.md        # Crossover / within-subject
├── designgeo/SKILL.md              # Geo / matched markets
├── designincrementality/SKILL.md   # Incrementality / PSA / ghost-ad
├── designcausalinference/SKILL.md  # Causal inference (DiD, RDD, PSM, IV, SC)
├── designfactorial/SKILL.md        # Factorial
├── designbandit/SKILL.md           # Multi-armed bandit
├── designinterleaving/SKILL.md     # Interleaving
└── designholdout/SKILL.md          # Holdout / holdback
```

Type-specific skills reference the shared `statistics.md` and `metrics-library.md` in `experiment-designer/` to avoid duplication.

## License

MIT
