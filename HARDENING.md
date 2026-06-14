<!-- markdownlint-disable -->

# Hardening Report: tj-actions--git-cliff/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **tj-actions--git-cliff/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references `tj-actions/setup-bin@v1.2.3` using a mutable version tag instead of a pinned 40-character commit SHA. This allows the referenced action to be silently replaced with malicious code if the tag is moved.

Locations:

- `action.yml:28`

### script-injection (severity: high)

The 'Generate a changelog' run: block directly interpolates GitHub Actions expressions inside the shell command string (rule a). The offending lines are:
  `${{ steps.install-git-cliff.outputs.binary_path }} --config "${{ steps.git-cliff.outputs.output_path }}" \`
  `--output "${{ inputs.output }}" \`
All three expressions — `steps.install-git-cliff.outputs.binary_path`, `steps.git-cliff.outputs.output_path`, and `inputs.output` — are substituted directly into the shell before execution. An attacker who can influence these values (e.g. via a crafted `inputs.output` value from a calling workflow) can inject arbitrary shell commands. These values should be passed via `env:` variables and referenced as double-quoted shell variables instead.

Locations:

- `action.yml:35`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.output }}" appears directly in run: block of step "Generate a changelog"; move to env: map

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

1. Pinned tj-actions/setup-bin@v1.2.3 to full SHA 99513d7b4f4e970d0c33c83c5a6d45a59e86f468 with tag preserved as comment. 2. Moved all three ${{ }} expressions from the 'Generate a changelog' run block into an env: block (BINARY_PATH, CONFIG_PATH, OUTPUT_FILE) and referenced them as double-quoted shell variables. Since inputs.output has a default value of 'HISTORY.md', it is never empty and can be safely used with a simple double-quoted reference.

