# Legacy Simulator Behavior Baseline

This guide captures the current, observable behavior of the Colab-exported
`DevOpsSimulation` workflow in `legacy/openai19pm10.py`. Treat it as the
baseline contract we must preserve while planning the major refactor. Until a
formal modernization plan is approved, keep the Python sources read-only and use
this document to understand how the simulator currently operates.

## Scope and intent

- **Audience:** anyone auditing the legacy implementation, writing regression
  tests, or preparing a refactor proposal.
- **Focus:** document real behavior, not desired behavior. All notes were pulled
  from the existing code paths and accompanying artifacts.
- **Change policy:** documentation under `legacy/documentation/**` may evolve to
  clarify behavior; code under `legacy/` stays frozen until the refactor plan is
  signed off.

## High-level flow

### Mode dispatch

`DevOpsSimulation.run_simulation()` is the single entry point. It inspects
`sim_config["backwardCompatibilityMode"]` and branches into one of two drivers:

- **Standard / backward-compatible mode** (`run_backcompat_iterations`)
  - Executes the configured macroiterations sequentially.
  - Forces exactly **one miniiteration per phase** and suppresses mini-level
    lessons learned.
  - Defers any issue or lesson feedback until the next macroiteration begins.
- **Enhanced phase/mini flow** (`run_phase_based_simulation`)
  - Still iterates by macroiteration but allows each phase to run multiple
    miniiterations as dictated by `phases[*]["miniIterations"]`.
  - Propagates detected issues immediately into the next miniiteration and also
    forward to subsequent macroiterations.
  - Records lessons at both miniiteration and macroiteration boundaries.

### Macroiteration cadence

Regardless of mode, macroiterations are the outermost loop and share common
bookkeeping:

1. Snapshot carry-over issues (`self._carryover_open_ids`) and phase counters
   before starting a macroiteration.
2. Inject persisted issue and lesson state (`self.feedback_storage`,
   `self.partial_lessons`) when entering the macroiteration.
3. Call `_write_client_spec_once()` so the student DOCX renders the client brief
   only once per run, then insert `Macro Iteration NN` and `Phase:` headings
   exclusively when `_is_final_macro()` returns `True`.
4. After all phases complete, call `compute_iteration_metrics()` and
   `write_iteration_reports()` to append macro-level CSV rows and update the
   DOCX summary.

### Miniiteration lifecycle (enhanced mode)

Enhanced mode expands each phase into one or more miniiterations:

1. `run_phase_based_simulation()` loops phases, and for each phase iterates
   `mini_iter_idx` from `1..miniIterations`.
2. For every miniiteration, `run_phase_single_pass()` generates artifacts,
   applies the latest `IssueTracker` findings, and updates counters.
3. `finalize_mini_iteration()` timestamps the pass, records lesson text, and
   emits mini-level metric rows before control returns to the phase loop.
4. At the end of the phase, counters roll up into the macro totals so the next
   phase sees accumulated context.

## Inputs and persistence

- **Required artifacts**
  - `client_specifications.docx` supplies the customer brief seeded into each
    run.
  - `artifact_data.json` defines tables, dependency ordering, trace columns, and
    review prompts consumed during document generation.
- **Optional configuration**
  - `simulation_config.json` overrides macro/mini counts, phase ordering, and
    `backwardCompatibilityMode`. The driver synthesizes a local fallback phase
    when none is supplied, but helpers like `get_lessons_for_new_mini()` still
    rely on `self.sim_config["phases"]`. In practice you must provide a
    configuration that enumerates every phase (even in standard mode) so the
    downstream lookups succeed across macroiterations.
- **State hand-off**
  - Issue and lesson histories persist via JSON fragments maintained in
    `self.feedback_storage` and `self.partial_lessons` so future iterations can
    replay past findings.
  - Global indices (`build_global_phase_map`, `build_global_mini_index`) map
    each phase/mini pair to ordered counters, enabling latency metrics even when
    configuration changes.

## Outputs and generated artifacts

Each execution regenerates the full artifact set in the repository root:

- `iteration_metrics.csv` is recreated (header plus fresh contents) by
  `setup_documentation()` before any iteration work begins.
- `mini_iteration_summary_metrics.csv` and `iteration_summary_metrics.csv`
  receive a header the first time they are missing and are appended to after
  each miniiteration or macroiteration is finalized.
- `issue_lifecycle.csv` is emitted during finalization to capture per-gap
  timelines based on `issue_first_seen_iteration`.
- `output_multi_iter.docx` (student) and `teacher_review_output_multi_iter.docx`
  (teacher) reflect the headings and lesson summaries accumulated through the
  run.
- `colab_console_output.txt` mirrors every `print` call because the notebook
  bootstrap replaces `sys.stdout` with `DualLogger` at import time.

Metric writers live inside `setup_documentation()`, `finalize_mini_iteration()`,
`compute_iteration_metrics()`, and `write_iteration_reports()`. Only the
per-artifact CSV is reset for each execution; the summary files append rows, so
regression checks should confirm both header shape and row ordering remain
stable.

## Operational checklist

Use this checklist to validate the simulator before and after the refactor:

1. Upload `openai19pm10.py` to Colab and ensure bootstrap cells install
   `python-docx`, `openai>=1.77`, and `nltk`.
2. Stage the three primary inputs (`client_specifications.docx`,
   `artifact_data.json`, optional `simulation_config.json`).
3. Run `orchestrate_process()` (wrapper around `DevOpsSimulation.run_simulation`) and
   confirm DOCX and CSV outputs appear with fresh timestamps.
4. Verify macroiteration CSV rows increment monotonically and that enhanced mode
   adds miniiteration rows between macro rows.
5. Capture the console output (`colab_console_output - *.txt`) for traceability
   and store alongside generated artifacts.

## Known limitations to preserve

- Aggregate reports favor macroiteration totals, making enhanced mode appear to
  generate more issues due to multiple miniiterations per macroiteration.
- Metrics do not normalize by the number of miniiterations, so comparing modes
  requires manual adjustment.
- State persistence optimizes whole-macro replays, making it tedious to inspect
  a single miniiteration in isolation.
- The code never sets `self.backwardCompatibilityMode`, so helpers such as
  `summarize_lessons_learned_for_mini_iteration()` always execute the enhanced
  aggregation branch even when `run_backcompat_iterations()` is active. Lesson
  rollups therefore operate on the current phase/mini scope in both modes.
- Because fallback phases are not written back into `self.sim_config`, running
  multiple macroiterations without an explicit `phases` array will raise lookups
  when helpers attempt to access the "last" phase. Always ship a phase list in
  `simulation_config.json` to avoid that crash.

Documenting these constraints ensures future changes call out intentional
behavior shifts instead of silently altering the baseline.

## Refactor guardrails

- Do **not** change `legacy/openai19pm10.py` until a signed-off refactor plan is
  in place. Update this document instead if new observations surface.
- Any future refactor proposal should cite this baseline and explain how new
  behavior diverges (or intentionally stays aligned).
- Regression tests derived from this document must continue to pass before code
  rewrites merge into `main`.

