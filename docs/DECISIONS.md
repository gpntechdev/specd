# Decisions

Running log of decisions that change or refine `docs/PLAN.md`. Newest last. Each entry:
context (why it came up), decision, consequences. The plan stays the source for the design;
this file is the diff history of its judgement calls.

## 2026-09-22 M0: specd.yml schema is an annotated YAML reference

Context: the schema needed a home that both skills and humans read, before any init code exists.
Decision: `skills/_shared/specd-yml.md` holds the default file with a comment per key plus
reader and writer rules; no JSON Schema.
Consequences: one source, copied verbatim by `init`; machine validation of user config, if ever
needed, is a later script reading the same file.

## 2026-09-22 M0: principles text lives in `_shared`, the skill is a protocol spine

Context: PLAN 3.9 lists principles among shared blocks; the skill is also the trigger surface.
Decision: `skills/_shared/principles.md` is canonical; `skills/principles/SKILL.md` is a short
protocol that reads it. Other skills link the shared file, not the skill.
Consequences: one extra file read when the skill fires; no duplicated text.

## 2026-09-22 M0: scripts are stdlib-only Python 3

Context: macOS ships Python 3.9 without PyYAML; the validator and path helper must run anywhere.
Decision: `scripts/*` use the standard library only. Config is read by a flat-key parser that
covers the subset specd.yml uses; full YAML parsing is not needed.
Consequences: `specd.yml` stays flat (two levels, scalar values, simple lists) so the parser
keeps working. A key needing deeper nesting is a decision to revisit here.

## 2026-09-22 M0: path resolution is a script, not prose

Context: every skill resolves `workspace_root`, `code_root`, `docs_root`, `spec_root`.
Decision: `scripts/resolve-paths` prints them as JSON; `skills/_shared/paths.md` documents the
rules and tells skills to run it.
Consequences: skills never reason about folder layout; wrapper and embedded are one code path.

## 2026-09-22 M0: no-attribution is a workspace setting written by init

Context: a plugin-level `settings.json` supports only `agent` and `subagentStatusLine`, so the
plugin cannot switch attribution off for its users.
Decision: `skills/_shared/no-attribution.md` holds the `attribution` snippet `init` writes into
each workspace's `.claude/settings.json`; this repo carries the same snippet for itself.
Consequences: attribution is off only after `init`; deliver enforces the wording rules regardless.

## 2026-09-22 M0: `git.docs_repo` dropped from specd.yml

Context: PLAN 2 says wrapper mode chooses at init whether `docs/` lands in the wrapper or the
code repo. A separate key would duplicate what `paths.docs_root` already says.
Decision: `init` writes `paths.docs_root` to the chosen folder; a skill committing a docs change
picks the repo that contains the path.
Consequences: one key fewer; the repo choice is derived, never stated twice.

## 2026-09-22 M0: marketplace and plugin are both named `specd`

Context: the repo serves itself as a marketplace with one plugin.
Decision: marketplace `specd`, plugin `specd`, install as `specd@specd`; `source: "./"`.
Consequences: a second plugin in this repo would need a `plugins/` subfolder and a rename of
`source`; not planned.

## 2026-09-22 M0: layout folders appear when first used

Context: PLAN 4 lists `agents/` and `hooks/`; git does not track empty folders.
Decision: no placeholder files. `agents/` arrives with the first agent (M1), `hooks/` with the
gate hook (M2). The validator already checks both when present.
Consequences: the tree matches PLAN 4 only after M2; README shows the full layout anyway.

## 2026-09-22 M0: principles auto-invocation stays probabilistic until after M2

Context: Claude Code invokes a skill when the model judges its `description` relevant; nothing
forces it. In a headless test the same small coding request skipped the skill with a descriptive
wording and invoked it with an imperative one ("Invoke first, before any tool call, whenever...,
however small the task"). One sample each, so the wording helps but is not a guarantee.
Decision: keep description-driven invocation for ad-hoc coding requests. Inside the flow the
skills that write code (`implement`, `fix`, `scaffold`) link `_shared/principles.md` in their own
protocol, so the principles load deterministically there. No hook.
Consequences: a request outside the flow may occasionally run without the principles. Revisit
after M2, once the flow has shipped a real feature: if ad-hoc requests miss the skill often
enough to matter, add a `UserPromptSubmit` hook that reminds the model to invoke it, and record
that here.
