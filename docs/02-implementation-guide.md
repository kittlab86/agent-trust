# Implementation Guide: Applying the Agent Trust Framework to an Application

This guide turns [01-framework.md](./01-framework.md) into steps you can run against **any target application** (web, Electron, mobile, CLI, or API). It doesn't depend on a particular stack or agent tool. Equivalents are listed for Cursor, GitHub Copilot, and Claude Code.

**How to use it**

1. Copy the **contents** of [templates/](../templates) into the target repo's root (`cp -R templates/. <target-repo>/`), and copy this guide to `<target-repo>/docs/agents/IMPLEMENTATION-GUIDE.md`.
2. Give an agent the **Bootstrap Prompt** (§11). It runs Phases 0–6 and stops at each 🧑 **Human Gate**.
3. Once every item in the **Autonomy Readiness Checklist** (§10) passes, switch the agent to the **Autonomous Operating Loop** (§9).

> Rule zero: **don't skip trust levels.** Each phase has an exit gate. Running more agents before the gate passes only burns tokens and erodes trust.

---

## Phase map

| Phase | Goal | Trust level reached | Main artifacts |
|---|---|---|---|
| 0 | Baseline and safety | L0 | `docs/agents/baseline.md`, scoped credentials |
| 1 | **Verification loop** | L0 → L1 | `scripts/agent/` control CLI, `docs/agents/feature-map.yaml`, `verify-app` skill |
| 2 | Skills from failure modes | L1 | `.agents/skills/*`, `docs/agents/failure-ledger.md` |
| 3 | Evals for skills | L1 | `evals/<skill>/rubric.md`, eval results |
| 4 | Constrained architecture and **hard enforcement** | L1 → L2 | lint/boundary/CI configs, `docs/agents/enforcement-registry.md`, scaffolds |
| 5 | PR discipline | L2 | PR template, `ship-pr` skill, branch protection |
| 6 | Scale out: cloud agents and automations | L2 → L3 | cloud env config, `triage-report` skill |
| 7 | Auto-merge | L3 → L4 | auto-merge policy, revert automation, metrics |

---

## Phase 0: Baseline and safety

**0.1 Classify the repo**
- **Greenfield / vibe-coded** (few conventions, no tests, mixed patterns): high risk. Plan for a constraint refactor in Phase 4.
- **Brownfield with infra** (conventions, CI, typed): lower risk. Phase 4 adds rules on top of what exists.

