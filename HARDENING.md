<!-- markdownlint-disable -->

# Hardening Report: tj-actions--git-cliff/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **tj-actions--git-cliff/v2.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Generate a changelog' run block in action.yml directly interpolates multiple ${{ ... }} expressions into the shell command string (sub-rule a). This includes attacker-controlled inputs: `${{ inputs.output }}` and `${{ inputs.args }}` are caller-supplied and embedded directly in the shell command without quoting or sanitization. `${{ inputs.args }}` is completely unquoted, allowing shell metacharacter injection (`;`, `|`, `&`, `$(...)`, etc.). Additionally, `${{ steps.install-git-cliff.outputs.binary_path }}` and `${{ steps.git-cliff.outputs.output_path }}` are also interpolated directly. All four expressions should be moved to `env:` variables and referenced as double-quoted shell variables instead.

Offending lines:
  Line 41: `${{ steps.install-git-cliff.outputs.binary_path }} --config "${{ steps.git-cliff.outputs.output_path }}" \`
  Line 42: `  --output "${{ inputs.output }}" \`
  Line 43: `  ${{ inputs.args }}`

Locations:

- `action.yml:41`
- `action.yml:42`
- `action.yml:43`

### unpinned-uses (severity: high)

The composite action step 'Install git-cliff' uses `tj-actions/setup-bin@v1.2.3`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling supply-chain attacks. It should be pinned to a full SHA, e.g. `tj-actions/setup-bin@<40-char-sha> # v1.2.3`.

Locations:

- `action.yml:33`

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

1. Pinned tj-actions/setup-bin@v1.2.3 to full SHA 99513d7b4f4e970d0c33c83c5a6d45a59e86f468 (comment preserves tag name). 2. Moved all four ${{ }} expressions from the 'Generate a changelog' run block into an env: map: GIT_CLIFF_BINARY (${{ steps.install-git-cliff.outputs.binary_path }}), GIT_CLIFF_CONFIG (${{ steps.git-cliff.outputs.output_path }}), GIT_CLIFF_OUTPUT (${{ inputs.output }}), and GIT_CLIFF_ARGS (${{ inputs.args }}). The run block now references these as shell variables with the binary and config/output paths double-quoted. GIT_CLIFF_ARGS is left unquoted to preserve word-splitting for multiple CLI flags (matching original intent).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in action.yml at line 53. The `$GIT_CLIFF_ARGS` variable (sourced from user-controlled `${{ inputs.args }}` input) was used unquoted in the `run:` block of the 'Generate a changelog' step. Changed `$GIT_CLIFF_ARGS` to `"$GIT_CLIFF_ARGS"` to prevent shell metacharacter injection. The variable was already correctly placed in the `env:` block; only the unquoted expansion in the shell script needed to be fixed.

