<!--
Example repo context for test remediation.
Copy to .github/claude-prompts/test-remediation.md in your repo and edit it.
-->

### CI jobs and how to reproduce them

| CI job | Local command |
| --- | --- |
| Lint | `pnpm lint` |
| Type Check | `pnpm type-check` |
| Unit Tests | `pnpm test` (Vitest; the API tests use `$DATABASE_URL`) |

### Escalate instead of fixing

- A failure in `packages/api-contract/**` tests: the contract is shared with another team.
- A migration test failure: schema changes need a human.
