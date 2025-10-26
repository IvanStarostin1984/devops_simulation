# Legacy simulator artifact templates

## Purpose

The legacy notebook loads `legacy/artifact_data.json` during simulator
construction and treats it as the contract for every artifact the LLM must
populate. The JSON payload seeds table definitions, dependency graphs, trace
columns, and review questions that the orchestrator reuses across all
macro/mini-iteration loops. This reference explains how each section is
interpreted so refactors can preserve the same prompts, metrics, and
artifacts without re-reading the notebook.

## JSON layout

`artifact_data.json` exposes four major collections that the simulation caches
in `DevOpsSimulation.__init__()` when the run starts.【F:legacy/openai19pm10.py†L2332-L2345】

- `tables`: ordered list of artifact definitions.
- `dependencies_map`: adjacency list describing which artifacts feed others.
- `artifact_key_map`: mapping of artifact indices to the primary-key column used
  for row-level diffing.
- `artifact_trace_column_map`: column to inspect when computing traceability.
- `artifact_review_questions`: reviewer prompts grouped by artifact index.

Each sub-section below documents how the orchestrator consumes these payloads.

## `tables` entries

Every element in `tables` is a dictionary describing a single artifact. Key
fields and their run-time usage:

- `row_number`: ordering hint propagated into process-change hooks and
  retrospective outputs.【F:legacy/openai19pm10.py†L7081-L7096】【F:legacy/openai19pm10.py†L7625-L7629】
- `Artifact_index`: primary identifier used throughout the orchestrator. Phase
  dispatch filters artifacts by index when running a phase/mini tuple, so every
  configured phase list must reference these values.【F:legacy/openai19pm10.py†L3445-L3483】
- `Artifact_name`: surfaced in student DOCX headings, teacher summaries, and
  issue overviews so readers can map metrics back to artifacts.【F:legacy/openai19pm10.py†L3473-L3487】【F:legacy/openai19pm10.py†L6704-L6721】
- `Artifact_type`: printed in retrospectives and overview text to clarify each
  artifact’s role.【F:legacy/openai19pm10.py†L6704-L6721】
- `Artifact_columns`: comma-separated schema string. The orchestrator splits it
  into a column list before prompting the writer LLM and reuses it when
  formatting DOCX tables.【F:legacy/openai19pm10.py†L7431-L7440】【F:legacy/openai19pm10.py†L3473-L3482】
- `Obligatory_rows`: descriptive guidance the prompts treat as mandatory when
  generating rows. The value is passed through untouched inside the
  instructions block that guides the writer LLM.【F:legacy/openai19pm10.py†L7446-L7520】
- `custom_instructions`: artifact-specific rules appended to reviewer and writer
  prompts so feedback stays grounded in each table’s scope.【F:legacy/openai19pm10.py†L1689-L1700】【F:legacy/openai19pm10.py†L7446-L7520】
- `Sample Data Rows`: example rows injected into prompts and updated whenever
  process changes add columns so downstream passes stay consistent.【F:legacy/openai19pm10.py†L7397-L7440】【F:legacy/openai19pm10.py†L4234-L4266】

### Process-change helpers

The notebook exposes helper methods so lessons-learned automation can mutate the
artifact catalog:

- `add_columns_to_table()` appends new columns to `Artifact_columns`, backfills
  `Sample Data Rows`, and warns if duplicate artifact indices exist.【F:legacy/openai19pm10.py†L4190-L4269】
- `add_new_table()` creates a brand-new table definition, updates
  `dependencies_map`, and logs the addition for traceability.【F:legacy/openai19pm10.py†L4291-L4321】
- `modify_artifact_instructions()` appends extra guidance to the existing
  `custom_instructions` field without duplicating text.【F:legacy/openai19pm10.py†L4324-L4340】

Process changes emitted by the retrospectives call these helpers directly,
allowing the simulation to evolve artifacts between iterations without editing
`artifact_data.json` by hand.【F:legacy/openai19pm10.py†L7041-L7109】

## `dependencies_map`

`dependencies_map` records which artifacts must precede others. The orchestrator
consults it when building dependency summaries for prompts so the writer LLM can
reference upstream context. Each dependency entry resolves to the latest stored
artifact snapshot, including its rendered rows, before the prompt is assembled.【F:legacy/openai19pm10.py†L4087-L4136】

## `artifact_key_map`

The simulator uses `artifact_key_map` in two places:

1. Row-level diffing during artifact processing so churn metrics align to the
   column that uniquely identifies rows.【F:legacy/openai19pm10.py†L3724-L3773】
2. Prompt generation, where the primary key is surfaced as a mandatory bullet to
   keep identifiers stable across iterations.【F:legacy/openai19pm10.py†L7397-L7440】

## `artifact_trace_column_map`

Traceability metrics compute coverage over whichever column is mapped to each
artifact. If no explicit mapping is provided, the simulator falls back to the
`Related Reqs` column. Populate this map to avoid measuring the wrong field when
artifacts rename their trace columns.【F:legacy/openai19pm10.py†L4140-L4157】

## `artifact_review_questions`

Reviewer questions drive the teacher feedback loop. For every artifact, the
notebook loads its corresponding question set, appends downstream phase context,
merges any custom instructions, and feeds the prompts to the review LLM so new
issues can be surfaced and deduplicated.【F:legacy/openai19pm10.py†L1632-L1700】【F:legacy/openai19pm10.py†L1707-L1779】

## Interaction with metrics and outputs

Artifacts parsed from `tables` flow through `process_single_artifact()`, which
computes metrics, tracks issue states, and writes rows to
`iteration_metrics.csv`. All schema-driven values—columns, dependency text,
review prompts, and primary keys—must remain consistent with the JSON so the CSV
and DOCX outputs continue to reflect the same structure.【F:legacy/openai19pm10.py†L3543-L3773】【F:legacy/openai19pm10.py†L3783-L3945】
