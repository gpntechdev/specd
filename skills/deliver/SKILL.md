---
name: deliver
description: >
  Invoke whenever the user asks to deliver, ship, push or open the pull request for a
  verified feature, or runs `/specd:deliver [feature]`; also as the next step after
  `/specd:verify`. It re-runs the project checks, shows you the full change (commits, diff
  stat, review verdict, evidence matrix) for the delivery gate (G5), then goes as far as
  `git.authority` allows: push the branch and open a draft PR or MR with a body that links
  the spec and the review, in your voice, with no AI attribution. Before the first push it
  offers to squash the `WIP:` task commits into one commit. It never merges.
---

# Skill: deliver

The last gate and the only step that touches the remote. Everything it pushes already exists
as commits; its own work is the G5 view, the optional squash, the push and the draft PR.

## Gate

`state check --step deliver`: `step` is exactly `deliver` (verify is done). G5 is passed here.

## Inputs

- `resolve-paths`, `detect-repo` (default branch, remote host, remote URL), `state find`,
  `state check`, `run-checks`, `squash-wip`.
- `<spec_root>/<feature>/state.yml`, `spec.md`, `review.md`, `tasks.md` (matrix).
- `<workspace_root>/specd.yml`: `git.authority`, `git.pr_host`, `git.ticket_key`.
- [`../_shared/no-attribution.md`](../_shared/no-attribution.md),
  [`../_shared/gates.md`](../_shared/gates.md), [`../_shared/state-yml.md`](../_shared/state-yml.md).
- `./templates/pr-body.md`, [`./references/git.md`](./references/git.md).

## Outputs

- `gates.G5`, `pr`, `step: close` in `state.yml`; one commit `spec(<feature>): deliver`.
- The branch on the remote and a draft PR or MR, as far as authority allows.

## Protocol

1. **Resolve.** `resolve-paths`, `detect-repo`, `state find`, `state check --step deliver`
   per `gates.md`.
2. **Checks.** `"${CLAUDE_PLUGIN_ROOT}/scripts/run-checks" --from <code_root>`. Red: refuse
   with the failing tail, handoff `Next: /specd:implement <feature>`.
3. **G5 view.** `git -C <code_root> log <default>..HEAD --oneline`, `git diff --stat
   <default>...HEAD`, the last review round's verdict and accepted findings from
   `review.md`, the matrix summary from `tasks.md` (ACs total, with test evidence, with a
   check, manual, accepted). Uncommitted changes in the tree: list them and stop; they are
   committed by `implement` or by the user, never here.
4. **Squash choice.** When the log holds `WIP:` commits, `git.authority` is not `none` and
   the branch has no upstream: ask one question, **squash** into one commit (show the title
   per `git.md`) or **keep** the commits as they are. Squash: run `squash-wip --dry-run`,
   show its subjects, run it for real, show the new log line and the `before` sha as the
   recovery point. Branch already pushed, or authority `none`: no question, one line saying
   why the `WIP:` commits stay. Then pass G5 per `gates.md` on the branch as it now is.
5. **Authority.** Read `git.authority` and follow `git.md`: `none` or `commit` → record G5,
   say what push and PR would do, stop. `push` → push the branch. `draft_pr` → push, then
   render `./templates/pr-body.md` (title from `spec.md`'s problem line, what changed from
   the commit list or, after a squash, from the squash commit's body, how it was verified
   from the matrix, spec and review paths, `Refs:` line when `git.ticket_key` names a
   ticket) and open the draft with the host's CLI. CLI missing or `git.pr_host: none`:
   print the exact command and the compare URL, stop at `push`.
6. **Record.** `state set gates.G5=now pr=<url or empty> step=close`, commit
   `spec(<feature>): deliver`, push again so the state commit is on the remote.
7. **Handoff** per [`../_shared/handoff.md`](../_shared/handoff.md). `Produced`: the PR URL.
   `Review`: what the PR body claims; the `before` sha when a squash ran. `Next:
   /specd:close <feature>`; add that merging is the user's call and that `close` pushes the
   distilled docs onto the same branch.

## Anti-patterns

- Any mention of an AI assistant, model or tool in the title, body or commits.
- `--force`, rebasing or merging; the only history rewrite is `squash-wip`, before the
  first push and on the user's choice.
- Pushing before G5, or opening a non-draft PR.
- Writing a PR body from memory instead of from `spec.md`, `review.md` and the matrix.