**0.2 Record a baseline** in `docs/agents/baseline.md`:
- stack, languages, how to build, run, and test (exact commands), CI duration, flaky tests
- current type strictness, lint config, test coverage, folder structure
- known performance/stability pain points (they'll become verification targets)

**0.3 Safety guardrails (do these before any autonomy)**
- Agents get **least-privilege credentials**: no production DB or prod secrets, dev/staging only. "The intern can't wipe prod."
- Secrets come from the environment or a vault, never from the repo. Add secret scanning (e.g. `gitleaks`) to CI.
- Protect `main`: PRs required, CI must pass, no force-push.
- Write down the **protected paths** that always need a human (e.g. `infra/`, `migrations/`, `auth/`, billing, CI config, `.agents/`).

**0.4 One-command dev environment**
- `make dev` / `npm run dev` / equivalent starts the app **non-interactively** with seed data and test accounts.
- `make test`, `make lint`, `make typecheck`, `make check` (all of them) are each one command with a clean exit code.

🧑 **Human Gate 0**: approve the baseline, credential scope, and protected-paths list.

---

## Phase 1: Verification loop (highest priority)

> Goal: the agent can **run the app the way a user does, act on it, observe it, and measure it**, then attach evidence to every change. Until this works, *you* are the bottleneck.

### 1.1 Build the control CLI (`scripts/agent/control`)
A small CLI the agent calls from the shell. It returns **machine-readable output** (JSON plus file paths to artifacts).

| Command | Purpose |
|---|---|
| `control launch [--build dev] [--seed <fixture>]` | Start the app in an isolated instance (own port and profile dir) and print the connection info |
| `control goto <feature-id>` | Navigate to a feature using the feature map |
| `control act <selector> <click\|type\|key> [value]` | Interact |
| `control snapshot` | Screenshot plus accessibility/DOM tree, saved to `artifacts/` |
| `control logs [--since]` | Console, main-process, and server logs |
| `control network` | Recent requests and their statuses |
| `control trace start\|stop` | CPU / performance trace (long tasks over 16 ms, FPS) |
| `control heap` | Heap snapshot / memory stats |
| `control stop` | Tear down |

**Driver per surface**

| Surface | Driver |
|---|---|
| Web | Playwright or Chrome DevTools Protocol (CDP) |
| Electron | CDP via `--remote-debugging-port`, plus main-process logs |
| iOS | `xcrun simctl` (boot, install, launch, screenshot, logs), XCUITest / `idb`, Instruments traces |
| Android | `adb`, UIAutomator / Espresso, `perfetto` traces |
| CLI / TUI | Pseudo-terminal driver (e.g. `node-pty`, `pexpect`) with output snapshots |
| API / backend | HTTP client scripts, DB inspection (read-only), structured logs, profiler (`py-spy`, `pprof`, `clinic`) |

Requirements:
- Runs **headless** in the cloud and headed locally (so a human can watch, per the advice to start local).
- **Parallel-safe**: several agents can each launch their own instance (random ports, temp profile dirs).
- Writes deterministic artifacts to `artifacts/<run-id>/`.

### 1.2 Make the UI addressable
- Add stable selectors (`data-testid` / accessibility identifiers) to every interactive element of mapped features.
- **Hard-enforce** it: a lint rule that requires test IDs on interactive components in feature directories (Phase 4).

### 1.3 Create the feature map (`docs/agents/feature-map.yaml`)
Use the [template](../templates/docs/agents/feature-map.yaml). One entry per user-facing feature:
- `id`, user-facing **names and synonyms** (what users and PMs actually say: "left sidebar", "chat list")
- **visual cues** (so a screenshot can be matched to it)
- **how to reach it**: navigation steps, keyboard shortcuts, deep link
- **selectors** for each sub-feature
- **code locations** (feature directory, entry points)
- **known perf/regression hotspots**, plus a **verification recipe** (what "working" looks like)

Generate the first draft with the `create-verification` skill (it explores routes, menus, and components), then have a human correct the names.

### 1.4 Keep the feature map honest (hard check)
Add `scripts/agent/check-feature-map` to CI:
- every `selector` in the map exists in source (grep or AST),
- every `code` path exists,
- every feature directory has a feature-map entry.

The map is then **hard-enforced** instead of drifting. `maintain-verification` fixes it when the check fails.

### 1.5 Install the skills
- [`verify-app`](../templates/.agents/skills/verify-app/SKILL.md): how to use the control CLI and feature map, and what evidence is required.
- [`create-verification`](../templates/.agents/skills/create-verification/SKILL.md) / [`maintain-verification`](../templates/.agents/skills/maintain-verification/SKILL.md).

**Exit criteria (Phase 1)**
- [ ] The agent can take a **vague report** ("sidebar is laggy") or a screenshot, map it to a feature, launch the app, reach it, and return a screenshot plus a perf trace **without help**.
- [ ] The agent can show **before/after evidence** for a bug fix.
- [ ] Feature-map validity check runs in CI.

🧑 **Human Gate 1**: watch 3 real tasks run locally end to end. You shouldn't have to copy-paste anything.

---

## Phase 2: Skills from observed failure modes

### 2.1 Observation protocol
For the first ~20 tasks, **read the agent's tool calls and thinking**, not only its final answer. Log every failure in [`docs/agents/failure-ledger.md`](../templates/docs/agents/failure-ledger.md):

| Failure | Evidence | Root cause | Fix type | Fix link |
|---|---|---|---|---|
| Claimed root cause without opening the file | transcript #12 | guessing | skill → `investigate` | … |
| Put heavy IO in the renderer | PR #45 | no boundary | **lint/CI** | … |

**Fix-type priority:** architecture removes the problem > lint/CI > rule in `AGENTS.md` > skill > AI-review rule. Always pick the **hardest** fix that's practical (see Phase 4).

### 2.2 Core skill set (templates provided)

| Skill | Prevents |
|---|---|
| `investigate` | Guessing and fake "smoking guns". Requires reading code, reproducing, citing `file:line`, and fanning out sub-agents |
| `verify-app` | Unverified "it works" claims |
| `create-verification` / `maintain-verification` | Missing or stale control tooling and feature map |
| `ship-pr` | Giant PRs, missing evidence, unclear history |
| `eval-skill` | Untested or regressed skills |
| `triage-report` | Hand-triaging bug reports (Phase 6) |

### 2.3 Skill authoring rules
- A skill = `SKILL.md` with front-matter (`name`, `description` that says **when** to use it), then **procedure**, **evidence required**, and **anti-patterns**.
- Write it the way you'd onboard a smart new hire with no context. Be concrete: include commands, paths, and examples.
- Keep the canonical copy in `.agents/skills/` and symlink it for each tool:
  - Cursor → `.cursor/skills/` (rules: `.cursor/rules/`)
  - GitHub Copilot → `.github/skills/` (instructions: `.github/copilot-instructions.md`, `AGENTS.md`)
  - Claude Code → `.claude/skills/` (`CLAUDE.md` → point to `AGENTS.md`)

**Exit criteria (Phase 2)**
- [ ] Ledger has ≥ 10 entries, each with a fix.
- [ ] No failure mode has recurred more than twice without a fix being escalated to a harder layer.

---

## Phase 3: Evals for skills

Follow [`eval-skill`](../templates/.agents/skills/eval-skill/SKILL.md). In short:

1. **Rubric**: the coordinator writes `evals/<skill>/rubric.md` with 5–10 scored criteria (0–10) plus 3–8 realistic task prompts.
2. **Neutral sandboxes**: each run gets its own git worktree or copy with a **neutral name** (e.g. `work/task-7f3a`, never `eval-...`). Prompts read like real tasks.
3. **Model matrix**: run every task on each model your team actually uses.
4. **Independent judge**: a judge agent on a **different model** scores the transcripts and outputs against the rubric. Flag disagreements with the coordinator's own score.
5. **Hill-climb**: edit the skill and re-run until the target score is reached (e.g. ≥ 9/10 on every criterion for every model). Use a loop command (`/loop` in Cursor, or a script) with a **max-iteration cap and a token budget**.
6. **Regression**: store results in `evals/<skill>/results/<date>.json`. **Any change to `.agents/skills/**` must re-run its eval.** Enforce this in CI: a skill change without a fresh results file fails.

**Exit criteria (Phase 3)**
- [ ] Every core skill has a rubric, at least one passing result per model, and a CI check that requires re-running the eval when the skill changes.

---

## Phase 4: Constrained architecture and hard enforcement

### 4.1 Define the conventions (the "nouns")
Write `docs/agents/architecture.md` with:
- the **small set of nouns** in your app (e.g. `feature`, `route/entry-point`, `card/component`, `service`, `store`, `job`) and **the one way** to create each one.
- **Co-location**: `src/features/<feature>/` holds UI, state, API calls, tests, fixtures, and its feature-map entry. Shared code goes only in `src/shared/` and must pass an explicit review gate.
- **Process/layer boundaries**: e.g. `main/` vs `renderer/`, `server/` vs `client/`, `domain/` vs `infra/`.
- **Scaffolds**: `make new-feature NAME=x` generates the standard structure. *The shortest path is the correct path.*

### 4.2 Hard gates (each one turns CI red)

| Gate | Examples of tools |
|---|---|
| Strict types / compiler | TS `strict` + `noUncheckedIndexedAccess`; mypy/pyright strict; Rust deny `unsafe`; Go vet + staticcheck |
| Import boundaries / dependency graph | `dependency-cruiser`, `eslint-plugin-boundaries`, Nx module boundaries, `import-linter` (Python), ArchUnit (JVM), Go `internal/`, Cargo workspace crates |
| Banned patterns (from the ledger) | ESLint `no-restricted-syntax` / `no-restricted-imports`, Semgrep, custom lint rules. E.g. ban `useEffect` outside approved hooks, ban `any`, ban direct `fetch` outside the API layer, ban sync IO in the UI thread |
| Comment policy | Ban comments **or** allow only a tagged form (e.g. `// INVARIANT:` / `// WHY:`) through a lint rule, which stops agents writing history and "X said" into comments |
| Tests required | changed feature directory → its tests must change or exist; coverage floor per feature |
| Performance budgets | bundle size limits, startup time, long-task / FPS checks from the control CLI in CI |
| Feature-map validity | `check-feature-map` (Phase 1.4) |
| Skill eval freshness | Phase 3.6 |
| Secret scanning | `gitleaks` |
| Protected paths | CODEOWNERS plus a required human review for paths from Phase 0.3 |

### 4.3 The enforcement registry
Track every invariant in [`docs/agents/enforcement-registry.md`](../templates/docs/agents/enforcement-registry.md) with the layer that enforces it:
`Architecture | Static (hard) | Rule (AGENTS.md) | Skill | AI review`.
Target: **every important invariant has at least one hard layer.** Soft layers back it up; they don't replace it.

### 4.4 The rule-conversion reflex (permanent process)
Whenever a human writes a review comment that enforces an invariant:
1. Add it to the failure ledger.
2. Turn it into the **hardest** enforcement that's practical (architecture > lint/CI > rule > skill > AI review).
3. Fix existing violations through codemod PRs, then switch the rule from *warn* to *error*.

### 4.5 Brownfield migration strategy
- Bring rules in as **warn**, run a codemod or agent migration in **small atomic PRs** per directory, then flip to **error**. That's how the "600 PRs" refactor gets done safely.
- Only **rewrite** when the greenfield code has no conventions to keep. Even then, rewrite feature by feature behind the new scaffolds.

### 4.6 AI code review (soft layer)
Turn on an AI reviewer (Cursor Bugbot, Copilot code review, etc.) and give it the ruleset from `AGENTS.md` and the enforcement registry.

**Exit criteria (Phase 4)**
- [ ] `make check` runs every hard gate locally and in CI.
- [ ] Every row in the enforcement registry marked **critical** has a hard layer.
- [ ] `make new-feature` exists, and an agent built one new feature using only the scaffold, without breaking any gate.

🧑 **Human Gate 4**: review the architecture doc and the enforcement registry.

---

## Phase 5: PR discipline

- Use the [`ship-pr`](../templates/.agents/skills/ship-pr/SKILL.md) skill and the [PR template](../templates/.github/pull_request_template.md).
- **One logical change per PR**. Split refactors, behavior changes, and renames apart. Stack PRs when they depend on each other.
- Every PR body includes: the intent, the feature-map IDs it touches, **verification evidence** (artifact links, before/after), and its risk class.
- Commit messages explain *why*. Git history is context for future agents.
- Branch protection: required checks = `make check` + tests + verification job.

---

## Phase 6: Scale out (cloud agents and automations)

### 6.1 Cloud environment
- A reproducible environment (Dockerfile / devcontainer / Cursor background-agent env / Copilot coding-agent `copilot-setup-steps.yml`) that installs dependencies, builds the app, and can run `scripts/agent/control launch` **headless**.
- The same skills, `AGENTS.md`, and least-privilege credentials as local.
- Check it: a cloud agent runs `verify-app` on a sample feature and uploads its artifacts.

### 6.2 Parallel local agents
- One **git worktree** per agent; the control CLI isolates ports and profiles.

### 6.3 Automations (L3)
| Automation | Trigger | Output |
|---|---|---|
| **Bug triage** (the "Benny" pattern) using [`triage-report`](../templates/.agents/skills/triage-report/SKILL.md) | New issue or Slack message, even just a screenshot | Comment: mapped feature, repro status on the **released build and on `main`**, evidence, classification (already fixed / reproduced / can't reproduce / needs info), and a fix PR when it's safe |
| Perf watchdog | Nightly | Runs the control CLI traces on hot features and opens an issue or PR on regressions |
| Feature-map upkeep | Weekly, or when the check fails | `maintain-verification` PR |
| Skill regression | When a skill or model changes | `eval-skill` run, with a report |

**Exit criteria (Phase 6)**
- [ ] Triage automation has processed ≥ 20 real reports, and humans agree with its classification ≥ 90% of the time.

---

## Phase 7: Auto-merge (L4)

Turn it on **one risk class at a time**, low-risk first (docs, tests, small fixes inside a single feature directory).

**An agent PR may auto-merge only if all of these hold:**
- every hard gate is green and verification evidence is attached
- the AI reviewer has no unresolved high-severity findings
- it touches **no protected paths**
- the diff stays within the size or scope policy (e.g. one feature directory, or the PR is labeled as part of an approved migration)
- the area has feature flags or a canary when user-facing

**Safety net**
- **Post-merge review on `main`**: a daily digest of what landed (an agent summarizes it; a human skims it).
- **Automatic revert**: if post-merge checks or canary fail, an agent opens and merges a revert PR.
- **Kill switch**: one label or config flag turns auto-merge off repo-wide.
- **Metrics** (weekly): revert rate, escaped defects per 100 agent PRs, CI flake rate, triage accuracy, human-review minutes per PR. **If the revert rate or escaped defects go up, drop back a trust level.**

---

## 9. Autonomous Operating Loop (after readiness)

Each task an agent picks up runs this loop. It's also written in [`AGENTS.md`](../templates/AGENTS.md):

1. **Intake**: read the task and map it to feature IDs using `feature-map.yaml`.
2. **Reproduce / baseline**: `verify-app` captures the current behavior (bugs: reproduce on `main`; if it's already fixed, report that and stop).
3. **Investigate**: `investigate` reads the code, cites `file:line`, and forms hypotheses backed by evidence. No guessing.
4. **Implement**: follow the scaffolds and conventions, staying inside the feature directory where possible.
5. **Check**: `make check` locally until green. Never weaken or bypass a gate (no `eslint-disable`, `@ts-ignore`, skipped tests, or lowered thresholds).
6. **Verify**: `verify-app` captures after-evidence that matches the feature's verification recipe.
7. **Ship**: `ship-pr` splits the work into atomic PRs with evidence.
8. **Merge**: auto-merge if eligible (Phase 7); otherwise request review.
9. **Learn**: if anything went wrong or felt unclear, add it to the failure ledger, propose a harder enforcement, and update the feature map or skills (with a re-run eval).

**Stop and escalate to a human when:** a protected path is involved, a gate would need to be weakened, requirements are ambiguous, the task can't be reproduced after the documented attempts, or there are security, privacy, data-migration, or billing implications.

---

## 10. Autonomy Readiness Checklist (Definition of Ready)

The agent may start working **autonomously** only when every item below is ✅. Track it in `docs/agents/readiness.md`.

**Safety**
- [ ] Least-privilege credentials; no production write access
- [ ] Branch protection plus required checks; protected paths have CODEOWNERS
- [ ] Secret scanning in CI

**Verification**
- [ ] One-command `dev`, `test`, `lint`, `typecheck`, `check`
- [ ] Control CLI: launch / goto / act / snapshot / logs / trace / heap / stop, headless and parallel-safe
- [ ] Feature map covers 100% of user-facing features, and `check-feature-map` passes in CI
- [ ] The agent maps a vague report to a feature and reproduces it without human help (3/3 trials)

**Skills and evals**
- [ ] `investigate`, `verify-app`, `create-/maintain-verification`, `ship-pr`, `eval-skill`, `triage-report` installed
- [ ] Each skill passes its eval on every model in use; CI requires a fresh eval when a skill changes
- [ ] Failure ledger active, and every recurring failure has been escalated to a hard layer

**Constraints**
- [ ] `docs/agents/architecture.md` defines the nouns, co-location, boundaries, and scaffolds
- [ ] Strict types, import boundaries, and banned-pattern lints all **error** in CI
- [ ] Enforcement registry: every critical invariant has a hard layer
- [ ] AI reviewer configured with the repo rules

**Workflow**
- [ ] PR template plus `ship-pr`; atomic PRs are the norm
- [ ] Cloud environment runs `verify-app` headless
- [ ] Metrics dashboard (revert rate, escaped defects, triage accuracy)
- [ ] Auto-merge policy, kill switch, and automatic revert defined (they can start disabled)

---

## 11. Bootstrap Prompt (give this to the agent in the target repo)

```text
You are bootstrapping this repository for autonomous agent work using the Agent Trust Framework.
Read docs/agents/IMPLEMENTATION-GUIDE.md (copied from 02-implementation-guide.md) and AGENTS.md.

Execute Phases 0–6 in order. For each phase:
1. Create a short plan listing the files you'll add or change.
2. Implement it in small atomic PRs (or commits if no remote), following the ship-pr skill.
3. Prove the phase's exit criteria with evidence (commands run, artifacts, CI output).
4. Update docs/agents/readiness.md with the checked items and evidence links.
5. At every "Human Gate", STOP and summarize: what you did, the evidence, open questions, and decisions you need.

Constraints:
- Never use production credentials or touch protected paths without approval.
- Prefer the hardest practical enforcement (architecture > lint/CI > rule > skill > AI review).
- Never weaken an existing check to make something pass.
- Log every failure or surprise you hit in docs/agents/failure-ledger.md.

When every item in docs/agents/readiness.md is checked and the human approves, switch to the
Autonomous Operating Loop defined in AGENTS.md.
```

---

## 12. Target repo layout after the bootstrap

```
AGENTS.md                         # rules + autonomous operating loop (soft layer)
.agents/skills/                   # canonical skills (symlinked to .cursor/.github/.claude)
  investigate/ verify-app/ create-verification/ maintain-verification/
  ship-pr/ eval-skill/ triage-report/
scripts/agent/
  control                         # control CLI (launch/goto/act/snapshot/logs/trace/heap/stop)
  check-feature-map               # CI: feature map ↔ code consistency
docs/agents/
  baseline.md  architecture.md  feature-map.yaml
  failure-ledger.md  enforcement-registry.md  readiness.md
evals/<skill>/rubric.md, results/
.github/pull_request_template.md, CODEOWNERS, workflows/check.yml
src/features/<feature>/           # co-located feature code (convention)
artifacts/                        # verification output (gitignored)
```
