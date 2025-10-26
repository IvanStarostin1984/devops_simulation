# Legacy Simulator Flow Reference

This document records the as-is control flow of `legacy/openai19pm10.py` so that
future refactors can preserve the current behavior. It explains how the
Colab-focused notebook boots, how `DevOpsSimulation` orchestrates
macroiterations, phases, and miniiterations, and how supporting helpers persist
state and metrics between passes.

## Outcomes and acceptance criteria

- Readers can follow the chronological order of notebook bootstrap,
  configuration loading, and simulation execution without inspecting the code
  directly.
- Every major orchestrator hook (`run_simulation`, `run_backcompat_iterations`,
  `run_phase_based_simulation`, `run_phase_single_pass`) is summarized with its
  responsibilities and exit conditions.
- State-propagation helpers (`apply_global_unresolved_issues`,
  `update_global_unresolved_issues`, `finalize_mini_iteration`, and
  `compute_iteration_metrics`) are described alongside the CSV and DOCX writers
  they feed.

## Execution entry points

1. **Stdout mirroring.** `DualLogger` replaces `sys.stdout` so every `print`
   statement is duplicated to `colab_console_output.txt`. The file is reopened in
   write mode for each run, matching the Colab workflow.【F:legacy/openai19pm10.py†L10-L37】
2. **Environment bootstrap.** The notebook cell installs `python-docx`,
   `openai>=1.77`, and `nltk`, then calls `ensure_nltk_resources()` to provision
   tokenizers and taggers under `/root/nltk_data`. This step is required before
   any simulation code runs, ensuring later LLM prompts and NLP metrics have the
   expected dependencies.【F:legacy/openai19pm10.py†L41-L111】
3. **File ingestion.** `orchestrate_process()` prompts the user to upload the
   client specification (TXT or DOCX), `artifact_data.json`, and optionally
   `simulation_config.json`. The uploaded spec is exposed globally via
   `builtins.CLIENT_SPEC` so `DevOpsSimulation.__init__()` can embed it in the
   student report.【F:legacy/openai19pm10.py†L7763-L7845】
4. **Simulation start.** `orchestrate_process()` instantiates
   `DevOpsSimulation(artifact_data, sim_config)` and immediately calls
   `run_simulation()`. Once execution finishes it downloads the regenerated CSV
   and DOCX outputs for local archival.【F:legacy/openai19pm10.py†L7834-L7848】

## Initialization and global state

`DevOpsSimulation.__init__()` performs the following setup before any iteration
logic executes:

- Calls `setup_documentation()` to create new student (`output_multi_iter.docx`)
  and teacher (`teacher_review_output_multi_iter.docx`) documents and to recreate
  `iteration_metrics.csv` with the full artifact-level header. It also ensures
  `iteration_summary_metrics.csv` and `mini_iteration_summary_metrics.csv` exist
  with the expected columns.【F:legacy/openai19pm10.py†L2273-L2338】【F:legacy/openai19pm10.py†L2781-L2889】
- Stores the provided configuration, derives the final mini-iteration count per
  phase (`_phase_final_mini`), and seeds data structures for issue tracking,
  lesson storage, artifact versions, and carry-over calculations. Many of these
  structures—such as `feedback_storage`, `partial_lessons`,
  `global_open_issue_ids`, and `_carryover_open_ids`—act as the single source of
  truth when later helpers reconcile state between iterations.【F:legacy/openai19pm10.py†L2280-L2334】
- Builds reusable indices (`build_phase_index_map`, `build_global_mini_index`,
  `build_global_phase_map`) that translate macro/phase/mini positions into
  globally comparable counters. These indices let the simulator tag every gap and
  lesson with stable identifiers regardless of the execution branch.【F:legacy/openai19pm10.py†L2336-L2418】【F:legacy/openai19pm10.py†L2420-L2471】
- Initializes `IssueTracker` with a reference back to the simulation so teacher
  reviews can ask for the global mini index during deduplication and carry-over
  checks.【F:legacy/openai19pm10.py†L2330-L2342】

## Mode dispatch (`run_simulation`)

`run_simulation()` chooses between the standard and enhanced flows by inspecting
`sim_config['backwardCompatibilityMode']`:

