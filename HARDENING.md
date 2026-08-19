<!-- markdownlint-disable -->

# Hardening Report: sclorg--testing-farm-as-github-action/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sclorg--testing-farm-as-github-action/v4.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable version tags instead of pinned 40-character SHA commit hashes for their `uses:` references. This exposes the action to supply-chain attacks where a compromised upstream action tag could execute malicious code. Affected references include: actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4, actions-cool/check-user-permission@v2, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3, stefanbuck/github-issue-parser@v3, redhat-plumbers-in-action/advanced-issue-labeler@v3, Actions-R-Us/actions-tagger@v2, release-drafter/release-drafter@v6, codecov/codecov-action@v5.

Locations:

- `.github/workflows/base-test-pr-trigger.yml:37`
- `.github/workflows/base-test-pr-trigger.yml:52`
- `.github/workflows/base-test.yml:17`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:27`
- `.github/workflows/check-dist.yml:46`
- `.github/workflows/codeql-analysis.yml:20`
- `.github/workflows/codeql-analysis.yml:23`
- `.github/workflows/codeql-analysis.yml:26`
- `.github/workflows/codeql-analysis.yml:29`
- `.github/workflows/issue-labeler.yml:14`
- `.github/workflows/issue-labeler.yml:17`
- `.github/workflows/issue-labeler.yml:22`
- `.github/workflows/lint.yml:16`
- `.github/workflows/lint.yml:19`
- `.github/workflows/publish-release.yml:16`
- `.github/workflows/publish-release.yml:19`
- `.github/workflows/release-drafter.yml:16`
- `.github/workflows/secrets_test.yml:17`
- `.github/workflows/tests.yml:16`
- `.github/workflows/tests.yml:19`
- `.github/workflows/tests.yml:33`
- `.github/workflows/timeout-test.yml:17`
- `.github/workflows/variables_test.yml:17`

### script-injection (severity: high)

GitHub Actions expressions (${{ ... }}) are interpolated directly inside run: shell command strings, enabling script injection. In base-test-pr-trigger.yml (lines 46-48), the run: block echoes ${{ github.triggering_actor }}, ${{ steps.checkAccess.outputs.user-permission }}, and ${{ github.actor }} directly into shell commands — an attacker controlling the triggering_actor name could inject shell metacharacters. In timeout-test.yml (line 30), `curl ${{ steps.tf_results.outputs.test_log_url }} > results.log` interpolates a step output URL directly into the shell command (sub-rule a). In timeout-test.yml (line 37), `if [[ ${{ steps.tf_results.conclusion }} == 'success' ]]` interpolates a step output directly into a shell conditional (sub-rule a). All these values should be moved to env: variables and then double-quoted in the shell script.

Locations:

- `.github/workflows/base-test-pr-trigger.yml:46`
- `.github/workflows/base-test-pr-trigger.yml:47`
- `.github/workflows/base-test-pr-trigger.yml:48`
- `.github/workflows/timeout-test.yml:30`
- `.github/workflows/timeout-test.yml:37`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be read/write for contents), violating the principle of least privilege. Affected files: base-test.yml, secrets_test.yml, timeout-test.yml, variables_test.yml.

Locations:

- `.github/workflows/base-test.yml:1`
- `.github/workflows/secrets_test.yml:1`
- `.github/workflows/timeout-test.yml:1`
- `.github/workflows/variables_test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all 24 unpinned action references across 10 workflow files by resolving each tag to its full 40-character SHA using lookup_action_sha. Fixed 5 script injection locations: in base-test-pr-trigger.yml moved github.triggering_actor, steps.checkAccess.outputs.user-permission, and github.actor into env: blocks; in timeout-test.yml moved steps.tf_results.outputs.test_log_url and steps.tf_results.conclusion into env: blocks. Added top-level 'permissions: contents: read' to base-test.yml, secrets_test.yml, timeout-test.yml, and variables_test.yml.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Fixed hardcoded secret values in .github/workflows/secrets_test.yml line 31. Replaced the literal plaintext values 'FOO_BAR_SECRET' and 'DUMMY_SECRET_OS' with GitHub Actions secrets expressions: ${{ secrets.SOME_DUMMY_SECRET }} and ${{ secrets.OS_SECRET }} respectively. The secrets should now be stored as repository secrets in GitHub and referenced via expressions rather than being committed as plaintext literals.

