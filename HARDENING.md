<!-- markdownlint-disable -->

# Hardening Report: agrc--reminder-action/v1.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **agrc--reminder-action/v1.0.22** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable tags or version strings instead of full 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised.

Failing references:
- pull_request.yml: actions/checkout@v7, pnpm/action-setup@v6.0.9, actions/setup-node@v7 (multiple), actions/create-github-app-token@v3.2.0, peter-evans/create-or-update-comment@v5
- push.yml: agrc/release-composite-action@v1
- issue_comment.yml: agrc/create-reminder-action@v1
- scheduled.yml: agrc/reminder-action@v1

Locations:

- `.github/workflows/pull_request.yml:14`
- `.github/workflows/pull_request.yml:19`
- `.github/workflows/pull_request.yml:24`
- `.github/workflows/pull_request.yml:43`
- `.github/workflows/pull_request.yml:48`
- `.github/workflows/pull_request.yml:53`
- `.github/workflows/pull_request.yml:62`
- `.github/workflows/pull_request.yml:68`
- `.github/workflows/pull_request.yml:73`
- `.github/workflows/pull_request.yml:78`
- `.github/workflows/pull_request.yml:100`
- `.github/workflows/pull_request.yml:105`
- `.github/workflows/pull_request.yml:110`
- `.github/workflows/pull_request.yml:119`
- `.github/workflows/pull_request.yml:130`
- `.github/workflows/push.yml:21`
- `.github/workflows/issue_comment.yml:14`
- `.github/workflows/scheduled.yml:14`

### missing-permissions (severity: medium)

pull_request.yml has no top-level `permissions:` key, and the jobs `test-check` and `test-unit` also have no job-level `permissions:` block. This means those jobs run with the default (broad) repository permissions. Only the `build` and `integration-test` jobs define explicit job-level permissions.

Locations:

- `.github/workflows/pull_request.yml:1`

### script-injection (severity: high)

Sub-rule (a): The 'Commit and push if needed' run block in pull_request.yml directly interpolates `${{ secrets.UGRC_RELEASE_BOT_NAME }}` and `${{ secrets.UGRC_RELEASE_BOT_EMAIL }}` inside shell commands. Any `${{ ... }}` expression interpolated directly into a `run:` block undergoes YAML template substitution before the shell processes it, bypassing shell quoting and enabling injection if the secret values contain shell metacharacters.

Offending lines:
  git config user.name "${{ secrets.UGRC_RELEASE_BOT_NAME }}"
  git config user.email "${{ secrets.UGRC_RELEASE_BOT_EMAIL }}"

Locations:

- `.github/workflows/pull_request.yml:91`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across four workflow files:

1. unpinned-uses: Pinned all 8 action references to full 40-char SHA digests with original tags preserved as comments. Files affected: pull_request.yml (actions/checkout, pnpm/action-setup, actions/setup-node, actions/create-github-app-token, peter-evans/create-or-update-comment), push.yml (agrc/release-composite-action), issue_comment.yml (agrc/create-reminder-action), scheduled.yml (agrc/reminder-action).

2. missing-permissions: Added top-level `permissions: contents: read` to pull_request.yml, and added explicit `permissions: contents: read` to the test-check and test-unit jobs that previously had no permissions block.

3. script-injection: In the 'Commit and push if needed' step of the build job, moved `${{ secrets.UGRC_RELEASE_BOT_NAME }}` and `${{ secrets.UGRC_RELEASE_BOT_EMAIL }}` into an `env:` block as BOT_NAME and BOT_EMAIL, then referenced them as plain shell variables in the run script.

