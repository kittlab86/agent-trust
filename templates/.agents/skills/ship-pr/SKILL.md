---
name: ship-pr
description: Use when work is ready to be committed or opened as pull requests. Splits work into atomic PRs with verification evidence.
---

# Ship atomic PRs

## Split
- One logical change per PR: **refactor**, **behavior change**, **rename/move**, **feature-map/skill update**, and **dependency bump** each go in separate PRs.
- Stack dependent PRs (base each on the previous one). Each one must pass `make check` on its own.
- Aim for a diff a reviewer can understand in ≤ 10 minutes. There's no hard cap, but a mechanical migration should be split by directory.

## Before opening
1. `make check` green locally.
2. `verify-app` evidence collected (before/after) for behavior changes.
3. No suppressions added (`eslint-disable`, `@ts-ignore`, `# type: ignore`, `skip`, `xfail`).
4. No protected paths touched, or else request human review explicitly.

## PR content (fill in `.github/pull_request_template.md`)
- Title: `<type>(<feature-id>): <imperative summary>`
- **Why** (intent/problem), **What** (the change), **Feature IDs**, **Evidence**, **Risk class**, **Rollback** (usually "revert this PR").
- Commit messages explain *why*. Git history is context for future agents.

## After merge
- If you were surprised by anything, add it to `docs/agents/failure-ledger.md`.
