---
name: close
description: >
  Invoke whenever the user asks to close, finish, wrap up or clean up a delivered feature
  whose pull request is settled, or runs `/specd:close [feature]`; the last step before the
  merge. It applies the retention policy: distill (the docs step must have written the
  feature record; then delete the spec folder), clean (delete the spec folder) or keep
  (mark closed and leave it), commits and pushes so the open PR carries the result. Not for
  closing the PR itself; merging is yours, after this push.
---

# Skill: close

Ends the feature's working set. The durable knowledge was written by `docs` before the PR;
the spec folder is ephemeral and leaves here, once nobody needs it: run it when the PR has
no open comments left, then merge. Git history is the archive. The default policy comes
from the tier and `flow.retention`; the user confirms it per feature.

## Gate

`state check --step close`: `step` is `close` (delivered) or `closed` (kept earlier; close
again to clean it). A merged PR is refused: the folder is already on the default branch.

## Inputs

- `resolve-paths`, `detect-repo` (remote host), `state find`, `state check`.
- `<spec_root>/<feature>/state.yml` (`retention`, `tier`, `pr`, `branch`).
- `<docs_root>/features/<feature>.md` (must exist for `distill`).
- `<workspace_root>/specd.yml`: `flow.retention`, `git.authority`, `git.pr_host`.
- [`../_shared/state-yml.md`](../_shared/state-yml.md).

## Outputs

- distill or clean: spec folder removed. keep: `step: closed`.
- One commit `docs(<feature>): close (<policy>)`, pushed when authority ≥ `push` and the
  branch is on the remote.

## Protocol

1. **Resolve.** `resolve-paths`, `detect-repo`, `state find`, `state check --step close`.
   When `pr` is set and the host CLI exists (`gh pr view <url> --json state`, `glab mr view
   <url>`): state `MERGED` → refuse in one line (the spec folder is on `<default>`; remove
   it on a branch by hand) and end with the handoff. CLI missing: continue. Policy =
   `retention` in `state.yml`, else `clean` for trivial and quick, `flow.retention` for
   full. Confirm in one question: distill / clean / keep, with the default first.
2. **distill.** `<docs_root>/features/<feature>.md` missing: stop, handoff `Next:
   /specd:docs <feature>` (the record is written there, then close again). Present:
   `git rm -r <spec_root>/<feature>`.
3. **clean.** `git rm -r <spec_root>/<feature>`.
4. **keep.** `state set step=closed retention=keep`. The folder stays; `status` warns after
   `flow.keep_days`.
5. **Commit** `docs(<feature>): close (<policy>)` in the repo containing the paths; push when
   authority ≥ `push` and `branch` is set.
6. **Handoff** per [`../_shared/handoff.md`](../_shared/handoff.md). `Produced`: the removal
   or the closed marker. `Review`: none. `Next: /specd:status`; say the PR (from `pr`, if
   any) now carries the removal and that merging is the user's call, after this push.

## Anti-patterns

- Writing the feature record or decisions here; that is `docs`, before the PR.
- Closing while PR comments are still open; `feedback` needs the folder.
- Keeping "just in case"; keep is a choice with a reason, and `status` will nag.
- Renaming or rebasing the branch; the removal is one more commit on it.
