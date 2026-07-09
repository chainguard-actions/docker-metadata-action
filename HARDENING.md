<!-- markdownlint-disable -->

# Hardening Report: docker--metadata-action/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **docker--metadata-action/v6.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned 40-character SHA commits, making them vulnerable to supply-chain attacks. Unpinned refs found:
- ci.yml: actions/checkout@v6, docker/setup-buildx-action@v3, docker/build-push-action@v6, docker/bake-action@v6, actions/github-script@v8, crazy-max/ghaction-dump-context@v2
- test.yml: actions/checkout@v6, docker/bake-action@v6, codecov/codecov-action@v5
- update-dist.yml: actions/create-github-app-token@v2, actions/checkout@v6, docker/bake-action@v6
- validate.yml: actions/checkout@v6, docker/bake-action/subaction/list-targets@v6, docker/bake-action@v6
- publish.yml: actions/checkout@v6, actions/publish-immutable-action@v0.0.4

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-dist.yml:1`
- `.github/workflows/validate.yml:1`
- `.github/workflows/publish.yml:1`

### script-injection (severity: high)

Multiple run: blocks in ci.yml directly interpolate ${{ }} expressions into shell commands (sub-rule a), allowing expression values to be parsed by the shell before execution. (1) 'JSON output' step: echo commands embed ${{ fromJSON(steps.meta.outputs.json).labels['...'] }} directly into shell strings. (2) 'Inspect image' step: docker pull and docker image inspect commands embed ${{ env.DOCKER_IMAGE }} and ${{ steps.docker_meta.outputs.version }} directly. (3) 'Check manifest' step: docker buildx imagetools inspect embeds the same expressions directly. These should be moved to env: variables and referenced as quoted shell variables.

Locations:

- `.github/workflows/ci.yml:196`
- `.github/workflows/ci.yml:248`
- `.github/workflows/ci.yml:255`

### missing-permissions (severity: medium)

The following workflow files have no top-level permissions: key and no job-level permissions: keys on any job, meaning they run with the default (potentially broad) GITHUB_TOKEN permissions: ci.yml, test.yml, update-dist.yml, and validate.yml.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-dist.yml:1`
- `.github/workflows/validate.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

All five workflow files updated. Pinned all action references to full 40-char SHAs. Added top-level 'permissions: contents: read' to ci.yml, test.yml, update-dist.yml, and validate.yml (update-dist.yml also has job-level 'contents: write' for the git push). Fixed script injection in ci.yml: JSON output step now uses META_JSON env var with jq; Inspect image and Check manifest steps use DOCKER_IMAGE_NAME and META_VERSION env vars instead of inline ${{ }} expressions.

