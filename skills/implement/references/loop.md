# The implement loop

## Picking the next task

- Candidates: tasks whose `state.yml` value is `todo`. Take the first, in id order, whose
  `depends-on` are all `done` (or `none`). Ids from review-fix phases come after the original
  ones and are picked the same way.
- A `todo` task whose dependency is `skipped` cannot run: say which and ask whether to skip
  it too (reason recorded in `tasks.md`, `state set tasks.T<n>=skipped`) or unskip the
  dependency.

## G4 stops

| `flow.gate_granularity` | Stop after |
|---|---|
| `task` | every task |
| `phase` (default) | the last task of each phase in `tasks.md` |
| `end` | the last `todo` task |

A task with `risk: high` stops after itself whatever the setting. The Verify task never
stops on its own; it is covered by step 6 of the protocol. The setting is read once at the
start of the run; a change via `/specd:config` applies at the next run.

## Commits

- Message: the task's `commit:` line, verbatim. Files: the agent's `## Changed` plus the
  files it named under `## Deviations`. Anything else `git status` shows stays unstaged and
  is mentioned in the handoff.
- `git.authority: none`: no commit; the task still flips to `done` and the handoff says the
  tree holds uncommitted work.

## Repair bounds

- Inside the agent: three attempts per failing check, then `## Blocked`.
- After all tasks: one repair dispatch on a red `run-checks`, then stop.
- A block is a handoff, not a retry: `Next` is this command, `Review` names the block.

## Resuming after `/clear`

The command is the same. `state.yml` says which tasks are `done`; `tasks.md` says what they
were. The board line at the start of the run is the whole recap the user needs.
