# workflows

Centralized, reusable GitHub Actions workflows. This repo is **public**, so any repo (including
private ones) can call these via `workflow_call`.

## Available reusable workflows

| Workflow | Purpose |
|---|---|
| `.github/workflows/claude-code-review.yml` | Automatic Claude PR review — posts inline diff comments. |
| `.github/workflows/claude.yml` | On-demand `@claude` mentions in issues/PRs. |
| `.github/workflows/sync-shared-rules.yml` | Syncs the shared-rules block in `AGENTS.md` from [JINGBANZ/rules](https://github.com/JINGBANZ/rules) — opens a PR on drift. |

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
- [`templates/sync-shared-rules.yml`](./templates/sync-shared-rules.yml) — weekly shared-rules sync
  (needs "Allow GitHub Actions to create and approve pull requests" enabled in the target repo).

Drop one into the target repo's `.github/workflows/`. The caller owns the triggers; the logic stays
here, so updates roll out to every caller automatically. The two Claude templates also own the
`@claude` gate and pass the `CLAUDE_CODE_OAUTH_TOKEN` secret down; the sync template needs no secret.
