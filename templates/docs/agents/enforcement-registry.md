# Enforcement Registry

Every invariant of this codebase and the layers that enforce it.
Goal: every **critical** invariant has at least one **hard** layer (it turns CI red). Soft layers (rules, skills, AI review) back hard layers up; they never replace them.

Legend: **A** = Architecture/convention · **S** = Static analysis / CI (hard) · **R** = Rule in AGENTS.md · **K** = Skill · **B** = AI review rule

| Invariant | Critical | A | S (tool/rule) | R | K | B | Notes |
|---|---|---|---|---|---|---|---|
| Feature code is co-located in `src/features/<id>/` | yes | scaffold `make new-feature` | boundary lint | ✓ | | ✓ | |
| UI/renderer may not import main/server/infra modules | yes | dir split | dependency-cruiser rule `no-ui-to-main` | ✓ | | ✓ | |
| No type escapes (`any`, `@ts-ignore`, `# type: ignore`) | yes | | strict TS/pyright + lint | ✓ | | ✓ | |
| Banned footgun pattern (e.g. raw `useEffect` outside approved hooks) | yes | approved hooks | `no-restricted-syntax` | ✓ | | ✓ | |
| No narrative/history code comments | no | | comment lint (allow `INVARIANT:`/`WHY:`) | ✓ | | ✓ | |
| Interactive elements have stable test IDs | yes | | custom lint | ✓ | verify-app | | |
| Feature map matches code | yes | | `check-feature-map` | ✓ | maintain-verification | | |
| Skill changes re-evaluated | yes | | CI: fresh `evals/<skill>/results` | ✓ | eval-skill | | |
| Perf budget (bundle, startup, long tasks) | yes | | size-limit / perf job | ✓ | verify-app | | |
| No secrets in repo | yes | | gitleaks | ✓ | | ✓ | |
| Protected paths need human review | yes | | CODEOWNERS + branch protection | ✓ | | | |
