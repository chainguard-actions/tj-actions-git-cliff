<!-- markdownlint-disable -->

# Hardening Report: tj-actions--git-cliff/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--git-cliff/v2.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Generate a changelog' run: block in action.yml directly interpolates ${{ }} expressions inside the shell command string (sub-rule a). Three expressions are interpolated: `${{ steps.install-git-cliff.outputs.binary_path }}` is used as the command to execute (an attacker who can influence step outputs controls what binary is run), `${{ steps.git-cliff.outputs.output_path }}` is passed as a --config argument, and `${{ inputs.output }}` (a caller-controlled input) is passed as --output. All three are ${{ ... }} template substitutions that occur before the shell parses the command, enabling shell metacharacter injection. These should be moved to env: variables and referenced as quoted shell variables (e.g., "$BINARY_PATH").

Locations:

- `action.yml:41`
- `action.yml:42`

### unpinned-uses (severity: high)

The composite action step 'Install git-cliff' references `tj-actions/setup-bin@v1.2.3`, which uses a mutable version tag rather than a full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit without notice, enabling supply-chain attacks. Pin to a specific commit SHA, e.g., `tj-actions/setup-bin@<40-char-sha> # v1.2.3`.

Locations:

- `action.yml:31`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.output }}" appears directly in run: block of step "Generate a changelog"; move to env: map

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed three findings in hardened/action/action.yml: (1) Pinned tj-actions/setup-bin@v1.2.3 to full commit SHA 99513d7b4f4e970d0c33c83c5a6d45a59e86f468. (2) & (3) Moved all three ${{ }} expressions from the 'Generate a changelog' run: block into an env: map (BINARY_PATH, CONFIG_PATH, OUTPUT_FILE) and referenced them as quoted shell variables ("$BINARY_PATH", "$CONFIG_PATH", "$OUTPUT_FILE") to prevent shell metacharacter injection.

