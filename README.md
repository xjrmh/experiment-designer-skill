# Experiment Designer Skill

A Claude Code skill that guides you through designing statistically rigorous experiments step-by-step. Covers A/B tests, cluster randomized, switchback, causal inference, factorial, and multi-armed bandit designs.

## What It Does

Invoke `/experiment-designer` in Claude Code and describe what you want to test. The skill walks you through a structured 8-step workflow:

1. **Experiment Type** - Recommends the right design based on your use case
2. **Metrics** - Helps define primary, guardrail, and secondary metrics
3. **Statistical Parameters** - Calculates sample size, power, and experiment duration
4. **Randomization** - Configures user assignment strategy
5. **Variance Reduction** - Recommends CUPED, stratification, or other techniques
6. **Risk Assessment** - Documents risks, mitigation, and pre-launch checklist
7. **Monitoring & Stopping Rules** - Sets up SRM detection, stopping criteria, ship/kill decisions
8. **Summary & Export** - Produces a complete experiment design document

## Supported Experiment Types

| Type | Best For |
|------|----------|
| **A/B Test** | Feature changes, UI tests, conversion optimization |
| **Cluster Randomized** | City/store-level treatments, network effects |
| **Switchback** | Marketplace pricing, supply-demand experiments |
| **Causal Inference** | Post-hoc analysis, natural experiments (DiD, RDD, PSM, IV) |
| **Factorial** | Testing multiple factors simultaneously |
| **Multi-Armed Bandit** | Continuous optimization, content recommendations |

## Installation

### Option A: Project-level (recommended)

Copy the skill into your project:

```bash
mkdir -p .claude/skills
cp -r skills/experiment-designer .claude/skills/
```

### Option B: Personal (all projects)

Copy to your personal Claude config:

```bash
mkdir -p ~/.claude/skills
cp -r skills/experiment-designer ~/.claude/skills/
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

Claude will guide you through all 8 steps and produce a structured experiment design document at the end.

## Files

```
skills/experiment-designer/
├── SKILL.md              # Main skill (8-step workflow, inference rules, output template)
├── experiment-types.md   # Detailed reference for 6 experiment types
├── statistics.md         # Sample size formulas, adjustments, duration estimation
└── metrics-library.md    # 14 common metrics with baselines and variance values
```

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
