# state.yml

**Reference-only.** Not a skill. The per-feature state file at `<spec_root>/<feature>/state.yml`.
Written by the step that creates the feature (`start`, or `scaffold` for the skeleton) and
patched by every later step; read by every step that needs to know where the feature stands.
Flat two-level YAML, like `specd.yml`, so the same parser rules apply.

## Default file

The creating step writes this verbatim, then sets `feature`, `tier`, `created` and `updated`.

```yaml
feature: ""                  # folder name under spec_root: slug or ticket key
tier: full                   # trivial | quick | full
step: start                  # start | specify | design | tasks | implement | review | verify
                             # | deliver | close | closed
retention: ""                # distill | clean | keep; empty = project default for the tier

gates:                       # ISO-8601 UTC timestamp when approved, empty otherwise
  G1: ""                     # spec approved
  G2: ""                     # design approved
  G3: ""                     # task list approved
  G4: ""                     # implementation approved (last stop of the granularity)
  G5: ""                     # diff and review approved for delivery

tasks: {}                    # T1: todo | done | skipped; mirrors tasks.md, one key per task

created: ""                  # ISO-8601 UTC
updated: ""                  # ISO-8601 UTC; every writer sets it
```

## Rules for readers

- A missing key means its default above. Unknown keys are ignored, never removed.
- A malformed file: stop and show the problem. State is too important to guess.
- `step` names the step that runs next or is in progress, not the last one finished. A step
  sets `step` to its own name when it starts and to the next step's name when it ends.

## Rules for writers

- Patch key by key; never regenerate the file. Comments and unknown keys survive.
- `tasks` keys are the task ids from `tasks.md` (`T1`, `T2` …). Add them when `tasks.md` is
  written; flip each to `done` as it lands, `skipped` with a reason in `tasks.md`.
- Set `updated` on every write.
- Later milestones add keys here (triage signals, review rounds); they are recorded in
  `docs/DECISIONS.md` when they arrive.
