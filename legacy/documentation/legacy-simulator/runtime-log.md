# Legacy simulator runtime & logging reference

## Purpose

This guide documents the bootstrap expectations, logging side effects, and file
handoffs enforced by the legacy Colab notebook so operators can reproduce the
same environment before the refactor changes anything.

## Stdout capture and log files

- The notebook replaces `sys.stdout` with `DualLogger`, mirroring every print
  call to `colab_console_output.txt` while still displaying output in the
  notebook UI.【F:legacy/openai19pm10.py†L10-L42】
- `DualLogger` opens the transcript in write mode, so each run overwrites the
  previous log. Archive copies if you need a historical trail.【F:legacy/openai19pm10.py†L21-L40】
- `DEBUG_MODE` toggles verbose diagnostic output; leave it `True` when auditing
  prompt payloads or issue lifecycles.【F:legacy/openai19pm10.py†L45-L52】

## Environment bootstrap

- The first cell installs `python-docx`, `openai>=1.77`, and `nltk` using Colab
  magic commands. Re-run the cell whenever a new runtime starts to restore these
  wheels.【F:legacy/openai19pm10.py†L54-L58】
- `ensure_nltk_resources()` patches NLTK’s download/find behavior, prepares a
  writable `/root/nltk_data` directory, downloads the required corpora, and
  amends the search path so tokenizers work offline.【F:legacy/openai19pm10.py†L82-L138】
- The helper is invoked immediately after definition and again at the end of the
  notebook before orchestration to guarantee the resources exist for downstream
  prompts.【F:legacy/openai19pm10.py†L137-L141】【F:legacy/openai19pm10.py†L7823-L7825】

## Credentials and configuration

- The simulation reads the OpenAI API key from `google.colab.userdata` and copies
  it into `os.environ["OPENAI_API_KEY"]` during initialization. Populate the
  secret in Colab before running or inject an equivalent environment variable
  locally.【F:legacy/openai19pm10.py†L2280-L2335】
- `builtins.CLIENT_SPEC` acts as the single source of truth for the client brief.
  `orchestrate_process()` populates it from the uploaded specification before
  constructing `DevOpsSimulation`, so local runs must seed the same global if they
  bypass the upload helper.【F:legacy/openai19pm10.py†L2332-L2335】【F:legacy/openai19pm10.py†L7764-L7794】

## Running the orchestration cell

`orchestrate_process()` bundles all runtime I/O into one entry point.【F:legacy/openai19pm10.py†L7760-L7825】

1. Prompt the operator to upload `artifact_data.json`, an optional
   `simulation_config.json`, and exactly one client specification file.
   `.docx` uploads require the earlier `python-docx` install.【F:legacy/openai19pm10.py†L7764-L7808】
2. Store the uploaded specification in `builtins.CLIENT_SPEC`, reload
   `artifact_data.json`, and merge configuration overrides before instantiating
   `DevOpsSimulation`.【F:legacy/openai19pm10.py†L7792-L7811】
3. After the run, trigger downloads for the CSV and DOCX artifacts so the Colab
   operator can save them locally.【F:legacy/openai19pm10.py†L7813-L7822】

## Operational checklist

- Confirm the API key is registered in `google.colab.userdata` (or exported to
  the environment when running locally) before calling `orchestrate_process()`.
- Run the bootstrap cell whenever a fresh Colab session starts so the pip
  installs and `ensure_nltk_resources()` side effects are available.
- Archive `colab_console_output.txt` after each run if you need the transcript
  for regression analysis; subsequent executions overwrite it.
