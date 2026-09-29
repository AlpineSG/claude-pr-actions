# claude-pr-actions

Reusable GitHub Actions workflows that put Claude to work on your pull requests:

| Workflow | What it does |
| --- | --- |
| [`claude-complexity-grade.yml`](./.github/workflows/claude-complexity-grade.yml) | Grades each PR's **risk** (LOW / MEDIUM / HIGH) with a structured comment and a `complexity-*` label. Can optionally approve low-risk PRs as your GitHub App. |
| [`claude-comment-addressing.yml`](./.github/workflows/claude-comment-addressing.yml) | Responds to **review feedback** from humans and from an AI review bot (CodeRabbit by default). Makes the change or replies, resolves the thread, and pushes. |
| [`claude-test-remediation.yml`](./.github/workflows/claude-test-remediation.yml) | When **CI fails** on a PR, Claude diagnoses it, applies a minimal fix, checks it locally and pushes. Otherwise it escalates to a human. |
| [`pr-restack.yml`](./.github/workflows/pr-restack.yml) | When a PR in a **stack** merges, retargets its children to the new base and rebases away the parent's commits (squash-merge safe). No Claude, apart from dispatching a sweep if a rebase conflicts. |

Each repo keeps a short caller workflow that owns the triggers and permissions, and calls these with `secrets: inherit`. Improvements here reach every caller at the next `@v1` tag.

## Quickstart

1. **Create a GitHub App** for your organization and add three secrets (`CLAUDE_APP_ID`, `CLAUDE_APP_PRIVATE_KEY`, `ANTHROPIC_API_KEY`). See [docs/github-app-setup.md](./docs/github-app-setup.md).
2. **Copy the caller workflows** you want from [`examples/`](./examples/) into your repo's `.github/workflows/`. Change `workflows: ['CI']` to your CI workflow's name.
3. *(Recommended)* **Give Claude repo context.** Add `.github/claude-prompts/<workflow>.md` files on your default branch listing sensitive paths, house rules and files that must not be edited. See [`examples/prompts/`](./examples/prompts/). Claude also reads `CLAUDE.md` / `AGENTS.md` if you have them.

## How the prompts work

Each Claude workflow ships a generic **base prompt** ([`prompts/`](./prompts/)) that covers the mechanics: what to post, how to label, how to resolve threads, what never to touch, and when to escalate.

Your repo can add to it with `.github/claude-prompts/<name>.md` (`complexity-grade`, `comment-addressing`, `test-remediation`):

| `prompt-mode` | Result |
| --- | --- |
| `append` (default) | Base prompt, then your file under "Repository-specific context". Your file is optional. |
| `replace` | Your file on its own, which must exist. Use this if you've outgrown the base prompt. |

Your file is always read from the **default branch**, never from the PR head. A PR can't rewrite the instructions it is graded or fixed under, and older branches that don't have the file still work.

## Inputs

### Shared by the three Claude workflows

| Input | Default | Notes |
| --- | --- | --- |
| `model` | `claude-sonnet-5` (grade), `claude-opus-5-5` (others) | Any Claude model ID |
| `max-turns` | `20` / `24` / `32` | |
| `extra-allowed-tools` | `''` | Comma-separated extra allow-list entries, e.g. `Bash(make build:*)` |
| `prompt-mode` | `append` | See above |
| `telemetry` | `false` | Export Claude Code OpenTelemetry (cost, tokens, tool use). Needs `otel-endpoint`. |
| `otel-endpoint` | `''` | OTLP/HTTP endpoint. Auth headers go in the `OTEL_EXPORTER_OTLP_HEADERS` secret. |

### `claude-complexity-grade.yml`

| Input | Default | Notes |
| --- | --- | --- |
| `head-sha` | **required** | The SHA the triggering event ran against. A stale event exits. |
| `pr-number` | `''` | Worked out from `head-sha` if empty |
| `can-approve` | `false` | Approve LOW/MEDIUM PRs as the App, and dismiss that approval on HIGH |

### `claude-comment-addressing.yml` and `claude-test-remediation.yml`

| Input | Default | Notes |
| --- | --- | --- |
| `package-manager` | `pnpm` | `pnpm` / `npm` / `yarn` / `none` |
| `node-version` | `24` | |
| `install-command` | set by the package manager | e.g. `npm ci` |
| `verify-commands` | lint, type-check and test for the package manager | Comma-separated. Each becomes a `Bash(<cmd>:*)` allow-list entry and is shown to Claude. |
| `registry-url` / `registry-scope` | `''` | Private npm registry (e.g. GitHub Packages). The token comes from the `NPM_REGISTRY_TOKEN` secret, or `GITHUB_TOKEN` if that isn't set. |

Comment addressing only:

