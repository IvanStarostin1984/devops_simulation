# Task Log: Relocate legacy simulator documentation under legacy/documentation

- **Prompt**: User requested keeping all as-is legacy simulator documentation inside the `legacy/` folder and ensuring references stay accurate.
- **Reformulation**: Move the legacy-focused docs currently under `docs/` into `legacy/documentation/`, then update all links and references so they remain valid.
- **Assumptions**:
  - Only documentation about the legacy simulator needs relocation; other general docs remain in `docs/`.
  - Moving files will not break build or tooling since there are no automated scripts consuming these paths.
- **Goals**:
  1. Relocate `docs/simulation-overview.md` and the `docs/legacy-simulator/` directory beneath `legacy/documentation/`.
  2. Update repository references (README, NOTES, TODO, cross-doc links) to point to the new paths.
  3. Verify no references remain to the old `docs/` locations.
- **Acceptance Criteria**:
  - All legacy simulator documentation files reside under `legacy/documentation/`.
  - Internal and external links resolve to the relocated files.
  - TODO and NOTES entries reflect the completed move.
  - `docs/` no longer contains legacy-specific docs.
- **Plan** (Mode: one-pass; Scope: relocate documentation and update references; Risks: missing a reference, causing broken link; Acceptance tests: manually inspect repository for stale paths; Deliverables: relocated files, updated links, updated TODO/NOTES entries):
  1. Move the relevant files into `legacy/documentation/` (create subdirectories as needed).
  2. Update README.md, TODO.md, NOTES.md, and cross-doc links to match the new structure.
  3. Search for old paths to confirm they no longer exist.
  4. Record changes, run formatting checks if applicable, and document the outcome.

## Actions Taken
- Created a plan entry in `NOTES.md` and corresponding TODO checklist item.
- Moved `docs/simulation-overview.md` and `docs/legacy-simulator/**` into
  `legacy/documentation/`.
- Updated README.md, TODO.md, and the new documentation paths for accurate
  cross-references.
- Added an implementation log entry in `NOTES.md` and marked the TODO item
  complete.
- Populated this `.agent` log with execution details.

## Diffs
- `README.md`: point overview links to `legacy/documentation/**` and clarify the
  now-empty `docs/` area.
- `TODO.md`: mark the relocation task complete and retarget all legacy doc
  references.
- `NOTES.md`: record planning and implementation entries for the relocation.
- `legacy/documentation/simulation-overview.md`: adjust related-doc links for
  the new directory.
- `legacy/documentation/legacy-simulator/`: add `artifacts.md`,
  `configuration.md`, and `flow.md` relocated from `docs/`.
- `.agent/`: add workspace index and task log per custom instructions.

## Test Results
- Not applicable (documentation-only change).

## Metrics / Time Tracking
- Not tracked for this task.

## Blockers / Issues
- None identified.

## Next Steps
- Monitor for future documentation updates that should also live under
  `legacy/documentation/`.
