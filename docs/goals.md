# jev-router: goals (what good looks like)

> **Last reviewed:** 2026-10-14 · **Last updated:** 2026-10-14 · **Change:** major (first write) · **Owner:** Krasen Hristov / TypeSafe (upstream author); this fork's operator is Filip

Ready scores in [parity.md](parity.md) and the gate in [ROADMAP](../ROADMAP.md) are measured against the criteria below. Cite them by ID (G1, G2…). A criterion only counts as met when its check has been run and the evidence is linked.

## Target outcome
In a real Hermes session, mechanical/verification tool rounds finish without a second main-model API call and large tool outputs enter history compacted, so the main model is paid for less often — with the plugin failing open so Hermes never depends on it for correctness.

## Success criteria
| ID | Criterion (checkable) | How to verify | Evidence when met |
|---|---|---|---|
| G1 | Tool-result compaction keeps only relevant original chunks (never a summary; diagnostics preserved) and only above `JEV_COMPACTION_MIN_CHARS` | `pytest tests/test_compactor.py`; live chars-before/after in `telemetry.jsonl` | unit tests pass (2026-10-14); live unchecked — **partial** |
| G2 | The core patch applies cleanly to the current Hermes build and registers the `post_tool_round_control` hook | `patch -p1 < patches/hermes-post-tool-round-control.patch` on a Hermes checkout; hook visible in logs | none — **unchecked** |
| G3 | On a deterministic successful tool round in a real Hermes session, the next main-model call is **skipped** (1 call vs 2 baseline) | live session with `TYPESAFE_API_KEY`; count provider calls; `tests/test_integration_skip.py` is the mocked proxy | mocked test passes; live none — **partial (offline only)** |
| G4 | A measured token and cost reduction on a real workload | before → after comparison from `telemetry.jsonl` (skips, chars before/after) | none — **unchecked** |
| G5 | Fail-open: TypeSafe down / no key / validation error / timeout → Hermes behaves exactly as without the plugin | kill the TypeSafe path live; run a turn; confirm normal 2-call behaviour; `tests/test_tool_calling_regression.py` | unit tests pass; live unchecked — **partial** |
| G6 | Secrets are redacted before anything is sent to TypeSafe (`Authorization`, `*_TOKEN`, `*_SECRET`, `*_PASSWORD`, private keys) | `pytest tests/test_redact.py` | unit tests pass — **met (unit)** |
| G7 | No tool-calling regression: the plugin never blocks or corrupts a normal tool round | `pytest tests/test_tool_calling_regression.py`, `test_register.py`, `test_round_control.py`; a live turn | unit tests pass; live unchecked — **partial** |
| G8 | The duplicate observational tool guard blocks a repeated read-only call with no state change | `pytest tests/test_duplicate.py`; live `pre_tool_call` block in `decisions.jsonl` | unit tests pass — **met (unit)** |

## Quality bar
| Aspect | Good | Bad |
|---|---|---|
| Finish reply | A deterministic "Done …" line built from tool evidence | Generative prose invented when evidence is thin |
| Skip decision | Skipped only when `goal_satisfied ≥ 0.90` etc. hold, or a clear fast path | Skipping a turn the user asked to be explained |
| Compaction | Selected original slices, diagnostics kept | A rewritten/summarised result, or a truncated error |
| Dependency | Plugin can vanish and Hermes still works (fail-open) | Any correctness path that requires TypeSafe to answer |

## Non-goals
- Not another chat model — Jev scores small pieces of existing state (probabilities + enums).
- Does not replace Hermes approval / safety controls.
- Does not starve real generation: "research and explain the tradeoffs" always continues to the main model.
- Does not inject into the system prompt (cache-friendly preflight class only).

## Constraints
- Requires the Hermes core patch for skip; a manual re-apply on Hermes upgrades (no automated rebase).
- Requires `TYPESAFE_API_KEY` for live Jev; user plugins are opt-in (`hermes plugins enable jev-router`).
- Python version must match the Python Hermes uses; dependency `pydantic-ai-slim[typesafe]`.
- Windows-first install path (`WINDOWS-INSTALL.md`); the default Hermes profile must not be modified by a reviewer.

## Stage mapping
Map acceptance criteria to stages (cite IDs in ROADMAP Gate):
- **alpha:** the live minimum path — **G2 + G3 + G5** (patch applies, skip fires on a real turn, fail-open holds; rough is fine, builders only)
- **beta:** all feature IDs in (G1–G8) with live evidence **and** readiness >75%
- **release:** full acceptance gate below + a stable period with no blocking bugs and docs done

## Acceptance gate
Ready = 100% when the criteria below pass on real runs. Each criterion adds its weight:
| ID | Weight | Status (met / partial / not met / unchecked) |
|---|---|---|
| G1 | 8 | partial (unit only) |
| G2 | 18 | unchecked |
| G3 | 20 | partial (offline mocked only) |
| G4 | 18 | unchecked |
| G5 | 12 | partial (unit only) |
| G6 | 8 | met (unit) |
| G7 | 10 | partial (unit only) |
| G8 | 6 | met (unit) |

Weights sum to 100. Weighted readiness ≈ **20%** (only the unit-test-backed portion counts) — see [parity.md](parity.md) for the maths and method.

## Revision history
| Date | Change | Summary |
|---|---|---|
| 2026-10-14 | major | First goals write. G1–G8 defined with weights summing to 100; alpha gate set to G2+G3+G5 (live) |
