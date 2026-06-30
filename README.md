# workflows

Centralized, reusable GitHub Actions workflows. This repo is **public**, so any repo (including
private ones) can call these via `workflow_call`.

## Available reusable workflows

| Workflow | Purpose |
|---|---|
| `.github/workflows/claude-code-review.yml` | Automatic Claude PR review — posts inline diff comments. |
| `.github/workflows/claude.yml` | On-demand `@claude` mentions in issues/PRs. |

## Quick start (in a target repo)

1. Open Claude Code in the target repository.
2. Run `/install-github-app` — this installs the [Claude GitHub App](https://github.com/apps/claude)
   and adds the `CLAUDE_CODE_OAUTH_TOKEN` secret to the repo.
3. Ask Claude to read this repo and copy the caller templates from [`templates/`](./templates) into
   the target repo's `.github/workflows/`.

## Caller templates

Ready-to-copy caller workflows live in [`templates/`](./templates):

- [`templates/claude-code-review.yml`](./templates/claude-code-review.yml) — automatic PR review.
- [`templates/claude.yml`](./templates/claude.yml) — `@claude` mentions.

Drop one into the target repo's `.github/workflows/`. The caller owns the triggers (and the
`@claude` gate) and passes the `CLAUDE_CODE_OAUTH_TOKEN` secret down; the review logic stays here, so
updates roll out to every caller automatically.
