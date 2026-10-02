<!-- markdownlint-disable -->

# Hardening Report: tj-actions--git-cliff/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--git-cliff/v2.2.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references `tj-actions/setup-bin@v1.2.3` using a mutable version tag instead of a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved or the repository is compromised.

Locations:

- `action.yml:38`

### script-injection (severity: high)

The 'Generate a changelog' run: block directly interpolates multiple ${{ }} expressions into the shell command string (rule a), enabling script injection:
- `${{ steps.install-git-cliff.outputs.binary_path }}` is used as the executable command itself (line 47)
- `${{ steps.git-cliff.outputs.output_path }}` is interpolated in the --config argument (line 47)
- `${{ inputs.output }}` is interpolated in the --output argument (line 48)
- `${{ inputs.args }}` is interpolated unquoted at the end of the command (line 49), allowing an attacker to inject arbitrary shell commands via the `args` input.
All four expressions undergo YAML template substitution before the shell ever sees them, bypassing any quoting.

Locations:

- `action.yml:47`
- `action.yml:48`
- `action.yml:49`

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed all four findings in hardened/action/action.yml:
1. Pinned tj-actions/setup-bin@v1.2.3 to full commit SHA 99513d7b4f4e970d0c33c83c5a6d45a59e86f468 with tag comment.
2. Moved all four ${{ }} expressions from the 'Generate a changelog' run: block into an env: block: BINARY_PATH, CONFIG_PATH, INPUT_OUTPUT, and INPUT_ARGS.
3. The binary path and config/output paths are referenced as double-quoted shell variables.
4. inputs.args is a list-style input, so it is tokenized via xargs into a bash array (with the required guard for empty values) to preserve argument boundaries and prevent injection.

