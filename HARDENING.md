<!-- markdownlint-disable -->

# Hardening Report: tspascoal--get-user-teams-membership/v4.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tspascoal--get-user-teams-membership/v4.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple `run:` blocks in the 'validate teams' step directly interpolate `${{ steps.get-teams.outputs.teams }}` and `${{ env.user }}` into shell commands. The action's output (`steps.get-teams.outputs.teams`) is attacker-influenced data that flows through YAML template substitution before the shell sees it, enabling command injection. Offending lines include: `echo "found teams ${{ steps.get-teams.outputs.teams }}"`, `$(echo '${{ steps.get-teams.outputs.teams}}' | jq length)`, `echo "Expected 4 team membership for ${{ env.user }}"`, and `$(echo '${{ steps.get-teams.outputs.teams }}' | jq --sort-keys .)`. Similarly, the 'Not a team member? Fail' step interpolates `${{ env.check-team }}` and the 'Dummy Team membership? Fail' step interpolates `${{ env.check-not-team }}` directly in run: blocks. All ${{ ... }} expressions must be moved to env: vars and then double-quoted in the shell script.

Locations:

- `.github/workflows/tests.yml:48`
- `.github/workflows/tests.yml:49`
- `.github/workflows/tests.yml:51`
- `.github/workflows/tests.yml:56`
- `.github/workflows/tests.yml:58`
- `.github/workflows/tests.yml:73`
- `.github/workflows/tests.yml:88`

### unpinned-uses (severity: high)

Two `uses:` references in the workflow are pinned to mutable tags rather than immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the tag is moved: `actions/checkout@v6` (line 25) and `actions/create-github-app-token@v3` (line 33). These should be pinned to full SHA digests, e.g. `actions/checkout@<40-char-sha> # v6`.

Locations:

- `.github/workflows/tests.yml:25`
- `.github/workflows/tests.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed two findings in hardened/action/.github/workflows/tests.yml:
1. unpinned-uses: Pinned actions/checkout@v6 to SHA df4cb1c069e1874edd31b4311f1884172cec0e10 and actions/create-github-app-token@v3 to SHA bcd2ba49218906704ab6c1aa796996da409d3eb1, preserving original tags in comments.
2. script-injection: Moved all ${{ steps.get-teams.outputs.teams }}, ${{ env.user }}, ${{ env.check-team }}, and ${{ env.check-not-team }} expressions from run: blocks into step-level env: blocks (as TEAMS_OUTPUT, USER_NAME, CHECK_TEAM, CHECK_NOT_TEAM respectively). Shell scripts now reference these as plain environment variables, preventing command injection via YAML template substitution.

