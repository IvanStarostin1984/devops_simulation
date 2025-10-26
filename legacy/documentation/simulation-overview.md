# DevOps Simulation Overview

This document summarizes how the simulation models standard and enhanced DevOps
processes, how work iterates through macroiterations and miniiterations, the
key data artifacts exchanged, and the metrics we expect to evaluate. Treat it
as the single source of truth for terminology and flow until the refactoring
effort introduces new adapters or interfaces. The observations below mirror the
current implementation in `legacy/openai19pm10.py`, specifically the
`DevOpsSimulation` class and its orchestrator helpers, so any future refactor
must preserve the described behavior (or explicitly document deviations).

## Implementation reference points

The orchestration logic lives inside `legacy/openai19pm10.py` and is concentrated
in the following entry points and helpers:

- `DevOpsSimulation.run_simulation()` loads `simulation_config.json` and routes to
  `run_backcompat_iterations()` or `run_phase_based_simulation()` depending on
  `backwardCompatibilityMode`.
- `run_backcompat_iterations()` executes one miniiteration per phase and still
  calls the same artifact, issue-tracking, and metrics helpers used by the
  enhanced path so parity remains intact.
- `run_phase_based_simulation()` iterates macroiterations, phases, and configured
  miniiterations, assembling per-mini lesson summaries and teacher-doc bullets
  along the way.
- `run_phase_single_pass()` drives artifact processing for the active
  macro/phase/mini tuple, calling `process_single_artifact()`,
  `update_global_unresolved_issues()`, `_is_latest_artifact_pass()`, and
  `add_json_as_docx_table()` to update documents only when the final pass is
  reached.
- `finalize_mini_iteration()` timestamps each mini pass, records the lessons
  surfaced by `summarize_lessons_learned_for_mini_iteration()`, and appends a
  row to `mini_iteration_summary_metrics.csv`.
- `compute_iteration_metrics()` and `write_iteration_reports()` append
  macroiteration summaries to `iteration_summary_metrics.csv` and update the
  DOCX deliverables.
- `setup_documentation()` recreates `iteration_metrics.csv` for the active run
  and ensures the summary CSV headers are present before any iterations begin.
- `apply_global_unresolved_issues()` and `update_global_unresolved_issues()` move
  gap state between miniiterations, while `self.feedback_storage` and
  `self.partial_lessons` retain JSON fragments for later macro passes.
- The notebook bootstrap replaces `sys.stdout` with `DualLogger`, mirroring every
  print call into `colab_console_output.txt` for traceability.

## Modes of Operation

### Standard DevOps Mode (Backward Compatibility = `true`)

- Enforces a single miniiteration within each macroiteration.
- Issues detected during artifact generation are queued and only surface at the
  start of the next macroiteration.
- Lessons learned are generated at the end of a macroiteration and become
  available to creators at the start of the subsequent macroiteration.
- Optimized for predictable, linear delivery that prioritizes stability over
  rapid feedback.

### Enhanced DevOps Mode (Backward Compatibility = `false`)

- Allows multiple miniiterations inside each macroiteration, enabling tighter
  feedback loops.
- Issues identified after each artifact/table generation feed into the very
  next miniiteration; macroiteration-level feedback also feeds forward between
  macroiterations.
- Lessons learned are produced at the end of *every* miniiteration as well as at
  macroiteration boundaries, so creators can adapt continuously.
- Aims to maximize quality and responsiveness, accepting additional iteration
  overhead in exchange for faster course correction.

## Mode dispatch and iteration loops

`run_simulation()` governs the overall mode selection. When
`backwardCompatibilityMode` is `true`, `run_backcompat_iterations()` walks each
macroiteration and, within it, loops the configured `phases` list exactly once
while still calling `run_phase_single_pass()` and `finalize_mini_iteration()` to
reuse the enhanced helpers. The lessons text is harvested through
`summarize_lessons_learned_for_mini_iteration()` even in standard mode, so the
per-mini aggregation path remains active.

When the flag is `false`, `run_phase_based_simulation()` becomes the driver. It
iterates macroiterations, then each `phaseName` entry, and finally
`miniIterations` counts, calling the same artifact pipeline for every pass.
Student DOCX headings (`Macro Iteration NN` and `Phase: …`) are emitted only when
`_is_final_macro()` returns `True`, matching the legacy layout where intermediate
iterations accumulate silently until the final macroiteration is rendered.

