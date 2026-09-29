<!--
Example repo context for comment addressing.
Copy to .github/claude-prompts/comment-addressing.md in your repo and edit it.
-->

### House rules

- UI must use components from `packages/ui`. Never add raw `<button>` elements or build modal overlays from scratch.
- Use theme tokens (`var(--color-*)`). Never hardcode hex colours.
- No `any`: use `catch (err: unknown)` and narrow it.

### More files you must not edit

- `packages/api-contract/**`: owned together with the mobile team. Flag the request instead of editing.
- `apps/api/src/db/migrations/**`
