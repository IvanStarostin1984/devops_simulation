# Legacy simulator operations guide

## Purpose

Use this guide to run the legacy DevOps simulation exactly as it behaves today
and to understand the artifacts it produces before any refactoring begins. The
steps mirror the single-cell notebook embedded in `legacy/openai19pm10.py`, so
following them preserves the current workflow quirks that future updates must
continue to honor.

## Prerequisites

- A Google Colab runtime (the notebook relies on `google.colab.files` helpers
  for uploads and downloads).【F:legacy/openai19pm10.py†L68-L82】
- Install the pinned packages at the start of the cell to ensure document
  parsing, OpenAI access, and NLP tokenization succeed: `python-docx`,
  `openai>=1.77`, and `nltk`.【F:legacy/openai19pm10.py†L53-L58】
- Leave the `DualLogger` assignment in place so every print statement is copied
  into `colab_console_output.txt`, which is overwritten on each run for a clean
  transcript.【F:legacy/openai19pm10.py†L10-L39】
- Call `ensure_nltk_resources()` once per runtime session (the notebook invokes
  it automatically) to download and alias the NLTK corpora required by the
  artifact parsers without failing if `punkt_tab` is requested.【F:legacy/openai19pm10.py†L82-L118】【F:legacy/openai19pm10.py†L7823-L7825】

## Required inputs

Upload the following files when prompted by the notebook dialog:

1. **Client specification** — exactly one `.txt` or `.docx` file containing the
   problem statement. The notebook reads `.docx` files via `python-docx`,
   converts them to plain text, and stores the result in `builtins.CLIENT_SPEC`
   for later use. Empty files raise an error so the run halts before any
   iterations begin.【F:legacy/openai19pm10.py†L7764-L7794】
2. **`artifact_data.json`** — mandatory prompt and table metadata consumed by the
   orchestrator. The file is loaded unconditionally before the simulation starts
   processing iterations.【F:legacy/openai19pm10.py†L7795-L7798】
3. **`simulation_config.json` (optional)** — overrides the default
   `backwardCompatibilityMode = true`, `macroIterations = 1`, and empty `phases`
   list that the notebook seeds when no configuration is supplied. Provide this
   file to exercise enhanced-mode behavior or custom phase layouts.【F:legacy/openai19pm10.py†L7799-L7807】

## Running the simulation

1. Execute the cell so `files.upload()` opens the chooser and enforces the
   single-specification rule. If more than one spec file is detected or none are
   selected, the notebook raises a `ValueError` to prevent ambiguous runs.【F:legacy/openai19pm10.py†L7764-L7773】
2. Confirm the spec file has content. Empty uploads also trigger a `ValueError`
   before any state is mutated.【F:legacy/openai19pm10.py†L7789-L7790】
3. Supply `simulation_config.json` if you need to change iteration counts or
   switch off backward compatibility. Otherwise the seeded defaults remain in
   effect for the entire session.【F:legacy/openai19pm10.py†L7799-L7807】
4. Let `DevOpsSimulation` construct with the loaded artifact data and chosen
   configuration, then call `run_simulation()` to execute either the standard or
   enhanced driver based on `backwardCompatibilityMode`. No additional inputs
   are requested once execution begins.【F:legacy/openai19pm10.py†L7809-L7811】
5. After the run completes, download the generated artifacts surfaced in the
   notebook UI. The loop exports CSV metrics and DOCX reports that overwrite
   prior runs, so pull copies if you need to compare results later.【F:legacy/openai19pm10.py†L7813-L7822】
6. Collect `colab_console_output.txt` from the working directory when you need a
   transcript. Because `DualLogger` writes in overwrite mode, each run produces a
   fresh log without residual lines from previous sessions.【F:legacy/openai19pm10.py†L24-L39】

## Interpreting the outputs

The notebook offers the following files for download at the end of each run:

- `iteration_metrics.csv`, `iteration_summary_metrics.csv`, and
  `mini_iteration_summary_metrics.csv` — CSV datasets that capture per-artifact,
  per-mini, and per-macro metrics respectively.【F:legacy/openai19pm10.py†L7813-L7820】
- `issue_lifecycle.csv` — a cumulative log of issue states across runs, sourced
  from the same loop.【F:legacy/openai19pm10.py†L7813-L7819】
- `output_multi_iter.docx` and `teacher_review_output_multi_iter.docx` — student
  and teacher deliverables containing headings, artifacts, and lesson summaries
  from the final macroiteration.【F:legacy/openai19pm10.py†L7813-L7820】
- `colab_console_output.txt` — the mirrored stdout transcript for traceability.
  Because it is not part of the download loop, pull it manually from the runtime
  if needed.【F:legacy/openai19pm10.py†L24-L39】

Refer to the dedicated artifact and configuration references for deeper detail
on schema layouts, append/overwrite behavior, and iteration knobs:

- [Artifact inventory](artifacts.md)
- [Configuration surfaces](configuration.md)
- [Lifecycle flow](flow.md)
- [Runtime logging details](runtime-log.md)

## Troubleshooting

- **Multiple or missing specs** — Re-upload with exactly one `.txt` or `.docx`
  file; the guardrail intentionally raises a `ValueError` so runs stay
  deterministic.【F:legacy/openai19pm10.py†L7764-L7773】
- **`python-docx` import errors** — Ensure the prerequisite install cell ran; the
  notebook surfaces a descriptive `ImportError` if the library is missing when a
  `.docx` spec is processed.【F:legacy/openai19pm10.py†L7775-L7784】
- **Empty outputs** — Verify the configuration assigns tables to each phase.
  Without a `tableList` mapping, the simulation completes successfully but skips
  artifact generation. Use `simulation_config.json` to supply explicit phase
  definitions.【F:legacy/openai19pm10.py†L7799-L7807】【F:legacy/openai19pm10.py†L3092-L3491】

Following this guide keeps contributors aligned on the legacy experience so
future refactors can measure behavior changes against a well-documented baseline.

