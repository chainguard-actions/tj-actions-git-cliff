<!-- markdownlint-disable -->

# Hardening Report: tj-actions--git-cliff/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--git-cliff/v2.2.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Generate a changelog' run: step in action.yml directly interpolates GitHub Actions expressions inside the shell command string. The expressions ${{ steps.install-git-cliff.outputs.binary_path }}, ${{ steps.git-cliff.outputs.output_path }}, ${{ inputs.output }}, and ${{ inputs.args }} are all substituted by the Actions runner before the shell ever sees the command, allowing an attacker who controls these values (e.g. via inputs.args or inputs.output) to inject arbitrary shell commands. These must be moved to env: variables and then referenced as double-quoted shell variables.

Locations:

- `action.yml:46`

### unpinned-uses (severity: high)

action.yml references tj-actions/setup-bin@v1.2.3 (a mutable version tag, not a full 40-character commit SHA). This is vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `action.yml:35`

### unpinned-uses (severity: high)

sync-release-version.yml references three actions pinned to mutable tags instead of full commit SHAs: tj-actions/release-tagger@v4, tj-actions/sync-release-version@v13, and tj-actions/git-cliff@v2. These are vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/sync-release-version.yml:16`
- `.github/workflows/sync-release-version.yml:18`
- `.github/workflows/sync-release-version.yml:23`

### unpinned-uses (severity: high)

test.yml references reviewdog/action-shellcheck@v1 pinned to a mutable tag instead of a full commit SHA. This is vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/test.yml:15`

### unpinned-uses (severity: high)

update-readme.yml references three actions pinned to mutable tags instead of full commit SHAs: tj-actions/auto-doc@v3, tj-actions/remark@v3, and tj-actions/verify-changed-files@v20. These are vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/update-readme.yml:16`
- `.github/workflows/update-readme.yml:21`
- `.github/workflows/update-readme.yml:23`

### missing-permissions (severity: medium)

test.yml has no top-level permissions: key and no job-level permissions: keys on any of its jobs (shellcheck, test). Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad. A minimal permissions block (e.g. contents: read) should be added.

Locations:

- `.github/workflows/test.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.output }}" appears directly in run: block of step "Generate a changelog"; move to env: map

Locations:

- `action.yml:53`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step "Generate a changelog"; move to env: map

Locations:

- `action.yml:54`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 8 findings across 4 files:

1. action.yml - script-injection/static-inline-injection: Moved ${{ steps.install-git-cliff.outputs.binary_path }}, ${{ steps.git-cliff.outputs.output_path }}, ${{ inputs.output }}, and ${{ inputs.args }} from the run: block into an env: map. Used `read -ra extra_args <<< "$INPUT_ARGS"` to safely split the args string into an array for execution.

2. action.yml - unpinned-uses: Pinned tj-actions/setup-bin@v1.2.3 → SHA 99513d7b4f4e970d0c33c83c5a6d45a59e86f468.

3. .github/workflows/sync-release-version.yml - unpinned-uses: Pinned tj-actions/release-tagger@v4 → 1a9264b0fd99a1ef92c4fd2f077f292900cc79b6, tj-actions/sync-release-version@v13 → 2a7ef0deb39b3ecce887ee99d2261c6cef989d84, tj-actions/git-cliff@v2 → 679041f051a4d2ab452f7e5e7b0eed2abee21131.

4. .github/workflows/test.yml - unpinned-uses: Pinned reviewdog/action-shellcheck@v1 → 4c07458293ac342d477251099501a718ae5ef86e.

5. .github/workflows/test.yml - missing-permissions: Added top-level `permissions: contents: read`.

6. .github/workflows/update-readme.yml - unpinned-uses: Pinned tj-actions/auto-doc@v3 → b10ceedffd794ec29a8fa8700529f40c1b64a951, tj-actions/remark@v3 → 10fc40701928cbafcc4a2d241679579d218144ff, tj-actions/verify-changed-files@v20 → a1c6acee9df209257a246f2cc6ae8cb6581c1edf.

