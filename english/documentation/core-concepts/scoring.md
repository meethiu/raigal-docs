---
description: "How Raigal computes the 0-100 quality score, weighted penalties, and logarithmic compression."
icon: gauge
---

# Scoring engine

Raigal evaluates codebases on a scale from 0 to 100, where 100 represents zero detected issues. Rather than subtracting arbitrary flat points or dividing by arbitrary lines-of-code metrics, the scoring engine normalizes findings by source file count, applies per-rule saturation caps, and compresses total deductions logarithmically.

## Deduction calculation

Every finding generates a raw deduction based on severity, engine category, and rule impact tier:

$$\text{deduction} = \text{severityPenalty} \times \text{engineWeight} \times \text{ruleImpactMultiplier}$$

### Severity penalties

| Severity | Penalty | Description |
|---|---|---|
| `error` | 3.0 | Blocking bugs, security flaws, type failures, or critical pattern violations |
| `warning` | 1.0 | Unsafe casts, code complexity, or maintenance warnings |
| `info` / `advisory` | 0.25 | Style suggestions and informational notices |

### Engine weights

Each diagnostic engine applies a category multiplier to reflect risk:

| Engine | Weight | Scope |
|---|---|---|
| `security` | 1.5 | Hardcoded secrets, injection vulnerabilities, and dangerous APIs |
| `ai-slop` | 1.0 | Redundant null checks, hallucinated polyfills, trivial comments, and AI anti-patterns |
| `architecture` | 1.0 | Layer boundary rules and cross-module circular dependencies |
| `code-quality` | 0.8 | Unused exports (Knip), dead code, and function complexity limits |
| `lint` | 0.6 | Static analysis rules powered by Oxlint, Ruff, and compiler diagnostics |
| `format` | 0.3 | Formatting inconsistencies, whitespace, and import sorting |

### Rule impact tiers and default caps

Rules belong to impact tiers that define their multiplier and baseline deduction cap:

| Tier | Multiplier | Default cap | Typical finding types |
|---|---|---|---|
| `strict` | $1.0\times$ | Uncapped | Type check errors, syntax failures, and leaked credentials |
| `standard` | $1.0\times$ | Uncapped | Standard linter diagnostics and reliable defect checks |
| `maintainability` | $0.75\times$ | 24 | High cyclomatic complexity and large functions |
| `mechanical` | $0.5\times$ | 16 | Minor dead code and unreferenced local variables |
| `style` | $0.5\times$ | 8 | Formatting variances and naming conventions |
| `advisory` | $0.25\times$ | 8 | Trivial comments and informational notices |

### Dynamic rule capping

To ensure a single repetitive rule does not collapse the entire project score, per-rule deductions are capped dynamically based on project size:

$$\text{capScale} = \max\left(1, \frac{\text{fileCount}}{10}\right)$$

The deduction for any single rule is bounded by:

$$\text{effectiveCap} = \min(\text{maxPerRule}, \text{ruleCap}) \times \text{capScale}$$

By default, `maxPerRule` is 40. In a project with 50 source files, `capScale` is 5.0, so an advisory rule with a base cap of 8 cannot deduct more than 40 points across the repository.

## Deduction density and scaling

Deductions are summed across all rules and normalized against the project size:

$$\text{deductionDensity} = \frac{\sum \text{cappedDeductions}}{\text{fileCount} + \text{smoothingConstant}}$$

- `fileCount`: Total source files scanned. When running on pre-filtered diagnostics without file lists, Raigal uses observed files with findings.
- `smoothingConstant`: Set to 5 by default (with a fallback to 10 in raw calculations). This smoothing factor prevents small projects or single-file edits from receiving disproportionately harsh score drops.
- Density scaling: The density is multiplied by a calibration factor of 5 to determine final scaled deductions:

$$\text{scaledDeductions} = \text{deductionDensity} \times 5$$

## Logarithmic score compression

The final score compresses scaled deductions into a stable 0–100 range:

$$\text{score} = \max\left(0, \text{round}\left(100 - \frac{100 \times \ln(1 + \text{scaledDeductions})}{\ln(1 + 100 + \text{scaledDeductions})}\right)\right)$$

This mathematical curve provides key characteristics:
- A repository with 0 findings receives a score of 100.
- Marginal penalties are highest for the first few findings, rewarding clean codebases for staying clean.
- Large legacy codebases with hundreds of findings asymptotically approach 0, guaranteeing the score never drops below 0 without hard truncations.

## Score thresholds and status bands

Raigal relies exclusively on numeric 0–100 scores. Letter grades (A–E) are not used.

### CLI status labels

When running `raigal scan` or `raigal ci`, the CLI assigns status labels based on configured thresholds (`good: 75`, `ok: 50`):

| Score range | CLI label | Exit code in CI |
|---|---|---|
| $75 - 100$ | `Healthy` | `0` (Passing) |
| $50 - 74$ | `Needs Work` | `0` (or `1` if `--fail-on` or `failBelow` triggered) |
| $< 50$ | `Critical` | `1` (Fails quality gate by default) |

In CI environments, `raigal ci` fails with exit code 1 if the score is lower than `ci.failBelow` (default: 70) or if any error-level finding is detected.

### Web dashboard bands

The Raigal Cloud web interface (`app.raigal.dev`) categorizes repositories into five health bands:

| Score range | Dashboard band | Color token |
|---|---|---|
| $90 - 100$ | `Excellent` | Lime green |
| $80 - 89$ | `Good` | Emerald green |
| $70 - 79$ | `Fair` | Amber / yellow |
| $60 - 69$ | `Poor` | Orange |
| $< 60$ | `Critical` | Red |
