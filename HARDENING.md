<!-- markdownlint-disable -->

# Hardening Report: tj-actions--git-cliff/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--git-cliff/v2.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Generate a changelog' run: block in action.yml directly interpolates multiple ${{ }} expressions inside the shell command string (sub-rule a). This includes attacker-controlled inputs: `${{ inputs.args }}` (unquoted, sub-rule b) and `${{ inputs.output }}`, as well as workflow-controllable step outputs `${{ steps.install-git-cliff.outputs.binary_path }}` and `${{ steps.git-cliff.outputs.output_path }}`. Any of these values can contain shell metacharacters that will be interpreted by bash before the shell ever sees them. The offending lines are:
  - `${{ steps.install-git-cliff.outputs.binary_path }} --config "${{ steps.git-cliff.outputs.output_path }}" \`
  - `--output "${{ inputs.output }}" \`
  - `${{ inputs.args }}`
Fix: move all values into env: vars and reference them as quoted shell variables (e.g. `"$INPUT_ARGS"`)

Locations:

- `action.yml:43`
- `action.yml:44`
- `action.yml:45`

### unpinned-uses (severity: high)

The composite action step 'Install git-cliff' references `tj-actions/setup-bin@v1.2.3`, which uses a mutable version tag rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack. Fix: pin to a full SHA, e.g. `uses: tj-actions/setup-bin@<40-char-sha> # v1.2.3`.

Locations:

- `action.yml:34`

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

1. Pinned tj-actions/setup-bin@v1.2.3 to full SHA 99513d7b4f4e970d0c33c83c5a6d45a59e86f468.
2. In the 'Generate a changelog' step, moved all ${{ }} expressions into an env: block: BINARY_PATH, CONFIG_PATH, INPUT_OUTPUT, INPUT_ARGS. inputs.args is a list-style input (extra CLI args for git-cliff), so it is tokenized via xargs into a bash array using the null-delimited read loop pattern, preserving argument boundaries and quoting. The other three values are single values referenced as quoted shell variables.

