<!-- markdownlint-disable -->

# Hardening Report: maxim-lobanov--setup-xcode/v1.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **maxim-lobanov--setup-xcode/v1.7.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Print output variables' step in e2e.yml directly interpolates `${{ steps.setup-xcode.outputs.version }}` and `${{ steps.setup-xcode.outputs.path }}` inside a `run:` shell command string. These are `steps.*.outputs.*` expressions — workflow-controllable values that flow through YAML template substitution before the shell sees them, enabling script injection. They should be passed via `env:` variables and referenced as quoted shell variables (e.g., `"$VERSION"`) instead.

Locations:

- `.github/workflows/e2e.yml:41`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v4` (e2e.yml line 33, workflow.yml line 10), `actions/setup-node@v4` (workflow.yml line 14). Each should be replaced with a full SHA pin, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/e2e.yml:33`
- `.github/workflows/workflow.yml:10`
- `.github/workflows/workflow.yml:14`

### missing-permissions (severity: medium)

`workflow.yml` has no top-level `permissions:` key and no job-level `permissions:` key on the `Build` job. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all`), granting broader access than necessary. A minimal `permissions:` block (e.g., `contents: read`) should be added at the top level or on each job.

Locations:

- `.github/workflows/workflow.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings: (1) script-injection in e2e.yml 'Print output variables' step — moved steps.setup-xcode.outputs.version and steps.setup-xcode.outputs.path into an env: block as XCODE_VERSION and XCODE_PATH, referenced as plain shell variables; (2) unpinned-uses — pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 in both e2e.yml and workflow.yml, and actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 in workflow.yml; (3) missing-permissions — added top-level 'permissions: contents: read' block to workflow.yml (e2e.yml already had this block).

