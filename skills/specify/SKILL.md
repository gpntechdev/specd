---
name: specify
description: >
  Invoke whenever the user asks to write, draft or refine the spec, requirements or
  acceptance criteria for a started feature, or runs `/specd:specify [feature]`; also as the
  next step after `/specd:start`. It turns `brief.md` into `spec.md` through a short interview
  (problem, scope, out of scope, verifiable acceptance criteria, open questions), asks you to
  approve it (G1) and commits. Not for opening a feature (`/specd:start`) or for designing
  how it is built (`/specd:design`).
---

# Skill: specify

Writes *what we agreed to build*. The draft goes to disk before the first question, every
answer lands in the file as it arrives, and the gate summary is the user's view of it. The
critic pass has its place marked here and arrives with M3.

## Gate

`state check --step specify` must pass. `rerun: true` (a spec already approved) clears
G1..G5 on write; ask before continuing.

## Inputs

- `resolve-paths`, `detect-repo` (current branch), `state find`, `state check`.
- `<spec_root>/<feature>/brief.md` and `state.yml` (tier, triage signals).
- `<docs_root>/project.md`, `<docs_root>/conventions.md` when present; `sources/<feature>-*.md`
  when the brief cites them.
- `<workspace_root>/specd.yml`: `flow.critic`, `git.authority`.
- [`../_shared/gates.md`](../_shared/gates.md), [`../_shared/state-yml.md`](../_shared/state-yml.md).
- `./templates/spec.md`, [`./references/interview.md`](./references/interview.md).

## Outputs

- `<spec_root>/<feature>/spec.md`; `gates.G1`, `step: design` in `state.yml`.
- One commit `spec(<feature>): specify` (authority ≠ `none`).

## Protocol

1. **Resolve.** Run `resolve-paths`, `detect-repo`, then `"${CLAUDE_PLUGIN_ROOT}/scripts/state"
   find --spec-root <spec_root> [--feature <arg>] --branch <current branch>` and `state check
   --file <state.yml> --step specify` per `gates.md`. Set `step=specify` when it was `start`.
2. **Draft to disk first.** Copy `./templates/spec.md` to `<spec_root>/<feature>/spec.md`
   (draft marker on line 1) and fill what the brief already answers: problem, scope, out of
   scope, acceptance criteria, open questions (start with the brief's "Not said" list),
   provenance. Every AC follows the forms in `interview.md`; an AC that cannot be checked is
   an open question, not an AC.
3. **Interview** per `interview.md`: at most three rounds of at most four grouped questions,
   aimed at the open questions and at ACs that are still vague. Write each answer into the
   file as it arrives. "Don't know" becomes an open question with an owner and stays.
4. **Critic hook point.** <!-- specd:critic-hook: M3 --> When `flow.critic` is `gates` or
   `always` and `agents/critic.md` exists, dispatch it with `spec.md` and `project.md` and fold
   its findings into open questions. Until then say one line: `critic: not available until M3`.
5. **G1** per `gates.md`: path plus at most ten lines (problem in one line, scope and out of
   scope, AC count with the weakest AC named, open questions with owners). Approve: remove the
   marker, `state set gates.G1=now step=design`, clear G2..G5 when re-running, commit
   `spec(<feature>): specify`.
6. **Handoff** per [`../_shared/handoff.md`](../_shared/handoff.md). `Review`: the open questions
   and any AC the user accepted as manual. `Next: /specd:design <feature>`.

## Anti-patterns

- Writing the spec only at the end of the interview; the draft is on disk before question one.
- Restating the brief as the spec. The spec is the agreement, the brief is the ask.
- Acceptance criteria with "should", "properly", "fast" or "user-friendly" in them.
- Deciding architecture here (tables, endpoints, libraries); that is `design`.
- More than three rounds. Leftovers are open questions with owners, not more questions.
