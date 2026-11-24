---
name: eval-skill
description: Use whenever a skill in .agents/skills is created or changed, or when a new model is adopted. Unit-tests the skill's effect on agent behavior and hill-climbs it.
---

# Eval a skill (unit tests for agents)

## Inputs
- Skill under test: `.agents/skills/<skill>/SKILL.md`
- Models: the list in `evals/models.txt` (the models the team actually uses)
- Judge model: a model **not** used as the coordinator

## Procedure
1. **Rubric**: write or refresh `evals/<skill>/rubric.md`:
   - 5–10 criteria, each scored 0–10 with clear anchors (what 0, 5, and 10 look like)
   - 3–8 realistic task prompts that should trigger the skill (include vague, messy ones)
2. **Neutral sandboxes**: for each (task × model), create an isolated git worktree with a **neutral name** (e.g. `../wt/task-<random>`). Never put "eval", "test", or "benchmark" in paths, branch names, or prompts. Prompts must read like real work.
3. **Run** a sub-agent per sandbox with the repo's normal setup (AGENTS.md + skills). Save the full transcript (tool calls included) and the diff.
4. **Score**: the coordinator scores each run against the rubric. The **judge model** independently scores the same runs. Record both, and flag any criterion where they differ by ≥ 3.
5. **Hill-climb**: if any criterion is below target (default ≥ 9 for every model), edit the skill (clearer steps, examples, anti-patterns) and go back to step 3. Cap: 10 iterations or the token budget in `evals/budget.txt`.
6. **Record** `evals/<skill>/results/<YYYY-MM-DD>.json`:
   `{skill_sha, models, tasks, scores:{coordinator,judge}, iterations, notes}`
7. Clean up the worktrees.

## Output
A table of model × criterion scores, a list of the skill edits made, and the remaining weak spots.
