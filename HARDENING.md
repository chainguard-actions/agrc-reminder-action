<!-- markdownlint-disable -->

# Hardening Report: agrc--reminder-action/v1.0.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **agrc--reminder-action/v1.0.21** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references across workflow files use mutable tags or version strings instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved or hijacked. Failing references include: `actions/checkout@v6`, `pnpm/action-setup@v6.0.5`, `actions/setup-node@v6`, `actions/create-github-app-token@v3.1.1`, `peter-evans/create-or-update-comment@v5` (pull_request.yml); `agrc/create-reminder-action@v1` (issue_comment.yml); `agrc/release-composite-action@v1` (push.yml); `agrc/reminder-action@v1` (scheduled.yml).

Locations:

- `.github/workflows/pull_request.yml:15`
- `.github/workflows/pull_request.yml:19`
- `.github/workflows/pull_request.yml:23`
- `.github/workflows/pull_request.yml:41`
- `.github/workflows/pull_request.yml:45`
- `.github/workflows/pull_request.yml:49`
- `.github/workflows/pull_request.yml:70`
- `.github/workflows/pull_request.yml:79`
- `.github/workflows/pull_request.yml:83`
- `.github/workflows/pull_request.yml:87`
- `.github/workflows/pull_request.yml:113`
- `.github/workflows/pull_request.yml:117`
- `.github/workflows/pull_request.yml:121`
- `.github/workflows/pull_request.yml:130`
- `.github/workflows/pull_request.yml:143`
- `.github/workflows/issue_comment.yml:17`
- `.github/workflows/push.yml:18`
- `.github/workflows/scheduled.yml:13`

### missing-permissions (severity: medium)

`.github/workflows/pull_request.yml` has no top-level `permissions:` key, and the `test-check` and `test-unit` jobs also have no job-level `permissions:` block. This means those jobs run with the default (broad) token permissions. The `build` and `integration-test` jobs do have job-level permissions, but the remaining jobs do not, satisfying the FAIL criterion.

Locations:

- `.github/workflows/pull_request.yml:1`

### script-injection (severity: high)

Sub-rule (a): The `run:` block in the 'Commit and push if needed' step of the `build` job directly interpolates `${{ secrets.UGRC_RELEASE_BOT_NAME }}` and `${{ secrets.UGRC_RELEASE_BOT_EMAIL }}` inside shell commands (`git config user.name` and `git config user.email`). Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk — the value is substituted into the shell command string before the shell parses it, allowing embedded shell metacharacters to be executed. These should be moved to `env:` variables and referenced as `"$ENV_VAR"` instead.

Locations:

- `.github/workflows/pull_request.yml:96`
- `.github/workflows/pull_request.yml:97`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across four workflow files:

1. **unpinned-uses** (pull_request.yml, issue_comment.yml, push.yml, scheduled.yml): Pinned all 8 action references to full 40-character SHA digests using `lookup_action_sha`. Original tags preserved as inline comments.

2. **missing-permissions** (pull_request.yml): Added `permissions: {}` at the workflow top level, and `permissions: { contents: read }` to the `test-check` and `test-unit` jobs (minimum needed for checkout). The `build` and `integration-test` jobs already had appropriate job-level permissions.

3. **script-injection** (pull_request.yml, build job): Moved `${{ secrets.UGRC_RELEASE_BOT_NAME }}` and `${{ secrets.UGRC_RELEASE_BOT_EMAIL }}` from inline `run:` shell commands into the step's `env:` block as `GIT_USER_NAME` and `GIT_USER_EMAIL`, then referenced them as `"$GIT_USER_NAME"` and `"$GIT_USER_EMAIL"` in the shell script.

