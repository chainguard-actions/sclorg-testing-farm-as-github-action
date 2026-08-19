<!-- markdownlint-disable -->

# Hardening Report: sclorg--testing-farm-as-github-action/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sclorg--testing-farm-as-github-action/v4.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable version tags instead of pinned full-length SHA commit hashes. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Affected references include: actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3, stefanbuck/github-issue-parser@v3, redhat-plumbers-in-action/advanced-issue-labeler@v3, Actions-R-Us/actions-tagger@v2, actions-cool/check-user-permission@v2, release-drafter/release-drafter@v6, codecov/codecov-action@v5.

Locations:

- `.github/workflows/base-test-pr-trigger.yml:36`
- `.github/workflows/base-test-pr-trigger.yml:50`
- `.github/workflows/base-test.yml:16`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:27`
- `.github/workflows/check-dist.yml:46`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:26`
- `.github/workflows/codeql-analysis.yml:29`
- `.github/workflows/codeql-analysis.yml:32`
- `.github/workflows/issue-labeler.yml:14`
- `.github/workflows/issue-labeler.yml:17`
- `.github/workflows/issue-labeler.yml:22`
- `.github/workflows/lint.yml:16`
- `.github/workflows/lint.yml:19`
- `.github/workflows/publish-release.yml:16`
- `.github/workflows/publish-release.yml:19`
- `.github/workflows/release-drafter.yml:14`
- `.github/workflows/secrets_test.yml:16`
- `.github/workflows/tests.yml:18`
- `.github/workflows/tests.yml:21`
- `.github/workflows/tests.yml:33`
- `.github/workflows/timeout-test.yml:16`
- `.github/workflows/variables_test.yml:16`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be broad). Affected files: base-test.yml, secrets_test.yml, timeout-test.yml, variables_test.yml.

Locations:

- `.github/workflows/base-test.yml:1`
- `.github/workflows/secrets_test.yml:1`
- `.github/workflows/timeout-test.yml:1`
- `.github/workflows/variables_test.yml:1`

### script-injection (severity: high)

Direct GitHub Actions expression interpolation inside `run:` shell command strings (sub-rule a). These expressions are substituted into the shell script before execution, allowing an attacker to inject arbitrary shell commands if the expression value is attacker-controlled.

1. `.github/workflows/base-test-pr-trigger.yml`: The 'Check User Permission' step interpolates `${{ github.triggering_actor }}`, `${{ steps.checkAccess.outputs.user-permission }}`, and `${{ github.actor }}` directly inside `echo` commands in a `run:` block. The `github.triggering_actor` and `github.actor` values can be attacker-influenced usernames.

2. `.github/workflows/timeout-test.yml`: The 'Check if testing farm test reached past the timeout limit' step interpolates `${{ steps.tf_results.outputs.test_log_url }}` directly into a `curl` command, and the 'Check if timeout was reached' step interpolates `${{ steps.tf_results.conclusion }}` directly into an `if` condition — both inside `run:` blocks. These step output values flow through YAML template substitution before the shell parses them.

Locations:

- `.github/workflows/base-test-pr-trigger.yml:44`
- `.github/workflows/timeout-test.yml:35`
- `.github/workflows/timeout-test.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 11 workflow files:

**unpinned-uses**: Pinned all 12 action references to full commit SHAs:
- actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
- actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
- actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02
- github/codeql-action/init@v3 → @b7351df727350dca84cb9d725d57dcf5bc82ba26
- github/codeql-action/autobuild@v3 → @b7351df727350dca84cb9d725d57dcf5bc82ba26
- github/codeql-action/analyze@v3 → @b7351df727350dca84cb9d725d57dcf5bc82ba26
- stefanbuck/github-issue-parser@v3 → @cb6e97157cbf851e3a393ff8d57c93a484cc323f
- redhat-plumbers-in-action/advanced-issue-labeler@v3 → @b80ae64e3e156e9c111b075bfa04b295d54e8e2e
- Actions-R-Us/actions-tagger@v2 → @330ddfac760021349fef7ff62b372f2f691c20fb
- actions-cool/check-user-permission@v2 → @c21884f3dda18dafc2f8b402fe807ccc9ec1aa5e
- release-drafter/release-drafter@v6 → @6a93d829887aa2e0748befe2e808c66c0ec6e4c7
- codecov/codecov-action@v5 → @0fb7174895f61a3b6b78fc075e0cd60383518dac

**missing-permissions**: Added `permissions: contents: read` top-level block to base-test.yml, secrets_test.yml, timeout-test.yml, and variables_test.yml.

**script-injection**: 
- base-test-pr-trigger.yml: Moved github.triggering_actor, steps.checkAccess.outputs.user-permission, and github.actor into an env: block; referenced as $TRIGGERING_ACTOR, $USER_PERMISSION, $ACTOR in the run: script.
- timeout-test.yml: Moved steps.tf_results.outputs.test_log_url into env: as TEST_LOG_URL for the curl command; moved steps.tf_results.conclusion into env: as TF_CONCLUSION for the if condition check.

