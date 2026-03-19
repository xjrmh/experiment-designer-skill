# Experiment Designer Skill

A Claude Code skill that guides you through designing statistically rigorous experiments step-by-step. Covers A/B tests, cluster randomized, switchback, causal inference, factorial, and multi-armed bandit designs.

## What It Does

This repository provides a suite of experiment design skills for Claude Code. Use the general-purpose `/experiment-designer` when you're unsure which type you need, or jump directly to a type-specific skill for a streamlined experience.

### General-Purpose Skill

Invoke `/experiment-designer` in Claude Code and describe what you want to test. The skill walks you through a structured 8-step workflow:

1. **Experiment Type** - Recommends the right design based on your use case
2. **Metrics** - Helps define primary, guardrail, and secondary metrics
3. **Statistical Parameters** - Calculates sample size, power, and experiment duration
4. **Randomization** - Configures user assignment strategy
5. **Variance Reduction** - Recommends CUPED, stratification, or other techniques
6. **Risk Assessment** - Documents risks, mitigation, and pre-launch checklist
7. **Monitoring & Stopping Rules** - Sets up SRM detection, stopping criteria, ship/kill decisions
8. **Summary & Export** - Produces a complete experiment design document

### Type-Specific Skills

For a faster, more focused workflow, use the type-specific skills directly:

| Skill | Type | Best For |
|-------|------|----------|
| `/designabtest` | A/B Test | Feature changes, UI tests, conversion optimization |
| `/designclustertest` | Cluster Randomized | City/store-level treatments, network effects |
| `/designswitchback` | Switchback | Marketplace pricing, supply-demand experiments |
| `/designcausalinference` | Causal Inference | Post-hoc analysis, natural experiments (DiD, RDD, PSM, IV) |
| `/designfactorial` | Factorial | Testing multiple factors simultaneously |
| `/designbandit` | Multi-Armed Bandit | Continuous optimization, content recommendations |

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

Or use a type-specific skill directly:

```
/designabtest I want to test a new checkout flow
/designclustertest compare driver incentives across cities
/designbandit optimize homepage hero banner
```

Claude will guide you through the workflow and produce a structured experiment design document at the end.

## Files

```
skills/
├── experiment-designer/           # General-purpose skill (all 6 types)
│   ├── SKILL.md                   # 8-step workflow, inference rules, output template
│   ├── experiment-types.md        # Detailed reference for 6 experiment types
│   ├── statistics.md              # Sample size formulas, adjustments, duration estimation
│   └── metrics-library.md         # 14 common metrics with baselines and variance values
├── designabtest/SKILL.md          # A/B test shortcut (7-step workflow)
├── designclustertest/SKILL.md     # Cluster randomized shortcut
├── designswitchback/SKILL.md      # Switchback shortcut
├── designcausalinference/SKILL.md # Causal inference shortcut (DiD, RDD, PSM, IV)
├── designfactorial/SKILL.md       # Factorial design shortcut
└── designbandit/SKILL.md          # Multi-armed bandit shortcut
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

## License

MIT
