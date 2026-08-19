<!-- markdownlint-disable -->

# Hardening Report: tspascoal--get-user-teams-membership/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tspascoal--get-user-teams-membership/v4.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The 'validate teams' run block directly interpolates GitHub Actions expressions inside shell commands. Specifically, `${{ steps.get-teams.outputs.teams }}` is interpolated into `echo "found teams ${{ steps.get-teams.outputs.teams }}"` and `$(echo '${{ steps.get-teams.outputs.teams}}' | jq length)`, and `${{ env.user }}` is interpolated into an echo command. Step outputs and env context values flow through YAML template substitution before the shell sees them, allowing an attacker to inject shell metacharacters. These expressions must be moved to an `env:` block and the env vars must be double-quoted in the shell script.

Locations:

- `.github/workflows/integration tests.yml:38`

### unpinned-uses (severity: high)

Two `uses:` references in the workflow are pinned to mutable tags rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved: (1) `actions/checkout@v6` — should be pinned to a SHA; (2) `peter-murray/workflow-application-token-action@v4` — should be pinned to a SHA.

Locations:

- `.github/workflows/integration tests.yml:22`
- `.github/workflows/integration tests.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed in .github/workflows/integration tests.yml: (1) Pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and peter-murray/workflow-application-token-action@v4 to SHA d17e3a9a36850ea89f35db16c1067dd2b68ee343, preserving original tags as comments. (2) Fixed script injection in the 'validate teams' step by moving ${{ steps.get-teams.outputs.teams }} and ${{ env.user }} into an env: block as TEAMS and USER, then referencing them as double-quoted shell variables ($TEAMS, $USER) throughout the run script.

