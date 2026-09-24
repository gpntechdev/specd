# Distilling a feature

What survives the spec folder, and where it goes.

## Durable, goes to `docs/`

- The feature record: what it does, where it lives, its ACs with evidence, how to verify.
  One page in `features/<feature>.md`.
- Decisions: every `### D<n>` in `design.md` that still holds after implementation. One file
  each in `decisions/`, numbered after the highest existing `NNNN-` file (four digits, never
  reused), status accepted, date today, context and alternatives from the design block,
  consequences updated with what the implementation showed. A row in `decisions/README.md`.
- Deltas to the architecture map: a new area, an entry point, a boundary that changed. Edit
  the area file or `overview.md` sections in place; keep the pointers current; do not
  narrate the change ("added in feature X"), state the new fact.
- Deltas to data models: new or changed entities and fields in `data-models/<domain>.md`;
  a new domain gets a file from the onboard template and a README row.

## Ephemeral, leaves with the folder

The brief, the interview trail, the task list, the review rounds, `state.yml`. The PR body
already summarises them and git history keeps the files at the delivering commit.

## Rules

- Show each patch to an existing doc before applying it; the user approves per file.
- Never add draft markers to distilled files; this run is the sign-off.
- Keep `features/<feature>.md` under 60 lines and `decisions/NNNN-*.md` under 40.
- A decision dropped or reversed during implementation is recorded as superseded, not
  deleted, when an earlier file exists; otherwise it is simply not written.
