<!-- markdownlint-disable -->

# Hardening Report: docker--metadata-action/v6.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--metadata-action/v6.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `run:` blocks in ci.yml directly interpolate `${{ }}` expressions into shell command strings, allowing script injection. (1) The "JSON output" step interpolates `${{ fromJSON(steps.meta.outputs.json).labels['maintainer'] }}` and similar step-output values directly into `echo` commands. (2) The "Inspect image" step interpolates `${{ env.DOCKER_IMAGE }}` and `${{ steps.docker_meta.outputs.version }}` directly into `docker pull` and `docker image inspect` commands. (3) The "Check manifest" step interpolates `${{ env.DOCKER_IMAGE }}` and `${{ steps.docker_meta.outputs.version }}` directly into a `docker buildx imagetools inspect` command. Any `${{ ... }}` expression inside a `run:` block is substituted by the Actions template engine before the shell ever sees the string, enabling injection of shell metacharacters.

Locations:

- `.github/workflows/ci.yml:234`
- `.github/workflows/ci.yml:284`
- `.github/workflows/ci.yml:290`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection vulnerabilities in .github/workflows/ci.yml:
1. 'JSON output' step (line ~234): Moved four `fromJSON(steps.meta.outputs.json).labels[...]` expressions into env vars (META_MAINTAINER, META_VERSION, META_REVISION, META_CREATED) and replaced inline ${{ }} interpolation with plain $VAR references in the echo commands.
2. 'Inspect image' step (line ~284): Moved `${{ env.DOCKER_IMAGE }}` and `${{ steps.docker_meta.outputs.version }}` into env vars (DOCKER_IMAGE_REF, DOCKER_META_VERSION) and replaced inline interpolation with quoted "$VAR" references in docker pull and docker image inspect commands.
3. 'Check manifest' step (line ~290): Same env vars (DOCKER_IMAGE_REF, DOCKER_META_VERSION) used to replace inline interpolation in the docker buildx imagetools inspect command.