| Input | Default | Notes |
| --- | --- | --- |
| `pr-number` | `''` | Pass `inputs.pr_number` for `workflow_dispatch` sweeps |
| `install` | `'false'` | Install before Claude starts. Off by default because Claude can install when it needs to. |
| `review-bot-login` | `coderabbitai[bot]` | Review submissions from this bot trigger round-capped runs. `''` turns this off. |
| `bot-round-cap` | `3` | |
| `push-workflow-changes` | `false` | Request `workflows: write` so merging a base that changed `.github/workflows/**` can be pushed. The App needs the Workflows permission. |

Test remediation only:

| Input | Default | Notes |
| --- | --- | --- |
| `standard-base-refs` | `main\|master\|develop\|release/*\|hotfix/*` | PRs on any other base count as stacked and are skipped |
| `attempt-cap` | `3` | |
| `ci-log-tail-lines` | `2000` | |
| `postgres-image` | `''` | e.g. `postgres:17`. Starts a service container. |
| `postgres-user` / `postgres-password` / `postgres-db` | `postgres` / `postgres` / `app_test` | Throwaway CI credentials |
| `database-url` | `''` | Exported as `DATABASE_URL` |

### `pr-restack.yml`

| Input | Default | Notes |
| --- | --- | --- |
| `dispatch-ref` | **required** | The branch carrying the comment-addressing caller that sweeps are dispatched against |
| `sweep-workflow` | `claude-comment-addressing.yml` | |
| `push-workflow-changes` | `false` | Same as for comment addressing: needed to force-push children whose commits change workflows |

### Secrets

| Secret | Required | Used by |
| --- | --- | --- |
| `CLAUDE_APP_ID`, `CLAUDE_APP_PRIVATE_KEY` | yes | all |
| `ANTHROPIC_API_KEY` **or** `CLAUDE_CODE_OAUTH_TOKEN` | one of them | the Claude workflows |
| `NPM_REGISTRY_TOKEN` | no | private registries |
| `OTEL_EXPORTER_OTLP_HEADERS` | no | telemetry |

## Labels

The workflows create and read these labels (the grader creates missing labels itself).

| Label | Meaning |
| --- | --- |
| `claude-skip` | Opts the PR out of **all** of these workflows, including restacking |
| `complexity-low` / `-medium` / `-high` | The grader's result |
| `needs-human-review` | Added along with `complexity-high` |
| `claude-review-round-N`, `claude-review-max` | Comment-addressing bot-round counter and cap. Remove `claude-review-max` to turn bot-triggered runs back on. |
| `claude-attempt-N`, `claude-attempt-max` | Test-remediation attempt counter and cap |

Workflow comments carry hidden markers (`<!-- claude:complexity-grade -->`, `<!-- claude:test-remediation -->` and so on), so later runs update them in place instead of piling up.

## Security model

Claude runs with `--permission-mode bypassPermissions`, **limited by an explicit tool allow-list** for each workflow. The grader can't edit files, and nothing is allowed to merge or publish. On top of that:

- **App tokens are scoped per job**: the grader's token can read contents and write only PRs and issues, and none of the tokens gets `workflows: write` unless you opt in.
- **Fork PRs are skipped** by every workflow that checks code out with write credentials.
- **Comment addressing checks the author's write access** before a human-triggered run checks out the PR head.
- **Event data goes through env vars** and is never interpolated into shell.
- **Loops are bounded**: bot-triggered comment runs are capped by round, remediation by attempt, and the App's own events are excluded from triggering comment addressing.
- **Prompts come from the default branch**, so a PR can't change its own instructions.
- **The grader checks out with `persist-credentials: false`** and never runs PR code.
- The base prompts forbid editing `.github/workflows/**`, `.github/claude-prompts/**`, lockfiles, build config and top-level docs. Your repo context can add to that list.
- All third-party actions are pinned to a commit SHA, and Dependabot bumps them weekly.

This repo's own CI runs `actionlint` and `zizmor` on every change. To report a vulnerability, see [SECURITY.md](./SECURITY.md).

## Cost controls

- Comment addressing **waits 60 s** so a burst of comments becomes one run (newer runs cancel older ones through the caller's concurrency group).
- Remediation skips **stacked PRs**. Their failures show up once the stack is retargeted.
- The grader exits early for drafts, closed PRs, `claude-skip`, and stale SHAs.
- Round and attempt caps put a hard limit on automated spend per PR.
- Turn on `telemetry` to see actual cost per workflow and repo in your observability stack.

## Versioning

Callers pin `@v1`, a moving major tag. Every release also gets an immutable `v1.x.y` tag for callers who want to pin exactly. Breaking changes (renamed or removed inputs, or changed defaults that affect behaviour) get a new major version.

The reusable workflows load their base prompts through `AlpineSG/claude-pr-actions/load-prompt@v1`. The prompts therefore follow the **latest `v1`**, even if you pin a workflow to `v1.x.y`.

## License

[MIT](./LICENSE)
