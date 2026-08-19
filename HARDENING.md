<!-- markdownlint-disable -->

# Hardening Report: tj-actions--git-cliff/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--git-cliff/v2.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags/versions instead of immutable full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved.

In action.yml:
- `tj-actions/setup-bin@v1.2.3` (line 22)

In .github/workflows/sync-release-version.yml:
- `tj-actions/release-tagger@v4` (line 19)
- `tj-actions/sync-release-version@v13` (line 21)
- `tj-actions/git-cliff@v2` (line 28)

In .github/workflows/test.yml:
- `reviewdog/action-shellcheck@v1` (line 20)

In .github/workflows/update-readme.yml:
- `tj-actions/auto-doc@v3` (line 21)
- `tj-actions/remark@v3` (line 23)
- `tj-actions/verify-changed-files@v20` (line 26)

Locations:

- `action.yml:22`
- `.github/workflows/sync-release-version.yml:19`
- `.github/workflows/sync-release-version.yml:21`
- `.github/workflows/sync-release-version.yml:28`
- `.github/workflows/test.yml:20`
- `.github/workflows/update-readme.yml:21`
- `.github/workflows/update-readme.yml:23`
- `.github/workflows/update-readme.yml:26`

### script-injection (severity: high)

Sub-rule (a): The 'Generate a changelog' step in action.yml directly interpolates GitHub Actions expressions inside a `run:` shell command string. The expressions `${{ steps.install-git-cliff.outputs.binary_path }}`, `${{ steps.git-cliff.outputs.output_path }}`, and `${{ inputs.output }}` are substituted by the Actions template engine before the shell ever sees them. An attacker controlling the calling workflow can supply a malicious `inputs.output` value (e.g. containing shell metacharacters) that gets executed as shell code. The offending lines are:
  `${{ steps.install-git-cliff.outputs.binary_path }} --config "${{ steps.git-cliff.outputs.output_path }}" \`
  `  --output "${{ inputs.output }}" \`
All three expressions must be moved to `env:` variables and then referenced as double-quoted shell variables (e.g. `"$BINARY_PATH"`).

Locations:

- `action.yml:36`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key and neither of its jobs (`shellcheck`, `test`) defines a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents and other scopes). A minimal `permissions: read-all` or specific per-job permissions should be added.

Locations:

- `.github/workflows/test.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.output }}" appears directly in run: block of step "Generate a changelog"; move to env: map

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, missing-permissions

**Notes:**

Fixed all findings across 4 files:

1. action.yml:
   - Pinned `tj-actions/setup-bin@v1.2.3` → `@99513d7b4f4e970d0c33c83c5a6d45a59e86f468 # v1.2.3`
   - Fixed script injection in 'Generate a changelog' step: moved `${{ steps.install-git-cliff.outputs.binary_path }}`, `${{ steps.git-cliff.outputs.output_path }}`, and `${{ inputs.output }}` into env vars (BINARY_PATH, CONFIG_PATH, OUTPUT_FILE) and referenced them as double-quoted shell variables.

2. .github/workflows/sync-release-version.yml:
   - Pinned `tj-actions/release-tagger@v4` → `@1a9264b0fd99a1ef92c4fd2f077f292900cc79b6 # v4`
   - Pinned `tj-actions/sync-release-version@v13` → `@2a7ef0deb39b3ecce887ee99d2261c6cef989d84 # v13`
   - Pinned `tj-actions/git-cliff@v2` → `@679041f051a4d2ab452f7e5e7b0eed2abee21131 # v2`

3. .github/workflows/test.yml:
   - Added top-level `permissions: contents: read` block
   - Pinned `reviewdog/action-shellcheck@v1` → `@4c07458293ac342d477251099501a718ae5ef86e # v1`

4. .github/workflows/update-readme.yml:
   - Pinned `tj-actions/auto-doc@v3` → `@b10ceedffd794ec29a8fa8700529f40c1b64a951 # v3`
   - Pinned `tj-actions/remark@v3` → `@10fc40701928cbafcc4a2d241679579d218144ff # v3`
   - Pinned `tj-actions/verify-changed-files@v20` → `@a1c6acee9df209257a246f2cc6ae8cb6581c1edf # v20`

