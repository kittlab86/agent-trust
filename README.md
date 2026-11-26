# agent-trust

A guide and framework for applying the Agent Trust Framework to **any repository** — so you can select a repo, build the required trust infrastructure, and have agents work autonomously.

## The core idea

You can only parallelize agents you trust, and trust is engineered, not granted. This framework gives you a repeatable process: pick a repo, follow the phases, and end up with agents that can merge their own PRs while you review on `main`.

## Trust levels

| Level | Description |
|---|---|
| **L0: In the loop** | 1–3 agents. You watch every tool call and verify output yourself. |
| **L1: Local parallel** | Agents verify themselves; you review the evidence. |
| **L2: Cloud agents** | Agents run in cloud machines with the same verification skills. |
| **L3: Automations** | Agents triggered by signals (bug reports, CI) with no human prompt. |
| **L4: Auto-merge** | Agents merge their own PRs; you review on `main` after the fact. |

## The four pillars

1. **Verification** — agents can run the real app, reach every feature, and collect evidence (screenshots, traces, logs).
2. **Skills from failure modes** — skills are built incrementally from real agent failures, not guessed.
3. **Evals for skills** — every skill is unit-tested against a rubric, across models, with a judge cross-checking scores.
4. **Hard enforcement** — architecture constraints, lint rules, and CI gates turn violations into red builds, not just comments.

## How to apply this framework to your repo

1. **Copy the templates** into your target repo:
   ```
   cp -R templates/. <target-repo>/
   ```
2. **Read the framework** (`docs/01-framework.md`) and the **implementation guide** (`docs/02-implementation-guide.md`).
3. **Give an agent the Bootstrap Prompt** (§11 of the implementation guide) to run Phases 0–6.
4. **Select your target repo** — any repo (web, Electron, mobile, CLI, API) — and follow the phases in order. Each phase has an exit gate; don't skip trust levels.

## Key artifacts (copied into your target repo)

| Artifact | Purpose |
|---|---|
| `docs/agents/feature-map.yaml` | Maps every user-facing feature to selectors, code paths, and verification recipes |
| `docs/agents/architecture.md` | Defines the nouns, co-location rules, and import boundaries |
| `docs/agents/enforcement-registry.md` | Every invariant and the hard/soft layer that enforces it |
| `docs/agents/failure-ledger.md` | Every observed agent failure, with root cause and fix layer |
| `docs/agents/readiness.md` | Autonomy readiness checklist with evidence |
| `.agents/skills/` | Skills: `investigate`, `verify-app`, `ship-pr`, `eval-skill`, `triage-report`, etc. |
| `scripts/agent/control` | CLI to launch, navigate, act on, snapshot, and profile the app |

## Recommended path

Start local so you can observe, then move to the cloud. Don't skip trust levels — running more agents before the gate passes only burns tokens and erodes trust.

## Source

Based on *"Lauren Tan Reveals How to Build a 21+ Agent GrokBot Team"* (Cursor). The talk explains how one engineer went from watching a few agents closely to having about 20 agent PRs auto-merged overnight, ~1,000 PRs/month, with PMs and designers safely shipping code.
