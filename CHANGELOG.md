# Changelog

## v1.0.0 (unreleased)

First public release.

- `claude-complexity-grade.yml`: grades PR risk, applies labels, and can optionally approve.
- `claude-comment-addressing.yml`: responds to human and review-bot feedback, capped by round, with dispatchable sweeps.
- `claude-test-remediation.yml`: fixes failing CI on PRs, capped by attempt count.
- `pr-restack.yml`: retargets and rebases stacked PRs when their parent merges.
- `load-prompt`: base prompts shipped with the workflows, which repos can extend or replace.