## Iteration Workflow

The simulation advances through repeating macroiterations that contain one or
more miniiterations. Each iteration pulls forward the latest issues and lessons
learned so that subsequent passes can react to new information quickly.

### Macroiterations

- Represent the major planning and delivery cycles for both modes.
- Snapshot carry-over issue identifiers via `self._carryover_open_ids` and
  `self._carryover_by_type` so latency metrics can distinguish reopened items.
- Hydrate persisted JSON state from `self.feedback_storage` and
  `self.partial_lessons` before the first phase executes, ensuring prior issues
  and lessons flow into the new macroiteration.
- Insert the client specification and headings through `_write_client_spec_once()`
  and `_is_final_macro()` to keep intermediate macroiterations quiet until the
  last pass is rendered to DOCX.
- Finalize by calling `compute_iteration_metrics()` and
  `write_iteration_reports()` to append macro-level CSV rows and refresh DOCX
  summaries.

### Miniiterations

- Cover the repeated artifact/table generation cycles within a macroiteration.
- Begin with `apply_global_unresolved_issues()` to rehydrate unresolved gaps and
  record a pre-pass snapshot in `self.pre_mini_open_issues` for churn analysis.
- Invoke `run_phase_single_pass()` to process each artifact, update the
  `IssueTracker`, and, when `_is_latest_artifact_pass()` returns `True`, write the
  student DOCX tables for the final miniiteration of the final macroiteration.
- Call `finalize_mini_iteration()` immediately after each pass to timestamp the
  execution, compute per-mini metrics, and capture the lessons surfaced by
  `summarize_lessons_learned_for_mini_iteration()`.
- Standard mode always executes one miniiteration per phase, so newly discovered
  issues surface in the *next* macroiteration. Enhanced mode may loop multiple
  miniiterations, allowing issues and lessons to feed the very next pass.

## Artifact processing and metrics

- `phase_tables` determine which entries from `artifact_data.json` flow into a
  miniiteration. `run_phase_single_pass()` fetches the relevant rows and routes
  each through `process_single_artifact()` to populate parsed tables and
  `IssueTracker` updates.
- `_is_latest_artifact_pass()` gates student DOCX writes so only the final
  miniiteration of the final macroiteration produces `level=3` headings and
  embedded tables. Every pass still adds teacher-document bullets via
  `record_artifact_issues_in_teacher_doc()`.
- `log_artifact_metrics_to_csv()` emits per-artifact statistics, including
  corrected "new issue" counts, diff metadata, and derived defect density, to the
  iteration metrics CSVs.
- `finalize_mini_iteration()` and `roll_up_counters()` aggregate per-mini
  counters into macro totals while persisting timestamps and lesson summaries.

## Data Inputs

- `legacy/artifact_data.json` provides prompt templates, table schemas,
  dependency ordering, and trace column mappings consumed by
  `run_phase_single_pass()` and downstream helpers.
- `legacy/simulation_config.json` toggles backward compatibility mode and sets
  macro/mini iteration counts. Although the code synthesizes default phases when
  none are supplied, helpers such as `get_lessons_for_new_mini()` still expect a
  `phases` array; provide explicit phase definitions to avoid lookup errors.
- `legacy/client_specifications.docx` is copied into the student deliverable via
  `_write_client_spec_once()` the first time a run executes.
- Additional runtime inputs include detected issues and lessons learned, which
  are persisted between iterations via `self.feedback_storage` and
  `self.partial_lessons`. The implementation serializes these as JSON fragments
  so state can be replayed during analysis.

## Data Outputs

- `output_multi_iter.docx` and `teacher_review_output_multi_iter.docx` collect
  headings, lessons learned, and artifacts for the student and teacher
  audiences. Saving the DOCX files overwrites any prior run.
- `iteration_metrics.csv` is recreated at the start of each run and records one
  row per artifact per miniiteration.
- `mini_iteration_summary_metrics.csv` appends a row for every finalized
  miniiteration, creating the file with headers if it does not yet exist.
