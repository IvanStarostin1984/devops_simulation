...
- [x] Documented legacy artifacts inventory (NOTES 2025-02-14 PR #draft) and linked from main README.
# TODO

- [ ] Document simulation overview (see NOTES.md entry dated 2025-02-14).
  - Draft docs/simulation-overview.md covering modes, workflows, data IO, metrics.
  - Describe reporting limitations and planned remediation.
  - Cross-link overview from README and future developer guides section.
- [x] Verify legacy behavior baseline doc accuracy ([Plan](NOTES.md#2025-10-31--pr-draft)).
- [x] Fill any gaps found in legacy/documentation/behavior-baseline.md so it captures
      actual control flow, IO contracts, and guardrails before refactoring (NOTES 2025-10-31 PR #draft).
- [x] Captured legacy simulator behavior baseline (NOTES 2025-10-30 PR #draft)
  and published `legacy/documentation/behavior-baseline.md`.
- [x] Refresh README.md and docs/simulation-overview.md per NOTES.md entry dated
  2025-10-27 to remove typos, clarify guardrails, and highlight next steps.

# Documentation backlog — aligned with NOTES.md plan

1. **README overhaul** — restructure the introduction, add a quickstart and
   clarify repo purpose vs. simulation assets. ([Plan](NOTES.md#2025-10-26--pr-draft))
2. **Simulation reference guide** — document how to locate, run, and interpret
   simulation outputs, including sample workflows. ([Plan](NOTES.md#2025-10-26--pr-draft))
3. **Legacy inventory audit** — catalogue `legacy/` artifacts, record retention
   rationale, and flag candidates for archival or cleanup. ([Plan](NOTES.md#2025-10-26--pr-draft))
4. **Follow-up documentation tasks** — track subsequent updates (architecture
   notes, contribution guidelines, changelog entries) resulting from the above
   work. ([Plan](NOTES.md#2025-10-26--pr-draft))

## Legacy simulator behavior documentation plan ([Plan](NOTES.md#2025-10-29--pr-draft))

> All tracks below execute in parallel and target distinct documentation files
> under `docs/legacy-simulator/` to avoid cross-file contention.

- [ ] **Track: Orchestrator flow and helper contracts**
  - Inspect `DevOpsSimulation.__init__`, `run_simulation`,
    `run_backcompat_iterations`, `run_phase_based_simulation`, and
    `run_phase_single_pass` inside `legacy/openai19pm10.py` to outline macro vs.
    mini iteration transitions and decision points.
  - Capture how supporting classes (`IssueTracker`, `MetricsCalculator`,
    `LLMClient`) and utilities (`DualLogger`, `_write_client_spec_once`,
    `apply_global_unresolved_issues`) feed metrics, headings, and carry-over
    state into each pass.
  - Draft `docs/legacy-simulator/flow.md` synthesising the lifecycle narrative
    and citing where CSV writers and DOCX generation hooks fire.
- [ ] **Track: Artifact template structures**
  - Review how `DevOpsSimulation` consumes `artifact_data.json` keys such as
    `tables`, `dependencies_map`, `artifact_key_map`,
    `artifact_trace_column_map`, and `artifact_review_questions` when preparing
    artifacts and trace coverage.
  - Document how helper functions like `add_json_as_docx_table` translate rows
    into the student/teacher DOCX outputs and how dependency ordering affects
    phase execution.
  - Author `docs/legacy-simulator/templates.md` describing the schema fields,
    prompt text, and downstream consumers.
- [ ] **Track: Simulation configuration surfaces**
  - Audit `simulation_config.json` alongside `DevOpsSimulation.__init__` to
    explain controls for `macroIterations`, `phases`, `_phase_final_mini`, and
    toggles like `backwardCompatibilityMode`.
  - Note fallbacks used when keys are missing (e.g., default phases list,
    macro/mini counts) and how they steer orchestrator branching.
  - Create `docs/legacy-simulator/configuration.md` summarising every knob and
    linking them to the decision logic they influence.
- [ ] **Track: Runtime inputs and generated outputs**
  - Trace how `setup_documentation` and related writers emit
    `iteration_metrics.csv`, `mini_iteration_summary_metrics.csv`,
    `iteration_summary_metrics.csv`, and the DOCX reports under timestamped
    filenames.
  - Include coverage of seeded assets (`client_specifications.docx`, existing
    CSV history) and how the simulator appends vs. rewrites data.
  - Compile the findings into `docs/legacy-simulator/artifacts.md`, including
    downstream analysis hints for each artifact.
- [ ] **Track: Runtime logging and environment bootstrap**
  - Document the `DualLogger` stdout hijack, notebook `print` mirroring, and the
    Colab-specific bootstrap commands (`!pip install …`, `ensure_nltk_resources`
    guards) at the top of `legacy/openai19pm10.py`.
  - Record expectations for storing `colab_console_output - *.txt`, handling
    dependency versions, and reproducing local installs.
  - Produce `docs/legacy-simulator/runtime-log.md` consolidating these
    environment and logging considerations.
