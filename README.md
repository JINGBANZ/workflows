# workflows

Centralized, reusable GitHub Actions workflows. This repo is **public**, so any repo (including
private ones) can call these via `workflow_call`.

## Available reusable workflows

| Workflow | Purpose |
|---|---|
| `.github/workflows/claude-code-review.yml` | Automatic Claude PR review — posts inline diff comments. |
| `.github/workflows/claude.yml` | On-demand `@claude` mentions in issues/PRs. |

Each has only an `on: workflow_call` trigger, so they never run here — they run in the repo that
calls them.

## Using them from another repo

Add a thin caller workflow to the target repo's `.github/workflows/`. The caller owns the triggers
and passes the `CLAUDE_CODE_OAUTH_TOKEN` secret (stored in the target repo). Install the
[Claude GitHub App](https://github.com/apps/claude) on the target repo first.

**Automatic PR review** — `.github/workflows/claude-code-review.yml`:

```yaml
name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]
jobs:
  review:
    uses: JINGBANZ/workflows/.github/workflows/claude-code-review.yml@main
    permissions:
      contents: read
      pull-requests: read
      issues: read
      id-token: write
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    # Optional: override the driver model (default: claude-sonnet-4-6)
    # with:
    #   model: claude-opus-4-7
```

**`@claude` mentions** — `.github/workflows/claude.yml`:

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]
  pull_request_review:
    types: [submitted]
jobs:
  claude:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review' && contains(github.event.review.body, '@claude')) ||
      (github.event_name == 'issues' && (contains(github.event.issue.body, '@claude') || contains(github.event.issue.title, '@claude')))
    uses: JINGBANZ/workflows/.github/workflows/claude.yml@main
    permissions:
      contents: read
      pull-requests: read
      issues: read
      id-token: write
      actions: read
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

> Pin `@main` to a tag or commit SHA if you want changes to roll out only when you bump the ref.
