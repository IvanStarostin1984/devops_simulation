# Task Log: Align backlog references with legacy documentation paths

- **Prompt**: Address review feedback ensuring all legacy simulator documentation
  references reside under `legacy/documentation/**` and update any lingering
  links.
- **Reformulation**: Update project guidance so TODO backlog entries and related
  docs point at the relocated legacy documentation paths instead of the removed
  `docs/` tree.
- **Assumptions**:
  - Only Markdown guidance files (README, TODO, NOTES) reference these documentation paths.
  - Historical NOTES entries remain immutable even if they cite outdated paths, per repository policy.
- **Constraints**:
  - Follow repository documentation style (≤100 columns, markdownlint-friendly formatting).
  - Preserve NOTES.md ordering and append-only behavior.
  - Maintain TODO.md structure and link formatting.
- **Stakeholders**:
  - Repository contributors relying on TODO.md for next actions.
  - Reviewers ensuring documentation stays co-located with legacy assets.
- **Goals**:
  1. Update TODO.md backlog items to reference `legacy/documentation/**`
     exclusively.
  2. Verify README.md still reflects the relocation and contains no stale
     `docs/` links to legacy docs.
  3. Record plan and implementation entries in NOTES.md per workflow
     expectations.
- **Acceptance Criteria**:
  - TODO.md contains only `legacy/documentation/**` paths when directing
    contributors to legacy docs.
  - README.md references remain accurate without pointing to removed `docs/`
    files.
  - NOTES.md logs planning and implementation outcomes for this adjustment.
- **Plan** (Mode: one-pass; Scope: Markdown guidance updates; Risks: missing a
  stale link leading to confusion; Acceptance tests: search repo for outdated
  `docs/legacy` references, review README.md; Deliverables: updated TODO.md,
  README verification, NOTES entries, .agent log):
  1. Append planning entry to NOTES.md and add a TODO.md task describing the
     cleanup.
  2. Update TODO.md references from `docs/legacy-simulator/**` to
     `legacy/documentation/legacy-simulator/**` and ensure language matches the
     relocation.
  3. Review README.md to confirm no outdated links remain.
  4. Mark the TODO item complete, append implementation entry to NOTES.md, and
     document actions here.

## Actions Taken
- Logged planning intent in NOTES.md and added the TODO backlog entry for link
  realignment.
- Replaced the lingering `docs/legacy-simulator/**` references in TODO.md with
  `legacy/documentation/legacy-simulator/**` and ensured acceptance criteria
  reflect the change.
- Reviewed README.md to confirm its guidance already points at the legacy
  documentation tree.
- Marked the TODO item complete and captured implementation details in NOTES.md.

## Diffs
- `TODO.md`: Updated backlog task to use the legacy documentation paths and
  marked the cleanup complete.
- `NOTES.md`: Added planning and implementation entries for the link
  realignment work.

## Test Results
- Not applicable (documentation-only change).

## Metrics / Time Tracking
- Not tracked.

## Blockers / Issues
- None encountered.

## Next Steps
- None required; backlog guidance now matches the relocated documentation
  structure.
