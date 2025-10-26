# Legacy Simulator Artifact and IO Reference

## Outcomes and acceptance criteria

- Readers understand which files must exist before invoking the notebook and how optional configuration defaults are applied when missing.
- Documentation clarifies how each CSV, DOCX, and log file is produced, including whether the run overwrites or appends to an existing file.
- Contributors can cite this document to preserve the current download list and artifact lifecycles when refactoring the simulator.

## Required inputs before execution

The Colab notebook prompts for specific assets when `orchestrate_process()`
runs. It accepts exactly one non-empty client specification file (`.txt` or
`.docx`), loads `artifact_data.json`, and applies `simulation_config.json` only
when the user uploads an override. Otherwise the simulator defaults to
backward-compatibility mode with a single macroiteration and no explicit
phases.【F:legacy/openai19pm10.py†L7763-L7822】 The extracted specification text
is stored in `builtins.CLIENT_SPEC`, giving `DevOpsSimulation.__init__()` a
default message if no upload occurred in previous sessions.【F:legacy/openai19pm10.py†L2320-L2335】【F:legacy/openai19pm10.py†L7792-L7793】

### Configuration fallbacks and seeded state

- `simulation_config.json` defaults to `{"backwardCompatibilityMode": true,
  "macroIterations": 1, "phases": []}` whenever the upload dialog omits a
  configuration file, so the orchestrator always has safe values for macro and
  mini iteration counts.【F:legacy/openai19pm10.py†L7785-L7804】
- The current client specification persists across notebook runs through the
  global `builtins.CLIENT_SPEC`. If the user skips a new upload, the prior
  value remains available to `DevOpsSimulation.__init__()` and downstream
  helpers that need the textual prompt.【F:legacy/openai19pm10.py†L2320-L2335】【F:legacy/openai19pm10.py†L7792-L7793】

These defaults are part of the observable contract and should remain intact
until the refactor introduces an explicit configuration surface.

## Logging side effect

The first cell replaces `sys.stdout` with `DualLogger`, mirroring every print
statement to `colab_console_output.txt`. Because the file handle is opened in
write mode and preserved for the entire session, each run truncates the prior
log before capturing new console output.【F:legacy/openai19pm10.py†L10-L39】

## Document and CSV initialization

`DevOpsSimulation.setup_documentation()` prepares fresh student and teacher
DOCX files and reinitializes `iteration_metrics.csv` with the artifact-level
header on every run, ensuring the artifact log starts empty.【F:legacy/openai19pm10.py†L2781-L2854】
The method then ensures the macro and mini summary CSVs exist, only creating
headers when the files are missing so existing histories continue to append
between runs.【F:legacy/openai19pm10.py†L2856-L2990】 These paths live alongside
the notebook so repeated executions reuse the same filenames.

### Artifact lifecycle matrix

| Artifact | Producer | Lifecycle mode | Seeded defaults / notes |
| --- | --- | --- | --- |
| `colab_console_output.txt` | `DualLogger` (first cell) | Overwrite per run | File opened with mode `"w"`, so prior transcripts are truncated before capturing the new session.【F:legacy/openai19pm10.py†L10-L39】 |
| `iteration_metrics.csv` | `setup_documentation()` | Overwrite per run | File handle opened in write mode, clearing the artifact-level log each time the notebook starts.【F:legacy/openai19pm10.py†L2798-L2824】 |
| `mini_iteration_summary_metrics.csv` | `setup_documentation()` + `finalize_mini_iteration()` | Append | Headers created on first run; subsequent mini iterations append one row per call.【F:legacy/openai19pm10.py†L2964-L2990】【F:legacy/openai19pm10.py†L5332-L5353】 |
| `iteration_summary_metrics.csv` | `setup_documentation()` + `write_iteration_reports()` | Append | Header generated only when missing; each macro iteration appends a new row with aggregated metrics.【F:legacy/openai19pm10.py†L2856-L2937】【F:legacy/openai19pm10.py†L6363-L6478】 |
| `output_multi_iter.docx` | `setup_documentation()` / `generate_final_summary_doc()` | Overwrite per run | New `Document()` instances guarantee the exported DOCX captures only the current execution’s content.【F:legacy/openai19pm10.py†L2790-L2806】【F:legacy/openai19pm10.py†L5458-L5466】 |
| `teacher_review_output_multi_iter.docx` | `setup_documentation()` / `generate_final_summary_doc()` | Overwrite per run | Mirror of the student DOCX: new document every start, saved once finalization completes.【F:legacy/openai19pm10.py†L2790-L2806】【F:legacy/openai19pm10.py†L5458-L5466】 |
| `issue_lifecycle.csv` | `generate_final_summary_doc()` | Overwrite per run | Regenerated during finalization, replacing any earlier lifecycle export.【F:legacy/openai19pm10.py†L5468-L5487】 |

## Mini-iteration persistence

During each miniiteration, `finalize_mini_iteration()` computes metrics and appends one row to `mini_iteration_summary_metrics.csv` using append mode. This preserves the cumulative record across runs and enforces the observable behavior that the simulator never rewrites prior miniiteration summaries.【F:legacy/openai19pm10.py†L5157-L5353】

## Macro-iteration rollups

After every macroiteration, `write_iteration_reports()` augments the teacher review DOCX with summary bullets and appends a row to `iteration_summary_metrics.csv`. Like the mini summary, the macro report uses append mode so earlier macroiterations remain intact when the simulator runs multiple times or across modes.【F:legacy/openai19pm10.py†L6296-L6489】

## Finalization outputs

When all macroiterations complete, `generate_final_summary_doc()` saves the
student (`output_multi_iter.docx`) and teacher (`teacher_review_output_multi_iter.docx`)
documents, closes the artifact metrics CSV, and regenerates `issue_lifecycle.csv`.
These steps overwrite any prior versions to keep the artifacts aligned with the
latest execution.【F:legacy/openai19pm10.py†L5458-L5487】 Closing the CSV handle
after the docx exports finalizes the `iteration_metrics.csv` contents before
the lifecycle regeneration runs.【F:legacy/openai19pm10.py†L5458-L5473】

## Download bundle

At the end of the notebook, `orchestrate_process()` calls `files.download` for the six primary artifacts: the regenerated artifact metrics CSV, both summary CSVs, the refreshed lifecycle CSV, and the two DOCX reports. This list defines the baseline outputs contributors should continue delivering until the refactor explicitly changes the distribution contract.【F:legacy/openai19pm10.py†L7813-L7822】
