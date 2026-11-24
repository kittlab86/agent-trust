---
name: investigate
description: Use for any bug, regression, "why does X happen", or perf question. Turns guessing into evidence-backed diagnosis. Use before proposing a fix.
---

# Investigate (no guessing)

## Principles
- **No root cause without evidence.** "Smoking gun" claims must cite the code you actually read (`file:line`) and the runtime evidence (log line, trace, screenshot).
- **Reproduce before you diagnose.** Use the `verify-app` skill to reproduce. A bug you couldn't reproduce stays a *hypothesis*.
- **Read, don't assume.** Open the files on the execution path. Don't infer behavior from names.

## Procedure
1. **Map** the report to feature IDs in `docs/agents/feature-map.yaml`. Note its `code` paths and `hotspots`.
2. **Reproduce** with `verify-app` on `main`. Save the artifacts. If it doesn't reproduce on `main` but does on the released build, report **"fixed on main"** and stop.
3. **Trace the path**: start at the entry point from the feature map and follow calls to where behavior diverges. For large searches, **fan out sub-agents** (e.g. one per module or hypothesis), each returning cited findings.
4. **List hypotheses** (2–4), each with *evidence for*, *evidence against*, and *a test that tells them apart*.
5. **Run the tests that tell them apart** (logs, breakpoints, traces, minimal repro, `git bisect`, `git log -p` on suspect files).
6. **Conclude** with a root-cause statement in this form:
   `Cause: <mechanism>. Evidence: <file:line>, <artifact>. Confidence: high|medium|low.`

## Output
```
Feature: <ids>
Repro: yes/no (artifacts: …)
Root cause: … (file:line)
Evidence: …
Rejected hypotheses: …
Proposed fix: … (smallest change; which conventions it follows)
```

## Anti-patterns (stop if you catch yourself)
- Saying "it must be X" before opening X.
- Fixing symptoms (retry, timeout bumps, try/catch swallowing) without a mechanism.
- Changing several things at once and then calling it fixed.
