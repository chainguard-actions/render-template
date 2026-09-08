<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template--binary/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template--binary/v1.12** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell command strings. In the 'Download binary' step, `${{ runner.os }}` and `${{ runner.arch }}` are interpolated directly into the shell script: `OS=$(echo "${{ runner.os }}" | tr '[:upper:]' '[:lower:]')` and `ARCH=$(echo "${{ runner.arch }}" | tr '[:upper:]' '[:lower:]')`. Any ${{ ... }} expression inside a run: block is a script-injection risk regardless of context (runner.*, env.*, etc.) because the value is substituted by the YAML template engine before the shell ever sees it.

Locations:

- `action.yml:44`
- `action.yml:46`

### script-injection (severity: high)

Sub-rule (a): The entire 'Run' step command is a direct ${{ }} expression: `run: "${{ env.RENDER_TEMPLATE_BIN }}"`. The env.RENDER_TEMPLATE_BIN context value is interpolated directly into the run: field before the shell executes it, making this a script-injection vulnerability.

Locations:

- `action.yml:82`

### github-env-injection (severity: high)

The 'Download binary' step writes `echo "RENDER_TEMPLATE_BIN=$DEST/$BINARY" >> "$GITHUB_ENV"` without sanitization. The $BINARY variable is constructed from $OS and $ARCH, which were set via direct ${{ runner.os }} and ${{ runner.arch }} expression interpolations — workflow-controllable values. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write to $GITHUB_ENV, allowing newline injection to add arbitrary environment variables.

Locations:

- `action.yml:72`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all three findings in action.yml:
1. Moved `${{ runner.os }}` and `${{ runner.arch }}` from direct shell interpolation into the 'Download binary' step's env: block as RUNNER_OS and RUNNER_ARCH, then referenced them as plain shell variables.
2. Replaced `run: "${{ env.RENDER_TEMPLATE_BIN }}"` in the Run step with `run: "$RENDER_TEMPLATE_BIN"` where RENDER_TEMPLATE_BIN is set in the env: block from `${{ steps.download.outputs.bin }}`.
3. Replaced the unsanitized write to $GITHUB_ENV with a sanitized write to $GITHUB_OUTPUT using `printf '%s' "$DEST/$BINARY" | tr -d '\n\r'`, and added `id: download` to the Download binary step so the Run step can reference `steps.download.outputs.bin`.

