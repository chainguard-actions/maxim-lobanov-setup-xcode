<!-- markdownlint-disable -->

# Hardening Report: maxim-lobanov--setup-xcode/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **maxim-lobanov--setup-xcode/v1.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In .github/workflows/e2e.yml, three 'Print output variables' run: blocks directly interpolate ${{ steps.setup-xcode.outputs.version }} and ${{ steps.setup-xcode.outputs.path }} into shell commands. These are steps.*.outputs.* context values — a workflow-controllable source — and are subject to script injection (sub-rule a). An attacker who can influence the action's outputs could inject arbitrary shell commands. Offending lines:
  echo "Version: ${{ steps.setup-xcode.outputs.version }}"
  echo "Path: ${{ steps.setup-xcode.outputs.path }}"
These appear in all three job blocks (versions-macOS-11, versions-macOS-12, versions-macOS-13).

Locations:

- `.github/workflows/e2e.yml:29`
- `.github/workflows/e2e.yml:30`
- `.github/workflows/e2e.yml:45`
- `.github/workflows/e2e.yml:46`
- `.github/workflows/e2e.yml:61`
- `.github/workflows/e2e.yml:62`

### unpinned-uses (severity: high)

Multiple uses: references in workflow files are pinned to mutable version tags rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved or the upstream repository is compromised.

.github/workflows/e2e.yml:
  - uses: actions/checkout@v3  (lines 21, 37, 53)

.github/workflows/workflow.yml:
  - uses: actions/checkout@v3  (line 10)
  - uses: actions/setup-node@v3  (line 13)

All should be pinned to their full SHA digest, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3

Locations:

- `.github/workflows/e2e.yml:21`
- `.github/workflows/e2e.yml:37`
- `.github/workflows/e2e.yml:53`
- `.github/workflows/workflow.yml:10`
- `.github/workflows/workflow.yml:13`

### missing-permissions (severity: medium)

Neither .github/workflows/e2e.yml nor .github/workflows/workflow.yml declares a top-level permissions: key, and no individual job within either file declares its own permissions: block. Without explicit permissions, GitHub Actions grants the default token permissions (which may include write access to contents, packages, etc. depending on repository settings), violating the principle of least privilege. Both files should declare minimal required permissions at the top level or per job.

Locations:

- `.github/workflows/e2e.yml:1`
- `.github/workflows/workflow.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across .github/workflows/e2e.yml and .github/workflows/workflow.yml:

1. script-injection: Moved all six ${{ steps.setup-xcode.outputs.version }} and ${{ steps.setup-xcode.outputs.path }} expressions from run: shell strings into env: blocks (as XCODE_VERSION and XCODE_PATH) in all three job blocks of e2e.yml. Shell commands now reference plain $XCODE_VERSION and $XCODE_PATH variables.

2. unpinned-uses: Pinned actions/checkout@v3 to @a37ce9120846195fa4ece8f58b268e6043cb2f26 (all 4 occurrences across both files) and actions/setup-node@v3 to @3235b876344d2a9aa001b8d1453c930bba69e610 in workflow.yml. SHA comments preserved with # v3.

3. missing-permissions: Added `permissions: {}` at the top level of both e2e.yml and workflow.yml to enforce least privilege.

