# The implement loop

## Arguments

`/specd:implement [feature] [tasks]`. Every token after the command that matches `T<n>` or
`T<n>-T<m>`, separated by commas and/or spaces, belongs to the selection: `T1-T3`,
`T1, T4, T5`, `T1-T3,T5`. A range expands by id number, both ends included. The one remaining
token, if any, is the feature; with none, `state find` resolves it as usual, so
`/specd:implement T1-T3` works on the current branch. No selection: every `todo` task.

Validating a selection, before anything is dispatched:

- An id not in `tasks.md`: refuse in one line that lists the known ids.
- A `done` or `skipped` id: dropped, said in the board line. Nothing left: say so and stop,
  without the finish step.
- A selected task whose `depends-on` has a task that is neither `done` nor selected: refuse
  in one line, `T4 depends on T3 (todo, not selected); run with T1, T3, T4 or without a
  selection`. Never widen the selection on the user's behalf.

## Reading a task

`tasks.md` holds the phases, one block per task, and the matrix. A heavy task is an index
line ending in `tasks/T<n>.md` instead of a block; read that file for its body before
dispatching, and nothing else from the folder. Phase boundaries for G4 come from `tasks.md`
either way.

## Picking the next task

- Candidates: tasks whose `state.yml` value is `todo`, within the selection when one was
  given. Take the first, in id order, whose `depends-on` are all `done` (or `none`). Ids from
  review-fix phases come after the original ones and are picked the same way.
- A `todo` task whose dependency is `skipped` cannot run: say which and ask whether to skip
  it too (reason recorded in `tasks.md`, `state set tasks.T<n>=skipped`) or unskip the
  dependency.

## G4 stops

| Stop | When | Commit of the stopping task |
|---|---|---|
| `flow.gate_granularity: task` | after every task | after approve |
| `phase` (default) | after the last task of each phase in `tasks.md` | before the stop |
| `end` | after the last `todo` task | before the stop |
| `risk: high` | after that task, whatever the setting | after approve |
| last selected task | at the end of a selected run, whatever the setting | per the rows above |

The Verify task never stops on its own; it is covered by step 6 of the protocol. The setting
is read once at the start of the run; a change via `/specd:config` applies at the next run.

At a stop whose task is still uncommitted ("after approve" rows): **approve** flips it to
`done` and commits it; **edit** re-dispatches it with the user's request (revision block in
`implementer-prompt.md`), lands the result in the tree and asks again; **reject** leaves it
`todo`, its files uncommitted in the tree, and ends with the handoff. At a `phase` or `end`
stop the landed tasks are already committed; an edit there lands as one more commit per
revised task, `<commit line> (G4 edit)`, prefixed per the rule below.

## Commits

- Prefix rule: while `pr` in `state.yml` is empty, every code commit this skill makes
  starts with `WIP: `; the prefix marks what `deliver` may squash into one commit before
  the first push. Once `pr` is set (fixes from `feedback`, or any later task), there is no
  prefix: those commits are pushed to the open PR as they are and stay in the history.
- Message: the task's `commit:` line, verbatim, under the prefix rule. Files: the agent's `## Changed`
  plus the files it named under `## Deviations`, plus `state.yml` with the task flipped to
  `done`. Anything else `git status` shows stays unstaged and is mentioned in the handoff.
- The end-of-run repair commits as `fix(<scope>): make <check> pass` (same prefix rule), scope as in the
  feature's tasks. G4 stamps and the final `step=review` ride in `spec(<feature>): implement`
  when the run ends with `state.yml` dirty.
- `git.authority: none`: no commit; the task still flips to `done` and the handoff says the
  tree holds uncommitted work.

## Repair bounds

- Inside the agent: three attempts per failing check, then `## Blocked`.
- After all tasks: one repair dispatch on a red `run-checks`, then stop.
- A block is a handoff, not a retry: `Next` is this command, `Review` names the block.
- G4 edits are not bounded; the user drives them.

## Resuming after `/clear`

The command is the same. `state.yml` says which tasks are `done`; `tasks.md` says what they
were. The board line at the start of the run is the whole recap the user needs. Uncommitted
files left by a rejected stop appear under the board; the next dispatch of that task names
them in the earlier-attempt line so the agent continues from them instead of starting over.
