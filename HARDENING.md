<!-- markdownlint-disable -->

# Hardening Report: docker--metadata-action/v6.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **docker--metadata-action/v6.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `run:` blocks in ci.yml directly interpolate `${{ ... }}` expressions into shell commands, which is a script injection risk.

1. "JSON output" step: `echo "maintainer=${{ fromJSON(steps.meta.outputs.json).labels['maintainer'] }}"` — `steps.meta.outputs.json` is interpolated directly into the shell command string.

2. "Inspect image" step: `docker pull ${{ env.DOCKER_IMAGE }}:${{ steps.docker_meta.outputs.version }}` — both `env.DOCKER_IMAGE` and `steps.docker_meta.outputs.version` are interpolated directly.

3. "Check manifest" step: `docker buildx imagetools inspect ${{ env.DOCKER_IMAGE }}:${{ steps.docker_meta.outputs.version }}` — same issue as above.

All three cases allow YAML template substitution to inject arbitrary shell metacharacters before the shell ever parses the command. The values should be passed via `env:` variables and then referenced as `"$VAR"` (double-quoted) in the shell script.

Locations:

- `.github/workflows/ci.yml:232`
- `.github/workflows/ci.yml:285`
- `.github/workflows/ci.yml:291`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection vulnerabilities in .github/workflows/ci.yml:
1. 'JSON output' step: Moved four fromJSON() label expressions into env: block (META_MAINTAINER, META_VERSION, META_REVISION, META_CREATED) and referenced them as plain shell variables.
2. 'Inspect image' step: Moved steps.docker_meta.outputs.version into env: block as DOCKER_META_VERSION; used double-quoted $DOCKER_IMAGE:$DOCKER_META_VERSION in shell commands.
3. 'Check manifest' step: Same fix as 'Inspect image' — moved steps.docker_meta.outputs.version into env: block as DOCKER_META_VERSION and double-quoted the shell variable references.

