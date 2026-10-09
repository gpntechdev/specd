# Distilling a feature

What survives the spec folder, and where it goes. Written by `docs` before the PR opens, so
the PR carries it; the folder itself is removed later by `close`.

## Durable, goes to `docs/`

- The feature record: what it does and how it is reached, its behaviour as shipped, the
  design's picture of it (approach summary, user flow, sequences), where it lives by role,
  the interfaces other features can use, how to verify. One page in `features/<feature>.md`,
  one row in `features/README.md`. The ACs are rewritten as grouped behaviour, without ids
  or evidence: the matrix is bookkeeping and leaves with the spec folder; a later fix
  reads the behaviour and greps the tests.
- Diagrams: the `## UX flows` and `## Sequences` blocks of `design.md` are copied verbatim
  (with `· AC<n>` stripped from labels), not redrawn; what the design called the flow is
  what shipped, unless the Notes say otherwise.
- Decisions: the `### D<n>` blocks of `design.md` that pass the durability test: a later
  feature could reasonably choose otherwise and would want to know why this one did not.
  A choice that only affects this feature's own files, or that the conventions already
  dictate, is a Notes line in the feature record, not a decision file. One file each in
  `decisions/`, numbered after the highest existing `NNNN-` file (four digits, never
  reused), status accepted, date today, context and alternatives from the design block,
  consequences updated with what the implementation showed. A row in `decisions/README.md`.
- Deltas to the architecture map: a new area, an entry point, a boundary that changed. Edit
  the area file or `overview.md` sections in place; keep the pointers current; do not
  narrate the change ("added in feature X"), state the new fact.
- Deltas to data models: new or changed entities and fields in `data-models/<domain>.md`;
  a new domain gets a file from the onboard template and a README row.

## Ephemeral, leaves with the folder

The brief, the interview trail, the task list, the review rounds, `state.yml`. The PR body
already summarises them and git history keeps the files at the delivering commit. They stay
until `close` removes the folder, after the PR is settled, because `feedback` and the fix
loop still need them.

## Refreshing after a feedback cycle

When `features/<feature>.md` already exists: update `Shipped` and `PR` (in the record and
its README row), the `Behaviour` bullets the feedback cycle changed, `Where it lives` for
files the fixes added, and `How to verify`, from the current `tasks.md`, `review.md` and
`state.yml`; add Notes lines for new accepted findings and PR replies. A decision whose
title is already in `decisions/README.md` is not written again. Architecture and data-model
deltas are asked again only for the sections the user names.

## Rules

- Show each patch to an existing doc before applying it; the user approves per file.
- Never add draft markers to distilled files; this run is the sign-off.
- Keep `features/<feature>.md` under 150 lines, diagrams included, and
  `decisions/NNNN-*.md` under 40; a README row is one line.
- A decision dropped or reversed during implementation is recorded as superseded, not
  deleted, when an earlier file exists; otherwise it is simply not written.
