## Why
<!-- Problem / intent. Link issue or report. -->

## What
<!-- The single logical change in this PR. -->

## Feature IDs
<!-- From docs/agents/feature-map.yaml -->

## Verification evidence
<!-- verify-app artifacts: before/after screenshots, logs, trace summary (long tasks, FPS), heap numbers -->

## Risk
- [ ] Low (single feature dir, no protected paths) — auto-merge eligible
- [ ] Medium (cross-feature / shared code)
- [ ] High (protected paths, data, auth, billing, infra) — human review required

## Checklist
- [ ] `make check` green
- [ ] No suppressions added (lint/type ignores, skipped tests, lowered thresholds)
- [ ] Feature map / skills updated if behavior or navigation changed
- [ ] Rollback: revert this PR