- When `True`, it announces "backward compatibility mode" and calls
  `run_backcompat_iterations()` to execute exactly one miniiteration per phase
  while still reusing the shared helpers that update metrics, issue state, and
  deliverables.【F:legacy/openai19pm10.py†L3056-L3089】
- When `False`, it routes to `run_phase_based_simulation()` so each configured
  phase can run multiple miniiterations per macroiteration with the same
  artifact-processing pipeline.【F:legacy/openai19pm10.py†L3060-L3064】【F:legacy/openai19pm10.py†L3197-L3356】

Both modes rely on the same setup, artifact processing, and teardown helpers, so
metrics and documents stay consistent regardless of the branching choice.

## Standard mode (`run_backcompat_iterations`)

The backward-compatibility driver preserves the historical single-mini behavior
while wiring in the enhanced metrics pipeline:

1. **Macroiteration loop.** For each macroiteration it snapshots the set of open
   gap IDs into `_carryover_open_ids_start` and `_carryover_by_type_start`. These
   frozen snapshots provide the carried-over counts that
   `compute_iteration_metrics()` later validates (identity ID-1).【F:legacy/openai19pm10.py†L3094-L3142】
2. **Heading hygiene.** `_write_client_spec_once()` injects the client
   specification at the top of the student DOCX. Phase and macro headings are
   only emitted in the final macroiteration to mirror the legacy final-report
   layout.【F:legacy/openai19pm10.py†L3144-L3167】【F:legacy/openai19pm10.py†L2766-L2779】
3. **Single miniiteration per phase.** Each phase runs `run_phase_single_pass()`
   once, followed by `finalize_mini_iteration()` and `roll_up_counters()`. Even
   though lessons are blank in this mode, the call chain updates the same CSVs,
   state caches, and teacher-doc bullets for parity with the enhanced path.【F:legacy/openai19pm10.py†L3169-L3243】
4. **Macro summary.** After all phases finish, the driver collects lessons,
   invokes `compute_iteration_metrics()` to append macro-level CSV rows, and
   calls `write_iteration_reports()` to refresh DOCX summaries before generating
   the final artifacts.【F:legacy/openai19pm10.py†L3245-L3279】

## Enhanced mode (`run_phase_based_simulation`)

The enhanced driver extends the macro→phase→mini loops to support multiple
miniiterations per phase:

1. **Macro setup.** Each macroiteration begins by snapshotting carry-over state,
   emitting student and teacher headings, and initializing counters plus baseline
   row totals for churn calculations.【F:legacy/openai19pm10.py†L3207-L3243】
2. **Phase loop.** For every configured phase the simulator records headings and
   iterates the requested miniiteration count. Before each mini pass it logs a
   stderr breadcrumb (`retrospective_problem|LOOP|…`) used by downstream lesson
   tooling.【F:legacy/openai19pm10.py†L3245-L3294】
3. **Miniiteration loop.** Each miniiteration records start/end timestamps,
   delegates artifact handling to `run_phase_single_pass()`, finalizes via
   `finalize_mini_iteration()`, and rolls counters into the macro accumulator.
   Teacher-doc bullets summarize lessons per mini, and `partial_lessons` retains
   the text for later macro rollups.【F:legacy/openai19pm10.py†L3295-L3339】
4. **Macro close-out.** After the phase loop completes, the driver ensures row
   totals are populated, computes macro metrics, writes reports, and consolidates
   lessons learned into the teacher document before saving the final artifacts.
   The same invariants enforced in standard mode guard the carried-over counts
   and open-issue totals.【F:legacy/openai19pm10.py†L3340-L3366】【F:legacy/openai19pm10.py†L3368-L3383】

## Artifact processing (`run_phase_single_pass`)

`run_phase_single_pass()` is the shared workhorse for both modes:

- **Global carry-over.** `apply_global_unresolved_issues()` injects placeholder
  issues for any still-open gap IDs so every miniiteration starts with an
  accurate backlog snapshot. The function writes these placeholders into the
  `_GLOBAL_CARRY_` artifact and converts lingering `NewIssue` statuses to
  `KnownIssue` to avoid double-counting.【F:legacy/openai19pm10.py†L2679-L2758】
