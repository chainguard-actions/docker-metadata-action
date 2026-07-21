<!-- markdownlint-disable -->

# Hardening Report: docker--metadata-action/v4.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--metadata-action/v4.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in the workflow files use mutable version tags instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references include: actions/checkout@v3, docker/setup-qemu-action@v2, docker/setup-buildx-action@v2, docker/build-push-action@v4, crazy-max/ghaction-dump-context@v2, docker/bake-action@v3, actions/github-script@v6, codecov/codecov-action@v3.

Locations:

- `.github/workflows/ci.yml:23`
- `.github/workflows/ci.yml:163`
- `.github/workflows/ci.yml:167`
- `.github/workflows/ci.yml:178`
- `.github/workflows/ci.yml:183`
- `.github/workflows/ci.yml:186`
- `.github/workflows/ci.yml:220`
- `.github/workflows/ci.yml:224`
- `.github/workflows/ci.yml:237`
- `.github/workflows/ci.yml:258`
- `.github/workflows/ci.yml:262`
- `.github/workflows/test.yml:15`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:23`
- `.github/workflows/validate.yml:17`
- `.github/workflows/validate.yml:32`

### script-injection (severity: high)

Multiple `run:` blocks in ci.yml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing template substitution before the shell parses the string. (1) The 'JSON output' step echoes `${{ fromJSON(steps.meta.outputs.json).labels['maintainer'] }}` and similar expressions directly into echo commands. (2) The 'Inspect image' step runs `docker pull ${{ env.DOCKER_IMAGE }}:${{ steps.docker_meta.outputs.version }}` and `docker image inspect ${{ env.DOCKER_IMAGE }}:${{ steps.docker_meta.outputs.version }}`. (3) The 'Check manifest' step runs `docker buildx imagetools inspect ${{ env.DOCKER_IMAGE }}:${{ steps.docker_meta.outputs.version }}`. All of these should use env: variables with quoted shell expansions instead.

Locations:

- `.github/workflows/ci.yml:156`
- `.github/workflows/ci.yml:175`
- `.github/workflows/ci.yml:181`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no individual job within them defines a `permissions:` key either. Without explicit permissions, workflows run with the default repository permissions (which may include write access to contents, packages, etc.), violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/validate.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across ci.yml, test.yml, and validate.yml:

1. unpinned-uses: Pinned all 8 action references to full 40-char SHAs with original tags as comments: actions/checkout@v3, docker/setup-qemu-action@v2, docker/setup-buildx-action@v2, docker/build-push-action@v4, crazy-max/ghaction-dump-context@v2, docker/bake-action@v3, actions/github-script@v6, codecov/codecov-action@v3.

2. script-injection: Fixed 3 vulnerable run: blocks in ci.yml by moving ${{ }} expressions into env: blocks and referencing them as plain shell variables. The 'JSON output' step was refactored to use jq for JSON parsing instead of fromJSON() template expressions.

3. missing-permissions: Added 'permissions: contents: read' top-level block to all three workflow files, enforcing least-privilege access.

