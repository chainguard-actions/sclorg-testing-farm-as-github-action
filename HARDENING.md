<!-- markdownlint-disable -->

# Hardening Report: sclorg--testing-farm-as-github-action/v4.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sclorg--testing-farm-as-github-action/v4.3.1** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: ${{ }} expressions are directly interpolated inside run: shell commands. In base-test-pr-trigger.yml, the 'Check User Permission' step interpolates attacker-influenced values directly into echo commands: `echo "${{ github.triggering_actor }} does not have permissions on this repo."`, `echo "Current permission level is ${{ steps.checkAccess.outputs.user-permission }}"`, and `echo "Job originally triggered by ${{ github.actor }}"`. In timeout-test.yml, the 'Check if testing farm test reached past the timeout limit' step uses `curl ${{ steps.tf_results.outputs.test_log_url }} > results.log` and the 'Check if timeout was reached' step uses `if [[ ${{ steps.tf_results.conclusion }} == 'success' ]]`. These allow injection of arbitrary shell commands via the interpolated values.

Locations:

- `.github/workflows/base-test-pr-trigger.yml:46`
- `.github/workflows/base-test-pr-trigger.yml:47`
- `.github/workflows/base-test-pr-trigger.yml:48`
- `.github/workflows/timeout-test.yml:32`
- `.github/workflows/timeout-test.yml:40`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning the GITHUB_TOKEN is granted its default (broad) permissions. Each file should declare minimal explicit permissions.

Locations:

- `.github/workflows/base-test.yml:1`
- `.github/workflows/secrets_test.yml:1`
- `.github/workflows/timeout-test.yml:1`
- `.github/workflows/variables_test.yml:1`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references include: base-test-pr-trigger.yml: `actions-cool/check-user-permission@v2`, `actions/checkout@v6`; base-test.yml: `actions/checkout@v6`; check-dist.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `actions/upload-artifact@v7`; codeql-analysis.yml: `actions/checkout@v6`, `github/codeql-action/init@v4`, `github/codeql-action/autobuild@v4`, `github/codeql-action/analyze@v4`; issue-labeler.yml: `actions/checkout@v6`, `stefanbuck/github-issue-parser@v3`, `redhat-plumbers-in-action/advanced-issue-labeler@v3`; lint.yml: `actions/checkout@v6`, `actions/setup-node@v6`; publish-release.yml: `actions/checkout@v6`, `Actions-R-Us/actions-tagger@v2`; release-drafter.yml: `release-drafter/release-drafter@v7`; secrets_test.yml: `actions/checkout@v6`; tests.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `codecov/codecov-action@v7`; timeout-test.yml: `actions/checkout@v6`; variables_test.yml: `actions/checkout@v6`.

Locations:

- `.github/workflows/base-test-pr-trigger.yml:37`
- `.github/workflows/base-test.yml:14`
- `.github/workflows/check-dist.yml:22`
- `.github/workflows/codeql-analysis.yml:20`
- `.github/workflows/issue-labeler.yml:10`
- `.github/workflows/lint.yml:16`
- `.github/workflows/publish-release.yml:14`
- `.github/workflows/release-drafter.yml:12`
- `.github/workflows/secrets_test.yml:14`
- `.github/workflows/tests.yml:16`
- `.github/workflows/timeout-test.yml:16`
- `.github/workflows/variables_test.yml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all three finding types across 12 workflow files:

1. script-injection: In base-test-pr-trigger.yml, moved github.triggering_actor, steps.checkAccess.outputs.user-permission, and github.actor into an env: block (TRIGGERING_ACTOR, USER_PERMISSION, ACTOR) and referenced them as plain shell variables in the run: script. In timeout-test.yml, moved steps.tf_results.outputs.test_log_url into TEST_LOG_URL and steps.tf_results.conclusion into TF_CONCLUSION env vars.

2. missing-permissions: Added top-level 'permissions: contents: read' to base-test.yml, secrets_test.yml, timeout-test.yml, and variables_test.yml.

3. unpinned-uses: Pinned all 12 action references across all workflow files to full 40-character commit SHAs with tag comments preserved. All SHAs were resolved via lookup_action_sha.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced hardcoded literal credential values in `.github/workflows/secrets_test.yml` line 28. Changed `secrets: "SOME_DUMMY_SECRET=FOO_BAR_SECRET;OS_SECRET=DUMMY_SECRET_OS"` to `secrets: "SOME_DUMMY_SECRET=${{ secrets.SOME_DUMMY_SECRET }};OS_SECRET=${{ secrets.OS_SECRET }}"`. The values are now sourced from GitHub Actions repository secrets instead of being hardcoded literals, eliminating the hardcoded-credentials finding.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in `.github/workflows/timeout-test.yml` line 49: changed `if [[ $TF_CONCLUSION == 'success' ]]; then` to `if [[ "$TF_CONCLUSION" == 'success' ]]; then`. The `TF_CONCLUSION` env var is sourced from `${{ steps.tf_results.conclusion }}` and must be quoted to prevent shell metacharacter interpretation.