- **Open-set snapshot.** Before processing artifacts it records the set of open
  issues in `pre_mini_open_issues[(macro, phase, mini)]`, allowing
  `finalize_mini_iteration()` to detect false closures later.【F:legacy/openai19pm10.py†L3405-L3438】
- **Artifact loop.** For each table mapped to the active phase it calls
  `process_single_artifact()`, merges the resulting issues into
  `feedback_storage`, updates `global_open_issue_ids`, and accumulates mini-level
  metrics. Student DOCX tables are only rendered for the final miniiteration of
  the final macroiteration, preserving the legacy "final output only" rule. The
  teacher DOCX records every artifact and associated issues regardless of mode.【F:legacy/openai19pm10.py†L3439-L3505】【F:legacy/openai19pm10.py†L3506-L3547】
- **Post-loop bookkeeping.** After processing artifacts it tracks which gap IDs
  were newly discovered and which remained open, then calls
  `finalize_mini_iteration()` to persist metrics and lessons for the pass.【F:legacy/openai19pm10.py†L3549-L3597】

## State propagation and metrics

Several helpers keep iteration state consistent between passes:

- **`update_global_unresolved_issues()`** scans the freshly collected issues for
  the current miniiteration and maintains `global_open_issue_ids`, removing gaps
  that have transitioned to `Resolved`. This set drives both carry-over snapshots
  and density calculations.【F:legacy/openai19pm10.py†L2650-L2677】
- **`apply_global_unresolved_issues()`** (see above) guarantees the upcoming
  miniiteration sees every unresolved gap, even if the artifact that owns it is
  not scheduled in the current phase.【F:legacy/openai19pm10.py†L2679-L2758】
- **`finalize_mini_iteration()`** validates that no gap disappears without a
  matching `resolvedIn` tuple, computes mini-level metrics via
  `compute_mini_iteration_metrics()`, writes a row to
  `mini_iteration_summary_metrics.csv`, updates `partial_lessons`, and refreshes
  the authoritative caches (`new_ids_in_mini`, `open_ids_at_end_of_mini`).【F:legacy/openai19pm10.py†L5157-L5336】
- **`roll_up_counters()`** aggregates text, churn, and defect counters from the
  mini accumulator into the macro accumulator. Although macro-level metrics later
  recompute table sizes from the final artifacts, these aggregates keep the mini
  data available for density calculations and historical comparisons.【F:legacy/openai19pm10.py†L3366-L3402】
- **`compute_iteration_metrics()`** enforces the carried-over/open-issue
  identities, classifies every gap by type and status, normalizes churn and
  density metrics, and updates rolling averages. The resulting dictionary powers
  `write_iteration_reports()` and the macro-level CSV output.【F:legacy/openai19pm10.py†L5868-L6089】
- **`generate_final_summary_doc()`** saves both DOCX files, closes the artifact
  metrics CSV handle, and regenerates `issue_lifecycle.csv` so the runtime trace
  matches the updated issue states.【F:legacy/openai19pm10.py†L5458-L5485】

Together these helpers ensure that every miniiteration begins with the correct
backlog, every artifact update is captured in the CSV and DOCX artifacts, and the
macro-level reports remain internally consistent across modes.

## Deliverables and side effects

Each simulation run overwrites or appends the following artifacts in place:

- `output_multi_iter.docx` — student-facing narrative produced after the final
  macroiteration.
- `teacher_review_output_multi_iter.docx` — teacher review log updated after
  every miniiteration.
- `iteration_metrics.csv` — recreated per run with one row per artifact per
  miniiteration.
- `mini_iteration_summary_metrics.csv` — append-only log of miniiteration
  summaries with actual resolution counts.
- `iteration_summary_metrics.csv` — append-only macro-level rollups validated by
  the ID-1/ID-2 invariants.
- `issue_lifecycle.csv` — regenerated at the end of each run based on the latest
  status transitions.【F:legacy/openai19pm10.py†L5458-L5485】

By documenting the orchestration, state handling, and artifact writers here, we
can refactor the simulation with confidence that any new implementation maintains
parity with the established behavior.
