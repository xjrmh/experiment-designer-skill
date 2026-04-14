# Experiment Designer Skills

A suite of Claude Code skills for designing statistically rigorous experiments end-to-end. Covers **16 designs** — from standard A/B tests to always-valid sequential monitoring, stepped-wedge rollouts, geo/matched-markets, and post-launch holdouts. Every run produces a complete design document with sample-size math, randomization strategy, variance-reduction plan, risk assessment, and ship/iterate/kill criteria.

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

## Installation

**Project-level** (recommended):
```bash
mkdir -p .claude/skills
cp -r skills/* .claude/skills/
```

**Personal** (available in all projects):
```bash
mkdir -p ~/.claude/skills
cp -r skills/* ~/.claude/skills/
```

## Usage

```
/experiment-designer
/experiment-designer I want to test a new checkout flow
/experiment-designer --quick A/B test, conversion rate, 5% MDE, 10K daily traffic
```

Or go straight to a specific design:

```
/designabtest I want to test a new checkout flow
/designsequential test checkout with an early-stopping option
/designalwaysvalid live conversion dashboard with continuous peeking
/designbayesian pricing test with prior on recent lift
/designequivalence prove new vendor match rate is within 1% of incumbent
/designclustertest compare driver incentives across cities
/designsteppedwedge phased rollout of a new scheduling algorithm
/designswitchback marketplace pricing experiment
/designcrossover latency test where each user sees both algorithms
/designgeo measure TV campaign lift across DMAs
/designincrementality paid-social uplift vs PSA control
/designbandit optimize homepage hero banner
/designinterleaving compare v1 vs v2 search ranking
/designholdout long-term retention impact of the search redesign
```

## Output

Each run produces a Markdown design document containing:

- Hypothesis and success criteria
- Metrics table with baselines
- Sample size calculation and duration estimate
- Randomization and assignment strategy
- Variance reduction plan
- Risk assessment and pre-launch checklist
- Monitoring and stopping rules
- Ship / iterate / kill decision framework
- Post-experiment guidance — effect size, practical significance, follow-ups
- Sign-off checklist (statistician, engineer, PM)
- Optional JSON export for platforms like Statsig, Eppo, and GrowthBook

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
