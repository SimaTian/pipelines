---
name: pr-build-status
description: "Get PR build status from GitHub Actions CI. Use when asked about CI status, build results, test failures, or pipeline status for a pull request."
---

# PR Build Status

Retrieve and summarize CI build status for a pull request.

## Usage

1. **Detect PR**: auto-detect from current branch or accept PR number as input
2. **Fetch check runs**: use `gh pr checks <number>` or GitHub API
3. **Summarize**:
   - Overall status (pass/fail/in-progress)
   - Failed jobs with error summaries
   - Link to full logs

## Commands

```bash
# Get all check statuses for a PR
gh pr checks <PR_NUMBER>

# Get detailed view
gh pr checks <PR_NUMBER> --watch

# Get failed checks only
gh pr checks <PR_NUMBER> --fail-only

# View workflow run logs
gh run view <RUN_ID> --log-failed
```

## Interpretation

- **All checks pass**: PR is ready for review
- **Some checks fail**: investigate failed jobs
  - Build failures → likely code issue
  - Test failures → check test output for assertion details
  - Lint/format failures → run `dotnet format` locally
- **Checks in progress**: wait or check partial results

## Aggregation Rules

When multiple jobs exist for the same workflow:
- If ANY job is `failure` → overall is `failure`
- If ANY job is `in_progress` and none failed → `in_progress`
- Only if ALL jobs `success` → `success`

Do NOT pick an arbitrary job and report its status as the overall status.
