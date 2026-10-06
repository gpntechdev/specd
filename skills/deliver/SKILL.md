---
name: deliver
description: >
  Invoke whenever the user asks to deliver, ship, push or open the pull request for a
  verified feature, or runs `/specd:deliver [feature]`; also as the next step after
  `/specd:verify`. It re-runs the project checks, shows you the full change (commits, diff
  stat, review verdict, evidence matrix) for the delivery gate (G5), then goes as far as
  `git.authority` allows: push the branch and open a draft PR or MR with a body that links
  the spec and the review, in your voice, with no AI attribution. Before the first push it
  offers to squash the `WIP:` task commits into one commit; you choose per run whether to
  push and whether to open the PR. Re-run after a feedback cycle it pushes to the open PR.
  It never merges.
---

# Skill: deliver

The last gate and the only step that touches the remote. Everything it pushes already exists
as commits; its own work is the G5 view, the optional squash, the push and the draft PR.

## Gate

`state check --step deliver`: `step` is exactly `deliver` (docs is done). G5 is passed here,
again on every re-run after a feedback cycle.

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
- The branch on the remote and a draft PR or MR, as far as the user chose within authority.

## Protocol

1. **Resolve.** `resolve-paths`, `detect-repo`, `state find`, `state check --step deliver`
   per `gates.md`.
2. **Checks.** `"${CLAUDE_PLUGIN_ROOT}/scripts/run-checks" --from <code_root>`. Red: refuse
   with the failing tail, handoff `Next: /specd:implement <feature>`.
3. **G5 view.** `git -C <code_root> log <default>..HEAD --oneline` (and, when the branch
   has an upstream, `git log @{u}..HEAD --oneline` as "since the last push"), `git diff
   --stat <default>...HEAD`, the last review or PR round's verdict and accepted findings
   from `review.md`, the matrix summary from `tasks.md` (ACs total, with test evidence,
   with a check, manual, accepted), the docs written by `docs` or "none". Uncommitted
   changes in the tree: list them and stop; they are committed by `implement` or by the
   user, never here.
4. **Squash choice.** When the log holds `WIP:` commits, `git.authority` is not `none` and
   the branch has no upstream: ask one question, **squash** into one commit (show the title
   per `git.md`) or **keep** the commits as they are. Squash: run `squash-wip --dry-run`,
   show its subjects, run it for real, show the new log line and the `before` sha as the
   recovery point. Branch already pushed, or authority `none`: no question, one line saying
   why the `WIP:` commits stay. Then pass G5 per `gates.md` on the branch as it now is.
5. **How far.** One question, options in this order and only those `git.authority`
   allows (`git.md`): **push + draft PR** (authority `draft_pr`, and `pr` not yet set),
   **push** (≥ `push`), **record only** (G5 stamped, nothing leaves). Default is the first
   listed. `authority: none`: no question, record only. `pr` already set: the push updates
   the open PR; say so with its URL and never open another. Push per `git.md`. For the PR
   render `./templates/pr-body.md` (title from `spec.md`'s problem line, what changed from
   the commit list or, after a squash, from the squash commit's body, how it was verified
   from the matrix, spec, docs and review paths, `Refs:` line when `git.ticket_key` names a
   ticket) and open the draft with the host's CLI. CLI missing or `git.pr_host: none`:
   push, then print the exact command and the compare URL.
6. **Record.** `state set gates.G5=now pr=<url or unchanged> step=close`, commit
   `spec(<feature>): deliver`, push again when a push was chosen so the state commit is on
   the remote.
7. **Handoff** per [`../_shared/handoff.md`](../_shared/handoff.md). `Produced`: the PR URL.
   `Review`: what the PR body claims; the `before` sha when a squash ran. `Next:
   /specd:close <feature>` once the PR is settled, then merge; second line
   `/specd:feedback <feature>` when comments arrive.

## Anti-patterns

- Any mention of an AI assistant, model or tool in the title, body or commits.
- `--force`, rebasing or merging; the only history rewrite is `squash-wip`, before the
  first push and on the user's choice.
- Pushing before G5, opening a non-draft PR, or a second PR for a feature whose `pr` is set.
- Going further than the user chose this run, whatever `git.authority` would allow.
- Writing a PR body from memory instead of from `spec.md`, `review.md` and the matrix.
