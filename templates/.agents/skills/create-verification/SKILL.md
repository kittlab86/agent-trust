---
name: create-verification
description: Use once per repo (or per new app surface) to build the control CLI, add stable selectors, and generate the initial feature map.
---

# Create verification tooling

## 1. Pick the driver
| Surface | Driver |
|---|---|
| Web | Playwright / CDP |
| Electron | CDP via `--remote-debugging-port` + main-process log capture |
| iOS | `xcrun simctl`, XCUITest / `idb`, Instruments |
| Android | `adb`, UIAutomator, `perfetto` |
| CLI/TUI | pty driver with output snapshots |
| API | HTTP scripts, read-only DB inspection, profiler |

## 2. Build `scripts/agent/control`
Commands: `launch`, `goto <feature-id>`, `act`, `snapshot`, `logs`, `network`, `trace start|stop`, `heap`, `stop`.
Requirements: JSON on stdout; artifacts saved to `artifacts/<run-id>/`; headless by default and `--headed` for local watching; **parallel-safe** (random port, temp profile dir); non-zero exit on failure.

## 3. Make the UI addressable
Add `data-testid` (or accessibility IDs) to interactive elements, using the naming `<feature>.<element>`. Propose a lint rule that requires them in `src/features/**`.

## 4. Generate the feature map draft
Explore routes, menus, commands, keyboard shortcut registries, settings, and feature directories. For each user-facing feature, fill in an entry in `docs/agents/feature-map.yaml` (see the schema at the top of that file). Use **the words users would say** for `names` and `synonyms`; flag uncertain ones with `# TODO(human): confirm name`.

## 5. Add the consistency check
`scripts/agent/check-feature-map` fails if a selector or code path in the map doesn't exist, or a `src/features/*` dir has no entry. Wire it into `make check` and CI.

## 6. Prove it
For 3 sample features: launch → goto → act → snapshot → trace, all headless, with artifacts attached to the PR.
