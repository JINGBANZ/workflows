# workflows

Centralized, reusable GitHub Actions workflows. This repo is **public**, so any repo (including
private ones) can call these via `workflow_call`.

## Available reusable workflows

| Workflow | Purpose |
|---|---|
| `.github/workflows/claude-code-review.yml` | Automatic Claude PR review — posts inline diff comments. The recommended default reviewer is [hosted Codex code review](#pr-review-hosted-codex-code-review) (no workflow needed); use this for repos that want the review in Actions instead. |
| `.github/workflows/claude.yml` | On-demand `@claude` mentions in issues/PRs. |
| `.github/workflows/sync-shared-rules.yml` | Syncs the shared-rules block in `AGENTS.md` from [JINGBANZ/rules](https://github.com/JINGBANZ/rules) — opens a PR on drift. |
| `.github/workflows/issue-opener.yml` | Scheduled agent that explores the repo and files one small, actionable issue. |
| `.github/workflows/issue-worker.yml` | When an issue is opened by a write-access author, an agent implements it and opens a PR. |
| `.github/workflows/web-dogfood.yml` | Scheduled agent that drives your live website in a real browser (agent-browser `dogfood` skill), hunts bugs/UX issues, proposes growth improvements, and files one detailed report issue. |

The opener and worker chain with the PR review into a fully autonomous pipeline:
**opener files issue → worker opens PR → review comments** — every hand-off machine-to-machine.
Both take an `engine` input (default `claude`) so the agent can be swapped inside this hub without
touching callers. The worker's trust gate (issue author must have write access) also lives here,
engine-independent. The web-dogfood auditor files its report with the default `GITHUB_TOKEN`,
which never triggers other workflows, so its reports never reach the worker.

## PR review: hosted Codex code review

The recommended reviewer is [Codex code review](https://developers.openai.com/codex/cloud/code-review),
OpenAI's hosted GitHub integration — zero workflow code, runs on the ChatGPT subscription (no API
key), and posts inline comments plus a summary like a human reviewer. It reviews the worker's PRs
too (they're authored by your account via `BOT_PAT`).

Setup, per repo:

1. Set up [Codex cloud](https://chatgpt.com/codex) for the repository (installs the Codex GitHub app).
2. In [Codex code review settings](https://chatgpt.com/codex/settings/code-review), toggle
   **Code review** on for the repo and enable **Automatic reviews** so every new PR is reviewed
   without a mention. On-demand: comment `@codex review` (optionally with a focus, e.g.
   `@codex review for security regressions`); follow up with `@codex fix the P1 issue`.
3. Codex flags only P0/P1 issues by default. To keep reviews strict and multi-angle, add a
   `## Review guidelines` section to the repo's top-level `AGENTS.md` (candidate block below —
   it can also live in [JINGBANZ/rules](https://github.com/JINGBANZ/rules) so the shared-rules
   sync distributes it everywhere):

   ```markdown
   ## Review guidelines
   - Be strict and critical: actively look for reasons the change is not safe to merge, and
     verify against surrounding code, callers, and tests rather than judging hunks in isolation.
   - Cover all of: correctness (logic, edge cases, error handling), breaking changes (API
     contracts, callers, config/migrations), security (injection, authz, secrets), performance
     (complexity, IO in loops, memory), test coverage, and maintainability.
   - Flag every issue the author would fix if they knew about it — don't stop at the first few.
   - Only flag issues introduced by the change, not pre-existing ones.
   ```

The `claude-code-review.yml` reusable workflow remains for repos that prefer the review to run in
GitHub Actions with subscription-authenticated Claude (`CLAUDE_CODE_OAUTH_TOKEN`). Running both on
one repo double-reviews every PR — pick one.

## Quick start (in a target repo)

1. Open Claude Code in the target repository.
2. Run `/install-github-app` — this installs the [Claude GitHub App](https://github.com/apps/claude)
   and adds the `CLAUDE_CODE_OAUTH_TOKEN` secret to the repo.
3. For the issue-opener/issue-worker pipeline, also add a `BOT_PAT` secret: a fine-grained PAT with
   read/write on the repo's contents, issues, and pull requests. It exists because events created
   with the default `GITHUB_TOKEN` never trigger other workflows — the PAT is what lets
   issue → worker → PR → review chain automatically.
4. For PR reviews, enable [hosted Codex code review](#pr-review-hosted-codex-code-review) (no
   secret or workflow needed), or copy the `claude-code-review.yml` caller template instead.
5. Ask Claude to read this repo and copy the caller templates from [`templates/`](./templates) into
   the target repo's `.github/workflows/`.

## Caller templates

Ready-to-copy caller workflows live in [`templates/`](./templates):

- [`templates/claude-code-review.yml`](./templates/claude-code-review.yml) — automatic PR review
  in Actions (skip if the repo uses hosted Codex code review — don't run both).
- [`templates/claude.yml`](./templates/claude.yml) — `@claude` mentions.
- [`templates/sync-shared-rules.yml`](./templates/sync-shared-rules.yml) — weekly shared-rules sync
  (needs "Allow GitHub Actions to create and approve pull requests" enabled in the target repo).
- [`templates/issue-opener.yml`](./templates/issue-opener.yml) — daily issue-opening agent.
- [`templates/issue-worker.yml`](./templates/issue-worker.yml) — issue-to-PR agent.
- [`templates/web-dogfood.yml`](./templates/web-dogfood.yml) — weekly browser dogfood auditor
  (set your site's `url:` in the caller).

Drop one into the target repo's `.github/workflows/`. The caller owns the triggers; the logic stays
here, so updates roll out to every caller automatically. The Claude templates pass the
`CLAUDE_CODE_OAUTH_TOKEN` secret down (`claude.yml` also owns the `@claude` gate; the opener and
worker also need `BOT_PAT`); the web-dogfood and sync templates need only `CLAUDE_CODE_OAUTH_TOKEN`
and no secret respectively.
