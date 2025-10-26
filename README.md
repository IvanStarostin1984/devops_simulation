# DevOps Simulation

A lightweight environment for exploring two DevOps operating modes—standard and
enhanced—so we can document the current behavior, plan refactors, and evaluate
future improvements.

## Quick Summary

- **Standard mode** (`backward compatibility = true`) runs a single
  miniiteration per macroiteration and delays feedback until the next major
  cycle.
- **Enhanced mode** (`backward compatibility = false`) allows multiple
  miniiterations inside a macroiteration so issues and lessons learned can be
  applied immediately.
- Both modes operate on two JSON inputs: one describing the client/domain
  specification and another configuring simulation parameters.
- See the detailed [simulation overview](docs/simulation-overview.md) for
  workflows, data flow, intended metrics, and known reporting gaps.

## Repository Structure & Guardrails

- `docs/` — canonical documentation of modes, workflows, and metrics.
- `legacy/` — reference implementation destined for refactoring. Treat the
  directory as read-only **except** for `legacy/documentation/**` and any
  `README.md` files.
- `AGENTS.md`, `TODO.md`, `NOTES.md` — collaboration process, queued work, and
  decision log respectively.

## Current Focus

1. **Document the existing behavior** (in progress).
2. Plan the refactoring strategy for the legacy code base.
3. Execute the refactor while preserving functionality.
4. Plan targeted enhancements once the code is modular.
5. Implement enhancements and validate with updated metrics.

## Documentation Expectations

- Keep `docs/simulation-overview.md` as the single source of truth for how the
  simulation operates; link to it from any future developer guides or onboarding
  materials.
- Record decisions in `NOTES.md` and track actionable work in `TODO.md` before
  making changes.

## Getting Started

1. Read `AGENTS.md`, `TODO.md`, `NOTES.md`, and `docs/simulation-overview.md` to
   understand constraints and goals.
2. Treat documentation updates as code: follow the process logs and submit PRs
   with clear rationale and test evidence when applicable.
3. Avoid modifying the `legacy/` sources until a refactoring plan is approved.

