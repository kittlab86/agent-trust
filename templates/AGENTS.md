# AGENTS.md

Rules for every agent working in this repo. This file is a **soft** layer: anything critical here must also be enforced by CI (see `docs/agents/enforcement-registry.md`).

## Orientation
- Product map: `docs/agents/feature-map.yaml`. **Map every task to feature IDs first.**
- Architecture and conventions: `docs/agents/architecture.md`
- Skills: `.agents/skills/` (use them; they encode how we work here)
- Commands: `make dev` · `make test` · `make lint` · `make typecheck` · `make check` (all gates)
- Drive the app: `scripts/agent/control --help`

## Operating loop (every task)
1. **Intake**: map the request to feature IDs (`feature-map.yaml`). If nothing matches, ask.
2. **Reproduce / baseline**: use the `verify-app` skill. For bugs, reproduce on `main` first. If it's already fixed on `main`, report that and stop.
3. **Investigate**: use the `investigate` skill. Read the code before claiming a cause. Cite `file:line`.
4. **Implement**: use the scaffolds (`make new-<noun>`) and existing patterns. Stay inside `src/features/<feature>/` where possible.
5. **Check**: `make check` until green.
6. **Verify**: use `verify-app` to capture after-evidence that matches the feature's `verify` recipe.
7. **Ship**: use the `ship-pr` skill. Atomic PRs with evidence.
8. **Learn**: add surprises and failures to `docs/agents/failure-ledger.md` and propose a harder enforcement.

## Hard rules
- Never weaken a gate: no lint/type suppressions, skipped tests, lowered thresholds, or edits to CI to make a check pass.
- Never use production credentials or data. Dev/staging only.
- Protected paths need human approval: see `CODEOWNERS` (e.g. `infra/`, `migrations/`, `auth/`, `billing/`, `.github/workflows/`, `.agents/`).
- Don't claim something works without verification evidence.
- Don't put history, attributions, or reviewer remarks in code comments ("X said…", "previously…"). Put context in the PR or commit message.
- Respect the import boundaries in `docs/agents/architecture.md`. Shared code goes only in `src/shared/`.

## Escalate to a human when
- the task touches protected paths, security, privacy, data migrations, or billing
- a gate would need to change
- requirements are ambiguous or conflict with the feature map
- you can't reproduce the issue after following `verify-app` and `investigate`
