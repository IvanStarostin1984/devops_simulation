# DevOps Simulation Overview

This document summarizes how the simulation models standard and enhanced DevOps
processes, how work iterates through macroiterations and miniiterations, the
key data artifacts exchanged, and the metrics we expect to evaluate.

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

- Two JSON files provide the primary inputs:
  - **Client specification data** that seeds requirements, constraints, and
    context for artifacts.
  - **Mode configuration data** that determines whether the simulation runs in
    standard or enhanced mode and captures iteration parameters (counts,
    thresholds, toggles).
- Additional runtime inputs include detected issues and lessons learned, which
  are persisted between iterations and fed back into the workflow per the rules
  above.

## Data Outputs

- Generated documentation artifacts and structured tables per iteration.
- Issue logs that capture defects detected during each generation pass.
- Lessons learned summaries keyed by macroiteration and miniiteration.
- Metric payloads (see below) emitted for downstream analysis and reporting.

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

## Related Documentation

- The root [`README.md`](../README.md) provides a project overview and links to
  this document.
- Future developer guides should reference this overview to ensure consistent
  terminology and shared understanding of workflows and metrics.

