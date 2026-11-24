---
name: triage-report
description: Use for incoming bug reports or user feedback (issue, Slack, or a bare screenshot). Maps the report to a feature, reproduces it on the released build and on main, and classifies it.
---

# Triage a report (bug-triage automation)

## Procedure
1. **Parse** the report: text, screenshots, version/build, OS, steps. Low-quality input ("???" plus a screenshot) is normal. Use the `visual_cues`, `names`, and `synonyms` in `docs/agents/feature-map.yaml` to identify the feature.
2. **Reproduce on the reported/released build** with `verify-app`.
3. **Reproduce on `main`** with `verify-app`.
4. **Classify**:
   | Released | main | Classification | Action |
   |---|---|---|---|
   | ✅ repro | ❌ no repro | **Fixed on main** | Comment with evidence and the likely fixing commit (`git log` on feature paths); suggest a release |
   | ✅ | ✅ | **Confirmed** | Run `investigate`; if the fix is small, inside one feature directory, and on no protected path → open a fix PR via `ship-pr`; otherwise attach the diagnosis |
   | ❌ | ❌ | **Can't reproduce** | List the attempted steps and environment; ask 1–3 precise questions |
   | n/a | n/a | **Needs info** | Can't map to a feature; ask, quoting the candidate features |
5. Look for **duplicates** among open issues for the same feature ID.

## Report format
```
Feature: <id> (confidence)
Classification: Fixed on main | Confirmed | Can't reproduce | Needs info
Evidence: <screenshots/traces/logs links>
Diagnosis: <from investigate, if confirmed>
Next step: <PR link | release | question>
```
