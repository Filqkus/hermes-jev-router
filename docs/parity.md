# jev-router: parity (breadth vs ready)

> **Last reviewed:** 2026-10-14 · **Last updated:** 2026-10-14 · **Change:** major (first write)

Breadth = the feature exists. Ready = it works for real work today, graded against [goals.md](goals.md) IDs, err low. Weights are the share of real value in the token-savings outcome; they sum to 100. Everything here is **estimated** unless a row says measured.

## Method
- **Inventory (measured, 2026-10-14):** file tree of `plugins/jev-router/`, `patches/`, `tests/`; `plugins/jev-router/plugin.yaml` (8 hooks + `requires_core_patch`); plugin code 4929 lines, tests 1446 lines (14 test modules).
- **Test evidence (measured, 2026-10-14):** `pytest tests/ -q` → **65 passed in 0.41s** on a fresh `uv` venv (CPython 3.14.0, pytest + pydantic installed); `--collect-only` → 65 tests collected. All TypeSafe calls are mocked — no network.
- **Breadth:** 8/8 features exist (100%).
- **Ready:** per-feature share of goal criteria with a *run check* and linked evidence. Offline unit coverage earns partial credit (the mechanism is exercised, but not on real input); only the live checks push a feature to "met".
- **Weights:** share of real value; the sum is printed and divides by the actual total (100).

## Units
| # | Feature | Weight | Breadth | Ready | Contribution (w × ready/100) |
|---|---|---|---|---|---|
| 1 | Tool-result compaction (`transform_tool_result`) | 8 | exists | 45% | 3.6 |
| 2 | Core patch + `post_tool_round_control` hook | 18 | exists | 10% (unapplied) | 1.8 |
| 3 | Skip next main-model call (`post_tool_round_control`) | 20 | exists | 30% (mocked only) | 6.0 |
| 4 | Measured token/cost savings | 18 | telemetry exists | 5% | 0.9 |
| 5 | Fail-open (timeout / no key / error) | 12 | exists | 45% | 5.4 |
| 6 | Secret redaction before TypeSafe | 8 | exists | 70% | 5.6 |
| 7 | No tool-calling regression | 10 | exists | 45% | 4.5 |
| 8 | Duplicate observational tool guard | 6 | exists | 45% | 2.7 |
| | **Weighted ready** | **100** | **100%** | | **≈ 30.5%** |

Two readings of Ready are reported: **~30%** above counts offline unit coverage at partial credit (mechanism exercised, real input not). The ROADMAP states **~20%**, which credits only runs on real input — the stricter, err-low reading.

### Audiences
| Audience | Value | Kind |
|---|---|---|
| Builders (can clone, install, and run the tests) | **~85%** | measured: suite green on a fresh venv |
| A Hermes user depending on it today | **~5%** | estimated: patch unapplied, hook unproven, fail-open not live-tested |

## What was not checked
- No live Hermes session and no live TypeSafe call were made (the reviewer must not modify the running Hermes install).
- The core patch was **not** applied anywhere; `WINDOWS-INSTALL.md` was not executed.
- No token, cost, or latency measurement was taken.
- `upstream/main` vs `origin/main` was compared (identical, `14eb5c8`); branch `origin/grokbot-use` was not reviewed.

## Revision history
| Date | Change | Summary |
|---|---|---|
| 2026-10-14 | major | First write: 8 units weighted to 100; breadth 100%, weighted ready ~30% (offline-inclusive) / ~20% (live-only, quoted on ROADMAP); suite 65/65 |
