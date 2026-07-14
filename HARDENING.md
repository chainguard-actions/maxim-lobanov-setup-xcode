<!-- markdownlint-disable -->

# Hardening Report: maxim-lobanov--setup-xcode/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **maxim-lobanov--setup-xcode/v1.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of `steps.*.outputs.*` values inside `run:` shell command strings. In e2e.yml, three 'Print output variables' steps (one per job) directly interpolate `${{ steps.setup-xcode.outputs.version }}` and `${{ steps.setup-xcode.outputs.path }}` into echo commands. These are `steps.*.outputs.*` context values — workflow-controllable data that flows through YAML template substitution before the shell ever sees them, making them a script-injection risk. They must be moved to `env:` variables and referenced as quoted shell variables instead.

Locations:

- `.github/workflows/e2e.yml:30`
- `.github/workflows/e2e.yml:31`
- `.github/workflows/e2e.yml:46`
- `.github/workflows/e2e.yml:47`
- `.github/workflows/e2e.yml:63`
- `.github/workflows/e2e.yml:64`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v3` (e2e.yml lines 21, 38, 55; workflow.yml line 13) and `actions/setup-node@v3` (workflow.yml line 17). Each should be replaced with the corresponding full SHA, e.g. `actions/checkout@<40-hex-sha> # v3`.

Locations:

- `.github/workflows/e2e.yml:21`
- `.github/workflows/e2e.yml:38`
- `.github/workflows/e2e.yml:55`
- `.github/workflows/workflow.yml:13`
- `.github/workflows/workflow.yml:17`

### missing-permissions (severity: medium)

Neither workflow file declares a top-level `permissions:` key, and no individual job within them declares job-level `permissions:` either. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be `write-all` for older repositories), granting unintended write access. A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level of each workflow.

Locations:

- `.github/workflows/e2e.yml:1`
- `.github/workflows/workflow.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across e2e.yml and workflow.yml: (1) script-injection: moved `steps.setup-xcode.outputs.version` and `steps.setup-xcode.outputs.path` from inline `${{ }}` expressions in `run:` blocks to `env:` variables (SETUP_XCODE_VERSION, SETUP_XCODE_PATH) referenced as plain shell variables in all three jobs; (2) unpinned-uses: pinned `actions/checkout@v3` to SHA `f43a0e5ff2bd294095638e18286ca9a3d1956744` and `actions/setup-node@v3` to SHA `3235b876344d2a9aa001b8d1453c930bba69e610`, with `# v3` comments for readability; (3) missing-permissions: added `permissions: contents: read` at the top level of both workflow files.

