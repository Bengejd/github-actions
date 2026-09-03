# setup-git-bot

[![Test: setup-git-bot](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-git-bot.yaml/badge.svg)](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-git-bot.yaml)

Configures git so that commits made later in the job are attributed to the `github-actions[bot]` account. Use it before any step that commits or tags, such as a release step, a changelog update, or a generated-file refresh.

Works on Linux, macOS, and Windows.

## Usage

```yaml
steps:
  - uses: actions/checkout@v7
  - uses: Bengejd/github-actions/setup-git-bot@main
  - run: |
      git add CHANGELOG.md
      git commit -m "chore: update changelog"
      git push
```

For production workflows, pin to a commit SHA instead of a branch so the behavior can't change underneath you.

## What it sets

The action has no inputs. It writes two values to the global git config:

| Key | Value |
| --- | --- |
| `user.name` | `github-actions[bot]` |
| `user.email` | `github-actions[bot]@users.noreply.github.com` |

GitHub recognizes this email and renders the commits with the bot's avatar and a **bot** label.

## Behavior

**Global scope.** The values go into the runner's global config, not the repository's, so they apply to every repository the job touches, including ones cloned after this step runs. On GitHub-hosted runners the global config starts empty and is discarded with the runner. On a self-hosted runner it persists between jobs, so run the action in every job that needs it rather than relying on an earlier job.

**Replaces, never appends.** An identity configured earlier in the job is overwritten. Each key ends up with exactly one value.

**Composes with `actions/checkout`.** The checkout action stores its auth token in the repository's local config and leaves identity alone, so the two don't conflict. Run this action in either order relative to checkout.

**Pushing still needs a token.** This action only sets who the commits are from. Pushing requires `contents: write` permission on the job, or a token with equivalent scope passed to `actions/checkout`.

## Tests

The [test workflow](../.github/workflows/test-setup-git-bot.yaml) runs whenever this directory changes and covers:

| Job | What it checks |
| --- | --- |
| Validate action.yaml | The metadata file passes the GitHub Action schema. |
| Configures identity | On Ubuntu, macOS, and Windows, the runner starts with no identity, the action sets both values, and a real commit in the checked-out repository has the bot as both author and committer. |
| Overrides an existing identity | A previously configured name and email are replaced, with a single value per key. |
| Running twice is a no-op | A second run leaves the global config unchanged. |
