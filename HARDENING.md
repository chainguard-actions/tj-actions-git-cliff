<!-- markdownlint-disable -->

# Hardening Report: tj-actions--git-cliff/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--git-cliff/v2.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Generate a changelog' step in action.yml directly interpolates GitHub Actions expressions inside a `run:` shell command string. The offending lines are:
  `${{ steps.install-git-cliff.outputs.binary_path }} --config "${{ steps.git-cliff.outputs.output_path }}" \
    --output "${{ inputs.output }}" \
    --verbose`
All three expressions — `steps.install-git-cliff.outputs.binary_path`, `steps.git-cliff.outputs.output_path`, and `inputs.output` — are substituted directly into the shell command before the shell parses it, enabling command injection if any value contains shell metacharacters.

Locations:

- `action.yml:39`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks:
- action.yml: `tj-actions/setup-bin@v1.2.3`
- .github/workflows/sync-release-version.yml: `tj-actions/release-tagger@v4`, `tj-actions/sync-release-version@v13`, `tj-actions/git-cliff@v2`
- .github/workflows/test.yml: `reviewdog/action-shellcheck@v1`
- .github/workflows/update-readme.yml: `tj-actions/auto-doc@v3`, `tj-actions/remark@v3`, `tj-actions/verify-changed-files@v20`

Locations:

- `action.yml:30`
- `.github/workflows/sync-release-version.yml:14`
- `.github/workflows/sync-release-version.yml:16`
- `.github/workflows/sync-release-version.yml:21`
- `.github/workflows/test.yml:14`
- `.github/workflows/update-readme.yml:14`
- `.github/workflows/update-readme.yml:17`
- `.github/workflows/update-readme.yml:20`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the workflow inherits the default repository token permissions, which may be overly broad.

Locations:

- `.github/workflows/test.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.output }}" appears directly in run: block of step "Generate a changelog"; move to env: map

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings:
1. script-injection/static-inline-injection in action.yml: Moved all three ${{ }} expressions (binary_path, output_path, inputs.output) from the 'Generate a changelog' run: block into an env: map (BINARY_PATH, CONFIG_PATH, OUTPUT_FILE). Shell script now uses plain env vars with proper quoting.
2. unpinned-uses: Pinned all mutable tag references to immutable commit SHAs across action.yml, sync-release-version.yml, test.yml, and update-readme.yml. All SHAs verified via lookup_action_sha.
3. missing-permissions in test.yml: Added top-level 'permissions: {}' and job-level 'permissions: contents: read' for both the shellcheck and test jobs.

