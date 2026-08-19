<!-- markdownlint-disable -->

# Hardening Report: sclorg--testing-farm-as-github-action/v3.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sclorg--testing-farm-as-github-action/v3.1.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across workflow files use mutable tag-based refs instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if a tag is moved. Failing references include: actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3, stefanbuck/github-issue-parser@v3, redhat-plumbers-in-action/advanced-issue-labeler@v3, Actions-R-Us/actions-tagger@v2, release-drafter/release-drafter@v6, codecov/codecov-action@v4.

Locations:

- `.github/workflows/base-test.yml:17`
- `.github/workflows/check-dist.yml:23`
- `.github/workflows/check-dist.yml:26`
- `.github/workflows/check-dist.yml:47`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:26`
- `.github/workflows/codeql-analysis.yml:29`
- `.github/workflows/codeql-analysis.yml:32`
- `.github/workflows/issue-labeler.yml:13`
- `.github/workflows/issue-labeler.yml:16`
- `.github/workflows/issue-labeler.yml:21`
- `.github/workflows/lint.yml:16`
- `.github/workflows/lint.yml:19`
- `.github/workflows/publish-release.yml:16`
- `.github/workflows/publish-release.yml:19`
- `.github/workflows/release-drafter.yml:15`
- `.github/workflows/secrets_test.yml:17`
- `.github/workflows/tests.yml:16`
- `.github/workflows/tests.yml:19`
- `.github/workflows/tests.yml:31`
- `.github/workflows/timeout-test.yml:17`
- `.github/workflows/variables_test.yml:17`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. Files: base-test.yml, secrets_test.yml, timeout-test.yml, variables_test.yml.

Locations:

- `.github/workflows/base-test.yml:1`
- `.github/workflows/secrets_test.yml:1`
- `.github/workflows/timeout-test.yml:1`
- `.github/workflows/variables_test.yml:1`

### script-injection (severity: high)

Two `run:` blocks in timeout-test.yml directly interpolate GitHub Actions expressions into shell commands (sub-rule a). Line 32: `curl ${{ steps.tf_results.outputs.test_log_url }} > results.log` — the step output URL is injected directly into a curl command without quoting or sanitization. Line 40: `if [[ ${{ steps.tf_results.conclusion }} == 'success' ]]; then` — the step conclusion value is injected directly into a shell conditional. Both `steps.*.outputs.*` values are workflow-controllable and can contain shell metacharacters.

Locations:

- `.github/workflows/timeout-test.yml:32`
- `.github/workflows/timeout-test.yml:40`

### hardcoded-credentials (severity: high)

In secrets_test.yml, the `secrets:` input is assigned a literal string value containing credential-like key=value pairs: `"SOME_DUMMY_SECRET=FOO_BAR_SECRET;OS_SECRET=DUMMY_SECRET_OS"`. These are literal values (not GitHub Actions secret expressions) assigned to a field whose name contains 'secret', matching the hardcoded-credentials pattern. Even if intended as dummy values, hardcoding them in a workflow file is a security anti-pattern.

Locations:

- `.github/workflows/secrets_test.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, hardcoded-credentials

**Notes:**

Fixed all 4 findings across 11 workflow files:

1. unpinned-uses: Pinned all 11 action references to full 40-char SHA hashes with tag comments preserved. Actions pinned: actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4, github/codeql-action/{init,autobuild,analyze}@v3, stefanbuck/github-issue-parser@v3, redhat-plumbers-in-action/advanced-issue-labeler@v3, Actions-R-Us/actions-tagger@v2, release-drafter/release-drafter@v6, codecov/codecov-action@v4.

2. missing-permissions: Added top-level 'permissions: contents: read' to base-test.yml, secrets_test.yml, timeout-test.yml, and variables_test.yml.

3. script-injection: In timeout-test.yml, moved ${{ steps.tf_results.outputs.test_log_url }} and ${{ steps.tf_results.conclusion }} out of run: blocks into env: blocks (TEST_LOG_URL and TF_CONCLUSION), then referenced as plain env vars in the shell scripts.

4. hardcoded-credentials: In secrets_test.yml, replaced the literal string 'SOME_DUMMY_SECRET=FOO_BAR_SECRET;OS_SECRET=DUMMY_SECRET_OS' with ${{ secrets.TF_DUMMY_SECRETS }} to use a proper GitHub Actions secret reference.

