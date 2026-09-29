# GitHub App setup

Each organization that uses these workflows creates **its own** GitHub App. The workflows mint a short-lived installation token from it on every run.

## Why an App, and not `GITHUB_TOKEN`?

- **Pushes made with `GITHUB_TOKEN` don't trigger workflows.** GitHub blocks that to prevent recursive runs. When Claude pushes a fix or restacks a PR, CI has to run again on the new head, or branch protection never sees a green check. App-token pushes trigger CI normally.
- **Approvals count.** A review from the App counts toward required approvals, because the App is never the PR author (`can-approve` on the grader).
- **Clear authorship.** Automated commits, comments and labels all show up as `<your-app>[bot]`.

## 1. Create the App

**Organization settings → Developer settings → GitHub Apps → New GitHub App**

- **Name**: anything, e.g. `acme-claude-automations`. The workflows get the bot login and commit email from the App at runtime, so nothing is hardcoded.
- **Homepage URL**: your org or repo URL.
- **Webhook**: untick **Active**. The App receives no events itself.
- **Repository permissions**:

  | Permission | Access | Used by |
  | --- | --- | --- |
  | Contents | Read & write | pushing fixes and restacks |
  | Pull requests | Read & write | comments, labels, retargeting, reviews |
  | Issues | Read & write | labels (GitHub stores PR labels as issue labels) |
  | Actions | Read & write | reading failed CI logs, dispatching restack sweeps |
  | Checks | Read | reading CI status |
  | Metadata | Read | required |
  | Workflows | Read & write | *optional*: only needed with `push-workflow-changes: true`, i.e. pushing merges or rebases whose commits change `.github/workflows/**` |

- **Where can this App be installed?** Only on this account.

[`github-app-manifest.json`](./github-app-manifest.json) has the same settings, if you would rather create the App from a manifest.

## 2. Generate a private key and install the App

1. On the App's page, **Generate a private key** and download the `.pem`.
2. **Install App** → choose your org → pick the repositories that will use the workflows.

Each workflow asks for only the subset of these permissions it needs when it mints its token.

## 3. Add the secrets

Add these as **organization** Actions secrets (visible to the repos above), or per repository:

| Secret | Value |
| --- | --- |
| `CLAUDE_APP_ID` | The App ID from the App's settings page |
| `CLAUDE_APP_PRIVATE_KEY` | The full contents of the `.pem` file |
| `ANTHROPIC_API_KEY` | An Anthropic API key. Or set `CLAUDE_CODE_OAUTH_TOKEN` (from `claude setup-token`) to bill a Claude subscription instead. |

Callers pass these with `secrets: inherit`.

## 4. Branch protection (recommended)

- Keep CI as a **required status check**. The grader's approval never replaces CI.
- If you use `can-approve`, the App's approval satisfies one required approval. Turn on **Dismiss stale approvals** if you want every push to need a fresh grade (the grader re-approves the new head automatically if it is still LOW/MEDIUM).
