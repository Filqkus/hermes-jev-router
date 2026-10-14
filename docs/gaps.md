# Where jev-router falls short of "skip the main model when it isn't needed"

> **Last reviewed:** 2026-10-14 · **Last updated:** 2026-10-14 · **Change:** major (first write)

Every known shortfall, one entry each, ranked by how much it stops real work. Agents: pick from the top unless the owner says otherwise. Numbers behind the ranking: [parity.md](parity.md). Hours = one agent, sequential, tests included. "Estimated" unless marked measured.

## Ranked
The **alpha gate is not passed** — blockers #1 and #2 must clear before any "does it save tokens?" claim is meaningful.

| # | Gap | Kind | Hours | Doc |
|---|---|---|---|---|
| 1 | No live end-to-end run: patch applicability to the current Hermes build + the hook firing are unproven (**alpha blocker**) | Integration | 4–8 | [parity](parity.md) |
| 2 | Live skip unverified: the 2-vs-1 main-model call pattern has only been asserted with mocked TypeSafe (**alpha blocker**) | QA | 4–6 | [parity](parity.md) |
| 3 | Token/cost savings unmeasured on a real workload (telemetry exists; no before → after) | Measurement | 4–6 | [parity](parity.md) |
| 4 | Fail-open only unit-tested, never exercised live against a TypeSafe outage/timeout | QA | 2–4 | [parity](parity.md) |
| 5 | Added Jev latency per tool round unmeasured (a Jev call per round could add wall-clock) | Perf | 3–5 | [parity](parity.md) |
| 6 | Manual core patch, no automated rebase or version pin — breaks silently on Hermes upgrades | Maintenance | 4–8 | [parity](parity.md) |
| 7 | Windows install path documented but not executed end-to-end during this review | Docs/QA | 2–4 | [parity](parity.md) |
| 8 | `jev_execution_plan` (autonomous pre-planned continuation) experimental, off by default, untested live | Feature | 4–8 | [parity](parity.md) |

## Entries
### 1. No live end-to-end run (alpha blocker)
- **Goal:** G2
- **Landed:** `patches/hermes-post-tool-round-control.patch` touching only `hermes_cli/plugins.py`, `hermes_cli/plugins_dispatch.py`, `agent/turn_tool_round.py`
- **Missing:** the patch applied to the Hermes build in use, and the `post_tool_round_control` hook confirmed registered and firing
- **Evidence:** `patches/`, `hermes-core-snippets/` (full patched copies for review/rebase); no live-apply log in-repo
- **Impact:** without the hook, the headline "skip the next call" cannot happen at all — the plugin degrades to compaction only
- **Estimate:** 4–8 h; needs a live Hermes install and a restart

### 2. Live skip unverified (alpha blocker)
- **Goal:** G3
- **Landed:** `tests/test_integration_skip.py` asserts the 2-vs-1 call pattern for a deterministic successful tool round
- **Missing:** the same assertion on a real session with real TypeSafe scoring — mocked tests cannot show the thresholds (`goal_satisfied ≥ 0.90` etc.) behave on live judgments
- **Evidence:** `tests/test_integration_skip.py`; suite passes 65/65 (2026-10-14)
- **Impact:** blocks the central value claim; a mocked pass can hide threshold drift
- **Estimate:** 4–6 h; needs a `TYPESAFE_API_KEY`

### 3. Savings unmeasured (Measurement)
- **Goal:** G4
- **Landed:** telemetry writing (`plugins/jev-router/telemetry.py`, `LOGGING.md`) — skips, chars before/after, fail-open counts
- **Missing:** a real workload run with a before → after comparison written up
- **Evidence:** `README.md` (Telemetry section); no measured numbers in-repo
- **Impact:** "it saves money" is an intuition, not a number
- **Estimate:** 4–6 h

### 4. Fail-open only unit-tested (QA)
- **Goal:** G5
- **Landed:** timeout-bounded, fail-open dispatch; `tests/test_tool_calling_regression.py`
- **Missing:** a live exercise of the TypeSafe-down / no-key / validation-error paths inside a real turn
- **Evidence:** `hermes-core-snippets/plugins_dispatch.py`; unit tests pass
- **Impact:** the safety guarantee is the plugin's licence to exist; it must hold live
- **Estimate:** 2–4 h

### 5. Added latency unmeasured (Perf)
- **Goal:** target outcome (cost, not just tokens)
- **Landed:** preflight class + per-round Jev call
- **Missing:** wall-clock delta per tool round on a real session
- **Evidence:** `README.md` (Preflight goal class); no timing numbers
- **Impact:** savings in tokens could be paid back in latency
- **Estimate:** 3–5 h

### 6. Manual core patch (Maintenance)
- **Goal:** G2 (durability)
- **Landed:** patch + `hermes-core-snippets/` for rebasing
- **Missing:** an automated apply/verify step or a pinned Hermes version; upgrades re-open the manual step
- **Evidence:** `README.md` install step 1; `plugins/jev-router/plugin.yaml` `requires_core_patch`
- **Impact:** on every Hermes upgrade, skip silently stops working until re-patched
- **Estimate:** 4–8 h

### 7. Windows install not executed here (Docs/QA)
- **Goal:** installability
- **Landed:** `WINDOWS-INSTALL.md` paste-and-check guide
- **Missing:** an end-to-end execution of that guide during this review
- **Evidence:** `WINDOWS-INSTALL.md`
- **Impact:** doc drift is unlikely to be caught
- **Estimate:** 2–4 h

### 8. Autonomous plan experimental (Feature)
- **Goal:** future capability
- **Landed:** `plugins/jev-router/plan.py` behind `JEV_AUTONOMOUS_PLAN_ENABLED` (default false); `tests/test_plan.py` passes
- **Missing:** live exercise; decision on whether it ships
- **Evidence:** `plugin.yaml` (`autonomous_plan_enabled: false`), `tests/test_plan.py`
- **Impact:** none today; keep off until proven
- **Estimate:** 4–8 h

## Revision history
| Date | Change | Summary |
|---|---|---|
| 2026-10-14 | major | First write: 8 gaps ranked; 2 alpha blockers (#1 patch applies + hook fires, #2 live skip verified) |
