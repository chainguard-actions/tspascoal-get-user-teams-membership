<!-- markdownlint-disable -->

# Hardening Report: tspascoal--get-user-teams-membership/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tspascoal--get-user-teams-membership/v2.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'validate teams' run: block in the integration tests workflow directly interpolates GitHub Actions expressions into shell commands (rule a). The expressions `${{ steps.get-teams.outputs.teams }}`, `${{ steps.get-teams.outputs.teams}}`, and `${{ env.user }}` are substituted into the shell script before execution, allowing an attacker who can influence step outputs or env vars to inject arbitrary shell commands. Offending lines include:
  - `echo "found teams ${{ steps.get-teams.outputs.teams }}"`
  - `numberTeams=$(echo '${{ steps.get-teams.outputs.teams}}' | jq length)`
  - `echo "Expected 4 team membership for ${{ env.user }}" >> $GITHUB_SUMMARY`
  - `< $(echo '${{ steps.get-teams.outputs.teams }}' | jq --sort-keys .)`
  - `echo 'Teams mismatch ${{ steps.get-teams.outputs.teams }}' >> $GITHUB_SUMMARY`

Locations:

- `.github/workflows/integration tests.yml:41`

### unpinned-uses (severity: high)

Two `uses:` references in the integration tests workflow use mutable tag refs instead of pinned 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced tags are moved or overwritten:
  - `actions/checkout@v3` (line 20) — should be pinned to a full SHA
  - `peter-murray/workflow-application-token-action@v2` (line 27) — should be pinned to a full SHA

Locations:

- `.github/workflows/integration tests.yml:20`
- `.github/workflows/integration tests.yml:27`

### missing-permissions (severity: medium)

The workflow file '.github/workflows/integration tests.yml' has no top-level `permissions:` key and the single job 'test' also has no `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions (which may include write access to contents, packages, etc.), violating the principle of least privilege.

Locations:

- `.github/workflows/integration tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/integration tests.yml: (1) Pinned actions/checkout@v3 to full SHA a37ce9120846195fa4ece8f58b268e6043cb2f26 and peter-murray/workflow-application-token-action@v2 to full SHA 8e1ba3bf1619726336414f1014e37f17fbadf1db, preserving original tags in comments. (2) Added top-level 'permissions: contents: read' block. (3) Moved all ${{ steps.get-teams.outputs.teams }} and ${{ env.user }} expressions from the 'validate teams' run: block into the step's env: block as TEAMS_OUTPUT and ENV_USER, referencing them as plain shell variables to prevent script injection.

