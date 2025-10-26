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

Key orchestration logic lives inside `legacy/openai19pm10.py`:

- `DevOpsSimulation.run_simulation()` — entry point that dispatches to the
  standard (`run_backcompat_iterations`) or enhanced
  (`run_phase_based_simulation`) drivers based on `simulation_config.json`.
- `run_phase_single_pass()` — executes a single phase/miniiteration pair,
  updating artifacts, the `IssueTracker`, and phase counters.
- `finalize_mini_iteration()` and `compute_iteration_metrics()` — emit
  `mini_iteration_summary_metrics.csv` and `iteration_summary_metrics.csv`
  respectively.
- `setup_documentation()` — prepares the CSV/DOCX outputs and ensures headers
  exist before iteration work begins.
- `DualLogger` bootstrap — mirrors notebook stdout into
  `colab_console_output - *.txt` so console history accompanies generated
  artifacts.

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

## Iteration Workflow

The simulation advances through repeating macroiterations that contain one or
more miniiterations. Each iteration pulls forward the latest issues and lessons
learned so that subsequent passes can react to new information quickly.

### Macroiterations

- Represent the major planning and delivery cycles for both modes.
- Define the scope of objectives, inputs, and success criteria for their
  contained miniiterations.
- Begin by injecting accumulated issues and lessons learned from the previous
  macroiteration (standard mode) or both the previous macroiteration and the
  final miniiteration (enhanced mode).

### Miniiterations

- Cover the repeated artifact/table generation cycles within a macroiteration.
- Standard mode always executes exactly one miniiteration, so issue feedback
  waits until the next macroiteration.
- Enhanced mode can execute multiple miniiterations; each miniiteration ingests
  issues discovered in the immediately preceding cycle alongside any macro-level
  lessons.

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

- `output_multi_iter - <timestamp>.docx` and
  `teacher_review_output_multi_iter - <timestamp>.docx` collect headings,
  lessons learned, and artifacts for the student and teacher audiences.
- `iteration_metrics - <timestamp>.csv` is recreated on each run with
  macro-level iteration metrics. `iteration_summary_metrics - <timestamp>.csv`
  appends
  per-macroiteration summary rows, while `mini_iteration_summary_metrics -
  <timestamp>.csv` captures every miniiteration finalized by
  `finalize_mini_iteration()`.
- `issue_lifecycle - <timestamp>.csv` records discovery and resolution timing
  based on the `IssueTracker` state machine.
- `colab_console_output - <timestamp>.txt` mirrors stdout for traceability.

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
   confirm DOCX, CSV, and console files appear with fresh timestamps in
   `legacy/`.
4. Review `mini_iteration_summary_metrics` between `iteration_summary_metrics`
   rows to verify enhanced mode emitted multiple miniiterations when
   configured.
5. Archive the generated artifacts and console transcript to maintain a replay
   trail for regression analysis.

## Related Documentation

- The root [`README.md`](../README.md) provides a project overview and links to
  this document.
- `legacy/documentation/behavior-baseline.md` records the current simulator
  behavior contract that refactors must preserve.
- Future developer guides should reference this overview to ensure consistent
  terminology and shared understanding of workflows and metrics.

