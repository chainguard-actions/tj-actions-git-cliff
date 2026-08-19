<!-- markdownlint-disable -->

# Hardening Report: tj-actions--git-cliff/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--git-cliff/v2.1.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Generate a changelog' run block in action.yml directly interpolates GitHub Actions expressions inside the shell command string (sub-rule a). The offending lines are:
  `${{ steps.install-git-cliff.outputs.binary_path }} --config "${{ steps.git-cliff.outputs.output_path }}" \
    --output "${{ inputs.output }}" \
    ${{ inputs.args }}`
Both `inputs.output` and `inputs.args` are user-controlled inputs, and `steps.*` outputs are workflow-controllable. Any of these values can contain shell metacharacters that will be interpreted by the shell before quoting can take effect. The values must be moved into `env:` variables and then referenced as double-quoted shell variables (e.g. `"$INPUT_ARGS"`) instead of being interpolated directly.

Locations:

- `action.yml:44`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved.

In action.yml:
  - `tj-actions/setup-bin@v1.2.3`

In .github/workflows/sync-release-version.yml:
  - `tj-actions/release-tagger@v4`
  - `tj-actions/sync-release-version@v13`
  - `tj-actions/git-cliff@v2`

In .github/workflows/test.yml:
  - `reviewdog/action-shellcheck@v1`

In .github/workflows/update-readme.yml:
  - `tj-actions/auto-doc@v3`
  - `tj-actions/remark@v3`
  - `tj-actions/verify-changed-files@v20`

All of these should be replaced with full SHA pins (e.g. `uses: tj-actions/setup-bin@<40-hex-char-sha> # v1.2.3`).

Locations:

- `action.yml:35`
- `.github/workflows/sync-release-version.yml:14`
- `.github/workflows/sync-release-version.yml:16`
- `.github/workflows/sync-release-version.yml:20`
- `.github/workflows/test.yml:14`
- `.github/workflows/update-readme.yml:14`
- `.github/workflows/update-readme.yml:18`
- `.github/workflows/update-readme.yml:20`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all` in some configurations), granting broader access than necessary. A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level or on each job.

Locations:

- `.github/workflows/test.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.output }}" appears directly in run: block of step "Generate a changelog"; move to env: map

Locations:

- `action.yml:48`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step "Generate a changelog"; move to env: map

Locations:

- `action.yml:49`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across 4 files:

1. action.yml - script-injection/static-inline-injection: Moved ${{ steps.install-git-cliff.outputs.binary_path }}, ${{ steps.git-cliff.outputs.output_path }}, ${{ inputs.output }}, and ${{ inputs.args }} out of the run: block into an env: map (BINARY_PATH, CONFIG_PATH, INPUT_OUTPUT, INPUT_ARGS). The shell script now references these as environment variables. Also pinned tj-actions/setup-bin@v1.2.3 → @99513d7b4f4e970d0c33c83c5a6d45a59e86f468.

2. .github/workflows/test.yml - missing-permissions: Added top-level `permissions: contents: read`. unpinned-uses: Pinned reviewdog/action-shellcheck@v1 → @4c07458293ac342d477251099501a718ae5ef86e.

3. .github/workflows/sync-release-version.yml - unpinned-uses: Pinned tj-actions/release-tagger@v4 → @1a9264b0fd99a1ef92c4fd2f077f292900cc79b6, tj-actions/sync-release-version@v13 → @2a7ef0deb39b3ecce887ee99d2261c6cef989d84, tj-actions/git-cliff@v2 → @679041f051a4d2ab452f7e5e7b0eed2abee21131.

4. .github/workflows/update-readme.yml - unpinned-uses: Pinned tj-actions/auto-doc@v3 → @b10ceedffd794ec29a8fa8700529f40c1b64a951, tj-actions/remark@v3 → @10fc40701928cbafcc4a2d241679579d218144ff, tj-actions/verify-changed-files@v20 → @a1c6acee9df209257a246f2cc6ae8cb6581c1edf.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Generate a changelog' step of action.yml. The original code expanded `$INPUT_ARGS` unquoted (with `# shellcheck disable=SC2086`), allowing shell metacharacter injection via `inputs.args`. The fix uses a bash array: `read -ra args <<< "$INPUT_ARGS"` to split arguments on whitespace only (not on shell metacharacters), then expands with `"${args[@]}"` to keep each element properly quoted. This prevents injection of `;`, `|`, `&`, `$(...)`, etc. while still allowing multiple space-separated arguments to be passed correctly.