- `iteration_summary_metrics.csv` appends macroiteration roll-ups once
  `compute_iteration_metrics()` completes.
- `issue_lifecycle.csv` captures discovery and resolution timing based on the
  `IssueTracker` state machine and is rewritten each run.
- `colab_console_output.txt` mirrors stdout for traceability via `DualLogger`.

Metric payloads (see below) remain optimized for aggregate trend analysis rather
than final-iteration comparisons, so enhanced-mode runs appear to generate more
activity because they include multiple miniiterations per macroiteration.

## Simulation Lifecycle Checklist

When executing the legacy simulation, keep the following order of operations in
mind:

1. Load the client specification and configuration JSON files.
2. Initialize iteration counters and seed the backlog of existing issues and
   lessons learned (empty on a fresh run).
3. Execute each macroiteration, generating artifacts and collecting issues for
   every miniiteration within it.
4. Persist the generated artifacts, issue logs, lessons learned, and metric
   payloads for reporting.
5. Feed the accumulated issues and lessons into the next iteration cycle until
   the configured stopping conditions are met.

## Intended Metrics

- **Artifact quality indicators:** tabular complexity scores, completeness
  ratios, or other structural quality heuristics per iteration.
- **Issue concentration:** number and severity of issues remaining after the
  final macro/miniiteration cycle.
- **Iteration efficiency:** counts and durations of macroiterations and
  miniiterations required to reach completion.
- **Responsiveness to feedback:** elapsed iterations between issue detection and
  resolution, tracked separately for standard vs. enhanced modes.
- **Stability vs. adaptability trade-offs:** comparative analysis of how often
  requirements shift and how quickly each mode incorporates them.

## Reporting Limitations & Planned Fixes

### Known Limitations

- Current reports compare entire macroiterations, so enhanced mode runs—whose
  macroiterations may include many miniiterations—appear to deliver more output
  but also carry inflated issue totals, making comparisons with standard mode
  misleading.
- Metrics do not yet normalize for the varying number of miniiterations, nor do
  they highlight the quality of the *final* iteration outputs specifically.
- State persistence is optimized for replaying complete macroiterations, which
  makes it cumbersome to isolate a single miniiteration when debugging.

### Planned Fixes

- Introduce a reporting layer that focuses on comparable checkpoints: the last
  miniiteration of the final macroiteration in enhanced mode versus the last
  macroiteration output in standard mode.
- Normalize metrics (issue rates, artifact complexity, cycle counts) by
  miniiteration count to enable apples-to-apples trend analysis.
- Capture and expose per-iteration timestamps or sequence numbers so analysts
  can trace feedback latency explicitly.
- Extend documentation and dashboards to clarify when aggregate macro-level
  metrics versus final-iteration metrics should be used.
- Add tooling to extract and inspect single miniiterations so teams can debug
  discrepancies without replaying entire macroiterations.

## Operational checklist

Use this quick-reference list when running the legacy notebook (Colab or local):

1. Install dependencies (`python-docx`, `openai>=1.77`, `nltk`) as shown at the
   top of `legacy/openai19pm10.py` and ensure NLTK resources are downloaded.
2. Stage the three primary inputs (`client_specifications.docx`,
   `artifact_data.json`, and `simulation_config.json`) alongside the notebook.
3. Invoke `orchestrate_process()` (wrapper around `DevOpsSimulation`), then
   confirm `output_multi_iter.docx`, `teacher_review_output_multi_iter.docx`,
   `iteration_metrics.csv`, `mini_iteration_summary_metrics.csv`,
   `iteration_summary_metrics.csv`, `issue_lifecycle.csv`, and
   `colab_console_output.txt` update in `legacy/`.
4. Review `mini_iteration_summary_metrics` between `iteration_summary_metrics`
   rows to verify enhanced mode emitted multiple miniiterations when
   configured.
5. Archive the generated artifacts and console transcript to maintain a replay
   trail for regression analysis.

## Related Documentation

- The root [`README.md`](../../README.md) provides a project overview and links to
  this document.
- [`behavior-baseline.md`](behavior-baseline.md) records the current simulator
  behavior contract that refactors must preserve.
- Future developer guides should reference this overview to ensure consistent
  terminology and shared understanding of workflows and metrics.

