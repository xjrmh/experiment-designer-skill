# Experiment Designer Skill

A Claude Code skill that guides you through designing statistically rigorous experiments step-by-step. Covers A/B tests, group sequential, cluster randomized, switchback, causal inference (DiD, RDD, PSM, IV, Synthetic Control), factorial, multi-armed bandit, and interleaving designs.

## What It Does

This repository provides a suite of experiment design skills for Claude Code. Use the general-purpose `/experiment-designer` when you're unsure which type you need, or jump directly to a type-specific skill for a streamlined experience.

### General-Purpose Skill

Invoke `/experiment-designer` in Claude Code and describe what you want to test. The skill walks you through a structured 8-step workflow:

1. **Experiment Type** - Recommends the right design based on your use case (8 types)
2. **Metrics** - Helps define primary, guardrail, and secondary metrics
3. **Statistical Parameters** - Calculates sample size, power, and experiment duration
4. **Randomization** - Configures user assignment strategy
5. **Variance Reduction** - Recommends CUPED, stratification, or other techniques
6. **Risk Assessment** - Documents risks, mitigation, and pre-launch checklist
7. **Monitoring & Stopping Rules** - Sets up SRM detection, stopping criteria, ship/kill decisions
8. **Summary & Export** - Produces a complete experiment design document (Markdown + optional JSON)

**New features:**
- **Quick mode**: Skip confirmations with `/experiment-designer --quick` for experienced users
- **Backtracking**: Say "go back to Step N" to revise any previous decision
- **Post-experiment guidance**: Includes effect size interpretation, practical significance, and follow-up recommendations
- **JSON export**: Machine-readable output for platform integration (Statsig, Eppo, GrowthBook, etc.)

### Type-Specific Skills

For a faster, more focused workflow, use the type-specific skills directly:

| Skill | Type | Best For |
|-------|------|----------|
| `/designabtest` | A/B Test | Feature changes, UI tests, conversion optimization |
| `/designsequential` | Group Sequential | A/B tests with formal early stopping (OBF, Pocock, Lan-DeMets alpha-spending) |
| `/designclustertest` | Cluster Randomized | City/store-level treatments, network effects |
| `/designswitchback` | Switchback | Marketplace pricing, supply-demand experiments |
| `/designcausalinference` | Causal Inference | Post-hoc analysis, natural experiments (DiD, RDD, PSM, IV, Synthetic Control) |
| `/designfactorial` | Factorial | Testing multiple factors simultaneously |
| `/designbandit` | Multi-Armed Bandit | Continuous optimization, content recommendations |
| `/designinterleaving` | Interleaving | Comparing ranking/recommendation systems (10-100x more sensitive than A/B) |

Each type-specific skill skips the type-selection step, pre-fills sensible defaults, and includes only the relevant statistical formulas and parameters.

## Installation

### Option A: Project-level (recommended)

Copy the skill into your project:

```bash
mkdir -p .claude/skills
cp -r skills/* .claude/skills/
```

### Option B: Personal (all projects)

Copy to your personal Claude config:

```bash
mkdir -p ~/.claude/skills
cp -r skills/* ~/.claude/skills/
```

## Usage

In Claude Code, type:

```
/experiment-designer
```

Or with a description:

```
/experiment-designer I want to test a new checkout flow
```

Quick mode for experienced users:

```
/experiment-designer --quick A/B test, conversion rate, 5% MDE, 10K daily traffic
```

Or use a type-specific skill directly:

```
/designabtest I want to test a new checkout flow
/designsequential test checkout with early stopping option
/designclustertest compare driver incentives across cities
/designbandit optimize homepage hero banner
/designinterleaving compare v1 vs v2 search ranking
```

Claude will guide you through the workflow and produce a structured experiment design document at the end.

## Files

```
skills/
├── experiment-designer/           # General-purpose skill (all 8 types)
│   ├── SKILL.md                   # 8-step workflow, inference rules, output template
│   ├── experiment-types.md        # Detailed reference for 8 experiment types
│   ├── statistics.md              # Sample size formulas, ratio metrics, sequential testing,
│   │                              # one-sided tests, MDE guidance, effect size interpretation
│   └── metrics-library.md         # Common metrics with baselines, ratio metrics,
│                                  # industry-specific baselines (e-commerce, SaaS, marketplace, etc.)
├── designabtest/SKILL.md          # A/B test shortcut (7-step workflow)
├── designsequential/SKILL.md      # Group sequential test shortcut (8-step with alpha-spending)
├── designclustertest/SKILL.md     # Cluster randomized shortcut
├── designswitchback/SKILL.md      # Switchback shortcut
├── designcausalinference/SKILL.md # Causal inference shortcut (DiD, RDD, PSM, IV, Synthetic Control)
├── designfactorial/SKILL.md       # Factorial design shortcut
├── designbandit/SKILL.md          # Multi-armed bandit shortcut
└── designinterleaving/SKILL.md    # Interleaving experiment shortcut (ranking/recommendation systems)
```

The type-specific skills reference the shared `statistics.md` and `metrics-library.md` from the `experiment-designer` directory to avoid duplication.

## Example Output

The skill produces a structured markdown document including:

- Experiment overview and hypothesis
- Metrics table with baselines
- Sample size calculation and duration estimate
- Randomization strategy
- Variance reduction plan
- Risk assessment with pre-launch checklist
- Monitoring and stopping rules
- Ship/iterate/kill decision framework
- Post-experiment guidance (effect size interpretation, practical significance)
- Review checklist (statistician, engineer, PM sign-off)
- Optional JSON export for platform integration

## License

MIT

