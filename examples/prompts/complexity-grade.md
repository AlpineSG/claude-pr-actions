<!--
Example repo context for the complexity grader.
Copy to .github/claude-prompts/complexity-grade.md in your repo and edit it.
It is read from your DEFAULT branch and appended to the shared base prompt.
Keep it short and specific: sensitive paths, and what counts as High here.
-->

This is a TypeScript monorepo: a React web app (`apps/web`), a Node API (`apps/api`), and shared packages (`packages/*`).

### Sensitive paths (Tier 1)

- `apps/api/src/auth/**`: session and JWT handling. Any logic change is **High** under Auth.
- `apps/api/src/db/migrations/**`: schema migrations. **High** under Data integrity.
- `packages/api-contract/**`: the wire contract shared with mobile clients. A breaking change is **High** under External contracts. Additive changes are **Medium**.
- Every query in `apps/api` must be scoped by `org_id`. A query without that scope is **High** under Data integrity.

### Additional Tier 2 dimensions

- **Accessibility**: new interactive UI that doesn't use the design-system primitives in `packages/ui` is **Medium**.
- **Internationalization**: new user-facing strings that aren't in `packages/i18n` are **Medium**.
