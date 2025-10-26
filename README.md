# DevOps Simulation

This repository documents and experiments with two DevOps operating modes so we
can record the legacy behavior, plan a safe refactor, and stage future
enhancements. The code that drives the simulation lives in `legacy/`; the
current focus is capturing how it behaves before any modernization work begins.

## Repository Goals

- **Document the existing system** so contributors share a single source of
  truth for workflows, inputs, and outputs.
- **Protect the legacy implementation** until a refactoring strategy is agreed
  upon and scheduled.
- **Surface improvement opportunities** that will guide enhancements once the
  refactor is complete.

## Operating Modes at a Glance

The simulation can execute in two configurations. Both consume the same client
and configuration JSON files but differ in how quickly feedback loops close.

### Standard DevOps (`backward compatibility = true`)

- Executes exactly **one miniiteration per macroiteration**.
- Defers issue feedback until the **start of the next macroiteration**.
- Generates lessons learned at the end of a macroiteration and exposes them to
  creators in the following macroiteration.
- Prioritizes predictable, linear delivery and emphasizes stability over rapid
  adaptation.

### Enhanced DevOps (`backward compatibility = false`)

- Allows **multiple miniiterations** inside a macroiteration.
- Injects detected issues directly into the **next miniiteration** so teams can
  react immediately.
- Produces lessons learned at **both** miniiteration and macroiteration
  boundaries.
- Trades additional iteration overhead for faster feedback and higher artifact
  quality.

For a deeper walkthrough of inputs, iteration flow, and metrics, see the
[simulation overview](docs/simulation-overview.md).

## Inputs, Outputs, and Metrics

- **Inputs:** two JSON files (client/domain specification and mode/config
  parameters) plus accumulated issues and lessons from prior iterations.
- **Outputs:** generated documentation artifacts, issue logs, lessons learned,
  and metric payloads describing iteration efficiency and quality indicators.
- **Reporting gap:** enhanced mode currently aggregates all miniiterations per
  macroiteration, which inflates comparisons with standard mode. Planned fixes
  will normalize metrics around the final iterations of each mode.

## Repository Structure & Guardrails

- `docs/` — canonical documentation for workflows, metrics, and current
  limitations.
- `legacy/` — reference implementation targeted for refactoring. Treat it as
  read-only **except** for files under `legacy/documentation/**` and any
  `README.md` files within the directory tree.
- `AGENTS.md`, `TODO.md`, `NOTES.md` — process guardrails, queued work, and the
  decision log. Update them before making substantive changes.

## Working With the Project

1. Review `AGENTS.md`, `TODO.md`, `NOTES.md`, and the
   [`docs/simulation-overview.md`](docs/simulation-overview.md) document to align
   with current constraints and vocabulary.
2. Treat documentation updates like code changes: log intent in `NOTES.md`, add
   actionable items to `TODO.md`, and keep diffs focused and well explained.
3. Avoid modifying the `legacy/` sources until the refactoring plan unlocks
   specific modules or adapters.

## Next Steps

The current backlog prioritizes documentation polish before refactoring work:

1. Finish enriching the simulation overview with workflows, metrics, and known
   limitations.
2. Outline how to run the simulation and interpret outputs in a dedicated
   reference guide.
3. Audit the legacy inventory to catalogue immutable assets and identify
   candidates for cleanup.

Refer to `TODO.md` for the latest actionable tasks and to `NOTES.md` for the
decision history behind them.
