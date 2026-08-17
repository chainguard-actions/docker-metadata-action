<!-- markdownlint-disable -->

# Hardening Report: docker--metadata-action/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--metadata-action/v6.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Three `run:` blocks in ci.yml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing script injection if any of those values contain shell metacharacters.

1. "JSON output" step: `echo "maintainer=${{ fromJSON(steps.meta.outputs.json).labels['maintainer'] }}"` and three similar echo lines — `steps.*.outputs.*` values are interpolated directly into the shell string without going through an env: variable.

2. "Inspect image" step: `docker pull ${{ env.DOCKER_IMAGE }}:${{ steps.docker_meta.outputs.version }}` — both `env.*` and `steps.*.outputs.*` are interpolated directly.

3. "Check manifest" step: `docker buildx imagetools inspect ${{ env.DOCKER_IMAGE }}:${{ steps.docker_meta.outputs.version }}` — same issue.

All three should route values through `env:` variables and double-quote the shell expansions.

Locations:

- `.github/workflows/ci.yml:196`
- `.github/workflows/ci.yml:240`
- `.github/workflows/ci.yml:246`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions and reusable workflows by mutable version tags instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

Failing references include (non-exhaustive):
- `actions/checkout@v6` (all workflow files)
- `docker/setup-buildx-action@v3` (ci.yml)
- `docker/build-push-action@v6` (ci.yml)
- `docker/bake-action@v6` (ci.yml, test.yml, update-dist.yml, validate.yml)
- `docker/bake-action/subaction/list-targets@v6` (validate.yml)
- `actions/github-script@v8` (ci.yml)
- `crazy-max/ghaction-dump-context@v2` (ci.yml)
- `actions/create-github-app-token@v2` (update-dist.yml)
- `codecov/codecov-action@v5` (test.yml)
- `actions/publish-immutable-action@v0.0.4` (publish.yml)

All should be pinned to full SHA digests (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`).

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/publish.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-dist.yml:1`
- `.github/workflows/validate.yml:1`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` block and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, GitHub Actions grants the default token permissions (which may include `contents: write` on some repository configurations), violating the principle of least privilege.

Affected files:
- `ci.yml` — 18 jobs, none with permissions
- `test.yml` — 1 job, no permissions
- `update-dist.yml` — 1 job, no permissions (note: this job also checks out a PR head ref via `github.event.pull_request.head.ref`, making broad permissions especially risky)
- `validate.yml` — 2 jobs, none with permissions

Each file should declare a minimal top-level `permissions:` block (e.g. `permissions: contents: read`) or add job-level permissions to every job.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-dist.yml:1`
- `.github/workflows/validate.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across five workflow files:

1. script-injection (ci.yml): Moved all ${{ }} expressions out of run: blocks into env: blocks for the 'JSON output', 'Inspect image', and 'Check manifest' steps. Shell commands now reference plain environment variables with double-quoting.

2. unpinned-uses: Pinned all 10 action references to full 40-character commit SHAs across ci.yml, publish.yml, test.yml, update-dist.yml, and validate.yml. Tag names preserved as comments.

3. missing-permissions: Added top-level `permissions: contents: read` to ci.yml, test.yml, update-dist.yml, and validate.yml. For update-dist.yml, the job that commits and pushes dist changes has a job-level `permissions: contents: write` override since it needs to push to the repository.

