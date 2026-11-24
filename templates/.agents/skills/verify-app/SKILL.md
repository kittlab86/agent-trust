---
name: verify-app
description: Use to run the real application, reach a feature, reproduce a bug, check a change works, or measure performance. Required before claiming anything works.
---

# Verify the app for real

Tooling: `scripts/agent/control` (JSON output; artifacts saved under `artifacts/<run-id>/`).
Product map: `docs/agents/feature-map.yaml`.

## Procedure
1. **Find the feature**: match the request, screenshot, or user wording against `names`, `synonyms`, and `visual_cues` in the feature map. Record the feature ID(s).
2. **Launch** an isolated instance:
   `scripts/agent/control launch --build dev --seed <fixture-from-feature-map>`
3. **Navigate**: `control goto <feature-id>` (or follow the `reach.steps` / `reach.shortcut` in the map).
4. **Baseline**: `control snapshot`, `control logs`, plus `control trace start|stop` for perf issues and `control heap` for memory issues.
5. **Act**: follow the user's steps with `control act <selector> …`, using selectors from the map.
6. **Judge** the result against the feature's `verify` recipe. Check the console/main logs for errors.
7. After the change, **repeat 2–6** and compare with the baseline.
8. `control stop`.

## Evidence required in the PR / report
- Feature ID(s), the steps run, and **before/after** screenshots or snapshots
- Log excerpts (no errors introduced)
- For perf: trace summary (long tasks > 16 ms, FPS, duration) before and after
- For memory: heap size before and after the scenario repeated N times

## If something is missing
- Feature not in the map, or a selector is stale → run the `maintain-verification` skill. Don't guess selectors.
- The control CLI can't do something you need → extend it (small PR) instead of working around it by hand.
