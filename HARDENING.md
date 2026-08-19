<!-- markdownlint-disable -->

# Hardening Report: sclorg--testing-farm-as-github-action/v4.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sclorg--testing-farm-as-github-action/v4.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files is pinned to a mutable tag or version string rather than a full 40-character commit SHA. This exposes the workflows to supply-chain attacks if any referenced action is compromised or its tag is moved. Affected references include: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v7, actions-cool/check-user-permission@v2, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, stefanbuck/github-issue-parser@v3, redhat-plumbers-in-action/advanced-issue-labeler@v3, Actions-R-Us/actions-tagger@v2, release-drafter/release-drafter@v7, codecov/codecov-action@v7.

Locations:

- `.github/workflows/base-test-pr-trigger.yml:37`
- `.github/workflows/base-test.yml:16`
- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:25`
- `.github/workflows/check-dist.yml:44`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:25`
- `.github/workflows/codeql-analysis.yml:30`
- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/issue-labeler.yml:13`
- `.github/workflows/issue-labeler.yml:17`
- `.github/workflows/issue-labeler.yml:23`
- `.github/workflows/lint.yml:16`
- `.github/workflows/lint.yml:19`
- `.github/workflows/publish-release.yml:16`
- `.github/workflows/publish-release.yml:19`
- `.github/workflows/release-drafter.yml:16`
- `.github/workflows/secrets_test.yml:16`
- `.github/workflows/tests.yml:16`
- `.github/workflows/tests.yml:19`
- `.github/workflows/tests.yml:28`
- `.github/workflows/timeout-test.yml:16`
- `.github/workflows/variables_test.yml:16`

### script-injection (severity: high)

Rule (a) violation: GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing an attacker to inject arbitrary shell commands. In base-test-pr-trigger.yml, the 'Check User Permission' step echoes `${{ github.triggering_actor }}`, `${{ steps.checkAccess.outputs.user-permission }}`, and `${{ github.actor }}` directly into shell commands. In timeout-test.yml, `${{ steps.tf_results.outputs.test_log_url }}` is interpolated directly into a `curl` command and `${{ steps.tf_results.conclusion }}` is interpolated directly into a bash `if` condition — both are step outputs that could be attacker-influenced.

Locations:

- `.github/workflows/base-test-pr-trigger.yml:46`
- `.github/workflows/base-test-pr-trigger.yml:47`
- `.github/workflows/base-test-pr-trigger.yml:48`
- `.github/workflows/timeout-test.yml:32`
- `.github/workflows/timeout-test.yml:41`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be `write-all` for older repositories), granting unnecessarily broad access. The affected files are: base-test.yml, secrets_test.yml, timeout-test.yml, and variables_test.yml.

Locations:

- `.github/workflows/base-test.yml:1`
- `.github/workflows/secrets_test.yml:1`
- `.github/workflows/timeout-test.yml:1`
- `.github/workflows/variables_test.yml:1`

### hardcoded-credentials (severity: high)

secrets_test.yml contains hardcoded literal secret values in the `secrets:` input field: `"SOME_DUMMY_SECRET=FOO_BAR_SECRET;OS_SECRET=DUMMY_SECRET_OS"`. The names contain 'secret' and the values are non-expression literals (not `${{ secrets.* }}` references). Even if these are intended as test/dummy values, hardcoding them in a workflow file is a security anti-pattern and matches the hardcoded-credentials check pattern.

Locations:

- `.github/workflows/secrets_test.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all four findings across 10 workflow files:

1. unpinned-uses: Pinned all 12 action references to full 40-char commit SHAs with original tag preserved as comment. Files updated: base-test-pr-trigger.yml, base-test.yml, check-dist.yml, codeql-analysis.yml, issue-labeler.yml, lint.yml, publish-release.yml, release-drafter.yml, secrets_test.yml, tests.yml, timeout-test.yml, variables_test.yml.

2. script-injection: In base-test-pr-trigger.yml, moved github.triggering_actor, steps.checkAccess.outputs.user-permission, and github.actor from run: shell strings into env: block variables. In timeout-test.yml, moved steps.tf_results.outputs.test_log_url and steps.tf_results.conclusion into env: block variables referenced as $TEST_LOG_URL and $TF_CONCLUSION.

3. missing-permissions: Added top-level 'permissions: contents: read' to base-test.yml, secrets_test.yml, timeout-test.yml, and variables_test.yml.

4. hardcoded-credentials: Replaced hardcoded literal 'SOME_DUMMY_SECRET=FOO_BAR_SECRET;OS_SECRET=DUMMY_SECRET_OS' in secrets_test.yml with ${{ secrets.TF_SECRETS }} reference.

