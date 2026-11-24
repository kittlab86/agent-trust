# The Agent Trust Framework

Source: *"Lauren Tan Reveals How to Build a 21+ Agent GrokBot Team"* (Cursor). The talk explains how one engineer went from watching a few agents closely to having about 20 agent PRs **auto-merged overnight**, about 1,000 PRs a month, with PMs and designers safely shipping code too.

The idea underneath all of it: **you can only parallelize agents you trust, and trust is engineered, not granted.** Every part below exists to build trust, and each one is something you can check.

---

## 0. Mental model: the engineer as manager / head chef

| Old role | New role |
|---|---|
| You write the code | You **set up the kitchen**: environment, tools, conventions, stations |
| You check every output | You build **systems that check the output for you** |
| You enforce standards by reviewing PRs | You turn standards into **hard constraints** the agents can't get around |
| Your output = your typing | Your output = the throughput of your agent team **plus everyone else's** (PMs, designers, GTM) |

New agents are like a strong coder hired 5 seconds ago with **zero business context**. You onboard them the way you'd onboard a person: with documented skills, a map of the product, and guardrails.

---

## 1. The Trust Curve (the overall shape of the framework)

How much you trust agents sets how many you can run in parallel. **There is no shortcut.** If you spawn 100 agents before you trust 1, you only burn tokens.

| Level | Description | What unlocks the next level |
|---|---|---|
| **L0: In the loop** | 1–3 agents. You watch every tool call, prompt constantly, and *you* are the verifier (copy-pasting screenshots and console errors). | Agents can verify their own work (§2) |
| **L1: Local parallel** | A handful of local agents, each verifying itself; you review the evidence. | Skills cover the observed failure modes (§3) and are eval-tested (§4) |
| **L2: Cloud agents** | Agents run in cloud machines with the same control/verification skills. | Hard guardrails in the codebase and CI (§5, §6) |
| **L3: Automations** | Agents get triggered by signals (bug reports, Slack, CI) with no human prompt. Example: **"Benny"**, a triage bot that reproduces every bug report. | Low defect and revert rate on agent PRs over time |
| **L4: Auto-merge** | Agents merge their own PRs; the human reviews **on `main`** after the fact. | (End state) |

Recommended path: **start local so you can observe**, then move to the cloud.

---

## 2. Pillar 1: Verification (the most important skill)

> Verification = the agent can **run the code for real**, the same way users reach it, and collect evidence.

- Examples: drive the app through the Chrome DevTools Protocol (Electron/web), the iOS simulator through Apple tooling, take **CPU traces and heap snapshots**, read console logs.
- It **closes the loop**. It doesn't make code *good*, but it makes code **correct**, and that's the big step toward trust.
- Without it, **you are the verifier and the bottleneck**, and you can't parallelize.

### 2.1 Control skill (example: "control-glass")
A skill plus a small CLI that lets the agent **launch a dev build, operate the UI, observe it, and profile it**. The agent can write the CLI itself.

### 2.2 Feature map (the key artifact)
Being able to drive the app wasn't enough. The agent **didn't know what the product was**, so it flailed when someone said "the left sidebar is laggy." The feature map tells the agent, for every feature:
- what it is from **the user's point of view** (names, visual cues, sub-features),
- **how to reach it** in the UI (navigation path, **keyboard shortcuts**),
- the **selectors / DOM attributes** used to drive it through automation,
- where the code lives.

With it, even a **vague report or a screenshot captioned "???"** can be mapped to a feature, reproduced, and diagnosed.

### 2.3 Lifecycle skills
- **create-verification**: explores the code, builds the control tooling, and drafts the first feature map.
- **maintain-verification**: keeps the control tooling and feature map current as the product changes.

---

## 3. Pillar 2: Skills built from observed failure modes

- A skill is **just markdown**, but it carries instructions and context that pull the model toward better behavior ("high-quality tokens in, smarter pattern-matching out").
- **Build skills incrementally from real failures.** Read the agent's **tool calls and thinking blocks**. Example: an agent claimed "found the smoking gun" without ever reading the affected code. The fix was an investigation skill (the talk calls it "how"): *stop guessing, actually read the code, use sub-agents, don't claim a root cause without evidence.*
- Be an active **backseat driver** while you build up your skills, like a pair-programming partner who keeps asking "why didn't you do X?"
- Skill libraries (like the speaker's **pstack**, modeled on Garry Tan's *gstack*) can be forked, but **your own engineering standards** are what you need to encode.

---

## 4. Pillar 3: Evals for skills (unit tests for agents)

How skills are maintained and trusted:
1. A **coordinator agent** writes a **rubric** for what the skill should achieve.
2. It spawns **sub-agents** in separate working directories with **neutral names**, so they don't realize they're being evaluated. Agents that notice they're under test change their behavior.
3. Run the same eval across **several models** (a model matrix).
4. A **judge agent on a different model** cross-checks the scores so the coordinator's bias doesn't skew them.
5. **Hill-climb**: loop until the eval scores e.g. 10/10.
6. Run the eval **every time a skill changes**.

