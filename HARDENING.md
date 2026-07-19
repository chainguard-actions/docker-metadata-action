<!-- markdownlint-disable -->

# Hardening Report: docker--metadata-action/v6.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--metadata-action/v6.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Three `run:` blocks in ci.yml directly interpolate `${{ ... }}` expressions inside shell commands, enabling script injection. (1) The 'JSON output' step uses `${{ fromJSON(steps.meta.outputs.json).labels['maintainer'] }}` and similar `steps.*.outputs.*` expressions directly in `echo` commands. (2) The 'Inspect image' step uses `${{ env.DOCKER_IMAGE }}` and `${{ steps.docker_meta.outputs.version }}` directly in `docker pull` and `docker image inspect` commands. (3) The 'Check manifest' step uses `${{ env.DOCKER_IMAGE }}` and `${{ steps.docker_meta.outputs.version }}` directly in a `docker buildx imagetools inspect` command. All three cases allow YAML template substitution to inject arbitrary shell metacharacters before the shell ever sees the value.

Locations:

- `.github/workflows/ci.yml:264`
- `.github/workflows/ci.yml:321`
- `.github/workflows/ci.yml:327`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection vulnerabilities in .github/workflows/ci.yml:
1. 'JSON output' step: moved fromJSON(steps.meta.outputs.json).labels[...] expressions into env: block (META_MAINTAINER, META_VERSION, META_REVISION, META_CREATED) and referenced as plain shell variables.
2. 'Inspect image' step: moved ${{ env.DOCKER_IMAGE }} and ${{ steps.docker_meta.outputs.version }} into env: block (DOCKER_IMAGE_REF, DOCKER_META_VERSION) and used double-quoted shell variables in docker pull/inspect commands.
3. 'Check manifest' step: same fix as 'Inspect image' — moved expressions into env: block and used double-quoted shell variables in docker buildx imagetools inspect command.

