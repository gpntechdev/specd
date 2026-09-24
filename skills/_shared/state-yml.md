# state.yml

**Reference-only.** Not a skill. The per-feature state file at `<spec_root>/<feature>/state.yml`.
Created by the step that opens the feature (`start`, or `scaffold` for the skeleton) and
patched by every later step; read by every step that needs to know where the feature stands.
Flat two-level YAML, like `specd.yml`, so the same parser rules apply. The only writer is
`"${CLAUDE_PLUGIN_ROOT}/scripts/state"`; no skill edits this file with a text tool.

## Default file

`state init` copies this verbatim, then sets `feature`, `tier`, `step`, `retention`, `branch`,
`created` and `updated`.

```yaml
feature: ""                  # folder name under spec_root: slug or ticket key
tier: full                   # trivial | quick | full
step: start                  # start | specify | design | tasks | implement | review | verify
                             # | deliver | close | closed
retention: ""                # distill | clean | keep; empty = project default for the tier
branch: ""                   # git branch created by start; empty when git.authority is none
pr: ""                       # draft PR or MR URL written by deliver

gates:                       # ISO-8601 UTC timestamp when approved, empty otherwise
  G1: ""                     # spec approved
  G2: ""                     # design approved
  G3: ""                     # task list approved
  G4: ""                     # implementation approved (re-stamped at every G4 stop)
  G5: ""                     # diff and review approved for delivery

triage:                      # written by start
  proposed: ""               # tier the triage proposed before the user confirmed
  signals: ""                # comma-separated signals that fired, empty when none

review:                      # written by review
  round: 0                   # review rounds completed
  open: 0                    # findings sent back to implement in the last round

tasks: {}                    # T1: todo | done | skipped; mirrors tasks.md, one key per task

created: ""                  # ISO-8601 UTC
updated: ""                  # ISO-8601 UTC; every writer sets it
```

## The script

```
state init  --file PATH --feature NAME --tier TIER [--retention R] [--step S] [--branch B]
state get   --file PATH [key ...]           # JSON of the whole file or the named keys
state set   --file PATH key=value ...       # patch in place; `now` = current UTC timestamp
state check --file PATH --step STEP         # may STEP run? exit 1 with run_first otherwise
state find  --spec-root DIR [--feature NAME] [--branch BRANCH]
state list  --spec-root DIR                 # every feature: step, gates, task counts
```

`set` may add `tasks.<id>` keys that do not exist yet; every other key must already be in
the file. All-or-nothing: a refused key leaves the file untouched.

## Finding the feature

Every step takes an optional feature argument. Resolve it with `state find`, in this order:

1. The argument, when given; it must name an existing folder under `<spec_root>`.
2. The current git branch (from `detect-repo`), when exactly one feature's `branch` equals it.
3. The only feature whose `step` is not `closed`.
4. Otherwise the script lists the open candidates; ask the user which one and re-run.

## Rules for readers

- A missing key means its default above. Unknown keys are ignored, never removed.
- A malformed file: stop and show the problem. State is too important to guess.
- `step` names the step that runs next or is in progress, not the last one finished. A step
  sets `step` to its own name when it starts and to the next step's name when it ends.
- Run `state check --step <name>` before anything else; it encodes the step order and the
  gate each step depends on (see [`gates.md`](./gates.md)).

## Rules for writers

- Patch key by key through `state set`; never regenerate the file.
- `tasks` keys are the task ids from `tasks.md` (`T1`, `T2` …). `tasks` adds them when
  `tasks.md` is approved; `implement` flips each to `done` as it lands, `skipped` with a
  reason in `tasks.md`; `review` appends ids for review fixes.
- `updated` is set by the script on every write.
- Owners of the M2 keys: `branch`, `triage.*` by `start`; `pr` by `deliver`; `review.*` by
  `review`. Later milestones add keys here and record them in `docs/DECISIONS.md`.
