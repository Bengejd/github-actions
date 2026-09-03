# set-timezone

[![Test: set-timezone](https://github.com/Bengejd/github-actions/actions/workflows/test-set-timezone.yaml/badge.svg)](https://github.com/Bengejd/github-actions/actions/workflows/test-set-timezone.yaml)

Sets the system timezone on the current runner. Use it when a build or test suite depends on local time, for example when snapshot tests contain formatted dates or when a scheduler is exercised against a specific zone.

Works on Linux, macOS, and Windows, including Linux jobs that run inside a container.

## Usage

```yaml
- uses: Bengejd/github-actions/set-timezone@main
  with:
    timezone-linux: America/Chicago
```

Only the input for the current operating system is read, so one step can serve a matrix job:

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v7
      - uses: Bengejd/github-actions/set-timezone@main
        with:
          timezone-linux: Europe/Berlin
          timezone-macos: Europe/Berlin
          timezone-windows: W. Europe Standard Time
      - run: date
```

For production workflows, pin to a commit SHA instead of a branch so the behavior can't change underneath you.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `timezone-linux` | `UTC` | IANA name, such as `America/New_York`. List valid names with `timedatectl list-timezones`. |
| `timezone-macos` | `GMT` | IANA name, such as `Europe/London`. List valid names with `sudo systemsetup -listtimezones`. |
| `timezone-windows` | `UTC` | Windows identifier, such as `Eastern Standard Time`. List valid names with `tzutil /l`. |

An invalid timezone fails the step with an error annotation that names the command for listing valid values.

## Platform notes

Each operating system has its own quirks. The action handles them so your workflow doesn't have to.

**Linux.** Uses `timedatectl`. Inside containers and on some self-hosted runners, `timedatectl` is missing or polkit denies it. The action then points `/etc/localtime` at the zoneinfo file and writes `/etc/timezone`, which is what `timedatectl` would have done. Invalid names are rejected before anything changes.

**macOS.** `systemsetup -settimezone` returns exit code 0 whether or not it worked, and on recent macOS versions it intermittently logs `Error:-99` and leaves the zone unchanged. The action validates the name against the zoneinfo database first, retries `systemsetup` up to five times while confirming the result through the `/etc/localtime` symlink, and writes the symlink itself if the tool keeps failing.

**Windows.** Runs `tzutil` from PowerShell rather than bash. Git Bash rewrites the `/s` switch into the drive path `S:/` before `tzutil` sees it.

## Tests

The [test workflow](../.github/workflows/test-set-timezone.yaml) runs whenever this directory changes and covers:

| Job | What it checks |
| --- | --- |
| Validate action.yaml | The metadata file passes the GitHub Action schema. |
| Explicit timezone | The requested zone is applied on Ubuntu, macOS, and Windows, verified with the platform's own tooling and the UTC offset reported by `date`. |
| Default timezone | Running with no inputs resets a previously changed zone to the documented defaults. |
| Rejects invalid timezone | A nonexistent zone fails the step on all three runners. |
| Linux fallback | Inside an `ubuntu:24.04` container with no `timedatectl`, the symlink path produces the correct zone. |
