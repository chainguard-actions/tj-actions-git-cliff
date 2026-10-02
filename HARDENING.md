<!-- markdownlint-disable -->

# Hardening Report: tj-actions--git-cliff/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--git-cliff/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Generate a changelog' run: block in action.yml directly interpolates GitHub Actions expressions inside shell command strings (sub-rule a). Three expressions are used: `${{ steps.install-git-cliff.outputs.binary_path }}` is used as the executable path, `${{ steps.git-cliff.outputs.output_path }}` is passed as a --config argument, and `${{ inputs.output }}` is passed as an --output argument. Any of these values could contain shell metacharacters that would be interpreted by the shell before quoting takes effect. These should be moved to env: variables and referenced as quoted shell variables (e.g., "$BINARY_PATH").

Locations:

- `action.yml:36`

### unpinned-uses (severity: high)

The action uses `tj-actions/setup-bin@v1.2.3`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack. It should be pinned to a full SHA, e.g. `tj-actions/setup-bin@<40-char-sha> # v1.2.3`.

Locations:

- `action.yml:27`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.output }}" appears directly in run: block of step "Generate a changelog"; move to env: map

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

1. Pinned tj-actions/setup-bin@v1.2.3 to full SHA 99513d7b4f4e970d0c33c83c5a6d45a59e86f468 with the tag preserved as a comment. 2. Moved all three ${{ }} expressions from the 'Generate a changelog' run: block into an env: block (BINARY_PATH, CONFIG_PATH, OUTPUT_FILE) and referenced them as properly quoted shell variables in the run: script, eliminating both the script-injection and static-inline-injection findings.

