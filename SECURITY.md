# Security policy

These workflows run an AI agent with write access to your repository, so we take reports seriously.

## Reporting a vulnerability

Please **don't open a public issue**. Report it privately through GitHub's
[private vulnerability reporting](https://github.com/AlpineSG/claude-pr-actions/security/advisories/new) for this repository.

Include the workflow, the triggering event, and the steps to reproduce. We aim to acknowledge reports within 3 business days.

## Scope

Things we especially want to hear about: ways for an untrusted user (fork PRs, commenters without write access) to trigger a privileged run, to inject into shell steps or prompts, to get around the round or attempt caps, or to make the App push to a branch it shouldn't.
