# setup-node-with-cache

[![Test: setup-node-with-cache](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-node-with-cache.yaml/badge.svg)](https://github.com/Bengejd/github-actions/actions/workflows/test-setup-node-with-cache.yaml)

Installs Node.js and your package manager, restores the dependency cache keyed on the lockfile, and runs a frozen-lockfile install. One step replaces the usual four, and every project in an organization gets the same install behavior.

Works on Linux, macOS, and Windows with npm, yarn, or pnpm.

## Usage

```yaml
steps:
  - uses: actions/checkout@v7
  - uses: Bengejd/github-actions/setup-node-with-cache@main
  - run: pnpm test
```

Pick a package manager and pin its version:

```yaml
- uses: Bengejd/github-actions/setup-node-with-cache@main
  with:
    node-version: 26
    package-manager: npm
    package-manager-version: 11.4.1
```

Install a package inside a monorepo:

```yaml
- uses: Bengejd/github-actions/setup-node-with-cache@main
  with:
    working-directory: packages/api
```

Authenticate to GitHub Packages for a private scope:

```yaml
- uses: Bengejd/github-actions/setup-node-with-cache@main
  with:
    registry-token: ${{ secrets.GITHUB_TOKEN }}
    registry-scope: "@acme"
```

For production workflows, pin to a commit SHA instead of a branch so the behavior can't change underneath you.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `node-version` | `26` | Node.js release to install. Accepts anything `actions/setup-node` accepts. |
| `package-manager` | `pnpm` | `npm`, `yarn`, or `pnpm`. Any other value fails the step. |
| `package-manager-version` | `latest` | Release of the package manager. `latest` resolves to the newest available. |
| `working-directory` | `.` | Directory that contains `package.json` and the lockfile. |
| `patch-package` | `false` | Run `patch-package` after the install. Requires `patch-package` in `devDependencies` and a `patches` directory. |
| `registry-token` | | Auth token for a private npm registry. Pass a secret. When empty, no registry authentication is configured. |
| `registry-scope` | | Scopes to resolve from the private registry, such as `@acme`. Accepts a comma- or space-separated list; the leading `@` is optional. Ignored unless `registry-token` is set. |
| `registry-host` | `npm.pkg.github.com` | Hostname of the private registry. A protocol or trailing slash is stripped. |

## Outputs

| Output | Description |
| --- | --- |
| `cache-hit` | `true` when the dependency cache was restored, otherwise `false`. |
| `node-version` | The Node.js version that was installed, for example `26.5.0`. |

## Behavior

**Frozen installs.** The install runs `npm ci`, `yarn install --immutable` (or `--frozen-lockfile` on Yarn 1), or `pnpm install --frozen-lockfile`. If `package.json` and the lockfile disagree, the step fails rather than silently rewriting the lockfile. Commit an updated lockfile to fix it.

**Cache scope.** The cache key is derived from every `package-lock.json`, `yarn.lock`, and `pnpm-lock.yaml` under `working-directory`. Changing any of them produces a new key. What gets cached is the package manager's download cache, not `node_modules`, so the install step still runs on every job but skips network fetches on a hit.

**Package manager provisioning.** pnpm is installed before Node.js because `actions/setup-node` shells out to it to locate the store. yarn is provisioned through corepack, which the action installs on demand since Node 25 and later don't bundle it. npm is upgraded in place with `npm install --global`.

**Empty inputs.** An input set to an empty string, for example from an unset variable, falls back to its default instead of breaking the install.

**Heap size on Linux.** The install step raises Node's old-space limit to three quarters of the machine's memory. Large monorepo installs can exceed the default heap without this.

**Registry credentials.** With `registry-token` set, the action appends an auth line for `registry-host` and a registry line for each scope to `~/.npmrc`. Unscoped packages keep resolving from the public registry. If `registry-scope` is set but the token is empty, the action logs a warning and skips authentication.

## Tests

The [test workflow](../.github/workflows/test-setup-node-with-cache.yaml) runs whenever this directory or its [fixtures](../.github/fixtures/setup-node-with-cache) change. Each fixture is a one-dependency project with a committed lockfile, so the install path runs against a real registry.

| Job | What it checks |
| --- | --- |
| Validate action.yaml | The metadata file passes the GitHub Action schema. |
| Installs with npm, yarn, pnpm | Node 26 and the requested manager are on `PATH`, the dependency resolves from the fixture directory, and both outputs are populated. |
| Installs with npm on macOS and Windows | The npm path works outside Linux. |
| Warm and restore the cache | A second job after a warming job reports `cache-hit: true`. |
| Writes private registry credentials | `~/.npmrc` has the expected auth and scope lines, with the host normalized and both scope spellings accepted. |
| Applies patch-package patches | A committed patch to the fixture's dependency is visible at require time. |
| Rejects an unknown package manager | `package-manager: bun` fails the step. |
