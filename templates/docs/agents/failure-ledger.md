# Failure Ledger

Every observed agent failure (from reading tool calls and thinking, PR reviews, reverts, or escaped bugs).
Pick the **hardest** practical fix: Architecture > Static/CI (hard) > Rule (AGENTS.md) > Skill > AI review.
If a failure recurs after a soft fix, **escalate it** to a harder layer.

| # | Date | Failure (what the agent did) | Evidence (transcript/PR) | Root cause | Fix layer | Fix (link) | Recurred? |
|---|---|---|---|---|---|---|---|
| 1 | YYYY-MM-DD | Claimed root cause without reading the affected file | session/PR link | guessing | Skill: `investigate` | link | no |
| 2 | YYYY-MM-DD | Imported heavy module into UI thread | PR #… | no boundary | Static: import-boundary rule | link | no |
| 3 | YYYY-MM-DD | Wrote "per reviewer, never do X" code comment | PR #… | treats feedback as global rule | Static: comment lint | link | no |
