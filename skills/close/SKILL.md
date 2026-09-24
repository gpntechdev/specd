---
name: close
description: >
  Invoke whenever the user asks to close, finish, wrap up, distil or clean up a delivered
  feature, or runs `/specd:close [feature]`; also as the next step after `/specd:deliver`.
  It applies the retention policy: distill (write the feature record, decision files and
  doc deltas into the project docs, then delete the spec folder), clean (delete the spec
  folder) or keep (mark closed and leave it), commits and pushes so the open PR carries the
  result. Not for closing the PR itself; merging is yours.
---

# Skill: close

Ends the feature's working set. Durable knowledge goes to `<docs_root>/`; the spec folder is
ephemeral and leaves; git history is the archive. The default policy comes from the tier
and `flow.retention`; the user confirms it per feature.

## Gate

`state check --step close`: `step` is `close` (delivered) or `closed` (kept earlier; close
again to distill or clean it).

## Inputs

- `resolve-paths`, `state find`, `state check`.
- `<spec_root>/<feature>/state.yml`, `spec.md`, `design.md`, `tasks.md`, `review.md`.
- `<docs_root>/decisions/README.md`, `architecture/overview.md` and areas, `data-models/`,
  `features/`.
- `<workspace_root>/specd.yml`: `flow.retention`, `git.authority`.
- [`../_shared/state-yml.md`](../_shared/state-yml.md), `./templates/feature.md`,
  [`../_shared/templates/decision.md`](../_shared/templates/decision.md),
  [`./references/distill.md`](./references/distill.md).

## Outputs

- distill: `<docs_root>/features/<feature>.md`, `decisions/NNNN-*.md` with README rows,
  approved deltas in architecture or data-model files; spec folder removed.
- clean: spec folder removed. keep: `step: closed`.
- One commit `docs(<feature>): close (<policy>)`, pushed when authority ≥ `push` and the
  branch is on the remote.

## Protocol

1. **Resolve.** `resolve-paths`, `state find`, `state check --step close`. Policy =
   `retention` in `state.yml`, else `clean` for trivial and quick, `flow.retention` for full.
   Confirm in one question: distill / clean / keep, with the default first.
2. **distill.** Per `distill.md`: (a) copy `./templates/feature.md` to
   `<docs_root>/features/<feature>.md` and fill it from `spec.md`, `tasks.md`, `review.md`
   and `state.yml` (PR URL); (b) for each `### D<n>` in `design.md`, copy
   `_shared/templates/decision.md` to `<docs_root>/decisions/NNNN-<kebab-title>.md` with the
   next free number, status accepted, and append its row to `decisions/README.md`; (c) for a
   `## Data model` or `## API and contracts` section, name the `architecture/<area>.md` or
   `data-models/<domain>.md` it changes and propose one patch per file: show it, ask, apply.
   No draft markers on any of these; they are signed off by this run. Then `git rm -r` the
   spec folder.
3. **clean.** `git rm -r <spec_root>/<feature>`.
4. **keep.** `state set step=closed retention=keep`. The folder stays; `status` warns after
   `flow.keep_days`.
5. **Commit** `docs(<feature>): close (<policy>)` in the repo containing the paths; push when
   authority ≥ `push` and `branch` is set.
6. **Handoff** per [`../_shared/handoff.md`](../_shared/handoff.md). `Produced`: the docs
   written. `Review`: decisions recorded, deltas applied. `Next: /specd:status`; say the PR
   (from `state.yml` `pr`, if any) now carries the docs and that merging is the user's call.

## Anti-patterns

- Copying `spec.md` into `features/<feature>.md`; the record is what shipped, in a page.
- Recording a decision that has no alternatives, or renumbering existing decisions.
- Editing an architecture or data-model doc without showing the patch first.
- Keeping "just in case"; keep is a choice with a reason, and `status` will nag.
