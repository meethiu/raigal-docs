---
description: "Configure project rules, score thresholds, and engines in .raigal/config.yml."
icon: gear
---

# Configuration

Raigal reads its configuration from `.raigal/config.yml` in your repository root.

## Minimal configuration

```yaml
version: 1

scoring:
  threshold: 85
  fail_under: 80

engines:
  format: true
  lint: true
  quality: true
  ai_slop: true
  security: true
  architecture: true

ignore:
  - "dist/**"
  - "node_modules/**"
  - "**/*.min.js"
```

## Options reference

### `scoring`

- `threshold` (number, default: 80): target benchmark score for the repository.
- `fail_under` (number, default: 80): minimum passing score for `raigal ci`. Scores below this trigger exit code `1`.

### `engines`

Enables or disables individual diagnostic modules:
- `ai_slop`: repetitive patterns, redundant defensive checks, and obvious comments.
- `security`: exposed tokens, private keys, and insecure function calls.
- `lint`: static analysis powered by Oxlint and Ruff.
- `format`: code style checks powered by Biome, Gofmt, and Ruff.
- `quality`: unused code detection (Knip) and cyclomatic complexity limits.
- `architecture`: boundary and cross-module import rules.

### `telemetry`

To disable cloud telemetry reporting:

```yaml
telemetry:
  enabled: false
```

Telemetry events contain numeric metrics and rule counts only. Code contents and file contents are never transmitted.
