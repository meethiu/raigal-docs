---
description: "Command-line interface reference and flags for Raigal."
icon: terminal
---

# CLI commands and flags

Raigal provides commands for local code checks, automatic fixes, agent repair sessions, and CI quality gates.

## Command index

| Command | Action |
|---|---|
| `raigal [dir]` | Interactive terminal dashboard or default scan |
| `raigal scan [dir]` | Run full quality scan and output score |
| `raigal fix [dir]` | Apply deterministic autofixes or generate agent prompts |
| `raigal ci [dir]` | Quality gate with threshold exit codes for CI/CD |
| `raigal agent [dir]` | Autonomous repair sessions in isolated worktrees (Claude, Cursor, OpenCode) |
| `raigal hook` | Manage AI coding agent real-time interception hooks |
| `raigal report-pr-closed` | Synchronize PR closed/merged lifecycle with Raigal Cloud |
| `raigal telemetry` | Inspect recorded cloud run payloads and outbox status |
| `raigal login` | Sign in via browser OAuth or API token |
| `raigal logout` | Clear local credentials and cached offline leases |
| `raigal whoami` | Inspect active credentials, user, organization, and allowed GitHub repos |
| `raigal doctor` | Check system binaries, tools, and engine status |
| `raigal rules` | Display rule explanations and impact weights |
| `raigal init` | Create configuration file and optional CI workflow |

## Command reference

### `raigal scan`

Evaluates the target directory and outputs the score breakdown.

```bash
raigal scan [options] [path]
```

Options:
- `--changes`: scan files changed relative to git `HEAD` only.
- `--staged`: scan staged git files only (useful in pre-commit hooks).
- `--base <ref>`: compare diff against a specific git reference (such as `origin/main`).
- `-d, --verbose`: show file-level details for each rule.
- `--json`: output full diagnostic tree as JSON.
- `--sarif`: output diagnostics in OASIS SARIF 2.1.0 format.
- `--format <format>`: set output format (`json` or `sarif`).
- `--fail-on <level>`: exit with code 1 on findings matching `none`, `error`, or `warning`.

### `raigal ci`

Runs quality evaluations in CI runners and exits with non-zero codes on failure.

```bash
raigal ci [options] [path]
```

- Blocks pull requests if the overall score falls below the configured threshold (default: 80).
- Accepts `--human` to output formatted summaries in CI logs.
- `--changes`: gate files changed relative to `--base` (or `HEAD`) only.
- `--staged`: gate staged files only.
- `--base <ref>`: diff base for `--changes` (such as `origin/main`).
- `--sarif`: output SARIF 2.1.0 for GitHub code scanning annotations.

### `raigal fix`

Applies automated fixes for safe formatting and pattern violations.

```bash
raigal fix [options] [path]
```

- `--safe`: applies changes with preserved semantics only (imports, comment removal, standard formatting).
- `-f, --force`: runs broader fixes including dependency audits and framework alignment.
- `--dry-run`: previews proposed fixes without modifying files.
- `-p, --prompt`: formats remaining unfixed issues into a prompt for your coding agent.
- `--changes` / `--staged`: restrict fixes to changed or staged files.
- `--base <ref>`: diff base for `--changes`.

### `raigal agent`

Runs automated repair sessions using local coding tools inside an isolated git worktree.

```bash
raigal agent [options] [path]
```

Flags:
- `--provider <provider>`: agent provider to invoke (`auto`, `codex`, `claude`, `opencode`).
- `--target-score <n>`: score threshold to reach (default: 90).
- `--max-turns <n>`: maximum provider turns for one repair attempt (default: 4).
- `--limit <n>`: maximum number of findings to pass to the provider (default: 8).
- `--in-place`: edit current working tree directly instead of creating an isolated worktree.
- `--apply`: apply accepted diff back to the original worktree upon completion.
- `-y, --yes`: skip confirmation prompts for `--apply`.
- `--dry-run`: print the selected provider and execution plan without running.
- `--background`: start the agent process in the background and return immediately.
- `--no-fix`: skip safe automated fixes before handing off to the provider.
- `--commit`: commit the verified diff onto a repair branch.
- `--pr`: push the branch and open a pull request on GitHub.
- `--branch <name>`: branch name for `--commit` or `--pr`.
- `--base <branch>`: target base branch for `--pr`.
- `--title <title>`: pull request title for `--pr`.
- `--commit-message <msg>`: commit message.
- `--ready`: open a ready-for-review pull request instead of a draft.
- `--cleanup`: remove the generated worktree even if changes remain.

Subcommands:
- `raigal agent plan [dir]`: preview provider, worktree, findings, and publish actions without editing.
- `raigal agent connect [provider]`: authenticate with a local coding-agent provider using its CLI credentials.
- `raigal agent providers`: list detected local coding-agent providers and status.
- `raigal agent use [provider]`: set or display the default provider for this repository.
- `raigal agent sessions [dir]`: list active and past repair sessions.
- `raigal agent show <session-id>`: inspect diff and metadata from an agent session.
- `raigal agent apply <session-id>`: merge changes from a session into your working branch.
- `raigal agent monitor [dir]`: watch git changes and run repairs when scores drop below target.

### `raigal report-pr-closed`

Reports a pull request closed or merged event to Raigal Cloud to update the remediation board.

```bash
raigal report-pr-closed --pr <number> [--merged]
```

- `--pr <number>`: pull request number (read automatically from `GITHUB_EVENT_PATH` in GitHub Actions).
- `--merged`: indicates whether the pull request was merged into the base branch.

### `raigal login` and `raigal logout`

Manage Raigal Cloud credentials and offline lease authentication.

```bash
# Interactive browser OAuth login
raigal login

# Headless login via API token from stdin
echo "rgl_live_..." | raigal login --with-token

# Clear local credentials and offline cache
raigal logout
```

### `raigal whoami`

Displays active user, organization, plan tier, and authorized GitHub repositories.

```bash
raigal whoami
```

### `raigal telemetry`

Inspects recorded cloud run telemetry payloads and outbox transmission status.

```bash
raigal telemetry --show
raigal telemetry --show --json
```

### `raigal doctor`

Verifies local environment, linters, formatters, and coding-agent binaries.

```bash
raigal doctor
```
