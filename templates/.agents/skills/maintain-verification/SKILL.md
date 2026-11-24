---
name: maintain-verification
description: Use when check-feature-map fails, a selector or navigation path is stale, a feature is added, renamed, or removed, or verify-app couldn't reach a feature.
---

# Maintain verification tooling

## Triggers
- CI `check-feature-map` red
- A PR adds, renames, or removes a feature dir, route, shortcut, or test ID
- `verify-app` couldn't find or reach a feature
- Weekly scheduled run

## Procedure
1. Run `scripts/agent/check-feature-map` and list the mismatches.
2. For each changed feature: update `names/synonyms`, `visual_cues`, `reach`, `selectors`, `code`, and `verify`. Keep user-facing language.
3. Re-run `verify-app` on every touched feature (launch → goto → snapshot) to confirm the map is right **at runtime**, not only statically.
4. If the control CLI lacks something needed, extend it, with a test.
5. Ship as its own atomic PR: `chore(feature-map): …`, with evidence.

## Rules
- Never delete an entry for a feature that still exists. Mark it `status: deprecated` with the replacement ID.
- Keep synonyms that real users have used in bug reports; they help triage.
