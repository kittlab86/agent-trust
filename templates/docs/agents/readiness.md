# Autonomy Readiness

The agent may operate autonomously only when every item is ✅ **with evidence**.
Current trust level: **L0** (L0 in-the-loop · L1 local parallel · L2 cloud · L3 automations · L4 auto-merge)

| Area | Item | Status | Evidence |
|---|---|---|---|
| Safety | Least-privilege credentials; no prod write access | ☐ | |
| Safety | Branch protection + required checks; CODEOWNERS on protected paths | ☐ | |
| Safety | Secret scanning in CI | ☐ | |
| Verification | One-command dev / test / lint / typecheck / check | ☐ | |
| Verification | Control CLI (launch/goto/act/snapshot/logs/trace/heap/stop), headless, parallel-safe | ☐ | |
| Verification | Feature map covers all user-facing features; `check-feature-map` in CI | ☐ | |
| Verification | Vague report → feature → reproduction without help (3/3) | ☐ | |
| Skills | investigate, verify-app, create-/maintain-verification, ship-pr, eval-skill, triage-report installed | ☐ | |
| Skills | Each skill passes eval on all models in use; CI requires fresh eval on change | ☐ | |
| Skills | Failure ledger active; recurring failures escalated to hard layers | ☐ | |
| Constraints | `architecture.md`: nouns, co-location, boundaries, scaffolds | ☐ | |
| Constraints | Strict types, import boundaries, banned-pattern lints at **error** | ☐ | |
| Constraints | Every critical invariant has a hard layer (enforcement registry) | ☐ | |
| Constraints | AI reviewer configured with repo rules | ☐ | |
| Workflow | PR template + ship-pr; atomic PRs | ☐ | |
| Workflow | Cloud env runs verify-app headless | ☐ | |
| Workflow | Metrics: revert rate, escaped defects, triage accuracy | ☐ | |
| Workflow | Auto-merge policy, kill switch, auto-revert defined | ☐ | |

## Gate log
| Gate | Date | Approved by | Notes |
|---|---|---|---|
| 0 Baseline & safety | | | |
| 1 Verification | | | |
| 4 Constraints | | | |
| Autonomy | | | |
