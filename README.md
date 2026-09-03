# GitHub Actions

Reusable composite actions for GitHub workflows. Each action lives in its own directory with its own README, runs on Linux, macOS, and Windows, and ships with a workflow that tests it on all three.

[![Test: set-timezone](https://github.com/Bengejd/github-actions/actions/workflows/test-set-timezone.yaml/badge.svg)](https://github.com/Bengejd/github-actions/actions/workflows/test-set-timezone.yaml)
[![Test: setup-git-bot](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-git-bot.yaml/badge.svg)](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-git-bot.yaml)
[![Test: setup-node-with-cache](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-node-with-cache.yaml/badge.svg)](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-node-with-cache.yaml)

## Actions

| Action | Description | Status |
| --- | --- | --- |
| [`set-timezone`](set-timezone/README.md) | Set the runner's system timezone | [![Test: set-timezone](https://github.com/Bengejd/github-actions/actions/workflows/test-set-timezone.yaml/badge.svg)](https://github.com/Bengejd/github-actions/actions/workflows/test-set-timezone.yaml) |
| [`setup-git-bot`](setup-git-bot/README.md) | Configure git to commit as `github-actions[bot]` | [![Test: setup-git-bot](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-git-bot.yaml/badge.svg)](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-git-bot.yaml) |
| [`setup-node-with-cache`](setup-node-with-cache/README.md) | Install Node.js and npm, yarn, or pnpm with a cached, frozen-lockfile install | [![Test: setup-node-with-cache](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-node-with-cache.yaml/badge.svg)](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-node-with-cache.yaml) |

## Usage

Reference an action by its directory. Each action's README documents its inputs and platform behavior.

```yaml
- uses: Bengejd/github-actions/set-timezone@main
  with:
    timezone-linux: America/Chicago
```

For production workflows, pin to a commit SHA instead of a branch so the behavior can't change underneath you:

```yaml
- uses: Bengejd/github-actions/set-timezone@<commit-sha>
```

## Testing

Every action has a dedicated workflow named `test-<action>.yaml`. The workflow runs only when the action or the workflow itself changes, so unrelated commits don't burn runner minutes, and its badge reports on that action alone. Each workflow covers the success path on every supported OS, the failure path for bad input, and any fallback branches the action contains.

To validate an action's metadata locally without pushing:

```bash
npx --yes @action-validator/cli set-timezone/action.yaml
```

The runner jobs need real GitHub-hosted machines. Trigger them from the **Actions** tab with **Run workflow**, or push a change under the action's directory.

## Repository layout

```
.
├── .github/workflows/
│   └── test-<action>.yaml   # One test workflow per action
└── <action>/
    ├── action.yaml          # The action
    └── README.md            # Usage, inputs, and platform notes
```

## Adding an action

1. Create a directory named after the action and add an `action.yaml` with `runs.using: composite`.
2. Give every `run:` step an explicit `shell:`, and pass inputs through `env:` rather than interpolating `${{ inputs.* }}` into shell text.
3. Add a `README.md` in the directory covering usage, inputs, and anything platform-specific.
4. Add `.github/workflows/test-<action>.yaml`, scoped with `paths:` filters to the action's directory and the workflow file.
5. Add the action to the table at the top of this file with its badge.
