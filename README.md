# devops_simulation
simulation of two modes of devops
Must run with any client specifications from any domain/domains

Legacy folder and it's contents are read only, except contents of subfolder legacy/documentation and contents of README.md files in folder legacy and all it's subfolders. It is source code that must be refactored into clear modular code with the same behavior (+ some improvements, but at least the same behavior) and will be executed in Visual Studio 2022. Refer to `legacy/documentation/assets.md` for a full inventory of legacy inputs, outputs, and dependencies that must remain immutable outside the documented exceptions.
Aim of this code is multiple runs in standard DEVOPS mode (backword compatibility = true) and enhanced devops (backword compatibility = false), produce metrics, which will be later used to compare standard devops and enhanced devops. Both modes act on 2 json files.
Explanation/context:
Generally, standard devops is process of generation of software engineering documentation, when macroiterations may be from one to many, miniiterations always = 1, issues are being detected after each artifact/table is being generated, but are being fed into creator prompt only on next macroiteration. Lessons are learned only in the end of macroiteration, and shown to corresponding artifacts creator prompt only in next macroiteration.
In contrast, in enhanced devops  when macroiterations may be from one to many, miniiterations may be from one to many, issues are being detected after each artifact/table is being generated, and are being fed into creator prompt in next miniiteration. Lessons are learned only in the end of each miniiteration and macroiteration, and shown to corresponding artifacts creator prompt in next miniiteration/macroiteration.
As issues are being provided to creater prompt earlier and lessons learned are generated and provided to creator prompt earlier, enhanced devops is expected to produce artifact of higher quality (which I think may be reflected in either higher tabular complexity of artifacts of last macroiteration of enhanced devops vs standard devops, or lower concentration of issues of last macroiteration). However, as for now reporting system is working, it seems that it is not correctly implemented - each macroiteration of enhanced devops may contain more than one miniiteration, while standard devops has one only, wich skewes results of assessment of both modes and makes them incomparable. So later some additional reporting mode/layer? must be implemented, comparing really only last set of mini of last macro of enhanced devops mode vs last macro of standard devops, making them comparable.
However, to keep process starigtforward, we must proceed with the project in nexxt steps:
1. Document fully current behavior
2. Plan refactoring
3. Refactor
4. Plan enhancements
5. Implement enhancements. 
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