The same loop was used to build and harden the verification/control skill itself.

---

## 5. Pillar 4: A constrained, agent-first architecture (example: "Dune")

### 5.1 Why
- **Greenfield / vibe-coded apps are the biggest risk** (and the biggest opportunity). With no guardrails, agents take the most convenient path every time. The result is "organic architecture" that spirals out of control.
- **Brownfield apps with big-tech-style infra** (conventions, frameworks, restricted credentials so "the intern can't wipe the prod DB") are already fairly safe for agents.
- So a **rewrite or refactor toward strict constraints can be worth it**. The speaker spent ~600 PRs refactoring GrokBot and afterwards stopped needing to read the code.

### 5.2 Design principles
1. **The shortest path is the best path.** Agents take shortcuts, so make the shortcut the correct way.
2. **Design for the least capable agent.** Even small, cheap models should produce good code.
3. **Conventional "nouns"**: a small set of framework concepts (e.g. *feature*, *entry point*, *transcript card*), each created in one standard way. Agents copy existing patterns, so conventions are the **strongest** form of enforcement.
4. **Co-location**: everything for one feature lives in **one directory**. About 80% of work stays inside it, with no grepping around.
5. **Boundaries that can't be crossed**, enforced on the dependency graph. Example: `electron-main/` and `electron-renderer/` can't import each other, which keeps heavy or IO work off the 16 ms render frame.
6. **Ban whatever agents are bad at.** Examples: React `useEffect` is banned, and **code comments are banned**, because agents write comments that record a passing reviewer remark ("Lauren said never do this") as if it were a permanent rule.
7. Pick a **strict tech stack**: strong compilers and type systems (e.g. Rust with no `unsafe`) mean "it compiles" carries real information.
8. Strict CI is annoying for humans, **but agents absorb the annoyance**.

---

## 6. Pillar 5: Layered enforcement (hard > soft)

| # | Layer | Strength |
|---|---|---|
| 1 | **Codebase / architecture conventions** (structure, nouns, co-location) | Strongest, because agents copy patterns |
| 2 | **Static analysis**: CI checks, lint rules for observed bad patterns, compiler diagnostics, import-graph checks | **Hard**: turns CI red |
| 3 | **Rules** (`AGENTS.md`, rules files) | Soft: can be forgotten |
| 4 | **Skills** | Soft |
| 5 | **AI code review** (e.g. Bugbot with rules) | Soft |

- Use **every** layer, but **never rely on the soft layers alone**. Otherwise "it's only a matter of time before your code base looks like complete trash."
- **The rule-conversion reflex:** each time a human writes a review comment that enforces an invariant, treat it as a **code smell**. Ask: *how do I make this a lint rule, a CI failure, or remove the problem category entirely?* Enforcing invariants through human code review is the worst place to be.

---

## 7. Pillar 6: Scaling out (cloud agents and automations)

- Once the environment is set up, **control and verification skills benefit the whole team and company**, not just you.
- **Bug-triage automation ("Benny")**: takes every incoming bug report, starts a cloud machine running the app, uses the control skill and feature map to **reproduce** it, and reports back, e.g. *"reproduced, but already fixed on `main`; just needs a release."* That saves an hour of human investigation per report.
- Non-engineers (PMs, designers, GTM) contribute code safely **because the constraints hold**. One example: a PM's fix was "just perfect → stamp."

---

## 8. Pillar 7: PR discipline and auto-merge

- Agents should **split work into multiple atomic PRs**. Each PR describes one small change. There's no hard size cap (typically 50 to ~1,000 lines).
- Why: **git history is rich context** for agents and humans, and atomic PRs are easy to **bisect and revert** (unlike a "40,000-line PR").
- End state: agents **auto-merge**, and the human reviews what landed **on `main`**.

---

## 9. Economics (ROI)

- Large token spend **up front**: refactoring and building guardrails and verification.
- Payoff: cheaper, smaller models do excellent work in a constrained codebase; the whole team (including non-engineers) gets faster; you stay a **lean org** instead of a 10,000-person engineering org.
- Compare it with hiring: *pay an engineer for years, or spend tokens once so that even the "dumbest agent" does a good job.*
- Use models on the cost-versus-intelligence **Pareto frontier**.

---

## 10. One-page summary

```
TRUST = Verification × Skills(evaluated) × Hard Constraints
        └─ control CLI + feature map   └─ from failures, eval-tested   └─ architecture + CI

Climb:  in-the-loop → local parallel → cloud agents → automations → auto-merge (review on main)
Reflex: every human review comment → lint / CI check / architectural elimination
Shape:  shortest path = correct path; co-located features; banned footguns; atomic PRs
```
