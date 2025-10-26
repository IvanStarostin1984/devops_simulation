# Legacy simulator configuration surfaces

## Purpose

This reference explains every configuration input the legacy Colab notebook
consumes so refactors can reproduce the current branching logic without reading
the full notebook. It focuses on the JSON payload loaded from
`simulation_config.json` and the defaults applied when that file is absent.

## Loading sequence and defaults

1. `orchestrate_process()` always loads `artifact_data.json`, then seeds a
default configuration with `backwardCompatibilityMode = true`,
`macroIterations = 1`, and an empty `phases` list before attempting to read
`simulation_config.json`.【F:legacy/openai19pm10.py†L7795-L7807】
2. `DevOpsSimulation.__init__()` repeats the same defaults when the caller
passes `sim_config=None`, caches the final mini-iteration count per phase, and
creates helper indexes that depend on the configured phase order.【F:legacy/openai19pm10.py†L2272-L2343】
3. Because those indexes are built once at construction time, any edits to the
configuration after instantiation will be ignored unless the object is rebuilt.

## Top-level toggles

### `backwardCompatibilityMode`

- Controls which driver `run_simulation()` executes: `true` calls the
single-mini back-compat engine; `false` activates the phase/mini loop that
allows multiple minis per phase.【F:legacy/openai19pm10.py†L3056-L3268】
- Downstream helpers do **not** read the config directly. Instead they consult
`getattr(self, "backwardCompatibilityMode", False)`, so unless another caller
assigns the attribute on the simulation instance, the enhanced-mode branch stays
active even when the config flag is `true`.【F:legacy/openai19pm10.py†L2272-L2307】【F:legacy/openai19pm10.py†L6578-L6637】

### `macroIterations`

- Sets the outer loop for both drivers via `self.sim_config.get("macroIterations", 1)`.
- `_is_final_macro()` compares the current macro index against the configured
maximum to decide when to emit document headings or guard student output so only
the final macro renders client specs and phase headings.【F:legacy/openai19pm10.py†L2768-L3177】
- Global phase indexes also multiply the configured macro count by the number of
phases, so increasing `macroIterations` expands the lookup tables used for
latency calculations.【F:legacy/openai19pm10.py†L2349-L2396】

## Phase definitions (`phases` array)

Each phase entry controls iteration structure and artifact routing.

### `phaseName`

- Required string used everywhere the orchestrator labels phase-specific
outputs, including CSV rows, DOCX headings, and teacher review notes.【F:legacy/openai19pm10.py†L3092-L3491】
- `build_phase_index_map()` records the phase order so lesson summaries and
issue timelines can sort by the configured sequence.【F:legacy/openai19pm10.py†L2562-L2568】

### `miniIterations`

- Optional integer that defaults to `1` per phase. Phase-based mode loops from
`1..miniIterations`, while back-compat mode still forces a single mini per phase
but uses the configured count when calculating previous/next pointers.【F:legacy/openai19pm10.py†L2422-L3491】
- `_phase_final_mini` caches the last mini for each phase and lets the
orchestrator decide whether the current pass should print the final artifact
into the student DOCX.【F:legacy/openai19pm10.py†L2272-L2777】

### `tableList`

- Optional list of artifact indices assigned to the phase. The simulator
filters `artifact_data["tables"]` against this list so only the nominated
artifacts run in that phase; an empty list means no artifacts execute for that
phase.【F:legacy/openai19pm10.py†L3092-L3491】
- When the configuration omits `tableList` entirely, the phase still exists but
no artifacts will be processed because the filter never matches any entries.【F:legacy/openai19pm10.py†L3445-L3491】

## Fallbacks and failure modes

- If the configuration omits the `phases` array, back-compat mode injects a
synthetic `[{"phaseName": "Backcompat"}]` list so the loop runs, but the phase
has no `tableList` so no artifacts change unless the config file provides real
assignments.【F:legacy/openai19pm10.py†L3090-L3141】
- Phase-based mode tolerates an empty `phases` list but reports the phase as
`"NoPhases"` in metrics, leaving most artifact work undone. Provide the real
phase list to keep metrics meaningful.【F:legacy/openai19pm10.py†L3208-L3360】
- Helpers that compute carry-over state rely on the phase order captured during
construction; if the configuration is edited mid-run those caches are not
recalculated. Recreate the simulation object after changing the config to avoid
misaligned indexes.【F:legacy/openai19pm10.py†L2272-L2396】
- Because the enhanced lesson summarizer always follows the `getattr` fallback,
lesson aggregation treats every run as enhanced mode even when
`backwardCompatibilityMode` is `true`. Preserve this quirk until the refactor
changes how mode-specific storage works.【F:legacy/openai19pm10.py†L6578-L6637】

## Practical checklist

1. Always ship a `simulation_config.json` that enumerates every phase, its
`tableList`, and the desired `miniIterations` counts before running the legacy
notebook.
2. Set `macroIterations` explicitly to guarantee document headings and global
indexes line up with test expectations.
3. If you need to simulate standard mode behavior, be aware that lesson reports
will still aggregate per-mini data because the helper ignores the config flag;
compare results using the CSV append patterns instead of the lesson summaries.
