---
description: "How Raigal computes the 0-100 quality score, weighted penalties, and density-adjusted normalization."
icon: gauge
---

# Scoring engine

Raigal scores codebases on a scale from 0 to 100, where 100 means no issues were detected.

## Score bands

| Score range | Status | Meaning | Exit code in CI |
|---|---|---|---|
| 90-100 | Healthy | Clean architecture, no critical defects, no pattern violations | `0` (Pass) |
| 75-89 | Warning | Minor formatting issues, trivial comments, or small debt | `0` (Pass unless threshold set) |
| 0-74 | Failing | Structural problems, high violation density, or security warnings | `1` (Blocked) |

## Scoring formula

Instead of subtracting a fixed number of points for each finding, Raigal scales penalties by code density:

$$D = \frac{\sum (\text{weight}_i \times \text{severity}_i)}{\text{KLOC}}$$

Where:
- $\text{KLOC}$ is total physical lines of code divided by 1,000.
- Each rule carries a weight multiplier. Security findings carry heavier penalties, while trivial comment penalties taper off.
- Files under 100 lines use a baseline denominator so small commits are not penalized disproportionately.

## Category weights

- Security and vulnerabilities (1.5x): exposed credentials, SQL injections, insecure deserialization.
- Pattern violations and slop (1.2x): redundant null checks, unnecessary fallbacks, repetitive comments.
- Architecture and boundaries (1.0x): circular dependencies, cross-module boundary violations.
- Code quality and complexity (0.9x): high cyclomatic complexity, dead code branches.
- Formatting and style (0.6x): indentation and whitespace inconsistencies.
