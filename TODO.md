...
- [x] Documented legacy artifacts inventory (NOTES 2025-02-14 PR #draft) and linked from main README.
# TODO

- [x] Document simulation overview (see NOTES.md entries dated 2025-02-14,
      2025-11-01, and 2025-11-02).
  - Draft `legacy/documentation/simulation-overview.md` covering modes, workflows, data IO,
    metrics, and explicit references to the legacy runtime files they stem
    from.
  - Describe reporting limitations and planned remediation.
  - Cross-link the overview from `README.md` and future developer guides
    section.
  - Acceptance criteria: overview reflects observed legacy behavior, enumerates
    required inputs/outputs, and preserves current limitations for refactor
    parity.
  - Completed per `NOTES.md#2025-11-02--pr-draft`: overview now cites
    `run_backcompat_iterations`, `run_phase_based_simulation`, and state
    propagation helpers alongside artifact emission points.
- [x] Verify legacy behavior baseline doc accuracy ([Plan](NOTES.md#2025-10-31--pr-draft)).
- [x] Fill any gaps found in legacy/documentation/behavior-baseline.md so it captures
      actual control flow, IO contracts, and guardrails before refactoring (NOTES 2025-10-31 PR #draft).
- [x] Captured legacy simulator behavior baseline (NOTES 2025-10-30 PR #draft)
  and published `legacy/documentation/behavior-baseline.md`.
- [x] Refresh README.md and legacy/documentation/simulation-overview.md per
  NOTES.md entry dated 2025-10-27 to remove typos, clarify guardrails, and
  highlight next steps.
- [x] Realign simulation overview artifact references ([Plan](NOTES.md#2025-11-03--pr-draft)).
  - Confirm the document lists `iteration_metrics.csv`,
    `mini_iteration_summary_metrics.csv`,
    `iteration_summary_metrics.csv`, `output_multi_iter.docx`,
    `teacher_review_output_multi_iter.docx`, and `colab_console_output.txt`.
  - Describe which outputs are overwritten on each run versus appended when
    existing files are present.
  - Acceptance criteria: documentation references match the filenames and
    behaviors in `setup_documentation()` and `generate_final_summary_doc()`.

- [x] Relocate legacy simulator documentation into `legacy/documentation/**`
      ([Plan](NOTES.md#2025-11-09--pr-draft)).
  - Ensure `legacy/documentation/simulation-overview.md` and
    `legacy/documentation/legacy-simulator/**` contain the relocated docs and
    remove redundant copies from `docs/`.
  - Update README.md, TODO.md, NOTES.md, and intra-document links so they target
    the new locations.
  - Acceptance criteria: No files under `docs/` describe the legacy simulator,
    and all repository references resolve to the relocated documents.
- [x] Realigned backlog links to the legacy documentation tree
      ([Plan](NOTES.md#2025-11-10--pr-draft)).
  - Replaced remaining `docs/legacy-simulator/**` references with
    `legacy/documentation/legacy-simulator/**` so contributors follow the
    relocated structure.
  - Confirmed README.md still accurately describes the repository layout and
    does not point to the removed `docs/` directory for legacy guidance.
  - Acceptance criteria: TODO.md active items now reference
    `legacy/documentation/**` exclusively, and spot-checking README.md found no
    stale `docs/` links to legacy material.

# Documentation backlog — aligned with NOTES.md plan

1. **README overhaul** — restructure the introduction, add a quickstart and
   clarify repo purpose vs. simulation assets. ([Plan](NOTES.md#2025-10-26--pr-draft))
2. **Simulation reference guide** — document how to locate, run, and interpret
   simulation outputs, including sample workflows.
   (Completed per [NOTES.md#2025-11-12--pr-draft](NOTES.md#2025-11-12--pr-draft))
3. **Legacy inventory audit** — catalogue `legacy/` artifacts, record retention
   rationale, and flag candidates for archival or cleanup. ([Plan](NOTES.md#2025-10-26--pr-draft))
4. **Follow-up documentation tasks** — track subsequent updates (architecture
   notes, contribution guidelines, changelog entries) resulting from the above
   work. ([Plan](NOTES.md#2025-10-26--pr-draft))

## Legacy simulator behavior documentation plan ([Plan](NOTES.md#2025-10-29--pr-draft))

> All tracks below execute in parallel and target distinct documentation files
> under `legacy/documentation/legacy-simulator/` to avoid cross-file contention.

- [x] **Track: Orchestrator flow and helper contracts** ([Outcome](NOTES.md#2025-11-05--pr-draft))
  - Inspect `DevOpsSimulation.__init__`, `run_simulation`,
    `run_backcompat_iterations`, `run_phase_based_simulation`, and
    `run_phase_single_pass` inside `legacy/openai19pm10.py` to outline macro vs.
    mini iteration transitions and decision points.
  - Capture how supporting classes (`IssueTracker`, `MetricsCalculator`,
    `LLMClient`) and utilities (`DualLogger`, `_write_client_spec_once`,
    `apply_global_unresolved_issues`) feed metrics, headings, and carry-over
    state into each pass.
  - Draft `legacy/documentation/legacy-simulator/flow.md` synthesising the
    lifecycle narrative and citing where CSV writers and DOCX generation hooks
    fire.
  - [x] Document orchestrator flow narrative (see
    [NOTES.md#2025-11-04--pr-draft](NOTES.md#2025-11-04--pr-draft)).
- [x] **Track: Artifact template structures**
  - Documented how the orchestrator consumes `tables`, `dependencies_map`,
    `artifact_key_map`, `artifact_trace_column_map`, and
    `artifact_review_questions` inside
    `legacy/documentation/legacy-simulator/templates.md` with direct code
    citations.
  - Covered helper behavior (`add_columns_to_table`, `add_new_table`,
    `modify_artifact_instructions`) so future process changes preserve the
    observed automation contract.
  - Completed per `NOTES.md#2025-11-11--pr-draft`.
- [ ] **Track: Simulation configuration surfaces** ([Plan](NOTES.md#2025-11-08--pr-draft))
  - Audit `simulation_config.json` alongside `DevOpsSimulation.__init__` to
    explain controls for `macroIterations`, `phases`, `_phase_final_mini`, and
    toggles like `backwardCompatibilityMode`.
  - Note fallbacks used when keys are missing (e.g., default phases list,
    macro/mini counts) and how they steer orchestrator branching.
  - Create `legacy/documentation/legacy-simulator/configuration.md` summarising
    every knob and linking them to the decision logic they influence.
- [x] **Track: Runtime inputs and generated outputs** ([Plan](NOTES.md#2025-11-06--pr-draft))
  - Documented current inputs, append/overwrite behavior, and download list in
    `legacy/documentation/legacy-simulator/artifacts.md`.
  - Captured how `setup_documentation` and downstream writers emit
    `iteration_metrics.csv`, `mini_iteration_summary_metrics.csv`,
    `iteration_summary_metrics.csv`, and the DOCX reports without introducing
    timestamped filenames.
  - Noted the seeded assets (`CLIENT_SPEC` fallback, existing CSV history) and
    whether each file is overwritten or appended on subsequent runs.
  - Summarised analysis hints for every artifact so refactors preserve the
    current bundle.
- [x] **Track: Artifact lifecycle polish** ([Plan](NOTES.md#2025-11-07--pr-draft))
  - Added a lifecycle matrix summarising producers, append/overwrite modes, and seeded defaults in legacy/documentation/legacy-simulator/artifacts.md.
  - Clarified configuration fallbacks when `simulation_config.json` is absent and how the notebook reuses `builtins.CLIENT_SPEC`.
  - Documented logging guarantees and truncation behaviour for `colab_console_output.txt` to preserve observable transcripts.
- [x] **Track: Runtime logging and environment bootstrap**
  - Captured the stdout mirroring, pip bootstrap, and `ensure_nltk_resources`
    idempotence in
    `legacy/documentation/legacy-simulator/runtime-log.md`.
  - Described credential handling, upload/download flow, and log preservation so
    future runs remain reproducible.
  - Completed per `NOTES.md#2025-11-11--pr-draft`.
- [x] Authored legacy simulator operations guide detailing prerequisites,
  execution flow, and artifact interpretation.
  ([Plan](NOTES.md#2025-11-12--pr-draft))
