---
name: implement
description: >
  Invoke whenever the user asks to implement, build, code or continue coding a feature that
  has an approved task list, or runs `/specd:implement [feature]`; also as the next step
  after `/specd:tasks` and after a review that sent findings back. It runs the tasks in
  `tasks.md` one at a time through the implementer agent, verifies each with the project
  checks, commits per task with the message fixed at G3, stops for your approval (G4) at the
  configured granularity and on every high-risk task, and resumes from the first open task
  after `/clear`. Not for ad-hoc coding outside a feature; that is `/specd:principles`.
---

# Skill: implement

Builds the feature task by task. The main thread orchestrates: pick a task, dispatch the
`implementer` in its own context, check its report, commit, flip the task, stop at G4 when
the granularity says so. Code never enters this conversation; the implementer's summaries do.

## Gate

G3 approved (`state check --step implement`). Re-entry after a G4 stop or a review round is
the same command: the first `todo` task is where it continues.

## Inputs

- `resolve-paths`, `detect-repo`, `state find`, `state check`, `detect-commands`.
- `<spec_root>/<feature>/tasks.md`, `state.yml`, `spec.md`, `design.md` (when present);
  `tasks/T<n>.md` when the task in hand is an index line pointing there.
- `<docs_root>/conventions.md`, [`../_shared/principles.md`](../_shared/principles.md) (path
  passed to the agent, read by it).
- `<workspace_root>/specd.yml`: `models.execution`, `flow.tdd`, `flow.gate_granularity`,
  `git.authority`, `layers.*` (files per task `layer`, relative to `code_root`).
- [`../_shared/gates.md`](../_shared/gates.md), [`../_shared/state-yml.md`](../_shared/state-yml.md).
- [`./references/implementer-prompt.md`](./references/implementer-prompt.md),
  [`./references/loop.md`](./references/loop.md).

## Outputs

- Code and tests in `<code_root>`, one commit per task (its `commit:` line).
- `tasks.T<n>` flipped in `state.yml`; `gates.G4` stamped at each stop; `step: review` at
  the end.

## Protocol

1. **Resolve.** `resolve-paths`, `detect-repo`, `state find`, `state check --step implement`
   per `gates.md`; `detect-commands`. Set `step=implement`. Read `tasks.md` and `state.yml`
   once; show the task board in one line per task (`T1 done · T2 todo …`).
2. **Pick** the first `todo` task whose `depends-on` are all `done` (`loop.md`). None left
   with open dependencies: stop and say which task blocks.
3. **Dispatch** `implementer` at `models.execution` with `implementer-prompt.md` filled: the
   task verbatim (its block from `tasks.md`, or the body of `tasks/T<n>.md` when its entry
   is an index line), the ACs it covers quoted from `spec.md`, the design headings that
   apply, absolute paths of `conventions.md` and `principles.md`, then of the files under
   `layers.<task.layer>` resolved against `code_root`, the check commands, the TDD flag. A
   `layer` with no entry in `layers`, or a listed file that does not exist: dispatch without
   it and name it in the handoff `Review`. One task per dispatch, always.
4. **Land.** Show the agent's four sections as they came back. `## Blocked` not "none":
   leave the task `todo`, end with the handoff naming the block. Otherwise run
   `git -C <code_root> status --short`; files not in `## Changed` or `## Deviations` are
   shown and left unstaged. `state set tasks.T<n>=done`, then stage the listed files plus
   `state.yml` and commit with the task's `commit:` line (authority ≠ `none`; `commit: none`
   skips the commit).
5. **G4 stop** when `loop.md` says so: granularity `task`; `phase` and this task ends its
   phase; `end` and no `todo` remains; or `risk: high`. Pass G4 per `gates.md` (summary:
   tasks landed since the last stop, files, checks run, deviations). Approve: `state set
   gates.G4=now`, continue at step 2 in this same context. Reject: handoff, `Next` is this
   command. Otherwise continue at step 2 without asking.
6. **Finish.** No `todo` left: run `"${CLAUDE_PLUGIN_ROOT}/scripts/run-checks" --from
   <code_root>`. Red: one repair dispatch with the failing output, commit its files as
   `fix(<scope>): make <check> pass`, re-run; still red: handoff with the output, `step`
   stays `implement`. Green: `state set step=review`. Whenever the run ends, a dirty
   `state.yml` is committed alone as `spec(<feature>): implement`.
7. **Handoff** per [`../_shared/handoff.md`](../_shared/handoff.md). `Review`: deviations the
   agents reported, tests marked as looking wrong. `Next: /specd:review <feature>`.

## Anti-patterns

- Writing or fixing code in the main thread "because it is a one-liner".
- Two tasks in one dispatch, or a dispatch without the task's ACs quoted.
- Pasting a layer skill's text into the prompt; the agent gets paths and reads them itself.
- Committing files the agent did not list, or amending a task's commit message.
- Skipping a G4 stop because the change looked small; the granularity is the user's setting.
- Repairing past the bound: three attempts inside the agent, one more dispatch at the end.
