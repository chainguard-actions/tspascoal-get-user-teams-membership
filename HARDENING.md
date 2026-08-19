<!-- markdownlint-disable -->

# Hardening Report: tspascoal--get-user-teams-membership/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tspascoal--get-user-teams-membership/v3.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### missing-permissions (severity: medium)

The workflow file '.github/workflows/integration tests.yml' has no top-level 'permissions:' key and the single job 'test' also has no 'permissions:' key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/integration tests.yml:1`

### unpinned-uses (severity: high)

Two 'uses:' references in '.github/workflows/integration tests.yml' are pinned to mutable tags rather than full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the tag is moved:
- 'actions/checkout@v3' (line 19)
- 'peter-murray/workflow-application-token-action@v2' (line 27)

Locations:

- `.github/workflows/integration tests.yml:19`
- `.github/workflows/integration tests.yml:27`

### script-injection (severity: high)

The 'validate teams' step in '.github/workflows/integration tests.yml' directly interpolates GitHub Actions expressions inside run: shell commands (sub-rule a). The values from 'steps.get-teams.outputs.teams' and 'env.user' are substituted into the shell script before the shell parses it, allowing an attacker who controls those values to inject arbitrary shell commands. Offending lines include:
- echo "found teams ${{ steps.get-teams.outputs.teams }}" (line 41)
- numberTeams=$(echo '${{ steps.get-teams.outputs.teams}}' | jq length) (line 42)
- echo "Expected 4 team membership for ${{ env.user }}" >> $GITHUB_SUMMARY (line 44)
- < $(echo '${{ steps.get-teams.outputs.teams }}' | jq --sort-keys .) (line 49)
- echo 'Teams mismatch ${{ steps.get-teams.outputs.teams }}' >> $GITHUB_SUMMARY (line 51)

Locations:

- `.github/workflows/integration tests.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** missing-permissions, unpinned-uses, script-injection

**Notes:**

Fixed all three findings in '.github/workflows/integration tests.yml': (1) Added 'permissions: {}' at the top level to enforce least-privilege; (2) Pinned 'actions/checkout@v3' to SHA a37ce9120846195fa4ece8f58b268e6043cb2f26 and 'peter-murray/workflow-application-token-action@v2' to SHA 8e1ba3bf1619726336414f1014e37f17fbadf1db; (3) Moved all ${{ steps.get-teams.outputs.teams }} and ${{ env.user }} expressions in the 'validate teams' step into an env: block (TEAMS_OUTPUT and USER_NAME), then referenced them as plain shell variables to prevent script injection.

