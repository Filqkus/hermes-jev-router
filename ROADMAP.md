# jev-router Roadmap

**Stage: pre-alpha** · **Build: pre-alpha-0.1.8** (build = `git rev-list --count HEAD` on this branch tip) · next: alpha, ~20–35 h away (one agent; ~20–25 h wall-clock with a live Hermes + TypeSafe environment)

> **Last reviewed:** 2026-10-14 · **Last updated:** 2026-10-14 · **Change:** major (first progress-docs baseline: stage assigned, goals + gaps written, offline suite measured) · **Target:** in a real Hermes session, mechanical/verification tool rounds finish without a second main-model call and large tool outputs enter history compacted, with measured token/cost drop and fail-open

Stage gates: **pre-alpha** = parts exist, no e2e, readiness <25%; **alpha** = e2e acceptance check passes on real input; **beta** = all features in and readiness >75%; **release** = full goals.md acceptance gate + stable period with no blockers. See the progress-docs SKILL.

hermes-jev-router (plugin name `jev-router`, v1.0.0) is a Hermes Agent plugin: a cheap, typed Jev/TypeSafe judgment layer that decides WHETHER the main model is needed, while Python decides HOW (keep/drop chunks, block duplicate tools, render a deterministic finish). It ships a small generic core patch (`post_tool_round_control`) plus the plugin and an offline test suite. This page is the summary — to pick work, start with [gaps.md](docs/gaps.md).

## Headline numbers
| | Value | Kind |
|---|---|---|
| Breadth (documented features present) | **8/8 (100%)** | measured: `plugins/jev-router/plugin.yaml` hooks + file tree, 2026-10-14 |
| Ready for real work | **~20%** | estimated, [parity](docs/parity.md) (weighted per feature) |
| Offline test suite | **65/65 pass** | measured: `pytest tests/ -q` → 65 passed in 0.41s (Python 3.14.0, 2026-10-14) |
| Live token/cost savings | **not measured** | unchecked — no real-workload run |
| Remaining to alpha | **~20–35 h** | estimated |

## Stage
**pre-alpha** — every part exists (compaction, duplicate guard, skip, renderer, preflight, telemetry, fail-open, redaction) and the offline suite is fully green (65/65, mocked TypeSafe), but there is **no end-to-end run on real input**: the core patch has not been shown to apply to the current Hermes build, the `post_tool_round_control` hook has not been seen firing in a live session, and no token or cost saving has been measured. Readiness (~20%) is under the pre-alpha ceiling (25%). The next stage, alpha, is exactly those live checks (G2 + G3 + G5).

## Gate: alpha
Gate criteria come from [docs/goals.md](docs/goals.md): **alpha = G2 + G3 + G5** (patch applies + live skip works + fail-open holds).

| Feature | Works end to end? (goal IDs) | Evidence | Hours to pass |
|---|---|---|---|
| Core patch (`post_tool_round_control`) | no — **G2 unchecked** | `patches/hermes-post-tool-round-control.patch` touches 3 files; never applied to a live Hermes here | 4–8 |
| Skip next main-model call | no — **G3 unchecked** | `tests/test_integration_skip.py` asserts the 2-vs-1 pattern, but mocked | (in #1) |
| Fail-open | unit-only — **G5 partial** | `tests/test_tool_calling_regression.py`, `test_round_control.py` (pass) | 2–4 |
| Tool-result compaction | unit-only — **G1/G6 partial** | `tests/test_compactor.py`, `test_redact.py` (pass) | 4–6 |
| Duplicate guard | unit-only — **G8 partial** | `tests/test_duplicate.py` (passes) | 2–4 |

**Verdict: alpha gate NOT passed** — no live evidence yet. All five rows are offline-only.

## Milestones
| Milestone | Status | Left | Hours |
|---|---|---|---|
| M0 Plugin extracted + README/WINDOWS-INSTALL | done | — | — |
| M1 Plugin + 3-file core patch + offline suite | done (2026-10) | — | — |
| M2 Analyzable decision logs (`decisions.jsonl`, richer fields) | done (`7d174e2`, `18cdb6d`) | — | — |
| M3 Live verification on a real Hermes session + real TypeSafe | not started | everything in the alpha gate | 20–35 |
| M4 Measured savings report (before → after on a real workload) | not started | telemetry run + write-up | 4–6 |

## Current focus
1. Apply the core patch to the current Hermes build and confirm `post_tool_round_control` registers and fires: 4–8 h
2. Run one real Hermes turn with `TYPESAFE_API_KEY` and confirm the skip (1 vs 2 main-model calls): 4–6 h
3. Exercise fail-open live (TypeSafe down / timeout) and record `telemetry.jsonl` before → after: 4–6 h

## Progress log
- **2026-10 (early)** · Plugin + core patch + offline suite landed (`ab1f660`…`9118b9a`).
- **2026-10 (mid)** · Tool-calling regression fixes; over-eager fast-path and fail-closed `pre_tool_call` fixed (`9118b9a`, `93db6a5`).
- **2026-10 (mid)** · Analyzable decision logs for every continue/finish path (`18cdb6d`), richer fields (`7d174e2`).
- **2026-10 (latest)** · `14eb5c8` — fix `status_failure` false positive on terminal verify (`"error": null`).
- **2026-10-14** · Offline suite re-run as part of this review: 65 passed (`pytest tests/ -q`).

## Notes
- **Fork hygiene (measured 2026-10-14):** `Filqkus/hermes-jev-router` main == `rsdkrasen/hermes-jev-router` `upstream/main` (`14eb5c8`, 7 commits, 0 ahead / 0 behind); one other branch, `origin/grokbot-use` (`56d7b72`).
- **Proposed name (proposal only — no rename performed):** `hermes-jev-router` is accurate but ambiguous (it is a Hermes *plugin*, not a router service). If renamed, `hermes-typesafe-router` reads better. Renaming is owned by the upstream author — not done here.
- The local `Desktop\hermes-jev-router-repo` copy has a wrong `origin` (points at `jarvis-orchestrator`); this branch was pushed from a fresh clone, not from that copy.

## Revision history
| Date | Change | Stage/Build | Summary (before → after numbers) |
|---|---|---|---|
| 2026-10-14 | major | pre-alpha / pre-alpha-0.1.8 | First baseline. No prior progress docs. Assigned stage **pre-alpha**; breadth 8/8 (100%), ready ~20%, offline suite 65/65 pass; goals G1–G8 and 8 ranked gaps written; build 8 = `git rev-list --count` on `docs/progress-v1` tip |
